# Docs: APIs Folder

This folder documents **your own project's internal API endpoints and wrapper methods**.

## Structure

```
docs/apis/
├── README.md                      ← this file
├── auth/
│   ├── login.api.md               ← POST /auth/login
│   └── refresh-token.api.md       ← POST /auth/refresh
├── orders/
│   ├── create.api.md              ← POST /orders
│   └── get-by-id.api.md           ← GET /orders/:id
└── wrappers/
    ├── dynamics-wrapper.md        ← All DynamicsWrapper methods
    └── stripe-wrapper.md          ← All StripeWrapper methods
```

## When to Write an API Doc

- When you **create** a new endpoint → write the doc immediately (`/generate-api-doc`)
- When you **modify** an endpoint → update the doc in the same commit (`/update-api-doc`)
- When you **create** a new wrapper method → add it to the relevant wrapper doc

## How to Create a New API Doc

```bash
cp docs/templates/api-doc-template.md docs/apis/[domain]/[endpoint].api.md
```

Or use the slash command — Copilot will generate it from your code:
```
/generate-api-doc
```

## Example

See [`auth/login.api.example.md`](./auth/login.api.example.md) for a complete endpoint doc.
