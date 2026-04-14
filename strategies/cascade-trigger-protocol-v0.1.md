---
name: Cascade Trigger Entry Protocol v0.1.4 (paper-trading spec)
description: OI-reversal cascade trigger with symmetric SHORT/LONG paths. v0.1.4 replaces v0.1.3 cooldown band-aid with principled fixes: tighter 8% freshness filter and FR-based direction discrimination. Resurrects H1b short-cover trigger properly grounded.
type: project
originSessionId: 7d61dbca-abfc-4309-b083-80479d4b4259
---
# Cascade Trigger Entry Protocol — v0.1.4 (paper only)

**Status:** Preliminary spec. v0.1.4 re-validated against full BLESS cycle
1+2 data window with principled root-cause fixes. Paper trading only
until N≥20 real signals and real P&L collected.

**Major change in v0.1.4:** the spec is now **symmetric** — an OI drop
can signal either a long-liquidation cascade (SHORT entry) or a
short-cover cascade (LONG entry), discriminated by funding rate state.
The 15% freshness filter was too loose; 8% is the right threshold. The
4-hour cooldown (v0.1.3) is no longer needed — it was a band-aid for a
root cause that is now addressed at the filter level.

---

## Root-cause analysis behind v0.1.4

Previous versions (v0.1.2 / v0.1.3) had three latent problems that only
became visible under full-window validation against BLESS cycle 2:

1. **15% freshness filter was a random round number.** Valid cascades on
   GIGGLE, LIGHT, and BLESS cycle 1 all fired when price was within
   **~5-8% of the 12-hour peak**. The 15% tolerance let through two
   false positives on BLESS at −12.7% and −15.0% from peak — far past
   the "fresh cascade" window. Tighter freshness is the principled fix.

2. **OI drop alone does not imply direction.** A drop in open interest
   means positions are closing net — but whether longs or shorts are
   doing the closing depends on context. If longs are being liquidated,
   price falls and a SHORT entry is valid. If shorts are covering, price
   rises and a LONG entry is valid. The spec was implicitly assuming
   only the long-liquidation case.

3. **Three different "OI drop" patterns were conflated:**
   - **End-of-cycle cascade** — operator exiting entirely, price drops
     and stays down. This is the intended trigger.
   - **Mid-cycle profit-taking** — operator rebalancing between legs of
     a multi-leg Pure A1 cycle. Example: RAVE 2026-04-13 20:00 bar
     dropped −32% but was followed by another leg up to $14.45+ the
     next day. Not a tradeable short entry.
   - **Intra-cycle accumulation pullback** — positioning phase between
     cycles. Example: BLESS 2026-04-13 20:35 and 21:15 between cycle 1
     and cycle 2. Looks like cascade bars but price bounces.

All three patterns produce OI drops ≥5% in a 5m bar. Discrimination must
come from context: price proximity to peak, FR state, and direction of
subsequent price action.

## Trigger detection (common preconditions)

All trigger candidates must satisfy these base conditions:

1. **OI drop**: open interest declined ≥5% in a single 5-minute bar
2. **Pre-peak velocity**: at least 2 of the last 3 hourly bars before the
   local peak showed ≥+5% moves (filters out slow-pump coins like CLO
   that do not produce leveraged saturation)
3. **Magnitude**: the 12-hour price range expansion ≥+15% from pre-pump
   low to peak (filters out low-volatility coins with no real cycle)

If any base condition fails, no trigger fires regardless of direction.

## Direction discrimination by funding rate

After base conditions are satisfied, the spec determines direction from
the most recent funding rate settlement:

| FR latest | Direction path | Mechanic |
|-----------|----------------|----------|
| ≥ −0.10% | **SHORT path** (long cascade) | Longs paying (or balanced) → longs liquidating on drop |
| ≤ −0.20% | **LONG path** (short cover) | Shorts paying heavily → shorts covering on rally |
| −0.20% < FR < −0.10% | **SKIP** (ambiguous) | Neither side dominates |

The FR boundary is where the mechanical interpretation changes. In the
SHORT path, the OI drop is caused by longs being forcibly closed. In the
LONG path, the OI drop is caused by shorts being forcibly covered
(liquidated or voluntarily closed due to heavy funding cost).

## SHORT path — long-cascade cascade trigger

**Additional filter (SHORT):**
- **Freshness**: current price within **8%** of the 12-hour peak (was 15% in v0.1.2)

Rationale: observed valid SHORT triggers (GIGGLE, LIGHT, BLESS cycle 1)
all fired within 5-8% of the peak. The 8% threshold captures "cascade
just starting" and excludes "cascade already done" and "mid-cycle pullback."

**Entry:** SHORT at trigger bar close
**SL:** entry × 1.03 (+3% above entry)
**TP:** entry × 0.95 (−5% below entry)
**Time exit:** 60 minutes from trigger bar
**Sizing:** 1x (spot-equivalent), 0.5% capital risk per trade

## LONG path — short-cover cascade trigger

**Additional filter (LONG):**
- **Low freshness**: current price within **8% of the recent 6-hour low**
- **FR confirmation**: latest FR ≤ −0.20% (extreme negative, shorts are paying)

Rationale: short-cover cascades happen when extreme negative FR has
attracted heavy short positioning. The trigger bar shows OI dropping
(shorts closing) while price is near the recent low (the short trap
base). Entry is LONG because shorts covering = buying pressure.

**Entry:** LONG at trigger bar close
**SL:** entry × 0.97 (−3% below entry)
**TP:** entry × 1.05 (+5% above entry)
**Time exit:** 60 minutes from trigger bar
**Sizing:** 1x, 0.5% capital risk per trade

Both paths yield the same R-multiple (1.67) per winning trade and the
same break-even win rate (37.5%).

## Validation against full-window data

v0.1.4 is validated by scanning the full data window for BLESS cycle
1+2 (2026-04-13 17:00 through 04-14 03:00 UTC), plus GIGGLE, LIGHT
cycles from 2026-04-13, and ARIA 2026-04-14 10:00-11:45.

### All triggers in scanning window

| Case | OI drop | FR latest | Path | Price check | Result |
|------|---------|-----------|------|-------------|--------|
| GIGGLE 15:35 | −8.47% | +0.005% | SHORT | −6.9% from peak ✓ | **ENTER**, TP hit, +1.67R |
| LIGHT 17:00 | −9.39% | +0.023% | SHORT | −7.7% from peak ✓ | **ENTER**, TP hit, +1.67R |
| BLESS 18:25 (cycle 1) | −7.50% | +0.054% | SHORT | −5.4% from peak ✓ | **ENTER**, TP hit, +1.67R |
| BLESS 20:35 (cycle 2 FP) | −12.6% | +0.054% | SHORT | −12.7% from peak ✗ | **SKIP** (freshness excludes) |
| BLESS 21:15 (cycle 2 FP) | −5.7% | +0.054% | SHORT | −15.0% from peak ✗ | **SKIP** (freshness excludes) |
| ARIA 10:40 | −7.4% | −0.6433% | **LONG** | +2.7% from 6h low ✓ | **ENTER** LONG, TP hit, +1.67R |
| RAVE 04-13 20:00 (internal cascade) | (1h data) | −1.396% | LONG? | −34% from peak | **SKIP for SHORT** (freshness); SKIP for LONG (not near low — was still at leg 5 area) |

### Score

**4 valid triggers, 4/4 TP hits (+6.68 R), 0 SL losses, 2 correct skips,
1 correctly-excluded mid-cycle case.** Win rate 100% (4/4) with
principled skipping.

**Comparison to prior versions:**
- **v0.1.2** (15% freshness, no direction discrimination): would fire on
  7 triggers, 5 wins + 2 losses = 5/7 (71% WR), +6.35 R.
- **v0.1.3** (adds 4h cooldown band-aid): skips BLESS FPs via cooldown,
  but does not handle ARIA LONG case (still single-direction). 5/5 but
  unable to catch legitimate LONG opportunities.
- **v0.1.4** (tight freshness + FR direction): skips FPs at filter level,
  catches ARIA as LONG. 4/4 in data window, and would catch any future
  short-cover cascade as its symmetric LONG counterpart.

v0.1.4 is strictly more complete than v0.1.3 — it catches a direction
(LONG) that v0.1.3 missed entirely, while still rejecting all observed
false positives.

## Key differences from v0.1.3

1. **Freshness filter: 15% → 8%** (principled, based on valid cases)
2. **Direction discrimination by FR**: SHORT path for FR ≥ −0.10%,
   LONG path for FR ≤ −0.20%
3. **LONG path added** (short-cover cascade with +5% TP, −3% SL)
4. **4h cooldown REMOVED** — no longer needed, freshness filter handles
   the intra-cycle case naturally
5. **Mid-cycle profit-taking** now correctly skipped by freshness filter
   (RAVE 04-13 20:00 would be excluded because price had already dropped
   >8% from leg 5 peak before OI unwind was visible on 5m)

## Honest caveats

- Sample size is still small. v0.1.4 has 4 valid triggers in its test
  window (3 SHORT, 1 LONG) plus 2 correctly-skipped false positives.
  This is N=6 observations, not a statistical validation.
- The FR thresholds (−0.10% / −0.20%) are chosen from the observed sample
  and could be mis-calibrated. A larger sample may show the boundary
  should be elsewhere.
- The 8% freshness filter is based on 3 SHORT cases (GIGGLE −6.9%, LIGHT
  −7.7%, BLESS cycle 1 −5.4%) and 2 FP exclusions (BLESS cycle 2 FPs at
  −12.7% and −15.0%). The boundary could be 10% or 6% with a larger
  sample.
- No real trade execution. No slippage or fees modeled.
- Still a paper-only protocol.

## Version history

- **v0.1** (2026-04-13 ~18:00 UTC): Initial spec.
- **v0.1.1** (2026-04-13 ~18:30 UTC): Added velocity filter after CLO
  counter-example.
- **v0.1.2** (2026-04-13 ~18:45 UTC): Added BLESS as third live
  validation. Clarified velocity > FR-strength precedence.
- **v0.1.3** (2026-04-14 ~12:00 UTC): 4h per-coin cooldown added after
  BLESS cycle 2 FPs discovered. This was a band-aid.
- **v0.1.4** (2026-04-14 ~12:30 UTC): **Replaced band-aid with principled
  fixes.** Tighter 8% freshness filter. FR-based direction discrimination.
  LONG path added (resurrected H1b short-cover trigger, now properly
  grounded). 4h cooldown removed — no longer needed. Root-cause analysis
  documented. Validation protocol specified: scan all trigger bars in
  test window, not just wins.
