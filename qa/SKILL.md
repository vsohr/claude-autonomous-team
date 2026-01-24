---
name: qa
description: Verify a built application meets its specification. Use after implementation is complete to check startup, generate E2E tests from acceptance criteria, run them, and produce a verification report. Simplified verification focused on functional correctness - Product Owner handles final intent alignment.
---

# QA Verification

Verify the built application works correctly against its specification.

## Core Principle

Prove the app works. Don't assume. Run it, test it, report what you find.

---

## Process

### Step 1: Startup Verification

**Goal:** Confirm the app runs without errors.

```bash
# 1. Detect project type
ls package.json  # Node.js
ls requirements.txt  # Python
ls Cargo.toml  # Rust
ls go.mod  # Go

# 2. Install dependencies
npm install  # or pip install, cargo build, etc.

# 3. Start the application
npm run dev  # or appropriate start command

# 4. Verify accessibility
curl http://localhost:<port>  # or appropriate health check
```

**Check for:**
- [ ] No startup errors in console
- [ ] App responds to requests
- [ ] No unhandled promise rejections or uncaught exceptions
- [ ] Environment variables/config loaded correctly

**If startup fails:** Stop. Report the failure. Don't proceed with tests.

**Setup testing tools if missing:**
- If Playwright not installed: `npm init playwright@latest`
- If test runner not configured: Set up appropriate test framework
- Configure `playwright.config.ts` with project's dev server URL

---

### Step 2: Generate E2E Tests

**Input:** SPEC.md acceptance criteria

**For each acceptance criterion, write a Playwright test:**

```typescript
import { test, expect } from '@playwright/test';

// Maps to: Feature X, Criterion Y
test('should [acceptance criterion description]', async ({ page }) => {
    // Arrange: Navigate to relevant page/state
    await page.goto('/relevant-path');

    // Act: Perform the user action
    await page.click('[data-testid="action-button"]');

    // Assert: Verify the expected outcome
    await expect(page.locator('[data-testid="result"]')).toBeVisible();
});
```

**Test generation rules:**
- One test per acceptance criterion
- Use `data-testid` attributes for selectors (add to code if missing)
- Each test is independent (no shared state between tests)
- Include the spec reference in a comment
- Test observable behavior, not implementation details
- Keep tests simple: navigate → act → assert

**Test file location:** `e2e/spec-verification.spec.ts` (or project's E2E test directory)

---

### Step 3: Run E2E Tests

```bash
npx playwright test e2e/spec-verification.spec.ts
```

**Collect:**
- Pass/fail status per test
- Error messages for failures
- Screenshots on failure (Playwright does this automatically)

**If tests fail:**
- Do NOT fix the code
- Do NOT modify tests to pass
- Report exactly what fails and why
- Include error messages and screenshots

---

### Step 4: Functional Smoke Test

Beyond E2E tests, manually verify critical paths:

1. **Happy path:** Can a user complete the primary workflow start to finish?
2. **Error handling:** Do errors show helpful messages (not stack traces)?
3. **Empty states:** Does the app handle no-data gracefully?
4. **Navigation:** Can the user reach all documented features?

Use the app through its interface. Document what you find.

---

### Step 5: Basic Quality Checks

Run available automated checks:

```bash
# TypeScript type checking (if applicable)
npx tsc --noEmit

# Linting (if configured)
npm run lint

# Unit tests (if they exist)
npm test
```

**Report:** Pass/fail for each check. Don't fix issues, just report.

---

### Step 6: Produce Verification Report

Write `VERIFICATION.md`:

```markdown
# Verification Report

## Summary
- **Status:** PASS / FAIL
- **Date:** [Date]
- **Tests:** [X/Y passing]

## Startup
- **Status:** ✅ Running / ❌ Failed
- **Notes:** [Any startup issues]

## Spec Compliance

| # | Feature | Criterion | Test | Status | Notes |
|---|---------|-----------|------|--------|-------|
| 1 | [Feature] | [Criterion from spec] | [test name] | ✅/❌ | [Details] |
| 2 | [Feature] | [Criterion from spec] | [test name] | ✅/❌ | [Details] |

## Smoke Test Results

| Path | Status | Notes |
|------|--------|-------|
| Happy path | ✅/❌ | [What happened] |
| Error handling | ✅/❌ | [What happened] |
| Empty states | ✅/❌ | [What happened] |
| Navigation | ✅/❌ | [What happened] |

## Quality Checks

| Check | Status | Issues |
|-------|--------|--------|
| TypeScript | ✅/❌ | [Count of errors] |
| Linting | ✅/❌ | [Count of warnings/errors] |
| Unit tests | ✅/❌ | [X/Y passing] |

## Issues Found

### Issue 1: [Title]
- **Severity:** Critical / Important / Minor
- **Location:** [Where in the app]
- **Expected:** [What should happen]
- **Actual:** [What happens]
- **Screenshot:** [If applicable]

## Determination

**PASS** if:
- App starts without errors
- All acceptance criteria tests pass
- Happy path works end-to-end
- No critical issues found

**FAIL** if:
- App doesn't start
- Any acceptance criterion test fails
- Happy path is broken
- Critical issues found
```

---

## What This Skill Does NOT Do

- **Visual quality scoring** — Product Owner evaluates polish and intent
- **Performance benchmarking** — Only basic "does it load" check
- **Accessibility audits** — Can be added later if needed
- **Cross-browser testing** — Single browser (Chromium) is sufficient for verification

---

## Integration Points

- **Input:** Working code + SPEC.md (with acceptance criteria)
- **Output:** VERIFICATION.md
- **Consumed by:** Product Owner (for ship/no-ship decision)
- **If FAIL:** Product Owner produces GAPS.md and routes back to builder

---

## Red Flags

**Never:**
- Fix code during verification (you're the tester, not the builder)
- Modify tests to make them pass
- Skip tests that are hard to automate (report as manual check)
- Declare PASS when tests are failing

**Always:**
- Run the actual app (don't just read code)
- Test against the spec (not against what the code does)
- Report honestly (failed is better than falsely passed)
- Include enough detail to reproduce issues
