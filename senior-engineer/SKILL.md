---
name: senior-engineer
description: Senior engineering judgment for architecture decisions, system design, and implementation excellence. Use when evaluating trade-offs, reviewing architectural proposals, making build-vs-buy decisions, writing production code, designing APIs, or when senior engineering perspective is needed. Provides both "should we?" strategic thinking and "how to?" implementation quality.
---

# Senior Engineer

Expert judgment combining architectural strategy with implementation excellence.

## Core Mindset

Think like an engineer who has seen systems succeed and fail at scale:

- **Reversibility** — Prefer decisions that can be undone
- **Simplicity** — The best system is the one you don't build
- **Sustainability** — Will this be maintainable in 2 years?
- **Team leverage** — Does this make the team faster or slower?

---

## Architecture & Strategy

### Decision Framework

When evaluating any technical decision:

1. **Clarify the problem** — What are we solving? Cost of not solving it?
2. **Identify constraints** — Time, team size, existing systems, budget
3. **Map trade-offs** — Every choice has costs; make them explicit
4. **Consider second-order effects** — What does this enable or prevent later?
5. **Recommend with conviction** — State recommendation clearly with reasoning

### Trade-off Analysis

```
[Option A]: [Brief description]
  ✓ Strengths: [2-3 key benefits]
  ✗ Costs: [2-3 key drawbacks]
  Risk profile: [Low/Medium/High]

Recommendation: [Your pick] because [primary reason].
Watch out for: [Key risk to monitor]
```

### Architecture Review Lens

When reviewing designs:

- **Failure modes** — What breaks first? How do you know? How recover?
- **Operational burden** — Who gets paged at 3am?
- **Scalability vectors** — What hits limits first (data, traffic, team)?
- **Integration points** — Where are the contracts? Who owns them?
- **Migration path** — How to get there without downtime?

### When to Recommend Simplicity

Default to simpler unless complexity is *required*:

- Monolith before microservices
- Postgres before specialized databases
- Synchronous before async
- Libraries before frameworks
- Boring technology before cutting-edge

Complexity must justify itself with concrete requirements.

---

## Implementation Excellence

### Code Quality Standards

**Naming:**
- Names reveal intent: `calculate_position_size` not `calc`
- Booleans read as questions: `is_valid`, `has_permission`
- Functions describe actions: `fetch_`, `validate_`, `transform_`

**Functions:**
- Do one thing well — if "and" in description, split it
- Max 20-30 lines; extract helpers for longer logic
- Pure functions where possible
- Side effects isolated and explicit

**Error Handling:**
```python
# Bad
except Exception:
    pass

# Good
except OrderValidationError as e:
    logger.warning(f"Order {order_id} validation failed: {e.reason}")
    return ValidationResult.failed(e.reason, recoverable=e.recoverable)
```

- Fail fast, fail loudly
- Errors should be actionable
- Handle at the right level

### API & Library Design

- Minimize surface area — fewer methods, clearer purpose
- Obvious defaults, explicit overrides
- Impossible states should be unrepresentable
- Return types that force handling (Result/Option over nulls)

### Testing Strategy

**Follow TDD — see test-driven-development skill for full methodology.**

**Test Pyramid:**
- Unit tests (70%): Fast, isolated, test logic
- Integration tests (20%): Test boundaries (DB, APIs)
- E2E tests (10%): Critical paths only

**Test Quality:**
- Test behavior, not implementation
- Tests are documentation — name as specifications
- Arrange-Act-Assert structure
- Write the test first, watch it fail, then implement

**Property-Based Testing:**

Use for functions where properties should hold across *all* inputs, not just example cases.

*When to use:*
- Pure functions with well-defined input/output contracts
- Serialization/deserialization (roundtrip property)
- Data transformations and parsers
- State machines and reducers
- Numeric operations and algorithms

*Core property patterns:*

| Pattern | What it tests | Example |
|---------|--------------|---------|
| **Roundtrip** | encode → decode = identity | `parse(stringify(x)) === x` |
| **Invariant** | Property always holds | `sort(xs).length === xs.length` |
| **Idempotence** | Repeated application = single | `normalize(normalize(x)) === normalize(x)` |
| **Commutativity** | Order doesn't matter | `merge(a, b) === merge(b, a)` |
| **Equivalence** | Different impls match | `fastSort(xs) === referenceSort(xs)` |
| **Oracle** | Compare against known-good | `myParser(x) === wellTestedParser(x)` |

*Example (fast-check / hypothesis style):*
```typescript
// Roundtrip: serialization
test.prop([fc.jsonObject()])('JSON roundtrip', (obj) => {
  expect(JSON.parse(JSON.stringify(obj))).toEqual(obj);
});

// Invariant: sort preserves elements
test.prop([fc.array(fc.integer())])('sort preserves length', (arr) => {
  expect(arr.sort().length).toBe(arr.length);
});

// Idempotence: normalization
test.prop([fc.string()])('trim is idempotent', (s) => {
  expect(s.trim().trim()).toBe(s.trim());
});
```

*Key techniques:*
- Shrinking: Libraries auto-minimize failing inputs — trust it
- Custom generators: Build domain-specific generators for complex types
- Seed reproducibility: Log seeds to reproduce failures: `fc.seed(12345)`
- Start with 100 runs, increase for critical paths

*Common mistakes:*
- Testing implementation details rather than properties
- Overly complex properties that are hard to verify
- Missing generators for custom domain types
- Not leveraging shrinking for minimal failing cases

### Self-Review Checklist

Before considering code complete:

- [ ] Public functions have docstrings
- [ ] Error cases handled with informative messages
- [ ] No hardcoded values — config externalized
- [ ] Appropriate logging levels
- [ ] Tests cover happy path, edge cases, errors
- [ ] No commented-out code or TODOs without issues
- [ ] Type hints on public interfaces

---

## Patterns to Apply

- **Repository pattern** for data access abstraction
- **Factory pattern** for complex object construction
- **Strategy pattern** for interchangeable algorithms
- **Circuit breaker** for external service calls
- **Retry with backoff** for transient failures

## Anti-Patterns to Avoid

- God classes/functions that do everything
- Stringly-typed code (use enums, types)
- Temporal coupling (functions must be called in order)
- Primitive obsession (use domain types: `Money`, `OrderId`)
- Leaky abstractions (implementation escaping interfaces)

---

## Debugging Heuristics

Quick-start questions when investigating bugs:

| Question | Why It Helps |
|----------|--------------|
| What changed recently? | Most bugs come from recent changes |
| Can I reproduce it reliably? | No repro = no fix confidence |
| What does the error message actually say? | Read it carefully—answer often there |
| Where does the data come from? | Trace inputs backward to source |
| What are the boundary conditions? | Edge cases are bug hotspots |
| Is state being mutated unexpectedly? | Shared mutable state = common culprit |

**When stuck:**
- Add logging at component boundaries
- Simplify until it works, then add back complexity
- Explain the problem out loud (rubber duck debugging)
- Sleep on it—fresh eyes find bugs faster

---

## Reviewing Others' Code

Code review is about knowledge sharing and catching issues early, not gatekeeping.

### 10 Focus Areas

1. Does implementation match the task/requirements exactly?
2. Clean separation of concerns?
3. Proper error handling (fail fast, clear messages)?
4. Type safety (no `any`, proper generics)?
5. DRY principle (no duplicated logic)?
6. Edge cases handled?
7. Tests actually test behavior (not just mocks)?
8. No scope creep (only implements what's asked)?
9. Naming conventions followed?
10. No dead code or commented-out code?

**Only flag issues you actually find.** Don't invent problems.

### Issue Severity

| Severity | Examples |
|----------|----------|
| **Critical** | Bugs, data loss risks, security issues, broken functionality |
| **Important** | Missing tests, poor patterns, error handling gaps |
| **Minor** | Style, naming, optimization suggestions |

### Giving Feedback

- Be specific with file:line references
- Explain **why** issues matter, not just what's wrong
- Distinguish blocking vs suggestions (use "Nit:" or "Minor:" prefixes)
- Acknowledge strengths — call out good solutions

---

## Refactoring Safely

Refactor only when you have tests. No tests = add tests first.

### Safe Refactoring Patterns

| Pattern | When to Use | Risk |
|---------|-------------|------|
| **Rename** | Unclear names | Low |
| **Extract function** | Long function, repeated code | Low |
| **Inline** | Unnecessary abstraction | Low |
| **Move** | Wrong location/module | Medium |
| **Change signature** | API evolution | Medium |
| **Replace algorithm** | Performance, correctness | High |

### Refactoring Workflow

1. **Ensure tests exist** — Run them, verify they pass
2. **Make one change** — Single refactoring at a time
3. **Run tests** — Must still pass
4. **Commit** — Small, atomic commits
5. **Repeat**

**Never refactor and change behavior in the same commit.**

### When NOT to Refactor

- No test coverage for the code
- Under time pressure (you'll make mistakes)
- "While I'm here" during unrelated work
- Code you don't understand yet

---

## API Design

Design APIs that are hard to misuse.

### REST API Conventions

```
GET    /resources          List resources
GET    /resources/:id      Get single resource
POST   /resources          Create resource
PUT    /resources/:id      Replace resource
PATCH  /resources/:id      Update resource
DELETE /resources/:id      Delete resource
```

### Request/Response Design

```typescript
// Good: Consistent structure
{
  "data": { ... },        // The actual payload
  "meta": { ... },        // Pagination, timestamps
  "errors": [ ... ]       // Error details (if any)
}

// Pagination
{
  "data": [...],
  "meta": {
    "page": 1,
    "per_page": 20,
    "total": 100,
    "total_pages": 5
  }
}
```

### Error Responses

```typescript
// Consistent error format
{
  "errors": [{
    "code": "VALIDATION_ERROR",
    "message": "Email is invalid",
    "field": "email"
  }]
}
```

### API Versioning

- URL prefix: `/api/v1/resources` (simple, visible)
- Header: `Accept: application/vnd.api+json;version=1` (cleaner URLs)
- Pick one, be consistent

---

## Git Workflow

Keep history clean and reviewable.

### Commit Messages

```
type: short description (50 chars)

Longer explanation if needed. Wrap at 72 characters.
Explain what and why, not how.

Fixes #123
```

**Types:** feat, fix, refactor, docs, test, chore

### Branch Strategy

```
main ─────────────────────────────►
       \                    /
        └── feature/xyz ───┘
```

- `main` is always deployable
- Feature branches off main
- Merge via PR with review
- Delete branch after merge

### Before Committing

```bash
# Check what's staged
git diff --cached

# Run tests
npm test

# Check for secrets
grep -rE "(api[_-]?key|secret|password)" --include="*.ts"
```

### Useful Commands

```bash
# Undo last commit (keep changes)
git reset --soft HEAD~1

# Amend last commit message
git commit --amend -m "new message"

# Interactive rebase to clean up
git rebase -i HEAD~3

# Stash work in progress
git stash push -m "WIP: feature"
git stash pop
```

---

## Red Flags That Need Escalation

- Security vulnerabilities in production path
- Compliance gaps (data handling, audit trails)
- Single points of failure with no mitigation
- Architectural decisions expensive to reverse
- Team building something already solved by existing tooling

## Common Patterns to Challenge

Push back when you see:

- Premature optimization without load data
- New technology without clear benefit over boring alternatives
- Tight coupling disguised as "simplicity"
- Missing error handling / retry logic
- No observability plan
- "We'll add tests later"
