# Plant Disease Diagnosis — Project Introduction

## 1. Problem Statement

Crop diseases cause an estimated **20–40% global yield loss** annually (FAO), translating to over **$220 billion** in economic damage. Smallholder farmers and hobby gardeners—who often lack access to agronomic expertise—are disproportionately affected. Timely, accurate diagnosis is the single most effective intervention, yet:

- **Expert scarcity**: Plant pathologists are concentrated in research institutions, not in fields
- **Diagnostic delay**: Lab-based analysis takes days to weeks; by the time results arrive, the infection has spread
- **Knowledge gap**: Farmers rely on visual inspection, which is error-prone and biased toward familiar diseases
- **Scalability**: Traditional extension services cannot cover millions of dispersed smallholdings

The result: **preventable losses, excessive pesticide use (environmental harm), and food insecurity.**

---

## 2. Solution Overview

**Plant Disease Diagnosis** is an **ML-powered decision-support component** that turns a single leaf photograph into an actionable diagnosis within seconds.

| Input | Output |
|-------|--------|
| RGB image of a single leaf (mobile/web upload) | Top-*k* predictions: `(crop_species, disease_or_healthy)` + confidence scores + human-readable advisory |

The component is delivered as a **production-grade, containerized web API** (FastAPI) with an **MCP server** interface, enabling integration into:
- Farmer-facing mobile apps
- Advisory chatbots (via MCP)
- IoT/edge devices in smart greenhouses
- Agricultural extension platforms

---

## 3. Target Users

| Persona | Context | Pain Point | How They Use It |
|---------|---------|------------|-----------------|
| **Smallholder farmer** (Kenya, India, Brazil…) | 0.5–5 ha, smartphone, intermittent connectivity | No local agronomist; loses 30%+ yield to disease | Snaps leaf photo → gets diagnosis + treatment advice in local language |
| **Hobby gardener / urban farmer** (Europe, NA) | Backyard / balcony / community plot | Over/under-treats; misidentifies benign spots | Quick check before buying chemicals; learns disease patterns |
| **Agricultural extension officer** | Field visits, tablet, multiple farms/day | Too many farms, too little time | Batch upload → triage priority cases; shares reports via WhatsApp |
| **Agri-tech platform / coop** | B2B SaaS, integrates via API/MCP | Needs white-label diagnostic engine | Embeds component in their app; pays per inference or subscription |

---

## 4. Key Use Cases

| ID | Use Case | Actor | Trigger | Success Criteria |
|----|----------|-------|---------|------------------|
| **UC-01** | **Real-time field diagnosis** | Farmer | Sees suspicious leaf | Diagnosis < 5 s; confidence ≥ 80%; actionable advice shown |
| **UC-02** | **Offline-first capture** | Farmer | No internet in field | Photo queued locally; auto-syncs when online; result pushed via notification |
| **UC-03** | **Batch scouting** | Extension officer | Weekly farm visit | Upload 20+ images → ranked list of urgent cases in < 30 s |
| **UC-04** | **Chatbot integration** | Farmer (via WhatsApp/Telegram bot) | Sends photo to bot | Bot replies with diagnosis + MCP-tool call for treatment lookup |
| **UC-05** | **Greenhouse monitoring** | IoT gateway | Camera captures leaf daily | Anomaly detection → alert if new disease class appears |

---

## 5. Value Proposition

| Dimension | Before | After (with this component) |
|-----------|--------|------------------------------|
| **Time to diagnosis** | Days–weeks (lab) / never (no expert) | **Seconds** |
| **Accessibility** | Expert-dependent | **Smartphone-only** |
| **Cost per diagnosis** | $10–50 (lab + travel) | **~$0.001** (cloud inference) |
| **Pesticide misuse** | Calendar spraying / guesswork | **Targeted, evidence-based** |
| **Scalability** | 1 expert : 1,000 farmers | **1 model : millions of users** |
| **Data generation** | None | **Geo-tagged, time-series disease surveillance** |

---

## 6. Scope & Boundaries

| In Scope | Out of Scope |
|----------|--------------|
| Multi-class classification (crop × disease) on **single-leaf images** | Whole-plant / canopy / drone imagery |
| Top-*k* predictions with calibrated confidence | Segmentation, lesion measurement, severity scoring |
| Advisory text (treatment, prevention, resistant varieties) | Prescription-grade pesticide recommendation (regulatory) |
| API + MCP interfaces for integration | Native mobile app (consumer-facing UI) |
| Reproducible pipeline, CI/CD, monitoring, container deploy | On-device / edge optimization (TFLite, CoreML) — future work |

---

## 7. Dataset & Modeling Approach

- **Dataset**: **PlantVillage** (Mohanty et al.) — ~54,000 labeled images, 38 classes (14 crops × 26 diseases + healthy)
- **Source**: Hugging Face `mohanty/PlantVillage` (cleaned, split, versioned via DVC)
- **Model**: Transfer learning (EfficientNet-B0 / ResNet-50 / ViT-Base) fine-tuned on PlantVillage
- **Target metrics**: Top-1 accuracy ≥ 95%, macro-F1 ≥ 0.93, calibration error (ECE) < 0.05
- **Robustness**: Test-time augmentation, OOD detection for non-leaf / unknown-crop images

---

## 8. Technical Architecture (High Level)

```
┌─────────────┐     ┌──────────────┐     ┌─────────────┐
│  Client     │────▶│  FastAPI     │────▶│  Model      │
│  (App/Bot)  │     │  /predict    │     │  Server     │
└─────────────┘     └──────┬───────┘     └──────▲──────┘
                           │                    │
                    ┌──────▼───────┐     ┌──────┴──────┐
                    │  MCP Server  │     │  MLflow     │
                    │  (tools)     │     │  Registry   │
                    └──────────────┘     └─────────────┘
                           │
                    ┌──────▼───────┐
                    │  Docker      │
                    │  Compose     │
                    └──────┬───────┘
                           ▼
                    ┌─────────────┐
                    │  Cloud      │
                    │  (AWS/GCP/  │
                    │   Azure/    │
                    │   Render)   │
                    └─────────────┘
```

- **Reproducibility**: DVC (data + model), MLflow (experiments), Prefect (pipelines)
- **Quality**: Great Expectations (data), Deepchecks (model), pytest + Ruff (code)
- **CI/CD**: GitHub Actions → build → test → scan → deploy
- **Monitoring**: Prometheus + Grafana (latency, error rate, drift), Alibi Detect (data/concept drift)

---

## 9. Success Criteria (Project-Level)

| Milestone | Deliverable | Acceptance |
|-----------|-------------|------------|
| **M1 Inception** | ML Canvas, repo, Cookiecutter structure | Canvas approved; repo passes template lint |
| **M2 Reproducibility** | DVC-tracked data, MLflow experiments, Prefect pipeline | `dvc repro` → identical results; pipeline runs on CI |
| **M3 Quality Assurance** | Test suite (unit + integration + data/model), Great Expectations suite | Coverage ≥ 80%; all checks pass on PR |
| **M4 API & MCP** | FastAPI `/predict`, `/health`, MCP server with `diagnose_leaf` tool | OpenAPI docs; MCP inspector connects; latency p95 < 200 ms |
| **M5 Deployment** | Docker image, GitHub Actions CI/CD, cloud deployment | `docker compose up` works; CI/CD deploys to staging/prod |
| **M6 Monitoring** | Prometheus metrics, Grafana dashboards, drift alerts | Dashboards live; alert fires on synthetic drift injection |

---

## 10. Ethical & Regulatory Considerations

- **Bias**: PlantVillage is skewed toward major crops / well-studied regions. Mitigation: report per-class metrics; document known gaps; plan fine-tuning on local data.
- **Misdiagnosis risk**: Low-confidence predictions (`< 0.6`) return *"Uncertain — consult expert"* instead of a guess.
- **Data privacy**: No PII in leaf images; EXIF GPS stripped by default; opt-in for geo-tagged surveillance.
- **Regulatory**: Not a medical device; agricultural advisory only. Disclaimer included in API response.

---

## 11. Roadmap (Post-Exam)

| Phase | Focus |
|-------|-------|
| **v1.1** | On-device inference (TFLite), offline queue, multi-language advisory |
| **v1.2** | Severity scoring (lesion area %), time-series tracking per field |
| **v2.0** | Multi-image (whole plant + context), integration with weather / satellite data |

---

## 12. References

1. Mohanty, S. P., Hughes, D. P., & Salathé, M. (2016). *Using Deep Learning for Image-Based Plant Disease Detection*. Frontiers in Plant Science.
2. FAO. (2019). *The State of Food and Agriculture 2019: Moving Forward on Food Loss and Waste Reduction*.
3. Kastner, C. (2024). *Machine Learning in Production*. https://mlip-cmu.github.io/book/
4. Sculley et al. (2015). *Hidden Technical Debt in Machine Learning Systems*. NIPS.
5. Lanubile, F., Martínez-Fernández, S., & Quaranta, L. (2024). *Training Future Machine Learning Engineers: A Project-Based Course on MLOps*. IEEE Software.