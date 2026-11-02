---
name: go-sql-adapters
description: Structure of SQL repository adapters in Go for MySQL and PostgreSQL. Use whenever creating or editing an outbound adapter that reads or writes a relational database, such as adapters/out/mysql or adapters/out/postgres, including queries, row scanning, column lists and driver error handling.
---

# Go SQL adapters

One package per database technology, three kinds of files, plain SQL written where it runs. The column list lives in a function next to the scan, so the two lists that must match sit side by side.

## Files

```
adapters/out/mysql/            # or postgres/
├── db.go                      # Open: pool settings and ping
├── <entity>_repository.go     # struct, constructor and the port methods
└── <entity>_row.go            # rowScanner, scan<Entity>, <entity>Columns() and value helpers
```

### db.go

`Open(ctx, dsn)` creates the pool, sets max open and idle connections and connection lifetime, pings with a timeout and closes the pool if the ping fails. `main` calls it and owns the `Close`.

### <entity>_repository.go

- A struct holding the pool, its constructor, and a compile-time check against the port:

```go
type JobRepository struct {
	db *sql.DB
}

var _ ports.JobRepository = (*JobRepository)(nil)

func NewJobRepository(db *sql.DB) *JobRepository {
	return &JobRepository{db: db}
}
```

- One method per port method, each with its SQL inline (see the `inline-endpoints` skill). The select list comes from the columns function:

```go
row := r.db.QueryRowContext(ctx, `
	SELECT `+jobColumns()+`
	FROM jobs
	WHERE id = ? AND tenant_id = ?`,
	jobID[:], tenantID[:])

job, err := scanJob(row)
```

- Private helpers shared by several methods, such as `listJobs`, `save` or `translateSelectError`, go at the end of the same file.

### <entity>_row.go

Everything that converts between a row and the domain:

```go
type rowScanner interface {
	Scan(destinations ...any) error
}

func scanJob(scanner rowScanner, extraDestinations ...any) (*domain.Job, error) {
	// scan nullable columns into sql.Null*, then map them to the domain
}

func jobColumns() string {
	return `id, tenant_id, task_id, scheduled_at,
	status, attempt, next_attempt_at,
	created_at, updated_at`
}
```

- `rowScanner` accepts `*sql.Row` and `*sql.Rows` alike, and also `pgx.Row` and `pgx.Rows`, so one scan function serves single reads and lists.
- `extraDestinations` lets a query select a few columns after the list without a second scan function.
- The order of `<entity>Columns()` and of the scan destinations must match. Group both by concept on the same lines so a diff shows when one changes without the other.
- The write side lives here too: a function that turns the entity into insert or update values, and small helpers such as `nullableString`, `nullableTime` or `encodeHeaders`.

## Rules

- **Columns as a function, never a constant.** It returns a `string` that is concatenated into the query.
- **Nullable columns** are scanned into `sql.NullString`, `sql.NullTime`… and mapped to the domain zero value or to a pointer. The domain never sees `sql.Null*`.
- **Translate driver errors to domain errors in the adapter**: no rows becomes `domain.ErrNotFound`, a unique violation becomes `domain.ErrDuplicateKey`, and 0 rows affected on a conditional update becomes a conflict. Wrap everything else with the technology and the action: `fmt.Errorf("mysql: list jobs: %w", err)`.
- **Iterating rows**: `defer rows.Close()`, scan in the loop, and check `rows.Err()` after it.
- **Transactions**: `BeginTx`, `defer tx.Rollback()`, `Commit` at the end. Helpers that run inside the transaction take `*sql.Tx` as a parameter.
- **Pagination** is keyset (`WHERE id > ? ORDER BY id LIMIT ?`), asking for `limit + 1` rows to know whether there is a next page.
- An index hint or any non-obvious SQL gets a one-line comment saying why.

## MySQL and PostgreSQL

| | MySQL (`go-sql-driver/mysql`) | PostgreSQL (`pgx`) |
|---|---|---|
| Placeholders | `?` | `$1`, `$2`… |
| No rows | `sql.ErrNoRows` | `pgx.ErrNoRows` with the native pool, `sql.ErrNoRows` through `database/sql` |
| Unique violation | `*mysql.MySQLError` with `Number == 1062` | `*pgconn.PgError` with `Code == "23505"` |
| Returning generated values | separate select or `LastInsertId` | `RETURNING` |
| Pool type | `*sql.DB` | `*pgxpool.Pool` or `*sql.DB` |

The structure and the rules are the same for both. Only these details change.

## References

This layout combines three projects; none of them does all three things together.

- Mattermost, `server/channels/store/sqlstore/user_store.go`: the columns in a function, `getUsersColumns()`. Theirs returns `[]string` for a query builder; ours returns a `string`.
- Temporal, `common/persistence/sql/sqlplugin/mysql/execution.go`: the column list concatenated into the SQL, `INSERT INTO executions(` + executionsColumns + `)`. Theirs is a constant; ours is a function.
- Dex, `storage/sql/crud.go`: the `scanner` interface shared by row and rows, `scanRefresh(s scanner)`, and the SQL written inside each method. Dex also repeats the columns in every query.
