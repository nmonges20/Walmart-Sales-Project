# Walmart Data Analysis: End-to-End SQL + Python Project P-9

## Project Overview

![Project Pipeline](https://github.com/najirh/Walmart_SQL_Python/blob/main/walmart_project-piplelines.png)


This project is an end-to-end data analysis solution designed to extract critical business insights from Walmart sales data. I utilized Python for data processing and analysis, SQL for advanced querying, and structured problem-solving techniques to solve key business questions.

---

## Project Steps

### 1. Set Up the Environment
   - **Tools Used**: Visual Studio Code (VS Code), Python, SQL (PostgreSQL)
   - **Goal**: Create a structured workspace within VS Code and organize project folders for smooth development and data handling.

### 2. Set Up Kaggle API
   - **API Setup**: Obtain my Kaggle API token from [Kaggle](https://www.kaggle.com/) by navigating to my profile settings and downloading the JSON file.
   - **Configure Kaggle**: 
      - Placed the downloaded `kaggle.json` file in my local `.kaggle` folder.
      - Use the command `kaggle datasets download -d <dataset-path>` to pull datasets directly into my project.

### 3. Download Walmart Sales Data
   - **Data Source**: Used the Kaggle API to download the Walmart sales datasets from Kaggle.
   - **Dataset Link**: [Walmart Sales Dataset](https://www.kaggle.com/najir0123/walmart-10k-sales-datasets)
   - **Storage**: Save the data in the `data/` folder for easy reference and access.

### 4. Install Required Libraries and Load Data
   - **Libraries**: Install necessary Python libraries using:
     ```bash
     pip install pandas numpy sqlalchemy mysql-connector-python psycopg2
     ```
   - **Loading Data**: Read the data into a Pandas DataFrame for initial analysis and transformations.

### 5. Explore the Data
   - **Goal**: Conduct an initial data exploration to understand data distribution, check column names, types, and identify potential issues.
   - **Analysis**: Use functions like `.info()`, `.describe()`, and `.head()` to get a quick overview of the data structure and statistics.

### 6. Data Cleaning
   - **Remove Duplicates**: Identify and remove duplicate entries to avoid skewed results.
   - **Handle Missing Values**: Drop rows or columns with missing values if they are insignificant; fill values where essential.
   - **Fix Data Types**: Ensure all columns have consistent data types (e.g., dates as `datetime`, prices as `float`).
   - **Currency Formatting**: Use `.replace()` to handle and format currency values for analysis.
   - **Validation**: Check for any remaining inconsistencies and verify the cleaned data.

### 7. Feature Engineering
   - **Create New Columns**: Calculate the `Total Amount` for each transaction by multiplying `unit_price` by `quantity` and adding this as a new column.
   - **Enhance Dataset**: Adding this calculated field will streamline further SQL analysis and aggregation tasks.

### 8. Load Data into MySQL and PostgreSQL
   - **Set Up Connections**: Connect to PostgreSQL using `sqlalchemy` and load the cleaned data into each database.
   - **Table Creation**: Set up tables in PostgreSQL using Python SQLAlchemy to automate table creation and data insertion.
   - **Verification**: Ran initial SQL queries to confirm that the data has been loaded accurately.

### 9. SQL Analysis: Complex Queries and Business Problem Solving
   - **Business Problem-Solving**: Write and execute complex SQL queries to answer critical business questions, such as:
     - Revenue trends across branches and categories.
     - Identifying best-selling product categories.
     - Sales performance by time, city, and payment method.
     - Analyzing peak sales periods and customer buying patterns.
     - Profit margin analysis by branch and category.
   - **Documentation**: Keep clear notes of each query's objective, approach, and results.

---

## Requirements

- **Python 3.8+**
- **SQL Databases**: MySQL, PostgreSQL
- **Python Libraries**:
  - `pandas`, `numpy`, `sqlalchemy`, `psql-connector-python`, `psycopg2`
- **Kaggle API Key** (for data downloading)

## Getting Started

1. Clone the repository:
   ```bash
   git clone <repo-url>
   ```
2. Install Python libraries:
   ```bash
   pip install -r requirements.txt
   ```
3. Set up your Kaggle API, download the data, and follow the steps to load and analyze.

---

## Project Structure

```plaintext
|-- data/                     # Raw data and transformed data
|-- sql_queries/              # SQL scripts for analysis and queries
|-- notebooks/                # Jupyter notebooks for Python analysis
|-- README.md                 # Project documentation
|-- requirements.txt          # List of required Python libraries
|-- main.py                   # Main script for loading, cleaning, and processing data
```
---

## Results and Insights

This section will include your analysis findings:
- **Sales Insights**: Key categories, branches with highest sales, and preferred payment methods.
- **Profitability**: Insights into the most profitable product categories and locations.
- **Customer Behavior**: Trends in ratings, payment preferences, and peak shopping hours.
1) Find the different payment methods, number of transactions and number of quantity sold.
```sq
SELECT 
	payment_method, 
	COUNT(*) AS num_transactions, 
	SUM(quantity) AS quantity_sold
FROM walmart
GROUP BY payment_method;
```
2) Identify the highest-rated category in each branch. Display the branch, category and average rating.
```sq
WITH branch_rank AS (
SELECT 
	branch, 
	category,
	AVG(rating) AS avg_rating, 
	RANK() OVER(PARTITION BY branch ORDER BY AVG(rating) DESC) AS rank
FROM walmart
GROUP BY branch, category
)

SELECT *
FROM branch_rank
WHERE rank = 1;
```
3) Identify the busiest day for each branch based on the number of transactions.
```sq
WITH day_name AS(
SELECT 
	branch, 
	TO_CHAR(TO_DATE(date, 'DD-MM-YY'), 'Day') AS day_name, 
	COUNT(*) AS transaction_count,
	RANK() OVER(PARTITION BY branch ORDER BY COUNT(*) DESC) AS rank
FROM walmart
GROUP BY branch, day_name;
)

SELECT *
FROM day_name
WHERE rank = 1;
```
4) Calculate the total quantity of items sold per payment method. List paymet_method and total_quantity.
```sq
SELECT 
	payment_method, 
	COUNT(*) AS num_sold
FROM walmart 
GROUP BY payment_method;
```
5) Determine the average, minimum, and maximum rating of category for each city. List the city, average_rating, min_rating and max_rating.
```sq
SELECT 
	city, 
	category, 
	AVG(rating) AS avg_rating, 
	MAX(rating) AS max_rating, 
	MIN(rating) AS min_rating
FROM walmart 
GROUP BY city, category
ORDER BY city;
```
6) Calculate the total profit for each category. List the category and total_profit, ordered from highest to lowest. 
```sq
SELECT 
	category,
	ROUND(SUM(total)::numeric, 2) AS revenue, 
	ROUND(SUM(total * profit_margin)::numeric, 2) AS profit
FROM walmart
GROUP BY category
ORDER BY profit DESC;
```
7) Determine the most common payment method for each branch. Display branch and the preferred payment_method.
```sq
WITH payment AS(
SELECT 
	branch, 
	payment_method,
	COUNT(*), 
	RANK () OVER(PARTITION BY branch ORDER BY COUNT(*) DESC) AS rank
FROM walmart
GROUP BY branch, payment_method
ORDER BY branch
)

SELECT 
	branch, 
	payment_method
FROM payment 
WHERE rank = 1;
```
8) Categorize sales into 3 groups - morning, afternoon, evening. Determine shift and the number of invoices for each branch. 
```sq
SELECT 
	branch, 
	CASE 
		WHEN EXTRACT(HOUR FROM (time::time)) < 12 THEN 'Morning'
		WHEN EXTRACT(HOUR FROM (time::time)) BETWEEN 12 AND 17 THEN 'Afternoon'
		ELSE 'Evening'
	END AS shift, 
	COUNT(*) AS num_invoices
FROM walmart
GROUP BY branch, shift
ORDER BY branch, num_invoices DESC;
```
9) Identify 5 branches with highest decrease ratio in revenue and compare it to last year. (current year 2023 and last year 2022)
```sq
-- 2022 sales
WITH revenue_2022 AS(
SELECT
	branch,
	SUM(total) AS revenue
FROM walmart
WHERE EXTRACT(YEAR FROM TO_DATE(date, 'DD-MM-YY')) = 2022
GROUP BY branch
), 
-- 2023 sales
revenue_2023 AS(
SELECT
	branch,
	SUM(total) AS revenue
FROM walmart
WHERE EXTRACT(YEAR FROM TO_DATE(date, 'DD-MM-YY')) = 2023
GROUP BY branch
)

SELECT 
	ls.branch, 
	ls.revenue AS last_year_revenue,
	cs.revenue AS cr_year_revenue,
	ROUND((ls.revenue - cs.revenue)::numeric/ls.revenue::numeric * 100, 2) AS rev_dec_ratio
FROM revenue_2022 AS ls
LEFT JOIN revenue_2023 AS cs
ON ls.branch = cs.branch
WHERE ls.revenue > cs.revenue
ORDER BY rev_dec_ratio DESC
LIMIT 5;
```
---

## Future Enhancements

Possible extensions to this project:
- Integration with a dashboard tool (e.g., Power BI or Tableau) for interactive visualization.
- Additional data sources to enhance analysis depth.
- Automation of the data pipeline for real-time data ingestion and analysis.

---

## License

This project is licensed under the MIT License. 

---

## Acknowledgments

- **Data Source**: Kaggle’s Walmart Sales Dataset
- **Inspiration**: Walmart’s business case studies on sales and supply chain optimization.

---
