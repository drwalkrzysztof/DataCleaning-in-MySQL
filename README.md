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

## I created a Layoffs_staging table to work with

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

##
