# POST /auth/login

> **Method**: `POST`
> **Path**: `/auth/login`
> **Auth required**: No
> **Last updated**: 2026-03-02

> **This is an example API doc.** Copy `docs/templates/api-doc-template.md` to create your own.

---

## Description

Authenticates a user with email and password. Returns a JWT access token and refresh token.
Rate limited to 5 failed attempts per 15 minutes per email address. Admin users are exempt.

---

## Request

**Headers**
```
Content-Type: application/json
```

**Body**
```json
{
  "email": "user@example.com",
  "password": "SecurePassword123!"
}
```

**Validation rules**
- `email`: required, valid email format
- `password`: required, min 8 characters

---

## Response

### 200 OK — Success
```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refreshToken": "dGhpcyBpcyBhIHJlZnJlc2ggdG9rZW4...",
  "expiresIn": 3600,
  "user": {
    "id": "usr_abc123",
    "email": "user@example.com",
    "name": "John Smith",
    "role": "customer"
  }
}
```

### 401 Unauthorized — Invalid credentials
```json
{
  "error": "INVALID_CREDENTIALS",
  "message": "Email or password is incorrect"
}
```

### 429 Too Many Requests — Rate limit exceeded
```json
{
  "error": "RATE_LIMIT_EXCEEDED",
  "message": "Too many failed attempts. Try again in 847 seconds.",
  "retryAfter": 847
}
```
**Headers**: `Retry-After: 847`

### 422 Unprocessable Entity — Validation error
```json
{
  "error": "VALIDATION_ERROR",
  "fields": {
    "email": "Must be a valid email address",
    "password": "Must be at least 8 characters"
  }
}
```

---

## Implementation

| Layer | File |
|-------|------|
| Controller | `src/controllers/auth.controller.ts` → `login()` |
| Middleware | `src/middleware/rate-limit.ts` |
| Service | `src/services/auth.service.ts` → `login()` |

---

## Notes

- The JWT token expires in **1 hour**. Use the refresh token to get a new one.
- Rate limit applies **per email address**, not per IP.
- Admin users (`role: admin`) bypass the rate limit entirely.
