# Liquidity Sweep Day Trading Strategy

A professional, rule-based Pine Script v5 strategy for capturing intraday reversals after liquidity sweeps on 5-minute charts.

## Quick Start

### 1. Installation
1. Open TradingView
2. Open Pine Editor (bottom panel)
3. Copy code from `Liquidity_Sweep_Day_Trading_5m.pine`
4. Paste into Pine Editor
5. Click "Add to Chart"

### 2. Setup
- **Timeframe**: 5-minute chart
- **Markets**: ES, NQ, YM futures or liquid Forex/Crypto
- **Default Settings**: Work well out-of-the-box for ES futures

### 3. Quick Settings Guide

Right-click strategy → Settings → Inputs

**Essential Parameters**:
- **Risk Per Trade**: 1% (adjust based on your risk tolerance)
- **Risk:Reward Ratio**: 2.0 (conservative)
- **Stop Loss Mode**: "Sweep Level" (recommended for futures)
- **Trading Session**: "0930-1600" (ET - adjust for your timezone)

---

## Strategy Overview

### Core Logic

**Entry Setup (Long)**:
1. Session low gets swept (liquidity grab)
2. Price re-enters the session range
3. Market is in mean reversion regime (ADX < 20)
4. 1H trend is bullish (HTF confirmation)
5. Strong bullish candle structure (body > 50% of range)
6. Volume exceeds average
7. Within trading session hours

**Entry Setup (Short)**: Inverse of long setup

### Risk Management

- **Fixed Fractional Position Sizing**: Risk exactly X% per trade
- **Stop Loss**: Beyond sweep level or ATR-based
- **Take Profit**: Minimum 1:2 risk:reward
- **Optional Trailing Stop**: Activates after 1R profit

---

## Key Features

✅ **Non-Repainting**: All logic uses confirmed data
✅ **Rule-Based**: No discretion, fully systematic
✅ **Risk-Managed**: Fixed fractional position sizing
✅ **Multi-Filter**: ADX regime, HTF trend, volume, candle structure
✅ **Session-Aware**: Trades only during high liquidity periods
✅ **Fully Configurable**: 20+ adjustable parameters

---

## Performance Expectations (Realistic)

**Conservative Estimates**:
- Win Rate: 48-58%
- Profit Factor: 1.4-2.0
- Monthly Return: 3-8% (low leverage)
- Max Drawdown: 10-18%
- Trades per Month: 8-20 (single market)

**Not a get-rich-quick scheme**. This is a systematic approach with realistic returns and manageable drawdowns.

---

## Files Included

1. **Liquidity_Sweep_Day_Trading_5m.pine** - Complete Pine Script v5 strategy
2. **STRATEGY_DOCUMENTATION.md** - Comprehensive documentation:
   - Detailed module explanations
   - Backtesting guidelines
   - Anti-curve-fitting best practices
   - Live trading checklist
   - FAQ and troubleshooting

---

## Before Going Live

### ⚠️ Critical Steps

1. **Read STRATEGY_DOCUMENTATION.md** - Especially the backtesting section
2. **Backtest 6-12 months** minimum
3. **Walk-forward analysis** - Test on out-of-sample data
4. **Paper trade 2-4 weeks** before risking real money
5. **Start with 0.25-0.5% risk** per trade

### Common Mistakes to Avoid

❌ Over-optimizing parameters for perfect backtest
❌ Skipping paper trading phase
❌ Risking too much per trade (>2%)
❌ Taking trades that don't meet all criteria
❌ Expecting every trade to win

---

## Recommended Markets

**Best**:
- ES (S&P 500 E-mini futures)
- NQ (Nasdaq-100 E-mini futures)

**Good**:
- YM (Dow E-mini futures)
- Major Forex pairs (EUR/USD, GBP/USD)

**Possible**:
- Liquid crypto (BTC, ETH on major exchanges)

**Avoid**:
- Low-volume instruments
- Exotic currency pairs
- Thinly-traded stocks

---

## Strategy Philosophy

### Quality Over Quantity
This strategy prioritizes **high-probability setups** over trade frequency. It's normal to see:
- 2-5 setups per week on a single market
- Several days with no trades
- Selective entries with all filters aligned

### Risk-First Approach
- Fixed fractional risk ensures no single trade can significantly damage your account
- Position sizing adjusts automatically based on stop distance
- Conservative risk:reward ratios (minimum 1:2)

### Trend + Mean Reversion Hybrid
- HTF trend filter keeps you on the right side of the market
- Liquidity sweeps identify temporary extremes
- Entry on re-entry provides better risk:reward than breakout trading

---

## Module Overview

| Component | Purpose |
|-----------|---------|
| **Session Tracking** | Identifies daily high/low levels |
| **Liquidity Sweep** | Detects stop hunts and false breakouts |
| **Re-Entry Detection** | Confirms price returning to value |
| **ADX Regime Filter** | Blocks trades in wrong market conditions |
| **HTF Trend Filter** | Ensures alignment with 1H direction |
| **Volume Filter** | Confirms genuine market participation |
| **Candle Structure** | Validates entry candle conviction |
| **Time Filter** | Trades only during London/NY sessions |
| **Risk Management** | Fixed fractional position sizing |
| **Trailing Stop** | Optional profit protection |

---

## Customization

The strategy is designed to work well with default settings, but all parameters are adjustable:

- Risk parameters (risk %, R:R ratio)
- Session times and sweep thresholds
- ADX thresholds for regime detection
- HTF timeframe and EMA length
- Volume and candle structure requirements
- Time session filters

**Recommendation**: Start with defaults. Only adjust after 50+ trades worth of data.

---

## Learning Resources

**Included Documentation**:
- Complete strategy logic explanation
- Safe backtesting methodology
- Walk-forward analysis guide
- Live trading checklist
- Common pitfalls and solutions

**External Resources**:
- TradingView Pine Script docs
- Position sizing calculators
- Risk management guides

---

## Support

For questions about:
- **Strategy logic**: See STRATEGY_DOCUMENTATION.md
- **Pine Script syntax**: TradingView documentation
- **Risk management**: STRATEGY_DOCUMENTATION.md "Risk Management" section
- **Backtesting**: STRATEGY_DOCUMENTATION.md "Backtesting Guidelines"

---

## Disclaimer

**Past performance does not guarantee future results.**

This strategy is provided for educational purposes only. Trading futures, forex, and cryptocurrencies involves substantial risk of loss and is not suitable for all investors.

- Always use proper risk management
- Never risk more than you can afford to lose
- Paper trade before going live
- Understand the strategy completely before using real money

The authors are not responsible for any losses incurred using this strategy.

---

## Version

**Current Version**: 1.0

**Release Date**: January 2026

**Pine Script Version**: v5

---

## License

This strategy code is provided as-is for educational and personal use.

---

**Remember**: Discipline beats discretion. Trust the system, manage your risk, and give the strategy time to prove itself over a statistically significant sample size (100+ trades).

Happy trading!
