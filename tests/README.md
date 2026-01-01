# Tests

Comprehensive test suite for the AQMRG AI Analytics Backend Platform.

## Overview

This directory contains all tests organized by type:

- **Unit Tests**: Test individual functions and components
- **Integration Tests**: Test service interactions
- **E2E Tests**: Test complete API workflows
- **Load Tests**: Test performance and scalability
- **Contract Tests**: Test API contracts

## Directory Structure

### integration/

Cross-service integration tests

```
integration/
├── test_auth_flow.js           # Authentication integration
├── test_data_pipeline.js       # Data ingestion to storage
├── test_prediction_flow.js     # ML prediction pipeline
├── test_alert_system.js        # Alert triggering and delivery
└── fixtures/                   # Test fixtures and data
```

### e2e/

End-to-end API tests

```
e2e/
├── test_public_api.js          # Public endpoint tests
├── test_authenticated_api.js   # Authenticated endpoint tests
├── test_admin_api.js           # Admin endpoint tests
└── scenarios/                  # Complete user scenarios
    ├── new_user_signup.js
    ├── create_alert.js
    └── export_data.js
```

### load/

Performance and load tests

```
load/
├── load_test_dashboard.js      # Dashboard load testing
├── load_test_predictions.js    # Prediction API load testing
├── stress_test.js              # Stress testing
└── spike_test.js               # Spike testing
```

### contract/

API contract tests

```
contract/
├── test_openapi_contract.js    # Validate OpenAPI spec
├── test_kafka_contracts.js     # Validate Kafka messages
└── test_graphql_schema.js      # Validate GraphQL schema
```

## Running Tests

### All Tests

```bash
# Run all tests
npm test

# Or using make
make test
```

### Unit Tests

```bash
# Run unit tests
npm run test:unit

# With coverage
npm run test:unit -- --coverage
```

### Integration Tests

```bash
# Run integration tests
npm run test:integration

# Run specific integration test
npm run test:integration -- test_auth_flow
```

### E2E Tests

```bash
# Run e2e tests
npm run test:e2e

# Run in headless mode
npm run test:e2e:headless
```

### Load Tests

```bash
# Run load tests
npm run test:load

# Run specific load test
k6 run tests/load/load_test_dashboard.js
```

### Contract Tests

```bash
# Run contract tests
npm run test:contract

# Validate OpenAPI spec
npm run test:contract:openapi
```

## Test Examples

### Integration Test Example

**File**: `integration/test_auth_flow.js`

```javascript
const request = require("supertest");
const { expect } = require("chai");

describe("Authentication Flow Integration", () => {
  let app;
  let accessToken;

  before(async () => {
    // Setup test environment
    app = require("../../services/api-gateway/src/app");
  });

  describe("User Login Flow", () => {
    it("should reject invalid credentials", async () => {
      const res = await request(app).post("/api/v1/auth/login").send({
        email: "invalid@example.com",
        password: "wrongpassword",
      });

      expect(res.status).to.equal(401);
      expect(res.body).to.have.property("error");
    });

    it("should login with valid credentials", async () => {
      const res = await request(app).post("/api/v1/auth/login").send({
        email: "test@example.com",
        password: "testpassword123",
      });

      expect(res.status).to.equal(200);
      expect(res.body).to.have.property("access_token");
      expect(res.body).to.have.property("refresh_token");

      accessToken = res.body.access_token;
    });

    it("should access protected endpoint with token", async () => {
      const res = await request(app)
        .get("/api/v1/analytics/historical")
        .set("Authorization", `Bearer ${accessToken}`)
        .query({
          location: "Lagos",
          start_date: "2026-01-01T00:00:00Z",
          end_date: "2026-01-02T00:00:00Z",
        });

      expect(res.status).to.equal(200);
      expect(res.body).to.be.an("array");
    });

    it("should refresh access token", async () => {
      const res = await request(app).post("/api/v1/auth/refresh").send({
        refresh_token: refreshToken,
      });

      expect(res.status).to.equal(200);
      expect(res.body).to.have.property("access_token");
    });
  });
});
```

### E2E Test Example

**File**: `e2e/test_public_api.js`

```javascript
const axios = require("axios");
const { expect } = require("chai");

describe("Public API E2E Tests", () => {
  const API_URL = process.env.API_URL || "http://localhost:8000";

  describe("Dashboard Endpoint", () => {
    it("should return real-time dashboard data", async () => {
      const response = await axios.get(`${API_URL}/api/v1/dashboard/realtime`, {
        params: { location: "Lagos" },
      });

      expect(response.status).to.equal(200);
      expect(response.data).to.have.property("current_conditions");
      expect(response.data).to.have.property("health_advisory");
      expect(response.data.current_conditions).to.be.an("array");
    });

    it("should handle invalid location gracefully", async () => {
      try {
        await axios.get(`${API_URL}/api/v1/dashboard/realtime`, {
          params: { location: "InvalidLocation123" },
        });
      } catch (error) {
        expect(error.response.status).to.equal(404);
        expect(error.response.data).to.have.property("error");
      }
    });
  });

  describe("Predictions Endpoint", () => {
    it("should return air quality predictions", async () => {
      const response = await axios.get(
        `${API_URL}/api/v1/predictions/forecast`,
        {
          params: {
            location: "Lagos",
            hours: 24,
          },
        }
      );

      expect(response.status).to.equal(200);
      expect(response.data).to.have.property("predictions");
      expect(response.data.predictions).to.be.an("array");
      expect(response.data.predictions.length).to.be.greaterThan(0);

      const prediction = response.data.predictions[0];
      expect(prediction).to.have.property("timestamp");
      expect(prediction).to.have.property("pm25");
      expect(prediction).to.have.property("aqi");
      expect(prediction).to.have.property("confidence");
    });
  });

  describe("Health Check", () => {
    it("should return healthy status", async () => {
      const response = await axios.get(`${API_URL}/health`);

      expect(response.status).to.equal(200);
      expect(response.data).to.have.property("status", "healthy");
      expect(response.data).to.have.property("services");
    });
  });
});
```

### Load Test Example (K6)

**File**: `load/load_test_dashboard.js`

```javascript
import http from "k6/http";
import { check, sleep } from "k6";
import { Rate } from "k6/metrics";

const failureRate = new Rate("failed_requests");

export let options = {
  stages: [
    { duration: "2m", target: 100 }, // Ramp up to 100 users
    { duration: "5m", target: 100 }, // Stay at 100 users
    { duration: "2m", target: 200 }, // Ramp up to 200 users
    { duration: "5m", target: 200 }, // Stay at 200 users
    { duration: "2m", target: 0 }, // Ramp down to 0 users
  ],
  thresholds: {
    http_req_duration: ["p(95)<500"], // 95% of requests under 500ms
    failed_requests: ["rate<0.1"], // Less than 10% failure rate
  },
};

export default function () {
  const API_URL = __ENV.API_URL || "http://localhost:8000";

  // Test dashboard endpoint
  let dashboardRes = http.get(
    `${API_URL}/api/v1/dashboard/realtime?location=Lagos`
  );

  let dashboardCheck = check(dashboardRes, {
    "dashboard status is 200": (r) => r.status === 200,
    "dashboard has current_conditions": (r) =>
      JSON.parse(r.body).hasOwnProperty("current_conditions"),
  });

  failureRate.add(!dashboardCheck);

  // Test predictions endpoint
  let predictionRes = http.get(
    `${API_URL}/api/v1/predictions/forecast?location=Lagos&hours=24`
  );

  let predictionCheck = check(predictionRes, {
    "prediction status is 200": (r) => r.status === 200,
    "prediction has predictions array": (r) =>
      JSON.parse(r.body).hasOwnProperty("predictions"),
  });

  failureRate.add(!predictionCheck);

  sleep(1);
}
```

**Running Load Tests:**

```bash
k6 run tests/load/load_test_dashboard.js
```

### Contract Test Example

**File**: `contract/test_openapi_contract.js`

```javascript
const SwaggerParser = require("@apidevtools/swagger-parser");
const { expect } = require("chai");

describe("OpenAPI Contract Tests", () => {
  let apiSpec;

  before(async () => {
    // Parse and validate OpenAPI spec
    apiSpec = await SwaggerParser.validate(
      "api-contracts/openapi/v1/api-gateway.yaml"
    );
  });

  it("should have valid OpenAPI 3.0 specification", () => {
    expect(apiSpec.openapi).to.equal("3.0.0");
  });

  it("should define all required endpoints", () => {
    const requiredPaths = [
      "/health",
      "/auth/login",
      "/dashboard/realtime",
      "/predictions/forecast",
      "/analytics/historical",
    ];

    requiredPaths.forEach((path) => {
      expect(apiSpec.paths).to.have.property(path);
    });
  });

  it("should define security schemes", () => {
    expect(apiSpec.components.securitySchemes).to.have.property("bearerAuth");
  });

  it("should have proper error responses defined", () => {
    Object.values(apiSpec.paths).forEach((pathItem) => {
      Object.values(pathItem).forEach((operation) => {
        if (operation.responses) {
          // Should have at least one error response
          const hasErrorResponse = Object.keys(operation.responses).some(
            (code) => code >= 400
          );
          expect(hasErrorResponse).to.be.true;
        }
      });
    });
  });
});
```

## Test Configuration

### Jest Configuration

**File**: `jest.config.js` (in root)

```javascript
module.exports = {
  testEnvironment: "node",
  coverageDirectory: "coverage",
  collectCoverageFrom: [
    "services/**/src/**/*.js",
    "!**/node_modules/**",
    "!**/tests/**",
  ],
  testMatch: ["**/tests/**/*.test.js", "**/tests/**/*.spec.js"],
  setupFilesAfterEnv: ["./tests/setup.js"],
  coverageThreshold: {
    global: {
      branches: 70,
      functions: 70,
      lines: 70,
      statements: 70,
    },
  },
};
```

### Test Setup

**File**: `tests/setup.js`

```javascript
// Global test setup
const { MongoMemoryServer } = require("mongodb-memory-server");
const Redis = require("ioredis-mock");

// Setup in-memory databases for testing
beforeAll(async () => {
  // Setup MongoDB Memory Server
  global.mongoServer = await MongoMemoryServer.create();
  process.env.DATABASE_URL = global.mongoServer.getUri();

  // Setup Redis Mock
  global.redisMock = new Redis();
});

afterAll(async () => {
  // Cleanup
  if (global.mongoServer) {
    await global.mongoServer.stop();
  }
});
```

## CI/CD Integration

### GitHub Actions

```yaml
name: Test Suite

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest

    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_PASSWORD: postgres
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

      redis:
        image: redis:7
        options: >-
          --health-cmd "redis-cli ping"
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    steps:
      - uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: "18"

      - name: Install dependencies
        run: npm install

      - name: Run unit tests
        run: npm run test:unit

      - name: Run integration tests
        run: npm run test:integration
        env:
          DATABASE_URL: postgresql://postgres:postgres@localhost:5432/test
          REDIS_URL: redis://localhost:6379

      - name: Run e2e tests
        run: npm run test:e2e

      - name: Upload coverage
        uses: codecov/codecov-action@v3
```

## Best Practices

1. **Test Isolation**: Each test should be independent
2. **Use Fixtures**: Reuse test data with fixtures
3. **Mock External Services**: Don't rely on external APIs
4. **Test Edge Cases**: Test error conditions, not just happy paths
5. **Maintainable Tests**: Keep tests simple and readable
6. **Fast Feedback**: Run unit tests frequently
7. **Comprehensive Coverage**: Aim for >70% code coverage

## Test Data

### Fixtures

Create reusable test data in `fixtures/`:

```javascript
// fixtures/sensors.js
module.exports = {
  validSensor: {
    id: "TEST001",
    name: "Test Sensor",
    manufacturer: "TestCo",
    location: {
      lat: 6.5244,
      lon: 3.3792,
    },
  },
  invalidSensor: {
    id: "",
    name: null,
  },
};
```

## Debugging Tests

```bash
# Run tests in debug mode
node --inspect-brk node_modules/.bin/jest --runInBand

# Run specific test file
npm test -- tests/integration/test_auth_flow.js

# Run tests with verbose output
npm test -- --verbose

# Update snapshots
npm test -- --updateSnapshot
```

## Dependencies

```json
{
  "devDependencies": {
    "jest": "^29.0.0",
    "supertest": "^6.3.0",
    "chai": "^4.3.0",
    "mocha": "^10.2.0",
    "@apidevtools/swagger-parser": "^10.1.0",
    "k6": "^0.47.0",
    "axios": "^1.6.0"
  }
}
```
