# pine-indicators

TradingView indicators I made (Pine Script v5)

## Liquidity Sweep Pro

Marks stop hunts - when price wicks through a level and closes back inside.

Tracks the last few swing highs/lows plus prev day and prev week high/low. Red triangle = highs swept, green = lows swept. PDH/PDL/PWH/PWL labels show sweeps of the daily/weekly levels.

There's a small table with stats: how many sweeps were on the chart and how often price reversed by X% in N bars after. Useful to check if it actually works on a coin before trading it.

Best on 15m - 4h. Has alerts. I don't trade it alone, only together with market structure.
