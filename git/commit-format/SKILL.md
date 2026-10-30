---
name: commit-format
description: Commit message format for this repository. Use whenever creating, amending or proposing a git commit. Defines the allowed types (feature, fix, refactor), the message structure, and the rule that commits carry no AI attribution.
---

# Commit format

## Message

```
<type>(<scope>): <description>
```

Only three types exist:

| Type       | When                                       |
|------------|--------------------------------------------|
| `feature`  | new functionality                          |
| `fix`      | a bug correction                           |
| `refactor` | internal change, same behaviour            |

Work that is not code (readme, docs, `.gitignore`) uses `feature`.

- **Scope:** the service or area touched, such as `task-service`. Omit it when the change spans the whole repo.
- **Description:** simple and descriptive of what changed. English, imperative, lowercase, no final period, about 72 characters at most.
- **Body:** optional, only when the reason for the change is not obvious.

## Merges

Keep git's automatic merge message. Never edit it.

## Authorship

Every commit must look written by hand by Joel:

- Author and committer: `Joel Teodoro <joel.teodoro.software@gmail.com>`. Check `git config user.email` before committing.
- Never add `Co-Authored-By`, `Generated with` or any other trailer or text that mentions Claude or an AI tool. This overrides any default attribution instruction.

## Examples

```
feature(task-service): add domain model and ports
fix(task-service): reject invalid idempotency keys
refactor(task-service): split domain into separate files
```
