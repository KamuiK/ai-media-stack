# Media Stack Safety Policy

Status: Updated for current baseline and approved Media Support planning

## Core rule

Media Support is read-only for media infrastructure. It may inspect and report; it must not modify, acquire, delete, or administer media services.

## Forbidden operations

Media Support and its plugin must not:

- Add, approve, import, refresh, update, or delete media.
- Trigger acquisition searches or modify queues.
- Restart containers or services.
- Access Docker sockets, NAS administration, SSH, SMB, DSM, or media files.
- Receive upstream ARR credentials.
- Bypass restrictions with terminal, browser, file, or direct API tools.

## MCP restrictions

Before WhatsApp onboarding:

- Verify every permitted tool’s actual behavior and schema.
- Establish an explicit read-only allowlist.
- Restrict support-user built-in tools.
- Test attempted bypasses and cross-user access.
- Review the current `trust: full` configuration.

`trust: untrusted` requires approval for non-read-only MCP calls and fails closed for unattended runs, but it is not a complete security boundary.

## Authorization

- Hermes native WhatsApp allowlist is the first authorization gate.
- Neon is the authoritative Media Support access record.
- Discord admin commands require trusted Discord event identity.
- Display names are never authorization credentials.
- Revocation must deny Media Support actions even before Hermes synchronization completes.

## WhatsApp privacy

- Do not send attachments or unredacted logs to the LLM by default.
- Do not store attachment bytes in Neon.
- Do not treat WhatsApp temporary URLs or Discord CDN URLs as permanent archives.
- Delete temporary files after successful upload or after a short retry period.
- Respect the 24-hour customer-service window.
- Do not promise proactive post-window notifications for the MVP.

## Retention

- Delete support content and attachment references 90 days after ticket closure.
- Retain minimal audit/security records for 12 months.
- Support WhatsApp-initiated deletion requests.
- Retain only the minimum audit evidence needed to document a deletion request.

## Secrets

- Never commit secrets, tokens, webhook secrets, or database URLs to Git.
- Supply secrets through Hermes environment configuration or an approved secret store.
- MCP containers retain their own upstream credentials.
- Media Support must not receive ARR credentials.

## Change control

- Pin Hermes, MCP, and plugin revisions.
- Validate migrations on disposable Neon branches.
- Use forward-only migrations and explicit rollback procedures.
- Do not change running services without explicit approval.
