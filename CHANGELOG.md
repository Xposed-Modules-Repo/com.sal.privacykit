# Changelog

## 6-1.5

Privacy Kit 6-1.5 is a focused LSPosed release that improves browser isolation,
profile setup, diagnostics, backup safety, and release hardening while keeping
the public repository release-only.

### Highlights

- Reduced APK size by replacing GeckoView with Android System WebView. The
  official release APK is 16,861,742 bytes, approximately 16.08 MiB.
- Added isolated browser account sessions through a dedicated `:browser`
  process and WebView `dataDirectorySuffix` support on Android 9 and newer.
- Added AI Auto Profile as a one-tap profile creation flow.
- Added Maximum Privacy as a one-tap profile preset that sets supported
  identifiers to static spoofed values.
- Improved Hook Diagnostics and history reporting for easier LSPosed
  troubleshooting.

### New Features

- AI Auto Profile can generate a profile in one guided action.
- Maximum Privacy profile preset applies static spoofed values for supported
  identifiers.
- Profile screens now show per-profile app data size where available.
- Added a clear app data action for profile-managed app data.
- Added improved labels for custom and AI-generated device profiles in profile
  lists and clone flows.

### Fixes

- Hardened backup and restore against malformed archives and path traversal by
  validating zip/tar paths before extraction.
- Improved crash handling around backup and restore flows.
- Fixed fresh profile cleanup and mount reliability issues.
- Fixed long-list scrolling issues in screens that can contain many profiles,
  apps, identifiers, or diagnostics entries.
- Improved Hook Diagnostics and history state for LSPosed service status, scope
  checks, active profile display, exported config state, hook tracking, last
  launch/event reporting, and identifiers that have not yet been observed.

### Security and License Hardening

- Added tamper-aware license cache handling.
- Added release signing certificate integrity checks.
- Added clock rollback detection.
- Improved server-side license validation, rate limiting, and signed response
  handling.

### Browser Changes

- Removed GeckoView from the distributed APK and moved browser functionality to
  Android System WebView.
- Browser account sessions are isolated per account/profile on Android 9+ using
  WebView data directory suffixes.
- Browser work runs in a dedicated `:browser` process to keep WebView state more
  clearly separated from the main app process.

### Verification

- Official APK asset: `app-release.apk`
- SHA-256:
  `37AA9A963252520C3F77C44D85D5A579591F4CA601D7D488989D2CC8E6393C93`
- Public repository contents verified as release metadata only:
  `README.md`, `CHANGELOG.md`, and `NOTICE.md`.

### Limitations

- Android System WebView behavior depends on the installed WebView provider and
  Android version.
- WebView `dataDirectorySuffix` isolation requires Android 9 or newer.
- Identifier spoofing remains limited to supported identifiers and configured
  LSPosed scope. Apps can still use accounts, IP address, network metadata,
  cookies, local storage, sensors, permissions, and server-side signals.
- This public repository remains release-only. Source code, signing material,
  build project files, private tooling, and generated build outputs are not
  published here.

## 5-1.4

- Release build for LSPosed distribution.
- Provides per-app Android identifier spoofing under package
  `com.sal.privacykit`.
- Public repository changed to release-only metadata.
