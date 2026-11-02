---
name: go-domain-files
description: How Go domain and model types are grouped into files. Use whenever creating, moving or splitting structs, enums or value objects in a domain, model or types package of a Go project, even when file layout is not mentioned.
---

# Go domain files

Go groups types by concept, not one file per struct. A file per type is the Java habit: it scatters an entity across many tiny files and the reader has to open all of them to understand it. One giant file for the whole domain is the opposite mistake.

## Rule

Each entity gets one file named after it, and that file holds everything that only exists for the entity: its struct, its sub-structs, its enums with their constants and its value objects.

A type gets its own file only when one of these is true:

1. Several entities use it, such as a status shared by runs, jobs and tasks.
2. It has its own behavior with enough code to stand alone, such as methods, validation or parsing.
3. It is a separate concept that makes sense without the entity, such as a worker, or the HTTP request and response a job sends.

When none applies, the type lives with its entity. Its validation methods and constants move with it.

## errors.go

Business errors always live in `errors.go`, separate from the entities, even when there is only one. The helpers that build them, such as an `invalid(message)` wrapper, go there too. Every layer that checks an error with `errors.Is` knows where to find it.

## Naming

Inside the entity file, prefix generic names with the entity: `JobConfig`, not `Config`. A bare `domain.Config` reads like the service configuration.

## Example

Avoid:

```
domain/
├── config.go     # Config of the job
├── job.go
├── priority.go   # only used by Job
├── request.go
├── result.go
├── retry.go      # only used by Job
├── status.go     # only used by Job
└── worker.go
```

Good:

```
domain/
├── job.go        # Job, JobConfig, Status, Priority, RetryPolicy
├── http.go       # Request, Result
├── worker.go     # Worker
└── errors.go     # business errors
```

If a second entity starts using `Priority` or `RetryPolicy`, rule 1 applies and they move to their own file.

## References

- Kubernetes `k8s.io/api/batch/v1/types.go`: `Job`, `JobSpec`, `JobStatus`, its conditions, failure policies and enums in one file.
- Grafana `pkg/services/ngalert/models/alert_rule.go`: `AlertRule` with its state enums and keys; other entities get their own files.
- Gitea `models/actions/status.go`: `Status` has its own file because runs, jobs, tasks and steps share it.
- Wild Workouts `internal/trainer/domain/hour/availability.go`: `Availability` has its own file because of its behavior.
