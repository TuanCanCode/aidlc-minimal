# Phase 0: PROJECT INIT

**Purpose**: Define workspace folder structure before any code is written.
Runs **once**, on the very first request for a greenfield project. Skipped for brownfield.

---

## When to Run

Run this phase when **ALL** of the following are true:
- `aidlc-docs/state.md` does NOT exist
- Workspace scan finds **no source code** (no `.ts`, `.js`, `.py`, `.go`, `.java`, `.kt`, `.swift`, `.dart`, `package.json`, `pom.xml`, `build.gradle`, `go.mod`, `Cargo.toml`, etc.)

**Skip** this phase if either condition is false (state.md exists, or code already exists → proceed directly to Step 1 TASK INIT).

---

## Step 1: Create Initial State File

Write `aidlc-docs/state.md` immediately (marks init as in-progress so a resumed session can detect it):

```markdown
# State
- Project: Greenfield
- Started: [ISO timestamp]
- Phase: INIT
- Workspace: [absolute path]
- Structure: (pending)
```

Log to `aidlc-docs/audit.md`:
```
[timestamp] PROJECT INIT start: empty workspace detected
```

---

## Step 2: Ask About Project Type

Present this prompt to the user:

```
## New Project: Define Workspace Structure

Before the first task, let's set up your folder layout so all future code lands in the right place.

**What kind of project are you building?**

1. Fullstack — separate `frontend/` + `backend/`
2. Backend only — `src/` (or a custom name)
3. Frontend only — `src/` (or a custom name)
4. Mobile + Backend — `mobile/` + `backend/`
5. Monorepo — multiple packages under `packages/`
6. Custom — I'll describe the structure myself

Reply with a number, or describe the folders you want directly.
```

Wait for the user's response.

---

## Step 3: Propose Folder Layout

Based on the user's answer, propose a concrete folder layout using the templates below. Adapt names if the user gave custom input.

**1. Fullstack**:
```
frontend/     — UI layer
backend/      — API and business logic
```
Optional: add `shared/` if user mentions shared types/utilities.

**2. Backend only**:
```
src/          — Application source
```
Or use the name the user specified.

**3. Frontend only**:
```
src/          — Application source
```
Or use the name the user specified.

**4. Mobile + Backend**:
```
mobile/       — Mobile application
backend/      — API and business logic
```
Optional: add `shared/` if user mentions shared code.

**5. Monorepo**:
```
packages/
  [service-1]/
  [service-2]/
  shared/
```
Ask the user to name the initial packages if not already stated.

**6. Custom**:
Use exactly what the user described.

Present the proposal like this:

```
## Proposed Structure

[list each root folder with its purpose]

Anything to rename, add, or remove before I create these?
```

Wait for user confirmation or adjustments. Iterate until confirmed.

---

## Step 4: Create Folder Scaffold

Once the user confirms the layout:

1. Create each root folder with a `.gitkeep` placeholder (so git tracks the empty folders):
   - e.g., `frontend/.gitkeep`, `backend/.gitkeep`
2. Update `aidlc-docs/state.md`:
   - Set `Phase: COMPLETE`
   - Set `Structure:` to the confirmed folder names (comma-separated, e.g. `frontend, backend`)

```markdown
# State
- Project: Greenfield
- Started: [ISO timestamp]
- Phase: COMPLETE
- Workspace: [absolute path]
- Structure: frontend, backend
```

3. Log to `aidlc-docs/audit.md`:
```
[timestamp] PROJECT INIT complete: [folder list]
```

---

## Gate: Init Approval

After creating the folders, present:

```
## Workspace Structure Created ✓

[list each folder created]

Ready. What would you like to build first?
```

No explicit "approve" needed here — the user replying with their first task is the implicit approval. Proceed to Step 1 TASK INIT with whatever request the user gives next.
