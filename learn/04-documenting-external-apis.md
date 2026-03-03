---
layout: default
parent: "Copilot Team Workflow"
title: "Part 04 - Documenting External APIs"
description: "How to document external API field names so Copilot never guesses wrong field names at runtime."
nav_order: 5
---

# Learn: Documenting External APIs


---

## The Core Problem

Your project calls external APIs — Microsoft Dynamics, Stripe, SendGrid, or others. These APIs have their own field names, auth flows, and response shapes.

When Copilot writes wrapper code, it has to know:
- What is the exact URL to call?
- What headers are required?
- What are the exact field names in the response? (`emailaddress1`? `email_address`? `email`?)
- What does a 429 response look like?

Without this information, Copilot **guesses**. It will use `email` when the field is actually `emailaddress1`. The code looks right but breaks at runtime.

**The solution**: Document external APIs in `docs/external-apis/` BEFORE writing wrapper code. This folder is Copilot's reference library for every external service.

---

## The Folder Structure

```
docs/external-apis/
├── README.md                    ← Index of all APIs
└── dynamics/                    ← One folder per external API
    ├── README.md                ← Auth, base URL, rate limits, common patterns
    ├── accounts.api.md          ← Account entity: every field, query examples
    ├── contacts.api.md          ← Contact entity: every field, query examples
    └── orders.api.md            ← Order entity: every field, query examples
```

**Rule**: One folder per external API. One file per entity/resource within that API.

---

## What Goes in Each File

### `docs/external-apis/dynamics/README.md`

This file covers everything that applies to ALL calls to this API:

1. **Authentication** — how to get a token, how long it lasts, what headers to send
2. **Base URL** — exact URL with version
3. **Common headers** — headers required on every request
4. **Rate limits** — requests per window, what happens when exceeded
5. **Error responses** — what 400/401/403/404/429/500 look like
6. **Common query patterns** — OData filters, select, expand examples
7. **Known gotchas** — things that will surprise developers

### `docs/external-apis/dynamics/accounts.api.md`

This file covers everything specific to the accounts entity:

1. **The field mapping table** — the most important part

| API Field Name | Our Internal Name | Type | Description |
|:---|:---|:---|:---|
| `accountid` | `id` | GUID | Primary key |
| `name` | `name` | string | Company name |
| `emailaddress1` | `email` | string | Primary contact email |
| `telephone1` | `phone` | string | Main phone number |
| `statecode` | `isActive` | `0=Active, 1=Inactive` | Account status |
| `createdon` | `createdAt` | ISO datetime string | When created |

2. **Example API calls** — exact request/response so developers can copy-paste

```markdown
### Get account by ID
GET /accounts(12345678-abcd-...)
?$select=accountid,name,emailaddress1,statecode,createdon

Response:
{
  "@odata.context": "...",
  "accountid": "12345678-abcd-...",
  "name": "Acme Corp",
  "emailaddress1": "billing@acme.com",
  "statecode": 0,
  "createdon": "2026-01-15T09:00:00Z"
}
```

3. **OData filter examples** — common queries with exact syntax

---

## How to Add Documentation for a New API

### Step 1: Copy the template

```bash
cp docs/templates/external-api-doc-template.md docs/external-apis/dynamics/README.md
cp docs/templates/external-api-doc-template.md docs/external-apis/dynamics/accounts.api.md
```

### Step 2: Fill in the field mapping table

This is the most critical step. Go to the external API's documentation and find:
- The exact field name sent in the JSON response
- The data type
- What values it can have (especially for enums like `statecode`)

**Do not guess field names**. Open the API in Postman, make a real call, copy the actual response, map every field.

### Step 3: Add authentication details

Get the exact token URL, required parameters, and headers from the API vendor's docs. Document:
- How the token is requested (what grant type, what scopes)
- How long the token lasts
- What the Authorization header looks like
- Which env vars hold the credentials

### Step 4: Commit the doc before writing any code

```bash
git add docs/external-apis/dynamics/
git commit -m "docs: add Dynamics accounts API reference"
```

Then write the wrapper code. The ApiBuilder agent will read this doc automatically when you run `/add-new-api`.

---

## What Happens When Copilot Reads These Docs

When a developer runs `/add-new-api` and specifies "dynamics" + "accounts":

1. The ApiBuilder agent reads `docs/external-apis/dynamics/accounts.api.md`
2. It sees the field mapping table: `emailaddress1 → email`
3. It writes the transformer correctly:
   ```typescript
   email: raw.emailaddress1,  // not raw.email — correct!
   ```
4. It writes the OData query with the right `$select` fields
5. It writes error handling that matches the observed error shape

**Without the doc**, step 2 fails and Copilot guesses `email` → `raw.email` → runtime crash.

---

## Internal API Docs

Alongside external API docs, document your own API endpoints in `docs/apis/`:

```
docs/apis/
├── auth/
│   └── login.api.md           ← POST /auth/login
├── orders/
│   ├── create.api.md          ← POST /orders
│   └── get-by-id.api.md       ← GET /orders/:id
└── wrappers/
    └── dynamics-wrapper.md    ← All wrapper methods documented
```

Use `/generate-api-doc` to auto-generate from existing code, or `/update-api-doc` after changes.

---

## Real Example: Documenting Dynamics Rate Limits

Don't just write "Dynamics has rate limits". Document the exact behavior:

| Limit | Value | Behavior when exceeded |
|:---|:---|:---|
| Requests per 5 min | 6000 | HTTP 429, Retry-After header (seconds to wait) |
| Concurrent requests | 52 | Request is queued, eventually completes |

## Our Retry Strategy
Wrapper uses exponential backoff: 1s → 2s → 4s, max 3 retries.
429 response: read `Retry-After` header, wait exact seconds, then retry once.
```

This goes in the README for the Dynamics folder. The wrapper uses it, tests verify it.

---

## Summary

| What | Where | Why |
|:---|:---|:---|
| External API auth + base URL | `docs/external-apis/[api]/README.md` | Wrapper reads it before making calls |
| Entity field name mappings | `docs/external-apis/[api]/[entity].api.md` | Transformer reads it to map fields correctly |
| Your own internal endpoints | `docs/apis/[domain]/[endpoint].api.md` | Other developers and Copilot know what the API does |
| Wrapper methods reference | `docs/apis/wrappers/[name]-wrapper.md` | Prevents duplicate wrapper methods being created |

**Next: Your Daily Developer Workflow →** [05-daily-workflow.md](./05-daily-workflow.md)

---

## Further Reading

- [OData Query Syntax](https://learn.microsoft.com/en-us/odata/concepts/queryoptions-overview) — ``, ``, `` query options used in Dynamics examples
- [Microsoft Dynamics 365 Web API reference](https://learn.microsoft.com/en-us/power-apps/developer/data-platform/webapi/reference/entitytypes) — entity type reference for Dynamics
- [Postman](https://www.postman.com/) — the recommended tool for making real API calls and capturing actual response field names
- Example template: [`docs/templates/external-api-doc-template.md`](../docs/templates/external-api-doc-template.md) — copy this and fill in for each external API
---

[← Part 03: API Architecture](./03-api-architecture.md) · 📚 [Learn Series](../index.md) · [Part 05: Daily Developer Workflow →](./05-daily-workflow.md)
