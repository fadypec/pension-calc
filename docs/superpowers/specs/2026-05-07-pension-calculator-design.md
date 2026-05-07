# Pension Growth Calculator — Design Spec

## Overview

A single-file HTML pension growth calculator and visualisation tool. Shows projected pension pot growth over time with interactive sliders to vary contribution rates, salary growth, inflation, expected returns, and retirement age. Displays results as nominal values, inflation-adjusted purchasing power, and projected retirement income.

## User Profile

- 30 years old, UK-based
- Defined contribution pension with Aviva
- Fund: Aviva Pensions International Index Tracking S2 (FTSE World ex UK tracker, 0.60% total charge)
- Base salary: £85,000, total remuneration £90,000
- Salary sacrifice scheme with employer NIC savings passed to pension
- Current monthly contribution: £3,183.96 (employee + employer + NIC savings)
- Current pot balance: £31,614.51
- Receives annual cost of living adjustment (CoLA) equal to October-to-October CPIH, plus discretionary performance raise

## Architecture

Single HTML file (~2,000–2,500 lines), no build step. Chart.js loaded from CDN. All data, logic, styles, and markup in one file. Open in any browser.

## Layout: Tabbed Dashboard (Dark Theme)

### Structure

```
+------------------------------------------------------------------+
|  [Summary Card 1]  [Summary Card 2]  [Summary Card 3]  [Card 4] |
+---------------+--------------------------------------------------+
|               |  [ Growth ]  [ Real Value ]  [ Income ]          |
|   Controls    |--------------------------------------------------|
|   Sidebar     |                                                  |
|   (280px)     |              Chart Area                          |
|               |                                                  |
|               |                                                  |
+---------------+--------------------------------------------------+
|  Data Sources  |  Download Data  |  Disclaimer                   |
+------------------------------------------------------------------+
```

### Summary Cards (always visible, update live)

1. **Pot at Retirement** — nominal value, gradient blue (#1e40af → #3b82f6), shows target age and year
2. **Annual Income** — withdrawal-rate-based income, gradient green (#065f46 → #10b981), shows monthly equivalent
3. **Real Value (Today's £)** — inflation-adjusted pot, gradient amber (#92400e → #f59e0b), labelled "after inflation erosion"
4. **Annuity Estimate** — estimated guaranteed annual income, gradient purple (#581c87 → #a855f7), labelled "guaranteed for life"

### Sidebar Controls

Organised into labelled groups:

**Your Details:**
- Current balance — text input, default £31,614.51
- Monthly contribution — text input, default £3,183.96 (recalculated when contribution % or salary changes)
- Base salary — text input, default £85,000

**Contribution Settings:**
- Employee contribution % — slider 0–50%, default 33%
- Employer contribution % — slider 0–20%, default 7%
- Checkbox: "I receive a cost of living adjustment (CoLA)"
  - When ticked: salary growth slider relabels to "Real raise (above inflation)" and effective salary growth = inflation + real raise, shown as hint text
  - When unticked: salary growth is independent of inflation
- Salary growth / Real raise slider — 0–10%, default 3.5% (unticked) or 1.0% (ticked, representing performance raise above CoLA)

**Assumptions:**
- Expected annual return — slider 0–15%, default 10.3% (fund 5-year annualised average)
- Inflation rate (CPIH) — slider 0–10%, default 2.5% (derived from embedded CPIH history)
- Retirement age — slider 55–75, default 68

**Retirement Income:**
- Withdrawal rate — slider 1–10%, default 4%

**Static Info (below controls):**
- Fund charge (OCF): 0.60% — display only
- Fund name: Aviva Pensions International Index Tracking S2

### Tabs

**Growth Tab:**
- Line chart, X-axis = age (30 → retirement age), Y-axis = £ with k/M suffixes
- Two lines: (1) static contributions (dashed green), (2) salary-linked contributions (solid blue)
- Shaded area fill under each curve
- Hover tooltips showing exact value, age, and year at each point
- Legend at bottom

**Real Value Tab:**
- Same two scenario lines but deflated to today's pounds using cumulative CPIH
- Additional visualisation: nominal line shown faded alongside real line, with shaded gap between labelled "inflation erosion"
- Makes purchasing power loss tangible

**Income Tab:**
- Projected annual and monthly retirement income from withdrawal rule
- Side-by-side comparison with annuity estimate
- Both figures shown in nominal and today's £
- Annuity rate varies by retirement age (lookup table)

### Footer

- "Data Sources" link — opens a panel (not a separate page) listing:
  - ONS CPIH series ID and URL
  - Annuity rate sources and methodology
  - Fund performance data source (Aviva factsheet, dated)
  - Any derived/interpolated values clearly noted
- "Download Data" button — exports embedded CPIH series and annuity rate table as CSV
- Disclaimer: "This is a projection tool for personal use. It is not financial advice. Past performance is not a guide to future performance. The value of investments can go down as well as up."

## Calculation Engine

### Growth Projection

Monthly compounding:

```
net_annual_return = expected_return - fund_charge
net_monthly_return = (1 + net_annual_return)^(1/12) - 1

For each month from current age to retirement:
  pot = pot * (1 + net_monthly_return) + monthly_contribution
  If start of new year:
    If CoLA ticked:
      salary = salary * (1 + inflation_rate + real_raise)
    Else:
      salary = salary * (1 + salary_growth_rate)
    monthly_contribution = recalculate(salary, contribution_pct, employer_pct, nic_savings)
```

### Two Scenarios

1. **Static**: monthly contribution stays at current £3,183.96 forever
2. **Salary-linked**: contribution recalculated each year based on salary growth

### NIC Savings Calculation

Employer NIC rate: 15% (from April 2025) on salary above the Secondary Threshold (£5,000/yr from April 2025).
When salary sacrifice reduces gross salary, employer saves NIC on the sacrificed amount.
NIC saving = sacrifice_amount × 0.15.
100% of this saving is added to pension contribution.

This is recalculated whenever salary or contribution % changes.

### Real Value (Purchasing Power)

```
real_value_at_year_N = nominal_value_at_year_N / (1 + inflation_rate)^N
```

Deflates future nominal values back to today's pounds.

### Retirement Income

**Withdrawal rule:**
```
annual_income = pot_at_retirement * withdrawal_rate
monthly_income = annual_income / 12
```

**Annuity estimate:**
```
annual_annuity_income = pot_at_retirement * annuity_rate_for_age
```

Annuity rate lookup table (approximate current UK market rates for single life, level annuity):

| Retirement Age | Annuity Rate |
|---|---|
| 55 | 4.0% |
| 58 | 4.3% |
| 60 | 4.5% |
| 62 | 4.8% |
| 65 | 5.0% |
| 67 | 5.3% |
| 68 | 5.5% |
| 70 | 6.0% |
| 75 | 7.0% |

Linear interpolation for ages between entries.

Both income figures also shown in today's £ (deflated by cumulative inflation).

## Embedded Data

### CPIH Annual Series (1990–2025)

Source: ONS series L55O (CPIH annual rate).
Embedded as a JavaScript array of {year, rate} objects.
Used to derive the default inflation rate (arithmetic mean of the series ≈ 2.5%).

**Why CPIH over CPI or RPI:**
- CPIH is the ONS's lead measure of inflation since 2017
- Unlike CPI, it includes owner-occupier housing costs — a real expense for retirees
- Unlike RPI, it has no known methodological upward bias (RPI lost National Statistic status in 2013)
- RPI is being aligned to CPIH from 2030
- CPIH gives the most realistic picture of what retirees actually spend

**Why 1990–2025:**
- 35-year window captures multiple regimes: high-inflation 1990s, low/near-zero post-2008, 2022 energy shock
- Includes both pre- and post-Bank of England independence (1997)
- Long enough to smooth out anomalies, recent enough to reflect modern monetary policy

### Annuity Rate Table

Source: current UK bulk annuity market rates for single life, level (non-escalating) annuity. Approximate rates derived from publicly available annuity comparison data.

### Fund Performance Data

Source: Aviva Pensions International Index Tracking factsheet (April 2026, TM10024).
- 5-year cumulative return: 63.13% → ~10.3% annualised (used as default expected return)
- 10-year cumulative return: 215.51% → ~12.2% annualised
- Total fund charge: 0.60%

## Visual Design

### Theme
- Background: #0f172a (main), #1e293b (sidebar, cards)
- Text: #e2e8f0 (primary), #94a3b8 (secondary), #64748b (muted)
- Borders: #334155
- Primary accent: #3b82f6 (blue)
- Secondary accent: #10b981 (green)
- Chart grid: #334155 (solid), dashed for intermediate lines

### Typography
- System font stack: `system-ui, -apple-system, sans-serif`
- Summary card values: 26px bold
- Slider labels: 12px
- Section headers: 13px uppercase, letter-spaced, #94a3b8

### Charts
- Chart.js via CDN
- Area fill under lines with low opacity (~0.15)
- Smooth curves (tension: 0.3)
- Tooltips on hover with formatted values
- Legend below chart

### Responsiveness
- Sidebar collapses to a top drawer on screens < 768px
- Summary cards stack to 2×2 grid on narrow screens
- Charts remain full width

## Technology

- Single HTML file, no build step
- Chart.js loaded from CDN (`https://cdn.jsdelivr.net/npm/chart.js`)
- All calculations in vanilla JavaScript
- All styles in a `<style>` block
- No external dependencies beyond Chart.js CDN

## Not in Scope

- Tax-free lump sum (25% PCLS) modelling
- Lifetime allowance / annual allowance checks
- Multiple fund allocation
- Drawdown modelling post-retirement
- State pension integration
- Login or data persistence (it's a static calculator)
