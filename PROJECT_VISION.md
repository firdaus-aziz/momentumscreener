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

### Current State
```
TradingView Screener → Python Tracker → Telegram Alerts → Manual Review
```

### Target State
```
Multi-Market Data → Momentum Engine → Risk Manager → Order Executor → Moomoo API
                         ↓                              ↓
                   Signal Validator              Position Manager
                         ↓                              ↓
                  Telegram Reports ←──────────── Performance Tracker
```

### Key Components to Build
1. **Moomoo API Integration** - Order execution layer
2. **Risk Manager** - Position sizing, stop-loss management, exposure limits
3. **Signal Validator** - Confirm signals before execution
4. **Position Manager** - Track open positions, manage exits
5. **Performance Tracker** - P&L tracking, win rate, drawdown monitoring
6. **SGX Momentum Module** - Parallel system for Singapore market

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

---

## Revision History

| Date | Changes |
|------|---------|
| 2026-01-01 | Initial vision document created |

---

*This document should be reviewed and updated quarterly or when significant strategic decisions are made.*
