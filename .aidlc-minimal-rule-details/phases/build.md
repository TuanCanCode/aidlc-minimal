# Phase 3: BUILD

**Purpose**: Generate code, tests, and build instructions.

---

## Step 1: Show Code Plan (inline, no separate approval)

Before generating any code, present a brief inline checklist:

```
## Code Generation Plan

- [ ] [Step 1 — description — target file path]
- [ ] [Step 2 — description — target file path]
- [ ] [Step N — ...]
```

Then **immediately proceed** to generate — do NOT wait for approval of the plan itself.

---

## Step 2: Generate Code

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

### UI code
Add `data-testid` to all interactive elements. Format: `{component}-{element-role}`.
Example: `login-form-submit-button`, `user-list-search-input`.
Use stable, semantic names — only change when element purpose changes.

---

## Step 3: Testing (optional — ask user first)

After code generation is complete, present this prompt **before** generating any tests:

```
## Testing

Code generation is complete. Do you want to generate test cases for TASK-NNN?

A) Yes — generate unit tests and integration test scenarios now
B) No  — skip tests for now (you can run "generate tests for TASK-NNN" at any time later)
```

Wait for user response.

### If user chooses A — Generate tests

Generate tests for every file created or modified in Step 2:

**Unit tests**:
- Co-locate test files with source files (follow existing test file conventions if brownfield)
- Cover: happy path, edge cases, error/null handling, boundary values
- Follow the test framework already in use (Jest, JUnit, pytest, etc.)

**Integration test scenarios**:
- Describe end-to-end flows that cross component boundaries
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

Proceed directly to Step 4. Tests can be generated at any time using the retroactive flow (see "Retroactive Test Generation" section below).

---

## Step 4: Build & Test Instructions (auto, no approval)

After code generation, append instructions inline:

```
## Build & Test

**Install dependencies**: `[command]`
**Build**: `[command]`
**Unit tests**: `[command]`
**Integration tests**: `[command or manual steps]`
```

**For complex multi-unit projects**: save full instructions in `aidlc-docs/build-and-test.md` instead.

---

## Gate: Phase 3 Approval

Present a completion summary:

```
## Build Complete ✓

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
- Fill in `## Deliverables` section with created/modified files
- Set `**Tests**` field to `Generated — [N tests]` or `Skipped`
Update registry row status to `Complete`.
Log to `aidlc-docs/audit.md`:
```
[timestamp] TASK-NNN BUILD complete: [N files created/modified], tests [generated/skipped]
[timestamp] TASK-NNN BUILD approved: confirmed
```

---

## Step 5: Documentation (optional — ask user)

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

Record in `aidlc-docs/tasks/TASK-NNN.md`:
```
**Docs**: Skipped
```

---

## Multi-Unit Execution

When Phase 2 defined multiple units, execute Phase 3 for each unit sequentially:
- Complete steps 1–2 (plan + code) for Unit 1 before starting Unit 2
- Ask the testing question **once** after all units are generated (not per unit)
- A single Phase 3 Gate at the end covers all units

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

After all unit tests, generate integration test scenarios covering cross-component flows introduced by TASK-NNN.

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
