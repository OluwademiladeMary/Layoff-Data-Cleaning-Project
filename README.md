# Layoff-Data-Cleaning-Project

-- MySQL Project - Data Cleaning

## Project Overview
The goal of this project was to clean and preprocess a dataset by identifying and resolving inconsistencies, ensuring data accuracy, and improving overall data quality for effective analysis.

## Data Source
-- https://www.kaggle.com/datasets/swaptr/layoffs-2022

## Steps

```sql
SELECT * 
FROM layoffs;
```
### Created a staging table. This is the one where I cleaned and worked on the data.
```sql
CREATE TABLE layoffs_staging
LIKE layoffs;

INSERT layoffs_staging
SELECT *
FROM layoffs;
```

### Remove Duplicates
```sql
SELECT *
FROM layoffs_staging;

SELECT *,
  ROW_NUMBER() OVER(
  PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised_millions) as row_num
FROM layoffs_staging;

WITH duplicate_cte as
  (SELECT *,
  ROW_NUMBER() OVER(
  PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised_millions) as row_num
  FROM layoffs_staging
  )
SELECT *
FROM duplicate_cte
WHERE row_num > 1;
```

I created a new column nad added those row numbers in. Then deleted where row numbers are over 2, then delete te columns. 

```sql
ALTER TABLE layoffs.layoffs_staging ADD row_num INT;


SELECT *
FROM layoffs.layoffs_staging
;

CREATE TABLE `layoffs_staging2` (
  `company` text,
  `location` text,
  `industry` text,
  `total_laid_off` int DEFAULT NULL,
  `percentage_laid_off` text,
  `date` text,
  `stage` text,
  `country` text,
  `funds_raised_millions` int DEFAULT NULL,
  `row_num` INT
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

ELECT *
FROM layoffs_staging2;

INSERT INTO layoffs_staging2
select *,
ROW_NUMBER() OVER(
PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, 'date', stage,
 country, funds_raised_millions) as row_num
FROM layoffs_staging;
```

Then deletes rows were row_num is greater than 2
```sql
SELECT *
FROM layoffs_staging2
WHERE row_num > 1;

DELETE
FROM layoffs_staging2
WHERE row_num > 1;
```

### Standardizing the data
```sql
SELECT *
FROM layoffs_staging2;

SELECT company, trim(company)
FROM layoffs_staging2;

UPDATE layoffs_staging2
SET company = trim(company); 

SELECT DISTINCT industry
FROM layoffs_staging2; 

UPDATE layoffs_staging2
SET industry = 'Crypto'
WHERE industry LIKE 'Crypto%' ;

SELECT DISTINCT country 
FROM layoffs_staging2
ORDER BY 1;  

SELECT *
FROM layoffs_staging2
WHERE country LIKE 'United States%'
ORDER BY 1;  

SELECT distinct country,  trim(TRAILING '.' FROM country)
FROM layoffs_staging2
ORDER BY 1;  

UPDATE layoffs_staging2
SET country = trim(TRAILING '.' FROM country)
WHERE country LIKE 'United States%' ;
```

### Fixing the date column
```sql
SELECT `date`,
STR_TO_DATE (`date`, '%m/%d/%Y' )
FROM layoffs_staging2;

UPDATE layoffs_staging2
SET `date` = STR_TO_DATE (`date`, '%m/%d/%Y' );

ALTER TABLE layoffs_staging2
MODIFY COLUMN `date` DATE;
```

### Null Values
The  null values in total_laid_off, percentage_laid_off, and funds_raised_millions all look normal.

### Removed unwanted Rows and Columns
```sql
SELECT *
FROM layoffs_staging2
WHERE total_laid_off IS NULL
AND percentage_laid_off IS NULL;

SELECT *
FROM layoffs_staging2
WHERE industry IS NULL
OR industry = '';

UPDATE layoffs_staging2
SET industry = null
where industry = '';
 
SELECT *
FROM layoffs_staging2
WHERE company LIKE 'Bally%';

SELECT *
FROM layoffs_staging2 t1
JOIN layoffs_staging2 t2
	ON t1.company = t2.company
    AND t1.location = t2.location
WHERE (t1.industry IS NULL OR t1.industry = '')
AND t2.industry IS NOT NULL;		


UPDATE layoffs_staging2 t1
JOIN layoffs_staging2 t2
	ON t1.company = t2.company
SET t1.industry = t2.industry
WHERE (t1.industry IS NULL OR t1.industry = '')
AND t2.industry IS NOT NULL;	 

SELECT *
FROM layoffs_staging2;

SELECT *
FROM layoffs_staging2
WHERE total_laid_off IS NULL
AND percentage_laid_off IS NULL;

DELETE
FROM layoffs_staging2
WHERE total_laid_off IS NULL
AND percentage_laid_off IS NULL;

ALTER TABLE layoffs_staging2
DROP COLUMN row_num;

SELECT *
FROM layoffs_staging2;
```



