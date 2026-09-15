# GUI guide

## Framework choice

The GUI uses native Win32 for the window/event loop, Direct2D for shapes, and
DirectWrite for high-quality Unicode text. This keeps the project dependency-
free and directly integrated with
the C++ WASAPI core. DirectWrite provides Cyrillic shaping through installed
Windows fonts. Per-monitor-v2 DPI awareness is declared in the application
manifest.

The portable package has one obvious product entry point: `twomore-gui.exe`.
The optional diagnostic CLI is kept separately at
`tools\twomore-cli.exe` so it does not look like a second GUI application.

Qt 6 would provide more controls but adds LGPL dynamic-link deployment duties.
Dear ImGui is permissively licensed but would need vendored sources, font assets,
and a custom accessibility/navigation layer. WinUI 3 adds Windows App SDK
deployment complexity. Native Direct2D is the smallest reproducible milestone.

## Layout and behavior

- **Test both headphones** is the primary action. It opens both endpoints,
  renders a configurable low-level warm-up, plays a repeating burst pattern,
  animates **Now playing** per renderer, and asks what was actually heard.
- The confirmation view has BOTH/ONE/NEITHER branches. SUCCESS requires both
  renderer results and the user's BOTH answer.
- **Align** provides device-first selection, 1/5 ms nudges, a draggable 0–500 ms
  control, timeline bars, replay/auto-replay, and stable pair preset save/clear.
- **Guided check** displays adapter, pairing, connection, Media endpoint,
  Hands-Free, selection, renderer, and audible-confirmation steps with live
  watcher refresh and one-click Settings shortcuts.
- **About / Diagnostics** displays app/Windows versions, detected adapters,
  verified configurations, virtual-cable detection, and diagnostics export.
- **Settings** exposes warm-up duration, low-level warm-up gain, repeats, and
  the System-wide setup view.
- **System-wide** detects a separately installed cable, validates the Windows
  default output and two real Media destinations, requires a one-time local
  capture acknowledgement, and routes live cable audio through the same
  multi-endpoint renderer used by tests and WAV playback.
- A persistent ACTIVE/STOPPED/ERROR banner and bottom-bar SILENCE WARNING make
  the default-cable-but-stopped state unambiguous. Start and explicit
  restore-previous-output actions are available from the warning.
- A live peak/RMS meter proves whether the chosen stable capture endpoint is
  receiving non-silent audio. **Verify capture** injects a short tone into the
  matching cable playback input and checks the complete capture-to-render path.
- When several cable variants are installed, the capture picker remembers the
  exact endpoint ID and prefers the pair whose input is the Windows default.
- Gaming, Video, and Music presets trade requested buffering for approximate
  added latency. Live source/renderer counters produce a conservative global
  quality status and per-device explanation.
- A 1/5/10-minute stability test records underruns, soft resynchronization,
  buffer fill, and indicative drift. It can expose poor conditions but cannot
  prove codec quality or exact acoustic synchronization.
- The readiness banner names Bluetooth-connected devices whose Media output is
  not active and provides the next action.
- The device area groups Media and Hands-Free sibling endpoints by device name.
- The default Active view includes active endpoints and Windows-connected
  Bluetooth devices even when their Media endpoint is not ready.
- Search and filter chips keep irrelevant historical endpoints out of view.
- Technical IDs are opt-in and expose raw endpoint/Bluetooth IDs plus the
  cross-layer mapping state.
- Device cards expose selection, volume, mute, and delay.
- The first inventory after startup restores the last selected Media devices by
  stable Bluetooth/container identity, preferring an active Media endpoint and
  never substituting a Hands-Free sibling.
- The activity panel shows session, refresh, warning, and endpoint-error events.
- The bottom transport starts tone, impulse, left/right, or WAV playback and can
  cancel an active session.

Audio and WAV decoding run away from the UI thread. Results return through
posted window messages. `IMMNotificationClient` callbacks are debounced before
they post a refresh; enumeration occurs on a background COM thread. The visible
log coalesces rapid duplicates and is capped, while the persistent debug log
retains raw events.

## Demo and smoke modes

`twomore.exe gui --demo` displays two Bluetooth-connected/media-not-ready
devices, a connected Hands-Free-only device, Cyrillic text with Unicode status
glyphs, USB, offline, disabled, and empty-name endpoints. It is the preferred
screenshot mode.

`twomore.exe gui --smoke --mock` creates the hidden application, initializes the
rendering surface and mock device model, drives warm-up before burst, renderer
activity, confirmation and all verdict branches, 120 ms stable-preset
save/reload, Guided check, About, Settings, a mock cable setup, live quality
branches, and a one-second mock stability run, then exits. Mock mode never opens
a real audio endpoint.

## Secondary CLI mirrors

| GUI action | CLI mirror |
|---|---|
| Test both headphones | `tone --bt 2 --type burst --warmup 500 --repeat 3 --loudness-match` |
| Alignment preset | `latency --bt 2 --delay index:ms --save`; `--clear` |
| Guided machine check/report | `check --guided --json report.json` |
| Device diagnostics | `list --json`; `check-bt --verbose`; `diagnose output.json` |
| Test tone / file | `tone ...`; `play file.wav ...` |
| System-wide setup | `systemwide --list-cables`; `systemwide --check` |
| System-wide session | `systemwide --preset video --acknowledge-local-capture` |
| Stability test | `systemwide --stability --minutes 5 --acknowledge-local-capture` |
| Stop system-wide CLI session | `systemwide --stop` |

The CLI cannot collect audible confirmation and says so explicitly.

`list --json` uses schema version 2 and includes each endpoint's negotiated
sample rate/channels/bits, Windows volume, transport, format, and honest
quality tier. It never guesses an exact Bluetooth codec.

## Current interaction boundary

Window position, size, and maximized state plus the last Media selection and
per-device volume, mute, and delay are persisted locally by stable device
identity. Volume, mute, and delay edits also apply live to an active session;
the saved values are used when the next session starts.
