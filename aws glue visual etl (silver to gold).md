# Visual ETL Data Transformation: [Insert Dataset Name] (Gold Layer)

## 📌 Overview
This job extracts cleansed and joined data from the Silver layer, applies business-level aggregations and filtering, and saves it as an optimized Iceberg table in the Gold layer. This final dataset is structured specifically for reporting, BI tools (like Amazon QuickSight), and advanced analytics.

Things to consider:
- Ingestion to the Gold layer usually involves heavy transformations, aggregations (e.g., group by, sum, count), and dropping unnecessary columns to improve query performance.
- The schema should be highly refined and strictly enforce business rules.

Gemini said
Here is a breakdown of how aggregate functions are used when moving data from the Silver to the Gold layer, and why this step is so critical for a modern data architecture.

## 📌 The Relevance: Why Aggregate for the Gold Layer?
In a medallion data architecture (Bronze ➡️ Silver ➡️ Gold), each layer serves a distinct purpose:

Silver Layer: This is your "single source of truth." It contains cleansed, standardised, and joined data, but it is still at the row-level (granular). For example, it might contain 10 million rows representing every individual research application over the last ten years.

Gold Layer: This is thepresentation layer. Business intelligence (BI) tools (like Amazon QuickSight, Tableau, or Power BI), executives, and data analysts usually don't want to look at 10 million individual rows. They want to see summaries and trends.

## 🖼️ Visual Pipeline
*(Below is the conceptual layout of the AWS Glue Visual Canvas for this job)*



## ⚙️ Step-by-Step Technical Details

### 1. Set the name of the Visual ETL
Name the file clearly to indicate the target dataset and layer, for example: `silver_to_gold_[dataset_name]`.

### 2. Set the job details (minimal things to do)
 - **IAM Role:** select 'AWSGlueServiceRole'
 - **Glue Version:** Default selection 
 - **Language:** Default Selection ('Python 3')
 - **Worker type:** choose minimal 'G 2x' (the higher the type, processing will be fast but it will incur more cost!!)
 - **Requested number of workers:** set it to minimal '2' or more (the more the worker, the higher the cost charge!!)

### 3. Add node (click on the blue '+' button at top right of the canvas)

### 3a. Data Source (Silver Layer) - select 'Source' > 'AWS Glue Data Catalog'
* **Name Nodes/source:** `[insert_silver_table_name]`
* **Database:** `prism_silver`
* **Table Name:** `[insert_silver_table_name]`
* **Settings:** Ensure no partitions are accidentally excluded unless specifically filtering the read.

### 3b. Transform: Aggregate / SQL Query - select 'Transform' > 'Aggregate' (or 'SQL Query')

Refine the data to meet business reporting requirements.
* **Node Parents:** Source Node `[insert_silver_table_name]`
* **Group By:** Select the columns to group by (e.g., `year`, `department`, `region`).
* **Aggregate Functions:** Define the math (e.g., `COUNT` on `id`, `SUM` on `revenue`).
* *(Optional)* Use a **Drop Fields** transform to remove any intermediate columns that BI tools do not need to see.

### 3c. Data Target (Gold Layer) - select 'Target' > 'Amazon S3'
The aggregated, business-ready data is saved to the Gold layer inside the Glue Data Catalogue.
* **Node Parents:** Transform Node 'Aggregate' (or your final transform node)
* **Database:** Select the database `prism_gold`
* **Table Name:** Set the table name as `[insert_gold_table_name]`
* **Target Location:** `s3://[your-gold-bucket-name]/prism_gold/[insert_gold_table_name]`
* **Format:** Select Apache Iceberg (Format version 2)
* **Compression:** Snappy
* **Partition Key:** Add partition keys highly optimized for your BI queries (e.g., `reporting_year`, `region`).
* **Save Logic:** The script automatically checks if the table exists. If it is new, it creates it. If it already exists, it either appends or overwrites based on your business requirement for this specific table (Overwrite is common for Gold aggregate tables to prevent duplicate counts).

### 4. Save the visual ETL file , then click "RUN"
- Progress of the job run can be monitored in the 'Job Run Monitoring' menu on the left.