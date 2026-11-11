# SHIPUCHKA launcher update-server contract

This repository currently contains release metadata and documentation, not the Java source code of the launcher. The following contract is the integration target for the launcher source.

## Manifest URL

`https://shipuchka-updates.onrender.com/api/manifest`

The launcher must request this URL over HTTPS on startup (or when the user presses Check for updates).

## Manifest fields

The server returns both naming styles for compatibility:

- `published`: boolean; do not offer an update when false.
- `version` and `latest_version`: published client version.
- `downloadUrl` and `download_url`: direct HTTPS JAR download URL.
- `sha256`: 64-character hexadecimal SHA-256 digest.
- `releaseNotes` and `release_notes`: release notes.
- `file_size`: uploaded file size in bytes.
- `published_at`: ISO-8601 publication time.
- `mandatory`: whether the update is mandatory.

## Required client behavior

1. Use the manifest URL above; do not read the GitHub `latest.json` as the live update source.
2. Check HTTP status and JSON types before using fields.
3. If `published` is false, continue normal launch.
4. Compare the installed version with `version` (fallback: `latest_version`).
5. Ask the player before downloading optional updates.
6. Download only from HTTPS.
7. Verify SHA-256 before replacing any installed JAR.
8. Keep the previous JAR until the new JAR has passed verification; restore it if installation fails.
9. Never embed developer credentials, invite codes, JWTs, or GitHub tokens in the player launcher.

## Release publishing API

Developer tools authenticate using:

- `POST /api/auth/register` (first account is admin; invite code required)
- `POST /api/auth/login` (returns a 12-hour bearer token)
- `POST /api/admin/release` (multipart fields: `version`, `notes`, and `file` containing a `.jar`)

The server calculates SHA-256 itself. Do not trust a hash supplied by the publishing client.

## Important implementation status

This file documents the agreed integration contract. It does not itself modify or rebuild a launcher binary. The Java source for the player launcher and the Developer Launcher must be connected to this contract and rebuilt before end-to-end testing.
