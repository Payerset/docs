---
description: Fields available in Rate Explorer
---

# Data Dictionary

## Comparison Analysis & Fee Schedule Generator

<table data-full-width="false"><thead><tr><th width="239.5">Field Name</th><th width="347">Description / Sample Values</th><th>Original Source</th><th data-hidden>Field Type</th></tr></thead><tbody><tr><td>Additional Information</td><td>Additional information relevant to rate</td><td>PAYER MRF</td><td>VARCHAR</td></tr><tr><td>Billing Class</td><td>Institutional or Professional</td><td>PAYER MRF</td><td>VARCHAR</td></tr><tr><td>Billing Code</td><td>Billing code as-is in MRF</td><td>PAYER MRF</td><td>VARCHAR</td></tr><tr><td>Billing Code Modifier</td><td>Modifiers for different rates (22-99, P1)</td><td>PAYER MRF</td><td>VARCHAR</td></tr><tr><td>Billing Code Name</td><td>Billing code name as-is in MRF</td><td>PAYER MRF</td><td>VARCHAR</td></tr><tr><td>Billing Code Type</td><td>HCPCS, CPT, MS-DRG, etc</td><td>PAYER MRF</td><td>VARCHAR</td></tr><tr><td>Billing Code Type Version</td><td>Version of the billing code (2023, 2024)</td><td>PAYER MRF</td><td>VARCHAR</td></tr><tr><td>Expiration Date</td><td>Expiration date of the rate (12/31/2024)</td><td>PAYER MRF</td><td>VARCHAR</td></tr><tr><td>Negotiated Rate</td><td>Negotiated Rate value</td><td>PAYER MRF</td><td>FLOAT</td></tr><tr><td>Negotiated Type</td><td>Fee schedule, negotiated, percentage, per diem</td><td>PAYER MRF</td><td>VARCHAR</td></tr><tr><td>Negotiation Arrangement</td><td>Fee for service (ffs) or Bundle</td><td>PAYER MRF</td><td>VARCHAR</td></tr><tr><td>NPI</td><td>Provider ID</td><td>PAYER MRF</td><td>VARCHAR</td></tr><tr><td>Place of Service Codes</td><td>Location of service (11, 21, 22, etc)</td><td>PAYER MRF</td><td>VARCHAR</td></tr><tr><td>TIN Type</td><td>EIN or NPI</td><td>PAYER MRF</td><td>VARCHAR</td></tr><tr><td>TIN Value</td><td>Value of associated EIN or NPI</td><td>PAYER MRF</td><td>VARCHAR</td></tr><tr><td>City</td><td>City Name from NPPES</td><td>NPPES</td><td>VARCHAR</td></tr><tr><td>State</td><td>State Name from NPPES</td><td>NPPES</td><td>VARCHAR</td></tr><tr><td>Organization Name</td><td>Organization Name from NPPES</td><td>NPPES</td><td>VARCHAR</td></tr><tr><td>Organization Friendly Name</td><td>Org Name + DBA Name if present</td><td>ADDED BY PAYERSET</td><td>VARCHAR</td></tr><tr><td>Taxonomy Grouping</td><td>High-level taxonomy grouping (Allopathic, Hospitals, Hospital Units, etc)</td><td>NUCC</td><td>VARCHAR</td></tr><tr><td>Taxonomy Classification</td><td>More granular taxonomy classification (Clinic/Center, Abmulance, Nursing Care, etc)</td><td>NUCC</td><td>VARCHAR</td></tr><tr><td>Taxonomy Specialization</td><td>Most granular taxonomy (Adult Mental Health, Oncology, Orthopedic Surgery, etc...)</td><td>NUCC</td><td>VARCHAR</td></tr><tr><td>Payer</td><td>Payer column added for convenience</td><td>ADDED BY PAYERSET</td><td>VARCHAR</td></tr><tr><td>Payerset Billing Code Name</td><td>Payerset standardized name of 21k+ billing codes</td><td>ADDED BY PAYERSET</td><td>VARCHAR</td></tr><tr><td>Payerset Billing Code Category</td><td>Payerset categorization of 21k+ billing codes</td><td>ADDED BY PAYERSET</td><td>VARCHAR</td></tr><tr><td>Payerset Billing Code Subcategory</td><td>Payerset subcategorization of 21k+ billing codes</td><td>ADDED BY PAYERSET</td><td>VARCHAR</td></tr><tr><td>Payerset Billing Code Type</td><td>Payerset categorization of billing code types</td><td>ADDED BY PAYERSET</td><td>VARCHAR</td></tr><tr><td>Plan ID</td><td>Payerset-created Plan ID that links back to original plan via PLAN_MAP table</td><td>ADDED BY PAYERSET</td><td>VARCHAR</td></tr><tr><td>Plan Name</td><td>Friendly name of the plan, created by Payerset</td><td>ADDED BY PAYERSET</td><td></td></tr><tr><td>Plan Type</td><td>EPO, PPO, HMO, POS, or Other Type of the plan, created by Payerset</td><td>ADDED BY PAYERSET</td><td></td></tr></tbody></table>

## Claims Data

Please note that the Claims data is only available with the upgraded Payerset Pricing Intelligence Solution. For more information, please contact info@payerset.com&#x20;

| Field Name                              | Friendly Name                                          | Description / Sample Value                                                                     |
| --------------------------------------- | ------------------------------------------------------ | ---------------------------------------------------------------------------------------------- |
| `NPI`                                   | NPI                                                    | Unique 10-digit National Provider Identifier. e.g. `1043270564`                                |
| `SETTING`                               | Setting                                                | Derived from Type of Bill on the claim data, this is the Inpatient or Outpatient setting.      |
| `PLACE_OF_SERVICE_CODE`                 | Service Code                                           | Two-digit CMS Place-of-Service code. e.g. `11`                                                 |
| `PLACE_OF_SERVICE_NAME`                 | Place of Service                                       | Human-readable name for the POS code. e.g. `Office`                                            |
| `CHANNEL`                               | Channel                                                | High-level payer channel grouping. e.g. `Commercial`                                           |
| `SUBCHANNEL`                            | Subchannel                                             | Sub-segment within the channel. e.g. `Fully Insured`                                           |
| `PAYER`                                 | Payer                                                  | Name of the payer organization. e.g. `UnitedHealthcare`                                        |
| `BILLING_CODE`                          | Billing Code                                           | Procedure code billed (CPT / HCPCS / RC). e.g. `99213`                                         |
| `BILLING_CODE_TYPE`                     | Billing Code Type                                      | Classification of the billing code. e.g. `CPT`                                                 |
| `BILLING_CODE_DESCRIPTION`              | Billing Code Description                               | Short description of the procedure. e.g. `Office/outpatient visit, established`                |
| `MODIFIER`                              | Billing Code Modifier                                  | CPT/HCPCS modifier (if any). e.g. `25`                                                         |
| `MODIFIER_DESCRIPTION`                  | Billing Code Modifier Description                      | Meaning of the modifier. e.g. `Significant, separately identifiable E/M service`               |
| `OPEN_CLAIMS_COUNT`                     | Open Claims Count                                      | Number of open (unadjudicated) claims. e.g. `42`                                               |
| `AVG_CLAIM_AMOUNT`                      | Avg Claim Amount                                       | Average submitted claim charge. e.g. `$123.45`                                                 |
| `MIN_REMIT_AMOUNT`                      | Min Remit Amount                                       | Minimum remitted (paid) amount. e.g. `$10.00`                                                  |
| `MAX_REMIT_AMOUNT`                      | Max Remit Amount                                       | Maximum remitted amount. e.g. `$1,200.00`                                                      |
| `AVG_REMIT_AMOUNT`                      | Avg Remit Amount                                       | Average remitted amount. e.g. `$95.67`                                                         |
| `MEDIAN_REMIT_AMOUNT`                   | Median Remit Amount                                    | Median remitted amount. e.g. `$80.00`                                                          |
| `REMIT_COUNT`                           | Remit Count                                            | Count of paid remits for the record set. e.g. `37`                                             |
| `OPEN_CLAIMS_VOLUME_NPI_CODE_PAYER_PCT` | # Open Claims Count Ranking (NPI, Billing Code, Payer) | Percentile rank of open-claim volume within the (NPI, Billing Code, Payer) cohort. e.g. `93.2` |
| `REMITS_VOLUME_NPI_CODE_PAYER_PCT`      | # Remit Count Ranking (NPI, Billing Code, Payer)       | Percentile rank of remit count within the same cohort. e.g. `88.7`                             |

## Medicare Data

### Inpatient Data

| Field Name                       | Friendly Name              | Description / Sample Value                                                        |
| -------------------------------- | -------------------------- | --------------------------------------------------------------------------------- |
| NPI                              | Provider NPI               | Unique 10-digit National Provider Identifier. e.g. `1043270564` or `No NPI Found` |
| carrier\_number                  | Carrier Number             | Payer’s internal carrier code. e.g. `110107`                                      |
| Rndrng\_Prvdr\_Org\_Name         | Provider Organization Name | Name of the rendering provider’s organization. e.g. `Atrium Health Navicent`      |
| Rndrng\_Prvdr\_City              | Provider City              | City where the provider is located. e.g. `Macon`                                  |
| Rndrng\_Prvdr\_St                | Provider Street Address    | Street address of the provider. e.g. `777 Hemlock Street`                         |
| Rndrng\_Prvdr\_State\_FIPS       | State FIPS Code            | U.S. Census state FIPS code. e.g. `13`                                            |
| Rndrng\_Prvdr\_Zip5              | Provider ZIP Code          | 5-digit postal ZIP code. e.g. `31201`                                             |
| Rndrng\_Prvdr\_State\_Abbrvtn    | State Abbreviation         | USPS state abbreviation. e.g. `GA`                                                |
| cbsa                             | CBSA Code                  | Core-Based Statistical Area code. e.g. `12060`                                    |
| drg\_code                        | DRG Code                   | Diagnosis-Related Group code. e.g. `470`                                          |
| fee\_schedule\_dollar\_amount    | Fee Schedule Amount        | CMS fee schedule amount in dollars. e.g. `125.00`                                 |
| Total\_Discharges                | Total Discharges           | Total number of discharges reported. e.g. `250` (may be blank/null)               |
| Avg\_Submitted\_Covered\_Charges | Avg Covered Charges        | Average submitted covered charges. e.g. `3250.50`                                 |
| Avg\_Total\_Payment\_Amount      | Avg Total Payment          | Average total payment amount. e.g. `2900.75`                                      |
| Avg\_Medicare\_Payment\_Amount   | Avg Medicare Payment       | Average amount paid by Medicare. e.g. `1800.25`                                   |
| Avg\_Medicare\_Payment\_Percent  | Medicare Payment %         | Medicare payment as percentage of covered charges. e.g. `55.33`                   |
| latitude                         | Latitude                   | Geographic latitude of the provider. e.g. `32.8095`                               |
| longitude                        | Longitude                  | Geographic longitude of the provider. e.g. `-83.6168`                             |

### Outpatient Data

| Field Name                           | Friendly Name                                   | Description / Sample Value                                                 |
| ------------------------------------ | ----------------------------------------------- | -------------------------------------------------------------------------- |
| HCPCS Code                           | HCPCS Procedure Code                            | Healthcare Common Procedure Coding System code. e.g. `0275T`               |
| Modifier                             | HCPCS Modifier                                  | Optional two-character modifier. e.g. `""` (empty if none)                 |
| Short Description                    | Service Short Description                       | Brief description of the procedure. e.g. `Perq lamot/lam lumbar`           |
| Mac Locality                         | MAC Locality Code                               | Medicare Administrative Contractor locality code. e.g. `111205`            |
| Locality County                      | County                                          | County that corresponds with Mac Locality                                  |
| Locality State                       | State                                           | State that corresponds with Mac Locality                                   |
| Non-Facility Price                   | Non-Facility Price                              | Allowed charge in a non-facility setting. e.g. `"$0.00"`                   |
| Facility Price                       | Facility Price                                  | Allowed charge in a facility setting. e.g. `"$0.00"`                       |
| Non-Facility Limiting Charge         | Non-Facility Limiting Charge                    | Payment limit for non-facility. e.g. `"$0.00"`                             |
| Facility Limiting Charge             | Facility Limiting Charge                        | Payment limit for facility. e.g. `"$0.00"`                                 |
| GPCI Work                            | Work GPCI Factor                                | Geographic practice cost index for work. e.g. `1.088`                      |
| GPCI PE                              | Practice Exp. GPCI Factor                       | Geographic practice cost index for practice expense. e.g. `1.419`          |
| GPCI MP                              | Malpractice GPCI Factor                         | Geographic malpractice cost index. e.g. `0.445`                            |
| Proc Stat                            | Procedure Status                                | Status indicator (e.g. R=revised). e.g. `"R"`                              |
| Work RVU                             | Work RVU                                        | Relative value unit for physician work. e.g. `0.00`                        |
| NA Flag for Trans Non-FAC PE RVU     | Flag: Transitional Non-Facility PE RVU Missing  | `"NA"` if no transitional practice-expense RVU available                   |
| Transitioned Non-FAC PE RVU          | Transitional Non-Facility PE RVU                | Transitional practice-expense RVU, non-facility. e.g. `0.00`               |
| NA Flag for Fully IMP Non-FAC PE RVU | Flag: Fully Implemented Non-FAC PE RVU Missing  | `"NA"` if no fully implemented practice-expense RVU available              |
| Fully Implemented Non-FAC PE RVU     | Fully Impl. Non-Facility PE RVU                 | Final practice-expense RVU, non-facility. e.g. `0.00`                      |
| NA Flag for Trans Facility PE RVU    | Flag: Transitional Facility PE RVU Missing      | `"NA"` if no transitional practice-expense RVU for facility available      |
| Transitioned Facility PE RVU         | Transitional Facility PE RVU                    | Transitional practice-expense RVU, facility. e.g. `0.00`                   |
| NA Flag for Fully IMP FAC PE RVU     | Flag: Fully Implemented Facility PE RVU Missing | `"NA"` if no fully implemented practice-expense RVU for facility available |
| Fully Implemented Facility PE RVU    | Fully Impl. Facility PE RVU                     | Final practice-expense RVU, facility. e.g. `0.00`                          |
| MP RVU                               | Malpractice RVU                                 | Relative value unit for malpractice. e.g. `0.00`                           |
| Transitioned Non-FAC Total           | Transitional Non-Facility Total RVU             | Sum of work + PE + MP RVUs (transitional)(non-facility). e.g. `0.00`       |
| Transitioned Facility Total          | Transitional Facility Total RVU                 | Sum of work + PE + MP RVUs (transitional)(facility). e.g. `0.00`           |
| Fully Implemented Non-Fac Total      | Fully Impl. Non-Facility Total RVU              | Sum of RVUs (work+PE+MP) final, non-facility. e.g. `0.00`                  |
| Fully Implemented Facility Total     | Fully Impl. Facility Total RVU                  | Sum of RVUs (work+PE+MP) final, facility. e.g. `0.00`                      |
| PCTC                                 | Multiple-Procedure Indicator                    | Indicator for multiple-procedure payment reduction. e.g. `"YYY"`           |
| Global                               | Global Surgical Indicator                       | Global surgery period indicator (0=no global). e.g. `0`                    |
| Pre Op                               | Pre-Operative RVU                               | RVU for pre-operative period. e.g. `0.00`                                  |
| Intra Op                             | Intra-Operative RVU                             | RVU for intra-operative period. e.g. `0.00`                                |
| Post Op                              | Post-Operative RVU                              | RVU for post-operative period. e.g. `0.00`                                 |
| Mult Surg                            | Multiple Surgery RVU                            | RVU adjustment for multiple surgeries. e.g. `0.00`                         |
| Bilt Surg                            | Bilateral Surgery RVU                           | RVU adjustment for bilateral procedures. e.g. `0.00`                       |
| Asst Surg                            | Assistant Surgeon RVU                           | RVU for assistant surgeon. e.g. `0.00`                                     |
| Co Surg                              | Co-Surgeon RVU                                  | RVU for co-surgeon. e.g. `0.00`                                            |
| Team Surg                            | Team Surgery RVU                                | RVU for team surgery. e.g. `0.00`                                          |
| Phys Supv                            | Physician Supervision RVU                       | RVU for physician supervision. e.g. `0.00`                                 |
| Endobase                             | Endoscopic Base RVU Indicator                   | Indicator if code is endoscopic base. e.g. `""`                            |
| Conv Fact                            | Conversion Factor                               | Dollar-to-RVU conversion factor. e.g. `32.3465`                            |
| Not Used for Medicare                | Excluded from Medicare                          | Flag if code is not payable by Medicare. e.g. `""`                         |
| Diag Imaging Family Ind              | Diagnostic Imaging Family Indicator             | Family group code. e.g. `99`                                               |
| Opps Non-Facility Payment Amount     | OPPS Non-Facility Payment                       | Payment amount under OPPS non-facility. e.g. `"NA"`                        |
| Opps Facility Payment Amount         | OPPS Facility Payment                           | Payment amount under OPPS facility. e.g. `"NA"`                            |
| Non-Fac PE Used For Opps PMT AMT     | Non-Facility PE Weight for OPPS Payment         | Practice-expense index used in OPPS non-facility calculation. e.g. `0.0`   |
| Facility PE Used For Opps PMT AMT    | Facility PE Weight for OPPS Payment             | Practice-expense index used in OPPS facility calculation. e.g. `0.0`       |
| Malpractice Used For Opps PMT AMT    | Malpractice PE Weight for OPPS Payment          | Malpractice index used in OPPS payment calculation. e.g. `0.0`             |

## Utilities

### UnitedHealthcare CSTM-ALL

| Field Name                     | Friendly Name                | Description / Sample Value                                                                      |
| ------------------------------ | ---------------------------- | ----------------------------------------------------------------------------------------------- |
| NPI                            | Provider NPI                 | National Provider Identifier. e.g. `1417993361`                                                 |
| TIN\_TYPE                      | Tax ID Type                  | Type of tax identifier (e.g., `ein`, `ssn`).                                                    |
| TIN\_VALUE                     | Tax ID Value                 | Taxpayer Identification Number. e.g. `390286215`                                                |
| BILLING\_CODE                  | Payer Billing Code           | Code used by payer for billing. e.g. `MISC`, `THR1`                                             |
| BILLING\_CLASS                 | Billing Class                | Class of billing (e.g., `institutional`, `professional`).                                       |
| EXPIRATION\_DATE               | Agreement Expiration Date    | Date the agreement expires. e.g. `1999-12-31` (from `12/31/99`)                                 |
| NEGOTIATED\_RATE               | Negotiated Rate              | Agreed-upon rate. e.g. `75`, `200`, `28`, `210`, `90`                                           |
| NEGOTIATED\_TYPE               | Rate Type                    | Type of negotiated rate. e.g. `percentage`, `per diem`                                          |
| SERVICE\_CODES                 | Applicable Service Codes     | Comma-separated list of service codes. (empty if none)                                          |
| FACILITY\_FLAG                 | Facility Flag                | Indicator if facility setting. e.g. `Y`/`N` or blank                                            |
| PLACE\_OF\_SERVICE             | Place of Service             | Payer’s place-of-service code/name. e.g. `No Service Code`                                      |
| NEGOTIATION\_ARRANGEMENT       | Negotiation Arrangement      | Arrangement type. e.g. `ffs`                                                                    |
| ADDITIONAL\_INFORMATION        | Additional Information       | Extra qualifiers. e.g. `age[18-64]`                                                             |
| BILLING\_CODE\_MODIFIER        | Billing Code Modifier        | Modifier for billing code. e.g. (empty)                                                         |
| BILLING\_CODE\_TYPE            | Billing Code Type            | Code system used. e.g. `CSTM-ALL`                                                               |
| BILLING\_CODE\_TYPE\_VERSION   | Billing Code Version         | Version/year of the code system. e.g. `2025`                                                    |
| BILLING\_CODE\_NAME            | Billing Code Description     | Human-readable description of billing code. e.g. `OUTPATIENT MISCELLANEOUS (DEFAULT)`           |
| PAYER                          | Payer Name                   | Name of the insurance payer. e.g. `UNITED_HEALTHCARE`                                           |
| NPPES\_PRIMARY\_TAXONOMY\_CODE | Primary Taxonomy Code        | NPPES taxonomy code. e.g. `101Y00000X`                                                          |
| NPPES\_STATE                   | Provider State               | USPS state abbreviation. e.g. `WI`, `AZ`, `CA`                                                  |
| NPPES\_CITY                    | Provider City                | City of provider. e.g. `FORT ATKINSON`, `PHOENIX`                                               |
| NPPES\_COUNTY                  | Provider County              | County of provider. e.g. `JEFFERSON`, `MARICOPA`                                                |
| NPPES\_ORGFRIENDLYNAME         | Provider Organization Name   | Official organization name. e.g. `FORT HEALTHCARE INC - FORT ATKINSON MEMORIAL HEALTH SERVICES` |
| NUCC\_TAXONOMY\_GROUPING       | NUCC Taxonomy Grouping       | Broad taxonomy grouping. e.g. `Behavioral Health & Social Service Providers`                    |
| NUCC\_TAXONOMY\_CLASSIFICATION | NUCC Taxonomy Classification | Taxonomy classification. e.g. `Counselor`                                                       |
| NUCC\_TAXONOMY\_SPECIALIZATION | NUCC Taxonomy Specialization | Taxonomy specialization. e.g. `Addiction (Substance Use Disorder)` or `None`                    |
| NUCC\_TAXONOMY\_DISPLAYNAME    | NUCC Display Name            | Display name for taxonomy. e.g. `Counselor`                                                     |
| PAYERSET\_BILLING\_CODE\_NAME  | Payerset Billing Code Name   | Internal billing code name in Payerset. e.g. `OUTPATIENT MISCELLANEOUS (DEFAULT)`               |
| PAYERSET\_BILLING\_CODE\_TYPE  | Payerset Billing Code Type   | Internal billing code type in Payerset. e.g. `CSTM-ALL`                                         |
