# Overall Process

## 1. Azure Data Factory (ADF) – Data Ingestion

- Data can come from various sources, such as SQL databases, ERP systems, cloud applications, and files.
- To represent all these sources in our project, we are using CSV files.
- Our first responsibility is to move the source data into the cloud environment. For this, we use **Azure Data Factory (ADF)**.
- Instead of creating a separate pipeline for every file, we use a **single reusable pipeline** to handle different types of files.
- We use different ADF activities, such as:
  - Lookup
  - Filter
  - ForEach
  - Copy Activity

These activities work together to automate the data ingestion process.

Once the data is ingested, we store it in **Azure Data Lake Storage Gen2 (ADLS Gen2)** for further processing.

## 2. ADLS Gen2 – Medallion Architecture

In ADLS Gen2, we implement the **Medallion Architecture**, which divides data into three layers based on the level of processing performed.

1. **Bronze (Raw):** Stores the original ingested data.
2. **Silver (Cleansed):** Stores cleaned and standardized data.
3. **Gold (Curated):** Stores data transformed according to business requirements.

This architecture plays an important role in data processing. If something goes wrong during any processing stage, we can refer back to the previous layer and restart the required transformations.

After each processing stage, we store the data in **Parquet format**.

For example:
- Bronze to Silver: After data cleaning and standardization, the processed data is stored in Parquet format in the Silver layer.
- Silver to Gold: After applying business rules and business requirements, the transformed data is stored in Parquet format in the Gold layer.

### Data Access and Permissions

Each layer in ADLS Gen2 has a specific set of access permissions to ensure that not everyone can access data across all layers.

- Bronze team members can access only Bronze data.
- Silver team members can access only Silver data.
- Gold team members can access only Gold data.

This helps maintain data security and access control.

## 3. Databricks – Data Transformation

For data processing and transformations, we use **Azure Databricks**.

We configure Databricks to access data stored in ADLS Gen2. Once the connection is established, we perform transformations using:

- Spark DataFrames
- Spark SQL
- Temporary Views and Tables

### Bronze to Silver Processing

1. Read the raw data from the Bronze layer.
2. Perform data cleansing.
3. Handle NULL values and duplicate records.
4. Remove unwanted whitespaces.
5. Perform data type conversions.
6. Apply data standardization.
7. Write the processed data into the Silver layer in Parquet format.

### Silver to Gold Processing

1. Read the cleansed data from the Silver layer.
2. Apply business rules and business transformations.
3. Perform aggregations and calculations wherever required.
4. Prepare the final curated datasets.
5. Write the transformed data into the Gold layer in Parquet format.

## 4. Snowflake – Target Data Warehouse

Once the data is successfully processed and stored in the Gold layer, we use **Azure Data Factory (ADF)** to move the curated data into Snowflake.

Snowflake acts as our final target data warehouse, where the business-ready data is stored for reporting and analytics.

## 5. Overall Data Movement

The complete data flow of our project is:

1. **Source → ADLS Gen2 Bronze:** ADF ingests source data and stores it in the Bronze layer.
2. **Bronze → Databricks → Silver:** Databricks performs data cleansing and standardization and stores the output in Silver.
3. **Silver → Databricks → Gold:** Databricks applies business rules and transformations and stores the output in Gold.
4. **Gold → Snowflake:** ADF loads the curated data into Snowflake.

### Overall Architecture Flow

**Source Systems → Azure Data Factory → ADLS Gen2 (Bronze) → Databricks → ADLS Gen2 (Silver) → Databricks → ADLS Gen2 (Gold) → Azure Data Factory → Snowflake**