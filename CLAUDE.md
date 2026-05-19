# AIDLC Minimal

## Rule Files
Load these at the start of EVERY software development request:
- `.aidlc-minimal-rule-details/common/task-management.md`
- `.aidlc-minimal-rule-details/common/session-continuity.md`
- `.aidlc-minimal-rule-details/phases/ideate.md`
- `.aidlc-minimal-rule-details/phases/backlog.md`
- `.aidlc-minimal-rule-details/phases/init.md`
- `.aidlc-minimal-rule-details/phases/understand.md`
- `.aidlc-minimal-rule-details/phases/plan.md`
- `.aidlc-minimal-rule-details/phases/build.md`
- `.aidlc-minimal-rule-details/phases/document.md`

---

## Workflow

### Idea Capture & Backlog — STANDALONE (runs independently, any time)
Triggered by: `"capture idea: [title]"` / `"new idea"` / `"idea: [title]"`

Generate an idea file in `aidlc-docs/ideas/` → present the file path → stop. No AIDLC workflow runs.

After capture, two paths:
- **Small idea** → `"promote IDEA-NNN"` — converts directly into a TASK, starts the workflow at Step 1
- **Larger idea** → `"backlog IDEA-NNN"` — AI reads the idea and generates a checklist of high-level implementation items in `aidlc-docs/backlog/IDEA-NNN.md`; then create tasks one by one with `"task IDEA-NNN [item number or description]"`

Backlog commands: `"list backlog"` · `"show backlog IDEA-NNN"` · `"task IDEA-NNN [item]"`

See `ideate.md` and `backlog.md` for full rules.

---

### Step 0 — SESSION CHECK (always first)
Check `aidlc-docs/state.md`:
- **Not exists** → scan workspace for source code:
  - **No source code found (empty workspace)** → run **Phase 0: PROJECT INIT** before anything else
  - **Source code found (brownfield)** → proceed to Step 1
- **Exists, phase is `INIT`** → init was interrupted → follow `session-continuity.md` to resume
- **Exists, phase is `COMPLETE`** → existing project, new task → proceed to Step 1
- **Exists, in-progress task found** → follow `session-continuity.md` to resume

### Phase 0 — PROJECT INIT (greenfield only, runs once)
Ask user to define workspace folder layout (frontend/backend/etc.) → propose structure → create folders with `.gitkeep` → save to `state.md`.

See `init.md` for full rules.

**No approval gate** — user's first task request is the implicit signal to proceed to Step 1.

### Step 1 — TASK INIT
Ask for task prefix (Jira ID or auto-assign) → read task registry → create task file with Requirements template → present task summary for review.

**Gate**: Present task summary → user opens task file, fills `## Requirements`, then confirms, adds context, or corrects → then proceed to Phase 1.
See `task-management.md` for full rules.

### Phase 1 — UNDERSTAND (always)
Workspace scan → **Reverse Engineering (MANDATORY for brownfield)** → Requirements gathering

**IMPORTANT**: Phase 1 does NOT check `state.md` again. Session routing already happened in Step 0.
**MANDATORY**: On brownfield projects, Reverse Engineering MUST run. If `aidlc-docs/reverse-engineering/` artifacts do not exist, the AI MUST perform the full codebase scan and generate all RE artifacts before proceeding to requirements. If artifacts exist and are current, load them from files (no re-scan needed).

**Gate**: Present understanding summary → wait for user approval

### Phase 2 — PLAN (conditional)
Scope analysis → Component/unit decomposition → Design for complex cases

**Skip entirely when**: trivial change, single-file fix, bug fix with clear scope, pure refactor with no new components.

**Gate**: Present execution plan inline → wait for user approval

### Phase 3 — BUILD (always)
Detect implementation tracks (Backend / Frontend / Mobile) → user chooses order if multi-track → generate code per track with per-track gate → testing → build instructions

**Single-track**: skips track selection, one code generation pass.
**Multi-track**: completes one track → user reviews → next track → user reviews.

**Gate**: Present completion summary → wait for user approval

### Phase 4 — DOCUMENT (optional — user choice after BUILD)
Ask user whether to write/update docs → init `docs/` scaffold if missing → load defined sources → map deliverables to doc targets → write under `docs/`

**Gate**: Present doc summary → wait for user approval
**Skip**: User declines, or user runs `"document TASK-ID"` retroactively

---

## Core Rules

### Questions
Create a markdown Q&A file with `[Answer]:` tags. Tell the user to fill it in and return.
- Phase 1 questions → `aidlc-docs/qa-understand.md`
- Phase 2 questions → `aidlc-docs/qa-plan.md`

After user fills in answers: read the file, snapshot decisions to `aidlc-docs/decisions.md`, then proceed.
Skip Q&A file entirely when the request is crystal clear (0 questions needed).

### Testing
Testing is **optional**. After code generation, the agent asks the user:
- Yes → generate unit tests + integration scenarios immediately
- No → skip, record `Tests: Skipped` in the task file

**Retroactive test generation**: User can request tests for any completed task at any time:
> "generate tests for TASK-ID"

The agent reads the task's Deliverables, loads the source files, and generates tests.
See `phases/build.md` — "Retroactive Test Generation" section for full rules.

### Approvals
- Maximum 5 user approvals per task (task init gate + one per phase gate)
- Auto-proceed between stages within the same phase
- Phase 2 approval gate is skipped when Phase 2 itself is skipped
- Phase 4 approval gate is skipped when user declines documentation

### Audit Trail
Log only key decisions in `aidlc-docs/audit.md`:
```
[YYYY-MM-DDTHH:MM:SSZ] [PHASE] [EVENT]: brief summary
```
Events: `start`, `task-confirmed`, `requirements-approved`, `plan-approved`, `code-approved`, `docs-approved`

### Directory Structure
```
aidlc-docs/
  state.md              # workspace info + current phase
  audit.md              # key approval events (timestamps)
  decisions.md          # Q&A snapshots tagged by task ID
  ideas/                # standalone idea space — not part of AIDLC workflow
    registry.md         # index of all ideas + status
    IDEA-001.md         # individual idea files (user fills in)
    IDEA-002.md
    ...
  backlog/              # per-idea implementation breakdowns
    IDEA-001.md         # checklist of implementation items for IDEA-001
    IDEA-002.md         # one file per idea that has been broken down
    ...
  reverse-engineering/  # brownfield only — written once, reused across all tasks
    business-overview.md
    architecture.md
    code-structure.md
    api-documentation.md
    technology-stack.md
    timestamp.md          # staleness check — compared against git last commit
  qa-understand.md      # Phase 1 Q&A with [Answer]: tags (user fills in)
  qa-plan.md            # Phase 2 Q&A with [Answer]: tags (user fills in)
  requirements.md       # only for standard/complex projects
  design.md             # only when complex design decisions exist
  build-and-test.md     # only for complex multi-unit projects
  tasks/
    registry.md         # task counter + status list
    TASK-001.md         # individual task files (one per request)
    PROJ-123.md         # user-supplied prefix example
    ...
```
**Application code**: workspace root only — NEVER inside `aidlc-docs/`

### Adaptive Depth
| Complexity  | Phase 1     | Phase 2       | Phase 3      |
|-------------|-------------|---------------|--------------|
| Trivial     | Auto (fast) | **SKIP**      | Inline plan  |
| Simple      | 0–2 Q&A     | **SKIP**      | Inline plan  |
| Standard    | 3–5 Q&A     | Brief plan    | Full codegen |
| Complex     | 5–8 Q&A     | Full design   | Full codegen |

### Security
Apply security basics automatically (input validation, no hardcoded secrets, proper auth checks, SQL injection prevention). No opt-in required.

### Brownfield: File Modification
- Always modify existing files in-place
- Never create `ClassName_modified.java` or similar copies
- Check file existence before generating

### UI Code
Add `data-testid` attributes to all interactive elements. Format: `{component}-{role}` (e.g., `login-form-submit-button`).
