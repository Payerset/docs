# 📇 Nonstandard Codes

In some cases, negotiated rates in healthcare price transparency files use codes that fall outside the standard CPT, HCPCS, RC, or MS-DRG structures. These can represent carveouts or other situations where standard codes aren’t used. In the Payerset platform, these are referred to as nonstandard codes.

You can easily find and work with nonstandard codes in two places:

1. The Nonstandard Codes module
2. Any Fee Schedule you generate (nonstandard codes are always included by default)

## Accessing and Using Nonstandard Codes in Payerset

#### Access the Nonstandard Codes Module

From the homepage, click the Nonstandard Codes menu item and select “Create an Analysis”.

First, use the filters on the left-hand panel along with the search bar to choose the Payers you want to include in this analysis.

<figure><img src="https://lh7-rt.googleusercontent.com/docsz/AD_4nXfleOPUDqW6DYwh0yflzvwVZmOI0i-wchExvnHPnQqiw0MBCqfDlydAohAIcyy3MUFHpkHpbtTvkDiY95bqdqyj8dszsOyHhssygX5Fblp5BLHurfIMbX2OIKV-2O2ccM3oidnEjw?key=SGZTz22p2LTiPFHHBn7quQ" alt=""><figcaption></figcaption></figure>

Similarly, use the filters along with the search bar to choose the Payers to include in your final nonstandard code analysis. You can also name your analysis to refer back to or re-run to pull new data in the future.

<figure><img src="https://lh7-rt.googleusercontent.com/docsz/AD_4nXcQHc5itlsuNe-PyOkm0HLjXffOCCM7jbkAhbotCEvEysfPhUVh87H0iGzgyf2qdH6JA_FPThGLYe-46TFDceaHzAeQtnmU30RWX00z24jqBv1Qeijo5lwux5xNEmzec9AWNxMj?key=SGZTz22p2LTiPFHHBn7quQ" alt=""><figcaption></figcaption></figure>

Once you've set these parameters and clicked run your analysis, you can view the codes in two ways:

* The Data tab for a quick overview of available codes and counts.
* The Explore tab for deeper exploration and detailed analysis

<figure><img src="https://lh7-rt.googleusercontent.com/docsz/AD_4nXdOn7kcDKY_FkYPOwMAUSBRC-HI75IbmAiK3jswVr9aHRzWvhV7IW4zVhRfD8ix0CO5Y4vYzUSqyL44tIvJPjBdn8bztNw2DB7ed1GbBnQrdTYCn3aeT70EA-01AFofedp5kNfv?key=SGZTz22p2LTiPFHHBn7quQ" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
For guidance on using the Explore tab, see our [Analyzing Your Data page](analyzing-your-data.md)
{% endhint %}

## Viewing Nonstandard Codes in Fee Schedules

Every Fee Schedule in Payerset automatically includes any nonstandard codes that appear in the source data. These codes are displayed alongside standard codes, ensuring you get a complete view of all available negotiated rates.No additional setup is required—nonstandard codes are included by default.

Within a Fee Schedule, you can identify nonstandard codes by the value “CSTM-ALL” within the “MRF Billing Code Type” field/filter.

<figure><img src="../../.gitbook/assets/mrf_billing_code.gif" alt=""><figcaption></figcaption></figure>

## Helpful Tips for Working with Nonstandard Codes

* We have cleaned the data for ease of use and nonstandard codes show in the field “Billing Code” along with standard code. If you are looking at the files directly, this may not match.
* Use the field “MRF Billing Code Name” to see the description of the nonstandard code<br>
