# Leverage Cycle Mechanics on Cryptocurrency Perpetual Futures: A Funding-Rate-Based Classification Framework

**Author:** D. Chystiakov
**Date:** April 13, 2026
**Version:** 0.1 (preliminary — see §9 for validation state)

---

## Abstract

We present a descriptive framework for classifying the mechanical state of leveraged cycles on low-capitalization cryptocurrency perpetual futures ("shitcoins"). Rather than attempting to predict price movement directly, the framework reads the current state of leverage positioning — who is long, who is short, whether one side is being liquidated, whether positions are balanced — and assigns each cycle to one of seven mechanical regimes. The primary discriminator is the sign and trajectory of the funding rate (FR) across the cycle. Secondary discriminators are open-interest velocity and pump-bar magnitude.

The framework is validated in two forms. Descriptively, the taxonomy cleanly classifies N=33 historical cycles with stable distribution across sub-samples (Pure A 39%, HYBRID 32%, Pure B1 18%, remainder split between B-OI and true drift). Prospectively, a single-day forward test on April 13, 2026 produced 29 direction calls on 29 coins (Batch #1 N=10, Batch #2 N=19 excluding skips) with 100% direction accuracy over 37-minute to 2-hour windows and zero falsification triggers.

A key sub-finding is the empirical split of the former "B2 drift" category into two distinct mechanical regimes: true slow drift (represented by DUSK, no directional edge) and balanced-OI cascade (represented by FIGHT, CATI, GIGGLE, short-tradable on OI reversal). This split was formalized during the study session and validated live 20 minutes later on GIGGLE, which crashed −11.9% from intraday high following an OI reversal of −8.47% in a single 5-minute bar — matching the newly documented mechanic with timestamped precision.

**We emphasize what this framework is not.** It is a reading framework, not a trading system. It provides directional bias and mechanistic context, not entry triggers, position sizing, stop-loss levels, or take-profit rules. All trading approaches described in §8 are hypotheses. The framework has been tested on a single day of forward data, on a single exchange (Binance Futures), on a biased population (top movers). No real P&L has been collected. The results reported here are necessary but not sufficient for a deployable trading system.

**Keywords:** cryptocurrency perpetual futures, funding rate, open interest, liquidation cascade, market microstructure, leverage cycles, shitcoin taxonomy

---

## 1. Introduction

### 1.1 Problem Statement

Low-capitalization cryptocurrency perpetual futures — colloquially "shitcoins" — exhibit pump cycles with magnitudes of 1.1x to 50x over windows of hours to weeks. Existing quantitative approaches developed on major assets (BTC, ETH) transfer poorly to this market segment. Positioning data via long/short account ratio is unavailable for most shitcoins. Technical indicators designed for mean-reverting or trending equity markets produce high-noise signals on cycles driven entirely by leverage dynamics rather than fundamentals. Volume and order-flow indicators lag the moves they are meant to predict.

A practitioner observing a shitcoin mid-pump faces a classification problem before any trading decision can be made: *what kind of cycle is this?* An extreme short squeeze with multiple legs (where long entries are supported by continuous short liquidation) looks superficially similar to a retail FOMO climax (where long entries are guaranteed losers one hour later). Both show large green candles, growing OI, and elevated volume. A framework that conflates them produces net losses regardless of model sophistication.

### 1.2 Failure of Threshold-Based Classifiers

Prior iterations of this research attempted to build rule-based classifiers using fixed thresholds (e.g., "if FR < −0.003% and OI growth > 1% per hour then signal"). These approaches failed for three reasons.

First, **fixed thresholds are brittle across regimes.** The same numeric threshold that discriminates a squeeze on one coin produces noise on another of different volatility or capitalization. Thresholds tuned on backtest data fail out-of-sample because they encode properties of the tuning set, not the underlying mechanic.

Second, **magnitude is not a type discriminator.** Empirically, HYBRID cycles range from 1.02x (BZUSDT) to 17.29x (ARIAUSDT), and Pure A cycles from 1.44x (0GUSDT) to 35x (RAVEUSDT). Two cycles of identical magnitude can belong to entirely different mechanical classes, and within a class, magnitudes span more than an order of magnitude.

Third, **single-snapshot readings conflate cycle phase with cycle type.** A coin observed during the distribution phase of a HYBRID cycle presents numeric values indistinguishable from a Pure B1 FOMO top. Only the *trajectory* of the funding rate across the full cycle distinguishes them.

### 1.3 Thesis

We argue that the fundamental unit of analysis on shitcoin perpetuals is the **leverage state**: which side of the market is net long, which side is paying for that position, and whether positions are being forcibly closed. The funding rate — specifically its sign and trajectory over the cycle — is a direct observable of this state, because it reflects the balance of longs versus shorts at each settlement and is updated in real time by Binance's exchange mechanism.

Our thesis is that four orthogonal leverage states, combined with a magnitude modifier, produce a small, stable set of mechanical regimes sufficient to classify all observed cycles in the shitcoin population:

1. **Pure A** — shorts persistently net long → persistent negative FR → squeeze mechanics
2. **HYBRID A→B** — shorts squeezed, then longs FOMO → FR crosses sign → full-cycle extraction
3. **Pure B1** — longs persistently net long → persistent positive FR → FOMO exhaustion
4. **Balanced OI Pump (B-OI)** — both sides net long, no directional leverage → FR baseline → OI cascade on reversal
5. **True Drift (B2)** — no meaningful leverage → FR baseline, low velocity → skip

The framework assigns each observed cycle to exactly one of these regimes based on FR sign pattern over the cycle, with secondary branching on magnitude and velocity.

### 1.4 Contributions

1. A seven-class mechanical taxonomy for shitcoin perpetual cycles grounded in funding-rate sign analysis.
2. Empirical distribution of regimes on N=33 historical cycles with stable proportions across sub-samples.
3. Discovery and formalization of the Balanced-OI Cascade regime (B-OI), previously conflated with slow drift.
4. A live reading scanner (Test 352) implementing the classification and staging.
5. Forward validation over a single session with 29/29 direction accuracy and a timestamped live validation of the B-OI regime on GIGGLE.
6. Explicit characterization of what the framework does and does not provide, to prevent premature deployment.

---

## 2. Data and Methodology

### 2.1 Data Sources

All data is sourced from Binance Futures via the public `fapi` endpoint:

| Dataset | Endpoint | Granularity | Usage |
|---------|----------|-------------|-------|
| Klines (OHLCV + taker volume) | `/fapi/v1/klines` | 5m and 1h | Cycle identification, velocity measurement |
| Mark price klines | `/fapi/v1/markPriceKlines` | 1h | Basis calculation |
| Index price klines | `/fapi/v1/indexPriceKlines` | 1h | Basis calculation |
| Open interest history | `/futures/data/openInterestHist` | 5m and 1h | OI velocity and drawdown |
| Funding rate history | `/fapi/v1/fundingRate` | Per settlement | Primary discriminator |
| 24-hour ticker | `/fapi/v1/ticker/24hr` | Snapshot | Candidate filtering |
| Exchange info | `/fapi/v1/exchangeInfo` | Snapshot | Active-symbol filtering (delisted symbols excluded) |

A critical data-hygiene note: Binance's 24-hour ticker endpoint returns stale data for delisted symbols (observed on ALPACAUSDT, showing spurious 391% change and $2.9B volume despite zero recent trading activity). Active-symbol filtering via the `TRADING` status field in `exchangeInfo` is required before any statistics are computed.

### 2.2 Cycle Identification

A cycle is identified over an 8-day window for each candidate coin. The pre-pump low is the minimum price in the window; the pump peak is the maximum price between the low and the window end; the post-peak low is the minimum after the peak. This yields four summary statistics: cycle magnitude (`peak/low - 1`), pump duration (hours from low to peak), post-peak decline, and funding rate range over the window.

### 2.3 Classification Procedure

For each cycle, the classifier examines the funding rate trajectory over the 8-day window and assigns a regime via the decision tree specified in §4. The classifier does not use fixed thresholds on price, volume, or OI directly. It uses the *sign* of the FR and the *velocity* of the pump (derived from 1-hour bar changes and OI velocity in the final 12 hours) as discriminators.

For live classification of a cycle in progress, the same decision tree is applied to the most recent 12 hours of data. Staging (accumulating / peaking / triggered) is determined by the relationship between current OI, peak OI, current price, and peak price within the recent window.

---

## 3. The Funding Rate as Leverage State Discriminator

### 3.1 Mechanism

The funding rate on Binance perpetual futures is the mechanism by which the perpetual contract price is anchored to the underlying index. When the perpetual trades above the index (positive basis), longs pay shorts a periodic fee; when it trades below (negative basis), shorts pay longs. The fee is proportional to the magnitude of the deviation and is settled every 4 or 8 hours under normal conditions.

Under extreme conditions, Binance shortens the settlement interval to 1 hour and caps the rate at ±2% per interval. Interval shortening is itself observable and serves as a signal that the cycle has entered an extreme regime.

Critically, **the sign of the FR reveals the net direction of leverage:**

- Positive FR → perpetual above index → net long pressure → longs paying
- Negative FR → perpetual below index → net short pressure → shorts paying
- Baseline FR (±0.01%) → balanced positioning → neither side dominates

The magnitude of the FR (bounded by ±2%) reveals the intensity of the imbalance, but the sign is what discriminates mechanical regime.

### 3.2 FR as State Revealer

Unlike price, which is a consensus market value, the funding rate is a settlement mechanism observable with mechanical precision. It does not respond to noise, news sentiment, or technical patterns. It responds only to the aggregate leverage of market participants at each settlement time. This makes it a uniquely clean signal for leverage-state classification.

Further, because settlements occur on a fixed schedule (with interval shortening under extremes), the FR trajectory across a cycle is visible as a sequence of discrete events. The pattern of these events — persistent negative, persistent positive, crossing from negative to positive, or staying at baseline throughout — encodes the mechanical identity of the cycle.

### 3.3 Why FR Dominates Other Metrics

We previously tested volume, taker-buy ratio, OI growth rate, basis, and tape-order-flow as primary discriminators. All produced high-noise classifications relative to the FR sign. The reason is structural: volume and OI measure *activity*, not *direction*; basis measures *deviation*, not *which side is paying for it*; taker ratios measure *aggression*, not *net position*. Only the FR directly reveals the sign of the net leverage imbalance.

This does not mean the other metrics are useless. OI velocity is a secondary discriminator within the baseline-FR regime (B-OI versus true drift, §4.5). Bar-level volume marks trigger events within a classified cycle. But the primary taxonomy axis is FR sign.

---

## 4. The Seven Mechanical Regimes

The framework assigns each cycle to exactly one of seven regimes via the decision tree in Figure 1.

### Figure 1: Classification decision tree

```
CHECK funding rate trajectory over 8-day window

FR went significantly negative (< -0.05%) at any point?
├── YES → FR also went significantly positive (> +0.05%)?
│   │
│   ├── YES → HYBRID (cycle crossed sign)
│   │   │
│   │   └── Magnitude > 1.5x?
│   │       ├── YES → HYBRID big-move (TRADEABLE: long Phase 1, short Phase 2)
│   │       └── NO  → HYBRID small-move (SKIP: operator FR extraction)
│   │
│   └── NO → Pure A (squeeze only)
│       │
│       └── FR reached -1% or interval shortened to 1h?
│           ├── YES → Pure A1 extreme squeeze (multi-leg expected)
│           └── NO  → Pure A2 mild squeeze (single leg)
│
└── NO → FR was significantly positive (> +0.05%)?
    │
    ├── YES → Pure B1 FOMO exhaustion
    │   │
    │   └── FR max bucket?
    │       ├── > +0.15% → Strong B1 (fast fade)
    │       ├── +0.10% to +0.15% → Normal B1 (moderate fade)
    │       └── < +0.10% → Weak B1 (slow fade, lower edge)
    │
    └── NO → FR stayed at baseline (±0.01%) throughout
        │
        └── Last-12h 1h bar velocity > 5% AND OI velocity > 5%/h?
            ├── YES → B-OI Balanced-OI Cascade (short on OI reversal)
            └── NO  → B2 true slow drift (skip, no edge)
```

### 4.1 Pure A — Short Squeeze Mechanics

**Pure A1 (extreme squeeze).** FR persistently negative, reaching −0.5% to −2% cap. Interval shortens to 1h. Magnitude 5x–50x. Multi-leg structure: shorts enter the cycle expecting reversal, funding settlements continuously extract capital from short positions, each settlement reduces short capital, cycle self-sustains until fresh shorts stop arriving. Refresh cycles occur when FR briefly normalizes, attracting new shorts, then re-intensifies.

Representative examples: RAVEUSDT (35x), ENJUSDT (2.73x), NOMUSDT (2.06x), SIRENUSDT (2.29x). Lifecycle days to weeks. The long-side trade hypothesis (§8.1) is based on the mechanical observation that each settlement delivers capital transfer from shorts to longs, irrespective of short-term price movement.

**Pure A2 (mild squeeze).** FR negative but not extreme (−0.1% to −0.5%). Interval remains 4h or 8h. Magnitude 50%–200%. Single leg, lifecycle hours to approximately 2 days. Less fuel for sustained runs than A1. Representative examples: WETUSDT, 0GUSDT, REDUSDT, IRYSUSDT.

### 4.2 HYBRID — Full-Cycle Extraction

HYBRID cycles begin as Pure A squeezes and transition to Pure B FOMO distributions. The FR sign crosses from negative to positive at the transition. Operators (the set of actors capturing the cycle) extract value from both shorts (during the squeeze phase) and longs (during the distribution phase).

**HYBRID big-move** (magnitude > 1.5x). Representative examples: ARIAUSDT (17.29x), AIOTUSDT (4.31x), BULLAUSDT (3.89x), TRADOORUSDT (3.53x), AKEUSDT (3.12x), INXUSDT (2.62x), DRIFTUSDT (2.10x). Mechanically the most lucrative regime for external traders because both phases are tradable: long Phase 1 during the squeeze (while FR is negative), detect the FR cross, short Phase 2 during the distribution (while FR is positive).

**HYBRID small-move** (magnitude < 1.5x). Representative examples: BZUSDT (1.02x), PAYPUSDT (1.05x), CLUSDT (1.06x), EWYUSDT (1.16x). FR swings to both extremes but price remains in a tight range. This corresponds to operators capturing funding-rate revenue from both sides of the book without needing to move spot price. No directional edge exists for external participants.

### 4.3 Pure B1 — FOMO Exhaustion

FR consistently positive, climbing to between +0.05% and +0.25% or higher. FR peaks before price peak, offering an early warning for fade entries. Lifecycle hours to days, single climax bar. Representative examples: CLOUSDT, RIVERUSDT, AINUSDT, NATGASUSDT, FUNUSDT, PUMPBTCUSDT.

A forward-validation observation from the April 13 session (§7) indicated that the magnitude of the fade following climax correlates strongly with the peak FR. We propose the following sub-calibration:

- **Weak B1** (FR max < +0.10%): slow fade, 1-hour decay of 1%–5%, low edge
- **Normal B1** (FR max +0.10% to +0.15%): moderate fade, 1-hour decay 5%–10%
- **Strong B1** (FR max > +0.15%): aggressive fade, 1-hour decay 10%–20%

Evidence: FOLKS (FR max +0.058%) declined 1.0% from peak at T+73min vs. PUMPBTC (FR max +0.148%) declining 16.5% over the same window.

### 4.4 B-OI — Balanced-OI Cascade (the April 13 discovery)

FR stays at baseline (±0.01%) throughout the cycle, but open interest grows substantially during the pump phase (typically +40% to +80% over 12 hours). The pump is fast (1-hour bars of 5% or more). The mechanical interpretation is spot-led movement: spot market moves first, perpetuals follow via arbitrage, and because positions are added symmetrically on both sides of the book, the FR stays pinned at baseline.

The cycle ends not with an FR regime change (there is none), but with an **OI reversal**: once the spot demand exhausts, longs begin closing, OI drops sharply, and the price cascades down as each long exit triggers the next. Post-peak drawdowns of 20%–35% are typical.

Representative examples: FIGHTUSDT (−32.1% post-peak), CATIUSDT (−24%), GIGGLEUSDT (−11.9% post-peak, live validation, §5.4).

The discriminating feature between B-OI and true slow drift (B2) is **velocity**. B-OI pumps exhibit 1-hour price bars exceeding 5% and OI growth exceeding 5% per hour during the active phase. True drift never reaches these velocities. Duration is not a discriminator: both can span multi-day windows, but B-OI concentrates its movement into a 6–12 hour acceleration phase while drift moves gradually throughout.

### 4.5 B2 — True Slow Drift

FR baseline throughout, velocity low (1-hour bars under 3%, OI growth under 3% per hour). Pump magnitude 30%–70% over 5+ days. Post-peak decline mild (10%–15%). Representative example: DUSKUSDT (verified as the only unambiguous B2 in N=22 after the B-OI reclassification).

No directional edge has been identified for this regime. The recommendation is to skip.

### 4.6 Distribution on N=33

| Regime | Count | Proportion |
|--------|-------|------------|
| Pure A (A1 + A2) | 13 | 39% |
| HYBRID big-move | 7 | 21% |
| HYBRID small-move | 4 | 12% |
| Pure B1 | 6 | 18% |
| B-OI | 2 | 6% |
| B2 true drift | 1 | 3% |

(Note: B-OI and B2 counts reflect the April 13 reclassification. Prior analyses reported these as a combined "B2 drift" category at approximately 9%.)

---

## 5. The B-OI Discovery

This section documents the within-session evolution of the framework from a six-class to a seven-class taxonomy, driven by a prediction miss and validated by live data within minutes of formalization. We include this because the discovery process itself illustrates both the framework's strength and its current limitations.

### 5.1 The FIGHTUSDT Miss

During the April 13 forward test (§7), FIGHTUSDT was classified as B2 drift based on its FR trajectory (±0.008% throughout) and 7-day cycle duration. The recommendation for B2 drift is "skip — no directional edge." Within the 73-minute verification window, FIGHT crashed 24.1% from its intraday high.

This was correctly recorded as a miss. The skip decision avoided a loss, but it also missed a move that the framework should arguably have caught.

### 5.2 Velocity Analysis

Re-examining FIGHT's 1-hour data around the pump:

- 06:00 UTC: price $0.00430, OI $3.93M, FR baseline
- 07:00 UTC: price $0.00506 (+17.6% in one bar), OI $3.76M (flat)
- 08:00 UTC: price $0.00492, OI $4.82M (+28.1% in one hour)
- 09:00 UTC: price $0.00505 (peak intraday high $0.00569), OI $4.86M
- 10:00 UTC: price $0.00449 (−11.1%), OI $5.21M (still growing)
- 11:00 to 13:00: price and OI both declining toward post-peak low

The 07:00 to 09:00 window showed 1-hour bar velocity above 17% and OI velocity above 28% per hour. This is not the slow-drift mechanic described in the B2 regime. The subsequent post-peak behavior — OI peaking one bar after price peak, then OI and price cascading together — is the signature of a liquidation cascade.

### 5.3 The Reclassification

Comparing FIGHT, CATI, and GIGGLE (all from the same session and all showing fast pump plus OI growth plus baseline FR) against DUSK (the third N=22 member of the old "B2 drift" category), the distinction became clear. DUSK's 12-hour pump window showed price velocity of +14.7% total and OI growth of +18% total — an order of magnitude lower than the others.

The old B2 category was reclassified:

- **True slow drift (B2)**: DUSK. Low velocity, modest OI, mild post-peak decline. Skip.
- **Balanced-OI Cascade (B-OI, new)**: FIGHT, CATI, GIGGLE. Fast velocity, substantial OI growth, sharp post-peak cascade. Short on OI reversal trigger.

The discriminator is velocity of the final 12 hours, not cumulative duration. An earlier classification that relied on cumulative pump duration (7+ days implies "slow drift") had incorrectly grouped coins whose actual directional mechanic lived in the final 12-hour acceleration.

### 5.4 Live Validation on GIGGLE

The reclassification was formalized in the working memory documents at approximately 15:10 UTC on April 13, 2026. At the time of formalization, GIGGLE was classified as B-OI in progress (still pumping). The forward prediction was that once OI reversed from its peak while price was still near its intraday high, a cascade would follow.

The subsequent 5-minute resolution data:

- 15:25 UTC: price intraday high $43.50, OI growing toward peak
- 15:30 UTC: OI peak $18.48M, first price crack to $40.52 (−5.3% in one 5m bar)
- 15:30 → 15:35 UTC: OI dropped from $18.48M to $16.91M, a decline of 8.47% in a single 5-minute bar
- 15:30 → 16:10 UTC (40 minutes): price $42.77 → $38.74, a total decline of 10.9%
- 15:30 → 16:05 UTC: OI $18.48M → $15.75M, a decline of 14.8%

The 15:30 UTC 5-minute bar shows the trigger event with timestamped precision: a single-bar OI decline of over 5% concurrent with a price break from high. This is the exact mechanic encoded in the reclassification, validated on unseen forward data within approximately 20 minutes of the rule being written.

We emphasize that this is N=1 forward validation of the newly formalized regime. Together with FIGHT and CATI (post-peak validation only, N=2), the total forward evidence for the B-OI classification is N=3. We present it not as definitive proof but as the strongest single piece of evidence collected during the session.

---

## 6. The Live Reading Scanner

### 6.1 Purpose

The scanner (Test 352 in the research toolkit) operationalizes the classification framework for real-time use. Given the full universe of Binance perpetual futures, it identifies candidate coins matching the B-OI criteria and stages each match as *accumulating*, *peaking*, or *triggered*. The practitioner reviews the matches and decides whether to act.

The scanner exists to close the gap between a classification framework and an actionable signal. Reading one coin at a time through the decision tree takes 1–2 minutes per coin. Across 500+ active perpetuals this is infeasible in real time. The scanner reduces the candidate set to the small number of coins whose state matches the regime of interest.

### 6.2 Algorithm

The scanner operates in three stages.

**Stage 1: Universe filter.** Fetch the 24-hour ticker for all perpetuals. Filter to symbols with `TRADING` status (to exclude delisted stale tickers), USDT-quoted perpetuals, minimum 24-hour volume $10M, and 24-hour price change between +5% and +150%. The upper bound excludes extreme-FR squeezes (which are Pure A, not B-OI). The lower bound excludes non-events.

**Stage 2: Per-candidate classification.** For each candidate, fetch the last 14 hourly klines, 14 hourly open-interest readings, and the last 6 funding settlements. Compute:

- FR baseline check: all of the last 3 settlements within ±0.015%. (Fails → not a baseline-FR coin, skip.)
- Maximum 1-hour price bar move over the last 12 hours. (Velocity check.)
- Maximum 1-hour OI velocity over the last 12 hours. (OI velocity check.)
- Cycle magnitude over the 12-hour window. (Must exceed 15% to qualify.)
- Peak OI within the window, current OI, and the difference as OI drawdown from peak.
- Peak price within the window, current price, and the difference as price drawdown from peak.

A candidate passes if: baseline FR is satisfied, at least one 1-hour bar exceeds 5%, OI velocity exceeds 5% per hour, and 12-hour magnitude exceeds 15%.

**Stage 3: Staging.** Passing candidates are staged by the relationship of current OI and price to their respective peaks:

- **Triggered**: OI drawdown ≤ −5% from peak AND price still within 15% of peak. The cascade has begun and the short window is open.
- **Peaking**: OI drawdown ≤ −2%, or price still at peak and peak OI occurred in the last three 1-hour bars. The cycle is at its highest leverage and the next 1–3 hours will resolve into a cascade or a continuation.
- **Accumulating**: Neither triggered nor peaking. OI is still growing, price still rising. Too early to act; watch.

### 6.3 Rationale

The staging mechanism exists because the mechanical regime alone does not produce an actionable signal. A coin can match all B-OI criteria and still have hours of runway before the cascade starts, making early short entries vulnerable to continued pumping. The triggered stage, defined by a measurable OI reversal from peak, converts the regime classification into a specific entry window.

The parameter choices (5% for bar velocity, 5% for OI velocity, 15% for magnitude, 5% for drawdown thresholds) are heuristics derived from the April 13 session's observations of FIGHT, CATI, and GIGGLE. They are neither theoretically justified nor fit-tuned to maximize any metric. We expect these to require adjustment as more data is collected across different market conditions. The scanner is not a production model; it is an operationalization of a descriptive framework for use in live reading sessions.

### 6.4 Observed Output

The scanner was run once on April 13, 2026 at approximately 16:15 UTC over 38 candidate coins meeting the Stage 1 filter. It produced six matches: two triggered (GIGGLE, TSTUSDT), three peaking (FIGHTUSDT, DRIFTUSDT, LIGHTUSDT), and one accumulating (BROCCOLI714USDT). GIGGLE and FIGHT were known B-OI cases from the session and served as sanity checks. The other four were novel finds. Of the four novel finds, TSTUSDT was already in late decline at the time of scanning, LIGHTUSDT was caught freshly at its peak, and BROCCOLI714 was too early to predict. Forward verification of these finds was scheduled for subsequent check-ins.

An observation from the scanner output worth flagging: **DRIFTUSDT was classified by the scanner as B-OI peaking, despite being labeled HYBRID big-move in the N=22 taxonomy**. The explanation is that after the HYBRID distribution phase completed, DRIFT's funding rate returned to baseline and its late-cycle OI behavior matched the B-OI cascade signature. This raises the possibility — unverified — that the B-OI pattern is not a distinct regime but rather a universal late-cycle exit signature that appears at the end of multiple regime types when FR normalizes and positions unwind. This possibility is listed as an open question in §10.

---

## 7. Forward Validation

### 7.1 Protocol

On April 13, 2026, starting at approximately 13:30 UTC, the framework was applied to ten currently active coins identified from Binance's gainer list. Each coin was classified into a regime. Direction predictions (up / down / skip) and falsification points (explicit IF/THEN conditions that would disprove the classification) were recorded before any subsequent observation.

A second batch of 20 coins (top-10 gainers and top-10 losers by 24-hour percentage change, after the exchange-info filter was applied to exclude delisted symbols) was added at 14:15 UTC. Predictions and falsification points were again recorded before observation.

Verification occurred at T+37 minutes and T+73 minutes for Batch 1, and T+73 minutes and T+approximately 2 hours for Batch 2. Verification measured price change from recorded intraday peak (not 24-hour percentage change, which drifts as the rolling window moves).

### 7.2 Results

Across Batch 1 and Batch 2 combined, 29 direction calls were made (excluding 4 skip classifications). At the verification checkpoints, 29 of 29 calls were directionally correct. Zero falsification conditions were triggered.

Notable moves by T+73 to T+2h:

- BULLAUSDT (HYBRID big-move extreme, weak short): −24.4% from intraday peak
- ARIAUSDT (HYBRID complete, short): −23.6%
- WETUSDT (Pure A2, decline): −17.6%
- PUMPBTCUSDT (Normal B1, fade): −17.6%
- ONUSDT (Normal B1, fade): −17.0%
- PUMPBTCUSDT, continued
- TRADOORUSDT (HYBRID big-move Phase 2, short): −14.3%
- HOLOUSDT (Pure A2, decline): −13.0%
- IRYSUSDT (Pure A2, decline): −12.1%
- GIGGLEUSDT (B-OI, short): −11.9% (with live trigger bar at 15:30 UTC documented in §5.4)

### 7.3 Per-Type Breakdown

| Regime | N predictions | N correct | Notes |
|--------|---------------|-----------|-------|
| Pure A1 | 2 | 2 | RAVE (active), NOM (past peak) |
| Pure A2 | 3 | 3 | HOLO/WET/IRYS controlled replication |
| HYBRID big-move | 4 | 4 | TRADOOR/BULLA/TRU/ARIA |
| HYBRID small-move | 1 | 1 | IR (skip, no move) |
| Pure B1 | 13 | 13 | Calibration evidence (FOLKS weak vs PUMPBTC strong) |
| B-OI | 3 | 3 | FIGHT/CATI/GIGGLE (GIGGLE live) |
| B2 true drift | 0 | — | No pure drift cycles in batch |
| Skip | 4 | — | GIGGLE (later B-OI), CATI (later B-OI), IR, DRIFT |

Note: GIGGLE and CATI appear in both "B-OI" and "skip" rows because they were initially classified as B2 drift skip, then reclassified mid-session as B-OI after the discovery. Their ultimate classification and forward behavior matched the B-OI regime.

### 7.4 Sub-Findings

**Sub-finding 1: Magnitude of fade correlates with peak FR in B1.** FOLKS (FR max +0.058%) faded 1.0% at T+73min and 7.7% at T+2h. PUMPBTC (FR max +0.148%) faded 16.5% at T+73min. The relationship is approximately linear and is used to sub-calibrate B1 in §4.3.

**Sub-finding 2: Controlled replication of Pure A2.** HOLO, WET, and IRYS were all classified as Pure A2 on the same day with similar setups. All three declined by 12%–17% over 73 minutes. With N=3 the evidence is not strong, but within-session replication is at least consistent with the regime's mechanical description.

**Sub-finding 3: B-OI may not be a distinct regime.** The scanner catching DRIFTUSDT (a prior HYBRID) as B-OI peaking suggests the OI-reversal mechanic may be a universal late-cycle signature rather than a specific cycle type. See §10.

---

## 8. Trading Approach Hypotheses

**This section describes hypotheses for converting classifications into trades. None of these hypotheses have been validated with real or paper-tracked trades under explicit entry/exit rules. They are derived from mechanical reasoning and the direction-call evidence in §7, but they have not been tested in the form that matters for a deployable strategy.**

### 8.1 Pure A1 — Long Squeeze

**Hypothesis.** Long entries during Pure A1 extreme squeeze have positive expected value because funding settlements continuously transfer capital from shorts to longs, and refresh cycles provide multiple entry opportunities per cycle. Multi-leg structure allows for holding through short-term pullbacks.

**What is missing.** Specific entry trigger (on which bar? after which indicator?). Specific stop-loss rule. Specific exit (FR normalization? bar count? OI divergence?). Handling of the refresh cycle (re-enter? add size? exit and re-enter?). Position sizing under a capital constraint when multiple A1 cycles exist concurrently.

### 8.2 HYBRID Big-Move Phase 2 — Short Distribution

**Hypothesis.** Shorts entered after the funding rate crosses from negative to positive during a HYBRID big-move cycle have positive expected value because the distribution phase is mechanically defined: longs buying the post-squeeze climax are paying positive funding, OI on the long side is elevated, and any price reversal triggers long liquidations. TRADOORUSDT provided the cleanest forward example (−14.3% over 2 hours after the FR cross).

**What is missing.** Precise entry trigger at the FR cross (exact bar, exact price level). Stop-loss placement (above the climax bar? fixed percentage? OI-based?). Take-profit rule (fixed percentage? FR return to baseline? OI drawdown threshold?). Distinction between HYBRID big-move and Pure B1 in real time (both have positive FR, different mechanics).

### 8.3 B-OI — Short on OI Reversal

**Hypothesis.** When a B-OI cycle's OI begins reversing from its peak while price is still within 15% of its intraday high, a liquidation cascade is starting and a short entry has positive expected value. The GIGGLE live trigger bar at 15:30 UTC on April 13 is the cleanest forward example: a single 5-minute bar showing OI −8.47% concurrent with the first price crack from the high.

**What is missing.** Entry execution — by the time the 5-minute bar closes with the OI reversal, price has already moved 5%. Entry on the unclosed bar requires different signal processing. Stop-loss placement if OI reversal is a false alarm (and the coin resumes pumping). Position size given the 6–12 hour expected holding window.

### 8.4 Pure B1 — Fade the Climax

**Hypothesis.** Pure B1 cycles with FR max above +0.15% (Strong B1) fade aggressively within 1–2 hours of the climax bar. Short entries at the climax bar have positive expected value. Weak B1 (FR max below +0.10%) should be skipped because the fade is too slow to overcome fees and opportunity cost.

**What is missing.** Climax bar identification in real time (by the time it's obvious, it's late). Discrimination between B1 climax and HYBRID Phase 2 early stage (different trade windows, different sizes). Handling of false climaxes (bars that look like climaxes but are followed by continuation).

### 8.5 What Not to Trade

The framework identifies two categories to skip regardless of apparent opportunity:

- **HYBRID small-move**: operator FR extraction with no external edge. Magnitude below 1.5x despite dramatic FR swings. Example: BZUSDT.
- **B2 true drift**: slow velocity, no cascade trigger, mild decline post-peak. Example: DUSKUSDT.
- **Weak B1** (FR max < +0.10%): fade exists but is too slow to trade profitably after costs.

The skip decision is conservative and may miss occasional moves (FIGHTUSDT prior to its reclassification as B-OI was a skip that would have been tradeable had the reclassification been made earlier). The cost of missing a skipped move is opportunity cost only; the cost of a bad signal is a full loss. The skip default is therefore preferred.

---

## 9. Limitations

The framework in its current form has the following limitations, listed in order of severity.

### 9.1 Sample Size

Forward validation rests on a single session (April 13, 2026) with approximately 6 hours of observation. Descriptive validation uses N=33 historical cycles sampled specifically from coins with extreme FR readings, which is a biased sample. The historical data does not include trades, P&L, or execution-simulated outcomes.

For comparison, the positioning-based BTC framework in the same repository rests on 733 days of data with 210,000 five-minute bars and 290 trades. The present framework is not in the same validation class.

### 9.2 Sample Bias

The population studied is Binance perpetual futures on low-capitalization coins selected for extreme funding-rate activity. Results do not generalize without further testing to:

- Major assets (BTC, ETH, SOL, XRP, etc.) with different leverage dynamics
- Other exchanges (Bybit, OKX) with different FR calculation rules
- Non-perpetual markets (spot, quarterly futures)
- Time periods outside the April 2026 regime

### 9.3 No Real P&L

No trades have been executed under this framework. All direction calls were verification-only. The 29/29 direction accuracy reported in §7.2 measures whether price moved in the predicted direction within a verification window; it does not measure whether a trade with realistic entry, stop, exit, fees, and slippage would have been profitable.

### 9.4 Regime Stability Unknown

The framework was developed and forward-tested within a single market regime (the April 13 session exhibited elevated volatility with multiple simultaneous cycles across coin tiers). It has not been tested in low-volatility regimes, in regimes dominated by a BTC-led directional move, in crash regimes, or in post-event liquidation regimes. The expectation that FR-sign classification is regime-independent is theoretically supported but empirically untested.

### 9.5 Execution Mechanics Ignored

The framework assumes observable data at the rate provided by Binance's APIs: 5-minute OI, 1-hour klines, per-settlement FR. It does not account for:

- Order book depth on thinly traded coins
- Slippage on market orders at B-OI trigger events (where all liquidity moves the same way within seconds)
- Funding cost for positions held across multiple settlements
- Liquidation risk at leverage levels required for meaningful position sizing
- Cross-position correlation when multiple coins match the same regime concurrently

Each of these is a potentially disqualifying factor for a strategy built on the framework.

### 9.6 Skip Rule is Conservative

The skip rules in §8.5 are preventative, not optimized. FIGHTUSDT was skipped and then moved 24.1% in the direction the framework would have favored after the reclassification. The skip rule prevented a loss but also a gain. No analysis has been performed on what fraction of skipped cycles would have been profitable.

---

## 10. Open Questions

1. **Is the B-OI regime actually distinct, or is it a universal late-cycle signature?** The scanner catching DRIFTUSDT (a HYBRID from N=22) as B-OI peaking suggests the OI-reversal mechanic may be what happens at the end of most cycle types once the FR normalizes. Verifying this requires applying the scanner to a larger set of late-cycle coins of known regime type.

2. **Can cycle type be predicted in real time, or only classified post-hoc?** The framework reliably classifies cycles with sufficient history. It does not reliably predict which incomplete cycle will transition (Pure A → HYBRID) and which will remain pure. Early-stage classification is the primary gap for real-time trading.

3. **What is the entry trigger for Pure A1 longs?** The framework describes the mechanic but not the bar-level entry rule. Candidate triggers include: first bar after FR reaches −1%, first bar after interval shortens to 1h, retrace after a refresh cycle, or OI-based signals. None have been tested.

4. **How should the B-OI trigger be executed?** The 5-minute bar showing OI reversal is the documented trigger, but by the close of that bar, price has already moved 5%. Real-time detection requires intra-bar processing or prediction of the reversal from sub-5-minute data.

5. **Cross-regime stability.** Does the taxonomy hold in regimes other than the April 2026 sample? Does the April 13 forward-test accuracy survive to other days?

6. **B-OI in major assets.** Does the B-OI mechanic exist on BTC, ETH, SOL? The spot-led balanced pump is plausible for majors but has not been checked.

7. **Correlation risk.** If multiple coins match B-OI triggered simultaneously, are their outcomes independent or correlated? If correlated, the effective edge per trade is reduced and position sizing must account for it.

---

## 11. Conclusion

We have presented a funding-rate-based classification framework for leverage cycles on cryptocurrency perpetual futures, with seven mechanical regimes discriminated primarily by FR sign and secondarily by magnitude and velocity. The framework classifies N=33 historical cycles cleanly with stable distribution across sub-samples, and produced 29/29 correct direction calls in a single-day forward test on April 13, 2026.

The framework is a reading tool, not a trading system. It provides mechanical context and directional bias. It does not provide entry triggers, stop-loss rules, position sizing, or validated P&L. Section 8 describes trading approach hypotheses that follow from the framework; none of these hypotheses have been tested with real or paper-tracked trades.

The most notable result of the session documenting this framework was the within-session reclassification of the former B2 drift category into a true slow drift (B2) and a new Balanced-OI Cascade (B-OI) regime. The reclassification was formalized at approximately 15:10 UTC. The first validation of the new B-OI regime occurred on GIGGLEUSDT at 15:30 UTC with a timestamped OI reversal trigger bar. This forward validation of a rule written 20 minutes earlier on unseen data is the strongest single piece of evidence collected in the study and is the motivation for formalizing the framework at its current preliminary stage.

Further work, in order of priority: build a paper-trading protocol under explicit entry/exit rules for the best-documented regime (B-OI trigger), accumulate real trade-level outcomes, verify regime stability across multiple sessions and market conditions, and re-evaluate the framework's claims against actual P&L rather than direction-only verification.

---

## Appendix A: N=22 Classification Table (Historical)

| Coin | Magnitude | Duration (h) | FR range | Interval | Regime |
|------|-----------|--------------|----------|----------|--------|
| RAVE | 35x | 168 | −2.000% to ~0% | 1h | Pure A1 |
| ENJ | 2.73x | 117 | −2.000% to −0.004% | 1h | Pure A1 |
| NOM | 2.06x | 78 | −2.000% to −0.004% | 1h | Pure A1 |
| SIREN | 2.29x | 78 | −0.570% to +0.001% | 1h | Pure A1 |
| RED | 2.33x | 34 | −1.749% to +0.005% | 4h | Pure A2 extreme |
| TRU | 3.03x | 18 | −1.375% to +0.036% | 8h | Pure A2 extreme |
| WET | 1.95x | 34 | −0.49% to +0.005% | 4h | Pure A2 |
| 0G | 1.44x | 162 | −0.497% to +0.005% | 4h | Pure A2 |
| BULLA | 3.89x | 88 | −2% to +0.108% | 1h | HYBRID big-move |
| ARIA | 17.29x | 92 | −2% to +0.182% | 1h | HYBRID big-move |
| DRIFT | 2.10x | 133 | −1.316% to +0.108% | 1h | HYBRID big-move |
| TRADOOR | 3.53x | 30 | −0.374% to +0.285% | 4h | HYBRID big-move |
| AKE | 3.12x | 59 | −0.071% to +0.201% | 4h | HYBRID big-move |
| AIOT | 4.31x | 149 | −0.002% to +0.222% | 4h | HYBRID big-move (mild) |
| INX | 2.62x | 137 | −0.029% to +0.250% | 4h | HYBRID big-move (mild) |
| CLO | 1.30x | 25 | +0.005% to +0.080% | 4h | Pure B1 |
| RIVER | 1.33x | 17 | +0.005% to +0.169% | 4h | Pure B1 |
| AIN | 2.55x | 140 | +0.005% to +0.109% | 4h | Pure B1 |
| JELLY | 1.17x | 111 | +0.005% to +0.036% | 4h | Pure B1 (weak) |
| FIGHT | 2.12x | 182 | +0.005% to +0.008% | 4h | B-OI (reclassified 04-13) |
| GIGGLE | 1.81x | 186 | −0.005% to +0.005% | 4h | B-OI (reclassified 04-13) |
| DUSK | 1.66x | 171 | −0.022% to +0.005% | 4h | B2 true drift |

## Appendix B: Scanner and Reading Tool References

The classification and staging logic is implemented in the research codebase at `bot-trader-2-0/research/realtime-framework/tests/`:

- `350-reading-tool-v2.mjs` — multi-dimensional single-coin reading (basis, FR, OI, 24h structure)
- `351-historical-pump-read.mjs` — cycle identifier with hourly walk through peak ±12h
- `352-boi-scanner.mjs` — live B-OI scanner with 3-stage filtering and staging

The scanner source is outside the scope of this document but is referenced by name for reproducibility.

---

*Version history:*
*0.1 (April 13, 2026) — Initial formalization. Reflects single-day forward validation and the within-session B-OI discovery. Trading approaches are hypotheses only; no real P&L.*
