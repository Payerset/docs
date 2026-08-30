# Claude - Anthropic

## Connect Payerset to Claude

**Last reviewed:** August 30, 2026

Bring detailed healthcare price-transparency intelligence into Claude Team, Claude Enterprise, and Claude Code.

> **Every payer, provider, and negotiated rate included in your licensed Payerset data - available at source-record detail through Claude.**

Payerset connects as an authenticated remote Model Context Protocol (MCP) service. Each user signs in to Payerset, and Payerset applies your organization's data rights, analytical access, and usage terms to every request. Source-detail records remain distinct from any comparison, aggregation, or explanation Claude produces.

### At a glance

| Item                      | Value                                              |
| ------------------------- | -------------------------------------------------- |
| Integration type          | Organization-installed custom remote MCP connector |
| Payerset endpoint         | `https://mcp.tpi.payerset.com/mcp`                 |
| Transport                 | HTTPS with Streamable HTTP MCP                     |
| Authentication            | Per-user OAuth 2.0                                 |
| Claude administrator      | Owner or Primary Owner                             |
| Supported organizations   | Claude Team and Claude Enterprise                  |
| Optional developer access | Claude Code remote MCP                             |

No local server, browser extension, or downloadable connector package is required. Your Claude administrator registers the hosted Payerset endpoint, and each authorized user connects their own Payerset account.

### What your organization receives after purchase

Your Payerset implementation team provides:

* A provisioned Payerset organization with your licensed datasets and features.
* Authorized users and customer-specific entitlements.
* The approved Claude tool profile.
* OAuth discovery or customer-specific OAuth client information.
* Usage limits, pilot prompts, and support contacts.
* Claude Code client information when that route is included in the deployment.

If your deployment requires a customer-specific OAuth client ID or secret, Payerset delivers those details through an approved secure channel.

Never place an API key, client secret, password, access token, or refresh token in the MCP URL, email, source control, support tickets, or AI conversations.

### How the connection works

```
Authorized user
  -> Claude
  -> Payerset MCP over HTTPS
  -> Payerset authentication and entitlement checks
  -> Licensed Payerset records and analytical services
  -> Authorized result returned to the Claude conversation
```

Organization-installed connectors used through Claude web, Desktop, Cowork, and mobile are brokered through Anthropic's cloud infrastructure. A Payerset endpoint added directly with Claude Code connects from the user's Claude Code environment.

When Claude invokes Payerset:

1. Claude processes the user's request and selects an available Payerset tool.
2. Claude sends the required tool arguments to Payerset.
3. Payerset verifies the user, customer organization, entitlements, and usage policy.
4. Payerset executes the authorized query or analysis.
5. The selected result returns to Claude and becomes part of the conversation.

### Supported Claude surfaces

This guide covers organization-managed installation for Claude Team and Claude Enterprise. Anthropic supports remote connectors across Claude web, Claude Desktop, Cowork, mobile, and Claude Code. Individual features and administrative controls can vary by plan and surface.

Validate every Claude surface your organization intends to support during the pilot.

### Prerequisites

* An active Payerset subscription.
* A provisioned Payerset organization and authorized users.
* The required Payerset data, feature, and usage entitlements.
* A Claude Team or Enterprise organization.
* A Claude Owner or Primary Owner available to add the connector.
* Approval from your security and data-governance teams for the Claude-to-Payerset data flow.
* Browser access for each user to complete Payerset OAuth.

Payerset identifies the correct OAuth client option during onboarding. If a customer-specific OAuth client is used for Claude web, register this exact callback:

```
https://claude.ai/api/mcp/auth_callback
```

Do not add a trailing slash or substitute a different hostname.

### Step 1: Add Payerset to the Claude organization

A Claude Owner or Primary Owner completes the organization-level setup.

1. Sign in to the customer's Claude organization.
2. Open **Organization settings -> Connectors**.
3. Select **Add**, then **Custom**.
4. If Claude asks for the connector type, select **Web**.
5. Enter:
   * **Name:** `Payerset`
   * **Remote MCP server URL:** `https://mcp.tpi.payerset.com/mcp`
6. Continue to the authentication settings.
7. Set authentication to **Always required**.
8. Configure OAuth using the method specified in your Payerset onboarding information:
   * If Claude detects the required OAuth configuration, retain the detected setting.
   * If Payerset issued a customer-specific OAuth client, select **Use your own OAuth client** and enter the supplied client ID.
   * Enter a client secret only when Payerset supplied one for confidential-client authentication.
9. Review the configuration and select **Add**.

Anthropic is rolling out an updated connector dialog, so fields may appear on one screen or across multiple steps. The required name, endpoint, and OAuth information are the same.

### Step 2: Configure organization access

After adding Payerset, review its connector and tool permissions.

Claude Team and Enterprise Owners and Primary Owners can assign organization-level tool policies such as:

* **Always allow**
* **Needs approval**
* **Blocked**

Use these controls to narrow which Payerset tools Claude may invoke. Claude's controls supplement Payerset authorization; they do not expand a user's Payerset rights.

Payerset independently validates the user, customer organization, entitlements, contracted tool profile, and usage allowance on every call.

#### Recommended pilot

For Claude Enterprise, use a custom role to make Payerset available first to a defined pilot group.

For Claude Team, provision only the designated pilot users in Payerset and instruct those users to connect during the pilot.

After validation, expand Payerset access according to your organization's deployment policy.

### Step 3: Connect each user

After an administrator adds Payerset, every authorized user connects their own account.

1. Sign in to the correct Claude organization.
2. Open **Customize -> Connectors**.
3. Find **Payerset**. Organization-installed remote connectors are usually labeled **Custom**.
4. Select **Connect**.
5. Complete the Payerset sign-in and OAuth consent flow.
6. Return to Claude after authentication succeeds.
7. Start or open a conversation.
8. Use the **+** menu, select **Connectors**, and enable Payerset for the conversation.

Claude can now invoke the Payerset tools available to that user. Different users may receive different results or tool access based on their Payerset entitlements.

### Start using Payerset

Example requests include:

> Find the detailed published rate records available for Aetna, NPI 1346607017, and CPT 99213. Include plan/network, service setting, rate type, source context, and data vintage.

> Compare the authorized source-detail rates for this provider and billing code. Preserve the individual published rates and label any calculation as analysis.

> Identify the plans and networks represented in these price-transparency records.

> Run the Payerset analysis available under my organization's subscription and explain the result, filters, assumptions, and data vintage.

> Separate the underlying source rates from any analysis you calculate.

Payerset returns source and dataset-vintage context where available. If you request an aggregation or comparison, treat it as computed analysis rather than as an underlying published rate.

### Optional: Connect from Claude Code

Approved technical users can add the same hosted Payerset MCP endpoint to Claude Code.

Run:

```bash
claude mcp add --transport http payerset https://mcp.tpi.payerset.com/mcp
```

Then start Claude Code and enter:

```
/mcp
```

Select Payerset and complete the browser-based OAuth flow. You can also authenticate from the command line:

```bash
claude mcp login payerset
```

Claude Code uses a local loopback callback rather than the hosted Claude web callback. If your organization uses a customer-specific OAuth client for Claude Code, coordinate its callback registration with Payerset before deployment.

### Validate the deployment

Complete these checks with authorized pilot users before broad rollout.

#### Connection

* [ ] Payerset appears in the intended Claude organization.
* [ ] An authorized user completes Payerset OAuth.
* [ ] Payerset can be enabled for a Claude conversation.
* [ ] The user sees only the Payerset tools permitted by both Claude policy and Payerset entitlement.

#### Data and analytics

* [ ] A detailed, entitled price-transparency query succeeds.
* [ ] The result includes the expected payer, provider, billing code, rate, plan/network, source, and vintage context when available.
* [ ] Source records remain distinguishable from computed analysis.
* [ ] Truncation and unavailable fields are identified where applicable.
* [ ] A normal contracted analytical workflow completes successfully.

#### Authorization

* [ ] Two pilot users with different Payerset entitlements receive appropriately different access.
* [ ] An unentitled query is denied rather than approximated or fabricated.
* [ ] A tool blocked by the Claude administrator cannot be invoked.
* [ ] A tool outside the user's Payerset profile is rejected by Payerset even if called directly.

#### Operations

* [ ] Usage is attributed to the correct Payerset customer and user.
* [ ] Token expiration and refresh work as expected.
* [ ] Disconnecting and reconnecting restores a valid session.
* [ ] Revoking Payerset authorization prevents subsequent calls.

### Security and data flow

#### Per-user access

Payerset uses per-user OAuth authentication. Claude receives an authorization token for the connected user; Payerset then applies that user's organization, data, feature, and usage entitlements to every request.

A Claude administrator may restrict connector tools further. Claude administrative settings cannot grant access that the user does not already have in Payerset.

#### Data processing

Returned Payerset results become part of the Claude conversation. Do not enter data that your organization has not approved for processing in Claude. Apply your organization's Claude retention, access, and sharing policies to conversations containing Payerset results.

Anthropic states that connector transfers are encrypted and that commercial-product chats and coding sessions are not used for model training by default unless the customer explicitly opts in or provides qualifying feedback. Review Anthropic's current policies as part of your security assessment.

#### Remote network path

Organization-installed connector traffic from Claude web, Desktop, Cowork, and mobile originates from Anthropic's infrastructure, not from the user's computer. A connector added directly with `claude mcp add` originates from the user's Claude Code environment.

If your organization applies inbound, identity-provider, conditional-access, or local egress restrictions, approve the applicable path for the Claude surface being deployed.

#### Secrets

Never send Payerset passwords, OAuth client secrets, access tokens, refresh tokens, or authorization headers in Claude conversations, email, screenshots, support tickets, or shared documents.

### Service limits and response sizing

Anthropic currently documents these limits for Claude web and Desktop remote connectors:

* Tool execution timeout: approximately five minutes.
* Tool-result size: approximately 150,000 characters.

Payerset may paginate or bound large results to remain within the active Claude surface's limits. This changes how records are delivered, not the precision of the underlying rate values.

For large requests, narrow by payer, provider/NPI, billing code, geography, care setting, plan/network, data vintage, or another available Payerset dimension.

### Usage and cost governance

Payerset usage remains governed by your Payerset agreement.

* Begin with a limited pilot.
* Grant access only to licensed users.
* Review query volume, latency, authorization failures, and model-assisted usage.
* Confirm attribution to the correct Payerset organization and user.
* Contact Payerset before materially increasing the user population or workload.
* Periodically review Claude tool policies and Payerset entitlements.

One Claude question can invoke several Payerset tools. Evaluate complete workflows rather than prompt counts alone.

### Troubleshooting

| Issue                          | What to check                                                                                                                                                                |
| ------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Payerset does not appear       | Confirm the correct Claude Team/Enterprise organization, Owner installation, role policy, and Payerset provisioning.                                                         |
| Claude cannot reach the server | Confirm the exact URL `https://mcp.tpi.payerset.com/mcp` with no credentials, query parameters, or trailing slash.                                                           |
| OAuth fails                    | Confirm the active Payerset account, correct customer organization, OAuth client values, and exact callback for the Claude surface. Disconnect and retry a fresh connection. |
| A tool is missing              | Enable Payerset for the conversation, review Claude tool policy and custom role, and confirm the user's Payerset entitlements.                                               |
| A query is denied              | Confirm the requested data and workflow are included in the user's Payerset entitlement and usage envelope.                                                                  |
| A query times out              | Narrow payer, provider, billing code, plan, geography, setting, or vintage. Record the request ID and timestamp if the bounded request still approaches five minutes.        |
| A result is truncated          | Request the next page, narrow the query, or retrieve specific payers/providers/codes separately.                                                                             |
| OAuth settings changed         | Remove and re-add the custom connector. Claude does not edit an installed connector's OAuth credentials in place, and users must reconnect.                                  |

When contacting support, include the customer organization, affected user, date/time/time zone, Claude surface, attempted workflow, request ID, displayed error, and whether one or multiple users are affected. Never include secrets or raw tokens.

### Disconnect an individual user

1. Open **Customize -> Connectors**.
2. Find Payerset.
3. Open the connector menu.
4. Choose **Disconnect** or **Remove**.
5. Revoke the Payerset authorization when immediate service-side cutoff is required.

Disabling Payerset for one conversation does not disconnect the account; it only prevents Claude from using the connector in that conversation.

### Remove Payerset from the organization

1. Open **Organization settings -> Connectors**.
2. Select Payerset.
3. Block or remove the connector.
4. Revoke user tokens and disable the customer OAuth client through the Payerset offboarding process.
5. Confirm that former users can no longer invoke Payerset.
6. Retain the deployment and audit records required by your organization.

Removing the Claude configuration stops organization-level availability. Revoking Payerset authorization provides the authoritative service-side cutoff.

### Get support

For Payerset authentication, entitlements, rate data, query execution, usage, or billing, contact your Payerset account team or [info@payerset.com](mailto:info@payerset.com).

For Claude account, plan, interface, or organization-administration issues, use the [Claude Help Center](https://support.claude.com/).

### Anthropic references

* [Add a custom remote MCP connector](https://claude.com/docs/connectors/custom/remote-mcp)
* [Authentication for connectors](https://claude.com/docs/connectors/building/authentication)
* [Use connectors on Claude Team and Enterprise](https://support.claude.com/en/articles/11176164-use-connectors-to-extend-claude-s-capabilities)
* [Connect MCP servers to Claude Code](https://code.claude.com/docs/en/mcp)
* [Commercial data use and model training](https://privacy.claude.com/en/articles/7996885-how-do-you-use-personal-data-in-model-training)

***

Payerset is an independent service provided by Payerset Inc. Claude and Anthropic are trademarks of Anthropic. Separate Payerset and Claude subscriptions are required.
