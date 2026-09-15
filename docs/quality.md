# Quality monitoring

When no tone, WAV, alignment, or system-wide session is active, the verdict is
**Pipeline quality: idle / not measuring**. Stored metrics from a completed session never
produce an active warning while the app is idle.

TwoMore translates engine observations into useful warnings without claiming
laboratory precision.

## Per-device metrics

- **Underruns:** render-event stalls or starvation. One is amber; repeated
  underruns are red.
- **Buffer fill:** how much queued audio remains. A low thin bar means less
  safety margin.
- **Indicative drift:** deviation between expected and observed renderer
  progression. It explains possible long-session desynchronization but is not a
  calibrated acoustic measurement.
- **Ring overruns/drops:** capture reached a full consumer ring. The producer
  drops that consumer's oldest block rather than blocking capture.
- **Torn-read guards:** a consumer saw a block change while copying. Output is
  zero-filled and the counter increments. This is a data-race/corruption-class
  fault and must remain zero.
- **Profile/codec:** Hands-Free is always a degraded call-mode warning. Codec is
  `unknown` unless Windows provides reliable evidence.

Capture health separately reports sample rate, channels, captured buffers,
discontinuities, ring underruns/overruns, and torn-read guards. All counters are
atomic; the UI reads them on its timer rather than receiving work from an audio
thread.

## Global verdict

- **Good:** clean renderer/capture counters, healthy fill, stable indication.
- **At risk:** occasional underrun, low fill, discontinuity, or moderate drift.
- **Degraded:** frequent underruns, Hands-Free, capture error, or large drift.

The pipeline verdict is intentionally independent of setup. A 24% cable-input
level or mismatched Windows output levels produces a separate **Setup warning**
chip; it cannot turn clean pipeline counters amber. Set CABLE Input to 100%
before judging noise or loudness.

## White-noise defense

v0.5.2 proved that real system audio reached both Bluetooth devices, but its
shared mutable timeline could not prove that a reader never observed stale or
partially replaced sample storage. v0.6.0 replaces the active route with one
preallocated SPSC ring per renderer:

- capture copies a complete float32 stereo block and release-publishes it only
  after the copy;
- a renderer acquire-loads the published cursor and verifies the block sequence
  before and after its fixed-memory copy;
- empty rings produce exact zeros; full rings drop the oldest complete block
  (or the incoming block if the consumer already owns the oldest);
- all float samples and per-device gain results are clipped to `[-1, 1]`;
- Windows shared mode performs genuine endpoint sample-rate conversion, so
  TwoMore does not apply a second software resampler.

Stress tests cover silence, a pure sine, stalled consumers, wrap/overrun
recovery, and live gain. The synchronous sine residual RMS is exactly zero in
the test. This proves the software invariants, not that a particular Bluetooth
driver or radio path is noise-free; real browser/game retesting is still
required.

## Presets

| Preset | Requested buffer | Approx. added latency | Trade-off |
|---|---:|---:|---|
| Gaming | 50 ms | ~70 ms | Fastest; most vulnerable to Bluetooth/radio stalls |
| Video | 100 ms | ~120 ms | Default balance; use saved alignment for lip-sync |
| Music | 200 ms | ~230 ms | Most stable; least suitable for interaction |

Actual latency also includes Windows, codec, radio, and headset buffering.

## Stability test

Choose 1, 5, or 10 minutes. The report includes per-device underruns, soft
re-syncs, buffer fill, maximum indicative drift, and a stable/at-risk verdict.
Saving/exporting the report does not turn an indicative result into a hardware
certification.

The realtime path now uses phase-continuous, bounded rate matching driven by
endpoint-clock drift and fill feedback. It is designed to prevent audible
timeline jumps during long sessions; it does not claim perfect acoustic phase
alignment between independent Bluetooth radios.
