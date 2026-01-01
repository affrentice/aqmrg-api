# Data Ingestion Service

Real-time sensor data streaming and validation service for the AQMRG AI Analytics Platform.

## Overview

The Data Ingestion Service handles real-time air quality sensor data from multiple manufacturers, validates the data, and publishes it to Kafka topics for downstream processing and storage.

## Responsibilities

- **Multi-Source Data Collection**: Pull data from various sensor manufacturer APIs
- **Data Validation**: Validate sensor readings against quality rules
- **Data Normalization**: Convert different formats to standard schema
- **Kafka Publishing**: Stream validated data to Kafka topics
- **Error Handling**: Retry failed ingestions and log errors
- **Rate Limiting**: Respect API rate limits for external sources
- **Data Enrichment**: Add metadata (location, sensor info, timestamps)
- **Monitoring**: Track data flow metrics and alert on issues
- **Duplicate Detection**: Prevent duplicate sensor readings

## Technology Stack

**Language**: Node.js (TypeScript) or Python  
**Framework**: Express.js or FastAPI  
**Message Queue**: Apache Kafka  
**Database**: InfluxDB (time-series), PostgreSQL (metadata)  
**Dependencies**:

- `kafkajs` or `kafka-python` - Kafka client
- `axios` or `httpx` - HTTP requests
- `@influxdata/influxdb-client` or `influxdb-client` - InfluxDB
- `pg` or `psycopg2` - PostgreSQL
- `joi` or `pydantic` - Data validation
- `node-cron` or `apscheduler` - Job scheduling

## Port

**Default**: `8002`

## Data Flow

```
Sensor APIs → Data Ingestion Service → Validation → Kafka → [InfluxDB, Analytics, Alerts]
                                          ↓
                                   Error Logging
```

## API Endpoints

### Data Ingestion

```
POST /ingest/sensor-reading     - Manually ingest sensor reading
POST /ingest/batch              - Batch ingest multiple readings
GET  /ingest/status             - Ingestion pipeline status
```

### Sensor Management

```
GET  /sensors                   - List configured sensors
GET  /sensors/:id               - Get sensor details
POST /sensors                   - Register new sensor (admin)
PUT  /sensors/:id               - Update sensor (admin)
DELETE /sensors/:id             - Remove sensor (admin)
```

### Monitoring

```
GET  /health                    - Health check
GET  /metrics                   - Prometheus metrics
GET  /stats                     - Ingestion statistics
```

## Directory Structure

```
data-ingestion-service/
├── src/
│   ├── index.ts                    # Application entry point
│   ├── app.ts                      # Express/FastAPI app setup
│   ├── config/
│   │   ├── kafka.ts               # Kafka configuration
│   │   ├── influxdb.ts            # InfluxDB configuration
│   │   └── sensors.ts             # Sensor configurations
│   ├── adapters/
│   │   ├── BaseAdapter.ts         # Base adapter interface
│   │   ├── AirQoAdapter.ts        # AirQo sensor adapter
│   │   ├── PurpleAirAdapter.ts    # PurpleAir sensor adapter
│   │   └── CustomAdapter.ts       # Custom sensor adapter
│   ├── services/
│   │   ├── ingestionService.ts    # Main ingestion logic
│   │   ├── validationService.ts   # Data validation
│   │   ├── kafkaProducer.ts       # Kafka producer
│   │   └── influxWriter.ts        # InfluxDB writer
│   ├── schedulers/
│   │   └── dataPoller.ts          # Scheduled data polling
│   ├── models/
│   │   └── SensorReading.ts       # Data models
│   ├── validators/
│   │   └── sensorDataSchema.ts    # Validation schemas
│   └── utils/
│       ├── logger.ts              # Winston logger
│       └── metrics.ts             # Prometheus metrics
├── tests/
│   ├── unit/
│   │   ├── validation.test.ts
│   │   └── adapters.test.ts
│   └── integration/
│       └── ingestion.test.ts
├── .env.example
├── Dockerfile
├── package.json
└── README.md
```

## Data Schema

### Standard Sensor Reading Format

```typescript
interface SensorReading {
  sensor_id: string; // Unique sensor identifier
  timestamp: string; // ISO 8601 timestamp
  location: {
    lat: number; // Latitude
    lon: number; // Longitude
    name?: string; // Location name
  };
  measurements: {
    pm25?: number; // PM2.5 (µg/m³)
    pm10?: number; // PM10 (µg/m³)
    temperature?: number; // Temperature (°C)
    humidity?: number; // Relative humidity (%)
    co?: number; // Carbon monoxide (ppm)
    no2?: number; // Nitrogen dioxide (ppb)
    o3?: number; // Ozone (ppb)
    so2?: number; // Sulfur dioxide (ppb)
  };
  metadata?: {
    manufacturer: string; // Sensor manufacturer
    model: string; // Sensor model
    firmware_version?: string;
    calibration_date?: string;
  };
  quality_flags?: {
    validated: boolean; // Passed validation
    anomaly_score?: number; // Anomaly detection score
    issues?: string[]; // Any quality issues
  };
}
```

## Configuration

### Environment Variables

```bash
# Service
DATA_INGESTION_SERVICE_PORT=8002

# Kafka
KAFKA_BROKERS=localhost:9092
KAFKA_TOPIC_RAW=sensor.raw.airquality
KAFKA_TOPIC_VALIDATED=sensor.processed.validated
KAFKA_CLIENT_ID=data-ingestion-service

# InfluxDB
INFLUXDB_URL=http://localhost:8086
INFLUXDB_TOKEN=your_influxdb_token
INFLUXDB_ORG=aqmrg
INFLUXDB_BUCKET=sensor_data

# PostgreSQL (for sensor metadata)
DATABASE_URL=postgresql://aqmrg:password@localhost:5432/aqmrg

# Sensor APIs
AIRQO_API_KEY=your_airqo_api_key
AIRQO_API_URL=https://api.airqo.net/api/v2
PURPLEAIR_API_KEY=your_purpleair_api_key
PURPLEAIR_API_URL=https://api.purpleair.com/v1

# Polling Schedule
POLLING_INTERVAL=60000        # Poll every 60 seconds
BATCH_SIZE=100                # Process 100 readings per batch

# Validation
VALIDATION_STRICT_MODE=false
ENABLE_ANOMALY_DETECTION=true
```

## Setup & Installation

### Node.js

```bash
# Install dependencies
npm install

# Development
npm run dev

# Production
npm start

# Tests
npm test
```

### Python

```bash
# Create virtual environment
python -m venv venv
source venv/bin/activate

# Install dependencies
pip install -r requirements.txt

# Development
python main.py

# Tests
pytest
```

## Example Implementation

### Base Sensor Adapter

```typescript
// src/adapters/BaseAdapter.ts
export abstract class BaseAdapter {
  protected apiKey: string;
  protected apiUrl: string;

  constructor(apiKey: string, apiUrl: string) {
    this.apiKey = apiKey;
    this.apiUrl = apiUrl;
  }

  abstract async fetchReadings(): Promise<any[]>;
  abstract transformReading(raw: any): SensorReading;

  async collectData(): Promise<SensorReading[]> {
    try {
      const rawData = await this.fetchReadings();
      return rawData.map((reading) => this.transformReading(reading));
    } catch (error) {
      console.error(
        `Error collecting data from ${this.constructor.name}:`,
        error
      );
      throw error;
    }
  }
}
```

### AirQo Adapter Implementation

```typescript
// src/adapters/AirQoAdapter.ts
import axios from "axios";
import { BaseAdapter } from "./BaseAdapter";

export class AirQoAdapter extends BaseAdapter {
  async fetchReadings(): Promise<any[]> {
    const response = await axios.get(`${this.apiUrl}/devices/measurements`, {
      headers: {
        Authorization: `Bearer ${this.apiKey}`,
      },
      params: {
        recent: "yes",
        limit: 100,
      },
    });

    return response.data.measurements || [];
  }

  transformReading(raw: any): SensorReading {
    return {
      sensor_id: raw.device_id,
      timestamp: raw.time,
      location: {
        lat: raw.site?.latitude,
        lon: raw.site?.longitude,
        name: raw.site?.name,
      },
      measurements: {
        pm25: raw.pm2_5?.value,
        pm10: raw.pm10?.value,
        temperature: raw.temperature?.value,
        humidity: raw.humidity?.value,
      },
      metadata: {
        manufacturer: "AirQo",
        model: raw.device?.device_number,
      },
    };
  }
}
```

### Data Validation Service

```typescript
// src/services/validationService.ts
import Joi from "joi";

const sensorReadingSchema = Joi.object({
  sensor_id: Joi.string().required(),
  timestamp: Joi.date().iso().required(),
  location: Joi.object({
    lat: Joi.number().min(-90).max(90).required(),
    lon: Joi.number().min(-180).max(180).required(),
    name: Joi.string().optional(),
  }).required(),
  measurements: Joi.object({
    pm25: Joi.number().min(0).max(1000).optional(),
    pm10: Joi.number().min(0).max(2000).optional(),
    temperature: Joi.number().min(-50).max(60).optional(),
    humidity: Joi.number().min(0).max(100).optional(),
  }).required(),
});

export class ValidationService {
  validate(reading: SensorReading): { valid: boolean; errors?: string[] } {
    const { error } = sensorReadingSchema.validate(reading);

    if (error) {
      return {
        valid: false,
        errors: error.details.map((d) => d.message),
      };
    }

    // Additional custom validations
    const customErrors = this.customValidations(reading);

    if (customErrors.length > 0) {
      return { valid: false, errors: customErrors };
    }

    return { valid: true };
  }

  private customValidations(reading: SensorReading): string[] {
    const errors: string[] = [];

    // Check timestamp is not in the future
    if (new Date(reading.timestamp) > new Date()) {
      errors.push("Timestamp cannot be in the future");
    }

    // Check timestamp is not too old (e.g., older than 1 hour)
    const oneHourAgo = new Date(Date.now() - 3600000);
    if (new Date(reading.timestamp) < oneHourAgo) {
      errors.push("Timestamp is too old");
    }

    // Check at least one measurement is present
    const hasMeasurement = Object.values(reading.measurements).some(
      (v) => v !== undefined
    );
    if (!hasMeasurement) {
      errors.push("At least one measurement is required");
    }

    return errors;
  }

  async detectAnomalies(reading: SensorReading): Promise<number> {
    // Simple anomaly detection based on statistical outliers
    // In production, use ML-based anomaly detection

    let anomalyScore = 0;

    // Check for extreme values
    if (reading.measurements.pm25 && reading.measurements.pm25 > 500) {
      anomalyScore += 0.5;
    }

    if (reading.measurements.temperature) {
      if (
        reading.measurements.temperature < -20 ||
        reading.measurements.temperature > 50
      ) {
        anomalyScore += 0.3;
      }
    }

    return anomalyScore;
  }
}
```

### Kafka Producer Service

```typescript
// src/services/kafkaProducer.ts
import { Kafka, Producer } from "kafkajs";

export class KafkaProducerService {
  private producer: Producer;
  private kafka: Kafka;

  constructor() {
    this.kafka = new Kafka({
      clientId: process.env.KAFKA_CLIENT_ID,
      brokers: process.env.KAFKA_BROKERS!.split(","),
    });

    this.producer = this.kafka.producer();
  }

  async connect() {
    await this.producer.connect();
    console.log("Kafka producer connected");
  }

  async publishReading(reading: SensorReading, topic: string) {
    try {
      await this.producer.send({
        topic,
        messages: [
          {
            key: reading.sensor_id,
            value: JSON.stringify(reading),
            timestamp: new Date(reading.timestamp).getTime().toString(),
          },
        ],
      });
    } catch (error) {
      console.error("Error publishing to Kafka:", error);
      throw error;
    }
  }

  async publishBatch(readings: SensorReading[], topic: string) {
    const messages = readings.map((reading) => ({
      key: reading.sensor_id,
      value: JSON.stringify(reading),
      timestamp: new Date(reading.timestamp).getTime().toString(),
    }));

    await this.producer.send({
      topic,
      messages,
    });
  }

  async disconnect() {
    await this.producer.disconnect();
  }
}
```

### Main Ingestion Service

```typescript
// src/services/ingestionService.ts
import { AirQoAdapter } from "../adapters/AirQoAdapter";
import { PurpleAirAdapter } from "../adapters/PurpleAirAdapter";
import { ValidationService } from "./validationService";
import { KafkaProducerService } from "./kafkaProducer";
import { InfluxWriterService } from "./influxWriter";

export class IngestionService {
  private adapters: BaseAdapter[];
  private validator: ValidationService;
  private kafkaProducer: KafkaProducerService;
  private influxWriter: InfluxWriterService;

  constructor() {
    this.adapters = [
      new AirQoAdapter(process.env.AIRQO_API_KEY!, process.env.AIRQO_API_URL!),
      new PurpleAirAdapter(
        process.env.PURPLEAIR_API_KEY!,
        process.env.PURPLEAIR_API_URL!
      ),
    ];

    this.validator = new ValidationService();
    this.kafkaProducer = new KafkaProducerService();
    this.influxWriter = new InfluxWriterService();
  }

  async initialize() {
    await this.kafkaProducer.connect();
    console.log("Ingestion service initialized");
  }

  async ingestData() {
    console.log("Starting data ingestion...");

    for (const adapter of this.adapters) {
      try {
        const readings = await adapter.collectData();
        console.log(
          `Collected ${readings.length} readings from ${adapter.constructor.name}`
        );

        await this.processReadings(readings);
      } catch (error) {
        console.error(`Error with adapter ${adapter.constructor.name}:`, error);
      }
    }
  }

  async processReadings(readings: SensorReading[]) {
    const validReadings: SensorReading[] = [];
    const invalidReadings: SensorReading[] = [];

    for (const reading of readings) {
      // Validate
      const validation = this.validator.validate(reading);

      if (validation.valid) {
        // Detect anomalies
        const anomalyScore = await this.validator.detectAnomalies(reading);

        reading.quality_flags = {
          validated: true,
          anomaly_score: anomalyScore,
          issues: anomalyScore > 0.5 ? ["High anomaly score"] : [],
        };

        validReadings.push(reading);
      } else {
        console.warn(
          `Invalid reading from ${reading.sensor_id}:`,
          validation.errors
        );
        invalidReadings.push(reading);
      }
    }

    // Publish to Kafka
    if (validReadings.length > 0) {
      await this.kafkaProducer.publishBatch(
        validReadings,
        process.env.KAFKA_TOPIC_VALIDATED!
      );

      // Write to InfluxDB
      await this.influxWriter.writeBatch(validReadings);

      console.log(`Published ${validReadings.length} valid readings`);
    }

    if (invalidReadings.length > 0) {
      console.log(`Skipped ${invalidReadings.length} invalid readings`);
    }
  }
}
```

### Scheduled Data Polling

```typescript
// src/schedulers/dataPoller.ts
import cron from "node-cron";
import { IngestionService } from "../services/ingestionService";

export class DataPoller {
  private ingestionService: IngestionService;
  private pollingInterval: string;

  constructor(ingestionService: IngestionService) {
    this.ingestionService = ingestionService;
    // Default: every minute
    this.pollingInterval = process.env.POLLING_CRON || "*/1 * * * *";
  }

  start() {
    console.log(`Starting data polling (interval: ${this.pollingInterval})`);

    cron.schedule(this.pollingInterval, async () => {
      try {
        await this.ingestionService.ingestData();
      } catch (error) {
        console.error("Polling error:", error);
      }
    });
  }
}
```

## InfluxDB Writer

```typescript
// src/services/influxWriter.ts
import { InfluxDB, Point } from "@influxdata/influxdb-client";

export class InfluxWriterService {
  private influxDB: InfluxDB;
  private writeApi: any;

  constructor() {
    this.influxDB = new InfluxDB({
      url: process.env.INFLUXDB_URL!,
      token: process.env.INFLUXDB_TOKEN!,
    });

    this.writeApi = this.influxDB.getWriteApi(
      process.env.INFLUXDB_ORG!,
      process.env.INFLUXDB_BUCKET!
    );
  }

  async writeBatch(readings: SensorReading[]) {
    for (const reading of readings) {
      const point = new Point("air_quality")
        .tag("sensor_id", reading.sensor_id)
        .tag("location", reading.location.name || "unknown")
        .timestamp(new Date(reading.timestamp));

      // Add measurements as fields
      if (reading.measurements.pm25 !== undefined) {
        point.floatField("pm25", reading.measurements.pm25);
      }
      if (reading.measurements.pm10 !== undefined) {
        point.floatField("pm10", reading.measurements.pm10);
      }
      if (reading.measurements.temperature !== undefined) {
        point.floatField("temperature", reading.measurements.temperature);
      }
      if (reading.measurements.humidity !== undefined) {
        point.floatField("humidity", reading.measurements.humidity);
      }

      this.writeApi.writePoint(point);
    }

    await this.writeApi.flush();
  }
}
```

## Monitoring & Metrics

```typescript
// src/utils/metrics.ts
import promClient from "prom-client";

export const readingsIngested = new promClient.Counter({
  name: "readings_ingested_total",
  help: "Total number of sensor readings ingested",
  labelNames: ["source", "status"],
});

export const ingestionDuration = new promClient.Histogram({
  name: "ingestion_duration_seconds",
  help: "Duration of data ingestion process",
  labelNames: ["source"],
});

export const validationErrors = new promClient.Counter({
  name: "validation_errors_total",
  help: "Total number of validation errors",
  labelNames: ["error_type"],
});
```

## Error Handling & Retry Logic

```typescript
async function retryWithBackoff(
  fn: () => Promise<any>,
  maxRetries: number = 3,
  initialDelay: number = 1000
) {
  for (let attempt = 0; attempt < maxRetries; attempt++) {
    try {
      return await fn();
    } catch (error) {
      if (attempt === maxRetries - 1) {
        throw error;
      }

      const delay = initialDelay * Math.pow(2, attempt);
      console.log(`Retry attempt ${attempt + 1} after ${delay}ms`);
      await new Promise((resolve) => setTimeout(resolve, delay));
    }
  }
}
```

## Testing

```typescript
// tests/unit/validation.test.ts
import { ValidationService } from "../../src/services/validationService";

describe("ValidationService", () => {
  const validator = new ValidationService();

  it("should validate correct sensor reading", () => {
    const reading = {
      sensor_id: "TEST001",
      timestamp: new Date().toISOString(),
      location: { lat: 6.5244, lon: 3.3792 },
      measurements: { pm25: 35.2, pm10: 45.8 },
    };

    const result = validator.validate(reading);
    expect(result.valid).toBe(true);
  });

  it("should reject reading with invalid coordinates", () => {
    const reading = {
      sensor_id: "TEST001",
      timestamp: new Date().toISOString(),
      location: { lat: 100, lon: 200 }, // Invalid
      measurements: { pm25: 35.2 },
    };

    const result = validator.validate(reading);
    expect(result.valid).toBe(false);
  });
});
```

## Dependencies

```json
{
  "dependencies": {
    "express": "^4.18.0",
    "kafkajs": "^2.2.0",
    "@influxdata/influxdb-client": "^1.33.0",
    "axios": "^1.6.0",
    "joi": "^17.11.0",
    "node-cron": "^3.0.0",
    "pg": "^8.11.0",
    "winston": "^3.11.0",
    "prom-client": "^15.0.0"
  }
}
```
