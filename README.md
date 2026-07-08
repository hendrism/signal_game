# SIGNAL

SIGNAL is a text-based exploration and logistics idle game for iPhone and
iPad. It currently runs as a dependency-free, single-file web app intended to
be added to the home screen.

## Current status

The repository contains an early prototype spanning the Stage 1 scaffold and
parts of the Stage 2 gameplay loop. The prototype is useful for validating the
direction, but it is not yet a complete or tested MVP.

## Source-of-truth documents

- [`plans/signal-design-and-roadmap.md`](plans/signal-design-and-roadmap.md) —
  authoritative game design, schemas, balance, roadmap, and agent protocol.
- [`signal-story-bible.md`](signal-story-bible.md) — narrative continuity and
  tone rules for content work.
- [`plans/signal-2.0-ideas.md`](plans/signal-2.0-ideas.md) — parked ideas that
  must not be implemented before 1.0.

When documents conflict, the detailed specifications in sections 9–12 of the
design and roadmap take precedence.

## Running the prototype

Play the hosted build at <https://hendrism.github.io/signal_game/> (deployed
from `main` by `.github/workflows/deploy-pages.yml`), or open
[`signal.html`](signal.html) directly in a modern browser. There is no package
manager, build step, server, or external dependency.

For the intended target, test through iPhone and iPad Safari as well as the
home-screen launch flow. Save data is stored in browser `localStorage`; use the
Logs tab to export a backup before destructive testing.

## Architecture constraints

- Keep static content in `GAME_DATA` and saved progress in the state object.
- Keep simulation rules separate from rendering and UI behavior.
- Use seeded randomness for all gameplay rolls.
- Keep the project dependency-free and single-file through Stage 2.
- At Stage 3, split only into `index.html`, `engine.js`, and `data.js` unless a
  different file plan is explicitly approved.
- Obtain approval before changing schemas, balance numbers, mechanics,
  screens, dependencies, files, build steps, or save format.

## Engine fixes incorporated

The prototype now includes the following foundational corrections:

1. **Workshop accumulation:** Jobs retain fractional live-tick progress and
   spill excess elapsed time into the next queued job.
2. **Import catch-up:** Offline catch-up runs before the imported `lastSaved`
   timestamp is replaced.
3. **Batch uniqueness:** A per-batch reservation set prevents unique nodes from
   being selected more than once in a multi-discovery expedition.
4. **Duration rules:** The 5-minute extractor weighting, 2-hour duplicate
   reroll, and 8-hour guaranteed ruin-or-anomaly behavior are implemented.
5. **Strict migration:** Invalid and newer schema versions are rejected while
   autosave is paused, preventing an unreadable save from being overwritten.

## Approved save-schema extensions

The authoritative state schema now includes these approved prototype fields:

- `lastBackup` tracks when the player last downloaded a backup.
- `features` stores feature and expedition-duration unlock flags.
- Expedition-slot `pending` stores discoveries awaiting player review.

Workshop jobs also persist `progress`, the accumulated seconds toward the
current unit. The roadmap defines the strict rejection and stepwise migration
policy for schema versions.

## Baseline verification

Until automated tests are approved, baseline checks are manual:

- Validate the embedded JavaScript with `node --check` after extracting the
  script block.
- Verify fresh-load, save/reload, export/import, offline progress, expedition
  resolution, and mobile Safari layout manually.
- Review `git diff --check` before committing.
