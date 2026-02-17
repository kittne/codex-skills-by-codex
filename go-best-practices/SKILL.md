---
name: go-best-practices
description: >
  Use when writing or reviewing Go code to ensure idiomatic style, up-to-date language features, and
  best practices.
---

# Go Best Practices

## Workflow
1. Confirm Go version, module layout, and package boundaries (`go env`, `go list`, `go mod tidy` when appropriate).
2. Apply idiomatic Go style and naming (gofmt/goimports, no stutter, small packages).
3. Handle errors explicitly and consistently (wrapping, `errors.Is/As`, no panic for expected errors).
4. Validate context usage, cancellation/timeouts, and resource cleanup (files, HTTP bodies, goroutines).
5. Review concurrency for races, leaks, and lifecycle management.
6. Add/update tests and validate with `go test`, `go vet`, and `-race` when relevant.

## Preflight (Ask / Check First)
- Target Go version (from `go.mod` and CI).
- Package boundaries and public API surface (exported identifiers).
- Whether this is library code vs service code (logging, config, main).
- Concurrency model: goroutines per request? background workers? cancellation?

## Project Structure
- Use `go.mod` and keep modules small and focused.
- Organize packages by domain, not by type.
- Avoid circular dependencies and overly deep package trees.
- Use `internal/` for packages that should not be imported externally.
- Use `cmd/<name>/` for multiple binaries when applicable.

## Naming and Style
- Use concise, descriptive names; avoid stutter (`package user`, `type User`).
- Favor small, focused functions.
- Use `gofmt` and `goimports` consistently.
- Avoid `GetX()`; prefer `X()` for getters.
- Preserve initialisms (`ID`, `HTTP`, `URL`) in exported names.

## Error Handling
- Return errors explicitly; avoid panics in normal flows.
- Wrap errors with context and sentinel errors where appropriate.
- Prefer `errors.Is` and `errors.As` for inspection.
- Keep error strings lowercase and without trailing punctuation.

## Context and Cancellation
- Pass `context.Context` as the first param when needed.
- Respect cancellation and timeouts in I/O and goroutines.
- Avoid storing context in structs.
- In servers, set timeouts (HTTP clients and servers) to avoid hanging requests.

## Concurrency
- Avoid shared mutable state; use channels or mutexes appropriately.
- Close channels only by senders; avoid closing from multiple goroutines.
- Use `sync.WaitGroup` for lifecycle coordination.
- Run `go test -race` for concurrent code and treat races as correctness bugs.
- Ensure goroutines have an exit condition to avoid leaks.

## Interfaces and Abstractions
- Keep interfaces small and consumer-driven.
- Prefer concrete types; introduce interfaces at call sites for testing.
- Avoid "utility" mega-packages; keep abstractions close to where they are used.

## Performance
- Avoid unnecessary allocations; use `bytes.Buffer` and preallocate slices.
- Benchmark before optimizing; use `pprof` when needed.

## Testing
- Table-driven tests for coverage and clarity.
- Use subtests (`t.Run`) and helpers for shared setup.
- Prefer integration tests for real-world flows.
- Add regression tests for bug fixes.

## Tooling
- Run `gofmt`/`goimports` on changed files.
- Run `go test ./...` and `go test -race ./...` when concurrency is involved.
- Run `go vet ./...` and a linter suite (e.g., golangci-lint/staticcheck) when configured.

Common commands:
```bash
gofmt -w .
goimports -w .
go test ./...
go test -race ./...
go vet ./...
go mod tidy
```

## Review Checklist
- Code is gofmt/goimports clean and idiomatic.
- Errors are handled with context.
- Concurrency is safe and cancellable.
- Tests cover critical paths.
- No leaked goroutines or unbounded resource use.

## References
- `references/go-best-practices.md`

## Extended Guidance
Use this for larger Go services or when concurrency is involved.

## Project Structure Checklist
- Keep packages small and purpose-driven.
- Avoid cyclic imports by pushing shared types to a lower layer.
- Prefer `internal/` for non-public packages.

## Error Handling Rules
- Wrap errors with context at boundaries.
- Avoid panic in normal error paths.
- Use sentinel errors sparingly; prefer typed errors when needed.

## Concurrency Rules
- Use contexts for cancellation and deadlines.
- Limit goroutine fan-out; use worker pools for large workloads.
- Avoid sharing mutable state without clear locking.

## Testing and Tooling
- Use table-driven tests for repeatable cases.
- Add `go test ./...` to CI with race detector when possible.
- Run `golangci-lint` with a minimal, agreed set of linters.

## Reference Index
- `rg -n "Error handling|wrap" references/go-best-practices.md`
- `rg -n "Concurrency|goroutines|channels" references/go-best-practices.md`
- `rg -n "Testing|table-driven" references/go-best-practices.md`

## API Design Notes
- Prefer small interfaces; keep implementations internal.
- Return concrete types from constructors when possible.
- Avoid exposing struct fields unless stable.

## Reference Index (Expanded)
- `rg -n "Project layout|packages" references/go-best-practices.md`

## Quick Questions (When Stuck)
- What is the minimal change that solves the issue?
- What is the rollback plan?
- What is the highest-risk assumption?
- What is the simplest validation step?
- What is the known-good baseline?
- What evidence would change the decision?
- What is the user-visible impact?
- What is the operational impact?
- What is the most likely failure mode?
- What is the fastest safe experiment?

## Reference Index (Extra)
- `rg -n "Checklist|checklist" references/go-best-practices.md`
- `rg -n "Example|examples" references/go-best-practices.md`
- `rg -n "Workflow|process" references/go-best-practices.md`
- `rg -n "Pitfall|anti-pattern" references/go-best-practices.md`
- `rg -n "Testing|validation" references/go-best-practices.md`
- `rg -n "Security|risk" references/go-best-practices.md`
- `rg -n "Configuration|config" references/go-best-practices.md`
- `rg -n "Deployment|operations" references/go-best-practices.md`
- `rg -n "Troubleshoot|debug" references/go-best-practices.md`
- `rg -n "Performance|latency" references/go-best-practices.md`
- `rg -n "Reliability|availability" references/go-best-practices.md`
- `rg -n "Monitoring|metrics" references/go-best-practices.md`
- `rg -n "Error|failure" references/go-best-practices.md`
- `rg -n "Decision|tradeoff" references/go-best-practices.md`
- `rg -n "Migration|upgrade" references/go-best-practices.md`
