# Copilot Instructions for AICRUSH Repository

## Project Overview

AICRUSH is an AI coding assistant project focused on building reliable, accurate code generation tools. This repository contains the infrastructure, data processing pipelines, and development environment for training and deploying AI models that assist with code generation across multiple programming languages.

## Repository-Specific Guidelines

### 🎯 Project Goals and Context
- **Primary Objective**: Build a practical, reliable AI coding assistant
- **Focus**: Accuracy and production-ready code generation over flashy features
- **Technology Stack**: Python, Google Cloud Platform, Docker, Cloud Run
- **Current Phase**: Data collection and infrastructure (Phase 1 complete, Phase 2 in progress)

### 🏗️ Architecture and Infrastructure
- **Cloud Platform**: Google Cloud (Project: gen-lang-client-0311226580)
- **Compute**: Cloud Run with NVIDIA L4 GPU (us-east4 region)
- **Storage**: Google Cloud Storage (ai-storage-bucket-6658)
- **Deployment**: Containerized services with auto-scaling
- **Cost Management**: $66-145/month with scale-to-zero capability

## 🔧 Development Standards

### Code Quality Standards
```python
# Example: Follow these patterns for Python code
class DataProcessor:
    """Process training data for AI model training.
    
    This class handles data validation, preprocessing,
    and storage operations for the AICRUSH training pipeline.
    """
    
    def __init__(self, config: Dict[str, Any]) -> None:
        self.config = config
        self.storage_client = None
        
    def validate_data(self, data: List[Dict]) -> bool:
        """Validate training data format and quality."""
        # Implementation with proper error handling
        pass
```

### Security-First Development
- **NEVER commit credentials**: Use environment variables and Google Cloud IAM
- **Service Account Management**: Use minimal permissions and rotate keys regularly
- **Secret Management**: All sensitive data in .env files (gitignored) or GitHub Secrets
- **Code Review**: All changes require PR review before merging to main

### File and Directory Structure
```
AICRUSH/
├── docs/                    # Comprehensive documentation
│   ├── api/                # API documentation
│   ├── architecture/       # System design docs
│   └── deployment/         # Deployment guides
├── src/                    # Source code (when added)
│   ├── data_processing/    # Data collection and processing
│   ├── training/          # Model training scripts
│   └── deployment/        # Deployment configurations
├── tests/                 # Test suites
├── scripts/               # Utility scripts
└── .github/               # GitHub workflows and templates
```

## 📝 Documentation Standards

### README Structure
- Clear project description with badges
- Current status and roadmap
- Quick start guide
- Contributing guidelines
- License information

### Code Documentation
```python
def process_training_data(
    input_files: List[str], 
    output_bucket: str,
    validation_threshold: float = 0.95
) -> ProcessingResult:
    """Process raw code files for AI training.
    
    Args:
        input_files: List of file paths to process
        output_bucket: GCS bucket for processed data
        validation_threshold: Minimum quality score (0.0-1.0)
        
    Returns:
        ProcessingResult with success status and metrics
        
    Raises:
        ValidationError: If data quality below threshold
        StorageError: If unable to write to bucket
    """
```

### Commit Message Standards
```
feat: add data validation pipeline for training files
fix: resolve GPU memory leak in model training
docs: update API documentation for v2.0 endpoints
security: rotate service account keys and update IAM
refactor: optimize data processing for large files
test: add integration tests for Cloud Run deployment
```

## 🔒 Security Guidelines

### Cloud Security
- Use Google Cloud IAM for all access control
- Enable audit logging on all services
- Implement least-privilege access principles
- Regular security reviews and access audits

### Development Security
- Use GitHub branch protection rules
- Enable secret scanning and push protection
- Require signed commits for production releases
- Regular dependency security updates

### Environment Management
```bash
# .env.template (commit this)
GOOGLE_CLOUD_PROJECT=your-project-id
STORAGE_BUCKET=your-bucket-name
GPU_REGION=us-east4

# .env (never commit this)
GOOGLE_APPLICATION_CREDENTIALS=/path/to/service-account.json
API_KEY=actual-secret-key
```

## 🧪 Testing Standards

### Test Categories
1. **Unit Tests**: Individual component testing
2. **Integration Tests**: Service-to-service communication
3. **Cloud Tests**: GCP service integration
4. **Security Tests**: Vulnerability and access control validation

### Test File Structure
```python
# tests/test_data_processor.py
import pytest
from unittest.mock import Mock, patch
from src.data_processing.processor import DataProcessor

class TestDataProcessor:
    """Test suite for DataProcessor class."""
    
    def setup_method(self):
        """Set up test fixtures."""
        self.config = {"bucket": "test-bucket"}
        self.processor = DataProcessor(self.config)
    
    def test_validate_data_success(self):
        """Test successful data validation."""
        # Test implementation
        pass
```

## 🚀 Deployment Guidelines

### Cloud Run Deployment
```yaml
# cloudbuild.yaml example
steps:
  - name: 'gcr.io/cloud-builders/docker'
    args: ['build', '-t', 'gcr.io/$PROJECT_ID/ai-service:$SHORT_SHA', '.']
  - name: 'gcr.io/cloud-builders/docker'
    args: ['push', 'gcr.io/$PROJECT_ID/ai-service:$SHORT_SHA']
  - name: 'gcr.io/cloud-builders/gcloud'
    args: [
      'run', 'deploy', 'ai-service',
      '--image', 'gcr.io/$PROJECT_ID/ai-service:$SHORT_SHA',
      '--region', 'us-east4',
      '--platform', 'managed'
    ]
```

### Environment-Specific Configurations
- **Development**: Local development with service account keys
- **Staging**: Cloud-based testing with limited resources
- **Production**: Full auto-scaling with monitoring and alerting

## 📊 Performance and Monitoring

### Key Metrics to Track
- **Training Data Quality**: Validation scores, file processing rates
- **Model Performance**: Accuracy, inference time, resource usage
- **Infrastructure**: Cost, scaling events, error rates
- **Security**: Failed authentication attempts, unusual access patterns

### Monitoring Tools
- Google Cloud Monitoring for infrastructure
- Application-level logging for debugging
- Performance profiling for optimization
- Security audit logs for compliance

## 🔄 CI/CD Pipeline Standards

### GitHub Actions Workflow
```yaml
name: AICRUSH CI/CD
on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  security-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Run security scan
        run: |
          # Security scanning commands
  
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Run tests
        run: |
          # Test execution commands
```

## 🤝 Collaboration Guidelines

### Code Review Process
1. **Create Feature Branch**: `git checkout -b feature/description`
2. **Implement Changes**: Follow coding standards above
3. **Write Tests**: Ensure adequate test coverage
4. **Submit PR**: Use PR template with checklist
5. **Code Review**: Address reviewer feedback
6. **Merge**: Squash and merge after approval

### Communication Standards
- **Issues**: Use templates for bug reports and feature requests
- **Discussions**: Use GitHub Discussions for architecture decisions
- **Documentation**: Update docs alongside code changes
- **Security**: Report security issues privately to maintainers

## 🎓 Learning and Development

### Recommended Reading
- Google Cloud AI/ML best practices
- Python asyncio and concurrent programming
- Docker containerization for ML workloads
- Infrastructure as Code (Terraform/Pulumi)

### Code Examples and Patterns
```python
# Async data processing pattern
async def process_batch(files: List[str]) -> List[ProcessingResult]:
    """Process multiple files concurrently."""
    tasks = [process_single_file(file) for file in files]
    return await asyncio.gather(*tasks, return_exceptions=True)

# Error handling pattern
def safe_cloud_operation(operation: Callable) -> Result:
    """Safely execute cloud operations with retry logic."""
    for attempt in range(3):
        try:
            return operation()
        except (CloudError, TimeoutError) as e:
            if attempt == 2:
                raise
            time.sleep(2 ** attempt)
```

## 📋 Checklist Templates

### Pre-Commit Checklist
- [ ] Code follows style guidelines
- [ ] Tests pass locally
- [ ] Documentation updated
- [ ] No credentials in code
- [ ] Security scan clean
- [ ] Performance impact considered

### PR Review Checklist
- [ ] Code quality standards met
- [ ] Security review completed
- [ ] Test coverage adequate
- [ ] Documentation accurate
- [ ] Performance acceptable
- [ ] Cloud costs considered

---

**Remember**: AICRUSH is focused on practical, reliable AI coding assistance. Every contribution should advance this goal while maintaining high standards for security, performance, and code quality.

For questions about these guidelines, please create a GitHub Discussion or contact the repository maintainers.