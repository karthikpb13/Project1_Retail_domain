
# Azure Databricks Compute and ADLS Gen2 Access Configuration

## 1. Creating Azure Databricks Compute

Before starting the data transformations, we need to create compute in Azure Databricks.

1. Open Azure Databricks.
2. Navigate to Compute and create new compute.
3. Provide a suitable name for the compute.
4. Select an LTS (Long-Term Support) Runtime.
5. Select the required node type.
6. Since this is a small project, select Single Node where supported.
7. Configure auto-termination after 20 minutes to reduce compute costs.
8. Since All-Purpose Compute is not working in this environment, we will use Serverless Compute.

---

## 2. Configure Databricks Access to Azure Data Lake Storage

Before starting transformations, we need to ensure that Databricks can access the data stored in Azure Data Lake Storage Gen2 (ADLS Gen2).

For this, we need to configure Authentication and Authorization.

- **Authentication:** Verifies the identity of the application requesting access.
- **Authorization:** Determines what the authenticated application is allowed to do.

We will configure access using a Service Principal.

### Service Principal

A Service Principal is an application identity that Azure services use to authenticate and access Azure resources.

The overall process involves three steps:

1. Register an application in Microsoft Entra ID.
2. Assign permission to that application to access the Azure Storage Account.
3. Configure Databricks to use the application credentials when accessing ADLS.

### Microsoft Entra ID

Microsoft Entra ID is Microsoft's cloud-based Identity and Access Management (IAM) service for Azure.

---

## 3. Creating an Application in Microsoft Entra ID

1. Open the Azure Portal.
2. Search for Microsoft Entra ID.
3. Navigate to Manage → App registrations.
4. Select New registration.
5. Provide a new application name.
6. Keep the remaining details as required and click Register.

After registration, the application overview page displays the following details:

| Field | Description |
|---|---|
| Application (Client) ID | Uniquely identifies the application. |
| Directory (Tenant) ID | Identifies the Microsoft Entra ID directory where the application is registered. |
| Object ID | Identifies the application object within the directory. |

### Application Details

- Application Name: `azure_de_prj1-rd-app`
- Application (Client) ID: `909d980a-f380-4bb0-a777-31fffa7a5bab`
- Directory (Tenant) ID: `448faffe-f8c6-41ae-84f5-746dd4242501`
- Object ID: `0cfd7e1c-e884-47b3-99ba-1efa9fa57ccd`

To authenticate our application, we also need a Client Secret.

---

## 4. Creating the Client Secret

1. Open the registered application.
2. Navigate to Certificates & Secrets.
3. Select New Client Secret.
4. Provide a description.
5. Select an appropriate expiration period.
6. Click Add.

Azure will provide two fields:

- **Value:** The actual client secret used for authentication.
- **Secret ID:** Identifies the client secret object.

The Client Secret Value acts like a password for our application identity.

### Important

- Copy the secret value immediately after creation because it cannot be viewed again.
- Store the value securely in Azure Key Vault or an approved secrets-management solution.
- Never expose the secret value in notebooks, GitHub repositories, or documentation.
- Use a Databricks secret scope to access the secret securely.

If a real client secret has been exposed, revoke or rotate it.

---

## 5. Granting the Application Access to the Storage Account

1. Open the Azure Storage Account of the project.
2. Navigate to Access Control (IAM).
3. Select Add → Add Role Assignment.
4. Search for Storage Blob Data Contributor.
5. Select the role and click Next.
6. Under Members, click Select Members.
7. Search for the application name:
   `azure_de_prj1-rd-app`
8. Select the application.
9. Click Select.
10. Click Review + Assign.

Azure will assign the required permissions to the application.

### ADLS Gen2 Permissions

For ADLS Gen2, access control depends on:

1. **Azure RBAC:** Provides role-based permissions at the storage account or container level.
2. **File System ACLs:** Provides directory-level and file-level permissions.

If we are using more restrictive permissions, we need to configure the required ACLs separately.

---

## 6. Configuring Databricks to Access ADLS Gen2

Open the Azure Databricks workspace.

Create separate notebooks for the transformations:

- `cleansed`
- `curated`

Now we need to configure Databricks so that it can access data from ADLS Gen2.

For this, we will use the application registered in Microsoft Entra ID.

Configuration reference:

https://learn.microsoft.com/en-us/azure/databricks/connect/storage/azure-storage

In the configuration code, we need to replace the following placeholders:

| Placeholder | Replacement |
|---|---|
| `<secret-scope>` | Databricks secret scope name |
| `<service-credential-key>` | Key containing the client secret |
| `<storage-account>` | Azure Storage Account name |
| `<application-id>` | Application (Client) ID |
| `<directory-id>` | Directory (Tenant) ID |

### Creating a Databricks Secret Scope

We need to add the client secret value to a Databricks secret scope so that it can be accessed securely.

1. Create a Databricks secret scope.
2. Add the client secret value under a key.
3. Reference the secret scope and key in the authentication configuration.
4. Avoid hardcoding the secret value in notebook code.

Reference:

https://learn.microsoft.com/en-us/azure/databricks/security/secrets/

---

## 7. Reading Data from ADLS Gen2

After configuring authentication and authorization, we can read the data stored in ADLS Gen2 using Spark.

### Reading Category Data

```python
df_category = (
    spark.read
        .option("header", True)
        .option("inferSchema", True)
        .csv(
            "abfss://bronze-deprj1-rd@kpbdeprj1rdstgacc.dfs.core.windows.net/category"
        )
)
```

### Understanding the ADLS Path

```text
abfss://<container>@<storage-account>.dfs.core.windows.net/<directory>
```

- Container: `bronze-deprj1-rd`
- Storage Account: `kpbdeprj1rdstgacc`
- Directory: `category`

### Display the Data

```python
display(df_category)
```

### Check Schema

```python
df_category.printSchema()
```

### Check Record Count

```python
df_category.count()
```

---

## 8. References

1. Azure Databricks - Connect to Azure Storage:
   https://learn.microsoft.com/en-us/azure/databricks/connect/storage/azure-storage

2. Databricks Secret Management:
   https://learn.microsoft.com/en-us/azure/databricks/security/secrets/

3. Microsoft Entra ID App Registration:
   https://learn.microsoft.com/en-us/entra/identity-platform/quickstart-register-app

4. Azure Storage Blob Data Contributor:
   https://learn.microsoft.com/en-us/azure/role-based-access-control/built-in-roles/storage#storage-blob-data-contributor
