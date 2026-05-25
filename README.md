# de-foundations

> Foundations for building production-grade data engineering systems on the lakehouse architecture — PySpark, Delta Lake, and the patterns that scale from a laptop to a cluster.

[![Python 3.11](https://img.shields.io/badge/python-3.11-blue.svg)](https://www.python.org/downloads/release/python-3110/)
[![PySpark 3.5](https://img.shields.io/badge/pyspark-3.5.3-orange.svg)](https://spark.apache.org/docs/3.5.3/)
[![Delta Lake 3.2](https://img.shields.io/badge/delta--lake-3.2.1-00ADD8.svg)](https://docs.delta.io/3.2.1/)
[![Code Style: Ruff](https://img.shields.io/badge/code%20style-ruff-d7ff64.svg)](https://github.com/astral-sh/ruff)
[![Type-Checked: Mypy](https://img.shields.io/badge/type--checked-mypy%20strict-blue.svg)](https://mypy-lang.org/)

---

## Scope

This repository is the working ground for foundational data engineering patterns: distributed compute mental models, Delta Lake transaction semantics, lakehouse design (bronze / silver / gold), and the operational discipline that takes pipelines from notebooks to production.

It is not a tutorial collection. Each module is built to a standard that would be acceptable in a production pull request — explicit types, no silent failures, tests where logic warrants them, and architectural intent documented before code is written.

## Design Principles

| Principle | What it means here |
|---|---|
| Reproducibility | Pinned Python (3.11) and Spark (3.5.3) versions, locked dependencies via `uv`, environment defined in `pyproject.toml` |
| Type safety | `mypy --strict` across `src/`; runtime data contracts enforced via explicit Spark schemas, never inferred in production paths |
| Separation of concerns | Exploratory work in `notebooks/`, reusable logic graduated to `src/de_foundations/` with tests |
| Fail loudly | No silent exception suppression; bad data routed to quarantine, not dropped |
| Cost awareness | Local development by default; cloud spend justified per workload |

## Architecture (target)

The repository builds toward a medallion lakehouse pattern:
Source  --->  Bronze (raw, append-only)  --->  Silver (cleaned, conformed)  --->  Gold (modelled, consumable)
|                                  |                                |
+---- Delta transaction log -------+---- Schema enforcement --------+
Each layer has different guarantees, retention, and downstream contracts. Layer transitions are explicit, idempotent, and testable.

## Repository Layout
de-foundations/
├── .github/workflows/      CI: lint, type-check, test (GitHub Actions)
├── data/
│   ├── raw/                Source data — never committed
│   └── processed/          Pipeline outputs — never committed
├── docs/                   Architecture decision records (ADRs), diagrams
├── notebooks/              Exploratory PySpark / Delta work
├── src/de_foundations/     Production-grade reusable modules
└── tests/                  Pytest unit + integration tests
## Toolchain

| Concern | Tool | Why |
|---|---|---|
| Python version management | `pyenv` | Per-project pinning via `.python-version` |
| Dependency + project management | `uv` | Reproducible installs, lockfile, 10-100x faster than pip |
| Linter + formatter | `ruff` | Replaces black + flake8 + isort; one tool, sub-second |
| Static type checker | `mypy --strict` | Catches contract violations at edit time, not in prod |
| Test runner | `pytest` | Standard; coverage via `pytest-cov` |
| Pre-commit hygiene | `pre-commit`, `nbstripout` | Notebook outputs stripped, secrets scanned before commit |
| Runtime engine | Apache Spark 3.5.3 (local) | Matches Databricks Runtime 15.4 LTS Python 3.11 surface |
| Storage layer | Delta Lake 3.2.1 | ACID, time travel, schema evolution on top of Parquet |

## Getting Started

Prerequisites: macOS or Linux, `pyenv`, `uv`, Java 17 (Temurin).

```bash
git clone git@github.com:AbhishekVadlamudi/de-foundations.git
cd de-foundations
pyenv install 3.11.10  # if not already installed
uv sync --all-extras   # installs runtime + dev dependencies into .venv
```

Verify the environment:

```bash
uv run python -c "from pyspark.sql import SparkSession; print(SparkSession.builder.appName('verify').getOrCreate().version)"
```

Expected output: `3.5.3`.

## Roadmap

- [x] Toolchain and repository scaffold
- [ ] Local Spark + Delta hello-world with transaction log inspection
- [ ] Schema enforcement patterns (Bronze → Silver)
- [ ] Slowly Changing Dimensions on Delta (Type 2)
- [ ] Streaming ingest with structured streaming + Delta sink
- [ ] CI pipeline (GitHub Actions): lint, type-check, test on every PR
- [ ] Migration to Azure Databricks with Unity Catalog (Phase 2)

## License

MIT — see `LICENSE`.
