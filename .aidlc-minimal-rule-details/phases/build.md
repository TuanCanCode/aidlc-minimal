# Phase 3: BUILD

**Purpose**: Generate code, tests, and build instructions.

---

## Step 1: Determine Implementation Tracks

Analyse the plan (Phase 2) or requirements (Phase 1) to identify which implementation tracks are needed:

| Track | When to include |
|---|---|
| **Backend** | API endpoints, services, data models, business logic, database changes, infrastructure |
| **Frontend** | Web UI components, pages, state management, API integration |
| **Mobile** | Mobile app screens, native features, mobile API integration |

### Track detection rules

- **Single-track** (backend only, frontend only, or mobile only) → skip the track selection prompt, proceed directly to code generation for that track
- **Multi-track** (e.g. backend + frontend, backend + mobile) → present the track selection prompt (Step 2)
- A task can have at most 3 tracks: Backend, Frontend, Mobile

---

## Step 2: Track Selection (multi-track only)

When multiple tracks are detected, present the implementation order choice:

```
## Implementation Tracks

This task requires work across multiple tracks:

| # | Track | Scope |
|---|---|---|
| 1 | Backend | [brief scope — e.g. "3 API endpoints, 2 services, 1 migration"] |
| 2 | Frontend | [brief scope — e.g. "2 pages, 3 components, API integration"] |

**Recommended order**: Backend → Frontend
(Backend APIs need to exist before the frontend can integrate with them)

Which track do you want to implement first?
- A) Backend first → then Frontend
- B) Frontend first → then Backend
- C) [Custom order if 3 tracks]
```

**Wait for user response.**

After user chooses, record the order and proceed to Step 3 for the first track.

**Recommendation logic**:
- If frontend/mobile depends on backend APIs → recommend Backend first
- If backend depends on frontend contracts (e.g. BFF pattern) → recommend Frontend first
- If independent → recommend Backend first (convention)
- Always let user override

---

## Step 3: Code Plan (per track, inline, no separate approval)

Before generating code for the current track, present a brief inline checklist scoped to that track:

```
## Code Plan — [Track Name] (Track [N] of [M])

- [ ] [Step 1 — description — target file path]
- [ ] [Step 2 — description — target file path]
- [ ] [Step N — …]
```

Then **immediately proceed** to generate — do NOT wait for approval of the plan itself.

---

## Step 4: Generate Code (per track)

For each step in the plan:
1. Generate the code
2. Mark the step `[x]` in the plan

### Code location rules
- Application code → workspace root (NEVER `aidlc-docs/`)
- Read workspace root from `aidlc-docs/state.md`

### Project structure patterns
| Project type | Structure |
|---|---|
| Brownfield | Follow existing structure |
| Greenfield, single unit | `src/`, `tests/` in workspace root |
| Greenfield, multi-unit microservices | `{unit}/src/`, `{unit}/tests/` |
| Greenfield, multi-unit monolith | `src/{unit}/`, `tests/{unit}/` |

### Brownfield modification rules
- Check if target file exists before generating
- **Exists** → modify in-place
- **Doesn't exist** → create new file
- Never create `ClassName_modified.java`, `ClassName_new.ts`, or similar copies

### Security (always apply, no opt-in)
- Validate all inputs at system boundaries (user input, external APIs)
- No hardcoded secrets, credentials, or connection strings
- Apply proper authentication and authorization checks
- Prevent SQL injection (use parameterized queries)
- Sanitize output to prevent XSS

### UI code (Frontend and Mobile tracks)
Add `data-testid` to all interactive elements. Format: `{component}-{element-role}`.
Example: `login-form-submit-button`, `user-list-search-input`.
Use stable, semantic names — only change when element purpose changes.

---

## Step 5: Track Gate (per track)

After completing code generation for the current track, present a track completion summary:

```
## [Track Name] Complete ✓ (Track [N] of [M])

**Created**:
- `[path]` — [one-line description]

**Modified** (brownfield):
- `[path]` — [what changed]

Please review. Request changes or approve to continue to [Next Track Name / Testing].
```

**Wait for explicit user confirmation.**

- User approves → proceed to next track (Steps 3–5) or to Step 6 (Testing) if this was the last track
- User requests changes → apply changes, re-present the track summary

Log to `aidlc-docs/audit.md`:
```
[timestamp] TASK-NNN BUILD [track-name] complete: [N files created/modified]
```

**Repeat Steps 3–5 for each remaining track.**

---

## Step 6: Testing (optional — ask user after ALL tracks complete)

After all tracks are complete, present this prompt **before** generating any tests:

```
## Testing

All implementation tracks are complete. Do you want to generate test cases for TASK-NNN?

A) Yes — generate unit tests and integration test scenarios now
B) No  — skip tests for now (you can run "generate tests for TASK-NNN" at any time later)
```

Wait for user response.

### If user chooses A — Generate tests

Generate tests for every file created or modified across all tracks:

**Unit tests**:
- Co-locate test files with source files (follow existing test file conventions if brownfield)
- Cover: happy path, edge cases, error/null handling, boundary values
- Follow the test framework already in use (Jest, JUnit, pytest, etc.)

**Integration test scenarios**:
- Describe end-to-end flows that cross component boundaries
- Include cross-track integration (e.g. frontend calling backend APIs)
- Create test file if project already has integration test infrastructure; otherwise describe scenarios as structured comments

After generating, record in `aidlc-docs/tasks/TASK-NNN.md`:
```
**Tests**: Generated — [N unit tests], [N integration scenarios]
```

### If user chooses B — Skip tests

Record in `aidlc-docs/tasks/TASK-NNN.md`:
```
**Tests**: Skipped
```

Proceed directly to Step 7. Tests can be generated at any time using the retroactive flow (see "Retroactive Test Generation" section below).

---

## Step 7: Build & Test Instructions (auto, no approval)

After all tracks and testing, append instructions inline:

```
## Build & Test

### Backend
**Install dependencies**: `[command]`
**Build**: `[command]`
**Run**: `[command]`
**Unit tests**: `[command]`

### Frontend / Mobile
**Install dependencies**: `[command]`
**Build**: `[command]`
**Run**: `[command]`
**Unit tests**: `[command]`

### Integration
**Integration tests**: `[command or manual steps]`
```

Only include sections for tracks that were implemented.
**For complex multi-unit projects**: save full instructions in `aidlc-docs/build-and-test.md` instead.

---

## Gate: Phase 3 Approval

Present a final completion summary across all tracks:

```
## Build Complete ✓

**Tracks completed**: [Backend, Frontend, Mobile — list only those implemented]
**Implementation order**: [Backend → Frontend]

**Created**:
- `[path]` — [one-line description]

**Modified** (brownfield):
- `[path]` — [what changed]

**Tests**: [Generated — N unit tests, N integration scenarios] OR [Skipped — run "generate tests for TASK-NNN" to create later]

Please review the generated code. Request changes or approve to complete.
```

Wait for explicit user confirmation.

Update `aidlc-docs/state.md` Phase field to `COMPLETE`.
Update `aidlc-docs/tasks/TASK-NNN.md`:
- Mark `[x] BUILD — approved [timestamp]`
- Set Status to `Complete`
- Fill in `## Deliverables` section with created/modified files grouped by track
- Set `**Tests**` field to `Generated — [N tests]` or `Skipped`
Update registry row status to `Complete`.
Log to `aidlc-docs/audit.md`:
```
[timestamp] TASK-NNN BUILD complete: [tracks], [N files created/modified], tests [generated/skipped]
[timestamp] TASK-NNN BUILD approved: confirmed
```

---

## Step 8: Documentation (optional — ask user)

After BUILD is approved, present:

```
## Documentation

Do you want to write/update project documentation for TASK-NNN?

A) Yes — load sources and write docs under `docs/` now
B) No  — skip (you can run "document TASK-NNN" at any time later)
```

Wait for user response.

### If user chooses A — Generate docs

Proceed to `document.md` (Phase 4: DOCUMENT).

### If user chooses B — Skip

Update `aidlc-docs/tasks/TASK-NNN.md`:
- Mark `[-] DOCUMENT — skipped` in the `## Phases` checklist
- Set `**Docs**: Skipped`

---

## Multi-Unit Execution

When Phase 2 defined multiple units, execute Phase 3 for each unit sequentially:
- Within each unit, follow the track order (Steps 3–5 per track)
- Ask the testing question **once** after all units and all tracks are generated (not per unit or per track)
- A single Phase 3 Gate at the end covers all units

---

## Single-Track Shortcut

When only one track is detected (Step 1):
- Skip Step 2 (track selection)
- Run Steps 3–4 once (no track label needed in the code plan)
- Skip Step 5 (per-track gate) — proceed directly to Step 6 (Testing)
- **Step 6 (Testing) MUST still run** — present the A/B prompt and wait for user response before continuing
- Step 7 (Build instructions) runs as normal (auto, no approval)
- Phase 3 Gate is the only approval gate for the whole task

This keeps the workflow lean for backend-only or frontend-only tasks.

---

## Retroactive Test Generation

**Trigger**: User requests tests for a completed task — e.g.:
- `"generate tests for TASK-001"`
- `"create test cases for TASK-003"`
- `"I want tests for TASK-NNN now"`

### Execution

**Step R1 — Load task context**

Read `aidlc-docs/tasks/TASK-NNN.md`:
- Confirm `**Tests**: Skipped` (if already Generated, tell user and ask if they want to regenerate)
- Read `## Deliverables` section to get the list of created/modified files

**Step R2 — Load code context**

Read each file listed in Deliverables. For brownfield projects also load:
- `aidlc-docs/reverse-engineering/code-structure.md` for conventions
- `aidlc-docs/reverse-engineering/api-documentation.md` for contracts

**Step R3 — Generate tests**

For each file in Deliverables:
- Generate unit tests covering: happy path, edge cases, error handling, boundary values
- Co-locate with source (follow existing test file naming conventions)
- Follow the test framework already in use

After all unit tests, generate integration test scenarios covering cross-component and cross-track flows introduced by TASK-NNN.

**Step R4 — Update task file**

Update `aidlc-docs/tasks/TASK-NNN.md`:
```
**Tests**: Generated (retroactive) — [N unit tests], [N integration scenarios]
```

**Step R5 — Present summary**

```
## Tests Generated for TASK-NNN ✓

**Unit tests**:
- `[test file path]` — [N tests] covering [component]

**Integration scenarios**:
- `[test file path or description]`

Tests have been added. Review and run with: `[test command]`
```

Log to `aidlc-docs/audit.md`:
```
[timestamp] TASK-NNN TESTS generated (retroactive): [N unit], [N integration]
```
