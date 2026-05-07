---
name: stock-quick-check
description: Quick stock check using Yahoo Finance. Get watchlist prices + market overview + detailed news headlines.
---

# Stock Quick Check

Get quick price & daily change from Yahoo Finance, plus market overview and detailed news headlines.

## Stocks
- **HK:** 0700 (Tencent), 2840 (SPDR Gold), 2899 (Zijin)
- **US:** GOOG, TSLA, ALB

## Command

```bash
cd /Users/jaydensmac && python3 -c "
import yfinance as yf

# === WATCHLIST PRICES ===
print('='*60)
print('STOCK WATCHLIST - Daily Performance')
print('='*60)
watchlist = {'0700.HK': 'Tencent', '2840.HK': 'SPDR Gold', '2899.HK': 'Zijin', 'GOOG': 'Google', 'ALB': 'Albemarle', 'TSLA': 'Tesla'}
for sym, name in watchlist.items():
    try:
        t = yf.Ticker(sym)
        h = t.history(period='2d')
        if len(h) >= 2:
            prev = h['Close'].iloc[-2]
            curr = h['Close'].iloc[-1]
            change = curr - prev
            pct = (change/prev)*100
            print(f\"{sym.replace('.HK',''):8s} {name:12s} {curr:>8.2f} {change:>+8.2f} ({pct:>+6.2f}%)\")
    except: pass

# === MARKET OVERVIEW ===
print()
print('='*60)
print('MARKET OVERVIEW')
print('='*60)
indices = {'^HSI': 'HK (HSI)', '^GSPC': 'US (S&P500)', '^DJI': 'US (Dow)', '^IXIC': 'US (Nasdaq)'}
for sym, name in indices.items():
    try:
        t = yf.Ticker(sym)
        h = t.history(period='2d')
        if len(h) >= 2:
            prev = h['Close'].iloc[-2]
            curr = h['Close'].iloc[-1]
            change = curr - prev
            pct = (change/prev)*100
            print(f\"{name:15s} {curr:>10.2f} {change:>+10.2f} ({pct:>+6.2f}%)\")
    except: pass
" 2>/dev/null

# === NEWS (using curl) ===
echo ""
echo "============================================================"
echo "US MARKET NEWS"
echo "============================================================"
curl -s "https://news.google.com/rss/search?q=stock+market&hl=en-US&gl=US&ceid=US:en" | grep -o '<title>[^<]*</title>' | sed 's/<title>//;s/<\/title>//' | grep -v "^Google News$" | head -7

echo ""
echo "============================================================"
echo "HK MARKET NEWS"
echo "============================================================"
curl -s "https://news.google.com/rss/search?q=hong+kong+stock&hl=en-US&gl=US&ceid=US:en" | grep -o '<title>[^<]*</title>' | sed 's/<title>//;s/<\/title>//' | grep -v "^Google News$" | head -7
```

## Usage
Run directly - outputs prices, market indices, and detailed news headlines.
