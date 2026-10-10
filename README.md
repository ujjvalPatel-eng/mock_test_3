# Delivery Delay Analysis

## Project Overview

This project analyzes delivery performance and delay patterns using data prepared in Excel and then imported into CSV files for analysis.

The original Excel workbook contains multiple sheets. For this project, the two required sheets are:

- **Clean** — delivery-level data
- **Lookup** — route and service-type lookup data

These two sheets are exported to CSV files and used as the main input files for the Python analysis and SQL database work.

### Excel-to-CSV Mapping

| Excel Sheet | CSV File | Purpose |
|---|---|---|
| `Clean` | `Delivery.csv` | Delivery records, including promised and actual delivery days |
| `Lookup` | `Routes.csv` | Route IDs, route names, and service types |

The Python notebook loads these files with:

```python
deliveries = pd.read_csv('../data/Delivery.csv')
routes = pd.read_csv('../data/Routes.csv')
```

---

## Project Structure

A typical project structure is:

```text
Mock_3/
│
├── data/
│   ├── Delivery.csv
│   └── Routes.csv
│
├── Output/
│   └── python_chart.png
│
├── Power BI/
│   ├── dashboard.png
│   └── dashboard.pbix
│
├── python/
│   └── analysis.ipynb
│
├── SQL/
│   ├── setup.sql
│   └── queries.sql
│
└── README.md
```

> Keep the main folder named `Mock_3`. The notebook expects the CSV files inside `data/` and saves the Python chart inside `Output/`. The `Power BI/` folder contains the Power BI report file and its screenshot.

---

## Data Description

### `Delivery.csv`

This file is created from the Excel **Clean** sheet.

Important fields include:

- `record_id` — unique delivery record identifier
- `month` — delivery month
- `route_id` — identifier used to connect delivery records with route information
- `hub` — delivery hub
- `promised_days` — number of days promised for delivery
- `actual_days` — actual number of days taken
- `service_type` — service category associated with the route
- `Delay_days` — delay calculated from actual versus promised delivery time in the cleaned Excel data

### `Routes.csv`

This file is created from the Excel **Lookup** sheet.

Fields include:

- `route_id` — unique route identifier
- `route` — route name
- `service_type` — service category for the route

---

## Data Preparation

The Excel workbook is used as the initial source of the data.

The workflow is:

1. Open the Excel workbook.
2. Use the **Clean** sheet as the delivery dataset.
3. Use the **Lookup** sheet as the route lookup dataset.
4. Export **Clean** to `Delivery.csv`.
5. Export **Lookup** to `Routes.csv`.
6. Place both CSV files in the project's `data` folder.
7. Run the Python notebook and/or SQL scripts using the CSV data.

The cleaned dataset contains 12 delivery records after removing duplicate rows.

---

## Python Analysis

The main analysis is performed in `python/analysis.ipynb`.

### 1. Load the CSV files

Because the notebook is inside the `python` folder, it reads the CSV files from the parent project's `data` folder:

```python
import pandas as pd

deliveries = pd.read_csv('../data/Delivery.csv')
routes = pd.read_csv('../data/Routes.csv')
```

### 2. Validate numeric fields

The notebook checks that `promised_days` and `actual_days` are numeric.

```python
deliveries['promised_days'] = pd.to_numeric(
    deliveries['promised_days'],
    errors='raise'
)

deliveries['actual_days'] = pd.to_numeric(
    deliveries['actual_days'],
    errors='raise'
)
```

### 3. Remove duplicate records

Exact duplicate rows are removed before analysis.

```python
deliveries = deliveries.drop_duplicates()
```

### 4. Join delivery and route data

The delivery data is merged with the route lookup using `route_id`.

```python
df = deliveries.merge(
    routes,
    on='route_id',
    how='left'
)
```

The notebook verifies that:

- the final dataset contains exactly 12 rows
- every `route_id` has a matching route/service type

### 5. Calculate delay

Delay is calculated as actual delivery time minus promised delivery time. Negative values are converted to zero.

```python
df['delay_days'] = (
    df['actual_days'] - df['promised_days']
).clip(lower=0)
```

This means deliveries completed on or before the promised date have `0` delay days.

### 6. Service-type analysis

The notebook calculates:

- total delay days by service type
- delay incidence rate by service type

### 7. Route analysis

The notebook calculates the total delay days for each route and identifies the route with the greatest summed delay.

It also calculates that route's share of the overall delay.

### 8. Monthly analysis

The notebook groups the data by month and calculates total delay days.

A bar chart is created showing:

**Monthly Total Delay Days**

The chart is saved as:

```text
Output/python_chart.png
```

---

## SQL Database

The SQL setup creates a database named `Delibery_db` and two tables:

- `Delivery`
- `Routes`

The `Delivery` table stores delivery-level information such as route, promised days, actual days, service type, and delay days.

The `Routes` table stores route and service-type lookup information.

The delivery and route tables are connected using `route_id`.

---

## SQL Analysis

The SQL queries analyze delay at different levels.

### Delay by Service Type

The project calculates total delay days for each service type by joining `Delivery` with `Routes` on `route_id`.

### Routes With More Than 8 Delay Days

The project identifies routes where the summed delay is greater than 8 days.

### Hub-Level Delay

The project also calculates total delay days by hub and returns the two hubs with the highest total delay.

---

## Key Data Quality Checks

The Python workflow includes the following validation checks:

- `promised_days` must be numeric.
- `actual_days` must be numeric.
- Exact duplicate delivery rows are removed.
- The final delivery dataset should contain 12 rows.
- Every delivery `route_id` should have a matching record in the route lookup.
- Delay days cannot be negative.

These checks help ensure that the data is suitable for the downstream analysis.

---

## Power BI Dashboard

The Power BI report is stored in the `Power BI` folder:

```text
Mock_3/
└── Power BI/
    ├── dashboard.png
    └── dashboard.pbix
```

- **`dashboard.pbix`** — the editable Power BI report.
- **`dashboard.png`** — a screenshot of the dashboard for quick preview and submission evidence.

### Data Sources

Use the delivery and route CSV files in the project's `data` folder as the report's data sources:

- `data/Delivery.csv` — cleaned delivery records.
- `data/Routes.csv` — route and service-type lookup data.

If Power BI reports a missing file after the project is moved to another computer, open `Power BI/dashboard.pbix` in Power BI Desktop and update the CSV locations through **Home → Transform data → Data source settings → Change Source**. Point each source to the matching CSV file inside the local `Mock_3/data/` folder, select **Close & Apply**, and refresh the report.

### Recommended Report Contents

Use the dashboard to analyze delivery delays by service type, route, hub, and month. Include visuals that make the following easy to identify:

- Total delay days by service type.
- Total delay days by hub.
- Monthly total delay days, ordered Jan → Feb → Mar.
- Delivery count and delay incidence rate, where available.
- A hub slicer or filter to explore hub-level performance.

Delay days should be calculated as `MAX(actual_days - promised_days, 0)`. Confirm that the Power BI data has 12 unique delivery records after duplicate removal and that the route lookup matches through `route_id`.

### Dashboard Screenshot

Save or replace the dashboard screenshot at:

```text
Power BI/dashboard.png
```

Make sure the screenshot shows the report clearly and reflects the same cleaned data used in the analysis. The `.pbix` file should remain in the folder so the report is editable; the PNG is only a visual preview.

---

## Tools Used

- **Microsoft Excel** — initial data source and data preparation
- **CSV** — data exchange format between Excel and the analysis workflow
- **Python**
  - Pandas — data cleaning and analysis
  - Matplotlib — visualization
  - Jupyter Notebook — analysis workflow
- **SQL** — database setup and analytical queries

---

## How to Run the Project

### Step 1 — Prepare the CSV files

Export the following Excel sheets:

```text
Clean   → data/Delivery.csv
Lookup  → data/Routes.csv
```

### Step 2 — Run the Python notebook

Open:

```text
python/analysis.ipynb
```

Run the notebook from top to bottom.

### Step 3 — Generate the Python visualization

The notebook generates:

```text
Output/python_chart.png
```

### Step 4 — Open the Power BI report

Open `Power BI/dashboard.pbix` in Power BI Desktop. If necessary, update the data-source paths to the CSV files inside `Mock_3/data/`, then refresh the report. The dashboard screenshot is stored at `Power BI/dashboard.png`.

### Step 5 — Run the SQL setup

Execute:

```text
SQL/setup.sql
```

This creates the database and required tables.

### Step 6 — Run the SQL analysis

Execute:

```text
SQL/queries.sql
```

to perform the service-type, route, and hub-level delay analysis.

---

## Analysis Workflow

```text
Excel Workbook
     │
     ├── Clean Sheet
     │       │
     │       └── Delivery.csv
     │
     └── Lookup Sheet
             │
             └── Routes.csv
                     │
                     ▼
              Python / SQL Analysis
                     │
          ┌──────────┼──────────┐
          ▼          ▼          ▼
       Cleaning   Delay       SQL Queries
                  Analysis
          │
          ▼
     Monthly Chart
```

---

## Notes

- The Excel workbook is the source used to prepare the CSV input files.
- `Delivery.csv` and `Routes.csv` should be kept in the `data` directory expected by the notebook.
- The notebook recalculates `delay_days` from `actual_days` and `promised_days`, so the analysis does not rely only on the pre-existing delay column in the Excel sheet.
- SQL and Python provide two complementary approaches for analyzing the prepared delivery data.
