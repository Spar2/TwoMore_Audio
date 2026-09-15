# Bluetooth troubleshooting

## First run is silent in one headphone

Bluetooth standby and A2DP stream activation can swallow an initial transient.
Different Windows endpoint volumes can also make one device seem silent. Use
the GUI's **Test both headphones** action instead of a single impulse: it sends
a low-level warm-up before three audible burst sequences. Watch for **Now
playing** on both cards, use **Match volumes** if the GUI reports a mismatch,
then answer the audible-confirmation question. A renderer returning `HRESULT=0`
proves Windows accepted buffers; it does not prove the sound was audible.

If only one plays, run the test again, raise the quieter headset safely, close
apps using headset microphones, and power-cycle the silent headset.

## Align and save latency from the GUI

1. Confirm that both headphones were audible.
2. Open **Align** and click **Play alignment test**.
3. Click the name of the headphone whose clicks arrive first.
4. Use the −5/−1/+1/+5 ms buttons or drag the delay control.
5. Replay until the clicks coincide, then click **Save preset for pair**.

The preset is keyed by sorted stable Bluetooth/container/endpoint identities,
not list indexes, and auto-applies when that exact pair is selected again.
Static delay does not correct long-term independent-clock drift.

## Device is offline or not present

The device may be powered off, connected to a phone, outside radio range, or
paired without an active Windows render endpoint. Open Bluetooth Settings,
disconnect it from other sources, reconnect it to the PC, and wait for the GUI
to refresh. Sound Settings must show it as an output before TwoMore can render.

## Connected in Bluetooth, but Media is not ready

This is a real split in Windows state: the Bluetooth association can be
connected while MMDevice reports the Media render endpoint as disabled,
unplugged, or not present. TwoMore keeps the device visible with a **BT
connected** badge and names affected devices in the top banner.

1. Use the card's **Sound** button and enable/select the headset's Media output.
2. Close voice/chat applications that may be holding its microphone.
3. Use **Services** to open the legacy Devices view and, if appropriate, disable
   **Hands-Free Telephony**. This removes headset call/microphone support.
4. Power-cycle/reconnect the headset, then click **Refresh**.
5. If only one Media endpoint activates, test updated Bluetooth drivers or a
   different adapter. Some hardware cannot sustain two A2DP sinks.

Run `twomore.exe check-bt --verbose` from the build tree to compare the WinRT
Bluetooth state with the MMDevice endpoint state and see how each layer was
mapped. In the portable package, use `tools\twomore-cli.exe` for this command.

## Only one active Bluetooth media device

Connect the second device, close apps using headset microphones, move both
devices closer, reduce 2.4 GHz and USB 3.x interference, and update the PC's
Bluetooth driver. Some Classic A2DP radios/drivers cannot sustain two sinks.
Another USB adapter is an experiment, not a guaranteed fix. On supported PCs,
check whether Windows LE Audio and Shared Audio are available.

## Hands-Free warning

Hands-Free is the call profile and normally has low-quality mono Classic audio.
Choose the Media endpoint. Close voice-chat applications that hold the headset
microphone. Disabling Hands-Free Telephony in Windows is an advanced system
change and may remove call functionality; TwoMore does not do it automatically.

## Endpoint opens but no sound is heard

Confirm device volume, battery, and connection; test the endpoint in Windows;
then try a low-volume sine test. Review the Activity panel and exported HRESULT.
An accepted WASAPI stream does not prove that the Bluetooth transport remained
active.

## Outputs sound out of sync

Add delay to the device heard earlier. Use the impulse test for sharper timing.
Manual delay corrects the initial offset only; independent clocks can drift over
long sessions.

## System-wide says no virtual cable is detected

Install a compatible cable from its publisher, restart TwoMore, and click
**Re-detect**. TwoMore only recognizes installed MMDevice endpoints by their
reported names; it does not download, install, license, or configure a driver.
If the cable is visible in Windows Sound settings but not TwoMore, export
diagnostics and inspect `virtual_cable_capture_endpoints`.

## Setup says the cable is not the default playback device

Open **Sound settings** and choose the cable's playback/input endpoint as the
Windows default output. The selected applications must also use Default rather
than a per-app output. TwoMore captures the cable's recording/output side and
routes it to the two selected real Bluetooth Media endpoints.

Do not select either side of the cable as a TwoMore destination. The setup check
blocks this feedback path, which would otherwise cause echo or runaway audio.

## A game, browser, or protected video remains silent

Check the Windows app-volume mixer for an app-specific output. Restart apps that
cached the previous default device. Exclusive-mode applications can bypass the
shared default path, and DRM/protected content may refuse or bypass capture.
TwoMore cannot override those application or content policies.

## System-wide quality is At risk or Degraded

Open **Why?** and inspect the per-device rows:

- source underruns or discontinuities indicate the cable capture was starved;
- renderer underruns or low buffer fill indicate output scheduling pressure;
- repeated soft resynchronization or growing indicative drift points to
  independent endpoint clocks;
- a Hands-Free endpoint means the low-quality call profile was selected.

Try the Video or Music preset, close CPU-heavy and microphone-using apps, keep
both headphones close to the adapter, and reduce USB 3.x/2.4 GHz interference.
Run the 5- or 10-minute stability test after changing one factor. Presets trade
latency for buffering; they do not guarantee codec, radio, or clock quality.

## Play WAV works, but YouTube or system audio is silent

This proves the direct headphone renderers work; it does not prove system-wide
capture was started. Check these causes in order:

1. Open **System-wide**. It must say **SYSTEM-WIDE ACTIVE**, not STOPPED, Error,
   or Idle. If the cable is default while routing is stopped, use the prominent
   **Start routing now** button. This is the default-cable-but-not-capturing
   silence trap.
2. Play audio and watch the capture peak/RMS meter. If it remains silent, use
   **Verify capture**. Activity must show the exact capture endpoint ID, its
   negotiated format, and a first non-silent buffer.
3. With multiple VB-Audio endpoints, select the standard
   `CABLE Output (VB-Audio Virtual Cable)` for the standard
   `CABLE Input`. Do not accidentally pair it with `CABLE In 16ch`.
4. In Windows Sound settings, set the CABLE Input level to 100%. A value such as
   31% attenuates everything before TwoMore captures it.
5. Check the application's output override and exclusive-mode setting. Protected
   content may bypass or reject capture.

To recover immediately, click **Restore previous output**. TwoMore restores the
last non-cable default it observed; if none was observed, it opens Sound settings
and does not guess. Then select speakers or one headset manually.

If the meter moves and both device rows show renderer activity but the result is
quiet, use **Match volumes**. It compensates for the reported Windows endpoint
levels with per-device in-app gain. In v0.6.0 gain and mute are atomic live
controls and do not restart active system-wide routing.

## System-wide audio repeatedly cuts out and devices keep refreshing

Build v0.5.1 treated every Windows endpoint property notification as a topology
change. Some Bluetooth and virtual-cable drivers emit hundreds of these events
within seconds while streams are active. That caused repeated full MMDevice and
WinRT Bluetooth enumeration, visible as repeated `Refreshing audio endpoints`
messages and sometimes an empty Windows audio picker.

v0.6.0 keeps the property-churn filtering and adds a 1.5 second stable-ID
recheck before a suspected selected-device loss may stop routing. A single
transient notification no longer authorizes a stop.

After upgrading, the Activity panel should not show continuous refresh lines
while **SW ACTIVE**. If sound still cuts out, export the test report and inspect:

- `routing_stop_reason`: user stop, genuine device loss, capture open/runtime
  failure, renderer failure, format change, a safety threshold, stability-test
  completion, or application close;
- capture `discontinuities`, ring underruns/overruns, and torn-read guards;
- each renderer's underruns, drops, buffer fill, and HRESULT.

Try the Music preset for the next hardware run. A real selected-endpoint
disconnect should still stop routing and produce an explicit reason. If white
noise appears while underruns remain zero, inspect `torn_read_guards` and
`ring_overruns`: a nonzero torn guard is a software-corruption-class failure,
while overruns identify a consumer that fell behind.

## Mixed wired plus Bluetooth

Turn off the Bluetooth-only filter, select one active Bluetooth Media endpoint
and one active Wired/USB/HDMI endpoint, then run **Align** and save the pair.
Large initial offset is expected because wired and Bluetooth paths have very
different transport latency. Start with Video or Music while validating
stability. The card badges and mixed-output buffer label confirm that the set is
intentional.
