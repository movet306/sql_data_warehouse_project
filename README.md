# SQL Data Warehouse Project

This repository documents the process of building a layered SQL data warehouse using CRM and ERP data sources.

The project focuses on the practical work behind data systems:

- understanding raw operational datasets  
- cleaning inconsistent records  
- integrating multiple source systems  
- exposing a clear analytical model for querying  

All transformations are written in SQL Server and organized into three layers:

**Bronze → Silver → Gold**

---

# Architecture

The warehouse follows a layered structure separating raw ingestion, data preparation, and analytical modeling.

![Architecture](docs/high_level_architecture.png)

Data moves through the system as follows:

1. Raw operational data is loaded into Bronze tables  
2. Silver layer applies cleaning and standardization  
3. Gold layer exposes an analytical model ready for querying  

---

# Source Systems

Two operational systems are used in this project.

## CRM

The CRM system contains transactional and core master data.

Tables:

- `crm_sales_details`
- `crm_cust_info`
- `crm_prd_info`

These tables describe sales activity, customer records, and product information.

---

## ERP

The ERP system provides additional attributes and reference data.

Tables:

- `erp_cust_az12`
- `erp_loc_a101`
- `erp_px_cat_g1v2`

These tables contain:

- product category information  
- customer location data  
- additional customer attributes  

The CRM and ERP systems overlap on several entities, which makes data integration necessary.

---

# Understanding the Data

Before building the warehouse layers, relationships between the tables were mapped.

![Integration](docs/data_integration.png)

Key findings during exploration:

- CRM contains transactional sales records  
- CRM maintains product history information  
- ERP stores product category hierarchies  
- ERP includes additional customer attributes  
- ERP stores customer location information  

These relationships drive how the Silver and Gold layers are built.

---

# Project Workflow

Project planning and task tracking were organized in Notion.

![Notion Workflow](docs/notion_project.png)

Major project stages:

1. Architecture design  
2. Project initialization  
3. Bronze layer ingestion  
4. Silver layer transformations  
5. Gold layer modeling  
6. Data validation and documentation  

This structure ensured the work progressed systematically rather than as disconnected SQL scripts.

---

# Bronze Layer

The Bronze layer stores the raw source data.

Purpose:

- preserve original data exactly as received  
- provide traceability for debugging transformations  
- maintain a consistent ingestion layer  

Characteristics:

- tables mirror the source structure  
- no transformations are applied  
- data is loaded using full refresh (truncate + insert)

Bronze tables correspond directly to CRM and ERP datasets.

At this stage the goal is **data preservation, not data quality.**

---

# Silver Layer

The Silver layer prepares the data for analytical use.

Here the raw operational data begins to become usable.

Main responsibilities of this layer include:

### Data Cleaning

Several inconsistencies existed in the raw data.

Examples addressed during transformation:

- whitespace trimming  
- invalid date handling  
- missing value handling  
- inconsistent text formatting  

---

### Standardization

Operational systems often encode attributes differently.

Examples standardized in this layer:

- gender values  
- marital status codes  
- country names  

Standardization ensures downstream queries behave consistently.

---

### Product History Handling

Products can appear multiple times due to historical records.

Window functions were used to determine the active product record by analyzing product start dates and deriving end dates.

Only the relevant product version is exposed to the analytical layer.

---

### Sales Data Validation

Sales records occasionally contained incorrect totals.

Where inconsistencies were detected, totals were recomputed using quantity and price to ensure reliable measures.

---

### Data Enrichment

Additional attributes were introduced using ERP tables.

Examples include:

- product categories  
- customer birthdate  
- customer location  

These enrichments allow the final model to support analytical queries without repeatedly joining operational tables.

---

# Gold Layer

The Gold layer exposes the data in a structure designed for analysis.

Instead of mirroring operational systems, the data is reorganized into a dimensional model.

Objects created:

- `dim_customers`
- `dim_products`
- `fact_sales`

These objects are implemented as **views** so the analytical layer always reflects the latest cleaned data from the Silver layer.

---

# Dimensional Model

The final analytical structure follows a star schema.

![Star Schema](docs/star_schema.png)

---

## Fact Table

**fact_sales**

This table represents transactional sales events.

Measures include:

- sales amount  
- quantity  
- price  

Each record references both customer and product dimensions.

---

## Customer Dimension

**dim_customers**

Built primarily from CRM data and enriched with ERP attributes.

Attributes include:

- customer identifiers  
- name information  
- marital status  
- gender  
- birthdate  
- country  

Where the same attribute existed in both systems, clear precedence rules were applied.

For example, CRM values were treated as authoritative for gender when available.

---

## Product Dimension

**dim_products**

Constructed by combining CRM product records with ERP category data.

Attributes include:

- product identifiers  
- product name  
- category and subcategory  
- product line  
- maintenance flag  
- cost  

Historical product records are filtered so that only the relevant version is exposed to the analytical layer.

---

# Data Validation

Before finalizing the model, several validation checks were performed.

Examples include:

- detecting duplicate rows after joins  
- verifying fact records map correctly to dimension keys  
- confirming transformations produce consistent results  

Validation queries are available in the `tests/` directory.

---

# Repository Structure
# SQL Data Warehouse Project

This repository documents the process of building a layered SQL data warehouse using CRM and ERP data sources.

The project focuses on the practical work behind data systems:

- understanding raw operational datasets  
- cleaning inconsistent records  
- integrating multiple source systems  
- exposing a clear analytical model for querying  

All transformations are written in SQL Server and organized into three layers:

**Bronze → Silver → Gold**

---

# Architecture

The warehouse follows a layered structure separating raw ingestion, data preparation, and analytical modeling.

![Architecture](docs/high_level_architecture.png)

Data moves through the system as follows:

1. Raw operational data is loaded into Bronze tables  
2. Silver layer applies cleaning and standardization  
3. Gold layer exposes an analytical model ready for querying  

---

# Source Systems

Two operational systems are used in this project.

## CRM

The CRM system contains transactional and core master data.

Tables:

- `crm_sales_details`
- `crm_cust_info`
- `crm_prd_info`

These tables describe sales activity, customer records, and product information.

---

## ERP

The ERP system provides additional attributes and reference data.

Tables:

- `erp_cust_az12`
- `erp_loc_a101`
- `erp_px_cat_g1v2`

These tables contain:

- product category information  
- customer location data  
- additional customer attributes  

The CRM and ERP systems overlap on several entities, which makes data integration necessary.

---

# Understanding the Data

Before building the warehouse layers, relationships between the tables were mapped.

![Integration](docs/data_integration.png)

Key findings during exploration:

- CRM contains transactional sales records  
- CRM maintains product history information  
- ERP stores product category hierarchies  
- ERP includes additional customer attributes  
- ERP stores customer location information  

These relationships drive how the Silver and Gold layers are built.

---

# Project Workflow

Project planning and task tracking were organized in Notion.

![Notion Workflow](docs/notion_project.png)

Major project stages:

1. Architecture design  
2. Project initialization  
3. Bronze layer ingestion  
4. Silver layer transformations  
5. Gold layer modeling  
6. Data validation and documentation  

This structure ensured the work progressed systematically rather than as disconnected SQL scripts.

---

# Bronze Layer

The Bronze layer stores the raw source data.

Purpose:

- preserve original data exactly as received  
- provide traceability for debugging transformations  
- maintain a consistent ingestion layer  

Characteristics:

- tables mirror the source structure  
- no transformations are applied  
- data is loaded using full refresh (truncate + insert)

Bronze tables correspond directly to CRM and ERP datasets.

At this stage the goal is **data preservation, not data quality.**

---

# Silver Layer

The Silver layer prepares the data for analytical use.

Here the raw operational data begins to become usable.

Main responsibilities of this layer include:

### Data Cleaning

Several inconsistencies existed in the raw data.

Examples addressed during transformation:

- whitespace trimming  
- invalid date handling  
- missing value handling  
- inconsistent text formatting  

---

### Standardization

Operational systems often encode attributes differently.

Examples standardized in this layer:

- gender values  
- marital status codes  
- country names  

Standardization ensures downstream queries behave consistently.

---

### Product History Handling

Products can appear multiple times due to historical records.

Window functions were used to determine the active product record by analyzing product start dates and deriving end dates.

Only the relevant product version is exposed to the analytical layer.

---

### Sales Data Validation

Sales records occasionally contained incorrect totals.

Where inconsistencies were detected, totals were recomputed using quantity and price to ensure reliable measures.

---

### Data Enrichment

Additional attributes were introduced using ERP tables.

Examples include:

- product categories  
- customer birthdate  
- customer location  

These enrichments allow the final model to support analytical queries without repeatedly joining operational tables.

---

# Gold Layer

The Gold layer exposes the data in a structure designed for analysis.

Instead of mirroring operational systems, the data is reorganized into a dimensional model.

Objects created:

- `dim_customers`
- `dim_products`
- `fact_sales`

These objects are implemented as **views** so the analytical layer always reflects the latest cleaned data from the Silver layer.

---

# Dimensional Model

The final analytical structure follows a star schema.

![Star Schema](docs/star_schema.png)

---

## Fact Table

**fact_sales**

This table represents transactional sales events.

Measures include:

- sales amount  
- quantity  
- price  

Each record references both customer and product dimensions.

---

## Customer Dimension

**dim_customers**

Built primarily from CRM data and enriched with ERP attributes.

Attributes include:

- customer identifiers  
- name information  
- marital status  
- gender  
- birthdate  
- country  

Where the same attribute existed in both systems, clear precedence rules were applied.

For example, CRM values were treated as authoritative for gender when available.

---

## Product Dimension

**dim_products**

Constructed by combining CRM product records with ERP category data.

Attributes include:

- product identifiers  
- product name  
- category and subcategory  
- product line  
- maintenance flag  
- cost  

Historical product records are filtered so that only the relevant version is exposed to the analytical layer.

---

# Data Validation

Before finalizing the model, several validation checks were performed.

Examples include:

- detecting duplicate rows after joins  
- verifying fact records map correctly to dimension keys  
- confirming transformations produce consistent results  

Validation queries are available in the `tests/` directory.

---

# Repository Structure
datasets/
raw CSV files used as source data

docs/
architecture diagrams
data integration diagrams
star schema model

scripts/
bronze layer ingestion
silver layer transformations
gold layer modeling

tests/
data validation queries


---

# Tools Used

- SQL Server  
- T-SQL  
- SQL Server Management Studio  
- Draw.io (architecture & data modeling)  
- Notion (project planning and task tracking)  
- Git & GitHub  

---

# Author

**Mert Övet**

GitHub  
https://github.com/movet306

LinkedIn  
https://www.linkedin.com/in/mertovet

---

This project documents the process of designing and implementing a layered SQL data warehouse, from raw source ingestion to an analytical star schema model.
