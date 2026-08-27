---
description: >-
  Hospital outpatient payment data by provider and HCPCS, including APC
  classification, national rates, wage indexes, and adjusted Medicare rates.
---

# OPPS - Outpatient Prospective Payment System

Select **OPPS - Outpatient Prospective Payment System** in the Medicare dataset picker to work with this data. There is one row per hospital outpatient provider and HCPCS code.

{% hint style="warning" %}
Packaged status indicators carry no separate payment. Exclude them when computing percent of Medicare, or the denominator will be understated.
{% endhint %}

### Fields

| Field Name          | UI Name           | Description / Sample Value                                                                                                                                                                         |
| ------------------- | ----------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| NPI                 | NPI               | Unique 10-digit National Provider Identifier of the hospital outpatient provider. e.g. `1003185455`                                                                                                |
| CCN                 | CCN               | CMS Certification Number for the hospital or facility, also known as a PTAN. e.g. `400104`                                                                                                         |
| HCPCS\_Code         | HCPCS Code        | Healthcare Common Procedure Coding System code. e.g. `22600`                                                                                                                                       |
| Status\_Indicator   | Status Indicator  | OPPS status indicator determining payment status and bundling logic. e.g. `J1` (comprehensive APC), `T` (separately payable), `N` (packaged)                                                       |
| APC                 | APC               | Ambulatory Payment Classification group the HCPCS maps to. Used to group HCPCS into weight levels. e.g. `5116`                                                                                     |
| Relative\_Weight    | Relative Weight   | APC relative payment weight — the resource intensity of the APC relative to the average OPPS service. Used to determine the payment rate when combined with the conversion factor. e.g. `195.9590` |
| National\_Rate      | National Rate     | Unadjusted national base payment rate. Calculated by multiplying the OPPS conversion factor by the APC weight. e.g. `$17,913.59`                                                                   |
| Wage\_Index         | Wage Index        | Geographic wage index multiplier applied to the labor portion, based on CBSA or CCN. e.g. `0.3382`                                                                                                 |
| Wage\_Index\_Source | Wage Index Source | Provenance of the wage index used for this provider — CBSA assignment versus rural floor or another CMS special rule. e.g. `CCN`                                                                   |
| Medicare\_Rate      | Medicare Rate     | Final wage-adjusted Medicare reimbursement rate. This is the amount paid for a given HCPCS code for a specific provider in isolation. e.g. `$10,800.46`                                            |

### How the rate is calculated

The national rate comes from the APC weight, and the Medicare rate wage-adjusts the labor portion of it:

```
National Rate = OPPS Conversion Factor × APC Relative Weight
Medicare Rate = National Rate × ((0.6 × Wage Index) + 0.4)
```

Only 60% of the national rate is treated as labor and exposed to the wage index; the remaining 40% is not adjusted.
