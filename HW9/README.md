# Hair Classifier - ML Zoomcamp Homework 9

This repository contains the solution for Homework 9 of the ML Zoomcamp 2025 course - deploying a hair type classifier (Straight vs Curly) using AWS Lambda and Docker.

## Homework Answers

| Question | Answer |
|----------|--------|
| Q1 - Output node name | `output` |
| Q2 - Target size | `200x200` |
| Q3 - First pixel R channel | `-1.073` |
| Q4 - Model output | `0.09` |
| Q5 - Docker image size | `1208 Mb` |
| Q6 - Lambda model output | `-0.10` |

## Project Structure

```
hair-classifier/
├── Dockerfile                    # Docker config for Lambda deployment
├── lambda_function.py            # Lambda handler with preprocessing
├── hair_classifier_homework.ipynb # Jupyter notebook for Q1-Q4
└── README.md                     # This file
```

## Prerequisites

- Docker installed
- Python 3.11+ (for local testing)
- Required packages: `onnxruntime`, `pillow`, `numpy`

## Quick Start

### For Questions 1-4 (Local ONNX Model)

1. Download the model files:
```bash
PREFIX="https://github.com/alexeygrigorev/large-datasets/releases/download/hairstyle"
wget ${PREFIX}/hair_classifier_v1.onnx.data
wget ${PREFIX}/hair_classifier_v1.onnx
```

2. Run the Jupyter notebook `hair_classifier_homework.ipynb`

### For Questions 5-6 (Docker Lambda)

1. Pull the base image:
```bash
docker pull agrigorev/model-2025-hairstyle:v1
```

2. Check the image size (Q5):
```bash
docker images agrigorev/model-2025-hairstyle:v1
```

3. Build the Lambda container:
```bash
docker build -t hair-classifier-lambda .
```

4. Run the container:
```bash
docker run -it --rm -p 8080:8080 hair-classifier-lambda
```

5. Test the prediction (Q6) - in a new terminal:
```bash
curl -X POST "http://localhost:8080/2015-03-31/functions/function/invocations" \
  -H "Content-Type: application/json" \
  -d '{"url": "https://habrastorage.org/webt/yf/_d/ok/yf_dokzqy3vcritme8ggnzqlvwa.jpeg"}'
```

Expected output:
```json
{"prediction": -0.10220836102962494}
```

## Preprocessing Details

The model uses ImageNet normalization (from Homework 8):

```python
x = x / 255.0  # Scale to [0, 1]
mean = [0.485, 0.456, 0.406]
std = [0.229, 0.224, 0.225]
for i in range(3):
    x[:, :, i] = (x[:, :, i] - mean[i]) / std[i]
```

## Notes

- Q1-Q4 use `hair_classifier_v1.onnx` (downloaded model)
- Q5-Q6 use `hair_classifier_empty.onnx` (model inside Docker image)
- The two models produce different outputs for the same image

## Optional: Deploy to AWS

1. Publish image to ECR
2. Create Lambda function using the ECR image
3. Increase RAM and timeout
4. Expose via API Gateway

## References

- [ML Zoomcamp Course](https://github.com/DataTalksClub/machine-learning-zoomcamp)
- [Homework 9 Instructions](https://github.com/DataTalksClub/machine-learning-zoomcamp/blob/master/cohorts/2025/09-serverless/homework.md)
