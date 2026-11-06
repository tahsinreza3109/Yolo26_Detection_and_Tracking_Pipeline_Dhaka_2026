# YOLO26 vehicle detection model for Dhaka mixed traffic (2026)

This repository holds a YOLO26-nano detector fine-tuned to detect and classify vehicles in overhead video of Dhaka's non-lane-based, heterogeneous traffic.
It was used to count vehicles at a screen line and derive passenger-car-unit (PCU) flow and composition for the time-headway study of pedal rickshaws and battery-operated three-wheelers:

- Headway study repository: [Pair_Specific_Time_Headway_Distributions_and_Covariate_Conditioned_AFT_Models](https://github.com/tahsinreza3109/Pair_Specific_Time_Headway_Distributions_and_Covariate_Conditioned_AFT_Models).

**Author:** Tahsin Reza, Department of Urban and Regional Planning, BUET ([ORCID 0009-0006-0889-3589](https://orcid.org/0009-0006-0889-3589)).

## Contents
```
model/
  weights/best.pt            trained weights (Ultralytics format)
  training/args.yaml         full training configuration
  training/results.csv       per-epoch training and validation log
  training/*.png, labels.jpg training curves, PR/F1 curves, confusion matrices, label statistics (Ultralytics output)
```
- `best.pt` SHA-256: `5dc7c7935a5a987042f120add58e4ea21471564782f7221088211ea8d97e0d47`

## Classes
| ID | Class | Description | PCU used for flow (RHD 2005) |
|---|---|---|---|
| 0 | Bicycle | Bicycle | 0.50 |
| 1 | Bike | Motorcycle | 0.75 |
| 2 | 3W | Non-CNG three-wheelers: pedal rickshaw, battery-operated three-wheeler, van, etc. | 2.00 |
| 3 | CNG | CNG auto-rickshaw | 0.75 |
| 4 | 4W | Four-wheeled motor vehicles other than buses | 1.00 |
| 5 | Bus | Bus | 3.00 |

`3W` is a composite class. The detector does **not** separate pedal rickshaws from battery-operated three-wheelers; that distinction was coded manually in the headway study.

## Training
- **Base model and settings:** `yolo26n.pt` (COCO-pretrained), Ultralytics 8.4, 50 epochs, image size 1280, batch 4, optimizer `auto`, seed 0, deterministic. See `model/training/args.yaml` for the full configuration.
- **Training data:** 1,627 self-labelled video frames of Dhaka traffic, holding 20,702 instances:
  - 3W: 9,601
  - 4W: 5,803
  - Bus: 1,904
  - CNG: 1,561
  - Bike: 1,297
  - Bicycle: 536
- **Validation data:** 287 frames with 3,700 instances. No frame appears in both the training and validation sets.

## Performance (validation split)
`best.pt` was re-evaluated on the validation split with imgsz 1280 and IoU 0.7. The confusion matrix uses conf 0.26, the same threshold used in tracking.

| | AP@0.5 | AP@0.5:0.95 |
|---|---|---|
| **All classes** | **0.820** | **0.703** |
| 3W | 0.919 | |
| Bus | 0.897 | |
| CNG | 0.895 | |
| 4W | 0.862 | |
| Bike | 0.804 | |
| Bicycle | 0.544 | |

- **Overall precision and recall:** P = 0.769 and R = 0.751.
- **Recall by class at conf 0.26:** 3W 0.90, Bus 0.85, CNG 0.80, 4W 0.80, Bike 0.79, Bicycle 0.47.
- **Main confusions:** about 12% of CNGs are labelled 3W. Bicycles are the weakest class: 35% are missed at conf 0.26.
- **No independent test set:** these are validation-split figures, and the validation split was also used to select `best.pt`. Treat them as optimistic estimates of performance on new footage.
- **Training log vs re-evaluation:** the training log's final in-training validation values (mAP@0.5 0.780, mAP@0.5:0.95 0.670) differ from the re-evaluation above because the in-training and final evaluation settings differ.

## Usage
```python
from ultralytics import YOLO

model = YOLO("model/weights/best.pt")

# detection
model.predict("frame.jpg", imgsz=1280, conf=0.26)

# tracking (settings used in the study: BoT-SORT, the Ultralytics default tracker)
for r in model.track("video.mp4", imgsz=1280, conf=0.26, persist=True, stream=True):
    ...
```
In the study, each track was counted once when its box centroid crossed a horizontal screen line. The crossing time and class were logged and converted to PCU flow with a centred 30-s rolling window.

## Licence
The weights are fine-tuned from Ultralytics YOLO26. Ultralytics models and derivatives are distributed under **AGPL-3.0**, or under an Ultralytics Enterprise licence. Use of these weights must follow those terms.
