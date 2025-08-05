# Netflix ELT Pipeline with dbt, Snowflake, S3, and Looker Studio
## Project Overview

This project showcases a real-world, end-to-end data analytics pipeline for Netflix-style datasets using the modern data stack.
It demonstrates ELT (Extract → Load → Transform) best practices with Amazon S3, Snowflake, dbt, and Looker Studio to ingest, transform, test, and visualize large-scale datasets.

### Key Features:

- Cloud ingestion from Amazon S3 to Snowflake using COPY INTO

- Role-based access control and Snowflake resource provisioning

- Layered dbt transformations: raw → staging → dim/fact → mart

- Over 10+ dbt tests for data quality validation

- SCD Type 2 tracking with dbt snapshots

- Incremental models for handling large datasets efficiently

- Interactive dashboards in Looker Studio

## Dataset Description

| File Name         | Description                    |
|-------------------|--------------------------------|
| movies.csv        | Movie ID, title, genres        |
| ratings.csv       | User ratings with timestamps   |
| tags.csv          | User-submitted tags            |
| genome_scores.csv | Tag relevance scores           |
| genome_tags.csv   | Tag metadata                  |
| links.csv         | Mapping to IMDb & TMDb IDs     |

All files are stored in Amazon S3 under `raw/` before ingestion into Snowflake.


## Architecture Diagram
Pipeline Flow:

- Extract → Raw CSVs uploaded to Amazon S3

- Load → Snowflake external stage reads data from S3

- Raw Tables → Loaded via COPY INTO commands

- dbt Transformations → Raw → Staging → Fact/Dim → Mart

- Snapshots → SCD Type 2 historical tracking

- Visualization → Looker Studio dashboards

Netflix CSVs
   ↓
Amazon S3 (Raw Data)
   ↓
Snowflake External Stage
   ↓
Raw Tables
   ↓
dbt Raw Models
   ↓
dbt Staging Models
   ↓
Dimensional & Fact Models
   ↓
dbt Snapshots (SCD Type 2)
   ↓
Mart Tables
   ↓
Looker Studio / Power BI / Tableau


