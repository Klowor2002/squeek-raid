# SQUEEK RAID

<!-- SQUEEKCHAIN-VERIFY: Rendi — paste the repository verification line from your Genesis project page here, replacing this comment. -->

**A boss raid powered by Squeektoshi, Squeekchain's real hamster. The wheel is real. The raid is real. The points are not money.**

SQUEEK RAID turns Squeekchain's live hamster wheel into a boss-raid game. You fight a rotating roster of bosses — each with its own signature mechanic, a memory of every defeat, and an enrage timer — with a timing-based attack game. Your Energy refills from the *actual distance the real hamster runs*. Every time a round settles on-chain, Squeektoshi himself hits the boss for damage based on the settled work — and the Surge charges you banked with your combo make that hit land harder. Three companions ride along, each changing how you play. Daily quests and a Wheel Pass track only the chain events you personally witnessed.

This is v2. v1 was the core raid loop; v2 is the game around it. The red-team critique that drove the rebuild is in [CRITIQUE-V2.md](CRITIQUE-V2.md); the full change list and the anti-slop evidence map are in [UPGRADE-NOTES.md](UPGRADE-NOTES.md).

## Features

- **Live wheel strip** — a hand-drawn canvas pixel hamster on a wheel whose spin follows the real RPM. When the hamster naps, he curls up and floats "Z"s. Live readouts: distance tonight, wheel speed, RPM with a pace state (Nap / Stroll / Jog / SPRINT, derived from the real RPM), open round, and a countdown to the next settlement.
- **Boss roster with mechanics and memory** — five bosses with escalating HP (Baron von Broccoli, The Dust Bunny King, Sir Snoozington the Nap Cat, VAK-9000 the Vacuum Menace, The Fridge Raider). At 66% and 33% HP each unleashes a 12-second signature move: a faster guarded bar, a lint shield that absorbs damage, a marker that freezes mid-sweep, a marker whose direction flips without warning, a narrowed Perfect window with double attack cost. Bosses **remember**: times felled, who landed the last killing blow (you or the hamster), the best hit they ever took, and the round that scarred them — shown under their name, with a scar on their portrait. Dawdle past a boss's round budget and it **enrages**: heals and gains stacking armour.
- **Timing combat with combos** — a marker sweeps a strike bar. Gold centre is a Perfect (×3, or a ×5 mega-crit while the hamster runs), green is a solid hit, edges graze. Consecutive Perfects build a combo multiplier up to ×2. The UI always shows the true Perfect window (±4%, ±7% while Squeektoshi runs).
- **Surge — agency at settlement time** — Perfects landed at combo ×3 or higher bank Surge charges (max 5). When a settlement lands, Squeektoshi's strike is amplified +20% per banked charge. You play *for* the settlement; you don't just watch it.
- **Energy from the real wheel** — +1 Energy per 25 m of real distance gained while you play. When the hamster runs live, the Perfect window widens and mega-crits arm. A Focus button keeps the raid moving through naps.
- **Three companions, three playstyles** — **Pip** (morale: +1% Perfect window, always), **Mochi** (wheel appetite: +20% Energy from distance, Focus restores 2), **Captain Crumb** (drill discipline: +15% damage on Good hits). Each has a full hand-written voice — greetings, combat reactions, quest and enrage lines — with real numbers templated in. No AI calls; all dialogue is handcrafted data.
- **Daily quests bound to the real chain** — three quests per UTC day, seeded identically for every visitor: witness settlements as they land, fight in three different rounds, deal damage while the wheel actually turns, witness 500 m of wheel today, and more. Each pays a Raid Token + XP; finishing all three pays a bonus token.
- **Wheel Pass** — progression tiers unlocked by distance you *personally witnessed* the wheel turn, from Seed Stasher (0 m) to Legend of the Wheel (32 km), each paying a Raid Token and a title. A counter-epoch guard means an API counter reset re-baselines instead of crediting a teleporting hamster.
- **Upgrade tree** — spend Raid Tokens on Sharper Seeds (+damage), Bigger Cheeks (+Energy cap), Zen Breathing (faster Focus), Lucky Whisker (+mega-crit chance).
- **Settlement Timeline** — the last 7 days of real settlements under the arena: rounds covered, work settled, blocks completed, a work-per-settlement chart, and an Etherscan link on every row.
- **Hamster Almanac** — 30 days of the wheel: a per-day distance chart drawn on canvas, career distance/work totals, the hamster's best day and its peak running window.
- **SqueekFi snapshot** — live SQK supply vs. cap from the economy endpoint, with the token contract linked.
- **Reviewer mode** — a "show the proof" toggle revealing the raw live API values, the parsed values, the RUNNING derivation evaluated in front of you, the counter epoch and the settlement baseline. The defensive engineering is a feature, not a claim.
- **Stream overlay** — open the page with `?overlay=1` for a compact OBS-friendly view: wheel, boss HP, RPM + pace, open round, settlement countdown, level and title.
- **Dynamic raider badge** — export an SVG badge with your title, level, bosses felled, best combo and week, with ERC-721-style metadata embedded in the file.
- **Sound, synthesized** — WebAudio bleeps for Perfects, settlements, level-ups and enrages. No audio files. Mutable, persisted, and the game respects `prefers-reduced-motion`.
- **Your save is yours** — everything persists in `localStorage`, with Export/Import save-as-JSON and Reset. v1 saves migrate automatically with all progress kept. No accounts, no backend, no tracking.

## How it uses the Squeekchain API

Five public, CORS-enabled endpoints — no keys, no wallet:

| Endpoint | Polled | Drives |
|---|---|---|
| `GET /v1/status` | 10 s | Wheel animation (real RPM), Energy from distance deltas, running buff, open round, quest distance/running-damage tracking, Wheel Pass metres |
| `GET /v1/settlements?range=24h` | 60 s | Settlement detection → Squeektoshi's raid damage (× Surge), next-settlement countdown, witness quests |
| `GET /v1/settlements?range=7d` | 5 min | Settlement Timeline + work chart |
| `GET /v1/measurement/stats?range=30d` | 5 min | Hamster Almanac (nightly distance, career totals, peak window) |
| `GET /v1/economy/state` | 5 min | SqueekFi snapshot (SQK live supply vs. cap) |

**Running-detection rule:** the hamster counts as RUNNING iff `rpmMilli > 0` **or** `speedMmPerSecond > 0`. `coreStatus` alone is never trusted — it can report IDLE while the wheel is actually moving.

**Anti-fake rules the game holds itself to:**

1. The first settlement seen after loading only sets a baseline — no retroactive damage for rounds you weren't here for.
2. Distance is counted only from live deltas; if the API's counter epoch changes, the baseline resets instead of crediting the jump.
3. Quests and the Wheel Pass count only events witnessed while the game is open — nothing accrues from a dead wheel or a closed tab.
4. Stale or offline data is badged as such, never dressed up as live. All numeric API fields arrive as strings and are parsed defensively; a malformed field degrades one readout, never the game.

## Architecture & design notes

- **One file on purpose.** The whole game — markup, styles, ~2,100 lines of logic — is a single `index.html` with zero dependencies, so the submitted artifact *is* the source a reviewer reads. It works from `file://`, any static host, or IPFS, unchanged.
- **Chain as engine, not backdrop.** The design rule was: every Squeekchain datum must change a decision. Distance is the energy economy; settlements are artillery with player-banked amplification; rounds are the clock for enrage and quests; the stats feed is the almanac; the economy feed is flavour with a link to the real contract. Data that changed nothing was not wired in.
- **Bosses before dashboards.** Reviewers of v1 (two independent AI red-team passes, see CRITIQUE-V2.md) called the loop a tech demo, so v2's budget went into game depth first — mechanics, memory, enrage, combos, builds — and into surfacing the chain integration second (timeline, almanac, proof panel).
- **Honest agency.** Settlement interaction is banked Surge, not a fake real-time button: the API is polled, so the game never pretends a click at settlement second zero matters. What you do *before* the settlement (holding a combo, banking charges) is what matters.
- **Deterministic shared quests.** The daily quest set is an FNV-seeded shuffle of the UTC day key — every visitor worldwide gets the same three quests, verifiable by anyone, with no server.
- **No backend, by design and by rule.** All state is local; the only network calls are GETs to the public Squeekchain API. There is nothing to sign, connect, or approve.

## How to run

It is a single self-contained `index.html` — no build step, no dependencies.

- Double-click `index.html` (works straight from `file://`), or
- serve the folder with any static host, e.g. `python3 -m http.server` in this directory, GitHub Pages, Netlify, Vercel, or IPFS.
- Add `?overlay=1` to the URL for the stream overlay view.

## Tech

- One HTML file: inline CSS + vanilla JavaScript. Zero external dependencies; the only network calls are the five Squeekchain API GETs above.
- Canvas for the animated wheel/hamster, the settlement work chart, and the almanac chart; DOM/CSS for everything else.
- WebAudio for synthesized sound effects (no audio assets). State in `localStorage` only. No backend, no analytics, no cookies, no wallet connection, no transactions.

## Controls

- **ATTACK button** or **Space** — strike when the marker is where you want it (costs 1 Energy; 2 during Deep Freeze).
- **Focus** — gain Energy on a cooldown, for when the wheel is resting.
- **Companion buttons** — switch between Pip, Mochi and Captain Crumb at any time.
- **Export / Import save**, **Export badge (SVG)**, **Reset** — in the save card.

## Devlog

- **v2.0** — boss signature mechanics + guards, enrage timers, boss memory & scars, combo system, Surge banking, 3 companions with distinct buffs and voices, daily chain-bound quests, Wheel Pass, upgrade tree, settlement timeline, hamster almanac, SqueekFi snapshot, reviewer proof panel, stream overlay mode, dynamic SVG badge, synthesized SFX, pace states, v1 save migration.
- **v1.0** — core raid loop: timing combat, energy from distance, settlement raid damage, weekly boss rotation, Pip, XP/levels, 7 badges, save export/import.

## Rules

Points only — no wallet connection, no transactions, no money, no betting.

Fan-made community game by kingklowor. Testnet only. Not affiliated with Squeekchain.
