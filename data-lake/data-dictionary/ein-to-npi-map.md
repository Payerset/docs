---
description: >-
  These files contain mappings of EIN values to NPI values based on the
  Transparency in Coverage data across our available payers.
---

# EIN to NPI Map

## TIN Rollup File

| Field Name    | Field Type | Notes                                                                                                            |
| ------------- | ---------- | ---------------------------------------------------------------------------------------------------------------- |
| tin\_value    | varchar    | The EIN Value present in the tin\_value field from the TiC Data                                                  |
| npi           | varchar    | The NPI associated with the given EIN in the tin\_value field                                                    |
| payer\_list   | array\[]   | The payers that include the given EIN-NPI relationship                                                           |
| payer\_count  | int        | The number of payers that contain the given EIN-NPI relationship                                                 |
| record\_count | double     | The count of provider records in total associated with the given EIN-NPI relationship based on our file sampling |

Please see below for a screenshot example of this dataset.

<figure><img src="../../.gitbook/assets/CleanShot 2025-01-24 at 17.48.12.png" alt=""><figcaption></figcaption></figure>
