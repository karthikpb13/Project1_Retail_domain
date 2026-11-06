
# ADF Static Pipeline – Source-to-Bronze Ingestion

## Overview

This document explains how to create a static Azure Data Factory (ADF) pipeline to copy CSV files from a public GitHub repository into the Bronze layer of Azure Data Lake Storage Gen2 (ADLS Gen2).

In a static pipeline, file details are configured manually. For this demonstration, we create a separate pipeline for each file and explicitly provide the source path, destination directory, and file name.

**Source:** GitHub  
**Destination:** ADLS Gen2 (Bronze Layer)  
**Activity:** Copy Data

---

## 1. Creating a Folder in ADF

1. Open Azure Data Factory.
2. Navigate to the **Author** section.
3. Create a new folder.
4. Provide a suitable name, such as `Retail_Source_To_Bronze`.

*Note: This folder is only for organizing ADF resources. It does not create a physical folder in ADLS Gen2.*

## 2. Creating a Pipeline

1. Navigate to the **Author** section.
2. Create a new pipeline.
3. Provide a pipeline name, such as `PL_Category_GitHub_To_Bronze`.
4. Under Activities, navigate to **Move and Transform**.
5. Drag and drop the **Copy Data** activity onto the pipeline canvas.

## 3. Creating Linked Services

Linked services establish connections between ADF and external data sources or destinations.

### 3.1 Source Linked Service – GitHub (HTTP)

1. Navigate to **Manage → Linked Services**.
2. Click **New**.
3. Search for and select **HTTP**.
4. Provide the linked service name: `LS_GitHub_HTTP`.
5. Configure the Base URL:

   `https://raw.githubusercontent.com/karthikpb13/Data_engineering_project1_data/main/`

6. Select **Anonymous** authentication (repository must be public).
7. Select `AutoResolveIntegrationRuntime`.
8. Keep server certificate validation enabled.
9. Click **Test Connection**.
10. If successful, create the linked service.

### 3.2 Bronze Linked Service – ADLS Gen2

1. Navigate to **Manage → Linked Services**.
2. Click **New**.
3. Search for and select **Azure Data Lake Storage Gen2**.
4. Provide the linked service name: `LS_ADLS_Gen2`.
5. Select the required storage account.
6. Configure the appropriate authentication method.
7. Test the connection and create the linked service.

## 4. Creating Datasets

A dataset defines the location and format of the source or destination data.

### 4.1 Source Dataset – GitHub CSV

1. Open the Copy Data activity.
2. Navigate to the **Source** tab.
3. Click **New** to create a source dataset.
4. Select **HTTP**.
5. Choose the `LS_GitHub_HTTP` linked service.
6. Select **DelimitedText** as the file format.
7. Provide the Relative URL:

   `Data/category.csv`

8. keep first row as header and import schema as none
9. Save the dataset.

The complete URL will be:

`https://raw.githubusercontent.com/karthikpb13/Data_engineering_project1_data/main/Data/category.csv`

### 4.2 Sink Dataset – ADLS Gen2

1. Navigate to the **Sink** tab.
2. Click **New** to create a sink dataset.
3. Select **Azure Data Lake Storage Gen2**.
4. Choose the `LS_ADLS_Gen2` linked service.
5. Select **DelimitedText** as the file format.
6. Select the Bronze container.
7. Provide the directory name:

   `category`

8. Provide the output file name:

   `category.csv`

9. Save the dataset.

## 5. Debugging the Pipeline

1. Select the Copy Data activity.
2. Verify the source and sink configurations.
3. Click **Debug** to execute the pipeline.
4. Wait for the execution to complete.
5. Verify that the pipeline status is **Succeeded**.

## 6. Validating Data in the Bronze Layer

After successful execution:

1. Open the ADLS Gen2 storage account.
2. Navigate to the Bronze container.
3. Open the `category` directory.
4. Verify that `category.csv` is available.
5. Open the file using the available preview or download option.
6. Verify the actual data.

### Validation Checklist

- [ ] Pipeline execution status is Succeeded.
- [ ] File is available in the Bronze layer.
- [ ] Source and target file names match.
- [ ] Correct directory structure is maintained.
- [ ] File is not empty or corrupted.
- [ ] Column headers and delimiters are correct.
- [ ] Source and target record counts match.
- [ ] Source and target file sizes are checked.

## 7. Expected Result

The `category.csv` file from GitHub is successfully copied into the Bronze layer of ADLS Gen2.

### Expected Folder Structure

```text
Bronze/
└── category/
    └── category.csv
```

## 8. Static Pipeline – Key Points

- File details are provided manually.
- Source Relative URL is configured explicitly.
- Destination directory and file name are configured manually.
- The configuration is repeated for every additional file.
- This approach is useful for understanding the fundamentals of ADF.

**Next Step:** Implement a dynamic pipeline using Lookup, Filter, ForEach, and parameterized datasets to process multiple files using a reusable pipeline.
