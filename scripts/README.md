# Scripts

Automation and utility scripts for development, deployment, and operations.

## Directory Structure

### deployment/

Deployment automation scripts

```
deployment/
├── deploy.sh                # Main deployment script
├── rollback.sh             # Rollback deployment
├── health_check.sh         # Post-deployment health checks
├── blue_green_deploy.sh    # Blue-green deployment
└── canary_deploy.sh        # Canary deployment
```

### database/

Database management utilities

```
database/
├── backup.sh               # Backup all databases
├── restore.sh              # Restore from backup
├── migrate.sh              # Run migrations
├── seed.sh                 # Seed development data
└── optimize.sh             # Database optimization
```

### monitoring/

Health check and diagnostic scripts

```
monitoring/
├── health_check.sh         # Check all services
├── performance_test.sh     # Run performance tests
├── check_kafka.sh          # Kafka health check
└── check_databases.sh      # Database connectivity check
```

### data-migration/

Data migration utilities

```
data-migration/
├── migrate_sensors.py      # Migrate sensor data
├── migrate_users.py        # Migrate user data
└── bulk_import.py          # Bulk data import
```

### seed-data/

Development data seeding scripts

```
seed-data/
├── seed.py                 # Main seeding script
├── sensors.json            # Sample sensor data
├── users.json              # Sample user data
└── readings.json           # Sample sensor readings
```

## Script Usage

### Deployment Scripts

#### deploy.sh

Deploy services to Kubernetes cluster

```bash
#!/bin/bash
# Usage: ./deploy.sh <environment> <version>

ENVIRONMENT=$1
VERSION=$2

if [ -z "$ENVIRONMENT" ] || [ -z "$VERSION" ]; then
  echo "Usage: ./deploy.sh <environment> <version>"
  echo "Example: ./deploy.sh staging v1.2.3"
  exit 1
fi

echo "Deploying version $VERSION to $ENVIRONMENT..."

# Build and push Docker images
docker-compose build
docker tag aqmrg-api-gateway:latest aqmrg-api-gateway:$VERSION
docker push aqmrg-api-gateway:$VERSION

# Update Kubernetes deployments
kubectl set image deployment/api-gateway \
  api-gateway=aqmrg-api-gateway:$VERSION \
  -n $ENVIRONMENT

# Wait for rollout
kubectl rollout status deployment/api-gateway -n $ENVIRONMENT

# Health check
./scripts/monitoring/health_check.sh $ENVIRONMENT

echo "Deployment complete!"
```

**Usage:**

```bash
./scripts/deployment/deploy.sh staging v1.2.3
./scripts/deployment/deploy.sh production v1.2.3
```

#### rollback.sh

Rollback to previous deployment

```bash
#!/bin/bash
# Usage: ./rollback.sh <environment> <service>

ENVIRONMENT=$1
SERVICE=$2

if [ -z "$ENVIRONMENT" ] || [ -z "$SERVICE" ]; then
  echo "Usage: ./rollback.sh <environment> <service>"
  exit 1
fi

echo "Rolling back $SERVICE in $ENVIRONMENT..."
kubectl rollout undo deployment/$SERVICE -n $ENVIRONMENT
kubectl rollout status deployment/$SERVICE -n $ENVIRONMENT

echo "Rollback complete!"
```

### Database Scripts

#### backup.sh

Backup all databases

```bash
#!/bin/bash
# Usage: ./backup.sh

BACKUP_DIR="backups/$(date +%Y%m%d_%H%M%S)"
mkdir -p $BACKUP_DIR

echo "Backing up databases to $BACKUP_DIR..."

# PostgreSQL backup
echo "Backing up PostgreSQL..."
docker-compose exec -T postgres pg_dump -U aqmrg -d aqmrg -F c \
  > $BACKUP_DIR/postgres.dump

# InfluxDB backup
echo "Backing up InfluxDB..."
docker-compose exec -T influxdb influx backup /tmp/backup
docker cp $(docker-compose ps -q influxdb):/tmp/backup $BACKUP_DIR/influxdb

# Redis backup
echo "Backing up Redis..."
docker-compose exec -T redis redis-cli BGSAVE
sleep 5
docker cp $(docker-compose ps -q redis):/data/dump.rdb $BACKUP_DIR/redis.rdb

echo "Backup complete: $BACKUP_DIR"
```

**Usage:**

```bash
./scripts/database/backup.sh
```

#### migrate.sh

Run database migrations

```bash
#!/bin/bash
# Usage: ./migrate.sh <direction>

DIRECTION=${1:-up}

echo "Running database migrations ($DIRECTION)..."

# PostgreSQL migrations
echo "Migrating PostgreSQL..."
if [ "$DIRECTION" = "up" ]; then
  docker-compose exec postgres psql -U aqmrg -d aqmrg \
    -f /docker-entrypoint-initdb.d/migrations/
else
  echo "Rollback not implemented"
  exit 1
fi

# Run Python/Alembic migrations if using
# alembic upgrade head

echo "Migrations complete!"
```

**Usage:**

```bash
./scripts/database/migrate.sh up
./scripts/database/migrate.sh down
```

### Monitoring Scripts

#### health_check.sh

Check health of all services

```bash
#!/bin/bash
# Usage: ./health_check.sh [environment]

ENVIRONMENT=${1:-development}
API_URL="http://localhost:8000"

if [ "$ENVIRONMENT" = "production" ]; then
  API_URL="https://api.aqmrg.org"
elif [ "$ENVIRONMENT" = "staging" ]; then
  API_URL="https://staging-api.aqmrg.org"
fi

echo "Checking health of services in $ENVIRONMENT..."

# Check API Gateway
echo -n "API Gateway: "
if curl -sf $API_URL/health > /dev/null; then
  echo "✓ Healthy"
else
  echo "✗ Unhealthy"
  exit 1
fi

# Check Auth Service
echo -n "Auth Service: "
if curl -sf $API_URL/auth/health > /dev/null; then
  echo "✓ Healthy"
else
  echo "✗ Unhealthy"
fi

# Check Model Serving
echo -n "Model Serving: "
if curl -sf $API_URL/predictions/health > /dev/null; then
  echo "✓ Healthy"
else
  echo "✗ Unhealthy"
fi

# Check databases
./scripts/monitoring/check_databases.sh

echo "Health check complete!"
```

**Usage:**

```bash
./scripts/monitoring/health_check.sh
./scripts/monitoring/health_check.sh production
```

#### check_kafka.sh

Check Kafka cluster health

```bash
#!/bin/bash
# Usage: ./check_kafka.sh

KAFKA_BROKER="localhost:9092"

echo "Checking Kafka cluster..."

# Check broker connectivity
echo -n "Broker connectivity: "
if docker-compose exec kafka kafka-broker-api-versions.sh \
  --bootstrap-server $KAFKA_BROKER > /dev/null 2>&1; then
  echo "✓ Connected"
else
  echo "✗ Failed"
  exit 1
fi

# List topics
echo "Topics:"
docker-compose exec kafka kafka-topics.sh \
  --list \
  --bootstrap-server $KAFKA_BROKER

# Check consumer groups
echo "Consumer groups:"
docker-compose exec kafka kafka-consumer-groups.sh \
  --list \
  --bootstrap-server $KAFKA_BROKER

echo "Kafka check complete!"
```

### Data Migration Scripts

#### migrate_sensors.py

Migrate sensor data between systems

```python
#!/usr/bin/env python3
"""
Migrate sensor data from old system to new system
Usage: python migrate_sensors.py --source old_db --target new_db
"""

import argparse
import psycopg2
from datetime import datetime

def migrate_sensors(source_conn, target_conn):
    """Migrate sensor data"""
    source_cur = source_conn.cursor()
    target_cur = target_conn.cursor()

    # Fetch sensors from old system
    source_cur.execute("SELECT * FROM old_sensors")
    sensors = source_cur.fetchall()

    print(f"Migrating {len(sensors)} sensors...")

    for sensor in sensors:
        # Transform and insert into new system
        target_cur.execute("""
            INSERT INTO sensors (id, name, manufacturer, location_id, status)
            VALUES (%s, %s, %s, %s, %s)
            ON CONFLICT (id) DO NOTHING
        """, (sensor[0], sensor[1], sensor[2], sensor[3], 'active'))

    target_conn.commit()
    print("Migration complete!")

if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument('--source', required=True)
    parser.add_argument('--target', required=True)
    args = parser.parse_args()

    # Connect and migrate
    # ... connection logic ...
```

**Usage:**

```bash
python scripts/data-migration/migrate_sensors.py \
  --source postgresql://old_db \
  --target postgresql://new_db
```

### Seed Data Scripts

#### seed.py

Seed development database with sample data

```python
#!/usr/bin/env python3
"""
Seed development database with sample data
Usage: python seed.py
"""

import json
import psycopg2
from influxdb_client import InfluxDBClient
import os
from datetime import datetime, timedelta

def seed_users(conn):
    """Seed user data"""
    print("Seeding users...")
    with open('scripts/seed-data/users.json') as f:
        users = json.load(f)

    cur = conn.cursor()
    for user in users:
        cur.execute("""
            INSERT INTO users (email, password_hash, role)
            VALUES (%s, %s, %s)
            ON CONFLICT (email) DO NOTHING
        """, (user['email'], user['password_hash'], user['role']))
    conn.commit()
    print(f"Seeded {len(users)} users")

def seed_sensors(conn):
    """Seed sensor data"""
    print("Seeding sensors...")
    with open('scripts/seed-data/sensors.json') as f:
        sensors = json.load(f)

    cur = conn.cursor()
    for sensor in sensors:
        cur.execute("""
            INSERT INTO sensors (id, name, manufacturer, location_id, status)
            VALUES (%s, %s, %s, %s, %s)
            ON CONFLICT (id) DO NOTHING
        """, (sensor['id'], sensor['name'], sensor['manufacturer'],
              sensor['location_id'], sensor['status']))
    conn.commit()
    print(f"Seeded {len(sensors)} sensors")

def seed_readings(influx_client):
    """Seed sensor readings"""
    print("Seeding sensor readings...")
    with open('scripts/seed-data/readings.json') as f:
        readings = json.load(f)

    write_api = influx_client.write_api()
    for reading in readings:
        point = {
            "measurement": "air_quality",
            "tags": {
                "sensor_id": reading['sensor_id'],
                "location": reading['location']
            },
            "fields": {
                "pm25": reading['pm25'],
                "pm10": reading['pm10']
            },
            "time": reading['timestamp']
        }
        write_api.write(bucket="sensor_data", record=point)

    print(f"Seeded {len(readings)} readings")

if __name__ == "__main__":
    # PostgreSQL connection
    pg_conn = psycopg2.connect(
        host=os.getenv('DATABASE_HOST', 'localhost'),
        port=os.getenv('DATABASE_PORT', 5432),
        database=os.getenv('DATABASE_NAME', 'aqmrg'),
        user=os.getenv('DATABASE_USER', 'aqmrg'),
        password=os.getenv('DATABASE_PASSWORD', 'aqmrg_dev_password')
    )

    # InfluxDB connection
    influx_client = InfluxDBClient(
        url=os.getenv('INFLUXDB_URL', 'http://localhost:8086'),
        token=os.getenv('INFLUXDB_TOKEN'),
        org=os.getenv('INFLUXDB_ORG', 'aqmrg')
    )

    # Seed data
    seed_users(pg_conn)
    seed_sensors(pg_conn)
    seed_readings(influx_client)

    print("Seeding complete!")
```

**Usage:**

```bash
python scripts/seed-data/seed.py
```

## Common Tasks

### Initial Setup

```bash
# Make scripts executable
chmod +x scripts/**/*.sh

# Setup environment
cp .env.example .env
```

### Daily Development

```bash
# Start services
docker-compose up -d

# Seed development data
python scripts/seed-data/seed.py

# Check health
./scripts/monitoring/health_check.sh
```

### Deployment

```bash
# Deploy to staging
./scripts/deployment/deploy.sh staging v1.2.3

# Health check
./scripts/monitoring/health_check.sh staging

# Deploy to production
./scripts/deployment/deploy.sh production v1.2.3
```

### Backup & Recovery

```bash
# Backup databases
./scripts/database/backup.sh

# Restore from backup
./scripts/database/restore.sh backups/20260101_120000
```

## Best Practices

1. **Make scripts executable**: `chmod +x script.sh`
2. **Add error handling**: Use `set -e` for bash scripts
3. **Document usage**: Include help text in scripts
4. **Use environment variables**: Don't hardcode credentials
5. **Log operations**: Add logging for debugging
6. **Test scripts**: Test in development before production
7. **Version control**: Track all scripts in Git

## Dependencies

```bash
# Bash scripts
curl
jq
kubectl
docker
docker-compose

# Python scripts
psycopg2-binary
influxdb-client
redis
```
