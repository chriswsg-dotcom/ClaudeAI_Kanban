---
description: Security-scan, then push code, README, repo About section and GitHub Pages workflow to a GitHub repo you specify
argument-hint: <github-repo> [branch] [-- change request]
allowed-tools: Bash(git:*), Bash(gh:*), Bash(grep:*), Bash(ls:*), Bash(cat:*), Read, Edit, Write
---

# Publish to GitHub

Arguments: `$ARGUMENTS`

Work through the steps in order. **Step 3 (security scan) is a gate: nothing is pushed or published until it passes.**

## 1. Work out the target

Parse `$ARGUMENTS`:

- **Repo** (required): `owner/repo`, `https://github.com/owner/repo(.git)` or `git@github.com:owner/repo.git`. Normalise to `owner/repo` and `https://github.com/owner/repo.git`.
- **Branch** (optional): defaults to the current branch (`git branch --show-current`).
- **Change request** (optional): anything after `--` describes code edits to make.

If no repo was given, show `git remote -v` and ask the user which repo to use. Don't guess.

Then run these in parallel: `git status --short`, `git remote -v`, `git log --oneline -5`, `gh auth status`, `gh repo view owner/repo --json name,description,homepageUrl,repositoryTopics,visibility`.

- If `gh` isn't logged in, tell the user to run `gh auth login` themselves and stop. Never ask for, or type, tokens or passwords.
- If the repo doesn't exist, ask whether to create it (`gh repo create owner/repo --public|--private --source . --remote <name>`), and which visibility. GitHub Pages on a free account needs a **public** repo.

## 2. Make the edits

### 2a. Code (only if a change request was given)

- Read the files before you touch them, and follow every rule in `CLAUDE.md`: vanilla HTML/CSS/JS only, no persistence, no `alert()` or `confirm()`, `escapeHtml()` on user strings, re-render from `state`, and so on.
- Keep the change focused on the request.

### 2b. README.md (create it, or update it)

Read `index.html`, `CLAUDE.md` and any existing `README.md`. Write or refresh `README.md` so it covers:

- the title and a one-line summary,
- a live demo link: `https://<owner>.github.io/<repo>/`,
- features (columns, filters, add/move/delete, drag-and-drop, overdue flags, summary),
- how to run it locally (open `index.html`, or `python3 -m http.server` to test FormSubmit),
- FormSubmit setup (`FORMSUBMIT_ENDPOINT`, one-time activation email),
- tech notes: vanilla HTML/CSS/JS, single file, no persistence on purpose,
- deployment: GitHub Pages through `.github/workflows/pages.yml`,
- a disclaimer that this is an internal demo, not an official UOB system and not affiliated with UOB.

When updating, keep the sections that are still accurate and fix the ones that are stale. Don't add badges or images that point to outside hosts.

### 2c. GitHub Pages workflow

Make sure `.github/workflows/pages.yml` exists and:

- triggers on `push` to the target branch, plus `workflow_dispatch`,
- has `permissions: contents: read, pages: write, id-token: write` and a `concurrency: pages` group,
- copies **only the public site files** (`index.html`) into `_site/`. Never publish `CLAUDE.md`, `.claude/` or other repo internals,
- uses `actions/checkout@v4`, `actions/configure-pages@v5`, `actions/upload-pages-artifact@v3` and `actions/deploy-pages@v4`.

Create it if it's missing. Fix it if it's out of date or its branch doesn't match the target branch.

## 3. Security scan (gate)

Scan everything that will be pushed: `git diff --cached`, the unstaged diff, untracked files that will be added, and on a first push to a new repo, the whole tree (`git ls-files`). Check for:

1. **Secrets:** API keys, tokens, passwords, private keys, `.env` files, cloud credentials, and patterns such as `AKIA`, `ghp_`, `github_pat_`, `sk-`, `xox[bp]-`, `-----BEGIN .*PRIVATE KEY-----`, `password\s*[:=]`, `secret\s*[:=]` and `token\s*[:=]`. Use `grep -rnE` over the files you're pushing. If `gitleaks` or `trufflehog` is installed, run it as well.
2. **Personal data:** real email addresses (other than the `YOUR_EMAIL@example.com` placeholder), phone numbers, staff names or IDs, internal hostnames, IPs, and internal URLs.
   - If `FORMSUBMIT_ENDPOINT` holds a real address, warn that it will be public in the repo and on the Pages site, and ask whether to keep it or restore the placeholder.
3. **Code risks in `index.html`:** user data reaching `innerHTML` or template HTML without `escapeHtml()`; `eval`, `new Function` or string `setTimeout`; `alert()` or `confirm()`; localStorage, sessionStorage, IndexedDB or cookies; external `<script>` or `<link>` or CDN references; `fetch` to anything other than FormSubmit; `target="_blank"` without `rel="noopener"`.
4. **Branding:** no real UOB logo, trademark assets or anything that imitates an official UOB system.
5. **Workflow safety:** least-privilege `permissions`; no `pull_request_target`; no untrusted `${{ github.event.* }}` values interpolated into `run:`; actions pinned to a major version or SHA.
6. **Repo hygiene:** no large binaries, and no OS/editor junk (`.DS_Store`, `*.swp`). Offer to add a `.gitignore` if it's missing.

Report the findings in a short table: severity, file:line, issue, fix.

- **High (secrets, personal data, XSS):** stop. Fix the issue, or ask the user, and re-scan. Don't push until it's clean, or until the user explicitly accepts the specific risk in this session.
- **Medium/Low:** list them, fix the easy ones, and ask whether to continue.

## 4. Commit

- Show `git diff --stat` and a short summary of the changes.
- Stage only the intended paths (`git add <paths>`), never `git add -A` blindly. Never stage `.env`, credentials or `.claude/settings.local.json`.
- Use one commit per logical change, or a single commit if the changes are small. Write short imperative messages and end each one with:

  ```
  Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>
  ```

## 5. Push

- Remote: use an existing remote that points at the target repo. If `origin` points somewhere else, **ask** before changing it; the alternative is adding a new remote named after the owner. If there's no remote, add it as `origin`.
- Run `git fetch <remote>`. If the remote branch has commits you don't have, stop and offer `git pull --rebase`. **Never force-push** unless the user explicitly asks in this session.
- Push with `git push -u <remote> <branch>`. Pushing to the Pages branch triggers a deploy, so tell the user that before you push.

## 6. About section

Update the repo's About panel with `gh repo edit owner/repo`:

- `--description`: one line, no more than about 120 characters, for example "Single-file Kanban board demo for an IT PMO — vanilla HTML/CSS/JS, deployed on GitHub Pages."
- `--homepage`: `https://<owner>.github.io/<repo>/`
- `--add-topic`: for example `kanban`, `project-management`, `pmo`, `vanilla-js`, `html`, `css`, `github-pages`, `demo`

Show the old and new values before applying them. This is a public change, so confirm it with the user.

## 7. Enable Pages and verify

- Make sure Pages uses GitHub Actions as its source:
  - check it with `gh api repos/owner/repo/pages`,
  - if you get a 404, create it with `gh api -X POST repos/owner/repo/pages -f build_type=workflow`,
  - if it exists with a different `build_type`, switch it with `gh api -X PUT repos/owner/repo/pages -f build_type=workflow`.
- Watch the deploy: `gh run list --workflow pages.yml --limit 1`, then `gh run watch <id> --exit-status`.
- If the run fails, show the failing step's log (`gh run view <id> --log-failed`) and propose a fix.

## 8. Report

Finish with:

- a security scan summary (clean, or what was fixed or accepted),
- the commit hashes and messages, and the remote and branch pushed to,
- the README status (created or updated),
- the About section values that were applied,
- the Pages run result and the live URL `https://<owner>.github.io/<repo>/`,
- links: `https://github.com/owner/repo` and `https://github.com/owner/repo/actions`.
