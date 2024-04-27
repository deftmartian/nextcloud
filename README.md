# Nextcloud AIO — Manual Stack

Manual Docker Compose deployment of Nextcloud AIO, adapted for ipvlan networking with an external reverse proxy.

## Updating

**Do not auto-update images.** AIO image updates can require compose.yml and .env changes (new env vars, changed defaults, added/removed services). Pulling new images without syncing those files can break the stack silently.

### Update procedure

1. Stop the stack:
   ```bash
   docker compose down
   ```

2. Check upstream for compose changes:
   ```bash
   curl -sO https://raw.githubusercontent.com/nextcloud/all-in-one/main/manual-install/latest.yml
   diff compose.yml latest.yml
   ```
   Review the diff carefully. Look for:
   - New or removed environment variables
   - New services or changed service definitions
   - Changed volume mounts, ports, or healthchecks

3. Check upstream for .env changes:
   ```bash
   curl -sO https://raw.githubusercontent.com/nextcloud/all-in-one/main/manual-install/sample.conf
   diff .env sample.conf
   ```
   Look for new variables or renamed ones. Add any new required variables to your .env.

4. Apply relevant changes to your compose.yml and .env, keeping your customizations (ipvlan, removed services, memories transcoder, etc.).

5. Pull new images and restart:
   ```bash
   docker compose pull
   docker compose up -d
   ```

### What NOT to do

- **Don't use Arcane's image update feature for this stack.** It would pull new images without updating compose.yml/.env, which can cause breakage.
- **Don't fork the AIO repo to track changes.** Your compose.yml is heavily customized (ipvlan networking, removed services, added memories transcoder). Merge conflicts on every upstream change would be constant and error-prone. A manual diff is lower effort and safer.

### Recommended workflow

Check for upstream changes monthly or when you notice a new AIO release. The diff-based approach above takes a few minutes and gives you full control over what changes to adopt. Bookmark the AIO releases page to watch for breaking changes:
https://github.com/nextcloud/all-in-one/releases

## Architecture

```
                         ┌─────────────────────────┐
                         │     Reverse Proxy        │
                         │     (handles TLS)        │
                         └────────┬────────────────┘
                                  │
                    ┌─────────────┴─────────────────┐
                    │  ipvlan (vlan)                  │
                    │                                │
              .10 Apache              .11 Talk (TURN) │
                    │                                │
                    └─────────────┬─────────────────┘
                                  │
                    ┌─────────────┴─────────────────┐
                    │  bridge (nextcloud-internal)    │
                    │                                │
                    │  Nextcloud    PostgreSQL        │
                    │  Redis       Notify-push       │
                    │  Collabora   OnlyOffice        │
                    │  ClamAV      Imaginary          │
                    │  Fulltextsearch                 │
                    │  Memories Transcoder (NVIDIA)   │
                    └────────────────────────────────┘
```

- **Apache** is on both networks (ipvlan for reverse proxy access, bridge for backend containers)
- **Talk** is ipvlan-only (eturnal relay IP detection breaks with dual NICs)
- **All other services** are bridge-only

## Post-migration setup

After first startup with the memories transcoder:
```bash
docker compose exec nextcloud-aio-nextcloud php occ config:system:set memories.vod.external --value true --type bool
docker compose exec nextcloud-aio-nextcloud php occ config:system:set memories.vod.connect --value nextcloud-aio-memories-transcoder:47788
```

## Firewall rules

- Forward `TALK_PORT` (3478) TCP+UDP to `192.168.<VLAN_TAG>.11`
- Reverse proxy targets `http://192.168.<VLAN_TAG>.10:<APACHE_PORT>`