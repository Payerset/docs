---
description: >-
  This data contains the price transparency machine-readable files provided by
  CareSource that have been normalized into easy-to-use tables.
---

# 🟠 CareSource

### Payerset Notes

**Table of Contents**

[https://mrf.payerset.com/caresource](https://mrf.payerset.com/caresource)

{% hint style="info" %}
Note that the employer files for CareSource are created and maintained by UnitedHealthcare - https://mrf.payerset.com/united-healthcare
{% endhint %}

### Compliance Scorecard

Overall Rating: <mark style="color:orange;">**3/5**</mark>** - Below Expectations**

<table data-view="cards"><thead><tr><th></th><th></th><th></th><th></th><th data-hidden data-card-cover data-type="files"></th></tr></thead><tbody><tr><td><strong>Table of Contents</strong></td><td><strong>★★★★☆</strong></td><td><mark style="color:yellow;"><strong>4/5</strong></mark></td><td><ul><li>Are the MRFs kept up to date each month? </li><li>Is the Table of Contents link easily accessible?</li><li>Is the Table of Contents properly formatted?</li></ul></td><td></td></tr><tr><td><strong>File Accessibility</strong></td><td><strong>★★★☆☆</strong></td><td><mark style="color:orange;"><strong>3/5</strong></mark></td><td><ul><li>Are there any barriers to downloading the files?</li><li>Do the Table of Contents links expire before publishing new links?</li></ul></td><td></td></tr><tr><td><strong>Data Quality</strong></td><td><strong>★★☆☆☆</strong></td><td><mark style="color:red;"><strong>2/5</strong></mark></td><td><ul><li><p>What percentage of the MRFs are properly formatted and parseable</p><ul><li>5 Stars - 100%</li><li>4 Stars - 80%...</li></ul></li></ul></td><td></td></tr></tbody></table>

### Schema: CARESOURCE

**Rates Records**: 63,104

**Provider Records**: 107,727

### Additional Observations

**Machine-Readable Price Transparency Files Review**

* **MRFs Up-to-Date:** ✔️ Yes, the MRFs are kept up to date each month.
* **Table of Contents Accessibility:** ✔️ The Table of Contents link is easily accessible.
* **Table of Contents Formatting:** ✔️ Yes, the Table of Contents is properly formatted.
* **File Download Barriers:** ❌ Several files linked in the Table of Contents are not accessible.
* **File Accessibility Percentage:** ❌ only a few of the files are accessible and several of those have no data. Several files are corrupted (see below).

**Overall Assessment:** CareSource is currently not compliant with the CMS regulations and has several files that are invalid from a format and data perspective. These are potentially fixable manually but would require us to guess at what the data should be, and as such we do not manually change these particular files.

There were a number of errors in the CareSource files, see below for details:

```python
An error occurred while processing the file 46beb1d9-f84d-4df8-bcd8-5b60dfd43b81: parse error: unallowed token at this point in JSON text
                                       ],"provider_references":[  ],"l
                     (right here) ------^

An error occurred while processing the file 103649fa-1a67-4e24-a68e-6ec5b53862fb: parse error: unallowed token at this point in JSON text
          :"Individual","in_networks":[,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,
                     (right here) ------^

An error occurred while processing the file fe3e3dd6-6e99-42ef-a8f7-0ed9a11b8c68: lexical error: invalid char in json text.
                                       PK                     (right here) ------^

An error occurred while processing the file 8651e28b-ad78-49d0-8640-9c8096b7d69e: lexical error: invalid char in json text.
                                       PK                     (right here) ------^
```
