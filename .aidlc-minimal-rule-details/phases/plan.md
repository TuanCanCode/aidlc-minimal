# Phase 2: PLAN

**Purpose**: Define what to build and how, for non-trivial work.

---

## Skip Criteria (evaluate before running any step)

**Skip this entire phase** when the request is:
- Single file change
- Bug fix with a clear reproduction path
- Simple enhancement with an obvious implementation
- Pure internal refactor with no new components or APIs
- UI-only change with no business logic

If skipping: proceed directly to Phase 3 (BUILD).

---

## Step 1: Scope Analysis

Based on Phase 1 findings, identify:
- Which existing components/modules are affected
- New components or services to create
- Data model changes (new tables, schema updates)
- API changes (new endpoints, modified contracts)
- Dependencies and ordering constraints between changes

---

## Step 2: Unit Decomposition (only for multi-component work)

If the work spans multiple independent units, define each unit briefly:

```
Unit: [name]
  Scope:       [what it covers]
  Depends on:  [other units or external services it needs]
  Delivers:    [interface or artifact it produces]
```

Build order: list units in dependency sequence.

**Single-unit work**: skip this step.

---

## Step 3: Design (only when needed)

Execute when:
- New components/services are being created
- Complex business logic with multiple scenarios
- Non-obvious architectural decision exists
- NFR constraints need to be addressed (performance, security, scale)

Design output (present inline in chat):
- Component responsibilities and boundaries
- Key method signatures / interface contracts
- Data models (only new ones; reference existing by name)
- NFR approach (caching strategy, auth pattern, etc.)

**For complex designs**: save as `aidlc-docs/design.md`.
**For simple designs**: keep inline in chat only.

---

## Q&A Decision Snapshot

If design decisions need user input (architecture choice, tech stack, patterns), create `aidlc-docs/qa-plan.md`:

```markdown
# Design Q&A

**Instructions**: Fill in each `[Answer]:` tag below, then return to the AI session.

---

**Q1**: [question]
[Answer]: 

**Q2**: [question]
[Answer]: 
```

Tell the user: *"Please fill in `aidlc-docs/qa-plan.md` and come back."*

Wait for the user to return, then read the file and append to `aidlc-docs/decisions.md`:

```markdown
## PLAN — [ISO timestamp]

| Question | Answer | Decision |
|---|---|---|
| [question] | [user's answer] | [what was decided] |
```

Capture architectural choices, tech stack picks, pattern decisions — anything non-obvious that future sessions should know.

---

## Gate: Phase 2 Approval

Present the plan inline:

```
## Execution Plan ✓

**Units**: [1 / N — list names if multiple]
**Build order**: [unit sequence or "single unit"]

[For each unit:]
### [Unit name]
- Files to create: [list]
- Files to modify: [list]
- Key logic: [1–2 sentences]

**Total steps**: ~[N]

Approve to start code generation?
```

Wait for explicit user confirmation before advancing to Phase 3.

Update `aidlc-docs/state.md` Phase field to `BUILD`.
Update `aidlc-docs/tasks/TASK-NNN.md`: mark `[x] PLAN — approved [timestamp]`.
Log to `aidlc-docs/audit.md`:
```
[timestamp] TASK-NNN PLAN approved: [N units], [key decisions summary]
```
