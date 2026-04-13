---
name: Operator Cluster Hypothesis
description: Same-founder coin families share pump mechanics but rotate attention asymmetrically. RAVE/DEEP/WAL Coinbase co-listing case. Explains similar cycle signatures across "unrelated" coins.
type: project
originSessionId: 7d61dbca-abfc-4309-b083-80479d4b4259
---
# Operator Cluster Hypothesis — 2026-04-13 ~19:00 UTC

## Origin

User observation: RaveDAO (RAVE), DeepBook (DEEP), and Walrus (WAL) were
co-listed on Coinbase on 2026-02-11 — a single listing announcement for all
three. User's hypothesis: **same founder / operator → same playbook → that
explains why their cycles look similar**.

## Evidence at 2026-04-13 19:00 UTC

| Coin | Price | 24h% | Vol(M) | State |
|------|-------|------|--------|-------|
| RAVE | $11.68 | +188% | $4,071M | Active Pure A1 multi-leg squeeze, 3rd peak today |
| DEEP | $0.0279 | +2.2% | $1M | Dormant — no active cycle |
| WAL | $0.0684 | −1.9% | $2M | Dormant — no active cycle |

All three share:
- Common co-listing event (2026-02-11 Coinbase)
- Presumed common backer/founder (per user observation; DeepBook and Walrus
  are both confirmed Mysten Labs / Sui ecosystem, RaveDAO association
  uncertain but user asserts it)
- Similar structural profile when they do pump (per user)

**But only RAVE is currently active.** DEEP and WAL are in idle state.

## Hypothesis

**"Operator cluster" pattern:** a small team / founder / fund manages
multiple coins with a consistent playbook (same FR manipulation pattern,
same OI cycle, same phase structure). They rotate attention asymmetrically
because simultaneous pumping across 3+ coins would require 3x+ capital and
3x+ risk of attention dilution.

**Consequences:**

1. **Structural signature reproducibility.** A cycle's type (Pure A1 /
   HYBRID / B-OI / etc.) is more predictable on a coin whose cluster-
   sisters have demonstrated consistent patterns in prior cycles.

2. **Asymmetric timing.** When one sister is in active cycle, the others
   are in idle or accumulation. You cannot trade all three at once because
   only one is live at a time.

3. **Sequential cycles.** After RAVE completes its current cycle, DEEP or
   WAL is more likely to start a cycle in the following days/weeks. The
   operator cluster rotation creates a pipeline of cycles rather than
   simultaneous events.

4. **Correlated risk on multi-leg.** If you're long a sister coin during
   its active cycle and the operator shifts attention to another sister,
   your position becomes distressed. Cross-sister exit correlation matters.

5. **Framework implication.** Cluster detection could be a meta-filter on
   top of the FR taxonomy: "is this coin part of a known operator cluster
   with an established cycle template?" If yes → higher confidence in
   classification; if no → standard descriptive taxonomy.

## Implications for existing framework

The N=33 taxonomy might actually reflect a **small number of operator
playbooks** rather than a population of independent cycles. If there are
only ~5-10 active operators on Binance shitcoin futures, and each operator
has a preferred playbook (Pure A1, HYBRID big-move, B1 climax, etc.), then
the "types" we've identified are operator signatures, not statistical
regularities of a random market.

This would mean:
- The 39% Pure A / 32% HYBRID / 18% B1 / 9% other distribution reflects
  the OPERATOR POPULATION composition, not some underlying market physics
- Changes in operator activity or new operators entering the market could
  shift the distribution
- Some operators might rotate between playbooks (an operator whose usual
  play is Pure A1 might occasionally execute a HYBRID)

This also explains why the framework forward-validated so well (29/29
direction calls) despite being based on a biased sample — if the sample is
a catalog of operator playbooks rather than coin cycles, then new cycles
from the same operators will indeed match the catalog.

## Open questions from this hypothesis

1. **Cluster mapping:** how do we identify which coins share operators?
   Sources: co-listings, founding team, VC backer, social media
   announcements, token distribution patterns.

2. **Rotation cadence:** how long after RAVE's current cycle ends should
   we expect DEEP or WAL to start? Days? Weeks?

3. **Is cluster-sharing predictive?** If RAVE's cycle is known to be Pure
   A1 multi-leg, does DEEP's next cycle default to Pure A1 multi-leg? Or
   do operators vary the playbook per coin to avoid detection?

4. **Cross-cluster correlation with BTC.** Do operator clusters time their
   activations to BTC conditions (risk-on environments) or operate
   independently?

5. **What's the minimum N to verify a cluster?** Need ≥2 observed cycles
   on at least one cluster member before claiming the template applies.

## Connection to cascade universality finding

Earlier today I found that cascade trigger (liquidation OI reversal) is
universal across cycle types — not specific to B-OI. This is CONSISTENT
with the operator cluster hypothesis:

- If operators run a small set of playbooks that all end in leveraged-long
  saturation + cascade, then the cascade signature should appear universally
  regardless of whether the pre-peak phase was squeeze (Pure A) or FOMO (B1)
- The cascade is the operator's EXIT, and all operators exit the same way
  (forced liquidation of the final longs)
- What varies across playbooks is the PRE-peak buildup (squeeze vs FOMO vs
  hybrid), but the POST-peak exit mechanic converges

This gives cascade-trigger-protocol-v0.1 stronger theoretical grounding:
the universal cascade signature is the universal operator exit, not a
market-wide phenomenon.

## What to do with this

**Short term:** observe DEEP and WAL for the next 1-7 days. If either starts
a cycle that matches RAVE's Pure A1 template, the cluster hypothesis gains
evidence. If they diverge (different playbook), the hypothesis weakens.

**Medium term:** build a cluster registry: a list of known operator
clusters with their member coins and observed playbooks. Use this as an
input to the framework, not a replacement.

**Long term:** if clusters are real and rotation is systematic, front-run
sister cycles by pre-positioning when a cluster member peaks and operator
attention is about to shift. This is speculative and high-risk.

## Honest caveats

- N=1 observation (RAVE cluster) with 2 dormant sisters. Not a verified
  pattern, just a hypothesis.
- User's "same founder" claim unverified at source. DeepBook/Walrus =
  Mysten Labs confirmed; RaveDAO association asserted by user.
- "Rotation" pattern requires data from multiple past cycles across
  sister coins — not collected yet.
- The hypothesis is compatible with but does not require the FR taxonomy.
  Could coexist as a meta-filter or replace the taxonomy's "type" concept.
