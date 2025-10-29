---
title: React Native Client
nav_order: 50
parent: Architecture Overview
---

The Expo-based mobile app provides a native wrapper around the Django experience
while exposing a dedicated ECA chat interface.

## Stack

- Expo SDK 51, React Native 0.73.
- React Navigation (native stack).
- `@tanstack/react-query` for data fetching/caching.
- `react-native-render-html` to render Django pages.
- Context provider that manages Django session cookies & CSRF tokens.

## Session management

`DjangoSessionProvider` handles:

- Fetching CSRF tokens by scraping the `/login/` page.
- Sending login POST requests with form data.
- Maintaining cookies using `credentials: 'include'`.
- Fetching HTML pages and updating authentication state automatically.
- Posting JSON payloads to `/api/eca/*` with the stored CSRF token.

## Screens

- `LoginScreen` – native email/password form; on success navigates to the dashboard.
- `HtmlScreen` – renders any Django route, intercepts anchor tags, and pushes new
  screens inside the navigation stack.
- `EcaSessionScreen` – native CBT/MI controls similar to the web chat but fully
  implemented in TypeScript.

## Running the mobile app

1. Start the Django server (see [Local Development Overview](local-development.md)).
2. In `mobile/`, run:
   ```bash
   npm install
   export EXPO_PUBLIC_DJANGO_BASE_URL="http://<HOST>:8000"
   npm run start
   ```
3. Launch Expo Go or an emulator to test the app. The ECA screen calls the same
   `/api/eca/start|respond|end` endpoints as the web client.

{: .note }
If you need to point the app to a staging server, update `EXPO_PUBLIC_DJANGO_BASE_URL`
and rebuild the project (or restart Expo with the new env var).
