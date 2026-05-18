# AIDLC Minimal

A lightweight AI-assisted software development workflow for Claude Code. Streamlined from the full AIDLC system — same adaptive quality, far less ceremony.

---

## What It Does

Every time you ask Claude to implement something, it automatically:

1. **Captures ideas** in a standalone backlog before they become tasks (optional)
2. **Initialises** a folder structure for brand-new empty projects — runs once, skipped for existing codebases
3. **Creates a task ticket** with a custom ID (Jira number) or auto-assigned `TASK-NNN`
4. **Understands** your codebase and requirements before writing a line of code
5. **Plans** the approach for non-trivial work (skips this for simple tasks)
6. **Builds** the code across Backend / Frontend / Mobile tracks
7. **Tests** what it built — optional, or generate retroactively at any time
8. **Documents** what was built under `docs/` — optional, user-triggered after BUILD
9. **Records** all key decisions in a traceable audit log

---

## Setup

### Step 1 — Copy files into your project

**Option A: Project root**
```text
your-project/
├── CLAUDE.md
└── .aidlc-minimal-rule-details/
    ├── common/
    │   ├── task-management.md
    │   └── session-continuity.md
    └── phases/
        ├── ideate.md
        ├── init.md
        ├── understand.md
        ├── plan.md
        ├── build.md
        └── document.md
```

**Option B: `.claude` folder**
```text
your-project/
└── .claude/
    ├── CLAUDE.md
    └── .aidlc-minimal-rule-details/
        ├── common/
        │   ├── task-management.md
        │   └── session-continuity.md
        └── phases/
            ├── ideate.md
            ├── init.md
            ├── understand.md
            ├── plan.md
            ├── build.md
            └── document.md
```

### Step 2 — Open project in Claude Code

That's it. Claude reads `CLAUDE.md` automatically on startup.

---

## How to Use

### Capture an idea (before it's a task)

Park rough ideas without triggering the full workflow:

```
capture idea: smarter search with semantic ranking
```

Claude creates `aidlc-docs/ideas/IDEA-001.md` with a template and stops. Fill it in at your own pace. When it's ready:

```
promote IDEA-001
```

Claude reads the idea file, pre-populates a task from it, then starts the normal workflow.

### Start a new task

Describe what you want to build:

```
Add a user authentication module with JWT
```

```
Fix the login redirect bug when session expires
```

Claude will ask for a Jira ticket number (or auto-assign `TASK-NNN`), create the task file, and present it for your review.

### Fill in requirements

After the task file is created, open it and complete the `## Requirements` section before confirming:

```markdown
## Requirements

### Acceptance Criteria
- [ ] Users can log in with email + password
- [ ] JWT expires after 24h

### Constraints / Notes
- Use existing AuthModule, do not replace it

### Out of Scope
- Social login (next phase)
```

Reply `ok` or `proceed` to start the workflow.

### Answer clarifying questions

For non-trivial requests, Claude creates a Q&A file:

- **Phase 1 questions** → `aidlc-docs/qa-understand.md`
- **Phase 2 questions** → `aidlc-docs/qa-plan.md`

Open the file, fill in each `[Answer]:` tag, then reply "done" or "continue".

### Approve each phase

Claude pauses at the end of each phase. Reply to continue:

- `ok`, `yes`, `approve`, `continue`, `looks good` — proceed
- `change X to Y` — request modifications before continuing

---

## Workflow Overview

```
"capture idea: …"  ──────────────────────────────────────────────────────┐
                                                                          │
                                                              IDEA CAPTURE (standalone)
                                                              Creates IDEA-NNN.md
                                                              User fills template at own pace
                                                              "promote IDEA-NNN" → TASK INIT
                                                                          │
New Request ──────────────────────────────────────────────────────────────┘
    │
    ▼
PROJECT INIT ───────── (greenfield only, runs once)
    │                  Ask for folder layout → create frontend/, backend/, etc.
    │                  Skipped if workspace already has code
    │
    ▼
TASK INIT ──────────── Ask for task prefix (Jira ID or auto-assign TASK-NNN)
    │                  Creates task file with Requirements template
    │                  ⛔ Gate: user fills Requirements, reviews, confirms
    │
    ▼
UNDERSTAND ─────────── Scans workspace + Reverse Engineering (brownfield)
    │                  Gathers requirements
    │                  Q&A file if clarification needed
    │                  ⛔ Gate: user approves summary
    │
    ▼
PLAN ───────────────── Scope analysis, component design
    │                  Q&A file if design decisions needed
    │                  ⛔ Gate: user approves plan
    │                  (skipped for simple / trivial tasks)
    │
    ▼
BUILD ──────────────── Detects tracks: Backend / Frontend / Mobile
    │                  User chooses track order (multi-track only)
    │                  Generates code per track → per-track gate
    │                  Optional: generate tests (Yes/No prompt)
    │                  Appends build & run instructions
    │                  ⛔ Gate: user approves output
    │
    ▼
DOCUMENT ───────────── Loads sources → writes docs under docs/
    │                  Optional: user chooses Yes / No after BUILD
    │                  ⛔ Gate: user approves doc output
    │                  (skippable; run "document TASK-ID" anytime)
    │
    ▼
COMPLETE ───────────── Task marked done in registry
```

### Complexity → what runs

| Request type          | UNDERSTAND | PLAN        | BUILD        | DOCUMENT    |
|-----------------------|------------|-------------|--------------|-------------|
| Trivial (1-liner fix) | Auto       | **Skipped** | Inline plan  | User choice |
| Simple bug fix        | 0–2 Q&A    | **Skipped** | Inline plan  | User choice |
| Standard feature      | 3–5 Q&A    | Brief plan  | Full codegen | User choice |
| Complex system change | 5–8 Q&A    | Full design | Full codegen | User choice |

---

## Supported Guidelines

### Idea Capture (`ideate.md`)

| Guideline | Detail |
|---|---|
| Standalone phase | Runs independently — never triggers UNDERSTAND / PLAN / BUILD |
| Trigger phrases | `"capture idea: [title]"`, `"new idea"`, `"idea: [title]"` |
| Idea template | Problem Statement, Motivation, Rough Solution, Open Questions, Rough Acceptance, References |
| Auto ID assignment | `IDEA-NNN` (zero-padded, never reused) |
| Status lifecycle | `Raw` → `Refined` → `Ready` → `Promoted` / `Parked` / `Dropped` |
| Promotion | `"promote IDEA-NNN"` — pre-populates a task file from idea content, then starts TASK INIT |
| List backlog | `"list ideas"` — shows all ideas grouped by status |
| Folder organization | Flat `aidlc-docs/ideas/` by default; sub-folders on request |

---

### Task Management (`task-management.md`)

| Guideline | Detail |
|---|---|
| Custom task prefix | User can supply a Jira-style ID (e.g. `PROJ-123`) — used verbatim |
| Auto-assigned ID | `TASK-NNN` zero-padded; counter only increments for auto-assigns |
| Duplicate check | Warns if supplied ID already exists as a task file |
| Requirements template | Task file includes Acceptance Criteria, Constraints/Notes, Out of Scope sections |
| Registry | `tasks/registry.md` tracks all tasks with status; auto-created if missing |
| Review gate | Task summary + file path presented; user fills Requirements, then confirms |
| Status progression | `Open` → `In Progress` → `Complete` |
| Phase checkboxes | Each phase marked `[x]` with approval timestamp when complete |
| Deliverables | Populated after BUILD: created/modified files grouped by track |
| Decisions tagging | All Q&A decisions written to `decisions.md` prefixed with task ID |
| Retroactive tests | `"generate tests for TASK-ID"` — works on any completed task |
| Retroactive docs | `"document TASK-ID"` — works on any completed task |

---

### Session Continuity (`session-continuity.md`)

| Guideline | Detail |
|---|---|
| Auto-detection | Reads `aidlc-docs/state.md` on every session start |
| Resume interrupted init | Detects `Phase: INIT`, re-presents layout prompt |
| Resume in-progress task | Detects active phase, shows task + last audit line, offers to continue or start new |
| Context loading | Loads only the files relevant to the current phase (no unnecessary reads) |
| New task on complete project | `Phase: COMPLETE` → skips resume, goes straight to TASK INIT |

---

### Project Init (`init.md`)

| Guideline | Detail |
|---|---|
| Trigger condition | `state.md` absent **and** no source code found in workspace |
| Skipped for brownfield | Any existing source files → skips directly to TASK INIT |
| Runs once | Never re-runs after the first greenfield setup |
| Presets | Fullstack, Backend only, Frontend only, Mobile + Backend, Monorepo, Custom |
| Output | Root folders created with `.gitkeep`; `state.md` written with `Phase: COMPLETE` |
| Audit | Logs start + completion events to `audit.md` |

---

### Understand Phase (`understand.md`)

| Guideline | Detail |
|---|---|
| Always runs | Every task goes through UNDERSTAND (never skipped) |
| Workspace scan | Reads project structure before generating anything |
| Reverse Engineering | **Mandatory for brownfield** — generates 5 artifacts: business overview, architecture, code structure, API documentation, technology stack |
| RE staleness check | Compares `reverse-engineering/timestamp.md` against `git log -1`; re-scans only when stale |
| RE reuse | If artifacts are current, loads from files — no re-scan needed |
| Q&A file | Created at `qa-understand.md` when clarification is needed (0 questions → skipped) |
| Decisions snapshot | Q&A answers written to `decisions.md` tagged with task ID |
| Gate | Presents understanding summary → waits for user approval |

---

### Plan Phase (`plan.md`)

| Guideline | Detail |
|---|---|
| Conditional | Skipped for trivial / simple / single-file / bug-fix tasks |
| Scope analysis | Identifies components and units affected |
| Component decomposition | For complex tasks: breaks work into units with clear interfaces |
| Q&A file | Created at `qa-plan.md` when design decisions are needed |
| Gate | Presents execution plan inline → waits for user approval |

---

### Build Phase (`build.md`)

| Guideline | Detail |
|---|---|
| Always runs | Every task that passes UNDERSTAND proceeds to BUILD |
| Track detection | Identifies Backend / Frontend / Mobile tracks from plan/requirements |
| Track order | User chooses order for multi-track tasks; AI recommends Backend-first by default |
| Per-track gate | Code plan + completion summary presented per track (multi-track only) |
| Single-track shortcut | Skips track selection and per-track gate; one code generation pass |
| Brownfield safety | Checks file existence before generating; always modifies in-place |
| Security (auto) | Input validation, no hardcoded secrets, auth checks, SQL injection prevention, XSS sanitization — applied automatically, no opt-in |
| UI `data-testid` | All interactive elements get `data-testid="{component}-{role}"` automatically |
| Optional testing | After all tracks: user chooses Yes (generate now) or No (skip) |
| Test coverage | Unit tests: happy path, edge cases, error handling, boundary values |
| Integration scenarios | Cross-component / cross-track flows |
| Test co-location | Test files placed alongside source following existing project conventions |
| Retroactive tests | `"generate tests for TASK-ID"` generates tests for any completed task |
| Build instructions | Appended inline after testing; saved to `build-and-test.md` for complex multi-unit projects |
| Gate | Final summary across all tracks → user approves before COMPLETE |

---

### Document Phase (`document.md`)

| Guideline | Detail |
|---|---|
| Optional | Presented after BUILD; user chooses Yes / No |
| Retroactive | `"document TASK-ID"` — triggers Phase 4 for any completed task |
| Scaffold | Creates `docs/` structure at workspace root if missing |
| Sections | README hub, getting-started, overview, architecture, API reference, guides, features, backend conventions, ADRs |
| Selective creation | Only creates sections relevant to the project (e.g. no `frontend/` for backend-only) |
| Source-driven | Loads deliverable files and existing docs to write accurate content |
| Gate | Presents doc summary → waits for user approval |

---

## Files Created in Your Project

All workflow files go inside `aidlc-docs/`. Your application code is never touched by the workflow.

```
aidlc-docs/
  state.md              # current phase tracker
  audit.md              # approval timestamps log
  decisions.md          # Q&A decision snapshots, tagged by task ID
  ideas/                # standalone idea backlog — independent of workflow
    registry.md         # index of all ideas + status
    IDEA-001.md         # individual idea files (user fills in)
    ...
  reverse-engineering/  # brownfield only — written once, reused across tasks
    business-overview.md
    architecture.md
    code-structure.md
    api-documentation.md
    technology-stack.md
    timestamp.md        # staleness check vs git last commit
  qa-understand.md      # Phase 1 questions for you to answer
  qa-plan.md            # Phase 2 questions for you to answer
  requirements.md       # requirements doc (standard/complex only)
  design.md             # design decisions (complex only)
  build-and-test.md     # build instructions (complex multi-unit only)
  tasks/
    registry.md         # master list of all tasks + status
    TASK-001.md         # auto-assigned task ticket
    PROJ-123.md         # user-supplied Jira prefix example
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

Sections are created only when relevant.

---

## Example Files

### Idea file

```markdown
# IDEA-001: Smarter search with semantic ranking

**Status**: Refined
**Created**: 2026-05-18T09:00:00Z
**Tags**: search, UX

## Problem Statement
Search returns exact keyword matches only — misses synonyms and intent.

## Motivation
Users abandon search when results are poor; churn increases.

## Rough Solution
Add a vector embedding layer on top of the existing index.

## Open Questions
- [ ] Which embedding model? (OpenAI vs local)
- [ ] Acceptable latency budget?

## Rough Acceptance
Search returns relevant results even when the query uses different words.
```

### Task ticket

```markdown
# PROJ-123: Add password reset flow

**Status**: Complete
**Created**: 2026-05-01T10:00:00Z
**Tests**: Generated — 12 unit tests, 3 integration scenarios

## Request
Add a password reset flow with email verification

## Requirements

### Acceptance Criteria
- [ ] User receives email with reset link within 2 minutes
- [ ] Link expires after 1 hour

### Constraints / Notes
- Use existing mailer service (MailerModule)

### Out of Scope
- SMS verification

## Phases
- [x] UNDERSTAND — approved 2026-05-01T10:05:00Z
- [x] PLAN — approved 2026-05-01T10:25:00Z
- [x] BUILD — approved 2026-05-01T11:40:00Z

## Deliverables
- Created: `src/auth/password-reset.service.ts`
- Created: `src/auth/password-reset.controller.ts`
- Modified: `src/auth/auth.module.ts`
```

### Task registry

```markdown
# Task Registry

**Next Task**: TASK-002

| Task     | Title                    | Status   | Created              |
|----------|--------------------------|----------|----------------------|
| PROJ-123 | Add password reset flow  | Complete | 2026-05-01T10:00:00Z |
| TASK-001 | Fix login redirect bug   | Complete | 2026-05-01T11:00:00Z |
```

---

## Resuming a Session

If you close and reopen Claude Code mid-task, Claude detects the in-progress task automatically:

```
## Resuming Project

Active Task: PROJ-123 — Add password reset flow
Phase: BUILD
Last action: PLAN approved 2026-05-01T10:25:00Z

Continue PROJ-123, or start a new task?
```

---

## Key Differences from Full AIDLC

| | Full AIDLC | AIDLC Minimal |
|---|---|---|
| Approval gates | 13+ | **5** (TASK INIT + UNDERSTAND + PLAN + BUILD + DOCUMENT) |
| Rule files | 28 | **8** |
| Docs per project | 20+ | **2–8** |
| Task ID format | None | **Custom prefix or auto-assigned TASK-NNN** |
| Requirements capture | In conversation | **Structured template in task file** |
| Idea backlog | None | **Standalone IDEA-NNN files with promotion flow** |
| Q&A format | `[Answer]:` in file | `[Answer]:` in file |
| Decision log | Per-interaction audit | **Key decisions only** |
| Documentation phase | Manual | **Optional, sources-driven** |
| Welcome message | Every session | **None** |

---

## Credits

This project was heavily inspired by and adapted from the official [AWS AIDLC Workflows](https://github.com/awslabs/aidlc-workflows#usage). The minimal version strips away the enterprise ceremony while keeping the core adaptive and phase-based philosophy intact.
