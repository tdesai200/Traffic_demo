# AI-Based Traffic Flow Classification Using Traffic Camera Images

Project Progress Report  
Generated on: 2026-03-14

## 1\. Problem Statement

This project aims to classify traffic flow conditions (Free, Moderate, Heavy) using still images from existing traffic camera infrastructure. Traditional traffic reporting systems rely on user reports or physical road sensors, which can be delayed or costly to maintain. Our approach leverages computer vision techniques to detect and count vehicles from images and automatically determine traffic density levels.

## 2\. Phase 1 – Proof of Concept Development

During phase 1, we developed a working proof-of-concept system using a pretrained YOLOv8 object detection model. The system detects vehicles in traffic camera images, counts them, and assigns traffic states based on simple threshold rules.

Key accomplishments in Phase 1 include:

\- Integrated pretrained YOLOv8 model for vehicle detection.

\- Processed validation images and generated bounding box overlays.

\- Implemented initial traffic classification thresholds.

\- Produced CSV outputs containing vehicle counts per image.

At this stage, the system functioned as a visual demo without quantitative validation. Traffic thresholds were heuristic and not calibrated using labeled data.

## 3\. Phase 2 – Calibration and Evaluation

In phase 2, we transitioned from a demonstration-based approach to a data-driven evaluation framework. We manually labeled a subset of images with ground-truth traffic states and introduced confidence-filtered vehicle counting to reduce false positives.

Key improvements in Phase 2 include:

\- Added confidence threshold filtering to stabilize vehicle counts.

\- Increased inference image size to improve detection of small vehicles.

\- Created a manually labeled calibration dataset.

\- Implemented grid-search optimization to tune traffic classification thresholds.

\- Evaluated performance using accuracy, macro F1-score, and confusion matrix.

## 4\. Results Summary

Using the calibrated thresholds from Phase 2, the system achieved **70.0% accuracy** with a **macro F1-score of 0.655** on the 90 labeled traffic-state images used for evaluation. Performance was strongest in **Free** scenes, while **Moderate** remained the hardest class because it sits between low- and high-density traffic conditions.

## 5\. Technical Architecture

The system pipeline consists of the following stages:  
1\. Image Input (Traffic Camera Still Image)  
2\. YOLOv8 Vehicle Detection  
3\. Confidence-Filtered Vehicle Counting  
4\. Calibrated Threshold-Based Traffic Classification  
5\. Evaluation and Visualization Outputs

## 6\. Key Insights from Phase 1–2

\- Vehicle count alone is sensitive to detection thresholds.

\- Calibration significantly improves classification reliability.

\- Moderate traffic is inherently harder to classify than extreme conditions.

\- Image resolution impacts small-vehicle detection accuracy.


## 7. Phase 3 - Fine-Tuning YOLO and Boundary Refinement

Phase 3 focused on improving upstream vehicle detection so that downstream traffic-state classification (Free / Moderate / Heavy) becomes more reliable, especially for borderline scenes that are often labeled **Moderate**.

### 7.1 Why Fine-Tune?
In Phase 2, traffic state was derived from **vehicle counts** produced by the detector, then mapped to Free/Moderate/Heavy via calibrated thresholds. That means any missed detections (false negatives) directly reduce counts and can shift a scene from Moderate to Free or Heavy to Moderate. Improving **recall** is therefore the most impactful way to stabilize counts.

### 7.2 Fine-Tuning Setup (Local)
- Base model: **YOLOv8n** pretrained weights (`yolov8n.pt`)
- Training type: **transfer learning / fine-tuning** (not training from scratch)
- Training epochs: **20**
- Image size: **640** (CPU-friendly; can be increased later to improve small-object recall)
- Batch size: **16** (tuned for local memory limits)
- Data split: created a refreshed **train/val/test** split to support unbiased evaluation on a held-out **test** set.

### 7.3 Detector Evaluation (Object Detection Metrics)
After fine-tuning, the detector was evaluated on the held-out **test** split and achieved the following metrics:

- Precision: **0.898**
- Recall: **0.901**
- mAP@0.50: **0.964**
- mAP@0.50:0.95: **0.701**

Interpretation:
- **Recall (0.901)** indicates the model detects most vehicles present in the scene, reducing under-counting.
- **mAP@0.50 (0.964)** indicates strong localization and classification at the standard IoU threshold.
- **mAP@0.50:0.95 (0.701)** is stricter; this is a solid result for a compact model (YOLOv8n) on a domain-specific traffic-camera dataset.

### 7.4 Downstream Re-Calibration (Traffic-State)
Using the fine-tuned detector, we regenerated vehicle counts for labeled images and re-ran the boundary tuning workflow (grid search over thresholds). The post-fine-tune end-to-end results were:

Best thresholds found by grid search:
- **Free <= 8 vehicles**
- **Moderate <= 14 vehicles**

Confusion matrix (true rows x predicted columns; order: Free, Moderate, Heavy)

```
[[35, 0, 0],
 [ 9, 6, 5],
 [ 2, 4,29]]
```

- Accuracy: **70 / 90 = 0.7778 (77.8%)**
- Macro F1-score: **0.7016**

Class-level precision/recall:
- **Free**: Precision = **0.7609**, Recall = **1.0000**, F1 = **0.8642**
- **Moderate**: Precision = **0.6000**, Recall = **0.3000**, F1 = **0.4000**
- **Heavy**: Precision = **0.8529**, Recall = **0.8286**, F1 = **0.8406**

Key takeaway:
- Fine-tuning improved overall accuracy and strengthened separation between **Free** and **Heavy**.
- **Moderate** remains the hardest class because it sits at the boundary between low and high traffic density; small count changes can flip the label.

### 7.5 Before vs After Fine-Tune (Phase 2 vs Phase 3)
Phase 2 traffic-state results (baseline calibration on the same 90 labeled images):
- Accuracy: **0.7000 (70.0%)**
- Macro F1: **0.6555**
- Moderate F1: **0.4615**

Phase 2 confusion matrix:

```
[[35, 0, 0],
 [10, 9, 1],
 [ 6,10,19]]
```

Phase 3 traffic-state results (after fine-tune + re-calibration):
- Accuracy: **0.7778 (77.8%)**
- Macro F1: **0.7016**
- Moderate F1: **0.4000**
- Most remaining errors are **Moderate to Free** and **Moderate to Heavy**.

Overall, fine-tuning the YOLOv8 detector for the project's traffic-camera environment materially improved upstream detection quality and raised end-to-end traffic-state accuracy from **70.0%** to **77.8%**. The updated system shows clearer separation between **Free** and **Heavy** traffic, with the remaining mistakes concentrated in borderline **Moderate** scenes.
