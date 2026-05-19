# 🏦 Banking Transactions - Data Cleaning & Analytics Dashboard

<div align="center">

![SQL Server](https://img.shields.io/badge/SQL_Server-CC2927?style=for-the-badge&logo=microsoft-sql-server&logoColor=white)
![T-SQL](https://img.shields.io/badge/T--SQL-4479A1?style=for-the-badge&logo=databricks&logoColor=white)
![Power BI](https://img.shields.io/badge/Power_BI-F2C811?style=for-the-badge&logo=powerbi&logoColor=black)
![Power Query](https://img.shields.io/badge/Power_Query-217346?style=for-the-badge&logo=microsoft-excel&logoColor=white)

*End-to-end pipeline from synthetic dirty data generation → SQL cleaning → Power BI reporting.*

</div>

---

## Project Overview

This project simulates a real-world banking data pipeline using **10,000 AI-generated synthetic transaction records** intentionally loaded with data quality issues. The goal: clean, transform, and join the data in SQL Server, then build an analytics dashboard in Power BI.

<div align="center">

| Phase | Tool | Description |
|---|---|---|
| Data Generation | Perplexity AI | Generated 10,000 synthetic banking transactions with intentional data issues |
| Database Setup | SQL Server Management Studio | Created `Power_BI2` database with three relational tables |
| Data Cleaning | T-SQL | Standardized dates, fixed casing, removed nulls and outliers |
| Data Integration | T-SQL (`LEFT JOIN`) | Joined Transactions, Accounts, and Customers into a combined reporting table |
| Final Prep | Power Query Editor | Resolved remaining type, null, and duplicate issues |
| Reporting | Power BI | Built and published the analytics dashboard |

</div>

---

## Database Schema

The `Power_BI2` database contains three tables, each intentionally seeded with data quality issues to simulate realistic dirty data scenarios.

### `Customers`

<div align="center">

| Column | Type | Notes |
|---|---|---|
| `CustomerID` | INT (PK) | Unique customer identifier |
| `Name` | NVARCHAR(100) | Inconsistent casing in raw data |
| `Gender` | VARCHAR(10) | Nullable; some records missing |
| `DateOfBirth` | VARCHAR(20) | Mixed date formats (`yyyy-MM-dd`, `dd-MM-yyyy`, `yyyy/MM/dd`) |
| `Address` | NVARCHAR(200) | Nullable; some empty strings |
| `Email` | NVARCHAR(100) | Nullable |
| `Phone` | VARCHAR(20) | — |
| `AccountID` | INT | Soft reference; not a true FK |

</div>

### `Accounts`

<div align="center">

| Column | Type | Notes |
|---|---|---|
| `AccountID` | INT (PK) | Unique account identifier |
| `CustomerID` | INT | Soft reference; includes orphaned records |
| `Type` | NVARCHAR(20) | Inconsistent casing (`SAVINGS`, `Savings`, `current`) |
| `OpenDate` | VARCHAR(20) | Mixed date formats |
| `Balance` | DECIMAL(18,2) | Includes negative and zero outlier balances |

</div>

### `Transactions`

<div align="center">

| Column | Type | Notes |
|---|---|---|
| `TransactionID` | INT | No PK enforced — duplicates allowed for demo |
| `AccountID` | INT | Some mismatched/orphaned IDs (`9999`) |
| `TransactionDate` | VARCHAR(20) | Mixed date formats |
| `Type` | VARCHAR(20) | Inconsistent casing (`Credit`, `DEBIT`) |
| `Amount` | DECIMAL(18,2) | Includes negative outliers (`-99,999.99`) and extreme positives (`1,000,000.99`) |
| `Description` | NVARCHAR(200) | ~2% NULL; inconsistent casing |
| `Currency` | VARCHAR(10) | Mixed casing (`USD`, `usd`, `INR`) |

</div>

---

## Data Cleaning Workflow

### Issue 1 — Mixed Date Formats

All three tables stored dates as `VARCHAR` with multiple inconsistent formats. A `TRY_CONVERT` cascade was used to detect and standardize every date to `MM/dd/yyyy`:

```sql
UPDATE Transactions
SET TransactionDate = 
    CASE
        WHEN TRY_CONVERT(date, TransactionDate, 101) IS NOT NULL  -- MM/DD/YYYY
            THEN FORMAT(TRY_CONVERT(date, TransactionDate, 101), 'MM/dd/yyyy')
        WHEN TRY_CONVERT(date, TransactionDate, 23)  IS NOT NULL  -- YYYY-MM-DD
            THEN FORMAT(TRY_CONVERT(date, TransactionDate, 23),  'MM/dd/yyyy')
        WHEN TRY_CONVERT(date, TransactionDate, 111) IS NOT NULL  -- YYYY/MM/DD
            THEN FORMAT(TRY_CONVERT(date, TransactionDate, 111), 'MM/dd/yyyy')
        WHEN TRY_CONVERT(date, TransactionDate, 105) IS NOT NULL  -- DD-MM-YYYY
            THEN FORMAT(TRY_CONVERT(date, TransactionDate, 105), 'MM/dd/yyyy')
        WHEN TRY_CONVERT(date, TransactionDate, 103) IS NOT NULL  -- DD/MM/YYYY
            THEN FORMAT(TRY_CONVERT(date, TransactionDate, 103), 'MM/dd/yyyy')
        ELSE TransactionDate  -- leave unparseable values in place
    END;
```

Applied to: `Transactions.TransactionDate`, `Accounts.OpenDate`, `Customers.DateOfBirth`

---

### Issue 2 — Synthetic Data Quality Problems (by design)

The 10,000-row transaction dataset was generated with the following intentional flaws:

<div align="center">

| Issue | Example | Frequency |
|---|---|---|
| Orphaned AccountIDs | `AccountID = 9999` | Every 20th row |
| Mixed date formats | `yyyy/MM/dd` vs `dd-MM-yyyy` | Every 3rd row alternates |
| Inconsistent Type casing | `'Credit'` vs `'DEBIT'` | Alternating rows |
| Negative amount outliers | `-99,999.99` | Every 1,000th row |
| Positive amount outliers | `1,000,000.99` | Every 250th row |
| NULL descriptions | `NULL` | Every 50th row |
| Mixed currency casing | `'usd'` vs `'USD'` vs `'INR'` | Every 3rd / 5th row |

</div>

---

## Data Integration — Combined Reporting Table

After cleaning, all three tables were joined via `LEFT JOIN` and exported into a single flat reporting table (`CombinedBankingDataset`) for Power BI:

```sql
SELECT
    t.TransactionID,
    t.AccountID        AS Transaction_AccountID,
    t.TransactionDate,
    t.Type             AS TransactionType,
    t.Amount,
    t.Description,
    t.Currency,
    a.AccountID        AS Account_AccountID,
    a.CustomerID       AS Account_CustomerID,
    a.Type             AS AccountType,
    a.OpenDate,
    a.Balance,
    c.CustomerID,
    c.Name,
    c.Gender,
    c.DateOfBirth,
    c.Address,
    c.Email,
    c.Phone
INTO CombinedBankingDataset
FROM      Transactions t
LEFT JOIN Accounts     a ON t.AccountID  = a.AccountID
LEFT JOIN Customers    c ON a.CustomerID = c.CustomerID;
```

> `LEFT JOIN` was used throughout to preserve all transaction records, including those with orphaned or mismatched account/customer references.

---

## Power Query Transformations

After importing `CombinedBankingDataset` into Power BI, the following final transformations were applied in the Power Query Editor:

<div align="center">

| Task | Action |
|---|---|
| Null values | Replaced with appropriate defaults or flagged |
| Duplicate records | Identified and removed |
| Data types | Assigned correct types to all columns (date, decimal, text) |
| Report preparation | Final column renaming and formatting for dashboard readiness |

</div>

---

## Dashboard

The published Power BI report (`Project 5 Dashboard.pbix`) visualizes the cleaned and joined banking dataset, providing insights across transactions, account activity, and customer segments.

> See `Project 5 Dashboard.pbix` for the full interactive report.

---

## Project Files

<div align="center">

| File | Description |
|---|---|
| `SQLQuery.sql` | Full SQL script — schema creation, data seeding, cleaning, and JOIN |
| `SQL_Query.txt` | Alternate version of the SQL script with table drop guards |
| `Project 5 Dashboard.pbix` | Power BI report file |
| `Project 5. Walkthrough.txt` | High-level project summary |

</div>

---

## Technologies

<div align="center">

| Tool | Purpose |
|---|---|
| **SQL Server Management Studio (SSMS)** | Database creation, data loading, and T-SQL execution |
| **T-SQL** | Schema design, synthetic data generation, cleaning, and JOIN logic |
| **Power BI Desktop** | Dashboard creation and publishing |
| **Power Query Editor** | Final in-report data transformation and type assignment |
| **Perplexity AI** | Synthetic dataset generation (10,000 transaction rows) |

</div>

---

<div align="center">
<sub>Built to demonstrate end-to-end data engineering - from intentionally dirty synthetic data to a clean, publishable analytics report.</sub>
</div>
