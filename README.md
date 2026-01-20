PCB DEFECT DETECTION USING PATCH-BASED ANOMALY DETECTION
=======================================================

This project implements an AI-based computer vision system for PCB defect
detection using a Golden PCB reference and patch-level anomaly detection.
The approach does not require labeled defect data for training and is
designed to reflect how industrial AOI (Automated Optical Inspection)
systems operate.


WHY THIS DATASET
----------------
The dataset was chosen primarily because it provides PCBs that are
precisely placed and spatially aligned across all images.

Precise alignment is critical for reference-based and patch-based anomaly
detection methods, since each spatial region of a test PCB must correspond
to the same region in the Golden PCB.

This dataset was the only publicly available PCB dataset found that
satisfied this requirement reliably.

LIMITATION:
- Only ONE fully good (Golden) PCB image is available
- All remaining images contain defects

To address this limitation, normal variation is simulated using carefully
constrained augmentations, and anomaly thresholds are statistically
calibrated using good-vs-good evaluation. This makes the system suitable
for prototyping, research, and interviews, while acknowledging that
production systems require multiple real good samples.


DEFECT TYPES COVERED
--------------------
The system can detect and localize:
- Missing holes
- Mouse bites
- Open circuits
- Short circuits
- Spurious copper


CORE IDEA
---------
Instead of learning what defects look like, the system learns what
"normal" looks like.

1. Extract deep features using a pretrained CNN
2. Convert feature maps into patch-level embeddings
3. Store Golden PCB embeddings in a memory bank
4. Compare test PCB patches with memory using nearest-neighbor distance
5. Treat distance as anomaly score
6. Apply statistical thresholding
7. Produce heatmaps and PASS / FAIL decisions


ARCHITECTURE OVERVIEW
---------------------
Input PCB Image
      |
      v
Pretrained CNN (Frozen)
      |
      v
Multi-layer Feature Extraction
      |
      v
Patch Embedding Fusion
      |
      v
Nearest Neighbor Search (Memory Bank)
      |
      v
Patch-level Anomaly Scores
      |
      v
Thresholding & Aggregation
      |
      v
Heatmap + PASS / FAIL


METRICS USED
------------
Image-Level:
- ROC-AUC (ranking performance)
- Mean anomaly score

Patch-Level:
- Abnormal patch ratio
- Spatial heatmap localization

Decision Logic:
- FAIL if more than 2% of patches exceed the anomaly threshold
- Threshold calibrated using good-vs-good evaluation


PROJECT STRUCTURE
-----------------
.
|-- images/
|   |-- good_pcb.jpg
|   |-- 01_missing_hole_*.jpg
|   |-- 01_mouse_bite_*.jpg
|   |-- 01_open_circuit_*.jpg
|   |-- 01_short_*.jpg
|   |-- 01_spurious_copper_*.jpg
|
|-- test.ipynb          (Main notebook)
|-- requirements.txt
|-- README.txt


INSTALLATION
------------
1. Create a virtual environment (recommended)
   python -m venv venv
   source venv/bin/activate
   (Windows: venv\\Scripts\\activate)

2. Install dependencies
   pip install -r requirements.txt

Main dependencies:
- TensorFlow
- OpenCV
- NumPy
- scikit-learn
- Matplotlib


HOW TO RUN
----------
1. Launch Jupyter Notebook
   jupyter notebook

2. Open:
   test.ipynb

3. Run all cells from top to bottom

The notebook will:
- Build memory from the Golden PCB
- Calibrate anomaly thresholds
- Evaluate multiple defective PCBs
- Compute metrics (ROC-AUC, patch ratios)
- Generate heatmaps and overlays


EXAMPLE RESULTS
---------------
- ROC-AUC: 1.0 (Only 1 Good PCB image has been used, the metric will improve after adding more images)
- All defective PCBs correctly classified as FAIL
- Localized heatmaps highlight defect regions
- Different defect types show different abnormal patch ratios


KEY DESIGN DECISIONS
--------------------
- No supervised training is used
- CNN weights are frozen to avoid overfitting
- Patch-based approach enables defect localization
- Statistical thresholding suppresses normal variation
- Decision logic mimics industrial AOI systems


LIMITATIONS AND FUTURE WORK
---------------------------
Current limitations:
- Only one real good PCB is available
- Augmentations simulate, but do not replace, real variation

Future improvements:
- Use multiple real good PCBs
- ROI-based masking for components
- Faster NN search using FAISS
- OCR for serial numbers
- CAD-assisted component verification


SUMMARY
-----------------
This system uses patch-based anomaly detection to compare test PCBs against
a golden reference. It learns the distribution of normal features using a
pretrained CNN and flags deviations statistically, without requiring defect
labels. Thresholds are calibrated using good-vs-good evaluation, and final
decisions combine image-level and patch-level reasoning, similar to
industrial AOI systems.
