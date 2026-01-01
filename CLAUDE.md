# Claude Code Project Context

## Project Summary

Momentum trading alert system for US small-cap stocks. Currently in Phase 1 (Alert System), evolving toward fully automated trading.

**Trading philosophy:** LONG TRADES ONLY - bullish momentum plays

## Key Files

| File | Purpose |
|------|---------|
| `volume_momentum_tracker.py` | Core alert system - scans, detects patterns, sends Telegram alerts |
| `paper_trading_system.py` | Simulates trades based on alerts using EMA entry/exit |
| `market_sentiment_scorer.py` | Scores market conditions (0-100) for position sizing |
| `end_of_day_analyzer.py` | Post-market analysis of alert success rates |
| `tradingview_screener_bot.py` | TradingView data collection |

## Architecture

```
TradingView API (via cookies)
    → volume_momentum_tracker.py (detection)
    → Telegram alerts
    → paper_trading_system.py (validation)
    → end_of_day_analyzer.py (performance tracking)
```

## Alert Types

1. **flat_to_spike** - Consolidation → breakout (highest success rate)
2. **price_spike** - Intraday price surge
3. **volume_climber** - Rising volume rankings
4. **volume_newcomer** - New high-volume entry
5. **premarket alerts** - Pre-market movements

## Current Development Phase

**Phase 1: Alert System** (current)
- Paper trading validation in progress
- Target: 1 month paper trading before live

**Next:** Phase 2 (Manual Execute) - Feb 2026 with $1,100 capital

See `PROJECT_VISION.md` for full roadmap.

## Key Technical Patterns

### Flat-to-Spike Detection
```python
# Thresholds
flat_volatility_threshold = 3.0%    # Max volatility for "flat"
min_flat_duration = 8 minutes
flat_period_window = 30 minutes
```

### Momentum Scoring
```python
score = (price_change * 0.4) + (rel_volume * 0.3) + (change_from_open * 0.3)
# Priority: >50 high, 20-50 medium, <20 low
```

### Market Sentiment (strongest predictor)
- IWM vs SPY outperformance: 0.988 correlation with alert success
- Score 80-100: Excellent, 65-79: Good, 45-64: Fair, 0-44: Poor

## Important Constraints

1. **Risk management approach:** User has no prior knowledge - use established frameworks (Kelly criterion, fixed fractional)
2. **Capital:** Starting with ~$1,100 USD in Feb 2026
3. **PDT rule:** Under $25k means max 3 day trades per 5 days
4. **Markets:** US small-caps primary, SGX secondary consideration (timezone alignment)
5. **Excluded:** Cryptocurrency

## Common Commands

```bash
# Single scan
python volume_momentum_tracker.py --single

# Continuous with Telegram
python volume_momentum_tracker.py --continuous \
  --bot-token "TOKEN" --chat-id "CHAT_ID"

# With paper trading
python volume_momentum_tracker.py --continuous --paper-trading

# Paper trading report
python volume_momentum_tracker.py --paper-report

# End of day analysis
python end_of_day_analyzer.py --date 2025-11-06
```

## Data Locations

- `momentum_data/` - Alert JSON files
- `momentum_data/paper_trades/` - Paper trading history
- `volume_momentum_tracker.log` - System logs
- `telegram_alerts_sent.jsonl` - Sent alert log

## Current Blockers

See `SETUP_STATUS.md` for:
- rookiepy Python 3.14 compatibility issue
- Moomoo API integration pending

## When Making Changes

1. **Alert logic changes:** Test with `--single` first
2. **Threshold changes:** Document reasoning in commit
3. **New features:** Align with automation phase goals in `PROJECT_VISION.md`
4. **Risk management:** Prefer established frameworks over custom solutions

## Historical Context

Archived documentation in `docs/archive/` contains detailed implementation notes from development (bug fixes, feature enhancements, analysis reports).
