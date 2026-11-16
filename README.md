# MacTouchCentral

**An evidence-first bridge for external touch displays on macOS.**

[![macOS 12+](https://img.shields.io/badge/macOS-12%2B-111111?style=flat-square&logo=apple&logoColor=white)](#requirements) [![Apple silicon](https://img.shields.io/badge/Apple%20silicon-arm64-555555?style=flat-square)](#build) [![License: MIT](https://img.shields.io/badge/license-MIT-yellow?style=flat-square)](LICENSE)

MacTouchCentral connects an external USB HID or infrared touch panel to macOS in user space. It turns the panel's reports into pointer input, keeps the touch surface associated with the intended external display, and exposes diagnostics so device support can be tested instead of guessed.

The project is deliberately conservative: a capability demonstrated on Windows is recorded as evidence, not advertised as working on macOS until the corresponding report stream has been observed and reproduced safely.

## At a glance

| Area | Current state |
| --- | --- |
| MK Technology MK S2300-G100 on macOS | Single touch, tap, movement and drag verified |
| MK Technology MK S2300-G100 on Windows 11 | Ten simultaneous contacts verified on the mapped display |
| macOS multitouch for this frame | Investigation open; the connected interface currently exposes one contact |
| Startup behavior | Native login-item setting with enable/disable control |
| Distribution | Source build verified; signing and notarization remain open |

## Why this repository exists

External touch panels often behave differently on each operating system. The video connection determines where a display appears, while the USB connection determines which HID reports are available. Those two pieces can disagree, especially when several monitors are connected.

MacTouchCentral keeps the problem visible in three layers:

1. **Device layer** — identify the USB interface, descriptor and report format.
2. **Mapping layer** — select the external display that should receive pointer events.
3. **Interaction layer** — classify taps, movement and drag gestures without hiding device failures.

This separation makes a failed multitouch experiment diagnosable: it tells us whether the panel did not send contacts, the parser did not understand them, or the interaction policy intentionally ignored them.

## Verified reference device

The current reference device is the **MK Technology MK S2300-G100** infrared frame (`VID:PID 1FF7:0013`).

### macOS

- A stable single-contact stream is received.
- A tap moves the pointer to the touched location.
- Single-finger dragging is supported.
- The current physical ten-finger test still produces single-contact reports on the connected macOS interface.

### Windows 11

- The frame can be assigned to the correct monitor through the Tablet PC touch setup.
- A browser multitouch test detected ten simultaneous contacts.
- This confirms hardware capability in the Windows configuration, but does not reveal the initialization sequence used before the multi-contact stream becomes available.

The detailed evidence and the safe investigation boundary are in [the MK S2300-G100 report](docs/MK-S2300-G100.md) and [PROJECT_STATUS.md](PROJECT_STATUS.md).

## Device matrix

Support is recorded per model and per operating system. “Candidate” means a useful lead, not a compatibility promise.

| Device or family | macOS status | Next evidence |
| --- | --- | --- |
| MK S2300-G100 | Single touch verified; multitouch open | Read-only Windows USB comparison |
| LG Smart Monitor Swing | Existing test lead | Revalidate with the current build |
| ELAN touch panels | Candidate | Capture descriptor and validate input mapping |
| LG CreateBoard | Candidate | Validate USB HID reports and display assignment |
| Other USB HID touchscreens | Generic path only | Add a reproducible model-specific report |

## Safety boundary

The multitouch investigation is read-only until the device protocol is understood. The project must not replay undocumented vendor commands, send speculative `SET_REPORT` requests, update firmware or publish raw captures containing machine-specific information.

Public artifacts exclude credentials, serial numbers, raw packet captures, raw HID logs and local paths. Hardware claims should include a model, operating system, test method and observed result.

## Build

### Requirements

- macOS 12 or later
- Xcode with the macOS SDK
- Apple silicon is the validated build target; Intel builds are not currently part of the test matrix

### From source

```bash
git clone https://github.com/marinimauricio/MacTouchCentral.git
cd MacTouchCentral
open MacTouchCentral.xcodeproj
```

Command-line build:

```bash
xcodebuild \
  -project MacTouchCentral.xcodeproj \
  -scheme MacTouchCentral \
  -destination 'platform=macOS,arch=arm64' \
  MACOSX_DEPLOYMENT_TARGET=12.0 \
  CODE_SIGNING_ALLOWED=NO \
  CODE_SIGNING_REQUIRED=NO \
  build
```

The helper script builds, installs to `/Applications` and launches the app:

```bash
scripts/build_install_run.sh
```

## First run

1. Launch MacTouchCentral and grant **Accessibility** access when macOS asks.
2. Grant **Input Monitoring** only if the connected panel requires it to expose raw reports.
3. If more than one external display is present, choose the intended target in the app's **Touchscreen** menu.
4. Use the startup setting to enable or disable launch at login.
5. Open diagnostics before reporting a device problem; include the counters and selected display, never private raw logs.

## Development map

- Keep single-touch input stable while new devices are investigated.
- Identify the Windows-to-macOS multitouch transition for the MK frame using read-only captures.
- Add a parser only after the real multi-contact report format is observed.
- Confirm that multi-contact interaction does not move the pointer as an unintended side effect.
- Validate two, five and ten contacts on macOS with a repeatable test protocol.
- Turn each additional panel into a model-specific compatibility entry.
- Produce a signed and notarized release after the installation path is stable.

## Repository layout

```text
MacTouchCentral/   macOS app, settings, menu bar control and diagnostics
Core/              HID decoding, touch tracking and event bridge
scripts/           build, install and test helpers
docs/              hardware reports and project evidence
PROJECT_STATUS.md  verified behavior, open work and safety rules
```

## Support

If this project helps your setup, you can support continued hardware testing and macOS development through [PayPal](https://paypal.me/mlsmarini).

## Credits and lineage

The current codebase retains parts of the upstream TouchMyMac and TouchUpCore work by [Sebastian Hueber](https://hci.rwth-aachen.de/touchup) and subsequent contributors. MacTouchCentral is a new project identity and documentation direction, but this rename does not retroactively make the existing implementation entirely original. Future clean-room components will be identified as they are introduced.

## License

[MIT](LICENSE) © 2024 Sebastian Hueber, © 2026 Jinghui Chen. Additional changes and documentation are maintained in this repository with their respective authorship preserved.
