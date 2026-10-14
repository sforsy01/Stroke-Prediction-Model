# Stroke Prediction: A Stacked Ensemble Model

An R Markdown analysis that builds and compares classification models to predict stroke risk from routinely collected patient information. Three base models (bagged elastic-net logistic regression, balanced random forest, and XGBoost) are tuned for F1 and combined into a stacked ensemble, then evaluated on a held-out test set using F1 and AUC.

This was a Fall 2025 project for the MS in Bioinformatics program at Northeastern University.

## Data

[Stroke Prediction Dataset](https://www.kaggle.com/datasets/fedesoriano/stroke-prediction-dataset/data) by fedesoriano on Kaggle: 5,110 patients, with 249 stroke cases (about 4.9%). Predictors are gender, age, hypertension, heart disease, marital status, work type, residence type, average glucose level, BMI, and smoking status.

The data file is not included in this repository. The analysis downloads it automatically with the `kagglehub` Python package. Please follow the dataset's license and terms on Kaggle.

## Methods

- **Cleaning:** one patient with gender "Other" was set to the most common category. Missing BMI values were imputed with a linear model, chosen over median and random forest imputation by MAE and RMSE.
- **Exploration:** bar charts and histograms, Shapiro-Wilk tests, log transformation of skewed numeric variables, a correlation check, and PCA (exploratory only).
- **Splitting:** stratified 64% train, 16% validation, and 20% test. The test set is used only for final evaluation.
- **Class imbalance:** class weights for the GLM and XGBoost, and balanced sampling for the random forest.
- **Base models:**
  - Bagged elastic-net logistic regression (`glmnet`, alpha = 0.5, 10 bootstrap bags)
  - Balanced random forest (`randomForest`), with `mtry` and `ntree` tuned by cross-validated F1
  - XGBoost (`xgboost` through `caret`), with a small grid tuned for F1
- **Thresholds:** each model's classification threshold was chosen on the validation set to maximize F1.
- **Stacking:** out-of-fold predictions from the three models on the combined train and validation sets are blended by a ridge logistic regression meta-learner, with its penalty and threshold tuned for F1.
- **Evaluation:** F1 and AUC on the held-out test set for every base model and the ensemble.

## Findings

The analysis compares the stacked ensemble against its base models and recommends XGBoost alone, because the ensemble did not improve AUC and was not better in F1 while adding complexity. The exact test-set numbers are produced when the document is knit; see `stroke_prediction.html` for the full results and discussion.

## How to run

### Requirements

- R (4.x recommended) and RStudio, or any environment that can run `rmarkdown`
- Python 3 with the `kagglehub` package
- R packages: `reticulate`, `tidyverse`, `psych`, `randomForest`, `caret`, `glmnet`, `xgboost`, `pROC`, `rmarkdown`

### Setup

```r
install.packages(c("reticulate", "tidyverse", "psych", "randomForest",
                   "caret", "glmnet", "xgboost", "pROC", "rmarkdown"))
```

```bash
pip install kagglehub
```

`reticulate` must be able to find the Python environment where `kagglehub` is installed. If it picks the wrong one, point it at the right one before knitting, for example `reticulate::use_python("/path/to/python")`. If the download fails, you may need to set up Kaggle credentials as described in the `kagglehub` documentation.

### Knit

```r
rmarkdown::render("stroke_prediction.Rmd")
```

Tuning and cross-validated stacking involve many model fits, so knitting can take a while. Random seeds are set throughout, but results can still differ slightly across package versions and platforms.

## Limitations

- BMI imputation, and the PCA, were done on the full data set before the train/validation/test split. The imputation model therefore saw test rows, which can leak a small amount of information into training. A stricter pipeline would fit the imputation on the training set only.
- Stroke is rare, so the test set contains few positive cases. F1 and AUC estimates are uncertain, and differences between models may be within noise.
- Results come from a single split and seed. Repeated cross-validation would give more stable comparisons.
- This is a coursework project using a public data set. It is not a validated clinical tool and should not be used for medical decisions.

## Files

- `stroke_prediction.Rmd`: full analysis and write-up
- `stroke_prediction.html`: knitted report (if included)

## Author

Sara Forsythe, MS Candidate in Bioinformatics, Northeastern University

## References

- Kuriakose, D., & Xiao, Z. (2020). Pathophysiology and Treatment of Stroke: Present Status and Future Perspectives. *International Journal of Molecular Sciences*, 21(20), 7609. https://doi.org/10.3390/ijms21207609
- fedesoriano. (2020). Stroke Prediction Dataset (Version 1) [Data set]. Kaggle.
