# CODEX TASK — Publish HaWake Verse of the Day

You are a scheduled task that keeps the **HaWake verse-of-the-day content
repo** stocked. Run this whole checklist top to bottom, verbatim, each time
you are invoked. This file is self-contained — you do not need any other doc
to run. (The human-facing rationale lives in `26-VERSE-OF-THE-DAY.md`.)

The app has **no server**: it reads static files from this repo's GitHub
Pages site. Your job is to keep `manifest.json`, `history.json`, and the
`images/` folder correct and ahead of schedule.

## Goal each run

Ensure `manifest.json` contains exactly **16 contiguous days: yesterday
through today + 14**. Yesterday is intentionally retained because the app
uses it for a spoiler-free preview; today's entry is used when the alarm
fires. Every day needs a valid image and public-domain verse, with no verse
repeating within the last **183 days**, then commit and push.

## Repo files you touch

- `manifest.json` — the live schedule (fixed schema below). READ + REWRITE.
- `history.json` — rolling recency ledger `{ "used": [ {date, reference} ] }`.
  READ + APPEND + PRUNE.
- `verse-pool.json` — curated candidates `{ reference, themes[] }`. READ only.
- `images/YYYY-MM-DD.jpg` — one portrait JPEG per day. CREATE.

## Fixed manifest schema — DO NOT DEVIATE

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

- `schemaVersion` = `1`. `updated` = today, `YYYY-MM-DD`.
- `baseImageURL` = absolute URL of `images/`, **trailing slash required**.
- `days[]` chronological, **contiguous (no gaps)**, one per date.
- `date` = bare local-calendar key `YYYY-MM-DD` — **no time, no timezone**.
- `text` = **public-domain translation only** (see hard rule).
- `translation` = `"WEB"` (preferred) or `"KJV"`.
- `image` = filename only, e.g. `"2026-07-23.jpg"`.

## HARD RULES (never break)

1. **Public-domain text only.** Use **WEB** (World English Bible; renders the
   divine name as "Yahweh" — that is correct) or **KJV**. NEVER NIV, ESV,
   NLT, NASB, CSB, NKJV, The Message, AMP, TPT, or any copyrighted
   translation. When unsure, use WEB.
2. **No verse repeats within 183 days.** Enforce via `history.json`.
3. **Images contain NO text/letters/logos/watermarks.** The app overlays the
   verse itself.
4. **Selection is curated, not random.** Draw only from `verse-pool.json`.

## Procedure

1. **Read state.**
   - Load `history.json` → set `recent` = every `reference` whose `date` is
     within the last **183 days** of today.
   - Load `verse-pool.json` (candidates).
   - Load `manifest.json` (or treat as empty if missing/blank).

2. **Compute the window.**
   - Build `targetDates` as the **16 calendar dates from yesterday through
     today + 14, inclusive**.
   - Inspect each target date and classify it:
     - `existingValid`: a manifest entry exists and its image passes all
       filename, JPEG, dimension, size, and visual checks. Preserve it
       unchanged.
     - `newDates`: no manifest entry exists. These need a new verse and image.
     - `repairDates`: an entry exists but its image is missing or invalid.
       Preserve that entry's date, verse, text, and translation; regenerate
       only its image.
   - Calculate and report:
     - `N_new = newDates.length`
     - `N_repair = repairDates.length`
     - `N_images = N_new + N_repair`
   - Generate exactly `N_images` images—no more. Add exactly `N_new` verse
     entries—no more. Never regenerate a valid published date merely to fill
     a batch.
   - A missing yesterday entry is an integrity problem: recover its original
     published entry from git history and its image if possible. Never assign
     a different verse retroactively to a past date.
   - If `N_images = 0`, still normalize the manifest to the exact 16-day
     window and run validation. Commit only if normalization changed a file;
     never create an empty commit.

3. **Pick verses** (one per date in `newDates`, and no others):
   - Choose pool entries whose `reference` is **not** in `recent` and not
     already chosen in this batch.
   - **Vary `themes`** across the batch — don't stack same-theme days.
   - Favor **morning-appropriate** verses where natural (this is a wake-up
     alarm): e.g. Psalm 118:24, Lamentations 3:22-23, Psalm 143:8, Psalm 30:5,
     Psalm 90:14, Psalm 100:4-5.

4. **Fetch + verify text.** For each chosen reference, get the **WEB** (or
   KJV) text from a public-domain source. Verify it is genuinely WEB/KJV
   wording (not a copyrighted translation). Do not paraphrase or modernize.
   Record exact `text` + `translation`.

5. **Generate exactly `N_images` images.** For `newDates`, create one image
   for the newly selected verse. For `repairDates`, create one replacement
   image for the existing paired verse. Every output is a portrait **JPEG,
   1080×1920**, compressed to **≤ 400 KB**, saved as
   `images/YYYY-MM-DD.jpg`.

   Choose each visual concept from the paired verse itself: its words,
   imagery, emotional tone, and `themes`. The image must clearly relate to
   that specific verse and leave the app's text readable, but otherwise use
   real stylistic freedom. Vary subject matter, lighting, palette, visual
   metaphor, and medium where it serves the verse. Photographic, painterly,
   illustrative, atmospheric, and restrained abstract scenes are all valid.
   Avoid making every verse another version of the same dawn landscape.

   Prompt template:
   ```
   A polished portrait 9:16 image meaningfully inspired by
   {REFERENCE} and its themes {THEMES}: {VERSE_SPECIFIC_CONCEPT}.
   Artistic direction: choose the style, medium, palette, lighting, and
   visual metaphor that best express this particular verse. Be creative
   and distinctive while remaining reverent and avoiding religious kitsch.
   Composition: reserve a calm, low-contrast, uncluttered area across the
   LOWER THIRD (negative space, soft depth of field, sky, water, mist, or
   another scene-appropriate treatment) so the app's verse overlay remains
   easy to read.
   Absolutely NO text, letters, words, watermarks, logos, or people's faces.
   High quality, intentional, timeless.
   ```
   Optional theme inspiration — **not a required theme → scene mapping**:
   | Theme | MOOD | SCENE | PALETTE |
   |-------|------|-------|---------|
   | strength/courage | resolute, calm | dawn over a mountain ridge | warm gold + deep blue |
   | peace/rest | serene | still lake at first light, soft mist | soft teal + cream |
   | hope/future | hopeful, bright | sunrise through clouds, open path | amber + rose |
   | comfort/healing | tender | gentle rain on leaves, warm light | muted green + warm grey |
   | gratitude/joy | luminous, glad | sunlit field, golden hour | golden yellow + green |
   | morning | fresh, awakening | first light over hills, dew | pale gold + light blue |
   | love | warm | soft glowing sheltering sky | rose + warm amber |
   | trust/faith | steady | calm sea to horizon | deep blue + silver |
   | protection/refuge | secure | sheltering trees, safe harbor | forest green + slate |

6. **Validate** (fix any failure before writing):
   - [ ] `schemaVersion` == 1; `updated` == today.
   - [ ] `baseImageURL` absolute, ends with `/`.
   - [ ] `days` has exactly **16** entries, chronological and contiguous,
         covering **yesterday through today + 14**.
   - [ ] Yesterday exists for the intentional spoiler-free preview; today
         exists for the alarm.
   - [ ] Every `date` a bare `YYYY-MM-DD`.
   - [ ] Every `translation` is `WEB` or `KJV`; text matches reference+translation.
   - [ ] No `reference` in the new days used within last 183 days; none repeats in-batch.
   - [ ] Every `image` file exists, name matches its `date`, JPEG 1080×1920, ≤ 400 KB.
   - [ ] No baked-in text in any image; lower third calm.
   - [ ] `manifest.json` and `history.json` are valid JSON and parse.

7. **Write output.**
   - Save all images into `images/`.
   - Rewrite `manifest.json`: bump `updated` to today, merge old + new days,
     preserve valid target entries, repair only invalid images, then retain
     exactly the 16 sorted contiguous target dates from yesterday through
     today + 14. Remove older and farther-future entries from `days[]`; image
     files may remain in the repository for caching/history.
   - Append one `{ "date", "reference" }` to `history.json` `used[]` for every
     date in `newDates` only. Do not append duplicates for `existingValid` or
     `repairDates`. **Prune** entries older than ~400 days.

8. **Commit & push.**
   - `git add manifest.json history.json images/`
   - `git commit -m "Verses: publish through <last date>"`
   - `git push`  (GitHub Pages redeploys automatically.)

## If something is wrong

- Pool exhausted by the recency filter (too few eligible verses): widen the
  pool by editing `verse-pool.json` first, or shorten the batch — never
  silently reuse a recent verse or use a copyrighted translation to fill a gap.
- Can't verify a translation is public domain: skip that reference and pick
  another. A short-but-correct manifest beats a licensing violation.
- Image over 400 KB: recompress (lower JPEG quality / re-render), do not ship
  it oversized.

## Why ahead-of-schedule matters (app context)

The app intentionally uses yesterday's verse for a spoiler-free preview, then
shows today's verse after **every alarm the user has enabled it on** (no
once-per-day cap; the same verse serves the whole local day). The 16-day
window therefore starts at yesterday and extends through today + 14. If the
manifest ever loses yesterday, falls behind today, or has a gap, users hit
the bundled offline fallback instead of the intended verse.
