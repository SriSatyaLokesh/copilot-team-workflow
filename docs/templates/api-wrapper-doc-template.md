---
wrapper-name: "[WrapperClassName, e.g., DynamicsWrapper]"
external-api: "[External API name]"
external-api-doc: "docs/external-apis/[api-name]/README.md"
file: "src/wrappers/[name]-wrapper.ts"
last-updated: "YYYY-MM-DD"
---

# [WrapperName] — API Wrapper Reference

> This wrapper handles all HTTP communication with **[External API Name]**.
> It is the ONLY place in the codebase that should directly call this API.

---

## Architecture Position

```
Service → [WrapperName] → [External API] → Transformer → Service
```

The wrapper:
- ✅ Manages authentication (token storage + refresh)
- ✅ Makes HTTP calls with correct headers
- ✅ Handles retries and transport errors
- ✅ Calls transformers before returning data
- ❌ Does NOT contain business logic
- ❌ Does NOT decide which method to call

---

## External API Reference

Full external API docs: [docs/external-apis/[api-name]/README.md](../external-apis/[api-name]/README.md)

**Base URL**: `[base-url-from-external-api-doc]`
**Auth Type**: `[e.g., OAuth2 Bearer Token]`
**Env Vars Required**:
- `[API_NAME]_BASE_URL`
- `[API_NAME]_CLIENT_ID`
- `[API_NAME]_CLIENT_SECRET`

---

## Class Location

```
src/wrappers/[name]-wrapper.ts
  └── class [WrapperName]
       ├── constructor(config: WrapperConfig)
       ├── private getAuthToken(): Promise<string>
       └── [public methods listed below]
```

---

## Methods

### `get[Entity](id: string): Promise<[InternalType]>`

**What it does**: Fetches a single [entity] by ID from [External API].

**Calls**:
```
GET [external-endpoint-url]
Headers: Authorization: Bearer <token>
         [other required headers]
```

**Returns**: `[InternalType]` — the external data transformed to our internal shape

**Throws**:
- `NotFoundError` — when external API returns 404
- `ExternalApiError` — for all other HTTP errors

**Transformer used**: `[entityName]Transformer.from[ApiName]()`

**Example usage** (from a service):
```typescript
const customer = await this.dynamicsWrapper.getCustomer('12345-...')
// customer is already transformed: { id, name, email, isActive }
```

---

### `list[Entities](filters?: [FilterType]): Promise<[InternalType][]>`

**What it does**: Fetches a list of [entities] with optional filtering.

**Calls**:
```
GET [external-endpoint-url]
?$select=[field1],[field2]
&$filter=[odata-filter-if-any]
&$top=[default-page-size]
```

**Supported filters**:
| Filter | External OData | Description |
|--------|---------------|-------------|
| `isActive` | `statecode eq 0` | Only active records |
| `ownerId` | `_ownerid_value eq [guid]` | Records owned by user |
| `createdAfter` | `createdon ge [date]` | Created after date |

**Returns**: `[InternalType][]`

---

### `create[Entity](data: Create[Entity]Input): Promise<{ id: string }>`

**What it does**: Creates a new [entity] in [External API].

**Calls**:
```
POST [external-endpoint-url]
Body: [external-field-name]: value,
      [external-field-name]: value
```

**Field mapping** (internal → external):
| Our Input Field | External Field |
|----------------|---------------|
| `name` | `name` |
| `email` | `emailaddress1` |
| [add all fields] | |

**Returns**: `{ id: string }` — the new record's ID from the `OData-EntityId` header

---

### `update[Entity](id: string, data: Update[Entity]Input): Promise<void>`

**What it does**: Updates an existing [entity] in [External API].

**Calls**:
```
PATCH [external-endpoint-url]/([id])
Body: [only the fields being updated]
```

**Returns**: `void` — external API returns 204

> ⚠️ **Gotcha**: External API does not return the updated record.
> If you need the updated data, call `get[Entity](id)` after the PATCH.

---

### [Add more methods as they are created]

---

## Error Handling

This wrapper normalizes all external API errors:

| External Error | Wrapper Throws | When |
|---------------|---------------|------|
| 404 | `NotFoundError` | Record does not exist |
| 401 | Retries once after refresh | Token expired |
| 429 | Retries with backoff | Rate limited |
| 4xx (other) | `ExternalApiError(statusCode, message)` | Bad request |
| 5xx | `ExternalApiError(statusCode, message)` + log | Server error |

---

## Transformer Reference

**File**: `src/transformers/[name]-transformer.ts`

| Method | Input Type | Output Type | Used By |
|--------|-----------|-------------|---------|
| `from[ApiName]([raw])` | External response shape | Internal type | All GET methods |
| `to[ApiName](data)` | Internal type | External request shape | POST/PATCH methods |

---

## Testing

**Unit test file**: `src/wrappers/[name]-wrapper.test.ts`

Pattern for mocking this wrapper in service tests:
```typescript
jest.mock('src/wrappers/[name]-wrapper')
const mockWrapper = [WrapperName] as jest.Mocked<typeof [WrapperName]>
mockWrapper.getCustomer.mockResolvedValue({ id: '1', name: 'Test', isActive: true })
```

---

## Change History

| Date | Method | Change | Developer | Issue |
|------|--------|--------|-----------|-------|
| YYYY-MM-DD | `getCustomer` | Initial implementation | [Name] | ISSUE-001 |
