---
description: Remote ops console for rockvaultar-site (status, preview, deploy checks)
argument-hint: "[status|preview|check|deploy-status|live|logs] (default: status)"
allowed-tools: Bash(git:*), Bash(curl:*), Bash(python3:*), Bash(ls:*), Bash(find:*), Bash(grep:*), Bash(npx:*), Bash(gh:*)
---

You are the remote-control operator for the `rockvaultar-site` static site (GitHub Pages-style repo at `rodelrepulle/rockvaultar-site`).

The user invoked `/remote-control` with arguments: `$ARGUMENTS`

If no argument was provided, treat the action as `status`.

## Actions

### `status` (default)
Run these in parallel and report a compact dashboard:
- `git status --short` and current branch
- `git log --oneline -5`
- `git rev-list --count HEAD ^origin/main 2>/dev/null` (commits ahead of main, if applicable)
- `ls -la *.html privacy/ support/ terms/ 2>/dev/null` (verify expected pages exist)
- Use the GitHub MCP tools (`mcp__github__list_pull_requests`, `mcp__github__list_commits`) to check for open PRs and the latest pushed commit on the default branch.

Report as a short bulleted summary. No headers, no fluff.

### `preview`
Start a local preview server in the background:
- `python3 -m http.server 8080` via Bash with `run_in_background: true`
- Report the URL `http://localhost:8080` and remind the user to stop it when done.

### `check`
Static-site sanity check — run in parallel:
- Grep all `index.html` files for broken-looking references: `grep -nE 'href="(#|javascript:|TODO|FIXME)"' **/*.html index.html`
- Verify each HTML file has `<html>`, `<head>`, `<title>`, `<body>` tags
- Report any missing tags or suspicious links. Silence = clean.

### `live`
Check whether the live site is reachable. The user has not told you the live URL yet — first try `curl -sSI https://rockvaultar.com` and `curl -sSI https://rodelrepulle.github.io/rockvaultar-site/` in parallel. Report HTTP status and `last-modified` header for whichever responds. If neither works, ask the user for the live URL.

### `deploy-status`
Use the GitHub MCP tools to check deployment health:
- `mcp__github__list_commits` on the default branch — show the latest commit SHA and message.
- If GitHub Pages is wired up, the deploy is tied to commits on the default branch. Report the latest commit and let the user know that a push to the default branch triggers redeploy.

### `logs`
Show recent activity:
- `git log --oneline --all -20`
- Use `mcp__github__list_pull_requests` (state: all, sort: updated, perPage: 5) for recent PR activity.

## Rules
- Run independent checks in parallel for speed.
- Keep the final report tight — bullets, not paragraphs.
- Never push, force-push, or modify files. This command is read-only ops; if the user wants changes, they will ask separately.
- If an action name doesn't match the list above, list the available actions and stop.
