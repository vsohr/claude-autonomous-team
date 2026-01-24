---
name: writing-plans
description: Use when design is complete and you need detailed implementation tasks for engineers with zero codebase context - creates comprehensive implementation plans with exact file paths, complete code examples, and verification steps assuming engineer has minimal domain knowledge
---

# Writing Plans

## Overview

Write comprehensive implementation plans assuming the engineer has zero context for our codebase and questionable taste. Document everything they need to know: which files to touch for each task, code, testing, docs they might need to check, how to test it. Give them the whole plan as bite-sized tasks. DRY. YAGNI. TDD. Frequent commits.

Assume they are a skilled developer, but know almost nothing about our toolset or problem domain. Assume they don't know good test design very well.

**Announce at start:** "I'm using the writing-plans skill to create the implementation plan."

**Context:** This should be run in a dedicated worktree (created by brainstorming skill).

**Save plans to:** `docs/plans/YYYY-MM-DD-<feature-name>.md`

## Bite-Sized Task Granularity

**Each step is one action (2-5 minutes):**
- "Write the failing test" - step
- "Run it to make sure it fails" - step
- "Implement the minimal code to make the test pass" - step
- "Run the tests and make sure they pass" - step
- "Commit" - step

## Plan Document Header

**Every plan MUST start with this header:**

```markdown
# [Feature Name] Implementation Plan

> **For Claude:** Use executing-plans skill to implement this plan task-by-task.

**Goal:** [One sentence describing what this builds]

**Architecture:** [2-3 sentences about approach]

**Tech Stack:** [Key technologies/libraries]

**Risks:**
- [Risk 1]: [Mitigation]
- [Risk 2]: [Mitigation]

---
```

## Task Structure

```markdown
### Task N: [Component Name] [S/M/L]

**Depends on:** Task X (if any)

**Files:**
- Create: `exact/path/to/file.py`
- Modify: `exact/path/to/existing.py:123-145`
- Test: `tests/exact/path/to/test.py`

**Step 1: Write the failing test**

```python
def test_specific_behavior():
    result = function(input)
    assert result == expected
```

**Step 2: Run test to verify it fails**

Run: `pytest tests/path/test.py::test_name -v`
Expected: FAIL with "function not defined"

**Step 3: Write minimal implementation**

```python
def function(input):
    return expected
```

**Step 4: Run test to verify it passes**

Run: `pytest tests/path/test.py::test_name -v`
Expected: PASS

**Step 5: Commit**

```bash
git add tests/path/test.py src/path/file.py
git commit -m "feat: add specific feature"
```
```

## Task Sizing

| Size | Scope | Example |
|------|-------|---------|
| **S** | Single file, <30 min | Add validation to existing function |
| **M** | 2-3 files, 30-60 min | New endpoint with tests |
| **L** | 4+ files, 1-2 hours | New feature with multiple components |

If a task is **L**, consider splitting it into smaller tasks.

---

## Dependency Mapping

Before finalizing, verify task order:

```
Task 1: Create types/models (no deps)
Task 2: Create data layer (depends: 1)
Task 3: Create service layer (depends: 2)
Task 4: Create API endpoints (depends: 3)
Task 5: Create UI components (depends: 4)
```

**Identify parallelizable tasks** - Tasks with no shared dependencies can run concurrently.

---

## Remember
- Exact file paths always
- Complete code in plan (not "add validation")
- Exact commands with expected output
- DRY, YAGNI, TDD, frequent commits
- Size every task (S/M/L)
- Map dependencies between tasks

## Execution Handoff

After saving the plan, offer execution:

**"Plan complete and saved to `docs/plans/<filename>.md`. Ready to execute?"**

Use executing-plans skill for implementation:
- Batch mode (human review between batches)
- Subagent mode (automated code review between tasks)
