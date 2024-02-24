---
description: >-
  This data contains the price transparency machine-readable files provided by
  Sentara that have been normalized into easy-to-use tables.
---

# Sentara

### Payerset Notes

**Table of Contents**&#x20;



{% hint style="warning" %}
Sentara's JSON files are not parseable in their posted format. Specifically, there are escape characters present in the JSON (\\) and a few entries where there are multiple double quotes ("CPT "LOADED" .."). This makes the JSON invalid, but these errors are fixable with a tool like UltraEdit or similar text editing tool, at which point they can be parsed.
{% endhint %}

{% embed url="https://optimahealth.healthsparq.com/healthsparq/public/#/one/city=&state=&postalCode=&country=&insurerCode=OPTIMAH_I&brandCode=OPTIMAH&alphaPrefix=&ProductId=/machine-readable-transparency-in-coverage" %}

### Schema: SENTARA

### Table 1: PROVIDERS

#### 1. Overview

The `PROVIDERS` table contains the mapping of provider identification numbers (NPI) and other related provider information. This table is designed to store provider-specific data.

#### 2. Columns

| Column Name    | Data Type | Description                                                          |
| -------------- | --------- | -------------------------------------------------------------------- |
| NPI            | VARCHAR   | The National Provider Identifier assigned to a healthcare provider.  |
| NPI\_RATE\_KEY | VARCHAR   | A unique identifier for the provider in the rate negotiation system. |
| TIN\_TYPE      | VARCHAR   | Type of Taxpayer Identification Number associated with the provider. |
| TIN\_VALUE     | VARCHAR   | Taxpayer Identification Number value for the provider.               |

### Table 2: NEGOTIATED\_RATES

#### 1. Overview

The `NEGOTIATED_RATES` table contains information about negotiated rates for various billing classes and codes. This table is used to store pricing data and rate negotiations.

#### 2. Columns

<table><thead><tr><th>Column Name</th><th width="159.33333333333331">Data Type</th><th>Description</th></tr></thead><tbody><tr><td>BILLING_CLASS</td><td>VARCHAR</td><td>The class or category of the billing code.</td></tr><tr><td>BILLING_CODE</td><td>VARCHAR</td><td>A unique identifier for the specific billing code.</td></tr><tr><td>EXPIRATION_DATE</td><td>VARCHAR</td><td>The expiration date of the negotiated rate.</td></tr><tr><td>NEGOTIATED_RATE</td><td>NUMBER</td><td>The negotiated rate for the specified billing code.</td></tr><tr><td>NEGOTIATED_TYPE</td><td>VARCHAR</td><td>The type of the negotiated rate (e.g., fixed, percentage, etc.).</td></tr><tr><td>NEGOTIATION_ARRANGEMENT</td><td>VARCHAR</td><td>The arrangement for the negotiated rate.</td></tr><tr><td>NPI_RATE_KEY</td><td>VARCHAR</td><td>A unique identifier for the provider in the rate negotiation system.</td></tr><tr><td>SERVICE_CODES</td><td>VARCHAR</td><td>The associated service codes for the negotiated rate.</td></tr><tr><td>BILLING_CODE_TYPE</td><td>VARCHAR</td><td>The type or standard of the billing code (e.g., ICD-10, CPT, etc.).</td></tr><tr><td>BILLING_CODE_TYPE_VERSION</td><td>VARCHAR</td><td>The version of the billing code type.</td></tr><tr><td>DESCRIPTION</td><td>VARCHAR</td><td>A description of the billing code.</td></tr><tr><td>NAME</td><td>VARCHAR</td><td>The name or title of the billing code.</td></tr><tr><td>TOC_ID</td><td>UUID</td><td>Links to the Reporting Plan ID in the table of contents.</td></tr></tbody></table>
