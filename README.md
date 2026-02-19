# Lightwieght Telematic Data Pipeline
Status: In Progress (Phase: Ingestion & Transformation)

## 1. Architecture Overview
I have implemented a **Medallion Architecture** using **Delta Live Tables (DLT)** on Azure Databricks. The pipeline is designed to be "production-ready," utilizing **Unity Catalog** for governance and **Volumes** for raw file staging.

## 2. Infrastructure & Data Ingestion
- **Landing Zone**: Created a Managed Volume in the `00_landing` schema. This serves as the decoupled storage layer for raw data.

- **Data Staging**: Successfully uploaded sample Parquet files from GitHub to the landing volume to ensure schema consistency with the project requirements.

- **Auto Loader (CloudFiles)**: Implemented Spark's Auto Loader to ingest data from the volume.

  - **Why:** This provides incremental, streaming ingestion and handles schema evolution automatically.

  - **Checkpointing:** Metadata is stored in a dedicated `_checkpoints` directory within the landing volume to ensure the pipeline is restartable.

## 3. Data Layers (DLT Implementation)
- Bronze Layer (`telematics_test`): * Location: `workspace.01_bronze`
    - Function: Uses Auto Loader to simulate reading the stream of the data and captures raw json data with full fidelity.

- Bronze Layer (`telematics`): * Location: `workspace.01_bronze` 
    - Function: A finalized telematics table using simulated reading parquet files in full fidelity from Volumes. 

## 4. Technical Hurdles & Solutions
- **DBFS Access Denied**: Encountered `DBFS_DISABLED` errors due to modern security policies.

- **Resolution**: Moved from legacy `dbfs:/` paths to Unity Catalog Volumes `(/Volumes/...)`, aligning the project with 2026 Databricks best practices for security and governance.

---

$\quad$

## Foreign Data Source - Infrastructure & Data Ingestion
(Supabase Postgresql)

### External Database Integration (Supabase/PostgreSQL):

- **Relational Source**: Provisioned a PostgreSQL instance on Supabase to act as the primary "Source of Truth".

- **Lakehouse Federation**: Implemented Unity Catalog's Federation to query external tables as native Databricks objects.

- **Connection Pooler**: Utilized the Supabase Transaction Pooler (Port 6543) to ensure stable, serverless-compatible connectivity.

- **Foreign Catalog**: Established a `supabase_data` catalog to provide a governed, single pane of glass for relational data.

- **Ingestion Strategy**: Configured a **Triggered Micro-Batch** approach.

---

## Technical Hurdles & Solutions

### Serverless Ingestion Restrictions (SQLSTATE 42000/0A000):

- **Problem**: Attempted raw JDBC ingestion and `readStream` from the Foreign Catalog, both of which are restricted in Serverless/Shared compute environments for security reasons.

- **Resolution**: Pivoted to Lakehouse Federation for governed access and switched from "Continuous Streaming" to "Triggered Micro-Batches." This bypasses connector restrictions while maintaining a near-real-time data flow.

### Networking & Firewall (PSQLException):

- **Problem**: Initial connection attempts failed because the database host was confused with the API URL, and the Supabase firewall blocked inbound Databricks traffic.

- **Resolution**: Whitelisted Databricks IP ranges in the Supabase Network Restrictions and updated the connection to use the IPv4 Pooler Host and project-specific username.
