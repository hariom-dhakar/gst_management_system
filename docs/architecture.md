# GSTSmart — System Architecture

## Overview

GSTSmart is a full-stack AI-powered GST compliance platform built as a monorepo with a React frontend, FastAPI backend, and MongoDB database.

```
┌─────────────────────────────────────────────────────────────────┐
│                         BROWSER (React SPA)                     │
│  Login → Dashboard → Upload → Invoices → Reports → Forecast     │
└───────────────────────────┬─────────────────────────────────────┘
                            │ REST API (JWT Auth)
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│                      FastAPI Backend                             │
│                                                                  │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────────┐   │
│  │  Auth    │  │ Invoice  │  │  Returns │  │   Forecast   │   │
│  │  Routes  │  │  Routes  │  │  Routes  │  │   Routes     │   │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘  └──────┬───────┘   │
│       │             │              │                │           │
│  ┌────▼─────────────▼──────────────▼────────────────▼───────┐  │
│  │                    Service Layer                           │  │
│  │  gst_service  │  fraud_service  │  forecast_service       │  │
│  └──────────────────────┬──────────────────────────────────┘  │
│                         │                                       │
│  ┌──────────────────────▼──────────────────────────────────┐   │
│  │                    ML Pipeline                           │   │
│  │  invoice_parser │ fraud_detection │ forecasting          │   │
│  │  (hybrid OCR+LLM│ (IsolationForest│ (Prophet/ARIMA)      │   │
│  │  +regex)        │ +rules)         │ +WMA fallback)        │   │
│  └──────────────────────┬──────────────────────────────────┘   │
└───────────────────────── │──────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│                       MongoDB                                    │
│  users │ business_profiles │ invoices │ monthly_returns          │
│  forecasts │ audit_logs                                          │
└─────────────────────────────────────────────────────────────────┘
```

## Invoice Processing Pipeline

```
Upload → Store File → Background Task:
  1. Digital Text Extraction (PyMuPDF - fast if digital text exists)
  2. OCR Fallback (if no digital text: render to 250 DPI image + RapidOCR)
  3. LLM Structuring (raw text → Groq Llama-3.3-70b → JSON schema validation)
  4. Field Extraction (regex on structured text)
  5. GST Calculation (intra/inter-state classification)
  6. Fraud Detection (rule-based + Isolation Forest)
  7. Status Update → Notify frontend
```

## Authentication Flow

```
POST /auth/login → Verify credentials → Issue JWT (24h)
All protected routes: Bearer <token> → decode → get user
Role check: admin > accountant > business_owner
```

## Key Design Decisions

- **Beanie ODM** over raw pymongo for clean async MongoDB models
- **Background tasks** (FastAPI native) over Celery for simplicity
- **Fallback-first ML** — every ML model has a rule-based fallback so the app runs on any hardware
- **Hybrid OCR Pipeline** — PyMuPDF digital text → RapidOCR → LLM structuring (Groq Llama-3.3-70b)
- **Isolation Forest** trained on synthetic normal data at startup (no separate training step)
- **Prophet with ARIMA and WMA fallbacks** for time-series forecasting
- **Demo data** — seeded on first run so judges see real data immediately
