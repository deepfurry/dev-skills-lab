---
name: go-embed-fullstack
description: Use when the task asks to initialize, standardize, or refactor a Go full-stack project that embeds a Vue frontend UI into the Go binary using Go embed. Defaults to Vite, Vue, TypeScript, Tailwind CSS, and Vue Router, with optional Axios, Pinia, and Element Plus for admin or console UI. This skill focuses on single-binary deployment, static frontend serving, API routing, frontend build output, and development versus production workflow.
---

# Go Embed Fullstack Skill

Use this skill when the user wants a Go project that serves a Vue frontend UI from the same Go binary.

The goal is to create a practical full-stack project foundation where the frontend is built as static assets and embedded into the Go executable using Go `embed`.

This skill does not design a large frontend architecture, does not impose a heavy backend framework, and does not generate unrelated deployment infrastructure.

## Working Model

Before generating files, determine four things:

- backend identity: module path, project name, Go version, and preferred HTTP framework if any
- frontend mode: basic UI, admin console, dashboard, or unknown
- dependency needs: whether Axios, Pinia, Element Plus, or additional Vue Router configuration is necessary
- scaffold scope: create missing files only, rewrite existing files, or generate a fresh project

If the backend framework is unknown, default to `net/http`.

If the frontend mode is unknown, default to a lightweight Vite + Vue + TypeScript + Tailwind CSS frontend with Vue Router.

If the user asks for an admin, console, dashboard, management panel, or operational UI, use Element Plus by default.

## Default Assumptions

- The project is a Go full-stack application.
- The final application should be deployable as a single Go binary.
- The frontend stack is Vite + Vue + TypeScript + Tailwind CSS + Vue Router.
- The frontend source lives under `frontend/`.
- The frontend build output lives under `web/dist/`.
- Go embeds files from `web/dist/`.
- API routes use `/api` or `/api/v1`.
- Frontend routes must not use `/api` as a client-side route prefix.
- Development mode uses Vite dev server and Go API server separately.
- Production mode serves both UI and API from the Go binary.
- Tailwind CSS should use the modern Tailwind v4 style when appropriate.
- Vue Router should be included by default for page organization and future expansion.

## Required Repository Layout

Generate or ensure this layout by default:

```txt
.
├── cmd/
│   └── server/
│       └── main.go
├── internal/
│   ├── config/
│   │   └── config.go
│   ├── server/
│   │   ├── routes.go
│   │   └── server.go
│   └── web/
│       ├── embed.go
│       └── static.go
├── frontend/
│   ├── public/
│   ├── src/
│   │   ├── router/
│   │   │   └── index.ts
│   │   ├── views/
│   │   │   ├── HomeView.vue
│   │   │   └── AboutView.vue
│   │   ├── App.vue
│   │   ├── main.ts
│   │   └── style.css
│   ├── index.html
│   ├── package.json
│   ├── tsconfig.json
│   └── vite.config.ts
├── web/
│   └── dist/
│       └── index.html
├── go.mod
├── README.md
└── .gitignore
```

Preserve existing files unless the user explicitly asks to rewrite them.

When updating an existing repository, first report which required files are present, missing, or likely need updates.

## Frontend Build Output

The frontend build must output static assets into the Go embed target directory.

Default Vite build output:

```ts
build: {
  outDir: '../web/dist',
  emptyOutDir: true,
}
```

Default command:

```bash
cd frontend
npm run build
```

After the build, the generated files must exist under:

```txt
web/dist/
```

Go must embed this directory.

Do not configure the frontend build output to a location that Go does not embed.

## Go Embed Rules

Use Go `embed` to include frontend build artifacts.

Default embedded target:

```go
//go:embed dist/*
var dist embed.FS
```

The recommended source file is:

```txt
internal/web/embed.go
```

The embedded filesystem should expose a static filesystem that the HTTP server can serve.

The project must not assume that the frontend is deployed separately in production.

## Placeholder Dist File

A fresh repository should compile predictably or clearly explain the build order.

Use one of these approaches:

1. Generate `web/dist/index.html` as a placeholder file.
2. Or clearly document that the frontend must be built before `go build`.

Prefer generating a minimal placeholder `web/dist/index.html`:

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Embedded UI Placeholder</title>
  </head>
  <body>
    <div id="app">Run frontend build to replace this placeholder.</div>
  </body>
</html>
```

The real `npm run build` output will replace this placeholder.

## API Route Rules

All backend API routes must live under:

```txt
/api
```

or:

```txt
/api/v1
```

Default health routes:

```txt
GET /api/healthz
GET /api/v1/ping
```

API routes must return API responses.

If an API route is missing, it should return a proper API 404 response, not the frontend HTML file.

## Frontend Route Rules

Use Vue Router by default.

Prefer a router mode that works reliably with embedded static serving and simple single-binary deployment.

Default router choice:

```ts
createWebHashHistory()
```

Use this default because it avoids server-side route rewriting requirements and works well when the frontend is served from a Go binary as static files.

Use `createWebHistory()` only when the user explicitly asks for clean URLs and the Go server is updated accordingly.

Frontend route examples:

```txt
/#/
/#/about
/#/settings
```

Do not use `/api` as a frontend route prefix.

## Development Mode

Development mode should run the frontend and backend separately.

Default backend development command:

```bash
go run ./cmd/server
```

Default frontend development command:

```bash
cd frontend
npm run dev
```

The Vite dev server should proxy API requests to the Go backend.

Default Vite proxy:

```ts
server: {
  host: '127.0.0.1',
  port: 5173,
  proxy: {
    '/api': {
      target: 'http://127.0.0.1:8080',
      changeOrigin: true,
    },
  },
}
```

In development mode, users should access the UI through the Vite dev server.

In production mode, users should access the UI through the Go server.

## Production Build

Production build must build the frontend before building the Go binary.

Default production sequence:

```bash
cd frontend
npm install
npm run build
cd ..
go build -o bin/app ./cmd/server
```

After building, the binary should serve:

```txt
/
```

and the API under:

```txt
/api
```

The generated binary should not require a separate frontend server.

## Frontend Stack

Default frontend stack:

- Vite
- Vue
- TypeScript
- Tailwind CSS
- Vue Router

Use Tailwind CSS v4 style when appropriate.

Default CSS entry:

```css
@import "tailwindcss";
```

Do not generate legacy Tailwind v3 directives unless the project explicitly uses Tailwind v3.

Avoid generating unnecessary UI frameworks for simple pages.

## Optional Frontend Dependencies

Add Axios when the scaffold includes frontend API calls.

Add Pinia when the app includes:

- authentication state
- user state
- theme settings
- global settings
- cross-page shared state
- dashboard filters or shared query state

Do not add Pinia for a single static page or tiny demo UI.

Vue Router is included by default.

Add additional router modules, route guards, or layout routing only when the UI needs them.

## Element Plus Rules

Use Element Plus when the user asks for:

- admin UI
- console UI
- dashboard
- management panel
- control panel
- operational workspace
- internal tools

When Element Plus is used, include:

- Element Plus
- `@element-plus/icons-vue`
- Vue Router
- Pinia when shared state is needed
- Axios when API calls are scaffolded

Do not use Element Plus by default for:

- landing pages
- blogs
- documentation sites
- visual showcases
- marketing pages
- minimal demos

## Runtime Configuration

Prefer runtime configuration from the Go backend over hardcoded frontend deployment values.

When needed, generate:

```txt
GET /api/config
```

Example response:

```json
{
  "code": 1,
  "data": {
    "appName": "PROJECT_NAME",
    "apiBase": "/api",
    "version": "dev"
  }
}
```

Use frontend environment variables only for build-time values.

Do not hardcode production API hosts into frontend code when the UI and API are served from the same Go binary.

## Static Asset Serving

Serve hashed static assets with long cache when possible.

Recommended behavior:

```txt
/assets/*    -> Cache-Control: public, max-age=31536000, immutable
/index.html  -> Cache-Control: no-cache
```

Avoid caching `index.html` aggressively.

If custom handlers are used, preserve correct content types for:

- `.html`
- `.js`
- `.css`
- `.svg`
- `.json`
- `.ico`
- `.png`
- `.jpg`
- `.webp`

Prefer standard library static file serving behavior unless custom routing logic is required.

## Subpath Deployment

Default UI base path is:

```txt
/
```

If the UI is served under a subpath such as:

```txt
/admin/
```

then configure both Vite and Go consistently.

Vite must set the correct `base`.

Go must serve the embedded UI under the same path prefix.

Do not set a Vite `base` without updating the Go routing behavior.

## Backend Framework Rules

Default to `net/http` for maximum portability.

If the user explicitly asks for Fiber, Gin, Echo, Chi, or another Go HTTP framework, adapt the static serving implementation to that framework.

Do not introduce a backend framework just to serve embedded files.

Do not assume the project uses Fiber, Gin, GORM, Redis, Docker, Kubernetes, or a database unless requested.

## GitHub Actions

If CI is requested, include a workflow that verifies both frontend and backend builds.

The workflow should include:

- checkout
- setup Go
- setup Node
- install frontend dependencies
- build frontend
- run `go test`
- run `go build`

Default checks:

```bash
cd frontend
npm ci
npm run build
cd ..
go test ./...
go build ./cmd/server
```

Do not add release automation, Docker publishing, coverage upload, or deployment steps unless requested.

## README Requirements

The README should explain:

- project purpose
- single-binary deployment model
- directory structure
- development mode
- production build
- frontend build output
- Go embed behavior
- API route prefix
- Vue Router behavior
- optional dependencies

Required command examples:

```bash
# Start Go API server
go run ./cmd/server

# Start frontend dev server
cd frontend
npm run dev

# Build frontend
cd frontend
npm run build

# Build Go binary
go build -o bin/app ./cmd/server

# Verify API
curl http://127.0.0.1:8080/api/healthz
```

## .gitignore

Generate a `.gitignore` that includes both Go and frontend artifacts.

Minimum required entries:

```gitignore
# Go binaries
/bin/
/dist/
/build/
*.exe
*.out
*.test

# Go coverage
coverage.out
coverage.html

# Frontend dependencies
frontend/node_modules/

# Frontend build cache
frontend/.vite/
frontend/dist/

# Environment
.env
.env.local
frontend/.env
frontend/.env.local

# IDE
.idea/
.vscode/

# OS
.DS_Store
Thumbs.db

# Logs
*.log
npm-debug.log*
pnpm-debug.log*
yarn-debug.log*
```

Do not ignore:

```txt
web/dist/
go.sum
frontend/package-lock.json
```

`web/dist/` may contain the placeholder and production build artifacts required by Go embed.

If the repository chooses not to commit generated frontend assets, clearly document that `npm run build` must run before `go build`.

## Generation Rules

- Start with the single-binary architecture.
- Keep backend and frontend responsibilities separate.
- Keep frontend source under `frontend/`.
- Keep embedded build artifacts under `web/dist/`.
- Use Go `embed` for production UI serving.
- Use Vue Router by default.
- Prefer Vue Router hash history for the default embedded static UI.
- Use Vite proxy for development API calls.
- Use `/api` or `/api/v1` for backend routes.
- Add optional frontend dependencies only when justified.
- Keep the scaffold minimal and runnable.
- Preserve existing files unless rewrite is requested.
- Use obvious placeholders: `PROJECT_NAME`, `MODULE_PATH`, `APP_NAME`.

## Output Format

When producing files in chat, group content by file path:

````md
## File: cmd/server/main.go

```go
...
```

## File: frontend/vite.config.ts

```ts
...
```

## File: internal/web/static.go

```go
...
```
````

When many files are required, first show the tree and the most important files. Continue with the rest only when needed.

When generating an artifact, create a downloadable `SKILL.md` or repository zip when requested.

## Verification

After generating or updating the scaffold, provide local verification commands:

```bash
cd frontend
npm install
npm run build
cd ..
go test ./...
go build -o bin/app ./cmd/server
./bin/app
```

Then verify:

```bash
curl http://127.0.0.1:8080/api/healthz
```

Also verify that the UI is available at:

```txt
http://127.0.0.1:8080/
```

And verify router pages through hash routes, for example:

```txt
http://127.0.0.1:8080/#/about
```

## Hard Rules

- Use Go `embed` to include frontend build artifacts in the Go binary.
- Default frontend stack is Vite + Vue + TypeScript + Tailwind CSS + Vue Router.
- `npm run build` must output files to `web/dist/` by default.
- Go must embed `web/dist/`.
- Frontend build must happen before production Go build.
- API routes must use `/api` or `/api/v1`.
- API routes must return API responses, not frontend HTML.
- Development mode should use Vite dev server with API proxy.
- Production mode should serve UI and API from the Go binary.
- Prefer Vue Router hash history for the default embedded static UI.
- Use Element Plus only for admin, console, dashboard, management, or operational UI.
- Add Axios only when API calls are scaffolded.
- Add Pinia only when shared state is needed.
- Do not ignore `go.sum`.
- Do not ignore `frontend/package-lock.json` when npm is used.

## Reject These Failures

- Go build fails because `web/dist/` is missing without explanation.
- Frontend builds to `frontend/dist/` while Go embeds `web/dist/`.
- API 404 responses return frontend HTML.
- Production deployment still requires a separate frontend server.
- The scaffold introduces a backend framework without user request.
- The scaffold adds Element Plus for a simple landing page or demo.
- The scaffold adds Pinia without shared state.
- The scaffold hardcodes production API hosts into frontend code.
- Static assets are served with incorrect content types.
- `web/dist/` is ignored while required by Go embed.
- Vue Router is missing from the generated frontend.

## Litmus Checks

- Can the app be built into one Go binary?
- Does `npm run build` place assets exactly where Go embeds them?
- Can the API be accessed under `/api`?
- Does Vue Router exist and organize frontend pages?
- Can frontend and backend be developed independently?
- Are optional dependencies justified by the UI type?
- Can a new developer understand the build flow from the README?
