---
description: Fields available in our Rate Explorer Portal
---

# Data Dictionary

For pricing information, please see [https://payerset.com/pricing](https://payerset.com/pricing)

<table data-full-width="false"><thead><tr><th width="239.5">Field Name</th><th width="347">Description / Sample Values</th><th>Original Source</th><th data-hidden>Field Type</th></tr></thead><tbody><tr><td>Additional Information</td><td>Additional information relevant to rate</td><td>PAYER MRF</td><td>VARCHAR</td></tr><tr><td>Billing Class</td><td>Institutional or Professional</td><td>PAYER MRF</td><td>VARCHAR</td></tr><tr><td>Billing Code</td><td>Billing code as-is in MRF</td><td>PAYER MRF</td><td>VARCHAR</td></tr><tr><td>Billing Code Modifier</td><td>Modifiers for different rates (22-99, P1)</td><td>PAYER MRF</td><td>VARCHAR</td></tr><tr><td>Billing Code Name</td><td>Billing code name as-is in MRF</td><td>PAYER MRF</td><td>VARCHAR</td></tr><tr><td>Billing Code Type</td><td>HCPCS, CPT, MS-DRG, etc</td><td>PAYER MRF</td><td>VARCHAR</td></tr><tr><td>Billing Code Type Version</td><td>Version of the billing code (2023, 2024)</td><td>PAYER MRF</td><td>VARCHAR</td></tr><tr><td>Expiration Date</td><td>Expiration date of the rate (12/31/2024)</td><td>PAYER MRF</td><td>VARCHAR</td></tr><tr><td>Negotiated Rate</td><td>Negotiated Rate value</td><td>PAYER MRF</td><td>FLOAT</td></tr><tr><td>Negotiated Type</td><td>Fee schedule, negotiated, percentage, per diem</td><td>PAYER MRF</td><td>VARCHAR</td></tr><tr><td>Negotiation Arrangement</td><td>Fee for service (ffs) or Bundle</td><td>PAYER MRF</td><td>VARCHAR</td></tr><tr><td>NPI</td><td>Provider ID</td><td>PAYER MRF</td><td>VARCHAR</td></tr><tr><td>Place of Service Codes</td><td>Location of service (11, 21, 22, etc)</td><td>PAYER MRF</td><td>VARCHAR</td></tr><tr><td>TIN Type</td><td>EIN or NPI</td><td>PAYER MRF</td><td>VARCHAR</td></tr><tr><td>TIN Value</td><td>Value of associated EIN or NPI</td><td>PAYER MRF</td><td>VARCHAR</td></tr><tr><td>City</td><td>City Name from NPPES</td><td>NPPES</td><td>VARCHAR</td></tr><tr><td>State</td><td>State Name from NPPES</td><td>NPPES</td><td>VARCHAR</td></tr><tr><td>Organization Name</td><td>Organization Name from NPPES</td><td>NPPES</td><td>VARCHAR</td></tr><tr><td>Organization Friendly Name</td><td>Org Name + DBA Name if present</td><td>ADDED BY PAYERSET</td><td>VARCHAR</td></tr><tr><td>Taxonomy Grouping</td><td>High-level taxonomy grouping (Allopathic, Hospitals, Hospital Units, etc)</td><td>NUCC</td><td>VARCHAR</td></tr><tr><td>Taxonomy Classification</td><td>More granular taxonomy classification (Clinic/Center, Abmulance, Nursing Care, etc)</td><td>NUCC</td><td>VARCHAR</td></tr><tr><td>Taxonomy Specialization</td><td>Most granular taxonomy (Adult Mental Health, Oncology, Orthopedic Surgery, etc...)</td><td>NUCC</td><td>VARCHAR</td></tr><tr><td>Payer</td><td>Payer column added for convenience</td><td>ADDED BY PAYERSET</td><td>VARCHAR</td></tr><tr><td>Payerset Billing Code Name</td><td>Payerset standardized name of 21k+ billing codes</td><td>ADDED BY PAYERSET</td><td>VARCHAR</td></tr><tr><td>Payerset Billing Code Category</td><td>Payerset categorization of 21k+ billing codes</td><td>ADDED BY PAYERSET</td><td>VARCHAR</td></tr><tr><td>Payerset Billing Code Subcategory</td><td>Payerset subcategorization of 21k+ billing codes</td><td>ADDED BY PAYERSET</td><td>VARCHAR</td></tr><tr><td>Payerset Billing Code Type</td><td>Payerset categorization of billing code types</td><td>ADDED BY PAYERSET</td><td>VARCHAR</td></tr><tr><td>Plan Type</td><td>Payerset-created Plan ID that links back to original plan via PLAN_MAP table</td><td>ADDED BY PAYERSET</td><td>VARCHAR</td></tr></tbody></table>

\