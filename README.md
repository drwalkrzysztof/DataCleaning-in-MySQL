# DataCleaning-in-MySQL
I am building a data cleaning project in MySQL

## Loading raw csv Data using Table Date Import Wizard

```SELECT *
FROM layoffs;
```
## Stages 
  1. Removing Duplicates 
  2. Standardizing the Data 
  3. Null values or Blank values 
  4. Removing Any Columns or Rows

## I created a Layoffs_staging table to work with

```CREATE TABLE layoffs_staging
LIKE layoffs;

SELECT *
FROM layoffs_staging;

INSERT layoffs_staging
SELECT *
FROM layoffs;
```
