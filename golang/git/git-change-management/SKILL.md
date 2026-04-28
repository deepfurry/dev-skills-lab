---
name: git-change-management
description: Use when the task involves modifying a repository and the user wants safe Git management, frequent commits, clear checkpoints, and easy rollback. This skill enforces small reviewable commits after meaningful changes, requires status/diff inspection before committing, runs relevant validation when possible, prevents accidental secret commits, and avoids destructive Git operations unless explicitly requested.
---

# Git Change Management Skill

Use this skill when the user wants repository changes to be managed safely with Git.

The goal is to keep every meaningful change recoverable. After each logical modification, inspect the diff, validate the change when possible, commit with a clear message, and leave the repository in a state that can be rolled back easily.

This skill is not a release automation workflow. It focuses on local change safety, commit hygiene, rollback readiness, and avoiding accidental destructive operations.

## Working Model

Before making or committing changes, determine five things:

- repository state: current branch, working tree status, staged files, unstaged files, and untracked files
- change scope: bug fix, refactor, documentation, config, dependency update, feature, or experiment
- validation scope: tests, build, lint, formatting, or manual verification
- commit strategy: one commit per logical change or one final commit after a small task
- remote policy: local commit only, push after commit, or no push

If remote policy is unknown, default to local commits only.

If the working tree already contains user changes, do not overwrite or mix them into unrelated commits without reporting them first.

If the repository is not a Git repository, report that Git change management cannot be applied until a repository is initialized.

## Default Assumptions

- Preserve user work.
- Prefer small, logical commits.
- Commit only after checking `git status` and `git diff`.
- Run relevant validation before committing when practical.
- Do not push unless the user explicitly asks.
- Do not amend, rebase, reset hard, clean, force push, or delete branches unless explicitly requested.
- Do not stage unrelated files.
- Do not commit secrets, tokens, private keys, `.env` files, credentials, or local-only artifacts.
- Commit messages should be clear, short, and action-oriented.
- If a change is risky, create a checkpoint commit before deeper refactoring.

## Required Pre-Change Checks

Before modifying files, inspect the repository state.

Recommended commands:

```bash
git status --short
git branch --show-current
```

If needed, inspect existing changes:

```bash
git diff
git diff --staged
```

If there are existing user changes, report them before editing.

Do not assume all existing modifications belong to the current task.

## Required Pre-Commit Checks

Before every commit, inspect:

```bash
git status --short
git diff
git diff --staged
```

If files are already staged, verify whether they belong to the current change.

If unrelated files are present, do not include them in the commit.

If untracked files appear, decide whether they are generated artifacts, useful source files, or accidental local files.

## Secret and Sensitive File Checks

Before staging or committing, check for sensitive files and obvious secrets.

Never commit by default:

```txt
.env
.env.*
*.pem
*.key
*.crt
id_rsa
id_ed25519
*.p12
*.pfx
credentials.json
service-account.json
```

Also watch for:

- API keys
- access tokens
- refresh tokens
- passwords
- database DSNs
- private endpoints
- cloud credentials
- SSH private keys
- session cookies

If sensitive content is detected, stop and report it.

Do not commit the sensitive file.

Recommend adding safe examples such as:

```txt
.env.example
config.example.yaml
```

## Commit Granularity

Use one commit per logical change.

Good commit boundaries:

- documentation scaffold
- config update
- API behavior change
- bug fix
- test addition
- dependency update
- refactor without behavior change
- generated artifact update when intentional

Avoid commits that mix unrelated changes, such as:

```txt
docs + backend logic + formatting + dependency update
```

If many unrelated changes are needed, split them into multiple commits.

## Commit Message Rules

Use clear, conventional-style commit messages when possible.

Recommended format:

```txt
type(scope): short summary
```

Common types:

```txt
feat
fix
docs
refactor
test
chore
build
ci
perf
style
```

Examples:

```txt
docs: add code audit report
chore: add Go OSS scaffold skill
fix(api): handle missing config safely
perf(cache): reduce repeated allocations
ci: add Go test workflow
```

If the repository does not use Conventional Commits, still use a concise imperative message.

Avoid vague messages:

```txt
update
fix
changes
misc
wip
```

## Validation Rules

Run relevant validation before committing when possible.

For Go projects, prefer:

```bash
gofmt -w .
go test ./...
go vet ./...
```

For frontend projects, prefer:

```bash
npm run build
npm run lint
npm test
```

For mixed Go + frontend projects, prefer:

```bash
go test ./...
cd frontend && npm run build
```

Only run commands that are likely to exist.

If validation is skipped, explain why.

If validation fails, do not commit unless the user explicitly wants a checkpoint commit of the failing state.

## Staging Rules

Stage only files that belong to the current logical change.

Prefer explicit staging:

```bash
git add path/to/file.go path/to/README.md
```

Use `git add .` only when the working tree has been inspected and all changes belong to the same commit.

Before committing, re-check:

```bash
git status --short
git diff --staged
```

## Commit Workflow

Default workflow:

```bash
git status --short
git diff
# run relevant validation
git add <specific-files>
git diff --staged
git commit -m "type(scope): summary"
```

After committing, verify:

```bash
git status --short
git log --oneline -5
```

If the working tree is clean, report the new commit hash and summary.

If files remain modified, explain whether they were intentionally left out.

## Checkpoint Commits

Use checkpoint commits for risky or multi-step work.

Good checkpoint situations:

- before a large refactor
- after completing a working intermediate state
- before dependency upgrades
- before changing project layout
- before editing generated or complex files
- before applying automated formatting across many files

Checkpoint commit messages may use:

```txt
chore: checkpoint before refactor
chore: checkpoint working scaffold
```

Do not create noisy checkpoint commits for tiny changes unless rollback safety matters.

## Rollback Guidance

After committing, provide simple rollback guidance when useful.

For local rollback of the latest commit while keeping changes:

```bash
git reset --soft HEAD~1
```

For local rollback of the latest commit and unstaging changes:

```bash
git reset --mixed HEAD~1
```

For reverting a committed change safely:

```bash
git revert <commit>
```

Avoid recommending destructive rollback by default.

Do not run:

```bash
git reset --hard
git clean -fd
```

unless the user explicitly asks and understands that local changes may be lost.

## Branch Rules

If the task is risky or larger than a small edit, recommend working on a feature branch.

Recommended branch naming:

```txt
feat/<short-name>
fix/<short-name>
docs/<short-name>
chore/<short-name>
audit/<short-name>
```

Create a branch only when requested or when the user asks for the full workflow.

Do not switch branches if there are uncommitted changes unless those changes are handled safely first.

## Push Rules

Do not push by default.

Only push when the user explicitly asks.

Before pushing, verify:

```bash
git status --short
git log --oneline origin/$(git branch --show-current)..HEAD
```

If upstream is not configured, explain the needed command.

Do not force push unless explicitly requested.

If force push is requested, prefer:

```bash
git push --force-with-lease
```

Never use plain force push by default.

## Existing User Changes

If existing user changes are present before the task:

- identify them
- avoid overwriting them
- avoid staging them into unrelated commits
- ask or clearly state a safe assumption when necessary
- use explicit paths when staging

If a file must be edited but already has user changes, preserve the user changes and report the overlap.

## Generated Files

Generated files should only be committed when they are intentionally part of the repository.

Examples that may be committed when required:

- generated docs
- embedded frontend build output when required by Go embed
- lock files
- generated code required for builds

Examples that should usually not be committed:

- local binaries
- temporary files
- caches
- logs
- local IDE files
- local environment files

If unsure, inspect `.gitignore` and project conventions.

## Dependency Changes

When dependency files change, include the related lock file.

Examples:

- `go.mod` with `go.sum`
- `package.json` with `package-lock.json`
- `pnpm-lock.yaml`
- `yarn.lock`

Do not commit dependency changes that were not required by the task.

If a dependency change appears unexpectedly, report it.

## Report Format in Chat

After a commit, report:

```txt
Committed: <short-hash> <message>
Files included:
- path/to/file
- path/to/other-file

Validation:
- go test ./...: passed
- npm run build: skipped, not applicable

Rollback:
- git revert <short-hash>
```

If no commit was made, report why.

## Optional Markdown Log

If the user wants a persistent Git activity log, create or update:

```txt
docs/git-change-log.md
```

Use this structure:

```md
# Git Change Log

## YYYY-MM-DD

### <commit-hash> <commit-message>

- Summary:
- Files:
- Validation:
- Rollback:
```

Do not create this log by default unless the user asks.

## Hard Rules

- Always inspect Git status before committing.
- Always inspect staged diff before committing.
- Commit only logical, related changes together.
- Do not stage unrelated files.
- Do not commit obvious secrets or local environment files.
- Do not push unless explicitly requested.
- Do not use destructive commands unless explicitly requested.
- Do not overwrite user changes.
- Do not amend or rebase unless explicitly requested.
- Provide rollback guidance for created commits.
- Report validation results or explain why validation was skipped.

## Reject These Failures

- Commit made without checking status.
- Commit made with unrelated files.
- Commit made with `.env`, private keys, tokens, or credentials.
- Commit message is vague, such as `update` or `fix`.
- Generated binaries or caches are committed accidentally.
- Existing user changes are overwritten.
- Validation failure is ignored.
- Push is performed without explicit request.
- `reset --hard`, `clean -fd`, rebase, or force push is used without explicit request.
- Rollback path is not provided after a commit.

## Litmus Checks

- Can the user easily identify what changed in each commit?
- Can the latest change be rolled back safely?
- Are unrelated changes kept out of the commit?
- Were tests or relevant validation run when practical?
- Are secrets and local-only files excluded?
- Does the commit history remain useful for debugging?
- Can the repository return to a known good state quickly?
