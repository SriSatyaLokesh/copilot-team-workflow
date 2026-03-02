---
issue-id: "ISSUE-001"
title: "User Login with Rate Limiting"
status: "done"
branch: "issue/ISSUE-001-login-rate-limiting"
created: "2026-03-01"
completed: "2026-03-02"
developer: "sri-lokesh"
---

# ISSUE-001: User Login with Rate Limiting

> **This is an example Issue doc.** Copy `docs/templates/issue-template.md` to create your own.
> Name your file: `ISSUE-XXX-short-name.md` where XXX is your issue number.

---

## Phase 1: Discuss ✅

**Requirements agreed:**
- Login endpoint must rate-limit after 5 failed attempts
- Lockout window is 15 minutes per email address
- Admin users (role: `admin`) are exempt from rate limiting
- After lockout, return HTTP 429 with a `Retry-After` header

**Acceptance criteria:**
1. A user who fails login 5 times within 15 minutes receives a 429 response
2. The same user can successfully login after 15 minutes
3. An admin user is never rate-limited regardless of failed attempts
4. The 429 response includes a `Retry-After` header with seconds remaining

**Out of scope:**
- Per-IP rate limiting (only per email)
- Permanent account lockout

---

## Phase 2: Research ✅

**Files to modify:**
- `src/middleware/rate-limit.ts` ← create new
- `src/api/auth/route.ts` ← apply middleware

**Existing patterns to follow:**
- OTP rate limiting: `src/middleware/otp-rate-limit.ts`
  - Use the same Redis key pattern: `rate:${type}:${identifier}`
  - Use the same `rateLimiter` utility from `src/utils/rate-limiter.ts`

**Related schemas:**
- `users` table → `src/db/schema/users.ts` (need `role` field for admin check)
- No new tables needed — use Redis for rate limit state

**Risk:**
- Must check `user.role === 'admin'` BEFORE applying rate limit check

---

## Phase 3: Plan ✅

**Architecture decision:**
Use existing Redis client and `rate:${type}:${identifier}` key pattern from `otp-rate-limit.ts`.
Window: 15 minutes. Max attempts: 5.

**Implementation checklist:**
- [x] Test: rate limit middleware — 5 attempts then block (unit)
- [x] Implement: `src/middleware/rate-limit.ts`
- [x] Test: admin bypass — admin is never rate limited (unit)
- [x] Implement: admin role check before rate limit in middleware
- [x] Test: `POST /auth/login` with rate limiting (integration)
- [x] Apply: middleware to login route in `src/api/auth/route.ts`

---

## Phase 4: Execute ✅

**Progress:**
- [x] `src/middleware/rate-limit.ts` — created
- [x] `src/api/auth/route.ts` — middleware applied
- [x] `src/middleware/rate-limit.test.ts` — 8 unit tests
- [x] `src/api/auth/route.test.ts` — 4 integration tests

**Commits:**
- `abc1234` — test: ISSUE-001 rate limit middleware unit tests
- `def5678` — feat: ISSUE-001 rate limit middleware (red→green)
- `ghi9012` — test: ISSUE-001 admin bypass unit tests
- `jkl3456` — feat: ISSUE-001 admin bypass (red→green)
- `mno7890` — test: ISSUE-001 login route integration tests
- `pqr1234` — feat: ISSUE-001 apply rate limiting to login route

---

## Phase 5: Verify ✅

**Requirements: 4/4 ✅**
- ✅ Rate limit after 5 failed attempts
- ✅ 15-minute lockout window
- ✅ Admin bypass working
- ✅ 429 with `Retry-After` header

**Tests: 12/12 ✅**
- Unit: 8/8 ✅
- Integration: 4/4 ✅

**Verdict: ✅ READY FOR PR**
