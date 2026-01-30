
# Short Report — Detecting LLM‑Generated Emails Using Machine Learning

**Course:** EIE4121 Machine Learning in Cyber-security

**Task:** Binary classification — **Human-written** vs **LLM-generated** emails

**Repository evidence:**
- Notebook: `notebooks/phase1_analysis.ipynb`
- Dataset: `data/archive_data/` (with repaired `data/archive_data/llm-generated/phishing_repaired.csv`)
- Results: `results/baseline_metrics.json`, `results/cm_logreg.png`, `results/cm_linear_svm.png`

---

## 1. Problem statement and motivation

Large Language Models (LLMs) can generate fluent, polished text at scale. This capability increases the risk that attackers can produce phishing emails that appear more professional and persuasive than traditional spam. This project studies whether machine learning methods can distinguish **human-written** from **LLM-generated** emails using only the email text.

We focus on the primary classification task:

> Given an email’s text (subject + body), predict whether it was written by a **human** or by an **LLM**.

---

## 2. Dataset and preprocessing

### 2.1 Dataset composition

The dataset contains four groups (each intended to have 1,000 samples):

- Human‑legitimate
- Human‑phishing
- LLM‑legitimate
- LLM‑phishing

After repairing the malformed LLM phishing CSV, the dataset used in experiments contains:

| Source | Count |
|---|---:|
| Human | 2,000 |
| LLM | 2,000 |
| **Total** | **4,000** |

**Data integrity note.** The original file `data/archive_data/llm-generated/phishing.csv` was malformed (broken CSV structure / concatenated records). A repaired version is saved as `data/archive_data/llm-generated/phishing_repaired.csv` and is used to restore the intended 1,000 LLM‑phishing samples.

### 2.2 Text field construction and features

To ensure consistent input across sources, we construct a unified text field:

$$\text{text} = \text{subject} + "\n\n" + \text{body}$$

For human emails, the dataset already contains `subject` and `body`; for LLM emails which contain only a `text` field, we treat it as the body and set subject to empty. This unified `text` is then used for TF‑IDF modeling.

---

## 3. Methods employed (two models)

Both methods use the same text representation:

### 3.1 TF‑IDF representation (shared)

We convert emails into a sparse vector representation using **TF‑IDF**:

- Lowercasing
- English stopword removal
- Word n‑grams (1–2 grams)
- `min_df=2`, `max_df=0.95` (to reduce noise from extremely rare or extremely common tokens)

This representation is effective for high-dimensional text and is commonly paired with linear classifiers.

### 3.2 Method 1: Logistic Regression (TF‑IDF + LogReg)

Logistic Regression is a strong baseline for text classification because it:

- scales well to sparse TF‑IDF features
- is relatively interpretable (feature weights can be inspected)
- provides a simple, well-understood linear decision boundary

In practice, it learns a weight for each n‑gram and predicts the class from the weighted sum.

### 3.3 Method 2: Linear Support Vector Machine (TF‑IDF + Linear SVM)

Linear SVM (implemented as a linear support vector classifier) is widely used in text classification because it:

- is effective in high-dimensional sparse spaces
- often yields strong generalization with a margin-maximizing objective
- is competitive with Logistic Regression for many NLP tasks

We choose a **linear** SVM for efficiency and suitability to TF‑IDF matrices.

### 3.4 Method 3 (beyond lecture scope): One‑Class Anomaly Detection ("negative learning")

To explore a more cybersecurity-style setting where we only have reliable *normal* data, we treat **human-written emails** as the normal class and aim to detect **LLM-generated emails** as anomalies.

Implementation:

- Train split: human-only (80% train, 20% held-out human test)
- Representation: TF‑IDF (same settings as Methods 1–2)
- Dimensionality reduction: Truncated SVD (200 components)
- Model: One-Class SVM (RBF kernel, `nu=0.05`)

We evaluate on a balanced set: held-out human test vs an equal number of sampled LLM emails. Metrics and curves are saved as:

- `results/oneclass_svm_metrics.json`
- `results/oneclass_svm_curves.png`

---

## 4. Experimental setup

### 4.1 Train/test protocol

- Train/test split: **80% / 20%**
- Stratification: by class (human vs LLM)
- Random seed: fixed for reproducibility

### 4.2 Evaluation metrics

We report standard binary classification metrics:

- Accuracy
- Precision
- Recall
- F1 score
- Confusion matrix

---

## 5. Results

The metrics below are taken from `results/baseline_metrics.json`.

| Model | Accuracy | Precision | Recall | F1 |
|---|---:|---:|---:|---:|
| TF‑IDF + Logistic Regression | 0.99875 | 1.00 | 0.9975 | 0.9987 |
| TF‑IDF + Linear SVM | 1.00000 | 1.00 | 1.00 | 1.00 |

Confusion matrices are saved as:

- `results/cm_logreg.png`
- `results/cm_linear_svm.png`

**Observation.** Both linear models achieve extremely high performance on this dataset. Linear SVM slightly outperforms Logistic Regression (perfect test performance in this split).

### 5.1 Method 3 results (one-class anomaly detection)

The one-class model performs poorly for detecting LLM emails in this dataset/split (from `results/oneclass_svm_metrics.json`):

| Model | Accuracy | Precision | Recall | F1 | ROC AUC | PR AUC |
|---|---:|---:|---:|---:|---:|---:|
| TF‑IDF + SVD + One‑Class SVM (train on human) | 0.4775 | 0.0000 | 0.0000 | 0.0000 | 0.0245 | 0.3103 |

Confusion matrix summary (true rows = [human, llm], predicted cols = [human, llm]):

- Human predicted as LLM: 18/400 (false positives)
- LLM predicted as LLM: 0/400 (true positives)

---

## 6. Comparison and discussion

### 6.1 Performance comparison

The two methods are very close in performance. In this run:

- Logistic Regression makes a very small number of errors (reflected by recall < 1.0).
- Linear SVM reaches perfect scores.

Given that both models are linear classifiers over TF‑IDF features, it is expected that they behave similarly. Differences may come from:

- different loss functions (log loss vs hinge loss)
- different regularization behavior
- margin maximization in SVM

### 6.2 Practical considerations

| Consideration | Logistic Regression | Linear SVM |
|---|---|---|
| Interpretability | Strong (inspect coefficients) | Moderate (weights exist, but no probabilities by default) |
| Probability outputs | Natural | Not native (needs calibration) |
| Typical text performance | Strong baseline | Often slightly stronger |
| Training stability | Good | Good |

In a cybersecurity setting, interpretability can be valuable (e.g., analyzing which n‑grams indicate “LLM-like” text), but high accuracy alone is not sufficient if the model relies on spurious artifacts.

---

## 7. Limitations (including possible overfitting)

Even though the results are strong, there are important limitations and threats to validity.

### 7.1 Overfitting and dataset artifacts

Near‑perfect test scores can occur when the dataset contains strong shortcuts that separate classes, for example:

- **Formatting/structure cues**: human emails may contain replies, quoted threads, headers, signatures, or boilerplate that are less common in LLM emails.
- **Length effects**: human emails show heavy‑tailed length distributions (very short and very long messages). If LLM emails are more consistent, models may rely on length-correlated tokens.
- **Template repetition / near duplicates**: phishing templates, boilerplate disclaimers, or repeated patterns can produce near-duplicate samples. If duplicates appear across train and test, accuracy is inflated.

These are forms of **overfitting to the dataset distribution** rather than learning generalizable “human vs LLM” characteristics.

**Measured leakage indicators (this dataset / split).** We computed duplicate and near-duplicate overlap between train and test:

- Exact duplicate overlap (normalized text): ~33.1% of test-unique texts also appear in the train set.
- Near-duplicate rate: ~33.5% of test samples have similarity ≥ 0.98 to a training sample (character n-gram TF‑IDF nearest neighbor).

These results support the concern that near-perfect scores may be inflated by repeated templates/boilerplate.

### 7.2 Generalization beyond this dataset

The dataset’s LLM emails may come from a limited set of generation prompts or styles. In real deployments:

- LLM models and prompting strategies change over time
- attackers can paraphrase or post-edit
- email formatting varies across organizations

As a result, strong in-dataset performance may not translate to strong real-world performance under distribution shift.

### 7.3 What Method 3 suggests

The failure of the one-class approach indicates that, under the chosen TF‑IDF feature space, the LLM emails in this dataset are not "outliers" relative to human emails. In fact, they appear *more* inlier-like than human (which includes highly diverse real-world formatting). This negative result is useful: it suggests that one-class novelty detection is not a reliable detector here without stronger representations, different anomaly models, or a different definition of "normal".

### 7.3 Model-specific limitations

- **Logistic Regression**: can underfit non-linear patterns; may require careful regularization tuning.
- **Linear SVM**: does not directly output probabilities; interpreting confidence requires calibration.

- **One‑Class SVM**: highly sensitive to feature scaling and class overlap; if the anomaly class lies inside the normal distribution in feature space, recall can collapse to 0.

### 7.4 Planned mitigations

To strengthen validity in the next phase, we will add:

- cross-validation / repeated splits
- evaluation within phishing-only and legit-only subsets
- length-controlled evaluation (bucket/match by word count)
- near-duplicate checks or template-aware splitting (where feasible)
- manual inspection of errors (if any)

Additional saved evidence:
- `results/baseline_validation.json` (cross-validation summary + leakage indicators)
- `results/phishing_within_source_metrics.json` (phishing vs legit baselines within human-only and within LLM-only)

---

## 8. Conclusion

This short report compared two linear text classification methods for detecting LLM-generated emails: **TF‑IDF + Logistic Regression** and **TF‑IDF + Linear SVM**. Both achieve very high performance on the repaired 4,000-email dataset, with Linear SVM performing best in the current split.

However, the near-perfect results highlight a key risk: **overfitting to dataset-specific artifacts** (formatting, length distributions, and near-duplicate templates). Therefore, the next steps emphasize stronger validation protocols and sanity checks to confirm that performance reflects genuine source-detection ability rather than shortcuts.

