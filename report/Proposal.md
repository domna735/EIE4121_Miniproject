# EIE4121 Mini‑Project Proposal
Detecting LLM‑Generated Emails Using Machine Learning

---

## 1. Introduction and Problem Definition

Large Language Models (LLMs), such as ChatGPT and WormGPT, have rapidly advanced in their ability to generate coherent, human‑like text. While these models enable many beneficial applications, they also introduce new cybersecurity risks. In particular, attackers can exploit LLMs to automatically generate phishing emails that are grammatically correct, stylistically polished, and more convincing than traditional phishing attempts. As a result, distinguishing between human‑written and LLM‑generated emails has become an emerging challenge for email security systems.

This project investigates whether machine learning methods can effectively detect LLM‑generated emails based on linguistic and structural features. The dataset used in this study contains both human‑generated and LLM‑generated emails, each further categorized as legitimate or phishing. The goal is to determine whether measurable differences exist between these classes and whether machine learning models can learn to classify the source of an email.

### Classification Problem

The primary task addressed in this proposal is:

### Binary Classification: Human‑Generated vs LLM‑Generated Emails

Given an email’s subject and body text, the objective is to classify whether the email was written by:

- a **human**, or  
- an **LLM** (ChatGPT or WormGPT)

This problem is directly aligned with the course theme of machine learning in cybersecurity and reflects a real‑world challenge faced by modern email filtering systems.

### Dataset Description

The dataset contains four categories:

- Human‑legitimate (1000)  
- Human‑phishing (1000)  
- LLM‑legitimate (1000)  
- LLM‑phishing (1000)  

During preprocessing, the file `data/archive_data/llm-generated/phishing.csv` was found to be malformed (broken CSV structure / concatenated records). A repaired version was produced at `data/archive_data/llm-generated/phishing_repaired.csv` to restore the intended 1,000 LLM‑phishing samples.

The dataset used in this project contains:

| Class | Count |
|---|---:|
| Human (legit + phishing) | 2,000 |
| LLM (legit + phishing) | 2,000 |
| **Total** | **4,000** |

This dataset is used for feature analysis (Phase 1) and baseline text classification (Phase 2).

---

## 2. Feature Analysis

To explore whether human‑generated and LLM‑generated emails exhibit measurable differences, several linguistic and structural features were extracted from the dataset. These features were selected based on stylometric principles and prior research on LLM‑generated text detection.

### 2.1 Length‑Based Features

**Character Count & Word Count**
Human emails show much higher variability (including extreme long threads / quoted text). LLM emails are more consistent in length in this dataset. Because length distributions are heavy‑tailed, we use robust statistics (median/IQR) and log‑scale plots.

### 2.2 Stylometric Features

**Average Word Length**
Included as a lightweight proxy for formality; human emails tend to show more informal phrasing and formatting variability.

**Vocabulary Richness (Type‑Token Ratio)**
We compute TTR to quantify lexical diversity, but interpret it cautiously because it is sensitive to text length.

### 2.3 Punctuation‑Based Features

**Exclamation Marks**  
Human phishing emails frequently use excessive punctuation (e.g., “URGENT!!!”). LLM phishing emails are more polished and restrained.

**Question Marks**  
Human emails include more conversational questions. LLM emails tend to present information declaratively.

### 2.4 Structural Features

**URL Count**
URL features are zero‑inflated (many emails contain no links). We analyze both URL count and the proportion of emails containing at least one URL.

**Subject Line Length**  
LLM‑generated subjects are longer and more descriptive. Human subjects are shorter and sometimes incomplete.

### 2.5 Summary of Observed Differences

| Feature | Human emails (this dataset) | LLM emails (this dataset) |
|---|---|---|
| Length (chars/words) | Heavy‑tailed; extreme outliers appear | Narrower distribution; more consistent |
| Structure | Often includes replies/quotes/boilerplate | Often single, self‑contained message |
| URLs | Zero‑inflated; compare count and “has URL” | Similar; often 0–1 links |

These patterns suggest that machine learning models can potentially distinguish between human and LLM‑generated emails based on stylistic and structural cues.

---

## 3. Proposed Methods

To classify emails as human‑generated or LLM‑generated, two machine learning methods are proposed. Both methods operate on TF‑IDF representations of the email text.

### 3.1 Text Representation: TF‑IDF

The email subject and body are combined and transformed using Term Frequency–Inverse Document Frequency (TF‑IDF). This representation:

- captures word importance  
- reduces the influence of common words  
- works well with linear models  
- is widely used in spam detection and text classification  

Preprocessing includes lowercasing, English stopword removal, and n‑gram extraction (1–2 grams). TF‑IDF is applied to a unified `text` field (subject + body).

---

### 3.2 Method 1: Logistic Regression (Baseline)

Logistic Regression is chosen as the baseline due to its simplicity, interpretability, and strong performance on high‑dimensional text data.

**Advantages:**

- Fast to train  
- Works well with TF‑IDF  
- Coefficients reveal influential words  
- Provides a strong baseline for comparison  

---

### 3.3 Method 2: Linear Support Vector Machine (SVM)

A Linear SVM is selected as the second method. SVMs are widely used in text classification due to their ability to handle high‑dimensional sparse data and find robust decision boundaries.

**Advantages:**

- Strong performance on text data  
- Effective in high‑dimensional spaces  
- Maximizes class separation  
- Often outperforms logistic regression in text tasks  

Kernel SVMs are avoided due to poor scalability with large TF‑IDF matrices.

---

### 3.4 Method 3 (exploratory, beyond lecture scope): One‑Class Anomaly Detection ("negative learning")

To explore a more realistic cybersecurity constraint (only trusted *normal* data available), we additionally evaluate a one-class novelty detection approach:

- Treat **human-written emails** as the normal class
- Detect **LLM-generated emails** as anomalies
- Pipeline: TF‑IDF → Truncated SVD → scaling → One-Class SVM

This is not required for the core deliverable, but it helps test whether “LLM-ness” is detectable as an outlier pattern rather than a separable supervised class.

---

### 3.4 Evaluation Strategy

Both models will be evaluated using:

- Train–test split (e.g., 80/20)  
- Accuracy  
- Precision, Recall, F1‑score  
- Confusion matrix  

We use a stratified 80/20 train–test split with a fixed random seed and report accuracy, precision, recall, F1, and confusion matrices.

Baseline results (human vs LLM) are saved in `results/baseline_metrics.json`:

- Logistic Regression: accuracy 0.99875
- Linear SVM: accuracy 1.00

These near‑perfect results suggest that the dataset may contain strong source‑specific cues (e.g., boilerplate/reply formatting, length effects). In Phase 2 we will add sanity checks (e.g., length‑controlled evaluation, subset evaluation by phishing/legit) to confirm the model is not relying on trivial artifacts.

Additional Phase 2 evidence is saved as:

- `results/phishing_within_source_metrics.json`: phishing vs legit baselines evaluated within human-only and within LLM-only subsets.
- `results/baseline_validation.json`: stronger validation outputs (5-fold CV summary, duplicate/near-duplicate indicators, and length-binned accuracy).

Exploratory Method 3 evidence is saved as:

- `results/oneclass_svm_metrics.json`
- `results/oneclass_svm_curves.png`

In this dataset/split, the One-Class SVM approach performs poorly for detecting LLM emails (LLM recall = 0, ROC AUC ≈ 0.024). This negative result suggests that LLM emails are not outliers relative to human emails in the chosen TF‑IDF feature space.

---

## 3.5 Limitations / Threats to Validity (including overfitting)

The very high baseline scores are encouraging, but they also raise the risk that the models are exploiting dataset artifacts rather than learning general “LLM vs human” properties. Key threats include:

- **Overfitting to this dataset**: even with regularization, a TF‑IDF model can overfit to frequent phrases, templates, or repeated patterns that are specific to the dataset rather than the true source.
- **Train–test leakage via near‑duplicates**: email datasets often contain near‑duplicate messages (e.g., repeated boilerplate, forwarded threads, repeated phishing templates). Random splits can place highly similar samples in both train and test, inflating accuracy.
- **Spurious cues unrelated to authorship**: differences in reply formatting, quoted text, headers, or message length can become shortcuts for classification.
- **Metric inflation from easy separability**: if classes are separable due to formatting/length, high accuracy does not guarantee robustness under distribution shift.

To mitigate these risks in Phase 2, we will add stronger validation:

- Cross‑validation and/or repeated splits to check stability
- Length‑controlled evaluation (bucket/match by word count) and ablations (remove quoted text / signatures if present)
- Subset evaluation within legit‑only and phishing‑only partitions
- Duplicate/near‑duplicate checks (or template-aware splits where possible)
- Manual error analysis of any misclassifications

### Measured leakage indicators (current dataset)

We explicitly checked train/test leakage indicators on the baseline split:

- **Exact duplicate overlap (normalized text):** ~33.1% of test-unique texts also appear in the train set.
- **Near-duplicate rate:** ~33.5% of test samples have a nearest-neighbor similarity ≥ 0.98 to a training sample using character n-gram TF-IDF.

These results strongly suggest the dataset contains repeated templates/boilerplate (or very similar messages) that can inflate performance under random splitting. This motivates template-aware splitting or de-duplication strategies in Phase 2.

## 4. Conclusion

This proposal outlines a machine learning approach to detecting LLM‑generated emails using linguistic and structural features. The repaired dataset of 4,000 emails provides a solid foundation for analysis. Feature exploration and baseline TF‑IDF models indicate strong separability between human and LLM‑generated text in this dataset, motivating deeper validation in Phase 2.

Two machine learning methods—Logistic Regression and Linear SVM—are proposed for Phase 2. These models, combined with TF‑IDF text representation, are expected to provide strong performance and meaningful insights into the detectability of LLM‑generated emails.
The results will contribute to understanding the challenges of LLM‑generated phishing detection and inform future cybersecurity strategies.