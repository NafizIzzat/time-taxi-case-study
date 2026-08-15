# Chicago Taxi Trips Analytics Case Study

## Overview

This project builds a production-style analytics pipeline for the Chicago Taxi Trips public dataset using Google BigQuery, GCP Dataform, and Looker Studio.

The objective is to transform raw taxi trip records into analytical models that answer business questions about tip earners, overworking patterns, holiday impact, and other operational insights.

## Technology Stack

- Data Warehouse: Google BigQuery
- Transformation: GCP Dataform
- Visualisation: Looker Studio

## Source Dataset

Source table:

`bigquery-public-data.chicago_taxi_trips.taxi_trips`

The dataset contains taxi trips reported to the City of Chicago from 2013 onward. To protect privacy, taxi IDs are anonymized and timestamps are rounded.

## Output Dataset

Project:

`time-taxi-case-study`

Dataset:

`us_taxi_trips`

## Dashboard

The Looker Studio dashboard is publicly available here:

[Chicago Taxi Trips Analytics Dashboard](https://datastudio.google.com/s/rsqsezgJqdQ)

## Dataform Models

| Model | Purpose |
|---|---|
| `stg_taxi_trips` | Cleaned staging model with derived date/time fields and data quality flags |
| `int_taxi_trip_sequence` | Sequences taxi trips by taxi ID to calculate breaks between trips |
| `top_tip_earners` | Identifies top 100 taxi IDs by total tips |
| `top_overworkers` | Identifies taxi IDs with frequent long estimated shifts |
| `holiday_trip_impact` | Compares public holiday trip activity against comparable weekday baselines |
| `daily_trip_summary` | Daily trend table for dashboard reporting |
| `payment_type_tip_behavior` | Bonus insight: tipping behavior by payment type |
| `pickup_area_hourly_demand` | Bonus insight: pickup demand by community area and hour |
| `assert_stg_taxi_trips_has_rows` | Data quality assertion to ensure staging table is not empty |

## Assumptions

- The analysis uses the latest available period in the public dataset rather than the current calendar date.
- Taxi IDs represent anonymized taxi medallions, not necessarily individual human drivers.
- Invalid records are flagged in the staging model using `has_data_quality_issue`.
- Final analytical models exclude records with critical data quality issues.
- A new estimated work shift starts after a break of at least 8 hours.
- A long shift is defined as 12 or more hours from first trip start to last trip end within a continuous work session.
- Public holiday analysis uses Google’s public holiday/event reference dataset.
- Holiday impact is measured against non-holiday days with the same day of week.

## Question 1: Top 100 Tip Earners

The `top_tip_earners` model ranks taxi IDs by total tips earned during the staged analysis period.

Supporting metrics include:

- trip count
- active days
- total tips
- average tip per trip
- total fare
- total trip revenue
- tip-to-fare ratio
- trips per active day

This approach identifies taxi IDs that consistently generate higher tip income, not only those with one unusually large tip.

## Question 2: Top 100 Overworkers

The `top_overworkers` model estimates long work sessions using trip sequencing.

Methodology:

1. Trips are ordered by taxi ID and trip start timestamp.
2. The previous trip end timestamp is calculated using `LAG()`.
3. A break of at least 8 hours starts a new estimated shift.
4. Shifts lasting 12 or more hours are classified as long shifts.
5. Taxi IDs are ranked by long shift count, maximum shift duration, and total driving hours.

Because the dataset identifies taxis rather than individual drivers, this should be interpreted as an operational risk signal rather than a definitive driver-level labor record.

## Question 3: Public Holiday Impact

The `holiday_trip_impact` model compares trip activity on US public holidays against non-holiday days with the same weekday.

Metrics include:

- holiday trip count
- comparable weekday trip baseline
- trip count percentage difference
- holiday revenue
- comparable weekday revenue baseline
- revenue percentage difference

This helps identify whether holidays are associated with increased or decreased taxi demand.

## Bonus Insight 1: Payment Type And Tipping

The `payment_type_tip_behavior` model compares tipping behavior across payment types.

Business value:

Payment methods with higher tip ratios may indicate opportunities to improve digital payment experiences, promote card usage, or optimize tip prompts to increase driver earnings.

## Bonus Insight 2: Pickup Area Hourly Demand

The `pickup_area_hourly_demand` model identifies high-demand pickup community areas by hour.

Business value:

This insight can support driver positioning, incentive planning, and operational coverage decisions by showing where and when taxi demand is strongest.

## How To Run

1. Open the Dataform repository.
2. Open the development workspace.
3. Run all actions from Dataform.
4. Confirm that all tables are created in `time-taxi-case-study.us_taxi_trips`.
5. Open the Looker Studio dashboard to view results.

## Limitations

- Public taxi data may not include every trip.
- Some census tract and location fields are suppressed for privacy.
- Taxi IDs are anonymized.
- Shift analysis is estimated from taxi activity and does not prove individual driver working hours.
- Holiday impact analysis is descriptive and does not prove causation.

## Future Improvements

- Add incremental Dataform models for lower-cost refreshes.
- Add more assertions for invalid values and duplicate keys.
- Add geospatial mapping for pickup and dropoff hotspots.
- Add company-level benchmarking.
- Add weather or event data to improve demand analysis.

