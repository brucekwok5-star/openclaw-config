---
name: stock-analysis-us
description: Run US stock analysis using custom script at /Users/jaydensmac/stock-analysis/stock_analysis.py us --signals
---

# Stock Analysis US

Run the US stock analysis script with --signals flag (shows only BUY/SELL recommendations, runs in background).

## Data Sources
- **Klines:** iTick API
- **News:** NewsAPI
- **Output:** Stock list, BUY/SELL signals, technical analysis

## Command

```bash
cd /Users/jaydensmac/stock-analysis && python3 stock_analysis.py us --signals
```

## Usage

When user selects this skill, execute the command above in background.
Only outputs: stock list, BUY/SELL signals, final JSON file.
