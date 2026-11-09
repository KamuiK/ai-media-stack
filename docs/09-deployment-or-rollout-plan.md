# Media Support — Deployment and Rollout Plan

Status: Approved Phase 2 planning; no deployment change authorized

## Deployment principle

Media Support runs inside the existing Hermes container. It does not introduce a separate support container for the MVP.

## Production pins

| Component | Pin type |
|---|---|
| Hermes | Exact image digest |
| MCP images | Exact image digest |
| Media Support plugin | Full commit SHA |
| Plugin dependencies | Locked versions |
| Neon project | Dedicated Media Support project |

Do not use floating tags in production.

## Rollout stages

1. Create the private plugin repository and initial documentation.
2. Implement and test plugin logic locally.
3. Validate the initial Neon migration on a disposable Neon branch.
4. Install the pinned plugin revision into Hermes.
5. Configure the WhatsApp test number and Cloudflare Tunnel webhook route.
6. Configure the private Discord test channel.
7. Restrict Hermes to verified read-only MCP tools.
8. Run the full acceptance suite.
9. Obtain explicit production-onboarding approval.
10. Allow the first real user and monitor closely.

## Webhook ingress

Use a dedicated hostname:

```text
https://media-support-webhook.<domain>/webhook
```

Requirements:

- Cloudflare Tunnel routes only the required webhook path.
- Meta webhook signature validation is mandatory.
- Only `GET` verification and `POST` event delivery are exposed.
- Hermes dashboard, API, MCP, and other services remain private.
- Secrets are supplied through Hermes environment configuration, never Git.

## Allowlist synchronization

```text
Discord admin command
  -> plugin validates Discord identity
  -> Neon updates user and audit event
  -> plugin renders canonical Hermes allowlist
  -> controlled Hermes configuration update/restart
  -> verify effective state
  -> mark synchronized or sync_failed
```

A revoked user must be denied Media Support actions even if Hermes has not yet applied the native allowlist change. The plugin must alert administrators to any synchronization mismatch.

## WhatsApp window

- Send immediately while the customer-service window is open.
- Store `pending_user_reply` after expiry.
- Deliver pending updates on the user’s next inbound message.
- Do not promise proactive post-window notifications for the MVP.
- Use a Hermes script-only cron job for reconciliation and bounded retries.

## Backup and recovery

| Component | Recovery approach |
|---|---|
| Neon | Managed snapshots and point-in-time recovery where available |
| Plugin source | Private GitHub repository and pinned release |
| Hermes configuration | Version-controlled non-secret configuration; external secrets |
| Discord references | Stored in Neon; Discord content may not be recoverable |
| WhatsApp delivery state | Stored in Neon |

## Rollback

- Revert the plugin to the previous pinned commit.
- Keep database migrations forward-only.
- Use a compensating migration for schema repairs.
- Verify Neon, Discord, WhatsApp, and MCP health before reopening support.
- Never remove the active `mcp-arr-anime` service as part of a Media Support rollback.

## Monitoring

Monitor:

- Hermes container health
- Neon connectivity
- Discord API failures
- WhatsApp webhook reachability
- WhatsApp API credential validity
- MCP tool discovery and read-only allowlist
- Pending notification backlog
- Plugin revision mismatch
