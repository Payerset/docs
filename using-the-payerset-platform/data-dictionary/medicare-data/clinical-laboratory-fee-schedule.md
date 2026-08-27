---
description: >-
  National payment rates for clinical laboratory services by HCPCS, including
  modifiers, effective dates, and code descriptions.
---

# CLFS - Clinical Laboratory Fee Schedule

Select **CLFS - Clinical Laboratory Fee Schedule** in the Medicare dataset picker to work with this data.

{% hint style="info" %}
Medicare does not apply geographic adjustments to the CLFS. The national **Rate** is the benchmark for every provider, regardless of where the lab is located — there is no wage index or locality step.
{% endhint %}

### Fields

| Field Name            | UI Name              | Description / Sample Value                                                                   |
| --------------------- | -------------------- | -------------------------------------------------------------------------------------------- |
| Year                  | Year                 | Effective fee schedule year. e.g. `2026`                                                     |
| HCPCS\_Code           | HCPCS Code           | Healthcare Common Procedure Coding System laboratory code. e.g. `0011U`                      |
| Modifier              | Modifier             | Optional CPT/HCPCS modifier (or N/A code indicator). e.g. `QW`                               |
| Effective\_Date       | Effective Date       | Effective date for the rate, in YYYYMMDD format. e.g. `20260101`                             |
| Indicator             | Indicator            | National rate indicator code. e.g. `N`                                                       |
| Rate                  | Rate                 | Clinical Laboratory Fee Schedule Medicare national reimbursement amount. e.g. `$114.43`      |
| Short\_Description    | Short Description    | Brief description of the laboratory procedure. e.g. `Rx mntr lc-ms/ms oral fluid`            |
| Long\_Description     | Long Description     | Standard CMS description of the procedure. e.g. `Prescription drug monitoring in oral fluid` |
| Extended\_Description | Extended Description | Full comprehensive descriptor including specific target panels and methods.                  |

### Working with modifiers

Modifiers in the CLFS adjust the base rates. Other modifiers are applied on claims and adjust payment separately.
