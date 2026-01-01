# Momentum Screener

A real-time momentum trading system for small-cap stocks, designed to evolve from manual alerts to fully automated trading.

## What It Does

Monitors US small-cap stocks for momentum signals and sends alerts via Telegram. The system detects:

- **Price spikes** - Significant intraday price movements
- **Flat-to-spike patterns** - Consolidation followed by breakout (highest success rate)
- **Volume surges** - Unusual volume indicating institutional activity
- **Premarket movers** - Early momentum before market open

## Current Status

**Phase 1: Alert System** - Active development

See [PROJECT_VISION.md](PROJECT_VISION.md) for the full roadmap from alerts to automated trading.

## Quick Start

```bash
# Activate environment
source venv/bin/activate

# Single scan
python volume_momentum_tracker.py --single

# Continuous monitoring with Telegram alerts
python volume_momentum_tracker.py --continuous \
  --bot-token "YOUR_BOT_TOKEN" \
  --chat-id "YOUR_CHAT_ID"

# With paper trading enabled
python volume_momentum_tracker.py --continuous --paper-trading \
  --bot-token "YOUR_BOT_TOKEN" \
  --chat-id "YOUR_CHAT_ID"

# View paper trading performance
python volume_momentum_tracker.py --paper-report
```

## Requirements

- Python 3.12+ (3.14 has rookiepy compatibility issues)
- TradingView account (for screener data via browser cookies)
- Telegram bot (for alerts)

```bash
pip install pandas tradingview-screener rookiepy python-telegram-bot yfinance
```

## Project Structure

```
/
├── volume_momentum_tracker.py    # Main tracking system
├── paper_trading_system.py       # Paper trading simulation
├── market_sentiment_scorer.py    # Market condition analysis
├── end_of_day_analyzer.py        # Post-market performance analysis
├── PROJECT_VISION.md             # Long-term roadmap
├── SETUP_STATUS.md               # Current setup progress
├── CLAUDE.md                     # Claude Code context
└── docs/
    ├── TECHNICAL_GUIDE.md        # How everything works
    ├── PAPER_TRADING.md          # Paper trading guide
    └── RISK_MANAGEMENT.md        # Risk framework
```

## Documentation

| Document | Purpose |
|----------|---------|
| [PROJECT_VISION.md](PROJECT_VISION.md) | Long-term roadmap, automation phases, capital strategy |
| [SETUP_STATUS.md](SETUP_STATUS.md) | Current setup progress and blockers |
| [docs/TECHNICAL_GUIDE.md](docs/TECHNICAL_GUIDE.md) | How the system works |
| [docs/PAPER_TRADING.md](docs/PAPER_TRADING.md) | Paper trading validation |
| [docs/RISK_MANAGEMENT.md](docs/RISK_MANAGEMENT.md) | Risk management framework |

## Key Features

### Alert Intelligence
- **Momentum scoring** - Composite score prioritizing high-probability setups
- **Sector-specific tuning** - Adjusted thresholds per sector characteristics
- **Smart cooldowns** - Performance-based alert frequency

### Market Awareness
- **Sentiment scoring** - Real-time market condition assessment
- **Position sizing recommendations** - Based on market favorability
- **Small-cap correlation** - Uses IWM vs SPY outperformance (0.988 correlation with success)

### Validation
- **Paper trading** - EMA-based entry/exit simulation
- **End-of-day analysis** - Track alert success rates
- **Performance metrics** - Win rate, drawdowns, pattern effectiveness

## Trading Philosophy

**LONG TRADES ONLY** - This system focuses exclusively on bullish momentum in small-cap stocks. No short selling, no hedging.

## Disclaimer

This project is for educational and research purposes. Not financial advice. Always conduct your own research and never risk more than you can afford to lose.

## License

MIT License
