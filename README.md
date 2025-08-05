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

<img width="1876" height="753" alt="t855iplx" src="https://github.com/user-attachments/assets/f04330b5-60b1-40ac-8c54-1f3ae1e26f45" />


---

##  Step-by-Step Instructions

### 1.  Clone Repository

git clone https://github.com/your-username/netflix_dbt_project.git
cd netflix_dbt_project


### 2.Upload CSV Files to S3
Create an S3 bucket (e.g., netflixdataset-srujan).

Upload all CSVs into the root folder of the bucket.

### 3. Set Up Snowflake Roles, User, Warehouse, Schema, Stage, and Raw Tables

```sql
-- Switch to admin role
USE ROLE ACCOUNTADMIN;

-- Create role and assign to ACCOUNTADMIN
CREATE ROLE IF NOT EXISTS TRANSFORM;
GRANT ROLE TRANSFORM TO ROLE ACCOUNTADMIN;

-- Create compute warehouse
CREATE WAREHOUSE IF NOT EXISTS COMPUTE_WH;
GRANT OPERATE ON WAREHOUSE COMPUTE_WH TO ROLE TRANSFORM;

-- Create user for dbt
CREATE USER IF NOT EXISTS dbt
  PASSWORD = 'dbtPassword123'
  LOGIN_NAME = 'dbt'
  MUST_CHANGE_PASSWORD = FALSE
  DEFAULT_WAREHOUSE = COMPUTE_WH
  DEFAULT_ROLE = TRANSFORM
  DEFAULT_NAMESPACE = MOVIELENS.RAW
  COMMENT = 'dbt user used for data transformation';
ALTER USER dbt SET TYPE = LEGACY_SERVICE;
GRANT ROLE TRANSFORM TO USER dbt;

-- Create database and schema
CREATE DATABASE IF NOT EXISTS MOVIELENS;
CREATE SCHEMA IF NOT EXISTS MOVIELENS.RAW;

-- Grant access to TRANSFORM role
GRANT ALL ON WAREHOUSE COMPUTE_WH TO ROLE TRANSFORM;
GRANT ALL ON DATABASE MOVIELENS TO ROLE TRANSFORM;
GRANT ALL ON ALL SCHEMAS IN DATABASE MOVIELENS TO ROLE TRANSFORM;
GRANT ALL ON FUTURE SCHEMAS IN DATABASE MOVIELENS TO ROLE TRANSFORM;
GRANT ALL ON ALL TABLES IN SCHEMA MOVIELENS.RAW TO ROLE TRANSFORM;
GRANT ALL ON FUTURE TABLES IN SCHEMA MOVIELENS.RAW TO ROLE TRANSFORM;

-- Create stage to connect to S3
CREATE STAGE netflixstage
  URL='s3://netflixdataset-srujan'
  CREDENTIALS=(
    AWS_KEY_ID='your_aws_key'
    AWS_SECRET_KEY='your_secret_key'
  );

-- Create raw tables and load data
CREATE OR REPLACE TABLE raw_movies (
  movieId INTEGER,
  title STRING,
  genres STRING
);
COPY INTO raw_movies
FROM '@netflixstage/movies.csv'
FILE_FORMAT = (TYPE = 'CSV' SKIP_HEADER = 1 FIELD_OPTIONALLY_ENCLOSED_BY = '"');

CREATE OR REPLACE TABLE raw_ratings (
  userId INTEGER,
  movieId INTEGER,
  rating FLOAT,
  timestamp BIGINT
);
COPY INTO raw_ratings
FROM '@netflixstage/ratings.csv'
FILE_FORMAT = (TYPE = 'CSV' SKIP_HEADER = 1 FIELD_OPTIONALLY_ENCLOSED_BY = '"');

CREATE OR REPLACE TABLE raw_tags (
  userId INTEGER,
  movieId INTEGER,
  tag STRING,
  timestamp BIGINT
);
COPY INTO raw_tags
FROM '@netflixstage/tags.csv'
FILE_FORMAT = (TYPE = 'CSV' SKIP_HEADER = 1 FIELD_OPTIONALLY_ENCLOSED_BY = '"')
ON_ERROR = 'CONTINUE';

CREATE OR REPLACE TABLE raw_genome_scores (
  movieId INTEGER,
  tagId INTEGER,
  relevance FLOAT
);
COPY INTO raw_genome_scores
FROM '@netflixstage/genome-scores.csv'
FILE_FORMAT = (TYPE = 'CSV' SKIP_HEADER = 1 FIELD_OPTIONALLY_ENCLOSED_BY = '"');

CREATE OR REPLACE TABLE raw_genome_tags (
  tagId INTEGER,
  tag STRING
);
COPY INTO raw_genome_tags
FROM '@netflixstage/genome-tags.csv'
FILE_FORMAT = (TYPE = 'CSV' SKIP_HEADER = 1 FIELD_OPTIONALLY_ENCLOSED_BY = '"');

CREATE OR REPLACE TABLE raw_links (
  movieId INTEGER,
  imdbId INTEGER,
  tmdbId INTEGER
);
COPY INTO raw_links
FROM '@netflixstage/links.csv'
FILE_FORMAT = (TYPE = 'CSV' SKIP_HEADER = 1 FIELD_OPTIONALLY_ENCLOSED_BY = '"');


