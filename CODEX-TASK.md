# CODEX TASK — Publish HaWake Verse of the Day

You are a scheduled task that keeps the **HaWake verse-of-the-day content
repo** stocked. Run this whole checklist top to bottom, verbatim, each time
you are invoked. This file is self-contained — you do not need any other doc
to run. (The human-facing rationale lives in `26-VERSE-OF-THE-DAY.md`.)

The app has **no server**: it reads static files from this repo's GitHub
Pages site. Your job is to keep `manifest.json`, `history.json`, and the
`images/` folder correct and ahead of schedule.

## Goal each run

Ensure `manifest.json` covers **today through today + 13** (a rolling 14-day
horizon), with a valid image and public-domain verse for every day, no verse
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
   - Target coverage = **today … today + 13**.
   - The days to (re)fill = every date in that window not already present in
     `manifest.days` with a valid image. Let **N** = count. If N = 0, the
     manifest is already current — still run validation (step 6) and stop
     without an empty commit.

3. **Pick verses** (one per new day):
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

5. **Generate one image per verse.** Portrait **JPEG, 1080×1920**,
   compressed to **≤ 400 KB**, saved as `images/YYYY-MM-DD.jpg`. Use the
   prompt template, filling slots from the verse's `themes`. Keep a
   **consistent visual identity across the batch** (shared palette/mood).

   Prompt template:
   ```
   A reverent, {MOOD} portrait 9:16 image evoking {THEME}: {SCENE}.
   Style: {STYLE} — soft natural light, {PALETTE} palette, gentle and
   uplifting, abstract/scenic, no religious kitsch.
   Composition: keep the LOWER THIRD visually calm and uncluttered (soft
   gradient / sky / water / negative space) for legible text overlay.
   Absolutely NO text, letters, words, watermarks, logos, or people's faces.
   Photographic/painterly, high quality, timeless.
   ```
   Theme → slot cheat sheet:
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
   - [ ] `days` chronological, **contiguous**, covers **today + ≥7** (aim 14).
   - [ ] Every `date` a bare `YYYY-MM-DD`.
   - [ ] Every `translation` is `WEB` or `KJV`; text matches reference+translation.
   - [ ] No `reference` in the new days used within last 183 days; none repeats in-batch.
   - [ ] Every `image` file exists, name matches its `date`, JPEG 1080×1920, ≤ 400 KB.
   - [ ] No baked-in text in any image; lower third calm.
   - [ ] `manifest.json` and `history.json` are valid JSON and parse.

7. **Write output.**
   - Save all images into `images/`.
   - Rewrite `manifest.json`: bump `updated` to today, merge old + new days,
     keep them sorted/contiguous, ensure today + ≥7 coverage.
   - Append one `{ "date", "reference" }` to `history.json` `used[]` for every
     newly published day. **Prune** entries older than ~400 days.

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

The app caches ~7 days offline and shows the day's verse after **every alarm
the user has enabled it on** (no once-per-day cap; the same verse serves the
whole local day). If the manifest ever falls behind today or has a gap, users
hit the bundled offline fallback instead of the intended verse. Always leave
the horizon full.
