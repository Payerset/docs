# ChatGPT - OpenAI

## Connect Payerset to ChatGPT and the OpenAI API

**Last reviewed:** August 30, 2026

Bring the price-transparency data and analytical capabilities included in your Payerset subscription into ChatGPT Enterprise or an application built with the OpenAI Responses API.

> **Every payer, provider, and negotiated rate included in your licensed Payerset data - available at source-record detail through OpenAI's MCP-capable tools.**

Payerset connects through Model Context Protocol (MCP). Each user signs in to Payerset, and Payerset applies your organization's data rights, analytical access, and usage terms to every request. Payerset does not substitute synthetic or composite rates for licensed source-detail records.

### Choose a deployment route

| Route                     | Best for                                    | Administration                                                         |
| ------------------------- | ------------------------------------------- | ---------------------------------------------------------------------- |
| Direct ChatGPT connection | Pilot users and individually managed access | Each user adds the MCP connection in ChatGPT developer mode            |
| Private workspace plugin  | Managed ChatGPT Enterprise rollout          | Workspace admin imports a private GitHub marketplace and assigns roles |
| OpenAI Responses API      | Customer-built applications and agents      | Application manages OpenAI calls and per-user Payerset OAuth tokens    |

The hosted Payerset MCP endpoint is:

```
https://mcp.tpi.payerset.com/mcp
```

### What your organization receives after purchase

Your Payerset implementation team provides:

* A provisioned Payerset organization with your licensed datasets and features.
* Authorized users and customer-specific entitlements.
* The approved ChatGPT/OpenAI tool profile.
* OAuth discovery and, where required, customer-specific OAuth client details.
* A versioned private-plugin package or repository handoff for managed ChatGPT Enterprise rollout.
* Pilot prompts, usage limits, and support contacts.

If a client ID, client secret, or other sensitive value is required, Payerset delivers it through an approved secure channel. Never store secrets in Markdown, plugin repositories, browser bookmarks, source code, tickets, or AI conversations.

### How the connection works

```
Authorized user or customer application
  -> ChatGPT or the OpenAI Responses API
  -> OpenAI's MCP tool runtime
  -> Payerset MCP over HTTPS
  -> Payerset authentication and entitlement checks
  -> Licensed Payerset records and analytical services
  -> Authorized Payerset result returned to the AI workflow
```

Payerset serves tools over public HTTPS using Streamable HTTP MCP. The model can select an available tool, while OpenAI's service performs the remote MCP call. Payerset verifies identity, customer organization, entitlements, and usage policy before executing the request.

For ChatGPT Enterprise, Payerset is a non-synced app connection: retrieved app data is processed without being indexed, while resulting conversations remain subject to workspace retention and Compliance API controls. For API deployments, OpenAI project data controls and the application's own storage govern the workflow. Payerset separately processes MCP requests and results under the Payerset service.

### Prerequisites

#### ChatGPT

* A ChatGPT workspace and account with developer mode available.
* For managed rollout, a ChatGPT Enterprise workspace administrator.
* For private marketplace distribution, the workspace administrator's GitHub account must be able to read the marketplace repository and every repository it references. Ordinary users do not need GitHub access.
* Workspace approval for Payerset and the required app actions.
* An active Payerset subscription and provisioned users.
* Security approval for the ChatGPT-to-Payerset data flow.

#### OpenAI API

* An OpenAI API project and server-side application.
* A secure process for obtaining and refreshing each user's Payerset OAuth access token.
* A secret manager for OpenAI API keys and Payerset OAuth credentials.
* Application-level tool allowlists, confirmation policy, logging, and error handling.

#### Payerset connection readiness

Payerset handles the MCP and OAuth service requirements, including protected-resource discovery, authorization-server discovery, Authorization Code flow with PKCE, resource/audience validation, and access-token validation.

For a ChatGPT connection, register the exact redirect URI displayed on the app-management page. Depending on the connection flow, OpenAI can present the stable `https://chatgpt.com/connector_platform_oauth_redirect` URI or a connection-specific callback. Use the value ChatGPT actually displays.

For a Responses API application, your application completes Payerset OAuth separately and uses its own registered callback URI; OpenAI does not supply the application's OAuth callback. Do not reuse a callback from another route.

### Route A: Direct connection in ChatGPT

This is the fastest route for a controlled pilot.

#### Step 1: Enable developer mode

Each pilot user:

1. Opens **Settings** in ChatGPT.
2. Selects **Security and login**.
3. Turns on **Developer mode**.

Developer mode availability can depend on account and workspace policy. A workspace administrator may need to enable the feature.

#### Step 2: Add Payerset

1. Open **ChatGPT Plugins** from your workspace.
2. Select the plus button.
3. Enter:
   * **Name:** `Payerset`
   * **Description:** `Detailed healthcare price-transparency data and analytics from Payerset.`
4. Under **Connection**, choose the public endpoint option.
5.  Enter the complete MCP URL, including `/mcp`:

    `https://mcp.tpi.payerset.com/mcp`
6. Create the connection.
7. Review the discovered Payerset tools and metadata.
8. Review the saved connection.

#### Step 3: Use the connection

1. Start a new ChatGPT conversation.
2. Open the tools menu.
3. Add or enable Payerset.
4. Ask a validation question from the examples below.
5. When ChatGPT prompts during first use, complete Payerset OAuth with the user's licensed Payerset identity.

Different users may see different capabilities or results because Payerset enforces each user's organization and entitlements.

### Route B: Managed ChatGPT Enterprise rollout

Use a private workspace plugin when administrators want consistent naming, workflow instructions, role-based availability, and version control.

#### Deployment architecture

The managed package references an existing Payerset app connection in the customer workspace. It does not place a Payerset token or secret in GitHub.

```
.agents/plugins/marketplace.json
plugins/payerset/.codex-plugin/plugin.json
plugins/payerset/.app.json
plugins/payerset/skills/                 # optional workflow guidance
```

Payerset supplies or coordinates the versioned package during onboarding. The customer may host it in an approved private GitHub repository.

#### Step 1: Register the Payerset app connection

1. Connect the Payerset MCP endpoint in the target workspace using the direct-connection steps above.
2. Open the resulting Payerset connection in **ChatGPT Plugins**.
3. Copy the technical identifier from the browser URL. It begins `plugin_asdk_app_...`.
4. Remove only the leading `plugin_` prefix. The `.app.json` file uses the resulting `asdk_app_...` value.
5. Provide that app ID to the administrator or Payerset implementation contact preparing the private package.

The package's `.app.json` references this existing app ID:

```json
{
  "apps": {
    "payerset": {
      "id": "asdk_app_CUSTOMER_VALUE",
      "required": true
    }
  }
}
```

No client secret or Payerset access token belongs in this file.

#### Step 2: Import the private marketplace

A ChatGPT Enterprise workspace administrator:

1. Opens **Admin -> Plugins**.
2. Selects **Add -> Import marketplace**.
3. Enters the approved GitHub repository URL.
4. If the marketplace is in a subdirectory, enters the directory in **Path**. Do not enter the manifest filename.
5. Selects a branch, tag, or fixed commit.
6. Selects **Import marketplace** and authorizes GitHub access when prompted.
7. Reviews the import results.
8. Opens the imported Payerset plugin.
9. Configures its **Installation policy** as **Available** or **Installed** for each eligible role.
10. Enables the required Payerset app and approved actions.
11. Has every pilot user authorize their own Payerset account.

For a change-controlled production deployment, select a fixed commit. A branch follows future commits, and new marketplaces have automatic daily synchronization enabled by default.

> Do not declare the remote Payerset server through `.mcp.json` in the imported plugin. OpenAI currently treats GitHub-imported plugins with that form as desktop-only. The managed plugin should reference the workspace app through `.app.json`.

#### Step 3: Pilot and roll out

1. Assign the plugin only to the pilot role.
2. Run the validation plan below.
3. Confirm authorization, data fidelity, tool behavior, latency, and usage attribution.
4. Record the marketplace source, fixed commit or branch, plugin version, app ID, and acceptance date.
5. Expand installation policy to the contracted production roles.

### Route C: Use Payerset from the OpenAI Responses API

Customer-built applications can attach Payerset as a remote MCP tool in a Responses API request.

Your application is responsible for completing Payerset OAuth, securely storing or refreshing the user's token, enforcing its own user session, and sending the current access token with each OpenAI request. OpenAI documents that the `authorization` value is not stored in the Response object, so the application must send it on every request.

Example in JavaScript:

```javascript
import OpenAI from "openai";

const client = new OpenAI();

const response = await client.responses.create({
  model: "gpt-5.6",
  store: false,
  input: "Using Payerset, find the detailed rate records requested by the user.",
  tools: [
    {
      type: "mcp",
      server_label: "payerset",
      server_description:
        "Licensed healthcare price-transparency data and analytics from Payerset.",
      server_url: "https://mcp.tpi.payerset.com/mcp",
      authorization: process.env.PAYERSET_USER_ACCESS_TOKEN,
      allowed_tools: ["lookup_rates"],
      require_approval: "never"
    }
  ]
});

console.log(response.output_text);
```

Implementation requirements:

* Use a short-lived Payerset token belonging to the effective user.
* Never expose the OpenAI API key or Payerset token to browser code or logs.
* Use `allowed_tools` to limit the Payerset tools imported for this request. Continue enforcing identity, scopes, entitlements, and usage limits on the Payerset server; `allowed_tools` is not a server-side authorization boundary.
* Set approval behavior deliberately. Only bypass approval for tools your security review has confirmed are read-only and appropriate for automatic execution.
* Preserve Payerset request IDs, source/vintage context, and authorization errors.
* Review the current OpenAI Responses API storage setting and use `store: false` when required by your data-governance policy. This prevents the generated Response from being stored for later API retrieval, but it does not by itself provide Zero Data Retention or disable default abuse-monitoring retention. Configure approved Zero Data Retention or Modified Abuse Monitoring separately when required.
* Attribute Payerset usage to the correct customer and user.

### Validate the deployment

Use data your pilot users already know and are licensed to access.

#### Suggested prompts

> Using Payerset, retrieve source-detail negotiated-rate records for a payer, provider NPI, billing code, and plan included in our subscription. Include the network, service setting, rate type, source date, and data vintage available in the result.

> Using Payerset, compare authorized source-detail records across several plans or networks. Preserve the individual published rates and label any calculation as analysis.

> Using Payerset, identify the plans and networks represented for this provider and billing code, and explain the source context.

> Using Payerset, run the analytical workflow available under our organization's subscription and identify the data, filters, and assumptions used.

#### Acceptance checklist

* [ ] The intended Payerset connection or private plugin is visible to the pilot role.
* [ ] Each user completes Payerset OAuth with their own identity.
* [ ] The visible tool catalog matches the approved Payerset profile.
* [ ] A detailed, entitled rate query succeeds with source and vintage context.
* [ ] Source rates remain distinct from computed comparisons or summaries.
* [ ] Two users with different entitlements receive appropriately different access.
* [ ] An unentitled request is denied rather than estimated or fabricated.
* [ ] Usage and model-assisted analysis are attributed to the correct customer and user.
* [ ] Token expiration, refresh, disconnect, and revocation behave as expected.
* [ ] The managed plugin behaves consistently across the ChatGPT surfaces your organization intends to support.

### Security and data handling

#### Per-user authorization

Payerset OAuth identifies the effective user. Payerset applies that user's organization, dataset, payer, provider, geography, feature, and usage entitlements to every request.

ChatGPT role and action controls can narrow availability. They cannot grant access that Payerset has not authorized.

#### Data processing

When Payerset is invoked:

1. ChatGPT or the customer application processes the user's request.
2. The required tool arguments are sent to Payerset.
3. Payerset authenticates and authorizes the call.
4. Payerset executes the permitted query or analysis.
5. The selected result returns to the OpenAI workflow.

For ChatGPT Enterprise, Payerset is a non-synced app connection: retrieved app data is processed without being indexed, while conversations containing Payerset results remain subject to workspace retention and Compliance API controls. For API deployments, apply your OpenAI project data controls and your application's storage, access, logging, and deletion policies. Payerset separately processes MCP requests and results to provide the Payerset service.

#### Secrets

Never place Payerset passwords, OAuth secrets, access tokens, refresh tokens, OpenAI API keys, or raw authorization headers in:

* ChatGPT conversations
* Plugin repositories
* Browser-side application code
* Email or shared documents
* Screenshots or ordinary support tickets

#### Usage and cost

Payerset usage is governed by your Payerset agreement. OpenAI usage is governed by your OpenAI plan or API project.

One user request can generate several tool calls. Monitor complete workflows, not only prompt counts. Use role-based availability, tool allowlists, per-user authorization, and the limits established during Payerset onboarding.

### Troubleshooting

| Issue                                      | What to check                                                                                                                              |
| ------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------ |
| Developer mode is unavailable              | Confirm the user's plan and workspace policy. Ask the ChatGPT administrator to review developer-mode access.                               |
| ChatGPT cannot connect                     | Confirm the exact HTTPS endpoint including `/mcp`, then verify Payerset provisioning and network reachability.                             |
| OAuth loops or fails                       | Confirm the active Payerset account, correct customer organization, approved scopes, and exact callback registered for the OpenAI surface. |
| Payerset connects but tools are missing    | Refresh the connection metadata. Confirm workspace action policy and the user's Payerset entitlements.                                     |
| Private marketplace import fails           | Confirm GitHub access, repository URL, optional path, marketplace filename, and plugin manifests. Review the import report.                |
| Imported plugin is desktop-only            | Remove any `.mcp.json` server declaration and reference the existing Payerset app through `.app.json`.                                     |
| User receives access denied                | Confirm the user is licensed for the requested data and workflow. Payerset will not replace an unauthorized request with an estimate.      |
| Result is too large or slow                | Narrow payer, provider, billing code, plan/network, geography, setting, or vintage.                                                        |
| API request returns an authorization error | Confirm the application sends a current Payerset user token in every request and validates its own user session.                           |
| Recent tool metadata is missing            | Refresh the direct connection, or update and resynchronize the managed plugin according to change policy.                                  |

When contacting support, include the customer organization, ChatGPT workspace or OpenAI project, event time and time zone, affected user, deployment route, attempted workflow, and request/correlation ID. Never include secrets or raw tokens.

### Disconnect or remove Payerset

#### Individual ChatGPT connection

1. Open the Payerset connection in ChatGPT.
2. Select **Disconnect** or remove the connection.
3. Revoke the user's Payerset OAuth grant when immediate service-side cutoff is required.

#### Managed workspace plugin

1. Set the Payerset plugin to unavailable for affected roles.
2. Disable the required Payerset app/actions.
3. Revoke affected Payerset grants or disable the customer OAuth client when required.
4. Remove the marketplace only when all plugins imported from it are intended to be deleted.

#### API application

1. Remove Payerset from the application's tool configuration.
2. Revoke stored Payerset refresh tokens and user grants.
3. Disable or rotate the application-specific OAuth client.
4. Verify that the retired application cannot execute Payerset tools.

### Get support

For Payerset provisioning, entitlements, authentication, tool availability, data coverage, usage, or results, contact your Payerset account team or [info@payerset.com](mailto:info@payerset.com).

For ChatGPT workspace policy or plugin administration, contact your ChatGPT administrator. For OpenAI platform issues, use OpenAI Support.

### OpenAI references

* [Connect and test an MCP server in ChatGPT](https://developers.openai.com/plugins/deploy/connect-chatgpt)
* [Package a ChatGPT plugin](https://developers.openai.com/plugins/build/plugins)
* [Manage private workspace plugins](https://learn.chatgpt.com/docs/enterprise/plugin-management)
* [Authenticate ChatGPT plugins](https://developers.openai.com/plugins/build/auth)
* [Use MCP and connectors with the OpenAI API](https://developers.openai.com/api/docs/guides/tools-connectors-mcp)
* [Manage ChatGPT plugin controls](https://learn.chatgpt.com/docs/enterprise/apps-and-connectors)
* [OpenAI API data controls](https://platform.openai.com/docs/models/default-usage-policies-by-endpoint)

***

Payerset is an independent service provided by Payerset Inc. ChatGPT and OpenAI are trademarks of OpenAI. An active Payerset subscription plus an eligible ChatGPT plan or a separately billed OpenAI API project is required.
