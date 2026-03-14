# 🧾 GSTSmart — AI-Powered GST Return Management System

> A production-grade, full-stack SaaS platform for automated GST compliance, hybrid AI invoice parsing, and advanced tax forecasting for Indian small businesses.

![Dashboard Preview](docs/screenshots/dashboard.png)

---

## 🌟 Overview

**GSTSmart** is a comprehensive solution designed to simplify the complex world of GST compliance for Indian SMBs. By leveraging cutting-edge Artificial Intelligence for automated data entry and sophisticated time-series models for tax forecasting, GSTSmart transforms raw invoices into actionable financial insights.

Built with an intense focus on **visual excellence** and **operational reliability**, the platform features a premium glassmorphic interface, high-performance asynchronous background processing, and a highly resilient hybrid OCR pipeline.

---

## 🚀 Exhaustive Feature Map

### 1. Invoice Management & Hybrid Intelligence Pipeline
- **Smart Drag & Drop Upload**: Upload PDF, JPG, or PNG files (up to 10MB) with immediate UI feedback and animated transitions.
- **Asynchronous Processing**: Invoices are ingested immediately while Heavy ML processing runs quietly in the FastAPI background tasks via `asyncio.to_thread`.
- **Hybrid Extraction Engine (`parser.py` & `extractor.py`)**: 
  - **Stage 1 (Digital Text)**: Attempts native extraction using `PyMuPDF` (`fitz`). If successful, yields 100% accurate text at incredibly high speeds.
  - **Stage 2 (Vision/OCR)**: If no digital text is found, renders the document to high-DPI images and passes them through an embedded `RapidOCR` (ONNX Runtime) engine for state-of-the-art layout-aware text recognition.
  - **Stage 3 (LLM Structuring)**: The raw text is passed to an ultra-fast **Llama-3.3-70b** LLM (hosted via the **Groq API**) to deeply understand the context and extract specific fields: GSTINs, invoice numbers, dates, HSN codes, base amounts, and CGST/SGST/IGST splits with strict JSON schema enforcement.
- **Data Sandbox (`/debug-ocr`)**: A dedicated developer endpoint to view raw extraction data and character counts before LLM structuring.
- **Human-in-the-Loop Review**: A dedicated `Review` UI allows accountants to verify, correct, and ultimately "Lock" parsed data into the permanent ledger.
- **Asynchronous Reparse**: Ability to manually re-trigger the OCR extraction pipeline on a specific invoice if the initial automated scan was inadequate.
- **Excel Data Export (`.xlsx`)**: Instantly download fully-styled, filtered Excel sheets directly from the Dashboard or Invoice Repository using the `openpyxl` engine. Includes custom cell styling and auto-adjusted column widths.

### 2. Tax Calculation & Compliance Engine
- **Automated HSN/SAC Tracking**: Extracts and logs commodity codes for compliance.
- **Inter/Intra-state Tax Logic**: Automatically recognizes whether an invoice requires IGST (inter-state) or a CGST/SGST split (intra-state) based on GSTIN origin logic.
- **Input Tax Credit (ITC) Aggregation**: Separates liability (sales/purchases) and automatically sums available ITC for the current period.
- **Monthly Return Generation (`returns.py`)**: One-click generation of GSTR-like monthly summaries. Aggregates all verified invoices for a specific month/year into a unified tax document.

### 3. Analytics, Reporting & Forecasting
  - **Real-time KPI Dashboard**: Live metrics for Total Invoices, Net GST Payable, Available ITC, and System Extraction Accuracy.
  - **Data Visualization**: Interactive, gradient-filled Area Charts mapping "Sales Tax" vs "Net Payable" over a 6-month historical rolling window (powered by `Recharts`).
  - **Comprehensive Excel Reports (`invoices.py -> /export`)**: 
    - Generates downloadable `.xlsx` files using `openpyxl`.
    - Includes custom cell styling, dynamic column width adjustments, and reflects currently applied UI filters.
    - Export buttons are natively integrated into the "Recently Processed" dashboard and the main "Invoice Repository" page.
  - **Predictive Liability Engine (`model.py`)**:
    - **Primary Engine**: Uses **Facebook Prophet** to forecast the next month's GST liability, natively handling seasonal trends and business cycles.
    - **Secondary Engine**: Includes an automatic fallback to **ARIMA (statsmodels)** or a sophisticated Weighted Moving Average if the Prophet model lacks sufficient historical data.
    - Generates a full confidence interval (Upper bounds/Lower bounds) to aid in financial planning.
  
  ### 4. Admin, Security & Audit Architecture
  - **System Administration Console (`/admin`)**: A globally restricted dashboard (via RBAC) that tracks system-wide KPIs: active users, business tenants, overall processed invoices, and flagged extraction errors.
  - **Immutable Audit Logging (`audit.py`)**: All critical state changes enforce background writes to a MongoDB `AuditLog` collection, preserving strict ledgers with user IDs, IP addresses, entity mutations, and UTC timestamps.
  - **User Management Map**: Enables Admin roles to visualize user registries, active/inactive login states, and securely manage global organizational access.
  - **Role-Based Access Control (RBAC)**: Protects all API endpoints via layered permissions (Roles: `Admin`, `Business Owner`, `Accountant`).
  
  ### 5. Premium UI/UX & Real-time Feedback
  - **Asynchronous Notification Center**: A global, animated dropdown tracking real-time events (e.g., background OCR completion statuses), complete with unread badging, timestamp formatting (`date-fns`), and `markAsRead` capabilities.
  - **Glassmorphism Design**: extensive use of `backdrop-blur`, semi-transparent backgrounds, and subtle gradient borders to create a macOS-like aesthetic.
  - **Micro-animations (`Framer Motion`)**: Staggered list reveals, hover scaling (`scale-105`), and dynamic layout transitions.
  - **Responsive Architecture**: Fully mobile-optimized layouts using Tailwind breakpoints, featuring a custom collapsible sliding sidebar.
  - **Typography Standards**: Enforced tabular numbering (`font-tabular`) across all financial digits for perfect vertical alignment of currency values.

---

## 🛠 Complete Tech Stack

### Frontend Application (`/frontend`)
- **Core Framework:** React 18 (Bootstrapped via Vite)
- **Routing:** React Router DOM v6
- **Styling Architecture:** Tailwind CSS v3.4 + Custom Vanilla CSS Utilities
- **Animation Engine:** Framer Motion v12
- **Iconography:** Lucide React
- **Data Visualization:** Recharts v2.12
- **State & Form Management:** React Context API + React Hook Form v7
- **Network Client:** Axios
- **Notifications:** React Hot Toast

### Backend API Server (`/backend`)
- **Core Framework:** FastAPI (Python 3.11+)
- **Server:** Uvicorn (ASGI)
- **Database:** MongoDB 7.0 
- **ODM (Object Document Mapper):** Beanie (Motor Asyncio)
- **Data Validation & Settings:** Pydantic v2 + Pydantic Settings
- **Authentication:** PyJWT + passlib (bcrypt)
- **File Handling:** python-multipart, aiofiles
- **Report Generation:** openpyxl

### AI & Machine Learning Pipeline
- **Core ML Frameworks:** PyTorch, ONNX Runtime
- **LLM Engine:** Llama-3.3-70b-versatile (via Groq API)
- **OCR Engine:** RapidOCR (onnxruntime) + PyTesseract
- **Document Rendering:** PyMuPDF, Pillow, OpenCV (headless)
- **Forecasting Models:** Facebook Prophet, statsmodels (ARIMA)
- **Data Science Utilities:** Pandas, Scikit-learn, Numpy, Scipy

---

## 🔌 Complete API Ecosystem (v1)

### 1. Authentication (`/api/v1/auth`)
| Method | Route | Description |
|--------|-------|-------------|
| `POST` | `/register` | Register new user |
| `POST` | `/login` | Authenticate and return JWT token |
| `GET`  | `/me` | Get current logged-in user profile |

### 2. Business Profile (`/api/v1/business`)
| Method | Route | Description |
|--------|-------|-------------|
| `GET`  | `/profile` | Get current user's business profile |
| `POST` | `/profile` | Create business profile |
| `PUT`  | `/profile` | Update existing business profile |

### 3. Invoice Management (`/api/v1/invoices`)
| Method | Route | Description |
|--------|-------|-------------|
| `GET`  | `/` | List invoices with pagination, filtering (status/type), and search |
| `POST` | `/upload` | Upload PDF/image; saves file and dispatches async background parsing |
| `GET`  | `/export` | Download filtered invoice data as a formatted Excel (`.xlsx`) sheet |
| `POST` | `/debug-ocr` | Developer capability: Returns raw text generated by the OCR engines pre-LLM |
| `GET`  | `/{id}` | Get detailed, structured invoice data by UUID |
| `PUT`  | `/{id}/review` | Accountant capability: Update & Verify parsed data for final ledger entry |
| `POST` | `/{id}/reparse` | Synchronously re-triggers the ML extraction pipeline |

### 4. Returns & Compliance (`/api/v1/returns`)
| Method | Route | Description |
|--------|-------|-------------|
| `GET`  | `/monthly-summary` | Fetch the aggregated GST summary for a specific month and year |
| `POST` | `/generate` | Aggregate all verified invoices into a new permanent monthly return snapshot |

### 5. Analytics & Reports (`/api/v1/reports` & `/forecast`)
| Method | Route | Description |
|--------|-------|-------------|
| `GET`  | `/reports/tax-summary` | Aggregated, time-series data of tax liabilities for dashboard charts |
| `GET`  | `/reports/invoice-summary` | Status/Categorical metrics (e.g., number of verified vs pending invoices) |
| `GET`  | `/forecast/next-month` | Executes Prophet/ARIMA to predict the next month's tax liability and trends |
| `POST` | `/forecast/train` | Manually retrain the forecasting models against recent historical data |

### 6. Administration (`/api/v1/admin`)
| Method | Route | Description |
|--------|-------|-------------|
| `GET`  | `/stats` | Global system statistics (restricted via RBAC to `admin` roles only) |
| `GET`  | `/users` | Global user registry listing with active/inactive states and role map |
| `GET`  | `/audit-logs` | Immutable security ledger of all critical system mutations and logins |

---

## 📊 Database Schema

The system uses MongoDB with the following collections:

### Users Collection
```json
{
  "_id": "ObjectId",
  "name": "string",
  "email": "string (unique)",
  "password_hash": "string (bcrypt)",
  "role": "enum['admin', 'accountant', 'business_owner']",
  "is_active": "boolean",
  "created_at": "datetime",
  "updated_at": "datetime"
}
```

### BusinessProfiles Collection
```json
{
  "_id": "ObjectId",
  "user_id": "string (FK → Users)",
  "business_name": "string",
  "gstin": "string (15-char GSTIN)",
  "state": "string",
  "state_code": "string (2-digit)",
  "contact_email": "string",
  "contact_phone": "string",
  "address": "string",
  "filing_frequency": "enum['monthly', 'quarterly']",
  "pan": "string",
  "created_at": "datetime",
  "updated_at": "datetime"
}
```

### Invoices Collection
```json
{
  "_id": "ObjectId",
  "user_id": "string (FK → Users)",
  "business_id": "string (FK → BusinessProfiles)",
  "file_path": "string",
  "original_filename": "string",
  "invoice_type": "enum['purchase', 'sale']",
  "invoice_number": "string",
  "invoice_date": "string (DD/MM/YYYY)",
  "supplier_name": "string",
  "supplier_gstin": "string",
  "buyer_name": "string",
  "buyer_gstin": "string",
  "hsn_sac_code": "string",
  "place_of_supply": "string",
  "taxable_amount": "float",
  "cgst": "float",
  "sgst": "float",
  "igst": "float",
  "total_amount": "float",
  "is_interstate": "boolean",
  "parser_confidence": "float (0-1)",
  "parsed_data": "object",
  "corrected_data": "object",
  "line_items": "array",
  "fraud_score": "float",
  "status": "enum['uploaded', 'parsed', 'verified', 'flagged', 'included_in_return']",
  "created_at": "datetime",
  "updated_at": "datetime"
}
```

### MonthlyReturns Collection
```json
{
  "_id": "ObjectId",
  "user_id": "string (FK → Users)",
  "business_id": "string (FK → BusinessProfiles)",
  "month": "int (1-12)",
  "year": "int",
  "total_invoices": "int",
  "total_taxable_value": "float",
  "total_sales_tax": "float",
  "total_purchase_tax": "float",
  "input_tax_credit": "float",
  "net_gst_payable": "float",
  "cgst_payable": "float",
  "sgst_payable": "float",
  "igst_payable": "float",
  "flagged_invoices": "int",
  "status": "enum['draft', 'filed']",
  "generated_at": "datetime"
}
```

### Forecasts Collection
```json
{
  "_id": "ObjectId",
  "business_id": "string (FK → BusinessProfiles)",
  "month": "int",
  "year": "int",
  "predicted_liability": "float",
  "lower_bound": "float (95% CI)",
  "upper_bound": "float (95% CI)",
  "model_type": "string ['prophet', 'arima', 'wma']",
  "trend": "enum['increasing', 'decreasing', 'stable']",
  "explanation": "string",
  "historical_data": "array",
  "forecast_data": "array",
  "created_at": "datetime"
}
```

### AuditLogs Collection
```json
{
  "_id": "ObjectId",
  "user_id": "string",
  "action": "string",
  "entity_type": "string",
  "entity_id": "string",
  "metadata": "object",
  "ip_address": "string",
  "timestamp": "datetime"
}
```

---

## 🤖 ML Pipeline Details

### Invoice Parser Pipeline
1. **File Type Detection**: PDF vs Image
2. **Text Extraction**:
   - PDF: PyMuPDF text extraction (fast if digital text exists)
   - Fallback: Render at 250 DPI + RapidOCR
3. **Field Extraction**: Regex patterns on cleaned text (GSTIN, invoice number, date, amounts, tax components)
4. **LLM Structuring**: Groq Llama-3.3-70b → JSON validation
5. **Confidence Score**: 0.50 + (0.05 × fields_found)

### Fraud Detection
- **Rule-Based Scores** (0-1):
  - Invalid GSTIN: +0.25
  - Duplicate invoice: +0.40
  - High tax ratio: +0.20
  - Other anomalies: +0.10-0.15
- **ML Model**: Isolation Forest trained on synthetic normal records
- **Trigger**: Score ≥ 0.40 or duplicate = **FLAGGED**

### Forecasting
- **Primary Model**: Facebook Prophet (seasonal decomposition)
- **Fallback 1**: ARIMA(1,1,1) via statsmodels
- **Fallback 2**: Weighted Moving Average
- **Output**: Prediction + 95% confidence bounds + trend analysis

---

## 🚀 Deployment Configuration

### Docker Setup
```bash
docker-compose up --build
# Services:
# - Backend: http://localhost:8000
# - Frontend: http://localhost:5173
# - Database: MongoDB Atlas (cloud)
```

### Backend Dockerfile
- Base: Python 3.11 slim
- System deps: tesseract-ocr, poppler-utils
- ENTRYPOINT: uvicorn main:app --host 0.0.0.0 --port 8000
- Volume: `/app/uploads`

### Frontend Dockerfile
- Build: Node 20 Alpine (npm ci, build)
- Runtime: Nginx Alpine
- Reverse proxy: `/api/*` → backend:8000
- SPA routing: all requests → index.html

---

## ⚡ Quick Start (Local Setup)

### Prerequisites
- Python 3.11+
- Node.js 20+
- MongoDB instance (Listen on localhost:27017 or use Atlas)

### 1. Backend Setup
```bash
git clone <repo-url>
cd gst-management-system/backend

# Create and activate a pristine Python virtual environment
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate

# Install all backend dependencies
pip install -r requirements.txt

# Create .env file with your variables (MONGODB_URL, SECRET_KEY, GROQ_API_KEY)
cp .env.example .env

# Run the FastAPI ASGI server
uvicorn main:app --reload --port 8000
```

### 2. Frontend Setup
```bash
cd ../frontend

# Install node dependencies
npm install

# Start the Vite HMR server
npm run dev
# The application UI will be available at http://localhost:5173
```

### 3. Seed Demo Data
To view the powerful dashboard analytics immediately, run the synthetic data script. This will populate MongoDB with 6 months of historical invoices and dummy business accounts to showcase the forecasting engine.
```bash
cd backend
python datasets/seed_data.py
```

### Demo Test Credentials
| Role | Email | Password |
|------|-------|----------|
| Business Owner | `raj@business.demo` | `demo1234` |
| Admin | `admin@gst.demo` | `admin123` |
| Accountant | `priya@accounts.demo` | `demo1234` |

---

## 🔮 Future Roadmap (Phase 2)
- [ ] Direct API integration with approved GST Suvidha Providers (GSPs) for live state filing.
- [ ] Multi-GSTIN corporate support for holding companies with unified dashboards.
- [ ] Automated Ledger Reconciliation directly against GSTR-2B JSON payloads.
- [ ] Bulk ZIP upload processing arrays.
