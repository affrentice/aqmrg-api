# AQMRG AI Backend - Quick Start Guide

Get the AQMRG AI Analytics Backend up and running in minutes.

## Prerequisites

- Docker & Docker Compose installed
- Git installed
- Node.js 18+ OR Python 3.10+ (depending on service language choice)
- kubectl (for Kubernetes deployment)
- 8GB+ RAM available

## Step 1: Clone and Setup

```bash
# Clone the repository
git clone https://github.com/affrentice/aqmrg-api.git
cd aqmrg-api

# Copy environment template
cp .env.example .env

# Edit .env with your configuration
nano .env
```

## Step 2: Start Infrastructure Services

```bash
# Start databases, Kafka, MLflow, and monitoring
docker-compose up -d

# Verify services are running
docker-compose ps
```

You should see:

- PostgreSQL (port 5432)
- InfluxDB (port 8086)
- Redis (port 6379)
- Kafka + Zookeeper (port 9092)
- MLflow (port 5000)
- Prometheus (port 9090)
- Grafana (port 3000)

## Step 3: Run Database Migrations

```bash
# Run PostgreSQL migrations
make migrate
```

## Step 4: Seed Development Data (Optional)

```bash
# Seed with sample data for testing
make seed
```

## Step 5: Develop Your First Service

### Option A: Node.js Service (TypeScript)

```bash
cd services/api-gateway

# Create package.json
npm init -y

# Install dependencies
npm install express cors helmet dotenv
npm install -D typescript @types/node @types/express ts-node nodemon

# Create src/index.ts
mkdir src
cat > src/index.ts << 'EOF'
import express from 'express';
import cors from 'cors';
import helmet from 'helmet';

const app = express();
const PORT = process.env.PORT || 8000;

app.use(helmet());
app.use(cors());
app.use(express.json());

app.get('/health', (req, res) => {
  res.json({ status: 'healthy', service: 'api-gateway' });
});

app.listen(PORT, () => {
  console.log(`API Gateway running on port ${PORT}`);
});
EOF

# Run in development mode
npm run dev
```

### Option B: Python Service (FastAPI)

```bash
cd services/model-serving-service

# Create virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install fastapi uvicorn python-dotenv

# Create main.py
cat > main.py << 'EOF'
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware

app = FastAPI(title="Model Serving Service")

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

@app.get("/health")
async def health_check():
    return {"status": "healthy", "service": "model-serving"}

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8003)
EOF

# Run service
python main.py
```

## Step 6: Access Services

### Monitoring Dashboards

- **Grafana**: http://localhost:3000 (admin/admin)
- **Prometheus**: http://localhost:9090
- **MLflow**: http://localhost:5000

### Databases

```bash
# PostgreSQL
docker-compose exec postgres psql -U aqmrg -d aqmrg

# InfluxDB
# Access UI at http://localhost:8086

# Redis
docker-compose exec redis redis-cli
```

### Kafka

```bash
# List topics
docker-compose exec kafka kafka-topics --list --bootstrap-server localhost:9092

# Create a topic
docker-compose exec kafka kafka-topics --create \
  --topic sensor.raw.airquality \
  --bootstrap-server localhost:9092 \
  --partitions 3 \
  --replication-factor 1
```

## Step 7: Test Your Setup

```bash
# Check if API Gateway is healthy
curl http://localhost:8000/health

# Check Prometheus targets
curl http://localhost:9090/api/v1/targets

# View Grafana dashboards
# Navigate to http://localhost:3000
```

## Common Development Tasks

### View Logs

```bash
# All services
docker-compose logs -f

# Specific service
docker-compose logs -f postgres
```

### Restart Services

```bash
# Restart all
docker-compose restart

# Restart specific service
docker-compose restart kafka
```

### Stop Everything

```bash
# Stop services but keep data
docker-compose down

# Stop and remove all data
docker-compose down -v
```

### Run Tests

```bash
make test
```

## Next Steps

1. **Build Your Services**: Start implementing microservices in `/services`
2. **Add ML Models**: Place trained models in `/ml/models`
3. **Configure Data Pipeline**: Set up Kafka producers/consumers
4. **Create API Contracts**: Define OpenAPI specs in `/api-contracts`
5. **Write Tests**: Add tests in `/tests`

## Troubleshooting

### Ports Already in Use

```bash
# Check what's using the port
lsof -i :8000  # Replace with your port

# Kill the process
kill -9 <PID>
```

### Docker Out of Space

```bash
# Clean up Docker
docker system prune -a --volumes
```

### Database Connection Issues

```bash
# Verify database is running
docker-compose ps postgres

# Check logs
docker-compose logs postgres
```

### Kafka Not Starting

```bash
# Ensure Zookeeper is healthy first
docker-compose logs zookeeper

# Then check Kafka
docker-compose logs kafka
```

## Development Workflow

1. Create feature branch: `git checkout -b feature/new-service`
2. Develop your changes
3. Run tests: `make test`
4. Commit: `git commit -m "Add new service"`
5. Push: `git push origin feature/new-service`
6. Create Pull Request

## Production Deployment

See detailed deployment guides in `/docs/deployment/`

Basic Kubernetes deployment:

```bash
# Deploy to staging
make deploy-staging

# Deploy to production (with confirmation)
make deploy-production
```

## Getting Help

- 📖 Full Documentation: `/docs`
- 🐛 Report Issues: [GitHub Issues]
- 💬 Team Chat: [Slack/Discord]
- 📧 Email: support@aqmrg.org

## Useful Commands Cheatsheet

```bash
# Start everything
make start

# Stop everything
make stop

# View logs
make logs

# Run migrations
make migrate

# Seed data
make seed

# Run tests
make test

# Clean up
make clean

# Format code
make format

# Lint code
make lint
```

Happy coding! 🚀
