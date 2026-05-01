# Task Management

**Purpose**: Every new implementation request gets a unique task number (like a Jira ticket) that increments automatically. The AI reads and updates the task file throughout the workflow.

---

## Task Lifecycle

```
New request → Create TASK-NNN → Review Gate → UNDERSTAND → PLAN → BUILD → Complete
```

---

## Step 1: Initialise Task (run BEFORE Phase 1)

### 1.0 Pre-load reverse engineering artifacts (brownfield only)

> **Note**: This step only **loads** existing artifacts. The decision to **run** or **re-run** reverse engineering is made in `understand.md` Step 2 — not here.

Before creating the task, check `aidlc-docs/reverse-engineering/timestamp.md`:
- **Exists and current** (artifact newer than `git log -1` timestamp) → load all files in `aidlc-docs/reverse-engineering/` into context immediately — this gives the AI codebase knowledge before the task even starts
- **Exists but stale / not exists** → do NOT load stale artifacts; Phase 1 UNDERSTAND will run the full reverse engineering scan (MANDATORY — see understand.md Step 2)
- **Greenfield** → skip this step

This ensures every task starts with codebase knowledge already loaded, not re-read.

### 1.1 Read or create the registry

Check `aidlc-docs/tasks/registry.md`:
- **Exists** → read `Next Task` number from it
- **Not exists** → create it with `Next Task: TASK-001`

Registry format:
```markdown
# Task Registry

**Next Task**: TASK-001

| Task | Title | Status | Created |
|---|---|---|---|
```

### 1.2 Create the task file

Create `aidlc-docs/tasks/TASK-NNN.md`:

```markdown
# TASK-NNN: [Short title derived from user request]

**Status**: Open
**Created**: [ISO timestamp]
**Tests**: Pending
**Docs**: Pending

## Request
[Original user request — verbatim, never summarised]

## Phases
- [ ] UNDERSTAND
- [ ] PLAN
- [ ] BUILD
- [ ] DOCUMENT

## Build Tracks
[Populated after UNDERSTAND/PLAN — when tracks are determined]
<!-- Example for multi-track:
- [ ] Backend
- [ ] Frontend
-->

## Deliverables
[Populated after BUILD phase completes]
<!-- Group by track:
### Backend
- Created: `[path]` — [description]

### Frontend
- Created: `[path]` — [description]
-->
```

`**Tests**` field values:
- `Pending` — build not yet complete
- `Generated — [N unit tests], [N integration scenarios]` — tests created during BUILD
- `Skipped` — user opted out (retroactive generation available)
- `Generated (retroactive) — [N unit], [N integration]` — tests created after task completed

`**Docs**` field values:
- `Pending` — build not yet complete
- `Generated — [list of doc files]` — docs written after BUILD
- `Skipped` — user opted out (retroactive generation available via `"document TASK-NNN"`)
- `Generated (retroactive) — [list of doc files]` — docs written after task completed

### 1.3 Update the registry

Append a row to the registry table and increment `Next Task`:

```markdown
| TASK-NNN | [Short title] | Open | [ISO timestamp] |
```

### 1.4 Present task for review (GATE)

Present the task summary and wait for user confirmation before proceeding:

```
## Task Created: TASK-NNN — [Short title]

**Request**:
> [Original user request — verbatim]

**Initial Assessment**:
- **Type**: [New Feature / Bug Fix / Refactor / Upgrade / etc.]
- **Scope**: [Single file / Component / Multi-component / System-wide]
- **Project**: [Greenfield / Brownfield]

**What I understand**:
- [bullet 1 — key intent parsed from the request]
- [bullet 2 — if applicable]
- [bullet 3 — if applicable]

Please review:
- **Confirm** → proceed to UNDERSTAND phase
- **Add context** → provide additional details, constraints, or references before I proceed
- **Correct** → clarify if I misunderstood something
```

**Wait for explicit user response** before proceeding to Phase 1.

**User responses**:
- User confirms (e.g. "ok", "proceed", "looks good") → proceed to Phase 1
- User provides additional context → append the new context to the `## Request` section in the task file, update the assessment if needed, then proceed to Phase 1
- User corrects the understanding → update the task file title / request / assessment, re-present the summary for confirmation

Log to `aidlc-docs/audit.md`:
```
[timestamp] TASK-NNN INIT confirmed: [type], [scope]
```

---

## Step 2: Update Task During Workflow

Update `aidlc-docs/tasks/TASK-NNN.md` at each phase gate:

| Event | Status change | Phase / Track checkbox |
|---|---|---|
| Task confirmed | (unchanged — stays `Open`) | — |
| Phase 1 approved | `Open` → `In Progress` | `[x] UNDERSTAND — approved [timestamp]` |
| Phase 2 approved | (unchanged) | `[x] PLAN — approved [timestamp]` |
| Phase 2 skipped | (unchanged) | `[x] PLAN — skipped (simple scope)` |
| Track completed (per track) | (unchanged) | `[x] Backend — completed [timestamp]` |
| Phase 3 approved | `In Progress` → `Complete` | `[x] BUILD — approved [timestamp]` |
| Phase 4 generated | (unchanged) | `[x] DOCUMENT — approved [timestamp]` |
| Phase 4 skipped | (unchanged) | `[-] DOCUMENT — skipped` |

Also update the registry row status column to match.
Also update `## Build Tracks` checkboxes as each track completes.

---

## Step 3: Populate Deliverables on Completion

When BUILD phase is approved, fill in the `## Deliverables` section:

```markdown
## Deliverables

### Backend
- Created: `[file path]` — [one-line description]
- Modified: `[file path]` — [what changed]

### Frontend
- Created: `[file path]` — [one-line description]

### Tests
- [N unit tests], [N integration scenarios]
```

For single-track tasks, omit the track headers — list deliverables flat.

---

## Task Number Format

- Format: `TASK-NNN` (zero-padded to 3 digits)
- Sequence: TASK-001, TASK-002, ... TASK-099, TASK-100, ...
- Never reuse or skip numbers

---

## Decisions Tagging

When writing to `aidlc-docs/decisions.md`, prefix each section header with the task number:

```markdown
## TASK-001 | UNDERSTAND — [ISO timestamp]

| Question | Answer | Decision |
|---|---|---|
...
```

This makes decisions traceable back to the task that produced them.

---

## Resuming an In-Progress Task

When `aidlc-docs/state.md` shows `Phase: BUILD` (or any in-progress phase):
1. Read registry to find the `In Progress` task
2. Load that task file
3. Resume from the last unchecked phase checkbox
