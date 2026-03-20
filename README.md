# TrafficVision

TrafficVision is a traffic-flow classification project built on top of YOLOv8. It detects vehicles in traffic-camera still images, converts detections into vehicle counts, and classifies each scene as `Free`, `Moderate`, or `Heavy`.

The project evolved across three phases:
- Phase 1: proof-of-concept detection and rule-based traffic classification
- Phase 2: labeled calibration, threshold tuning, and quantitative evaluation
- Phase 3: YOLOv8 fine-tuning and downstream re-calibration on updated vehicle counts

## Current Results

### Traffic-State Classification

Phase 2 baseline on 90 labeled images:
- Accuracy: `70.0%`
- Macro F1: `0.6555`
- Moderate-class F1: `0.4615`

Phase 3 after fine-tuning and re-calibration:
- Accuracy: `77.8%`
- Macro F1: `0.7016`
- Moderate-class F1: `0.4000`

Best traffic-state thresholds after fine-tuning:
- `Free <= 8` vehicles
- `Moderate <= 14` vehicles
- `Heavy >= 15` vehicles

### Fine-Tuned Detector

Held-out test split metrics from `fine_tune_local.ipynb`:
- Precision: `0.898`
- Recall: `0.901`
- mAP@0.50: `0.964`
- mAP@0.50:0.95: `0.701`

## Pipeline

The current workflow is:
1. Load a traffic-camera image.
2. Run YOLOv8 vehicle detection.
3. Count detected vehicles after confidence filtering.
4. Map the count to `Free`, `Moderate`, or `Heavy` using calibrated thresholds.
5. Evaluate with confusion matrices, accuracy, and F1 metrics.

## Repository Layout

- `Traffic_Flow_Project_Report.md`: project report with phase-by-phase summary and metrics
- `src/traffic-flow-reporter-poc/demo.ipynb`: proof-of-concept and calibrated traffic classification workflow
- `src/traffic-flow-reporter-poc/fine_tune_local.ipynb`: local fine-tuning, detector evaluation, and before/after comparison
- `src/traffic-flow-reporter-poc/requirements.txt`: Python dependencies
- `src/traffic-flow-reporter-poc/outputs/`: generated label/count CSV files
- `src/traffic-flow-reporter-poc/runs/detect/`: YOLO training and validation artifacts
- `data/Vehicle_Detection_Image_Dataset/`: original detection dataset and metadata
- `data/Vehicle_Detection_Image_Dataset_FineTune/`: dataset split used for fine-tuning

## Key Files and Outputs

Generated during the notebook workflows:
- `src/traffic-flow-reporter-poc/outputs/traffic_labels.csv`
- `src/traffic-flow-reporter-poc/outputs/labeled_counts.csv`
- `src/traffic-flow-reporter-poc/before_after_confusion_matrices.png`
- `src/traffic-flow-reporter-poc/before_after_metrics.png`

Fine-tuning artifacts are saved under:
- `src/traffic-flow-reporter-poc/runs/detect/`

## Setup

1. Create or activate a Python virtual environment.
2. Install dependencies from `src/traffic-flow-reporter-poc/requirements.txt`.
3. Open the notebooks from `src/traffic-flow-reporter-poc/`.

Example:

```powershell
cd src/traffic-flow-reporter-poc
pip install -r requirements.txt
```

## How to Reproduce

### Phase 1 / Phase 2 Workflow

Run `src/traffic-flow-reporter-poc/demo.ipynb` to:
- load the pretrained YOLOv8 model
- run inference on traffic images
- label traffic-state samples
- generate `traffic_labels.csv` and `labeled_counts.csv`
- calibrate traffic thresholds
- evaluate the traffic-state classifier

### Phase 3 Workflow

Run `src/traffic-flow-reporter-poc/fine_tune_local.ipynb` to:
- fine-tune `yolov8n.pt` on the fine-tuning dataset
- evaluate the detector on the held-out test split
- regenerate vehicle counts using the fine-tuned model
- perform grid search for the best traffic-state thresholds
- compare Phase 2 vs Phase 3 confusion matrices and metrics


## Notes

- The project currently uses still images rather than video streams.
- Traffic-state quality depends on both detector quality and threshold calibration.
- `Moderate` remains the hardest class because it lies near the decision boundary between low and high traffic density.

## Summary

The current repository includes a calibrated evaluation pipeline, a fine-tuned detector, and measured improvements in end-to-end traffic-state classification performance.
