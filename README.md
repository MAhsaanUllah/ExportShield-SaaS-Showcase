# 🛡️ ExportShield Pro — AI-Powered ZATCA Phase 1 Compliance Engine

![Status](https://img.shields.io/badge/Status-Gold_Master_v1.0.4-brightgreen)
![Platform](https://img.shields.io/badge/Platform-Windows_Desktop-0078D6?logo=windows)
![Python](https://img.shields.io/badge/Python-3.9+-3776AB?logo=python)
![ZATCA](https://img.shields.io/badge/Compliance-ZATCA_Phase_1_Ready-00C853)
![License](https://img.shields.io/badge/License-Commercial_Node--Locked-red)

> **ExportShield Pro** is a fully offline, AI-powered desktop application that automates ZATCA Phase 1 e-invoicing compliance for Pakistani exporters dealing with Saudi Arabia. Upload an Excel file → AI validates & translates → get bilingual Arabic/English PDFs with cryptographic TLV QR codes — entirely offline, no cloud dependency.

> **⚠️ NOTE:** This repository is a **Technical Showcase** for portfolio purposes. Source code is maintained in a private repository as this is a shipped commercial product (Gold Master v1.0.4).

---

## 🎯 The Problem

Saudi Arabia's ZATCA mandate requires all VAT-registered businesses to generate cryptographically compliant e-invoices with:
- Bilingual Arabic/English content
- TLV-encoded QR codes with 5 mandatory ZATCA tags
- Strict VAT calculation and formatting rules

**Enterprise ERPs (SAP, Oracle) cost thousands of dollars.** Pakistani SME exporters had no affordable, offline solution — until ExportShield.

---

## ✨ What Makes It Different

| Feature | ExportShield Pro | Cloud SaaS Alternatives |
|---|---|---|
| Internet required | ❌ 100% Offline | ✅ Always online |
| Data privacy | ✅ Never leaves machine | ❌ Uploaded to cloud |
| AI extraction | ❌ Phase 2 | ❌ Manual entry |
| Arabic translation | ✅ Dictionary-based (offline) | ❌ Manual or paid API |
| Packaging | ✅ Single .exe, no install | ❌ Complex setup |
| Price | ✅ One-time node-locked | ❌ Monthly subscription |
| ZATCA TLV QR | ✅ Cryptographically correct | ✅ Varies |

---



## 🔐 ZATCA TLV QR Engine

Custom cryptographic implementation — **not a library wrapper:**

```python
# 5 Mandatory ZATCA Tags encoded in TLV binary format
Tag 1: Seller Name (bilingual)
Tag 2: VAT Registration Number (15 digits)
Tag 3: Invoice Timestamp (ISO 8601)
Tag 4: Invoice Total (including VAT)
Tag 5: VAT Amount

→ Binary TLV → Base64 encode → QR Code PNG
→ Embedded in bilingual A4 PDF via ReportLab
```

**AI Compliance Audit** validates every row before generation:
- Missing VAT number → flagged
- Date format errors → flagged  
- Math mismatch (line items vs total) → flagged
- Generation gated at **100% data health**

---

## 🏗️ System Architecture

```
┌─────────────────────────────────────────────────────────┐
│              USER (Windows Desktop)                      │
│         Uploads Excel / CSV / Invoice Image              │
└──────────────────────┬──────────────────────────────────┘
                       │
┌──────────────────────▼──────────────────────────────────┐
│           ExportShield Pro (.exe — 329 MB)               │
│                                                          │
│  ┌─────────────┐  ┌──────────────┐  ┌───────────────┐  │
│  │  Streamlit  │  │  Data        │  │  AI Module    │  │
│  │  UI Engine  │  │  Validator   │  │  Gemini Vision│  │
│  │             │  │  + Cleaner   │  │  + Translator │  │
│  └─────────────┘  └──────────────┘  └───────────────┘  │
│                                                          │
│  ┌─────────────┐  ┌──────────────┐  ┌───────────────┐  │
│  │  ZATCA TLV  │  │  ReportLab   │  │  Node-Lock    │  │
│  │  QR Engine  │  │  PDF Engine  │  │  License      │  │
│  │  (custom)   │  │  (bilingual) │  │  (SHA-256)    │  │
│  └─────────────┘  └──────────────┘  └───────────────┘  │
│                                                          │
│  ┌──────────────────────────────────────────────────┐   │
│  │         SQLite (Embedded — fully offline)         │   │
│  │  Users · Invoices · Product Dictionary · History  │   │
│  └──────────────────────────────────────────────────┘   │
└──────────────────────┬──────────────────────────────────┘
                       │
┌──────────────────────▼──────────────────────────────────┐
│              OUTPUT (Local Machine)                      │
│   Bilingual A4 PDFs + TLV QR Codes → ZIP Download       │
└─────────────────────────────────────────────────────────┘
```

---

## ✨ Core Features

### 1. 📊 End-to-End Invoice Pipeline
- Upload Excel/CSV → auto-detect columns (Invoice No, Date, Customer, VAT ID, Items, Qty, Price)
- AI compliance audit with row-level validation
- Batch generate bilingual PDFs → packaged into ZIP

### 2. 🔐 Cryptographic TLV QR Codes
- Custom ZATCA-spec TLV binary encoder
- 5 mandatory tags: Seller, VAT No, Timestamp, Total, VAT Amount
- Base64 encoded → QR PNG → embedded in PDF

### 3. 🌐 Arabic Translation Engine
- Built-in SQLite product dictionary (19 pre-seeded Pakistan-KSA trade goods)
- `arabic_reshaper` + `python-bidi` for correct RTL Arabic ligature rendering
- CSV/JSON/Excel/TXT bulk dictionary import supported
- UPSERT on save — factory defaults never deleted

### 5. 📄 Bilingual PDF Generation
- A4 format, Arabic (RTL) + English side-by-side
- Multi-page table support for long invoices
- `arabic_reshaper` + `python-bidi` for correct Arabic ligatures
- Company logo, IBAN, bank details auto-filled from settings

### 6. 🔒 Node-Locked Licensing
- Hardware fingerprint via Windows UUID (WMIC + MAC fallback)
- SHA-256 license hash with proprietary salt
- Admin keygen tool (not shipped to users)
- Subscription tier tracking with monthly usage counters

### 7. 📈 Analytics Dashboard
- KPIs: batch cycles, invoice count, gross SAR value
- Daily activity charts (Altair)
- Hour × Day heatmap from usage history

### 8. 🏢 Company Settings
- Configurable company profile (EN + Arabic name, VAT number, address, IBAN)
- Logo upload for PDF branding
- Factory reset with user data preservation option

---

## 🛠️ Full Tech Stack

| Component | Technology |
|---|---|
| UI Framework | Streamlit |
| PDF Engine | ReportLab (A4 bilingual) |
| Arabic Rendering | `arabic_reshaper` + `python-bidi` |
| QR Code | `qrcode` (custom TLV encoder) |
| Database | SQLite3 (embedded, offline) |
| Data Processing | Pandas, openpyxl, xlrd |
| Security | bcrypt, SHA-256 node-lock |
| Charts | Altair |
| Packaging | PyInstaller (single .exe, 329 MB) |
| Language | Python 3.9+ |
| OS Target | Windows 10/11 |

---

## 📦 Delivery Package (v1.0.4)

```
ExportShield_v1.0.4_Delivery/
├── ExportShield_Pro.exe        # 329 MB standalone Windows executable
├── SETUP_GUIDE.txt             # Bilingual EN + Roman Urdu installation guide
└── assets/                     # App assets
```

No Python installation required. No internet connection required. Double-click and run.

---

## 🗺️ Roadmap

**Phase 1 — Complete ✅**
- [x] ZATCA TLV QR code generation (5 mandatory tags)
- [x] Bilingual A4 PDF invoice generation
- [x] AI invoice image extraction (Gemini Vision)
- [x] Bulk Arabic translation with retry logic
- [x] AI compliance audit (row-level validation)
- [x] Node-locked licensing system
- [x] Analytics dashboard with heatmap
- [x] PyInstaller .exe packaging (Gold Master v1.0.4)
- [x] Bilingual setup guide (EN + Roman Urdu)

**Phase 2 — Planned 🔜**
- [ ] AI Invoice Image Extraction (Gemini Vision OCR)
- [ ] Bulk Arabic Translation via Gemini API
- [ ] REST API integration with FATOORA portal
- [ ] Cryptographic Stamp (CSID) generation
- [ ] JSON/XML (UBL 2.1) payload construction
- [ ] Streamlit Cloud demo mode (no license gate)
- [ ] Multi-user auth layer activation

---

## 🔒 Security & Privacy

> **Note:** This is a closed-source commercial product. Source code is maintained in a private repository.

- All data processing happens **100% locally** — no data ever leaves the machine
- Node-locked license tied to hardware UUID — non-transferable
- bcrypt password hashing for user auth module
- SQLite database stored in `~/ExportShield_Data/` — user-controlled

---

## 👨‍💻 Built By

**Muhammad Ahsaan Ullah**  
AI Automation & Full-Stack Engineer | Building production systems for Pakistani businesses

[![LinkedIn](https://img.shields.io/badge/LinkedIn-mahsaanullah-0A66C2?logo=linkedin)](https://linkedin.com/in/mahsaanullah)
[![GitHub](https://img.shields.io/badge/GitHub-MAhsaanUllah-181717?logo=github)](https://github.com/MAhsaanUllah)

---

<div align="center">

**ExportShield Pro — Making ZATCA compliance accessible for every Pakistani exporter. 🇵🇰**

</div>