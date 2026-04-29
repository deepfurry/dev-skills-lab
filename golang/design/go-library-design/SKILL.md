---
name: go-library-design
description: Use when the task asks to initialize, design, or review a Go library, SDK, middleware, or utility package. This skill focuses on public API quality, package boundaries, dependency minimalism, context-aware operations, streaming-friendly I/O, error design, concurrency safety, examples-first documentation, and long-term maintainability.
---

# Go Library Design Skill

Use this skill when the user wants to initialize, design, or review a Go library, SDK, middleware, or utility package.

The goal is to create Go packages that are easy to use, easy to test, hard to misuse, and stable enough for open-source users.

This skill is not a general code security audit and is not a full application architecture workflow. It focuses on public API design, package boundaries, dependency strategy, error handling, context and I/O design, concurrency safety, examples, and documentation.

## Working Model

Before designing or reviewing a Go library, determine seven things:

- project type: library, SDK, middleware, utility package, CLI support package, or unknown
- target users: internal developers, open-source users, framework users, service developers, or unknown
- public API surface: exported functions, exported types, constructors, interfaces, config structs, and options
- dependency policy: standard library only, minimal third-party dependencies, framework-specific, or unknown
- operation type: CPU-only, file I/O, network I/O, streaming, middleware, background worker, or mixed
- concurrency expectation: single-use, reusable client, concurrent-safe client, per-request object, or unknown
- review target: initialize a new design, review existing code, or produce an API design review report

If the project type is unknown, assume a lightweight Go library or utility package.

If the dependency policy is unknown, prefer the standard library and avoid heavy third-party dependencies.

If concurrency expectations are unknown, require the generated or reviewed design to document whether exported clients and types are safe for concurrent use.

## Default Assumptions

- The package should be usable by other Go projects.
- The public API should be small and stable.
- The first example should be minimal and runnable.
- The package should avoid unnecessary dependencies.
- Blocking, I/O, network, retry, and long-running operations should accept `context.Context`.
- Data transfer APIs should prefer `io.Reader`, `io.Writer`, `io.ReadCloser`, or `fs.FS` when appropriate.
- Recoverable errors should return `error`, not panic.
- Internal implementation details should not leak into public API names.
- Concurrency safety must be documented.
- Framework-specific code should only be generated when the project is explicitly framework-specific.

## Project Type Rules

### General Go Library

For a general-purpose library, prefer a small public API.

Common layout:

```txt
.
├── README.md
├── go.mod
├── client.go
├── options.go
├── types.go
├── errors.go
├── internal/
└── examples/
    └── basic/
        └── main.go
```

Do not create `cmd/`, `pkg/`, `service/`, `repository/`, or `handler/` folders by default.

### SDK

For a Go SDK, prefer a reusable `Client`.

Common API shape:

```go
client, err := NewClient(Config{
    Endpoint: "http://127.0.0.1:8080",
    Timeout:  10 * time.Second,
})
```

SDKs should support:

- reusable clients
- custom `http.Client` when network calls are involved
- context-aware operations
- streaming-friendly I/O
- bounded retry behavior
- clear pagination models when listing resources
- typed or sentinel errors for common failure cases

### Middleware

For a middleware package, prefer a clear constructor and config.

Common API shape:

```go
mw := New(Config{
    Enabled: true,
})
```

Middleware packages should support:

- safe defaults
- a skip or next function when appropriate
- minimal hot-path overhead
- no unnecessary request body reads
- safe response body capture only when explicitly enabled
- clear framework compatibility
- clear performance notes

### Utility Package

For a utility package, keep the API function-oriented unless state is necessary.

Prefer:

```go
result, err := Analyze(input)
```

over a stateful client unless state, configuration, caching, or dependency injection is required.

## Public API Design Rules

Public APIs should be easy to discover and hard to misuse.

Check for:

- too many exported symbols
- exported types that expose implementation details
- constructors with unclear defaults
- APIs that require users to understand internal state
- confusing naming
- too many interfaces
- interfaces exported before there is a real need
- options that conflict with each other
- functions that do too many things
- surprising side effects
- public fields that should be private
- exported errors that are not useful for callers

Prefer:

- small constructors
- clear config
- narrow methods
- stable exported types
- unexported helpers
- examples that show intended usage

Avoid public APIs that are only convenient for the package author but awkward for users.

## Interface Rules

Do not introduce interfaces by default.

Use interfaces when:

- callers need to provide an implementation
- tests need a seam and concrete types are hard to use
- the package consumes behavior rather than owns it
- the interface is small and stable

Avoid:

- large interfaces
- exported interfaces with only one implementation and no user benefit
- interface pollution for every service or dependency
- forcing users to mock internals

Prefer accepting standard library interfaces:

```go
io.Reader
io.Writer
io.ReadCloser
fs.FS
http.RoundTripper
```

Do not export framework-shaped interfaces unless the library is explicitly framework-specific.

## Package Boundary Rules

Keep package boundaries simple.

Use `internal/` only for implementation that external users must not import.

Do not create empty folders for architecture theater.

Avoid default layouts such as:

```txt
cmd/
pkg/
internal/
service/
repository/
handler/
```

unless the project truly needs them.

Good boundaries:

- public package contains stable API
- `internal/` contains implementation details
- `examples/` contains runnable usage
- tests cover public behavior
- framework adapters are isolated when needed

For multi-framework middleware or adapters, consider separate packages:

```txt
middleware/
adapters/fiber/
adapters/gin/
```

or separate repositories if the dependency would otherwise pollute the core package.

## Dependency Rules

Prefer the standard library.

Add third-party dependencies only when they provide clear value.

Before adding a dependency, consider:

- whether the standard library is enough
- maintenance status
- transitive dependency weight
- security exposure
- API stability
- whether the dependency locks the package into a framework
- whether the dependency is only needed for examples or tests

Do not add heavy dependencies for small helpers.

Do not add a web framework dependency to a general library.

Do not add logging, metrics, database, cache, or configuration libraries unless the package's core purpose requires them.

When a dependency is optional, isolate it behind a separate package or build tag when appropriate.

## Config Design Rules

Use a `Config` struct when settings are clear and stable.

Example:

```go
type Config struct {
    Endpoint string
    Timeout  time.Duration
}
```

Use functional options when optional settings are many, layered, or likely to grow.

Example:

```go
client, err := NewClient(
    WithEndpoint("http://127.0.0.1:8080"),
    WithTimeout(10*time.Second),
)
```

Do not overuse both Config and functional options at the same time.

Good config design should:

- provide safe defaults
- validate required fields
- explain zero values
- avoid hidden global config
- avoid environment variable reads inside library code unless explicitly part of the design
- keep credentials out of logs and error messages

If using a `Config` struct, consider:

```go
func DefaultConfig() Config
func New(config Config) (*Client, error)
```

or document zero-value behavior clearly.

## Context Rules

Use `context.Context` for operations that may block, wait, perform I/O, retry, or call external systems.

Preferred shape:

```go
func (c *Client) Upload(ctx context.Context, name string, r io.Reader) (*UploadResult, error)
```

Rules:

- `context.Context` should be the first parameter.
- Do not store `context.Context` in structs.
- Do not use `context.Background()` inside library operations unless there is a strong reason.
- Propagate context to HTTP requests, database calls, file-like operations, workers, and retries when possible.
- Respect cancellation and deadlines.
- Do not ignore `ctx.Err()` in long-running loops.

Do not add context to pure CPU helpers unless cancellation is meaningful.

## I/O and Streaming Rules

Prefer streaming-friendly APIs.

Use:

```go
io.Reader
io.Writer
io.ReadCloser
fs.FS
```

when the operation transfers data.

Good patterns:

```go
func (c *Client) Upload(ctx context.Context, name string, r io.Reader) error
func (c *Client) Download(ctx context.Context, name string, w io.Writer) error
func Parse(r io.Reader) (*Result, error)
func Write(w io.Writer, result *Result) error
```

Avoid requiring:

```go
[]byte
string file path
large in-memory buffers
```

unless the API is explicitly a convenience wrapper.

For large data:

- do not read entire payloads into memory by default
- support streaming
- close resources when owned by the package
- do not close caller-owned readers or writers unless documented
- support progress or hooks only when useful and not intrusive

## Error Design Rules

Recoverable failures should return `error`.

Do not panic for normal runtime failures.

Use panics only for programmer errors when there is no reasonable recovery path.

Good error design should include:

- useful context
- wrapping with `%w`
- sentinel errors for common stable conditions
- typed errors when callers need structured information
- no secret leakage
- no huge response bodies in error strings
- documentation for common errors

Examples:

```go
var ErrNotFound = errors.New("not found")

type ResponseError struct {
    StatusCode int
    Message    string
}

func (e *ResponseError) Error() string {
    return fmt.Sprintf("unexpected response status: %d: %s", e.StatusCode, e.Message)
}
```

Prefer:

```go
return fmt.Errorf("upload %q: %w", name, err)
```

Avoid:

```go
return err
```

when the caller would lose important context.

## Concurrency Safety Rules

Document concurrency behavior.

For exported clients, caches, middleware, and reusable types, state whether they are safe for concurrent use.

Examples:

```txt
Client is safe for concurrent use.
```

or:

```txt
Client is not safe for concurrent use. Create one client per worker.
```

Check for:

- package-level mutable global state
- maps shared across goroutines
- mutable config after construction
- shared buffers
- unsafe caches
- unsynchronized metrics
- non-thread-safe callback usage
- request-scoped state stored globally

If a type should be concurrent-safe:

- make config immutable after construction
- protect shared state with locks or atomics
- avoid holding locks during slow I/O
- document callback concurrency behavior
- test with `go test -race`

If a type is not concurrent-safe, document it clearly and keep the API hard to misuse.

## Client Design Rules

For SDK clients:

- make clients reusable
- avoid global default clients unless safe and justified
- support custom `http.Client` for network SDKs
- set sane timeout defaults
- avoid infinite retries
- separate request options from client config
- make pagination explicit
- support context cancellation
- keep low-level HTTP details available only when useful

Recommended shape:

```go
type Client struct {
    endpoint   string
    httpClient *http.Client
}

func NewClient(config Config) (*Client, error)
```

Do not expose fields that should remain stable implementation details.

## Middleware Design Rules

For middleware packages:

- keep middleware responsibility focused
- provide a `Config` with safe defaults
- support skip logic when appropriate
- avoid heavy work in request hot paths
- avoid reading request bodies by default
- if reading request bodies, restore the body for downstream handlers
- make response capture optional
- avoid logging secrets or full request bodies
- avoid storing request-specific data globally
- do not keep framework context beyond request lifetime
- document performance impact
- document compatibility with framework versions

If framework-specific, keep the API idiomatic for that framework.

For Fiber-like middleware, a config may include:

```go
type Config struct {
    Next func(ctx *fiber.Ctx) bool
}
```

Only use framework-specific types when the package is explicitly for that framework.

## Testing Rules

A Go library should have tests that exercise public behavior.

Prefer:

- table-driven tests
- tests for config validation
- tests for error behavior
- tests for context cancellation when relevant
- tests for streaming behavior
- race tests for concurrent-safe types
- integration tests gated behind environment variables when needed

Do not make tests depend on live external services by default.

If integration tests need external services, document them clearly.

## Examples Rules

Every library, SDK, middleware, or utility package should include at least one minimal runnable example.

Default path:

```txt
examples/basic/main.go
```

The first example should be small enough to understand quickly.

README should show the same minimal path before advanced features.

Good examples:

- basic client creation
- one successful operation
- basic error handling
- minimal middleware usage
- minimal utility function usage

Avoid examples that require complex infrastructure unless the package itself requires it.

## Documentation Rules

README should include:

- Installation
- Quick Start
- Minimal Example
- Configuration
- Error Handling
- Concurrency Safety
- Examples
- Versioning or Compatibility Notes when relevant

For SDKs, document:

- timeout behavior
- retry behavior
- context behavior
- pagination behavior
- error types
- custom client support

For middleware, document:

- framework compatibility
- config defaults
- skip behavior
- body handling
- logging behavior
- performance impact

Do not write only marketing copy. Documentation must help users integrate the package.

## API Design Review Report

When reviewing an existing library, create or propose a Markdown report under:

```txt
docs/api-design-review.md
```

For dated reports, use:

```txt
docs/reviews/YYYY-MM-DD-api-design-review.md
```

Default report structure:

```md
# Go Library API Design Review

## Summary

## Scope

## Project Type

## Public API Review

## Package Boundary Review

## Dependency Review

## Config Design Review

## Context and I/O Review

## Error Handling Review

## Concurrency Safety Review

## Documentation and Examples Review

## Recommended Changes

## Breaking Change Risk

## Next Steps
```

Use the user's language for the report unless they ask otherwise.

## Design Finding Levels

Use these levels for design findings:

### Blocking

Use for issues that seriously harm usability, compatibility, or long-term maintainability.

Examples:

- public API exposes unstable internals
- no clear constructor or default usage path
- network API lacks context support
- large data API forces full memory loading
- concurrency safety is unclear for a reusable client
- library panics on recoverable errors

### Recommended

Use for important improvements that should be addressed soon.

Examples:

- config defaults are unclear
- errors lack useful wrapping
- examples are incomplete
- dependency can be removed
- package boundary is confusing
- tests are missing for public behavior

### Optional

Use for nice-to-have improvements.

Examples:

- naming polish
- additional examples
- convenience wrappers
- extra docs
- minor refactors
- optional benchmarks

## Finding Format

Use this format in design review reports:

```md
### Recommended-001: Short issue title

- Level: Recommended
- Category: Public API / Package Boundary / Dependency / Config / Context / I/O / Error / Concurrency / Documentation / Testing
- Location: `path/to/file.go` or `path/to/file.go:line`
- Breaking Change Risk: Low / Medium / High
- Confidence: High / Medium / Low

#### Problem

Explain the design issue.

#### Impact

Explain how it affects users or maintainers.

#### Recommendation

Provide a concrete improvement.

#### Suggested API

Show a better API shape when useful.

#### Migration Notes

Explain how users can migrate if the change is breaking.
```

## Breaking Change Rules

Always consider compatibility.

Before recommending a public API change, classify breaking risk:

```txt
Low
Medium
High
```

Prefer additive changes when possible.

For high-risk breaking changes, suggest:

- keeping old API temporarily
- adding a new API first
- deprecating old symbols
- documenting migration path
- using a major version bump if needed

Do not casually recommend breaking public APIs without explaining migration.

## Initialization Output Rules

When initializing a new library design, provide:

- recommended directory layout
- public API sketch
- config design
- error design
- context and I/O rules
- concurrency safety statement
- minimal example
- README outline
- testing plan

Do not generate large amounts of implementation code unless the user asks.

Focus on the design contract first.

## Review Output Rules

When reviewing an existing library, provide:

- report path
- design summary
- top Blocking findings
- top Recommended findings
- low-risk quick wins
- breaking change risks
- suggested next steps

Do not treat this as a security audit. Use `go-code-audit` for vulnerabilities, concurrency bugs, or production safety risks.

## Hard Rules

- Do not over-abstract public APIs.
- Do not expose internal implementation details.
- Prefer standard library dependencies.
- Prefer context-aware APIs for blocking, network, file, retry, or long-running operations.
- Prefer `io.Reader` and `io.Writer` for data transfer APIs.
- Do not read large payloads fully into memory by default.
- Do not use package-level mutable global state unless justified.
- Document whether exported clients, middleware, and reusable types are safe for concurrent use.
- Provide a minimal runnable example.
- Do not generate framework-specific code unless the project is framework-specific.
- Do not create empty `cmd/`, `pkg/`, or `internal/` directories without purpose.
- Do not panic in library code for recoverable errors.
- Do not recommend breaking public APIs without migration notes.

## Reject These Failures

- Public API is large but lacks a minimal usage path.
- The design exposes implementation internals.
- Blocking operations lack `context.Context`.
- SDK data transfer APIs only accept file paths or `[]byte` when streaming is needed.
- Errors are swallowed or returned without useful context.
- Library code panics for normal errors.
- Concurrency safety is not documented.
- A general library imports a web framework without a strong reason.
- Middleware reads request bodies by default without restoring them.
- README lacks a minimal runnable example.
- Review gives vague advice without suggested API shapes.
- Recommended changes ignore breaking change risk.

## Litmus Checks

- Can a new user understand the package from the first README example?
- Can the default usage fit in a short snippet?
- Are advanced options available without making basic usage hard?
- Are package boundaries clear?
- Can the package evolve without breaking users unnecessarily?
- Are dependencies justified?
- Are blocking operations cancellable?
- Can large data be streamed?
- Are errors useful to callers?
- Is concurrency behavior explicit?
