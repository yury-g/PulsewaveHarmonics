> **Personal independent research by Yury Gitman.** This repository is not official World Famous Electronics or PulseSensor documentation, does not define product behavior, and must not be used as current WFE source of truth. Current company and product information lives under [`WorldFamousElectronics`](https://github.com/WorldFamousElectronics).

# Pulsewave Harmonics

Finds harmonics in a signal. Nothing else.

Connects to a signal source either over Bluetooth LE (an M5StickS3 running
`pulsewave.py`, from the MStackSTICK-S3 project) or over USB Serial (Arduino
or any device that just prints numbers) and runs two independent, comparable
algorithms on the live spectrum:

- **Harmonic series** — finds the single strongest fundamental frequency,
  then checks whether real energy exists at 2x, 3x, 4x, and 5x that
  frequency. Reports only the harmonics that are actually present; a
  fundamental with none confirmed shows just the fundamental.
- **Top 5 peaks** — the 5 most prominent frequency components in the
  signal, independent of whether they're related to each other or to
  anything else.

A cross-reference table and matching color coding on the spectrum's dashed
marker lines show, at a glance, which of the top 5 peaks are actually part
of the harmonic series (teal, same color as the harmonic-series line) and
which are unrelated (violet).

A **guessing table** attaches a plain-language, frequency-band guess to each
top-5 peak — heart-rate fundamental, cardiac harmonic, respiratory
modulation/envelope, motion/pressure change, or sensor/electrical noise.
Every row says explicitly that it's a guess: a frequency-band heuristic, not
a diagnosis. Band edges (breathing ~0.1–0.6 Hz, pulse ~0.6–3.5 Hz) match the
ones already established for this sensor in the GameEngine sibling project.
The motion/pressure band is architecturally correct but, with the current
~8.2 s analysis window and 0.2 Hz search floor, will rarely populate —
genuinely slower content isn't reliably resolvable at this window length,
and reporting nothing is more honest than guessing at content the window
can't actually see.

Each guess also gets a 0–4 star confidence score, from two independent
signals: how far the peak clears the noise floor, and whether it's
corroborated by the harmonic-series match. A guess the code itself flags as
unconfirmed is capped at 2 stars no matter how strong the raw peak is —
amplitude alone never confirms *which* category something belongs to.
Nothing is hidden by a low score; every peak that clears the noise floor
still gets a row.

No terrain visualization. Verified against synthetic signals with known
harmonic content and against 30 independent trials of pure noise (zero
false positives) before shipping — see the commit history for the
reasoning, including a real peak-detection edge-case bug found and fixed
along the way (a genuine peak landing exactly on the search floor could be
skipped entirely).

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

## USB Serial wire format

Deliberately protocol-agnostic, for "any serial device" rather than one
specific firmware: **the first comma- or whitespace-separated field on each
line must be the raw numeric sample.** That's exactly what

```cpp
Serial.println(analogRead(A0));
```

already prints — no custom firmware needed. A CSV line like `2043,1990,2100`
also works (the first field is used). A *labeled* line where the value isn't
first — `preview,ms,raw,...`, the GameEngine sibling project's own XIAO
firmware format — is deliberately **rejected, not misread**: silently
reading the wrong field as the sample would be worse than dropping the line.

A generic sketch has no onboard clock to report the way the BLE stick's
ISR-timestamped packets do, so the sample rate is *estimated* from how often
lines actually arrive (a rolling median of inter-line intervals, resistant
to the occasional slow or dropped line). Pick the baud rate in the dropdown
to match the device's `Serial.begin(...)` call before connecting.

Only one transport can safely feed the shared analysis buffer at a time —
connecting one disables the other's button until you disconnect.
