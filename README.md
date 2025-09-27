# LILY
Python workflow for extracting structured information on Nature-based Solutions (NbS) using LLM prompt engineering.  Developed during COST Action LILY Virtual Mobility Grant (NbS-Health Data Platform).

# LILY LLM Extraction Workflow

This repository contains the Python workflow developed during the COST Action LILY Virtual Mobility Grant 
(NbS-Health Data Platform Development: LLM Prompt Engineering).

## Features
- Controlled and Uncontrolled prompt families for extracting NbS project data
- Normalisation of key fields (status, scale, impacts)
- Outputs in JSON, NDJSON, and CSV formats
- Step-by-step User Guide and training slides included

# LILY – HTML → Structured Data (OpenAI)

Extracts structured fields from local HTML files using OpenAI and saves results to JSON / NDJSON / CSV.

---

## 1. Prerequisites
- Python **3.10+** (Anaconda recommended)
- An **OpenAI API key**
- Local copies of project **HTML** files

---

## 2. Install dependencies
Create/activate a clean environment, then install:
```bash
pip install openai pandas beautifulsoup4 python-dotenv requests
```

## 3. Folder set-up

Create a project folder like:
```bash
project_root/
  ├─ raw_html_data_1/          # put your .html files here
  ├─ outputs/                  # created automatically
  ├─ .env                      # holds your API key
  └─ CostAction_Controlled.py  # your Python file (this code)

```

## 4. Configure your API key

Create a file named .env in `project_root`:
```ini
OPENAI_API_KEY=sk-xxxxxxxxxxxxxxxx
```

## 5. Place input files

```bash
Copy your source HTML files into:

project_root/raw_html_data_1/
```

## 6. Check the input path in the script

Use a portable path (no hard-coded absolute paths):
```python
from pathlib import Path

BASE_DIR = Path.cwd() / "raw_html_data_1"
files = sorted([str(p) for p in BASE_DIR.glob("*.html")])
```

This reads all `.html` files from `raw_html_data_1` under the current working directory.

## 7. Choose prompt strategy (Controlled vs Uncontrolled)

- Controlled (default): uses `CONTROLLED_SOLUTION_TYPES` in `SYSTEM_PROMPT` for consistent `solution_types`.

- Uncontrolled (flexible): create `SYSTEM_PROMPT_UNCONTROLLED` that omits the “Choose ONLY from this controlled list…” block and pass it when you want inductive coding.

Minimal pattern:

```python
USE_CONTROLLED = True  # set False for Uncontrolled

sys_prompt = SYSTEM_PROMPT if USE_CONTROLLED else SYSTEM_PROMPT_UNCONTROLLED
# then pass sys_prompt into extract_with_gpt(_extended)
```

## 8. Run the script

From `project_root`:

```bash
python CostAction_Controlled.py
```

Expected console messages:

```bash
✅ Packages loaded successfully
✅ Requests package loaded successfully
Base dir: .../raw_html_data_1
⚠️ schema mismatch ... (only if fields are missing/extra)
✅ Saved JSON, NDJSON, and CSV → outputs
```

## 9. Outputs (in `outputs/`)

- `nbs_sample.json` — full records (pretty-printed JSON)

- `nbs_sample2.ndjson` — line-delimited JSON (one record per line)

- `nbs_sample2.csv` — spreadsheet-friendly view (list fields JSON-encoded)

Tip: Open the CSV first for a quick scan. Key fields include:

- `title`, `summary`, `status`, `location_name`, `country`, `scale`

- `solution_types`, `challenges_addressed`, `health_linkages_primary`

- `impacts` (array of `{description, type}`), `governance`, `url_source`, `environmental_context`

## 10. Quality checks

- Spot-check a few rows against their source HTML.

- Compare Controlled vs Uncontrolled runs when categories look uncertain.

- Watch for schema mismatch warnings and inspect those files.

## 11. Troubleshooting

- `PENAI_API_KEY` is not set → Check `.env` location/name; run from `project_root`.

- API error 401/429 → Invalid key or rate limit; retry later or reduce batch size.

- JSON parse failed → Very long/dirty HTML; try truncating input:

```python
html = html[:100_000]
```

or re-save a cleaner copy.

- schema mismatch → A field is missing/extra; review the raw output for that file.

- No files found → Confirm `raw_html_data_1` exists and filenames end with `.html`; confirm the `BASE_DIR.glob("*.html")` line above.

## 12. Data privacy & costs

- Privacy: anonymise or redact sensitive personal data before tests.

- Costs: start with 5–10 files; keep `temperature=0.0`; `gpt-4o-mini` is cost-efficient.

## 13. Reproducibility tips

- Keep a small test set for regression checks.

- Save prompt versions (Controlled/Uncontrolled) and note any edits.

- Log run parameters (date, model, prompt mode) alongside outputs.