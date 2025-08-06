---
description: >-
  Data dictionary for the Hospital Price Transparency data for medical and
  pharmaceutical services. Below are the fields available for Data Lake
  customers.
---

# Hospital Transparency

Additional metadata for the hospital address can be made available but is not part of our standard offering.

| Field Name                       | Description / Sample Values                                                                                                                                   | Original Source                                  |
| -------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------ |
| **ADDITIONAL\_PAYER\_NOTES**     | Free-text notes hospitals sometimes include for a payer or plan. _e.g., “Only applicable to self-pay patients seen in ER,” “BCBS rates exclude lab fees”_     | HOSPITAL MRF                                     |
| **BILLING\_CLASS**               | Claim classification. _“Institutional”, “Professional”_                                                                                                       | HOSPITAL MRF                                     |
| **BILLING\_CODE\_CATEGORY**      | Broad clinical grouping of the billing code. _“Imaging”, “Surgery–Outpatient”, “Lab & Pathology”_                                                             | ADDED BY PAYERSET                                |
| **CODE**                         | Billing code exactly as published. _“99213”, “0274”, “J9206”, “30145”_                                                                                        | HOSPITAL MRF                                     |
| **CODE\_TYPE**                   | Coding system. _“CPT”, “HCPCS”, “MS-DRG”, “APC”, “NDC”_                                                                                                       | HOSPITAL MRF                                     |
| **DESCRIPTION**                  | Description from file (often truncated/abbreviated). _“Office/outpatient visit est low-level”, “Knee arthroscopy w/ meniscus repair”_                         | HOSPITAL MRF                                     |
| **DISCOUNTED\_CASH**             | Hospital’s cash price offered to self-pay patients.                                                                                                           | HOSPITAL MRF                                     |
| **DRUG\_CATEGORY**               | Therapeutic class if row represents a drug. _“Antineoplastic Agents”, “Analgesics”_                                                                           | HOSPITAL MRF                                     |
| **DRUG\_TYPE**                   | Brand / generic / biosimilar flag. _“Brand”, “Generic”, “Biosimilar”_                                                                                         | HOSPITAL MRF                                     |
| **DRUG\_UNIT**                   | Unit of measure for drug price. _“mg”, “mL”, “tablet”_                                                                                                        | HOSPITAL MRF                                     |
| **GROSS**                        | Hospital’s standard (chargemaster) price.                                                                                                                     | HOSPITAL MRF                                     |
| **HOSPITAL**                     | Facility name from file header. _“Mayo Clinic Hospital – Rochester”_                                                                                          | HOSPITAL MRF                                     |
| **HOSPITAL\_ID**                 | Payerset stable hash / surrogate key for the facility.                                                                                                        | ADDED BY PAYERSET                                |
| **MAXIMUM**                      | Highest negotiated rate among all payers/plans for this code.                                                                                                 | HOSPITAL MRF                                     |
| **METHODOLOGY**                  | Hospital’s narrative on how standard charges were derived. _“Cost-to-charge ratio”, “Rate-setting committee approved”_                                        | HOSPITAL MRF                                     |
| **MINIMUM**                      | Lowest negotiated rate among all payers/plans for this code.                                                                                                  | HOSPITAL MRF                                     |
| **PAYER\_NAME**                  | Payer name as listed by hospital. _“UnitedHealthcare”, “BCBS MN”, “Medicare FFS”_                                                                             | HOSPITAL MRF                                     |
| **PLAN\_NAME**                   | Plan or network label from hospital. _“UMR PPO”, “Horizon Omnia Tier 1”_                                                                                      | HOSPITAL MRF                                     |
| **PS\_PAYER**                    | Payerset-standardized payer name (mapped across hospitals).                                                                                                   | ADDED BY PAYERSET                                |
| **PS\_PLAN**                     | Payerset-standardized plan/network name.                                                                                                                      | ADDED BY PAYERSET                                |
| **SETTING**                      | Care setting or place of service (hospital-reported). _“Inpatient”, “Outpatient”, “Emergency Dept”, “Ambulatory Surgery”_                                     | HOSPITAL MRF                                     |
| **STANDARD\_CHARGE\_ALGORITHM**  | Text describing how **STANDARD\_CHARGE\_PERCENTAGE** or **STANDARD\_CHARGE\_DOLLAR** was calculated. _“Gross × 25%”, “Average of top 3 commercial contracts”_ | HOSPITAL MRF                                     |
| **STANDARD\_CHARGE\_DOLLAR**     | A flat-dollar “standard charge” (CMS-defined).                                                                                                                | HOSPITAL MRF                                     |
| **STANDARD\_CHARGE\_PERCENTAGE** | Percent-based standard charge, if reported. _“150% of Medicare OPPS”_                                                                                         | HOSPITAL MRF                                     |
| **SYSTEM**                       | Parent health-system / IDN, if hospital provided or we inferred it. _“HCA Healthcare”, “Sutter Health”_                                                       | ADDED BY PAYERSET (fallback to MRF when present) |
