---
name: Session Walk-Forward Simulation 2026-04-13
description: Hypothetical P&L if cascade-trigger-protocol + ignition-long-protocol were executed on live signals during 2026-04-13 session. Paper-trade walkthrough.
type: project
originSessionId: 7d61dbca-abfc-4309-b083-80479d4b4259
---
# Session Walk-Forward — 2026-04-13

**What this is:** A hypothetical paper-trade walkthrough of the cascade
trigger protocol (v0.1.2) and ignition long protocol (v0.1) executed against
the actual signals observed during the 2026-04-13 research session. Used to
estimate:
- Number of trades per session
- Cumulative R-multiple
- Win rate
- Per-capital return

**What this is NOT:** Real P&L. No positions were actually opened. No
slippage or fees are modeled. The signals are taken from what was observed,
not from a real-time scanner running against the protocol in perfect
execution. Treat as an upper bound.

---

## Assumptions

- **Starting capital**: $1,000 (paper)
- **Risk per trade**: 0.5% = $5
- **Fees ignored**
- **Slippage ignored**
- **Parallel positions**: allowed (no correlation adjustment)
- **Entry timing**: assume perfect fill at trigger bar close

---

## Live Cascade Trigger signals (SHORT)

| Trigger time | Coin | Entry | SL | TP | Actual exit | Time to exit | R realized |
|--------------|------|-------|----|----|----|--------------|------------|
| 15:35 UTC | GIGGLE | $40.85 | $42.07 | $38.81 | TP hit ~16:00 | ~25 min | **+1.67 R** |
| 17:00 UTC | LIGHT | $0.1836 | $0.1891 | $0.1744 | TP hit ~17:30 | ~30 min | **+1.67 R** |
| 18:25 UTC | BLESS | $0.01774 | $0.01827 | $0.01685 | TP hit ~18:35 | ~10 min | **+1.67 R** |

**Cascade short result: 3 trades, 3 wins, +5.01 R total**

## Live Ignition Long signals (LONG)

The ignition long spec fires on "first strong 1h bar from silence with
OI ≥+10%." Looking at the day's activity:

| Candidate | First strong bar | OI on bar | Passed filter? | Notes |
|-----------|------------------|-----------|----------------|-------|
| BULLA (today's mini-cycle) | 18:00 +4.80% | +8.53% | ✗ | OI below 10% threshold, near miss |
| BLESS | 17:55 +5.29% | −0.39% | ✗ | OI slightly negative |
| WET | (earlier session, pre-peak) | — | — | Needed, not observed in real-time |
| IRYS | (earlier session, pre-peak) | — | — | Needed, not observed in real-time |
| TRADOOR | 12:00 bar late in cycle | +23.7% | ✗ | Preceded by strong bar (not silence) |

**Ignition long result: 0 signals fired during the session (no qualifying
setups observed in my scan windows).**

This is an important finding. The ignition long pattern is **rarer** than
the cascade pattern on this day. In a full-session scanner running
continuously, more ignition opportunities would likely be caught on the
1-3 hours BEFORE the cascades fire (BLESS ignition ≈ 17:55 was close but
failed the OI threshold).

---

## Session totals

| Protocol | Trades | Wins | Losses | Time exits | R total |
|----------|--------|------|--------|------------|---------|
| Cascade short | 3 | 3 | 0 | 0 | +5.01 |
| Ignition long | 0 | — | — | — | 0 |
| **Combined** | **3** | **3** | **0** | **0** | **+5.01** |

**Per-capital return on $1,000 paper account:** +$25.05 (2.5% of capital
from 0.5% risk per trade × 1.67 R × 3 wins).

**Session duration:** approximately 3 hours (first trigger at 15:35, last
exit ~18:35 UTC).

---

## What this tells us

### Strengths
1. **Three clean cascade signals** in one 3-hour window on a single day.
   Live frequency is meaningful — not 1 per week, not 1 per day, but ~1 per
   hour in an active market.
2. **100% win rate** on cascade protocol across all live signals. Too small
   a sample to be statistically meaningful, but consistent with the N=5
   validation sample (all TP hits).
3. **Fast execution** — longest time to TP was 30 minutes, shortest 10
   minutes. The cascade mechanic resolves quickly.
4. **Paper return ~2.5% per session** on fixed 0.5% risk. Extrapolated,
   one similar session per week produces 2.5% weekly return, or ~130% annual
   at constant frequency. This is a fantasy number because of selection bias
   and non-independence, but it bounds the upside.

### Weaknesses
1. **Zero long signals fired.** The ignition pattern is narrower and today's
   session did not present a qualifying setup during my scan windows. A
   continuous real-time scanner would likely catch more, but the claim that
   "LONG side works symmetrically" is unsupported at this sample.
2. **Day selection bias.** April 13 was a session with multiple active
   cycles (RAVE ATH, several HYBRID distributions, etc.). A quiet day may
   produce zero signals.
3. **No losses observed.** Without at least one stop-loss case, the SL rule
   is untested in live conditions. Expected behavior is known from historical
   (never hit in 5 validated cases) but not from failures.
4. **Monitoring cost ignored.** I spent approximately 5 hours of active
   research to catch 3 trades. In production this would need to be automated.

### Quiet cases (nothing happened)

| Coin | Protocol | Why skipped |
|------|----------|-------------|
| DUSK | both | No velocity, no OI signal — correctly skipped |
| RIVER | both | Slow fade, OI drop below threshold — correctly skipped |
| CLO | both | Slow pump (velocity filter), no cascade — correctly skipped |
| IR | cascade | HYBRID small-move, explicit skip per framework |
| GIGGLE / CATI / FIGHT (initial classification) | — | Originally "B2 skip", later reclassified as B-OI. Framework adapted mid-session. |

---

## Estimated one-day edge (if sustained)

Under the **unrealistic** assumption that the 3/3 pattern holds over time:

- Expected R per cascade signal: +1.67 (100% WR)
- Expected signals per session: 3 (today's count)
- Sessions per week: ~5 (active markets)
- R per week: +25 (100% WR assumption)
- Capital return per week: 12.5% (0.5% risk × 25 R)

Under a **realistic** assumption of 55% WR (break-even is 37.5%):

- Per-trade EV: 0.55 × (+1.67) + 0.45 × (−1) = +0.469 R
- Per-session EV (3 signals): +1.41 R
- Per-week (5 sessions × 3): +21 R = 1.41 × 5 = 7.04 R/week
- Capital return: 3.5% per week = 180% annualized

Under a **pessimistic** assumption of 40% WR (barely above break-even):

- Per-trade EV: 0.40 × 1.67 + 0.60 × (−1) = +0.068 R
- Per-week: +1.02 R
- Capital return: 0.5% per week = 26% annualized

**The interesting finding:** even at a pessimistic 40% WR, the 1.67 R-per-win
is high enough that the protocol has positive expected value. The break-even
is 37.5%. The session sample shows 100% but is not generalizable.

## Next steps

1. **Let scanner 353 run in background** over multiple sessions. Record every
   signal with its outcome. Target: 20 cascade signals across 5 days.
2. **Wait for first real loss.** A 0-loss sample is statistically
   meaningless. Until a SL is hit, we don't know if SL behavior matches
   expectations.
3. **Run paper trades with actual timestamp discipline.** Today's report is
   post-hoc. Tomorrow, record trigger time, entry, SL, TP BEFORE any
   subsequent bar forms. Only that kind of discipline tests the real edge.
4. **Compare against oi-flow M3 Pump Exhaustion SHORT** on same signals.
   Their M3 fires on similar conditions (>25% pump + OI drop + volume dying).
   If both fire on same coin, do they agree on direction?
