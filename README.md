# agent-docs-sync

A Claude skill that keeps your project's docs true to the code. After Claude changes your code, it checks `AGENTS.md`, `CLAUDE.md`, `README.md`, `docs/` and similar files, fixes anything the change made out of date, and tells you what it updated.

Agent docs only help if they're right. An `AGENTS.md` that still lists a deleted file or an old command sends the next agent the wrong way. This skill catches that the moment it happens.

## What it does

- **Runs automatically** at the end of any task where Claude edited code, config, scripts or dependencies. You don't have to ask.
- **Reads the real diff** (`git diff`, including new files), not just what Claude remembers changing.
- **Finds every claim the change touched**, including negative ones like "there is no `typecheck` script", and fixes every copy (folder trees, tables, gotchas).
- **Follows your project's own rules first.** If your `AGENTS.md` says "added a file → update Folder structure", it does exactly that.
- **Makes small, in-style edits.** It deletes statements that are no longer true instead of piling on notes, and it leaves tool-managed blocks (`<!-- BEGIN:… -->`) alone.
- **Checks its work:** no old names left behind, no broken links.
- **Leaves docs alone when nothing documented changed.** An internal refactor gets "no change needed" and the reason, not churn.
- **Always reports**, in one line, which docs and sections it updated, or why none needed changing.

## Example

You ask Claude to add a `typecheck` script. Your `AGENTS.md` said:

```
There is no `typecheck` script. Use `bunx tsc --noEmit`.
```

Claude adds the script, removes that line, adds `bun run typecheck` to the Commands list, and ends with:

> Docs: updated `AGENTS.md` Commands (added `bun run typecheck`, removed the "no typecheck script" note).

## Usage

- **Automatic:** just work as usual. Claude runs it after code changes.
- **On demand:** `/agent-docs-sync`, or ask "sync the docs".
- **For committed work:** `/agent-docs-sync HEAD~3..HEAD` or `/agent-docs-sync main..` catches the docs up for a range of commits.

For very large changes (about 50+ files), it summarizes the changes and asks you to confirm before editing. If it's unclear which section a change belongs in, it asks you instead of guessing.

## Install

Put the `agent-docs-sync` folder in `~/.claude/skills/` (all your projects) or in `<project>/.claude/skills/` (one project). Works with Claude Code and any tool that supports Agent Skills.

## Requirements

A git repository, and at least one doc file (`AGENTS.md`, `CLAUDE.md`, `README.md`, …). If a project has no docs, the skill does nothing; it never creates docs you didn't ask for.

---

Made by Vivek Dahiya.
