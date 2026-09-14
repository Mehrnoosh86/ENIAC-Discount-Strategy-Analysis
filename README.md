# ENIAC Discount Strategy Analysis

A data-driven investigation into the impact of discounts on ENIAC's e-commerce performance.

## Project Overview

This repository contains the analysis for ENIAC's second data project. Its objective is to assess whether discounting products supports sustainable business growth or negatively affects revenue and margins.

The project addresses an internal debate between the Marketing Team, which views discounts as a driver of customer acquisition, satisfaction, and retention, and the Board's investors, who are concerned about aggressive discounting and the company's positioning in the quality segment.

## Business Questions

- How many products are being discounted?
- How large are discounts as a percentage of product prices?
- How do seasonality and dates such as Christmas and Black Friday affect sales?
- How should products be classified to simplify reporting and analysis?
- What is the distribution of product prices across categories?
- How could data collection and data quality be improved?

## Data Sources

The analysis uses four CSV files:

| File | Description |
|---|---|
| `orders.csv` | One row per order, including order date, total paid, and order state. |
| `orderlines.csv` | One row per product line, including SKU, quantity, unit price, and processing date. |
| `products.csv` | Product catalog containing SKU, name, description, base price, promotional price, stock status, and product type. |
| `brands.csv` | Mapping between three-character brand codes and brand names. |

## Data Quality and Cleaning

The cleaning process prioritized preserving real sales information over aggressively deleting unusual observations. Suspicious values were cross-checked against related tables, order totals, product names, order states, and business logic before being removed or modified.

### `orders.csv`

**Issues identified:**

- `created_date` was stored as text rather than a datetime field.
- Five `total_paid` values were missing.
- `total_paid` contained extreme values reaching approximately €214,747.
- The `state` column contained five states; only `Completed` represented paid and completed sales.

**Treatment:**

- Converted `created_date` to datetime.
- Removed the five missing `total_paid` records because they belonged to `Pending` orders and represented only 0.035% of that state.
- Retained extreme values for documentation because they occurred only in `Shopping Basket` orders.
- Restricted revenue analysis to `Completed` orders.

### `orderlines.csv`

**Issues identified:**

- `unit_price` and `date` were stored as text.
- Approximately 36,169 `unit_price` values contained formatting errors, such as duplicated decimal points (`1.137.99`).
- `product_id` was constant at `0` and unusable.
- `product_quantity` contained extreme values, reaching 999 units.
- One negative unit price (`-119.0`) was detected.
- 865 rows had a unit price of €0.
- Differences between order-line totals and `total_paid` frequently matched shipping fees such as €6.99, €4.99, and €3.99.

**Treatment:**

- Repaired duplicated-decimal formatting where the value could be reliably reconstructed.
- Validated repairs against `products.price`, product names, and order totals. Reconstructed line values matched `orders.total_paid` with 99.97% agreement.
- Set three unrecoverable prices to `NaN`; none belonged to a completed order.
- Corrected the negative price sign because its absolute value matched the corresponding product price.
- Preserved €0 values and created an `is_free_item` indicator. Forty-nine free-item rows belonged to completed orders.
- Converted `date` to datetime and removed `product_id`.
- Retained extreme quantities after validating them against order totals.
- Documented the likely presence of shipping costs that were not stored separately.

### `products.csv`

**Issues identified:**

- `sku` was not reliably unique: 8,746 complete duplicate rows and one additional SKU duplicate were found.
- Missing values occurred in `desc` (7), `price` (46), and `type` (50).
- `price`, `promo_price`, and `type` were stored as text.
- 542 `price` values contained formatting problems.
- More than 92% of `promo_price` values contained similar formatting errors.

**Treatment:**

- Removed complete duplicates, followed by the remaining duplicate SKU.
- Filled missing descriptions with the corresponding product name.
- Did not impute `type` because the numerical code's meaning was unclear.
- Removed products with missing or unrecoverable prices.
- Discarded `promo_price` from discount calculations because it was too corrupted to trust.

> **Important:** Discounts could not be calculated directly from `promo_price`. Instead, they were reconstructed by comparing catalog prices with actual prices paid in `orderlines`.

### `brands.csv`

Six brand names appeared with two different short codes: Apple, Bose, Jaybird, Mophie, Startech, and Unknown. No automatic correction was applied because the source did not provide enough information to identify the correct code. The inconsistency was documented while preserving the short code as the product-join key.

## Discount Calculation

The reconstructed discount percentage was calculated as:

\[
\text{Discount \%} = \frac{\text{Catalog Price} - \text{Actual Paid Price}}{\text{Catalog Price}} \times 100
\]

## Categorization Strategy

Product categories were not taken directly from the numerical `type` column because its meaning was unclear and it contained missing values. Instead, categories were constructed in the analysis layer.

- **Primary signal:** text patterns in `products.name`. For example, products beginning with `iMac` were assigned to the `iMac` category, while names containing `iPhone` or `Tablet` were assigned accordingly.
- **Supplementary signal:** brand codes from `brands.csv`, which appear at the start of product SKUs.
- **Objective:** create revenue-focused, board-readable categories such as `iMac`, `iPhone`, and `Tablet` rather than opaque numerical codes.
- **Price heterogeneity:** each category includes entry-level and professional high-end variants, so a single discount rate should not automatically be applied to an entire category.
- **Implementation note:** categorization was implemented in Python. Future versions should store categories consistently in the source data.

The top five revenue-driving categories generated approximately 55% of total revenue.

## Key Findings

### Discounting Is the Norm

- 91% of sold items were sold below catalog price.
- The average discount was approximately 22.5% per transaction.
- Only 9% of sales occurred at full price.

### Seasonality Matters

- Black Friday produced an extreme single-day revenue peak of approximately ten times normal daily revenue.
- The pre-Christmas period showed a longer, more gradual increase in sales over several weeks.
- Black Friday likely cannibalized part of the traditional Christmas shopping period because customers completed purchases earlier to access the strongest discounts.

### A Few Premium Categories Drive Revenue

- The five highest-revenue categories generated approximately 55% of total revenue.
- iMac and iPhone were the leading categories.
- These categories contained a wide range of prices, from entry-level models to professional high-end variants.

### Higher Discounts Do Not Guarantee Higher Revenue

| Category | Revenue | Average Discount |
|---|---:|---:|
| iMac | Highest | Low |
| Tablet | Low | High |

These results suggest that iMac generated the highest revenue with a comparatively low discount, while tablets received higher discounts but contributed less revenue.

## Methodology and Tools

- Python
- pandas and NumPy for data preparation and analysis
- Matplotlib and Seaborn for visualization
- Jupyter Notebooks for exploratory analysis
- Git and GitHub for version control

## Recommended Repository Structure

```text
eniac-discount-strategy/
├── README.md
├── data/
│   ├── orders.csv
│   ├── orderlines.csv
│   ├── products.csv
│   └── brands.csv
├── notebooks/
│   ├── 01_data_exploration.ipynb
│   ├── 02_data_cleaning.ipynb
│   ├── 03_analysis.ipynb
│   └── 04_visualization.ipynb
├── reports/
│   └── Eniac-Rabattstrategie.pdf
├── .gitignore
└── LICENSE
```

## Recommendations

1. **Concentrate promotions around key dates.** Focus discounts on Black Friday and the weeks before Christmas instead of maintaining broad, continuous discounts.
2. **Protect margins in premium categories.** Avoid aggressive discounts on high-revenue categories such as iMac and iPhone, which perform well with lower discounts.
3. **Improve data collection and pipeline reliability.** Record shipping costs separately, store product categories consistently, and fix data transfer between the online store and database.
4. **Add cost data.** Purchase or manufacturing costs are needed to calculate margin impact by category and promotion.
5. **Repeat the analysis.** Re-run the analysis after future Black Friday and Christmas periods to determine whether these patterns recur.

## Limitations

- The dataset covers only one annual cycle, including one Black Friday and one Christmas period.
- Cost and margin data were unavailable, so the analysis measures revenue rather than profit.
- The corrupted `promo_price` field required indirect discount reconstruction.
- The data cannot fully establish that discounts caused the observed seasonal sales patterns.
- Some product and brand inconsistencies could be documented but not reliably corrected.

## Team

This project was completed collaboratively by **Navid**, **Georgie**, and **Mehrnoosh**. The team worked across data exploration, cleaning, validation, analysis, visualization, interpretation, and recommendations. Each member independently produced analysis outputs before the team compared results and selected the final approach through discussion.

## License

This project is intended for educational purposes.

## Authors

ENIAC Data Analytics Team: Navid, Georgie, Mehrnoosh

_Last updated: September 2026._
