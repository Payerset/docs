---
description: >-
  This data contains the price transparency machine-readable files provided by
  Aetna that have been normalized into easy-to-use tables.
---

# 🟠 Aetna

### Payerset Notes

**Table of Contents**

Aetna has multiple tables of contents:

[https://mrf.payerset.com/aetna](https://mrf.payerset.com/aetna)

* [Fully insured](https://health1.aetna.com/app/public/#/one/insurerCode=AETNACVS_I\&brandCode=ALICFI/machine-readable-transparency-in-coverage)
* [Self - insured - ALICSI](https://health1.aetna.com/app/public/#/one/insurerCode=AETNACVS_I\&brandCode=ALICSI/machine-readable-transparency-in-coverage) / [ALICUNDER100](https://health1.aetna.com/app/public/#/one/insurerCode=AETNACVS_I\&brandCode=ALICUNDER100/machine-readable-transparency-in-coverage)
* [AetnaCVS Individual Exchange](https://health1.aetna.com/app/public/#/one/insurerCode=AETNACVS_I\&brandCode=AETNACVS/machine-readable-transparency-in-coverage)
* [Texas Health Plans](https://health1.aetna.com/app/public/#/one/insurerCode=AETNACVS_I\&brandCode=TEXASFI/machine-readable-transparency-in-coverage)
* [Aetna Signature Administrators](https://health1.aetna.com/app/public/#/one/insurerCode=AETNACVS_I\&brandCode=ASA/machine-readable-transparency-in-coverage?searchTerm=ASA_01\&lock=true)

### Compliance Scorecard

Overall Rating: <mark style="color:orange;">**3/5**</mark>**&#x20;- Needs Improvement**

<table data-view="cards"><thead><tr><th></th><th></th><th></th><th></th><th data-hidden data-card-cover data-type="files"></th></tr></thead><tbody><tr><td><strong>Table of Contents</strong></td><td><strong>★★★☆☆</strong></td><td><mark style="color:orange;"><strong>3/5</strong></mark></td><td><ul><li>Are the MRFs kept up to date each month? </li><li><mark style="color:red;">Is the Table of Contents link easily accessible?</mark></li><li>Is the Table of Contents properly formatted?</li></ul></td><td></td></tr><tr><td><strong>File Accessibility</strong></td><td><strong>★★★★☆</strong></td><td><mark style="color:yellow;"><strong>4/5</strong></mark></td><td><ul><li>Are there any barriers to downloading the files?</li><li>Do the Table of Contents links expire before publishing new links?</li></ul></td><td></td></tr><tr><td><strong>Data Quality</strong></td><td><strong>★★★☆☆</strong></td><td><mark style="color:orange;"><strong>3/5</strong></mark></td><td><ul><li>Is the data present in the files complete</li><li>Are there significant gaps in any type of data</li><li>Is the payer compliant with the regulations</li></ul></td><td></td></tr></tbody></table>

### Schema: AETNA

**Rates Records**: 2,494,235,787,461

**Provider Records**: 18,288,687,914

### Additional Observations

**Machine-Readable Price Transparency Files Review**

* **MRFs Up-to-Date:** ✔️ Yes, the MRFs are kept up to date each month.
* **Table of Contents Accessibility:** ❌ The Table of Contents link is not easily accessible, it spans thousands of individual files that must be downloaded in multiple steps.
* **Table of Contents Formatting:** ✔️ Yes, the Table of Contents is properly formatted.
* **File Download Barriers:** ✔️ No, there are no barriers to downloading the files.
* **File Accessibility Percentage:** 99.7% of the files are accessible.
* **Data Assessment:** Aetna duplicates their rate data for every related NPI under each TIN and for each Reporting Plan, even if rates are the same, making the data volumes very large. It is debatable whether this interpretation of the rules is acceptable, but it does at least give very good coverage of their rates across the country.

**Overall Assessment:** Aetna publishes many thousands of duplicated data and much of the data is their base fee schedules. This is especially true for Professional rates for non-hospitals, which makes the data integrity questionable as it cannot be verified via Hospital MRFs. This is an ongoing issue that must be addressed and as such Aetna has been downgraded from "Good" to "Acceptable".

_As of May 2024 there are new rate limits on Aetna's Table of Contents API - this causes the process to download all of the ToC files to take a little longer, but is not unreasonable._
