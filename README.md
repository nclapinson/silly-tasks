# The Ministry of Silly Tasks

A lightweight shared household chore tracker, styled as a British government bureaucracy that has been underfunded since 1987.

Built for two people on one shared Supabase table. No accounts, no notifications, no nonsense — just a running list of what needs doing and who hasn't done it yet.

---

## Stack

- **Frontend**: Single static HTML file (`index.html`) — vanilla JS, no build step
- **Database**: [Supabase](https://supabase.com) (Postgres) via the JS client
- **Fonts**: Playfair Display + Lora (Google Fonts)
- **Hosting**: Any static host (GitHub Pages, Netlify, etc.)

---

## Running locally

Open `index.html` in a browser. No server required — the Supabase client calls the API directly.

---

## Database setup

### Table: `tasks`

| Column | Type | Notes |
|---|---|---|
| `id` | uuid | Primary key |
| `title` | text | Required |
| `tag` | text | Department / category (optional) |
| `interval_days` | integer | Recurrence in days (7, 14, 30, 90, 365) |
| `due_date` | date | Date of next reckoning |
| `last_completed_at` | timestamptz | Null if never completed |
| `flagged` | text | Set to `'possible duplicate'` by the weekly cleanup job |
| `created_at` | timestamptz | Auto |

### Row-level security

RLS should be disabled for this app (no auth):

```sql
ALTER TABLE tasks DISABLE ROW LEVEL SECURITY;
```

### Weekly duplicate cleanup

Run once in the Supabase SQL editor to set up automatic duplicate detection:

```sql
-- Detection function
CREATE OR REPLACE FUNCTION flag_duplicate_tasks()
RETURNS void AS $$
BEGIN
  UPDATE tasks SET flagged = NULL
  WHERE flagged = 'possible duplicate'
  AND lower(trim(title)) NOT IN (
    SELECT lower(trim(title)) FROM tasks
    GROUP BY lower(trim(title)) HAVING count(*) > 1
  );

  UPDATE tasks SET flagged = 'possible duplicate'
  WHERE id IN (
    SELECT id FROM (
      SELECT id,
        ROW_NUMBER() OVER (
          PARTITION BY lower(trim(title)) ORDER BY created_at ASC
        ) AS rn
      FROM tasks
      WHERE lower(trim(title)) IN (
        SELECT lower(trim(title)) FROM tasks
        GROUP BY lower(trim(title)) HAVING count(*) > 1
      )
    ) sub WHERE rn > 1
  );
END;
$$ LANGUAGE plpgsql;

-- Schedule weekly (Sundays at 2am)
SELECT cron.schedule(
  'flag-duplicate-tasks',
  '0 2 * * 0',
  'SELECT flag_duplicate_tasks()'
);
```

Flagged tasks show a **⚑ Review** badge in the app. Tap the card to dismiss the flag or delete the duplicate.

---

## How it works

### Task lifecycle

1. **Active** — surfaced in the main list, sorted by urgency (most overdue first)
2. **Filed** — completed today, or next due date is more than 21 days away; shown below the divider at 50% opacity
3. **Resurfaced** — the day after completion, if the next due date is within 21 days, the card returns to the active list

### Urgency heat system

Cards are colour-coded by how overdue or upcoming they are:

| Class | When | Colour |
|---|---|---|
| `heat-5` | 7+ days overdue | Crimson |
| `heat-4` | 3–6 days overdue | Burnt orange |
| `heat-3` | 1–2 days overdue | Amber |
| `heat-2` | Due today | Yellow-gold |
| `heat-1` | 1–3 days away | Forest green |
| `cool-1` | 4–10 days away | Teal |
| `cool-2` | 11–21 days away | Slate blue |

### Interactions

| Action | How |
|---|---|
| Complete a task | Tap the card → FC-1 confirmation → File with the Ministry |
| Undo a completion | Tap a filed card → PCA-1 sheet → Whoops — Rescind the Filing |
| Add a task | Tap the + FAB → Form ST-7 |
| Edit a task | Tap any card → Amend Details → Form AM-3 |
| Delete a task | Tap any card → Strike from Record → Form DX-1 |
| Filter by department | Tap a tag pill |
| Clear a duplicate flag | Tap a ⚑ Review card → Not a duplicate — clear review flag |

### Form references

| Form | Purpose |
|---|---|
| ST-7 | Standard Task — new duty |
| AM-3 | Amendment — edit existing duty |
| DX-1 | Expungement — delete duty |
| FC-1 | Field Completion Notice — confirm completion |
| PCA-1 | Premature Completion Amendment — re-file or undo a filed task |

---

## Design

Full design artefacts are in `.design/silly-tasks-app/`:

- `DESIGN_BRIEF.md` — problem, solution, aesthetic direction, component inventory
- `INFORMATION_ARCHITECTURE.md` — view map, user flows, naming conventions
- `DESIGN_TOKENS.css` — full CSS custom property system (colours, type, spacing, motion)
- `TASKS.md` — original build task list

**Philosophy**: Institutional Paper — a lovingly recreated British government document aesthetic. Cream parchment, Playfair Display + Lora, heat-coded urgency stamps, and the quiet authority of a department that has been underfunded since 1987.

---

## Known issues / backlog

- Swipe-to-edit and swipe-to-delete are temporarily removed pending a reliable pointer event solution across desktop and mobile
