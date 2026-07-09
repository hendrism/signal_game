# SIGNAL — project instructions for Claude Code

Single-file HTML idle/exploration game for iOS Safari (home-screen web app). Personal project. This repo's source of truth is `signal-design-and-roadmap.md` — read it before any task. `signal-story-bible.md` is for content-writing tasks only. `signal-2.0-ideas.md` is where deferred ideas go.

## Hard rules
- Follow §12 of the design doc (Agent Decision Protocol) exactly. Ask Sean before: schema changes, changing any §10 number, new mechanics/screens/settings, dependencies, files, build steps, or save-format changes. When the doc is silent, pick the simplest option and flag it as "DECISION MADE — review me."
- No frameworks, no build step, no npm. One file (`signal.html`) until the Stage 3 split into `index.html`/`engine.js`/`data.js`.
- Protect the GAME_DATA/engine boundary. Content is data; the engine never hardcodes content ids.
- Balance numbers live in §10 of the doc. Implement them; never invent or "fix" them.
- New feature ideas: append to `signal-2.0-ideas.md`, do not implement.

## Working practices
- Targeted edits over rewrites. Never regenerate the whole file to change one function.
- After any JS change: extract the script block and run `node --check`. For engine-math changes (production, workshop, expeditions, offline, saves), write a small smoke test with DOM stubs and run it before committing.
- Live tick and offline catch-up share the same functions (`productionGains`, `Workshop.advance`). Never fork that logic.
- Commit per task with plain descriptive messages. Never commit a broken syntax check.
- The `EXT:` comments mark planned extension points — build there, not around there.

## Current status (update this section as stages complete)
- Stage 1 + core loop engine: done, tested (production, bandwidth, expeditions with duration rules, pity, workshop accumulation, offline catch-up, strict save migration with overwrite protection).
- 2h/8h expedition durations wired to tech unlocks (`t2_extended_survey`, `t2_deep_sweep`; fragCost 0 — DECISION under review, see §12 flag in commit history).
- GitHub Pages live at https://hendrism.github.io/signal_game/ (workflow build via `.github/workflows/deploy-pages.yml`; Pages build_type must stay "workflow", not "legacy").
- Tier 1 content integrated (12 nodes, 8 logs, 3 choice events; validated + engine-simulated). Stage 2 build tasks are complete.
- Early-game agency fix (Sean-approved, doc §2.7): manual sweep (+5 scrap/60s, `GAME_DATA.manualSweep`) and fresh saves seed `startDiscovered` nodes; `state.lastSweep` added via migration default.
- Roadmap position: end of Stage 2 — Sean plays the MVP for 2–3 days, then the Fable/Opus gate audit (bring play notes + the flagged fragCost/linkCost decisions).
- PROPOSAL pending Sean's markup: `plans/signal-production-redesign-proposal.md` (rate-pooled production system, Stage 3 candidate; from Sean's playtest notes). Do not implement until approved; design doc still governs.
- Fable/Opus gate comes after Sean plays the MVP for a few days.
