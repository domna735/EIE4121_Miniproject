# Section 2 — Feature Analysis

## 2. Feature Analysis

To investigate whether measurable differences exist between human‑generated and LLM‑generated emails, we extracted a set of simple linguistic and structural features from the email text. These features are commonly used in stylometry and practical email classification.

This analysis uses the repaired dataset of **4,000 emails** (2,000 human and 2,000 LLM‑generated). The LLM phishing file was originally malformed; a repaired version was produced at `data/archive_data/llm-generated/phishing_repaired.csv` and used to restore the intended 1,000 LLM‑phishing samples.

---

## 2.1 Length‑Based Features

### (a) Character Count of Email Body
This feature measures the total number of characters in the email body.

**Observation:**  
- Human emails show **much higher variability** (including very long threads, quoted replies, and boilerplate), producing extreme outliers.
- LLM emails are **more consistent** in length in this dataset.

### (b) Word Count
Counts the number of words in the email body.

**Observation:**  
- Human emails include many short, task‑oriented messages, but also very long emails/threads, so the distribution is heavy‑tailed.
- LLM emails tend to cluster into a narrower band of word counts.

---

## 2.2 Stylometric Features

### (a) Average Word Length
Computed as total characters divided by total words.

**Observation:**  
- LLM emails often use more uniform, formal phrasing; human emails exhibit more informal phrasing and formatting variability.
- This feature is included as a lightweight proxy for formality, but it is not expected to be decisive alone.

### (b) Vocabulary Richness (Type‑Token Ratio)
Measures lexical diversity.

**Observation:**  
- We compute type‑token ratio (TTR) as a simple lexical diversity indicator.
- Interpretation is handled cautiously because TTR is sensitive to document length; we compare distributions and robust statistics rather than relying on means only.

---

## 2.3 Punctuation‑Based Features

### (a) Exclamation Marks
Counts occurrences of “!”.

**Observation:**  
- Human phishing emails frequently use **excessive punctuation** to create urgency (e.g., “URGENT!!!”).  
- LLM phishing emails are more **polished and restrained**, rarely using multiple exclamation marks.

### (b) Question Marks
Counts occurrences of “?”.

**Observation:**  
- Human emails often include conversational questions.  
- LLM emails use fewer questions and tend to present information in declarative form.

---

## 2.4 Structural Features

### (a) URL Count
For human emails, the dataset provides a `urls` count. For LLM emails, we estimate URL count via regex matching.

**Observation:**  
- URL features are **zero‑inflated** (many emails contain no URLs), so we also compare the proportion of emails containing at least one URL.

### (b) Subject Line Length
Length of the subject field.

**Observation:**  
- LLM‑generated subjects are often **longer and more descriptive**.  
- Human subjects are shorter and sometimes incomplete.

---

## 2.5 Summary of Observed Differences

The feature analysis reveals several consistent patterns:

| Feature | Human emails (this dataset) | LLM emails (this dataset) |
|---|---|---|
| Length (chars/words) | Heavy‑tailed; extreme long threads appear | More consistent; narrower distribution |
| Structure | Often includes headers/quotes/boilerplate | Often single, self‑contained message |
| Punctuation | Can be noisy (replies, forwarding, informal style) | Generally more uniform |
| URLs | Zero‑inflated; use both count and “has URL” | Often 0–1 links; also zero‑inflated |

These differences suggest that machine learning models can potentially learn to distinguish between human and LLM‑generated emails based on stylistic and structural cues alone.

---

## 2.6 Differences Among Classes: Phishing vs Legit (Within Each Source)

In addition to comparing **human vs LLM**, we also examine how **phishing vs legit** differs *within* each source. This addresses the Phase 1 requirement of showing differences among the dataset classes, and it helps identify potential dataset shortcuts (e.g., URLs) that could inflate model performance.

### (a) URL Presence (Has URL)

We use a binary feature **has_url** (whether an email contains ≥1 URL), because URL counts are zero‑inflated.

- **Human emails:** phishing is much more likely to contain URLs than legit.
	- Legit: ~0.433 contain ≥1 URL
	- Phishing: ~0.952 contain ≥1 URL
- **LLM emails:** both legit and phishing frequently contain URLs.
	- Legit: ~0.936 contain ≥1 URL
	- Phishing: ~0.856 contain ≥1 URL

**Interpretation:** URL presence is a strong phishing indicator for *human* emails in this dataset, but it is less reliable for *LLM* emails because URLs are common in both LLM legit and LLM phishing samples.

### (b) Length and Stylometric Differences

- **Human emails:** legit messages are much longer on average than phishing messages (likely due to threads/quoted text/boilerplate in legit human emails).
- **LLM emails:** phishing and legit are closer in length and stylometric statistics (more uniform generation style).

These within-source comparisons support the need for stronger validation in Phase 2 (e.g., controlling for length and checking near-duplicate leakage).

---

## 2.7 Visual Analysis (Performed in Notebook)

The notebook generates figures used as proposal evidence, including:

- Log‑scale histograms for length features (saved to `results/phase1_length_distributions_log.png`)
- URL feature plots (saved to `results/phase1_url_features.png`)
- Within-source phishing vs legit plots (saved to `results/phase1_within_source_boxplots.png` and `results/phase1_within_source_has_url.png`)

Because the length distributions are heavy‑tailed, we report robust statistics (median/IQR) alongside plots.

