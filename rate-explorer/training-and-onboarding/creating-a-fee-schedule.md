---
description: Learn how to create an analysis
---

# 💲 Creating a Fee Schedule

### Fee Schedule Creation

The Fee Schedule tool allows you to select a single NPI and payer to generate the full list of rates by billing code for that particular NPI and Payer.&#x20;

{% hint style="warning" %}
**Note:** Fee Schedules only include rates where the Billing Code Modifier is either blank, 00, 26, or TC.
{% endhint %}

In the top right-hand side of the screen, you will see a button called **New analysis** - click that to start choosing billing codes, payers, and providers for your dataset.

<figure><img src="../../.gitbook/assets/CleanShot 2024-03-19 at 21.09.36@2x.png" alt=""><figcaption><p>Create a new fee schedule</p></figcaption></figure>

Click Fee Schedule

<figure><img src="../../.gitbook/assets/CleanShot 2024-09-24 at 11.47.50@2x.png" alt=""><figcaption></figcaption></figure>

### Choose a Provider

<figure><img src="../../.gitbook/assets/CleanShot 2024-09-24 at 11.50.29.gif" alt=""><figcaption></figcaption></figure>

### Payers

You can then choose a single payer to generate the full fee schedule. The list of payers available on this screen will automatically filter down based on the NPI you selected on the previous screen, so you can expect that there will be rate data for these payers.

<figure><img src="../../.gitbook/assets/CleanShot 2024-09-24 at 11.55.35.gif" alt=""><figcaption></figcaption></figure>

### Generate Your Fee Schedule

Finally, we're ready to create our fee schedule. It's best to give your analysis a descriptive name so it's easy to identify later. When you're ready, click "Generate Analysis" and Rate Explorer will crawl our data lake to pull all of the data you requested. This can take from a few seconds up to 10 minutes - fee schedules tend to take longer than typical comparisons because we are pulling all the rates for a given provider.

Once the analysis completes, you will see some KPIs detailing how many rates were found, as well as a visualization showing the distribution of those rates by billing code for each payer.

<figure><img src="../../.gitbook/assets/CleanShot 2024-03-19 at 21.42.27@2x.png" alt=""><figcaption></figcaption></figure>

In the next section, we'll go over how to use this visualization, as well as the data details table and the exploration pivot table.
