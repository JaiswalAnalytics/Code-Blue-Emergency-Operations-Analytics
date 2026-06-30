# DAX Measures — Code Blue Dashboard

All measures are organized in a dedicated `_Measures` table, grouped by category. Each measure includes the DAX formula and an explanation of why it's written that way.

---

## Patient volume

**Total Visits**
```dax
Total Visits = 
COUNTROWS(Fact_Patient_Visits)
```
The base count behind every other measure on the report.

**Emergency Visits**
```dax
Emergency Visits = 
CALCULATE(
    COUNTROWS(Fact_Patient_Visits),
    Fact_Patient_Visits[Admission_Type] = "Emergency"
)
```

---

## Wait time and patient flow

**Avg Wait Time (mins)**
```dax
Avg Wait Time (mins) = 
AVERAGEX(
    Fact_Patient_Visits,
    Fact_Patient_Visits[Wait_Time_Minutes]
)
```
Uses `AVERAGEX` rather than `AVERAGE` so blanks are excluded automatically without distorting the result.

**% Breaching 4Hr Target**
```dax
% Breaching 4Hr Target = 
DIVIDE(
    CALCULATE(
        COUNTROWS(Fact_Patient_Visits),
        Fact_Patient_Visits[Wait_Time_Minutes] > 240
    ),
    [Total Visits],
    0
) * 100
```
240 minutes maps to the NHS 4-hour emergency department standard — the single most-watched ED metric in the UK.

**Avg Length of Stay (Hrs)**
```dax
Avg Length of Stay (Hrs) = 
AVERAGEX(
    Fact_Patient_Visits,
    Fact_Patient_Visits[Length_of_Stay_Hours]
)
```

---

## Patient outcomes

**Mortality Rate %**
```dax
Mortality Rate % = 
DIVIDE(
    CALCULATE(
        COUNTROWS(Fact_Patient_Visits),
        Fact_Patient_Visits[Mortality_Flag] = TRUE()
    ),
    [Total Visits],
    0
) * 100
```

**Readmission Rate %**
```dax
Readmission Rate % = 
DIVIDE(
    CALCULATE(
        COUNTROWS(Fact_Patient_Visits),
        Fact_Patient_Visits[Readmission_30_Days_Flag] = TRUE()
    ),
    [Total Visits],
    0
) * 100
```
NHS target is under 11%. The network average of 17% is the headline finding of the dashboard.

**Avg Satisfaction Score**
```dax
Avg Satisfaction Score = 
AVERAGEX(
    FILTER(
        Fact_Patient_Visits,
        NOT(ISBLANK(Fact_Patient_Visits[Satisfaction_Score]))
    ),
    Fact_Patient_Visits[Satisfaction_Score]
)
```
Explicitly filters out blanks before averaging. 294 visits had no recorded satisfaction score — including them as zero would have understated the true average.

**Complaint Rate %**
```dax
Complaint Rate % = 
DIVIDE(
    CALCULATE(
        COUNTROWS(Fact_Patient_Visits),
        Fact_Patient_Visits[Complaint_Flag] = TRUE()
    ),
    [Total Visits],
    0
) * 100
```

**Critical Patient Rate %**
```dax
Critical Patient Rate % = 
DIVIDE(
    CALCULATE(
        COUNTROWS(Fact_Patient_Visits),
        Fact_Patient_Visits[Severity_Level] >= 4
    ),
    [Total Visits],
    0
) * 100
```
Severity levels 4 and 5 represent high-acuity patients — this measure shows what proportion of total demand is genuinely critical.

**ICU Rate %**
```dax
ICU Rate % = 
DIVIDE(
    CALCULATE(
        COUNTROWS(Fact_Patient_Visits),
        Fact_Patient_Visits[ICU_Required_Flag] = TRUE()
    ),
    [Total Visits],
    0
) * 100
```

**Ambulance Rate %**
```dax
Ambulance Rate % = 
DIVIDE(
    CALCULATE(
        COUNTROWS(Fact_Patient_Visits),
        Fact_Patient_Visits[Ambulance_Arrival_Flag] = TRUE()
    ),
    [Total Visits],
    0
) * 100
```

---

## Capacity and staffing

**Avg Bed Occupancy Rate**
```dax
Avg Bed Occupancy Rate = 
AVERAGE(Fact_Financials[Bed_Occupancy_Rate]) * 100
```
NHS guidance treats occupancy above 85% as a capacity risk threshold.

**Avg Burnout Index**
```dax
Avg Burnout Index = 
AVERAGE(Fact_Staffing[Burnout_Risk_Index])
```
A 0–1 scale measured at shift level. This measure underpins the dashboard's strongest insight — its correlation with patient satisfaction.

**Total Overtime Hours**
```dax
Total Overtime Hours = 
SUM(Fact_Staffing[Overtime_Hours])
```

---

## Financial

**Total Revenue**
```dax
Total Revenue = 
SUM(Fact_Patient_Visits[Revenue_Amount])
```

**Total Treatment Cost**
```dax
Total Treatment Cost = 
SUM(Fact_Patient_Visits[Treatment_Cost])
```

**Revenue vs Cost Ratio**
```dax
Revenue vs Cost Ratio = 
DIVIDE([Total Revenue], [Total Treatment Cost], 0)
```
A ratio below 1.0 indicates the network is spending more on treatment than it recovers in revenue — expected under an NHS funding model, but useful for tracking relative efficiency between hospitals.

---

## Time intelligence

**Visits Last Year**
```dax
Visits Last Year = 
CALCULATE(
    [Total Visits],
    FILTER(
        ALL(Dim_Date),
        Dim_Date[Year] = 2024
    )
)
```
Written against a fixed year rather than `SAMEPERIODLASTYEAR` because the dataset only spans two complete years (2024–2025); relative date functions return unreliable results without an active date filter in context.

**Visits YoY %**
```dax
Visits YoY % = 
VAR CurrentYearVisits =
    CALCULATE(
        [Total Visits],
        FILTER(ALL(Dim_Date), Dim_Date[Year] = 2025)
    )
VAR LastYearVisits =
    CALCULATE(
        [Total Visits],
        FILTER(ALL(Dim_Date), Dim_Date[Year] = 2024)
    )
RETURN
IF(
    ISBLANK(LastYearVisits) || LastYearVisits = 0,
    BLANK(),
    DIVIDE(CurrentYearVisits - LastYearVisits, LastYearVisits) * 100
)
```
Uses `VAR` to calculate each year's total once and reuse it, rather than repeating the `CALCULATE` logic inline. Returns `BLANK()` instead of a misleading number when prior-year data doesn't exist.

---

## Design notes

- All measures live in a dedicated `_Measures` table rather than being scattered across fact tables, keeping the field list organized and matching standard Power BI modeling practice.
- `DIVIDE()` is used instead of the `/` operator throughout to avoid divide-by-zero errors when filters return an empty result set.
- Rate measures consistently use `CALCULATE` + `COUNTROWS` rather than `COUNTIF`-style logic, which keeps them correctly responsive to report-level filters and slicers.
