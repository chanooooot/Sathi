# AGENTS.md — Tone Orb

Instructions for any AI agent (Codex, Claude Code, etc.) working on this repo.
Read `SPEC.md` first. It is the source of truth for all design decisions (D1–D13).

## What this is

A single-file meditation web game. User hums a steady tone; an orb smooths and
warms as pitch + volume stay stable. Personal-use tool. Not a product.

## Repo layout

```
index.html   # the entire app — HTML + CSS + JS in one file
SPEC.md      # locked decisions + rationale + success criteria
AGENTS.md    # this file
```

## Hard constraints — do not violate

1. **Single file.** All HTML/CSS/JS stays in `index.html`. No build step, no
   bundler, no framework, no npm. If asked to add a framework, refuse and cite
   SPEC D5.
2. **Zero cost / zero network.** No external requests: no CDNs, no fonts, no
   analytics, no APIs, no backend. App must work fully offline once loaded.
3. **No scoring shown to the user.** Never display a stability %, grade, streak,
   or pass/fail. Duration is the only number the user ever sees (SPEC D12).
4. **No timers, no failure states.** Sessions are open-ended and end only by
   user action (SPEC D4).
5. **Silence is neutral.** Going quiet must never be penalized visually —
   orb rests dim/neutral (SPEC D10 rationale).
6. **No sound output.** The app is silent; the user's voice is the only audio.
7. **Mobile-first, HTTPS.** Must work in iOS Safari + Android Chrome. Mic
   requires HTTPS and a user gesture — keep mic request inside the Begin
   button handler.

## Engineering guidelines (Karpathy rules — follow strictly)

- **Think before coding.** State assumptions. If a request conflicts with
  SPEC.md, stop and flag the conflict instead of silently deciding.
- **Simplicity first.** Minimum code that solves the problem. No speculative
  abstractions, config options, or "flexibility" nobody asked for.
- **Surgical changes.** Touch only what the task requires. Don't reformat,
  rename, or "improve" adjacent code. Every changed line must trace to the
  request.
- **Goal-driven.** Before changing behavior, restate the verifiable success
  criterion you're targeting (see SPEC "Success Criteria"), then verify it.

## Key implementation facts

- Pitch: autocorrelation on `AnalyserNode` Float32 time-domain data
  (fftSize 2048), valid range 60–500 Hz, periodicity ratio ≥ 0.5 to accept.
- Volume: RMS of the same buffer. RMS < 0.012 → resting state.
- Stability: coefficient of variation of pitch and RMS over a 2 s rolling
  window; `instability = min(1, pitchCV*14 + rmsCV*1.6)`.
- Rendering: Canvas 2D, 140-vertex circle displaced by layered-sine noise;
  displayed instability is lerp-smoothed (`k=0.045`) so visuals stay calm —
  keep visual smoothing lagging the metric; never make the orb twitchy.
- `prefers-reduced-motion` disables ambient drift; state colors remain.

## Tuning knobs (safe to adjust if user reports feel is off)

| Constant | Purpose | Current |
|----------|---------|---------|
| `WINDOW_MS` | stability window | 2000 |
| `RMS_FLOOR` | silence threshold | 0.012 |
| `pCV*14 + vCV*1.6` | instability weights | see measure() |
| smoothing `0.045` | orb calmness | draw loop |

Change one knob at a time. Verify against SPEC success criteria 2–4 by
manual hum test before and after.

## Deferred — do NOT build unless the user explicitly re-opens the gate

- Motion/stillness mechanic (gate: voice v1 used 2+ weeks)
- localStorage session history (gate: core loop validated as calming)
- PWA / installability
- Thai or bilingual UI
- Any backend, accounts, or sync

## Deployment

Static host only. GitHub Pages: push repo → Settings → Pages → deploy from
main branch root. Verify mic prompt works on the deployed HTTPS URL from a
real phone before calling any change done.
