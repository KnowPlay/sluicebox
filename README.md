<!-- PROJECT SUMMARY -->
<div align="center">
  <h1 align="center">sluicebox</h1>

  <p align="center">
    An ELT pipeline that mines job market intelligence from 130k+ LinkedIn postings.
    <br>
    <a href="https://github.com/KnowPlay/sluicebox/issues">» submit a suggestion </a>
    ·
    <a href="https://github.com/KnowPlay/sluicebox/issues">» report a bug </a>
    ·
    <a href="https://github.com/KnowPlay/sluicebox">» contact </a>
  </p>

  <div align="center">

![GitHub forks](https://img.shields.io/github/forks/KnowPlay/sluicebox?style=social) ![GitHub stars](https://img.shields.io/github/stars/KnowPlay/sluicebox?style=social)

[![CI](https://github.com/KnowPlay/sluicebox/actions/workflows/ci.yml/badge.svg)](https://github.com/KnowPlay/sluicebox/actions/workflows/ci.yml)
![GitHub Pull Request (open)](https://img.shields.io/github/issues-pr/KnowPlay/sluicebox?color=blue) ![GitHub last commit](https://img.shields.io/github/last-commit/KnowPlay/sluicebox?color=pink) ![GitHub License](https://img.shields.io/github/license/KnowPlay/sluicebox?color=green) ![contributions welcome](https://img.shields.io/badge/contributions-welcome-purple.svg?style=flat)

  </div>
</div>

<!-- TABLE OF CONTENT -->
<details open="open">
  <summary><h2 style="display: inline-block">🕹 Table of Content</h2></summary>
  <ol>
    <li>
      <a href="#🌻-about">About</a>
      <ul>
        <li><a href="#🔧-tech-stack">Tech Stack</a></li>
        <li><a href="#🍄-features">Features</a></li>
      </ul>
    </li>
    <li>
      <a href="#🌵-documentation">Documentation</a>
      <ul>
        <li><a href="#🍯-setup">Setup</a></li>
        <li><a href="#🍎-development">Development</a></li>
      </ul>
    </li>
    <li><a href="#🌾-contributing">Contributing</a></li>
    <li><a href="#📜-license">License</a></li>
  </ol>
</details>

<!-- ABOUT -->

## :sunflower: About

**sluicebox** is an end-to-end ELT pipeline built on the [LinkedIn Job Postings 2023–2024 dataset](https://www.kaggle.com/datasets/arshkon/linkedin-job-postings) — 130k+ real job listings with titles, descriptions, salaries, skills, and company metadata. The pipeline extracts raw postings, cleans and parses them into structured signals (salary ranges, skill flags, experience levels), loads them into a normalized analytics warehouse, and answers real questions about the data engineering job market using SQL.

> A sluice box is a gold mining tool that separates valuable material from raw rock. That's the pipeline.

### :hammer_and_wrench: Tech Stack

#### :heavy_plus_sign: Development Tools

- [x] Python 3.11+
- [x] pytest + pytest-cov
- [x] GitHub Actions (CI)

#### :heavy_plus_sign: Data & Backend

- [x] pandas — chunked extraction and transformation
- [x] NumPy — vectorized operations
- [x] SQLAlchemy — database abstraction and connection management
- [x] SQLite — relational storage, swappable for PostgreSQL or SQL Server

#### :heavy_plus_sign: DevOps

- [x] GitHub Actions — automated test runs on every push to `main`
- [x] `.github/ISSUE_TEMPLATE` — structured bug, feature, chore, and deployment templates

### :mushroom: Features

#### :heavy_plus_sign: Extract

- [x] Class-based `JobExtractor` with chunked-read and full-load modes
- [x] Separate `extract_companies()` loader for the companion companies CSV
- [x] `profile()` utility — reports nulls, dtypes, unique counts, and samples per column before transforms run
- [x] `detect_mixed_types()` — quantifies numeric vs string mix in chaotic columns like salary

#### :heavy_plus_sign: Transform

- [x] `parse_salary()` — normalizes 6 salary formats (`$80K–$120K`, `95K`, `$75/hr`, plain float, null, free text) into `salary_low`, `salary_mid`, `salary_high` floats using regex named groups
- [x] `strip_html()` / `clean_description()` — removes HTML tags from raw descriptions using stdlib `html.parser`, computes description length
- [x] `extract_skills()` — dictionary-driven, whole-word regex detection of 18 tech skills; produces binary `skill_{name}` flag columns and a `skill_count` total
- [x] `normalize_location()` — cleans location strings, flags remote postings
- [x] `normalize_experience()` — maps raw experience level values to standard buckets via a `LEVEL_MAP` dictionary
- [x] `validate_schema()` — circuit breaker that raises before a malformed DataFrame can reach the database
- [x] Configurable `PIPELINE` list — add, remove, or reorder transforms without touching orchestration logic
- [x] Row-count logging at every step for full pipeline observability

#### :heavy_plus_sign: Load

- [x] Star schema: `dim_companies`, `dim_dates`, `fact_postings`
- [x] `dim_dates` derived dynamically from posting months found in the data — no static seed file
- [x] Dependency-ordered loading: dimensions before facts to satisfy foreign keys
- [x] Idempotent full-refresh load — safe to re-run without duplicating data
- [x] DDL version-controlled in `sql/schema.sql` alongside the code

#### :heavy_plus_sign: SQL Analytics

- [x] Top 10 most demanded skills with % share of all postings
- [x] Average, min, and max salary by experience level
- [x] Salary premium per skill — avg salary with vs without each skill, and the delta
- [x] Skill co-occurrence with Python — which skills appear most alongside it
- [x] Monthly posting volume with running total and 3-month rolling average (window function)
- [x] Remote vs on-site salary comparison by experience level

#### :heavy_plus_sign: Testing

- [x] Unit tests for every UDF covering normal cases, edge cases, and null inputs
- [x] `@pytest.mark.parametrize` on salary parsing — 6 input formats in one test function
- [x] pytest fixtures for shared sample DataFrames
- [x] 80%+ code coverage target enforced in CI

<!-- DOCUMENTATION -->

## :cactus: Documentation

### :honey_pot: Setup

**Prerequisites:** Python 3.11+, a Kaggle account with [API credentials](https://www.kaggle.com/docs/api) configured at `~/.kaggle/kaggle.json`.

```bash
# 1. Clone the repo
git clone https://github.com/KnowPlay/sluicebox.git
cd sluicebox

# 2. Create and activate a virtual environment
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate

# 3. Install dependencies
pip install -r requirements.txt

# 4. Download the source dataset
kaggle datasets download arshkon/linkedin-job-postings --path data/raw/ --unzip
```

### :apple: Development

**Run the full pipeline:**

```bash
python -m etl.pipeline
```

Initializes the database, extracts job postings and company data, applies all transforms, validates the output schema, and loads into the star schema warehouse. Every step is logged to stdout with timestamps and row counts.

**Run the test suite:**

```bash
pytest tests/ -v                 # all tests with verbose output
pytest tests/ --cov=etl          # with coverage report
```

**Query the warehouse:**

```python
import sqlite3, pandas as pd

conn = sqlite3.connect("sluicebox.db")

# example: top 10 skills by posting frequency
df = pd.read_sql("""
    SELECT skill, SUM(mentions) AS total_postings,
           ROUND(100.0 * SUM(mentions) / (SELECT COUNT(*) FROM fact_postings), 1) AS pct
    FROM (
        SELECT 'python' AS skill, SUM(skill_python) AS mentions FROM fact_postings UNION ALL
        SELECT 'sql',              SUM(skill_sql)               FROM fact_postings UNION ALL
        SELECT 'spark',            SUM(skill_spark)             FROM fact_postings UNION ALL
        SELECT 'aws',              SUM(skill_aws)               FROM fact_postings
    )
    GROUP BY skill ORDER BY total_postings DESC
""", conn)
```

Or open `sql/analytics.sql` directly in any SQLite client pointed at `sluicebox.db`.

**Project structure:**

```
sluicebox/
├── .github/
│   ├── ISSUE_TEMPLATE/     # bug, feature, chore, deployment, documentation templates
│   └── workflows/
│       └── ci.yml          # runs pytest on every push to main
├── data/
│   ├── raw/                # source CSVs — gitignored, download via Kaggle CLI
│   └── processed/          # intermediate outputs
├── etl/
│   ├── extract.py          # JobExtractor — chunked, full-load, and company modes
│   ├── transform.py        # UDF library, PIPELINE list, schema validator
│   ├── load.py             # DB init, dimension + fact loaders, load_all orchestrator
│   ├── pipeline.py         # single entry point: extract → transform → load
│   └── utils.py            # profile(), detect_mixed_types(), shared helpers
├── sql/
│   ├── schema.sql          # DDL — dim_companies, dim_dates, fact_postings
│   └── analytics.sql       # 6 analytical queries
├── tests/
│   ├── test_transform.py   # UDF unit tests — fixtures, parametrize, edge cases
│   └── test_load.py        # load function tests with mocked DB
├── config.py               # paths, DB_URL, get_logger() factory
├── requirements.txt
└── README.md
```

**Key design decisions:**

- **Functional transform chain** — each UDF is a pure function (DataFrame in, DataFrame out). Pure functions are independently testable, composable, and produce no side effects. The `PIPELINE` list means steps can be reordered without touching orchestration logic.
- **Idempotent loading** — the load step uses truncate-and-replace so re-running the pipeline never produces duplicate rows. Critical for pipelines that get re-triggered on failure.
- **Schema validation as a gate** — `validate_schema()` runs between transform and load, acting as a circuit breaker that fails fast before bad data reaches the database.
- **Denormalized skill flags** — skill presence is stored as individual integer columns on `fact_postings` rather than a separate skills table. This is a deliberate tradeoff: it makes co-occurrence and aggregation queries significantly faster at the cost of a wider table.
- **Config-driven paths** — all file paths and the DB URL live in `config.py`, never hardcoded. Swap SQLite for PostgreSQL by changing one line.

<!-- CONTRIBUTING -->

## :ear_of_rice: Contributing

Contributions, questions, and suggestions are welcome — especially from others learning data engineering.

> 1. Fork the Project
> 2. Create your Branch (`git checkout -b my-branch`)
> 3. Commit your Changes (`git commit -m 'add my contribution'`)
> 4. Push to the Branch (`git push --set-upstream origin my-branch`)
> 5. Open a Pull Request

<!-- LICENSE -->

## :pencil: License

This project is licensed under the [Apache License 2.0](./LICENSE).

<!-- Gratitude to Arsh Kon and Kaggle for the LinkedIn Job Postings dataset. -->
