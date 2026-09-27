# AI Engineering Pipeline: Encoding, Decoding, and Model Inference

This document details the text processing pipeline, feature representation, inference flow, and interpretability mechanisms implemented in **Social Sentinel**.

---

## 1. Pipeline Architecture Overview

The system processes incoming unstructured text (emails, SMS) through a deterministic feature transformation pipeline before routing it to the classification models.

```text
[ Raw Text Input ]
        │
        ▼
[ Text Preprocessing & Cleaning ]
        │
        ▼
[ Feature Encoding: N-Gram TF-IDF Vectorization ]
        │
        ├───► [ Primary Classifier: Status Prediction (Phishing vs. Safe) ]
        │            │
        │            ▼
        │     [ Calibrated Probabilities & Binary Verdict ]
        │
        ├───► [ Auxiliary Classifiers: Tactical Feature Detectors ]
        │            │
        │            ▼
        │     [ Flags: Urgency, Authority, Manipulation ]
        │
        └───► [ Decoding & Signal Attribution ]
                     │
                     ▼
              [ Active N-Gram & Trigger Attribution ]

```

---

## 2. Text Ingestion & Preprocessing

Before numerical transformation, raw text undergoes token-level cleaning to normalize lexical variance across attack vectors:

* **Lowercasing:** Eliminates duplicate vocabulary states caused by irregular casing (e.g., `URGENT` vs `urgent`).
* **Noise & Whitespace Normalization:** Strips superfluous whitespace, hidden unicode characters, and formatting artifacts.
* **Preservation of Lexical Urgency Markers:** Special characters that frequently signal psychological manipulation or urgency (such as exclamation marks, currency signs, and punctuation bursts) are either retained or tokenized rather than silently stripped.

---

## 3. Encoding: Numerical Representation (Vectorization)

Social Sentinel uses **Term Frequency-Inverse Document Frequency (TF-IDF)** to convert text documents into high-dimensional, sparse feature matrices.

### Mathematical Formulation

Given a term $t$ and a document $d$ within a corpus $D$:

1. **Term Frequency ($\text{TF}$):**

$$\text{TF}(t, d) = \frac{f_{t,d}}{\sum_{t' \in d} f_{t',d}}$$



Where $f_{t,d}$ is the raw count of term $t$ in document $d$.
2. **Inverse Document Frequency ($\text{IDF}$):**

$$\text{IDF}(t, D) = \log\left(\frac{1 + \vert{}D\vert{}}{1 + \vert{}\{d \in D : t \in d\}\vert{}}\right) + 1$$



This penalizes ubiquitously occurring words across the corpus while boosting the weights of terms specific to social engineering domains.
3. **TF-IDF Weighting & Vector Output:**

$$\text{TF-IDF}(t, d, D) = \text{TF}(t, d) \times \text{IDF}(t, D)$$



The resulting sparse vector $\mathbf{x} \in \mathbb{R}^{\vert{}V\vert{}}$ is L2-normalized:

$$\mathbf{x}_{\text{norm}} = \frac{\mathbf{x}}{\Vert{}\mathbf{x}\Vert{}_2}$$



### Vectorizer Configuration

* **N-gram Range (`ngram_range=(1, 2)`):** Captures single vocabulary tokens (unigrams like `"verify"`, `"suspend"`) as well as multi-word tactical phrases (bigrams like `"immediate action"`, `"account locked"`, `"wire transfer"`).


* **Sublinear TF Scaling:** Dampens the influence of repeatedly spammed words within a single message by mapping frequency $f \to 1 + \log(f)$.
* **Sparsity Handling:** Input representations are stored as Compressed Sparse Row (CSR) matrices, keeping inference memory footprints minimal and eliminating GPU requirements.



---

## 4. Inference & Model Scoring

The pipeline executes two parallel evaluation tracks over the encoded vector $\mathbf{x}_{\text{norm}}$:

### Track A: Primary Threat Classification

Determines the global label $y \in \{\text{safe}, \text{phishing}\}$:

* **Confidence Calibration:** Computes posterior class probabilities $P(y = \text{phishing} \mid \mathbf{x})$ rather than a hard threshold label.


* **Model Flexibility:** Evaluates or switches between Multinomial Naive Bayes, Logistic Regression, Support Vector Machines (linear hyperplane margin), or Random Forest ensembles.



### Track B: Multi-Label Tactical Feature Extraction

Evaluates three secondary binary classifiers to isolate behavioral levers used in social engineering:

1. **Urgency Detector (`urgency_flag`):** Evaluates temporal pressure, rapid deadlines, and severe consequences for delay.


2. **Authority Exploitation Detector (`authority_flag`):** Evaluates impersonation of IT administrators, executives, law enforcement, or financial institutions.


3. **Psychological Manipulation Detector (`manipulation_flag`):** Evaluates emotional hooks such as fear, obligation, curiosity, or greed.

---

## 5. Decoding & Signal Attribution

Decoding translates the model’s internal numerical parameters back into human-interpretable security insights.

### Vocabulary Mapping & Signal Extraction

1. **Active Feature Identification:** When an input text produces vector $\mathbf{x}_{\text{norm}}$, non-zero indices are resolved against the vectorizer's vocabulary mapping:

$$\text{Active Indices } \mathcal{I} = \{j \mid x_j > 0\}$$


2. **Weight/Attribution Scoring:** For linear classifiers (e.g., Logistic Regression with weight vector $\mathbf{w}$):

$$\text{Attribution Score}_j = w_j \cdot x_j$$


* Tokens where $w_j \cdot x_j > 0$ provide positive evidence toward a **Phishing** classification.
* Tokens where $w_j \cdot x_j < 0$ represent neutral or **Safe** communication patterns.


3. **Dashboard Output Serialization:** The extracted key signals, along with probability distributions, are passed directly to the Gradio Blocks UI layer for real-time visualization.

Dataset-Driven Feature Calibration & Multi-Flag Optimization

To ensure the TF-IDF vectorizer and dual-track classifiers generalize effectively against zero-day social engineering vectors, feature encoding is calibrated using a balanced, multi-label training corpus stored within the `dataset/` directory.

* **Target Space Alignment:** Training vectors are mapped directly against orthogonal behavioral indicator vectors (`urgency_flag`, `authority_flag`, `manipulation_flag`), allowing the auxiliary regression heads to isolate psychological levers independently of the primary threat label.
* **Class Balance & Noise Mitigation:** Corpus curation combines high-risk spoofed notifications with neutral operational baseline texts. This prevents the high-dimensional sparse space from over-indexing on generic high-frequency words, ensuring high precision and low false-positive rates during real-time Gradio UI inference.
