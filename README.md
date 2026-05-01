# 📊 Freqtrade Backtesting & Data Engine

[![ClawHub Skill](https://img.shields.io/badge/ClawHub-Skill-blue)](https://clawhub.ai/djc00p/freqtrade-backtester) [![Agent Skill](https://img.shields.io/badge/Agent-Skill-blue)](#) [![Freqtrade Crypto](https://img.shields.io/badge/Freqtrade-Crypto-green)](https://github.com/freqtrade/freqtrade)

**This tool helps you validate Freqtrade strategies. You can download historical market data, run large-scale backtests, and analyze performance to move from strategy ideas to live trading.**

The **Freqtrade Backtester** turns raw historical market data into useful performance metrics. It lets you test your trading logic against past market ups and downs, so you can make sure your strategies are strong before risking real money.

---

## 📥 Data Ingestion & Management

Before a backtest can occur, the engine requires high-fidelity historical data. This module manages the downloading and caching of candlestick (OHLCV) data from exchanges (default: Kraken).

### ⚠️ The "Single-Pair" Constraint

To prevent data corruption and command failures within the Docker environment, **always download one trading pair at a time.** Do not pass multiple pairs to a single command.

### Usage

Run the following command to fetch historical data for a specific timeframe:

```bash
docker-compose run --rm freqtrade download-data \
  --exchange kraken \
  --pairs BTC/USDT \
  --timeframe 5m \
  --timerange 20240101-
```

**Parameters:**

* `--pairs`: The trading pair to fetch (e.g., `ETH/USDT`).
* `--timeframe`: The candlestick interval (e.g., `5m`, `1h`, `1d`).
* `--timerange`: The historical window in `YYYYMMDD-YYYYMMDD` format. Use `20240101-` to fetch everything from Jan 1st, 2024, to the present.
* **Pro-Tip:** Use `--erase` if you are extending an existing timeframe to prevent overlapping data errors.

---

## 🧪 Execution: Running the Backtest

The backtest engine simulates trades by applying your strategy logic to the downloaded historical dataset.

```bash
docker-compose run --rm freqtrade backtesting \
  --strategy YourStrategyName \
  --timerange 20240101-20260101 \
  --export trades \
  --export-filename user_data/backtest_results/results.json
```

**Key Flags:**

* `--strategy`: The exact class name of your Python strategy.
* `--export trades`: **Critical.** Generates a detailed JSON log of every trade executed, which is essential for deep-dive analysis.
* `--export-filename`: Defines the destination for your performance report.

---

## 📈 Quantitative Performance Metrics

When analyzing backtest results, look beyond simple profit. Use these four pillars to judge a strategy's viability:

| Metric | Target Threshold | Meaning |
| :--- | :--- | :--- |
| **Win Rate** | `> 55%` | The percentage of trades that closed in profit. |
| **Max Drawdown** | `< 20%` | The largest peak-to-trough decline. High drawdown indicates excessive risk. |
| **Sharpe Ratio** | `> 1.5` | Risk-adjusted return. High Sharpe means consistent, smooth equity growth. |
| **Avg Profit/Trade**| `> 0` | The average margin per trade. Must be large enough to cover slippage/fees. |

> [!IMPORTANT]
>
> ### The Profitability Paradox
>
> **A high win rate does not equal a profitable strategy.**
> A strategy with a **70% win rate** that loses **5%** on every loss will eventually go to zero. Conversely, a strategy with a **40% win rate** that captures **2%** gains while only losing **1%** is a mathematical winner. **Focus on the Profit/Loss ratio, not the Win Rate.**

---

🔀 **The Iteration Protocol**
To optimize a strategy without "overfitting," follow this scientific loop:

1. **Baseline:** Run a backtest on a fixed 6-month window.
2. **Isolate:** Change **exactly one** parameter (e.g., adjust the RSI threshold).
3. **Compare:** Run the same 6-month backtest.
4. **Validate:** If the performance metric (Sharpe/Drawdown) improved, keep the change. If it degraded, revert immediately.

---

## 🔐 Security & Environment Configuration

To maintain security, **never hardcode API keys in your strategy or config files.** Use environment variables to inject secrets into the Docker container at runtime.

### The Double-Underscore (`__`) Pattern

Freqtrade uses a specific syntax to map environment variables to the `config.json` hierarchy.

**Example:** To set the exchange secret, use:
`export FREQTRADE__EXCHANGE__SECRET=your_api_secret_here`

**In your `config.json`:**

```json
{
  "exchange": {
    "key": "",
    "secret": ""
  }
}
```

*This approach ensures your credentials are never committed to version control (Git).*

---
**Original implementation by [@djc00p](https://github.com/djc00p)**
