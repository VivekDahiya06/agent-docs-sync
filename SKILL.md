---
name: agent-docs-sync
description: Keep a project's agent and contributor docs (AGENTS.md, CLAUDE.md, README.md, docs/, CONTRIBUTING.md, .cursorrules, copilot-instructions.md) in sync with the code. Use this at the end of ANY task that edited, added, deleted or renamed code, config, scripts or dependencies, before telling the user the task is done, even if they didn't mention docs and even for small fixes. Also use it when the user asks to "sync the docs", "update AGENTS.md", check whether the docs still match the code, or catch the docs up for recent commits (accepts a commit range like HEAD~3..HEAD).
---

# Agent docs sync

Agent docs like `AGENTS.md` are only useful if they're true. The next agent (or person) trusts them instead of reading the code, so a doc that still names a deleted file or an old command sends them the wrong way. This skill closes that gap at the moment it opens: right after a code change, while you still know exactly what changed.

Run it as the last step of a task that touched the codebase, before your final message.

## 1. Find the docs

Look in the repo root (and `docs/`) for: `AGENTS.md`, `CLAUDE.md`, `README.md`, `CONTRIBUTING.md`, `.cursorrules`, `.github/copilot-instructions.md`, and any `*.md` they link or `@import`. Nested `AGENTS.md`/`CLAUDE.md` files in subfolders cover that subfolder.

If a doc states its own update rules (e.g. "Rule 0: added a file → update Folder structure"), follow them; they're the project owner's mapping and beat the generic one below.

If the project has no such docs, skip this skill. Don't create docs nobody asked for.

## 2. List what changed

Use `git status --porcelain` and `git diff` (plus `git diff --cached`) to see the real change, including untracked files. Don't rely only on memory of what you edited; the diff also catches files changed by tools, generators or package managers.

If the user gave a commit or range (e.g. `/agent-docs-sync HEAD~3..HEAD` or `/agent-docs-sync main..`), diff that instead (`git diff --name-status <range>`, then `git diff <range>`), so docs can be caught up for work that's already committed. With no argument and a clean working tree, report that there's nothing to sync and mention the range option.

If the diff is large (roughly 50+ files), group the changes by kind (new files, renames, config, deps…), show the user that summary and confirm the scope before editing, so a big refactor doesn't turn into a sprawling doc rewrite.

From the diff, pull out the **doc-level facts** that changed. These are the things docs typically state:

| Change in code | Where docs usually mention it |
|---|---|
| File/folder added, removed, renamed, moved | folder tree, "where things live", file references |
| Export, function, component, constant renamed or its signature changed | architecture, data flow, recipes, examples |
| Config field / constant / env var added or renamed | config/content sections, setup steps |
| Script, command, dependency, tool, version | tech stack, commands, install steps |
| Route, id, URL, port | page/route tables, links |
| Behavior or convention changed, bug/gotcha fixed | conventions, gotchas, "known issues" |

Pure internal changes (logic inside a function, styling tweaks, refactors that keep names and behavior) usually change no doc-level fact. That's a valid outcome.

## 3. Find every doc claim the change touches

For each changed fact, search all the docs for it, not just the section you expect:

- grep for the **old** name/path/command (these are the claims that are now false), and for the new one (to see whether it's already documented);
- check folder trees and tables line by line for the affected folder;
- look for negative claims the change made false ("there is no `typecheck` script", "nothing links to `#agents`", "no tests").

The same fact is often written in several places (a tree, a table and a gotcha). Fix all copies.

If a change genuinely could belong in more than one place and the doc gives no rule for it (e.g. a new convention that could go under Conventions or Gotchas), pick the best fit when it's clear; when it isn't, list the options and ask the user rather than guessing.

## 4. Edit the docs

- Make the smallest edit that makes the doc true again, in the doc's existing style (table, tree, terse bullet). Match its tone and detail level.
- Delete statements that are no longer true rather than adding "update:" notes on top. Docs describe the code as it is now, not its history.
- Only write what you verified in the code. If you're unsure whether something is still true, check the code.
- If you notice a doc claim that was already wrong before your change, fix it too and mention it to the user.
- Leave tool-managed blocks alone (sections wrapped in markers like `<!-- BEGIN:... -->` / `<!-- END:... -->` or labeled as generated).
- Put each fact in the doc it belongs to: if one doc imports another (e.g. `CLAUDE.md` imports `AGENTS.md`), keep shared facts in the imported one and don't duplicate them.

## 5. Check your edits

Before reporting, do a quick pass over the docs you touched:

- grep the docs again for every old name/path/command from step 3; none should be left;
- check that relative links in the edited docs still point to files that exist, especially after a rename or move. For example:

  ```bash
  grep -oE '\]\([^)#:]+' AGENTS.md README.md | sed 's/.*](//' | sort -u | while read -r f; do [ -e "$f" ] || echo "broken link: $f"; done
  ```

  (paths are relative to each doc's folder; adjust for nested docs). Fix any broken link you caused; mention pre-existing ones.

## 6. Report

In your final message, say in one or two lines which docs and sections you updated, e.g. "Docs: updated `AGENTS.md` Folder structure and Commands." If nothing needed to change, say so and why ("Docs: no change needed, the edit was internal to `parseWords`."). The user should never have to wonder whether the docs were checked.
