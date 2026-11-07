# Steps to Run Azure Data Factory Pipelines

## 1. Pipeline execution order

Run the retail data pipelines in the following order:

1.  **Source-to-Bronze pipeline** --- loads source data into the Bronze
    layer.
2.  **Silver pipeline** --- transforms Bronze data and writes the
    cleansed data to Silver.
3.  **Gold pipeline** --- reads Silver data, performs business-level
    aggregations, and writes the results to Gold.

The ADF workspace currently shows these pipeline folders and pipeline
entries:

  -----------------------------------------------------------------------
  ADF folder                          Pipeline shown
  ----------------------------------- -----------------------------------
  `silver_pipeline`                   `de_prj1_rd_silver_pipeline`

  `gold_pipeline`                     `de_prj1_rd_gold_pipeline`

  `source_to_bronze_Dynamic_p...`     Pipeline name is truncated in the
                                      screenshot; select the
                                      source-to-Bronze pipeline shown in
                                      ADF.
  -----------------------------------------------------------------------

## 2. Before running the Silver pipeline: start Databricks compute

The Silver and Gold notebooks use the Databricks **All-Purpose Compute**
cluster:

`kpb-de-prj1-rd-adf-compute`

Because this is an existing interactive All-Purpose cluster, start it
before triggering the Silver pipeline if it is stopped or terminated.
ADF's Notebook activity configured to use an existing interactive
cluster does not automatically start a terminated cluster.

1.  Open the Azure Databricks workspace.
2.  Go to **Compute**.
3.  Find `kpb-de-prj1-rd-adf-compute`.
4.  If its status is **Terminated**, select **Start**.
5.  Wait until the cluster status becomes **Running**.
6.  Keep the cluster available while the Silver and Gold notebook
    activities execute.

If the cluster is already Running, no manual start is required.

## 3. Run the Source-to-Bronze pipeline

1.  Open Azure Data Factory Studio.
2.  Select **Author**.
3.  Find the source-to-Bronze pipeline in the pipeline list.
4.  Open the pipeline and confirm that it is the intended
    source-to-Bronze pipeline.
5.  Select **Add trigger** → **Trigger now**.
6.  Confirm the trigger.
7.  Open **Monitor** → **Pipeline runs**.
8.  Check the pipeline run status.
9.  Continue only after the pipeline succeeds and the expected Bronze
    data is available.

## 4. Start Databricks All-Purpose Compute

Before triggering Silver, follow the steps in Section 2 and confirm that
`kpb-de-prj1-rd-adf-compute` is Running.

## 5. Run the Silver pipeline

1.  In ADF Studio, open **Author**.
2.  Expand the `silver_pipeline` folder.
3.  Open `de_prj1_rd_silver_pipeline`.
4.  Confirm that its Databricks Notebook activity uses the
    `LS_AzureDatabricks` linked service and the Silver transformation
    notebook.
5.  Select **Add trigger** → **Trigger now**.
6.  Confirm the trigger.
7.  Open **Monitor** → **Pipeline runs**.
8.  Open the pipeline run and inspect the Notebook activity status and
    output.
9.  Confirm that the pipeline succeeds and the expected transformed data
    is written to the Silver layer.

## 6. Run the Gold pipeline

Run Gold only after the Silver pipeline succeeds and its output is
available.

1.  In ADF Studio, open **Author**.
2.  Expand the `gold_pipeline` folder.
3.  Open `de_prj1_rd_gold_pipeline`.
4.  Confirm that its Databricks Notebook activity uses the
    `LS_AzureDatabricks` linked service and the Gold transformation
    notebook.
5.  Select **Add trigger** → **Trigger now**.
6.  Confirm the trigger.
7.  Open **Monitor** → **Pipeline runs**.
8.  Open the pipeline run and inspect the Notebook activity status and
    output.
9.  Confirm that the pipeline succeeds and the expected Gold datasets
    are written to the Gold layer.

## 7. Monitor pipeline execution

For each pipeline:

1.  Open **Monitor** in ADF Studio.
2.  Select **Pipeline runs**.
3.  Locate the latest run using its pipeline name and start time.
4.  Check the **Status** column.
5.  Open the run to view its activity runs.
6.  Select the Databricks Notebook activity to inspect its output and
    any error details.

Do not start the next layer if the preceding pipeline failed. Resolve
the failure, rerun the failed pipeline, and verify that its output is
available before continuing.

## 8. After execution

-   Verify the expected files or tables in the target layer.
-   Check that Silver output is available before running Gold.
-   Check that all five Gold transformation outputs are present in the
    Gold layer.
-   Stop the All-Purpose cluster when no further notebook execution is
    required, to avoid unnecessary compute charges. Only stop it after
    all dependent notebook runs have completed.

## 9. Quick execution checklist

-   [ ] Run the source-to-Bronze pipeline.
-   [ ] Verify that Bronze data is available.
-   [ ] Start `kpb-de-prj1-rd-adf-compute` if it is terminated.
-   [ ] Confirm that the cluster status is Running.
-   [ ] Run `de_prj1_rd_silver_pipeline`.
-   [ ] Verify Silver pipeline success and Silver output.
-   [ ] Run `de_prj1_rd_gold_pipeline`.
-   [ ] Verify Gold pipeline success and Gold outputs.
-   [ ] Stop the All-Purpose cluster when all notebook executions are
    complete.

> **Important:** The cluster must be Running before the Silver pipeline
> is triggered because the notebook activity is configured to use the
> existing All-Purpose interactive cluster. If the cluster is
> terminated, start it manually first. The same cluster is reused by the
> Gold pipeline.
