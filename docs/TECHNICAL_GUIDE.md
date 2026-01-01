# Technical Guide

How the momentum screening system works.

## System Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                     Data Flow                                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   TradingView Screener API                                      │
│          │                                                       │
│          ▼                                                       │
│   ┌──────────────────────┐                                      │
│   │ volume_momentum_     │                                      │
│   │ tracker.py           │──────┐                               │
│   └──────────────────────┘      │                               │
│          │                      │                               │
│          ▼                      ▼                               │
│   ┌──────────────┐    ┌─────────────────┐                      │
│   │ Telegram     │    │ paper_trading_  │                      │
│   │ Alerts       │    │ system.py       │                      │
│   └──────────────┘    └─────────────────┘                      │
│                              │                                   │
│                              ▼                                   │
│                       ┌─────────────────┐                       │
│                       │ end_of_day_     │                       │
│                       │ analyzer.py     │                       │
│                       └─────────────────┘                       │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

## Core Components

### 1. Volume Momentum Tracker

**File:** `volume_momentum_tracker.py`

Main entry point. Scans TradingView every 2 minutes and detects momentum patterns.

**Key class:** `VolumeMomentumTracker`

```python
# Core methods
fetch_data()              # Get screener data via TradingView API
analyze_price_spikes()    # Detect price movements
_detect_flat_period()     # Identify consolidation patterns
calculate_momentum_score() # Score alert quality
should_send_alert()       # Filter based on cooldowns, scores
send_telegram_alert()     # Deliver to user
```

**Scan filters:**
- Price < $20
- Excludes OTC exchanges
- Top 200 by relative volume

### 2. Pattern Detection

#### Flat-to-Spike Detection

Highest success rate pattern. Detects consolidation followed by breakout.

```python
# Detection parameters
flat_volatility_threshold = 3.0%    # Max price range for "flat"
min_flat_duration = 8 minutes       # Minimum consolidation time
flat_period_window = 30 minutes     # Lookback window

# Logic
volatility = ((max_price - min_price) / avg_price) * 100
is_flat = volatility <= 3.0% AND duration >= 8 minutes
```

**Output in alerts:** `FLAT→SPIKE(15m,1.8%)` = 15 min flat, 1.8% volatility

#### Price Spike Detection

Standard intraday momentum without flat verification.

```python
# Thresholds
immediate_alert_threshold = 15%     # Bypass cooldowns
standard_threshold = 5%             # Regular alerts
```

#### Volume Patterns

- **Volume climbers:** Rising in volume rankings
- **Volume newcomers:** New entries in top 50

### 3. Momentum Scoring

Composite score for alert prioritization.

```python
score = (price_change * 0.4) + (rel_volume_normalized * 0.3) + (change_from_open * 0.3)

# Bonuses
if alert_type == "flat_to_spike": score += 50
if change_pct >= 75: score += 30
if alert_type in ["premarket"]: score -= 15  # Penalty

# Priority tiers
HIGH = score > 50      # Immediate, bypass cooldowns
MEDIUM = 20-50         # Standard processing
LOW = score < 20       # Extended cooldowns
```

### 4. Market Sentiment Scorer

**File:** `market_sentiment_scorer.py`

Assesses overall market conditions for position sizing recommendations.

**Scoring factors (100 points total):**

| Factor | Weight | Source |
|--------|--------|--------|
| Small-cap outperformance | 25 pts | IWM vs SPY (0.988 correlation!) |
| NASDAQ performance | 20 pts | QQQ daily return |
| Biotech strength | 20 pts | IBB return |
| Volatility conditions | 20 pts | SPY range |
| Market breadth | 15 pts | % sectors positive |

**Categories:**
- 80-100: Excellent - Full position sizes
- 65-79: Good - Normal operations
- 45-64: Fair - Reduced positions
- 0-44: Poor - Avoid or paper only

### 5. Alert Filtering

Multiple layers prevent spam:

**Cooldown system:**
```python
# Performance-based cooldowns
high_performers (>20% avg): 5 min cooldown
regular (5-20% avg): 10 min cooldown
poor (<5% avg): 20 min cooldown
```

**Sector concentration limits:**
```python
# Prevent single ticker dominance
utilities_sector: max 30% of alerts (PCG/PC issue)
```

**Sector-specific multipliers:**
```python
# Adjust thresholds per sector
finance/health_tech: 1.5x rel_volume (reduce noise)
technology_services: 0.9x thresholds (catch early)
electronic_tech: 0.8x (prioritize high momentum)
```

### 6. End of Day Analyzer

**File:** `end_of_day_analyzer.py`

Post-market analysis of alert performance.

```bash
python end_of_day_analyzer.py --date 2025-11-06
```

**Metrics calculated:**
- Success rate (alerts hitting target %)
- Maximum gain per alert
- Drawdown before success
- Performance by alert type
- Pattern comparison (flat-to-spike vs regular)

**Success threshold:** Default 30% gain

### 7. Telegram Integration

Alerts formatted with:
- Ticker and price
- Change percentage
- Volume metrics
- Sector
- Pattern flags
- Win probability estimate
- Recommended stop-loss
- Target price
- Position size recommendation
- Market conditions score
- TradingView chart link

## Data Storage

### Alert Files
```
momentum_data/
├── alerts_20251106_143215.json    # Timestamped snapshots
├── latest_alerts.json              # Most recent (overwritten)
└── paper_trades/
    ├── trade_history.json          # Completed trades
    └── active_positions.json       # Open positions
```

### Alert JSON Schema
```json
{
  "timestamp": "2025-11-06T14:32:15",
  "price_spikes": [
    {
      "ticker": "ABCD",
      "current_price": 5.25,
      "change_pct": 35.5,
      "volume": 1500000,
      "relative_volume": 150.5,
      "sector": "Technology",
      "alert_type": "flat_to_spike",
      "flat_analysis": {
        "is_flat": true,
        "flat_duration_minutes": 15.0,
        "flat_volatility": 2.1
      },
      "change_from_open": 42.3
    }
  ],
  "volume_climbers": [],
  "volume_newcomers": [],
  "premarket_volume_alerts": [],
  "premarket_price_alerts": []
}
```

### Logs
```
volume_momentum_tracker.log     # Detailed operations
telegram_alerts_sent.jsonl      # Sent alert history
```

## Authentication

TradingView data requires browser cookies:

```python
# Uses rookiepy to extract cookies from browser
# Requires active TradingView session
BROWSER = "firefox"  # or "chrome", "edge", "safari"
```

**Known issue:** rookiepy has compatibility problems with Python 3.14. Use Python 3.12 or 3.13.

## Configuration

### Environment

No `.env` file currently. Credentials passed via CLI:

```bash
python volume_momentum_tracker.py \
  --bot-token "TELEGRAM_BOT_TOKEN" \
  --chat-id "TELEGRAM_CHAT_ID"
```

### Key Parameters

Located in `volume_momentum_tracker.py`:

```python
monitor_interval = 120          # Scan every 2 minutes
max_history = 50               # Keep last 50 data points
limit = 200                    # Stocks to analyze
price_filter = 20              # Max price $20
immediate_threshold = 15       # Immediate alert %
flat_volatility_threshold = 3.0
min_flat_duration = 8
```

## Troubleshooting

### No data returned
1. Check TradingView session in browser
2. Verify browser selection matches logged-in browser
3. Try outside market hours with caution (limited data)

### Cookie extraction fails
1. Clear browser cache, re-login to TradingView
2. Try different browser
3. Check Python version (3.12/3.13 recommended)

### Telegram not sending
1. Verify bot token and chat ID
2. Check bot has permission to message chat
3. Review logs for rate limiting

### False positives
1. Check sector-specific multipliers
2. Review cooldown settings
3. Adjust momentum score thresholds

## Performance Considerations

- **Memory:** 30-minute sliding window per ticker, auto-cleanup
- **API calls:** 1 TradingView call per 2-minute cycle
- **Telegram:** Rate limited to avoid spam
- **CPU:** Minimal - mostly waiting between scans
