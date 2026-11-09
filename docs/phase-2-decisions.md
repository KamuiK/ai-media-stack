# Media Support — Phase 2 Decision Checkpoint

Checkpoint date: 2026-11-09  
Status: Phase 2 architecture approved; implementation approval not granted

## Purpose

This document records approved Media Support decisions, technical verification gaps, and the boundary before implementation.

GitHub is the durable source of truth. Repository configuration describes intended deployment; it does not prove runtime state.

## Accepted decisions

| ID | Decision | Status |
|---|---|---|
| D-001 | Product name: Media Support; no Stackarr branding. | Accepted |
| D-002 | Run Media Support inside the existing Hermes container. | Accepted |
| D-003 | Use a separate private plugin repository installed at a pinned full commit SHA. | Accepted; installation mechanism conditional |
| D-004 | Use a separate Neon project for Media Support. | Accepted |
| D-005 | Use Neon PostgreSQL as the Media Support system of record. | Accepted |
| D-006 | Use yearly ticket IDs such as `TCK-2026-0001`. | Accepted |
| D-007 | Download WhatsApp attachments and upload them to Discord ticket threads; store metadata/references in Neon. | Accepted |
| D-008 | Manage WhatsApp allowlist through Discord admin commands backed by Neon. | Accepted |
| D-009 | Display requester name alongside number when available. | Accepted |
| D-010 | Use the existing Cloudflare Tunnel for webhook ingress. | Accepted |
| D-011 | Use a private Discord text channel with one thread per ticket. | Accepted |
| D-012 | Reuse `mcp-arr-anime` initially with verified read-only tools. | Accepted |
| D-013 | Plan three MCP instances: general, anime, and Asian. | Accepted |
| D-014 | Use category names `general`, `anime`, and `asian`. | Accepted |
| D-015 | Route by explicit category or verified title match; ask when ambiguous. | Accepted |
| D-016 | Pin Hermes, MCP, and plugin revisions. | Accepted |
| D-017 | Replace stale Stackarr-era documentation. | Accepted |
| D-018 | Hermes native WhatsApp allowlist is the mandatory pre-agent gate. | Accepted |
| D-019 | Neon is authoritative for Media Support access; Discord commands trigger controlled Hermes synchronization. | Accepted |
| D-020 | Allow multiple concurrent tickets; prevent duplicates for active user/category/issue/title combinations. | Accepted |
| D-021 | Use versioned forward-only SQL migrations validated on disposable Neon branches. | Accepted |
| D-022 | Use a dedicated Cloudflare Tunnel webhook hostname with Meta signature validation. | Accepted |
| D-023 | Accept JPEG/PNG, MP4, and PDF up to 20 MB; maximum three per message; Discord upload; Neon metadata only. | Accepted |
| D-024 | Send within the 24-hour window; persist `pending_user_reply` after expiry; reconcile with script-only cron. | Accepted |
| D-025 | Retain support content 90 days after closure and minimal audit records 12 months; support deletion requests. | Accepted |
| D-026 | Use external secrets, pinned releases, Neon recovery, forward-only migrations, and explicit rollback. | Accepted |

## Security boundary

Media Support must not:

- Add, approve, import, refresh, update, or delete media.
- Trigger acquisition searches or modify queues.
- Restart containers or services.
- Access Docker sockets, NAS administration, SSH, SMB, DSM, or media files.
- Receive upstream ARR credentials.
- Bypass restrictions through terminal, browser, file, or direct API tools.

Before WhatsApp onboarding, Hermes must use a verified read-only MCP allowlist. The current `trust: full` configuration and write-capable tool surface are not acceptable for support users.

## Open verification

- Hermes v0.22.5 plugin, hook, command, and adapter APIs.
- WhatsApp pre-agent authorization and media APIs.
- Discord thread creation, attachment upload, and command identity.
- Hermes script-only cron behavior.
- Effective MCP tool schemas and read-only enforcement.
- Hermes allowlist reload/restart semantics.

## Approval boundary

Phase 2 architecture is approved. Implementation, Neon provisioning, migrations, deployment changes, and production WhatsApp onboarding require separate explicit approval.
