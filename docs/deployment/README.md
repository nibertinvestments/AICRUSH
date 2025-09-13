# Deployment Guide

## Overview

This guide covers deploying AICRUSH components to Google Cloud Platform. The deployment process is designed for reliability, security, and scalability.

## Prerequisites

### Required Tools
- **Google Cloud SDK** ([Installation Guide](https://cloud.google.com/sdk/docs/install))
- **Docker** ([Installation Guide](https://docs.docker.com/get-docker/))
- **kubectl** (included with gcloud)
- **Git** for version control

### Required Access
- Google Cloud Project: `gen-lang-client-0311226580`
- IAM permissions for deployment
- GitHub repository access

### Environment Setup
```bash
# Authenticate with Google Cloud
gcloud auth login
gcloud auth application-default login

# Set project and region
gcloud config set project gen-lang-client-0311226580
gcloud config set run/region us-east4

# Verify setup
gcloud config list
```

## Deployment Architecture

### Current Production Environment
```
Production Setup:
├── Project: gen-lang-client-0311226580
├── Region: us-east4
├── Service: ai-gpu-service-east
├── Storage: ai-storage-bucket-6658
├── GPU: NVIDIA L4
└── Auto-scaling: Enabled
```

### Container Registry
```
Registry: us-central1-docker.pkg.dev/gen-lang-client-0311226580/ai-gpu-repo
Current Image: ai-gpu-container:v3
```

## Deployment Process

### 1. Build Container Image

#### Dockerfile Structure
```dockerfile
# Multi-stage build for optimization
FROM python:3.9-slim as builder

# Install dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Production stage
FROM python:3.9-slim

# Copy dependencies from builder
COPY --from=builder /usr/local/lib/python3.9/site-packages /usr/local/lib/python3.9/site-packages

# Copy application code
COPY src/ /app/src/
WORKDIR /app

# Set up non-root user
RUN groupadd -r appuser && useradd -r -g appuser appuser
USER appuser

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=60s \
  CMD curl -f http://localhost:8080/health || exit 1

# Start application
CMD ["python", "-m", "src.main"]
```

#### Build Command
```bash
# Build the container image
gcloud builds submit --tag us-central1-docker.pkg.dev/gen-lang-client-0311226580/ai-gpu-repo/ai-gpu-container:latest

# Tag with version
gcloud builds submit --tag us-central1-docker.pkg.dev/gen-lang-client-0311226580/ai-gpu-repo/ai-gpu-container:v4
```

### 2. Deploy to Cloud Run

#### Basic Deployment
```bash
# Deploy new version
gcloud run deploy ai-gpu-service-east \
  --image us-central1-docker.pkg.dev/gen-lang-client-0311226580/ai-gpu-repo/ai-gpu-container:v4 \
  --region us-east4 \
  --platform managed \
  --allow-unauthenticated=false \
  --memory 4Gi \
  --cpu 2 \
  --gpu 1 \
  --gpu-type nvidia-l4 \
  --max-instances 10 \
  --min-instances 1
```

#### Advanced Configuration
```bash
# Deploy with full configuration
gcloud run deploy ai-gpu-service-east \
  --image us-central1-docker.pkg.dev/gen-lang-client-0311226580/ai-gpu-repo/ai-gpu-container:v4 \
  --region us-east4 \
  --platform managed \
  --allow-unauthenticated=false \
  --memory 8Gi \
  --cpu 4 \
  --gpu 1 \
  --gpu-type nvidia-l4 \
  --max-instances 10 \
  --min-instances 1 \
  --concurrency 80 \
  --timeout 300 \
  --set-env-vars="PROJECT_ID=gen-lang-client-0311226580,BUCKET_NAME=ai-storage-bucket-6658" \
  --service-account ai-service@gen-lang-client-0311226580.iam.gserviceaccount.com \
  --labels="app=aicrush,version=v4,environment=production"
```

### 3. Verify Deployment

#### Health Check
```bash
# Get service URL
SERVICE_URL=$(gcloud run services describe ai-gpu-service-east --region us-east4 --format 'value(status.url)')

# Test health endpoint
TOKEN=$(gcloud auth print-identity-token)
curl -H "Authorization: Bearer $TOKEN" $SERVICE_URL/health

# Test full system
curl -H "Authorization: Bearer $TOKEN" $SERVICE_URL/test-all
```

#### Performance Test
```bash
# Load test (use with caution)
for i in {1..10}; do
  curl -H "Authorization: Bearer $TOKEN" $SERVICE_URL/health &
done
wait
```

## Configuration Management

### Environment Variables

#### Production Configuration
```bash
# Set production environment variables
gcloud run services update ai-gpu-service-east \
  --region us-east4 \
  --set-env-vars="
    PROJECT_ID=gen-lang-client-0311226580,
    BUCKET_NAME=ai-storage-bucket-6658,
    ENVIRONMENT=production,
    LOG_LEVEL=info,
    GPU_ENABLED=true,
    MODEL_VERSION=v1.0,
    MAX_BATCH_SIZE=32
  "
```

#### Secrets Management
```bash
# Create secrets in Secret Manager
gcloud secrets create api-key --data-file=api-key.txt

# Use secrets in Cloud Run
gcloud run services update ai-gpu-service-east \
  --region us-east4 \
  --set-secrets="API_KEY=api-key:latest"
```

### Service Account Configuration

#### Create Service Account
```bash
# Create service account
gcloud iam service-accounts create ai-service \
  --display-name="AICRUSH Service Account" \
  --description="Service account for AICRUSH application"

# Grant necessary permissions
gcloud projects add-iam-policy-binding gen-lang-client-0311226580 \
  --member="serviceAccount:ai-service@gen-lang-client-0311226580.iam.gserviceaccount.com" \
  --role="roles/storage.objectViewer"

gcloud projects add-iam-policy-binding gen-lang-client-0311226580 \
  --member="serviceAccount:ai-service@gen-lang-client-0311226580.iam.gserviceaccount.com" \
  --role="roles/aiplatform.user"
```

## Monitoring and Logging

### Enable Monitoring
```bash
# Enable monitoring
gcloud run services update ai-gpu-service-east \
  --region us-east4 \
  --set-labels="monitoring=enabled"

# Create uptime check
gcloud alpha monitoring uptime-check-configs create \
  --display-name="AICRUSH Health Check" \
  --http-check-path="/health" \
  --http-check-port=443 \
  --hostname="ai-gpu-service-east-249675450204.us-east4.run.app" \
  --use-ssl
```

### Log Configuration
```bash
# View logs
gcloud logging read "resource.type=cloud_run_revision AND resource.labels.service_name=ai-gpu-service-east" \
  --limit 50 \
  --format json

# Create log-based metric
gcloud logging metrics create error_rate \
  --description="Error rate for AICRUSH service" \
  --log-filter='resource.type="cloud_run_revision" AND resource.labels.service_name="ai-gpu-service-east" AND severity>=ERROR'
```

### Alerting
```bash
# Create alerting policy
gcloud alpha monitoring policies create \
  --notification-channels=$NOTIFICATION_CHANNEL \
  --display-name="AICRUSH High Error Rate" \
  --condition-display-name="Error rate > 5%" \
  --condition-filter='resource.type="cloud_run_revision"' \
  --condition='thresholdValue: 0.05'
```

## CI/CD Pipeline

### GitHub Actions Workflow

```yaml
# .github/workflows/deploy.yml
name: Deploy to Cloud Run

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

env:
  PROJECT_ID: gen-lang-client-0311226580
  SERVICE_NAME: ai-gpu-service-east
  REGION: us-east4

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.9'
          
      - name: Install dependencies
        run: |
          pip install -r requirements.txt
          pip install -r requirements-dev.txt
          
      - name: Run tests
        run: |
          pytest tests/
          
      - name: Security scan
        run: |
          bandit -r src/
          safety check

  deploy:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up Google Cloud SDK
        uses: google-github-actions/setup-gcloud@v1
        with:
          service_account_key: ${{ secrets.GCP_SA_KEY }}
          project_id: ${{ env.PROJECT_ID }}
          
      - name: Configure Docker for Artifact Registry
        run: gcloud auth configure-docker us-central1-docker.pkg.dev
        
      - name: Build and push Docker image
        run: |
          IMAGE_URI=us-central1-docker.pkg.dev/$PROJECT_ID/ai-gpu-repo/ai-gpu-container:$GITHUB_SHA
          docker build -t $IMAGE_URI .
          docker push $IMAGE_URI
          
      - name: Deploy to Cloud Run
        run: |
          gcloud run deploy $SERVICE_NAME \
            --image us-central1-docker.pkg.dev/$PROJECT_ID/ai-gpu-repo/ai-gpu-container:$GITHUB_SHA \
            --region $REGION \
            --platform managed
            
      - name: Verify deployment
        run: |
          SERVICE_URL=$(gcloud run services describe $SERVICE_NAME --region $REGION --format 'value(status.url)')
          curl -f $SERVICE_URL/health
```

### Manual Deployment Script

```bash
#!/bin/bash
# deploy.sh - Manual deployment script

set -e

PROJECT_ID="gen-lang-client-0311226580"
SERVICE_NAME="ai-gpu-service-east"
REGION="us-east4"
IMAGE_NAME="ai-gpu-container"

echo "🚀 Starting deployment..."

# Build image
echo "📦 Building container image..."
gcloud builds submit --tag us-central1-docker.pkg.dev/$PROJECT_ID/ai-gpu-repo/$IMAGE_NAME:$(git rev-parse --short HEAD)

# Deploy to Cloud Run
echo "🚢 Deploying to Cloud Run..."
gcloud run deploy $SERVICE_NAME \
  --image us-central1-docker.pkg.dev/$PROJECT_ID/ai-gpu-repo/$IMAGE_NAME:$(git rev-parse --short HEAD) \
  --region $REGION \
  --platform managed

# Verify deployment
echo "✅ Verifying deployment..."
SERVICE_URL=$(gcloud run services describe $SERVICE_NAME --region $REGION --format 'value(status.url)')
TOKEN=$(gcloud auth print-identity-token)

if curl -f -H "Authorization: Bearer $TOKEN" $SERVICE_URL/health; then
  echo "🎉 Deployment successful!"
else
  echo "❌ Deployment verification failed!"
  exit 1
fi
```

## Rollback Strategy

### Immediate Rollback
```bash
# List recent revisions
gcloud run revisions list --service ai-gpu-service-east --region us-east4

# Rollback to previous revision
PREVIOUS_REVISION=$(gcloud run revisions list --service ai-gpu-service-east --region us-east4 --limit 2 --format "value(metadata.name)" | tail -n 1)

gcloud run services update-traffic ai-gpu-service-east \
  --region us-east4 \
  --to-revisions=$PREVIOUS_REVISION=100
```

### Gradual Rollback
```bash
# Split traffic between versions
gcloud run services update-traffic ai-gpu-service-east \
  --region us-east4 \
  --to-revisions=LATEST=50,$PREVIOUS_REVISION=50

# Complete rollback after verification
gcloud run services update-traffic ai-gpu-service-east \
  --region us-east4 \
  --to-revisions=$PREVIOUS_REVISION=100
```

## Troubleshooting

### Common Issues

#### Build Failures
```bash
# Check build logs
gcloud builds log $(gcloud builds list --limit 1 --format "value(id)")

# Common fixes:
# 1. Check Dockerfile syntax
# 2. Verify requirements.txt
# 3. Check build context size
```

#### Deployment Failures
```bash
# Check service logs
gcloud logging read "resource.type=cloud_run_revision AND resource.labels.service_name=ai-gpu-service-east" --limit 20

# Common fixes:
# 1. Check memory/CPU limits
# 2. Verify environment variables
# 3. Check service account permissions
```

#### Performance Issues
```bash
# Monitor resource usage
gcloud run services describe ai-gpu-service-east --region us-east4

# Scale up if needed
gcloud run services update ai-gpu-service-east \
  --region us-east4 \
  --memory 8Gi \
  --cpu 4 \
  --max-instances 20
```

### Debug Commands
```bash
# Check service status
gcloud run services describe ai-gpu-service-east --region us-east4

# View recent logs
gcloud logging read "resource.type=cloud_run_revision" --limit 100

# Test connectivity
gcloud run services proxy ai-gpu-service-east --port 8080 --region us-east4
```

## Security Considerations

### Container Security
- Use non-root user in containers
- Implement health checks
- Regular base image updates
- Vulnerability scanning

### Network Security
- Private Google Access
- VPC connector (if needed)
- SSL/TLS enforcement
- Authentication required

### Access Control
- Service account with minimal permissions
- IAM bindings review
- Secret Manager for sensitive data
- Audit logging enabled

## Cost Optimization

### Resource Optimization
```bash
# Set appropriate limits
gcloud run services update ai-gpu-service-east \
  --region us-east4 \
  --memory 4Gi \
  --cpu 2 \
  --min-instances 0 \
  --max-instances 10 \
  --concurrency 100
```

### Monitoring Costs
```bash
# Check current costs
gcloud billing budgets list

# Set up billing alerts
gcloud alpha billing budgets create \
  --billing-account BILLING_ACCOUNT_ID \
  --display-name "AICRUSH Budget Alert" \
  --budget-amount 200USD
```

## Best Practices

### Deployment Best Practices
1. **Blue-Green Deployments**: Use traffic splitting for safe deployments
2. **Health Checks**: Always implement comprehensive health checks
3. **Resource Limits**: Set appropriate CPU and memory limits
4. **Auto-scaling**: Configure based on actual usage patterns
5. **Monitoring**: Implement comprehensive monitoring and alerting

### Security Best Practices
1. **Least Privilege**: Minimal IAM permissions
2. **Secret Management**: Use Secret Manager for sensitive data
3. **Network Security**: Private networking where possible
4. **Regular Updates**: Keep base images and dependencies updated
5. **Audit Logs**: Enable and monitor audit logging

For additional support, contact [josh@nibertinvestments.com](mailto:josh@nibertinvestments.com).