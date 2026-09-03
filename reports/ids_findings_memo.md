# Network Intrusion Detection — Model Evaluation Memo

**Dataset:** NSL-KDD benchmark (Tavallaee et al., 2009) — real, published network traffic data
**Training set:** 125,973 labeled connection records, 22 attack types + normal
**Test set:** 22,544 labeled connection records, 39 attack types + normal (17 never seen in training)

---

## 1. Executive Summary

We trained and compared three classifiers (Logistic Regression, Decision Tree, Random Forest) to detect and categorize network intrusions into four attack families — DoS, Probe, R2L (remote-to-local), and U2R (user-to-root) — versus normal traffic.

**Logistic Regression was the best-performing model** (macro F1 = 0.610), not because it was the most accurate on training-distribution attacks, but because it generalized meaningfully better to attack types the model had never encountered during training — a critical property for real-world intrusion detection, where new attack variants appear constantly.

## 2. Model Comparison

| Model | Macro F1 | Weighted F1 | Novel-Attack Detection Rate |
|---|---|---|---|
| **Logistic Regression** | **0.610** | 0.782 | **43.4%** |
| Decision Tree | 0.502 | 0.712 | 42.8% |
| Random Forest | 0.473 | 0.690 | **14.6%** |

**Key finding:** Random Forest — often assumed to be the strongest default choice — had the *worst* detection rate on attack types it had never seen during training (14.6%, vs. 43.4% for Logistic Regression). This indicates Random Forest overfit to the specific statistical signatures of the 22 training-set attack types rather than learning generalizable indicators of malicious behavior. For a production intrusion detection system, this matters more than raw training accuracy: attackers do not repeat identical patterns, and a model that cannot recognize novel attack behavior provides limited real-world security value.

## 3. Per-Class Performance (Logistic Regression, best model)

| Attack Category | Precision | Recall | F1 | Test Support |
|---|---|---|---|---|
| DoS | 0.98 | 0.78 | 0.87 | 7,460 |
| Probe | 0.75 | 0.88 | 0.81 | 2,421 |
| R2L | 0.87 | 0.26 | 0.40 | 2,885 |
| U2R | 0.08 | 0.58 | 0.15 | 67 |
| Normal | 0.74 | 0.94 | 0.83 | 9,711 |

**DoS and Probe detection is strong** — these attacks involve high-volume or high-frequency connection patterns that are statistically distinct from normal traffic, and the model captures this reliably (F1 0.87 and 0.81 respectively).

**R2L and U2R detection is weak** — this is a known, well-documented limitation of the NSL-KDD benchmark itself, not a modeling error. These attack types are behaviorally subtle (e.g., a single guessed password, a single privilege escalation) and occur too rarely in the training data (U2R: only 52 of 125,973 training records) for any model to learn robust patterns. This matches published NSL-KDD literature — no published model achieves strong R2L/U2R performance on this dataset without additional feature engineering specific to those attack types.

## 4. Novel Attack Type Analysis

17 of the 39 attack types in the test set never appeared during training (e.g., `apache2`, `mscan`, `httptunnel`, `worm`, `sqlattack`). This is intentional in the NSL-KDD test design — it simulates the real-world scenario where a deployed system encounters attacks it was never explicitly trained on.

The best model detected 43.4% of these novel attacks as "not normal" (correctly flagging them as suspicious, even without correctly categorizing the specific attack type). This is the single most important number in this evaluation for a real security context: it directly answers "how well does this system protect against attacks we haven't seen before?"

## 5. Explainability

Feature importance (via Random Forest, used here purely for interpretability alongside the deployed Logistic Regression model) shows the top predictive signals are `src_bytes`, `dst_host_srv_count`, `dst_bytes`, `service`, and `logged_in` — all directly interpretable as connection-volume and service-access patterns, which aligns with domain expectations for network-traffic-based intrusion detection.

Logistic Regression coefficients by class show the model relies on intuitive signals: DoS detection is driven by `hot` (suspicious command indicators) and connection error rates; U2R detection is driven by `num_compromised` and `num_file_creations` (privilege-escalation indicators). This gives a security analyst a clear, explainable basis for any flagged event.

## 6. Recommendations

1. **Deploy Logistic Regression, not Random Forest**, despite Random Forest's higher apparent complexity — its poor novel-attack generalization makes it a weaker choice for production security monitoring.
2. **Treat R2L/U2R alerts as a separate, lower-confidence tier** — given the low precision/recall on these categories, route them for manual analyst review rather than automated action.
3. **Retrain on a rolling window of recent traffic** to reduce the novel-attack gap — the 43.4% novel-detection rate, while better than alternatives, still means over half of genuinely new attack types would be missed.
4. **Combine with rule-based signature detection** for known attack types (where this model already performs well, e.g., DoS at F1 0.87) and reserve the ML layer specifically for anomaly flagging on traffic that doesn't match known signatures.

## 7. Limitations

- NSL-KDD was collected in a research testbed, not live production traffic — feature distributions may not perfectly match a specific organization's network.
- The dataset dates to 2009; attack techniques have evolved substantially since. A production system would need retraining on recent traffic.
- R2L and U2R performance is genuinely weak in absolute terms — this evaluation is transparent about that rather than only reporting favorable aggregate metrics.

## 8. Extension Experiment — XGBoost & SMOTE

Two follow-up techniques were tested to see if they could beat the Logistic Regression baseline.

| Model | Macro F1 | Novel-Attack Detection |
|---|---|---|
| **Logistic Regression (baseline)** | **0.610** | 43.4% |
| Logistic Regression + SMOTE | 0.607 | 45.6% |
| XGBoost | 0.534 | 36.9% |
| Decision Tree (baseline) | 0.502 | 42.8% |
| Random Forest (baseline) | 0.473 | 14.6% |

**XGBoost** partially avoided Random Forest's overfitting problem (36.9% vs. 14.6% novel-attack detection) but still underperformed the simpler Decision Tree and Logistic Regression models — reinforcing that additional tree-ensemble complexity does not translate into better generalization on this dataset.

**SMOTE**, applied to address the severe U2R class imbalance (52 real training examples), did not help and for U2R made results worse: recall dropped from 0.58 to 0.45 despite training on tens of thousands of synthetic U2R examples. R2L saw only a marginal improvement (recall 0.26 → 0.28). The likely explanation: synthetic interpolation from just 52 real points creates narrow, tightly-clustered synthetic examples that don't capture genuine attack diversity, and can make the model overconfident about an unrepresentative pattern.

**Conclusion:** neither technique beat the original Logistic Regression baseline. This is a legitimate, useful negative result — it demonstrates that "add more sophistication" and "rebalance the rare classes" are not universal fixes, and that with only 52 real examples of a class, no resampling technique can manufacture information that was never collected in the first place.
