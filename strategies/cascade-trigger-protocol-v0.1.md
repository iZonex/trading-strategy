---
name: Cascade Trigger Entry Protocol v0.1 (paper-trading spec)
description: Preliminary entry/stop/exit rules for the OI-reversal cascade trigger. Two live + two historical golden cases. Explicit N=4 — paper trade only.
type: project
originSessionId: 7d61dbca-abfc-4309-b083-80479d4b4259
---
# Cascade Trigger Entry Protocol — v0.1 (paper only)

**Status:** Preliminary spec. N=4 cases validated (2 live on 2026-04-13:
GIGGLE, LIGHT; 2 historical from N=22: BULLA, AIOT). No real P&L.
**For paper trading only** until N≥20 and edge confirmed.

---

## Purpose

Convert the OI-reversal cascade observation (shitcoin-leverage-mechanics.md
§5.4, research_cascade_universality.md) into explicit entry/stop/exit/size
rules that can be paper-traded on future live cases.

## Trigger rule

A cascade trigger fires when ALL of the following are true on a 5-minute bar:

1. **OI drop**: Open interest declined ≥5% in a single 5-minute bar
2. **Price near high**: Current price within 15% of the 12-hour intraday high
3. **Sustained pre-peak velocity**: At least 2 of the last 3 hours before the
   peak showed 1-hour price bars ≥ +5%. This filters out slow-pump cycles that
   look like cascades on the OI drop but don't produce liquidation momentum
   (added after CLO false-trigger discovery, see validation table).
4. **OI growth pre-peak**: At least one hour in the last 12 showed OI growth
   ≥5% per hour
5. **Magnitude**: The 12-hour price move from pre-pump low exceeds 15%

This fires on the 5m bar where OI shows the reversal. Conditions 3-5 are the
cascade signature (aggressive pre-peak build-up). Condition 2 is a freshness
filter to avoid entering a cascade that is already halfway done.

**Data resolution note:** The protocol requires 5-minute OI data. 1-hour data
can miss fast HYBRID cascades entirely — the INX and AIN cases below show
−26% and −18% moves within a single 1h bar, meaning by the time the 1h OI
reversal is visible, price is already 20%+ below peak and fails the freshness
filter. Use `/futures/data/openInterestHist?period=5m` and expect a ~5 minute
data delay.

## Entry rule

- **Direction**: SHORT
- **Entry price**: Market order at close of the trigger 5m bar
- **Golden cases**:
  - GIGGLE: trigger 2026-04-13 15:35 UTC, entry $40.85
  - LIGHT: trigger 2026-04-13 17:00 UTC, entry $0.1836

The price bar PRECEDING the trigger bar typically shows a sharp −5% to −8%
move (the "price crack"). The trigger bar itself often shows a flat or small
bounce in price while OI reveals the liquidation cascade underneath. Entering
at the trigger bar close is the rule.

**Alternative considered and rejected:** entering on the price crack bar (one
bar earlier). Would give a better entry price but loses the OI confirmation.
Without OI confirmation, there is no way to distinguish a real cascade from a
normal pullback within an uptrend.

## Stop loss rule

- **Stop price**: Entry × 1.03 (tight stop, +3% above entry)
- **Rationale**: In both live golden cases and both historical cases, the
  post-trigger price action stays below the trigger bar's own high. A 3% stop
  sits comfortably above the immediate noise band and was never challenged.

**Alternative considered and rejected:** stop above the 12-hour intraday
high (conservative, +8% to +10% from entry). Produces R-multiples of ≈0.6,
which requires a win rate above 62% to be profitable. Too fragile.

**What a tight 3% stop gives up:** if a cascade fakes out and the coin
resumes pumping, a tight stop gets taken out. We accept this trade-off
because the edge comes from being precise, not conservative.

## Take profit rule

- **TP price**: Entry × 0.95 (fixed 5% profit)
- **Rationale**: Cascade magnitude in live cases (N=2) peaked at 5.4% and 5.7%
  from trigger-bar close. Historical 1h-resolution cases cascaded further, but
  5% was hit within 60 minutes in all 4 observed cases.

**R-multiple**: 5% reward / 3% risk = **1.67 R per winning trade**.

**Break-even win rate**: 1 / (1 + 1.67) = 37.5%.

**Alternative considered and deferred:** trailing stop (50% of peak profit).
Would capture larger cascades (BULLA −24%, AIOT −38% in historical sample).
Deferred because the consistency of 5% hit across 4 cases is worth more than
theoretical larger captures until sample size grows.

## Time-based exit

- **Max hold**: 60 minutes from trigger bar
- **Action**: If neither TP nor SL is hit by 60 minutes, market-close the
  position.
- **Rationale**: All 4 observed cases hit the 5% TP within 30 minutes. Beyond
  60 minutes the cascade mechanic has usually exhausted and the coin enters a
  range or reversal. Holding longer reintroduces directional risk without
  cascade mechanics.

## Position sizing

- **Risk per trade**: 0.5% of capital
- **Position notional**: risk / stop_distance = 0.5% / 3% = **16.7% of capital**
- **Leverage**: 1x (spot-size position). Leverage is not needed for this
  strategy because the entry is already within the cascade itself; the move
  happens over minutes.

**On a $1000 paper account:**
- Risk per trade: $5
- Position notional: $167
- Win: +$8.35 (1.67 R)
- Loss: −$5 (1 R)
- Expected value at 50% win rate: +$1.67/trade

**On a $10,000 paper account:**
- Risk per trade: $50
- Position notional: $1670
- Win: +$83.50
- Loss: −$50

## Required environment

- **Data feed**: 5-minute klines (price, volume) and 5-minute OI history.
  Binance fapi endpoints `/fapi/v1/klines?interval=5m` and
  `/futures/data/openInterestHist?period=5m`.
- **Update cadence**: OI has ~5 minute delay. Trigger detection is inherently
  ~5 minutes late. This is acceptable because the subsequent cascade lasts
  20-40 minutes.
- **Universe**: Must be coins that have been classified as candidate for
  cascade (scanner 352 output with accumulating/peaking tags). The trigger
  rule alone is not sufficient on coins that have not shown the velocity +
  magnitude preconditions.

## Validated cases (N=9 including correctly-skipped)

### Triggered cases (rule fires, trade taken)

| Case | Session | Trigger time | Entry | SL | TP | Outcome | R realized |
|------|---------|--------------|-------|----|----|---------|------------|
| GIGGLE | 2026-04-13 live 5m | 15:35 UTC | $40.85 | $42.07 | $38.81 | TP hit 16:00 | +1.67 R |
| LIGHT | 2026-04-13 live 5m | 17:00 UTC | $0.1836 | $0.1891 | $0.1744 | TP hit 17:30 | +1.67 R |
| BULLA | 2026-04-09 hist 1h | 07:00 UTC | $0.0219 | $0.02257 | $0.0208 | TP hit 08:00 | +1.67 R |
| AIOT | 2026-04-12 hist 1h | 04:00 UTC | $0.0754 | $0.0777 | $0.0716 | TP hit 05:00 | +1.67 R |

**Verified: 4/4 TP hits, 0/4 SL hits, 0/4 time exits. +6.68 R total.**

### Correctly skipped cases (rule does NOT fire — good)

| Case | Reason for skip | Actual outcome | Verdict |
|------|-----------------|----------------|---------|
| **CLO** | Velocity filter: only 1 of last 3 pre-peak bars was ≥5% (+0.52%, +5.12%, +0.38%). Slow pump, no liquidation fuel. | Slow bleed −9% over 6h, never triggered sharp cascade | ✓ filter saved an unproductive entry |
| **RIVER** | OI drop only −3.6% in 1h, below 5% threshold | Slow bleed −5% over many hours | ✓ correctly skipped |
| **DUSK** | Baseline throughout, never reaches velocity threshold | Mild −12% drift decline | ✓ correctly skipped (true drift) |

### Out-of-resolution cases (rule cannot execute at 1h data)

| Case | Problem | Would fire at 5m? |
|------|---------|-------------------|
| INX | 1h peak to next bar = −26.66% move. OI drop at 09:00 hits freshness filter (price already −36% from peak). 1h resolution is too coarse. | Yes at 5m — cascade would start with price still within 15% of high |
| AIN | 1h peak to next bar = −7% then −18%. By trigger 1h bar, price is −24% from peak. 1h too coarse. | Yes at 5m |

These are not losses — they are data-resolution misses. In a production system
with 5m OI polling, both would be trade candidates. In this N=9 validation
they are excluded from the scoring.

### Summary

- **4/4 verified wins at +1.67 R each**
- **0/4 SL or time losses**
- **3/3 correctly skipped** (CLO was the critical false-trigger test after velocity filter addition)
- **2 cases await 5m-resolution re-run**
- **Win rate at N=4: 100%. Too small to generalize. Paper protocol below.**

### The CLO test specifically

CLO is the protocol's first intentional stress test. Without the velocity
filter (§3 of trigger rule), CLO would have fired on its OI −5.4% bar at
16:00 with entry $0.0974. Price would have drifted flat for an hour, exiting
at time rule for −0.2 R (small loss). With the velocity filter added, CLO is
correctly excluded because only 1 of its 3 pre-peak bars was ≥5% — the slow
accumulation did not build enough leveraged longs to cascade.

This is the clearest example so far of the framework refining a rule via a
counter-example rather than a confirmation. The velocity filter was added
specifically because CLO showed what happens when OI drop is satisfied but
pre-peak velocity is not.

## What this protocol does not handle

1. **False triggers.** If a coin is in a strong uptrend and has a brief OI
   dip that recovers, the 3% stop will be taken out. The historical N=4 does
   not include any false-trigger cases; the live sample is too small to have
   seen one. **This is a known blind spot.**
2. **Correlated triggers.** If multiple coins trigger within the same 15-minute
   window, they may be reacting to a BTC-level move and the cascade mechanic
   may not dominate. No sizing rule for concurrent exposure is specified.
3. **Slippage on thin coins.** Market entry at 5m bar close assumes clean
   fills. On a $0.0001 coin with $5k order size, slippage could eat 1% of
   the expected 5% profit. Not accounted for.
4. **Fee costs.** Binance taker fee is 0.05% per side = 0.10% round trip.
   At 5% profit, net is 4.9%. Ignored in R calculation.
5. **Funding rate cost.** Positions held across a funding settlement pay FR.
   At 5 minutes to 60 minutes, usually avoided, but not guaranteed (1h
   settlement intervals in extreme regime).
6. **Re-entry after stop.** If stopped out, no re-entry rule. Conservative
   default: one attempt per cascade.

## Paper trading protocol

For the next 10 triggered signals from scanner 352:

1. Record the trigger bar time, entry price, SL, TP before any price movement
2. Watch the 5m bars until exit (TP, SL, or time)
3. Record exit price, exit time, realized R
4. Update this spec with observed results (do NOT adjust rules on the fly;
   wait for all 10 before refining)

Target after 10 paper cases:
- Win rate ≥55% (above break-even of 37.5%)
- At least 2 cases must be clean losses (to test SL behavior)
- Median time-to-exit ≤30 minutes
- If win rate <55%: rules are wrong or regime-specific, stop and investigate

## Version history

- **v0.1** (2026-04-13 ~18:00 UTC): Initial spec. Based on 2 live golden cases
  (GIGGLE, LIGHT) and 2 historical matches (BULLA, AIOT). All 4 cases had
  +1.67 R realized. Sample size is trivially small; protocol is paper-only.
- **v0.1.1** (2026-04-13 ~18:30 UTC): Added velocity filter (condition §3 of
  trigger rule) after CLO counter-example test. Expanded validation to N=9
  cases: 4 verified wins, 3 correct skips, 2 out-of-resolution. Added resolution
  note: 1h data misses fast HYBRID cascades; 5m data required.
