# Privacy Policy — Psitransfer for TerraMaster TOS (`psitransferdocker`)

**Last updated:** 19 November 2026
**Publisher / data controller:** Moechz — moechz@163.com

This policy covers the TOS application package `psitransferdocker` (the “app”) and the
Psitransfer instance it runs on your TerraMaster NAS. It is linked from the package itself
(the `help` field of `config.ini`, i.e. the “Help” link of the App Center entry), from the
store listing and from the developer website (<https://www.moechz.com/privacy.html>,
section 6).

## 1. Summary

- The app runs **entirely on your NAS**. The developer does not receive, collect or have
  access to the files you transfer or to any data the app processes.
- **No telemetry, no analytics, no tracking, no advertising SDK, no crash reporting and no
  update check.** The developer operates no server in this path.
- Psitransfer is made for **third parties to upload files to you** (and for you to send them
  files), so this policy also explains what those users can do about the data they hand over
  — see §4 and §5.

## 2. What data is processed and where it is stored

Everything lives in the application data folder, mounted at `/data`
(`PSITRANSFER_UPLOAD_DIR=/data`) → `/Volume1/DockerAppData/psitransferdocker/data`:

| Data | Where | Purpose |
|---|---|---|
| Uploaded files and their metadata (size, upload time, chosen retention, one-time flag, optional bucket password) | `/data/<bucket-id>/` and the bucket metadata file | The transfer itself: generated share links point here |
| Access log lines — timestamp, HTTP method, URL, response status, response time, **remote IP address** | container stdout (`docker logs psitransferdocker`), rotated by DockerEngine | Troubleshooting and abuse diagnosis; never sent anywhere |
| Configuration (admin/upload password, retention options, webhook URLs) | environment (`PSITRANSFER_*`), set by you in the compose file or a Docker override | Runtime behaviour |
| No database, no account system, no device fingerprint | — | — |

The app writes nothing outside `/data` and the container's own temporary directory (discarded
with the container).

## 3. Retention period

Psitransfer deletes data by itself, following the retention you (or the uploader) choose:

- **Per-upload retention** — the uploader picks a retention when creating the transfer; the
  default is **7 days** (`defaultRetention: 604800`). Options shipped by the image: one-time
  download, 1 hour, 6 hours, 1 day, 3 days, 1 week, 2 weeks, 4 weeks, 8 weeks.
- **Global maximum age** — every file is removed after `maxAge`, **75 days** by default
  (`3600 * 24 * 75`), which also expires one-time files that were never downloaded.
- **One-time uploads** are deleted after the first successful download.
- **Manual deletion** — the optional admin page (`/admin`, enabled by setting `adminPass`)
  lists the buckets and can delete them, including all their files.
- **Logs** — the access log lives in the DockerEngine log stream and follows its rotation
  settings, not this package.
- **Uninstalling** the app keeps the data folder unless you choose “delete data”.

## 4. Your rights — access, correction, deletion

This applies to you as the NAS owner and to the people who upload files to you.

| Right | How |
|---|---|
| **Access** | The uploader sees and downloads their own uploads through the share link they received. As the NAS owner you can read `/Volume1/DockerAppData/psitransferdocker/data` (SSH/SFTP) or enable the admin page by setting `adminPass` in the container environment; it lists every bucket, its files, size and expiry. |
| **Correct** | Files cannot be edited in place by design (an upload is a snapshot). “Correct” means: delete the bucket and upload again, or the uploader re-uploads a corrected file. Retention and the admin password are corrected in the container environment. |
| **Delete** | Uploader: re-open the share link and use “delete” (allowed until expiry). NAS owner: admin page → delete bucket, or remove `/Volume1/DockerAppData/psitransferdocker/data/<bucket-id>` directly; deleting the whole data folder removes every transfer. |
| **Restrict** | Per-bucket password (`requireBucketPassword`), upload password (`uploadPass`), admin password (`adminPass`), per-upload retention and the 75-day maximum age; keep the port on the LAN or behind a TLS reverse proxy. |
| **Assistance / complaints** | E-mail **moechz@163.com** with the subject “Psitransfer data request”; a reply follows within 30 days. |

**Deletion channel.** The developer holds no copy of your data, so deletion is completed on
your own NAS (self-service: share-link delete, admin page, or removing the folder), which is
immediate and complete. The e-mail address above is the channel for questions, for requests
about the website, the store listing and the publishing records, and for anyone who uploaded
a file and no longer has the link — the NAS owner can always delete it locally.

## 5. Security measures

- The container runs as **uid 1000** (`user: "1000:1000"`), never as root; it is **not**
  privileged, does **not** use host networking, does **not** mount the Docker socket and adds
  no capabilities.
- Exactly **one** port is published (host `8045` → container `3000`). Nothing else is exposed.
- The image is pinned to `psitrax/psitransfer:v2.4.1` (Docker Hub, used unmodified); the
  package is rebuilt when upstream ships fixes.
- Data is confined to the application data folder and owned by the app user; no secrets are
  baked into the package.
- The web UI is **plain HTTP**. Keep it on the LAN or put it behind a TLS reverse proxy, and
  set `adminPass`/`uploadPass` if other people can reach the port.
- The `/admin` page is **disabled by default** (`adminPass: false`). Without it, deleting
  files happens on the file system as the NAS administrator.
- Uploaded content is served from your NAS; nothing is scanned for malware unless you add
  your own scanner. Only accept files from people you trust.

## 6. Third-party sharing and API disclosure

The developer shares nothing and operates no server in this path. Psitransfer has no
telemetry and no automatic update check.

| Service | Endpoint | Data sent | Destination / region |
|---|---|---|---|
| The uploader's browser → your app | `http://<NAS-IP>:8045` | the file they upload, its name/size, the chosen retention | your own NAS / LAN |
| Download link you share | same address, `/download/<id>` | the file, to whoever holds the link | your own NAS |
| Docker image pull | `registry-1.docker.io` or your registry mirror | anonymous pull request | Docker Hub, US / your mirror |
| Optional **webhooks** you configure (`fileUploadedWebhook`, `fileDownloadedWebhook`) | the URL you enter | file name, size, bucket id, client IP | the endpoint you chose |
| Optional mail link (`mailTemplate`) | your own mail client — the app never sends e-mail itself | — | — |
| Update check | **none** | — | — |

With the defaults, the app talks to nothing except the browser that opened it and the
registry that pulled the image.

## 7. Children

The app is a self-hosted file-transfer utility and is not directed at children. It has no
account system and no advertising.

## 8. Changes to this policy

Material changes are published in this file and in the repository
(<https://github.com/Moechz/psitransfer>), and are reflected in the “Last updated” date above.
The policy shipped with a given package version is the one that applies to it.

## 9. Contact

Moechz — **moechz@163.com** — <https://www.moechz.com/privacy.html>

Package repository: <https://github.com/Moechz/psitransfer> ·
Upstream project: <https://github.com/psi-4ward/psitransfer> (BSD-2-Clause)
