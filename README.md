![Privacy Kit banner](assets/privacy-kit-banner.png)

# Privacy Kit

Privacy Kit is a multi-layer Android anti-fingerprinting toolkit built around an
LSPosed module. It lets you give each selected app its own controlled device
identity instead of changing identifiers globally, so apps that rely on stable
device identifiers see a coherent, per-app fake device rather than your real one.

It is designed for users who want more control over what apps can read from their
device — for separating app profiles, reducing cross-app tracking based on stable
identifiers, and testing how apps behave when device identity values change.

Privacy Kit does not make a device anonymous by itself. Apps can still use many
other signals — account login, IP address, network metadata, browser state,
app-specific storage, sensors, permissions, and server-side behavior.

## What's in a Release

A Privacy Kit release ships up to four components. Only the first is required;
the rest are optional layers that reach deeper below the app.

| Component | File | What it does | Needs |
|---|---|---|---|
| **Privacy Kit module** | `PrivacyKit-<ver>.apk` | The app + LSPosed module. Hooks Java-layer identifier APIs per selected app and returns coherent spoofed values. | LSPosed |
| **Zygisk native module** | `privacykit-zygisk-*.zip` | Closes native/file read paths the Java hooks can't reach — system properties, `/proc`, `/sys`, `boot_id`, sysfs Wi-Fi MAC, kernel, `/proc/meminfo`. | Magisk/KernelSU/APatch Zygisk |
| **Kernel module (KPM)** | `privacykit_kpm.kpm` | **Optional, advanced.** Spoofs file timestamps, `/proc/cpuinfo`, CPU cluster frequencies and `/proc/net/if_inet6` in the kernel, below every app and service. | APatch with KPM support |
| **PK Probe** | `PKProbe-<ver>.apk` | Companion self-test app. Verifies which identifiers are actually being spoofed vs. still leaking. | — (optional) |

## How the Layers Work

Privacy Kit spoofs the same identifier at more than one level, because a real
fingerprinting SDK reads the same value through several paths (a Java getter,
a system property, a `/proc` or `/sys` file, or directly via the kernel).

- **Per-app (LSPosed)** — hooks Java getters (`Build.*`, `TelephonyManager`,
  `Settings.Secure`, `WifiInfo`, media DRM, GMS identifiers, …) inside each
  target app you enable in LSPosed scope. This is the core layer.
- **Native (Zygisk)** — rewrites what the app sees from `SystemProperties`,
  `getprop`, and file reads under `/proc` and `/sys`, so a native or file-based
  read returns the same fake value as the Java getter.
- **Framework mode (optional, experimental)** — spoofs at the provider process
  (`system_server` / the phone process), per calling app, so the target app
  reads the fake value with **no hook in its own process**. Covers `android_id`,
  build serial, and telephony identifiers. Requires adding "Android System
  Framework" and the Phone app (`com.android.phone`) to Privacy Kit's LSPosed
  scope, then a reboot.
- **Kernel (KPM, optional)** — for APatch users, spoofs surfaces below libc and
  below any service (file timestamps, `/proc/cpuinfo`, cpufreq, IPv6 link-local).

Coherence is the point: a spoofed profile draws one real, consistent device
(brand, model, SoC, fingerprint, baseband, RAM, screen, etc.) so the values do
not contradict each other across paths.

## Identifiers Covered

Coverage depends on the layers you enable and the app's read path. Grouped by kind:

- **Advertising & app identity** — Advertising ID (GAID), App Set ID, Google
  Services Framework ID (GSF), `ANDROID_ID`, Firebase Installation ID, Firebase
  App Instance ID, Media DRM / Widevine device ID.
- **Device & build** — model, brand, manufacturer, device/product/board/hardware,
  build fingerprint, build ID/display, bootloader, serial, build time, SoC model
  & manufacturer, baseband/radio version.
- **OS & boot** — Android release, API level (SDK), security patch, build
  incremental, kernel version (`os.version`/`uname`), `boot_id`, boot time,
  uptime.
- **Telephony & SIM** — IMEI, MEID, IMSI (subscriber ID), ICCID (SIM serial),
  phone number, network & SIM operator (MCC/MNC + name), network & SIM country,
  SIM carrier ID, phone count, type-allocation code.
- **Network** — Wi-Fi MAC, BSSID, SSID, Bluetooth MAC & name, hostname, IPv6
  link-local address (`/proc/net/if_inet6`).
- **Hardware** — `/proc/cpuinfo` (SoC / CPU part), CPU cluster max frequencies,
  total RAM (`MemTotal`), display metrics (xdpi/ydpi/density, resolution).
- **Locale & environment** — time zone, locale, HTTP / WebView user-agent,
  first-install and last-update time.
- **Location** — GPS coordinate spoofing, Region Presets, per-app walk/jog/drive
  Activity Simulation, saved Favorite Locations, map-based coordinate picker.

Some surfaces are intentionally **not** changed because doing so is a bigger tell
or a stability risk (for example, CPU ABI list and core count are left real).

## Key Features

- Per-app identifier spoofing for selected target apps, with coherent whole-device
  profiles rather than random unrelated values.
- One-tap AI Auto Profile and Maximum Privacy profile flows.
- Native (Zygisk) layer for file/property/`/proc`/`/sys` reads, and an optional
  Framework mode and Kernel (KPM) tier for deeper coverage.
- **PK Probe** companion for verifying exactly which values are spoofed vs. leaking.
- Isolated browser account sessions using Android System WebView on supported
  Android versions, with WebView compatibility hotfixes for checkout/login/payment.
- Live in-process updates so location, Build fields, and identifier changes apply
  to an already-running target app without a relaunch.
- Region Presets, Activity Simulation scheduling, Favorite Locations, and a
  map-based coordinate picker for location spoofing.
- Hook Diagnostics, an in-app hook log viewer, Export Diagnostics, and history
  views for troubleshooting LSPosed scope, profile, and hook activity.
- Package name: `com.sal.privacykit`. Companion package: `com.test.gmsprobe`.

## Requirements

- Android device with root (Magisk, KernelSU, or APatch).
- LSPosed installed and working (for the module).
- A Zygisk implementation (for the native layer).
- APatch with KPM support (only for the optional kernel module).
- Basic understanding of Xposed module behavior and app-compatibility risks.

## Installation

Install from an official public release:

- LSPosed module repository listing for `com.sal.privacykit`.
- GitHub release: [Privacy Kit 24-1.22](https://github.com/Xposed-Modules-Repo/com.sal.privacykit/releases/tag/24-1.22).
- Direct APK asset: [PrivacyKit-1.22.apk](https://github.com/Xposed-Modules-Repo/com.sal.privacykit/releases/download/24-1.22/PrivacyKit-1.22.apk).

Install the APK on the device where LSPosed is set up. Do not install APKs from
unknown mirrors unless you verify the checksum against the values below.

## Enabling the Module in LSPosed

1. Open the LSPosed manager app.
2. Go to Modules and select Privacy Kit.
3. Enable the module.
4. Select the apps you want Privacy Kit to affect.
5. Reboot if LSPosed asks, or force stop and reopen the selected target apps.

Keep the scope narrow — only enable it for apps you intend to manage.

## Verifying Spoofing with PK Probe

PK Probe (`PKProbe-1.22.apk`) checks what is actually being spoofed, by baseline
diff rather than guesswork:

1. Install PK Probe **unhooked** (not in Privacy Kit's LSPosed scope) and tap
   **SAVE BASELINE** — it records the real values via every read path.
2. Add `com.test.gmsprobe` to Privacy Kit's LSPosed scope, apply a full spoof
   profile, and relaunch.
3. Tap **COMPARE** — each path is diffed against its baseline: **changed = hooked**,
   **unchanged = the real value is still leaking**.

Per-path granularity catches the common failure where a Java getter is spoofed but
a `/proc`, `/sys`, `getprop`, or native read still leaks the real value.

## Optional: Framework Mode

Framework mode moves selected identifiers to the provider process so target apps
carry no in-process hook. It is **experimental**: these are system-wide provider
hooks, it requires adding "Android System Framework" and `com.android.phone` to
Privacy Kit's LSPosed scope plus a reboot, and a fault could bootloop. Leave it
off unless you are testing on a device you can recover.

## Optional: Kernel Module (KPM) — APatch only

> ⚠️ **Advanced and dangerous. Experimental — not yet verified on device.**
> The KPM (`privacykit_kpm.kpm`) runs in the kernel. A bad kernel module can
> **brick boot**. It requires **APatch with KPM support**. Load it only on a
> device you can recover (fastboot / OrangeFox). It is written fail-open (any
> error leaves the real syscall in place) and stays **off by default** in the app.

Load and verify:

```
adb push privacykit_kpm.kpm /data/local/tmp/
apd kpm load /data/local/tmp/privacykit_kpm.kpm
apd kpm list
```

Then enable **Developer Settings → Kernel module (KPM)** in Privacy Kit and confirm
with PK Probe. This build targets **KernelPatch 0.13.5**. If it fails to load,
rebuild it against your APatch's exact KernelPatch version.

## Warnings and Limitations

- Identifier spoofing is **one** privacy layer, not full anonymity. Apps also use
  accounts, cookies, local storage, IP address, sensors, and server-side records.
- Some apps may block access, fail integrity checks (e.g. Play Integrity), reset
  sessions, or behave unpredictably when identifiers change.
- Framework mode and the KPM are system/kernel-level and higher risk (bootloop /
  brick-boot). Both are optional and off by default.
- Spoofing certain low-level properties can crash an app's graphics or ABI paths;
  Privacy Kit deliberately avoids the riskiest ones.
- Use separate accounts, network protections, and browser hygiene when your
  privacy model requires stronger separation.

## Basic Usage Flow

1. Install Privacy Kit from the official release and enable it in LSPosed.
2. Select the target apps that should receive spoofed identifiers.
3. (Optional) Flash the Zygisk module for native coverage.
4. Configure identifier values or apply a profile in the app.
5. Restart the target apps so they read the updated values.
6. Verify with PK Probe, then adjust one app or profile at a time.

## Troubleshooting

### The module does not appear in LSPosed
- Confirm the APK installed and that LSPosed is active; reboot and re-check.

### A target app is not affected
- Ensure Privacy Kit is enabled for that specific app in LSPosed.
- Force stop and reopen the app; reboot if it was running before enabling.
- Use PK Probe to see which path is still leaking, and enable the native/framework
  layer for that surface.

### An app crashes or blocks access
- Disable Privacy Kit for that app and restart it.
- Try a fresh profile or less aggressive changes; some apps are incompatible.

### The APK cannot be installed
- Download from the official release page.
- Remove conflicting old builds on a signature or downgrade error.

## Release and APK Verification

Official release: https://github.com/Xposed-Modules-Repo/com.sal.privacykit/releases/tag/24-1.22

Expected SHA-256:

```text
PrivacyKit-1.22.apk      95B1C0816A684186A8EA082F14F68E9728A40212944A57865C5BEA1F08CAB990
PKProbe-1.22.apk         1E1104E0FAAFB7088C45B40D01307DEC01BF697F7853AE4BB48400594C54AABB
privacykit_kpm.kpm       C9D4EB74CD04D73C2AEEE8BA23B9A5D2976286C040889658E5157BFBBD52AE67
```

Compare the downloaded file's SHA-256 with the value above before installing. If
the hash does not match, do not install.

## Source Availability

This public repository is intentionally release-only. It hosts public
documentation, release metadata, and links to official APK releases for LSPosed
distribution. The application source, build project, signing material, and private
tooling are maintained privately by the developer.

## Disclaimer

Use Privacy Kit only on devices and apps where you understand and accept the
privacy, compatibility, legal, and policy implications. You are responsible for
how you use it. Some apps and services may restrict access, enforce integrity
checks, terminate sessions, or behave unexpectedly when device identifiers change.
The Framework mode and Kernel (KPM) features carry additional bootloop / brick-boot
risk and are provided for advanced users on recoverable devices only.
