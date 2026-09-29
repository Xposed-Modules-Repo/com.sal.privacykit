![Privacy Kit banner](assets/privacy-kit-banner.png)

# Privacy Kit

Privacy Kit is an LSPosed module for Android. It gives each app you pick its own
fake device identity, instead of changing identifiers for the whole phone. Apps
that read stable device IDs see a made-up device, not your real one.

It does not make you anonymous. Apps can still track you by login, IP address,
network metadata, browser state and server-side checks. Privacy Kit only covers
on-device identifiers.

Package: `com.sal.privacykit`. Current version: `3.2` (versionCode 89).

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

## Spoof modes

Every app you add has a spoof mode. The mode decides which layers do the spoofing,
from the widest and most detectable to the quietest. Standard is the default. Move
an app to a quieter mode only if it notices the hooks or crashes.

There are five layers underneath:

- T1 LSPosed Java hooks. Widest coverage, easiest for an app to detect. Needs the
  app ticked in LSPosed.
- T2 Zygisk native inline hooks. GL strings and some properties. A few hardened
  apps crash on this layer.
- T3 Zygisk native JNI. `Build.*` fields and system properties, with no inline
  trampolines, so it survives most anti-hook checks.
- T4 Framework. Android ID, serial and telephony IDs, spoofed inside system_server.
  Nothing is added to the app.
- T5 KPM. A kernel module. cpuinfo, CPU frequencies, `if_inet6`, install dates, the
  Wi-Fi MAC, and Android ID / advertising ID at the kernel. APatch only. See below.

| Mode | Layers | In LSPosed scope | Covers | Reads real |
|---|---|---|---|---|
| Standard | T1 + T2 + T3 | yes | everything Privacy Kit can spoof | nothing extra |
| Balanced | T2 + T3 + T4 | no, remove it | Build, properties and IDs, no Xposed in the process | nothing extra |
| Stealth | T3 + T4 + T5 | no, remove it | Build and properties via JNI, IDs, kernel files, no inline hooks | nothing extra |
| Ultra | T4 + T5 | no, remove it | Android ID and IDs at the source, plus kernel files, nothing in the app | Build, model and properties |

Set the mode on each app's Spoof Mode screen. Balanced, Stealth and Ultra want the
app out of the LSPosed scope; Privacy Kit makes its own Java hooks inert for that
app and reminds you. Stealth and Ultra use the kernel module, so they need APatch.
Ultra is the quietest, but since nothing runs in the app it shows the real model
and build, so a check that compares your Android ID against your model can still
notice. Stealth adds the native layer back for a coherent device without inline
hooks.

## KPM guide (APatch kernel module)

The KPM is the T5 kernel layer. It runs below libc and below the app, so an app's
anti-hook checks find nothing in its own process. It is optional, and only the
Stealth and Ultra modes use it.

You need:

- APatch with KernelPatch (KPM) support. Magisk, KernelSU and SukiSU cannot run a
  KPM. The KPM controls still appear on those, but the kernel tier is marked
  unavailable and turning it on just does nothing.
- The module file `privacykit_kpm.kpm`. It ships next to the release, not inside
  the APK. It is built against a kernel, so you may have to build it for yours. The
  module source is in `zygisk-privacykit/kpm/` in the Privacy_Kit repo.
- Your APatch superkey.
- A way back in, such as fastboot or a custom recovery. A bad kernel module can
  stop the phone from booting.

Set it up:

1. Push the module once with `adb push privacykit_kpm.kpm /data/local/tmp/privacykit_kpm.kpm`. Any root file copy works too.
2. Open Privacy Kit, then Settings, Developer, the KPM section. Paste your APatch
   superkey and tap Save superkey. It is checked once, stored encrypted, never
   shown again, and left out of backups.
3. Set the apps you want to Stealth or Ultra, or turn on Kernel module (KPM) in
   Developer settings. Privacy Kit loads the module through APatch
   (`kpatch <superkey> kpm load`). Setup Doctor has a one-tap Load if it is not up.
4. The module does not stay loaded across a reboot on its own. Privacy Kit reloads
   it on boot when you have KPM apps and a saved superkey. If it is ever down,
   Setup Doctor shows it and offers Load.

What it spoofs:

- Files: `/proc/cpuinfo`, `/proc/version`, `/proc/net/if_inet6`, CPU frequencies,
  the Wi-Fi MAC at `/sys/class/net/wlan0/address`, and the app install date.
- IDs, per app: Android ID, advertising ID and the Widevine device ID, rewritten
  in the kernel. This reaches the normal read path; the cursor query path stays
  real. It needs a current module (1.6.0) and only runs once Privacy Kit arms it,
  so an older loaded KPM may leave these IDs real.
- Everything is per app and only for normal apps. Root and system still read the
  real device.

What it does not do:

- It does not touch `Build.*` or system properties. Those come from the native or
  framework layers, so Ultra on its own shows the real model and build. Use Stealth
  if you want those spoofed too.
- It does not defeat hardware keystore or Play Integrity attestation.
- It does not spoof IMEI, serial, IMSI, ICCID or Bluetooth. A normal app cannot
  read those anyway.
- The cpuinfo, frequency and version text is filled in by your AI provider if you
  set one. Without it, the install date, `if_inet6` and Wi-Fi MAC are still
  spoofed, and the rest is skipped rather than faked badly.

## Source

Source code, build instructions and the licensing server:
https://github.com/Mohithash/privacy-kit-lite

## Build your own ROM with it

Besides this LSPosed module, the same per-app spoofing runs inside the Android
framework, with no injection, in BestROM, an open-source AOSP 17 ROM (first device
POCO F6 / peridot). Nothing is injected into the app process, so this is the
cleanest mode. You can build your own ROM with Privacy Kit baked in; the source
tree ships an MCP server so a coding agent can sync, build and verify it.

- BestROM: https://github.com/Mohithash/bestrom-project
- Build (manifest): https://github.com/Mohithash/manifest

## A note from me

I am an independent developer from India, and I work on this alone. I want to be
honest about why you will see a paid Pro tier and a donate option here.

The last while has been hard. I lost my job, I was cheated out of money, and I am
dealing with a legal case, all while supporting my wife and our home. I still like
building this and I want to keep it going.

If Privacy Kit is useful to you, buying Pro or sending a small donation genuinely
helps me and my family. It is never required, and the free features stay free. If
you would rather help with code, testing or ideas, that is very welcome too.

I made this private project public to get through a hard time. Once it reaches my goal of about $5,000 in total earnings I plan to open-source it, so buying Pro or donating now both helps my family and brings that closer.

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
