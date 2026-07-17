# Nextcloud AIO — Manual Stack

Manual Docker Compose deployment of Nextcloud AIO, adapted for ipvlan networking with an external reverse proxy.

## Updating

**Do not auto-update images.** AIO image updates can require `compose.yaml` and environment changes (new env vars, changed defaults, added/removed services). Pulling new images without syncing those files can break the stack silently.

### Update procedure

1. Check upstream for compose changes while the stack remains online:
   ```bash
   UPSTREAM_AIO_SHA="$(git ls-remote https://github.com/nextcloud/all-in-one.git refs/heads/main | awk '{print $1}')"
   curl -fsSL -o /tmp/nextcloud-aio-latest.yml "https://raw.githubusercontent.com/nextcloud/all-in-one/${UPSTREAM_AIO_SHA}/manual-install/latest.yml"
   diff -u compose.yaml /tmp/nextcloud-aio-latest.yml
   ```
   Review the diff carefully. Look for:
   - New or removed environment variables
   - New services or changed service definitions
   - Changed volume mounts, ports, or healthchecks

2. Check upstream for tracked environment example changes:
   ```bash
   curl -fsSL -o /tmp/nextcloud-aio-sample.conf "https://raw.githubusercontent.com/nextcloud/all-in-one/${UPSTREAM_AIO_SHA}/manual-install/sample.conf"
   diff -u .env.example /tmp/nextcloud-aio-sample.conf
   ```
   Look for new variables or renamed ones. Add any new required variables to `.env.example`, then update the deployment `.env` manually without committing it.

3. Apply relevant changes to `compose.yaml` and `.env.example`, keeping your customizations (ipvlan, removed services, memories transcoder, etc.).

4. Add an entry to `UPSTREAM-SYNC.md` with the exact `${UPSTREAM_AIO_SHA}`, the source files reviewed, changes adopted, and notable upstream changes intentionally not adopted.

5. Enter the maintenance window and establish a rollback point:
   ```bash
   docker compose down
   ```
   After the containers stop, create or confirm a current rollback point using the stack's established backup process. This repository does not prescribe the backup implementation.

6. Pull new images and restart:
   ```bash
   docker compose pull
   docker compose up -d
   ```

7. Confirm that the containers are healthy and Nextcloud completed its startup:
   ```bash
   docker compose ps
   docker compose exec --user www-data nextcloud-aio-nextcloud php occ status
   ```

### What NOT to do

- **Don't use Arcane's image update feature for this stack.** It would pull new images without updating `compose.yaml` or environment variables, which can cause breakage.
- **Don't fork the AIO repo to track changes.** Your `compose.yaml` is heavily customized (ipvlan networking, removed services, added memories transcoder). Merge conflicts on every upstream change would be constant and error-prone. A manual diff is lower effort and safer.

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
                    │  Collabora                     │
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
