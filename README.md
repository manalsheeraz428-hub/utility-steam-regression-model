# Retail Chain Data Analysis with Pandas

A data analysis project exploring a multi-table retail dataset (stores, employees, and sales transactions) using Python and pandas. The project demonstrates core data wrangling skills: aggregation, filtering, merging, and derived-metric calculation across relational tables.

## Overview

This notebook works with three related datasets from a retail chain and answers four business questions:

1. **Store footprint by city** — How many stores does each city have, and what's the total square footage?
2. **Headcount by city** — How many employees work in each city?
3. **Sales by city** — Which city did each sales transaction occur in?
4. **Revenue by city** — What's the total revenue generated per city?

The goal is to take normalized, transaction-level data and roll it up into city-level business insights.

## Datasets

| File | Description |
|---|---|
| `tStores.csv` | One row per store: `Store` ID, `City`, `County`, `SqFt` |
| `tEmployees.csv` | One row per employee: `emplID`, `Store` (which store they work at) |
| `tSales.csv` | One row per transaction: `salesID`, `Store`, `Brand`, `SalesQuantity`, `SalesPrice`, `SalesDate`, `ExciseTax` |

## Techniques Used

- **`groupby().agg()`** — grouping records by `City` and computing multiple named aggregations (`sum`, `count`) in a single step
- **Boolean filtering** — `.loc[]` to filter grouped results (e.g., cities with more than one store)
- **`pd.merge()`** — left joins to bring `City` information into the employee and sales tables via the shared `Store` key
- **Derived columns** — calculating `Revenue` as `SalesPrice * SalesQuantity` per transaction before aggregating
- **Multi-table rollups** — combining store, employee, and sales summaries into a single city-level view

## Key Steps

```python
# 1. Stores and square footage per city (filtered to cities with >1 store)
cityStores = tStores.groupby('City').agg(
    sumSqFt=('SqFt', 'sum'),
    numStores=('Store', 'count')
)
cityStores.loc[cityStores['numStores'] > 1]

# 2. Employees per city (merge employees -> stores to get City)
cityEmployees = pd.merge(tEmployees, tStores, on='Store', how='left')
cityEmployees.groupby('City').agg(totalEmployees=('emplID', 'count'))

# 3. Sales transactions tagged with City (merge sales -> stores)
Store_CitySales = pd.merge(tSales, tStores, on='Store', how='left')

# 4. Revenue per transaction, then summed by city
Store_CitySales['Revenue'] = Store_CitySales['SalesPrice'] * Store_CitySales['SalesQuantity']
tSales_Summary = Store_CitySales.groupby('City').agg(totalRevenue=('Revenue', 'sum'))
```

## How to Run

1. Clone or download this repo, keeping `tEmployees.csv`, `tStores.csv`, and `tSales.csv` in the same directory as the notebook.
2. Install dependencies:
   ```bash
   pip install pandas numpy
   ```
3. Open the notebook in Jupyter or Google Colab and run all cells in order.

## Notes for Future Improvement

- The final combined `final` DataFrame merges city-level summaries (`cityStores`, `cityEmployees`) with **row-level** transaction data (`Store_CitySales`) on `City`. Because the join key isn't unique on the sales side, this produces a many-to-many merge that duplicates the store/employee summary values across every transaction row. A cleaner final rollup would merge `cityStores`, `cityEmployees`, and `tSales_Summary` (the *aggregated* revenue table) together — all at the same one-row-per-city grain — before displaying the combined result.
- Adding a data dictionary or ER diagram for the three source tables would make the relationships easier to follow for anyone reviewing the project.
- Could extend the analysis with per-city metrics like revenue per employee or revenue per square foot.

## Skills Demonstrated

`pandas` · data aggregation · relational merges · derived metrics · exploratory data analysis
