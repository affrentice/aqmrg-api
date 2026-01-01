# AQMRG AI Analytics Backend Platform

> A microservices-based backend system providing real-time air quality monitoring, prediction, and analytics APIs

## Overview

This monorepo contains the complete backend infrastructure for AQMRG's AI Analytics Platform. It provides APIs for air quality data ingestion, processing, ML-powered predictions, and advanced analytics, designed to be consumed by the existing React/Sanity frontend.

## Architecture

The platform follows a microservices architecture with the following key components:

- **API Gateway**: Central entry point for all client requests
- **Authentication Service**: JWT-based auth with role-based access control
- **Data Ingestion**: Real-time sensor data streaming and validation
- **Model Serving**: ML model inference and prediction endpoints
- **Analytics Service**: Data processing and reporting
- **Notification Service**: Alert generation and delivery
- **Sensor Adapters**: Multi-manufacturer sensor integration
- **Export Service**: Data export and custom reports

## Technology Stack

### Backend Services

- **Runtime**: Node.js (Express/NestJS) or Python (FastAPI/Django)
- **API**: REST + optional GraphQL
- **Authentication**: JWT with refresh tokens

### Data & Streaming

- **Message Queue**: Apache Kafka
- **Relational DB**: PostgreSQL
- **Time-Series DB**: InfluxDB
- **Cache**: Redis

### ML/AI Stack

- **Model Serving**: TensorFlow Serving
- **Experiment Tracking**: MLflow
- **ML Libraries**: TensorFlow, PyTorch, scikit-learn
- **Workflow Orchestration**: Apache Airflow

### Infrastructure

- **Containerization**: Docker
- **Orchestration**: Kubernetes
- **IaC**: Terraform
- **Monitoring**: Prometheus, Grafana
- **Logging**: ELK Stack

## Project Structure

```
.
├── services/                    # Microservices
│   ├── api-gateway/            # Request routing, rate limiting, CORS
│   ├── auth-service/           # JWT authentication & authorization
│   ├── data-ingestion-service/ # Real-time sensor data streaming
│   ├── model-serving-service/  # ML model inference endpoints
│   ├── analytics-service/      # Data processing & reporting APIs
│   ├── notification-service/   # Alert generation & delivery
│   ├── sensor-adapter-service/ # Multi-manufacturer integration
│   └── export-service/         # Data export & report generation
│
├── ml/                         # Machine Learning
│   ├── models/                 # Trained model artifacts
│   ├── training/               # Training scripts & experiments
│   ├── feature-engineering/    # Feature extraction pipelines
│   ├── model-registry/         # MLflow configurations
│   └── evaluation/             # Model validation & testing
│
├── data-pipeline/              # Data Processing
│   ├── kafka/                  # Kafka producers, consumers, topics
│   ├── airflow/                # Airflow DAGs for ETL
│   ├── stream-processors/      # Real-time transformations
│   ├── sensor-adapters/        # Manufacturer-specific adapters
│   └── data-validators/        # Quality checks & anomaly detection
│
├── infrastructure/             # Infrastructure as Code
│   ├── kubernetes/             # K8s manifests
│   ├── terraform/              # Cloud infrastructure
│   ├── docker/                 # Dockerfiles & base images
│   ├── helm/                   # Helm charts
│   └── monitoring/             # Prometheus, Grafana configs
│
├── databases/                  # Database Schemas
│   ├── postgres/               # PostgreSQL migrations & schemas
│   ├── influxdb/               # Time-series configurations
│   └── redis/                  # Cache schemas
│
├── shared/                     # Shared Code
│   ├── proto/                  # Protocol buffer definitions
│   ├── types/                  # TypeScript/Python types
│   ├── utils/                  # Common utilities
│   ├── config/                 # Configuration schemas
│   ├── constants/              # API codes, errors, enums
│   ├── middleware/             # Auth, logging, error handling
│   └── validators/             # Request/response validation
│
├── api-contracts/              # API Specifications
│   ├── openapi/                # OpenAPI/Swagger specs
│   ├── graphql/                # GraphQL schemas
│   └── asyncapi/               # Async API specs
│
├── scripts/                    # Automation Scripts
│   ├── deployment/             # Deployment automation
│   ├── database/               # DB management utilities
│   ├── monitoring/             # Health checks & diagnostics
│   ├── data-migration/         # Data migration tools
│   └── seed-data/              # Development data seeding
│
├── tests/                      # Testing
│   ├── integration/            # Cross-service tests
│   ├── e2e/                    # End-to-end API tests
│   ├── load/                   # Performance & load tests
│   └── contract/               # API contract tests
│
├── docs/                       # Documentation
│   ├── api/                    # API documentation
│   ├── architecture/           # Architecture diagrams & ADRs
│   ├── deployment/             # Deployment guides
│   ├── integration/            # Third-party integration guides
│   └── runbooks/               # Operational runbooks
│
└── config/                     # Configuration
    ├── environments/           # Environment-specific configs
    └── feature-flags/          # Feature toggles
```

## API Endpoints

### Public Endpoints (No Authentication)

- `GET /api/v1/dashboard/realtime` - Real-time air quality dashboard data
- `GET /api/v1/predictions/current` - Current air quality predictions

### Optional Authentication (Enhanced Features)

- `GET /api/v1/analytics/*` - Advanced analytics and filtering
- `POST /api/v1/data/export` - Data export and custom reports
- `POST /api/v1/alerts/*` - Alert configuration

### Admin Only

- `POST /api/v1/admin/models/*` - Model deployment and management
- `POST /api/v1/admin/sensors/*` - Sensor configuration
- `GET /api/v1/admin/monitoring/*` - System monitoring

### Authentication

- `POST /api/v1/auth/login` - User login
- `POST /api/v1/auth/refresh` - Refresh access token
- `POST /api/v1/auth/logout` - User logout

## Getting Started

### Prerequisites

- Docker & Docker Compose
- Node.js 18+ (if using Node.js services)
- Python 3.10+ (if using Python services)
- Kubernetes cluster (for production deployment)
- Terraform (for infrastructure provisioning)

### Local Development Setup

1. **Clone the repository**

   ```bash
   git clone https://github.com/affrentice/aqmrg-api.git
   cd aqmrg-api
   ```

2. **Set up environment variables**

   ```bash
   cp .env.example .env
   # Edit .env with your configuration
   ```

3. **Start all services with Docker Compose**

   ```bash
   docker-compose up -d
   ```

4. **Run database migrations**

   ```bash
   make migrate
   ```

5. **Seed development data**

   ```bash
   make seed
   ```

6. **Access services**
   - API Gateway: http://localhost:8000
   - Grafana: http://localhost:3000
   - MLflow: http://localhost:5000

### Running Individual Services

Each service can be run independently for development:

```bash
cd services/api-gateway
npm install
npm run dev
```

## Development Workflow

### Adding a New Service

1. Create service directory under `/services`
2. Follow the service template structure
3. Add service to `docker-compose.yml`
4. Update API gateway routing
5. Add integration tests
6. Update documentation

### Deploying ML Models

1. Train and validate model
2. Register in MLflow
3. Export model artifacts to `/ml/models`
4. Update model serving service
5. Deploy via admin API

### Adding Sensor Adapters

1. Use template in `/data-pipeline/sensor-adapters/template`
2. Implement required interfaces
3. Add configuration to adapter service
4. Test data ingestion pipeline
5. Deploy and monitor

## Testing

```bash
# Run all tests
make test

# Run integration tests
make test-integration

# Run load tests
make test-load

# Run contract tests
make test-contract
```

## Deployment

### Staging Deployment

```bash
make deploy-staging
```

### Production Deployment

```bash
make deploy-production
```

### Infrastructure Provisioning

```bash
cd infrastructure/terraform
terraform init
terraform plan
terraform apply
```

## Monitoring & Observability

- **Metrics**: Prometheus + Grafana dashboards
- **Logs**: ELK Stack (Elasticsearch, Logstash, Kibana)
- **Tracing**: Jaeger for distributed tracing
- **Alerts**: Alertmanager for critical notifications

Access monitoring dashboards:

- Grafana: http://localhost:3000
- Prometheus: http://localhost:9090
- Kibana: http://localhost:5601

## Authentication Tiers

### Public Tier

- No authentication required
- Access to real-time dashboards
- Basic air quality predictions
- Current conditions

### Professional Tier (Optional Auth)

- Enhanced analytics features
- Custom data exports
- Personalized alerts
- Historical data access

### Admin Tier

- Model deployment
- System configuration
- Sensor management
- Performance monitoring

## Security

- All APIs use HTTPS in production
- JWT-based authentication with refresh tokens
- Role-based access control (RBAC)
- Rate limiting on all endpoints
- API key authentication for external integrations
- Secrets managed via external secret management (e.g., AWS Secrets Manager)

## Performance Targets

- **Real-time Processing**: Dashboard updates within 30 seconds
- **Concurrent Users**: Support 1,000+ simultaneous users
- **Prediction Latency**: <5 seconds for forecast generation
- **Data Throughput**: Handle 10,000 sensor readings/minute
- **Uptime**: 99.5% availability

## Contributing

1. Create a feature branch
2. Make your changes
3. Add tests
4. Update documentation
5. Submit pull request

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

The MIT License is a permissive open-source license that allows:

- ✅ Commercial use
- ✅ Modification
- ✅ Distribution
- ✅ Private use

## Support

For questions or issues:

- **Email**: hello@affrentice.com
- **Issue Tracker**: https://github.com/affrentice/aqmrg-api/issues
- **Discussions**: https://github.com/affrentice/aqmrg-api/discussions

## Changelog

See [CHANGELOG.md](CHANGELOG.md) for version history and release notes.

---

**Maintained by**: AQMRG Development Team
**Last Updated**: January 2026
