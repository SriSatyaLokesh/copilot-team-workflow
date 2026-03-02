# Docs: External APIs Folder

This folder documents **every external/third-party API** your project calls.

**This is the most important folder for Copilot accuracy.** When Copilot reads these docs before writing wrapper code, it uses the correct field names — no guessing, no runtime crashes.

## Structure

```
docs/external-apis/
├── README.md                        ← this file
├── dynamics/                        ← Microsoft Dynamics 365 CRM
│   ├── README.md                    ← Auth, base URL, rate limits, gotchas
│   ├── accounts.api.md              ← Account entity field mapping
│   ├── contacts.api.md              ← Contact entity field mapping
│   └── orders.api.md                ← Order entity field mapping
├── stripe/                          ← Stripe Payments
│   ├── README.md                    ← Auth, webhooks, test mode
│   └── payment-intent.api.md        ← PaymentIntent field mapping
└── sendgrid/                        ← SendGrid Email
    ├── README.md                    ← Auth, rate limits
    └── send-email.api.md            ← Send email field mapping
```

## Rule: Document Before Coding

> **Never write a wrapper method before filling in the entity field mapping table.**

The field mapping table is what Copilot reads to know that `emailaddress1` (Dynamics) maps to your internal `email` field. Without it, Copilot guesses — and gets it wrong.

## How to Add a New External API

```bash
mkdir -p docs/external-apis/[api-name]
cp docs/templates/external-api-doc-template.md docs/external-apis/[api-name]/README.md
cp docs/templates/external-api-doc-template.md docs/external-apis/[api-name]/[entity].api.md
```

Fill in the field mapping table FIRST — from the real API response. Then write the wrapper code using `/add-new-api`.

## Example

See [`dynamics/accounts.api.example.md`](./dynamics/accounts.api.example.md) for a complete entity doc with field mapping.
