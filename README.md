# Handwritten Digit OCR — Receipt Number Extraction

## Overview
This project extracts handwritten numbers from receipt images using a custom CNN trained on MNIST(60,000 training / 10,000 test images), combined with OpenCV-based region detection and digit segmentation. No external OCR APIs are used.

---
## System Architecture
<p align="center">
  <img width="423" height="933" alt="mermaid-diagram" 
       src="https://github.com/user-attachments/assets/a7fe54f6-cb5a-446f-b989-4a7a6caf7b33" />
</p>

---
## Model Performance

- MNIST Test Accuracy: **99.37%**
- Total Receipt Images Processed: **20**
- Regions Extracted: item, total, date_time
- Uncertainty Threshold: 0.6

### Confusion Matrix
<p align="center"><img width="658" height="547" alt="image" src="https://github.com/user-attachments/assets/43e3c6e6-8243-4838-b48c-6d2859319781" /></p>

---
## Project Structure
```
SimplyFI_OCR/
├── dataset/
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
 <p align="center"> <img width="900" height="356" alt="image" src="https://github.com/user-attachments/assets/42e9067c-b151-4b50-a012-fd685fdbb2b7" />
</p>


**`predict_sequence(digit_crops, model)`**
- Predicts each digit, returns sequence string + confidence list
- Digits with confidence < 0.6 shown as `?`

---

## Output

```text
[TOTAL     ] → 511        Confidences: [1.00, 1.00, 1.00]
[ITEM      ] → 5481       Confidences: [1.00, 1.00, 1.00, 0.99]
[DATE_TIME ] → 08122015   Confidences: [...]
```
<p align="center"><img width="1008" height="736" alt="image" src="https://github.com/user-attachments/assets/d91049bc-c6d5-49ef-9686-2360968e9c6d" /></p>

Results are saved to `ocr_results.xlsx` with the following columns:

- Image  
- Label  
- Predicted Sequence  
- Average Confidence  
- Minimum Confidence  
- Number of Digits  
- Has Uncertain Prediction  
- All Confidences  

---

## Limitations
- The model is trained only on digits (0–9).  
  Separators such as `.`, `/`, `:` are not explicitly classified.
- Requires XML bounding box annotations for region localization.
- Performance may degrade on unseen receipt layouts or different fonts.
- MNIST-trained model has a domain gap with real receipt handwriting — 
  fine-tuning on receipt crops would improve accuracy further.
