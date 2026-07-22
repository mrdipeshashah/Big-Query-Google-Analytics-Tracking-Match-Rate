# OVERVIEW

This repository contains the data requirements, BigQuery SQL architecture, and Data Studio configurations for monitoring the revenue tracking health between Shopify (or any other E-commerce platform) v Google Analytics 4 (GA4).

## DASHBOARD

To track match rate this dashboard (https://datastudio.google.com/reporting/871d3ef2-a853-49ef-94c1-4bbd9bfb1608) helps bring the insights to life. 

## Objective
To provide automated, daily visibility into data discrepancies between Shopify (source of truth) and GA4 (marketing attribution). This pipeline highlights tracking failures when the daily match rate drops below the target threshold of **90%**.

## DATA REQUIREMENTS & PREREQUISITES

To build a match rate view it requires a daily-aggregated breakdown of matched dates and raw revenue numbers from Google Analytics and Shopify or Stripe or any other E-commerce Platform. 

### Schema Requirements
| Column Name | Data Type | Description |
| :--- | :--- | :--- |
| `Date` | DATE | The calendar date of the transactions (`YYYY-MM-DD`). |
| `Shopify_Revenue` | FLOAT64 / NUMERIC | Total gross/net revenue recorded in Shopify (Source of Truth). |
| `GA_Revenue` | FLOAT64 / NUMERIC | Total purchase revenue recorded in Google Analytics 4. |

## CALULATED FIELDS & LOGIC

*   **Revenue Difference (£):** 
    `Shopify_Revenue - GA_Revenue`
    *Identifies the exact amount of revenue under-reported or missed by the GA4 tag.*
*   **Match Rate (%):** 
    `GA_Revenue / Shopify_Revenue`
    *Represents the efficiency of the tracking setup. The target operational benchmark is >= 90%.*


