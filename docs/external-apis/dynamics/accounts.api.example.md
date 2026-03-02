# External API: Microsoft Dynamics 365 — Accounts Entity

> **API**: Microsoft Dynamics 365 CRM
> **Entity**: Accounts (companies / organisations)
> **Base URL**: See `.env` → `DYNAMICS_BASE_URL`

> **This is an example entity doc.** Copy `docs/templates/external-api-doc-template.md` to create your own.

---

## Authentication

Dynamics uses **OAuth 2.0 Client Credentials** (app-to-app, no user login).

```
POST https://login.microsoftonline.com/{TENANT_ID}/oauth2/v2.0/token

Body (form-encoded):
  grant_type=client_credentials
  client_id={DYNAMICS_CLIENT_ID}
  client_secret={DYNAMICS_CLIENT_SECRET}
  scope=https://{DYNAMICS_ORG}.crm.dynamics.com/.default
```

**Token lifetime**: 1 hour. The wrapper refreshes automatically before expiry.

**Required env vars**:
```
DYNAMICS_TENANT_ID=
DYNAMICS_CLIENT_ID=
DYNAMICS_CLIENT_SECRET=
DYNAMICS_BASE_URL=https://yourorg.crm.dynamics.com/api/data/v9.2
```

---

## Required Headers (every request)

```
Authorization: Bearer {token}
OData-MaxVersion: 4.0
OData-Version: 4.0
Accept: application/json
Content-Type: application/json
```

---

## Rate Limits

| Limit | Value | What happens when exceeded |
|-------|-------|--------------------------|
| Requests / 5 min | 6000 | HTTP 429, `Retry-After` header (seconds) |
| Concurrent requests | 52 | Queued, eventually completes |

Our wrapper uses **exponential backoff**: 1s → 2s → 4s, max 3 retries on 429.

---

## Field Mapping Table — Accounts

> ⚠️ These are the **exact field names from the Dynamics API response**. Do not guess.

| Dynamics Field | Our Internal Field | Type | Notes |
|---------------|-------------------|------|-------|
| `accountid` | `id` | GUID string | Primary key |
| `name` | `name` | string | Company name |
| `emailaddress1` | `email` | string | Primary email |
| `telephone1` | `phone` | string | Main phone |
| `address1_city` | `city` | string | City |
| `address1_country` | `country` | string | Country |
| `statecode` | `isActive` | `0=Active, 1=Inactive` → boolean | Account status |
| `createdon` | `createdAt` | ISO datetime string | When created in Dynamics |
| `_primarycontactid_value` | `primaryContactId` | GUID string | FK to main contact |

---

## Common Query Examples

### Get account by ID
```
GET /accounts(12345678-abcd-1234-efgh-567890abcdef)
?$select=accountid,name,emailaddress1,telephone1,statecode,createdon
```

### Filter by email
```
GET /accounts
?$filter=emailaddress1 eq 'billing@acme.com'
&$select=accountid,name,emailaddress1,statecode
```

### Get account with primary contact
```
GET /accounts(12345678-abcd-...)
?$expand=primarycontactid($select=contactid,fullname,emailaddress1)
&$select=accountid,name,statecode
```

---

## Example Response

```json
{
  "@odata.context": "https://yourorg.crm.dynamics.com/api/data/v9.2/$metadata#accounts",
  "accountid": "12345678-abcd-1234-efgh-567890abcdef",
  "name": "Acme Corporation",
  "emailaddress1": "billing@acme.com",
  "telephone1": "+1-555-0100",
  "address1_city": "San Francisco",
  "address1_country": "United States",
  "statecode": 0,
  "createdon": "2026-01-15T09:00:00Z",
  "_primarycontactid_value": "abcdef12-3456-7890-abcd-ef1234567890"
}
```

---

## Known Gotchas

1. **`emailaddress1`** — the "1" is not a typo. `emailaddress2` and `emailaddress3` also exist.
2. **`statecode: 0` = Active** — it's NOT a boolean. `0=Active`, `1=Inactive`. Our transformer converts this to `isActive: boolean`.
3. **`_primarycontactid_value`** — lookup fields always have the `_` prefix and `_value` suffix in OData responses.
4. **Deleted records** — Dynamics soft-deletes by setting `statecode: 1`. Records are never truly deleted via the API.
5. **`$select` is mandatory** — never omit `$select`. Without it, you get 100+ fields including internal metadata, making responses 10x larger.
