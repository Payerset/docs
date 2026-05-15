---
description: >-
  Below outlines the Payerset methodology for calculating Medicare Inpatient
  rates.
---

# Medicare Inpatient Methodology

## Medicare Inpatient Prospective Payment System (IPPS) Fee Schedule Methodology

#### Document Information

* **Version**: 2.0
* **Fiscal Year**: FY 2026 (Discharges October 1, 2025 – September 30, 2026)
* **Last Updated**: January 2026
* **Regulatory Authority**: 42 CFR Part 412; CMS-1833-F (August 4, 2025)

***

#### Executive Summary

This document describes the methodology used to calculate estimated Medicare IPPS fee schedule amounts for acute care hospital inpatient stays. The calculations are based on the Centers for Medicare & Medicaid Services (CMS) FY 2026 IPPS Final Rule and associated data files. These estimates represent the **expected base Medicare payment** for a given hospital and MS-DRG combination, excluding outlier payments, new technology add-on payments, and certain other case-specific adjustments.

***

#### Regulatory Framework

**Legal Authority**

Medicare payments to acute care hospitals for inpatient stays are governed by Section 1886(d) of the Social Security Act, which establishes the Inpatient Prospective Payment System (IPPS). Under IPPS, hospitals receive a predetermined payment for each discharge based on the patient's diagnosis-related group (MS-DRG), adjusted for geographic and hospital-specific factors.

**Primary Data Sources**

| Source      | Description                          | Publication Date |
| ----------- | ------------------------------------ | ---------------- |
| CMS-1833-F  | FY 2026 IPPS/LTCH PPS Final Rule     | August 4, 2025   |
| Table 1A-1E | National Standardized Amounts        | August 2025      |
| Table 2     | Case-Mix Index and Wage Index by CCN | August 2025      |
| Table 5     | MS-DRG Relative Weights              | August 2025      |
| Impact File | Provider-Specific Payment Factors    | August 2025      |

***

#### Payment Components

IPPS payments consist of two separate components, each calculated independently:

1. **Operating Payment**: Covers the operating costs of providing inpatient care (labor, supplies, overhead)
2. **Capital Payment**: Covers capital-related costs (building depreciation, interest, rent, property taxes)

**Total Estimated Payment**

```
Total IPPS Payment = Operating Payment + Capital Payment
```

***

#### Operating Payment Calculation

**Formula Structure**

The operating payment combines quality-adjusted base payments with DSH, IME, and Uncompensated Care add-ons:

```
Operating Payment = (Base DRG Payment × VBP × HRRP)
                  + (Base DRG Payment × DSH Operating Factor)
                  + Uncompensated Care Per Claim
                  + (Base DRG Payment × IME Factor)
```

**Important**: VBP and HRRP adjustments apply **only** to the base DRG payment. DSH and IME amounts are calculated on the unadjusted base and added separately. Uncompensated Care is a flat per-discharge amount.

**Step 1: Determine Adjusted Base Rate**

The base rate varies depending on whether the hospital's wage index exceeds 1.0:

**If Wage Index > 1.0** (Table 1A rates apply):

```
Adjusted Base Rate = (Labor Share × Wage Index) + (Nonlabor Share × COLA)
                   = ($4,456.72 × Wage Index) + ($2,295.89 × COLA)
```

**If Wage Index ≤ 1.0** (Table 1B rates apply):

```
Adjusted Base Rate = (Labor Share × Wage Index) + (Nonlabor Share × COLA)
                   = ($4,186.62 × Wage Index) + ($2,565.99 × COLA)
```

**FY 2026 Standardized Amounts**

| Component      | Wage Index > 1.0 | Wage Index ≤ 1.0 |
| -------------- | ---------------- | ---------------- |
| Labor Share    | 66% ($4,456.72)  | 62% ($4,186.62)  |
| Nonlabor Share | 34% ($2,295.89)  | 38% ($2,565.99)  |
| Total          | $6,752.61        | $6,752.61        |

This reflects CMS's rebasing of the IPPS market basket to a 2023 base year, as finalized in the FY 2026 IPPS Final Rule.

**Cost of Living Adjustment (COLA)**

* Applies **only** to the nonlabor portion
* Applicable **only** to hospitals in Alaska and Hawaii
* Default value: 1.0 (no adjustment)
* Source: `Cost of Living Adjustment` field in Impact File

**Step 2: Apply MS-DRG Weight**

```
Base DRG Payment = Adjusted Base Rate × MS-DRG Relative Weight
```

MS-DRG weights are published in Table 5 of the IPPS Final Rule and reflect the average resources required to treat patients in each diagnostic category relative to the average Medicare patient.

**Note**: Use the `Weights - 10% Cap Applied` column, which incorporates the regulatory cap on year-over-year weight decreases.

**Step 3: Apply Quality Adjustments to Base**

Quality adjustments apply to the base DRG payment before DSH/IME/UCP add-ons:

```
Quality-Adjusted Base = Base DRG Payment × VBP Factor × HRRP Factor
```

| Adjustment | Description                             | Typical Range | Source Field                                     |
| ---------- | --------------------------------------- | ------------- | ------------------------------------------------ |
| **VBP**    | Hospital Value-Based Purchasing Program | 0.98–1.02     | `Proxy Value Based Purchasing Adjustment Factor` |
| **HRRP**   | Hospital Readmissions Reduction Program | 0.97–1.00     | `Proxy Readmission Adjustment Factor`            |

**Step 4: Calculate DSH Add-On**

```
DSH Amount = Base DRG Payment × DSH Operating Factor
```

**Disproportionate Share Hospital (DSH) Adjustment**

The DSH adjustment compensates hospitals that serve a disproportionate share of low-income patients. Post-ACA (FY 2014+), the DSH payment structure includes:

* **Empirically Justified DSH (25%)**: A case-mix adjusted operating add-on, reflected in the `DSHOPP` field
* **Uncompensated Care (75%)**: Paid as a separate per-discharge amount (see Step 6)

**Source**: `DSHOPP` field from CMS Impact File (this is the operating DSH factor, already computed)

**Step 5: Calculate IME Add-On**

```
IME Amount = Base DRG Payment × IME Factor
```

**Indirect Medical Education (IME) Adjustment**

Teaching hospitals receive additional payments to account for the higher indirect costs associated with residency training programs.

**Source**: `TCHOP` field from CMS Impact File (this is the operating IME factor, already computed using the formula below)

**IME Formula Reference** (for understanding; use pre-computed `TCHOP` value):

```
IME Factor = 1.35 × [(1 + r)^0.405 - 1]
```

Where:

* `r` = Resident-to-Bed Ratio
* `1.35` = Congressional multiplier
* `0.405` = Empirically estimated teaching cost coefficient

**Step 6: Add Uncompensated Care Payment**

```
Uncompensated Care = UCP Per Claim Amount
```

This is a **flat dollar amount per discharge** that varies by hospital, representing that hospital's share of the 75% uncompensated care pool. This amount is added directly to the payment (not multiplied).

**Source**: `UCP Per Claim Amount` field from CMS Impact File

**Step 7: Calculate Total Operating Payment**

```
Operating Payment = (Base DRG Payment × VBP × HRRP)
                  + (Base DRG Payment × DSHOPP)
                  + UCP Per Claim Amount
                  + (Base DRG Payment × TCHOP)
```

***

#### Capital Payment Calculation

**Formula Structure**

```
Capital Payment = Federal Rate × DRG Weight × GAF × COLA × (1 + Capital DSH + Capital IME)
```

**Components**

| Component            | FY 2026 Value      | Source Field                                    |
| -------------------- | ------------------ | ----------------------------------------------- |
| Capital Federal Rate | $524.15            | Table 1D (fixed)                                |
| DRG Weight           | Same as operating  | Table 5 `Weights - 10% Cap Applied`             |
| GAF                  | Hospital-specific  | Impact File `GAF`                               |
| Capital COLA         | Alaska/Hawaii only | Impact File `Capital Cost of Living Adjustment` |
| Capital DSH          | Hospital-specific  | Impact File `DSHCPP`                            |
| Capital IME          | Hospital-specific  | Impact File `TCHCP`                             |

**Geographic Adjustment Factor (GAF)**

The GAF adjusts capital payments for geographic variation in capital costs. Unlike the operating wage index adjustment (which splits labor/nonlabor), the GAF is a single multiplicative factor.

**Capital DSH and IME**

Capital DSH and IME use different formulas than their operating counterparts:

**Capital DSH Formula** (reference; use pre-computed `DSHCPP`):

```
Capital DSH = e^(0.2025 × DSH Patient Percentage) - 1
```

**Capital IME Formula** (reference; use pre-computed `TCHCP`):

```
Capital IME = e^(0.2822 × RADC) - 1
```

Where RADC = Resident-to-Average Daily Census Ratio (capped at 1.5)

**Key Difference from Operating IME**: Capital IME uses Average Daily Census in the denominator, while Operating IME uses Beds.

***

#### Summary: Data Fields Used

| Calculation Component | Impact File Field                                | Notes                           |
| --------------------- | ------------------------------------------------ | ------------------------------- |
| Wage Index            | Table 2: `3,6 FY 2026 Wage Index With Cap`       | Determines labor/nonlabor split |
| DRG Weight            | Table 5: `Weights - 10% Cap Applied`             | Use capped weights              |
| Operating COLA        | `Cost of Living Adjustment`                      | Default 1.0                     |
| Operating DSH         | `DSHOPP`                                         | Pre-computed factor             |
| Operating IME         | `TCHOP`                                          | Pre-computed factor             |
| Uncompensated Care    | `UCP Per Claim Amount`                           | Flat dollar amount              |
| VBP Adjustment        | `Proxy Value Based Purchasing Adjustment Factor` | Default 1.0                     |
| HRRP Adjustment       | `Proxy Readmission Adjustment Factor`            | Default 1.0                     |
| Capital GAF           | `GAF`                                            | Geographic adjustment           |
| Capital COLA          | `Capital Cost of Living Adjustment`              | Default 1.0                     |
| Capital DSH           | `DSHCPP`                                         | Pre-computed factor             |
| Capital IME           | `TCHCP`                                          | Pre-computed factor             |

***

#### Key Assumptions and Limitations

**Assumptions**

1. **Standard Discharge**: Calculations assume a standard discharge without transfer, early discharge, or post-acute care transfer adjustments.
2. **No Outlier Payments**: Extremely high-cost cases may qualify for additional outlier payments not reflected in these estimates.
3. **No New Technology Add-Ons**: Cases involving qualifying new technologies may receive additional payments (NTAP) not included here.
4. **Quality Program Participation**: Calculations assume hospitals participate in quality reporting programs and are meaningful EHR users.
5. **No Special Payment Provisions**: Calculations exclude:
   * Sole Community Hospital (SCH) provisions
   * Medicare-Dependent Hospital (MDH) provisions
   * Low-Volume Hospital adjustments
   * Rural Emergency Hospital provisions

**Limitations**

1. **Estimation vs. Actual Payment**: These are estimated fee schedule amounts. Actual Medicare payments may vary due to:
   * Outlier payments for high-cost cases
   * Transfer payment adjustments
   * New technology add-on payments
   * Cost report settlements
   * Sequestration adjustments (currently 2%)
2. **Provider-Specific Factors**: Some hospital-specific payment adjustments may not be fully captured in public data files.
3. **Timing**: Wage index values and other factors may be updated by CMS during the fiscal year through correction notices.

***

#### Data Quality Notes

**Validation Approach**

Calculated fee schedule amounts have been validated against the CMS IPPS Web Pricer (https://webpricer.cms.gov/) and produce exact matches for operating and capital components when using consistent input parameters.

**Data Refresh Frequency**

This methodology should be updated annually upon publication of the IPPS Final Rule (typically August).

***

#### Glossary

| Term       | Definition                                             |
| ---------- | ------------------------------------------------------ |
| **ADC**    | Average Daily Census                                   |
| **CCN**    | CMS Certification Number (6-digit provider identifier) |
| **COLA**   | Cost of Living Adjustment                              |
| **DRG**    | Diagnosis-Related Group                                |
| **DSH**    | Disproportionate Share Hospital                        |
| **GAF**    | Geographic Adjustment Factor                           |
| **HRRP**   | Hospital Readmissions Reduction Program                |
| **IME**    | Indirect Medical Education                             |
| **IPPS**   | Inpatient Prospective Payment System                   |
| **MS-DRG** | Medicare Severity Diagnosis-Related Group              |
| **NTAP**   | New Technology Add-On Payment                          |
| **UCP**    | Uncompensated Care Payment                             |
| **VBP**    | Value-Based Purchasing                                 |

***

#### Regulatory References

1. **42 CFR Part 412** – Prospective Payment Systems for Inpatient Hospital Services
2. **Social Security Act §1886(d)** – Payments to Hospitals for Inpatient Hospital Services
3. **CMS-1833-F** – FY 2026 IPPS/LTCH PPS Final Rule (90 FR 36536, August 4, 2025)
4. **42 CFR §412.64** – Federal Rates for Inpatient Operating Costs
5. **42 CFR §412.105** – Indirect Medical Education Adjustment
6. **42 CFR §412.106** – Disproportionate Share Adjustment

***

#### Change Log

| Version | Date          | Changes                                                                                                                                                                                                                       |
| ------- | ------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.0     | November 2025 | Initial release                                                                                                                                                                                                               |
| 2.0     | January 2026  | Added Uncompensated Care Payment (UCP Per Claim Amount); Corrected VBP/HRRP application to base only; Clarified DSH field usage (DSHOPP vs DSHPCT); Added Impact File field reference table; Validated against CMS Web Pricer |

***

#### Contact and Support

For questions regarding CMS payment policy:

* CMS IPPS website: https://www.cms.gov/medicare/payment/prospective-payment-systems/acute-inpatient-pps
* CMS Web Pricer: https://webpricer.cms.gov/

***

_This documentation is provided for informational purposes. Users should verify calculations against official CMS sources and consult with qualified healthcare reimbursement professionals for specific payment determinations._
