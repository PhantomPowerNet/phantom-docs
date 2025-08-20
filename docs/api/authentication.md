# Authentication API

## Overview

The Phantom Power authentication system uses JWT (JSON Web Tokens) for stateless authentication. All authenticated endpoints require a valid JWT token in the Authorization header.

## Base URL

```
Development: http://localhost:8080/api/auth
Production: https://api.phantompower.app/api/auth
```

## Authentication Flow

1. User registers or logs in with credentials
2. Server validates credentials and returns JWT token
3. Client stores token and includes it in subsequent requests
4. Server validates token on each protected endpoint access

## Endpoints

### Register User

Create a new user account.

**Endpoint:** `POST /register`

**Request Body:**
```json
{
  "email": "user@example.com",
  "password": "securePassword123",
  "confirmPassword": "securePassword123"
}
```

**Response (201 Created):**
```json
{
  "success": true,
  "message": "User registered successfully",
  "data": {
    "user": {
      "id": "01HYN8FXZ2W6X8K6YTNPEKXYM7",
      "email": "user@example.com",
      "isVerified": false,
      "createdAt": "2025-07-19T12:00:00Z"
    },
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "expiresIn": "24h"
  }
}
```

**Error Responses:**
- `400 Bad Request` - Invalid input data
- `409 Conflict` - Email already exists
- `422 Unprocessable Entity` - Password validation failed

### Login User

Authenticate existing user and return JWT token.

**Endpoint:** `POST /login`

**Request Body:**
```json
{
  "email": "user@example.com",
  "password": "securePassword123"
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Login successful",
  "data": {
    "user": {
      "id": "01HYN8FXZ2W6X8K6YTNPEKXYM7",
      "email": "user@example.com",
      "isVerified": true,
      "lastActiveAt": "2025-07-19T15:30:00Z"
    },
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "expiresIn": "24h"
  }
}
```

**Error Responses:**
- `400 Bad Request` - Invalid input data
- `401 Unauthorized` - Invalid credentials
- `423 Locked` - Account temporarily locked due to failed attempts

### Refresh Token

Refresh an existing JWT token before it expires.

**Endpoint:** `POST /refresh`

**Headers:**
```
Authorization: Bearer <jwt_token>
```

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Token refreshed successfully",
  "data": {
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "expiresIn": "24h"
  }
}
```

**Error Responses:**
- `401 Unauthorized` - Invalid or expired token
- `403 Forbidden` - Token not eligible for refresh

### Logout User

Invalidate the current JWT token (adds to blacklist).

**Endpoint:** `POST /logout`

**Headers:**
```
Authorization: Bearer <jwt_token>
```

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Logged out successfully"
}
```

**Error Responses:**
- `401 Unauthorized` - Invalid token

### Verify Email

Verify user email address using verification token.

**Endpoint:** `POST /verify-email`

**Request Body:**
```json
{
  "token": "verification_token_string"
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Email verified successfully",
  "data": {
    "user": {
      "id": "01HYN8FXZ2W6X8K6YTNPEKXYM7",
      "email": "user@example.com",
      "isVerified": true
    }
  }
}
```

**Error Responses:**
- `400 Bad Request` - Invalid or expired verification token
- `409 Conflict` - Email already verified

### Resend Verification Email

Send a new email verification link.

**Endpoint:** `POST /resend-verification`

**Request Body:**
```json
{
  "email": "user@example.com"
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Verification email sent successfully"
}
```

**Error Responses:**
- `400 Bad Request` - Invalid email format
- `404 Not Found` - User not found
- `409 Conflict` - Email already verified
- `429 Too Many Requests` - Rate limit exceeded

### Forgot Password

Initiate password reset process.

**Endpoint:** `POST /forgot-password`

**Request Body:**
```json
{
  "email": "user@example.com"
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Password reset email sent successfully"
}
```

**Error Responses:**
- `400 Bad Request` - Invalid email format
- `429 Too Many Requests` - Rate limit exceeded

### Reset Password

Reset password using reset token.

**Endpoint:** `POST /reset-password`

**Request Body:**
```json
{
  "token": "password_reset_token",
  "password": "newSecurePassword123",
  "confirmPassword": "newSecurePassword123"
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Password reset successfully"
}
```

**Error Responses:**
- `400 Bad Request` - Invalid or expired reset token
- `422 Unprocessable Entity` - Password validation failed

### Change Password

Change password for authenticated user.

**Endpoint:** `POST /change-password`

**Headers:**
```
Authorization: Bearer <jwt_token>
```

**Request Body:**
```json
{
  "currentPassword": "oldPassword123",
  "newPassword": "newSecurePassword123",
  "confirmPassword": "newSecurePassword123"
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Password changed successfully"
}
```

**Error Responses:**
- `400 Bad Request` - Invalid current password
- `401 Unauthorized` - Invalid token
- `422 Unprocessable Entity` - New password validation failed

### Get Current User

Retrieve current authenticated user information.

**Endpoint:** `GET /me`

**Headers:**
```
Authorization: Bearer <jwt_token>
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "user": {
      "id": "01HYN8FXZ2W6X8K6YTNPEKXYM7",
      "email": "user@example.com",
      "isVerified": true,
      "role": "user",
      "createdAt": "2025-07-19T12:00:00Z",
      "lastActiveAt": "2025-07-19T15:30:00Z"
    }
  }
}
```

**Error Responses:**
- `401 Unauthorized` - Invalid or expired token

## JWT Token Structure

### Header
```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

### Payload
```json
{
  "sub": "01HYN8FXZ2W6X8K6YTNPEKXYM7",
  "email": "user@example.com",
  "role": "user",
  "iat": 1642680000,
  "exp": 1642766400
}
```

### Usage in Requests

Include the JWT token in the Authorization header:

```http
GET /api/users/profile
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
Content-Type: application/json
```

## Security Features

### Password Requirements
- Minimum 8 characters
- At least one lowercase letter
- At least one uppercase letter
- At least one number
- At least one special character
- Maximum 128 characters

### Rate Limiting
- **Login attempts**: 5 attempts per 15 minutes per IP
- **Registration**: 3 attempts per hour per IP
- **Password reset**: 3 requests per hour per email
- **Email verification**: 5 requests per hour per email

### Token Security
- **Expiration**: Tokens expire after 24 hours
- **Blacklisting**: Logged out tokens are blacklisted
- **Refresh window**: Tokens can be refreshed within 1 hour of expiry
- **Secure transmission**: Tokens only sent over HTTPS in production

### Account Security
- **Account lockout**: After 5 failed login attempts, account locked for 15 minutes
- **Email verification**: Required for account activation
- **Password hashing**: Using bcrypt with salt rounds
- **Session tracking**: Monitor active sessions and unusual activity

## Error Response Format

All error responses follow this standard format:

```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Password must contain at least 8 characters",
    "details": {
      "field": "password",
      "rejectedValue": "***"
    }
  },
  "timestamp": "2025-07-19T15:30:00Z",
  "path": "/api/auth/register"
}
```

### Common Error Codes
- `VALIDATION_ERROR` - Input validation failed
- `AUTHENTICATION_ERROR` - Invalid credentials
- `AUTHORIZATION_ERROR` - Insufficient permissions
- `RATE_LIMIT_EXCEEDED` - Too many requests
- `TOKEN_EXPIRED` - JWT token has expired
- `TOKEN_INVALID` - JWT token is malformed or invalid
- `ACCOUNT_LOCKED` - Account temporarily locked
- `EMAIL_NOT_VERIFIED` - Email verification required

## Testing Examples

### cURL Examples

**Register a new user:**
```bash
curl -X POST http://localhost:8080/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "email": "test@example.com",
    "password": "SecurePass123!",
    "confirmPassword": "SecurePass123!"
  }'
```

**Login:**
```bash
curl -X POST http://localhost:8080/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "test@example.com",
    "password": "SecurePass123!"
  }'
```

**Get current user (with token):**
```bash
curl -X GET http://localhost:8080/api/auth/me \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

## Integration Notes

### Frontend Integration
- Store JWT token in secure HTTP-only cookie or localStorage
- Implement automatic token refresh logic
- Handle authentication state globally (React Context/Redux)
- Redirect to login on 401 responses

### Security Best Practices
- Always use HTTPS in production
- Implement CSRF protection for cookie-based auth
- Store secrets in environment variables
- Use secure, random JWT signing keys
- Implement proper CORS configuration
- Log authentication events for security monitoring

<!-- TODO: Implement OAuth2 integration (Google, GitHub) -->
<!-- TODO: Add two-factor authentication (2FA) support -->
<!-- TODO: Implement device/session management -->
<!-- TODO: Add audit logging for authentication events -->