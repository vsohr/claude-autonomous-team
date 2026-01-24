# Autonomous Team — Workflow Specification

## What It Is

A multi-agent orchestration system for Claude Code that converts a rough project idea into a fully working, verified application. A single orchestrator agent coordinates specialized sub-agents through 7 phases, with built-in quality gates (security review + code review) after every implementation task.

## Key Design Decisions

| Decision | Rationale |
|----------|-----------|
| Sub-agents do all work | Keeps context small; each agent sees only what it needs |
| Handoffs via files on disk | Artifacts in `docs/team/` survive agent restarts and crashes |
| Fresh agent per task | Prevents context pollution between tasks |
| Parallel reviews, serial builds | Reviews don't conflict; builds do (same files) |
| Max 5 iterations | Prevents infinite loops on unsolvable problems |
| Scope locked after Phase 1 | Prevents feature creep during build |

---

## How the Skills Work Together

```
User Idea
    │
    ▼
┌─────────────────────────────────────────────────────────┐
│  ORCHESTRATOR (autonomous-team)                         │
│  Reads artifacts, decides next phase, dispatches agents │
└─────────────────────────────────────────────────────────┘
    │
    ├── Phase 1: Discovery ─────────── [Interactive with user]
    │       Output: SPEC.md (draft)
    │       May use: researcher (feasibility questions)
    │
    ├── Phase 2: Definition ─────────── product-owner
    │       Input:  SPEC.md (draft)
    │       Output: SPEC.md (enriched with acceptance criteria)
    │
    ├── Phase 3: Architecture ───────── senior-engineer
    │       Input:  SPEC.md (final)
    │       Output: ARCHITECTURE.md
    │
    ├── Phase 4: Planning ───────────── writing-plans
    │       Input:  SPEC.md + ARCHITECTURE.md
    │       Output: TASKS.md + PROGRESS.md
    │
    ├── Phase 5: Build Loop ─────────── [Per task cycle]
    │       │
    │       │   ┌─── Step 1: Implement ──── senior-engineer
    │       │   │                           (+ frontend-engineer for UI tasks)
    │       │   │
    │       │   ├─── Step 2: Review ─────── security-review ──┐ PARALLEL
    │       │   │                           code-reviewer ─────┘
    │       │   │
    │       │   ├─── Step 3: Fix ────────── general-purpose (conditional)
    │       │   │
    │       │   └─── Step 4: Progress ───── Update PROGRESS.md
    │       │
    │       └── Repeat for each task in TASKS.md
    │
    ├── Phase 6: Verification ───────── qa
    │       Input:  Code + SPEC.md
    │       Output: VERIFICATION.md
    │
    └── Phase 7: Ship Decision ──────── product-owner
            Input:  VERIFICATION.md + SPEC.md
            Output: APPROVED or GAPS.md
```

---

## Skill Roles

### autonomous-team (Orchestrator)
The conductor. Never writes code itself. Reads artifact files, decides which phase to enter next, dispatches sub-agents with precise instructions, and interprets their results to determine next steps.

### senior-engineer
The code quality foundation. Used for:
- Phase 3: Designing technical architecture (stack choices, project structure, APIs)
- Phase 5: Implementing backend/logic tasks with proper error handling, types, and tests

Always present during implementation. Provides the engineering mindset: single responsibility, fail-fast errors, strict typing, no over-engineering.

### frontend-engineer
Layered on top of senior-engineer for UI tasks. Adds:
- React/TypeScript component patterns
- Design quality principles (spacing, typography, color)
- Accessibility and internationalization
- Performance optimization (memoization, virtualization)
- CSS architecture (Tailwind, CSS Modules, etc.)

Never used alone — always paired with senior-engineer.

### product-owner
Owns the definition of "done". Used in three modes:
1. **Definition** (Phase 2): Enriches spec with Given/When/Then acceptance criteria
2. **Scope decisions** (Phase 5): Resolves ambiguity during build
3. **Ship gate** (Phase 7): Final approval comparing spec intent vs. verification results

### writing-plans
Converts architecture into bite-sized, ordered implementation tasks. Each task includes:
- Exact file paths
- Complete code examples (not "add validation")
- TDD steps (write test → verify fail → implement → verify pass)
- Dependency ordering
- Size labels (S/M — never L)

### security-review
Post-implementation audit. Checks every commit against:
- OWASP Top 10 (injection, XSS, broken auth, etc.)
- Hardcoded secrets or credentials
- Input validation gaps
- SSRF/open redirect risks
- Missing rate limiting

Writes findings with severity (Critical/Important/Notes) and a PASS/FAIL verdict.

### requesting-code-review (code-reviewer.md)
Post-implementation quality review. Checks every commit for:
- Spec compliance (does it match the task exactly?)
- Architecture adherence
- Type safety (no `Any`, proper generics)
- DRY violations
- Dead code or scope creep
- Test quality (behavior testing, not mock testing)

Writes findings with severity (Critical/Important/Minor) and PASS/NEEDS FIXES verdict.

### qa
End-to-end verification after all tasks complete. Process:
1. Start the application
2. Generate E2E tests from spec acceptance criteria
3. Run tests
4. Smoke test critical paths
5. Run lint, type checks, unit tests
6. Produce VERIFICATION.md with honest PASS/FAIL

Never fixes issues — only reports them.

### researcher
Optional feasibility research in Phase 1. Dispatched when the orchestrator needs to answer technical questions before writing the spec (e.g., "Can this API do X?" or "What's the best library for Y?").

---

## The Build Loop in Detail

Phase 5 is where most time is spent. Here's the exact flow for each task:

```
Task N from TASKS.md
        │
        ▼
┌──────────────────────────────┐
│ Step 1: Implementation       │
│ Agent reads:                 │
│   - SPEC.md (requirements)   │
│   - ARCHITECTURE.md (design) │
│   - TASKS.md (this task)     │
│   - PROGRESS.md (context)    │
│ Agent writes: code + commit  │
└──────────────────────────────┘
        │
        ▼
┌──────────────────────────────┐
│ Step 2: Parallel Reviews     │
│                              │
│  ┌─────────────┐  ┌────────────────┐
│  │ Security    │  │ Code           │
│  │ Review      │  │ Review         │
│  │             │  │                │
│  │ git diff    │  │ git diff       │
│  │ OWASP check │  │ spec match     │
│  │ secrets scan│  │ type safety    │
│  │             │  │ architecture   │
│  │ → SECURITY- │  │ → CODE-        │
│  │   REVIEW.md │  │   REVIEW.md    │
│  └─────────────┘  └────────────────┘
│                              │
└──────────────────────────────┘
        │
        ▼
┌──────────────────────────────┐
│ Step 3: Fix (conditional)    │
│                              │
│ IF Critical or Important:    │
│   Agent fixes specific issues│
│   Commits: "fix(task-N):..." │
│   Re-runs Step 2             │
│                              │
│ Max 2 fix attempts, then     │
│ log and move on              │
└──────────────────────────────┘
        │
        ▼
┌──────────────────────────────┐
│ Step 4: Update PROGRESS.md   │
│ Mark task complete            │
│ Note deferred Minor issues   │
│ Proceed to Task N+1          │
└──────────────────────────────┘
```

### Why Parallel Reviews Work

Security review writes to `SECURITY-REVIEW.md`. Code review writes to `CODE-REVIEW.md`. Different files, no conflicts. Both read `git diff` (read-only). This halves the review time.

### Why Serial Implementation

Implementation agents modify source files. Running two in parallel would cause merge conflicts. Tasks are ordered by dependency, so they must run sequentially anyway.

### Fix Attempt Limits

After 2 failed fix attempts on the same task, the orchestrator logs the unresolved issues to PROGRESS.md and moves to the next task. This prevents infinite loops where a reviewer keeps finding new issues in the fixes.

---

## Artifact Flow

```
Phase 1  →  SPEC.md (draft)
Phase 2  →  SPEC.md (final, with acceptance criteria)
Phase 3  →  ARCHITECTURE.md
Phase 4  →  TASKS.md + PROGRESS.md
Phase 5  →  Code commits + SECURITY-REVIEW.md + CODE-REVIEW.md (appended per task)
Phase 6  →  VERIFICATION.md
Phase 7  →  APPROVED stamp on VERIFICATION.md  OR  GAPS.md
```

Each phase reads the output of previous phases. This is the contract system — if an artifact is malformed or missing, the consuming phase will fail clearly.

---

## Iteration Logic

If verification fails, the orchestrator routes gaps back to the appropriate phase:

| Gap Type | Routes To |
|----------|-----------|
| Missing feature | Phase 5 (implement it) |
| Bug in code | Phase 5 (fix it) |
| Architecture issue | Phase 3 (redesign) |
| Unclear spec | Phase 2 (clarify) |
| Task breakdown wrong | Phase 4 (replan) |

After re-entering a phase, the system always returns to Phase 6 (Verification) to re-check. Maximum 5 total iterations before declaring BLOCKED.

---

## Context Management Strategy

The key insight is that each sub-agent starts fresh with minimal context:

| Agent | Reads | Ignores |
|-------|-------|---------|
| Implementation | SPEC + ARCH + TASKS + PROGRESS | Previous task code, review history |
| Security reviewer | git diff only | Full codebase, spec details |
| Code reviewer | git diff + TASKS + ARCH | Full codebase, security concerns |
| Fix agent | Review findings only | Spec, architecture, other tasks |
| QA | SPEC + running app | Implementation details |

This keeps each agent focused and prevents the "too much context" problem where agents lose track of their specific job.
