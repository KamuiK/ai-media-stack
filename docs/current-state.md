# Media Support — Current-State Baseline

Checkpoint date: 2026-11-08
Status: Phase 2 planning; no implementation approval granted

## Purpose

Document the deployed baseline separately from the planned Media Support
architecture.

GitHub is the durable source of truth for project decisions and configuration.
Repository configuration describes intended deployment; it does not prove
runtime state. Differences must be documented and reconciled explicitly.

## Evidence and precedence

Use the following evidence categories:

- Runtime-confirmed: explicitly confirmed by the administrator.
- Repository-observed: present in repository configuration at checkpoint.
- Planned: accepted direction, not deployed.
- Unverified: requires source-code inspection or runtime validation.

When evidence conflicts, record the discrepancy. Do not silently replace
runtime facts with stale documentation.

Relevant repository paths:

- compose/hermes.yml
- compose/mcp-arr-anime.yml
- config/hermes/config.yaml
- config/hermes/config.example.yaml
- env/hermes.env.example
- README.md
- docs/runbook.md
- docs/service-map.md
- docs/safety-policy.md

## Runtime-confirmed baseline

| Item | Confirmed state |
|---|---|
| Host | Synology DS423+ |
| Deployment management | Docker / Portainer |
| Agent runtime | Hermes Agent, deployed and running |
| Hermes image | nousresearch/hermes-agent:v0.22.5 |
| Hermes digest | sha256:49b70739d2e8b4206fabf8b089f88337db73f8edaccb93e64b0836694ef10a03 |
| Active media MCP | mcp-arr-anime |
| Diagnostic integration coverage | Anime Sonarr and anime Radarr |
| Stackarr | Not deployed and not used |
| Public ingress infrastructure | Cloudflare Tunnel already in use |
| Meta developer app / WhatsApp setup | Not yet established |
| Spare production WhatsApp number | Not available |
| Neon | Already used for another application |

The administrator clarified that mcp-arr-anime is active. Earlier discussion
incorrectly described it as obsolete; that interpretation is superseded.

## Current Hermes configuration

Source: config/hermes/config.yaml, identified by the administrator as current.

| Setting | Repository-observed value |
|---|---|
| Model provider | OpenRouter |
| Default model | qwen/qwen3.8-flash |
| MCP server key | arr_anime |
| MCP URL | http://mcp-arr-anime:3000/mcp |
| MCP trust setting | full |
| MCP connection timeout | 15 seconds |
| MCP tool timeout | 30 seconds |
| MCP resources | Disabled |
| MCP prompts | Disabled |
| Terminal backend | local |
| Agent max_turns | 500 |
| Internal state database | SQLite, WAL mode |
| Telegram | Enabled |
| WhatsApp / Discord | No active enabled configuration established in the reviewed file |
| Platform toolsets | Broad platform presets configured |

Platform toolset entries do not themselves enable messaging platforms.

Environment variables, profile selection, runtime overrides, and successful
tool discovery have not been independently inspected.

## Current MCP permissions

The configured allowlist includes inspection and write-capable operations.

Examples of inspection-oriented entries:

- arr_status
- arr_search_all
- sonarr_get_series
- sonarr_get_episodes
- sonarr_get_queue
- sonarr_get_health
- radarr_get_movies
- radarr_get_queue
- radarr_get_health

Examples of acquisition or modification entries:

- sonarr_search_episode
- sonarr_search_missing
- sonarr_add_series
- sonarr_refresh_series
- radarr_search_movie
- radarr_search_movies
- radarr_add_movie
- radarr_refresh_movie
- radarr_update_movie
- radarr_delete_queue_item

Therefore, the current configured MCP surface is not strictly read-only.

Tool names alone are not sufficient evidence of behavior. Verify implementation
and discovered schemas before approving the support allowlist.

The meaning and security effects of trust: full versus trust: untrusted must
be verified for the pinned Hermes version. Trust configuration is not a
substitute for restricting write-capable tools.

## Repository/deployment discrepancies

The reviewed documentation and templates contain stale assumptions:

- Stackarr is described as active despite not being deployed.
- Some documentation describes Docker socket access and approval-gated writes.
- Documented Hermes image versions differ from the confirmed deployed image.
- Network-isolation descriptions differ from the reviewed Compose definition.
- config.example.yaml and environment examples contain obsolete Stackarr
  references.
- The existing MCP Compose definition uses a floating image tag.
- Current config.yaml contains a broader tool surface than the planned
  read-only support MVP permits.

These discrepancies must be corrected during the documentation reset.

Do not remove the active mcp-arr-anime service as part of documentation cleanup.

## Planned, not deployed

The following are target architecture decisions, not current capabilities:

- Media Support product.
- Support logic inside the Hermes container.
- Separate private plugin source repository.
- Separate Neon project for support data.
- WhatsApp Business Cloud API integration.
- Private Discord ticket channel and one thread per ticket.
- General and Asian MCP instances.
- Read-only support tool restrictions.
- Discord-managed user authorization.
- Durable resolution notifications and reconciliation.

See phase-2-decisions.md for details.

## Verification still required

- Running MCP image tag and digest.
- Actual discovered MCP tools and their side effects.
- Version-specific Hermes plugin, command, hook, and messaging APIs.
- WhatsApp Cloud API support in the pinned Hermes build.
- Programmatic Discord ticket-thread creation.
- Runtime network membership, mounts, and effective tool exposure.
- How plugin installation and dependencies persist across redeployments.
- How credentials are supplied without exposing them to support-user tools.
- How the existing Telegram workflow is affected by restrictions.

Current online documentation is not proof that a feature exists in v0.22.5.

## Change boundary

This checkpoint authorizes no deployment changes.

Do not:

- Change running services.
- Create the plugin implementation.
- Create the Neon project or apply schema changes.
- Register production WhatsApp infrastructure.
- Create deployment files or implementation work packages.

Complete Phase 2 documentation and obtain explicit architecture/setup approval
before implementation.