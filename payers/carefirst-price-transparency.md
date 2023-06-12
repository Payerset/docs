---
description: >-
 This data contains the price transparency machine-readable files provided by
 Carefirst that have been normalized into easy-to-use tables.
---

# Carefirst Price Transparency

### Payerset Notes

Please see below for metadata.

### Schema: CAREFIRST

### Table 1: PROVIDER\_MAP - JUNE 2023 DATA PULL IN PROGRESS

#### 1. Overview

The `PROVIDER_MAP` table contains the mapping of provider identification numbers (NPI) and other related provider information. This table is designed to store provider-specific data.

#### 2. Columns

| Column Name     | Data Type | Nullable | Default | Description                             |
| ------------------- | --------- | -------- | ------- | -------------------------------------------------------------------- |
| NPI         | VARCHAR  | Yes   | NULL  | The National Provider Identifier assigned to a healthcare provider. |
| NPI\_RATE\_KEY | VARCHAR  | Yes   | NULL  | A unique identifier for the provider in the rate negotiation system. |
| TIN\_TYPE      | VARCHAR  | Yes   | NULL  | Type of Taxpayer Identification Number associated with the provider. |
| TIN\_VALUE     | VARCHAR  | Yes   | NULL  | Taxpayer Identification Number value for the provider.        |

### Table 2: NEGOTIATED\_RATES - JUNE 2023 DATA PULL IN PROGRESS

#### 1. Overview

The `NEGOTIATED_RATES` table contains information about negotiated rates for various billing classes and codes. This table is used to store pricing data and rate negotiations.

#### 2. Columns

| Column Name       | Data Type | Nullable | Default | Description                             |
| ------------------------ | --------- | -------- | ------- | -------------------------------------------------------------------- |
| BILLING\_CLASS      | VARCHAR  | Yes   | NULL  | The class or category of the billing code.              |
| BILLING\_CODE      | VARCHAR  | Yes   | NULL  | A unique identifier for the specific billing code.          |
| EXPIRATION\_DATE     | VARCHAR  | Yes   | NULL  | The expiration date of the negotiated rate.             |
| NEGOTIATED\_RATE     | NUMBER  | Yes   | NULL  | The negotiated rate for the specified billing code.         |
| NEGOTIATED\_TYPE     | VARCHAR  | Yes   | NULL  | The type of the negotiated rate (e.g., fixed, percentage, etc.).   |
| NEGOTIATION\_ARRANGEMENT | VARCHAR  | Yes   | NULL  | The arrangement for the negotiated rate.               |
| NPI\_RATE\_KEY   | VARCHAR  | Yes   | NULL  | A unique identifier for the provider in the rate negotiation system. |
| SERVICE\_CODES      | VARCHAR  | Yes   | NULL  | The associated service codes for the negotiated rate.        |

### Table 3: BILLING\_CODES - JUNE 2023 DATA PULL IN PROGRESS

#### 1. Overview

The `BILLING_CODES` table contains information about various billing codes, their types, and other related data. This table is designed to store billing code-specific data.

#### 2. Columns

| Column Name         | Data Type | Nullable | Default | Description                                 |
| ---------------------------- | --------- | -------- | ------- | --------------------------------------------------------------------------- |
| BILLING\_CODE        | VARCHAR  | Yes   | NULL  | A unique identifier for the specific billing code.             |
| BILLING\_CODE\_TYPE     | VARCHAR  | Yes   | NULL  | The type or standard of the billing code (e.g., ICD-10, CPT, etc.).     |
| BILLING\_CODE\_TYPE\_VERSION | VARCHAR  | Yes   | NULL  | The version of the billing code type.                    |
| DESCRIPTION         | VARCHAR  | Yes   | NULL  | A description of the billing code.                     |
| NAME             | VARCHAR  | Yes   | NULL  | The name or title of the billing code.                   |
| TOC\_ID           | VARCHAR  | Yes   | NULL  | The unique identifier for the table of contents entry for the billing code. |
