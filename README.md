# Plant Disease Diagnosis

Phytosanitary decision-support component that diagnoses crop diseases from a
single leaf photograph and returns the top-k `(crop, disease)` labels with
confidence scores and an actionable advisory.

> **Course:** Software Engineering for AI-Enabled Systems (SE4AI), University of Bari
> **Exam track:** Pair project (Marco Garofalo, Luan Nurja)
> **Milestone:** M1 Inception (Project structure & ML Canvas)

## 1. Problem & value proposition

Crop diseases cause 20–40% of global yield losses every year. Smallholder farmers
and hobby gardeners often have no fast access to an agronomist, and lab-based
diagnosis takes days to weeks. This project turns a leaf photo into a diagnosis in
seconds, at a cost of roughly $0.001 per inference.

| Dimension | Before | After |
|---|---|---|
| Time to diagnosis | days–weeks (lab) / never (no expert) | seconds |
| Access | expert-dependent | smartphone |
| Cost per diagnosis | $10–50 | ~$0.001 |
| Pesticide use | calendar spraying / guesswork | targeted, evidence-based |

Target users: smallholder farmers, hobby/urban gardeners, agricultural extension
officers, agri-tech platforms (white-label API/MCP).

Key use cases: real-time field diagnosis (UC-01), offline-first capture (UC-02),
batch scouting (UC-03), chatbot integration via MCP (UC-04), greenhouse IoT
monitoring (UC-05). See [`docs/introduction.md`](docs/introduction.md) and the
[`docs/ml_canvas.tex`](docs/ml_canvas.tex) canvas for the full analysis.

## 2. Dataset

- **PlantVillage** (Mohanty et al., 2016): ~54k colour leaf images, 38 classes
  (14 crops × diseases + healthy).
- Recommended source: Hugging Face `mohanty/PlantVillage` (archive `data.zip`).
- Tracked with DVC; the raw data is never committed to Git.

## 3. Project organization (Cookiecutter Data Science)

```
├── data
│   ├── external       <- Third-party data.
│   ├── interim        <- Intermediate transformed data.
│   ├── processed      <- Canonical datasets for modeling (manifest, splits).
│   └── raw            <- Original immutable dump (DVC-tracked).
├── docs               <- MkDocs site, ML Canvas, project introduction.
├── models             <- Trained/serialized models (DVC output).
├── notebooks          <- Jupyter notebooks (naming: 1.0-mg-description).
├── references         <- Data dictionaries and manuals.
├── reports            <- Metrics and generated figures.
├── tests              <- pytest suite.
├── plant_disease_diagnosis
│   ├── config.py      <- Paths and params loader.
│   ├── dataset.py     <- PlantVillage download/extract.
│   ├── features.py    <- Manifest + class mapping.
│   ├── splitting.py   <- Stratified train/val/test split.
│   ├── modeling
│   │   ├── data.py    <- Torch dataset + transforms.
│   │   ├── model.py   <- Backbone factory (EfficientNet-B0).
│   │   ├── train.py   <- Training + MLflow tracking.
│   │   ├── evaluate.py<- Test metrics (accuracy, macro-F1, ECE).
│   │   └── predict.py <- Reusable inference API.
│   ├── api/main.py    <- FastAPI service + Prometheus metrics.
│   └── mcp_server.py  <- MCP server exposing `diagnose_leaf`.
├── dvc.yaml           <- Declarative pipeline (download → ... → evaluate).
├── params.yaml        <- Shared parameters.
├── Dockerfile         <- API container image.
├── docker-compose.yml <- API + MLflow + Prometheus + Grafana.
└── .github/workflows  <- CI: lint, tests, container build.
```

## 4. Quickstart

Prerequisites: `uv`, `git`, `dvc` (installed with `uv tool install ...`).

```bash
# Install the environment (Python 3.12 + lockfile)
uv sync

# Download PlantVillage (~2 GB) and reproduce the pipeline
uv run python -m plant_disease_diagnosis.dataset download
uv run dvc repro

# Inspect the pipeline and metrics
uv run dvc dag
uv run dvc metrics show
```

### Development commands

```bash
make lint          # ruff format --check + ruff check
make test          # pytest
make dvc-repro     # reproduce the full DVC pipeline
make mlflow        # MLflow UI on http://localhost:5000
make api           # FastAPI on http://localhost:8000 (docs at /docs)
make mcp           # run the MCP server (stdio)
make docker-build  # build the API image
```

## 5. MLOps tooling (course stack)

| Module | Tools |
|---|---|
| M1 Inception | Cookiecutter Data Science, ML Canvas, Git/GitHub Flow |
| M2 Reproducibility | DVC (data/model versioning, pipeline), MLflow, Prefect |
| M3 Quality assurance | Ruff, pytest, Great Expectations, Deepchecks |
| M4 API | FastAPI, Model Cards, MCP Python SDK |
| M5 Deployment | Docker, Docker Compose, GitHub Actions |
| M6 Monitoring | Prometheus, Grafana, Locust, Alibi Detect (optional) |

## 6. Milestones checklist

- [x] M1 — CCDS structure, Git repo, DVC init, ML Canvas, project introduction
- [ ] M2 — Data download + DVC pipeline running, MLflow experiments logged
- [ ] M3 — Data/model test suites, coverage ≥ 80%
- [ ] M4 — `/predict` + `/health`, MCP `diagnose_leaf`, model card
- [ ] M5 — Docker image + CI/CD + cloud deployment
- [ ] M6 — Prometheus/Grafana dashboards + drift alerting

## 7. Ethics & safety

- Low-confidence predictions (`action = abstain`) return “consult an expert”
  instead of a guess, to avoid dangerous pesticide misuse.
- PlantVillage is geographically skewed; per-class metrics and known gaps must be
  documented in the model card.
- EXIF/GPS is stripped at ingestion; no personally identifiable information is
  processed.
- Advisory support only — not a legal pesticide mandate.

## 8. License

MIT — see [`LICENSE`](LICENSE).
