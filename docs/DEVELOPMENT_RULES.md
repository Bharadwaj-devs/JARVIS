# JARVIS Development Rules

## Purpose

This document is the engineering contract for coding agents working on the JARVIS Mark LIV foundation.

The repository should evolve incrementally. Preserve working Mark LIV machinery unless there is a demonstrated reason to change it.

## Source of truth

Use the existing repository, Git history, tests, diagnostics, and this document together.

The architecture blueprint is the authoritative high-level design specification.

Do not invent missing architecture from assumptions. Inspect the implementation first.

## Non-negotiable rules

1. Do not replace working machinery unnecessarily.
   Preserve the existing Gemini Live flow, tool/action architecture, memory system, UI, wake-word system, configuration system, session handling, and dynamic prompt-token mechanism unless a specific task requires a change.

2. Build and validate in phases.
   Work on one bounded task at a time. Do not implement the whole roadmap in one change.

3. No unverified success.
   A tool returning successfully is not proof that the requested real-world state was achieved. Verify stateful actions where practical.

4. No fake user turns.
   Do not inject free-form spoken text into the live conversation as a substitute for the real user while a tool-response pipeline is active.

5. No runaway loops.
   Watchdogs, recovery, proactive behaviour, and autonomous work must be bounded and one-shot where appropriate.

6. Preserve user control.
   No unrestricted autonomous source-code rewriting. Behavioural learning may use controlled memory, lessons, preferences, metrics, and user-approved routines, but must not rewrite core identity, safety boundaries, or fundamental execution rules.

7. Keep background work subordinate to interactive work.
   Background intelligence must yield to the primary conversational pipeline and must not block or corrupt it.

8. Prefer the smallest safe change.
   Reuse existing mechanisms before introducing new abstractions.

## Required execution discipline

For each task:

INSPECT -> DIAGNOSE -> PLAN -> MINIMAL CHANGE -> TEST -> REVIEW DIFF -> COMMIT

Do not skip diagnosis when the task is a bug investigation.

Before editing:
- identify the owner of the behaviour;
- trace all relevant state transitions and callbacks;
- inspect synchronization/threading boundaries;
- identify existing configuration and helper mechanisms;
- check related known bugs/issues when relevant.

During editing:
- keep the change scoped to the task;
- avoid unrelated formatting or refactors;
- do not silently change thresholds, APIs, prompts, or architecture outside the task;
- do not add dependencies unless necessary and justified.

After editing:
- run syntax/import validation;
- run focused tests or a reproducible manual check;
- inspect the Git diff;
- report exactly what changed and what was verified;
- commit only the completed bounded change.

## Failure handling

When an implementation attempt fails:

STOP -> reproduce -> identify the shared failure boundary -> determine root cause -> make the smallest corrective patch -> retest.

Do not stack speculative fixes on top of an unverified failure.

## Evidence levels

Use this hierarchy when reporting completion:

- Level 0: the model/tool says it happened.
- Level 1: the tool returned success.
- Level 2: the expected process/resource exists.
- Level 3: the requested state is directly observed.
- Level 4: the requested state persists after a stability check.

Prefer the highest practical evidence level.

## Agent handoff

An agent may be replaced at any time.

Never rely on hidden context from a previous agent conversation. A handoff must be understandable from:
- the current Git commit;
- the current phase/task document;
- the previous agent's written findings;
- the files changed;
- the tests and reproduction steps.

The next agent must inspect the current repository before modifying it.

## Current priority

Reliability comes before major new capabilities, advanced autonomy, proactive behaviour, polished HUD work, and broad self-improvement.

The immediate work is Phase 1 reliability.