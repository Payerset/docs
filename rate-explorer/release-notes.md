---
description: >-
  Below you will find release notes for improvements and fixes for the Payerset
  Price Intelligence platform.
---

# Release Notes

## December 2024

**View Negotiated Rates Over Time**

Users can now access historical negotiated rates between payers and providers, making it possible to compare pricing trends over time. With historical rate comparisons, you can now analyze trends in negotiated rates over time, assess payer-provider contract changes, compare pricing fluctuations across different periods, and identify patterns in cost shifts across various services and regions.\
\
This can be found on both Rate Comparisons as well as Fee Schedules.

\
**Access Standard Hospital Charges (Beta)**

The Standard Charges tool (currently in beta as we continue to validate the dataset) enables users to view the published prices for hospital services across various hospital systems and locations. With the Standard  Charges feature, you can now compare published prices for services across different hospital systems and locations, analyze cost variations by service type, and assess how pricing differs between facilities within the same region or network.

\
**Improved Navigation**

The updated user interface offers a more streamlined way to view and analyze payer and provider data. It simplifies the process of accessing saved analyses and provides easier navigation to our documentation.

## August 2024

#### **New Features**

**Fee Schedule Generator**

* Added a new mode that allows users to select a single NPI and generate an analysis with all billing codes and only modifiers 26, TC, and none.

**Place of Service Code Mapping**

* Implemented mapping of CMS place of service codes to their descriptive names in price transparency data. Users can now filter by "Place of Service" in global filters and add it to both pivot and detailed views to better understand where rates apply, such as inpatient versus outpatient facilities.

**Filter Grid**

* Introduced a new filter grid interface to help users quickly understand and filter data in their analyses. This feature allows users to see relationships across columns more intuitively, enhancing the overall data analysis experience and enabling much faster filtering to relevant data.

**New Field: FACILITY\_FLAG**

* Added a new "Facility Flag" field to differentiate between Facility and Non-Facility settings. This field is now available in the straight table, pivot table, and filters, providing users with more precise data categorization.

#### **Bug Fixes**

* **Copy to New Analysis**: Corrected functionality to ensure copying an analysis maintains the correct type.
* **Pop-Up Tooltip Issue**: Fixed an issue where tooltips failed to appear for columns with numerous plan entries.

## July 2024

**Normalize TIN Values**

* **Data Improvement**: TIN Values have been normalized with dashes removed for EINs, enabling simpler sorting.

**Add EIN and County to Provider Selection**

* **New Feature**: County has been mapped to each NPI and is now available for search when selecting providers. Additionally, County will be included in result sets to enable users to aggregate data at the county level.
* **New Feature**: EINs have been linked to each NPI using the most common matches across 75 payers and is now available for search when selecting providers.

**Clear All Filters Button**

* **New Feature**: Added a "Clear All" button in the filter bar. This allows users to clear all applied filters with a single click, simplifying the process of resetting filters.

**Select All Columns in the Data Tab (Detail Table View)**

* **New Feature**: Users can now add all columns to the straight table in the Data tab with a single click. This functionality streamlines the process of viewing comprehensive data without manually selecting each column.

**Limit Increases for Excel Export**

* **Enhancement**: Increased the export limit for both straight tables and pivot tables to 500,000 rows. Users can now export larger datasets to Excel, accommodating more extensive data analysis and reporting needs.

## June 2024

**Edit Analysis Throwing Error**

* **Bug Fix**: Resolved a consistent error occurring during analysis edits. Users should no longer encounter crashes or errors when editing their analyses.

**Screen "Run Off"**

* **Bug Fix**: Fixed an issue where global filters and graphs were cut off on the right edge when viewed on a 15-inch laptop screen at 100% zoom in Chrome. This fix ensures all content is fully visible without requiring zoom adjustments.

**Expiration Date as a Global Filter**

* **New Feature**: Added the ability to filter by expiration date in the global filters. This enhancement allows users to easily exclude expired rates from their pivot tables during data analysis.

**Tool Tip - Hover and Pop Up Text on Provider Name in Selection List**

* **Improvement**: Implemented tooltips that display the full provider name when hovering over truncated names in the provider selection list. This enhancement improves usability by making it easier to identify providers without needing to click on them.

**Showing the Full Column Names in Data Tab**

* **Improvement**: Users can now hover over column headers in the Data tab to view the full names or adjust the column width manually. This update addresses visibility issues for long column names, ensuring that all information is accessible.

## May 2024

**Revenue Code Addition**

* Add support for Revenue Code selection in addition to CPT/HCPCS/DRG, including categories and subcategories.&#x20;

**Improved Pivot Dimension Deselection**

* Made it more intuitive to deselect pivot dimensions to enhance user experience.

**Global Filter for Expiration Date**

* Added the ability to filter by expiration date in global filters, providing better control over data visibility.

**Authentication Bug Fixed**

* Fixed an issue where references to the old authentication link prevented analysis creation and testing.

**Plan Filtering Options Added**

* Implemented plan filtering options to allow for more precise data segmentation.

**Pivot Table Freezing Issue Resolved**

* Resolved an issue where adding multiple dimensions and measures would freeze the pivot table.

**New Fields for Reporting Plan Types**

* Added new fields for reporting plan type and plan name in both the explore tab and global filters.

**Data Parsing Improvements**

* Added new payers: BCBS\_MONTANA, BCBS\_VERMONT, CONNECTICARE, DEAN, HMSA, HORIZON\_BCBS\_NJ, MEDICA, WELLMARK.
* Added support for fixing incorrect service code formatting.
* Updated parsing process to handle United Healthcare's new API.
* Added support to process nested zip files in EC2 parser.
* Increased request timeout in EC2 parsers.
* Improved error handling, file extension validation, and task submission in EC2 parsers.
* Refactored large file skipping, downloading logic, and removed unused code in EC2 parser scripts.
* Optimized file processing, encoding conversion, and error messages in EC2 parsers.

## April 2024

**Analysis Editing Error Fixed**

* Fixed an issue where editing an analysis could result in an error, improving reliability.

**Payer List Filter Correction**

* Corrected a bug where all payers appeared in the list regardless of row-level security settings.

**Pivot Table Stability Enhancements**

* Resolved an issue causing the pivot table to freeze when adding all measures and dimensions, ensuring smoother performance.

**Pivot Table Apostrophe Bug Fix**

* Fixed a bug where the pivot table would freeze when filtering by organization names containing apostrophes.

**State-Based Data Access Control**

* Implemented a feature to limit data access by state for specific users, enhancing data security and compliance.

**Custom Domain for Authentication**

* Configured the login mechanism to use a custom domain for Authentication Service to avoid issues with corporate firewalls.

**Plan Column Addition Error Resolved**

* Addressed an error that occurred when adding the Plan column in the QA environment, preventing application freezes.

**Password Recovery and Magic Links Fixed**

* Ensured that the password recovery and magic link functionality work correctly with the new domain.

**Filter Counts Feature Implemented**

* Added the Filter Counts feature to the analysis view, providing users with more precise filtering options.

**Intermittent Analysis Update Error Fixed**

* Fixed an issue where updating an analysis would sometimes result in an error.

**Hover Box Display Issue Resolved**

* Resolved an intermittent issue with hover boxes not displaying correctly.

**Min and Max Rate Calculations Added**

* Added Min and Max calculations for rate $ and rate % in the Explore View for enhanced analysis.

**Visual Bug Fix for Single Code Selection**

* Fixed a visual bug where a blur appeared when selecting only one billing code.

**Min and Max Calculations Stability**

* Addressed an issue where Min and Max calculations caused the pivot table to freeze.

## March 2024

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

## February 2024

* **Viva Place of Service Code Delimitation**
  * Corrected place of service codes for Viva, ensuring proper format and accuracy.
* **Enhanced BCBS of FL Analysis Stability**
  * Addressed freezing issues when pulling BCBS of FL, improving reliability and performance.
* **Browser Memory Optimization**
  * Implemented fixes for "Aw Snap!" browser memory errors, enhancing stability during intensive analyses.

## January 2024

* **Payer Display Name Standardization**
  * Standardized payer display names on summary page for consistency across analyses.
* **NPI Filter Enhancement**
  * Improved NPI selection filters for better usability and clarity in the "Selected" view.
* **Analysis Timeout Resolution**
  * Addressed CORS error and gateway timeout issues during provider matching, improving reliability.
* **Analysis Edit Update Delay**
  * Fixed delays in updating analyses after edits, ensuring immediate reflection of changes.

## August 2023

* **Dashboard Initialization Improvement**
  * Fixed an issue where the dashboard wouldn't populate without a browser refresh after initial login, enhancing user experience.
* **Viva Payer Addition Request**
  * Addressed a customer request to add Viva data to their payer list, improving data comprehensiveness.
* **Service Code Label Update**
  * Updated "service codes" field name to "Place of Service Codes" for clarity and accuracy
