# 🔍 Using Provider Lookup

Instantly access benchmarked negotiated rates by specific providers, billing codes, and plan types.

### Step 1: Find Your Provider

Start by searching for the exact practitioner or facility you want to analyze.

**Search:** Enter a specific Name or NPI into the search bar in the left column.

**Filter:** Refine the 9+ million records using the drop-down filters for State, Taxonomy, Provider Type, Tax Class, Tax Spec, Class of Trade, and City.

{% hint style="info" %}
**Note on Data Availability:** If a provider shows "0 RATES," no rate data exists in the public machine-readable files for that NPI.
{% endhint %}

<figure><img src="../../.gitbook/assets/Kapture 2026-08-26 at 16.02.05.gif" alt=""><figcaption></figcaption></figure>

### Step 2: Choose Your Pathway

#### Pathway A: Explore Rolled-Up Rates (PTA Benchmark View)

the PTA benchmark provides an overview of all of the contracted rates published for an NPI and a payer. You can read more about the definitions [behind the PTA data set here](../../data-lake/payerset-price-transparency-algorithm.md).

1. Click on the provider in the first column.
2. In the second column (Payer), search for and select your target Payer.
3. Review the summary card in the third column (PTA Rate Lookup) and click **View PTA Rates**.

<figure><img src="../../.gitbook/assets/Kapture 2026-08-26 at 16.07.49.gif" alt=""><figcaption></figcaption></figure>

#### Pathway B: Find Specific Direct Rates

Use this pathway if you know exactly what codes you need, or if PTA benchmark rates are not available for your selected payer in the guided lookup.

1. Click **Run a direct rate lookup** at the top of the page.
2. Alternatively, click **Find specific rates →** next to an individual payer card in the middle column to jump straight to a direct query for that payer.

<figure><img src="../../.gitbook/assets/Kapture 2026-08-26 at 16.10.10.gif" alt=""><figcaption></figcaption></figure>

### Step 3: Analyze Benchmarks and Drill Into Raw Data (Pathway A)

If you chose Pathway A and clicked **View PTA Rates**, you will enter the PTA Rate Summary:

**Review PTA Benchmarks:** View rolled-up reimbursement rates across billing codes, including Avg Rate, Rate Min, Rate Max, % Medicare, and algorithm Confidence scores.

**Select Specific Codes:** Click the checkbox next to one or more billing codes you want to investigate further.

**Access Raw Rates:** Click **View raw rates** in the bottom action bar. The platform will fetch the exact, unaggregated rate records underlying those specific codes.

{% hint style="success" %}
**Pro-Tip:** You can click **Back to PTA rate summary** at the top of the screen at any time to return to your aggregated benchmark view.
{% endhint %}

### Step 4: Query Direct Rates (Pathway B)

If you chose Pathway B to jump straight to specific codes or bypass the PTA rollup, you will land on the Direct Rate Lookup page. This allows you to query published rates directly by NPI, payer, and billing code — including combinations the algorithm doesn't benchmark.

**Select Provider and Payer:** Confirm your target provider is selected in Step 1, then search for and choose your specific payer from the dropdown menu. (If you clicked **Find specific rates** next to a payer in the previous step, this will be pre-filled.)

**Select Billing Codes:** In Step 2, search for specific codes by name or number, or seamlessly paste a list of codes directly into the search bar. Check the boxes next to the ones you need (you can select up to 20 codes at once).

**Review Selection Summary:** Look at the right side of your screen to verify your selected NPI, Provider, Payer, and Billing Codes.

**Pull the Data:** Click the **View raw rates** button to instantly generate the underlying, unaggregated rate records for those exact combinations.
