# Network Intrusion Detection — ML Classification on NSL-KDD

An intrusion detection system (IDS) built on the real NSL-KDD benchmark dataset, comparing multiple ML models on their ability to detect network attacks — including attack types never seen during training.

**Live dashboard:** _[add your Streamlit Cloud URL here after deploying](https://network-intrusion-detection-ml-classification-project-on-nsl-k.streamlit.app/)_

## What this project does

1. Loads the real NSL-KDD dataset (125,973 training / 22,544 test network connection records)
2. Groups 39 specific attack labels into 4 standard attack categories (DoS, Probe, R2L, U2R)
3. Trains and compares 5 models: Logistic Regression, Decision Tree, Random Forest, XGBoost, and Logistic Regression + SMOTE
4. Evaluates each model specifically on **attack types never seen during training** — the real test of whether a model generalizes or just memorizes
5. Delivers results via an interactive dashboard, a written evaluation report, and explainability analysis

## Key finding

**The simplest model won.** Logistic Regression (macro F1 = 0.610) outperformed every other model — including more complex ones like Random Forest and XGBoost — on the metric that matters most for real-world security: detecting attack types it had never trained on (43.4% detection rate vs. Random Forest's 14.6%).

Two follow-up experiments confirmed this wasn't a fluke:
- **XGBoost** partially avoided Random Forest's overfitting (36.9% novel-attack detection) but still underperformed the simpler baseline models.
- **SMOTE** (used to address severe class imbalance — only 52 training examples of the rarest attack type) did not help, and for that rare class actually made recall worse (0.58 → 0.45).

**Takeaway:** model complexity and imbalance-correction techniques are not universal fixes. On this dataset, generalization to unseen attacks mattered more than raw training accuracy — and the simplest model generalized best.

## Project structure

```
dashboard.py                              # interactive Streamlit dashboard
requirements.txt                          # dependencies for local run + deployment

notebooks/
  01_data_exploration.ipynb               # load data, inspect, find novel attack types
  02_preprocessing.ipynb                  # attack category mapping, categorical encoding
  03_model_training.ipynb                 # train & compare 3 baseline models
  04_evaluation_explainability.ipynb      # confusion matrix, feature importance
  05_xgboost_smote_extension.ipynb        # XGBoost + SMOTE follow-up experiment

data/
  KDDTrain+.txt, KDDTest+.txt             # raw NSL-KDD files (real, public benchmark data)
  train_processed.csv, test_processed.csv # cleaned/encoded data (output of notebook 2)
  model_comparison.csv                    # baseline model results (output of notebook 3)
  confusion_matrix.csv                    # best model's confusion matrix
  feature_importance.csv                  # Random Forest feature importances
  test_predictions.csv                    # per-record predictions for dashboard use
  final_model_comparison_with_extensions.csv  # all 5 models compared (output of notebook 5)

reports/
  ids_findings_memo.md                    # full written evaluation report
  ids_extension_findings.md               # XGBoost/SMOTE experiment deep-dive
  data_dictionary.md                      # column-level documentation
```

## How to run it yourself

```bash
pip install -r requirements.txt

# run notebooks in order (each saves CSVs the next one needs)
jupyter lab
# open and run 01 → 02 → 03 → 04 → 05 in sequence

# then launch the dashboard
streamlit run dashboard.py
```

## Dataset

[NSL-KDD](https://www.unb.ca/cic/datasets/nsl.html) (Tavallaee et al., 2009) — a refined version of the KDD Cup 1999 dataset, addressing known redundancy issues in the original. A real, published benchmark widely used in network intrusion detection research. Notably, the test set contains 17 attack types absent from the training set — an intentional design choice that tests genuine generalization rather than memorization.

## Model comparison

| Model | Macro F1 | Novel-Attack Detection Rate |
|---|---|---|
| **Logistic Regression** | **0.610** | **43.4%** |
| Logistic Regression + SMOTE | 0.607 | 45.6% |
| XGBoost | 0.534 | 36.9% |
| Decision Tree | 0.502 | 42.8% |
| Random Forest | 0.473 | 14.6% |

Full analysis, per-class breakdowns, and reasoning in `reports/ids_findings_memo.md`.

## Limitations

- NSL-KDD was collected in a research testbed, not live production traffic.
- The dataset dates to 2009; real-world attack techniques have evolved since.
- R2L and U2R attack categories are inherently hard to detect on this dataset due to severe class imbalance (as few as 52 training examples) — a known, documented limitation, not a modeling error.
