# Game Harmonics

Finds harmonics in a signal. Nothing else.

Connects to an M5StickS3 running `pulsewave.py` (from the MStackSTICK-S3
project) over Bluetooth LE and runs two independent, comparable algorithms
on the live spectrum:

- **Harmonic series** — finds the single strongest fundamental frequency,
  then checks whether real energy exists at 2x, 3x, 4x, and 5x that
  frequency. Reports only the harmonics that are actually present; a
  fundamental with none confirmed shows just the fundamental.
- **Top 5 peaks** — the 5 most prominent frequency components in the
  signal, independent of whether they're related to each other or to
  anything else.

No cardiac/breath semantics, no confidence gating beyond noise-floor
significance, no terrain visualization. Verified against synthetic signals
with known harmonic content and against 30 independent trials of pure noise
(zero false positives) before shipping — see the commit history for the
reasoning.

## Run

```
python3 -m http.server 8126
```

Open `http://localhost:8126/harmonics.html` in desktop Chrome (Web
Bluetooth requires a secure context; `localhost` qualifies).

## Bluetooth wire format

Reuses the exact packet format the stick already streams — no firmware
changes needed:

```
<uint32 first_index><uint16 count><uint16 rate_hz><uint32 t_us>
then count x uint16 little-endian samples
```
