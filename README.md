# ENIAC-Discount-Strategy-Analysis
A data-driven investigation into the impact of discounts on ENIAC's e-commerce performance.
## Project Overview
This repository contains the analysis for ENIAC's second data project. The objective is to assess whether discounting products supports sustainable business growth or negatively affects revenue and margins.
The project addresses an internal debate between the Marketing Team, which views discounts as a driver of customer acquisition, satisfaction, and retention, and the Board's investors, who are concerned about aggressive discounting and the company's positioning in the quality segment.
## Business Questions
	How many products are being discounted?
	How large are the discounts as a percentage of product prices?
	How do seasonality and special dates such as Christmas and Black Friday affect sales?
	How should products be classified to simplify reporting and analysis?
	What is the distribution of product prices across categories?
	How could data collection and data quality be improved?
## Data Sources
The analysis uses four CSV files:
File	Description
orders.csv	One row per order, including order date, total paid, and order state.
orderlines.csv	One row per product line in an order, including SKU, quantity, unit price, and processing date.
products.csv	Product catalog containing SKU, name, description, base price, promotional price, stock status, and product type.
brands.csv	Mapping between three-character brand codes and brand names.

Exploration Findings
The exploratory phase revealed substantial quality and consistency issues across the source tables. These findings shaped the cleaning strategy and the interpretation of the final results.
orders.csv
Problems identified:
	created_date was stored as text rather than as a datetime field.
	Five values were missing in total_paid.
	total_paid contained extreme values, reaching approximately €214,747.
	The state column contained five order states; only Completed represented paid and completed sales.
Treatment:
	Converted created_date to a proper datetime format.
	Removed the five missing total_paid records because they all belonged to Pending orders and represented only 0.035% of that state.
	Retained the extreme values for documentation. They occurred only in Shopping Basket orders and therefore did not affect completed-revenue analysis.
	Restricted revenue analysis to Completed orders.
orderlines.csv
Problems identified:
	unit_price was stored as text.
	Approximately 36,169 unit_price values contained formatting errors, such as duplicated decimal points (1.137.99).
	date was stored as text rather than as a datetime field.
	product_id was constant at 0 and therefore unusable.
	product_quantity contained extreme values, reaching 999 units.
	One negative unit price (-119.0) was detected.
	865 rows had a unit price of €0, probably representing free items.
	Differences between order-line totals and total_paid frequently matched shipping-fee amounts such as €6.99, €4.99, and €3.99.
Treatment:
	Repaired the duplicated-decimal formatting in unit_price where the value could be reliably reconstructed.
	Validated the repair in three ways: comparison with products.price, review of affected product names, and comparison with order totals. The reconstructed line values matched orders.total_paid with 99.97% agreement.
	Set three unrecoverable price values to NaN; none belonged to a completed order.
	Corrected the negative price sign because its absolute value matched the corresponding real product price.
	Preserved €0 values and created an is_free_item indicator instead of deleting them. Forty-nine free-item rows belonged to completed orders and remained relevant for price analysis.
	Converted date to datetime and removed the unusable product_id field.
	Validated extreme quantities against order totals. The largest quantities were consistent with real orders, including possible bulk purchases, and were therefore not automatically removed. Nine of the ten most extreme cases belonged to Shopping Basket orders.
	Documented the likely presence of shipping costs that were not stored in a separate field.
products.csv
Problems identified:
	sku was not reliably unique: 8,746 complete duplicate rows and one additional SKU duplicate were found.
	Missing values occurred in desc (7), price (46), and type (50).
	price, promo_price, and type were stored as text.
	542 price values contained formatting problems, including duplicated decimal points and excessive decimal places.
	More than 92% of promo_price values contained similar formatting errors.
Treatment:
	Removed duplicates in two stages: complete duplicate rows first, followed by the remaining duplicate SKU.
	Filled missing descriptions with the corresponding product name.
	Did not impute type because the meaning of the numerical code was unclear.
	Removed rows with missing price when the value could not be reconstructed from orderlines.
	Removed the 542 products with unreliable price formatting when the original value could not be recovered without loss of precision.
	Discarded promo_price from the discount calculation because more than 90% of the column was corrupted and could not be trusted.
Important consequence: discounts could not be calculated directly from promo_price. Instead, discounts were reconstructed by comparing catalog prices with actual prices paid in orderlines.
brands.csv
Problems identified:
	Six brand names appeared with two different short codes: Apple, Bose, Jaybird, Mophie, Startech, and Unknown.
Treatment:
	No automatic correction was applied because the source did not provide enough information to identify the correct code.
	The inconsistency was documented, while preserving the short code as the key used to connect brands with products.
	Otherwise, the table contained no missing values or duplicate rows and had appropriate data types.
Data Quality Strategy
The cleaning process prioritized preserving real sales information over aggressively deleting unusual observations. Suspicious values were cross-checked against related tables, order totals, product names, order states, and business logic before being removed or modified.
The reconstructed discount percentage was calculated as:
"Discount \%"=("Catalog Price" -"Actual Paid Price" )/"Catalog Price" ×100
Categorization Strategy
Product categories were not taken directly from the type column in products.csv, because that field is a numerical code with unclear meaning and 50 missing values. Instead, categories were constructed during the analysis layer using the following approach:
	Primary signal: product name
Categories were defined based on text patterns in products.name. For example, products whose names start with iMac were assigned to the iMac category, products containing iPhone to the iPhone category, and products containing Tablet to the Tablet category, and so on.
	Supplementary signal: brand code
The brands.csv table provides three-character brand codes that appear at the start of products.sku. These codes were available as a secondary signal, but the final categories used in the analysis (such as iMac, iPhone, Tablet) are closer to "brand + product type" than to brand alone.
	Objective: revenue-focused and board-readable
The categories were designed to:
	Enable calculation of each category's share of total revenue.
	Identify the top five revenue-driving categories (which together generate about 55% of total revenue).
	Use labels that are meaningful to non-technical stakeholders (e.g., iMac, iPhone, Tablet) rather than opaque numerical codes.
	Within-category price heterogeneity
Each category contains a wide price range, from entry-level models to professional high-end variants of the same product type. This is why the analysis cautions against applying a single uniform discount rate to an entire category: entry-level and high-end products likely require different pricing strategies.
	Implementation note
Categorization was implemented in the analysis code (Python) rather than being cleanly stored in the database. One of the recommendations is to store product categories correctly and consistently from the start, so that future analyses can rely on a stable, well-defined category field.
Team and Collaboration
This project was completed by a three-person team: Navid, Georgie, and Mehrnoosh.
All three team members worked collaboratively across all project phases, including:
	Data exploration and quality assessment
	Data cleaning and validation
	Analysis and visualization
	Interpretation of results and formulation of recommendations
Each team member independently produced their own analysis outputs. The team then compared results, and each member presented and defended their approach and findings. The final version of the analysis was selected through team discussion: the output that was judged most logical and well-supported by the majority of team members was chosen as the final deliverable.
This collaborative process ensured that key decisions (such as handling of corrupted fields, outlier treatment, and categorization logic) were critically reviewed from multiple perspectives before being finalized.
Key Findings
Discounting Is the Norm
	91% of sold items were sold below catalog price.
	The average discount was approximately 22.5% per transaction.
	Only 9% of sales occurred at full price.
Seasonality Drives Revenue More Than Discounts Alone
	Black Friday produced an extreme, single-day revenue peak, approximately ten times normal daily revenue.
	The pre-Christmas period showed a longer and more gradual increase in sales over several weeks.
	Black Friday likely cannibalizes part of the traditional Christmas shopping period, as customers complete purchases earlier to benefit from the best discounts.
Few Premium Categories Carry the Business
	The five highest-revenue categories generated approximately 55% of total revenue.
	iMac and iPhone were the leading categories.
	Within these categories, price ranges spanned from entry-level models to professional high-end variants of the same product type.
Discount ≠ Revenue Driver
Category	Revenue Share	Average Discount
iMac	Highest	Low
Tablet	Low	High

	iMac generated the highest revenue with a comparatively low discount.
	Tablets received higher discounts but contributed less revenue.
	High discounts do not automatically mean high revenue.
Methodology and Tools
	Python
	pandas and NumPy for data preparation and analysis
	Matplotlib and Seaborn for visualization
	Jupyter Notebooks for exploratory analysis
	Git and GitHub for version control
Recommended Repository Structure
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
├── src/
│   ├── data_loading.py
│   ├── data_cleaning.py
│   ├── analysis.py
│   └── visualization.py
├── reports/
│   └── final_presentation.pdf
├── .gitignore
└── LICENSE

Recommendations
Strategic Recommendations
	Concentrate promotions around key dates
Focus discounts on Black Friday and the weeks before Christmas, where the revenue effect is demonstrably largest, instead of maintaining broad, continuous discounts.
	Protect margins in premium categories
Avoid aggressive discounts on high-revenue categories such as iMac and iPhone. These products perform well with lower discounts, allowing the company to preserve margins.
	Improve data collection and pipeline reliability
	Record shipping costs separately instead of embedding them in the total amount.
	Store product categories consistently and correctly from the start.
	Fix the data pipeline between the online store and the database to prevent future corruption.
	Incorporate cost data for margin analysis
Add purchase or manufacturing costs to enable direct calculation of margin impact per category and per promotion.
	Repeat the analysis over multiple annual cycles
Run the same analysis after the next Black Friday and Christmas period to confirm that the observed patterns are recurring and not one-off effects.
Limitations
	The dataset covers only one annual cycle, including one Black Friday and one Christmas period.
	Cost and margin data were unavailable, so the analysis measures revenue rather than profit.
	The corrupted promo_price field required indirect discount reconstruction.
	The data cannot fully establish that discounts caused the observed seasonal sales patterns.
	Some product and brand inconsistencies could be documented but not reliably corrected.
License
This project is intended for educational purposes.
Authors
ENIAC Data Analytics Team : Navid, Georgie, Mehrnoosh
Last updated: September 2026
<img width="476" height="644" alt="image" src="https://github.com/user-attachments/assets/2a3ad2c9-33a0-42f2-8239-144167c5793d" />
