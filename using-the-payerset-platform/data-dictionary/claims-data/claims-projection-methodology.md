---
description: >-
  This is the methodology by which we project claims using multiple sources as
  detailed below.
---

# Claims Projection Methodology

## Claims Projection Methodology

This document describes how Payerset estimates the universe-level claim volume that each rendering provider performs, given a partial sample of observed claims. It is intended as a transparent, auditable account of every input, transformation, and assumption that produces the projected unit counts published in our APIs and dashboards.

Current version: **V4.2** (August 2026). See §12 for the changelog.

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
* `PROJECTED_UNITS_P5`, `PROJECTED_UNITS_P95` — the 5th and 95th percentile of a parametric bootstrap (`run.bootstrap_draws`, 500 in the shipped config) that adds observation and coverage noise around the deterministic point.
* `ATTRIBUTED_CLAIM_COUNT` — the sum of PurpleLab `ATTRIBUTED_CLAIM_COUNT` over the cell, i.e. observed claim count at the source grain.
* `ATTRIBUTED_CLAIM_COUNT_PROJECTED` — projected claim count. Computed as observed claim count × `1 / COVERAGE_ESTIMATE` (the same factor used on units and charge dollars). Bootstrap percentiles are not published for claim count; the bootstrap is parametric on units only.
* `ATTRIBUTED_CHARGE_AMT` — the sum of PurpleLab `ATTRIBUTED_CHARGE_AMT` over the cell, i.e. observed billed-charge dollars at the source grain.
* `ATTRIBUTED_CHARGE_AMT_PROJECTED` — projected billed-charge dollars. Computed as observed charge dollars × the same `1 / COVERAGE_ESTIMATE` factor used on units.
* `COVERAGE_ESTIMATE` — the implied share of the universe captured by the source feed for this cell. Bounded to `[0.05, 1.00]`.
* `COVERAGE_SOURCE` — which tier of the hierarchy produced the estimate (see §6).
* `COVERAGE_GRAIN` — the grain at which the coverage fraction was estimated, including the ratio metric (e.g. `npi|procedure|medicaid_universe|claims_vs_lines`) and, for the channel prior, its provenance (`national|channel|empirical` vs `national|channel|seed`).
* `CONFIDENCE_BUCKET` — `high`, `medium`, `low`, or `suppress`.

**Capture is a rate (V4.1/V4.2).** Capture — the share of a provider's true volume that reaches the feed — is a property of the provider's claims routing (which clearinghouses feed the source), not of any particular calendar year. It is therefore measured once, at a complete reference claims year (`coverage.capture_reference_year`), and applied to every rate year. Measuring per-year would conflate capture with year completeness and silently annualize partial rate years. Measured capture has been empirically stable across complete years (NJ Medicare FFS: 0.778 in 2024, 0.776 in 2025); the QA capture-drift report (§8) re-measures it for every claims year in every release so the assumption is monitored, not presumed. The Medicaid tiers are measured at the T-MSIS denominator year (§6.1), which may sit below the global reference year; both years are recorded in the QA report.

Within a coverage key — `(NPI, county, procedure, channel, subchannel class, claim class, rate year)` — the estimate is shared across every output row that maps to it: different modifier / facility / payer rows for the same key get the same `1 / COVERAGE_ESTIMATE` factor applied to their individual observed values.

The point estimate is `PROJECTED_UNITS = OBSERVED_UNITS / COVERAGE_ESTIMATE`. The bootstrap, used only for `PROJECTED_UNITS_P5` and `PROJECTED_UNITS_P95`, adds observation noise (Negative Binomial around `OBSERVED_UNITS`) and coverage noise (lognormal around `COVERAGE_ESTIMATE`) to characterize the uncertainty around that point.

#### 1.1 What `OBSERVED_UNITS` is — and what it is not

`OBSERVED_UNITS` is the unit count from the **source claims feed (PurpleLab) only**. It is not a pooled or deduplicated combination of multiple data sources. The pipeline ingests several other datasets, but each plays a different role and none of them adds to `OBSERVED_UNITS`:

* The provider directory (NPPES) attaches geography and taxonomy to each NPI.
* T-MSIS Medicaid Provider Spending is the denominator for the Medicaid direct tiers.
* The CMS Physician & Other Practitioners PUF is the denominator for the Medicare FFS state benchmark and the yardstick for the Commercial capture calibration.
* The CMS Medicare Inpatient (by Provider and Service) PUF is the denominator for the Medicare FFS inpatient tiers.
* The CMS Outpatient PUF (+ OPPS Addendum B crosswalk) is the denominator for the Medicare FFS outpatient tier.
* The CMS Geographic Variation PUF supplies county MA participation rates for the MA-scaled tier.
* The CMS Physician provider-level rollup is used only in QA (per-NPI envelope) and never enters the projection math.

The reference datasets shape `COVERAGE_ESTIMATE`, which determines how `OBSERVED_UNITS` is scaled to a universe estimate. `OBSERVED_UNITS` itself is always a single-source quantity. This distinction matters when reconciling counts: any reader who tries to add observed PurpleLab units to T-MSIS or PUF totals is double-counting.

#### 1.2 Which PurpleLab field underlies `OBSERVED_UNITS`

The PurpleLab `CLAIMS_ORDERED` feed exposes two volume fields per row:

* **`ATTRIBUTED_TOTAL_UNITS`** — units on the original-claim grain, attributed to the billing NPI on that row. Populated for \~94% of professional rows and \~97% of institutional rows.
* **`COUNT_OF_UNITS`** — units sourced from a matched remittance advice. Only populated for \~13–16% of rows (the remit-matched subset).

We sum `ATTRIBUTED_TOTAL_UNITS` because it reflects the full claim population in the feed. Rows where `ATTRIBUTED_TOTAL_UNITS` is null are remits without a matched open claim and are skipped — they have no billing-NPI grain row to attribute to. Rows where it is zero are likewise excluded (they carry no volume to project). This is implemented in `stages/ingest.py`.

#### 1.3 Ratio metrics: like-for-like coverage

Coverage is a ratio of "what we saw" to "what happened", and the two sides must count the same thing:

| Tier family           | Numerator (PurpleLab)    | Denominator (administrative) | Why                                                                                                                                                                                                                             |
| --------------------- | ------------------------ | ---------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| T-MSIS (Medicaid)     | `ATTRIBUTED_CLAIM_COUNT` | T-MSIS `TOTAL_CLAIM_LINES`   | T-MSIS reports claim lines per HCPCS. Units-per-line for drug J-codes run into the hundreds; a units numerator drove raw coverage far above 1.0 and clamped to no-projection. Claims vs lines is the closest available pairing. |
| Medicare inpatient    | `ATTRIBUTED_CLAIM_COUNT` | CMS `Total_Discharges`       | An inpatient stay maps to a claim, not to revenue-line units.                                                                                                                                                                   |
| Medicare professional | `ATTRIBUTED_TOTAL_UNITS` | PUF `Tot_Srvcs`              | CMS counts `Tot_Srvcs` as units for drug codes and services otherwise — the closest like-for-like for a professional-claim feed.                                                                                                |
| Medicare outpatient   | `ATTRIBUTED_TOTAL_UNITS` | Outpatient PUF `CAPC_Srvcs`  | APC service counts are line-level services.                                                                                                                                                                                     |

On the T-MSIS pairing, one residual mismatch is disclosed rather than hidden: within an HCPCS cell, a claim can carry the same code on more than one line, so lines ≥ claims and the claims-vs-lines ratio _understates_ coverage — which _overstates_ the projection for the affected cells. (Earlier versions of this document called that direction "conservative"; for market sizing it is not, and we no longer characterize it that way.) T-MSIS does not publish claim counts, so the mismatch cannot be eliminated with public data; the QA units-per-claim report (§8) monitors its plausible magnitude, and codes where the effect is extreme surface in the raw-coverage diagnostics.

The ratio metric used by each tier is recorded in `COVERAGE_GRAIN` so a reviewer never has to guess. The resulting factor `1 / COVERAGE_ESTIMATE` is dimensionless (a capture share) and is applied uniformly to units, claim counts, and charge dollars. This assumes captured and uncaptured claims share their units-per-claim and charge-per-unit mix; the QA units-per-claim distribution by tier monitors that assumption.

### 2. Inputs

| Input              | Source                     | Grain                                                                                                                                                                                                | Notes                                                                                                                                                                                                                                                                                                                                                                                                                     |
| ------------------ | -------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Source claims      | PurpleLab `CLAIMS_ORDERED` | one row per `BILLING_NPI_NBR × FACILITY_NPI_NBR × PROCEDURE_CD × PROCEDURE_MODIFIER × PAYER_ID × PAYER_SUBCHANNEL_NAME × PAYER_CHANNEL_NAME × CLAIM_TYPE_CD × SETTING × RATE_YEAR` after aggregation | Includes professional and institutional (`CLAIM_TYPE_CD = 'P' / 'I'`); `SETTING` resolves to `Office` / `Outpatient` / `Inpatient` / `Other`. Volume sourced from `ATTRIBUTED_TOTAL_UNITS` (see §1.2); claim count from `ATTRIBUTED_CLAIM_COUNT`; charge dollars from `ATTRIBUTED_CHARGE_AMT`. `FACILITY_NPI_NBR` is sparsely populated (mostly on institutional rows). `PROCEDURE_MODIFIER` is NULL for unmodified codes |
| Provider directory | Enriched NPPES (Type 2)    | one row per organization NPI                                                                                                                                                                         | Includes county FIPS, primary taxonomy, CBSA, lifetime claim count                                                                                                                                                                                                                                                                                                                                                        |

Claims are joined to the provider directory on billing NPI to attach county, primary taxonomy, and CBSA before any aggregation; the join also restricts the projection universe to active Type 2 organizational NPIs (see §10). Ingest additionally classifies each row for coverage-tier eligibility: `subch_class` (`ffs` / `ma` / `other` within the Medicare channel, driven by configured subchannel-name lists; `all` elsewhere) and `claim_class` (`prof` / `drg` / `op` / `other_inst`). Medicare rows whose subchannel matches neither configured list only qualify for the channel prior, and the QA report itemizes them.

### 3. Reference data sources

**Single source of truth (V4.2).** Every reference is built from versioned releases in the Payerset data lake, which is populated by dedicated Medicare and Medicaid capture pipelines. The projection no longer fetches anything directly from CMS or HHS; a single orchestrator pulls the pinned data-lake release and builds the slim reference files the run consumes. All references except the seed channel priors are optional: when a file is absent the loader registers an empty view and the corresponding tier produces zero rows, with the affected cells falling through to lower tiers.

| Reference                                          | Datalake object                                                                                                                                                                                   | Used for                                                                                                                                | Grain                                            |
| -------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------ |
| Channel priors (seed)                              | shipped in-repo (`data/seed/channel_priors.csv`)                                                                                                                                                  | Fallback per-channel coverage prior (Commercial 0.60, Medicare 0.75, Medicaid 0.80); the effective prior is empirical by default (§6.9) | national × channel                               |
| T-MSIS Medicaid Provider Spending                  | `MEDICAID/<RELEASE>/PROVIDER_SPENDING_BY_HCPCS.parquet`                                                                                                                                           | Denominator for `tmsis_npi_direct` and the `tmsis_direct` county fallback                                                               | NPI × HCPCS × year (claim lines)                 |
| CMS Physician & Other Practitioners PUF            | `MEDICARE/<RELEASE>/PHYSICIAN_PUF.parquet` (data year 2024)                                                                                                                                       | Denominator for `medicare_ffs_state_benchmark`; yardstick for the Commercial capture calibration; QA cross-check                        | NPI × HCPCS (FFS, single vintage year)           |
| CMS Medicare Inpatient (by Provider & Service) PUF | `MEDICARE/<RELEASE>/INPATIENT_DATA.parquet` — see [Payerset Medicare Inpatient methodology](https://docs.payerset.com/using-the-payerset-platform/data-dictionary/medicare-inpatient-methodology) | Denominator for `medicare_inpatient_npi_direct` / `direct_inpatient_benchmark`                                                          | NPI × MS-DRG (FFS Part A discharges)             |
| CMS Outpatient PUF + OPPS Addendum B               | `MEDICARE/<RELEASE>/OUTPATIENT_PUF.parquet` + `OPPS_ADDENDUM_B.parquet`                                                                                                                           | Denominator for `medicare_outpatient_benchmark`                                                                                         | county × APC (FFS services), HCPCS→APC crosswalk |
| CMS Geographic Variation PUF                       | `MEDICARE/<RELEASE>/GEO_VARIATION_PUF.parquet`                                                                                                                                                    | County MA participation rates for `ma_scaled_benchmark`                                                                                 | county                                           |
| CMS Physician provider-level rollup                | `MEDICARE/<RELEASE>/PHYSICIAN_PROVIDER_PUF.parquet`                                                                                                                                               | QA per-NPI envelope only                                                                                                                | NPI totals                                       |

The PUF vintage is data year 2024 as of the current datalake release, matching the capture reference year — the 2023-vintage growth residual documented in the V4.1 QA (+0.082 signed delta) is removed by construction. `references.cms_puf_year` records the data year and must be updated when a new PUF vintage lands in the lake.

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

present in the claims grain, the pipeline assigns one coverage estimate by walking the tier hierarchy described in §6 (first match wins). The output is a row-level `coverage_rowmap` table on that key; projection joins it back to every claims row. A per-source summary (`coverage_cells`), the state-benchmark code screen (`medffs_code_screen`), and the channel-prior provenance table (`channel_priors_effective`) feed QA and the release notes.

#### Stage 3 — Projection and bootstrap

For each cell, the deterministic point estimate is `PROJECTED_UNITS = OBSERVED_UNITS / COVERAGE_ESTIMATE`, with `COVERAGE_ESTIMATE` clamped to `[min_coverage, max_coverage]`. Because `max_coverage = 1.0`, the implied factor `1 / COVERAGE_ESTIMATE` is in `[1.0, 1 / min_coverage]` and `PROJECTED_UNITS >= OBSERVED_UNITS` on every projectable row by construction.

Confidence intervals (`PROJECTED_UNITS_P5`, `PROJECTED_UNITS_P95`) are produced by a parametric bootstrap that combines two noise sources:

* **Observation noise:** Negative Binomial around `OBSERVED_UNITS` with per-cell dispersion `k = mu / obs_phi` (default `obs_phi = 0.10`), i.e. variance `= mu × (1 + obs_phi)`. Relative SD falls as `1/sqrt(mu)` with a constant overdispersion multiplier. (V3 used a constant `k = 10`, which floored the relative SD at \~32% even for cells with hundreds of thousands of observed units and made P95 unrealistically wide on large cells.)
* **Coverage noise:** Lognormal around `COVERAGE_ESTIMATE`, parameterised so that `E[c] ≈ coverage_estimate` and `sd(c)/E[c] ≈ rel_sd`, then clipped to `[min_coverage, max_coverage]`.

The relative SD on the coverage draw is chosen by tier (see `coverage.coverage_rel_sd` in config): NPI-direct tiers 0.10, county-fallback tiers and the state benchmark 0.15, outpatient 0.20, calibrated Commercial 0.25/0.35, MA-scaled 0.30, channel prior 0.50. The rel SD encodes our prior on how noisy each tier is; the published intervals are heuristic uncertainty bands, not calibrated frequentist intervals. Per draw, `projected_draw = observed_draw / coverage_draw`. `PROJECTED_UNITS_P5` and `PROJECTED_UNITS_P95` are the 5th and 95th percentiles across the draws; the median is not published, since the deterministic point estimate already serves that role.

**Hard invariants: `PROJECTED_UNITS / P5 / P95 >= OBSERVED_UNITS`, and `P5 <= PROJECTED_UNITS <= P95`.** The deterministic point already satisfies the first because `COVERAGE_ESTIMATE` is clamped to `<= max_coverage = 1.0`. The bootstrap percentiles can otherwise drift slightly below `OBSERVED_UNITS` when `COVERAGE_ESTIMATE` is at the 1.0 ceiling — lognormal coverage draws clipped at the ceiling are biased below 1.0, and the Negative Binomial observation noise is right-skewed for low counts. We therefore floor `PROJECTED_UNITS_P5` and `PROJECTED_UNITS_P95` at `OBSERVED_UNITS` after the bootstrap, and additionally cap `P95` at no less than the point estimate so the published interval always contains it (this binds only on degenerate cells with fractional observed units below \~0.1, where the integer NB draws are almost all zero).

The bootstrap is parallelised: stages 1–2 are pure DuckDB SQL and use DuckDB's intra-query thread pool (`run.threads`); stage 3 dispatches projection batches to a `concurrent.futures.ProcessPoolExecutor` (`run.workers`). Determinism is enforced end to end (V4.2): the projection input is streamed under a total ORDER BY on the (unique) projection grain, per-batch RNGs are derived from `(seed, batch_idx)` via `numpy.random.SeedSequence`, and completed batches are written through an in-order reorder buffer — so output is byte-identical across runs and across worker counts (§11).

#### Stage 4 — QA

See §8.

### 5. Channel attribution

Each service line is attributed to one of three channels — `Commercial`, `Medicare`, or `Medicaid` — using the source claim's `PAYER_CHANNEL_NAME` directly. Non-projected channels (e.g. "Other", "Dual (Medicaid/Medicare)") are filtered out at ingest and do not appear in the output. Channel attribution is done at ingest and is not revisited downstream. Within the Medicare channel, `PAYER_SUBCHANNEL_NAME` drives the FFS / MA split used by the coverage tiers (§2).

### 6. Coverage hierarchy

The coverage estimate for a cell is selected by the first tier (top to bottom) that the cell qualifies for.

**Denominator-side gates (V4.2).** Direct tiers qualify on the size of their administrative _denominator_ (`coverage.direct_min_denominator`, default 20 — claim lines, discharges, or services at the tier's own grain); county fallbacks additionally require `>= min_matched_npis` (default 3) distinct matched NPIs. Earlier versions gated on _observed_ volume, which conditioned tier entry on the numerator: at a fixed cell size, high-capture cells cleared the gate while low-capture cells fell through to pools estimated from the high-capture survivors — a selection bias in exactly the quantity being estimated. Gating on the denominator is independent of capture.

Every tier records `raw_coverage` (the pre-clamp ratio); values materially above 1.0 signal a denominator mismatch and are surfaced in QA (§8).

**Trust filter.** A direct denominator implying raw coverage above `coverage.raw_trust_max` (default 1.5) is treated as evidence that the source does not measure that cell's universe — most commonly billing-org vs rendering-provider attribution mismatch against the CMS PUFs (PurpleLab attributes volume to the billing organization; CMS attributes to the rendering NPI, so an org billing for many individual renderers shows observed ≫ "truth"), or source-side suppression / vintage incompleteness. Such cells are _not_ clamped to 1.0; the pair is discarded and the cell falls through to the next tier. The same pair-level filter applies inside the matched-NPI county pools and the Commercial capture calibration, so a mismatched provider can neither anchor its own cells nor distort a pooled estimate. Raw coverage between 1.0 and the trust ceiling is treated as noise and clamps to 1.0 as before.

**T-MSIS completeness cap and year matching (V4.2).** The latest T-MSIS year is typically preliminary; `references.tmsis_max_complete_year` (default 2023 in the shipped config) caps the denominator year below the file's maximum. The Medicaid capture _numerator_ is measured at that same claims year, so numerator and denominator always describe the same calendar year and cross-year volume growth cannot leak into the capture rate. (If the claims feed has no rows for that year, the pipeline falls back to the global reference year and flags the mismatch in QA.) Set the cap to null to use the newest data and compare the per-year raw-coverage distributions in QA before trusting it.

**The estimand.** Coverage means "the share of this provider's true volume that the source feed captured." Capture is a property of the provider's claims-routing (which clearinghouses feed the source), so it varies more across providers than across payers or procedures. The hierarchy therefore measures capture at the provider grain wherever administrative truth names the provider, and generalizes outward only when it must.

**Matched-NPI county fallbacks.** Where the denominator source is NPI-grained (T-MSIS, Inpatient PUF), the county fallback is a _matched_ ratio-of-sums: both numerator and denominator are restricted to NPIs present on both sides for that procedure. This removes the asymmetry of comparing a Type-2-only numerator against an all-provider denominator (which biased coverage low and over-projected), and makes the county estimate "pooled capture among verifiable providers," which is then applied to the county's unmatched providers.

#### 6.1 `tmsis_npi_direct` _(Medicaid)_

Applied when the billing NPI × HCPCS exists in T-MSIS at the Medicaid capture year with at least `direct_min_denominator` claim lines.

```
medicaid_year     = min(capture_reference_year, tmsis_max_complete_year)
COVERAGE_ESTIMATE = observed_claims(npi, hcpcs, medicaid_year)
                    / tmsis_claim_lines(npi, hcpcs, medicaid_year)
```

The rate is applied to every rate year of that NPI × HCPCS (§1). `CONFIDENCE_BUCKET` is `high` if `observed_units >= high_confidence_min_units` (default 100), otherwise `medium` — the confidence label uses observed volume, but tier _qualification_ never does. T-MSIS is treated as administrative truth for the Medicaid universe up to CMS suppression rules (cells with fewer than 11 beneficiaries are suppressed at source) and state-by-state managed-care encounter completeness (see §10).

#### 6.2 `tmsis_direct` _(Medicaid, county fallback)_

Matched-NPI ratio-of-sums at county × HCPCS at the Medicaid capture year, applied to Medicaid cells whose NPI is not usable in T-MSIS for that HCPCS (absent, below the denominator gate, or trust-filtered).

#### 6.3 `medicare_inpatient_npi_direct` _(Medicare FFS, inpatient)_

Applied to Medicare FFS institutional-inpatient (MS-DRG) cells when the billing NPI × DRG exists in the CMS Inpatient PUF with at least `direct_min_denominator` discharges. Ratio: observed claims vs discharges at the reference year. The Inpatient PUF is facility-grain, so the NPI match is same-role and trustworthy (unlike the Physician PUF; see §6.5).

#### 6.4 `direct_inpatient_benchmark` _(Medicare FFS, inpatient county fallback)_

Matched-NPI ratio-of-sums at county × DRG against the Inpatient PUF.

#### 6.5 `medicare_ffs_state_benchmark` _(Medicare FFS, professional)_

Cross-role NPI matching against the _Physician_ PUF (billing org ↔ rendering individual) is unreliable in both directions: orgs billing for individual renderers show observed ≫ PUF (caught by the trust filter), and fragmented orgs show observed ≪ PUF, which is indistinguishable from low capture and biased the V4.0 per-NPI / county tiers toward over-projection (the V4.0 county benchmark applied 5.08× in NJ where measured truth was 1.29×). Professional FFS coverage therefore uses the state aggregate, where attribution washes out.

State capture is the **trimmed pooled ratio** at the reference year:

```
capture(state) = Σ observed_units / Σ puf_tot_srvcs
```

over (state × procedure) pairs gated on a PUF service floor (`state_benchmark_min_pair_services`, default 1000) and a minimum reporting-NPI count (`state_benchmark_min_pair_npis`, default 10 — attribution washes out only when enough renderers contribute). States need `>= state_benchmark_min_pairs` (default 20) qualifying pairs; otherwise their FFS professional cells fall through to the channel prior.

Two trims remove metric-inconsistent volume before pooling:

* **Per-pair upper guard:** pairs with `observed > raw_trust_max × PUF` are dropped (unit mismatch inflating the ratio).
* **Two-sided code screen (V4.2):** unit-definition mismatch is a property of the _code_, not the state. The national pooled ratio per code is compared against the national all-code pooled ratio; codes outside `[pooled / k, pooled × k]` (`state_benchmark_code_trim_k`, default 3) are excluded from every state pool — in both directions. The low side matters as much as the high side: code families where the PUF counts more units than the feed (e.g. per-drug-unit `Tot_Srvcs`) dragged pooled capture down and over-projected. The screen is published as `medffs_code_screen` and summarized in QA.

Pooling (volume weighting) is what makes projected state totals equal PUF totals on validatable codes **by construction** — the property market sizing needs. Two rejected alternatives, for the record: an untrimmed pooled ratio was inflated toward 1.0 by unit-mismatched codes (under-projected), and a median-of-pairs scalar aligned the median pair but over-projected totals \~18% in the median state because capture correlates with code volume. `CONFIDENCE_BUCKET` is `medium`; bootstrap rel SD 0.15.

#### 6.6 `medicare_outpatient_benchmark` _(Medicare FFS, outpatient)_

Applied to Medicare FFS institutional-outpatient cells whose HCPCS maps to an APC (OPPS Addendum B, payable status indicators only). Observed units roll up to county × APC and are compared against CMS Outpatient PUF `CAPC_Srvcs` (CCN-grain, geocoded to county). The source is CCN-grain, so no matched-NPI restriction is possible; this is a plain ratio-of-sums benchmark gated on `ffs_services >= direct_min_denominator`.

#### 6.7 `ma_scaled_benchmark` _(Medicare Advantage)_

MA has no public per-provider denominator. Where a county FFS denominator exists (professional, inpatient, or outpatient), the MA universe is estimated as

```
ma_denominator = ffs_denominator_county × ma_rate / (1 − ma_rate)
```

with `ma_rate` the county MA participation rate from the CMS Geographic Variation PUF. Published rates outside `[0.05, 0.85]` are **clamped** to those bounds (V4.2 — previously such counties were dropped, which sent exactly the highest-MA counties to the channel prior; clamping keeps them on the locally-measured FFS base). The count of clamped counties is reported in QA. This tier assumes similar per-enrollee utilization across FFS and MA within a county — a documented approximation (§10). The FFS denominator here is the _full-county_ total (all provider types), since the MA universe is not restricted to providers we can match. `CONFIDENCE_BUCKET` is `medium`; bootstrap rel SD 0.30.

#### 6.8 `commercial_calibrated` / `commercial_calibrated_pooled` _(Commercial — shadow mode)_

Capture rate is a property of the provider's clearinghouse footprint, not the payer. Where Medicare FFS truth exists for an NPI, we measure the feed's capture directly:

```
capture(npi) = Σ observed_ffs_units(npi, hcpcs) / Σ puf_tot_srvcs(npi, hcpcs)
```

summed over matched HCPCS at the capture reference year, and gated on `Σ puf_tot_srvcs >= min_puf_services` (default 100) and `>= min_matched_codes` (default 3) matched codes. NPIs that fail the gates fall to a state × taxonomy pooled capture (requires `>= pooled_min_npis` calibrated NPIs, default 25).

**These tiers run in shadow mode in the shipped config** (`commercial_calibration.apply: false`): the per-NPI yardstick shares the Physician PUF's attribution-fragmentation problem (§6.5), so applied per-NPI calibration over-projects Commercial. The capture distribution is computed and reported in QA every release, but Commercial projects at the channel prior — which, in empirical mode (§6.9), is the _aggregate_ Medicare FFS capture, i.e. the same payer-agnostic-capture reasoning applied at the grain where attribution is trustworthy. When `apply: true`, `CONFIDENCE_BUCKET` is `medium` (NPI-level) / `low` (pooled).

#### 6.9 `channel_prior`

Applied to every remaining cell whose pooled signal (county × procedure × channel × year; NPI-pooled when the county is unknown) is at least `peer_min_observed_units` (default 5).

**Empirical priors (V4.2).** With `coverage.channel_priors.mode: empirical` (the shipped default), the prior for each channel is re-derived every run from the measured tiers, instead of using a hand-set constant that the measured tiers were contradicting:

| Channel    | Empirical prior definition                                                                                                                                           |
| ---------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Medicaid   | Pooled measured capture of the T-MSIS tiers: `Σ observed / Σ projected` over measured Medicaid cells at the Medicaid capture year                                    |
| Medicare   | Pooled measured capture of the FFS-measured tiers (state benchmark + inpatient + outpatient) at the reference year                                                   |
| Commercial | National aggregate Medicare FFS capture (`Σ observed / Σ PUF` over the trimmed, screened state pools) — the payer-agnostic-capture assumption at the aggregate grain |

A channel whose measured pool is below `channel_priors.empirical_min_units` observed units (default 50,000) falls back to its seed value from `data/seed/channel_priors.csv` (Commercial 0.60, Medicare 0.75, Medicaid 0.80); `mode: seed` restores the seed values entirely and is the documented rollback path. The effective prior, its provenance, the seed value, and the pool size are published in the QA report, and provenance is recorded per row in `COVERAGE_GRAIN` (`national|channel|empirical` vs `national|channel|seed`).

The MA-scaled tier is deliberately excluded from the Medicare pool (its denominator is itself modeled, §6.7), and the per-NPI Commercial calibration is excluded from the Commercial prior (its known fragmentation bias is the reason it is in shadow, §6.8).

`CONFIDENCE_BUCKET` is `low`.

#### 6.10 `suppress`

Cells below the signal threshold with no administrative-truth denominator are not projected. They appear in the output with `PROJECTED_UNITS / P5 / P95` all NULL, `COVERAGE_SOURCE = 'suppress'`, and `CONFIDENCE_BUCKET = 'suppress'`. Downstream consumers can decide whether to filter them or pass them through.

### 7. Output schema

Output is a single streamed parquet file at `output_root/part-00000.parquet` (zstd compression), fully ordered on the projection grain. `PROVIDER_STATE` and `RATE_YEAR` are ordinary columns, so no information was lost by dropping the earlier hive partitioning — it only added consolidation overhead downstream (`output.partitioning: "state_year"` restores the legacy `state=XX/year=YYYY/` layout if ever needed). Each row corresponds to one cell at the projection grain (see §1). The column set is unchanged from V3; `COVERAGE_SOURCE` and `COVERAGE_GRAIN` carry the tier vocabulary of §6.

| Column                             | Type              | Description                                                        |
| ---------------------------------- | ----------------- | ------------------------------------------------------------------ |
| `BILLING_NPI_NBR`                  | int64             | Rendering / billing provider NPI                                   |
| `FACILITY_NPI_NBR`                 | int64 (nullable)  | Service-facility NPI when present on the source claim              |
| `PROCEDURE_CD`                     | string            | HCPCS / CPT code (or MS-DRG for inpatient rows)                    |
| `PROCEDURE_CD_DESC`                | string            | PurpleLab's short description for `PROCEDURE_CD`                   |
| `PROCEDURE_MODIFIER`               | string (nullable) | Procedure modifier; NULL if none                                   |
| `PROCEDURE_MODIFIER_DESC`          | string (nullable) | Description for `PROCEDURE_MODIFIER`                               |
| `PAYER_ID`                         | int64             | Source payer identifier                                            |
| `PAYER_NAME`                       | string            | Payer name as reported in PurpleLab                                |
| `PAYER_SUBCHANNEL_NAME`            | string            | Subchannel within `PAYER_CHANNEL_NAME`                             |
| `PAYER_CHANNEL_NAME`               | string            | `Commercial` \| `Medicare` \| `Medicaid`                           |
| `CLAIM_TYPE_CD`                    | string            | `P` \| `I`                                                         |
| `SETTING`                          | string            | `Office` \| `Outpatient` \| `Inpatient` \| `Other`                 |
| `OBSERVED_UNITS`                   | float64           | Source-claims unit count                                           |
| `PROJECTED_UNITS`                  | float64           | `OBSERVED_UNITS / COVERAGE_ESTIMATE`                               |
| `PROJECTED_UNITS_P5`               | float64           | 5th percentile (bootstrap), floored at `OBSERVED_UNITS`            |
| `PROJECTED_UNITS_P95`              | float64           | 95th percentile (bootstrap), floored at `OBSERVED_UNITS`           |
| `ATTRIBUTED_CLAIM_COUNT`           | int64             | Source-claims claim count                                          |
| `ATTRIBUTED_CLAIM_COUNT_PROJECTED` | float64           | `ATTRIBUTED_CLAIM_COUNT / COVERAGE_ESTIMATE`                       |
| `ATTRIBUTED_CHARGE_AMT`            | float64           | Source-claims billed-charge dollars                                |
| `ATTRIBUTED_CHARGE_AMT_PROJECTED`  | float64           | `ATTRIBUTED_CHARGE_AMT / COVERAGE_ESTIMATE`                        |
| `COVERAGE_ESTIMATE`                | float64           | Coverage fraction used to scale observed → projected               |
| `COVERAGE_SOURCE`                  | string            | Tier name (see §6)                                                 |
| `COVERAGE_GRAIN`                   | string            | Grain + ratio metric (+ prior provenance) of the coverage estimate |
| `CONFIDENCE_BUCKET`                | string            | `high` \| `medium` \| `low` \| `suppress`                          |
| `PROVIDER_COUNTY_FIPS`             | string            | 5-digit county FIPS (from NPPES)                                   |
| `PROVIDER_STATE`                   | string            | Two-letter state (from NPPES)                                      |
| `DATA_PERIOD_START`                | date32            | Min `REPORT_DD` over the cell                                      |
| `DATA_PERIOD_END`                  | date32            | Max `REPORT_DD` over the cell                                      |
| `CALIBRATION_VINTAGE`              | string            | Run identifier from `config.yaml`                                  |
| `RATE_YEAR`                        | int64             | Source `RATE_YEAR`                                                 |

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
4. **Commercial calibration report** — per-NPI capture distribution (p10–p90, mean, share with raw capture > 1), Commercial volume by calibration source, and the implied shift vs the applied prior.
5. **Medicare FFS cross-check.** For every (state × procedure) with both FFS-subchannel projections and a CMS Physician PUF total at the reference year, the relative delta `(projected − puf) / puf` — restricted to pairs with `puf_total_services > qa.puf_min_services` (default 1000) and `>= qa.puf_min_npis` (default 10) reporting NPIs. Reports `observed_over_puf` (the direct capture measurement), the signed median delta (bias), and `medicare_ffs_totals_ratio_by_state_median` (per-state Σprojected/ΣPUF over validatable pairs — the market-sizing pass bar, ≈1.0 by construction of the pooled benchmark; it validates construction, not external accuracy).
6. **Per-NPI envelope.** Projected Medicare FFS professional volume per NPI vs the provider-level PUF total; NPIs whose ratio exceeds `qa.npi_envelope_flag` (default 3.0) are surfaced.
7. **Channel prior provenance (V4.2)** — per channel: seed prior, empirical prior, effective prior, provenance, and measured pool size. The empirical-vs-seed delta is the headline sensitivity of the release.
8. **State-benchmark code screen (V4.2)** — codes kept/excluded by the two-sided screen and the PUF volume excluded; top screened-out codes listed.
9. **Capture drift by rate year (V4.2)** — per-state pooled observed/PUF ratio for every claims year on the screened code set. Complete years should cluster near the applied capture if capture is truly a rate; partial years sit lower by their completeness fraction (expected).
10. **Units-per-claim by coverage source (V4.2)** — monitors the uniform-factor assumption of §1.3.
11. **MA rate clamping (V4.2)** — counties whose published MA rate fell outside the scaling bounds.
12. **Capture years (V4.2)** — the reference, T-MSIS denominator, and Medicaid capture years actually used, with a warning when the Medicaid numerator and denominator years diverge.

Manual checks before each release: review the raw\_coverage flags, the channel-prior provenance and empirical-vs-seed deltas, the code-screen exclusions, the calibration distribution, all flagged cross-check pairs, the capture-drift table, and the envelope list.

### 9. Reference run results

V4.2 reference run, calibration vintage `20260815`, full PurpleLab feed (`CLAIMS_ORDERED_202604` extract), executed 2026-08-16. Capture reference year 2024; CMS PUF data year 2024; T-MSIS denominator year 2023; Commercial calibration in shadow mode; channel priors empirical.

#### 9.1 Headline

| Metric         | Observed           | Projected           | Uplift |
| -------------- | ------------------ | ------------------- | ------ |
| Units          | 67,204,206,939     | 178,170,766,718     | 2.651× |
| Claims         | 11,149,227,303     | 27,071,468,255      | 2.428× |
| Billed charges | $8,564,064,338,136 | $20,889,865,032,281 | 2.439× |

Output rows: 380,593,164 across 485,512 active billing NPIs. The step up from V4.1's 2.067× unit uplift is dominated by the empirical channel priors: the prior tier (75.5% of observed units) now projects at measured capture (Medicare 0.516, Commercial 0.488, Medicaid 0.236) instead of the optimistic seed constants (0.75 / 0.60 / 0.80) that the V4.1 run's own measurements already contradicted. The denominator-side gates, two-sided code screen, and 2024 PUF vintage contribute the remainder.

#### 9.2 Coverage source mix

Shares are of observed units; uplifts are per-tier projected/observed over projectable rows.

| Coverage source                 | Output rows | Share of observed units | Unit uplift | Claim uplift |
| ------------------------------- | ----------- | ----------------------- | ----------- | ------------ |
| `channel_prior`                 | 291,790,967 | 75.49%                  | 2.284×      | 2.186×       |
| `tmsis_npi_direct`              | 17,007,436  | 11.43%                  | 5.086×      | 3.217×       |
| `medicare_ffs_state_benchmark`  | 32,643,755  | 9.23%                   | 1.933×      | 2.014×       |
| `tmsis_direct`                  | 18,041,017  | 3.42%                   | 4.309×      | 4.579×       |
| `ma_scaled_benchmark`           | 5,123,754   | 0.38%                   | 5.068×      | 4.498×       |
| `medicare_outpatient_benchmark` | 1,024,460   | 0.01%                   | 1.897×      | 1.899×       |
| `direct_inpatient_benchmark`    | 336,942     | <0.01%                  | 3.448×      | 3.448×       |
| `medicare_inpatient_npi_direct` | 140,419     | <0.01%                  | 2.879×      | 2.879×       |
| `suppress`                      | 14,484,414  | 0.03%                   | —           | —            |

#### 9.3 QA outcomes

| Check                                                                       | Result                                                                                          | Disposition                                                                                                                      |
| --------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| Channel priors (empirical)                                                  | Medicare 0.516 / Commercial 0.488 / Medicaid 0.236, pools 2.9B / 1.2B / 4.5B observed units     | All three channels on measured priors; seed deltas −0.23 / −0.11 / −0.56                                                         |
| Medicare FFS totals alignment (`medicare_ffs_totals_ratio_by_state_median`) | 0.989                                                                                           | Pass (≈1.0 by construction of the trimmed-pooled state benchmark)                                                                |
| Medicare FFS vs PUF signed median delta (professional, reference year)      | +0.051                                                                                          | Pass (bar ±0.10); improved from V4.1's +0.082 by the 2024 PUF vintage                                                            |
| Direct FFS capture measurement (`observed_over_puf`, pair median)           | 0.509                                                                                           | Consistent with the applied state captures                                                                                       |
| Capture drift by year (52 states)                                           | 2024 median 0.479 (p10 0.318, p90 0.660); 2025 median 0.449                                     | Rate assumption holds within \~6% across complete years; 2026 partial at 0.117 as expected                                       |
| Two-sided code screen                                                       | 84 of 2,071 codes excluded                                                                      | Published in `medffs_code_screen`                                                                                                |
| Unclassified Medicare subchannel volume                                     | 212M units (\~0.2% of Medicare)                                                                 | `Medicare / Unspecified` + `Part D`; projects at the channel prior by design                                                     |
| Medicaid capture years                                                      | numerator 2024 vs T-MSIS 2023 (flagged)                                                         | Expected: the feed's `RATE_YEAR` starts at 2024, so no 2023 numerator exists; resolves when a complete 2024 T-MSIS vintage lands |
| Output invariants (row-level, all 380.6M rows)                              | projected ≥ observed; P5 ≤ point ≤ P95; coverage within \[0.05, 1.00]; suppress rows fully NULL | Pass                                                                                                                             |
| Commercial calibration                                                      | shadow                                                                                          | Capture distribution reported in QA; Commercial projects at the empirical channel prior                                          |

Pair-level absolute deltas in the FFS cross-check remain wide (23,617 of 35,993 gated pairs above the 0.30 flag): capture heterogeneity across procedures is real, and a state-scalar coverage cannot remove it. The check's role across releases is drift detection on the signed median and the totals ratio.

### 10. Limitations

* **Coverage is a model, not a measurement.** Direct tiers are subject to source-side suppression, vintage drift, and reporting completeness. We bound the coverage estimate to `[0.05, 1.00]`, surface confidence labels, and publish raw-coverage diagnostics every release.
* **T-MSIS managed-care completeness varies by state.** States with weak encounter reporting under-state the Medicaid denominator, biasing coverage high and projections low in those states. The raw\_coverage distribution by state is the monitoring tool; a state-level completeness adjustment is future work.
* **T-MSIS publishes claim lines, not claims.** Within an HCPCS cell, lines ≥ claims, so the claims-vs-lines ratio understates coverage and overstates the projection for codes that repeat on multiple lines of one claim (see §1.3). Public data cannot eliminate this; the units-per-claim QA slice monitors it.
* **The latest T-MSIS year is preliminary.** Rate years beyond the latest complete T-MSIS year reuse the latest complete denominator, and the capture numerator is measured at that same year (V4.2).
* **Capture-as-a-rate rests on year stability.** Measured on two complete years to date; the QA capture-drift table re-measures it every release.
* **MA scaling assumes FFS-like per-enrollee utilization.** The `ma_scaled_benchmark` denominator is an enrollment-scaled FFS universe; MA plan-mix and utilization-management differences are not modeled. The tier carries a wide bootstrap rel SD (0.30) and `medium` confidence for this reason.
* **Commercial coverage rests on the payer-agnostic-capture assumption.** In V4.2 the aggregate FFS capture serves as the Commercial prior; providers whose Commercial claims route differently from their Medicare FFS claims violate the assumption. The per-NPI calibration (shadow) and its QA distribution monitor this.
* **Bootstrap intervals are heuristic bands, not calibrated intervals.** Observation noise is a stand-in for feed-resampling variability, and coverage rel SDs are configured per tier, not estimated from data.
* **The projection is per-NPI, not per-patient.** We do not deduplicate patients across NPIs.
* **The projection universe is Type 2 billing NPIs.** Claims billed under individual (Type 1) NPIs are excluded at ingest. County-fallback coverage is estimated on matched (verifiable) NPIs and generalized to unmatched Type 2 NPIs in the county; it is an estimate of capture among the published universe, not of the all-provider universe. Consumers should not read county sums of projections as "all services in the county."
* **Each organization NPI carries a single NPPES county.** Multi-site organizations concentrate in one county, which blurs county-grain pools and MA scaling. State- and NPI-grain tiers are unaffected.
* **Channel attribution depends on `PAYER_CHANNEL_NAME` / `PAYER_SUBCHANNEL_NAME` in the source feed.** Unclassified Medicare subchannels project at the channel prior and are itemized in QA.
* **No fully external validation source.** The Physician PUF is both a calibration input and the QA yardstick, so the totals-ratio pass bar validates construction rather than independent accuracy. Candidate external checks (state APCDs, hospital cost-report volumes, a held-out code family) are future work.

### 11. Reproducibility

Every release is reproducible from a single config file plus the reference data snapshot (built from pinned Payerset data-lake releases). The config records the source claims vintage, NPPES vintage, per-reference vintages, bootstrap seed/draws, `obs_phi`, all tier thresholds and bounds, the Medicare subchannel lists, the channel-prior mode, and the calibration gates.

A given combination of inputs and config produces **byte-identical output**, independent of thread or worker count: the projection stream is totally ordered on the projection grain, per-batch RNG streams derive from `(seed, batch_idx)`, and batches are written through an in-order reorder buffer (V4.2 — earlier versions were value-identical per batch but could write batches in completion order). The test suite asserts byte-identity across runs and worker counts.

### 12. Changelog

#### V4.2 — August 2026

Accuracy release: the priors and pools now come from the data, and the remaining one-sided filters became two-sided. All reference data now comes from versioned releases in the Payerset data lake.

* **Empirical channel priors** (`coverage.channel_priors.mode: empirical`, new default). The V4.1 run projected 75.5% of observed units through hand-set seed priors (0.60/0.75/0.80) while its own measured tiers reported capture of roughly 0.20–0.63 — an internal contradiction that under-projected the prior tier. Priors are now re-derived every run from the measured tiers (Medicaid: pooled T-MSIS capture; Medicare: pooled FFS-measured capture; Commercial: national aggregate FFS capture), with per-channel fallback to the seed CSV below a volume gate and a one-line rollback (`mode: seed`). Provenance is recorded in `COVERAGE_GRAIN` and QA. This is the largest single change to projected totals in V4.2.
* **2024 Physician PUF vintage.** The datalake now carries the data-year-2024 PUF; `references.cms_puf_year` moves 2023 → 2024, matching the 2024 capture reference year and removing the +0.082 vintage-growth residual documented in the V4.1 QA.
* **Medicaid capture year matches the T-MSIS denominator year.** V4.1 divided 2024 observed claims by 2023 T-MSIS lines; cross-year volume growth leaked into the rate. Numerator and denominator are now the same calendar year (min of the reference year and the T-MSIS completeness cap), with a QA warning if the feed lacks that year.
* **Denominator-side direct-tier gates** (`coverage.direct_min_denominator` replaces `direct_min_observed_units`). Gating on observed volume conditioned tier entry on the numerator and selection-biased the measured pools; direct tiers now gate on T-MSIS lines / discharges / PUF services / the scaled MA denominator. Confidence labels still use observed volume; qualification never does.
* **Two-sided code screen on the state benchmark** (`coverage.state_benchmark_code_trim_k`, default 3). The V4.1 trim was one-sided (obs ≫ PUF); codes whose unit definitions mismatch in the other direction survived and dragged pooled capture down (over-projection). Codes are now screened nationally in both directions before state pooling; the screen ships as `medffs_code_screen` and in QA.
* **MA rate clamping.** Counties with published MA participation outside \[0.05, 0.85] are clamped to the bounds instead of dropped (dropping sent precisely the highest-MA counties to the channel prior). Clamp counts in QA.
* **Byte-identical output.** The projection input stream is now totally ordered on the projection grain and parallel batches are written through an in-order reorder buffer, making the §11 reproducibility claim true in the parallel path (V4.1 was value-deterministic per batch but write order followed worker completion). The sequential path now uses the same per-batch RNG derivation, so worker count never changes values.
* **Interval contains the point.** `P95` is additionally capped at no less than the point estimate (§4 Stage 3). The first V4.2 full run surfaced 131 cells (of 380.6M) with fractional observed units below 0.1 where the integer NB draws were almost all zero and the floored P95 collapsed to observed, below the point.
* **Unpartitioned output by default** (`output.partitioning: "none"`). The state/year hive layout added no information (`PROVIDER_STATE` / `RATE_YEAR` are columns) and required a consolidation re-sort before upload; the pipeline now streams one grain-ordered parquet file, and the upload step copies it as-is. `"state_year"` restores the legacy layout.
* **Corrected disclosure:** the claims-vs-lines T-MSIS pairing understates coverage and therefore _overstates_ projection for same-code multi-line claims; earlier docs mislabeled this direction "conservative" (§1.3, §10).
* **Data-lake-sourced references.** Direct PUF and T-MSIS downloads from CMS/HHS are retired; all six references build from pinned Payerset data-lake releases, sharing a single capture path with the Payerset Medicare and Medicaid data pipelines.
* **New QA:** channel-prior provenance, code-screen summary, capture drift by rate year, units-per-claim by tier, MA clamp counts, capture-year disclosure.
* Documentation corrections vs the V4.1 page: §1's per-rate-year coverage keying and `COVERAGE_GRAIN` year token (stale since V4.1's reference-year semantics), §6's dead `medicare_ffs_npi_direct` / `medicare_ffs_county_benchmark` tier entries (replaced by §6.5), the undocumented state-benchmark gates, §6.8's unconditional "applied" phrasing for the shadow-mode Commercial calibration, and §11's reproducibility claim.

#### V4.1 — July 2026

Corrections from the first V4 full-population run, where the (now per-year, FFS-only) QA cross-check plus a direct observed-vs-PUF measurement showed Medicare FFS over-projection concentrated in the Physician-PUF NPI-matched tiers:

* **Reference-year capture semantics.** Capture is a rate: it is measured once at a complete reference claims year (`coverage.capture_reference_year`) and applied to every rate year. Per-year ratios conflated capture with year completeness and silently annualized partial rate years (the 2026 partial year projected at \~5× observed). Measured FFS capture was stable across complete years (NJ: 0.778 in 2024, 0.776 in 2025), validating the rate treatment.
* **`medicare_ffs_state_benchmark` replaces the per-NPI / county Physician-PUF tiers.** Cross-role NPI matching (billing org ↔ rendering individual) is unreliable in both directions; the trust filter catches the high side, but fragmented pairs survive with falsely low coverage (the V4.0 county benchmark applied 5.08× in NJ where measured truth was 1.29×). State capture is the trimmed pooled ratio `Σ observed / Σ PUF` over gated (state × procedure) pairs; pooling makes projected state totals equal PUF totals on validatable codes by construction.
* **Commercial calibration moved to shadow mode** (`commercial_calibration.apply: false`). The per-NPI capture yardstick shares the Physician PUF's attribution-fragmentation problem (capture p50 0.085 vs measured aggregate 0.78), so applied calibration over-projects Commercial. Capture is still computed and reported in QA each release.
* **QA cross-check fixed to the reference year and professional claims.** V4.0 summed projected volume across all rate years and all claim classes against a single-year professional-only PUF, fabricating deltas of roughly (n\_years − 1) × 100% plus the institutional share.
* Stage 2 restructured into sequential per-tier assignments with per-statement timing; QA reads `coverage_rowmap` instead of re-joining the claims grain; `scripts/qa_from_output.py` can rebuild the QA report from a written output dataset.

#### V4 — July 2026

Coverage correctness release plus four new administrative denominators. Three bugs fixed, all of which biased projections **low**:

1. **Per-rate-year coverage.** V3 pooled every rate year of observed volume into one numerator and divided by single-year denominators, inflating coverage by roughly the number of years in the feed and clamping many direct-tier cells to 1.0 (no projection).
2. **Like-for-like ratio metrics.** V3 divided unit counts by T-MSIS claim _lines_ and CMS _discharges_. Units-per-line for drug J-codes run into the hundreds, so raw coverage blew through 1.0 and the clamp silently disabled projection on exactly the high-dollar cells.
3. **Matched-NPI county denominators.** V3 compared a Type-2-only numerator against all-NPI county denominators, biasing coverage low and over-projecting Medicaid.

New coverage tiers: `tmsis_npi_direct`, `medicare_inpatient_npi_direct`, `medicare_outpatient_benchmark`, `ma_scaled_benchmark`, `commercial_calibrated` (+ pooled). Trust filter on direct denominators (`coverage.raw_trust_max`). Observation-noise dispersion scaling (`obs_phi`). `raw_coverage` retained per cell and reported in QA. Synthetic end-to-end test harness with hand-computed expected values for every tier.

#### V3 — April 2026

Architectural simplification. Removed the population-benchmark and peer-group infrastructure (ACS/KFF/MEPS/HCUPnet denominators, Bayesian shrinkage) after the pre-customer review identified a 99213/99214 Commercial double-count bug in that scaffold. Coverage hierarchy reduced to four tiers: `tmsis_direct` → `direct_inpatient_benchmark` → `channel_prior` → `suppress`.

#### V2 — April 2026

Added the two administrative-truth coverage tiers (`tmsis_direct`, `direct_inpatient_benchmark`), the reference-data orchestrator, and the parallelised bootstrap.

#### V1 — March 2026

Initial release. Coverage hierarchy: `direct_pop_benchmark` → `peer_group` → `channel_prior` → `suppress`.
