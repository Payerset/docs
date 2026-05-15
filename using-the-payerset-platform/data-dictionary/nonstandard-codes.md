---
description: Data dictionary fields for nonstandard codes in Payerset.
---

# Nonstandard Codes

| Field Name                         | Friendly Name                | Description / Sample Value                                                                      |
| ---------------------------------- | ---------------------------- | ----------------------------------------------------------------------------------------------- |
| **NPI**                            | Provider NPI                 | National Provider Identifier. e.g. `1417993361`                                                 |
| **TIN\_TYPE**                      | Tax ID Type                  | Type of tax identifier (e.g., `ein`, `ssn`).                                                    |
| **TIN\_VALUE**                     | Tax ID Value                 | Taxpayer Identification Number. e.g. `390286215`                                                |
| **BILLING\_CODE**                  | Payer Billing Code           | Code used by payer for billing. e.g. `MISC`, `THR1`                                             |
| **BILLING\_CLASS**                 | Billing Class                | Class of billing (e.g., `institutional`, `professional`).                                       |
| **EXPIRATION\_DATE**               | Agreement Expiration Date    | Date the agreement expires. e.g. `1999-12-31` (from `12/31/99`)                                 |
| **NEGOTIATED\_RATE**               | Negotiated Rate              | Agreed-upon rate. e.g. `75`, `200`, `28`, `210`, `90`                                           |
| **NEGOTIATED\_TYPE**               | Rate Type                    | Type of negotiated rate. e.g. `percentage`, `per diem`                                          |
| **SERVICE\_CODES**                 | Applicable Service Codes     | Comma-separated list of service codes. (empty if none)                                          |
| **FACILITY\_FLAG**                 | Facility Flag                | Indicator if facility setting. e.g. `Y`/`N` or blank                                            |
| **PLACE\_OF\_SERVICE**             | Place of Service             | Payer’s place-of-service code/name. e.g. `No Service Code`                                      |
| **NEGOTIATION\_ARRANGEMENT**       | Negotiation Arrangement      | Arrangement type. e.g. `ffs`                                                                    |
| **ADDITIONAL\_INFORMATION**        | Additional Information       | Extra qualifiers. e.g. `age[18-64]`                                                             |
| **BILLING\_CODE\_MODIFIER**        | Billing Code Modifier        | Modifier for billing code. e.g. (empty)                                                         |
| **BILLING\_CODE\_TYPE**            | Billing Code Type            | Code system used. e.g. `CSTM-ALL`                                                               |
| **BILLING\_CODE\_TYPE\_VERSION**   | Billing Code Version         | Version/year of the code system. e.g. `2025`                                                    |
| **BILLING\_CODE\_NAME**            | Billing Code Description     | Human-readable description of billing code. e.g. `OUTPATIENT MISCELLANEOUS (DEFAULT)`           |
| **PAYER**                          | Payer Name                   | Name of the insurance payer. e.g. `UNITED_HEALTHCARE`                                           |
| **NPPES\_PRIMARY\_TAXONOMY\_CODE** | Primary Taxonomy Code        | NPPES taxonomy code. e.g. `101Y00000X`                                                          |
| **NPPES\_STATE**                   | Provider State               | USPS state abbreviation. e.g. `WI`, `AZ`, `CA`                                                  |
| **NPPES\_CITY**                    | Provider City                | City of provider. e.g. `FORT ATKINSON`, `PHOENIX`                                               |
| **NPPES\_COUNTY**                  | Provider County              | County of provider. e.g. `JEFFERSON`, `MARICOPA`                                                |
| **NPPES\_ORGFRIENDLYNAME**         | Provider Organization Name   | Official organization name. e.g. `FORT HEALTHCARE INC - FORT ATKINSON MEMORIAL HEALTH SERVICES` |
| **NUCC\_TAXONOMY\_GROUPING**       | NUCC Taxonomy Grouping       | Broad taxonomy grouping. e.g. `Behavioral Health & Social Service Providers`                    |
| **NUCC\_TAXONOMY\_CLASSIFICATION** | NUCC Taxonomy Classification | Taxonomy classification. e.g. `Counselor`                                                       |
| **NUCC\_TAXONOMY\_SPECIALIZATION** | NUCC Taxonomy Specialization | Taxonomy specialization. e.g. `Addiction (Substance Use Disorder)` or `None`                    |
| **NUCC\_TAXONOMY\_DISPLAYNAME**    | NUCC Display Name            | Display name for taxonomy. e.g. `Counselor`                                                     |
| **PAYERSET\_BILLING\_CODE\_NAME**  | Payerset Billing Code Name   | Internal billing code name in Payerset. e.g. `OUTPATIENT MISCELLANEOUS (DEFAULT)`               |
| **PAYERSET\_BILLING\_CODE\_TYPE**  | Payerset Billing Code Type   | Internal billing code type in Payerset. e.g. `CSTM-ALL`                                         |
