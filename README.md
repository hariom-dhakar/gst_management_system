# 🧾 GSTSmart — AI-Powered GST Return Management System

> A production-grade, full-stack SaaS platform for automated GST compliance, hybrid AI invoice parsing, and advanced tax forecasting for Indian small businesses.

![Dashboard Preview](docs/screenshots/dashboard.png)

---

## 🌟 Overview

**GSTSmart** is a comprehensive solution designed to simplify the complex world of GST compliance for Indian SMBs. By leveraging cutting-edge AI for automated data entry and sophisticated time-series models for tax forecasting, GSTSmart transforms raw invoices into actionable financial insights.

Built with a focus on **visual excellence** and **operational reliability**, the platform features a premium glassmorphic interface, high-performance background processing, and a robust hybrid OCR pipeline.

---

## 🚀 Key Features

| Feature | Technical Implementation |
|---------|-------------------------|
| 📄 **Hybrid AI Parsing** | **PyMuPDF** text extraction with **RapidOCR** fallback for scanned images. |
| 📊 **Precision GST Engine** | Automated HSN-aware CGST/SGST/IGST splitting and ITC eligibility tracking. |
| 📈 **Advanced Forecasting** | Dual-engine **Prophet + ARIMA** monthly GST liability predictions. |
| 📋 **Return Management** | One-click GSTR-style summaries and multi-month return history. |
| 📥 **Excel Data Export** | High-fidelity **.xlsx** reporting with custom styling and filtered exports. |
| 🔐 **Enterprise Auth** | Secure **JWT-based** authentication with Role-Based Access Control (RBAC). |
| 💎 **Premium UI/UX** | **Framer Motion** animations, **Tailwind CSS** glassmorphism, and **Recharts** analytics. |
| 📱 **Mobile First** | Fully responsive architecture with a dedicated sliding sidebar for on-field usage. |

---

## 🛠 Tech Stack

### Frontend
- **Framework:** React 18 (Vite)
- **Styling:** Tailwind CSS + Vanilla CSS
- **Animations:** Framer Motion
- **Visualization:** Recharts
- **State Management:** React Context API + Hook Form

### Backend
- **Framework:** FastAPI (Python 3.11+)
- **Database:** MongoDB 7.0 (Beanie ODM / Motor)
- **Security:** JWT + Bcrypt (v3.2.2 compatibility layer)
- **Task Mgmt:** Asynchronous I/O with Python `asyncio`

### AI / Machine Learning
- **OCR Engine:** RapidOCR (ONNX Runtime) + PyTesseract
- **Document Rendering:** PyMuPDF (fitz)
- **Time-Series:** Facebook Prophet + ARIMA
- **Feature Extraction:** Regex-Hybrid NLP Pipeline

---

## ⚡ Quick Start

### 1. Backend Setup
```bash
cd backend
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate
pip install -r requirements.txt
uvicorn main:app --reload --port 8000
```

### 2. Frontend Setup
```bash
cd frontend
npm install
npm run dev
```

### 3. Database
Ensure **MongoDB** is running locally or provide a `MONGODB_URL` in your `.env`.

---

## 📁 Project Architecture

```text
gst-management-system/
├── frontend/                 # React + Vite SPA
│   ├── src/
│   │   ├── components/       # Reusable UI & Layouts
│   │   ├── pages/            # Feature-specific Page Views
│   │   ├── services/         # Axios API Integration
│   │   └── utils/            # Formatters & Business Logic
├── backend/                  # FastAPI Application
│   ├── app/
│   │   ├── api/v1/routes/    # REST Endpoints (Auth, Invoices, Forecast)
│   │   ├── ml/               # AI Engine (OCR, Forecasting)
│   │   ├── models/           # Beanie ODM Schema Definitions
│   │   ├── services/         # Core Business Logic Layer
│   │   └── core/             # Auth, Security, Config
├── datasets/                 # Demo Seed Data & Scripts
└── docs/                     # Technical Handbooks & Screenshots
```

---

## 🔌 API Ecosystem (v1)

| Method | Route | Feature |
|--------|-------|---------|
| `POST` | `/api/v1/auth/login` | Secure JWT Session Generation |
| `POST` | `/api/v1/invoices/upload` | Hybrid Parse + Store Document |
| `GET` | `/api/v1/invoices/export` | Excel Report Generation (.xlsx) |
| `PUT` | `/api/v1/invoices/{id}/review` | Human-in-the-loop Correction |
| `GET` | `/api/v1/forecast/next-month` | Predictive Tax Liability |
| `GET` | `/api/v1/reports/tax-summary` | Real-time Monthly Analytics |

---

## 🤖 Deep Dive: AI Pipeline

### Hybrid Invoice Parsing
The system uses a sophisticated 2-stage extraction process:
1. **Digital Extraction:** Attempts native text layer extraction using `PyMuPDF` for high speed and 100% accuracy on digital PDFs.
2. **Vision Extraction:** If the document is an image or scanned PDF, it renders pages as high-DPI images and uses the `RapidOCR` ONNX-powered engine for robust field detection.

### Smart Forecasting
Unlike simple averages, GSTSmart uses **Facebook Prophet** to account for seasonality (e.g., higher sales during festive months) and **ARIMA** for short-term trend validation, providing a confidence interval for next-month tax planning.

---

## 🔮 Roadmap
- [ ] Multi-GSTIN corporate support.
- [ ] GSP direct filing API integration.
- [ ] T+1 Real-time tax liability alerts.
- [ ] Bulk invoice ZIP processing.
