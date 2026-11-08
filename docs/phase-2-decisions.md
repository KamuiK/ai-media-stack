# Media Support — Phase 2 Decision Checkpoint

Checkpoint date: 2026-11-08
Status: Architecture planning in progress
Overall architecture approval: Not yet granted

## Purpose

Preserve accepted decisions, proposals, unresolved questions, and technical
verification gaps so planning can continue without relying on chat history.

GitHub is the durable source of truth. Changes to decisions must be recorded
with their rationale. Do not silently overwrite conflicting decisions.

Companion document: current-state.md

## Status definitions

- Accepted: explicitly selected by the administrator.
- Conditional: selected direction, subject to technical verification.
- Proposed: discussed but not explicitly approved.
- Open: unresolved.
- Superseded: replaced by a later decision.

Acceptance of individual planning decisions is not implementation approval.

## Accepted architecture decisions

| ID | Decision | Status |
|---|---|---|
| D-001 | Product name: Media Support. Do not use Stackarr branding. | Accepted |
| D-002 | Run Media Support inside the existing Hermes container, not a separate support container. | Accepted |
| D-003 | Use a separate, private GitHub repository for plugin source; install a reviewed pinned commit into Hermes. | Accepted; installation mechanism conditional |
| D-004 | Use a completely separate Neon project for Media Support. Do not modify the unrelated existing project. | Accepted |
| D-005 | Use Neon PostgreSQL for ticket/workflow persistence instead of SQLite for support data. | Accepted |
| D-006 | Use yearly ticket numbering, such as TCK-2026-0001. | Accepted |
| D-007 | Make user attachments viewable directly in Discord ticket threads; store metadata/references in Neon. | Accepted |
| D-008 | Manage allowed WhatsApp users through administrator commands in Discord, backed by persistent records. | Accepted |
| D-009 | Display requester name alongside the number when available. | Accepted |
| D-010 | Use the existing Cloudflare Tunnel for public WhatsApp webhook ingress. | Accepted |
| D-011 | Use a private Discord text channel with one ticket thread per issue. This replaces the earlier forum preference. | Accepted |
| D-012 | Reuse the active mcp-arr-anime service initially and restrict Hermes to verified read-only tools. | Accepted |
| D-013 | Plan three MCP instances, each pairing one Sonarr and one Radarr instance. | Accepted |
| D-014 | Use category names general, anime, and asian—not tv for a service handling both shows and movies. | Accepted |
| D-015 | Route diagnostics by explicit category or verified title matches; ask when ambiguous. | Accepted |
| D-016 | Pin production Hermes/MCP images and plugin revisions; avoid floating production tags. | Accepted |
| D-017 | Replace stale documentation with an accurate current/target distinction. | Accepted |

Proposed private plugin repository name: KamuiK/media-support-plugin.
Confirm the exact repository name before creating it.

## Target components

- Hermes: conversational runtime, sessions, MCP client, messaging integrations
  where supported.
- Media Support extension: deterministic support workflow inside Hermes.
- Skill: conversational triage instructions and diagnostic presentation.
- Neon: tickets, users/access decisions, message/window state, audit records,
  notification attempts, attachment metadata, Discord references.
- Discord: private administrator interface and attachment viewer.
- Three mcp-arr containers: media-stack diagnostics.
- Cloudflare Tunnel: public ingress only for the required webhook endpoint.

Hermes retains its internal SQLite state unless a supported alternative is
separately verified. Neon support persistence does not imply replacement of
Hermes's internal database.

## MCP layout

| Hermes server key | Container/service name | Coverage |
|---|---|---|
| arr_general | mcp-arr-general | General TV and movies |
| arr_anime | mcp-arr-anime | Anime TV and movies |
| arr_asian | mcp-arr-asian | Asian TV and movies |

The design uses the administrator-confirmed constraint that each current
mcp-arr instance handles one Sonarr and one Radarr instance.

Keep the active anime integration; add category instances rather than
rewriting the upstream MCP to support multiple instances.

Use the same verified pinned image revision across category instances unless
a documented reason requires otherwise.

## Routing rules

1. Use explicit service/category context from the user or linked ticket.
2. If only a title is known, use verified read-only title lookup.
3. Each arr_search_all call is scoped to its MCP instance; it is not assumed
   to search across all three containers.
4. Search the relevant category endpoints when cross-category lookup is needed.
5. If more than one match or no reliable match exists, ask a concise question.
6. Never infer category solely from title language or genre.
7. An unavailable category MCP must be reported as unavailable—not treated as
   proof that a title is absent.

Exact orchestration and tool names remain to be verified.

## Read-only security boundary

Media Support must not:

- Add, approve, import, refresh, update, or delete media.
- Trigger acquisition searches or modify queues/torrents.
- Restart containers or services.
- Modify ARR, indexer, download-client, or VPN configuration.
- Access Docker sockets, NAS administration, SSH, SMB, DSM, or media files.
- Receive upstream ARR credentials.
- Bypass restrictions with terminal, browser, file, or direct API tools.

MCP containers hold only their category's upstream credentials.

Required before user onboarding:

- Verify tool behavior and establish an explicit read-only allowlist.
- Restrict support-user built-in tools.
- Test attempted bypasses and cross-user access.
- Review current trust: full configuration.

Proposed change to trust: untrusted remains subject to version-specific
verification; do not claim it enforces read-only access by itself.

## Ticket lifecycle

Planned ticket statuses:

- open
- in_progress
- waiting_on_user
- resolved
- closed

Required information includes requester identity, service/category, title when
relevant, user description, triage findings, recommended action, timestamps,
resolution notes, Discord references, notification state, and audit history.

Yearly numbering requires:

- A defined year boundary, preferably Europe/Vienna.
- Atomic number allocation per year.
- Uniqueness on year and sequence.
- Formatting that remains unique beyond 9,999 tickets.

The previous SQL snippets are exploratory, not an approved executable schema.

Still open:

- Close-without-resolution behavior.
- Reopening versus creating a follow-up ticket.
- Exact status-transition constraints.
- Concurrent update handling and duplicate-ticket prevention.

## WhatsApp window policy

The MVP follows the most recent inbound user message:

- Resolve while the window is open: attempt the resolution update promptly.
- Resolve after expiry: persist pending_user_reply.
- On the user's next message: deliver pending updates and handle their new
  message without automatically creating a duplicate ticket.
- Tell users at ticket creation to message again if they have not received an
  update within 24 hours.
- No routine proactive template notification is required for the MVP.

Check eligibility immediately before each send, not only at resolution time.
Use UTC timestamps and do not move the last-inbound timestamp backwards when
events arrive out of order.

Do not mark a message delivered merely because an API accepted it. Track
submission, provider message ID, delivery confirmation, failure, and ambiguous
outcomes separately.

A scheduler can reconcile durable pending work and transient failures.
Its exact mechanism is proposed, not finalized or version-verified.

Do not claim exactly-once external delivery. A database idempotency key alone
does not prevent duplicate sends after an ambiguous provider response.

## WhatsApp onboarding

Current constraints:

- No Meta developer app or business setup exists.
- No spare production number is available.

Proposed approach:

- Start with Meta's test setup, subject to account eligibility and actual
  test-number restrictions.
- Defer production number acquisition.
- Do not migrate or delete the administrator's personal WhatsApp account.
- Keep account IDs, webhook configuration, and credentials configurable.

Still verify:

- Test recipient and inbound-message limitations.
- Business/identity verification requirements for this use case.
- Production-number options and costs.
- Whether the intended Community announcement/membership behavior is supported.

Fallback for unsupported Community integration: the administrator manually
announces the bot and its DM contact details.

## Discord workflow

Use a private text channel, provisionally media-support-tickets.

Each ticket should have:

- One thread titled with ticket code and short summary.
- Requester name/identifier.
- Service/category and affected title.
- User report and evidence-based triage.
- Attachments.
- Status, resolution notes, and notification progress.

Neon is the ticket system of record. Discord is the administrative interface.

Supported behavior must be verified:

- Programmatic thread creation for a WhatsApp-originated ticket.
- Attachment upload.
- Thread updates and command handling.

Hermes auto-thread-on-mention behavior does not by itself prove automatic
ticket-thread creation from WhatsApp events.

Desired admin operations:

- Allow/revoke/list users.
- Resolve with mandatory notes.
- Close.
- Get status.

Thread replies are an acceptable MVP interface. Exact slash-command support,
syntax, permissions, and role-versus-user-ID authorization remain open.

Authorization must use trusted Discord event identity—not model-supplied
administrator IDs.

## User identity and allowlist

Store a stable provider identifier when available, phone number when available,
and display name.

Display names are unverified labels, never authorization credentials.

The accepted user experience is easy Discord-based access management.

A previous suggestion combined a static Hermes environment allowlist with
Neon-managed permissions. Synchronization is unresolved. Do not finalize that
combination if every allow/revoke requires manual edits or restarts.

Determine a supported authorization path that:

- Rejects unauthorized users before LLM/tool execution.
- Applies revocation promptly.
- Preserves admin audit history.
- Handles provider identifier changes safely.

## Attachments and privacy

Intended flow:

1. Receive WhatsApp media reference.
2. Validate type and size.
3. Download through the authenticated provider API.
4. Upload to the private Discord ticket thread.
5. Persist metadata, transfer outcome, and stable Discord IDs.

Do not treat temporary WhatsApp URLs or Discord CDN URLs as permanent archives.
Keep message/attachment IDs for retrieval where supported.

Still open:

- Retention and deletion policy.
- Maximum size and permitted file types.
- Handling failed/oversized uploads.
- Temporary-file cleanup and access restrictions.
- Whether separate archival is needed.
- When images may be sent to an LLM provider.

Do not send attachments or unredacted logs to the LLM by default merely because
they were attached to a ticket.

## Plugin implementation direction

The administrator selected a private Git-installed plugin approach.

The intended split is:

- Skill: natural-language triage and diagnostic guidance.
- Plugin/executable workflow: validation, persistence, authorization, audit,
  ticket state, and external integration orchestration.

However, exact Hermes v0.22.5 APIs remain unverified.

Earlier example manifests, ctx methods, installation commands, dependency
versions, and package layouts were illustrative. They are not approved working
code and must not be copied as a validated implementation.

Verify before finalizing:

- Manifest schema and entrypoint.
- Supported public plugin APIs.
- Inbound hooks before agent execution.
- Discord event and command access.
- WhatsApp outbound access.
- Private Git installation at a pinned revision.
- Dependency installation and container persistence.
- Lifecycle/background-task handling.

If an API is unavailable, present alternatives without silently adding a new
container or upgrading Hermes.

## Alternatives considered

| Area | Selected direction | Tradeoff / alternative |
|---|---|---|
| Support runtime | Inside Hermes | Simpler deployment; shared failure/security boundary versus separate service |
| Support persistence | Neon | Existing familiarity; depends on network/database availability |
| MCP expansion | Three category containers | No upstream multi-instance rewrite; additional configuration and namespaces |
| Discord tickets | Text channel plus threads | Simpler than forums; requires verified ticket-driven thread creation |
| Plugin source | Separate private repository | Independent revisions; coordinated release/deployment documentation needed |
| Skill-only workflow | Not sufficient alone for critical state changes | Simpler prompting, but requires deterministic operations underneath |
| Attachments | Discord viewer | Convenient admin access; not a guaranteed independent archive |

## Remaining Phase 2 decisions

1. Verify pinned Hermes extension/messaging capabilities.
2. Finalize plugin package/install/dependency approach.
3. Finalize database schema and migration strategy.
4. Resolve dynamic allowlist enforcement.
5. Define Discord admin identity, command handling, and permissions.
6. Finalize restricted support toolsets and verified MCP tool allowlists.
7. Decide rollout order for category MCP instances.
8. Define webhook hostname, listener port/path, signature validation, and
   public-route isolation.
9. Define retry, delivery-confirmation, recovery, and reconciliation behavior.
10. Define privacy, retention, attachment limits, and deletion.
11. Define local development, CI, test environments, and acceptance criteria.
12. Define secrets, backup/restore, health checks, updates, and rollback.
13. Document Meta onboarding and production-number blockers.

Previously proposed recommendations that are not yet accepted include plain SQL
migrations, specific retention periods, exact dependency versions, and exact
CI configuration.

## Required Phase 2 deliverables

Complete or consolidate:

- docs/06-architecture-decision-record.md
- docs/07-data-model.md
- docs/08-project-or-environment-setup.md
- docs/09-deployment-or-rollout-plan.md
- AGENTS.md draft
- README.md draft
- Proposed folder structure
- Initial Git/GitHub setup commands
- Environment-variable and secret inventory without values
- Alternatives, tradeoffs, risks, and open questions
- Explicit request for overall architecture/setup approval

Update stale README, runbook, service map, and safety policy without presenting
planned changes as deployed.

These checkpoint files preserve context; they do not replace the final
Phase 2 deliverables.

## Resume instructions

In a new thread:

1. Read this file and current-state.md from GitHub.
2. Inspect relevant configuration and recent commits.
3. Reconcile discrepancies before recommending changes.
4. Verify exact Hermes v0.22.5 extension APIs first.
5. Continue remaining decisions one at a time.
6. Do not begin implementation or create work packages.
7. Request explicit approval when Phase 2 is complete.

## Approval boundary

No implementation, infrastructure creation, migrations, or deployment changes
are authorized by this checkpoint.

GitHub writes require review and approval of the exact target/content.
Stop after Phase 2 until overall architecture/setup approval is granted.