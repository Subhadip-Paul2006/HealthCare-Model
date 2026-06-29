# 🛠️ Technical Requirements Document (TRD) — HealthAI India

This document defines the full system architecture, C4 models, interaction sequences, data pipeline logic, API specifications, and security constraints.

---

## 🏗️ 1. System Architecture & Block Diagram

HealthAI India uses a decoupled, three-tier architecture: Streamlit frontend → FastAPI gateway → Supabase cloud database, with an AI Inference Engine as a co-located service.

```mermaid
flowchart TD
    subgraph FrontendBlock ["🖥️ Streamlit Frontend"]
        AppUI["app.py — Main Multi-page Router"]
        AuthUI["auth_ui.py — Login / Register / Reset"]
        FormUI["Prediction Form Widgets (sliders, selects)"]
        HistoryUI["Health History & Trend Charts"]
        LLMUI["AI Report Viewer (streaming markdown)"]
        AppUI --> AuthUI
        AppUI --> FormUI
        AppUI --> HistoryUI
        AppUI --> LLMUI
    end

    subgraph BackendBlock ["⚙️ FastAPI Backend Gateway"]
        CoreAPI["FastAPI App (main.py)"]
        AuthMiddleware["JWT Verification Middleware"]
        PydanticLayer["Pydantic Request Schema Validation"]
        ModelCache["In-Memory Model Cache (app.state)"]
        PreProcEngine["Preprocessing Engine"]
        InferenceEngine["ML Inference Engine"]
        FeedbackHandler["Feedback Route Handler"]
        LLMProxy["LLM Proxy Route (Ollama/Gemma)"]
        CoreAPI --> AuthMiddleware --> PydanticLayer
        PydanticLayer --> ModelCache --> PreProcEngine --> InferenceEngine
        InferenceEngine --> FeedbackHandler
        CoreAPI --> LLMProxy
    end

    subgraph AIBlock ["🤖 AI Inference Models"]
        DiabetesM["diabetes_model.pkl + scaler.pkl"]
        HeartM["heart_model.pkl + scaler.pkl"]
        StrokeM["stroke_model.pkl + scaler.pkl"]
        PersonalM["personality_kmeans.pkl"]
        MentalM["mental_model.pkl + encoder.pkl"]
        SleepM["sleep_model.pkl + scaler.pkl"]
    end

    subgraph SupabaseBlock ["🗄️ Supabase Storage Layer"]
        AuthTbl[("users + user_profiles")]
        PredictTbl[("predictions")]
        RecordsTbl[("6x disease_records tables")]
        FeedbackTbl[("feedback + consent_logs")]
    end

    FrontendBlock <-->|"HTTPS REST + Bearer JWT"| CoreAPI
    InferenceEngine --> DiabetesM
    InferenceEngine --> HeartM
    InferenceEngine --> StrokeM
    InferenceEngine --> PersonalM
    InferenceEngine --> MentalM
    InferenceEngine --> SleepM
    CoreAPI <-->|"Supabase Client SDK"| AuthTbl
    InferenceEngine -.->|"Save prediction"| PredictTbl
    InferenceEngine -.->|"Save disease record"| RecordsTbl
    FeedbackHandler -.->|"Save feedback"| FeedbackTbl
```

---

## 🎛️ 2. C4 Architecture Diagrams

### C4 Level 1 — System Context

Shows the entire HealthAI India system as a black box with its external actors and integrations:

```mermaid
graph TD
    Patient([Patient / End User]) <-->|"Uses Web App via Browser"| HealthAI["HealthAI India Platform\n[Python / Streamlit / FastAPI]"]
    Doctor([Healthcare Professional]) <-->|"Reviews patient reports"| HealthAI
    HealthAI <-->|"User Auth, DB Reads/Writes\n[Supabase SDK over HTTPS]"| Supabase["Supabase Cloud\n[PostgreSQL + Auth + RLS]"]
    HealthAI <-->|"LLM Inference via REST\n[Ollama / OpenAI API]"| LLM["LLM Service\n[Gemma 7B / Llama 3 / GPT-4o]"]
    HealthAI <-->|"Analytics Queries\n[DirectQuery / ODBC]"| PowerBI["Power BI Service\n[Microsoft Cloud]"]
    HealthAI <-->|"Model deployment artifacts\n[GitHub Actions]"| GitHub["GitHub Repository\n[CI/CD Pipeline]"]
```

---

### C4 Level 2 — Container Diagram

Shows the individual deployable units inside HealthAI India:

```mermaid
graph TB
    subgraph UserBrowser ["User Browser"]
        BrowserUI["Streamlit Web App\n[Python 3.10, Streamlit 1.31]\nPort: 8501"]
    end

    subgraph APIServer ["API Server Container"]
        FastAPIApp["FastAPI Application\n[Python 3.10, FastAPI 0.109]\nPort: 8000"]
        MLRuntime["ML Inference Runtime\n[scikit-learn, XGBoost, LightGBM]\nLoaded at startup"]
        FastAPIApp <-->|"In-process call"| MLRuntime
    end

    subgraph SupabaseCloud ["Supabase Cloud"]
        SupaAuth["Supabase Auth Service\n[GoTrue JWT]"]
        SupaDB[("Supabase PostgreSQL\n[12 Tables + RLS Policies]")]
        SupaRealtime["Supabase Realtime\n[WebSocket for live updates]"]
    end

    subgraph LLMServer ["LLM Inference Server"]
        OllamaRuntime["Ollama Runtime\n[Gemma 7B or Llama 3 8B]\nPort: 11434"]
    end

    BrowserUI <-->|"HTTP REST + JSON\nBearer JWT Token"| FastAPIApp
    BrowserUI <-->|"Supabase JS SDK\nAuth Token Exchange"| SupaAuth
    FastAPIApp <-->|"Supabase Python SDK\nSQL SELECT/INSERT"| SupaDB
    FastAPIApp <-->|"POST /api/generate\nJSON prompt"| OllamaRuntime
    SupaAuth -->|"JWT Validation"| SupaDB
    SupaDB -->|"Change notifications"| SupaRealtime
    SupaRealtime -->|"WebSocket push"| BrowserUI
```

---

### C4 Level 3 — Component Diagram (FastAPI Backend)

Internal component breakdown of the FastAPI application:

```mermaid
graph TD
    subgraph FastAPIComponents ["FastAPI Application Components"]
        Router["APIRouter Registry\nmain.py — includes all sub-routers"]
        AuthComp["auth.py Router\nPOST /auth/login\nPOST /auth/register\nPOST /auth/refresh"]
        DiabComp["diabetes.py Router\nPOST /predict/diabetes\nGET /history/diabetes"]
        HeartComp["heart.py Router\nPOST /predict/heart\nGET /history/heart"]
        StrokeComp["stroke.py Router\nPOST /predict/stroke"]
        PersonComp["personality.py Router\nPOST /predict/personality"]
        MentalComp["mental.py Router\nPOST /predict/mental"]
        SleepComp["sleep.py Router\nPOST /predict/sleep"]
        FeedComp["feedback.py Router\nPOST /feedback"]
        LLMComp["llm.py Router\nPOST /explain"]
    end

    subgraph SharedServices ["Shared Services"]
        AuthDep["get_current_user()\nJWT Dependency"]
        DBClient["SupabaseSyncManager\ndatabase.py"]
        ModelLoader["ModelLoader\nmodels/loader.py"]
    end

    Router --> AuthComp
    Router --> DiabComp & HeartComp & StrokeComp
    Router --> PersonComp & MentalComp & SleepComp
    Router --> FeedComp & LLMComp
    DiabComp & HeartComp & StrokeComp --> AuthDep
    PersonComp & MentalComp & SleepComp --> AuthDep
    DiabComp & HeartComp --> ModelLoader & DBClient
    StrokeComp & PersonComp & MentalComp & SleepComp --> ModelLoader & DBClient
```

---

## 🔄 3. Interaction Sequence Diagrams

### 3.1 Full Prediction Request Flow

```mermaid
sequenceDiagram
    autonumber
    actor User as Patient (Browser)
    participant UI as Streamlit Frontend
    participant API as FastAPI Backend
    participant Cache as Model Cache
    participant ML as ML Inference Engine
    participant DB as Supabase Database
    participant LLM as Gemma/Ollama LLM

    User->>UI: Fill Diabetes form and click Submit
    UI->>UI: Client-side validation (required fields)
    UI->>API: POST /api/predict/diabetes + Bearer JWT
    API->>API: JWT Middleware: verify token signature & expiry
    API->>API: Pydantic: validate request body schema
    API->>Cache: Request DiabetesPipeline instance
    Cache-->>API: Return cached model + scaler objects
    API->>ML: preprocess(raw_data) → scaled_array
    ML-->>API: Return numpy array shape (1, 8)
    API->>ML: model.predict_proba(array)
    ML-->>API: Return [[0.24, 0.76]]
    API->>API: Apply threshold: 0.76 >= 0.5 → Class 1
    API->>DB: INSERT INTO predictions {user_id, type, probability, label}
    DB-->>API: Return prediction_id = "uuid-xxxx"
    API->>DB: INSERT INTO diabetes_records {prediction_id, all_features}
    DB-->>API: Confirm insert OK
    API-->>UI: JSON {risk_probability: 0.76, label: 1, prediction_id: "uuid-xxxx"}
    UI->>User: Display High Risk card with probability gauge

    Note over User, LLM: User clicks "Explain My Results"
    User->>UI: Click Explain button
    UI->>API: POST /api/explain {prediction_id}
    API->>DB: SELECT history WHERE user_id = uid LIMIT 5
    DB-->>API: Return last 5 predictions
    API->>LLM: POST /api/generate {prompt with context + clinical guidelines}
    LLM-->>API: Stream tokens (health report)
    API-->>UI: Server-sent event stream
    UI->>User: Display streaming markdown health report
```

---

### 3.2 User Authentication Sequence

```mermaid
sequenceDiagram
    autonumber
    actor User as User
    participant UI as Streamlit UI
    participant Supa as Supabase Auth
    participant API as FastAPI Backend
    participant DB as Supabase DB

    User->>UI: Enter email + password on Login form
    UI->>Supa: supabase.auth.sign_in_with_password({email, password})
    Supa-->>UI: Return access_token (JWT) + refresh_token
    UI->>UI: Store tokens in st.session_state
    UI->>API: GET /api/profile with Bearer JWT
    API->>API: Decode JWT → extract user_id
    API->>DB: SELECT * FROM user_profiles WHERE user_id = uid
    DB-->>API: Return profile row
    API-->>UI: Return profile JSON
    UI->>User: Render dashboard with user's name
    Note over UI, Supa: Token Refresh Flow (every 3600s)
    UI->>Supa: supabase.auth.refresh_session(refresh_token)
    Supa-->>UI: Return new access_token
```

---

## 📈 4. Advanced Prediction Lifecycle Flowchart

Complete logic covering validation, preprocessing branching, inference, threshold decisions, DB transactions, and error handling:

```mermaid
flowchart TD
    START([User Submits Prediction Form]) --> Validate{Pydantic\nSchema Valid?}
    Validate -- No --> Error422["Return HTTP 422\nValidation Error to UI"]
    Validate -- Yes --> AuthCheck{JWT Token\nValid & Not Expired?}
    AuthCheck -- No --> Error401["Return HTTP 401\nUnauthorized"]
    AuthCheck -- Yes --> ModelRoute{Which Model\nType Requested?}

    ModelRoute --> |diabetes| DiabLoad["Load DiabetesPipeline\nfrom app.state cache"]
    ModelRoute --> |heart| HeartLoad["Load HeartDiseasePipeline"]
    ModelRoute --> |stroke| StrokeLoad["Load StrokePipeline"]
    ModelRoute --> |personality| PersonLoad["Load PersonalityPipeline"]
    ModelRoute --> |mental| MentalLoad["Load MentalHealthPipeline"]
    ModelRoute --> |sleep| SleepLoad["Load SleepHealthPipeline"]

    subgraph PreprocessingEngine ["🔧 Data Preprocessing Engine"]
        DiabLoad --> NullD{Missing\nValues?}
        NullD -- Yes --> KNN["KNN Imputation K=5"]
        NullD -- No --> BMICalc["Auto-calculate BMI\nif not provided"]
        KNN --> BMICalc
        BMICalc --> ScaleD["MinMaxScaler.transform()"]

        HeartLoad --> NullH{Missing\nValues?}
        NullH -- Yes --> MedianH["Median Imputation"]
        NullH -- No --> EncodeH["One-Hot Encode:\nCP, RestECG, Slope, Thal"]
        MedianH --> EncodeH --> ScaleH["StandardScaler.transform()"]

        StrokeLoad --> NullS{BMI\nMissing?}
        NullS -- Yes --> MedianS["Median Impute BMI"]
        NullS -- No --> EncodeS["One-Hot Encode:\nGender, Work, Residence, Smoking"]
        MedianS --> EncodeS --> ScaleS["StandardScaler.transform()"]

        PersonLoad --> ReverseCode["Apply Reverse Scoring:\n6 - score for items 2,6,8,9,10"]
        ReverseCode --> OCEANCalc["Calculate O, C, E, A, N\nscores as averages"]
        OCEANCalc --> Normalize["Scale to 0-100%"]
    end

    ScaleD --> InferD["XGBClassifier.predict_proba()"]
    ScaleH --> InferH["RandomForest.predict_proba()"]
    ScaleS --> InferS["LGBMClassifier.predict_proba()"]
    Normalize --> InferP["KMeans.predict() → Archetype"]

    InferD --> ThreshD{Prob >= 0.50?}
    ThreshD -- Yes --> HighD["Label: High Risk Diabetes"]
    ThreshD -- No --> LowD["Label: Low Risk Diabetes"]

    InferS --> ThreshS{Prob >= 0.35\nLow threshold for Recall}
    ThreshS -- Yes --> HighS["Label: High Stroke Risk"]
    ThreshS -- No --> LowS["Label: Low Stroke Risk"]

    HighD & LowD & HighS & LowS & InferH & InferP --> ConsentCheck{User Consent\nGranted?}
    ConsentCheck -- No --> ReturnOnly["Return prediction JSON\nDo NOT save to DB"]
    ConsentCheck -- Yes --> SavePred["INSERT INTO predictions"]
    SavePred --> SaveRecord["INSERT INTO disease-specific table"]
    SaveRecord --> SaveConsent["UPDATE consent_logs timestamp"]
    SaveConsent --> Response["Return 200 OK\nwith prediction payload"]
    ReturnOnly --> Response
    Response --> END([UI Renders Prediction Card])
```

---

## 🔒 5. Security & Performance Constraints

### Authentication & Authorization
- All `/api/predict/*` and `/api/history/*` routes require a `Bearer <JWT>` token issued by Supabase Auth.
- Supabase RLS enforces `user_id = auth.uid()` on every table — no cross-user data leakage is possible at the DB layer.
- FastAPI `Depends(get_current_user)` dependency injects the verified user on every protected route.
- Tokens expire after 3600 seconds; refresh tokens are rotated on every refresh.

### Performance Requirements

| Constraint | Target | Implementation |
|:---|:---:|:---|
| ML Inference Latency | ≤ 100ms | Models cached in `app.state` on startup |
| API Round-trip (predict) | ≤ 350ms | Async DB writes using `BackgroundTasks` |
| LLM First Token | ≤ 1.5s | Streaming via Server-Sent Events |
| Concurrent Users | 100+ | Uvicorn `--workers 4` + async endpoints |
| DB Query Time | ≤ 50ms | Indexed on `user_id` and `created_at` |
