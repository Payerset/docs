---
description: >-
  This data contains the price transparency machine-readable files provided by
  Aetna that have been normalized into easy-to-use tables.
---

# Aetna

### Payerset Notes

Aetna's table of contents was not immediately available on their website but can be found in its entirety here: [https://health1.aetna.com/app/public/#/one/insurerCode=AETNACVS\_I\&brandCode=ALICSI/machine-readable-transparency-in-coverage](https://health1.aetna.com/app/public/#/one/insurerCode=AETNACVS\_I\&brandCode=ALICSI/machine-readable-transparency-in-coverage)

If you are looking for a specific plan, please reach out to info@payerset.com to get access to the mapping back to the full plan list, which we maintain in a separate table.

### Schema: AETNA

### Table 1: PROVIDER\_MAP - 15,984,744,618 Records

#### 1. Overview

The `PROVIDER_MAP` table contains the mapping of provider identification numbers (NPI) and other related provider information. This table is designed to store provider-specific data.

#### 2. Columns

| Column Name    | Data Type | Description                                                          |
| -------------- | --------- | -------------------------------------------------------------------- |
| NPI            | VARCHAR   | The National Provider Identifier assigned to a healthcare provider.  |
| NPI\_RATE\_KEY | VARCHAR   | A unique identifier for the provider in the rate negotiation system. |
| TIN\_TYPE      | VARCHAR   | Type of Taxpayer Identification Number associated with the provider. |
| TIN\_VALUE     | VARCHAR   | Taxpayer Identification Number value for the provider.               |

### Table 2: NEGOTIATED\_RATES - 2,725,735,036,850 Records

#### 1. Overview

The `NEGOTIATED_RATES` table contains information about negotiated rates for various billing classes and codes. This table is used to store pricing data and rate negotiations.

#### 2. Columns

<table><thead><tr><th>Column Name</th><th width="159.33333333333331">Data Type</th><th>Description</th></tr></thead><tbody><tr><td>BILLING_CLASS</td><td>VARCHAR</td><td>The class or category of the billing code.</td></tr><tr><td>BILLING_CODE</td><td>VARCHAR</td><td>A unique identifier for the specific billing code.</td></tr><tr><td>EXPIRATION_DATE</td><td>VARCHAR</td><td>The expiration date of the negotiated rate.</td></tr><tr><td>NEGOTIATED_RATE</td><td>NUMBER</td><td>The negotiated rate for the specified billing code.</td></tr><tr><td>NEGOTIATED_TYPE</td><td>VARCHAR</td><td>The type of the negotiated rate (e.g., fixed, percentage, etc.).</td></tr><tr><td>NEGOTIATION_ARRANGEMENT</td><td>VARCHAR</td><td>The arrangement for the negotiated rate.</td></tr><tr><td>NPI_RATE_KEY</td><td>VARCHAR</td><td>A unique identifier for the provider in the rate negotiation system.</td></tr><tr><td>SERVICE_CODES</td><td>VARCHAR</td><td>The associated service codes for the negotiated rate.</td></tr><tr><td>BILLING_CODE_TYPE</td><td>VARCHAR</td><td>The type or standard of the billing code (e.g., ICD-10, CPT, etc.).</td></tr><tr><td>BILLING_CODE_TYPE_VERSION</td><td>VARCHAR</td><td>The version of the billing code type.</td></tr><tr><td>DESCRIPTION</td><td>VARCHAR</td><td>A description of the billing code.</td></tr><tr><td>NAME</td><td>VARCHAR</td><td>The name or title of the billing code.</td></tr><tr><td>TOC_ID</td><td>UUID</td><td>Links to the Reporting Plan ID in the table of contents.</td></tr></tbody></table>
