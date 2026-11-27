# Safety Stock Analysis (OM 620)

This project cleans and analyzes about 712,000 inventory transactions to calculate **safety stock by SKU** for **make-to-stock (MTS)** products, so the business can hold enough buffer inventory to meet demand at a chosen service level.

Safety stock is calculated as:

$$SS = z_{\alpha} \times \sigma_{d} \times \sqrt{LT}$$

where $z_{\alpha}$ is the z-score for the service level, $\sigma_{d}$ is the standard deviation of demand (order quantity), and $LT$ is the lead time in days.

---

## Repository Structure

```
├── data/
│   ├── transaction_data.csv   Raw inventory transactions (711,662 rows, 9 columns)
│   └── meta_data.csv          Description of each feature
└── notebooks/
    └── safety_stock_analysis.ipynb   Combined notebook for Assignments 1 and 2
├── README.md
```

## Data

| Feature | Description |
|---|---|
| `sku_number` | Eight-character stock keeping unit number |
| `inventory_type` | Finished good (FG), work in progress (WIP), or raw material (RM) |
| `stocking_type` | Stocking policy, e.g. make to stock (MTS), make to order (MTO), assemble to order (ATO) |
| `lead_time` | Total lead time in days |
| `unit_price` | Unit price in US dollars |
| `manufacturing_site` | Site ID starting with a country code (e.g. `PL-38`) |
| `division_code` | Four-character business division ID |
| `transaction_date` | Transaction date (Year-Month-Day) |
| `order_quantity` | Order quantity |

---

## Assignment 1: Data Cleaning and Exploration

Prepared the raw transaction data and explored the finished-good, make-to-stock subset.

**Accomplishments and findings:**
- **Column formatting:** Renamed three inconsistent column names (`leadTime`, `division Code`, `Order-Quantity`) to match the metadata convention.
- **Anomalies found and removed:**
  - 36,437 transactions with negative order quantities (likely returns, cancellations, or scrap)
  - 4,148 transactions with an order quantity of 0
  - 10 transactions with negative unit prices (likely credits or cost adjustments)
  - 10 lead-time outliers of 1,400–2,800 days (more than 3 standard deviations from the mean)
  - This left 671,057 transactions, with lead times between 14 and 28 days.
- **Missing values:** Removed 298 transactions with no SKU. Kept transactions with missing inventory type (12,511), stocking type (13,224), manufacturing site (130,188), and division code (47,011), since these rows still hold valid demand and lead-time data.
- **New country feature:** Created a `country` column from the manufacturing site prefix (Canada, Japan, Poland, USA).
- **Finished-good MTS subset:** 236,212 transactions covering 375 unique SKUs, 19 manufacturing sites, and 54 divisions.
- **Largest transaction:** 198 units of SKU `7E0B9A17` (site CA-B4), worth about $196,236, the highest by both quantity and sales value.

## Assignment 2: Safety Stock Calculation

Revised the cleaning approach, aggregated demand by SKU, and calculated safety stock at 75%, 90%, and 95% service levels.

**Accomplishments and findings:**
- **Revised cleaning:** Rather than dropping anomalies, lead-time outliers were treated as data entry errors and corrected (e.g., 2,800 → 28 days), negative quantities and prices were converted to absolute values, zero quantities were dropped, and missing categories were filled with "Unknown." Filtering to MTS left 442,022 transactions.
- **Aggregation by SKU:** Built a table of 497 SKUs with the minimum, maximum, mean, median, variance, and standard deviation of order quantity, plus average lead time.
- **Demand distribution check:**
  - 100 SKUs have stable demand (coefficient of variation ≤ 0.3), so a **normal distribution** was used.
  - The remaining 397 SKUs have erratic demand (CV > 0.3, variance-to-mean ratio ≥ 1.2), so a **negative binomial distribution** was used, with safety stock calculated as reorder point minus mean demand during lead time.
- **Results at a 95% service level:**
  - Largest safety stock: SKU `AC3ADABF` at 370 units
  - Smallest safety stock: SKU `58F77D5E` at 30 units
  - Average safety stock: about 123 units per SKU
- **Sidequest (non-parametric method):** An empirical quantile approach produced a lower average safety stock of about 114 units (maximum 323 for `AC3ADABF`, minimum 32 for `58F77D5E`), offering a more conservative option for the business.

---

## How to Run

1. Clone or download this repository.
2. Install the required packages:
   ```
   pip install pandas numpy scipy plotnine jupyter
   ```
3. Open `notebooks/safety_stock_analysis.ipynb` and run all cells. The notebook reads data from `../data/`.

## Tools

Python, pandas, NumPy, SciPy, Jupyter Notebook
