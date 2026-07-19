# Tone Orb — Meditation Mini Web Game (v1 SPEC)

**Owner:** Ham
**Date:** 2026-07-19
**Status:** v1 scope locked
**Purpose:** Personal-use meditation (สมาธิ) tool. Not a product. Zero incremental cost.

---

## Concept

User sustains a hum/tone into the microphone. An abstract orb on screen smooths
out and warms in color as their pitch and volume stay steady. Steadiness of
voice = steadiness of mind. No scores, no failure states, no timers.

---

## Locked Decisions (with rationale)

| # | Decision | Choice | Rationale |
|---|----------|--------|-----------|
| D1 | Core mechanic | Voice (sustained tone), not motion | (fact) Web Audio works everywhere; iOS DeviceMotion is permission-flaky. (opinion) Humming is itself a real breath-control practice — mechanic *is* the meditation. Motion parked for v1.1+. |
| D2 | Scoring model | Self-consistency — no reference tone | (opinion) "Correct vs incorrect" framing creates performance anxiety, counter to meditation. Also removes need for reference audio assets. |
| D3 | Visual feedback | Abstract orb | (opinion) Calming, non-clinical, intuitive (steady = smooth). A waveform/tuner view would induce analytical mindset. |
| D4 | Session structure | Open-ended, manual start/stop | (opinion) Timers reintroduce clock-watching; streak/high-score framing adds failure anxiety. |
| D5 | Stack | Single-file HTML/JS/CSS, vanilla, no build step | (fact) Zero hosting cost as static file (GitHub Pages / Vercel free). (opinion) Framework adds complexity with no payoff at this scope. |
| D6 | Sharing | Public static URL | (fact) Anyone with link can use it; mic permission is per-device; all data local. No backend needed. |
| D7 | History | None — ephemeral sessions | Ship core loop first; validate calming value before building tracking. localStorage history is a clean v1.1 add. |
| D8 | UI language | English | Keeps copy consistent with owner's other project docs; ~5 UI strings total. |
| D9 | Calibration | 2–3s "take a breath" prompt before tracking | (opinion) Grounding micro-ritual; prevents first unstable seconds (throat clearing) from feeling discouraging. |
| D10 | Stability metric | Pitch variance + volume variance | (fact) Volume drift (trailing off, cracking) is as perceptible as pitch wobble; same mic input, near-zero extra cost. Breath-continuity penalty rejected — risks punishing normal breathing. |
| D11 | Orb rendering | Edge jitter + color shift; NO size pulsing | (opinion) Jitter = most intuitive unstable/rough mapping. Size pulsing looks like a loading spinner or breathing pacer — ambiguous signal. |
| D12 | End summary | Duration only ("You held tone for 4m 12s") | (opinion) A stability % is a grade — re-introduces evaluation at the calm close. Duration is neutral practice info. |
| D13 | Mic denied | Basic fallback message | ~5 lines of code; prevents blank-screen confusion when shared with friends. (fact) iOS Safari needs HTTPS + user gesture for mic. |

---

## Core Loop

1. Landing screen → **Begin** button (user gesture required for mic on iOS)
2. Request mic permission
   - Denied/unavailable → show fallback message (D13), stop
3. Calibration: ~3s "Take a breath…" prompt (D9). No tracking yet.
4. Session: orb renders live
   - Rolling window (~2s) of pitch (autocorrelation) + volume (RMS)
   - Instability = normalized combination of pitch CV + volume CV
   - Orb edge jitter amplitude ∝ instability; color interpolates
     unstable (cool/desaturated) → stable (warm/clear) (D11)
   - Silence (below noise floor): orb rests in neutral dim state — not
     penalized, not rewarded (aligned with D10 rejection of breath penalty)
5. **End session** button → summary screen: duration only (D12) + **Again**

## Non-Goals (v1)

- Motion/stillness mechanic (v1.1+ candidate)
- Session history / stats / streaks
- Accounts, sync, backend of any kind
- PWA/installability
- Thai or bilingual UI
- Any stability score shown to the user
- Sound output (drones, chimes) — silent app, user's voice is the only audio

## Technical Notes

- Web Audio API: `getUserMedia` → `AnalyserNode`; time-domain data for both
  autocorrelation pitch estimate and RMS volume
- Rendering: Canvas 2D, requestAnimationFrame; orb = circle with per-vertex
  radial noise displacement, smoothed over time so transitions feel organic,
  never twitchy — visual smoothing must lag the metric slightly (calm > accurate)
- Pitch range of interest: ~60–500 Hz (human hum). Ignore estimates outside it.
- Mobile-first layout; must work in iOS Safari and Android Chrome
- HTTPS required for mic (any static host provides this; `file://` won't work
  on iOS — test via localhost or deployed URL)
- prefers-reduced-motion: reduce/disable ambient orb drift, keep state colors

## Success Criteria (verifiable)

1. Open deployed URL on iPhone Safari + Android Chrome → mic prompt appears
   after tapping Begin; denial shows fallback text
2. Humming a steady tone for 10s → orb visibly smooth + warm色
3. Deliberately wavering pitch or volume → orb visibly jittery + cooler within ~1s
4. Going silent → orb rests neutral (no punishment visual)
5. End session → duration shown matches wall clock ±2s
6. Total: one `index.html`, no build step, no network calls, no console errors

## Deployment (zero cost)

GitHub Pages: push `index.html` to a public repo → enable Pages. Share URL.
(Alternative: Vercel free tier drag-and-drop.)

## Deferred / Revisit Gates

| Item | Gate |
|------|------|
| Motion mechanic | After voice v1 used regularly for 2+ weeks and still wanted |
| localStorage history | After core loop validated as genuinely calming |
| PWA install | Only if friction of opening URL becomes real annoyance |
