# Liver Cancer Prediction

*Machine learning–based risk stratification for hepatocellular carcinoma from clinical laboratory data.*

---

## 1. Clinical context

**Hepatocellular carcinoma (HCC)** is the most common primary liver malignancy and the third leading cause of cancer-related death worldwide, with approximately 830,000 deaths per year (GLOBOCAN 2020). The five-year survival rate for early-stage HCC (BCLC 0/A) reaches 70 % with surgical resection or ablation; for late-stage disease it falls below 10 %. The survival gap is almost entirely attributable to stage at diagnosis — making early, accurate risk stratification a high-impact clinical problem.

Current screening guidelines recommend biannual ultrasound plus AFP (alpha-fetoprotein) for high-risk patients (cirrhosis, chronic HBV/HCV). However, AFP has a sensitivity of only 40–65 % for early HCC. There is a clear need for supplementary models that integrate multiple biomarkers.

---

## 2. Dataset & features

The project uses a structured clinical dataset with laboratory and demographic features:

| Feature category | Variables |
|---|---|
| Liver function | ALT, AST, ALP, GGT, bilirubin (total/direct), albumin |
| Synthetic function | PT, INR, fibrinogen |
| Tumour markers | AFP, AFP-L3, PIVKA-II (DCP) |
| Viral serology | HBsAg, anti-HCV |
| Imaging / staging | tumour size, vascular invasion (where available) |
| Demographics | age, sex, BMI, alcohol history |

Class label: **HCC present / absent** (binary classification). The dataset is imbalanced (HCC prevalence ~10–15 % in high-risk cohorts), requiring class-weight adjustment or oversampling.

---

## 3. Pipeline

### 3.1 Exploratory data analysis

- Distribution plots (histograms + KDE) per feature, split by label
- Pearson / Spearman correlation heatmap to detect collinear biomarkers (ALT–AST, PT–INR)
- Missing-value audit — MCAR vs. MNAR analysis; imputation strategy (median for continuous, mode for categorical)

### 3.2 Preprocessing

```python
# Imputation → scaling → encoding
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler
from sklearn.impute import SimpleImputer

prep = Pipeline([
    ('imputer', SimpleImputer(strategy='median')),
    ('scaler',  StandardScaler()),
])
```

Log-transformation is applied to AFP (right-skewed, span of 4–5 decades).

### 3.3 Feature selection

1. **Univariate screening** — Mann-Whitney U test (non-parametric) with Benjamini–Hochberg FDR correction
2. **Recursive Feature Elimination** (RFE) with cross-validated estimator
3. **SHAP values** — post-hoc Shapley-based importance for the final model

### 3.4 Models compared

| Model | Notes |
|---|---|
| Logistic Regression | L2-regularized; interpretable baseline |
| Random Forest | Ensemble; handles interactions and non-linearity |
| Gradient Boosting (XGBoost) | State-of-the-art for tabular clinical data |
| SVM (RBF kernel) | Effective on small datasets with high-dim features |

Hyper-parameters tuned via **stratified 5-fold cross-validation** with `GridSearchCV`.

### 3.5 Evaluation

Because the clinical cost of a false negative (missed cancer) far exceeds a false positive (unnecessary biopsy), the primary metric is **AUROC** with supplementary reporting of:
- Sensitivity, specificity at the operating point (e.g. 90 % sensitivity)
- Precision-recall AUC (more informative than AUROC under class imbalance)
- Calibration (Brier score, reliability diagram)

---

## 4. Key findings

- **AFP + PIVKA-II + age** together dominate feature importance across all models, consistent with published multi-marker panels (Johnson et al. 2020)
- Gradient Boosting achieves the highest AUROC; Logistic Regression offers competitive sensitivity with better calibration
- Class weighting (`class_weight='balanced'`) recovers ~8% sensitivity vs. training on raw imbalanced data

---

## 5. Limitations

- Dataset size limits statistical power for rare subgroups (HCC in non-cirrhotic patients)
- Models trained on a single cohort may not generalize across ethnicities or HBV vs. HCV-dominant populations (distribution shift)
- No temporal/longitudinal features — surveillance trajectories (rising AFP trend) are discarded by the cross-sectional design

---

## 6. Stack

- Python · scikit-learn · XGBoost · SHAP
- Pandas · NumPy · Matplotlib · Seaborn
- Jupyter Notebooks

---

## 7. Run

```bash
pip install scikit-learn xgboost shap pandas numpy matplotlib seaborn jupyter
jupyter notebook
```

Open the notebook in the repository root and run all cells sequentially.

---

## 8. References

1. GLOBOCAN 2020. *Global Cancer Observatory.* IARC, 2021.
2. P. J. Johnson et al. "Assessment of liver function in patients with hepatocellular carcinoma." *Hepatology*, 72(3):980–998, 2020.
3. EASL. "EASL Clinical Practice Guidelines: Management of hepatocellular carcinoma." *J. Hepatol.*, 69(1):182–236, 2018.
