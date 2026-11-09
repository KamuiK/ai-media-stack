# Media Stack Service Map

Status: Updated for current baseline and approved Media Support planning

## Active services

```text
User
  |
  v
Hermes Agent
  |
  +--> mcp-arr-anime --> Anime Sonarr / Radarr
  |
  +--> Telegram
```

## Planned Media Support flow

```text
WhatsApp user
  |
  v
Cloudflare Tunnel
  |
  v
Hermes WhatsApp webhook
  |
  v
Hermes native allowlist
  |
  v
Media Support plugin
  |
  +--> Neon PostgreSQL
  +--> Discord ticket thread
  +--> Hermes agent / MCP
  +--> WhatsApp Cloud API
```

## Component boundaries

| Component | Access |
|---|---|
| Hermes | Neon, Discord, WhatsApp, approved MCP tools |
| Media Support plugin | Neon, Discord, WhatsApp, Hermes APIs |
| MCP containers | Their own Sonarr/Radarr credentials only |
| Neon | Media Support data only |
| Discord | Private admin interface and attachment viewer |
| Cloudflare Tunnel | Public webhook ingress only |

## Network isolation

- MCP containers remain on the internal Docker network.
- Neon is accessed only by the Media Support plugin.
- Discord and WhatsApp use outbound API access.
- Only the WhatsApp webhook path is publicly exposed.
- Hermes dashboard, API, MCP endpoints, Docker socket, NAS, SSH, SMB, and DSM remain private.

## Planned category instances

| Hermes key | Service | Coverage |
|---|---|---|
| `arr_general` | `mcp-arr-general` | General TV and movies |
| `arr_anime` | `mcp-arr-anime` | Anime TV and movies |
| `arr_asian` | `mcp-arr-asian` | Asian TV and movies |

## Non-goals

Media Support does not provide:

- Docker or NAS administration
- SSH, SMB, or DSM access
- Media-file access
- Upstream ARR credential access
- Automatic media acquisition or modification
