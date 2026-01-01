# Paper Trading Guide

Validate the momentum strategy before risking real capital.

## Purpose

Per the [project vision](../PROJECT_VISION.md), paper trading is a critical validation step before live trading:

- **Minimum duration:** 1 month
- **Goal:** Confirm positive expectancy of alert-based strategy
- **Exit criteria:** Confidence in system before Phase 2 (Manual Execute)

## Strategy Rules

### Entry
- Alert triggered by momentum tracker
- Current price > 5-minute 9 EMA

### Exit
- Current price < 5-minute 25 EMA

### Position Size
- Fixed $100 per trade (configurable)
- No pyramiding (one position per ticker)

## Quick Start

```bash
# Activate environment
source venv/bin/activate

# Run with paper trading enabled
python volume_momentum_tracker.py --continuous --paper-trading \
  --bot-token "YOUR_TOKEN" \
  --chat-id "YOUR_CHAT_ID"

# Single scan with paper trading
python volume_momentum_tracker.py --single --paper-trading

# View performance report
python volume_momentum_tracker.py --paper-report
```

## How It Works

### Trade Flow

1. **Alert fires** - Momentum tracker detects pattern
2. **Entry check** - Is price > 9 EMA?
   - Yes → Buy $100 worth of shares
   - No → Skip entry
3. **Position tracked** - Entry price, shares, time logged
4. **Exit monitoring** - Each scan checks if price < 25 EMA
5. **Exit executed** - Position closed, P&L calculated

### EMA Calculation

```python
# 9 EMA - faster, for entry signals
ema_9 = prices.ewm(span=9, adjust=False).mean()

# 25 EMA - slower, for exit signals
ema_25 = prices.ewm(span=25, adjust=False).mean()

# Uses 5-minute price data
# Requires minimum 9/25 data points respectively
```

## Data Storage

```
momentum_data/paper_trades/
├── trade_history.json      # Completed trades
└── active_positions.json   # Currently held positions
```

### Trade Record Format

```json
{
  "ticker": "ABCD",
  "entry_price": 10.50,
  "exit_price": 10.80,
  "shares": 9.5238,
  "entry_timestamp": "2025-09-06T10:30:00",
  "exit_timestamp": "2025-09-06T11:15:00",
  "holding_time_minutes": 45,
  "position_size": 100,
  "exit_value": 102.86,
  "profit_loss": 2.86,
  "profit_pct": 2.86,
  "alert_type": "price_spike",
  "exit_reason": "EMA_EXIT"
}
```

## Performance Metrics

### Report Output

```
PAPER TRADING PERFORMANCE REPORT
==================================================

ACCOUNT SUMMARY:
   Initial Balance: $10,000.00
   Current Balance: $10,150.30
   Total Return: +1.50%

TRADING STATISTICS:
   Total Trades: 25
   Win Rate: 64.0%
   Winning Trades: 16
   Losing Trades: 9

PROFIT/LOSS:
   Total P&L: $+150.30
   Average Return: +0.60%
   Best Trade: $+15.20
   Worst Trade: $-8.50

TIMING:
   Avg Holding Time: 45.2 minutes
   Active Positions: 3
```

### Key Metrics to Track

| Metric | Target | Why |
|--------|--------|-----|
| Win rate | >40% | Minimum for profitable expectancy |
| Avg winner vs loser | >2:1 | Ensures profitable even at 40% win rate |
| Max drawdown | <20% | Validates risk management |
| Avg holding time | Document | Informs exit strategy tuning |

## Validation Period

### What to Monitor

**Week 1-2:**
- Is the system generating trades?
- Are entries firing at reasonable prices?
- Are exits happening before major drawdowns?

**Week 3-4:**
- Overall P&L trend (up, down, flat?)
- Win rate stabilizing?
- Any patterns in losers? (sector, time of day, alert type)

### Success Criteria

Before moving to Phase 2 (Manual Execute):

1. **Positive P&L** - Any profit, even small
2. **Reasonable win rate** - >40% with good risk/reward
3. **Controlled drawdowns** - No catastrophic losses
4. **Pattern insights** - Understanding of what works

### Failure Modes

If paper trading shows losses:

1. **Adjust EMA periods** - Try 12/26 instead of 9/25
2. **Add confirmation** - Require volume spike with price
3. **Filter alert types** - Focus only on flat-to-spike
4. **Market conditions** - Only trade in favorable conditions (score >65)

## Configuration

### PaperTradingSystem Parameters

```python
PaperTradingSystem(
    initial_balance=10000,    # Virtual starting capital
    position_size=100,        # Dollars per trade
    data_dir="paper_trades"   # Storage location
)
```

### Modifying Strategy

Current strategy is simple EMA crossover. Future enhancements could include:

- Stop-loss integration
- Profit targets
- Time-based exits
- Market condition filters
- Position sizing based on volatility

## Log Messages

```
PAPER TRADE ENTRY: ABCD at $11.50
PAPER TRADE EXIT: ABCD - P&L: $+2.50 (+2.50%)
AUTO EXIT: XYZ - P&L: $-1.20 (-1.20%)
ENTRY SIGNAL: ABCD at $11.50 > 9EMA $11.35
EXIT SIGNAL: ABCD at $11.20 < 25EMA $11.40
NO ENTRY: ABCD at $11.20 <= 9EMA $11.35
```

## Troubleshooting

### No trades executing
- Check EMA data availability (need 25+ price points)
- Verify alerts are firing
- Ensure `--paper-trading` flag is set

### Frequent whipsaws
- Market may be too choppy for EMA strategy
- Consider longer EMA periods
- Add confirmation requirements

### Poor performance
- EMA crossover struggles in sideways markets
- Consider market condition filter
- Focus on flat-to-spike alerts only

## Next Steps

After successful paper trading validation:

1. Review performance report thoroughly
2. Document learnings and patterns
3. Prepare for Phase 2 (Manual Execute) with real capital
4. Start small ($300-500 positions max with $1,100 capital)

See [PROJECT_VISION.md](../PROJECT_VISION.md) for full automation roadmap.
