# Volume Profile Alert — MQL4 Script

A MetaTrader 4 script that constructs a **tick-volume distribution profile** across configurable price bins over a rolling lookback window, normalizes each bin's accumulated volume as a percentage of total volume, and fires high-volume node or low-volume node alerts when the current close falls within a `Point` tolerance of any bin whose normalized volume percentage meets or exceeds `HighVolumeThreshold` or falls at or below `LowVolumeThreshold` — providing a bar-data-derived approximation of a professional Volume Profile analysis without tick-level data feeds.

---

## Overview

Volume Profile is an advanced charting technique used extensively by institutional traders and CME futures participants to understand where the majority of trading activity has occurred within a price range. A high-volume node (HVN) is a price level where a disproportionately large share of historical volume has traded — indicating strong market acceptance and an area likely to act as a magnet or sticky zone. A low-volume node (LVN) is a price level where very little volume has traded — indicating rapid price rejection and an area that price tends to move through quickly rather than pause at. In the traditional application, Volume Profile uses tick-by-tick trade data to build the distribution; this script approximates it using bar-level tick volume and the bar's closing price as the distribution anchor. Each close price is mapped to a price bin, its volume added to that bin's total, and the final distribution normalized to percentage form. The result is an accessible proxy for institutional volume analysis using only standard MT4 data.

---

## Features

- **`ProfileBins`-count price distribution** — `highPrice` and `lowPrice` resolved via `iHighest()` / `iLowest()` over `LookbackBars`; `binSize = (highPrice − lowPrice) / ProfileBins`; `volumeProfile[]` struct array allocated via `ArrayResize(volumeProfile, ProfileBins)`
- **Bin assignment via `MathFloor()` index mapping** — `binIndex = MathFloor((close − lowPrice) / binSize)`; bounds-checked `[0, bins)` before accumulation; volume accumulated into `volumeProfile[binIndex].volumePercentage`
- **Total-volume normalization** — `totalVolume` summed across all bins; each bin divided by `totalVolume`; `totalVolume == 0.0` guard returns `false` if no volume data is available
- **`Point`-precision price proximity scan** — `MathAbs(currentPrice − nodePrice) < Point` matches current close against each bin's representative price level
- **Dual threshold classification** — `volumePercentage >= HighVolumeThreshold` → **Price Entered High Volume Node**; `volumePercentage <= LowVolumeThreshold` → **Price Entered Low Volume Node**
- **Rich alert message** — `AlertVolumeNode()` formats with price and `volumePercentage × 100` as `"Volume Node: %.2f%%"` for direct percentage context
- **Three notification channels:** sound alert, email, and mobile push
- **Lightweight loop** — polls once per minute (`Sleep(60000)`)

---

## How It Works

1. Every minute, `CalculateVolumeProfile()` builds the bin structure, distributes volume, normalizes percentages
2. `currentPrice = iClose(..., 0)` fetched; all `ProfileBins` bins scanned for proximity match
3. On match: `volumePercentage >= HighVolumeThreshold` → HVN alert; `<= LowVolumeThreshold` → LVN alert

---

## Input Parameters

| Parameter               | Type            | Default     | Description                                                        |
|-------------------------|-----------------|-------------|--------------------------------------------------------------------|
| `TradeSymbol`           | string          | `EURUSD`    | Symbol for analysis                                                |
| `Timeframe`             | ENUM_TIMEFRAMES | `PERIOD_H1` | Timeframe for volume profile construction                          |
| `LookbackBars`          | int             | `50`        | Number of bars over which to build the volume distribution         |
| `ProfileBins`           | int             | `20`        | Number of price bins dividing the lookback high-low range          |
| `HighVolumeThreshold`   | double          | `0.7`       | Normalized volume proportion at or above which an HVN alert fires  |
| `LowVolumeThreshold`    | double          | `0.3`       | Normalized volume proportion at or below which an LVN alert fires  |
| `EnableAlerts`          | bool            | `true`      | Fire an on-screen/sound alert                                      |
| `EnableEmail`           | bool            | `false`     | Send an email notification                                         |
| `EnablePush`            | bool            | `false`     | Send a mobile push notification                                    |

---

## Alert Message Format

```
Price Entered High Volume Node detected on EURUSD (Timeframe: PERIOD_H1)
Price: 1.08490, Volume Node: 74.32%
```

---

## Installation

1. Copy `Volume_Profile_001.mq4` to `MQL4/Scripts/` in your MT4 data folder
2. Compile in MetaEditor (F7)
3. Drag onto any chart from Navigator → Scripts
4. Configure inputs and click **OK**

> **Note:** Default thresholds of `0.7` and `0.3` classify the top 30% and bottom 30% of normalized volume distribution as HVN and LVN respectively. Calibrate these values to your instrument's volume characteristics before use on live capital.

---

## Requirements

- MetaTrader 4 (`#property strict` compatible build)
- MQL4 compiler (MetaEditor)

---

## License

MIT License

Copyright (c) 2026

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
