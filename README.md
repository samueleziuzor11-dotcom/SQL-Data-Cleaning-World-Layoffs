# 🧹 SQL Data Cleaning — World Layoffs Dataset

## 📌 Project Overview
Raw data is rarely clean. This project demonstrates a **professional SQL data cleaning workflow** applied to a real-world global layoffs dataset sourced from Kaggle.

The goal was to take messy, inconsistent raw data and transform it into a clean, reliable dataset ready for Exploratory Data Analysis (EDA) and reporting.

🔗 Dataset Source: [Kaggle — Layoffs 2022](https://www.kaggle.com/datasets/swaptr/layoffs-2022)

---

## 🛠️ Tools Used

| Tool | Purpose |
|---|---|
| **SQL (MySQL)** | Full data cleaning pipeline |
| **Staging Tables** | Protected raw data while cleaning a working copy |
| **Window Functions** | Used ROW_NUMBER() with PARTITION BY to detect duplicates |
| **CTEs** | Used WITH clause for clean, readable duplicate removal logic |

---

## 🔄 Data Cleaning Steps

### Step 1 — Remove Duplicates
- Created a **staging table** to preserve the original raw data
- Used `ROW_NUMBER()` with `PARTITION BY` across all columns to identify true duplicates
- Verified duplicates manually before deleting (e.g. checked Oda company entries)
- Added a `row_num` column to a second staging table to safely delete rows where `row_num >= 2`

```sql
SELECT company, location, industry, total_laid_off, percentage_laid_off, date,
       ROW_NUMBER() OVER (
           PARTITION BY company, location, industry, total_laid_off,
           percentage_laid_off, date, stage, country, funds_raised_millions
       ) AS row_num
FROM world_layoffs.layoffs_staging;
```

---

### Step 2 — Standardize Data
- Converted blank industry values to `NULL` for easier handling
- Used a **self-join** to populate NULL industry values from matching company rows
- Standardized inconsistent industry labels (e.g. `Crypto Currency`, `CryptoCurrency` → all unified to `Crypto`)
- Removed trailing periods from country names (e.g. `United States.` → `United States`)
- Fixed date format using `STR_TO_DATE()` and converted column type to `DATE`

```sql
-- Unify Crypto variations
UPDATE layoffs_staging2
SET industry = 'Crypto'
WHERE industry IN ('Crypto Currency', 'CryptoCurrency');

-- Fix trailing period in country names
UPDATE layoffs_staging2
SET country = TRIM(TRAILING '.' FROM country);

-- Fix date format
UPDATE layoffs_staging2
SET `date` = STR_TO_DATE(`date`, '%m/%d/%Y');

ALTER TABLE layoffs_staging2
MODIFY COLUMN `date` DATE;
```

---

### Step 3 — Handle NULL Values
- Reviewed NULL values in `total_laid_off`, `percentage_laid_off`, and `funds_raised_millions`
- Made a deliberate decision to **keep NULLs** where data was genuinely unknown — preserving integrity for future EDA calculations rather than filling with assumptions

---

### Step 4 — Remove Unnecessary Rows & Columns
- Deleted rows where both `total_laid_off` AND `percentage_laid_off` were NULL (no analytical value)
- Dropped the helper `row_num` column after it was no longer needed

```sql
DELETE FROM world_layoffs.layoffs_staging2
WHERE total_laid_off IS NULL
AND percentage_laid_off IS NULL;

ALTER TABLE layoffs_staging2
DROP COLUMN row_num;
```

---

## 💡 Key Skills Demonstrated

- Creating and using **staging tables** to protect raw data
- Detecting and removing duplicates using **ROW_NUMBER() + PARTITION BY**
- Writing **CTEs (Common Table Expressions)** for clean query logic
- **Self-joins** to fill missing values intelligently
- **STR_TO_DATE()** and **ALTER TABLE** for type conversion
- Making **judgment calls** on NULLs vs. dropping data

---

## 📁 Files in This Repository

| File | Description |
|---|---|
| `Portfolio Project - Data Cleaning.sql` | Full SQL data cleaning script with comments |

---

## 👤 Author
**samueleziuzor11-dotcom**
Aspiring Data Analyst | SQL • Excel • Power BI • Power Pivot | Open to first role
