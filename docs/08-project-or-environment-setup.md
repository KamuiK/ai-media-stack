# Media Support — Project and Environment Setup

Status: Approved Phase 2 planning; no implementation or infrastructure change authorized

## Repositories

| Repository | Purpose |
|---|---|
| `KamuiK/ai-media-stack` | Infrastructure, configuration, documentation, and deployment baseline |
| Proposed: `KamuiK/media-support-plugin` | Private Hermes plugin and skill source; confirm exact name before creation |

The plugin repository must be private and installable at a pinned full commit SHA.

## Proposed plugin layout

```text
media-support-plugin/
├── plugin.yaml
├── pyproject.toml
├── src/media_support/
│   ├── __init__.py
│   ├── plugin.py
│   ├── tickets.py
│   ├── authorization.py
│   ├── whatsapp.py
│   ├── discord.py
│   ├── audit.py
│   └── reconciliation.py
├── skills/
│   └── media-support/
│       └── SKILL.md
├── migrations/
│   └── 0001_initial_schema.sql
└── tests/
```

The exact manifest schema, entrypoint, dependency mechanism, and installation command must be verified against Hermes v0.22.5 source before implementation.

## Environment variables

| Variable | Consumer | Purpose | Secret |
|---|---|---|---|
| `MEDIA_SUPPORT_DATABASE_URL` | Plugin | Neon PostgreSQL connection | Yes |
| `DISCORD_BOT_TOKEN` | Hermes Discord integration | Discord API access | Yes |
| `WHATSAPP_ACCESS_TOKEN` | Hermes WhatsApp integration | WhatsApp Cloud API access | Yes |
| `WHATSAPP_VERIFY_TOKEN` | Hermes webhook verification | Meta verification handshake | Yes |
| `WHATSAPP_APP_SECRET` | Hermes webhook verification | Meta signature validation | Yes |
| `WHATSAPP_PHONE_NUMBER_ID` | Hermes WhatsApp integration | Sending identity | No |
| `WHATSAPP_BUSINESS_ACCOUNT_ID` | Hermes WhatsApp integration | Meta account reference | No |
| `OPENROUTER_API_KEY` | Hermes model provider | Model access | Yes |

Never commit real values. Use Hermes environment secrets or an approved secret store.

## Local development

1. Clone the private plugin repository.
2. Create a Python virtual environment.
3. Install pinned development dependencies.
4. Run unit tests for ticket, authorization, duplicate, audit, and reconciliation logic.
5. Use synthetic fixtures; do not use real user data or production credentials.
6. Validate SQL migrations against a disposable Neon branch.
7. Run Hermes plugin validation with the approved pinned Hermes revision.

## Test environments

| Environment | Purpose |
|---|---|
| Local unit tests | Deterministic plugin logic |
| Neon test branch | Migration and constraint validation |
| Hermes integration test | Plugin loading, commands, hooks, adapters, MCP |
| Meta test number | Real WhatsApp webhook and message flow |
| Discord test channel | Ticket threads, commands, attachments |

## Acceptance gate

Do not onboard real users until:

- Plugin installs and loads from its pinned revision.
- Unauthorized WhatsApp users cannot reach the LLM or MCP tools.
- Allow/revoke synchronization is auditable and verifiable.
- Duplicate prevention works correctly.
- Ticket creation and Discord thread creation work.
- Read-only MCP restrictions are verified.
- Attachment transfer and cleanup work.
- The 24-hour window and reconciliation behavior work.
- Privacy and deletion workflows are documented and testable.
- Hermes, MCP, and plugin revisions are pinned.
