# Momentum Screener Setup Status
*Last Updated: 2026-01-01*

## Completed

### Telegram Bot Setup
- **Bot Name:** Firdaus Momentum Alerts
- **Bot Username:** @firdaus_momentum_bot
- **Bot Token:** `8322136479:AAHyCr9ZBo2nMqwAjhK6ROcvNebP1Cqt7Bk`
- **Chat ID:** `282646714`
- **Status:** Working - test message sent successfully

### Brokerage Decision
- **Chosen:** Moomoo Singapore (over Moomoo Malaysia)
- **Reasons:**
  - Lifetime $0 commission (vs 0.03% after 180 days in MY)
  - No Malaysian stamp duty
  - Lower margin rate (4.8% vs 6.8%)
  - MAS regulation

## Blocked

### Python Environment - rookiepy Installation
**Problem:** `rookiepy` fails to compile on Python 3.14

**Error:** The package has Rust dependencies (PyO3) that aren't compatible with Python 3.14.

**Potential Fix:**
```bash
export PYO3_USE_ABI3_FORWARD_COMPATIBILITY=1
pip install rookiepy
```

**Alternative:** Use Python 3.12 or 3.13 instead of 3.14

**What's Installed (in venv):**
- pandas 2.3.3
- python-telegram-bot 22.5
- tradingview-screener 3.0.0
- yfinance 1.0

**Still Needed:**
- rookiepy (for TradingView cookie extraction)

## Pending Tasks

### 1. Fix Python Environment
- Either fix rookiepy on Python 3.14 with compatibility flag
- Or recreate venv with Python 3.12/3.13

### 2. Test Momentum Screener
```bash
source venv/bin/activate
python3 volume_momentum_tracker.py --test-bot \
  --bot-token "8322136479:AAHyCr9ZBo2nMqwAjhK6ROcvNebP1Cqt7Bk" \
  --chat-id "282646714"
```

### 3. Open Moomoo SG Account
- Download moomoo app or visit moomoo.com/sg
- Complete KYC verification
- Fund account with SGD/USD

### 4. Set Up Moomoo OpenAPI (for automated trading)
- Download OpenD gateway from https://www.moomoo.com/download/OpenAPI
- Install moomoo-api: `pip install moomoo-api`
- Configure credentials

### 5. Create Trade Executor Integration
- Create `moomoo_executor.py` module
- Integrate with volume_momentum_tracker.py
- Implement entry/exit logic based on EMA signals

## Running the Screener (Once Environment Fixed)

### Single Scan
```bash
source venv/bin/activate
python3 volume_momentum_tracker.py --single
```

### Continuous with Telegram Alerts
```bash
source venv/bin/activate
python3 volume_momentum_tracker.py --continuous \
  --bot-token "8322136479:AAHyCr9ZBo2nMqwAjhK6ROcvNebP1Cqt7Bk" \
  --chat-id "282646714"
```

### With Paper Trading
```bash
source venv/bin/activate
python3 volume_momentum_tracker.py --continuous --paper-trading \
  --bot-token "8322136479:AAHyCr9ZBo2nMqwAjhK6ROcvNebP1Cqt7Bk" \
  --chat-id "282646714"
```
