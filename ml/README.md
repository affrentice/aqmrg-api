# Machine Learning

This directory contains all ML/AI components for air quality prediction and analysis.

## Directory Structure

### models/

Trained model artifacts organized by purpose:

#### predictive/

Air quality prediction models:

- **4-hour forecasts**: Short-term predictions
- **24-hour forecasts**: Daily predictions
- **72-hour forecasts**: 3-day predictions

Model formats: TensorFlow SavedModel, ONNX, PyTorch

#### correlation/

Health-pollution correlation models:

- Respiratory illness correlation
- Cardiovascular impact analysis
- Demographic-based risk models

#### anomaly-detection/

Data quality and outlier detection:

- Sensor malfunction detection
- Data drift monitoring
- Unusual pollution event detection

### training/

Model training scripts and experiment tracking:

- Training pipelines
- Hyperparameter tuning scripts
- Cross-validation frameworks
- Experiment configurations
- MLflow experiment tracking

### feature-engineering/

Feature extraction and transformation pipelines:

- Time-based features (hour, day, season)
- Meteorological features
- Spatial features
- Lag features
- Rolling statistics

### model-registry/

MLflow model registry configurations:

- Model versioning
- Model staging (dev, staging, production)
- Model metadata and tags
- Model performance metrics

### evaluation/

Model validation and performance testing:

- Validation datasets
- Performance metrics calculation
- A/B testing frameworks
- Model comparison scripts
- Benchmark datasets

## ML Workflow

### 1. Feature Engineering

```bash
cd feature-engineering
python extract_features.py --data-source influxdb --output features.parquet
```

### 2. Model Training

```bash
cd training
python train_predictor.py --config configs/4hr_forecast.yaml
```

### 3. Model Evaluation

```bash
cd evaluation
python evaluate_model.py --model-uri models:/air-quality-4hr/1 --test-data test.parquet
```

### 4. Model Registration

```bash
cd model-registry
python register_model.py --experiment-id 123 --run-id abc123 --model-name air-quality-4hr
```

### 5. Model Deployment

Deploy via model-serving-service API or admin portal

## MLflow Tracking

Access MLflow UI:

```bash
mlflow ui --host 0.0.0.0 --port 5000
```

URL: http://localhost:5000

## Model Performance Metrics

### Prediction Models

- MAE (Mean Absolute Error)
- RMSE (Root Mean Square Error)
- R² Score
- Directional Accuracy

### Classification Models (if applicable)

- Precision, Recall, F1-Score
- ROC-AUC
- Confusion Matrix

## Model Versioning

Models follow semantic versioning:

- **Major**: Breaking changes in input/output format
- **Minor**: Performance improvements, new features
- **Patch**: Bug fixes, minor updates

Example: `air-quality-predictor-v2.1.3`

## Dependencies

```
tensorflow>=2.12.0
pytorch>=2.0.0
scikit-learn>=1.3.0
mlflow>=2.8.0
pandas>=2.0.0
numpy>=1.24.0
```

## Environment Setup

```bash
# Create virtual environment
python -m venv venv
source venv/bin/activate  # or venv\Scripts\activate on Windows

# Install dependencies
pip install -r requirements.txt

# Set MLflow tracking URI
export MLFLOW_TRACKING_URI=http://localhost:5000
```

## Best Practices

1. **Always log experiments** to MLflow
2. **Version all datasets** used for training
3. **Document model assumptions** and limitations
4. **Monitor model performance** in production
5. **Retrain models** when performance degrades
6. **Test models** before deployment

## GPU Support

For GPU-accelerated training:

```bash
# Check GPU availability
python -c "import tensorflow as tf; print(tf.config.list_physical_devices('GPU'))"

# Train with GPU
CUDA_VISIBLE_DEVICES=0 python train_predictor.py
```

## Model Deployment Checklist

- [ ] Model achieves target performance metrics
- [ ] Model validated on hold-out test set
- [ ] Model registered in MLflow registry
- [ ] Model artifacts exported to correct format
- [ ] Model documentation completed
- [ ] API endpoint tested
- [ ] Monitoring alerts configured
- [ ] Rollback plan prepared
