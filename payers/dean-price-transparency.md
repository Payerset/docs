---
description: >-
  This data contains the price transparency machine-readable files provided by
  Dean Health Plan that have been normalized into easy-to-use tables.
---

# 🔴 Dean Health Plan

### Payerset Notes

**Table of Contents**

[https://mrf.payerset.com/dean](https://mrf.payerset.com/dean)

### Compliance Scorecard

Overall Rating: <mark style="color:red;">**2/5**</mark>**&#x20;- Below Expectations**

<table data-view="cards"><thead><tr><th></th><th></th><th></th><th></th><th data-hidden data-card-cover data-type="files"></th></tr></thead><tbody><tr><td><strong>Table of Contents</strong></td><td><strong>★★★★★</strong></td><td><mark style="color:green;"><strong>5/5</strong></mark></td><td><ul><li>Are the MRFs kept up to date each month? </li><li>Is the Table of Contents link easily accessible?</li><li>Is the Table of Contents properly formatted?</li></ul></td><td></td></tr><tr><td><strong>File Accessibility</strong></td><td><strong>★★★★☆</strong></td><td><mark style="color:yellow;"><strong>4/5</strong></mark></td><td><ul><li>Are there any barriers to downloading the files?</li><li>Do the Table of Contents links expire before publishing new links?</li></ul></td><td></td></tr><tr><td><strong>Data Quality</strong></td><td><strong>★★☆☆☆</strong></td><td><mark style="color:red;"><strong>2/5</strong></mark></td><td><ul><li><p>What percentage of the MRFs are properly formatted and parseable</p><ul><li>5 Stars - 100%</li><li>4 Stars - 80%...</li></ul></li></ul></td><td></td></tr></tbody></table>

### Schema: DEAN

**Rates Records**: 2,279,448,413

**Provider Records**: 5,914,864

### Additional Observations

**Machine-Readable Price Transparency Files Review**

* **MRFs Up-to-Date:** ✔️ Yes, the MRFs are kept up to date each month.
* **Table of Contents Accessibility:** ✔️ The Table of Contents link is easily accessible.
* **Table of Contents Formatting:** ✔️ Yes, the Table of Contents is properly formatted.
* **File Download Barriers:** ✔️ No barriers to downloading the MRFs, though the provider links do not work properly (see below).
* **File Accessibility Percentage:** ❌ The provider references in many of the Dean MRFs link to non-existent files, making it impossible to trace the rates back to the respective providers.

**Overall Assessment:** Dean does have MRF data but due to many of the provider links missing is not compliant with the mandate.

Here is example MRF with 470 rates but invalid links to their provider data: [https://app.deancare.com/price-transparency/Files/InNetwork/DEAN/2024-03-01\_MANCARECOML2\_DHCF\_rates\_in-network-rates.json](https://app.deancare.com/price-transparency/Files/InNetwork/DEAN/2024-03-01_MANCARECOML2_DHCF_rates_in-network-rates.json)

The provider references in many of the Dean MRFs link to non-existent files, making it impossible to trace the rates back to the respective providers. While Dean’s files do include DRGs, such as 470, the absence of provider data means we cannot determine to which providers these rates are associated, leading to their exclusion from our Rate Explorer portal.

This Dean issue exists for both February and March, so while it appears to be a mistake, it’s a mistake they have not caught. We are reporting this Dean issue through official CMS channels.\
As you can see in the screenshot below, there are 36 rates present for code 470, which may give you an idea of the ranges they are paying out for this service, but unfortunately, we are unable to map those back to their specific providers.

<figure><img src="../.gitbook/assets/image (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
