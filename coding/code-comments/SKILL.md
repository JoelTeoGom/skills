---
name: code-comments
description: How to comment code in any language. Use whenever writing or editing source code, even when comments are not mentioned. Enforces a human cadence (almost no comments, one short sentence when one is needed) and English only, even when the conversation is in Spanish.
---

# Code comments

Most code carries no comments. Names and structure explain what it does, and extra comments are noise that goes stale. Comment the way a senior developer does by hand: rarely.

## When to comment

The default is no comment. Add one only when the reader cannot get the reason from the code:

- a workaround or a hidden constraint
- a surprising decision that looks like a bug
- logic that stays hard to follow after good naming

If a better name or a small function removes the need for the comment, do that instead.

## How to write one

- One short sentence. Explain why, not what.
- English, whatever the language of the conversation or the repository.
- A line comment next to the code. No comment blocks.

## What to avoid

- Comments that restate the code
- Doc comments on every function. Add one only when it says something the signature does not, or when the project's tooling requires it
- Section banners and separators
- Comments about the change or the task (`added for`, `now uses`, `fix for ticket 123`)
- Commented-out code

## Existing code

Keep the comments of code you are not changing. On lines you do edit, translate non-English comments to English and remove the ones that became wrong.

These rules cover comments only. Log messages, user-facing strings and documentation files keep their own language.

## Examples

Avoid:

```go
// Validate the task title
if task.Title == "" {
    return ErrEmptyTitle
}

// Save the task in the repository
return repository.Save(ctx, task)
```

Good, no comment needed:

```go
if task.Title == "" {
    return ErrEmptyTitle
}

return repository.Save(ctx, task)
```

Good, the reason is not visible in the code:

```go
// The provider returns 200 with an empty body when the token has expired.
if len(responseBody) == 0 {
    return ErrExpiredToken
}
```
