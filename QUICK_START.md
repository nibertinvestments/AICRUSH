# Quick Start Guide - AI GPU Service

## Current Working Service
- **URL**: https://ai-gpu-service-east-249675450204.us-east4.run.app/
- **Region**: us-east4
- **GPU**: NVIDIA L4 24GB
- **Status**: ✅ Fully Functional

## Quick Test
```powershell
$token = gcloud auth print-identity-token
Invoke-RestMethod -Uri "https://ai-gpu-service-east-249675450204.us-east4.run.app/test-all" -Headers @{"Authorization" = "Bearer $token"}
```

## Project Files Location
- Container code: `C:\Users\Josh\Documents\GitHub\AICRUSH\gpu-container\`
- Service name: `ai-gpu-service-east`
- Image: `us-central1-docker.pkg.dev/gen-lang-client-0311226580/ai-gpu-repo/ai-gpu-container:v3`

## Next Session Commands
```bash
# Check service status
gcloud run services describe ai-gpu-service-east --region us-east4

# If you need to update the service
cd C:\Users\Josh\Documents\GitHub\AICRUSH\gpu-container
gcloud builds submit --tag us-central1-docker.pkg.dev/gen-lang-client-0311226580/ai-gpu-repo/ai-gpu-container:v4
gcloud run services update ai-gpu-service-east --image us-central1-docker.pkg.dev/gen-lang-client-0311226580/ai-gpu-repo/ai-gpu-container:v4 --region us-east4
```

## All Systems Working
- ✅ GPU: NVIDIA L4 
- ✅ Bucket: ai-storage-bucket-6658
- ✅ Vertex AI: Connected
- ✅ Auto-scaling: Enabled
