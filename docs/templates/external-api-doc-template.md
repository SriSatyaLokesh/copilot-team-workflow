---
api-name: "[External API Name, e.g., Microsoft Dynamics 365]"
base-url: "https://[your-org].api.crm.dynamics.com/api/data/v9.2"
auth-type: "OAuth2 Client Credentials"
last-updated: "YYYY-MM-DD"
wrapper-class: "src/wrappers/[name]-wrapper.ts"
---

# [External API Name] — Integration Reference

> This document is the **single source of truth** for everything Copilot needs
> to write code that calls this external API correctly.

---

## 1. Authentication

### Method
[e.g., OAuth2 Client Credentials Flow / API Key / Basic Auth]

### How to Get a Token
```
Token URL: https://login.microsoftonline.com/[tenant-id]/oauth2/v2.0/token
Method:    POST
Body (form-urlencoded):
  grant_type:    client_credentials
  client_id:     [from env: DYNAMICS_CLIENT_ID]
  client_secret: [from env: DYNAMICS_CLIENT_SECRET]
  scope:         https://[org].api.crm.dynamics.com/.default

Response:
  access_token:  <JWT>
  expires_in:    3600   ← token lasts 1 hour
  token_type:    Bearer
```

### How to Use the Token
```
Header: Authorization: Bearer <access_token>
```

### Token Refresh Strategy
- Token expires after [X] minutes
- Our wrapper auto-refreshes: checks `expires_at` before each call
- Environment variables needed:
  - `[API_NAME]_CLIENT_ID` 
  - `[API_NAME]_CLIENT_SECRET`
  - `[API_NAME]_TENANT_ID`
  - `[API_NAME]_BASE_URL`

---

## 2. Base URL & API Version

```
Base URL: https://[org].api.crm.dynamics.com/api/data/v9.2
```

All endpoints are relative to this base URL.

---

## 3. Common Request Headers

Every call must include:
```
Authorization: Bearer <token>
Content-Type:  application/json
OData-MaxVersion: 4.0
OData-Version: 4.0
Accept: application/json
```

---

## 4. Rate Limits & Quotas

| Limit | Value | What Happens |
|-------|-------|-------------|
| Requests per 5 min | [X] | HTTP 429, Retry-After header |
| Concurrent requests | [X] | Queued |
| Max response size | [X MB] | 413 error |

**Our retry strategy**: Exponential backoff, max 3 retries on 429 / 503.

---

## 5. Error Responses

| HTTP Code | Meaning | What to Do |
|-----------|---------|-----------|
| 400 | Bad request / invalid OData | Fix the query — log raw error |
| 401 | Auth failed | Refresh token and retry once |
| 403 | No permission to entity | Check user permissions — do not retry |
| 404 | Record not found | Return null / throw NotFoundError |
| 429 | Rate limited | Wait for Retry-After, then retry |
| 500 | Server error | Log details, retry with backoff |

**Error response shape:**
```json
{
  "error": {
    "code": "0x80040217",
    "message": "account With Id = ... Does Not Exist"
  }
}
```

---

## 6. Entities / Resources

### Entity Index

| Entity | Our Name | Endpoint | Used For |
|--------|---------|----------|---------|
| accounts | Customer / Account | `/accounts` | Company records |
| contacts | Contact | `/contacts` | Individual people |
| opportunities | Opportunity | `/opportunities` | Sales deals |
| [entity] | [internal name] | `/[entity]` | [purpose] |

---

### Entity: accounts (Customer)

**Endpoint**: `GET/POST/PATCH /accounts`

#### Available Fields

| API Field Name | Our Internal Name | Type | Description |
|---------------|------------------|------|-------------|
| `accountid` | `id` | GUID | Primary key |
| `name` | `name` | string | Company name |
| `emailaddress1` | `email` | string | Primary email |
| `telephone1` | `phone` | string | Primary phone |
| `statecode` | `isActive` | `0=Active, 1=Inactive` | Status |
| `createdon` | `createdAt` | ISO datetime | Creation timestamp |
| `_primarycontactid_value` | `primaryContactId` | GUID | Lookup to contacts |
| `customernumber` | `accountNumber` | string | Business identifier |
| `address1_city` | `city` | string | City |
| `address1_country` | `country` | string | Country |

#### Example: Get by ID
```
GET /accounts([account-guid])
?$select=accountid,name,emailaddress1,statecode
```

Response:
```json
{
  "@odata.context": "...",
  "accountid": "12345678-...",
  "name": "Acme Corp",
  "emailaddress1": "billing@acme.com",
  "statecode": 0
}
```

#### Example: Query with Filters
```
GET /accounts
?$select=accountid,name,statecode
&$filter=statecode eq 0 and _primarycontactid_value ne null
&$orderby=createdon desc
&$top=50
```

#### Example: Create
```
POST /accounts
Body: {
  "name": "New Corp",
  "emailaddress1": "info@newcorp.com"
}
Response header: OData-EntityId: [url with new guid]
```

#### Example: Update
```
PATCH /accounts([account-guid])
Body: { "emailaddress1": "new@email.com" }
Response: 204 No Content
```

---

### Entity: [Next Entity]
[Copy the above section and fill in for each entity]

---

## 7. Common Query Patterns

### OData Filter Examples

```
-- Active records only
$filter=statecode eq 0

-- Records belonging to a user
$filter=_ownerid_value eq [user-guid]

-- Date range
$filter=createdon ge 2026-01-01T00:00:00Z and createdon le 2026-12-31T23:59:59Z

-- Contains text
$filter=contains(name,'Acme')

-- Lookup field
$filter=_accountid_value eq [account-guid]
```

### Expand Related Records

```
GET /contacts
?$select=contactid,fullname
&$expand=parentcustomerid_account($select=name,emailaddress1)
```

---

## 8. Our Wrapper Class Reference

**File**: `src/wrappers/[name]-wrapper.ts`

| Method | Calls | Returns |
|--------|-------|---------|
| `getAccount(id)` | `GET /accounts(id)` | `CustomerAccount` |
| `listAccounts(filters)` | `GET /accounts?$filter=...` | `CustomerAccount[]` |
| `createAccount(data)` | `POST /accounts` | `{ id: string }` |
| `updateAccount(id, data)` | `PATCH /accounts(id)` | `void` |
| [add rows as methods are created] | | |

---

## 9. Transformer Reference

**File**: `src/transformers/[name]-transformer.ts`

Documents which external field maps to which internal field.
[See the entity tables in Section 6 — the "Our Internal Name" column is the transformer output.]

---

## 10. Known Gotchas

1. [Gotcha 1 — e.g., "Lookup fields come back as `_fieldname_value` with a leading underscore"]
2. [Gotcha 2 — e.g., "PATCH returns 204, not the updated record — must call GET after if you need updated data"]
3. [Gotcha 3 — e.g., "OData boolean filters use `true`/`false`, not `1`/`0`"]

---

## 11. Change History

| Date | Change | Developer |
|------|--------|-----------|
| YYYY-MM-DD | Initial documentation | [Name] |
