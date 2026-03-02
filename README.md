# Handwritten Digit OCR — Receipt Number Extraction

## Overview
This project extracts handwritten numbers from receipt images using a custom CNN trained on MNIST, combined with OpenCV-based region detection and digit segmentation. No external OCR APIs are used.

---

## Project Structure
```
SimplyFI_OCR/
├── ocr_receipts/
│   └── dataset/
│       ├── images/          # 20 receipt images (0.jpg - 19.jpg)
│       ├── annotations.xml  # Bounding box coordinates per region
│       └── receipts.csv     # Image ID to filename mapping
├── ocr_notebook.ipynb       # Main notebook (all steps inside)
├── digit_cnn_model.h5       # Saved trained CNN model
└── ocr_results.xlsx         # Final output (Excel)
```

---

## Requirements
```
tensorflow >= 2.19.0
opencv-python
numpy
pandas
openpyxl
matplotlib
scikit-learn
```

Install:
```bash
pip install tensorflow opencv-python numpy pandas openpyxl matplotlib scikit-learn
```

---

## How to Run

### Google Colab (Recommended)
1. Upload `archive/` folder to Google Drive at: `MyDrive/SimplyFI_OCR/ocr_receipts/archive/`
2. Open `ocr_notebook.ipynb` in Colab
3. Run all cells **top to bottom**

### Local Jupyter
1. Install requirements above
2. Update `base_path` in the notebook to your local `archive/` path
3. Run all cells in order

---

## Notebook Steps

| Step | What it does |
|------|-------------|
| Step 1 | Mount Google Drive, set paths |
| Step 2 | Load and normalize MNIST dataset |
| Step 3 | Build CNN, train for 10 epochs, save model |
| Step 4 | Parse annotations.xml, extract bounding boxes |
| Step 5 | Define crop/preprocess/predict functions |
| Step 6 | Run full pipeline on all 20 images |
| Step 7 | Save results to CSV and Excel |

---

## Key Functions

**`crop_region(image_path, xtl, ytl, xbr, ybr, label)`**
- `item` / `total` → crops right 35% (numbers are right-aligned)
- `date_time` → full crop (entire region is the value)

**`preprocess_and_segment(crop)`**
- Grayscale → resize → Gaussian blur → adaptive threshold → morphological closing → contour detection → returns 28×28 digit crops

**`predict_sequence(digit_crops, model)`**
- Predicts each digit, returns sequence string + confidence list
- Digits with confidence < 0.6 shown as `?`

---

## Output
```
[TOTAL     ] → 511        Confidences: [1.0, 1.0, 1.0] 
[ITEM      ] → 5481       Confidences: [1.0, 1.0, 1.0, 0.99] 
[DATE_TIME ] → 08122015   Confidences: [...] 
```

Results saved to `ocr_results.xlsx` with columns: Image, Label, Predicted Sequence, Avg Confidence, Min Confidence, Num Digits, Has Uncertain, All Confidences.
```


---

**TECHNICAL ASSIGNMENT REPORT**
**Handwritten Number Extraction from Receipt Images**
**Submitted to: SimplyFI Softech**

---

### 1. Objective

The goal of this assignment was to extract handwritten numbers from receipt images using deep learning. The system had to identify specific regions on receipts — item prices, total amounts, and date/time — and recognize the digit sequences within those regions without using any external OCR APIs.

---

### 2. Dataset

The dataset used was the **OCR Receipts Text Detection Dataset**, consisting of 20 receipt photographs. Each image is accompanied by bounding box annotations in an XML file, which marks four types of regions:

- **item** — individual item lines with prices (176 total across all images)
- **total** — the final total amount (19–20 per dataset)
- **date_time** — date and time of transaction (19–20 per dataset)
- **shop** — shop name/identifier

Only `item`, `total`, and `date_time` regions were used for digit extraction, as the shop region contains primarily alphabetic text.

---

### 3. Assumptions Made

- Numbers on `item` and `total` lines are always right-aligned (price on the right side of the line).
- The `date_time` region contains only numeric characters (digits and separators like `/`, `:`, `-`).
- The CNN model trained on MNIST (clean, centered, black-on-white digits) is sufficient as a base model for handwritten receipt digits.
- Regions with confidence below 0.6 for any digit are flagged as uncertain rather than silently accepted.

---

### 4. Overall Approach

The solution follows a three-stage pipeline:
```
Receipt Image
    ↓
Stage 1: Region Localization (XML annotations → bounding box coordinates)
    ↓
Stage 2: Preprocessing & Digit Segmentation (OpenCV)
    ↓
Stage 3: Digit Recognition (Custom CNN trained on MNIST)
    ↓
Output: Digit sequence + confidence per image region
