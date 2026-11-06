
# Setting Up Azure Data Lake Storage Gen2 – Medallion Architecture

## Overview

After creating the Azure Data Lake Storage Gen2 (ADLS Gen2) account, the next step is to create the required folders to implement the **Medallion Architecture**.

In our retail data engineering project, we divide the data lake into three layers based on the level of data processing:

1. Bronze – Raw Data
2. Silver – Cleansed Data
3. Gold – Curated Data

Each layer has a specific responsibility in the overall ETL process.

---

## 1. Opening the Storage Account

1. Log in to the Azure Portal.
2. Search for **Storage Accounts**.
3. Open the storage account created for our retail project.

   Example: `stretaildata001`

4. Navigate to **Data Storage → Containers**.

---

## 2. Creating a Container

Before creating the folders, we need to create a container in the storage account.

1. Click **+ Container**.
2. Provide a container name.

   Example: `retail-data`

3. Select the appropriate access level, preferably **Private (no anonymous access)**.
4. Click **Create**.

The container acts as the top-level storage location for our project data.

---

## 3. Creating Medallion Architecture Folders

After creating the container, open the `retail-data` container.

We will create three main directories inside this container.

### 3.1 Bronze Layer – Raw Data

The Bronze layer stores the original data ingested from different source systems.

For our project, the source data comes from GitHub CSV files representing retail business systems.

**Steps:**

1. Open the `retail-data` container.
2. Click **Add Directory**.
3. Enter the directory name:

   `Bronze`

4. Click **Save**.

**Purpose:**
- Stores raw ingested data.
- Maintains the original source structure.
- Acts as the starting point for data processing.
- Helps preserve source data for reprocessing.

### 3.2 Silver Layer – Cleansed Data

The Silver layer stores cleaned and standardized data after applying data quality transformations.

**Steps:**

1. Navigate back to the `retail-data` container.
2. Click **Add Directory**.
3. Enter the directory name:

   `Silver`

4. Click **Save**.

**Purpose:**
- Stores cleansed data.
- Handles NULL values and duplicate records.
- Removes unwanted whitespaces.
- Performs data type conversions.
- Applies data standardization.

### 3.3 Gold Layer – Curated Data

The Gold layer stores business-ready data after applying business rules and transformations.

**Steps:**

1. Navigate back to the `retail-data` container.
2. Click **Add Directory**.
3. Enter the directory name:

   `Gold`

4. Click **Save**.

**Purpose:**
- Stores curated and business-ready data.
- Applies business rules and calculations.
- Performs aggregations.
- Prepares datasets for reporting and analytics.

---

## 4. Expected Folder Structure

After creating all three directories, our ADLS Gen2 structure will look like this:

```text
retail-data/
│
├── Bronze/
│   ├── category/
│   ├── customers/
│   ├── products/
│   ├── orders/
│   └── sales/
│
├── Silver/
│   ├── category/
│   ├── customers/
│   ├── products/
│   ├── orders/
│   └── sales/
│
└── Gold/
    ├── sales_summary/
    ├── product_performance/
    ├── customer_analysis/
    └── store_performance/
```

*Note: The subdirectories shown above are examples of the planned project structure. They can be created as the corresponding datasets are implemented.*

---

## 5. Data Storage Format

We will use Parquet format for storing processed data in the Silver and Gold layers.

| Layer | Data Type | Storage Format |
|---|---|---|
| Bronze | Raw source data | CSV |
| Silver | Cleansed and standardized data | Parquet |
| Gold | Curated and business-ready data | Parquet |

---

## 6. Access Control and Permissions

Each layer can have its own access permissions to ensure data security.

- Bronze access: Authorized users or services can access raw data.
- Silver access: Authorized users or services can access cleansed data.
- Gold access: Authorized users or services can access curated data.

Access can be managed using Azure RBAC and ADLS Gen2 ACLs, depending on the security requirements.

---

## 7. Final Outcome

After completing this setup, our ADLS Gen2 storage account will contain three main directories:

- **Bronze:** Stores raw ingested data.
- **Silver:** Stores cleansed and standardized data.
- **Gold:** Stores curated data after applying business requirements.

These three layers will serve as the foundation for our retail ETL pipeline.

Azure Data Factory will handle data ingestion and movement, while Azure Databricks will perform the transformations between the Bronze, Silver, and Gold layers.
