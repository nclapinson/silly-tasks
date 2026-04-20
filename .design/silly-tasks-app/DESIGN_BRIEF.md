lets continue# Design Brief: The Ministry of Silly Tasks

## Problem

Managing recurring household chores as a couple is a low-stakes but genuinely irritating problem. Nothing is urgent enough to warrant a full project management tool, but things quietly pile up — the bin that hasn't been taken out, the filter that hasn't been changed, the shower that's overdue. There's no shared memory of when things were last done, and no gentle system to surface what needs attention without nagging.

## Solution

A lightweight, single-page web app that lives on both partners' phones and acts as a shared, opinionated chore clerk. It surfaces what's overdue and upcoming in a single glance, lets either person mark a task done in one tap, and allows new duties to be registered or amended with minimal friction. The tone is bureaucratic and comedic — it takes the tedium of chores and reframes them as official Ministry business, which makes the whole thing more bearable.

## Experience Principles

1. **Glanceability over granularity** — The most important information (what's overdue, what's due today) must be readable in under three seconds. Stats and heat colours do this job. No drill-downs required for the default use case.

2. **Character over chrome** — Every UI surface is an opportunity to reinforce the Ministry voice. Form labels, button copy, confirmation dialogs, and empty states should all feel like they were written by a reluctant civil servant. Neutral, generic UI language is a failure state.

3. **Confidence over reversibility** — Actions should feel decisive. Completing a task, deleting a duty, filing an amendment — these are bureaucratic acts. Confirmations exist for destructive actions only. We do not second-guess the user at every step.

## Aesthetic Direction

- **Philosophy**: Institutional Paper — a lovingly recreated British government document aesthetic. Cream parchment backgrounds, serif typography (Playfair Display for display, Lora for body), subtle paper grain, heat-coded urgency stamps, and a general sense that this app was produced by a department that has been underfunded since 1987.
- **Tone**: Deadpan bureaucratic humour. Warm but unhurried. Authoritative but absurd.
- **Reference points**: GOV.UK design system (structural clarity), aged British institutional print design (tactile warmth), Monty Python's Ministry of Silly Walks (the joke).
- **Anti-references**: Sterile productivity apps (Jira, Linear), bubbly pastel to-do apps (Todoist's marketing, Any.do), generic Material Design. Nothing that looks like it was designed for a SaaS dashboard.

## Existing Patterns

- **Typography**: Playfair Display (900/700/400, display use) + Lora (600/400 + italics, body/meta). Both loaded from Google Fonts.
- **Colors**: CSS custom properties on `:root` — `--bg` (#f0e6d0 cream), `--card` (#faf5ea off-white), `--border` (#c8b48a tan), `--text` (#2a1e0c deep brown), `--muted` (#8a7858), `--faint` (#b8a888), `--accent` (#7a3018 burnt sienna), `--shadow`. Heat classes (heat-5 through cool-2) define per-card urgency colour via `--card-accent`, `--card-text`, `--card-shadow`.
- **Spacing**: Informal scale using rem. Cards: 0.9rem/1.15rem padding. Page inner: max-width 760px, 1.25rem horizontal padding.
- **Components**: Task card (with heat class, stinky stamp, tilt, grain texture), stats bar, tag filter pills, section divider, loading screen with poster image, FAB, bottom sheets (task form + delete confirm), swipe-reveal action buttons.

## Component Inventory

| Component | Status | Notes |
|---|---|---|
| Task card | Exists | Well-styled; swipe interaction rebuilt in current session |
| Card swipe actions (edit/delete reveal) | New (just built) | Directional swipe — left = edit, right = delete |
| Stats bar | Exists | No changes needed |
| Tag filter pills | Exists | No changes needed |
| FAB (+ button) | New (just built) | Wax-seal radial gradient style |
| Task sheet (add/edit) | New (just built) | Bottom sheet, Form ST-7 / AM-3, Ministry themed |
| Delete confirm sheet | New (just built) | Dark crimson sheet, Form DX-1 |
| Loading screen | Exists | Poster image + staggered text animation |
| Section divider | Exists | "Filed with the Ministry" rule |
| Empty state | Missing | Needed when no tasks exist or filter returns nothing |
| Toast / feedback | Missing | Success feedback after create/edit/delete |

## Key Interactions

**Complete a task** — Tap card body → task marked done, rescheduled by interval, card transitions to `just-done` state (strikethrough, muted, 50% opacity) and drops below the divider.

**Swipe to edit** — Swipe card left past threshold (~70px) → card snaps open, brown "Amend Record" button revealed on right. Tap button → card snaps closed, edit sheet slides up. Tap card or swipe back → snaps closed.

**Swipe to delete** — Swipe card right past threshold → card snaps open, crimson "Strike from Record" button revealed on left. Tap button → dark delete confirmation sheet slides up. Confirm → task removed, card exits. Cancel → sheet closes, card snaps back.

**Add a task** — Tap FAB → "Lodge a New Duty" sheet slides up. Fill title (required), department tag (optional), recurrence interval, due date. Submit → task created, sheet closes, card enters list with animation.

**Filter by tag** — Tap tag pill → list and stats refresh to show only that department's tasks. Active pill styled with accent fill.

**Empty state** — When no tasks exist (or filter returns nothing), show in-character message rather than blank space.

## Responsive Behavior

- **Mobile (< 600px)**: Single column. FAB bottom-right. Sheets full-width. Swipe targets are touch-friendly (min 44px height). Stats bar wraps to 2×2 grid if needed.
- **Tablet / Desktop (≥ 760px)**: Page inner max-width 760px, centred. FAB respects the max-width constraint (currently positioned using `calc(50vw - 380px + 1.25rem)`). Sheets also max-width 760px centred. Hover states on cards and buttons.
- Cards do not go multi-column — single column at all breakpoints to preserve the swipe interaction.

## Accessibility Requirements

- Best-effort. No keyboard-only navigation requirement.
- Colour is not the sole indicator of urgency — heat badges (text labels) accompany heat colours.
- Minimum contrast ratio: AA for body text against card backgrounds.
- Sheet dialogs use `role="dialog"` and `aria-modal="true"` (already in place).
- FAB has `aria-label="Lodge a new duty"`.
- Tap targets minimum 44×44px on mobile for interactive elements.

## Out of Scope

- Dark mode (planned for a future iteration)
- Multi-user auth / per-user task lists (single shared Supabase table)
- Notifications or push reminders
- Task history / activity log
- Custom recurrence intervals (non-standard day counts)
- Offline support / service worker
- Search / text filter
- Sorting options beyond due-date order
- Data export / import
- Print stylesheet
