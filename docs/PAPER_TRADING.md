# Paper Trading Guide

Validate the momentum strategy before risking real capital.

## Purpose

Per the [project vision](../PROJECT_VISION.md), paper trading is a critical validation step:

- **Minimum duration:** 1 month
- **Goal:** Confirm positive expectancy of alert-based strategy
- **Exit criteria:** Confidence in system before Phase 2 (Manual Execute)

## Strategy Rules

### Entry
- Alert triggered by momentum screener
- Current price > 5-minute 9 EMA

### Exit
- Current price < 5-minute 25 EMA

### Position Size
- Fixed $100 per trade (configurable via RiskProfile)
- No pyramiding (one position per ticker)

## Implementation

### C# Trading Engine

```csharp
public class TradingEngine : ITradingEngine
{
    public bool ShouldEnterPosition(Alert alert, MarketCondition condition)
    {
        // Check market conditions
        if (condition.Category == MarketConditionCategory.Poor)
            return false;

        // Check EMA condition
        decimal ema9 = CalculateEma(alert.Ticker, 9);
        return alert.Price > ema9;
    }

    public bool ShouldExitPosition(Position position, decimal currentPrice)
    {
        decimal ema25 = CalculateEma(position.Ticker, 25);
        return currentPrice < ema25;
    }
}
```

### Database Entities

Paper trades are stored using the same `Position` and `Trade` entities:

```csharp
// Position created on entry
Position position = new()
{
    Ticker = alert.Ticker,
    EntryPrice = alert.Price,
    Shares = positionSize / alert.Price,
    EntryTime = DateTime.UtcNow,
    Status = PositionStatus.Open,
    AlertId = alert.Id
};

// Trade created on exit
Trade trade = new()
{
    Ticker = position.Ticker,
    EntryPrice = position.EntryPrice,
    ExitPrice = currentPrice,
    Shares = position.Shares,
    EntryTime = position.EntryTime,
    ExitTime = DateTime.UtcNow,
    ProfitLoss = (currentPrice - position.EntryPrice) * position.Shares,
    ExitReason = TradeExitReason.EmaExit,
    PositionId = position.Id
};
```

## Dashboard Monitoring

### Paper Trading View

The Blazor dashboard shows:

- **Active Positions:** Current open paper trades
- **Trade History:** Completed paper trades with P&L
- **Performance Metrics:** Win rate, total P&L, average return

### Metrics Tracked

| Metric | Calculation |
|--------|-------------|
| Win Rate | Winning trades / Total trades |
| Total P&L | Sum of all trade P&L |
| Avg Return | Average profit percent per trade |
| Max Drawdown | Largest peak-to-trough decline |
| Avg Hold Time | Average position duration |

## Configuration

### RiskProfile for Paper Trading

```csharp
RiskProfile paperTradingProfile = new()
{
    Phase = AutomationPhase.PaperTrading,
    MaxRiskPerTrade = 1.00m,      // 100% (no limit for paper)
    MaxDailyLoss = 1.00m,         // 100% (no limit for paper)
    MaxPositions = 10,            // Allow many for testing
    IsActive = true
};
```

### appsettings.json

```json
{
  "Trading": {
    "CurrentPhase": "PaperTrading",
    "DefaultPositionSize": 100.00,
    "ScanIntervalMinutes": 2
  }
}
```

## Validation Period

### What to Monitor

**Week 1-2:**
- Are alerts generating?
- Are entries firing at reasonable prices?
- Are exits happening before major drawdowns?

**Week 3-4:**
- Overall P&L trend
- Win rate stabilizing?
- Patterns in losers (sector, time, alert type)

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
4. **Market conditions** - Only trade when score >65

## Quick Reference

### View Paper Trading Results

Access via Blazor dashboard at `/positions` and `/trades`.

### Key Database Queries

```csharp
// Get open positions
IEnumerable<Position> openPositions = await _context.Positions
    .Where(p => p.Status == PositionStatus.Open)
    .ToListAsync();

// Get today's trades
IEnumerable<Trade> todayTrades = await _context.Trades
    .Where(t => t.ExitTime.Date == DateTime.Today)
    .ToListAsync();

// Calculate win rate
decimal winRate = (decimal)trades.Count(t => t.ProfitLoss > 0) / trades.Count();
```

## Next Steps

After successful paper trading validation:

1. Review performance metrics thoroughly
2. Document learnings and patterns
3. Update RiskProfile for Manual Execute phase
4. Prepare for Phase 2 with real capital ($1,100)

See [PROJECT_VISION.md](../PROJECT_VISION.md) for full automation roadmap.
