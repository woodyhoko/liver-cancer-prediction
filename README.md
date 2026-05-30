# Liver Cancer Prediction

*Exploring whether liver-cancer patient survival can be linked to tumour gene (mRNA) expression, using the public TCGA-LIHC cohort.*

---

> **Note on data & scope.** Because of the privacy and usage policies that govern medical data, this repository does **not** redistribute or describe any private/identifiable patient information, and only limited detail is shared here. The analysis is built entirely on the **publicly released, de-identified TCGA Liver Hepatocellular Carcinoma (LIHC)** study as published on [cBioPortal](https://www.cbioportal.org/). Nothing here is a clinical tool or medical advice.

---

## 1. What this project does

Hepatocellular carcinoma (HCC) is the most common primary liver cancer, and survival varies enormously between patients. This project asks a focused question on the public TCGA-LIHC cohort:

**Is a patient's overall survival associated with the expression levels of particular genes (mRNA) in their tumour — and can we predict survival outcome from expression?**

The work has two parts:

1. **mRNA ↔ survival correlation analysis** — for each gene, test whether its tumour mRNA expression differs between patients who survived and patients who did not.
2. **Survival outcome prediction** — frame survival as a binary outcome and measure how well it can be predicted.

---

## 2. Dataset

The TCGA-LIHC study download (cBioPortal format) under `lihc_tcga/`, including:

| File | Contents |
|---|---|
| `data_RNA_Seq_v2_expression_median.txt` | Per-gene RNA-Seq (mRNA) expression, one column per tumour sample |
| `data_RNA_Seq_v2_mRNA_median_Zscores.txt` | The same expression as z-scores |
| `data_bcr_clinical_data_patient.txt` | De-identified clinical table (~377 patients) including `Overall Survival Status`, `Overall Survival (Months)`, stage, etc. |
| `data_CNA.txt`, `data_methylation_*`, `data_mutations_*` | Additional public molecular layers (not central to this analysis) |

Patients are matched to their tumour expression column by their TCGA identifier (e.g. `TCGA-2V-A95S` → `…-01` primary-tumour sample). A handful of patients have no matching expression column and are skipped.

---

## 3. Method

The analysis lives in `lihc_tcga/Untitled.ipynb` (pandas + statsmodels) and is intentionally simple:

1. **Load** the median RNA-Seq expression matrix and the clinical patient table.
2. **Split patients by outcome** using `Overall Survival Status` — `DECEASED` vs. `LIVING`.
3. **Pull each group's expression vectors** for the matched tumour samples.
4. **Per-gene two-sample z-test** (`statsmodels … ztest`) comparing the deceased group against the living group, scanning across genes to find the mRNA whose expression differs most between the two survival groups.

Genes with near-constant expression produce undefined statistics (division by zero) and are ignored.

> This is an **exploratory, simplified** screen — a first pass to surface candidate survival-associated genes, not a multiple-testing-corrected biomarker discovery pipeline.

---

## 4. Result

Framed as a binary task — **does the patient survive beyond 5 years (yes / no)** — the analysis reached roughly **≈ 70 % accuracy** at separating the two outcomes (i.e. predicting whether 5-year survival is above ~50 % likelihood for a patient versus not). Given the cohort size and the deliberately simple approach, this is treated as a directional finding that there *is* mRNA signal related to survival, rather than a validated clinical predictor.

---

## 5. Limitations

- **Exploratory statistics.** The per-gene z-test screen does not apply multiple-hypothesis correction; reported gene-level differences should be regarded as candidates only.
- **Single cohort, modest size.** ~377 patients from one public study; no external validation, so findings may not generalize.
- **Cross-sectional.** Survival *time* and censoring are simplified into a binary outcome; a proper survival analysis would use Cox/Kaplan–Meier with right-censoring.
- **Not a clinical tool.** See the data/scope note at the top.

---

## 6. Stack

- Python · pandas · statsmodels (`ztest`)
- Jupyter Notebook
- Public TCGA-LIHC data via cBioPortal

---

## 7. Run

```bash
pip install pandas statsmodels jupyter
cd lihc_tcga
jupyter notebook Untitled.ipynb
```

Run the cells top to bottom; they load the expression and clinical tables, split patients by survival status, and run the per-gene comparison.

---

## 8. References

1. The Cancer Genome Atlas (TCGA) Research Network — Liver Hepatocellular Carcinoma (LIHC) study.
2. cBioPortal for Cancer Genomics — <https://www.cbioportal.org/>.
