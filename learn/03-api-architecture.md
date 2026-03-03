---
layout: default
parent: "Copilot Team Workflow"
title: "Part 03 - API Architecture"
description: "Controller → Service → Wrapper → Transformer layer rules and why they matter for Copilot accuracy."
nav_order: 4
---

# Learn: API Architecture — The Layer Rules


---

## Why Architecture Rules Matter for Copilot

When you ask Copilot to "add a Dynamics CRM call to the login flow", it needs to know:
- Where does the HTTP call go? (wrapper)
- Where does the business logic go? (service)
- Where does auth happen? (resolver/controller)
- How do field names get translated? (transformer)

Without these rules in `api-architecture.instructions.md`, Copilot puts HTTP calls in services, business logic in wrappers, and the codebase becomes inconsistent.

With the rules loaded automatically when you open any wrapper or service file, Copilot always knows exactly which layer to put code in — without you having to explain it every session.

---

## The Request Lifecycle

Every API request in this project travels through exactly these layers:

```
① HTTP Request arrives
        ↓
② Controller / GraphQL Resolver
        ↓
③ Service
        ↓
④ API Wrapper
        ↓
⑤ External API (3rd party)
        ↓
⑥ API Wrapper (receives response)
        ↓
⑦ Transformer
        ↓
⑧ Service (receives clean data)
        ↓
⑨ Controller / Resolver
        ↓
⑩ HTTP Response sent
```

Authentication + Authorization happens at ② — before any business logic.

---

## Layer 1: Controller / Resolver

**File location**: `src/controllers/` or `src/resolvers/`
**Job**: Entry point. Parse, validate, authorize.

### What belongs here
- Parse the raw HTTP request / GraphQL args
- Validate with a schema (zod, class-validator)
- Check authentication (`req.user` must exist if protected)
- Check authorization (role/permission check)
- Call one service method
- Return the result

### What does NOT belong here
- Business logic (doesn't belong here)
- Database queries (doesn't belong here)  
- External API calls (never directly)

```typescript
// ✅ Correct controller
export const loginController = async (req: Request, res: Response) => {
  const body = loginSchema.parse(req.body)        // ① validate
  // (no auth check needed — this is the login endpoint)
  const result = await authService.login(body)    // ② call service
  return res.json(result)                         // ③ return
}

// ❌ Wrong — business logic in controller
export const loginController = async (req: Request, res: Response) => {
  const user = await db.users.findOne({ email: req.body.email })
  if (!user || !bcrypt.compare(req.body.password, user.passwordHash)) {
    return res.status(401).json({ error: 'Invalid credentials' })
  }
  // ... this is service logic, not controller logic
}
```

---

## Layer 2: Service

**File location**: `src/services/`
**Job**: Business logic. Orchestrate everything.

### What belongs here
- Business rules and decisions
- Conditional logic ("if customer is inactive, throw error")
- Calling wrapper methods
- Combining data from multiple sources
- Transformations of business meaning (not field names — that's the transformer)

### What does NOT belong here
- HTTP calls (use the wrapper)
- Response formatting for HTTP (that's the controller)
- Raw field name mapping (that's the transformer)

```typescript
// ✅ Correct service
export class AuthService {
  async login(input: LoginInput): Promise<LoginResult> {
    // Business rule: check rate limit before anything
    await this.rateLimiter.check(input.email)

    // Call wrapper — gets clean internal Customer object back
    const customer = await this.dynamicsWrapper.getCustomerByEmail(input.email)
    
    // Business rule: inactive customers can't log in
    if (!customer.isActive) {
      throw new ForbiddenError('Account is inactive. Contact support.')
    }

    const token = this.jwtService.sign({ customerId: customer.id })
    return { token, customer }
  }
}
```

---

## Layer 3: API Wrapper

**File location**: `src/wrappers/`
**Job**: One wrapper per external API. Handles HTTP and nothing else.

### What belongs here
- Base URL and auth token management
- Making HTTP calls (axios, fetch)
- Refreshing tokens when they expire
- Retry logic on 429 / 503
- Calling the transformer on the response
- Throwing typed errors (NotFoundError, ExternalApiError)

### What does NOT belong here
- Business logic
- Deciding what to call (that's the service)

```typescript
// ✅ Correct wrapper
export class DynamicsWrapper {
  async getCustomerByEmail(email: string): Promise<Customer> {
    const token = await this.getOrRefreshToken()  // handles expiry
    
    const response = await axios.get(`${this.baseUrl}/contacts`, {
      headers: { Authorization: `Bearer ${token}`, ...this.defaultHeaders },
      params: {
        '$filter': `emailaddress1 eq '${email}'`,
        '$select': 'contactid,fullname,emailaddress1,statecode'
      }
    })
    
    if (response.data.value.length === 0) throw new NotFoundError()
    
    return customerTransformer.fromDynamics(response.data.value[0])
    //                         ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    //                         Always transform before returning
  }
}
```

---

## Layer 4: Transformer

**File location**: `src/transformers/`
**Job**: Field name mapping. Pure functions. No business logic.

### What belongs here
- Map external field names → internal field names
- Convert types (string date → Date, enum number → boolean)
- Strip fields the service doesn't need

### What does NOT belong here
- Business logic
- Decisions of any kind

```typescript
// ✅ Correct transformer
export const customerTransformer = {
  fromDynamics: (raw: DynamicsContact): Customer => ({
    id: raw.contactid,              // contactid → id
    name: raw.fullname,             // fullname → name
    email: raw.emailaddress1,       // emailaddress1 → email (external name is horrible)
    isActive: raw.statecode === 0,  // 0=Active, 1=Inactive enum → boolean
  }),

  toDynamics: (c: CreateCustomerInput): Partial<DynamicsContact> => ({
    fullname: c.name,
    emailaddress1: c.email,
  })
}
```

---

## Why the Transformer Is Not Optional

External APIs have terrible field names. Microsoft Dynamics uses:
- `emailaddress1` (why the "1"?)
- `_primarycontactid_value` (leading underscore, trailing "value")
- `statecode` (0 means active, 1 means inactive)
- `createdon` (past tense, no separator)

If you let these leak into your service or controller layer, every developer has to know Dynamics field names. Tests become unreadable. When you change APIs, you touch every layer.

The transformer is a **firewall**: internal code never sees external field names.

---

## Adding a New External API Call: The Order

Always implement in this order. Never code bottom-up.

```
Step 1: Read docs/external-apis/[api-name]/[entity].api.md
        (field names, auth, query patterns)

Step 2: Transformer
        - Define external type (exactly as API returns it)
        - Define internal type (clean, business-friendly)
        - Write fromExternal() and toExternal()
        - Write transformer unit tests (pure functions, no mocks needed)

Step 3: Wrapper method
        - HTTP call with correct headers
        - Call transformer on response
        - Handle errors

Step 4: Service method
        - Business logic using wrapper

Step 5: Controller/Resolver endpoint
        - Validation + auth + call service

Step 6: Update docs
        - docs/apis/wrappers/ and docs/apis/[domain]/[endpoint].api.md
```

> Use `/add-new-api` prompt to have Copilot walk through this automatically.

---

## The Architecture in One Rule

> **Copilot's rule (from `api-architecture.instructions.md`)**:
> 
> Every external API call goes: Service → Wrapper → (External API) → Transformer → Service.
> 
> No exceptions. No shortcuts.

---

## Summary

| Layer | File location | Does | Does NOT |
|:---|:---|:---|:---|
| Controller | `src/controllers/` | Validate, auth, call service | Business logic |
| Service | `src/services/` | Business logic, orchestrate | HTTP calls |  
| Wrapper | `src/wrappers/` | HTTP + transform | Decisions |
| Transformer | `src/transformers/` | Field name mapping | Logic |

**Next: Documenting External APIs →** [04-documenting-external-apis.md](./04-documenting-external-apis.md)

---

## Further Reading

- [VS Code Copilot Custom Instructions](https://code.visualstudio.com/docs/copilot/copilot-customization) — how `applyTo` globs load architecture rules automatically
- [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/intro.html) — if you're new to the TypeScript syntax used in the code examples
- [Zod documentation](https://zod.dev/) — the validation library used in Controller examples
- [Repository Pattern](https://martinfowler.com/eaaCatalog/repository.html) — the design pattern that informs the Wrapper layer
---

[← Part 02: The 5-Phase Workflow](./02-five-phase-workflow.md) · 📚 [Learn Series](/) · [Part 04: Documenting External APIs →](./04-documenting-external-apis.md)
