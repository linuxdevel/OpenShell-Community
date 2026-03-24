# Base Sandbox

The foundational sandbox image that all other OpenShell Community sandbox images build from.

## What's Included

| Category | Tools |
|----------|-------|
| OS | Ubuntu 24.04 |
| Languages | `python3` (3.13), `node` (22.22.1) |
| Package managers | `npm` (11.11.0), `uv` (0.10.8), `pip` |
| Coding agents | `claude`, `opencode`, `codex` |
| Developer | `gh`, `git`, `vim`, `nano` |
| Networking | `ping`, `dig`, `nslookup`, `nc`, `traceroute`, `netstat`, `curl` |

### Users

| User | Purpose |
|------|---------|
| `supervisor` | Privileged process management (nologin shell) |
| `sandbox` | Unprivileged user for agent workloads (default) |

### Directory Layout

```
/sandbox/                  # Home directory (sandbox user)
  .bashrc, .profile        # Root-owned shell init, readable but not writable
  .bash_history            # Sandbox-user shell history
  .venv/                   # Root-owned seeded Python venv on PATH
  .agents/skills/          # Root-owned agent skill discovery
  .claude/skills/          # Root-owned Claude skill discovery (symlinked from .agents/skills)
  .config/opencode/        # Writable opencode home
  .config/opencode/plugins # Root-owned baked plugin tree
```

### Skills

The base image ships with the following agent skills:

| Skill | Description |
|-------|-------------|
| `github` | REST-only GitHub CLI usage guide (GraphQL is blocked in sandboxes) |

## Hardening Defaults

- `/etc/opencode/opencode.json` is the managed opencode config and is read-only to the sandbox user.
- `/sandbox/.config/opencode/plugins/` is baked into the image and locked `root:root` so the agent cannot rewrite plugin code or plugin-provided skills at runtime.
- `/sandbox/.agents/` and `/sandbox/.claude/` are locked `root:root` so baked skill instructions stay immutable.
- `/sandbox/.bashrc` and `/sandbox/.profile` are locked `root:root`; `/sandbox/.bash_history` remains writable by `sandbox`.
- `/sandbox/.venv/` is baked and locked `root:root` so `/sandbox/.venv/bin` cannot be used for PATH hijacking.

## Build

```bash
docker build -t openshell-base .
```

## Building a Sandbox on Top

Other sandbox images should use this as their base:

```dockerfile
ARG BASE_IMAGE=ghcr.io/nvidia/openshell-community/sandboxes/base:latest
FROM ${BASE_IMAGE}

# Add your sandbox-specific layers here
```

See `sandboxes/openclaw/` for an example.
