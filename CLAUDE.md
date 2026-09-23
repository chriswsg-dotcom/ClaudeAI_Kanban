# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-file Kanban board for an IT PMO, built as an internal demo/training tool. Everything lives in `index.html`: markup, one `<style>` block and one `<script>` block. There is no build, no package manager, no linter and no test suite.

Run it by opening `index.html` directly in a browser (`open index.html`). A server is only needed to test FormSubmit delivery, which may reject `file://` origins (`python3 -m http.server`, then load `http://localhost:8000`).

## Hard constraints (from the original brief — keep them)

- Vanilla HTML/CSS/JS only: no frameworks, libraries, CDNs, web fonts, image files or build step. Use the system font stack and inline SVG or Unicode glyphs for icons.
- **No persistence.** The board state lives in memory only. Do not use localStorage, sessionStorage, IndexedDB or cookies. A refresh reloads the seeded demo tasks on purpose, and the page shows a note saying so.
- The only backend is FormSubmit's AJAX JSON endpoint, called with `fetch`. Never use a plain form POST that navigates away.
- Branding: a neutral "UOB IT PMO" text wordmark and a corporate blue palette. No real UOB logo or trademarks, and nothing that imitates an official UOB system.
- Never use native `alert()` or `confirm()`. Validation errors appear inline under each field, and deleting a card uses an inline "Delete? Yes / No" toggle.
- CSS custom properties for the palette and spacing, and no `!important`.

## Script architecture

- **One source of truth:** `state = { tasks, filters, nextId, ui }`. `filters` holds `search`, `project`, `assignee`, `priority` and `sort`. The `ui` part holds per-card transient state (`moveMenuId`, `confirmDeleteId`, and an `expanded` Set of cards showing details), a pending `focus` target and `submitting`.
- **Always re-render from state.** Data operations (`addTask`, `moveTask`, `deleteTask`) change `state`, then call `renderBoard()`. It rebuilds every column's HTML from `applyFilters(state.tasks)` via `renderColumn()` and `renderCard()`, then calls `renderSummary()` and `restoreFocus()`. Don't change card DOM anywhere else.
- **Focus after re-render:** handlers set `state.ui.focus = { action, id }`. `restoreFocus()` then focuses the matching `[data-action][data-id]` element, which keeps keyboard users in place after a re-render.
- **Board events use delegation.** One click handler on `#board` switches on `data-action` (`details-toggle`, `move-toggle`, `move-to`, `move-cancel`, `delete`, `delete-yes`, `delete-no`). Escape closes any open inline menu. Native HTML5 drag-and-drop listeners are also on `#board`, and dropping a card calls `moveTask`.
- **Escaping:** every user-supplied string goes through `escapeHtml()` before it is placed in template-literal HTML, including values inside attributes.
- **Dates** are local `YYYY-MM-DD` strings, compared as text against `todayIso()`. Seed tasks use `daysFromToday(n)`, so overdue examples stay overdue whatever day it is. A task is overdue when its due date is before today and its status isn't Done.
- **Task IDs** look like `UOB-ITPM-####` and come from `generateId()` (`state.nextId`). The 8 seed tasks use 0001–0008.
- **The Add Task flow is optimistic.** `handleSubmit` validates the form (`validateTask` → `showFieldErrors`), adds the card, resets the form and shows a success toast. It then awaits `notifyNewTask()` while the submit button shows "Sending…". If that fails, only a warning toast appears ("Card added locally — email notification failed"). A FormSubmit failure must never break the board.
- **Filtering and sorting** both happen in `applyFilters()`. Search matches ID, title and description. Sort is by date added (array order), due date, or priority then due date. `exportCsv()` downloads the same filtered and sorted list, with formula-injection guarding in `csvCell()` and a UTF-8 BOM for Excel.
- **Layout:** a sticky `.topbar` holds the header, demo note and filter bar. `trackTopbarHeight()` writes its height to `--topbar-h`, which each column's `.card-list` uses to scroll on its own at 768px and wider. Below 768px the top bar isn't sticky and columns stack. Cards are compact (ID, priority, overdue, title, assignee, due date), and project, category, status and description show under "Details ▾".
- **Add Task is a native `<dialog>`** opened by `openTaskDialog()`. It closes on submit, Cancel, ×, Esc or a backdrop click. Closing keeps the typed draft but clears error messages.
- **Reference lists** (`STATUSES`, `PROJECTS`, `CATEGORIES`, `PRIORITIES`) sit at the top of the script. The form selects, filter selects and columns are all generated from these lists.

## FormSubmit config

`FORMSUBMIT_ENDPOINT` is the one place to set the notification email address, at the top of the script. While it still holds the `YOUR_EMAIL@example.com` placeholder, `notifyNewTask` throws before making any request. A new address needs one-time activation: the first submission sends a confirmation email, and nothing is delivered until its link is clicked. Unactivated addresses can return `success: "false"`, which the code treats as a failure. A 15 s `AbortController` timeout keeps the button from staying on "Sending…" forever.

## Published artifact

A copy is published at https://claude.ai/artifact/59ZJ43x4QRgNieS9c6NpYu. The copy has no document wrapper tags, a shorter `<title>` and extra dark-theme tokens. It is not generated from `index.html`, so rebuild and republish it after changes you want shared. FormSubmit calls are blocked inside the artifact sandbox, so the warning toast always appears there.
