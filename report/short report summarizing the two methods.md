
# Short Report (Two Methods Only) — Detecting LLM‑Generated Emails Using Machine Learning

**Course:** EIE4121 Machine Learning in Cyber-security

**Task:** Binary classification — **Human-written** vs **LLM-generated** emails

**Repository evidence:**
- Notebook: `notebooks/phase1_analysis.ipynb`
- Dataset: `data/archive_data/` (with repaired `data/archive_data/llm-generated/phishing_repaired.csv`)
- Results: `results/baseline_metrics.json`, `results/cm_logreg.png`, `results/cm_linear_svm.png`, `results/baseline_validation.json`, `results/leakage_resistant_exact_dedup.json`, `results/leakage_resistant_groupkfold.json`, `results/source_aware_stress_tests.json`

---

## 1. Problem statement

Large Language Models (LLMs) can generate fluent, polished text at scale. This creates a cybersecurity risk: attackers can produce phishing emails that look more professional and persuasive than traditional spam.

We study the primary classification task:

> Given an email’s text (subject + body), predict whether it was written by a **human** or by an **LLM**.

---

## 2. Dataset and preprocessing

### 2.1 Dataset composition

The dataset includes four groups (intended 1,000 samples each):

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

### 2.2 Unified text construction

To ensure consistent input across sources, we construct a unified text field:

$$\text{text} = \text{subject} + "\n\n" + \text{body}$$

For human emails, the dataset contains `subject` and `body`. For LLM emails which contain only a `text` field, we treat it as the body and set subject to empty.

---

## 3. Methods employed (two models)

Both methods share the same text representation.

### 3.1 TF‑IDF representation (shared)

We convert emails into a sparse vector representation using TF‑IDF with:

- lowercasing
- English stopword removal
- word n‑grams (1–2 grams)
- `min_df=2`, `max_df=0.95`

This is a standard and effective representation for linear models on high-dimensional text.

### 3.2 Method 1: Logistic Regression (TF‑IDF + LogReg)

Logistic Regression is a strong baseline for text classification because it:

- scales well to sparse TF‑IDF features
- is relatively interpretable (feature weights)
- provides a simple linear decision boundary

### 3.3 Method 2: Linear SVM (TF‑IDF + Linear SVM)

Linear SVM is widely used in text classification because it:

- performs well in high-dimensional sparse spaces
- often generalizes strongly via margin maximization
- is competitive with Logistic Regression on many NLP tasks

We use a **linear** SVM for efficiency and suitability to TF‑IDF.

---

## 4. Experimental setup

- Train/test split: **80% / 20%**
- Stratification: by class (human vs LLM)
- Random seed: fixed for reproducibility

Metrics reported:

- Accuracy
- Precision
- Recall
- F1 score
- Confusion matrix

---

## 5. Results

Metrics are taken from `results/baseline_metrics.json`.

| Model | Accuracy | Precision | Recall | F1 |
|---|---:|---:|---:|---:|
| TF‑IDF + Logistic Regression | 0.99875 | 1.00 | 0.9975 | 0.9987 |
| TF‑IDF + Linear SVM | 1.00000 | 1.00 | 1.00 | 1.00 |

Confusion matrices:

- `results/cm_logreg.png`
- `results/cm_linear_svm.png`

---

## 6. Comparison of the two methods

Both methods perform extremely well on this dataset.

- Logistic Regression makes a very small number of errors (slightly lower recall).
- Linear SVM achieves perfect test performance in the current split.

Because both models are linear classifiers over TF‑IDF features, similar performance is expected. Differences can arise from:

- log loss (LogReg) vs hinge loss (SVM)
- different regularization behavior
- margin maximization in SVM

---

## 7. Limitations

Despite the strong results, there are important threats to validity:

### 7.1 Dataset artifacts and shortcuts

Near‑perfect scores can occur when models exploit dataset-specific cues rather than generalizable “LLM vs human” characteristics, such as:

- formatting/structure differences (reply threads, quoted text, boilerplate)
- length distribution differences (human emails are heavy‑tailed)
- repeated templates / near duplicates

### 7.2 Measured leakage indicators (this dataset / split)

We computed duplicate and near-duplicate overlap between train and test:

- Exact duplicate overlap (normalized text): ~33.1% of test-unique texts also appear in the train set.
- Near-duplicate rate: ~33.5% of test samples have similarity ≥ 0.98 to a training sample (character n‑gram TF‑IDF nearest neighbor).

These values indicate that random splitting can inflate reported accuracy.

### 7.3 Generalization risk

LLM generation styles, prompts, and post-editing can change over time. Therefore, high in-dataset performance may not translate to robust real-world detection under distribution shift.

### 7.4 Additional robustness checks (leakage-resistant + stress test)

To avoid confusing “near-perfect accuracy” with true generalization, we also ran stricter evaluations while keeping the same two TF‑IDF linear models:

- **Exact de-dup before split:** removing exact duplicate normalized texts reduces the dataset from 4,000 → 3,200 unique samples; performance remains very high (see `results/leakage_resistant_exact_dedup.json`).
- **Near-duplicate GroupKFold:** grouping near-duplicates (character n-gram similarity ≥ 0.98) and evaluating with GroupKFold still yields very high mean accuracy (see `results/leakage_resistant_groupkfold.json`).
- **Topic-shift stress test:** training on legit-only and testing on phishing-only reduces accuracy to ≈ 0.918 (LogReg) and ≈ 0.9325 (Linear SVM), showing sensitivity to distribution shift (see `results/source_aware_stress_tests.json`).

---

## 8. Conclusion

Two TF‑IDF linear baselines (Logistic Regression and Linear SVM) achieve very high performance on the repaired 4,000-email dataset. Linear SVM performs best in the current split.

However, measured train/test overlap and near-duplicate rates suggest that results may be inflated by dataset artifacts, motivating stronger validation and leakage-aware evaluation.

