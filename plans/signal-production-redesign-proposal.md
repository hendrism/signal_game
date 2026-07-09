# SIGNAL — Production Redesign Proposal (Stage 3 candidate)

*Status: PROPOSAL — Sean marks this up before anything is built. Nothing here is
implemented. Source: Sean's playtest notes + design session, 2026-07-08/09.
The design doc (`signal-design-and-roadmap.md`) remains the source of truth
until Sean approves this and it is merged into that doc as a §2 revision.*

---

## 1. Why (playtest finding)

After playing the Stage 2 MVP, the core gap: **discovery poses no creative
decision.** Current loop is discover → link → passive drip. The only choices
are which node to link (bandwidth) and which to upgrade. What's missing is the
moment: *"I found something new… what can I make here?"*

This proposal makes every discovery a production decision while keeping the
game's idle math linear and its check-in rhythm intact.

## 2. New design pillar (Sean, recorded verbatim in spirit)

> One of my favorite things is finding new products to make. I love the
> upgrading, I love collecting.

**Pillar: deep catalogs.** The long game (tiers 3–6) should scale toward many
products, many recipes, many techs — a collector's tech tree, not a short
checklist. Content sessions should treat "a new thing to make" as a
first-class reward alongside logs and zones. (Collection UI: Survey Atlas,
already planned Stage 3, extends naturally to a product/recipe catalog.)

## 3. The system (rate-pooled production)

### 3.1 Core model
- Every linked **raw node** contributes its extraction rate to a single
  network-wide pool (no adjacency, no spatial logistics — the hub pools
  everything). Example: Rustvein Deposit adds 10 iron ore/min.
- A **production assignment** at a node commits input rates from the pool and
  produces an output rate. Example: Gas Filters commit 5 iron ore/min +
  2 coal/min + 1 sulfur/min → 0.5 gas filters/min.
- **Surplus flows to inventory.** Net rate per resource = produced/min −
  committed/min. Sean's example: 10 iron/min produced, 5 committed to Gas
  Filters → +5/min to inventory; assign 3 more elsewhere → +2/min.
- Products are rates too: they accumulate in inventory or feed downstream
  assignments (ore/min → ingots/min → filters/min). Chains fall out for free.

### 3.2 The three rules that keep it idle-simple
1. **Commitment, not starvation.** An assignment can only be created if the
   free rate exists at that moment. Factories never run partial, never stall.
   This mirrors bandwidth: bandwidth answers "can this node be on?", the rate
   budget answers "can this factory be fed?" Two parallel budgets, one mental
   model.
2. **Factories eat rates, never stockpiles.** Banked inventory feeds research,
   link costs, and one-time spends — never production. Keeps offline catch-up
   strictly `net rate × elapsed` (same function live and offline, per current
   engine law). No piecewise "ran until the stockpile emptied" math, ever.
3. **One product per node (extract OR manufacture).** Assigning production to
   a resource node forfeits its extraction. Every discovery asks: mine or
   factory? — the decision moment this whole redesign exists to create.

### 3.3 Node slots (future-proofing, approved direction)
- Sean's clutter concern: many single-purpose nodes could sprawl. Bandwidth
  already self-limits sprawl (every active node costs bandwidth), so tier 1–3
  ships with **one slot per node**.
- **Schema is a slot list from day one** (`slots:[assignment]`, length 1) so
  multi-slot nodes can arrive later without a save-format break. Slot-count
  upgrades become a research/upgrade sink in later tiers — feeds the deep
  catalogs pillar.

### 3.4 Disconnect cascade (Sean's flow, adopted)
Disconnecting a node whose output feeds assignments shows a consequence
dialog: *"Disconnecting this will idle: Gas Filters (missing 5 iron
ingot/min) → Filter Assembly (missing 0.5 filters/min)"* with **Disconnect
anyway / Cancel**. On confirm, affected assignments go **idle — missing
inputs**: configuration kept, commitments released, output zero. Idle = zero
rate, so offline math is unaffected.
- OPEN (playtest): auto-resume idle assignments when inputs return (lean yes,
  with a toast) vs. manual restart.

## 4. UI

### 4.1 Node view (the centerpiece — Sean's radial concept)
Opening a node centers it with circular buttons around it: everything it
could produce. Available options highlighted; locked options visible with
missing requirements shown ("needs 2 more iron ore/min" / "research Gas
Filtration"). Discovering a new resource can instantly light up options on
existing nodes — exploration feeds production feeds exploration.

### 4.2 Production page (replaces Workshop tab — stays at 5 tabs)
Per-resource rate ledger. The headline number is always net flow to
inventory:

```
Iron ore    +2/min   (10/min total · 5 → Gas Filters · 3 → Smelter)
Iron ingot  +1/min   (3/min total · 2 → Filter Assembly)
Gas filter  +0.5/min
```

### 4.3 Inventory page
Stockpile view: every raw / intermediate / finished good with amount and cap.
- OPEN: own tab vs. expanded storage-strip panel (lean: section on the
  Production page — rates above, stockpiles below, one scroll).

## 5. What this replaces / what stays

| Piece | Fate |
|---|---|
| Workshop queues (§2.4) | Replaced — recipes become node assignments; tab becomes Production. Workshop tech reflavored as unlocking first conversion recipes. |
| Bandwidth (§2.1) | **Stays, unchanged.** Primary scarcity; anti-sprawl valve. |
| Expeditions, discovery, pity, logs, events, story | Untouched. |
| Manual sweep, seeded start (§2.7) | Untouched. |
| Link tiers | Reinterpreted: raise a node's extraction rate or slot throughput (OPEN — decide during spec merge). |
| Nodes schema | `produces/ratePerMin` generalizes to material contents + slot list. Schema change → save migration + `schemaVersion` bump. |

## 6. Rejected alternatives (so we don't relitigate)
- **Adjacency / "nearby node" connections** — needs a spatial system the
  constellation doesn't have; global rate pooling delivers the same choices
  at a fraction of the cost.
- **Zone-pooled manufacturing** (Fable's earlier suggestion) — superseded by
  Sean's simpler global pooling. Could return later as a tier 4+ wrinkle if
  the pool ever feels too free.
- **Partial/starved factory operation** — breaks linear offline math; the
  commitment model replaces it.

## 7. Story integration (free thematic win)
Tier 3's turn is "the network isn't dead, it's waiting." The colony fled
because the network completed work they hadn't approved. Production
assignment IS the player doing what spooked the colonists — telling the
network what to make. Mechanic reveal and story reveal can land together:
introduce assignments in tier 1–2 simply, let the hub start *anticipating*
assignments in tier 3 (suggesting recipes, pre-lighting options).

## 8. Phasing (recommendation)
1. **Now:** Sean marks up this doc. No engine work.
2. **Prototype (one session, throwaway):** radial node UI with fake data —
   validate the "open a node, see what it could become" moment on iPhone
   before any engine surgery.
3. **Stage 3 opens with the planned file split** (`index.html` / `engine.js`
   / `data.js`) — the natural surgery point. Production system becomes the
   centerpiece of revised Stage 3, displacing some planned Stage 3 items
   (workshop UI polish; relays/second slot stay).
4. Balance framework for rates/recipes (§10 extension) written *before*
   content, same as the original doc's numbers-first rule.
5. Fable/Opus gate reviews the marked-up proposal + prototype feel before the
   engine rewrite is approved.

## 9. Open questions for Sean's markup
1. Auto-resume idled assignments when inputs return? (lean yes)
2. Inventory: own tab or Production-page section? (lean section)
3. Link tiers: extraction-rate multiplier, slot throughput, or both?
4. Do anomaly/ruin nodes ever host production, or extractors only?
5. Recipe unlock grain: per-recipe techs (collector-friendly, many small
   unlocks — fits the pillar) vs. category techs (fewer, chunkier)?
