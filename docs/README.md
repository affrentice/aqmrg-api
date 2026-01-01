# Documentation

Comprehensive documentation for the AQMRG AI Analytics Backend Platform.

## Directory Structure

### api/

API documentation and usage guides

```
api/
├── postman/                    # Postman collections
│   ├── AQMRG-API.postman_collection.json
│   └── AQMRG-Environments.postman_environment.json
├── examples/                   # API usage examples
│   ├── authentication.md
│   ├── real-time-data.md
│   ├── predictions.md
│   └── data-export.md
└── README.md                   # API overview
```

### architecture/

System architecture and design documentation

```
architecture/
├── diagrams/                   # Architecture diagrams
│   ├── system-overview.png
│   ├── microservices-architecture.png
│   ├── data-flow.png
│   └── deployment-architecture.png
├── decisions/                  # Architecture Decision Records (ADRs)
│   ├── 001-microservices-architecture.md
│   ├── 002-database-selection.md
│   ├── 003-kafka-for-streaming.md
│   └── 004-kubernetes-deployment.md
└── README.md                   # Architecture overview
```

### deployment/

Deployment guides and procedures

```
deployment/
├── local-development.md        # Local setup guide
├── staging-deployment.md       # Staging deployment
├── production-deployment.md    # Production deployment
├── kubernetes-setup.md         # Kubernetes cluster setup
├── monitoring-setup.md         # Monitoring infrastructure
└── README.md                   # Deployment overview
```

### integration/

Third-party integration guides

```
integration/
├── sensor-manufacturers.md     # Adding sensor integrations
├── external-apis.md            # External API integrations
├── frontend-integration.md     # Frontend integration guide
└── webhook-setup.md            # Webhook configuration
```

### runbooks/

Operational runbooks for common scenarios

```
runbooks/
├── incident-response.md        # Incident response procedures
├── scaling-services.md         # How to scale services
├── database-maintenance.md     # Database maintenance tasks
├── backup-restore.md           # Backup and restore procedures
├── security-incident.md        # Security incident response
└── disaster-recovery.md        # Disaster recovery plan
```

## Key Documentation Files

### API Documentation Overview

**File**: `api/README.md`

````markdown
# API Documentation

## Overview

The AQMRG AI Analytics API provides access to real-time air quality data, predictions, and analytics.

## Base URLs

- **Production**: `https://api.aqmrg.org/v1`
- **Staging**: `https://staging-api.aqmrg.org/v1`
- **Development**: `http://localhost:8000/v1`

## Authentication

### Public Endpoints

No authentication required:

- `GET /dashboard/realtime`
- `GET /predictions/forecast`
- `GET /health`

### Authenticated Endpoints

Require Bearer token:

- `GET /analytics/historical`
- `POST /data/export`
- `POST /alerts`

### Admin Endpoints

Require admin role:

- `POST /admin/models/deploy`
- `POST /admin/sensors/configure`

## Authentication Flow

1. **Login**

```bash
curl -X POST https://api.aqmrg.org/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"user@example.com","password":"password123"}'
```
````

Response:

```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIs...",
  "refresh_token": "eyJhbGciOiJIUzI1NiIs...",
  "expires_in": 3600
}
```

2. **Use Token**

```bash
curl -X GET https://api.aqmrg.org/v1/analytics/historical \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIs..."
```

## Rate Limiting

- **Public**: 100 requests per 15 minutes
- **Authenticated**: 1,000 requests per 15 minutes
- **Admin**: Unlimited

## Error Responses

All errors follow this format:

```json
{
  "error": {
    "code": "INVALID_REQUEST",
    "message": "The location parameter is required",
    "details": {}
  }
}
```

Common error codes:

- `INVALID_REQUEST` - Bad request parameters
- `UNAUTHORIZED` - Authentication required
- `FORBIDDEN` - Insufficient permissions
- `NOT_FOUND` - Resource not found
- `RATE_LIMIT_EXCEEDED` - Too many requests

## Postman Collection

Import the Postman collection for easy testing:

1. Download `postman/AQMRG-API.postman_collection.json`
2. Import into Postman
3. Set environment variables
4. Start making requests

## Code Examples

See `examples/` directory for detailed code examples in:

- JavaScript/Node.js
- Python
- cURL

````

### Architecture Decision Record Template

**File**: `architecture/decisions/000-template.md`

```markdown
# [Number]. [Title]

Date: YYYY-MM-DD

## Status

[Proposed | Accepted | Deprecated | Superseded]

## Context

What is the issue that we're seeing that is motivating this decision or change?

## Decision

What is the change that we're proposing and/or doing?

## Consequences

What becomes easier or more difficult to do because of this change?

### Positive Consequences

- List positive impacts
- Another positive impact

### Negative Consequences

- List negative impacts
- Another negative impact

## Alternatives Considered

What other options were considered?

### Alternative 1: [Name]

Description, pros, and cons

### Alternative 2: [Name]

Description, pros, and cons

## References

- Links to related documents
- Research papers
- Tool documentation
````

### Example Architecture Decision Record

**File**: `architecture/decisions/001-microservices-architecture.md`

```markdown
# 1. Adopt Microservices Architecture

Date: 2026-01-01

## Status

Accepted

## Context

The AQMRG platform needs to handle multiple distinct responsibilities:

- Real-time sensor data ingestion
- ML model serving for predictions
- User authentication and authorization
- Data analytics and reporting
- Alert notifications

A monolithic architecture would couple these concerns and make independent scaling difficult.

## Decision

We will adopt a microservices architecture with the following services:

- API Gateway (request routing)
- Auth Service (authentication)
- Data Ingestion Service (sensor data)
- Model Serving Service (ML predictions)
- Analytics Service (data processing)
- Notification Service (alerts)
- Sensor Adapter Service (integrations)
- Export Service (data exports)

## Consequences

### Positive Consequences

- **Independent Scaling**: Each service can scale based on its specific load
- **Technology Flexibility**: Services can use different tech stacks
- **Fault Isolation**: Failure in one service doesn't crash the entire system
- **Team Autonomy**: Teams can work independently on different services
- **Deployment Independence**: Services can be deployed separately

### Negative Consequences

- **Increased Complexity**: More moving parts to manage
- **Network Overhead**: Inter-service communication adds latency
- **Data Consistency**: Distributed data management is more complex
- **Testing Complexity**: Integration testing becomes more challenging
- **Operational Overhead**: More services to monitor and maintain

## Alternatives Considered

### Alternative 1: Monolithic Architecture

**Pros:**

- Simpler to develop initially
- Easier to test
- Single deployment

**Cons:**

- Difficult to scale specific components
- Technology lock-in
- Harder to maintain as codebase grows

### Alternative 2: Modular Monolith

**Pros:**

- Organized code structure
- Easier than pure monolith
- Single deployment

**Cons:**

- Still coupled at deployment
- Cannot scale components independently
- Harder to migrate later

## References

- [Microservices Pattern](https://microservices.io/)
- [Building Microservices by Sam Newman](https://samnewman.io/books/building_microservices/)
```

### Deployment Guide Example

**File**: `deployment/production-deployment.md`

````markdown
# Production Deployment Guide

## Prerequisites

- Kubernetes cluster (v1.28+)
- kubectl configured
- Docker images built and pushed to registry
- Production environment variables configured
- SSL certificates ready

## Pre-Deployment Checklist

- [ ] All tests passing in CI/CD
- [ ] Code reviewed and approved
- [ ] Database migrations prepared
- [ ] Backup completed
- [ ] Rollback plan documented
- [ ] Stakeholders notified
- [ ] Monitoring alerts configured

## Deployment Steps

### 1. Database Migrations

```bash
# Run migrations
kubectl exec -it postgres-pod -n production -- \
  psql -U aqmrg -d aqmrg -f /migrations/v1.2.3.sql

# Verify migration
kubectl exec -it postgres-pod -n production -- \
  psql -U aqmrg -d aqmrg -c "SELECT version FROM schema_migrations;"
```
````

### 2. Deploy Services

```bash
# Update image tags in deployments
export VERSION=v1.2.3

# Apply Kubernetes manifests
kubectl apply -f infrastructure/kubernetes/namespaces/production.yaml
kubectl apply -f infrastructure/kubernetes/deployments/ -n production
kubectl apply -f infrastructure/kubernetes/services/ -n production
kubectl apply -f infrastructure/kubernetes/ingress/ -n production
```

### 3. Monitor Rollout

```bash
# Watch deployment status
kubectl rollout status deployment/api-gateway -n production
kubectl rollout status deployment/auth-service -n production
kubectl rollout status deployment/model-serving-service -n production

# Check pods
kubectl get pods -n production
```

### 4. Health Checks

```bash
# Run health check script
./scripts/monitoring/health_check.sh production

# Manual health checks
curl https://api.aqmrg.org/health
curl https://api.aqmrg.org/v1/dashboard/realtime?location=Lagos
```

### 5. Smoke Tests

```bash
# Run smoke tests
npm run test:smoke -- --env=production

# Or manually test critical paths
# - User login
# - Dashboard load
# - Prediction generation
# - Data export
```

## Post-Deployment

### Verify Monitoring

1. Check Grafana dashboards
2. Verify Prometheus metrics
3. Check application logs
4. Review error rates

### Performance Verification

```bash
# Check response times
curl -w "@curl-format.txt" -o /dev/null -s https://api.aqmrg.org/health

# Run load test
k6 run --vus 100 --duration 5m tests/load/production_smoke_test.js
```

## Rollback Procedure

If issues are detected:

```bash
# Rollback deployment
kubectl rollout undo deployment/api-gateway -n production

# Verify rollback
kubectl rollout status deployment/api-gateway -n production

# Check health
./scripts/monitoring/health_check.sh production
```

## Troubleshooting

### Pods Not Starting

```bash
# Check pod status
kubectl describe pod <pod-name> -n production

# Check logs
kubectl logs <pod-name> -n production

# Check events
kubectl get events -n production --sort-by='.lastTimestamp'
```

### Service Not Accessible

```bash
# Check service
kubectl get svc -n production

# Check ingress
kubectl get ingress -n production

# Test internal connectivity
kubectl run -it --rm debug --image=busybox --restart=Never -n production -- \
  wget -O- http://api-gateway:8000/health
```

## Contact

- **On-Call Engineer**: [Phone/Slack]
- **Team Lead**: [Contact]
- **DevOps Lead**: [Contact]

````

### Runbook Example

**File**: `runbooks/incident-response.md`

```markdown
# Incident Response Runbook

## Severity Levels

- **P0 (Critical)**: Complete service outage
- **P1 (High)**: Major feature unavailable
- **P2 (Medium)**: Minor feature degraded
- **P3 (Low)**: Cosmetic issue

## P0: Complete Service Outage

### Immediate Actions (First 5 minutes)

1. **Acknowledge Alert**
   - Acknowledge in PagerDuty/monitoring system
   - Post in #incidents Slack channel

2. **Assess Impact**
   ```bash
   # Check all services
   kubectl get pods -n production

   # Check health endpoints
   ./scripts/monitoring/health_check.sh production
````

3. **Check Recent Changes**
   - Review recent deployments
   - Check recent code merges
   - Review infrastructure changes

### Investigation (5-15 minutes)

1. **Check Logs**

   ```bash
   # API Gateway logs
   kubectl logs -f deployment/api-gateway -n production --tail=100

   # Check for errors
   kubectl logs deployment/api-gateway -n production | grep ERROR
   ```

2. **Check Metrics**

   - Open Grafana dashboard
   - Check error rates
   - Review response times
   - Check resource utilization

3. **Check Dependencies**

   ```bash
   # Check database
   ./scripts/monitoring/check_databases.sh

   # Check Kafka
   ./scripts/monitoring/check_kafka.sh
   ```

### Resolution

#### If Caused by Recent Deployment

```bash
# Rollback
kubectl rollout undo deployment/<service> -n production
kubectl rollout status deployment/<service> -n production
```

#### If Database Issue

```bash
# Check connections
kubectl exec -it postgres-pod -n production -- \
  psql -U aqmrg -d aqmrg -c "SELECT count(*) FROM pg_stat_activity;"

# Restart if needed
kubectl rollout restart deployment/postgres -n production
```

#### If Infrastructure Issue

```bash
# Check node status
kubectl get nodes

# Check pod distribution
kubectl get pods -n production -o wide

# Drain problematic node
kubectl drain <node-name> --ignore-daemonsets
```

### Communication

1. **Update Status Page**

   - Post incident notice
   - Update every 15 minutes

2. **Notify Stakeholders**

   - Email to stakeholders list
   - Update in #incidents channel

3. **Post-Incident**
   - Schedule post-mortem
   - Document timeline
   - Identify action items

## Common Issues Quick Reference

### High Memory Usage

```bash
# Check memory usage
kubectl top pods -n production

# Restart high-memory pod
kubectl delete pod <pod-name> -n production
```

### High Database Connections

```bash
# Check connections
kubectl exec -it postgres-pod -n production -- \
  psql -U aqmrg -d aqmrg -c "SELECT count(*) FROM pg_stat_activity;"

# Kill idle connections
kubectl exec -it postgres-pod -n production -- \
  psql -U aqmrg -d aqmrg -c "SELECT pg_terminate_backend(pid) FROM pg_stat_activity WHERE state = 'idle';"
```

### Kafka Consumer Lag

```bash
# Check consumer lag
kubectl exec -it kafka-pod -n production -- \
  kafka-consumer-groups.sh --bootstrap-server localhost:9092 \
  --describe --group sensor-consumers

# Reset offsets if needed
kubectl exec -it kafka-pod -n production -- \
  kafka-consumer-groups.sh --bootstrap-server localhost:9092 \
  --group sensor-consumers --reset-offsets --to-latest --execute --topic sensor.raw.airquality
```

## Escalation

If unable to resolve within 30 minutes:

1. Escalate to Senior Engineer
2. Page DevOps Lead
3. Consider emergency maintenance window

```

## Documentation Best Practices

1. **Keep It Current**: Update docs with code changes
2. **Use Examples**: Include real-world examples
3. **Be Specific**: Avoid vague descriptions
4. **Link Related Docs**: Cross-reference related documentation
5. **Include Diagrams**: Use visuals when helpful
6. **Version Control**: Track doc changes in Git
7. **Review Regularly**: Quarterly documentation review

## Contributing to Documentation

1. Follow the template for each doc type
2. Use clear, concise language
3. Include code examples where applicable
4. Add diagrams for complex concepts
5. Update table of contents
6. Get peer review before merging

## Tools

- **Diagrams**: draw.io, Lucidchart, PlantUML
- **API Docs**: Swagger UI, ReDoc
- **Markdown**: VSCode, Typora
- **Screenshots**: Snagit, CloudApp
```
