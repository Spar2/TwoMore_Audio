# System-wide routing

## What the virtual cable does

Windows normally targets one default playback device. A virtual cable provides
a playback INPUT that applications can target and a capture OUTPUT that TwoMore
can read. TwoMore captures that output locally and fans it out to the selected
active outputs. Bluetooth remains the primary use case, but a pair may also mix
Bluetooth with wired, USB, or HDMI audio.

TwoMore does not bundle, download, install, update, license, or control
VB-Cable/Voicemeeter. Install one from its vendor:

- [VB-Cable official site](https://vb-audio.com/Cable/)
- [Voicemeeter official site](https://vb-audio.com/Voicemeeter/)

## GUI setup

1. Open **System-wide** and click **Re-detect**.
2. Confirm that the correct capture endpoint and matching playback input are
   selected. If several VB-Cable variants are present, use the capture picker.
   TwoMore remembers the exact capture endpoint ID and prefers the pair whose
   playback input is currently the Windows default.
3. Open Sound settings and manually select the cable INPUT as default playback.
4. Select two active, non-Hands-Free output cards. A Bluetooth+wired/USB/HDMI
   pair is supported; every card shows its transport badge.
5. Acknowledge the one-time notice that audio is processed locally and never
   recorded, stored, or transmitted.
6. Click **Check my setup**. The cable, default-output, destination, and loop
   checks must pass. If cable INPUT volume is below 90%, set it to 100% in
   Windows Sound settings.
7. Choose Gaming, Video, or Music and click **Start**.

TwoMore never switches the default to the cable automatically. It remembers a
non-cable default output while it can observe one. The explicit **Restore
previous output** escape hatch restores that remembered endpoint; if no previous
endpoint was observed, it opens Sound settings instead of guessing.

## Active state, silence guard, and level meter

System-wide mode always displays one of three persistent states:

- **SYSTEM-WIDE ACTIVE** names the opened capture endpoint and destination count;
- **SYSTEM-WIDE STOPPED** means no cable audio is being consumed and names the
  recorded stop reason;
- **SYSTEM-WIDE ERROR** includes the capture or renderer failure.

If the cable INPUT is the Windows default while routing is stopped, a loud
**SILENCE WARNING** explains that browser/game/video audio is going into an
unconsumed cable. Use **Start routing now** or **Restore previous output**. The
bottom status also says SILENCE WARNING rather than Idle.

While active, the capture meter displays peak and RMS. Capture open, exact
endpoint ID/name, negotiated or attempted formats, first non-silent buffer,
HRESULT failure, and close are written to Activity and the exported test report.
The diagnostic distinguishes:

- zero capture level: wrong/inactive endpoint, wrong default, low cable volume,
  an app-specific output, exclusive mode, or protected content;
- capture level present but no renderer activity: downstream routing failure;
- capture and renderer activity with low level: cable/device volume or gain.

## Verify capture

**Verify capture** briefly renders a built-in tone directly to the selected
cable INPUT while the matching cable OUTPUT is being captured and routed to both
headphones. A pass proves the selected cable pair and capture-to-render path
worked for the test signal without YouTube. It does not bypass protected paths
or prove that another application is targeting the cable.

CLI mirror:

```powershell
twomore.exe systemwide --verify-capture --acknowledge-local-capture
```

The command uses the build-tree executable name. In the portable package, run
the optional mirror as `tools\twomore-cli.exe`; the normal product entry point
remains `twomore-gui.exe`.

## Loop and double-audio rules

- A virtual-cable capture or playback endpoint can never be selected as a
  renderer. TwoMore blocks the configuration.
- The matching cable INPUT should remain the Windows default while routing.
- The Bluetooth targets should receive audio only from TwoMore. Leaving one as
  the Windows default can cause direct plus routed/double audio.
- A suspected selected-endpoint loss starts a 1.5 second control-plane debounce.
  TwoMore re-queries only that stable endpoint ID and stops only if it is still
  unavailable. Harmless notification storms are coalesced.

During an active session, property-only MMDevice notifications are ignored and
unrelated topology/default changes are deferred. The data plane uses one
preallocated SPSC ring per renderer. Capture release-publishes complete float32
stereo blocks; renderers acquire-read and sequence-check them. Empty rings are
zero-filled, full rings drop the oldest complete block, and neither side locks
or allocates while running. Per-device gain/mute are atomically published and
take effect without restarting the route.

## Mixed Bluetooth and wired outputs

Clear the Bluetooth-only filter if necessary, then manually select one
Bluetooth card and one Wired/USB/HDMI card. **Auto-select 2 BT** remains a
Bluetooth convenience, not a restriction. Open **Align** because a wired path
may be roughly 10 ms while Bluetooth is commonly much higher; save the result
for the exact stable pair. The same stable-ID preset mechanism restores mixed
pairs after reconnecting. The Gaming/Video/Music value is TwoMore's requested
buffer contribution, not total acoustic latency; alignment remains essential.

## Games, streaming, and protected audio

Shared-mode applications normally follow the default cable. Some games select a
specific endpoint or use exclusive mode. Use **App volume overrides** to target
the cable for a silent application, or disable exclusive mode where appropriate.

DRM-protected streams may refuse capture. TwoMore reports silence/capture health
and does not bypass protection.

## Troubleshooting

- **No cable detected:** confirm the vendor driver is installed, then re-detect.
- **Cable not default:** open Sound settings and select the cable INPUT.
- **Loop/double audio:** remove cable endpoints from render selection and keep
  the cable INPUT—not a Bluetooth headset—as the Windows default.
- **One app silent:** set that app's output to the cable; check exclusive mode.
- **Underruns:** choose Music, reduce radio interference, or test one headset.
- **Drift:** re-run Align; a static delay does not correct independent clocks.
- **Hands-Free quality:** close microphone users or disable Hands-Free Telephony.
