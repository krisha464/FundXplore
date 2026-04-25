# 📐 DAX Measures 
> **Project:** FundXplore – Power BI Mutual Funds Analysis Dashboard  
> **Author:** Kanika Dogra  
> **Tool:** Power BI Desktop  
> **DAX Version:** Compatible with Power BI Desktop (March 2024+)  

All measures live in a dedicated **`_Measures`** table (hidden, display-folder organised).  
Reference tables: `Funds`, `NAV_History`, `Returns`, `RiskMetrics`, `Date`.

---

## 📁 Measure Table Structure

```
_Measures/
├── 📂 Fund Overview
│   ├── Total Funds
│   ├── Total Equity Funds
│   ├── Total Debt Funds
│   ├── Total Hybrid Funds
│   └── Total AUM (Crores)
├── 📂 NAV & Growth
│   ├── Latest NAV
│   ├── NAV at Period Start
│   └── NAV Growth % (Point-to-Point)
├── 📂 Returns
│   ├── Avg Return 1Y
│   ├── Avg Return 3Y
│   ├── Avg Return 5Y
│   ├── % Profitable Funds (1Y)
│   └── Avg Excess Return vs Benchmark (1Y)
├── 📂 Risk & Ratios
│   ├── Avg Sharpe Ratio
│   ├── Avg Standard Deviation
│   ├── Best Sharpe Ratio Fund
│   └── Avg Expense Ratio
├── 📂 Investor Considerations
│   ├── Avg Fund Tenure (Years)
│   └── Recommended Category Mix
└── 📂 Time Intelligence
    ├── NAV Growth MoM %
    └── Return YTD
```

---

## 🏦 Fund Overview

### Total Funds

```dax
Total Funds = 
COUNTROWS( Funds )
```

**Usage:** KPI card on Funds Overview page.  
**Format:** `0` (integer). Currently returns 18.

---

### Total Funds by Category

```dax
Total Equity Funds = 
CALCULATE( COUNTROWS( Funds ), Funds[FundCategory] = "Equity" )

Total Debt Funds = 
CALCULATE( COUNTROWS( Funds ), Funds[FundCategory] = "Debt" )

Total Hybrid Funds = 
CALCULATE( COUNTROWS( Funds ), Funds[FundCategory] = "Hybrid" )
```

**Usage:** Donut chart labels or KPI mini-cards on Funds Overview. Also used as denominators in category-level ratio measures.

---

### Total AUM (Crores)

Sum of AUM across all funds currently visible in filter context.

```dax
Total AUM (Crores) = 
SUM( Funds[AUM_Crores] )
```

**Usage:** KPI card on Funds Overview. Slice by `FundCategory` to see AUM distribution across Equity / Debt / Hybrid.  
**Format:** `"₹ " & FORMAT([Total AUM (Crores)], "#,##0") & " Cr"`

---

### Avg AUM per Fund (Crores)

```dax
Avg AUM per Fund (Crores) = 
DIVIDE( [Total AUM (Crores)], [Total Funds], 0 )
```

---

## 📈 NAV & Growth

> These measures operate on the `NAV_History` fact table, filtered by `Date`.  
> **Pre-requisite:** `Date` table must be marked as a Date Table.

---

### Latest NAV

Returns the most recent NAV for each fund in the current filter context.

```dax
Latest NAV = 
CALCULATE(
    LASTNONBLANK( NAV_History[NAV], 1 ),
    ALLEXCEPT( NAV_History, NAV_History[FundID] )
)
```

**Usage:** Table visual on Performance Trend page alongside `Funds[FundName]`. Format: `₹ #,##0.0000`.

---

### NAV at Period Start

Retrieves NAV at the start of the selected date range. Pair with `Latest NAV` to calculate point-to-point growth.

```dax
NAV at Period Start = 
CALCULATE(
    FIRSTNONBLANK( NAV_History[NAV], 1 ),
    ALLEXCEPT( NAV_History, NAV_History[FundID] )
)
```

---

### NAV Growth % (Point-to-Point)

Calculates absolute return from the first to the last NAV within the current date filter (e.g., slicer range).

```dax
NAV Growth % (Point-to-Point) = 
VAR StartNAV = [NAV at Period Start]
VAR EndNAV   = [Latest NAV]
RETURN
    DIVIDE(
        EndNAV - StartNAV,
        StartNAV,
        BLANK()
    )
```

**Usage:** Line chart Y-axis on Performance Trend page with `Date[MonthName]` or `Date[Year]` on X-axis.  
**Format:** `0.00%`  
**Tip:** Add a date range slicer so users can dynamically select 6M / 1Y / 3Y / 5Y windows.

---

## 💹 Returns

### Avg Return — 1Y / 3Y / 5Y

```dax
Avg Return 1Y = 
AVERAGE( Returns[Return_1Y] )

Avg Return 3Y = 
AVERAGE( Returns[Return_3Y] )

Avg Return 5Y = 
AVERAGE( Returns[Return_5Y] )
```

**Usage:** KPI cards on Performance Trend page. Slice by `Funds[FundCategory]` to compare Equity vs Debt vs Hybrid across horizons.  
**Format:** `0.00%`

---

### % Profitable Funds (1Y)

Percentage of funds with a positive 1-year return.

```dax
% Profitable Funds (1Y) = 
VAR ProfitableFunds =
    CALCULATE(
        COUNTROWS( Returns ),
        Returns[Return_1Y] > 0
    )
VAR TotalFunds = COUNTROWS( Returns )
RETURN
    DIVIDE( ProfitableFunds, TotalFunds, 0 )
```

**Usage:** KPI card on Funds Overview — "Profitable Funds" badge.  
**Format:** `0%`  
**Conditional Format:** Green if > 70%, amber if 50–70%, red if < 50%.

---

### Avg Excess Return vs Benchmark (1Y)

Average alpha (outperformance) relative to each fund's own benchmark over 1 year.

```dax
Avg Excess Return vs Benchmark (1Y) = 
AVERAGE( Returns[ExcessReturn_1Y] )
```

> `ExcessReturn_1Y` is a Power Query calculated column: `Return_1Y − BenchmarkReturn_1Y`.  
> A positive value means funds are collectively outperforming their benchmarks.

**Usage:** KPI card. If negative, flag with red indicator — the fund basket is collectively underperforming.  
**Format:** `+0.00%;-0.00%;0.00%` (shows +/- sign explicitly).

---

### Best Fund by 1Y Return (Dynamic Label)

Returns the name of the top-performing fund in the current filter context. Useful for "Best Performer" callout card.

```dax
Best Fund by 1Y Return = 
VAR MaxReturn = MAXX( Returns, Returns[Return_1Y] )
RETURN
    CALCULATE(
        SELECTEDVALUE( Funds[FundName], "Multiple" ),
        Returns[Return_1Y] = MaxReturn
    )
```

**Usage:** Card visual labelled "🏆 Best Performing Fund (1Y)".  
**Format:** Text.

---

### Worst Fund by 1Y Return

```dax
Worst Fund by 1Y Return = 
VAR MinReturn = MINX( Returns, Returns[Return_1Y] )
RETURN
    CALCULATE(
        SELECTEDVALUE( Funds[FundName], "Multiple" ),
        Returns[Return_1Y] = MinReturn
    )
```

**Usage:** Card visual labelled "⚠️ Underperforming Fund (1Y)". Aligns with README insight about Parag Parikh funds.

---

## ⚖️ Risk & Ratios

### Avg Sharpe Ratio

```dax
Avg Sharpe Ratio = 
AVERAGE( RiskMetrics[SharpeRatio] )
```

**Usage:** KPI card on Risk-Return Analysis page. Higher is better.  
**Format:** `0.00`  
**Context:** HSBC Liquid Fund has the best Sharpe Ratio at 2.90.

---

### Best Sharpe Ratio Fund

```dax
Best Sharpe Ratio Fund = 
VAR MaxSharpe = MAXX( RiskMetrics, RiskMetrics[SharpeRatio] )
RETURN
    CALCULATE(
        SELECTEDVALUE( Funds[FundName], "Multiple" ),
        RiskMetrics[SharpeRatio] = MaxSharpe
    )
```

**Usage:** Highlighted callout card: *"Best Risk-Adjusted Fund: HSBC Liquid Fund (Sharpe: 2.90)"*.

---

### Avg Standard Deviation

```dax
Avg Standard Deviation = 
AVERAGE( RiskMetrics[StandardDeviation] )
```

**Usage:** Scatter plot axis (Y = Std Dev, X = 1Y Return) on Risk-Return Analysis page. Each data point is a fund.  
**Format:** `0.00"%"`

---

### Avg Expense Ratio

```dax
Avg Expense Ratio = 
AVERAGE( Funds[ExpenseRatio] )
```

**Usage:** KPI card and bar chart on Investors Consideration page. Lower is better.  
**Format:** `0.00"%"`  
**Insight:** Compare across categories — Equity funds typically have higher expense ratios than Debt/Liquid funds.

---

### Risk % by Fund Type

Used internally in a 100% stacked bar chart — computes each category's share of total standard deviation (risk weight).

```dax
Risk % by Fund Type = 
VAR CategoryStdDev =
    CALCULATE( SUM( RiskMetrics[StandardDeviation] ) )
VAR TotalStdDev =
    CALCULATE(
        SUM( RiskMetrics[StandardDeviation] ),
        ALL( Funds[FundCategory] )
    )
RETURN
    DIVIDE( CategoryStdDev, TotalStdDev, 0 )
```

**Usage:** 100% stacked bar chart with `Funds[FundCategory]` on axis. Confirms Equity funds carry highest aggregate risk.  
**Format:** `0.0%`

---

## 🧑‍💼 Investor Considerations

### Avg Fund Tenure (Years)

```dax
Avg Fund Tenure (Years) = 
AVERAGE( Funds[FundTenure_Years] )
```

**Usage:** KPI card on Investors Consideration page. Slice by `FundCategory` — mature funds generally have longer track records.  
**Format:** `0.0" yrs"`

---

### Recommended Category for Growth Investors

A simple text measure to drive a dynamic recommendation card, based on which category has the highest 5Y return in context.

```dax
Top Category by 5Y Return = 
VAR MaxReturn =
    MAXX(
        SUMMARIZE(
            Funds,
            Funds[FundCategory],
            "_Avg5Y", CALCULATE( AVERAGE( Returns[Return_5Y] ) )
        ),
        [_Avg5Y]
    )
RETURN
    CALCULATE(
        SELECTEDVALUE( Funds[FundCategory], "Equity" ),
        CALCULATE( AVERAGE( Returns[Return_5Y] ) ) = MaxReturn
    )
```

**Usage:** Text card: *"For long-term growth, [Top Category by 5Y Return] funds currently lead."*

---

## 📅 Time Intelligence

> **Pre-requisite:** `Date` table marked as a Date Table, connected to `NAV_History[Date]`.

---

### NAV Growth — Month-over-Month %

How much the (average) NAV has grown vs the previous month.

```dax
NAV Growth MoM % = 
VAR CurrentMonthNAV =
    CALCULATE( AVERAGE( NAV_History[NAV] ) )
VAR PrevMonthNAV =
    CALCULATE(
        AVERAGE( NAV_History[NAV] ),
        DATEADD( Date[Date], -1, MONTH )
    )
RETURN
    DIVIDE(
        CurrentMonthNAV - PrevMonthNAV,
        PrevMonthNAV,
        BLANK()
    )
```

**Usage:** Area/line chart on Performance Trend page. Sliced per fund shows monthly NAV momentum.  
**Format:** `+0.00%;-0.00%;0.00%`

---

### Return YTD (via NAV)

Point-to-point NAV return from the start of the current calendar year to the latest available NAV.

```dax
Return YTD = 
VAR StartOfYear =
    DATE( YEAR( TODAY() ), 1, 1 )
VAR StartNAV =
    CALCULATE(
        FIRSTNONBLANK( NAV_History[NAV], 1 ),
        Date[Date] >= StartOfYear
    )
VAR EndNAV = [Latest NAV]
RETURN
    DIVIDE(
        EndNAV - StartNAV,
        StartNAV,
        BLANK()
    )
```

**Usage:** KPI card for "YTD Return" — gives investors a quick current-year performance snapshot.  
**Format:** `0.00%`

---

### Indian FY Return (April–March)

Returns the NAV-based return for the current Indian financial year (April 1 to March 31).

```dax
Return Current FY = 
VAR FYStart =
    IF(
        MONTH( TODAY() ) >= 4,
        DATE( YEAR( TODAY() ), 4, 1 ),
        DATE( YEAR( TODAY() ) - 1, 4, 1 )
    )
VAR StartNAV =
    CALCULATE(
        FIRSTNONBLANK( NAV_History[NAV], 1 ),
        Date[Date] >= FYStart
    )
VAR EndNAV = [Latest NAV]
RETURN
    DIVIDE(
        EndNAV - StartNAV,
        StartNAV,
        BLANK()
    )
```

**Usage:** Particularly relevant for Indian investor reporting — AMFI and most platforms report returns on FY basis.  
**Format:** `0.00%`

---

## 🔖 Formatting Conventions

| Measure Category | Format String | Example |
|---|---|---|
| Fund counts | `0` | `18` |
| Returns / % | `0.00%` | `15.20%` |
| Returns with sign | `+0.00%;-0.00%;0.00%` | `+5.32%` / `-2.10%` |
| NAV (INR) | `₹ #,##0.0000` | `₹ 1,452.3400` |
| AUM (Crores) | `"₹ " & FORMAT(val, "#,##0") & " Cr"` | `₹ 12,450 Cr` |
| Ratios (Sharpe, Beta) | `0.00` | `2.90` |
| Expense Ratio | `0.00"%"` | `0.85%` |
| Tenure | `0.0" yrs"` | `8.5 yrs` |

---

## 🛡️ Measure Authoring Best Practices

1. **`DIVIDE` always** — never use raw `/`; use `DIVIDE(num, den, BLANK())` for safe division.
2. **`VAR / RETURN` blocks** — define all intermediate calculations as variables for readability and performance. DAX evaluates `VAR` lazily.
3. **`SELECTEDVALUE` for dynamic labels** — use `SELECTEDVALUE(Table[Column], "All")` in text card measures to show the selected fund/category or a fallback.
4. **`ALLEXCEPT` for cross-fund calculations** — when computing fund-level stats in a matrix visual, use `ALLEXCEPT(NAV_History, NAV_History[FundID])` to preserve fund context while removing date context.
5. **Mark the Date table** — all `DATEADD`, `DATESYTD`, `FIRSTNONBLANK`-with-date-filter patterns require the Date table to be properly marked in Power BI (Table Tools → Mark as date table).
6. **Separate `Returns` from `NAV_History`** — never try to compute 1Y / 3Y / 5Y returns via DAX directly from NAV_History unless you have complete daily history. Use pre-computed `Returns` table for KPI cards; use `NAV_History` only for trend line visuals.
7. **Risk-Free Rate assumption** — Sharpe Ratio in source data is typically computed using the RBI repo rate (≈ 6.5% in 2024) as the risk-free rate. Document this assumption in your report tooltip.

---
