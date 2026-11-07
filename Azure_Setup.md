
# Azure Environment Setup – Resource Creation

## Overview

Before implementing our ETL pipeline, we need to create the required Azure resources.

For our retail data engineering project, we will create the following resources:

1. Azure Resource Group
2. Azure Data Lake Storage Gen2 (ADLS Gen2)
3. Azure Data Factory (ADF)
4. Azure Databricks

---

## 1. Creating a Resource Group

A **Resource Group** is a logical container that holds related Azure resources.

It is similar to a folder where we organize and manage all the Azure services required for our project.

### Steps to Create a Resource Group

1. Log in to the Azure Portal.
2. Search for **Resource Groups**.
3. Click **Create**.
4. Select the required Subscription.
5. Provide a Resource Group name.

   Example: `RG_Retail_DataEngineering`

6. Select the required Azure Region.
7. Click **Review + Create**.
8. Click **Create**.

Once created, this Resource Group will act as a container for our project's Azure resources.

---

## 2. Creating an Azure Data Lake Storage Gen2 Account

After creating the Resource Group, the next step is to create a Storage Account that will be used as our data lake.

By default, an Azure Storage Account provides Blob Storage capabilities.

To enable Azure Data Lake Storage Gen2 capabilities, we need to enable the **Hierarchical Namespace (HNS)** feature.

### Steps to Create a Storage Account

1. Navigate to the Azure Portal.
2. Search for **Storage Accounts**.
3. Click **Create**.
4. Select the previously created Resource Group.
5. Provide a Storage Account name.

   Example: `stretaildata001`

6. Select the required Region.
7. Select the appropriate Performance and Redundancy options.

### Enabling Hierarchical Namespace

1. Navigate to the **Advanced** tab.
2. Locate the **Hierarchical Namespace** option.
3. Enable Hierarchical Namespace.
4. Review the remaining configuration.
5. Click **Review + Create**.
6. Click **Create**.

**Important:** Hierarchical Namespace must be enabled to use ADLS Gen2 capabilities. It is generally configured during storage account creation.

### Why Do We Enable Hierarchical Namespace?

Hierarchical Namespace provides a directory-based structure for organizing data.

It supports:

- Directories and subdirectories.
- File and directory-level access control.
- Efficient data management.
- Big data analytics workloads.

We will use this storage account to implement the Medallion Architecture:

- Bronze – Raw Data
- Silver – Cleansed Data
- Gold – Curated Data

---

## 3. Creating Azure Data Factory (ADF)

After creating the Storage Account, the next step is to create Azure Data Factory.

**Azure Data Factory (ADF)** is a cloud-based data integration and orchestration service.

It helps us move data between different sources and destinations, execute data pipelines, and orchestrate data processing activities.

### Steps to Create Azure Data Factory

1. Navigate to the Azure Portal.
2. Search for **Data Factories (V2)**.
3. Click **Create**.
4. Select the previously created Resource Group.
5. Provide a Data Factory name.

   Example: `adf-retail-dataengineering`

6. Select the required Azure Region.
7. Keep the remaining settings as required for the project.
8. Click **Review + Create**.
9. Click **Create**.

Once deployment is completed, open the newly created Data Factory.

Click **Launch Studio** to access the ADF development environment.

### Key Components of Azure Data Factory

We will use the following components while developing our ETL pipelines:

1. **Linked Services:** Establish connections between ADF and external data sources or destinations.

2. **Datasets:** Represent the structure, location, and format of source and destination data.

3. **Activities:** Define individual operations performed inside a pipeline, such as Copy Activity, Lookup, Filter, and ForEach.

4. **Pipelines:** Group multiple activities together to implement an end-to-end data integration workflow.

5. **Integration Runtime:** Provides the compute infrastructure required for data movement, activity dispatch, and supported data transformation operations.

6. **Triggers:** Execute pipelines based on schedules, events, or manual execution.

7. **Monitoring:** Helps monitor pipeline execution, activity status, execution duration, and failures.

---

## 4. Creating Azure Databricks

After creating Azure Data Factory, the next step is to create an Azure Databricks workspace.

**Azure Databricks** is a cloud-based data engineering and analytics platform built on Apache Spark.

In our project, Databricks will be responsible for processing and transforming data stored in ADLS Gen2.

We will use Databricks for:

- Reading raw data from the Bronze layer.
- Performing data cleansing and standardization.
- Handling NULL values and duplicate records.
- Performing data type conversions.
- Applying business rules and transformations.
- Writing the processed data into Silver and Gold layers in Parquet format.

### Steps to Create an Azure Databricks Workspace

1. Navigate to the Azure Portal.
2. click on create resource
2. Search for **Azure Databricks**.
3. Click **Create**.
4. Select the previously created Resource Group.
5. Provide a Workspace name.

   Example: `kpb-de-prj1rd-adb-acc`

6. Select the required Azure Region.
7. Select the appropriate Pricing Tier based on project requirements.
8. Configure the remaining networking and security settings as required.
9. Click **Review + Create**.
10. Click **Create**.

Once deployment is completed, open the Databricks workspace.

Click **Launch Workspace** to access the Databricks environment.

### Configuring Databricks for Data Processing

After creating the workspace, we need to configure Databricks to access our ADLS Gen2 storage account.

1. Open the Azure Databricks workspace.
2. Configure the required authentication mechanism for accessing ADLS Gen2.
3. Grant the required storage permissions to the Databricks identity.
4. Configure the necessary storage access and catalog settings.
5. Verify that Databricks can read and write data in the required ADLS Gen2 locations.

**Note:** Storage access can be configured using supported authentication methods such as managed identities or service principals. The exact configuration depends on the project's security setup.

### Databricks Compute

Databricks uses compute resources to execute Spark workloads.

For our project, we will use Databricks compute to execute data transformations.

The main components include:

1. **Driver Node:** Coordinates the execution of Spark applications and manages job execution.
2. **Worker Nodes:** Execute distributed data processing tasks.
3. **Cluster:** A group of compute resources used to execute Spark workloads.

### Data Processing Using Databricks

We will perform two major transformation stages.

**Bronze to Silver:**

1. Read raw data from ADLS Gen2 Bronze.
2. Perform data cleansing.
3. Handle NULL values and duplicates.
4. Remove unwanted whitespaces.
5. Perform data type conversions.
6. Apply data standardization.
7. Write the cleansed data into ADLS Gen2 Silver in Parquet format.

**Silver to Gold:**

1. Read cleansed data from ADLS Gen2 Silver.
2. Apply business rules.
3. Perform aggregations and calculations.
4. Prepare business-ready datasets.
5. Write the curated data into ADLS Gen2 Gold in Parquet format.

We will use **PySpark and Spark SQL** for these transformations.

---

## 5. Expected Azure Resource Structure

After completing the resource creation process, our Azure environment will look like this:

```text
Azure Subscription
│
└── Resource Group
    │
    ├── Storage Account (ADLS Gen2)
    │   │
    │   ├── Bronze
    │   ├── Silver
    │   └── Gold
    │
    ├── Azure Data Factory
    │   │
    │   ├── Linked Services
    │   ├── Datasets
    │   ├── Pipelines
    │   ├── Activities
    │   ├── Integration Runtime
    │   ├── Triggers
    │   └── Monitoring
    │
    └── Azure Databricks
        │
        ├── Workspace
        ├── Compute
        ├── Catalogs
        ├── Schemas
        ├── Tables
        └── Notebooks
```

---

## 6. Summary

In this initial setup, we create four important Azure resources:

| Azure Resource | Purpose |
|---|---|
| Resource Group | Logically organizes all project resources |
| ADLS Gen2 | Stores data across Bronze, Silver, and Gold layers |
| Azure Data Factory | Orchestrates data movement and pipeline execution |
| Azure Databricks | Performs data cleansing, transformations, and business logic |

### Overall Project Responsibility

- **Azure Data Factory:** Responsible for data ingestion, movement, and orchestration.
- **ADLS Gen2:** Responsible for storing data across the Medallion Architecture.
- **Azure Databricks:** Responsible for data processing and transformations.

Once all four resources are successfully created and configured, we can proceed with implementing our end-to-end retail ETL pipeline.
