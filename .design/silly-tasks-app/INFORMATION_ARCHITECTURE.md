# Information Architecture: The Ministry of Silly Tasks

## View Map

This is a single-page application with no routing. "Views" are layered states on top of the base page — sheets, loading states, and empty states rather than distinct URLs.

```
/ (single URL, static HTML)
│
├── [State: Loading]           — Poster image + fade-in sequence, auto-dismisses
│
├── [State: App / Base View]   — Always visible once loaded
│   ├── Header zone            — Ministry label, title, today's date
│   ├── Stats bar              — Overdue / Due Today / Upcoming / Total counts
│   ├── Tag filter strip       — Department filter pills (dynamic)
│   ├── Task list              — Active tasks, then divider, then filed tasks
│   │   ├── [State: Empty]     — No tasks, or filter returns nothing
│   │   └── [State: Card snap] — Card swiped open revealing action button
│   └── Footer                 — Byline
│
├── [Layer: FAB]               — Persistent floating action button
│
├── [Layer: Sheet — Add/Edit]  — Slides up over base view (Form ST-7 / AM-3)
│   └── Backdrop (dismisses sheet on tap)
│
└── [Layer: Sheet — Delete]    — Slides up over base view (Form DX-1)
    └── Backdrop (dismisses sheet on tap)
```

## Navigation Model

- **Primary navigation**: None. There is no nav bar, sidebar, or menu. The entire product lives on one view.
- **Secondary navigation**: Tag filter pills act as in-page content filters. Not navigation — they don't change the URL or view structure, only the visible task set.
- **Utility navigation**: None. No account, settings, or help links.
- **Mobile navigation**: No hamburger or bottom tabs. The FAB is the only persistent action affordance. Everything else is accessed through the task list itself (swipe) or the FAB.

## Content Hierarchy

### Base View (Task List)

1. **Overdue tasks** — Highest urgency, rendered first with heat-5/4/3 classes and stinky stamp. The user needs to see these immediately on open.
2. **Due today** — heat-2 class. Second most critical.
3. **Upcoming tasks** — heat-1, cool-1, cool-2. Present but not alarming.
4. **Stats bar** — Summary counts that confirm what the list shows. Sits above the list to give a fast numeric read before scrolling.
5. **Tag filters** — Below stats. Used occasionally, not on every visit.
6. **Filed tasks** — Below the divider. Completed tasks the user doesn't need to act on. Visually de-emphasised (50% opacity).
7. **Footer** — Purely decorative copy. Lowest priority.

### Add / Edit Sheet

1. **Title field** ("Nature of the Duty") — Autofocused on open. The only required field.
2. **Recurrence selector** ("Recurrence") — Defaults to Monthly. Most decisions are made here.
3. **Department tag** ("Department") — Optional. Datalist-assisted from existing tags.
4. **Due date** ("Date of Next Reckoning") — Defaults to today for add. Pre-filled for edit.
5. **Submit / cancel** — Submit is primary action; "Abandon Post" is low-prominence text link.

### Delete Confirmation Sheet

1. **Task name** — Shown prominently so the user confirms they're deleting the right duty.
2. **Confirm button** ("Expunge the Duty") — Primary destructive action.
3. **Cancel** ("Retain on File") — Low-prominence, no visual alarm.

## User Flows

### Mark a task complete
1. User opens app, sees task list sorted by urgency
2. User taps a task card
3. Card transitions to `just-done` state (strikethrough, muted, 50% opacity)
4. Task is rescheduled by its interval; card moves below the "filed with the Ministry" divider
5. Stats bar updates

### Add a new task
1. User taps FAB (+)
2. "Lodge a New Duty" sheet slides up; title field autofocuses
3. User enters title (required), optionally sets department and recurrence
4. User sets due date (defaults to today)
5. User taps "File with the Ministry"
   - Success → sheet closes, new card enters list with stagger animation
   - Error → inline error, sheet stays open
6. Stats bar updates

### Edit an existing task
1. User swipes a task card left past threshold (~70px)
2. Card snaps open; "Amend Record" button revealed on right
3. User taps "Amend Record"
4. Card snaps closed; "Amend the Record" sheet slides up, fields pre-filled
5. User edits fields, taps "Amend the Record"
   - Success → sheet closes, card updates in place
   - Error → inline error, sheet stays open
6. User taps elsewhere or swipes card back to dismiss without editing

### Delete a task
1. User swipes a task card right past threshold
2. Card snaps open; "Strike from Record" button revealed on left
3. User taps "Strike from Record"
4. Dark "Form DX-1" delete confirmation sheet slides up
5. User taps "Expunge the Duty"
   - Success → sheet closes, card exits, task removed from list
   - User taps "Retain on File" → sheet closes, card snaps back
6. Stats bar updates

### Filter by department
1. User taps a tag pill (e.g., "Kitchen")
2. Task list re-renders showing only tasks with that tag
3. Stats bar updates to reflect filtered set
4. Active pill gets accent fill
5. User taps "All" to reset

### Empty state (no tasks / filtered to nothing)
1. Task list is empty
2. In-character message shown in place of cards
3. If filtered: prompt to clear filter or add a task in this department
4. If genuinely empty: prompt to lodge a first duty via FAB

## Naming Conventions

Consistency in voice is a core principle. These are the canonical terms used in UI copy.

| Concept | Label in UI | Notes |
|---|---|---|
| Task / chore | Duty | Always "duty" in Ministry copy. "Task" acceptable in technical/code contexts only. |
| Create | Lodge / File | "Lodge a New Duty" for the action; "File with the Ministry" for submit |
| Edit | Amend | "Amend the Record" — implies official correction |
| Delete | Expunge / Strike | "Expunge the Duty" (confirm) / "Strike from Record" (swipe label) |
| Cancel | Abandon Post | Sheet dismissal. Keeps the character. |
| Tag / category | Department | Tags are "departments" in UI copy |
| Completed task | Filed | "Filed with the Ministry" — past the divider |
| Recurrence / interval | Recurrence | "Recurrence" in labels; cadence labels (Weekly, Monthly, etc.) in card meta |
| Due date | Date of Next Reckoning | Form label. Cards use natural language: "Mon, Apr 20" |
| Overdue | Overdue / [heat badge] | Stats bar uses "Overdue"; cards use "3 days overdue" etc. |
| Form reference numbers | Form ST-7, AM-3, DX-1 | ST = Standard Task, AM = Amendment, DX = Delete/Expunge |

## Component Reuse Map

| Component | Used in | Notes |
|---|---|---|
| Bottom sheet pattern | Task sheet (add/edit) + Delete sheet | Same slide-up animation and backdrop. Different internal content and colour scheme. |
| Sheet backdrop | Both sheets | Shared element; closes whichever sheet is open on tap |
| Field input styles | Task sheet only | `.field-input` class — text, select, date inputs |
| Heat class system | Task cards + stats bar | Cards use full class; stats bar uses hardcoded colour per stat type |
| Playfair Display headings | Header h1, sheet titles, delete title, stat numbers, tag pills, urgency badges | Universal display type |
| Lora body | Ministry labels, metadata, footer, field labels, cancel buttons | Universal body/meta type |
| Paper grain texture | Task cards, task sheet, delete sheet | SVG noise filter via `::after` pseudo or background-image |
| Ministry label pattern | App header, sheet headers | Small-caps Lora, wide letter-spacing, muted colour |

## Content Growth Plan

**Tasks** are the only content that grows. Current structure handles growth via:
- **Tag filtering** — scales well to 10–20 tags (departments) before needing a different UI
- **Sorted by urgency** — sort order is deterministic, no manual reordering needed
- **Filed section** — completed tasks automatically segregated below divider; they disappear from the active list once within 7 days of next due date
- **No pagination needed** — household chore lists rarely exceed 30–50 tasks; single scrollable list is sufficient

If the task count grows beyond ~50, consider collapsing the "filed" section by default.

## URL Strategy

Single static HTML file. No URL strategy required.

- All state is held in memory (JavaScript `tasks` array)
- Supabase is the persistence layer; no client-side routing
- Bookmarking or sharing a specific task is out of scope
- If routing is added in future: `/` (list), `/?tag=Kitchen` (filtered) would be the natural extension
