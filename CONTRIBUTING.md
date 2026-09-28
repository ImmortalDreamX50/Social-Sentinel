# Contributing to Social Sentinel - AI Phishing Analyzer

Welcome! We are excited you want to contribute. This project aims to create an open-source, accessible tool for detecting social engineering attacks using NLP.

## Table of Contents

1. [How You Can Help](#️-how-you-can-help)
2. [Development Setup](#-development-setup)
3. [Project Structure](#project-structure)
4. [Pull Request Process](#-pull-request-process)
5. [Code of Conduct](#-code-of-conduct)

## 🛠️ How You Can Help

We are currently looking for contributions in three areas:
1.  **Data Science:** Improving our training dataset by adding novel phishing templates.
2.  **AI Engineering:** Swapping the current BERT-tiny model for a more robust model (e.g., RoBERTa) or implementing SHAP (SHapley Additive exPlanations) to highlight exactly *which* words triggered the alert.
3.  **Cybersecurity / Threat Intel:** Connecting the app to external APIs (like VirusTotal or URLScan) to check links found within the text.

## 💻 Development Setup

1. Fork the repository and clone it locally.
2. Create a virtual environment: `python -m venv venv`
3. Activate the environment: `source venv/bin/activate`
4. Install dependencies: `pip install -r requirements.txt`
5. Train the model: `python src/modeling/train.py`
6. Run the app: `python src/dashboard/app.py` (or `gradio run src/dashboard/app.py`)

## Project Structure

social-sentinel/
├── config/
│   └── config.yaml # Central hyperparameters, paths and model selection
├── dataset/
│   ├── training-dataset/
│   │   └── social-sentinel-training-dataset-v1.csv # Labelled training corpus
│   └── test-dataset/
│       └── samples.txt # Unlabelled samples for manual evaluation
├── src/
│   ├── modeling/
│   │   ├── train.py # Trains the TF-IDF + classifier pipeline, saves artifact
│   │   └── predict.py # PhishingPredictor wrapper for inference
│   └── dashboard/
│       └── app.py # Gradio web interface
├── models/
│   └── artifacts/ # Generated model artifacts (git-ignored)
├── assets/
│   └── images/ # Diagrams and static assets
├── .github/
│   ├── Issue_Template/ # Issue forms
│   └── PULL_REQUEST_TEMPLATE.md
├── AI-Engineering-Documentation/ # Architecture & pipeline write-up
├── Machine-Learning-Model-Documentation/ # Model evaluation write-up
├── Cybersecurity-Documentation/ # Threat intel & attack-tactic write-up
├── ML-Models/ # Model showcase assets
├── social_sentinel_starter_dataset.csv # Starter/Colab dataset
├── Dockerfile # Containerised runtime
├── requirements.txt # Python dependencies (Gradio, scikit-learn, joblib, etc.)
├── .gitignore # Ignores virtual environments and cache
├── README.md # Project documentation
├── CONTRIBUTING.md # This file
└── LICENSE # MIT License

### Typical workflow

```bash
pip install -r requirements.txt
python src/modeling/train.py     # trains and writes models/artifacts/*.joblib
python src/dashboard/app.py     # or: gradio run src/dashboard/app.py
```

Edit `config/config.yaml` to change the dataset path, TF-IDF settings, or which
classifier is used. See the top-level `README.md` for dataset schema and usage.

## 🔀 Pull Request Process

1. Create a new branch for your feature (`git checkout -b feature/AddShapleyValues`).
2. Ensure the Gradio UI doesn't break by testing locally.
3. Update the `README.md` if you add new Python libraries.
4. Submit a PR with a clear description of the problem you solved.

## 📜 Code of Conduct
Be kind, write clean code, and never upload actual, un-anonymized malicious payloads or PII (Personally Identifiable Information) to the repository.
