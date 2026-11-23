# Malaria RBC Screener (Scratch-Built CNN + TensorFlow.js)

A lightweight Convolutional Neural Network (CNN) trained **from scratch** on thin blood smear microscopy images to classify red blood cells as **Parasitized** or **Uninfected**, deployed as a zero-backend web application running 100% locally in the browser via **TensorFlow.js**.

**[Try the Live Demo](https://guess098.github.io/malaria-cnn-web/)**

---

## Highlights

* **Trained from scratch:** Randomly initialized (`he_normal`) ~94k-parameter CNN—no ImageNet transfer learning or pretrained backbones.
* **Privacy-first client-side inference:** The model (~400 KB) downloads once and runs directly on the visitor's device via TensorFlow.js; uploaded images never leave the browser.
* **Interactive screening threshold:** Includes a live decision-threshold slider (`0.10`–`0.90`) to explore the clinical trade-off between **Sensitivity** and **Specificity** without retraining.

---

## Validation Performance

Evaluated on **5,511 unseen validation cells** (20% stratified holdout from the NIH Malaria dataset, `seed=42`, default threshold `0.50`):

| Metric | Score |
| :--- | :--- |
| **Sensitivity (Recall)** | **94.64%** |
| **Specificity** | **97.37%** |
| **Overall Accuracy** | **96.01%** |

### Confusion Matrix (n = 5,511)

| | Predicted: Parasitized | Predicted: Uninfected |
| :--- | :---: | :---: |
| **Actual: Parasitized** | **2,593** (True Positive) | **147** (False Negative) |
| **Actual: Uninfected** | **73** (False Positive) | **2,698** (True Negative) |

> **Clinical note:** At the default `0.50` cutoff, the model favors high specificity (`97.37%`). Lowering the decision threshold in the web app increases sensitivity to capture more borderline infected cells at the cost of additional false alarms.

---

## Model Architecture & Training

Built and trained in **Keras / TensorFlow** on a Google Colab T4 GPU:

* **Dataset:** [NIH Malaria Cell Images Dataset](https://lhncbc.nlm.nih.gov/LHC-downloads/downloads.html#malaria-datasets) — 27,558 balanced thin blood smear images (`22,047` train / `5,511` validation), resized to `64×64×3`.
* **Preprocessing & Augmentation:** Pixel rescaling (`1/255`), plus training-only random horizontal/vertical flips and random rotations (`0.25`) to leverage microscopy rotational invariance.
* **Backbone (~94,273 parameters):**
  1. `Conv2D(32, 3×3)` → `BatchNormalization` → `ReLU` → `MaxPooling2D(2×2)`
  2. `Conv2D(64, 3×3)` → `BatchNormalization` → `ReLU` → `MaxPooling2D(2×2)`
  3. `Conv2D(128, 3×3)` → `BatchNormalization` → `ReLU` → `MaxPooling2D(2×2)`
  4. `GlobalAveragePooling2D` → `Dropout(0.3)` → `Dense(1, activation="sigmoid")`
* **Optimization:** Adam (`lr=0.001`), Binary Cross-Entropy loss, batch size `32`, `10` epochs.
* **Web Deployment:** Stripped training-only layers (`RandomFlip`, `RandomRotation`, `Dropout`), exported as a TensorFlow SavedModel, converted to a TensorFlow.js `GraphModel`, and hosted statically via GitHub Pages.

---

## Repository Structure

```text
malaria-cnn-web/
├── index.html              # Single-file UI + TensorFlow.js inference script
├── model.json              # Converted TF.js GraphModel architecture
├── group1-shard1of1.bin    # Trained CNN weights (~380 KB)
└── README.md               # Project documentation
```

---

## Usage & Limitations

* **Expected input:** Upload a **single cropped red blood cell** from a Giemsa-stained thin blood smear (matching the NIH dataset format). Full wide-field microscope images are not supported.
* **Disclaimer:** This project is an **educational prototype** built to explore scratch-trained medical imaging models and browser-based deployment. It is **not** a validated medical device and must not be used for clinical diagnosis or patient care.

## Dataset Attribution

Cell images originate from the **National Library of Medicine (NIH) Lister Hill National Center for Biomedical Communications** (Rajaraman et al., *PeerJ*, 2018).
