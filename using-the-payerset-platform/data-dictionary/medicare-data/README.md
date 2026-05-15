---
description: Data dictionary fields for Medicare data in Payerset.
---

# Medicare Data

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

### Outpatient Data

| Field Name                               | Friendly Name                                   | Description / Sample Value                                                 |
| ---------------------------------------- | ----------------------------------------------- | -------------------------------------------------------------------------- |
| **HCPCS Code**                           | HCPCS Procedure Code                            | Healthcare Common Procedure Coding System code. e.g. `0275T`               |
| **Modifier**                             | HCPCS Modifier                                  | Optional two-character modifier. e.g. `""` (empty if none)                 |
| **Short Description**                    | Service Short Description                       | Brief description of the procedure. e.g. `Perq lamot/lam lumbar`           |
| **Mac Locality**                         | MAC Locality Code                               | Medicare Administrative Contractor locality code. e.g. `111205`            |
| **Locality County**                      | County                                          | County that corresponds with Mac Locality                                  |
| **Locality State**                       | State                                           | State that corresponds with Mac Locality                                   |
| **Non-Facility Price**                   | Non-Facility Price                              | Allowed charge in a non-facility setting. e.g. `"$0.00"`                   |
| **Facility Price**                       | Facility Price                                  | Allowed charge in a facility setting. e.g. `"$0.00"`                       |
| **Non-Facility Limiting Charge**         | Non-Facility Limiting Charge                    | Payment limit for non-facility. e.g. `"$0.00"`                             |
| **Facility Limiting Charge**             | Facility Limiting Charge                        | Payment limit for facility. e.g. `"$0.00"`                                 |
| **GPCI Work**                            | Work GPCI Factor                                | Geographic practice cost index for work. e.g. `1.088`                      |
| **GPCI PE**                              | Practice Exp. GPCI Factor                       | Geographic practice cost index for practice expense. e.g. `1.419`          |
| **GPCI MP**                              | Malpractice GPCI Factor                         | Geographic malpractice cost index. e.g. `0.445`                            |
| **Proc Stat**                            | Procedure Status                                | Status indicator (e.g. R=revised). e.g. `"R"`                              |
| **Work RVU**                             | Work RVU                                        | Relative value unit for physician work. e.g. `0.00`                        |
| **NA Flag for Trans Non-FAC PE RVU**     | Flag: Transitional Non-Facility PE RVU Missing  | `"NA"` if no transitional practice-expense RVU available                   |
| **Transitioned Non-FAC PE RVU**          | Transitional Non-Facility PE RVU                | Transitional practice-expense RVU, non-facility. e.g. `0.00`               |
| **NA Flag for Fully IMP Non-FAC PE RVU** | Flag: Fully Implemented Non-FAC PE RVU Missing  | `"NA"` if no fully implemented practice-expense RVU available              |
| **Fully Implemented Non-FAC PE RVU**     | Fully Impl. Non-Facility PE RVU                 | Final practice-expense RVU, non-facility. e.g. `0.00`                      |
| **NA Flag for Trans Facility PE RVU**    | Flag: Transitional Facility PE RVU Missing      | `"NA"` if no transitional practice-expense RVU for facility available      |
| **Transitioned Facility PE RVU**         | Transitional Facility PE RVU                    | Transitional practice-expense RVU, facility. e.g. `0.00`                   |
| **NA Flag for Fully IMP FAC PE RVU**     | Flag: Fully Implemented Facility PE RVU Missing | `"NA"` if no fully implemented practice-expense RVU for facility available |
| **Fully Implemented Facility PE RVU**    | Fully Impl. Facility PE RVU                     | Final practice-expense RVU, facility. e.g. `0.00`                          |
| **MP RVU**                               | Malpractice RVU                                 | Relative value unit for malpractice. e.g. `0.00`                           |
| **Transitioned Non-FAC Total**           | Transitional Non-Facility Total RVU             | Sum of work + PE + MP RVUs (transitional)(non-facility). e.g. `0.00`       |
| **Transitioned Facility Total**          | Transitional Facility Total RVU                 | Sum of work + PE + MP RVUs (transitional)(facility). e.g. `0.00`           |
| **Fully Implemented Non-Fac Total**      | Fully Impl. Non-Facility Total RVU              | Sum of RVUs (work+PE+MP) final, non-facility. e.g. `0.00`                  |
| **Fully Implemented Facility Total**     | Fully Impl. Facility Total RVU                  | Sum of RVUs (work+PE+MP) final, facility. e.g. `0.00`                      |
| **PCTC**                                 | Multiple-Procedure Indicator                    | Indicator for multiple-procedure payment reduction. e.g. `"YYY"`           |
| **Global**                               | Global Surgical Indicator                       | Global surgery period indicator (0=no global). e.g. `0`                    |
| **Pre Op**                               | Pre-Operative RVU                               | RVU for pre-operative period. e.g. `0.00`                                  |
| **Intra Op**                             | Intra-Operative RVU                             | RVU for intra-operative period. e.g. `0.00`                                |
| **Post Op**                              | Post-Operative RVU                              | RVU for post-operative period. e.g. `0.00`                                 |
| **Mult Surg**                            | Multiple Surgery RVU                            | RVU adjustment for multiple surgeries. e.g. `0.00`                         |
| **Bilt Surg**                            | Bilateral Surgery RVU                           | RVU adjustment for bilateral procedures. e.g. `0.00`                       |
| **Asst Surg**                            | Assistant Surgeon RVU                           | RVU for assistant surgeon. e.g. `0.00`                                     |
| **Co Surg**                              | Co-Surgeon RVU                                  | RVU for co-surgeon. e.g. `0.00`                                            |
| **Team Surg**                            | Team Surgery RVU                                | RVU for team surgery. e.g. `0.00`                                          |
| **Phys Supv**                            | Physician Supervision RVU                       | RVU for physician supervision. e.g. `0.00`                                 |
| **Endobase**                             | Endoscopic Base RVU Indicator                   | Indicator if code is endoscopic base. e.g. `""`                            |
| **Conv Fact**                            | Conversion Factor                               | Dollar-to-RVU conversion factor. e.g. `32.3465`                            |
| **Not Used for Medicare**                | Excluded from Medicare                          | Flag if code is not payable by Medicare. e.g. `""`                         |
| **Diag Imaging Family Ind**              | Diagnostic Imaging Family Indicator             | Family group code. e.g. `99`                                               |
| **Opps Non-Facility Payment Amount**     | OPPS Non-Facility Payment                       | Payment amount under OPPS non-facility. e.g. `"NA"`                        |
| **Opps Facility Payment Amount**         | OPPS Facility Payment                           | Payment amount under OPPS facility. e.g. `"NA"`                            |
| **Non-Fac PE Used For Opps PMT AMT**     | Non-Facility PE Weight for OPPS Payment         | Practice-expense index used in OPPS non-facility calculation. e.g. `0.0`   |
| **Facility PE Used For Opps PMT AMT**    | Facility PE Weight for OPPS Payment             | Practice-expense index used in OPPS facility calculation. e.g. `0.0`       |
| **Malpractice Used For Opps PMT AMT**    | Malpractice PE Weight for OPPS Payment          | Malpractice index used in OPPS payment calculation. e.g. `0.0`             |
