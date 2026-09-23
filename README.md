# ClaudeAI_Kanban

A single-file Kanban board for an IT PMO, built as an internal demo and training tool.

**Live demo:** https://chriswsg-dotcom.github.io/ClaudeAI_Kanban/

## Features

- Four columns: **Backlog**, **In Progress**, **Blocked** and **Done**
- Add tasks with inline validation (no pop-up dialogs)
- Move cards with drag-and-drop or the keyboard-friendly **Move ▸** menu
- Delete cards with an inline **Delete? Yes / No** confirmation
- Filter by project and priority, with a live summary of task counts
- Overdue flags for tasks past their due date that aren't Done
- Optional email notification for new tasks through [FormSubmit](https://formsubmit.co)

## Run locally

No build step or dependencies are needed. Open the file in a browser:

```bash
open index.html
```

To test FormSubmit delivery (it may reject `file://` pages), serve the folder instead:

```bash
python3 -m http.server
```

Then browse to http://localhost:8000.

## Email notifications (optional)

Set the address in `FORMSUBMIT_ENDPOINT` at the top of the script in `index.html`:

```js
const FORMSUBMIT_ENDPOINT = "https://formsubmit.co/ajax/you@example.com";
```

The first submission sends an activation email, and nothing is delivered until you click its link. While the placeholder is in place, or if sending fails, the card is still added and a warning toast appears. Anything you put here is public once pushed, so use an address you're happy to publish.

## Tech notes

- Vanilla HTML, CSS and JavaScript in one `index.html` file: no frameworks, libraries, CDNs or web fonts.
- **No persistence, on purpose.** The board lives in memory, so refreshing the page reloads the demo tasks.
- All user input is HTML-escaped before rendering.

## Deployment

Every push to `main` deploys to GitHub Pages through [`.github/workflows/pages.yml`](.github/workflows/pages.yml). Only `index.html` is published.

## Disclaimer

This is an internal demo, not an official UOB system, and it is not affiliated with or endorsed by UOB. "UOB IT PMO" is a plain text wordmark, and the app uses no UOB logos or trademarks.
