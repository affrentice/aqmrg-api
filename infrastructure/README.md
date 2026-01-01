# Infrastructure

Infrastructure as Code (IaC) and deployment configurations for the AQMRG AI Analytics Platform.

## Directory Structure

### kubernetes/

Kubernetes manifests for container orchestration

#### namespaces/

Environment isolation:

- `development.yaml` - Dev environment
- `staging.yaml` - Staging environment
- `production.yaml` - Production environment

#### deployments/

Service deployment configurations:

- Replica counts
- Resource limits
- Health checks
- Environment variables

#### services/

Kubernetes service definitions:

- ClusterIP for internal services
- LoadBalancer for external access
- Service discovery

#### ingress/

API Gateway ingress rules:

- Route configurations
- SSL/TLS termination
- Path-based routing

#### configmaps/

Non-sensitive configuration:

- Application settings
- Feature flags
- Environment-specific configs

#### secrets/

Secret templates (actual secrets use external management):

- Database credentials
- API keys
- JWT secrets

### terraform/

Cloud infrastructure provisioning

#### aws/

AWS infrastructure:

- EKS cluster
- RDS databases
- ElastiCache (Redis)
- S3 buckets
- IAM roles and policies

#### azure/

Azure infrastructure:

- AKS cluster
- Azure Database
- Azure Cache for Redis
- Blob Storage
- Azure AD integration

#### gcp/

GCP infrastructure:

- GKE cluster
- Cloud SQL
- Memorystore
- Cloud Storage
- IAM configuration

#### modules/

Reusable Terraform modules:

- VPC/networking
- Database clusters
- Monitoring stack
- Security groups

### docker/

Docker configurations

#### base-images/

Common base images:

- Node.js base
- Python base
- Alpine Linux variants

### helm/

Helm charts for complex deployments:

- Chart templates
- Values files per environment
- Chart dependencies

### monitoring/

Observability stack configurations

#### prometheus/

Metrics collection:

- Scrape configurations
- Recording rules
- Alert rules

#### grafana/

Dashboard definitions:

- Service dashboards
- Infrastructure dashboards
- ML model performance

#### alertmanager/

Alert routing and notifications:

- Alert routing rules
- Notification channels
- Silencing rules

## Deployment Workflows

### Local Development (Docker Compose)

```bash
docker-compose up -d
```

### Kubernetes Development

```bash
# Create namespace
kubectl apply -f kubernetes/namespaces/development.yaml

# Deploy services
kubectl apply -f kubernetes/deployments/
kubectl apply -f kubernetes/services/
kubectl apply -f kubernetes/ingress/

# Check deployment status
kubectl get pods -n development
```

### Production Deployment with Helm

```bash
# Install/upgrade release
helm upgrade --install aqmrg-backend ./helm/aqmrg-backend \
  --namespace production \
  --values helm/aqmrg-backend/values-production.yaml

# Check release status
helm status aqmrg-backend -n production
```

## Infrastructure Provisioning

### Terraform Workflow

1. **Initialize Terraform**

```bash
cd terraform/aws  # or azure, gcp
terraform init
```

2. **Plan Changes**

```bash
terraform plan -var-file=environments/production.tfvars
```

3. **Apply Changes**

```bash
terraform apply -var-file=environments/production.tfvars
```

4. **Destroy Resources (if needed)**

```bash
terraform destroy -var-file=environments/production.tfvars
```

### State Management

- State stored in remote backend (S3, Azure Blob, GCS)
- State locking enabled
- Separate state files per environment

## Kubernetes Configurations

### Resource Limits (Production)

```yaml
resources:
  requests:
    cpu: 500m
    memory: 512Mi
  limits:
    cpu: 1000m
    memory: 1Gi
```

### Horizontal Pod Autoscaling

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: api-gateway
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: api-gateway
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
```

### Health Checks

```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 8000
  initialDelaySeconds: 30
  periodSeconds: 10

readinessProbe:
  httpGet:
    path: /ready
    port: 8000
  initialDelaySeconds: 5
  periodSeconds: 5
```

## Monitoring Setup

### Prometheus Installation

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install prometheus prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --create-namespace \
  -f monitoring/prometheus/values.yaml
```

### Grafana Access

```bash
# Get admin password
kubectl get secret -n monitoring prometheus-grafana \
  -o jsonpath="{.data.admin-password}" | base64 --decode

# Port forward
kubectl port-forward -n monitoring svc/prometheus-grafana 3000:80
```

Access: http://localhost:3000

## Security Best Practices

### Secret Management

- Use Kubernetes Secrets or external secret managers (AWS Secrets Manager, Azure Key Vault)
- Never commit secrets to version control
- Rotate secrets regularly
- Use RBAC to restrict secret access

### Network Policies

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: api-gateway-policy
spec:
  podSelector:
    matchLabels:
      app: api-gateway
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: frontend
      ports:
        - protocol: TCP
          port: 8000
```

### Pod Security Standards

- Use non-root containers
- Read-only root filesystem
- Drop unnecessary capabilities
- Use security contexts

## CI/CD Integration

### GitHub Actions Example

```yaml
name: Deploy to Production
on:
  push:
    branches: [main]
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Configure kubectl
        run: |
          echo "${{ secrets.KUBECONFIG }}" > kubeconfig
          export KUBECONFIG=./kubeconfig
      - name: Deploy
        run: kubectl apply -f kubernetes/deployments/
```

## Database Infrastructure

### PostgreSQL (RDS/Cloud SQL)

- Multi-AZ deployment
- Automated backups
- Read replicas for scaling
- Connection pooling (PgBouncer)

### InfluxDB

- Clustered deployment
- Retention policies configured
- Continuous queries for downsampling

### Redis

- Redis Cluster or Sentinel
- Persistence enabled
- Memory limits configured

## Backup and Disaster Recovery

### Backup Strategy

- Database: Daily automated backups, 30-day retention
- Application state: Kubernetes etcd backups
- Configuration: Version controlled in Git

### Disaster Recovery

- RTO: 2 hours
- RPO: 15 minutes
- Multi-region deployment for critical services
- Regular disaster recovery drills

## Cost Optimization

### Resource Optimization

- Right-size instances based on actual usage
- Use spot/preemptible instances for non-critical workloads
- Configure autoscaling to scale down during off-peak

### Monitoring Costs

- Set up billing alerts
- Use cost allocation tags
- Review and optimize monthly

## Troubleshooting

### Pod Not Starting

```bash
# Check pod status
kubectl describe pod <pod-name> -n <namespace>

# View logs
kubectl logs <pod-name> -n <namespace>

# Check events
kubectl get events -n <namespace> --sort-by='.lastTimestamp'
```

### Service Not Accessible

```bash
# Check service
kubectl get svc -n <namespace>

# Check endpoints
kubectl get endpoints -n <namespace>

# Test connectivity
kubectl run -it --rm debug --image=busybox --restart=Never -- wget -O- http://service-name:port
```

### High Resource Usage

```bash
# Check resource usage
kubectl top pods -n <namespace>
kubectl top nodes

# Review metrics in Grafana
# Check autoscaling events
kubectl describe hpa -n <namespace>
```

## Environment Variables

### Required for Terraform

```bash
# AWS
export AWS_ACCESS_KEY_ID=xxx
export AWS_SECRET_ACCESS_KEY=xxx
export AWS_REGION=us-east-1

# Azure
export ARM_CLIENT_ID=xxx
export ARM_CLIENT_SECRET=xxx
export ARM_SUBSCRIPTION_ID=xxx
export ARM_TENANT_ID=xxx

# GCP
export GOOGLE_APPLICATION_CREDENTIALS=/path/to/key.json
export GOOGLE_PROJECT=project-id
```

## Dependencies

```
kubectl >= 1.28
helm >= 3.12
terraform >= 1.6
docker >= 24.0
```
