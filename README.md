# Preprocessing Impact on Classical ML for Phishing URL Detection

---

## 📄 Research Report

The full written report for this project is available in [`project_report.pdf`](./project_report.pdf).

> 📖 **ArXiv preprint:** _link coming soon_

---

## Overview

This project investigates how feature preprocessing affects classical machine learning models for phishing URL detection. We evaluate **five preprocessing strategies** against **five classifiers** on the [PhiUSIIL Phishing URL Dataset](https://archive.ics.uci.edu/dataset/967/phiusiil+phishing+url+dataset) (235,795 samples, 50 numerical features), using stratified 5-fold cross-validation, paired t-tests, and a controlled scale-distortion experiment.

The key finding: most models achieve near-perfect performance regardless of preprocessing — but **RBF-SVM is critically sensitive to feature scaling**, collapsing from ROC-AUC 0.997 to 0.532 (barely above random) under scale distortion, while any scaling method fully restores it.

---

## Table of Contents

- [Motivation](#motivation)
- [Dataset](#dataset)
- [Preprocessing Strategies](#preprocessing-strategies)
- [Models](#models)
- [Experimental Design](#experimental-design)
- [Key Results](#key-results)
- [Project Structure](#project-structure)
- [Getting Started](#getting-started)
- [Authors](#authors)

---

## Motivation

In the PhiUSIIL dataset, the variance ratio between the highest-variance feature (`LargestLineLength`, σ² ≈ 2.3 × 10¹⁰) and the lowest (`ObfuscationRatio`, σ² ≈ 1.5 × 10⁻⁵) exceeds **10¹⁵**. For any distance- or kernel-based model, one feature dominates everything else. This study asks: how much does this actually matter in practice, and for which models?

---

## Dataset

**PhiUSIIL Phishing URL Dataset** — Prasad & Chandra (2024), UCI Machine Learning Repository.

| Property | Value |
|---|---|
| Total samples | 235,795 |
| Features | 50 numerical (URL structure, lexical, webpage metadata) |
| Class balance | 57% legitimate, 43% phishing |
| Used subsample | 50,000 (stratified; full dataset exceeded 11 hrs for SVM) |

Non-numeric columns were excluded prior to experiments. A label-shuffle test (ROC-AUC = 0.499) confirmed no data leakage. Domain-aware group k-fold produced identical results, further validating the setup.

---

## Preprocessing Strategies

| Strategy | Description |
|---|---|
| **None** | Raw features — baseline |
| **Min-Max Scaling** | Rescales each feature to `[0, 1]` |
| **Standardisation** | Zero mean, unit variance (fit on training fold only) |
| **PCA** | Standardised + PCA retaining 95% variance (36/50 components) |
| **Whitening** | PCA + scale retained components to unit variance |

Min-max scaling and standardisation were also implemented from scratch and verified against scikit-learn (max absolute difference < 2.2 × 10⁻¹⁶).

---

## Models

| Model | Sensitivity to Scale |
|---|---|
| Logistic Regression | ❌ Not sensitive (weights absorb scale) |
| Linear SVM | ❌ Not sensitive |
| **RBF-SVM** | ✅ **Critically sensitive** |
| **k-NN** | ✅ Sensitive (metric-dependent) |
| Naïve Bayes | ⚠️ Harmed by PCA/whitening |

Hyperparameters were selected via exhaustive grid search with cross-validation on a 10,000-instance subsample, applied to the 50,000-instance evaluation set.

---

## Experimental Design

- **Evaluation:** Stratified 5-fold cross-validation; metrics: ROC-AUC, Balanced Accuracy, F1
- **Statistical testing:** Paired t-tests across fold-level results (α = 0.05)
- **Scale-distortion experiment:** Controlled artificial distortion of feature magnitudes to isolate the effect of scale on RBF-SVM
- **Feature importance analysis:** Examines how preprocessing changes which features appear most informative

---

## Key Results

### RBF-SVM

The most striking finding. Without any scaling, the RBF-SVM performs normally on clean data (ROC-AUC = 0.997), but drops to **0.532** under controlled scale distortion (p < 0.001) — barely above random chance. Any of the three scaling methods (min-max, standardisation, whitening) fully restores performance.

### k-NN

Standardisation produced a statistically significant improvement (p = 0.003). Min-max scaling did **not** help (p = 0.75). The reason: min-max compresses features with extreme outliers into a narrow range near zero, reducing their discriminative power under the Manhattan distance metric selected by grid search. Scaler choice is **not** interchangeable with distance metric choice.

### Naïve Bayes

PCA and whitening **harmed** Naïve Bayes (ROC-AUC dropped from 0.9999 to 0.9887, p < 0.001). PCA merges 50 features into 36 components, discarding per-feature information that Naïve Bayes models independently. More preprocessing is not always better.

### Logistic Regression & Linear SVM

Completely unaffected by preprocessing choice (p > 0.05 for all comparisons), consistent with theory — linear models absorb scale differences into their weight vectors during optimisation.

### Feature Importance

Even when preprocessing does not change predictive accuracy, it changes **which features appear important** — relevant for any downstream interpretation or decision-making.

### Full Results Summary

| Preprocessing | Model | ROC-AUC | Bal. Acc. | F1 |
|---|---|---|---|---|
| Standardisation | Logistic Regression | 1.0000 | 0.9999 | 0.9999 |
| Min-Max | Linear SVM | 1.0000 | 0.9999 | 0.9999 |
| Standardisation | RBF-SVM | 1.0000 | 0.9999 | 0.9999 |
| Min-Max | RBF-SVM | 1.0000 | 0.9998 | 0.9998 |
| PCA | RBF-SVM | 0.9999 | 0.9995 | 0.9996 |
| None | RBF-SVM | 0.9970 | 0.9653 | 0.9748 |
| Whitening | Naïve Bayes | 0.9887 | 0.9696 | 0.9728 |

_Full table of all 25 model–preprocessing combinations is in Appendix A of the report._

---

### Dataset

Download the PhiUSIIL dataset from the [UCI ML Repository](https://archive.ics.uci.edu/dataset/967/phiusiil+phishing+url+dataset).

---

## Authors

- **Baris Kaban** — Vrije Universiteit Amsterdam
- **Goktug Yildirim** — Vrije Universiteit Amsterdam
- **Mert Arslan** — Vrije Universiteit Amsterdam

---

## 📁 Repository Structure

```
Phishing-Url-Preprocessing-Study/
├── LICENSE     # MIT License
├── experiments.ipynb     # All preprocessing experiments, models and evaluations
├── project_report.pdf    # Full research paper
└── README.md             # Project overview and documentation
```

---

## Citation

If you use this work, please cite:

```bibtex
@article{kaban2026phishing,
  title   = {Impact of Feature Preprocessing on Classical Machine Learning Models for Phishing URL Detection},
  author  = {Kaban, Baris and Yildirim, Goktug and Arslan, Mert},
  year    = {2026},
  note    = {Vrije Universiteit Amsterdam}
}
```

> ArXiv preprint: _link coming soon_

## License

This project is licensed under the MIT License.

---

## References

- Prasad, A. & Chandra, S. (2024). PhiUSIIL: A diverse security profile empowered phishing URL detection framework. *Computers & Security*, 136, 103545.
- Singh, D. & Singh, B. (2020). Investigating the impact of data normalization on classification performance. *Applied Soft Computing*, 97, 105524.
- Fernandes et al. (2022). The choice of scaling technique matters for classification performance. *Applied Soft Computing*, 133, 109924.
- Ahsan et al. (2021). Effect of data scaling methods on machine learning algorithms. *Technologies*, 9(3), 52.
- Hsu, C.-W., Chang, C.-C., & Lin, C.-J. (2003). A practical guide to support vector classification. Technical report, NTU.
