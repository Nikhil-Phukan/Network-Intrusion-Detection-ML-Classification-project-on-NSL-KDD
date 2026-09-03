# Extension Experiment — XGBoost & SMOTE

**Follow-up to the main intrusion detection evaluation.** Tests whether a more powerful model (XGBoost) or a class-imbalance technique (SMOTE) can beat the original best model (Logistic Regression, macro F1 = 0.610, 43.4% novel-attack detection).

---

## 1. Does XGBoost repeat Random Forest's overfitting mistake?

| Model | Macro F1 | Novel-Attack Detection Rate |
|---|---|---|
| Logistic Regression (baseline) | 0.610 | 43.4% |
| Decision Tree (baseline) | 0.502 | 42.8% |
| Random Forest (baseline) | 0.473 | 14.6% |
| **XGBoost** | **0.534** | **36.9%** |

**Finding: partially.** XGBoost's novel-attack detection (36.9%) is far better than Random Forest's (14.6%), showing that boosting — where each tree corrects the previous tree's mistakes — generalizes better than bagging (Random Forest's independent-trees approach) on this dataset. However, XGBoost still falls short of both simpler models (Decision Tree 42.8%, Logistic Regression 43.4%).

**Conclusion:** additional tree-ensemble sophistication does not translate into better generalization to unseen attacks on NSL-KDD. The original finding — simpler models generalize better here — holds even against a stronger, more modern algorithm.

## 2. Does SMOTE help the rare classes it targets (U2R, R2L)?

| Class | Metric | Baseline LR | LR + SMOTE | Change |
|---|---|---|---|---|
| U2R | Precision | 0.08 | 0.08 | No change |
| U2R | Recall | 0.58 | 0.45 | **Worse** |
| U2R | F1 | 0.15 | 0.14 | Slightly worse |
| R2L | Precision | 0.87 | 0.87 | No change |
| R2L | Recall | 0.26 | 0.28 | Marginal improvement |
| R2L | F1 | 0.40 | 0.42 | Marginal improvement |

**Finding: no — and for U2R, SMOTE made things worse.** U2R recall dropped from 0.58 to 0.45, meaning the SMOTE-trained model correctly identified *fewer* real U2R attacks than the baseline, despite training on tens of thousands of synthetic U2R examples generated from just 52 real ones. R2L saw a small, likely-insignificant improvement.

**Why this likely happened:** with only 52 real U2R examples, SMOTE's interpolation creates synthetic points tightly clustered around those few examples rather than capturing genuine attack diversity. This can make the model overconfident about a narrow, synthetic notion of "what U2R looks like" — at the cost of recognizing real U2R attacks that don't match that narrow pattern.

## 3. Full comparison — all 5 models

| Model | Macro F1 | Novel-Attack Detection |
|---|---|---|
| **Logistic Regression (baseline)** | **0.610** | 43.4% |
| Logistic Regression + SMOTE | 0.607 | 45.6% |
| XGBoost | 0.534 | 36.9% |
| Decision Tree (baseline) | 0.502 | 42.8% |
| Random Forest (baseline) | 0.473 | 14.6% |

Note: SMOTE's small bump in novel-attack detection (43.4% → 45.6%) is likely a side effect of the model becoming generally less biased toward predicting "normal" after rebalancing — not a direct benefit of the synthetic examples, since SMOTE only adds examples of attack types already present in training and does nothing to teach the model about the 17 genuinely novel attack types.

## 4. Overall conclusion

**Logistic Regression (the original baseline) remains the best model overall.** Neither added complexity (XGBoost) nor a targeted imbalance-correction technique (SMOTE) beat it, and SMOTE actively regressed on the exact class (U2R) it was intended to help.

The broader lesson: **class-imbalance techniques cannot manufacture information that isn't in the data.** With only 52 real U2R training examples, no amount of synthetic oversampling can teach a model genuine U2R attack diversity — it can only multiply and recombine what little is already known, in a narrow way that doesn't clearly generalize.

This is a legitimate, defensible conclusion for a real project write-up — it demonstrates rigorous testing of "obvious" improvements rather than assuming they would work, and reports the negative result honestly rather than omitting it.
