# Shared Code

Common code, utilities, and configurations shared across all microservices.

## Directory Structure

### proto/

Protocol buffer definitions (if using gRPC):

- Service definitions
- Message schemas
- Shared data contracts

### types/

Shared type definitions:

- TypeScript interfaces (for Node.js services)
- Python type hints and Pydantic models
- Common data models

### utils/

Common utility functions:

- Date/time helpers
- Data validation utilities
- Formatting functions
- Conversion utilities

### config/

Shared configuration schemas:

- Environment variable definitions
- Configuration validation
- Default values
- Feature flags schema

### constants/

Application-wide constants:

- API response codes
- Error messages
- Status codes
- Enum definitions

### middleware/

Shared middleware for services:

- Authentication middleware
- Logging middleware
- Error handling middleware
- Request validation
- CORS configuration

### validators/

Request/response validation schemas:

- Input validation
- Output validation
- JSON schema definitions
- Data sanitization

## Usage Examples

### Using Shared Types (TypeScript)

```typescript
import { SensorReading, AirQualityPrediction } from "@shared/types";

const reading: SensorReading = {
  sensorId: "ABC123",
  timestamp: new Date(),
  pm25: 35.2,
  location: { lat: 6.5244, lon: 3.3792 },
};
```

### Using Shared Utils (Python)

```python
from shared.utils.date_helpers import parse_iso_datetime
from shared.utils.validators import validate_sensor_id

timestamp = parse_iso_datetime('2026-01-01T12:00:00Z')
is_valid = validate_sensor_id('ABC123')
```

### Using Shared Middleware (Express)

```typescript
import { authMiddleware, loggingMiddleware } from "@shared/middleware";

app.use(loggingMiddleware);
app.use("/api/protected", authMiddleware);
```

### Using Shared Constants

```typescript
import { HTTP_STATUS, ERROR_CODES } from "@shared/constants";

res.status(HTTP_STATUS.BAD_REQUEST).json({
  error: ERROR_CODES.INVALID_SENSOR_DATA,
  message: "Invalid sensor reading format",
});
```

## Development

### Adding New Shared Code

1. Determine appropriate directory
2. Create well-documented, reusable code
3. Add comprehensive tests
4. Update this README
5. Version bump if breaking changes

### TypeScript Shared Package

```json
{
  "name": "@aqmrg/shared",
  "version": "1.0.0",
  "main": "dist/index.js",
  "types": "dist/index.d.ts"
}
```

### Python Shared Package

```python
# setup.py
from setuptools import setup, find_packages

setup(
    name='aqmrg-shared',
    version='1.0.0',
    packages=find_packages(),
    install_requires=[
        'pydantic>=2.0.0',
        'python-dotenv>=1.0.0'
    ]
)
```

## Best Practices

1. **Keep it DRY**: Avoid code duplication across services
2. **Version carefully**: Breaking changes affect all services
3. **Document thoroughly**: Clear docs for all shared code
4. **Test extensively**: High test coverage required
5. **Maintain backward compatibility** when possible

## Testing

```bash
# TypeScript
npm test

# Python
pytest tests/
```
