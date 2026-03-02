# Docs: Flows Folder

This folder documents **user journeys** — the business flows that span multiple features, controllers, and services.

## When to Write a Flow Doc

Write a flow doc when:
- A user journey involves 3+ steps across different parts of the API
- You need to explain a complex sequence to a new developer
- Copilot needs to understand a flow before planning a feature that touches it

## Naming Convention

```
auth-flow.md          ← login, register, refresh token, logout
order-flow.md         ← create order → payment → fulfillment → tracking
quotation-flow.md     ← create quote → send → approve → convert to order
signup-flow.md        ← user registration end to end
```

## How to Create a New Flow Doc

```bash
cp docs/templates/flow-doc-template.md docs/flows/your-flow-name.md
```

## How Copilot Uses These

When starting an Issue that touches a flow (e.g., "add order status webhook"), reference the flow doc:

```
"Read #docs/flows/order-flow.md to understand the order lifecycle before planning."
```

## Example

See [`order-flow.example.md`](./order-flow.example.md) for a complete example.
