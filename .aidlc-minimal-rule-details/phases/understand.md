# Phase 1: UNDERSTAND

**Purpose**: Know what exists and what to build.

---

## Step 1: Workspace Scan (auto, no approval needed)

> **Session routing already happened before this phase** (CLAUDE.md Step 0).
> Phase 1 always runs for every new task — do NOT re-check `state.md` or redirect to session-continuity here.

Scan workspace for source code (`.java`, `.ts`, `.py`, `.go`, `.kt`, etc.) and build files (`package.json`, `pom.xml`, `build.gradle`, etc.):
- **Has code** → `brownfield = true`, proceed to Step 2
- **No code** → `brownfield = false`, skip to Step 3

If `aidlc-docs/state.md` does not exist yet, create it now (see session-continuity.md for format).
If it already exists, update the `Phase` field to `UNDERSTAND`.

Log to `aidlc-docs/audit.md`:
```
[timestamp] TASK-NNN UNDERSTAND start: [one-line request summary]
```

---

## Step 2: Reverse Engineering (brownfield only)

### 2.1 Check for existing artifacts

Check `aidlc-docs/reverse-engineering/timestamp.md`:

**If exists** → read `Analysis Date` from it:
- Run `git log -1 --format=%cI -- .` to get last commit timestamp
- **Artifact is newer than last commit** → load all files in `aidlc-docs/reverse-engineering/` into context, **skip to Step 3** (no approval needed, auto-proceed)
- **Artifact is older than last commit** → codebase changed, re-run reverse engineering below
- **User explicitly requests rerun** → re-run regardless

**If not exists** → run full reverse engineering below.

---

### 2.2 Discovery — scan the full codebase before writing any files

#### Discovery A — Project & Build System
- Identify all packages/modules (`package.json`, `pom.xml`, `build.gradle`, `go.mod`, `Cargo.toml`, etc.)
- Determine package types: Application, Infrastructure (CDK/Terraform), Shared/Models, Clients, Tests
- Map build dependencies between packages
- Read CI/CD config files (`.github/workflows/`, `Jenkinsfile`, `buildspec.yml`, etc.)

#### Discovery B — Business Context
- Read entry points (main files, Lambda handlers, controllers, routes) to understand what the system *does*
- Identify key business transactions (e.g., "Process order", "Authenticate user", "Send notification")
- Identify domain terminology used in the code (ubiquitous language)

#### Discovery C — Architecture & Components
- Identify architecture pattern: Monolith / Layered / Microservices / Serverless / Event-driven / Hexagonal
- For each component/package: purpose, responsibilities, what it owns, what it calls
- Map service-to-service communication (REST, events, queues, shared DB)
- Identify infrastructure services: databases, caches, queues, object storage, CDN

#### Discovery D — Code Structure & Patterns
- Identify layers within each component (controller → service → repository, etc.)
- Identify design patterns (Repository, Factory, Strategy, Observer, CQRS, etc.)
- Note naming conventions (camelCase, snake_case, PascalCase per language/file type)
- Note folder/module structure conventions
- Identify error handling patterns (custom exceptions, error codes, Result types)

#### Discovery E — API Contracts
- Scan all REST/GraphQL endpoints: method, path, purpose, auth requirement
- Read request/response schemas for key endpoints
- Identify all data models/entities (DB schemas, DTOs, domain objects) with their fields
- Note API versioning strategy if present

#### Discovery F — Dependencies
- Internal: which packages depend on which, and why (build order)
- External: key third-party libraries, versions, and usage
- Cloud services in use (specific services: DynamoDB, S3, SQS, Pub/Sub, etc.)

#### Discovery G — Code Quality
- Test coverage indicators (test files present? coverage config? approximate coverage)
- Linting/formatting config (ESLint, Checkstyle, Pylint, etc.)
- CI/CD pipeline summary
- Known technical debt (TODO/FIXME/HACK comments, deprecated usages)

---

### 2.3 Generate artifact files

Create all files below inside `aidlc-docs/reverse-engineering/`. Output all files before presenting the approval message.

#### File 1: `business-overview.md`
```markdown
# Business Overview

## System Purpose
[1–2 sentence description of what the system does from a business perspective]

## Business Transactions
| Transaction | Description |
|---|---|
| [name] | [end-to-end description] |

## Domain Terminology
| Term | Meaning in this codebase |
|---|---|
| [term] | [definition] |

## Component Business Descriptions
### [Package/Component]
- **Purpose**: [business perspective]
- **Responsibilities**: [key responsibilities]
```

#### File 2: `architecture.md`
```markdown
# System Architecture

## Architecture Pattern
[Monolith / Layered / Microservices / Serverless / Event-driven / Hexagonal]

## Component Overview
| Package/Module | Type | Purpose |
|---|---|---|
| `[name]` | [App/Infra/Shared/Test] | [purpose] |

## Communication Map
- `[ComponentA]` → `[ComponentB]` via [REST / event / shared DB / queue] — [reason]

## Infrastructure Services
| Service | Type | Purpose |
|---|---|---|
| [name] | [DynamoDB/S3/SQS/RDS/etc.] | [purpose] |

## Component Detail
### [Component/Package Name]
- **Entry points**: [main files, handlers, controllers]
- **Layers**: [e.g. controller → service → repository]
- **Key interfaces/contracts**: [interfaces or abstract classes exposed]
- **Owns data models**: [entities/tables]
- **Calls**: [other components or external services]
```

#### File 3: `code-structure.md`
```markdown
# Code Structure

## Build System
- **Type**: [Maven/Gradle/npm/etc.]
- **Key config files**: [list]

## Design Patterns
| Pattern | Location | Purpose |
|---|---|---|
| [pattern] | [where used] | [why] |

## Conventions
**Naming**:
- Files: [convention]
- Classes: [convention]
- Variables/methods: [convention]
- DB tables/columns: [convention]

**Folder structure**: [observed pattern]
**Error handling**: [pattern in use]

## File Inventory
> Used during BUILD to determine modify-vs-create for each file.

### [Component/Package]
- `[path/to/file]` — [class/module]: [purpose]
```

#### File 4: `api-documentation.md`
```markdown
# API Documentation

## REST / GraphQL Endpoints
### [Component Name]
| Method | Path | Purpose | Auth |
|---|---|---|---|
| GET | `/api/[path]` | [purpose] | Yes/No |
| POST | `/api/[path]` | [purpose] | Yes/No |

## Data Models
### [ModelName]
| Field | Type | Required | Description |
|---|---|---|---|
| [field] | [type] | Yes/No | [description] |

## Internal Interfaces
### [Interface/Class Name]
- **Methods**: [signatures]
- **Purpose**: [what it abstracts]
```

#### File 5: `technology-stack.md`
```markdown
# Technology Stack & Dependencies

## Tech Stack
| Category | Technology | Version | Usage |
|---|---|---|---|
| Language | [e.g. TypeScript] | [version] | [what for] |
| Framework | [e.g. NestJS] | [version] | [what for] |
| Build | [e.g. npm] | [version] | [what for] |
| Test | [e.g. Jest] | [version] | [what for] |

## Internal Dependencies
| Package | Depends On | Reason |
|---|---|---|
| `[pkg A]` | `[pkg B]` | [why] |

**Build order**: [pkg1] → [pkg2] → [pkg3]

## External Dependencies
| Library | Version | Purpose |
|---|---|---|
| [name] | [version] | [usage] |

## Code Quality
- **Unit tests**: [Present/Absent — coverage]
- **Integration tests**: [Present/Absent]
- **Linting**: [tool + config file]
- **CI/CD**: [tool + pipeline file]
- **Technical debt**: [notable TODOs, FIXMEs, deprecated usages]
```

#### File 6: `timestamp.md`
```markdown
# Reverse Engineering Metadata

**Analysis Date**: [ISO timestamp]
**Workspace**: [absolute path]
**Total files analyzed**: [N]

## Artifacts Generated
- [x] business-overview.md
- [x] architecture.md
- [x] code-structure.md
- [x] api-documentation.md
- [x] technology-stack.md
```

---

### 2.4 Present completion message and wait for approval

```
## Reverse Engineering Complete

[AI-generated bullet-point summary of key findings]:
- Architecture: [pattern], [N] components
- Business transactions: [list key ones]
- API endpoints: [N total]
- Tech stack: [key technologies]
- Files analyzed: [N]

Artifacts saved to: aidlc-docs/reverse-engineering/

Please review the artifacts. Request changes or approve to continue to Requirements.
```

**Wait for explicit user approval before proceeding to Step 3.**
- User approves → proceed to Step 3
- User requests changes → update relevant artifact files, re-present message
- Log to `aidlc-docs/audit.md`: `[timestamp] TASK-NNN REVERSE-ENGINEERING approved`

---

## Step 3: Requirements Gathering

### How many questions to generate

| Request clarity | Questions |
|---|---|
| Crystal clear, specific | 0 — skip Q&A file, go straight to summary |
| Mostly clear, minor gaps | 1–3 |
| Needs clarification | 3–6 |
| Vague or ambiguous | up to 8 |

### Question focus areas (only ask what is genuinely unclear)

- What is the core goal / problem being solved?
- Which users or roles are affected?
- What are the key functional requirements?
- Edge cases or error scenarios that must be handled?
- Any performance, security, or scale constraints?
- Integration points with existing systems?
- Any existing patterns or conventions to follow?

### Create Q&A file

Create `aidlc-docs/qa-understand.md` with questions using `[Answer]:` tags:

```markdown
# Requirements Q&A

**Instructions**: Fill in each `[Answer]:` tag below, then return to the AI session.

---

**Q1**: [question]
[Answer]: 

**Q2**: [question]
[Answer]: 

**Q3**: [question]
[Answer]: 
```

Tell the user: *"Please fill in `aidlc-docs/qa-understand.md` and come back."*

Wait for the user to return with answers before proceeding.

### After receiving answers

Read `aidlc-docs/qa-understand.md`, then append a decision snapshot to `aidlc-docs/decisions.md`:

```markdown
## UNDERSTAND — [ISO timestamp]

| Question | Answer | Decision |
|---|---|---|
| [question] | [user's answer] | [what was decided] |
```

Only record questions that influenced a decision. Skip trivial confirmations.

**Trivial / Simple**: keep requirements in context only.
**Standard / Complex**: create `aidlc-docs/requirements.md`:
```markdown
# Requirements

## Request
[Original user request]

## Intent
- Type: [New Feature / Bug Fix / Refactor / Upgrade / etc.]
- Scope: [Single file / Component / Multi-component / System-wide]
- Complexity: [Trivial / Simple / Standard / Complex]

## Functional Requirements
- [FR1] ...
- [FR2] ...

## Non-Functional Requirements
- [NFR1] ...

## Out of Scope
- ...
```

---

## Gate: Phase 1 Approval

Present this summary in chat:

```
## Understanding Complete ✓

**Project**: [Greenfield / Brownfield — tech stack if brownfield]
**Type**: [New Feature / Bug Fix / Refactor / etc.]
**Scope**: [Single file / Component / Multi-component / System-wide]
**Complexity**: [Trivial / Simple / Standard / Complex]

**What will be built**:
- [bullet 1]
- [bullet 2]
- [bullet 3 if needed]

**Phase 2 (Plan)**: [WILL RUN — reason] OR [SKIPPED — simple scope, proceeding directly to Build]

Ready to proceed?
```

Wait for explicit user confirmation before advancing to Phase 2 (or Phase 3 if Phase 2 is skipped).

Update `aidlc-docs/state.md` Phase field to `PLAN` (or `BUILD` if skipping).
Update `aidlc-docs/tasks/TASK-NNN.md`: mark `[x] UNDERSTAND — approved [timestamp]`, set Status to `In Progress`.
Update registry row status to `In Progress`.
Log to `aidlc-docs/audit.md`:
```
[timestamp] TASK-NNN UNDERSTAND approved: [scope], [complexity], Phase 2 [run/skip]
```
