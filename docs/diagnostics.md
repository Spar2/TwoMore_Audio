# Diagnostics and privacy

The GUI warns before writing `%APPDATA%\TwoMore\twomore-diagnostics.json`.
Portable mode writes beside the executable. The report contains:

- schema, application version, and the short source build commit;
- runtime-detected Windows product name, display version, major/minor/build/UBR,
  formatted version text, and CPU architecture;
- Bluetooth adapter address inferred from WinRT association IDs when available;
- each paired Bluetooth audio association endpoint discovered through WinRT,
  including its ID and paired/connected state;
- every MMDevice render endpoint ID, name, interface, container ID, state,
  default flag, Bluetooth confidence, profile hints, Windows volume, negotiated
  sample rate/channels/bits, transport, and an honest format/quality tier;
- detected virtual-cable capture endpoints and their paired playback endpoint
  identity when available;
- the mapping between both layers, including connected-without-active-media and
  active-media-with-unknown-Bluetooth-state markers.

Recent UI activity is stored in `%APPDATA%\TwoMore\twomore.log`. The file is
rotated at 2 MiB and up to three numbered archives (`twomore.log.1` through
`twomore.log.3`) are retained. The Activity panel can copy recent in-memory
entries or open the data folder.

Stable Media selection and per-device playback controls are stored locally in
`selected-endpoints.tsv` and `device-controls.tsv`; neither file is included
in diagnostics exports.

Device names and stable IDs can reveal local hardware information. Review both
files before attaching them to a public issue. No audio samples, media contents,
microphone input, account credentials, or network data are written to
diagnostics.

CLI export:

```powershell
twomore.exe diagnose twomore-diagnostics.json
twomore.exe check-bt --verbose
```

These commands use the build-tree name. In the portable release, the optional
diagnostic executable is packaged as `tools\twomore-cli.exe`; the main product
is `twomore-gui.exe`.

The portable release also includes `collect-support-bundle.ps1`. It creates a
timestamped support folder containing diagnostics, the test report when
available, bounded logs, and a SHA-256 manifest. Settings, selected-device
presets, audio samples, and source code are excluded. Review names and IDs
before sharing the folder.

Diagnostics schema version 8 contains top-level `windows`,
`bluetooth_devices`, `render_endpoints`, `virtual_cables`,
`virtual_cable_capture_endpoints`, and `verified_configurations`. In the GUI,
enable **Technical IDs** to display the same mapping state and the raw
MMDevice/Bluetooth IDs on each card.

The separate GUI **Export report** action writes renderer buffer counts,
HRESULTs, the explicit audible answer, guided verdict, active system-wide
preset, capture format/counters, pipeline-quality verdict, routing stop reason,
and per-renderer underrun, overrun/drop, fill, and indicative-drift values.
Renderer success without a BOTH answer remains PARTIAL.

The system-wide report also includes the selected capture endpoint ID/name,
negotiated and attempted formats, HRESULT, silent/non-silent buffer counts,
ring underruns/overruns, torn-read guards, peak/RMS, and the last capture error.
Static diagnostics include cable
container/interface identities, matching playback endpoint identity, default
state, and reported cable-input volume.
