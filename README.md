# AI-Powered Network Anomaly Detection System

An unsupervised machine learning system that detects malicious network traffic by learning what *normal* traffic looks like — and flagging anything that deviates from it. Built with a deep learning autoencoder trained on the NSL-KDD network intrusion benchmark dataset.

## Problem

Modern networks generate far too much traffic for humans to manually monitor for threats. Traditional intrusion detection relies on hand-written rules and known attack signatures, which means it only catches attacks someone has already seen before. This project takes a different approach: instead of teaching a model what attacks look like, it teaches a model what **normal** traffic looks like, and flags anything that doesn't fit that pattern — including attack types the model has never explicitly seen.

## Approach

The core of this system is an **autoencoder**, a neural network trained to compress network connection data down to a small internal representation and then reconstruct it back to its original form. Trained exclusively on normal traffic, the model becomes very good at reconstructing normal patterns — but performs noticeably worse when given traffic that doesn't match those learned patterns, such as attacks.

That reconstruction error becomes the anomaly signal: **the worse the model is at rebuilding a connection, the more likely it is to be malicious.**

### Pipeline

1. **Data ingestion** — Loaded the NSL-KDD dataset (125,973 labeled network connections)
2. **Preprocessing** — One-hot encoded categorical features (`protocol_type`, `service`, `flag`), expanding the feature set to 123 numeric columns
3. **Scaling** — Standardized all numeric features (zero mean, unit variance) so no single feature dominated due to scale differences
4. **Train/test split** — Trained exclusively on the 67,343 rows labeled `normal`; evaluated on the full dataset (normal + 11 attack categories including `neptune`, `smurf`, `satan`, and `back`)
5. **Model architecture** — A symmetric autoencoder (123 → 64 → 32 → 16 → 32 → 64 → 123) with a 16-neuron bottleneck, ~21,000 trainable parameters
6. **Training** — Optimized with Adam and Mean Squared Error loss over 20 epochs
7. **Thresholding** — Set the anomaly threshold at the 95th percentile of reconstruction error on normal traffic
8. **Evaluation** — Measured precision, recall, F1-score, and visualized error separation and confusion matrix

## Results

| Metric | Normal | Attack |
|---|---|---|
| Precision | 0.98 | 0.94 |
| Recall | 0.95 | 0.98 |
| F1-score | 0.97 | 0.96 |

**Overall accuracy: 96%**

- Average reconstruction error on normal traffic: **0.042**
- Average reconstruction error on attack traffic: **1.597** (~38x higher)
- Correctly flagged **98% of all attacks** in the test set, with a 6% false-positive rate on normal traffic

These results show a clear, strong separation between normal and malicious traffic based purely on reconstruction error — without the model ever being trained on a single labeled attack example.

### Reconstruction Error Distribution
*(insert chart image here — see Visuals section below)*

### Confusion Matrix
*(insert chart image here — see Visuals section below)*

## Tech Stack

- **Python** — pandas, NumPy
- **scikit-learn** — preprocessing, evaluation metrics
- **TensorFlow / Keras** — autoencoder architecture and training
- **Matplotlib / Seaborn** — visualization
- **Google Colab** — development environment
- **Dataset:** [NSL-KDD](https://www.unb.ca/cic/datasets/nsl.html)

## What This Project Demonstrates

- Applying unsupervised deep learning to a real cybersecurity problem
- Understanding *why* an approach works, not just implementing it (autoencoder reconstruction error as an anomaly signal)
- Proper ML evaluation practices for imbalanced, security-relevant data (precision/recall over raw accuracy)
- End-to-end pipeline thinking: data → preprocessing → modeling → evaluation → interpretation

## Limitations & Future Work

- NSL-KDD is a well-established benchmark, but it's synthetic/simulated traffic from the late 1990s — a production system would need retraining on real, modern network traffic
- Currently a batch-evaluation system rather than a live, real-time detector; a natural next step would be wrapping the trained model in an API to score live traffic
- Could benchmark against additional models (e.g., Isolation Forest, One-Class SVM) to compare detection approaches

## How to Run

1. Open the notebook in Google Colab (`notebooks/` folder)
2. Run all cells in order — the notebook downloads the NSL-KDD dataset automatically
3. Training takes roughly 1–2 minutes on Colab's default runtime

## Author

Built by Soma as a hands-on portfolio project at the intersection of machine learning and cybersecurity.
