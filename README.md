# ETL Project: End-to-end Sales Data Analysis

## Overview
This project demonstrates an **end-to-end ETL (Extract, Transform, Load) pipeline** for sales data analysis. The data is extracted from **Kaggle API**, cleaned using **pandas**, and loaded into **SQL Server** for further analysis. Several SQL queries are then used to derive insights, including revenue analysis, sales growth trends, and category-wise performance.

## Steps Implemented

### 1. **Extract Data**
- Extracted sales dataset from **Kaggle API**.
- Loaded the dataset into **Jupyter Notebook** for preprocessing.

### 2. **Transform Data**
- Read data from the extracted file and handled **null values**.
- Renamed columns for consistency.
- Derived new columns: **discount, sale price, and profit**.
- Converted `order_date` from object data type to **datetime**.
- Dropped unnecessary columns: **cost price list** and **discount percent**.

### 3. **Load Data into SQL Server**
- Connected to **SQL Server** using `sqlalchemy`.
- Used **replace** and **append** options to insert data into the database.

```python
import sqlalchemy as sal
engine = sal.create_engine('mssql://sreeman/master?driver=ODBC+DRIVER+17+FOR+SQL+SERVER')
conn = engine.connect()
df.to_sql('df_orders', con=conn, index=False, if_exists='append')
```

## SQL Analysis Performed

### **1. Fetch All Data**
```sql
SELECT * FROM df_orders;
```

### **2. Finding Top 10 Highest Revenue-Generating Products**
```sql
SELECT TOP 10 product_id, SUM(sale_price) AS sales
FROM df_orders
GROUP BY product_id
ORDER BY sales DESC;
```

### **3. Finding Top 5 Highest Selling Products in Each Region**
```sql
WITH cte AS (
    SELECT region, product_id, SUM(sale_price) AS sales
    FROM df_orders
    GROUP BY region, product_id
)
SELECT * FROM (
    SELECT *, ROW_NUMBER() OVER (PARTITION BY region ORDER BY sales DESC) AS rn
    FROM cte
) A
WHERE rn <= 5;
```

### **4. Month-over-Month Growth Comparison (2022 vs 2023 Sales)**
```sql
WITH cte AS (
    SELECT YEAR(order_date) AS order_year, MONTH(order_date) AS order_month,
    SUM(sale_price) AS sales
    FROM df_orders
    GROUP BY YEAR(order_date), MONTH(order_date)
)
SELECT order_month,
    SUM(CASE WHEN order_year=2022 THEN sales ELSE 0 END) AS sales_2022,
    SUM(CASE WHEN order_year=2023 THEN sales ELSE 0 END) AS sales_2023
FROM cte
GROUP BY order_month
ORDER BY order_month;
```

### **5. Finding the Best Month for Each Category by Sales**
```sql
WITH cte AS (
    SELECT category, FORMAT(order_date, 'yyyyMM') AS order_year_month,
    SUM(sale_price) AS sales
    FROM df_orders
    GROUP BY category, FORMAT(order_date, 'yyyyMM')
)
SELECT * FROM (
    SELECT *, ROW_NUMBER() OVER (PARTITION BY category ORDER BY sales DESC) AS rn
    FROM cte
) a
WHERE rn = 1;
```

### **6. Subcategory with the Highest Growth in Profit (2023 vs 2022)**
```sql
WITH cte AS (
    SELECT sub_category, YEAR(order_date) AS order_year,
    SUM(sale_price) AS sales
    FROM df_orders
    GROUP BY sub_category, YEAR(order_date)
),
cte2 AS (
    SELECT sub_category,
        SUM(CASE WHEN order_year=2022 THEN sales ELSE 0 END) AS sales_2022,
        SUM(CASE WHEN order_year=2023 THEN sales ELSE 0 END) AS sales_2023
    FROM cte
    GROUP BY sub_category
)
SELECT TOP 1 *,
    (sales_2023 - sales_2022) * 100 / sales_2022 AS growth_percentage
FROM cte2
ORDER BY growth_percentage DESC;
```

## Technologies Used
- **Python** (Jupyter Notebook, pandas, sqlalchemy)
- **SQL Server** (Data Storage & Querying)
- **Kaggle API** (Data Extraction)

## Repository Structure
```
📂 ETL-Project
│-- 📜 etl_script.py        # Python script for ETL pipeline
│-- 📜 sql_queries.sql      # SQL queries for data analysis
│-- 📜 requirements.txt     # Dependencies list
│-- 📜 README.md            # Project documentation (this file)
```

## How to Run the Project
1. Clone the repository:
   ```sh
   git clone https://github.com/yourusername/ETL-Project.git
   cd ETL-Project
   ```
2. Install dependencies:
   ```sh
   pip install -r requirements.txt
   ```
3. Run the ETL script:
   ```sh
   python etl_script.py
   ```
4. Run SQL queries using **SQL Server Management Studio (SSMS)**.

## Future Enhancements
- Implement **Automated Data Pipeline** using **Apache Airflow**.
- Add **Data Visualization** using Power BI/Tableau.
- Improve **Performance Optimization** in SQL Queries.

---
### **Author:** [Shresta Kota](https://github.com/shrestakota)
### **License:** MIT

# ETL-End-To-End-Data-Analysis
