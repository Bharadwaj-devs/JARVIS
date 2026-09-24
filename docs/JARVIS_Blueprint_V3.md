# JARVIS Architecture Blueprint V3 — Mark LIV (54)

**Status:** Engineering specification / working constitution  
**Updated:** 2026-09-24  
**Canonical development repository:** `Bharadwaj-devs/JARVIS`  
**Frozen foundation tag:** `v0.1.0` — Mark LIV Foundation  
**Foundation commit:** `1078760a65e622bb1aa65dad09e0cea953834bfe`  
**Upstream reference:** `FatihMakes/Mark-LIV`

---

## 0. WHY V3 EXISTS

V3 replaces the previous generic V2 phase ordering with an evidence-driven plan based on two things that are now available together:

1. direct inspection of the actual `Bharadwaj-devs/JARVIS` implementation,
2. current upstream Mark LIV issue reports and recent upstream maintenance work.

This matters because the first version of the blueprint described what Mark 54 should eventually become, but it did not yet have a sufficiently detailed map of the exact failure boundaries in the code we are actually going to evolve.

V3 therefore treats the currently observed reliability failures as first-class engineering requirements instead of postponing them behind future architecture.

The development philosophy remains:

```text
inspect -> isolate -> extend -> test -> verify -> commit
```

not:

```text
rewrite -> hope -> patch regressions
```

---

# 1. CURRENT BASELINE — DO NOT LOSE MARK LIV

## 1.1 Canonical baseline

`Bharadwaj-devs/JARVIS` is the canonical development repository.

The repository currently preserves the clean Mark LIV foundation and has a published release tag:

```text
v0.1.0
Mark LIV Foundation
```

All future changes must remain recoverable to that baseline.

## 1.2 Existing architecture confirmed by repository inspection

The following systems already exist and should be reused rather than recreated:

- Gemini Live conversational runtime
- `main.py` session/orchestration layer
- dynamic action discovery through `core/action_loader.py`
- dynamic plugin discovery through `core/plugin_loader.py`
- persistent memory through `memory/memory_manager.py`
- configuration through `memory/config_manager.py`
- shared undo through `core/undo.py`
- user-issued confirmation through `core/confirm.py`
- local wake word through `core/wake_word.py`
- audio device selection through `core/audio_devices.py`
- echo handling through `core/echo.py`
- push-to-talk through `core/hotkey.py`
- screen/camera vision
- existing PyQt6/QPainter HUD in `ui.py`
- remote dashboard in `dashboard/`
- existing proactive/background monitoring
- existing application, browser, file, system and other actions

## 1.3 Existing implementation facts that affect the plan

The inspected repository is not a primitive prototype. It already contains substantial reliability work:

- session resumption,
- explicit turn-completion tracking,
- output interruption and audio draining,
- output latency measurement,
- an echo-tail guard,
- a dedicated wake-word inference thread,
- audio-device probing,
- safe file paths,
- trash-based deletion,
- undo registration,
- UI confirmation for irreversible operations,
- action/plugin isolation through registries.

The correct strategy is therefore **targeted correction of shared boundaries**, not a wholesale architectural rewrite.

---

# 2. ENGINEERING LAW

Every meaningful request should conceptually pass through:

```text
USER INPUT
   ↓
CONVERSATIONAL INTERPRETATION
   ↓
OBJECTIVE / INTENT
   ↓
EXECUTION STATE
   ↓
TOOL / ACTION / PLUGIN
   ↓
OBSERVED RESULT
   ↓
VERIFICATION
   ↓
FINAL STATE
   ↓
RESPONSE
   ↓
MEMORY / METRICS / LESSONS
```

The central rule is:

> **A tool returning successfully is not the same thing as the requested state being true.**

This single distinction drives application verification, file verification, browser verification, setting verification, and eventually the task engine.

---

# 3. NON-NEGOTIABLE CONSTRAINTS

- Do not rewrite working Mark LIV machinery without a demonstrated technical reason.
- Do not implement multiple unrelated architectural layers in one coding-agent task.
- Do not claim an action succeeded without evidence appropriate to that action.
- Do not use a fake user message to report an error inside an active tool-response pipeline.
- Do not treat an existing process as proof that a new application launch occurred.
- Do not treat a tool return string as verification.
- Do not let a background task starve the live voice path.
- Do not allow autonomous loops without timeouts, step limits, cancellation and observability.
- Do not allow the model to forge human confirmation.
- Do not activate risky autonomy or self-modification before reliability is stable.
- Do not silently turn one-off behaviour into permanent preferences.
- Do not let UI state become the source of truth for execution state.
- Do not diagnose a regression by immediately redesigning the entire system.
- Do not merge a reliability change without focused testing and live validation when the feature touches real hardware or OS state.

---

# 4. UPSTREAM MARK LIV ISSUE TRIAGE

The upstream repository currently exposes a large active issue backlog. The issues below are the ones that materially inform the first Mark 54 reliability work.

## 4.1 Immediate reliability signals

### #115 — sleep / wake / resume problems

Reported behaviour includes JARVIS going into system sleep, failing to resume conversation correctly, remaining in an apparent thinking state, and wake-word interactions failing after sleep/resume.

This is directly relevant to our implementation and is now a P0 reliability requirement.

**Reference:** `https://github.com/FatihMakes/Mark-LIV/issues/115`

### #117 — wake-up latency + credential handling

The report explicitly calls out that the wake-up call takes time and needs optimization.

The credential-manager request is important, but it is a later security phase rather than the first reliability fix.

**Reference:** `https://github.com/FatihMakes/Mark-LIV/issues/117`

### #111 — wake-word model load failure

An upstream report shows an `openwakeword` model-construction incompatibility:

```text
AudioFeatures.__init__() got an unexpected keyword argument 'wakeword_models'
```

Therefore wake-word reliability must include dependency/API compatibility checks rather than only threshold tuning.

**Reference:** `https://github.com/FatihMakes/Mark-LIV/issues/111`

### #105 — slow / non-responsive assistant

An upstream report describes the assistant becoming slow or stopping its real-time conversational behaviour.

This supports treating the live voice pipeline and its scheduling boundaries as a first-class subsystem.

**Reference:** `https://github.com/FatihMakes/Mark-LIV/issues/105`

### #94 — stuck thinking / sleeping / initialization

This report concerns startup/session state getting stuck. Upstream discussion specifically points toward initialization, microphone availability, API/network failures, and reconnect behaviour as possible contributors.

**Reference:** `https://github.com/FatihMakes/Mark-LIV/issues/94`

### #88 — microphone not audible / not capturing user speech

This directly matches our own observed user requirement: JARVIS must reliably capture and attend to the user's speech.

**Reference:** `https://github.com/FatihMakes/Mark-LIV/issues/88`

### #18 — queue full / stops responding

An upstream report describes the assistant becoming unresponsive after the audio queue fills.

This makes queue pressure, backpressure and drop-policy observability part of reliability testing.

**Reference:** `https://github.com/FatihMakes/Mark-LIV/issues/18`

### #66 — repeated lifecycle/status loop

An upstream report describes repeated `JARVIS online` / `SLEEPING` / `THINKING` / `LISTENING` transitions.

This is a lifecycle/reconnect integrity problem, not a cosmetic UI problem.

**Reference:** `https://github.com/FatihMakes/Mark-LIV/issues/66`

## 4.2 Truthful computer-state issues

### #90 — JARVIS says Chrome closed when it remains open

This is a direct example of false completion reporting: the assistant's language says the operation happened, but the requested machine state remains unchanged.

**Reference:** `https://github.com/FatihMakes/Mark-LIV/issues/90`

### #108 — incorrect browser/application targeting

The report describes Chrome requests resolving to other files with similar names rather than the intended application.

**Reference:** `https://github.com/FatihMakes/Mark-LIV/issues/108`

### #40 — failed browser action is not retried intelligently

The report describes JARVIS failing to open the browser and then behaving as though the failure became a permanent state until restart.

This informs the recovery model: a recoverable failure should not become a permanent learned assumption, and retries must use evidence and changed strategy rather than blind repetition.

**Reference:** `https://github.com/FatihMakes/Mark-LIV/issues/40`

## 4.3 Browser, camera and tool-state issues

Relevant upstream reports also include:

- #106 — web-app interaction failures, browser-selection mistakes, and camera state closing unexpectedly.
- #107 — web-app/search/play problems and camera closing without user instruction.
- #32 — extra browser tabs/windows being created when only a new tab was requested.
- #31 — browser context/page instability after short operation periods.
- #12 — vision connection failure during screen processing.

These should become computer-intelligence and verification test cases after the core input/reliability path is stable.

---

# 5. OUR DIRECTLY OBSERVED FAILURES — FIRST-CLASS REQUIREMENTS

## 5.1 Sleep button race: sleep immediately wakes again

Current user-observed behaviour:

```text
USER clicks SLEEP
    ↓
JARVIS enters SLEEPING as intended
    ↓
JARVIS sometimes immediately wakes on its own
    ↓
this may repeat several times
    ↓
eventually JARVIS remains SLEEPING normally
    ↓
user says the wake word
    ↓
JARVIS wakes normally
```

The bug is the unintended wake after an explicit Sleep command. Remaining asleep afterward is the expected behaviour.

The same physical control is used for wake and sleep, so state transitions must be atomic and race-safe.

### Current code path inspected

`main.py` exposes:

```python
wake()
sleep()
_ui_wake_manual()
```

and `core/wake_word.py` runs detection in a dedicated thread.

The detector calls `on_detect()` directly from its inference thread. There is currently no explicit detection-generation token, re-arm epoch, or post-sleep suppression window.

### Engineering interpretation

A strong **race candidate** exists:

```text
WakeWordThread is processing a valid wake frame
            ↓
user presses SLEEP
            ↓
sleep() sets _awake = False
            ↓
detector callback arrives from another thread
            ↓
wake() sees _awake == False
            ↓
assistant wakes again
```

Repeated detections are also possible because the detector currently relies mainly on queue draining after a detection and has no explicit cooldown/re-arm window.

This is a hypothesis from code inspection, not a claim that this is the only root cause. The first task must instrument and reproduce the race.

### Required invariant

Once the user explicitly presses SLEEP:

```text
explicit sleep wins over stale wake detections
```

A wake event that began before the explicit sleep command must not undo that user action.

### Required design

Use a small explicit state boundary such as:

```text
WAKE EPOCH / GENERATION
WAKE EVENT TIMESTAMP
LAST MANUAL STATE CHANGE
WAKE COOLDOWN / RE-ARM WINDOW
```

The exact mechanism may differ, but it must guarantee that stale detector events cannot reverse an explicit user state change.

## 5.2 JARVIS sometimes does not hear / attend to the user

This is the highest-priority functional problem after the sleep/wake race.

The current `main.py` mic callback contains multiple legitimate gates:

```text
wake-word sleep gate
speaker-speaking gate
push-to-talk gate
phone-active gate
mute gate
echo-tail gate
```

Those gates make the system sophisticated, but they also create several places where user speech can disappear.

### Current code observations

- microphone input arrives through `sounddevice.InputStream` at 16 kHz,
- the callback may return early while JARVIS is speaking,
- the callback may return early during echo-tail handling,
- the callback may return early while PTT is not held,
- input transcription updates `_last_user_speech`,
- raw mic frames themselves are not currently exposed through a structured diagnostic counter set,
- wake detection occurs in its own thread while asleep.

### Required instrumentation before tuning

The first microphone reliability task must add lightweight counters/diagnostics for:

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

This is not optional polish. Without it, changing thresholds blindly will not reveal where the user's speech disappears.

### Required invariant

When JARVIS is awake, not muted, not using PTT, and no explicit higher-priority audio mode owns the microphone:

```text
real user speech must have a clear path to Gemini
```

A transient audio-processing failure must not silently become permanent deafness.

---

# 6. RELIABILITY PRIORITY ORDER — REVISED PHASE 1

The first phase is now explicitly ordered around the observed failures and upstream issue evidence.

```text
PHASE 1A  INPUT + SLEEP/WAKE RELIABILITY
PHASE 1B  LIVE VOICE RESPONSIVENESS
PHASE 1C  TRUTHFUL TOOL FAILURE HANDLING
PHASE 1D  STATE VERIFICATION
PHASE 1E  RESULT DELIVERY WATCHDOG
PHASE 1F  LIVE REGRESSION / HARDWARE VALIDATION
```

Only after the entire phase passes do we publish `v0.2.0`.

---

# 7. PHASE 1A — INPUT + SLEEP/WAKE RELIABILITY

## 7.1 Manual sleep/wake state machine

Replace implicit toggling semantics with an explicit state model:

```text
SLEEPING
AWAKENING
AWAKE
SLEEPING_BY_USER
```

A lighter implementation is acceptable if it still enforces the same invariants.

### Rules

- user-issued SLEEP has priority over stale wake events,
- user-issued WAKE has priority over ordinary auto-sleep until explicitly superseded,
- wake detector events must be re-armable after manual state changes,
- stale detector events must be suppressible,
- the toggle button must reflect actual core state rather than optimistic UI state,
- concurrent wake/sleep callbacks must be serialized or made generation-safe.

## 7.2 Wake detector hardening

`core/wake_word.py` should gain a state-safe detection contract.

Minimum requirements:

- explicit enable/disable state,
- explicit detector running state,
- wake cooldown / re-arm interval,
- stale-event suppression after manual sleep,
- detector readiness state distinct from package installation,
- compatibility failure reported clearly,
- no duplicate wake transition from one spoken wake phrase.

Do not simply lower the threshold to make wake feel faster.

## 7.3 Wake startup latency

The upstream backlog includes a complaint about wake-up latency.

Measure:

```text
speech reaches detector
        ↓
detection callback
        ↓
_awake becomes true
        ↓
UI becomes LISTENING
```

Optimize only after measuring these intervals.

---

# 8. PHASE 1B — LIVE VOICE RESPONSIVENESS

## 8.1 Microphone observability

Add the counters listed in Section 5.2.

The diagnostic path must remain lightweight enough to run in the audio callback.

## 8.2 Audio queue integrity

The upstream #18 report makes queue pressure a release-gate concern.

Required properties:

- bounded queue,
- explicit drop policy,
- visible queue pressure in diagnostics,
- no unbounded memory growth,
- no silent permanent input starvation,
- queue recovery after interruption or reconnect.

## 8.3 Speaking-state watchdog

The current playback path sets `_is_speaking` and has event-driven clearing, but upstream maintenance has also proposed a speaking watchdog because some tool flows can leave the assistant appearing to speak and consequently keep the microphone gated.

The upstream PR #118 is **open and not merged**. Its speaking-watchdog idea is therefore reference material, not a change to copy blindly.

Required invariant:

```text
no audio activity for a bounded interval
        ↓
_stale speaking state cannot hold forever
```

This must be implemented without clearing the flag while valid audio is still being played.

## 8.4 User-speech capture path

Test these separately:

```text
awake + idle
awake + after JARVIS speech
awake + immediate reply
awake + echo-tail period
awake + PTT disabled
awake + selected microphone
awake + device fallback
```

Each case must record whether mic frames reached Gemini.

---

# 9. PHASE 1C — TRUTHFUL TOOL FAILURE HANDLING

## 9.1 Current defect confirmed

`main.py::_execute_tool()` currently catches an exception, creates a failure string, and calls `speak_error()` before returning the `FunctionResponse`.

`sp​eak_error()` then uses `speak()`, which injects a client-content message into the live model session.

This creates a fake conversational turn while the tool-response pipeline is still in progress.

## 9.2 Required behavior

Tool exceptions must remain inside the actual tool response:

```text
[TOOL_FAILED] Tool '<name>' failed: <reason>
```

Then the model may:

```text
retry with a changed strategy
fallback
ask one clarification
report failure
```

Blindly issuing the exact same failed tool call is not recovery.

## 9.3 Async error speech

`speak_error()` may remain for genuinely asynchronous, unrecoverable conditions, but it must not inject a second conversational turn into an active function-response cycle.

---

# 10. PHASE 1D — STATE VERIFICATION

This is the bridge from a collection of tools to a trustworthy agent.

## 10.1 Shared evidence vocabulary

Use explicit outcome fields where practical:

```text
requested
attempted
started
completed_by_tool
verified
failed
cancelled
blocked
```

## 10.2 Application verification

The existing `actions/open_app.py` contains launch fallbacks, but its successful launcher paths return `True` primarily because the launcher mechanism itself did not throw.

That is insufficient evidence for strong language such as:

```text
"Opened Chrome."
```

Required semantics:

```text
launch requested
launch mechanism invoked
process observed
window observed where possible
requested state verified
```

Do not conflate:

```text
process already existed
```

with:

```text
new launch happened
```

Distinguish at minimum:

```text
ENSURE AVAILABLE
FOCUS EXISTING
OPEN NEW WINDOW
OPEN NEW TAB
RESTART
CLOSE
```

## 10.3 File verification

The current `actions/file_controller.py` already has strong safety and undo behavior, but mutation functions generally return success immediately after the operation.

Required postconditions:

```text
CREATE
→ destination exists + expected content/state

MOVE
→ source absent + destination present

COPY
→ destination exists

RENAME
→ old path absent + new path present

DELETE
→ original path absent / trash state observed
```

Undo registration must remain intact.

## 10.4 Settings verification

For settings operations where the OS permits re-reading the value:

```text
set value
   ↓
read value
   ↓
compare
   ↓
verified / mismatch / unavailable
```

## 10.5 Browser verification

Later extensions must distinguish:

```text
browser exists
browser window exists
correct window focused
requested tab exists
requested URL loaded
requested page state observed
```

This addresses upstream patterns represented by #31, #32 and #108.

---

# 11. PHASE 1E — TOOL-RESULT DELIVERY WATCHDOG

## 11.1 Purpose

A tool may complete successfully while the Live model fails to produce the expected follow-up turn.

The system needs a delivery watchdog, not a blind retry engine.

## 11.2 Required flow

```text
TOOL RESPONSE SENT
       ↓
WATCHDOG ARMED
       ↓
MODEL OUTPUT ARRIVES
       ↓
WATCHDOG CANCELLED
```

or:

```text
TOOL RESPONSE SENT
       ↓
NO VALID MODEL OUTPUT WITHIN TIMEOUT
       ↓
ONE [TOOL_RESULT_READY] NUDGE
       ↓
WAIT
       ↓
NO SECOND NUDGE
```

## 11.3 Required properties

- one watchdog per invocation/turn boundary,
- bounded timeout,
- one-shot nudge maximum,
- no recursive re-arming,
- no duplicate final response,
- cancellation on legitimate model output,
- visibility in diagnostics.

The current `main.py` has turn events and audio timeouts, but it does not yet contain this dedicated tool-result delivery watchdog.

---

# 12. PHASE 1F — LIVE VALIDATION GATE

The phase is not complete when the code imports. It is complete only after real machine behavior passes.

## 12.1 Sleep / wake test matrix

```text
1. WAKE → sleep manually
2. repeat wake/sleep 10 times
3. sleep while wake phrase is spoken
4. sleep immediately after a wake phrase
5. wake after remaining asleep for >10 s
6. enable/disable wake mode
7. system sleep → system resume
8. system sleep → resume → microphone test
9. repeated wake phrases during cooldown
```

Pass condition:

```text
no stale wake event can reverse a manual sleep
no duplicate wake transition
resume does not leave JARVIS permanently stuck
```

## 12.2 Microphone test matrix

```text
1. quiet normal speech
2. quiet speech after JARVIS finishes speaking
3. immediate reply after JARVIS speech
4. repeated short commands
5. 60-second conversation
6. selected microphone
7. fallback microphone
8. PTT off
9. wake word enabled
10. wake word disabled
```

For each test capture:

```text
mic frames received
frames sent
queue behavior
Gemini input transcription
response latency
any gate that discarded frames
```

Pass condition:

```text
no unexplained user-speech loss
```

## 12.3 Tool reliability matrix

```text
forced action exception
slow action
successful action
application launch
file create
file move
file copy
file rename
file delete
```

Pass condition:

```text
truthful result
appropriate verification
no fake user turn
no duplicate spoken answer
```

## 12.4 Regression gate

Also verify that these existing capabilities still work:

- normal conversation,
- session resumption,
- vision flow,
- confirmation UI,
- undo,
- selected audio devices,
- push-to-talk,
- background monitors without starving voice input.

---

# 13. VERSIONING AFTER V3

The release strategy remains simple.

```text
v0.1.0  Mark LIV Foundation

v0.2.0-alpha.1  Phase 1A — sleep/wake
v0.2.0-alpha.2  Phase 1B — voice responsiveness
v0.2.0-alpha.3  Phase 1C — truthful failures
v0.2.0-beta.1   Phase 1D — verification
v0.2.0-beta.2   Phase 1E — delivery watchdog

v0.2.0          Mark LIV Reliability Core
```

Intermediate tags are optional. The full release `v0.2.0` is created only after the live validation gate passes.

---

# 14. WHAT IS EXPLICITLY NOT PART OF v0.2.0

Do not add these merely because they appear elsewhere in the blueprint:

- lesson manager,
- preference miner,
- routine inference,
- full autonomous task engine,
- self-improvement pipeline,
- major prompt/personality rewrite,
- new HUD architecture,
- headless architecture,
- local-model framework,
- multi-model abstraction,
- large browser redesign,
- massive refactor,
- autonomous source-code editing.

Those are higher-level phases.

The goal of `v0.2.0` is simple:

> **JARVIS must become dependable at hearing, sleeping/waking, executing, verifying and truthfully reporting.**

---

# 15. PHASE 2 — CONVERSATIONAL CORE

Only after Phase 1 is stable.

Scope:

- rewrite `core/prompt.txt`,
- evidence-based completion language,
- concise butler-style voice behavior,
- adaptive recovery instructions,
- stronger tool discipline,
- explicit distinction between tool result and proof.

The prompt should describe behavior.

The code must continue enforcing:

```text
permissions
state
verification
timeouts
confirmation
cancellation
```

Self-learning remains disabled during this phase.

---

# 16. PHASE 3 — MEMORY / LESSONS / METRICS

Separate these concepts:

```text
MEMORY   = what JARVIS knows about the user/context
LESSONS  = reusable behavior learned from evidence
METRICS  = what operationally happened
```

New targets:

```text
memory/lessons_manager.py
memory/lessons.json
core/ops_metrics.py
```

Explicit user instruction outranks inference.

One-off behavior must not become permanent preference.

---

# 17. PHASE 4 — INFERRED LEARNING

Add only after explicit learning works.

Scope:

- preference mining,
- communication learning,
- operational review,
- conservative lesson drafting.

Every inferred rule should have:

```text
evidence
confidence
scope
created_at
last_confirmed
```

---

# 18. PHASE 5 — TASK ENGINE

New target:

`core/tasks.py`

Complex objectives gain explicit state:

```text
QUEUED
RUNNING
WAITING
VERIFYING
COMPLETED
FAILED
BLOCKED
CANCELLED
```

Task state contains:

```text
objective
steps
dependencies
priority
completed_steps
remaining_steps
active_step
last_tool
last_result
verification_state
failure_state
recovery_state
cancellation_state
```

Long-running ownership:

```text
accept
→ begin
→ work quietly
→ surface meaningful blockers/progress
→ verify final state
→ proactively report completion
```

---

# 19. PHASE 6 — COMPUTER INTELLIGENCE

Treat the computer as a stateful environment.
Important entities:

```text
APP
WINDOW
TAB
FILE
FOLDER
WEBSITE
PERSON
PROJECT
DEVICE
SETTING
```

Each stateful operation should define:

```text
requested_state
attempt_method
evidence
verification_rule
failure_reason
```

This is where the broader browser/camera/app issue backlog is resolved systematically rather than as isolated patches.

---

# 20. PHASE 7 — SYSTEM INTELLIGENCE

Target:

`core/diagnostics.py`

Provide a unified health model:

```text
FULLY_OPERATIONAL
PARTIALLY_OPERATIONAL
TEMPORARILY_UNAVAILABLE
FAILED
```

Diagnostics should be able to answer:

```text
What is running?
What is broken?
What is waiting?
What is consuming resources?
What task is active?
What recently failed?
```

---

# 21. PHASE 8 — PROACTIVE JARVIS

Build on the existing proactive engine.

Proactive behavior must be:

- useful,
- context-aware,
- time-appropriate,
- interrupt-safe,
- suppressed by quiet/focus/privacy settings.

Long-running completion reports are event-driven task outputs, not generic proactive chatter.

---

# 22. PHASE 9 — SECURITY / IDENTITY

Move the upstream security concerns into a dedicated phase.

Relevant upstream security issue:

### #2 — Tool prompt injection / API-key storage

The issue raises concerns about unrestricted high-impact tools being driven by model output, as well as plaintext API-key storage.

Reference:
`https://github.com/FatihMakes/Mark-LIV/issues/2`

This phase may introduce:

```text
authorization tiers
sensitive-action policies
credential-manager integration
privacy mode
audit trail
```

The existing `core/confirm.py` remains the base for human confirmation of genuinely irreversible actions.

---

# 23. PHASE 10 — PRESENCE / HUD / DASHBOARD

The existing PyQt6/QPainter UI remains the presentation layer.

Target direction:

```text
charcoal / black background
orange / amber visual language
central JARVIS core/avatar
left navigation
right activity/task panel
bottom command bar
system metrics
notifications
active-task state
```

The UI consumes structured truth from the core.

It must never decide:

```text
whether an app opened
whether a task completed
whether a routine should exist
whether a confirmation is valid
```

---

# 24. PHASE 11 — ROUTINES / ADVANCED AUTONOMY

Recurring routines may be created only through:

```text
explicit user command
OR
explicit user acceptance of a proposal
```

Autonomous workflows require:

```text
max steps
max duration
max retries
resource limits
cancellation
verification
audit trail
```

---

# 25. PHASE 12 — CONTROLLED SELF-IMPROVEMENT

Self-improvement is deliberately last.

Required pipeline:

```text
OBSERVE
  ↓
DIAGNOSE
  ↓
PROPOSE
  ↓
ISOLATE
  ↓
TEST
  ↓
BENCHMARK
  ↓
VERIFY
  ↓
VERSION
  ↓
APPROVE
  ↓
DEPLOY
  ↓
MONITOR
  ↓
ROLLBACK
```

The live system must never be allowed to rewrite its own:

- authorization logic,
- security boundaries,
- emergency shutdown,
- rollback mechanism,
- recovery invariants.

---

# 26. CONFIGURATION CONTRACT

New optional subsystems must extend the existing `memory/config_manager.py` mechanism.

Relevant reliability settings should include at minimum:

| Setting | Default | Purpose |
|---|---:|---|
| `watchdog_enabled` | `true` | Tool-result watchdog master switch |
| `watchdog_timeout` | `6` | Result-delivery timeout in seconds |
| `wake_word_enabled` | existing config | Enable local wake word |
| `wake_word_sensitivity` | `medium` | Detection sensitivity profile |
| `wake_cooldown` | safe bounded value | Prevent duplicate/re-entrant wake transitions |
| `voice_barge_enabled` | `false` | Experimental voice interruption |
| `verification_enabled` | `true` | Verification master switch |
| `diagnostics` | `true` | Reliability diagnostics |
| `privacy_mode` | `false` | Restrict persistence/proactive behavior |

Every added setting must have a safe default.

---

# 27. TESTING CONTRACT

## 27.1 Unit tests

Use for:

- state transitions,
- wake re-arm logic,
- cooldown logic,
- verification predicates,
- result classification,
- watchdog state transitions.

## 27.2 Integration tests

Use for:

- tool registration,
- execution-state propagation,
- UI callback wiring,
- confirmation/undo interaction,
- reconnect interaction,
- task/tool interaction later.

## 27.3 Live tests

Mandatory for:

- microphone,
- speakers,
- wake word,
- sleep/resume,
- app launching,
- browser navigation,
- file state,
- window focus,
- camera/screen interaction,
- notification timing.

---

# 28. EVIDENCE LEVELS

```text
LEVEL 0 — model says it happened
LEVEL 1 — tool returned success
LEVEL 2 — process/resource exists
LEVEL 3 — requested state observed
LEVEL 4 — requested state persists after a stability check
```

The assistant must only use strong completion language when the evidence is strong enough for the operation.

Examples:

```text
LEVEL 1:
"The launch command was accepted."

LEVEL 2:
"Chrome is running."

LEVEL 3:
"Chrome's window is open and focused."

LEVEL 4:
"Chrome remains open and focused after the stability check."
```

This vocabulary should eventually be reflected in task state and audit records.

---

# 29. CODING-AGENT OPERATING CONTRACT

Every implementation task must contain:

```text
OBJECTIVE
SCOPE
FILES ALLOWED TO CHANGE
FILES FORBIDDEN TO CHANGE
EXISTING MECHANISMS TO REUSE
TESTS
LIVE VALIDATION
ROLLBACK POINT
```

Before editing, the coding agent must inspect the exact current implementation.

During editing it must not:

- rewrite unrelated modules,
- create a duplicate mechanism,
- move architectural state into UI widgets,
- silently broaden scope.

After editing it must:

- run syntax/import checks,
- run focused tests,
- perform relevant live validation,
- report what changed,
- report what remains unverified.

Failure protocol:

```text
STOP
↓
IDENTIFY SHARED BOUNDARY
↓
REPRODUCE
↓
LOCATE ROOT CAUSE
↓
PATCH SMALLEST LAYER
↓
RETEST
```

---

# 30. FIRST FIVE CODING TASKS AFTER V3

These are the first tasks to hand to the coding agent. Do not collapse them into one giant prompt.

## Task 1 — Sleep/Wake Race Reproduction + Instrumentation — COMPLETE

Inspect and instrument:

```text
main.py
core/wake_word.py
ui.py wake/sleep callbacks
```

Do not redesign yet.

Goal:

```text
prove exactly how a stale wake event can reverse manual sleep
OR
prove that the wake detector is not the cause
```

Acceptance test:

```text
10 repeated manual sleep/wake cycles
+ stale-event reproduction attempt
+ event ordering logged
```

## Task 2 — Sleep/Wake State Fix — COMPLETE

Only after Task 1 identifies the boundary.

Goal:

```text
manual sleep cannot be reversed by stale wake detections
no duplicate wake from one phrase
```

Acceptance test:

```text
10 repeated toggles
system sleep/resume
wake-word enabled
no immediate wake after manual sleep
```

## Task 3 — Microphone Path Instrumentation — NEXT (PHASE 1B)

Inspect and instrument:

```text
main.py:_listen_audio
main.py:_send_realtime
core/echo.py
core/audio_devices.py
```

Goal:

```text
identify exactly where user speech is discarded or delayed
```

No threshold redesign before the discard path is known.

## Task 4 — Microphone Reliability Fix

Patch the smallest proven boundary.

Acceptance:

```text
user speech reaches Gemini reliably
no permanent speaking-state lockout
no unexplained queue starvation
```

## Task 5 — Truthful Tool Failure Channel

Inspect and patch:

```text
main.py::_execute_tool
main.py::speak_error
```

Acceptance:

```text
forced tool exception
→ [TOOL_FAILED] FunctionResponse
→ no fake user turn
→ no duplicate spoken response
```

Only after these five tasks should application/file verification and the result-delivery watchdog be implemented.

---

# 31. LONG-TERM FILE MAP

| File | Responsibility | Strategy |
|---|---|---|
| `main.py` | Live session/orchestration | extend carefully |
| `ui.py` | presentation/HUD | presentation only |
| `core/wake_word.py` | wake detection | harden |
| `core/echo.py` | echo discrimination | measure + tune |
| `core/audio_devices.py` | device discovery/resolution | preserve |
| `core/hotkey.py` | PTT | preserve |
| `core/confirm.py` | human authorization | preserve/extend |
| `core/undo.py` | reversible actions | preserve/extend |
| `core/action_loader.py` | action discovery | preserve |
| `core/plugin_loader.py` | plugin discovery | preserve |
| `core/tasks.py` | task state | future/new |
| `core/diagnostics.py` | system diagnostics | future/new |
| `core/ops_metrics.py` | operational metrics | future/new |
| `memory/memory_manager.py` | persistent user memory | preserve |
| `memory/lessons_manager.py` | lessons | future/new |
| `memory/routines_manager.py` | recurring routines | future/new |
| `actions/open_app.py` | application operations | add verification |
| `actions/file_controller.py` | file operations | add verification |
| `actions/browser_control.py` | browser operations | add state verification later |
| `actions/screen_processor.py` | screen/camera | preserve, improve later |
| `dashboard/*` | remote control | after local reliability |
| `core/prompt.txt` | communication/behavior | Phase 2 |

---

# 32. DEFINITION OF A RELIABLE MARK LIV → MARK 54

A reliable Mark 54 system behaves like this:

```text
I speak.
   ↓
JARVIS reliably receives me.
   ↓
JARVIS understands the objective.
   ↓
JARVIS acts.
   ↓
JARVIS knows which state the operation is in.
   ↓
JARVIS verifies what actually happened.
   ↓
JARVIS tells me the truth.
   ↓
If something fails, JARVIS recovers intelligently or reports the failure.
   ↓
If the task takes time, JARVIS owns it without blocking the conversation.
   ↓
Only then does JARVIS learn, personalize and act proactively.
```

The central development standard is:

> **Fix the machine we actually have before building the machine we eventually want.**

And the first release objective is:

> **Make JARVIS dependable at hearing, sleeping/waking, executing, verifying and reporting before adding more autonomy.**