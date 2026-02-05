# Payerset Trusted Price Algorithm

### Overview

The Payerset Trusted Price Algorithm selects and scores healthcare pricing data from payer Transparency in Coverage (TiC) files. Given the scale of data, the algorithm operates within a sparse matrix architecture that processes data in partitioned chunks, aggregating across plans and prioritizing rate records based on the various criteria outlined below.

The algorithm produces a single trusted rate for each payer per NPI + billing code combination, along with benchmark comparisons and a confidence score that reflects data quality and consistency.

### Three-Track Processing

Provider NPIs are processed through one of three tracks based on entity type and data availability:

| NPI Track         | Entity Type                    | Key Criteria                                                                           |
| ----------------- | ------------------------------ | -------------------------------------------------------------------------------------- |
| **Individuals**   | Type 1 NPI                     | Single practitioners; taxonomy filtering viable; tighter medicare bands                |
| **Organizations** | Type 2 NPI (no hospital match) | Clinics, groups, ASCs; no taxonomy filtering other than hospital; wider medicare bands |
| **Hospitals**     | Type 2 NPI (hospital match)    | Matched against hospital MRF dataset; medicare IP/OP benchmark                         |

### Benchmark Bands

#### Medicare Benchmark

Rates are scored based on their ratio to locality-adjusted Medicare rates. Each track has different acceptable bands reflecting the natural variation in that provider type.

**Individuals (Type 1 NPI)**

| Medicare Ratio           | Confidence Level |
| ------------------------ | ---------------- |
| 75% - 250%               | HIGH             |
| 50% - 75% OR 250% - 350% | MEDIUM           |
| Outside above ranges     | LOW              |

**Organizations (Type 2 NPI, no hospital taxonomy match)**

| Medicare Ratio           | Confidence Level |
| ------------------------ | ---------------- |
| 85% - 350%               | HIGH             |
| 65% - 85% OR 350% - 500% | MEDIUM           |
| Outside above ranges     | LOW              |

**Hospitals (Type 2 NPI, with hospital taxonomy match)**

| Medicare Ratio            | Confidence Level |
| ------------------------- | ---------------- |
| 100% - 400%               | HIGH             |
| 75% - 100% OR 400% - 500% | MEDIUM           |
| Outside above ranges      | LOW              |

#### Hospital MRF Benchmark (Hospitals Only)

For NPIs matched to our hospital pricing dataset, the selected rate is compared against the hospital's published MRF rate.

| Hospital Ratio           | Score  |
| ------------------------ | ------ |
| 80% - 120%               | HIGH   |
| 50% - 80% OR 120% - 150% | MEDIUM |
| Outside above ranges     | LOW    |

**Note:** The payer TiC rate is always prioritized over the hospital MRF rate for the final `rate_selected` value. Hospital MRF data is used only as a validation benchmark, not as a source of truth, due to known accuracy issues in hospital-published data.

***

### Rate Selection Priority

Within each plan, rates are prioritized using a scoring system. Lower priority scores are preferred.

#### Negotiated Type Priority (Base Score)

| Negotiated Type | Base Score |
| --------------- | ---------- |
| negotiated      | 100        |
| fee schedule    | 200        |
| derived         | 300        |
| percentage      | 400        |
| other/unknown   | 500        |

#### Billing Class Priority (Added to Base)

**For Facilities (Type 2 NPI):**

| Billing Class | Score |
| ------------- | ----- |
| institutional | +10   |
| professional  | +20   |

**For Individuals (Type 1 NPI):**

| Billing Class | Score |
| ------------- | ----- |
| professional  | +10   |
| institutional | +20   |

#### Place of Service Priority (Added to Base)

**For Facilities:**

| Service Code | Meaning         | Score |
| ------------ | --------------- | ----- |
| 22           | Outpatient      | +1    |
| (empty)/null | All/unspecified | +2    |
| 11           | Office          | +3    |
| 21           | Inpatient       | +4    |
| other        |                 | +5    |

The reason 22 is prioritized here is because it is atypical for an outpatient rate to exist for a facility, usually indicating that there is an explicit exception that provides an outpatient rate, so we prioritize that rate over the more typical office rate. This is uncommon, but useful.

**For Individuals:**

| Service Code | Meaning         | Score |
| ------------ | --------------- | ----- |
| 11           | Office          | +1    |
| (empty)/null | All/unspecified | +2    |
| 22           | Outpatient      | +3    |
| 21           | Inpatient       | +4    |
| other        |                 | +5    |

#### Selection Logic

1. For each plan, calculate priority score for all rates matching the NPI + billing code
2. Select rate(s) with the lowest (best) priority score within that plan
3. Aggregate selected rates across all plans (tracking min, max, sum, count)
4. When merging across plans, lower priority scores replace higher ones; equal priorities merge aggregates

***

### Cross-Plan Aggregation

As rates are processed across multiple plans for the same payer and network, the algorithm maintains:

* `rate_min` - Minimum rate observed across plans
* `rate_max` - Maximum rate observed across plans
* `rate_sum` - Sum of all rates (for average calculation)
* `rate_count` - Number of plans contributing rates

The **spread ratio** (`rate_max / rate_min`) serves as a consistency indicator:

| Spread Ratio | Interpretation    | Confidence Impact |
| ------------ | ----------------- | ----------------- |
| < 1.5        | Tight agreement   | HIGH              |
| 1.5 - 3.0    | Moderate variance | MEDIUM            |
| > 3.0        | Wide disagreement | LOW               |

The **rate count** also affects confidence:

| Rate Count | Interpretation   | Confidence Impact |
| ---------- | ---------------- | ----------------- |
| 5+ plans   | Strong support   | HIGH              |
| 2-4 plans  | Moderate support | MEDIUM            |
| 1 plan     | Single source    | LOW               |

***

### Confidence Calculation

The final confidence score combines multiple factors:

| Factor                              | Weight    | Possible Scores                                                      |
| ----------------------------------- | --------- | -------------------------------------------------------------------- |
| Medicare benchmark                  | Primary   | HIGH / MEDIUM / LOW / NULL                                           |
| Hospital benchmark (hospitals only) | Primary   | HIGH / MEDIUM / LOW / NULL                                           |
| Spread ratio                        | Secondary | HIGH / MEDIUM / LOW                                                  |
| Rate count                          | Secondary | HIGH / MEDIUM / LOW                                                  |
| Negotiated type                     | Tertiary  | HIGH (negotiated) / MEDIUM (fee schedule) / LOW (derived/percentage) |

#### Confidence Resolution

The final confidence level is determined by the **lowest score among primary factors**, adjusted down if secondary factors indicate problems:

```
confidence = min(
    medicare_benchmark_score,     -- NULL treated as MEDIUM
    hospital_benchmark_score,     -- NULL ignored for non-hospitals
    spread_score,
    count_score
)

-- Negotiated type can only lower confidence, not raise it
if negotiated_type in ('derived', 'percentage'):
    confidence = min(confidence, MEDIUM)
```

**Final Confidence Values:** `HIGH`, `MEDIUM`, `LOW`

***

### Input Filters

The following filters are applied before rate selection. MS-DRG will be filtered only for hospitals.

| Field                     | Filter                                                          |
| ------------------------- | --------------------------------------------------------------- |
| `billing_code_type`       | IN ('CPT', 'HCPCS','MS-DRG')                                    |
| `billing_code_modifier`   | IN ('00', '', ' ', ' ') — effectively unmodified codes only\*\* |
| `negotiation_arrangement` | = 'ffs' (fee-for-service)                                       |
| `npi`                     | Length = 10, starts with '1' or '2'                             |
| `service_codes`           | Contains '11', '21', '22', or is empty/null                     |

***

\*\* As some payers place multiple modifiers in an array, the filtering is slightly different but effectively achieves the same result.

### Taxonomy Filtering (Individuals Only)

For Type 1 NPIs (individuals), taxonomy codes from NPPES are used to validate that the billing code is appropriate for the provider's specialty. This categorization is based on substantial research and consultations with clinicians.

**Rationale:** Individual provider taxonomies are relatively stable (a cardiologist remains a cardiologist), whereas organization taxonomies are frequently outdated, incorrect, or incomplete.

**Implementation:** A mapping of taxonomy classifications to valid billing code ranges is applied. Rates for codes outside the provider's expected scope are deprioritized (not filtered) by adding a penalty to the priority score.

***

### Output Schema

Each output record contains:

| Field                     | Type    | Description                                                  |
| ------------------------- | ------- | ------------------------------------------------------------ |
| `npi`                     | VARCHAR | 10-digit National Provider Identifier                        |
| `billing_code`            | VARCHAR | CPT or HCPCS code                                            |
| `negotiated_type`         | VARCHAR | Type of rate (negotiated, fee schedule, derived, percentage) |
| `plan_network`            | VARCHAR | Payerset-created network                                     |
| `billing_class`           | VARCHAR | professional or institutional                                |
| `service_codes`           | VARCHAR | Place of service codes that were selected                    |
| `entity_type`             | VARCHAR | Individual, Organization, or Hospital                        |
| `taxonomy_grouping`       | VARCHAR | NPPES taxonomy grouping (if available)                       |
| `taxonomy_classification` | VARCHAR | NPPES taxonomy classification (if available)                 |
| `taxonomy_specialization` | VARCHAR | NPPES taxonomy specialization (if available)                 |
| `rate_selected`           | DOUBLE  | The trusted rate (average across contributing plans)         |
| `rate_min`                | DOUBLE  | Minimum rate observed across plans                           |
| `rate_max`                | DOUBLE  | Maximum rate observed across plans                           |
| `rate_avg`                | DOUBLE  | Average rate across plans                                    |
| `rate_count`              | INT     | Number of plans contributing to this rate                    |
| `medicare_benchmark`      | DOUBLE  | Locality-adjusted Medicare rate (NULL if unavailable)        |
| `medicare_ratio`          | DOUBLE  | rate\_selected / medicare\_benchmark (NULL if no benchmark)  |
| `hospital_benchmark`      | DOUBLE  | Hospital MRF rate (NULL if not a matched hospital)           |
| `hospital_ratio`          | DOUBLE  | rate\_selected / hospital\_benchmark (NULL if no benchmark)  |
| `priority_score`          | INT     | Internal priority score (lower = higher priority)            |
| `confidence`              | VARCHAR | HIGH, MEDIUM, or LOW                                         |

### Revision History

| Version | Date    | Changes                         |
| ------- | ------- | ------------------------------- |
| 1.0     | 2026-01 | Initial algorithm specification |
