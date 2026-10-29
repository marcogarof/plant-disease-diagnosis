# Plant Disease Diagnosis documentation

ML-powered phytosanitary decision support: diagnose crop diseases from a single
leaf photo and return the top-k `(crop, disease)` labels with confidence.

## Contents

- [Project introduction](introduction.md) — problem, users, use cases, scope.
- [ML Canvas (PDF)](ml_canvas.pdf) — Louis Dorard canvas for M1 Inception.

## Commands

The Makefile is the main entry point:

```bash
make requirements   # uv sync
make dvc-repro      # reproduce the DVC pipeline
make mlflow         # MLflow UI
make api            # FastAPI service
make test           # pytest
make lint           # ruff
```

Build the docs with `uv run mkdocs build -f docs/mkdocs.yml`.
