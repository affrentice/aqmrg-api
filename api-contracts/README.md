# API Contracts

API specifications and contracts for the AQMRG AI Analytics Platform.

## Overview

This directory contains formal API specifications using industry-standard formats:

- **OpenAPI (Swagger)**: REST API specifications
- **GraphQL**: GraphQL schema definitions (optional)
- **AsyncAPI**: Asynchronous API specifications for Kafka/messaging

## Directory Structure

### openapi/

OpenAPI/Swagger specifications for REST APIs

#### v1/

API version 1 specifications

```
openapi/v1/
├── api-gateway.yaml        # Main API gateway specification
├── auth-service.yaml       # Authentication endpoints
├── analytics-service.yaml  # Analytics endpoints
├── predictions.yaml        # Prediction endpoints
├── admin.yaml             # Admin endpoints
└── components/            # Reusable components
    ├── schemas.yaml       # Common schemas
    ├── responses.yaml     # Common responses
    └── parameters.yaml    # Common parameters
```

### graphql/

GraphQL schema definitions (if using GraphQL)

```
graphql/
├── schema.graphql         # Main schema
├── types/                # Type definitions
│   ├── sensor.graphql
│   ├── prediction.graphql
│   └── user.graphql
├── queries.graphql       # Query definitions
├── mutations.graphql     # Mutation definitions
└── subscriptions.graphql # Subscription definitions
```

### asyncapi/

AsyncAPI specifications for Kafka topics and messaging

```
asyncapi/
├── kafka-topics.yaml     # Kafka topic specifications
├── sensor-events.yaml    # Sensor data events
└── alert-events.yaml     # Alert notification events
```

## OpenAPI Specifications

### Main API Gateway Specification

**File**: `openapi/v1/api-gateway.yaml`

```yaml
openapi: 3.0.0
info:
  title: AQMRG AI Analytics API
  version: 1.0.0
  description: Air quality monitoring and prediction API
  contact:
    email: api@aqmrg.org

servers:
  - url: https://api.aqmrg.org/v1
    description: Production server
  - url: https://staging-api.aqmrg.org/v1
    description: Staging server
  - url: http://localhost:8000/v1
    description: Development server

paths:
  /health:
    get:
      summary: Health check
      tags: [System]
      responses:
        "200":
          description: Service is healthy
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/HealthResponse"

  /auth/login:
    post:
      summary: User login
      tags: [Authentication]
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: "#/components/schemas/LoginRequest"
      responses:
        "200":
          description: Login successful
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/LoginResponse"

  /dashboard/realtime:
    get:
      summary: Get real-time dashboard data
      tags: [Dashboard]
      security: [] # Public endpoint
      parameters:
        - name: location
          in: query
          schema:
            type: string
      responses:
        "200":
          description: Real-time dashboard data
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/DashboardData"

  /predictions/forecast:
    get:
      summary: Get air quality predictions
      tags: [Predictions]
      parameters:
        - name: location
          in: query
          required: true
          schema:
            type: string
        - name: hours
          in: query
          schema:
            type: integer
            enum: [4, 24, 72]
            default: 24
      responses:
        "200":
          description: Air quality predictions
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/PredictionResponse"

  /analytics/historical:
    get:
      summary: Get historical analytics data
      tags: [Analytics]
      security:
        - bearerAuth: [] # Optional authentication
      parameters:
        - name: location
          in: query
          required: true
          schema:
            type: string
        - name: start_date
          in: query
          required: true
          schema:
            type: string
            format: date-time
        - name: end_date
          in: query
          required: true
          schema:
            type: string
            format: date-time
      responses:
        "200":
          description: Historical data
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/HistoricalData"

  /admin/models/deploy:
    post:
      summary: Deploy ML model
      tags: [Admin]
      security:
        - bearerAuth: []
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: "#/components/schemas/ModelDeployRequest"
      responses:
        "200":
          description: Model deployed successfully

components:
  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT

  schemas:
    HealthResponse:
      type: object
      properties:
        status:
          type: string
          enum: [healthy, degraded, unhealthy]
        timestamp:
          type: string
          format: date-time
        services:
          type: object
          properties:
            database:
              type: string
            cache:
              type: string
            kafka:
              type: string

    LoginRequest:
      type: object
      required:
        - email
        - password
      properties:
        email:
          type: string
          format: email
        password:
          type: string
          format: password

    LoginResponse:
      type: object
      properties:
        access_token:
          type: string
        refresh_token:
          type: string
        expires_in:
          type: integer
        user:
          $ref: "#/components/schemas/User"

    User:
      type: object
      properties:
        id:
          type: string
          format: uuid
        email:
          type: string
        role:
          type: string
          enum: [public, authenticated, admin]

    DashboardData:
      type: object
      properties:
        current_conditions:
          type: array
          items:
            $ref: "#/components/schemas/SensorReading"
        health_advisory:
          $ref: "#/components/schemas/HealthAdvisory"
        recent_trends:
          type: array
          items:
            $ref: "#/components/schemas/TrendData"

    SensorReading:
      type: object
      properties:
        sensor_id:
          type: string
        timestamp:
          type: string
          format: date-time
        location:
          $ref: "#/components/schemas/Location"
        pm25:
          type: number
          format: float
        pm10:
          type: number
          format: float
        aqi:
          type: integer
          minimum: 0
          maximum: 500

    Location:
      type: object
      properties:
        lat:
          type: number
          format: double
        lon:
          type: number
          format: double
        name:
          type: string

    PredictionResponse:
      type: object
      properties:
        location:
          $ref: "#/components/schemas/Location"
        predictions:
          type: array
          items:
            type: object
            properties:
              timestamp:
                type: string
                format: date-time
              pm25:
                type: number
              pm10:
                type: number
              aqi:
                type: integer
              confidence:
                type: number
                minimum: 0
                maximum: 1

    Error:
      type: object
      properties:
        code:
          type: string
        message:
          type: string
        details:
          type: object
```

## GraphQL Schema Example

**File**: `graphql/schema.graphql`

```graphql
type Query {
  # Public queries
  currentAirQuality(location: LocationInput!): AirQuality
  predictions(location: LocationInput!, hours: Int = 24): [Prediction!]!

  # Authenticated queries
  historicalData(
    location: LocationInput!
    startDate: DateTime!
    endDate: DateTime!
  ): [HistoricalReading!]! @auth

  # Admin queries
  sensors: [Sensor!]! @auth(role: ADMIN)
}

type Mutation {
  # Authentication
  login(email: String!, password: String!): AuthPayload!

  # Alerts
  createAlert(input: AlertInput!): Alert! @auth

  # Admin mutations
  deploySensor(input: SensorInput!): Sensor! @auth(role: ADMIN)
}

type Subscription {
  # Real-time subscriptions
  airQualityUpdates(location: LocationInput!): AirQuality!
  alertTriggered(userId: ID!): Alert! @auth
}

type AirQuality {
  location: Location!
  timestamp: DateTime!
  pm25: Float!
  pm10: Float!
  aqi: Int!
  healthAdvisory: HealthAdvisory!
}

type Prediction {
  timestamp: DateTime!
  pm25: Float!
  pm10: Float!
  aqi: Int!
  confidence: Float!
}

type Location {
  lat: Float!
  lon: Float!
  name: String!
}

input LocationInput {
  lat: Float!
  lon: Float!
}

directive @auth(role: Role = AUTHENTICATED) on FIELD_DEFINITION

enum Role {
  PUBLIC
  AUTHENTICATED
  ADMIN
}
```

## AsyncAPI Specification Example

**File**: `asyncapi/kafka-topics.yaml`

```yaml
asyncapi: 2.6.0
info:
  title: AQMRG Kafka Events
  version: 1.0.0
  description: Kafka topics for sensor data and alerts

servers:
  development:
    url: localhost:9092
    protocol: kafka
  production:
    url: kafka.aqmrg.org:9092
    protocol: kafka

channels:
  sensor.raw.airquality:
    description: Raw air quality sensor readings
    publish:
      summary: Sensor publishes raw data
      message:
        $ref: "#/components/messages/SensorReading"

  sensor.processed.validated:
    description: Validated and processed sensor data
    subscribe:
      summary: Services consume validated data
      message:
        $ref: "#/components/messages/ValidatedReading"

  alerts.triggered:
    description: Air quality alerts
    subscribe:
      summary: Notification service consumes alerts
      message:
        $ref: "#/components/messages/Alert"

components:
  messages:
    SensorReading:
      contentType: application/json
      payload:
        type: object
        properties:
          sensor_id:
            type: string
          timestamp:
            type: string
            format: date-time
          location:
            type: object
            properties:
              lat:
                type: number
              lon:
                type: number
          pm25:
            type: number
          pm10:
            type: number

    Alert:
      contentType: application/json
      payload:
        type: object
        properties:
          alert_id:
            type: string
          user_id:
            type: string
          location:
            type: object
          pollutant:
            type: string
          threshold:
            type: number
          current_value:
            type: number
          triggered_at:
            type: string
            format: date-time
```

## Tools and Validation

### Swagger/OpenAPI Tools

**Swagger Editor** (Online):

```
https://editor.swagger.io/
```

**Swagger UI** (Local):

```bash
npm install -g swagger-ui-watcher
swagger-ui-watcher openapi/v1/api-gateway.yaml
```

**Validation**:

```bash
npm install -g @apidevtools/swagger-cli
swagger-cli validate openapi/v1/api-gateway.yaml
```

### GraphQL Tools

**GraphQL Playground**:

```bash
npm install -g graphql-playground
graphql-playground
```

### AsyncAPI Tools

**AsyncAPI Generator**:

```bash
npm install -g @asyncapi/generator
asyncapi generate asyncapi/kafka-topics.yaml @asyncapi/html-template
```

## Code Generation

### Generate Client SDK (OpenAPI)

**TypeScript/JavaScript**:

```bash
npm install -g @openapitools/openapi-generator-cli
openapi-generator-cli generate \
  -i openapi/v1/api-gateway.yaml \
  -g typescript-axios \
  -o generated/typescript-client
```

**Python**:

```bash
openapi-generator-cli generate \
  -i openapi/v1/api-gateway.yaml \
  -g python \
  -o generated/python-client
```

### Generate Server Stubs

```bash
openapi-generator-cli generate \
  -i openapi/v1/api-gateway.yaml \
  -g nodejs-express-server \
  -o generated/server-stub
```

## Best Practices

1. **Versioning**: Use semantic versioning (v1, v2, etc.)
2. **Documentation**: Include examples for all endpoints
3. **Validation**: Validate specs before committing
4. **Consistency**: Use consistent naming conventions
5. **Security**: Document authentication requirements
6. **Examples**: Provide request/response examples
7. **Errors**: Document all error responses

## API Documentation Publishing

Generate and publish API documentation:

```bash
# Generate HTML docs
npx redoc-cli bundle openapi/v1/api-gateway.yaml -o docs/api-reference.html

# Serve locally
npx redoc-cli serve openapi/v1/api-gateway.yaml
```

Access at: http://localhost:8080

## Dependencies

```
@apidevtools/swagger-cli
@openapitools/openapi-generator-cli
@asyncapi/generator
redoc-cli
```
