# Chapter 10 historical accounting and stock-return inputs

Snapshot date: November 16, 2026. Original universe: the same 50 course companies. The original `acct3343_financial_data_50.csv` is unchanged (248 rows, 76 columns; SHA-256 `b48ba2400340a2447e781abf82333e993aaf45b55ea15ad9138a0e2b196e1813`). No student notebook or completed strategy results are published here.

## Load directly in Colab

```python
import pandas as pd
base = "https://raw.githubusercontent.com/x2003xuy/acct3343-public/main/data/"
panel = pd.read_csv(base + "acct3343_backtest_panel_50.csv")
annual = pd.read_csv(base + "acct3343_annual_stock_returns_50.csv")
monthly = pd.read_csv(base + "acct3343_monthly_adjusted_prices_50.csv")
```

## Files and coverage

| File | Unit of observation | Coverage |
|---|---|---|
| `acct3343_backtest_panel_50.csv` | Company × July formation year | 200 rows: 50 firms × 2022–2025 |
| `acct3343_annual_stock_returns_50.csv` | Company × period type × starting year | 400 rows: calendar years 2022–2025 and July 2022–June 2026 |
| `acct3343_monthly_adjusted_prices_50.csv` | Company × completed calendar month | December 2021–October 2026; 2,950 rows |

There are **three accounting-usable backtest periods**, beginning in 2023, 2024 and 2025. The 2022 coverage rows are retained to show why that candidate period cannot be used: no firm has valid GPA or STA inputs in the selected annual statements. All 50 tickers have endpoint prices in every candidate holding period. Incomplete July 2026–June 2027 and calendar 2026 returns are excluded.

| Formation year | Lag-eligible statement | GPA inputs | STA inputs | Both inputs |
|---|---:|---:|---:|---:|
| 2022 | 37 | 0 | 0 | 0 |
| 2023 | 50 | 34 | 39 | 34 |
| 2024 | 50 | 45 | 50 | 45 |
| 2025 | 50 | 45 | 50 | 45 |

## Timing and measurement

Select the most recent annual statement whose **actual supplied `fiscal_period_end` + six calendar months is strictly before July 1**. Availability is conservatively assumed at end of day; same-day availability is insufficient for July 1 midnight formation. This is an assumption, not an observed filing date. Do not select by fiscal-year label alone. Do not fall back to an older statement when the most recent qualifying statement has missing inputs.

Enter at the first observed US trading-day **close on or after July 1**, and exit at the last close on or before the next June 30. Formation at midnight precedes the entry close. The common calendar uses dates observed for at least 45 of the 50 firms; fixed endpoints are shared across firms. Missing endpoint prices remain missing, never moved to a different date. Cash earns zero between June liquidation and the next July entry. Thus these are approximately 12-month close-to-close positions, with the brief re-entry gap explicitly out of market.

For calendar returns, enter at the last trading close before January 1 and exit at the last trading close on or before December 31. All returns are decimal fractions: 0.20 means 20%. Calculate `exit_adjusted_close / entry_adjusted_close - 1`.

Yahoo Finance daily history was downloaded with Python `yfinance`, `auto_adjust=False`, `repair=False`; the **Adj Close** field supplies dividend- and split-adjusted prices. Monthly values are actual last observed daily adjusted prices, not averages. Only completed calendar months are included. Adjusted price levels can be revised with corporate actions; ratios of prices from one consistent retrieval are the return inputs. Prices use each US listing's USD denomination. No current-price field is used.

## Column definitions

The panel retains the selected original historical columns, with their original names and values, including all financial-statement inputs. **All `current_` snapshot columns are omitted** to avoid accidental historical use. The original file remains the authoritative source for those unrelated fields. `CHAPTER10_COLUMNS.csv` lists every panel column and its definition.

Core accounting fields: `gross_profit` is annual gross profit; `net_income` is annual net income; `operating_cash_flow` is annual cash from operations; `total_assets` is same-period year-end assets. These accounting amounts are USD, not per-share values. GPA = gross profit / total assets. STA = (net income − operating cash flow) / total assets. Both require positive assets and finite inputs; no ratios, ranks, selections or strategy returns are supplied in these public inputs.

Panel timing fields are ISO dates; `formation_year` is the July calendar year. `accounting_age_days` counts days from fiscal period end to formation; `stale_accounting_flag` flags age over 548 days for review, without silently replacing the selected statement. `accounting_available`, `gpa_inputs_valid`, `sta_inputs_valid`, `return_available`, `eligible_gpa`, `eligible_sta` and `eligible_combined` are booleans. `eligible_*` combines the relevant accounting validity and endpoint coverage, not a portfolio selection. In a strategy, select on ex-ante accounting information and investigate any missing subsequent return rather than reweighting surviving observations. `missing_data_reason` is blank when no required-data issue is detected; multiple reasons are separated by semicolons.

The annual return file contains `ticker`, `period_type` (`calendar_year` or `july_june`), `period_year` (calendar year or July starting year), `entry_date`, `exit_date`, `entry_adjusted_close`, `exit_adjusted_close`, `stock_return` and `return_available`. Dates are intended common endpoints even if a price is absent; inspect the coverage flag and price fields.

The monthly file contains `ticker`, `month` (YYYY-MM), `price_date` (actual observed last daily date), and `adjusted_close` (USD adjusted-price level). Missing values are blank. No forward filling, zero-return imputation or fabricated observations is performed.

## Limitations and provenance

This is a classroom dataset, not a point-in-time institutional backtest. Yahoo statements may contain restatements or provider date conventions. The six-month lag cannot remove those problems. The 50-company selection has survivorship and sample-selection bias, and current industry/sector labels may not describe historical classifications. Financial-company GPA/STA comparisons need caution. Corporate actions, spinoffs and dividend methodology can affect Yahoo adjustments. Short returns computed as the negative of adjusted total return are a teaching proxy, not exact fixed-share dividend cash flows or a margin-account simulation. There are only three usable annual strategy observations.

Sources: [original course data](https://github.com/x2003xuy/acct3343-public/blob/main/data/acct3343_financial_data_50.csv), [Yahoo Finance](https://finance.yahoo.com/), and [yfinance API documentation](https://ranaroussi.github.io/yfinance/reference/api/yfinance.download.html). Daily caches, retrieval timestamps, hashes, collection code, independent checks and all completed solutions are retained in the private instructor repository. No textbook excerpts are redistributed.
