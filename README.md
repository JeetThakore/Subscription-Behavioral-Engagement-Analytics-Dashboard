# Subscription Behavioral & Engagement Analytics Dashboard

> An end-to-end analytics solution for subscription-based businesses — combining data engineering, behavioral analysis, and executive-ready storytelling to drive retention and revenue.

---

## Project Overview

This project delivers a multi-page interactive dashboard and a suite of analytical findings for a multi-site, multi-company subscription service. The work spans the full analytics lifecycle: raw data profiling, data modeling in Power BI, DAX-powered metric engineering, behavioral cohort analysis, and architecture recommendations for long-term data scalability.

The core business problem being solved: **customer retention**. The dashboard surfaces early warning signals, PI (personal information) capture gaps, and wash behavior patterns — enabling marketing and operations teams to act before members churn.

---

## Key Business Questions Answered

| # | Question | Method |
|---|----------|--------|
| 1 | How complete are member profiles across companies and sites? | Tabular KPI dashboard with conditional formatting |
| 2 | What does the active membership mix look like across wash tiers? | 100% Stacked Area Chart, filterable by company/site |
| 3 | How does early usage behavior predict long-term recharge volume? | Cohort segmentation + bar chart analysis |
| 4 | Do customers who wash 3+ times in their first 30 days retain longer? | Usage bucket aggregation + avg. recharge comparison |
| 5 | Do early-only users (days 1–7) underperform vs. consistent users? | Activity Segment DAX column + visual breakdown |
| 6 | Which members joined 10/31/2025 and are at high risk of churning? | High-risk cohort isolation using multi-condition DAX |

---

## Dashboard Pages

### Page 1 — Member Profile Completion & Active Membership Mix
- Drillable table: **Company → Site → Wash Type**
- KPIs: Total Memberships Sold, % Phone, % Email, % Address, % Name
- Conditional formatting: 🟢 >80% | 🟡 50–79% | 🔴 <50%
- 100% Stacked Area Chart of active memberships by wash type
- Date range filters for New Member Date and Termination Date

<img width="1198" height="693" alt="image" src="https://github.com/user-attachments/assets/bc4fe8eb-c2c6-4615-8746-fb00a6d04fa2" />

### Page 2 — Early Activity & Retention Analytics
- Bar chart: Average Total Recharges by **Usage Bucket** (0 → 10+ washes in first 30 days)
- Bar chart: Average Total Recharges by **Activity Segment** (consistent, early-only, late bloomers, inactive)
- Insight: Peak engagement window identified at **3–4 washes** in first 30 days
- Filterable by company via piano-style slicer interface

<img width="1231" height="689" alt="image" src="https://github.com/user-attachments/assets/e26821a2-5630-4f3f-a631-d067e8e597f9" />

### Page 3 — High-Risk Cohort & Churn Intervention
- Donut chart: High-risk member count broken out by company (cohort: joined 10/31/2025)
- Members flagged: zero usage in Month 0 and Month 1, no termination date recorded
- Recommendations surfaced directly on the page for actionable intervention

<img width="1195" height="691" alt="image" src="https://github.com/user-attachments/assets/6b696fef-05c3-4339-a794-72db03e84763" />

### Page 4 — Architectural Improvement Roadmap
- Proposed solutions for data latency, gaps, and pipeline fragility
- Technology stack recommendations across six domains

<img width="1216" height="693" alt="image" src="https://github.com/user-attachments/assets/1ef76f98-045c-46b2-8c5a-40ab810c3997" />

---

## DAX Measures & Calculated Columns

### `Usage Bucket` — Calculated Column
Segments members by how many times they visited in their first 30 days.
```dax
Usage Bucket = 
SWITCH(
    TRUE(),
    'consolidated'[Usage_First_30_Days] = 0, "0 Washes",
    'consolidated'[Usage_First_30_Days] = 1, "1 Wash",
    'consolidated'[Usage_First_30_Days] = 2, "2 Washes",
    'consolidated'[Usage_First_30_Days] = 3, "3 Washes",
    'consolidated'[Usage_First_30_Days] = 4, "4 Washes",
    'consolidated'[Usage_First_30_Days] >= 5 && 'consolidated'[Usage_First_30_Days] < 10, "5- 9 Washes",
    'consolidated'[Usage_First_30_Days] >= 10, "10+ Washes"
)
```

### `Activity Segment` — Calculated Column
Classifies members into behavioral cohorts based on when in their first 30 days they were active.
```dax
Activity Segment = 
VAR Washed_7_Days = 'consolidated'[Usage_First_7_Days] > 0
VAR Washed_8_to_30_Days = ('consolidated'[Usage_First_30_Days] - 'consolidated'[Usage_First_7_Days]) > 0

RETURN
SWITCH(
    TRUE(),
    Washed_7_Days && Washed_8_to_30_Days,       "Group A: Consistent (Days 1-30)",
    Washed_7_Days && NOT(Washed_8_to_30_Days),  "Group B: Early Only (Days 1-7)",
    NOT(Washed_7_Days) && Washed_8_to_30_Days,  "Group C: Late Bloomers (Days 8-30)",
    "Inactive First 30 Days"
)
```

### `Membership Status` — Calculated Column
Distinguishes active, inactive, and plan-switched members based on termination data.
```dax
Membership Status = 
SWITCH(
    TRUE(),
    ISBLANK('consolidated'[Termination Date]) || 'consolidated'[Termination Date] > DATE(2026, 2, 19), "Active",
    NOT ISBLANK('consolidated'[Termination Date]) && 'consolidated'[Termination Status] <> 2, "Inactive",
    NOT ISBLANK('consolidated'[Termination Date]) && 'consolidated'[Termination Status] = 2, "Plan Switched",
    "Unknown"
)
```

### `High Risk Cohort Count` — Measure
Counts distinct members from a specific cohort who show zero usage over 2 consecutive months and have no termination date.
```dax
High Risk Cohort Count = 
CALCULATE(
    DISTINCTCOUNT('consolidated'[Member Code]),
    'consolidated'[New Member Date] = DATE(2025, 10, 31),
    ISBLANK('consolidated'[Termination Date]),
    'consolidated'[Usage_Month0] = 0,
    'consolidated'[Usage_Month_1] = 0
)
```

---

## Key Findings

- **PI Capture Gap**: Physical address capture sits at ~40% across all companies — a significant data quality opportunity for targeted marketing.
- **Peak Engagement Window**: Members who wash **3–4 times** in their first 30 days show the highest average total recharges (~7.07), marking the optimal window for engagement campaigns.
- **Early Activity ≠ Loyalty**: Members active only in days 1–7 (Group B) average just **5.30 recharges** — lower than even those inactive in their first 30 days (5.73), challenging conventional assumptions about early adopters.
- **High-Risk Cohort**: Of the 28 members who joined on 10/31/2025 and show zero usage over 2 months, **46% belong to a single company** — signaling a company-specific operational or engagement issue requiring immediate attention.

---

## 🏗 Architectural Recommendations

| Area | Problem | Proposed Solution |
|------|---------|-------------------|
| **POS Data Refresh** | Stale or delayed transaction data | Incremental CDC via Kafka + Snowflake unified access + Datadog error logging |
| **Hybrid ETL** | Fragile full-load pipelines | Incremental pipelines (Talend) + CDC integration |
| **Data Gaps / Dispersion** | Missing fields, inconsistent schemas | Metadata monitoring (Apache Atlas) + centralized validation (dbt) |
| **Data Models** | Schema drift over time | Version control (Git) + schema update management (BigQuery) |
| **Variable Revenue Tracking** | Revenue fluctuations not surfaced in real time | PowerBI + Kafka streaming |
| **Niche Pipelines** | One-off pipelines with no reuse | Reusable pipeline templates |

---

## 🛠 Skills & Technologies Used

### Analytics & Visualization
- **Power BI** — Multi-page dashboard design, interactive slicers, drill-down hierarchies, conditional formatting
- **DAX** — Calculated columns, measures, SWITCH logic, time-based filtering, cohort isolation

### Analytical Methods
- Cohort analysis
- Behavioral segmentation
- Churn risk modeling
- KPI benchmarking
- Retention curve analysis

---

## How to Use

1. Open the `.pbix` file in **Power BI Desktop**
2. Use the **New Member Date** and **Termination Date** slicers to filter the time window
3. Use the **Field Name** dropdown to isolate specific sites or companies
4. Use the **Company Name Piano Keys** to isolate specific companies
5. Navigate pages using the tab bar at the bottom
6. Hover over visuals for tooltips; click chart segments to cross-filter

---

## License

This project is for portfolio and demonstration purposes.
