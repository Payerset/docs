---
description: Learn how to create an analysis
---

# 📊 Creating a Rate Comparison

### Analysis Creation

Now onto the fun part, creating an analysis!

The Comparison Analysis tool allows you to select a series of billing codes, payers, and providers to compare rates across these dimensions.

You can create a Comparison Analysis in two ways:

1. Select "Create a Rate Comparison" from the main tile on the home page OR
2. &#x20;in the top right-hand side of the screen, you will see a button called **New analysis** - click that to start choosing billing codes, payers, and providers for your dataset.

We'll select "Create a Rate Comparison from the homescreen.

<figure><img src="../../.gitbook/assets/image (8).png" alt=""><figcaption><p>Create a Rate Comparison</p></figcaption></figure>

### Choose Date Snapshot

Payerset provides you with both current and historical views of rates. The system automatically populates data with the most recent available date. Additionally, you can select an alternative snapshot date from the dropdown menu to explore how rates have evolved over time, gaining valuable insights into trends and changes.

{% hint style="warning" %}
Stay tuned for further enhancements to utilizing these historical rates for advanced analysis and comparisons&#x20;
{% endhint %}

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

### Choose your Services (Billing Codes)

First, choose the billing codes you are interested in analyzing. You can filter on code type (CPT/HCPCS/DRG) or categories/subcategories that Payerset has created. To learn more, see our [Payerset Billing Code Classification documentation](../../data-lake/payerset-billing-code-classification.md). You can also use the search to find specific codes or code families, like "2744" or "Knee".

<figure><img src="../../.gitbook/assets/Kapture 2024-12-18 at 13.01.52.gif" alt=""><figcaption></figcaption></figure>

You can select codes individually or click the "Select all" button at the bottom to select all of the codes you have in your current filtered view. We'll use the "Select all" option.

{% hint style="warning" %}
There is currently a limitation of 300 codes that can be chosen at once, which should encompass any single subcategory, but if you are looking to do a broader study, a good strategy is to copy your analysis and edit it to add different codes.
{% endhint %}

### Choose your Payers

Similar to billing codes, on the "Choose your payers" screen you will be able to select up to 5 payers to compare. You are able to filter by Category and Region to make the list easier to navigate.

<figure><img src="../../.gitbook/assets/Kapture 2024-12-18 at 13.12.11 (1).gif" alt=""><figcaption><p>Choose your payers</p></figcaption></figure>

Select "Next, choose your Providers" to move to the next step.

### Choose your Providers

Next, the Rate Comparison tool will crawl our data catalog using your selections of billing codes and payers to find which providers have negotiated rates posted in the payer price transparency data.&#x20;

{% hint style="warning" %}
It's important to note that this process is continuing to evolve and payers are getting more compliant each month, so it's possible that providers that _should_ have rates for a particular code and payer do not (yet).
{% endhint %}

From here, you will be able to search for providers you want to analyze.

As with the previous screens, you can also search by name or NPI to find specific providers.

<figure><img src="../../.gitbook/assets/Kapture 2024-12-18 at 13.18.17.gif" alt=""><figcaption><p>Choose your providers</p></figcaption></figure>

### Generate Your Analysis

Finally, we're ready to create our analysis. It's best to give your analysis a descriptive name so it's easy to identify later. When you're ready, click "Generate Analysis" and Rate Explorer will crawl our data lake to pull all of the data you requested. This can take from a few seconds up to 10 minutes, depending on which payers and how many codes and providers you selected, but the progress bar will help you get a feel for how long the analysis will take to run.

Once the analysis completes, you will see some KPIs detailing how many rates were found, as well as a visualization showing the distribution of those rates by billing code for each payer.

<figure><img src="../../.gitbook/assets/Kapture 2024-12-18 at 20.00.36 (3).gif" alt=""><figcaption><p>Generating an analysis</p></figcaption></figure>

In the next section of this guide, we'll go over how to create a fee schedule. If you'd like to skip ahead to learn how to analyze the comparison we just created, feel free to use the left-hand panel instead to skip to the "Analyzing your Data" section.
