# VEXAR Fleet Analytics — Power BI Dashboard

> **Data Scientist Intern Assignment | VexarDrive Technologies**  
> Built a complete, production-grade fleet telematics analytics suite in Power BI — covering **driver risk profiling** and **predictive vehicle maintenance** — from raw GPS and IMU sensor data.

---

## Project Overview

VexarDrive Technologies operates a two-wheeler delivery fleet across Bengaluru. This project analyses **one full week of trip data** — 30 drivers, 30 vehicles, 450 trips, and ~12,987 per-minute telemetry rows — to build two interactive dashboards that give fleet managers instant, data-driven decision support.

| Dashboard | Purpose |
|---|---|
| 🧑‍✈️ **Driver Behaviour Dashboard** | Identify and rank risky vs. safe drivers using a composite, exposure-normalized risk score |
| 🔧 **Vehicle Health Status Dashboard** | Flag vehicles showing mechanical wear or irregular sensor signatures using IMU-based vibration analysis |

---

## Tech Stack

![Power BI](https://img.shields.io/badge/Power%20BI-F2C811?style=for-the-badge&logo=powerbi&logoColor=black)
![DAX](https://img.shields.io/badge/DAX-0078D4?style=for-the-badge&logo=microsoft&logoColor=white)
![Power Query](https://img.shields.io/badge/Power%20Query-217346?style=for-the-badge&logo=microsoft-excel&logoColor=white)
![Excel](https://img.shields.io/badge/Data%20Source-Excel-217346?style=for-the-badge&logo=microsoft-excel&logoColor=white)

---

## Repository Contents

```
Dashboard-Power-BI/
│
├── VEXAR_Fleet_Analytics.pbix         # Main Power BI file (both dashboards)
├── VEXAR_Dataset.xlsx                 # Source dataset (Drivers, Vehicles, Trips, Telemetry)
├── VEXAR_Technical_Report.pdf         # Full methodology, justifications & assumptions
├── Dashboard_1_Preview.png            # Screenshot — Driver Behaviour Dashboard
├── Dashboard_2_Preview.png            # Screenshot — Vehicle Health Dashboard
└── README.md
```

---

## Dataset Structure

| Table | Grain | Rows | Key Columns |
|---|---|---|---|
| **Drivers** | One row per driver | 30 | Driver_ID, Age, Gender, License_Experience_Years, Home_Hub |
| **Vehicles** | One row per vehicle | 30 | Vehicle_ID, Make, Model, Manufacture_Year, Odometer_KM, Last_Service_Date |
| **Trips** | One row per trip | 450 | Trip_ID, Driver_ID, Vehicle_ID, Trip_Date, Distance_KM, Avg_Speed_kmph |
| **Telemetry** | One row per minute per trip | ~12,987 | Timestamp, GPS, Speed_kmph, Accel_X/Y/Z_g, Gyro_X/Y/Z_dps |

### Data Model Relationships

```
Telemetry [Trip_ID]   →   Trips [Trip_ID]       (Many-to-One)
Trips [Driver_ID]     →   Drivers [Driver_ID]    (Many-to-One)
Trips [Vehicle_ID]    →   Vehicles [Vehicle_ID]  (Many-to-One)
```

---

## Dashboard 1 — Driver Behaviour

### Visuals
| # | Visual | Insight |
|---|---|---|
| KPI Row | Total Distance · Total Trips · Avg Fleet Score · High-Risk Drivers · Total Drivers | Fleet-level snapshot |
| 1 | **Driver Risk Ranking** (Bar chart, sorted) | Who's riskiest right now |
| 2 | **Harsh Events Breakdown** (Stacked bar) | Braking vs. Acceleration vs. Cornering per driver |
| 3 | **Aggressiveness Matrix** (Scatter plot) | Speed × Harsh Events/100km, bubble = distance, color = experience group |
| 4 | **Daily Fleet Risk Trend** (Line + trend line) | Fatigue drift detection across the week |
| 5 | **GPS Route Map** | Speed density heatmap across Bengaluru hubs |
| Slicers | Hub · Gender · Experience Group · Date Range | Full cross-filter interactivity |

<img width="680" height="392" alt="image" src="https://github.com/user-attachments/assets/b7601f73-c48e-43b9-9a49-7c6e3963a4fd" />

### Risk Score Methodology

**Why not a fixed score?**  
Standard fixed-weight scoring (e.g., `braking_events × 10`) fails when fleet averages are low — everyone ends up clustered near 100 and the ranking becomes meaningless.

**The fix — Exposure Normalization + Min-Max Fleet-Relative Scoring:**

```
Step 1: Detect events per telemetry minute (Calculated Columns in Telemetry table)
  Harsh_Braking_Flag  → Accel_X_g < -0.4
  Harsh_Accel_Flag    → Accel_X_g > +0.4
  Sharp_Corner_Flag   → ABS(Gyro_Z_dps) > 5
  Speeding_Flag       → Speed_kmph > 60

Step 2: Normalize by distance (per 100 km)
  Harsh_Brake_Per_100km = (SUM of flags / Total KM) × 100

Step 3: Min-Max scale each metric across the fleet
  Norm_Score = ((Driver Value − Fleet Min) / (Fleet Max − Fleet Min)) × 100
  → Implemented with MINX(ALL(Drivers), [metric]) and MAXX(ALL(Drivers), [metric])

Step 4: Weighted composite score (higher = riskier)
  Driver_Risk_Score =
    0.30 × Norm_Overspeed_Risk  +
    0.25 × Norm_Brake_Risk      +
    0.20 × Norm_Accel_Risk      +
    0.25 × Norm_Corner_Risk

Step 5: Percentile-based Risk Tier (no hardcoded thresholds)
  High Risk   → Score ≥ P66 (top third of fleet)
  Medium Risk → Score ≥ P33 (middle third)
  Low Risk    → Score < P33 (safest third)
```

**Weight Justification:**

| Component | Weight | Reason |
|---|---|---|
| Overspeed % | 30% | Speeding is the #1 cause of road fatalities in India (MoRTH 2023) |
| Harsh Braking | 25% | Indicates tailgating or poor situational awareness |
| Sharp Cornering | 25% | Leading cause of two-wheeler side-slip crashes |
| Harsh Acceleration | 20% | Aggressive but lower direct crash risk than the above |

---

## Dashboard 2 — Vehicle Health Status

### Visuals
| # | Visual | Insight |
|---|---|---|
| KPI Row | Avg Days Since Service · Total Vehicles · Flagged Vehicles · Avg Vibration Index · Total Fleet KM | Fleet health snapshot |
| 1 | **Vibration Ranking** (Bar chart) | Most vibrating vehicles at top, colored by health flag |
| 2 | **Fleet Health Mix** (Donut chart) | Healthy vs. Watch vs. Service Needed proportion |
| 3 | **Wear Correlation** (Scatter plot) | Odometer vs. Vibration Index — validates higher mileage = higher vibration |
| 4 | **Service Overdue** (Column chart + 60-day threshold line) | Instant red-flag for overdue vehicles |
| 5 | **Daily Degradation** (Line chart, small multiples) | Per-vehicle vibration trend — detects mid-week mechanical deterioration |
| Slicers | Make · Model · Manufacture Year | Filter by vehicle specs |

<img width="678" height="388" alt="image" src="https://github.com/user-attachments/assets/bbe66db6-4912-499b-993d-bb41c5c4e461" />

### Health Score Methodology

```
Vibration Index   = STDEV.S(Accel_Z_g)
  → STDEV preferred over Average: consistent high variability = mechanical wear signal,
    not just a bumpy road (which would affect all vehicles equally that day)

Gyro Noise Index  = STDEV.S(Gyro_X) + STDEV.S(Gyro_Y) + STDEV.S(Gyro_Z)
  → Captures wheel wobble, loose bearings, chassis instability

Days Since Service = DATEDIFF(Last_Service_Date, MAX(Trip_Date), DAY)
  → MAX(Trip_Date) used instead of TODAY() to stay relative to the dataset timeline

Vehicle_Health_Flag:
  Service Needed → Vibration > 0.11  OR  Days Since Service > 60
  Watch          → Vibration > 0.08  OR  Days Since Service > 40
  Healthy        → Below both thresholds
```

**Threshold Calibration:** Thresholds were calibrated against the actual fleet distribution (Vibration range: 0.03–0.16, Avg service gap: 32.7 days) rather than using generic industry values.

---

## ⚙️ How to Open & Use

1. Download `VEXAR_Fleet_Analytics.pbix` from this repository.
2. Open with **Microsoft Power BI Desktop** (free download from [Microsoft](https://powerbi.microsoft.com/en-us/downloads/)).
3. Both dashboards load immediately — no credentials or data refresh required (data is embedded).
4. Use the **slicers** on the right panel to filter by Hub, Driver, Vehicle, Date, or Experience Group.
5. Click any bar or data point to cross-filter all other visuals on the page.

---

## Key Assumptions

| Assumption | Reasoning |
|---|---|
| Harsh event threshold: ±0.4g | Standard urban telematics guideline for two-wheelers; validated against actual Accel_X distribution |
| Speed limit: 60 km/h | Standard urban speed limit for two-wheelers in Bengaluru |
| Gyro_Z > 5 dps for sharp corners | Values above 5 dps form a clear tail above normal operational range in the dataset |
| Min-Max normalization | Ensures scores reflect fleet-relative standing, not an arbitrary hardcoded cap |
| Service threshold: 60 days | Manufacturer-recommended interval; 30-day threshold flagged ~90% of fleet (too aggressive) |
| MAX(Trip_Date) instead of TODAY() | Keeps date calculations accurate relative to the August 2026 dataset timeline |

---

## Future Improvements

- [ ] **Percentile Rank Scoring** (RANKX-based) — more robust than min-max when outliers exist
- [ ] **Predictive Maintenance Model** — XGBoost on historical vibration trends to predict breakdown 7–14 days out
- [ ] **Route Optimization** — K-Means clustering on GPS coordinates to identify inefficient corridors
- [ ] **Usage-Based Insurance (UBI)** — pipe the Driver Risk Score into a premium estimation model
- [ ] **Driver Coaching Flags** — auto-generate personalized training recommendations based on dominant event type per driver
- [ ] **Real-time Streaming** — connect to live IoT telemetry via Azure Event Hubs + Power BI Streaming Dataset

---

## Documentation

The full technical report (`VEXAR_Technical_Report.pdf`) covers:
- Complete DAX formulas with line-by-line reasoning
- Threshold justification for every metric
- Visual design decisions
- Cross-dashboard interaction model
- Extended use-case analysis beyond the two dashboards

---

