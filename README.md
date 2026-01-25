# Autonomous Team Skill Bundle

Self-contained bundle with all skills needed for the `autonomous-team` orchestration workflow.

## Included Skills

| Skill | Role |
|-------|------|
| `autonomous-team` | Orchestrator (coordinates all phases) |
| `senior-engineer` | Backend/logic implementation + code quality foundation |
| `frontend-engineer` | React/TypeScript/CSS implementation (used with senior-engineer) |
| `product-owner` | Spec enrichment, scope decisions, ship gate |
| `qa` | E2E test generation and verification |
| `writing-plans` | Task breakdown and implementation planning |
| `security-review` | OWASP checklist, vulnerability scanning |
| `requesting-code-review` | Code review dispatch + template |
| `researcher` | Technical feasibility research |

## Installation

1. Copy all skill folders into your Claude Code skills directory:

   **Windows:**
   ```
   Copy contents to: C:\Users\<YourUsername>\.claude\skills\
   ```

   **macOS/Linux:**
   ```
   Copy contents to: ~/.claude/skills/
   ```

2. Update paths in `autonomous-team/SKILL.md` (already configured for Linux):
   - Default: `/root/.claude/skills/`
   - Adjust to your actual skills path if needed (e.g., `/home/john/.claude/skills/`)

3. Verify the skill appears in Claude Code:
   ```
   claude /skills
   ```

## How It Works

The autonomous-team skill orchestrates 7 phases using sub-agents:

```
Phase 1: Discovery (interactive) -> SPEC.md draft
Phase 2: Definition (product-owner) -> SPEC.md with acceptance criteria
Phase 3: Architecture (senior-engineer) -> ARCHITECTURE.md
Phase 4: Planning (writing-plans) -> TASKS.md
Phase 5: Build Loop (per task):
   -> Implement (senior-engineer + frontend-engineer for UI)
   -> Security Review + Code Review (parallel)
   -> Fix issues (if any)
   -> Update PROGRESS.md
Phase 6: Verification (qa) -> VERIFICATION.md
Phase 7: Ship Decision (product-owner) -> SHIP or GAPS.md
```
