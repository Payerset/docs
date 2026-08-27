---
description: Data dictionary fields for the Medicare fee schedule datasets in Payerset.
---

# Medicare Data

Payerset publishes five Medicare fee schedule datasets. Each one matches a selection in the **Medicare** dataset picker in the platform, and each has its own dictionary below.

| Dataset                                       | Use it for                                                                                                                                       |
| --------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| IPPS - Inpatient Prospective Payment System   | Hospital inpatient payment data by provider and MS-DRG, including charges, operating and capital components, and reimbursement.                  |
| OPPS - Outpatient Prospective Payment System  | Hospital outpatient payment data by provider and HCPCS, including APC classification, national rates, wage indexes, and adjusted Medicare rates. |
| MPFS - Medicare Physician Fee Schedule        | Physician and practitioner payment rates by HCPCS and locality, including facility and non-facility amounts.                                     |
| CLFS - Clinical Laboratory Fee Schedule       | National payment rates for clinical laboratory services by HCPCS, including modifiers, effective dates, and code descriptions.                   |
| ASC - Ambulatory Surgical Center Fee Schedule | Ambulatory surgical center payment data by provider and HCPCS, including national rates, wage adjustments, and device portions.                  |

For how inpatient rates are calculated, see IPPS Methodology.

{% hint style="info" %}
Geographic adjustment differs by dataset. IPPS uses the wage index tied to CCN, OPPS uses CBSA, ASC uses county FIPS, MPFS uses MAC locality, and CLFS has no geographic adjustment at all.
{% endhint %}

Reach out to support@payerset.com with any questions or clarifications.
