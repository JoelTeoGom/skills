---
name: go-hexagonal-architecture
description: Folder and file structure of Joel's flat hexagonal template for Go. Use only when the project already follows it (internal/domain, internal/ports, internal/service, internal/adapters) or when Joel asks for this architecture. Do not use for other Go projects, such as operators, CLIs or libraries, which keep their own structure.
---

# Go hexagonal architecture

Follow the structure of [go-hexagonal-template](https://github.com/JoelTeoGom/go-hexagonal-template). It defines the architecture, the folders and the files. The rest of its code is an example: the login domain is filler, and naming, comments and style come from their own skills.

## When it applies

This architecture fits some projects, not all Go code. Use it only in two cases:

- The project already has this structure.
- Joel asks for it, for example to start a new service with it.

Any other project keeps its own structure, and this skill and the template are not consulted. An operator, a CLI or a library has its own natural layout. A new project that does not fit either case does not get this structure by default.

## Structure

```
cmd/api/main.go                     # wiring and process lifecycle
internal/
├── config/config.go                # environment variables, imported by main only
├── domain/
│   ├── <entity>.go                 # one file per entity
│   └── errors.go                   # business errors
├── ports/
│   ├── in.go                       # what the application offers
│   └── out.go                      # what the application needs
├── service/
│   ├── <use-case>.go               # one file per group of use cases
│   └── <use-case>_test.go          # tests with in-memory fakes of the ports
└── adapters/
    ├── in/httpapi/
    │   ├── server.go               # Server, constructor and routes
    │   ├── handlers.go             # handlers
    │   ├── dto.go                  # request and response types
    │   ├── errors.go               # error translation in one place
    │   ├── json.go                 # encode and decode helpers
    │   └── middleware.go           # middleware
    └── out/
        ├── mysql/
        │   ├── db.go               # connection
        │   └── <entity>_repository.go
        └── <technology>/           # one package per technology, such as token/ or password/
migrations/                         # schema SQL
```

## Files

Segmentation is the point: opening a file shows one thing. Each file has one responsibility and its name says which, so the code can be reviewed by clicking through the tree.

- Do not pile unrelated code into one file. A server, its handlers and its DTOs live in separate files.
- When a file starts doing two jobs, split it into a sibling file in the same folder.
- Keep it flat: one folder per layer, with no sublayers and no package per use case, until the project needs more depth.

## Dependencies

Dependencies always point toward the center. If a change needs to break this table, the code belongs in another layer.

| Package | May import |
|---|---|
| `domain` | stdlib |
| `ports` | `domain` |
| `service` | `domain`, `ports` |
| `adapters/in/*`, `adapters/out/*` | `domain`, `ports`, libraries for their technology |
| `config` | stdlib |
| `cmd/api` | everything |

## Adding a feature

Work from the inside out: `domain`, `ports`, `service` with its tests, `adapters/out`, `adapters/in`, and finally the wiring in `main`.

## When in doubt

Consult the repo instead of guessing. Fetch only the file that answers the doubt from:

```
https://raw.githubusercontent.com/JoelTeoGom/go-hexagonal-template/main/<path>
```

| Doubt | Path |
|---|---|
| Where something goes, what each layer holds | `README.md` |
| How a port is laid out | `internal/ports/in.go`, `internal/ports/out.go` |
| How a service is laid out | `internal/service/user.go` |
| How tests and fakes are laid out | `internal/service/auth_test.go` |
| How an HTTP adapter is split | `internal/adapters/in/httpapi/server.go`, then its sibling files |
| How an outbound adapter is laid out | `internal/adapters/out/mysql/user_repository.go` |
| How the pieces are wired | `cmd/api/main.go` |

If the repo cannot be reached, say so and do not invent the convention.
