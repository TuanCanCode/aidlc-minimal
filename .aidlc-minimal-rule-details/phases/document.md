# Phase 4: DOCUMENT

**Purpose**: Write or update project documentation to reflect what was built.

Triggered by user choice after BUILD is approved. Can also be triggered retroactively via `"document TASK-NNN"`.

---

## Step 0: Initialize Documentation Scaffold

**Run this step every time Phase 4 executes.** Check if `docs/` exists in the workspace root.

### If `docs/` does NOT exist → scaffold it

Create the folder structure and seed files below. Only create sections relevant to the project (e.g. skip `frontend/` for a backend-only project, skip `mobile/` if no mobile app).

```
docs/
├── README.md                          ← doc hub — created from Hub Template
├── getting-started.md                 ← how to clone, install, run locally
├── overview/
│   ├── project-description.md         ← product vision, goals, target users
│   └── tech-stack.md                  ← languages, frameworks, cloud services
├── architecture/
│   ├── system-overview.md             ← high-level system diagram + component map
│   ├── data-model.md                  ← database schemas, entity relationships
│   └── security.md                    ← auth strategy, endpoint protection, secrets
├── api/
│   ├── README.md                      ← API index — list of all services
│   └── <service-name>/
│       ├── endpoints.md               ← HTTP method + path + schema + auth
│       └── events.md                  ← webhook / event payloads
├── guides/
│   ├── README.md                      ← guides index
│   ├── contributing.md                ← code style, PR process, branch strategy
│   └── deployment.md                  ← how to deploy, environment config, CI/CD
├── features/
│   ├── README.md                      ← feature index — numbered list
│   └── 01-<feature-name>/
│       ├── README.md                  ← feature overview + file index
│       ├── <feature-name>.md          ← feature spec
│       └── implementation.md          ← implementation guideline (backend + frontend)
├── backend/
│   ├── README.md                      ← backend overview
│   └── conventions.md                 ← coding patterns, folder layout, naming
├── frontend/                          ← skip if backend-only
│   ├── README.md
│   └── conventions.md
├── mobile/                            ← skip if no mobile app
│   ├── README.md
│   └── conventions.md
└── decisions/
    └── README.md                      ← architecture decision log index
```

**Seed file rules**:
- Create `docs/README.md` using the **Hub Template** below (fill in project name, list only sections that were created)
- Create `docs/getting-started.md` with install + run instructions derived from the workspace (read `package.json`, `Makefile`, `docker-compose.yml`, etc.)
- Create each section's `README.md` with a one-line purpose and an empty index list
- Do NOT create content files yet — those come from Step 2

Present the scaffold summary:
```
## Documentation Initialized ✓

Created `docs/` with sections: [list of created sections]

Proceeding to map deliverables…
```

### If `docs/` already exists → skip scaffolding

Verify `docs/README.md` exists. If missing, create it from the Hub Template.
Proceed directly to Step 1.

---

## Documentation Structure — Reference

All project documentation lives under `docs/` in the workspace root.

### Section Purposes

| Section | What goes here | Audience |
|---|---|---|
| `getting-started.md` | Clone, install, configure, run locally | New developer |
| `overview/` | Product vision, goals, target users, tech stack | Anyone — read first |
| `architecture/` | System diagram, component map, data model, security | Developer, architect |
| `api/` | Per-service endpoint reference + event catalogue | Developer, integrator |
| `guides/` | How-to: contributing, deployment, environment setup | Developer, DevOps |
| `features/` | Feature specs, acceptance criteria, implementation notes | Developer, PM |
| `backend/` | Backend coding conventions, patterns, folder layout | Backend developer |
| `frontend/` | Frontend architecture, component patterns, state mgmt | Frontend developer |
| `mobile/` | Mobile app architecture, platform-specific patterns | Mobile developer |
| `decisions/` | Architecture Decision Records (ADRs) | Architect, tech lead |

### Naming & Structure Rules

- All file and folder names use **lowercase, hyphen-separated** words — no spaces, no underscores.
- Feature folders under `features/` are prefixed with a two-digit order number (`01-auth/`, `02-payments/`). Pick the next unused number.
- The primary feature spec file is named `<feature-name>.md`.
- Implementation guidelines are named `implementation.md` (covers both backend and frontend in one file; split into `backend-implementation.md` / `frontend-implementation.md` only when the file exceeds ~300 lines).
- Every section folder must contain a `README.md` that indexes its contents.
- Architecture Decision Records follow the format `NNN-<title>.md` (e.g. `001-use-postgresql.md`).

---

## Sources (load all that exist before writing anything)

| Source | Path | What it provides |
|---|---|---|
| Task file | `aidlc-docs/tasks/TASK-NNN.md` | Deliverables, original request, scope |
| Requirements | `aidlc-docs/requirements.md` | Functional requirements, user stories |
| Design | `aidlc-docs/design.md` | Architecture and design decisions |
| Business overview | `aidlc-docs/reverse-engineering/business-overview.md` | Domain context, terminology |
| Architecture | `aidlc-docs/reverse-engineering/architecture.md` | System architecture, component map |
| API documentation | `aidlc-docs/reverse-engineering/api-documentation.md` | Existing API contracts |
| Code structure | `aidlc-docs/reverse-engineering/code-structure.md` | Patterns, conventions, file inventory |
| Tech stack | `aidlc-docs/reverse-engineering/technology-stack.md` | Dependencies, build tools |
| Source files | Each file listed in `## Deliverables` of the task file | Actual implementation detail |

---

## Step 1: Map Deliverables to Doc Targets

Read `## Deliverables` from the task file. For each deliverable, determine which doc files to create or update:

| Deliverable type | Doc target |
|---|---|
| New feature or user-visible behaviour | `docs/features/NN-<feature>/README.md` + `<feature>.md` |
| Backend implementation | `docs/features/NN-<feature>/implementation.md` |
| Frontend implementation | `docs/features/NN-<feature>/implementation.md` |
| New or changed API endpoint | `docs/api/<service>/endpoints.md` |
| New webhook or event | `docs/api/<service>/events.md` |
| New service or architectural component | `docs/architecture/system-overview.md` |
| Database schema or entity change | `docs/architecture/data-model.md` |
| Security change (auth, permissions, secrets) | `docs/architecture/security.md` |
| Backend pattern or convention change | `docs/backend/conventions.md` |
| Frontend pattern or convention change | `docs/frontend/conventions.md` |
| Mobile-specific change | `docs/mobile/conventions.md` |
| Deployment or CI/CD change | `docs/guides/deployment.md` |
| Environment or config change | `docs/guides/deployment.md` (Environment section) |
| Technology stack change | `docs/overview/tech-stack.md` |
| Product-level behaviour change | `docs/overview/project-description.md` |
| Architectural decision (why, not just what) | `docs/decisions/NNN-<title>.md` |
| New setup steps or dependency | `docs/getting-started.md` |

Show the mapping inline before writing — then **immediately proceed** without waiting:

```
## Documentation Plan

Creating:
- `docs/features/01-auth/README.md`
- `docs/features/01-auth/auth.md` — feature spec
- `docs/features/01-auth/implementation.md` — implementation guideline

Updating:
- `docs/api/auth-service/endpoints.md` — adding POST /auth/login
- `docs/architecture/data-model.md` — adding User entity
- `docs/getting-started.md` — adding JWT_SECRET env var

Writing now…
```

---

## Step 2: Write Documentation

For each file in the plan:

1. Check if the file exists
2. **Exists** → update in-place: add the relevant section, append to a table, update a row, etc.
3. **Not exists** → create using the appropriate template below

### Content sourcing

| Content type | Pull from |
|---|---|
| User stories, acceptance criteria | `requirements.md` |
| Architecture, design decisions | `design.md` |
| Endpoint details, request/response schemas | source files + `api-documentation.md` |
| Database schemas, entity fields | source files + `architecture.md` |
| Domain terminology, system purpose | `business-overview.md` |
| Dependencies, build tools | `technology-stack.md` |
| Code patterns, folder conventions | `code-structure.md` |

### Writing guidelines

- **Be direct** — lead with what it does, not background context
- **Use tables** for structured data (endpoints, fields, env vars) — easier to scan than prose
- **Show examples** — a code snippet or curl command is worth a paragraph
- **Link, don't repeat** — cross-reference other docs instead of duplicating content (e.g. "See [data-model.md](../architecture/data-model.md) for the User entity")
- **Keep files focused** — one topic per file; split when a file exceeds ~400 lines
- **Use consistent heading depth** — `##` for major sections, `###` for subsections within

---

## Templates

### Template: Hub README (`docs/README.md`)

```markdown
# <Project Name> — Documentation

## Quick Start

See [Getting Started](./getting-started.md) for setup instructions.

## Documentation Map

| Section | What you'll find | Start here |
|---|---|---|
| [Overview](./overview/) | Product vision, goals, tech stack | [project-description.md](./overview/project-description.md) |
| [Architecture](./architecture/) | System design, data model, security | [system-overview.md](./architecture/system-overview.md) |
| [API](./api/) | Endpoint references, event catalogues | [README.md](./api/README.md) |
| [Features](./features/) | Feature specs and implementation guides | [README.md](./features/README.md) |
| [Guides](./guides/) | Contributing, deployment, how-tos | [README.md](./guides/README.md) |
| [Backend](./backend/) | Coding conventions, patterns | [conventions.md](./backend/conventions.md) |
| [Decisions](./decisions/) | Architecture decision records | [README.md](./decisions/README.md) |
```

Only list sections that actually exist. Add `frontend/` and `mobile/` rows if applicable.

---

### Template: Getting Started (`docs/getting-started.md`)

```markdown
# Getting Started

## Prerequisites
- [language] [version]
- [tool] [version]

## Setup

1. Clone the repository:
   ```bash
   git clone <repo-url>
   cd <project>
   ```

2. Install dependencies:
   ```bash
   <install command>
   ```

3. Configure environment:
   ```bash
   cp .env.example .env
   # Edit .env — see Environment Variables below
   ```

4. Run locally:
   ```bash
   <run command>
   ```

## Environment Variables

| Variable | Required | Description | Example |
|---|---|---|---|
| `DATABASE_URL` | Yes | PostgreSQL connection string | `postgresql://user:pass@localhost:5432/mydb` |

## Verify

After starting, confirm the app is running:
```bash
curl http://localhost:<port>/health
```
```

---

### Template: Feature Spec (`<feature-name>.md`)

```markdown
# <Feature Name>

## Overview
Brief summary: what this feature does and why it exists.

## User Stories
- As a <role>, I want to <action> so that <outcome>.

## Acceptance Criteria
- [ ] [criterion 1]
- [ ] [criterion 2]

## Technical Approach
High-level implementation strategy, key design choices, and dependencies.

## API Changes
| Method | Path | Description |
|---|---|---|
| POST | `/api/...` | ... |

Or: _No API changes._

## Data Model Changes
| Entity | Change | Description |
|---|---|---|
| `User` | Add field | `resetToken: string` |

Or: _No data model changes._

## Related Docs
- [System architecture](../../architecture/system-overview.md)
- [API reference](../../api/<service>/endpoints.md)
```

---

### Template: Implementation Guideline (`implementation.md`)

```markdown
# <Feature Name> — Implementation Guideline

## Scope
What this guideline covers.

## Backend

### Domain Layer
- Entities, value objects, domain services affected

### Application Layer
- Use cases / service methods

### Infrastructure Layer
- Repository implementations, external service integrations

### API Layer
- Controllers, request/response DTOs, validation

## Frontend

### UI Components
- New or modified components

### State Management
- Store changes, new actions/reducers

### API Integration
- Service calls, error handling

## Testing
- **Unit**: [what to test, expected coverage]
- **Integration**: [cross-boundary scenarios]
- **E2E**: [user flow scenarios]
```

Split into `backend-implementation.md` and `frontend-implementation.md` only when combined file exceeds ~300 lines.

---

### Template: Feature Folder `README.md`

```markdown
# <Feature Name>

One-sentence description of what this feature does.

## Contents

- [Feature spec](./<feature-name>.md) — requirements, acceptance criteria, technical approach
- [Implementation guideline](./implementation.md) — backend + frontend implementation details

**Task**: TASK-NNN
```

---

### Template: API Endpoint Entry (append to `endpoints.md`)

```markdown
## <METHOD> /<path>

**Description**: What this endpoint does.
**Auth**: Required / None
**Rate limit**: N/A or [limit]

**Request**:
```json
{
  "field": "type — description"
}
```

**Response** (`200`):
```json
{
  "field": "type — description"
}
```

**Errors**:
| Code | Reason |
|---|---|
| 400 | Invalid input |
| 401 | Not authenticated |
| 404 | Resource not found |
```

---

### Template: Data Model Entry (append to `data-model.md`)

```markdown
## <EntityName>

**Table**: `<table_name>`
**Owned by**: [service/component name]

| Column | Type | Required | Description |
|---|---|---|---|
| `id` | UUID | Yes | Primary key |
| `created_at` | timestamp | Yes | Row creation time |

**Relationships**:
- `<EntityName>` → `<OtherEntity>` (one-to-many via `<foreign_key>`)

**Indexes**:
- `idx_<table>_<column>` on `<column>` — [reason]
```

---

### Template: Architecture Decision Record (`decisions/NNN-<title>.md`)

```markdown
# ADR-NNN: <Title>

**Date**: [ISO date]
**Status**: Accepted / Proposed / Deprecated
**Task**: TASK-NNN

## Context
What situation prompted this decision?

## Decision
What did we decide and why?

## Consequences
What are the trade-offs? What becomes easier or harder?
```

---

### Template: Section `README.md` (generic)

```markdown
# <Section Name>

<One-sentence purpose of this section.>

## Contents

- [File name](./file-name.md) — brief description
```

---

## Keep Index Files Current

After writing, ensure:
- Every new folder has a `README.md` that lists its contents
- `docs/README.md` Documentation Map includes any newly added sections (create row if absent)
- `docs/features/README.md` includes an entry for any new feature folder
- `docs/api/README.md` includes an entry for any new service
- `docs/decisions/README.md` includes an entry for any new ADR

---

## Completion

Present a summary and wait for explicit user confirmation:

```
## Documentation Complete ✓

**Initialized**: [Yes — scaffolded docs/ | No — already existed]

**Created**:
- `docs/features/01-auth/README.md`
- `docs/features/01-auth/auth.md` — feature spec
- `docs/features/01-auth/implementation.md` — implementation guideline

**Updated**:
- `docs/api/auth-service/endpoints.md` — POST /auth/login, POST /auth/refresh
- `docs/architecture/data-model.md` — added User entity
- `docs/getting-started.md` — added JWT_SECRET to env vars

Please review. Request changes or approve to complete.
```

After approval, update `aidlc-docs/tasks/TASK-NNN.md`:
```
**Docs**: Generated — [list of created/updated doc files]
```

Log to `aidlc-docs/audit.md`:
```
[timestamp] TASK-NNN DOCUMENT complete: [N doc files created/updated]
[timestamp] TASK-NNN DOCUMENT approved: confirmed
```

---

## Retroactive Documentation

**Trigger**: User requests docs for a completed task — e.g.:
- `"document TASK-NNN"`
- `"write docs for TASK-NNN"`
- `"generate documentation for TASK-NNN"`

### Execution

**Step R1 — Load task context**

Read `aidlc-docs/tasks/TASK-NNN.md`:
- If `**Docs**: Generated` → tell the user docs already exist and ask if they want to regenerate/update
- Otherwise → proceed

**Step R2 — Load all sources**

Load every source listed in the Sources table above (skip missing files silently).

**Step R3 — Run Steps 0, 1, and 2**

Initialize scaffold if needed, map deliverables, and write documentation exactly as above.

**Step R4 — Update task file**

```
**Docs**: Generated (retroactive) — [list of doc files]
```

**Step R5 — Present summary**

```
## Documentation Generated for TASK-NNN ✓

**Initialized**: [Yes — scaffolded docs/ | No — already existed]
**Created**: ...
**Updated**: ...
```

Log to `aidlc-docs/audit.md`:
```
[timestamp] TASK-NNN DOCUMENT generated (retroactive): [N files]
```
