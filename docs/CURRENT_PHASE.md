# JARVIS Current Phase

## Baseline

Current release: v0.1.1 — Sleep/Wake Reliability
Baseline release: v0.1.0 — Mark LIV Foundation
Repository: Bharadwaj-devs/JARVIS

The v0.1.0 release remains the recoverable clean Mark LIV foundation. v0.1.1 records the completed Phase 1A sleep/wake reliability work.

## Current phase

Phase 1B — Live Voice Responsiveness

## Current objective

Identify exactly where real user speech is discarded, delayed, or prevented from reaching Gemini Live.

Phase 1B begins with instrumentation, not threshold tuning or an audio-path redesign.

## Phase 1A — COMPLETE

Phase 1A — Input + Sleep/Wake Reliability is complete.

Validated outcomes:

- Manual SLEEP NOW remains asleep instead of being reversed by stale/in-flight wake events.
- Wake word still wakes JARVIS after an explicit Sleep.
- The wake/sleep state path and visible UI state remain synchronized with the authoritative core state.
- The corrected sleeping microphone routing remains intact.
- The completed changes were physically validated on the actual Windows microphone/wake-word path.

Release milestone:

`v0.1.1` — Sleep/Wake Reliability

## Phase 1B — Task Status

- **Task 1 — Sleep/Wake Race Reproduction + Instrumentation: COMPLETE.**
- **Task 2 — Sleep/Wake State Fix: COMPLETE.**
- **Task 3 — Microphone Path Instrumentation: NEXT.**
- **Task 4 — Microphone Reliability Fix: NOT STARTED.**
- **Task 5 — Truthful Tool Failure Channel: LATER IN PHASE 1.**

## Phase 1B Scope

Use the existing V3 reliability contract.

Primary files to inspect:

`main.py:_listen_audio`
`main.py:_send_realtime`
`core/echo.py`
`core/audio_devices.py`

Required observability:

```text
mic callback frames received
frames dropped: asleep
frames dropped: speaking
frames dropped: mute
frames dropped: PTT
frames dropped: phone relay
frames dropped: echo guard
frames sent to Gemini
outgoing queue depth / drops
last microphone callback timestamp
last outgoing audio timestamp
last user transcription timestamp
wake detections
wake detections suppressed as stale/cooldown
```

No threshold redesign before the discard/delay boundary is identified.

## Next step

Run the Phase 1B Plan-mode investigation for microphone-path instrumentation. Do not modify Phase 1A wake/sleep code unless the investigation proves a regression there.
