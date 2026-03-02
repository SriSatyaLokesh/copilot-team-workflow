# Docs Folder Architecture

> **Purpose**: This file explains the `docs/` folder structure for new developers and AI agents.
> When starting any work, reference this file to know exactly where things live.

---

## The Golden Rule

**One thing = One file.** Every API endpoint, every user flow, every Issue (work item) has exactly one doc file. No exceptions.

This means:
- Copilot can load exactly the right context for any task
- Developers know exactly where to look
- There are no merge conflicts from multiple people editing the same overview file

---

## Folder Map

```
docs/
│
├── PROJECT.md                    ← What is this project? (2 pages max)
├── ARCHITECTURE.md               ← How is it built? (2 pages max)
├── CONTRIBUTING.md               ← How do we work? (1 page max)
│
├── flows/                        ← User journeys (one per major flow)
├── apis/                         ← API documentation (one per endpoint)
├── Issues/                       ← Work items / tasks (one per Issue)
├── decisions/                    ← Architectural decisions (ADRs)
├── team-notes/                   ← Developer notes (no merge conflicts)
└── templates/                    ← Templates for docs
    ├── ISSUE-template.md
    ├── api-doc-template.md
    └── flow-doc-template.md
```

---

## `docs/flows/` — User Journey Documents

### Who creates them?
Team lead creates the initial doc. Developers add to "Change History" when updating.

### What goes in them?
The complete user journey from user's perspective, including all steps, business rules, and which APIs are involved.

### When to update?
When a Issue changes the flow behavior. Update via PR, not directly to main.

### Current flows
| File | What it covers | APIs Involved |
|------|---------------|--------------|
| `auth-flow.md` | Register, login, logout, password reset | auth/* |
| `quotation-flow.md` | Quote creation, approval, revision | quotations/* |
| `order-flow.md` | Order creation through fulfillment | orders/* |
| `track-order-flow.md` | Shipment tracking, status updates | tracking/* |
| `payment-flow.md` | Payment, success, failure, refunds | payments/* |

*Add rows as new flows are built.*

---

## `docs/apis/` — API Documentation

### Structure (organized by domain)

```
docs/apis/
├── auth/
│   ├── register.api.md           POST /api/auth/register
│   ├── login.api.md              POST /api/auth/login
│   ├── logout.api.md             POST /api/auth/logout
│   ├── refresh-token.api.md      POST /api/auth/refresh
│   └── forgot-password.api.md    POST /api/auth/forgot-password
├── orders/
│   ├── create-order.api.md       POST /api/orders
│   ├── get-order.api.md          GET  /api/orders/:id
│   ├── list-orders.api.md        GET  /api/orders
│   ├── update-order.api.md       PUT  /api/orders/:id
│   └── cancel-order.api.md       DELETE /api/orders/:id
├── quotations/
│   ├── create-quotation.api.md
│   ├── get-quotation.api.md
│   ├── list-quotations.api.md
│   ├── approve-quotation.api.md
│   └── revise-quotation.api.md
├── tracking/
│   ├── track-order.api.md        GET /api/tracking/:orderId
│   └── update-tracking.api.md    POST /api/tracking/:orderId/status
├── payments/
│   ├── initiate-payment.api.md
│   ├── verify-payment.api.md
│   └── refund-payment.api.md
└── [add domain folders as needed]
```

### Naming convention

`[http-verb]-[resource].api.md` OR `[action]-[resource].api.md`

Examples:
- `create-order.api.md` (not `post-order.api.md`)
- `approve-quotation.api.md` (not `put-quotation-approval.api.md`)
- `track-order.api.md` (not `get-tracking.api.md`)

### What each API doc must have

1. ✅ Endpoint (`POST /api/orders`)
2. ✅ Purpose (1-2 sentences)
3. ✅ Authentication required? (yes/no + how)
4. ✅ Request body with field descriptions
5. ✅ Success response with example
6. ✅ All error responses as a table
7. ✅ Business rules (validation, limits, special cases)
8. ✅ Implementation pointers (file paths)
9. ✅ Related test files
10. ✅ Change history

---

## `docs/Issues/` — Work Items

### Who creates them?
The developer assigned the Issue creates it at the start.

### Naming
`ISSUE-[number]-[short-kebab-case-name].md`

Examples:
- `ISSUE-001-user-authentication.md`
- `ISSUE-042-login-rate-limiting.md`
- `ISSUE-099-quotation-pdf-export.md`

### When does it go to archive?
When the Issue is merged and verified, it stays in `Issues/` with `status: done`.
This is intentional — it's a permanent record of what was built and decided.

### Issue lifecycle in Git

1. Created on feature branch: `issue/ISSUE-042-login-rate-limiting`
2. Updated throughout all 5 phases on that branch
3. Merged to main when Issue is complete
4. Never deleted — only `status` changes to `done`

---

## `docs/decisions/` — Architecture Decision Records (ADRs)

### When to create one?
When you make a significant technical decision that will affect future work:
- Choosing a library over another
- Deciding on an architectural pattern
- Choosing a data model approach

### Naming
`ADR-[number]-[short-description].md`

Example: `ADR-001-jwt-over-sessions.md`

### Template structure
```markdown
# ADR-001: Use JWT instead of Server Sessions

## Status
Accepted

## Context
[What problem were we solving?]

## Decision
[What did we decide?]

## Consequences
[What are the trade-offs?]
```

---

## `docs/team-notes/` — Developer Notes

### Structure
```
docs/team-notes/
├── [developer-name]/
│   ├── notes-ISSUE-042.md
│   └── notes-ISSUE-043.md
└── shared/
    └── standup-template.md
```

### Key principle: Personal folders = No merge conflicts

Each developer ONLY writes to their own folder. Only they can edit their notes. This means:
- Git never creates merge conflicts in team-notes
- Developer can write freely without coordination
- Team lead reads everyone's notes for standup context

### What goes in personal notes
- Current status of your Issue
- Questions you have
- Things that blocked you
- Decisions you made and why
- Things other developers should know before working on related code

---

## How Copilot Uses This Structure

When you reference a file with `#file-name`, Copilot reads it as context.

**Best practice: always reference before asking**

Instead of:
```
"How should I implement rate limiting?"
```

Do this:
```
"Read #docs/apis/auth/login.api.md and #docs/flows/auth-flow.md.
 How should I add rate limiting to the login endpoint
 consistent with what's documented?"
```

Copilot will answer accurately based on YOUR system, not generic advice.

**For long sessions:** Reference your Issue doc at the start of every session:
```
"I'm continuing work on ISSUE-042.
 Read #docs/Issues/ISSUE-042-login-rate-limiting.md before we start."
```

---

## Keeping Docs in Sync With Code

**Rule: Code change + Doc change = Same commit**

Never commit a code change that changes API behavior without updating the API doc in the same commit. The commit message should say:

```
feat: ISSUE-042 add rate limiting to login endpoint

#docs/apis/auth/login.api.md updated:
- Added rate limit error responses
- Updated business rules section
```

**The /update-api-doc command**

After changing an API, run:
```
/update-api-doc
```

Copilot reads the code you just changed and updates the API doc automatically.
Review the changes before committing.

---

## Quick File Finder

| I'm working on... | Read this file |
|-------------------|---------------|
| The login endpoint | `docs/apis/auth/login.api.md` |
| How order flow works | `docs/flows/order-flow.md` |
| My current Issue | `docs/Issues/ISSUE-XXX-my-Issue.md` |
| Team conventions | `docs/CONTRIBUTING.md` |
| Why we use Redis | `docs/decisions/ADR-0XX-redis.md` |
| What APIs are in auth flow | `docs/flows/auth-flow.md` → Related APIs section |
