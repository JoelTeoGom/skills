---
name: inline-endpoints
description: Endpoints and SQL queries are written inline where they are used, never as constants. Use whenever writing or editing HTTP routes, URLs, path templates, HTTP clients or SQL queries, in any service, even when constants are not mentioned.
---

# Inline endpoints

Joel finds endpoints and queries declared as constants ugly. Reading a route or a query at its call site shows what the code does without jumping to another declaration.

## Rule

Never declare these as constants or package-level variables:

- HTTP routes and path templates
- URLs and paths used by clients or tests
- SQL and CQL queries

Write the literal where it is used, even when the same value appears several times.

## Examples

Avoid:

```go
const createTaskRoute = "POST /tasks"

mux.Handle(createTaskRoute, handler)
```

```go
const selectTaskQuery = `SELECT id, name FROM tasks WHERE id = ?`

row := r.db.QueryRowContext(ctx, selectTaskQuery, taskID[:])
```

Good:

```go
mux.Handle("POST /tasks", handler)
```

```go
row := r.db.QueryRowContext(ctx, `
	SELECT id, name
	FROM tasks
	WHERE id = ?`,
	taskID[:])
```

## Column lists

A long column list shared by several queries goes in a function next to the code that scans the row, not in a constant. It keeps the list out of the top of the repository, and the scan and the list stay together.

```go
func jobColumns() string {
	return `id, tenant_id, task_id, status`
}
```

## Not covered

Constants are still fine for values that are not endpoints or queries, such as timeouts, limits, default ports and error numbers.
