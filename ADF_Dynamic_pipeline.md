
# Azure Data Factory – Dynamic Pipeline Implementation

## Overview

In a static pipeline, we manually configure the source file path, destination folder, and file name for every file.

To overcome this limitation, we implement a **Dynamic Pipeline** in Azure Data Factory.

A dynamic pipeline allows us to process multiple files using a single reusable pipeline without manually configuring each file.

For our retail data engineering project, we use:

- **Lookup Activity:** Reads file metadata from a JSON configuration file.
- **ForEach Activity:** Iterates over each file's metadata.
- **Copy Data Activity:** Copies source files from GitHub to the Bronze layer in ADLS Gen2.
- **Parameterized Datasets:** Dynamically accept source and destination details.

---

## 1. Creating a Dynamic Pipeline

1. Open Azure Data Factory.
2. Navigate to the **Author** section.
3. Create a new folder named `Dynamic_Pipeline`.
4. Inside the folder, create a new pipeline.
5. Provide a suitable pipeline name.

   Example: `PL_Dynamic_GitHub_To_Bronze`

6. Navigate to the Activities section.
7. Under **Move and Transform**, drag and drop the Copy Data activity onto the pipeline canvas.

We will later move this Copy Data activity inside the ForEach activity.

---

## 2. Creating a Dynamic Source Dataset

The source dataset is responsible for reading CSV files from our GitHub repository.

Unlike a static dataset, we configure this dataset using parameters so that it can accept different file paths dynamically.

### Steps to Create the Source Dataset

1. Open the Copy Data activity.
2. Navigate to the **Source** tab.
3. Click **New** to create a source dataset.
4. Select **HTTP** as the connector.
5. Select **DelimitedText (CSV)** as the file format.
6. Provide the dataset name:

   `ds_dynamic`

7. Select the existing HTTP linked service:

   `HttpGithubLinkedService`

8. Configure the Relative URL using the repository's raw GitHub file path.

   Example:

   ```text
   karthikpb13/Data_engineering_project1_data/refs/heads/main/Data/category.csv
   ```

   The linked service Base URL should be:

   ```text
   https://raw.githubusercontent.com/
   ```

### Configuring the Dataset Parameter

1. Open the newly created dataset.
2. Navigate to the **Parameters** tab.
3. Click **New**.
4. Create the following parameter:

   | Parameter Name | Type | Default Value |
   |---|---|---|
   | p_relative_url | String | Leave empty |

5. Navigate to the **Connection** tab.
6. Locate the Relative URL field.
7. Click **Add dynamic content**.
8. Select the dataset parameter `p_relative_url` from the expression builder.

   Alternatively, enter:

   ```text
   @dataset().p_relative_url
   ```

9. Click OK and save the dataset.

Now the source dataset can accept different GitHub file paths dynamically.

---

## 3. Creating a Dynamic Sink Dataset

The sink dataset is responsible for storing source files in the Bronze layer of ADLS Gen2.

We will parameterize the destination directory and file name so that each source file can be stored in its respective folder.

### Steps to Create the Sink Dataset

1. Open the Copy Data activity.
2. Navigate to the **Sink** tab.
3. Click **New** to create a sink dataset.
4. Select **Azure Data Lake Storage Gen2**.
5. Select **DelimitedText (CSV)** as the file format.
6. Provide a dataset name:

   `dynamic_bronze`

7. Select the existing linked service:

   `AzureDataLakeStorage1`

8. Select the Bronze container/file system.
9. Navigate to the **Parameters** tab.

### Configuring Dataset Parameters

Create the following parameters:

| Parameter Name | Type | Default Value |
|---|---|---|
| p_sink_folder | String | Leave empty |
| p_sink_filename | String | Leave empty |

### Configuring Dynamic Directory and File Name

1. Navigate to the **Connection** tab.
2. Select the Bronze container/file system.
3. For the Directory field, click **Add dynamic content**.
4. Enter:

   ```text
   @dataset().p_sink_folder
   ```

5. For the File Name field, click **Add dynamic content**.
6. Enter:

   ```text
   @dataset().p_sink_filename
   ```

7. Keep Import Schema as None.
8. Save the dataset.

Now the sink dataset can dynamically receive the destination folder and file name.

---

## 4. Creating the JSON Configuration File

To process multiple files dynamically, we need a collection of file metadata.

For this purpose, we create a JSON configuration file containing:

- Source file path.
- Destination folder name.
- Destination file name.

### Creating the JSON File

Create a JSON file named:

`git_dynamic_content.json`

The file will contain the following information:

```json
[
  {
    "p_relative_url": "karthikpb13/Data_engineering_project1_data/refs/heads/main/Data/category.csv",
    "p_sink_folder": "category",
    "p_sink_filename": "category.csv"
  },
  {
    "p_relative_url": "karthikpb13/Data_engineering_project1_data/refs/heads/main/Data/products.csv",
    "p_sink_folder": "products",
    "p_sink_filename": "products.csv"
  },
  {
    "p_relative_url": "karthikpb13/Data_engineering_project1_data/refs/heads/main/Data/sales.csv",
    "p_sink_folder": "sales",
    "p_sink_filename": "sales.csv"
  },
  {
    "p_relative_url": "karthikpb13/Data_engineering_project1_data/refs/heads/main/Data/stores.csv",
    "p_sink_folder": "stores",
    "p_sink_filename": "stores.csv"
  },
  {
    "p_relative_url": "karthikpb13/Data_engineering_project1_data/refs/heads/main/Data/warranty.csv",
    "p_sink_folder": "warranty",
    "p_sink_filename": "warranty.csv"
  }
]
```

**Note:** These relative paths assume the HTTP linked service Base URL is:

```text
https://raw.githubusercontent.com/
```

### Uploading the JSON File to ADLS Gen2

1. Open the Azure Portal.
2. Navigate to the ADLS Gen2 storage account.
3. Open the required storage container.
4. Create a new directory named:

   `parameters`

5. Upload `git_dynamic_content.json` into this directory.

Expected structure:

```text
retail-data/
│
├── Bronze/
│
└── parameters/
    └── git_dynamic_content.json
```

This JSON file will act as the metadata source for our dynamic pipeline.

---

## 5. Configuring the Lookup Activity

The Lookup activity retrieves the JSON configuration file from ADLS Gen2.

It reads the file metadata and provides the output to the ForEach activity.

### Steps to Configure Lookup

1. Open the dynamic pipeline.
2. Navigate to Activities.
3. Search for **Lookup**.
4. Drag and drop the Lookup activity onto the pipeline canvas.
5. Provide the activity name:

   `Lookup_git`

6. Navigate to the **Settings** tab.
7. Create or select the source dataset.
8. Select Azure Data Lake Storage Gen2.
9. Select JSON as the file format.
10. Select the appropriate ADLS Gen2 linked service.
11. Configure the dataset to point to:

    - Container: `retail-data`
    - Directory: `parameters`
    - File: `git_dynamic_content.json`

12. Save the configuration.

### Configuring First Row Only

By default, Lookup can be configured to retrieve only the first row.

Since our JSON file contains multiple file metadata objects, we need the complete collection.

1. Navigate to the Lookup activity Settings.
2. Locate **First row only**.
3. Uncheck the option.

This allows Lookup to return the complete JSON array in its output.

### Debugging the Lookup Activity

1. Temporarily deactivate other activities if necessary.
2. Click **Debug**.
3. Wait for successful execution.
4. Open the pipeline execution output.
5. Navigate to the Lookup activity output.

The output should contain a `value` array with all the file metadata.

Example:

```json
{
  "value": [
    {
      "p_relative_url": ".../category.csv",
      "p_sink_folder": "category",
      "p_sink_filename": "category.csv"
    },
    {
      "p_relative_url": ".../products.csv",
      "p_sink_folder": "products",
      "p_sink_filename": "products.csv"
    }
  ],
  "count": 5
}
```

The actual output will contain all five file entries.

**Important:** The `value` property contains the array that we will pass to the ForEach activity.

---

## 6. Connecting Lookup to ForEach

After successfully configuring the Lookup activity, the next step is to connect it to the ForEach activity.

ADF provides different activity dependencies:

- Succeeded
- Failed
- Skipped
- Completed

For our pipeline, we select **Succeeded**.

### Steps

1. Drag and drop the ForEach activity onto the pipeline canvas.
2. Connect the Lookup activity to ForEach.
3. Select the Succeeded dependency.

This ensures that ForEach executes only after Lookup successfully retrieves the JSON configuration.

---

## 7. Configuring the ForEach Activity

The ForEach activity iterates over the collection received from the Lookup activity.

The most important property of ForEach is **Items**.

Items defines the collection over which the activity should iterate.

### Steps to Configure ForEach

1. Select the ForEach activity.
2. Navigate to the **Settings** tab.
3. Locate the Items field.
4. Click **Add dynamic content**.
5. Provide the following expression:

   ```text
   @activity('Lookup_git').output.value
   ```

6. Click OK.

### Understanding the Expression

```text
@activity('Lookup_git').output.value
```

- `activity('Lookup_git')` refers to the Lookup activity.
- `output` retrieves the activity output.
- `value` retrieves the array containing all file metadata.

Now ForEach receives all five file entries and iterates over them one by one.

---

## 8. Configuring Copy Data Activity Inside ForEach

Now we need to place the Copy Data activity inside the ForEach activity.

Instead of executing a separate Copy Data activity for every file, we will use one reusable Copy Data activity inside the loop.

### Steps

1. Select the existing Copy Data activity from the pipeline canvas.
2. Cut the activity.
3. Open the ForEach activity.
4. Click the pencil icon to open its inner activity canvas.
5. Paste the Copy Data activity inside the ForEach canvas.

Now the Copy Data activity will execute for every item received by ForEach.

### 8.1 Configuring the Source

1. Select the Copy Data activity.
2. Navigate to the Source tab.
3. Select the dynamic source dataset:

   `ds_dynamic`

4. Under Dataset Properties, locate `p_relative_url`.
5. Click **Add dynamic content**.
6. Provide the expression:

   ```text
   @item().p_relative_url
   ```

This expression retrieves the GitHub file path from the current ForEach iteration.

### 8.2 Configuring the Sink

1. Navigate to the Sink tab.
2. Select the dynamic sink dataset:

   `dynamic_bronze`

3. Under Dataset Properties, locate `p_sink_folder`.
4. Click **Add dynamic content** and provide:

   ```text
   @item().p_sink_folder
   ```

5. For `p_sink_filename`, provide:

   ```text
   @item().p_sink_filename
   ```

These expressions retrieve the destination directory and file name from the current JSON object.

---

## 9. Understanding the Dynamic Pipeline Execution

The complete execution process is as follows:

1. Lookup reads `git_dynamic_content.json` from ADLS Gen2.
2. Lookup returns an array containing five file metadata objects.
3. ForEach receives the array from Lookup.
4. ForEach iterates over each metadata object.
5. During each iteration, Copy Data receives the current source file path.
6. The source dataset reads the corresponding CSV file from GitHub.
7. The sink dataset receives the corresponding destination folder and file name.
8. Copy Data transfers the file into the Bronze layer.
9. The same Copy Data activity is reused for every iteration.

### Example Execution

| Iteration | Source File | Bronze Folder | Destination File |
|---|---|---|---|
| 1 | category.csv | category | category.csv |
| 2 | products.csv | products | products.csv |
| 3 | sales.csv | sales | sales.csv |
| 4 | stores.csv | stores | stores.csv |
| 5 | warranty.csv | warranty | warranty.csv |

---

## 10. Final Pipeline Architecture

```text
             ADLS Gen2
                 |
                 v
       git_dynamic_content.json
                 |
                 v
           Lookup Activity
                 |
                 v
           ForEach Activity
                 |
                 v
          Copy Data Activity
                 |
          +------+------+
          |             |
          v             v
        GitHub        ADLS Gen2
        Source        Bronze Layer
          |             |
          +------+------+
                 |
                 v
         Multiple CSV Files
         in Respective Folders
```

---

## 11. Debugging and Validating the Dynamic Pipeline

After completing the configuration:

1. Save the pipeline.
2. Click **Debug**.
3. Wait for the Lookup activity to complete.
4. Verify that ForEach receives the expected number of items.
5. Verify that Copy Data executes for each iteration.
6. Check the execution status of every iteration.
7. Open the ADLS Gen2 storage account.
8. Navigate to the Bronze container.
9. Verify that all five CSV files are available in their respective folders.

### Validation Checklist

- [ ] Lookup successfully reads the JSON configuration file.
- [ ] Lookup returns all five metadata objects.
- [ ] First row only is unchecked.
- [ ] ForEach receives the Lookup output array.
- [ ] ForEach iterates over all five objects.
- [ ] Source dataset receives the correct relative URL.
- [ ] Sink dataset receives the correct folder name.
- [ ] Sink dataset receives the correct file name.
- [ ] All five CSV files are successfully copied.
- [ ] Files are available in the correct Bronze directories.
- [ ] Source and target record counts are validated.

---

## 12. Static Pipeline vs Dynamic Pipeline

| Static Pipeline | Dynamic Pipeline |
|---|---|
| File paths are hardcoded | File paths are passed dynamically |
| Separate configurations for files | Reusable parameterized datasets |
| Destination folders are predefined | Destination folders are passed dynamically |
| Manual configuration | Metadata-driven configuration |
| More maintenance | Less maintenance |
| Suitable for initial demonstration | Suitable for scalable ingestion |

---

## 13. Final Outcome

We have designed a metadata-driven dynamic ingestion pipeline using Azure Data Factory.

Instead of creating separate pipelines for every CSV file, we use:

- One JSON configuration file.
- One Lookup activity.
- One ForEach activity.
- One reusable Copy Data activity.
- One parameterized source dataset.
- One parameterized sink dataset.

This approach makes the ingestion process reusable, scalable, and easier to maintain.

**Next Step:** Implement the Bronze-to-Silver transformation process using Azure Databricks and PySpark.
