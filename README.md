# MNQ / NQ Trend-Pullback Strategy (5-Minute)

This repository contains my first systematic implementation of a futures intraday strategy
based on ideas I developed while manually trading MNQ (Micro Nasdaq futures).

Because I don't have a direct historical feed for MNQ, I use **NQ=F (E-mini Nasdaq Futures)** 
from Yahoo Finance as a proxy, with 5-minute bars.

The code was implemented with **AI-assisted development** (ChatGPT-style tools) but the
**strategy idea, trading logic, and parameter choices are my own**.  
My goal is not to prove I can type everything from memory, but to show how I think about
trend, pullbacks, and risk in a systematic way.

---

## 1. Strategy Idea (from discretionary trading)

When I traded MNQ manually, I felt that:

- The **overall 60-period moving average on 5m** gave a good sense of trend direction  
  (uptrend / downtrend vs. choppy / flat).
- Entering **too late into a move** often led to chasing and whipsaw.
- Better entries tended to come from **pullbacks back toward a “fair value line”**,  
  which I model here as the **20-period moving average**.
- Once price returns to the trend and starts moving again, I want to enter *with* the trend,
  not against it.

So the core idea is:

> 1. Use **5m 60MA** to define trend direction (up / down).  
> 2. Wait for a **pullback toward 20MA** (no chasing).  
> 3. When price confirms in the direction of the trend, **enter**.  
> 4. If the 60MA slope reverses, **exit** (trend considered broken).

This is a very simple first version – more like a “prototype of my thinking”
rather than a final trading system.

---

## 2. Data

- Source: Yahoo Finance
- Symbol: `NQ=F` (E-mini Nasdaq futures, used as a proxy for MNQ)
- Timeframe: **5-minute bars**
- Lookback: last ~60 days (configurable in the code)

I use:

```python
import yfinance as yf

data = yf.download("NQ=F", interval="5m", period="60d")
# MNQ_backtest
3. Trading Rules (System Version)

All logic is implemented on 5-minute bars.

3.1 Indicators

ma20: 20-period simple moving average of Close

ma60: 60-period simple moving average of Close

ma60_slope: ma60[t] - ma60[t-5]
→ a rough way to capture the direction of the 60MA.

3.2 Entry Conditions

For each new 5m bar:

Compute:

close = current close

prev_close = previous bar close

ma20_prev = previous bar 20MA

slope = current ma60_slope

Define a pullback as:

abs(prev_close - ma20_prev) / prev_close < 0.5%

Long Entry (trend-up pullback):

slope > 0 (60MA trending up)

Price pulled back near ma20 (using previous bar)

Current bar close > previous close (price resumes up)

Short Entry (trend-down pullback):

slope < 0 (60MA trending down)

Price pulled back near ma20

Current bar close < previous close (price resumes down)

Only one position at a time: either long, short, or flat.

3.3 Exit Conditions

If I am long and slope becomes negative → exit at current close

If I am short and slope becomes positive → exit at current close

This roughly matches my discretionary idea:

"Exit when the 60MA trend breaks."

There is no explicit stop-loss or take-profit yet – that is a clear area for improvement.

4. Backtest Implementation (Python)

The backtest is intentionally simple:

Loop over all 5m bars starting after enough data for MA calculations.

Track:

position: 1 (long), -1 (short), 0 (flat)

entry_price

equity in points

For each bar:

Record current equity

If in position → check exit rules

If flat → check entry rules

The result is:

Total PnL (in points)

Number of trades

Win rate

An equity curve over time

See mnq_backtest.py for full implementation.

5. Sample Result (NQ=F, 5m, last 60 days)

Below is an example of the equity curve produced by this strategy
(using 1 unit per trade, PnL in index points; this is not slippage- or fee-adjusted):

This is not a production-ready strategy.
It is a starting point to:

Check whether my intuitive MNQ ideas have any structural edge.

Practice turning a personal trading concept into a systematic backtest.

Prepare for future improvements:

Time-of-day filters (e.g. around US cash open)

Volatility filters (skip ultra-choppy conditions)

Channel / Bollinger-band based take profit (e.g. exit at upper/lower band)

Risk-based position sizing and stop-loss

6. AI-Assisted Development (Honest note)

I used AI tools (like ChatGPT) to help:

Fix Python / pandas errors

Clean up loops and conditions

Speed up writing the backtest skeleton

However:

The strategy logic (60MA trend + 20MA pullback) is based on my own MNQ trading experience.

I have manually reviewed and modified the code, and I understand how the rules
map to my discretionary trading ideas.

My goal going forward is:

To rely less on copy-paste,

And more on my ability to modify, debug, and extend this code
(for example, adding channel-based exits and risk management).
