# CLAUDE.md — Futon fork (personal)

Personal fork of **AppFuton/Futon**, itself a fork of the now-archived Kotatsu.
Goals: a **working daily driver** + the ability to **fix/add manga sources**. Not features, not iOS.

Read `AGENTS.md` for code style, Hilt/coroutine conventions and test patterns — it is good and not duplicated here.
This file records what `AGENTS.md` gets **wrong** or omits.

---

## ⚠️ Corrections to AGENTS.md

`AGENTS.md` says manga sources go in `AppFuton/futon-parsers`. **That repo was archived 2026-03-26.**
The real dependency (see `app/build.gradle`) is:

```
implementation("com.github.clquwu:kotatsu-parsers-redo:$parsersVersion")
```

Parser work happens in the **sibling clone `../parsers`** (fork of `Kotatsu-Redo/kotatsu-parsers-redo`), never here.

`AGENTS.md` also claims "Gradle with Kotlin DSL" — the build files are **Groovy** (`build.gradle`, not `.kts`).

---

## Repo layout on this machine

```
KOTATSU/
  app/      <- this repo   (fork of AppFuton/Futon)
  parsers/  <- sibling     (fork of Kotatsu-Redo/kotatsu-parsers-redo)
```

Branch: work on **`perfect`**. Keep `devel` clean so it fast-forwards from `upstream`.
Remotes: `upstream` = AppFuton/Futon. (`KotatsuApp` remote points at the archived original; harmless, added by VS Code.)

---

## Building

**Always invoke gradlew from PowerShell with absolute paths.** The repo path contains an
apostrophe (`PERFECT'S ECOSYSTEM`) which breaks `.bat` invocation from bash, and the working
directory does not carry into a `cmd` child process.

```powershell
$env:JAVA_HOME = "C:\Program Files\Eclipse Adoptium\jdk-17.0.20.101-hotspot"
$env:ANDROID_HOME = "$env:LOCALAPPDATA\Android\Sdk"
$a = "C:\Users\ibkon\Desktop\CODE\PERFECT'S ECOSYSTEM\KOTATSU\app"
& "$a\gradlew.bat" -p "$a" assembleDebug --no-daemon
```

A bash-side failure here can report **exit 0 while gradle actually failed** — always verify the
APK timestamp changed, don't trust the exit code.

Install: `adb install -r` with a **Windows-style** path (adb.exe cannot read `/c/...` MSYS paths).

Cold build ≈ 45 min on this machine. Output: `app/build/outputs/apk/debug/app-debug.apk` (~42 MB),
applicationId `io.github.landwarderer.futon.debug` — installs alongside other readers.

### 🔴 Memory

This project commits `org.gradle.jvmargs=-Xmx8192M` plus an 8 GB Kotlin daemon.
**This machine has 7.8 GB total.** The override lives in `~/.gradle/gradle.properties`
(user level, so it survives rebases and never dirties the fork). Never remove it; never raise it.
Free RAM bottoms out near 1 GB during a build — close heavy apps first.

An emulator is not viable here. Deploy to the physical phone (TECNO CK6, Android 14 / SDK 34).

---

## Bumping the parsers pin

The single highest-value maintenance action — upstream parser repair is crowdsourced, so most
source breakage is fixed by taking their work, not writing our own.

1. Get the SHA: in `../parsers`, `git rev-parse HEAD | cut -c1-10`
2. Edit `gradle/libs.versions.toml` → `parsers = "<sha>"`
3. Rebuild and smoke-test.

`app/build.gradle` also supports `-DparsersVersionOverride=<sha>` for testing without editing.

**Before bumping, check API drift** — diff the old pin against the new one in `../parsers` and look
at anything outside `parsers/site/`, especially `MangaLoaderContext.kt` (the interface this app
implements in `core/parser/MangaLoaderContextImpl.kt`).

---

## Build facts

compileSdk 36 · minSdk 23 · targetSdk 36 · Java 17 · Kotlin 2.2.10 · AGP 9.1.1 · Gradle wrapper 9.3.1
Hilt/Dagger · Room · Coil · OkHttp · Moshi · viewBinding **and** Compose both enabled (mid-migration)

Harmless warning: "SDK XML version 4 encountered, understands up to 3" — the standalone
cmdline-tools are newer than AGP 9.1.1 expects.

Telemetry: `SENTRY_DSN` is empty in a plain clone, so crash reporting is **off by default**.
Verified on-device. Keep it that way.

---

## Verification bar

Test coverage is thin and most of the app is UI. Realistic order:

1. Unit test where logic is genuinely testable — `gradlew test`
2. Room migration tests for schema changes (`androidx.room.testing` is wired in)
3. Otherwise **manual on-device repro**, with exact steps in the commit message

After each merged change, exercise: open library → browse a source → read a chapter →
download a chapter → resume reading.

One item per commit.
