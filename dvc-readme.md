# DVC Setup (Data Version Control)

```bash
# Initialize DVC
dvc init

# Configure S3 remote
dvc remote add -d myremote s3://mlops-cicd-manoj-bucket

# Track model with DVC
dvc add models/churn_model.pkl

# Push to S3
dvc push

# Commit DVC metadata
git add models/churn_model.pkl.dvc .dvc/ .gitignore
git commit -m "Track model with DVC"
```
### 3. Push Model to S3

After training the model and setting up DVC:

```bash
# Configure AWS credentials (if not already done)
export AWS_ACCESS_KEY_ID=your-key
export AWS_SECRET_ACCESS_KEY=your-secret
export AWS_DEFAULT_REGION=us-east-1
