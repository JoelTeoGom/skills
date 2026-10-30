---
name: gitignore
description: Keeps Claude Code files (.claude/, CLAUDE.md, CLAUDE.local.md) out of every git repository. Use whenever creating or initialising a repository, creating or editing a .gitignore, or staging, committing or pushing files, even when ignoring files is not mentioned.
---

# Gitignore

Claude Code files are local tooling. They clutter a repository, so they are never committed.

## Required lines

The `.gitignore` at the root of every repository contains:

```gitignore
.claude/
CLAUDE.md
CLAUDE.local.md
```

Create the file if it does not exist. Add only the missing lines and leave the rest untouched.

Keep these exact names, which also match in subfolders. A wildcard like `*claude*` would hide real source files such as `claude_client.go`. This rule covers these files only, not files that merely mention Claude in their content.

## Before committing

`.gitignore` has no effect on files git already tracks, so remove any that slipped in:

```
git rm -r --cached --ignore-unmatch -- ':(glob)**/.claude/**' ':(glob)**/CLAUDE.md' ':(glob)**/CLAUDE.local.md'
```

`--cached` removes them from git only. The files stay on disk.

Never use `git add -f` on these files.

## Already pushed

Removing a file only stops tracking it from that commit on. It stays in the history of the remote. Tell Joel when this happens, and do not rewrite history without asking.

## Commit message

Follow `commit-format`. The message does not mention Claude, so use something like:

```
feature: update gitignore
```
