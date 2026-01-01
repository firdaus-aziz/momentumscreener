# Implementation Plan

*Created: 2026-01-01*
*Last Updated: 2026-01-01*

## Overview

This document outlines the systematic implementation plan for migrating the Momentum Screener from Python-only to a C# .NET primary architecture with Python as secondary tooling.

**Approach:** Big-Bang Rewrite with phased execution
**Reference Architecture:** `urus-blazor` project pattern

### Target Architecture Summary

- **Primary:** C# .NET 8+ (Blazor Server, EF Core, SQLite)
- **Secondary:** Python (TradingView screener, yfinance, analysis scripts)
- **UI:** Blazor Server + MudBlazor
- **Database:** SQLite
- **Integration:** C# ↔ Python via JSON over stdout

### Success Criteria

- [ ] C# .NET solution builds and runs
- [ ] Blazor dashboard displays alerts from Python screener
- [ ] SQLite persists alerts, positions, trades
- [ ] Telegram integration works from C#
- [ ] Paper trading logic migrated to C#
- [ ] Python retained only for data acquisition
- [ ] All tests pass
- [ ] Documentation complete and accurate

---

## Session Continuity

**For future sessions:** Check this section first.

### Current Status

| Phase | Status | Notes |
|-------|--------|-------|
| Phase 1: Infrastructure | Not Started | |
| Phase 2: Core Domain | Not Started | |
| Phase 3: Data Layer | Not Started | |
| Phase 4: Python Bridge | Not Started | |
| Phase 5: Services | Not Started | |
| Phase 6: Blazor UI | Not Started | |
| Phase 7: Integration | Not Started | |
| Phase 8: Testing | Not Started | |
| Phase 9: Migration | Not Started | |

### Last Session Summary

*Update this after each session*

```
Date:
Completed:
In Progress:
Blockers:
Next Steps:
```

---

## Phase 1: Infrastructure Setup

**Objective:** Establish the C# .NET project structure, build chain, and development tooling.

### Checklist

#### Solution Structure
- [ ] Create `momentum-screener.sln` solution file
- [ ] Create `src/MomentumScreener.Core/` project (Class Library)
- [ ] Create `src/MomentumScreener.Infrastructure/` project (Class Library)
- [ ] Create `src/MomentumScreener.Web/` project (Blazor Server)
- [ ] Create `tests/MomentumScreener.Core.Tests/` project (xUnit)
- [ ] Create `tests/MomentumScreener.Integration.Tests/` project (xUnit)
- [ ] Set up project references (Web → Infrastructure → Core)

#### Build Configuration
- [ ] Create `.editorconfig` (copy from urus-blazor, adapt)
- [ ] Create `Directory.Build.props` (shared build settings)
- [ ] Configure nullable reference types
- [ ] Configure implicit usings
- [ ] Set up analyzer rules

#### Development Tooling
- [ ] Create `.gitignore` updates for .NET
- [ ] Set up `tools/py/` directory structure
- [ ] Create `requirements.in` for Python deps
- [ ] Create `.python-version` file
- [ ] Verify solution builds: `dotnet build`
- [ ] Verify solution runs: `dotnet run --project src/MomentumScreener.Web`

#### NuGet Packages
- [ ] Add MudBlazor to Web project
- [ ] Add EF Core + SQLite to Infrastructure project
- [ ] Add FluentAssertions to test projects
- [ ] Add Moq to test projects

### Exit Criteria

- [ ] `dotnet build` succeeds with no errors
- [ ] `dotnet run --project src/MomentumScreener.Web` starts Blazor app
- [ ] Browser shows default Blazor page
- [ ] All EditorConfig rules enforced
- [ ] Project structure matches vision document

### Lessons Learned

*Fill in during/after implementation*

```
Date:
What worked well:
What was challenging:
Decisions made:
Things to remember:
```

---

## Phase 2: Core Domain

**Objective:** Define the domain entities, enums, and interfaces that represent the trading system.

### Checklist

#### Entities
- [ ] Create `Entities/Alert.cs`
  - Id, Ticker, AlertType, Price, Volume, RelativeVolume, Sector
  - ChangePercent, ChangeFromOpen, Timestamp
  - FlatAnalysis (nested or separate)
- [ ] Create `Entities/Position.cs`
  - Id, Ticker, EntryPrice, CurrentPrice, Shares, PositionSize
  - EntryTime, Status, AlertId (FK)
- [ ] Create `Entities/Trade.cs`
  - Id, Ticker, EntryPrice, ExitPrice, Shares
  - EntryTime, ExitTime, ProfitLoss, ProfitPercent
  - ExitReason, AlertType, PositionId (FK)
- [ ] Create `Entities/RiskProfile.cs`
  - Id, Phase, MaxRiskPerTrade, MaxDailyLoss, MaxPositions
  - IsActive
- [ ] Create `Entities/MarketCondition.cs`
  - Score, Category, SmallCapOutperformance
  - NasdaqPerformance, BiotechStrength, Timestamp

#### Enums
- [ ] Create `Enums/AlertType.cs`
  - FlatToSpike, PriceSpike, VolumeClimber, VolumeNewcomer
  - PremarketPrice, PremarketVolume
- [ ] Create `Enums/PositionStatus.cs`
  - Open, Closed, Pending
- [ ] Create `Enums/TradeExitReason.cs`
  - EmaExit, StopLoss, TakeProfit, Manual, EndOfDay
- [ ] Create `Enums/AutomationPhase.cs`
  - PaperTrading, ManualExecute, OneClick, AutoWithApproval, FullAuto
- [ ] Create `Enums/MarketConditionCategory.cs`
  - Excellent, Good, Fair, Poor

#### Interfaces
- [ ] Create `Interfaces/IAlertRepository.cs`
- [ ] Create `Interfaces/IPositionRepository.cs`
- [ ] Create `Interfaces/ITradeRepository.cs`
- [ ] Create `Interfaces/IAlertService.cs`
- [ ] Create `Interfaces/ITradingEngine.cs`
- [ ] Create `Interfaces/IRiskManager.cs`
- [ ] Create `Interfaces/IMarketDataService.cs`
- [ ] Create `Interfaces/IPythonBridge.cs`

#### Value Objects (optional)
- [ ] Create `ValueObjects/Money.cs` (if needed)
- [ ] Create `ValueObjects/Percentage.cs` (if needed)

### Exit Criteria

- [ ] All entities compile without errors
- [ ] All enums defined
- [ ] All interfaces defined
- [ ] Core project builds independently
- [ ] No external dependencies in Core (pure domain)

### Lessons Learned

```
Date:
What worked well:
What was challenging:
Decisions made:
Things to remember:
```

---

## Phase 3: Data Layer

**Objective:** Implement SQLite persistence with EF Core.

### Checklist

#### DbContext
- [ ] Create `Data/AppDbContext.cs`
- [ ] Configure entity mappings
- [ ] Configure relationships (Alert → Position → Trade)
- [ ] Set up SQLite connection string

#### Repositories
- [ ] Create `Repositories/AlertRepository.cs`
- [ ] Create `Repositories/PositionRepository.cs`
- [ ] Create `Repositories/TradeRepository.cs`
- [ ] Create `Repositories/RiskProfileRepository.cs`

#### Migrations
- [ ] Create initial migration: `dotnet ef migrations add InitialCreate`
- [ ] Apply migration: `dotnet ef database update`
- [ ] Verify `data/momentum.db` created

#### Seed Data
- [ ] Create seed data for RiskProfile (one per phase)
- [ ] Create seed script or migration

#### Configuration
- [ ] Add connection string to appsettings.json
- [ ] Register DbContext in DI container
- [ ] Register repositories in DI container

### Exit Criteria

- [ ] Database file created at `data/momentum.db`
- [ ] All tables created (Alerts, Positions, Trades, RiskProfiles)
- [ ] Can insert and query test data
- [ ] Repositories work via DI

### Lessons Learned

```
Date:
What worked well:
What was challenging:
Decisions made:
Things to remember:
```

---

## Phase 4: Python Bridge

**Objective:** Create the C# ↔ Python integration layer.

### Checklist

#### Python Scripts Refactoring
- [ ] Create `tools/py/` directory structure
  - `tools/py/screener/`
  - `tools/py/data/`
  - `tools/py/analysis/`
- [ ] Refactor `volume_momentum_tracker.py` for JSON output mode
- [ ] Create `tools/py/screener/fetch_alerts.py` (simplified alert fetcher)
- [ ] Create `tools/py/data/market_data.py` (market data fetcher)
- [ ] Test JSON output: `python tools/py/screener/fetch_alerts.py --json`

#### C# Bridge
- [ ] Create `Python/PythonBridge.cs`
- [ ] Create `Python/PythonSettings.cs` (config for Python path, venv)
- [ ] Implement `ExecuteScriptAsync(script, args)` method
- [ ] Implement `FetchAlertsAsync()` method
- [ ] Implement `FetchMarketDataAsync(ticker)` method
- [ ] Handle errors and timeouts

#### Configuration
- [ ] Add Python settings to appsettings.json
- [ ] Register IPythonBridge in DI container

### Exit Criteria

- [ ] Python scripts output valid JSON
- [ ] C# can execute Python scripts
- [ ] C# can parse JSON output into entities
- [ ] Error handling works (script failure, timeout)

### Lessons Learned

```
Date:
What worked well:
What was challenging:
Decisions made:
Things to remember:
```

---

## Phase 5: Services

**Objective:** Implement business logic services.

### Checklist

#### Alert Service
- [ ] Create `Services/AlertService.cs`
- [ ] Implement `FetchAndStoreAlertsAsync()`
- [ ] Implement `GetRecentAlertsAsync()`
- [ ] Implement `GetAlertsByTypeAsync()`
- [ ] Implement alert filtering/scoring logic

#### Trading Engine
- [ ] Create `Services/TradingEngine.cs`
- [ ] Implement `ProcessAlertAsync(alert)`
- [ ] Implement `ShouldEnterPosition(alert, marketCondition)`
- [ ] Implement `ShouldExitPosition(position)`
- [ ] Implement EMA-based entry/exit logic

#### Risk Manager
- [ ] Create `Services/RiskManager.cs`
- [ ] Implement `CalculatePositionSize(alert, riskProfile)`
- [ ] Implement `CanOpenPosition(riskProfile)`
- [ ] Implement `GetCurrentExposure()`
- [ ] Implement daily loss limit check

#### Market Data Service
- [ ] Create `Services/MarketDataService.cs`
- [ ] Implement `GetCurrentPriceAsync(ticker)`
- [ ] Implement `GetMarketConditionAsync()`
- [ ] Cache market data appropriately

#### Telegram Service
- [ ] Create `Services/TelegramService.cs`
- [ ] Implement `SendAlertAsync(alert)`
- [ ] Implement `SendPositionUpdateAsync(position)`
- [ ] Implement `SendDailySummaryAsync()`
- [ ] Handle Telegram API errors

#### Background Services
- [ ] Create `Services/AlertScannerBackgroundService.cs`
- [ ] Implement scheduled scanning (every 2 minutes)
- [ ] Implement market hours check

### Exit Criteria

- [ ] All services compile and register in DI
- [ ] AlertService can fetch and store alerts
- [ ] TradingEngine can process alerts
- [ ] RiskManager calculates position sizes
- [ ] TelegramService sends messages

### Lessons Learned

```
Date:
What worked well:
What was challenging:
Decisions made:
Things to remember:
```

---

## Phase 6: Blazor UI

**Objective:** Build the Blazor Server dashboard.

### Checklist

#### Layout
- [ ] Set up MudBlazor in Program.cs
- [ ] Create `Layout/MainLayout.razor`
- [ ] Create `Layout/NavMenu.razor`
- [ ] Configure MudBlazor theme

#### Dashboard Page
- [ ] Create `Pages/Dashboard.razor`
- [ ] Display current market condition score
- [ ] Display active positions summary
- [ ] Display today's P&L
- [ ] Display recent alerts feed
- [ ] Auto-refresh every 30 seconds

#### Alerts Page
- [ ] Create `Pages/Alerts.razor`
- [ ] Display alerts table with filtering
- [ ] Show alert details in dialog
- [ ] Link to TradingView chart

#### Positions Page
- [ ] Create `Pages/Positions.razor`
- [ ] Display open positions
- [ ] Display closed positions (history)
- [ ] Show P&L per position

#### Trades Page
- [ ] Create `Pages/Trades.razor`
- [ ] Display trade history
- [ ] Performance metrics (win rate, avg return)

#### Settings Page
- [ ] Create `Pages/Settings.razor`
- [ ] Display current risk profile
- [ ] Allow phase selection
- [ ] Telegram configuration
- [ ] Python path configuration

### Exit Criteria

- [ ] All pages render without errors
- [ ] Navigation works
- [ ] Data displays correctly
- [ ] Responsive layout
- [ ] MudBlazor components styled properly

### Lessons Learned

```
Date:
What worked well:
What was challenging:
Decisions made:
Things to remember:
```

---

## Phase 7: Integration

**Objective:** Wire everything together for end-to-end functionality.

### Checklist

#### Service Registration
- [ ] Register all services in Program.cs
- [ ] Configure dependency injection
- [ ] Set up configuration binding

#### Background Processing
- [ ] Start AlertScannerBackgroundService
- [ ] Verify scanning runs on schedule
- [ ] Verify alerts stored in database
- [ ] Verify Telegram notifications sent

#### UI Data Binding
- [ ] Dashboard pulls live data
- [ ] Alerts page shows database alerts
- [ ] Positions update in real-time

#### Error Handling
- [ ] Global error handling in Blazor
- [ ] Service error logging
- [ ] Python bridge error handling

#### Configuration
- [ ] Finalize appsettings.json
- [ ] Set up appsettings.Development.json
- [ ] User secrets for sensitive data (Telegram token)

### Exit Criteria

- [ ] End-to-end flow works: Python → C# → SQLite → Blazor
- [ ] Telegram alerts sent
- [ ] Dashboard shows real data
- [ ] System runs stable for 1 hour

### Lessons Learned

```
Date:
What worked well:
What was challenging:
Decisions made:
Things to remember:
```

---

## Phase 8: Testing

**Objective:** Implement comprehensive tests.

### Checklist

#### Unit Tests (Core.Tests)
- [ ] Entity creation tests
- [ ] Risk calculation tests
- [ ] Position sizing tests
- [ ] Alert filtering tests

#### Integration Tests (Integration.Tests)
- [ ] Repository tests with SQLite
- [ ] AlertService integration test
- [ ] TradingEngine integration test
- [ ] PythonBridge integration test

#### Test Infrastructure
- [ ] Set up test fixtures
- [ ] Set up test database
- [ ] Create test helpers

#### Code Coverage
- [ ] Run coverage report
- [ ] Target >70% coverage on Core
- [ ] Target >50% coverage on Infrastructure

### Exit Criteria

- [ ] All tests pass: `dotnet test`
- [ ] Coverage targets met
- [ ] No flaky tests

### Lessons Learned

```
Date:
What worked well:
What was challenging:
Decisions made:
Things to remember:
```

---

## Phase 9: Migration & Cleanup

**Objective:** Archive old Python files, finalize structure, update documentation.

### Checklist

#### Archive Old Python Files
- [ ] Move root `.py` files to `archive/python-legacy/`
- [ ] Keep only `tools/py/` Python files
- [ ] Update `.gitignore`

#### Documentation Updates
- [ ] Final pass on README.md
- [ ] Final pass on CLAUDE.md
- [ ] Final pass on TECHNICAL_GUIDE.md
- [ ] Update IMPLEMENTATION_PLAN.md status

#### Cleanup
- [ ] Remove unused dependencies
- [ ] Remove dead code
- [ ] Format all code
- [ ] Run linters

#### Final Verification
- [ ] Fresh clone and build
- [ ] Run all tests
- [ ] Start application
- [ ] Verify all features work

### Exit Criteria

- [ ] Clean project structure
- [ ] No legacy Python in root
- [ ] Documentation accurate
- [ ] Fresh clone builds and runs

### Lessons Learned

```
Date:
What worked well:
What was challenging:
Decisions made:
Things to remember:
```

---

## Appendix

### Quick Reference Commands

```bash
# Build
dotnet build

# Run web app
dotnet run --project src/MomentumScreener.Web

# Run tests
dotnet test

# Add migration
dotnet ef migrations add MigrationName --project src/MomentumScreener.Infrastructure --startup-project src/MomentumScreener.Web

# Update database
dotnet ef database update --project src/MomentumScreener.Infrastructure --startup-project src/MomentumScreener.Web

# Python venv
python3 -m venv .venv
source .venv/bin/activate
pip install pip-tools
pip-compile requirements.in
pip-sync requirements.txt
```

### Key File Locations

| Purpose | Location |
|---------|----------|
| Solution file | `momentum-screener.sln` |
| Web project | `src/MomentumScreener.Web/` |
| Database | `data/momentum.db` |
| Python tools | `tools/py/` |
| Python venv | `.venv/` |
| Configuration | `src/MomentumScreener.Web/appsettings.json` |

### Troubleshooting

| Issue | Solution |
|-------|----------|
| Build fails | Check .NET SDK version, run `dotnet restore` |
| EF migrations fail | Ensure startup project is Web |
| Python bridge fails | Check Python path, venv activation |
| SQLite locked | Close other connections, restart app |
| Blazor not hot-reloading | Use `dotnet watch` instead of `dotnet run` |

---

## Revision History

| Date | Changes |
|------|---------|
| 2026-01-01 | Initial plan created |

---

*This plan should be updated as implementation progresses. Mark checkboxes, fill in lessons learned, and update session continuity section after each work session.*
