---
description: Learn how to create an analysis
---

# ⌨️ Creating an Analysis

### Analysis Creation

Now onto the fun part, creating an analysis!

In the top right-hand side of the screen, you will see a button called **New analysis** - click that to start choosing billing codes, payers, and providers for your dataset.

<figure><img src="../../.gitbook/assets/CleanShot 2024-03-19 at 21.09.36@2x.png" alt=""><figcaption><p>Create a new analysis</p></figcaption></figure>

### Billing Codes

First, choose the billing codes you are interested in analyzing. You can filter on code type (CPT/HCPCS/DRG) or categories/subcategories that Payerset has created. To learn more, see our [Payerset Billing Code Classification documentation](../../data-lake/payerset-billing-code-classification.md). You can also use the search to find specific codes or code families, like "2744" or "Knee"

You can select codes individually or click the "Select all" button at the bottom to select all of the codes you have in your current filtered view.

{% hint style="warning" %}
There is currently a limitation of 300 codes that can be chosen at once, which should encompass any single subcategory, but if you are looking to do a broader study, a good strategy is to copy your analysis and edit it to add different codes.
{% endhint %}

<figure><img src="../../.gitbook/assets/CleanShot 2024-03-19 at 21.12.36.gif" alt=""><figcaption><p>Select billing codes GIF</p></figcaption></figure>

### Payers

Similar to billing codes, on the next screen you will be able to select up to 5 payers to compare. You are able to filter by Category and Region to make the list easier to navigate.

<figure><img src="../../.gitbook/assets/CleanShot 2024-03-19 at 21.22.37.gif" alt=""><figcaption><p>Select payers GIF</p></figcaption></figure>

### Providers

Next, Rate Explorer will crawl our data catalog using your selections of billing codes and payers to find which providers have negotiated rates posted in the payer price transparency data.&#x20;

{% hint style="warning" %}
It's important to note that this process is continuing to evolve and payers are getting more compliant each month, so it's possible that providers that _should_ have rates for a particular code and payer do not (yet).
{% endhint %}

From here, you will be able to search for providers you want to analyze.

<figure><img src="../../.gitbook/assets/CleanShot 2024-03-19 at 21.30.14.gif" alt=""><figcaption><p>Select providers GIF</p></figcaption></figure>

You'll see in the video above that BIRMINGHAM MEDICAL CENTER has no available rates for the selected billing codes and payers, which is indicated by the "no rates found" text next to the NPI.

As with the previous screens, you can also search by name or NPI to find specific providers.

### Generate Your Analysis

Finally, we're ready to create our analysis. It's best to give your analysis a descriptive name so it's easy to identify later. When you're ready, click "Generate Analysis" and Rate Explorer will crawl our data lake to pull all of the data you requested. This can take from a few seconds up to 10 minutes, depending on which payers and how many codes and providers you selected, but the progress bar will help you get a feel for how long the analysis will take to run.

<figure><img src="../../.gitbook/assets/CleanShot 2024-03-19 at 21.41.03.gif" alt=""><figcaption><p>Generate Analysis GIF</p></figcaption></figure>

Once the analysis completes, you will see some KPIs detailing how many rates were found, as well as a visualization showing the distribution of those rates by billing code for each payer.

<figure><img src="../../.gitbook/assets/CleanShot 2024-03-19 at 21.42.27@2x.png" alt=""><figcaption></figcaption></figure>

In the next section, we'll go over how to use this visualization, as well as the data details table and the exploration pivot table.
