---
description: >-
  The Market Analysis module combines claims & remittance data to explore
  real-world utilization at a detailed level.
---

# 🖥️ Market Analysis - Analyzing Claims and Remits

## Market View

The **Market View** screen provides a high-level overview of activity across the entire United States and serves as the primary entry point for exploratory analysis. From this view, you can begin drilling into areas of interest across geography, providers, billing codes, and payers.

### Geography Overview & Key Metrics

<figure><img src="../../.gitbook/assets/market view one.gif" alt=""><figcaption></figcaption></figure>

At the top of the screen, the interactive map displays nationwide activity. As you hover over the map, you will see summary metrics for the selected geography, including claims and remittance counts.

All map interactions update dynamically as you hover or select different regions.

#### Key Metrics by Geography

Accompanying the map is a **Key Metrics** table that summarizes activity by geography level.

This table is fully interactive. Hovering over any row will automatically update the map to reflect the selected geography, providing immediate visual context for where activity is concentrated.

***

### Claims and Remits by Service and Provider

Further down the page are two key metrics tables that break claims and remits down by:

* Billing code & Provider groupings
* Payer Channel Groupings & Payer Detail

These tables allow you to quickly identify which services and providers are driving utilization. As with the map and geography tables, all elements are interactive and update the rest of the view as you hover or select rows.

#### Service & Provider Breakdown

<figure><img src="../../.gitbook/assets/service &#x26; provider.gif" alt=""><figcaption></figcaption></figure>

The first chart allows to further refine your analysis by both Provider & Service groupings. It is common to focus on a subset of taxonomies (ex. Hospitals vs. Labs) and a subset of service lines.\
\
You can switch the Grouping in the chart using the dropdown and filter the data by clicking rows in the chart or by using the global filters at the top.

#### Payer Breakdown

<figure><img src="../../.gitbook/assets/LG FULL HD (1).gif" alt=""><figcaption></figcaption></figure>

The next chart focuses on payer-level breakdowns. Metrics are shown by:

* Payer Channel (for example, Commercial, Medicare, Medicaid)
* Payer Name

You can start at the channel level to understand high-level mix, then drill into a specific channel (such as Commercial) to see detailed payer-level performance. These interactions update all related visualizations in real time.

#### Cross-Tab and Market Share Analysis

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

At the bottom of the screen, the scorecard and grid enable deeper cross-tab analysis across multiple dimensions and metrics.

These views:

* Compare dimensions side-by-side against claims, remits, or average remit
* Automatically apply heat-map styling to highlight where utilization and market share are concentrated
* Make it simple to spot outliers

***

## Data Detail

Once you have narrowed your analysis using the Market View filters, you can switch to the **Data** tab to explore the underlying detail behind the aggregated visuals.

Each row represents a distinct combination of:

* Billing NPI
* Payer
* Billing Code
* Year

Additional contextual fields may also be included and can affect the granularity of a row, such as modifiers, place of service, and other attributes that further define the scope of the data.\
\
Full definitions can be found [here in our data dictionary](../data-dictionary/#claims-data).

***

## Exporting Data

Every chart on the Market View tab as well as the Data tab allows you to export the detailed dataset for offline analysis or integration into downstream workflows.

Key considerations:

* Exports are limited to **10,000 rows per export**
* Exported data respects all active filters and selections applied in the analysis

For larger datasets, refining filters before exporting is recommended to stay within row limits and maintain relevance.<br>

***

## Global Filters and Active Selections

At all times, the top of the screen displays the global filters and active selections applied to the analysis.

This ensures you can:

* See exactly what dimensions and values are currently selected
* Track your analytical path as you explore the data
* Interact directly with filters to refine or reset your view

These global filters persist as you move throughout the screen and continue drilling into more detailed views.
