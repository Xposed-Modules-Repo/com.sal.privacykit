# Privacy Kit

Privacy Kit is an LSPosed module for per-app Android identifier spoofing.

It is designed for users who want more control over the identifiers that apps
can read from their device. Instead of changing identifiers globally, Privacy
Kit lets you manage spoofed values on an app-by-app basis so each selected app
can receive its own controlled identity.

## What Privacy Kit Does

Privacy Kit hooks supported Android identifier access paths through LSPosed and
returns configured spoofed values to selected apps. This can help separate app
profiles, reduce cross-app tracking based on stable identifiers, and test how
apps behave when device identity values are changed.

This module does not make a device anonymous by itself. Apps can still use many
other signals, including account login, IP address, network metadata, browser
state, app-specific storage, sensors, permissions, and server-side behavior.

## Key Features

- Per-app identifier spoofing for selected target apps.
- LSPosed/Xposed module integration.
- Package name: `com.sal.privacykit`.
- Release-only public repository for LSPosed distribution.
- Official APK releases published through GitHub Releases.
- Public checksum information for release verification.

## Requirements

- Android device with root access.
- Magisk or another supported root environment.
- LSPosed installed and working.
- A compatible Android version for your LSPosed setup.
- Basic understanding of Xposed module behavior and app compatibility risks.

## Installation

Install Privacy Kit from one of the official public release locations:

- LSPosed module repository listing for `com.sal.privacykit`.
- GitHub release: [Privacy Kit 5-1.4](https://github.com/Xposed-Modules-Repo/com.sal.privacykit/releases/tag/5-1.4).
- Direct APK asset: [app-release.apk](https://github.com/Xposed-Modules-Repo/com.sal.privacykit/releases/download/5-1.4/app-release.apk).

After downloading, install the APK on the device where LSPosed is already set
up. Do not install APKs from unknown mirrors unless you can verify the checksum
against the official release information below.

## Enabling the Module in LSPosed

1. Open the LSPosed manager app.
2. Go to the Modules section.
3. Select Privacy Kit.
4. Enable the module.
5. Select the apps you want Privacy Kit to affect.
6. Reboot the device if LSPosed asks you to, or force stop and restart selected
   target apps when appropriate.

Only enable the module for apps you intend to manage. Keeping the scope narrow
usually makes troubleshooting easier and reduces unexpected behavior.

## Basic Usage Flow

1. Install Privacy Kit from the official LSPosed/GitHub release.
2. Enable Privacy Kit in LSPosed.
3. Select the target apps that should receive spoofed identifiers.
4. Open Privacy Kit and configure the identifier values or profiles you want to
   use.
5. Restart the target apps so they read the updated values.
6. Test each app carefully and adjust settings if an app behaves unexpectedly.

For best results, change one app or profile at a time and confirm that the app
still works before applying broader changes.

## Privacy and Security Notes

- Identifier spoofing is only one privacy layer and should not be treated as a
  complete anonymity solution.
- Apps may combine identifiers with account data, cookies, local storage, IP
  addresses, device signals, and server-side records.
- Some apps may block access, fail integrity checks, reset sessions, or behave
  unpredictably when identifiers change.
- Use separate accounts, network protections, browser hygiene, and permission
  controls when your privacy model requires stronger separation.
- Keep LSPosed, Magisk, Android security patches, and installed modules up to
  date whenever possible.

## Troubleshooting

### The module does not appear in LSPosed

- Confirm that the APK installed successfully.
- Confirm that LSPosed is active.
- Reboot the device and check the LSPosed manager again.

### A target app is not affected

- Make sure Privacy Kit is enabled for that specific app in LSPosed.
- Force stop the target app and open it again.
- Reboot if the app was running before the module was enabled.
- Check whether the app reads identifiers through a path not currently handled
  by the module.

### An app crashes or blocks access

- Disable Privacy Kit for that app and restart it.
- Try a fresh profile or less aggressive identifier changes.
- Clear the app's data only if you understand the consequences, including loss
  of local sessions or settings.
- Some apps may not be compatible with identifier spoofing.

### The APK cannot be installed

- Make sure you downloaded the APK from the official release page.
- Remove conflicting old builds if Android reports a signature or downgrade
  issue.
- Verify that your Android and LSPosed setup are supported and functional.

## Release and APK Verification

Official release:

https://github.com/Xposed-Modules-Repo/com.sal.privacykit/releases/tag/5-1.4

Official APK:

https://github.com/Xposed-Modules-Repo/com.sal.privacykit/releases/download/5-1.4/app-release.apk

Expected SHA-256 for `app-release.apk`:

```text
D030E290432B97D77E80ED515FD6B57CAB26A0905A47879771001D70B7D33B01
```

On a desktop system, compare the downloaded file's SHA-256 hash with the value
above before installing. If the hash does not match, do not install the APK.

## Support and Donations

Privacy Kit takes time to build, test, maintain, and support. If the module is
useful to you and you are able to help, a donation would mean a lot. Your
support helps the developer continue this work and helps support their family
with everyday needs, including food and living expenses.

Donation link: TODO: add donation link

Thank you for using Privacy Kit, sharing feedback, and supporting independent
Android module development.

## Source Availability

This public repository is intentionally release-only. It hosts public
documentation, release metadata, and links to official APK releases for LSPosed
distribution.

The Android application source code, build project, signing material, private
tooling, and generated build outputs are not published in this repository. The
source code is maintained privately by the developer.

## Disclaimer

Use Privacy Kit only on devices and apps where you understand and accept the
privacy, compatibility, legal, and policy implications. You are responsible for
how you use this module. Some apps and services may restrict access, enforce
integrity checks, terminate sessions, or behave unexpectedly when device
identifiers are changed.
