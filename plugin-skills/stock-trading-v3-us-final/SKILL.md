---
name: stock-trading-v3-us-final
description: Advanced US stock analysis with iTick Kline data, Google News RSS sentiment, and AI trading recommendations. Optimized for US stocks. Run in background.
---

# Stock Trading v3 US Final (Skill 20)

Advanced stock analysis for US stocks combining technical data, news sentiment, and AI trading recommendations.

## Input
Comma-separated US stock tickers: e.g. `NVDA,F,RIG,FSLY,PLUG,INTC,AMZN,NFLX,LYG,PLTR`

## API Keys Required
- **iTick API**: Token in `~/.openclaw/credentials/itick.json` OR use directly: `a782c443a2a5493d873850758154e14876bfd88dfb89470b8b97dff36e7938e3`
- **News**: Google News RSS (free, no key needed)

## ⚠️ API Notes (STRICT - NO FALLBACKS!)
- **iTick API**: Use ONLY iTick for Kline data. 
- **🚫 NO FALLBACKS**: Do NOT use Yahoo Finance, simulated data, or any alternative source
- **🚫 NO SIMULATED DATA**: Never generate fake Kline data - always use real API data
- If iTick API returns empty/error for any ticker:
  - For SPY/QQQ market context: Try different kType or report the issue
  - For individual stocks: Report error to user and ABORT - do NOT use alternatives
- **Correct iTick header format**: `-H "token: <token>"` (NOT `-H "accept: application/json"`)
- If API fails completely: Notify user and STOP - never use fake data

## Workflow

### Step 1: Get Market Context (ALWAYS do this first!)
- Get SPY (S&P 500 ETF) and QQQ (Nasdaq ETF) 1H Kline for market bias
- Endpoint: `https://api.itick.org/stock/kline?region=US&code=SPY&kType=5&limit=50`
- Endpoint: `https://api.itick.org/stock/kline?region=US&code=QQQ&kType=5&limit=50`
- Determine market bias: BULLISH (both above 20 EMA), BEARISH (both below), or MIXED
- **⏸️ PAUSE 15 SECONDS**

### Step 2: Get Stock Name from iTick
- Endpoint: `https://api.itick.org/stock/info?region=US&code=<TICKER>`
- Headers: `-H "accept: application/json" -H "token: a782c443a2a5493d873850758154e14876bfd88dfb89470b8b97dff36e7938e3"`
- Response returns `n` field with stock name (e.g., "NVIDIA Corporation")
- Use this name for news search - NEVER hardcode stock names!
- **⏸️ PAUSE 15 SECONDS**

### Step 3: Get Multiple Timeframes
| Timeframe | kType | Purpose |
|-----------|-------|---------|
| 5min | 2 | Entry timing |
| 15min | 3 | Entry timing |
| 1h | 5 | Intraday trend |
| 4h | 4 | Swing trend |
| 1D | 1 | Daily trend |

- Endpoint: `https://api.itick.org/stock/kline?region=US&code=<ticker>&kType=<kType>&limit=100`
- **⏸️ PAUSE 15 SECONDS after EACH call**

### Step 4: Calculate Technical Indicators
Calculate from Kline data:

**Moving Averages (EMAs):**
- EMA 20 = Exponential moving average of last 20 closes
- EMA 50 = Exponential moving average of last 50 closes
- EMA 200 = Exponential moving average of last 200 closes

**RSI (14):**
```
gain = avg of all up closes in last 14 candles
loss = avg of all down closes in last 14 candles
RSI = 100 - (100 / (1 + gain/loss))
```

**ATR (14):**
```
TR = max(High-Low, |High-PrevClose|, |Low-PrevClose|)
ATR = SMA of last 14 TR values
```

**VWAP (Intraday):**
```
VWAP = Sum(Price * Volume) / Sum(Volume)
Use typical price: (High + Low + Close) / 3
```

**MACD:**
- MACD Line = EMA 12 - EMA 26
- Signal Line = EMA 9 of MACD Line
- Histogram = MACD Line - Signal Line

### Step 5: Calculate Technical Levels
- Current price: Latest candle close (c)
- Previous close: Second-to-last candle close
- Today%: (current - previous) / previous * 100
- **ATR-based stops**: Stop = Entry - (ATR * 1.5) for aggressive, ATR * 2.5 for conservative
- **+3%, +5%, +8%**: Calculate as current * 1.03, 1.05, 1.08
- **-3%, -5%, -8%**: Calculate as current * 0.97, 0.95, 0.92

### Step 6: Get News from Google News RSS
- Use the stock name from Step 2 (e.g., "NVIDIA" not "NVDA")
- Endpoint: `curl -s "https://news.google.com/rss/search?q=<STOCK_NAME>+stock+when:48h&hl=en-US"`
- Parse top 5 headlines from RSS `<item><title>` elements
- No API key needed - free!

### Step 7: AI Sentiment Analysis
Use AI to score headlines from -1 (negative) to +1 (positive).

### Step 8: KLINE PATTERN ANALYSIS (CRITICAL for Day Trading!)
Analyze the actual Kline/candlestick patterns from each timeframe:

**Pattern Recognition (from 5m, 15m, 1h data):**
- **Bullish Patterns**: Hammer, Inverted Hammer, Bullish Engulfing, Morning Star, Three White Soldiers, Higher Highs sequence
- **Bearish Patterns**: Shooting Star, Bearish Engulfing, Evening Star, Three Black Crows, Lower Lows sequence
- **Continuation**: Doji, Spinning Top, Green 3+, Red 3+
- **Breakout**: Strong candle (2%+) with volume on narrow range consolidation

**Analyze last 5 candles per timeframe:**
- Are there higher highs + higher lows? (Bullish sequence)
- Are there lower highs + lower lows? (Bearish sequence)
- Is price consolidating in a tight range? (Potential breakout)
- Is there a gap up/down from previous close?

**Volume Analysis:**
- Is volume increasing on up candles? (Healthy)
- Is volume decreasing on corrections? (Bullish sign)
- Volume spike > 1.5x average = institutional interest

### Step 9: Generate Recommendation with Confidence Score
Use 1H, 4H, Daily Kline, Kline PATTERNS, technical indicators, sentiment, and market context:

```
You are an expert day trader specialising in high-probability setups. Analyse the data and provide a SINGLE trade recommendation with a CONFIDENCE SCORE (1-10).

TRADING STRATEGY FRAMEWORK:
Technical Data: 5m, 15m, 1h, 4h, Daily Kline
Indicators: RSI(14), MACD, EMA(20,50,200), ATR, VWAP
Market Context: SPY/QQQ trend bias
Sentiment: shortTermSentiment score

TRADING STRATEGY:
1. MARKET CONTEXT (CRITICAL - Check first!):
   - If SPY & QQQ BULLISH: Only long setups
   - If SPY & QQQ BEARISH: Only short setups
   - If MIXED: Reduce position size, be selective
   
2. STRICT TREND IDENTIFICATION:
   - 1h: Price above EMA 20 & 50 = BULLISH
   - 1h: Price below EMA 20 &ARISH
   50 = BE - 4h: Confirm trend direction
   - Daily: Weekly trend bias
   
3. HIGH-PROBABILITY ENTRY CRITERIA:
   - Breakout from consolidation (flags, triangles, wedges)
   - Volume 1.5x+ average on breakout
   - RSI 40-65 (bullish) or 35-60 (bearish) - room to run
   - Price >1% away from VWAP
   
4. TECHNICAL CONFLUENCE (Each factor adds +1 to confidence):
   - [ ] Price above EMA 20, 50 (1h)
   - [ ] MACD bullish crossover / bearish crossdown
   - [ ] RSI in favorable range
   - [ ] Volume spike on breakout
   - [ ] Market bias aligns with trade direction
   - [ ] Sentiment positive/negative matches direction
   - [ ] ATR > 2% (good volatility)
   
5. RISK MANAGEMENT (ATR-based):
   - Stop Loss: Entry - (ATR × 2) [conservative]
   - Take Profit: Entry + (ATR × 6) [minimum 1:3 R:R]
   - Position Size: (Portfolio × 1.5%) / (ATR × Entry)
   
6. TIME-OF-DAY FILTER:
   - Best entries: 9:30-11:30 AM ET, 2:00-3:00 PM ET
   - Avoid: Last 30 min of market (reduced liquidity)
   
7. BLACKLIST:
   - Skip if earnings within 7 days
   - Skip on major news days (Fed, CPI, etc.)

ANALYSIS PROCESS:
1. Check SPY/QQQ for market bias
2. Analyze 4H/Daily for trend direction
3. Confirm 1H for entry timing
4. Validate with 15m/5m for precise entry
5. Reject if market bias conflicts with setup

OUTPUT FORMAT:
Return a JSON object:
{
  "recommendation": "BUY/SELL/HOLD",
  "confidence": 1-10,
  "entry": price,
  "stop": price,
  "target": price,
  "rr": "X:1",
  "reasons": ["reason1", "reason2", ...],
  "warnings": ["warning1", ...]
}
```

### Step 9: Calculate Risk:Reward
- Entry: Current price
- Stop: ATR-based (Entry - ATR × 2)
- Target: Entry + (ATR × 6) minimum
- R:R: (Target - Entry) / (Entry - Stop)

## Output Format
```
| Stock | Code | Rec | Conf | Current | +3% | +5% | ATR% | Today% | Sentiment | Entry | Stop | Target | R:R |
|-------|------|-----|------|---------|-----|-----|------|--------|-----------|-------|------|--------|-----|
| NVIDIA | NVDA | BUY | 8/10 | 145.20 | 149.56 | 152.46 | 3.2% | +2.1% | +0.6 | 145.20 | 140.10 | 158.00 | 1.71:1 |
...
```

### Detailed Analysis Example (for top pick)
For the #1 recommendation, provide detailed breakdown:
```
📈 MARKET CONTEXT
| Index | Trend | Price | EMA20 | EMA50 |
|-------|-------|-------|-------|-------|
| SPY | BULLISH | 512.50 | 510.20 | 505.30 |
| QQQ | BULLISH | 445.80 | 442.10 | 438.50 |

📈 5-MINUTE KLINE (Last 5 candles) - PATTERN ANALYSIS
| # | Time | Open | High | Low | Close | Volume | Pattern |
|---|------|------|------|-----|-------|--------|---------|
| 1 | 09:30 | 145.00 | 145.50 | 144.80 | 145.20 | 1.2M | Green - Higher Low |
| 2 | 09:35 | 145.20 | 145.80 | 145.10 | 145.60 | 1.5M | Green - Breakout |
| 3 | 09:40 | 145.60 | 146.00 | 145.40 | 145.90 | 2.1M | Green - Volume Spike |
| 4 | 09:45 | 145.90 | 146.20 | 145.70 | 146.00 | 1.8M | Green - Higher High |
| 5 | 09:50 | 146.00 | 146.50 | 145.80 | 146.30 | 2.3M | Green - Strong Close |

📈 PATTERN SIGNAL: BULLISH - Higher highs + higher lows + volume expansion

📈 15-MINUTE KLINE (Last 5 candles)
...

📈 1-HOUR KLINE (Last 10 candles)
...

📈 4-HOUR KLINE (Last 10 candles)
...

📊 TECHNICAL INDICATORS (1H)
| Indicator | Value | Signal |
|-----------|-------|--------|
| EMA 20 | xxx | BULLISH |
| EMA 50 | xxx | BULLISH |
| RSI(14) | 58.2 | NEUTRAL |
| MACD | +0.25 | BULLISH CROSS |
| ATR | 3.2% | HIGH VOLATILITY |
| VWAP | xxx | ABOVE |

📰 NEWS SENTIMENT
| Headline | Sentiment |
|----------|-----------|
| NVIDIA reports strong earnings | +0.8 |
| ... | ... |
| Overall: +0.65 (POSITIVE)

🎯 BUY RECOMMENDATION - CONFIDENCE: 8/10
| Factor | Analysis | ✓/✗ |
|--------|----------|------|
| Market Bias | SPY/QQQ BULLISH - aligns with long | ✓ |
| Trend (1h) | BULLISH - above EMAs, higher highs | ✓ |
| Trend (4h) | BULLISH - confirmed uptrend | ✓ |
| Momentum | MACD bullish crossover | ✓ |
| Volume | 1.8x average on breakout | ✓ |
| RSI | 58.2 - room to run | ✓ |
| Sentiment | Positive news (+0.65) | ✓ |
| ATR | 3.2% - good volatility | ✓ |

| Parameter | Value |
|-----------|-------|
| Entry | $145.20 |
| Stop Loss | $140.10 (-3.5%, ATR×1.5) |
| Target | $158.00 (+8.8%, ATR×4) |
| R:R | 2.56:1 |

⚠️ WARNINGS:
- Earnings in 12 days
- Already up 5% today - wait for pullback
```

## ⚠️ Rate Limiting (IMPORTANT!)
- iTick Free plan: 5 calls/min → **MUST wait 15 seconds between EACH call**
- Use `sleep 15` or equivalent delay after each API call
- Always get live data, don't use mock data
- Add longer delay in case of rate limiting
- Failure to pause will result in 429 errors (rate limit exceeded)

## Confidence Score Guidelines
| Score | Meaning | Action |
|-------|---------|--------|
| 9-10 | High confidence | Full position |
| 7-8 | Good confidence | Normal position |
| 5-6 | Moderate | Reduced position |
| 3-4 | Low confidence | Small position or skip |
| 1-2 | Skip | Avoid trade |

## Run in Background
Use sessions_spawn for long-running analysis of multiple stocks.