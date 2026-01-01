# Model Serving Service

ML model inference and prediction service for air quality forecasting.

## Overview

The Model Serving Service hosts trained machine learning models and provides prediction endpoints for air quality forecasting. It supports multiple prediction horizons (4-hour, 24-hour, 72-hour) and handles model versioning, A/B testing, and performance monitoring.

## Responsibilities

- **Model Loading**: Load and initialize ML models from registry
- **Prediction API**: Serve real-time air quality predictions
- **Model Versioning**: Support multiple model versions simultaneously
- **A/B Testing**: Route traffic between model versions for comparison
- **Batch Predictions**: Process multiple prediction requests efficiently
- **Feature Engineering**: Transform input data for model consumption
- **Caching**: Cache predictions to reduce compute costs
- **Monitoring**: Track model performance and drift
- **Health Checks**: Monitor model availability and latency

## Technology Stack

**Language**: Python (required for ML frameworks)  
**Framework**: FastAPI  
**ML Frameworks**: TensorFlow, PyTorch, scikit-learn  
**Model Registry**: MLflow  
**Dependencies**:

- `fastapi` - Web framework
- `uvicorn` - ASGI server
- `tensorflow` or `torch` - ML frameworks
- `mlflow` - Model registry and tracking
- `pandas` - Data manipulation
- `numpy` - Numerical operations
- `redis` - Prediction caching
- `pydantic` - Data validation
- `prometheus-client` - Metrics

## Port

**Default**: `8003`

## Prediction Models

### Available Models

1. **4-Hour Forecast** (`prediction_4hr`)

   - Short-term predictions
   - High accuracy for immediate forecasts
   - Updated every 15 minutes

2. **24-Hour Forecast** (`prediction_24hr`)

   - Daily predictions
   - Includes weather integration
   - Updated hourly

3. **72-Hour Forecast** (`prediction_72hr`)

   - 3-day predictions
   - Trend analysis
   - Updated every 3 hours

4. **Anomaly Detection** (`anomaly_detection`)

   - Real-time anomaly scoring
   - Identifies unusual pollution events

5. **Health Correlation** (`health_correlation`)
   - Predict health impact scores
   - Demographic-based risk assessment

## API Endpoints

### Prediction Endpoints

```
POST /predictions/forecast        - Get air quality predictions
POST /predictions/batch           - Batch prediction requests
GET  /predictions/forecast/:location/:hours - Simple GET prediction
POST /predictions/anomaly         - Detect anomalies
POST /predictions/health-impact   - Health impact predictions
```

### Model Management (Admin)

```
GET  /models                      - List deployed models
GET  /models/:name                - Get model details
POST /models/deploy               - Deploy new model version
PUT  /models/:name/version        - Switch active version
DELETE /models/:name              - Remove model
POST /models/:name/reload         - Reload model
```

### Monitoring

```
GET  /health                      - Health check
GET  /metrics                     - Prometheus metrics
GET  /models/stats                - Model performance stats
```

## Directory Structure

```
model-serving-service/
├── src/
│   ├── main.py                     # FastAPI application
│   ├── config/
│   │   └── settings.py            # Configuration
│   ├── models/
│   │   ├── base_model.py          # Base model interface
│   │   ├── prediction_4hr.py      # 4-hour forecast model
│   │   ├── prediction_24hr.py     # 24-hour forecast model
│   │   ├── prediction_72hr.py     # 72-hour forecast model
│   │   └── model_loader.py        # Model loading logic
│   ├── services/
│   │   ├── prediction_service.py  # Prediction logic
│   │   ├── feature_service.py     # Feature engineering
│   │   ├── cache_service.py       # Prediction caching
│   │   └── mlflow_service.py      # MLflow integration
│   ├── schemas/
│   │   ├── prediction_request.py  # Request schemas
│   │   └── prediction_response.py # Response schemas
│   ├── routers/
│   │   ├── predictions.py         # Prediction routes
│   │   └── models.py              # Model management routes
│   └── utils/
│       ├── logger.py              # Logging
│       └── metrics.py             # Prometheus metrics
├── models/                         # Saved model artifacts
│   ├── prediction_4hr/
│   ├── prediction_24hr/
│   └── prediction_72hr/
├── tests/
│   ├── unit/
│   │   ├── test_models.py
│   │   └── test_features.py
│   └── integration/
│       └── test_predictions.py
├── requirements.txt
├── Dockerfile
└── README.md
```

## Request/Response Schema

### Prediction Request

```python
from pydantic import BaseModel
from typing import List, Optional

class PredictionRequest(BaseModel):
    location: Location
    hours: int = 24  # Prediction horizon (4, 24, or 72)
    include_confidence: bool = True
    model_version: Optional[str] = None  # Force specific version

class Location(BaseModel):
    lat: float
    lon: float
    name: Optional[str] = None
```

### Prediction Response

```python
class PredictionResponse(BaseModel):
    location: Location
    predictions: List[Prediction]
    model_version: str
    generated_at: str

class Prediction(BaseModel):
    timestamp: str
    pm25: float
    pm10: float
    aqi: int
    confidence: Optional[float] = None
    health_advisory: HealthAdvisory

class HealthAdvisory(BaseModel):
    level: str  # "Good", "Moderate", "Unhealthy", etc.
    message: str
    sensitive_groups: List[str]
```

## Configuration

### Environment Variables

```bash
# Service
MODEL_SERVING_SERVICE_PORT=8003

# MLflow
MLFLOW_TRACKING_URI=http://localhost:5000
MLFLOW_REGISTRY_URI=http://localhost:5000

# Model Configuration
MODEL_PREDICTION_4HR_VERSION=latest
MODEL_PREDICTION_24HR_VERSION=latest
MODEL_PREDICTION_72HR_VERSION=latest

# GPU Configuration
CUDA_VISIBLE_DEVICES=0
TF_FORCE_GPU_ALLOW_GROWTH=true

# Caching
REDIS_URL=redis://localhost:6379
CACHE_TTL_PREDICTIONS=300  # 5 minutes

# Performance
MODEL_BATCH_SIZE=32
MODEL_WORKERS=2
PREDICTION_TIMEOUT=30

# Monitoring
ENABLE_PERFORMANCE_LOGGING=true
LOG_LEVEL=info
```

## Setup & Installation

```bash
# Create virtual environment
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Download models from MLflow (if not using mounted volume)
python scripts/download_models.py

# Development
uvicorn main:app --reload --host 0.0.0.0 --port 8003

# Production with multiple workers
gunicorn -w 4 -k uvicorn.workers.UvicornWorker main:app --bind 0.0.0.0:8003

# Tests
pytest
```

## Example Implementation

### Main Application

```python
# src/main.py
from fastapi import FastAPI, HTTPException
from fastapi.middleware.cors import CORSMiddleware
from prometheus_fastapi_instrumentator import Instrumentator
import logging

from routers import predictions, models
from services.prediction_service import PredictionService
from services.mlflow_service import MLFlowService

# Initialize FastAPI
app = FastAPI(
    title="AQMRG Model Serving Service",
    version="1.0.0",
    description="Air quality prediction API"
)

# CORS
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# Initialize services
mlflow_service = MLFlowService()
prediction_service = PredictionService(mlflow_service)

# Include routers
app.include_router(predictions.router, prefix="/predictions", tags=["Predictions"])
app.include_router(models.router, prefix="/models", tags=["Models"])

# Prometheus metrics
Instrumentator().instrument(app).expose(app)

@app.on_event("startup")
async def startup_event():
    """Load models on startup"""
    logging.info("Loading ML models...")
    await prediction_service.load_models()
    logging.info("Models loaded successfully")

@app.get("/health")
async def health_check():
    """Health check endpoint"""
    model_health = prediction_service.get_models_health()
    return {
        "status": "healthy",
        "models": model_health
    }

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8003)
```

### Model Loading Service

```python
# src/services/mlflow_service.py
import mlflow
from mlflow.tracking import MlflowClient
import os
import logging

class MLFlowService:
    def __init__(self):
        self.tracking_uri = os.getenv("MLFLOW_TRACKING_URI")
        mlflow.set_tracking_uri(self.tracking_uri)
        self.client = MlflowClient()

    def load_model(self, model_name: str, version: str = "latest"):
        """Load model from MLflow registry"""
        try:
            if version == "latest":
                # Get latest production version
                versions = self.client.get_latest_versions(
                    model_name,
                    stages=["Production"]
                )
                if not versions:
                    versions = self.client.get_latest_versions(model_name)

                if not versions:
                    raise ValueError(f"No versions found for model {model_name}")

                version = versions[0].version

            model_uri = f"models:/{model_name}/{version}"
            logging.info(f"Loading model from {model_uri}")

            model = mlflow.pyfunc.load_model(model_uri)
            return model, version
        except Exception as e:
            logging.error(f"Error loading model {model_name}: {str(e)}")
            raise

    def get_model_metadata(self, model_name: str, version: str):
        """Get model metadata from MLflow"""
        try:
            model_version = self.client.get_model_version(model_name, version)
            return {
                "name": model_name,
                "version": version,
                "stage": model_version.current_stage,
                "created_at": model_version.creation_timestamp,
                "description": model_version.description,
                "tags": model_version.tags
            }
        except Exception as e:
            logging.error(f"Error getting metadata: {str(e)}")
            return None
```

### Prediction Service

```python
# src/services/prediction_service.py
from typing import Dict, List, Optional
import numpy as np
import pandas as pd
from datetime import datetime, timedelta
import logging

class PredictionService:
    def __init__(self, mlflow_service: MLFlowService):
        self.mlflow_service = mlflow_service
        self.models: Dict[str, any] = {}
        self.model_versions: Dict[str, str] = {}
        self.feature_service = FeatureService()
        self.cache_service = CacheService()

    async def load_models(self):
        """Load all prediction models"""
        model_configs = [
            ("prediction_4hr", os.getenv("MODEL_PREDICTION_4HR_VERSION", "latest")),
            ("prediction_24hr", os.getenv("MODEL_PREDICTION_24HR_VERSION", "latest")),
            ("prediction_72hr", os.getenv("MODEL_PREDICTION_72HR_VERSION", "latest"))
        ]

        for model_name, version in model_configs:
            try:
                model, loaded_version = self.mlflow_service.load_model(
                    model_name,
                    version
                )
                self.models[model_name] = model
                self.model_versions[model_name] = loaded_version
                logging.info(f"Loaded {model_name} version {loaded_version}")
            except Exception as e:
                logging.error(f"Failed to load {model_name}: {str(e)}")

    async def predict(self, request: PredictionRequest) -> PredictionResponse:
        """Generate air quality predictions"""

        # Check cache first
        cache_key = self.cache_service.generate_key(request)
        cached = await self.cache_service.get(cache_key)
        if cached:
            logging.info("Returning cached prediction")
            return cached

        # Select appropriate model based on horizon
        model_name = self._select_model(request.hours)

        if model_name not in self.models:
            raise HTTPException(
                status_code=503,
                detail=f"Model {model_name} not available"
            )

        # Prepare features
        features = await self.feature_service.prepare_features(request)

        # Make prediction
        model = self.models[model_name]
        predictions = model.predict(features)

        # Post-process predictions
        response = self._format_response(
            predictions,
            request,
            model_name,
            self.model_versions[model_name]
        )

        # Cache result
        await self.cache_service.set(cache_key, response, ttl=300)

        return response

    def _select_model(self, hours: int) -> str:
        """Select appropriate model based on prediction horizon"""
        if hours <= 4:
            return "prediction_4hr"
        elif hours <= 24:
            return "prediction_24hr"
        else:
            return "prediction_72hr"

    def _format_response(
        self,
        predictions: np.ndarray,
        request: PredictionRequest,
        model_name: str,
        version: str
    ) -> PredictionResponse:
        """Format model output into API response"""

        predictions_list = []
        current_time = datetime.utcnow()

        for i, pred in enumerate(predictions):
            timestamp = current_time + timedelta(hours=i+1)

            pm25 = float(pred[0])
            pm10 = float(pred[1])
            aqi = self._calculate_aqi(pm25, pm10)

            predictions_list.append(Prediction(
                timestamp=timestamp.isoformat(),
                pm25=pm25,
                pm10=pm10,
                aqi=aqi,
                confidence=float(pred[2]) if len(pred) > 2 else None,
                health_advisory=self._get_health_advisory(aqi)
            ))

        return PredictionResponse(
            location=request.location,
            predictions=predictions_list[:request.hours],
            model_version=f"{model_name}-{version}",
            generated_at=current_time.isoformat()
        )

    def _calculate_aqi(self, pm25: float, pm10: float) -> int:
        """Calculate Air Quality Index from pollutant concentrations"""
        # Simplified AQI calculation
        # In production, use official AQI calculation formulas

        aqi_pm25 = self._pm25_to_aqi(pm25)
        aqi_pm10 = self._pm10_to_aqi(pm10)

        return max(aqi_pm25, aqi_pm10)

    def _pm25_to_aqi(self, pm25: float) -> int:
        """Convert PM2.5 to AQI"""
        if pm25 <= 12.0:
            return int((50 / 12.0) * pm25)
        elif pm25 <= 35.4:
            return int(50 + ((100 - 50) / (35.4 - 12.1)) * (pm25 - 12.1))
        elif pm25 <= 55.4:
            return int(100 + ((150 - 100) / (55.4 - 35.5)) * (pm25 - 35.5))
        elif pm25 <= 150.4:
            return int(150 + ((200 - 150) / (150.4 - 55.5)) * (pm25 - 55.5))
        else:
            return min(int(200 + ((300 - 200) / (250.4 - 150.5)) * (pm25 - 150.5)), 500)

    def _get_health_advisory(self, aqi: int) -> HealthAdvisory:
        """Get health advisory based on AQI"""
        if aqi <= 50:
            return HealthAdvisory(
                level="Good",
                message="Air quality is satisfactory",
                sensitive_groups=[]
            )
        elif aqi <= 100:
            return HealthAdvisory(
                level="Moderate",
                message="Acceptable for most, but sensitive individuals may experience issues",
                sensitive_groups=["Unusually sensitive people"]
            )
        elif aqi <= 150:
            return HealthAdvisory(
                level="Unhealthy for Sensitive Groups",
                message="Sensitive groups may experience health effects",
                sensitive_groups=["Children", "Elderly", "People with respiratory conditions"]
            )
        elif aqi <= 200:
            return HealthAdvisory(
                level="Unhealthy",
                message="Everyone may begin to experience health effects",
                sensitive_groups=["Everyone"]
            )
        else:
            return HealthAdvisory(
                level="Very Unhealthy",
                message="Health warnings of emergency conditions",
                sensitive_groups=["Everyone"]
            )
```

### Feature Engineering Service

```python
# src/services/feature_service.py
import pandas as pd
import numpy as np
from datetime import datetime
import requests

class FeatureService:
    async def prepare_features(self, request: PredictionRequest) -> pd.DataFrame:
        """Prepare features for model input"""

        # Get current sensor data
        current_data = await self._get_current_sensor_data(request.location)

        # Get weather data
        weather_data = await self._get_weather_data(request.location)

        # Create time-based features
        time_features = self._create_time_features()

        # Combine all features
        features = pd.DataFrame({
            **current_data,
            **weather_data,
            **time_features
        }, index=[0])

        return features

    async def _get_current_sensor_data(self, location: Location) -> dict:
        """Fetch current sensor readings near location"""
        # In production, query InfluxDB or cache
        return {
            'current_pm25': 35.2,
            'current_pm10': 45.8,
            'pm25_avg_24h': 38.1,
            'pm10_avg_24h': 47.2
        }

    async def _get_weather_data(self, location: Location) -> dict:
        """Fetch weather forecast"""
        # In production, integrate with weather API
        return {
            'temperature': 28.5,
            'humidity': 65.0,
            'wind_speed': 3.2,
            'wind_direction': 180
        }

    def _create_time_features(self) -> dict:
        """Create time-based features"""
        now = datetime.utcnow()
        return {
            'hour': now.hour,
            'day_of_week': now.weekday(),
            'month': now.month,
            'is_weekend': int(now.weekday() >= 5)
        }
```

### Prediction Caching

```python
# src/services/cache_service.py
import redis
import json
import hashlib

class CacheService:
    def __init__(self):
        self.redis = redis.from_url(os.getenv("REDIS_URL"))
        self.ttl = int(os.getenv("CACHE_TTL_PREDICTIONS", 300))

    def generate_key(self, request: PredictionRequest) -> str:
        """Generate cache key from request"""
        key_data = f"{request.location.lat}_{request.location.lon}_{request.hours}"
        return f"prediction:{hashlib.md5(key_data.encode()).hexdigest()}"

    async def get(self, key: str) -> Optional[PredictionResponse]:
        """Get cached prediction"""
        try:
            cached = self.redis.get(key)
            if cached:
                return PredictionResponse(**json.loads(cached))
        except Exception as e:
            logging.error(f"Cache get error: {str(e)}")
        return None

    async def set(self, key: str, value: PredictionResponse, ttl: int = None):
        """Cache prediction"""
        try:
            self.redis.setex(
                key,
                ttl or self.ttl,
                json.dumps(value.dict())
            )
        except Exception as e:
            logging.error(f"Cache set error: {str(e)}")
```

## Monitoring

```python
# src/utils/metrics.py
from prometheus_client import Counter, Histogram

predictions_total = Counter(
    'predictions_total',
    'Total number of predictions made',
    ['model', 'status']
)

prediction_duration = Histogram(
    'prediction_duration_seconds',
    'Time spent generating predictions',
    ['model']
)

model_errors = Counter(
    'model_errors_total',
    'Total number of model errors',
    ['model', 'error_type']
)
```

## Testing

```python
# tests/integration/test_predictions.py
import pytest
from fastapi.testclient import TestClient
from main import app

client = TestClient(app)

def test_forecast_prediction():
    response = client.post("/predictions/forecast", json={
        "location": {"lat": 6.5244, "lon": 3.3792},
        "hours": 24
    })

    assert response.status_code == 200
    data = response.json()
    assert "predictions" in data
    assert len(data["predictions"]) == 24
    assert data["predictions"][0]["pm25"] > 0

def test_invalid_location():
    response = client.post("/predictions/forecast", json={
        "location": {"lat": 200, "lon": 300},  # Invalid
        "hours": 24
    })

    assert response.status_code == 422
```

## Dependencies

```txt
fastapi==0.104.0
uvicorn[standard]==0.24.0
pydantic==2.4.2
tensorflow==2.14.0
# OR
torch==2.1.0
mlflow==2.8.0
pandas==2.1.1
numpy==1.24.3
redis==5.0.1
prometheus-client==0.18.0
scikit-learn==1.3.0
requests==2.31.0
python-multipart==0.0.6
```
