# REST API Services

This directory contains the REST API implementation for the telecom fraud detection system. The API provides endpoints for fraud detection, system management, and integration with external systems.

## API Overview

The REST API is built using FastAPI and provides the following capabilities:
- Real-time fraud detection
- Batch fraud analysis
- System configuration management
- Model management
- Reporting and analytics
- Health and status monitoring

## Key Components

- `app.py` - Main FastAPI application
- `routes/` - API route definitions
- `middleware/` - API middleware components
- `models/` - Pydantic models for request/response validation
- `services/` - Business logic services used by routes

## API Routes

### Fraud Detection

- `POST /api/v1/detect` - Real-time fraud detection for a single event
- `POST /api/v1/detect/batch` - Batch fraud detection for multiple events
- `GET /api/v1/cases/{case_id}` - Get details of a fraud case
- `GET /api/v1/cases` - List fraud cases with filtering
- `POST /api/v1/feedback` - Submit feedback on a fraud detection

### Model Management

- `GET /api/v1/models` - List available models
- `GET /api/v1/models/{model_id}` - Get model details
- `POST /api/v1/models/{model_id}/deploy` - Deploy a model
- `GET /api/v1/models/{model_id}/metrics` - Get model performance metrics

### System Management

- `GET /api/v1/config` - Get system configuration
- `PATCH /api/v1/config` - Update system configuration
- `GET /api/v1/rules` - List business rules
- `POST /api/v1/rules` - Create a new business rule
- `PUT /api/v1/rules/{rule_id}` - Update a business rule
- `DELETE /api/v1/rules/{rule_id}` - Delete a business rule

### Monitoring and Health

- `GET /health` - Health check endpoint
- `GET /ready` - Readiness check endpoint
- `GET /metrics` - Prometheus metrics endpoint
- `GET /api/v1/stats` - System statistics

## API Documentation

The API is documented using OpenAPI (Swagger) and is available at:
- Development: `http://localhost:8080/docs`
- Testing: `http://api.test.telecom-fraud.internal/docs`
- Production: `http://api.prod.telecom-fraud.internal/docs`

## Authentication and Authorization

The API uses:
- OAuth 2.0 with JWT for authentication
- Role-based access control for authorization
- API keys for service-to-service communication
- Rate limiting to prevent abuse

## Example Usage

### Real-time Fraud Detection

```python
import requests
import json

# API endpoint
url = "http://localhost:8080/api/v1/detect"

# Authentication
headers = {
    "Authorization": "Bearer {token}",
    "Content-Type": "application/json"
}

# CDR data
payload = {
    "call_id": "12345",
    "timestamp": 1625097600,
    "caller_number": "+1234567890",
    "called_number": "+9876543210",
    "call_duration": 120,
    "call_type": "VOICE",
    "call_direction": "OUTBOUND",
    "switch_id": "SW001",
    "trunk_id": "TR001",
    "billing_type": "POSTPAID",
    "call_status": "COMPLETED",
    "disconnect_reason": "NORMAL",
    "source_country": "US",
    "destination_country": "XYZ",
    "roaming_status": False,
    "service_type": "STANDARD"
}

# Make request
response = requests.post(url, headers=headers, data=json.dumps(payload))

# Process response
result = response.json()
print(f"Fraud detection result: {result}")
```

### Response Example

```json
{
  "call_id": "12345",
  "timestamp": 1625097600,
  "risk_score": 0.87,
  "fraud_type": "IRSF",
  "confidence": 0.92,
  "recommended_action": "BLOCK",
  "explanation": [
    "Call to high-risk country XYZ",
    "Unusual call pattern detected",
    "IRSF fraud score: 0.92",
    "Rule triggered: High Risk Country Call (Call to a country with high IRSF risk)"
  ]
}
```

## Performance Characteristics

The API is designed for high performance:
- Response time < 100ms for real-time detection
- Throughput of 1000+ requests per second per instance
- Horizontal scaling for higher loads
- Connection pooling for database and service connections
- Caching for frequent requests

## Error Handling

The API implements comprehensive error handling:
- Standardized error responses
- Detailed error codes and messages
- Validation errors for request data
- Graceful handling of downstream service failures
- Comprehensive logging of errors

## Deployment

The API is deployed as a containerized service:
- Docker container with optimized Python runtime
- Kubernetes deployment with auto-scaling
- Health and readiness probes
- Resource limits and requests
- Rolling updates for zero-downtime deployment