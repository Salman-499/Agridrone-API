# Agridrone API

[![Python](https://img.shields.io/badge/Python-3.9%2B-blue?logo=python)](https://www.python.org/) [![FastAPI](https://img.shields.io/badge/FastAPI-API-green?logo=fastapi)](https://fastapi.tiangolo.com/) [![YOLO11](https://img.shields.io/badge/YOLO11-custom%204--channel-orange?logo=github)](https://github.com/ultralytics/ultralytics) [![Docker](https://img.shields.io/badge/Docker-ready-blue?logo=docker)](https://www.docker.com/)

## Overview

**Agridrone API** serves the models from the **AgriDrone** research project at NUST: crop instance segmentation and growth-stage estimation for vegetable crops from UAV imagery. It takes an ordinary RGB drone image, predicts a synthetic NDVI band from it, stacks RGB + NDVI into a 4-channel image, and runs a custom YOLO11 model that finds each plant, outlines it, and estimates its growth stage.

---

## Key Features
- **Synthetic NDVI from RGB**: predicts an NDVI band from RGB imagery, so the models work without a multispectral camera
- **Custom 4-channel YOLO11**: instance segmentation on RGB + NDVI input, with an extra output that regresses a growth-stage value for each detected plant (BBCH scale)
- **Full pipeline**: RGB → synthetic NDVI → 4-channel image → YOLO11 masks, classes and growth stages
- **FastAPI** backend with async endpoints and auto-generated docs
- **Next.js web frontend** for uploading images and viewing results
- **Dockerized**, with an **Nginx reverse proxy** for production

---

## Architecture
- **Models**:
  - NDVI predictor: segmentation network with an EfficientNet-B2 encoder (TensorFlow/Keras)
  - Crop model: modified Ultralytics YOLO11 with 4 input channels and a growth-stage regression output (PyTorch)
- **Backend**: FastAPI (Python 3.9+)
- **Frontend**: Next.js (`frontend/`)
- **Deployment**: Docker Compose, Nginx (optional)
- **Workflow**:
  1. Upload an RGB image
  2. Predict a synthetic NDVI band
  3. Stack RGB + NDVI into a 4-channel image
  4. Run YOLO11: crop masks, crop type and growth stage per plant
  5. Return results (JSON, annotated images, NDVI band)

---

## API Endpoints
- `GET /` — API info
- `POST /predict_ndvi/` — Predict NDVI from an RGB image (returns NDVI visualization and band)
- `POST /predict_yolo/` — YOLO11 prediction on a 4-channel TIFF (returns JSON)
- `POST /predict_pipeline/` — Full RGB → NDVI → YOLO11 pipeline (returns JSON)
- `POST /predict_yolo_image/` — YOLO11 prediction with annotated image (PNG)
- `POST /predict_pipeline_image/` — Full pipeline with annotated image (PNG)

---

## Use Cases
- **Precision agriculture**: locating individual plants and tracking their growth stage
- **Harvest timing**: estimating when crops reach target growth stages
- **Research**: crop phenotyping from UAV imagery

---

## Quickstart

### Prerequisites
- Docker & Docker Compose
- Model files in the `Agridrone-API/` directory

### 1. Build & Run (API only)
```bash
docker-compose up --build agridrone
```

### 2. Build & Run (API + nginx proxy)
```bash
docker-compose up --build
```
- API: http://localhost:8001/
- Via nginx: http://localhost/agridrone/

---

## File Structure
```
Agridrone-API/
├── Agridrone-API/          # FastAPI app and model code
│   ├── app.py
│   ├── ndvi_predictor.py
│   ├── yolo_predictor.py
│   ├── ndvi_best_model/
│   └── ultralytics/        # modified Ultralytics (4-channel YOLO11)
├── frontend/               # Next.js web frontend
├── Dockerfile.agridrone
├── requirements.agridrone.txt
├── docker-compose.yml
├── nginx.conf
└── README.md
```

---

## Example Request
```bash
curl -X POST "http://localhost:8001/predict_pipeline/" -F "file=@field_image.jpg"
```

---

## Project & Credits
This API is part of **AgriDrone**, a NUST-funded research project at SEECS (Principal Investigator: Dr. Mehvish Rashid). The models and FastAPI backend were developed by the AgriDrone team (Muhammad Mashood, Muhammad Ahad Hassan Khan, Salman Arif). This repository adds the Next.js web frontend and the Docker/Nginx deployment.

Related paper (under review): *Decoupled Two-Stage Deep Learning Pipeline for Vegetable Crop Instance Segmentation and BBCH Growth Stage Estimation from UAV Multispectral Imagery*, Computers and Electronics in Agriculture.
