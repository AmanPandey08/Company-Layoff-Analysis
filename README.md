# Company-Layoff-Analysis

-- Data Cleaning
-- 1. Remove Duplicates
-- 2. Standardize the data
-- 3. Remove Null or Blank Values
-- 4. Remove irrelevant column or rows(if any)

SELECT *
FROM layoffs;

CREATE TABLE layoff_stanging
LIKE layoffs;

SELECT * 
FROM layoff_stanging;

-- Copying raw data into new table
INSERT layoff_stanging
SELECT * 
FROM layoffs;

-- Creating CTE to know about duplicates values
WITH cte_duplicates AS
(
	SELECT * ,
    ROW_NUMBER() OVER(
    PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, `date`, 
    stage, country, funds_raised_millions) AS row_numbers
    FROM layoff_stanging
)
SELECT * 
FROM cte_duplicates
WHERE row_numbers > 1;

-- Creating another table to copy the CTE table
CREATE TABLE `layoffs_staging2` (
`company` text,
`location`text,
`industry`text,
`total_laid_off` INT,
`percentage_laid_off` text,
`date` text,
`stage`text,
`country` text,
`funds_raised_millions` int,
row_num INT
);

INSERT INTO layoffs_staging2
SELECT * ,
    ROW_NUMBER() OVER(
    PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, `date`, 
    stage, country, funds_raised_millions) AS row_numbers
    FROM layoff_stanging;
 
 -- Checking duplicate rows
SELECT * 
FROM layoffs_staging2
WHERE row_num > 1;

SET SQL_SAFE_UPDATES = 0;

 -- Removing duplicate rows
DELETE
FROM layoffs_staging2
WHERE row_num > 1;


SELECT * 
FROM layoffs_staging2;

-- Standardlizing the Data

-- Checking for extra spaces before and after the company names
SELECT company 
FROM layoffs_staging2;

-- Remvoing using xtra spaces TRIM() Function
UPDATE layoffs_staging2
SET company = TRIM(company);

-- We have identfied Cryto, Crypto Currency and CryptoCurrency are all same
SELECT DISTINCT(industry)
FROM layoffs_staging2
ORDER BY 1;

-- Updating all the same company name with different  style as one
UPDATE layoffs_staging2
SET industry = 'Crypto'
WHERE industry LIKE 'Crypto%';

SELECT DISTINCT(country)
FROM layoffs_staging2
ORDER BY 1;

-- Removing . from the end
UPDATE layoffs_staging2
SET country = TRIM(TRAILING '.' FROM country)
WHERE country LIKE 'United States%';


-- Date column is a text data type
SELECT `date` 
FROM layoffs_staging2;

-- Converting text to Date data type
 UPDATE layoffs_staging2
 SET `date` = STR_TO_DATE(`date`, '%m/%d/%Y');
 
 ALTER TABLE layoffs_staging2
 MODIFY `date` DATE;
 
 -- 3. Remove Null or Blank Values
 
 -- There are 4 industries with null or blank values
 SELECT *
 FROM layoffs_staging2
 WHERE industry IS NULL
 OR industry = '';

-- Indentify the type of industry and try to fill the null or blank spaces
 SELECT t1.company, t1.industry, t2.company, t2.industry
 FROM layoffs_staging2 t1
 JOIN layoffs_staging2 t2
	ON t1.company = t2.company
WHERE(t1.industry IS NULL OR t1.industry = '')
AND t2.industry IS NOT NULL
ORDER BY 1;

UPDATE layoffs_staging2
SET industry= NULL
WHERE industry = '';

UPDATE layoffs_staging2 t1
JOIN layoffs_staging2 t2
	SET t1.industry = t2.industry
WHERE(t1.industry IS NULL OR t1.industry = '')
AND t2.industry IS NOT NULL;

SELECT *
FROM layoffs_staging2
WHERE industry IS NULL 
OR industry = '';

-- 4. Remove irrelevant column or rows(if any)
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


