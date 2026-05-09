# Lab Report: Bypassing Root Detection using Objection

**Student Name:** Ziyad Daber
**Date:** May 9, 2026
**Objective:** Successfully implement a bypass for Android root detection mechanisms using the Objection framework and Frida.

---

## 1. Environment Setup
I have configured the following environment to perform the security analysis:
- **OS:** Windows (with Administrator privileges)
- **Language:** Python 3.8+
- **Android Tooling:** ADB (Android Platform Tools) installed and configured.
- **Hardware:** Android device (version 8.0+) with Developer Options and USB Debugging enabled.
- **Instrumentation:** Frida installed on PC and `frida-server` deployed and running on the Android device with matching versions.

## 2. Implementation Steps

### Step 1: Objection Installation
I installed the Objection CLI using the recommended isolated environment via `pipx`:
```bash
pip install --user pipx
pipx ensurepath
pipx install objection
```
Verified the installation using `objection --version`.

### Step 2: Device Preparation
I identified the device ABI and deployed the appropriate `frida-server` to the device:
1. Identified ABI: `adb shell getprop ro.product.cpu.abi`
2. Pushed server: `adb push frida-server /data/local/tmp/`
3. Set permissions: `adb shell chmod 755 /data/local/tmp/frida-server`
4. Started server: `adb shell "/data/local/tmp/frida-server -l 0.0.0.0"`
5. Verified visibility with `frida-ps -Uai`.

### Step 3: Root Detection Bypass
I targeted the application `com.example.rootcheck` using two different strategies to ensure a complete bypass:

**Strategy A: Spawn (Early Instrumentation)**
I launched the app directly through Objection to apply hooks before the application's main logic started:
```bash
objection -g com.example.rootcheck explore --startup-command "android root disable"
```

**Strategy B: Attach (Runtime Instrumentation)**
For scenarios where the app is already running, I attached to the process and manually disabled root detection:
```bash
objection -g com.example.rootcheck explore
# Inside Objection console:
android root disable
```

## 3. Technical Analysis
The `android root disable` command was used to neutralize the following checks:
- **System Tags:** Modified `android.os.Build.TAGS` to return "release-keys".
- **File Checks:** Intercepted `java.io.File.exists()` calls for common root binaries (e.g., `/system/xbin/su`, `busybox`).
- **Shell Execution:** Neutralized `Runtime.getRuntime().exec()` calls attempting to run `su` or `which su`.
- **Library Patches:** Applied specific patches for known libraries like RootBeer.

## 4. Validation and Results
- **Before Bypass:** The application successfully detected the rooted state of the device and blocked access to the main interface, displaying a "Root detected" warning.
- **After Bypass:** Upon executing `android root disable`, the application failed to detect the root state and allowed full access to the features, displaying a "Not rooted" status.
- **Verification:** I used `android hooking search methods isRoot` within the Objection console to confirm that the root-checking methods were being identified and intercepted.

## 5. Advanced Handling (Native Checks)
To ensure the bypass was robust against C/C++ native checks, I explored:
- **Java Bridge Hooking:** Identified Java classes acting as gateways to native code and forced a `false` return value using `android hooking set return_value`.
- **Native Tracing:** Used `frida-trace` to monitor `open`, `access`, and `stat` calls to identify which sensitive paths were being accessed by the native layer.

---
**Conclusion:** The lab was completed successfully. All root detection mechanisms (both Java-based and native) were identified and bypassed using the combination of Frida and Objection.
