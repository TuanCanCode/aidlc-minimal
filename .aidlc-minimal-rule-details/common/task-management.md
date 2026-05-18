# Task Management

**Purpose**: Every new implementation request gets a unique task ID (auto-generated or user-supplied, e.g. a Jira ticket number). The AI reads and updates the task file throughout the workflow.

---

## Task Lifecycle

```
New request → Ask for prefix → Create TASK-ID → Fill Requirements → Review Gate → UNDERSTAND → PLAN → BUILD → Complete
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

### 1.1 Ask for task prefix

Before reading the registry, ask the user:

```
Do you have an external ticket number for this task (e.g. a Jira ID like "PROJ-123")?
- Provide it → used as the task ID exactly as typed
- Skip / press Enter → auto-assign the next TASK-NNN
```

**Rules**:
- Accept any alphanumeric string with an optional hyphen separator (e.g. `PROJ-123`, `BE-42`, `FE-7`).
- Do NOT normalise or reformat the user-supplied prefix — use it verbatim.
- If the user skips (blank, "no", "skip", "n"), fall through to 1.2 to auto-assign.
- If the supplied ID already exists as a file in `aidlc-docs/tasks/`, warn the user and ask for a different one.

### 1.2 Read or create the registry

Check `aidlc-docs/tasks/registry.md`:
- **Exists** → read `Next Task` number from it (used only when auto-assigning)
- **Not exists** → create it with `Next Task: TASK-001`

Determine final task ID:
- **User supplied a prefix** → use that string as `TASK-ID` (e.g. `PROJ-123`)
- **Auto-assign** → use `TASK-NNN` from registry (e.g. `TASK-001`), then increment `Next Task`

Registry format:
```markdown
# Task Registry

**Next Task**: TASK-001

| Task | Title | Status | Created |
|---|---|---|---|
```

> `Next Task` only increments when auto-assigning. User-supplied IDs do not affect the counter.

### 1.3 Create the task file

Create `aidlc-docs/tasks/TASK-ID.md` (substituting the actual ID):

```markdown
# TASK-ID: [Short title derived from user request]

**Status**: Open
**Created**: [ISO timestamp]
**Tests**: Pending
**Docs**: Pending

## Request
[Original user request — verbatim, never summarised]

## Requirements
> Fill in this section before confirming. Add acceptance criteria, constraints, and any details not captured in the Request above. Delete placeholder lines that do not apply.

### Acceptance Criteria
- [ ] 

### Constraints / Notes
- 

### Out of Scope
- 

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
- `Skipped` — user opted out (retroactive generation available via `"document TASK-ID"`)
- `Generated (retroactive) — [list of doc files]` — docs written after task completed

### 1.4 Update the registry

Append a row to the registry table:

```markdown
| TASK-ID | [Short title] | Open | [ISO timestamp] |
```

### 1.5 Present task for review (GATE)

Present the task summary and explicitly ask the user to fill in the Requirements section:

```
## Task Created: TASK-ID — [Short title]

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

Task file written to: `aidlc-docs/tasks/TASK-ID.md`

**Next step — fill in the Requirements section**:
Open the task file and complete the `## Requirements` section:
- **Acceptance Criteria** — what must be true for this task to be done?
- **Constraints / Notes** — tech choices, deadlines, team conventions, links to designs
- **Out of Scope** — explicitly exclude anything that could be mistakenly assumed in scope

When done, reply with one of:
- **Confirm** → proceed to UNDERSTAND phase (uses requirements as-is)
- **Add context** → paste additional details here and I will append them to the file
- **Correct** → clarify if I misunderstood the request
```

**Wait for explicit user response** before proceeding to Phase 1.

**User responses**:
- User confirms (e.g. "ok", "proceed", "looks good") → read the task file to pick up any edits the user made to `## Requirements`, then proceed to Phase 1
- User pastes additional context → append it to the `## Requirements` section in the task file, update the assessment if needed, then proceed to Phase 1
- User corrects the understanding → update the task file title / request / assessment, re-present the summary for confirmation

Log to `aidlc-docs/audit.md`:
```
[timestamp] TASK-ID INIT confirmed: [type], [scope]
```

---

## Step 2: Update Task During Workflow

Update `aidlc-docs/tasks/TASK-ID.md` at each phase gate:

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

## Task ID Format

Two valid forms:

| Source | Format | Example |
|---|---|---|
| User-supplied | Any alphanumeric string, optional hyphen separator | `PROJ-123`, `BE-42`, `FE-7` |
| Auto-assigned | `TASK-NNN` — zero-padded to 3 digits | `TASK-001`, `TASK-042` |

- Auto-assigned sequence: TASK-001, TASK-002, … TASK-099, TASK-100, …
- Never reuse or skip auto-assigned numbers
- User-supplied IDs are used verbatim — never normalised or reformatted
- User-supplied IDs do not affect the auto-assign counter

---

## Decisions Tagging

When writing to `aidlc-docs/decisions.md`, prefix each section header with the task ID:

```markdown
## TASK-001 | UNDERSTAND — [ISO timestamp]

| Question | Answer | Decision |
|---|---|---|
...
```

Use the actual task ID (e.g. `PROJ-123`) in place of `TASK-001`. This makes decisions traceable back to the task that produced them.

---

## Resuming an In-Progress Task

When `aidlc-docs/state.md` shows `Phase: BUILD` (or any in-progress phase):
1. Read registry to find the `In Progress` task
2. Load that task file
3. Resume from the last unchecked phase checkbox

> The task file name matches the task ID exactly (e.g. `aidlc-docs/tasks/PROJ-123.md` or `aidlc-docs/tasks/TASK-001.md`).
