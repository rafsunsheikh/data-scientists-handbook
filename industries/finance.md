# Finance

> **TL;DR** — Finance is the industry where *time, identifier, and price* must all be exactly right or you're producing fiction. Survivorship bias, look-ahead bias, corporate-action mishandling, and timezone bugs are the four ways most "great backtests" silently lie. The data spans microsecond-tick order books to quarterly fundamentals, with strict regulatory and audit requirements layered on top.

## 1. Common data types

| Data type | Where it appears |
|---|---|
| **Time series — equally spaced** | OHLCV bars, intraday minute / second / tick bars |
| **Time series — irregular / event** | Tick data, order events, news, transactions |
| **Categorical (high-cardinality)** | Instruments (ISIN, CUSIP, FIGI), counterparties, accounts, MIC venues, GICS sectors |
| **Numerical** | Prices, volumes, returns, exposures, P&L, balances, ratios |
| **Order book** | Stacked depth at multiple price levels (L1 / L2 / L3) |
| **Text** | News, filings (10-K, 10-Q, 8-K), research, earnings transcripts, social, KYC docs |
| **Graph** | Counterparty networks, beneficial ownership, transaction networks (AML) |
| **Tabular reference** | Corporate actions, calendars, fundamentals |
| **Image** | KYC document scans, check imaging |

## 2. Common sources

| Source | What |
|---|---|
| **Exchanges and feeds (direct or consolidated)** | NYSE/Nasdaq TotalView/ITCH, CME, ICE, Eurex, LSE, TSE; SIP (CTA, UTP) |
| **Vendors** | Bloomberg (BPIPE, EMSX, B-API), Refinitiv (RDP, Elektron), FactSet, S&P Capital IQ, ICE Data |
| **Retail / API-grade** | Polygon, Alpaca, IEX Cloud, Tiingo, Alpha Vantage, Yahoo / Stooq |
| **Macro / fundamental** | FRED (Fed economic), Quandl / Nasdaq Data Link, World Bank, IMF, EDGAR (SEC filings), SimFin |
| **Crypto** | CEX APIs (Binance, Coinbase, Kraken), on-chain (Etherscan, Glassnode, Dune, Allium), DEX (Uniswap v3 subgraph) |
| **Internal** | OMS / EMS (Charles River, FlexTrade, Fidessa), risk systems, books and records, core banking, card networks |
| **Alt-data** | Satellite imagery, credit-card panel data, web-traffic, geo-foot-traffic, transcripts NLP scores |

## 3. Standard schemas, identifiers, and protocols

| Standard | Used for |
|---|---|
| **FIX (Financial Information eXchange)** | Order routing, market data; v4.2 / 4.4 / 5.0 |
| **FAST / SBE / ITCH / OUCH** | Exchange-native binary protocols |
| **ISO 20022** | Payments, securities settlement (replacing MT messages) |
| **SWIFT MT / MX** | Cross-bank messaging |
| **FpML** | OTC derivatives |
| **XBRL** | Regulatory filings (SEC, ESMA) |
| **PCAP / proprietary tick formats** | Captured raw market data |

### Identifiers

- **Instruments**: ISIN (global), CUSIP (US/CA), SEDOL (UK), RIC (Refinitiv), FIGI (OpenFIGI), Ticker + exchange (not unique alone).
- **Counterparties**: LEI (Legal Entity Identifier — global, free at GLEIF).
- **Funds**: ISIN at the share-class level.
- **Venues**: MIC (Market Identifier Code, ISO 10383).
- **Classifications**: GICS, ICB, NAICS, SIC, NACE.
- **Account / transaction**: BIC, IBAN, ACH ABA, card BIN.

Identifier hygiene is half of finance data work. Tickers reuse, change, and overlap across markets (`APPL` vs. `AAPL`, `BBVA.MC` vs. `BBVA.MX`). Persist on ISIN+MIC or FIGI.

## 4. Cleaning particulars

### 4.1 Corporate actions

Splits, dividends, spinoffs, mergers, ticker changes, delistings. A 2-for-1 split shows up in raw data as a 50% "drop" in price.

- Use *split-adjusted* and *dividend-adjusted* prices for return series.
- Use unadjusted prices for actual cash trade simulation.
- Two parallel series are often safest: `close` (unadjusted) and `adj_close` (total-return adjusted).
- Each vendor handles edge cases (special dividends, rights issues) differently.

### 4.2 Survivorship bias

Most "historical" data sources only contain currently-listed instruments. Bankrupt and delisted companies are *missing*. Any backtest on a non-survivorship-free dataset overstates returns. Use point-in-time databases (CRSP, S&P Capital IQ pit, Compustat Snapshot).

### 4.3 Point-in-time data and look-ahead bias

Fundamentals are *revised*. The Q1 2018 earnings release on May 1 had different numbers than the corrected 10-K filed in August. A backtest that uses today's restated values to make trades in May 2018 has a look-ahead bias. Always join on the *as-of* date, never the period-end date.

### 4.4 Timezone and trading-session alignment

- Stocks are tied to an exchange's local time and calendar. NYSE 9:30–16:00 ET; LSE 8:00–16:30 BST/GMT; TSE 9:00–15:00 JST with lunch.
- FX is 24×5 (Sunday 17:00 ET to Friday 17:00 ET).
- Crypto is 24×7.
- Storage: UTC internally; convert only for display and for session-boundary logic.
- Daylight saving creates one-hour shifts twice a year.

### 4.5 Holiday and half-day calendars

Use the right calendar (`pandas_market_calendars`, `exchange_calendars`). NYSE has ~9 full and ~3 half-day closures per year. Trading on a holiday because your data didn't tell you it was closed is a classic bug.

### 4.6 Outliers and bad ticks

Exchange tick data contains erroneous prints (fat-finger, busted trades). Filter:

- Vs. previous trade by a tolerance (5σ).
- Vs. NBBO at the time.
- Cross-checked across venues.

Bursty data (auctions, halts, opening/close) needs special handling.

### 4.7 Foreign exchange and currency

P&L in mixed currencies requires consistent FX. Decide:

- Spot at trade time? End-of-day? Custodian's rate?
- One source for everything (Bloomberg WMR fix), not whatever's handy.

### 4.8 Adjustments for stock-specific events

- Special dividends.
- Rights offerings.
- Spin-offs (the resulting two tickers can each look like 50% drops).
- Currency redenominations.

### 4.9 Tick-data volume

A liquid US equity produces 10⁵–10⁷ trades/day; full L2 order-book updates can reach 10⁹/day across the market. Use columnar storage (Parquet/Arrow), batch by day/symbol, and consider kdb+/q, ClickHouse, DuckDB, or QuestDB for analytical access.

## 5. Standard analyses

### 5.1 Trading and quant

| Analysis | Methods |
|---|---|
| Alpha / factor research | Cross-sectional regressions, Fama-MacBeth, Barra-style risk models |
| Backtesting | Walk-forward, purged k-fold (López de Prado), point-in-time joins |
| Execution / TCA | VWAP, IS, implementation shortfall, market impact (Almgren-Chriss) |
| Microstructure | Lee-Ready trade classification, order-flow imbalance |
| Portfolio construction | Mean-variance, Black-Litterman, risk parity, HRP |
| Risk | VaR (historical / Monte Carlo / parametric), CVaR / ES, stress testing |
| Derivatives | Black-Scholes / SABR / local-vol pricing, Greeks, vol surface |

### 5.2 Banking / payments / credit

| Analysis | Methods |
|---|---|
| Credit scoring (origination, behavioral) | Logistic regression / GBM with WOE binning |
| Fraud detection (card, account takeover, application) | Graph-based, isolation forest, deep + rules hybrid |
| Anti-money-laundering (AML) | Rules + transaction networks + alert triage |
| KYC / sanctions screening | Entity resolution against PEP / sanctions lists |
| Customer LTV, churn, NBA | Survival, GBM, multi-task |

### 5.3 Insurance and asset management adjacencies

Many overlap analyses live alongside finance — see [`insurance.md`](insurance.md).

## 6. Standard visualizations

| Question | Chart |
|---|---|
| Price history | **Candlestick** + volume bars |
| Order book | Stacked depth ("book"); time-and-sales tape |
| Strategy performance | **Equity curve** with **drawdown** below |
| Risk decomposition | **Stacked bar** of factor contributions |
| Correlation across assets | **Heatmap** (sorted by cluster) |
| Distribution of returns | Histogram + ECDF on log-y; QQ vs. normal |
| Vol surface | 3-D mesh or contour over (strike, maturity) |
| Portfolio composition | **Treemap** by sector / geography |
| Trade execution | Mark line + VWAP / arrival price |
| Network of counterparties (AML) | Force-directed with community coloring |

Avoid: bar charts of returns over time (use lines); dual-axis price-and-volume on one panel (use stacked panels).

## 7. Regulation and ethics

Finance is among the most heavily regulated industries on Earth.

| Regulation | Scope |
|---|---|
| **MiFID II / MiFIR (EU)** | Transparency, best execution, transaction reporting |
| **Dodd-Frank (US)** | Post-2008 reforms, OTC swaps reporting (SDR), Volcker |
| **Basel III/IV** | Bank capital, liquidity, leverage |
| **EMIR (EU)** | OTC derivative clearing & reporting |
| **SOX** | Internal controls (audit) for public companies |
| **Reg NMS (US)** | Equity market structure, SIP, ORF, Reg ATS |
| **Reg BI / Form CRS** | Broker-dealer standards |
| **MAR / Market Abuse** | Insider trading, manipulation |
| **AML / BSA / FinCEN** | Suspicious activity reporting |
| **OFAC, EU, UN sanctions** | Screening obligations |
| **PCI DSS** | Card data security |
| **FCRA, ECOA, Reg B** | Credit and fairness (US) |
| **GDPR + national** | Personal data |
| **CCAR, DFAST** | US bank stress tests |
| **IFRS 9 / CECL** | Credit loss accounting (forward-looking) |
| **EU AI Act** (incoming) | Credit scoring as "high-risk" |

### Model risk management

Banks operate under SR 11-7 (US), PRA SS1/23 (UK) — formal model risk management: independent validation, ongoing monitoring, inventory, governance. Data scientists in banks produce *model documentation* as a major work product.

### Fairness in lending

Adverse-action notices (Reg B/FCRA), disparate impact tests, prohibited variables. Explainability (SHAP / counterfactuals) is operationally required, not optional.

## 8. Public datasets

| Dataset | What |
|---|---|
| **FRED** | Macro / rates / employment / inflation |
| **EDGAR** | SEC filings (XBRL parseable) |
| **Yahoo Finance / Stooq** | Daily equity (free, basic) |
| **Polygon / Tiingo (free tiers)** | Better historicals |
| **CRSP** (paid) | Survivorship-free US equity gold standard |
| **Compustat** (paid) | Fundamentals |
| **TAQ** (paid) | NYSE/Nasdaq trades and quotes |
| **LOBSTER** | Sample order-book data |
| **OpenBB** | Open-source aggregation tool |
| **Kaggle Two Sigma, Optiver Trading at the Close** | Competition datasets |
| **NumerAI** | Pre-processed competition data |
| **Etherscan, Allium, Dune** | On-chain crypto |

## 9. Tools and Python ecosystem

| Tool | Use |
|---|---|
| `pandas`, `polars`, `duckdb` | Tabular |
| `vectorbt`, `backtrader`, `zipline-reloaded`, `nautilus_trader` | Backtesting |
| `bt`, `quantstats`, `pyfolio` | Portfolio metrics |
| `pyportfolioopt`, `riskfolio-lib` | Optimization |
| `arch`, `statsmodels` | Time-series, GARCH |
| `QuantLib` | Pricing |
| `mlfinlab` | López de Prado techniques |
| `tia`, `xbbg` | Bloomberg connectivity |
| `ccxt` | Crypto exchange APIs |
| `web3.py`, `multicall`, subgraph clients | On-chain |
| kdb+/q, ClickHouse, QuestDB | Tick-data stores |
| `pandas_market_calendars`, `exchange_calendars` | Trading sessions |

## 10. References

- Hasbrouck, J. *Empirical Market Microstructure*.
- Joshi, M. *The Concepts and Practice of Mathematical Finance*.
- López de Prado, M. *Advances in Financial Machine Learning*.
- López de Prado, M. *Machine Learning for Asset Managers*.
- Tsay, R. *Analysis of Financial Time Series*.
- Hull, J. *Options, Futures, and Other Derivatives*.
- Cartea, Á., Jaimungal, S., Penalva, J. *Algorithmic and High-Frequency Trading*.
- Federal Reserve SR 11-7 — Guidance on Model Risk Management.
