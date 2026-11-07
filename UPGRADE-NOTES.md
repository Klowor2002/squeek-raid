# SQUEEK RAID — Upgrade Notes: v1 → v2

*What changed, why, and the evidence that this build is not any of the things the
Genesis program bans. The red-team process behind this rebuild is documented in
[CRITIQUE-V2.md](CRITIQUE-V2.md). Nothing was deployed or pushed with these notes —
they describe the local build.*

## Numbers

| | v1 | v2 |
|---|---|---|
| Single file size | 45,863 bytes | **110,126 bytes** (2.4×) |
| Squeekchain endpoints used | 2 | **5** |
| Bosses with unique mechanics | 0 | **5** |
| Playable companions | 1 (fixed) | **3, with different buffs** |
| Progression systems | XP levels, 7 badges | levels, **12 badges, daily quests, Wheel Pass (10 tiers), upgrade tree (4 tracks, 13 levels)** |
| Failure state | none | **enrage timers with stacking armour** |
| Tests | none | **28-check runtime harness against live API payloads (all pass)** |

## What changed, system by system

**Bosses — mechanics, memory, grudges.**
Each boss gained a signature move that fires at 66% and 33% HP for 12 s: Photosynthesis
Rush (bar +35% speed behind a guard), Dust Shield (an absorbing lint shield equal to
12% of max HP), Drowse Field (marker freezes 0.7 s every ~4 s), Reverse Suction
(marker direction flips randomly), Deep Freeze (Perfect window −2, attacks cost
2 Energy). During guards, Good hits deal 25% and Weak hits deal 0 — Perfects are the
only full answer, which makes the timing skill the strategy. Bosses now persist
`{beaten, lastKiller, bestHit, scarRound}`: the memory line under the boss name and
a visible scar on the portrait after the first defeat. Each boss has a round budget
(60–80 rounds); overrun it and the boss enrages — +15% heal and a stacking −10%
damage-taken multiplier — so "eventually win by idling" is dead.

**Combat — combos.** Consecutive Perfects (4 s window) multiply damage ×1.0→×2.0.
The combo is displayed with its live multiplier. This turns the timing bar from a
reflex test into a streak-management game, and feeds Surge (below).

**Settlements — Surge banking.** Perfects at combo ×3+ bank Surge charges (max 5).
On settlement detection, the hamster's strike is `workAdded × 2 × (1 + 0.2 × charges)`
and the charges discharge, logged explicitly. Verified exactly in the test harness
(100 work, 3 charges → 320 damage).

**Companions.** Pip was joined by Mochi and Captain Crumb. Buffs are mechanical, not
cosmetic: Pip +1% Perfect window (applied inside `perfectHalfWidth()`), Mochi ×1.2
distance-to-Energy conversion and Focus +2 (+5 s cooldown), Captain Crumb +15% Good
damage. Each companion has a full dialogue set (16 event/context categories) with
real values templated in ({boss}, {rpm}, {pdmg}, {hdmg}…).

**Daily quests.** Three per UTC day from a six-quest pool, FNV-seeded by day key so
all visitors share the same board. Every quest is bound to a real chain event:
settlements witnessed (counted from feed deltas after a same-day baseline), damage
dealt while `RUNNING` is true, attacks landed in 3 distinct round numbers, metres
witnessed today, Focus uses, Perfects. Completion pays 1 Raid Token + 50 XP; a full
board pays a bonus token.

**Wheel Pass.** Ten tiers from 0 m to 32,000 m of *witnessed* distance — distance
counted only from live `/status` deltas while the page is open. The API's
`counterEpoch` is tracked; an epoch change re-baselines and logs, so a counter reset
can never be farmed. Tier-ups pay tokens and set the player's title.

**Upgrade tree.** Raid Tokens (bosses +1, quests +1, pass tiers +1) buy 13 upgrade
levels across 4 tracks: damage, Energy cap, Focus cooldown, mega-crit chance. Token
accounting is explicit in the save (`tokens`, `tokensSpent`, `tokensFromPass`).

**Chain surface.** New cards: Settlement Timeline (7 d feed, newest 12 settlements
with Etherscan links + a canvas chart of work per settlement), Hamster Almanac
(30 d per-day distance chart, career totals, best day, peak running window from the
stats feed), SqueekFi snapshot (SQK live supply vs. cap, token contract linked).
Pace states (Nap/Stroll/Jog/SPRINT) derive from real RPM in the live strip.

**Reviewer mode.** A toggle that prints the raw latest measurement, the RUNNING
derivation with both operands evaluated, the counter epoch, the settlement baseline
and poll timestamps. It exists because the v1 critique's sharpest point was that the
build's real engineering was invisible; now a reviewer can watch the derivation work.

**Stream overlay & badge.** `?overlay=1` renders a compact OBS view (second canvas
fed by the same draw routine). The SVG badge export generates a career portrait
with ERC-721-style metadata embedded — a dynamic-NFT-shaped artifact with zero
wallet involvement.

**Sound & motion.** WebAudio-synthesized effects (9 cues), a persisted mute, and
`prefers-reduced-motion` disables shake/floaters/transitions.

**Save v2 with migration.** New key `squeekraid_save_v2`; a v1 save is detected and
merged field-by-field (boss, cycle, HP, XP, level, stats, badges, settlement and
distance baselines, visits) — verified in the harness with a synthetic v1 save.

## Design rationale (the three judgement calls)

1. **Banked Surge instead of a real-time settlement button.** The API is polled
   (60 s); a "click NOW to channel" prompt would be fake interactivity layered on
   asynchronous data. Banking makes pre-settlement play matter, honestly.
2. **No fabricated rivals or leaderboards.** A "fantasy league" of invented
   hamsters would put fake data inside a build whose entire pitch is real data.
   The Wheel Pass measures the player against the one real athlete: the wheel.
3. **Depth before surface.** The critique's core charge was "tech demo". v2's budget
   went to mechanics/failure/builds first; the new data cards exist because quests
   and the pass needed the chain's history visible, not to pad an endpoint count
   (endpoints went 2 → 5 because the game systems consume them).

## Anti-slop checklist — banned category → concrete evidence

**"Read-only dashboard / explorer / tracker."**
The first screen is a game: an attack button, a timing bar, a boss that fights back.
Every datum on the page is wired to a decision — distance is spendable Energy,
settlements are ammunition the player amplifies, rounds are enrage clocks and quest
targets. The two data cards (Timeline, Almanac) sit *below* a full game and exist to
serve its systems. Remove the API and the game visibly degrades (no Energy economy,
no settlement strikes, frozen wheel); remove the game and nothing remains to submit.

**"Landing page / link hub / token page."**
No hero pitch, no call-to-action funnel, no token promotion. The SQK mention is a
supply figure with a testnet contract link, labelled "a snapshot, not a market".
Outbound links: Genesis, the live status page, this repo, and per-settlement
Etherscan proofs — evidence links, not a hub.

**"Launch with no product."**
The product is playable end-to-end in the submitted file: combat, progression,
failure, saves, sound, overlay. It was executed against the live API in a 28-check
harness before these notes were written (below).

**"Fork / template / light change."**
There is no upstream. The file is an original single-file build; v1 → v2 was a
ground-up deepening of the author's own v1 (git history in this repo), not a reskin
of anyone's project. The mechanics (settlement Surge, witnessed-metre pass, boss
memory) do not exist in the other Genesis projects we know of.

**"Thin API wrapper."**
A wrapper re-displays API fields. This build *transforms* them through game systems:
distance deltas → an energy economy with fractional banking and cap rules; settlement
work → damage with player-controlled amplification; round numbers → enrage deadlines
and quest state; 30 days of stats → charts and a peak-window strategy hint. The
running-detection rule (never `coreStatus` alone) and the counter-epoch guard are
domain logic a wrapper doesn't need and doesn't have.

**"Copy of someone else's project."**
Checked against the known Genesis field: not a round-prediction game (Nap or
Sprint?), not a puzzle (Live Garden), not a last-buyer token game (Hamster Nap),
not a stream odometer (Squeek Odometer) — although v2 does include an overlay,
it is one view of a full game, not the product. Boss-raid-versus-the-wheel with
settlement artillery is, to our knowledge, unoccupied.

**"Obvious slop / Aura-farm gaming."**
The anti-farm design is in the mechanics themselves: quests and the Wheel Pass
count only witnessed events, the first settlement read is baseline-only, the
distance counter is epoch-guarded, and progress requires the real wheel to really
turn while a human plays a timing game against bosses that escalate. The two
independent AI red-team reviews that called v1 borderline are preserved in
CRITIQUE-V2.md together with the fix mapping — this repo shows its working.

## Test evidence (2026-11-07, live API)

- All three inline script blocks pass `node --check`.
- Automated cross-check: all 84 element IDs referenced from JS exist in the HTML;
  zero external resource loads (only data-URI favicon; anchors are user-clicked).
- Served locally: HTTP 200, byte-identical 110,126 bytes.
- Node harness (DOM stub) executing the shipped code against **real fetched
  payloads** from all five endpoints — 28/28 checks pass, including: quest-seed
  determinism; RUNNING derivation vs. raw fields (`coreStatus=IDLE, rpmMilli=0 →
  resting` at fetch time); +250 m delta → witnessed metres and tier accounting;
  epoch change → zero teleport credit; centre strike = Perfect, combo ×3 → Surge
  banked; guard glance = 25% damage; dust shield absorption; settlement baseline
  (no retro damage), then crafted +1 round → exactly `100 × 2 × 1.6 = 320` hamster
  damage and Surge discharged; enrage heal + stack; upgrade costs 1 then 2 tokens
  with correct damage steps; quest completion payout; timeline/almanac/economy
  rendering from real payloads; save round-trip; v1 → v2 migration integrity.
- Live field shapes re-verified against the code's parsing for all five endpoints
  (string numerics, ISO `blockTimestamp` in settlements — handled via `Date.parse`
  fallback; nested `activity.days` in stats; `supply.liveSupplyBaseUnits` and
  `asset.decimals` in economy).

## Known limitations (stated, not hidden)

- `/status`'s anchor `settledThroughRound` can run ahead of the settlements feed's
  newest indexed batch; damage therefore lands from the feed (the same source as
  the baseline), occasionally in a larger combined hit. Displayed round comes from
  the anchor. This is the API's indexing lag, handled consistently, not papered over.
- Settlement countdown is an estimate from the newest settlement timestamp + 600 s,
  as in v1; the API publishes no exact next-settlement time.
- No backend means no cross-device sync and no global leaderboard — by design.
