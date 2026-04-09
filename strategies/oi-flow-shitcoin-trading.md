# OI-Flow Trading on Shitcoin Perpetual Futures: A Multi-Model Framework

**Author:** D. Chystiakov  
**Date:** April 9, 2026  
**Version:** 1.0 (Draft)  

---

## Abstract

We present a systematic framework for trading shitcoin perpetual futures on Binance, derived from a two-day intensive empirical study across 246 coins with 90--180 days of 5-minute data. The framework is built on a single unifying principle: **open interest flow direction determines price trajectory**. We develop seven distinct models, a universal coin qualification filter (OI-price divergence ≥40%), and demonstrate that the system produces cumulative returns exceeding +1,185% on combined barrel models alone. We further identify that divergence rate---the frequency with which OI moves opposite to price---serves as a participant fingerprint, distinguishing smart-money coins from retail-driven noise with 93% accuracy using only 7 days of data. The research systematically addresses the WHO/WHY/WHAT/HOW/WHEN/WHERE framework for every model, grounding each signal in mechanical market microstructure rather than statistical pattern-matching.

**Keywords:** shitcoin futures, open interest flow, liquidation cascade, barrel loading, OI-price divergence, market microstructure

---

## 1. Introduction

### 1.1 Problem Statement

The shitcoin perpetual futures market on Binance encompasses over 500 actively traded pairs with daily turnovers ranging from \$1M to \$3B. These instruments exhibit extreme volatility (daily ranges of 8--67%), thin order books, and participant bases dominated by high-leverage retail traders. Despite this apparent chaos, we demonstrate that systematic, mechanically-grounded trading is possible---provided one understands *who* is acting and *why*.

### 1.2 Failure of Naive Approaches

Our research began by auditing seven pre-existing backtested models (Tests 260--302). The results were sobering:

| Model | Backtest | Live Result | Failure Mode |
|-------|----------|-------------|--------------|
| Level Breakout | WR 81%, PF 6.5 (RIVER) | WR 55%, PF 1.7 (mid-cap) | Enters at pump tops; no cycle awareness |
| Cascade Fade | WR 71%, PF 1.9 (DOGE) | PF 0.25--0.71 (all losing) | Fat-tail losers; doesn't work on mid-cap |
| Mark-Index Spread | WR 98%, PF 56 | Not applicable | Arbitrage, not directional trading |
| Barrel + Judas Swing | Doesn't beat random | --- | No edge |
| Funding Pain | WR 82% (N=11) | Funding = response, not signal | Enters after squeeze already happened |
| Phase Contrarian | WR 55%, PF 1.1 | --- | Too weak standalone |
| OI-Type Model | +625% but DD -301% | --- | Untradeable drawdown |

The critical lesson: **models that describe price patterns fail; models that describe participant behavior succeed.** Every failed model attempted to trade *what price did*. Every successful model trades *what participants are doing with their money*.

### 1.3 The Unifying Principle: Follow the OI Flow

Through bar-by-bar study of 147 major moves across 5 coins (BULLA, SIREN, SWARMS, ZEC, RED), we discovered that open interest behavior relative to price is the single variable that explains all tradeable patterns:

$$\text{OI flow direction} \implies \text{next price direction}$$

Concretely:

- **OI loading in silence** → barrel loaded → pump coming (Model 1)
- **OI loading in volatility** → smart money masking entry → pump coming (Model 6)
- **OI growing during active pump** → smart money still adding → pump continues (Model 7)
- **OI dropping in silence** → whale exiting quietly → dump coming (Model 2)
- **OI dying after pump peak** → fuel exhausted → dump coming (Model 3)
- **OI growing on price dip** → absorption (buying the dip) → recovery coming (Model 4)
- **OI stabilizing after dump** → cascade over → bounce coming (Model 5)

This is not parameter optimization. This is market mechanics: when participants put money in (OI grows), the direction of that money relative to price tells you where price will go next.

![OI Flow Principle](shitcoin-figures/fig5-oi-flow-principle.svg)
*Figure 5: All seven models mapped to the OI flow direction. Top half: OI growing = LONG models. Bottom half: OI dropping = SHORT models. The central question is always the same: where is smart money putting capital?*

### 1.4 Contributions

1. A universal coin qualification metric (OI-price divergence) that identifies tradeable coins with 93% accuracy from 7 days of data.
2. Seven mechanically-grounded trading models covering 68% of all major shitcoin moves.
3. The volatile barrel model, which captures pumps on extreme-volatility coins where silence-based detection fails.
4. Demonstration that OI flow direction is the unifying principle behind all successful shitcoin models.
5. Comprehensive negative results: why cascade fade fails on mid-cap, why SHORT barrel doesn't exist, and why cascade warning without silence is noise.
6. Forward validation: barrel scanner correctly identified SWARMS (+13%) and NOM (+20%) pumps in real-time.

---

## 2. Data and Methodology

### 2.1 Data Sources

| Dataset | Granularity | Source | Coverage |
|---------|------------|--------|----------|
| Klines (OHLCV + taker buy) | 5 min | Binance Futures API (paginated) | 60--180 days per coin |
| OI history (USD) | 5 min | Binance S3 daily archives | 90 days (S3), 20 days (API) |
| Top trader long/short ratio | 5 min | Binance S3 / API | 90 days |
| Funding rate | 8 hr / variable | Binance API | 90 days |
| BTC reference klines | 5 min | Binance historical | 733 days |

Total data: 3.3 GB across 246 coins. Primary analysis on 14 coins with OI-price divergence ≥40%; extended testing on all 246.

### 2.2 Derived Metrics

**OI-Price Divergence Rate** (the master filter):

$$D(T) = \frac{|\{h \in T : \text{sign}(\Delta P_h) \neq \text{sign}(\Delta \text{OI}_h)\}|}{|T|}$$

where h indexes 1-hour windows over period T, ΔP_h is the price change and ΔOI_h is the OI change in window h.

**OI Predictive Power** (grey-zone filter):

$$\text{Pred}(T) = \frac{|\{h \in T_{\text{div}} : \text{sign}(\Delta P_{h+1}) = \text{sign}(\Delta \text{OI}_h)\}|}{|T_{\text{div}}|}$$

where T_div ⊂ T is the set of divergent hours (where OI and price moved opposite). If Pred ≥ 52%, the divergent OI correctly predicts the next hour's price---indicating smart money, not noise.

**OI Elasticity:**

$$\epsilon = \frac{1}{N}\sum \frac{\Delta OI_h \;/\; OI_{h-1}}{\Delta P_h \;/\; P_{h-1}}$$

*(summed over all 1-hour windows where price dropped more than 3%)*

When price drops >3%, how much does OI drop? Low elasticity (<0.5) = diamond hands. High (>0.8) = paper hands (instant liquidation).

### 2.3 Research Process

The research followed the principle: *understand mechanics first, build models second*. Every model must answer:

- **WHO** acts (smart money, retail, market makers)?
- **WHY** they act (overloaded positioning, funding pressure, information advantage)?
- **WHAT** happens structurally (cascade, squeeze, absorption)?
- **HOW** it unfolds bar-by-bar (gradual vs spike, OI growing vs dropping)?
- **WHEN** to enter (which session, what trigger quality)?
- **WHERE** in the regime (uptrend, downtrend, BTC context)?

![OI-Price Divergence Filter](shitcoin-figures/fig1-divergence-filter.svg)
*Figure 1: OI-price divergence rate as the universal coin filter. Coins with divergence ≥40% (green) produce +832% over 260 barrel trades. Coins below 40% (red) produce -1,232% on the same signals. FARTCOIN (yellow) has high divergence but fails the OI Predictive Power secondary filter.*

---

## 3. The Master Filter: OI-Price Divergence

### 3.1 Discovery

When we ran the barrel model on all 246 available coins, the aggregate result was negative: -400% on 6,400 trades. The model appeared broken. However, partitioning coins by OI-price divergence revealed a stark bifurcation:

| Divergence | Coins | Trades | PF | Total PnL | DD |
|------------|-------|--------|-----|-----------|-----|
| ≥40% | ~14 | 260 | 3.03 | **+832%** | 63% |
| <40% | ~232 | 6,140 | 0.93 | **-1,232%** | 1,489% |

The same model. The same parameters. The only difference: *who trades these coins*.

### 3.2 What Divergence Measures

A divergence rate of 50% means that half the time, OI moves *opposite* to price. When price goes up, someone is selling (OI goes down). When price goes down, someone is buying (OI goes up). This is the signature of a counter-trend participant---a smart money entity that provides liquidity against the crowd.

On coins with low divergence (<30%), OI and price move together: price up → FOMO longs (OI up), price down → liquidations (OI down). This is pure retail momentum. No one stands against the crowd. Barrel triggers on these coins are random noise.

### 3.3 Participant Fingerprint

The divergence rate, combined with OI elasticity, creates a taxonomy of coin participants:

| Divergence | OI Elasticity | Participants | Examples |
|------------|--------------|-------------|---------|
| < 0 (OI grows on dip) | < 0 | Institutions adding on weakness | COIN (Coinbase), HOOD (Robinhood) |
| 0--0.5 | <0.5 | Strong community (diamond hands) | SIREN, ZEC, SWARMS, RED, KERNEL |
| 0.5--0.8 | Mixed | Transitional | NOM, GLM |
| >0.8 | >0.8 | Pure retail on high leverage | BEAT (20%), 4U (18%), POWER (21%) |

**The filter also distinguishes shitcoins from real projects.** Tokenized stocks (TSLA, COIN, HOOD) naturally have high divergence because institutional holders behave counter-cyclically.

![Participant Fingerprint](shitcoin-figures/fig6-participant-fingerprint.svg)
*Figure 6: Participant fingerprint scatter. Smart money coins (green, top-left) have high divergence and low elasticity. Retail noise coins (red, bottom-right) have low divergence and high elasticity. FARTCOIN (yellow) is a false positive caught by the OI Predictive Power filter.*

### 3.4 Window Stability

A critical practical question: how much data is needed to compute reliable divergence?

| Window | Std across snapshots | Classification accuracy |
|--------|---------------------|----------------------|
| 7 days | 3.6 | **93%** |
| 14 days | 2.3 | 93% |
| 30 days | 1.5 | 93% |
| 60 days | 0.8 | 94% |

**Seven days suffice.** Divergence is a stable coin characteristic, not a noisy metric. This is computable via the Binance API using 1-hour OI history (limit 500 = 20 days of data).

### 3.5 The FARTCOIN Problem: OI Predictive Power

FARTCOIN has divergence = 53% but barrel fails spectacularly (SL rate 50%). Investigation revealed that its divergence is *noise*, not smart money. The OI Predictive Power metric distinguishes:

| Coin | Divergence | OI Predicts Next Hour | Verdict |
|------|-----------|----------------------|---------|
| KERNEL | 51% | **56%** | Smart money |
| SWARMS | 52% | 53% | Borderline smart |
| FARTCOIN | 53% | **49%** | Noise (coin flip) |

When OI diverges from price on KERNEL, the next hour's price moves in OI's direction 56% of the time. On FARTCOIN, 49%---pure chance. The divergence exists but predicts nothing.

**Combined filter:** Divergence ≥40% AND Predictive Power ≥52% eliminates all grey-zone false positives.

---

## 4. The Seven Models

### 4.1 Model 1: Silent Barrel (LONG)

**Conditions:**


- silence ≥ 12h (range < 4%/h, 50%+ hours quiet)
- ΔOI_silence ≥ +5%
- trigger: vol ≥ 5× avg AND bar ≥ +2%  (green close)
- top wick < 20%  of bar range


**WHO:** Smart money loads positions during silence. The loading is visible only in OI growth---price stays flat, volume dead.

**WHY:** Smart money sees value or positioning extreme. They enter gradually to avoid moving price. Silence = their accumulation window.

**WHAT:** OI grows 5--30% while price ranges <4%/h. Then a trigger bar breaks the silence with 5x+ volume.

**HOW:** Trigger bar = 3--15% of total move. Pump is GRADUAL, extending 12--48h. Two sub-types:

- *Short squeeze* (TopLong <40% during silence): shorts overloaded, trigger liquidates them. OI drops hour 1 (shorts closing), then new money enters hours 3+. MFE: +31--73%.
- *Fresh demand* (TopLong >55%): new buying interest. OI grows from hour 1. MFE: +14%.

**WHEN:** Asia session (0--7 UTC) strongly preferred. WR 79%, PF 5.0 vs US session WR 67%, PF 1.9.

**WHERE:** 7-day uptrend. BTC flat or down (BTC_4h < +2%). When BTC pumps, money flows to BTC, not shitcoins.

**Results:** 188 trades on 12 Div≥40% coins. WR 73%, PF 3.1, +663%. Quality filter (vol≥8x, bar≥2%, OI≥8%): N=28, PF 14.2, DD 22%.

**Exit:** Trail activate at +5% MFE, trail 30% of peak. SL -8%. Timeout 48h.

![Barrel Loading Lifecycle](shitcoin-figures/fig2-barrel-lifecycle.svg)
*Figure 2: The four-phase barrel lifecycle. Entry at Phase 3 (trigger), exit when trail hits in Phase 4.*

### 4.2 Model 2: Silent OI Exit (SHORT)

**Conditions:**


- silence ≥ 6h (50%+ hours quiet)
- ΔOI_bar < -3%  in single 5m bar
- vol < 10× avg (quiet exit, not panic)
- wait 1h:  ΔOI_next hour < -1%  (confirmed)


**WHO:** A whale or large fund exits quietly. OI drops suddenly during silence---but volume stays low, meaning no panic selling. The position was closed via OTC or dark pool.

**WHY:** The participant sees upcoming risk (news, technical breakdown) or simply takes profit. They exit before the crowd notices.

**WHAT:** OI drops 3%+ in a single bar while market is quiet. One hour later, OI is still dropping---confirming this is a real exit, not noise. Cascade follows 2--5 hours later.

**HOW:** Price holds initially (whale's exit is absorbed by the book). Then book thins as MM withdraws. Then cascade.

**Results:** Diamond hands coins only (Div≥40%): N=46, **WR 87%, PF 21.3**, +177%, DD 4.6%.

**Exit:** Hold 4 hours (SHORT is fast on shitcoins).

### 4.3 Model 3: Pump Exhaustion (SHORT)

**Conditions:**


- Δ P_12h > +15%  (pump happened)
- P(t) > 0.9 × max P_12h  (still near peak)
- ΔOI_2h < -2%  (money leaving)
- vol_2h < 0.8 × vol_peak  (volume dying)


**WHO:** Smart money and early longs exit, taking profit. OI drops as positions close. Volume dying = no new buyers.

**WHY:** The pump has exhausted its fuel. The key pre-signal: OI drops 12.5% in the 2 hours *before* the price peak (vs +3.2% when price holds). OI leads price by 2 hours.

**Results:** 1,423 trades, 215 coins. 4h WR 58%, PF 1.20, +710%. 24h WR 65%, PF 1.22, +1,885%. Beats random P95.

### 4.4 Model 4: Dip-Buy (LONG)

**Conditions:**


- Δ P_1h < -3%  (price dropped)
- ΔOI_1h > +2%  (someone buying the dip)


**WHO:** Smart money buying through the dip. Counter-intuitively, when *shorts* enter (TopLong drops) during the dip, the LONG outcome is best (+1,128%). The shorts become squeeze fuel.

**WHY:** Smart money sees value at lower prices. Their buying (OI up) against price decline = absorption. Larger OI growth = stronger signal: OI +12% at dip → avg +1.16%/trade (vs +0.40% at OI +2%).

**Results:** 3,604 trades (most frequent model). WR 49%, PF 1.18, +1,899%. Beats random P95.

### 4.5 Model 5: Bounce (LONG)

**Conditions:**


- Δ P_4h < -10%  (dump happened)
- ΔOI_1h > -1%  (OI stabilized---cascade over)


**WHO:** Cascade has finished. Forced sellers are done. OI stops dropping = no more liquidations.

**WHY:** After cascade, book refills, bargain hunters enter. Recovery is mechanical---price overshot fundamental value during cascade.

**Results:** 42 signals where OI stabilized. 4h WR 62%, avg +0.7%. 12h avg +13.6%. Enhanced: longs exhausted + OI stable + shorts loaded: N=9, avg 12h +24%.

### 4.6 Model 6: Volatile Barrel (LONG)

**Conditions:**


- volatile ≥ 6h (50%+ hours with range > 3%)
- ΔOI_volatile period ≥ +5%
- trigger: green bar ≥ +2% , vol ≥ 3×


**WHO:** Smart money loads during volatility---masked by noise. On extreme-volatility coins (BULLA, SIREN), silence rarely happens. This model catches what silent barrel misses.

**WHY:** Smart money uses volatility as cover. When everyone is distracted by whipsaws, smart money accumulates quietly in the noise.

**WHAT:** OI grows 5--30% during volatile period (not visible to casual observer because price is wild). Trigger = breakout from the noise.

**Critical difference from Model 1:** TopLong ≥50% = *better* on volatile barrel (PF 4.91 vs 1.68). Opposite of silent barrel. When longs dominate AND OI grows through volatility, this is conviction building.

**Results:** 136 trades, Div≥40% coins. WR 72%, PF 2.97, +522%. Quality filter: N=59, PF 4.45, DD 16%.

**Combined with Silent Barrel:** 324 trades, +1,185% (mutually exclusive, additive).

### 4.7 Model 7: Continuation (LONG)

**Conditions:**


- Δ P_4h > +5%  (pump already running)
- ΔOI_1h > +3%  (smart money STILL adding)
- vol ≥ 0.8 × vol_prior  (volume not dying)


**WHO:** Smart money continues to add positions during an active pump. OI +3%/h is aggressive new money, not retail trickle.

**WHY:** Smart money sees further upside. They have information/size advantage. They wouldn't add if they expected reversal.

**The key insight:** Pump size doesn't matter. Whether price is up +5% or +50%, the only predictor of continuation is OI behavior:

| OI Change (1h) | N | WR | PF | Avg/trade |
|----------------|---|----|----|-----------|
| OI growing over +3% | 138 | **81%** | **13.0** | **+12.0%** |
| OI growing 0--3% | 140 | 54% | 1.6 | +1.3% |
| OI dropping over 3% | 118 | **33%** | **0.56** | **-2.9%** |

**This model connects directly to Model 3:** OI growing = stay in (Model 7). OI dying = exit/short (Model 3). They are two sides of the same coin.

![Continuation OI](shitcoin-figures/fig8-continuation-oi.svg)
*Figure 8: Continuation model. OI growth rate during active pump determines outcome. >+3%/h = strong JOIN (WR 81%). Dropping >3%/h = EXIT immediately (WR 33%). Models 7 and 3 are mirror images.*

**Results:** Best combo (OI↑>3% + vol accelerating): N=85, **WR 81%, PF 13.8**, +12.9%/trade. Beats random P95.

---

## 5. Negative Results and Structural Limitations

### 5.1 SHORT Barrel Does Not Exist

We studied 54 dumps >10% across 5 coins. Two dump types exist; neither resembles a barrel:

1. **Silent dump** (SWARMS pattern): silence → sudden drop. But OI does NOT load before. No accumulation visible. Unpredictable with OI data.

2. **Pump-then-dump** (BULLA/SIREN): 62--69% of dumps follow pumps >20%. The dump is the *result* of the pump, not an independent event.

Barrel is a LONG-only pattern. For SHORT, use Models 2 and 3 instead.

### 5.2 Cascade Warning Without Silence = Noise

We tested OI drop >5% as SHORT signal without silence requirement: N=2,486, WR 54%, PF 1.02 = coin flip. On chaotic coins, OI drops >5% happen constantly. The silence requirement is what makes Model 2 work---silence + OI drop = anomaly; OI drop alone = normal noise.

### 5.3 Liq Map: Works Live, Weak on Backtest

Liquidation map (Sliq2%) correctly identified JOE pump exhaustion in real-time (Sliq2% shorts = \$22K ≈0 → pump dead). But backtest improvement was only 2--5pp WR. Cause: leverage distribution is estimated (15/25/30/20/10%), not real. **Liq map useful for extreme cases, not universal filter.**

### 5.4 The `topTraderLong` Data Fix

S3 metrics `topTraderLong` is a **proportion** (0.64 = 64% long), not a ratio. Our initial liq map treated it as percentage (/100), recording <1% longs. This bug was identified and fixed during the research.

### 5.5 Time-Based Early Exit: Cuts Both Tails

We tested "bail if negative after 2h": helped losers (+62% saved on losing coins) but destroyed winners (-375% lost on winning coins). Time-based exits are too aggressive. Mechanics-based exits (OI decel + vol decay while negative) work on NORMAL vol coins but kill EXTREME vol coins where price dips -5% before +200% pump.

---

## 6. Operational Parameters

### 6.1 Session Effect

| Session | WR | PF | N |
|---------|----|----|---|
| Asia (0--7 UTC) | **79%** | **5.0** | 97 |
| EU (7--13 UTC) | 74% | 2.7 | 68 |
| US (13--21 UTC) | 67% | 1.9 | 95 |

Shitcoins pump at night (Asia session) when volume is thin and one player can move the book. The 0--4 UTC window is optimal (WR 86%).

### 6.2 BTC Context

| BTC 4h Change | N | WR | PF |
|---------------|---|----|----|
| Down < -2% | 8 | **88%** | **33.8** |
| Flat ±0.5% | 117 | 76% | 2.9 |
| Up > +2% | 12 | **42%** | **0.69** |

Counter-intuitive: barrel works BEST when BTC dumps. Money rotates: BTC down → traders seek alt gains → shitcoin pumps are real. When BTC pumps, everyone is in BTC---shitcoin triggers are noise.

**Filter:** Skip barrel if BTC(4h) > +2%.

### 6.3 Trigger Quality: Top Wick

SL trades have 37% top wick (rejection). Winners have 30%. The top wick < 20% filter:

| Metric | Base | Top wick < 20% |
|--------|------|-------------------|
| WR | 77% | **84%** |
| PF | 3.8 | **8.9** |
| DD | 26% | **12%** |
| Calmar | 33.9 | **41.6** |

No look-ahead: all metrics from trigger bar close.

### 6.4 DD Source: Weak Triggers, Not Bad Coins

DD comes from weak triggers on GOOD coins, not from bad coins. Winners and losers have identical divergence rates (51%). The difference is trigger quality:

| | Winners | SL Trades |
|---|---------|-----------|
| Vol spike | 15.9x | 11.2x |
| Trigger bar | 2.02% | 1.52% |
| OI growth | 20% | 11% |

**Two-layer architecture:** Layer 1 (coin filter) = *which coins to watch*. Layer 2 (trigger filter) = *when to enter*.

---

## 7. Shitcoin Lifecycle

### 7.1 Phase Distribution

Analysis of SWARMS (90 days) reveals the time budget:

| Phase | % of Time | Description |
|-------|-----------|-------------|
| DEAD | 51% | Range < 2%, volume < 0.5x avg |
| QUIET | 30% | Range < 3% |
| NORMAL | 14% | Range 3--5% |
| VOLATILE | 4% | Range > 5% |
| PUMP | **1%** | > +5%/h |
| DUMP | 0.5% | > -5%/h |

A shitcoin spends 81% of its time dead or quiet. Pumps comprise 1% of time. We hunt this 1%.

### 7.2 Move Coverage

Of 147 moves >10% in 6h across 5 coins:

| Type | N | Models | Coverage |
|------|---|--------|----------|
| Barrel Pump | 10 | Model 1 | Yes |
| Silent Dump | 26 | Model 2 | Yes |
| Pump-then-Dump | 11 | Model 3 | Yes |
| FOMO Pump | 41 | Model 4 (partial) | Yes |
| Dump-then-Pump | 12 | Model 5 | Yes |
| Volatile Pump | ~25 | Model 6 | Yes |
| Continuation | ~35 | Model 7 | Yes |
| Cascade Dump | 9 | Not reliably modelable | No |
| Chaotic | 13 | Noise | No |

Covered: ~125/147 (85%). Remaining 15% = cascading liquidations and pure chaos.

### 7.3 TopLong as Pump-Type Predictor

TopLong during silence predicts not just *whether* a pump happens, but *what kind*:

| TopLong | Pump Type | WR | PF | MFE |
|---------|----------|----|----|-----|
| <30% | Short Squeeze | 80% | 1.9 | **+73%** |
| 40--50% | Balanced | **79%** | **4.2** | +19% |
| 60--70% | FOMO Demand | 55% | 1.0 | +11% |

**Sweet spot: TopLong 40--50%.** Balanced crowd → trigger = genuine breakout, not squeeze or FOMO. PF 4.2.

![Pump Types by TopLong](shitcoin-figures/fig7-pump-types.svg)
*Figure 7: TopLong during silence determines pump type and MFE. Squeeze (<30%) = massive MFE +73% but rare. Balanced (40-50%) = best PF 4.2. FOMO (60-70%) = PF 1.0, skip or tight trail.*

---

## 8. Forward Validation

### 8.1 Barrel Scanner (April 9, 2026)

At 09:19 UTC, the scanner identified 8 coins in barrel loading. Within hours:

- **SWARMS:** +13% (was showing silence 83%, OI +12.6%)
- **NOM:** +20% (was showing silence 50%, OI +6.3%)

Two correct calls from 8 candidates. Both Div≥40% coins.

### 8.2 Virtual Trades (April 8, 2026)

Five LONG entries on Top Movers (JOE, ZEC, ARIA, ORDER, CLOU). All entered at 49--98% of pump cycle. **All lost** (ARIA -80%, CLOU -19%, JOE -9%).

This failure was expected: the models at the time lacked cycle position awareness. The subsequent development of OI-price divergence, trigger quality filters, and BTC context would have prevented all five entries:

- JOE: Sliq2% shorts = \$22K ≈0 (no fuel) → reject
- All 5: entered during US session at 14:44 UTC (worst session) → deprioritize
- ARIA: pump already +34% → too late without OI confirmation

---

## 9. Complete Entry Rules (V2)

**Coin Qualification (7 days of data):**

1. OI-price divergence ≥40% (93% accuracy)
2. OI Predictive Power ≥52% (kills false positives like FARTCOIN)

**Trigger Conditions (real-time):**

3. Model-specific entry (barrel, dip-buy, bounce, continuation, etc.)
4. Trigger quality: vol ≥8x, bar ≥2%, top wick < 20%
5. Session: prefer 0--13 UTC (Asia/EU). Skip US afternoon.
6. BTC context: skip if BTC(4h) > +2%

**Exit:**

7. Trail: activate at +5% MFE, trail 30% of peak
8. Hard SL: -8% (for HIGH vol coins: -15%)
9. Timeout: 48h

**Adaptation by pump type (TopLong during silence):**

- <40%: Short squeeze → wider trail (activate +8%), hold longer
- 40--55%: Balanced → standard trail
- > 60%: Demand-driven → tight trail (+3%) or skip

---

## 10. Open Questions

1. **Forward test:** The scanner identified SWARMS and NOM correctly on April 9. Systematic forward testing over 2+ weeks with virtual trades is needed.
2. **Position sizing:** DD ranges from 12% (quality filter) to 63% (base). Kelly criterion or fixed-fractional sizing remains to be optimized.
3. **Execution costs:** Spreads 0.19--0.25% on mid-cap shitcoins not included in backtest. Need real orderbook analysis.
4. **Multi-coin portfolio:** Correlation between shitcoin pumps (common BTC trigger?) unexplored.
5. **Liq map improvement:** Real leverage bracket data (requires Binance auth) would improve Sliq2% accuracy.
6. **Cascade prediction:** 15% of moves (cascade dumps, chaotic) remain uncovered. Orderbook depth data might help.

---

## Acknowledgments

This research was conducted as an intensive two-day study (April 8--9, 2026), comprising 20+ test scripts, 6,400+ backtested trades across 246 coins, live virtual trading, and real-time scanner validation. The entire process was exploratory and hypothesis-driven, with each finding leading to the next question rather than a predetermined conclusion.

---

*Scripts and data: `research/realtime-framework/tests/303-309-*.mjs`, data in `data-binance/` (3.3 GB).*
