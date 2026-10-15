# Runbook — AI Media Server Stack

This runbook covers the AI agent layer: **Stackarr** and **Hermes Agent**.

The existing production media stack is maintained separately in:

- Repository: <https://github.com/KamuiK/media-server>
- Deployment method: Portainer
- Services: Jellyfin, qBittorrent behind Gluetun, multiple Sonarr/Radarr instances, Seerr, Prowlarr, and Jackett

This repository does not deploy, upgrade, or modify those media services. Stackarr connects to them; Hermes connects only to Stackarr.

## 1. Operating principles

- GitHub is the source of truth for this repository’s configuration.
- All changes are reviewed through Git before deployment.
- Portainer deploys the Compose files in `compose/`.
- Hermes has no Docker socket, no direct media-service API access, and no NAS administrative access.
- Stackarr is the only approved MCP tool layer between Hermes and the media services.
- Stackarr starts with the `observe` authority profile.
- Real secrets are stored outside Git and are never committed.
- Every image is pinned to an exact version or digest; floating tags are not used.
- Every deployment has a documented rollback path.

Stackarr’s supported Docker installation uses a Compose file with an adjacent `.env` file for overrides, and recommends using an exact image version rather than a moving tag for reproducible deployments and rollback.

## 2. Host and deployment model

| Item | Value |
|---|---|
| NAS | Synology DS423+ |
| Docker interface | Portainer |
| Existing media-stack repository | <https://github.com/KamuiK/media-server> |
| Agent-layer repository | <https://github.com/KamuiK/ai-media-stack> |
| Default branch | `main` |
| Deployment branch | `main` |
| Stackarr Compose file | `compose/stackarr.yml` |
| Hermes Compose file | `compose/hermes.yml` |
| Stackarr environment file | `env/stackarr.env` |
| Hermes environment file | `env/hermes.env` |
| Timezone | `Europe/Vienna` |
| Administrator | `KamuiK` |

### Deployment boundaries

| Component | May access | Must not access |
|---|---|---|
| Hermes | Stackarr MCP endpoint | Docker socket, media-service APIs, NAS filesystem, SSH, SMB, DSM, Portainer |
| Stackarr | Selected media-service APIs; Docker socket for its selected-container management | Hermes secrets, LLM provider API key |
| Media services | Their own configuration and media paths | Hermes configuration or credentials |

Stackarr requires the Docker socket to inspect and manage containers selected during setup. This is host-level access and must be limited to Stackarr; Hermes must never receive it.

## 3. Service inventory

### Stackarr

| Item | Value |
|---|---|
| Purpose | Controlled MCP tool layer for media services |
| Image | `polyphonic/stackarr:0.3.0-alpha.21` |
| Image digest | `TODO_AFTER_FIRST_DEPLOY` |
| Web UI port | `7777` |
| Published bind address | `127.0.0.1` initially |
| Config volume | `/volume1/docker/stackarr/config` → `/config` |
| Media root | `/volume1/data/media` → `/media` |
| Downloads root | `/volume1/data/torrents` → `/downloads` |
| Backup root | `/volume1/data/backup/stackarr` → `/backups` |
| Networks | `management`, `media` |
| Authority profile | `observe` |
| Docker socket | Mounted into Stackarr only; never mounted into Hermes |
| Dashboard exposure | Localhost only until authentication is configured |

Stackarr’s documented defaults are web port `7777`, bind address `127.0.0.1`, and environment variables for media, downloads, and backup roots.

### Hermes

| Item | Value |
|---|---|
| Purpose | Conversational agent using Stackarr MCP |
| Image | `nousresearch/hermes-agent:v2026.8.3` |
| Image digest | `sha256:16788311e2fa3035456bdc1bafb8ec2b1777db64ebf020af9bb7eb73c3712c9e` |
| Web/UI port | `8642` |
| Data volume | `/volume1/docker/hermes` → `/opt/data` |
| Network | `management` only |
| MCP server | Stackarr |
| LLM provider | OpenRouter |
| Default model | `TODO_CHOOSE_BEFORE_HERMES_DEPLOYMENT` |
| Docker socket | Never mounted |
| Direct media-service access | Never permitted |

Hermes stores non-secret settings in `config.yaml` and secrets such as API keys in its `.env`. In Docker, its Hermes data directory is mounted at `/opt/data`.

## 4. Networks

| Network | Purpose | Members | External exposure |
|---|---|---|---|
| `management` | Agent control plane | Stackarr, Hermes | LAN/admin access only |
| `media` | Existing media-service network | Stackarr plus existing media services | No direct WAN exposure |
| Existing Gluetun network | VPN-isolated torrent traffic | Gluetun, qBittorrent | No direct WAN exposure |

### Network rules

- Hermes joins `management` only.
- Stackarr joins `management` and `media`.
- Hermes reaches Stackarr by its Compose service name, for example `http://stackarr:7777`.
- Hermes must not be added to `media`.
- Stackarr must not be exposed publicly.
- Hermes must not be exposed publicly.
- The Stackarr dashboard remains bound to `127.0.0.1` until authentication and access controls are configured.

## 5. Paths and volumes

| Component | Host path | Container path | Backup priority | Notes |
|---|---|---|---|---|
| Stackarr configuration | `/volume1/docker/stackarr/config` | `/config` | Critical | Service selections, settings, activity data |
| Stackarr logs | `/volume1/docker/stackarr/logs` | `TODO_VERIFY_AFTER_FIRST_RUN` | High | Retain for incident review |
| Hermes data | `/volume1/docker/hermes` | `/opt/data` | Critical | `config.yaml`, `.env`, sessions, memories, state |
| Media root | `/volume1/data/media` | `/media` | No | Existing media library |
| Downloads root | `/volume1/data/torrents` | `/downloads` | No | Existing download location |
| Backup root | `/volume1/data/backup/stackarr` | `/backups` | Critical | Stackarr-managed backup destination |

Create host folders before starting Stackarr and ensure the Docker runtime can read and write them.

## 6. Credentials and secrets

| Secret | Stored in | Used by | Git-safe? |
|---|---|---|---|
| Stackarr dashboard/admin credential | NAS-side secret store or Stackarr configuration | Stackarr | No |
| Stackarr MCP credential/token | Hermes `.env` on NAS | Hermes | No |
| Media-service API keys | Stackarr configuration / NAS-side secret store | Stackarr | No |
| LLM provider API key | Hermes `.env` on NAS | Hermes | No |
| Telegram/Discord credentials | Hermes `.env` on NAS | Hermes | No |
| GitHub token, if required | NAS-side secret store or Portainer | Portainer | No |

### Rules

- `.env.example` files may be committed; real `.env` files must not be.
- Hermes receives only its LLM credentials and Stackarr MCP connection details.
- Hermes must not receive Sonarr, Radarr, Seerr, qBittorrent, Prowlarr, Jackett, or Jellyfin API keys.
- Stackarr holds the upstream media-service credentials.
- If a secret appears in Git, logs, screenshots, or agent transcripts, rotate it immediately.

Hermes distinguishes configuration from secrets: non-secret settings belong in `config.yaml`, while secrets belong in `.env`.

## 7. Backup and restore

### Backup scope

| Priority | Item | Frequency | Retention | Destination |
|---|---|---|---|---|
| Critical | Stackarr configuration and activity data | Daily | 30 daily, 12 monthly | `/volume1/data/backup/stackarr` |
| Critical | Hermes `.env` and configuration | Daily | 30 daily, 12 monthly | `/volume1/data/backup/hermes` |
| High | Hermes sessions, memories, and state database | Daily | 30 daily | `/volume1/data/backup/hermes` |
| Medium | Compose files and Git repository state | On change | Git history plus periodic export | GitHub and `/volume1/data/backup` |
| No | Media libraries and downloads | — | — | Managed by the existing media-stack workflow |

### Backup procedure

1. Confirm free space at `/volume1/data/backup`.
2. Stop Hermes before copying its SQLite state database if required.
3. Copy Stackarr configuration and Hermes configuration/state to the backup destination.
4. Verify that the backup archive opens and contains the expected files.
5. Record backup date, size, Stackarr image tag, and Hermes image tag.
6. Test a restore at least quarterly.

### Restore procedure

1. Stop the affected Stackarr or Hermes container in Portainer.
2. Restore only the affected configuration or state volume.
3. Restore the exact previous image tag or digest.
4. Start the container.
5. Validate the web UI, logs, and MCP connectivity.
6. Do not upgrade or change configuration during recovery.

## 8. Upgrade and rollback

### Pre-upgrade checklist

- [ ] Release notes and breaking changes reviewed.
- [ ] Current image tag and digest recorded.
- [ ] Configuration and state backup completed.
- [ ] Rollback image tag identified.
- [ ] Stackarr authority remains `observe`.
- [ ] Change is committed to a Git branch and reviewed.
- [ ] Maintenance window selected.

### Upgrade procedure

1. Create a branch such as `upgrade/stackarr-<old-version>-to-<new-version>`.
2. Update only the target service’s image tag and required environment/configuration.
3. Open a pull request and review the diff.
4. Merge after approval.
5. In Portainer, pull the updated Compose definition.
6. Redeploy only Stackarr or Hermes—not the existing media stack.
7. Check container logs, web UI, and MCP tool discovery.
8. Update this runbook with the new version and digest.

Stackarr recommends recording the exact image digest as well as the version tag; its registry also publishes commit-trace tags for correlating a container with a source commit.

### Rollback procedure

1. Stop the upgraded container.
2. Revert the image tag in Git to the previous pinned version.
3. Restore the pre-upgrade configuration/state backup if needed.
4. Redeploy through Portainer.
5. Validate logs, web UI, and MCP connectivity.
6. Record the failure cause before retrying.

## 9. Health checks

| Component | Check | Expected result |
|---|---|---|
| Stackarr | Web UI at `http://127.0.0.1:7777` | Dashboard loads |
| Stackarr | Container logs | No repeated authentication or mount errors |
| Stackarr | Selected service status | Configured media services visible |
| Stackarr | Agent authority | `observe` |
| Stackarr | Activity history | Hermes actions are recorded |
| Hermes | Container logs | No MCP connection errors |
| Hermes | MCP discovery | Stackarr tools are visible |
| Hermes | Read-only test | Can query service status without writing |
| Hermes | Forbidden-action test | Cannot access Docker or media services directly |

Stackarr provides a permissions audit command for checking access to its config, logs, downloads, backups, and bind mounts used by enabled services.

## 10. Incident response

| Situation | Immediate action |
|---|---|
| Hermes attempts a forbidden action | Deny it; inspect Stackarr activity history; keep authority at `observe` |
| Stackarr dashboard is publicly reachable | Remove public exposure immediately; bind to localhost; review access logs |
| API key exposed | Rotate the key; remove it from Git history/logs; restart affected service |
| Hermes can reach a media service directly | Stop Hermes; correct its network attachment before restarting |
| Stackarr becomes unavailable | Hermes loses media-stack tool access; operate media services manually |
| Configuration drift detected | Capture current state, compare with Git, reconcile through a reviewed commit |
| Backup restore fails | Do not overwrite the current state; preserve it and investigate using a separate restore target |

## 11. Change log

| Date | Change | Author | Git commit | Portainer deployment | Result |
|---|---|---|---|---|---|
| `2026-10-15` | Initial agent-layer runbook created | `KamuiK` | `<COMMIT>` | `<STACK_NAME>` | `<RESULT>` |

## References

- [Stackarr Docker installation](https://stackarr.app/docs/installation/docker)
- [Hermes Agent configuration](https://hermes-agent.nousresearch.com/docs/user-guide/configuration)
- [Hermes Agent Docker setup](https://hermes-agent.nousresearch.com/docs/user-guide/docker)