# 26 — Verse of the Day

## Overview

Verse of the Day (VOTD) shows one encouraging Scripture verse per calendar
day over a full-bleed portrait image. There is **no server**: the iOS app
fetches a small set of static files from a **GitHub Pages** content repo,
matches today's local date against a manifest, and renders the verse text as
an overlay on the day's image. Content is produced ahead of time and
published in batches.

Generation is **not** a GitHub Action. It is a **scheduled Codex task** that
the user runs (see `verse-repo-starter/CODEX-TASK.md`). This document is that
task's operating manual — repo layout, image and licensing rules, verse
selection and recency policy, the step-by-step procedure, and the app-side
behavior the content must serve.

The app is being built concurrently against the **fixed manifest schema**
below. Do not deviate from it — no renamed keys, no added required fields, no
date-format changes. Additive-only, behind a bumped `schemaVersion`.

---

## Repo layout (GitHub Pages)

The content lives in its own repo published via GitHub Pages (e.g.
`hawake-verses`). The app reads from the Pages URL, not the GitHub API.

```
/ (Pages root)
├── manifest.json        # the live schedule the app fetches (see schema)
├── history.json         # rolling ledger of published verses (recency guard)
├── verse-pool.json      # curated candidate pool (references + themes)
└── images/
    ├── 2026-07-23.jpg
    ├── 2026-07-24.jpg
    └── …                # one portrait JPEG per published day
```

- `manifest.json` — the only file the app strictly requires. Always covers
  **today + at least 7 days ahead**; recommended publish depth is **14 days**
  so a missed run still leaves a healthy buffer.
- `history.json` — the generator's memory. Read at the start of every run,
  appended at the end. The app does not read it.
- `verse-pool.json` — the curated candidate list the generator draws from.
  Editable by hand to grow/tune the canon; the app does not read it.
- `images/` — one file per day, named exactly `YYYY-MM-DD.jpg` to match the
  manifest's `image` field. Served under `baseImageURL`.

`baseImageURL` in the manifest is the Pages URL of `images/` with a trailing
slash; the app resolves each day's image as `baseImageURL + image`. Keeping
it explicit lets the images move (CDN, another Pages repo) without an app
update.

---

## Fixed manifest schema

`schemaVersion` is `1`. Document and produce **exactly** this shape:

```json
{
  "schemaVersion": 1,
  "updated": "2026-07-22",
  "baseImageURL": "https://EXAMPLE.github.io/hawake-verses/images/",
  "days": [
    { "date": "2026-07-23", "reference": "Philippians 4:13",
      "text": "I can do all things through Christ, who strengthens me.",
      "translation": "WEB", "image": "2026-07-23.jpg" }
  ]
}
```

| Field | Type | Notes |
|-------|------|-------|
| `schemaVersion` | int | Always `1` for this schema. Bump only for a breaking change. |
| `updated` | string | `YYYY-MM-DD`, the date this manifest was regenerated. |
| `baseImageURL` | string | Absolute URL of `images/`, **trailing slash required**. |
| `days` | array | Chronological, contiguous, one object per day. |
| `days[].date` | string | `YYYY-MM-DD`, a **local-calendar key** (see below). |
| `days[].reference` | string | Canonical citation, e.g. `Philippians 4:13`, `Proverbs 3:5-6`. |
| `days[].text` | string | The verse text, **public-domain translation only**. |
| `days[].translation` | string | Translation code, `WEB` (preferred) or `KJV`. |
| `days[].image` | string | Filename only (e.g. `2026-07-23.jpg`), resolved against `baseImageURL`. |

**Dates are local-calendar keys.** `date` is a plain `YYYY-MM-DD` string with
no timezone. The app forms *its own* local date the same way and looks for an
exact string match — so a user in Auckland and a user in Los Angeles see the
same verse on their respective local July 23rd. Never emit timezone offsets,
times, or `Z` suffixes. `days` must be **contiguous** (no gaps) and sorted
ascending starting at today (or earlier is fine, but today forward is what
matters).

---

## Image requirements

- **Format / size:** portrait **JPEG**, **1080×1920** (9:16). Target
  **≤ 400 KB** each. The app runs on a memory budget and downsamples at
  display time, so oversized files waste bandwidth and cache without looking
  better. Quality ~80 JPEG almost always lands under budget at this
  resolution.
- **Imagery ONLY — no text baked in.** The app overlays the verse text and
  reference itself. Any baked-in words fight the overlay and can't adapt to
  font/theme/localization. No watermarks, no logos, no captions.
- **Leave the lower third calm.** The overlay sits in the lower portion of
  the frame. Keep the bottom third visually quiet — soft gradients, sky,
  water, blur, negative space — so light overlay text stays legible. Avoid
  busy detail, hard edges, or bright highlights there.
- **Style:** tasteful, reverent, **abstract or scenic** — sunrises, mountains,
  calm water, fields, soft light, clouds, gentle bokeh. No people's faces as
  the subject, no on-the-nose religious kitsch, nothing that dates quickly.
- **Consistent visual identity across a run is encouraged.** A week (or a
  full 14-day batch) that shares a palette / mood / treatment reads as a
  designed set rather than stock. Pick a direction per batch.

---

## Translation licensing (HARD RULE)

Verse text MUST be **public domain**. There are no exceptions and no
"just this once."

- **Preferred:** World English Bible (**WEB**) — modern, readable, public
  domain. Note WEB renders the divine name as "Yahweh"; that is correct WEB
  and expected.
- **Acceptable:** King James Version (**KJV**) — public domain.
- **NEVER** use copyrighted translations: **NIV, ESV, NLT, NASB, CSB, NKJV,
  The Message (MSG), AMP, TPT**, or any other in-copyright text. Quoting them
  in a published product without a license is infringement.

Every `days[].text` must come from a WEB or KJV source and `translation` must
name which. When in doubt, use WEB. Verify the text against a public-domain
source at generation time (see the procedure) — do not paraphrase, "improve,"
or modernize the wording.

---

## Verse selection policy

Selection is **not random**. Draw from the curated **`verse-pool.json`** — the
well-loved encouragement / comfort / strength canon that popular VOTD apps
feature (Philippians 4:13, Jeremiah 29:11, Psalm 118:24, Isaiah 41:10,
Proverbs 3:5-6, John 3:16, Romans 8:28, Lamentations 3:22-23, Zephaniah 3:17,
Matthew 11:28, and onward across encouragement, peace, strength, hope,
gratitude, and morning-specific verses).

Each pool entry carries **`themes`** tags. Use them to:

- drive the image prompt (theme → mood/imagery, see the prompt template);
- keep a batch varied — avoid stacking several "strength" days in a row; mix
  themes across the run;
- optionally favor **morning-appropriate** verses (Psalm 118:24, Lamentations
  3:22-23, Psalm 143:8, Psalm 30:5) — this is a wake-up alarm, and a morning
  note lands well.

Grow the pool by editing `verse-pool.json`; references only, no copyrighted
text in it.

---

## Recency rule

**A verse must not repeat within 6 months (183 days).**

`history.json` is the rolling ledger. Its shape (JSON has no comments, so the
shape is documented here, not in the file):

```json
{ "used": [ { "date": "2026-07-23", "reference": "Philippians 4:13" } ] }
```

- `used` is an array of `{ "date", "reference" }`, one per **published** day.
- The generator **MUST** read `history.json` first and **exclude any
  `reference` used in the last 183 days** from selection.
- After publishing, **append** one entry per newly published day.
- **Prune** entries older than **~13 months (≈ 400 days)** so the ledger
  stays bounded. (13 months, not 6, gives margin around the 6-month window.)

Matching is by exact `reference` string, so keep citation formatting
canonical and consistent (e.g. always `Proverbs 3:5-6`, never
`Proverbs 3:5-6 ` or `Prov 3:5-6`).

---

## The Codex task procedure

This is the operating loop; `verse-repo-starter/CODEX-TASK.md` restates it as
a self-contained prompt/checklist the scheduled task follows verbatim.

1. **Read state.** Load `history.json` (`used[]`) and `verse-pool.json`.
   Compute the set of references used within the last **183 days**.
2. **Determine the window.** Read `manifest.json`. Find the last covered
   `date`; the new days to add run from there up to **today + 13** (a rolling
   14-day horizon). If the manifest is empty/missing, start at today. Let
   **N** = number of days to fill.
3. **Pick verses.** For each of the N days, choose a pool entry that is **not**
   in the recency exclusion set, varying `themes` across the batch and
   favoring morning-appropriate verses where natural. No reference repeats
   within the batch.
4. **Fetch text.** For each chosen reference, get the **WEB** (or KJV) text
   from a public-domain source and **verify the translation** — confirm it is
   genuinely WEB/KJV wording, not a copyrighted translation. Store exact
   `text` + `translation`.
5. **Generate images.** One portrait image per verse using the themed prompt
   template below. Reiterate: **no text in the image**, lower third calm,
   1080×1920, reverent/scenic. Export JPEG, compress to **≤ 400 KB**, name
   `YYYY-MM-DD.jpg`.
6. **Validate** against the checklist below. Fix any failure before writing.
7. **Write output.** Save images into `images/`; write the merged, updated
   `manifest.json` (bump `updated`, keep `days` contiguous and covering
   today + ≥7); **append** the new days to `history.json` and **prune**
   entries older than ~400 days.
8. **Commit & push** to the content repo so GitHub Pages republishes.

### Reusable image prompt template

Fill the slots from the verse's `themes`. Keep the fixed composition/no-text
rules verbatim.

```
A reverent, {MOOD} portrait 9:16 image evoking {THEME}: {SCENE}.
Style: {STYLE} — soft natural light, {PALETTE} palette, gentle and
uplifting, abstract/scenic, no religious kitsch.
Composition: keep the LOWER THIRD visually calm and uncluttered (soft
gradient / sky / water / negative space) for legible text overlay.
Absolutely NO text, letters, words, watermarks, logos, or people's faces.
Photographic/painterly, high quality, timeless.
```

Suggested slot values by theme:

| Theme | MOOD | SCENE | PALETTE |
|-------|------|-------|---------|
| strength / courage | resolute, calm | dawn over a mountain ridge, steady horizon | warm gold + deep blue |
| peace / rest | serene | still lake at first light, soft mist | soft teal + cream |
| hope / future | hopeful, bright | sunrise breaking through clouds, open path | amber + rose |
| comfort / healing | tender | gentle rain on leaves, warm window light | muted green + warm grey |
| gratitude / joy | luminous, glad | sunlit field, golden hour, wildflowers | golden yellow + green |
| morning | fresh, awakening | first light over hills, dew, quiet dawn | pale gold + light blue |
| love | warm | soft glowing light, sheltering sky | rose + warm amber |
| trust / faith | steady | calm sea to horizon, anchored stillness | deep blue + silver |
| protection / refuge | secure | sheltering trees, mountain shadow, safe harbor | forest green + slate |

### Validation checklist

- [ ] `schemaVersion` is `1`; `updated` is today (`YYYY-MM-DD`).
- [ ] `baseImageURL` is absolute and ends with `/`.
- [ ] `days` is chronological, **contiguous** (no missing dates), and covers
      **today + at least 7** (aim for 14).
- [ ] Every `date` is a bare `YYYY-MM-DD` (no time/zone).
- [ ] Every `translation` is `WEB` or `KJV` — **no copyrighted translation**.
- [ ] Every `text` matches its `reference` and the named translation exactly.
- [ ] **No recency violation:** no `reference` in the new days was used in the
      last 183 days (per `history.json`), and none repeats within this batch.
- [ ] Every `image` file exists in `images/`, name matches the `date`, is
      **portrait JPEG 1080×1920**, and is **≤ 400 KB**.
- [ ] No image contains baked-in text; lower third is calm.
- [ ] `history.json` appended for every new day; entries older than ~400 days
      pruned; JSON is valid (`{ "used": [ … ] }`).
- [ ] `manifest.json` is valid JSON and parses.

---

## App behavior (context)

The content must serve these app behaviors — publishing gaps degrade
gracefully but should be avoided:

- **7-day offline cache.** The app caches the manifest and upcoming images so
  it works offline for about a week. This is why the manifest publishes ahead
  (today + ≥7, ideally 14) — a user who goes offline still has verses queued.
- **Per-alarm enablement (no once-per-day rule).** There is no once-a-day
  display cap. The verse shows after **every alarm the user enables it on** —
  frequency is entirely the user's choice via per-alarm enablement. All of a
  given local date's showings use the **same verse** (the manifest's `days[]`
  entry matching today's local `YYYY-MM-DD`), so a user with three morning
  alarms sees that day's one verse three times, not three different verses.
- **Bundled fallback.** If the app is offline with a cold/exhausted cache and
  no matching day, it shows a bundled fallback verse rather than nothing.

Consequences for the generator: still one verse per local date (a day maps to
exactly one `days[]` entry no matter how many times it displays); never let
`manifest.json` fall behind today; keep `days` contiguous so no local date
misses; keep images small so the 7-day prefetch fits the cache budget.

---

## Key Files (content repo)

| File | Role |
|------|------|
| `manifest.json` | Live schedule the app fetches; fixed schema, today + ≥7 days. |
| `history.json` | Rolling `{ "used": [...] }` recency ledger; read+append each run. |
| `verse-pool.json` | Curated candidate pool (`reference` + `themes`); references only. |
| `images/YYYY-MM-DD.jpg` | One portrait JPEG per day, ≤ 400 KB, no baked-in text. |
| `CODEX-TASK.md` | Self-contained instructions the scheduled Codex task runs verbatim. |

Starter versions of these live in `Documents/verse-repo-starter/` (this
repo). Copy them into the content repo to bootstrap it.
