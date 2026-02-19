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
