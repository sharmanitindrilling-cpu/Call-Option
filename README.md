# Options Research Desk — Streamlit MVP

This is a **research-only**, explainable multi-agent prototype for comparing manually entered Fidelity **long call** contracts. **It never connects to Fidelity, places orders, or gives a guaranteed prediction.** Use one contract per row and ensure all quotes have the same timestamp.

## Agents
1. **Technical:** Yahoo Finance (via unofficial `yfinance`) adjusted daily close, MA20/MA50, RSI, momentum and historical realized volatility.
2. **Scenarios:** transparent lognormal scenario model for expiration break-even, estimated conditional P(profit), modeled expected P&L and shock scenarios.
3. **Catalyst:** dated user-pasted notes from Investing.com Pro. **Not automatically verified.** User explicitly flags event risk.
4. **Risk committee:** risk-budget, liquidity, quote freshness, event, expiration, consistency guards. Can return NO TRADE. Candidate rank is a heuristic for review, **not investment confidence**.

## Run locally
```bash
python -m venv .venv
# Windows: .venv\Scripts\activate
# macOS/Linux: source .venv/bin/activate
pip install -r requirements.txt
streamlit run app.py
```

## Deploy from GitHub
1. Create a **private GitHub repository** and upload all files, including `agents/core.py`, `.streamlit/config.toml` and `requirements.txt`.
2. Open https://share.streamlit.io and create an app from that repository.
3. Set entrypoint to `app.py` and deploy. Keep access restricted if quotes or research notes are sensitive.
4. Do **not** commit brokerage passwords, API keys, or `secrets.toml`. Use Streamlit Cloud secrets for future APIs.

## How to enter Fidelity quotes
In Fidelity's options chain: select a ticker, call, expiration and strike. Copy **expiration YYYY-MM-DD**, **strike**, **premium per share** (prefer intended limit price), **bid**, **ask**, and optionally **IV as decimal** and **call delta**. Replace the prefilled synthetic rows. Record the precise Fidelity quote timestamp with timezone. Check the chart's last daily-bar date and use only comparable snapshots. Data are not consolidated real-time market quotes.

## Important limits
- **Investing.com Pro subscription ≠ API authorization.** There is no Investing.com scraper, credential reuse or session-cookie extraction in this app. Integrate only a documented licensed endpoint or manually input your own notes.
- `yfinance` is unofficial, intended for personal research, may fail or be delayed, and requires reviewing Yahoo's terms. Do not assume commercial/redistribution rights.
- Intraday prices and manually entered option quotes may not align with the last adjusted daily close: the app conservatively **blocks** review when last daily market bar is not today's calendar date. **On weekends/non-trading days this intentionally yields NO TRADE.** Improve with market calendars and timestamped live market data.
- The lognormal scenario probability is conditional on assumed constant volatility and drift (default zero); it is **not calibrated** and is especially fragile around earnings or volatility spikes. It's not an actual option-pricing quote or probability of success.
- IV and delta are displayed and validated, but *not used* as model forecasts. A next iteration should compare implied versus realized vol and use calibrated distributions.
- The 'score' ranks eligible quotes for **research**, not a confidence percentage. No autonomous execution. Long calls can lose 100% of premium; exercising can demand stock purchase capital.
- At least 65 daily closes needed; the app stops if history cannot be loaded. No hidden sample data fallback.

## Production roadmap
Replace the personal research feed with a licensed market data API, add quote synchronization and US exchange trading calendar, independently sourced dated catalysts, walk-forward testing and calibration (Brier scores, reliability plots), volatility smile/event gap modeling, transaction costs, persistent evidence/audit history, and optional LLM narrative restricted to evidence with citations. Do not permit LLM to override risk gates.
