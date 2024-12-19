---
description: >-
  This data contains the price transparency machine-readable files provided by
  United that have been normalized into easy-to-use tables.
---

# 🟡 UnitedHealthcare

### Payerset Notes

**Table of Contents**

[https://mrf.payerset.com/united-healthcare](https://mrf.payerset.com/united-healthcare)

### Compliance Scorecard - September 2024

Overall Rating: <mark style="color:yellow;">**4/5**</mark>**&#x20;- Good**

<table data-view="cards"><thead><tr><th></th><th></th><th></th><th></th><th data-hidden data-card-cover data-type="files"></th></tr></thead><tbody><tr><td><strong>Table of Contents</strong></td><td><strong>★★★☆☆</strong></td><td><mark style="color:orange;"><strong>3/5</strong></mark></td><td><ul><li>Are the MRFs kept up to date each month? </li><li><mark style="color:red;">Is the Table of Contents link easily accessible?</mark></li><li>Is the Table of Contents properly formatted?</li></ul></td><td></td></tr><tr><td><strong>File Accessibility</strong></td><td><strong>★★★★☆</strong></td><td><mark style="color:yellow;"><strong>4/5</strong></mark></td><td><ul><li>Are there any barriers to downloading the files?</li><li>Do the Table of Contents links expire before publishing new links?</li></ul></td><td></td></tr><tr><td><strong>Data Quality</strong></td><td><strong>★★★★☆</strong></td><td><mark style="color:yellow;"><strong>4/5</strong></mark></td><td><ul><li>Is the data present in the files complete</li><li>Are there significant gaps in any type of data</li><li>Is the payer compliant with the regulations</li></ul></td><td></td></tr></tbody></table>

### Schema: UNITED\_HEALTHCARE

**Rates Records**: 2,220,454,683,340

**Provider Records**: 10,520,399,934

### Additional Observations

**Machine-Readable Price Transparency Files Review**

* **MRFs Up-to-Date:** ✔️ Yes, the MRFs are kept up to date each month.
* **Table of Contents Accessibility:** ❌ The Table of Contents link is not easily accessible, it spans thousands of individual files that must be downloaded in multiple steps.
* **Table of Contents Formatting:** ✔️ Yes, the Table of Contents is properly formatted once downloaded.
* **File Download Barriers:** ❌ Yes, there is an API that sits in front of the files that limits the download speed and availability, making it very difficult to download the data quickly.
* **File Accessibility Percentage:** 99.9% of the files are accessible.

#### API Issues

United has a new API in front of their files, which makes the download process much more difficult and slow. This can be circumvented, but there is definitely a hit to performance, and file metadata is no longer feasible to get prior to parsing the data.

#### Data Quality Issues

UnitedHealthcare publishes many MS-DRG codes with 4 digits, including a leading zero, causing them to be out of compliance in that regard (though we should assume this is not intentional). Additionally, many rates that are supposed to be tagged as `institutional` are instead tagged as `professional` which leads to confusing results. These issues must be fixed to get the maximum value out of this data for UnitedHealthcare.

These files are not accessible either:

An error occurred: 404 Client Error: Not Found for url: https://drive.google.com/uc?export=download\&id=1CMgbN8PJVHFUbVVTp34NZOpEnhLFJjGs&#x20;

An error occurred: 500 Server Error: Internal Server Error for url: https://transparency-in-coverage.uhc.com/api/v1/uhc/blobs/download?fd=2024-05-01\&fn=2024-05-01\_UMR--Inc-_Third-Party-Administrator\_REGENEXX%C2%A0-WONDR-HEALTH-AND-OC24HEALTH-DCI\_UHC-CHOICE-PLUS---PREMIUM-DESIGNATION_-ROW\_TC\_in-network-rates.json.gz&#x20;

An error occurred: 500 Server Error: Internal Server Error for url: https://transparency-in-coverage.uhc.com/api/v1/uhc/blobs/download?fd=2024-05-01\&fn=2024-05-01\_Oxford\_eviCore\_in-network-rates.json&#x20;

An error occurred: 500 Server Error: Internal Server Error for url: https://transparency-in-coverage.uhc.com/api/v1/uhc/blobs/download?fd=2024-05-01\&fn=2024-05-01\_UnitedHealthcare-of-California\_INSURER\_OPH-%EF%BF%BD-Optum-Physical-Health\_OPH-193\_in-network-rates.json.gz&#x20;

An error occurred: 500 Server Error: Internal Server Error for url: https://transparency-in-coverage.uhc.com/api/v1/uhc/blobs/download?fd=2024-05-01\&fn=2024-05-01\_TheAlliance\_AllianceComprehensiveWithBehavioralHealthAndChiropracticProviderExclusions\_in-network-rates\_json.zip

One interesting thing about UnitedHealthcare that we don't see in other payers is that they sometimes put multiple billing codes in the `billing_code` field. See the example below in which they put  `29888 AND 29882` in the same field. We do not know the reason for this, but don't believe it is in compliance.

<figure><img src="../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

**Additionally, many rates are "hidden"** in custom billing codes with the CSTM-ALL billing code type. Payerset has created a CSV export of all of this data by state so anyone can view the data. The links are available in this Google Sheet:[https://docs.google.com/spreadsheets/d/1P5ggBP6y9OeMmIUpHKE1hys0\_0sCuC72rAobbowh9t4/edit?usp=sharing](https://docs.google.com/spreadsheets/d/1P5ggBP6y9OeMmIUpHKE1hys0_0sCuC72rAobbowh9t4/edit?usp=sharing)

**Overall Assessment:** Despite the very large volumes and questionable interpretation of the assignment, UnitedHealthcare consistently puts out accessible, properly-formatted files with complete data.
