---
name: go-middleware-design
description: Use when the task asks to initialize, design, or review a Go Web middleware package. This skill focuses on middleware responsibility boundaries, config defaults, skip logic, request and response body handling, hot-path performance, concurrency safety, framework adapters, error handling, observability, examples, and documentation.
---

# Go Middleware Design Skill

Use this skill when the user wants to initialize, design, or review a Go Web middleware package.

The goal is to create middleware that is focused, safe under concurrent requests, easy to configure, low-overhead in hot paths, and friendly to framework users.

This skill is not a general Go library design workflow and is not a deep vulnerability audit. It focuses specifically on middleware API design, lifecycle behavior, request and response handling, framework adaptation, observability, performance, concurrency safety, examples, and documentation.

## Working Model

Before designing or reviewing a Go Web middleware, determine eight things:

- middleware type: logging, metrics, tracing, authentication, authorization, WAF, rate limiting, profiler, sensitive-word filtering, CORS, recovery, compression, or unknown
- target framework: `net/http`, Fiber, Gin, Echo, Chi, or another Go Web framework
- responsibility boundary: what the middleware should do and what it must not do
- request handling needs: whether it reads headers, path, query, body, cookies, client IP, or context values
- response handling needs: whether it observes status, headers, body, latency, errors, or streaming behavior
- runtime sensitivity: public service, internal service, high-concurrency service, security-sensitive service, or unknown
- configuration style: `Config` struct, functional options, framework-style variadic config, or unknown
- review target: initialize new middleware, review existing code, or produce a middleware design review report

If the framework is unknown, default to `net/http`.

If the middleware type is unknown, keep the design minimal and avoid security-sensitive assumptions.

If runtime sensitivity is unknown, assume the middleware may run in production under concurrent traffic and may receive untrusted input.

## Default Assumptions

- Middleware runs on every matching request.
- Middleware should have one clear responsibility.
- Middleware must be safe for concurrent requests after construction.
- Config should have safe defaults.
- Skip or Next logic should be supported when useful.
- Request body reading should be avoided by default.
- Response body capture should be disabled by default.
- Hot-path logic should be lightweight.
- Framework context must not be kept beyond request lifetime.
- Logs and metrics must avoid leaking sensitive data.
- Documentation must explain performance and security tradeoffs.

## Middleware Responsibility Rules

A middleware should have one primary job.

Good responsibility boundaries:

- logging middleware records request summaries
- metrics middleware collects stable request metrics
- authentication middleware verifies identity and injects identity context
- authorization middleware enforces permission decisions
- WAF middleware inspects requests and optionally responses
- recovery middleware catches panics and produces safe responses
- profiler middleware measures request behavior
- sensitive-word middleware detects, masks, tags, or blocks configured content
- rate-limiting middleware enforces request quotas

Avoid middleware that mixes unrelated responsibilities, such as:

```txt
authentication + logging + rate limiting + response formatting
```

Do not put business logic inside reusable middleware.

Do not make middleware responsible for application-specific database updates unless the middleware is explicitly part of that application.

## Public API Shapes

### net/http Middleware

For generic `net/http` middleware, prefer:

```go
func New(config Config) func(http.Handler) http.Handler
```

or:

```go
func Middleware(next http.Handler) http.Handler
```

when no config is needed.

### Fiber Middleware

For Fiber-specific middleware, prefer:

```go
func New(config ...Config) fiber.Handler
```

### Gin Middleware

For Gin-specific middleware, prefer:

```go
func New(config Config) gin.HandlerFunc
```

### Echo Middleware

For Echo-specific middleware, prefer:

```go
func New(config Config) echo.MiddlewareFunc
```

### Chi Middleware

For Chi-compatible middleware, prefer:

```go
func New(config Config) func(http.Handler) http.Handler
```

Do not introduce framework-specific types into a generic middleware core.

If the project supports multiple frameworks, keep framework adapters separate.

## Recommended Layouts

### Single-Framework Middleware

For a package dedicated to one framework:

```txt
.
├── README.md
├── go.mod
├── middleware.go
├── config.go
├── errors.go
├── internal/
└── examples/
    ├── basic/
    │   └── main.go
    └── with-config/
        └── main.go
```

### Generic Core With Framework Adapters

For middleware with framework-independent core logic:

```txt
.
├── README.md
├── go.mod
├── config.go
├── middleware.go
├── internal/
│   └── engine/
├── adapters/
│   ├── fiber/
│   ├── gin/
│   ├── echo/
│   └── chi/
└── examples/
    ├── nethttp/
    ├── fiber/
    └── gin/
```

Use adapters only when the core logic can remain framework-independent.

Do not create adapter folders if there is only one target framework.

## Config Design Rules

Middleware config must be easy to understand and safe by default.

Prefer a `Config` struct:

```go
type Config struct {
    Enabled bool
}
```

Provide defaults when useful:

```go
func DefaultConfig() Config
```

Good config should:

- have safe zero-value or documented zero-value behavior
- validate required fields
- avoid conflicting options
- avoid hidden global state
- avoid reading environment variables inside middleware code unless explicitly designed
- avoid storing request-specific values
- make expensive features opt-in
- clearly document callback timing and concurrency behavior

For framework-specific middleware, a variadic config can be idiomatic:

```go
func New(config ...Config) fiber.Handler
```

If no config is provided, use `DefaultConfig()`.

## Skip / Next Rules

Middleware should support skip logic when useful.

Common use cases:

- skip health checks
- skip static assets
- skip metrics endpoints
- skip public paths
- skip `OPTIONS` requests
- skip WebSocket upgrade requests
- skip streaming endpoints
- skip large file downloads

Fiber-style:

```go
type Config struct {
    Next func(c *fiber.Ctx) bool
}
```

Generic style:

```go
type Skipper func(r *http.Request) bool
```

Rules:

- run skip logic at the beginning of the middleware
- skip without side effects
- do not log, mutate, inspect body, or allocate large objects before skip checks
- default should usually be no skip unless the framework has a clear convention
- document skip behavior

## Request Body Handling Rules

Do not read request bodies by default.

Reading request bodies is expensive and risky.

If the middleware must read the body:

- make it opt-in
- enforce maximum body size
- restore the body for downstream handlers
- avoid logging full body content
- support body preview limits
- mask sensitive fields when logging or auditing
- avoid reading streaming bodies
- document memory impact
- document behavior for oversized bodies

Recommended config fields when body capture is needed:

```go
type Config struct {
    CaptureRequestBody bool
    MaxRequestBodySize int64
    BodyPreviewBytes   int
}
```

Never read unbounded request bodies.

Never consume the body without restoring it for downstream handlers.

Avoid body handling for:

- file upload endpoints
- streaming uploads
- large JSON payloads
- multipart forms
- SSE or upgraded connections

## Response Body Handling Rules

Do not capture response bodies by default.

Response body capture must be:

- opt-in
- size-limited
- compatible with status and headers
- disabled for streaming responses unless explicitly supported
- documented as a performance-sensitive feature

Recommended config fields:

```go
type Config struct {
    CaptureResponseBody bool
    MaxResponseBodySize int64
}
```

Be careful with:

- file downloads
- compressed responses
- streaming responses
- SSE
- WebSocket upgrades
- reverse proxy responses
- very large JSON responses

If response body capture is not required, observe only:

- status code
- content length if available
- content type
- latency
- error value when the framework exposes it

## Hot-Path Performance Rules

Middleware runs frequently, so hot-path cost matters.

Avoid in request hot paths:

- compiling regex
- parsing config
- reading large bodies
- blocking network calls
- synchronous slow logging
- unbounded map writes
- reflection-heavy logic
- high-cardinality metric labels
- global lock contention
- holding locks during slow I/O
- allocating large temporary buffers

Prefer:

- precomputing config at construction time
- compiling rules once
- small immutable config structs
- bounded buffers
- `sync.Pool` only when there is measured benefit
- stable metric labels
- route templates instead of raw URLs
- fast skip checks before expensive work

Do not call this skill's performance suggestions a benchmark result unless actual benchmarks were run.

## Concurrency Safety Rules

Middleware instances should usually be safe for concurrent use after construction.

Check for:

- mutable package-level global state
- maps shared across requests
- mutable config after construction
- shared buffers
- unsynchronized counters
- callbacks that mutate shared data
- framework context stored beyond request lifetime
- request-specific values stored globally
- hot reload or rule reload without synchronization

Rules:

- copy or freeze config at construction time
- protect shared state with locks or atomics
- avoid locks around slow operations
- document callback concurrency behavior
- avoid per-request mutation of shared config
- test with `go test -race` when feasible

README should state concurrency behavior:

```txt
Middleware instances are safe for concurrent use after construction.
```

If not safe, document the limitation clearly.

## Framework Context Rules

Do not keep framework request context after the request returns.

Never store long-lived references to:

- `*fiber.Ctx`
- `*gin.Context`
- `echo.Context`
- request-specific `context.Context` with canceled lifetime
- response writers
- request bodies

If asynchronous work is needed, extract safe values first:

- request ID
- method
- route template
- status code
- sanitized user ID
- latency
- sanitized error summary

Do not pass live framework contexts into background goroutines.

## Error Handling Rules

Middleware must define how errors are handled.

Clarify:

- what happens when middleware detects a violation
- what happens when middleware itself fails
- whether the request should continue or stop
- how block responses are generated
- how errors are logged
- whether errors are returned to the framework
- whether sensitive details are hidden from the client

Support an error handler when useful:

```go
type ErrorHandler func(ctx Context, err error) error
```

Framework-specific example:

```go
type ErrorHandler func(c *fiber.Ctx, err error) error
```

For security middleware, distinguish:

- allow
- block
- internal error
- configuration error
- unsupported request type

Do not leak internal detection details to clients by default.

## Observability Rules

Middleware often produces logs, metrics, and traces.

### Logging

Logs should be useful and safe.

Do not log by default:

- full request bodies
- full response bodies
- authorization headers
- cookies
- tokens
- passwords
- private keys
- session IDs
- raw query strings containing secrets

Prefer stable fields:

```txt
request_id
method
route
status
latency_ms
decision
error_kind
```

### Metrics

Metric labels must be low-cardinality.

Good labels:

```txt
method
route
status
decision
```

Bad labels:

```txt
url
user_id
ip
token
query
email
request_id
```

Use route templates instead of raw paths when possible.

### Tracing

Create spans only when useful.

Avoid trace noise for tiny middleware steps unless the middleware performs meaningful work.

Do not attach sensitive payloads to spans.

## Security Middleware Rules

For WAF, authentication, authorization, rate limiting, sensitive-word filtering, or risk-control middleware, define the security model clearly.

Clarify:

- fail-open or fail-closed behavior
- block response format
- audit logging behavior
- sensitive data masking
- rule update behavior
- maximum inspected body size
- timeout behavior
- bypass or skip rules
- ReDoS risk if regex is used
- whether detection details are exposed

Security middleware should avoid:

- exposing exact rule matches to attackers
- logging raw secrets
- regex patterns vulnerable to catastrophic backtracking
- unbounded input inspection
- global mutable rule sets without synchronization
- blocking all traffic on recoverable internal errors unless fail-closed is intended

Fail-open or fail-closed must be explicit and configurable when both modes are valid.

## Middleware Categories

### Logging Middleware

Should focus on:

- request summary
- latency
- status
- route
- request ID
- safe error summary

Avoid full body logging by default.

### Metrics Middleware

Should focus on:

- request count
- latency histogram
- in-flight requests
- status buckets
- stable route labels

Avoid high-cardinality labels.

### Recovery Middleware

Should focus on:

- catching panics
- logging sanitized panic information
- returning safe error responses
- preserving stack traces only in logs or debug mode

Do not expose stack traces to clients by default.

### Auth Middleware

Should focus on:

- extracting credentials
- validating identity
- injecting identity into request context
- rejecting invalid credentials

Do not mix authorization rules unless the middleware is explicitly designed for both.

### WAF / Risk Middleware

Should focus on:

- bounded request inspection
- rule execution
- allow/block decisions
- audit events
- safe failure behavior

Document performance cost and body handling clearly.

## Documentation Rules

README should include:

- Installation
- Quick Start
- Basic Usage
- Config
- Skip / Next
- Error Handling
- Request Body Handling
- Response Body Handling if supported
- Performance Notes
- Security Notes
- Framework Compatibility
- Concurrency Safety
- Examples

The first example must be minimal and runnable.

Do not write only marketing copy. Documentation must help users integrate the middleware correctly.

## Examples Rules

Every middleware package should include at least one minimal runnable example.

Recommended examples:

```txt
examples/basic/main.go
examples/with-config/main.go
examples/skip-paths/main.go
examples/error-handler/main.go
```

Only create examples that match the actual middleware features.

The basic example should show the smallest useful setup.

For framework-specific middleware, examples should use that framework idiomatically.

## Testing Rules

Middleware tests should cover behavior, not only construction.

Prefer tests for:

- default config
- config validation
- skip logic
- request pass-through
- request blocking
- error handling
- request body restoration when body reading is enabled
- response status observation
- response body capture when supported
- concurrency safety when shared state exists
- no secret leakage in logs when applicable

For Go middleware, also consider:

```bash
go test ./...
go test -race ./...
```

Do not require live external services for normal tests.

## Middleware Design Review Report

When reviewing an existing middleware package, create or propose a Markdown report under:

```txt
docs/middleware-design-review.md
```

For dated reports, use:

```txt
docs/reviews/YYYY-MM-DD-middleware-design-review.md
```

Default report structure:

```md
# Go Middleware Design Review

## Summary

## Scope

## Middleware Type

## Framework Compatibility

## Public API Review

## Config Review

## Skip / Next Review

## Request Handling Review

## Response Handling Review

## Performance Review

## Concurrency Safety Review

## Error Handling Review

## Observability Review

## Documentation and Examples Review

## Recommended Changes

## Breaking Change Risk

## Next Steps
```

Use the user's language for the report unless they ask otherwise.

## Design Finding Levels

Use these levels for middleware design findings.

### Blocking

Use for issues that seriously harm correctness, integration, compatibility, or safe production use.

Examples:

- middleware consumes request body and does not restore it
- middleware is not safe under concurrent requests
- API has no clear configuration path
- security middleware has unclear fail-open or fail-closed behavior
- response body capture breaks streaming responses
- framework context is stored beyond request lifetime

### Recommended

Use for important improvements that should be addressed soon.

Examples:

- missing skip logic
- unclear config defaults
- insufficient performance notes
- error handler is not configurable
- request body size is not configurable
- metrics labels are too high-cardinality
- examples are incomplete
- framework compatibility is unclear

### Optional

Use for nice-to-have improvements.

Examples:

- extra examples
- naming polish
- convenience helpers
- additional docs
- optional benchmarks
- additional framework adapter

## Finding Format

Use this format in design review reports:

```md
### Recommended-001: Short issue title

- Level: Recommended
- Category: API / Config / Skip / Request Handling / Response Handling / Performance / Concurrency / Error Handling / Observability / Documentation / Testing
- Location: `path/to/file.go` or `path/to/file.go:line`
- Breaking Change Risk: Low / Medium / High
- Confidence: High / Medium / Low

#### Problem

Explain the design issue.

#### Impact

Explain how it affects users, runtime behavior, performance, or maintainers.

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

Do not casually recommend breaking public middleware APIs without explaining migration.

## Initialization Output Rules

When initializing a new middleware design, provide:

- target framework
- recommended directory layout
- public API sketch
- config design
- skip or next design
- request body policy
- response body policy
- error handling policy
- observability policy
- concurrency safety statement
- minimal example
- README outline
- testing plan

Do not generate large amounts of implementation code unless the user asks.

Focus on the middleware contract first.

## Review Output Rules

When reviewing an existing middleware package, provide:

- report path
- design summary
- top Blocking findings
- top Recommended findings
- low-risk quick wins
- breaking change risks
- suggested next steps

Do not treat this as a deep security audit. Use `go-code-audit` for vulnerability analysis, exploit paths, race bugs, or production safety risks.

## Hard Rules

- Middleware must have one clear responsibility.
- Middleware must be safe for concurrent requests after construction.
- Config must have safe defaults or documented zero-value behavior.
- Skip or Next logic should be supported when useful.
- Do not read request body by default.
- If request body is read, size must be limited and the body must be restored.
- Response body capture must be optional and size-limited.
- Do not perform heavy work in request hot paths.
- Do not log secrets, tokens, full request bodies, full response bodies, or sensitive headers by default.
- Do not use high-cardinality values as metric labels.
- Do not keep framework request context beyond request lifetime.
- Do not introduce framework dependencies into a generic middleware core.
- Provide minimal runnable examples.
- Document performance impact and concurrency safety.

## Reject These Failures

- Middleware mixes unrelated responsibilities.
- Middleware consumes request body without restoring it.
- Middleware reads unbounded request bodies.
- Response capture is always enabled without size limits.
- Middleware stores framework context for later use.
- Middleware uses raw URL, user ID, IP, token, or query as metric labels.
- Middleware logs secrets or raw request bodies by default.
- Middleware is unsafe under concurrent requests.
- Config defaults are unclear or unsafe.
- Security middleware has unclear fail-open or fail-closed behavior.
- Framework-specific dependencies pollute a generic core package.
- README lacks minimal runnable examples.
- Review gives vague advice without suggested API shapes.
- Recommended changes ignore breaking change risk.

## Litmus Checks

- Can a user understand the middleware's responsibility in one sentence?
- Can the default usage fit in a short snippet?
- Are expensive features opt-in?
- Is request body handling safe and documented?
- Is response body handling safe and documented?
- Can the middleware run safely under concurrent traffic?
- Are logs and metrics safe for production?
- Are framework adapters separated when needed?
- Can users skip routes or endpoints cleanly?
- Can the package evolve without unnecessary breaking changes?
