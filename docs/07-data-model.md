# Media Support — Data Model

Status: Approved Phase 2 design; no migration has been applied  
Database: Separate Neon PostgreSQL project for Media Support

## Principles

- Neon is the durable system of record for Media Support.
- Hermes retains its own SQLite internal state.
- Attachments are not stored in Neon; only metadata and Discord references are stored.
- Audit events are append-only.
- Users may have multiple concurrent tickets.
- Duplicate prevention applies only to active tickets.
- All timestamps use UTC.

## Tables

| Table | Purpose |
|---|---|
| `users` | WhatsApp support users and access/synchronization state |
| `admin_users` | Authorized Discord administrators |
| `tickets` | Support tickets and lifecycle state |
| `ticket_messages` | Inbound and outbound messages |
| `attachments` | Attachment metadata and Discord transfer state |
| `audit_events` | Immutable security and workflow history |
| `notification_attempts` | WhatsApp delivery, retry, and reconciliation state |
| `discord_refs` | Stable Discord channel/thread/message references |
| `schema_migrations` | Applied migration ledger |

## Users

| Field | Type | Notes |
|---|---|---|
| `id` | UUID | Primary key |
| `whatsapp_provider_id` | text | Stable provider identifier; unique when present |
| `phone_e164` | text | Display/administration only; not sole authorization key |
| `display_name` | text | Unverified label |
| `access_state` | enum | `active`, `revoked` |
| `sync_state` | enum | `synchronized`, `pending_allow`, `pending_revoke`, `sync_failed` |
| `created_at` | timestamptz | UTC |
| `updated_at` | timestamptz | UTC |

## Tickets

| Field | Type | Notes |
|---|---|---|
| `id` | UUID | Primary key |
| `user_id` | UUID | References `users.id` |
| `ticket_year` | smallint | Ticket year |
| `ticket_sequence` | integer | Yearly sequence |
| `ticket_code` | generated text | `TCK-YYYY-NNNN` |
| `service_category` | enum | `general`, `anime`, `asian` |
| `issue_type` | enum | `queue`, `missing_content`, `download`, `metadata`, `playback`, `access`, `other` |
| `normalized_title` | text | Deterministic normalized title |
| `status` | enum | `open`, `in_progress`, `waiting_on_user`, `resolved`, `closed` |
| `duplicate_of_ticket_id` | UUID | Optional link to similar active ticket |
| `last_user_activity_at` | timestamptz | Most recent user activity |
| `duplicate_check_at` | timestamptz | Duplicate-check timestamp |
| `created_at` | timestamptz | UTC |
| `updated_at` | timestamptz | UTC |
| `resolved_at` | timestamptz | Nullable |
| `closed_at` | timestamptz | Nullable |

Ticket codes are generated from `ticket_year` and `ticket_sequence`, with unique constraints on both the composite key and generated code.

## Duplicate prevention

A duplicate candidate matches:

```text
user_id + service_category + issue_type + normalized_title
```

Only tickets in `open`, `in_progress`, or `waiting_on_user` are considered active duplicates.

The plugin asks the user whether to add information to the existing ticket or explicitly create a new one. If a new ticket is created, `duplicate_of_ticket_id` preserves the relationship.

A partial unique index provides a race-condition safeguard:

```sql
CREATE UNIQUE INDEX uq_active_ticket_issue
ON tickets (
    user_id,
    service_category,
    issue_type,
    normalized_title
)
WHERE status IN ('open', 'in_progress', 'waiting_on_user');
```

## Enums

```sql
CREATE TYPE ticket_status AS ENUM (
    'open', 'in_progress', 'waiting_on_user', 'resolved', 'closed'
);

CREATE TYPE service_category AS ENUM (
    'general', 'anime', 'asian'
);

CREATE TYPE issue_type AS ENUM (
    'queue', 'missing_content', 'download', 'metadata',
    'playback', 'access', 'other'
);

CREATE TYPE access_state AS ENUM (
    'active', 'revoked'
);

CREATE TYPE sync_state AS ENUM (
    'synchronized', 'pending_allow', 'pending_revoke', 'sync_failed'
);

CREATE TYPE delivery_state AS ENUM (
    'queued', 'submitted', 'delivered', 'failed', 'ambiguous'
);
```

## Messages and delivery

`ticket_messages` records both inbound and outbound messages. `notification_attempts` records each delivery attempt separately.

Delivery states:

- `queued`: prepared but not sent
- `submitted`: provider accepted the request
- `delivered`: delivery confirmation received
- `failed`: definitive provider failure
- `ambiguous`: timeout, network error, or unclear provider response

An accepted API response does not automatically mean `delivered`.

## Attachments

Store metadata only:

- WhatsApp media/message ID
- Discord message/attachment ID
- Filename, MIME type, size, SHA-256
- Transfer state and timestamps

Do not store attachment bytes in Neon. Do not treat WhatsApp temporary URLs or Discord CDN URLs as permanent archives.

## Audit events

`audit_events` is append-only. It records authorization changes, ticket transitions, attachment transfers, delivery outcomes, and deletion requests.

Do not update or delete audit events; record corrective events as new rows.

## Migration strategy

- Use plain versioned SQL files under `migrations/`.
- Start with `0001_initial_schema.sql`.
- Apply migrations only after review.
- Validate every migration on a disposable Neon branch.
- Use forward-only migrations; create a compensating migration rather than relying on automatic downgrade.
- Never store secrets in migrations or Git.
