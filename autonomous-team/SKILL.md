---
name: autonomous-team
description: Autonomous software engineering team that converts a rough idea into a fully working, verified application. Use when you have a project idea and want it built end-to-end with minimal intervention. Coordinates discovery, definition, architecture, planning, build, and verification phases using Task subagents with file-based handoffs.
---

# Autonomous Team

Convert an idea into a working, verified application through coordinated phases using Task subagents.

**Announce at start:** "I'm using the autonomous-team skill to build this project with subagents."

## Core Principles

1. **Documents are contracts** — Each phase produces an artifact the next phase consumes
2. **Subagents do the work** — Team lead coordinates, subagents execute and return results
3. **Artifacts live on disk** — All handoffs via files in `docs/team/`
4. **TodoWrite for tracking** — Use TodoWrite to track phase/milestone progress
5. **Max 5 iterations** — If verification fails 5 times, produce BLOCKED.md and stop
6. **Scope is locked** — Once spec is approved, new ideas go to `docs/team/FUTURE.md`
7. **Worktree isolation** — Build work happens in a git worktree on a feature branch, merged to main only on success

---

## Architecture: Subagents Instead of Teams

This skill uses **Task subagents** (not Claude agent teams). Each subagent:
- Is spawned via the `Task` tool with `subagent_type: "general-purpose"`
- Runs independently, completes its work, and returns a result
- Does NOT persist between phases — context is passed via files in `docs/team/`
- Can be run in parallel when phases are independent

**Trade-off:** Subagents don't retain context across phases (unlike persistent teammates). This is compensated by thorough artifact files that carry all context forward.

---

## Artifact Directory

All phase artifacts are written to `docs/team/` in the project root:

```
docs/team/
├── SPEC.md           # Specification (draft → final)
├── ARCHITECTURE.md   # Technical design
├── TASKS.md          # Implementation plan
├── PROGRESS.md       # Current state and decisions
├── SECURITY-REVIEW.md # Security findings (appended per milestone)
├── CODE-REVIEW.md    # Code review findings (appended per milestone)
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
3. If technical feasibility is unclear, spawn a quick research subagent:
   ```
   Task (general-purpose):
     description: "Research feasibility"
     prompt: "Research feasibility of [specific question]. Return findings."
   ```
4. Write `docs/team/SPEC.md` (draft) with features described
5. Show the user a summary and ask: **"Does this direction look right?"**

**Exit:** User approves direction. Proceed to Phase 2.

---

## Phase 2: Definition (Subagent)

**Goal:** Enrich the draft spec with testable acceptance criteria.

```
Task (general-purpose):
  description: "Define spec with acceptance criteria"
  prompt: |
    You are a Product Owner defining acceptance criteria.
    Read the product-owner skill at C:\Users\User\.claude\skills\product-owner\SKILL.md
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
    Every feature must have testable acceptance criteria when you're done.
```

**Exit:** SPEC.md has acceptance criteria for every feature.

---

## Phase 3: Architecture (Subagent)

**Goal:** Design the technical implementation.

```
Task (general-purpose):
  description: "Design architecture"
  prompt: |
    You are a senior architect designing the technical implementation.
    Read the senior-engineer skill at C:\Users\User\.claude\skills\senior-engineer\SKILL.md

    Input: Read docs/team/SPEC.md for the full specification with acceptance criteria.

    Your job:
    1. Choose tech stack with rationale (prefer boring technology)
    2. Design project/folder structure
    3. Define data models and schemas
    4. Design component architecture and interfaces
    5. Design API contracts (if applicable)
    6. Define error handling strategy
    7. Plan for testability

    Use WebSearch if you need to research libraries or APIs.

    Write output to docs/team/ARCHITECTURE.md with this structure:
    - Tech Stack table (Layer | Choice | Rationale)
    - Project Structure (file tree)
    - Data Models
    - Component Architecture
    - API Design (if applicable)
    - Error Handling Strategy
    - Testing Strategy
```

**Exit:** ARCHITECTURE.md is complete and implementable.

---

## Phase 4: Planning (Subagent)

**Goal:** Break architecture into atomic, ordered tasks.

```
Task (general-purpose):
  description: "Create implementation plan"
  prompt: |
    You are a senior engineer creating an implementation plan.
    Read the writing-plans skill at C:\Users\User\.claude\skills\writing-plans\SKILL.md
    Follow the format exactly.

    Input:
    - Read docs/team/SPEC.md for requirements and acceptance criteria
    - Read docs/team/ARCHITECTURE.md for design decisions

    Your job:
    1. Decompose into bite-sized tasks (each S or M sized)
    2. Sequence respecting dependencies
    3. Each task has acceptance criteria and exact file paths
    4. Include complete code in task steps (not "add validation")
    5. Group into milestones
    6. First task must produce something runnable (scaffold)
    7. Identify parallelizable tasks

    Write output to docs/team/TASKS.md

    Also create docs/team/PROGRESS.md with:
    - Phase: Planning complete
    - Iteration: 1 of 5
    - All milestones listed as pending
    - Total milestones: [count]
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

**Exit:** Worktree exists, branch created, docs copied. Ready for Phase 5.

---

## Phase 5: Build Loop

**Goal:** Implement all tasks from the plan efficiently, with reviews at strategic checkpoints.

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
Step 1: Subagent implements all tasks in milestone
Step 2: Subagent does Security + Code Review — IF review checkpoint
Step 3: Fix Critical issues (team lead for trivial, subagent for complex)
Step 4: Update PROGRESS.md, proceed to next milestone
```

### Step 1: Build Milestone (Subagent)

```
Task (general-purpose):
  description: "Build Milestone N"
  prompt: |
    You are a senior engineer implementing Milestone N: [milestone name].

    IMPORTANT: You are working in a git worktree at [worktree-path].
    All file operations, git commits, and test runs must happen in that directory.

    Context (read these in the worktree):
    - docs/team/SPEC.md for requirements
    - docs/team/ARCHITECTURE.md for design decisions
    - docs/team/TASKS.md for Milestone N tasks specifically
    - docs/team/PROGRESS.md for completed work so far

    Read the senior-engineer skill at C:\Users\User\.claude\skills\senior-engineer\SKILL.md
    For frontend/UI work, also read C:\Users\User\.claude\skills\frontend-engineer\SKILL.md

    For each task in Milestone N:
    1. Create the files as specified in TASKS.md
    2. Run tests to verify
    3. Commit with message referencing the task number

    Update docs/team/PROGRESS.md after completing the milestone.

    Do NOT implement tasks from other milestones.
    Return a summary of what was built and any issues encountered.
```

### Step 2: Review Milestone (Subagent — at checkpoints only)

**Only at designated review checkpoints** (see Review Strategy above).

```
Task (general-purpose):
  description: "Review Milestone N"
  model: haiku
  prompt: |
    You are a security and code reviewer.
    Read the security-review skill at C:\Users\User\.claude\skills\security-review\SKILL.md

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
    Return a summary of findings.
```

### Step 3: Fix Critical Issues (Conditional)

**Only run if reviews found CRITICAL issues.**

**Trivial fix path (team lead handles directly):**

If ALL critical issues are:
- Single file affected
- Less than 5 lines changed per issue
- Obvious fix (null check, typo, missing import, off-by-one)

Then the team lead applies fixes directly.

**Complex fix path (subagent):**

```
Task (general-purpose):
  description: "Fix review issues"
  prompt: |
    You are a senior engineer fixing critical issues found in Milestone N review.
    Working in worktree at [worktree-path].

    Read the latest sections of:
    - docs/team/SECURITY-REVIEW.md
    - docs/team/CODE-REVIEW.md

    Fix ONLY Critical issues. Commit with: "fix(milestone-N): [description]"
    Do NOT fix non-critical issues. Do NOT refactor.
    Return a summary of what was fixed.
```

### Step 4: Update Progress

After milestone complete:
1. Update `docs/team/PROGRESS.md` — mark milestone complete
2. Update TodoWrite — mark milestone task as completed
3. Proceed to next milestone

**Exit:** All milestones complete with passing reviews. App runs.

---

## Phase 6: Verification (Subagent)

**Goal:** Verify the app meets its specification.

```
Task (general-purpose):
  description: "QA verification"
  prompt: |
    You are a QA engineer verifying the application.
    Read the qa skill at C:\Users\User\.claude\skills\qa\SKILL.md

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

    Return the PASS/FAIL verdict and summary.
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

**Standard path (subagent decides):**

```
Task (general-purpose):
  description: "Ship decision"
  prompt: |
    You are a Product Owner making a ship/no-ship decision.
    Read the product-owner skill at C:\Users\User\.claude\skills\product-owner\SKILL.md
    Follow Mode 3 (Ship Gate).

    Working in worktree at [worktree-path].

    Input:
    - Read docs/team/SPEC.md for what was promised
    - Read docs/team/VERIFICATION.md for test results

    Your job:
    1. Check every acceptance criterion has a passing test
    2. Verify intent alignment (does this match the quality bar?)
    3. Make ship/no-ship decision

    If SHIP: Write "APPROVED" at the top of docs/team/VERIFICATION.md
    If NO-SHIP: Write docs/team/GAPS.md with specific, actionable gaps

    Return your decision and reasoning.
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

    Route each gap to the appropriate subagent:
      - "Missing feature" → builder subagent (Phase 5)
      - "Bug in implementation" → builder subagent (Phase 5)
      - "Architecture issue" → architect subagent (Phase 3)
      - "Spec unclear" → definition subagent (Phase 2)
      - "Task breakdown wrong" → planning subagent (Phase 4)

    → After fixes, route back to QA subagent (Phase 6)

ELSE (iteration = 5):
    → Write docs/team/BLOCKED.md:
      - What works
      - What's stuck
      - What was tried
      - Worktree preserved at: [worktree-path]
      - Branch: feature/<slug>
      - Recommendation for human intervention
    → Leave worktree intact for user inspection
    → Report to user
    → STOP
```

---

## Parallel Opportunities

Unlike persistent teammates, subagents can be spawned in parallel when phases are independent. Look for these opportunities:

| Opportunity | When |
|-------------|------|
| **Phases 2+3 cannot be parallelized** | Phase 3 depends on Phase 2 output |
| **Build + Review cannot overlap** | Review needs completed milestone |
| **Multiple small milestones** | If milestones have no file overlap, build in parallel |
| **Research tasks during Phase 1** | Feasibility checks while writing spec |

When parallelizing milestones, ensure they don't edit the same files. If in doubt, run sequentially.

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
```

---

## Rules

### Scope Management
- Once SPEC.md is approved (end of Phase 1), scope is locked
- New ideas discovered during build go to `docs/team/FUTURE.md`
- Only the PO subagent (Phase 7) can make scope decisions

### Quality Gates
- Reviews run at strategic checkpoints (not every milestone) — see Review Strategy
- Security-critical milestones ALWAYS get reviewed regardless of checkpoint schedule
- Critical issues block progress; must be fixed before next milestone
- Trivial fixes (<5 lines, single file) can be applied by team lead directly
- Max 2 fix attempts per issue before logging and moving on
- QA must run actual tests (not just read code)
- PO subagent has final say on ship/no-ship (or team lead for 100% clean PASS)

### Subagent Discipline
- Each subagent gets a clear, focused task — don't overload prompts
- Subagents write output to files in `docs/team/` (not just return messages)
- Team lead reads files to decide next steps
- Never run two subagents that edit the same files concurrently
- Pass all necessary context via the prompt (subagents don't share memory)

### Model Selection

| Subagent Role | Recommended Model | Rationale |
|---------------|-------------------|-----------|
| **Definition** | sonnet (default) | Needs strong reasoning for acceptance criteria |
| **Architecture** | sonnet (default) | Complex design work |
| **Planning** | sonnet (default) | Task decomposition needs precision |
| **Builder** | sonnet (default) | Standard implementation, code generation |
| **Reviewer** | haiku | Checklist-based reviews, scanning diffs |
| **QA** | sonnet (default) | Needs to run tests and make judgment calls |
| **Ship Decision** | sonnet (default) | Judgment call on quality |

Override with `model` parameter when spawning if needed:
- Use `opus` for architecture on novel/complex projects
- Use `haiku` for builder on simple S-sized milestones

### Progress Tracking
- Use TodoWrite for phase/milestone progress (visible to user)
- Update `docs/team/PROGRESS.md` after each phase completes (persists across sessions)
- Record decisions made and their rationale

---

## Quick Reference

| Phase | Executor | Input | Output |
|-------|----------|-------|--------|
| 1. Discovery | Team lead (interactive) | Idea | SPEC.md (draft) |
| 2. Definition | Subagent | SPEC.md (draft) | SPEC.md (final) |
| 3. Architecture | Subagent | SPEC.md (final) | ARCHITECTURE.md |
| 4. Planning | Subagent | ARCHITECTURE.md | TASKS.md |
| 4.5 Worktree | Team lead | PROGRESS.md | Worktree + branch |
| 5a. Build | Subagent (per milestone) | TASKS.md | Code + commits |
| 5b/c. Review | Subagent (at checkpoints) | git diff | SECURITY-REVIEW.md + CODE-REVIEW.md |
| 5d. Fix | Subagent or team lead | Review findings | Fixed code |
| 6. Verification | Subagent | Code + SPEC.md | VERIFICATION.md |
| 7. Ship Decision | Subagent or team lead | VERIFICATION.md | SHIP or GAPS.md |

**Note:** Reviews (5b/5c) only run at designated checkpoints based on project size. See Review Strategy.
