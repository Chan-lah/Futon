---
name: build-install
description: Build the Futon debug APK and install it on the connected phone. Use when asked to build, run, install, deploy, or test the app on device, or to verify a change works in the real app.
---

# Build and install Futon

## 1. Build — PowerShell only, absolute paths

The repo path contains an apostrophe (`PERFECT'S ECOSYSTEM`). This breaks `.bat` invocation from
bash, and the working directory does not carry into a `cmd` child process. **A bash-driven build can
report exit 0 while gradle actually failed.**

```powershell
$env:JAVA_HOME = "C:\Program Files\Eclipse Adoptium\jdk-17.0.20.101-hotspot"
$env:ANDROID_HOME = "$env:LOCALAPPDATA\Android\Sdk"
$env:ANDROID_SDK_ROOT = $env:ANDROID_HOME
$a = "C:\Users\ibkon\Desktop\CODE\PERFECT'S ECOSYSTEM\KOTATSU\app"
& "$a\gradlew.bat" -p "$a" assembleDebug --no-daemon
```

Run it in the background — cold ≈ 45 min, incremental ≈ 14 min, trivial changes are faster.

## 2. Verify it actually built

Never trust the exit code alone. Check both:

```bash
grep -E "BUILD SUCCESSFUL|BUILD FAILED" <task output file>
ls -l ".../app/build/outputs/apk/debug/app-debug.apk"   # timestamp must be new
```

## 3. Install — Windows-style path

`adb.exe` cannot read MSYS `/c/...` paths.

```bash
ADB="/c/Users/ibkon/AppData/Local/Android/Sdk/platform-tools/adb.exe"
APK='C:\Users\ibkon\Desktop\CODE\PERFECT'"'"'S ECOSYSTEM\KOTATSU\app\app\build\outputs\apk\debug\app-debug.apk'
"$ADB" install -r "$APK"
```

## 4. Launch and confirm healthy

```bash
"$ADB" shell monkey -p io.github.landwarderer.futon.debug -c android.intent.category.LAUNCHER 1
sleep 8
"$ADB" shell pidof io.github.landwarderer.futon.debug     # alive after 8s = not crash-looping
"$ADB" logcat -d -b crash | grep -i futon                 # empty = no crashes
```

Screenshot if visual confirmation helps: `"$ADB" exec-out screencap -p > out.png`

## Notes

- Device: TECNO CK6, Android 14 / SDK 34, arm64-v8a. No emulator — 7.8 GB RAM won't take it.
- applicationId is `io.github.landwarderer.futon.debug`, so it coexists with other readers.
- `adb shell pm list packages` throws a SecurityException about "user 999" on this device. Harmless
  TECNO multi-user quirk; use `pidof` instead.
- Free RAM drops to ~1 GB during a build. Close heavy apps first.
