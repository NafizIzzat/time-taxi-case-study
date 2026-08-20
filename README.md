# Chicago Taxi Trips Analytics Case Study

## Overview

This project analyzes the Chicago Taxi Trips public dataset using Google BigQuery, GCP Dataform, and Looker Studio.

The goal is to turn raw taxi trip records into clean, dashboard-ready tables that answer four business questions:

1. Which taxi IDs earned the most tips?
2. Which taxi IDs show repeated long-shift patterns?
3. How do U.S. public holidays affect taxi demand?
4. What other actionable insights can support taxi operations, driver earnings, and city planning?

The final output is an interactive Looker Studio dashboard supported by a layered Dataform transformation pipeline.

## Technology Stack

| Area | Tool |
|---|---|
| Data warehouse | Google BigQuery |
| Transformation | GCP Dataform |
| Visualization | Looker Studio |
| Version control | GitHub |

## Source Data

Main taxi trip source:

```text
bigquery-public-data.chicago_taxi_trips.taxi_trips
```

Holiday reference source:

```text
bigquery-public-data.ml_datasets.holidays_and_events_for_forecasting
```

The Chicago Taxi Trips dataset contains trip-level records reported to the City of Chicago from 2013 onward. Each row represents a completed taxi trip. Some fields are anonymized, rounded, or suppressed for privacy.

Important notes:

- `taxi_id` is anonymized and should be treated as a taxi/medallion/vehicle identifier, not a confirmed individual driver.
- Location data may be partially masked.
- Cash tips may be under-recorded, so tipping analysis is based on recorded tips only.
- The analysis is descriptive and should be interpreted as operational signals, not causal proof.

## Repository

```text
https://github.com/NafizIzzat/time-taxi-case-study
```

## Dashboard

The dashboard is built in Looker Studio and organized into five pages:

| Page | Purpose |
|---|---|
| Executive Summary | Dataset snapshot and taxi ID profile exploration |
| Tip Earners | Top 100 taxi IDs by recorded tips |
| Overworkers | Taxi IDs with repeated long estimated shifts |
| Holiday Impact | Public holiday demand compared with normal same-day-of-week activity |
| Additional Insights | Payment behavior and pickup-area opportunity signals |

## Data Model Architecture

The project follows a medallion-style structure.

| Dataform Layer | Medallion Equivalent | Purpose |
|---|---|---|
| `sources` | Bronze reference | Declares external BigQuery public tables |
| `staging` | Bronze / early Silver | Selects useful fields, standardizes dates, and flags data quality issues |
| `core` | Silver | Creates the clean trusted trip-level fact table |
| `intermediate` | Silver logic layer | Builds reusable calculations such as daily activity and estimated shifts |
| `marts` | Gold | Creates final dashboard-facing tables for each business question |
| `assertions` | Quality checks | Tests important assumptions and model outputs |

## Model Flow

```text
sources
  taxi_trips
  holidays_and_events_for_forecasting

staging
  stg_taxi_trips

core
  fct_taxi_trips

intermediate
  int_daily_trips
  int_taxi_trips_ordered
  int_taxi_shifts

marts
  taxi_profile_summary
  top_tip_earners
  top_overworkers
  holiday_trip_impact
  payment_tip_behavior
  area_yield_utilization
```

## Key Models

### `stg_taxi_trips`

Standardizes the raw taxi trip data and adds derived fields such as trip date, trip year, trip month, trip hour, and data quality flags.

### `fct_taxi_trips`

Keeps valid trip records from staging. This is the main trusted trip-level fact table used by downstream models.

### `int_daily_trips`

Aggregates clean trip records to daily level. This supports holiday baseline comparisons.

### `int_taxi_trips_ordered`

Orders trips by taxi ID and trip timestamp. This allows the project to estimate breaks between trips.

### `int_taxi_shifts`

Groups ordered trips into estimated taxi shifts. A new shift starts when the gap between trips is at least 8 hours. Shifts lasting 12+ hours are flagged as long shifts, while estimated shifts above 16 hours are excluded from the overworker mart as outliers.

### `top_tip_earners`

Identifies the top 100 taxi IDs by total recorded tips in the latest 3 months available in the dataset.

### `top_overworkers`

Identifies the top 100 taxi IDs with the most long estimated shifts in the latest 3 months available in the dataset.

### `holiday_trip_impact`

Compares taxi activity on U.S. public holidays against normal non-holiday activity on the same day of week.

### `payment_tip_behavior`

Compares recorded tipping behavior by payment type.

### `area_yield_utilization`

Compares pickup community areas by trip demand and revenue efficiency.

### `taxi_profile_summary`

Supports the Executive Summary page by combining taxi ID, company, payment type, date, location, trip volume, mileage, and revenue fields into a dashboard-friendly summary table.

## Dashboard Questions

### 1. Top Tip Earners

This page answers:

```text
Which taxi IDs earned the most recorded tips in the latest 3 months?
```

Key measures:

- Total tips
- Total trips
- Average tip per trip
- Tip-to-fare ratio
- Average trip distance

Business value:

> Helps identify taxi IDs with strong recorded tipping performance and understand whether high tips are driven by trip volume, trip distance, or tip quality.

### 2. Overworker Signals

This page answers:

```text
Which taxi IDs show repeated long estimated shifts?
```

Key measures:

- Long shift count
- Average long shift hours
- Average actual driving hours per shift
- Long shift ratio
- Revenue during shifts

Business value:

> Helps identify operating patterns that may require workload monitoring, safety review, or fleet management attention.

Important caveat:

> Taxi IDs represent vehicles or medallions, so this is an operating-intensity signal, not definitive proof of individual driver work hours.

### 3. Holiday Impact

This page answers:

```text
Do U.S. public holidays increase or decrease taxi demand?
```

Key measures:

- Holiday trip count
- Normal same-day-of-week baseline
- Trip count percentage difference
- Active taxi count difference
- Revenue percentage difference

Business value:

> Helps with holiday staffing, supply planning, and expectation-setting around reduced or increased demand.

### 4. Additional Insights

This page provides two additional actionable insights.

#### Payment Behavior

Compares fare volume and recorded tip rates by payment method.

Business value:

> Digital payment methods show stronger recorded tip rates than cash, suggesting digital-first payment options may support higher driver earnings. Cash tips may be under-recorded, so this should be treated as recorded tipping behavior only.

#### Pickup Area Opportunity

Compares pickup community areas by trips per active taxi and revenue per mile.

Business value:

> Helps identify pickup areas with stronger demand and better revenue efficiency. These areas may be good candidates for taxi staging, dispatch incentives, or further supply monitoring.

## Key Assumptions

- Latest 3 months means the latest 3 months available in the dataset, not the current calendar date.
- Long shifts are estimated from taxi trip activity, not driver clock-in records.
- A break is the gap between one trip ending and the next trip starting for the same taxi ID.
- A new shift starts when the break is at least 8 hours.
- Shifts of 12+ hours are flagged as long shifts.
- Estimated shifts above 16 hours are treated as outliers for overworker reporting.
- Public holiday impact compares holiday dates against non-holiday dates on the same day of week.
- Revenue is based on `trip_total`, which includes fare, tips, tolls, and extras.

## How To Run

1. Open the Dataform repository in GCP.
2. Compile the workspace.
3. Run all models from sources through marts.
4. Review assertion results.
5. Refresh Looker Studio data sources.
6. Validate dashboard filters and calculated fields.

## Looker Studio Notes

Most dashboard marts keep daily or shift-level grain so Looker Studio date filters can recalculate metrics correctly.

Common calculated fields used in Looker Studio:

```text
Avg Tip / Trip = SUM(total_tips) / SUM(trip_count)

Tip / Fare Ratio = SUM(total_tips) / SUM(total_fare)

Trips per Active Taxi = SUM(trip_count) / SUM(active_taxi_count)

Revenue per Mile = SUM(total_trip_revenue) / SUM(total_trip_miles)

Long Shift Ratio = SUM(long_shift_flag) / COUNT(shift_number)
```
