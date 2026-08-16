# Chicago Taxi Trips Analytics Case Study

## Overview

This project builds a production-style analytics pipeline for the Chicago Taxi Trips public dataset using Google BigQuery, GCP Dataform, and Looker Studio.

The objective is to transform raw taxi trip records into business-ready analytical models that answer questions about top tip earners, overworking patterns, public holiday impact, and additional city-level licensing insights.

## Technology Stack

- Data Warehouse: Google BigQuery
- Transformation: GCP Dataform
- Visualisation: Looker Studio
- Version Control: GitHub

## Source Data

Main source table:

`bigquery-public-data.chicago_taxi_trips.taxi_trips`

Holiday reference table:

`bigquery-public-data.ml_datasets.holidays_and_events_for_forecasting`

The Chicago taxi dataset contains trip-level records reported to the City of Chicago from 2013 onward. Taxi IDs are anonymized, timestamps are rounded, and some location fields are suppressed for privacy.

## Repository

https://github.com/NafizIzzat/time-taxi-case-study

## Dashboard

The Looker Studio dashboard will be added after visualisation is completed.

## Data Model Architecture

| Layer | Purpose |
|---|---|
| `sources` | Declares external BigQuery public source tables |
| `staging` | Standardizes source fields and adds data quality flags |
| `core` | Creates trusted business-ready fact and aggregate tables |
| `intermediate` | Builds reusable logic for shift/session analysis |
| `marts` | Produces final business-facing tables for analysis and dashboards |
| `assertions` | Defines automated data quality tests |

## Model Lineage

```text
taxi_trips
→ stg_taxi_trips
→ fct_taxi_trips
→ agg_daily_trips
→ daily_trip_summary