---
description: >-
  This data contains the price transparency machine-readable files provided by
  BCBS North Dakota that have been normalized into easy-to-use tables.
---

# 🟢 Blue Cross Blue Shield North Dakota

### Payerset Notes

**Table of Contents**

[https://mrf.payerset.com/bcbs-north-dakota](https://mrf.payerset.com/bcbs-north-dakota)

### Compliance Scorecard

Overall Rating: <mark style="color:green;">**5/5**</mark>**&#x20;- Excellent**

<table data-view="cards"><thead><tr><th></th><th></th><th></th><th></th><th data-hidden data-card-cover data-type="files"></th></tr></thead><tbody><tr><td><strong>Table of Contents</strong></td><td><strong>★★★★★</strong></td><td><mark style="color:green;"><strong>5/5</strong></mark></td><td><ul><li>Are the MRFs kept up to date each month? </li><li>Is the Table of Contents link easily accessible?</li><li>Is the Table of Contents properly formatted?</li></ul></td><td></td></tr><tr><td><strong>File Accessibility</strong></td><td><strong>★★★★★</strong></td><td><mark style="color:green;"><strong>5/5</strong></mark></td><td><ul><li>Are there any barriers to downloading the files?</li><li>Do the Table of Contents links expire before publishing new links?</li></ul></td><td></td></tr><tr><td><strong>Data Quality</strong></td><td><strong>★★★★★</strong></td><td><mark style="color:green;"><strong>5/5</strong></mark></td><td><ul><li><p>What percentage of the MRFs are properly formatted and parseable</p><ul><li>5 Stars - 100%</li><li>4 Stars - 80%...</li></ul></li></ul></td><td></td></tr></tbody></table>

### Schema: BCBS\_NORTH\_DAKOTA

**Rates Records**: 13,201,746,053

**Provider Records**: 122,523,145,976

### Additional Observations

**Machine-Readable Price Transparency Files Review**

* **MRFs Up-to-Date:** ✔️ Yes, the MRFs are kept up to date each month.
* **Table of Contents Accessibility:** ✔️ The Table of Contents link is easily accessible.
* **Table of Contents Formatting:** ✔️ Yes, the Table of Contents is properly formatted.
* **File Download Barriers:** ✔️ No barriers to downloading the MRFs.
* **File Accessibility Percentage:** ✔️ 100% of the files are accessible.

**Overall Assessment:** BCBS North Dakota generally does a great job of posting these files each month. They duplicate provider data substantially, but that can be a result of interpretation of the CMS rules.

**As of May 2024 this has still not been resolved:**

There are some issues with one of their Dental files that we weren't able to manually fix, but we did not ding them a star for it. See the full JSON error below:

#### Error Explanation - 1 file is corrupted, specifically the Dental In Network Rates file

An error was encountered while processing a file with ID `907002cf-7876-4b2f-95b2-da2b606393e1`. The error indicates a JSON parsing issue, specifically an "unallowed token" error at a certain point in the JSON text.

**Error Details**

* **File ID**: `907002cf-7876-4b2f-95b2-da2b606393e1`
* **Error**: Parse error
* **Issue**: Unallowed token in JSON
* **Location**: Just after a list of numbers before `], "tin": { "type": "ein", "val`

**Possible Cause**

The token causing the error is likely due to a trailing comma after the last item in the array (right before the closing square bracket `]`). JSON does not allow trailing commas in arrays or objects.

**Corrected JSON Excerpt**

```json
...1891139473,1932830130], "tin": { "type": "ein", "value"...
```
