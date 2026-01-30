# Phase 1 Proposal — Section 1: Problem Definition

## 1. Introduction & Motivation

Large Language Models (LLMs) such as ChatGPT and WormGPT have become powerful tools for generating human‑like text. While these models enable many beneficial applications, they also introduce new risks in cybersecurity. In particular, attackers can leverage LLMs to automatically generate phishing emails that are grammatically correct, stylistically polished, and more convincing than traditional phishing attempts. This raises an important question for cyber‑defense:

**Can machine learning methods reliably distinguish between human‑written and LLM‑generated emails?**

This mini‑project aims to explore this question by analyzing a dataset containing both human‑generated and LLM‑generated emails, including legitimate and phishing examples. The goal is to determine whether measurable linguistic or structural differences exist, and whether machine learning classifiers can detect the source of an email.

---

## 2. Classification Problem to Be Addressed

For this project, we focus on the following classification task:

### Binary Classification Problem: Human vs LLM‑Generated Emails

Given an email’s subject and body text, the objective is to classify whether the email was:

- **Human‑generated**, or  
- **LLM‑generated** (ChatGPT or WormGPT)

This problem is directly aligned with the course theme of machine learning in cybersecurity and reflects a real‑world challenge faced by email security systems.

### Dataset Used

The dataset contains four categories:

- Human‑legitimate  
- Human‑phishing  
- LLM‑legitimate  
- LLM‑phishing  

Originally, each category contained 1,000 samples (total 4,000).

During preprocessing, the file `data/archive_data/llm-generated/phishing.csv` was found to be malformed (broken CSV structure / concatenated records). Instead of discarding a large portion of data with `on_bad_lines='skip'`, we repaired the file using a heuristic parser and saved the recovered dataset to `data/archive_data/llm-generated/phishing_repaired.csv`.

Thus, the dataset used in this project contains:

| Class | Count |
|---|---:|
| Human (legit + phishing) | 2,000 |
| LLM (legit + phishing) | 2,000 |
| **Total** | **4,000** |

This dataset size supports both feature analysis (Phase 1) and baseline text classification (Phase 2).

---

## 3. Why This Problem Is Important

- LLM‑generated phishing emails are becoming harder to detect manually.  
- Traditional phishing filters rely on handcrafted rules that may not generalize to LLM‑generated content.  
- Understanding linguistic differences between human and LLM text can improve future email security systems.  
- This task provides hands‑on experience applying machine learning to a modern cybersecurity challenge.