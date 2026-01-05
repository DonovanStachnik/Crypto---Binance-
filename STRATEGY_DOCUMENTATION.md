# Liquidity Sweep Day Trading Strategy - Documentation

## Overview

This is a **rule-based, non-repainting** day trading strategy designed for **5-minute charts** with higher timeframe confirmation. The strategy captures intraday expansions after liquidity sweeps, prioritizing low drawdown and strict risk management over high frequency.

**Target Markets**: Futures (ES, NQ, YM, etc.), Forex majors, liquid crypto pairs
**Recommended Timeframe**: 5-minute
**Strategy Type**: Mean reversion after liquidity sweeps with trend filter

---

## Core Strategy Logic

### 1. Session Tracking Module

```pine
// Lines 87-98
```

**What it does**: Tracks the high and low of each trading session (user-configurable).

**How it works**:
- Detects new session starts based on user input (default: 0930-1600 ET)
- Records session high/low using `var` to maintain state
- Updates high/low as price moves during the session
- Resets on each new session

**Key Variables**:
- `sessionHigh`: Highest price reached in current session
- `sessionLow`: Lowest price reached in current session

---

### 2. Liquidity Sweep Detection

```pine
// Lines 100-139
```

**What it does**: Identifies when price has "swept" liquidity above session high or below session low.

**How it works**:
- **Sweep Threshold**: Price must exceed session level by a configurable percentage (default 0.15%)
- **High Sweep**: `high > sessionHigh + sweepDistance`
- **Low Sweep**: `low < sessionLow - sweepDistance`
- Records the bar where sweep occurred
- Resets on new session

**Why this matters**: Liquidity sweeps often represent stop hunts or liquidity grabs that can reverse, creating trading opportunities.

---

### 3. Re-Entry Detection

```pine
// Lines 141-154
```

**What it does**: Confirms price has returned inside the session range after the sweep.

**How it works**:
- After a high sweep, checks if `close < sessionHigh` (bearish setup)
- After a low sweep, checks if `close > sessionLow` (bullish setup)
- Must occur within N candles (default: 10) of the sweep
- This confirms the sweep was a false breakout

**Why this matters**: Re-entry suggests the breakout failed and price is returning to value, increasing probability of a reversal.

---

### 4. Regime Filter (ADX)

```pine
// Lines 156-169
```

**What it does**: Filters trades based on market regime to avoid poor setups.

**How it works**:
- Calculates ADX (Average Directional Index) using custom function
- **ADX < 20**: Mean reversion regime (ranging) → ALLOW trades
- **ADX > 25**: Trend continuation regime → BLOCK liquidity sweep trades
- **20-25**: Dead zone → NO trades

**Why this matters**: Liquidity sweep mean reversion works best in ranging markets. High ADX indicates strong trends where breakouts are more likely to continue.

**Custom ADX Function** (`calcADX`):
- Calculates +DI and -DI from directional movement
- Uses RMA smoothing (Wilder's method)
- Returns ADX value and directional indicators

---

### 5. Higher Timeframe Trend Filter

```pine
// Lines 171-179
```

**What it does**: Ensures trades align with 1-hour trend direction.

**How it works**:
- Fetches 1H EMA (default: 20 period) using `request.security()`
- Uses `lookahead=barmerge.lookahead_off` to prevent repainting
- Defines trend direction:
  - **Bullish**: HTF close > HTF EMA
  - **Bearish**: HTF close < HTF EMA

**Why this matters**: Trading with the higher timeframe trend increases win rate and reduces drawdown.

---

### 6. Volume Confirmation

```pine
// Lines 181-185
```

**What it does**: Confirms genuine interest in the move.

**How it works**:
- Calculates 20-period volume SMA
- Requires current volume > (average * multiplier)
- Default multiplier: 1.0 (can be adjusted)

**Why this matters**: Volume confirms participation. Low volume sweeps are more likely to fail.

---

### 7. Candle Structure Filter

```pine
// Lines 47-56
```

**What it does**: Ensures entry candle has strong directional structure.

**How it works**:
- Calculates body-to-range ratio
- **Bullish**: Close > Open and body ≥ 50% of range
- **Bearish**: Open > Close and body ≥ 50% of range

**Why this matters**: Strong candle bodies indicate conviction. Doji or spinning tops suggest indecision.

---

### 8. Time Filters

```pine
// Lines 187-197
```

**What it does**: Limits trading to high-liquidity sessions.

**How it works**:
- **London Session**: 03:00-11:00 ET (toggleable)
- **NY Session**: 09:30-16:00 ET (toggleable)
- Both can be enabled simultaneously

**Why this matters**: Major sessions have higher volume and cleaner price action.

---

### 9. Entry Logic

```pine
// Lines 199-217
```

**What it does**: Combines all filters to generate trade signals.

**Long Entry Requires**:
1. Low swept and price re-entered range
2. Mean reversion regime (ADX < 20)
3. HTF bullish (price > 1H EMA)
4. Bullish candle structure (body ≥ 50% of range)
5. Volume confirmation
6. Within trading session hours
7. No existing position

**Short Entry Requires**:
1. High swept and price re-entered range
2. Mean reversion regime (ADX < 20)
3. HTF bearish (price < 1H EMA)
4. Bearish candle structure
5. Volume confirmation
6. Within trading session hours
7. No existing position

---

### 10. Risk Management & Position Sizing

```pine
// Lines 219-280
```

**What it does**: Implements **fixed fractional risk** per trade.

**How it works**:

1. **Stop Loss Calculation** (two modes):
   - **Sweep Level Mode**: Stop beyond the sweep level by sweep distance
   - **ATR Mode**: Stop = Entry ± (ATR × multiplier)

2. **Position Sizing**:
   ```
   Risk per trade = Account Equity × (Risk % / 100)
   Risk per share = |Entry - Stop|
   Position size = Risk per trade / Risk per share
   ```

3. **Take Profit**:
   - Minimum 1:2 risk:reward ratio (configurable)
   - `TP = Entry ± (Risk per share × R:R ratio)`

4. **Trailing Stop** (optional):
   - Activates after 1R profit (configurable)
   - Trails at 0.5R distance (configurable)

**Why this matters**: Fixed fractional risk ensures you never risk more than X% of your account, regardless of market volatility or position size.

---

### 11. Visual Elements

```pine
// Lines 282-338
```

**What it does**: Provides clear visual feedback on the chart.

**Plots**:
- Session high/low lines (red/green)
- HTF EMA (blue line)
- Sweep markers (triangles)
- Entry signals (labels)
- Regime background color (green = mean rev, blue = trend, gray = dead zone)
- Info table showing current state

---

## Strategy Parameters (All Configurable)

### Risk Management
- **Risk Per Trade**: 0.1-5% (default: 1%)
- **Risk:Reward Ratio**: 1.0-5.0 (default: 2.0)
- **Use Trailing Stop**: true/false
- **Stop Loss Mode**: "Sweep Level" or "ATR-Based"
- **ATR Multiplier**: 0.5-5.0 (default: 1.5)

### Session Settings
- **Trading Session**: Time range (default: "0930-1600")
- **Sweep Threshold**: 0.05-1.0% (default: 0.15%)
- **Max Candles for Re-entry**: 3-30 (default: 10)

### Regime Filter
- **ADX Period**: 5-50 (default: 14)
- **ADX Max for Mean Reversion**: 10-30 (default: 20)
- **ADX Min for Trend**: 20-40 (default: 25)

### Trend Filter
- **Higher Timeframe**: Any (default: 60m)
- **HTF EMA Length**: 5-200 (default: 20)

### Volume Filter
- **Volume MA Period**: 5-100 (default: 20)
- **Volume Multiplier**: 0.5-3.0 (default: 1.0)

### Candle Structure
- **Min Body/Range Ratio**: 0.3-0.9 (default: 0.5)

### Time Filters
- **Trade London Session**: true/false
- **Trade NY Session**: true/false

---

## How to Use in TradingView

### 1. Installation
1. Open TradingView
2. Open Pine Editor (bottom panel)
3. Click "Create new strategy"
4. Paste the code from `Liquidity_Sweep_Day_Trading_5m.pine`
5. Click "Save" and name it
6. Click "Add to Chart"

### 2. Initial Setup
1. **Timeframe**: Set chart to 5-minute
2. **Market**: Works on any liquid futures contract (ES, NQ, YM, RTY, CL, GC) or Forex/Crypto
3. **Settings**: Right-click strategy → "Settings" to access all parameters

### 3. Recommended Starting Settings

**For ES Futures (S&P 500 E-mini)**:
- Risk Per Trade: 1%
- R:R Ratio: 2.0
- Stop Loss Mode: Sweep Level
- Sweep Threshold: 0.15%
- Session: 0930-1600 (ET)
- ADX Period: 14
- HTF: 60 minutes
- Time Filters: Both London and NY enabled

**For Crypto (BTC, ETH)**:
- Risk Per Trade: 0.5-1%
- Stop Loss Mode: ATR-Based (more volatility)
- ATR Multiplier: 2.0
- Sweep Threshold: 0.2%
- Session: Adjust to your timezone

### 4. Commission & Slippage
The strategy includes realistic assumptions:
- **Commission**: 0.06% (adjust for your broker)
- **Slippage**: 2 ticks
- **Margin**: 100 (1:1 leverage by default)

---

## Backtesting Guidelines

### ⚠️ CRITICAL: Avoiding Over-Optimization

**The Cardinal Sin**: Tweaking parameters until backtest results look amazing, then losing money live.

### Safe Backtesting Process

#### 1. **Use Walk-Forward Analysis**
- **Training Period**: Optimize on 60% of historical data
- **Testing Period**: Test on the next 20% (unseen data)
- **Validation Period**: Final 20% for out-of-sample validation
- Only proceed to live if performance holds across all three periods

#### 2. **Sample Size Requirements**
- **Minimum 100 trades** in backtest
- **Minimum 6 months** of data
- If sample is smaller, results are statistically unreliable

#### 3. **Key Metrics to Watch**

**Good Strategy Profile**:
- **Win Rate**: 45-60% (mean reversion typically 50-55%)
- **Profit Factor**: >1.5
- **Sharpe Ratio**: >1.0
- **Max Drawdown**: <15-20%
- **Average Win > Average Loss** (1.5-2.5x)
- **Consistent monthly returns** (not one huge month)

**Red Flags**:
- Win rate >70% (likely curve-fitted)
- Profit factor >3.0 (unrealistic)
- Max drawdown <5% (not testing enough scenarios)
- All profits from 1-2 months
- Very few trades (<50 in 6 months)

#### 4. **Parameter Stability Test**
1. Note your "optimal" parameters
2. Change each by ±20%
3. If performance collapses → over-optimized
4. Good strategy should degrade gracefully

**Example**:
- If ADX threshold = 20 works great
- But ADX = 18 or 22 fails completely
- → You've curve-fitted to noise

#### 5. **Market Regime Analysis**
Test separately on:
- **Bull markets** (2021, 2023-2024)
- **Bear markets** (2022)
- **Range-bound markets** (2015-2016)

Good strategy should profit in at least 2 of 3 regimes.

#### 6. **Avoid These Traps**

❌ **The "Best Settings" Trap**
- Don't search for the "perfect" ADX period or EMA length
- Use standard values (14, 20) unless you have a specific reason

❌ **The "Recent Data" Trap**
- Don't only backtest the last 3 months
- Recent market may not represent future conditions

❌ **The "Every Parameter" Trap**
- Don't optimize all 15+ parameters simultaneously
- Fix most parameters, only optimize 2-3 critical ones

❌ **The "Confirmation Bias" Trap**
- If backtest looks bad, don't immediately tweak settings
- Bad results are valuable information

### 7. **Recommended Optimization Approach**

**Phase 1: Fixed Parameters (Start Here)**
- Use default settings as-is
- Backtest 6-12 months
- Analyze results

**Phase 2: Light Optimization (If Needed)**
Only optimize these in order:
1. **Risk:Reward Ratio** (1.5, 2.0, 2.5)
2. **Sweep Threshold** (0.1%, 0.15%, 0.2%)
3. **ADX Mean Reversion Max** (18, 20, 22)

**Phase 3: Validation**
- Test optimized settings on out-of-sample data
- If results hold → proceed to paper trading
- If results degrade → return to defaults

---

## Live Trading Checklist

Before going live with real money:

### ✅ Pre-Flight Checks

1. **Paper Trade First**
   - Trade strategy on paper/sim for 2-4 weeks minimum
   - Verify signals match backtest logic
   - Check execution at your broker

2. **Start Small**
   - Begin with 0.25-0.5% risk per trade
   - Increase only after 20-30 successful trades

3. **Broker Compatibility**
   - Confirm your broker supports limit orders
   - Verify commission rates match backtest assumptions
   - Test with 1 micro contract first (MES, MNQ, etc.)

4. **Execution Plan**
   - How will you monitor signals? (TradingView alerts, manual checking)
   - Can you take trades during session hours?
   - What if you miss an entry signal?

5. **Risk Management Verification**
   - Double-check position sizing calculations
   - Verify stop loss orders are placed correctly
   - Confirm account has sufficient margin

6. **Psychological Preparation**
   - Are you comfortable with expected drawdowns (10-15%)?
   - Can you handle a string of 4-5 losses?
   - Do you trust the strategy enough to not interfere?

---

## Common Questions

### Q: Why does the strategy sometimes show no trades for days?

**A**: This is a **quality over quantity** strategy. The confluence of:
- Liquidity sweep
- Re-entry
- Correct ADX regime
- HTF trend alignment
- Volume confirmation
- Time window

...means signals are selective. This is intentional. **2-5 setups per week** is normal for a liquid market like ES.

---

### Q: Can I use this on lower timeframes (1m, 3m)?

**A**: Not recommended. Lower timeframes have:
- More noise
- Higher commission impact
- More false signals
- Increased execution difficulty

5-minute is the minimum for reliable mean reversion setups.

---

### Q: What markets work best?

**Best**: ES, NQ (high liquidity, tight spreads, active sessions)
**Good**: YM, RTY, major Forex pairs (EUR/USD, GBP/USD)
**Possible**: Liquid crypto (BTC, ETH on major exchanges)
**Avoid**: Low-volume stocks, exotic pairs, illiquid instruments

---

### Q: Why do I need HTF trend filter if this is mean reversion?

**A**: You're not trying to catch the top/bottom of the entire move. You're catching a **reversion within a larger trend**. Example:
- 1H trend is bullish (price > 1H EMA)
- Price sweeps session low (temporary dip)
- You buy the re-entry, expecting resumption of 1H trend

Trading against HTF trend (counter-trend mean reversion) has lower win rate and worse R:R.

---

### Q: What if ADX is always in the "dead zone" on my market?

**A**: Some markets are persistently trending or ranging. Options:
1. Adjust thresholds (e.g., 18/23 instead of 20/25)
2. Use the strategy only when ADX is favorable
3. Switch to a different market

---

### Q: Can I remove filters to get more trades?

**A**: You can, but you'll likely decrease edge. Each filter serves a purpose:
- **Remove volume filter** → more signals, but lower quality
- **Remove HTF filter** → trade against trend, worse R:R
- **Remove ADX filter** → entries during strong trends (worse for mean rev)

**Better approach**: If you want more trades, run the strategy on multiple correlated markets (ES + NQ) or add a second strategy (trend following).

---

### Q: How do I know if the strategy stops working?

**Warning Signs**:
1. **Drawdown exceeds historical max by 50%+**
   - E.g., if backtest max DD = 12%, live DD = 20%+
2. **Win rate drops significantly**
   - E.g., backtest = 55%, live = 35-40% after 30+ trades
3. **Average loss > Average win**
   - R:R profile has inverted
4. **Consecutive losses exceed historical**
   - If backtest max = 6 losses, you hit 10+

**Action**: Pause trading, re-run backtest on recent data, check if market regime has fundamentally changed.

---

## Performance Expectations (Realistic)

Based on sound strategy design and proper execution:

### Conservative Estimates
- **Monthly Return**: 3-8% (on low leverage)
- **Win Rate**: 48-58%
- **Profit Factor**: 1.4-2.0
- **Max Drawdown**: 10-18%
- **Trades per Month**: 8-20 (on single market)

### What This Strategy Is NOT
- ❌ A "get rich quick" system
- ❌ A 80%+ win rate strategy
- ❌ A zero-drawdown approach
- ❌ A high-frequency scalping bot
- ❌ Guaranteed profits

### What This Strategy IS
- ✅ A systematic, rule-based approach
- ✅ A risk-managed framework
- ✅ A selective, quality-focused method
- ✅ A foundation you can build on
- ✅ A teachable, understandable system

---

## Customization Ideas

Once you're comfortable with the base strategy:

### 1. **Multi-Timeframe Entries**
- Add 15m confirmation candle before entry
- Require 15m also bullish/bearish structure

### 2. **Additional Filters**
- RSI divergence (price makes lower low, RSI makes higher low)
- VWAP reversion (enter when price returns to VWAP)
- Order flow imbalance (requires footprint charts)

### 3. **Dynamic Risk Adjustment**
- Reduce risk after 3 consecutive losses
- Increase risk after 3 consecutive wins (carefully)

### 4. **Multiple Targets**
- Take 50% off at 1R
- Trail remaining 50% to 2R+

### 5. **Session-Specific Logic**
- Different parameters for London vs NY
- Skip first 30 minutes of NY open (high volatility)

---

## Module Summary

| Module | Purpose | Lines |
|--------|---------|-------|
| **User Inputs** | All configurable parameters | 12-55 |
| **Functions** | ADX calculation, candle structure checks | 57-67 |
| **Session Tracking** | Track daily high/low | 69-98 |
| **Liquidity Sweep Detection** | Identify stop hunts | 100-139 |
| **Re-Entry Detection** | Confirm false breakout | 141-154 |
| **Regime Filter (ADX)** | Mean reversion vs trend mode | 156-169 |
| **HTF Trend Filter** | 1H trend direction | 171-179 |
| **Volume Filter** | Confirm participation | 181-185 |
| **Time Filters** | London/NY sessions | 187-197 |
| **Entry Logic** | Combine all conditions | 199-217 |
| **Risk Management** | Position sizing, stops, targets | 219-280 |
| **Trailing Stop** | Optional profit protection | 282-311 |
| **Plotting** | Visual elements | 313-338 |

---

## Final Notes

### On Risk
- **Never risk more than 1-2% per trade**
- Most successful traders risk 0.5-1%
- Your first goal is survival, not profit

### On Discipline
- **Follow the rules exactly**
- Don't skip trades because "it doesn't feel right"
- Don't take trades that don't meet all criteria
- The strategy is only as good as your execution

### On Expectations
- **Expect losing streaks**
- Even a 60% win rate means 4 out of 10 trades lose
- Drawdowns are normal and healthy
- Judge performance over 50-100 trades, not 5-10

### On Learning
- **Keep a trading journal**
- Note: setup quality, execution, emotions, results
- Review weekly/monthly
- Iterate slowly based on data, not feelings

---

## Support & Resources

**TradingView Pine Script Documentation**:
https://www.tradingview.com/pine-script-docs/en/v5/

**Risk Management Calculator**:
https://www.myfxbook.com/forex-calculators/position-size-calculator

**Walk-Forward Analysis Tools**:
- TradingView Strategy Tester (built-in)
- AmiBroker (advanced users)
- Python backtrader library

---

## Version History

**v1.0** (Current)
- Initial release
- Core liquidity sweep logic
- ADX regime filter
- HTF trend filter
- Fixed fractional risk management
- Time filters
- Optional trailing stop

---

## License

This strategy is provided for educational purposes. Use at your own risk.

Past performance does not guarantee future results.

---

**Remember**: The best strategy in the world is useless without proper risk management and disciplined execution. Trade small, learn continuously, and protect your capital above all else.

Good luck!
