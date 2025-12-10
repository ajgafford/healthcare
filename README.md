# Healthcare Analytics, Part 1: Building the Dataset and Exploring Trends in SQL

## Essential Question

> How can patient, medical, and operational data be cleaned and analyzed to provide actionable insights?

## Motivation / Context

Healthcare data is notoriously sensitive, making real-world datasets difficult to access for learning and experimentation. This synthetic dataset provides a safe, realistic way to practice data analysis, modeling, and visualization in the healthcare domain. It includes patient demographics, medical conditions, admission details, billing information, and test results, mimicking the complexity of actual healthcare records.

This project serves as a showcase of end-to-end analytical skills. By working with this dataset, I aim to demonstrate the following:
- Proficiency in data cleaning with Python
- Exploratory Data Analysis (EDA)
- SQL-driven aggregation and insights
- Interactive visualization through Tableau dashboards (Coming in Part 2!)
- Predictive modeling of patient outcomes using machine learning (Coming in Part 3!)

The goal is not only to identify patterns in patient outcomes but also to provide actionable insights that could inform decision-making in a healthcare setting.

Through this project, the reader will see how structured analysis and thoughtful visualization can transform raw, complex data into meaningful insights.

## Dataset Description

This project uses a synthetic healthcare dataset available on Kaggle, originally published by the user “prasad22.” The dataset is designed for educational and analytic purposes and mirrors common attributes found in real‑world healthcare records while avoiding privacy issues.

The dataset is **synthetic but realistic**, designed to simulate a medium-sized hospital or healthcare system. It contains **over 40,000 patient records** spanning multiple years (2019–2024) and includes the following key fields:
- **Patient demographics:** `Name`, `Gender`, `Age`, `Age Group`, `Blood Type`
- **Admissions data:** `Date of Admission`, `Discharge Date`, `Admission Type` (e.g., elective, urgent, emergency)
- **Medical data:** `Medical Condition`, `Test Results`
- **Billing information:** `Billing Amount`, `Billing per Day` (engineered)
- **Operational features:** `Length of Stay`, temporal features derived from dates (Year, Month, Weekday)
- **Insurance information:** `Insurance Provider`

This combination of demographic, medical, operational, and financial data enables **end-to-end analysis**, from cleaning and feature engineering to SQL-based aggregation, visualization, and predictive modeling.
## Preprocessing / Cleaning the Dataset

To prepare the dataset for analysis, visualization, and modeling, the following preprocessing steps were completed:

- **Checked for missing values:**
	- Verified the dataset contains no null entries in any column.
	- If nulls had been present, appropriate imputation or removal strategies would have been applied depending on the field.
- **Standardized the `Name` Column:**
	- Corrected inconsistent capitalization using `.str.title()`
	- While the `Name` of the patient will likely not be used for analysis, having a clean, consistent look will make the dashboard and other visualizations look more professional.
- **Converted date fields to proper `datetime` datatypes:**
	- `Date of Admission` and `Discharge Date` were cast to `datetime64[ns]`.
	- Enables time-based operations such as calculating length of stay, sorting chronologically, and building time-based features.
- **Identified and removed duplicate records:**
	- Searched for fully duplicated rows and dropped them to ensure data integrity and avoid bias in downstream modeling.
- **Converted select fields to categorical datatype:**
	- Columns representing discrete groups were cast to `category`.
	- This improves memory efficiency and clarifies which fields are qualitative features for later modeling.

## Feature Engineering

To prepare the dataset for analysis, visualization, and modeling, the following features were added:

- `Length of Stay`:
	- Calculated as the difference in days between `Discharge Date` and `Date of Admission`
	- Provides insight into patient stay duration and supports operational analysis.
- Year, Month, and Weekday for each of `Date of Admission` and `Discharge Date`:
	- Enables exploration of temporal trends, seasonal patterns, and day-of-week effects in admissions and discharges.
- `Billing per Day`:
	- Calculated by dividing `Billing Amount` by `Length of Stay`.
	- Normalizes billing data to allow fair comparison between patients with different lengths of stay.
- `Age Group`:
	- Patients were categorized into: Child (0–12), Teen (13–18), Young Adult (19–35), Adult (36–65), and Senior (65+).
	- Supports demographic analysis and improves readability in dashboards and visualizations.

## SQL and Interpretation

For this stage of the project, I used **PostgreSQL** as the database platform and **DBeaver** as the GUI client. The processed and cleaned CSV was imported into a database called `healthcare` as a table called `patients`. Below are some of my queries and insights:

### 1. How many unique patients are represented in this dataset?

```sql
SELECT COUNT(DISTINCT Name) AS total_patients FROM patients;
```

**Insights:** There are 40,235 *unique* patients within this dataset.

### 2. What are the total number of visits by year?

```sql
SELECT "Admission Year" AS year,
	COUNT(*) AS total_visits
FROM patients
GROUP BY year
ORDER BY year ASC;
```

**Insights:** 2020 saw the most total visits at 11,172. This was a large increase from 2019, which saw only 7,300 total visits. From 2021 through 2023, the number of visits stayed in the upper 10-thousands. Roughly the first half of 2024 is in this dataset, which explains why only 3,827 records exist in that year.

![[healthcare_sql_query_02.png]]

### 3. How many patients are in each age group?

```sql
SELECT "Age Group",
	COUNT(*) AS total_visits
FROM patients
GROUP BY "Age Group"
ORDER BY total_visits DESC;
```

**Insights:** Adults (36-65) make up most visits with 24,465. No children are represented in this dataset.

![[healthcare_sql_query_03.png]]

### 4. How many patients have been admitted by type of care?

```sql
SELECT "Admission Type",
	COUNT(*) AS total_visits
FROM patients
GROUP BY "Admission Type"
ORDER BY total_visits DESC;
```

**Insights:** Visits are pretty much uniformly distributed across admission type.

![[healthcare_sql_query_04.png]]

### 5. What is the total and average amount billed?

```sql
SELECT ROUND(SUM("Billing Amount")) as total_billed,
	ROUND(AVG("Billing Amount")) as average_billed
FROM patients;
```

**Insights:** The total amount billed over all patients in the dataset is about $1.4 billion. The average amount billed per visit is $25,544.

### 6. How many patients are male vs. female?

```sql
SELECT Gender,
	COUNT(DISTINCT "Name") as total_patients
FROM patients
GROUP BY Gender
ORDER BY total_patients;
```

**Insights:** The total number of *unique* patients by gender is pretty much uniformly distributed, with females slightly outnumbering the males. There are 21,897 females versus 21,809 males.

### 7. What is the average length of a stay for each admission type?

```sql
SELECT "Admission Type",
	ROUND(AVG("Length of Stay"), 1) AS average_length
FROM patients
GROUP BY "Admission Type"
ORDER BY average_length DESC;
```

**Insights:** The average duration of a stay (in days) is pretty much uniform across the different admission types.

![[healthcare_sql_query_07.png]]

### 8. How many patients had each type of test result by blood type?

```sql
SELECT "Blood Type",
	"Test Results",
	COUNT(*) as number_of_results
FROM patients
GROUP BY "Blood Type", "Test Results"
ORDER BY "Blood Type", number_of_results DESC;
```

**Insights:** There does not appear to be a meaningful difference in test results by blood type. The result of the test seems to be not depend on the patient's blood type.

![[healthcare_sql_query_08.png]]

### 9. Which condition has the highest average bill?

```sql
SELECT "Medical Condition",
	ROUND(AVG("Billing Amount")) AS average_bill
FROM patients
GROUP BY "Medical Condition"
ORDER BY average_bill DESC
LIMIT 1;
```

**Insights:** Obesity has the highest average bill with $25,804 per visit.

### 10. Which individual patient visits had the highest billing amounts relative to the average billing for their admission type?

```sql
SELECT "Admission Type",
	"Medical Condition",
	"Length of Stay",
	ROUND("Billing Amount") AS amount_billed,
	ROUND(
		AVG("Billing Amount") OVER (PARTITION BY "Admission Type"),
	) AS avg_admission_bill,
	ROUND(
		"Billing Amount" - 
		AVG("Billing Amount") OVER (PARTITION BY "Admission Type")
	) AS diff_from_avg_bill
FROM patients
ORDER BY diff_from_avg_bill DESC
LIMIT 10;
```

**Insights:** The average bill, regardless of admission type, is around $25,000. The top 10 largest differences between the actual and average bill are more than double the average billing amount. There is a mix of conditions and lengths of stay, but the admission type does tend to be non-elective.

![[healthcare_sql_query_10.png]]

### 11. For each patient, calculate the difference between their amount billed and their own average amount billed across all visits. Filter out anyone who has 2 or fewer visits and has a difference of 0.

```sql
WITH patient_diff AS (
	SELECT "Name",
	"Admission Type",
	ROUND("Billing Amount") AS amount_billed,
	ROUND(
		AVG("Billing Amount") OVER (PARTITION BY "Name")
	) AS avg_bill,
	ROUND(
		100 * (
			"Billing Amount" - AVG("Billing Amount")
		) / AVG("Billing Amount") OVER (PARTITION BY "Name")
	) AS pct_from_avg_bill,
	COUNT(*) OVER (PARTITION BY "Name") AS num_visits
)
SELECT *
FROM patient_diff
WHERE num_visits >= 3
ORDER BY pct_from_avg_bill DESC
LIMIT 10;
```

**Insights:** The distribution shows a small number of extreme billing outliers. For some patients, individual visits cost **over triple** their average bill (a >200% increase). These events are disproportionately associated with **non-elective admission types**, suggesting that unexpected or urgent care drives the largest deviations from expected billing patterns.

![[healthcare_sql_query_11.png]]
### 12. Compute a running total of amount billed per month, across the years in the dataset where records are complete.

```sql
WITH monthly_totals AS (
	SELECT "Admission Year",
		"Admission Month",
		SUM("Billing Amount") as total_billed
	FROM patients
	GROUP BY "Admission Year", "Admission Month"
	ORDER BY "Admission Year", "Admission Month"
)
SELECT *,
	SUM(total_billed) OVER (
		PARTITION BY "Admission Year"
		ORDER BY "Admission Month"
	) AS running_total
FROM monthly_totals
-- Restrict to only the years where records are complete.
WHERE "Admission Year" BETWEEN 2020 AND 2023;
```

**Insights:** Despite month-to-month volatility, the total amount billed at year-end is remarkably consistent, hovering around $270–$280 million each year. This suggests the healthcare system operates with a stable patient volume and reimbursement structure year-to-year, and that short-term fluctuations tend to balance out over the course of the year.

### 13. Compute the running total of patient visits per month, across the years in the dataset where records are complete.

```sql
WITH monthly_totals AS (
	SELECT "Admission Year",
		"Admission Month",
		COUNT(*) as total_patients
	FROM patients
	GROUP BY "Admission Year", "Admission Month"
	ORDER BY "Admission Year", "Admission Month"
)
SELECT *,
	SUM(total_patients) OVER (
		PARTITION BY "Admission Year"
		ORDER BY "Admission Month"
	) AS running_total
FROM monthly_totals
-- Restrict to only the years where records are complete.
WHERE "Admission Year" BETWEEN 2020 AND 2023;
```

**Insights:** Patient volume appears highly stable over time, with monthly admissions consistently landing in the ~870–1,000 range. Annual totals converge around 10,800–11,200 patients each year, showing less than ~3% variation year-to-year. This aligns with the billing trend. The healthcare system seems to operate at a predictable capacity, where fluctuations at the monthly level smooth out over the full year.

### 14. Compute the percent change in the amount billed per month, across all years.

```sql
WITH monthly_totals AS (
    SELECT 
        "Admission Year",
        "Admission Month",
        SUM("Billing Amount") AS total_billed
    FROM patients
    GROUP BY "Admission Year", "Admission Month"
    ORDER BY "Admission Year", "Admission Month"
)
SELECT *,
       ROUND(
           100 * (
	           total_billed - 
	           LAG(total_billed) OVER (
		           PARTITION BY "Admission Year" 
		           ORDER BY "Admission Month"
		        )
		    )
           / LAG(total_billed) OVER (
	           PARTITION BY "Admission Year" 
	           ORDER BY "Admission Month"
		     )
       ) AS pct_change
FROM monthly_totals;
```

**Insights:** The month-to-month fluctuations in billing tend to be less than 10% in either direction. Combined with stable patient volumes, this indicates that the healthcare system maintains a consistent flow of patients and predictable revenue patterns throughout the year.

### 15. Which insurance providers are most represented in the sample?

```sql
SELECT "Insurance Provider",
	SUM("Billing Amount") AS total_billed
FROM patients
GROUP BY "Insurance Provider"
ORDER BY total_billed;
```

**Insights:** The total amount billed in the sample is pretty much uniform across the 5 major insurance providers.

![[healthcare_sql_query_15.png]]



## Interactive Tableau Dashboard (Coming Soon in Part 2!)

The dashboard will allow users to filter by year, admission type, or insurance provider to dynamically explore patterns in patient visits and billing.

Stay tuned for Part 2, where these queries will come to life in a fully interactive dashboard!
