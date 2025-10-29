---
title: Environment Reference
nav_order: 90
---

Use this table to configure local or deployed environments. All values can be supplied
via `.env`, shell variables, or container environment definitions.

| Variable | Description | Required By |
| --- | --- | --- |
| `DJANGO_SECRET_KEY` | Django secret key | Django |
| `DJANGO_DEBUG` | `True`/`False` for debug mode | Django |
| `DJANGO_ALLOWED_HOSTS` | Comma-separated hostnames | Django |
| `DJANGO_DB_ENGINE/NAME/USER/PASSWORD/HOST/PORT` | Database settings | Django |
| `OPENAI_API_KEY` | OpenAI authentication for LLM + Whisper | Django, Docker, CLI, Mobile |
| `ECA_LLM_MODEL` | Override default OpenAI model | All ECA runtimes |
| `ELEVENLABS_API_KEY` | ElevenLabs TTS (optional) | All ECA runtimes |
| `ECA_TTS_VOICE_ID` | Voice selection for ElevenLabs | Optional |
| `ECA_ANIMATION_PUSH_ADDR` | ZeroMQ address for Unity animation bridge | Unity interactive builds |
| `EXPO_PUBLIC_DJANGO_BASE_URL` | Base URL the mobile app points to | React Native |
| `SESSION_TYPE` | CBT/MI selection for unified Docker image | Docker (unified) |
| `PARTICIPANT_NAME`, `PARTICIPANT_CONTEXT`, `SESSION_GOAL`, `DIMENSION_COUNT` | Optional metadata for demo sessions | Docker, CLI |

{: .note }
When running Expo against a local Django server from an Android emulator, use
`http://10.0.2.2:8000` (Android) or `http://127.0.0.1:8000` (iOS simulator) as the base URL.

## Secrets management

- For local development, create `App/.env` mirroring `.env.example` and add the required keys.
- Production deployments should use a secret store (AWS SSM, GCP Secret Manager, etc.).
- Expo can use `.env` files with the `EXPO_PUBLIC_*` prefix or rely on build-time variable
  injection.
