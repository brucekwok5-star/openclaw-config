---
name: stock-market-movers
description: Get top 10 most active stocks by TURNOVER VALUE ($) for HKEX and US markets. Uses Yahoo Finance for all data. Output includes comma-separated stock codes (HK codes without leading zeros). Excludes ETFs, warrants, derivatives.
---

# Stock Market Movers

Get top 10 most active stocks by turnover for Hong Kong and US markets.

## ⚠️ Data Delay Notice
- **HK Market (Yahoo Finance)**: ~15 minute delay
- **US Market (Yahoo Finance)**: ~15-20 minute delay
- For real-time data, use a brokerage account

## Usage

When skill is run, check BOTH HK and US markets:

1. Get HK top 10 stocks
2. Get US top 10 stocks
3. Output both tables

## HKEX (Hong Kong) Top 10 by Turnover (Filtered)

### Data Source
- Yahoo Finance HK: https://hk.finance.yahoo.com/markets/stocks/most-active/

### Filters Applied
1. Get top 100 stocks by volume
2. **Filter: Price > HK$5**
3. **Filter: Today price increase (positive change %)**
4. **Sort by TURNOVER VALUE** (volume × price)
5. Pick top 10

### Output Format

**Stock codes (comma-separated, NO leading zeros):**
```
6869,1888,857,2338,992,1138,390,3323,386,1729
```

**Table:**
| Rank | Symbol | Price (HKD) | Change | Turnover (HKD) |
|------|--------|-------------|--------|--------|
| 1 | 6869.HK | 132.50 | +3.76% | H$2.1B |
| 2 | 1888.HK | 22.60 | +10.89% | H$599M |
| ... | ... | ... | ... | ... |

### Rules
- Get top 100 by volume first (API: count=100)
- Filter: `regularMarketPrice > 5` (HK$5)
- Filter: `regularMarketChangePercent > 0` (up today)
- **Sort by TURNOVER VALUE (volume × price in HKD)**
- Calculate turnover: `regularMarketVolume × regularMarketPrice`
- Focus on ordinary shares only (exclude ETFs, warrants, derivatives)
- Flag warrants if they appear (e.g., SENSETIME-W)
- Use M for million, B for billion (turnover in HKD)
- **ALWAYS output stock code list FIRST (comma-separated, NO leading zeros)**
- **Example: 6869,1888,857,2338,992,1138,390,3323,386,1729**
- **State data is delayed ~15 min**

### Turnover Calculation
Since Yahoo Finance API returns volume and price separately, calculate turnover:
```
Turnover (HKD) = regularMarketVolume × regularMarketPrice
```
- Format turnover as HKD (e.g., H$386.5M, H$1.2B)

## US (NYSE/NASDAQ) Top 10 by Turnover

### Data Source
- Yahoo Finance: https://finance.yahoo.com/most-active

### Output Format

**Stock symbols (comma-separated):**
```
TSLA,AAPL,NVDA,AMD,MSFT,META,GOOGL,AMZN,BABA,NIO
```

**Table:**
| Rank | Symbol | Price (USD) | Change | Turnover |
|------|--------|-------------|--------|----------|
| 1 | AAPL | 245.32 | +1.23% | $21.98B |
| ... | ... | ... | ... | ... |

### Rules
- **Sort by TURNOVER VALUE (volume × price in USD), not by share volume**
- Calculate turnover: `regularMarketVolume × regularMarketPrice`
- Focus on common stock only (exclude ETFs, funds, warrants)
- Use M for million, B for billion (turnover in USD)
- **Always output symbol list FIRST**
- **State data is delayed 15-20 min**

## API Commands

### HK Market
```bash
# Get top 100 by volume, then filter & sort
curl -s -A 'Mozilla/5.0' \
  'https://query1.finance.yahoo.com/v1/finance/screener/predefined/saved?scrIds=MOST_ACTIVES_HK&count=100&fields=symbol,regularMarketPrice,regularMarketChangePercent,regularMarketVolume&lang=en&region=US'
```
**Processing:**
1. Filter: `price > 5` AND `change > 0`
2. Calculate turnover: `volume × price`
3. Sort by turnover (descending)
4. Take top 10

### US Market
```bash
curl -s -A 'Mozilla/5.0' \
  'https://query1.finance.yahoo.com/v1/finance/screener/predefined/saved?scrIds=MOST_ACTIVE_US&count=10&fields=symbol,regularMarketPrice,regularMarketChangePercent,regularMarketVolume&lang=en&region=US'
```

## Output Template

```
=== HK Top 10 Stocks (Price > $5 + Up Today) ===

Stock codes: 6869,1888,857,2338,992,1138,390,3323,386,1729

| Rank | Symbol | Price (HKD) | Change | Turnover (HKD) |
|------|--------|-------------|--------|--------|
| 1 | 6869.HK | 132.50 | +3.76% | H$2.1B |
| 2 | 1888.HK | 22.60 | +10.89% | H$599M |
| 3 | 0857.HK | 9.58 | +0.10% | H$623M |
...

=== US Top 10 Stocks (Price > $5 + Up Today) ===

Stock symbols: NVDA,NFLX,PLTR,TSLA,AMD,META,GOOGL,AMZN,MSFT,INTC

| Rank | Symbol | Price (USD) | Change | Turnover |
|------|--------|-------------|--------|----------|
| 1 | NVDA | 195.62 | +1.44% | $34.9B |
| 2 | NFLX | 82.71 | +5.98% | $5.0B |
| 3 | PLTR | 134.19 | +4.15% | $6.3B |
...

⚠️ Data delayed ~15-20 min (Yahoo Finance)
Filters: Price > $5, Today Up only
Sorted by: Turnover Value
```

## Important Notes
- **When run, ALWAYS check BOTH HK and US markets**
- **Get top 100 by volume first, then filter and sort**
- **Filter: Price > HK$5 (HK) / $5 (US) AND Change > 0**
- **Sort by TURNOVER VALUE (volume × price), NOT just share volume**
- **Always output stock code/symbol list FIRST (comma-separated)**
- **Remove leading zeros from HK stock codes** (e.g., 0020 → 20, 0992 → 992)
- NO analysis, commentary, or opinion
- State data delays clearly
- Use consistent formatting for all numbers
