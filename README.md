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

Open [`signal.html`](signal.html) in a modern browser. There is no package
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

## Known engine issues — description only

These issues are intentionally documented rather than fixed in this baseline:

1. **Workshop progress does not accumulate during live ticks.**
   `Workshop.advance()` receives roughly one second at a time and rounds down
   to completed recipe units. The unused fraction is discarded, so recipes
   requiring several seconds only complete when one call supplies the full
   duration, such as during offline catch-up or a development time skip.
2. **Imported saves lose their offline interval.** The restore flow writes the
   imported state before calculating catch-up. That replaces `lastSaved` with
   the current time, leaving no elapsed interval to process.
3. **A multi-discovery expedition can roll the same unique more than once.**
   All pending discoveries are rolled before any are claimed into state, so
   later rolls cannot see unique nodes selected earlier in the same batch.
4. **Duration-specific expedition rules are incomplete.** The authoritative
   5-minute extractor weighting, 2-hour duplicate reroll, and 8-hour guaranteed
   ruin-or-anomaly behavior are not implemented yet.
5. **Schema-version handling is permissive.** Unknown save versions currently
   pass through migration unchanged rather than being migrated or rejected.

## Save-schema decisions required

The prototype persists fields not listed in the authoritative state schema:

- `lastBackup` tracks when the player last downloaded a backup.
- `features` stores feature and expedition-duration unlock flags.
- Expedition-slot `pending` stores discoveries awaiting player review.

Before the save format is treated as stable, explicitly approve these fields
and add them to the schema, or redesign the implementation to derive or store
the same information using approved fields. Any approved format change must
include an appropriate `schemaVersion` decision and migration policy.

## Baseline verification

Until automated tests are approved, baseline checks are manual:

- Validate the embedded JavaScript with `node --check` after extracting the
  script block.
- Verify fresh-load, save/reload, export/import, offline progress, expedition
  resolution, and mobile Safari layout manually.
- Review `git diff --check` before committing.
