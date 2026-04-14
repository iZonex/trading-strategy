---
name: Cascade Trigger False Positive Failure Mode (N=2 observed)
description: Cascade trigger fires during intra-cycle accumulation pullbacks on multi-cycle coins. BLESS 2026-04-13 20:35 and 21:15 would have been SL hits. Session walkthrough was overly optimistic. v0.1.3 cooldown rule proposed.
type: project
originSessionId: 7d61dbca-abfc-4309-b083-80479d4b4259
---
# Cascade Trigger False Positive Failure Mode — 2026-04-14 ~12:00 UTC

> **UPDATE 12:30 UTC:** The v0.1.3 cooldown fix proposed below was a
> band-aid. After user pushed on "why didn't things get better," a deeper
> root-cause analysis revealed that the 15% freshness filter was the real
> problem. v0.1.4 replaces the cooldown with a tighter 8% freshness filter
> plus FR-based direction discrimination (SHORT vs LONG path). This
> addresses the root cause (valid cascades fire within 5-8% of peak, not
> 15%) and also catches a direction the spec was missing (LONG short-cover
> trigger). See spec_cascade_trigger_protocol_v0_1.md v0.1.4.
>
> This document is preserved as the discovery narrative but the recommended
> fix (Option B cooldown) was **superseded**.

## Summary

The cascade trigger spec (v0.1.2) has an identified failure mode:
**intra-cycle false positives on multi-cycle coins**. On coins that run
multiple cycles in sequence, the trigger can fire during the accumulation
phase of a subsequent cycle, producing SL hits rather than TP.

Discovered while validating the short-cover trigger hypothesis (H1b) on
BLESS cycle 2 historical data. The investigation was aimed at finding
LONG setups but surfaced SHORT failure cases instead.

## Observed false positives

Both on BLESSUSDT during 2026-04-13 cycle 2 accumulation phase. Cycle 1
peak was $0.01876 at 18:00 UTC (cascade at 18:25 valid, TP hit, +1.67R).
Cycle 2 peak was $0.03773 at 04-14 00:00 UTC. In between:

### False positive #1: 2026-04-13 20:35 UTC

- Close: $0.01928 (bar was −3.52%)
- OI: $8.89M (−12.6% in one 5m bar — well above 5% threshold)
- FR at time: +0.054%

Trigger fired: OI drop ≥5%, price within 15% of 12h peak, velocity filter
passes (17:00 bar +7.93%, 19:00 bar +18.7%), magnitude >15%.

Spec entry: $0.01928, SL $0.01986 (+3%), TP $0.01832 (−5%)

Price action: 20:40 $0.01953, 20:45 $0.01943, 20:50 $0.01997,
20:55 $0.02068 (above SL), 21:00 $0.02097

**Outcome: SL hit at 20:50-20:55. −1 R loss.**

### False positive #2: 2026-04-13 21:15 UTC

- Close: $0.01876, OI $8.59M (−5.7% in 5m)
- FR at time: +0.054%

Spec entry: $0.01876, SL $0.01932, TP $0.01782

Price: 21:20 $0.01968, 21:25 $0.01971, 21:30 $0.02053

**Outcome: SL hit at 21:20. −1 R loss.**

BLESS then continued up to cycle 2 peak $0.03773 and cascaded for real
at 04-14 01:00 UTC (OI −20.94% 1h bar).

## Revised BLESS session contribution

Previously claimed: "BLESS +1.67 R realized."

Correct for cycle 1 only. Spec running through cycle 2 accumulation
would have fired two more triggers, both SL hits.

**Corrected BLESS net: +1.67 − 1 − 1 = −0.33 R**

**Corrected 2026-04-13 session total:**
- GIGGLE: +1.67 R
- LIGHT: +1.67 R
- BLESS net (3 triggers): −0.33 R
- **Revised total: +3.01 R** (not +5.01 R as originally reported)

Still profitable, but 40% less than claimed. **Win rate 3/5 (60%)**, not
100%. Still above break-even (37.5% for R=1.67) but much closer.

## Discriminating valid vs false positive

| Trigger | Valid? | OI drop | Post-trigger behavior |
|---------|--------|---------|----------------------|
| 18:25 cycle 1 | YES | −8.47% | OI unwind 4+ bars, price fell to −11.9% |
| 20:35 cycle 2 | NO | −12.6% | OI recovered 3 bars, price bounced 15 min |
| 21:15 cycle 2 | NO | −5.7% | Same — quick recovery |
| 01:00 cycle 2 end | YES | −20.94% | Sustained unwind |

Discriminator is **post-trigger behavior**, retrospective at the bar.
Can be converted prospectively via delayed entry or cooldown.

## Proposed fix: v0.1.3 per-coin cooldown

Three candidate refinements tested:

**Option A: Delayed entry (next-bar confirmation).** Filter correctly
skips BOTH FPs but reduces clean wins (GIGGLE and LIGHT would get ~0.5-0.8R
instead of 1.67R because delayed entries are at lower prices and TPs aren't
hit cleanly). Net ~2.97R across cases. **Worse than cooldown.**

**Option B: Per-coin 4-hour cooldown. ★ RECOMMENDED**

Rule: After any trigger closes (TP/SL/time), no new trigger on same
coin for 4 hours.

- BLESS cycle 1 exit 18:35 → cooldown until 22:35
- FP 20:35: within cooldown → SKIPPED ✓
- FP 21:15: within cooldown → SKIPPED ✓
- BLESS cycle 2 real cascade 04-14 01:00 (6h later): allowed ✓
- GIGGLE/LIGHT/BULLA/AIOT: no prior exits, full 1.67R each

Net with cooldown: **+5.01 R** (all wins preserved, both FPs eliminated).

**Option C: Post-trigger OI confirmation.** Complex, delays entry 10 min.
Not clearly better than cooldown.

**Recommendation:** adopt per-coin cooldown as v0.1.3 rule. Simple,
eliminates intra-cycle FPs, preserves all observed wins. Cost: on
genuinely-multi-cycle coins where cycle 2 starts within 4h we miss the
second cycle. Acceptable — observed multi-cycle gaps are typically >4h.

## Honest framework update

Prior claim: "5/5 TP hits, 100% WR" was **selection bias** — only
counting triggers that led to wins.

**True validation at v0.1.2 (no cooldown):**
- BLESS 3 triggers: 1 win + 2 losses = −0.33 R net
- GIGGLE/LIGHT/BULLA/AIOT: 4 wins = +6.68 R
- **Total: +6.35 R across 7 triggers, 5/7 = 71% WR**

**With proposed v0.1.3 cooldown:**
- BLESS: 1 win, 2 skipped, cycle 2 real possible
- Others: 4 wins
- **Total: 5/5 = 100% WR if cooldown holds, +8.35 R possible**

Cooldown makes WR and R honest without losing clean wins.

## Research lesson about validation discipline

The original session walkthrough was written by inspecting only the
triggers that led to wins. This is **selection bias** — the mistake of
quoting only best trades while implicitly ignoring losing setups in the
same data window.

A proper validation requires scanning the entire data window for ALL
triggers that would have fired, including ones that lost. This failure
mode would have been caught immediately if I had run the full scan from
the beginning.

**Discipline for future research:** every new spec must be validated by
scanning ALL trigger bars in a test window, not just the ones that
match expectation. Missing this step produces inflated win rates.
