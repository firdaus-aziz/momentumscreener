# Risk Management Framework

Established approaches to position sizing and risk control.

## Philosophy

Per the [project vision](../PROJECT_VISION.md):

- Use **established frameworks** over custom solutions
- Risk management **evolves with automation phases**
- **Preserve capital** in early phases - profits come later
- Never risk more than can be recovered from

## Phase-Specific Limits

Risk tolerance increases as system proves itself:

| Phase | Max Risk/Trade | Max Daily Loss | Max Positions |
|-------|---------------|----------------|---------------|
| Paper Trading | N/A | N/A | Unlimited |
| Manual Execute | 5% | 10% | 2-3 |
| One-Click | 2% | 6% | 3-4 |
| Auto + Approval | 2% | 5% | 4-5 |
| Full Auto | 1-2% | 4% | 5-6 |

## Position Sizing Methods

### 1. Fixed Dollar Amount (Current)

Simplest approach. Good for learning phase.

```
Position size = Fixed amount (e.g., $100, $300, $500)
```

**Pros:** Simple, predictable
**Cons:** Doesn't account for volatility or account growth

**Use in:** Paper trading, early Manual Execute phase

### 2. Fixed Fractional

Risk a fixed percentage of account per trade.

```
Position size = Account balance × Risk percentage
Risk amount = Position size × Stop-loss percentage

Example:
- Account: $1,100
- Risk per trade: 2%
- Stop-loss: 10%
- Risk amount = $1,100 × 2% = $22
- Position size = $22 / 10% = $220
```

**Pros:** Scales with account, limits maximum loss
**Cons:** Requires stop-loss discipline

**Use in:** One-Click phase and beyond

### 3. Kelly Criterion

Optimal position sizing based on win rate and reward/risk.

```
Kelly % = W - [(1-W) / R]

Where:
W = Win rate (e.g., 0.55 for 55%)
R = Reward/Risk ratio (e.g., 2.0 for 2:1)

Example:
Kelly % = 0.55 - [(1-0.55) / 2.0]
Kelly % = 0.55 - 0.225
Kelly % = 0.325 (32.5%)
```

**Important:** Use **Half-Kelly** (16.25% in this example) for safety margin.

**Pros:** Mathematically optimal for growth
**Cons:** Requires accurate win rate and R:R estimates, volatile

**Use in:** Full Auto phase with proven statistics

### 4. Volatility-Based (ATR)

Adjust position size based on stock volatility.

```
Position size = Risk amount / (ATR × Multiplier)

Example:
- Risk amount: $22
- ATR (14-day): $0.50
- Multiplier: 2
- Position size = $22 / ($0.50 × 2) = 22 shares
```

**Pros:** Accounts for individual stock volatility
**Cons:** Requires ATR calculation

**Use in:** Auto + Approval phase

## Stop-Loss Strategies

### 1. Fixed Percentage

```
Stop price = Entry price × (1 - Stop %)

Example:
- Entry: $10.00
- Stop: 10%
- Stop price: $10.00 × 0.90 = $9.00
```

**Recommended stops by phase:**
- Manual: 10-15% (learning buffer)
- One-Click: 8-12%
- Auto: 6-10%

### 2. ATR-Based

```
Stop price = Entry price - (ATR × Multiplier)

Example:
- Entry: $10.00
- ATR: $0.50
- Multiplier: 2
- Stop price: $10.00 - $1.00 = $9.00
```

Adapts to volatility of individual stocks.

### 3. EMA-Based (Current System)

```
Exit when: Price < 25 EMA

Example:
- Current price: $10.50
- 25 EMA: $10.60
- Action: Exit position
```

Technical exit, currently implemented in paper trading.

## Daily Loss Limits

Circuit breakers to prevent catastrophic days:

```python
# Pseudo-code
if daily_loss >= max_daily_loss:
    pause_trading()
    send_alert("Daily loss limit reached")
    resume_next_day()
```

**Recommended limits:**
- Manual: 10% of account
- One-Click: 6% of account
- Auto: 4% of account

## Portfolio Heat

Maximum total risk across all open positions:

```
Portfolio heat = Sum of (Position risk × Position count)

Example:
- 3 positions, each risking 2%
- Portfolio heat = 6%

Maximum recommended: 6-10%
```

If portfolio heat exceeds limit, no new positions until existing ones close.

## Market Condition Adjustment

The system includes real-time market scoring. Adjust risk based on conditions:

| Market Score | Position Size Multiplier |
|-------------|-------------------------|
| 80-100 (Excellent) | 1.5-2.0x |
| 65-79 (Good) | 1.0x (normal) |
| 45-64 (Fair) | 0.5x |
| 0-44 (Poor) | 0x (no trades) |

Currently implemented in `market_sentiment_scorer.py`.

## PDT Rule Considerations

Pattern Day Trader rule applies to US accounts under $25,000:

- Maximum 3 day trades per 5 rolling business days
- Day trade = Buy and sell same stock same day

**With $1,100 starting capital:**
- Limited to 3 round-trips per week
- Prioritize best setups only
- Consider swing trades (hold overnight) to avoid PDT

## Concentration Limits

Avoid overexposure to single stocks or sectors:

```
# Current implementation
max_ticker_concentration = 20%    # Max alerts from one ticker
max_sector_concentration = 30%    # Max alerts from one sector
```

**For positions:**
- No more than 50% of capital in one position
- No more than 70% in one sector

## Implementation Checklist

### Paper Trading Phase (Current)
- [ ] Fixed $100 positions
- [ ] EMA-based exits
- [ ] Track win rate and R:R
- [ ] Document drawdowns

### Manual Execute Phase (Feb 2026)
- [ ] Fixed 5% risk per trade
- [ ] 10% stop-losses
- [ ] 10% daily loss limit
- [ ] Maximum 2-3 positions
- [ ] Respect PDT rule

### One-Click Phase
- [ ] Implement fixed fractional sizing
- [ ] ATR-based stop-losses
- [ ] 6% daily loss limit
- [ ] Market condition multipliers

### Auto Phases
- [ ] Half-Kelly position sizing
- [ ] Volatility-adjusted sizing
- [ ] Automated circuit breakers
- [ ] Portfolio heat monitoring

## Resources

Established frameworks to study:

1. **Van Tharp** - Position sizing and expectancy
2. **Ralph Vince** - Optimal f and Kelly criterion
3. **Elder** - 2% and 6% rules
4. **O'Neil (CAN SLIM)** - Maximum 8% stop-loss

## Key Principles

1. **Never risk more than 2% per trade** (after learning phase)
2. **Daily loss limit is sacred** - stop when hit
3. **Position size before entry** - know your exit
4. **Market conditions matter** - reduce in poor conditions
5. **Start small, scale with success** - earn the right to size up
