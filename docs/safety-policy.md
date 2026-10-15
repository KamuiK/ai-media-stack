# Safety Policy — Stackarr and Hermes

## Purpose

This policy defines what Hermes may do through Stackarr, which actions require explicit human approval, and which actions are permanently forbidden.

Stackarr is the only approved tool layer between Hermes and the media services. Hermes connects to Stackarr through MCP and must not communicate directly with Sonarr, Radarr, Seerr, qBittorrent, Prowlarr, Jackett, Jellyfin, Docker, Portainer, SSH, SMB, or Synology DSM.

Stackarr only exposes agent actions for services selected during setup, and its safety model includes authority profiles, approvals, and activity history.

## Authority model

| Authority | Hermes may perform | Current setting |
|---|---|---|
| `observe` | Read-only status, search, queue, health, log, library, and request inspection | **Initial and default** |
| `manage` | Low-impact operational actions explicitly approved by the user | Not enabled initially |
| `admin` | Infrastructure or service configuration operations | Not enabled for Hermes |
| `unrestricted` | Autonomous control without per-action approval | Never enabled |

Stackarr’s quick-start guidance recommends starting agents with the `manage` profile after initial setup and reviewing prompts before actions that can interrupt a service. This deployment is more conservative: Hermes begins at `observe` and may not be promoted without a reviewed change.

## Read-only actions

Hermes may perform these actions without approval:

- List configured media services and their health.
- Search Sonarr or Radarr for existing series and movies.
- Check whether a title is already monitored, downloaded, missing, or queued.
- Read Seerr requests and their approval status.
- Read the qBittorrent queue, transfer progress, and basic errors.
- Read Sonarr/Radarr queue, history, warnings, and health issues.
- Read Prowlarr/Jackett indexer status.
- Read Jellyfin library availability and basic server status.
- Read recent, service-scoped logs and error summaries through Stackarr.
- Analyse download, import, indexer, API, VPN, and application errors.
- Identify patterns across logs, such as repeated authentication failures, missing root folders, VPN connectivity problems, or indexer rate limits.
- Explain likely causes of failures and propose safe remediation steps.
- Generate a Git-based configuration-change proposal or Portainer deployment plan.
- Ask clarifying questions when the media type, quality expectation, or target instance is unclear.

## Approval-required actions

Hermes may propose these only after explicit user approval:

- Create a Seerr request.
- Add a series or movie to the correct Sonarr/Radarr instance.
- Change monitoring, quality profile, root folder, or tags for a specific title.
- Trigger a search or automatic import for a specific title.
- Pause, resume, prioritize, or remove a specific torrent.
- Trigger a library scan or metadata refresh.
- Restart a non-critical application-level service, where Stackarr supports it.
- Apply a seeding-policy action, such as stopping a torrent after ratio or seed-time thresholds are met.
- Trigger a Stackarr-managed backup.

Before asking for approval, Hermes must resolve and display all relevant details. The user may approve the displayed plan with `yes`. Hermes must re-confirm if the plan changes before execution.

Approval must cover:

- The exact service instance.
- The exact title, torrent, request, or action target.
- The exact action.
- The target root folder, quality profile, category, or other relevant parameters.
- Whether the action is reversible.

## Approval workflow

Approval-required actions use a confirmation dialog.

Hermes must first gather the necessary context, resolve every ambiguous value, and present a complete action plan. The user may then approve the plan with a simple `yes`, provided the displayed plan is still accurate.

Hermes must not execute an approval-required action until it has shown:

- The action to be performed.
- The exact target service and instance.
- The exact title, torrent, request, or other target.
- The destination root folder, quality profile, category/tag, monitoring scope, or other relevant parameters.
- The available options when there is more than one valid choice.
- Whether the action is reversible.
- Any warnings, conflicts, or assumptions.

### Required dialog pattern

For every approval-required request, Hermes must respond in this structure:

```text
Proposed action: <action>

Service:
- <service type>: <instance name>
- Available instances: <list relevant instances>

Target:
- <title, torrent, request, or other target>
- Available matches: <list matches, or state that none were found>

Details:
- Root folder: <selected root folder>
- Available root folders: <list valid root folders>
- Quality profile: <selected quality profile>
- Available quality profiles: <list valid quality profiles>
- Category/tag: <selected category or tag>
- Monitoring: <selected monitoring scope>
- Reversible: <yes/no>
- Warnings: <none, or relevant warnings>

Reply `yes` to execute this exact action.
Reply `change` to adjust any detail.
```

### Example

User:

```text
Add Severance in 4K
```

Hermes:

```text
Proposed action: Add TV series

Service:
- Sonarr: sonarr-tv
- Available Sonarr instances: sonarr-tv, sonarr-anime, sonarr-asian

Target:
- Series: Severance
- Available matches:
  1. Severance (2022) — TVDb ID 12345
  2. Severance: The Complete History — TVDb ID 67890

Details:
- Root folder: /media/tv
- Available root folders: /media/tv
- Quality profile: Ultra HD
- Available quality profiles: HD, Ultra HD
- Category/tag: tv-4k
- Monitoring: All seasons
- Reversible: Yes, the series can be unmonitored or removed later
- Warnings: None

Reply `yes` to add match 1 to sonarr-tv.
Reply `change` to select another match, quality profile, or monitoring scope.
```

User:

```text
yes
```

Hermes may then execute exactly the displayed action.

### Approval rules

- A simple `yes` approves only the most recently displayed, unchanged plan.
- If the user says `yes` but the available matches, service status, root folders, or quality profiles have changed, Hermes must show the updated plan and ask again.
- If Hermes cannot resolve the service, target, root folder, quality profile, or category, it must ask a focused follow-up question instead of guessing.
- If multiple plausible matches exist, Hermes must list them and require the user to choose one.
- If no valid match exists, Hermes must say so and must not create a request automatically.
- For destructive, bulk, or irreversible actions, Hermes must repeat the final confirmation after displaying the complete consequences.
- The approved plan must be recorded in Stackarr activity history.

## Forbidden actions

Hermes must never perform these actions, even if explicitly asked:

- Access or mount `/var/run/docker.sock`.
- Read Docker logs directly, tail arbitrary containers, or use the Docker socket to inspect or manage containers.
- Execute commands inside containers.
- Start, stop, recreate, delete, or modify arbitrary Docker containers, images, volumes, or networks.
- Modify Portainer stacks, Git repositories, reverse-proxy configuration, firewall rules, or Synology DSM settings.
- Read, copy, delete, rename, or move media files directly.
- Access SSH, SMB, Synology DSM, or NAS administrative interfaces.
- Access NAS-level, system, SSH, authentication, or other logs outside the approved Stackarr service interfaces.
- Send full unredacted logs, credentials, tokens, API keys, internal URLs, or personal data to an external LLM provider when a redacted relevant excerpt is sufficient.
- Receive, store, display, or transmit raw API keys for any media service.
- Modify Stackarr’s own authority level.
- Enable `admin` or `unrestricted` authority.
- Delete Sonarr/Radarr/Seerr/Prowlarr/Jackett/Jellyfin configuration databases.
- Bulk-delete media, torrents, history, requests, indexers, quality profiles, or download clients.
- Add, remove, enable, disable, or modify indexers without explicit review.
- Change VPN or Gluetun configuration.
- Bypass, edit, or delete Stackarr activity history.

## Log-analysis boundary

Hermes may analyse logs only when they are exposed through Stackarr and are relevant to a selected media service.

| Capability | Allowed? | Boundary |
|---|---|---|
| Read Sonarr/Radarr history and warnings through Stackarr | Yes | Service-scoped diagnostics only |
| Read recent qBittorrent errors through Stackarr | Yes | Helps explain stalled or failed downloads |
| Read Stackarr activity history | Yes | Required for auditing agent actions |
| Read recent application logs exposed through Stackarr | Yes | Use relevant excerpts or summaries |
| Run `docker logs` itself | No | Hermes must not have Docker access |
| Read `/var/log`, DSM logs, SSH logs, or NAS system files | No | Outside the media-stack scope and potentially sensitive |
| Upload entire unredacted logs to OpenRouter | Avoid | May expose internal URLs, tokens, or personal data |

When analysing an incident, Hermes should prefer:

1. Service health and queue status.
2. Recent error entries and warnings.
3. A relevant, redacted log excerpt.
4. A proposed diagnosis.
5. A safe, reversible remediation plan.

## MCP tool filtering

Hermes must use the smallest useful Stackarr tool surface.

Hermes supports per-server MCP tool filtering through `tools.include`, `tools.exclude`, and disabling unused resources/prompts. For a sensitive integration such as Stackarr, an allowlist is preferable to an exclusion list.

Initial Hermes MCP policy:

```yaml
mcp_servers:
  stackarr:
    url: "${STACKARR_MCP_URL}"
    tools:
      include:
        - list_services
        - get_service_status
        - search_series
        - search_movies
        - get_series
        - get_movie
        - get_queue
        - get_downloads
        - get_requests
        - get_health
        - get_logs
      resources: false
      prompts: false
```

The exact tool names must be verified against the Stackarr MCP tool list after Stackarr is deployed. If a tool name differs, update this allowlist through a reviewed Git change; do not temporarily disable filtering.

## Git and Portainer rules

- All durable configuration changes must be represented in Git.
- Hermes may draft a change, but may not merge or deploy it.
- A human reviews and approves every configuration pull request.
- Deployment occurs through Portainer from the approved Git revision.
- Image tags are pinned; floating tags are prohibited.
- Every deployment has a documented rollback image tag and configuration backup.
- Stackarr and Hermes are deployed independently from the existing media stack.

## Audit requirements

Stackarr should retain an activity-history entry for every Hermes-initiated action, including:

- Requesting user or chat channel.
- Timestamp.
- Tool/action name.
- Target service and instance.
- Input parameters.
- Approval status.
- Outcome and error message.

Hermes must not bypass, edit, or delete Stackarr activity history.

## Escalation path

1. Hermes detects an issue and reports a read-only diagnosis.
2. Hermes proposes a safe, reversible remediation.
3. The user approves or rejects the specific action.
4. If configuration is required, Hermes creates a Git proposal.
5. The user reviews, merges, and deploys through Portainer.
6. Stackarr activity history and service logs are checked afterward.

## Review triggers

Review this policy before:

- Connecting a new media service to Stackarr.
- Changing Stackarr’s authority profile.
- Adding a new MCP client.
- Enabling automatic media requests or approvals.
- Enabling seeding-policy automation.
- Enabling MAL “Plan to watch” synchronization.
- Exposing Hermes or Stackarr beyond localhost.
- Migrating Hermes to a local LLM.