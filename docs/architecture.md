---
title: Architecture Overview
nav_order: 20
has_children: true
---

The Cairenna platform is composed of modular services that can be deployed together
or independently. This page summarises each layer and the data flow between them.

## High-level diagram

1. **Django (`App/`)**: REST + HTML backend, hosts onboarding, questionnaires, dashboards,
   therapist matching, and the `/api/eca/*` endpoints.
2. **Therapy runtime (`eca_core/`)**: Shared Python package used by Django, Docker images,
   CLI demos, and the mobile app to run CBT/MI sessions.
3. **Unity (`Full_UnityEnvironment/`)**: Renders the Tessa ECA and communicates with the
   runtime via ZeroMQ (when running interactively). Docker builds create a headless player.
4. **Mobile (Expo)**: React Native shell that wraps Django HTML and integrates with the
   JSON APIs for native CBT/MI experiences.
5. **Docker images**: Bundle the Unity headless player and CLI runtime for quick demos or
   non-interactive pipelines.

Data flows are orchestrated by Django. Questionnaire responses feed the `DimensionScore`
model, which is used to seed ECA sessions and therapist matching. Session transcripts and
summaries are stored in `ECASessionRecord` and can be surfaced in clinical reports or APIs.

## Key components

### Django apps

- `core/forms.py` – onboarding, questionnaire, and profile settings forms.
- `core/views.py` – page views plus ECA JSON endpoints.
- `core/services.py` – scoring logic for questionnaires, therapist matching, and reports.
- `core/eca.py` – bridge that instantiates `eca_core.SessionManager`, persists transcripts,
  and controls session lifecycle.

### eca_core package

- `config.py` – centralises environment configuration (LLM, speech, animation).
- `sessions/` – implements `CognitiveBehavioralTherapySession` and
  `MotivationalInterviewingSession` atop `BaseSession`.
- `io/` – adapters for OpenAI, ElevenLabs, Whisper, and ZeroMQ animation bridge.
- `resources/` – prompt templates and sample data for MI/CBT flows.
- `runtime.py` – `SessionManager` for creating and tracking sessions.

### Unity project

- `Assets/Editor/BuildScripts/CairennaBuild.cs` – headless Linux build pipeline.
- `Assets/` – Tessa avatar, animations, scenes, and audio logic.
- `Library/`, `.vs/`, `Logs/`, etc. – generated; exclude from source control.

### React Native client

- `src/context/DjangoSessionProvider.tsx` – manages cookies, CSRF tokens, and API calls.
- `src/screens/HtmlScreen.tsx` – renders Django pages using `react-native-render-html`.
- `src/screens/EcaSessionScreen.tsx` – native chat UI for `/api/eca/*` endpoints.

### Docker build artefacts

- `docker/Dockerfile.cbt` & `.mi` – build Unity, copy runtime, run scripted session.
- `docker/Dockerfile.unified` – runtime parameter to pick MI or CBT at launch.
- `docker/entrypoint/*.sh` – start Unity headless player and Python script.

## External services

- **OpenAI** – chat completions (MI/CBT prompts) + Whisper transcription.
- **ElevenLabs** (optional) – speech synthesis for agent replies.
- **ZeroMQ** – used by Unity runtime for real-time animation/audio cues in interactive
  configurations.

Configure credentials via environment variables documented on the
[Environment Reference](environment.md) page.
