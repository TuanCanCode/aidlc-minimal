# Idea Capture (Standalone Phase)

**Purpose**: A lightweight, freeform space to park, refine, and organize ideas before they are ready to become formal AIDLC tasks. Runs independently — never triggers UNDERSTAND / PLAN / BUILD.

---

## When to Run

Run this phase when the user says any of:
- `"capture idea: [title]"`
- `"new idea"`
- `"idea: [title]"`
- `"add idea"`

Do NOT run the normal AIDLC workflow (Step 0 session check, Task Init, phases) when this trigger is detected. This phase is fully standalone.

---

## Step 1: Ensure Ideas Folder Exists

Check if `aidlc-docs/ideas/registry.md` exists:
- **Not exists** → create it with the template below, then continue.
- **Exists** → continue.

Registry format:
```markdown
# Idea Registry

| ID | Title | Status | Created |
|---|---|---|---|
```

---

## Step 2: Assign Idea ID

Read `aidlc-docs/ideas/registry.md` and count existing rows to determine the next number.

- Format: `IDEA-NNN` (zero-padded to 3 digits, e.g. `IDEA-001`)
- Never reuse or skip numbers.

---

## Step 3: Generate the Idea File

Create `aidlc-docs/ideas/IDEA-NNN.md` using this template. Pre-fill the title from the user's prompt. Leave all body sections as placeholders — the user fills them in.

```markdown
# IDEA-NNN: [Title from user prompt]

**Status**: Raw
**Created**: [ISO timestamp]
**Tags**: 

---

## Problem Statement
> What problem or opportunity is this addressing? Be specific about the pain point.

_Fill in..._

## Motivation
> Why does this matter? What is the impact if solved — or if not solved?

_Fill in..._

## Rough Solution
> Initial thinking — what could the solution look like? Does not need to be precise.

_Fill in..._

## Open Questions
> Things to figure out before this idea is ready to plan.

- [ ] 
- [ ] 

## Rough Acceptance
> What would "done" look like? Even a vague definition helps scope later.

_Fill in..._

## References
> Links, screenshots, related tasks, Slack threads, competitor examples, designs.

- 

## Status Log

| Date | Status | Note |
|---|---|---|
| [ISO date] | Raw | Created |
```

---

## Step 4: Update the Registry

Append a row to `aidlc-docs/ideas/registry.md`:

```markdown
| IDEA-NNN | [Title] | Raw | [ISO timestamp] |
```

---

## Step 5: Present to User — No Gate, No Wait

```
## Idea Captured: IDEA-NNN — [Title]

File: `aidlc-docs/ideas/IDEA-NNN.md`

Open the file and fill in the sections at your own pace:
- **Problem Statement** — what pain or gap are you solving?
- **Motivation** — why does it matter?
- **Rough Solution** — initial thinking (can be vague)
- **Open Questions** — what do you still need to figure out?
- **Rough Acceptance** — what would done look like?
- **References** — links, designs, related context

No further action needed. Come back when ready and say:
> `"promote IDEA-NNN"` — converts it into a formal TASK and starts the AIDLC workflow.
> `"list ideas"` — shows the full idea backlog by status.
```

**Stop here.** Do NOT proceed to UNDERSTAND / PLAN / BUILD.

---

## Idea Status Values

| Status | Meaning |
|---|---|
| `Raw` | Just captured — sections not yet filled in |
| `Refined` | Key sections filled, open questions being resolved |
| `Ready` | Acceptance criteria clear enough to plan — waiting for promotion |
| `Promoted` | Converted to a TASK (links to task ID in Status Log) |
| `Parked` | On hold — not being actively developed |
| `Dropped` | Decided not to pursue |

The user updates `**Status**` in the idea file manually. The AI only updates it during promotion (to `Promoted`).

---

## Promoting an Idea to a Task

When the user says `"promote IDEA-NNN"`:

1. Read `aidlc-docs/ideas/IDEA-NNN.md`.
2. Run normal **Step 1: TASK INIT** from `task-management.md`, pre-populating the task file from idea content:
   - Task title ← idea title
   - `## Request` ← Problem Statement + Rough Solution (combined, verbatim)
   - `## Requirements / Acceptance Criteria` ← Rough Acceptance
   - `## Requirements / Constraints / Notes` ← Open Questions + References
3. Ask for a task prefix as normal (Step 1.1 in task-management.md). If References already names a ticket, suggest it as the default.
4. After creating the task file, update `aidlc-docs/ideas/IDEA-NNN.md`:
   - Set `**Status**: Promoted`
   - Append to Status Log: `| [ISO date] | Promoted | → TASK-ID |`
5. Update `aidlc-docs/ideas/registry.md` status column to `Promoted`.
6. Present the normal Task Init gate and continue the AIDLC workflow from there.

---

## Listing Ideas

When user says `"list ideas"` or `"show ideas"`:

Read `aidlc-docs/ideas/registry.md` and present grouped by status:

```
## Idea Backlog

### Ready (N)
| ID | Title | Created |
|---|---|---|
...

### Refined (N)
...

### Raw (N)
...

### Promoted / Parked / Dropped (N)
...
```

If the registry is empty, say: `No ideas captured yet. Say "capture idea: [title]" to add one.`

---

## Folder Organization (Optional)

The flat `aidlc-docs/ideas/` layout is the default. If the user wants sub-folders (by feature area, project, etc.), they can move files manually — the AI does not reorganize unless asked.

If the user asks to organize ideas into sub-folders:
1. Ask for folder names or derive from existing tags.
2. Move files and update registry paths accordingly.
3. Do not change idea IDs.
