# Data Cleaning and Exploratory Data Analysis on Layoff Dataset


## Table of Contents

- [Project Description](#project-description)
- [Data Source](#data-source)
- [Dataset Overview](#dataset-overview)
- [Project Objectives](#project-objectives)
- [Project Workflow](#project-workflow)
  - [DATA CLEANING PROCESS](#data-cleaning-process)
  - [EXPLORATORY DATA ANALYSIS (EDA)](#exploratory-data-analysis-(eda)) 
- [Tools and Technologies](#tools-and-technologies)
- [Conclusion](#conclusion)

## Project Description
This project demonstrates the process of cleaning and analyzing a dataset containing employee layoff information across various companies and countries. It includes SQL workflows for data cleaning, handling missing values, standardizing text and numeric columns, and performing exploratory data analysis (EDA) to uncover trends by company, industry, and country.


## Data Source
The layoff dataset was collected from [Kaggle: Layoff Data](https://www.kaggle.com/datasets/theakhilb/layoffs-data-2022)
It contains records of employee layoffs from various companies, industries, and countries.


## Dataset Overview
This dataset contains layoff records from global companies, capturing key details such as:
- Company information — company name, country, and industry
- Layoff metrics — number of employees laid off and the percentage of the workforce affected
- Financial data — total funds raised by each company
- Timestamp fields —
  - Date — general date of the layoff event
  - Date_Added — the exact date and time the layoff record was captured
The first few rows of the dataset provide a snapshot of the structure and type of information included.

| Company | Location_HQ	| Industry	| Laid_Off_Count |	Date	| Source	| Funds_Raised	| Stage	| Date_Added	| Country	| Percentage	| List_of_Employees_Laid_Off |
|-----|----|-----|----|-----|----|-----|----|-----|----|-----|----|
| Oda | Oslo | Food | 150 | 05/06/2024 |https://techcrunch.com/2024/06/05/softbank-backed-grocery-startup-oda-lays-off-150-resets-focus-on-norway-and-sweden/ | 691 | Unknown | 05/06/2024 18:01 | Norway | | Unknown |
| Pagaya	| Tel Aviv	| Finance	| 100 |	05/06/2024	| https://www.calcalistech.com/ctechnews/article/h19floaec	| 2000	| Post-IPO |	05/06/2024 23:11	| Israel	| 0.2	| Unknown |
| Aleph Farms	| Tel Aviv	| Food	| 30	| 05/06/2024	| https://www.calcalistech.com/ctechnews/article/hkdwxv0na	| 119	| Unknown	| 05/06/2024 23:13	| Israel	| 0.3	| Unknown |
| MoonPay	| Dover	| Crypto	| 30	| 05/06/2024	| https://www.theblock.co/post/298638/moonpay-lays-off-10-staff	| 651	| Unknown	| 05/06/2024 23:12	| United States	| 0.1	| Unknown |


## Project Objectives
- Clean and standardize the dataset to ensure accurate analysis
- Handle missing and inconsistent data
- Convert all columns to appropriate data types
- Perform exploratory data analysis to identify trends and insights

  

## Tools and Technologies
- SQL (MySQL) for data cleaning and analysis
- GitHub to host the project


## Project Workflow
### **DATA CLEANING PROCESS**
### Step 1: View Raw Dataset
- **Goal:** Inspect the dataset to understand structure of the dataset.
-   SQL Code: 
```sql
SELECT * FROM layoffs_data;
```
- **Result**: Raw data displayed, identifing the structure and content of the dataset.


### Step 2: Create a Copy to work on 
- **Goal**: Protect the original data before cleaning.
- SQL Code:
```sql
CREATE TABLE layoffs_clean
LIKE layoffs_data;
```
```sql
INSERT layoffs_clean
SELECT * FROM layoffs_data;
```
- **Result**: A new copy 'layoff_clean' was created for analysis


### Step 3: Check for duplicate records 
- **Goal**:
- SQL Code
```sql
WITH CTE_layoffs_clean AS
(
SELECT *, ROW_NUMBER() OVER(PARTITION BY Company, Location_HQ, Industry, Laid_Off_Count, `Date`, `Source`, Funds_Raised, Stage, Date_Added, Country, Percentage, List_of_Employees_Laid_Off  ) AS row_num
FROM layoffs_clean
)
SELECT * FROM CTE_layoffs_clean 
WHERE row_num > 1;
```
- **Result**: Confirmed that no duplcate entries existed in this dataset


### Step 4: Standardizing Text Columns
- **Goal**: To remove extra spaces from all the text column
- SQL Code
```sql
UPDATE layoffs_clean
SET company = TRIM(company),
	Location_HQ = TRIM(Location_HQ),
	Industry = TRIM(Industry),
    Stage = TRIM(Stage),
    Country = TRIM(Country),
    `Source` = TRIM( `Source`),
    List_of_Employees_Laid_Off = TRIM(List_of_Employees_Laid_Off);
```
- **Result**: All unnecessary spaces were successfully removed from the selected text columns, ensuring consistent formatting and improving the reliability of downstream analysis.

### Step 5 : Handling missing and blank values 
- **Goal**: To replace all empty or blank cells with null values in columns with missing and blank values for consistency and accurate analysis 
- SQL Code
```sql
UPDATE layoffs_clean
SET Funds_Raised = null
WHERE Funds_Raised = '';
```
```sql
UPDATE layoffs_clean
SET Laid_Off_Count = null
wHERE Laid_Off_Count = '';
```
```sql
UPDATE layoffs_clean
SET percentage = null
wHERE percentage = '';
```
- **Result**: All identified blank or empty entries were successfully updated to NULL, making the dataset cleaner, more structured, and easier to work with during querying and analysis.

### Step 6: Correcting Data types for all columns
- **Goal**: To ensure that every column in the dataset was assigned the appropriate data type for accurate computation, proper filtering, and reliable analysis.
- SQL Code to Identify data type for all columns
```sql
DESCRIBE layoffs_clean;
```
- SQL Code to remove trailing ".0" value to allow conversion in "Funds_Raised" column 
```sql
UPDATE layoffs_clean
SET Funds_Raised =  LEFT(Funds_Raised, LENGTH(Funds_Raised) - 2)
WHERE RIGHT(Funds_Raised, 2) = '.0';
```
- SQL Code to upadting all columns to correct data type
```sql
ALTER TABLE layoffs_clean
MODIFY COLUMN Company VARCHAR(100),
MODIFY COLUMN Location_HQ VARCHAR(100),
MODIFY COLUMN Industry VARCHAR(100),
MODIFY COLUMN `Date` Date,
MODIFY COLUMN Funds_Raised INT,
MODIFY COLUMN Laid_Off_Count INT,
MODIFY COLUMN Percentage DECIMAL(6,2),
MODIFY COLUMN `Source` VARCHAR(500),
MODIFY COLUMN Stage VARCHAR(100),
MODIFY COLUMN Country VARCHAR(100),
MODIFY COLUMN List_of_Employees_Laid_Off VARCHAR(255),
MODIFY COLUMN Date_Added DATETIME;
```
- **Result**: All columns were successfully converted to their correct data types. This correction improved the dataset’s integrity and ensured that subsequent analysis could be performed without errors or inconsistencies.


### Step 7 : Reviewing and correcting misspellings 
- **Goal**: To identify and correct any misspelled entries in the text-based columns, ensuring consistency and accuracy across the dataset.
- SQL Codes to review distinct values in key text columns
```sql
SELECT DISTINCT(company) FROM layoffs_clean
ORDER BY 1;
```
```sql
SELECT DISTINCT(industry) FROM layoffs_clean
ORDER BY 1;
```
```sql
SELECT DISTINCT(country) FROM layoffs_clean
ORDER BY 1;
```
```sql
SELECT DISTINCT(Location_HQ) FROM layoffs_clean
ORDER BY 1;
```
- SQL Codes to to identify similr inconsistent entries
```sql
SELECT * FROM layoffs_clean
WHERE Company like 'Ada%' and industry = 'Support';
```
```sql
SELECT * FROM layoffs_clean
WHERE Company like 'Air%';
```
```sql
SELECT * FROM layoffs_clean
WHERE Company like 'Apollo%';
```
```sql
SELECT * FROM layoffs_clean
WHERE Company like 'Bolt%';
```
```sql
SELECT * FROM layoffs_clean
WHERE Company like 'connect%';
```
- SQL Code to correct detected misspelings for uniformity
```sql 
UPDATE layoffs_clean
SET Company = 'Ada Support' 
WHERE Company LIKE 'Ada%' AND industry = 'Support';
```
- **Result**: Reviewed distinct values in key text columns and used pattern-matching techniques (including the LIKE operator) to detect similar but inconsistent entries. After careful validation, all identified misspellings and inconsistencies were corrected, resulting in cleaner and more reliable categorical data.


### **EXPLORATORY DATA ANALYSIS (EDA)**
### Step 1: Quick overview of table structure 
- **Goal**: To examine the structure of the dataset, including column names, data types, and overall layout, ensuring familiarity with the data before starting cleaning and analysis
- SQL Code
```sql
DESCRIBE layoffs_clean;
```

### Step 2: Summary statistics for numerical columns
- **Goal**: To summarize key numerical columns — total layoffs, percentage affected, and funds raised — using metrics like minimum, maximum, and average, in order to understand the scale and variability of workforce reductions.
- SQL Codes
```sql
SELECT 
	COUNT(*) AS total_records,
    COUNT(DISTINCT `Laid_Off_Count`) AS unique_values,
    MIN(Laid_Off_Count) AS min_laid_off,
    MAX(Laid_Off_Count) AS max_laid_off,
	AVG(Laid_Off_Count) AS avg_laid_off,
	SUM(Laid_Off_Count) AS total_laid_off
FROM layoffs_clean;
```
```sql
SELECT 
	COUNT(*) AS total_records,
    COUNT(DISTINCT percentage) AS unique_values,
    MIN(percentage) AS min_percentage,
    MAX(percentage) AS max_percentage,
	AVG(percentage) AS avg_percentage
FROM layoffs_clean;
```
```sql
SELECT 
	COUNT(*) AS total_records,
    COUNT(DISTINCT Funds_Raised) AS unique_values,
    MIN(Funds_Raised) AS min_fund_raised,
    MAX(Funds_Raised) AS max_fund_raised,
	AVG(Funds_Raised) AS avg_fund_raised
FROM layoffs_clean;
```
- **INSIGHT**: The dataset contains 3,642 records capturing global layoff events.

Total Laid-Off Employees: Layoffs range from 3 to 14,000 employees, with an average of ~258, showing that most reductions are moderate but some are very large. In total, 616,186 employees were affected.

Percentage of Workforce Laid Off: The layoff percentage ranges from 0% to 100%, with an average of ~28%, highlighting that companies vary greatly in how much of their workforce they reduce. Some records are missing, reflecting incomplete reporting.

Funds Raised: Funding varies from 0 to 121,900 (currency not specified), with some missing values. This provides context for understanding layoffs relative to company financial backing.

Overall, the summary statistics give a clear picture of the scale and variability of layoffs, the percentage impact on workforces, and the financial context across companies.

### Step 3 : Summary statistics for categorical columns
- **Goal**: To summarize the key categorical columns in the dataset—such as company, country, industry, stage, and HQ location by counting the occurrences of each unique value, in order to understand the distribution and identify patterns across these categories.
- SQL Codes
```sql
SELECT company, COUNT(*) AS frequency
FROM layoffs_clean
GROUP BY company
ORDER BY frequency DESC;
```
```sql
SELECT country, COUNT(*) AS frequency
FROM layoffs_clean
GROUP BY country
ORDER BY frequency DESC;
```
```sql
SELECT Location_HQ, COUNT(*) AS frequency
FROM layoffs_clean
GROUP BY Location_HQ
ORDER BY frequency DESC;
```
```sql
SELECT stage, COUNT(*) AS frequency
FROM layoffs_clean
GROUP BY stage
ORDER BY frequency DESC;
```
- **INSIGHT**: Most layoffs in the dataset are concentrated among a few companies and locations. Google and Amazon top the list with 12 records each, while most companies appear only once. The United States dominates the country-wise distribution, followed by India, Canada, the UK, and Germany. Layoffs span all company stages, with Post-IPO firms reporting the most, followed by Series B, Series C, and Unknown stages. Looking at headquarters, major tech hubs stand out: SF Bay Area, New York City, Boston, Bengaluru, Los Angeles, and Seattle, with London and Tel Aviv also featuring. Overall, layoffs are heavily concentrated in U.S.-based companies and tech hubs, though select international cities are also impacted.

### Step 4 : Missing data overview 
- **Goal**: To assess the completeness of the dataset by identifying columns with missing or null values. This helps understand potential gaps in the data and informs how missing information may impact subsequent analysis.
- SQL Code
```sql
SELECT 
	SUM(
		CASE 
			WHEN Laid_Off_Count IS NULL THEN 1 
            ELSE 0
		END) AS missing_layoff_count,
	SUM(
		CASE 
			WHEN percentage IS NULL THEN 1 
            ELSE 0
		END) AS missing_percentage,
	SUM(
		CASE 
			WHEN Funds_Raised IS NULL THEN 1 
            ELSE 0
		END) AS fund_raised_missing,
	SUM(
		CASE 
			WHEN company IS NULL THEN 1 
            ELSE 0
		END) AS missing_company,
	SUM(
		CASE 
			WHEN industry IS NULL THEN 1 
            ELSE 0
		END) AS missing_industry,
	SUM(
		CASE 
			WHEN country IS NULL THEN 1 
            ELSE 0
		END) AS missing_country,
	SUM(
		CASE 
			WHEN Location_HQ IS NULL THEN 1 
            ELSE 0
		END) AS missing_location_hq,
	SUM(
		CASE 
			WHEN `Date` IS NULL THEN 1 
            ELSE 0
		END) AS missing_date
FROM layoffs_clean;
```
- **INSIGHT**: A significant portion of the numerical data is missing, especially in the layoff count(1,253 records) and percentage column(1,300 records). This highlights areas where reporting was incomplete or unavailable, which may affect certain analyses. The text columns, by contrast, are complete, providing reliable categorical information for grouping, filtering, and trend analysis.


### Step 5: Frequency analysis & grouping
- **Goal**: To identify the most common companies, countries, and industries affected by layoffs by examining the top frequency counts in each categorical column. This helps highlight where layoffs are concentrated and reveals patterns across organizations, regions, and sectors.
- SQL Code for top 10 companies by total layoffs
```sql
SELECT Company, SUM(Laid_Off_Count) AS total_laid_off
FROM layoffs_clean
GROUP BY Company
ORDER BY total_laid_off DESC
LIMIT 10;
```
- SQL Code for top 10 countries by total layoffs
```sql
SELECT country, SUM(Laid_Off_Count) AS total_laid_off
FROM layoffs_clean
GROUP BY country
ORDER BY total_laid_off DESC
LIMIT 10;
```
- SQL Code for top 10 industries by total layoffs
```sql
SELECT industry, SUM(Laid_Off_Count) AS total_laid_off
FROM layoffs_clean
GROUP BY Industry
ORDER BY total_laid_off DESC
LIMIT 10;
```
- SQL Code for top 10 Location_hq by total layoffs
```sql
SELECT location_hq, SUM(Laid_Off_Count) AS total_laid_off
FROM layoffs_clean
GROUP BY location_hq
ORDER BY total_laid_off DESC
LIMIT 10;
```
- **INSIGHT**: As part of the exploratory data analysis, we examined the top 10 values for key categorical columns — company, country, and industry — to understand where layoffs are concentrated.
  - Top Companies by Layoffs:
    The companies with the most reported layoffs include Amazon (27,840), Meta (21,000), Tesla (14,500), Microsoft (14,058), and Google (13,472).
Insight: Layoffs are concentrated in large, well-known tech companies, indicating that high-profile firms experienced the largest workforce reductions in absolute numbers.
  - Top Countries by Layoffs:
    The majority of layoffs occurred in the United States (414,013), followed by India (51,234), Germany (26,353), United Kingdom (19,769), and Netherlands (18,445).
Insight: Layoffs are heavily skewed toward the U.S., reflecting either the size of the workforce, the number of companies reporting, or both. Other countries show smaller but notable layoff events, highlighting regional trends.
  - Top Industries by Layoffs:
    The industries with the most layoffs include Retail (70,157), Consumer (67,675), Transportation (59,417), Other (59,261), and Food (45,625).
Insight: Layoffs are concentrated in sectors like retail, consumer goods, and transportation, which may indicate industry-specific challenges or trends affecting workforce reductions.
  - Top HQ_location by Layoffs: 
    The largest layoffs are concentrated in major headquarters locations. The SF Bay Area leads by a wide margin with 184,151 layoffs, followed by Seattle (53,562), New York City (35,829), and Austin (33,545). Bengaluru (30,207), London (19,451), Amsterdam (18,065), Boston (16,591), Berlin (13,786), and Stockholm (13,692) complete the top ten. This highlights that U.S. tech hubs dominate layoff counts, while select international cities also face significant workforce reductions.

### Step 6:  Data anaylsis (Layoff date)
- **Goal**: To analyze layoff events over time, examining trends by year and month, and calculating cumulative monthly layoffs to understand how workforce reductions accumulate over time.
- SQL Code for layoff per year
```sql
SELECT YEAR(`Date`) AS year, SUM(Laid_Off_Count) AS total_laid_off
FROM layoffs_clean
GROUP BY YEAR(`Date`) 
ORDER BY 1;
```
- SQL Code for layoff per month
```sql
SELECT YEAR(`Date`) AS year,  MONTH(`Date`) AS month_num, SUM(Laid_Off_Count) AS total_laid_off
FROM layoffs_clean
GROUP BY YEAR(`Date`), MONTH(`Date`)
ORDER BY 1;
```
- SQL Code for cummulative layoffs over time (Monthly)
```sql
WITH layoffs_clean_CTE AS
(
SELECT YEAR(`Date`) AS year, MONTH(`Date`) AS month_num, SUM(Laid_Off_Count) AS total_laid_off 
FROM layoffs_clean
GROUP BY YEAR(`Date`), MONTH(`Date`)
ORDER BY 1
)
SELECT *, SUM(total_laid_off) OVER(ORDER BY `year`, `month_num` ) AS cummulative_laid_off
FROM layoffs_clean_CTE;
```
- **INSIGHT**
  - Yearly Trends: Layoffs fluctuate significantly across years. The largest number of layoffs occurred in 2023 (263,180), followed by 2022 (165,269) and 2024 (90,916 so far). Surprisingly, 2020 — the year COVID-19 began — does not have the highest layoffs, which could indicate either underreporting in the dataset or that post-pandemic economic adjustments led to more workforce reductions in later years.
  - Monthly Trends: Layoffs vary month to month within each year. For example, in 2023, the highest monthly layoffs were in January (89,709), while the lowest were in September (4,707). This demonstrates seasonal or event-driven fluctuations in workforce reductions.
  - Cumulative Trends: By calculating cumulative monthly layoffs, we track the total workforce reductions over time. As of June 2024, a total of 616,186 employees have been laid off across all recorded events, showing a clear upward accumulation of layoffs.
- **Key Takeaway**
The data highlights both short-term spikes in layoffs and long-term accumulation, revealing that layoffs are influenced by multiple factors which includes industry trends, economic shifts, and post-pandemic adjustments, rather than the pandemic alone.

### Step 7: Outlier detection
- **Goal**: To identify unusually high layoff events, defined as single layoffs greater than 5,000 employees, in order to highlight extreme workforce reductions.
- SQL Code
```sql
 SELECT `type`, `value`, frequency
 FROM
 (
 select 'company' AS `type`,
		Company AS `value`,
        count(*) as frequency
  from layoffs_clean
 where Laid_Off_Count> 5000
 group by company
 
  union all
  
  select 'country' AS `type`,
		Country AS `value`,
        count(*) as frequency
  from layoffs_clean
 where Laid_Off_Count> 5000
 group by country
 
 union all 
 
 select 'industry' AS `type`,
		Industry AS `value`,
        count(*) as frequency
  from layoffs_clean
 where Laid_Off_Count> 5000
 group by industry
 
 union all
 
 select 'Location_HQ' AS `type`,
		Location_HQ AS `value`,
        count(*) as frequency
  from layoffs_clean
 where Laid_Off_Count> 5000
 group by Location_HQ
 ) as combined
order by `type`, frequency desc;
```
- **INSIGHT**:There are 15 records where layoffs exceeded 5,000 employees. Amazon leads with 3 such occurrences, followed by Dell and Meta with 2 each. Other companies include Tesla, SAP, Flink, Ericsson, Philips, Google, Microsoft, and Salesforce. Most of these high layoffs occurred in the United States (11), with Germany (2), Sweden (1), and the Netherlands (1) also represented. By industry, Retail, Consumer, and “Other” dominate. The top headquarters affected are Seattle and SF Bay Area (4 each), followed by Austin (3), with Walldorf, Berlin, Stockholm, and Amsterdam also impacted.

## Conclusion
This project demonstrates the full lifecycle of working with a real-world dataset using SQL, from data cleaning to exploratory analysis. Through systematic cleaning, I ensured that all numeric, text, and date columns were properly formatted, missing or blank values were handled consistently, and potential inconsistencies and outliers were identified and corrected.

The exploratory analysis provided insights into global layoff trends across companies, industries, countries, and headquarters locations. Key observations include the variation in layoffs over time, identification of unusually high layoff events, and patterns in workforce reductions across sectors and geographies.

By combining SQL queries with thoughtful interpretation, this project showcases not only technical proficiency in SQL but also the ability to derive meaningful insights from raw data. This foundation can be further expanded with advanced analytics or visualization tools for deeper business insights.


