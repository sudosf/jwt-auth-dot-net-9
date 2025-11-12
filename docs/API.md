# API Documentation

Complete reference for all JWT Authentication API endpoints.

## Base URL

```
https://localhost:<port>/api/auth
```

## Authentication

Most endpoints require JWT authentication. Include the access token in the request header:

```
Authorization: Bearer {accessToken}
```

## Endpoints

### 1. Register a New User

Create a new user account.

**Endpoint:** `POST /api/auth/register`

**Authentication:** None

**Request Body:**

```json
{
  "username": "john_doe",
  "password": "SecurePassword123!"
}
```

**Request Parameters:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `username` | string | Yes | Unique username (min 3 characters) |
| `password` | string | Yes | User password (min 6 characters) |

**Success Response (200 OK):**

```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "username": "john_doe",
  "passwordHash": "$2a$11$H9w.PPQubzc.",
  "role": "",
  "refreshToken": null,
  "refreshTokenExpiryTime": null
}
```

**Error Response (400 Bad Request):**

```json
"Username already exists."
```

**cURL Example:**

```bash
curl -X POST https://localhost:<port>/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "username": "john_doe",
    "password": "SecurePassword123!"
  }'
```

---

### 2. Login

Authenticate user and receive JWT tokens.

**Endpoint:** `POST /api/auth/login`

**Authentication:** None

**Request Body:**

```json
{
  "username": "john_doe",
  "password": "SecurePassword123!"
}
```

**Request Parameters:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `username` | string | Yes | Registered username |
| `password` | string | Yes | User password |

**Success Response (200 OK):**

```json
{
  "accessToken": "eyJhbGciOiJIUzUxMiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJqb2huX2RvZSIsIm5hbWVpZCI6IjU1MGU4NDAwLWUyOWItNDFkNC1hNzE2LTQ0NjY1NTQ0MDAwMCIsInJvbGUiOiIiLCJleHAiOjE3MzEzNDU2MDB9.qF8Ky...",
  "refreshToken": "rN2Z5X9kL4mQ8pV3tY7wA6bC1dE2fG3hI4jK5lM6nO7pQ8rS9tU0vW1xY2zA3bC4d"
}
```

**Response Fields:**

| Field | Type | Description |
|-------|------|-------------|
| `accessToken` | string | JWT token for API requests (expires in 1 day) |
| `refreshToken` | string | Token used to refresh access token (expires in 7 days) |

**Error Response (400 Bad Request):**

```json
"Invalid username or password."
```

**cURL Example:**

```bash
curl -X POST https://localhost:<port>/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "username": "john_doe",
    "password": "SecurePassword123!"
  }'
```

---

### 3. Get Authenticated User Info

Access protected endpoint that requires valid JWT token.

**Endpoint:** `GET /api/auth`

**Authentication:** Required (Bearer Token)

**Request Headers:**

```
Authorization: Bearer {accessToken}
```

**Success Response (200 OK):**

```json
"You are authenticated!"
```

**Error Response (401 Unauthorized):**

```json
"Unauthorized"
```

**cURL Example:**

```bash
curl -X GET https://localhost:<port>/api/auth \
  -H "Authorization: Bearer eyJhbGciOiJIUzUxMiIsInR5cCI6IkpXVCJ9..."
```

---

### 4. Access Admin-Only Endpoint

Access endpoint restricted to users with Admin role.

**Endpoint:** `GET /api/auth/admin-only`

**Authentication:** Required (Bearer Token)

**Authorization:** User must have `Admin` role

**Request Headers:**

```
Authorization: Bearer {accessToken}
```

**Success Response (200 OK):**

```json
"You are an admin!"
```

**Error Responses:**

**401 Unauthorized** (No token or expired token):

```json
"Unauthorized"
```

**403 Forbidden** (Token valid but user is not admin):

```json
"Access Denied"
```

**cURL Example:**

```bash
curl -X GET https://localhost:<port>/api/auth/admin-only \
  -H "Authorization: Bearer eyJhbGciOiJIUzUxMiIsInR5cCI6IkpXVCJ9..."
```

---

### 5. Refresh Access Token

Get a new access token using a valid refresh token.

**Endpoint:** `POST /api/auth/refresh-token`

**Authentication:** None

**Request Body:**

```json
{
  "userId": "550e8400-e29b-41d4-a716-446655440000",
  "refreshToken": "rN2Z5X9kL4mQ8pV3tY7wA6bC1dE2fG3hI4jK5lM6nO7pQ8rS9tU0vW1xY2zA3bC4d"
}
```

**Request Parameters:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `userId` | GUID | Yes | The user's ID |
| `refreshToken` | string | Yes | Valid refresh token from login |

**Success Response (200 OK):**

```json
{
  "accessToken": "eyJhbGciOiJIUzUxMiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJqb2huX2RvZSIsIm5hbWVpZCI6IjU1MGU4NDAwLWUyOWItNDFkNC1hNzE2LTQ0NjY1NTQ0MDAwMCIsInJvbGUiOiIiLCJleHAiOjE3MzEzNDU2MDB9.qF8Ky...",
  "refreshToken": "mK9pL2tY5vX8qR3sW1zU4bV6cW7xY8zZ9aA0bB1cC2dD3eE4fF5gG6hH7iI8jJ9k"
}
```

**Response Fields:**

| Field | Type | Description |
|-------|------|-------------|
| `accessToken` | string | New JWT token for API requests |
| `refreshToken` | string | New refresh token (rotated for security) |

**Error Response (401 Unauthorized):**

```json
"Invalid refresh token."
```

**Possible Error Reasons:**

- Refresh token has expired
- Refresh token doesn't match user in database
- User ID doesn't exist
- Refresh token is invalid

**cURL Example:**

```bash
curl -X POST https://localhost:<port>/api/auth/refresh-token \
  -H "Content-Type: application/json" \
  -d '{
    "userId": "550e8400-e29b-41d4-a716-446655440000",
    "refreshToken": "rN2Z5X9kL4mQ8pV3tY7wA6bC1dE2fG3hI4jK5lM6nO7pQ8rS9tU0vW1xY2zA3bC4d"
  }'
```

## Request/Response Models

### UserDto

Used for registration and login requests.

```csharp
public class UserDto
{
    public string Username { get; set; } = string.Empty;
    public string Password { get; set; } = string.Empty;
}
```

### TokenResponseDto

Response from login and refresh token endpoints.

```csharp
public class TokenResponseDto
{
    public required string AccessToken { get; set; }
    public required string RefreshToken { get; set; }
}
```

### RefreshTokenRequestDto

Request for refreshing tokens.

```csharp
public class RefreshTokenRequestDto
{
    public Guid UserId { get; set; }
    public string RefreshToken { get; set; } = string.Empty;
}
```

### User Entity

User data stored in the database.

```csharp
public class User
{
    public Guid Id { get; set; }
    public string Username { get; set; } = string.Empty;
    public string PasswordHash { get; set; } = string.Empty;
    public string Role { get; set; } = string.Empty;
    public string? RefreshToken { get; set; }
    public DateTime? RefreshTokenExpiryTime { get; set; }
}
```

## HTTP Status Codes

| Code | Meaning | Description |
|------|---------|-------------|
| 200 | OK | Request succeeded |
| 400 | Bad Request | Invalid request data or validation error |
| 401 | Unauthorized | Missing or invalid authentication token |
| 403 | Forbidden | Authenticated but insufficient permissions (e.g., not admin) |
| 500 | Internal Server Error | Server-side error |

## Authentication Flow

### Complete Login Flow

```
1. User registers
   POST /api/auth/register
   → Response: User object
   
2. User logs in
   POST /api/auth/login
   → Response: { accessToken, refreshToken }
   
3. User calls protected endpoint with accessToken
   GET /api/auth
   Header: Authorization: Bearer {accessToken}
   → Response: Success message
   
4. When accessToken expires, use refreshToken
   POST /api/auth/refresh-token
   → Response: { accessToken, refreshToken }
   
5. Continue making API calls with new accessToken
```

## JWT Token Structure

Access tokens contain the following claims:

```json
{
  "alg": "HS512",
  "typ": "JWT"
}
{
  "sub": "john_doe",
  "nameid": "550e8400-e29b-41d4-a716-446655440000",
  "role": "User",
  "exp": 1731345600
}
```

**Claims:**

| Claim | Description |
|-------|-------------|
| `sub` | Subject - Username |
| `nameid` | Name ID - User's unique ID (GUID) |
| `role` | User's role for authorization |
| `exp` | Expiration time (Unix timestamp) |

## Token Expiration

| Token | Expiration | Purpose |
|-------|-----------|---------|
| Access Token | 1 day | Short-lived, used for API requests |
| Refresh Token | 7 days | Long-lived, used to get new access token |

## Common Issues & Solutions

### "Unauthorized" on Protected Endpoints

**Problem:** Getting 401 response on authorized endpoints.

**Solutions:**

1. Check token is included in Authorization header
2. Verify token hasn't expired (check exp claim)
3. Ensure token format is `Bearer {token}` (with space)
4. Check JWT signing key matches between server and client

### Invalid Refresh Token

**Problem:** Refresh token returns 401.

**Solutions:**

1. Verify refresh token hasn't expired (7 days from login)
2. Confirm userId matches the original login
3. Check refresh token hasn't been used after expiration
4. Try logging in again to get fresh tokens

### Token Validation Fails

**Problem:** Valid-looking token still fails validation.

**Solutions:**

1. Ensure `AppSettings:Token` secret is configured correctly
2. Verify `AppSettings:Issuer` matches token issuer claim
3. Verify `AppSettings:Audience` matches token audience claim
4. Check server time is synchronized (token may be future-dated)

## Testing Endpoints

### Using cURL

See individual endpoint sections for cURL examples.

### Using Postman

1. Get accessToken from login endpoint
2. In Postman, go to Authorization tab
3. Select "Bearer Token" type
4. Paste the accessToken
5. Make requests

### Using Scalar UI

1. Run the application
2. Navigate to `https://localhost:<port>/scalar/v1`
3. Browse endpoints and test directly

## Rate Limiting

Currently, there is no rate limiting implemented. For production, consider:

- Implementing AspNetCoreRateLimit middleware
- Using API Management (Azure API Management)
- Configuring server-side throttling

## CORS Configuration

Currently, CORS is not configured.

## Versioning

This API is currently at **v1**. All endpoints are prefixed with `/api/auth`.

Future versions may use `/api/v2/auth`, etc.

## 🤖 AI Disclaimer

This documentation was generated with the assistance of AI and reviewed by a human for clarity and accuracy. ❤️

**Last Updated:** November 2025
