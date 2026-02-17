# Payerset Trusted Price Algorithm

## Overview

The Payerset Trusted Price™ Algorithm procudes denormalized fee schedules from the Transparency in Coverage (TiC) machine-readable files published by health insurance carriers under federal price transparency regulations. The raw TiC data contains trillions of individual negotiated rate records spread across thousands of files per payer. This dataset condenses that data into a single, queryable fee schedule: one record per provider, billing code, entity type, and plan type, (along with a few other dimensions) with the most representative negotiated rate selected through a priority-based algorithm.

> **Schema Compatibility:** This documentation describes **Schema 2.0** of the Payerset Denormalized Fee Schedules. It is backwards-compatible with Schema 1.x — all fields present in Schema 1.x retain their original names and semantics. Schema 2.0 adds the `setting` field and removes the `taxonomy_grouping`, `taxonomy_classification`, and `taxonomy_specialization` fields that were present in Schema 1.x.

***

## Data Sources

Each payer's denormalized fee schedule is derived from three categories of source data.

### Payer TiC Data

Negotiated rates and provider-plan associations published by each payer in compliance with the Transparency in Coverage Final Rule, updated quarterly. These files are parsed, validated, and partitioned by Payerset into a standardized format with rate files partitioned by billing code prefix (`bc_left`) and provider files partitioned by NPI prefix (`npi_left`).

### NPPES (National Plan and Provider Enumeration System)

The CMS National Provider Identifier registry. Used to classify each NPI as Individual (Type 1) or Organization (Type 2), and to determine the MAC locality for Medicare benchmark lookups.

### Medicare Benchmarks

Three Medicare reference datasets provide benchmark rates against which negotiated rates are compared:

* **Outpatient (Physician Fee Schedule):** Facility and non-facility prices by HCPCS/CPT code and MAC locality.
* **Inpatient (DRG):** Total reimbursement amounts by NPI and MS-DRG code.
* **Labs (Clinical Lab Fee Schedule):** National lab rates by HCPCS code.

### Hospital Reference Data

* **Hospital NPIs:** Maps NPIs to hospital system identifiers, used to classify providers as hospitals regardless of their NPPES entity type.
* **Hospital Benchmarks:** Median hospital rates by system and billing code, derived from hospital chargemaster and machine-readable file data.

***

## How It Works

### Entity Classification

Every NPI is classified into exactly one of three entity types:

* **Individual:** NPPES EntityTypeCode = 1. Physicians, therapists, and other individual practitioners.
* **Hospital:** NPI appears in the Payerset Hospital NPI reference table, regardless of NPPES entity type.
* **Organization:** NPPES EntityTypeCode = 2 and NOT in the Hospital NPI table. Clinics, labs, imaging centers, and group practices.

A single NPI appears in exactly one entity type partition. Entity types are processed independently — the algorithm runs separately for each entity type, producing separate output files.

### Filtering

Before any rate selection occurs, raw TiC records are filtered to include only:

* **Billing code types:** CPT, HCPCS, or MS-DRG.
* **Negotiation arrangement:** Fee-for-service (`ffs`) only.
* **Billing code modifier:** Blank or `00` (base rates only, no modifier-specific variants).
* **Service codes:** `11` (Office), `21` (Inpatient Hospital), `22` (Outpatient Hospital), or blank (unspecified).
* **Valid NPIs:** 10 digits, starting with `1` or `2`.

Records that do not meet all of these criteria are excluded before priority scoring.

### The Trusted Price Algorithm

When multiple plans from the same payer report different negotiated rates for the same provider and billing code, the algorithm selects the most representative rate through a priority scoring system. Lower scores indicate higher trustworthiness.

The priority score is the sum of five components, each operating at a different order of magnitude to ensure strict dominance between layers.

**Tier (magnitude: 0 or 100,000)**

Some payers publish rates through multiple reporting entities, not all of which are equally authoritative. For example, a health system's own health plan typically reports more accurate rates for its own providers than a third-party rental network would. Tier 1 entities (the payer's own plans) receive a score of 0; Tier 2 entities (rental networks, third-party administrators) receive 100,000. This ensures any Tier 1 rate always takes precedence over any Tier 2 rate, regardless of the other components.

Tier assignments are payer-specific. For payers without an explicit tier configuration, all plans default to Tier 1 (no penalty applied).

**Negotiated Type (magnitude: 1,000–5,000)**

| Negotiated Type | Score | Rationale                                                                          |
| --------------- | ----- | ---------------------------------------------------------------------------------- |
| `negotiated`    | 1,000 | Directly negotiated dollar amount — most reliable                                  |
| `fee schedule`  | 2,000 | Published fee schedule rate — reliable but may not reflect actual negotiated terms |
| `derived`       | 3,000 | Calculated from other rates — less transparent methodology                         |
| `percentage`    | 4,000 | Percentage of an undisclosed base rate — requires conversion for usability         |
| Other / unknown | 5,000 | Catch-all for non-standard types                                                   |

**Billing Class (magnitude: 100–200)**

The preferred billing class depends on the entity type. Individual providers typically bill under `professional`, while hospitals and organizations typically bill under `institutional`.

| Entity Type  | Preferred (100) | Non-preferred (200) |
| ------------ | --------------- | ------------------- |
| Individual   | `professional`  | `institutional`     |
| Organization | `institutional` | `professional`      |
| Hospital     | `institutional` | `professional`      |

**Setting (magnitude: 10–20)**

The `setting` field indicates where a service is delivered. Hospitals prefer inpatient settings, while individual and organization providers prefer outpatient settings.

| Entity Type  | Preferred — score 10 | Non-preferred — score 20 |
| ------------ | -------------------- | ------------------------ |
| Individual   | `outpatient`, `both` | `inpatient`              |
| Organization | `outpatient`, `both` | `inpatient`              |
| Hospital     | `inpatient`, `both`  | `outpatient`             |

When the source data does not specify a setting, it defaults to `both`.

**Service Codes (magnitude: 1–5)**

Service codes (place of service) are ranked by relevance to the entity type.

**For Individuals:**

| Service Code  | Score | Label             |
| ------------- | ----- | ----------------- |
| Contains `11` | 1     | Office            |
| Blank         | 2     | All / unspecified |
| Contains `22` | 3     | Outpatient        |
| Contains `21` | 4     | Inpatient         |
| Other         | 5     | —                 |

**For Hospitals and Organizations:**

| Service Code  | Score | Label             |
| ------------- | ----- | ----------------- |
| Contains `22` | 1     | Outpatient        |
| Blank         | 2     | All / unspecified |
| Contains `11` | 3     | Office            |
| Contains `21` | 4     | Inpatient         |
| Other         | 5     | —                 |

**Priority Score Examples**

A directly negotiated, professional, outpatient, office-visit rate from a Tier 1 plan for an Individual provider:

`0 + 1,000 + 100 + 10 + 1 = 1,111`

A percentage-based, institutional, inpatient rate from a Tier 2 rental network for the same provider:

`100,000 + 4,000 + 200 + 20 + 4 = 104,224`

The algorithm selects the record with score 1,111. The magnitude gaps between components ensure strict dominance: no combination of billing class, setting, and service code preferences can override a worse negotiated type, and no negotiated type advantage can override a tier difference.

### State Merge

Plans within a plan type are processed sequentially. After each plan, the new records (staging) are merged into the accumulated state using these rules:

1. **Better priority wins:** If staging has a strictly lower priority score for an `(npi, billing_code)` pair, the existing state record is replaced entirely.
2. **Equal priority merges:** If priority scores match, rate statistics are combined — `rate_min` takes the lesser value, `rate_max` takes the greater, `rate_sum` and `rate_count` accumulate, and `plan_count` increments.
3. **Worse priority is discarded:** If staging has a higher (worse) priority score, it is ignored.

This produces a single record per `(npi, billing_code)` per entity type per plan type that represents the best-available rate across all plans.

### Medicare Benchmarking

After the state merge is complete for a billing code partition, each record is enriched with Medicare benchmark data. The benchmark lookup follows a waterfall:

1. **CPT/HCPCS codes** join to the Medicare Physician Fee Schedule on billing code and MAC locality. If the winning service code is Office (`11`), the non-facility price is used; otherwise, the facility price is used.
2. **MS-DRG codes** join to Medicare inpatient data on NPI and DRG code.
3. **Lab codes** fall through to the Clinical Lab Fee Schedule on billing code alone (used when neither outpatient nor inpatient benchmarks match).

The first match in this waterfall is used. The `medicare_ratio` is computed as `rate_avg / medicare_benchmark`. A ratio of 1.5 means the negotiated rate is 150% of Medicare.

For Hospital entity types, a `hospital_benchmark` is also computed from Payerset's hospital chargemaster/MRF reference data, joined on NPI (via hospital system identifier) and billing code. The `hospital_ratio` is `rate_avg / hospital_benchmark`.

### Confidence Scoring

Each record receives a composite confidence rating (`HIGH`, `MEDIUM`, or `LOW`) derived from up to four independent signals.

**Medicare Benchmark Confidence**

Based on `medicare_ratio`, with entity-specific thresholds reflecting expected pricing ranges:

| Entity Type  | HIGH        | MEDIUM                     | LOW                  |
| ------------ | ----------- | -------------------------- | -------------------- |
| Individual   | 0.75 – 2.50 | 0.50 – 0.75 or 2.50 – 3.50 | Outside these ranges |
| Organization | 0.85 – 3.50 | 0.65 – 0.85 or 3.50 – 5.00 | Outside these ranges |
| Hospital     | 1.00 – 4.00 | 0.75 – 1.00 or 4.00 – 5.00 | Outside these ranges |

When no Medicare benchmark is available, this component defaults to `MEDIUM`.

**Hospital MRF Confidence (Hospital entity type only)**

Based on `hospital_ratio`:

| Rating | Range                      |
| ------ | -------------------------- |
| HIGH   | 0.80 – 1.20                |
| MEDIUM | 0.50 – 0.80 or 1.20 – 1.50 |
| LOW    | Outside these ranges       |

When no hospital benchmark is available, this component is excluded from the composite — it does not penalize the score.

**Spread Confidence**

Based on the ratio of `rate_max / rate_min`:

| Rating | Spread Ratio |
| ------ | ------------ |
| HIGH   | < 1.5        |
| MEDIUM | 1.5 – 3.0    |
| LOW    | > 3.0        |

A narrow spread indicates consistent pricing across contributing rate records; a wide spread suggests heterogeneous inputs. When `rate_min` is zero, this component defaults to `HIGH`.

**Plan Count Confidence**

Based on the number of distinct plans that contributed rates at the winning priority level:

| Rating | Plan Count |
| ------ | ---------- |
| HIGH   | 5+         |
| MEDIUM | 2 – 4      |
| LOW    | 1          |

**Final Confidence**

The final confidence is the **minimum** of all applicable components. A single `LOW` component results in a `LOW` overall rating.

Additionally, if the negotiated type is `derived` or `percentage`, the final confidence is **capped at `MEDIUM`** regardless of the component scores. These rate types carry inherent uncertainty that prevents a `HIGH` rating.

***

## Output Schema

### Partitioning

The data is stored in a Hive-partitioned directory structure:

```
payer={PAYER}/
  plan_type={plan_type}/
    npi_left={4-digit NPI prefix}/
      entity_type={Individual|Organization|Hospital}/
        bc_left={2-digit billing code prefix}.parquet
```

`plan_type` and `entity_type` are stored as columns within each parquet file in addition to being partition directory keys. `npi_left` and `bc_left` are partition keys only — they are not stored as columns in the parquet files but are inferred automatically when querying with `hive_partitioning=true`.

### Column Definitions

| Column               | Type    | Description                                                                                                                  |
| -------------------- | ------- | ---------------------------------------------------------------------------------------------------------------------------- |
| `npi`                | VARCHAR | 10-digit National Provider Identifier.                                                                                       |
| `billing_code`       | VARCHAR | CPT, HCPCS, or MS-DRG code. MS-DRG codes are normalized to 3 digits (leading zeros stripped). See Notes.                     |
| `negotiated_type`    | VARCHAR | The type of the winning rate: `negotiated`, `fee schedule`, `derived`, or `percentage`.                                      |
| `plan_type`          | VARCHAR | Insurance plan type: `HMO`, `PPO`, `EPO`, `POS`, `Indemnity`, or other payer-reported values.                                |
| `billing_class`      | VARCHAR | `professional` or `institutional`.                                                                                           |
| `setting`            | VARCHAR | `inpatient`, `outpatient`, or `both`. New in Schema 2.0.                                                                     |
| `service_codes`      | VARCHAR | Place of service label: `Office`, `Outpatient`, `Inpatient`, or `All`.                                                       |
| `entity_type`        | VARCHAR | `Individual`, `Organization`, or `Hospital`.                                                                                 |
| `rate_min`           | DOUBLE  | Minimum negotiated rate observed across all contributing rate records.                                                       |
| `rate_max`           | DOUBLE  | Maximum negotiated rate observed across all contributing rate records.                                                       |
| `rate_avg`           | DOUBLE  | Mean negotiated rate (sum of all rates divided by count of all rates).                                                       |
| `rate_count`         | INTEGER | Number of individual rate records that contributed to this row.                                                              |
| `plan_count`         | INTEGER | Number of distinct plans that contributed rates at the winning priority level.                                               |
| `medicare_benchmark` | DOUBLE  | Medicare reference rate for this code and provider location. NULL when no benchmark is available.                            |
| `medicare_ratio`     | DOUBLE  | `rate_avg / medicare_benchmark`. NULL when no benchmark is available.                                                        |
| `hospital_benchmark` | DOUBLE  | Hospital chargemaster/MRF benchmark rate. Only populated for the Hospital entity type; NULL for Individual and Organization. |
| `hospital_ratio`     | DOUBLE  | `rate_avg / hospital_benchmark`. NULL when no benchmark is available.                                                        |
| `priority_score`     | INTEGER | Composite priority score of the winning rate (lower = more trustworthy). See The Trusted Price Algorithm.                    |
| `confidence`         | VARCHAR | Composite confidence rating: `HIGH`, `MEDIUM`, or `LOW`. See Confidence Scoring.                                             |

### Hive Partition Columns

When reading with `hive_partitioning=true`, these additional columns are available from the directory structure:

| Column     | Type    | Description                                                                                     |
| ---------- | ------- | ----------------------------------------------------------------------------------------------- |
| `npi_left` | VARCHAR | First 4 digits of the NPI. Useful for partition-aware queries.                                  |
| `bc_left`  | VARCHAR | First 2 characters of the original (pre-normalization) billing code. See the MS-DRG note below. |

### Key Cardinality

Each parquet file contains at most one row per `(npi, billing_code)`. Across the full dataset for a payer, the unique key is `(plan_type, entity_type, npi, billing_code)`.

***

## Notes and Limitations

### MS-DRG Code Normalization

MS-DRG codes are normalized to 3 digits by stripping leading zeros (e.g., `0470` becomes `470`). Because the source rate files are partitioned by the **original** billing code prefix, a normalized DRG code may reside in a partition that doesn't match its new prefix. For example, code `470` (originally `0470`) will appear in the `bc_left=04` partition rather than `bc_left=47`.

This does not affect query correctness when using `hive_partitioning=true` — the `billing_code` column inside the file is always the normalized value. However, it does mean that `bc_left` cannot be used as a reliable filter for MS-DRG billing codes. To query a specific DRG, filter on the `billing_code` column directly rather than relying on the `bc_left` partition.

### Setting Field

The `setting` field is new in Schema 2.0. It reflects the `setting` value from the payer's TiC data, defaulting to `both` when the source does not specify one. Not all payers populate this field; for those that don't, all records will show `both`.

### Percentage Rates

Records with `negotiated_type = 'percentage'` represent rates expressed as a percentage of an undisclosed base amount (typically billed charges). The `rate_avg` for these records is the percentage value itself (e.g., 80.0 meaning 80% of charges), not a dollar amount. These records always receive a confidence of `MEDIUM` or `LOW` and a priority score of at least 4,000, reflecting the inherent difficulty of interpreting percentage-based rates without knowing the base.

### Plan Count Interpretation

A `plan_count` of 1 does not necessarily mean only one plan covers that provider for that code. It means only one plan contributed rates at the winning priority level. Other plans may have reported rates that were superseded by higher-priority data during the state merge.

### Rate Statistics

The `rate_min`, `rate_max`, and `rate_avg` values reflect the aggregate of all individual rate records that contributed to the winning priority level across all plans. When `plan_count > 1`, these statistics span multiple plans. When `rate_count > 1` within a single plan, it typically means the rate file contained multiple rate entries for the same provider and billing code at the same priority level.

### Blue Cross Blue Shield

Blue Cross Blue Shield payers are processed individually by their regional Blue plan based on the Reporting Entities inside each machine-readable file (e.g., Blue Cross Blue Shield of Massachusetts, Highmark Blue Cross Blue Shield). Each regional plan's data appears as a separate payer directory in the output.
