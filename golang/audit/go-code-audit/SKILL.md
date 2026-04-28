---
name: go-code-audit
description: Use when the task asks to review, audit, or analyze Go code for potential vulnerabilities, security risks, performance problems, high-concurrency issues, correctness risks, and maintainability concerns. This skill produces a severity-ranked Markdown audit report under docs/ using P0/P1/P2/P3 levels and provides concrete remediation suggestions for each issue.
---

# Go Code Audit Skill

Use this skill when the user wants a structured review of Go code, especially for security, performance, concurrency, and production-readiness risks.

The goal is to identify real engineering risks, explain why they matter, rank them by severity, and provide practical improvement suggestions.

This skill is not a generic style review. It focuses on risks that can affect correctness, safety, reliability, performance, or maintainability in real Go projects.

## Working Model

Before reviewing code, determine five things:

- project type: library, SDK, middleware, CLI, web service, API service, worker, or unknown
- runtime context: single-user tool, public HTTP service, internal service, high-concurrency service, or unknown
- critical surfaces: HTTP handlers, middleware, authentication, file operations, network calls, goroutines, shared state, database access, serialization, or external commands
- review scope: changed files only, selected package, whole repository, or specific risk category
- report target: create a new Markdown report under `docs/` or update an existing audit report

If the project type is unknown, assume a general Go project and avoid framework-specific assumptions.

If the runtime context is unknown, assume the code may run in production and may receive untrusted input.

If the user provides a repository or files, review the provided code first before making general recommendations.

## Default Output

Create or propose a Markdown report under:

```txt
docs/code-audit.md
```

If the user wants date-based reports, use:

```txt
docs/audits/YYYY-MM-DD-code-audit.md
```

If the repository already has a docs structure, preserve it and add the report in the closest suitable location.

Do not overwrite an existing audit report unless the user explicitly asks.

## Severity Model

Use P0/P1/P2/P3 severity levels.

### P0 - Critical

Use P0 for issues that can directly cause severe production impact or security compromise.

Examples:

- remote code execution
- authentication or authorization bypass
- leaking secrets, tokens, credentials, private keys, or sensitive user data
- SQL injection, command injection, path traversal with write/read impact
- arbitrary file write or delete
- panic or crash reachable by untrusted input in a critical service
- data corruption in critical paths
- deadlock or goroutine leak that can fully take down the service under normal traffic

### P1 - High

Use P1 for serious issues that can cause major security, reliability, or performance impact.

Examples:

- missing input validation on exposed endpoints
- unsafe file path handling
- unbounded request body reads
- missing timeouts on network calls
- unbounded goroutine creation
- race-prone shared state
- serious resource leaks
- high-concurrency bottlenecks that can degrade the whole service
- weak cryptographic usage
- missing permission checks in sensitive operations
- dependency or configuration risk that can become exploitable

### P2 - Medium

Use P2 for issues that should be fixed but are not immediately critical.

Examples:

- inefficient algorithms in moderately hot paths
- avoidable allocations
- missing context cancellation propagation
- unclear error handling that can hide failures
- missing rate limiting on non-critical endpoints
- weak observability for important operations
- inconsistent API error responses
- missing tests around risky logic
- maintainability issues that increase bug probability

### P3 - Low

Use P3 for minor improvements, cleanup, and hardening suggestions.

Examples:

- naming clarity
- comments for exported APIs
- small simplifications
- minor logging improvements
- small defensive checks
- documentation improvements
- non-blocking refactors
- optional lint improvements

## Required Review Categories

Review the code through these categories when relevant.

### Security

Check for:

- unsafe input handling
- SQL injection
- command injection
- path traversal
- unsafe file upload or download
- insecure temporary files
- secret leakage in logs, errors, config, or repository files
- weak authentication or authorization boundaries
- missing permission checks
- CSRF, CORS, or cookie risks in web services
- SSRF risks in user-controlled URLs
- unsafe deserialization
- insecure crypto or random generation
- unsafe use of `unsafe`
- overly broad error messages that expose internals

### Performance

Check for:

- inefficient algorithms
- repeated expensive work
- unnecessary allocations
- excessive string concatenation
- avoidable reflection
- inefficient JSON encoding or decoding paths
- unbounded memory usage
- loading large files fully into memory
- missing streaming for large payloads
- excessive logging in hot paths
- slow operations inside request handlers
- lock contention
- poor cache usage
- lack of pooling where appropriate

### Concurrency and High-Concurrency Behavior

Check for:

- data races
- unsafe shared maps or slices
- incorrect mutex usage
- deadlocks
- goroutine leaks
- channel blocking risks
- unbounded goroutine creation
- missing worker limits
- missing backpressure
- missing context cancellation
- missing request or operation timeouts
- sync.Once misuse
- WaitGroup misuse
- improper closing of channels
- shared state mutation in middleware or handlers
- high-concurrency degradation caused by locks, global state, or synchronous I/O

### Reliability

Check for:

- panic paths
- ignored errors
- swallowed errors
- fragile assumptions
- missing nil checks where realistic
- missing retry boundaries
- missing timeout boundaries
- missing graceful shutdown behavior
- resource leaks such as files, response bodies, database rows, or timers
- unclear startup failure behavior
- unsafe default configuration

### Go-Specific Correctness

Check for:

- loop variable capture issues
- defer in loops causing resource retention
- copying mutexes or types containing mutexes
- incorrect context storage
- incorrect use of pointers
- shadowed errors
- nil interface traps
- incorrect JSON tags
- time parsing and timezone mistakes
- incorrect error wrapping
- misuse of `init`
- package-level mutable global state
- misuse of `recover`
- unsafe `reflect` or `unsafe` behavior

### Maintainability

Check for:

- unclear package boundaries
- overcomplicated abstractions
- missing tests around critical logic
- duplicated risky logic
- hidden side effects
- poor error messages
- missing documentation for exported APIs
- configuration spread across unrelated places
- difficult-to-test code paths

## Report Structure

The audit report must be written as Markdown.

Use this default structure:

```md
# Go Code Audit Report

## Summary

## Scope

## Severity Overview

| Severity | Count | Meaning |
|---|---:|---|
| P0 | 0 | Critical |
| P1 | 0 | High |
| P2 | 0 | Medium |
| P3 | 0 | Low |

## Findings

### P0 - Critical

### P1 - High

### P2 - Medium

### P3 - Low

## Recommended Fix Plan

## Verification Suggestions

## Notes
```

If no issues are found in a severity level, write:

```md
No findings.
```

Do not omit a severity section.

## Finding Format

Each finding must use this structure:

```md
### P1-001: Short issue title

- Severity: P1
- Category: Security / Performance / Concurrency / Reliability / Correctness / Maintainability
- Location: `path/to/file.go:line` or `path/to/file.go`
- Status: Open
- Confidence: High / Medium / Low

#### Problem

Explain the issue clearly.

#### Impact

Explain what can happen in production.

#### Evidence

Reference the relevant code, function, behavior, or pattern.

#### Recommendation

Provide a concrete fix.

#### Suggested Change

Provide code-level guidance or a patch-style example when useful.

#### Verification

Explain how to verify the fix.
```

Use stable IDs:

```txt
P0-001
P1-001
P2-001
P3-001
```

Do not mix severity levels in the same section.

## Issue Quality Rules

Only report issues that are grounded in the reviewed code.

If an issue is speculative, mark confidence as Low and explain what needs verification.

Do not inflate severity to make the report look more serious.

Do not list generic best practices unless they apply to the code.

Do not report style-only issues as security or performance issues.

Do not claim a vulnerability exists without a plausible exploit path or failure mode.

Do not produce a massive list of tiny issues when a smaller number of high-signal findings is more useful.

## Remediation Rules

For every finding, provide an improvement suggestion.

A recommendation should be specific enough that a developer can act on it.

Good recommendations include:

- exact validation to add
- timeout value range or configuration point
- concurrency limit strategy
- safer standard library function
- safer package boundary
- error handling change
- context propagation change
- resource closing fix
- test case to add

Avoid vague recommendations such as:

```txt
Improve security.
Optimize performance.
Handle errors better.
Add tests.
```

Replace them with concrete actions.

## Verification Suggestions

The report should include practical verification steps when applicable.

Suggested verification commands:

```bash
go test ./...
go test -race ./...
go vet ./...
gofmt -w .
```

When relevant, also suggest:

```bash
go test -run TestName ./path/to/package
go test -bench=. ./...
go test -bench=. -benchmem ./...
go test -race -count=10 ./...
```

If the repository uses specific tooling, adapt the commands.

Do not require tools that are not present in the repository unless clearly marked as optional.

## Security Review Rules

For security-sensitive code, prioritize:

- authentication and authorization boundaries
- untrusted input
- file paths
- external commands
- network calls
- logging of sensitive data
- secrets and tokens
- request body size
- CORS and cookies
- SSRF-prone URL fetching
- unsafe reflection or unsafe pointer usage

When a security issue depends on deployment configuration, describe the assumption.

If the assumption is unknown, mark confidence as Medium or Low.

## Performance Review Rules

For performance-sensitive code, distinguish between:

- cold path
- startup path
- request hot path
- batch processing path
- high-concurrency path

Do not label a micro-optimization as P1 unless it affects a hot path or can cause production degradation.

Prefer explaining:

- allocation source
- lock contention source
- I/O bottleneck
- algorithmic complexity
- unbounded resource usage
- lack of streaming
- missing cache or invalid cache strategy

## Concurrency Review Rules

For high-concurrency code, inspect:

- mutable shared state
- maps accessed by multiple goroutines
- goroutines launched per request
- channels that may block forever
- missing cancellation
- missing timeouts
- locks around slow operations
- global clients and connection pools
- request-scoped data stored globally
- graceful shutdown paths

When possible, recommend:

- `context.Context` propagation
- bounded worker pools
- semaphores
- buffered channels with backpressure
- `sync.RWMutex` or `sync.Map` only when appropriate
- atomic counters for simple metrics
- timeouts and cancellation
- request size limits
- connection pool configuration

## Documentation Output Rules

When generating a report, create Markdown under `docs/`.

Default file:

```txt
docs/code-audit.md
```

For dated reports:

```txt
docs/audits/YYYY-MM-DD-code-audit.md
```

If the user asks for Chinese output, write the report in Chinese.

If the user asks for English output, write the report in English.

If the user does not specify language, match the user's language.

## Updating Existing Reports

If `docs/code-audit.md` already exists and the user asks to update it:

- preserve historical findings unless the user asks to replace the report
- add a new section for the current review
- update status only when evidence is available
- do not mark an issue fixed without checking the code

Use statuses:

```txt
Open
In Progress
Fixed
Accepted Risk
False Positive
Needs Verification
```

## Output Format in Chat

When reporting in chat, provide:

- audit report path
- severity summary
- top P0/P1 findings
- next recommended fixes
- verification commands

Do not paste the entire report into chat if a downloadable or written file is created, unless the user asks.

## Hard Rules

- Always rank findings with P0/P1/P2/P3.
- Always provide a concrete recommendation for each finding.
- Always include location when available.
- Always include confidence level.
- Always create or propose a Markdown report under `docs/`.
- Do not invent issues not supported by the code.
- Do not hide uncertainty.
- Do not treat formatting-only issues as security risks.
- Do not overwrite an existing report unless requested.
- Do not require external tools unless they are optional or already present.
- Prefer actionable findings over long generic checklists.

## Reject These Failures

- Findings have no severity.
- Findings have no concrete remediation.
- Findings are generic and not tied to code.
- Security claims are made without a plausible risk.
- Performance claims ignore whether the code is on a hot path.
- Concurrency claims ignore actual shared state or goroutine behavior.
- API routes, file operations, or external commands are not reviewed when present.
- The report is only a chat summary and no `docs/` Markdown output is proposed.
- Existing audit reports are overwritten without permission.
- The review only comments on style and misses security or reliability risks.

## Litmus Checks

- Can a maintainer fix each finding without asking what to do next?
- Are P0/P1 findings truly high-impact?
- Are performance findings tied to hot paths or measurable cost?
- Are concurrency findings tied to real shared state, blocking behavior, or cancellation gaps?
- Does the report clearly distinguish evidence from speculation?
- Does the report help prioritize fixes?
- Does the report live under `docs/` and remain useful after the chat ends?
