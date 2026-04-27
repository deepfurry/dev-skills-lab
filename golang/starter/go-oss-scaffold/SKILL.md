---
name: go-oss-scaffold
description: Use when the task asks to initialize, standardize, or refactor a Go open-source repository structure. This skill creates a bilingual OSS layout with English and Chinese documentation, examples, GitHub Actions for Go quality checks, README badges, and basic maintenance docs while avoiding framework-specific overengineering.
---

# Go OSS Scaffold Skill

Use this skill when the user wants a clean, reusable, bilingual structure for a Go open-source repository.

The goal is not to generate business code or impose a framework. The goal is to create the repository foundation that makes a Go project easier to publish, document, test, maintain, and evolve.

## Working Model

Before generating files, determine three things:

- repository identity: owner, repository name, module path, license, and Go version
- project type: library, SDK, middleware, CLI, service template, or unknown
- scaffold scope: create missing files only, rewrite existing files, or generate a fresh structure

If the repository identity is unknown, use placeholders and clearly mark them for replacement.

If the project type is unknown, default to a lightweight Go library or utility package structure and avoid framework-specific assumptions.

## Default Assumptions

- The project is a Go open-source repository.
- English is the primary public-facing language.
- Chinese documentation lives under `docs/zh/`.
- English documentation lives under `docs/`.
- The repository has both `README.md` and `README_zh.md`.
- The two README files must contain the same badge set.
- The two README files must link to each other.
- Examples live under `examples/`.
- GitHub Actions live under `.github/workflows/`.
- The default workflow must run `gofmt`, `go vet`, and `go test`.
- The scaffold should stay minimal and practical.

## Required Repository Layout

Generate or ensure this layout:

```txt
.
├── .github/
│   └── workflows/
│       └── go.yml
├── docs/
│   ├── roadmap.md
│   ├── usage.md
│   ├── smoke-test.md
│   ├── hotfix.md
│   ├── todo.md
│   └── zh/
│       ├── roadmap.md
│       ├── usage.md
│       ├── smoke-test.md
│       ├── hotfix.md
│       └── todo.md
├── examples/
│   └── basic/
│       └── main.go
├── README.md
├── README_zh.md
├── LICENSE
├── .gitignore
└── go.mod
```

Preserve existing files unless the user explicitly asks to rewrite them.

When updating an existing repository, first report which required files are present, missing, or likely need updates.

## README Badges

Both `README.md` and `README_zh.md` must include the same badges.

Required badges:

- License
- Release
- Go Version
- Go Report Card

Default badge block:

```md
![License](https://img.shields.io/github/license/OWNER/REPO)
![Release](https://img.shields.io/github/v/release/OWNER/REPO?include_prereleases)
![Go Version](https://img.shields.io/github/go-mod/go-version/OWNER/REPO)
[![Go Report Card](https://goreportcard.com/badge/github.com/OWNER/REPO)](https://goreportcard.com/report/github.com/OWNER/REPO)
```

Replace `OWNER` and `REPO` when the repository owner and repository name are known.

Do not remove the Go Report Card badge unless the user explicitly asks.

## README Language Navigation

`README.md` must link to the Chinese README:

```md
[中文文档](./README_zh.md)
```

`README_zh.md` must link to the English README:

```md
[English](./README.md)
```

The two README files should have equivalent structure and equivalent information, not unrelated content.

## README.md Structure

Use this structure by default:

```md
# Project Name

Badge block

Language: [中文文档](./README_zh.md)

## Introduction

## Features

## Installation

## Quick Start

## Documentation

## Examples

## Project Structure

## Development

## Roadmap

## Contributing

## License
```

Keep the English README concise and public-facing.

## README_zh.md Structure

Use this structure by default:

```md
# Project Name

Badge block

语言：[English](./README.md)

## 项目简介

## 功能特性

## 安装方式

## 快速开始

## 文档

## 示例

## 项目结构

## 开发说明

## 路线图

## 参与贡献

## 开源协议
```

The Chinese README should not be a rough summary. It should mirror the English README in structure and intent.

## Documentation Layout

The English documentation directory must include:

```txt
docs/roadmap.md
docs/usage.md
docs/smoke-test.md
docs/hotfix.md
docs/todo.md
```

The Chinese documentation directory must include:

```txt
docs/zh/roadmap.md
docs/zh/usage.md
docs/zh/smoke-test.md
docs/zh/hotfix.md
docs/zh/todo.md
```

Do not put Chinese docs directly under `docs/` except for `docs/zh/`.

Do not put English docs under `docs/en/` by default.

## Documentation Content Rules

`docs/roadmap.md` should cover:

- current status
- planned milestones
- completed work
- future direction

`docs/usage.md` should cover:

- installation
- basic usage
- configuration
- common examples

`docs/smoke-test.md` should cover:

- quick verification steps
- required commands
- expected output
- common failure cases

`docs/hotfix.md` should cover:

- hotfix workflow
- branch naming
- patch versioning
- release notes
- rollback notes

`docs/todo.md` should cover:

- short-term tasks
- medium-term tasks
- long-term ideas
- known limitations

The files under `docs/zh/` must provide the same categories in Chinese.

## GitHub Actions

Create `.github/workflows/go.yml` by default.

The workflow must include:

- checkout
- setup Go
- dependency download
- gofmt check
- go vet
- go test

Default workflow:

```yaml
name: Go

on:
  push:
    branches:
      - main
      - dev
  pull_request:
    branches:
      - main
      - dev

jobs:
  test:
    name: Vet, Format and Test
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Set up Go
        uses: actions/setup-go@v5
        with:
          go-version-file: go.mod
          cache: true

      - name: Download dependencies
        run: go mod download

      - name: Check formatting
        run: |
          test -z "$(gofmt -l .)"

      - name: Run go vet
        run: go vet ./...

      - name: Run tests
        run: go test ./...
```

Do not add coverage upload, goreleaser, Docker publishing, GitHub Pages, or semantic release unless requested.

## Examples

Create `examples/` by default.

Default structure:

```txt
examples/
└── basic/
    └── main.go
```

The basic example must be minimal and runnable.

For a Go library or SDK, the example should demonstrate the smallest useful API call.

For a CLI project, prefer a short example command in `docs/usage.md` unless a runnable Go example is meaningful.

Do not create many example variants unless requested.

## .gitignore

Generate a Go-oriented `.gitignore` containing at least:

```gitignore
# Binaries
/bin/
/dist/
/build/
*.exe
*.out
*.test

# Go workspace
/vendor/

# Coverage
coverage.out
coverage.html

# Environment
.env
.env.local

# IDE
.idea/
.vscode/

# OS
.DS_Store
Thumbs.db

# Logs
*.log
```

Do not ignore `go.sum`.

## License

If the user specifies a license, use that license.

If the user does not specify a license, ask whether they prefer MIT or Apache-2.0 only when interaction is possible.

If a default must be chosen without asking, use Apache-2.0 for infrastructure, middleware, SDK, and engineering-tool repositories.

Use MIT only when the user asks for a shorter and more permissive license.

## go.mod

If the module path is known, generate:

```go
module github.com/OWNER/REPO

go 1.23
```

If the module path is unknown, generate:

```go
module example.com/project-name

go 1.23
```

and clearly mention that it must be replaced.

## Generation Rules

- Start with repository structure, then documentation, then automation.
- Keep the scaffold minimal.
- Prefer files that help users understand, test, and maintain the project.
- Keep English and Chinese documentation synchronized in structure.
- Keep badges identical across both README files.
- Use placeholders only when necessary.
- Make placeholder values obvious: `OWNER`, `REPO`, `PROJECT_NAME`, `MODULE_PATH`.
- Do not generate source package architecture unless the user asks.
- Do not assume the project is a web service.
- Do not assume Fiber, Gin, Cobra, GORM, Redis, Docker, or Kubernetes.
- Do not create empty `internal/`, `pkg/`, or `cmd/` folders by default.

## Output Format

When producing files in chat, group content by file path:

```md
## File: README.md

```md
...
```

## File: README_zh.md

```md
...
```

## File: .github/workflows/go.yml

```yaml
...
```
```

When many files are required, first show the tree and the most important files. Then continue with the rest if the user asks.

When generating an artifact, create a downloadable `SKILL.md` or repository zip when requested.

## Verification

After generating or updating the scaffold, provide local verification commands:

```bash
go mod tidy
gofmt -w .
go vet ./...
go test ./...
```

If GitHub Actions is generated, mention that the same checks run on push and pull request.

## Hard Rules

- Keep `docs/` for English documentation.
- Keep `docs/zh/` for Chinese documentation.
- Keep `README.md` and `README_zh.md` mutually linked.
- Keep the same badge set in both README files.
- Include License, Release, Go Version, and Go Report Card badges.
- Include `.github/workflows/go.yml` with `gofmt`, `go vet`, and `go test`.
- Preserve existing files unless rewrite is requested.
- Do not ignore `go.sum`.

## Reject These Failures

- README files with different badges
- English-only documentation when Chinese docs are requested
- Chinese docs mixed into the English docs root
- GitHub Actions that run tests but skip formatting
- workflows that require tools not declared in the repository
- overbuilt scaffolds with Docker, Kubernetes, database, or frontend files by default
- empty architectural folders with no purpose
- placeholder owner or repo values hidden in badge URLs without warning

## Litmus Checks

- Can a new visitor understand the project from either README?
- Can a maintainer find roadmap, usage, smoke test, hotfix, and todo docs quickly?
- Can CI catch formatting, vet, and test failures?
- Are English and Chinese documentation paths predictable?
- Is the scaffold useful without forcing a framework or architecture?
- Are all placeholders obvious and easy to replace?
