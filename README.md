# AI Media Server Stack

Controlled AI-agent layer for an existing self-hosted media server running on a Synology DS423+.

This repository contains the deployment definitions, safety policy, and operational documentation for:

- **Stackarr** — controlled MCP tool layer for the existing media services
- **Hermes Agent** — conversational AI agent that uses Stackarr through MCP

This repository does **not** deploy or modify the existing media stack.

## Architecture

```text
User
  |
  v
Hermes Agent
  |
  | MCP only
  v
Stackarr
  |
  | Controlled service APIs
  v
Jellyfin / qBittorrent / Sonarr / Radarr / Seerr / Prowlarr / Jackett
```

Hermes never communicates directly with Sonarr, Radarr, Seerr, qBittorrent, Prowlarr, Jackett, Jellyfin, Docker, Portainer, SSH, SMB, or Synology DSM.

## Upstream media stack

The production media stack is maintained in a separate repository:

- Repository: `<EXISTING_MEDIA_SERVER_REPO_URL>`
- Deployment method: Portainer
- Services: Jellyfin, qBittorrent behind Gluetun, multiple Sonarr/Radarr instances, Seerr, Prowlarr, and Jackett

Stackarr connects to those already-running services. This repository only adds the agent/control layer.

## Repository layout

```text
ai-media-stack/
├── README.md
├── .gitignore
├── docs/
│   ├── runbook.md
│   ├── service-map.md
│   └── safety-policy.md
├── env/
│   ├── stackarr.env.example
│   └── hermes.env.example
├── config/
│   ├── stackarr/
│   └── hermes/
└── compose/
    ├── stackarr.yml
    └── hermes.yml
```

## Safety model

- Hermes has no Docker socket access.
- Hermes has no raw API keys for any media service.
- Hermes connects only to Stackarr through MCP.
- Stackarr starts with `observe` authority.
- Write-capable actions require explicit user approval.
- Configuration changes are reviewed in Git and deployed through Portainer.
- Destructive, bulk, infrastructure, or irreversible actions are forbidden.

See [`docs/safety-policy.md`](docs/safety-policy.md).

## Deployment

Deployment is performed through Portainer using the Compose files in [`compose/`](compose/).

1. Create the required Docker networks and host folders.
2. Copy the `.env.example` files to the NAS-side environment location.
3. Fill in real secrets only outside Git.
4. Deploy `compose/stackarr.yml` through Portainer.
5. Validate Stackarr read-only access.
6. Deploy `compose/hermes.yml` through Portainer.
7. Validate that Hermes can discover Stackarr MCP tools.
8. Keep Stackarr authority at `observe` until explicitly changed through a reviewed change.

## Current status

| Work package | Status | Notes |
|---|---|---|
| WP0 — Runbook and safety baseline | `IN_PROGRESS` | Documentation and safety policy |
| WP1 — Deploy Stackarr | `NOT_STARTED` | Stackarr running via Portainer |
| WP2 — Connect media services | `NOT_STARTED` | Read-only service connections |
| WP3 — Hermes read-only MCP integration | `NOT_STARTED` | Hermes can query Stackarr |
| WP4 — Approval-gated actions | `NOT_STARTED` | Controlled write actions |
| WP5 — Automation policies | `NOT_STARTED` | Seeding, MAL sync, health automation |
| WP6 — Hardening and local LLM | `NOT_STARTED` | Later-phase work |