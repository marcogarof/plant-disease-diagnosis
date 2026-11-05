# Data dictionary — PlantVillage

| Column | Type | Description |
|---|---|---|
| `filepath` | string | Path relative to `data/raw/plantvillage` (e.g. `color/Tomato___healthy/img.jpg`). |
| `label` | int | Integer class id (0–37), consistent with `labels.json`. |
| `class_name` | string | Original PlantVillage folder name (`Crop___Disease` or `Crop___healthy`). |
| `crop` | string | Crop species parsed from `class_name` (e.g. `Tomato`, `Apple`). |
| `disease` | string | Disease name parsed from `class_name`; `healthy` for healthy controls. |

## Classes

38 mutually exclusive classes across 14 crops:

Apple, Blueberry, Cherry, Corn (maize), Grape, Orange, Peach, Bell Pepper,
Potato, Raspberry, Soybean, Squash, Strawberry, Tomato.

## Splits

Stratified 70/15/15 train/validation/test split, fixed seed (`params.yaml`), locked
with DVC so every run is reproducible.

## Provenance

- Mohanty, S. P., Hughes, D. P., & Salathé, M. (2016). *Using Deep Learning for
  Image-Based Plant Disease Detection*. Frontiers in Plant Science.
- Hugging Face: `mohanty/PlantVillage` (license CC-BY-SA-3.0).
