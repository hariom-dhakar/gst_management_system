# ML Pipeline Documentation

## 1. Invoice Parser

**File:** `backend/app/ml/invoice_parser/parser.py`

### Pipeline
1. Receive file path (PDF or image)
2. **Stage 1: Digital Text Extraction** - Attempt PyMuPDF text extraction (fast, 100% accurate if digital text exists)
3. **Stage 2: OCR Fallback** - If no digital text, render PDF to 250 DPI image and use RapidOCR (ONNX Runtime)
4. **Stage 3: LLM Structuring** - Pass raw text to Groq Llama-3.3-70b for context-aware field extraction with JSON schema
5. Apply regex rules to extract additional fields if needed
6. Return JSON with confidence score

### Fields Extracted
Primary extraction via LLM (Groq Llama-3.3-70b) with regex fallbacks:

| Field | Extraction Method |
|-------|-------------------|
| Invoice Number | LLM + Regex: `INVOICE NO[:\s]*([\w\-/]+)` |
| Date | LLM + Regex: `\d{1,2}[/-]\d{1,2}[/-]\d{2,4}` |
| GSTIN | LLM + Regex: 15-char GSTIN pattern |
| Amounts | LLM + Regex: `RS\.?\s*([\d,]+\.?\d*)` |
| Tax (CGST/SGST/IGST) | LLM + labeled amounts parsing |
| HSN/SAC | LLM + Regex: `HSN[:\s]*(\d{4,8})` |
| Supplier/Buyer Details | LLM context understanding |

### Confidence Score
Calculated as `0.50 + 0.05 * fields_found` — higher when more fields are extracted successfully. LLM provides structured output with validation.

### Error Handling
If OCR or LLM fails, the system logs the error and marks the invoice as "parsed" with available data. Users can manually review and correct fields through the UI.

---

## 2. Fraud Detection

**File:** `backend/app/ml/fraud_detection/model.py`

### Approach: Hybrid (Rules + ML)

#### Rule-Based Checks
| Rule | Score Impact |
|------|-------------|
| Invalid GSTIN format | +0.25 |
| Duplicate invoice number | +0.40 |
| Tax ratio > 50% | +0.20 |
| Zero tax on high-value invoice | +0.15 |
| Round number anomaly (>5L, divisible by 1L) | +0.10 |
| Tax type mismatch (inter-state but no IGST) | +0.20 |

#### ML-Based (Isolation Forest)
- Trained at startup on 500 synthetic "normal" invoice records
- Features: total_amount, taxable_amount, tax_ratio, cgst, sgst, igst
- If ML anomaly score > 0.5: adds +0.30 and flags `ML_ANOMALY_DETECTED`

**Threshold:** Score ≥ 0.40 OR `DUPLICATE_INVOICE_NUMBER` → flagged as fraudulent

---

## 3. GST Forecasting

**File:** `backend/app/ml/forecasting/model.py`

### Prophet (Primary)
- Trained on `monthly_returns` collection (monthly GST payables)
- Config: additive seasonality, low changepoint sensitivity
- Outputs: predicted value + 95% confidence interval + full forecast series

### ARIMA (Fallback)
- Used when Prophet unavailable
- Order: (1, 1, 1)
- Provides prediction + simple confidence interval

### Weighted Moving Average (Last Resort)
- Simple weighted average of recent historical GST liability values
- Used when neither Prophet nor ARIMA is available
- Provides basic prediction without confidence intervals

### Trend Detection
Compares last 3 months average vs all prior months:
- `recent > prior * 1.1` → **increasing**
- `recent < prior * 0.9` → **decreasing**
- Otherwise → **stable**
