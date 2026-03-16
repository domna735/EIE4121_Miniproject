Subject: Re: Methodology clarifications and extension updates

Dear Bonnie,

Thank you very much for your detailed comments. We have now updated both our methodology and experiments accordingly.

Yes, we are using a binary problem setting:
- Class 0: human-generated emails
- Class 1: LLM-generated emails
independent of whether an email is phishing or legitimate. The phishing label is used as an auxiliary analysis to check for dataset shortcuts and distribution shift.

For LR and SVM, you are right that they share TF-IDF features and that TF-IDF can be high-dimensional. We have now added explicit dimensionality-control experiments:
- TF-IDF + chi-square feature selection + Logistic Regression
- TF-IDF + SVD + Logistic Regression

Measured results show that dimensionality can be reduced substantially with little performance loss:
- Full TF-IDF LogReg (92,827 features): accuracy 0.99875
- Chi-square LogReg (20,000 features): accuracy 0.99875
- SVD LogReg (300 dimensions): accuracy 0.9975

We also explored your point on nonlinear decision boundaries for SVM:
- TF-IDF -> SVD -> RBF SVM

This achieved accuracy 0.99875, similar to linear baselines, but with heavier prediction cost. This supports our use of linear models as the practical default in sparse text settings.

For LR kernels: we clarified in the report that standard Logistic Regression does not directly support kernels; nonlinearity must be introduced through feature mappings.

For anomaly detection, we added an autoencoder-style formulation:
- Train on human-only data
- TF-IDF + SVD embedding
- MLP reconstruction model
- Reconstruction error as anomaly score

Results:
- ROC AUC 0.577, AP 0.520 (better ranking than one-class SVM)
- But thresholded detection remains weak (q95 recall 0.0075; q99 recall 0)

So in our current dataset, autoencoder anomaly detection provides an informative alternative formulation, but does not yet outperform supervised LR/SVM for practical detection.

We have updated:
- notebook methods and outputs
- process log
- full report methods/results/conclusion sections

Thank you again for the constructive guidance. It significantly improved the rigor of our method comparison.

Best regards,
Donovan and Chun Hei
