# Order Flow

> **Type**: User Journey Flow
> **Scope**: From order creation to delivery tracking
> **Last updated**: 2026-03-02

---

## Overview

The order flow covers the complete lifecycle of an order in the system — from when a customer places an order to when it is delivered and tracked.

```
Customer → Create Order → Payment → Fulfillment → Shipping → Tracking
```

---

## Step-by-Step Flow

### 1. Create Order
- **Trigger**: Customer clicks "Place Order" in the frontend
- **Endpoint**: `POST /orders`
- **Controller**: `src/controllers/order.controller.ts`
- **Service**: `src/services/order.service.ts`
- **What happens**:
  1. Validate cart items still in stock
  2. Calculate total (including tax and shipping estimate)
  3. Create order record in DB with `status: pending_payment`
  4. Return `orderId` to frontend

### 2. Payment
- **Trigger**: Customer completes checkout with Stripe
- **Endpoint**: `POST /payments/checkout` + Stripe webhook `POST /webhooks/stripe`
- **What happens**:
  1. Frontend calls `/payments/checkout` to create a Stripe checkout session
  2. Customer completes payment on Stripe's hosted page
  3. Stripe sends `payment_intent.succeeded` webhook
  4. Webhook handler updates order `status: paid`
  5. Triggers fulfillment flow

### 3. Fulfillment
- **Trigger**: Order `status` changes to `paid`
- **What happens**:
  1. Order synced to Dynamics 365 CRM via `DynamicsWrapper.createOrder()`
  2. Warehouse picks and packs items
  3. Dynamics updates order `status: dispatched` → webhook received
  4. Our system updates order `status: shipped`

### 4. Shipping & Tracking
- **Trigger**: Order `status` changes to `shipped`
- **What happens**:
  1. Tracking number stored on the order record
  2. Customer receives email with tracking link
  3. Customer can call `GET /orders/:id/tracking` to get live status

---

## Status State Machine

```
pending_payment → paid → processing → dispatched → shipped → delivered
                    ↓
                cancelled (only before dispatched)
```

---

## Key Files

| File | Role |
|------|------|
| `src/controllers/order.controller.ts` | HTTP entry point |
| `src/services/order.service.ts` | Business logic |
| `src/wrappers/dynamics-wrapper.ts` | Dynamics CRM integration |
| `src/transformers/order.transformer.ts` | Dynamics field mapping |
| `src/controllers/webhook.controller.ts` | Stripe webhook handler |

---

## External APIs Involved

- **Stripe**: Payment processing — see `docs/external-apis/stripe/`
- **Dynamics 365**: Order sync — see `docs/external-apis/dynamics/orders.api.md`
