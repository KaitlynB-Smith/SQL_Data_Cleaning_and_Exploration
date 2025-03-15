# SQL_Data_Cleaning_and_Exploration

The purpose of this SQL project is to clean and explore retail sales data, identifying patterns and trends in customer purchasing behavior. This project focuses on data cleaning techniques and initial exploration of the sales trends.

---

# **Data Overview**
The dataset contains sales data for transactions within a retail store between 2022 - 2025. The data consists of information about customers, items, payment methods, locations, dates, and discounts for each transaction. 

## Data Source
This dataset was created by [Ahmed Mohamed](https://www.kaggle.com/ahmedmohamed2003) to use for data analysis practice. This dataset was found on Kaggle and can be accessed by clicking [here](https://www.kaggle.com/datasets/ahmedmohamed2003/retail-store-sales-dirty-for-data-cleaning?select=retail_store_sales.csv).

## Column Descriptions
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
**Query:**
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
**Query:**
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
**Query:**
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

---

### **Price_Per_Unit Column**
Nulls were populated using the equation ('Total_Spent')/('Quantity') = 'Price_Per_Unit'.

**Query:**
```sql
UPDATE PortfolioProject.dbo.retail_store_sales
SET Price_Per_Unit = Total_Spent / Quantity
WHERE Price_Per_Unit IS NULL
```
---

### **Item Column**
Each item has a unique price within the item's category. To populate the 'Item' column, data from the 'Category' and 'Price_Per_Unit' columns are concatenated and inserted into a new column called 'Category_Price_Per_Unit'.

**Query:**
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

Using a self-join, populate nulls in the 'Item' column with item names that have matching records in the 'Category_Price_Per_Unit' column.
**Query:**
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

-- Check for null values
SELECT 
	SUM(CASE WHEN Item IS NULL THEN 1 ELSE 0 END) AS Item_Nulls
FROM PortfolioProject.dbo.retail_store_sales
```

**Output:**
|Item_Nulls|
|---|
|0|

---

### **Quantity Column**
The nulls in the 'Quantity' column are populated using a multi-stage imputation strategy:
  1. Calculate the average quantity purchased for each combination of 'Customer_ID', 'Category', and 'Payment_Method'.
  2. Use these calculations to populate null values in 'Quantity', prioritizing the specific average for the customer's category and payment method.
  3. If no specific average exists for a type of payment method, use the average quantity across all payment methods for that customer and category.
  4. If no average is available for any payment method for the customer and category, populate the field with the customer's overall average quantity.

Create a new column called 'Imputed_Quantity'. This column will contain the calculated quantities while maintaining the original data in the 'Quantity' column.

**Query:**
```sql
-- Create new column 'Imputed_Quantity'
ALTER TABLE PortfolioProject.dbo.retail_store_sales
ADD Imputed_Quantity int

-- Imputation strategy
WITH ImputedQuantities AS (
	SELECT 
		r.Customer_ID,
		r.Category,
		r.Item,
		r.Quantity AS Original_Quantity,
		CAST(ROUND(COALESCE( 
			CASE r.Payment_Method
				WHEN 'Digital Wallet' THEN iq.Avg_Digital_Wallet
				WHEN 'Credit Card' THEN iq.Avg_Credit_Card
				WHEN 'Cash' THEN iq.Avg_Cash
				ELSE NULL
			END,
			iq.Avg_Quantity,
			(SELECT AVG(Quantity) FROM PortfolioProject.dbo.retail_store_sales WHERE Customer_ID = r.Customer_ID)
		), 0) AS int) AS Imputed_Quantity
	FROM PortfolioProject.dbo.retail_store_sales AS r
	INNER JOIN (
		SELECT 
			Customer_ID,
			Category,
			Item, 
			AVG(Quantity) AS Avg_Quantity,
			AVG(CASE WHEN Payment_Method = 'Digital Wallet' THEN Quantity ELSE NULL END) AS Avg_Digital_Wallet,
			AVG(CASE WHEN Payment_Method = 'Credit Card' THEN Quantity ELSE NULL END) AS Avg_Credit_Card,
			AVG(CASE WHEN Payment_Method = 'Cash' THEN Quantity ELSE NULL END) AS Avg_Cash
		FROM PortfolioProject.dbo.retail_store_sales 
		GROUP BY
			Customer_ID, 
			Category, 
			Item
	) AS iq
		ON r.Customer_ID = iq.Customer_ID
		AND r.Category = iq.Category
		AND r.Item = iq.Item
	WHERE r.Quantity IS NULL
)
UPDATE r
SET r.Imputed_Quantity = iq.Imputed_Quantity
FROM PortfolioProject.dbo.retail_store_sales AS r
INNER JOIN ImputedQuantities AS iq
	ON r.Customer_ID = iq.Customer_ID
	AND r.Category = iq.Category
	AND r.Item = iq.Item
WHERE r.Quantity IS NULL
```

Create a new column 'Quantity_Combined' to contain the combined data from the 'Quantity' and 'Imputed_Quantity' columns.

**Query:**
```sql
-- Create column 'Combined_Quantity'
ALTER TABLE PortfolioProject.dbo.retail_store_sales
ADD Quantity_Combined int

-- Coalesce data from 'Quantity' and 'Imputed_Quantity'
UPDATE PortfolioProject.dbo.retail_store_sales
SET Quantity_Combined = COALESCE(CAST(Imputed_Quantity AS float), Quantity)

-- Check for nulls
SELECT 
	SUM(CASE WHEN Quantity_Combined IS NULL THEN 1 ELSE 0 END) AS Quantity_Nulls
FROM PortfolioProject.dbo.retail_store_sales
```
**Output:**

|Quantity_Nulls|
|---|
|0|

---

### **Total_Spent Column**
Populate nulls in 'Total_Spent' column using the equation ('Price_Per_Unit') * ('Quantity_Combined') = ('Imputed_Total_Spent')

**Query:**
```sql
-- Create column 'Imputed_Total_Spent'
ALTER TABLE PortfolioProject.dbo.retail_store_sales
ADD Imputed_Total_Spent float

-- Populate new column
UPDATE PortfolioProject.dbo.retail_store_sales
SET Imputed_Total_Spent = Price_Per_Unit * Quantity_Combined

-- Check for nulls
SELECT 
	SUM(CASE WHEN Imputed_Total_Spent IS NULL THEN 1 ELSE 0 END) AS Total_Spent_Nulls
FROM PortfolioProject.dbo.retail_store_sales
```

**Output:**
|Total_Spent_Nulls|
|---|
|0|

---

## **Standardizing Column Formatting**
### **Transaction_Date**
Convert data type from 'datetime' to 'date'.

**Query:**
```sql
-- Datetime to date conversion
ALTER TABLE PortfolioProject.dbo.retail_store_sales
ALTER COLUMN Transaction_Date date

UPDATE PortfolioProject.dbo.
SET Transaction_Date = CAST(Transaction_Date AS date)
```

---

## **Creating New Columns**
New columns are created for further data exploration. 

### **Week_Day Column**
The 'Week_Day' column is created to determine the day of the week (Monday - Sunday) that a transaction took place based on the 'Transaction_Date' column.

**Query:**
```sql
-- Create new column 'Week_Day'
ALTER TABLE PortfolioProject.dbo.retail_store_sales
ADD Week_Day nvarchar(50)

-- Populate column
UPDATE PortfolioProject.dbo.retail_store_sales
SET Week_Day = DATENAME(dw, Transaction_Date)

-- Sample data to verify
SELECT 
	Transaction_ID,
	Transaction_Date,
	Week_Day
FROM PortfolioProject.dbo.retail_store_sales
```

**Output:**
|Transaction_ID	|Transaction_Date	|Week_Day|
|---|---|---|
|TXN_6867343|	2024-04-08	|Monday|
|TXN_3731986|	2023-07-23	|Sunday|
|TXN_9303719|	2022-10-05	|Wednesday|
|TXN_9458126|	2022-05-07	|Saturday|
|TXN_4575373|	2022-10-02	|Sunday|

---

### **Month Column**
The 'Month' column is created to determine the month (January - December) in which a transaction took place based on the 'Transaction_Date' column.

**Query:**
```sql
-- Create new column 'Month'
ALTER TABLE PortfolioProject.dbo.retail_store_sales
ADD Month nvarchar(50)

-- Populate new column
UPDATE PortfolioProject.dbo.retail_store_sales
SET Month = DATENAME(mm, Transaction_Date)

-- Sample data to verify
SELECT 
	Transaction_ID,
	Transaction_Date,
	Month
FROM PortfolioProject.dbo.retail_store_sales
```

**Output:**
|Transaction_ID|	Transaction_Date	|Month|
|---|---|---|
|TXN_6867343|	2024-04-08	|April|
|TXN_3731986|	2023-07-23	|July|
|TXN_9303719|	2022-10-05	|October|
|TXN_9458126|	2022-05-07	|May|
|TXN_4575373|	2022-10-02	|October|

---

# **Data Exploration**
To determine trends in customer behavior and purchasing patterns, a basic data exploration is conducted. The analyses are divided into six categories:
1. Temporal Analysis
2. Transaction Overview
3. Category Analysis
4. Product Analysis
5. Payment Method Analysis
6. Location Analysis


##  **1. Temporal Analysis** 
<details>
	<summary>
		<b>Revenue by Week Day:</b>
		On average, Fridays generated the greatest revenue
	</summary>
<b>Query</b>
 
```sql
SELECT 
	Week_Day,
	AVG(Imputed_Total_Spent) AS Avg_Spent
FROM PortfolioProject.dbo.retail_store_sales
GROUP BY Week_Day
Order BY AVG(Imputed_Total_Spent) DESC
```

<b>Output</b>

|Week_Day	|Avg_Spent|
|---|---|
|Friday	|135.090707964602|
|Saturday|	131.195579182988|
|Sunday|	130.182495858642|
|Thursday|	129.618156424581|
|Tuesday|	128.873322147651|
|Wednesday|	127.891064533922|
|Monday|	125.267716535433|

</details>



<details>
	<summary>
		<b>Revenue by Month:</b> January generated the greatest revenue on average
	</summary>
<b>Query</b>
	
```sql
SELECT 
	Month,
	AVG(Imputed_Total_Spent) AS Avg_Spent
FROM PortfolioProject.dbo.retail_store_sales
GROUP BY Month
Order BY AVG(Imputed_Total_Spent) DESC
```

<b>Output</b>

|Month	|Avg_Spent|
|---|---|
|January	|134.060249816312|
|December	|132.70087124879|
|April	|132.198795180723|
|September	|132.141038197845|
|February	|132.10207253886|
|June	|130.760597302505|
|November	|128.736789631107|
|October	|128.255623721881|
|May	|127.548886737657|
|July	|126.884756657484|
|March	|126.680569185476|
|August	|123.749278152069|

</details>

## **2. Transaction Overview**
<details>
	<summary>
		<b>Average Revenue per Transaction:</b> Customers spent between 5 and 410 dollars per transaction, spending an average of 130 dollars per transaction
	</summary>
<b>Query</b>
	
```sql
SELECT 
	AVG(Imputed_Total_Spent) AS Avg_Spent,
	MAX(Imputed_Total_Spent) AS Max_Spent,
	MIN(Imputed_Total_Spent) AS Min_Spent
FROM PortfolioProject.dbo.retail_store_sales
```

<b>Output</b>

|Avg_Spent		|Max_Spent	|Min_Spent	|
|---|---|---|
|129.740397614314	|410		|5		|
</details>

<details>
	<summary>
		<b>Transaction Count per Year:</b> The greatest number of transactions occurred in 2024, with the fewest in 2025 as there are only 2 months of data to observe in 2025.
	</summary>
<b>Query</b>
	
```sql
SELECT 
	YEAR(Transaction_Date) AS Year,
	COUNT(Transaction_ID) AS Transactions
FROM PortfolioProject.dbo.retail_store_sales
GROUP BY YEAR(Transaction_Date) 
ORDER BY COUNT(Transaction_ID) DESC
```

<b>Output</b>
|Year	|Transactions	|
|---|---|
|2024	|4241		|
|2022	|4134		|
|2023	|3987		|
|2025	|213		|

</details>

<details>
	<summary>
		<b>January Transactions:</b> 2025 has the fewest amount of transactions and the lowest amount spent per transaction
	</summary>
<b>Query</b>
	
```sql
SELECT 
	YEAR(Transaction_Date) AS Year,
	COUNT(Transaction_ID) AS Transactions,
	ROUND(SUM(Imputed_Total_Spent) / COUNT(Transaction_ID), 2) AS Avg_Spent
FROM PortfolioProject.dbo.retail_store_sales
WHERE Month = 'January' 
GROUP BY YEAR(Transaction_Date) 
ORDER BY COUNT(Transaction_ID) DESC
```

<b>Output</b>

|Year	|Transactions	|Avg_Spent	|
|---|---|---|
|2022	|390		|143.46		|
|2024	|385		|131.47		|
|2023	|373		|132.1		|
|2025	|213		|124.97		|

</details>

<details>
	<summary>
		<b>Revenue per Transaction (by Customer):</b> Customers spent between 122.17 - 138.06 per transaction
	</summary>
<b>Query</b>
	
```sql
SELECT 
	Customer_ID, 
	ROUND(AVG(Imputed_Total_Spent), 2) AS Avg_Spent,
	COUNT(Transaction_ID) AS Num_of_Transactions,
	MIN(Transaction_Date) AS From_Date,
	MAX(Transaction_Date) AS To_Date
FROM PortfolioProject.dbo.retail_store_sales
GROUP BY Customer_ID
ORDER BY Avg_Spent DESC
```

<b>Output</b>

|Customer_ID	|Avg_Spent	|Num_of_Transactions	|From_Date	|To_Date|
|---|---|---|---|---|
|CUST_03	|138.06		|465			|2022-01-08	|2025-01-16|
|CUST_04	|136.51		|474			|2022-01-02	|2025-01-18|
|CUST_02	|133.8		|488			|2022-01-17	|2025-01-16|
|CUST_23	|133.37		|513			|2022-01-02	|2025-01-15|
|CUST_08	|133.06		|533			|2022-01-03	|2025-01-17|
|CUST_10	|132.67		|501			|2022-01-02	|2025-01-18|
|CUST_24	|132.44		|543			|2022-01-01	|2025-01-18|
|CUST_16	|132.13		|515			|2022-01-01	|2025-01-16|
|CUST_19	|131.55		|487			|2022-01-01	|2025-01-17|
|CUST_22	|131.2		|501			|2022-01-01	|2025-01-15|
|CUST_07	|130.74		|491			|2022-01-04	|2025-01-18|
|CUST_21	|130.62		|498			|2022-01-05	|2025-01-15|
|CUST_14	|129.73		|484			|2022-01-07	|2025-01-17|
|CUST_05	|129.1		|544			|2022-01-03	|2025-01-14|
|CUST_13	|128.88		|534			|2022-01-04	|2025-01-18|
|CUST_12	|127.64		|498			|2022-01-01	|2025-01-18|
|CUST_20	|127.32		|507			|2022-01-03	|2025-01-16|
|CUST_17	|127.3		|487			|2022-01-01	|2025-01-12|
|CUST_06	|127.25		|481			|2022-01-01	|2025-01-16|
|CUST_11	|126.79		|503			|2022-01-01	|2025-01-18|
|CUST_15	|126.75		|519			|2022-01-01	|2025-01-17|
|CUST_25	|126.49		|476			|2022-01-03	|2025-01-18|
|CUST_18	|125.26		|507			|2022-01-02	|2025-01-17|
|CUST_09	|123.54		|519			|2022-01-03	|2025-01-17|
|CUST_01	|122.17		|507			|2022-01-01	|2025-01-18|

</details>



## **3. Category Analysis**
<details>
	<summary>
		<b>Revenue by Category:</b> The butcher's category yielded the highest sales with an average of 25.25 dollars being spent per item
	</summary>
<b>Query</b>
	
```sql
SELECT 
	Category,
	SUM(Imputed_Total_Spent) AS Total_Spent,
	COUNT(Transaction_ID) AS Count_Transactions, 
	SUM(Quantity_Combined) AS Quantity, 
	SUM(Imputed_Total_Spent) / SUM(Quantity_Combined) AS Avg_Spent_Per_Item
FROM PortfolioProject.dbo.retail_store_sales
GROUP BY Category
ORDER BY SUM(Imputed_Total_Spent) DESC
```

<b>Output</b>

|Category				|Total_Spent	|Count_Transactions	|Quantity	|Avg_Spent_Per_Item|
|---|---|---|---|---|
|Butchers				|216936		|1568			|8592		|25.2486033519553|
|Electric household essentials		|214673.5	|1591			|8750		|24.5341142857143|
|Beverages				|205948.5	|1567			|8739		|23.5665980089255|
|Food					|205275.5	|1588			|8875		|23.1296338028169|
|Furniture				|204385.5	|1591			|8814		|23.1887338325391|
|Computers and electric accessories	|201837.5	|1558			|8758		|23.0460721625942|
|Patisserie				|193245.5	|1528			|8407		|22.9862614487927|
|Milk Products				|189183.5	|1584			|8737		|21.6531418106902|

</details>


<details>
	<summary>
		<b>Top Sold Items per Category (By Quantity):</b> 
	</summary>
<b>Query</b>
	
```sql
WITH Ranked_Items_Count AS (
	SELECT 
		Category,
		Item,
		COUNT(Transaction_ID) AS Transactions,
		ROW_NUMBER() OVER(PARTITION BY Category ORDER BY COUNT(Transaction_ID) DESC) AS Rank
	FROM PortfolioProject.dbo.retail_store_sales
	GROUP BY Category, Item
)
SELECT 
	Category,
	Item,
	Transactions,
	Rank
FROM Ranked_Items_Count
WHERE Rank <= 5
ORDER BY Category, Rank ASC
```
</details>

<table>
<tr>
	<th>Beverages</th>
	<th>Butchers</th>
	<th>Computers and electric accessories</th>
	<th>Electric household essentials</th>
	<th>Food</th>
	<th>Furniture</th>
	<th>Milk Products</th>
	<th>Patisserie</th>
</tr>
<tr><td>
	
|Category			|Item		|Transactions	|Rank	|
|---|---|---|---|
|Beverages			|Item_2_BEV	|132		|1	|
|Beverages			|Item_14_BEV	|113		|2	|
|Beverages			|Item_12_BEV	|108		|3	|
|Beverages			|Item_8_BEV	|87		|4	|
|Beverages			|Item_17_BEV	|86		|5	|

</td><td>
	
|Category	|Item		|Transactions	|Rank	|
|---|---|---|---|
|Butchers	|Item_22_BUT	|113		|1	|
|Butchers	|Item_20_BUT	|113		|2	|
|Butchers	|Item_23_BUT	|109		|3	|
|Butchers	|Item_25_BUT	|105		|4	|
|Butchers	|Item_12_BUT	|102		|5	|

</td><td>
	
|Category			|Item		|Transactions	|Rank	|
|---|---|---|---|
|Computers and electric accessories	|Item_19_CEA	|124	|1	|
|Computers and electric accessories	|Item_12_CEA	|114	|2	|
|Computers and electric accessories	|Item_5_CEA	|108	|3	|
|Computers and electric accessories	|Item_20_CEA	|83	|4	|
|Computers and electric accessories	|Item_15_CEA	|81	|5	|

</td><td>
	
|Category			|Item		|Transactions	|Rank	|
|---|---|---|---|
|Electric household essentials	|Item_25_EHE	|113		|1	|
|Electric household essentials	|Item_8_EHE	|113		|2	|
|Electric household essentials	|Item_15_EHE	|105		|3	|
|Electric household essentials	|Item_23_EHE	|103		|4	|
|Electric household essentials	|Item_20_EHE	|103		|5	|

</td><td>
	

|Category	|Item		|Transactions	|Rank	|
|---|---|---|---|
|Food		|Item_14_FOOD	|118		|1	|
|Food		|Item_13_FOOD	|113		|2	|
|Food		|Item_5_FOOD	|110		|3	|
|Food		|Item_20_FOOD	|102		|4	|
|Food		|Item_25_FOOD	|100		|5	|

</td><td>	

|Category	|Item		|Transactions	|Rank	|
|---|---|---|---|
|Furniture	|Item_25_FUR	|121		|1	|
|Furniture	|Item_11_FUR	|120		|2	|
|Furniture	|Item_24_FUR	|116		|3	|
|Furniture	|Item_5_FUR	|115		|4	|
|Furniture	|Item_2_FUR	|106		|5	|

</td><td>
	
|Category	|Item		|Transactions	|Rank	|
|---|---|---|---|
|Milk Products	|Item_16_MILK	|124		|1	|
|Milk Products	|Item_19_MILK	|121		|2	|
|Milk Products	|Item_1_MILK	|118		|3	|
|Milk Products	|Item_11_MILK	|117		|4	|
|Milk Products	|Item_3_MILK	|104		|5	|

</td><td>
	
|Category	|Item		|Transactions	|Rank	|
|---|---|---|---|
|Patisserie	|Item_12_PAT	|107		|1	|
|Patisserie	|Item_11_PAT	|104		|2	|
|Patisserie	|Item_17_PAT	|101		|3	|
|Patisserie	|Item_23_PAT	|98		|4	|
|Patisserie	|Item_20_PAT	|92		|5	|

</td></tr>
</table>


<details>
	<summary>
		<b>Top Sold Items per Category (By Revenue):</b> 
	</summary>
<b>Query</b>
	
```sql
WITH Ranked_Items_Sum AS (
	SELECT 
		Category,
		Item,
		SUM(Imputed_Total_Spent) AS Total_Spent,
		ROW_NUMBER() OVER(PARTITION BY Category ORDER BY SUM(Imputed_Total_Spent) DESC) AS Rank
	FROM PortfolioProject.dbo.retail_store_sales
	GROUP BY Category, Item
)
SELECT 
	Category,
	Item,
	Total_Spent,
	Rank
FROM Ranked_Items_Sum
WHERE Rank <= 5
ORDER BY Category, Rank ASC
```
</details>

<table>
<tr>
	<th>Beverages</th>
	<th>Butchers</th>
	<th>Computers and electric accessories</th>
	<th>Electric household essentials</th>
	<th>Food</th>
	<th>Furniture</th>
	<th>Milk Products</th>
	<th>Patisserie</th>
</tr>
<tr><td>
	
|Category	|Item		|Total_Spent	|Rank	|
|---|---|---|---|
|Beverages	|Item_25_BEV	|20090		|1	|
|Beverages	|Item_24_BEV	|16629.5	|2	|
|Beverages	|Item_20_BEV	|16080		|3	|
|Beverages	|Item_14_BEV	|15141		|4	|
|Beverages	|Item_17_BEV	|14732		|5	|

</td><td>
	
|Category	|Item		|Total_Spent	|Rank	|
|---|---|---|---|
|Butchers	|Item_25_BUT	|23329		|1	|
|Butchers	|Item_22_BUT	|22338		|2	|
|Butchers	|Item_23_BUT	|21888		|3	|
|Butchers	|Item_20_BUT	|19966		|4	|
|Butchers	|Item_16_BUT	|13255		|5	|

</td><td>
	
|Category				|Item		|Total_Spent	|Rank	|
|---|---|---|---|
|Computers and electric accessories	|Item_19_CEA	|21280		|1	|
|Computers and electric accessories	|Item_20_CEA	|15443.5	|2	|
|Computers and electric accessories	|Item_12_CEA	|13910.5	|3	|
|Computers and electric accessories	|Item_21_CEA	|12810		|4	|
|Computers and electric accessories	|Item_15_CEA	|12246		|5	|

</td><td>
	
|Category			|Item		|Total_Spent	|Rank	|
|---|---|---|---|
|Electric household essentials	|Item_25_EHE	|26855		|1	|
|Electric household essentials	|Item_23_EHE	|21128		|2	|
|Electric household essentials	|Item_20_EHE	|18559		|3	|
|Electric household essentials	|Item_15_EHE	|15262		|4	|
|Electric household essentials	|Item_17_EHE	|13920		|5	|

</td><td>
	

|Category	|Item		|Total_Spent	|Rank	|
|---|---|---|---|
|Food		|Item_25_FOOD	|22755		|1	|
|Food		|Item_20_FOOD	|19162		|2	|
|Food		|Item_14_FOOD	|16047.5	|3	|
|Food		|Item_13_FOOD	|14812		|4	|
|Food		|Item_18_FOOD	|14030		|5	|

</td><td>	

|Category	|Item		|Total_Spent	|Rank	|
|---|---|---|---|
|Furniture	|Item_25_FUR	|27183		|1	|
|Furniture	|Item_24_FUR	|23700		|2	|
|Furniture	|Item_20_FUR	|16482		|3	|
|Furniture	|Item_23_FUR	|16036		|4	|
|Furniture	|Item_21_FUR	|15295		|5	|

</td><td>
	
|Category	|Item		|Total_Spent	|Rank	|
|---|---|---|---|
|Milk Products	|Item_19_MILK	|21120		|1	|
|Milk Products	|Item_16_MILK	|18975		|2	|
|Milk Products	|Item_23_MILK	|17328		|3	|
|Milk Products	|Item_17_MILK	|14471		|4	|
|Milk Products	|Item_22_MILK	|13943		|5	|

</td><td>
	
|Category	|Item		|Total_Spent	|Rank	|
|---|---|---|---|
|Patisserie	|Item_23_PAT	|20786		|1	|
|Patisserie	|Item_20_PAT	|17252.5	|2	|
|Patisserie	|Item_17_PAT	|16820		|3	|
|Patisserie	|Item_24_PAT	|14101.5	|4	|
|Patisserie	|Item_12_PAT	|12298		|5	|

</td></tr>
</table>


## **4. Product Analysis**
<details>
	<summary>
		<b>Most Popular Items by Quantity Sold:</b> 
	</summary>
<b>Query</b>
	
```sql
WITH Ranked_Items_Quantity AS (
	SELECT 
		Item,
		SUM(Quantity_Combined) AS Quantity,
		ROW_NUMBER() OVER(ORDER BY SUM(Quantity_Combined) DESC) AS Rank
	FROM PortfolioProject.dbo.retail_store_sales
	GROUP BY Item
)
SELECT 
	Item,
	Quantity,
	Rank
FROM Ranked_Items_Quantity
WHERE Rank <= 10
ORDER BY Rank ASC
```
</details>

|Item		|Quantity	|Rank	|
|---|---|---|
|Item_2_BEV	|715		|1	|
|Item_16_MILK	|690		|2	|
|Item_19_CEA	|665		|3	|
|Item_25_FUR	|663		|4	|
|Item_19_MILK	|660		|5	|
|Item_14_FOOD	|655		|6	|
|Item_25_EHE	|655		|7	|
|Item_12_CEA	|647		|8	|
|Item_13_FOOD	|644		|9	|
|Item_1_MILK	|641		|10	|


<details>
	<summary>
		<b>Most Popular Items by Revenue:</b> 
	</summary>
<b>Query</b>
	
```sql
WITH Ranked_Items_Revenue AS (
	SELECT 
		Item,
		SUM(Imputed_Total_Spent) AS Revenue,
		ROW_NUMBER() OVER(ORDER BY SUM(Imputed_Total_Spent) DESC) AS Rank
	FROM PortfolioProject.dbo.retail_store_sales
	GROUP BY Item
)
SELECT 
	Item,
	Revenue,
	Rank
FROM Ranked_Items_Revenue
WHERE Rank <= 10
ORDER BY Rank ASC
```
</details>

|Item		|Revenue	|Rank	|
|---|---|---|
|Item_25_FUR	|27183		|1	|
|Item_25_EHE	|26855		|2	|
|Item_24_FUR	|23700		|3	|
|Item_25_BUT	|23329		|4	|
|Item_25_FOOD	|22755		|5	|
|Item_22_BUT	|22338		|6	|
|Item_23_BUT	|21888		|7	|
|Item_19_CEA	|21280		|8	|
|Item_23_EHE	|21128		|9	|
|Item_19_MILK	|21120		|10	|


## **5. Payment_Method Analysis**

<details>
	<summary>
		<b>Payment Method Frequency and Transaction Value:</b> The most commonly used payment method was cash, and the average transaction value was the greatest for cash transactions
	</summary>
<b>Query</b>
	
```sql
SELECT 
	Payment_Method,
	COUNT(Payment_Method) AS Count_Payment_Method, 
	AVG(Imputed_Total_Spent) AS Avg_Transaction_Value
FROM PortfolioProject.dbo.retail_store_sales
GROUP BY Payment_Method
```

<b>Output</b>

|Payment_Method	|Count_Payment_Method	|Avg_Transaction_Value|
|---|---|---|
|Cash		|4310			|131.13283062645|
|Credit Card	|4121			|129.247270080078|
|Digital Wallet	|4144			|128.782577220077|

</details>


<details>
	<summary>
		<b>Revenue by Location and Payment Method:</b> Online cash-payment transactions occurred most frequently and generated the highest total revenue
	</summary>
<b>Query</b>
	
```sql
SELECT 
	Payment_Method,
	Location,
	COUNT(Payment_Method) AS Count_Payment_Method, 
	SUM(Imputed_Total_Spent) AS Sum_Transaction_Value
FROM PortfolioProject.dbo.retail_store_sales
GROUP BY Payment_Method, Location
ORDER BY SUM(Imputed_Total_Spent) DESC
```

<b>Output</b>

|Payment_Method	|Location	|Count_Payment_Method	|Sum_Transaction_Value	|
|---|---|---|---|
|Cash		|Online		|2172			|288355			|
|Cash		|In-store	|2138			|276827.5		|
|Credit Card	|Online		|2101			|273013.5		|
|Digital Wallet	|Online		|2081			|266898			|
|Digital Wallet	|In-store	|2063			|266777			|
|Credit Card	|In-store	|2020			|259614.5		|

</details>



##  **6. Location Analysis**
<details>
	<summary>
		<b>Online vs. In-Store Sales:</b> More purchases occurred online as opposed to in-store, and online sales also generated greater revenue
	</summary>
<b>Query</b>
	
```sql
SELECT 
	Location,
	COUNT(Transaction_ID) AS Count_Transactions, 
	AVG(Imputed_Total_Spent) AS Avg_Spent,
	SUM(Imputed_Total_Spent) AS Total_Spent
FROM PortfolioProject.dbo.retail_store_sales
GROUP BY Location
```

<b>Output</b>

|Location	|Count_Transactions	|Avg_Spent		|Total_Spent	|
|---|---|---|---|
|Online		|6354			|130.353556814605	|828266.5	|
|In-store	|6221			|129.114129561164	|803219		|

</details>


<details>
	<summary>
		<b>Most Popular Items by Location (Based on Quantity):</b> 
	</summary>
<b>Query</b>
	
```sql
WITH Ranked_Items_Location AS (
	SELECT 
		Item,
		Location,
		SUM(Quantity_Combined) AS Sum_Quantity,
		ROW_NUMBER() OVER(PARTITION BY Location ORDER BY SUM(Quantity_Combined) DESC) AS Rank
	FROM PortfolioProject.dbo.retail_store_sales
	GROUP BY Location, Item
)
SELECT 
	Item,
	Location,
	Sum_Quantity,
	Rank
FROM Ranked_Items_Location
WHERE Rank <= 10
ORDER BY Location, Rank ASC
```
</details>


<table>
<tr>
	<th>In-Store</th>
	<th>Online</th>
</tr>
<tr><td>
	
|Item		|Location	|Sum_Quantity	|Rank	|
|---|---|---|---|
|Item_16_MILK	|In-store	|381		|1	|
|Item_2_BEV	|In-store	|376		|2	|
|Item_24_FUR	|In-store	|363		|3	|
|Item_25_FUR	|In-store	|357		|4	|
|Item_14_FOOD	|In-store	|357		|5	|
|Item_1_MILK	|In-store	|352		|6	|
|Item_25_EHE	|In-store	|343		|7	|
|Item_12_CEA	|In-store	|343		|8	|
|Item_13_FOOD	|In-store	|336		|9	|
|Item_15_FUR	|In-store	|327		|10	|

</td><td>
	
|Item		|Location	|Sum_Quantity	|Rank	|
|---|---|---|---|
|Item_19_CEA	|Online		|442		|1	|
|Item_5_CEA	|Online		|382		|2	|
|Item_14_BEV	|Online		|355		|3	|
|Item_19_MILK	|Online		|350		|4	|
|Item_2_BEV	|Online		|339		|5	|
|Item_13_EHE	|Online		|330		|6	|
|Item_11_PAT	|Online		|324		|7	|
|Item_11_FUR	|Online		|324		|8	|
|Item_20_EHE	|Online		|324		|9	|
|Item_23_PAT	|Online		|316		|10	|

</td></tr>
</table>

<details>
	<summary>
		<b>Most Popular Items by Location (Based on Revenue):</b> 
	</summary>
<b>Query</b>
	
```sql
WITH Ranked_Items_Location AS (
	SELECT 
		Item,
		Location,
		SUM(Imputed_Total_Spent) AS Total_Spent,
		ROW_NUMBER() OVER(PARTITION BY Location ORDER BY SUM(Imputed_Total_Spent) DESC) AS Rank
	FROM PortfolioProject.dbo.retail_store_sales
	GROUP BY Location, Item
)
SELECT 
	Item,
	Location,
	Total_Spent,
	Rank
FROM Ranked_Items_Location
WHERE Rank <= 10
ORDER BY Location, Rank ASC
```
</details>

<table>
<tr>
	<th>In-Store</th>
	<th>Online</th>
</tr>
<tr><td>
	
|Item		|Location		|Total_Spent	|Rank	|
|---|---|---|---|
|Item_25_FUR	|In-store		|14637		|1	|
|Item_24_FUR	|In-store		|14338.5	|2	|
|Item_25_EHE	|In-store		|14063		|3	|
|Item_23_EHE	|In-store		|12350		|4	|
|Item_25_BUT	|In-store		|11439		|5	|
|Item_22_BUT	|In-store		|11388		|6	|
|Item_23_BUT	|In-store		|10868		|7	|
|Item_16_MILK	|In-store		|10477.5	|8	|
|Item_25_FOOD	|In-store		|10414		|9	|
|Item_25_BEV	|In-store		|10086		|10	|

</td><td>
	
|Item		|Location	|Total_Spent	|Rank	|
|---|---|---|---|
|Item_19_CEA	|Online		|14144		|1	|
|Item_25_EHE	|Online		|12792		|2	|
|Item_25_FUR	|Online		|12546		|3	|
|Item_25_FOOD	|Online		|12341		|4	|
|Item_23_PAT	|Online		|12008		|5	|
|Item_25_BUT	|Online		|11890		|6	|
|Item_19_MILK	|Online		|11200		|7	|
|Item_23_BUT	|Online		|11020		|8	|
|Item_22_BUT	|Online		|10950		|9	|
|Item_20_EHE	|Online		|10854		|10	|

</td></tr>
</table>

---

# **Conclusion**
## Project Takeaways
This project emphasizes data cleaning and light exploration using SQL. Using these methods to analyze patterns and trends in customer purchasing behavior can allow for companies to make data-driven decisions regarding their business model. These decisions can help to increase customer satisfaction, business efficiency, and profits for a company.
<br>
<br>
## Further Analysis
It would be interesting to collect data of a greater number of customers and analyze customer purchasing trends among different demographics, such as age, sex, income level, etc. It would also be interesting to determine lifetime value of customers and identify factors that may affect the lifetime value. 

