# Executive Summary
## Predicting Food Processing and Environmental Impact
**Erdős Institute Data Science Boot Camp 2026 · March 2026**

---

### Project Questions

Can a machine-learning model trained only on standard nutrition-label data reproduce food-processing scores well enough to transfer to new national food databases? Does lower food processing imply a lower environmental footprint?

---

### Approach

We used **GroceryDB** (https://github.com/Barabasi-Lab/GroceryDB/tree/main), a dataset of 50,468 US grocery products that carries FPro scores (a continuous 0–1 food-processing index from the FoodProX model). After filtering to products with complete nutrition data and removing physically impossible values, we trained a Random Forest on **12 log-transformed nutrients** (protein, fat, carbohydrates, sugar, fiber, calcium, iron, sodium, cholesterol, saturated fat, vitamin A, vitamin C) to predict FPro as both a continuous score and a four-class label.

The trained model was then applied (without further retraining) to **CIQUAL 2025** (https://ciqual.anses.fr/#/cms/download/node/20), the French national food-composition reference (3,484 foods). Predicted FPro classes were merged with **Agribalyse 3.1** (https://data.ademe.fr/datasets?topics=TQJGtxm2_) life-cycle-assessment data to test whether processing level predicts environmental impact.

---

### Key Performance Indicators

| KPI | Target | Achieved |
|---|---|---|
| Regression R² on holdout | > 0.85 | **0.973** ✓ |
| Regression RMSE | < 0.10 | **0.041** ✓ |
| Classification Macro F1 | > 0.80 | **0.904** ✓ |
| Classification Accuracy | > 0.90 | **95.9 %** ✓ |
| Portable model (10 features) R² | > 0.90 | **0.974** ✓ |
| Portable model Macro F1 | > 0.80 | **0.891** ✓ |

---

### Principal Findings

**1. Nutrition labels closely reproduce FPro (R² = 0.973)**
The Random Forest achieves near-perfect reconstruction of the FPro score. This is expected because FPro is itself derived from these same nutrients inside FoodProX. The practical value is *portability*: any food dataset with a standard nutrition panel can now receive FPro-like scores without running the original FoodProX pipeline.

**2. A 10-feature model transfers to France without retraining**
Vitamin A (55% coverage in CIQUAL) and Vitamin C (69% coverage) were dropped from the feature set to avoid treating missing data as zero. The 10-feature model retains essentially the same performance (R² 0.974, Macro F1 0.891) and successfully classifies CIQUAL foods into interpretable processing tiers — raw vegetables score low; instant meals and soft drinks score high.

**3. The majority of CIQUAL foods are predicted as Ultra-Processed (Class 3)**
This result reflects the real composition of the CIQUAL catalogue (which includes many prepared dishes, cereals, and beverages), amplified by domain shift (the US grocery market is 73% ultra-processed, biasing the model toward higher classes). Food-group breakdowns confirm that the predictions are directionally sensible.

**4. Lower processing does NOT automatically mean lower environmental impact**
Among the 659 CIQUAL foods matched to Agribalyse data, correlation between predicted FPro and all 14 environmental metrics is near zero. Strikingly, **Class 2 "Processed" foods have the highest mean environmental footprint** (Environmental Footprint score = 1.11; climate change = 9.03 kg CO₂-eq/kg), exceeding even ultra-processed Class 3 (EF = 0.43). The explanation is food type, not processing level: processed animal products — cheese, cured meats, smoked fish — combine a high livestock footprint with industrial processing, while ultra-processed foods are predominantly grain-, sugar-, and oil-based with lower per-kg impact.

---

### Implications

- **Nutrition and sustainability are separate axes.** A food can be lightly processed yet environmentally intensive (e.g., aged cheese) or highly processed yet relatively low-impact (e.g., breakfast cereal). Policymakers and consumers should evaluate both dimensions independently.
- **Portable FPro approximation.** The 10-feature model can extend FPro-style scores to any dataset with a standard nutrition label, including national food-composition databases outside the US.
- **Agribalyse coverage limits conclusions.** Only ~29% of CIQUAL foods had Agribalyse matches; environmental findings are directional rather than definitive.

---

### Notebooks and Outputs

| File | Contents |
|---|---|
| `files FPro analysis/01_FPro_EDA_Ingredient_Analysis.ipynb` | EDA, price analysis, ingredient feature engineering |
| `files FPro analysis/02_Nutrition_FPro_Modeling.ipynb` | Model training, tuning, feature elimination, export |
| `files FPro analysis/03_CIQUAL_Environmental_Analysis.ipynb` | CIQUAL transfer, Agribalyse merge, environmental correlations |
| `files FPro analysis/models/fpro_nutrition_models_10feat.joblib` | Saved 10-feature Random Forest (regression + classification) |
| `files FPro analysis/presentation.pptx` | 5-minute final presentation (6 slides) |

---

*Erdős Institute Data Science Boot Camp 2026*
