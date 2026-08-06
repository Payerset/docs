---
description: Data dictionary fields for Medicare outpatient data in Payerset.
---

# Medicare Outpatient Data

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
