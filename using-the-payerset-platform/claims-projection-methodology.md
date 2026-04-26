# Claims Projection Methodology

## Claims Projection Methodology

This document describes how Payerset estimates the universe-level claim volume that each rendering provider performs, given a partial sample of observed claims. It is intended as a transparent, auditable account of every input, transformation, and assumption that produces the projected unit counts published in our APIs and dashboards.

### 1. What we project

Each row in our output represents a single cell at the grain:

```
NPI × procedure_cd × channel × age_band × county × year × month
```

For every cell we publish:

* `observed_units` — the count we directly observed in the source claims feed.
* `projected_units` — our estimate of the _true_ count of services rendered by that NPI in that cell, including services that fell outside our sample.
* `projected_units_lo`, `projected_units_hi` — the 5th and 95th percentile of a 500-draw parametric bootstrap.
* `coverage_estimate` — the implied share of the universe captured by `observed_units`. Bounded to `[0.05, 1.00]`.
* `coverage_source` — which tier of the hierarchy produced the estimate (see §6).
* `confidence` — `high`, `medium`, or `low`.

The projection is `projected_units = observed_units / coverage_estimate`.

#### 1.1 What `observed_units` is

`observed_units` is the unit count from the **source claims feed (PurpleLab) only**. It is not a pooled or deduplicated combination of multiple data sources. The pipeline ingests several other datasets, but each plays a different role and none of them adds to `observed_units`:

* The provider directory (NPPES) attaches geography and taxonomy to each NPI.
* ACS, KFF, MEPS, and HCUPnet are denominators for the `direct_pop_benchmark` tier (`expected_units = population × utilization_rate`).
* T-MSIS Medicaid Provider Spending is the denominator for the `tmsis_direct` tier on Medicaid cells.
* The CMS Medicare Inpatient (by Provider and Service) PUF is the denominator for the `direct_inpatient_benchmark` tier on Medicare-channel inpatient (MS-DRG) cells.
* The CMS Physician PUF is used only as a QA cross-check and never enters the projection math.

The reference datasets shape `coverage_estimate`, which determines how `observed_units` is scaled to a universe estimate. `observed_units` itself is always a single-source quantity. This distinction matters when reconciling counts: any reader who tries to add observed PurpleLab units to TMSIS or PUF totals is double-counting.

#### 1.2 Which PurpleLab field underlies `observed_units`

The PurpleLab `CLAIMS_ORDERED` feed exposes two volume fields per row:

* **`ATTRIBUTED_TOTAL_UNITS`** — units on the original-claim grain, attributed to the billing NPI on that row. Populated for \~94% of professional rows and \~97% of inpatient rows.
* **`COUNT_OF_UNITS`** — units sourced from a matched remittance advice. Only populated for \~13–16% of rows (the remit-matched subset).

We sum `ATTRIBUTED_TOTAL_UNITS` because it reflects the full claim population in the feed. Rows where `ATTRIBUTED_TOTAL_UNITS` is null are remits without a matched open claim and are skipped — they have no billing-NPI grain row to attribute to. This is implemented in `stages/ingest.py`.

### 2. Inputs

| Input              | Source                     | Grain                                                                        | Notes                                                                                                                      |
| ------------------ | -------------------------- | ---------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| Source claims      | PurpleLab `CLAIMS_ORDERED` | one row per (billing NPI × procedure × payer × POS × month × modifier × TOB) | Includes professional and inpatient (`CLAIM_TYPE_CD = 'P' / 'I'`). Volume sourced from `ATTRIBUTED_TOTAL_UNITS` (see §1.2) |
| Provider directory | Enriched NPPES (Type 2)    | one row per organization NPI                                                 | Includes county FIPS, primary taxonomy, CBSA, lifetime claim count                                                         |

Claims are joined to the provider directory on rendering NPI to attach county, primary taxonomy, and CBSA before any aggregation.

### 3. Reference data sources

The projection requires five denominators (population, channel enrollment, utilization rates, Medicaid spending, Medicare FFS inpatient discharges) and one cross-check (Medicare FFS). All references are normalized to county × age-band (or county × MS-DRG for inpatient) where applicable so they can be joined to the claims grain.

| Reference                                      | Provider                                                                                                                                                                                                                                          | Used for                                                                        | Required                          |
| ---------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------- | --------------------------------- |
| ACS 5-year                                     | U.S. Census Bureau                                                                                                                                                                                                                                | Population by county × age band × sex (Tables B01001, B27010, S2701, C27002)    | Yes                               |
| Medicare enrollment                            | CMS Geographic Variation PUF                                                                                                                                                                                                                      | Medicare beneficiaries by county                                                | Yes                               |
| Medicaid enrollment                            | KFF state-level + ACS share allocation                                                                                                                                                                                                            | Medicaid beneficiaries by county                                                | Yes                               |
| Commercial enrollment                          | ACS S2701 / C27002 (residual)                                                                                                                                                                                                                     | Commercial-covered population by county                                         | Yes                               |
| Utilization benchmarks                         | MEPS-HC consolidated + visit files; HCUPnet (inpatient)                                                                                                                                                                                           | Rate per 1,000 per year by procedure × channel × age band                       | Yes                               |
| Medicare FFS PUF                               | CMS Physician & Other Practitioners PUF                                                                                                                                                                                                           | NPI × HCPCS reference for QA                                                    | No (QA only)                      |
| Medicaid Provider Spending                     | T-MSIS Medicaid Provider Spending dataset                                                                                                                                                                                                         | Medicaid units by billing NPI × HCPCS × month, aggregated to county × procedure | No (V2+; falls through if absent) |
| Medicare Inpatient (by Provider & Service) PUF | CMS Medicare Inpatient Hospitals (Provider × MS-DRG × discharges, Medicare FFS Part A only) — see [Payerset Medicare Inpatient methodology](https://docs.payerset.com/using-the-payerset-platform/data-dictionary/medicare-inpatient-methodology) | Medicare FFS denominator for inpatient cells, aggregated to county × MS-DRG     | No (V2+; falls through if absent) |

All references are versioned and stored under `data/reference/` with the source URL, retrieval date, and any source-side suppression rules captured in `DOWNLOADS.md`.

### 4. Pipeline stages

The end-to-end run executes five ordered stages. All stages share a single in-process DuckDB connection; intermediate outputs are materialized as DuckDB tables, not files.

#### Stage 1 — Ingest and geography

The source parquet is filtered to the configured states (if any) and the configured `sample_fraction`. Sampling is deterministic and NPI-keyed: a hash of the NPI is compared to the fraction so that _all_ services for a sampled NPI are retained, never partial cuts of an NPI's history. This preserves within-NPI volume relationships, which is necessary for coverage estimation.

The claims grain is built by aggregating sampled service lines to NPI × procedure × channel × age band × county × year × month, joined to NPPES for geography and taxonomy.

#### Stage 2 — Expected units

For each cell we compute the expected universe count:

```
expected_units = population(age_band, county, channel) × utilization_rate(procedure, channel, age_band) / 1000
```

`population` comes from ACS × channel-share allocation. `utilization_rate` is the MEPS or HCUP rate per thousand per year, prorated to the months observed in the cell. The expected-units cube is the canonical denominator for the population-benchmark coverage tier.

#### Stage 4 — Projection and bootstrap

For each cell, `projected_units = observed_units / coverage_estimate`, clamped so that `coverage_estimate ∈ [min_coverage, max_coverage]`. Confidence intervals are produced by a 500-draw parametric bootstrap on `coverage_estimate`, with the relative standard deviation chosen by the coverage source:

| Coverage source              | Relative SD |
| ---------------------------- | ----------- |
| `tmsis_direct`               | 0.10        |
| `direct_inpatient_benchmark` | 0.10        |
| `direct_pop_benchmark`       | 0.15        |
| `peer_group`                 | 0.30        |
| `channel_prior`              | 0.50        |

The relative SD encodes our prior on how noisy each tier is: a Medicaid cell whose denominator comes directly from T-MSIS is treated as much tighter than a cell that fell through to a channel-wide prior.

For each draw, `coverage_estimate` is sampled from a truncated normal centered at the point estimate with the relative SD above, then `projected_units` is recomputed. The 5th and 95th percentiles across 500 draws become `projected_units_lo` and `projected_units_hi`.

#### Stage 5 — QA

The QA stage emits a JSON report containing:

* The coverage source mix (cells and observed units by tier).
* Population-envelope flags: cells whose `projected_units` exceeds `population_envelope_flag_multiplier × expected_units` (default 3.0×). A flag rate above 10% triggers methodology review.
* For Medicare cells with both PurpleLab observations and CMS PUF reference values, the top systematic deltas at the NPI × HCPCS grain. Deltas above `systematic_delta_flag` (default 0.30) are surfaced.

### 5. Channel attribution

Each service line is attributed to one of three channels — Medicare, Medicaid, or Commercial — using the source claim's payer information first. When the payer is unknown or ambiguous, the line is attributed using the channel prior for the patient's age band and county. Channel attribution is performed once at ingest and is not revisited downstream.

### 6. Coverage hierarchy

The coverage estimate for a cell is selected by the first tier (top to bottom) that the cell qualifies for. Tiers are evaluated in the order shown.

#### 6.1 `tmsis_direct` _(Medicaid only — V2+)_

Applied when the cell is Medicaid and a matching county × procedure row exists in the T-MSIS Medicaid Provider Spending aggregate. The denominator is the universe-of-truth Medicaid units for that county × procedure as reported by the state Medicaid agencies via T-MSIS.

```
coverage_estimate = observed_units / medicaid_units_county_procedure
```

Bounded to `[min_coverage, max_coverage]`. Confidence is `high`. This tier replaces what V1 would have produced via population × utilization benchmark for the same cells, and is preferred because the denominator is administrative truth rather than a survey-derived rate.

#### 6.2 `direct_inpatient_benchmark` _(Medicare only — V2+)_

Applied when the cell is Medicare and a matching county × MS-DRG row exists in the CMS Medicare Inpatient Hospitals (by Provider and Service) aggregate. The denominator is Medicare FFS Part A discharges for that county × DRG, aggregated from per-provider counts via the rendering NPI's NPPES county.

```
coverage_estimate = observed_units / medicare_inpatient_discharges_county_drg
```

Bounded to `[min_coverage, max_coverage]`. Confidence is `high`. The minimum-observed-units floor is `direct_min_observed_units` (default 20); cells below it fall through to the lower tiers. The denominator is Medicare FFS only — Medicare Advantage encounters are present in the PurpleLab Medicare channel but excluded from the CMS denominator; this is a documented artifact (see §8.1) and produces conservative (slightly low) projected uplifts on cells with high MA share. Field definitions for the source dataset are at the [Payerset Medicare Inpatient methodology page](https://docs.payerset.com/using-the-payerset-platform/data-dictionary/medicare-inpatient-methodology).

#### 6.3 `direct_pop_benchmark`

Applied when the cell has at least `direct_min_observed_units` (default 20) observed units. The denominator is the expected units cube from Stage 2.

```
coverage_estimate = observed_units / expected_units
```

Confidence is `high` if `observed_units ≥ high_confidence_min_units` (default 100), otherwise `medium`.

#### 6.4 `peer_group`

Applied when the cell has at least `peer_min_observed_units` (default 5) observed units but did not qualify for `direct_pop_benchmark`. The estimate is a Bayesian shrinkage of the cell-level coverage toward the peer-group mean coverage:

```
coverage_estimate = (n_obs × cell_coverage + κ × peer_coverage) / (n_obs + κ)
```

where `n_obs` is the observed unit count, `cell_coverage` is the raw `observed_units / expected_units`, `peer_coverage` is the mean `observed_units / expected_units` across peers, and `κ` is `shrinkage_prior_strength` (default 50 pseudo-observations). Peer groups are defined as cells sharing primary taxonomy and CBSA. Confidence is `medium`.

#### 6.5 `channel_prior`

Applied when the cell has fewer than `peer_min_observed_units` (default 5) observed units. The denominator is the channel-level prior coverage estimate, computed as the volume-weighted mean of all `direct_pop_benchmark` cells in the same channel.

```
coverage_estimate = channel_prior_coverage
```

Confidence is `low`.

#### 6.6 `suppress`

Cells with zero observed units or whose denominators are missing or invalid are not projected. They appear in the output with `projected_units = 0` and `coverage_source = 'suppress'`.

### 7. Output schema

Output is partitioned parquet at `data/output/projected_units/state=XX/year=YYYY/`:

| Column                | Type   | Description                                      |
| --------------------- | ------ | ------------------------------------------------ |
| `npi`                 | string | Rendering provider NPI                           |
| `procedure_cd`        | string | HCPCS / CPT code                                 |
| `channel`             | string | `Medicare` \| `Medicaid` \| `Commercial`         |
| `age_band`            | string | One of the bands in `data/seed/age_bands.csv`    |
| `county_fips`         | string | 5-digit FIPS                                     |
| `year`                | int    | Service year                                     |
| `month`               | int    | Service month                                    |
| `observed_units`      | int    | Source-claims unit count                         |
| `projected_units`     | float  | Point estimate                                   |
| `projected_units_lo`  | float  | 5th percentile (bootstrap)                       |
| `projected_units_hi`  | float  | 95th percentile (bootstrap)                      |
| `coverage_estimate`   | float  | `observed / projected`                           |
| `coverage_source`     | string | Tier name (see §6)                               |
| `confidence`          | string | `high` \| `medium` \| `low`                      |
| `expected_units`      | float  | Population-benchmark denominator (when computed) |
| `calibration_vintage` | string | Run identifier from `config.yaml`                |

### 8. QA and validation

Two automated checks run with every release:

1. **Population envelope.** For each cell with both an `expected_units` value and a `projected_units` value, we flag cells where `projected_units > flag_multiplier × expected_units` (default 3.0×). The aggregate flag rate is reported in the QA JSON. A flag rate above 10% is treated as a release blocker pending methodology review.
2. **Medicare FFS cross-check.** For every NPI × HCPCS pair where we have both a PurpleLab observation and a CMS Physician & Other Practitioners PUF reference value, we compute the relative delta `(projected - puf) / puf`. Pairs with `|delta| > systematic_delta_flag` (default 0.30) are surfaced in the QA report. Systematic same-signed deltas across an entire procedure or geography indicate either a coverage misestimate or an upstream attribution issue.

Manual checks performed before each release include: spot-checking the top 50 NPIs by projected volume against publicly reported activity, and reviewing all cells flagged by either automated check.

#### 8.1 Known QA artifacts

The two automated checks have known systematic artifacts that produce false positives. We document them rather than silently filter them so reviewers can apply judgment.

* **Population envelope is sensitive to unit-multiplier semantics.** The check compares `projected_units` to `population × utilization_rate`. Our `observed_units` sums per-row unit values (e.g., a single drug administration row may carry hundreds or thousands of units of dose), while the utilization-rate denominator counts services. For drug J-codes (e.g., `J7613`, `J2704`), per-trip codes (e.g., `P9604`), and other procedures with high per-row unit multipliers, the envelope ratio routinely exceeds 3× without indicating a real over-projection. The QA writer surfaces the raw flag count; release reviewers exclude unit-multiplier procedures from the materiality decision.
* **CMS PUF cross-check distorts at the suppression floor.** CMS suppresses any PUF cell with fewer than 11 services. When a (state × procedure) combination has most NPIs at or below the floor, `SUM(puf.total_services)` materially underestimates true Medicare FFS volume. Combined with the fact that our `Medicare` channel includes Medicare Advantage while the PUF is FFS only, deltas in the 100×–1000× range at low-PUF-volume cells are expected and are not evidence of over-projection. Reviewers restrict the cross-check to (state × procedure) cells with `puf_total_services > 1000` and at least 10 unsuppressed reporting NPIs.

A future release will refine these checks to operate at service-line counts and to split Medicare Advantage from Medicare FFS using `payer_subchannel`.

### 9. Reference run results

This section summarizes the V2 reference run on the full PurpleLab claims feed, calibration vintage `20260423-dev`, executed April 25, 2026. It is updated with each material release.

#### 9.1 Headline

| Metric                   | Value                                              |
| ------------------------ | -------------------------------------------------- |
| Active rendering NPIs    | 474,366 (27.79% of 1,706,813 Type-2 NPIs in NPPES) |
| Claims grain rows        | 208,236,867                                        |
| Observed units           | 54,866,012,445                                     |
| Projected units          | 81,871,313,806                                     |
| Implied average coverage | 67.0%                                              |
| Universe uplift          | 1.492×                                             |

The implied coverage is consistent with the source feed's documented market share. The universe uplift represents the multiplier from the observed claims captured by the source feed to the estimated true universe of services rendered.

#### 9.2 Coverage source mix

| Coverage source                     | Cells          | Observed units     | Share of observed |
| ----------------------------------- | -------------- | ------------------ | ----------------- |
| `tmsis_direct` _(V2)_               | 525,426        | 11,255,190,000     | 20.5%             |
| `direct_inpatient_benchmark` _(V2)_ | 61,824         | 15,405,070         | 0.03%             |
| `direct_pop_benchmark`              | 43,850         | 1,075,459,000      | 1.96%             |
| `peer_group`                        | 2,445          | 27,348             | <0.001%           |
| `channel_prior`                     | 8,798,184      | 42,508,710,000     | 77.48%            |
| `suppress`                          | 4,894,730      | 9,797,506          | 0.018%            |
| **Total**                           | **14,326,459** | **54,864,588,924** | **\~100%**        |

(Total reconciliation: the small \~0.003% delta between the table sum and the headline observed-units figure is from the scientific-notation rounding in the run log; raw aggregates reconcile to the byte.)

The `tmsis_direct` tier carries 20.5% of all observed units in V2 — every one of those units is projected from a denominator sourced directly from state Medicaid agency reporting rather than from a population × utilization estimate. In V1 this share was zero by construction.

The `direct_inpatient_benchmark` tier serves only 61,824 cells and 0.03% of observed units in this release. That is by design: PurpleLab's claims feed is overwhelmingly ambulatory, and this tier only fires on Medicare-channel inpatient cells (the source HCPCS must map to an MS-DRG with a county-level CMS Medicare FFS denominator, and the cell must clear the `direct_min_observed_units` floor of 20). Where it fires, it replaces a channel-prior projection with an administrative-truth denominator and upgrades the cell to `high` confidence; this is most material for inpatient surgical and diagnostic procedures concentrated in a small number of high-volume facilities.

The `channel_prior` tier remains the largest by both cells and units. This is a structural artifact of the long tail of HCPCS codes: the 34-row seed utilization-benchmark table covers high-volume procedures, and any cell whose procedure is not in that table — but which has at least one observed unit — falls through to channel-level priors.

The `suppress` tier accumulates 4.9M cells but only 9.8M observed units (0.018% of total). These are HCPCS × county × channel combinations with no benchmark, no peers, and observed counts too small to project independently. The aggregate impact on universe estimates is negligible.

#### 9.3 QA outcomes

| Check                                            | Result                                                     | Disposition                                                                                                                                                                                                                        |
| ------------------------------------------------ | ---------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Population envelope flag rate                    | Reviewed at the 10% threshold                              | Drug J-code unit semantics drive the bulk of flagged cells (see §8.1). No projection error identified.                                                                                                                             |
| Medicare-vs-PUF flagged pairs                    | Reviewed                                                   | Attributed to PUF suppression floor and Medicare Advantage inclusion in the source-feed Medicare channel (see §8.1). Restricted-scope review (PUF NPIs ≥ 10, total\_services ≥ 1000) shows median absolute delta within tolerance. |
| Coverage hierarchy completeness                  | 100% of cells assigned to a tier                           | Pass                                                                                                                                                                                                                               |
| `tmsis_direct` confidence upgrades               | 525,426 Medicaid cells moved to `high` confidence          | Pass                                                                                                                                                                                                                               |
| `direct_inpatient_benchmark` confidence upgrades | 61,824 Medicare inpatient cells moved to `high` confidence | Pass                                                                                                                                                                                                                               |

#### 9.4 Effect of the V2 `tmsis_direct` and `direct_inpatient_benchmark` tiers

The 525,426 Medicaid cells served by `tmsis_direct` would, under the V1 hierarchy, have been routed through the standard tiers based on observed-unit thresholds and benchmark availability. The vast majority would have landed in `channel_prior` because most county × procedure × Medicaid combinations in our claims feed have fewer than the `direct_min_observed_units` threshold (20) needed to qualify for `direct_pop_benchmark`, and Medicaid-specific peer groups defined by taxonomy × CBSA are sparse.

The 61,824 Medicare-channel inpatient cells served by `direct_inpatient_benchmark` are similarly displaced. Under V1, county × MS-DRG denominators were not available; the `direct_pop_benchmark` denominator (population × utilization rate) does not capture inpatient admission rates with the same fidelity that CMS PUF discharge counts do. The V2 tier swaps in administrative-truth discharge counts as the denominator for these cells.

V2 reclassifies all of these cells from `low` confidence (channel-prior) or `medium` (population benchmark) to `high` confidence (administrative-truth denominator). The point estimate for any individual cell can shift in either direction; the systematic improvement is in the variance of the projection rather than its level.

A formal V1-vs-V2 A/B comparison run is planned for a future release and will replace this qualitative description with measured deltas.

#### 9.5 Runtime

End-to-end runtime on a Mac Studio M3 Ultra (32 logical cores, 64 GB RAM), single-process baseline (`workers: 1`):

| Stage                                             | Time           |
| ------------------------------------------------- | -------------- |
| 0. reference loads                                | \~1 s          |
| 1. ingest + geography (208M-row claims\_grain)    | 7 s            |
| 2. expected units (83k cells)                     | <1 s           |
| 3. coverage hierarchy (six tiers)                 | 33 s           |
| 4. projection + bootstrap (208M rows × 500 draws) | 2 h 19 m       |
| 5. QA                                             | 3 s            |
| **Total**                                         | **\~2 h 20 m** |

Stage 4 dominates total runtime (\~99%). It is parallelized via `concurrent.futures.ProcessPoolExecutor` (`run.workers`); on the same hardware with `workers: 31` it scales near-linearly with worker count, as batches are independent and the bootstrap is CPU-bound NumPy.

### 10. Limitations

* **Coverage is a model, not a measurement.** The denominators in tiers 6.3–6.5 are themselves estimates derived from third-party reference data. The administrative-truth tiers (6.1 `tmsis_direct` and 6.2 `direct_inpatient_benchmark`) are tighter but still subject to source-side suppression and partial coverage (e.g., MA exclusion in the Medicare FFS denominator). We bound the coverage estimate to `[0.05, 1.00]` and surface confidence labels on every cell, but the estimate carries irreducible uncertainty that the bootstrap intervals attempt to characterize.
* **The projection is per-NPI, not per-patient.** We do not attempt to deduplicate patients across NPIs. A single patient receiving the same procedure from two different NPIs will appear in both NPIs' projections.
* **Channel attribution depends on payer fields in the source feed.** When payer is missing, we attribute via channel priors at the age × county level, which is correct in expectation but noisy at the individual claim level.
* **Inpatient denominator is Medicare FFS only.** The `direct_inpatient_benchmark` tier covers Medicare-channel cells and uses CMS Medicare FFS Part A discharges as the denominator. Inpatient cells in the Medicaid and Commercial channels do not have a comparable free, county-grained denominator and continue to flow through `peer_group` and `channel_prior`. Inpatient projections in those channels should be treated as lower-confidence than Medicare-channel inpatient or ambulatory projections in the same release.
* **T-MSIS lag.** T-MSIS Medicaid Provider Spending lags the source claims feed by approximately 12 months. When projecting recent months for Medicaid, the most recent T-MSIS vintage is used, which can produce small denominator drift. The bootstrap relative SD for `tmsis_direct` is sized to absorb this.

### 11. Reproducibility

Every release is reproducible from a single config file (`config.yaml`) plus the reference data snapshot under `data/reference/`. The config records:

* The source claims parquet path and vintage.
* The NPPES vintage.
* The reference data vintage for each source.
* The bootstrap seed (default 42) and number of draws (default 500).
* All tier thresholds, prior strengths, and bound parameters.

A given combination of inputs and config will produce a byte-identical output, modulo parquet metadata timestamps.

### 12. Changelog

#### V2 — April 2026

Adds two administrative-truth coverage tiers above the V1 `direct_pop_benchmark`:

1.  **`tmsis_direct` (Medicaid).** The denominator comes from the T-MSIS Medicaid Provider Spending dataset (parquet, \~3 GB, vintage 2026-02-09), aggregated to county × HCPCS using the rendering NPI's NPPES county. T-MSIS is the universe-of-truth for Medicaid services as reported by state Medicaid agencies, so this tier replaces what V1 would have produced via population × utilization benchmark for the same Medicaid cells.

    **Net effect at full population:** 525,426 Medicaid cells move into `tmsis_direct`, accounting for **20.5% of observed units across the run** (11.26B of 54.87B). Confidence on those cells is upgraded to `high`. Bootstrap relative SD for `tmsis_direct` is set to 0.10, the tightest of any tier. See §9 for full reference-run results.
2.  **`direct_inpatient_benchmark` (Medicare).** The denominator comes from the CMS Medicare Inpatient Hospitals (by Provider and Service) PUF, aggregated to county × MS-DRG using the rendering NPI's NPPES county. This is Medicare FFS Part A discharges; Medicare Advantage encounters in PurpleLab's Medicare channel are excluded from the denominator (documented in §8.1). Confidence on cells served by this tier is `high`. Bootstrap relative SD is 0.10.

    **Net effect at full population:** 61,824 Medicare-channel inpatient cells move into `direct_inpatient_benchmark` (0.03% of observed units). The small share is structural — PurpleLab's feed is overwhelmingly ambulatory and only inpatient HCPCS that map to an MS-DRG with a county-level FFS denominator clear the floor. Where it fires, the tier replaces a channel-prior projection with an administrative-truth denominator. See §9 for full reference-run results.

When either reference is absent (configuration choice or data unavailable), the corresponding tier produces zero rows and the affected cells fall through to lower tiers with no change in output schema.

Other V2 changes:

* CMS Geographic Variation PUF loader updated to handle the new schema with fuzzy column detection and `All` demographic stratifier filtering. This corrects a systematic over-counting of Medicare enrollment introduced when CMS restructured the PUF in early 2026.
* Reference data orchestrator (`fetch_all_references.py`) now caches downloaded files and skips re-download by default. Now also handles the Medicare Inpatient PUF.
* Stage 4 bootstrap parallelized via `concurrent.futures.ProcessPoolExecutor`. Two new config keys: `run.threads` (DuckDB intra-query parallelism for stages 1-3) and `run.workers` (process-pool size for stage 4). Per-batch RNGs are derived deterministically from `(seed, batch_idx)` so output is reproducible regardless of the order in which workers complete. The `__main__` guard in `claims_projection/__main__.py` is required because spawn-mode multiprocessing on macOS re-imports the parent's `__main__` in every child.
* Documentation expanded to include the §8.1 known QA artifacts disclosure and the §9 reference-run results section (refreshed for the V2 reference run with both new tiers active).

#### V1 — March 2026

Initial release. Coverage hierarchy: `direct_pop_benchmark` → `peer_group` → `channel_prior` → `suppress`. Reference data: ACS, KFF, MEPS, HCUPnet (inpatient, best-effort), CMS Physician PUF (QA only). All projection and QA logic described in §4–§8 was present in V1 with the exception of the `tmsis_direct` tier.
