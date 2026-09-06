---
name: sync-upstream
description: Rebase the perfect branch onto upstream to pick up fork updates. Use when asked to sync, update from upstream, pull in fork changes, or resolve drift from AppFuton/Futon.
---

# Sync with upstream

Futon has granular, rebaseable history — a main reason it was chosen over Kotatsu-Redo, whose
releases are single squashed commits. Don't squander that by drifting far.

## App repo

```bash
cd "/c/Users/ibkon/Desktop/CODE/PERFECT'S ECOSYSTEM/KOTATSU/app"
git fetch upstream
git log --oneline HEAD..upstream/devel        # what's new
git rebase upstream/devel                     # from the perfect branch
```

## Parsers repo

Same, against `upstream/master`.

## Rules

- Always rebase **from `perfect`**. Never commit to the tracking branch — keep it clean so it
  fast-forwards.
- Our commits that will always replay: `CLAUDE.md` in both repos, the removal of
  `.claude/settings.local.json` in parsers, `.claude/skills/` in the app.
- The parsers pin bump may conflict if upstream also bumped it. Take **whichever SHA is newer**,
  then re-run the drift check from the `bump-parsers` skill.
- Rebuild and install after any rebase before trusting it.

## Note

`~/.gradle/gradle.properties` holds the memory override at user level precisely so rebases never
touch it. If a build suddenly OOMs after a sync, check that file still exists rather than assuming
the rebase broke something.
