# Pine Indicators

A collection of TradingView indicators written in Pine Script v5, built from real crypto trading setups.

## Liquidity Sweep Detector

Detects **stop hunts / liquidity grabs**: a candle wicks beyond a recent swing high or low (triggering resting stop orders) and then closes back inside the range.

- 🔻 **Bearish sweep:** wick above a swing high, close back below it (buy-side liquidity taken)
- 🔺 **Bullish sweep:** wick below a swing low, close back above it (sell-side liquidity taken)

### Features
- Non-repainting swing detection (confirmed pivots)
- Optional volume-spike filter to cut noise
- Level age limit so stale swings are ignored
- Swept levels drawn on the chart
- Built-in alerts for both directions

### Settings
| Input | Default | Description |
|---|---|---|
| Swing Length | 5 | Bars on each side to confirm a swing high/low |
| Max Level Age | 50 | Ignore swing levels older than N bars |
| Require volume spike | on | Only signal when volume > MA × multiplier |
| Volume MA Length | 20 | Period of the volume moving average |
| Volume Spike Multiplier | 1.5 | How strong the volume spike must be |
| Draw swept levels | on | Show dashed lines at swept levels |

### How to use
1. Open TradingView → **Pine Editor**
2. Paste the contents of `liquidity_sweep.pine`
3. Click **Add to chart**
4. (Optional) Create alerts: **Alerts → Condition → Liquidity Sweep Detector**

> Signals are a context tool, not a standalone strategy. Combine with market structure and risk management.

## License
MIT
