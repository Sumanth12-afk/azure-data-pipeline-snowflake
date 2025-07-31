
# Azure Databricks to Snowflake ETL Project - Full Setup Guide

This README documents the **entire end-to-end setup** of a cloud data pipeline using **Azure Data Lake Storage Gen2**, **Azure Databricks**, and **Snowflake**, including configuration, authentication, transformation, and visualization.

---

## 🚀 Project Summary

**Objective**: Load a raw CSV file into Azure Data Lake, clean and transform it with PySpark in Databricks, push the processed data to Snowflake, and visualize it with a bar chart.

---

## ✅ 1. Azure Setup

### 🔹 Step 1: Create Azure Account and Resource Group
- Go to [portal.azure.com](https://portal.azure.com)
- Create an account
- Create a Resource Group: `RG_Databricks`

### 🔹 Step 2: Create Azure Storage Account
- Name: `databrickssumanth`
- Enable **Hierarchical Namespace** ✅ (required for ADLS Gen2)

### 🔹 Step 3: Create Container in Storage
- Navigate to `Containers`
- Create container: `sparksql`
- Inside `sparksql`, manually create folder structure:
  - `source/raw_data/`
  - `curated/orders_cleaned/`

### 🔹 Step 4: Upload Raw File
- Upload `ecommerce_orders_large.csv` to `source/raw_data/`

### 🔹 Step 5: Assign IAM Role
- Role: `Storage Blob Data Contributor`
- Assign to: **Databricks Access Connector** (created in next section)

---

## ✅ 2. Databricks Setup

### 🔹 Step 6: Create Azure Databricks Workspace
- Name: `databricks-sumanth`
- Region: UK South

### 🔹 Step 7: Create Access Connector
- Resource: `databricks_uksouth_connector`
- Location: Same as workspace (UK South)
- Role Assignment: IAM ➝ Assign to Storage Account with `Storage Blob Data Contributor`

### 🔹 Step 8: Launch Databricks Workspace
- Open your workspace UI
- Create a folder (e.g. `sparkSQL`) to hold notebooks

### 🔹 Step 9: Create Notebook
- Name: `ETL_ADLS_to_Snowflake`
- Language: Python (PySpark)

---

## ✅ 3. Data Processing in Databricks

### 🔹 Step 10: Read CSV from ADLS
```python
path = "abfss://sparksql@databrickssumanth.dfs.core.windows.net/source/raw_data/ecommerce_orders_large.csv"
df = spark.read.option("header", True).option("inferSchema", True).csv(path)
```

### 🔹 Step 11: Clean & Transform
```python
from pyspark.sql.functions import col, expr
cleaned_df = df.withColumn("order_date", col("order_date").cast("date"))                .withColumn("quantity", col("quantity").cast("int"))                .withColumn("price_per_unit", col("price_per_unit").cast("double"))                .withColumn("total_price", expr("quantity * price_per_unit"))                .dropna()
```

### 🔹 Step 12: Save Transformed Data to ADLS (CSV)
```python
cleaned_df.write.mode("overwrite").option("header", True).csv(
    "abfss://sparksql@databrickssumanth.dfs.core.windows.net/curated/snowflake_upload/")
```

---

## ✅ 4. Snowflake Setup

### 🔹 Step 13: Login to Snowflake
- URL: `https://MUWDJRR-WH22089.snowflakecomputing.com`
- Role: `ACCOUNTADMIN`
- Warehouse: `COMPUTE_WH`

### 🔹 Step 14: Create Database & Schema
```sql
CREATE DATABASE IF NOT EXISTS ECOMMERCE_DB;
USE DATABASE ECOMMERCE_DB;
CREATE SCHEMA IF NOT EXISTS DELIVERED_ORDERS;
```

### 🔹 Step 15: Upload File and Auto-Create Table
- Download the CSV from ADLS manually
- In Snowflake Web UI:
  - Navigate to `ECOMMERCE_DB.DELIVERED_ORDERS`
  - Click **"+ Table → Load Data"**
  - Upload CSV and let Snowflake infer the schema
  - Table name: `ORDERS_DELIVERED_CLEANED`

> ⚠️ If you're pushing data using `spark.write.format("snowflake")`, you must configure the Snowflake connection:

```python
sfOptions = {
  "sfURL": "<your_snowflake_url>",
  "sfDatabase": "ECOMMERCE_DB",
  "sfSchema": "DELIVERED_ORDERS",
  "sfWarehouse": "COMPUTE_WH",
  "sfRole": "ACCOUNTADMIN",
  "sfUser": "<your_snowflake_username>",
  "sfPassword": "<your_snowflake_password>"  # 🔒 Store this using a secret manager in production
}
```

---

## ✅ 5. Visualization

### 🔹 Step 16: Query Data
```sql
SELECT product_category, SUM(total_price) AS total_revenue
FROM ORDERS_DELIVERED_CLEANED
GROUP BY product_category
ORDER BY total_revenue DESC;
```

### 🔹 Step 17: Build Chart in Snowflake
- Click "Chart" tab
- Set:
  - **X-axis**: `product_category`
  - **Y-axis**: `total_revenue`
  - Chart type: `Bar`

✅ You now have a real-time revenue by category chart built from a fully governed cloud ETL pipeline.

---

## 🧱 Technologies Used
- **Azure Data Lake Storage Gen2**
- **Azure Databricks (PySpark)**
- **Azure IAM / Access Connector**
- **Snowflake** (Enterprise Edition)
- **Snowflake Chart Builder**

---

## 📦 Result
- 1000 orders processed
- 1 Snowflake table created with cleaned data
- Live bar chart of revenue by product category
- Fully real-world cloud-native ETL project ✅

---
## 📊 ETL Pipeline Architecture: Azure to Snowflake
  <img width="700" height="1000" alt="a0601aeb-4d09-4ec7-9185-b35571982d56" src="https://github.com/user-attachments/assets/4ae84d19-c3ed-4f53-b0ec-2a4dab5ff1ea" />

