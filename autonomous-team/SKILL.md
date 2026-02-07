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
6. **Worktree isolation** — Build work happens in a git worktree on a feature branch, merged to main only on success

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
      Read the researcher skill at C:\Users\User\.claude\skills\researcher\SKILL.md
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
  model: haiku  # Template-following task, structured output
  description: "Enrich spec with acceptance criteria"
  prompt: |
    You are the Product Owner for this project.
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
    Read the senior-engineer skill at C:\Users\User\.claude\skills\senior-engineer\SKILL.md
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
  model: haiku  # Template-following task, structured breakdown
  description: "Create implementation task breakdown"
  prompt: |
    You are creating an implementation plan.
    Read the writing-plans skill at C:\Users\User\.claude\skills\writing-plans\SKILL.md
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
    - Total milestones: [count] (for review strategy)
```

**Exit:** TASKS.md has actionable tasks. First task is immediately implementable.

---

## Phase 4.5: Worktree Setup (Orchestrator)

**Goal:** Isolate build work in a git worktree so main stays clean until success.

**Why:** Phases 1-4 only produce docs — safe on main. Phase 5+ writes code, so failures could leave dirty state. A worktree on a feature branch means main is untouched until we merge on success.

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

3. **Copy `docs/team/` artifacts** into the worktree (they were created on main during Phases 1-4):
   ```bash
   cp -r docs/team/ ../<project-name>-<slug>/docs/team/
   ```

4. **Record the worktree path** in `docs/team/PROGRESS.md`:
   ```
   Worktree: ../<project-name>-<slug>
   Branch: feature/<slug>
   ```

5. **All subsequent sub-agents** (Phase 5, 6, 7) receive the worktree path and must work there.

**Exit:** Worktree exists, branch created, docs copied. Ready for Phase 5.

---

## Phase 5: Build Loop

**Goal:** Implement all tasks from the plan efficiently, with reviews at strategic checkpoints.

**Key optimizations:**
- Batch tasks by milestone, not individual tasks
- Review at strategic checkpoints based on project size (not every milestone)
- Orchestrator handles trivial fixes directly

### Review Strategy (Based on Project Size)

Before starting the build loop, count total milestones and apply the appropriate review strategy:

| Project Size | Review Points | Rationale |
|--------------|---------------|-----------|
| **Small** (1-3 milestones) | After final milestone only | Low risk, review overhead not justified |
| **Medium** (4-6 milestones) | After Milestone 1 + final | Catch foundation issues early, verify at end |
| **Large** (7+ milestones) | Every 2-3 milestones | Regular checkpoints without excessive overhead |
| **Security-critical milestone** | Always review | Any milestone touching auth, payments, secrets, user data |

**Example for 5-milestone project:**
```
Milestone 1 → Review (foundation check)
Milestone 2 → No review
Milestone 3 → No review
Milestone 4 → No review
Milestone 5 → Review (final check)
```

### Milestone Cycle

For each milestone in `docs/team/TASKS.md`:

```
Step 1: Implement all tasks in milestone (single batch sub-agent)
Step 2: Security Review + Code Review (parallel sub-agents) — IF review checkpoint
Step 3: Fix Critical issues (orchestrator for trivial, sub-agent for complex)
Step 4: Update PROGRESS.md, proceed to next milestone
```

### Step 1: Milestone Implementation Sub-agent

Dispatch a **single sub-agent to implement all tasks in the milestone**:

```
Task (general-purpose):
  description: "Implement Milestone N: [milestone name]"
  prompt: |
    IMPORTANT: You are working in a git worktree at [worktree-path].
    All file operations, git commits, and test runs must happen in that directory.
    Use absolute paths or cd to the worktree before starting work.

    You are a senior engineer implementing a milestone.
    Read the senior-engineer skill at
    C:\Users\User\.claude\skills\senior-engineer\SKILL.md

    Context:
    - Read docs/team/SPEC.md for requirements
    - Read docs/team/ARCHITECTURE.md for design decisions
    - Read docs/team/TASKS.md for Milestone N tasks
    - Read docs/team/PROGRESS.md for completed work

    Your job:
    1. Implement ALL tasks in Milestone N sequentially
    2. For each task:
       - Create the files as specified
       - Run tests to verify
       - Commit with message referencing the task number
    3. Work efficiently - implement, test, commit, move on
    4. Update PROGRESS.md after completing the milestone

    Do NOT implement tasks from other milestones.
    Do NOT wait for external reviews between tasks.
```

**For large milestones (6+ tasks):** Can optionally dispatch parallel sub-agents for independent tasks within the milestone. Use a single message with multiple Task calls.

**Builder skill selection:**
- Backend/logic tasks → `senior-engineer` skill only
- Frontend/UI tasks → `senior-engineer` + `frontend-engineer` skills (both)

### Step 2: Milestone Review (Parallel Sub-agents)

**Only run reviews at designated checkpoints** (see Review Strategy above).

Skip this step if:
- This milestone is not a review checkpoint AND
- This milestone doesn't touch security-critical code (auth, payments, secrets, user data)

At review checkpoints, dispatch **both review sub-agents in parallel**:

**Security Review Sub-agent:**

```
Task (general-purpose):
  model: haiku
  description: "Security review Milestone N"
  prompt: |
    IMPORTANT: You are working in a git worktree at [worktree-path].
    All file operations, git commits, and test runs must happen in that directory.
    Use absolute paths or cd to the worktree before starting work.

    You are a security reviewer examining Milestone N changes.
    Read the security-review skill at
    C:\Users\User\.claude\skills\security-review\SKILL.md

    Your job:
    1. Run: git log --oneline -20 to see recent commits
    2. Run: git diff [first-commit-of-milestone]..HEAD to see all changes
    3. Run: npm audit to check dependencies
    4. Review against security checklist (OWASP Top 10, input validation, secrets)
    5. Focus on CRITICAL issues only - bugs that could cause security vulnerabilities

    Write findings to docs/team/SECURITY-REVIEW.md:

    ## Milestone N: [name] — Security Review
    **Date:** [timestamp]
    **Commits reviewed:** [range]

    ### Critical (Must fix before next milestone)
    - [Only security vulnerabilities that could be exploited]

    ### Notes
    - [Observations]

    ### Verdict: PASS | FAIL

    APPEND to the file. Be concise - milestone reviews should be brief.
```

**Code Review Sub-agent:**

```
Task (general-purpose):
  model: haiku
  description: "Code review Milestone N"
  prompt: |
    IMPORTANT: You are working in a git worktree at [worktree-path].
    All file operations, git commits, and test runs must happen in that directory.
    Use absolute paths or cd to the worktree before starting work.

    You are reviewing Milestone N implementation quality.

    Your job:
    1. Run: git diff [first-commit-of-milestone]..HEAD
    2. Spot-check: Does implementation match TASKS.md specs?
    3. Focus on CRITICAL issues only - bugs, data loss, broken functionality
    4. Skip style/minor issues - not blocking

    Write findings to docs/team/CODE-REVIEW.md:

    ## Milestone N: [name] — Code Review
    **Date:** [timestamp]

    ### Critical Issues (Must fix)
    - [Only blocking bugs]

    ### Verdict: PASS | NEEDS FIXES

    APPEND to the file. Be concise.
```

### Step 3: Fix Critical Issues (Conditional)

**Only run if reviews found CRITICAL issues.**

**Trivial fix path (orchestrator handles directly):**

If ALL critical issues meet these criteria, orchestrator applies fixes directly without dispatching a sub-agent:
- Single file affected
- Less than 5 lines changed per issue
- Obvious fix (null check, typo, missing import, off-by-one)

Example trivial fixes orchestrator can handle:
- Adding `?? ''` for null coalescing
- Adding missing `await`
- Fixing typo in variable name
- Adding missing import statement

**Complex fix path (sub-agent):**

If any issue requires multi-file changes OR complex logic, dispatch fix sub-agent:

```
Task (general-purpose):
  model: haiku  # Targeted fixes, minimal context needed
  description: "Fix critical issues in Milestone N"
  prompt: |
    IMPORTANT: You are working in a git worktree at [worktree-path].
    All file operations, git commits, and test runs must happen in that directory.
    Use absolute paths or cd to the worktree before starting work.

    Fix ONLY the Critical issues listed in:
    - docs/team/SECURITY-REVIEW.md (latest milestone section)
    - docs/team/CODE-REVIEW.md (latest milestone section)

    For each fix:
    1. Apply the fix
    2. Commit with message: "fix(milestone-N): [description]"

    Do NOT fix non-critical issues. Do NOT refactor.
```

After fixes, proceed to next milestone. Don't re-review unless security-critical.

### Step 4: Update Progress

After milestone complete:
1. Update `docs/team/PROGRESS.md` — mark milestone complete
2. Proceed to next task in TASKS.md

### Ambiguity Resolution

If a task is ambiguous, dispatch a **Product Owner sub-agent** before implementing:

```
Task (general-purpose):
  description: "Resolve ambiguity in task N"
  prompt: |
    You are the Product Owner.
    Read C:\Users\User\.claude\skills\product-owner\SKILL.md, Mode 2.
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
  model: haiku  # Checklist-based verification, structured output
  description: "QA verification of built application"
  prompt: |
    IMPORTANT: You are working in a git worktree at [worktree-path].
    All file operations, git commits, and test runs must happen in that directory.
    Use absolute paths or cd to the worktree before starting work.

    You are the QA engineer verifying this application.
    Read the qa skill at C:\Users\User\.claude\skills\qa\SKILL.md
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

    Include summary counts:
    - Acceptance criteria: X/Y passing
    - Critical issues: [count]
    - Warnings: [count]
```

**Exit:** VERIFICATION.md produced with clear PASS/FAIL status.

---

## Phase 7: Ship Decision

**After verification, evaluate the result:**

### If VERIFICATION.md shows PASS:

**Fast path (orchestrator decides directly):**

If VERIFICATION.md shows ALL of these conditions, orchestrator can approve without dispatching PO sub-agent:
- All acceptance criteria: PASS (100%)
- Critical issues: 0
- Warnings: 0

In this case, orchestrator writes "APPROVED" at the top of docs/team/VERIFICATION.md directly, then proceeds to **Merge & Cleanup** (below).

**Standard path (PO sub-agent decides):**

If any criteria failed, or there are warnings/issues requiring judgment, dispatch **Product Owner sub-agent**:

```
Task (general-purpose):
  model: haiku  # Checklist-based decision, structured output
  description: "Product Owner ship decision"
  prompt: |
    You are the Product Owner making the final ship decision.
    Read C:\Users\User\.claude\skills\product-owner\SKILL.md, Mode 3.

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

### Merge & Cleanup (on APPROVED)

After APPROVED is written (by either fast path or PO sub-agent), merge the feature branch back to main:

```bash
# 1. Switch to the main repo directory (NOT the worktree)
cd <original-project-path>

# 2. Merge feature branch (fast-forward preferred)
git merge feature/<slug> --ff-only
# If ff-only fails (main advanced during build), use:
git merge feature/<slug> -m "Merge feature/<slug>"

# 3. Remove the worktree directory
git worktree remove ../<project-name>-<slug>

# 4. Delete the feature branch
git branch -d feature/<slug>
```

After cleanup, proceed to **Completion** reporting.

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
      - Worktree preserved at: [worktree-path]
      - Branch: feature/<slug>
      - Recommendation for human intervention
    → Leave worktree intact for user inspection
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
Merged feature/<slug> to main. Worktree cleaned up.

Artifacts in docs/team/ for reference.
```

### On Block (5 iterations):

Report to user:
```
⚠ Project blocked after 5 iterations.

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
- Only Product Owner can make scope decisions

### Quality Gates
- Reviews run at strategic checkpoints (not every milestone) — see Review Strategy
- Security-critical milestones ALWAYS get reviewed regardless of checkpoint schedule
- When reviews run, BOTH security review AND code review run in parallel
- Critical issues block progress; must be fixed before next milestone
- Trivial fixes (<5 lines, single file) can be applied by orchestrator directly
- Max 2 fix attempts per issue before logging and moving on
- QA must run actual tests (not just read code)
- Product Owner has final say on ship/no-ship (or orchestrator for 100% clean PASS)

### Sub-agent Discipline
- Each sub-agent gets ONE clear job
- Sub-agents write output to files (not conversational responses)
- Orchestrator reads files to decide next steps
- Never dispatch implementation sub-agents in parallel (file conflicts)
- Security review + code review MUST run in parallel (independent, no file conflicts)
- Fix sub-agents only read review files and fix specific issues (minimal context)

### Context Efficiency
- **First invocation**: Sub-agent reads full skill file for context
- **Repeated invocations of same skill**: Consider passing condensed instructions inline instead of re-reading the full skill file
- **Example**: If dispatching multiple milestone implementations, the second+ can skip "Read the senior-engineer skill" and just state the key principles inline
- **Rationale**: Reduces context window usage, speeds up sub-agent startup

### Sub-agent Model Selection

You can choose which model to use for each sub-agent based on task complexity. Use the `model` parameter when dispatching Task calls:

| Task Type | Recommended Model | When to Use |
|-----------|-------------------|-------------|
| **opus** | Complex architecture, ambiguous requirements, novel problems | Large tasks, critical decisions |
| **sonnet** | Standard implementation, doc writing, code generation | Most tasks (default choice) |
| **haiku** | Pattern matching, checklist-based reviews, simple fixes | Reviews, small S-sized tasks |

**Guidelines:**
- **S-sized tasks** (exceptions, simple models, config): Consider `haiku`
- **M-sized tasks** (standard features, validators): Use `sonnet`
- **L-sized tasks** (complex orchestration, novel design): Use `sonnet` or `opus`
- **Reviews** (security, code): Use `haiku` - they follow checklists and scan diffs
- **When unsure**: Default to `sonnet` - good balance of capability and efficiency

**Example:**
```
Task (general-purpose):
  model: haiku  # Lightweight review task
  description: "Security review task N"
  prompt: |
    ...
```

### Progress Tracking
- Update `docs/team/PROGRESS.md` after each phase completes
- Record decisions made and their rationale
- Track current iteration count

---

## Quick Reference

| Phase | Agent/Skill | Model | Input | Output |
|-------|-------------|-------|-------|--------|
| 1. Discovery | Interactive + researcher | — | Idea | SPEC.md (draft) |
| 2. Definition | product-owner | haiku | SPEC.md (draft) | SPEC.md (final) |
| 3. Architecture | senior-engineer | sonnet | SPEC.md (final) | ARCHITECTURE.md |
| 4. Planning | writing-plans | haiku | ARCHITECTURE.md | TASKS.md |
| 4.5 Worktree | Orchestrator (direct) | — | PROGRESS.md | Worktree + branch |
| 5a. Build | senior/frontend-engineer | sonnet | TASKS.md | Code + commit |
| 5b. Security Review | security-review (parallel) | haiku | git diff | SECURITY-REVIEW.md |
| 5c. Code Review | code-reviewer (parallel) | haiku | git diff | CODE-REVIEW.md |
| 5d. Fix | general-purpose | haiku | Review findings | Fixed code |
| 6. Verification | qa | haiku | Code + SPEC.md | VERIFICATION.md |
| 7. Ship Decision | product-owner | haiku | VERIFICATION.md | SHIP or GAPS.md |

**Note:** Reviews (5b/5c) only run at designated checkpoints based on project size. See Review Strategy.
