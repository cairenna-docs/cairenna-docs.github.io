---
title: ECA Runtime (`eca_core`)
nav_order: 40
parent: Architecture Overview
---

`eca_core` is a reusable package that encapsulates all therapy-specific logic.
It exposes high-level session classes, prompt templates, IO adapters, and a
`SessionManager` that keeps runtime instances alive.

## Session types

- `CognitiveBehavioralTherapySession` – Implements a three-stage CBT flow:
  1. Identify unhelpful thought.
  2. Challenge with evidence-based questioning.
  3. Reframe into constructive alternative.
- `MotivationalInterviewingSession` – Follows MI phases: engage, focus, evoke, plan.
  Each phase uses tailored prompts to elicit change talk and reflections.

Both extend `BaseSession`, which handles transcript management, speech synthesis hooks,
and stage transitions. The runtime populates transcripts with `Utterance` objects that
include metadata such as role, stage, and timestamps.

## Key modules

- `sessions/base.py` – lifecycle hooks (`start`, `handle_response`, `end`), helper methods,
  and transcript serialization.
- `io/llm.py` – wraps OpenAI Chat Completions with Pydantic response models for CBT/MI
  question generation, evaluation, reflection/validation, and session summarisation.
- `io/speech.py` – optional ElevenLabs TTS + Whisper STT adapters.
- `io/animation.py` – ZeroMQ bridge for Unity animation/audio cues.
- `runtime.py` – `SessionManager` factory that shares IO clients across sessions.
- `resources/prompts/` – markdown templates for each prompt/evaluation class.

## Configuration

`config.RuntimeConfig` aggregates environment variables into structured dataclasses:

- `SpeechConfig` – provider, voice, and model IDs for TTS/STT.
- `LLMConfig` – OpenAI provider/model/temperature.
- `UnityConfig` – ZeroMQ addresses + toggles.
- `DataPaths` – root directories for prompts, templates, and sample data.

Override values via environment variables (e.g., `ECA_LLM_MODEL`, `ECA_TTS_VOICE_ID`)
or instantiate `RuntimeConfig` manually in tests.

## Using the runtime directly

```python
from eca_core import SessionManager, SessionType

manager = SessionManager()
session = manager.create_session("demo", SessionType.CBT, context)
agent_turns = session.start()
participant_turns = session.handle_response("I'm worried about failing")
```

Use `session.summary()` after completion to obtain a clinician-friendly recap. When
integrated inside Django, `ECASessionService` converts these objects into JSON responses
and stores transcripts in the database.
