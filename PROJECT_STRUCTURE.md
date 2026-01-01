# AQMRG AI Backend - Project Structure

Complete directory structure for the AQMRG AI Analytics Backend monorepo.

## 📁 Root Level Files

```
.
├── README.md              # Main project documentation
├── QUICK_START.md         # Quick start guide for developers
├── .gitignore            # Git ignore patterns
├── .env.example          # Environment variable template
├── docker-compose.yml    # Local development infrastructure
├── Makefile              # Common development tasks
└── directory_structure.txt # This file structure reference
```

## 📂 Main Directories

### `/services` - Microservices

All backend microservices for the platform.

```
services/
├── api-gateway/              # Central API entry point (Port 8000)
├── auth-service/             # Authentication & authorization (Port 8001)
├── data-ingestion-service/   # Real-time sensor data (Port 8002)
├── model-serving-service/    # ML predictions (Port 8003)
├── analytics-service/        # Data processing (Port 8004)
├── notification-service/     # Alerts & notifications (Port 8005)
├── sensor-adapter-service/   # Sensor integrations (Port 8006)
├── export-service/           # Data exports (Port 8007)
└── README.md                 # Services documentation
```

**Each service should contain:**

- `src/` - Source code
- `tests/` - Service tests
- `Dockerfile` - Container definition
- `package.json` or `requirements.txt` - Dependencies
- `.env.example` - Service-specific env vars
- `README.md` - Service documentation

### `/ml` - Machine Learning

All ML/AI components and model artifacts.

```
ml/
├── models/                   # Trained model artifacts
│   ├── predictive/          # Air quality predictions (4hr, 24hr, 72hr)
│   ├── correlation/         # Health-pollution correlations
│   └── anomaly-detection/   # Data quality & outliers
├── training/                # Training scripts & experiments
├── feature-engineering/     # Feature extraction pipelines
├── model-registry/          # MLflow configurations
├── evaluation/              # Model validation & testing
└── README.md               # ML documentation
```

**Key Technologies:**

- TensorFlow / PyTorch for model training
- MLflow for experiment tracking
- Scikit-learn for traditional ML

### `/data-pipeline` - Data Processing

Real-time and batch data processing infrastructure.

```
data-pipeline/
├── kafka/                   # Apache Kafka streaming
│   ├── producers/          # Data producers
│   ├── consumers/          # Data consumers
│   └── topics/             # Topic configurations
├── airflow/                 # Workflow orchestration
│   └── dags/               # ETL DAG definitions
├── stream-processors/       # Real-time transformations
├── sensor-adapters/         # Manufacturer-specific adapters
│   ├── airqo/              # AirQo integration
│   ├── purpleair/          # PurpleAir integration
│   └── template/           # Template for new adapters
├── data-validators/         # Quality checks
└── README.md               # Pipeline documentation
```

**Data Flow:**
Sensors → Kafka → Stream Processors → Databases
↓
Data Validators
↓
Airflow (Batch)

### `/infrastructure` - Infrastructure as Code

Deployment and infrastructure configurations.

```
infrastructure/
├── kubernetes/              # K8s orchestration
│   ├── namespaces/         # Environment isolation
│   ├── deployments/        # Service deployments
│   ├── services/           # Service definitions
│   ├── ingress/            # API gateway rules
│   ├── configmaps/         # Configuration
│   └── secrets/            # Secret templates
├── terraform/               # Cloud provisioning
│   ├── aws/                # AWS infrastructure
│   ├── azure/              # Azure infrastructure
│   ├── gcp/                # GCP infrastructure
│   └── modules/            # Reusable modules
├── docker/                  # Docker configs
│   └── base-images/        # Common base images
├── helm/                    # Helm charts
└── monitoring/              # Observability
    ├── prometheus/         # Metrics collection
    ├── grafana/            # Dashboards
    └── alertmanager/       # Alert routing
```

### `/databases` - Database Schemas

Database migrations, schemas, and configurations.

```
databases/
├── postgres/                # PostgreSQL
│   ├── migrations/         # Schema migrations
│   ├── schemas/            # SQL definitions
│   └── seeds/              # Test data
├── influxdb/                # Time-series DB
│   ├── schemas/            # Bucket configs
│   └── continuous-queries/ # Aggregations
└── redis/                   # Cache
    └── schemas/            # Data structures
```

### `/shared` - Shared Code

Common code used across all services.

```
shared/
├── proto/                   # Protocol buffers (gRPC)
├── types/                   # Type definitions
├── utils/                   # Utility functions
├── config/                  # Config schemas
├── constants/               # API codes, errors
├── middleware/              # Auth, logging, errors
├── validators/              # Request/response validation
└── README.md
```

### `/api-contracts` - API Specifications

API documentation and contracts.

```
api-contracts/
├── openapi/                 # OpenAPI/Swagger specs
│   └── v1/                 # API version 1
├── graphql/                 # GraphQL schemas
└── asyncapi/                # Async API specs
```

### `/scripts` - Automation Scripts

Development and operational scripts.

```
scripts/
├── deployment/              # Deployment automation
├── database/                # DB management
├── monitoring/              # Health checks
├── data-migration/          # Data migrations
└── seed-data/               # Dev data seeding
```

### `/tests` - Testing

All testing code organized by type.

```
tests/
├── integration/             # Cross-service tests
├── e2e/                     # End-to-end API tests
├── load/                    # Performance tests
└── contract/                # API contract tests
```

### `/docs` - Documentation

Comprehensive project documentation.

```
docs/
├── api/                     # API documentation
│   ├── postman/            # Postman collections
│   └── examples/           # API examples
├── architecture/            # System design
│   ├── diagrams/           # Architecture diagrams
│   └── decisions/          # ADRs
├── deployment/              # Deployment guides
├── integration/             # Integration guides
└── runbooks/                # Operational runbooks
```

### `/config` - Configuration

Environment-specific configurations.

```
config/
├── environments/            # Per-environment configs
│   ├── development.yaml
│   ├── staging.yaml
│   └── production.yaml
└── feature-flags/           # Feature toggles
```

## 🔌 Port Assignments

| Service         | Port | Description           |
| --------------- | ---- | --------------------- |
| API Gateway     | 8000 | Main API entry point  |
| Auth Service    | 8001 | Authentication        |
| Data Ingestion  | 8002 | Sensor data streaming |
| Model Serving   | 8003 | ML predictions        |
| Analytics       | 8004 | Data processing       |
| Notifications   | 8005 | Alerts                |
| Sensor Adapters | 8006 | Sensor integration    |
| Export Service  | 8007 | Data exports          |
| Grafana         | 3000 | Monitoring dashboards |
| MLflow          | 5000 | ML tracking           |
| PostgreSQL      | 5432 | Relational database   |
| Redis           | 6379 | Cache                 |
| InfluxDB        | 8086 | Time-series DB        |
| Kafka           | 9092 | Message queue         |
| Prometheus      | 9090 | Metrics               |

## 🚀 Getting Started

1. **Setup**: `make setup`
2. **Start**: `make start`
3. **Test**: `make test`

See [QUICK_START.md](QUICK_START.md) for detailed instructions.

## 📝 Development Guidelines

### Adding a New Service

1. Create directory in `/services`
2. Follow service template structure
3. Add to `docker-compose.yml`
4. Update API gateway routing
5. Add tests in `/tests`
6. Document in service README

### Adding a New ML Model

1. Train and validate model
2. Register in MLflow
3. Export to `/ml/models`
4. Update model serving service
5. Deploy via admin API
6. Monitor performance

### Adding a Sensor Adapter

1. Copy `/data-pipeline/sensor-adapters/template`
2. Implement required interfaces
3. Add configuration
4. Test data pipeline
5. Deploy and monitor

## 🔐 Security Considerations

- All secrets in external secret management (never in code)
- JWT authentication with refresh tokens
- Role-based access control (Public, Authenticated, Admin)
- HTTPS in production
- Rate limiting on all endpoints
- Regular security audits

## 📊 Monitoring & Observability

- **Metrics**: Prometheus + Grafana
- **Logs**: ELK Stack
- **Tracing**: Jaeger
- **Alerts**: Alertmanager
- **Health Checks**: `/health` and `/ready` endpoints

## 🏗️ Architecture Principles

1. **Microservices**: Independently deployable services
2. **Event-Driven**: Kafka for async communication
3. **Scalability**: Horizontal scaling via Kubernetes
4. **Observability**: Comprehensive monitoring
5. **Security**: Defense in depth
6. **Documentation**: Code as documentation

## 📚 Additional Resources

- [README.md](README.md) - Main documentation
- [QUICK_START.md](QUICK_START.md) - Quick start guide
- `/docs/architecture/` - Architecture documentation
- `/docs/api/` - API documentation
- Individual service READMEs in each `/services` directory

---

**Last Updated**: January 2026  
**Maintained By**: AQMRG Development Team
