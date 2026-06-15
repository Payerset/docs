---
description: >-
  This data contains the price transparency machine-readable files provided by
  Oscar that have been normalized into easy-to-use tables.
---

# 🟢 Oscar

### Payerset Notes

**Table of Contents**

[https://mrf.payerset.com/oscar](https://mrf.payerset.com/oscar)

### Compliance Scorecard

Overall Rating: <mark style="color:green;">**4/5**</mark>**&#x20;- Good**

<table data-view="cards"><thead><tr><th></th><th></th><th></th><th></th><th data-hidden data-card-cover data-type="files"></th></tr></thead><tbody><tr><td><strong>Table of Contents</strong></td><td><strong>★★★★★</strong></td><td><mark style="color:green;"><strong>5/5</strong></mark></td><td><ul><li>Are the MRFs kept up to date each month? </li><li>Is the Table of Contents link easily accessible?</li><li>Is the Table of Contents properly formatted?</li></ul></td><td></td></tr><tr><td><strong>File Accessibility</strong></td><td><strong>★★★★★</strong></td><td><mark style="color:green;"><strong>5/5</strong></mark></td><td><ul><li>Are there any barriers to downloading the files?</li><li>Do the Table of Contents links expire before publishing new links?</li></ul></td><td></td></tr><tr><td><strong>Data Quality</strong></td><td><strong>★★★★☆</strong></td><td><mark style="color:green;"><strong>4/5</strong></mark></td><td><ul><li><p>What percentage of the MRFs are properly formatted and parseable</p><ul><li>5 Stars - 100%</li><li>4 Stars - 80%...</li></ul></li></ul></td><td></td></tr></tbody></table>

### Schema: OSCAR

#### **Data Validation**

| Record Type                   | Record Count |
| ----------------------------- | ------------ |
| Providers                     | 532,044,605 |
| Rates                         | 643,320,811 |
| Nonstandard Codes             | 1 |
| Bundled Codes                 | 0 |
| Covered Services (Capitation) | 10,037 |

_Updated for May 2026._

### Additional Observations

**Machine-Readable Price Transparency Files Review**

* **MRFs Up-to-Date:** ✔️ Yes, the MRFs are kept up to date each month, though each region appears to be updated separately and at different times.
* **Table of Contents Accessibility:** ✔️ The Table of Contents link is easily accessible.
* **Table of Contents Formatting:** ✔️ Table of Contents is properly formatted.
* **File Download Barriers:** ✔️ There are no barriers to downloading the files, though there are mostly zip files that contain thousands of other files, which makes processing a little challenging.
* **File Accessibility Percentage:** ❌ 99% of the files are accessible and uncorrupted, there are a few files that are not accessible, and there is one file that is corrupted irreparably (see below). Unfortunately, this results in a lower rating.

**Overall Assessment:** Oscar publishes their files regularly and mostly very well, though there are many thousands of files inside zip folders that are linked in the Table of Contents, which can present challenges for processing and managing the data.

**File Error Details**

```
An error occurred while processing the file 68f47765-5847-4b92-b7e8-6f9800a952b1: parse error: after key and value, inside map, I expect ',' or '}'
          Ext wear ost skn barr <=4sq"","billing_code_type":"HCPCS","b
                     (right here) ------^
```
