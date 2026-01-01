# API Gateway Service

Central entry point for all client requests to the AQMRG AI Analytics Platform.

## Overview

The API Gateway acts as a reverse proxy and provides a unified interface for all backend microservices. It handles cross-cutting concerns like authentication, rate limiting, request routing, and monitoring.

## Responsibilities

- **Request Routing**: Route requests to appropriate backend services
- **Authentication**: Validate JWT tokens and enforce access control
- **Rate Limiting**: Prevent abuse with configurable rate limits
- **CORS Management**: Handle cross-origin requests from frontend
- **Request/Response Logging**: Track all API interactions
- **API Documentation**: Serve OpenAPI/Swagger documentation
- **Health Checks**: Monitor downstream service health
- **Request Validation**: Validate incoming requests against schemas
- **Response Caching**: Cache frequent requests for performance
- **Load Balancing**: Distribute load across service instances

## Technology Stack

**Language**: Node.js (TypeScript) or Python (FastAPI)  
**Framework**: Express.js or FastAPI  
**Dependencies**:

- `express` or `fastapi` - Web framework
- `cors` - CORS middleware
- `helmet` - Security headers
- `express-rate-limit` or `slowapi` - Rate limiting
- `jsonwebtoken` - JWT validation
- `http-proxy-middleware` - Service proxying
- `swagger-ui-express` or `fastapi[swagger]` - API docs
- `winston` or `loguru` - Logging
- `ioredis` or `redis` - Caching

## Port

**Default**: `8000`

## API Routes

### System Routes

```
GET  /health          - Health check endpoint
GET  /ready           - Readiness probe
GET  /metrics         - Prometheus metrics
GET  /api-docs        - Swagger UI documentation
```

### Authentication Routes (proxied to auth-service)

```
POST /api/v1/auth/login           - User login
POST /api/v1/auth/register        - User registration
POST /api/v1/auth/logout          - User logout
POST /api/v1/auth/refresh         - Refresh access token
GET  /api/v1/auth/me              - Get current user
```

### Public Routes

```
GET  /api/v1/dashboard/realtime   - Real-time dashboard data
GET  /api/v1/predictions/forecast - Air quality predictions
GET  /api/v1/sensors/current      - Current sensor readings
GET  /api/v1/locations            - Available locations
```

### Authenticated Routes

```
GET  /api/v1/analytics/historical - Historical analytics data
POST /api/v1/data/export          - Export data
GET  /api/v1/alerts               - User alerts
POST /api/v1/alerts               - Create alert
PUT  /api/v1/alerts/:id           - Update alert
DELETE /api/v1/alerts/:id         - Delete alert
```

### Admin Routes

```
POST /api/v1/admin/models/deploy  - Deploy ML model
GET  /api/v1/admin/models         - List deployed models
POST /api/v1/admin/sensors        - Configure sensor
GET  /api/v1/admin/sensors        - List sensors
GET  /api/v1/admin/users          - List users
PUT  /api/v1/admin/users/:id      - Update user
GET  /api/v1/admin/monitoring     - System monitoring data
```

## Directory Structure

```
api-gateway/
├── src/
│   ├── index.ts                 # Application entry point
│   ├── app.ts                   # Express/FastAPI app setup
│   ├── config/
│   │   ├── index.ts            # Configuration loader
│   │   └── routes.ts           # Route definitions
│   ├── middleware/
│   │   ├── auth.ts             # Authentication middleware
│   │   ├── rateLimit.ts        # Rate limiting middleware
│   │   ├── validation.ts       # Request validation
│   │   ├── logging.ts          # Request logging
│   │   └── errorHandler.ts    # Error handling
│   ├── routes/
│   │   ├── health.ts           # Health check routes
│   │   ├── auth.ts             # Auth proxy routes
│   │   ├── dashboard.ts        # Dashboard proxy routes
│   │   ├── predictions.ts      # Predictions proxy routes
│   │   └── admin.ts            # Admin proxy routes
│   ├── services/
│   │   ├── proxy.ts            # Service proxy logic
│   │   └── serviceRegistry.ts # Service discovery
│   └── utils/
│       ├── logger.ts           # Winston logger
│       └── redis.ts            # Redis client
├── tests/
│   ├── unit/
│   │   ├── middleware.test.ts
│   │   └── routes.test.ts
│   └── integration/
│       └── api.test.ts
├── .env.example                # Environment variables template
├── Dockerfile                  # Docker container definition
├── package.json                # Node.js dependencies
├── tsconfig.json               # TypeScript configuration
└── README.md                   # This file
```

## Configuration

### Environment Variables

```bash
# API Gateway
API_GATEWAY_PORT=8000
API_GATEWAY_HOST=0.0.0.0

# CORS
CORS_ORIGIN=http://localhost:3000,https://aqmrg.org
CORS_CREDENTIALS=true

# Rate Limiting
RATE_LIMIT_WINDOW=15m
RATE_LIMIT_PUBLIC=100
RATE_LIMIT_AUTHENTICATED=1000

# Backend Services
AUTH_SERVICE_URL=http://auth-service:8001
DATA_INGESTION_SERVICE_URL=http://data-ingestion-service:8002
MODEL_SERVING_SERVICE_URL=http://model-serving-service:8003
ANALYTICS_SERVICE_URL=http://analytics-service:8004
NOTIFICATION_SERVICE_URL=http://notification-service:8005
SENSOR_ADAPTER_SERVICE_URL=http://sensor-adapter-service:8006
EXPORT_SERVICE_URL=http://export-service:8007

# Authentication
JWT_SECRET=your_jwt_secret
JWT_ALGORITHM=HS256

# Redis (for rate limiting and caching)
REDIS_URL=redis://localhost:6379

# Logging
LOG_LEVEL=info
```

## Setup & Installation

### Node.js/TypeScript Version

```bash
# Install dependencies
npm install

# Development
npm run dev

# Build
npm run build

# Production
npm start

# Tests
npm test

# Lint
npm run lint
```

### Python/FastAPI Version

```bash
# Create virtual environment
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Development
uvicorn main:app --reload --host 0.0.0.0 --port 8000

# Production
gunicorn -w 4 -k uvicorn.workers.UvicornWorker main:app --bind 0.0.0.0:8000

# Tests
pytest

# Lint
black . && flake8
```

## Example Implementation

### Node.js/Express Example

```typescript
// src/index.ts
import express from "express";
import cors from "cors";
import helmet from "helmet";
import { createProxyMiddleware } from "http-proxy-middleware";
import rateLimit from "express-rate-limit";
import { authMiddleware } from "./middleware/auth";
import { loggingMiddleware } from "./middleware/logging";

const app = express();
const PORT = process.env.API_GATEWAY_PORT || 8000;

// Middleware
app.use(helmet());
app.use(
  cors({
    origin: process.env.CORS_ORIGIN?.split(","),
    credentials: true,
  })
);
app.use(express.json());
app.use(loggingMiddleware);

// Rate limiting
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // limit each IP to 100 requests per windowMs
});
app.use(limiter);

// Health check
app.get("/health", (req, res) => {
  res.json({ status: "healthy", timestamp: new Date().toISOString() });
});

// Public routes - no authentication
app.use(
  "/api/v1/dashboard",
  createProxyMiddleware({
    target: process.env.ANALYTICS_SERVICE_URL,
    changeOrigin: true,
    pathRewrite: { "^/api/v1/dashboard": "/dashboard" },
  })
);

app.use(
  "/api/v1/predictions",
  createProxyMiddleware({
    target: process.env.MODEL_SERVING_SERVICE_URL,
    changeOrigin: true,
    pathRewrite: { "^/api/v1/predictions": "/predictions" },
  })
);

// Authenticated routes
app.use(
  "/api/v1/analytics",
  authMiddleware,
  createProxyMiddleware({
    target: process.env.ANALYTICS_SERVICE_URL,
    changeOrigin: true,
    pathRewrite: { "^/api/v1/analytics": "/analytics" },
  })
);

// Admin routes
app.use(
  "/api/v1/admin",
  authMiddleware,
  (req, res, next) => {
    if (req.user.role !== "admin") {
      return res.status(403).json({ error: "Forbidden" });
    }
    next();
  },
  createProxyMiddleware({
    target: process.env.ANALYTICS_SERVICE_URL,
    changeOrigin: true,
    pathRewrite: { "^/api/v1/admin": "/admin" },
  })
);

app.listen(PORT, () => {
  console.log(`API Gateway running on port ${PORT}`);
});
```

### Python/FastAPI Example

```python
# main.py
from fastapi import FastAPI, Request, HTTPException, Depends
from fastapi.middleware.cors import CORSMiddleware
from fastapi.responses import JSONResponse
import httpx
from datetime import datetime
import os

app = FastAPI(title="AQMRG API Gateway", version="1.0.0")

# CORS
app.add_middleware(
    CORSMiddleware,
    allow_origins=os.getenv("CORS_ORIGIN", "").split(","),
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# Service URLs
SERVICES = {
    "auth": os.getenv("AUTH_SERVICE_URL"),
    "analytics": os.getenv("ANALYTICS_SERVICE_URL"),
    "model_serving": os.getenv("MODEL_SERVING_SERVICE_URL"),
}

@app.get("/health")
async def health_check():
    return {"status": "healthy", "timestamp": datetime.utcnow().isoformat()}

@app.get("/api/v1/dashboard/realtime")
async def get_realtime_dashboard(location: str = None):
    async with httpx.AsyncClient() as client:
        params = {"location": location} if location else {}
        response = await client.get(
            f"{SERVICES['analytics']}/dashboard/realtime",
            params=params
        )
        return response.json()

@app.get("/api/v1/predictions/forecast")
async def get_predictions(location: str, hours: int = 24):
    async with httpx.AsyncClient() as client:
        response = await client.get(
            f"{SERVICES['model_serving']}/predictions/forecast",
            params={"location": location, "hours": hours}
        )
        return response.json()

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

## Authentication Middleware

```typescript
// src/middleware/auth.ts
import jwt from "jsonwebtoken";
import { Request, Response, NextFunction } from "express";

export const authMiddleware = (
  req: Request,
  res: Response,
  next: NextFunction
) => {
  const authHeader = req.headers.authorization;

  if (!authHeader || !authHeader.startsWith("Bearer ")) {
    return res
      .status(401)
      .json({ error: "Missing or invalid authorization header" });
  }

  const token = authHeader.substring(7);

  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET!);
    req.user = decoded;
    next();
  } catch (error) {
    return res.status(401).json({ error: "Invalid or expired token" });
  }
};
```

## Rate Limiting

```typescript
// src/middleware/rateLimit.ts
import rateLimit from "express-rate-limit";
import RedisStore from "rate-limit-redis";
import Redis from "ioredis";

const redis = new Redis(process.env.REDIS_URL);

export const publicRateLimiter = rateLimit({
  store: new RedisStore({
    client: redis,
    prefix: "rate-limit:public:",
  }),
  windowMs: 15 * 60 * 1000,
  max: 100,
  message: "Too many requests from this IP, please try again later.",
});

export const authenticatedRateLimiter = rateLimit({
  store: new RedisStore({
    client: redis,
    prefix: "rate-limit:authenticated:",
  }),
  windowMs: 15 * 60 * 1000,
  max: 1000,
  keyGenerator: (req) => req.user?.id || req.ip,
});
```

## Monitoring

### Prometheus Metrics

```typescript
import promClient from "prom-client";

const httpRequestDuration = new promClient.Histogram({
  name: "http_request_duration_seconds",
  help: "Duration of HTTP requests in seconds",
  labelNames: ["method", "route", "status_code"],
});

const httpRequestTotal = new promClient.Counter({
  name: "http_requests_total",
  help: "Total number of HTTP requests",
  labelNames: ["method", "route", "status_code"],
});

app.get("/metrics", async (req, res) => {
  res.set("Content-Type", promClient.register.contentType);
  res.end(await promClient.register.metrics());
});
```

## Docker Deployment

```dockerfile
# Dockerfile
FROM node:18-alpine

WORKDIR /app

COPY package*.json ./
RUN npm ci --only=production

COPY . .
RUN npm run build

EXPOSE 8000

CMD ["node", "dist/index.js"]
```

## Testing

```typescript
// tests/integration/api.test.ts
import request from "supertest";
import app from "../../src/app";

describe("API Gateway", () => {
  describe("GET /health", () => {
    it("should return healthy status", async () => {
      const response = await request(app).get("/health");
      expect(response.status).toBe(200);
      expect(response.body.status).toBe("healthy");
    });
  });

  describe("Public endpoints", () => {
    it("should allow access without authentication", async () => {
      const response = await request(app).get(
        "/api/v1/dashboard/realtime?location=Lagos"
      );
      expect(response.status).toBe(200);
    });
  });

  describe("Protected endpoints", () => {
    it("should reject requests without token", async () => {
      const response = await request(app).get("/api/v1/analytics/historical");
      expect(response.status).toBe(401);
    });
  });
});
```

## Troubleshooting

### Service unavailable errors

- Check backend service health: `curl http://backend-service:port/health`
- Verify service URLs in environment variables
- Check network connectivity

### Rate limiting issues

- Verify Redis connection
- Check rate limit configuration
- Review IP whitelist settings

### CORS errors

- Verify CORS_ORIGIN includes frontend URL
- Check request headers
- Ensure credentials flag is set correctly

## Performance Optimization

1. **Enable caching**: Cache frequent requests in Redis
2. **Connection pooling**: Reuse HTTP connections to backend services
3. **Request compression**: Enable gzip compression
4. **Load balancing**: Deploy multiple instances behind load balancer
5. **Circuit breaker**: Implement circuit breaker pattern for failing services

## Security Best Practices

1. Use HTTPS in production
2. Validate all input data
3. Implement rate limiting
4. Use security headers (helmet)
5. Keep dependencies updated
6. Regular security audits
7. Monitor for suspicious activity

## Dependencies

```json
{
  "dependencies": {
    "express": "^4.18.0",
    "cors": "^2.8.5",
    "helmet": "^7.0.0",
    "http-proxy-middleware": "^2.0.6",
    "express-rate-limit": "^6.10.0",
    "rate-limit-redis": "^3.1.0",
    "jsonwebtoken": "^9.0.0",
    "ioredis": "^5.3.0",
    "winston": "^3.11.0",
    "prom-client": "^15.0.0"
  }
}
```
