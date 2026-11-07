# SQUEEK RAID — Red-Team Critique of v1, and the v2 Spec

*Compiled 2026-11-07 before the v2 rebuild. Method: the v1 build (README + full game
JavaScript) was handed to two independent AI reviewers instructed to act as hostile
Squeekchain Genesis reviewers deciding whether the submission is "low effort slop"
gaming the Aura Farm — the offence the Genesis page warns is punished with deletion
and an account ban. Reviewers: Nemotron-3-Ultra via OpenRouter (`riset`) and the same
weights via NVIDIA direct (`juri`). Two further helper calls (OpenRouter `saring`,
OpenRouter `cadangan`/Gemma) failed on provider credit/rate limits and contributed
nothing; the two completed reviews converged independently on the same core gaps,
which is why they were treated as decisive. This document merges them into one gap
list, then records the v2 spec that closed each gap.*

---

## Part 1 — The merged gap list (ranked by how damning a reviewer found each)

**Both reviewers' verdict on v1: borderline-reject.** The API integration was judged
correct and the save system real, but the *game* was judged a polished tech demo:
"a screenshot of a game, not a game that lives inside the chain's history" (Reviewer A);
"a single timing-bar with no boss mechanics, no player agency beyond 'press space'…
it reads like a polished tech demo, not a game" (Reviewer B).

1. **Bosses are HP sponges.** Five bosses differ only in name, colours, crown flag and
   an HP scalar. No phases, abilities, weaknesses or pattern changes. `damageBoss()`
   just subtracts. *(Both reviewers, ranked #1.)*
2. **One-button combat with no decisions.** The entire tactical depth is the Perfect
   window widening while the hamster runs. No resource trade-offs, no builds, no
   risk/reward beyond "hit gold." Reads as the most cloned web mini-game template
   there is. *(Both.)*
3. **Settlement damage is spectator mode.** The chain plays the game for you: damage
   pops in from a poll, kill credit is luck, and the player has zero agency during
   the one moment that is supposed to be the game's signature. *(Both.)*
4. **No persistent relationship with rounds.** Damage is an untracked number; no
   per-round ledger, no boss history, no "who felled this boss, in which round."
   The submission ignores its own best defence: the chain's history. *(A.)*
5. **Progression is linear stat inflation.** +2 cap / +3 damage per level; seven
   badges that are threshold `if`s on the same three counters. No choices anywhere,
   and no failure state — with enough time the player wins by default while the
   hamster does the bulk of late-game damage. *(Both.)*
6. **The anti-slop proof-of-work is invisible.** The defensive engineering (string
   parsing, no-retro-damage baseline, RUNNING derivation) is exactly what separates
   this from a wrapper — and a reviewer would never see it without reading source.
   "Verification theater": README claims presented as features instead of being
   surfaced in the product. *(A.)*
7. **No provenance.** No version, no build identity, no devlog, no tests, no
   instrumentation. "LLM-generated in an afternoon" is the default assumption a
   hostile reviewer brings; v1 gave them nothing to argue against it. *(A.)*
8. **Copy has the generated sparkle.** Pip's lines are competent and generic —
   interchangeable mascot quips; the UI never names its own real numbers
   (the actual ±% Perfect window). *(A, echoed by B on Pip being `pick([...])`.)*
9. **The chain is used as numbers, not as a world.** Rounds aren't levels, distance
   isn't a map, oracle closures and block height are ignored, and no other Genesis
   "wanted" category is touched (no overlay, no league flavour, no SqueekFi flavour,
   no dynamic-NFT-shaped artifact). *(A; B's fix list independently asks for overlay,
   league, NFT export, SqueekFi hooks.)*
10. **Art direction is generic CSS.** Gradient blob boss with two eyes; the pixel
    hamster is charm, not craft; no per-boss identity beyond hue. *(A.)*

**What both reviewers said NOT to lose:** the running-detection rule
(`rpmMilli > 0 || speed > 0`, never `coreStatus` alone); the no-retro-damage
settlement baseline; the fractional energy accumulator with cap-aware banking; the
ISO-week rotation; save validation on import; the points-only / no-wallet stance;
floaters, shake and cooldown UI ("juice is present").

---

## Part 2 — The v2 spec: how each gap was closed

| Gap | v2 response (all implemented, all in the same single file) |
|---|---|
| 1. HP-sponge bosses | **Signature mechanics**: at 66% and 33% HP each boss fires a 12 s signature move — Baron: bar +35% speed behind a guard; Dust Bunny King: a lint shield that absorbs damage (Perfects cut it at full value); Nap Cat: marker freezes mid-sweep; VAK-9000: marker direction flips unannounced; Fridge Raider: Perfect window −2 and attacks cost 2 Energy. During guards, Good hits glance (25%) and Weak hits glance off entirely. |
| 2. No decisions | **Combo system** (consecutive Perfects multiply damage up to ×2, 4 s decay), **upgrade tree** (4 tracks, bought with Raid Tokens), **3 companions with different real buffs** (Pip +1% Perfect window; Mochi +20% wheel Energy & Focus +2; Captain Crumb +15% Good damage). |
| 3. Spectator settlements | **Surge banking**: Perfects landed at combo ×3+ bank Surge charges (max 5); when a settlement lands, banked Surge amplifies Squeektoshi's strike +20% per charge and discharges. The player now plays *for* the settlement moment instead of watching it. (A real-time "channel/divert" choice was considered and rejected: polling is asynchronous, so a fake real-time choice would be theatre; banking is honest agency.) |
| 4. No round relationship | **Boss memory**: every boss records times felled, last killer (you vs. Squeektoshi), best hit taken, and the round it was scarred in — shown under its name, with a visible scar on the boss art after its first defeat. |
| 5. No failure, linear growth | **Enrage timers**: each boss has a round budget; overrun it and the boss enrages — heals 15% and gains stacking −10% damage-taken armour. Tokens create a real economy: bosses, daily quests and Wheel Pass tiers pay them; the upgrade tree is the sink. |
| 6. Invisible proof-of-work | **Reviewer mode**: a toggleable proof panel showing the raw latest API measurement, the parsed values, the RUNNING derivation evaluated live, the counter epoch, the settlement baseline, and poll timestamps. The defensive engineering is now a product feature. |
| 7. No provenance | Version in the header (v2), a devlog in the About section and README, an architecture section in the README, and this critique + UPGRADE-NOTES in the repo. |
| 8. Generic copy | Three companions with distinct voices and distinct things to say (Mochi's snack-ledger, Captain Crumb's regiment discipline), templated fills with real numbers (RPM, career damage, boss HP); the UI now states the true Perfect window (±4%, ±7% while running, +1% with Pip) and updates it live. |
| 9. Chain as numbers | **Daily quests bound to real events** (witness settlements, fight in 3 distinct rounds, deal damage while the wheel actually runs, witness 500 m today — seeded per UTC day, same for every visitor); **Wheel Pass** tiers unlocked only by distance the player personally witnessed, with a counter-epoch guard so a counter reset can't be farmed; **Settlement Timeline** (7 d feed, per-settlement work chart, Etherscan links); **Hamster Almanac** (30 d of nightly distance on canvas, career totals, peak running window); **SqueekFi snapshot** (live SQK supply vs. cap from `/economy/state`); **stream overlay mode** (`?overlay=1`); **dynamic SVG raider badge** export with embedded ERC-721-style metadata. |
| 10. Generic art | Bosses gained identity through behaviour and memory (shield ring visual while guarded, scar after defeat, mechanic banners); pace states (Nap/Stroll/Jog/SPRINT) derived from real RPM in the live strip. The hand-drawn pixel hamster stays — it was the one visual both reviewers liked. |

**Deliberately not done:** real-time settlement interaction (see gap 3), rival/hamster
league tables with invented competitors (fabricated "rivals" would be fake data in a
build whose whole pitch is real data), any wallet/address features (Genesis build
path does not require them; the game stays points-only by design).

**Verification of the v2 build** (see UPGRADE-NOTES.md for the full list): all inline
JS passes `node --check`; a Node harness with a DOM stub executed the real game code
against live API payloads — 28 behavioural checks pass (combat math, guard/shield,
Surge discharge = `workAdded × 2 × (1 + 0.2 × charges)`, enrage, epoch guard,
quest seeding determinism, Wheel Pass tiers, upgrade costs, save v1→v2 migration,
timeline/almanac/economy rendering).

*— End
...[truncated 2085 chars]