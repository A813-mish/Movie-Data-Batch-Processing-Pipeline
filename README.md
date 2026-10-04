# 🎬 Movie Data Batch Processing Pipeline

## Overview

This project demonstrates an end-to-end **batch data engineering pipeline on AWS** for processing movie data. The pipeline uses **Amazon S3** as the raw data landing zone, **AWS Glue Crawler** for schema discovery and cataloging, **AWS Glue Batch Job** for transformation and data quality checks, and **Amazon Redshift** as the analytical data warehouse.

The solution also includes an event-driven monitoring and notification layer using **Amazon CloudWatch, Amazon EventBridge, and Amazon SNS**. This allows the pipeline to be monitored and enables notifications when important pipeline events occur.

---

## 🏗️ Architecture

### High-Level Architecture

```text
                    ┌──────────────────────┐
                    │      Amazon S3       │
                    │   Raw Movie Data     │
                    └──────────┬───────────┘
                               │
                              Read
                               │
                               ▼
                    ┌──────────────────────┐
                    │    Glue Crawler      │
                    │ Schema Discovery     │
                    └──────────┬───────────┘
                               │
                            Register
                               │
                               ▼
                    ┌──────────────────────┐
                    │     Glue Catalog     │
                    │  Metadata / Schema   │
                    └──────────┬───────────┘
                               │
                              Read
                               │
                               ▼
              ┌────────────────────────────────┐
              │       AWS Glue Batch Job       │
              │                                │
              │ • Data Extraction              │
              │ • Transformation               │
              │ • Data Quality Checks          │
              │ • Business Rules               │
              └───────────────┬────────────────┘
                              │
                        Append / Write
                              │
                              ▼
                    ┌──────────────────────┐
                    │    Amazon Redshift   │
                    │  Analytics Table     │
                    └──────────────────────┘


       Pipeline / Job Events
                │
                ▼
       ┌──────────────────────┐
       │    CloudWatch        │
       │ Logs & Monitoring    │
       └──────────┬───────────┘
                  │
                  ▼
       ┌──────────────────────┐
       │   EventBridge        │
       │ Event Rules/Patterns │
       └──────────┬───────────┘
                  │
                  ▼
       ┌──────────────────────┐
       │       SNS            │
       │ Notifications/Alerts │
       └──────────────────────┘
```

### HLD Design

The architecture follows a simple **Raw → Catalog → Transform → Warehouse → Monitor → Notify** pattern.

1. **Amazon S3 – Raw Layer**
   - Stores the incoming raw movie data.
   - Acts as the durable landing zone for source files.
   - Keeps source data separate from transformed analytical data.

2. **AWS Glue Crawler – Schema Discovery**
   - Scans the raw data stored in S3.
   - Discovers the structure and data types of the source data.
   - Updates the Glue Data Catalog with table metadata.

3. **AWS Glue Data Catalog – Metadata Layer**
   - Maintains metadata about the S3 dataset.
   - Provides schema information to the Glue ETL job.
   - Acts as the central metadata repository for the data pipeline.

4. **AWS Glue Batch Job – ETL Layer**
   - Reads source data using the Glue Catalog.
   - Performs transformations and cleansing.
   - Applies business rules and data quality checks.
   - Produces the final dataset required for analytics.
   - Writes the processed data into Amazon Redshift.

5. **Amazon Redshift – Serving / Analytics Layer**
   - Stores the transformed movie dataset.
   - Provides a structured analytical layer for downstream reporting and SQL analysis.
   - The Glue job appends processed records into the Redshift target table.

6. **Amazon CloudWatch – Monitoring Layer**
   - Captures operational logs and pipeline/job information.
   - Helps monitor Glue job execution and troubleshoot failures.

7. **Amazon EventBridge – Event Routing Layer**
   - Uses event patterns to identify relevant pipeline or job events.
   - Routes matching events to downstream targets.

8. **Amazon SNS – Notification Layer**
   - Sends notifications when configured pipeline events occur.
   - Can be used for success, failure, or operational alerts.

---

## 🔄 End-to-End Data Flow

```text
Raw Movie Files
      │
      ▼
Amazon S3
      │
      ▼
Glue Crawler
      │
      ▼
Glue Data Catalog
      │
      ▼
Glue Batch Job
      │
      ├── Transformation
      │
      ├── Data Cleansing
      │
      ├── Data Quality Checks
      │
      └── Business Rules
      │
      ▼
Amazon Redshift
      │
      ▼
Analytics / SQL Queries


Glue / Pipeline Events
      │
      ▼
CloudWatch
      │
      ▼
EventBridge Rules
      │
      ▼
SNS Notifications
```

---

## 📊 Key Features

### 1. Scalable Raw Data Storage

- Uses **Amazon S3** as the raw data landing zone.
- Separates raw source data from processed analytical data.
- Provides durable and scalable object storage.

### 2. Automated Schema Discovery

- **AWS Glue Crawler** automatically discovers the schema of raw movie data.
- Metadata is stored in the **AWS Glue Data Catalog**.
- Reduces the need to manually define source schemas.

### 3. Batch ETL Processing

The Glue Batch Job performs the main data engineering operations:

- Read data from S3 through the Glue Catalog.
- Clean invalid or incomplete records.
- Transform source columns into analytics-ready fields.
- Apply business rules.
- Perform data quality checks.
- Write the final dataset to Redshift.

### 4. Data Quality Validation

The pipeline includes data quality checks before loading the analytical layer.

Typical checks can include:

- Null or missing required fields.
- Invalid movie identifiers.
- Invalid numeric values.
- Duplicate records.
- Invalid dates.
- Unexpected data types.
- Business-rule validation.

Records that fail validation can be filtered, logged, or handled according to the pipeline's data quality strategy.

### 5. Amazon Redshift Analytics Layer

- Stores the transformed movie dataset.
- Provides a structured warehouse layer.
- Supports analytical SQL queries.
- Separates analytical workloads from the raw S3 layer.

### 6. Event-Driven Monitoring

The pipeline uses AWS monitoring and event services to improve operational visibility:

```text
CloudWatch → EventBridge → SNS
```

This design allows important events to be detected and routed to notification channels.

---

## 🧩 AWS Services Used

| AWS Service | Purpose |
|---|---|
| Amazon S3 | Raw movie data storage |
| AWS Glue Crawler | Schema discovery and metadata registration |
| AWS Glue Data Catalog | Central metadata repository |
| AWS Glue | Batch ETL, transformation and data quality |
| Amazon Redshift | Analytical data warehouse |
| Amazon CloudWatch | Logs and monitoring |
| Amazon EventBridge | Event detection and routing |
| Amazon SNS | Pipeline notifications and alerts |

---

## 🗂️ Data Layers

### Raw Layer

**Amazon S3**

Contains the original movie data received from the source system.

```text
S3
└── raw/
    └── movies/
        └── movie_data.*
```

The raw layer should remain as close as possible to the original source data so that it can be reprocessed when required.

### Metadata Layer

**AWS Glue Data Catalog**

Stores the schema and metadata discovered by the Glue Crawler.

```text
S3 Raw Data
     │
     ▼
Glue Crawler
     │
     ▼
Glue Data Catalog
```

### Processing Layer

**AWS Glue Batch Job**

Responsible for converting raw data into analytics-ready data.

```text
S3
 │
 ▼
Glue Catalog
 │
 ▼
Glue Job
 │
 ├── Clean
 ├── Transform
 ├── Validate
 └── Apply Business Rules
 │
 ▼
Processed Dataset
```

### Analytics Layer

**Amazon Redshift**

Stores the final transformed dataset used for analytical queries.

```text
Glue Batch Job
      │
      ▼
Amazon Redshift
      │
      ▼
Analytics / Reporting
```

---

## 🔧 Transformation and Data Quality Logic

The Glue ETL layer can apply transformations such as:

### Data Cleaning

- Remove or handle null values.
- Standardize text fields.
- Remove duplicate records.
- Handle malformed records.
- Validate required columns.

### Data Transformation

- Convert data types.
- Standardize dates and timestamps.
- Create derived columns.
- Normalize categorical values.
- Apply business-specific transformations.

### Data Quality

Example validation flow:

```text
Incoming Record
      │
      ▼
Required Field Check
      │
      ▼
Data Type Validation
      │
      ▼
Duplicate Check
      │
      ▼
Business Rule Validation
      │
      ├───────────────┐
      │               │
    Valid           Invalid
      │               │
      ▼               ▼
 Redshift       Log / Reject /
                Quality Handling
```

---

## 🚀 Getting Started

### Prerequisites

Before running the project, make sure you have:

- An AWS account.
- Access to Amazon S3.
- AWS Glue permissions.
- Permission to create/use Glue Crawlers and Jobs.
- Access to Amazon Redshift.
- Appropriate IAM roles for Glue and related services.
- Access to CloudWatch, EventBridge, and SNS for monitoring and notifications.

---

## ⚙️ Setup Instructions

### 1. Create the S3 Raw Data Location

Create an S3 bucket or use an existing bucket and upload the raw movie dataset.

Example structure:

```text
s3://<bucket-name>/raw/movies/
```

Keep the raw files in a dedicated raw-data location.

---

### 2. Create the Glue Crawler

Create an AWS Glue Crawler pointing to the S3 raw-data location.

The crawler should:

1. Scan the raw movie data.
2. Discover the schema.
3. Create/update the corresponding table in the Glue Data Catalog.

After the crawler completes, verify the discovered table and columns in the Glue Data Catalog.

---

### 3. Configure the Glue Batch Job

Create a Glue ETL job that:

1. Reads the source dataset through the Glue Catalog.
2. Applies data cleansing.
3. Performs transformations.
4. Runs data quality checks.
5. Writes the processed dataset to Amazon Redshift.

The job should use an IAM role with access to the required S3, Glue, and Redshift resources.

---

### 4. Configure Amazon Redshift

Create the required target database/schema/table in Redshift.

The target table should contain the transformed movie fields required for analytics.

The Glue job then writes/appends the processed records into the Redshift table.

---

### 5. Configure CloudWatch Monitoring

Use CloudWatch to monitor Glue job execution and operational logs.

Useful information to monitor includes:

- Job execution status.
- Job failures.
- Execution duration.
- Error messages.
- Transformation issues.
- Data quality failures.

---

### 6. Configure EventBridge

Create an EventBridge rule using the relevant event pattern for the pipeline.

Example conceptual flow:

```text
Glue / AWS Event
       │
       ▼
EventBridge Rule
       │
       ▼
Matching Event
       │
       ▼
SNS Topic
```

The exact event pattern can be configured based on the events that need to trigger notifications.

---

### 7. Configure SNS Notifications

Create an SNS topic and subscribe the required notification endpoint.

Possible alert scenarios:

- Glue job failure.
- Glue job completion.
- Data quality failure.
- Operational pipeline events.

---

## 📋 Example Pipeline Execution

A typical batch execution follows this sequence:

```text
1. Raw movie data arrives in S3
              ↓
2. Glue Crawler discovers/updates schema
              ↓
3. Glue Catalog stores metadata
              ↓
4. Glue Batch Job reads the dataset
              ↓
5. Transformation and cleansing
              ↓
6. Data quality validation
              ↓
7. Processed data written to Redshift
              ↓
8. CloudWatch captures execution information
              ↓
9. EventBridge evaluates configured events
              ↓
10. SNS sends notification when conditions match
```

---

## 📈 Analytics Capabilities

Once the processed data is available in Redshift, it can be used for analytical queries such as:

- Movie-level performance analysis.
- Movie genre analysis.
- Revenue analysis.
- Rating analysis.
- Release-year analysis.
- Data quality reporting.
- Aggregation by movie attributes.
- Trend analysis.
- Business reporting.

Example:

```sql
SELECT
    genre,
    COUNT(*) AS movie_count
FROM movie_data
GROUP BY genre
ORDER BY movie_count DESC;
```

> Adjust the table and column names according to the actual Redshift schema used in the project.

---

## 🔍 Monitoring and Troubleshooting

### Glue Job Monitoring

Check:

- Job status.
- Execution logs.
- Failed transformations.
- Data quality failures.
- Runtime and resource usage.

### S3 Validation

Verify that:

- Raw files exist in the expected S3 location.
- File formats are supported.
- The Glue Crawler can access the files.

### Glue Catalog Validation

Confirm that:

- The crawler completed successfully.
- The expected table exists.
- Columns and data types are correctly discovered.

### Redshift Validation

Verify that:

- The target table exists.
- The Glue job has write access.
- Records are successfully loaded.
- Data types are compatible with the target schema.

### EventBridge / SNS Validation

Verify that:

- EventBridge rule is enabled.
- Event pattern matches the expected event.
- SNS topic exists.
- SNS subscription is confirmed.
- Notifications are delivered successfully.

---

## 🔐 Security Considerations

### IAM

Follow the principle of least privilege.

Use IAM roles and policies that provide only the permissions required by:

- Glue.
- S3.
- Redshift.
- CloudWatch.
- EventBridge.
- SNS.

### S3 Security

- Keep raw data private.
- Avoid unnecessary public bucket access.
- Use appropriate bucket policies.
- Enable encryption where required.

### Redshift Security

- Restrict database access.
- Use appropriate IAM/database permissions.
- Avoid exposing the warehouse publicly unless required.
- Protect sensitive data stored in analytical tables.

### Data Governance

- Maintain clear data ownership.
- Track data lineage from S3 to Redshift.
- Define retention requirements.
- Protect any sensitive or personally identifiable information.

---

## ⚡ Performance Optimization

### S3

- Use a logical folder/partition structure.
- Prefer efficient file formats such as Parquet for large datasets where appropriate.
- Avoid creating excessive numbers of very small files.

### AWS Glue

- Select appropriate worker capacity.
- Avoid unnecessary transformations.
- Push filtering earlier in the pipeline where practical.
- Reuse the Glue Catalog schema rather than manually defining metadata repeatedly.

### Redshift

- Design the target table according to analytical query patterns.
- Choose appropriate distribution and sorting strategies for larger datasets.
- Monitor query performance.
- Load data in efficient batches.

### Cost Optimization

- Use appropriate Glue worker sizing.
- Avoid unnecessarily frequent crawler/job executions.
- Monitor Redshift usage.
- Keep the raw and processed layers organized to simplify lifecycle management.

---

## 🎯 Use Cases

### Business Intelligence

- Movie performance reporting.
- Revenue and rating analysis.
- Genre-level insights.
- Historical movie trend analysis.

### Data Engineering

- Batch ETL development.
- Data lake to warehouse architecture.
- Schema discovery.
- Data quality validation.
- Data warehouse loading.

### Monitoring and Operations

- Pipeline failure detection.
- Job execution monitoring.
- Automated operational notifications.
- Event-driven alerting.

---

## 🏆 Key Project Highlights

- Built an end-to-end **AWS batch data engineering pipeline**.
- Used **Amazon S3** as the raw data lake/landing layer.
- Automated schema discovery using **AWS Glue Crawler**.
- Used **AWS Glue Data Catalog** for centralized metadata management.
- Implemented transformation and data quality checks using **AWS Glue**.
- Loaded analytics-ready data into **Amazon Redshift**.
- Integrated **CloudWatch** for operational monitoring.
- Used **Amazon EventBridge** for event-based routing.
- Used **Amazon SNS** for automated pipeline notifications.
- Designed a scalable **S3 → Glue → Redshift** data flow with event-driven monitoring.

---

## 🔮 Future Enhancements

Potential improvements include:

- Add AWS Step Functions for multi-step workflow orchestration.
- Add AWS Glue Data Quality rules for stronger automated validation.
- Implement incremental processing instead of processing the complete dataset.
- Introduce partitioned S3 datasets for larger volumes.
- Add a curated data layer between raw S3 data and Redshift.
- Build a BI dashboard using Amazon QuickSight or Power BI.
- Add automated retry and failure recovery mechanisms.
- Add data lineage and governance using AWS Glue/Lake Formation.
- Add CI/CD for Glue ETL code and infrastructure.
- Add automated testing for transformation logic.

---

## 📁 Suggested Repository Structure

```text
movie-data-batch-pipeline/
│
├── README.md
│
├── glue/
│   └── movie_etl_job.py
│
├── sql/
│   └── redshift_schema.sql
│
├── data/
│   └── sample_movie_data.csv
│
└── docs/
    └── architecture.md
```

> Rename files and folders according to the actual structure of your repository.

---

## 🧠 Architecture Summary

```text
                    MOVIE DATA BATCH PIPELINE

   ┌─────────────┐
   │     S3      │
   │  Raw Data   │
   └──────┬──────┘
          │
          ▼
   ┌─────────────┐
   │ Glue Crawler│
   └──────┬──────┘
          │
          ▼
   ┌─────────────┐
   │Glue Catalog │
   └──────┬──────┘
          │
          ▼
   ┌─────────────────────┐
   │   Glue Batch Job    │
   │ Transform + DQ      │
   └──────────┬──────────┘
              │
              ▼
       ┌─────────────┐
       │  Redshift   │
       │   Tables    │
       └─────────────┘

       Monitoring / Alerting

   CloudWatch → EventBridge → SNS
```

---

## 👨‍💻 Project Focus

This project focuses on demonstrating practical **Data Engineering and AWS Cloud** concepts including:

**Amazon S3 • AWS Glue • Glue Crawler • Glue Data Catalog • ETL • Data Quality • Amazon Redshift • CloudWatch • EventBridge • SNS • Batch Processing • Data Warehousing**

