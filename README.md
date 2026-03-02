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
1. Upload `dataset/` folder to Google Drive at: `MyDrive/SimplyFI_OCR/ocr_receipts/dataset/`
2. Open `ocr_notebook.ipynb` in Colab
3. Run all cells **top to bottom**

### Local Jupyter
1. Install requirements above
2. Update `base_path` in the notebook to your local `dataset/` path
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

