# Process Log
This document captures the daily activities, decisions, and reflections during the Project, focusing on machine learning methods to detect LLM-generated phishing emails.

Follow the template below to document your activities, decisions, and reflections. Entries are written in a compact but detailed format so they can be reused in the final report.

---

## Project kickoff + repo organization
Intent:
- Start Phase 1 work step-by-step and ensure the workspace structure supports reporting.

Action:
- Reviewed the course mini-project brief and mapped deliverables to folders: `notebooks/` for experiments, `report/` for write-ups, `results/` for exported figures/metrics.
- Confirmed dataset location and categories under `data/archive_data/`.

Result:
- Clear workflow: (1) load/clean dataset, (2) EDA + figures, (3) baseline models, (4) report write-up aligned to evidence.

Decision / Interpretation:
- Use the notebook as the “single source of truth” for dataset size, plots, and metrics; the report files should reference notebook outputs.

Next:
- Build a robust dataset loader that standardizes human vs LLM schema.

---

## Phase 1 notebook: loading + standardization
Intent:
- Create a unified dataset with consistent fields for analysis and modeling.

Action:
- Implemented a loader that reads all four CSV files (human legit/phishing, LLM legit/phishing).
- Standardized schema into a unified dataframe with columns: `text`, `subject`, `body`, `source`, `is_llm`, `is_phishing`, `url_count`.
- Created `text = subject + "\n\n" + body` for a single modeling field.

Result:
- Notebook can construct `df_all` consistently across sources.

Decision / Interpretation:
- Keep preprocessing simple and transparent for Phase 1; avoid heavy NLP normalization until Phase 2.

Next:
- Run EDA features (length, punctuation, URL indicators) and export proposal figures.

---

## Phase 1 EDA: features + robust plots
Intent:
- Produce evidence for “measurable differences” between human vs LLM emails.

Action:
- Engineered lightweight features: character length, word count, average word length, punctuation counts (!, ?), type-token ratio (TTR), and URL-based features.
- Generated boxplots and robust/log-scale plots to handle heavy-tailed length distributions.
- Saved key figures to `results/phase1_length_distributions_log.png` and `results/phase1_url_features.png`.

Result:
- Observed strong distribution differences: human emails show high variability and outliers; LLM emails are more consistent.

Decision / Interpretation:
- Report medians/IQR and log-scale plots rather than only means, to avoid outliers dominating conclusions.

Next:
- Train baseline TF-IDF classifiers for human vs LLM (start of Phase 2).

---

## Data integrity issue: LLM phishing count mismatch
Intent:
- Explain and fix why LLM data did not match expected counts.

Action:
- Investigated `data/archive_data/llm-generated/phishing.csv` and found it malformed (effectively one physical line / broken CSV structure).
- Confirmed that naive parsing with `on_bad_lines='skip'` drops many samples.
- Implemented a repair loader that reconstructs rows using a heuristic split on label boundaries and wrote `data/archive_data/llm-generated/phishing_repaired.csv`.

Result:
- Recovered the intended 1,000 LLM-phishing rows, restoring the full dataset size to 4,000 (2,000 human + 2,000 LLM).

Decision / Interpretation:
- Prefer repaired data over silently dropping rows. Document the repair clearly so results are reproducible.

Next:
- Re-run EDA and baseline models to ensure all saved artifacts reflect the repaired dataset.

---

## Phase 2 baseline models (TF-IDF)
Intent:
- Establish strong, reproducible baselines for human vs LLM classification.

Action:
- Trained TF-IDF + Logistic Regression and TF-IDF + Linear SVM using a stratified 80/20 split (fixed random seed).
- Exported confusion matrices and summary metrics.

Result:
- Very high performance on this dataset:
	- Logistic Regression accuracy ≈ 0.99875
	- Linear SVM accuracy = 1.0
- Saved metrics to `results/baseline_metrics.json`.

Decision / Interpretation:
- Near-perfect results may indicate strong dataset-specific cues (e.g., reply formatting/boilerplate/length). This is promising but requires sanity checks.

Next:
- Add validation/sanity checks (length-controlled evaluation, phishing-only/legit-only splits) and document limitations in the report.

---

## Phase 2 extension: "negative learning" via one-class anomaly detection
Intent:
- Implement a “negative learning” interpretation that matches a cybersecurity constraint: train on trusted normal data (human) and detect LLM emails as anomalies.

Action:
- Implemented a one-class pipeline in the notebook: TF‑IDF → Truncated SVD (200 comps) → StandardScaler → One-Class SVM (RBF, `nu=0.05`).
- Trained only on human emails (80% human train; de-duplicated by normalized text to reduce template leakage).
- Evaluated on a balanced set: held-out human test vs equal number of sampled LLM emails.
- Exported metrics and ROC/PR curves:
	- `results/oneclass_svm_metrics.json`
	- `results/oneclass_svm_curves.png`

Result:
- The one-class model fails to detect LLM emails in this setup:
	- Accuracy ≈ 0.4775
	- Precision/Recall/F1 for LLM detection = 0
	- ROC AUC ≈ 0.0245, PR AUC ≈ 0.3103
	- Confusion matrix (true rows [human, llm], pred cols [human, llm]): [[382, 18], [400, 0]]

Decision / Interpretation:
- Under TF‑IDF + SVD features, LLM emails in this dataset are not outliers relative to human emails; in fact they appear more “inlier-like” than human (human has diverse real-world formatting).
- This negative result is still useful evidence: novelty detection is not reliable here without stronger representations, alternative anomaly models, or a different definition of “normal”.

Next:
- Document Method 3 + interpretation in the short report.
- If time permits, explore alternative anomaly detectors (e.g., Isolation Forest) and/or richer features (stylometry + structure) to see if one-class detection becomes viable.

Update:
- Ran a salvage sweep (nu tuning + Isolation Forest) in both TF-IDF+SVD space and a small stylometry/structure feature space.
- Saved full results to `results/oneclass_salvage_metrics.json`. Outcome remained weak (text-space recall = 0; best stylometry-space recall ≈ 0.02).

---

## Methods justification refinement + notebook method expansion
Intent:
- Address reviewer concerns on (1) binary setup clarity, (2) TF-IDF dimensionality, (3) linear-only SVM choice, and (4) adding autoencoder anomaly detection.

Action:
- Updated report method section to explicitly justify:
	- binary target definition (human vs LLM, phishing as auxiliary attribute)
	- why linear LR/SVM are valid text baselines
	- why LR has no native kernel form and how nonlinearity can still be introduced
	- why dimensionality control is required for TF-IDF
	- why a reconstruction-based anomaly model (autoencoder) is a meaningful alternative formulation
- Extended notebook methods with new executable experiments:
	- TF-IDF + chi-square feature selection + Logistic Regression
	- TF-IDF + SVD + Logistic Regression
	- TF-IDF + SVD + RBF SVM
	- autoencoder-style anomaly detection using MLP reconstruction on human-only training data
- Added JSON artifact outputs for the new method block:
	- `results/method_comparison_extended.json`
	- `results/autoencoder_anomaly_metrics.json`

Result:
- The project now has a clearer methods narrative and a broader empirical framework that directly addresses dimensionality, linearity, and anomaly-structure concerns.

Decision / Interpretation:
- Keep LR + Linear SVM as core required baselines for comparability.
- Treat nonlinear and autoencoder methods as stress/extension checks rather than replacements.

Next:
- Execute the new notebook cells and report comparative trade-offs (accuracy vs complexity vs robustness) in the final write-up.

Update:
- Executed the new method-update notebook cell and generated:
	- `results/method_comparison_extended.json`
	- `results/autoencoder_anomaly_metrics.json`
- Key measured outcomes:
	- TF-IDF + chi2 + LogReg (20,000 features) matched full TF-IDF LogReg performance (accuracy 0.99875).
	- TF-IDF + SVD + LogReg (300 dims) remained high (accuracy 0.9975).
	- TF-IDF + SVD + RBF SVM (300 dims) reached accuracy 0.99875 but had heavier prediction cost than linear baselines.
	- Autoencoder anomaly detection improved score-level ranking (ROC AUC ~0.577) versus one-class SVM, but thresholded detection remained weak (q95 recall ~0.0075; q99 recall 0).
- Updated `report/Full Project Report.md` sections (Results/Conclusion) with these measured values.