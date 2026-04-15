---
name: Barrel Framework V1 — Honest OOS (2026-04-15)
description: Out-of-sample test of barrel framework on 15 fresh coins × 30 days shows WR 63% / EV +0.08% per trade, well below the 71.4% breakeven required for +2% TP / −5% SL. Memory's "N=21 WR 90%" claim not reproducible from code; earlier derivation-set MFE analysis showed 91% IS which collapses on OOS. Not tradeable with uniform rules. Contradicts prior "validated" claim.
type: research
---

# Barrel Framework V1 — Honest OOS (2026-04-15)

## Summary

Ran a fresh out-of-sample test of the Barrel Framework V1 hypothesis (silence → OI loading → volume/range trigger → LONG) on **15 coins NOT in the derivation set** over 30 days. Result: **63% WR on N=60 trades, EV +0.08% per trade** — below the 71.4% breakeven required for the +2% TP / −5% SL exit structure. Net effectively zero after fees.

This contradicts the memory file `research_barrel_framework_v1.md` claim of "WR 90%, PF 19, Validated". The claim is not reproducible from the backtest code that accompanied the research (`tests/307-barrel-pattern-backtest.mjs`) and collapses to breakeven-negative on fresh coins.

## Derivation-set baseline check (test 307 as-is)

Ran `tests/307-barrel-pattern-backtest.mjs` on its cached 5m data for the 5 original coins (SWARMS, BULLA, SIREN, RED, RIVER). Output from the built-in random baseline section:

| Coin | N | Total PnL | P95 random | Beats P95? |
|---|---|---|---|---|
| SWARMSUSDT | 46 | +61.6% | +57.9% | **YES** (marginal) |
| BULLAUSDT | 37 | +95.0% | +102.8% | NO |
| SIRENUSDT | 35 | +76.3% | +102.8% | NO |
| REDUSDT | 24 | +20.4% | +39.7% | NO |
| RIVERUSDT | 25 | **−14.4%** | +52.8% | NO (negative total) |

Only 1 of 5 derivation coins beats random P95 at all. The aggregate "90% WR N=21 PF 19" claim in memory does not correspond to any parameter set in the code. Individual per-coin claims in memory partially match (SWARMS STRICT LONG ~80%) but diverge strongly on others (BULLA memory 90% vs STRICT LONG 62.5%).

## The MFE analysis that motivated the fixed-TP exit

Before running OOS, parsed all STRICT LONG trades from test 307 (N=57 across 5 coins). Key finding:

**0 of 20 losses had negative MFE.** Every "losing" trade went up before coming back:

| MFE bucket | Losses in bucket | % of losses |
|---|---|---|
| < 0 (wrong direction) | 0 | 0% |
| 0 – 2% | 5 | 25% |
| 2 – 5% | 9 | 45% |
| 5 – 10% | 5 | 25% |
| > 10% | 1 | 5% |

- Wins avg MFE: 10.2%, avg PnL@4h: 14.8%
- Losses avg MFE: 4.2%, avg PnL@4h: −2.5%

Simulated fixed TP on the same N=57:

| Exit rule | WR (trades ever reaching TP threshold) |
|---|---|
| +2% TP | 91% (52/57) |
| +3% TP | 74% (42/57) |
| +5% TP | 49% (28/57) |

Interpretation at time of analysis: trigger catches real moves (100% of losses had the direction right), but the existing exit (4h fixed or "trail activate +5% giveback 50%") can't capture the small-move wins. Tight fixed TP at +2% looked like it reproduced the memory's claimed high WR. This became the hypothesis to OOS-test.

## OOS test (tests/362-barrel-oos-fixed-tp.mjs)

**Universe:** 15 coins NOT in derivation set — MYX, APR, BLESS, COAI, FIGHT, GIGGLE, LIGHT, ENJ, AIO, WET, IRYS, HOLO, TRADOOR, AIN, AKE. All active TRADING-status USDT-M perps. Deliberately excluded SWARMS/BULLA/SIREN/RED/RIVER/KERNEL/ZEC/ENA/WIF/ARIA/PEPE/TAO/NOM/FARTCOIN/JOE.

**Rules:** STRICT parameters from test 307 exactly — silence 24h, 50%+ quiet hours (range <3%), OI growth ≥10%, trigger volume ≥5× avg of prior 100 5m bars AND range ≥3× avg. LONG-only (green trigger bar).

**Exit:** fixed +2% TP / −5% SL / 4h (48 5m bars) timeout. Conservative intrabar priority: if SL and TP both touched in same bar, assume SL first.

**Window:** 30 days, 5m resolution, fetched live with pagination. OI from 4h period (500 bars = 83 days coverage, aligned to 5m bars).

## Results

**Aggregate N=60 trades across 15 coins:**

| Metric | Value |
|---|---|
| TP hits (+2%) | 38 |
| SL hits (−5%) | 12 |
| Timeouts | 10 |
| WR (pnl > 0) | 63% |
| Total PnL | +4.8% |
| EV per trade | +0.08% |

**Breakeven WR with +2% TP / −5% SL = 71.4%.** Actual 63% is 8 points below. After typical taker fees (2 × 0.04% = 0.08% round trip) and realistic slippage on thin alts, EV per trade is zero-to-slightly-negative.

### Per-coin breakdown

| Coin | N | WR | Total | EV/trade |
|---|---|---|---|---|
| ENJ | 5 | 100% | +10.0% | +2.00% |
| AIO | 4 | 100% | +8.0% | +2.00% |
| WET | 1 | 100% | +2.0% | +2.00% |
| AIN | 1 | 100% | +2.0% | +2.00% |
| COAI | 4 | 75% | +5.6% | +1.41% |
| IRYS | 8 | 75% | +2.0% | +0.25% |
| LIGHT | 3 | 67% | +3.3% | +1.11% |
| TRADOOR | 5 | 60% | +1.2% | +0.24% |
| BLESS | 8 | 63% | −2.3% | −0.28% |
| APR | 3 | 67% | −1.0% | −0.33% |
| AKE | 3 | 67% | −1.0% | −0.33% |
| FIGHT | 6 | 67% | −2.0% | −0.33% |
| MYX | 3 | 33% | −5.6% | −1.88% |
| HOLO | 3 | 33% | −6.6% | −2.19% |
| GIGGLE | 3 | 0% | −10.8% | −3.61% |

8 of 15 coins profitable, 7 of 15 negative. Per-coin outcome approximately coin-flip.

The largest positive contributions (ENJ +10, AIO +8, COAI +5.6) carry most of the +4.8% total; the negative tail (GIGGLE −10.8, HOLO −6.6, MYX −5.6) offsets most of it. Without a generic filter that predicts which coins will behave like ENJ vs GIGGLE, the aggregate is noise.

## Why the edge collapsed IS → OOS

**Derivation-set IS: ~85-91% with +2% TP (simulated from MFE).**
**Fresh-coin OOS: 63% with same rules.**

The ~25 percentage-point drop is not marginal degradation; it's the standard signature of overfitting. Likely mechanisms:

1. **Coin selection bias.** The original 14 derivation coins were chosen because the researcher had observed barrel-like patterns on them. Coins where barrel doesn't happen or doesn't work were never in the derivation set.
2. **Period selection bias.** The 90-day derivation window was a specific market regime (Jan-Apr 2026 alt pump cycle). Different 30-day window on different coins may have different regime.
3. **Per-coin mechanics differ.** Memory itself noted in expanded test (47 coins) that WR drops to 60%. OOS 63% is consistent with that finding, not with the 90% "best filter" claim.
4. **"Best filter" was post-hoc.** The N=21 WR 90% filter was selected after inspecting which combinations of params/coins produced the cleanest number. That's the same selection-bias disease documented elsewhere in this research corpus today.

## What the trigger DOES do

Not pure null. The trigger detects something real:

- ENJ: 5/5 TP hit, 100% WR
- AIO: 4/4 TP hit, 100% WR
- IRYS: 6/8 TP hit
- Overall directional accuracy 63% (> random 50%)

The mechanic — "silence then OI loading then volume+range spike with green close" — **does** precede real moves. But the move is coin-specific in both magnitude and reliability. On some coins a 63%-ish hit rate at +2% TP is enough; on others (GIGGLE, HOLO, MYX) the same setup produces false starts or reversals.

The uniform-rule approach cannot exploit this without a per-coin filter, and per-coin filters found on this same OOS data would be post-hoc fits.

## Broader pattern: 3 for 3 today

This is the third hypothesis rigorously verified in two days:

| Hypothesis | Memory claim | Honest test result | Status |
|---|---|---|---|
| Cascade Trigger Protocol v0.1.x | 5/5 = 100% WR | 41% WR (N=17 honest backtest 2026-04-14) | Marginal/no edge |
| Trapped Longs Fingerprint | "validated on RAVE cascade" | RAVE prediction was actually correct; post-mortem was fiction | Retracted 2026-04-14 |
| Barrel Framework V1 | WR 90% N=21 validated | 63% WR OOS on N=60 fresh coins | Breakeven-negative |

Three different hypotheses, three different flavors of the same failure mode: derivation-set confidence that doesn't survive honest forward testing. The research corpus has systemic selection bias in its "validated" claims.

## What to do next

1. **Do not trade barrel with uniform rules.** The 63% OOS rate is below breakeven for +2%/−5% R:R and likely below breakeven for any symmetric variant.
2. **Do not tune per-coin on this data.** Fitting "ENJ and AIO work, MYX and GIGGLE don't" as a rule is overfitting to this OOS window. Any per-coin filter would need a fresh OOS again.
3. **Do not iterate exit strategies on this data** for the same reason.
4. **Update memory.** Memory's barrel entry should be marked as "derivation-set-only, OOS 63%, contradicts 90% claim." Done in this session.
5. **Consider pivoting entirely.** The 3-for-3 failure rate on memory hypotheses suggests that continuing to test individual hypotheses from the corpus is a sunk cost. Better direction: forward-tracking discipline with live predictions log, accumulating honest track record from zero.

## Files

- `tests/362-barrel-oos-fixed-tp.mjs` — OOS backtest script, STRICT params + fixed TP/SL
- `data/barrel-oos-trades-v2.jsonl` — full trade list from the OOS run (60 trades)
- `tests/307-barrel-pattern-backtest.mjs` — original derivation backtest (unchanged)

## Bottom line

Barrel Framework V1 is not a tradeable edge on fresh coins with the rules as specified in either memory or test 307 code. The "validated" status claimed in memory does not hold. The trigger captures real signal but not enough to be tradeable with uniform rules.

Nothing in this document should be used as a trading reference.
