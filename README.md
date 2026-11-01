# scheme — legacy shared proto repository

The original c12s "one place for all `.proto` files" repository: per-service
protobuf definitions (`apollo`, `blackhole`, `celestial`, `core`, `flusher`,
`gravity`, `magnetar`, `meridian`, `stellar`) with the generated Go code
(`*.pb.go`, old `protoc-gen-go` style) committed next to them.

## Role in c12s

**Historical / legacy.** The active services no longer import this module:
each repo carries its own protos and generated code (`magnetar/pkg/api`,
`gravity/pkg/api`, `kuiper/pkg/api`, `star/pkg/api`, `apollo/proto1`,
`oort/pkg/api`), and consumers wire them via `go.mod` `replace` directives to
the sibling directories in this workspace.

Remaining users are the legacy repos only:

| Repo | Imports |
|---|---|
| `celestial`, `blackhole`, `stellar`, `stellar-go` | `github.com/c12s/scheme/...` |

The protos here also **diverge from today's services**: e.g.
`scheme/magnetar/magnetar.proto` still declares `MagnetarService`
(`Reserve`/`List`/`Free`/`Query`) while the current Magnetar gRPC API is
`GetFromNodePool`/`ClaimOwnership`/`SetDesiredOsImage`/… and lives in
`magnetar/pkg/api/proto/`; `scheme/gravity/gravity.proto` is the old
`GravityService.PutTask`, not the current `AgentQueue`.

For the bootc work (thesis Phases 1–5) this repository is **not involved**.

## Contents

```
<service>/<service>.proto     definition
<service>/<service>.pb.go     generated code (committed)
gen.sh                        regeneration script
```

## Regeneration (as shipped — stale)

```bash
# from the repo root, with $GOPATH set and the ancient toolchain:
protoc -I. --go_out=plugins=grpc:$GOPATH/src/. magnetar/magnetar.proto
# ... see gen.sh for all services
```

## Bugs, disabled and unfinished features

- `gen.sh` uses the **pre-`protoc-gen-go` v1.4 call style**
  (`--go_out=plugins=grpc:...`) and writes into `$GOPATH/src/`. It cannot run
  with any current protoc/protoc-gen-go and has not been used since the
  monorepo-import era.
- There is **no CI/verification** that the committed `.pb.go` files match
  their `.proto` sources.
- The protos are frozen at the ~2022 service model and mislead anyone who
  reads them as the current API (use the consumer repos' `pkg/api` instead).
- `README.md` does not exist prior to this file; the workspace AGENTS.md
  statement that "all `.proto` files live in `scheme/`" describes the old
  layout and is no longer true.

## Status

Legacy — kept for the out-of-scope legacy services. Do not use for
thesis-relevant API changes.
