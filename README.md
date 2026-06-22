# 📧 Phishing Email Detection Model

A machine learning project built with **Scikit-learn** that classifies emails as **Phishing** or **Safe** based on their textual content and URL/structural features. Built as a mini project to explore applied ML for cybersecurity.

> ⚠️ **Disclaimer**: This project uses a synthetically generated dataset (see [Dataset](#-dataset) below) for demonstration and learning purposes. For production use, train on a real labeled phishing corpus and validate carefully before relying on it to filter real email.

---

## ✨ Features

- **Synthetic dataset generator** — produces realistic phishing and legitimate emails (sender, subject, body) with no external download required
- **Feature extraction** covering:
  - TF-IDF text features (subject + body, 1–2 grams)
  - Number of URLs in the email
  - IP-address-based URLs (`http://192.168.x.x/...`)
  - Suspicious top-level domains (`.xyz`, `.top`, `.click`, `.ru`, etc.)
  - Brand-lookalike domains (`paypa1`, `amaz0n`, etc.)
  - Suspicious/urgency keyword counts ("verify", "urgent", "suspended"...)
  - Sender domain mismatch detection (brand name in a non-official domain)
  - Structural signals: subject/body length, exclamation marks, uppercase word count, HTTPS usage
- **Model training** — Logistic Regression (default), Naive Bayes, or Random Forest
- **Evaluation** — accuracy, precision, recall, F1, ROC-AUC, full classification report
- **Visual report** — confusion matrix heatmap + ROC curve saved as PNG
- **Interpretability** — prints the top words/features pushing predictions toward Phishing vs Safe
- **Standalone inference script** — classify any new email from the command line

---

## 📦 Requirements

```bash
pip install -r requirements.txt
```

Or individually: `pandas`, `numpy`, `scikit-learn`, `scipy`, `matplotlib`, `seaborn`, `joblib`.

---

## 🚀 Quick Start

### 1. Generate the dataset
```bash
python3 generate_dataset.py
```
This creates `emails_dataset.csv` with 600 labeled emails (300 phishing / 300 safe).

### 2. Train the model
```bash
python3 train_model.py
```
This trains a Logistic Regression model by default, prints evaluation metrics, and saves:
- `phishing_model.joblib` — the trained model bundle
- `phishing_model_evaluation.png` — confusion matrix + ROC curve

### 3. Classify a new email
```bash
python3 predict_email.py \
  --sender "security@paypa1-support.com" \
  --subject "Urgent: verify your account" \
  --body "Click here immediately to verify your account or it will be suspended: http://paypa1-verify.com/login"
```

Output:
```
==================================================
 Email Classification Result
==================================================
Prediction: 🚨 PHISHING
Phishing probability: 99.99%
==================================================
```

---

## ⚙️ Options

### `train_model.py`
| Flag | Description | Default |
|------|-------------|---------|
| `--data` | Path to CSV with `sender,subject,body,label` columns | `emails_dataset.csv` |
| `--model` | `logistic`, `naive_bayes`, or `random_forest` | `logistic` |
| `--test-size` | Fraction of data held out for testing | `0.25` |
| `--out-prefix` | Prefix for saved model/report files | `phishing_model` |

```bash
python3 train_model.py --model random_forest --out-prefix rf_model
```

### `predict_email.py`
| Flag | Description |
|------|-------------|
| `--model` | Path to saved `.joblib` model bundle |
| `--sender` | Sender email address |
| `--subject` | Email subject line |
| `--body` | Email body text |

---

## 📊 Sample Results

Using the default Logistic Regression model on the synthetic dataset (held-out test set, split so test-set email phrasing was never seen during training):

| Metric | Score |
|--------|-------|
| Accuracy | **98.3%** |
| Precision | 100% |
| Recall | 97.5% |
| F1-score | 98.7% |
| ROC-AUC | 99.9% |

**Confusion Matrix:**

|  | Predicted Safe | Predicted Phishing |
|---|---|---|
| **Actual Safe** | 60 | 0 |
| **Actual Phishing** | 3 | 115 |

The model makes zero false-positive errors on this test set (no safe email misclassified as phishing) and misses only 3 of 118 phishing emails — these tend to be the more subtle, business-toned phishing examples that don't use obvious urgency language.

**Top features pushing toward Phishing:** suspicious TLD, sender domain mismatch, suspicious keyword count, number of URLs, brand lookalikes.

**Top features pushing toward Safe:** longer body length, HTTPS usage, common legitimate phrasing (e.g. "shipping address," "info").

> Note: results vary by model. Naive Bayes and Random Forest reach 100% accuracy on this dataset, which is expected behavior for a relatively small templated dataset rather than evidence the problem is "solved" — see [Limitations](#-limitations).

---

## 🧠 How It Works

1. **Generate/Load data** — emails with sender, subject, body, and a `phishing`/`safe` label.
2. **Extract features** — combine engineered numeric features with TF-IDF vectors of the email text.
3. **Split train/test** — using a group-aware split (by underlying email template) so the test set isn't contaminated by near-duplicate phrasing seen in training, giving an honest accuracy estimate.
4. **Train** — fit a classifier (Logistic Regression by default) on the combined feature matrix.
5. **Evaluate** — compute accuracy, precision, recall, F1, ROC-AUC, and generate a confusion matrix + ROC curve plot.
6. **Save & reuse** — persist the model, vectorizer, and scaler together so new emails can be classified without retraining.

---

## 📁 Project Structure

```
phishing_detector/
├── generate_dataset.py      # Synthetic dataset generator
├── feature_extraction.py    # Feature engineering (URLs, keywords, domains, etc.)
├── train_model.py           # Training + evaluation pipeline
├── predict_email.py         # Classify a new email using the saved model
├── requirements.txt
├── emails_dataset.csv        # Generated dataset (created by generate_dataset.py)
├── phishing_model.joblib     # Saved trained model (created by train_model.py)
└── phishing_model_evaluation.png  # Confusion matrix + ROC curve (created by train_model.py)
```

---

## 🗂 Dataset

This project ships with a **synthetic dataset generator** rather than a real-world dataset, so it runs out-of-the-box with no download required. The generator creates phishing emails using common real-world patterns (urgency, account suspension threats, brand impersonation, look-alike domains) alongside realistic legitimate business/personal emails — including some intentionally "hard" examples on both sides (legitimate emails that use urgent language; phishing emails with a calm, business-like tone) so the classification task isn't trivially easy.

For a real project, swap in a genuine labeled dataset such as:
- [Kaggle: Phishing Email Detection](https://www.kaggle.com/datasets) (search "phishing email")
- The [Nazario Phishing Corpus](https://monkey.org/~jose/phishing/)
- The [Enron-Spam dataset](https://www2.aueb.gr/users/ion/data/enron-spam/) (for legitimate/spam baseline)

As long as your CSV has `sender`, `subject`, `body`, and `label` columns (with labels `phishing`/`safe`), `train_model.py` will work on it directly.

---

## ⚠️ Limitations

- The shipped dataset is synthetic/templated — real-world phishing is more varied, and results on real data will likely be lower than shown here
- Feature engineering uses simple heuristics (keyword lists, regex patterns) rather than a live threat-intelligence feed
- No handling of HTML email bodies, embedded images, or attachments — text-only analysis
- Random Forest / Naive Bayes hitting 100% accuracy on the synthetic set is a sign of the dataset's limited diversity, not a claim that phishing detection is a solved problem
- This is an educational tool, not a production email security filter

---

## 🛣️ Possible Extensions

- Train on a real-world dataset and report results
- Add cross-validation instead of a single train/test split
- Incorporate WHOIS/domain-age lookups as a feature
- Try deep learning approaches (e.g. fine-tuned transformer on email text)
- Build a simple web UI (Flask/Streamlit) for interactive classification

---

## 📚 Learning Outcomes

This project demonstrates:
- How to engineer domain-specific features (URLs, keywords, sender domains) for a security ML task
- How to combine structured features with TF-IDF text vectors in Scikit-learn
- Why naive train/test splitting can produce misleadingly perfect accuracy (and how grouping by template avoids this)
- How to evaluate a classifier properly: accuracy alone isn't enough — precision, recall, F1, and the confusion matrix tell the fuller story
- How to interpret a linear model's coefficients to understand *why* it makes its predictions

---

## 📝 License

This project is provided for educational purposes.
