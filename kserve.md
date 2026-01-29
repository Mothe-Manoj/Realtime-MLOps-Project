# 1. Kubernetes with KIND

```bash
# Create KIND cluster
kind create cluster --name churn-model
```

### 2. KServe Setup

```bash
# Install KServe
kubectl apply -f https://github.com/kserve/kserve/releases/download/v0.11.0/kserve.yaml

or

#install CRD
helm install kserve-crd oci://ghcr.io/kserve/charts/kserve-crd \
  --version v0.16.0 \
  -n kserve \
  --wait

#Controller
helm install kserve oci://ghcr.io/kserve/charts/kserve \
  --version v0.16.0 \
  -n kserve \
  --set kserve.controller.deploymentMode=RawDeployment \
  --wait


# Create namespace, ServiceAccount and S3 secret for KServe
# Update k8s/serviceaccount.yaml with your AWS credentials first
kubectl apply -f k8s/serviceaccount.yaml

# Deploy inference service
kubectl apply -f k8s/inference.yaml

# Check inference service
kubectl get inferenceservice -n churn-model

# Wait for it to be ready
kubectl get inferenceservice churn-predictor -n churn-model -w
```

**Important:** Before deploying, update `k8s/serviceaccount.yaml` with your actual AWS credentials.

### 3. Test KServe Inference

```bash
# Get the inference service URL
INGRESS_HOST=$(kubectl get inferenceservice churn-predictor -n churn-model -o jsonpath='{.status.url}' | cut -d/ -f3)
SERVICE_HOSTNAME=$(kubectl get inferenceservice churn-predictor -n churn-model -o jsonpath='{.status.url}' | cut -d/ -f3)

# For local KIND cluster, port-forward
kubectl port-forward -n churn-model service/churn-predictor-predictor-default 8080:80

# Test prediction with curl
# Note: sklearn models expect data as arrays, not named features
# Order: age, tenure_months, monthly_charges, total_charges, num_support_calls
curl -X POST http://localhost:8080/v1/models/churn-predictor:predict \
  -H "Content-Type: application/json" \
  -d '{
    "instances": [
      [45, 24, 79.99, 1920.00, 3]
    ]
  }'
```

Expected response:
```json
{
  "predictions": [1]
}
```
