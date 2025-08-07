# GEP Mass-Create Automation Tool

End-to-end pipeline that converts raw vendor spreadsheets into GEP-ready mass-create workbooks, including data quality checks, enrichment, and batch slicing.  
> **Impact**: what once required ~16 hours of manual copy-paste, validation, and formatting now runs in **≈ 5 minutes**.

---

## Contents
| Section | Description |
|---------|-------------|
| [1. Why this project?](#1-why-this-project) | Background & pain points |
| [2. Core workflow](#2-core-workflow) | Data-flow architecture |
| [3. Feature matrix](#3-feature-matrix) | Skills baked into the pipeline |
| [4. Repository layout](#4-repository-layout) | Where to find what |
| [5. Quick start](#5-quick-start) | Installation and first run |
| [6. Configuration](#6-configuration) | Paths, batch rules, logging |
| [7. Usage patterns](#7-usage-patterns) | Notebook, CLI, scheduled job |
| [8. Performance](#8-performance) | Benchmarks & hardware |
| [9. Troubleshooting](#9-troubleshooting) | Common issues & fixes |
| [10. Roadmap](#10-roadmap) | Planned enhancements |
| [11. Contributing](#11-contributing) | PR guidelines |
| [12. License](#12-license) | MIT |

---

## 1 · Why this project?

Vendor onboarding into **GEP** required analysts to:

1. Copy user-supplied templates into a master workbook  
2. VLOOKUP 30+ reference lists by hand  
3. Split/merge columns to fit GEP’s strict sheet layout  
4. Slice the file into small batches for interface testing

Even small roll-outs meant **days** of repetitive work that was error-prone and impossible to audit.  
This repository automates the entire chain.

---

## 2 · Core workflow

       ┌─────────────┐             ┌────────────┐
       │ User Input  │             │  Master    │
       │  workbook   │             │ reference  │
       │ (6 sheets)  │             │  data      │
       └─────┬───────┘             └────┬───────┘
             │                           │
             ▼                           ▼
    ┌─────────────────┐       ┌──────────────────┐
    │  Cleaning       │       │  Cleaning        │
    │  (strip/fill)   │       │  (standardise)   │
    └────────┬────────┘       └────────┬─────────┘
             │                           │
             ├─────▶ **Mapping & Validation** ◀────┤
             │            (VLOOKUP, logs)          │
             ▼                                     ▼
    ┌─────────────────┐                   optional logs
    │ Transformation  │  invalid rows  ────────────────▶  *_mismatch.csv
    │  (calc fields)  │
    └────────┬────────┘
             │
             ▼
    ┌─────────────────┐
    │   GEP Template  │  Partners / Org_Entities / …
    └────────┬────────┘
             │
    ┌────────▼────────┐
    │  Batch Export   │  1 / 10 / 50 / 100 / 350 rows
    └─────────────────┘

    
---

## 3 · Feature matrix

| Category               | Details |
|------------------------|---------|
| Cleaning               | Empty-row purge · dtype harmonisation · header normalisation |
| Mapping & Validation   | Look-ups against 30+ reference tables (currency, WHT, Incoterms, state, engagement model, buyer user, etc.) with **case-insensitive** and **substring** support |
| Mismatch reporting     | Every failed comparison logged to a dedicated CSV for fast triage |
| Transformation         | Name & address chunking (35 char blocks) · auto-generated IDs (`ClientBankReferenceID`, row keys) · default flags (`isHeadquarter`, `isPrimary`, `isDefault`) |
| Template output        | 20+ sheets exactly matching GEP bulk-load spec |
| Batch slicing          | Configurable sizes; default progression 1 → 10 → 50 → 100 → 350 rows |
| Runtime                | ≈ 5 min for ~10 000 vendors on Intel i7 / 16 GB RAM |

---

## 4 · Repository layout


.
├── src/
│   ├── mass_create_pipeline.ipynb   # step-by-step notebook
│   ├── mass_create_pipeline.py      # command-line entrypoint
│   └── helpers/
│       ├── cleaning.py
│       ├── mapping.py
│       ├── validation.py
│       └── export.py
├── data/
│   ├── input/     # sample anonymised user files
│   ├── master/    # sample reference data
│   └── output_sample/
├── docs/
│   ├── architecture.md
│   └── change_log.md
├── requirements.txt
└── README.md


 
## 5 · Quick Start


# 1. Clone and install
git clone https://github.com/<your-org>/GEP-Mass-Create-Automation-Tool.git
cd GEP-Mass-Create-Automation-Tool
pip install -r requirements.txt

# 2. Point the script to your own paths
vim src/mass_create_pipeline.py   # edit INPUT_DIR, MASTER_DIR, OUTPUT_DIR

# 3. Execute
python src/mass_create_pipeline.py

### Outputs

| File / Folder | Purpose |
|---------------|---------|
| `Mass Create_<timestamp>.xlsx` | Fully populated master workbook in GEP format |
| `Batch_Output/` | Segmented XLSX files for incremental upload |
| `*_mismatch.csv` | Optional QA logs for rows that failed validation |

---

### 6 · Configuration

| Option       | Location                           | Default            | Notes                                             |
|--------------|------------------------------------|--------------------|---------------------------------------------------|
| `INPUT_DIR`  | top of `mass_create_pipeline.py`   | `data/input`       | Folder containing user templates                  |
| `MASTER_DIR` | ”                                  | `data/master`      | Folder with reference sheets                      |
| `OUTPUT_DIR` | ”                                  | `data/output_sample` | Where processed files are written               |
| `BATCH_SIZES`| ”                                  | `[1, 10, 50, 100]` | Subsequent chunks are 350 rows each               |
| Log level    | env var `LOG_LEVEL`                | `INFO`             | Set to `DEBUG` for verbose logging                |

---

### 7 · Usage Patterns

| Mode                    | Command / Action                                      |
|-------------------------|-------------------------------------------------------|
| **One-off (CLI)**       | `python src/mass_create_pipeline.py`                  |
| **Jupyter analysis**    | Run `src/mass_create_pipeline.ipynb` cell-by-cell     |
| **Scheduled (cron/Airflow)** | Call the same script; exit code `0` = success, `1` = fail |

---

### 8 · Performance Benchmarks

| Dataset size | Runtime&nbsp;<sup>†</sup> |
|--------------|---------------------------|
| 1 000 vendors | ~40 s  |
| 10 000 vendors | ~5 min |
| 20 000 vendors | ~11 min |

<sup>†</sup> Intel i7-1165G7, 16 GB RAM. I/O (Excel read/write) dominates; CPU utilisation < 30 %.

---

### 9 · Troubleshooting

| Symptom                       | Possible Cause                       | Fix |
|-------------------------------|--------------------------------------|-----|
| `KeyError '<column>'`         | Missing column in user template      | Ensure template includes all six required sheets |
| Mapping returns blank         | Value absent in reference sheet      | Add mapping to `/data/master/...` or update VLOOKUP list |
| Excel file locked             | Output workbook open in Excel        | Close the file and rerun                           |
| `UnicodeEncodeError` on write | Non-ASCII path characters (Windows)  | Move repo to a plain-ASCII path (e.g. `C:\Projects\gep\`) |

---

### 10 · Roadmap

- Unit-test harness with **pytest** and sample fixtures  
- Parquet / CSV intermediary for sub-second reloads  
- **Docker** image for zero-setup execution  
- Optional **Power BI / Tableau** dashboards on validation logs  

---

### 11 · Contributing

1. **Fork** → create a feature **branch** → open a **PR**.  
2. Follow **black** & **isort** style guides.  
   ```bash
   pip install pre-commit
   pre-commit install

