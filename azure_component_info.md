
# Azure Data Factory – Linked Services and Datasets

## Overview

In Azure Data Factory, Linked Services and Datasets are important components used to establish connections and identify the location of source and destination data.

For our retail data engineering project, we have created two Linked Services and five Datasets to support both static and dynamic data ingestion from GitHub to ADLS Gen2.

---

## 1. Linked Services

Linked Services establish connections between Azure Data Factory and external data sources or destinations.

We have configured the following Linked Services:

### 1.1 AzureDataLakeStorage1

**Purpose:** Establishes a connection between Azure Data Factory and our Azure Data Lake Storage Gen2 account.

- Connector: Azure Data Lake Storage Gen2
- Target: Azure Storage Account
- Usage: Used to write ingested data into the Bronze layer.

### 1.2 HttpGithubLinkedService

**Purpose:** Establishes an HTTP connection between Azure Data Factory and our GitHub repository.

- Connector: HTTP
- Source: GitHub
- Repository: `Data_engineering_project1_data`
- Usage: Used to read CSV files from the GitHub repository.

---

## 2. Datasets

Datasets represent the location, structure, and format of the data that ADF reads or writes.

We have created the following datasets for static and dynamic ingestion.

### 2.1 ds_bronze

**Purpose:** Static sink dataset connected to ADLS Gen2.

- Linked Service: `AzureDataLakeStorage1`
- Layer: Bronze
- Directory: `category`
- File Name: `category.csv`
- File Format: CSV

This dataset is used to store the category CSV file in the Bronze layer.

### 2.2 ds_git

**Purpose:** Static source dataset connected to GitHub.

- Linked Service: `HttpGithubLinkedService`
- Repository: `Data_engineering_project1_data`
- Directory: `Data`
- File Name: `category.csv`
- File Format: CSV

This dataset reads the category CSV file directly from the GitHub repository using a predefined file path.

### 2.3 ds_git_parameter

**Purpose:** Dataset used by the Lookup activity to retrieve dynamic ingestion parameters from a JSON configuration file.

- Linked Service: `HttpGithubLinkedService`
- Source: GitHub repository
- File Format: JSON
- Usage: Provides file paths, destination folder names, and destination file names for dynamic ingestion.

The JSON configuration file contains metadata for multiple source files.

Example configuration:

```json
[
  {
    "p_relative_url": "Data/category.csv",
    "p_sink_folder": "category",
    "p_sink_filename": "category.csv"
  },
  {
    "p_relative_url": "Data/products.csv",
    "p_sink_folder": "products",
    "p_sink_filename": "products.csv"
  }
]
```

### 2.4 dsdynic

**Purpose:** Dynamic source dataset connected to GitHub.

- Linked Service: `HttpGithubLinkedService`
- Source: GitHub repository
- Relative URL: Parameterized
- File Format: CSV

Unlike the static source dataset, this dataset accepts the source file path dynamically through parameters.

The Lookup activity retrieves file information from the JSON configuration, and the ForEach activity passes the respective file path to this dataset.

### 2.5 dynamic_bronze

**Purpose:** Dynamic sink dataset connected to ADLS Gen2.

- Linked Service: `AzureDataLakeStorage1`
- Target: Bronze layer
- Directory: Parameterized
- File Name: Parameterized
- File Format: CSV

This dataset dynamically receives the destination folder name and file name from the pipeline parameters.

It allows multiple source files to be stored in their respective Bronze directories without creating separate sink datasets for every file.

---

## 3. Dataset Summary

| Dataset Name | Type | Linked Service | Purpose |
|---|---|---|---|
| ds_bronze | Static | AzureDataLakeStorage1 | Static Bronze destination |
| ds_git | Static | HttpGithubLinkedService | Static GitHub source |
| ds_git_parameter | JSON | HttpGithubLinkedService | Lookup configuration |
| dsdynic | Dynamic | HttpGithubLinkedService | Dynamic GitHub source |
| dynamic_bronze | Dynamic | AzureDataLakeStorage1 | Dynamic Bronze destination |

---

## 4. Static vs Dynamic Dataset Configuration

| Static Dataset | Dynamic Dataset |
|---|---|
| File path is predefined | File path is parameterized |
| Folder name is predefined | Folder name is parameterized |
| File name is predefined | File name is parameterized |
| Requires manual configuration for each file | Supports multiple files using reusable datasets |
| Used for initial demonstration | Used for automated ingestion |

---

## 5. Overall Data Flow

### Static Ingestion

```text
GitHub
   |
   v
ds_git
   |
   v
Copy Data Activity
   |
   v
ds_bronze
   |
   v
ADLS Gen2 – Bronze
   |
   v
category/category.csv
```

### Dynamic Ingestion

```text
GitHub JSON Configuration
          |
          v
   ds_git_parameter
          |
          v
      Lookup Activity
          |
          v
       ForEach
          |
          v
    Copy Data Activity
          |
          |-------------------|
          v                   v
       dsdynic          dynamic_bronze
          |                   |
          v                   v
       GitHub              ADLS Gen2
       Source              Bronze Layer
          |                   |
          |-------------------|
                  |
                  v
       Multiple CSV Files
       in Respective Folders
```

---

## 6. Expected Outcome

After configuring these Linked Services and Datasets:

- ADF can connect to the GitHub repository.
- ADF can connect to the ADLS Gen2 storage account.
- Static datasets can copy individual CSV files.
- The JSON configuration dataset can provide metadata to the Lookup activity.
- Dynamic datasets can accept file paths and destination details through parameters.
- The same dynamic pipeline can process multiple CSV files using Lookup, ForEach, and Copy Data activities.

This configuration establishes the foundation for implementing reusable and automated Source-to-Bronze ingestion in our retail data engineering project.
