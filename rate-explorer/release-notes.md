---
description: >-
  Below you will find release notes for improvements and fixes for Rate
  Explorer.
---

# Release Notes

### March 2024

* **Place of Service Codes Parsing Improvement**
  * Adjusted parsing logic to correctly differentiate between single and compound digits for Viva, improving accuracy in analysis.
* **BCBS of FL Analysis Freezing Issue**
  * Resolved a problem where selecting BCBS of FL along with BCBS of Arkansas would freeze the provider selection screen.
* **Memory Optimization for Chrome & Safari**
  * Fixed a browser out of memory issue encountered during the analysis, enhancing stability.
* **Duplicate Payer Listing Cleanup**
  * Addressed an issue with Highmark West Virginia appearing twice in the payer list.
* **Provider Selection Accuracy**
  * Improved select all functionality to accurately include NPIs without rates, enhancing user selection process.
* **NPI Selection Enhancements**
  * Corrected an issue causing duplicate NPIs to appear in selections, ensuring accurate data representation.
* **Service Selection Reset**
  * Fixed a bug where service selection would reset unexpectedly in the QA environment, enhancing usability.
* **Service Addition Consistency**
  * Resolved intermittent issues with adding new services to analyses, ensuring changes are consistently applied.

### February 2024

* **Viva Place of Service Code Delimitation**
  * Corrected place of service codes for Viva, ensuring proper format and accuracy.
* **Enhanced BCBS of FL Analysis Stability**
  * Addressed freezing issues when pulling BCBS of FL, improving reliability and performance.
* **Browser Memory Optimization**
  * Implemented fixes for "Aw Snap!" browser memory errors, enhancing stability during intensive analyses.

### September 2023

* **Payer Display Name Standardization**
  * Standardized payer display names on summary page for consistency across analyses.
* **NPI Filter Enhancement**
  * Improved NPI selection filters for better usability and clarity in the "Selected" view.
* **Analysis Timeout Resolution**
  * Addressed CORS error and gateway timeout issues during provider matching, improving reliability.
* **Analysis Edit Update Delay**
  * Fixed delays in updating analyses after edits, ensuring immediate reflection of changes.

### August 2023

* **Dashboard Initialization Improvement**
  * Fixed an issue where the dashboard wouldn't populate without a browser refresh after initial login, enhancing user experience.
* **Viva Payer Addition Request**
  * Addressed a customer request to add Viva data to their payer list, improving data comprehensiveness.
* **Service Code Label Update**
  * Updated "service codes" field name to "Place of Service Codes" for clarity and accuracy
