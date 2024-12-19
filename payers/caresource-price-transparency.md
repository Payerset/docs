---
description: >-
  This data contains the price transparency machine-readable files provided by
  CareSource that have been normalized into easy-to-use tables.
---

# 🔴 CareSource

### Payerset Notes

**Table of Contents**

[https://mrf.payerset.com/caresource](https://mrf.payerset.com/caresource)

{% hint style="info" %}
Note that the employer files for CareSource are created and maintained by UnitedHealthcare - https://mrf.payerset.com/united-healthcare
{% endhint %}

### Compliance Scorecard

Overall Rating: <mark style="color:red;">**2/5**</mark>**&#x20;- Below Expectations**

<table data-view="cards"><thead><tr><th></th><th></th><th></th><th></th><th data-hidden data-card-cover data-type="files"></th></tr></thead><tbody><tr><td><strong>Table of Contents</strong></td><td><strong>★★★★★</strong></td><td><mark style="color:green;"><strong>5/5</strong></mark></td><td><ul><li>Are the MRFs kept up to date each month? </li><li>Is the Table of Contents link easily accessible?</li><li>Is the Table of Contents properly formatted?</li></ul></td><td></td></tr><tr><td><strong>File Accessibility</strong></td><td><strong>★★★★☆</strong></td><td><mark style="color:yellow;"><strong>4/5</strong></mark></td><td><ul><li>Are there any barriers to downloading the files?</li><li>Do the Table of Contents links expire before publishing new links?</li></ul></td><td></td></tr><tr><td><strong>Data Quality</strong></td><td><strong>★★☆☆☆</strong></td><td><mark style="color:red;"><strong>2/5</strong></mark></td><td><ul><li><p>What percentage of the MRFs are properly formatted and parseable</p><ul><li>5 Stars - 100%</li><li>4 Stars - 80%...</li></ul></li></ul></td><td></td></tr></tbody></table>

### Schema: CARESOURCE

**Rates Records**: 0

**Provider Records**: 398,000

### Additional Observations

**Machine-Readable Price Transparency Files Review**

* **MRFs Up-to-Date:** ✔️ Yes, the MRFs are kept up to date each month.
* **Table of Contents Accessibility:** ✔️ The Table of Contents link is easily accessible.
* **Table of Contents Formatting:** ✔️ Yes, the Table of Contents is properly formatted.
* **File Download Barriers:** ❌ Several files linked in the Table of Contents are not accessible.
* **File Accessibility Percentage:** ❌ only a few of the files are accessible and several of those have no data. Several files are corrupted (see below).

**Overall Assessment:** CareSource is somehow less compliant as of May 2024 than previously.

Not only do they have .zip files nested within .zip files for several layers, but they also are not putting the correct fields into their machine-readable files. See the screenshot and description below:

<figure><img src="../.gitbook/assets/CleanShot 2024-05-31 at 14.26.20@2x.png" alt=""><figcaption></figcaption></figure>

The issues:

1. The main key is `in_networks` instead of `in_network` (it is not supposed to be plural)
2. The Price field should be negotiated\_price
3. The Exp\_Date field should be expiration\_date
4. The Expiration Date should not have a timestamp

These are easily-fixable issues and we hope CareSource will make the necessary changes to their files.
