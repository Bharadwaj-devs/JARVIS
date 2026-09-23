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

## First task

Diagnosis and instrumentation only.

Do not implement the fix until the diagnostic evidence has been reviewed.

## Diagnostic questions

Trace every path that can move the assistant between SLEEPING and AWAKE.

Determine:
- which code owns the authoritative awake/sleep state;
- every caller that can wake or sleep JARVIS;
- whether UI callbacks, wake-word callbacks, automatic sleep, or system/session logic can race;
- whether multiple threads/callbacks mutate the same state;
- whether stale wake-word frames or queued detection events survive a manual sleep;
- whether already-captured audio can trigger a wake after sleeping;
- whether the wake detector remains active after sleeping;
- whether duplicate UI events can toggle the state twice;
- whether there is any debounce, cooldown, generation, or session invalidation mechanism;
- which synchronization primitives protect shared state.

## Required diagnostics

Add only minimal, thread-safe diagnostic logging needed to establish the transition path.

For each state transition, capture:
- requested transition;
- previous state;
- new state;
- source;
- thread;
- timestamp.

Classify wake sources as:
- UI/manual;
- wake-word;
- automatic/system;
- unknown.

Classify sleep sources as:
- UI/manual;
- automatic sleep;
- shutdown/session;
- unknown.

For wake-word events, capture:
- timestamp;
- current sleep/awake state;
- detector enabled/disabled state;
- accepted/rejected;
- rejection reason.

Never log API keys, private conversation content, or raw audio.

## Scope restrictions

For this diagnostic task, do not:
- redesign the architecture;
- change wake-word thresholds;
- change microphone behaviour;
- change the Gemini Live session architecture;
- change unrelated tools;
- fix other upstream issues;
- implement the final race fix before diagnosis is reviewed.

## Completion criteria

The task is complete only when the diagnostic work produces a clear evidence trail showing the relevant SLEEPING <-> AWAKE transition path.

The agent report must include:
1. files inspected;
2. files changed;
3. state-transition paths found;
4. relevant threads/callbacks;
5. exact diagnostic sequence observed during reproduction, when reproducible;
6. likely race boundary only when supported by logs/code;
7. remaining uncertainty;
8. syntax/import validation performed.

## Next step

After this diagnosis is reviewed, create a separate bounded task for the smallest safe corrective change.

Do not combine diagnosis and the final fix into one uncontrolled change.