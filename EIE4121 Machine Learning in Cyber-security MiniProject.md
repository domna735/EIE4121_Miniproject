# EIE4121 Machine Learning in Cyber-security — Mini Project
## Phase 1 Proposal (Due: 13 March 2026)

**Project title:** Detecting LLM-Generated Emails Using Machine Learning

**Group members:**
- Student 1: (Name, Student ID)
- Student 2: (Name, Student ID)

**Repository structure (local):**
- Notebook: `notebooks/phase1_analysis.ipynb`
- Data: `data/archive_data/`
- Draft write-ups: `report/`

---

## 1. Introduction & Motivation

Large Language Models (LLMs) such as ChatGPT and WormGPT have become powerful tools for generating human-like text. While these models enable many beneficial applications, they also introduce new risks in cybersecurity. In particular, attackers can leverage LLMs to automatically generate phishing emails that are grammatically correct, stylistically polished, and more convincing than traditional phishing attempts.

This mini-project investigates whether machine learning methods can reliably distinguish between **human-written** and **LLM-generated** emails using only the email text.

---

## 2. Classification Problem

### 2.1 Primary Task (Phase 1)
**Binary classification: Human vs LLM-generated**

Given an email’s text (subject + body), classify whether it was written by:
- a **human**, or
- an **LLM** (ChatGPT or WormGPT)

This task aligns with the course theme (ML for cybersecurity) and reflects a real-world challenge for email security systems.

### 2.2 Optional Secondary Tasks (Phase 2 / Extension)
Depending on time, we may also explore:
- Human-only: **legit vs phishing**
- LLM-only: **legit vs phishing**

These are not the primary focus for Phase 1, but can be used to compare difficulty and failure modes.

---

## 3. Dataset & Preprocessing

### 3.1 Dataset source
Dataset: Human / LLM-generated phishing & legitimate emails (Kaggle)
https://www.kaggle.com/datasets/francescogreco97/human-llm-generated-phishing-legitimate-emails/data

The dataset contains four categories:
- Human-legitimate
- Human-phishing
- LLM-legitimate
- LLM-phishing

Originally, each category contained 1,000 samples (total 4,000).

### 3.2 Observed data quality issue
During loading, `llm-generated/phishing.csv` contains malformed rows (e.g., unescaped quotes / broken delimiters / multi-line entries).
In the current notebook, these rows are skipped during parsing.

### 3.3 Final dataset used for Phase 1
After skipping malformed entries, the working dataset contains:

| Class | Count |
|------|------:|
| Human (legit + phishing) | 2000 |
| LLM (legit + phishing) | 1595 |
| **Total** | **3595** |

### 3.4 Text fields and target labels
We will build a unified text field:

$$\text{text} = \text{subject} + \text{body}$$

Targets:
- `source`: {human, llm}
- `is_phishing`: {0, 1} (available for analysis; primary target is `source`)

---

## 4. Feature Analysis (Exploratory)

To investigate whether measurable differences exist between human-generated and LLM-generated emails, we examine a set of linguistic and structural features commonly used in stylometry and text classification.

### 4.1 Length-based features
- Character count of body
- Word count

**Observation:** LLM emails tend to be longer and more polished; human emails show higher variance (short replies, informal text).

### 4.2 Stylometric features
- Average word length
- Vocabulary richness (type-token ratio)

**Observation:** LLM text tends to have slightly more formal vocabulary and higher lexical diversity.

### 4.3 Punctuation-based features
- Exclamation mark count
- Question mark count

**Observation:** Human phishing emails may contain more emotional/urgent punctuation (e.g., repeated exclamation marks), while LLM emails are often more restrained.

### 4.4 Structural features
- URL count (when available)
- Subject length

**Observation:** Human phishing emails may contain multiple suspicious links; LLM emails often include fewer links and a more structured subject.

---

## 5. Proposed Methods

Both models use TF-IDF features extracted from the combined email text (subject + body).

### 5.1 Text representation: TF-IDF
TF-IDF is used because it:
- works well for sparse, high-dimensional text
- downweights common words
- pairs well with linear classifiers

Preprocessing (baseline):
- lowercase
- tokenization
- stopword removal
- optional n-grams (e.g., 1–2 grams)

### 5.2 Method 1: Logistic Regression (baseline)
Logistic Regression is selected as a strong, interpretable baseline for TF-IDF text classification.

Advantages:
- fast training
- good performance on sparse features
- interpretable coefficients (top indicative tokens)

### 5.3 Method 2: Linear Support Vector Machine (Linear SVM)
Linear SVM is widely used for text classification due to strong generalization in high-dimensional spaces.

Advantages:
- robust decision boundary (max-margin)
- efficient at this dataset scale

---

## 6. Evaluation Plan

We will evaluate models using:
- Train/test split (e.g., 80/20) with stratification on `source`
- Accuracy
- Precision, recall, F1-score
- Confusion matrix

We will also:
- compare model performance on human vs LLM subsets (error analysis)
- inspect which tokens/features are most discriminative (model interpretability)

---

## 7. Work Plan & Milestones

**Phase 1 (Proposal) — by 13 March 2026**
- Finalize problem statement + dataset description
- Complete feature analysis with summary tables/plots
- Specify two methods + evaluation metrics

**Phase 2 (Report + Notebook + Presentation) — by 10 April 2026**
- Implement and tune both models
- Produce final metrics, plots, and comparison
- Write limitations + discussion
- Prepare 8–10 minute video

---

## 8. Reference

Greco, F., Desolda, G., Esposito, A., Carelli, A. (2024). *David versus Goliath: Can Machine Learning Detect LLM-Generated Text? A Case Study in the Detection of Phishing Emails.* ITASEC 2024.

---

# Appendix A — Assignment Brief (Original)

THE HONG KONG POLYTECHNIC UNIVERSITY  
Department of Electronic and Information Engineering  
EIE4121 Machine Learning in Cyber-security  
Mini-project  

The mini-project is a critical component of this course. The work should reflect students’ understanding of specific topics beyond the level/scope covered in lectures. The project offers a hands-on opportunity for students to apply their knowledge and showcase their problem-solving skills in the realm of machine learning and data analysis.

Details  
Large language models (LLMs) have been utilized in various applications, including generating emails and texts. While LLMs can be very useful, they have been used to create phishing emails. In this project, the aim is to define a classification problem involving LLM-generated text and determine whether machine learning methods can be used to detect whether an LLM has generated the text.

Dataset link: https://www.kaggle.com/datasets/francescogreco97/human-llm-generated-phishing-legitimate-emails/data  

Phase 1: Project proposal (due date: 13 March 2026)
- Outline the classification problem
- Feature analysis on the data
- Identify two methods

Phase 2: Project Report and Presentation (due date: 10 April 2026)
- Notebook containing results
- Short report comparing two methods and limitations
- Video presentation