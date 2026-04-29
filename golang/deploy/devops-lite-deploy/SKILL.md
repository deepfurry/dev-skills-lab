---
name: devops-lite-deploy
description: Use when the task asks to deploy, package, or prepare a lightweight production setup for a small Go service. This skill defaults to systemd + Nginx deployment, generates systemd unit files, Nginx reverse proxy blocks, .env.example, log paths, health checks, deployment directories, and rollback steps while avoiding Kubernetes, heavy CI/CD, Docker, and over-engineered infrastructure by default.
---

# DevOps Lite Deploy Skill

Use this skill when the user wants a lightweight deployment plan for a small Go service.

The goal is to make a Go service easy to deploy, restart, monitor, reverse proxy, configure, and roll back on a single Linux server.

This skill is not a Kubernetes workflow, not a complex CI/CD workflow, and not a cloud platform automation workflow. It focuses on practical single-server deployment using a Go binary, systemd, Nginx, explicit config, logs, health checks, and rollback steps.

## Working Model

Before generating or reviewing a deployment plan, determine seven things:

- service type: HTTP API, Go web app, Go embed fullstack app, CLI daemon, background worker, or unknown
- runtime user: root, dedicated service user, existing user, or unknown
- listen address: `127.0.0.1:PORT`, `0.0.0.0:PORT`, Unix socket, or unknown
- public access: behind Nginx, internal only, direct access, or unknown
- config style: `.env`, environment variables, YAML, TOML, flags, or mixed
- deployment target: single Linux server, VPS, intranet host, bare metal host, or unknown
- rollback strategy: versioned binary directory, symlink switch, backup binary, package rollback, or unknown

If the service type is unknown, assume a Go HTTP service.

If the deployment target is unknown, assume a single Linux server with systemd and Nginx.

If public access is unknown, assume the Go service should listen on `127.0.0.1` and Nginx should expose it publicly.

If rollback strategy is unknown, prefer a versioned binary with a stable symlink.

## Default Assumptions

- Deploy a compiled Go binary.
- Use systemd to manage the process.
- Use Nginx as the public reverse proxy.
- Run the service as a dedicated non-root user when possible.
- Keep runtime configuration outside the binary.
- Use `.env.example` for documented environment variables.
- Do not commit real `.env` files.
- Send logs to journald by default.
- Use an explicit logs directory when file logging is needed.
- Provide a health check endpoint.
- Provide rollback steps for every deployment plan.
- Do not default to Kubernetes.
- Do not default to Docker.
- Do not default to complex CI/CD.

## Recommended Deployment Model

Default lightweight model:

```txt
Go binary
+ .env config
+ systemd service
+ Nginx reverse proxy
+ health check
+ journald logs
+ optional file log directory
+ versioned binary rollback
```

Default service exposure:

```txt
Public Internet -> Nginx :80/:443 -> Go service 127.0.0.1:8080
```

Do not expose the Go service directly to the public network by default when Nginx is used.

## Recommended Server Layout

Prefer a versioned binary layout:

```txt
/opt/myapp/
├── bin/
│   ├── myapp-20260429-153000
│   ├── myapp-previous
│   └── myapp -> myapp-20260429-153000
├── config/
│   └── .env
├── logs/
├── releases/
├── backups/
└── tmp/
```

A simplified layout is acceptable for very small services:

```txt
/opt/myapp/
├── myapp
├── .env
└── logs/
```

Use the versioned binary + symlink layout when rollback safety matters.

Rules:

- keep binaries under `/opt/<app>/bin/`
- keep config under `/opt/<app>/config/`
- keep logs under `/opt/<app>/logs/` when file logs are used
- keep temporary files under `/opt/<app>/tmp/`
- do not write runtime files into the source repository directory
- make paths explicit in generated docs and configs

## Service User and Permissions

Do not run services as root unless explicitly required.

Recommended service user:

```bash
sudo useradd --system --home /opt/myapp --shell /usr/sbin/nologin myapp
```

Recommended directories:

```bash
sudo mkdir -p /opt/myapp/bin /opt/myapp/config /opt/myapp/logs /opt/myapp/tmp
sudo chown -R myapp:myapp /opt/myapp
sudo chmod 750 /opt/myapp
sudo chmod 750 /opt/myapp/bin
sudo chmod 750 /opt/myapp/logs
sudo chmod 750 /opt/myapp/tmp
sudo chmod 750 /opt/myapp/config
sudo chmod 640 /opt/myapp/config/.env
```

Rules:

- `.env` should not be world-readable
- binary should be executable by the service user
- logs directory should be writable by the service user
- config directory should be readable only by the service user or trusted operators
- avoid putting secrets in command-line arguments
- avoid putting secrets in systemd unit files when an `EnvironmentFile` can be used

## systemd Unit Rules

Generate a systemd unit when deploying a long-running service.

Default path:

```txt
/etc/systemd/system/myapp.service
```

Recommended template:

```ini
[Unit]
Description=MyApp Go Service
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=myapp
Group=myapp
WorkingDirectory=/opt/myapp
EnvironmentFile=/opt/myapp/config/.env
ExecStart=/opt/myapp/bin/myapp
Restart=on-failure
RestartSec=3
KillSignal=SIGTERM
TimeoutStopSec=20

# Basic hardening
NoNewPrivileges=true
PrivateTmp=true
ProtectSystem=full
ProtectHome=true
ReadWritePaths=/opt/myapp/logs /opt/myapp/tmp

# Logs go to journald by default
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```

Rules:

- set `WorkingDirectory`
- use `EnvironmentFile` for runtime config
- use a dedicated `User` and `Group`
- configure restart behavior
- configure graceful stop timeout
- use journald by default
- add `ReadWritePaths` when using systemd hardening and file logs
- do not assume root is required
- do not hide startup errors

Common commands:

```bash
sudo systemctl daemon-reload
sudo systemctl enable myapp
sudo systemctl start myapp
sudo systemctl status myapp
sudo systemctl restart myapp
journalctl -u myapp -f
```

## Nginx Reverse Proxy Rules

Use Nginx as the public reverse proxy by default for HTTP services.

Default Go service address:

```txt
127.0.0.1:8080
```

Default Nginx server block:

```nginx
server {
    listen 80;
    server_name example.com;

    location / {
        proxy_pass http://127.0.0.1:8080;

        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        proxy_connect_timeout 5s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
    }

    location /healthz {
        proxy_pass http://127.0.0.1:8080/healthz;
        access_log off;
    }
}
```

Rules:

- include standard proxy headers
- keep health check simple
- turn off access log for health endpoint when useful
- do not default to complex caching
- do not default to Cloudflare-specific configuration
- do not hardcode TLS unless the user asks
- mention Certbot or TLS setup as optional when relevant
- test Nginx config before reload

Nginx validation commands:

```bash
sudo nginx -t
sudo systemctl reload nginx
sudo systemctl status nginx
```

## .env.example Rules

Generate `.env.example`, not a real `.env`.

Default example:

```env
APP_ENV=production
APP_NAME=myapp
APP_ADDR=127.0.0.1:8080

LOG_LEVEL=info
LOG_DIR=/opt/myapp/logs

HEALTH_PATH=/healthz

# DATABASE_DSN=
# REDIS_ADDR=
```

Rules:

- `.env.example` may be committed
- `.env` must not be committed
- never include real secrets
- document important variables
- default to `127.0.0.1:8080` when behind Nginx
- avoid `0.0.0.0` unless direct network exposure is intended
- keep secrets out of systemd unit files when possible

## Logging Rules

Default logging target should be journald.

Common log commands:

```bash
journalctl -u myapp -f
journalctl -u myapp --since "1 hour ago"
journalctl -u myapp -n 200 --no-pager
```

If the application writes file logs, use explicit paths:

```txt
/opt/myapp/logs/app.log
/opt/myapp/logs/error.log
```

Recommended permissions:

```bash
sudo mkdir -p /opt/myapp/logs
sudo chown -R myapp:myapp /opt/myapp/logs
```

Optional logrotate example:

```txt
/opt/myapp/logs/*.log {
    daily
    rotate 14
    compress
    missingok
    notifempty
    copytruncate
}
```

Rules:

- prefer journald by default
- file logs must have explicit path and permissions
- do not write logs to the source repository
- do not log secrets, tokens, cookies, or credentials
- include log viewing commands in deployment docs
- include log rotation only when file logging is used

## Health Check Rules

Every deployed HTTP service should expose a health check.

Recommended endpoint:

```txt
GET /healthz
```

For API-prefixed services:

```txt
GET /api/healthz
```

Recommended response:

```json
{
  "status": "ok"
}
```

Verification commands:

```bash
curl -fsS http://127.0.0.1:8080/healthz
curl -fsS http://example.com/healthz
```

Rules:

- health check should be fast
- health check should not depend on slow external services by default
- use one simple `/healthz` for small services
- readiness and liveness can be split later when needed
- Nginx may have a separate health check location
- health endpoint access logs may be disabled

## Deployment Script Rules

When scripts are requested, generate simple shell scripts under:

```txt
deploy/scripts/
```

Recommended scripts:

```txt
deploy/scripts/deploy.sh
deploy/scripts/rollback.sh
deploy/scripts/healthcheck.sh
```

Default script behavior should be explicit and conservative.

Scripts should:

- use `set -euo pipefail`
- require app name and paths to be clearly configured
- avoid deleting files aggressively
- back up or preserve the previous binary
- switch symlink atomically with `ln -sfn`
- restart systemd
- verify health after restart
- print clear status messages

Do not generate scripts that blindly run `rm -rf` on deployment directories.

Do not overwrite production config without backup.

## Default Deployment Steps

Default manual deployment flow:

```bash
# 1. Build binary
GOOS=linux GOARCH=amd64 go build -o myapp ./cmd/server

# 2. Upload binary
scp myapp user@example.com:/tmp/myapp

# 3. Prepare directories
sudo mkdir -p /opt/myapp/bin /opt/myapp/config /opt/myapp/logs /opt/myapp/tmp

# 4. Install binary as a versioned release
sudo mv /tmp/myapp /opt/myapp/bin/myapp-20260429-153000
sudo chmod +x /opt/myapp/bin/myapp-20260429-153000
sudo ln -sfn /opt/myapp/bin/myapp-20260429-153000 /opt/myapp/bin/myapp

# 5. Fix ownership
sudo chown -R myapp:myapp /opt/myapp

# 6. Restart service
sudo systemctl daemon-reload
sudo systemctl restart myapp
sudo systemctl status myapp

# 7. Verify health
curl -fsS http://127.0.0.1:8080/healthz
```

Adapt `./cmd/server`, binary name, service name, port, and domain to the actual project.

## Rollback Rules

Every deployment plan must include rollback steps.

Preferred rollback strategy:

```bash
# 1. Check available binaries
ls -lah /opt/myapp/bin/

# 2. Switch symlink to previous binary
sudo ln -sfn /opt/myapp/bin/myapp-previous /opt/myapp/bin/myapp

# 3. Restart service
sudo systemctl restart myapp

# 4. Verify service
sudo systemctl status myapp
curl -fsS http://127.0.0.1:8080/healthz

# 5. Check logs
journalctl -u myapp -n 100 --no-pager
```

Rollback section must include:

- rollback target
- rollback command
- service restart command
- health verification command
- log inspection command
- failure handling note

Avoid rollback strategies that depend on:

- deleting directories
- overwriting the only binary copy
- `git reset --hard` on production
- rebuilding from source during emergency rollback

## Security Hardening Rules

Lightweight deployment should still use basic hardening.

Recommended defaults:

- run as dedicated service user
- listen on `127.0.0.1` behind Nginx
- keep `.env` out of Git
- restrict `.env` permissions
- use systemd hardening
- avoid secrets in command arguments
- avoid public exposure of internal ports
- use Nginx for public TLS termination when needed
- keep logs free of secrets

Optional systemd hardening:

```ini
NoNewPrivileges=true
PrivateTmp=true
ProtectSystem=full
ProtectHome=true
ReadWritePaths=/opt/myapp/logs /opt/myapp/tmp
```

Do not apply hardening directives blindly if the service needs broader filesystem access.

Explain any hardening tradeoff.

## Firewall and Port Rules

Do not expose the Go service port publicly by default.

Recommended model:

```txt
Go service: 127.0.0.1:8080
Nginx: 80/443
```

If using UFW, recommend only when relevant:

```bash
sudo ufw allow 'Nginx Full'
```

Rules:

- prefer local-only Go service when Nginx is used
- open public ports only for Nginx
- internal services may listen on private network interfaces
- do not modify firewall rules automatically unless asked

## Docker and Kubernetes Rules

Do not default to Docker.

Do not default to Kubernetes.

Only generate Dockerfile, docker-compose, Kubernetes, Helm, Terraform, or complex CI/CD when the user explicitly asks.

This skill's default deployment path is:

```txt
Go binary + systemd + Nginx
```

If Docker is requested, keep it as a separate optional path and do not replace the systemd + Nginx baseline unless the user chooses that direction.

## Generated File Layout

When generating deployment files, prefer:

```txt
deploy/
├── systemd/
│   └── myapp.service
├── nginx/
│   └── myapp.conf
├── env/
│   └── .env.example
├── scripts/
│   ├── deploy.sh
│   ├── rollback.sh
│   └── healthcheck.sh
└── README.md
```

Use actual project names when known.

If the project already has a deployment structure, preserve it and add missing files rather than replacing everything.

## Deployment Review Report

When reviewing an existing deployment plan, create or propose a Markdown report under:

```txt
docs/deploy-review.md
```

For dated reports:

```txt
docs/reviews/YYYY-MM-DD-deploy-review.md
```

Default report structure:

```md
# Lightweight Deployment Review

## Summary

## Scope

## Runtime Model

## systemd Review

## Nginx Review

## Config Review

## Logging Review

## Health Check Review

## Permission Review

## Security Hardening Review

## Rollback Plan

## Recommended Changes

## Blocking Before Deployment

## Next Steps
```

Use the user's language for the report unless they ask otherwise.

## Finding Levels

Use these levels for deployment findings.

### Blocking

Use for issues that should be fixed before deployment.

Examples:

- no rollback path
- service runs as root without reason
- `.env` contains real secrets in repository
- Go service is publicly exposed unnecessarily
- Nginx config is invalid
- health check is missing
- old binary is overwritten without backup
- config permissions expose secrets

### Recommended

Use for important improvements that should be addressed soon.

Examples:

- logs path is unclear
- systemd hardening is missing
- Nginx proxy headers are incomplete
- service restart behavior is unclear
- deployment commands are not documented
- `.env.example` is missing
- health check is not wired through Nginx
- rollback verification is missing

### Optional

Use for nice-to-have improvements.

Examples:

- logrotate
- Certbot notes
- deploy script
- rollback script
- healthcheck script
- extra Nginx timeout tuning
- basic rate limiting
- monitoring integration

## Finding Format

Use this format in deployment review reports:

```md
### Recommended-001: Short issue title

- Level: Recommended
- Category: systemd / Nginx / Config / Logging / Health Check / Permissions / Security / Rollback / Scripts / Docs
- Location: `deploy/nginx/myapp.conf`, `deploy/systemd/myapp.service`, or file path
- Confidence: High / Medium / Low

#### Problem

Explain the issue.

#### Impact

Explain how it affects deployment reliability, safety, operations, or rollback.

#### Recommendation

Provide a concrete fix.

#### Suggested Change

Provide config, command, or file-level guidance when useful.

#### Verification

Provide a command or check to verify the fix.
```

## Output Format in Chat

When reporting in chat, provide:

- deployment model summary
- generated file paths
- service user and directory assumptions
- key commands
- health check command
- rollback command
- safety notes

Do not paste very long generated files into chat if downloadable files are created, unless the user asks.

## Hard Rules

- Prefer systemd + Nginx for small Go services.
- Do not default to Kubernetes.
- Do not default to Docker.
- Do not default to complex CI/CD.
- Do not run services as root unless explicitly required.
- Go services should listen on `127.0.0.1` by default when behind Nginx.
- Always generate or request a health check endpoint.
- Always provide rollback steps.
- Always provide systemd commands.
- Always provide Nginx test and reload commands.
- Never put real secrets in `.env.example`.
- Never recommend committing `.env`.
- Do not overwrite existing production configs without backup.
- Do not expose internal service ports publicly by default.
- Do not remove old binaries during deployment unless retention policy is explicit.

## Reject These Failures

- systemd unit runs as root without explanation
- Go service listens on `0.0.0.0` by default behind Nginx
- no health check
- no rollback steps
- `.env.example` contains real secrets
- `.env` is recommended for commit
- Nginx config misses proxy headers
- deployment overwrites old binary without backup
- logs path is unclear
- permissions are world-readable for secrets
- Kubernetes is introduced by default
- Docker is introduced without user request
- complex CI/CD is introduced without user request
- Nginx config is generated without test/reload commands
- rollback depends on rebuilding source during an outage

## Litmus Checks

- Can the service restart automatically after failure?
- Can the operator check logs quickly?
- Can the operator verify health with curl?
- Can the service be rolled back within one minute?
- Are secrets kept out of the repository?
- Is the Go service protected behind Nginx?
- Are config and log paths explicit?
- Can a new server reproduce the deployment steps?
- Are old binaries preserved for rollback?
- Is the solution lightweight enough for a small Go service?
