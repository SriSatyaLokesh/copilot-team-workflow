---
api-id: "[domain]-[action]"   # e.g., auth-login, orders-create
endpoint: "[METHOD] /api/[path]"  # e.g., POST /api/auth/login
status: "stable"  # stable | beta | deprecated
version: "v1"
last-updated: "YYYY-MM-DD"
related-flow: "[flow-name]-flow"
related-Issue: "ISSUE-XXX"
owner: "[Team Name]"
---

# [METHOD] /api/[path]

## Purpose
[1-2 sentences: what does this endpoint do? When would a client call it?]

---

## Authentication
- [ ] **Required** — Include `Authorization: Bearer <accessToken>` header
- [ ] **Not Required** — Public endpoint

**Permissions needed**: [e.g., `role: admin` or `any authenticated user`]

---

## Request

### Headers
```
Content-Type: application/json
Authorization: Bearer <accessToken>   ← (if required)
```

### URL Parameters
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `:id` | string | yes | [Description] |

### Query Parameters
| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `page` | number | no | 1 | Page number |
| `limit` | number | no | 20 | Items per page (max 100) |

### Request Body
```json
{
  "field1": "value",      // string, required | Description
  "field2": 42,           // number, optional | Description, default: X
  "field3": {
    "nested": "value"     // string, required if field3 is provided
  }
}
```

### Validation Rules
- `field1`: minimum 2 characters, maximum 100 characters
- `field2`: must be positive integer
- [other rules]

---

## Response

### Success Response

**Status**: `200 OK` (or `201 Created` for creates)

```json
{
  "id": "ord_01HZ...",
  "status": "pending",
  "createdAt": "2026-03-01T10:00:00Z",
  "updatedAt": "2026-03-01T10:00:00Z",
  "data": {
    "field1": "value"
  }
}
```

**Response fields**:
| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Unique identifier (prefix format: `ord_`) |
| `status` | string | Current status: `pending` \| `active` \| `cancelled` |
| `createdAt` | ISO8601 | When the record was created |

### Error Responses

| HTTP Status | Error Code | Message | When Does This Happen? |
|-------------|------------|---------|----------------------|
| 400 | `VALIDATION_ERROR` | "Field X is required" | Missing or invalid request body |
| 401 | `UNAUTHORIZED` | "Authentication required" | No or invalid access token |
| 403 | `FORBIDDEN` | "Insufficient permissions" | Valid token but wrong role |
| 404 | `NOT_FOUND` | "[Resource] not found" | ID doesn't exist |
| 409 | `CONFLICT` | "[Resource] already exists" | Duplicate creation attempt |
| 422 | `UNPROCESSABLE` | "Cannot process: [reason]" | Business rule violation |
| 429 | `RATE_LIMIT_EXCEEDED` | "Too many requests" | Rate limit hit |
| 500 | `SERVER_ERROR` | "An unexpected error occurred" | Internal server error |

**Error response format** (all errors use this shape):
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Email is required",
    "details": {
      "field": "email",
      "constraint": "required"
    }
  }
}
```

---

## Business Rules

1. [Rule 1 — e.g., "Users can only access their own orders"]
2. [Rule 2 — e.g., "Orders cannot be cancelled after they are shipped"]
3. [Rule 3 — e.g., "Maximum 100 orders can be created per day per account"]

---

## Rate Limiting

| Limit | Window | Applied Per |
|-------|--------|-------------|
| 10 requests | 1 minute | per user |
| 1000 requests | 1 hour | per IP |

---

## Implementation

```
Route file:     src/api/[domain]/route.ts
Handler:        [FunctionName]()
Service:        src/services/[domain].service.ts → [methodName]()
Repository:     src/repositories/[domain].repository.ts
Middleware:     [list middleware applied]
```

---

## Related Tests

```
Unit:          src/services/[domain].service.test.ts → describe('[methodName]')
Integration:   tests/integration/[domain]/[endpoint].test.ts
E2E:           tests/e2e/[flow-name]-flow.spec.ts → test('[scenario]')
```

---

## Examples

### Example: Successful request

```bash
curl -X POST https://api.example.com/api/[path] \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer eyJhbGci..." \
  -d '{
    "field1": "value",
    "field2": 42
  }'
```

Response:
```json
{
  "id": "res_01HZ...",
  "status": "success"
}
```

### Example: Validation error

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "field1 is required",
    "details": { "field": "field1", "constraint": "required" }
  }
}
```

---

## Change History

| Date | Change | Developer | Issue |
|------|--------|-----------|-------|
| YYYY-MM-DD | Initial implementation | [Name] | ISSUE-001 |
| YYYY-MM-DD | [What changed] | [Name] | ISSUE-XXX |
