<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="assets/icon-dark.svg">
    <img src="assets/icon.svg" alt="MinIO logo" width="128">
  </picture>
</p>

<h1 align="center">aac-minio</h1>

<p align="center">Self-hosted deployment definition for MinIO, an S3-compatible object storage server.</p>

<p align="center">
  <a href="https://github.com/minio/minio"><img alt="Upstream" src="https://img.shields.io/badge/upstream-minio%2Fminio-blue?logo=github"></a>
  <img alt="Image" src="https://img.shields.io/badge/image-quay.io%2Fminio%2Fminio%3Alatest-2496ED?logo=docker&logoColor=white">
  <a href="https://github.com/minio/minio/blob/master/LICENSE"><img alt="License" src="https://img.shields.io/badge/license-AGPL--3.0-green"></a>
</p>

## About

MinIO is a high-performance object storage server that exposes an Amazon S3 compatible API.
It runs on your own hardware and is used wherever applications or tools expect an S3
endpoint, for example for backups, artifacts or Terraform remote state. A built-in web
console provides browser-based access to buckets and objects.

Key features (as provided by upstream):

- S3-compatible API for existing S3 clients and SDKs
- Built-in web console for managing buckets, objects and users
- Server-side encryption and access policies
- Single binary, deployable as a container on a single node

## Upstream

- Project: <https://github.com/minio/minio>
- Website: <https://min.io>
- Documentation: <https://min.io/docs/minio/linux/index.html>
- Container image: `quay.io/minio/minio:latest` (tag as defined in `service.yml`)
- License: AGPL-3.0 (upstream project)

## Quick start (Docker Compose)

The Compose file is rendered from `service.yml` by the App-as-Code template engine; this
repository contains no hand-written Compose file. The rendered service joins the networks
`secured` and `stack_internal`, so create them first if they do not exist yet:

```bash
docker network create secured
docker network create stack_internal

# Provide the secrets listed under "Configuration" in the stage env files
docker compose up -d
```

Secrets are not stored in this repository. Provide them through the deployment pipeline
or create the environment files yourself (see below). Do not commit them.

## Configuration

Environment variables (defaults from `service.yml`; `CHANGE_ME` must be set per deployment):

| Variable | Default | Description |
|---|---|---|
| `MINIO_ROOT_USER` | `CHANGE_ME` (secret) | Root user name; also the access key used by S3 clients with full rights |
| `MINIO_ROOT_PASSWORD` | `CHANGE_ME` (secret) | Root password; also the secret key for the root user |
| `MINIO_BROWSER_REDIRECT_URL` | set per stage | Public URL of the web console (e.g. `https://minio.example.com`). Must match the console URL exactly, otherwise the login redirect fails behind a reverse proxy |
| `TZ` | `Europe/Berlin` | Time zone |

Command: `server /data --console-address :9001`

Ports:

| Port | Protocol | Purpose |
|---|---|---|
| `9000` | TCP | S3 API (`api`), published on host port 9000 |
| `9001` | TCP | Web console (`console`), published on host port 9010 |

Volumes:

| Mount | Description |
|---|---|
| `data` -> `/data` | Persistent object storage (bind mount from `/export/docker/aac-minio/data` on the host) |

Routing: the web console is exposed through Traefik (entrypoint `websecure`, service port
9001). DNS records are created automatically through the AutoDNS integration.

## Deploy with App-as-Code

1. Edit `service.yml`. It is the single source of truth for image, ports, environment,
   volumes, dependencies and per-stage overrides. Do not edit the rendered output.
2. Pushing to `main` (or a configured stage branch) runs the service pipeline, which renders
   the manifests with `aac-template-engine` and triggers the deployment.
3. Releases of this definition are published under <https://github.com/app-as-code/aac-minio>.

## Credits and license

MinIO is developed by MinIO, Inc. and the open-source community and licensed under AGPL-3.0.
This repository only contains the deployment definition for it and is not a fork of the
application. The logo is provided by the MinIO project; see
[`assets/ICON_SOURCE.md`](assets/ICON_SOURCE.md) for icon sources and licenses.
