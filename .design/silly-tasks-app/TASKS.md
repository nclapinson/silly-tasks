# Build Tasks — The Ministry of Silly Tasks

Generated from DESIGN_BRIEF.md. Each task is independently buildable.
Check off tasks as they complete.

---

## Remaining work

The core app is fully built: loading screen, task cards with heat classes, swipe actions, stats bar, tag filters, FAB, task sheet (add/edit), delete confirm sheet, and Supabase CRUD. Two components remain.

---

### Task 1 — Empty State

**What:** In-character message shown in `#task-list` when there are no cards to display. Two variants needed.

**Variant A — Genuinely empty** (no tasks in the database):
- Shown when `tasks.length === 0` after load
- Ministry voice: something like "No duties have been registered with this office." with a prompt to lodge one via the FAB
- Should feel like a blank government form waiting to be filled

**Variant B — Filter returns nothing** (tag selected, no tasks match):
- Shown when `activeTag !== 'All'` and the filtered list is empty
- Ministry voice: reference the department — e.g. "The [Kitchen] department has no outstanding duties on file."
- Include a nudge to clear the filter ("View All Departments") or lodge one in this department

**Implementation notes:**
- Replace the `#status-message` div (currently repurposed for loading/error states) with a proper `#empty-state` element, or generate the empty state inside `render()` when both `active` and `done` are empty
- Style: cream card surface, Playfair Display heading, Lora italic body, muted colour — like a form with no entries. No heat colours. Centered within the list area.
- The FAB is always present so no need to duplicate a "+" button inside the empty state — a text prompt is sufficient

**Acceptance:** Load the app with 0 tasks → empty state A shows. Select a tag with no matches → empty state B shows with correct department name.

---

### Task 2 — Toast / Feedback System

**What:** Transient confirmation message after successful CRUD actions. Replaces the current `alert()` error handling with in-character toasts.

**Success toasts** (auto-dismiss after ~3s):
- Create: "Duty lodged. The Ministry has taken note."
- Edit: "Record amended. Corrections are on file."
- Delete: "Duty expunged. It never happened."
- Complete: "Duty filed. Well done, eventually."

**Error toasts** (persist until dismissed, or longer auto-dismiss ~6s):
- Generic: "The Ministry regrets an error: [message]"
- Keep the voice deadpan — no exclamation marks, no apologies

**Design:**
- Fixed position, bottom of screen, centered (respects the 760px max-width)
- Sits above the FAB (z-index > FAB's 100)
- Cream/card background for success; dark crimson (`#1e0808`) for error — matches delete sheet palette
- Playfair Display for the message text, small, stamp-like
- Slides up from bottom on show, fades out on dismiss
- One toast at a time — new toast replaces old

**Implementation notes:**
- Single `#toast` element in the DOM, toggled via JS
- `showToast(message, type)` — type is `'success'` | `'error'`
- Remove the `alert()` calls in `createTask`, `updateTask`, `deleteTask`
- Add `showToast` call in `completeTask` on success

**Acceptance:** Complete a task → success toast appears and auto-dismisses. Force a Supabase error → error toast appears in crimson.
