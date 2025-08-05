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

| File Name         | Description                    |
|-------------------|--------------------------------|
| movies.csv        | Movie ID, title, genres        |
| ratings.csv       | User ratings with timestamps   |
| tags.csv          | User-submitted tags            |
| genome_scores.csv | Tag relevance scores           |
| genome_tags.csv   | Tag metadata                  |
| links.csv         | Mapping to IMDb & TMDb IDs     |

All files are stored in Amazon S3 under `raw/` before ingestion into Snowflake.


