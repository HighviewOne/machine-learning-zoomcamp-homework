# Homework 10: Kubernetes

## Overview
This homework covers deploying a machine learning model (lead scoring) to Kubernetes using `kind`.

## Answers

| Question | Answer |
|----------|--------|
| Q1: Conversion probability | **0.89** (actual: 0.993) |
| Q2: Kind version | **0.25.0** |
| Q3: Smallest deployable unit | **Pod** |
| Q4: Service type | **ClusterIP** |
| Q5: Command to register image | **kind load docker-image** |
| Q6: Container port | **9696** |
| Q7: Service selector | **subscription** |
| Q8: Maximum replicas (optional) | **3** |

## Files

- `deployment.yaml` - Kubernetes deployment configuration
- `service.yaml` - Kubernetes service configuration
- `q6_test.py` - Test script for the prediction endpoint
- `patch.yaml` - Metrics server patch for HPA

## Commands Used

### Building and Loading the Docker Image
```bash
# Build the image
docker build -f Dockerfile_full -t zoomcamp-model:3.13.10-hw10 .

# Load into kind cluster
kind load docker-image zoomcamp-model:3.13.10-hw10
```

### Creating the Cluster
```bash
# Create cluster
kind create cluster

# Verify
kubectl cluster-info
```

### Deploying the Application
```bash
# Apply deployment
kubectl apply -f deployment.yaml

# Apply service
kubectl apply -f service.yaml

# Check pods
kubectl get pods

# Check services
kubectl get services
```

### Testing the Service
```bash
# Port forward
kubectl port-forward service/subscription 9696:80

# In another terminal, run test
python q6_test.py
```

### Autoscaling (Question 8)
```bash
# Install metrics server
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

# Patch for kind compatibility
kubectl patch deployment metrics-server -n kube-system --patch-file patch.yaml

# Create HPA
kubectl autoscale deployment subscription --name subscription-hpa --cpu-percent=20 --min=1 --max=3

# Monitor HPA
kubectl get hpa subscription-hpa --watch
```

## Results

### Q1: Test Output
```
{'conversion_probability': 0.9933071490756734, 'conversion': True}
```

### Q8: Autoscaling Output
```
NAME               REFERENCE                 TARGETS       MINPODS   MAXPODS   REPLICAS   AGE
subscription-hpa   Deployment/subscription   cpu: 2%/20%   1         3         1          11m
subscription-hpa   Deployment/subscription   cpu: 38%/20%  1         3         2          14m
subscription-hpa   Deployment/subscription   cpu: 16%/20%  1         3         3          15m
```

The HPA successfully scaled from 1 to 3 replicas under load.
