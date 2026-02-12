# Payerset Trusted Price Algorithm

## Payerset Trusted Price Algorithm

### Overview

The Payerset Trusted Price Algorithm selects and scores healthcare pricing data from payer Transparency in Coverage (TiC) files. Given the scale of data, the algorithm operates within a sparse matrix architecture that processes data in partitioned chunks, aggregating across plans and prioritizing rate records based on the various criteria outlined below.

The algorithm produces a single trusted rate for each payer per NPI + billing code combination, along with benchmark comparisons and a confidence score that reflects data quality and consistency.

### Three-Track Processing

Provider NPIs are processed through one of three tracks based on entity type and data availability:

* **Individual** — Type 1 NPI (EntityTypeCode = '1')
* **Organization** — Type 2 NPI (EntityTypeCode = '2'), not matched to hospital NPI list
* **Hospital** — NPI present in the hospital NPI reference dataset (regardless of EntityTypeCode)

### Network Prioritization (Primary vs. Secondary)

Some payers publish rates under multiple reporting entities. In many cases, only a subset of these entities represent the payer's own negotiated networks — the rest are rental or third-party networks whose rates may be less representative.

#### Configuration

The `PAYER_ENTITY_MAP` defines which reporting entities are considered **primary** for a given payer. For example:

```
MASS_GENERAL_BRIGHAM:
  - Allways Health Partners
  - Mass General Brigham Health Plan
```

#### Prioritization Logic

When a payer is present in `PAYER_ENTITY_MAP`:

1. **Primary entities** — Plans belonging to reporting entities listed in the map are prioritized. These represent the payer's own negotiated rates.
2. **Secondary entities** — Plans from all other reporting entities (rental/third-party networks) serve as fallback. For a given NPI + billing code combination, secondary rates are used only when no rate exists from a primary entity.

When a payer is **not** in `PAYER_ENTITY_MAP`, all plans are treated equally with no primary/secondary distinction.

> **Current implementation note:** The code currently filters to primary entities only (excluding secondary plans entirely). The fallback behavior described above is planned but not yet implemented.

### Benchmark Bands

#### Medicare Benchmark

Rates are scored based on their ratio to locality-adjusted Medicare rates. Each track has different acceptable bands reflecting the natural variation in that provider type.

**Individuals (Type 1 NPI)**

| Band         | Range                   | Confidence |
| ------------ | ----------------------- | ---------- |
| Acceptable   | 75%–250% of Medicare    | HIGH       |
| Marginal     | 50%–75% or 250%–350%    | MEDIUM     |
| Outside      | Below 50% or above 350% | LOW        |
| No benchmark | NULL                    | MEDIUM     |

**Organizations (Type 2 NPI, no hospital match)**

| Band         | Range                   | Confidence |
| ------------ | ----------------------- | ---------- |
| Acceptable   | 85%–350% of Medicare    | HIGH       |
| Marginal     | 65%–85% or 350%–500%    | MEDIUM     |
| Outside      | Below 65% or above 500% | LOW        |
| No benchmark | NULL                    | MEDIUM     |

**Hospitals (matched to hospital NPI list)**

| Band         | Range                   | Confidence |
| ------------ | ----------------------- | ---------- |
| Acceptable   | 100%–400% of Medicare   | HIGH       |
| Marginal     | 75%–100% or 400%–500%   | MEDIUM     |
| Outside      | Below 75% or above 500% | LOW        |
| No benchmark | NULL                    | MEDIUM     |

#### Medicare Benchmark Source Selection

The Medicare benchmark rate is selected based on billing code type and service context:

* **CPT/HCPCS codes** — Uses Medicare Physician Fee Schedule (outpatient data). If service codes include `11` (Office), uses the **Non-Facility Price**; otherwise uses the **Facility Price**.
* **MS-DRG codes** — Uses Medicare Inpatient data, joined on NPI + DRG code to get locality-specific reimbursement.
* **Lab codes** — Uses Medicare Clinical Lab Fee Schedule as a fallback when outpatient and inpatient benchmarks are not available. Only unmodified rates (blank modifier) are used.

Priority: Outpatient > Inpatient > Lab (via `COALESCE`).

#### Hospital MRF Benchmark (Hospitals Only)

For NPIs matched to the hospital pricing dataset, the selected rate is compared against the hospital's published MRF rate (median rate by system).

| Band         | Range                   | Confidence |
| ------------ | ----------------------- | ---------- |
| Aligned      | 80%–120% of MRF         | HIGH       |
| Marginal     | 50%–80% or 120%–150%    | MEDIUM     |
| Outside      | Below 50% or above 150% | LOW        |
| No benchmark | NULL                    | (ignored)  |

**Note:** The payer TiC rate is always prioritized over the hospital MRF rate for the final rate\_selected value. Hospital MRF data is used only as a validation benchmark, not as a source of truth, due to known accuracy issues in hospital-published data.

### Rate Selection Priority

Within each plan, rates are prioritized using a scoring system. Lower priority scores are preferred.

#### Negotiated Type Priority (Base Score)

| Negotiated Type | Score |
| --------------- | ----- |
| negotiated      | 100   |
| fee schedule    | 200   |
| derived         | 300   |
| percentage      | 400   |
| (other)         | 500   |

#### Billing Class Priority (Added to Base)

**For Facilities (Type 2 NPI / Hospitals):**

| Billing Class | Score |
| ------------- | ----- |
| institutional | +10   |
| (other)       | +20   |

**For Individuals (Type 1 NPI):**

| Billing Class | Score |
| ------------- | ----- |
| professional  | +10   |
| (other)       | +20   |

#### Place of Service Priority (Added to Base)

**For Facilities (Organizations and Hospitals):**

| Service Code    | Score | Rationale                                            |
| --------------- | ----- | ---------------------------------------------------- |
| 22 (Outpatient) | +1    | Atypical for facility — indicates explicit exception |
| (blank)         | +2    | No specific place of service                         |
| 11 (Office)     | +3    | Standard office rate                                 |
| 21 (Inpatient)  | +4    | Standard inpatient rate                              |
| (other)         | +5    | —                                                    |

The reason 22 is prioritized here is because it is atypical for an outpatient rate to exist for a facility, usually indicating that there is an explicit exception that provides an outpatient rate, so we prioritize that rate over the more typical office rate. This is uncommon, but useful.

**For Individuals:**

| Service Code    | Score | Rationale                     |
| --------------- | ----- | ----------------------------- |
| 11 (Office)     | +1    | Most representative setting   |
| (blank)         | +2    | No specific place of service  |
| 22 (Outpatient) | +3    | Less typical for individuals  |
| 21 (Inpatient)  | +4    | Least typical for individuals |
| (other)         | +5    | —                             |

#### Selection Logic

1. For each plan, calculate priority score for all rates matching the NPI + billing code
2. Select rate(s) with the lowest (best) priority score within that plan
3. Aggregate selected rates across all plans (tracking min, max, sum, count)
4. When merging across plans, lower priority scores replace higher ones; equal priorities merge aggregates

### Cross-Plan Aggregation

As rates are processed across multiple plans for the same payer and network, the algorithm maintains:

* **rate\_min** — Minimum rate observed across plans
* **rate\_max** — Maximum rate observed across plans
* **rate\_sum** — Sum of all rates (for average calculation)
* **rate\_count** — Number of rate records contributing
* **plan\_count** — Number of plans contributing rates

The spread ratio (`rate_max / rate_min`) serves as a consistency indicator:

| Spread Ratio  | Confidence |
| ------------- | ---------- |
| < 1.5 or NULL | HIGH       |
| 1.5–3.0       | MEDIUM     |
| > 3.0         | LOW        |

The plan count also affects confidence:

| Plan Count | Confidence |
| ---------- | ---------- |
| 5+         | HIGH       |
| 2–4        | MEDIUM     |
| 1          | LOW        |

### Confidence Calculation

The final confidence score combines multiple factors:

#### Primary Factors

* Medicare benchmark score (NULL → MEDIUM)
* Hospital MRF benchmark score (NULL → ignored for non-hospitals)
* Spread ratio score
* Plan count score

#### Secondary Factor

* Negotiated type: `derived` or `percentage` can only lower confidence, never raise it

#### Confidence Resolution

The final confidence level is determined by the lowest score among primary factors, adjusted down if the secondary factor indicates problems:

```
confidence = min(
    medicare_benchmark_score,     -- NULL treated as MEDIUM
    hospital_benchmark_score,     -- NULL mapped to HIGH (effectively ignored for non-hospitals)
    spread_score,
    count_score
)

-- Negotiated type can only lower confidence, not raise it
if negotiated_type in ('derived', 'percentage'):
    confidence = min(confidence, MEDIUM)
```

**Final Confidence Values:** HIGH, MEDIUM, LOW

### Input Filters

The following filters are applied before rate selection:

| Field                    | Filter                                    |
| ------------------------ | ----------------------------------------- |
| billing\_code\_type      | IN ('CPT', 'HCPCS', 'MS-DRG')             |
| negotiation\_arrangement | = 'ffs'                                   |
| NPI format               | Starts with '1' or '2', length = 10       |
| service\_codes           | Contains '11', '21', or '22', or is blank |
| billing\_code\_modifier  | Blank or '00' \*\*                        |

\*\* As some payers place multiple modifiers in an array, the filtering is slightly different but effectively achieves the same result. Payers with simple comma-separated modifier lists (e.g., Aetna, United Healthcare) use strict equality checks. Other payers additionally check for blank elements within comma-separated arrays.

#### MS-DRG Leading Zero Handling

For United Healthcare specifically, MS-DRG billing codes are normalized by taking the rightmost 3 characters to strip leading zeros (e.g., `001` → `001`, `0001` → `001`).

#### Taxonomy Filtering

> **Note:** Taxonomy-based deprioritization for Individual NPIs is described in the algorithm design but is not yet implemented in the current codebase. When implemented, it will use taxonomy classifications from NPPES to validate that billing codes are appropriate for the provider's specialty, applying a priority penalty for out-of-scope codes rather than filtering them out.

### Blue Cross Blue Shield Handling

Blue Cross Blue Shield is processed differently from other payers. Because BCBS data is published as a single national dataset but represents many independent regional Blue plans, the algorithm splits processing by individual Blue payer (parsed from `PAYERSET_DISPLAY_PLAN_NAME`).

Each Blue payer is processed independently as if it were a separate payer, with its own:

* Directory structure (RATES, PROVIDERS, RP\_METADATA)
* Partition discovery
* Plan type grouping
* Output directory

### Output Schema

Each output record contains:

| Field                    | Description                                                               |
| ------------------------ | ------------------------------------------------------------------------- |
| npi                      | Provider NPI                                                              |
| billing\_code            | CPT, HCPCS, or MS-DRG code                                                |
| negotiated\_type         | Type of the selected rate (negotiated, fee schedule, derived, percentage) |
| plan\_type               | Payerset plan type classification                                         |
| billing\_class           | institutional or professional                                             |
| service\_codes           | Mapped to readable values: Office, Outpatient, Inpatient, or All          |
| entity\_type             | Individual, Organization, or Hospital                                     |
| taxonomy\_grouping       | Provider taxonomy grouping from NPPES                                     |
| taxonomy\_classification | Provider taxonomy classification from NPPES                               |
| taxonomy\_specialization | Provider taxonomy specialization from NPPES                               |
| rate\_min                | Minimum rate across plans                                                 |
| rate\_max                | Maximum rate across plans                                                 |
| rate\_avg                | Average rate (rate\_sum / rate\_count)                                    |
| rate\_count              | Number of rate records                                                    |
| plan\_count              | Number of plans contributing                                              |
| medicare\_benchmark      | Locality-adjusted Medicare rate                                           |
| medicare\_ratio          | rate\_avg / medicare\_benchmark                                           |
| hospital\_benchmark      | Hospital MRF median rate (hospitals only)                                 |
| hospital\_ratio          | rate\_avg / hospital\_benchmark                                           |
| priority\_score          | Final priority score of the selected rate                                 |
| confidence               | HIGH, MEDIUM, or LOW                                                      |
