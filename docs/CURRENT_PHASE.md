# JARVIS Current Phase

## Baseline

Version: v0.1.0
Baseline release: Mark LIV Foundation
Repository: Bharadwaj-devs/JARVIS

The v0.1.0 release is the clean Mark LIV foundation. Preserve that working architecture while reliability work is introduced incrementally.

## Current phase

Phase 1A — Sleep/Wake + Input Reliability

## Current objective

Diagnose and then fix the reported sleep/wake state transition bug:

AWAKE -> manual Sleep -> SLEEPING -> sometimes immediately AWAKE

The unintended transition from SLEEPING back to AWAKE is the bug. It may repeat several times after a manual Sleep command.

Once JARVIS remains in SLEEPING, that is the expected behaviour. Saying the wake word afterward should wake JARVIS normally.

A related upstream Mark LIV issue concerns unreliable sleep/wake resumption. Treat that as context, not proof of the local root cause.

## Task Status

- **Task 1 — Sleep/Wake Race Reproduction + Instrumentation: COMPLETE.**
- **Task 2 — Sleep/Wake State Fix: COMPLETE.**
- **Task 3 — Capture-Time Boundary Fix: IMPLEMENTED, pending physical validation.**
- **Manual Sleep Reliability: FIXES IMPLEMENTED, pending physical validation.**

## Confirmed Root Causes

1. **Epoch-stamp race (fixed in Task 2):** The microphone callback was stamping an in-flight audio frame with the later gate-check `epoch` (captured after the sleep transition) instead of the callback-entry `entry_epoch` (captured when the frame arrived). This allowed stale audio captured before manual Sleep to carry the post-sleep epoch value, bypassing the existing stale-epoch rejection in `_on_wake_detected()` and `wake()`.

2. **Capture-time boundary (fixed in Task 3):** A sounddevice callback block can arrive AFTER the Sleep transition while containing audio samples captured BEFORE Sleep. The callback-entry epoch is insufficient to identify the true audio capture boundary because a 1024-sample / 16 kHz block spans ~64 ms. Physical runlog showed manual Sleep occurred ~58 ms before the failing callback, meaning the block straddled the Sleep boundary.

## Fixes Applied

### Fix 1 — Epoch Stamp (main.py:1583)
**Change:** `det.feed(indata, epoch=epoch)` → `det.feed(indata, epoch=entry_epoch)`

Ensures audio frames are tagged with the epoch at callback entry, so the detector's existing epoch validation correctly rejects frames captured before a manual Sleep transition.

### Fix 2 — Capture-Time Boundary (main.py)
**New state:** `_stream_time_base`, `_sleep_stream_time` (PortAudio stream time ↔ Python monotonic correlation)

**Implementation:**
- Stream-time correlation established on first callback of each InputStream lifetime using `time_info.currentTime`
- Correlation reset when new InputStream is created
- Sleep boundary recorded in stream time at Sleep transition: `sleep_stream_time = monotonic - stream_time_base`
- In callback, while sleeping: compute `block_start = inputBufferAdcTime`, `block_end = block_start + frames/SEND_SAMPLE_RATE`
- Drop if `block_end <= sleep_boundary` (entirely pre-Sleep) or `block_start < sleep_boundary` (straddles)
- Feed normally if `block_start >= sleep_boundary` (entirely post-Sleep)
- Wake clears sleep boundary
- No debounce/cooldown/threshold changes

## Phase 1A Completion Status

**Phase 1A is NOT COMPLETE yet.**

Completion requires successful **physical validation** of:
1. Manual SLEEP NOW succeeds reliably (no multi-click requirement)
2. Explicit manual sleep cannot be immediately undone by an in-flight/stale wake event
3. Repeated wake/sleep cycles behave consistently
4. Wake word still wakes JARVIS normally after it is asleep
5. No duplicate wake transition from a single wake phrase
6. UI reflects the authoritative core state
7. No regression to the corrected sleeping microphone routing (Task 2 validated behavior)

Physical validation has NOT been performed yet. This must be done on hardware with the actual microphone and wake-word detector running.

## Scope restrictions (remain in effect)

For this phase, do not:
- redesign the architecture
- change wake-word thresholds
- change microphone behaviour
- change the Gemini Live session architecture
- change unrelated tools
- fix other upstream issues
- add a second parallel state system
- remove the existing wake epoch/generation protection
- add a new cooldown/debounce system

## Next step

Perform physical validation on hardware. If validation passes, Phase 1A can be marked complete and Phase 1B (Microphone Responsiveness) can begin. If validation reveals issues, diagnose and apply the smallest additional fix.