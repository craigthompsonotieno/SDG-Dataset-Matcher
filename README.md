# SDG Dataset Matcher

Automated pipeline that matches UN-DESA datasets to official SDMX SDG indicators using fuzzy matching — eliminating 40+ hours of manual reconciliation work.

---

## The Problem

UN-DESA maintains a catalog of 498 SDG datasets but lacks structured links to official SDMX indicator identifiers. Building those links manually would require 40+ hours of work and is prone to inconsistency. This project automates the process using algorithmic similarity scoring.

---

## Key Results

| Metric | Value |
|---|---|
| Datasets matched | 128 (75% of processed SDGs) |
| High-confidence matches (score > 0.7) | 63% — lower manual review burden |
| Data rows retrieved | 217,000+ across all matched datasets |
| Manual work saved | ~40 hours (90% reduction) |

---

## How It Works

```
UN-DESA Excel Catalog  →  Extract dataset names
SDMX Registry API      →  Parse XML, extract series IDs & descriptions
                              ↓
                     Fuzzy Matching (SequenceMatcher)
                              ↓
         HIGH (>0.7) / MEDIUM (0.5–0.7) / LOW (<0.5)
                              ↓
                     Export CSVs + Summary Stats
```

**Why fuzzy matching?** Exact matching fails because the same indicator is described differently across sources. For example:

- UN-DESA: `"1.1.1 Prevalence of income poverty"`
- SDMX: `"SI_POV_NAHC: Proportion of population living below national poverty line"`

`SequenceMatcher` catches these.

---

## Project Structure

```
sdg-dataset-matcher/
├── UNDESA_datasets.ipynb      # Main pipeline notebook
├── Data/                      # Input data (gitignored)
│   └── DataSet - Catalog.xlsx
├── outputs/                   # Generated outputs (gitignored)
│   ├── matched_datasets.csv
│   ├── high_confidence_matches.csv
│   ├── medium_confidence_matches.csv
│   ├── low_confidence_matches.csv
│   ├── dataset_to_series_mapping.csv
│   ├── summary_statistics.csv
│   ├── sdmx_registry_data.csv
│   └── original_catalog.csv
├── .ipynb_checkpoints/        # gitignored
├── .gitignore
└── README.md
```

---

## Getting Started

### Prerequisites

```bash
pip install pandas requests numpy lxml matplotlib seaborn openpyxl
```

### Input Data

Place the UN-DESA catalog file in the `Data/` folder:

```
Data/DataSet - Catalog.xlsx
```

### Run the Notebook

Open `UNDESA_datasets.ipynb` in Jupyter and run all cells top to bottom. The pipeline will:

1. Load and validate the UN-DESA catalog
2. Query the SDMX Registry API for official SDG series
3. Run fuzzy matching across all datasets
4. Generate visualizations and quality reports
5. Export results to `outputs/`

---

## Outputs

| File | Description |
|---|---|
| `matched_datasets.csv` | All matches with scores |
| `high_confidence_matches.csv` | Score > 0.7 — production ready |
| `medium_confidence_matches.csv` | Score 0.5–0.7 — review recommended |
| `low_confidence_matches.csv` | Score ≤ 0.5 — manual validation needed |
| `dataset_to_series_mapping.csv` | Reference mapping (score > 0.5 only) |
| `summary_statistics.csv` | Aggregate metrics |
| `sdmx_registry_data.csv` | Raw SDMX registry pull |
| `original_catalog.csv` | Cleaned input catalog |

---

## Confidence Score Interpretation

| Score Range | Confidence | Recommended Action |
|---|---|---|
| > 0.7 | HIGH ✅ | Safe to use in downstream pipelines |
| 0.5 – 0.7 | MEDIUM ⚠️ | Spot-check with domain expert |
| ≤ 0.5 | LOW ❌ | Manual validation required |

---

## Data Sources

- **UN-DESA Dataset Catalog** — provided as Excel input
- **SDMX Registry API** — `https://registry.sdmx.org/ws/public/sdmxapi/rest/datastructure/IAEG-SDGs/SDG/latest/`

---

## Next Steps

- Review medium/low-confidence matches with domain experts
- Automate monthly re-runs to capture new datasets
- Extend pipeline to other UN data catalogs (UNDATA, UNStats)
- Incorporate TF-IDF or embedding-based matching to improve low-confidence coverage
