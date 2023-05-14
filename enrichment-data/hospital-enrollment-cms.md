---
description: >-
  List of Hospital Enrollments from the CMS Public datasets, updated quarterly
  with other Enrichment Data.
---

# Hospital Enrollment CMS

{% hint style="info" %}
This data is automatically included with any purchase
{% endhint %}

## Schema: ENRICHMENT\_DATA

## Table Name: HOSPITAL\_ENROLLMENT

**Hospital Enrollments Data Dictionary**

| Term Name                    | Variable Name                | Description                                                                                                                                                                                                                                                                                                                                                                                                             | Type | Length |
| ---------------------------- | ---------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---- | ------ |
| Enrollment ID                | ENROLLMENT ID                | Hospital's enrollment ID.An enrollment ID is a unique 15-digit alphanumeric identifier that is assigned to each new provider enrollment application. All enrollment-level information (e.g. enrollment type, enrollment state, provider specialty and reassignment of benefits) is linked through the enrollment ID.                                                                                                    | CHAR | 15     |
| Enrollment State             | ENROLLMENT STATE             | Hospital's enrollment state, see State Code Reference Table for description of values.                                                                                                                                                                                                                                                                                                                                  | CHAR | 2      |
| Provider Type Code           | PROVIDER TYPE CODE           | Enrollment application and specialty type code, see Provider Type Code Reference Table for the full list of Part A provider types.                                                                                                                                                                                                                                                                                      | CHAR | 5      |
| Provider Type Text           | PROVIDER TYPE TEXT           | Description for Provider Type Code.                                                                                                                                                                                                                                                                                                                                                                                     | CHAR | 200    |
| NPI                          | NPI                          | Hospital's National Provider Identifier (NPI).An NPI is a unique 10-digit numeric identifier that all providers must obtain before enrolling in Medicare. It is assigned to health care providers upon application through the National Plan and Provider Enumeration System (NPPES).                                                                                                                                   | CHAR | 10     |
| Multiple NPI Flag            | MULTIPLE NPI FLAG            | A flag that indicates whether the hospital has more than 1 NPI (Y/N). If yes, additional NPIs are displayed in the Hospital Additional NPIs file.                                                                                                                                                                                                                                                                       | CHAR | 1      |
| CCN                          | CCN                          | Hospital's CMS Certification Number (CCN), formerly called an OSCAR Number.                                                                                                                                                                                                                                                                                                                                             | CHAR | 15     |
| Associate ID                 | ASSOCIATE ID                 | Hospital's PECOS Associate Control (PAC) ID.A PAC ID is a unique 10-digit numeric identifier that is assigned to each individual or organizational provider in PECOS. All entity-level information (e.g. tax identification numbers and provider names) is linked through the PAC ID. A PAC ID may be associated with multiple enrollment IDs if the provider is enrolled multiple times under different circumstances. | CHAR | 10     |
| Organization Name            | ORGANIZATION NAME            | Hospital's legal business name.                                                                                                                                                                                                                                                                                                                                                                                         | CHAR | 70     |
| Doing-Business-As Name       | DOING BUSINESS AS NAME       | Hospital's doing-business-as name.                                                                                                                                                                                                                                                                                                                                                                                      | CHAR | 70     |
| Incorporation Date           | INCORPORATION DATE           | Date on which the business is incorporated.                                                                                                                                                                                                                                                                                                                                                                             | NUM  | 8      |
| Incorporation State          | INCORPORATION STATE          | State in which the business is incorporated, see State Code Reference Table for description of values.                                                                                                                                                                                                                                                                                                                  | CHAR | 2      |
| Organization Type Structure  | ORGANIZATION TYPE STRUCTURE  | Hospital's organization structure type.                                                                                                                                                                                                                                                                                                                                                                                 | CHAR | 60     |
| Organization OtherType Text  | ORGANIZATION OTHER TYPE TEXT | Description of the organization structure if Organization Type Structure is "OTHER".                                                                                                                                                                                                                                                                                                                                    | CHAR | 60     |
| Proprietary/Non- Profit Flag | PROPRIETARY\_NONPROFIT       | "P" if the business is registered as proprietor with the IRS;"N" if registered as non-profit.                                                                                                                                                                                                                                                                                                                           | CHAR | 1      |
| Address Line 1               | ADDRESS LINE 1               | Address line 1 of the hospital's practice location address.                                                                                                                                                                                                                                                                                                                                                             | CHAR | 55     |
| Address Line 2               | ADDRESS LINE 2               | Address line 2 of the hospital's practice location address.                                                                                                                                                                                                                                                                                                                                                             | CHAR | 55     |
| City                         | CITY                         | City of the hospital's practice location address.                                                                                                                                                                                                                                                                                                                                                                       | CHAR | 30     |
| State                        | STATE                        | State of the hospital's practice location address, see State Code Reference Table for description of values.                                                                                                                                                                                                                                                                                                            | CHAR | 2      |
| Zip Code                     | ZIP CODE                     | Zip code of the hospital's practice location address.                                                                                                                                                                                                                                                                                                                                                                   | CHAR | 15     |
| Practice Location Type       | PRACTICE LOCATION TYPE       | Type of practice location.                                                                                                                                                                                                                                                                                                                                                                                              | CHAR | 32     |
| Location Other Type Text     | LOCATION OTHER TYPETEXT      | Other type of practice location found in the CMS -855 form.                                                                                                                                                                                                                                                                                                                                                             | CHAR | 60     |

| **Term Name**                      | **Variable Name**             | **Description**                                                                                                   | **Type** | **Length** |
| ---------------------------------- | ----------------------------- | ----------------------------------------------------------------------------------------------------------------- | -------- | ---------- |
| Subgroup – General Flag            | SUBGROUP – GENERAL            | A flag that indicates if the hospital's subgroup/unit is general (Y/N; blank if not reported).                    | CHAR     | 1          |
| Subgroup – Acute Care Flag         | SUBGROUP – ACUTE CARE         | A flag that indicates if the hospital's subgroup/unit is acute care (Y/N; blank if not reported).                 | CHAR     | 1          |
| Subgroup – Alcohol/Drug Flag       | SUBGROUP – ALCOHOL DRUG       | A flag that indicates if the hospital's subgroup/unit is alcohol/drug (Y/N; blank if not reported).               | CHAR     | 1          |
| Subgroup – Children's HospitalFlag | SUBGROUP – CHILDRENS          | A flag that indicates if the hospital's subgroup/unit is children's hospital (Y/N; blank if not reported).        | CHAR     | 1          |
| Subgroup – Long­Term Flag          | SUBGROUP – LONG-TERM          | A flag that indicates if the hospital's subgroup/unit is long - term (Y/N; blank if not reported).                | CHAR     | 1          |
| Subgroup –Psychiatric Flag         | SUBGROUP -PSYCHIATRIC         | A flag that indicates if the hospital's subgroup/unit is psychiatric (Y/N; blank if not reported).                | CHAR     | 1          |
| Subgroup –Rehabilitation Flag      | SUBGROUP -REHABILITATION      | A flag that indicates if the hospital's subgroup/unit is rehabilitation (Y/N; blank if not reported).             | CHAR     | 1          |
| Subgroup – Short­Term Flag         | SUBGROUP – SHORT-TERM         | A flag that indicates if the hospital's subgroup/unit is short­term (Y/N; blank if not reported).                 | CHAR     | 1          |
| Subgroup – Swing­Bed Approved Flag | SUBGROUP – SWING-BEDAPPROVED  | A flag that indicates if the hospital's subgroup/unit is swing - bed approved (Y/N; blank if not reported).       | CHAR     | 1          |
| Subgroup – Psychiatric UnitFlag    | SUBGROUP –PSYCHIATRIC UNIT    | A flag that indicates if the hospital's subgroup/unit is psychiatric unit (Y/N; blank if not reported).           | CHAR     | 1          |
| Subgroup –Rehabilitation UnitFlag  | SUBGROUP –REHABILITATION UNIT | A flag that indicates if the hospital's subgroup/unit is rehabilitation unit (Y/N; blank if not reported).        | CHAR     | 1          |
| Subgroup – Specialty HospitalFlag  | SUBGROUP – SPECIALTYHOSPITAL  | A flag that indicates if the hospital's subgroup/unit is specialty hospital (Y/N; blank if not reported).         | CHAR     | 1          |
| Subgroup – Other Flag              | SUBGROUP – OTHER              | A flag that indicates if the hospital's subgroup/unit is not listed on the CMS form (Y/N; blank if not reported). | CHAR     | 1          |
| Subgroup – Other Text              | SUBGROUP – OTHER TEXT         | Other type of hospital subgroup/unit that is not listed on the CMS form.                                          | CHAR     | 60         |

| State Code | Description               |
| ---------- | ------------------------- |
| AK         | Alaska                    |
| AL         | Alabama                   |
| AR         | Arkansas                  |
| AS         | American Samoa            |
| AZ         | Arizona                   |
| CA         | California                |
| CO         | Colorado                  |
| CT         | Connecticut               |
| DC         | District of Columbia      |
| DE         | Delaware                  |
| FL         | Florida                   |
| GA         | Georgia                   |
| GU         | Guam                      |
| HI         | Hawaii                    |
| IA         | Iowa                      |
| ID         | Idaho                     |
| IL         | Illinois                  |
| IN         | Indiana                   |
| KS         | Kansas                    |
| KY         | Kentucky                  |
| LA         | Louisiana                 |
| MA         | Massachusetts             |
| MD         | Maryland                  |
| ME         | Maine                     |
| MI         | Michigan                  |
| MN         | Minnesota                 |
| MO         | Missouri                  |
| MP         | Mariana Islands, Northern |
| MS         | Mississippi               |
| MT         | Montana                   |
| NC         | North Carolina            |
| ND         | North Dakota              |
| NE         | Nebraska                  |
| NH         | New Hampshire             |
| NJ         | New Jersey                |
| NM         | New Mexico                |
| NV         | Nevada                    |
| NY         | New York                  |
| OH         | Ohio                      |
| OK         | Oklahoma                  |
| OR         | Oregon                    |
| PA         | Pennsylvania              |
| PR         | Puerto Rico               |
| PW         | Palau                     |
| RI         | Rhode Island              |
| SC         | South Carolina            |
| SD         | South Dakota              |
| TN         | Tennessee                 |
| TX         | Texas                     |
| UT         | Utah                      |
| VA         | Virginia                  |
| VI         | Virgin Islands            |
| VT         | Vermont                   |

| **Code** | **Description** |
| -------- | --------------- |
| WA       | Washington      |
| WI       | Wisconsin       |
| WV       | West Virginia   |
| WY       | Wyoming         |

**Provider Type Code Reference Table**

| **Code** | **Description**                                                                              |
| -------- | -------------------------------------------------------------------------------------------- |
| 00-00    | PART A PROVIDER - RELIGIOUS NON-MEDICAL HEALTH CARE INSTITUTION (RNHCI)                      |
| 00-01    | PART A PROVIDER - COMMUNITY MENTAL HEALTH CENTER                                             |
| 00-02    | PART A PROVIDER - COMPREHENSIVE OUTPATIENT REHABILITATION FACILITY                           |
| 00-03    | PART A PROVIDER - END-STAGE RENAL DISEASE FACILITY (ESRD)                                    |
| 00-04    | PART A PROVIDER - FEDERALLY QUALIFIED HEALTH CENTER (FQHC)                                   |
| 00-05    | PART A PROVIDER - HISTOCOMPATIBILITY LABORATORY                                              |
| 00-06    | PART A PROVIDER - HOME HEALTH AGENCY                                                         |
| 00-08    | PART A PROVIDER - HOSPICE                                                                    |
| 00-09    | PART A PROVIDER - HOSPITAL                                                                   |
| 00-10    | PART A PROVIDER - INDIAN HEALTH SERVICES FACILITY                                            |
| 00-13    | PART A PROVIDER - ORGAN PROCUREMENT ORGANIZATION (OPO)                                       |
| 00-14    | PART A PROVIDER - OUTPATIENT PHYSICAL THERAPY/OCCUPATIONAL THERAPY/SPEECH PATHOLOGY SERVICES |
| 00-17    | PART A PROVIDER - RURAL HEALTH CLINIC                                                        |
| 00-18    | PART A PROVIDER - SKILLED NURSING FACILITY                                                   |
| 00-19    | PART A PROVIDER - OTHER                                                                      |
| 00-85    | PART A PROVIDER - CRITICAL ACCESS HOSPITAL                                                   |

#### Additional Information

[https://data.cms.gov/provider-characteristics/hospitals-and-other-facilities/hospital-enrollments/data](https://data.cms.gov/provider-characteristics/hospitals-and-other-facilities/hospital-enrollments/data)
