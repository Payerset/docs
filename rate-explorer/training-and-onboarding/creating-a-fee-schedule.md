---
description: Learn how to create an analysis
---

# 💲 Creating a Fee Schedule

### Fee Schedule Creation

The Fee Schedule tool allows you to select a single NPI and payer to generate the full list of rates by billing code for that particular NPI and Payer.&#x20;

{% hint style="warning" %}
**Note:** Fee Schedules only include rates where the Billing Code Modifier is either blank, 00, 26, or TC.
{% endhint %}

There are two ways to create a new Fee Schedule. You can use the "New analysis" button in the top-right corner of the Fee Schedule page or click on "Create Fee Schedule" from the home page. We'll create our first fee schedule from the home page.

<figure><img src="../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

### Choose Date Snapshot

Similar to creating a Rate Comparison, you can also choose a date snapshot when creating Fee Schedule. The data will auto populate to the most recent available date. You can select an alternative snapshot date from the dropdown to view rates and gain insights to how they may have changed over time.

{% hint style="warning" %}
Stay tuned for further enhancements to utilizing these historical rates for advanced analysis and comparisons&#x20;
{% endhint %}

<figure><img src="../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

### Choose a Provider

Use the filters on the left-hand panel to narrow your search down to a single NPI. You can also utilize the search bar to search for specific NPIs in conjunction with the filters.

<figure><img src="../../.gitbook/assets/Kapture 2024-12-18 at 20.38.38.gif" alt=""><figcaption></figcaption></figure>

### Payers

After selecting a provider, the next step in creating the Fee Schedule is to choose a single payer, which will allow you to generate the complete fee schedule. The list of available payers on this screen will automatically filter based on the NPI you selected previously, ensuring that there will be rate data for these payers.

<figure><img src="../../.gitbook/assets/Kapture 2024-12-18 at 20.47.56.gif" alt=""><figcaption></figcaption></figure>

### Generate Your Fee Schedule

When you're ready, click "Generate Fee Schedule" and Payerset will crawl our data lake to pull all of the data you requested. This can take from a few seconds up to 10 minutes - fee schedules tend to take longer than typical comparisons because we are pulling all the rates for a given provider.

Once the analysis completes, you will see some KPIs detailing how many rates were found, as well as a visualization showing the distribution of those rates by billing code for each payer.

<figure><img src="../../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

In the next section, we'll go over how to use this visualization, as well as the data details table and the exploration pivot table.
