# CHAIN-X: A Unified Multimodal Framework for Chain-of-Custody Evidence Trust Analysis

Developed as part of the course on "Forensic Analytics" at Vellore Institute of Technology, Chennai.
by "Arista Vania", "Tannya Pasricha"

Subject Code: CSE4074
Semester: Fall 2026-27
Professor: Dr. S. Johanan Joysingh

---

> **Project explanation** [`docs/PROJECT_EXPLAINER.md`](docs/PROJECT_EXPLAINER.md)
> **Research paper:** `docs/CHAIN-X_IEEE_Research_Paper.docx`

## What is CHAIN-X?

Digital evidence today is a package: images, video, text reports, file metadata and multi-party custody logs. Existing forensic tools check each of these in isolation. CHAIN-X asks a different question: **how trustworthy is all of the evidence together, across the entire chain of custody?**

It analyzes images, videos, reports, metadata and custody records together, detects cross-modal inconsistencies (for example a report stating one timestamp while the EXIF data records another) and custody-graph anomalies (unauthorized nodes, timing gaps), and produces two scores with an explanation of what drove them:

- **Evidence Health Index (EHI)**: weighted integrity score minus a cross-modal contradiction penalty, normalized to [0, 1].
- **Custody Risk Score (CRS)**: graph-aware risk probability, expressed as a percentage.

Explanations use SHAP attribution and are shown on a Streamlit dashboard.

![CHAIN-X architecture](docs/figures/fig_arch.png)

## Project status (honest summary)

| Component | Status |
|---|---|
| Image localization study (ELA + SRM, ResNet-18 U-Net, CASIA v2.0) | Implemented and evaluated (see `notebooks/`) |
| Evidence-survival experiments (JPEG, resize, blur, repeated transforms) | Implemented and evaluated |
| Image features for fusion (ELA, frozen EfficientNet-B0, RF/XGBoost) | Code in `src/image/` (update this row with your actual run results) |
| Metadata, NLP, graph engines, cross-modal check | Update with your actual status |
| Fusion (attention-gated MLP), EHI, CRS, SHAP, dashboard | Update with your actual status |
| Synthetic custody benchmark (2,000 cases) | Planned |
| Physical screen recapture, external dataset (IMD2020) | Not run; no results claimed |

## Key results so far (image engine, CASIA v2.0, 500-image source-disjoint test set)

| Metric | Value |
|---|---|
| ROC-AUC / accuracy | 0.9315 / 84.6% |
| Recall / precision / F1 (manipulated) | 0.892 / 0.817 / 0.853 |
| Mean IoU / **median IoU** | 0.337 / **0.023** |
| Mean Dice, original vs. after JPEG Q80 | 0.318 vs. 0.0004 |

Caveats stated in the paper: a trivial "TIFF means manipulated" rule scores 83.9% on the same data, localization is weak for most images, performance collapses under recompression, and the three-class classifier did not train successfully (macro-F1 0.249).

![Evidence survival](docs/figures/fig_survival.png)

## Repository structure

```
CHAIN-X-Multimodal-Digital-Evidence-Trust-Analysis/
├── README.md                  <- this file
├── docs/
│   ├── PROJECT_EXPLAINER.md   <- project explanation (parallel to the PPT)
│   ├── CHAIN-X_IEEE_Research_Paper.docx
│   └── figures/
├── notebooks/                 <- Kaggle notebooks (image localization + survival)
├── configs/  data/  experiments/  features/  models/  scripts/  src/
├── app_tier1.py  app_tier2.py <- dashboards
└── requirements.txt
```

## Setup

```bash
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

Datasets are not stored in the repository. See `scripts/DOWNLOAD_INSTRUCTIONS.md` for CASIA v2.0 and COVERAGE.

## How to reproduce

1. Get CASIA v2.0 onto disk (`scripts/DOWNLOAD_INSTRUCTIONS.md`), then run `scripts/build_master_csv.py` to create `data/master_index.csv`.
2. Image feature pipeline: `src/image/feature_extractor.py` (GPU, Kaggle/Colab), `src/image/ela.py` (CPU), `src/image/classifier.py`.
3. Localization and evidence-survival study: open the notebook in `notebooks/` on Kaggle with a GPU (Tesla T4 was used; about 60 s per epoch). Enable internet if you want the ImageNet-pretrained encoder; the main reported run had none and trained from random initialization.
4. Dashboards: `streamlit run app_tier1.py` (and `app_tier2.py`).

## Technology stack

Python 3.10+, PyTorch, OpenCV, timm, Pillow, Hugging Face Transformers (DeBERTa-v3 / Legal-BERT), spaCy, exifread, hachoir, XGBoost / scikit-learn, NetworkX, SHAP, SQLite, Streamlit. Runs on a 16 GB workstation (RTX 3050/4050) or Google Colab / Kaggle.

## Limitations

CHAIN-X is forensic decision support. A model output does not by itself establish authenticity or forgery, identify a responsible person, or determine legal admissibility. Hash verification shows a file is unchanged since it was recorded, not that its content is genuine.

## Team

Arista Vania (23MIA1036), Tannya Pasricha (23MIA1130)
Traditional image-forensics approaches primarily analyse original digital image files. However, manipulated images can be displayed on a laptop or mobile screen, photographed using another device, compressed, resized, and redistributed. These processes can introduce distortions that weaken or obscure traces of image manipulation.

SCREEN-TRACE addresses this challenge by combining image tampering classification, forgery localization, and evidence survival analysis to understand how forensic evidence changes under different image transformation conditions.

---

## Project Overview

The project focuses on three major tasks:

1. **Image Tampering Classification:** Classifying images into three categories:
   - Authentic
   - Copy-Move Forgery
   - Image Splicing

2. **Forgery Localization:** Identifying and highlighting potentially manipulated regions within an image using pixel-level segmentation.

3. **Evidence Survival Analysis:** Evaluating how effectively image-forensics models retain their detection and localization capabilities after digital degradation and physical screen recapture.

---

## Problem Statement

Image manipulation has become increasingly accessible with modern editing tools and generative technologies. Manipulated images can be circulated as screenshots, photographed from digital displays, or repeatedly compressed before reaching investigators.

Such transformations may alter important forensic characteristics, making conventional image manipulation detection less reliable.

SCREEN-TRACE aims to investigate whether manipulated regions can still be detected and localized after these transformations and quantify the resulting degradation in forensic performance.

---

## System Architecture

```text
              Input Image
                   |
                   v
          Image Preprocessing
                   |
          +--------+--------+
          |                 |
          v                 v
   Tampering             Forgery
 Classification        Localization
   EfficientNet-B0      U-Net Architecture
          |                 |
          v                 v
   Authentic /          Suspicious
   Copy-Move /          Region Mask
   Splicing
          |                 |
          +--------+--------+
                   |
                   v
          Evidence Survival
              Analysis
                   |
          +--------+--------+
          |                 |
          v                 v
   Digital Degradation   Physical Screen
                         Recapture
          |                 |
          +--------+--------+
                   |
                   v
          Comparative Analysis
                   |
                   v
          Gradio Live Dashboard
```

---

## Technologies and Models

| Component | Technology |
|---|---|
| Programming Language | Python |
| Deep Learning | PyTorch |
| Classification | EfficientNet-B0 |
| Localization | U-Net with lightweight encoder |
| Image Processing | OpenCV, PIL |
| Numerical Processing | NumPy |
| Data Analysis | Pandas |
| Visualization | Matplotlib, Seaborn |
| Model Evaluation | Scikit-learn |
| Interactive Interface | Gradio |
| Development Environment | Kaggle Notebooks |

### Forensic Features

The project explores the use of:

- RGB image information
- Error Level Analysis (ELA)
- High-frequency residual features
- Image compression characteristics
- Spatial and visual inconsistencies

These features are evaluated experimentally rather than assumed to improve performance.

---

## Datasets

The project uses public image-forensics datasets and a small controlled screen-recapture collection.

| Dataset | Purpose |
|---|---|
| CASIA v2.0 | Primary image manipulation classification and localization |
| CASIA Ground Truth Masks | Pixel-level forgery localization |
| IMD2020 | External evaluation and generalization analysis |
| SCREEN-TRACE Recapture Dataset | Controlled physical screen-recapture experiments |

Dataset labels, annotation quality, and image-to-mask correspondence are verified before training and evaluation.

### Physical Screen Recapture

A controlled pilot experiment is conducted using images displayed on a laptop or mobile screen and photographed using a camera or smartphone.

Capture conditions such as lighting, viewing angle, camera distance, and device information are recorded.

The recaptured images are compared with their original digital versions to investigate the effect of physical recapture on forensic detection.

---

## Experimental Analysis

### 1. Digital Degradation

Images are subjected to controlled transformations:

- JPEG compression at different quality levels.
- Image resizing and resampling.
- Gaussian blur.
- Repeated compression.
- Sequential image transformations.

### 2. Physical Screen Recapture

Images are displayed on a digital screen and photographed using another device.

This introduces real-world effects such as:

- Perspective distortion.
- Moiré patterns.
- Reflections and glare.
- Blur and resampling.
- Colour and brightness variations.

### 3. Evidence Survival Measurement

The change in forensic performance is measured using:

\[
\Delta M = M_{original} - M_{transformed}
\]

Where:

- \(M_{original}\) represents the metric obtained from the original image.
- \(M_{transformed}\) represents the metric obtained after degradation or recapture.

A positive value indicates a reduction in the measured performance.

---

## Evaluation Metrics

**Classification:**
- Accuracy
- Balanced Accuracy
- Precision
- Recall
- Macro F1-score
- Confusion Matrix
- AUROC (where applicable)

**Localization:**
- Intersection over Union (IoU)
- Dice Coefficient
- Pixel Precision
- Pixel Recall

**Evidence Survival:**
- Original versus degraded classification performance.
- Original versus recaptured classification performance.
- Localization performance degradation.
- Comparative analysis across transformation conditions.

---

## Interactive Demonstration

SCREEN-TRACE includes a planned interactive forensic dashboard using Gradio.

The dashboard is designed to allow users to:

1. Upload an image for forensic examination.
2. View the predicted manipulation category.
3. Inspect classification confidence scores.
4. Visualize potentially manipulated regions through segmentation masks.
5. Compare original and degraded or screen-recaptured images.
6. Examine changes in forensic performance.

The dashboard provides an accessible demonstration of the complete forensic analysis pipeline.

---

## Research Contribution

The primary research contribution of SCREEN-TRACE is its investigation of image-forensics reliability under screen-laundering conditions.

Unlike an evaluation limited to original digital image files, this project examines the combined problem of:

- Image manipulation classification.
- Pixel-level forgery localization.
- Digital degradation analysis.
- Physical screen-recapture evaluation.
- Evidence survival measurement.

The project aims to provide an experimental understanding of how forensic detection capabilities change when images undergo realistic display-and-camera transformations.

---

## Repository Structure

```text
SCREEN-TRACE/
│
├── README.md
├── PROJECT_EXPLAINER.md
│
├── notebooks/
│   ├── 01_data_preparation.ipynb
│   ├── 02_classification.ipynb
│   ├── 03_localization.ipynb
│   ├── 04_evidence_survival.ipynb
│   └── 05_change_validation.ipynb
│
├── src/
│   ├── models/
│   ├── preprocessing/
│   ├── evaluation/
│   └── inference/
│
├── app/
│   └── app.py
│
├── reports/
│   ├── figures/
│   └── evaluation/
│
├── requirements.txt
└── .gitignore
```

*Note: The structure represents the proposed repository organization and should be updated to match the final implementation.*

---
---

## Experimental Results

### Image-Level Detection Results

The image-level detection model was evaluated on authentic and manipulated images to assess its ability to distinguish between the two categories.

#### Performance Metrics

| Metric | Score |
|---|---:|
| ROC-AUC | 0.9315 |
| Accuracy | 84.6% |
| Balanced Accuracy | 84.6% |
| Precision | 81.68% |
| Recall | 89.2% |
| F1-score | 85.28% |

#### Confusion Matrix

| Actual / Predicted | Authentic | Manipulated |
|---|---:|---:|
| Authentic | 200 | 50 |
| Manipulated | 27 | 223 |

#### Results Interpretation

- **ROC-AUC (0.9315):** Indicates strong discrimination between authentic and manipulated images across classification thresholds.
- **Accuracy (84.6%):** The model correctly classified 84.6% of the evaluated images.
- **Precision (81.68%):** Of the images predicted as manipulated, 81.68% were actually manipulated.
- **Recall (89.2%):** The model successfully identified 89.2% of the actual manipulated images.
- **F1-score (85.28%):** Demonstrates a balance between precision and recall.

The results indicate that the model identifies manipulated images with relatively high recall, making it useful for preliminary forensic screening. However, false positives and false negatives remain, highlighting the need for further validation and complementary localization analysis.

**Note:** These results represent image-level binary detection (Authentic vs. Manipulated). They should not be interpreted as three-class classification or pixel-level localization results.

## Limitations

- The physical screen-recapture collection is exploratory and relatively small.
- Dataset annotations may contain inconsistencies.
- Different cameras, screens, and lighting conditions may affect model performance.
- Model predictions may contain false positives and false negatives.
- A predicted suspicious region does not independently establish that an image has been manipulated.
- Further validation is required before applying the system to real forensic investigations.

SCREEN-TRACE is intended as an academic research prototype and forensic decision-support tool, not a replacement for professional forensic examination.

---

## Project Domain

**Image and Video Analytics | Digital Image Forensics | Deep Learning | Computer Vision**

**SDG Alignment:** SDG 16 – Peace, Justice and Strong Institutions

The project contributes to the broader objective of supporting information integrity and the responsible examination of potentially manipulated digital evidence.

---

