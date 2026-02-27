# TrafficVision

This repository contains a proof-of-concept (POC) for vehicle detection and traffic flow classification using a pretrained YOLOv8 model.

## Project Phases

### Initial Phase

1. **Dataset preparation**
   - The `data/Vehicle_Detection_Image_Dataset` contains training and validation images as well as annotation files in YOLO format.
   - A `data.yaml` file defines class names.

2. **Detection & Visualization Demo**
   - Developed `demo.ipynb` to load the YOLO model (`yolov8n.pt`) and run inference on a sample of validation images.
   - Vehicles (car, bus, truck) are counted and simple thresholds classify traffic into three categories: Free-Flowing, Moderate, Heavy.
   - Results are visualized with annotated images and textual summaries.

3. **Traffic classification logic**
   - Thresholds hard‑coded as:
     - Free-Flowing: 0–10 vehicles
     - Moderate: 11–30 vehicles
     - Heavy: 31+ vehicles
   - Utility functions `classify_traffic` and image plotting were included.

### Second Phase

1. **Calibration dataset creation**
   - Added interactive notebook cells to label a random subset of validation images with true traffic states (Free/Moderate/Heavy).
   - Labels saved to `outputs/traffic_labels.csv`.

2. **Vehicle count extraction**
   - Wrote `count_vehicles` helper that filters detections by confidence and bounding box area.
   - Generated labeled counts (`outputs/labeled_counts.csv`) by running detector on labeled images.

3. **Threshold tuning**
   - Performed grid search over plausible free and moderate thresholds to maximize macro‑F1 score on calibration set.
   - Computed best thresholds and documented them in notebook.

4. **Evaluation**
   - Applied tuned thresholds to count data, produced classification report and confusion matrix.
   - Updated visualization cells to use calibrated thresholds and show the improved pipeline.

5. **Next steps & future work**
   - Suggests fine-tuning YOLO on the custom dataset and re-calibrating thresholds for final accuracy metrics.

## Repository Structure

- `data/` – dataset and metadata
- `src/traffic-flow-reporter-poc/` – notebook, model weights, outputs
- `outputs/` – generated CSVs with labels and counts

## Usage

1. Activate Python environment and install requirements from `src/traffic-flow-reporter-poc/requirements.txt`.
2. Run `demo.ipynb` sequentially to reproduce detection, calibration, and evaluation steps.

---

This README documents the project's evolution from an initial demonstration to a validated, data-driven traffic classification system.