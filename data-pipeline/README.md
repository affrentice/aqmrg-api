# Data Pipeline

Real-time data processing infrastructure for sensor data ingestion, transformation, and validation.

## Architecture Overview

```
Sensors → Kafka Producers → Kafka Topics → Stream Processors → Databases
                                         ↓
                                  Data Validators
                                         ↓
                                  Airflow DAGs (Batch Processing)
```

## Directory Structure

### kafka/

Apache Kafka streaming components

#### producers/

- Sensor data producers
- Event publishers
- Data ingestion clients

#### consumers/

- Real-time data processors
- Database writers
- Alert trigger consumers

#### topics/

- Topic configurations
- Schema definitions
- Partition strategies
- Retention policies

### airflow/

Apache Airflow workflow orchestration

#### dags/

- ETL workflow definitions
- Scheduled data processing jobs
- Data quality monitoring tasks
- Model retraining pipelines

### stream-processors/

Real-time data transformation logic:

- Data normalization
- Unit conversions
- Aggregation windows
- Enrichment with metadata

### sensor-adapters/

Pluggable adapters for sensor manufacturers

#### airqo/

AirQo sensor integration:

- API client
- Data format conversion
- Authentication handling

#### purpleair/

PurpleAir sensor integration:

- API client
- Rate limiting
- Data mapping

#### template/

Template for new sensor integrations:

- Base adapter class
- Standard interface
- Configuration schema

### data-validators/

Data quality and anomaly detection:

- Range validation
- Missing data detection
- Outlier identification
- Data drift monitoring

## Kafka Topics

### Raw Data Topics

- `sensor.raw.airquality` - Raw sensor readings
- `sensor.raw.weather` - Weather data
- `sensor.raw.health` - Health data

### Processed Data Topics

- `sensor.processed.validated` - Validated sensor data
- `sensor.processed.aggregated` - Aggregated metrics
- `alerts.triggered` - Triggered alerts

### System Topics

- `system.events` - System events
- `system.errors` - Error logs

## Data Flow

### 1. Sensor Data Ingestion

```python
# Producer sends data to Kafka
producer.send('sensor.raw.airquality', {
    'sensor_id': 'ABC123',
    'timestamp': '2026-01-01T12:00:00Z',
    'pm25': 35.2,
    'pm10': 45.8,
    'location': {'lat': 6.5244, 'lon': 3.3792}
})
```

### 2. Stream Processing

```python
# Consumer processes and validates
consumer = KafkaConsumer('sensor.raw.airquality')
for message in consumer:
    data = validate_sensor_data(message.value)
    enriched = enrich_with_metadata(data)
    save_to_influxdb(enriched)
```

### 3. Batch Processing (Airflow)

```python
# Daily aggregation DAG
@dag(schedule_interval='@daily')
def daily_aggregation():
    aggregate_task = PythonOperator(
        task_id='aggregate_daily_metrics',
        python_callable=aggregate_metrics
    )
```

## Sensor Adapter Development

### Creating a New Adapter

1. Copy the template:

```bash
cp -r sensor-adapters/template sensor-adapters/new-sensor
```

2. Implement required methods:

```python
class NewSensorAdapter(BaseSensorAdapter):
    def fetch_data(self):
        """Fetch data from sensor API"""

    def transform_data(self, raw_data):
        """Transform to standard format"""

    def validate_data(self, data):
        """Validate data quality"""
```

3. Register adapter in configuration:

```yaml
sensors:
  - type: new-sensor
    adapter: NewSensorAdapter
    config:
      api_key: ${NEW_SENSOR_API_KEY}
      endpoint: https://api.newsensor.com
```

## Data Validation Rules

### Required Fields

- `sensor_id` (string)
- `timestamp` (ISO 8601)
- `location` (lat/lon)
- At least one pollutant reading

### Value Ranges

- PM2.5: 0-500 µg/m³
- PM10: 0-600 µg/m³
- Temperature: -50 to 60°C
- Humidity: 0-100%

### Data Quality Checks

- Timestamp within last 5 minutes (for real-time)
- No duplicate readings
- Sensor ID exists in registry
- Location within valid bounds

## Running Components

### Start Kafka

```bash
docker-compose up -d kafka zookeeper
```

### Start Producers

```bash
cd kafka/producers
python sensor_producer.py --config config.yaml
```

### Start Consumers

```bash
cd kafka/consumers
python data_consumer.py --topics sensor.raw.airquality
```

### Start Airflow

```bash
airflow webserver --port 8080
airflow scheduler
```

## Monitoring

### Kafka Monitoring

```bash
# Check topic lag
kafka-consumer-groups.sh --bootstrap-server localhost:9092 --describe --group sensor-consumers

# View topic messages
kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic sensor.raw.airquality
```

### Airflow Monitoring

- Web UI: http://localhost:8080
- Task logs in `logs/` directory
- Metrics in Prometheus

## Configuration

### Kafka Configuration

```yaml
# kafka/config.yaml
bootstrap_servers:
  - localhost:9092
compression_type: gzip
batch_size: 16384
linger_ms: 10
```

### Airflow Configuration

```yaml
# airflow/airflow.cfg
executor: CeleryExecutor
parallelism: 32
dag_concurrency: 16
max_active_runs_per_dag: 3
```

## Performance Tuning

### Kafka Optimization

- Increase `batch.size` for higher throughput
- Tune `linger.ms` for latency vs throughput
- Use compression (`gzip`, `snappy`, `lz4`)
- Partition topics by sensor location

### Airflow Optimization

- Use appropriate executor (Celery for scale)
- Configure worker concurrency
- Set task timeout values
- Monitor task duration

## Troubleshooting

### Data Not Flowing

1. Check Kafka broker status
2. Verify producer connectivity
3. Check consumer group lag
4. Review data validation errors

### High Latency

1. Check network connectivity
2. Monitor Kafka broker metrics
3. Review consumer processing time
4. Check database write performance

### Data Quality Issues

1. Review validation logs
2. Check sensor adapter implementations
3. Verify data transformation logic
4. Monitor outlier detection alerts

## Dependencies

```
kafka-python==2.0.2
apache-airflow==2.7.0
influxdb-client==1.38.0
psycopg2-binary==2.9.9
redis==5.0.1
```
