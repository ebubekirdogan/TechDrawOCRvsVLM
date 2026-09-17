# TechDrawOCRvsVLM

Comparing **OCR-only**, **VLM-only**, and **OCR+VLM** pipelines for extracting manufacturing information (dimensions, tolerances, GD&T, surface finish, materials) from mechanical engineering technical drawings.

**[📊 View the full comparison report](https://ebubekirdogan.github.io/TechDrawOCRvsVLM/Project_Report.html?v=2)**

## Overview

Technical drawings encode manufacturing information in a dense visual language: rotated dimension text, GD&T symbols packed into feature control frames, tolerance callouts, and datum references overlaid on geometric line work. Standard OCR struggles with this format because it was built for natural text, not engineering notation. This project compares three extraction strategies on the same test drawing to measure how much each approach actually recovers.

| Pipeline | Description |
|---|---|
| **OCR-only** | [eDOCr2](https://github.com/javvi51/edocr2) — a specialized OCR system trained on engineering drawings, with dedicated recognizers for dimensions and GD&T symbols |
| **VLM-only** | Gemini 3.6 Flash — reads the drawing image directly and outputs structured JSON without a separate OCR step |
| **OCR+VLM** | eDOCr2's raw OCR output is passed to Gemini 3.6 Flash alongside the image, so the VLM can cross-check and structure the OCR result |

## Test Drawing

[`input/teknik_resim_test.png`](./input/teknik_resim_test.png) — an annotated technical drawing (`BEARING HOUSING`, Al 7075-T6, A3, 1st angle projection) with 38 manually labeled ground-truth characteristics spanning:

- Dimensions (diameters, lengths, angles, radii, chamfers, bolt circles)
- GD&T callouts (position, flatness, perpendicularity, parallelism, cylindricity, circularity, symmetry, profile of a surface)
- Thread specifications
- Surface roughness (Ra)
- Datum features (A, B, C)

## Pipeline

```
Technical Drawing (image)
        │
        ├──► eDOCr2 (OCR-only) ──────────────────────► edocr2_output.json
        │
        ├──► Gemini 3.6 Flash (VLM-only) ─────────────► gemini_output.json
        │
        └──► eDOCr2 → OCR text + image → Gemini ─────► edocr2_gemini_output.json

Each output is compared against ground_truth.json (38 hand-labeled characteristics)
```

## Repository Structure

```
TechDrawOCRvsVLM/
├── input/                      # test drawing image
├── ground_truth/               # ground_truth.json (38 characteristics)
├── edocr2_output/               # OCR-only pipeline output
├── gemini_output/                # VLM-only pipeline output
├── edocr2_gemini_output/         # OCR+VLM pipeline output
├── Untitled0.ipynb                # Colab notebook (pipelines)
├── Project_Report.html            # full visual comparison report
└── README.md
```

## Full Report

**[View the live report →](https://ebubekirdogan.github.io/TechDrawOCRvsVLM/Project_Report.html?v=2)**

The detailed, balloon-by-balloon comparison (ground truth vs. each pipeline's output, side by side with the drawing). Source file: [`Project_Report.html`](./Project_Report.html?v=2).

## Setup

Pipelines were run on Google Colab (T4 GPU) with outputs persisted to Google Drive after each step.

```bash
git clone https://github.com/javvi51/edocr2.git
pip install -r edocr2/requirements.txt
apt-get install -y tesseract-ocr-eng tesseract-ocr-nor tesseract-ocr-osd
pip install pytesseract google-genai
```

Recognizer model weights (`recognizer_gdts.keras`, `recognizer_dimensions_2.keras` + matching `.txt` alphabet files) are downloaded from the [eDOCr2 releases page](https://github.com/javvi51/edocr2/releases) and placed under `edocr2/edocr2/models/`.

## References

- eDOCr2: [github.com/javvi51/edocr2](https://github.com/javvi51/edocr2)
