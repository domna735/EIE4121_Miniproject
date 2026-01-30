# Section 3 — Proposed Methods

## 3. Proposed Methods

To address the classification problem of distinguishing between human‑generated and LLM‑generated emails, two machine learning methods are proposed. These methods were selected based on their strong performance in text classification tasks, interpretability, and suitability for high‑dimensional sparse data such as TF‑IDF features.

Both models will operate on the combined email text (subject + body), transformed into numerical features using the TF‑IDF representation described below.

---

## 3.1 Text Representation: TF‑IDF Features

Before applying machine learning models, the raw email text must be converted into numerical form. For this project, the **Term Frequency–Inverse Document Frequency (TF‑IDF)** representation will be used.

### **Why TF‑IDF?**

- It captures the importance of words relative to the entire dataset.  
- It reduces the influence of extremely common words (e.g., “the”, “and”).  
- It works well with linear models and tree‑based models.  
- It is widely used in email classification, spam detection, and authorship analysis.

### Preprocessing Steps

- Lowercasing
- English stopword removal
- N‑grams (1–2 grams) to capture short phrases and stylistic patterns
- TF‑IDF vectorization on the combined text (subject + body): `text = subject + body`

This produces a high‑dimensional sparse feature matrix suitable for linear text classifiers.

---

## 3.2 Method 1: Logistic Regression (Baseline Model)

Logistic Regression is chosen as the baseline classifier due to its simplicity, interpretability, and strong performance on high‑dimensional text data.

### **Advantages**

- Performs well with TF‑IDF features  
- Fast to train and evaluate  
- Coefficients provide insight into which words are most indicative of human vs LLM text  
- Serves as a strong baseline for comparison  

### **Expected Behavior**

- LLM‑generated emails may be associated with more formal vocabulary and longer phrases.  
- Human emails may show more informal language, typos, and inconsistent structure.  
- Logistic Regression can capture these linear separations effectively.

---

## 3.3 Method 2: Support Vector Machine (Linear SVM)

A **Linear Support Vector Machine (SVM)** is selected as the second method. SVMs are widely used in text classification due to their ability to handle high‑dimensional sparse data and find robust decision boundaries.

### **Advantages**

- Strong performance on text classification tasks  
- Effective in high‑dimensional spaces  
- Maximizes the margin between classes, improving generalization  
- Often outperforms logistic regression when classes are not linearly separable in simple feature space  

### **Why Linear SVM instead of Kernel SVM?**

- Kernel SVMs do not scale well to thousands of samples and tens of thousands of TF‑IDF features  
- Linear SVMs are computationally efficient and well‑suited for this dataset  

---

## 3.4 Model Evaluation Strategy

Both models are evaluated using:

- **Train–test split** (80% training, 20% testing), stratified by class
- **Accuracy**  
- **Precision, Recall, and F1‑score**  
- **Confusion matrix** to analyze misclassification patterns  

These metrics allow comparison of the two methods and provide visibility into error types.

Baseline results for the primary task (human vs LLM) are stored in `results/baseline_metrics.json`.

---

## 3.5 Validation and Sanity Checks (Phase 2)

Because initial baselines can achieve near‑perfect accuracy on this dataset, we will include additional checks to reduce the risk of learning trivial artifacts:

- Length‑controlled evaluation (e.g., match or bucket by word count)
- Evaluate within phishing‑only and legit‑only subsets
- Error analysis of misclassified samples (if any)

## 3.6 Summary of Proposed Methods

| Component | Description |
|----------|-------------|
| **Text Representation** | TF‑IDF on subject + body |
| **Model 1** | Logistic Regression (baseline) |
| **Model 2** | Linear SVM |
| **Evaluation** | Accuracy, Precision, Recall, F1, Confusion Matrix |

These two methods provide a balanced combination of interpretability, computational efficiency, and strong performance on text classification tasks. They are well‑suited for detecting stylistic and structural differences between human and LLM‑generated emails.