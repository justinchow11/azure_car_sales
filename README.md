# Azure Car Sales pipeline

## Overview
This project demonstrates an end-to-end incremental data load pipeline using Azure Data Factory, Azure SQL Database, Azure Data Lake Gen2, and Databricks with Unity Catalog. It follows a medallion architecture (Bronze → Silver → Gold) and implements Slowly Changing Dimension Type 1 (SCD1) logic during the final transformation phase.

## Architecture Diagram
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

## Technologies Used
| Technology| Purpose |
| ---------------- | ------ |
|Azure Data Factory|   Orchestration of data pipeline and incremental load control.   |
| SQL Hat           |   True   |
| Codecademy Tee    |  False   |
| Codecademy Hoodie |  False   |
	
🔹 Note: Triggers were not used to reduce costs; the pipeline is manually run to avoid charges on a free-tier account (which uses serverless only).
Azure SQL Database	Staging area for source data and watermark tracking
Azure Data Lake Storage Gen2	Scalable storage for Bronze, Silver, and Gold layers
Azure Databricks	Data transformation, star schema modeling, and SCD1 upserts
Unity Catalog	Centralized governance and access control across data layers
Apache Spark	Distributed processing within Databricks
Delta Lake	ACID-compliant storage format for reliable data management
GitHub	Hosting the raw source CSV file (SalesData.csv)

## Lessons Learned
