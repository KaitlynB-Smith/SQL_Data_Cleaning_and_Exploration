# SQL_Data_Cleaning_and_Exploration

The purpose of this SQL project is to clean and explore retail sales data, identifying patterns and trends in customer purchasing behavior. This project focuses on data cleaning techniques and initial exploration of the sales trends.

---

# **Data Overview**
The dataset contains sales data for each transaction of a retail store between 2022 - 2025. The data consists of information about the customers, items, payment methods, locations, dates, and discounts for each transaction. 

## **Data Source**
This dataset was created by [Ahmed Mohamed](https://www.kaggle.com/ahmedmohamed2003) on Kaggle and can be found [here](https://www.kaggle.com/datasets/ahmedmohamed2003/retail-store-sales-dirty-for-data-cleaning?select=retail_store_sales.csv).

## **Column Descriptions**
The dataset consists of 11 unique columns and 12,575 fields of transaction data.
  - **Transaction_ID**: Unique identifier for each transaction
  - **Customer_ID**: Unique identifier for each customer
  - **Category**: Category of the purchased item
  - **Item**: Name of the purchased item
  - **Price_Per_Unit**: Static price of a single unit of the item
  - **Quantity**: Quantity of the purchased item
  - **Total_Spent**: Total amount spent on the transaction
  - **Payment_Method**: Method of payment used for the transaction
  - **Location**: Location where the transaction occurred
  - **Transaction_Date**: Date of the transaction
  - **Discount_Applied**: Identifies if a discount was applied to a transaction

---

# **Data Cleaning**
## **Goals:**
  - Find and delete duplicates
  - Idenify and populate missing values
  - Standardize column formatting
  - Create new columns
    
----

## **Identifying Unique Column Values**
**Input:**
```sql
-- Count unique columns
SELECT 
	COUNT(DISTINCT Transaction_ID) AS Transaction_ID,
	COUNT(DISTINCT Customer_ID) AS Customer_ID, 
	COUNT(DISTINCT Category) AS Category, 
	COUNT(DISTINCT Item) AS Item, 
	COUNT(DISTINCT Price_Per_Unit) AS Price_Per_Unit, 
	COUNT(DISTINCT Quantity) AS Quantity, 
	COUNT(DISTINCT Total_Spent) AS Total_Spent, 
	COUNT(DISTINCT Payment_Method) AS Payment_Method, 
	COUNT(DISTINCT Location) AS Location, 
	COUNT(DISTINCT Transaction_Date) AS Transaction_Date, 
	COUNT(DISTINCT Discount_Applied) AS Discount_Applied
FROM PortfolioProject.dbo.retail_store_sales
```
**Output:**

| Transaction_ID | Customer_ID	| Category	| Item	| Price_Per_Unit	| Quantity	| Total_Spent	| Payment_Method	| Location	| Transaction_Date	| Discount_Applied |
|----------------|--------------|-----------|-------|-----------------|-----------|-------------|-----------------|-----------|-------------------|------------------|
| 12575	         | 25	          | 8         | 200   | 25	            | 10	      | 227	        | 3 	            | 2	        | 1114	            | 2                |

---

## **Finding and Deleting Duplicates**
**Input:**
```sql
-- Find Duplicates
SELECT COUNT(DISTINCT Transaction_ID) AS Unique_Fields
FROM PortfolioProject.dbo.retail_store_sales
```

**Output:**
|Unique_Fields|
|---|
|12575|

No duplicates found in dataset.

---

## **Identifying Missing Values**
**Input:**
```sql
-- Count column nulls
SELECT 
	SUM(CASE WHEN Transaction_ID IS NULL THEN 1 ELSE 0 END) AS Transaction_ID_Nulls,
	SUM(CASE WHEN Customer_ID IS NULL THEN 1 ELSE 0 END) AS Customer_ID_Nulls,
	SUM(CASE WHEN Category IS NULL THEN 1 ELSE 0 END) AS Category_Nulls,
	SUM(CASE WHEN Item IS NULL THEN 1 ELSE 0 END) AS Item_Nulls,
	SUM(CASE WHEN Price_Per_Unit IS NULL THEN 1 ELSE 0 END) AS Price_Per_Unit_Nulls,
	SUM(CASE WHEN Quantity IS NULL THEN 1 ELSE 0 END) AS Quantity_Nulls,
	SUM(CASE WHEN Total_Spent IS NULL THEN 1 ELSE 0 END) AS Total_Spent_Nulls,
	SUM(CASE WHEN Payment_Method IS NULL THEN 1 ELSE 0 END) AS Payment_Method_Nulls,
	SUM(CASE WHEN Location IS NULL THEN 1 ELSE 0 END) AS Locationt_Nulls,
	SUM(CASE WHEN Transaction_Date IS NULL THEN 1 ELSE 0 END) AS Transaction_Date_Nulls,
	SUM(CASE WHEN Discount_Applied IS NULL THEN 1 ELSE 0 END) AS Discount_Applied_Nulls
FROM PortfolioProject.dbo.retail_store_sales
```

**Output:**

|Transaction_ID_Nulls	|Customer_ID_Nulls	|Category_Nulls	|Item_Nulls	|Price_Per_Unit_Nulls	|Quantity_Nulls	|Total_Spent_Nulls	|Payment_Method_Nulls	|Locationt_Nulls	|Transaction_Date_Nulls	|Discount_Applied_Nulls |
|---|---|---|---|---|---|---|---|---|---|---|
|0	|0	|0	|1213	|609	|604	|604	|0	|0	|0	|4199|

Item, Price_Per_Unit, Quantity, and Total_Spent columns contain nulls that will be populated with data. The Discount_Applied column contains nulls but will not be populated.

---
## **Populating Missing Values**
**Goals:** Populate null values in 'Item', 'Price_Per_Unit', 'Quantity', and 'Total_Spent' 

### **Price_Per_Unit**
Nulls were populated using the equation ('Total_Spent')/('Quantity') = 'Price_Per_Unit'.

**Input:**
```sql
UPDATE PortfolioProject.dbo.retail_store_sales
SET Price_Per_Unit = Total_Spent / Quantity
WHERE Price_Per_Unit IS NULL
```

### **Item**
Each item has a unique price within the item's category. To populate the 'Item' column, data from the 'Category' and 'Price_Per_Unit' columns are concatenated and inserted into a new column called 'Category_Price_Per_Unit'.

**Input:**
```sql
-- Create new column
ALTER TABLE PortfolioProject.dbo.retail_store_sales
ADD Category_Price_Per_Unit nvarchar(50)

-- Populate new column
UPDATE PortfolioProject.dbo.retail_store_sales
SET Category_Price_Per_Unit = CONCAT(Category, '_', Price_Per_Unit)

-- Verify sample of data
SELECT 
	Category,
	Price_Per_Unit,
	Category_Price_Per_Unit
FROM PortfolioProject.dbo.retail_store_sales
```
**Output:**
|Category	|Price_Per_Unit	|Category_Price_Per_Unit|
|---|---|---|
|Patisserie	|18.5	|Patisserie_18.5|
|Milk Products	|29	|Milk Products_29|
|Butchers	|21.5	|Butchers_21.5|
|Beverages	|27.5	|Beverages_27.5|
|Food	|12.5	|Food_12.5|

Using a self-join, replace nulls in the 'Item' column with items that have matching values in the 'Category_Price_Per_Unit' column.
**Input:**
```sql
SELECT
    A.Category_Price_Per_Unit,
    A.Item AS A_Item,
    (SELECT TOP 1 B.Item 
     FROM PortfolioProject.dbo.retail_store_sales AS B
     WHERE B.Category_Price_Per_Unit = A.Category_Price_Per_Unit
       AND B.Item IS NOT NULL) AS FilledItem
FROM PortfolioProject.dbo.retail_store_sales AS A
WHERE A.Item IS NULL

UPDATE A
SET Item = (
	SELECT TOP 1 B.Item 
	FROM PortfolioProject.dbo.retail_store_sales AS B
	WHERE B.Category_Price_Per_Unit = A.Category_Price_Per_Unit
       AND B.Item IS NOT NULL
)	
FROM PortfolioProject.dbo.retail_store_sales AS A
WHERE A.Item IS NULL
	AND EXISTS (
		SELECT 1
		  FROM PortfolioProject.dbo.retail_store_sales AS B
		  WHERE B.Category_Price_Per_Unit = A.Category_Price_Per_Unit
			AND B.Item IS NOT NULL
	)
```

**Output:**

### **Quantity**

```sql

```

**Output:**


### **Total_Spent**
```sql

```

**Output:**

