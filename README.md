# E-Commerce Order Analytics Dashboard
### Excel | Power Query | Multi-Channel Sales Intelligence

---

## Overview

Built a consolidated analytics solution for a multi-channel e-commerce operation selling across three platforms — a proprietary website, noon.com, and Amazon.ae. Each platform exports independent monthly transaction files in different formats. The goal was to design a scalable data pipeline that merges these sources, resolves data quality issues, enriches transactional data with customer and product dimensions, and delivers a unified dashboard for commercial decision-making.

---

<img width="1813" height="651" alt="DOM_Dashboard" src="https://github.com/user-attachments/assets/62da56e1-b524-4dfd-bb83-ccda896f0dc3" />

## Business Context

E-commerce operations running across multiple marketplaces face a common challenge — fragmented data. Revenue, customer behaviour, and product performance exist in silos, making it impossible to answer cross-channel questions without manual consolidation. This project eliminates that gap by building a single source of truth updated by dropping new monthly files into a folder and hitting refresh.

---

## Data Architecture

```
├── DOM_Dashboard.png
└── DOM_Dashboard.xlsx  ← Single workbook: pipeline + dashboard
```

**Three transactional sources** feed into one enriched master table via Power Query. Reference data is maintained separately and joined at query time, meaning updates to the customer or product master propagate automatically on refresh.

---

## Technical Implementation

### Power Query Pipeline

**Data Ingestion**
- Connected each channel folder using Power Query's Folder connector, enabling dynamic file detection — new monthly files are picked up automatically without modifying any queries
- Appended three channel queries into a single master orders table using `Table.Combine`
- Added a `channel` identifier column at ingestion to preserve source context post-merge

**Data Quality & Standardisation**
Resolved the following real-world inconsistencies across source files:

| Issue | Resolution |
|---|---|
| SKU format inconsistency (SKU-2201 vs sku2201 vs SKU2201) | Normalised via UPPERCASE transformation and conditional dash insertion |
| Guest checkouts with no customer ID | Standardised null `customer_id` to "Guest"; platform-specific anonymous emails unified |
| Cancelled orders inflating volume metrics | Filtered at pipeline level; returned orders retained with status flag for return rate analysis |

**Dimensional Enrichment**
- Performed Left Outer join against CRM customer export on `customer_id` to append `segment`, `city`, and `registration_date`
- Performed Left Outer join against product catalog on normalised `sku` to append `product_name`, `category`, `subcategory`, and `cost_price`
- Null segments on guest rows handled explicitly — labelled "Guest" rather than left blank to avoid silent exclusion from PivotTable aggregations

**Calculated Metrics**

| Metric | Logic |
|---|---|
| `net_revenue` | `(unit_price × quantity) − discount_amount` |
| `gross_profit` | `net_revenue − (cost_price × quantity)` |
| `margin_pct` | `gross_profit ÷ net_revenue` |
| `order_month` | Extracted as MMM-YYYY for time series aggregation |
| `order_count` | Helper column (value = 1) for reliable PivotTable count aggregations |
| `total_orders` | Grouped order count per customer via `Table.Group` on master orders table — merged back into enriched table to avoid iterative row-by-row evaluation |
| `is_repeat` | Binary flag derived from `total_orders > 1` |

**Performance Consideration**
Repeat customer identification was implemented using `Table.Group` rather than row-level iteration (`List.Select`, `Table.SelectRows`). The grouped approach runs in O(n) versus O(n²), critical for datasets that grow month-on-month.

---

## Dashboard

Four analytical views built on PivotTables, connected via shared slicers (`channel`, `segment`, `order_month`):

**1. Revenue by Channel**
Clustered bar chart with margin % overlay — identifies which platform drives volume versus value.

**2. Customer Segment Analysis**
Revenue, order count, and AOV broken down by segment (VIP, Returning, New, Guest) with city-level drill down.

**3. Product Profitability**
Horizontal bar chart ranked by margin % — surfaces high-revenue/low-margin products that erode profitability despite strong top-line contribution.

**4. Repeat Customer Rate**
Repeat vs one-time customer breakdown per channel — indicates platform loyalty and customer retention health.

---

## Refresh Workflow

Monthly update process:
1. Drop new channel CSV files into the corresponding `Orders/` subfolder
2. Replace `customers.csv` with latest CRM export
3. `Data → Refresh All`

No query modifications required. Pipeline handles new files, new customers, and new SKUs automatically.

---

## Dataset

Sample dataset covers Q1 2025 (January–March) across three channels. Includes realistic data quality issues: mixed date formats, inconsistent SKU conventions, guest checkouts, and deleted account references. New customers introduced monthly to simulate organic CRM growth.

| Month | Orders (pre-filter) | Channels |
|---|---|---|
| January 2025 | ~196 | Website, Noon, Amazon |
| February 2025 | ~163 | Website, Noon, Amazon |
| March 2025 | ~186 | Website, Noon, Amazon |

---

## Tools

- Microsoft Excel (Microsoft 365)
- Power Query (M Language)

## Contact

**Gilchrist Jose**  
📁 [GitHub](https://github.com/gilchrist-jose) | 💼 [LinkedIn](https://www.linkedin.com/in/gilchrist-jose-a96658322/) | ✉️ gill.christ11@gmail.com

---

`Excel` `Power-Query` `ETL` `Automated-Refresh` `Data-Pipeline` `Business-Intelligence`
