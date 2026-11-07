> **SUPERSEDED — this project moved to [Klowor2002/squeek-works](https://github.com/Klowor2002/squeek-works).** Play it live: [GitHub Pages](https://klowor2002.github.io/squeek-works/) · [Vercel](https://squeek-works-kingklowor-project.vercel.app/). This repo (squeek-raid) is kept only as history; the SQUEEK WORKS project lives at the new home.

# SQUEEK WORKS

<!-- SQUEEKCHAIN-VERIFY: Rendi — paste the repository verification line from your Genesis project page here, replacing this comment. -->

**A workshop tycoon whose power plant is a real hamster. The wheel is real. The watts are real. The points are not money.**

SQUEEK WORKS is a production-chain management sim powered end-to-end by Squeekchain's live data. The watts your factory can draw right now are computed from Squeektoshi's measured wheel speed this minute. Your batteries only ever charge from distance you personally witnessed the wheel turn. Your delivery orders are born from real on-chain settlements, sized by the real work those rounds settled, and expire on the chain's own cadence. When the hamster sprints, your workshop overheats. When he naps, you are on night shift: battery creep and fast maintenance. Manage the chain — Seeds → Flour → Bread, Seeds → Oil, Bread + Oil → Snack Crates — or watch it starve and jam.

This is v3, and it is a new game, not a patch. v1–v2 were a boss-raid clicker; the owner played v2 and rejected it in one sentence ("looks low effort, change the concept entirely"). The raid was scrapped whole — no bosses, no timing bar. The independent red-team critique of THIS concept is in [CRITIQUE-V3.md](CRITIQUE-V3.md) (headline: borderline fail; our point-by-point response is in there too), and the full change record is in [UPGRADE-NOTES.md](UPGRADE-NOTES.md).

## How to play

1. **Build** a Seed Mill (40⚙) and a Seed Press (55⚙) on your floor. Building an Oven requires a Mill standing; a Packer requires an Oven.
2. **Watch the power.** Live watts = wheel speed × 0.08 + rpm × 3. Machines draw 14–26 W each. Oversubscribe the wheel and every line throttles by priority — or flip a machine's breaker to IDLE/OFF and choose which line lives.
3. **Mind the battery.** Machines burn stored Wh as they work; the battery refills only from witnessed wheel distance (1 Wh per 25 m). A dead battery stops the floor even under a spinning wheel.
4. **Fill orders.** Every real settlement delivers seeds and spawns a crate order (1–6 crates by settled work), due within 3 settlements. Deliver for Cogs + Reputation; miss and reputation drops.
5. **Sell surplus, hire staff, improve the shop.** Flour/oil/bread/crates sell at the market. Pip, Mochi and Captain Crumb are hired once and assigned to ONE machine each. Improvements add floor slots, battery, buffers, heat sinks and wiring.

## The systems, honestly stated

- **Production chains with starvation and back-pressure** — a machine with no inputs is STARVED; a machine whose output store is full is JAMMED, and jams back the chain up.
- **Power allocation** — priority-weighted proportional throttling, plus hard ON/IDLE/OFF circuit breakers per machine (a red-team demand; see CRITIQUE-V3.md).
- **Real-data events** — rpm surge vs its recent average = Overheat (−30% efficiency, 60 s; venting costs 15 Wh and only halves it), idle wheel = Night Shift (construction 2× fast, lines creep on battery), steady pace = +15% efficiency for 5 min. Every event is journalled with the real values that triggered it.
- **Staff with addresses** — buffs apply to the machine the hamster is assigned to, never globally.
- **Settlement Timeline** (7 d, tx-linked), **Workshop Almanac** (the wheel's real last 30 days), **SqueekFi snapshot** (live SQK supply vs cap, contract linked), **Shift Journal** (events plus your own typed foreman's notes), **Reviewer proof panel** (raw API values, the RUNNING derivation evaluated live, the baseline rules), **stream overlay** (`?overlay=1`: watts, state, current order).
- **Sound** is synthesized WebAudio, mutable. The game respects `prefers-reduced-motion`. Your save is `localStorage` only, with Export/Import/Reset. New save key — v2 raid saves are not imported; this is a different game.

## How it uses the Squeekchain API

Five public, CORS-enabled endpoints — no keys, no wallet:

| Endpoint | Polled | Drives |
|---|---|---|
| `GET /v1/status` | 10 s | Live watts, RUNNING state, battery charging from distance deltas, event detection, open-round readout |
| `GET /v1/settlements?range=24h` | 60 s | Order generation, seed deliveries, order deadlines (counted in settlements) |
| `GET /v1/settlements?range=7d` | 5 min | Settlement Timeline |
| `GET /v1/measurement/stats?range=30d` | 5 min | Workshop Almanac |
| `GET /v1/economy/state` | 5 min | SqueekFi snapshot |

**Running-detection rule:** RUNNING iff `rpmMilli > 0` **or** `speedMmPerSecond > 0`. `coreStatus` alone is never trusted — it can report IDLE while the wheel moves.

**Anti-fake rules the game holds itself to:**

1. The first status read only sets a distance baseline — energy is never credited for distance from before you arrived.
2. If the API's counter epoch changes or the counter drops, the baseline resets instead of crediting a teleporting hamster.
3. Orders spawn only for settlements witnessed after load; the first feed read is a baseline.
4. All numeric API fields arrive as strings and are parsed defensively; a malformed field degrades one readout, never the game.

## Architecture & design notes

- **One file on purpose.** Markup, styles and ~1,400 lines of logic in a single `index.html`, zero dependencies — the submitted artifact *is* the source a reviewer reads. Works from `file://`, any static host, or IPFS, unchanged.
- **Chain as engine, not backdrop.** Every Squeekchain datum must change a decision: speed is the power grid, distance is the battery economy, settlements are the order book and the clock, the stats feed is the almanac. Data that changed nothing was not wired in.
- **The critique is shipped, not hidden.** CRITIQUE-V3.md contains an independent reviewer's "borderline fail" verdict on this very concept, what we changed because of it (circuit breakers, placed staff, build-gated recipes, paid venting, typed logbook), and what we did not change (slot floor instead of a drawable belt graph) — stated plainly.
- **No backend, by design and by rule.** All state is local; the only network calls are GETs to the public Squeekchain API. There is nothing to sign, connect, or approve.

## How to run

Double-click `index.html` (works straight from `file://`), or serve the folder with any static host. Add `?overlay=1` for the stream overlay.

## Tech

One HTML file: inline CSS + vanilla JavaScript. Canvas for the animated factory floor (belt speed follows the real rpm) and the almanac chart; DOM/CSS for the rest. WebAudio SFX, no audio assets. `localStorage` only. No backend, no analytics, no cookies, no wallet, no transactions.

## Devlog

- **v3.0 — SQUEEK WORKS.** New concept after the owner rejected the raid: workshop tycoon / production-chain sim. Live power grid from wheel speed, witnessed-distance battery, 4 machine types in 2 chains, priority + breaker power allocation, settlement-born orders, real-data events (overheat / night shift / steady pace), placed staff, improvement tree, almanac, proof panel, overlay, typed foreman's notes.
- **v2.0 / v1.0 — SQUEEK RAID (scrapped).** Boss-raid clicker. Rejected by the owner as low-effort in concept; kept in git history as provenance, not as the product.

## Rules

Points only — no wallet connection, no transactions, no money, no betting.

Fan-made community game by kingklowor. Testnet only. Not affiliated with Squeekchain.
