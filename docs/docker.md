---
title: Docker Images
nav_order: 60
parent: Architecture Overview
---

Docker images encapsulate the Unity headless player and the ECA runtime for easy demo
deployments or automated pipelines.

## Build outputs

- `docker/Dockerfile.cbt` – CBT-only image (headless Unity + scripted CBT session).
- `docker/Dockerfile.mi` – MI-only image.
- `docker/Dockerfile.unified` – runtime selection via `SESSION_TYPE` env var.
- `docker/entrypoint/*.sh` – orchestrate Unity player, transcript logging, and Python CLI.

All Dockerfiles share a build stage that uses `unityci/editor:<version>-linux-il2cpp` to
produce a Linux headless build via the `CairennaBuild.BuildLinuxHeadless` editor script.

## Building images

```bash
docker build -f docker/Dockerfile.cbt -t cairenna-cbt .
docker build -f docker/Dockerfile.mi -t cairenna-mi .
docker build -f docker/Dockerfile.unified -t cairenna-session .
```

Set `UNITY_LICENSE_FILE` or use Unity CI credentials as required for automated builds.

## Running containers

```bash
docker run --rm \
  -e OPENAI_API_KEY=... \
  cairenna-cbt
```

Environment variables:

- `OPENAI_API_KEY` – required.
- `ELEVENLABS_API_KEY` – optional; enables TTS.
- `SESSION_TYPE` – (`cbt` or `mi`) accepted by the unified image.
- `PARTICIPANT_NAME`, `PARTICIPANT_CONTEXT`, `SESSION_GOAL`, `DIMENSION_COUNT` – custom
  metadata consumed by `demo_session.py`.
- `RESPONSES_FILE` – path to a newline-delimited list of scripted participant responses.

Unity logs are written to `/var/log/cairenna/unity.log`. Mount the directory to persist
them:

```bash
docker run --rm \
  -v $(pwd)/logs:/var/log/cairenna \
  cairenna-session
```

## When to use Docker

- Share reproducible demos without installing Unity.
- Run regression checks on the conversation engine.
- Capture transcripts + Unity performance metrics in CI.
