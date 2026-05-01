# Session Continuity

## On Project Start: Check for Existing State

This check runs ONCE in CLAUDE.md Step 0 — never inside Phase 1, 2, or 3.

Read `aidlc-docs/state.md`:

**Case 1 — state.md does not exist**:
New project. Proceed directly to Task Init (task-management.md Step 1).

**Case 2 — state.md exists, Phase is `COMPLETE`**:
Previous task finished. Proceed directly to Task Init for the new task.

**Case 3 — state.md exists, Phase is `UNDERSTAND` / `PLAN` / `BUILD`**:
There is an in-progress task. Read `aidlc-docs/tasks/registry.md` to find the `In Progress` task, then present:

```
## Resuming Project

**Active Task**: TASK-NNN — [title]
**Phase**: [phase from state.md]
**Last action**: [last line from audit.md]

Continue TASK-NNN, or start a new task?
```

Wait for user response:
- **Continue** → load context files for the current phase (see table below), resume from last unchecked phase checkbox
- **New task** → proceed to Task Init (task-management.md Step 1)

## Context Loading by Phase

Load only files that actually exist — skip missing files silently.

| Resuming at phase | Load |
|---|---|
| UNDERSTAND | all files in `aidlc-docs/reverse-engineering/` (if brownfield) |
| PLAN | `aidlc-docs/reverse-engineering/` files, `aidlc-docs/requirements.md` |
| BUILD | `aidlc-docs/reverse-engineering/` files, `aidlc-docs/requirements.md`, `aidlc-docs/design.md` |

**For every new task on a brownfield project**: always load all files in `aidlc-docs/reverse-engineering/` at task init — do NOT re-scan the codebase if `timestamp.md` shows artifacts are current (see understand.md Step 2.1 for staleness check).

## State File Format

Keep `aidlc-docs/state.md` minimal:

```markdown
# State
- Project: [Greenfield / Brownfield]
- Started: [ISO timestamp]
- Phase: [UNDERSTAND / PLAN / BUILD / COMPLETE]
- Workspace: [absolute path to workspace root]
```

Update `Phase` field when advancing between phases.
