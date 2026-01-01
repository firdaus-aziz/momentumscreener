# Momentum Screener - Project Vision
*Established: 2026-01-01*

## Executive Summary

Build a **fully automated momentum trading system** that generates sustainable supplemental income with minimal daily oversight. The system will evolve through structured phases from manual trading to complete automation, with disciplined risk management adapted to each phase.

---

## Ultimate Goal

**End State:** A set-and-forget automated trading system that:
- Executes trades autonomously based on validated momentum signals
- Covers all operational costs (target: $100+/month baseline)
- Generates supplemental income without active monitoring
- Adapts to changing market conditions

**Trading Philosophy:** LONG TRADES ONLY - capturing bullish momentum in small-cap stocks

---

## Financial Framework

### Starting Capital
- **Initial:** MYR 5,000 (~USD 1,100) - Available February 2026
- **Target "Serious Capital":** USD 5,000 (milestone for sustainable cost coverage)

### Capital Growth Strategy
| Phase | Capital | Expectation |
|-------|---------|-------------|
| Paper Trading | $0 real | Validation only - no P&L pressure |
| Manual Execute | $1,100 | "Tuition phase" - learning with real stakes, may not cover costs |
| One-Click Execute | $1,500-2,500 | Add funds if showing promise |
| Auto with Approval | $3,000-5,000 | Begin expecting cost coverage |
| Full Auto | $5,000+ | Sustainable supplemental income |

### Funding Approach
1. Start with available $1,100
2. Add $200-500/month if system shows profitability
3. Reinvest profits to compound growth
4. Do NOT pressure early phases to cover costs (leads to over-trading)

### Cost Baseline
- Estimated monthly costs: ~$100 USD
  - Server/VPS hosting
  - Data feeds (if needed)
  - Platform fees
  - Tax provisions
- Cost coverage becomes realistic target at $5,000 capital milestone

---

## Automation Phases

### Phase Progression Rules
- **Minimum duration:** 1 month per phase
- **Maximum duration:** 3 months per phase
- **Progression trigger:** Confidence in system performance within duration window

### Phase 1: Alert System (Current)
**Status:** Active development

**Capabilities:**
- Real-time momentum detection via TradingView screener
- Telegram alerts for price spikes, volume surges, flat-to-spike patterns
- Paper trading simulation for strategy validation

**Exit Criteria:**
- Consistent alert quality
- Paper trading shows positive expectancy
- 1 month minimum paper trading completed

---

### Phase 2: Manual Execute
**Timeline:** Feb 2026 onwards (post paper trading validation)

**Capabilities:**
- Receive alert on Telegram
- Manually review and execute via Moomoo app
- Track all trades for performance analysis

**Risk Management:**
- Fixed position size (suggested: $300-500 per trade)
- Maximum 2-3 concurrent positions
- Manual stop-loss placement
- Respect PDT rule (max 3 day trades per 5 days with <$25k)

**Exit Criteria:**
- Positive P&L over 1-3 month period
- Confidence in alert quality and execution timing
- Clear patterns in winning vs losing trades

---

### Phase 3: One-Click Execute
**Capabilities:**
- Alert includes pre-configured order parameters
- Single tap/click to execute in Moomoo
- Automated position sizing calculation
- Pre-set stop-loss and take-profit levels

**Technical Requirements:**
- Moomoo OpenAPI integration
- Order preview before execution
- Mobile-friendly interface

**Risk Management:**
- Automated position sizing (1-2% risk per trade)
- Hard stop-losses attached to every order
- Daily loss limit (suggested: 5% of portfolio)

**Exit Criteria:**
- Execution friction minimized
- Consistent profitability maintained
- Trust in automated order parameters

---

### Phase 4: Auto-Execute with Approval
**Capabilities:**
- System prepares and queues orders automatically
- Push notification for approval
- Approve/reject within time window (e.g., 2 minutes)
- Auto-cancel if no response

**Technical Requirements:**
- Full Moomoo OpenAPI integration
- Approval workflow via Telegram or app
- Order queue management
- Timeout handling

**Risk Management:**
- All Phase 3 safeguards plus:
- Maximum orders per day limit
- Sector/ticker concentration limits
- Drawdown circuit breaker (pause if daily loss exceeds threshold)

**Exit Criteria:**
- High approval rate (>80% of queued trades approved)
- System recommendations consistently aligned with manual judgment
- Profitability maintained or improved

---

### Phase 5: Full Auto
**Capabilities:**
- Autonomous trade execution without approval
- Self-managing position sizing and risk
- Automated exit management (trailing stops, EMA-based exits)
- Daily/weekly summary reports only

**Technical Requirements:**
- Robust error handling and recovery
- Market condition detection (pause in extreme volatility)
- Self-monitoring and alerting for anomalies
- Comprehensive logging for audit

**Risk Management:**
- Maximum daily loss limit (auto-pause trading)
- Maximum weekly drawdown limit
- Position size caps
- Sector exposure limits
- Correlation-aware position management
- "Kill switch" for manual override

**Operational Model:**
- Daily: Automated summary report
- Weekly: Performance review
- Monthly: Strategy tuning if needed
- Quarterly: Comprehensive review and optimization

---

## Market Strategy

### Primary Market: US Small-Caps
- Price filter: <$20 per share
- Exchange filter: Excludes OTC
- Focus: Momentum plays with high relative volume

### Secondary Market: SGX (Singapore Exchange)
**Rationale:**
- Timezone alignment (market hours during your day)
- No currency conversion friction with Moomoo SG
- Direct local market access
- Lower barrier for smaller capital

**Approach:**
- Develop parallel momentum detection for SGX
- Run alongside US system
- Provides diversification across market hours

### Future Expansion
- **Philosophy:** Purely opportunistic - "wherever the edge is"
- **Excluded:** Cryptocurrency
- **Potential:** Other APAC markets, options strategies, forex (to be evaluated)

---

## Risk Management Framework

### Guiding Principles
1. Risk management must evolve with each automation phase
2. Prefer established, proven frameworks over custom solutions
3. Never risk more than can be recovered from
4. Preserve capital in early phases - profits come later

### Suggested Frameworks to Evaluate
- **Position Sizing:** Fixed fractional (1-2% risk per trade)
- **Stop Losses:** ATR-based or fixed percentage
- **Portfolio Heat:** Maximum 6-10% total portfolio at risk
- **Kelly Criterion:** For optimal position sizing (use half-Kelly for safety)

### Phase-Specific Risk Limits

| Phase | Max Risk/Trade | Max Daily Loss | Max Positions |
|-------|---------------|----------------|---------------|
| Manual | 5% (learning) | 10% | 2-3 |
| One-Click | 2% | 6% | 3-4 |
| Auto+Approval | 2% | 5% | 4-5 |
| Full Auto | 1-2% | 4% | 5-6 |

*Note: These are starting points - adjust based on actual performance data*

---

## Technical Architecture Vision

### Architecture Philosophy

**Dual-Language Approach** - Leverage the strengths of each platform:

| Aspect | C# .NET (Primary) | Python (Secondary) |
|--------|-------------------|-------------------|
| **Role** | Core application, business logic, UI | Data tooling, market APIs |
| **Strengths Used** | Type safety, performance, Blazor UI, async patterns | Data libraries (pandas), market APIs (yfinance, tradingview-screener) |
| **Examples** | Trading engine, risk manager, dashboard | Market data fetching, analysis scripts, screener integration |

**Inspiration:** Architecture pattern from `urus-blazor` project - Clean Architecture with auxiliary Python tooling.

### Technology Stack

| Layer | Technology | Rationale |
|-------|------------|-----------|
| **UI** | Blazor Server + MudBlazor | Real-time updates, rich components, C# full-stack |
| **Backend** | ASP.NET Core | Robust, performant, excellent async support |
| **Database** | SQLite | Simple, file-based, sufficient for single-user trading system |
| **ORM** | EF Core | Standard .NET data access |
| **Testing** | xUnit + FluentAssertions | .NET standard, expressive assertions |
| **Python Tooling** | pip-tools | Reproducible Python dependencies |

### Current State
```
TradingView Screener → Python Tracker → Telegram Alerts → Manual Review
                              ↓
                    (All logic in Python)
```

### Target State
```
┌─────────────────────────────────────────────────────────────────────────┐
│                        C# .NET (Primary)                                 │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│   ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐    │
│   │  Blazor Web UI  │    │  Trading Engine │    │   Risk Manager  │    │
│   │   (Dashboard)   │◄──►│  (Core Logic)   │◄──►│ (Position Size) │    │
│   └─────────────────┘    └─────────────────┘    └─────────────────┘    │
│            │                      │                      │              │
│            ▼                      ▼                      ▼              │
│   ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐    │
│   │    SQLite DB    │    │  Telegram Bot   │    │   Moomoo API    │    │
│   │  (Persistence)  │    │    (Alerts)     │    │  (Execution)    │    │
│   └─────────────────┘    └─────────────────┘    └─────────────────┘    │
│                                  ▲                                      │
└──────────────────────────────────┼──────────────────────────────────────┘
                                   │
                    ┌──────────────┴──────────────┐
                    │     Python (Secondary)       │
                    ├──────────────────────────────┤
                    │  ┌────────────────────────┐  │
                    │  │  TradingView Screener  │  │
                    │  │  (Data Acquisition)    │  │
                    │  └────────────────────────┘  │
                    │  ┌────────────────────────┐  │
                    │  │  Market Data (yfinance)│  │
                    │  │  (Price/Volume Data)   │  │
                    │  └────────────────────────┘  │
                    │  ┌────────────────────────┐  │
                    │  │   Analysis Scripts     │  │
                    │  │  (pandas, numpy)       │  │
                    │  └────────────────────────┘  │
                    └──────────────────────────────┘
```

### Target Project Structure

```
momentum-screener/
├── src/
│   ├── MomentumScreener.Core/           # Domain entities, interfaces
│   │   ├── Entities/
│   │   │   ├── Alert.cs
│   │   │   ├── Position.cs
│   │   │   ├── Trade.cs
│   │   │   └── RiskProfile.cs
│   │   ├── Enums/
│   │   │   ├── AlertType.cs
│   │   │   └── TradeStatus.cs
│   │   └── Interfaces/
│   │       ├── IAlertService.cs
│   │       ├── ITradingEngine.cs
│   │       └── IRiskManager.cs
│   │
│   ├── MomentumScreener.Infrastructure/  # External integrations
│   │   ├── Data/
│   │   │   ├── AppDbContext.cs
│   │   │   └── Migrations/
│   │   ├── Services/
│   │   │   ├── TelegramService.cs
│   │   │   ├── MoomooService.cs
│   │   │   └── MarketDataService.cs
│   │   └── Python/
│   │       └── PythonBridge.cs           # Interop with Python tools
│   │
│   └── MomentumScreener.Web/             # Blazor Server UI
│       ├── Components/
│       │   ├── Layout/
│       │   └── Pages/
│       │       ├── Dashboard.razor
│       │       ├── Alerts.razor
│       │       ├── Positions.razor
│       │       └── Settings.razor
│       └── Program.cs
│
├── tests/
│   ├── MomentumScreener.Core.Tests/
│   └── MomentumScreener.Integration.Tests/
│
├── tools/
│   └── py/
│       ├── screener/                     # TradingView integration
│       │   └── volume_momentum_tracker.py
│       ├── data/                         # Market data fetching
│       │   └── market_data.py
│       └── analysis/                     # Analysis scripts
│           └── performance_analyzer.py
│
├── docs/
│   ├── TECHNICAL_GUIDE.md
│   ├── PAPER_TRADING.md
│   └── RISK_MANAGEMENT.md
│
├── data/                                 # SQLite database location
│   └── momentum.db
│
├── .editorconfig                         # Code style enforcement
├── Directory.Build.props                 # Shared build settings
├── momentum-screener.sln
├── requirements.in                       # Python dependencies (source)
├── requirements.txt                      # Python dependencies (locked)
├── PROJECT_VISION.md
├── CLAUDE.md
└── README.md
```

### C# Components to Build

| Component | Project | Purpose |
|-----------|---------|---------|
| **Alert Entity** | Core | Domain model for momentum alerts |
| **Position Entity** | Core | Track open/closed positions |
| **Trade Entity** | Core | Individual trade records |
| **RiskProfile** | Core | Risk parameters per phase |
| **ITradingEngine** | Core | Interface for trading logic |
| **IRiskManager** | Core | Interface for risk calculations |
| **AppDbContext** | Infrastructure | EF Core SQLite context |
| **TelegramService** | Infrastructure | Send alerts via Telegram |
| **MoomooService** | Infrastructure | Moomoo OpenAPI integration |
| **PythonBridge** | Infrastructure | Execute Python scripts, parse output |
| **Dashboard** | Web | Real-time overview |
| **Alerts Page** | Web | Alert history and management |
| **Positions Page** | Web | Current and historical positions |
| **Settings Page** | Web | Configuration management |

### Python Components to Retain/Refactor

| Component | Location | Purpose |
|-----------|----------|---------|
| **Screener** | tools/py/screener/ | TradingView data acquisition |
| **Market Data** | tools/py/data/ | yfinance integration |
| **Analyzers** | tools/py/analysis/ | Performance analysis scripts |

### C# ↔ Python Integration

Communication via JSON over stdout:

```csharp
// PythonBridge.cs
public async Task<List<Alert>> FetchAlertsAsync()
{
    ProcessStartInfo psi = new()
    {
        FileName = "python",
        Arguments = "tools/py/screener/fetch_alerts.py --json",
        RedirectStandardOutput = true
    };

    using Process process = Process.Start(psi);
    string json = await process.StandardOutput.ReadToEndAsync();
    return JsonSerializer.Deserialize<List<Alert>>(json);
}
```

Python scripts output JSON, C# parses and processes.

---

## Success Metrics

### Phase 1-2 (Learning)
- Paper trading positive expectancy
- Understanding of winning patterns
- Discipline in following system rules

### Phase 3-4 (Validation)
- Positive P&L (any amount)
- Win rate >40% with >2:1 reward/risk
- Maximum drawdown <20%

### Phase 5 (Sustainable)
- Monthly returns covering costs ($100+)
- Consistent positive months (>60% of months profitable)
- Maximum drawdown <15%
- Minimal required oversight (<30 min/week)

---

## Timeline Overview

```
2026
├── Jan: Paper trading validation, vision documentation
├── Feb: Live trading begins (Manual Execute phase) with $1,100
├── Mar-Apr: Manual Execute continues, performance evaluation
├── May-Jun: One-Click Execute (if Phase 2 successful)
├── Jul-Sep: Auto with Approval (if Phase 3 successful)
├── Q4: Full Auto evaluation (if Phase 4 successful)

2027
├── Q1: Full Auto operation (target)
├── Ongoing: Optimization, market expansion
```

*Timeline is indicative - progression based on confidence, not calendar*

---

## Open Questions & Future Decisions

1. **SGX Implementation Priority** - Build in parallel with US, or sequential?
2. **Server Infrastructure** - Local machine, cloud VPS, or dedicated server?
3. **Backup & Recovery** - What happens if primary system fails during market hours?
4. **Tax Optimization** - Structure for SG-based trading of US/SG markets?
5. **Scaling Strategy** - At what capital level to increase position sizes?
6. **Migration Strategy** - Incremental migration from Python-only to C#/.NET primary, or big-bang rewrite?

---

## Revision History

| Date | Changes |
|------|---------|
| 2026-01-01 | Initial vision document created |
| 2026-01-01 | Added technical architecture: C# .NET primary + Python secondary, Blazor UI, SQLite database |

---

*This document should be reviewed and updated quarterly or when significant strategic decisions are made.*
