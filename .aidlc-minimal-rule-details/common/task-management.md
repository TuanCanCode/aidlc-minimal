# Task Management

**Purpose**: Every new implementation request gets a unique task number (like a Jira ticket) that increments automatically. The AI reads and updates the task file throughout the workflow.

---

## Task Lifecycle

```
New request → Create TASK-NNN → UNDERSTAND → PLAN → BUILD → Complete
```

---

## Step 1: Initialise Task (run BEFORE Phase 1)

### 1.0 Load reverse engineering artifact (brownfield only)

Before creating the task, check `aidlc-docs/reverse-engineering/timestamp.md`:
- **Exists and current** (artifact newer than `git log -1` timestamp) → load all files in `aidlc-docs/reverse-engineering/` into context immediately — Phase 1 UNDERSTAND will skip the re-scan and skip the approval gate
- **Exists but stale / not exists** → Phase 1 UNDERSTAND will run the full reverse engineering scan, generate all files, and present the approval gate
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

## Deliverables
[Populated after BUILD phase completes]
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

### 1.4 Announce the task number

Tell the user at the very start:
```
**Task created: TASK-NNN** — [Short title]
```

---

## Step 2: Update Task During Workflow

Update `aidlc-docs/tasks/TASK-NNN.md` at each phase gate:

| Event | Status change | Phase checkbox |
|---|---|---|
| Phase 1 approved | `Open` → `In Progress` | `[x] UNDERSTAND — approved [timestamp]` |
| Phase 2 approved | (unchanged) | `[x] PLAN — approved [timestamp]` |
| Phase 2 skipped | (unchanged) | `[x] PLAN — skipped (simple scope)` |
| Phase 3 approved | `In Progress` → `Complete` | `[x] BUILD — approved [timestamp]` |
| Phase 4 generated | (unchanged) | `[x] DOCUMENT — approved [timestamp]` |
| Phase 4 skipped | (unchanged) | `[-] DOCUMENT — skipped` |

Also update the registry row status column to match.

---

## Step 3: Populate Deliverables on Completion

When BUILD phase is approved, fill in the `## Deliverables` section:

```markdown
## Deliverables
- Created: `[file path]` — [one-line description]
- Modified: `[file path]` — [what changed]
- Tests: [N unit tests]
```

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
