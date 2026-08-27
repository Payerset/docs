---
description: >-
  Hospital inpatient payment data by provider and MS-DRG, including charges,
  operating and capital components, and reimbursement.
---

# IPPS - Inpatient Prospective Payment System

Select **IPPS - Inpatient Prospective Payment System** in the Medicare dataset picker to work with this data.

For how these rates are calculated, see IPPS Methodology.

### Fields

| Field Name                       | UI Name                         | Description / Sample Value                                                            |
| -------------------------------- | ------------------------------- | ------------------------------------------------------------------------------------- |
| NPI                              | NPI                             | Unique 10-digit National Provider Identifier. e.g. `1164403861`                       |
| carrier\_number                  | Carrier Number                  | CMS Certification Number (6-digit provider identifier). e.g. `010001`                 |
| Rndrng\_Prvdr\_Org\_Name         | Rendering Provider Organization | Name of the rendering provider's organization. e.g. `Southeast Health Medical Center` |
| Rndrng\_Prvdr\_City              | Rendering Provider City         | City where the provider is located. e.g. `Dothan`                                     |
| Rndrng\_Prvdr\_St                | Rendering Provider Address      | Street address of the provider. e.g. `1108 Ross Clark Circle`                         |
| Rndrng\_Prvdr\_State\_FIPS       | Provider State FIPS             | U.S. Census state FIPS code. e.g. `01`                                                |
| Rndrng\_Prvdr\_Zip5              | Provider ZIP Code               | 5-digit postal ZIP code. e.g. `36301`                                                 |
| Rndrng\_Prvdr\_State\_Abrvtn     | Provider State Abbreviation     | USPS two-letter state abbreviation. e.g. `AL`                                         |
| cbsa                             | CBSA Code                       | Core-Based Statistical Area code for geographic wage adjustment. e.g. `20020`         |
| drg\_code                        | DRG Code                        | Medicare Severity Diagnosis-Related Group code. e.g. `023`                            |
| operating\_base\_drg\_payment    | Operating Base DRG Payment      | Wage-adjusted base payment × DRG weight, before adjustments. e.g. `15234.56`          |
| operating\_dsh\_factor           | Operating DSH Factor            | Disproportionate Share Hospital adjustment factor. e.g. `0.0298`                      |
| operating\_dsh\_amount           | Operating DSH Amount            | Dollar amount of the DSH add-on. e.g. `453.99`                                        |
| operating\_ime\_factor           | Operating IME Factor            | Indirect Medical Education adjustment factor. e.g. `0.0655`                           |
| operating\_ime\_amount           | Operating IME Amount            | Dollar amount of the IME add-on. e.g. `997.86`                                        |
| vbp\_factor                      | VBP Factor                      | Hospital Value-Based Purchasing adjustment (typically 0.98–1.02). e.g. `0.99879`      |
| hrrp\_factor                     | HRRP Factor                     | Hospital Readmissions Reduction Program adjustment (minimum 0.97). e.g. `0.9975`      |
| fee\_schedule\_dollar\_amount    | Fee Schedule Amount             | Total operating IPPS payment amount. e.g. `$41,148.42`                                |
| capital\_base\_payment           | Capital Base Payment            | Federal capital rate × DRG weight × GAF × COLA. e.g. `1245.67`                        |
| capital\_dsh\_factor             | Capital DSH Factor              | Capital Disproportionate Share adjustment factor. e.g. `0.05731`                      |
| capital\_dsh\_amount             | Capital DSH Amount              | Dollar amount of capital DSH. e.g. `71.39`                                            |
| capital\_ime\_factor             | Capital IME Factor              | Capital Indirect Medical Education factor. e.g. `0.05496`                             |
| capital\_ime\_amount             | Capital IME Amount              | Dollar amount of capital IME. e.g. `68.46`                                            |
| capital\_fee\_schedule\_amount   | Capital Fee Schedule Amount     | Total capital IPPS payment. e.g. `1385.52`                                            |
| total\_reimbursement\_amount     | Total Reimbursement Amount      | Combined operating + capital payment. e.g. `19636.27`                                 |
| Total\_Discharges                | Total Discharges                | Total number of discharges reported. e.g. `23`                                        |
| Avg\_Submitted\_Covered\_Charges | Avg. Covered Charges            | Average submitted covered charges per discharge. e.g. `$173,562.09`                   |
| Avg\_Total\_Payment\_Amount      | Avg. Total Payment              | Average total payment amount per discharge. e.g. `$40,220.22`                         |
| Avg\_Medicare\_Payment\_Amount   | Avg. Medicare Payment           | Average amount paid by Medicare per discharge. e.g. `$37,634.57`                      |
| Avg\_Medicare\_Payment\_Percent  | Avg. Medicare Payment %         | Ratio of average Medicare payment relative to average total payment. e.g. `0.91`      |
| latitude                         | Latitude                        | Geographic latitude of the provider location. e.g. `31.148100`                        |
| longitude                        | Longitude                       | Geographic longitude of the provider location. e.g. `-85.371800`                      |
