# 🪪 PAN Card Verification System

An end-to-end pipeline for verifying Indian PAN cards using **computer vision**, **OCR**, and **image forensics**. The system identifies whether an uploaded image is a PAN card, checks it for signs of forgery, and extracts structured data (PAN number, date of birth, name) from it.

---

## 📌 Overview

```
Input Image
     │
     ▼
┌─────────────────────┐
│  1. Identification  │  ── Is this a PAN card?
└─────────────────────┘
     │ Yes
     ▼
┌─────────────────────┐
│  2. Forgery Check   │  ── Has it been tampered with?
└─────────────────────┘
     │ Pass
     ▼
┌─────────────────────┐
│  3. Data Extraction │  ── Extract PAN, DOB, Name → JSON
└─────────────────────┘
```

---

## 📂 Project Structure

```
pan-card-verification/
│
├── README.md
├── requirements.txt
│
├── notebooks/
│   ├── 1_Pan_Identification.ipynb
│   ├── 2_Pan_Forgery.ipynb
│   └── 3_Pan_Data_Extraction_And_Processing.ipynb
│
├── templates/                  # Reference PAN card templates for SSIM matching, removed for privacy reasons
│   ├── TEMP1.png
│   ├── TEMP2.jpeg
│   ├── TEMP3.jpeg
│   └── TEMP4.jpg
│
└── sample_output/
    ├── extracted_data.json     # OCR output after data sorting
    ├── Forensics.json          # Forgery analysis results
    └── Valid_Forensics.json    # Schema for a valid (unflagged) forensics result
```

---

## 🧩 Modules

### 1. PAN Card Identification — `1_Pan_Identification.ipynb`

Determines whether an uploaded image is a PAN card using two complementary checks:

- **SSIM (Structural Similarity Index):** Compares the uploaded image against 4 reference PAN card templates. A score above 0.27 indicates structural resemblance.
- **Face Detection (Haar Cascade):** Validates that exactly one face is present in the image, consistent with a genuine PAN card.

**Libraries:** `OpenCV`, `scikit-image`

---

### 2. Forgery Detection — `2_Pan_Forgery.ipynb`

Runs a multi-test forensic analysis on the image to flag signs of digital manipulation:

| Test | Method | Flags |
|---|---|---|
| **ELA** | Error Level Analysis via JPEG re-compression | Score > 5.5 indicates tampering |
| **Metadata Check** | EXIF data scan | Presence of Photoshop, GIMP, Lightroom, etc. |
| **Steganography** | LSB (Least Significant Bit) analysis | Inconsistent LSBs across RGB channels |
| **Image Quality** | Laplacian variance | Variance ≤ 100 → blurry / potentially forged |
| **Color Consistency** | HSV standard deviation | Saturation stddev > 50 → color anomaly |
| **Noise Consistency** | Gaussian blur + Laplacian stddev | Irregular noise pattern → possible composite |

Results are exported to `Forensics/Forensics.json`. A summary report flags the number of failed tests and the ELA score.

**Libraries:** `OpenCV`, `Pillow`, `piexif`, `NumPy`

---

### 3. Data Extraction & Processing — `3_Pan_Data_Extraction_And_Processing.ipynb`

Extracts structured information from the PAN card image through a three-stage pipeline:

**Stage 1 — OCR (Image → Raw Text)**
- Uses **DocTR** (Document Text Recognition) with a pretrained model
- Filters words with confidence ≥ 50%
- Saves raw extracted text to a timestamped JSON file under `Extracted/`

**Stage 2 — Data Sorting (Raw Text → Structured Fields)**
- Extracts **PAN number** via regex: `[A-Z]{5}[0-9]{4}[A-Z]`
- Extracts **Date of Birth** in `DD/MM/YYYY` or `DD-MM-YYYY` formats
- Extracts **Name** by isolating uppercase-only strings, filtering out known boilerplate (e.g., `INCOME TAX DEPARTMENT`, `GOVT OF INDIA`)

**Stage 3 — Validation & Cleanup**
- Entries missing PAN, DOB, or name are moved to `rejected.json` with a reason
- Valid entries are saved to `extracted_data.json`
- Source images are deleted after processing to protect privacy

**Libraries:** `DocTR`, `NumPy`, `re`, `json`

---

## ⚙️ Setup & Usage

This project runs on **Google Colab** (GPU runtime recommended — T4).

### 1. Install dependencies

Each notebook includes an install cell. You can also install manually:

```bash
pip install python-doctr tf2onnx opencv-python scikit-image Pillow piexif numpy matplotlib
```

Or via `requirements.txt`:

```bash
pip install -r requirements.txt
```

### 2. Add reference templates

Place your reference PAN card template images in the root `/content/` directory on Colab:

```
TEMP1.png, TEMP2.jpeg, TEMP3.jpeg, TEMP4.jpg
```

> Note: These are not included in the repo. You must supply your own anonymized or synthetic PAN card templates.

### 3. Run notebooks in order

```
1_Pan_Identification.ipynb      ← Upload image, confirm it's a PAN card
2_Pan_Forgery.ipynb             ← Run forensic checks
3_Pan_Data_Extraction_And_Processing.ipynb  ← Extract and structure data
```

Each notebook prompts for an image upload via `google.colab.files.upload()`.

---

## 📤 Sample Output

**`extracted_data.json`**
```json
[
  {
    "file_name": "extracted_2024-01-15_10-30-00.json",
    "DOB": ["15/01/1990"],
    "PAN": ["ABCDE1234F"],
    "uppercase_strings": ["RAHUL", "KUMAR", "SHARMA"]
  }
]
```

**`Forensics.json`**
```json
{
  "metadata_result": true,
  "software_used": "",
  "ela_score": 3.21,
  "image_quality": true,
  "color_consistency": true,
  "steganography": true,
  "noise_consistency": true
}
```

---

## 🛠️ Tech Stack

| Category | Tools |
|---|---|
| OCR | DocTR |
| Computer Vision | OpenCV, scikit-image |
| Image Analysis | Pillow, piexif, NumPy |
| Runtime | Google Colab (T4 GPU) |
| Language | Python 3 |

---

## ⚠️ Limitations & Notes

- Template matching (SSIM) relies on the quality and diversity of reference templates. The more varied your templates, the more robust identification will be.
- ELA thresholds (5.5) and SSIM thresholds (0.27) were empirically tuned and may need adjustment for different image sources or qualities.
- The OCR stage works best on clear, front-facing, well-lit PAN card images.
- This project is intended for research and educational purposes. Ensure compliance with applicable data privacy regulations before processing real PAN card data.

---

## 📄 License

MIT License — feel free to use, modify, and distribute with attribution.
