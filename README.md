# Azure Car Sales pipeline

## Overview
This project demonstrates an end-to-end incremental data load pipeline using Azure Data Factory, Azure SQL Database, Azure Data Lake Gen2, and Databricks with Unity Catalog. It follows a medallion architecture (Bronze → Silver → Gold) and implements Slowly Changing Dimension Type 1 (SCD1) logic during the final transformation phase.

## Demo
<b>**Note, this project was created without triggers to reduce costs on a serverless only free account</b>
https://youtu.be/jUlBXRCVP1U

## Architecture Summary
1. Initial Data Ingestion (ADF)
 * Source: SalesData.csv file hosted on GitHub.
 * Tool: Azure Data Factory (Copy Data Activity).
 * Target: Azure SQL Database table: source_cars_data.
2. Incremental Load Check (ADF)
 * Comparison: date_id (aliased as current_load) in source_cars_data is compared with last_load in watermark_table.
 * Filter: Only records with current_load > last_load are considered for further processing.
 * Sink: Incremental data is written to the Bronze layer in Azure Data Lake Gen2 as Parquet files.
3. Data Transformation (Databricks with Unity Catalog)
 * Bronze → Silver:
   * Data is read into a Spark DataFrame.
   * Minor transformations are applied (e.g., renaming, typecasting).
   * Transformed data is saved to the Silver layer in Parquet format.
4. Star Schema Modeling and SCD1 (Databricks)
 * Silver → Gold:
   * Data is modeled into a star schema.
   * Upserts are performed on Delta Tables to handle Slowly Changing Dimensions (Type 1) for incremental loads.
   * Final output includes:
     * 4 Dimension Tables
     * 1 Fact Table
   * All are written to the Gold folder in Azure Data Lake Gen2 using Delta format.

## Data Flow Diagram
<pre>
📂 GitHub (SalesData.csv)
       ↓
🧩 Azure Data Factory
  └── 📥 Copy to Azure SQL (source_cars_data)
       ↓
🧮 Azure SQL Database
  └── 🔍 Compare current_load vs last_load
       ↓ (Only new rows)
📁 Azure Data Lake Gen2 – Bronze (Parquet)
       ↓
💻 Azure Databricks (Unity Catalog)
  └── 🧪 Minor transformations
       ↓
📁 Azure Data Lake Gen2 – Silver (Parquet)
       ↓
💻 Azure Databricks
  └── ⭐ Star schema modeling  
  └── 🔁 SCD1 upserts (Delta Tables)
       ↓
📁 Azure Data Lake Gen2 – Gold  
  ├── 📊 4 Dimension Tables (Delta)  
  └── 📈 1 Fact Table (Delta)

</pre>


## Technologies Used
| Technology| Purpose |
| ---------------- | ------ |
|Azure Data Factory|   Orchestration of data pipeline and incremental load control.<br>🔹 Note: Triggers were not used to reduce costs; the pipeline is manually run to avoid charges on a free-tier account (which uses serverless only).|
| Azure SQL Database           |   Staging area for source data and watermark tracking   |
| Azure Data Lake Storage Gen2    |  Scalable storage for Bronze, Silver, and Gold layers   |
| Azure Databricks |  Data transformation, star schema modeling, and SCD1 upserts   |
|Unity Catalog|	Centralized governance and access control across data layers|
|Apache Spark	|Distributed processing within Databricks|
|Delta Lake	|ACID-compliant storage format for reliable data management|
|GitHub	|Hosting the raw source CSV file (SalesData.csv)|

## Key Features
 * Incremental Load Logic: Ensures only new data is processed by comparing source vs. watermark.
 * Medallion Architecture: Clean separation of raw, cleaned, and business-ready data.
 * Delta Lake: Enables ACID-compliant upserts for SCD1 handling in the Gold layer.
 * Scalable & Modular: Each layer and task is independently manageable and scalable.

## Why I Built It / What I Learned
I built this project to gain hands-on experience with end-to-end data engineering workflows using the Azure ecosystem. My goal was to simulate a real-world pipeline, from ingestion to transformation to business-ready output, while learning how different Azure services integrate in a modern data platform.

I started by using Azure Data Factory (ADF) to ingest raw CSV data from GitHub, load it into Azure SQL, and implement a basic incremental load strategy using a watermarking table. This helped me understand how ADF can be used for both one-time and ongoing ingestion patterns.

From there, I moved the data into a bronze-silver-gold architecture in Azure Data Lake Gen2, learning how to stage data incrementally and persist it in Parquet format. I then used Databricks with Unity Catalog to perform transformations, apply a star schema design, and implement Slowly Changing Dimensions Type 1 (SCD1) using Delta Lake upserts — a key concept in real-world analytics pipelines.

Throughout the project, I developed a stronger understanding of:
 * Orchestrating incremental pipelines
 * Building scalable data lakehouse layers
 * Managing data catalogs and permissions with Unity Catalog
 * Writing efficient PySpark and SQL for transformations
 * The importance of ACID compliance in dimensional models

