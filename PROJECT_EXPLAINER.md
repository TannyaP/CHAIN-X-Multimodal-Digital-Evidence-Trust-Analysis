# CHAIN-X: Project Explanation

Course: Forensic Analytics (CSE4074), VIT Chennai, Fall 2026-27. Professor: Dr. S. Johanan Joysingh. Team: Arista Vania, Tannya Pasricha.*

## 1. The gap (problem statement)

1. **Fragmented evidence.** Digital evidence spans images, video, text reports, metadata and multi-party custody logs.
2. **Siloed tools.** Existing tools check one modality: image tampering *or* video deepfakes *or* log audits.
3. **No holistic trust question.** Nothing asks how trustworthy *all* the evidence is together, across the whole chain of custody.
4. **Manual, unscalable review.** Cross-checking media, reports and transfer logs by hand is slow and error-prone for courts and investigators.

## 2. Objectives

1. Design a unified pipeline integrating image, video, NLP, metadata and custody-graph analytics.
2. Formalize two metrics: Evidence Health Index (EHI) and Custody Risk Score (CRS).
3. Build a late-fusion, attention-based model that combines heterogeneous features into one risk prediction.
4. Generate human-readable, courtroom-oriented explanations with SHAP.
5. Create and release a synthetic multimodal chain-of-custody benchmark.
6. Deliver a lightweight interactive dashboard that runs on ordinary hardware.

## 3. What is new

- One evidence-trust score across image, video, NLP, metadata and custody graph, instead of single-modality forgery detection.
- Two new quantified metrics, EHI and CRS.
- Cross-modal contradiction detection (e.g. report timestamp vs. EXIF timestamp).
- Attention-gated fusion that shows which modality drove a score.
- SHAP explanations turned into plain-language justifications.
- Custody transfers modeled as a directed graph (NetworkX) for structural anomaly detection.
- An open synthetic benchmark, because no public multimodal custody dataset is known to us.

## 4. Review of existing systems

| Approach | Gap it leaves open |
|---|---|
| Image forensic detectors (ELA, CNNs on CASIA/COVERAGE) | Ignore video, reports and custody; an image can pass while its log is falsified |
| Video deepfake detectors (FaceForensics++, DFDC) | No cross-check against transcripts, metadata or custody |
| NLP contradiction / fraud-text detectors | Analyze text alone; cannot verify claims against the media |
| Metadata / EXIF auditing scripts | Check a file's own metadata only; not related to report content or transfers |
| Custody software (FTK, EnCase) | No AI trust scoring, no graph anomaly detection, no explanations |
| Audio-visual consistency research | Video+audio only; excludes documents, metadata and custody logs |

## 5. System overview and pipeline

![Architecture](figures/fig_arch.png)

1. **Ingestion.** The evidence package (.zip) is unpacked and routed to engines.
2. **Per-modality features (parallel).**
   - *Image:* ELA map, noise variance, frozen EfficientNet-B0 embedding (1280-d).
   - *Video:* 1 fps frames, MoViNet-A0 / X3D-S embedding, SSIM tamper flags.
   - *NLP:* spaCy entities (officer, timestamp, device, hash, location); DeBERTa-v3 / Legal-BERT embeddings for contradictions.
   - *Metadata:* EXIF / hash / codec features scored by XGBoost or Random Forest.
   - *Graph:* in/out-degree anomalies, temporal edge delays, node reputation on the custody graph.
3. **Cross-modal check.** Pairwise comparison (timestamp, device, ...) gives a penalty term.
4. **Fusion.** Concatenated vectors go through a two-layer MLP with a softmax attention gate.
5. **Scoring.** EHI = clip(sum of attention-weighted integrity scores minus lambda times the cross-modal penalty, 0, 1); CRS = 100 x sigmoid of the graph-aware fusion output.
6. **Explainability.** TreeSHAP / KernelSHAP rank which modality or feature drove the score.
7. **Dashboard.** Streamlit shows the custody graph, EHI/CRS gauges and SHAP cards.

## 6. Training and evaluation plan

- 80/20 split of a hybrid dataset: CASIA, FaceForensics++, SULFA for single-modality ground truth, plus a synthetic 2,000-case custody set (1,000 healthy, 1,000 anomalous with injected metadata mismatches, graph disruptions and media tampering).
- Metrics: precision, recall, F1, ROC-AUC; single-modality baselines vs. full fusion; ablation with and without the graph engine.

## 7. What has been built and measured

The image engine has been evaluated end-to-end; the other engines and fusion are the next stage (see the status table in the [README](../README.md)).

**Setup.** CASIA v2.0 (12,472 usable images: 7,491 authentic, 4,981 manipulated). Strict source-disjoint split (6,546 source components, zero overlap) giving 8,061 train / 500 validation / 500 test. A 7-channel (RGB + ELA + SRM residual) ResNet-18 U-Net, native resolution, thresholds chosen on validation only.

| Result (test set, n = 500) | Value |
|---|---|
| ROC-AUC / accuracy | 0.9315 / 84.6% |
| Recall / precision / F1 | 0.892 / 0.817 / 0.853 |
| Mean IoU / median IoU | 0.337 / 0.023 |

![Format shortcut](figures/fig_format.png)
![Localization by region size](figures/fig_area.png)

**Evidence survival.** The same 100 images under JPEG, resize, blur and repeated transforms: mean Dice drops from 0.318 to at most 0.036 in every condition. The order of operations matters (JPEG then resize differs from resize then JPEG), which is why the custody record keeps the full transformation sequence.

![Survival](figures/fig_survival.png)

**Channel sensitivity.** With RGB only the model localizes nothing (Dice 0.000); adding ELA and SRM together gives Dice 0.184.

![Channels](figures/fig_ablation.png)

**Qualitative examples** (first two rows succeed, last two miss the region):

![Examples](figures/fig_qual.png)

## 8. Honest limitations

- A "TIFF means manipulated" rule scores 83.9% on the same data, so accuracy is close to a format shortcut.
- Median IoU is 0.023; most images are poorly localized.
- Localization nearly vanishes after mild JPEG recompression.
- The main run trained from random initialization (no internet on Kaggle); a run with a pretrained encoder reached 89.0% accuracy / AUC 0.9456.
- The three-class classifier failed (macro-F1 0.249).
- Physical screen recapture and IMD2020 were not run; no results are claimed.
- Hashing, access control and audit logging are specified, not implemented.

## 9. Chain-of-custody layer

Each item gets an evidence ID and a SHA-256 hash (H0 = SHA256(I0)); working copies and derivatives carry parent IDs, operations, software versions and output hashes; every model run records checkpoint, preprocessing and thresholds. A matching hash shows a file is unchanged, not that it is genuine. Integrity and analytical validity are separate requirements.

## 10. Tech stack

Python 3.10+, PyTorch; OpenCV, timm, Pillow/imagehash; MoViNet-A0 / X3D-S; Hugging Face Transformers, spaCy; exifread, hachoir, XGBoost; NetworkX; SHAP; SQLite; Streamlit; local 16 GB workstation (RTX 3050/4050) or Google Colab / Kaggle.
