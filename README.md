# AI Media Stack

Infrastructure, configuration, and documentation for a self-hosted media stack.

## Current baseline

- **Hermes Agent** is deployed and running.
- **`mcp-arr-anime`** is the active media MCP service, covering anime Sonarr and Radarr.
- **Cloudflare Tunnel** provides existing public ingress infrastructure.
- **Stackarr** is not deployed and not used.
- **Media Support** is planned but not deployed.

## Media Support

Media Support is a planned WhatsApp-first, read-only support assistant. It will create persistent tickets, provide evidence-based diagnostics through approved MCP tools, and give administrators a private Discord workflow.

It will run inside the existing Hermes container as a Hermes plugin plus skill. A separate Neon PostgreSQL project will be the system of record for tickets, users, audit events, attachment metadata, WhatsApp window state, and Discord references.

Media Support will not manage Docker, NAS, SSH, SMB, DSM, media files, or upstream ARR credentials.

## Documentation

- [`docs/current-state.md`](docs/current-state.md) — deployed baseline and evidence precedence
- [`docs/phase-2-decisions.md`](docs/phase-2-decisions.md) — approved Phase 2 decisions
- [`docs/06-architecture-decision-record.md`](docs/06-architecture-decision-record.md) — architecture decisions
- [`docs/07-data-model.md`](docs/07-data-model.md) — Neon data model and migration strategy
- [`docs/08-project-or-environment-setup.md`](docs/08-project-or-environment-setup.md) — repository, secrets, and test setup
- [`docs/09-deployment-or-rollout-plan.md`](docs/09-deployment-or-rollout-plan.md) — rollout, rollback, and monitoring
- [`docs/runbook.md`](docs/runbook.md) — operational procedures
- [`docs/service-map.md`](docs/service-map.md) — service relationships and boundaries
- [`docs/safety-policy.md`](docs/safety-policy.md) — security and privacy rules

## Status

Phase 2 architecture is approved. Implementation, Neon provisioning, migrations, deployment changes, and production WhatsApp onboarding require separate explicit approval.
