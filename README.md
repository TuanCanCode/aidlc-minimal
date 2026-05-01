# AIDLC Minimal

A lightweight AI-assisted software development workflow for Claude Code. Streamlined from the full AIDLC system — same adaptive quality, far less ceremony.

---

## What It Does

Every time you ask Claude to implement something, it automatically:

1. **Creates a task ticket** (`TASK-001`, `TASK-002`, ...) to track the request
2. **Understands** your codebase and requirements before writing a line of code
3. **Plans** the approach for non-trivial work (skips this for simple tasks)
4. **Builds** the code, tests, and build instructions
5. **Documents** what was built under `docs/` — optional, user-triggered after BUILD
6. **Records** all key decisions in a traceable log

---

## Setup

### Step 1 — Copy files into your project

You can place the files directly in your project root, or inside a `.claude` folder to keep your root directory clean.

**Option A: Project root**
```text
your-project/
├── CLAUDE.md                                    ← copy from minimal/
└── .aidlc-minimal-rule-details/                 ← copy from minimal/
    ├── common/
    │   ├── task-management.md
    │   └── session-continuity.md
    └── phases/
        ├── understand.md
        ├── plan.md
        └── build.md
```

**Option B: `.claude` folder**
```text
your-project/
└── .claude/
    ├── CLAUDE.md                                ← copy from minimal/
    └── .aidlc-minimal-rule-details/             ← copy from minimal/
        ├── common/
        │   ├── task-management.md
        │   └── session-continuity.md
        └── phases/
            ├── understand.md
            ├── plan.md
            └── build.md
```

### Step 2 — Open project in Claude Code

That's it. Claude reads `CLAUDE.md` automatically on startup (or from `.claude/CLAUDE.md`).

---

## How to Use

### Start a new task

Just describe what you want to build:

```
Add a user authentication module with JWT
```

```
Fix the login redirect bug when session expires
```

```
Refactor the payment service to use the new Stripe API
```

Claude will immediately respond with the assigned task number and begin the workflow.

### Answer clarifying questions

For non-trivial requests, Claude creates a Q&A file for you to fill in:

- **Phase 1 questions** → `aidlc-docs/qa-understand.md`
- **Phase 2 questions** → `aidlc-docs/qa-plan.md`

Open the file, fill in each `[Answer]:` tag, then return to the chat and say "done" or "continue".

```markdown
# Requirements Q&A

**Q1**: Will the API require authentication?
[Answer]: Yes, JWT with 24h expiry

**Q2**: Should we support social login (Google, GitHub)?
[Answer]: Not in this phase, email/password only
```

### Approve each phase

Claude pauses at the end of each phase with a summary. Reply to continue:

- `ok`, `yes`, `approve`, `continue`, `looks good` — proceed
- `change X to Y` — request modifications before continuing

---

## Workflow Overview

```
New Request
    │
    ▼
TASK INIT ──────────── Creates TASK-NNN, updates registry
    │
    ▼
UNDERSTAND ─────────── Scans workspace, gathers requirements
    │                  Q&A file if clarification needed
    │                  ⛔ Gate: user approves summary
    │
    ▼
PLAN ───────────────── Scope analysis, component design
    │                  Q&A file if design decisions needed
    │                  ⛔ Gate: user approves plan
    │                  (skipped for simple/trivial tasks)
    │
    ▼
BUILD ──────────────── Generates code + tests
    │                  Appends build & test instructions
    │                  ⛔ Gate: user approves output
    │
    ▼
DOCUMENT ───────────── Loads sources → writes docs under docs/
    │                  Optional: user chooses Yes / No after BUILD
    │                  ⛔ Gate: user approves doc output
    │                  (skippable; run "document TASK-NNN" anytime)
    │
    ▼
COMPLETE ───────────── Task marked done in registry
```

### Complexity → what runs

| Request type          | UNDERSTAND | PLAN          | BUILD        | DOCUMENT         |
|-----------------------|------------|---------------|--------------|------------------|
| Trivial (1-liner fix) | Auto       | **Skipped**   | Inline plan  | User choice      |
| Simple bug fix        | 0–2 Q&A    | **Skipped**   | Inline plan  | User choice      |
| Standard feature      | 3–5 Q&A    | Brief plan    | Full codegen | User choice      |
| Complex system change | 5–8 Q&A    | Full design   | Full codegen | User choice      |

---

## Files Created in Your Project

All workflow files go inside `aidlc-docs/`. Your application code is never touched by the workflow.

```
aidlc-docs/
  state.md              # current phase tracker
  audit.md              # approval timestamps log
  decisions.md          # Q&A decision snapshots, tagged by task
  qa-understand.md      # Phase 1 questions for you to answer
  qa-plan.md            # Phase 2 questions for you to answer
  requirements.md       # requirements doc (standard/complex only)
  design.md             # design decisions (complex only)
  build-and-test.md     # build instructions (complex multi-unit only)
  tasks/
    registry.md         # master list of all tasks + status
    TASK-001.md         # individual task tickets
    TASK-002.md
    ...
```

### Project Documentation (Phase 4)

When you opt into documentation, Phase 4 scaffolds a `docs/` folder at your workspace root:

```
docs/
  README.md             # documentation hub — table of contents
  getting-started.md    # clone, install, run locally
  overview/             # product vision, goals, tech stack
  architecture/         # system diagram, data model, security
  api/                  # per-service endpoint reference
  guides/               # contributing, deployment, how-tos
  features/             # feature specs + implementation guides
  backend/              # coding conventions + patterns
  decisions/            # architecture decision records (ADRs)
```

Sections are created only when relevant (e.g. no `frontend/` for backend-only projects).

### Task ticket example

```markdown
# TASK-003: Add password reset flow

**Status**: Complete
**Created**: 2026-05-01T10:00:00Z

## Request
Add a password reset flow with email verification

## Phases
- [x] UNDERSTAND — approved 2026-05-01T10:05:00Z
- [x] PLAN — approved 2026-05-01T10:25:00Z
- [x] BUILD — approved 2026-05-01T11:40:00Z

## Deliverables
- Created: `src/auth/password-reset.service.ts`
- Created: `src/auth/password-reset.controller.ts`
- Modified: `src/auth/auth.module.ts`
- Tests: 12 unit tests
```

### Task registry example

```markdown
# Task Registry

**Next Task**: TASK-004

| Task     | Title                    | Status      | Created              |
|----------|--------------------------|-------------|----------------------|
| TASK-001 | Add JWT authentication   | Complete    | 2026-05-01T09:00:00Z |
| TASK-002 | Fix login redirect bug   | Complete    | 2026-05-01T11:00:00Z |
| TASK-003 | Add password reset flow  | Complete    | 2026-05-01T14:00:00Z |
```

---

## Resuming a Session

If you close and reopen Claude Code mid-task, Claude detects the in-progress task automatically:

```
## Resuming Project

Active Task: TASK-003 — Add password reset flow
Phase: BUILD
Last action: PLAN approved

Continue TASK-003, or create a new task for a fresh request?
```

---

## Key Differences from Full AIDLC

| | Full AIDLC | AIDLC Minimal |
|---|---|---|
| Approval gates | 13+ | **4** (UNDERSTAND + PLAN + BUILD + DOCUMENT) |
| Rule files | 28 | **7** |
| Docs per project | 20+ | **2–8** |
| Q&A format | `[Answer]:` in file | `[Answer]:` in file |
| Decision log | Per-interaction audit | **Key decisions only** |
| Task tracking | None | **TASK-NNN tickets** |
| Documentation phase | Manual | **Optional, sources-driven** |
| Welcome message | Every session | **None** |

---

## Credits

This project was heavily inspired by and adapted from the official [AWS AIDLC Workflows](https://github.com/awslabs/aidlc-workflows#usage). The minimal version strips away the enterprise ceremony while keeping the core adaptive and phase-based philosophy intact.
