# Trading Strategy Research

Empirical research on cryptocurrency trading strategies. Data-driven, mechanically explainable, no lagging indicators.

## Principles

- **Understand mechanics first, build models second.** Every model must answer: WHO acts, WHY they act, WHAT happens, HOW it unfolds, WHEN to enter, WHERE (which regime).
- **No lagging indicators.** EMA, RSI, MACD are backtest artifacts. Live trading requires structural signals observable in real-time.
- **Entry BEFORE the move, exit by understanding the END.** Detect loaded barrels, not explosions in progress.
- **Honest research.** Errors documented, look-ahead bias audited, OOS validation mandatory.

## Strategies

### [Positioning-Based BTC Trading](strategies/positioning-based-btc.md)

**733-day empirical study** on BTC perpetual futures using crowd positioning data (long/short ratios from Binance Futures).

- **8 models** grounded in market microstructure theory
- **232 trades** (with 72h post-SL cooldown), **WR 80%**, **PF 3.71**
- **+867% cumulative PnL**, walk-forward efficiency 0.97
- Cross-asset execution validated on **ETH** (+981%) and **SOL** (+256%)
- Monte Carlo significance $p < 0.01\%$

Key findings:
- Participant positioning predicts direction more reliably than price action, volume, or funding rates
- Signal horizon is **7-14 days** (no intraday edge from positioning data)
- Drawdown is mathematically irreducible at given WR/SL/leverage — practical limit **x2 leverage**
- Cross-asset execution requires **regime alignment** (alt must also be in corresponding positioning zone)

### [OI-Flow Shitcoin Trading](strategies/oi-flow-shitcoin-trading.md)

**246-coin empirical study** on shitcoin perpetual futures using open interest flow as the unifying signal.

- **7 models** (barrel LONG, silent exit SHORT, pump exhaustion SHORT, dip-buy LONG, bounce LONG, volatile barrel LONG, continuation LONG)
- **324 barrel trades** (silent + volatile combined), **+1,185% cumulative PnL**
- **OI-price divergence** as universal coin filter — 93% accuracy from 7 days of data
- Forward validation: scanner called **SWARMS +13%** and **NOM +20%** in real-time
- Tested on **6,400+ trades** across 246 coins; model works on 14 coins with divergence ≥40%

Key findings:
- **OI flow direction = the single unifying principle.** All 7 models reduce to: follow where smart money puts capital
- **OI-price divergence ≥40%** separates smart-money coins from retail noise. Below 40% = -1,232% on same signals
- **Shitcoins pump in Asia session** (0-7 UTC): WR 79% PF 5.0 vs US WR 67% PF 1.9
- **Shitcoins pump when BTC dumps.** BTC 4h down >2% → barrel PF 33.8. BTC up >2% → PF 0.69
- **SHORT barrel doesn't exist.** Dumps are either unpredictable (silent death) or pump aftermath (exhaustion)
- **TopLong 40-50% = sweet spot** for barrel. Balanced crowd → genuine breakout (PF 4.2)
- **Trigger quality matters more than coin selection.** DD comes from weak triggers on good coins, not bad coins

## Data Sources

All research data sourced from **Binance Futures** (API + S3 public archive). No Bybit data in research.

## License

Research shared for educational purposes. Not financial advice.
