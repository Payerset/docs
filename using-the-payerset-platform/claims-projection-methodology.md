# Claims Projection Methodology

This document describes how Payerset estimates the universe-level claim volume that each rendering provider performs, given a partial sample of observed claims. It is intended as a transparent, auditable account of every input, transformation, and assumption that produces the projected unit counts published in our APIs and dashboards.

### 1. What we project

Each row in our output represents a single cell at the source-claims grain:

```
BILLING_NPI_NBR × PROCEDURE_CD × PAYER_ID × PAYER_CHANNEL_NAME × POS_CD × RATE_YEAR
```

For every cell we publish:

* `OBSERVED_UNITS` — the count we directly observed in the source claims feed.
* `PROJECTED_UNITS` — our estimate of the _true_ count of services rendered by that NPI in that cell, including services that fell outside our sample. This is the median of a 500-draw parametric bootstrap.
* `PROJECTED_UNITS_P5`, `PROJECTED_UNITS_P95` — the 5th and 95th percentile of the same bootstrap.
* `COVERAGE_ESTIMATE` — the implied share of the universe captured by `OBSERVED_UNITS`. Bounded to `[0.05, 1.00]`.
* `COVERAGE_SOURCE` — which tier of the hierarchy produced the estimate (see §6).
* `COVERAGE_GRAIN` — the grain at which the coverage fraction was estimated (e.g. `county|procedure|medicaid_universe`).
* `CONFIDENCE_BUCKET` — `high`, `medium`, `low`, or `suppress`.

The point estimate is `PROJECTED_UNITS = OBSERVED_UNITS / COVERAGE_ESTIMATE`. The bootstrap adds observation noise (Negative Binomial around `OBSERVED_UNITS`) and coverage noise (lognormal around `COVERAGE_ESTIMATE`) to produce the percentile bounds.

#### 1.1 What `OBSERVED_UNITS` is — and what it is not

`OBSERVED_UNITS` is the unit count from the **source claims feed (PurpleLab) only**. It is not a pooled or deduplicated combination of multiple data sources. The pipeline ingests several other datasets, but each plays a different role and none of them adds to `OBSERVED_UNITS`:

* The provider directory (NPPES) attaches geography and taxonomy to each NPI.
* T-MSIS Medicaid Provider Spending is the denominator for the `tmsis_direct` tier on Medicaid cells.
* The CMS Medicare Inpatient (by Provider and Service) PUF is the denominator for the `direct_inpatient_benchmark` tier on Medicare-channel inpatient (MS-DRG) cells.
* The CMS Physician PUF is used only as a QA cross-check and never enters the projection math.

The reference datasets shape `COVERAGE_ESTIMATE`, which determines how `OBSERVED_UNITS` is scaled to a universe estimate. `OBSERVED_UNITS` itself is always a single-source quantity. This distinction matters when reconciling counts: any reader who tries to add observed PurpleLab units to T-MSIS or PUF totals is double-counting.

#### 1.2 Which PurpleLab field underlies `OBSERVED_UNITS`

The PurpleLab `CLAIMS_ORDERED` feed exposes two volume fields per row:

* **`ATTRIBUTED_TOTAL_UNITS`** — units on the original-claim grain, attributed to the billing NPI on that row. Populated for \~94% of professional rows and \~97% of inpatient rows.
* **`COUNT_OF_UNITS`** — units sourced from a matched remittance advice. Only populated for \~13–16% of rows (the remit-matched subset).

We sum `ATTRIBUTED_TOTAL_UNITS` because it reflects the full claim population in the feed. Rows where `ATTRIBUTED_TOTAL_UNITS` is null are remits without a matched open claim and are skipped — they have no billing-NPI grain row to attribute to. This is implemented in `stages/ingest.py`.

### 2. Inputs

| Input              | Source                     | Grain                                                                                                               | Notes                                                                                                                      |
| ------------------ | -------------------------- | ------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| Source claims      | PurpleLab `CLAIMS_ORDERED` | one row per `BILLING_NPI_NBR × PROCEDURE_CD × PAYER_ID × PAYER_CHANNEL_NAME × POS_CD × RATE_YEAR` after aggregation | Includes professional and inpatient (`CLAIM_TYPE_CD = 'P' / 'I'`). Volume sourced from `ATTRIBUTED_TOTAL_UNITS` (see §1.2) |
| Provider directory | Enriched NPPES (Type 2)    | one row per organization NPI                                                                                        | Includes county FIPS, primary taxonomy, CBSA, lifetime claim count                                                         |

Claims are joined to the provider directory on rendering NPI to attach county, primary taxonomy, and CBSA before any aggregation.

### 3. Reference data sources

The projection requires two administrative-truth denominators (T-MSIS Medicaid, CMS Medicare Inpatient FFS), one channel-level prior (the seed CSV), and one cross-check (CMS Physician PUF). All projection-side references are normalized to a county × procedure key so they can be joined to the claims grain.

| Reference                                          | Provider                                                                                                                                                  | Used for                                                                                                                          | Required                                      |
| -------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------- |
| Channel priors                                     | Payerset seed (`data/seed/channel_priors.csv`)                                                                                                            | Per-channel coverage prior (Commercial 0.60, Medicare 0.75, Medicaid 0.80)                                                        | Yes                                           |
| T-MSIS Medicaid Provider Spending                  | CMS T-MSIS Analytic Files                                                                                                                                 | Medicaid units by billing NPI × HCPCS, aggregated to county × procedure. Denominator for `tmsis_direct`.                          | No (falls through to channel prior if absent) |
| CMS Medicare Inpatient (by Provider & Service) PUF | CMS — see [Payerset Medicare Inpatient methodology](https://docs.payerset.com/using-the-payerset-platform/data-dictionary/medicare-inpatient-methodology) | Medicare FFS Part A discharges by provider × MS-DRG, aggregated to county × MS-DRG. Denominator for `direct_inpatient_benchmark`. | No (falls through to channel prior if absent) |
| CMS Physician & Other Practitioners PUF            | CMS                                                                                                                                                       | NPI × HCPCS Medicare FFS reference. Used only for the QA cross-check (§8).                                                        | No (QA only)                                  |

All references are versioned and stored under `data/reference/` with the source URL, retrieval date, and any source-side suppression rules captured in `DOWNLOADS.md`.

### 4. Pipeline stages

The end-to-end run executes four ordered stages plus a reference-load step. All stages share a single in-process DuckDB connection; intermediate outputs are materialized as DuckDB tables, not files.

#### Stage 0 — Reference loads

Channel priors, the CMS Physician PUF, the T-MSIS Medicaid aggregate, and the CMS Medicare Inpatient aggregate are registered as DuckDB views. T-MSIS and CMS Inpatient are optional: when the reference file is absent, the loader registers an empty view and the corresponding tier produces zero rows.

#### Stage 1 — Ingest and geography

The source parquet is filtered to the configured states (if any) and the configured `sample_fraction`. Sampling is deterministic and NPI-keyed: a hash of the NPI is compared to the fraction so that _all_ services for a sampled NPI are retained, never partial cuts of an NPI's history. This preserves within-NPI volume relationships, which is necessary for coverage estimation.

The claims grain is built by aggregating sampled service lines to `BILLING_NPI_NBR × PROCEDURE_CD × PAYER_ID × PAYER_CHANNEL_NAME × POS_CD × RATE_YEAR`, joined to NPPES for geography and taxonomy. Multiple modifier / TOB rows in the source collapse into one grain row at this stage.

#### Stage 2 — Coverage hierarchy

For each `(provider_county_fips, procedure_cd, payer_channel_name)` combination present in the claims grain, the pipeline assigns one coverage estimate by walking the four-tier hierarchy described in §6. The output is a single `coverage_cells` table keyed on that triple; downstream projection joins this back to every claim row.

#### Stage 3 — Projection and bootstrap

For each cell, `PROJECTED_UNITS = OBSERVED_UNITS / COVERAGE_ESTIMATE`, with `COVERAGE_ESTIMATE` clamped to `[min_coverage, max_coverage]`. Confidence intervals are produced by a 500-draw parametric bootstrap that combines two noise sources:

* **Observation noise:** Negative Binomial around `OBSERVED_UNITS` with dispersion `k = 10` (variance = mu + mu²/k).
* **Coverage noise:** Lognormal around `COVERAGE_ESTIMATE`, parameterised so that `E[c] ≈ coverage_estimate` and `sd(c)/E[c] ≈ rel_sd`, then clipped to `[min_coverage, max_coverage]`.

The relative SD on the coverage draw is chosen by tier:

| Coverage source              | Relative SD |
| ---------------------------- | ----------- |
| `tmsis_direct`               | 0.10        |
| `direct_inpatient_benchmark` | 0.10        |
| `channel_prior`              | 0.50        |

The relative SD encodes our prior on how noisy each tier is: a Medicaid cell whose denominator comes directly from T-MSIS is treated as much tighter than a cell that fell through to a channel-wide prior. Per draw, `projected_draw = observed_draw / coverage_draw`. The 5th, 50th, and 95th percentiles across 500 draws become `PROJECTED_UNITS_P5`, `PROJECTED_UNITS` (median), and `PROJECTED_UNITS_P95`.

The bootstrap is parallelised: stages 1–2 are pure DuckDB SQL and use DuckDB's intra-query thread pool (`run.threads`); stage 3 dispatches projection batches to a `concurrent.futures.ProcessPoolExecutor` (`run.workers`). Per-batch RNGs are derived deterministically from `(seed, batch_idx)` via `numpy.random.SeedSequence`, so output is byte-identical regardless of worker completion order.

#### Stage 4 — QA

The QA stage emits a JSON report containing:

* The coverage source mix: cells and observed units by tier.
* For Medicare cells with both PurpleLab observations and CMS Physician PUF reference values, the top systematic deltas at the (state × procedure) grain. Pairs with `|delta| > systematic_delta_flag` (default 0.30) are surfaced. Known artifacts of this check are documented in §8.1.

### 5. Channel attribution

Each service line is attributed to one of three channels — `Commercial`, `Medicare`, or `Medicaid` — using the source claim's `PAYER_CHANNEL_NAME` directly. Non-projected channels (e.g. "Other", "Dual (Medicaid/Medicare)") are filtered out at ingest and do not appear in the output. Channel attribution is done at ingest and is not revisited downstream.

### 6. Coverage hierarchy

The coverage estimate for a cell is selected by the first tier (top to bottom) that the cell qualifies for. Tiers are evaluated in the order shown.

#### 6.1 `tmsis_direct` _(Medicaid only)_

Applied when the cell is Medicaid and a matching county × procedure row exists in the T-MSIS Medicaid Provider Spending aggregate. The denominator is the Medicaid units for that county × procedure as reported by the state Medicaid agencies via T-MSIS.

```
COVERAGE_ESTIMATE = OBSERVED_UNITS / medicaid_units_county_procedure
```

Bounded to `[min_coverage, max_coverage]`. Fires only when `OBSERVED_UNITS ≥ direct_min_observed_units` (default 20). `CONFIDENCE_BUCKET` is `high` if `OBSERVED_UNITS ≥ high_confidence_min_units` (default 100), otherwise `medium`. T-MSIS is treated as administrative truth for the Medicaid universe up to CMS suppression rules (cells with fewer than 11 beneficiaries are suppressed at source).

#### 6.2 `direct_inpatient_benchmark` _(Medicare only)_

Applied when the cell is Medicare and a matching county × MS-DRG row exists in the CMS Medicare Inpatient Hospitals (by Provider and Service) aggregate. The denominator is Medicare FFS Part A discharges for that county × DRG, aggregated from per-provider counts via the rendering NPI's NPPES county.

```
COVERAGE_ESTIMATE = OBSERVED_UNITS / medicare_inpatient_discharges_county_drg
```

Bounded to `[min_coverage, max_coverage]`. Fires only when `OBSERVED_UNITS ≥ direct_min_observed_units` (default 20). `CONFIDENCE_BUCKET` is `high` if `OBSERVED_UNITS ≥ high_confidence_min_units`, otherwise `medium`.

The denominator is Medicare FFS only — Medicare Advantage encounters are present in the PurpleLab Medicare channel but excluded from the CMS denominator. This is a documented artifact (see §8.1) and produces conservative (slightly low) projected uplifts on cells with high MA share. Field definitions for the source dataset are at the [Payerset Medicare Inpatient methodology page](https://docs.payerset.com/using-the-payerset-platform/data-dictionary/medicare-inpatient-methodology).

#### 6.3 `channel_prior`

Applied to every remaining cell that has at least `peer_min_observed_units` (default 5) observed units. The denominator is the per-channel coverage prior from `data/seed/channel_priors.csv`:

| Channel    | `pl_coverage_prior` | Implied uplift |
| ---------- | ------------------- | -------------- |
| Commercial | 0.60                | 1.67×          |
| Medicare   | 0.75                | 1.33×          |
| Medicaid   | 0.80                | 1.25×          |

```
COVERAGE_ESTIMATE = clamp(channel_prior, min_coverage, max_coverage)
```

`CONFIDENCE_BUCKET` is `low`. This tier is the workhorse for Commercial and for the long tail of HCPCS that have no county × procedure denominator from T-MSIS or CMS Inpatient. The priors are derived from PurpleLab's published payer-mix vs all-payer denominators and are reviewed each release.

#### 6.4 `suppress`

Cells with `OBSERVED_UNITS < peer_min_observed_units` and no administrative-truth denominator are not projected. They appear in the output with `PROJECTED_UNITS / P5 / P95` all NULL, `COVERAGE_SOURCE = 'suppress'`, and `CONFIDENCE_BUCKET = 'suppress'`. Downstream consumers can decide whether to filter them or pass them through.

### 7. Output schema

Output is partitioned parquet at `output_root/state=XX/year=YYYY/part-*.parquet`, written with zstd compression. Each row corresponds to one `(BILLING_NPI_NBR, PROCEDURE_CD, PAYER_ID, PAYER_CHANNEL_NAME, POS_CD, RATE_YEAR)` cell.

| Column                 | Type    | Description                                                |
| ---------------------- | ------- | ---------------------------------------------------------- |
| `BILLING_NPI_NBR`      | int64   | Rendering / billing provider NPI                           |
| `PROCEDURE_CD`         | string  | HCPCS / CPT code (or MS-DRG for inpatient rows)            |
| `PAYER_ID`             | int64   | Source payer identifier                                    |
| `PAYER_CHANNEL_NAME`   | string  | `Commercial` \| `Medicare` \| `Medicaid`                   |
| `POS_CD`               | string  | Place-of-service code (`UNK` if missing)                   |
| `OBSERVED_UNITS`       | float64 | Source-claims unit count (sum of `ATTRIBUTED_TOTAL_UNITS`) |
| `PROJECTED_UNITS`      | float64 | Bootstrap median                                           |
| `PROJECTED_UNITS_P5`   | float64 | 5th percentile (bootstrap)                                 |
| `PROJECTED_UNITS_P95`  | float64 | 95th percentile (bootstrap)                                |
| `COVERAGE_ESTIMATE`    | float64 | Coverage fraction used to scale observed → projected       |
| `COVERAGE_SOURCE`      | string  | Tier name (see §6)                                         |
| `COVERAGE_GRAIN`       | string  | Grain at which coverage was estimated                      |
| `CONFIDENCE_BUCKET`    | string  | `high` \| `medium` \| `low` \| `suppress`                  |
| `PROVIDER_COUNTY_FIPS` | string  | 5-digit county FIPS (from NPPES)                           |
| `PROVIDER_STATE`       | string  | Two-letter state (from NPPES)                              |
| `DATA_PERIOD_START`    | date32  | Min `REPORT_DD` over the cell                              |
| `DATA_PERIOD_END`      | date32  | Max `REPORT_DD` over the cell                              |
| `CALIBRATION_VINTAGE`  | string  | Run identifier from `config.yaml`                          |
| `RATE_YEAR`            | int64   | Source `RATE_YEAR`                                         |

Suppressed cells pass through with `OBSERVED_UNITS` set, `PROJECTED_UNITS / P5 / P95` NULL, and `COVERAGE_SOURCE = 'suppress'`.

### 8. QA and validation

One automated check runs with every release:

**Medicare FFS cross-check.** For every (state × procedure) pair where we have both PurpleLab Medicare-channel observations and a CMS Physician & Other Practitioners PUF reference total, we compute the relative delta `(projected - puf_total_services) / puf_total_services`. Pairs with `|delta| > systematic_delta_flag` (default 0.30) are surfaced in the QA report (`medicare_vs_puf_top_deltas`, `medicare_vs_puf_flagged_count`). Systematic same-signed deltas across an entire procedure or geography indicate either a coverage misestimate or an upstream attribution issue.

The QA stage also writes the coverage source mix (cells and observed units per tier) so that the share routed through each denominator is visible in every release.

Manual checks performed before each release: spot-check the top 50 NPIs by projected volume against publicly reported activity, and review all flagged (state × procedure) pairs from the cross-check.

#### 8.1 Known QA artifacts

The Medicare-vs-PUF check has two systematic artifacts that produce false positives. We document them rather than silently filter them so reviewers can apply judgment.

* **CMS PUF suppression floor.** CMS suppresses any PUF cell with fewer than 11 services. When a (state × procedure) combination has most NPIs at or below the floor, `SUM(puf.total_services)` materially underestimates true Medicare FFS volume. Combined with the fact that our `Medicare` channel includes Medicare Advantage while the PUF is FFS only, deltas in the 100×–1000× range at low-PUF-volume cells are expected and are not evidence of over-projection. Reviewers restrict the cross-check to (state × procedure) cells with `puf_total_services > 1000` and at least 10 unsuppressed reporting NPIs.
* **Medicare Advantage in the source channel.** PurpleLab's `Medicare` channel is Medicare FFS _plus_ Medicare Advantage. The CMS PUF denominator is FFS only. Procedures concentrated in MA populations (e.g. preventive screenings, certain managed-care drug administrations) will systematically project higher than the PUF total. This is correct behavior, not a projection error.

A future release will refine this check to operate at service-line counts and to split Medicare Advantage from Medicare FFS using `payer_subchannel`.

### 9. Reference run results

This section summarizes the V3 reference run on the full PurpleLab claims feed, calibration vintage `20260423-dev`, executed April 25, 2026. It is updated with each material release.

#### 9.1 Headline

| Metric                   | Value                                              |
| ------------------------ | -------------------------------------------------- |
| Active rendering NPIs    | 474,366 (27.79% of 1,706,813 Type-2 NPIs in NPPES) |
| Claims grain rows        | 208,236,867                                        |
| Observed units           | 54,866,012,445                                     |
| Projected units          | 82,372,848,585                                     |
| Implied average coverage | 67.0%                                              |
| Universe uplift          | 1.501×                                             |

The implied coverage is consistent with the source feed's documented market share. The universe uplift represents the multiplier from the observed claims captured by the source feed to the estimated true universe of services rendered.

#### 9.2 Coverage source mix

| Coverage source              | Cells          | Observed units | Share of observed |
| ---------------------------- | -------------- | -------------- | ----------------- |
| `tmsis_direct`               | 525,426        | 11,255,190,000 | 20.51%            |
| `direct_inpatient_benchmark` | 61,824         | 15,405,070     | 0.03%             |
| `channel_prior`              | 8,844,479      | 43,580,000,000 | 79.44%            |
| `suppress`                   | 4,894,730      | 9,797,506      | 0.02%             |
| **Total**                    | **14,326,459** | **\~54.86B**   | **\~100%**        |

The `tmsis_direct` tier carries 20.51% of all observed units — every one of those units is projected from a denominator sourced directly from state Medicaid agency reporting rather than from a channel-wide prior.

The `direct_inpatient_benchmark` tier serves only 61,824 cells and 0.03% of observed units. That is by design: PurpleLab's claims feed is overwhelmingly ambulatory, and this tier only fires on Medicare-channel inpatient cells (the source HCPCS must map to an MS-DRG with a county-level CMS Medicare FFS denominator, and the cell must clear the `direct_min_observed_units` floor of 20). Where it fires, it replaces a channel-prior projection with an administrative-truth denominator and upgrades the cell to `high` confidence.

The `channel_prior` tier is the largest by both cells and units. This is structural: every Commercial cell falls here (no comparable free Commercial denominator is available), as does every Medicare or Medicaid cell whose county × procedure key isn't covered by T-MSIS or CMS Inpatient.

The `suppress` tier accumulates 4.9M cells but only 9.8M observed units (0.02% of total). These are HCPCS × county × channel combinations with no administrative-truth denominator and observed counts below `peer_min_observed_units` (default 5). The aggregate impact on universe estimates is negligible.

#### 9.3 QA outcomes

| Check                                            | Result                                                                | Disposition |
| ------------------------------------------------ | --------------------------------------------------------------------- | ----------- |
| Medicare-vs-PUF flagged pairs                    | 112,074 (state × procedure) pairs with \`                             | delta       |
| Coverage hierarchy completeness                  | 100% of cells assigned to a tier                                      | Pass        |
| `tmsis_direct` confidence upgrades               | 525,426 Medicaid cells moved to `high` / `medium` confidence          | Pass        |
| `direct_inpatient_benchmark` confidence upgrades | 61,824 Medicare inpatient cells moved to `high` / `medium` confidence | Pass        |

#### 9.4 Effect of administrative-truth tiers

The 525,426 Medicaid cells served by `tmsis_direct` would otherwise route through `channel_prior` (0.80 prior, 1.25× uplift). T-MSIS swaps in the actual reported Medicaid universe for that county × procedure, replacing a national channel constant with a denominator that varies by geography and procedure. The point estimate for any individual cell can shift in either direction; the systematic improvement is in the variance of the projection.

The 61,824 Medicare-channel inpatient cells served by `direct_inpatient_benchmark` are similarly displaced — under a channel-prior-only configuration they would be projected at the Medicare prior (0.75, 1.33× uplift) regardless of geography or DRG mix. The CMS Inpatient PUF substitutes county × MS-DRG Medicare FFS discharges as the denominator. Both administrative-truth tiers carry a tighter bootstrap relative SD (0.10) than the channel prior (0.50).

#### 9.5 Runtime

End-to-end runtime on a Mac Studio M3 Ultra (32 logical cores, 64 GB RAM):

| Stage                                             | Time                               |
| ------------------------------------------------- | ---------------------------------- |
| 0. reference loads                                | \~1 s                              |
| 1. ingest + geography (208M-row claims\_grain)    | 7 s                                |
| 2. coverage hierarchy                             | \~25 s                             |
| 3. projection + bootstrap (208M rows × 500 draws) | dominant; depends on `run.workers` |
| 4. QA                                             | 3 s                                |

Stage 3 dominates total runtime (\~99%). It is parallelized via `concurrent.futures.ProcessPoolExecutor` (`run.workers`); on the same hardware with `workers: 31` it scales near-linearly with worker count, as batches are independent and the bootstrap is CPU-bound NumPy.

### 10. Limitations

* **Coverage is a model, not a measurement.** The administrative-truth tiers (`tmsis_direct`, `direct_inpatient_benchmark`) are tighter than channel priors but still subject to source-side suppression and partial coverage (e.g. MA exclusion in the Medicare FFS denominator). The `channel_prior` tier is a national constant per channel and does not vary by geography or procedure. We bound the coverage estimate to `[0.05, 1.00]` and surface confidence labels on every cell, but the estimate carries irreducible uncertainty that the bootstrap intervals attempt to characterize.
* **The projection is per-NPI, not per-patient.** We do not attempt to deduplicate patients across NPIs. A single patient receiving the same procedure from two different NPIs will appear in both NPIs' projections.
* **Channel attribution depends on `PAYER_CHANNEL_NAME` in the source feed.** When the source channel is missing or out-of-scope (Other, Dual), the row is filtered at ingest and does not appear in the output.
* **Inpatient denominator is Medicare FFS only.** The `direct_inpatient_benchmark` tier covers Medicare-channel cells and uses CMS Medicare FFS Part A discharges. Inpatient cells in Medicaid and Commercial channels do not have a comparable free, county-grained denominator and flow through `channel_prior`. Inpatient projections in those channels should be treated as lower-confidence than Medicare-channel inpatient or any cell projected via T-MSIS.
* **T-MSIS lag.** T-MSIS Medicaid Provider Spending lags the source claims feed by approximately 12 months. When projecting recent months for Medicaid, the most recent T-MSIS vintage is used, which can produce small denominator drift. The bootstrap relative SD for `tmsis_direct` (0.10) is sized to absorb this.
* **Commercial coverage has no administrative-truth tier.** Every Commercial cell projects at the channel prior. There is no free, comprehensive Commercial denominator equivalent to T-MSIS or the CMS PUFs. Commercial projections inherit the wider `channel_prior` bootstrap interval (0.50 rel SD).

### 11. Reproducibility

Every release is reproducible from a single config file (`config.yaml`) plus the reference data snapshot under `data/reference/`. The config records:

* The source claims parquet path and vintage.
* The NPPES vintage.
* The reference data vintage for each source.
* The bootstrap seed (default 42) and number of draws (default 500).
* All tier thresholds and bound parameters.

A given combination of inputs and config will produce a byte-identical output, modulo parquet metadata timestamps. Per-batch RNGs are derived deterministically from `(seed, batch_idx)` so output is reproducible regardless of worker completion order.

### 12. Changelog

#### V3 — April 2026

Architectural simplification. Removes the population-benchmark and peer-group infrastructure that depended on a half-built utilization-benchmarks scaffold (\~10 procedures with mixed seed-placeholder + MEPS data). The pre-customer review identified a 99213 / 99214 Commercial double-count bug rooted in that scaffold; rather than patch it, V3 removes the scaffold entirely and routes the cells it served to `channel_prior`.

What was removed:

* The `expected_units` stage (population × utilization\_rate) and all of its denominator inputs: ACS population, KFF Medicaid enrollment, MEPS-HC utilization, HCUPnet inpatient rates, Medicare enrollment from the CMS Geographic Variation PUF, and the `utilization_benchmarks.csv` scaffold.
* The `direct_pop_benchmark` coverage tier.
* The `peer_group` coverage tier and its Bayesian shrinkage model (`models/shrinkage.py`).
* Config keys `coverage.shrinkage_prior_strength` and `qa.population_envelope_flag_multiplier`, and the `coverage_rel_sd` entries for `direct_pop_benchmark` and `peer_group`.
* The population-envelope QA check.
* All download scripts and reference loaders for the removed datasets, plus the seed `age_bands.csv` and `utilization_benchmarks.seed.csv`.

What was simplified:

* Coverage hierarchy reduced from six tiers to four: `tmsis_direct` → `direct_inpatient_benchmark` → `channel_prior` → `suppress`.
* Pipeline reduced from five stages to four: refs → ingest → coverage → projection → QA.
* Reference data set reduced from seven sources to four (channel priors, T-MSIS, CMS Inpatient, CMS Physician PUF for QA).

Net effect at full population (vs the V2 reference run on the same input):

* `tmsis_direct` cells: unchanged (525,426; 20.51% of observed units).
* `direct_inpatient_benchmark` cells: unchanged (61,824; 0.03% of observed units).
* Cells previously in `direct_pop_benchmark` or `peer_group` reroute to `channel_prior`. The number of cells in `channel_prior` increases from 8.80M to 8.84M and its observed-unit share rises from 77.48% to 79.44%. Universe uplift moves from 1.492× to 1.501× — a small, expected bump because cells previously projected at the population-benchmark coverage now project at the (slightly higher) channel-prior coverage.
* `suppress` cells unchanged (4.89M; 0.02% of observed units).

The simplification removes a code-and-data dependency that was not pulling its weight: at V2 release, `direct_pop_benchmark` accounted for 1.96% of observed units and `peer_group` accounted for less than 0.001%, while requiring six external reference datasets and a roughly 12-stage download/build pipeline. The two administrative-truth tiers (T-MSIS and CMS Inpatient) carry the projection fidelity that the population benchmark was supposed to deliver, with denominators drawn from administrative reporting rather than survey-derived utilization rates.

#### V2 — April 2026

Added two administrative-truth coverage tiers above the V1 `direct_pop_benchmark`:

1. **`tmsis_direct` (Medicaid).** Denominator: T-MSIS Medicaid Provider Spending, aggregated to county × HCPCS using the rendering NPI's NPPES county. Confidence on those cells upgraded to `high`. Bootstrap relative SD: 0.10.
2. **`direct_inpatient_benchmark` (Medicare).** Denominator: CMS Medicare Inpatient Hospitals (by Provider and Service) PUF, aggregated to county × MS-DRG. Medicare FFS Part A only — Medicare Advantage encounters in PurpleLab's Medicare channel are excluded from the denominator (documented in §8.1). Confidence: `high`. Bootstrap relative SD: 0.10.

When either reference is absent, the corresponding tier produces zero rows and the affected cells fall through to lower tiers.

Other V2 changes:

* CMS Geographic Variation PUF loader updated to handle the new schema with fuzzy column detection and `All` demographic stratifier filtering.
* Reference data orchestrator (`fetch_all_references.py`) caches downloaded files and skips re-download by default.
* Bootstrap parallelized via `concurrent.futures.ProcessPoolExecutor`. Two new config keys: `run.threads` (DuckDB intra-query parallelism) and `run.workers` (process-pool size for the bootstrap stage). Per-batch RNGs derived deterministically from `(seed, batch_idx)` so output is reproducible regardless of worker completion order. The `__main__` guard in `claims_projection/__main__.py` is required because spawn-mode multiprocessing on macOS re-imports the parent's `__main__` in every child.

#### V1 — March 2026

Initial release. Coverage hierarchy: `direct_pop_benchmark` → `peer_group` → `channel_prior` → `suppress`. Reference data: ACS, KFF, MEPS, HCUPnet (inpatient, best-effort), CMS Physician PUF (QA only). Both V1 tiers below the channel prior were removed in V3.
