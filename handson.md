# Hands-on Test — Tone Orb

Live: https://chanooooot.github.io/Sathi/

Test on a real phone (Chrome/Android or Safari/iOS) — mic needs real HTTPS, not localhost.

1. Open link → tap **Begin** → mic permission prompt shows
2. Deny → fallback message shows
3. Allow → 3s "Take a breath…" → hum steady tone 10s → orb smooth + warm
4. Wobble pitch/volume → orb visibly jittery + cooler within ~1s
5. Go silent → orb rests dim, no punish visual
6. **End session** → duration matches wall clock ±2s
7. Screen shouldn't sleep mid-session (wake lock)
8. After End, mic indicator (browser icon) turns off — stream released

## Known tuning knobs (adjust if feel is off)

| Constant | Purpose | Current |
|----------|---------|---------|
| `RMS_FLOOR` | silence threshold | 0.006 |
| `WINDOW_MS` | stability window | 2000 |
| smoothing `0.045` | orb calmness | draw loop |

If orb reads dim/gray while actively humming → `RMS_FLOOR` still too high for your mic, lower further.
