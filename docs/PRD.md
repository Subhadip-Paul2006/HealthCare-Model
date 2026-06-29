# 📋 Product Requirements Document (PRD) — HealthAI India

HealthAI India is a modular, AI-powered preventive healthcare platform to democratize access to health risk assessments. It combines clinical ML models, psychometric evaluations, LLM-powered explanations, and interactive dashboards.

---

## 🎯 1. Product Vision & Goals

1. **Preventive Risk Analysis**: Early screening for Diabetes, Heart Disease, Stroke.
2. **Holistic Health Mapping**: Mental wellbeing, sleep quality, and personality tracking.
3. **Explainable AI (XAI)**: LLMs (Gemma/Llama) translate model outputs to plain-English recommendations.
4. **Consented Health Registry**: Strict user ownership with consent logging in Supabase.
5. **Continuous Improvement**: Citizen-science data flywheel — user feedback retrains models.

---

## 🧠 2. Product Architecture & Feature Mindmap

```mermaid
mindmap
  root((HealthAI India Platform))
    Physiological AI Models
      Diabetes Prediction
        PIMA Indians Dataset
        XGBoost Classifier
        BMI Auto-calculation
      Heart Disease Prediction
        Cleveland Clinic Dataset
        Random Forest
        13 Clinical Indicators
      Stroke Prediction
        Kaggle Stroke Dataset
        LightGBM + SMOTE
        Threshold 0.35 for Recall
    Psychological AI Models
      OCEAN Personality
        10-item TIPI Scale
        K-Means Archetypes
        Reverse Scoring Logic
      Mental Health Assessment
        OSMI Survey Dataset
        Workplace & Support Factors
        Random Forest
      Sleep Health Predictor
        Sleep Lifestyle Dataset
        Multi-class XGBoost
        3 Disorder Categories
    Core Platform Architecture
      FastAPI Backend
        JWT Auth Middleware
        Pydantic Validation
        Modular Routers
        Model Cache Layer
      Streamlit Frontend
        Session State Management
        Multi-page Navigation
        Plotly Dashboards
      Supabase PostgreSQL
        Row Level Security
        Auth & Profiles
        12 Relational Tables
        Consent Logs
    AI Assist & Analytics
      LLM Explainability
        Gemma Local Model
        Llama via Ollama
        RAG with Health History
      Power BI Dashboards
        Disease Distribution
        Risk Trend Over Time
        User Demographics
      Health Report Generation
        Per-user PDF Export
        Trend Analysis
    Continuous Learning System
      Feedback Loop
        Binary Feedback Correct/Wrong
        Confidence Rating
      Retraining Pipeline
        Weekly Cron Job
        Verified Data Only
        Version-Controlled Models
      Data Governance
        Consent-Only Data Use
        Anonymization Layer
```

---

## 👥 3. User Personas & Journey Maps

### Persona 1: Raj (42, Software Engineer)
- **Need**: Family history of diabetes; wants frictionless self-screening.
- **Behavior**: High digital literacy, values privacy, seeks direct explanations.

### Persona 2: Dr. Priya (38, General Physician)
- **Need**: Wants a second-opinion screening tool for patients with limited lab access.
- **Behavior**: Trusts data, wants to see model confidence and feature importance.

### Persona 3: Anjali (26, Graduate Student)
- **Need**: Experiencing sleep disruption and anxiety; wants holistic mental + sleep assessment.
- **Behavior**: Prefers conversational UI, concerned about data privacy.

---

### User Journey: Raj — Diabetes Self-Screening

```mermaid
journey
    title Raj's Diabetes Screening Journey on HealthAI India
    section Discovery & Onboarding
      Finds HealthAI via Google search: 4: Raj
      Reads landing page features: 5: Raj
      Clicks Sign Up with email: 4: Raj, Supabase Auth
      Verifies email and logs in: 3: Raj, Supabase Auth
      Completes demographic profile: 4: Raj, FastAPI
    section First Prediction — Diabetes
      Selects Diabetes module from dashboard: 5: Raj
      Enters height, weight, blood pressure: 4: Raj
      BMI auto-calculated: 5: Raj, FastAPI
      Enters family history pedigree score: 3: Raj
      Submits form: 5: Raj
      Sees High Risk 76% result instantly: 5: Raj, XGBoost Model
      Clicks Explain this result: 4: Raj
      Reads AI-generated lifestyle report: 5: Raj, Gemma LLM
    section Consent & Data Storage
      Prompted to consent to save record: 4: Raj, Consent Manager
      Grants consent for research use: 5: Raj, Supabase
      Record saved to diabetes_records: 5: Supabase
    section Ongoing Usage
      Returns 3 months later for recheck: 4: Raj
      Views historical risk trend chart: 5: Raj, Power BI
      Provides feedback on last prediction: 3: Raj
      Contributes to retraining dataset: 5: Raj, Research
```

---

### User Journey: Anjali — Mental Health + Sleep Assessment

```mermaid
journey
    title Anjali's Mental & Sleep Health Journey
    section Onboarding
      Discovers platform via college wellness app: 5: Anjali
      Registers anonymously with username: 4: Anjali, Supabase Auth
      Chooses to skip demographic profile: 3: Anjali
    section Sleep Assessment
      Selects Sleep Health module: 5: Anjali
      Enters sleep duration 5 hrs, stress 8/10: 4: Anjali
      Selects occupation and BMI category: 4: Anjali
      Views Insomnia High Risk result: 4: Anjali, XGBoost Model
      Reads LLM sleep hygiene recommendations: 5: Anjali, Gemma LLM
    section Mental Health Assessment
      Also completes Mental Health questionnaire: 5: Anjali
      Answers 9 workplace and support questions: 3: Anjali
      Views Treatment Recommended result: 4: Anjali, Random Forest
      Reads empathetic AI-generated guidance: 5: Anjali, Gemma LLM
    section Privacy & Feedback
      Opts out of data storage: 3: Anjali, Consent Manager
      Provides anonymous feedback on accuracy: 4: Anjali
```

---

## 📅 4. Roadmap & Gantt Diagram

```mermaid
gantt
    title HealthAI India — Full Project Roadmap
    dateFormat  YYYY-MM-DD
    section Phase 0: Foundation
    Dataset Collection & Research      :done,    2026-06-01, 2026-06-07
    Supabase Schema Design             :done,    2026-06-05, 2026-06-10
    Auth & Session Implementation      :active,  2026-06-08, 2026-06-12
    Documentation System               :active,  2026-06-25, 2026-06-29
    section Phase 1: Diabetes AI
    PIMA Dataset EDA & Cleaning        :done,    2026-06-13, 2026-06-15
    Feature Engineering & XGBoost      :active,  2026-06-15, 2026-06-18
    Streamlit Diabetes UI              :         2026-06-19, 2026-06-22
    FastAPI Diabetes Route             :         2026-06-20, 2026-06-23
    Supabase Integration & Testing     :         2026-06-23, 2026-06-29
    section Phase 2: Heart Disease AI
    Cleveland Dataset Preprocessing    :         2026-06-30, 2026-07-03
    Random Forest Training & Tuning    :         2026-07-03, 2026-07-06
    UI + API + DB Integration          :         2026-07-07, 2026-07-12
    section Phase 3: Stroke AI
    Stroke Dataset Cleaning & SMOTE    :         2026-07-13, 2026-07-16
    LightGBM Training & Threshold Opt  :         2026-07-16, 2026-07-19
    Full Integration & Testing         :         2026-07-19, 2026-07-22
    section Phase 4: Personality AI
    TIPI Scoring Algorithm             :         2026-07-23, 2026-07-26
    K-Means Archetype Clustering       :         2026-07-26, 2026-07-30
    Assessment UI & Integration        :         2026-07-30, 2026-08-02
    section Phase 5: Mental Health AI
    OSMI Dataset Preprocessing         :         2026-08-03, 2026-08-06
    Random Forest Training             :         2026-08-06, 2026-08-08
    UI + API + DB Integration          :         2026-08-08, 2026-08-10
    section Phase 6: Sleep Health AI
    Sleep Lifestyle Dataset EDA        :         2026-08-11, 2026-08-14
    Multi-class XGBoost Training       :         2026-08-14, 2026-08-16
    UI + API + DB Integration          :         2026-08-16, 2026-08-18
    section Phase 7: AI Assistant
    Ollama + Gemma Setup               :         2026-08-19, 2026-08-22
    RAG Prompt Engineering             :         2026-08-22, 2026-08-25
    Health Report Generation           :         2026-08-25, 2026-08-28
    section Phase 8: Analytics
    Power BI Supabase Connection       :         2026-08-29, 2026-09-01
    Disease Distribution Dashboards    :         2026-09-01, 2026-09-05
    section Phase 9 & 10: CI/CD & Deploy
    Feedback Loop & Retraining Cron    :         2026-09-06, 2026-09-12
    Docker Compose Build               :         2026-09-12, 2026-09-15
    Railway / Render Production Deploy :         2026-09-15, 2026-09-20
```

---

## 📊 5. Feature Effort Allocation & Risk Distribution

### Development Effort Allocation
```mermaid
pie title Development Effort by Module (%)
    "Phase 0: Foundation & DB" : 12
    "Diabetes Prediction AI" : 14
    "Heart Disease AI" : 12
    "Stroke Prediction AI" : 10
    "Personality Assessment AI" : 10
    "Mental Health AI" : 10
    "Sleep Health AI" : 10
    "AI Assistant (LLM)" : 12
    "Power BI & Deployment" : 10
```

---

## 📈 6. Success Metrics (KPIs)

| Metric | Target | Measurement Method |
|:---|:---:|:---|
| F1-Score (Physiological models) | ≥ 0.80 | sklearn classification_report |
| F1-Score (Psychological models) | ≥ 0.75 | sklearn classification_report |
| Stroke Recall (Sensitivity) | ≥ 0.80 | True Positive Rate @ threshold 0.35 |
| API Inference Latency | ≤ 350ms | FastAPI response time middleware |
| LLM First Token Latency | ≤ 1.5s | Streamlit timer |
| User Feedback Rate | ≥ 15% | Supabase feedback count / prediction count |
| DAU (Daily Active Users) | 500+ after 3 months | Supabase auth logs |
| Consent Grant Rate | ≥ 70% | consent_logs table |
