# Banking Transactions Analytics Dashboard (Power BI + SQL Server)

## Project Overview

This project demonstrates an end-to-end Business Intelligence workflow using SQL Server and Power BI.

The objective was to create a banking transaction analytics solution by generating, cleaning, transforming, and visualizing transaction data.

The project includes:

- Synthetic banking dataset generation (10,000+ records)
- SQL Server database creation
- Data cleaning and transformation
- Power Query preprocessing
- Interactive Power BI dashboard creation

---

## Tech Stack

- SQL Server Management Studio (SSMS)
- SQL
- Power BI
- Power Query
- DAX

---

## Dataset

The dataset contains:

### Customers Table
- Customer ID
- Name
- Gender
- Date of Birth
- Contact information

### Accounts Table
- Account Type
- Opening Date
- Balance

### Transactions Table
10,000+ synthetic records including:

- Credits / Debits
- Transaction dates
- Account relationships
- Currency information
- Intentional data quality issues

Examples of introduced issues:

- Duplicate records
- Null values
- Mixed date formats
- Invalid references
- Case inconsistencies
- Outlier transactions

---

## Data Cleaning Process

### SQL Layer

Performed:

✔ Database creation

✔ Table creation

✔ Synthetic transaction generation

✔ Data issue simulation

Examples:

- Invalid Account IDs
- Outlier balances
- Missing descriptions
- Mixed date formats

---

### Power Query Transformations

Inside Power BI:

- Removed duplicates
- Replaced null values
- Standardized date formats
- Corrected datatypes
- Built cleaned combined table

---

## Dashboard Features

Dashboard KPIs include:

- Total Transactions
- Credits vs Debits
- Account Distribution
- Transaction Trends
- Customer Insights
- Balance Analysis

---

## Dashboard Preview

### Main Dashboard

![Dashboard](screenshots/dashboard-overview.png)

### Power Query Cleaning

![Cleaning](screenshots/power-query-cleaning.png)

### Data Model

![Model](screenshots/model-view.png)

---

## SQL Example

```sql
CREATE DATABASE Power_BI2;

CREATE TABLE Customers (
    CustomerID INT PRIMARY KEY,
    Name NVARCHAR(100),
    Gender VARCHAR(10)
);
```

---

## Project Workflow

Dataset Generation

↓

SQL Database Creation

↓

Data Cleaning

↓

Power Query Transformation

↓

Power BI Dashboard

↓

Reporting & Publishing

---

## Future Improvements

- Add incremental refresh
- Add DAX measures
- Add forecasting visuals
- Deploy via Power BI Service
