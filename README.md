# Data Warehouse and Analytics Project

Welcome to the **Data Warehouse and Analytics Project** repository! 🚀  
This project demonstrates a comprehensive data warehousing and analytics solution, from building a data warehouse to generating actionable insights. Designed as a portfolio project, it highlights industry best practices in data engineering and analytics.

---
## 🏗️ Data Architecture

The data architecture for this project follows Medallion Architecture **Bronze**, **Silver**, and **Gold** layers:
![Data Architecture](docs/data_architecture.png)

1. **Bronze Layer**: Stores raw data as-is from source systems. Data is ingested from CSV files into SQL Server Database.
2. **Silver Layer**: Includes data cleansing, standardization, and normalization processes to prepare data for analysis.
3. **Gold Layer**: Houses business-ready data modeled into star schema for reporting and analytics.

---
## 📖 Project Overview

This project implements:

1. **Modern Data Warehouse** using Medallion Architecture
2. **ETL Pipelines** for data extraction, transformation and loading
3. **Star Schema Modeling** with optimized fact/dimension tables
4. **SQL Analytics** for customer behavior, product performance and sales trends

🎯 Ideal showcase for expertise in:
- SQL Development
- Data Architecture  
- ETL Pipeline Development  
- Dimensional Modeling  
- Analytical Reporting  

---

## 🛠️ Tools & Technologies

- **Database**: SQL Server Express
- **ETL**: SQL Scripts (T-SQL)
- **Data Modeling**: Draw.io
- **Version Control**: Git/GitHub
- **Analytics**: SQL Reporting

---

---

## 🛠️ Important Links & Tools:

Everything is for Free!
- **[Datasets](datasets/):** Access to the project dataset (csv files).
- **[SQL Server Express](https://www.microsoft.com/en-us/sql-server/sql-server-downloads):** Lightweight server for hosting your SQL database.
- **[SQL Server Management Studio (SSMS)](https://learn.microsoft.com/en-us/sql/ssms/download-sql-server-management-studio-ssms?view=sql-server-ver16):** GUI for managing and interacting with databases.
- **[Git Repository](https://github.com/):** Set up a GitHub account and repository to manage, version, and collaborate on your code efficiently.
- **[DrawIO](https://www.drawio.com/):** Design data architecture, models, flows, and diagrams.
- **[Notion](https://www.notion.com/templates/sql-data-warehouse-project):** Get the Project Template from Notion
- **[Notion Project Steps](https://thankful-pangolin-2ca.notion.site/SQL-Data-Warehouse-Project-16ed041640ef80489667cfe2f380b269?pvs=4):** Access to All Project Phases and Tasks.

---

## 📂 Repository Structure
```
data-warehouse-project/
│
├── datasets/              # Source CSV datasets (ERP + CRM)
│
├── docs/                  # Documentation
│   ├── data_models.drawio # Star schema diagrams
│   ├── etl_design.drawio  # ETL workflow diagrams
│   └── requirements.md    # Detailed specifications
│
├── scripts/               # SQL implementation
│   ├── bronze/            # Raw data ingestion
│   ├── silver/            # Data cleansing 
│   └── gold/              # Star schema modeling
│
├── tests/                 # Data quality checks
│
└── README.md              # Project overview
```

---
## 🚀 Key Implementation Steps

### 1. Data Ingestion (Bronze Layer)
```sql
-- Example: Load raw CRM data
CREATE TABLE bronze.crm_raw (
    CustomerID INT,
    Name NVARCHAR(100),
    SignupDate DATE,
    ...
);
BULK INSERT bronze.crm_raw
FROM 'datasets/crm_data.csv'
WITH (FORMAT = 'CSV');
```

### 2. Data Cleansing (Silver Layer)
```sql
-- Example: Standardize customer data
CREATE TABLE silver.customers AS
SELECT 
    CustomerID,
    TRIM(UPPER(FirstName)) + ' ' + TRIM(UPPER(LastName)) AS FullName,
    CASE WHEN ISDATE(SignupDate) = 1 
         THEN CAST(SignupDate AS DATE) 
         ELSE NULL END AS SignupDate,
    ...
FROM bronze.crm_raw
WHERE CustomerID IS NOT NULL;
```

### 3. Dimensional Modeling (Gold Layer)
```sql
-- Example: Create fact_sales table
CREATE TABLE gold.fact_sales (
    SaleID INT IDENTITY(1,1) PRIMARY KEY,
    DateKey INT REFERENCES gold.dim_date(DateKey),
    CustomerKey INT REFERENCES gold.dim_customer(CustomerKey),
    ProductKey INT REFERENCES gold.dim_product(ProductKey),
    Quantity INT,
    Amount DECIMAL(10,2)
);

-- Populate with transformed data
INSERT INTO gold.fact_sales (...)
SELECT ... FROM silver.sales;
```

### 4. Analytical Queries
```sql
-- Top performing products
SELECT 
    p.ProductName,
    SUM(f.Amount) AS TotalRevenue
FROM gold.fact_sales f
JOIN gold.dim_product p ON f.ProductKey = p.ProductKey
GROUP BY p.ProductName
ORDER BY TotalRevenue DESC
```

---
## 📊 Sample Analytics Output

### Customer Lifetime Value Analysis
| Customer Tier | Avg. Orders | Avg. Revenue |
|---------------|-------------|--------------|
| Platinum      | 15.2        | $8,750       |
| Gold          | 7.8         | $2,300       |
| Silver        | 3.2         | $650         |

### Monthly Sales Trend
```mermaid
graph LR
    Jan[January] -->|$12,450| Feb[February]
    Feb -->|$18,200| Mar[March]
    Mar -->|$22,780| April[April]
    April -->|$19,650| May[May]
```

---
## 💡 Learning Resources

1. [Medallion Architecture Guide](https://www.databricks.com/glossary/medallion-architecture)
2. [Star Schema Best Practices](https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/star-schema/)
3. [SQL Server ETL Techniques](https://learn.microsoft.com/en-us/sql/integration-services/ssis-etl)

---
## 📜 License
This project is licensed under the [MIT License](LICENSE). # Data_Warehouse_principle
# Data_Warehouse_principle
