# spring-2026-strength-and-nutrition
Team project: spring-2026-strength-and-nutrition
# Predicting Food Processing & Environmental Impact
---

## Project Summary

This project builds a machine learning pipeline that:
1. Learns to predict the **FPro food-processing score** from 12 standard nutrition-label nutrients using US grocery data (GroceryDB)
2. Transfers the trained model to **French CIQUAL 2025** reference foods without retraining
3. Asks whether **lower food processing also means lower environmental footprint** using Agribalyse 3.1 life-cycle assessment data

The short answer to question 3 is **no** — processed animal products (cheese, cured meats, smoked fish) can have a higher environmental footprint than many ultra-processed grain- or sugar-based products.

---

## Key Results

| Metric | Value |
|---|---|
| RF Regression R² (GroceryDB holdout) | **0.973** |
| RF Regression RMSE | **0.041** |
| RF Classification Accuracy | **95.9 %** |
| RF Classification Macro F1 | **0.904** |
| 10-feature portable model R² | **0.974** |
| FPro vs climate-change correlation (CIQUAL) | **0.002** (near zero) |

---

## Repository Structure

```
nutrition/
│
├── files FPro analysis/          ← Main analysis (current phase)
│   ├── 01_FPro_EDA_Ingredient_Analysis.ipynb   # EDA, price vs FPro, ingredient features
│   ├── 02_Nutrition_FPro_Modeling.ipynb         # RF training, tuning, holdout evaluation, model export
│   ├── 03_CIQUAL_Environmental_Analysis.ipynb   # CIQUAL transfer + Agribalyse environmental analysis
│   ├── presentation.pptx                        # 5-minute final presentation (6 slides)
│   ├── nutrition_fpro_presentation.md           # Slide notes / speaker outline
│   └── models/
│       ├── fpro_nutrition_models.joblib         # 12-feature model
│       └── fpro_nutrition_models_10feat.joblib  # 10-feature portable model (used for CIQUAL)
│
├── Ciqual data/
│   ├── Table Ciqual 2025_ENG_2025_11_03.xlsx   # 3,484 French reference foods
│   └── agribalyse-31-detail-par-ingredient.csv # Agribalyse 3.1 (semicolon-separated)
│
├── GroceryDB-main/
│   └── data/
│       ├── GroceryDB_foods.csv          # 50,468 US products with FPro scores and nutrients
│       ├── GroceryDB_data_uncurated.csv # 54,000 products with raw ingredient lists
│       └── GroceryDB_IgFPro.csv         # 10,361 per-ingredient FPro scores
│
├── preliminary_analysis.py        # Phase 1: CIQUAL nutrient density regression
├── EXECUTIVE_SUMMARY.md           # One-page project summary
└── README.md                      # This file
```

---

## Notebooks (run in order)

### Notebook 01 — EDA & Ingredient Analysis
`files FPro analysis/01_FPro_EDA_Ingredient_Analysis.ipynb`

- Merges GroceryDB products with ingredient lists and per-ingredient FPro scores
- Explores price vs FPro correlation (r = −0.245)
- Weighted ingredient FPro vs product FPro correlation (r = 0.656)
- Builds binary and rank-regression ingredient feature matrices (top-500 ingredients)

### Notebook 02 — Nutrition FPro Modeling
`files FPro analysis/02_Nutrition_FPro_Modeling.ipynb`

- Trains Random Forest regression and classification models on 12 log-transformed nutrients
- 5-fold cross-validation, hyperparameter tuning, final holdout evaluation
- Feature importance + backward elimination → 10-feature portable model
- Exports models to `models/fpro_nutrition_models_10feat.joblib`

### Notebook 03 — CIQUAL + Environmental Analysis
`files FPro analysis/03_CIQUAL_Environmental_Analysis.ipynb`

- Parses and cleans CIQUAL 2025 (French decimal format, trace values, inequalities)
- Applies the 10-feature model to CIQUAL foods (no retraining, no imputation)
- Analyses predicted processing levels by food sub-group (alim_ssssgrp_nom_eng)
- Merges Agribalyse 3.1 life-cycle assessment data; computes FPro–environment correlations
- Key finding: Processed class (Class 2) has the highest environmental footprint due to processed animal products

---

## Data Sources

| Dataset | Description | Size |
|---|---|---|
| [GroceryDB](https://github.com/ChaosStates/GroceryDB) | US grocery products with FPro scores | 50,468 products |
| [CIQUAL 2025](https://ciqual.anses.fr/) | French food composition reference | 3,484 foods |
| [Agribalyse 3.1](https://agribalyse.ademe.fr/) | French LCA environmental data | 6,161 ingredient rows |
| [FoodProX / FoodProX paper](https://doi.org/10.1016/j.celrep.2022.110345) | FPro score source | — |

---

## Quick Start

```bash
# 1. Install dependencies
pip install pandas numpy scikit-learn matplotlib seaborn scipy openpyxl joblib python-pptx

# 2. Launch Jupyter for the FPro analysis notebooks
jupyter notebook "files FPro analysis/"

# 3. Run notebooks in order: 01 → 02 → 03
#    NB02 must complete before NB03 (it exports the model file)
```

**Note:** NB02 Section 11 must be run before NB03 to produce `fpro_nutrition_models_10feat.joblib`.

---

## Technical Details

### FPro Score
- **Source:** FoodProX model (Meziani et al., *Cell Reports* 2022)
- **Range:** 0 (unprocessed) → 1 (ultra-processed)
- **Classes:** 0 Unprocessed | 1 Minimally processed | 2 Processed | 3 Ultra-processed

### Feature Engineering
- **Log-transform:** `log(x)` if x > 0 else −20 (matches FoodProX convention)
- **10 features used for CIQUAL:** Protein, TotalFat, Carbs, Sugars, Fiber, Calcium, Iron, Sodium, Cholesterol, SatFat
- **VitaminA and VitaminC dropped** for portability: 55% and 69% CIQUAL coverage respectively; missing ≠ zero in food databases

### Model
- **Algorithm:** Random Forest (scikit-learn)
- **Train/test split:** 80/20 stratified by FPro class
- **Cross-validation:** 5-fold stratified CV for classification; 5-fold KFold for regression
- **Class imbalance:** `class_weight='balanced'` for classifiers (Class 3 = 73% of data)

---

## Limitations

- FPro is itself derived from the same 12 nutrients in FoodProX, so high R² is expected (approximating an existing scoring rule, not discovering novel structure)
- Agribalyse covers only ~29% of CIQUAL foods; environmental conclusions are directional
- Domain shift: US model may over-classify French reference foods as ultra-processed
- Ingredient-based features (NB01) have lower predictive power than nutrition-based features (NB02)

---

*Erdős Institute Data Science Boot Camp 2026 — Final Project*
