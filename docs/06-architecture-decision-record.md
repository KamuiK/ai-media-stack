# Media Support — Architecture Decision Record

Status: Phase 2 approved for documentation; implementation approval not yet granted  
Date: 2026-11-09

## Scope

Media Support is a WhatsApp-first, read-only support assistant for the media stack. It creates persistent tickets, provides evidence-based diagnostics through approved MCP tools, and gives administrators a private Discord workflow.

It does not manage Docker, NAS, SSH, SMB, DSM, media files, or upstream ARR credentials.

## Approved decisions

| ID | Decision | Status |
|---|---|---|
| D-001 | Product name is Media Support; do not use Stackarr branding. | Accepted |
| D-002 | Media Support runs inside the existing Hermes container. | Accepted |
| D-003 | Plugin source lives in a separate private GitHub repository and is installed at a pinned full commit SHA. | Accepted; installation mechanism requires verification |
| D-004 | Media Support uses a separate Neon project. | Accepted |
| D-005 | Neon PostgreSQL is the system of record for Media Support workflow data. | Accepted |
| D-006 | Ticket IDs use yearly formatting such as `TCK-2026-0001`. | Accepted |
| D-007 | WhatsApp attachments are downloaded and uploaded to the Discord ticket thread; Neon stores metadata and references. | Accepted |
| D-008 | Discord administrators manage the WhatsApp allowlist; Neon persists access decisions. | Accepted |
| D-009 | Requester name is displayed alongside the number when available. | Accepted |
| D-010 | The existing Cloudflare Tunnel provides WhatsApp webhook ingress. | Accepted |
| D-011 | A private Discord text channel contains one thread per ticket. | Accepted |
| D-012 | The active `mcp-arr-anime` service is reused initially with verified read-only tools. | Accepted |
| D-013 | Three MCP instances are planned: general, anime, and Asian. | Accepted |
| D-014 | Category names are `general`, `anime`, and `asian`; do not use `tv`. | Accepted |
| D-015 | Route diagnostics by explicit category or verified title match; ask when ambiguous. | Accepted |
| D-016 | Pin Hermes, MCP, and plugin production revisions. | Accepted |
| D-017 | Replace stale Stackarr-era documentation. | Accepted |
| D-018 | Hermes native WhatsApp allowlist is the mandatory pre-agent authorization gate. | Accepted |
| D-019 | Neon is the authoritative Media Support access record; Discord admin commands update it and trigger controlled Hermes synchronization. | Accepted |
| D-020 | Users may create multiple concurrent tickets; duplicate prevention applies to active tickets with the same user, category, issue type, and normalized title. | Accepted |
| D-021 | Use plain, versioned, forward-only SQL migrations; validate each on a disposable Neon branch before production. | Accepted |
| D-022 | Use a dedicated Cloudflare Tunnel hostname for the WhatsApp webhook, with Meta signature validation and narrow path/method exposure. | Accepted |
| D-023 | Accept JPEG/PNG, MP4, and PDF attachments up to 20 MB; maximum three attachments per message; upload to Discord and retain metadata only in Neon. | Accepted |
| D-024 | Send WhatsApp updates immediately within the 24-hour window; store `pending_user_reply` after expiry; reconcile with a Hermes script-only cron job. | Accepted |
| D-025 | Retain support content for 90 days after closure and minimal audit/security records for 12 months; support WhatsApp-initiated deletion requests. | Accepted |
| D-026 | Use external Hermes environment secrets, pinned releases, Neon-managed recovery, forward-only migrations, and explicit rollback procedures. | Accepted |

## Component responsibilities

| Component | Responsibility |
|---|---|
| Hermes | Runtime, sessions, native allowlist, MCP client, messaging integrations where supported |
| Media Support plugin | Deterministic ticket, authorization, audit, Neon, Discord, WhatsApp, and reconciliation logic |
| Media Support skill | Conversational triage and diagnostic presentation |
| Neon | Tickets, users, messages, attachments metadata, audit events, delivery state, Discord references |
| Discord | Private administrator interface and attachment viewer |
| `mcp-arr-anime` | Initial read-only diagnostics for anime Sonarr and Radarr |
| Cloudflare Tunnel | Public ingress for the WhatsApp webhook only |

## MCP layout

| Hermes key | Service | Coverage |
|---|---|---|
| `arr_general` | `mcp-arr-general` | General TV and movies |
| `arr_anime` | `mcp-arr-anime` | Anime TV and movies |
| `arr_asian` | `mcp-arr-asian` | Asian TV and movies |

Each MCP instance handles one Sonarr and one Radarr instance. The active anime service remains in place; category instances are added rather than rewriting the upstream MCP.

## Routing rules

1. Use explicit service/category context when available.
2. Use verified read-only title lookup when only a title is known.
3. Scope `arr_search_all` to its own MCP instance.
4. Search relevant category endpoints for cross-category lookup.
5. Ask a concise question when matches are ambiguous or absent.
6. Never infer category solely from title language or genre.
7. Report an unavailable category as unavailable, not as proof that a title is absent.

## Security boundary

Media Support must not add, approve, import, refresh, update, or delete media; trigger acquisition searches; modify queues; restart services; access Docker, NAS, SSH, SMB, DSM, or media files; or receive upstream ARR credentials.

The current Hermes configuration uses `trust: full` and includes write-capable MCP tools. Before WhatsApp onboarding, Hermes must be restricted to a verified read-only tool surface. `trust: untrusted` requires approval for non-read-only MCP calls and fails closed for unattended runs, but it is not a complete security boundary by itself.

## WhatsApp workflow

1. Meta delivers a signed webhook event through Cloudflare Tunnel.
2. Hermes validates the signature and applies its native WhatsApp allowlist.
3. An unauthorized sender is rejected or ignored before LLM/tool execution.
4. The plugin validates Media Support access, records audit data, and checks for an existing active duplicate.
5. If a similar active ticket exists, the user chooses to add to it or explicitly create a new ticket.
6. A new ticket creates a Neon record and Discord thread.
7. Attachments are downloaded, uploaded to Discord, referenced in Neon, and locally deleted.
8. The agent uses only approved read-only MCP tools.
9. Admin resolution updates Neon and attempts WhatsApp delivery according to the 24-hour window.

## Open verification items

- Exact Hermes v0.22.5 plugin, hook, command, and adapter APIs.
- WhatsApp pre-agent authorization and media APIs.
- Discord thread creation, attachment upload, and trusted command identity.
- Hermes cron script-only reconciliation behavior.
- Effective MCP tool schemas and read-only enforcement.
- Hermes allowlist reload/restart semantics.

## Approval boundary

This record approves the Phase 2 architecture and documentation only. It does not authorize implementation, Neon provisioning, migrations, deployment changes, or production WhatsApp onboarding.
