# Service Map — Media Types, Instances, and Routing

This file maps user-facing media types to the correct Sonarr/Radarr instance, root folder, quality profile, download client, and Seerr workflow.

Stackarr and Hermes must use this map when routing requests. If a request does not clearly match one media type, Hermes must ask the user instead of guessing.

## Existing stack facts

- All existing Sonarr/Radarr containers use `network_mode: synobridge`.
- Each Sonarr/Radarr container mounts `${DATA_ROOT}` at `/media`.
- Therefore, all Sonarr/Radarr root folders begin with `/media/` inside the respective application.
- The exact root-folder subpaths, quality profiles, categories/tags, and Seerr behavior must be verified from each running application.

## Routing rules

| User request type | Target service | Compose service | Container name | Approval model |
|---|---|---|---|---|
| General TV series | Sonarr | `sonarr-tv` | `prod-sonarr-tv` | Manual approval |
| Anime TV series | Sonarr | `sonarr-anime` | `prod-sonarr-anime` | Manual approval |
| Asian TV series | Sonarr | `sonarr-asian` | `prod-sonarr-asian` | Manual approval |
| General movie | Radarr | `radarr-movies` | `prod-radarr-movies` | Manual approval |
| Anime movie | Radarr | `radarr-anime` | `prod-radarr-anime` | Manual approval |
| Asian movie | Radarr | `radarr-asian` | `prod-radarr-asian` | Manual approval |
| Ambiguous request | Seerr | `seerr` | `TODO_VERIFY_CONTAINER_NAME` | Manual approval |
| Unknown media type | None | — | — | Ask the user |

Until WP4 is complete, Hermes must not add, monitor, approve, or import media automatically. It may only search, inspect, and propose an action.

## Sonarr instances

| Instance | Media type | Compose service | Container name | Internal port | Published port | Root folder | Quality profile | Download client | Category/tag | Notes |
|---|---|---|---|---:|---:|---|---|---|---|---|
| General TV | TV series | `sonarr-tv` | `prod-sonarr-tv` | `8989` | `${SONARR_TV_PORT}` | `TODO_VERIFY_ROOT_FOLDER` | `TODO_VERIFY_QUALITY_PROFILE` | `TODO_VERIFY` | `TODO_VERIFY_CATEGORY` | General television |
| Anime TV | Anime series | `sonarr-anime` | `prod-sonarr-anime` | `8989` | `${SONARR_ANIME_PORT}` | `TODO_VERIFY_ROOT_FOLDER` | `TODO_VERIFY_QUALITY_PROFILE` | `TODO_VERIFY` | `TODO_VERIFY_CATEGORY` | Anime television |
| Asian TV | Asian series | `sonarr-asian` | `prod-sonarr-asian` | `8989` | `${SONARR_ASIAN_PORT}` | `TODO_VERIFY_ROOT_FOLDER` | `TODO_VERIFY_QUALITY_PROFILE` | `TODO_VERIFY` | `TODO_VERIFY_CATEGORY` | Asian television |

## Radarr instances

| Instance | Media type | Compose service | Container name | Internal port | Published port | Root folder | Quality profile | Minimum availability | Download client | Category/tag | Notes |
|---|---|---|---|---:|---:|---|---|---|---|---|---|
| General movies | Movies | `radarr-movies` | `prod-radarr-movies` | `7878` | `${RADARR_MOVIES_PORT}` | `TODO_VERIFY_ROOT_FOLDER` | `TODO_VERIFY_QUALITY_PROFILE` | `TODO_VERIFY` | `TODO_VERIFY` | `TODO_VERIFY_CATEGORY` | General films |
| Anime movies | Anime films | `radarr-anime` | `prod-radarr-anime` | `7878` | `${RADARR_ANIME_PORT}` | `TODO_VERIFY_ROOT_FOLDER` | `TODO_VERIFY_QUALITY_PROFILE` | `TODO_VERIFY` | `TODO_VERIFY` | `TODO_VERIFY_CATEGORY` | Anime films |
| Asian movies | Asian films | `radarr-asian` | `prod-radarr-asian` | `7878` | `${RADARR_ASIAN_PORT}` | `TODO_VERIFY_ROOT_FOLDER` | `TODO_VERIFY_QUALITY_PROFILE` | `TODO_VERIFY` | `TODO_VERIFY` | `TODO_VERIFY_CATEGORY` | Asian films |

## Network access for Stackarr

Because the existing services use `network_mode: synobridge`, Stackarr must use one of these access models:

| Option | How Stackarr reaches services | Recommendation |
|---|---|---|
| Join `synobridge` | Use container names such as `prod-sonarr-tv:8989` | Preferred if Synology permits joining Stackarr to `synobridge` |
| Use NAS host IP | Use `http://<NAS_LAN_IP>:<PUBLISHED_PORT>` | Simple, but depends on published ports and firewall rules |
| Use `host.docker.internal` / host gateway | Reach services through their published host ports | Useful when joining `synobridge` is not practical |

Do **not** assume `http://sonarr-tv:8989` works. The Compose service name is not necessarily reachable from Stackarr because the existing services do not share a user-defined Compose network with Stackarr.

Suggested Stackarr service URLs:

```text
Sonarr TV:         http://prod-sonarr-tv:8989
Sonarr Anime:      http://prod-sonarr-anime:8989
Sonarr Asian:      http://prod-sonarr-asian:8989
Radarr Movies:     http://prod-radarr-movies:7878
Radarr Anime:      http://prod-radarr-anime:7878
Radarr Asian:      http://prod-radarr-asian:7878
```

If container-name resolution fails on `synobridge`, replace `prod-sonarr-tv` etc. with the NAS LAN IP and each service’s published port.

## Download clients

| Client | Connected instances | Host from service perspective | Port | Category mapping | Completed download path | Post-import action | Notes |
|---|---|---|---|---|---|---|---|
| `qbittorrent` | `TODO_VERIFY_INSTANCES` | `TODO_VERIFY_QBIT_HOST` | `TODO_VERIFY_QBIT_PORT` | `TODO_VERIFY_CATEGORY_MAPPING` | `TODO_VERIFY_COMPLETED_PATH` | `TODO_VERIFY_MOVE_HARDLINK_OR_COPY` | qBittorrent runs behind Gluetun |

The qBittorrent connection details must be copied from the existing Sonarr/Radarr download-client settings. Do not infer them from this repository.

## Seerr routing

| Seerr request type | Destination instance | Automatic action | Manual action |
|---|---|---|---|
| TV request | `sonarr-tv` | `TODO_VERIFY` | Approve and add |
| Anime TV request | `sonarr-anime` | `TODO_VERIFY` | Approve and add |
| Asian TV request | `sonarr-asian` | `TODO_VERIFY` | Approve and add |
| Movie request | `radarr-movies` | `TODO_VERIFY` | Approve and add |
| Anime movie request | `radarr-anime` | `TODO_VERIFY` | Approve and add |
| Asian movie request | `radarr-asian` | `TODO_VERIFY` | Approve and add |

Seerr remains the user-facing request intake. Hermes may inspect requests, but it must not approve or create them without explicit approval once write actions are enabled.

## Indexers

| Indexer | Managed by | Available to | Protocol | Purpose | Status |
|---|---|---|---|---|---|
| `TODO_INDEXER_NAME` | Prowlarr | `TODO_INSTANCES` | Torrent | General indexers | `TODO_VERIFY` |
| `TODO_INDEXER_NAME` | Jackett | `TODO_INSTANCES` | Torrent | Legacy or specialized indexers | `TODO_VERIFY` |

Do not allow Hermes to add, remove, enable, disable, or modify indexers. Indexer changes can affect all Sonarr/Radarr instances and should be handled through the existing media-server repository.

## Quality and naming policy

| Media type | Preferred quality | Accepted upgrades | Forbidden qualities | Naming convention |
|---|---|---|---|---|
| General TV | `TODO_VERIFY` | `TODO_VERIFY` | `TODO_VERIFY` | `TODO_VERIFY` |
| Anime TV | `TODO_VERIFY` | `TODO_VERIFY` | `TODO_VERIFY` | `TODO_VERIFY` |
| Asian TV | `TODO_VERIFY` | `TODO_VERIFY` | `TODO_VERIFY` | `TODO_VERIFY` |
| General movies | `TODO_VERIFY` | `TODO_VERIFY` | `TODO_VERIFY` | `TODO_VERIFY` |
| Anime movies | `TODO_VERIFY` | `TODO_VERIFY` | `TODO_VERIFY` | `TODO_VERIFY` |
| Asian movies | `TODO_VERIFY` | `TODO_VERIFY` | `TODO_VERIFY` | `TODO_VERIFY` |

## Validation checklist

- [ ] Stackarr can reach all six Sonarr/Radarr instances.
- [ ] The selected access model—`synobridge`, NAS IP, or host gateway—is documented and tested.
- [ ] Every Sonarr/Radarr instance has exactly one intended media type.
- [ ] Every instance’s root folder has been copied from its own Settings → Root Folders page.
- [ ] Every instance has an explicit quality profile.
- [ ] Every instance has an explicit download client and category/tag.
- [ ] Seerr request routing matches the intended Sonarr/Radarr instance.
- [ ] No instance writes into another instance’s root folder.
- [ ] Stackarr can read each service’s status without requiring elevated authority.
- [ ] Hermes cannot add or approve media during WP0–WP3.