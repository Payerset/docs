---
description: Data dictionary fields for Medicare inpatient data in Payerset.
---

# Medicare Inpatient Data

For Medicare inpatient rate calculations, see [Medicare Inpatient Methodology](medicare-inpatient-methodology.md).

### Inpatient Data

| Field Name                       | Friendly Name                 | Description / Sample Value                                                        |
| -------------------------------- | ----------------------------- | --------------------------------------------------------------------------------- |
| NPI                              | Provider NPI                  | Unique 10-digit National Provider Identifier. e.g. `1043270564` or `No NPI Found` |
| carrier\_number                  | CCN                           | CMS Certification Number (6-digit provider identifier). e.g. `110107`             |
| Rndrng\_Prvdr\_Org\_Name         | Provider Organization Name    | Name of the rendering provider's organization. e.g. `Atrium Health Navicent`      |
| Rndrng\_Prvdr\_City              | Provider City                 | City where the provider is located. e.g. `Macon`                                  |
| Rndrng\_Prvdr\_St                | Provider Street Address       | Street address of the provider. e.g. `777 Hemlock Street`                         |
| Rndrng\_Prvdr\_State\_FIPS       | State FIPS Code               | U.S. Census state FIPS code. e.g. `13`                                            |
| Rndrng\_Prvdr\_Zip5              | Provider ZIP Code             | 5-digit postal ZIP code. e.g. `31201`                                             |
| Rndrng\_Prvdr\_State\_Abrvtn     | State Abbreviation            | USPS two-letter state abbreviation. e.g. `GA`                                     |
| CBSA                             | CBSA Code                     | Core-Based Statistical Area code for geographic wage adjustment. e.g. `12060`     |
| drg\_code                        | DRG Code                      | Medicare Severity Diagnosis-Related Group code. e.g. `470`                        |
| operating\_base\_drg\_payment    | Operating Base DRG Payment    | Wage-adjusted base payment × DRG weight, before adjustments. e.g. `15234.56`      |
| operating\_dsh\_factor           | Operating DSH Factor          | Disproportionate Share Hospital adjustment factor. e.g. `0.0298`                  |
| operating\_dsh\_amount           | Operating DSH Amount          | Dollar amount of DSH add-on. e.g. `453.99`                                        |
| operating\_ime\_factor           | Operating IME Factor          | Indirect Medical Education adjustment factor. e.g. `0.0655`                       |
| operating\_ime\_amount           | Operating IME Amount          | Dollar amount of IME add-on. e.g. `997.86`                                        |
| vbp\_factor                      | VBP Adjustment Factor         | Hospital Value-Based Purchasing adjustment (typically 0.98–1.02). e.g. `0.99879`  |
| hrrp\_factor                     | HRRP Adjustment Factor        | Hospital Readmissions Reduction Program adjustment (min 0.97). e.g. `0.9975`      |
| fee\_schedule\_dollar\_amount    | Operating Fee Schedule Amount | Total operating IPPS payment. e.g. `18250.75`                                     |
| capital\_base\_payment           | Capital Base Payment          | Federal capital rate × DRG weight × GAF × COLA. e.g. `1245.67`                    |
| capital\_dsh\_factor             | Capital DSH Factor            | Capital Disproportionate Share adjustment factor. e.g. `0.05731`                  |
| capital\_dsh\_amount             | Capital DSH Amount            | Dollar amount of capital DSH. e.g. `71.39`                                        |
| capital\_ime\_factor             | Capital IME Factor            | Capital Indirect Medical Education factor. e.g. `0.05496`                         |
| capital\_ime\_amount             | Capital IME Amount            | Dollar amount of capital IME. e.g. `68.46`                                        |
| capital\_fee\_schedule\_amount   | Capital Fee Schedule Amount   | Total capital IPPS payment. e.g. `1385.52`                                        |
| total\_reimbursement\_amount     | Total Reimbursement Amount    | Combined operating + capital payment. e.g. `19636.27`                             |
| Total\_Discharges                | Total Discharges              | Total number of discharges reported. e.g. `250` (may be null)                     |
| Avg\_Submitted\_Covered\_Charges | Avg Covered Charges           | Average submitted covered charges per discharge. e.g. `32500.50`                  |
| Avg\_Total\_Payment\_Amount      | Avg Total Payment             | Average total payment amount per discharge. e.g. `19500.75`                       |
| Avg\_Medicare\_Payment\_Amount   | Avg Medicare Payment          | Average amount paid by Medicare per discharge. e.g. `18000.25`                    |
| Avg\_Medicare\_Payment\_Percent  | Medicare Payment %            | Ratio of avg Medicare payment to operating fee schedule. e.g. `0.99`              |
| latitude                         | Latitude                      | Geographic latitude of the provider location. e.g. `32.8095`                      |
| longitude                        | Geographic longitude          | Geographic longitude of the provider location. e.g. `-83.6168`                    |
