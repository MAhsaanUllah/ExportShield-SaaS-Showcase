# 🛡️ ExportShield Pro — ZATCA E-Invoicing Compliance Middleware (Phase 1 ✓ offline, Phase 2 sandbox)

![Next.js](https://img.shields.io/badge/Next.js_14-000000?style=flat&logo=nextdotjs&logoColor=white)
![FastAPI](https://img.shields.io/badge/FastAPI-009688?style=flat&logo=fastapi&logoColor=white)
![Supabase](https://img.shields.io/badge/Supabase-3ECF8E?style=flat&logo=supabase&logoColor=white)
![Python](https://img.shields.io/badge/Python_3.11-3776AB?style=flat&logo=python&logoColor=white)
![ZATCA](https://img.shields.io/badge/Compliance-ZATCA_Phase_1_Ready-00C853)

> **ExportShield Pro** is a ZATCA e-invoicing compliance middleware — Phase 1 validated offline, Phase 2 sandbox-tested. It generates ZATCA-compliant invoices in two ways: a **Cloud SaaS** (Next.js + FastAPI + Supabase) and an **Offline Desktop app** (Streamlit, air-gapped).

**At a glance** — 2 products in 1 codebase · 33 backend API endpoints · 20+ frontend pages · 3 subscription tiers with plan enforcement · 9/9 QA tests passing · 123-entry bilingual trade dictionary · ZATCA Phase 1 + Phase 2 groundwork.

---

## 🔗 Live Demo

- **Live app:** [exportshieldpro.online](https://exportshieldpro.online)
- **Demo screenshots:** available below — see the Screenshots section.
- This is a **technical showcase** of a commercial product in pilot/sandbox stage; source is maintained privately.

---

## Problem

Exporters in Pakistan/Saudi Arabia issue invoices that don't meet Saudi tax authority (ZATCA) requirements. Manual conversion of Excel sales sheets into compliant, bilingual Arabic/English PDFs with valid QR codes is error-prone and time-consuming. ExportShield Pro automates this: upload an Excel sheet, auto-map its columns, fix invalid rows, and get a ZIP of ZATCA-compliant PDFs.

---

## Architecture

```
[Cloud SaaS]
Frontend (Next.js 14, localhost:3000) → FastAPI backend (localhost:8000) → Supabase (PostgreSQL/Storage/Auth)
                                          ├─ ZATCA Fatoora Sandbox client (B2B clearance / B2C reporting)
                                          └─ Local XAdES-BES XML signing (crypto_service)

[Offline App]
launch.bat → Streamlit engine (main.py) → Local SQLite/JSON database + PDF/XML files (air-gapped)
```

**Backend entry point:** `backend/app/main.py` — FastAPI app that registers routers for `invoices`, `bulk`, `sales`, `admin`, `dictionary`, `zatca`, `webhooks`, `analytics`, and `auth`. Storage buckets (`invoice-bucket`, `assets`, `proofs`) are auto-created on startup.

---

## Engineering Challenges Solved

- **Bilingual PDF rendering (Arabic RTL)** — ReportLab doesn't shape Arabic; I built a re-shaping/bidi pipeline and patched a regex bug that rendered Arabic inside parentheses as black blocks. (Now emerald-green branded, ZATCA-style.)
- **A single TLV encoder that handles 2 tax regimes** — accepts 15-digit Saudi VAT *and* 7–15-char Pakistani NTNs, so one QR pipeline serves both markets.
- **Encryption that doesn't break analytics** — client-side AES-GCM ciphertext stored as JSONB; the backend can't decrypt, so it skips encrypted rows instead of corrupting revenue totals.
- **Monthly plan-limit reset that survives free-tier Supabase** — `pg_cron` (Pro) *plus* a lazy app-level sync fallback, so limits reset even without the cron extension.
- **Multi-model AI-fix with quota control** — one endpoint that swaps OpenRouter/OpenAI/DeepSeek providers and charges per-tenant AI-fix quotas.
- **10MB upload, 500-row preview caps, per-tenant rate limiting** — defense-in-depth for a multitenant billing product.

---

## Results / Performance (REAL, from code)

| Metric | Value | Source |
|--------|-------|--------|
| Bulk upload file size limit | 10 MB | `bulk.py` `_read_file_safe` |
| Bulk preview row cap | 500 rows | `bulk.py` `/preview` |
| Rate limit | 60 requests/min per tenant (JWT `sub`-keyed) | `rate_limiter.py` |
| Plan limits | Trial 3 / Starter 25 / Growth 1000 / Business ∞ per month | `plan_enforcement.py` |
| Monthly count reset | `pg_cron` (1st of month) + lazy app-level sync | `security.py`, migration `11` |
| AI-fix row batch | Max 20 rows per LLM call | `bulk.py` `/ai-fix` |

*(No accuracy/benchmark/latency numbers are hardcoded in the code, so none are claimed here.)*

---

## Tech Stack

**Backend (from `requirements.txt`):**
- FastAPI 0.111 + Uvicorn
- Supabase 2.4.6 (Postgres, Storage, Auth)
- ReportLab 4.1 + arabic-reshaper + python-bidi (bilingual PDFs)
- lxml (UBL 2.1 XML)
- `qrcode` + Pillow (QR embedding)
- OpenPyXL + pandas (Excel/CSV processing)
- slowapi (rate limiting), python-jose (JWT), cryptography (ECDSA/X.509)
- matplotlib (analytics charts)
- deep-translator + OpenRouter API (translation)

**Frontend (from `package.json`):**
- Next.js 14.2 (App Router), React 18, TypeScript, Tailwind CSS + shadcn/ui
- Supabase SSR client, react-hook-form + zod
- recharts, framer-motion, xlsx (client-side parsing)

---

## Key Features

- **ZATCA-compliant invoice engine** — UBL 2.1 XML generation, bilingual (Arabic/English) PDF rendering with company logo + bank details, and a TLV (Tag 1–5) QR code. Phase 1 TLV QR codes verified on the official ZATCA-1 app (offline). Phase 2 QR validated in FATOORA sandbox. (`xml_generator.py`, `pdf_renderer.py`, `tlv_encoder.py`)
- **Bulk upload wizard** — 3-step flow (preview → audit → generate ZIP) with a SmartMap column auto-detector that matches messy Excel headers to schema fields with confidence scores. (`bulk.py`, `bulk-upload/page.tsx`)
- **Inline audit grid with auto-fix + AI-fix** — every row is validated (dates, VAT format, quantities, math); a heuristic `/autofix` corrects common errors, and an LLM-based `/ai-fix` (OpenRouter — Gemini/OpenAI/DeepSeek) corrects complex ones with quota tracking. (`bulk.py`)
- **ZATCA Phase 2 groundwork** — ECDSA (secp256k1) key generation, CSR creation with Microsoft template OIDs, a sandbox handshake endpoint, XAdES-BES XML signing, and a Fatoora Sandbox client for B2B clearance / B2C reporting with local simulation fallback. (`crypto_service.py`, `zatca_client.py`, `zatca.py`)
- **Subscription & plan enforcement** — 3 tiers (Starter/Growth/Business), per-month invoice limits, bulk gating (Growth+), payment-proof upload + admin approval desk, and admin email notifications. (`plan_enforcement.py`, `admin.py`, `subscriptions.py`)
- **Admin Control Center** — dark-themed `/control-center` portal for user management (create users, reset passwords, change emails, toggle roles via Supabase Admin API), subscription approvals, and AI API key management. (`admin.py`, `control-center/*`)
- **Analytics & reports** — sales aggregation (period filters: month/quarter/year/all), downloadable PDF report with charts, and a compliance audit dashboard. (`sales.py`, `analytics.py`)
- **Offline desktop app** — air-gapped Streamlit engine with node-locked licensing (WMIC UUID + SHA-256 salt), local SQLite product dictionary with auto-translation, inline validation grid, and analytics dashboards. (`reference_logic/ExportShield_FINAL_PRO/_internal`)

---

## Security & Controls

- **Multi-tenancy / RLS:** Supabase Row-Level Security policies isolate data per tenant (`auth.uid()`) across profiles, invoices, products, customers, bulk jobs, and subscriptions (migration `21`).
- **Auth:** JWT validation (HS256, local-first) with fallback to the Supabase auth API; `get_current_admin` guards admin routes.
- **Encryption:** Client-side AES-GCM (`WebCrypto`) encrypts amounts and line-item prices before they hit the database; stored as JSONB ciphertext payloads. Backend treats encrypted rows safely (doesn't try to sum them).
- **Rate limiting:** slowapi keyed by JWT `sub` claim (not IP) — 60 req/min per tenant on generate endpoints.
- **Log scrubbing:** global exception handler redacts `total_with_vat`, `vat_amount`, `buyer_name`, and encrypted payloads from tracebacks.

---

## Status

- **Cloud SaaS:** built and **live at [exportshieldpro.online](https://exportshieldpro.online)** — deployed and functional; ongoing development.
- **Offline desktop app:** built, packaged (launcher scripts + `_internal` bundle), works air-gapped.
- **ZATCA Phase 2:** **partial / in progress** — signing + sandbox client exist, but full production certification flow requires a real ZATCA-issued certificate.

---

## Getting Started

### Backend

```bash
cd backend
pip install -r requirements.txt
# create .env from .env.example (SUPABASE_URL, SERVICE_ROLE_KEY, JWT_SECRET)
uvicorn app.main:app --reload --port 8000
```

### Frontend

```bash
cd frontend
npm install
npm run dev   # → http://localhost:3000
```

### Offline app (optional)

```bash
cd reference_logic/ExportShield_FINAL_PRO/_internal
pip install -r requirements.txt
python main.py   # Streamlit app; node-locked license required
```

---

## Screenshots

> ⚠️ **Action required before sharing this repo:** add real screenshots here — this is the #1 trust signal for a recruiter. Grab them from your local run:

- **Cloud SaaS (run `npm run dev` + backend):** `/home` (landing), `/dashboard`, `/generate-invoice` (fill the form), `/bulk-upload` (upload `sample_bulk_invoices.csv` from the repo root), `/sales`.
- **Offline app:** `reference_logic/ExportShield_FINAL_PRO/_internal/` → run `main.py`, capture the Dashboard, Reports & Analytics, and Company Settings pages.

| Landing Page | Cloud Dashboard | Batch Processing | Generate Invoice |
|---|---|---|---|
| [Add screenshot here] | [Add screenshot here] | [Add screenshot here] | [Add screenshot here] |

| Offline Dashboard | Analytical Intelligence | Company Settings |
|---|---|---|
| [Add screenshot here] | [Add screenshot here] | [Add screenshot here] |

---

## Repo Layout (what a reviewer will open first)

```
frontend/        Next.js 14 (App Router, 20+ pages: dashboard, generate, bulk, history, sales…)
backend/         FastAPI (33 endpoints, UBL XML, PDF, TLV QR, ECDSA, sandbox client)
  └─ migrations/ 24 SQL migrations (RLS, plan reset, storage buckets, buyer details)
reference_logic/ Offline Streamlit app + legacy suites
  └─ ExportShield_FINAL_PRO/_internal/  Node-locked, air-gapped desktop engine
showcase_readme.md                     ← you are here
```

---

## 👨‍💻 Architected By

**Muhammad Ahsaan Ullah** — AI Automation & Full-Stack Engineer

[![LinkedIn](https://img.shields.io/badge/LinkedIn-mahsaanullah-0A66C2?logo=linkedin)](https://linkedin.com/in/mahsaanullah)
[![GitHub](https://img.shields.io/badge/GitHub-MAhsaanUllah-181717?logo=github)](https://github.com/MAhsaanUllah)

<div align="center">
<b>ExportShield Pro — Making ZATCA compliance accessible and secure.</b>
</div>
