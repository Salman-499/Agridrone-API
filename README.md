# Agridrone API

Inference API and web front end for **AgriDrone**, a NUST-funded research project at SEECS (PI: Dr. Mehvish Rashid). The project collected UAV imagery of six vegetable crops at the National Agricultural Research Centre (NARC), Islamabad, and trained models that find and outline each plant and estimate its BBCH growth stage.

This repo serves the models from that project.

## How it works

1. Upload an ordinary RGB drone image.
2. A segmentation network with an EfficientNet encoder (Keras) predicts a synthetic NDVI band from the RGB image, so no multispectral camera is needed.
3. RGB and NDVI are stacked into a 4-channel image.
4. A YOLO11 instance-segmentation model, adapted to take 4 input channels, finds and outlines each plant and labels the crop.

The `ultralytics/` folder is a copy of the Ultralytics library with its image loaders changed to keep the fourth (NDVI) channel.

The paper describes a second stage that estimates each plant's growth stage. That stage isn't part of this API; the `growth_stages` field in the response is only filled if the loaded model returns it.

## Endpoints

| Method | Path | Input | Output |
|---|---|---|---|
| POST | `/predict_ndvi/` | RGB image | ZIP with an NDVI preview (PNG) and the NDVI band (`.npy`) |
| POST | `/predict_yolo/` | 4-channel TIFF | JSON: boxes, classes, masks |
| POST | `/predict_pipeline/` | RGB image | JSON for the full RGB → NDVI → YOLO11 pipeline |
| POST | `/predict_yolo_image/` | 4-channel TIFF | Annotated PNG |
| POST | `/predict_pipeline_image/` | RGB image | Annotated PNG |

## Running it

Model weights are not included. Put them here before starting:

- `Agridrone-API/best_yolo_model.pt` (YOLO11)
- `Agridrone-API/ndvi_best_model/best_model_weights.weights.h5` (NDVI model; the architecture file is already there)

```bash
docker-compose up --build agridrone      # API on http://localhost:8001
docker-compose up --build                # API + Nginx on http://localhost/agridrone/

cd frontend && npm install && npm run dev   # Next.js front end
```

## Credits

The models and backend were developed by the AgriDrone team: Muhammad Mashood, Muhammad Ahad Hassan Khan and Salman Arif. I worked on the backend and on adapting YOLO11 to the 4-channel input, and built the Next.js front end and the Docker/Nginx setup.

Paper (under review): *Decoupled Two-Stage Deep Learning Pipeline for Vegetable Crop Instance Segmentation and BBCH Growth Stage Estimation from UAV Multispectral Imagery*, Computers and Electronics in Agriculture.
