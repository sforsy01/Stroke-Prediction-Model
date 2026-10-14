# Stroke-Prediction-Model
An R Markdown analysis that builds and compares classification models to predict stroke risk from routinely collected patient information. Three base models (bagged elastic-net logistic regression, balanced random forest, and XGBoost) are tuned for F1 and combined into a stacked ensemble, then evaluated on a held-out test set using F1 and AUC.
