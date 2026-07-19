# CLAUDE.md — Tone Orb

## What this project is

A single-file meditation (สมาธิ) web game for personal use. The user hums one
steady note into the microphone; an abstract orb on screen smooths out and
warms in color as their pitch and volume stay stable. Steady voice = steady
mind. There are no scores, no timers, no failure states — this is a calm tool,
not a game to win.

**Owner intent:** personal use + sharing a URL with friends. Not a product.
Zero incremental cost, forever.

## Repo layout

```
index.html   # the entire app — HTML + CSS + JS in one file
SPEC.md      # locked design decisions D1–D13 with rationale + success criteria
AGENTS.md    # agent guardrails, tuning knobs, deferred-feature gates
CLAUDE.md    # this file
```

Read `SPEC.md` before making any design-level change. Read `AGENTS.md` before
making any code change.

## How the app works (core loop)

1. Landing screen → **Begin** (user gesture required — iOS mic rule)
2. `getUserMedia` mic request; denied → fallback message, stop
3. ~3 s "Take a breath…" calibration screen (no tracking yet)
4. Session (open-ended):
   - `AnalyserNode` time-domain data → autocorrelation pitch (60–500 Hz)
     + RMS volume
   - Stability = coefficient of variation of pitch and RMS over a 2 s
     rolling window: `instability = min(1, pitchCV*14 + rmsCV*1.6)`
   - Orb: Canvas 2D circle, edge jitter ∝ instability; color lerps
     cool slate (unstable) → warm ember (stable)
   - Silence (RMS < 0.012) → orb rests dim/neutral. Never penalized.
5. **End session** → summary shows duration only → **Again**

## Non-negotiable constraints

1. **Single file** — everything stays in `index.html`. No framework, no build
   step, no npm, ever.
2. **Zero network** — no CDNs, fonts, analytics, APIs, backend. Works offline.
3. **No evaluation shown** — never display stability %, grades, streaks, or
   pass/fail. Duration is the only number the user sees.
4. **Open-ended sessions** — no timers, no auto-end, no failure states.
5. **Silence is neutral** — going quiet to breathe is never punished visually.
6. **Silent app** — no sound output; the user's voice is the only audio.
7. **Mobile-first** — must work on iOS Safari + Android Chrome over HTTPS.

If a request conflicts with these, stop and flag it — don't silently comply.

## Engineering style (follow strictly)

- State assumptions before coding; ask when uncertain
- Minimum code that solves the problem — no speculative abstractions or config
- Surgical edits only — every changed line traces to the request
- Restate the SPEC success criterion you're targeting, then verify against it
- The orb must stay visually calm: displayed instability is lerp-smoothed
  (`k=0.045`) and must lag the raw metric. Never make it twitchy.

## Tuning knobs (safe to adjust on user feedback)

| Constant | Purpose | Current |
|----------|---------|---------|
| `WINDOW_MS` | stability window | 2000 |
| `RMS_FLOOR` | silence threshold | 0.012 |
| `pCV*14 + vCV*1.6` | instability weights | in `measure()` |
| smoothing `0.045` | orb calmness | in `draw()` |

One knob at a time; verify by manual hum test (SPEC criteria 2–4).

## Deferred — do NOT build unless the owner explicitly re-opens the gate

- Motion/stillness mechanic (gate: voice v1 used regularly 2+ weeks)
- localStorage session history (gate: core loop validated as calming)
- PWA / installability
- Thai or bilingual UI
- Backend, accounts, sync — any of them

## Deployment

Static hosting only. GitHub Pages: push → Settings → Pages → main branch root.
Mic requires HTTPS — always verify changes on the deployed URL from a real
phone, not just desktop.
