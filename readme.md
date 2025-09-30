# PDF Table Extraction & Statistical Analysis (Gradio App)

An interactive **Python + Gradio** app that:

* Extracts tables from PDFs via an **LLM** (JSON schema output).
* Splits **mean ± SD** (or `mean(sd)`) into separate columns.
* Detects **group sample sizes (N)** and lets you **override** them.
* Computes per-row **t** (sqrt(F)) or **F (ANOVA)** with rounding-error–aware **Nominal / Min / Max** ranges.
* Previews PDFs, displays tables, and exports CSVs.

---

## Features

* **PDF Upload & Preview**: Convert PDF pages to images and prompt an LLM to extract tables as JSON.
* **Automated Parsing**: Recognizes `mean ± sd` and `mean(sd)`; splits into `Mean` / `SD` columns.
* **Sample Size (N) Handling**:

  * Uses LLM-provided `group_ns` when available.
  * Allows manual override via JSON (e.g., `{"24-h group":56, "72-h group":51}`).
  * Heuristically infers Ns from cells like `56 (89.6%)` if needed.
* **Statistics**:

  * One-way or two-way ANOVA from summaries.
  * Toggle **t** (sqrt(F)) vs **F**.
  * Brackets rounding error using an enumerated/sampled range.
* **UI**:

  * Built with **Gradio**: select PDF/table, view data and analysis, download CSV.

---

## Project Structure (key parts)

* `extract_ai(pdf_path)`: PDF → images → LLM → JSON tables (with optional `group_ns`).
* `detect_group_mean_sd_headers(headers)`: Finds paired Mean/SD columns per group.
* `f_range(...)`: Computes nominal/min/max **F or t** accounting for rounding error.
* `analyze_selected_table(...)`: Per-row stats using means/SDs/Ns.
* Gradio app: upload, extract, pick table, load Ns, analyze, export.

---

## Installation

### 1) Clone

```bash
git clone https://github.com/rvhjo/table_extraction.git
cd table_extraction
```

### 2) Python deps

Use a virtual environment and install:

```bash
pip install -r requirements.txt
```

Example `requirements.txt`:

```
numpy
pandas
gradio
pdf2image
openai
pillow
```

### 3) Poppler (required by `pdf2image`)

* macOS: `brew install poppler`
* Ubuntu/Debian: `sudo apt-get install poppler-utils`
* Windows: `!apt-get -qq update && apt-get -qq install -y poppler-utils`

---


## Environment Variables

You need to provide your OpenAI API key so the app can call the API.

### Option 1: Temporary (per session)

Run before starting the app:
**PowerShell (Windows)**

```powershell
$env:OPENAI_API_KEY="your_api_key"
$env:DEPLOYMENT_NAME="gpt-4o"
python app.py
```

**macOS/Linux (bash/zsh)**

```bash
export OPENAI_API_KEY="your_api_key"
export DEPLOYMENT_NAME="gpt-4o"
python app.py
```

### Option 2: Permanent (system environment variable)

Add the variables in your OS environment settings:

* Name: `OPENAI_API_KEY` → Value: your API key
* Name: `DEPLOYMENT_NAME` → Value: `gpt-4o`

Then restart PowerShell/terminal.

### Option 3: Project-level `.env` file (recommended for Jupyter/teams)

Create a `.env` file in the project root:

```
OPENAI_API_KEY=your_api_key
DEPLOYMENT_NAME=gpt-4o
```

Install **python-dotenv** if not already:

```bash
pip install python-dotenv
```

Then load it in your Notebook or script:

```python
from dotenv import load_dotenv
import os

load_dotenv()  # reads the .env file
api_key = os.getenv("OPENAI_API_KEY")
deployment = os.getenv("DEPLOYMENT_NAME")
print("Key loaded?", api_key is not None)
```

⚠️ Make sure `.env` is in your `.gitignore` so your key is not exposed if you push to GitHub.




## How to Use (UI)

1. **Upload PDF(s)**.
2. Click **Extract** to run LLM extraction.
3. Pick **Select PDF** and **Select Table** to view data.
4. Click **Load detected Ns** to populate/modify group Ns (JSON).
5. Optionally add **Manual Ns override** (JSON).
6. Choose **Show t (sqrt of F)** and **dp** (auto = `-1`).
7. Click **Analyze selected table** for per-row **Nominal / Min / Max**.
8. Use **Download CSV** to export the table.

**Manual Ns example:**

```json
{"24-h group": 56, "72-h group": 51}
```

---

## 📊 Output (per-row)

| row_index | row_label   | nominal | min   | max   | dp | enumeration_mode | total_combinations |
| --------- | ----------- | ------- | ----- | ----- | -- | ---------------- | ------------------ |
| 0         | Age (years) | 1.273   | 1.201 | 1.335 | 2  | full             | 16                 |
| 1         | Height (cm) | 0.876   | 0.812 | 0.940 | 2  | full             | 16                 |

* `nominal/min/max` may contain multiple values when multiple effects are tested.
* If **Show t** is checked, values are `sqrt(F)`.

---

## Notes & Limitations

* LLM extraction may contain errors for complex/low-quality PDFs—use the UI to correct Ns and verify values.
* Stats are computed from **summaries** (means/SDs/Ns), so results are approximate and sensitivity to rounding is explicitly reported.
* Very large tables can hit token limits; split PDFs or run in batches.

---

## Security

* **Do not** hard-code API keys in source files.
* If you previously pushed a key, revoke it immediately and rotate credentials.
* Consider using `.gitignore`, `.env`, or a secrets manager.
