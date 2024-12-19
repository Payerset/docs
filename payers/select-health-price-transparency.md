---
description: >-
  This data contains the price transparency machine-readable files provided by
  Select Health that have been normalized into easy-to-use tables.
---

# 🟠 Select Health

### Payerset Notes

**Table of Contents**

[https://mrf.payerset.com/select-health](https://mrf.payerset.com/select-health)

{% hint style="info" %}
We have created a Table of Contents in order to make the files parseable, but **please note** that this does not come from Select Health and **may have incorrect or missing data**. This is intended for us and others to be able to parse the negotiated rates as part of our standardized process.

You can find the Table of Contents we created at this link:

[https://mrf.payerset.com/select-health-toc](https://mrf.payerset.com/select-health-toc)
{% endhint %}

### Compliance Scorecard

Overall Rating: <mark style="color:orange;">**3/5**</mark>**&#x20;- Needs Improvement**

<table data-view="cards"><thead><tr><th></th><th></th><th></th><th></th><th data-hidden data-card-cover data-type="files"></th></tr></thead><tbody><tr><td><strong>Table of Contents</strong></td><td><strong>★★☆☆☆</strong></td><td><mark style="color:orange;"><strong>2/5</strong></mark></td><td><ul><li>Are the MRFs kept up to date each month? </li><li>Is the Table of Contents link easily accessible?</li><li>Is the Table of Contents properly formatted?</li></ul></td><td></td></tr><tr><td><strong>File Accessibility</strong></td><td><strong>★★★★★</strong></td><td><mark style="color:green;"><strong>5/5</strong></mark></td><td><ul><li>Are there any barriers to downloading the files?</li><li>Do the Table of Contents links expire before publishing new links?</li></ul></td><td></td></tr><tr><td><strong>Data Quality</strong></td><td><strong>★★★★★</strong></td><td><mark style="color:orange;"><strong>3/5</strong></mark></td><td><ul><li><p>What percentage of the MRFs are properly formatted and parseable</p><ul><li>5 Stars - 100%</li><li>4 Stars - 80%...</li></ul></li></ul></td><td></td></tr></tbody></table>

### Schema: SELECT\_HEALTH

**Rates Records**: 1,675,198,630

**Provider Records**: 54,734,651

### Additional Observations

**Machine-Readable Price Transparency Files Review**

* **MRFs Up-to-Date:** ✔️ Yes, the MRFs are kept up to date each month.
* **Table of Contents Accessibility:** ❌ The Table of Contents link is easily accessible.
* **Table of Contents Formatting:** ❌ Yes, the Table of Contents is properly formatted.
* **File Download Barriers:** ✔️ No barriers to downloading the MRFs.
* **File Accessibility Percentage:** ✔️ 100% of the files are accessible.

**Update November 2024**

Select Health spells billing\_code\_modifier incorrectly in their files, which deviates from the standard schema and causes parsed data to be missing this critical field. Additionally, having gone through \~10 million records manually, this field is not populated, which leaves me to believe that the process they are employing is also not entering the data values either.

<figure><img src="../.gitbook/assets/CleanShot 2024-10-31 at 14.17.34@2x.png" alt=""><figcaption></figcaption></figure>

**Overall Assessment:** Select Health has not created a Table of Contents, which makes the data difficult to get. For smaller payers, this is less of an issue, as the Table of Contents can be easily crafted, but it was a very large effort to craft a Table of Contents for Select Health and we are not sure it's 100% correct because we had to make some assumptions. Additionally, we only included "in network files" as those are what typically contain the rates (and there are many rates for Select Health in these files, all of which are accessible and updated) but there could be other naming conventions we are not aware of.
