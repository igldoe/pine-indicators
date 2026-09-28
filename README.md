# pine-indicators

My TradingView indicators (Pine Script v5).

## Liquidity Sweep

Shows when price takes out a recent swing high/low with a wick and closes back inside. Basically a stop hunt.

- red triangle = swept a high
- green triangle = swept a low

There's an optional volume filter so it only fires on high volume candles. Levels older than 50 bars are ignored.

### How to use
Copy `liquidity_sweep.pine` into the Pine Editor on TradingView and add it to the chart. Alerts are included.

Not a strategy on its own, I use it with market structure.
