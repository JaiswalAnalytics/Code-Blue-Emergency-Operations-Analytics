# Code Blue — Emergency Operations & Patient Flow Analytics

An interactive Power BI dashboard analyzing emergency department performance across 11 NHS hospitals, built for the **FP20 Analytics x ZoomCharts Challenge 38**. This is a validated competition entry.

🔗 **[View the live dashboard](https://app.powerbi.com/view?r=eyJrIjoiMGM2Y2M5ZmItNjU3Ny00YzgzLTk0YjYtZjEwM2Q2YmEyYmUxIiwidCI6IjQ2NTRiNmYxLTBlNDctNDU3OS1hOGExLTAyZmU5ZDk0M2M3YiIsImMiOjl9)**

![Code Blue Dashboard](./screenshots/home.png)

## Business problem

NHS emergency departments are under sustained operational pressure — long wait times, rising readmissions, and staff burnout are degrading patient outcomes across the network. Hospital leadership needs a clear, data-driven view of where the system is failing and which interventions to prioritize.

This project analyzes 9,994 patient visits across 11 hospitals over 2024–2025 to answer three questions:
- Where is patient flow breaking down, and for whom?
- Which hospitals and departments are underperforming, and why?
- What should leadership prioritize first?

## Dashboard overview

The report is built across 4 pages:

| Page | Focus |
|---|---|
| **Home** | Headline findings and navigation |
| **Executive Summary** | Network-level KPIs, monthly trends, admission mix, hospital satisfaction comparison |
| **Patient Flow & Outcomes** | Severity distribution, wait time analysis, outcome breakdown by hospital |
| **Operations & Staffing** | Burnout vs satisfaction correlation, staffing stress, bed occupancy, leadership recommendations |

## Key findings

- **Readmission rate of 17%** against the NHS target of under 11% — affecting 6 of 11 hospitals
- **Maximum recorded wait time of 32 hours** (1,967 minutes) — a significant outlier driving network-wide averages
- **Sandwell General Hospital** is the network's critical outlier: 28.9/100 satisfaction, 6% mortality rate, and CRITICAL staffing stress
- **Strong negative correlation between staff burnout and patient satisfaction** — hospitals with low burnout (Nuffield, Spire) score 84–88 on satisfaction, while high-burnout hospitals score below 60
- Private hospitals in the network significantly outperform NHS-funded hospitals on satisfaction and complaint rate, suggesting staffing model and resourcing are key levers

## Recommendations delivered

1. **Immediate workforce intervention at Sandwell General** — staffing stress and burnout are driving measurably worse patient outcomes
2. **Discharge planning review** — the 17% network readmission rate, concentrated in cardiovascular cases, points to gaps in post-discharge care
3. **Scale the operating model of high-performing hospitals** — Nuffield and Spire demonstrate that low burnout and high satisfaction are achievable within the same network

## Data model

Built on a star schema with **11 tables** (3 fact, 7 dimension, 1 reference) and **9 relationships**:

- `Fact_Patient_Visits` — 9,994 rows, core transactional data
- `Fact_Staffing` — shift-level staffing and burnout data
- `Fact_Financials` — monthly hospital-level financial data
- `Dim_Hospital`, `Dim_Date`, `Dim_Patient`, `Dim_Doctor`, `Dim_Department`, `Dim_Diagnosis`, `Dim_Region`

## Technical approach

**Power Query**
- Resolved a text-typed date column (`Full_Date`) silently failing standard type conversion, using a custom M formula (`Date.FromText(Text.From(_))`) to force correct parsing
- Built calculated columns for wait time banding and date hierarchies
- Cleaned and validated 9,994 rows across the core fact table, preserving nulls as meaningful operational signals (e.g. missing triage timestamps) rather than discarding them

**DAX**
- 20 measures covering patient flow (wait time, length of stay, breach rate), outcomes (mortality, readmission, satisfaction), staffing (burnout index, overtime), and financials (revenue, cost ratio)
- Time intelligence measures (YoY comparisons) using `CALCULATE` and `FILTER` against an explicit date table
- See [`DAX_Measures.md`](./DAX_Measures.md) for the full list with explanations

**Visualization**
- ZoomCharts Drill Down PRO visuals (Donut, Combo Bar, Scatter) for interactive cross-filtering and drill-down
- Custom dark-theme landing page with page navigation buttons
- Color-coded insight callouts (red/amber/green) to surface critical findings without requiring the viewer to read every chart

## Tools used

`Power BI Desktop` · `DAX` · `Power Query (M)` · `ZoomCharts Drill Down PRO Visuals` · `Excel`

## Files in this repo

| File | Description |
|---|---|
| `Code_Blue_Dashboard.pbix` | Full Power BI report file |
| `DAX_Measures.md` | All 20 DAX measures with explanations |
| `/data` | Source dataset (Excel) |
| `/screenshots` | Dashboard page screenshots |

---

**Challenge:** [FP20 Analytics x ZoomCharts Challenge 38](https://zoomcharts.com/en/microsoft-power-bi-custom-visuals/challenges/fp20-analytics-challenge-38) — validated entry, June 2026
