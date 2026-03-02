---
layout: default
parent: "Copilot Team Workflow"
title: "Part 09 - Test-Driven Development (TDD)"
description: "How to follow the Red-Green-Refactor ritual and avoid common testing anti-patterns."
nav_order: 10
---

# Learn: Test-Driven Development (TDD)

[← Part 08: Using as Boilerplate](./08-boilerplate-setup.md) · 📚 [Learn Series](../index.md) · [Part 10: Subagent-Driven Development →](./10-subagent-driven-development.md)

---

## Why TDD?

Most developers treat testing as an "afterthought" or a verification step. In the Copilot workflow, **TDD is a design tool**.

| Reason | The Reality |
|:---|:---|
| **Design Specification** | The test is a "wish list" of how your API should look. It forces you to think about usability before implementation. |
| **Hallucination Guard** | Copilot easily hallucinates field names. A failing test proves your code actually works, rather than just "looking right." |
| **Refactoring Safety** | With green tests, you can refactor or optimize code with 100% confidence. |
| **Documentation** | Tests are living documentation that never goes stale. |

---

## The Iron Law

> **NO PRODUCTION CODE WITHOUT A FAILING TEST FIRST.**

If you wrote code before the test: **Delete it. Start over.**
Thinking "skip TDD just this once"? No. That's a rationalization that leads to technical debt and runtime bugs.

---

## The TDD Ritual: Red-Green-Refactor

### 1. 🔴 RED — Write a Failing Test
Write a minimal test for a single behavior.
- **Mandatory**: Run the test and watch it fail.
- **Why**: If you didn't watch it fail, you don't know if the test actually verifies the behavior or if it's passing for the wrong reasons.

### 2. 🟢 GREEN — Minimal Code
Write the simplest code to make the test pass.
- **Minimal**: No extra features, no "I might need this later" (YAGNI).
- **Goal**: Get to green as fast as possible.

### 3. 🔵 REFACTOR — Clean Up
Now that you're green, clean up the mess.
- Improve variable names.
- Remove duplication (DRY).
- Extract helpers.
- **Rule**: Keep the tests green during this phase.

---

## When to Use TDD

- **New Features**: Always.
- **Bug Fixes**: Reproduce the bug with a test FIRST.
- **Refactoring**: Use existing tests to ensure no regressions.
- **Behavior Changes**: Update the test to reflect the new desired behavior.

---

## Real-World Example: Fixing a Validation Bug

**Scenario**: A user reports that an empty email allows them to submit a form.

### Step 1: RED (Reproduce the Bug)
```typescript
test('rejects empty email with "Email required" error', async () => {
  const result = await submitForm({ email: '' });
  expect(result.error).toBe('Email required');
});
```
*Run test: FAIL (Expected "Email required", got undefined).*

### Step 2: GREEN (Fix it)
```typescript
function submitForm(data: FormData) {
  if (!data.email?.trim()) {
    return { error: 'Email required' };
  }
  // implementation...
}
```
*Run test: PASS.*

### Step 3: REFACTOR
Ensure the error message is a constant and clean up the validation logic if more fields are added.

---

## Common Rationalizations (And Why They're Wrong)

| Excuse | The Hard Truth |
|:---|:---|
| "Too simple to test" | Simple code is where silly bugs hide. A test takes 30 seconds. |
| "I'll test after" | Tests written after code are biased. You'll test what you built, not what was required. |
| "Already manually tested" | Manual tests aren't repeatable. You'll have to do it again for every future change. |
| "TDD slows me down" | TDD is faster than debugging in production. It feels slower because you're catching bugs *now* instead of later. |

---

## Testing Anti-Patterns (Avoid These!)

### ❌ 1. Testing Mock Behavior
Don't assert that `expect(mock).toHaveBeenCalled()`. Assert on the **actual side effect** or the **returned result**. Mocks are tools to isolate, not things to test.

### ❌ 2. Test-Only Methods
Never add `public revealInternalState()` to a production class just for a test. Use public APIs or test utilities to verify state.

### ❌ 3. Incomplete Mocks
Always mock the **COMPLETE** data structure. If an API returns an object with 10 fields, don't mock 2. Downstream code might rely on the other 8 and crash in production.

---

## Summary

| Phase | Action |
|:---|:---|
| **RED** | Prove the feature is missing or the bug exists. |
| **GREEN** | Implement the simplest fix. |
| **REFACTOR** | Clean up the design while staying green. |

**You have finished the expanded TDD guide!**

---

[← Part 08: Using as Boilerplate](./08-boilerplate-setup.md) · 📚 [Learn Series](../index.md) · [Part 10: Subagent-Driven Development →](./10-subagent-driven-development.md)
