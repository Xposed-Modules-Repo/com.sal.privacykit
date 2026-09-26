![Privacy Kit banner](assets/privacy-kit-banner.png)

# Privacy Kit

Privacy Kit is an LSPosed module for Android. It gives each app you pick its own
fake device identity, instead of changing identifiers for the whole phone. Apps
that read stable device IDs see a made-up device, not your real one.

It does not make you anonymous. Apps can still track you by login, IP address,
network metadata, browser state and server-side checks. Privacy Kit only covers
on-device identifiers.

Package: `com.sal.privacykit`. Current version: `2.3.1` (versionCode 68).

## What a release contains

Only the app APK is required. The rest are optional layers that reach deeper
below the app.

| File | What it is | Needs |
|---|---|---|
| `PrivacyKit-<ver>.apk` | The app and LSPosed module. Spoofs Java-layer IDs per selected app. | LSPosed |
| `privacykit-zygisk-<ver>.zip` | Native companion. Covers system properties, `/proc`, `/sys`, Wi-Fi MAC and the kernel string. | Magisk, KernelSU or APatch (Zygisk) |
| `privacykit_kpm.kpm` | Kernel module, advanced. File timestamps, `/proc/cpuinfo`, CPU frequencies. | APatch with KPM |
| `PrivacyKit-Probe-<ver>.apk` | Self-test app. Shows which IDs are actually spoofed and which still leak. | none |

## Features

- Per-app profiles for the apps you choose.
- Spoofs Android ID, GSF ID, Advertising ID, App Set ID, Widevine/DRM ID, serial,
  build properties, Wi-Fi and Bluetooth values, hostname, IMEI, MEID, IMSI, ICCID
  and phone number.
- Per-identifier rules: real value, fixed value, random per launch, random daily,
  or your own value.
- Device templates for common phones, or build your own.
- Optional root-based data isolation between profiles.
- Optional per-profile SOCKS5 proxy (Pro).
- Backup and restore, import and export, launch history, and a built-in verify screen.

## Requirements

- Android 8.0 or newer.
- LSPosed, on Magisk, KernelSU or APatch.
- Root is optional. It is only needed for data isolation and the native and kernel layers.

## Install

1. Download the APK from Releases and install it.
2. Open Privacy Kit once so LSPosed can detect it.
3. Enable the module in LSPosed and tick the apps you want in scope.
4. Reboot, or force-stop the target apps.
5. In Privacy Kit, add an app, create a profile, and launch it.

For the native layer, flash `privacykit-zygisk-<ver>.zip` in your root manager and reboot.

## Source

Source code, build instructions and the licensing server:
https://github.com/Mohithash/Privacy_Kit

## A note from me

I am an independent developer from India, and I work on this alone. I want to be
honest about why you will see a paid Pro tier and a donate option here.

The last while has been hard. I lost my job, I was cheated out of money, and I am
dealing with a legal case, all while supporting my wife and our home. I still like
building this and I want to keep it going.

If Privacy Kit is useful to you, buying Pro or sending a small donation genuinely
helps me and my family. It is never required, and the free features stay free. If
you would rather help with code, testing or ideas, that is very welcome too.

Thank you for trying it.

## Support and contact

- Pro, donations or questions: Telegram @couponxdealer (message me for UPI or other options).
- Updates: Telegram @privacykit.

Please do not send money to anyone who is not listed here.

## License

No open-source license is set. All rights reserved by the author. If you want to
reuse, redistribute or modify it publicly, please ask first.

## Disclaimer

Use Privacy Kit only on devices and accounts you are allowed to modify. It is for
privacy, testing and personal device control. I am not responsible for account
bans, data loss, device problems, or any app's terms being broken.
