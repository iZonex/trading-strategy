---
name: Trapped Longs Fingerprint + FR-Isolation Anti-Pattern
description: Post-mortem of 3 failed mechanics reads (RAVE, ARIA, MYX) on 2026-04-14. Documents (a) a sharp N=1 hypothesis for distribution-top detection and (b) an anti-pattern where reading FR signature in isolation produces opposite-direction errors. Hypothesis only — requires out-of-sample backtest before use.
type: research
---
# Trapped Longs Fingerprint + FR-Isolation Anti-Pattern — 2026-04-14

## Summary

Two linked findings from 2026-04-14 afternoon session. Both originate from post-mortem of 3 mechanics reads made at 16:03/16:17 UTC that went wrong within 25–45 minutes.

1. **Trapped Longs Fingerprint (hypothesis, N=1):** a specific 5-attribute observable state that marks distribution-top / imminent-cascade conditions. Validated on RAVE 16:03 UTC → −19% cascade at 16:45 UTC. Needs out-of-sample historical backtest before any use.
2. **FR-Isolation Anti-Pattern (documented failure mode, N=3):** reading funding rate signature without cross-checking OI direction and price motion produces opposite-direction errors. Same numerical FR signature maps to multiple opposing mechanics.

Neither finding is an edge. The fingerprint is a hypothesis. The anti-pattern is a process rule that prevents a specific class of errors.

## Context: the 3 failed reads

At 16:03 UTC I produced narrative mechanics reads on RAVE, ARIA, MYX using `tests/357-mechanics-reader.mjs` (a plain-language narrative reader built earlier in the session). Each read was logged as a prediction with falsifier conditions in `data/predictions-log.jsonl`.

The reader had two bugs that silently corrupted the FR data:
- `/fapi/v1/fundingRate?startTime=t-24h&limit=20` truncates the newest settlements on 1h-funding coins (24 records > limit 20, Binance returns oldest-first).
- Filter `fundingTime <= t + 1h` leaks future settlements into historical reads.

On RAVE the tool showed latest FR as `-0.887%` when reality was `-2.0000%` pinned at cap for 2 consecutive settlements. The cap signal — mechanically critical — was invisible.

### RAVE 16:03 UTC

Observable state (after tool fix, retrospectively):
- FR pinned at −2.0000% × 2 consecutive hourly settlements
- OI shrinking: $88.54M (14:30) → $80.83M (16:00), −9% over 90 min
- Price flat: last 1h move +0.11%, 5m bar range ±1.6%
- Cycle peak was 6h ago at $14.84; current $13.18 = −11.2% from peak
- Duration post-peak: 6h

My call: "Pure A1 multi-leg mid-cycle pullback, next leg UP within 2–4h above $13.50."

Reality: at 16:45 UTC, one 5m candle printed O $13.08 → C $11.84 (−9.47%), low $10.53, volume $58M (vs prior $5–8M bars). OI dropped $80M → $71M in 5 minutes. Long liquidation cascade.

Why I was wrong: I matched FR signature ("deeply negative, deepening") to the "shorts piling in for squeeze" template without checking OI direction. Shorts piling in requires OI to GROW — on RAVE OI was SHRINKING. The two observables were inconsistent with my story but I didn't run the consistency check.

### ARIA 16:03 UTC

Observable state:
- FR −2.0000% (deepening from −1.8065%, pinned at cap)
- OI shrinking mildly: −3.6% over 2h
- Price falling fast: last 1h move −24.31%, largest 5m bar −15.78%
- Cycle peak $1.014 was 8h ago, currently $0.5666 = −44% from peak

My call: "Post-crash short-trap forming, 50/50 bounce vs further down."

Reality: further −54.8% crash in 40 min. Price hit $0.2100. OI cascade −47.5% over 2h (mass long liquidation).

Why I was wrong: I read "FR deepening negative while price falls" as "shorts piling into falling knife = trap forming." But a short trap requires price to move AGAINST shorts (up), not WITH them (down). Shorts on a falling market are winning, not trapped. The price motion contradicted my story.

### MYX 16:17 UTC

Observable state:
- FR +0.0050% (baseline, flat)
- OI growing fast: +18.6% over 2h
- Price recovering: last 1h move +2.25%
- Post-crash: −56% from 12h cycle peak

My call: "OI re-loading, conditional on FR direction — if goes positive → long build → bounce."

Reality: −9.3% in 25 min, OI reversed from +18.6% to −7.8% (loading unwound, i.e. whoever was building got wiped).

Why I was wrong: If ONE side were truly building a directional position, FR would have started drifting off zero. It didn't. Flat FR with growing OI means BOTH sides are entering simultaneously (disagreement market), not a directional build. In a post-crash context, disagreement typically resolves with dip-buyers (longs) getting wiped on continuation.

## Finding 1 — Trapped Longs Fingerprint (HYPOTHESIS, N=1)

### The fingerprint

A set of five co-occurring observables at a specific cycle phase:

1. **FR at cap** (−2.0% for Binance extreme coins, or whatever the coin-specific cap is) for **≥2 consecutive settlements**
2. **OI slowly shrinking** (not cascading, not growing) — typical rate −3% to −10% over 1–2h
3. **Price approximately flat** (|1h move| < 1.5%) near or slightly below cycle peak
4. **Cycle post-peak duration ≥4h** (distribution time)
5. **Bars quiet** — 5m bar ranges compressed, volumes normal-to-low

### Proposed mechanic

Late longs bought near cycle peak. Price retraced but they can't exit as a block without crashing the market. The core long mass is stuck. FR is pinned at cap because the funding mechanism is trying to balance a positioning book that is catastrophically long-heavy, and it has maxed out its balancing force. Some longs trickle out (slow OI bleed). Most sit, paying extreme funding per settlement, hoping for a bounce that isn't coming.

Resolution: when price breaks a key support level, the stuck longs trigger a liquidation cascade. OI drops sharply in a single candle, price gaps down, volume spikes.

### Why "Pure A1 multi-leg squeeze" template reads this wrong

The general Pure A1 / multi-leg template says: "FR deep negative + still-active cycle = shorts being squeezed, more legs up likely." That template catches `FR negative + OI growing + price flat/up`.

The fingerprint above is `FR negative + OI shrinking + price flat`. The OI direction is opposite. Same FR, same duration, same cycle magnitude — but the mechanic is reversed. This is the specific template-matching error that led to the RAVE miscall.

### Validation status

- **N=1:** RAVE 2026-04-14 16:03 UTC → cascade at 16:45 UTC, −19% from read price, confirmed as trapped-long liquidation via OI drop pattern.
- **Zero out-of-sample backtesting.**
- **Zero negative cases:** coins that matched this fingerprint and DID bounce instead of cascading are unknown.

This is a hypothesis on a single case. It fits the data it was derived from, trivially. It may or may not survive contact with historical data.

### Out-of-sample protocol

Before using this fingerprint to call direction on new coins:

1. **Historical scan.** Identify 30+ coins over the past 30–90 days where:
   - FR hit cap for ≥2 consecutive settlements
   - Coin had a prior cycle ≥30% magnitude in the preceding 12h
   - Duration since cycle peak was ≥4h
   - Post-cycle OI was in slow decline (not cascade, not growing)
2. **Classify each.** For each match, check the next 4h of price/OI action. Did it cascade down? Bounce? Drift? Record magnitude and timing of resolution.
3. **Minimum validation bar.** The fingerprint should produce a cascade within 4h on **≥70% of matches**. Below that it's noise.
4. **Negative-case inventory.** Specifically search for matches that BOUNCED. Understand what the bounce cases had that the cascade cases didn't. Without this, any apparent edge is unfalsified.
5. **Do not trade this fingerprint until steps 1–4 are complete.**

A quick-and-dirty pass can be built on `tests/357-mechanics-reader.mjs` (post-fix). Feed it historical timestamps and look for coins where the `MECHANIC DISCRIMINATION` output says `TRAPPED LONGS BLEEDING OUT`.

## Finding 2 — FR-Isolation Anti-Pattern (documented failure mode)

### The rule

**Any narrative mechanics read must be consistent with FR, OI direction, AND price motion simultaneously. If your proposed mechanic needs OI to be growing and OI is shrinking, the read is wrong. If your proposed mechanic needs price pressure and price is flat, the read is wrong.**

Reading one observable (typically FR) and pattern-matching to a template from memory produces systematic errors because the same FR signature maps to multiple opposing mechanics depending on OI direction and price motion.

### Ambiguous-signature table

Each row below is a numerical signature that by itself does not determine direction. The `disambiguator` column shows which additional observables resolve the ambiguity.

| FR | Ambiguous interpretation | Disambiguator → actual mechanic |
|---|---|---|
| Deep negative (≤−0.3%) | "shorts piling in → squeeze" OR "trapped longs bleeding out" | OI growing → squeeze; OI shrinking + price flat → trapped longs |
| Deep negative + price falling | "shorts trapping themselves" OR "shorts profiting from cascade" | OI growing → new shorts winning = continuation; OI shrinking → long liquidation in progress |
| Baseline (\|FR\| < 0.05%) + OI growing | "directional build forming" | If FR does not drift off zero during the build, BOTH sides are entering → disagreement market, not directional build |
| FR normalizing from extreme | "cycle exhausting" OR "settlement adjustment, will deepen again" | Multi-settlement trajectory needed; look at OI trend simultaneously |

All four rows are based on specific errors I made on 2026-04-14. Row 2 is ARIA. Row 1 (second interpretation) is RAVE. Row 3 is MYX. The fourth is a general principle.

### Why I kept falling for it

The template-matching approach is fast and feels grounded. "FR deep negative + deepening = Pure A1 multi-leg squeeze → buy pullbacks." It looks like mechanics reading but it's actually one-metric pattern matching with a mechanic-flavored label on top. The label does not replace the consistency check.

Once I'd labeled a coin "Pure A1," I stopped checking whether OI and price actually fit the template. I only went back and checked after the prediction failed.

### The QC discipline

Before declaring a direction from a snapshot:

1. Write the proposed mechanic in one sentence: "Operator is doing X because Y."
2. Ask: "If this mechanic is running, what should OI be doing?" Check if actual OI matches.
3. Ask: "If this mechanic is running, what should price be doing?" Check if actual price matches.
4. Ask: "If this mechanic is running, what should FR trajectory look like?" Check if actual FR matches.
5. If any of the three observables contradicts the proposed mechanic, **the read is wrong.** Do not weaken the mechanic to fit — pick a different mechanic or accept that the state is ambiguous.

This is a checklist against my own confirmation bias. It caught 0 of 3 errors today because I never ran it. If I had run it on RAVE, the "shorts piling in → squeeze" story would have died at step 2 (OI shrinking, not growing).

## Combined take-away

The RAVE miss was not a fundamental limitation of snapshot mechanics reading. It was a combination of a tool data bug and a missing QC step. Fixing the tool and applying cross-observable consistency catches both errors retrospectively (confirmed via fixed tool: RAVE retro-reads as `TRAPPED LONGS BLEEDING OUT`, ARIA as `LONG LIQUIDATION IN PROGRESS`, MYX as `AMBIGUOUS POST-CRASH DISAGREEMENT with bearish bias`).

Neither finding is an edge. The fingerprint is a sharp, testable hypothesis that may or may not survive a historical backtest. The anti-pattern is a QC rule that prevents a class of errors.

**Do not trade the fingerprint until it is out-of-sample validated on N≥30 historical cases with explicit negative-case inventory.**

## What was not added to the repo

- Tool fixes to `tests/357-mechanics-reader.mjs` live in the private research directory, not here. They enable the retro-validation but are not "findings."
- The predictions log (`data/predictions-log.jsonl`) with today's 4 predictions and their scoring is private working data, not research.
- Memory notes with QC discipline and Binance API quirks live in `~/.claude/.../memory/` and are operator-level tooling.
