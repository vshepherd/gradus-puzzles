# Gradus content hosting on GitHub Pages

This repository serves the Gradus daily-puzzle data as static JSON over GitHub
Pages. The app fetches the JSON from a fixed web address. This repo holds only the
files that need to be public; the spreadsheet, the converter, and the reserve pool
stay out of it (see "Keep private" below).

Account: `vshepherd`
Repository name used throughout: `gradus-puzzles` (change it everywhere below if you
pick a different name).

## What gets served

```
docs/
  puzzles-active.json   the live feed, one entry per day, each with an "index"
  config.json           launchDate, version, count, updated
  index.html            a small human-readable landing page
```

Once Pages is on, the files are reachable at:

- https://vshepherd.github.io/gradus-puzzles/puzzles-active.json
- https://vshepherd.github.io/gradus-puzzles/config.json
- https://vshepherd.github.io/gradus-puzzles/ (landing page)

## One-time setup

1. Sign in as `vshepherd` and create a new **public** repository named
   `gradus-puzzles`. A public repo is required for GitHub Pages on the free plan;
   with GitHub Pro you may instead use a private repo.
2. Put the contents of this scaffold (`docs/`, `README.md`, `.gitignore`) at the
   root of the repository.
3. Commit and push to the `main` branch.
4. In the repository, open **Settings > Pages**. Under **Build and deployment** set
   **Source** to *Deploy from a branch*, **Branch** to `main`, and the folder to
   `/docs`. Click **Save**.
5. Wait one to two minutes for the first publish, then open
   https://vshepherd.github.io/gradus-puzzles/puzzles-active.json to confirm it
   loads.

## Point the app at the feed

- Base URL: `https://vshepherd.github.io/gradus-puzzles/`
- On launch the app reads `config.json`, takes `launchDate`, computes
  `index = (today - launchDate)` in whole days, then reads `puzzles-active.json`
  and shows the puzzle whose `index` matches. Index 0 is the launch-day puzzle.
- `launchDate` is currently `2026-08-11`. If you launch on a different day, change
  it in `config.json` before going live.

## Updating the puzzles (routine workflow)

Work on the spreadsheet in your normal (non-public) location, then publish only the
built feed:

1. Edit and sign off puzzles in `Gradus_v2_puzzles-verified.xlsx`.
2. Rebuild the feed with the converter (approved puzzles only):

   ```
   python3 gradus_convert.py Gradus_v2_puzzles-verified.xlsx \
       --start 2026-08-11 \
       -o docs/puzzles-active.json \
       --reserve puzzles-reserve.json \
       --approved-only
   ```

   Leave off `--approved-only` only if you deliberately want to publish drafts.
3. In `docs/config.json`, increase `version` by 1 and update `count` and `updated`.
4. Commit and push just the two published files:

   ```
   git add docs/puzzles-active.json docs/config.json
   git commit -m "Update puzzles"
   git push
   ```

5. The change goes live within about ten minutes (GitHub Pages serves through a
   CDN that caches for roughly that long).

## Reusing an earlier puzzle

The 71 earlier puzzles live in `puzzles-reserve.json` (kept private). To bring one
back, copy its row into the spreadsheet with a `Publish date` on or after the launch
date, in the schedule slot you want, then rebuild. Because the feed is ordered by a
running index rather than a fixed calendar date, inserting one only shifts the
indexes that come after it.

## Keep private (do not commit to this public repo)

- `Gradus_v2_puzzles-verified.xlsx` (the single source of truth)
- `gradus_convert.py` (optional to publish; fine to keep private)
- `puzzles-reserve.json` (your unused pool)

`.gitignore` already excludes `*.xlsx` and `puzzles-reserve.json` so they will not be
added by accident if you run the converter inside the repo folder.

## Important: upcoming answers are public

Anyone can open `puzzles-active.json` and read every upcoming puzzle, including the
correct order. This is normal for daily games whose content is fetched by the client
(this is how Wordle worked). If you would rather players could not see future
answers, the feed needs to publish only the current and past days, refreshed on a
schedule. That requires a small scheduled build (for example a GitHub Actions job).
Ask if you want that added.

## Caching notes

- GitHub Pages caches for roughly ten minutes. For most daily updates that is fine.
- For faster pickup, have the app read the small `config.json` first to spot a
  `version` change, then request `puzzles-active.json?v=VERSION` so a new version
  bypasses any stale cache.
