# API Documentation

## Overview

The AICRUSH API provides programmatic access to our AI coding assistant capabilities. This RESTful API is designed for developers who want to integrate AI-powered code generation into their applications and workflows.

> **Note**: The API is currently in development. This documentation reflects the planned implementation.

## Base URL

```
Production: https://ai-gpu-service-east-249675450204.us-east4.run.app
Development: http://localhost:8080
```

## Authentication

### Bearer Token Authentication

All API requests require authentication using Google Cloud IAM tokens:

```bash
# Get authentication token
TOKEN=$(gcloud auth print-identity-token)

# Use in API requests
curl -H "Authorization: Bearer $TOKEN" \
     https://ai-gpu-service-east-249675450204.us-east4.run.app/api/v1/generate
```

### Service Account Authentication

For programmatic access:

```python
from google.auth import default
import requests

# Get default credentials
credentials, project = default()
token = credentials.token

headers = {"Authorization": f"Bearer {token}"}
response = requests.get(api_url, headers=headers)
```

## API Endpoints

### Health Check

#### `GET /health`

Check API service health and status.

**Response:**
```json
{
  "status": "healthy",
  "version": "1.0.0",
  "timestamp": "2024-01-15T10:30:00Z",
  "gpu": "NVIDIA L4",
  "model_loaded": true
}
```

### System Status

#### `GET /test-all`

Comprehensive system status check.

**Response:**
```json
{
  "status": "success",
  "gpu": "NVIDIA L4",
  "storage": "connected",
  "vertex_ai": "connected",
  "model_status": "ready",
  "performance": {
    "gpu_utilization": "15%",
    "memory_usage": "2.1GB/24GB",
    "response_time": "0.8s"
  }
}
```

## Code Generation API

### Generate Code

#### `POST /api/v1/generate`

Generate code based on natural language description or code context.

**Request Body:**
```json
{
  "prompt": "Create a Python function to validate email addresses",
  "language": "python",
  "context": {
    "file_type": "utility",
    "existing_code": "import re",
    "style": "google"
  },
  "options": {
    "max_length": 100,
    "temperature": 0.7,
    "include_comments": true,
    "include_tests": false
  }
}
```

**Response:**
```json
{
  "generated_code": "def validate_email(email: str) -> bool:\n    \"\"\"\n    Validate email address format.\n    \n    Args:\n        email: Email address to validate\n        \n    Returns:\n        bool: True if valid, False otherwise\n    \"\"\"\n    pattern = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}$'\n    return re.match(pattern, email) is not None",
  "confidence": 0.95,
  "language": "python",
  "metadata": {
    "lines_generated": 12,
    "processing_time": "1.2s",
    "model_version": "aicrush-v1.0"
  }
}
```

### Code Completion

#### `POST /api/v1/complete`

Complete partial code snippets.

**Request Body:**
```json
{
  "code": "def calculate_fibonacci(n):",
  "language": "python",
  "cursor_position": 25,
  "max_completions": 3
}
```

**Response:**
```json
{
  "completions": [
    {
      "code": "\n    if n <= 1:\n        return n\n    return calculate_fibonacci(n-1) + calculate_fibonacci(n-2)",
      "confidence": 0.92,
      "type": "recursive"
    },
    {
      "code": "\n    if n <= 1:\n        return n\n    a, b = 0, 1\n    for _ in range(2, n + 1):\n        a, b = b, a + b\n    return b",
      "confidence": 0.88,
      "type": "iterative"
    }
  ],
  "metadata": {
    "processing_time": "0.9s",
    "model_version": "aicrush-v1.0"
  }
}
```

### Code Review

#### `POST /api/v1/review`

Analyze code for quality, security, and best practices.

**Request Body:**
```json
{
  "code": "def process_user_input(input):\n    result = eval(input)\n    return result",
  "language": "python",
  "check_types": ["security", "performance", "style"]
}
```

**Response:**
```json
{
  "issues": [
    {
      "type": "security",
      "severity": "critical",
      "line": 2,
      "message": "Use of eval() is dangerous and can lead to code injection",
      "suggestion": "Use ast.literal_eval() for safe evaluation or validate input explicitly"
    },
    {
      "type": "style",
      "severity": "medium",
      "line": 1,
      "message": "Parameter name 'input' shadows built-in function",
      "suggestion": "Use a more descriptive parameter name like 'user_input'"
    }
  ],
  "score": 2.5,
  "max_score": 10,
  "metadata": {
    "lines_analyzed": 3,
    "processing_time": "0.6s"
  }
}
```

## Request/Response Format

### Standard Request Headers

```
Content-Type: application/json
Authorization: Bearer <token>
User-Agent: YourApp/1.0
```

### Standard Response Format

All API responses follow this structure:

```json
{
  "success": true,
  "data": { ... },
  "metadata": {
    "request_id": "req_123456789",
    "timestamp": "2024-01-15T10:30:00Z",
    "processing_time": "1.2s",
    "model_version": "aicrush-v1.0"
  },
  "rate_limit": {
    "remaining": 995,
    "reset_time": "2024-01-15T11:00:00Z"
  }
}
```

### Error Response Format

```json
{
  "success": false,
  "error": {
    "code": "INVALID_INPUT",
    "message": "The provided code is not valid Python syntax",
    "details": {
      "line": 5,
      "column": 12,
      "syntax_error": "unexpected token"
    }
  },
  "metadata": {
    "request_id": "req_123456789",
    "timestamp": "2024-01-15T10:30:00Z"
  }
}
```

## Error Codes

| Code | HTTP Status | Description |
|------|-------------|-------------|
| `INVALID_INPUT` | 400 | Request validation failed |
| `UNAUTHORIZED` | 401 | Authentication required |
| `FORBIDDEN` | 403 | Insufficient permissions |
| `NOT_FOUND` | 404 | Resource not found |
| `RATE_LIMITED` | 429 | Too many requests |
| `MODEL_UNAVAILABLE` | 503 | AI model temporarily unavailable |
| `INTERNAL_ERROR` | 500 | Internal server error |

## Rate Limits

### Current Limits

| Plan | Requests/Minute | Requests/Day | Concurrent |
|------|-----------------|--------------|------------|
| **Developer** | 60 | 1,000 | 5 |
| **Professional** | 300 | 10,000 | 20 |
| **Enterprise** | 1,000 | 100,000 | 50 |

### Rate Limit Headers

```
X-RateLimit-Limit: 60
X-RateLimit-Remaining: 59
X-RateLimit-Reset: 1642234567
```

## SDKs and Libraries

### Python SDK

```python
from aicrush import AIcrushClient

client = AIcrushClient(
    project_id="your-project-id",
    credentials_path="path/to/credentials.json"
)

# Generate code
result = client.generate_code(
    prompt="Create a function to validate email",
    language="python"
)

print(result.generated_code)
```

### JavaScript SDK

```javascript
import { AIcrushClient } from '@aicrush/sdk';

const client = new AIcrushClient({
  projectId: 'your-project-id',
  credentials: 'path/to/credentials.json'
});

// Generate code
const result = await client.generateCode({
  prompt: 'Create a function to validate email',
  language: 'javascript'
});

console.log(result.generatedCode);
```

## Code Examples

### Python Integration

```python
import requests
import json
from google.auth import default

class AIcrushAPI:
    def __init__(self, base_url):
        self.base_url = base_url
        self.credentials, _ = default()
        
    def get_headers(self):
        return {
            "Authorization": f"Bearer {self.credentials.token}",
            "Content-Type": "application/json"
        }
    
    def generate_code(self, prompt, language="python"):
        url = f"{self.base_url}/api/v1/generate"
        payload = {
            "prompt": prompt,
            "language": language
        }
        
        response = requests.post(
            url, 
            headers=self.get_headers(),
            json=payload
        )
        
        if response.status_code == 200:
            return response.json()["data"]
        else:
            raise Exception(f"API Error: {response.text}")

# Usage
api = AIcrushAPI("https://ai-gpu-service-east-249675450204.us-east4.run.app")
result = api.generate_code("Create a binary search function")
print(result["generated_code"])
```

### cURL Examples

```bash
# Generate code
curl -X POST \
  https://ai-gpu-service-east-249675450204.us-east4.run.app/api/v1/generate \
  -H "Authorization: Bearer $(gcloud auth print-identity-token)" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "Create a function to sort an array",
    "language": "python"
  }'

# Check system status
curl -X GET \
  https://ai-gpu-service-east-249675450204.us-east4.run.app/test-all \
  -H "Authorization: Bearer $(gcloud auth print-identity-token)"
```

## Best Practices

### Request Optimization

1. **Batch Requests**: Combine multiple small requests when possible
2. **Caching**: Cache responses for repeated requests
3. **Specificity**: Provide clear, specific prompts for better results
4. **Context**: Include relevant code context for better completions

### Error Handling

```python
def safe_api_call(api_func, *args, **kwargs):
    try:
        return api_func(*args, **kwargs)
    except requests.exceptions.Timeout:
        # Handle timeout
        return {"error": "Request timed out"}
    except requests.exceptions.HTTPError as e:
        # Handle HTTP errors
        return {"error": f"HTTP error: {e.response.status_code}"}
    except Exception as e:
        # Handle other errors
        return {"error": f"Unexpected error: {str(e)}"}
```

### Performance Tips

1. **Use appropriate timeouts**: Set reasonable request timeouts
2. **Handle rate limits**: Implement exponential backoff
3. **Monitor usage**: Track API usage and performance metrics
4. **Optimize prompts**: Use clear, concise prompts for faster processing

## Webhooks (Future)

### Event Notifications

Planned webhook support for real-time notifications:

```json
{
  "event": "model.updated",
  "timestamp": "2024-01-15T10:30:00Z",
  "data": {
    "model_version": "aicrush-v1.1",
    "improvements": ["Better Python support", "Faster inference"]
  }
}
```

## Support

### API Support
- **Documentation**: [API Docs](./README.md)
- **Status Page**: [Status](https://status.aicrush.com) (planned)
- **Support Email**: [josh@nibertinvestments.com](mailto:josh@nibertinvestments.com)

### Resources
- **GitHub Issues**: [Report Issues](https://github.com/nibertinvestments/AICRUSH/issues)
- **Discussions**: [Community](https://github.com/nibertinvestments/AICRUSH/discussions)
- **Updates**: [Changelog](../guides/changelog.md)

---

**Note**: This API is currently in development. Features and endpoints may change before the official release. Subscribe to our updates for the latest information.