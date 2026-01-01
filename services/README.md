# Microservices

This directory contains all backend microservices for the AQMRG AI Analytics Platform.

## Services Overview

### api-gateway

Central entry point for all client requests. Handles:

- Request routing to appropriate services
- Rate limiting and throttling
- CORS configuration for frontend
- Request/response logging
- API documentation serving (OpenAPI/Swagger)

**Port**: 8000  
**Tech**: Node.js/Express or Python/FastAPI  
**Dependencies**: All downstream services

### auth-service

Authentication and authorization service. Handles:

- JWT token generation and validation
- Refresh token mechanism
- Role-based access control (Public, Authenticated, Admin)
- User session management
- Password hashing and security

**Port**: 8001  
**Tech**: Node.js/Express or Python/FastAPI  
**Database**: PostgreSQL

### data-ingestion-service

Real-time sensor data streaming and validation. Handles:

- Kafka producer integration
- Data validation and sanitization
- Multi-sensor manufacturer support
- Data quality checks
- Real-time data routing

**Port**: 8002  
**Tech**: Node.js or Python  
**Dependencies**: Kafka, InfluxDB

### model-serving-service

ML model inference and prediction endpoints. Handles:

- Model loading and versioning
- Prediction API endpoints
- Model performance monitoring
- A/B testing support
- Batch prediction capabilities

**Port**: 8003  
**Tech**: Python/FastAPI  
**Dependencies**: TensorFlow Serving, MLflow

### analytics-service

Data aggregation, processing, and reporting APIs. Handles:

- Time-series data aggregation
- Statistical analysis
- Custom report generation
- Data visualization preparation
- Historical data queries

**Port**: 8004  
**Tech**: Node.js or Python  
**Dependencies**: PostgreSQL, InfluxDB, Redis

### notification-service

Alert generation and delivery management. Handles:

- Threshold-based alert triggering
- Multi-channel delivery (email, SMS, push)
- Alert configuration management
- Notification scheduling
- Delivery status tracking

**Port**: 8005  
**Tech**: Node.js or Python  
**Dependencies**: Redis, Email/SMS providers

### sensor-adapter-service

Multi-manufacturer sensor API integration hub. Handles:

- Unified sensor data interface
- Manufacturer-specific adapter loading
- Protocol translation
- Connection pooling
- Error handling and retries

**Port**: 8006  
**Tech**: Node.js or Python  
**Dependencies**: Various sensor APIs

### export-service

Data export and custom report generation. Handles:

- CSV, JSON, Excel export formats
- Custom date range queries
- Report template management
- Large file handling
- Download link generation

**Port**: 8007  
**Tech**: Node.js or Python  
**Dependencies**: PostgreSQL, InfluxDB

## Service Communication

Services communicate via:

- **Synchronous**: REST APIs (service-to-service calls)
- **Asynchronous**: Kafka message queues
- **Service Discovery**: Kubernetes DNS or Consul

## Development

Each service follows a standard structure:

```
service-name/
├── src/
├── tests/
├── Dockerfile
├── package.json or requirements.txt
├── .env.example
└── README.md
```

## Running Services

### Individual Service

```bash
cd service-name
npm install  # or pip install -r requirements.txt
npm run dev  # or python main.py
```

### All Services (Docker Compose)

```bash
docker-compose up -d
```

## Health Checks

All services expose:

- `GET /health` - Basic health check
- `GET /ready` - Readiness probe
- `GET /metrics` - Prometheus metrics

## Common Environment Variables

```bash
NODE_ENV=development
LOG_LEVEL=info
DATABASE_URL=postgresql://...
REDIS_URL=redis://...
KAFKA_BROKERS=localhost:9092
JWT_SECRET=your-secret-key
```

See individual service READMEs for specific configuration.
