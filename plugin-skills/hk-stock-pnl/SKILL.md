---
name: hk-stock-pnl
description: Calculate profit/loss for HK stock transactions from PDF brokerage statements. Use when user provides a PDF transaction statement (e.g., DUO619108.TRANSACTIONS.MTD.pdf) and wants to find sell transactions with related buy transactions, profit/loss %, and dates.
---

# HK Stock P&L Calculator

Calculate profit/loss for Hong Kong stock transactions from PDF brokerage statements.

## Input
- PDF file containing stock transaction history (e.g., `DUO619108.TRANSACTIONS.MTD.pdf`)
- Typically from Hong Kong brokers (e.g., Duo, Bright Smart, etc.)

## How It Works

### Step 1: Extract PDF Text
Use `pdfplumber` to extract text from PDF:
```python
import pdfplumber

with pdfplumber.open(pdf_path) as pdf:
    text = "\n".join(page.extract_text() for page in pdf.pages)
```

### Step 2: Parse Transactions
Extract transaction rows from text. Look for patterns like:
- Date (DD/MM/YYYY)
- Stock code (5 digits)
- Buy/Sell indicator
- Quantity
- Price
- Amount

### Step 3: Match Sell to Buy Transactions
For each SELL transaction:
1. Find matching stock code
2. Find earliest UNMATCHED BUY for that stock (FIFO)
3. Calculate: P/L = (Sell Price - Buy Price) × Quantity

### Step 4: Output Results
Generate table:
| Stock | Buy Date | Sell Date | Buy Price | Sell Price | Qty | P/L (HK$) | P/L % |
|-------|----------|-----------|-----------|------------|-----|-----------|-------|
| 0700 | 15/01/2026 | 20/02/2026 | 500.00 | 520.00 | 1000 | +20,000 | +4.0% |

## Example Output Format
```
=== Stock P&L Summary ===

| Stock | Buy Date | Sell Date | Buy | Sell | Qty | P/L | P/L % |
|-------|----------|-----------|-----|------|-----|-----|-------|
| 0700 | 15/01 | 20/02 | 500 | 520 | 1000 | +20,000 | +4.0% |
| 9988 | 10/02 | 25/02 | 140 | 138 | 500 | -1,000 | -1.4% |

Total P/L: +19,000 HK$
```

## Notes
- Use FIFO (First In, First Out) matching
- Only match BUY → SELL (not vice versa)
- Handle multiple lots of same stock
- PDF formats vary by broker - adapt parsing as needed
