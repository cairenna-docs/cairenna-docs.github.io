---
title: Local Development Overview
nav_order: 10
---

This section outlines every way you can run Cairenna on a development machine. Each
option can be used independently, depending on whether you want to exercise the full
web experience, run the ECA in isolation, explore mobile clients, or experiment with
Unity builds.

## Prerequisites

- Python 3.11+
- Node.js 18+ (for the React Native client)
- Docker (for container demos)
- Expo CLI (optional, for the mobile app)
- Unity 6000.0.29f1 (only if you plan to open the project in the editor)

{: .note }
The Django backend, `eca_core` package, Unity player, and React Native app all share
the same `OPENAI_API_KEY` (and optional `ELEVENLABS_API_KEY.`) Make sure they are set
before you test the ECA.

## Option A — Django development server (full web stack)

1. Create and activate a virtual environment.
2. Install dependencies:
   ```bash
   pip install -r App/requirements.txt
   ```
3. Apply migrations and seed therapists (optional):
   ```bash
   python App/manage.py migrate
   python App/manage.py load_therapists --clear
   ```
4. Run the server:
   ```bash
   python App/manage.py runserver 0.0.0.0:8000
   ```

Visit `http://localhost:8000` to onboard, complete screenings, and launch CBT/MI
sessions. The `/api/eca/*` endpoints power the in-browser chat UI as well as mobile
and Docker clients.

## Option B — React Native mobile client

1. Start the Django server (Option A).
2. In `mobile/`, install dependencies and launch Expo:
   ```bash
   npm install
   export EXPO_PUBLIC_DJANGO_BASE_URL="http://<HOST>:8000"
   npm run start
   ```
3. Use Expo Go or an emulator to run the app. Sign in with the credentials you created on
   the Django server and navigate links rendered via `react-native-render-html`. Links to
   `/sessions/eca/` open the native ECA session screen.

## Option C — Dockerized Unity + ECA demos

Build the images (Unity license required for CI/CD builds):

```bash
docker build -f docker/Dockerfile.cbt -t cairenna-cbt .
docker build -f docker/Dockerfile.mi -t cairenna-mi .
docker build -f docker/Dockerfile.unified -t cairenna-session .
```

Run a container:

```bash
docker run --rm -e OPENAI_API_KEY=... cairenna-cbt
```

The container launches the headless Unity player and executes the scripted CBT
conversation using `scripts/demo_session.py`. The MI variant works the same way. Use
`cairenna-session` with `SESSION_TYPE=cbt` or `SESSION_TYPE=mi` to choose at runtime.

## Option D — CLI demo without Unity

If you only need to validate the therapy prompts, run the CLI script directly:

```bash
python scripts/demo_session.py cbt
python scripts/demo_session.py motivational_interviewing
```

This exercises `eca_core` independent of Unity or any web client.

## Option E — Unity editor

Opening the scene in Unity lets you iterate on visuals or animations:

1. Install Unity 6000.0.29f1.
2. Open `Full_UnityEnvironment/` with Unity Hub.
3. Clear generated folders (`Library/`, `Logs/`, `.vs/`, etc.) before committing.

Headless builds for Docker are generated automatically via the `CairennaBuild` editor
script (`Assets/Editor/BuildScripts/CairennaBuild.cs`).
