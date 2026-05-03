# Agent Instructions

This repository is a customized manual Docker Compose deployment of Nextcloud AIO. It is not the upstream Nextcloud AIO source tree.

## Update Requests

When the user asks to "update this repo", "pull the latest", or "do a diff" in this repository, do not assume they mean `git pull`.

The expected workflow is to compare this customized stack against upstream Nextcloud AIO manual install files:

1. Fetch upstream manual install files:
   ```bash
   curl -fsSL -o /tmp/nextcloud-aio-latest.yml https://raw.githubusercontent.com/nextcloud/all-in-one/main/manual-install/latest.yml
   curl -fsSL -o /tmp/nextcloud-aio-sample.conf https://raw.githubusercontent.com/nextcloud/all-in-one/main/manual-install/sample.conf
   ```
2. Diff upstream Compose against the local stack:
   ```bash
   diff -u compose.yaml /tmp/nextcloud-aio-latest.yml
   ```
3. Diff upstream environment variables against the tracked example:
   ```bash
   diff -u .env.example /tmp/nextcloud-aio-sample.conf
   ```
4. Report changes that may need manual adoption, especially new or removed environment variables, service changes, volume changes, ports, healthchecks, image behavior, or changed defaults.

Only run `git pull` if the user explicitly asks to update the Git checkout itself. The remote may require user-controlled SSH hardware-key authentication, so do not treat a failed `git pull` as part of the normal AIO update workflow.

## Local Customizations To Preserve

- `compose.yaml` is heavily customized for ipvlan networking and an external reverse proxy.
- Apache uses ipvlan for reverse proxy access and the internal bridge for backend services.
- Talk is ipvlan-only.
- Most other services are bridge-only.
- Arcane image updates are disabled and should stay disabled for this stack.
- The memories transcoder customization should be preserved.
- `.env` is ignored because it contains deployment-specific values. Do not read, edit, or ask to commit `.env` unless the user explicitly requests it. Use `.env.example` for tracked variable comparisons.

## Operational Caution

Do not auto-update container images for this stack. Upstream AIO image changes can require matching Compose or environment changes, so the diff review must happen before any `docker compose pull` or restart.
