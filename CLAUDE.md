# Momentum Screener - Claude Code Context

## Project Summary

Momentum trading system for US small-cap stocks.

**Architecture:** C# .NET 8 (primary) + Python (secondary)
**UI:** Blazor Server + MudBlazor
**Database:** SQLite + EF Core
**Status:** Implementation in progress

---

## Quick Reference

### Solution Structure

```
momentum-screener/
├── src/
│   ├── MomentumScreener.Core/           # Domain (entities, interfaces)
│   ├── MomentumScreener.Infrastructure/ # Data, services, integrations
│   └── MomentumScreener.Web/            # Blazor Server UI
├── tests/
│   ├── MomentumScreener.Core.Tests/
│   └── MomentumScreener.Integration.Tests/
├── tools/py/                            # Python data tools
└── data/                                # SQLite database
```

### Key Commands

```bash
# Build
dotnet build

# Run web app
dotnet run --project src/MomentumScreener.Web

# Run with hot reload
dotnet watch --project src/MomentumScreener.Web

# Run tests
dotnet test

# EF migrations
dotnet ef migrations add Name --project src/MomentumScreener.Infrastructure --startup-project src/MomentumScreener.Web
dotnet ef database update --project src/MomentumScreener.Infrastructure --startup-project src/MomentumScreener.Web

# Python setup
source .venv/bin/activate
pip-sync requirements.txt
```

---

## Implementation Progress

**Always check `IMPLEMENTATION_PLAN.md` first** for current phase, checklist status, and session continuity notes.

| Phase | Focus |
|-------|-------|
| 1 | Infrastructure (solution, projects, build) |
| 2 | Core Domain (entities, enums, interfaces) |
| 3 | Data Layer (SQLite, EF Core, repos) |
| 4 | Python Bridge (C# ↔ Python integration) |
| 5 | Services (trading engine, risk, telegram) |
| 6 | Blazor UI (dashboard, pages) |
| 7 | Integration (wire together) |
| 8 | Testing (unit, integration) |
| 9 | Migration (cleanup, archive) |

---

## Domain Model

### Core Entities

| Entity | Purpose |
|--------|---------|
| `Alert` | Momentum alert from screener |
| `Position` | Open or closed trading position |
| `Trade` | Completed trade record |
| `RiskProfile` | Risk parameters per phase |
| `MarketCondition` | Market sentiment score |

### Alert Types

```csharp
enum AlertType
{
    FlatToSpike,      // Consolidation → breakout (highest success)
    PriceSpike,       // Intraday price surge
    VolumeClimber,    // Rising in volume rankings
    VolumeNewcomer,   // New high-volume entry
    PremarketPrice,   // Premarket price movement
    PremarketVolume   // Premarket volume surge
}
```

### Automation Phases

```csharp
enum AutomationPhase
{
    PaperTrading,      // Validation, no real money
    ManualExecute,     // Alerts → manual trades
    OneClick,          // Pre-configured orders
    AutoWithApproval,  // Auto-queue, approve/reject
    FullAuto           // Autonomous execution
}
```

---

## Coding Guidelines

### C# Conventions (from urus-blazor)

```csharp
// Use explicit types with target-typed new()
List<Alert> alerts = new();

// Nullable reference types
string? description = null;

// Null checks
if (entity is null) throw new ArgumentNullException(nameof(entity));

// Guard clauses
ArgumentNullException.ThrowIfNull(parameter);
ArgumentException.ThrowIfNullOrWhiteSpace(name);

// Money is always decimal
decimal price = 10.50m;
```

### Project Dependencies

```
Web → Infrastructure → Core
Tests → Core, Infrastructure
```

Core has no external dependencies (pure domain).

---

## Python Integration

Python is used for data acquisition only:

```
tools/py/
├── screener/    # TradingView integration
├── data/        # yfinance market data
└── analysis/    # Performance analysis
```

### C# ↔ Python Communication

Python scripts output JSON to stdout. C# parses with `System.Text.Json`.

```csharp
// PythonBridge example
ProcessStartInfo psi = new()
{
    FileName = "python",
    Arguments = "tools/py/screener/fetch_alerts.py --json",
    RedirectStandardOutput = true
};
```

---

## Key Files

| File | Purpose |
|------|---------|
| `PROJECT_VISION.md` | Long-term roadmap, architecture decisions |
| `IMPLEMENTATION_PLAN.md` | Phased plan with checklists |
| `momentum-screener.sln` | Solution file |
| `data/momentum.db` | SQLite database |
| `.editorconfig` | Code style rules |
| `Directory.Build.props` | Shared build settings |

---

## Trading Context

### Philosophy
- **LONG TRADES ONLY** - bullish momentum
- Small-cap US stocks (price < $20, no OTC)

### Risk Limits by Phase

| Phase | Max Risk/Trade | Max Daily Loss |
|-------|---------------|----------------|
| Paper | N/A | N/A |
| Manual | 5% | 10% |
| OneClick | 2% | 6% |
| Auto+Approval | 2% | 5% |
| FullAuto | 1-2% | 4% |

### Capital
- Starting: ~$1,100 USD (Feb 2026)
- Target: $5,000 for sustainable operation

---

## When Making Changes

1. **Check IMPLEMENTATION_PLAN.md** for current phase
2. **Follow phase checklist** - don't skip ahead
3. **Update lessons learned** after completing tasks
4. **Run tests** before committing: `dotnet test`
5. **Follow coding guidelines** - EditorConfig enforced

### Before Writing Code

1. Check if entity/interface exists in Core
2. Follow existing patterns in the codebase
3. Use MudBlazor components for UI
4. Prefer established frameworks for new features

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Build fails | `dotnet restore`, check SDK version |
| EF migration fails | Ensure startup project is Web |
| Python bridge fails | Check `.venv` activation, Python path |
| Blazor not updating | Use `dotnet watch`, not `dotnet run` |
| SQLite locked | Close other connections |

---

## Session Start Checklist

1. Read `IMPLEMENTATION_PLAN.md` Session Continuity section
2. Check current phase status
3. Review any blockers noted
4. Continue from last session's "Next Steps"
5. Update status after work is done
