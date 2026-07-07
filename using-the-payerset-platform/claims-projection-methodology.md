---
description: >-
  This is the methodology by which we project claims using multiple sources as
  detailed below.
---

# Claims Projection Methodology

## Claims Projection Methodology

This document describes how Payerset estimates the universe-level claim volume that each rendering provider performs, given a partial sample of observed claims. It is intended as a transparent, auditable account of every input, transformation, and assumption that produces the projected unit counts published in our APIs and dashboards.

### 1. What we project

Each row in our output represents a single cell at the source-claims grain:

```
BILLING_NPI_NBR × FACILITY_NPI_NBR × PROCEDURE_CD × PROCEDURE_MODIFIER
× PAYER_ID × PAYER_SUBCHANNEL_NAME × PAYER_CHANNEL_NAME
× CLAIM_TYPE_CD × SETTING × RATE_YEAR
```

`CLAIM_TYPE_CD` is `P` (professional) or `I` (institutional); `SETTING` is `Office`, `Outpatient`, `Inpatient`, or `Other`. Together they replace the place-of-service / type-of-bill dimensions used in earlier versions — POS and TOB are no more granular than (claim\_type, setting) for downstream consumers. The projection grain is otherwise identical to the source PurpleLab grain, so the output can be LEFT JOINed onto the raw claims feed by joining on the ten key columns above; see §7 for the join semantics.

For every cell we publish three observed/projected pairs and the coverage metadata:

* `OBSERVED_UNITS` — the count we directly observed in the source claims feed.
* `PROJECTED_UNITS` — our estimate of the _true_ count of services rendered by that NPI in that cell, including services that fell outside our sample. Computed as `OBSERVED_UNITS / COVERAGE_ESTIMATE`.
* `PROJECTED_UNITS_P5`, `PROJECTED_UNITS_P95` — the 5th and 95th percentile of a 500-draw parametric bootstrap that adds observation and coverage noise around the deterministic point.
* `ATTRIBUTED_CLAIM_COUNT` — the sum of PurpleLab `ATTRIBUTED_CLAIM_COUNT` over the cell, i.e. observed claim count at the source grain.
* `ATTRIBUTED_CLAIM_COUNT_PROJECTED` — projected claim count. Computed as observed claim count × `1 / COVERAGE_ESTIMATE` (the same factor used on units and charge dollars). Bootstrap percentiles are not published for claim count; the bootstrap is parametric on units only.
* `ATTRIBUTED_CHARGE_AMT` — the sum of PurpleLab `ATTRIBUTED_CHARGE_AMT` over the cell, i.e. observed billed-charge dollars at the source grain.
* `ATTRIBUTED_CHARGE_AMT_PROJECTED` — projected billed-charge dollars. Computed as observed charge dollars × the same `1 / COVERAGE_ESTIMATE` factor used on units.
* `COVERAGE_ESTIMATE` — the implied share of the universe captured by the source feed for this cell. Bounded to `[0.05, 1.00]`.
* `COVERAGE_SOURCE` — which tier of the hierarchy produced the estimate (see §6).
* `COVERAGE_GRAIN` — the grain at which the coverage fraction was estimated, including the ratio metric (e.g. `npi|procedure|year|medicaid_universe|claims_vs_lines`).
* `CONFIDENCE_BUCKET` — `high`, `medium`, `low`, or `suppress`.

**Coverage keying (V4).** `COVERAGE_ESTIMATE` is computed per `RATE_YEAR`, at the finest grain the denominator supports: NPI × procedure where the administrative source reports that NPI, county × procedure as a fallback, and coarser calibrated or prior tiers below that. Within a coverage key, the estimate is shared across every output row that maps to it — different modifier / facility / payer rows for the same key get the same `1 / COVERAGE_ESTIMATE` factor applied to their individual observed values. V3 pooled all rate years of observed volume against single-year denominators, which inflated coverage by roughly the number of years in the feed; V4 keys the numerator by rate year and year-matches denominators where the source is year-grained (T-MSIS), capping at the latest available source year.

The point estimate is `PROJECTED_UNITS = OBSERVED_UNITS / COVERAGE_ESTIMATE`. The bootstrap, used only for `PROJECTED_UNITS_P5` and `PROJECTED_UNITS_P95`, adds observation noise (Negative Binomial around `OBSERVED_UNITS`) and coverage noise (lognormal around `COVERAGE_ESTIMATE`) to characterize the uncertainty around that point.

#### 1.1 What `OBSERVED_UNITS` is — and what it is not

`OBSERVED_UNITS` is the unit count from the **source claims feed (PurpleLab) only**. It is not a pooled or deduplicated combination of multiple data sources. The pipeline ingests several other datasets, but each plays a different role and none of them adds to `OBSERVED_UNITS`:

* The provider directory (NPPES) attaches geography and taxonomy to each NPI.
* T-MSIS Medicaid Provider Spending is the denominator for the Medicaid direct tiers.
* The CMS Physician & Other Practitioners PUF is the denominator for the Medicare FFS professional tiers and the yardstick for the Commercial capture calibration.
* The CMS Medicare Inpatient (by Provider and Service) PUF is the denominator for the Medicare FFS inpatient tiers.
* The CMS Outpatient PUF (+ OPPS Addendum B crosswalk + POS Facilities geography) is the denominator for the Medicare FFS outpatient tier.
* The CMS Geographic Variation PUF supplies county MA participation rates for the MA-scaled tier.
* The CMS Physician provider-level rollup is used only in QA (per-NPI envelope) and never enters the projection math.

The reference datasets shape `COVERAGE_ESTIMATE`, which determines how `OBSERVED_UNITS` is scaled to a universe estimate. `OBSERVED_UNITS` itself is always a single-source quantity. This distinction matters when reconciling counts: any reader who tries to add observed PurpleLab units to T-MSIS or PUF totals is double-counting.

#### 1.2 Which PurpleLab field underlies `OBSERVED_UNITS`

The PurpleLab `CLAIMS_ORDERED` feed exposes two volume fields per row:

* **`ATTRIBUTED_TOTAL_UNITS`** — units on the original-claim grain, attributed to the billing NPI on that row. Populated for \~94% of professional rows and \~97% of institutional rows.
* **`COUNT_OF_UNITS`** — units sourced from a matched remittance advice. Only populated for \~13–16% of rows (the remit-matched subset).

We sum `ATTRIBUTED_TOTAL_UNITS` because it reflects the full claim population in the feed. Rows where `ATTRIBUTED_TOTAL_UNITS` is null are remits without a matched open claim and are skipped — they have no billing-NPI grain row to attribute to. This is implemented in `stages/ingest.py`.

#### 1.3 Ratio metrics: like-for-like coverage (V4)

Coverage is a ratio of "what we saw" to "what happened", and V4 requires the two sides to count the same thing:

| Tier family           | Numerator (PurpleLab)    | Denominator (administrative) | Why                                                                                                                                                                                                                                                                                                                                |
| --------------------- | ------------------------ | ---------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| T-MSIS (Medicaid)     | `ATTRIBUTED_CLAIM_COUNT` | T-MSIS `TOTAL_CLAIM_LINES`   | T-MSIS reports claim lines. Units-per-line for drug J-codes run into the hundreds; a units numerator drove raw coverage far above 1.0 and clamped to no-projection. Claims vs lines is the closest available pairing (a claim has ≥ 1 line, so coverage is slightly _under_-estimated — conservative in the projection direction). |
| Medicare inpatient    | `ATTRIBUTED_CLAIM_COUNT` | CMS `Total_Discharges`       | An inpatient stay maps to a claim, not to revenue-line units.                                                                                                                                                                                                                                                                      |
| Medicare professional | `ATTRIBUTED_TOTAL_UNITS` | PUF `Tot_Srvcs`              | CMS counts `Tot_Srvcs` as units for drug codes and services otherwise — the closest like-for-like for a professional-claim feed.                                                                                                                                                                                                   |
| Medicare outpatient   | `ATTRIBUTED_TOTAL_UNITS` | Outpatient PUF `CAPC_Srvcs`  | APC service counts are line-level services.                                                                                                                                                                                                                                                                                        |

The ratio metric used by each tier is recorded in `COVERAGE_GRAIN` so a reviewer never has to guess. The resulting factor `1 / COVERAGE_ESTIMATE` is dimensionless (a capture share) and is applied uniformly to units, claim counts, and charge dollars.

### 2. Inputs

| Input              | Source                     | Grain                                                                                                                                                                                                | Notes                                                                                                                                                                                                                                                                                                                                                                                                                     |
| ------------------ | -------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Source claims      | PurpleLab `CLAIMS_ORDERED` | one row per `BILLING_NPI_NBR × FACILITY_NPI_NBR × PROCEDURE_CD × PROCEDURE_MODIFIER × PAYER_ID × PAYER_SUBCHANNEL_NAME × PAYER_CHANNEL_NAME × CLAIM_TYPE_CD × SETTING × RATE_YEAR` after aggregation | Includes professional and institutional (`CLAIM_TYPE_CD = 'P' / 'I'`); `SETTING` resolves to `Office` / `Outpatient` / `Inpatient` / `Other`. Volume sourced from `ATTRIBUTED_TOTAL_UNITS` (see §1.2); claim count from `ATTRIBUTED_CLAIM_COUNT`; charge dollars from `ATTRIBUTED_CHARGE_AMT`. `FACILITY_NPI_NBR` is sparsely populated (mostly on institutional rows). `PROCEDURE_MODIFIER` is NULL for unmodified codes |
| Provider directory | Enriched NPPES (Type 2)    | one row per organization NPI                                                                                                                                                                         | Includes county FIPS, primary taxonomy, CBSA, lifetime claim count                                                                                                                                                                                                                                                                                                                                                        |

Claims are joined to the provider directory on billing NPI to attach county, primary taxonomy, and CBSA before any aggregation. Ingest additionally classifies each row for coverage-tier eligibility: `subch_class` (`ffs` / `ma` / `other` within the Medicare channel, driven by configured subchannel-name lists; `all` elsewhere) and `claim_class` (`prof` / `drg` / `op` / `other_inst`). Medicare rows whose subchannel matches neither configured list only qualify for the channel prior, and the QA report itemizes them.

### 3. Reference data sources

All references except the channel priors are optional: when a file is absent the loader registers an empty view and the corresponding tier produces zero rows, with the affected cells falling through to lower tiers.

| Reference                                             | Provider                                                                                                                                                  | Used for                                                                                                                                      | Grain                                            |
| ----------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------ |
| Channel priors                                        | Payerset seed (`data/seed/channel_priors.csv`)                                                                                                            | Per-channel coverage prior (Commercial 0.60, Medicare 0.75, Medicaid 0.80)                                                                    | national × channel                               |
| T-MSIS Medicaid Provider Spending                     | CMS T-MSIS Analytic Files                                                                                                                                 | Denominator for `tmsis_npi_direct` and the `tmsis_direct` county fallback                                                                     | NPI × HCPCS × year (claim lines)                 |
| CMS Physician & Other Practitioners PUF               | CMS                                                                                                                                                       | Denominator for `medicare_ffs_npi_direct` / `medicare_ffs_county_benchmark`; yardstick for the Commercial capture calibration; QA cross-check | NPI × HCPCS (FFS, single vintage year)           |
| CMS Medicare Inpatient (by Provider & Service) PUF    | CMS — see [Payerset Medicare Inpatient methodology](https://docs.payerset.com/using-the-payerset-platform/data-dictionary/medicare-inpatient-methodology) | Denominator for `medicare_inpatient_npi_direct` / `direct_inpatient_benchmark`                                                                | NPI × MS-DRG (FFS Part A discharges)             |
| CMS Outpatient PUF + OPPS Addendum B + POS Facilities | CMS                                                                                                                                                       | Denominator for `medicare_outpatient_benchmark`                                                                                               | county × APC (FFS services), HCPCS→APC crosswalk |
| CMS Geographic Variation PUF                          | CMS                                                                                                                                                       | County MA participation rates for `ma_scaled_benchmark`                                                                                       | county                                           |
| CMS Physician provider-level rollup                   | CMS                                                                                                                                                       | QA per-NPI envelope only                                                                                                                      | NPI totals                                       |

All references are versioned and stored under `data/reference/` with the source URL, retrieval date, and any source-side suppression rules captured in `DOWNLOADS.md`. `scripts/fetch_all_references.py` orchestrates all downloads.

### 4. Pipeline stages

The end-to-end run executes four ordered stages plus a reference-load step. All stages share a single in-process DuckDB connection; intermediate outputs are materialized as DuckDB tables, not files.

#### Stage 0 — Reference loads

All references in §3 are registered as DuckDB views; absent files register empty views.

#### Stage 1 — Ingest and geography

The source parquet is filtered to the configured states (if any) and the configured `sample_fraction`. Sampling is deterministic and NPI-keyed: a hash of the NPI is compared to the fraction so that _all_ services for a sampled NPI are retained, never partial cuts of an NPI's history. This preserves within-NPI volume relationships, which is necessary for coverage estimation.

The claims grain is built by aggregating sampled service lines to the ten-column grain (§1), joined to NPPES for geography and taxonomy, with `subch_class` and `claim_class` computed per §2. Multiple POS codes within a setting, TOB rows, and `REPORT_DD` values in the source collapse into one grain row at this stage. Description fields (`PROCEDURE_CD_DESC`, `PROCEDURE_MODIFIER_DESC`, `PAYER_NAME`) are carried through with `ANY_VALUE` since they are 1:1 with their respective code columns.

#### Stage 2 — Coverage hierarchy

For each combination of

```
(BILLING_NPI_NBR, county, PROCEDURE_CD, channel, subch_class, claim_class, RATE_YEAR)
```

present in the claims grain, the pipeline assigns one coverage estimate by walking the tier hierarchy described in §6 (first match wins). The output is a row-level `coverage_rowmap` table on that key; projection joins it back to every claims row. A per-source summary (`coverage_cells`) feeds QA and the release notes.

#### Stage 3 — Projection and bootstrap

For each cell, the deterministic point estimate is `PROJECTED_UNITS = OBSERVED_UNITS / COVERAGE_ESTIMATE`, with `COVERAGE_ESTIMATE` clamped to `[min_coverage, max_coverage]`. Because `max_coverage = 1.0`, the implied factor `1 / COVERAGE_ESTIMATE` is in `[1.0, 1 / min_coverage]` and `PROJECTED_UNITS >= OBSERVED_UNITS` on every projectable row by construction.

Confidence intervals (`PROJECTED_UNITS_P5`, `PROJECTED_UNITS_P95`) are produced by a 500-draw parametric bootstrap that combines two noise sources:

* **Observation noise:** Negative Binomial around `OBSERVED_UNITS` with per-cell dispersion `k = mu / obs_phi` (default `obs_phi = 0.10`), i.e. variance `= mu × (1 + obs_phi)`. Relative SD falls as `1/sqrt(mu)` with a constant overdispersion multiplier. (V3 used a constant `k = 10`, which floored the relative SD at \~32% even for cells with hundreds of thousands of observed units and made P95 unrealistically wide on large cells.)
* **Coverage noise:** Lognormal around `COVERAGE_ESTIMATE`, parameterised so that `E[c] ≈ coverage_estimate` and `sd(c)/E[c] ≈ rel_sd`, then clipped to `[min_coverage, max_coverage]`.

The relative SD on the coverage draw is chosen by tier (see `coverage.coverage_rel_sd` in config): NPI-direct tiers 0.10, county-fallback tiers 0.15, outpatient 0.20, calibrated Commercial 0.25/0.35, MA-scaled 0.30, channel prior 0.50. The rel SD encodes our prior on how noisy each tier is. Per draw, `projected_draw = observed_draw / coverage_draw`. `PROJECTED_UNITS_P5` and `PROJECTED_UNITS_P95` are the 5th and 95th percentiles across the draws; the median is not published, since the deterministic point estimate already serves that role.

**Hard invariant: `PROJECTED_UNITS / P5 / P95 >= OBSERVED_UNITS`.** The deterministic point already satisfies this because `COVERAGE_ESTIMATE` is clamped to `<= max_coverage = 1.0`. The bootstrap percentiles can otherwise drift slightly below `OBSERVED_UNITS` when `COVERAGE_ESTIMATE` is at the 1.0 ceiling — lognormal coverage draws clipped at the ceiling are biased below 1.0, and the Negative Binomial observation noise is right-skewed for low counts. We therefore floor `PROJECTED_UNITS_P5` and `PROJECTED_UNITS_P95` at `OBSERVED_UNITS` after the bootstrap.

The bootstrap is parallelised: stages 1–2 are pure DuckDB SQL and use DuckDB's intra-query thread pool (`run.threads`); stage 3 dispatches projection batches to a `concurrent.futures.ProcessPoolExecutor` (`run.workers`). Per-batch RNGs are derived deterministically from `(seed, batch_idx)` via `numpy.random.SeedSequence`, so output is byte-identical regardless of worker completion order.

#### Stage 4 — QA

See §8.

### 5. Channel attribution

Each service line is attributed to one of three channels — `Commercial`, `Medicare`, or `Medicaid` — using the source claim's `PAYER_CHANNEL_NAME` directly. Non-projected channels (e.g. "Other", "Dual (Medicaid/Medicare)") are filtered out at ingest and do not appear in the output. Channel attribution is done at ingest and is not revisited downstream. Within the Medicare channel, `PAYER_SUBCHANNEL_NAME` drives the FFS / MA split used by the coverage tiers (§2).

### 6. Coverage hierarchy

The coverage estimate for a cell is selected by the first tier (top to bottom) that the cell qualifies for. All direct tiers require `observed_units >= direct_min_observed_units` (default 20) at the tier's own grain; county fallbacks additionally require `>= min_matched_npis` (default 3) distinct matched NPIs. Every tier records `raw_coverage` (the pre-clamp ratio); values materially above 1.0 signal a denominator mismatch and are surfaced in QA (§8).

**Trust filter.** A direct denominator implying raw coverage above `coverage.raw_trust_max` (default 1.5) is treated as evidence that the source does not measure that cell's universe — most commonly billing-org vs rendering-provider attribution mismatch against the CMS PUFs (PurpleLab attributes volume to the billing organization; CMS attributes to the rendering NPI, so an org billing for many individual renderers shows observed ≫ "truth"), or source-side suppression / vintage incompleteness. Such cells are _not_ clamped to 1.0; the pair is discarded and the cell falls through to the next tier. The same pair-level filter applies inside the matched-NPI county pools and the Commercial capture calibration, so a mismatched provider can neither anchor its own cells nor distort a pooled estimate. Raw coverage between 1.0 and the trust ceiling is treated as noise and clamps to 1.0 as before.

**T-MSIS completeness cap.** The latest T-MSIS year is typically preliminary; `references.tmsis_max_complete_year` (default 2023 in the shipped config) caps the denominator year below the file's maximum. Set it to null to use the newest data and compare the per-year raw-coverage distributions in QA before trusting it.

**The estimand.** Coverage means "the share of this provider's true volume that the source feed captured." Capture is a property of the provider's claims-routing (which clearinghouses feed the source), so it varies more across providers than across payers or procedures. The hierarchy therefore measures capture at the provider grain wherever administrative truth names the provider, and generalizes outward only when it must.

**Matched-NPI county fallbacks.** Where the denominator source is NPI-grained (T-MSIS, Physician PUF, Inpatient PUF), the county fallback is a _matched_ ratio-of-sums: both numerator and denominator are restricted to NPIs present on both sides for that procedure × year. This removes the asymmetry of comparing a Type-2-only numerator against an all-provider denominator (which biased coverage low and over-projected), and makes the county estimate "pooled capture among verifiable providers," which is then applied to the county's unmatched providers.

#### 6.1 `tmsis_npi_direct` _(Medicaid)_

Applied when the billing NPI × HCPCS × year exists in T-MSIS.

```
COVERAGE_ESTIMATE = observed_claims(npi, hcpcs, year) / tmsis_claim_lines(npi, hcpcs, min(year, latest_tmsis_year))
```

`CONFIDENCE_BUCKET` is `high` if `observed_units >= high_confidence_min_units` (default 100), otherwise `medium`. T-MSIS is treated as administrative truth for the Medicaid universe up to CMS suppression rules (cells with fewer than 11 beneficiaries are suppressed at source) and state-by-state managed-care encounter completeness (see §10).

#### 6.2 `tmsis_direct` _(Medicaid, county fallback)_

Matched-NPI ratio-of-sums at county × HCPCS × year, applied to Medicaid cells whose NPI is not in T-MSIS for that HCPCS × year.

#### 6.3 `medicare_ffs_npi_direct` _(Medicare FFS, professional)_

Applied to Medicare FFS-subchannel professional cells when the billing NPI × HCPCS exists in the CMS Physician PUF. The PUF is a single-vintage annual file; each rate year's numerator is compared against the same denominator (documented drift for years beyond the vintage).

#### 6.4 `medicare_inpatient_npi_direct` _(Medicare FFS, inpatient)_

Applied to Medicare FFS institutional-inpatient (MS-DRG) cells when the billing NPI × DRG exists in the CMS Inpatient PUF. Ratio: observed claims vs discharges.

#### 6.5 `medicare_ffs_county_benchmark` _(Medicare FFS, professional county fallback)_

Matched-NPI ratio-of-sums at county × HCPCS × year against the Physician PUF.

#### 6.6 `direct_inpatient_benchmark` _(Medicare FFS, inpatient county fallback)_

Matched-NPI ratio-of-sums at county × DRG × year against the Inpatient PUF.

#### 6.7 `medicare_outpatient_benchmark` _(Medicare FFS, outpatient)_

Applied to Medicare FFS institutional-outpatient cells whose HCPCS maps to an APC (OPPS Addendum B, payable status indicators only). Observed units roll up to county × APC and are compared against CMS Outpatient PUF `CAPC_Srvcs` (CCN-grain, mapped to county via POS Facilities). The source is CCN-grain, so no matched-NPI restriction is possible; this is a plain ratio-of-sums benchmark.

#### 6.8 `ma_scaled_benchmark` _(Medicare Advantage)_

MA has no public per-provider denominator. Where a county FFS denominator exists (professional, inpatient, or outpatient), the MA universe is estimated as

```
ma_denominator = ffs_denominator_county × ma_rate / (1 − ma_rate)
```

with `ma_rate` the county MA participation rate from the CMS Geographic Variation PUF (bounded to \[0.05, 0.85]). This assumes similar per-enrollee utilization across FFS and MA within a county — a documented approximation (§10). The FFS denominator here is the _full-county_ total (all provider types), since the MA universe is not restricted to providers we can match. `CONFIDENCE_BUCKET` is `medium`; bootstrap rel SD 0.30.

#### 6.9 `commercial_calibrated` / `commercial_calibrated_pooled` _(Commercial)_

Capture rate is a property of the provider's clearinghouse footprint, not the payer. Where Medicare FFS truth exists for an NPI, we measure the feed's capture directly:

```
capture(npi) = Σ observed_ffs_units(npi, hcpcs) / Σ puf_tot_srvcs(npi, hcpcs)
```

summed over matched HCPCS, restricted to the claims rate year closest to the PUF vintage (`references.cms_puf_year`), and gated on `Σ puf_tot_srvcs >= min_puf_services` (default 100) and `>= min_matched_codes` (default 3) matched codes. The clamped capture is applied as the coverage estimate for **all of that NPI's Commercial cells**. NPIs that fail the gates fall to a state × taxonomy pooled capture (requires `>= pooled_min_npis` calibrated NPIs, default 25), then to the channel prior.

The payer-agnostic-capture assumption is the tier's load-bearing beam. It is monitored, not presumed: the QA report publishes the capture distribution and the implied shift vs the 0.60 prior every release (§8). `CONFIDENCE_BUCKET` is `medium` (NPI-level) / `low` (pooled).

#### 6.10 `channel_prior`

Applied to every remaining cell whose pooled signal (county × procedure × channel × year; NPI-pooled when the county is unknown) is at least `peer_min_observed_units` (default 5). The denominator is the per-channel coverage prior from `data/seed/channel_priors.csv`:

| Channel    | `pl_coverage_prior` | Implied uplift |
| ---------- | ------------------- | -------------- |
| Commercial | 0.60                | 1.67×          |
| Medicare   | 0.75                | 1.33×          |
| Medicaid   | 0.80                | 1.25×          |

`CONFIDENCE_BUCKET` is `low`. The priors are derived from PurpleLab's published payer-mix vs all-payer denominators and are reviewed each release.

#### 6.11 `suppress`

Cells below the signal threshold with no administrative-truth denominator are not projected. They appear in the output with `PROJECTED_UNITS / P5 / P95` all NULL, `COVERAGE_SOURCE = 'suppress'`, and `CONFIDENCE_BUCKET = 'suppress'`. Downstream consumers can decide whether to filter them or pass them through.

### 7. Output schema

Output is partitioned parquet at `output_root/state=XX/year=YYYY/part-*.parquet`, written with zstd compression. Each row corresponds to one cell at the projection grain (see §1). The column set is unchanged from V3; `COVERAGE_SOURCE` and `COVERAGE_GRAIN` carry the new tier vocabulary of §6.

| Column                             | Type              | Description                                              |
| ---------------------------------- | ----------------- | -------------------------------------------------------- |
| `BILLING_NPI_NBR`                  | int64             | Rendering / billing provider NPI                         |
| `FACILITY_NPI_NBR`                 | int64 (nullable)  | Service-facility NPI when present on the source claim    |
| `PROCEDURE_CD`                     | string            | HCPCS / CPT code (or MS-DRG for inpatient rows)          |
| `PROCEDURE_CD_DESC`                | string            | PurpleLab's short description for `PROCEDURE_CD`         |
| `PROCEDURE_MODIFIER`               | string (nullable) | Procedure modifier; NULL if none                         |
| `PROCEDURE_MODIFIER_DESC`          | string (nullable) | Description for `PROCEDURE_MODIFIER`                     |
| `PAYER_ID`                         | int64             | Source payer identifier                                  |
| `PAYER_NAME`                       | string            | Payer name as reported in PurpleLab                      |
| `PAYER_SUBCHANNEL_NAME`            | string            | Subchannel within `PAYER_CHANNEL_NAME`                   |
| `PAYER_CHANNEL_NAME`               | string            | `Commercial` \| `Medicare` \| `Medicaid`                 |
| `CLAIM_TYPE_CD`                    | string            | `P` \| `I`                                               |
| `SETTING`                          | string            | `Office` \| `Outpatient` \| `Inpatient` \| `Other`       |
| `OBSERVED_UNITS`                   | float64           | Source-claims unit count                                 |
| `PROJECTED_UNITS`                  | float64           | `OBSERVED_UNITS / COVERAGE_ESTIMATE`                     |
| `PROJECTED_UNITS_P5`               | float64           | 5th percentile (bootstrap), floored at `OBSERVED_UNITS`  |
| `PROJECTED_UNITS_P95`              | float64           | 95th percentile (bootstrap), floored at `OBSERVED_UNITS` |
| `ATTRIBUTED_CLAIM_COUNT`           | int64             | Source-claims claim count                                |
| `ATTRIBUTED_CLAIM_COUNT_PROJECTED` | float64           | `ATTRIBUTED_CLAIM_COUNT / COVERAGE_ESTIMATE`             |
| `ATTRIBUTED_CHARGE_AMT`            | float64           | Source-claims billed-charge dollars                      |
| `ATTRIBUTED_CHARGE_AMT_PROJECTED`  | float64           | `ATTRIBUTED_CHARGE_AMT / COVERAGE_ESTIMATE`              |
| `COVERAGE_ESTIMATE`                | float64           | Coverage fraction used to scale observed → projected     |
| `COVERAGE_SOURCE`                  | string            | Tier name (see §6)                                       |
| `COVERAGE_GRAIN`                   | string            | Grain + ratio metric of the coverage estimate            |
| `CONFIDENCE_BUCKET`                | string            | `high` \| `medium` \| `low` \| `suppress`                |
| `PROVIDER_COUNTY_FIPS`             | string            | 5-digit county FIPS (from NPPES)                         |
| `PROVIDER_STATE`                   | string            | Two-letter state (from NPPES)                            |
| `DATA_PERIOD_START`                | date32            | Min `REPORT_DD` over the cell                            |
| `DATA_PERIOD_END`                  | date32            | Max `REPORT_DD` over the cell                            |
| `CALIBRATION_VINTAGE`              | string            | Run identifier from `config.yaml`                        |
| `RATE_YEAR`                        | int64             | Source `RATE_YEAR`                                       |

Suppressed cells pass through with the three observed columns set, the projected columns NULL, and `COVERAGE_SOURCE = 'suppress'`.

#### 7.1 Joining the projection onto the source claims feed

The projection grain is a strict subset of the source PurpleLab grain (we collapse POS, TOB, and `REPORT_DD`; we keep everything else). This means the projection table is unique on the ten join keys

```
(BILLING_NPI_NBR, FACILITY_NPI_NBR, PROCEDURE_CD, PROCEDURE_MODIFIER,
 PAYER_ID, PAYER_SUBCHANNEL_NAME, PAYER_CHANNEL_NAME,
 CLAIM_TYPE_CD, SETTING, RATE_YEAR)
```

and a `LEFT JOIN` from the source to the projection on those keys returns one row per source row, with the projection columns populated when the cell was projectable and NULL when it falls through `suppress` or is outside the projection universe (out-of-scope channel, NPI not in NPPES, etc.).

What this does **not** mean: it does **not** mean projected values are distributed across source rows. The same `PROJECTED_*` value will appear repeated once for every source row that maps to the same cell. Consumers should be aware of two consequences:

* **Do not `SUM(PROJECTED_*)` over a joined source × projection result without first deduplicating to the projection grain.** Doing so over-counts the projection by the source-row multiplicity (typically 2–5×).
* **Do aggregate the source to the projection grain before joining**, or aggregate the joined result by the projection grain with an idempotent aggregate (`MAX`), if a single projection value per cell is what you want.

Suppress rows can be elided by filtering `COVERAGE_SOURCE != 'suppress'`.

### 8. QA and validation

The QA stage emits a JSON report with every release:

1. **Coverage source mix** — cells, observed units and claims per tier × confidence bucket.
2. **`raw_coverage` distribution per tier** — p50/p90/p99 plus counts above 1.0 and above `qa.raw_coverage_flag` (default 1.2). Raw coverage materially above 1.0 means the denominator is too small for that cell (year drift, source suppression, or a metric mismatch); the top 100 offending cells are listed. This check would have caught all three of the V3 coverage bugs (§12) and is the first thing to review each release.
3. **Medicare subchannel classification audit** — observed volume by `PAYER_SUBCHANNEL_NAME` × class; any volume classified `other` (matching neither configured list) is called out, since those rows only qualify for the channel prior.
4. **Commercial calibration report** — per-NPI capture distribution (p10–p90, mean, share with raw capture > 1), Commercial volume by calibration source, and the implied shift vs the 0.60 prior.
5. **Medicare FFS cross-check.** For every (state × procedure) with both FFS-subchannel projections and a CMS Physician PUF total, the relative delta `(projected − puf) / puf` — restricted to pairs with `puf_total_services > qa.puf_min_services` (default 1000) and `>= qa.puf_min_npis` (default 10) reporting NPIs, which removes the suppression-floor artifact documented in V3's §8.1. The comparison is FFS-only on both sides, removing the MA-contamination artifact. Median absolute delta and flagged pairs (`|delta| > systematic_delta_flag`, default 0.30) are reported.
6. **Per-NPI envelope.** Projected Medicare FFS professional volume per NPI vs the provider-level PUF total; NPIs whose ratio exceeds `qa.npi_envelope_flag` (default 3.0) are surfaced. This automates the manual "top 50 NPIs" spot check.

Manual checks before each release: review the raw\_coverage flags, the calibration distribution, all flagged cross-check pairs, and the envelope list.

### 9. Reference run results

V4.1 reference run, calibration vintage `20260707`, full PurpleLab feed (`CLAIMS_ORDERED_202604` extract), executed 2026-07-07. Capture reference year 2024; T-MSIS denominator year 2023; CMS PUF vintage 2023; Commercial calibration in shadow mode.

#### 9.1 Headline

| Metric         | Observed           | Projected           | Uplift |
| -------------- | ------------------ | ------------------- | ------ |
| Units          | 67,204,206,939     | 138,933,111,244     | 2.067× |
| Claims         | 11,149,227,303     | 22,211,793,674      | 1.992× |
| Billed charges | $8,564,064,338,063 | $15,137,545,321,932 | 1.768× |

Output rows: 380,593,164 across 485,512 active billing NPIs (28.45% of Type-2 NPIs in NPPES). The three uplifts differ because coverage varies by cell and the three volumes are distributed differently across cells: charge dollars concentrate in better-covered cells (institutional, Medicare FFS), units in Medicaid drug/service codes where T-MSIS-measured capture is lowest.

The V3 run on the same input produced a 1.408× unit uplift. The increase to 2.067× is the cumulative effect of the V4 bug fixes (all three biased projections low — year pooling, unit-metric mismatches, denominator asymmetry) plus measured denominators replacing optimistic priors: T-MSIS-measured Medicaid capture (\~0.20–0.33 at the NPI grain) is far below the old 0.80 prior, and measured Medicare FFS state capture (0.33–0.63 across states on metric-consistent codes) is below the old 0.75 prior.

#### 9.2 Coverage source mix

Shares are of observed units; uplifts are per-tier projected/observed.

| Coverage source                 | Output rows | Share of observed units | Unit uplift | Claim uplift |
| ------------------------------- | ----------- | ----------------------- | ----------- | ------------ |
| `channel_prior`                 | 293,023,232 | 75.51%                  | 1.499×      | 1.552×       |
| `tmsis_npi_direct`              | 15,798,410  | 11.42%                  | 5.070×      | 3.113×       |
| `medicare_ffs_state_benchmark`  | 32,643,755  | 9.23%                   | 2.063×      | 2.123×       |
| `tmsis_direct`                  | 18,584,470  | 3.42%                   | 4.301×      | 4.554×       |
| `ma_scaled_benchmark`           | 4,591,363   | 0.37%                   | 4.914×      | 4.345×       |
| `medicare_outpatient_benchmark` | 955,106     | 0.01%                   | 1.722×      | 1.724×       |
| `direct_inpatient_benchmark`    | 326,241     | <0.01%                  | 2.984×      | 2.984×       |
| `medicare_inpatient_npi_direct` | 58,364      | <0.01%                  | 1.785×      | 1.785×       |
| `suppress`                      | 14,612,223  | 0.03%                   | —           | —            |

Measured (non-prior) tiers anchor 24.5% of observed units, up from 19.5% in V3, and the Medicare FFS professional book moved from the national prior to per-state measured capture.

#### 9.3 QA outcomes

| Check                                                                       | Result                            | Disposition                                                                                          |
| --------------------------------------------------------------------------- | --------------------------------- | ---------------------------------------------------------------------------------------------------- |
| Medicare FFS totals alignment (`medicare_ffs_totals_ratio_by_state_median`) | 1.000                             | Pass by construction (trimmed-pooled state benchmark); this is the market-sizing pass bar            |
| Medicare FFS vs PUF signed median delta (professional, reference year)      | +0.082                            | Pass (bar ±0.10); residual is 2024-obs vs 2023-PUF vintage growth                                    |
| Direct FFS capture measurement (`observed_over_puf`, pair median)           | 0.503                             | Consistent with the applied state captures (0.33–0.63)                                               |
| Commercial calibration                                                      | shadow                            | Capture distribution reported in QA; Commercial projects at the 0.60 prior pending a clean yardstick |
| Unclassified Medicare subchannel volume                                     | 212M units (\~0.2% of Medicare)   | `Medicare / Unspecified` + `Part D`; projects at the channel prior by design                         |
| raw\_coverage trust filter                                                  | 0 state-benchmark cells above 1.0 | Pass                                                                                                 |
| Coverage assignment completeness                                            | 100% of keys assigned a tier      | Pass                                                                                                 |

Pair-level absolute deltas in the FFS cross-check remain wide (median \~0.45): capture heterogeneity across procedures is real, and a state-scalar coverage cannot remove it. The check's role across releases is drift detection on the signed median and the totals ratio.

### 10. Limitations

* **Coverage is a model, not a measurement.** Direct tiers are subject to source-side suppression, vintage drift (single-vintage PUF denominators applied to later rate years; T-MSIS capped at its latest year), and reporting completeness. We bound the coverage estimate to `[0.05, 1.00]`, surface confidence labels, and publish raw-coverage diagnostics every release.
* **T-MSIS managed-care completeness varies by state.** States with weak encounter reporting under-state the Medicaid denominator, biasing coverage high and projections low in those states. The raw\_coverage distribution by state is the monitoring tool; a state-level completeness adjustment is future work.
* **The latest T-MSIS year is preliminary.** Rate years beyond the latest complete T-MSIS year reuse the latest available denominator.
* **MA scaling assumes FFS-like per-enrollee utilization.** The `ma_scaled_benchmark` denominator is an enrollment-scaled FFS universe; MA plan-mix and utilization-management differences are not modeled. The tier carries a wide bootstrap rel SD (0.30) and `medium` confidence for this reason.
* **Commercial calibration assumes payer-agnostic capture.** Providers whose Commercial claims route differently from their Medicare FFS claims violate the assumption. The QA capture distribution and the pooled-vs-NPI comparison monitor this; the tier can be disabled per release via `coverage.commercial_calibration.enabled`.
* **The projection is per-NPI, not per-patient.** We do not deduplicate patients across NPIs.
* **The projection universe is Type 2 billing NPIs.** Claims billed under individual (Type 1) NPIs are excluded at ingest. County-fallback coverage is estimated on matched (verifiable) NPIs and generalized to unmatched Type 2 NPIs in the county; it is an estimate of capture among the published universe, not of the all-provider universe. Consumers should not read county sums of projections as "all services in the county."
* **Channel attribution depends on `PAYER_CHANNEL_NAME` / `PAYER_SUBCHANNEL_NAME` in the source feed.** Unclassified Medicare subchannels project at the channel prior and are itemized in QA.

### 11. Reproducibility

Every release is reproducible from a single config file (`config.yaml`) plus the reference data snapshot under `data/reference/`. The config records the source claims vintage, NPPES vintage, per-reference vintages, bootstrap seed/draws, `obs_phi`, all tier thresholds and bounds, the Medicare subchannel lists, and the calibration gates. A given combination of inputs and config produces byte-identical output, modulo parquet metadata timestamps.

### 12. Changelog

#### V4.1 — July 2026

Corrections from the first V4 full-population run, where the (now per-year, FFS-only) QA cross-check plus a direct observed-vs-PUF measurement showed Medicare FFS over-projection concentrated in the Physician-PUF NPI-matched tiers:

* **Reference-year capture semantics.** Capture is a rate: it is measured once at a complete reference claims year (`coverage.capture_reference_year`) and applied to every rate year. Per-year ratios conflated capture with year completeness and silently annualized partial rate years (the 2026 partial year projected at \~5× observed). Measured FFS capture was stable across complete years (NJ: 0.778 in 2024, 0.776 in 2025), validating the rate treatment.
* **`medicare_ffs_state_benchmark` replaces the per-NPI / county Physician-PUF tiers.** Cross-role NPI matching (billing org ↔ rendering individual) is unreliable in both directions; the trust filter catches the high side, but fragmented pairs survive with falsely low coverage (the V4.0 county benchmark applied 5.08× in NJ where measured truth was 1.29×). State capture is the **trimmed pooled ratio** `Σ observed / Σ PUF` over (state × procedure) pairs gated on a PUF service floor and a minimum reporting-NPI count, trimmed of metric-inconsistent pairs (`observed/PUF > raw_trust_max`, code families whose unit definitions differ from `Tot_Srvcs`). Pooling makes projected state totals equal PUF totals on validatable codes **by construction** — the property market sizing needs. Two rejected intermediates, for the record: an untrimmed pooled ratio was inflated toward 1.0 by unit-mismatched codes (under-projected), and a median-of-pairs scalar aligned the median pair but over-projected totals \~18% in the median state because capture correlates with code volume.
* **Commercial calibration moved to shadow mode** (`commercial_calibration.apply: false`). The per-NPI capture yardstick shares the Physician PUF's attribution-fragmentation problem (capture p50 0.085 vs measured aggregate 0.78), so applied calibration over-projects Commercial. Capture is still computed and reported in QA each release; Commercial projects at the channel prior until a clean yardstick exists (candidate: restrict calibration pairs to NPIs whose FFS matching is aggregate-coherent).
* **QA cross-check fixed to the reference year and professional claims.** V4.0 summed projected volume across all rate years and all claim classes against a single-year professional-only PUF, fabricating deltas of roughly (n\_years − 1) × 100% plus the institutional share. The check now compares professional claims at the reference year only, and reports `observed_over_puf` (the direct capture measurement), the signed median delta (bias), and `medicare_ffs_totals_ratio_by_state_median` (per-state Σprojected/ΣPUF over validatable pairs — the market-sizing pass bar, ≈1.0 by construction).
* Stage 2 restructured into sequential per-tier assignments with per-statement timing (the V4.0 single 10-way join was pathologically slow at 380M rows); QA reads `coverage_rowmap` instead of re-joining the claims grain; `scripts/qa_from_output.py` can rebuild the QA report from a written output dataset.

#### V4 — July 2026

Coverage correctness release plus four new administrative denominators. Three bugs fixed, all of which biased projections **low**:

1. **Per-rate-year coverage.** V3 pooled every rate year of observed volume into one numerator and divided by single-year denominators, inflating coverage by roughly the number of years in the feed and clamping many direct-tier cells to 1.0 (no projection). V4 keys coverage by `RATE_YEAR` and year-matches T-MSIS denominators (capped at the latest available year).
2. **Like-for-like ratio metrics.** V3 divided unit counts by T-MSIS claim _lines_ and CMS _discharges_. Units-per-line for drug J-codes run into the hundreds, so raw coverage blew through 1.0 and the clamp silently disabled projection on exactly the high-dollar cells. V4 uses claims-vs-lines (T-MSIS), claims-vs-discharges (inpatient), and units-vs-Tot\_Srvcs (Physician PUF), recorded per tier in `COVERAGE_GRAIN`.
3. **Matched-NPI county denominators.** V3 compared a Type-2-only numerator against all-NPI county denominators (T-MSIS build had no entity-type filter), biasing coverage low and over-projecting Medicaid. V4 county fallbacks restrict both sides to matched NPIs.

New coverage tiers (see §6): `tmsis_npi_direct`, `medicare_ffs_npi_direct`, `medicare_inpatient_npi_direct`, `medicare_ffs_county_benchmark`, `medicare_outpatient_benchmark`, `ma_scaled_benchmark`, `commercial_calibrated` (+ pooled). The V3 tier names `tmsis_direct` and `direct_inpatient_benchmark` are retained for the county fallbacks with corrected math.

Other changes:

* Trust filter on direct denominators (`coverage.raw_trust_max`; §6): cells whose yardstick implies >150% capture fall through instead of clamping to 1.0, and the mismatched pairs are excluded from county pools and the Commercial calibration. Added after the first V4 smoke run showed billing-vs-rendering attribution mismatch on PUF-matched org NPIs (raw coverage p90 ≈ 3.8 on the professional NPI tier) and T-MSIS 2024 incompleteness (46% of NPI-tier cells above raw 1.0); the same run motivated `references.tmsis_max_complete_year` (default 2023).
* Observation-noise dispersion now scales with cell size (`obs_phi`; §4 Stage 3), replacing the constant `k = 10` that floored relative SD at \~32%.
* `raw_coverage` is retained per cell and reported in QA; cells above `qa.raw_coverage_flag` are surfaced as probable denominator mismatches.
* The Medicare-vs-PUF QA cross-check is now FFS-subchannel-only and scope-restricted (puf services / NPI floors), implementing the refinement deferred in V3 §8.1. A per-NPI envelope check against the provider-level PUF automates the manual top-50 review.
* Channel-prior gating no longer drops NPIs without an NPPES county (they gate on their own pooled volume instead of silently suppressing).
* Reference grains changed: T-MSIS to NPI × HCPCS × year parquet, CMS Inpatient to NPI × DRG; download scripts updated accordingly (`aggregate_to_npi_year` / `aggregate_to_npi_drg` replace the county aggregators, whose contracts they superseded). New download scripts: `download_geo_variation.py`, `download_outpatient_opps.py`, `download_physician_provider.py`. Config keys `references.tmsis_medicaid_npi_parquet` / `references.cms_inpatient_npi_csv` replace the county-CSV keys.
* Synthetic end-to-end test harness (`tests/synthetic/`, `tests/test_synthetic_e2e.py`) with hand-computed expected values for every tier.

#### V3 — April 2026

Architectural simplification. Removed the population-benchmark and peer-group infrastructure (ACS/KFF/MEPS/HCUPnet denominators, Bayesian shrinkage) after the pre-customer review identified a 99213/99214 Commercial double-count bug in that scaffold. Coverage hierarchy reduced to four tiers: `tmsis_direct` → `direct_inpatient_benchmark` → `channel_prior` → `suppress`. At V3 release the two removed tiers carried under 2% of observed units while requiring six external datasets.

#### V2 — April 2026

Added the two administrative-truth coverage tiers (`tmsis_direct`, `direct_inpatient_benchmark`), the reference-data orchestrator, and the parallelised bootstrap.

#### V1 — March 2026

Initial release. Coverage hierarchy: `direct_pop_benchmark` → `peer_group` → `channel_prior` → `suppress`.
