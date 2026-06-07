# Performance Analysis — Complete DAX Measure Library
**Power BI File:** Performance Analysis  
**Fact Table:** FactActivities  
**Date Table:** DimDate (marked as official Date Table)  
**Total measures:** 54  

---

> **Setup instruction:** Create a hidden table called `_Measures`  
> (New Table → `_Measures = {BLANK()}`) and place every measure below  
> into it. This keeps the field list clean and all measures in one place.

---

## GROUP 1 · Foundational Counts & Totals
*Back the KPI strip on Page 1 and the context KPIs on Page 2.*

```dax
-- ─────────────────────────────────────────────────────────────────
-- [1] # Activities
-- ─────────────────────────────────────────────────────────────────
# Activities =
COUNTROWS( FactActivities )


-- ─────────────────────────────────────────────────────────────────
-- [2] # Outdoor Activities
-- ─────────────────────────────────────────────────────────────────
# Outdoor Activities =
CALCULATE(
    COUNTROWS( FactActivities ),
    DimActivityType[is_outdoor] = TRUE()
)


-- ─────────────────────────────────────────────────────────────────
-- [3] # HIIT Sessions
-- ─────────────────────────────────────────────────────────────────
# HIIT Sessions =
CALCULATE(
    COUNTROWS( FactActivities ),
    DimActivityType[activity_category] = "HIIT"
)


-- ─────────────────────────────────────────────────────────────────
-- [4] Total Distance (km)
-- ─────────────────────────────────────────────────────────────────
Total Distance (km) =
SUM( FactActivities[distance_km] )


-- ─────────────────────────────────────────────────────────────────
-- [5] Total Moving Time (min)
-- ─────────────────────────────────────────────────────────────────
Total Moving Time (min) =
SUM( FactActivities[moving_time_min] )


-- ─────────────────────────────────────────────────────────────────
-- [6] Total Moving Time (hrs)
-- Displayed on the KPI card as "52.6 hours total"
-- ─────────────────────────────────────────────────────────────────
Total Moving Time (hrs) =
DIVIDE( [Total Moving Time (min)], 60 )


-- ─────────────────────────────────────────────────────────────────
-- [7] Total Elevation Gain (m)
-- ─────────────────────────────────────────────────────────────────
Total Elevation Gain (m) =
SUM( FactActivities[total_elevation_gain] )
```

---

## GROUP 2 · Session Averages
*Back the KPI strip, the activity log table, and the weekly summary matrix.*

```dax
-- ─────────────────────────────────────────────────────────────────
-- [8] Avg Distance (km)
-- ─────────────────────────────────────────────────────────────────
Avg Distance (km) =
AVERAGE( FactActivities[distance_km] )


-- ─────────────────────────────────────────────────────────────────
-- [9] Avg Moving Time (min)
-- ─────────────────────────────────────────────────────────────────
Avg Moving Time (min) =
AVERAGE( FactActivities[moving_time_min] )


-- ─────────────────────────────────────────────────────────────────
-- [10] Avg Elevation Gain (m)
-- ─────────────────────────────────────────────────────────────────
Avg Elevation Gain (m) =
AVERAGE( FactActivities[total_elevation_gain] )


-- ─────────────────────────────────────────────────────────────────
-- [11] Avg Heart Rate
-- FILTER excludes the 2 sessions with no HR monitor.
-- ─────────────────────────────────────────────────────────────────
Avg Heart Rate =
AVERAGEX(
    FILTER( FactActivities, NOT ISBLANK( FactActivities[average_heartrate] ) ),
    FactActivities[average_heartrate]
)


-- ─────────────────────────────────────────────────────────────────
-- [12] Max Heart Rate (session peak)
-- ─────────────────────────────────────────────────────────────────
Max Heart Rate =
MAX( FactActivities[max_heartrate] )


-- ─────────────────────────────────────────────────────────────────
-- [13] Avg Pace (min/km)
-- FILTER excludes HIIT (zero distance → infinite pace).
-- ─────────────────────────────────────────────────────────────────
Avg Pace (min/km) =
AVERAGEX(
    FILTER( FactActivities, NOT ISBLANK( FactActivities[pace_min_per_km] ) ),
    FactActivities[pace_min_per_km]
)


-- ─────────────────────────────────────────────────────────────────
-- [14] Avg Cadence (spm)
-- Excludes sessions with no cadence data.
-- ─────────────────────────────────────────────────────────────────
Avg Cadence (spm) =
AVERAGEX(
    FILTER( FactActivities, NOT ISBLANK( FactActivities[average_cadence] ) ),
    FactActivities[average_cadence]
)


-- ─────────────────────────────────────────────────────────────────
-- [15] Avg Suffer Score
-- Excludes the 2 no-HR sessions where suffer_score is blank.
-- ─────────────────────────────────────────────────────────────────
Avg Suffer Score =
AVERAGEX(
    FILTER( FactActivities, NOT ISBLANK( FactActivities[suffer_score] ) ),
    FactActivities[suffer_score]
)


-- ─────────────────────────────────────────────────────────────────
-- [16] Total Suffer Score
-- Weekly/monthly aggregation for the trend line.
-- ─────────────────────────────────────────────────────────────────
Total Suffer Score =
SUMX(
    FILTER( FactActivities, NOT ISBLANK( FactActivities[suffer_score] ) ),
    FactActivities[suffer_score]
)
```

---

## GROUP 3 · Personal Bests & Extremes
*Back the Pace trend annotation, the Cadence target line, and the cardiac KPI strip on Page 3.*

```dax
-- ─────────────────────────────────────────────────────────────────
-- [17] Best Pace (min/km)
-- Minimum pace, filtered to sessions > 3 km to exclude
-- short warm-up / cool-down outliers.
-- ─────────────────────────────────────────────────────────────────
Best Pace (min/km) =
MINX(
    FILTER(
        FactActivities,
        NOT ISBLANK( FactActivities[pace_min_per_km] )
            && FactActivities[distance_km] > 3
    ),
    FactActivities[pace_min_per_km]
)


-- ─────────────────────────────────────────────────────────────────
-- [18] Best Pace Label
-- Returns "9:43 /km" for the annotation on the pace chart.
-- ─────────────────────────────────────────────────────────────────
Best Pace Label =
VAR BestMin = [Best Pace (min/km)]
VAR Mins    = INT( BestMin )
VAR Secs    = ROUND( ( BestMin - Mins ) * 60, 0 )
RETURN
    Mins & ":" & FORMAT( Secs, "00" ) & " /km"


-- ─────────────────────────────────────────────────────────────────
-- [19] Longest Distance (km)
-- ─────────────────────────────────────────────────────────────────
Longest Distance (km) =
MAX( FactActivities[distance_km] )


-- ─────────────────────────────────────────────────────────────────
-- [20] Max Cadence (spm)
-- ─────────────────────────────────────────────────────────────────
Max Cadence (spm) =
MAXX(
    FILTER( FactActivities, NOT ISBLANK( FactActivities[average_cadence] ) ),
    FactActivities[average_cadence]
)


-- ─────────────────────────────────────────────────────────────────
-- [21] Max Suffer Score
-- ─────────────────────────────────────────────────────────────────
Max Suffer Score =
MAXX(
    FILTER( FactActivities, NOT ISBLANK( FactActivities[suffer_score] ) ),
    FactActivities[suffer_score]
)
```

---

## GROUP 4 · Cumulative & Running Totals
*Back the cumulative distance area chart on Page 1.*

```dax
-- ─────────────────────────────────────────────────────────────────
-- [22] Cumulative Distance (km)
-- Running total from the earliest date in the current filter
-- context up to the current date. Works with date slicer.
-- ─────────────────────────────────────────────────────────────────
Cumulative Distance (km) =
CALCULATE(
    [Total Distance (km)],
    FILTER(
        ALL( DimDate ),
        DimDate[Date] <= MAX( DimDate[Date] )
            && DimDate[IsInFactRange] = TRUE()
    )
)


-- ─────────────────────────────────────────────────────────────────
-- [23] Cumulative Moving Time (hrs)
-- Same pattern as above for the time axis.
-- ─────────────────────────────────────────────────────────────────
Cumulative Moving Time (hrs) =
CALCULATE(
    [Total Moving Time (hrs)],
    FILTER(
        ALL( DimDate ),
        DimDate[Date] <= MAX( DimDate[Date] )
    )
)


-- ─────────────────────────────────────────────────────────────────
-- [24] Distance to 300 km Target
-- Powers the reference line annotation on the area chart.
-- ─────────────────────────────────────────────────────────────────
Distance to 300km Target =
300 - CALCULATE( [Total Distance (km)], ALL( DimDate ) )


-- ─────────────────────────────────────────────────────────────────
-- [25] 300 km Target (constant)
-- Used as a constant reference line in the area chart visual.
-- ─────────────────────────────────────────────────────────────────
Target Distance 300km =
300
```

---

## GROUP 5 · HR Zone Distribution
*Back the donut chart and zone table on Page 1.*

```dax
-- ─────────────────────────────────────────────────────────────────
-- [26] # Activities in Zone
-- Generic measure — filters to whichever zone is in context
-- (use in a matrix with DimHRZone[zone_label] as rows).
-- ─────────────────────────────────────────────────────────────────
# Activities in Zone =
CALCULATE( COUNTROWS( FactActivities ) )


-- ─────────────────────────────────────────────────────────────────
-- [27] Zone % of Total
-- Percentage share for the donut chart labels.
-- ─────────────────────────────────────────────────────────────────
Zone % of Total =
DIVIDE(
    CALCULATE( COUNTROWS( FactActivities ) ),
    CALCULATE( COUNTROWS( FactActivities ), ALL( DimHRZone ) )
)


-- ─────────────────────────────────────────────────────────────────
-- [28] Zone 2 % (explicit)
-- Used directly in the KPI card "72.7% Zone 2 dominance".
-- ─────────────────────────────────────────────────────────────────
Zone 2 % =
DIVIDE(
    CALCULATE( COUNTROWS( FactActivities ), DimHRZone[hr_zone] = "Zone 2" ),
    CALCULATE( COUNTROWS( FactActivities ), ALL( DimHRZone ) )
)


-- ─────────────────────────────────────────────────────────────────
-- [29] Zone Distance (km)
-- Distance covered per zone — for the zone table.
-- ─────────────────────────────────────────────────────────────────
Zone Distance (km) =
CALCULATE( SUM( FactActivities[distance_km] ) )
```

---

## GROUP 6 · Week-over-Week Comparisons
*Back the weekly distance bar chart and the Δ Pace column in the weekly summary matrix on Page 3.*

```dax
-- ─────────────────────────────────────────────────────────────────
-- [30] Prev Week Distance (km)
-- OFFSET-style lookup using WeekOfYear as the ordinal.
-- ─────────────────────────────────────────────────────────────────
Prev Week Distance (km) =
VAR CurrentWeek = SELECTEDVALUE( DimDate[WeekOfYear] )
RETURN
    CALCULATE(
        [Total Distance (km)],
        ALL( DimDate ),
        DimDate[WeekOfYear] = CurrentWeek - 1
    )


-- ─────────────────────────────────────────────────────────────────
-- [31] WoW Distance Δ (km)
-- ─────────────────────────────────────────────────────────────────
WoW Distance Δ (km) =
[Total Distance (km)] - [Prev Week Distance (km)]


-- ─────────────────────────────────────────────────────────────────
-- [32] WoW Distance Δ %
-- Used for conditional formatting on the bar chart (green/red).
-- ─────────────────────────────────────────────────────────────────
WoW Distance Δ % =
DIVIDE(
    [WoW Distance Δ (km)],
    [Prev Week Distance (km)]
)


-- ─────────────────────────────────────────────────────────────────
-- [33] Prev Week Avg Pace
-- ─────────────────────────────────────────────────────────────────
Prev Week Avg Pace =
VAR CurrentWeek = SELECTEDVALUE( DimDate[WeekOfYear] )
RETURN
    CALCULATE(
        [Avg Pace (min/km)],
        ALL( DimDate ),
        DimDate[WeekOfYear] = CurrentWeek - 1
    )


-- ─────────────────────────────────────────────────────────────────
-- [34] WoW Pace Δ (min/km)
-- Negative = improvement (faster). Used for the Δ Pace column
-- in the weekly summary matrix with colour rules:
--   negative → green, positive → red.
-- ─────────────────────────────────────────────────────────────────
WoW Pace Δ (min/km) =
[Avg Pace (min/km)] - [Prev Week Avg Pace]


-- ─────────────────────────────────────────────────────────────────
-- [35] Prev Week Avg Cadence
-- ─────────────────────────────────────────────────────────────────
Prev Week Avg Cadence =
VAR CurrentWeek = SELECTEDVALUE( DimDate[WeekOfYear] )
RETURN
    CALCULATE(
        [Avg Cadence (spm)],
        ALL( DimDate ),
        DimDate[WeekOfYear] = CurrentWeek - 1
    )


-- ─────────────────────────────────────────────────────────────────
-- [36] WoW Cadence Δ (spm)
-- Positive = improvement (higher cadence).
-- ─────────────────────────────────────────────────────────────────
WoW Cadence Δ (spm) =
[Avg Cadence (spm)] - [Prev Week Avg Cadence]
```

---

## GROUP 7 · Month-over-Month Comparisons
*Back Page 1 KPI delta badges and Page 3 monthly context.*

```dax
-- ─────────────────────────────────────────────────────────────────
-- [37] Prev Month Distance (km)
-- ─────────────────────────────────────────────────────────────────
Prev Month Distance (km) =
CALCULATE(
    [Total Distance (km)],
    DATEADD( DimDate[Date], -1, MONTH )
)


-- ─────────────────────────────────────────────────────────────────
-- [38] MoM Distance Δ %
-- ─────────────────────────────────────────────────────────────────
MoM Distance Δ % =
DIVIDE(
    [Total Distance (km)] - [Prev Month Distance (km)],
    [Prev Month Distance (km)]
)


-- ─────────────────────────────────────────────────────────────────
-- [39] Prev Month Avg HR
-- ─────────────────────────────────────────────────────────────────
Prev Month Avg HR =
CALCULATE(
    [Avg Heart Rate],
    DATEADD( DimDate[Date], -1, MONTH )
)


-- ─────────────────────────────────────────────────────────────────
-- [40] MoM HR Δ (bpm)
-- Negative = lower avg HR at same volume = cardiac improvement.
-- ─────────────────────────────────────────────────────────────────
MoM HR Δ (bpm) =
[Avg Heart Rate] - [Prev Month Avg HR]
```

---

## GROUP 8 · Cardiac Efficiency Index (CEI)
*Back the CEI combo chart and CEI KPI card on Page 3. This is the model's most analytically important measure.*

```dax
-- ─────────────────────────────────────────────────────────────────
-- [41] Cardiac Efficiency Index (CEI)
-- Formula: (avg_heartrate × moving_time_min) ÷ distance_km
-- = HR·min per km. Lower = more efficient heart.
-- Only calculated for sessions with HR data and distance > 0.
-- ─────────────────────────────────────────────────────────────────
Cardiac Efficiency Index =
VAR HRSessions =
    FILTER(
        FactActivities,
        NOT ISBLANK( FactActivities[average_heartrate] )
            && FactActivities[distance_km] > 0
    )
RETURN
    DIVIDE(
        SUMX( HRSessions, FactActivities[average_heartrate] * FactActivities[moving_time_min] ),
        SUMX( HRSessions, FactActivities[distance_km] )
    )


-- ─────────────────────────────────────────────────────────────────
-- [42] CEI — First Week (baseline)
-- Hardcoded to Week 16 for the Δ% calculation.
-- ─────────────────────────────────────────────────────────────────
CEI Baseline (W16) =
CALCULATE(
    [Cardiac Efficiency Index],
    ALL( DimDate ),
    DimDate[WeekOfYear] = 16
)


-- ─────────────────────────────────────────────────────────────────
-- [43] CEI Δ vs Baseline %
-- Negative = improvement. Used on the KPI card "−3.5%".
-- ─────────────────────────────────────────────────────────────────
CEI Δ vs Baseline % =
DIVIDE(
    [Cardiac Efficiency Index] - [CEI Baseline (W16)],
    [CEI Baseline (W16)]
)


-- ─────────────────────────────────────────────────────────────────
-- [44] CEI — 7-Day Rolling Average
-- Smooths day-to-day noise on the daily CEI line chart.
-- ─────────────────────────────────────────────────────────────────
CEI 7-Day Rolling Avg =
CALCULATE(
    [Cardiac Efficiency Index],
    DATESINPERIOD( DimDate[Date], MAX( DimDate[Date] ), -7, DAY )
)
```

---

## GROUP 9 · Rolling Averages (Trend Smoothing)
*Back the HR trend line, pace trend line, and cadence trend on Page 3.*

```dax
-- ─────────────────────────────────────────────────────────────────
-- [45] Avg HR — 7-Day Rolling
-- Smooths the HR trend line so day-to-day variation
-- doesn't obscure the adaptation signal.
-- ─────────────────────────────────────────────────────────────────
Avg HR 7-Day Rolling =
CALCULATE(
    [Avg Heart Rate],
    DATESINPERIOD( DimDate[Date], MAX( DimDate[Date] ), -7, DAY )
)


-- ─────────────────────────────────────────────────────────────────
-- [46] Avg Pace — 7-Day Rolling
-- ─────────────────────────────────────────────────────────────────
Avg Pace 7-Day Rolling =
CALCULATE(
    [Avg Pace (min/km)],
    DATESINPERIOD( DimDate[Date], MAX( DimDate[Date] ), -7, DAY )
)


-- ─────────────────────────────────────────────────────────────────
-- [47] Avg Cadence — 7-Day Rolling
-- ─────────────────────────────────────────────────────────────────
Avg Cadence 7-Day Rolling =
CALCULATE(
    [Avg Cadence (spm)],
    DATESINPERIOD( DimDate[Date], MAX( DimDate[Date] ), -7, DAY )
)
```

---

## GROUP 10 · Cadence Target Tracking
*Back the cadence chart target band and the cadence KPI card on Page 3.*

```dax
-- ─────────────────────────────────────────────────────────────────
-- [48] Cadence Target Low
-- Lower bound of the 57–58 spm target band.
-- Constant reference line in the cadence chart.
-- ─────────────────────────────────────────────────────────────────
Cadence Target Low =
57


-- ─────────────────────────────────────────────────────────────────
-- [49] Cadence Target High
-- ─────────────────────────────────────────────────────────────────
Cadence Target High =
58


-- ─────────────────────────────────────────────────────────────────
-- [50] Cadence in Target Band %
-- % of outdoor sessions where cadence ≥ 57 spm.
-- ─────────────────────────────────────────────────────────────────
Cadence in Target Band % =
DIVIDE(
    CALCULATE(
        COUNTROWS( FactActivities ),
        FILTER(
            FactActivities,
            NOT ISBLANK( FactActivities[average_cadence] )
                && FactActivities[average_cadence] >= 57
        )
    ),
    CALCULATE(
        COUNTROWS( FactActivities ),
        NOT ISBLANK( FactActivities[average_cadence] )
    )
)


-- ─────────────────────────────────────────────────────────────────
-- [51] Cadence Growth (first → last session)
-- Used in the Page 3 KPI card "+4.2 steps/min".
-- ─────────────────────────────────────────────────────────────────
Cadence Growth (spm) =
VAR FirstCadence =
    CALCULATE(
        [Avg Cadence (spm)],
        TOPN( 1, ALL( DimDate ), DimDate[Date], ASC )
    )
VAR LastCadence =
    CALCULATE(
        [Avg Cadence (spm)],
        TOPN( 1, ALL( DimDate ), DimDate[Date], DESC )
    )
RETURN
    LastCadence - FirstCadence
```

---

## GROUP 11 · Pace Display Formatting
*Back the pace columns in the activity log and weekly matrix, which need mm:ss format.*

```dax
-- ─────────────────────────────────────────────────────────────────
-- [52] Avg Pace Label (mm:ss)
-- Converts decimal min/km to display string "10:36 /km".
-- Use this in card visuals and table columns.
-- Do NOT use for scatter axis — use [Avg Pace (min/km)] there.
-- ─────────────────────────────────────────────────────────────────
Avg Pace Label =
VAR PaceVal = [Avg Pace (min/km)]
VAR Mins    = INT( PaceVal )
VAR Secs    = ROUND( ( PaceVal - Mins ) * 60, 0 )
RETURN
    IF(
        ISBLANK( PaceVal ),
        "—",
        Mins & ":" & FORMAT( Secs, "00" ) & " /km"
    )


-- ─────────────────────────────────────────────────────────────────
-- [53] WoW Pace Δ Label
-- Formats the weekly delta as "+0:19" or "−0:09" with sign.
-- ─────────────────────────────────────────────────────────────────
WoW Pace Δ Label =
VAR Delta   = [WoW Pace Δ (min/km)]
VAR AbsDelta = ABS( Delta )
VAR Mins    = INT( AbsDelta )
VAR Secs    = ROUND( ( AbsDelta - Mins ) * 60, 0 )
VAR Sign    = IF( Delta < 0, "+", IF( Delta > 0, "−", "" ) )
RETURN
    IF(
        ISBLANK( Delta ),
        "—",
        Sign & Mins & ":" & FORMAT( Secs, "00" )
    )
```

---

## GROUP 12 · Scatter Chart Helpers
*Back the Pace vs Distance and HR vs Duration scatter charts on Page 2, and the HR vs Distance scatter on Page 3.*

```dax
-- ─────────────────────────────────────────────────────────────────
-- [54] Outdoor Only Flag
-- Returns 1 for outdoor sessions, BLANK() for HIIT.
-- Use as a visual-level filter (= 1) on all outdoor charts
-- to suppress HIIT zero-distance points from scatter plots.
-- ─────────────────────────────────────────────────────────────────
Outdoor Only Flag =
IF(
    CALCULATE( MAX( DimActivityType[is_outdoor] ) ) = TRUE(),
    1,
    BLANK()
)
```

---

## Summary by Dashboard Page

| Page | Measures used |
|---|---|
| **P1 — Bird's Eye** | [1–7] counts & totals · [8–10] averages · [22–25] cumulative · [26–29] zone distribution · [30–32] WoW distance |
| **P2 — Activity Detail** | [1–7] context KPIs · [8–16] session averages · [17–21] personal bests · [52–54] display labels & filters |
| **P3 — Cardiovascular Progress** | [11–14] HR/pace/cadence averages · [17–18] best pace · [30–36] WoW deltas · [37–40] MoM deltas · [41–44] CEI measures · [45–47] rolling averages · [48–51] cadence target · [52–53] formatting labels |

---

## Conditional Formatting Rules (apply in Power BI)

| Visual | Column/Measure | Rule |
|---|---|---|
| Weekly summary matrix | `WoW Pace Δ Label` | Negative value → green (#70AD47), positive → red (#C00000) |
| Weekly summary matrix | `WoW Cadence Δ (spm)` | Positive → green, negative → red |
| Activity log table | `Avg Heart Rate` | < 110 → blue (#5B9BD5) · 110–128 → green (#70AD47) · ≥ 129 → amber (#FFC000) |
| KPI card | `CEI Δ vs Baseline %` | Negative → green (lower CEI = better) |
| KPI card | `MoM HR Δ (bpm)` | Negative → green (lower HR = better) |
| KPI card | `Zone 2 %` | ≥ 70% → green · 60–70% → amber · < 60% → red |
| Cadence chart | Point colour | `Avg Cadence (spm)` ≥ 57 → accent green · < 57 → muted |

