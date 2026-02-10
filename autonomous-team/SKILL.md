---
name: autonomous-team
description: Autonomous software engineering team that converts a rough idea into a fully working, verified application. Use when you have a project idea and want it built end-to-end with minimal intervention. Coordinates discovery, definition, architecture, planning, build, and verification phases using persistent agent teammates with shared task tracking.
---

# Autonomous Team

Convert an idea into a working, verified application through coordinated phases using Claude Code agent teams.

**Announce at start:** "I'm using the autonomous-team skill to build this project with an agent team."

## Core Principles

1. **Documents are contracts** — Each phase produces an artifact the next phase consumes
2. **Persistent teammates do the work** — Team lead coordinates, teammates execute and retain context
3. **Artifacts live on disk** — All handoffs via files in `docs/team/`
4. **Shared task list** — Use TaskCreate/TaskUpdate for progress tracking alongside `docs/team/PROGRESS.md`
5. **Max 5 iterations** — If verification fails 5 times, produce BLOCKED.md and stop
6. **Scope is locked** — Once spec is approved, new ideas go to `docs/team/FUTURE.md`
7. **Worktree isolation** — Build work happens in a git worktree on a feature branch, merged to main only on success

---

## Team Structure

Create a team with **4 persistent teammates**, each retaining context across their tasks:

| Teammate | Role | Phases | Why Persistent? |
|----------|------|--------|-----------------|
| **architect** | Definition, architecture, planning | 2, 3, 4 | Retains spec context through design → planning pipeline |
| **builder** | Implementation | 5 | Retains codebase understanding across milestones |
| **reviewer** | Security + code review | 5b/5c | Retains review context, knows prior findings |
| **qa** | Verification + ship decision | 6, 7 | Retains verification context for ship decision |

**Team lead (you)** handles: Phase 1 (Discovery), Phase 4.5 (Worktree), coordination, trivial fixes, iteration routing, merge & cleanup.

---

## Phase 0: Team Setup

**Goal:** Create the agent team and shared task list.

### Process:

1. **Create the team:**
   ```
   TeamCreate:
     team_name: "<project-slug>"
     description: "Building <project name>"
   ```

2. **Spawn all 4 teammates** (in a single message for parallel startup):
   ```
   Task (general-purpose):
     team_name: "<project-slug>"
     name: "architect"
     description: "Architect teammate"
     prompt: |
       You are the architect for this project. You handle definition, architecture, and planning.
       Read the following skills for reference:
       - C:\Users\User\.claude\skills\product-owner\SKILL.md (for definition work)
       - C:\Users\User\.claude\skills\senior-engineer\SKILL.md (for architecture)
       - C:\Users\User\.claude\skills\writing-plans\SKILL.md (for planning)

       Wait for task assignments from the team lead.
       Write all artifacts to docs/team/ in the project root.
       When you complete a task, mark it completed with TaskUpdate and message the team lead.

   Task (general-purpose):
     team_name: "<project-slug>"
     name: "builder"
     description: "Builder teammate"
     prompt: |
       You are the senior engineer builder for this project. You implement milestones.
       Read the senior-engineer skill at C:\Users\User\.claude\skills\senior-engineer\SKILL.md
       Also read the frontend-engineer skill at C:\Users\User\.claude\skills\frontend-engineer\SKILL.md for UI work.

       Wait for task assignments from the team lead.
       You will work in a git worktree (path provided per task).
       When you complete a milestone, mark the task completed with TaskUpdate and message the team lead.

   Task (general-purpose):
     team_name: "<project-slug>"
     name: "reviewer"
     description: "Reviewer teammate"
     model: haiku
     prompt: |
       You are the security and code reviewer for this project.
       Read the security-review skill at C:\Users\User\.claude\skills\security-review\SKILL.md

       Wait for task assignments from the team lead.
       You review git diffs at milestone checkpoints.
       Write findings to docs/team/SECURITY-REVIEW.md and docs/team/CODE-REVIEW.md.
       When done, mark the task completed with TaskUpdate and message the team lead.

   Task (general-purpose):
     team_name: "<project-slug>"
     name: "qa"
     description: "QA teammate"
     prompt: |
       You are the QA engineer and Product Owner for this project.
       Read the following skills:
       - C:\Users\User\.claude\skills\qa\SKILL.md (for verification)
       - C:\Users\User\.claude\skills\product-owner\SKILL.md (for ship decisions)

       Wait for task assignments from the team lead.
       When you complete a task, mark it completed with TaskUpdate and message the team lead.
   ```

3. **Create initial tasks** in the shared task list:
   ```
   TaskCreate: "Phase 1: Discovery" (assigned to team-lead)
   TaskCreate: "Phase 2: Definition" (pending, blocked by Phase 1)
   TaskCreate: "Phase 3: Architecture" (pending, blocked by Phase 2)
   TaskCreate: "Phase 4: Planning" (pending, blocked by Phase 3)
   TaskCreate: "Phase 5: Build" (pending, blocked by Phase 4)
   TaskCreate: "Phase 6: Verification" (pending, blocked by Phase 5)
   TaskCreate: "Phase 7: Ship Decision" (pending, blocked by Phase 6)
   ```

**Exit:** Team is running, tasks are created. Proceed to Phase 1.

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

## Phase 1: Discovery (Team Lead — Interactive)

**Goal:** Understand the idea well enough to write a draft spec.

**This phase is interactive — team lead asks the user directly.**

### Process:

1. Read any existing context (project files, README, prior docs)
2. Ask 3-5 batched clarifying questions:
   - What is the core problem this solves?
   - Who is the target user?
   - What are the must-have features vs nice-to-haves?
   - Any technical constraints (stack, hosting, integrations)?
   - What does "done" look like? (Reference product if helpful)
3. If technical feasibility is unclear, message the **architect** to research:
   ```
   SendMessage:
     type: "message"
     recipient: "architect"
     content: "Research feasibility of [specific technical question]. Write findings to docs/team/research-notes.md"
     summary: "Research feasibility question"
   ```
4. Write `docs/team/SPEC.md` (draft) with features described
5. Show the user a summary and ask: **"Does this direction look right?"**

**Exit:** User approves direction. Mark Phase 1 task complete. Proceed to Phase 2.

---

## Phase 2: Definition (Architect Teammate)

**Goal:** Enrich the draft spec with testable acceptance criteria.

**Assign to architect:**

```
SendMessage:
  type: "message"
  recipient: "architect"
  content: |
    Phase 2: Definition.
    Follow the product-owner skill "Mode 1: Definition" exactly.

    Input: Read docs/team/SPEC.md

    Your job:
    1. Scan every feature for ambiguity
    2. Add acceptance criteria (Given/When/Then format)
    3. Document intent for each feature
    4. Identify and decide edge cases
    5. Set explicit quality bar
    6. Mark out-of-scope items

    Write the enriched spec back to docs/team/SPEC.md
    Every feature must have testable acceptance criteria when you're done.
    Mark the Phase 2 task as completed when finished.
  summary: "Definition: enrich spec with acceptance criteria"
```

**Exit:** SPEC.md has acceptance criteria for every feature.

---

## Phase 3: Architecture (Architect Teammate)

**Goal:** Design the technical implementation.

**Advantage:** The architect retains full context from Phase 2 — no need to re-read or re-explain the spec.

**Assign to architect:**

```
SendMessage:
  type: "message"
  recipient: "architect"
  content: |
    Phase 3: Architecture.
    Follow the senior-engineer skill Architecture & Strategy section.

    You already know the spec from Phase 2. Now design the implementation:
    1. Choose tech stack with rationale (prefer boring technology)
    2. Design project/folder structure
    3. Define data models and schemas
    4. Design component architecture and interfaces
    5. Design API contracts (if applicable)
    6. Define error handling strategy
    7. Plan for testability

    Use WebSearch and Context7 MCP tools if you need to research libraries or APIs.

    Write output to docs/team/ARCHITECTURE.md with this structure:
    - Tech Stack table (Layer | Choice | Rationale)
    - Project Structure (file tree)
    - Data Models
    - Component Architecture
    - API Design (if applicable)
    - Error Handling Strategy
    - Testing Strategy

    Mark the Phase 3 task as completed when finished.
  summary: "Architecture: design technical implementation"
```

**Exit:** ARCHITECTURE.md is complete and implementable.

---

## Phase 4: Planning (Architect Teammate)

**Goal:** Break architecture into atomic, ordered tasks.

**Advantage:** The architect retains full context from Phases 2 AND 3 — understands both the spec and the architecture deeply.

**Assign to architect:**

```
SendMessage:
  type: "message"
  recipient: "architect"
  content: |
    Phase 4: Planning.
    Follow the writing-plans skill format exactly.

    You already know the spec and architecture. Now break it into tasks:
    1. Decompose into bite-sized tasks (each S or M sized)
    2. Sequence respecting dependencies
    3. Each task has acceptance criteria and exact file paths
    4. Include complete code in task steps (not "add validation")
    5. Group into milestones
    6. First task must produce something runnable (scaffold)
    7. Identify parallelizable tasks

    Write output to docs/team/TASKS.md

    Also create initial docs/team/PROGRESS.md with:
    - Phase: Planning complete
    - Iteration: 1 of 5
    - All tasks listed as pending
    - Total milestones: [count]

    Create TaskCreate entries in the shared task list for each milestone:
    - "Milestone 1: [name]" (blocked by Phase 4)
    - "Milestone 2: [name]" (blocked by Milestone 1)
    - etc.

    Mark the Phase 4 task as completed when finished.
  summary: "Planning: create implementation task breakdown"
```

**Exit:** TASKS.md has actionable tasks. First task is immediately implementable.

---

## Phase 4.5: Worktree Setup (Team Lead)

**Goal:** Isolate build work in a git worktree so main stays clean until success.

**Why:** Phases 1-4 only produce docs — safe on main. Phase 5+ writes code, so failures could leave dirty state.

### Process:

1. **Derive branch name** from the feature slug:
   ```
   feature/<slug>   (e.g. feature/agent-data-export)
   ```

2. **Create branch and worktree:**
   ```bash
   git branch feature/<slug>
   git worktree add ../<project-name>-<slug> feature/<slug>
   ```

3. **Copy `docs/team/` artifacts** into the worktree:
   ```bash
   cp -r docs/team/ ../<project-name>-<slug>/docs/team/
   ```

4. **Record the worktree path** in `docs/team/PROGRESS.md`:
   ```
   Worktree: ../<project-name>-<slug>
   Branch: feature/<slug>
   ```

5. **Notify the builder** of the worktree path for all subsequent work.

**Exit:** Worktree exists, branch created, docs copied. Ready for Phase 5.

---

## Phase 5: Build Loop

**Goal:** Implement all tasks from the plan efficiently, with reviews at strategic checkpoints.

**Key advantage over subagents:** The builder retains context across milestones — no re-reading SPEC.md, ARCHITECTURE.md, or understanding previous code each time.

### Review Strategy (Based on Project Size)

Before starting the build loop, count total milestones and apply:

| Project Size | Review Points | Rationale |
|--------------|---------------|-----------|
| **Small** (1-3 milestones) | After final milestone only | Low risk |
| **Medium** (4-6 milestones) | After Milestone 1 + final | Catch foundation issues early |
| **Large** (7+ milestones) | Every 2-3 milestones | Regular checkpoints |
| **Security-critical milestone** | Always review | Auth, payments, secrets, user data |

### Milestone Cycle

For each milestone in `docs/team/TASKS.md`:

```
Step 1: Builder implements all tasks in milestone
Step 2: Reviewer does Security + Code Review — IF review checkpoint
Step 3: Fix Critical issues (team lead for trivial, builder for complex)
Step 4: Update PROGRESS.md, proceed to next milestone
```

### Step 1: Assign Milestone to Builder

```
SendMessage:
  type: "message"
  recipient: "builder"
  content: |
    Implement Milestone N: [milestone name]

    IMPORTANT: You are working in a git worktree at [worktree-path].
    All file operations, git commits, and test runs must happen in that directory.

    Context (read these in the worktree):
    - docs/team/SPEC.md for requirements
    - docs/team/ARCHITECTURE.md for design decisions
    - docs/team/TASKS.md for Milestone N tasks
    - docs/team/PROGRESS.md for completed work

    For each task:
    1. Create the files as specified
    2. Run tests to verify
    3. Commit with message referencing the task number

    Update PROGRESS.md after completing the milestone.
    Mark the Milestone N task as completed when finished.

    Do NOT implement tasks from other milestones.
  summary: "Build Milestone N: [name]"
```

**For subsequent milestones:** The builder already knows the codebase — just send the milestone number. No need to re-explain the context.

```
SendMessage:
  type: "message"
  recipient: "builder"
  content: "Implement Milestone N+1: [name]. Same worktree. Mark task completed when done."
  summary: "Build Milestone N+1"
```

**Builder skill selection:**
- Backend/logic tasks → `senior-engineer` skill only
- Frontend/UI tasks → `senior-engineer` + `frontend-engineer` skills (both)

### Step 2: Assign Review to Reviewer (at checkpoints only)

**Only at designated review checkpoints** (see Review Strategy above).

```
SendMessage:
  type: "message"
  recipient: "reviewer"
  content: |
    Review Milestone N at [worktree-path].

    Do BOTH security review and code review:

    Security Review:
    1. Run: git log --oneline -20 in the worktree
    2. Run: git diff [first-commit-of-milestone]..HEAD
    3. Check dependencies if applicable
    4. Review against OWASP Top 10, input validation, secrets
    5. Focus on CRITICAL issues only

    Code Review:
    1. Spot-check: Does implementation match docs/team/TASKS.md specs?
    2. Focus on CRITICAL issues only - bugs, data loss, broken functionality
    3. Skip style/minor issues

    Write findings to:
    - docs/team/SECURITY-REVIEW.md (append section for Milestone N)
    - docs/team/CODE-REVIEW.md (append section for Milestone N)

    Each with: Verdict: PASS | FAIL/NEEDS FIXES
    Mark the review task as completed when finished.
  summary: "Review Milestone N"
```

### Step 3: Fix Critical Issues (Conditional)

**Only run if reviews found CRITICAL issues.**

**Trivial fix path (team lead handles directly):**

If ALL critical issues are:
- Single file affected
- Less than 5 lines changed per issue
- Obvious fix (null check, typo, missing import, off-by-one)

Then the team lead applies fixes directly.

**Complex fix path (message builder):**

```
SendMessage:
  type: "message"
  recipient: "builder"
  content: |
    Fix critical issues found in Milestone N review.
    Read the latest sections of:
    - docs/team/SECURITY-REVIEW.md
    - docs/team/CODE-REVIEW.md

    Fix ONLY Critical issues. Commit with: "fix(milestone-N): [description]"
    Do NOT fix non-critical issues. Do NOT refactor.
  summary: "Fix critical review issues"
```

### Step 4: Update Progress

After milestone complete:
1. Update `docs/team/PROGRESS.md` — mark milestone complete
2. Mark milestone task as completed via TaskUpdate
3. Proceed to next milestone

### Ambiguity Resolution

If the builder encounters ambiguity, they can message the architect directly:

```
# Builder → Architect (direct communication)
SendMessage:
  type: "message"
  recipient: "architect"
  content: "Question about [specific behavior]. What's the intended approach?"
  summary: "Clarifying implementation question"
```

The architect retains full spec/architecture context and can respond immediately.

**Exit:** All milestones complete with passing reviews. App runs.

---

## Phase 6: Verification (QA Teammate)

**Goal:** Verify the app meets its specification.

**Assign to QA:**

```
SendMessage:
  type: "message"
  recipient: "qa"
  content: |
    Phase 6: Verification.
    Follow the qa skill process exactly.

    Working in worktree at [worktree-path].

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

    Include summary counts:
    - Acceptance criteria: X/Y passing
    - Critical issues: [count]
    - Warnings: [count]

    Mark the Phase 6 task as completed when finished.
  summary: "QA verification"
```

**Exit:** VERIFICATION.md produced with clear PASS/FAIL status.

---

## Phase 7: Ship Decision

**After verification, evaluate the result:**

### If VERIFICATION.md shows PASS:

**Fast path (team lead decides directly):**

If VERIFICATION.md shows ALL:
- All acceptance criteria: PASS (100%)
- Critical issues: 0
- Warnings: 0

Then team lead writes "APPROVED" at the top of docs/team/VERIFICATION.md directly, then proceeds to **Merge & Cleanup**.

**Standard path (QA teammate decides):**

**Advantage:** QA retains full verification context from Phase 6 — knows exactly what passed/failed.

```
SendMessage:
  type: "message"
  recipient: "qa"
  content: |
    Phase 7: Ship Decision.
    Follow the product-owner skill Mode 3.

    You already know the verification results from Phase 6.
    Read docs/team/SPEC.md to compare against what was promised.

    Your job:
    1. Check every acceptance criterion has a passing test
    2. Verify intent alignment (does this match the quality bar?)
    3. Make ship/no-ship decision

    If SHIP: Write "APPROVED" at the top of docs/team/VERIFICATION.md
    If NO-SHIP: Write docs/team/GAPS.md with specific, actionable gaps

    Mark the Phase 7 task as completed when finished.
  summary: "Ship decision"
```

### Merge & Cleanup (on APPROVED)

After APPROVED, merge the feature branch:

```bash
# 1. Switch to the main repo directory (NOT the worktree)
cd <original-project-path>

# 2. Merge feature branch (fast-forward preferred)
git merge feature/<slug> --ff-only
# If ff-only fails, use:
git merge feature/<slug> -m "Merge feature/<slug>"

# 3. Remove the worktree
git worktree remove ../<project-name>-<slug>

# 4. Delete the feature branch
git branch -d feature/<slug>
```

### Team Shutdown

After merge (or on block), gracefully shut down all teammates:

```
SendMessage:
  type: "shutdown_request"
  recipient: "architect"
  content: "Project complete, shutting down team"

SendMessage:
  type: "shutdown_request"
  recipient: "builder"
  content: "Project complete, shutting down team"

SendMessage:
  type: "shutdown_request"
  recipient: "reviewer"
  content: "Project complete, shutting down team"

SendMessage:
  type: "shutdown_request"
  recipient: "qa"
  content: "Project complete, shutting down team"
```

After all teammates have shut down:

```
TeamDelete
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
    → Shutdown team
    → Announce completion to user
    → Show summary of what was built
    → DONE

ELSE IF iteration < 5:
    → Increment iteration in PROGRESS.md
    → Analyze GAPS.md to determine routing:

    Route each gap to the appropriate teammate:
      - "Missing feature" → builder (Phase 5)
      - "Bug in implementation" → builder (Phase 5)
      - "Architecture issue" → architect (Phase 3)
      - "Spec unclear" → architect (Phase 2)
      - "Task breakdown wrong" → architect (Phase 4)

    → After fixes, route back to qa (Phase 6)

ELSE (iteration = 5):
    → Write docs/team/BLOCKED.md:
      - What works
      - What's stuck
      - What was tried
      - Worktree preserved at: [worktree-path]
      - Branch: feature/<slug>
      - Recommendation for human intervention
    → Leave worktree intact for user inspection
    → Shutdown team
    → Report to user
    → STOP
```

---

## Completion

### On Success (SHIP):

Report to user:
```
Project complete.

Built: [one-line summary]
Files: [count of files created/modified]
Tests: [X passing]
Iterations: [N of 5 used]
Merged feature/<slug> to main. Worktree cleaned up.
Team shut down.

Artifacts in docs/team/ for reference.
```

### On Block (5 iterations):

Report to user:
```
Project blocked after 5 iterations.

What works: [summary]
What's stuck: [summary]
Worktree preserved at [worktree-path] for inspection.
See docs/team/BLOCKED.md for details and recommendations.
Team shut down.
```

---

## Rules

### Scope Management
- Once SPEC.md is approved (end of Phase 1), scope is locked
- New ideas discovered during build go to `docs/team/FUTURE.md`
- Only the QA/PO teammate can make scope decisions

### Quality Gates
- Reviews run at strategic checkpoints (not every milestone) — see Review Strategy
- Security-critical milestones ALWAYS get reviewed regardless of checkpoint schedule
- Critical issues block progress; must be fixed before next milestone
- Trivial fixes (<5 lines, single file) can be applied by team lead directly
- Max 2 fix attempts per issue before logging and moving on
- QA must run actual tests (not just read code)
- QA/PO has final say on ship/no-ship (or team lead for 100% clean PASS)

### Teammate Discipline
- Each teammate has a clear role — don't send builder work to reviewer
- Teammates write output to files in `docs/team/` (not just messages)
- Team lead reads files to decide next steps
- Never assign two teammates to edit the same files concurrently (file conflicts)
- Teammates can message each other directly for clarification (builder → architect)
- Mark tasks completed via TaskUpdate after each phase/milestone

### Context Advantages (vs Subagents)
- **Architect** retains spec → architecture → planning context (Phases 2-4)
- **Builder** retains codebase understanding across milestones (no re-reading)
- **Reviewer** remembers prior review findings (can catch recurring patterns)
- **QA** retains verification context for ship decision (Phases 6-7)
- Subsequent messages to teammates can be brief — they already have context

### Model Selection

| Teammate | Recommended Model | Rationale |
|----------|-------------------|-----------|
| **architect** | sonnet (default) | Complex design work, needs strong reasoning |
| **builder** | sonnet (default) | Standard implementation, code generation |
| **reviewer** | haiku | Checklist-based reviews, scanning diffs |
| **qa** | sonnet (default) | Needs to run tests and make judgment calls |

Override with `model` parameter when spawning if needed:
- Use `opus` for architect on novel/complex projects
- Use `haiku` for builder on simple S-sized milestones

### Progress Tracking
- Use TaskCreate/TaskUpdate for shared task list (teammates can see progress)
- Update `docs/team/PROGRESS.md` after each phase completes (persists across sessions)
- Record decisions made and their rationale

---

## Quick Reference

| Phase | Teammate | Input | Output |
|-------|----------|-------|--------|
| 0. Setup | Team lead | Idea | Team + task list |
| 1. Discovery | Team lead (interactive) | Idea | SPEC.md (draft) |
| 2. Definition | architect | SPEC.md (draft) | SPEC.md (final) |
| 3. Architecture | architect (has Phase 2 context) | SPEC.md (final) | ARCHITECTURE.md |
| 4. Planning | architect (has Phase 2+3 context) | ARCHITECTURE.md | TASKS.md |
| 4.5 Worktree | Team lead | PROGRESS.md | Worktree + branch |
| 5a. Build | builder (retains context per milestone) | TASKS.md | Code + commits |
| 5b/c. Review | reviewer | git diff | SECURITY-REVIEW.md + CODE-REVIEW.md |
| 5d. Fix | builder or team lead | Review findings | Fixed code |
| 6. Verification | qa | Code + SPEC.md | VERIFICATION.md |
| 7. Ship Decision | qa (has Phase 6 context) | VERIFICATION.md | SHIP or GAPS.md |

**Note:** Reviews (5b/5c) only run at designated checkpoints based on project size. See Review Strategy.
