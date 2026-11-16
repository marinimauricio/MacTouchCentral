# MacTouchCentral — project status

Last updated: 2026-11-16

## Objective

Provide a user-space macOS bridge for external USB touchscreens, with reliable single-touch input now and a safe path toward multitouch support for the MK Technology MK S2300-G100 IR frame.

## Completed

- Recovered and versioned the MacTouchCentral macOS application and its TouchUpCore HID/input framework.
- Confirmed the target device identity: MK Technology MK S2300-G100, USB VID `1FF7`, PID `0013`.
- Implemented and validated single-finger touch on macOS: tap-to-click, movement and hold-then-drag behavior.
- Corrected the single-finger drag mapping so a drag gesture is sent as a drag action instead of being misclassified as scrolling.
- Added diagnostics for HID connection state, touch counts, gesture decisions and raw report capture.
- Added macOS login-item control through the native ServiceManagement API, with an enable/disable option and System Settings approval flow on supported macOS versions.
- Confirmed the hardware's multitouch capability on Windows 11: the browser test reported 10 simultaneous contacts after the touch input was mapped to the correct monitor.
- Built the app successfully for Apple silicon with the macOS deployment target used by the project.

## Current verified behavior

### macOS

- Single touch works.
- The app can move the pointer to the touched location.
- Single-finger dragging works after the gesture mapping correction.
- The current connected MK device sends 8-byte reports with report ID `0x01` to macOS.
- During the physical ten-finger test, all observed reports were the single-contact format; no multi-contact report stream was received.

### Windows 11

- The same physical frame was mapped to the correct external monitor using Windows' Tablet PC touch setup.
- A browser multitouch test detected 10 simultaneous contacts.
- This proves that the frame hardware and its Windows-facing configuration can provide 10-point multitouch; it does not by itself reveal the macOS activation sequence.

## Open work

1. Capture the Windows USB control and interrupt traffic while the frame changes from single-touch behavior to 10-point behavior.
2. Identify whether Windows uses a standard HID report descriptor, a vendor feature report, a firmware profile, or a device-specific initialization command.
3. Reproduce only a documented or independently verified initialization sequence on macOS.
4. Add a parser for the device's real multi-contact report format once the format is observed from hardware.
5. Test two-, five- and ten-contact behavior on macOS and verify that multi-contact gestures do not move the mouse pointer independently.
6. Produce a signed/notarized distribution build and validate login-item registration in the installed `/Applications` location.

## Safety boundary

No undocumented USB `SET_REPORT`, firmware update, or vendor command should be sent to the device. The next multitouch investigation must be read-only until a safe command sequence is known. Raw captures, serial numbers, local paths, and machine-specific logs stay outside the public repository.

## Reproducible checks

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

The public repository intentionally contains source and documentation only. DerivedData, app bundles, raw HID logs, packet captures, credentials and machine-specific configuration are ignored.
