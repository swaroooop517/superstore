# Business & Financial Performance Analysis: Superstore Retail

An end-to-end analytics project using **SQL, Python, Excel and a BI layer** to find where a US retailer makes and loses money.
It turns 9,994 raw order lines (2014-2017) into a star schema, 29 SQL queries, an interactive dashboard, a formula-driven Excel
model, a discount what-if scenario and a loss-prediction model.

**Live dashboard:** `https://<your-github-username>.github.io/<repo-name>/` (enable GitHub Pages: see [Publish](#publish-the-live-dashboard))

![Yearly performance](docs/images/01_yearly_performance.png)

## Business questions
1. How have sales, profit and margin trended over four years?
2. Which categories, products, regions and states create or destroy profit?
3. How much does discounting cost the business, and what would a discount cap be worth?
4. Which customers drive profit, and which lose money?

## Key findings
| # | Finding | Evidence |
|---|---|---|
| 1 | Growth is strong but margin softened in 2017 | Sales CAGR 14.8%, profit CAGR 23.6%; margin 13.4% (2016) to 12.7% (2017) |
| 2 | Furniture earns almost nothing | 32% of sales, 6% of profit, 2.5% margin; Tables (-8.6%), Bookcases (-3.0%), Supplies (-2.5%) lose money |
| 3 | Discounts above 20% destroy profit | 1,393 order lines lose **$135K**; no-discount lines earn a 29.5% margin; removing deep-discount lines would lift profit 47% |
| 4 | Regional gap is a discount-policy gap | Central: 24% avg discount, 7.9% margin. West: 10.9% discount, 14.9% margin. TX, OH, PA, IL each lose $13-26K |
| 5 | Profit is concentrated | Top 20% of customers deliver 81% of profit; 155 customers are loss-making ($-71K combined) |
| 6 | Strong year-end seasonality | September, November and December are the three biggest sales months (avg $77K, $88K, $81K vs a $38K typical month) |

![Discount bands](docs/images/05_discount_bands.png)

### What-if: cap discounts
Cutting discounts above a cap down to the cap (unit cost unchanged) changes profit as follows, depending on how many affected
order lines still sell after the price rise:

| Discount cap | 100% retained | 50% retained | 0% retained (lines lost) |
|---|---|---|---|
| 30% | +51% | +47% | +44% |
| **20%** | **+77%** | **+62%** | **+47%** |
| 10% | +136% | +76% | +15% |

A 20% cap improves profit by roughly **47% to 77%** in this simple model. These are scenario estimates, not forecasts:
real demand response is unknown (see [Assumptions](#assumptions-and-limitations)).

![Scenario](docs/images/10_discount_cap_scenario.png)

### Predictive model: which order lines lose money?
| Model | AUC | Accuracy | Precision | Recall |
|---|---|---|---|---|
| Rule: discount > 20% | n/a | 0.944 | 0.969 | 0.724 |
| Logistic regression | 0.986 | 0.946 | 0.937 | 0.763 |
| Random forest | 0.985 | 0.949 | 0.957 | 0.761 |

Discount is by far the dominant driver (permutation importance), followed by sub-category. A one-line business rule is nearly as
accurate as the machine-learning models, which is why the main recommendation is a policy rule rather than a model.

![Feature importance](docs/images/11_feature_importance.png)

## Recommendations
1. Cap discounts at 20% and require manager approval above it.
2. Reprice or rationalise Tables and Bookcases; renegotiate supplier cost.
3. Audit discounting in the Central region and in TX, OH, PA and IL.
4. Push high-margin lines (Copiers, Paper, Accessories, Phones) and target Home Office and Corporate customers.
5. Track margin, not just sales growth, as a headline KPI.

## Tech stack
| Layer | Tools |
|---|---|
| Data modelling and querying | SQL (SQLite): star schema, CTEs, window functions (LAG, RANK, NTILE, running totals) |
| Data prep, analysis, modelling | Python: pandas, NumPy, matplotlib, scikit-learn |
| Spreadsheet model | Excel (openpyxl): SUMIFS-driven tabs, charts, heat map, key insights |
| BI | Interactive HTML dashboard (Chart.js, GitHub Pages) plus Power BI guide with DAX measures |
| Engineering | Git, GitHub Actions CI, pytest data-reconciliation tests |

## Repository structure
```
data/raw/            original Superstore CSV
data/processed/      cleaned CSV and SQLite database (superstore.db)
sql/                 star-schema DDL + 5 files with 29 named queries
python/              pipeline scripts 01 to 05 and run_all.py
excel/               formula-driven workbook with charts
bi/                  Power BI guide, DAX measures, star-schema CSVs
docs/                dashboard (index.html) and chart images
outputs/             SQL result CSVs, scenario and model outputs
tests/               pytest checks (row counts, reconciliation, keys)
.github/workflows/   CI that runs the whole pipeline on every push
```

## How to run
```bash
git clone https://github.com/<your-github-username>/<repo-name>.git
cd <repo-name>
python -m venv .venv && source .venv/bin/activate      # Windows: .venv\Scripts\activate
pip install -r requirements.txt
python python/run_all.py        # builds DB, runs 29 SQL queries, charts, model, dashboard, workbook
pytest -q                        # verifies totals reconcile with the source data
```
Open `docs/index.html` in a browser for the dashboard, or `excel/Superstore_Business_Financial_Analysis.xlsx` for the model.
To explore the database directly: `sqlite3 data/processed/superstore.db` then `.read sql/03_kpi_and_trend_analysis.sql`
(or open it in DB Browser for SQLite).

## Data model
Star schema: `fact_sales` (one row per order line) with `dim_date`, `dim_customer`, `dim_location`, `dim_product`, `dim_ship_mode`
plus the analytical view `vw_sales_flat`. See [`sql/01_schema.sql`](sql/01_schema.sql).

## Data quality
- 9,994 rows, no missing values, no fully duplicated rows, no ship-before-order dates.
- 32 Product IDs map to more than one product name, so the product dimension is keyed on (Product ID, Product Name).
- 8 order/product pairs appear on two lines (kept, since they are valid separate lines).
- Pipeline asserts that SQL totals reconcile with the raw CSV (Sales $2,297,201; Profit $286,397).

## Assumptions and limitations
- The dataset has no cost, overhead, balance sheet or cash flow data, so "financial performance" means sales, profit, margin and discount behaviour.
- Scenario model: Sales is net of discount, so list price = Sales / (1 - Discount); unit cost = Sales - Profit and stays constant;
  no change in customer behaviour other than the retention percentage shown.
- Findings show association, not proven causation. Product cost structure also affects margin.
- Customer and order data are a sample dataset, not real company results.

## Publish the live dashboard
1. Push this repo to GitHub.
2. **Settings > Pages > Build and deployment > Deploy from a branch > `main` / `/docs`** and Save.
3. After a minute the dashboard is live at `https://<your-github-username>.github.io/<repo-name>/`. Put that link at the top of this README.

## Author
[Your Name] | MBA, [University] | [LinkedIn] | [Email]

*Dataset: Sample Superstore (public sample data commonly used for BI training).*
