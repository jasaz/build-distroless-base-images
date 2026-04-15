# Build Distroless Base Images

This repository contains [apko](https://github.com/chainguard-dev/apko) specs for distroless base images and the GitHub Actions workflows that build, scan, publish, and prune them.

## Image specs

### [wolfi/](wolfi/)

Wolfi-based apko specs. All Python variants run as `nonroot` (uid/gid `65532`) with entrypoint `/usr/bin/python3` and target `x86_64` + `aarch64`.

| Spec | Packages |
| --- | --- |
| [wolfi-python3-12.yaml](wolfi/wolfi-python3-12.yaml) | `python-3.12`, `busybox` |
| [wolfi-python3-12-pip.yaml](wolfi/wolfi-python3-12-pip.yaml) | `py3.12-pip`, `python-3.12`, `busybox` |
| [wolfi-python3-13.yaml](wolfi/wolfi-python3-13.yaml) | `python-3.13`, `busybox` |
| [wolfi-python3-13-pip.yaml](wolfi/wolfi-python3-13-pip.yaml) | `py3.13-pip`, `python-3.13`, `busybox` |
| [wolfi-python3-14.yaml](wolfi/wolfi-python3-14.yaml) | `python-3.14`, `busybox` |
| [wolfi-python3-14-pip.yaml](wolfi/wolfi-python3-14-pip.yaml) | `py3.14-pip`, `python-3.14`, `busybox` |
| [wolfi-jre17.yaml](wolfi/wolfi-jre17.yaml) | `openjdk-17-jre`, `openjdk-17-default-jvm`, `glibc-locale-en`, `libstdc++` — runs as `java`, entrypoint `/usr/bin/java`, `x86_64` only |
| [wolfi-jre-libjpeg-17.yaml](wolfi/wolfi-jre-libjpeg-17.yaml) | JRE 17 variant bundling `libjpeg` |

### [alpine/](alpine/)

- [alpine-py3-10-pip.yaml](alpine/alpine-py3-10-pip.yaml) — Alpine edge + `python3=~3.10`, runs as `non-root` (uid/gid `65532`), entrypoint `/bin/sh -l`, `x86_64`.

## Workflows

### [wolfi-base-image-builder.yaml](.github/workflows/wolfi-base-image-builder.yaml)

Reusable `workflow_call` that builds one image spec. Inputs:

- `file-path` — directory containing the spec (e.g. `./wolfi/`).
- `file-name` — apko spec filename.
- `base-image-name` — image name used for local tag and pushed tag.
- `repo-name` — target Docker Hub repository.

Steps: pull `cgr.dev/chainguard/apko` → `apko build` → `docker load` → Trivy scan (report written to `vulnerabilities-<base-image-name>`) → Docker Hub login → push two tags:

- **Stable:** `<DOCKERHUB_USERNAME>/<repo-name>:<base-image-name>` (overwritten each build).
- **Audit:** `<DOCKERHUB_USERNAME>/<repo-name>:<base-image-name>-YYYYMMDD` (immutable, for rollback).

Required secrets: `DOCKERHUB_USERNAME`, `DOCKERHUB_TOKEN`.

### [wolfi-base-image-deployer.yaml](.github/workflows/wolfi-base-image-deployer.yaml)

Runs on push to `main` (ignoring `README.md` and `LICENSE`). Fans out a matrix over the six Wolfi Python specs (3.12, 3.13, 3.14 — each with and without `pip`) into the builder workflow, pushing to the `wolfi-python` Docker Hub repository.

### [wolfi-image-cleanup.yaml](.github/workflows/wolfi-image-cleanup.yaml)

Scheduled daily at `06:00 UTC` (also `workflow_dispatch`). Authenticates to Docker Hub, paginates through all tags of `<DOCKERHUB_USERNAME>/wolfi-python`, and deletes any tag matching the `-YYYYMMDD` audit suffix that is older than `RETENTION_DAYS` (default `7`).
