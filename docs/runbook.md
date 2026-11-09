# Media Stack Runbook

Status: Updated for current baseline and approved Media Support planning

## Current deployed services

| Service | State |
|---|---|
| Hermes Agent | Deployed and running; image `nousresearch/hermes-agent:v0.22.5` |
| `mcp-arr-anime` | Active; covers anime Sonarr and Radarr |
| Cloudflare Tunnel | Active; existing ingress infrastructure |
| Stackarr | Not deployed and not used |
| Media Support | Planned; not deployed |

## Hermes baseline

- Image: `nousresearch/hermes-agent:v0.22.5`
- Digest: `sha256:49b70739d2e8b4206fabf8b089f88337db73f8edaccb93e64b0836694ef10a03`
- MCP server key: `arr_anime`
- MCP URL: `http://mcp-arr-anime:3000/mcp`
- Current MCP trust: `full`
- Telegram: enabled
- WhatsApp and Discord: not currently enabled

The current MCP surface includes both inspection and write-capable operations. It is not read-only.

## Media Support rollout procedure

When implementation is approved:

1. Create the private plugin repository.
2. Apply and validate the initial Neon migration on a disposable branch.
3. Install the pinned plugin revision into Hermes.
4. Configure the Meta test number and webhook route.
5. Configure the private Discord ticket channel.
6. Restrict Hermes to verified read-only MCP tools.
7. Run the acceptance suite.
8. Obtain explicit production-onboarding approval.

## Allowlist procedure

1. Administrator issues `allow` or `revoke` in Discord.
2. Plugin validates Discord identity and updates Neon.
3. Plugin renders the canonical Hermes WhatsApp allowlist.
4. Apply the controlled Hermes configuration update/restart.
5. Verify effective Hermes state.
6. Mark the Neon synchronization state accordingly.

A revoked user must be denied Media Support actions even if Hermes has not yet applied the native allowlist change.

## Ticket procedure

1. Authorized WhatsApp user reports an issue.
2. Hermes native allowlist admits the sender.
3. Plugin checks for an active duplicate.
4. User chooses to add to an existing ticket or create a new one.
5. Plugin creates/updates Neon and Discord thread.
6. Agent performs read-only diagnostics only.
7. Administrator resolves or closes the ticket in Discord.
8. Plugin updates Neon and handles the WhatsApp 24-hour window.

## Incident response

| Symptom | Action |
|---|---|
| Neon unavailable | Media Support fails closed; do not create untracked tickets |
| Discord unavailable | Mark Discord operation failed; retry only where safe |
| WhatsApp webhook unavailable | Check Cloudflare Tunnel and Hermes health |
| MCP unavailable | Report category unavailable; do not infer absence |
| Plugin/Neon allowlist mismatch | Deny Media Support actions and alert administrator |
| Suspicious webhook | Reject invalid signature; investigate without exposing secrets |

## Rollback

- Revert plugin to previous pinned commit.
- Keep migrations forward-only.
- Use a compensating migration for schema repair.
- Verify Neon, Discord, WhatsApp, and MCP health before reopening.
- Do not remove `mcp-arr-anime` during a Media Support rollback.
