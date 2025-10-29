---
title: Django Backend
nav_order: 30
parent: Architecture Overview
---

The Django application (`App/`) exposes both HTML views and JSON APIs. It is the canonical
source of truth for user data, screening results, therapist records, and ECA transcripts.

## Applications & structure

- `core/` – main app containing models, forms, views, services, static assets, and templates.
- `manage.py` – standard Django entrypoint.
- `cairenna/settings.py` – project configuration; note the extended `INSTALLED_APPS` and
  path injection for `eca_core`.

### Models

- `Profile` – extended user metadata captured during onboarding.
- `ScreeningSession` & `QuestionnaireResponse` – store questionnaire submissions, allowing
  repeated screenings to be tracked over time.
- `DimensionScore` – severity across 25 mental health dimensions; used for therapist matching
  and to seed ECA sessions.
- `Therapist` & `TherapistMatch` – scraped provider data plus personalised matches.
- `ClinicalReportSnapshot` – cached summary used for clinician handoff.
- `ECASessionRecord` – transcripts, metadata, and status for CBT/MI conversations.

### Services

- `core/services.py`
  - `record_questionnaire_response` – saves answers and computes severity.
  - `build_dimension_insights` – aggregates severity into human-readable cards.
  - `match_therapists_for_user` – scoring algorithm that combines dimension tags, insurance,
    and location.
  - `generate_clinical_report` – summarises longitudinal data for clinician review.
- `core/eca.py`
  - Bridges Django models with `eca_core.SessionManager`.
  - Provides `start`, `respond`, and `end` helpers used by the JSON API.

### Views & routes

- HTML views for login, onboarding, dashboard, questionnaires, therapist matching, reports,
  settings, and the ECA chat shell (`ECASessionPageView`).
- JSON API endpoints (all CSRF-protected):
  - `POST /api/eca/start/` – start CBT or MI session (`session_type` payload).
  - `POST /api/eca/respond/` – send participant message, receive agent turns.
  - `POST /api/eca/end/` – gracefully terminate session and persist summary.

### Static assets

- CSS/JS under `core/static/core/`, including `eca-session.css` and `eca-session.js` which
  power the in-browser ECA chat experience.

## Developing against Django

1. Configure environment variables (see [Environment Reference](environment.md)).
2. Run `python manage.py runserver`.
3. Use Django admin (`/admin/`) to inspect or edit questionnaire results, therapist data, and
   ECA transcripts.
4. When testing MI/CBT flows, watch the server console for OpenAI responses and verify that
   `ECASessionRecord` entries are created as expected.

### Testing without Unity

You can exercise the same logic via cURL or HTTP clients:

```bash
curl -X POST http://localhost:8000/api/eca/start/ \
  -H "Content-Type: application/json" -H "X-CSRFToken: <token>" \
  -d '{"session_type": "cbt"}'
```

Subsequent calls to `/api/eca/respond/` with the returned `session_id` will stream agent
messages identical to the web/mobile chat.
