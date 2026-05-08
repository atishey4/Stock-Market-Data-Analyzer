# Stock Market Data Analyzer

![Python](https://img.shields.io/badge/Python-3.10%2B-blue)
![FastAPI](https://img.shields.io/badge/FastAPI-API-009688)

A Python project for downloading market data, calculating technical indicators, running a simple SMA crossover backtest, tracking portfolio positions, evaluating alerts, and producing charts and CSV reports.

The project is designed as an end-to-end learning system: raw market data flows into SQLite, indicators are computed with `ta`, strategy performance is evaluated, and results are exposed through a terminal pipeline, FastAPI endpoints, and a Streamlit dashboard.

## Disclaimer

This project is for educational purposes only. It is not financial advice, investment advice, or a recommendation to buy, sell, or hold any security. Market data may be delayed, incomplete, or inaccurate. Always do your own research and consult a qualified financial professional before making investment decisions.

## Tech Stack

- Python
- yfinance
- pandas and numpy
- ta
- SQLite
- FastAPI
- Streamlit
- Plotly
- Matplotlib and Seaborn
- APScheduler

## Folder Structure

```text
Stock-Market-Data-Analyzer/
|-- api/
|   `-- app.py
|-- data/
|-- db/
|   |-- init_db.py
|   `-- schema.sql
|-- docs/
|-- images/
|-- jobs/
|   `-- alerts.py
|-- notebooks/
|-- outputs/
|-- reports/
|-- src/
|   |-- app_streamlit.py
|   |-- backtest.py
|   |-- indicators.py
|   |-- ingest.py
|   |-- plots.py
|   |-- portfolio.py
|   `-- report.py
|-- main.py
|-- requirements.txt
`-- README.md
```

## Setup

Create and activate a virtual environment, then install dependencies:

```bash
python -m venv .venv
.venv\Scripts\activate
pip install -r requirements.txt
```

Initialize the SQLite database:

```bash
python db/init_db.py
```

## Run The Full Pipeline

The main pipeline initializes the database, downloads candles, computes indicators, runs the SMA backtest, generates charts, creates CSV reports, and prints a terminal summary.

```bash
python main.py
```

Default watchlist:

```python
["RELIANCE.NS", "TCS.NS", "INFY.NS", "^NSEI"]
```

If Yahoo Finance cannot return a ticker, the ingestion layer falls back to:

```text
data/{ticker}.csv
```

## Run The Streamlit Dashboard

```bash
streamlit run src/app_streamlit.py
```

The dashboard includes:

- Ticker dropdown and date range controls
- Latest daily close, 52-week high/low, daily return, and annualized volatility KPIs
- Candlestick chart with SMA20 and SMA50
- RSI chart with 70/30 levels
- MACD line and histogram chart
- Bollinger Bands chart
- SMA backtest metrics and equity curve

The dashboard uses the latest completed daily candle from Yahoo Finance. During market hours, this is usually the previous trading day's close, not a live market price.

## Run The FastAPI App

```bash
uvicorn api.app:app --reload
```

Useful endpoints:

```text
POST /refresh/{ticker}
GET  /chart/{ticker}?days=252
POST /backtest/sma
GET  /portfolio
GET  /alerts
POST /alerts
```

Example SMA backtest request:

```json
{
  "ticker": "RELIANCE.NS",
  "fee_bps": 5,
  "start": "2024-05-01",
  "end": "2026-05-01",
  "save": true
}
```

## Backtest Strategy

The included strategy is a simple long-only SMA crossover system:

- Go long when SMA20 is above SMA50.
- Stay flat when SMA20 is below or equal to SMA50.
- Apply the signal on the next trading day to reduce lookahead bias.
- Subtract transaction costs whenever the position changes.

Fees are entered in basis points. For example, `fee_bps = 5` means a 0.05% transaction cost on entry or exit.

## Sample Backtest Output

| Ticker | Latest Close | Signal | RSI14 | Sharpe | PnL |
|---|---:|---|---:|---:|---:|
| RELIANCE.NS | 1,436.20 | HOLD | 59.28 | 0.09 | 0.71% |
| TCS.NS | 2,401.40 | HOLD | 39.24 | -0.69 | -13.02% |
| INFY.NS | 1,162.70 | HOLD | 34.36 | 0.06 | -0.46% |
| ^NSEI | 24,326.65 | HOLD | 55.33 | 0.35 | 4.97% |

Results will vary depending on the date range, downloaded data, and transaction cost assumptions.

## Charts

Generated charts are saved to `outputs/`:

- Price with moving averages: close price with SMA20 and SMA50 overlays
- Daily returns: histogram with a normal distribution curve
- Rolling volatility: 30-day annualized volatility
- Bollinger Bands: close price with upper, middle, and lower bands shaded

CSV analysis reports are saved to `reports/` with:

```text
date, close, sma20, sma50, rsi14, daily_return, signal
```

## Learning Outcomes

By working through this project, you will practice:

- Fetching and normalizing financial OHLCV data
- Designing a small SQLite schema for market analytics
- Computing technical indicators with the `ta` library
- Avoiding lookahead bias with next-day backtest positions
- Measuring PnL, Sharpe ratio, drawdown, trades, and win rate
- Building REST endpoints with FastAPI
- Creating interactive dashboards with Streamlit and Plotly
- Generating static charts and CSV reports for analysis
- Structuring a Python analytics project into reusable modules
