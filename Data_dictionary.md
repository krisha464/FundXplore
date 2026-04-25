# 📖 Data Dictionary 
> **Project:** FundXplore – Power BI Mutual Funds Analysis Dashboard  
> **Author:** Kanika Dogra  
> **Tool:** Power BI Desktop (Power Query + DAX)  

---

## 🗂️ Table Overview

| Table Name | Type | Description | Row Grain |
|---|---|---|---|
| `Funds` | Dimension | Master list of all 18 mutual funds analysed | One row per fund |
| `NAV_History` | Fact | Daily Net Asset Value per fund | One row per fund per date |
| `Returns` | Fact | Standardised return horizons (6M, 1Y, 3Y, 5Y) per fund | One row per fund |
| `RiskMetrics` | Fact | Risk statistics per fund (Std Dev, Sharpe, Beta, Alpha) | One row per fund |
| `Date` | Dimension | Standard calendar table | One row per calendar day |

---

## 💼 Table: `Funds`

Master dimension table. One row per mutual fund scheme. 18 rows in the current dataset.

| Column Name | Data Type | Example Values | Description |
|---|---|---|---|
| `FundID` | Text | `F-001`, `F-012` | Unique identifier for each fund scheme. Primary key. |
| `FundName` | Text | `HSBC Liquid Fund`, `Tata India Pharma & Healthcare Fund` | Full scheme name as per AMFI registration. |
| `FundHouse` | Text | `HSBC Mutual Fund`, `Tata Mutual Fund`, `Parag Parikh` | Asset Management Company (AMC) that manages the fund. |
| `FundCategory` | Text | `Equity`, `Debt`, `Hybrid` | Broad SEBI-mandated category. Used for all category-level slicing. |
| `FundSubCategory` | Text | `Large Cap`, `Mid Cap`, `Liquid`, `Arbitrage`, `Balanced Advantage` | SEBI sub-category within the broad category. |
| `BenchmarkIndex` | Text | `Nifty 50 TRI`, `CRISIL Liquid Fund Index`, `Nifty 500 TRI` | The index against which the fund's performance is measured. |
| `LaunchDate` | Date | `2003-06-01`, `2015-12-04` | Date the fund scheme was launched. Used to calculate Fund Tenure. |
| `FundManager` | Text | `Neelotpal Sabarwal`, `Mihir Vora` | Name of the primary fund manager. |
| `MinSIPAmount` | Whole Number | `500`, `1000`, `5000` | Minimum monthly SIP instalment amount (INR). |
| `ExpenseRatio` | Decimal Number | `0.19`, `0.85`, `1.74` | Annual expense ratio as a percentage of AUM (%). Lower is better for investors. |
| `AUM_Crores` | Decimal Number | `12450.30`, `890.45`, `45200.00` | Assets Under Management in Indian Crores (₹). Snapshot as of data pull date. |
| `RiskCategory` | Text | `Low`, `Moderate`, `High`, `Very High` | SEBI-mandated risk-o-meter label for the fund. |
| `IsActive` | True/False | `TRUE`, `FALSE` | Whether the fund is currently open for investment. |
| `FundTenure_Years` | Decimal Number | `8.5`, `21.2`, `4.1` | Calculated column: years since `LaunchDate` to report date. See Power Query note. |

> **Power Query Note — `FundTenure_Years`:**  
> `= Duration.Days(DateTime.LocalNow() - [LaunchDate]) / 365.25`  
> Round to 1 decimal place. Used in Investors Consideration page tenure analysis.

---

## 📈 Table: `NAV_History`

The primary time-series fact table. Drives NAV growth charts and point-in-time return calculations.

| Column Name | Data Type | Example Values | Description |
|---|---|---|---|
| `FundID` | Text | `F-001`, `F-007` | Foreign key → `Funds[FundID]`. |
| `Date` | Date | `2024-01-02`, `2024-06-28` | The NAV observation date. Foreign key → `Date[Date]`. Excludes market holidays. |
| `NAV` | Decimal Number | `1452.34`, `28.91`, `1000.00` | Net Asset Value per unit on that date (INR). 4 decimal places for accuracy. |
| `AUM_Crores_Daily` | Decimal Number | `12380.55` | Daily AUM snapshot (INR Crores). Optional — include if source data provides it. |

> **Note:** NAV is never negative. Liquid fund NAVs typically start at ₹1,000 and grow slowly; equity fund NAVs can be highly variable.

---

## 📊 Table: `Returns`

Pre-computed standardised return horizons per fund. One row per fund. Populated from source data (Groww / MoneyControl) at data refresh.

| Column Name | Data Type | Example Values | Description |
|---|---|---|---|
| `FundID` | Text | `F-001` | Foreign key → `Funds[FundID]`. |
| `Return_6M` | Decimal Number | `0.0842`, `-0.0210`, `0.1130` | Absolute return over the trailing 6 months. Store as decimal (e.g. 8.42% → 0.0842). |
| `Return_1Y` | Decimal Number | `0.1520`, `0.0320`, `0.2840` | Absolute return over the trailing 1 year. |
| `Return_3Y` | Decimal Number | `0.1840`, `0.0710`, `0.3120` | CAGR over the trailing 3 years. |
| `Return_5Y` | Decimal Number | `0.2150`, `0.0980`, `0.4230` | CAGR over the trailing 5 years. |
| `BenchmarkReturn_1Y` | Decimal Number | `0.1380` | Benchmark index return over the trailing 1 year. Used to calculate excess return. |
| `BenchmarkReturn_3Y` | Decimal Number | `0.1610` | Benchmark index CAGR over the trailing 3 years. |
| `AsOfDate` | Date | `2024-12-31` | The date these return figures were computed. Helps identify stale data. |

> **Format Note:** Store all return columns as decimals (not percentages). Apply `0.00%` format string in Power BI visuals. This avoids division errors in DAX.

---

## ⚖️ Table: `RiskMetrics`

Risk statistics per fund. Typically sourced from Groww, MoneyControl, or Value Research. One row per fund.

| Column Name | Data Type | Example Values | Description |
|---|---|---|---|
| `FundID` | Text | `F-001` | Foreign key → `Funds[FundID]`. |
| `StandardDeviation` | Decimal Number | `14.32`, `2.10`, `18.75` | Annualised standard deviation of monthly returns (%). Measures total volatility. Higher = riskier. |
| `SharpeRatio` | Decimal Number | `2.90`, `0.84`, `1.45` | Risk-adjusted return: (Return − Risk-Free Rate) / Std Dev. Higher is better. HSBC Liquid Fund = 2.90. |
| `Beta` | Decimal Number | `0.92`, `1.15`, `0.45` | Fund sensitivity to market movements. Beta > 1 = more volatile than market; < 1 = less volatile. Not applicable to Liquid/Debt funds. |
| `Alpha` | Decimal Number | `2.40`, `-1.20`, `5.10` | Excess return over benchmark adjusted for Beta (%). Positive Alpha = fund outperformed. |
| `SortinoRatio` | Decimal Number | `1.85`, `0.60`, `2.20` | Like Sharpe but penalises only downside volatility. Higher is better. |
| `AsOfDate` | Date | `2024-12-31` | Date these metrics were computed. Usually trailing 3-year window. |

---

## 📅 Table: `Date`

Standard Power BI calendar dimension. Mark as Date Table on `Date[Date]`.

| Column Name | Data Type | Example Values | Description |
|---|---|---|---|
| `Date` | Date | `2020-01-01` | Calendar date. Primary key. |
| `Year` | Whole Number | `2020`, `2024` | Calendar year. |
| `Quarter` | Text | `Q1`, `Q4` | Quarter label. |
| `Month` | Whole Number | `1` – `12` | Month number. Use for sort order. |
| `MonthName` | Text | `January`, `December` | Full month name. |
| `MonthShort` | Text | `Jan`, `Dec` | Abbreviated month name. |
| `FY_Year` | Text | `FY2024-25` | Indian Financial Year label (April–March). Useful for AMFI annual return reporting. |
| `FY_Quarter` | Text | `FY-Q1`, `FY-Q2` | Indian FY quarter (Q1 = Apr–Jun, Q2 = Jul–Sep, etc.). |
| `IsWeekend` | True/False | `TRUE`, `FALSE` | True for Saturday/Sunday. NAV is not published on weekends — filter this out in NAV charts. |
| `IsMarketHoliday` | True/False | `TRUE`, `FALSE` | True for NSE market holidays. Populate from an NSE holiday list to exclude non-trading days from NAV trend lines. |

---

## 🔗 Relationships

```
NAV_History[FundID]    →  Funds[FundID]      (Many-to-One, active)
Returns[FundID]        →  Funds[FundID]      (Many-to-One, active)
RiskMetrics[FundID]    →  Funds[FundID]      (Many-to-One, active)
NAV_History[Date]      →  Date[Date]         (Many-to-One, active)
```

> All filters flow from dimension to fact (single direction). The `Date` table connects only to `NAV_History` because `Returns` and `RiskMetrics` store pre-computed snapshots, not time-series rows.

---

## 🧠 Calculated Columns (Power Query)

| Table | Column Name | Logic |
|---|---|---|
| `Funds` | `FundTenure_Years` | `Duration.Days(DateTime.LocalNow() - [LaunchDate]) / 365.25` |
| `Funds` | `TenureBucket` | `if [FundTenure_Years] < 3 then "New (<3 Yrs)" else if [FundTenure_Years] < 7 then "Established (3–7 Yrs)" else "Mature (7+ Yrs)"` |
| `Returns` | `IsOutperforming_1Y` | `[Return_1Y] > [BenchmarkReturn_1Y]` |
| `Returns` | `ExcessReturn_1Y` | `[Return_1Y] - [BenchmarkReturn_1Y]` |
| `Returns` | `IsProfitable_1Y` | `[Return_1Y] > 0` |

---

## 🏷️ Named Fund Reference (from Dashboard)

| FundName | Category | Notable For |
|---|---|---|
| HSBC Liquid Fund | Debt – Liquid | Best Sharpe Ratio (2.90) |
| Tata India Pharma & Healthcare Fund | Equity – Sectoral | Best long-term performer |
| Parag Parikh Liquid Fund | Debt – Liquid | Identified as underperformer |
| Parag Parikh Arbitrage Fund | Hybrid – Arbitrage | Identified as underperformer |

---
