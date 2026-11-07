# Configure Azure Data Factory to Run an Azure Databricks Notebook

## 1. Overview

Azure Data Factory (ADF) can orchestrate the execution of notebooks in
Azure Databricks.

For this retail data project, create two separate ADF pipelines:

-   `PL_Silver_Transformation` --- runs the Silver-layer notebook.
-   `PL_Gold_Transformation` --- runs the Gold-layer notebook.

Both pipelines can reuse the same Azure Databricks Linked Service,
provided they use the same workspace, authentication configuration, and
compatible compute.

## 2. Prerequisites

Before creating the pipelines, make sure that:

-   The Azure Databricks workspace is available.
-   The Databricks all-purpose cluster is available and running when
    required.
-   The user or service identity used by ADF has permission to access
    the workspace and use the cluster.
-   The Silver and Gold notebooks exist in the Databricks workspace.
-   The required storage access is already configured in Databricks.

> **Security:** Do not paste access tokens or client secrets into
> project documentation, screenshots, notebooks, or source control.
> Store credentials securely and rotate any credential that has been
> exposed.

## 3. Create the Azure Databricks Linked Service

1.  Open the Azure Data Factory resource in Azure Portal.
2.  Select **Launch Studio**.
3.  In ADF Studio, open **Manage**.
4.  Select **Linked Services**.
5.  Select **+ New**.
6.  Search for **Azure Databricks**, select it, and continue.
7.  Configure the connection:

  Property               Value for this project
  ---------------------- ---------------------------------------------
  Name                   `LS_AzureDatabricks`
  Azure subscription     Select the project subscription
  Databricks workspace   Select the project workspace
  Select cluster         Existing interactive cluster
  Workspace URL          Select/populate the project workspace URL
  Authentication type    Access Token (as configured for this setup)
  Existing cluster ID    Select the project all-purpose cluster

8.  Generate a Databricks personal access token (PAT), if using Access
    Token authentication:
    -   Open the Databricks workspace.
    -   Open user settings.
    -   Find **Developer** / **Access tokens**.
    -   Generate a token according to the organization's security
        policy.
    -   Copy it securely and paste it into the ADF Access token field.
9.  After entering the token, open **Existing cluster ID** and select
    the intended cluster.
10. Select **Test connection**.
11. If the test succeeds, select **Create**.

**Project cluster:** `kpb-de-prj1-rd-adf-compute`

**Important:** ADF must be able to authenticate and the identity must
have permission to use the selected cluster. Keep the cluster available
for the initial test. Cluster state, auto-termination, permissions, and
the integration runtime can affect execution.

## 4. Create the Silver pipeline

1.  Open **Author** in ADF Studio.

2.  Select **+** and choose **Pipeline**.

3.  Rename the pipeline to:

    `PL_Silver_Transformation`

4.  In the Activities pane, search for **Databricks**.

5.  Drag the **Notebook** activity onto the pipeline canvas.

6.  Select the activity and open the **General** tab.

7.  Set the activity name to:

    `Execute_Silver_Notebook`

## 5. Configure the Silver Notebook activity

1.  Open the **Azure Databricks** tab.

2.  Select the linked service:

    `LS_AzureDatabricks`

3.  Open the **Settings** tab.

4.  For **Notebook path**, select **Browse**.

5.  Navigate through the Databricks workspace folders and select the
    actual Silver notebook.

6.  Confirm that the selected notebook is the Silver transformation
    notebook.

7.  Leave **Base parameters** empty unless the notebook requires
    parameters.

8.  Leave **Append libraries** empty unless additional libraries are
    required by the notebook.

> Prefer selecting the notebook through **Browse** so that ADF uses the
> workspace path it can resolve. Do not paste the browser URL. If
> entering the path manually, use the notebook workspace path expected
> by the ADF activity; do not assume that a browser URL or a local
> `.ipynb` file path is valid.

## 6. Validate and run the Silver pipeline

1.  Select **Validate** in ADF Studio.
2.  Resolve any validation errors.
3.  Select **Publish All**.
4.  Select **Add Trigger** → **Trigger Now**.
5.  Confirm the trigger.
6.  Open **Monitor** → **Pipeline runs**.
7.  Open the pipeline run and inspect the Notebook activity status and
    output.
8.  Confirm that the Silver notebook completed and wrote its expected
    output to the Silver storage location.

Do not start the Gold pipeline until the Silver output is ready.

## 7. Create the separate Gold pipeline

After the Silver pipeline has been created and tested:

1.  Return to **Author**.

2.  Create another pipeline.

3.  Rename it to:

    `PL_Gold_Transformation`

4.  Add a Databricks **Notebook** activity.

5.  Set its activity name to:

    `Execute_Gold_Notebook`

6.  In the **Azure Databricks** tab, select the same linked service:

    `LS_AzureDatabricks`

7.  In **Settings** → **Notebook path**, select the Gold notebook using
    **Browse**.

8.  Leave Base parameters and Append libraries empty unless required by
    the notebook.

9.  Validate, publish, and test the Gold pipeline.

## 8. Project pipeline summary

  -------------------------------------------------------------------------------
  Component               Silver pipeline              Gold pipeline
  ----------------------- ---------------------------- --------------------------
  Pipeline                `PL_Silver_Transformation`   `PL_Gold_Transformation`

  Activity                `Execute_Silver_Notebook`    `Execute_Gold_Notebook`

  Activity type           Databricks Notebook          Databricks Notebook

  Linked Service          `LS_AzureDatabricks`         `LS_AzureDatabricks`

  Notebook                Silver transformation        Gold transformation
                          notebook                     notebook

  Input layer             Bronze                       Silver

  Output layer            Silver                       Gold
  -------------------------------------------------------------------------------

### Execution order

For initial testing, run the pipelines independently in this order:

1.  Run `PL_Silver_Transformation`.
2.  Confirm its run succeeds and Silver data is available.
3.  Run `PL_Gold_Transformation`.
4.  Confirm its run succeeds and the Gold output datasets are written.

Later, a separate master pipeline can use **Execute Pipeline**
activities to call the Silver pipeline first and the Gold pipeline only
after Silver succeeds.

## 9. Do ADF datasets need to be created?

For this design, the Databricks **Notebook** activity uses the
Databricks Linked Service and notebook path. It does not require an ADF
dataset.

ADF datasets are needed when activities such as **Copy Data** use
dataset-based source or sink definitions. They are not required just to
invoke these two notebooks.

## 10. Troubleshooting checklist

  -----------------------------------------------------------------------
  Symptom                             Checks
  ----------------------------------- -----------------------------------
  Notebook resource not found         Use Browse; confirm the notebook
                                      exists in the selected workspace
                                      and folder; verify the selected
                                      path.

  Cluster is not listed               Confirm the token is entered; check
                                      workspace access and cluster
                                      permissions; refresh the cluster
                                      list.

  Authentication failure              Verify the token is valid, not
                                      expired or revoked, and belongs to
                                      an authorized identity.

  Cluster permission error            Confirm the identity has permission
                                      to use the selected all-purpose
                                      cluster.

  Notebook execution fails            Open the ADF activity output and
                                      the Databricks run details; inspect
                                      the notebook error.

  Storage access error                Check the Databricks storage
                                      authentication and Unity Catalog
                                      permissions/external locations used
                                      by the notebook.
  -----------------------------------------------------------------------

## 11. Final checklist

-   [ ] `LS_AzureDatabricks` created and connection test succeeded.
-   [ ] Existing all-purpose cluster selected.
-   [ ] `PL_Silver_Transformation` created.
-   [ ] Silver Notebook activity configured and notebook path verified.
-   [ ] Silver pipeline published and run successfully.
-   [ ] `PL_Gold_Transformation` created separately.
-   [ ] Gold Notebook activity configured and notebook path verified.
-   [ ] Gold pipeline published and run after Silver completed.
-   [ ] Gold outputs verified in the Gold storage location.
-   [ ] No secrets or access tokens committed to the project repository.
