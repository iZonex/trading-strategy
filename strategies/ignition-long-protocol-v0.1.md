---
name: Ignition Long Entry Protocol v0.1 (paper-trading spec)
description: Preliminary entry/stop/exit rules for fresh-ignition long entries. Narrow scope — only first strong bar out of silence with confirmed OI. N=2 verified wins.
type: project
originSessionId: 7d61dbca-abfc-4309-b083-80479d4b4259
---
# Ignition Long Entry Protocol — v0.1 (paper only)

**Status:** Preliminary spec, narrow scope. N=2 verified wins (BULLA, AIOT).
Explicitly excludes Pure A1 refresh cycles, mid-squeeze continuation, and
HYBRID Phase 1 late-stage longs. Use only for **fresh-ignition-from-silence**
setups. Paper trading only until N≥20.

This is the LONG-side companion to `spec_cascade_trigger_protocol_v0_1.md`.
Cascade trigger is the clean SHORT signal at cycle end; ignition entry is
the clean LONG signal at cycle start. The space between (refresh cycles,
Phase 1 mid-squeeze) has its own mechanics and is out of scope for v0.1.

## Purpose

Capture the ignition bar where a pump transitions from silent accumulation
to explosive run, entering long with confirmation that open interest is
loading in sync with price. Based on the observation that BULLA (04-09) and
AIOT (04-11) both showed a single "first strong bar" with simultaneous
massive OI growth, and both continued up immediately without pullback.

## Trigger rule

A long ignition trigger fires when ALL of the following are true on a 1-hour
bar:

1. **Strong price bar**: Close-to-close ≥ +5% in the current 1h bar
2. **OI confirmation**: Open interest on the bar grew ≥ +10% per hour.
   This is stricter than the cascade trigger's +5% OI velocity filter and
   is the key discriminator between fresh ignition (works) and late-cycle
   entries (fails — see INX/TRADOOR in validation table).
3. **Silence preceding**: None of the prior 3 hourly bars were ≥ +5%.
   This is the "first strong bar from silence" filter. It excludes entries
   in continuation bars where the ignition has already happened and price
   is already elevated.
4. **Magnitude context**: 12-hour price range < +20% before the trigger bar.
   (Prevents entering a coin that is already mid-pump.)

## Entry rule

- **Direction**: LONG
- **Entry price**: Market order at close of the trigger 1h bar
- **Golden cases**:
  - BULLA: trigger 2026-04-09 01:00 UTC, bar +6.76% with OI +17.7%, entry $0.0184
  - AIOT: trigger 2026-04-11 15:00 UTC, bar +9.54% with OI +28.3%, entry $0.0601

**Why trigger bar close, not next bar open:** The confirmation of OI
saturation happens on the trigger bar itself. Waiting a full hour for the
next bar risks missing the move entirely on fast setups.

## Stop loss rule

- **Stop price**: Entry × 0.95 (−5% below entry)
- **Rationale**: The long-side is noisier than the short-side. Cascade
  shorts benefit from the fact that once a liquidation begins, price rarely
  retests the high in the short term. Long ignitions can and do retest the
  pre-ignition low during the first hour, especially when the trigger
  happens on 1-hour resolution (vs 5-minute). A 5% stop sits below typical
  intra-hour volatility and above the pre-ignition accumulation zone.

**Why wider than cascade's 3% stop:** Long moves up have larger upside per
R because cycles often run for hours, but also larger initial volatility.
The cascade trigger's 3% stop works because the cascade is a one-direction
event; the ignition is a start that needs to survive initial volatility.

## Take profit rule

- **TP price**: Entry × 1.10 (+10% above entry)
- **Rationale**: Observed move sizes from ignition in the golden cases were
  very large (BULLA +24% to peak, AIOT +73% to peak). A +10% TP captures
  the early portion of the move and exits before the subsequent refresh or
  distribution phases complicate the picture. In both verified cases, +10%
  was hit within 1-3 hours of entry without any drawdown below entry.

- **R-multiple**: 10% reward / 5% risk = **2.0 R per winning trade**
- **Break-even win rate**: 1 / (1 + 2.0) = **33.3%**

**Alternative considered (trailing stop):** Capture more of the large long
moves by trailing 50% of peak profit. Would have captured BULLA +15-20% and
AIOT +35-40% instead of +10%. Deferred until more validation cases confirm
the exit mechanic — fixed TP is more robust at small N.

## Time-based exit

- **Max hold**: 4 hours from trigger bar
- **Action**: If neither TP nor SL hit by 4 hours, market-close
- **Rationale**: Both golden cases hit TP within 1-3 hours. The long side
  has more time variability than the short side (cascades complete in 30-60
  min, ignitions can take 1-6 hours), but beyond 4 hours the initial
  ignition thesis has either worked or failed. Holding longer reintroduces
  distribution-phase risk without the ignition edge.

**Note on funding costs:** 4 hours is typically within one funding settlement
window (4h is the most common interval on the classified coins). Expect to
pay at most one funding settlement, usually less than 0.05% at the R values
involved.

## Position sizing

- **Risk per trade**: 0.5% of capital
- **Position notional**: risk / stop_distance = 0.5% / 5% = **10% of capital**
- **Leverage**: 1x spot-size. Leverage is not needed because the move is
  expected to be +10% on a 10% position (+1% portfolio return per win).

**On a $1000 paper account:**
- Risk per trade: $5
- Position notional: $100
- Win: +$10 (2.0 R)
- Loss: −$5 (1 R)
- EV at 50% win rate: +$2.50/trade

## Validated cases

### Triggered cases (rule fires, trade taken)

| Case | Session | Trigger bar | Pre-bar OI | Entry | TP | Hit | R realized |
|------|---------|-------------|------------|-------|----|----|------------|
| BULLA | 2026-04-09 | 01:00 +6.76% | +17.7% | $0.0184 | $0.0202 | bar 04:00 | +2.0 R |
| AIOT | 2026-04-11 | 15:00 +9.54% | +28.3% | $0.0601 | $0.0661 | bar 16:00 | +2.0 R |

Both cases: entry at bar close, never drew down below entry, TP hit cleanly,
max profits went far beyond TP (+31% BULLA, +73% AIOT — a fixed TP leaves
significant upside on the table but is more robust at small N).

### Correctly skipped (rule does NOT fire — good)

| Case | Why rejected | What actually happened |
|------|--------------|------------------------|
| **INX 22:00 (04-12)** | First +5% bar was 20:00 (+13.78%), OI only +2.6% fails ≥10% threshold | Entry would have been followed by pullback chop, SL hit. Filter correct. |
| **INX 23:00** | OI +6.7% fails threshold | Same as above. Filter correct. |
| **INX 04:00** | Preceded by bar at 03:00 +3.86% (not ≥5%, technically passes silence) — but OI only +6.4% fails threshold | Filter correct — weak OI means weak commitment. |
| **TRADOOR 02:00 (04-13)** | First +5% bar at 02:00 with OI −1.3% fails threshold | Filter correct. |
| **TRADOOR 12:00** | Preceded by 11:00 +26.65% (not silence) | Filter correct — trigger is not "first from silence." |
| **CLO** | No bar ≥+5% anywhere in cycle | Correctly excluded — slow pump, no ignition. |
| **RIVER** | No bar ≥+5% | Correctly excluded. |

### Out of scope

| Scenario | Why out of scope |
|----------|------------------|
| **Pure A1 refresh cycles** (RAVE multi-leg) | Trigger fires only on first ignition from silence. Refresh cycles happen mid-trend and need a different rule (pullback-bounce). |
| **Pure A1 extreme-FR entry** | When FR is at cap and interval shortened, the mechanical signal is different — mechanical funding transfer dominates. Covered by a future squeeze-entry protocol. |
| **HYBRID Phase 2 long** | Phase 2 is distribution phase; long entries fade. Not this protocol's target. |
| **Post-cascade bounce long** | Different pattern (bounce after liquidation). Covered by oi-flow-shitcoin-trading.md M5 Bounce Long. |

## Summary

- **Verified: 2/2 wins at +2.0 R each**
- **Correctly skipped: 5 cases (INX×3, TRADOOR×2, CLO, RIVER)**
- **Sample size: tiny**. This is a narrow pattern, not a generic long rule.
- **Break-even WR: 33.3%**, currently observing 100% but statistically meaningless.
- **Known blind spots**: rule only fires on fresh-ignition-from-silence. Does
  not handle refresh cycles, mid-squeeze continuations, late Phase 1 HYBRID
  entries, post-cascade bounces, squeeze-mechanical entries.

## Paper trading protocol

For the next 10 LONG signals from a scanner implementing this trigger:

1. Record trigger bar, pre-bar OI, entry price, SL, TP before any post-trigger bar forms
2. Watch hourly bars until exit (TP, SL, or 4h time)
3. Record exit, realized R
4. Do not adjust rules on the fly; wait for all 10 before refining

Targets after 10 paper trades:
- Win rate ≥55% (above 33.3% break-even)
- At least 2 cases must fail (to test SL behavior)
- If win rate <55%: narrow scope may be too narrow or OI threshold miscalibrated

## Relationship to cascade trigger spec

| Aspect | Cascade Trigger (SHORT) | Ignition Long (THIS) |
|--------|-------------------------|----------------------|
| Trigger event | OI reversal during active pump | First strong bar from silence with OI loading |
| Mechanical basis | Liquidation cascade | Position loading before run |
| Data resolution needed | 5-min OI | 1-hour OI (5-min optional) |
| Holding period | 30-60 min | 1-4 hours |
| SL distance | 3% | 5% |
| TP distance | 5% | 10% |
| R per win | +1.67 | +2.0 |
| Break-even WR | 37.5% | 33.3% |
| Scope | Any type with cascade mechanics | Fresh-ignition only |

The two specs are complementary ends of the same cycle. A coin can produce
both trades within 4-6 hours of each other: long at ignition, short at
cascade. Running both protocols in parallel is the natural deployment.

## Version history

- **v0.1** (2026-04-13 ~19:00 UTC): Initial narrow-scope spec. Based on 2
  historical golden cases (BULLA, AIOT). Trigger rule requires "first strong
  bar from silence with ≥+10% OI growth." Validated against INX and TRADOOR
  as negative tests (correctly skipped). Explicitly out of scope: refresh
  cycles, mid-squeeze, post-cascade bounce.
