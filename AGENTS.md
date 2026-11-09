# Repository Agent Guidance

## Source of truth

GitHub is the durable source of truth for this repository. When chat history conflicts with repository state, inspect the repository and reconcile the discrepancy before making recommendations.

## Product naming

Use **Media Support** for the support product. Do not use **Stackarr** as a product or service name.

## Current baseline

- Hermes Agent v0.22.5 is deployed.
- `mcp-arr-anime` is active and covers anime Sonarr and Radarr.
- Cloudflare Tunnel is already in use.
- Telegram is enabled; WhatsApp and Discord are not currently enabled.
- Stackarr is not deployed.
- Media Support is planned, not deployed.

## Media Support boundaries

Media Support must remain read-only for media infrastructure. It must not:

- Add, approve, import, refresh, update, or delete media.
- Trigger acquisition searches or modify queues.
- Restart containers or services.
- Access Docker sockets, NAS administration, SSH, SMB, DSM, or media files.
- Receive upstream ARR credentials.
- Bypass restrictions with terminal, browser, file, or direct API tools.

## Planning rules

- Verify Hermes v0.22.5-specific capabilities from version-specific documentation and source before relying on online documentation.
- Present one architecture decision at a time when planning.
- Do not create implementation code, deployment files, work packages, or implementation prompts without explicit approval.
- Do not create Neon projects, apply migrations, or change running services without explicit approval.
- Keep secrets out of Git.
- Treat repository evidence as authoritative over chat memory.

## Documentation rules

Clearly distinguish:

- Runtime-confirmed facts
- Repository-observed configuration
- Approved planned architecture
- Unverified capabilities

Do not present planned capabilities as deployed.
