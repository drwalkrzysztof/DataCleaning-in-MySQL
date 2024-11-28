# DataCleaning-in-MySQL
I am building a data cleaning project in MySQL

## Loading raw csv Data using Table Date Import Wizard

```
SELECT *
FROM layoffs;
```
## Stages 
  1. Removing Duplicates 
  2. Standardizing the Data 
  3. Null values or Blank values 
  4. Removing Any Columns or Rows

## Ad.1 I created a Layoffs_staging table to work with

```
CREATE TABLE layoffs_staging
LIKE layoffs;

SELECT *
FROM layoffs_staging;

INSERT layoffs_staging
SELECT *
FROM layoffs;
```
## I get rid of the duplicates by partitioning through each column. I create an additional column that performs the duplicate rows

```
SELECT *,
	ROW_NUMBER() OVER (PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, 
    `date`, stage, country, funds_raised_millions) AS row_num
FROM layoffs_staging;
```

## I then created a sub-query WITH

```
WITH duplicate_cte AS 
(
SELECT *,
	ROW_NUMBER() OVER (PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, 
    `date`, stage, country, funds_raised_millions) AS row_num
FROM layoffs_staging
)
SELECT *
FROM duplicate_cte
WHERE row_num >1;
```
## As I am using mySQL deleting with sub_query duplicate_cte is not possible, so I create an additional table layoffs_staging2 and use PARTITION BY 

```
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

SELECT *
FROM layoffs_staging2;
```

## In the table created, I have a new row_num column that will allow me to remove duplicates from the layoffs_staging2 table

```
INSERT INTO layoffs_staging2
SELECT *,
	ROW_NUMBER() OVER (PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, 
    `date`, stage, country, funds_raised_millions) AS row_num
FROM layoffs_staging;

SELECT *
FROM layoffs_staging2
WHERE row_num>1;

DELETE 
FROM layoffs_staging2
WHERE row_num >1;
```

## Ad.2 Standardizing Data  - I am removing white spaces from company columns and rows 

```
SELECT company, TRIM(company)
FROM layoffs_staging2;

UPDATE layoffs_staging2
SET company = TRIM(company);
```

## I am checking DISTINCT industries and playing with NULL, blank and non-standardised categories such as Crypto, Crypto Currency and CryptoCurrency 

```
SELECT DISTINCT industry
FROM layoffs_staging2
ORDER BY 1;

SELECT *
FROM layoffs_staging2
WHERE industry LIKE 'Crypto%';

UPDATE layoffs_staging2
SET industry = 'Crypto'
WHERE industry LIKE 'Crypto%';
```
## I am checking further the location and the country columns 
```
SELECT DISTINCT location 
FROM layoffs_staging2
ORDER BY 1;
```
## Using TRAILING I am getting rid of dot from United States. 

```
SELECT DISTINCT country, TRIM(TRAILING '.' FROM country)
FROM layoffs_staging2
ORDER BY 1;

UPDATE layoffs_staging2
SET country = TRIM(TRAILING '.' FROM country)
WHERE country LIKE 'United States%';
```

## I am changing `date` column TEXT format to DATE format 

```
SELECT `date`,
STR_TO_DATE(`date`, '%m/%d/%Y')
FROM layoffs_staging2;

UPDATE layoffs_staging2
SET `date` = STR_TO_DATE(`date`, '%m/%d/%Y');

SELECT *
FROM layoffs_staging2; 
```

## I am geting rid of NULL and blank values 

```
SELECT *
FROM layoffs_staging2 
WHERE total_laid_off IS NULL
AND percentage_laid_off IS NULL;

SELECT *
FROM layoffs_staging2
WHERE industry IS NULL
OR industry = '';

SELECT *
FROM layoffs_staging2
WHERE company = 'Airbnb';

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
WHERE t1.industry IS NULL 
AND t2.industry IS NOT NULL;

SELECT *
FROM layoffs_staging2
WHERE industry IS NULL;
```

## And I am deleting the rows with empty values from columns total_laid_off & percentage_laid_off

```
DELETE 
FROM layoffs_staging2
WHERE total_laid_off IS NULL
AND percentage_laid_off IS NULL; 

ALTER TABLE layoffs_staging2
DROP COLUMN row_num;
```
