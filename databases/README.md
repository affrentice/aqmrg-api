# Databases

Database schemas, migrations, and configurations for the AQMRG AI Analytics Platform.

## Overview

The platform uses three primary databases:

- **PostgreSQL**: Relational data (users, configurations, metadata)
- **InfluxDB**: Time-series sensor data (high-frequency readings)
- **Redis**: Caching and session storage

## Directory Structure

### postgres/

PostgreSQL database for relational data

#### migrations/

Database schema migrations using Alembic (Python) or Flyway (Java/Node.js)

**Migration Files:**

```
migrations/
├── V001__initial_schema.sql
├── V002__add_users_table.sql
├── V003__add_sensors_table.sql
├── V004__add_alerts_table.sql
└── V005__add_indexes.sql
```

**Running Migrations:**

```bash
# Using Alembic (Python)
alembic upgrade head

# Using Flyway
flyway migrate

# Using raw SQL
psql -U aqmrg -d aqmrg -f migrations/V001__initial_schema.sql
```

#### schemas/

SQL schema definitions and table structures

**Key Tables:**

- `users` - User accounts and authentication
- `sensors` - Sensor registry and metadata
- `locations` - Geographic locations
- `alerts` - Alert configurations
- `predictions` - Cached prediction results
- `audit_logs` - System audit trail

#### seeds/

Test and development data

**Seeding Database:**

```bash
# Seed development data
psql -U aqmrg -d aqmrg -f seeds/dev_data.sql

# Seed test data
psql -U aqmrg -d aqmrg -f seeds/test_data.sql
```

### influxdb/

Time-series database for sensor readings

#### schemas/

InfluxDB bucket and retention policy configurations

**Buckets:**

- `sensor_data` - Raw sensor readings (30-day retention)
- `sensor_data_hourly` - Hourly aggregates (2-year retention)
- `sensor_data_daily` - Daily aggregates (indefinite retention)
- `predictions` - Model predictions (1-year retention)

**Creating Buckets:**

```bash
influx bucket create \
  --name sensor_data \
  --org aqmrg \
  --retention 30d

influx bucket create \
  --name sensor_data_hourly \
  --org aqmrg \
  --retention 730d
```

#### continuous-queries/

Downsampling and aggregation queries

**Example Continuous Query:**

```flux
// Downsample raw data to hourly averages
option task = {name: "downsample_hourly", every: 1h}

from(bucket: "sensor_data")
  |> range(start: -1h)
  |> filter(fn: (r) => r["_measurement"] == "air_quality")
  |> aggregateWindow(every: 1h, fn: mean)
  |> to(bucket: "sensor_data_hourly")
```

### redis/

Cache and session store

#### schemas/

Redis data structure definitions and key patterns

**Key Patterns:**

```
# User sessions
session:{session_id} → Hash {user_id, expires_at, ...}

# API rate limiting
rate_limit:{user_id}:{endpoint} → Counter with TTL

# Cached predictions
prediction:{location_id}:{timestamp} → JSON

# Real-time sensor data cache
sensor:{sensor_id}:latest → JSON

# Alert state
alert:{alert_id}:state → String {triggered|normal}
```

## Database Schemas

### PostgreSQL Schema

#### Users Table

```sql
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    role VARCHAR(50) NOT NULL DEFAULT 'public',
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_role ON users(role);
```

#### Sensors Table

```sql
CREATE TABLE sensors (
    id VARCHAR(100) PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    manufacturer VARCHAR(100),
    location_id UUID REFERENCES locations(id),
    status VARCHAR(50) DEFAULT 'active',
    metadata JSONB,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_sensors_location ON sensors(location_id);
CREATE INDEX idx_sensors_status ON sensors(status);
```

#### Alerts Table

```sql
CREATE TABLE alerts (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id),
    location_id UUID REFERENCES locations(id),
    pollutant VARCHAR(50) NOT NULL,
    threshold DECIMAL(10,2) NOT NULL,
    enabled BOOLEAN DEFAULT true,
    notification_channels JSONB,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_alerts_user ON alerts(user_id);
CREATE INDEX idx_alerts_enabled ON alerts(enabled);
```

### InfluxDB Schema

#### Measurement: air_quality

```
air_quality,sensor_id=ABC123,location=Lagos pm25=35.2,pm10=45.8,temperature=28.5 1609459200000000000
```

**Tags:**

- `sensor_id` - Sensor identifier
- `location` - Location name or ID
- `manufacturer` - Sensor manufacturer

**Fields:**

- `pm25` - PM2.5 concentration (µg/m³)
- `pm10` - PM10 concentration (µg/m³)
- `temperature` - Temperature (°C)
- `humidity` - Relative humidity (%)
- `co` - Carbon monoxide (ppm)
- `no2` - Nitrogen dioxide (ppb)
- `o3` - Ozone (ppb)

## Connection Examples

### PostgreSQL Connection

**Node.js (pg):**

```javascript
const { Pool } = require("pg");

const pool = new Pool({
  host: process.env.DATABASE_HOST,
  port: process.env.DATABASE_PORT,
  database: process.env.DATABASE_NAME,
  user: process.env.DATABASE_USER,
  password: process.env.DATABASE_PASSWORD,
  max: 20,
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 2000,
});

const result = await pool.query("SELECT * FROM sensors WHERE status = $1", [
  "active",
]);
```

**Python (psycopg2):**

```python
import psycopg2
from psycopg2.extras import RealDictCursor

conn = psycopg2.connect(
    host=os.getenv('DATABASE_HOST'),
    port=os.getenv('DATABASE_PORT'),
    database=os.getenv('DATABASE_NAME'),
    user=os.getenv('DATABASE_USER'),
    password=os.getenv('DATABASE_PASSWORD')
)

cursor = conn.cursor(cursor_factory=RealDictCursor)
cursor.execute("SELECT * FROM sensors WHERE status = %s", ('active',))
results = cursor.fetchall()
```

### InfluxDB Connection

**Node.js:**

```javascript
const { InfluxDB } = require("@influxdata/influxdb-client");

const influxDB = new InfluxDB({
  url: process.env.INFLUXDB_URL,
  token: process.env.INFLUXDB_TOKEN,
});

const queryApi = influxDB.getQueryApi(process.env.INFLUXDB_ORG);
const query = `
  from(bucket: "sensor_data")
    |> range(start: -1h)
    |> filter(fn: (r) => r._measurement == "air_quality")
`;
const data = await queryApi.collectRows(query);
```

**Python:**

```python
from influxdb_client import InfluxDBClient

client = InfluxDBClient(
    url=os.getenv('INFLUXDB_URL'),
    token=os.getenv('INFLUXDB_TOKEN'),
    org=os.getenv('INFLUXDB_ORG')
)

query_api = client.query_api()
query = '''
from(bucket: "sensor_data")
  |> range(start: -1h)
  |> filter(fn: (r) => r._measurement == "air_quality")
'''
result = query_api.query(query)
```

### Redis Connection

**Node.js:**

```javascript
const Redis = require("ioredis");

const redis = new Redis({
  host: process.env.REDIS_HOST,
  port: process.env.REDIS_PORT,
  password: process.env.REDIS_PASSWORD,
});

await redis.set("key", "value", "EX", 3600); // Set with 1-hour expiry
const value = await redis.get("key");
```

**Python:**

```python
import redis

r = redis.Redis(
    host=os.getenv('REDIS_HOST'),
    port=os.getenv('REDIS_PORT'),
    password=os.getenv('REDIS_PASSWORD'),
    decode_responses=True
)

r.setex('key', 3600, 'value')  # Set with 1-hour expiry
value = r.get('key')
```

## Backup and Recovery

### PostgreSQL Backup

```bash
# Create backup
pg_dump -U aqmrg -d aqmrg -F c -f backup_$(date +%Y%m%d).dump

# Restore backup
pg_restore -U aqmrg -d aqmrg backup_20260101.dump
```

### InfluxDB Backup

```bash
# Create backup
influx backup /path/to/backup

# Restore backup
influx restore /path/to/backup
```

### Redis Backup

```bash
# Trigger RDB snapshot
redis-cli BGSAVE

# Copy RDB file
cp /var/lib/redis/dump.rdb /backup/
```

## Performance Optimization

### PostgreSQL

- Create indexes on frequently queried columns
- Use connection pooling
- Regularly run `VACUUM ANALYZE`
- Partition large tables by date

### InfluxDB

- Use appropriate retention policies
- Create continuous queries for downsampling
- Optimize tag cardinality
- Use batch writes

### Redis

- Set appropriate TTLs on keys
- Use pipelining for bulk operations
- Monitor memory usage
- Configure eviction policies

## Monitoring

### PostgreSQL

```sql
-- Check active connections
SELECT count(*) FROM pg_stat_activity;

-- Check slow queries
SELECT query, calls, total_time, mean_time
FROM pg_stat_statements
ORDER BY mean_time DESC
LIMIT 10;

-- Check table sizes
SELECT schemaname, tablename,
       pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) AS size
FROM pg_tables
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC;
```

### InfluxDB

- Monitor bucket sizes
- Check query performance
- Monitor write rates
- Track cardinality

### Redis

```bash
# Check memory usage
redis-cli INFO memory

# Check key count
redis-cli DBSIZE

# Monitor commands
redis-cli MONITOR
```

## Troubleshooting

### PostgreSQL Connection Issues

```bash
# Check if PostgreSQL is running
docker-compose ps postgres

# View logs
docker-compose logs postgres

# Test connection
psql -U aqmrg -d aqmrg -h localhost -p 5432
```

### InfluxDB Issues

```bash
# Check bucket list
influx bucket list

# Verify token
influx auth list

# Test query
influx query 'from(bucket:"sensor_data") |> range(start: -1h) |> limit(n:1)'
```

### Redis Issues

```bash
# Test connection
redis-cli ping

# Check info
redis-cli INFO

# Monitor commands
redis-cli MONITOR
```

## Dependencies

```
# Node.js
pg@8.11.0
ioredis@5.3.0
@influxdata/influxdb-client@1.33.0

# Python
psycopg2-binary==2.9.9
redis==5.0.1
influxdb-client==1.38.0
```
