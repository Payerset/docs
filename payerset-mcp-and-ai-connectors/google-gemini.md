# Google Gemini

## Connect Payerset to Gemini Enterprise

**Last reviewed:** August 30, 2026

Bring detailed healthcare price-transparency intelligence into your organization's Gemini Enterprise experience.

> **Every payer, provider, and negotiated rate included in your licensed Payerset data - available at source-record detail through Gemini Enterprise.**

Payerset connects through a Custom MCP Server data store. Each user signs in to Payerset, and Payerset applies your organization's data rights, analytical access, and usage terms to every request. Payerset does not substitute synthetic or composite rates for licensed source-detail records.

### At a glance

| Item                   | Value                                                                                |
| ---------------------- | ------------------------------------------------------------------------------------ |
| Recommended deployment | Gemini Enterprise Standard or Plus                                                   |
| Integration type       | Custom MCP Server data store                                                         |
| Payerset endpoint      | `https://mcp.tpi.payerset.com/mcp`                                                   |
| Transport              | HTTPS with Streamable HTTP MCP                                                       |
| Authentication         | Per-user OAuth 2.0                                                                   |
| Google Cloud roles     | Discovery Engine Editor; Organization Policy Administrator for initial policy change |
| Enabled actions        | Up to 100 at one time                                                                |

Gemini Enterprise discovers Payerset capabilities through MCP. When Gemini selects a Payerset action, Payerset validates the user, checks contractual entitlements and usage limits, executes the request, and returns an authorized Payerset result.

### What your organization receives after purchase

Your Payerset implementation team provides the following through the approved secure channel:

* OAuth authorization URL.
* OAuth token URL.
* Customer-specific OAuth client ID.
* Customer-specific OAuth client secret.
* Required OAuth scopes and PKCE setting.
* Authorized Payerset users and customer entitlements.
* Licensed tool profile and usage limits.
* Pilot validation cases and support contacts.

Never place the client secret, production tokens, or OAuth grants in tickets, email, shared documents, source control, or AI conversations.

### How the connection works

```
Authorized user
  -> Gemini Enterprise app
  -> Payerset Custom MCP data store
  -> Payerset MCP over HTTPS
  -> Payerset authentication, entitlements, and usage controls
  -> Licensed Payerset records and analytical services
  -> Authorized result returned to Gemini Enterprise
```

1. Gemini Enterprise imports the customer-authorized Payerset actions through `tools/list`.
2. A user authorizes Payerset with their individual account.
3. Gemini selects an enabled Payerset action for a relevant question.
4. Payerset verifies identity, organization, entitlements, and usage policy.
5. The authorized result returns to Gemini Enterprise.

Returned results enter the Gemini Enterprise conversation and are subject to your Google environment's data, retention, access, sharing, and audit controls.

### Prerequisites

#### Payerset

* An active Payerset subscription.
* A provisioned Payerset organization and pilot users.
* Customer-specific data, feature, and usage entitlements.
* The approved Gemini Enterprise tool profile.
* A customer-specific OAuth client and secure credential handoff.

Register this exact redirect URI in the customer-specific Payerset OAuth client:

```
https://vertexaisearch.cloud.google.com/oauth-redirect
```

#### Google Cloud

* A Gemini Enterprise Standard or Plus app and its Google Cloud project.
* An administrator with Discovery Engine Editor (`roles/discoveryengine.editor`) to create the data store.
* An Organization Policy Administrator (`roles/orgpolicy.policyAdmin`) for the initial Custom MCP policy change.
* Approval to allow outbound HTTPS traffic to the Payerset MCP and OAuth hostnames.
* Security approval for the Gemini Enterprise-to-Payerset data flow.
* `custom_mcp` included in allowed data sources when the project enforces that policy or uses applicable VPC Service Controls.

### Step 1: Enable Custom MCP for the Google Cloud project

An Organization Policy Administrator completes this step.

1. Open **Organization Policies** in Google Cloud Console.
2. Select the project containing the Gemini Enterprise app.
3. Search for **Disable custom MCP server connector for Gemini Enterprise**.
4. Open the policy and select **Manage policy**.
5. Select **Override parent's policy**.
6. Add a rule and set enforcement to **Off**.
7. Select **Set Policy**.
8. Confirm that the project-level policy shows **Not enforced**.

Apply the override to the intended project rather than the entire organization unless your organization has deliberately approved broader use.

### Step 2: Allow the Payerset endpoints

Add the hostnames for these services to the permitted egress FQDN configuration:

* `mcp.tpi.payerset.com`
* The Payerset OAuth authorization hostname supplied during onboarding.
* The Payerset OAuth token hostname supplied during onboarding.

Enter hostnames only, not complete URLs.

If the project is protected by VPC Service Controls or an enforced permitted-data-source policy, add `custom_mcp` to the project's allowed data sources.

### Step 3: Create the Payerset data store

An administrator with `roles/discoveryengine.editor` completes this step.

1. Open **Gemini Enterprise** in Google Cloud Console.
2. Select **Data stores**.
3. Select **Create data store**.
4. Find **Custom MCP Server**.
5. Select **Add MCP server**.
6. Under authentication settings, select **OAuth 2.0**.
7. Enter the connection values:

| Field                        | Value                                                    |
| ---------------------------- | -------------------------------------------------------- |
| MCP Server URL               | `https://mcp.tpi.payerset.com/mcp`                       |
| Authorization URL            | Supplied by Payerset                                     |
| Authorization URL Parameters | Enter values supplied by Payerset; otherwise leave blank |
| Token URL                    | Supplied by Payerset                                     |
| Client ID                    | Customer-specific Payerset client ID                     |
| Client Secret                | Transferred through the secure handoff                   |
| Scopes                       | Space-separated scopes supplied by Payerset              |
| Enable PKCE Support          | Enable when specified by Payerset                        |

8. Select **Verify Auth** and complete the Payerset sign-in.
9. Select **Continue**.
10. Choose the data-store multi-region approved by your organization.
11. Enter `Payerset` as the connector name.
12. Select **Create**.
13. Wait until the data-store state changes from **Creating** to **Active**.

Google supports Streamable HTTP for this connector type. Legacy SSE is not supported.

### Step 4: Load and enable Payerset actions

New Custom MCP data stores have no actions enabled by default.

1. Open the active Payerset data store.
2. Select **Actions**.
3. Select **Reload custom actions** and reauthenticate if prompted.
4. Review the tools retrieved from Payerset.
5. Select the Payerset actions approved for your organization.
6. Select **Enable actions**.

Gemini Enterprise can enable up to 100 actions at one time for a Custom MCP data store.

Gemini Enterprise bypasses confirmation for actions whose live MCP definition truthfully includes `readOnlyHint: true`; other actions require confirmation by default.

After Payerset adds or changes tools, return to this page and select **Reload custom actions** to refresh the catalog.

### Step 5: Connect Payerset to the Gemini Enterprise app

1. In Gemini Enterprise, select **Apps**.
2. Open the app that will provide Payerset access.
3. Select **Connected data sources**.
4. Select **Add existing data stores**.
5. Select the Payerset data store.
6. Select **Connect**.
7. Make the app available to the approved pilot users according to your normal Gemini Enterprise access policy.

### Step 6: Authorize each user

Every user authorizes Payerset with an individual account.

1. Open the Gemini Enterprise web app.
2. Select **Manage your data**.
3. Find Payerset and select **Enable actions**.
4. Sign in to Payerset.
5. Complete any required multifactor or conditional-access checks.
6. Approve the requested Payerset scopes.

Do not use a shared user account or shared bearer token. Individual authorization allows Payerset to enforce user-level access, metering, revocation, and auditability.

### Validate the deployment

Use customer-authorized payers, providers, codes, markets, and data vintages.

#### Suggested prompts

> Find the detailed negotiated-rate records for a payer, NPI, and billing code included in our Payerset subscription. Include the plan, billing class, service setting, data vintage, and available source context.

> Compare the source-detail records for the same billing code between two providers for an authorized payer. Preserve the individual published records rather than replacing them with a synthetic or composite rate.

> Run the authorized Payerset analytical workflow for our use case and identify the source data, filters, assumptions, and data vintage used.

#### Acceptance checklist

* [ ] The Payerset data store is **Active**.
* [ ] The intended Payerset actions are enabled.
* [ ] A pilot user completes Payerset OAuth.
* [ ] An unprovisioned user cannot execute Payerset actions.
* [ ] Payerset applies the correct user and organization entitlements.
* [ ] A detailed, entitled rate query includes expected source and vintage context.
* [ ] Source records remain distinct from computed analysis.
* [ ] Two users with different Payerset entitlements receive appropriately different access.
* [ ] Usage and model-assisted analysis are attributed to the correct customer.
* [ ] Token refresh, reauthorization, revocation, and removal work as expected.

Also test a request outside the user's licensed payer/market, a request at the configured result-size boundary, and any model-assisted workflow subject to a contracted usage allowance.

### Security and data flow

#### Authentication and authorization

* Each user authenticates through Payerset OAuth.
* Payerset validates user, organization, data, feature, and usage entitlements on every call.
* Revoking a Payerset user or OAuth grant prevents future authorized calls.
* A customer-specific OAuth client allows the integration to be rotated or disabled independently.

#### Transport and network controls

* The MCP connection uses HTTPS and Streamable HTTP.
* The Payerset MCP and OAuth hostnames must remain reachable from Gemini Enterprise.
* Private Service Connect is not supported for Custom MCP Server data stores.
* Custom MCP traffic does not pass through Gemini Enterprise Agent Gateway, so Agent Gateway policies do not govern this connection.
* Your organization can evaluate Google Model Armor separately for prompt and response screening.

#### Tool and commercial controls

* Customer administrators choose which imported Payerset actions to enable.
* Payerset enforces the licensed tool profile again at execution time.
* Read-only and state-changing actions are annotated separately.
* Payerset applies contractual query, concurrency, result-size, export, and model-assisted-analysis controls.

### Platform limits

| Limit                   | Current behavior                                                            |
| ----------------------- | --------------------------------------------------------------------------- |
| MCP transport           | Streamable HTTP only                                                        |
| Legacy SSE              | Not supported                                                               |
| Enabled actions         | Up to 100 at one time                                                       |
| Default action state    | Disabled                                                                    |
| Private Service Connect | Not supported                                                               |
| Authentication          | OAuth 2.0 with customer-specific client credentials                         |
| Read-only confirmation  | Bypassed only when the live action truthfully includes `readOnlyHint: true` |
| Other actions           | User confirmation required by default                                       |

Platform behavior can change. Recheck Google's current Custom MCP documentation before a new production deployment.

### Usage and cost governance

Payerset usage remains governed by your Payerset agreement.

* Begin with a defined pilot group.
* Enable only licensed Payerset actions required for approved workflows.
* Review query volume, latency, authorization failures, and model-assisted usage.
* Confirm attribution to the correct customer and user.
* Contact Payerset before materially increasing the user population or workload.
* Review enabled actions and user entitlements periodically.

One Gemini question can invoke several Payerset actions. Evaluate complete workflows rather than prompt counts alone.

### Troubleshooting

| Symptom                            | What to check                                                                                                                   |
| ---------------------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| Custom MCP Server does not appear  | Confirm the project, policy override, and `roles/discoveryengine.editor`.                                                       |
| Data-store creation is blocked     | Check the policy override, egress FQDNs, and `custom_mcp` allowed-data-source setting where applicable.                         |
| Verify Auth fails                  | Confirm authorization/token URLs, client values, scopes, PKCE, network reachability, and the exact redirect URI.                |
| Data store does not become Active  | Confirm the exact endpoint, Streamable HTTP support, valid TLS certificate, and reachability.                                   |
| No Payerset actions appear         | Select **Reload custom actions**, reauthenticate, and confirm the user's Payerset access.                                       |
| A recently changed tool is missing | Reload custom actions so Gemini retrieves the current catalog.                                                                  |
| Gemini does not select Payerset    | Confirm the action is enabled, the user enabled Payerset actions, and OAuth completed. Retry with an explicit Payerset request. |
| User receives 401 or 403           | Confirm Payerset provisioning, OAuth grant, and entitlement to the requested data/tool.                                         |
| Unexpected confirmation appears    | Review the action annotations. Only genuinely read-only actions should carry `readOnlyHint: true`.                              |
| Request exceeds a limit            | Review contracted query, result-size, concurrency, export, and model-assisted-analysis allowances.                              |

When contacting support, include the customer organization, Google Cloud project ID, Gemini app and data-store names, affected user, time/time zone, tool, error, and Payerset request ID if shown. Never include secrets, tokens, or sensitive returned data.

### Disconnect or remove Payerset

#### Temporary disablement

1. Disconnect Payerset from the Gemini Enterprise app or remove affected group access.
2. Disable the Payerset actions in the data store.
3. Revoke affected user grants in Payerset when required.
4. Confirm that disabled users can no longer execute Payerset actions.

#### Permanent removal

1. Disconnect the Payerset data store from every Gemini Enterprise app.
2. Revoke Payerset OAuth grants.
3. Disable or rotate the customer-specific OAuth client.
4. Delete the Payerset data store.
5. Confirm that the former client and users can no longer call Payerset.

### Alternate route: Gemini Enterprise Business edition

Gemini Enterprise Business edition supports Custom MCP connections under Google's current Public Preview terms.

Requirements include a team administrator, Streamable HTTP, a publicly accessible HTTPS MCP endpoint and identity provider, and customer-specific Payerset OAuth credentials. Workspace-purchased Business edition also requires a Services Admin or administrator with the Gemini Enterprise management privilege.

To connect:

1. Sign in at `business.gemini.google`.
2. Open **Settings & help**.
3. Select the customer's team.
4. Open **Manage team -> Connected apps**.
5. Select **Add MCP Server**.
6. Enter the Payerset endpoint, name, description, authorization URL, token URL, optional authorization parameters, client ID, secret, and scopes.
7. Select **Add**.
8. Enable the connection; new custom connections are disabled by default.
9. Have every authorized user connect with their individual Payerset account.

Google's Business edition article does not publish a universal OAuth callback. Your Payerset implementation contact will coordinate callback validation before enabling the production OAuth client.

### Optional: Gemini CLI

Approved analysts and developers can also connect Gemini CLI directly to Payerset. Use a CLI-specific OAuth client and the settings structure documented by Google.

```json
{
  "mcpServers": {
    "payerset": {
      "httpUrl": "https://mcp.tpi.payerset.com/mcp",
      "authProviderType": "dynamic_discovery",
      "oauth": {
        "enabled": true,
        "clientId": "${PAYERSET_GEMINI_CLIENT_ID}",
        "scopes": ["SCOPE_SUPPLIED_BY_PAYERSET"]
      },
      "timeout": 600000,
      "trust": false
    }
  },
  "mcp": {
    "allowed": ["payerset"]
  }
}
```

The interactive OAuth flow normally returns to `http://localhost:7777/oauth/callback`. Register that callback for the CLI-specific client. The flow requires a local browser and localhost callback; Google notes that it does not work in headless environments, remote SSH without X11 forwarding, or containers without browser support. Keep `trust` set to `false` unless your organization deliberately approves bypassing all tool-call confirmation prompts, and use `includeTools` when a narrower catalog is required.

### Get support

For Payerset provisioning, entitlements, authentication, tool availability, data coverage, usage, or results, contact your Payerset account team or [info@payerset.com](mailto:info@payerset.com).

For Google Cloud policy, licensing, or Gemini Enterprise interface issues, contact your Google Cloud administrator or Google Cloud Support.

### Google references

* [Set up a Custom MCP Server data store](https://docs.cloud.google.com/gemini/enterprise/docs/connectors/custom-mcp-server/set-up-custom-mcp-server)
* [Override the Custom MCP organization policy](https://docs.cloud.google.com/gemini/enterprise/docs/connectors/custom-mcp-server/override-constraint-for-custom-mcp-data-stores)
* [Configure allowed egress FQDNs](https://docs.cloud.google.com/gemini/enterprise/docs/connectors/configure-allowed-egress-fqdns)
* [Configure allowed data sources](https://docs.cloud.google.com/gemini/enterprise/docs/connectors/configure-allowed-data-sources)
* [Connect a data store to a Gemini Enterprise app](https://docs.cloud.google.com/gemini/enterprise/docs/connectors/connect-existing-data-store)
* [Gemini Enterprise Business edition Custom MCP setup](https://support.google.com/g/answer/17106276?hl=en)
* [Gemini Enterprise Business edition release notes](https://support.google.com/g/answer/16719009?hl=en)
* [Gemini CLI MCP configuration](https://google-gemini.github.io/gemini-cli/docs/tools/mcp-server.html)

***

Payerset is an independent service provided by Payerset Inc. Gemini and Google are trademarks of Google LLC. Separate Payerset and Google subscriptions are required.
