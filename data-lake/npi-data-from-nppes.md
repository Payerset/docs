---
description: >-
  Data downloaded and stored for convenience, updated quarterly to ensure
  matching with the Payerset datasets
---

# NPI Data from NPPES

There are \~300 fields in this dataset, but the primary fields with their descriptions are listed below. All fields are available when you purchase any payer data.

The original dataset is available here from CMS: [https://npiregistry.cms.hhs.gov/search](https://npiregistry.cms.hhs.gov/search)

{% hint style="info" %}
Payerset also lightly transforms this data to provide Primary Taxonomy Code, using the switches in the dataset.
{% endhint %}

Here is a link directly to the NPPES download file: [https://mrf.payerset.com/nppes](https://mrf.payerset.com/nppes)

## Schema: ENRICHMENT\_DATA

## Table Name: NPPES

* **NPI Number:** Enter the NPI Number to search for a specific NPI. When entered, no other search criteria is needed.
* **NPI Type:** The search can be refined to search for only **Individual** (Type 1) NPIs or only **Organization** (Type 2) NPIs; or the search can include all NPIs (both Individual and Organization) by selecting **Any**. Select the desired value from the drop-down list. The NPI Type cannot be the only search criterion entered. Searching by NPI Type and Country is acceptable as long as the Country selected is not **United States**. When searching by NPI Type and the Country **United States**, at least one other search criterion field besides State must be entered.
* **Taxonomy Description:** Search for providers by their taxonomy by entering the taxonomy description.
* **Provider First Name:** This field only applies to Individual Providers. This field allows the following special characters: ampersand, apostrophe, colon, comma, forward slash, hyphen, left and right parentheses, period, pound sign, quotation mark, and semi-colon.
* **Provider Last Name:** This field only applies to Individual Providers. This field allows the following special characters: ampersand, apostrophe, colon, comma, forward slash, hyphen, left and right parentheses, period, pound sign, quotation mark, and semi-colon.
* **Organization Name:** This field only applies to Organizational Providers. This field allows the following special characters: ampersand, apostrophe, colon, comma, forward slash, hyphen, left and right parentheses, period, pound sign, quotation mark, and semi-colon. NPPES searches both the Organization Name and Other Organization Names associated with the NPI, such as the Doing Business As or Former Legal Business Name; therefore, results displayed on the NPI Registry Search Results page might contain an organization name different from the one entered in the Organization Name search criterion.
* **Authorized Official First Name:** This field only applies to Authorized Officials. This field allows the following special characters: ampersand, apostrophe, colon, comma, forward slash, hyphen, left and right parentheses, period, pound sign, quotation mark, and semi-colon.
* **Authorized Official Last Name:** This field only applies to Authorized Officials. This field allows the following special characters: ampersand, apostrophe, colon, comma, forward slash, hyphen, left and right parentheses, period, pound sign, quotation mark, and semi-colon.
* **City:** The City associated with the provider's practice location address. To search for a Military Address enter either APO or FPO into the City field. This field allows the following special characters: ampersand, apostrophe, colon, comma, forward slash, hyphen, left and right parentheses, period, pound sign, quotation mark, and semi-colon.
* **State:** The State associated with the provider's practice location address. This field cannot be used as the only search criterion. If this field is used, at least one other field, besides the NPI type and Country, must be populated. Select the desired state from the drop down list.
* **Country:** The Country associated with the provider's practice location address. This field can be used as the only search criterion as long as the value selected is neither **United States** nor **Any**. When selecting **United States** as a search criterion, at least one other search criterion field besides the NPI Type and State must be entered. Select the desired Country from the drop down list.
* **Postal Code:** The Postal Code associated with the provider's practice location address.
* **Address Type:** Primary and Secondary Address associated with the provider's practice location address. Must be used with City, State, Country, and/or Postal Code.
