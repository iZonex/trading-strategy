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

## Data Sources

All research data sourced from **Binance Futures** (API + S3 public archive). No Bybit data in research.

## License

Research shared for educational purposes. Not financial advice.
