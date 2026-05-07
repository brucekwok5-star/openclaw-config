---
name: stock-pnl-calculator
description: Calculate profit/loss % by matching sell transactions with buy transactions using FIFO. Input is tabular transaction data with Date, Account, Description, Transaction Type, Symbol, Quantity, Price, Gross Amount, Commission, Net Amount columns. Output shows each sell matched with its buy, including buy date, sell date, and gain/loss %.
---

# Stock P&L Calculator

Calculate profit/loss by matching sell transactions with buy transactions using FIFO (First In, First Out).

## Output Format (ALWAYS INCLUDES DATES)
| Symbol | Buy Date | Sell Date | Buy Price | Sell Price | Qty | P/L | P/L % |
|--------|----------|-----------|-----------|------------|-----|-----|--------|
| 1729 | 2026-02-25 | 2026-02-26 | 19.18 | 19.96 | 1000 | +780.00 | +4.07% |

**Note: Output ALWAYS includes Buy Date and Sell Date columns.**

## Input Format

Tab-separated or space-separated transaction data with these columns:
```
Date | Account | Description | Transaction Type | Symbol | Quantity | Price | Gross Amount | Commission | Net Amount
```

Example:
```
2026-02-26 DUO619108 TIME INTERCONNECT TECHNOLOGY Sell 1729 -1,000.00 19.9600 HKD 19,960.00 -18.00 19,921.43
2026-02-26 DUO619108 TIME INTERCONNECT TECHNOLOGY Buy 1729 1,000.00 20.2200 HKD -20,220.00 -18.00 -20,259.58
```

## How It Works

### Step 1: Parse Transactions
Parse each line to extract:
- Date (YYYY-MM-DD)
- Symbol (stock code or ticker)
- Transaction Type (Buy/Sell)
- Quantity (positive for Buy, negative for Sell)
- Price
- Net Amount

### Step 2: Match Sell to Buy (FIFO)
For each SELL transaction:
1. Group all buys by symbol
2. Find earliest unmatched buy (FIFO)
3. Calculate P/L: (Sell Price - Buy Price) × Quantity
4. Calculate P/L %: ((Sell Price - Buy Price) / Buy Price) × 100

### Step 3: Output Results
Sort by sell date descending and output table:
| Symbol | Buy Date | Sell Date | Buy Price | Sell Price | Qty | P/L | P/L % |
|--------|----------|-----------|-----------|------------|-----|-----|--------|
| 1729 | 2026-02-25 | 2026-02-26 | 19.18 | 19.96 | 1000 | +780 | +4.07% |

## Example

Input:
```
2026-02-26 DUO619108 TIME INTERCONNECT TECHNOLOGY Sell 1729 -1,000.00 19.9600 HKD 19,960.00 -18.00 19,921.43
2026-02-26 DUO619108 HSBC HOLDINGS PLC Sell 5 -400.00 147.4000 HKD 58,960.00 -47.17 58,852.15
2026-02-26 DUO619108 TIME INTERCONNECT TECHNOLOGY Buy 1729 1,000.00 20.2200 HKD -20,220.00 -18.00 -20,259.58
2026-02-25 DUO619108 TIME INTERCONNECT TECHNOLOGY Buy 1729 1,000.00 19.1800 HKD -19,180.00 -18.00 -19,218.55
```

Output:
```
=== Stock P&L (Sorted by Sell Date Desc) ===

| Symbol | Buy Date | Sell Date | Buy Price | Sell Price | Qty | P/L | P/L % |
|--------|----------|-----------|-----------|------------|-----|-----|--------|
| 1729 | 2026-02-25 | 2026-02-26 | 19.18 | 19.96 | 1000 | +780.00 | +4.07% |
| 5 | 2026-02-25 | 2026-02-26 | 139.40 | 147.40 | 400 | +3,200.00 | +5.74% |

Total P/L: +3,980.00 HK$
```

## Python Script

```python
#!/usr/bin/env python3
import re
import sys
from collections import defaultdict

def parse_transactions(data):
    """Parse transaction data into structured records."""
    transactions = []
    
    for line in data.strip().split('\n'):
        line = line.strip()
        if not line:
            continue
        
        # Skip header-like lines or FX adjustments
        if 'Date' in line or 'FX Translations' in line or 'Debit Interest' in line or 'Credit Interest' in line or 'Payment in Lieu' in line:
            continue
        
        # Parse: Date Account Description Transaction Type Symbol Quantity Price Currency Gross Commission Net
        # Example: 2026-02-26 DUO619108 TIME INTERCONNECT TECHNOLOGY Sell 1729 -1,000.00 19.9600 HKD 19,960.00 -18.00 19,921.43
        parts = line.split()
        if len(parts) < 11:
            continue
        
        # Extract date (first)
        date = parts[0]
        
        # Transaction type is after description (3rd index after date and account)
        # Find the Transaction Type (Buy/Sell)
        type_idx = None
        for i, p in enumerate(parts):
            if p in ['Buy', 'Sell', 'Payment', 'Debit', 'Credit']:
                type_idx = i
                break
        
        if type_idx is None:
            continue
            
        trans_type = parts[type_idx]
        
        # Skip non-trade transactions
        if trans_type not in ['Buy', 'Sell']:
            continue
        
        # Symbol is after transaction type
        symbol = parts[type_idx + 1]
        
        # Quantity is after symbol
        # Remove commas and convert to float
        qty_str = parts[type_idx + 2].replace(',', '')
        qty = float(qty_str)
        
        # Price is after quantity
        price_str = parts[type_idx + 3].replace(',', '')
        price = float(price_str)
        
        # Currency
        currency = parts[type_idx + 4]
        
        # Net amount (last field)
        net_str = parts[-1].replace(',', '')
        net = float(net_str)
        
        transactions.append({
            'date': date,
            'symbol': symbol,
            'type': trans_type,
            'qty': abs(qty),
            'price': price,
            'currency': currency,
            'net': net
        })
    
    return transactions

def calculate_pnl(transactions):
    """Calculate P/L using FIFO matching."""
    # Group by symbol
    buys_by_symbol = defaultdict(list)
    sells_by_symbol = defaultdict(list)
    
    for t in transactions:
        if t['type'] == 'Buy':
            buys_by_symbol[t['symbol']].append(t)
        elif t['type'] == 'Sell':
            sells_by_symbol[t['symbol']].append(t)
    
    # Sort buys by date (oldest first - FIFO)
    for symbol in buys_by_symbol:
        buys_by_symbol[symbol].sort(key=lambda x: x['date'])
    
    # Sort sells by date (newest first for output)
    for symbol in sells_by_symbol:
        sells_by_symbol[symbol].sort(key=lambda x: x['date'], reverse=True)
    
    results = []
    used_buys = set()  # Track (symbol, date, price, qty) used
    
    # Process each sell
    all_sells = []
    for symbol, sells in sells_by_symbol.items():
        for sell in sells:
            all_sells.append((symbol, sell))
    
    # Sort all sells by date descending
    all_sells.sort(key=lambda x: x[1]['date'], reverse=True)
    
    for symbol, sell in all_sells:
        remaining_qty = sell['qty']
        
        # Find matching buy (FIFO)
        for buy in buys_by_symbol[symbol]:
            if remaining_qty <= 0:
                break
            
            # Check if this buy is already fully used
            buy_key = (buy['date'], buy['price'], buy['qty'])
            if buy_key in used_buys:
                continue
            
            # Skip if buy is after sell (can't sell what you haven't bought)
            if buy['date'] > sell['date']:
                continue
            
            # Calculate how much we can match
            used_qty = min(remaining_qty, buy['qty'])
            
            # Calculate P/L
            pl = (sell['price'] - buy['price']) * used_qty
            pl_pct = ((sell['price'] - buy['price']) / buy['price']) * 100
            
            results.append({
                'symbol': symbol,
                'buy_date': buy['date'],
                'sell_date': sell['date'],
                'buy_price': buy['price'],
                'sell_price': sell['price'],
                'qty': int(used_qty),
                'pl': pl,
                'pl_pct': pl_pct,
                'currency': sell['currency']
            })
            
            remaining_qty -= used_qty
            used_buys.add(buy_key)
    
    # Sort by sell date descending
    results.sort(key=lambda x: x['sell_date'], reverse=True)
    
    return results

def format_output(results):
    """Format results as table."""
    if not results:
        return "No matched transactions found."
    
    # Group by currency
    hkd_total = 0
    usd_total = 0
    
    output = "=== Stock P&L (Sorted by Sell Date Desc) ===\n\n"
    output += "| Symbol | Buy Date | Sell Date | Buy Price | Sell Price | Qty | P/L | P/L % |\n"
    output += "|--------|----------|-----------|-----------|------------|-----|-----|--------|\n"
    
    for r in results:
        pl_str = f"+{r['pl']:.2f}" if r['pl'] >= 0 else f"{r['pl']:.2f}"
        pct_str = f"+{r['pl_pct']:.2f}%" if r['pl_pct'] >= 0 else f"{r['pl_pct']:.2f}%"
        
        output += f"| {r['symbol']} | {r['buy_date']} | {r['sell_date']} | {r['buy_price']:.4f} | {r['sell_price']:.4f} | {r['qty']} | {pl_str} | {pct_str} |\n"
        
        if r['currency'] == 'HKD':
            hkd_total += r['pl']
        else:
            usd_total += r['pl']
    
    output += "\n"
    if hkd_total != 0:
        hkd_str = f"+{hkd_total:.2f} HK$" if hkd_total >= 0 else f"{hkd_total:.2f} HK$"
        output += f"Total HK$ P/L: {hkd_str}\n"
    if usd_total != 0:
        usd_str = f"+{usd_total:.2f} US$" if usd_total >= 0 else f"{usd_total:.2f} US$"
        output += f"Total US$ P/L: {usd_str}\n"
    
    return output

if __name__ == "__main__":
    # Read from stdin
    data = sys.stdin.read()
    transactions = parse_transactions(data)
    results = calculate_pnl(transactions)
    print(format_output(results))
