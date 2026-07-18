# Shopify vs. GA4 Revenue Match Rate Dashboard

This repository contains the data requirements, BigQuery SQL architecture, and Looker Studio metric configurations for monitoring the revenue tracking health between Shopify and Google Analytics 4 (GA4).

## Objective
To provide automated, daily visibility into data discrepancies between Shopify (source of truth) and GA4 (marketing attribution). This pipeline highlights tracking failures (e.g., pixel drops, broken cookie consent, checkout scripting issues) when the daily match rate drops below the target threshold of **90%**.

---

## 📋 1. Data Requirements & Prerequisites

The pipeline expects a daily-aggregated source table containing matched dates and raw revenue numbers from both systems.

### Schema Requirements
| Column Name | Data Type | Description |
| :--- | :--- | :--- |
| `Date` | DATE | The calendar date of the transactions (`YYYY-MM-DD`). |
| `Shopify_Revenue` | FLOAT64 / NUMERIC | Total gross/net revenue recorded in Shopify (Source of Truth). |
| `GA_Revenue` | FLOAT64 / NUMERIC | Total purchase revenue recorded in Google Analytics 4. |

---

## 💻 2. BigQuery Data Transformation (SQL View)

To optimize performance and avoid redundant calculated fields in the visualization layer, the raw metrics are transformed via a BigQuery View. 

Run the following query in your Google Cloud Console and save it as a View (e.g., `v_MatchRate_Dashboard`):

```sql
SELECT
  Date,
  Shopify_Revenue,
  GA_Revenue,
  -- Calculate the exact absolute monetary gap
  (Shopify_Revenue - GA_Revenue) AS Revenue_Difference,
  -- Calculate match rate percentage; handles division by zero safely
  IF(Shopify_Revenue > 0, GA_Revenue / Shopify_Revenue, 0) AS Match_Rate
FROM
  `mrdipeshashah.lookerstudiodashboards.MatchRate_GoogleAnalytics_Shopify`
```

---

## 🧮 3. Calculated Fields & Logic

*   **Revenue Difference (£):** 
    `Shopify_Revenue - GA_Revenue`
    *Identifies the exact amount of revenue under-reported or missed by the GA4 tag.*
*   **Match Rate (%):** 
    `GA_Revenue / Shopify_Revenue`
    *Represents the efficiency of the tracking setup. The target operational benchmark is >= 90%.*

---

## 🎨 4. Looker Studio Dashboard Configuration

Connect Looker Studio directly to your saved BigQuery View and configure the following components:

### Summary Scorecards (Top Row)
1.  **Average Match Rate**
    *   **Metric:** `Match_Rate` (Aggregation: `Average`, Format: `Percent`)
    *   **Conditional Formatting:** Background fills **Green** if value is >= 0.90.
2.  **Total Discrepancy**
    *   **Metric:** `Revenue_Difference` (Aggregation: `Sum`, Format: `Currency GBP`)
3.  **Tracking Alert Days**
    *   **Metric:** `Date` (Aggregation: `Count Distinct`)
    *   **Missing Data Setting:** Select **"Show 0"** (ensures a clean `0` displays on perfect tracking weeks instead of a blank box).
    *   **Filter Panel:** Add an inline chart filter: `Include Match_Rate Less than (<) 0.9`
    *   **Conditional Formatting:** Background fills **Red** if value is > 0.

### Main Visualization: Combo Chart
*   **Title:** `DAILY TRACKING HEALTH AND REVENUE DISCREPANCY`
*   **Dimension:** `Date` (Granularity: `Day`)
*   **Metrics:** 
    *   `Match_Rate` -> Style: **Line** (Plot on **Left Axis**; Axis Min: `0.70`, Axis Max: `1.00`)
    *   `Revenue_Difference` -> Style: **Bars** (Plot on **Right Axis**)
*   **Reference Line:** Add a constant metric reference line on the Left Y-Axis at `0.90`, labeled **"Baseline"**.

### Action Item Table (Audit Log)
*   **Title:** `TRACKING FAILURES AND HIGH DISCREPANCY AUDIT LOG`
*   **Dimensions:** `Date`, `Shopify_Revenue`, `Google Analytics Revenue`, `Revenue_Difference`, `Match_Rate`
*   **Sorting:** `Revenue_Difference` (Descending)
*   **Table Filter:** Reuse the `Match_Rate < 0.9` filter to isolate and surface only the days requiring dev team investigation.
