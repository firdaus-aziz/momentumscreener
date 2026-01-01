# Momentum Screener

A real-time momentum trading system for small-cap stocks, built with C# .NET and Blazor.

## Architecture

**Primary:** C# .NET 8 (Blazor Server, EF Core, SQLite)
**Secondary:** Python (TradingView data acquisition, market analysis)

```
┌─────────────────────────────────────────────────────────┐
│                   C# .NET (Primary)                      │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐     │
│  │ Blazor UI   │  │  Trading    │  │    Risk     │     │
│  │ (Dashboard) │  │   Engine    │  │   Manager   │     │
│  └─────────────┘  └─────────────┘  └─────────────┘     │
│         │                │                │             │
│         └────────────────┼────────────────┘             │
│                          ▼                              │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐     │
│  │   SQLite    │  │  Telegram   │  │   Moomoo    │     │
│  │     DB      │  │    Bot      │  │    API      │     │
│  └─────────────┘  └─────────────┘  └─────────────┘     │
└─────────────────────────────────────────────────────────┘
                          ▲
                          │ JSON
┌─────────────────────────┴───────────────────────────────┐
│                 Python (Secondary)                       │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐     │
│  │ TradingView │  │  yfinance   │  │  Analysis   │     │
│  │  Screener   │  │   Data      │  │   Scripts   │     │
│  └─────────────┘  └─────────────┘  └─────────────┘     │
└─────────────────────────────────────────────────────────┘
```

## Features

- **Real-time alerts** for price spikes, volume surges, flat-to-spike patterns
- **Blazor dashboard** for monitoring alerts and positions
- **Paper trading** simulation with EMA-based entry/exit
- **Risk management** with phase-appropriate position sizing
- **Telegram integration** for mobile notifications
- **Market sentiment scoring** for position size recommendations

## Quick Start

### Prerequisites

- .NET 8 SDK
- Python 3.12+
- SQLite (bundled with .NET)

### Build and Run

```bash
# Clone repository
git clone https://github.com/firdaus-aziz/momentumscreener.git
cd momentumscreener

# Restore and build
dotnet restore
dotnet build

# Run the web application
dotnet run --project src/MomentumScreener.Web

# Open browser to https://localhost:5001
```

### Python Setup (for data acquisition)

```bash
# Create virtual environment
python3 -m venv .venv
source .venv/bin/activate

# Install dependencies
pip install pip-tools
pip-compile requirements.in
pip-sync requirements.txt
```

## Project Structure

```
momentum-screener/
├── src/
│   ├── MomentumScreener.Core/           # Domain entities, interfaces
│   ├── MomentumScreener.Infrastructure/ # EF Core, services, integrations
│   └── MomentumScreener.Web/            # Blazor Server UI
├── tests/
│   ├── MomentumScreener.Core.Tests/
│   └── MomentumScreener.Integration.Tests/
├── tools/
│   └── py/
│       ├── screener/                    # TradingView integration
│       ├── data/                        # Market data fetching
│       └── analysis/                    # Analysis scripts
├── data/                                # SQLite database
├── docs/                                # Documentation
├── momentum-screener.sln
├── PROJECT_VISION.md                    # Long-term roadmap
├── IMPLEMENTATION_PLAN.md               # Migration plan with phases
└── CLAUDE.md                            # Claude Code context
```

## Documentation

| Document | Purpose |
|----------|---------|
| [PROJECT_VISION.md](PROJECT_VISION.md) | Long-term roadmap, automation phases, architecture |
| [IMPLEMENTATION_PLAN.md](IMPLEMENTATION_PLAN.md) | Phased implementation with checklists |
| [docs/TECHNICAL_GUIDE.md](docs/TECHNICAL_GUIDE.md) | Architecture and implementation details |
| [docs/PAPER_TRADING.md](docs/PAPER_TRADING.md) | Paper trading validation guide |
| [docs/RISK_MANAGEMENT.md](docs/RISK_MANAGEMENT.md) | Risk framework and position sizing |

## Technology Stack

| Layer | Technology |
|-------|------------|
| UI | Blazor Server + MudBlazor |
| Backend | ASP.NET Core |
| Database | SQLite + EF Core |
| Testing | xUnit + FluentAssertions |
| Data Acquisition | Python (tradingview-screener, yfinance) |

## Trading Philosophy

**LONG TRADES ONLY** - This system focuses exclusively on bullish momentum in small-cap stocks.

See [PROJECT_VISION.md](PROJECT_VISION.md) for automation phases and capital strategy.

## Development

```bash
# Run with hot reload
dotnet watch --project src/MomentumScreener.Web

# Run tests
dotnet test

# Add EF migration
dotnet ef migrations add MigrationName \
  --project src/MomentumScreener.Infrastructure \
  --startup-project src/MomentumScreener.Web
```

## Current Status

**Phase:** Implementation in progress

See [IMPLEMENTATION_PLAN.md](IMPLEMENTATION_PLAN.md) for current phase and progress.

## Disclaimer

This project is for educational and research purposes. Not financial advice. Always conduct your own research and never risk more than you can afford to lose.

## License

MIT License
