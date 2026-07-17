# Upstream AIO Sync Log

This file records the exact `nextcloud/all-in-one` upstream commit used when
manually syncing this customized stack. The local compose file intentionally
preserves ipvlan networking, external reverse-proxy handling, Arcane labels, and
the memories transcoder customization.

## 2026-07-16 - `nextcloud/all-in-one@a36d46987daa2b28110d9860e8bbee42d3da92a1`

Source files reviewed:

- `manual-install/latest.yml`
- `manual-install/sample.conf`

Changes adopted:

- No runtime Compose or environment-contract changes were required; both
  upstream manual-install files were unchanged from the previous review.
- Separated the online upstream audit from the maintenance window in the
  update procedure.
- Added a generic rollback-point checkpoint and concise post-update health
  checks without prescribing a backup implementation.
- Corrected the architecture diagram and environment comments to show that
  OnlyOffice is intentionally omitted and its toggle is not active.

Not adopted:

- Upstream optional services `nextcloud-aio-eurooffice`,
  `nextcloud-aio-onlyoffice`, `nextcloud-aio-talk-recording`, and
  `nextcloud-aio-whiteboard` remain omitted from `compose.yaml`.
- Local-only `nextcloud-aio-memories-transcoder` remains preserved.

## 2026-07-03 - `nextcloud/all-in-one@92cd622226c54c7333174f7c28315cbbbd16f952`

Source files reviewed:

- `manual-install/latest.yml`
- `manual-install/sample.conf`

Changes adopted:

- Added this upstream sync log so future edits record the exact upstream commit.
- Updated the repository workflow docs to fetch upstream manual files by commit
  SHA before comparing them against the local stack.
- No runtime Compose or environment-variable changes were required for the
  services currently used by this stack.

Not adopted:

- Upstream optional services `nextcloud-aio-eurooffice`,
  `nextcloud-aio-onlyoffice`, `nextcloud-aio-talk-recording`, and
  `nextcloud-aio-whiteboard` remain omitted from `compose.yaml`.
- Local-only `nextcloud-aio-memories-transcoder` remains preserved.
