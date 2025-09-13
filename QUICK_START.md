# 🚀 Quick Start Guide - AICRUSH

Get up and running with AICRUSH development environment in minutes. This guide will help you set up your local development environment and connect to our Google Cloud infrastructure.

## 📋 Table of Contents

- [Prerequisites](#prerequisites)
- [Environment Setup](#environment-setup)
- [Verification](#verification)
- [Development Workflow](#development-workflow)
- [Troubleshooting](#troubleshooting)
- [Next Steps](#next-steps)

## ✅ Prerequisites

Before you begin, ensure you have the following installed:

### **Required Software**
- **Python 3.8+** - [Download Python](https://python.org/downloads/)
- **Google Cloud SDK** - [Installation Guide](https://cloud.google.com/sdk/docs/install)
- **Git** - [Download Git](https://git-scm.com/downloads)
- **VS Code** (recommended) - [Download VS Code](https://code.visualstudio.com/)

### **Required Access**
- Google Cloud Project access: `gen-lang-client-0311226580`
- GitHub repository access: `nibertinvestments/AICRUSH`
- Service account credentials (provided separately)

### **Verify Prerequisites**
```bash
# Check Python version
python --version  # Should be 3.8+

# Check Google Cloud SDK
gcloud --version

# Check Git
git --version
```

## 🔧 Environment Setup

### **1. Clone the Repository**
```bash
# Clone the repository
git clone https://github.com/nibertinvestments/AICRUSH.git
cd AICRUSH

# Create a new branch for your work
git checkout -b feature/your-feature-name
```

### **2. Set Up Python Environment**
```bash
# Create virtual environment
python -m venv venv

# Activate virtual environment
# On macOS/Linux:
source venv/bin/activate
# On Windows:
venv\Scripts\activate

# Upgrade pip
pip install --upgrade pip
```

### **3. Configure Google Cloud**
```bash
# Authenticate with Google Cloud
gcloud auth login

# Set the project
gcloud config set project gen-lang-client-0311226580

# Set the region
gcloud config set run/region us-east4

# Verify configuration
gcloud config list
```

### **4. Environment Variables**
```bash
# Copy the environment template
cp .env.template .env

# Edit .env with your configuration
# (Never commit the .env file - it's in .gitignore)
```

**Example .env file:**
```bash
# Google Cloud Configuration
GOOGLE_CLOUD_PROJECT=gen-lang-client-0311226580
STORAGE_BUCKET=ai-storage-bucket-6658
GPU_REGION=us-east4

# Development Configuration  
DEBUG=true
LOG_LEVEL=info

# Service URLs
CLOUD_RUN_URL=https://ai-gpu-service-east-249675450204.us-east4.run.app
```

## ✅ Verification

### **Test Google Cloud Connection**
```bash
# Test authentication
gcloud auth list

# Test project access
gcloud projects describe gen-lang-client-0311226580

# Test Cloud Run service
gcloud run services describe ai-gpu-service-east --region us-east4
```

### **Test Storage Access**
```bash
# List storage buckets
gsutil ls

# Test bucket access
gsutil ls gs://ai-storage-bucket-6658/ | head -10
```

### **Test Service Endpoint**
```bash
# Get authentication token
TOKEN=$(gcloud auth print-identity-token)

# Test the service endpoint
curl -H "Authorization: Bearer $TOKEN" \
     https://ai-gpu-service-east-249675450204.us-east4.run.app/test-all
```

**Expected response:**
```json
{
  "status": "success",
  "gpu": "NVIDIA L4",
  "storage": "connected",
  "vertex_ai": "connected"
}
```

## 🔄 Development Workflow

### **Daily Development Commands**

1. **Start Development Session**
   ```bash
   cd AICRUSH
   source venv/bin/activate  # or venv\Scripts\activate on Windows
   git pull origin main
   ```

2. **Make Changes**
   ```bash
   # Create feature branch
   git checkout -b feature/your-feature
   
   # Make your changes
   # ... edit files ...
   
   # Test your changes locally
   python test_script.py  # (when available)
   ```

3. **Submit Changes**
   ```bash
   # Stage and commit changes
   git add .
   git commit -m "feat: add your feature description"
   
   # Push to GitHub
   git push origin feature/your-feature
   
   # Create pull request on GitHub
   ```

### **Common Tasks**

#### **Update Dependencies**
```bash
# Activate virtual environment
source venv/bin/activate

# Install/update requirements (when available)
pip install -r requirements.txt

# Install development dependencies (when available)
pip install -r requirements-dev.txt
```

#### **Deploy to Cloud Run**
```bash
# Build and deploy (when available)
gcloud builds submit --tag us-central1-docker.pkg.dev/gen-lang-client-0311226580/ai-gpu-repo/ai-gpu-container:latest

# Update service
gcloud run services update ai-gpu-service-east \
  --image us-central1-docker.pkg.dev/gen-lang-client-0311226580/ai-gpu-repo/ai-gpu-container:latest \
  --region us-east4
```

## 🐛 Troubleshooting

### **Common Issues**

#### **Authentication Error**
```bash
# Error: (gcloud.auth.application-default.login) 
# Solution: Re-authenticate
gcloud auth application-default login
gcloud auth login
```

#### **Permission Denied**
```bash
# Error: Permission denied for project
# Solution: Verify project access
gcloud projects get-iam-policy gen-lang-client-0311226580
# Contact josh@nibertinvestments.com if you need access
```

#### **Service Unavailable**
```bash
# Error: Service endpoint not responding
# Solution: Check service status
gcloud run services describe ai-gpu-service-east --region us-east4

# Check logs
gcloud logging read "resource.type=cloud_run_revision AND resource.labels.service_name=ai-gpu-service-east" --limit 50 --format json
```

#### **Python Environment Issues**
```bash
# Error: Command not found or import errors
# Solution: Verify virtual environment
which python  # Should point to venv/bin/python
pip list  # Check installed packages

# Recreate if needed
deactivate
rm -rf venv
python -m venv venv
source venv/bin/activate
```

### **Getting Help**

If you encounter issues not covered here:

1. **Check Documentation**: Review [docs/](../docs/) directory
2. **Search Issues**: Look through [GitHub Issues](https://github.com/nibertinvestments/AICRUSH/issues)
3. **Create Issue**: Report new issues with detailed information
4. **Contact Support**: Email [josh@nibertinvestments.com](mailto:josh@nibertinvestments.com)

## 🎯 Next Steps

Once your environment is set up:

1. **📖 Read Documentation**
   - [Contributing Guide](../CONTRIBUTING.md)
   - [Copilot Instructions](../Copilot-instructions.md)
   - [Architecture Overview](../docs/architecture/)

2. **🧪 Explore the Code**
   - Review existing scripts and configurations
   - Understand the data processing pipeline
   - Study the Cloud Run deployment setup

3. **🛠️ Start Contributing**
   - Look for "good first issue" labels
   - Review the project roadmap
   - Submit your first pull request

4. **📊 Monitor Resources**
   - [Google Cloud Console](https://console.cloud.google.com/home/dashboard?project=gen-lang-client-0311226580)
   - [Cloud Run Services](https://console.cloud.google.com/run?project=gen-lang-client-0311226580)
   - [Storage Buckets](https://console.cloud.google.com/storage/browser?project=gen-lang-client-0311226580)

## 🔗 Quick Links

- **📁 Repository**: [GitHub Repository](https://github.com/nibertinvestments/AICRUSH)
- **☁️ Cloud Console**: [Google Cloud Dashboard](https://console.cloud.google.com/home/dashboard?project=gen-lang-client-0311226580)
- **🚀 Service**: [Cloud Run Service](https://ai-gpu-service-east-249675450204.us-east4.run.app/)
- **💾 Storage**: [Cloud Storage](https://console.cloud.google.com/storage/browser/ai-storage-bucket-6658)
- **📧 Support**: [josh@nibertinvestments.com](mailto:josh@nibertinvestments.com)

---

**🎉 Welcome to AICRUSH development!** 

You're now ready to contribute to building practical, reliable AI coding tools. If you have any questions, don't hesitate to reach out to the team.
