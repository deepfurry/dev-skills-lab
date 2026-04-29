---
name: github-repo-polish
description: Use when the task asks to polish, prepare, or review a GitHub repository before public release, open-source publishing, portfolio display, promotion, or first stable tag. This skill focuses on README quality, badges, repository metadata, topics, license, CI status, examples, documentation, release readiness, security hygiene, and overall first-impression quality.
---

# GitHub Repository Polish Skill

Use this skill when the user wants to prepare a GitHub repository for public release, open-source publishing, portfolio display, promotion, or a stable tag.

The goal is to make the repository clear, trustworthy, runnable, safe to publish, and easy for new visitors to understand.

This skill is not a code refactoring workflow, not a security audit, and not a release automation tool. It focuses on repository presentation, documentation quality, metadata, badges, examples, CI, license clarity, release readiness, and publication hygiene.

## Working Model

Before polishing or reviewing a repository, determine five things:

- repository type: Go library, Go middleware, CLI, web app, frontend app, documentation repository, template, or unknown
- release stage: pre-release, first public release, stable release, portfolio polish, maintenance update, or unknown
- target audience: developers, operators, users, contributors, recruiters, or unknown
- polish scope: README only, metadata only, docs, CI, examples, release readiness, security hygiene, or full repository polish
- output target: chat checklist, direct file edits, `docs/repo-polish.md`, or dated report under `docs/reviews/`

If the repository type is unknown, use a general GitHub open-source repository checklist.

If the release stage is unknown, assume the repository is being prepared for public viewing but not necessarily for a stable release.

If the user does not explicitly ask to publish, tag, push, or create a release, do not perform those actions.

## Default Assumptions

- The repository is being prepared for public display.
- The user wants better first impression and maintainability.
- Do not rewrite project code by default.
- Do not redesign public APIs by default.
- Do not publish releases by default.
- Do not push commits or tags by default.
- Prefer actionable recommendations over praise.
- Preserve useful existing README and documentation content.
- Match the repository's actual maturity instead of making it look more mature than it is.

## Repository First Impression

A new visitor should understand the repository quickly.

The first screen of the repository should answer:

- What is this project?
- Who is it for?
- Why is it useful?
- How do I try it quickly?
- Is it maintained or experimental?
- What license does it use?

Good first impression signals:

- clear repository description
- readable README title and summary
- working badges
- quick start near the top
- minimal runnable example
- visible license
- passing CI or clear status
- examples and docs easy to find

Avoid:

- vague description
- README starts with long background story
- no installation or quick start
- broken badges
- too many badges
- outdated TODO-heavy text
- no license
- no examples
- hidden setup assumptions

## README Review Rules

README is the primary entry point.

A polished README should include:

- project name
- short description
- badges
- language switch if bilingual README files exist
- introduction
- features
- installation
- quick start
- minimal runnable example
- configuration
- documentation links
- examples links
- development notes
- roadmap or project status
- license

For small projects, keep the README concise.

For libraries, SDKs, middleware, and tools, the README must include a minimal usage example.

For bilingual repositories, keep `README.md` and `README_zh.md` structurally aligned.

The first practical example should appear before advanced explanations.

Avoid:

- pure marketing copy
- missing quick start
- outdated screenshots
- broken internal links
- badges pointing to the wrong repository
- examples that require private infrastructure
- claims that are not supported by the repository

## Badge Rules

Badges should be truthful, useful, and working.

Common badges for Go repositories:

```md
![License](https://img.shields.io/github/license/OWNER/REPO)
![Release](https://img.shields.io/github/v/release/OWNER/REPO?include_prereleases)
![Go Version](https://img.shields.io/github/go-mod/go-version/OWNER/REPO)
[![Go Report Card](https://goreportcard.com/badge/github.com/OWNER/REPO)](https://goreportcard.com/report/github.com/OWNER/REPO)
```

If CI exists:

```md
![CI](https://github.com/OWNER/REPO/actions/workflows/go.yml/badge.svg)
```

Rules:

- Replace `OWNER` and `REPO` with real values when known.
- Do not add badges that point to non-existent workflows.
- Do not add coverage badges unless coverage upload is configured.
- Do not add misleading stability badges.
- Do not add too many decorative badges.
- Keep badge sets consistent across bilingual README files.
- If a badge is broken, fix it or remove it.

## GitHub Metadata Rules

Review GitHub repository metadata.

Check:

- About description
- website URL
- topics
- repository visibility
- default branch
- releases
- packages if relevant
- README rendering
- social preview image if relevant

Good description style:

```txt
A lightweight Go middleware for ...
A minimal Go SDK for ...
A bilingual offline operations handbook for ...
A Vue + Go tool for ...
```

Avoid vague descriptions:

```txt
A simple tool.
My demo.
Useful project.
Powerful framework.
```

Topic rules:

- use 5 to 12 relevant topics when possible
- include language
- include framework when relevant
- include project type
- include core problem domain
- avoid unrelated trending topics

Example topics for a Go middleware repository:

```txt
go
golang
middleware
fiber
http
web-security
waf
observability
open-source
```

Do not invent metadata values. If metadata cannot be changed directly, provide recommended values.

## License Review Rules

A public repository should have a clear license.

Check:

- `LICENSE` file exists
- README mentions the license
- license badge works
- license text matches the stated license
- there are no conflicting license claims

Rules:

- If the user already chose a license, preserve it.
- Do not change license without explicit instruction.
- For Go libraries, SDKs, and middleware, MIT or Apache-2.0 are common choices.
- Do not rely only on README text without a `LICENSE` file.
- Do not mix multiple license names unless the project intentionally uses dual licensing.

If license is missing, recommend adding one before public release.

## CI and Workflow Review Rules

Public repositories should have basic validation.

For Go projects, check for:

- `gofmt`
- `go vet`
- `go test`
- `go build` when useful

For frontend projects, check for:

- `npm ci`
- `npm run build`
- `npm run lint` when script exists
- `npm test` when script exists

For Go + frontend projects, check for:

- frontend install
- frontend build
- Go tests
- Go build

Avoid:

- workflow badges that point to missing workflows
- workflows that call scripts that do not exist
- CI that only builds but never tests
- overly heavy CI for a small repository
- failing CI ignored in release readiness

Do not add deployment or release automation unless requested.

## Examples Review Rules

Examples are critical for adoption.

Check:

- `examples/basic` exists when appropriate
- the basic example is minimal and runnable
- README example matches actual code
- examples do not rely on private credentials
- examples do not require unavailable infrastructure unless documented
- examples include useful comments
- examples are not overly complex

Recommended examples for Go libraries:

```txt
examples/basic/main.go
examples/with-config/main.go
```

Recommended examples for Go middleware:

```txt
examples/basic/main.go
examples/skip-paths/main.go
examples/error-handler/main.go
```

Recommended examples for SDKs:

```txt
examples/basic/main.go
examples/upload/main.go
examples/download/main.go
```

Only recommend examples that fit the repository type.

## Documentation Review Rules

Documentation depth should match repository maturity.

For small repositories, README plus examples may be enough.

For more formal open-source repositories, consider:

```txt
docs/
docs/zh/
docs/usage.md
docs/roadmap.md
docs/smoke-test.md
docs/hotfix.md
docs/todo.md
CHANGELOG.md
CONTRIBUTING.md
SECURITY.md
```

Rules:

- Do not force heavy docs onto tiny repositories.
- Keep docs easy to find.
- Keep README and docs links valid.
- For bilingual repositories, keep English docs under `docs/` and Chinese docs under `docs/zh/`.
- Keep README.md and README_zh.md equivalent in structure when both exist.
- Do not leave stale TODO text in public-facing docs unless clearly marked as roadmap.

## Release Readiness Rules

Before recommending a release, check:

- CI status is known
- examples are runnable
- README quick start works
- license exists
- changelog or release notes are prepared
- version number is appropriate
- sensitive files are absent
- known breaking changes are documented
- release notes are specific

For first public release:

- use `v0.1.0` when the project is usable but API may still change
- use `v1.0.0` only when API and behavior are stable enough
- do not overstate maturity

Suggested release notes structure:

```md
## Highlights

## Added

## Changed

## Fixed

## Documentation

## Migration Notes
```

Do not create tags, releases, or push changes unless explicitly requested.

## Security and Hygiene Review Rules

Before public release, check for sensitive files and local artifacts.

Watch for:

```txt
.env
.env.*
*.pem
*.key
*.crt
*.p12
*.pfx
credentials.json
service-account.json
id_rsa
id_ed25519
*.log
*.db
*.sqlite
node_modules/
tmp/
.cache/
```

Also watch for:

- API keys
- tokens
- passwords
- cloud credentials
- private endpoints
- session cookies
- local debug dumps
- local binaries
- large accidental files
- IDE-only files

Rules:

- If secrets are found, stop and report them.
- Do not recommend publishing while secrets are present.
- Do not remove files automatically unless asked.
- Do not assume every generated file is unwanted.
- Some generated artifacts may be intentional, such as embedded frontend `web/dist/` for Go embed projects.
- Check `.gitignore` for local artifacts, logs, caches, binaries, and environment files.

## Community Health Files

Suggest community files when useful, but do not force them.

Optional files:

```txt
CONTRIBUTING.md
SECURITY.md
CODE_OF_CONDUCT.md
.github/ISSUE_TEMPLATE/
.github/pull_request_template.md
```

Guidelines:

- `SECURITY.md` is important for security-sensitive projects.
- `CONTRIBUTING.md` is useful when external contributions are expected.
- PR templates are useful for multi-maintainer projects.
- Small personal projects do not need every community file at once.

## Repository Structure Review

Review whether important files are easy to find.

Check:

- README exists
- license exists
- examples are visible
- docs are visible
- source layout is understandable
- no duplicate or stale docs
- no unnecessary deep nesting
- no committed local binaries or caches
- no unrelated generated files
- no broken relative links

Classify structure as:

```txt
Clean
Needs attention
Blocking before release
```

## Polish Report

When producing a persistent report, create or propose:

```txt
docs/repo-polish.md
```

For dated reports:

```txt
docs/reviews/YYYY-MM-DD-repo-polish.md
```

Default report structure:

```md
# GitHub Repository Polish Report

## Summary

## Scope

## Repository Identity

## First Impression

## README Review

## Badges Review

## GitHub Metadata Review

## License Review

## CI Review

## Examples Review

## Documentation Review

## Release Readiness

## Security and Hygiene Review

## Recommended Changes

## Blocking Before Release

## Nice-to-have Improvements

## Next Steps
```

Use the user's language for the report unless they ask otherwise.

## Finding Levels

Use these levels for repository polish findings.

### Blocking

Use for issues that should be fixed before public release.

Examples:

- secrets or credentials are present
- license is missing for a public open-source repository
- README does not explain how to run or use the project
- CI is failing and the release claims stability
- badges point to the wrong repository
- examples are broken but advertised as working
- release notes are missing for a planned release

### Recommended

Use for important improvements that should be addressed soon.

Examples:

- README lacks quick start
- topics are missing
- repository description is vague
- examples are incomplete
- docs links are broken
- CI does not test the main use case
- changelog is missing
- bilingual README files are inconsistent

### Optional

Use for nice-to-have improvements.

Examples:

- social preview image
- extra badges
- extra examples
- issue templates
- PR template
- extra screenshots
- more detailed roadmap
- extra docs

## Finding Format

Use this format in polish reports:

```md
### Recommended-001: Short issue title

- Level: Recommended
- Category: README / Badges / Metadata / License / CI / Examples / Docs / Release / Security / Structure
- Location: `README.md`, `.github/workflows/go.yml`, repository metadata, or file path
- Confidence: High / Medium / Low

#### Problem

Explain the issue.

#### Impact

Explain how it affects first impression, usability, trust, release readiness, or safety.

#### Recommendation

Provide a concrete fix.

#### Suggested Change

Provide text, badge, metadata, command, or file-level guidance when useful.
```

## Output Format in Chat

When reporting in chat, provide:

- release-readiness summary
- Blocking findings
- top Recommended fixes
- optional polish ideas
- suggested report path
- next steps

Do not paste a very long report into chat if a file is created, unless the user asks.

## Hard Rules

- README must explain what the project does and how to run or use it.
- Do not recommend publishing if secrets are present.
- Do not recommend a stable release if CI is failing without noting it.
- Do not add misleading badges.
- Do not add coverage badges unless coverage is configured.
- Do not invent project maturity.
- Do not overwrite existing README content without preserving useful information.
- Do not remove generated files that are intentionally required.
- Do not push, tag, or publish releases unless explicitly requested.
- Always provide a clear release-readiness summary.
- Always distinguish Blocking, Recommended, and Optional improvements.

## Reject These Failures

- README has no Quick Start.
- Repository description is vague.
- Badges point to the wrong repository.
- License badge exists but `LICENSE` file is missing.
- CI badge exists but workflow does not exist.
- Topics are missing or irrelevant.
- Examples are missing or not runnable.
- Release notes are generic.
- Sensitive files are ignored during polish.
- Review only praises the repository and gives no actionable fixes.
- Stable release is recommended despite unknown or failing validation.
- Bilingual README files exist but contain inconsistent badge sets or structure.

## Litmus Checks

- Can a new visitor understand the project in 30 seconds?
- Can a developer run or use the project from README alone?
- Are badges truthful and working?
- Is the repository safe to publish?
- Does CI validate the main use case?
- Are examples minimal and runnable?
- Is the license clear?
- Are docs easy to find?
- Is release readiness honestly described?
- Are recommended changes specific enough to apply?
