---
description: >-
  Below outlines the Payerset methodology for calculating Medicare Inpatient
  rates.
---

# Medicare Inpatient Methodology

## Medicare Inpatient Prospective Payment System (IPPS) Fee Schedule Methodology

### Document Information

* **Version**: 1.0
* **Fiscal Year**: FY 2026 (Discharges October 1, 2025 – September 30, 2026)
* **Last Updated**: November 2025
* **Regulatory Authority**: 42 CFR Part 412; CMS-1833-F (August 4, 2025)

***

### Executive Summary

This document describes the methodology used to calculate estimated Medicare IPPS fee schedule amounts for acute care hospital inpatient stays. The calculations are based on the Centers for Medicare & Medicaid Services (CMS) FY 2026 IPPS Final Rule and associated data files. These estimates represent the **expected base Medicare payment** for a given hospital and MS-DRG combination, excluding outlier payments, new technology add-on payments, and certain other case-specific adjustments.

***

### Regulatory Framework

#### Legal Authority

Medicare payments to acute care hospitals for inpatient stays are governed by Section 1886(d) of the Social Security Act, which establishes the Inpatient Prospective Payment System (IPPS). Under IPPS, hospitals receive a predetermined payment for each discharge based on the patient's diagnosis-related group (MS-DRG), adjusted for geographic and hospital-specific factors.

#### Primary Data Sources

| Source      | Description                          | Publication Date |
| ----------- | ------------------------------------ | ---------------- |
| CMS-1833-F  | FY 2026 IPPS/LTCH PPS Final Rule     | August 4, 2025   |
| Table 1A-1E | National Standardized Amounts        | August 2025      |
| Table 2     | Case-Mix Index and Wage Index by CCN | August 2025      |
| Table 5     | MS-DRG Relative Weights              | August 2025      |
| Impact File | Provider-Specific Payment Factors    | August 2025      |

***

### Payment Components

IPPS payments consist of two separate components, each calculated independently:

1. **Operating Payment**: Covers the operating costs of providing inpatient care (labor, supplies, overhead)
2. **Capital Payment**: Covers capital-related costs (building depreciation, interest, rent, property taxes)

#### Total Estimated Payment

```
Total IPPS Payment = Operating Payment + Capital Payment
```

***

### Operating Payment Calculation

#### Formula Structure

```
Operating Payment = Adjusted Base Rate × DRG Weight × Quality Adjustments × (1 + DSH + IME)
```

#### Step 1: Determine Adjusted Base Rate

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

**Labor-Related Share**

* **66%** for hospitals with wage index > 1.0 (Table 1A)
* **62%** for hospitals with wage index ≤ 1.0 (Table 1B)

This reflects CMS's rebasing of the IPPS market basket to a 2023 base year, as finalized in the FY 2026 IPPS Final Rule.

**Cost of Living Adjustment (COLA)**

* Applies **only** to the nonlabor portion
* Applicable **only** to hospitals in Alaska and Hawaii
* Default value: 1.0 (no adjustment)

#### Step 2: Apply MS-DRG Weight

```
DRG-Adjusted Payment = Adjusted Base Rate × MS-DRG Relative Weight
```

MS-DRG weights are published in Table 5 of the IPPS Final Rule and reflect the average resources required to treat patients in each diagnostic category relative to the average Medicare patient.

**Note**: Weights incorporate a 10% cap on year-over-year decreases, as mandated by regulation.

#### Step 3: Apply Quality Adjustments

```
Quality-Adjusted Payment = DRG-Adjusted Payment × VBP Factor × HRRP Factor
```

| Adjustment | Description                             | Application                |
| ---------- | --------------------------------------- | -------------------------- |
| **VBP**    | Hospital Value-Based Purchasing Program | Ranges typically 0.98–1.02 |
| **HRRP**   | Hospital Readmissions Reduction Program | Maximum reduction of 3%    |

These adjustments apply to the base operating DRG payment before DSH and IME add-ons.

#### Step 4: Apply DSH and IME Add-Ons

```
Final Operating Payment = Quality-Adjusted Payment × (1 + DSH% + IME%)
```

**Disproportionate Share Hospital (DSH) Adjustment**

The DSH adjustment compensates hospitals that serve a disproportionate share of low-income patients. Post-ACA (FY 2014+), hospitals receive:

* **25%** of their pre-ACA DSH payment as an empirically justified operating add-on (case-mix adjusted)
* **75%** as uncompensated care payments (not included in this per-discharge calculation)

**Source**: `DSHPCT` field from CMS Impact File

**Indirect Medical Education (IME) Adjustment**

Teaching hospitals receive additional payments to account for the higher indirect costs associated with residency training programs.

**Formula**:

```
IME Adjustment = 1.35 × [(1 + r)^0.405 - 1]
```

Where:

* `r` = Resident-to-Bed Ratio (from Impact File)
* `1.35` = Congressional multiplier (represents 5.5% increase per 10% increase in teaching intensity)
* `0.405` = Empirically estimated teaching cost coefficient

***

### Capital Payment Calculation

#### Formula Structure

```
Capital Payment = Federal Rate × DRG Weight × GAF × COLA × (1 + DSH + Capital IME)
```

#### Components

| Component            | FY 2026 Value      | Source      |
| -------------------- | ------------------ | ----------- |
| Capital Federal Rate | $524.15            | Table 1D    |
| DRG Weight           | Same as operating  | Table 5     |
| GAF                  | Hospital-specific  | Impact File |
| Capital COLA         | Alaska/Hawaii only | Impact File |

#### Geographic Adjustment Factor (GAF)

The GAF adjusts capital payments for geographic variation in capital costs. Unlike the operating wage index adjustment, the GAF is a single multiplicative factor published in the Impact File.

#### Capital IME Adjustment

The capital IME formula differs from the operating IME formula:

```
Capital IME = e^(0.2822 × RADC) - 1
```

Where:

* `e` = Euler's number (≈2.71828)
* `0.2822` = Empirically estimated capital teaching coefficient
* `RADC` = Resident-to-Average Daily Census Ratio

**RADC Calculation**:

```
RADC = (Resident-to-Bed Ratio × Beds) ÷ Average Daily Census
```

***

### Key Assumptions and Limitations

#### Assumptions

1. **Standard Discharge**: Calculations assume a standard discharge without transfer, early discharge, or post-acute care transfer adjustments.
2. **No Outlier Payments**: Extremely high-cost cases may qualify for additional outlier payments not reflected in these estimates.
3. **No New Technology Add-Ons**: Cases involving qualifying new technologies may receive additional payments (NTAP) not included here.
4. **Quality Program Participation**: Calculations assume hospitals participate in quality reporting programs and are meaningful EHR users (receiving the full 2.6% update).
5. **No Special Payment Provisions**: Calculations exclude:
   * Sole Community Hospital (SCH) provisions
   * Medicare-Dependent Hospital (MDH) provisions (expired September 30, 2025)
   * Low-Volume Hospital adjustments
   * Rural Emergency Hospital provisions
6. **DSH Methodology**: The DSH percentage used (`DSHPCT`) represents the empirically justified DSH operating adjustment. Uncompensated care payments (75% of DSH) are paid as a separate flat amount per discharge and are **not** included in these per-DRG estimates.

#### Limitations

1. **Estimation vs. Actual Payment**: These are estimated fee schedule amounts. Actual Medicare payments may vary due to:
   * Outlier payments for high-cost cases
   * Transfer payment adjustments
   * New technology add-on payments
   * Cost report settlements
   * Sequestration adjustments (currently 2%)
2. **Provider-Specific Factors**: Some hospital-specific payment adjustments may not be fully captured in public data files.
3. **Timing**: Wage index values and other factors may be updated by CMS during the fiscal year through correction notices.

***

### Data Quality Notes

#### Validation Approach

Calculated fee schedule amounts should be validated against:

1. CMS IPPS Web Pricer (https://webpricer.cms.gov/)
2. Actual Medicare payment data from claims
3. Hospital cost reports

#### Expected Variance

When comparing estimated fee schedule amounts to actual Medicare payments:

* **±5%** variance is typical and expected
* **±10%** variance may indicate outlier cases, transfers, or special circumstances
* **>10%** variance warrants investigation

#### Data Refresh Frequency

This methodology should be updated annually upon publication of the IPPS Final Rule (typically August).

***

### Glossary

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
| **VBP**    | Value-Based Purchasing                                 |

***

### Regulatory References

1. **42 CFR Part 412** – Prospective Payment Systems for Inpatient Hospital Services
2. **Social Security Act §1886(d)** – Payments to Hospitals for Inpatient Hospital Services
3. **CMS-1833-F** – FY 2026 IPPS/LTCH PPS Final Rule (89 FR 36536, August 4, 2025)
4. **42 CFR §412.64** – Federal Rates for Inpatient Operating Costs
5. **42 CFR §412.105** – Indirect Medical Education Adjustment
6. **42 CFR §412.106** – Disproportionate Share Adjustment

***

### Contact and Support

For questions regarding CMS payment policy:

* CMS IPPS website: https://www.cms.gov/medicare/payment/prospective-payment-systems/acute-inpatient-pps
* CMS Web Pricer: https://webpricer.cms.gov/

***

_This documentation is provided for informational purposes. Users should verify calculations against official CMS sources and consult with qualified healthcare reimbursement professionals for specific payment determinations._
