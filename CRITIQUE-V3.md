# CRITIQUE-V3 — independent red-team review of SQUEEK WORKS

Reviewer: Nemotron-3-Ultra (NVIDIA direct, `juri` role), given only the concept spec and the Genesis banned-category list, and told to be unsparing. Its full verdict is reproduced verbatim below — including the headline, which is **BORDERLINE FAIL**. We asked for a red team, not a compliment, and we are publishing what we got.

## What we changed because of this review (before shipping v3)

| Criticism | Response in the shipped build |
|---|---|
| "Priority sliders = fake agency; add hard circuit breakers" | **Adopted.** Every machine has an ON / IDLE / OFF breaker next to its priority slider. IDLE draws 10% and creeps; OFF draws 0. When the wheel slows, the player chooses which line lives — the slider alone no longer decides. |
| "Staff = global multipliers" | **Adopted.** Staff are hired once and *assigned to one machine* (mutually exclusive). Pip only supercharges the Packer he's standing on; Mochi only thins a Mill's seed appetite on her Mill; Crumb doubles the buffers of his machine only. |
| "Reputation tiers = prestige gates for recipes" | **Adopted.** Recipes unlock by *building*: the Oven appears once a Mill stands, the Packer once an Oven stands. Reputation is standing/titles, not a recipe lock. |
| "Events = threshold tooltips; vent = free click" | **Partly adopted.** Venting costs 15 Wh of battery — charge you may need for the ovens — and only halves the penalty, so riding out the heat is a real alternative. Night Shift halves construction time (a scheduling decision: build at night). |
| "Almanac = read-only chart" | **Partly adopted.** The Shift Journal accepts the player's own typed foreman's notes (140 chars, saved with the shift), so the logbook is partly player-authored. The Almanac chart stays — it is context, clearly labelled as the wheel's history, not gameplay. |
| "No back-pressure" | **Partly adopted.** Finished-goods stores are capped; a full store jams the machine feeding it (JAMMED state), and a jammed Packer starves nothing but backs bread and oil up the chain. Machine output buffers are visible per machine. |
| "Full tile-graph belts / event minigames" | **Not adopted — stated honestly.** A drawable belt graph and three input minigames did not fit the single-file scope alongside the full economy sim. The floor is slot-based; belts on the canvas are visual. This remains the concept's known weakest flank, and the reviewer is right to name it. |

---

## The review, verbatim

**(a) VERDICT: BORDERLINE FAIL — HIGH RISK OF "THIN WRAPPER" REJECTION**  
The concept *sounds* like a game, but the spec reads as an API visualizer wearing a tycoon skin. Every "mechanic" (power budget, settlement orders, events, almanac) is a 1:1 mapping of a JSON field → UI widget. There is no *player-authored complexity*: no spatial logistics (belts are cosmetic), no scheduling decisions (priority sliders are just weight math), no trade-offs (upgrade tree is linear stat sticks). If the hamster API went offline, the "game" collapses to a broken spreadsheet. Anti-slop reviewers will flag this as "dashboard with build buttons."

**(b) 5 WEAKEST POINTS (LOW-EFFORT SIGNALS)**  
1. **Zero Spatial/Topological Gameplay** — "Floor slots" + "belts" = grid buttons + CSS animation. No routing, no back-pressure visualization, no layout optimization. It’s a list with pictures.  
2. **Priority Sliders = Fake Agency** — Proportional throttling is a single `Math.min(draw, budget * weight / totalWeight)` line. Player "decision" reduces to "max everything, watch numbers equalize." No hard choices (shut down Mill to save Oven).  
3. **Settlement Orders = Polling Quests** — "Fulfil before next N settlements" is a timer bar fed by `workAdded`. No routing, no inventory reservation, no partial fulfilment strategy. It’s a progress bar with a deadline.  
4. **Events = Threshold Tooltips** — Overheat/Night Shift/Efficiency are `if (rpm > avg*1.2) log("Overheat")`. No mitigation minigame, no risk/reward (vent = click button). Log entry ≠ gameplay.  
5. **Almanac/Reputation = Read-Only Charts** — 30-day chart, career totals, SQK snapshot are pure `fetch → Chart.js`. Zero player input. This is the definition of "read-only dashboard" the rules ban.

**(c) CONCRETE SINGLE-FILE FIXES**  
1. **Spatial Belts → Tile Graph** — Replace slot list with a 12×8 grid. Machines occupy 1–4 tiles; belts are *edges* player draws (click-drag). Items move as discrete tokens along edges (requestAnimationFrame). Back-pressure = token queue on edge. *Cost:* ~120 lines (grid, pathfind, token pool).  
2. **Hard Power Switches** — Remove sliders. Add **Circuit Breakers**: each machine has a toggle (ON/IDLE/OFF). IDLE draws 10% power, keeps buffers, cools faster. OFF draws 0, loses heat bonus. Player must *choose* which line lives. *Cost:* 20 lines + UI toggle.  
3. **Settlement Logistics** — Orders require **crates** (crafted item: 1 Bread + 1 Oil → Packer → Crate). Crates occupy 1×1 warehouse tiles (new 6×4 grid). Player reserves crates for orders; unfulfilled crates rot after 3 settlements (spoilage mechanic). *Cost:* 40 lines (crate item, warehouse grid, rot timer).  
4. **Event Minigames** — Overheat: 3s "vent" click-spam (spacebar) reduces efficiency loss 50% → 10%. Night Shift: "Maintenance" minigame (Simon-sequence on 4 keys) cuts repair time 40%. Efficiency Bonus: "Rhythm" — press key on RPM peak (visual metronome) for +15% next 60s. *Cost:* 80 lines (state machines, input capture).  
5. **Almanac → Player Logbook** — Replace charts with **hand-written entries** player *types* (max 140 chars/settlement). Store in `localStorage`; render as scrollable notebook. Reputation tiers unlock *ink colors* (cosmetic). *Cost:* 30 lines (textarea, array, render).

**(d) RESKIN SMELLS (CLICKER DNA)**  
- **"Upgrade Tree" = Clicker Perks** — "+10% Mill speed", "+5% Oil yield" are idle-game stat sticks. *Fix:* Upgrades unlock *new topology* (e.g., "Splitter Belt" → 1 input → 2 outputs; "Buffer Chest" → 50-slot tile).  
- **"Staff" = Global Multipliers** — Hamster buffs = `globalRate *= 1.15`. *Fix:* Staff are *placed* on tiles (Mechanic on Mill → reduces jam chance; Lubricator on Belt → +token speed). One per tile, mutually exclusive.  
- **"Power Budget" = Energy Bar** — Same "wait for bar, spend" loop. *Fix:* Power is *physical* — generator tile (hamster wheel) outputs tokens on belt; machines consume tokens. No global budget.  
- **"Reputation Tiers" = Prestige Gates** — Unlock recipes at Rep 10/25/50. *Fix:* Recipes unlocked by *building* prerequisite machine (Oven built → Bread recipe appears). Rep unlocks *cosmetic* factory themes.  
- **Single-File Excuse** — "One HTML" is used to hide lack of asset pipeline. *Fix:* Embed 16×16 pixel art as base64 `<img>` tags (machines, tokens, hamsters). 30 sprites < 40 KB. Proves art effort, not code golf.

**Bottom line:** Strip every global slider, chart, and auto-log. Make the player *route*, *switch*, *click*, *type*, and *place*. If the hamster API dies, the factory logic (tokens, grids, crates, vents) still runs. That’s a game. Current spec is a dashboard.
