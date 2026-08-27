---
description: >-
  Physician and practitioner payment rates by HCPCS and locality, including
  facility and non-facility amounts.
---

# MPFS - Medicare Physician Fee Schedule

Select **MPFS - Medicare Physician Fee Schedule** in the Medicare dataset picker to work with this data. There is one row per HCPCS code, modifier, and MAC locality.

{% hint style="warning" %}
Using **Facility Amount** for an office service overstates percent of Medicare. Place-of-service 11 (office) claims benchmark against **Non-Facility Amount**.
{% endhint %}

### Fields

| Field Name            | UI Name             | Description / Sample Value                                                                                                                                                                  |
| --------------------- | ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| HCPCS\_Code           | HCPCS Code          | Healthcare Common Procedure Coding System code. e.g. `0001F`                                                                                                                                |
| Modifier              | Modifier            | Optional two-character CPT/HCPCS pricing modifier that splits a code into separate rows. e.g. `TC`                                                                                          |
| Status\_Code          | Status Code         | Procedure status indicator determining Medicare coverage and payment status. `A` is active and priced; statuses such as `C`, `B`, `I`, and `N` carry no usable fee-schedule price. e.g. `I` |
| MAC                   | MAC                 | Medicare Administrative Contractor identifier code. There are 12 MACs across the country. e.g. `01182`                                                                                      |
| Locality              | Locality            | Geographic payment locality code under the MAC. There are 48 unique locality codes across 112 localities. e.g. `18`                                                                         |
| Mac\_Locality         | Mac Locality        | Combined MAC and locality identifier code, stored as a 7-digit zero-padded string. This is the most specific MPFS factor identifier used to alter rates. e.g. `0230299`                     |
| Facility\_Amount      | Facility Amount     | Medicare allowed charge when services are performed in a facility setting like a hospital or ASC. e.g. `$4,412.86`                                                                          |
| Non\_Facility\_Amount | Non-Facility Amount | Medicare allowed charge when services are performed in a non-facility setting like a physician's office. e.g. `$4,154.90`                                                                   |
| State                 | State               | State corresponding to the MAC locality. e.g. `California`                                                                                                                                  |
| County                | County              | County or list of counties included in the MAC locality area. e.g. `NAPA`                                                                                                                   |

### Working with modifiers

Modifiers in the MPFS adjust the base rates. Other modifiers are applied on claims and adjust payment separately, so match on code **plus** modifier rather than code alone.

### Working with MAC localities

`Mac_Locality` is the zero-padded concatenation of `MAC` and `Locality`, and the leading zero is significant — left-pad 6-digit external codes before filtering on an exact value. `0000000` is the National row and is not a default to fall back to.
