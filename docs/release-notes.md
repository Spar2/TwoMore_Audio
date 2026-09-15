# TwoMore Audio 0.7.0

Engineering preview for Windows x64.

## What is included

- Drift-locked, phase-continuous realtime rendering with bounded recovery.
- Per-device gain, mute, delay, level metering, and quality diagnostics.
- N-device routing with graceful degradation when a selected endpoint
  disappears.
- Audio control center, quiet-cable boost, and opt-in auto-level.
- Native dark/light GUI, keyboard focus, guided checks, and mock smoke mode.
- Bounded persistent logging with three retained diagnostic archives.
- Diagnostics, About, and test reports identify the source build commit.
- Schema-8 diagnostics include negotiated endpoint format, Windows volume,
  transport, and honest quality-tier fields without guessing a Bluetooth codec.
- The portable archive now ships a user-facing quick-start README instead of
  private engineering/build instructions.
- CLI `list --json` now exposes the same endpoint format and transport facts as
  diagnostics, with an explicit schema version for automation.
- Public showcase generation rejects URLs that resolve to the private source
  repository, adding a second guard against an accidental self-link.
- Release verification now rejects a stale binary whose embedded source
  revision differs from the manifest revision.
- The retained Inno Setup definition now targets the verified CMake output and
  curated user documentation; it remains unshipped until compiled and signed.
- Portable releases include a privacy-aware support-bundle collector with a
  manifest and per-file SHA-256 hashes.
- Removed the obsolete duplicate CI workflow so local verification and GitHub
  Actions use one canonical Windows preset and packaging path.
- CI now smoke-tests the support-bundle collector from inside the extracted
  portable archive, not only from the source checkout.
- Portable packaging defaults now match the checked-in `windows-release`
  preset, so the documented no-argument command uses the canonical paths.
- Product-facing release links now point to the separate public showcase rather
  than the private source repository.
- Verified portable ZIP with SHA-256 and JSON metadata; installer metadata is
  retained in the private source tree for a later signed distribution.

## Before installing

Pair and connect the Bluetooth Media endpoints in Windows first. Hands-Free
endpoints are intended for calls and normally provide lower-quality mono audio.
For system-wide routing, install VB-CABLE or Voicemeeter separately from its
official vendor and follow the in-app setup check.

## Known boundaries

Bluetooth radios have independent clocks and buffering. TwoMore Audio reduces
drift and exposes the observed health of each path, but it does not promise
perfect acoustic phase alignment, codec selection, automatic pairing, a virtual
driver, or DRM bypass.

The release contains compiled binaries and documentation only. The source code
is private and is covered by the proprietary binary license in `LICENSE`.
