---
name: fix-parser
description: Diagnose and fix a broken manga source parser in the sibling parsers repo. Use when a manga source stops working, returns no results, 403s, or fails to load pages or chapters.
---

# Fix a broken source

Parsers live in the **sibling clone** `../parsers` (fork of `Kotatsu-Redo/kotatsu-parsers-redo`),
never in this repo. `AGENTS.md` points at `AppFuton/futon-parsers` — that repo was archived
2026-03-26 and is wrong.

## 0. First, check whether upstream already fixed it

Cheapest possible fix. See the `bump-parsers` skill. ~136 parser commits land upstream per ~7 app
commits; someone has often already done the work. Only continue here if they haven't.

## 1. Identify the failing source

Reproduce in the app, then find the parser:

```bash
cd "/c/Users/ibkon/Desktop/CODE/PERFECT'S ECOSYSTEM/KOTATSU/parsers"
grep -rl "<SourceName>" src/main/kotlin/org/koitharu/kotatsu/parsers/site/ | head
```

Layout is `site/<country_code>/` plus engine-family dirs (`heancms`, `foolslide`, `madara`-likes).
If the source uses a shared engine, the bug may be in the family base class, affecting many sources.

## 2. Scope the test to that parser

`src/test/kotlin/org/koitharu/kotatsu/parsers/MangaSources.kt` ships as:

```kotlin
@EnumSource(MangaParserSource::class, names = [], mode = EnumSource.Mode.INCLUDE)
```

`names = []` with `INCLUDE` means **zero parsers run by default**. Add the parser's
`MangaParserSource` enum name:

```kotlin
@EnumSource(MangaParserSource::class, names = ["THE_SOURCE"], mode = EnumSource.Mode.INCLUDE)
```

## 3. Run it

```
gradlew :test --tests "org.koitharu.kotatsu.parsers.MangaParserTest"
```

These are **live-site tests** — they hit the real website. Expect flakiness and rate limiting; keep
the scope to one or two parsers. JUnit 5, not JUnit 4.

## 4. Diagnose

In likelihood order:

1. **Domain migration** — the site moved. Fix `configKeyDomain`'s default. Never hardcode a domain
   anywhere else; read the live value via `domain`.
2. **HTML restructure** — selectors no longer match. Fetch the page and compare against the parser's
   selectors.
3. **Cloudflare** — 403s. `CloudFlareInterceptor.kt` is in the test harness. Note the newer
   `MangaLoaderContext.requestCloudflareVerification` hook exists for hosts to implement.

## 5. Fix within the contract

- `@MangaSourceParser` annotation (internal name, title, language)
- Extend `MangaParser` / `PagedMangaParser` / `SinglePageMangaParser`
- Exactly one primary constructor param: `MangaLoaderContext`
- IDs via `generateUid`; `availableSortOrders` non-empty
- Prefer this repo's `util` extensions over raw JSoup

## 6. Reset and publish

**Revert `MangaSources.kt` to `names = []` before committing.** Never commit a scoped test.

Then push to the parsers fork, and follow `bump-parsers` to point the app at the new SHA.

## Context worth knowing

There is **no CI in the parsers repo** — `.github/workflows/` contains only `discord.yml`, across
1,328 site parsers. Upstream's original workflow only ever ran `compileKotlin`, never a live test.
Breakage is detected purely by user reports. A scheduled check over the ~15-20 sources actually read
would be genuinely novel; nobody in this ecosystem has one.
