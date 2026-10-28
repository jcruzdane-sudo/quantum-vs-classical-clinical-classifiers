# Quantum Kernel versus Classical Classifiers for Clinical Malignancy Prediction

> **Disclaimer:** Educational research prototype. Not a medical device. Not for clinical use.

---

## Abstract

We compared a 4-qubit quantum kernel support vector machine (SVM), simulated exactly without hardware noise, with classical classifiers on the **Breast Cancer Wisconsin (Diagnostic)** dataset ($569$ patients, $30$ numeric features, $212$ malignant cases). We used five repeated stratified $70/30$ splits, training-only preprocessing and feature selection, cross-validated hyperparameters, and four training-set sizes ($10\%$, $25\%$, $50\%$, and $100\%$). 

The best classical model (logistic regression on all 30 features) reached AUPRC $0.991 \pm 0.005$ on the held-out test sets. The quantum model reached $0.932 \pm 0.013$ at full training size. It was also below a classical RBF SVM given the same four features ($0.978 \pm 0.008$). The quantum model did not outperform classical methods in any condition tested. Because this is a simulation on a small, well-studied dataset, the results do not establish quantum advantage.

> **Dataset Selection Note:** The original target was diabetic retinopathy using the Dryad dataset (`doi:10.5061/dryad.jwstqjqrm`). We could not download that dataset as the sandbox proxy refused the connection to `datadryad.org` (HTTP 403). We therefore switched to a clinical dataset bundled inside `scikit-learn` that works offline. No diabetic retinopathy results are reported here.

---

## 1. Clinical Problem and Why It Matters

Clinicians often use numerical measurements from a biopsy or scan to judge whether a tumour is malignant (cancerous). Missing a malignant case (a false negative) can delay treatment, so sensitivity matters. A model that simply predicts the common class is useless, which is why we report precision-recall measures and not accuracy alone. This project is a methods study. Its output is a classifier score, and it does not diagnose anyone or replace a clinician.

- **Dataset:** Breast Cancer Wisconsin (Diagnostic), bundled with `scikit-learn`. Derived from digitised images of fine-needle aspirate cell nuclei. Thirty features describe the mean, standard error, and worst value of radius, texture, perimeter, area, smoothness, and related shape measures. 
- **Class Labels:** Malignancy (positive class, $212$ cases) and Benign ($357$ cases).
- **Limitations of Dataset:** Small size ($569$ rows), single source, no demographic data (fairness across groups cannot be assessed), not representative of routine screening populations, contains already-curated image-derived features rather than raw records.

---

## 2. Research Question, Hypotheses and Outcomes

**Primary Question:** With controlled training-data sizes, can a 4-qubit quantum kernel SVM match or outperform strong classical classifiers in classifying malignant cases on held-out data?

* **H1:** The quantum kernel SVM matches or exceeds a classical RBF SVM given the same four features (primary outcome: AUPRC on test set, 100% training data).  
  * **Result: Not supported** ($0.932 \pm 0.013$ vs $0.978 \pm 0.008$).
* **H2:** The quantum model's advantage is largest when training data are scarce.  
  * **Result: Not supported** (At 10% training data: $0.737 \pm 0.118$ vs $0.946 \pm 0.047$ for classical SVM).
* **H3:** The quantum model stays competitive on ranking metrics (AUROC) while its operating point is less stable.  
  * **Result: Partly supported** (AUROC: $0.953 \pm 0.013$ vs $0.982 \pm 0.007$, but sensitivity was unstable at $0.731 \pm 0.227$).

---

## 3. Methods

- **Splits:** Five random seeds ($0$ to $4$), stratified $70/30$ split yielding $398$ training and $171$ test rows. Subsets at $10\%$, $25\%$, and $50\%$ drawn by stratified sampling from training rows only. Test set remained unchanged.
- **Data Leakage Control:** Standard scaler, ANOVA F-test top-$k$ feature selector, and quantum angle range fitted on training rows only. Test values outside training range clipped to $[0, \pi]$.
- **Classical Models:** 
  - Majority-class baseline
  - Logistic Regression ($C \in \{0.01, 0.1, 1, 10\}$)
  - RBF SVM ($C \in \{0.1, 1, 10\}$, $\gamma \in \{\text{'scale'}, 0.01, 0.1\}$)
  - Random Forest ($300$ trees, depth and leaf size tuned via 5-fold stratified CV)
- **Quantum Model:** Top 4 features selected via F-score, angle-encoded into 4 qubits via `ZZFeatureMap` (2 repetitions, linear entanglement). Fidelity kernel computed as exact statevector overlap $|\langle\psi(x)|\psi(x')\rangle|^2$. Regularisation constant $C$ tuned via CV on training kernel.

---

## 4. Quantum vs Classical Model Mechanics (Theoretical Premise vs Findings)

### Theoretical Mechanics
In theory, quantum machine learning approaches like Quantum Kernel Support Vector Machines aim to achieve performance gains over classical algorithms through:
- **Exponential Hilbert Space Mapping:** A quantum feature map (e.g., `ZZFeatureMap`) encodes classical features into quantum states in an exponential $2^n$-dimensional Hilbert space (where $n$ is qubit count), mapping non-linear relations into high dimensions.
- **Quantum Entanglement & Superposition:** Entanglement captures intricate, multi-feature interactions that are hard to compute classically.
- **Low-Data Efficiency:** Quantum state representations are hypothesized to discern fine geometric data structures even under severe data scarcity.

### Empirical Findings in Practice
- **Lower Performance:** On full training data, the best classical model (logistic regression) reached an AUPRC of $0.991 \pm 0.005$, and the matched 4-feature classical RBF SVM reached $0.978 \pm 0.008$, compared to $0.932 \pm 0.013$ for the 4-qubit quantum kernel.
- **Kernel Concentration:** Increasing qubits (e.g., to 6 qubits) caused state overlaps to concentrate around similar low values (mean off-diagonal value dropped to $\sim 0.05$), making data points appear uniformly dissimilar and diminishing classifier discriminative power.
- **Clinical Safety Concerns:** The quantum model yielded significantly more false negatives ($17$ missed malignant cases at 100% data vs $6$ for logistic regression), which represents a major clinically undesirable outcome.

---

## 5. Results Summary

### Table 1: Full Training Data Performance (100% Data, Test Set, Mean ± SD)

| Model | AUROC | AUPRC | Sensitivity | Specificity |
| :--- | :---: | :---: | :---: | :---: |
| **Logistic Regression (30 feat.)** | $0.993 \pm 0.005$ | $0.991 \pm 0.005$ | $0.912 \pm 0.041$ | $0.987 \pm 0.014$ |
| **RBF SVM (30 feat.)** | $0.987 \pm 0.010$ | $0.983 \pm 0.013$ | $0.916 \pm 0.024$ | $0.966 \pm 0.033$ |
| **Random Forest (30 feat.)** | $0.985 \pm 0.009$ | $0.982 \pm 0.009$ | $0.906 \pm 0.033$ | $0.963 \pm 0.015$ |
| **RBF SVM (same 4 feat.)** | $0.982 \pm 0.007$ | $0.978 \pm 0.008$ | $0.853 \pm 0.039$ | $0.974 \pm 0.024$ |
| **Quantum Kernel SVM (4 qubits)** | $0.953 \pm 0.013$ | $0.932 \pm 0.013$ | $0.731 \pm 0.227$ | $0.946 \pm 0.043$ |

### Table 2: AUPRC Performance Across Training Data Sizes

| Model | 10% Data | 25% Data | 50% Data | 100% Data |
| :--- | :---: | :---: | :---: | :---: |
| **Logistic Regression (30 feat.)** | $0.969 \pm 0.037$ | $0.985 \pm 0.009$ | $0.989 \pm 0.006$ | $0.991 \pm 0.005$ |
| **RBF SVM (same 4 feat.)** | $0.946 \pm 0.047$ | $0.978 \pm 0.008$ | $0.978 \pm 0.008$ | $0.978 \pm 0.008$ |
| **Quantum Kernel SVM (4 qubits)** | $0.737 \pm 0.118$ | $0.826 \pm 0.046$ | $0.905 \pm 0.023$ | $0.932 \pm 0.013$ |

---

## 6. Quantum Resource and Efficiency Analysis

- **Circuit Measurements:** 4 qubits, circuit depth 19 (decomposed), 12 two-qubit gates per feature-map circuit, 34 total gates.
- **Kernel Evaluations:** 8,190 (10%), 26,730 (25%), 73,630 (50%), 226,462 (100%).
- **Scaling Bottlenecks:** Exact simulation statevector memory grows exponentially ($2^n$). Kernel evaluations scale quadratically with dataset size $\mathcal{O}(N^2)$.

---

## 7. Circuit Configuration Ablation

- **2 Qubits, 1 Rep:** AUPRC $0.781 \pm 0.055$ | Mean off-diagonal kernel $0.349$
- **4 Qubits, 1 Rep:** AUPRC $0.950 \pm 0.013$ | Mean off-diagonal kernel $0.102$
- **4 Qubits, 2 Reps (Main):** AUPRC $0.932 \pm 0.013$ | Mean off-diagonal kernel $0.117$
- **6 Qubits, 2 Reps:** AUPRC $0.952 \pm 0.009$ | Mean off-diagonal kernel $0.049$

---

## 8. Discussion and Limitations

- **Key Conclusions:** The quantum model underperformed compared to matched and full classical models. It exhibited higher sensitivity variance and greater false negative rates.
- **Limitations:** Single small dataset, simulation-only (no hardware noise or shots), threshold held fixed at 0 rather than tuned on validation data, exploratory post-hoc ablation.

---

## 9. Ethical & Reproducibility Considerations

- Educational prototype; not for clinical deployment.
- Public dataset (Breast Cancer Wisconsin Diagnostic); requires larger cohort validation and demographic subgroup analysis before real-world use.
- Fully reproducible with fixed random seeds (0–4) using **Python 3.13**, **Scikit-Learn 1.9.1**, **Qiskit 2.6.1**, and **Qiskit Machine Learning 0.9.1**.
