# psitransfer-for-tos

> TerraMaster TOS 7 App Center — **Docker application** package for
> [Psitransfer](https://hub.docker.com/r/psitrax/psitransfer).

## What this is

A four-file TOS Docker application archive (`psitransferdocker.tar.gz`) that installs
Psitransfer on TOS 7 through the App Center:

- `config.ini` — TOS application metadata (`application_type: docker`, id `psitransferdocker`)
- `psitransferdocker.lang` — 23-language superset (TOS requires 14)
- `psitransferdocker.svg` — application icon (274 bytes, 2 SVG elements)
- `docker-compose.yml` — one service, image `psitrax/psitransfer:v2.4.1`

## How it works

| | |
|---|---|
| Container image | `psitrax/psitransfer:v2.4.1` (Docker Hub, used unmodified) |
| Open | `http://<NAS-IP>:8045/` (container port 3000) |
| Runs as | non-root, `user: "1000:1000"` |
| Persistent data | `/data` → `/Volume1/DockerAppData/psitransferdocker/data` |
| Logs | `docker logs psitransferdocker` (stdout/stderr) |



## Privacy

This application runs entirely on your NAS. **The developer does not collect, receive or have
access to any file you transfer or to any data the application processes.** There is no
telemetry, no analytics, no crash reporting and no update check.

| Topic | Detail |
|---|---|
| Data processed | The files you transfer with their metadata (size, upload time, retention, one-time flag, optional bucket password) and the access log (timestamp, method, URL, status, response time, remote IP). No database, no accounts. |
| Where it is stored | The application data folder (`/Volume1/DockerAppData/psitransferdocker/`) on your NAS. Nothing is uploaded to the developer. |
| Retention period | **The app deletes transfers by itself**: the retention chosen per upload (default **7 days**), a hard **75-day** maximum age, and one-time links after the first download. Logs follow the DockerEngine rotation. Uninstalling keeps the data unless you tick “delete data”. |
| Your rights | **Access**: the uploader uses the share link; the owner reads the data folder or enables `/admin` (`adminPass`). **Correct**: re-upload (uploads are immutable snapshots). **Delete**: share-link delete, admin page delete, or remove the bucket folder. **Restrict**: `uploadPass`, `requireBucketPassword`, retention, LAN-only or TLS proxy. |
| Deletion channel | Self-service: share-link delete, admin page, or delete `/Volume1/DockerAppData/psitransferdocker/data/<bucket-id>` (immediate, complete). Questions or lost links: **moechz@163.com** (answered within 30 days). |
| Security measures | uid 1000 (never root), no `privileged`, no host network, no `cap_add`, no Docker socket, one published port; image pinned to `psitrax/psitransfer:v2.4.1`. The UI is plain HTTP and `/admin` is **off by default** (`adminPass`); set `adminPass`/`uploadPass` before exposing the port, and use a TLS reverse proxy. |
| Children | Not directed at children. |

**Full privacy policy:** [PRIVACY.md](PRIVACY.md) —
<https://github.com/Moechz/psitransfer/blob/main/PRIVACY.md> — and the website version at
<https://www.moechz.com/privacy.html>.

## Third-party services (API disclosure)

The developer operates no server in this path, and Psitransfer has no telemetry and no
automatic update check.

| Service | Endpoint | Data sent | Destination / region |
|---|---|---|---|
| Uploader's browser → the app | `http://<NAS-IP>:8045` | the uploaded file, its name/size, chosen retention | your own NAS / LAN |
| Download link you share | same address, `/download/<id>` | the file, to whoever holds the link | your own NAS |
| Image pull | `registry-1.docker.io` or your registry mirror | anonymous pull | Docker Hub, US / your mirror |
| Optional webhooks you configure (`fileUploadedWebhook` / `fileDownloadedWebhook`) | the URL you enter | file name, size, bucket id, client IP | the endpoint you chose |
| Update check | none | — | — |

See `PRIVACY.md` §6 for the full table. Disable any integration in the app's settings if you
do not want it.

## Licence and attributions

| Component | Licence | Notes |
|---|---|---|
| Psitransfer | see upstream | upstream application, used unmodified inside the image |
| Container image `psitrax/psitransfer:v2.4.1` | see the image's own metadata | pulled from Docker Hub at install time |
| This packaging | GPL-3.0 (see `LICENSE`) | compose file, metadata, translations, icon |

“Psitransfer”, “TerraMaster” and “TOS” are trademarks of their respective owners. This is an
independent community package by Moechz and is not affiliated with the upstream project or
TerraMaster. Full details: [NOTICE.md](NOTICE.md).

## Design highlights

- **Non-root.** `user: "1000:1000"`; verified on real hardware (the container reports uid 1000).
- **Fixed image tag, never `:latest`**, Docker Hub only, no `privileged`, no `network_mode: host`, and no Docker socket, no `cap_add`.
- **Health check** probing the service on container port 3000.
- **Deterministic build**: GNU tar with uid/gid 0 and zeroed timestamps, gzip without a timestamp, plus the review-standards self-check above.

## Runtime file manifest

Everything the application writes on the NAS lives inside its own data
directory; the container-internal `/tmp` is discarded with the container. The
table lists what is created and how it behaves. Nothing else is written.

| Path | Purpose | Created when | Growth bound | Lifecycle |
|---|---|---|---|---|
| `/Volume1/DockerAppData/psitransferdocker/data` | Psitransfer application data (everything the container writes outside /tmp) | on first start / on demand | bounded by your own data | persistent; kept on uninstall unless you ask to delete it |

## Build

```bash
scripts/build.sh                  # -> out/psitransferdocker.tar.gz + .sha256
scripts/build.sh 2.4.1-2   # packaging iteration bump
scripts/build.sh 2.4.1-1 aarch64
```

The build stages the four files, substitutes the version/platform placeholders,
removes CRLF/BOM/AppleDouble, then runs
`tools/verify.py`, a review-standards self-check (archive layout, JSON validity,
required fields, 14+ language coverage, version consistency, Docker-Hub-only
images, fixed tag, reserved/recommended host ports, non-root user, per-service
healthcheck/TZ/restart, data path below the application data root, never the
`/Volume*` wildcard, comment-free sequence blocks, `x-app-meta` position, no
literal secrets and the SVG size/element limits).

## Local verification

```bash
./TOSAppSelfTestingTool -c out/psitransferdocker.tar.gz   # official spec check
bash test/install-and-verify.sh                  # on-device E2E (needs root)
```

> Docker applications cannot be side-loaded through the App Center's manual
> install page (it accepts `.deb` only); the official self-testing tool's `-i`
> mode or `docker compose` are the only ways to install one for testing.

## Store submission checklist

1. Public GitHub repo (code + README only).
2. Release **tag `v2.4.1-1`**, assets `psitransferdocker.tar.gz` +
   `psitransferdocker.tar.gz.sha256` (Docker asset naming: `<app_id>.tar.gz`, **no
   platform suffix**).
3. Developer platform → Add Application: ID `psitransferdocker`, package type Docker,
   repository URL, architecture `x86_64`.
4. Version Management → Add Version `2.4.1-1` → automated validation → review.


## Limitations

- The application runs inside a container; data lives in the application data directory (`/Volume1/DockerAppData/psitransferdocker/`) as described above.

---

Packaged for TOS 7 by Moechz. Upstream image used unmodified from Docker Hub.

## Support

- Issues and questions: <https://github.com/Moechz/psitransfer/issues>
- Email: moechz@163.com
- Privacy: [PRIVACY.md](PRIVACY.md) · Licence: [NOTICE.md](NOTICE.md)
