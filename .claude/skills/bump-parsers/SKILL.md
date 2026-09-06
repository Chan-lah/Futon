---
name: bump-parsers
description: Update the pinned kotatsu-parsers-redo SHA to pick up upstream source fixes, checking API drift first. Use when manga sources are broken or stale, or when asked to update parsers or refresh sources.
---

# Bump the parsers pin

Upstream parser repair is crowdsourced by ~15 recurring contributors. **For most broken sources the
correct action is to take their fix, not write your own.** This is the highest value-to-effort
maintenance action in the project.

## 1. Get the candidate SHA

```bash
cd "/c/Users/ibkon/Desktop/CODE/PERFECT'S ECOSYSTEM/KOTATSU/parsers"
git fetch upstream && git log --oneline -5 upstream/master
git rev-parse upstream/master | cut -c1-10      # JitPack uses 10 chars
```

## 2. Check API drift BEFORE bumping — this is the whole risk

```bash
OLD=<current pin from gradle/libs.versions.toml>
NEW=<candidate>
git diff --name-only $OLD..$NEW | grep -v "parsers/site/"
```

Anything under `parsers/site/` is just source repair — ignore it, that's what you want.
**Only files outside `site/` can break the build.** The one that matters most is
`MangaLoaderContext.kt`, the interface the app implements in
`core/parser/MangaLoaderContextImpl.kt`.

Read any such diff. Additive `open` methods with default bodies are safe. New `abstract` members,
signature changes, or removals require a matching change in `MangaLoaderContextImpl`.

## 3. Apply

Edit `gradle/libs.versions.toml`:

```
parsers = "<new-sha>"
```

To trial without editing: `-DparsersVersionOverride=<sha>` on the gradle command line.

## 4. Verify

Rebuild and install (see the `build-install` skill). Confirm:
- Gradle resolved the new artifact — it appears under
  `~/.gradle/caches/modules-2/files-2.1/com.github.clquwu/kotatsu-parsers-redo/`
- The APK timestamp changed
- The app launches and a source you actually read still loads

JitPack builds **on demand**. The first request for a new SHA triggers a remote build that can be
slow or time out — retry before debugging locally.

## 5. Commit

One commit, recording the drift check result and what was verified.
