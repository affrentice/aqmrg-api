# Configuration

Environment-specific configurations and feature flags for the AQMRG AI Analytics Platform.

## Directory Structure

### environments/

Environment-specific configuration files

```
environments/
├── development.yaml        # Development environment config
├── staging.yaml           # Staging environment config
└── production.yaml        # Production environment config
```

### feature-flags/

Feature toggle configurations

```
feature-flags/
├── features.yaml          # Feature flag definitions
└── README.md             # Feature flag documentation
```

## Environment Configurations

### Development Configuration

**File**: `environments/development.yaml`

```yaml
environment: development

api:
  base_url: http://localhost:8000
  version: v1
  cors:
    enabled: true
    origins:
      - http://localhost:3000
      - http://localhost:3001
  rate_limiting:
    enabled: false
    window: 15m
    max_requests: 1000

database:
  postgres:
    host: localhost
    port: 5432
    database: aqmrg_dev
    pool:
      min: 2
      max: 10

  influxdb:
    url: http://localhost:8086
    org: aqmrg
    bucket: sensor_data_dev

  redis:
    host: localhost
    port: 6379
    db: 0

messaging:
  kafka:
    brokers:
      - localhost:9092
    client_id: aqmrg-dev
    consumer_groups:
      sensors: aqmrg-sensors-dev
      alerts: aqmrg-alerts-dev

ml:
  mlflow:
    tracking_uri: http://localhost:5000
    experiment_name: aqmrg-dev

  models:
    prediction_4hr:
      version: latest
      endpoint: http://localhost:8003/predict/4hr
    prediction_24hr:
      version: latest
      endpoint: http://localhost:8003/predict/24hr

auth:
  jwt:
    secret: dev_jwt_secret_change_in_production
    expires_in: 1h
    refresh_expires_in: 7d

  password:
    min_length: 8
    require_special_chars: false

logging:
  level: debug
  format: json
  output: stdout

monitoring:
  enabled: true
  prometheus:
    port: 9090
  grafana:
    url: http://localhost:3000

features:
  enable_predictions: true
  enable_alerts: true
  enable_export: true
  enable_admin_panel: true
```

### Staging Configuration

**File**: `environments/staging.yaml`

```yaml
environment: staging

api:
  base_url: https://staging-api.aqmrg.org
  version: v1
  cors:
    enabled: true
    origins:
      - https://staging.aqmrg.org
  rate_limiting:
    enabled: true
    window: 15m
    max_requests:
      public: 100
      authenticated: 1000

database:
  postgres:
    host: staging-postgres.aqmrg.org
    port: 5432
    database: aqmrg_staging
    ssl: true
    pool:
      min: 5
      max: 20

  influxdb:
    url: https://staging-influxdb.aqmrg.org
    org: aqmrg
    bucket: sensor_data_staging

  redis:
    host: staging-redis.aqmrg.org
    port: 6379
    ssl: true
    db: 0

messaging:
  kafka:
    brokers:
      - staging-kafka-1.aqmrg.org:9092
      - staging-kafka-2.aqmrg.org:9092
    client_id: aqmrg-staging
    ssl: true
    consumer_groups:
      sensors: aqmrg-sensors-staging
      alerts: aqmrg-alerts-staging

ml:
  mlflow:
    tracking_uri: https://staging-mlflow.aqmrg.org
    experiment_name: aqmrg-staging

  models:
    prediction_4hr:
      version: v1.2.0
      endpoint: https://staging-model-serving.aqmrg.org/predict/4hr
    prediction_24hr:
      version: v1.2.0
      endpoint: https://staging-model-serving.aqmrg.org/predict/24hr

auth:
  jwt:
    secret: ${JWT_SECRET} # From environment variable
    expires_in: 1h
    refresh_expires_in: 7d

  password:
    min_length: 10
    require_special_chars: true

logging:
  level: info
  format: json
  output: stdout

monitoring:
  enabled: true
  prometheus:
    url: https://staging-prometheus.aqmrg.org
  grafana:
    url: https://staging-grafana.aqmrg.org

features:
  enable_predictions: true
  enable_alerts: true
  enable_export: true
  enable_admin_panel: true
```

### Production Configuration

**File**: `environments/production.yaml`

```yaml
environment: production

api:
  base_url: https://api.aqmrg.org
  version: v1
  cors:
    enabled: true
    origins:
      - https://aqmrg.org
      - https://www.aqmrg.org
  rate_limiting:
    enabled: true
    window: 15m
    max_requests:
      public: 100
      authenticated: 1000
      admin: unlimited

database:
  postgres:
    host: prod-postgres-primary.aqmrg.org
    port: 5432
    database: aqmrg
    ssl: true
    ssl_mode: require
    pool:
      min: 10
      max: 50
    read_replicas:
      - prod-postgres-replica-1.aqmrg.org
      - prod-postgres-replica-2.aqmrg.org

  influxdb:
    url: https://prod-influxdb.aqmrg.org
    org: aqmrg
    bucket: sensor_data
    retention: 730d # 2 years

  redis:
    cluster:
      enabled: true
      nodes:
        - prod-redis-1.aqmrg.org:6379
        - prod-redis-2.aqmrg.org:6379
        - prod-redis-3.aqmrg.org:6379
    ssl: true
    password: ${REDIS_PASSWORD}

messaging:
  kafka:
    brokers:
      - prod-kafka-1.aqmrg.org:9092
      - prod-kafka-2.aqmrg.org:9092
      - prod-kafka-3.aqmrg.org:9092
    client_id: aqmrg-prod
    ssl: true
    sasl:
      mechanism: PLAIN
      username: ${KAFKA_USERNAME}
      password: ${KAFKA_PASSWORD}
    consumer_groups:
      sensors: aqmrg-sensors-prod
      alerts: aqmrg-alerts-prod

ml:
  mlflow:
    tracking_uri: https://mlflow.aqmrg.org
    experiment_name: aqmrg-production

  models:
    prediction_4hr:
      version: v2.1.0
      endpoint: https://model-serving.aqmrg.org/predict/4hr
      replicas: 3
    prediction_24hr:
      version: v2.1.0
      endpoint: https://model-serving.aqmrg.org/predict/24hr
      replicas: 3
    prediction_72hr:
      version: v2.0.5
      endpoint: https://model-serving.aqmrg.org/predict/72hr
      replicas: 2

auth:
  jwt:
    secret: ${JWT_SECRET}
    algorithm: HS256
    expires_in: 1h
    refresh_expires_in: 7d

  password:
    min_length: 12
    require_special_chars: true
    require_uppercase: true
    require_numbers: true

  mfa:
    enabled: true
    required_for_admin: true

logging:
  level: warn
  format: json
  output:
    - stdout
    - elasticsearch
  elasticsearch:
    url: https://elasticsearch.aqmrg.org
    index: aqmrg-logs

monitoring:
  enabled: true
  prometheus:
    url: https://prometheus.aqmrg.org
  grafana:
    url: https://grafana.aqmrg.org
  alertmanager:
    url: https://alertmanager.aqmrg.org

  alerts:
    email:
      - ops@aqmrg.org
      - oncall@aqmrg.org
    slack:
      webhook_url: ${SLACK_WEBHOOK_URL}

features:
  enable_predictions: true
  enable_alerts: true
  enable_export: true
  enable_admin_panel: true
  enable_beta_features: false

security:
  ssl:
    enabled: true
    cert_path: /etc/ssl/certs/aqmrg.crt
    key_path: /etc/ssl/private/aqmrg.key

  encryption:
    at_rest: true
    in_transit: true

  audit_logging:
    enabled: true
    retention_days: 90

backup:
  enabled: true
  schedule: "0 2 * * *" # Daily at 2 AM
  retention_days: 30
  destinations:
    - s3://aqmrg-backups/postgres
    - s3://aqmrg-backups/influxdb

performance:
  cache_ttl:
    dashboard: 30s
    predictions: 5m
    historical: 1h

  connection_timeouts:
    database: 30s
    api: 10s
    model_serving: 30s
```

## Feature Flags

### Feature Flag Configuration

**File**: `feature-flags/features.yaml`

```yaml
features:
  # Prediction Features
  predictions_4hr:
    enabled: true
    description: "4-hour air quality predictions"
    rollout_percentage: 100

  predictions_24hr:
    enabled: true
    description: "24-hour air quality predictions"
    rollout_percentage: 100

  predictions_72hr:
    enabled: true
    description: "72-hour air quality predictions"
    rollout_percentage: 50 # Gradual rollout

  # Analytics Features
  advanced_analytics:
    enabled: true
    description: "Advanced analytics and filtering"
    requires_auth: true
    rollout_percentage: 100

  custom_reports:
    enabled: true
    description: "Custom report generation"
    requires_auth: true
    allowed_roles:
      - authenticated
      - admin

  # Export Features
  csv_export:
    enabled: true
    description: "CSV data export"
    max_rows: 10000

  excel_export:
    enabled: false
    description: "Excel data export"
    beta: true

  api_export:
    enabled: true
    description: "Programmatic API export"
    requires_auth: true
    rate_limit: 100

  # Alert Features
  email_alerts:
    enabled: true
    description: "Email alert notifications"

  sms_alerts:
    enabled: true
    description: "SMS alert notifications"
    premium: true

  push_notifications:
    enabled: false
    description: "Push notifications"
    beta: true

  # Admin Features
  model_deployment:
    enabled: true
    description: "ML model deployment"
    requires_role: admin

  sensor_management:
    enabled: true
    description: "Sensor configuration"
    requires_role: admin

  user_management:
    enabled: true
    description: "User account management"
    requires_role: admin

  # Experimental Features
  ai_insights:
    enabled: false
    description: "AI-generated insights"
    experimental: true
    rollout_percentage: 5

  predictive_alerts:
    enabled: false
    description: "Predictive air quality alerts"
    experimental: true
    beta: true
```

## Using Configuration

### Loading Configuration in Code

**Node.js Example:**

```javascript
const yaml = require("js-yaml");
const fs = require("fs");

function loadConfig(environment) {
  const configPath = `config/environments/${environment}.yaml`;
  const config = yaml.load(fs.readFileSync(configPath, "utf8"));

  // Resolve environment variables
  return resolveEnvVars(config);
}

function resolveEnvVars(obj) {
  for (let key in obj) {
    if (typeof obj[key] === "string" && obj[key].startsWith("${")) {
      const envVar = obj[key].slice(2, -1);
      obj[key] = process.env[envVar];
    } else if (typeof obj[key] === "object") {
      obj[key] = resolveEnvVars(obj[key]);
    }
  }
  return obj;
}

// Usage
const config = loadConfig(process.env.NODE_ENV || "development");
```

**Python Example:**

```python
import yaml
import os
from pathlib import Path

def load_config(environment='development'):
    config_path = Path(__file__).parent / 'environments' / f'{environment}.yaml'

    with open(config_path, 'r') as f:
        config = yaml.safe_load(f)

    # Resolve environment variables
    return resolve_env_vars(config)

def resolve_env_vars(obj):
    if isinstance(obj, dict):
        return {k: resolve_env_vars(v) for k, v in obj.items()}
    elif isinstance(obj, list):
        return [resolve_env_vars(item) for item in obj]
    elif isinstance(obj, str) and obj.startswith('${') and obj.endswith('}'):
        env_var = obj[2:-1]
        return os.getenv(env_var)
    return obj

# Usage
config = load_config(os.getenv('ENVIRONMENT', 'development'))
```

### Feature Flag Usage

```javascript
const features = require("./feature-flags/features.yaml");

function isFeatureEnabled(featureName, user = null) {
  const feature = features.features[featureName];

  if (!feature || !feature.enabled) {
    return false;
  }

  // Check authentication requirement
  if (feature.requires_auth && !user) {
    return false;
  }

  // Check role requirement
  if (feature.requires_role && user.role !== feature.requires_role) {
    return false;
  }

  // Check rollout percentage
  if (feature.rollout_percentage < 100) {
    const userHash = hashUser(user);
    return userHash % 100 < feature.rollout_percentage;
  }

  return true;
}

// Usage
if (isFeatureEnabled("predictions_72hr", currentUser)) {
  // Show 72-hour predictions
}
```

## Best Practices

1. **Environment Variables**: Use environment variables for secrets
2. **No Secrets in Configs**: Never commit secrets to version control
3. **Consistent Structure**: Keep same structure across environments
4. **Documentation**: Document all configuration options
5. **Validation**: Validate configuration on startup
6. **Defaults**: Provide sensible defaults
7. **Feature Flags**: Use feature flags for gradual rollouts

## Configuration Validation

```javascript
const Joi = require("joi");

const configSchema = Joi.object({
  environment: Joi.string()
    .valid("development", "staging", "production")
    .required(),
  api: Joi.object({
    base_url: Joi.string().uri().required(),
    version: Joi.string().required(),
  }).required(),
  database: Joi.object({
    postgres: Joi.object({
      host: Joi.string().required(),
      port: Joi.number().required(),
    }).required(),
  }).required(),
});

// Validate on startup
const { error } = configSchema.validate(config);
if (error) {
  throw new Error(`Configuration validation failed: ${error.message}`);
}
```
