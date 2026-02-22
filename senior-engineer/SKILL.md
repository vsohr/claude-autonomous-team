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

## 10 Principles of Coding Excellence

Use these as a checklist when writing, reviewing, or refactoring code. Every principle is actionable and measurable.

### 1. Single Responsibility

Every function, class, and module should have exactly one reason to change.

- **Functions**: If description needs "and", split it. `process_order()` not `process_order_and_send_email()`
- **Classes**: One domain concept per class. `OrderValidator` not `OrderManager` (which validates, saves, notifies)
- **Files**: One cohesive concept per file. Route handlers separate from business logic separate from data access

**Test**: Can you describe what it does in one sentence without "and" or "or"?

### 2. Strict Typing

No `any`, no `Any`, no implicit types. The type system is your first line of defense.

```python
# Bad
def get_data(params: dict[str, Any]) -> Any: ...

# Good
def get_positions(account_id: str) -> list[Position]: ...
```

```typescript
// Bad
function processResult(data: any): any { ... }

// Good
function processResult(data: TradeResult): ExecutionSummary { ... }
```

- Use union types over `Any`: `str | int` not `Any`
- Use `object` for truly unknown types (Python), `unknown` (TypeScript)
- Generic types for containers: `list[Candle]` not `list`
- Frozen dataclasses for immutable domain objects

### 3. Fail Fast & Loud

Errors should be impossible to ignore. Silent failures are the hardest bugs to find.

```python
# Bad — silent swallow
try:
    result = calculate_pnl(trade)
except Exception:
    logger.error("PnL calculation failed")
    # continues with no result...

# Good — fail fast with context
try:
    result = calculate_pnl(trade)
except (ValueError, ArithmeticError) as e:
    raise PnLCalculationError(
        f"Failed to calculate PnL for trade {trade.id}: {e}"
    ) from e
```

- Catch specific exceptions, never bare `except Exception: pass`
- Validate at system boundaries (API inputs, config loading, external data)
- Re-raise with context when catching for logging purposes
- Let internal errors propagate — don't handle what you can't fix

### 4. Small Units

Small code is readable code. Enforce hard limits.

| Unit | Limit | Action When Exceeded |
|------|-------|---------------------|
| Function | 50 lines | Extract sub-operations into named helpers |
| File | 800 lines | Split by concern (see splitting patterns below) |
| Parameters | 3 max | Use a config/options object |
| Nesting depth | 3 levels | Use early returns, extract conditions |

**File splitting patterns** (proven approaches):
- Route handlers → group by API domain (accounts, trading, status)
- Mixed concerns → separate parsing, execution, orchestration
- State management → separate IO/persistence from logic
- Large clients → separate API calls, response parsing, business logic

**Always**: grep entire codebase for old import paths after splitting.

### 5. DRY Without Over-Abstraction

Eliminate duplication, but don't create abstractions for hypothetical reuse.

```python
# Bad — premature abstraction for 2 uses
class GenericDataProcessor:
    def process(self, data, transform_fn, validate_fn, output_fn): ...

# Good — extract only when pattern is proven (3+ uses)
def calculate_return_pct(pnl: Decimal, cost_basis: Decimal) -> Decimal:
    """Used by analytics, dashboard, and reporting."""
    return pnl / cost_basis if cost_basis else Decimal(0)
```

- **Rule of three**: Extract on the third duplication, not the second
- Shared helpers in a `helpers.py` or `utils.ts` within the module, not globally
- Three similar lines of code is better than one premature abstraction
- DRY applies to logic, not to similar-looking code with different intent

### 6. Intent-Revealing Names

Code should read like well-written prose. If you need a comment to explain a name, the name is wrong.

| Category | Convention | Examples |
|----------|-----------|---------|
| Functions | Verb-first, describes action | `fetch_candles`, `validate_order`, `calculate_pnl` |
| Booleans | Reads as yes/no question | `is_valid`, `has_positions`, `should_retry` |
| Classes | PascalCase noun, domain concept | `OrderExecutor`, `PositionTracker` |
| Constants | UPPER_SNAKE, self-documenting | `MAX_RETRY_DURATION_SECONDS`, `DUST_THRESHOLD_USD` |
| Private | Single underscore prefix | `_parse_response`, `_validate_config` |

**Anti-patterns**: `data`, `info`, `result`, `tmp`, `val`, `process()`, `handle()`, `do_stuff()`

### 7. Clean Interfaces

Public APIs should be obvious to use correctly and hard to misuse.

```python
# Bad — 6 params, unclear which are required
def place_order(symbol, side, qty, price, tif, effect, reduce, client_id): ...

# Good — required params explicit, rest in typed config
@dataclass(frozen=True)
class OrderRequest:
    symbol: str
    side: Literal["BUY", "SELL"]
    quantity: Decimal
    time_in_force: str = "GTC"
    side_effect_type: str | None = None

def place_order(request: OrderRequest) -> OrderResult: ...
```

- Max 3 positional parameters; use objects/dataclasses for more
- Required params first, optional params with defaults
- Return types that force handling: `Result` over nulls, specific types over `Any`
- Impossible states should be unrepresentable (use enums, unions, Literal types)

### 8. Test Behavior, Not Implementation

Tests should survive refactoring. If you change *how* something works but not *what* it does, tests should still pass.

```python
# Bad — tests implementation details
def test_uses_fifo_internally():
    engine._cost_basis_entries = [...]  # accessing private state
    assert engine._calculate_fifo() == expected  # testing private method

# Good — tests observable behavior
def test_sell_returns_correct_pnl():
    engine.record_buy(symbol="ETH", quantity=1, price=Decimal("2000"))
    result = engine.record_sell(symbol="ETH", quantity=1, price=Decimal("2500"))
    assert result.pnl == Decimal("500")
```

- Test public API, not private methods
- Use Arrange-Act-Assert structure
- Name tests as specifications: `test_close_position_calculates_fifo_pnl`
- Mock at boundaries (external APIs, I/O), not internal collaborators

### 9. No Dead Code

Dead code is misleading code. If it's not used, delete it.

- **No commented-out code** — that's what git history is for
- **No unused imports** — linters catch these; fix them immediately
- **No unused variables** — prefix with `_` only if required by framework/protocol
- **No TODO without an issue** — `TODO(#123)` is acceptable; naked `TODO` is not
- **No backwards-compatibility shims** — if the old code path is dead, remove it entirely
- **No re-exports for removed code** — don't add `# removed` comments or alias stubs

### 10. Separation of Concerns

Each layer should be independently testable and replaceable.

```
┌─────────────┐
│  API Routes  │  ← HTTP handling, request/response mapping
├─────────────┤
│  Services    │  ← Business logic, orchestration
├─────────────┤
│  Exchange    │  ← External API communication
├─────────────┤
│  Models      │  ← Dataclasses only, no behavior
└─────────────┘
```

- **Routes** don't contain business logic — they call services
- **Services** don't know about HTTP — they work with domain types
- **Exchange clients** don't contain parsing logic — separate parsers
- **Models** are pure data — no I/O, no side effects

**Practical test**: Can you swap the exchange layer without touching service logic? Can you test services without a running server?

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

> The 10 Principles above are the foundation. This section covers additional implementation guidance.

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

Before considering code complete, verify against the 10 Principles:

- [ ] Single Responsibility — each function/class does one thing
- [ ] Strict Typing — no `any`/`Any`, all interfaces typed
- [ ] Fail Fast — specific exceptions caught, errors actionable
- [ ] Small Units — functions <50 lines, files <800 lines
- [ ] DRY — no duplicated logic, no premature abstractions
- [ ] Intent-Revealing Names — code reads without comments
- [ ] Clean Interfaces — max 3 params, obvious defaults
- [ ] Tests cover behavior — happy path, edge cases, errors
- [ ] No Dead Code — no commented-out code, no unused imports
- [ ] Separation of Concerns — layers independently testable

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

### Review Against the 10 Principles

1. **Single Responsibility** — Does each function/class do one thing?
2. **Strict Typing** — No `any`/`Any`, all interfaces typed?
3. **Fail Fast & Loud** — Specific exceptions, actionable errors?
4. **Small Units** — Functions <50 lines, files <800 lines?
5. **DRY** — No duplicated logic, no premature abstractions?
6. **Intent-Revealing Names** — Code reads without comments?
7. **Clean Interfaces** — Max 3 params, obvious defaults?
8. **Test Behavior** — Tests survive refactoring, cover edge cases?
9. **No Dead Code** — No commented-out code, unused imports?
10. **Separation of Concerns** — Layers independently testable?

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
