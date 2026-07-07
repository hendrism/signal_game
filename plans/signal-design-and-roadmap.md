# SIGNAL — Design Doc & Roadmap to 1.0
*A text-based exploration/logistics idle game for iPhone/iPad. Single HTML file, home-screen app. Personal project for Sean.*

---

## 1. Vision
You are a lone salvage operator waking a dead planet's dormant network. Discover nodes through timed expeditions, connect them into a growing constellation, and research lost technology. Quiet, mysterious, uncluttered. FarmRPG's check-in rhythm meets Satisfactory's "find it, connect it, grow" loop, minus the visual sprawl.

**Core loop:** Expedition (timed, runs offline) → discovery (short text scene, sometimes a choice) → connect node to network (spend materials) → connected nodes produce automatically (also offline) → spend output on research → unlock deeper zones + better links → repeat.

**Design pillars:** discovery over efficiency · clean over cluttered · both 2-min check-ins and 30-min sessions feel good · story in fragments, never walls of text.

## 2. Systems Spec

### 2.1 Nodes & Network
- Node types: **Extractor** (produces a resource), **Ruin** (one-time loot + log fragment), **Relay** (extends network range into a zone), **Anomaly** (choice event, rare rewards).
- A discovered node is inert until **linked** (costs materials; cost scales with zone depth). Linked extractors produce per minute into global storage.
- Network rendered as abstract SVG constellation: dots + lines, auto-layout (simple force/radial layout by zone). No player-placed positions. Tapping a node opens its detail card.
- Link tiers (researchable): raise throughput cap per node. Throughput caps, not belt spaghetti, are the logistics puzzle: deciding *which* nodes to link and upgrade with limited materials.
- **Bandwidth (the core logistics tension):** the hub has finite bandwidth. Linked extractors consume 2^(linkTier−1) each; relays and certain techs add capacity. Capacity always runs ~20–30% short of "link everything," so each discovery poses a real choice: link the new node, or upgrade an old one? Nodes can be disconnected freely (link tier kept, materials not refunded) so the network can be rearranged. Numbers: base cap 6 · Signal Amplifier tech +4 · relays +6 each (tier 2+) · tier-gate techs add +4/+6/+8/+10 by tier.

### 2.2 Expeditions
- Pick a zone + duration (5m / 30m / 2h / 8h). Longer = deeper finds. Timer continues offline.
- On resolve: 1–3 discoveries, each a short text scene. ~20% include a choice (e.g., "salvage now for parts, or investigate for a log + chance of anomaly").
- One expedition slot at start; research unlocks a second (cap at 2, keep it simple).

### 2.3 Research
- Tech tree per tier. Costs resources + occasionally "tech fragments" (from ruins). Research is instant-buy (no timers; expeditions are the timers).
- Unlock types: new zones, link tiers, refining recipes, expedition slot, storage caps, QoL.

### 2.4 Refining (light crafting)
- Some techs need refined goods (e.g., Ore→Ingots). Refiners are *not* nodes: a simple "Workshop" tab with queue slots. Converts A→B at a rate, runs offline. Keeps crafting satisfying without factory sprawl.

### 2.5 Offline Progress
- On load: `elapsed = now − lastSaved`. Resolve expeditions if timers elapsed; add production × elapsed (capped by storage); advance refining queues. Show a "While you were away" summary card.

### 2.6 Saves
- Autosave to localStorage every 30s + on visibility change.
- Export/Import: JSON to clipboard / paste. Save includes `schemaVersion` for future migrations.

## 3. Data Architecture (critical)
All content lives in one `GAME_DATA` object, fully separate from engine code. Adding tiers 4–6 later = appending data only.

```js
GAME_DATA = {
  resources: { iron_ore: {name, tier, icon} , ...},
  recipes:   { iron_ingot: {in:{iron_ore:2}, out:{iron_ingot:1}, secs:10} , ...},
  zones:     { z1_lowlands: {name, tier, unlockTech, discoveryTable:[...]} , ...},
  nodes:     { /* templates: type, zone, produces, rate, linkCost */ },
  tech:      { t1_basic_links: {name, tier, cost:{...}, unlocks:[...], desc} , ...},
  logs:      { log_001: {title, text} , ...},
  events:    { ev_cache: {text, choices:[{label, effects}]} , ...}
}
```
State object mirrors this: discovered/linked nodes, inventory, researched techs, active expeditions, refining queues, seen logs, rng seed, timestamps.

> §9–§12 below are the authoritative detailed specs. Where they are more specific than §1–§8, they win.

## 4. Tier Arc (world plan — full 6, build 1–3 first)
1. **The Lowlands** — iron, copper, scrap. Learn the loop. Tech: basic links, workshop, 30m expeditions.
2. **Ashfields** — coal, quartz → steel, glass. Link tier 2, second expedition slot, 2h expeditions.
3. **The Sunken Grid** — old-world machinery: circuits, power cells. Relays introduced (network range). Story turn: the network isn't dead, it's *waiting*. **v1 "complete-feeling" ending beat here.**
4. **Cavern Veins** — rare ores, crystal. Throughput puzzles deepen (nodes with big output vs low caps).
5. **The Aerie** — high-altitude relays, weather anomalies, exotic materials. Anomaly events become central.
6. **The Signal Source** — endgame zone. Assemble the Beacon (big multi-resource project). Final log sequence. 1.0 ending.

## 5. UI Spec (bottom tab bar, thumb-friendly)
- **Network** (home): constellation SVG, storage bar, "collect/summary" cards.
- **Expeditions**: slots, zone/duration picker, resolve scenes appear here.
- **Workshop**: refining queues.
- **Research**: tech tree as tiered card list (not a graph — simpler, scannable).
- **Logs**: found fragments, reread anytime. Settings + export/import live here.
- Style: dark bg (#0d1117-ish), one cyan/amber accent, system font stack, generous spacing, subtle transitions. No icon libraries; tiny inline SVG glyphs only where needed.

## 6. Roadmap to 1.0

> **Model key:** 🟣 Fable/Opus (scarce — decisions & audits only) · 🔵 Sonnet (chat: writing, review, small code) · 🟢 Codex (agentic coding in the file) · 🟡 Gemini/Antigravity (agentic coding, testing, second opinions)
> **Workflow:** keep `signal.html` in a folder (ideally a git repo, even local-only). Codex and Antigravity edit the file directly; Sonnet gets pasted excerpts, never the whole file.

### Stage 0 — Design lock ✅ (this document)
🟣 Done. This doc is the source of truth. Give it to every tool at the start of every session.

### Stage 1 — Scaffold (engine skeleton)
Goal: file opens, tabs work, state saves/loads, clock runs. No real content.
1. 🟢 **Codex:** Create `signal.html`: HTML shell, tab bar, CSS theme, `GAME_DATA` stub, state object, save/load + export/import, game tick, offline-elapsed calc with summary card. Placeholder content: 1 zone, 2 resources, 1 node.
2. 🟡 **Antigravity:** Test pass — save/reload, offline math (change device clock), iPhone Safari layout, add-to-home-screen meta tags.
3. 🔵 **Sonnet:** quick code-structure review (paste engine sections). Fix nits via Codex.
- **Gate:** none needed. This is well-trodden ground.

### Stage 2 — MVP (the loop is real)
Goal: Tier 1 fully playable and fun.
1. 🟢 **Codex:** Expedition system (slots, timers, resolve), discovery scenes UI, node linking, production, storage caps.
2. 🟢 **Codex:** Constellation SVG render + node detail cards.
3. 🔵 **Sonnet:** Write Tier 1 content — zone flavor, ~12 discovery scenes, ~8 log fragments, 2 choice events. (Writing is Sonnet's sweet spot; give it §1 + §4 as context.)
4. 🟡 **Antigravity:** Integrate content data, playtest, bug pass.
5. **You:** play it for 2–3 days. Note what feels flat.
- **Gate 🟣 (recommended):** one Fable/Opus session — paste this doc + your play notes + engine outline. Ask: "audit the architecture and loop before I scale content." Cheap insurance before everything multiplies.

### Stage 3 — Beta (tiers 1–3, polished)
1. 🟢 **Codex:** First task: split the single file into `index.html` / `engine.js` / `data.js` per §8 (plain script tags, no build step). Then: Workshop/refining, research tree UI, relays, second expedition slot, link tiers.
1b. 🟢 **Codex:** **Survey Atlas** (per-zone found/unfound collection screen with completion %) and **expedition gear** (workshop-crafted items that weight discovery odds — e.g. magnetometer → ore, archive key → ruins/logs; consumed on use; schema addition needs Sean's approval per §12).
1c. 🔵 **Sonnet:** **Hub messages** — milestone-triggered lines from the network itself (garbled → coherent → aware), written against the story bible (separate file, give it to every content session). **Daily flare** — first expedition each day returns one bonus find (🟢 Codex wires the trigger).
1d. 🟢 **Codex + 🔵 Sonnet:** **Surveyor drone** — build the second expedition slot as a repairable drone found in tier 2 (workshop repair job). Same mechanics as a plain slot; Sonnet writes its name and a pool of one-line field remarks shown on expedition returns.
2. 🔵 **Sonnet:** Tiers 2–3 content: numbers first (draft balance table: rates, costs, timings), then scenes/logs including the tier-3 story beat.
3. 🟡 **Antigravity:** Balance testing (it can simulate/script progression math), edge cases: full storage, expired expeditions, importing old saves.
4. 🟢 **Codex:** Polish — transitions, "while away" card, number formatting, empty states, iPad layout.
5. **You:** play a full week.
- **Gate 🟣:** Fable/Opus balance + story audit with your notes. This is the highest-value checkpoint of the whole project.

### Stage 4 — Content expansion (tiers 4–6)
Engine is frozen; this is data + writing.
1. 🔵 **Sonnet:** Tier 4–6 balance tables, then content per tier (one tier per session, keeps context small).
2. 🟢 **Codex:** the Beacon endgame project (only new mechanic), anomaly event variety, save migration if schema changed, and **content packs**: a "Load content pack" option (Logs tab) that imports a JSON file matching the GAME_DATA schemas and merges it at runtime (validate ids, reject collisions). Ships with one tiny example pack as documentation.
3. 🟡 **Antigravity:** full-run playtest simulation, regression pass on tiers 1–3.
- **Gate:** 🟣 only if the Beacon design feels off or balance breaks. Otherwise skip.

### Stage 5 — 1.0 polish
1. 🟢 **Codex:** performance (large state, SVG with 100+ nodes), haptics-ish touches (subtle animations), a settings pane (reset, reduce motion), **ambient sound** (Web Audio: one low hum gaining a faint harmonic layer per zone connected; off by default, toggle in settings, no audio assets), **constellation export** (render the network to a downloadable wallpaper-sized PNG), final QA checklist below.
2. 🔵 **Sonnet:** proofread every string (you'll catch the rest — you're the grammar backstop).
3. **1.0 done when:** plays start-to-Beacon with no dead ends · offline math correct across days-long gaps · export/import round-trips · smooth on iPhone and iPad · every discovery has a scene · you'd rather check it than FarmRPG for a week.

## 7. How to brief each tool (paste-ready openers)
- **Codex/Antigravity:** "Here is the design doc for a single-file HTML game (attached). You're implementing Stage N, task X. Engine and data must stay separate per §3. Don't add frameworks, build steps, or extra files. Make targeted edits."
- **Sonnet (writing):** "Design doc §1 and §4 attached. Write [content batch] as JSON matching the schemas in §3. Tone: quiet, mysterious, fragments not paragraphs, 40–80 words per scene."
- **Fable/Opus (gates):** "Design doc + my playtest notes attached. Audit [architecture/balance/story] and give me the 3 highest-impact changes only."

## 8. Guardrails (read when tempted)
- No frameworks, no build step, no npm — ever. File plan: one file through MVP (Stage 2). At Stage 3, split into exactly three files: `index.html` (shell/CSS), `engine.js`, `data.js`, loaded with plain script tags. No further splitting without Sean's approval.
- No new mechanics after Stage 3 except the Beacon. New ideas go in a "2.0 ideas" note.
- If a session goes sideways, stop and restore from git/backup rather than patching patches.
- Content problems are cheap; engine problems are expensive. Protect the engine/data boundary.

---

## 9. Authoritative Data Schemas
Agents must use these exact field names. New fields require Sean's approval (§12).

```js
// ---- GAME_DATA (static content) ----
resources: { iron_ore: { name:"Iron Ore", tier:1, glyph:"◆" } }

recipes: { iron_ingot: { name:"Iron Ingot", tier:1,
  in:{ iron_ore:2 }, out:{ iron_ingot:1 }, secs:10, unlockTech:"t1_workshop" } }

zones: { z1_lowlands: { name:"The Lowlands", tier:1, unlockTech:null,
  flavor:"...", discoveryTable:[ { nodeId:"n_iron_a", weight:30 }, ... ] } }

nodes: { n_iron_a: { type:"extractor",           // extractor|ruin|relay|anomaly
  zone:"z1_lowlands", name:"Rustvein Deposit",
  produces:"iron_ore", ratePerMin:1,             // extractors only
  loot:{ scrap:25 }, logId:"log_003",            // ruins only
  eventId:"ev_cache",                            // anomalies only
  bwProvide:6,                                   // relays only: bandwidth capacity added when linked
  linkCost:{ scrap:20 }, sceneId:"sc_iron_a",
  unique:false } }                               // true = removed from table once found

tech: { t1_basic_links: { name:"Basic Links", tier:1,
  cost:{ iron_ingot:10 }, fragCost:0,            // fragCost = tech fragments
  requires:[],                                   // tech ids
  unlocks:[ {kind:"zone",id:"z2_ashfields"} ],   // kind: zone|linkTier|recipe|slot|storage|feature|bandwidth
  desc:"..." } }

logs:   { log_001: { title:"Field Note 7", text:"..." } }
scenes: { sc_iron_a: { text:"...", choiceId:null } }          // choiceId → events
events: { ev_cache: { text:"...", choices:[
  { label:"Salvage it", effects:{ grant:{scrap:40} } },
  { label:"Investigate", effects:{ grant:{}, logId:"log_009", anomalyChance:0.3 } } ] } }

// ---- STATE (saved) ----
state = { schemaVersion:1, lastSaved:0, createdAt:0, rngSeed:0,
  inv:{ iron_ore:0 }, storageCap:500,            // cap is per-resource
  nodes:{ n_iron_a:{ discovered:true, linked:false, linkTier:1 } },
  tech:[ "t1_basic_links" ],
  expeditions:[ { slot:1, zone:"z1_lowlands", ends:0, durationKey:"m30" } ],
  workshop:[ { recipe:"iron_ingot", qtyQueued:20, startedAt:0 } ],
  logsSeen:[ "log_001" ], pity:0, beacon:{ /* stage flags, Stage 4 */ } }
```

## 10. Balance Framework (design-locked)
Agents implement these numbers; changing them requires Sean's approval.

**Pacing targets:** Tier 1 ≈ 2 days casual play · each tier ≈ +50% longer · full run to Beacon ≈ 3–5 weeks.

**Tier 1 concrete numbers:**
| Thing | Value |
|---|---|
| Extractor base rate | 1/min (iron, copper), 0.5/min (scrap-rich ruinsfield node) |
| Link tier multiplier | ×2 per link tier (T1=×1, T2=×2, T3=×4) |
| Link cost, zone 1 | 20–40 scrap or iron_ore per node |
| Node link-tier upgrade | costs ≈ 3× the node's original linkCost, per tier |
| Storage cap | start 500/resource; upgrades ×2 (500→1k→2k…) |
| First techs | cost ≈ 10–15 min of production |
| Tier-gate tech (unlocks zone 2) | ≈ 1 day of casual production + 2 tech fragments |
| Refining | iron_ingot: 2 ore→1, 10s · copper_wire: 1 ore→2, 8s |

**Scaling formulas (tiers 2–6):**
- Extractor rates: ×3 per tier. Link costs: ×4 per tier (in that tier's materials). Net effect: each new zone feels expensive at first, trivial once integrated. That's the loop's emotional beat; don't smooth it out.
- Tier-gate tech: ≈ 1–1.5 days of then-current production + tech fragments (2, 3, 4, 6, 8 by tier).
- Tech fragments come only from ruins (guaranteed 1) and anomalies (0–2).

**Expeditions:**
| Duration | Discoveries | Notes |
|---|---|---|
| 5m | 1 | surface finds only (weight ×2 toward extractors) |
| 30m | 2 | full zone table |
| 2h | 3 | +1 reroll on duplicates |
| 8h | 3 | guaranteed ≥1 ruin or anomaly |

## 11. RNG, Discovery, and Layout Rules
- **RNG:** seeded mulberry32 (seed in state). All rolls go through one `rand()`.
- **Duplicates:** ruins/relays/anomalies are `unique:true` (removed from table when found). Extractors repeat by design; a duplicate extractor is a new node instance (`n_iron_a#2`).
- **Pity:** if 3 consecutive discoveries are duplicate extractors, next discovery is forced from the zone's unfound uniques. `state.pity` tracks this.
- **Exhaustion:** when a zone's uniques are all found, expeditions there yield extractors + small resource grants; UI marks zone "surveyed."
- **Constellation layout (deterministic, no physics):** hub at center. Each zone is an arc sector (zone 1 = inner ring, deeper zones = outer rings). Nodes placed along their zone's arc in discovery order, evenly spaced. Links draw node→its zone's relay, relay→hub (zone 1 nodes draw straight to hub). Unlinked discovered nodes render dimmed. Result: growth reads as an expanding constellation, and it can never tangle.

## 12. Agent Decision Protocol (paste into every session)
**You (the agent) may decide freely:** variable names, internal code structure, CSS details within the stated theme, bug-fix approaches, copy edits for typos.
**You must ASK Sean before:** adding/renaming any schema field · changing any number in §10 · adding mechanics, screens, tabs, or settings · adding dependencies, files, or build steps · changing save format (bump `schemaVersion` + write a migration if approved) · "improving" the design (log the idea in a 2.0-ideas note instead).
**When the doc is silent:** pick the simplest option consistent with §1's pillars, implement it, and flag it clearly in your summary as "DECISION MADE — review me."
**Escalation path:** Sean reviews flagged decisions; anything architectural or balance-related goes to a Fable/Opus gate session.
