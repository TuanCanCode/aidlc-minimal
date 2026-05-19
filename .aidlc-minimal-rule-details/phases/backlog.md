# Backlog Management

**Purpose**: Break down a fleshed-out idea into a trackable list of high-level implementation items. Each item can become a task, created one by one, tracked through the full AIDLC workflow. For small ideas that need no breakdown, skip this phase and promote directly to a task.

---

## Tier Overview

```
Idea (vision, context, rough solution)
    │
    ├── [small idea]  "promote IDEA-NNN"  ──────────────────→  Task → AIDLC workflow
    │
    └── [larger idea] "backlog IDEA-NNN"
            │
            ▼
        Backlog: IDEA-NNN.md  (AI-generated checklist of implementation items)
            │
            └── "task IDEA-NNN [item]"  ──────────────────────→  Task → AIDLC workflow
                                                                  (one item at a time)
```

- **Idea**: freeform vision — the "what" and "why"
- **Backlog**: AI-generated breakdown — the "what to build" at high level, item by item
- **Task**: one backlog item in flight — goes through UNDERSTAND → PLAN → BUILD

---

## Triggers

| Phrase | Action |
|---|---|
| `"backlog IDEA-NNN"` | AI reads the idea and generates an implementation breakdown |
| `"task IDEA-NNN [item description or number]"` | Create a task from a specific backlog item |
| `"task IDEA-NNN"` | Present unchecked items and ask which one to turn into a task |
| `"list backlog"` / `"show backlog"` | List all backlog files with per-idea progress |
| `"show backlog IDEA-NNN"` | Show the full checklist for a specific idea |

Do NOT run the normal AIDLC workflow when triggering `"backlog IDEA-NNN"` or listing/showing backlog. Only `"task IDEA-NNN …"` continues into TASK INIT.

---

## Folder Structure

```
aidlc-docs/
  backlog/
    IDEA-001.md     # breakdown for IDEA-001 — one file per idea
    IDEA-002.md
    ...
```

No registry file needed — the AI lists backlog files by scanning the folder.

---

## Generating a Backlog (`"backlog IDEA-NNN"`)

### Step 1: Read the idea

Read `aidlc-docs/ideas/IDEA-NNN.md`. Extract:
- Title
- Problem Statement
- Rough Solution
- Rough Acceptance
- Open Questions
- References

If the idea file does not exist, tell the user: `IDEA-NNN not found. Capture it first with "capture idea: [title]".`

### Step 2: Check if a backlog already exists

Check `aidlc-docs/backlog/IDEA-NNN.md`:
- **Exists** → warn the user: `A backlog for IDEA-NNN already exists. Say "show backlog IDEA-NNN" to review it, or confirm to regenerate.` Wait for confirmation before overwriting.
- **Not exists** → continue.

### Step 3: Generate implementation items

Analyze the idea content and produce a list of high-level implementation items. Rules:
- Each item is a concrete unit of work, implementable as a single task
- Items should be independent where possible (no hard sequential dependency)
- Aim for 3–10 items; more than 10 suggests the idea needs splitting
- Write each item as an action phrase: `"Build X"`, `"Add Y"`, `"Design Z"`
- Do NOT include items for testing or documentation — those are handled per-task in BUILD/DOCUMENT phases
- Order items logically (data model / API / UI / integration is a common flow)

### Step 4: Create the backlog file

Create `aidlc-docs/backlog/IDEA-NNN.md`:

```markdown
# Backlog: IDEA-NNN — [Idea Title]

**Idea**: IDEA-NNN
**Created**: [ISO timestamp]
**Progress**: 0 / N items

---

## Implementation Items

- [ ] [Item 1]
- [ ] [Item 2]
- [ ] [Item 3]
...
```

### Step 5: Update the idea file

Update `aidlc-docs/ideas/IDEA-NNN.md`:
- Set `**Status**: Backlogged`
- Append to Status Log: `| [ISO date] | Backlogged | Breakdown → aidlc-docs/backlog/IDEA-NNN.md |`

Update `aidlc-docs/ideas/registry.md` status column to `Backlogged`.

### Step 6: Present for review (GATE)

Present the generated breakdown and ask the user to review:

```
## Backlog Generated: IDEA-NNN — [Title]

File: `aidlc-docs/backlog/IDEA-NNN.md`

**Implementation Items** (N total):
1. [Item 1]
2. [Item 2]
3. [Item 3]
...

Review the list and:
- **Confirm** → backlog is ready; create tasks with `"task IDEA-NNN [item number or description]"`
- **Adjust** → tell me what to add, remove, or reword
```

Wait for user confirmation or adjustment before stopping.

**Stop here after confirmation.** Do NOT proceed to UNDERSTAND / PLAN / BUILD.

---

## Creating a Task from a Backlog Item (`"task IDEA-NNN …"`)

### When the user specifies an item

`"task IDEA-NNN 3"` or `"task IDEA-NNN Build the notification service"`:

1. Read `aidlc-docs/backlog/IDEA-NNN.md`.
2. Match the item by number or description.
3. If the item is already checked (`[x]`), warn: `Item N is already done (→ TASK-ID). Pick a different item or say "task IDEA-NNN" to see remaining items.`
4. Run **Step 1: TASK INIT** from `task-management.md`, pre-populating from:
   - Task title ← item description
   - `## Request` ← item description + idea context (Problem Statement + Rough Solution, summarised)
   - `## Requirements / Acceptance Criteria` ← blank (user fills in during TASK INIT gate)
5. After the task file is created and the user confirms (TASK INIT gate), update the backlog:
   - Mark item as `[x]` and append `→ TASK-ID`
   - Update `**Progress**` count
6. Continue the AIDLC workflow from UNDERSTAND.

### When the user does not specify an item

`"task IDEA-NNN"`:

1. Read `aidlc-docs/backlog/IDEA-NNN.md`.
2. List all unchecked items:

```
## IDEA-NNN — Remaining Items

1. [Item 1]
2. [Item 3]
4. [Item 4]

Which item would you like to turn into a task? Reply with the number or description.
```

3. Wait for the user's reply, then proceed as above.

---

## Listing All Backlogs (`"list backlog"`)

Scan `aidlc-docs/backlog/` for all `IDEA-NNN.md` files. For each, read the progress count and present:

```
## Backlog Overview

| Idea | Title | Progress |
|---|---|---|
| IDEA-001 | Real-time notifications | 2 / 5 items |
| IDEA-002 | Dark mode support | 0 / 3 items |
| IDEA-003 | Smarter search | 4 / 4 items ✓ |
```

If the folder is empty or does not exist: `No backlogs yet. Say "backlog IDEA-NNN" to generate a breakdown for an idea.`

---

## Showing a Single Backlog (`"show backlog IDEA-NNN"`)

Read `aidlc-docs/backlog/IDEA-NNN.md` and present the full checklist:

```
## Backlog: IDEA-NNN — [Title]

Progress: N / M items

- [x] Item 1 → TASK-001
- [x] Item 2 → TASK-003
- [ ] Item 3
- [ ] Item 4
```

---

## Updating Backlog Progress

The AI updates the backlog file automatically whenever a task is created from a backlog item (see "Creating a Task from a Backlog Item" above). The user does not update the backlog manually.

Progress format in the file header: `**Progress**: N / M items` — update N each time an item is checked off.

---

## Idea Status: `Backlogged`

When a backlog is generated for an idea, the idea's status is set to `Backlogged`. This means:
- The idea has been broken down into implementation items
- Work has not necessarily started — items may still all be unchecked
- The status does NOT change as items are completed (idea status is set once)
- When all items are done (all tasks complete), the idea can be manually set to `Promoted` or `Complete` by the user
