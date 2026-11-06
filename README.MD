# Retail Data Engineering Project – Electronic Stores

## Business Scenario

A company sells electronic products across multiple stores and online channels.

1. **Online Stores (E-Commerce):** Online product sales and customer transactions.
2. **Store Operations (POS):** Sales transactions from physical retail stores.
3. **Product Management (Inventory System):** Product details and inventory management.
4. **Warranty Services:** Warranty registrations, claims, and service records.

## Data Formats

Data is stored in multiple systems and different formats.

- **CSV Files:** Sales, Products, Stores, Categories, Inventory.
- **Databases:** ERP, CRM, Operational Systems.
- **SaaS Applications:** CRM, E-Commerce.
- **Files / Logs:** System Logs, Event Data.
- **Web APIs:** External Data.

## Common Data Issues

Source data is not always analytics-ready.

- Duplicate records.
- Missing values.
- Inconsistent column names.
- Different date formats (DD/MM/YYYY, MM-DD-YYYY).
- Inconsistent data types.
- Invalid or incorrect data.

## Business Requirements

- How much revenue are we generating?
- Which products sell the most?
- Which stores perform better?
- How are sales changing over time?
- Which products have warranty claims?
- Which products are frequently out of stock?

## What We Will Do

- Build and maintain data pipelines.
- Ingest data from multiple source systems.
- Store data in a scalable data lake.
- Clean, transform, and enrich data.
- Perform data quality checks.
- Make data available for analytics and business use.

## How We Will Do It

### Architecture of the Project

```mermaid
flowchart LR

    subgraph SOURCE["Source Systems - Electronic Retail"]
        A["Categories"]
        B["Products"]
        C["Sales / POS"]
        D["Stores"]
        E["Inventory"]
        F["Warranty Claims"]
        G["E-Commerce"]
    end

    subgraph LAKE["Azure Data Lake Storage Gen2"]
        H["Bronze Layer<br/>Raw Data"]
        I["Silver Layer<br/>Cleansed & Standardized Data"]
        J["Gold Layer<br/>Business Transformed Data"]
    end

    K["Azure Data Factory<br/>ADF"]
    L["Databricks<br/>Data Transformation"]
    M[("Snowflake<br/>Data Warehouse")]

    A --> K
    B --> K
    C --> K
    D --> K
    E --> K
    F --> K
    G --> K

    K --> H
    H --> L
    L --> I
    I --> L
    L --> J
    J --> K
    K --> M

    style H fill:#CD7F32,color:#fff,stroke:#8B4513
    style I fill:#A0A0A0,color:#fff,stroke:#555
    style J fill:#D4AF37,color:#000,stroke:#997A00
    style M fill:#29B5E8,color:#fff,stroke:#147A9C
    style L fill:#E36C09,color:#fff,stroke:#A34B00
    style K fill:#2874A6,color:#fff,stroke:#154360
```

### Data Flow

`Source Systems → ADF → ADLS Gen2 Bronze → Databricks → ADLS Gen2 Silver → Databricks → ADLS Gen2 Gold → ADF → Snowflake`