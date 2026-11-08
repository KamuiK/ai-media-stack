# Media Support — Phase 2 Continuation Prompt

Use this document to continue Phase 2 planning in a new conversation.

## How to continue

In a new thread, say:

```text
Continue Phase 2 using docs/phase-2-continuation.md in KamuiK/ai-media-stack.
```

The assistant must first read:

- `docs/current-state.md`
- `docs/phase-2-decisions.md`
- `docs/phase-2-continuation.md`

It must then inspect the current repository state, especially:

- `README.md`
- `docs/`
- `compose/`
- `config/`
- `env/`
- Recent commits

GitHub is the durable source of truth. If repository state and prior
conversation context conflict, inspect the repository, identify the
discrepancy, and ask before changing anything.

## Continuation prompt

```text
Continue Phase 2 planning for the Media Support project.

Single source of truth: the GitHub repository KamuiK/ai-media-stack.

First read:
- docs/current-state.md
- docs/phase-2-decisions.md
- docs/phase-2-continuation.md

Then inspect the current repository state, especially:
- README.md
- docs/
- compose/
- config/
- env/
- recent commits

Do not rely on assumptions from prior chat history if they conflict with
GitHub. Flag and reconcile discrepancies before making recommendations.

Project context:
- Product name: Media Support.
- Do not use “Stackarr” as a product or service name.
- The currently deployed MCP is mcp-arr-anime, covering anime Sonarr and
  Radarr only.
- Hermes is deployed and running with:
  nousresearch/hermes-agent:v0.22.5
  digest: sha256:49b70739d2e8b4206fabf8b089f88337db73f8edaccb93e64b0836694ef10a03
- The current Hermes MCP configuration uses:
  http://mcp-arr-anime:3000/mcp
  trust: full
  and includes both read-only and write-capable tools.
- Telegram is enabled. WhatsApp and Discord are not currently enabled.
- The existing Neon project is volleyball-tournament-manager. Do not use it
  for Media Support.

Agreed Phase 2 decisions:
- Media Support runs inside the existing Hermes container.
- Implement it as a Hermes plugin in a separate private GitHub repository,
  installed into Hermes at a pinned full commit SHA.
- Use Neon PostgreSQL in a separate Neon project/database as the source of
  truth for Media Support tickets, users, audit events, attachment metadata,
  WhatsApp window state, and Discord references.
- Hermes retains its own SQLite internal state.
- Use yearly ticket IDs: TCK-2026-0001.
- Download WhatsApp attachments immediately and upload them to the Discord
  ticket thread; store metadata and Discord references in Neon.
- Manage the WhatsApp allowlist through Discord admin commands, persisted in
  Neon. Hermes’s native WhatsApp allowlist remains the first authorization
  gate.
- Start with Meta’s test number; defer the production WhatsApp number.
- Use the existing Cloudflare Tunnel for the WhatsApp webhook.
- Use a private Discord text channel named media-support-tickets, with one
  Hermes-created thread per ticket.
- For the MVP, admins resolve/close tickets by replying in the ticket thread.
- Restrict Hermes to verified read-only MCP tools before WhatsApp onboarding.
- Change MCP trust from full to untrusted unless a verified reason requires
  otherwise.
- Use category-based MCP routing: arr_general, arr_anime, arr_asian.
- Use category-specific title lookup when needed; never silently choose an
  MCP server.
- Use Hermes native WhatsApp Cloud API and Discord integrations where
  supported.
- Use a Hermes no-agent cron job for notification reconciliation and retries.
- Use a Hermes plugin plus skill: the plugin owns deterministic ticket,
  Neon, Discord, WhatsApp, authorization, and audit logic; the skill guides
  conversational triage.
- Rewrite stale Stackarr-era documentation to reflect the deployed baseline.

Required Phase 2 behavior:
- WhatsApp-first, read-only support assistant.
- Only explicitly allowed WhatsApp users may use it.
- Read-only diagnostics through approved MCP tools only.
- Persistent Neon-backed tickets.
- Private Discord admin workflow.
- Respect WhatsApp’s 24-hour customer-service window.
- If resolution occurs within 24 hours, send immediately.
- If the window has expired, store pending_user_reply and deliver it when the
  user next messages.
- Do not promise proactive WhatsApp notifications after the window expires,
  because Hermes does not currently support templates.
- No Docker, NAS, SSH, SMB, DSM, media-file, or upstream ARR credential
  access from Hermes or Media Support.
- Keep secrets out of Git.

Current task:
Begin with the first remaining Phase 2 decision:
1. Verify Hermes v0.22.5 extension, plugin, skill, command, hook, Discord,
   and WhatsApp capabilities from version-specific documentation and source.
2. Identify exactly which parts of the planned Media Support workflow can be
   implemented with Hermes-native capabilities.
3. Identify which parts require a plugin, a companion service, or a changed
   approach.
4. Present one decision at a time, with alternatives, tradeoffs, risks, and
   an explicit approval request.
5. Do not create implementation code, deployment files, work packages, or
   implementation prompts.
6. Do not create Neon projects, apply migrations, or change running services.

Stop after presenting the first decision and wait for my approval.
```

## Guardrails

- Do not begin implementation.
- Do not create work packages.
- Do not write implementation prompts.
- Do not create deployment files.
- Do not create Neon projects or apply migrations.
- Do not change running services.
- Do not expose secrets, tokens, private user data, or unredacted logs.
- Treat repository evidence as authoritative over chat memory.
- Verify Hermes v0.22.5-specific capabilities before relying on current online
  documentation.
- Stop after each individual decision and request explicit approval.