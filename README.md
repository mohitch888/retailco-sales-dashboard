# RetailCo Sales Performance Dashboard

An interactive Power BI dashboard analysing three years of retail transactions, built on a deliberately messy dataset. The project covers data cleaning, a data-quality audit, DAX measures and a one-page report.

> **Note:** "RetailCo" is a fictional company name used for presentation. The data is a sample retail dataset.

![Dashboard](images/dashboard.png)

## Business questions

- How much revenue are we generating, and how is it trending year over year?
- Which sales channels, categories, items and payment methods drive revenue?
- Which days of the week perform best?
- Do discounts change how much customers spend?
- How trustworthy is the underlying data?

## Key insights

| Insight | Detail |
|---|---|
| Revenue is steady and growing | $1.55M total from 11,971 clean transactions. 2024 revenue ($524.9K) was **+6.8%** vs 2023 ($491.3K). |
| Channels are balanced | Online 51% / In-store 49% of revenue. |
| Categories are evenly spread | Butchers leads at $208.1K; Milk Products is lowest at $180.1K, a gap of about 15%. |
| Friday is the strongest day | $232.8K, versus $213.1K on Monday (the weakest). |
| Discounts show no measurable effect | Average basket is $130.49 (discounted), $129.95 (not discounted) and $128.51 (unknown). `Total Spent` always equals price x quantity, so the discount flag is not reflected in the amount. |
| Payment mix is even | Cash 34.6%, Credit Card 32.7%, Digital Wallet 32.7% of revenue. |

## Data quality

The raw file has 12,575 rows. 11,971 (95.2%) were kept after cleaning.

| Issue | Raw rows | Action |
|---|---|---|
| Missing Total Spent | 604 | Excluded (unrecoverable, Quantity also missing) |
| Missing Price Per Unit | 609 | Recovered as Total Spent / Quantity |
| Missing Item | 1,213 | 609 recovered via Category + Price lookup; 604 were in excluded rows |
| Missing Discount Flag | 4,199 | 3,988 labelled "Unknown" in clean data; 211 in excluded rows |

Full details: [docs/data_cleaning.md](docs/data_cleaning.md)

## Repository structure

```
├── dashboard/   Power BI file (.pbix) and PDF export
├── data/        Raw CSV and data dictionary
├── docs/        Data cleaning steps and DAX measures
└── images/      Dashboard screenshot
```

## Tools

Power BI Desktop, Power Query (M), DAX

## How to open

1. Install [Power BI Desktop](https://powerbi.microsoft.com/desktop/) (free, Windows).
2. Download `dashboard/RetailCo_Sales_Dashboard.pbix`.
3. Open it. If prompted, point the data source to `data/retail_store_sales.csv`.

## Limitations

- 2025 contains only 18 days (to 2025-01-18), so year comparisons involving 2025 are not meaningful.
- 604 rows (4.8%) were excluded and could not be estimated.
- 33% of clean rows have an unknown discount flag, so discount analysis is indicative only.

## Data source

Retail store sales sample dataset (12,575 rows, 2022-01-01 to 2025-01-18).
<!--https://www.kaggle.com/datasets/ahmedmohamed2003/retail-store-sales-dirty-for-data-cleaning?utm_source=chatgpt.com  -->
