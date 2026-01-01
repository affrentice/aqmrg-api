# Auth Service

Authentication and authorization service for the AQMRG AI Analytics Platform.

## Overview

The Auth Service manages user authentication, authorization, and session management. It provides JWT-based authentication with refresh token support and role-based access control (RBAC).

## Responsibilities

- **User Registration**: Create new user accounts
- **User Login**: Authenticate users and issue tokens
- **Token Management**: Generate, validate, and refresh JWT tokens
- **Password Management**: Hashing, validation, and reset
- **Role-Based Access Control**: Manage user roles and permissions
- **Session Management**: Track active user sessions
- **Multi-Factor Authentication**: Optional 2FA support
- **Audit Logging**: Track authentication events
- **OAuth Integration**: Support for third-party authentication (optional)

## Technology Stack

**Language**: Node.js (TypeScript) or Python (FastAPI)  
**Framework**: Express.js or FastAPI  
**Database**: PostgreSQL  
**Dependencies**:

- `bcrypt` or `bcryptjs` - Password hashing
- `jsonwebtoken` - JWT generation/validation
- `express-validator` or `pydantic` - Input validation
- `pg` or `psycopg2` - PostgreSQL client
- `redis` - Session and token blacklist storage
- `nodemailer` or `sendgrid` - Email for password reset
- `speakeasy` - TOTP for 2FA (optional)

## Port

**Default**: `8001`

## User Roles

- **Public**: Unauthenticated users (read-only public data)
- **Authenticated**: Logged-in users (advanced features, data export)
- **Admin**: System administrators (model deployment, user management)

## API Endpoints

### Authentication

```
POST /auth/register          - Register new user
POST /auth/login             - User login (returns access + refresh tokens)
POST /auth/logout            - User logout (blacklist token)
POST /auth/refresh           - Refresh access token
POST /auth/verify-email      - Verify email address
```

### User Management

```
GET  /auth/me                - Get current user profile
PUT  /auth/me                - Update user profile
PUT  /auth/me/password       - Change password
DELETE /auth/me              - Delete account
```

### Password Reset

```
POST /auth/password/reset-request   - Request password reset email
POST /auth/password/reset           - Reset password with token
```

### Multi-Factor Authentication (Optional)

```
POST /auth/mfa/setup         - Setup 2FA
POST /auth/mfa/verify        - Verify 2FA code
POST /auth/mfa/disable       - Disable 2FA
```

### Admin Only

```
GET  /auth/users             - List all users
GET  /auth/users/:id         - Get user by ID
PUT  /auth/users/:id         - Update user
DELETE /auth/users/:id       - Delete user
PUT  /auth/users/:id/role    - Update user role
```

## Directory Structure

```
auth-service/
├── src/
│   ├── index.ts                 # Application entry point
│   ├── app.ts                   # Express/FastAPI app setup
│   ├── config/
│   │   └── database.ts         # Database configuration
│   ├── models/
│   │   ├── User.ts             # User model
│   │   └── Session.ts          # Session model
│   ├── controllers/
│   │   ├── authController.ts   # Authentication logic
│   │   ├── userController.ts   # User management
│   │   └── adminController.ts  # Admin operations
│   ├── middleware/
│   │   ├── auth.ts             # JWT validation middleware
│   │   ├── validateRole.ts     # Role validation
│   │   └── validation.ts       # Input validation
│   ├── services/
│   │   ├── tokenService.ts     # JWT token operations
│   │   ├── passwordService.ts  # Password hashing/validation
│   │   ├── emailService.ts     # Email notifications
│   │   └── mfaService.ts       # 2FA operations
│   ├── routes/
│   │   ├── auth.ts             # Auth routes
│   │   ├── user.ts             # User routes
│   │   └── admin.ts            # Admin routes
│   └── utils/
│       ├── logger.ts           # Winston logger
│       └── validators.ts       # Custom validators
├── tests/
│   ├── unit/
│   │   ├── tokenService.test.ts
│   │   └── passwordService.test.ts
│   └── integration/
│       ├── auth.test.ts
│       └── user.test.ts
├── migrations/
│   └── 001_create_users_table.sql
├── .env.example                # Environment variables
├── Dockerfile                  # Docker container
├── package.json                # Dependencies
└── README.md                   # This file
```

## Database Schema

### Users Table

```sql
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    role VARCHAR(50) NOT NULL DEFAULT 'authenticated',
    email_verified BOOLEAN DEFAULT FALSE,
    mfa_enabled BOOLEAN DEFAULT FALSE,
    mfa_secret VARCHAR(255),
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW(),
    last_login_at TIMESTAMP,
    CONSTRAINT valid_role CHECK (role IN ('public', 'authenticated', 'admin'))
);

CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_role ON users(role);
```

### Sessions Table (optional - can use Redis instead)

```sql
CREATE TABLE sessions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    refresh_token VARCHAR(500) NOT NULL,
    expires_at TIMESTAMP NOT NULL,
    created_at TIMESTAMP DEFAULT NOW(),
    ip_address VARCHAR(45),
    user_agent TEXT
);

CREATE INDEX idx_sessions_user_id ON sessions(user_id);
CREATE INDEX idx_sessions_refresh_token ON sessions(refresh_token);
```

### Password Reset Tokens Table

```sql
CREATE TABLE password_reset_tokens (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    token VARCHAR(255) UNIQUE NOT NULL,
    expires_at TIMESTAMP NOT NULL,
    created_at TIMESTAMP DEFAULT NOW(),
    used BOOLEAN DEFAULT FALSE
);

CREATE INDEX idx_password_reset_tokens_token ON password_reset_tokens(token);
```

## Configuration

### Environment Variables

```bash
# Service
AUTH_SERVICE_PORT=8001

# Database
DATABASE_URL=postgresql://aqmrg:password@localhost:5432/aqmrg

# JWT Configuration
JWT_SECRET=your_jwt_secret_minimum_32_characters
JWT_REFRESH_SECRET=your_jwt_refresh_secret_minimum_32_characters
JWT_ALGORITHM=HS256
JWT_EXPIRES_IN=1h
JWT_REFRESH_EXPIRES_IN=7d

# Password Policy
PASSWORD_MIN_LENGTH=8
PASSWORD_REQUIRE_UPPERCASE=true
PASSWORD_REQUIRE_LOWERCASE=true
PASSWORD_REQUIRE_NUMBERS=true
PASSWORD_REQUIRE_SPECIAL_CHARS=true
BCRYPT_ROUNDS=10

# Email Configuration
EMAIL_HOST=smtp.gmail.com
EMAIL_PORT=587
EMAIL_USER=your_email@gmail.com
EMAIL_PASSWORD=your_app_password
EMAIL_FROM=noreply@aqmrg.org

# Redis (for token blacklist and sessions)
REDIS_URL=redis://localhost:6379

# Multi-Factor Authentication (Optional)
MFA_ENABLED=false
MFA_ISSUER=AQMRG
```

## Setup & Installation

### Node.js/TypeScript

```bash
# Install dependencies
npm install

# Run migrations
npm run migrate

# Development
npm run dev

# Build
npm run build

# Production
npm start

# Tests
npm test
```

### Python/FastAPI

```bash
# Create virtual environment
python -m venv venv
source venv/bin/activate

# Install dependencies
pip install -r requirements.txt

# Run migrations
alembic upgrade head

# Development
uvicorn main:app --reload --port 8001

# Production
gunicorn -w 4 -k uvicorn.workers.UvicornWorker main:app --bind 0.0.0.0:8001

# Tests
pytest
```

## Example Implementation

### User Registration

```typescript
// src/controllers/authController.ts
import bcrypt from "bcrypt";
import { pool } from "../config/database";
import { generateTokens } from "../services/tokenService";

export async function register(req, res) {
  const { email, password } = req.body;

  // Validate password strength
  if (!isPasswordStrong(password)) {
    return res.status(400).json({
      error: "Password does not meet requirements",
    });
  }

  try {
    // Check if user already exists
    const existingUser = await pool.query(
      "SELECT id FROM users WHERE email = $1",
      [email]
    );

    if (existingUser.rows.length > 0) {
      return res.status(409).json({ error: "Email already registered" });
    }

    // Hash password
    const passwordHash = await bcrypt.hash(
      password,
      parseInt(process.env.BCRYPT_ROUNDS || "10")
    );

    // Create user
    const result = await pool.query(
      `INSERT INTO users (email, password_hash, role) 
       VALUES ($1, $2, 'authenticated') 
       RETURNING id, email, role, created_at`,
      [email, passwordHash]
    );

    const user = result.rows[0];

    // Generate tokens
    const { accessToken, refreshToken } = generateTokens(user);

    // Send verification email (optional)
    await sendVerificationEmail(user.email);

    res.status(201).json({
      user: {
        id: user.id,
        email: user.email,
        role: user.role,
      },
      accessToken,
      refreshToken,
    });
  } catch (error) {
    console.error("Registration error:", error);
    res.status(500).json({ error: "Registration failed" });
  }
}
```

### User Login

```typescript
// src/controllers/authController.ts
export async function login(req, res) {
  const { email, password } = req.body;

  try {
    // Find user
    const result = await pool.query(
      `SELECT id, email, password_hash, role, mfa_enabled 
       FROM users WHERE email = $1`,
      [email]
    );

    if (result.rows.length === 0) {
      return res.status(401).json({ error: "Invalid credentials" });
    }

    const user = result.rows[0];

    // Verify password
    const isValid = await bcrypt.compare(password, user.password_hash);
    if (!isValid) {
      return res.status(401).json({ error: "Invalid credentials" });
    }

    // Check if MFA is enabled
    if (user.mfa_enabled) {
      // Return temporary token requiring MFA verification
      const mfaToken = generateMFAToken(user.id);
      return res.json({
        requiresMFA: true,
        mfaToken,
      });
    }

    // Generate tokens
    const { accessToken, refreshToken } = generateTokens(user);

    // Update last login
    await pool.query("UPDATE users SET last_login_at = NOW() WHERE id = $1", [
      user.id,
    ]);

    res.json({
      user: {
        id: user.id,
        email: user.email,
        role: user.role,
      },
      accessToken,
      refreshToken,
      expiresIn: 3600,
    });
  } catch (error) {
    console.error("Login error:", error);
    res.status(500).json({ error: "Login failed" });
  }
}
```

### Token Service

```typescript
// src/services/tokenService.ts
import jwt from "jsonwebtoken";

export function generateTokens(user: any) {
  const payload = {
    id: user.id,
    email: user.email,
    role: user.role,
  };

  const accessToken = jwt.sign(payload, process.env.JWT_SECRET!, {
    expiresIn: process.env.JWT_EXPIRES_IN || "1h",
    algorithm: "HS256",
  });

  const refreshToken = jwt.sign(
    { id: user.id },
    process.env.JWT_REFRESH_SECRET!,
    {
      expiresIn: process.env.JWT_REFRESH_EXPIRES_IN || "7d",
      algorithm: "HS256",
    }
  );

  return { accessToken, refreshToken };
}

export function verifyAccessToken(token: string) {
  try {
    return jwt.verify(token, process.env.JWT_SECRET!);
  } catch (error) {
    throw new Error("Invalid or expired token");
  }
}

export function verifyRefreshToken(token: string) {
  try {
    return jwt.verify(token, process.env.JWT_REFRESH_SECRET!);
  } catch (error) {
    throw new Error("Invalid or expired refresh token");
  }
}
```

### Password Reset

```typescript
// src/controllers/authController.ts
import crypto from "crypto";

export async function requestPasswordReset(req, res) {
  const { email } = req.body;

  try {
    const result = await pool.query("SELECT id FROM users WHERE email = $1", [
      email,
    ]);

    if (result.rows.length === 0) {
      // Don't reveal if email exists
      return res.json({
        message: "If the email exists, a reset link has been sent",
      });
    }

    const user = result.rows[0];

    // Generate reset token
    const resetToken = crypto.randomBytes(32).toString("hex");
    const expiresAt = new Date(Date.now() + 3600000); // 1 hour

    // Store reset token
    await pool.query(
      `INSERT INTO password_reset_tokens (user_id, token, expires_at)
       VALUES ($1, $2, $3)`,
      [user.id, resetToken, expiresAt]
    );

    // Send reset email
    await sendPasswordResetEmail(email, resetToken);

    res.json({
      message: "If the email exists, a reset link has been sent",
    });
  } catch (error) {
    console.error("Password reset request error:", error);
    res.status(500).json({ error: "Request failed" });
  }
}

export async function resetPassword(req, res) {
  const { token, newPassword } = req.body;

  try {
    // Find valid token
    const result = await pool.query(
      `SELECT user_id FROM password_reset_tokens 
       WHERE token = $1 AND expires_at > NOW() AND used = FALSE`,
      [token]
    );

    if (result.rows.length === 0) {
      return res.status(400).json({ error: "Invalid or expired token" });
    }

    const userId = result.rows[0].user_id;

    // Hash new password
    const passwordHash = await bcrypt.hash(newPassword, 10);

    // Update password
    await pool.query("UPDATE users SET password_hash = $1 WHERE id = $2", [
      passwordHash,
      userId,
    ]);

    // Mark token as used
    await pool.query(
      "UPDATE password_reset_tokens SET used = TRUE WHERE token = $1",
      [token]
    );

    res.json({ message: "Password reset successful" });
  } catch (error) {
    console.error("Password reset error:", error);
    res.status(500).json({ error: "Password reset failed" });
  }
}
```

### Role-Based Access Control Middleware

```typescript
// src/middleware/validateRole.ts
export function requireRole(...allowedRoles: string[]) {
  return (req, res, next) => {
    if (!req.user) {
      return res.status(401).json({ error: "Authentication required" });
    }

    if (!allowedRoles.includes(req.user.role)) {
      return res.status(403).json({
        error: "Insufficient permissions",
      });
    }

    next();
  };
}

// Usage in routes
app.get("/admin/users", authMiddleware, requireRole("admin"), listUsers);
```

## Token Blacklisting (Logout)

```typescript
// src/services/tokenService.ts
import Redis from "ioredis";

const redis = new Redis(process.env.REDIS_URL);

export async function blacklistToken(token: string) {
  const decoded: any = jwt.decode(token);
  if (!decoded || !decoded.exp) {
    throw new Error("Invalid token");
  }

  // Calculate TTL (time until token expires)
  const ttl = decoded.exp - Math.floor(Date.now() / 1000);

  if (ttl > 0) {
    await redis.setex(`blacklist:${token}`, ttl, "1");
  }
}

export async function isTokenBlacklisted(token: string): Promise<boolean> {
  const result = await redis.get(`blacklist:${token}`);
  return result !== null;
}
```

## Testing

```typescript
// tests/integration/auth.test.ts
import request from "supertest";
import app from "../../src/app";

describe("Auth Service", () => {
  describe("POST /auth/register", () => {
    it("should register a new user", async () => {
      const response = await request(app).post("/auth/register").send({
        email: "test@example.com",
        password: "SecurePass123!",
      });

      expect(response.status).toBe(201);
      expect(response.body).toHaveProperty("accessToken");
      expect(response.body).toHaveProperty("refreshToken");
      expect(response.body.user.email).toBe("test@example.com");
    });

    it("should reject weak passwords", async () => {
      const response = await request(app).post("/auth/register").send({
        email: "test@example.com",
        password: "123",
      });

      expect(response.status).toBe(400);
    });
  });

  describe("POST /auth/login", () => {
    it("should login with valid credentials", async () => {
      const response = await request(app).post("/auth/login").send({
        email: "test@example.com",
        password: "SecurePass123!",
      });

      expect(response.status).toBe(200);
      expect(response.body).toHaveProperty("accessToken");
    });
  });
});
```

## Security Best Practices

1. **Password Hashing**: Use bcrypt with 10+ rounds
2. **Token Security**: Use strong secrets (32+ characters)
3. **HTTPS Only**: Never send tokens over HTTP
4. **Token Expiration**: Short-lived access tokens (1 hour)
5. **Rate Limiting**: Prevent brute force attacks
6. **Input Validation**: Validate all user inputs
7. **SQL Injection**: Use parameterized queries
8. **Account Lockout**: Lock accounts after failed attempts
9. **Audit Logging**: Log all authentication events

## Dependencies

```json
{
  "dependencies": {
    "express": "^4.18.0",
    "bcrypt": "^5.1.0",
    "jsonwebtoken": "^9.0.0",
    "pg": "^8.11.0",
    "ioredis": "^5.3.0",
    "express-validator": "^7.0.0",
    "nodemailer": "^6.9.0",
    "speakeasy": "^2.0.0"
  }
}
```
