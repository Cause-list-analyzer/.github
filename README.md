<div align="center">

# ⚖️ High Court Cause List — PDF Parser & Search Portal

**A full-stack web application that parses Allahabad High Court daily cause list PDFs,
extracts structured case data, and provides a powerful search, filter, and Excel export interface.**

[![Python](https://img.shields.io/badge/Python-3.11+-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://python.org)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.109-009688?style=for-the-badge&logo=fastapi&logoColor=white)](https://fastapi.tiangolo.com)
[![React](https://img.shields.io/badge/React-18-61DAFB?style=for-the-badge&logo=react&logoColor=black)](https://react.dev)
[![SQLite](https://img.shields.io/badge/SQLite-3-003B57?style=for-the-badge&logo=sqlite&logoColor=white)](https://sqlite.org)
[![Docker](https://img.shields.io/badge/Docker-Ready-2496ED?style=for-the-badge&logo=docker&logoColor=white)](https://docker.com)

---

[Features](#-features) •
[Architecture](#-architecture) •
[Quick Start](#-quick-start) •
[API Reference](#-api-reference) •
[PDF Parsing Logic](#-pdf-parsing-logic) •
[Excel Export](#-excel-export) •
[Tech Stack](#-tech-stack)

</div>

---

## Screenshots

<p align="center">
  <img src="docs/screenshots/dashboard.png" alt="Dashboard" width="48%" />
  <img src="docs/screenshots/upload.png" alt="Upload" width="48%" />
</p>
<p align="center"><b>Dashboard</b> — Analytics & Overview &nbsp;&nbsp;|&nbsp;&nbsp; <b>Upload</b> — Drag & Drop PDF</p>

<p align="center">
  <img src="docs/screenshots/search.png" alt="Search" width="98%" />
</p>
<p align="center"><b>Search</b> — Advanced Filtering & Results</p>

---

## Features

### 📄 PDF Parsing Engine
- **Context-aware parsing** of 2500+ page cause list PDFs
- **X-coordinate column splitting** — separates party names (left) from counsel names (right) using word-level coordinates
- **Court context propagation** — hearing date, judges, list type automatically assigned to every case
- **Multi-court support** — handles Court No-1 through Court No-42+ in a single PDF
- **Connected case linking** — detects "30.1 With" patterns and links companion cases
- **Criminal court specifics** — in-jail dates, crime numbers, police stations, GOVAD/CRLA case types

### Search & Filter
- **Full-text search** across case numbers, party names, counsel, and districts
- **Advanced filters** — by court number, list type, sub-type, case type, district, hearing date
- **Special filters** — in-jail cases only, SC Expedited only, flag-based filtering
- **Judge & counsel search** — find cases by specific judge or counsel name
- **Pagination** with configurable page size and sort direction

### Excel Export
- **4-sheet workbook** — Fresh List, Daily Cause List, Misc Applications, All Cases
- **Formatted output** — blue headers, alternating row colors, frozen top row, auto-fit widths
- **Self-verification** — automated checklist validates data quality after export
- **Clean data** — no serial numbers in petitioner cells, no mixed-up columns

### Dashboard
- Overview statistics (total cases, list types, in-custody count, SC expedited)
- Cases by list type & court number charts
- Top districts breakdown
- Recent uploads tracking

---

## Architecture

> Detailed architecture diagrams are available at [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md).

### High-Level Overview

```text
┌──────────────────────────────────────────────────────────────┐
│                     CLIENT (Browser)                         │
│  React 18 + TailwindCSS + TanStack Query + Zustand           │
│  ┌────────┐  ┌────────┐  ┌───────────┐  ┌──────────────┐     │
│  │ Upload │  │ Search │  │ Dashboard │  │ Case Detail  │     │
│  └────────┘  └────────┘  └───────────┘  └──────────────┘     │
└──────────────────────────┬───────────────────────────────────┘
                           │ HTTP / REST API
┌──────────────────────────┴───────────────────────────────────┐
│                  BACKEND (FastAPI)  :8000                    │
│  ┌──────────────────────────────────────────────────────┐    │
│  │  API Layer (Routers)                                 │    │
│  │  /api/upload  /api/search  /api/stats  /api/export   │    │
│  └──────────────────────┬───────────────────────────────┘    │
│  ┌──────────────────────┴───────────────────────────────┐    │
│  │  Business Logic                                      │    │
│  │  ┌─────────────┐  ┌──────────────┐  ┌─────────────┐  │    │
│  │  │ PDF Parser  │  │ Column Split │  │ Excel Export│  │    │
│  │  │  Pipeline   │  │ (x-coords)   │  │ (openpyxl)  │  │    │
│  │  └─────────────┘  └──────────────┘  └─────────────┘  │    │
│  └──────────────────────┬───────────────────────────────┘    │
│  ┌──────────────────────┴───────────────────────────────┐    │
│  │  Data Layer (SQLAlchemy 2.0 Async)                   │    │
│  │  Models: PdfUpload, Case                             │    │
│  └──────────────────────-┬──────────────────────────────┘    │
└──────────────────────────┴───────────────────────────────────┘
                           │
              ┌────────────┴────────────┐
              │   SQLite (local dev)    │
              │   PostgreSQL (prod)     │
              └─────────────────────────┘
```
---

## Quick Start

### Prerequisites

- **Python 3.11+**
- **Node.js 18+** and **npm 9+**
- **Git**

### 1. Clone & Install Backend

```bash
cd mini_project/backend
python -m venv venv
venv\Scripts\activate    # Windows
pip install -r requirements.txt
```

### 2. Install & Build Frontend

```bash
cd mini_project/frontend
npm install
npx vite build           # Builds to backend/static/
```

### 3. Start the Application

```bash
cd mini_project/backend
python -m uvicorn main:app --reload --host 0.0.0.0 --port 8000
```

### 4. Open in Browser

```bash
http://localhost:8000
```

> **Note:** The frontend is served as static files from the FastAPI backend.
> No separate frontend server is needed.

### Docker (Production)

```bash
docker compose up -d
# Access at http://localhost:80
```

---

## API Reference

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/api/upload` | Upload a cause list PDF for parsing |
| `GET` | `/api/uploads` | List all uploaded files with status |
| `GET` | `/api/uploads/{id}/status` | Get parsing progress for an upload |
| `GET` | `/api/uploads/{id}/excel` | Download the generated Excel file |
| `DELETE` | `/api/uploads/{id}` | Delete upload and its parsed cases |
| `POST` | `/api/search` | Search cases with filters (JSON body) |
| `GET` | `/api/filters/options` | Get available filter dropdown values |
| `GET` | `/api/stats` | Dashboard statistics |
| `GET` | `/api/health` | Health check |

### Search Request Body Example

```json
{
  "query": "SHUKLA",
  "court_number": "Court No-1",
  "list_type": "DAILY CAUSE LIST",
  "district": "ALLAHABAD",
  "in_jail": true,
  "page": 1,
  "page_size": 50,
  "sort_by": "serial_number",
  "sort_dir": "asc"
}
```

Full API documentation with interactive testing is available at:
**`http://localhost:8000/docs`** (Swagger UI)

---

## PDF Parsing Logic

### The Parsing Pipeline

The PDF parser processes Allahabad High Court cause list PDFs with a **5-stage pipeline**:

```
PDF File
    │
    ▼
┌──────────────────┐
│ 1. PAGE ITERATOR │  Read each page with pdfplumber
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│ 2. CONTEXT       │  Detect court headers, judge names,
│    DETECTOR      │  hearing date/time, list type, sub-type,
│                  │  SC Expedited markers
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│ 3. CASE BLOCK    │  Split page text into individual case
│    SPLITTER      │  blocks using serial number regex
│                  │  (e.g., "1 WRIC/...", "30.1 With")
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│ 4. COLUMN        │  For each case block, extract word-level
│    SPLITTER      │  positions via pdfplumber.
│                  │  x0 < 300px → Left (party names)
│                  │  x0 ≥ 300px → Right (counsel names)
│                  │  Split each column at "VS" line.
└────────┬─────────┘
         │
         ▼
┌──────────────────────┐
│ 5. FIELD EXTRACTOR   │  Extract structured fields:
│                      │  serial, case_number, flags,
│                      │  district, parties, counsel,
│                      │  notice_no, tc_no, in_jail,
│                      │  crime_no, police_station,
│                      │  misc app fields, etc.
│                      │  + Assign context (court, date,
│                      │    judges, list_type) to every case
└──────────────────────┘
         │
         ▼
    Structured Records → Database + Excel
```

### Two-Column Layout

The PDF uses a **physical two-column layout** where party names and counsel names
are on the same horizontal line but at different x-coordinates:

```
 x=0                    x=300                          x=600
  │                       │                              │
  │  LOK NATH SHUKLA      │  KRISHNA MOHAN ASTHANA       │
  │                       │  SHANU                       │
  │        VS             │        VS                    │
  │  STATE OF U.P.        │  C.S.C.                      │
  │  AND 5 OTHERS         │                              │
  │                       │                              │
  ├── LEFT COLUMN ────────┤── RIGHT COLUMN ──────────────┤
  │  (Party Names)        │  (Counsel Names)             │
```

### Context Propagation

Court headers contain metadata that applies to **all cases** in that court section:

```
Court No-1
HON'BLE JUSTICE AJIT KUMAR         ← bench_judges[]
HON'BLE JUSTICE INDRAJEET SHUKLA
31-03-2026 At 10:30 AM             ← hearing_date, hearing_time
```

These values are stored in a `ParserContext` object and assigned to every case
record parsed within that court section. When a new `Court No-X` header is
detected, the context resets.

---

## Excel Export

After parsing, an Excel workbook is automatically generated with **4 sheets**:

| Sheet | Contents | Key Columns |
|-------|----------|-------------|
| **Fresh List** | Cases from `FRESH LIST` sections | Sr.No, Case No, District, Parties, Counsel, Flags |
| **Daily Cause List** | Cases from `DAILY CAUSE LIST` sections | + In Jail Since, Crime No, Police Station, SC Expedited |
| **Misc Applications** | Cases from `FRESH MISC. APPLICATION` | + App No, App Type, Original Case, Applied By |
| **All Cases** | Combined — all rows with List Type column | All columns union |

### Formatting
- **Blue headers** (#0284C7) with white bold text
- **Alternating rows** — white / light blue (#E0F2FE)
- **Frozen top row** for easy scrolling
- **Auto-fitted column widths**

### Self-Verification Checklist
The export runs automated quality checks:
- No petitioner cell starts with a number (serial number leak)
- No petitioner cell contains `/` (case number leak)
- No petitioner cell contains `VS`
- No serial number is `0`
- No hearing date is `None`

---

## Tech Stack

### Backend

| Technology | Purpose |
|-----------|---------|
| **Python 3.11+** | Core language |
| **FastAPI** | Async REST API framework |
| **SQLAlchemy 2.0** | ORM with async support |
| **pdfplumber** | PDF text + coordinate extraction |
| **openpyxl** | Excel file generation |
| **Pydantic v2** | Request/response validation |
| **SQLite** (dev) / **PostgreSQL** (prod) | Database |
| **uvicorn** | ASGI server |

### Frontend

| Technology | Purpose |
|-----------|---------|
| **React 18** | UI framework |
| **Vite** | Build tool & dev server |
| **TailwindCSS** | Utility-first CSS |
| **TanStack Query** | Server-state caching |
| **TanStack Table** | Data table with sorting/filtering |
| **Zustand** | Global state management |
| **Recharts** | Dashboard charts |
| **Lucide React** | Icon library |
| **Axios** | HTTP client |

### Infrastructure

| Technology | Purpose |
|-----------|---------|
| **Docker Compose** | Multi-container orchestration |
| **Nginx** | Reverse proxy (production) |
| **Redis** | Task queue backend (production) |
| **Celery** | Distributed task processing (production) |

---

## 📁 Project Structure

```
mini_project/
├── backend/
│   ├── main.py                 # FastAPI app entry point + static file serving
│   ├── config.py               # Configuration (DB URLs, upload dir)
│   ├── database.py             # SQLAlchemy engine & session setup
│   ├── models.py               # ORM models (PdfUpload, Case)
│   ├── schemas.py              # Pydantic validation schemas
│   ├── parsers/
│   │   ├── patterns.py         # All regex patterns (court, case, fields)
│   │   ├── pdf_parser.py       # Main parsing orchestrator
│   │   ├── case_extractor.py   # Case block boundary detection
│   │   └── field_extractor.py  # Individual field extraction
│   ├── routers/
│   │   ├── upload.py           # Upload, parse, Excel generation endpoints
│   │   ├── search.py           # Search & filter endpoint
│   │   ├── stats.py            # Dashboard statistics endpoint
│   │   └── export.py           # Data export endpoint
│   ├── utils/
│   │   ├── column_splitter.py  # X-coordinate column splitting
│   │   ├── excel_export.py     # 4-sheet Excel workbook generator
│   │   └── text_cleaner.py     # Text normalization utilities
│   ├── uploads/                # Uploaded PDFs & generated Excel files
│   ├── static/                 # Built frontend (served by FastAPI)
│   ├── requirements.txt
│   └── Dockerfile
├── frontend/
│   ├── src/
│   │   ├── App.jsx             # Root component with routing
│   │   ├── main.jsx            # React entry point
│   │   ├── index.css           # TailwindCSS + custom design system
│   │   ├── pages/
│   │   │   ├── Upload.jsx      # PDF upload with drag-and-drop
│   │   │   ├── Search.jsx      # Advanced search with filters
│   │   │   ├── Dashboard.jsx   # Analytics dashboard
│   │   │   └── CaseDetail.jsx  # Individual case view
│   │   ├── api/
│   │   │   └── index.js        # Axios API client with interceptors
│   │   ├── store/              # Zustand global state
│   │   └── utils/              # Helper utilities
│   ├── vite.config.js
│   ├── package.json
│   └── Dockerfile
├── docs/
│   ├── ARCHITECTURE.md         # Detailed architecture & design diagrams
│   └── screenshots/            # Application screenshots
├── docker-compose.yml          # Multi-service orchestration
├── nginx.conf                  # Reverse proxy configuration
└── README.md                   # ← You are here
```

---

## Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `DATABASE_URL` | `sqlite+aiosqlite:///courtdb.sqlite3` | Async DB connection string |
| `SYNC_DATABASE_URL` | `sqlite:///courtdb.sqlite3` | Sync DB connection string (for background threads) |
| `UPLOAD_DIR` | `./uploads` | Directory for PDF uploads & Excel output |
| `MAX_UPLOAD_SIZE_MB` | `500` | Maximum upload file size |
| `USE_CELERY` | `""` (disabled) | Enable Celery task queue (production) |
| `REDIS_URL` | `""` | Redis connection URL (production) |

---

## Testing

### Verify Backend

```bash
# Health check
curl http://localhost:8000/api/health
# → {"status":"ok"}

# Swagger UI
open http://localhost:8000/docs
```

### Upload & Parse a PDF

```bash
curl -X POST http://localhost:8000/api/upload \
  -F "file=@cause_list.pdf"
```

### Search Cases

```bash
curl -X POST http://localhost:8000/api/search \
  -H "Content-Type: application/json" \
  -d '{"query": "STATE OF UP", "page": 1}'
```

---

## License

This project is developed for academic purposes as part of a mini project.

---

<div align="center">

**Built with ❤️ for the legal community**

*Parsing justice, one PDF at a time.*

</div>

