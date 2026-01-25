---
name: autonomous-team
description: Autonomous software engineering team that converts a rough idea into a fully working, verified application. Use when you have a project idea and want it built end-to-end with minimal intervention. Coordinates discovery, definition, architecture, planning, build, and verification phases using specialized sub-agents.
---

# Autonomous Team

Convert an idea into a working, verified application through coordinated phases.

**Announce at start:** "I'm using the autonomous-team skill to build this project."

## Core Principles

1. **Documents are contracts** — Each phase produces an artifact the next phase consumes
2. **Sub-agents do the work** — Orchestrator coordinates, sub-agents execute
3. **Artifacts live on disk** — All handoffs via files in `docs/team/`
4. **Max 5 iterations** — If verification fails 5 times, produce BLOCKED.md and stop
5. **Scope is locked** — Once spec is approved, new ideas go to `docs/team/FUTURE.md`

---

## Artifact Directory

All phase artifacts are written to `docs/team/` in the project root:

```
docs/team/
├── SPEC.md           # Specification (draft → final)
├── ARCHITECTURE.md   # Technical design
├── TASKS.md          # Implementation plan
├── PROGRESS.md       # Current state and decisions
├── SECURITY-REVIEW.md # Security findings per task (appended)
├── CODE-REVIEW.md    # Code review findings per task (appended)
├── VERIFICATION.md   # QA results
├── GAPS.md           # Issues found (if any)
├── FUTURE.md         # Out-of-scope ideas
└── BLOCKED.md        # If stuck after 5 iterations
```

---

## Phase 1: Discovery (Interactive)

**Goal:** Understand the idea well enough to write a draft spec.

**This phase is interactive — ask the user directly.**

### Process:

1. Read any existing context (project files, README, prior docs)
2. Ask 3-5 batched clarifying questions:
   - What is the core problem this solves?
   - Who is the target user?
   - What are the must-have features vs nice-to-haves?
   - Any technical constraints (stack, hosting, integrations)?
   - What does "done" look like? (Reference product if helpful)
3. If technical feasibility is unclear, dispatch a **researcher sub-agent**:
   ```
   Task (general-purpose):
     "Research feasibility of [specific technical question].
      Read the researcher skill at /root/.claude/skills/researcher\SKILL.md
      and follow its methodology. Write findings to docs/team/research-notes.md"
   ```
4. Write `docs/team/SPEC.md` (draft) with features described
5. Show the user a summary and ask: **"Does this direction look right?"**

**Exit:** User approves direction. Proceed to Phase 2.

---

## Phase 2: Definition (Sub-agent)

**Goal:** Enrich the draft spec with testable acceptance criteria.

**Dispatch sub-agent:**

```
Task (general-purpose):
  description: "Enrich spec with acceptance criteria"
  prompt: |
    You are the Product Owner for this project.
    Read the product-owner skill at /root/.claude/skills/product-owner\SKILL.md
    Follow "Mode 1: Definition" exactly.

    Input: Read docs/team/SPEC.md

    Your job:
    1. Scan every feature for ambiguity
    2. Add acceptance criteria (Given/When/Then format)
    3. Document intent for each feature
    4. Identify and decide edge cases
    5. Set explicit quality bar
    6. Mark out-of-scope items

    Write the enriched spec back to docs/team/SPEC.md

    Important: Every feature must have testable acceptance criteria when you're done.
```

**Exit:** SPEC.md has acceptance criteria for every feature.

---

## Phase 3: Architecture (Sub-agent)

**Goal:** Design the technical implementation.

**Dispatch sub-agent:**

```
Task (general-purpose):
  description: "Design technical architecture"
  prompt: |
    You are a senior architect designing this application.
    Read the senior-engineer skill at /root/.claude/skills/senior-engineer\SKILL.md
    Follow the Architecture & Strategy section.

    Input: Read docs/team/SPEC.md (the full, enriched spec)

    Your job:
    1. Choose tech stack with rationale (prefer boring technology)
    2. Design project/folder structure
    3. Define data models and schemas
    4. Design component architecture and interfaces
    5. Design API contracts (if applicable)
    6. Define error handling strategy
    7. Plan for testability (what testing framework, how to run)

    If you need to research libraries or APIs, use WebSearch and Context7 MCP tools.

    Write output to docs/team/ARCHITECTURE.md

    Use this structure:
    - Tech Stack table (Layer | Choice | Rationale)
    - Project Structure (file tree)
    - Data Models (TypeScript interfaces or equivalent)
    - Component Architecture (relationships and responsibilities)
    - API Design (if applicable)
    - Error Handling Strategy
    - Testing Strategy
```

**Exit:** ARCHITECTURE.md is complete and implementable.

---

## Phase 4: Planning (Sub-agent)

**Goal:** Break architecture into atomic, ordered tasks.

**Dispatch sub-agent:**

```
Task (general-purpose):
  description: "Create implementation task breakdown"
  prompt: |
    You are creating an implementation plan.
    Read the writing-plans skill at /root/.claude/skills/writing-plans\SKILL.md
    Follow its format exactly.

    Input: Read both:
    - docs/team/SPEC.md (what to build)
    - docs/team/ARCHITECTURE.md (how to build it)

    Your job:
    1. Decompose into bite-sized tasks (each S or M sized)
    2. Sequence respecting dependencies
    3. Each task has acceptance criteria and exact file paths
    4. Include complete code in task steps (not "add validation")
    5. Group into milestones
    6. First task must produce something runnable (scaffold)
    7. Identify parallelizable tasks

    Write output to docs/team/TASKS.md

    Use the plan header format from writing-plans skill.
    Include TDD steps (write test, verify fail, implement, verify pass, commit).

    Also create initial docs/team/PROGRESS.md with:
    - Phase: Planning complete
    - Iteration: 1 of 5
    - All tasks listed as pending
```

**Exit:** TASKS.md has actionable tasks. First task is immediately implementable.

---

## Phase 5: Build Loop

**Goal:** Implement all tasks from the plan, with security and code review after each task.

**Each task follows a 4-step sub-agent cycle. All steps use fresh sub-agents to keep context manageable.**

### Task Cycle

For each task in `docs/team/TASKS.md`:

```
Step 1: Implement (sub-agent)
Step 2: Security Review + Code Review (parallel sub-agents)
Step 3: Fix issues (sub-agent, only if Critical/Important found)
Step 4: Update PROGRESS.md, proceed to next task
```

### Step 1: Implementation Sub-agent

Dispatch a fresh sub-agent to implement the current task:

```
Task (general-purpose):
  description: "Implement task N: [task title]"
  prompt: |
    You are a senior engineer implementing a specific task.
    Read the senior-engineer skill at
    /root/.claude/skills/senior-engineer\SKILL.md
    [IF FRONTEND TASK: Also read the frontend-engineer skill at
    /root/.claude/skills/frontend-engineer\SKILL.md]

    Context:
    - Read docs/team/SPEC.md for requirements
    - Read docs/team/ARCHITECTURE.md for design decisions
    - Read docs/team/TASKS.md for your specific task (Task N)
    - Read docs/team/PROGRESS.md for completed work and decisions

    Your job:
    1. Implement ONLY Task N as described
    2. Follow the architecture decisions exactly
    3. Write tests as specified in the task
    4. Commit your changes with a clear message referencing Task N

    Do NOT implement other tasks. Do NOT refactor unrelated code.
    Stick to the task scope.
```

**Builder skill selection:**
- Backend/logic tasks → `senior-engineer` skill only
- Frontend/UI tasks → `senior-engineer` + `frontend-engineer` skills (both)

The `senior-engineer` skill is always used as the foundation (code quality, architecture, error handling). For frontend tasks, the `frontend-engineer` skill adds React/TypeScript patterns, design quality, and accessibility guidance on top.

### Step 2: Security Review + Code Review (Parallel Sub-agents)

After implementation commits, dispatch **both review sub-agents in parallel** (single message, multiple Task calls):

**Security Review Sub-agent:**

```
Task (general-purpose):
  description: "Security review task N"
  prompt: |
    You are a security reviewer examining changes from Task N.
    Read the security-review skill at
    /root/.claude/skills/security-review\SKILL.md

    Your job:
    1. Run: git diff HEAD~1..HEAD to see what changed
    2. Run: git diff --stat HEAD~1..HEAD for file overview
    3. Review ALL changes against the security checklist:
       - Authentication & Authorization gaps
       - Input validation (SQL injection, XSS, command injection)
       - Secrets in code or logs
       - Insecure dependencies
       - SSRF / open redirect risks
       - Unsafe deserialization
       - Missing rate limiting on public endpoints
    4. Check for OWASP Top 10 vulnerabilities
    5. Scan for hardcoded secrets or credentials

    Write findings to docs/team/SECURITY-REVIEW.md using this format:

    ## Task N: [task title] — Security Review
    **Date:** [timestamp]
    **Files reviewed:** [list]

    ### Critical (Block — must fix before proceeding)
    - Issue: [description]
      - Location: [file:line]
      - Risk: [what could go wrong]
      - Fix: [how to resolve]

    ### Important (Fix before next task)
    - Issue: ...

    ### Notes
    - [Observations, recommendations]

    ### Verdict: PASS | FAIL
    (FAIL if any Critical issues exist)

    APPEND to the file (don't overwrite previous reviews).
    If no issues found, write "### Verdict: PASS" with a brief note.
```

**Code Review Sub-agent:**

```
Task (general-purpose):
  description: "Code review task N"
  prompt: |
    You are a senior code reviewer examining changes from Task N.
    Read the code-reviewer template at
    /root/.claude/skills/requesting-code-review\code-reviewer.md

    Context:
    - Read docs/team/TASKS.md for Task N requirements
    - Read docs/team/ARCHITECTURE.md for design expectations

    Your job:
    1. Run: git diff HEAD~1..HEAD to see the full diff
    2. Run: git diff --stat HEAD~1..HEAD for file overview
    3. Review against these criteria:
       - Does implementation match the task spec exactly?
       - Clean separation of concerns?
       - Proper error handling (fail fast, clear messages)?
       - Type safety (no Any, proper generics)?
       - DRY principle (no duplicated logic)?
       - Edge cases handled?
       - Tests actually test behavior (not just mocks)?
       - No scope creep (only implements what's asked)?
       - Naming conventions followed?
       - No dead code or commented-out code?

    Write findings to docs/team/CODE-REVIEW.md using this format:

    ## Task N: [task title] — Code Review
    **Date:** [timestamp]
    **Files reviewed:** [list]

    ### Strengths
    - [What's well done, be specific with file:line]

    ### Critical (Must fix — bugs, data loss, broken functionality)
    - Issue: [description]
      - Location: [file:line]
      - Why: [impact]
      - Fix: [recommendation]

    ### Important (Should fix — architecture, missing tests, poor patterns)
    - Issue: ...

    ### Minor (Nice to have — style, optimization)
    - Issue: ...

    ### Verdict: PASS | NEEDS FIXES
    **Reasoning:** [1-2 sentence technical assessment]

    APPEND to the file (don't overwrite previous reviews).
```

### Step 3: Fix Sub-agent (Conditional)

**Only dispatch if either review produced Critical or Important issues.**

Read both `docs/team/SECURITY-REVIEW.md` and `docs/team/CODE-REVIEW.md` for the latest task's findings.

```
Task (general-purpose):
  description: "Fix review issues for task N"
  prompt: |
    You are fixing issues found during security and code review of Task N.

    Read the following review findings (look at the LATEST Task N section only):
    - docs/team/SECURITY-REVIEW.md
    - docs/team/CODE-REVIEW.md

    Fix ALL Critical and Important issues listed. Ignore Minor issues.

    For each fix:
    1. Read the file at the referenced location
    2. Apply the recommended fix (or a better solution if the recommendation is wrong)
    3. Ensure tests still pass after your fix
    4. Commit with message: "fix(task-N): [brief description of fixes]"

    Do NOT fix Minor issues. Do NOT refactor unrelated code.
    Do NOT re-implement the task — only fix the specific issues listed.
```

After fixes, **re-run Step 2** (both reviews again on the fix commit). If reviews pass, proceed. If still failing after 2 fix attempts, log to PROGRESS.md and continue (don't block indefinitely on a single task).

### Step 4: Update Progress

After reviews pass (or fix attempts exhausted):
1. Update `docs/team/PROGRESS.md` — mark task complete, note any deferred Minor issues
2. Proceed to next task in TASKS.md

### Ambiguity Resolution

If a task is ambiguous, dispatch a **Product Owner sub-agent** before implementing:

```
Task (general-purpose):
  description: "Resolve ambiguity in task N"
  prompt: |
    You are the Product Owner.
    Read /root/.claude/skills/product-owner\SKILL.md, Mode 2.
    Read docs/team/SPEC.md for context.

    Question: [specific question about behavior]

    Decide and write your decision to docs/team/PROGRESS.md under "Decisions Made".
```

**Exit:** All tasks complete with passing reviews. App runs.

---

## Phase 6: Verification (Sub-agent)

**Goal:** Verify the app meets its specification.

**Dispatch sub-agent:**

```
Task (general-purpose):
  description: "QA verification of built application"
  prompt: |
    You are the QA engineer verifying this application.
    Read the qa skill at /root/.claude/skills/qa\SKILL.md
    Follow its process exactly.

    Input:
    - Read docs/team/SPEC.md for acceptance criteria
    - Read docs/team/ARCHITECTURE.md for how to start the app

    Your job:
    1. Start the application
    2. Verify it runs without errors
    3. Generate E2E tests from spec acceptance criteria
    4. Run the tests
    5. Perform smoke test of critical paths
    6. Run available quality checks (types, lint, unit tests)
    7. Produce docs/team/VERIFICATION.md

    Do NOT fix any issues you find. Report them honestly.
    Mark overall status as PASS or FAIL.
```

**Exit:** VERIFICATION.md produced with clear PASS/FAIL status.

---

## Phase 7: Ship Decision

**After verification, evaluate the result:**

### If VERIFICATION.md shows PASS:

Dispatch **Product Owner sub-agent** for final approval:

```
Task (general-purpose):
  description: "Product Owner ship decision"
  prompt: |
    You are the Product Owner making the final ship decision.
    Read /root/.claude/skills/product-owner\SKILL.md, Mode 3.

    Read:
    - docs/team/SPEC.md (what was promised)
    - docs/team/VERIFICATION.md (what QA found)

    Your job:
    1. Check every acceptance criterion has a passing test
    2. Verify intent alignment (does this match the quality bar?)
    3. Make ship/no-ship decision

    If SHIP: Write "APPROVED" at the top of docs/team/VERIFICATION.md
    If NO-SHIP: Write docs/team/GAPS.md with specific, actionable gaps
```

### If VERIFICATION.md shows FAIL:

1. Read VERIFICATION.md to understand failures
2. Produce `docs/team/GAPS.md` from the failures
3. Route gaps to appropriate phase (see Iteration Logic below)

---

## Iteration Logic

```
Read docs/team/PROGRESS.md for current iteration count.

IF PASS and PO approves:
    → Announce completion to user
    → Show summary of what was built
    → DONE

ELSE IF iteration < 5:
    → Increment iteration in PROGRESS.md
    → Analyze GAPS.md to determine routing:

    Route each gap:
      - "Missing feature" → Phase 5 (Build - add the feature)
      - "Bug in implementation" → Phase 5 (Build - fix the bug)
      - "Architecture issue" → Phase 3 (Architecture - redesign)
      - "Spec unclear" → Phase 2 (Definition - clarify)
      - "Task breakdown wrong" → Phase 4 (Planning - replan)

    → Re-enter at the appropriate phase
    → After fixes, return to Phase 6 (Verification)

ELSE (iteration = 5):
    → Write docs/team/BLOCKED.md:
      - What works
      - What's stuck
      - What was tried
      - Recommendation for human intervention
    → Report to user
    → STOP
```

---

## Completion

### On Success (SHIP):

Report to user:
```
✓ Project complete.

Built: [one-line summary]
Files: [count of files created/modified]
Tests: [X passing]
Iterations: [N of 5 used]

Artifacts in docs/team/ for reference.
```

### On Block (5 iterations):

Report to user:
```
⚠ Project blocked after 5 iterations.

What works: [summary]
What's stuck: [summary]
See docs/team/BLOCKED.md for details and recommendations.
```

---

## Rules

### Scope Management
- Once SPEC.md is approved (end of Phase 1), scope is locked
- New ideas discovered during build go to `docs/team/FUTURE.md`
- Only Product Owner can make scope decisions

### Quality Gates
- No task proceeds without BOTH security review AND code review passing
- Reviews run as parallel sub-agents after each implementation task
- Critical issues block progress; Important issues must be fixed before next task
- Max 2 fix attempts per task before logging and moving on
- QA must run actual tests (not just read code)
- Product Owner has final say on ship/no-ship

### Sub-agent Discipline
- Each sub-agent gets ONE clear job
- Sub-agents write output to files (not conversational responses)
- Orchestrator reads files to decide next steps
- Never dispatch implementation sub-agents in parallel (file conflicts)
- Security review + code review MUST run in parallel (independent, no file conflicts)
- Fix sub-agents only read review files and fix specific issues (minimal context)

### Progress Tracking
- Update `docs/team/PROGRESS.md` after each phase completes
- Record decisions made and their rationale
- Track current iteration count

---

## Quick Reference

| Phase | Agent/Skill | Input | Output |
|-------|-------------|-------|--------|
| 1. Discovery | Interactive + researcher | Idea | SPEC.md (draft) |
| 2. Definition | product-owner | SPEC.md (draft) | SPEC.md (final) |
| 3. Architecture | senior-engineer | SPEC.md (final) | ARCHITECTURE.md |
| 4. Planning | writing-plans | ARCHITECTURE.md | TASKS.md |
| 5a. Build | senior/frontend-engineer | TASKS.md | Code + commit |
| 5b. Security Review | security-review (parallel) | git diff | SECURITY-REVIEW.md |
| 5c. Code Review | code-reviewer (parallel) | git diff | CODE-REVIEW.md |
| 5d. Fix | general-purpose | Review findings | Fixed code |
| 6. Verification | qa | Code + SPEC.md | VERIFICATION.md |
| 7. Ship Decision | product-owner | VERIFICATION.md | SHIP or GAPS.md |
