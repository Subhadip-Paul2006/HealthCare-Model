# 🚀 Usage & Deployment Guide — HealthAI India

This guide covers local development, testing, Docker containerization, CI/CD pipelines, and production cloud deployment on Railway or Render.

---

## 🌳 1. Git Branching Strategy & Release History

HealthAI India uses a **GitFlow** branching strategy. All features are developed in isolated branches, tested, then merged into `develop` and released to `main`.

```mermaid
gitGraph
    commit id: "init: project scaffold"
    commit id: "docs: add idea.md roadmap"

    branch develop
    checkout develop
    commit id: "feat: Supabase schema SQL"
    commit id: "feat: auth routes (login/register)"
    commit id: "test: auth unit tests pass"

    branch feature/diabetes-model
    checkout feature/diabetes-model
    commit id: "data: PIMA EDA notebook"
    commit id: "feat: XGBoost training pipeline"
    commit id: "feat: diabetes_pipeline.py class"
    commit id: "feat: POST /predict/diabetes route"
    commit id: "feat: Streamlit diabetes form UI"
    commit id: "test: diabetes integration tests"
    checkout develop
    merge feature/diabetes-model id: "merge: diabetes AI complete"
    commit id: "fix: MinMaxScaler fit on train only"

    branch feature/heart-model
    checkout feature/heart-model
    commit id: "data: Cleveland dataset EDA"
    commit id: "feat: RandomForest heart pipeline"
    commit id: "feat: POST /predict/heart route"
    commit id: "feat: Streamlit heart form UI"
    checkout develop
    merge feature/heart-model id: "merge: heart disease AI complete"

    checkout main
    merge develop tag: "v1.0.0-milestone-2"
    commit id: "release: v1.0.0 Diabetes + Heart"

    checkout develop
    branch feature/stroke-model
    checkout feature/stroke-model
    commit id: "data: SMOTE rebalancing notebook"
    commit id: "feat: LightGBM stroke pipeline"
    commit id: "feat: threshold=0.35 for recall"
    checkout develop
    merge feature/stroke-model id: "merge: stroke AI complete"

    branch feature/llm-assistant
    checkout feature/llm-assistant
    commit id: "feat: Ollama + Gemma integration"
    commit id: "feat: RAG prompt builder"
    commit id: "feat: streaming /explain endpoint"
    checkout develop
    merge feature/llm-assistant id: "merge: LLM assistant ready"

    checkout main
    merge develop tag: "v2.0.0-milestone-6"
    commit id: "release: v2.0.0 Full Platform"
```

---

## 🛠️ 2. Local Development Setup

### Prerequisites
- Python 3.10+
- pip + virtualenv
- A [Supabase](https://supabase.com) project (free tier works)
- [Ollama](https://ollama.ai) installed for local LLM (optional)

### Step 1 — Clone & Configure Environment

```bash
git clone https://github.com/your-org/HealthAI.git
cd HealthAI
```

Create a `.env` file in the project root:

```env
# Supabase Config
SUPABASE_URL=https://your-project-id.supabase.co
SUPABASE_KEY=your-supabase-anon-key
SUPABASE_SERVICE_ROLE_KEY=your-service-role-key

# FastAPI Config
JWT_SECRET=your-supabase-jwt-secret
FASTAPI_URL=http://localhost:8000

# LLM Config (optional)
OLLAMA_URL=http://localhost:11434
LLM_MODEL=gemma:7b
```

### Step 2 — Set Up Supabase Database

```bash
# Run SQL migrations in Supabase SQL Editor (in order):
# 1. database/01_users.sql
# 2. database/02_predictions.sql
# 3. database/03_disease_records.sql
# 4. database/04_feedback.sql
# 5. database/05_rls_policies.sql
```

### Step 3 — Install Backend & Run FastAPI

```bash
cd backend
python -m venv venv
# Windows:
venv\Scripts\activate
# Linux/Mac:
source venv/bin/activate

pip install -r requirements.txt

# Start server with auto-reload
uvicorn main:app --reload --host 0.0.0.0 --port 8000
```

FastAPI interactive docs available at: `http://localhost:8000/docs`

### Step 4 — Install Frontend & Run Streamlit

```bash
# Open a second terminal
cd frontend
python -m venv venv
venv\Scripts\activate  # Windows

pip install -r requirements.txt

streamlit run app.py
```

Streamlit app available at: `http://localhost:8501`

### Step 5 — Pull & Run Local LLM (Optional)

```bash
# Install Ollama from https://ollama.ai then:
ollama pull gemma:7b
ollama serve
# Runs at http://localhost:11434
```

---

## 🧪 3. Testing Pipeline

```mermaid
flowchart LR
    subgraph UnitTests ["🧪 Unit Tests (pytest)"]
        T1["test_diabetes.py\npreprocessing + prediction logic"]
        T2["test_heart.py\nOne-hot encoding validation"]
        T3["test_auth.py\nJWT decode + user profile"]
    end

    subgraph IntegTests ["🔗 Integration Tests (pytest + httpx)"]
        I1["test_api_diabetes.py\nFull POST /predict/diabetes"]
        I2["test_api_feedback.py\nFull POST /feedback"]
        I3["test_supabase_sync.py\nDB insert + RLS check"]
    end

    subgraph E2ETests ["🌐 E2E Tests (Playwright)"]
        E1["test_login_flow.py\nFull login → predict → view history"]
    end

    T1 & T2 & T3 --> |"pytest -v backend/tests/"| Results["Coverage Report"]
    I1 & I2 & I3 --> Results
    E1 --> Results
```

Run tests:

```bash
# Unit + integration tests
cd backend
pytest tests/ -v --cov=. --cov-report=html

# View coverage report
open htmlcov/index.html
```

---

## 🐳 4. Docker Containerization

### Backend Dockerfile (`backend/Dockerfile`)

```dockerfile
FROM python:3.10-slim

WORKDIR /app

# Install dependencies first (layer caching)
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy models directory (large files via Git LFS)
COPY ../models/ /app/models/

# Copy app source
COPY . .

EXPOSE 8000
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000", "--workers", "4"]
```

### Frontend Dockerfile (`frontend/Dockerfile`)

```dockerfile
FROM python:3.10-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 8501
HEALTHCHECK CMD curl --fail http://localhost:8501/_stcore/health || exit 1
CMD ["streamlit", "run", "app.py", \
     "--server.port=8501", \
     "--server.address=0.0.0.0", \
     "--server.headless=true"]
```

### Multi-Container Orchestration (`docker-compose.yml`)

```yaml
version: '3.8'

services:
  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
    container_name: healthai-backend
    ports:
      - "8000:8000"
    environment:
      - SUPABASE_URL=${SUPABASE_URL}
      - SUPABASE_KEY=${SUPABASE_KEY}
      - JWT_SECRET=${JWT_SECRET}
      - OLLAMA_URL=http://ollama:11434
    volumes:
      - ./models:/app/models:ro
    depends_on:
      - ollama
    restart: unless-stopped

  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
    container_name: healthai-frontend
    ports:
      - "8501:8501"
    environment:
      - FASTAPI_URL=http://backend:8000
      - SUPABASE_URL=${SUPABASE_URL}
      - SUPABASE_KEY=${SUPABASE_KEY}
    depends_on:
      - backend
    restart: unless-stopped

  ollama:
    image: ollama/ollama:latest
    container_name: healthai-ollama
    ports:
      - "11434:11434"
    volumes:
      - ollama_data:/root/.ollama
    restart: unless-stopped

volumes:
  ollama_data:
```

```bash
# Build and launch all services
docker-compose up --build -d

# Pull the LLM model into the ollama container
docker exec healthai-ollama ollama pull gemma:7b

# View logs
docker-compose logs -f backend
```

---

## ☁️ 5. CI/CD & Cloud Deployment Pipeline

```mermaid
flowchart TD
    subgraph Dev ["👨‍💻 Developer Workstation"]
        Code["Write Feature Code"] --> LocalTest["Run pytest locally"]
        LocalTest --> GitPush["git push origin feature/xyz"]
    end

    subgraph GitHub ["📦 GitHub Repository"]
        PR["Open Pull Request to develop"] --> Review["Code Review"]
        Review --> Merge["Merge to develop"]
        Merge --> MainMerge["Merge develop to main\n(Tag release)"]
    end

    subgraph CI ["⚙️ GitHub Actions CI Pipeline"]
        Trigger["On push to main"] --> Lint["flake8 linting"]
        Lint --> UnitTest["pytest unit tests"]
        UnitTest --> IntTest["pytest integration tests"]
        IntTest --> DockerBuild["docker build backend + frontend"]
        DockerBuild --> DockerPush["Push images to\nGitHub Container Registry"]
    end

    subgraph Railway ["🚂 Railway Cloud Hosting"]
        RailBack["Backend Service\n(FastAPI Docker)"]
        RailFront["Frontend Service\n(Streamlit Docker)"]
        RailBack <-->|"Internal networking"| RailFront
    end

    subgraph SupabaseCloud ["🗄️ Supabase Cloud DB"]
        SupaDB[("PostgreSQL + RLS\n+ Auth Service")]
    end

    GitPush --> PR
    MainMerge --> Trigger
    DockerPush --> |"Railway auto-deploys\nlatest image"| RailBack
    DockerPush --> RailFront
    RailBack <-->|"Supabase SDK"| SupaDB
```

### Railway Deployment Steps

1. Go to [railway.app](https://railway.app) → **New Project** → **Deploy from GitHub**
2. Add **Backend Service**: set Root Directory to `/backend`, auto-detects Dockerfile.
3. Add **Frontend Service**: set Root Directory to `/frontend`.
4. Set environment variables in Railway dashboard for each service.
5. Railway auto-assigns public URLs with SSL. Copy the backend URL into the frontend's `FASTAPI_URL` variable.
6. Enable **Auto-Deploy on push to main** in Railway settings.

### Health Check Endpoints

```
GET http://your-backend-url/health
→ {"status": "ok", "models_loaded": 6, "db_connected": true}
```
