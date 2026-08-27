---
description: >-
  Ambulatory surgical center payment data by provider and HCPCS, including
  national rates, wage adjustments, and device portions.
---

# ASC - Ambulatory Surgical Center Fee Schedule

Select **ASC - Ambulatory Surgical Center Fee Schedule** in the Medicare dataset picker to work with this data. There is one row per ASC and HCPCS code.

{% hint style="warning" %}
Packaged payment indicators carry no separate payment. Exclude them when computing percent of Medicare.
{% endhint %}

### Fields

| Field Name         | UI Name           | Description / Sample Value                                                                                                                                                         |
| ------------------ | ----------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| NPI                | NPI               | Unique 10-digit National Provider Identifier of the ambulatory surgical center. e.g. `1013463157`                                                                                  |
| FIPS\_Code         | FIPS Code         | U.S. Census Federal Information Processing Standards geographic code — a 5-digit value combining a 2-digit state code and a 3-digit county code. e.g. `37021`                      |
| HCPCS\_Code        | HCPCS Code        | Healthcare Common Procedure Coding System procedure code. e.g. `65600`                                                                                                             |
| Payment\_Indicator | Payment Indicator | ASC payment status indicator determining payment status and bundling logic. e.g. `P3` (office-based), `A2` (surgical procedure priced under OPPS relative weight), `N1` (packaged) |
| Addendum           | Addendum          | CMS ASC payment addendum the row comes from. e.g. `AA`                                                                                                                             |
| National\_Rate     | National Rate     | Unadjusted national payment rate for the procedure, taken from Addendum AA or BB. e.g. `$296.07`                                                                                   |
| Wage\_Index        | Wage Index        | Geographic wage index adjustment factor for the ASC's county. e.g. `0.8847`                                                                                                        |
| Device\_Portion    | Device Portion    | Device-intensive payment portion amount. This portion is not wage-adjusted. e.g. `$70.13`                                                                                          |
| Medicare\_Rate     | Medicare Rate     | Final calculated Medicare reimbursement amount. e.g. `$296.07`                                                                                                                     |
| Adjustment         | Adjustment        | How the rate was adjusted for this ASC — wage-adjusted versus unadjusted for device-intensive or flat-rated codes. e.g. `national`                                                 |

{% hint style="info" %}
ASC wage indexes are tied to **FIPS**, not to CBSA (as in OPPS) or CCN (as in IPPS). The county of the ASC's practice location drives the adjustment.
{% endhint %}

### How the rate is calculated

```
Medicare Rate = National Rate × ((0.5 × Wage Index) + 0.5)
```

Half of the national rate is treated as labor and exposed to the wage index. Where a procedure has a device component, the **unadjusted** device portion is added on top of that result to get the true Medicare rate.

For payment indicator `J8`, the national service portion rate is calculated by subtracting the device offset amount (Addendum FF) from the payment rate (Addendum AA) for the HCPCS.

### CMS ASC addenda

Rates in this dataset are drawn from Addendum AA and Addendum BB. The full set of CMS ASC addenda is:

| Addendum | Contents                                                 |
| -------- | -------------------------------------------------------- |
| AA       | Surgical procedures payable under the ASC payment system |
| BB       | Covered ancillary services                               |
| DD1      | Payment indicator definitions                            |
| DD2      | Comment indicators                                       |
| EE       | Surgical procedures excluded from payment                |
| FF       | Device offset percentages                                |
