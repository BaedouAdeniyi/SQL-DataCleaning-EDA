# Data Cleaning and Exploratory Data Analysis on Layoff Dataset

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
- **Result**:

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
- **Result**:

### Step 6: Correcting Data types for all columns
- **Goal**:
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
- **Result**

### Step 7 : Reviewing and correcting misspellings 
- **Goal**:
- SQL Code
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
- **Result**

### Step : 
- **Goal**:
- SQL Code
```
```
- **Result**

### Step : 
- **Goal**:
- SQL Code
```
```
- **Result**

### Step : 
- **Goal**:
- SQL Code
```
```
- **Result**

### Step : 
- **Goal**:
- SQL Code
```
```
- **Result**





