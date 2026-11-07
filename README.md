# SQUEEK RAID

<!-- SQUEEKCHAIN-VERIFY: Rendi — paste the repository verification line from your Genesis project page here, replacing this comment. -->

**A co-op boss raid powered by Squeektoshi, Squeekchain's real hamster. The wheel is real. The raid is real. The points are not money.**

SQUEEK RAID turns Squeekchain's live hamster wheel into a boss-raid game. You fight a rotating roster of bosses with a timing-based attack game, your Energy refills from the *actual distance the real hamster runs*, and every time a round settles on-chain, Squeektoshi himself hits the boss for damage based on the settled work. A sidekick named Pip remembers your career and reacts to what the hamster is doing right now.

## Features

- **Live wheel strip** — a canvas pixel hamster on a wheel whose spin follows the real RPM. When the hamster naps, he curls up and floats "Z"s. Live readouts: distance tonight, wheel speed, RPM, open round, and a countdown to the next settlement.
- **Boss roster** — five bosses with escalating HP (Baron von Broccoli, The Dust Bunny King, Sir Snoozington the Nap Cat, VAK-9000 the Vacuum Menace, The Fridge Raider). The roster rotates weekly and advances on defeat; beaten the whole roster and they come back tougher.
- **Timing combat** — a marker sweeps a strike bar. Stop it in the gold for a Perfect crit, in the green for a solid hit, on the edges for a graze. Keyboard (Space) and button friendly, with floating damage numbers and screen shake on Perfects.
- **Energy from the real wheel** — +1 Energy per 25 m of real distance gained while you play. When the hamster runs live, the Perfect window widens and mega-crits arm. When he naps, a Focus button (+1 Energy, 20 s cooldown) keeps the raid moving.
- **Hamster raid damage** — when a settlement lands, the boss takes damage equal to the settlement's settled work × 2, logged in the raid log with a link to the settlement transaction on Etherscan.
- **Pip, a sidekick with memory** — handcrafted reactive dialogue (no AI calls) that responds to your history: bosses beaten, career damage, perfect count, long absences, first visits, and whether Squeektoshi is running or napping right now.
- **Progression** — XP and levels from damage dealt (level-ups raise your Energy cap and base damage), lifetime stats, and achievement badges.
- **Your save is yours** — everything persists in `localStorage`, with Export/Import save-as-JSON buttons. No accounts, no backend, no tracking.

## How it uses the Squeekchain API

Two public, CORS-enabled endpoints — no keys, no wallet:

- `GET https://api.squeekchain.com/v1/status` (polled every 10 s) — the live `measurement` (distance, speed, RPM), the builder `anchor` (settled-through round, block height), and the latest oracle closure. Energy gains and the live strip are driven from this.
- `GET https://api.squeekchain.com/v1/settlements?range=24h` (polled every 60 s) — recent settlement batches. When `settledThroughRound` advances, the boss takes `workAdded × 2` damage, with the settlement transaction linked from the raid log. The next-settlement countdown is estimated from settlement spacing (rounds are 600 s).

**Running-detection rule:** the hamster counts as RUNNING iff `rpmMilli > 0` **or** `speedMmPerSecond > 0`. `coreStatus` alone is never trusted — it can report IDLE while the wheel is actually moving. All numeric API fields arrive as strings and are parsed defensively; if the API is unreachable, the game keeps its last good data and shows an offline/stale badge rather than presenting stale data as live.

## How to run

It is a single self-contained `index.html` — no build step, no dependencies.

- Double-click `index.html` (works straight from `file://`), or
- serve the folder with any static host, e.g. `python3 -m http.server` in this directory, GitHub Pages, Netlify, Vercel, or IPFS.

## Tech

- One HTML file: inline CSS + vanilla JavaScript. Zero external dependencies, zero network calls except the two Squeekchain API endpoints above.
- Canvas for the animated wheel/hamster; DOM/CSS for everything else.
- State in `localStorage` only. No backend, no analytics, no cookies, no wallet connection, no transactions.

## Controls

- **ATTACK button** or **Space** — strike when the marker is where you want it (costs 1 Energy).
- **Focus +1** — gain 1 Energy, 20-second cooldown, for when the wheel is resting.
- **Export / Import save** — move your progress between browsers as a JSON file.

## Rules

Points only — no wallet connection, no transactions, no money, no betting.

Fan-made community game by kingklowor. Testnet only. Not affiliated with Squeekchain.
