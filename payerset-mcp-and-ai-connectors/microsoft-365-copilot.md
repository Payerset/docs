# Microsoft 365 Copilot

## Connect Payerset to Microsoft 365 Copilot

**Last reviewed:** August 30, 2026

Bring the price-transparency data and analytical capabilities included in your Payerset subscription into the Microsoft 365 Copilot experience your teams already use.

> **Every payer, provider, and negotiated rate included in your licensed Payerset data - available at source-record detail in Microsoft Copilot.**

Payerset retrieves data live through Model Context Protocol (MCP). Each user signs in to Payerset, and Payerset applies your organization's data rights, tool access, and usage terms to every request. Source-detail records remain distinct from any comparison or calculation Copilot performs.

### At a glance

| Item                    | Value                                                                            |
| ----------------------- | -------------------------------------------------------------------------------- |
| Integration type        | Customer-managed custom federated connector                                      |
| Payerset endpoint       | `https://mcp.tpi.payerset.com/mcp`                                               |
| Transport               | HTTPS with Streamable HTTP MCP                                                   |
| Authentication          | Per-user OAuth 2.0                                                               |
| Microsoft administrator | Global Administrator or AI Administrator                                         |
| Initial rollout         | Staged rollout to selected users or groups                                       |
| Data access             | Live access through a provisioned, Microsoft-specific read-only Payerset profile |

Microsoft currently supports federated connectors in several Microsoft 365 Copilot experiences, including Microsoft Copilot Chat, Copilot in Excel, Researcher, and Cowork. Validate every experience you intend to use during your pilot because host availability and behavior can change.

### What your organization receives after purchase

Your Payerset implementation team provides:

* Access to the hosted Payerset MCP endpoint.
* A provisioned Payerset organization with your licensed datasets and features.
* User and entitlement configuration aligned to your Payerset agreement.
* A customer-specific OAuth configuration delivered through an approved secure channel.
* A read-only Microsoft 365 Copilot tool profile.
* Pilot validation guidance and support contacts.
* Contract-based usage metering and limits.

> **Required deployment gate:** Do not create the connector until Payerset confirms that the Microsoft-specific read-only profile is active for your organization. That profile must limit tool discovery to approved read-only operations and reject excluded operations during execution. The unrestricted Payerset MCP catalog is not the profile intended for Microsoft Copilot.

The secure handoff includes:

| Configuration value          | Source                                                                  |
| ---------------------------- | ----------------------------------------------------------------------- |
| OAuth client ID              | Payerset onboarding handoff                                             |
| OAuth client secret          | Secure Payerset handoff only                                            |
| Authorization endpoint       | Payerset onboarding handoff                                             |
| Token endpoint               | Payerset onboarding handoff                                             |
| Refresh endpoint             | Payerset onboarding handoff                                             |
| Required OAuth scopes        | Payerset onboarding handoff                                             |
| PKCE support status          | Payerset onboarding handoff                                             |
| Pilot users and entitlements | Confirmed jointly                                                       |
| Developer name               | `Payerset Inc.`                                                         |
| Product website              | `https://payerset.com`                                                  |
| Product documentation        | `https://docs.payerset.com`                                             |
| Privacy policy               | `https://payerset.com/privacy/`                                         |
| Terms of use                 | Publicly reachable, approved URL supplied by Payerset during onboarding |

Never copy client secrets, access tokens, or recovery information into documentation, source control, tickets, email, or AI conversations.

### How the connection works

```
Authorized user
  -> Microsoft 365 Copilot
  -> Payerset MCP over HTTPS
  -> Payerset authentication and entitlement checks
  -> Licensed Payerset records and analytical services
  -> Authorized Payerset result returned to Copilot
```

1. A user selects Payerset as a source or asks a question that can use a Payerset tool.
2. Microsoft 365 Copilot sends the relevant tool request to the Payerset MCP endpoint.
3. Payerset validates the user's OAuth token, organization, scopes, and entitlements.
4. Payerset executes the authorized read-only operation.
5. The result returns to Microsoft 365 Copilot for presentation to the user.

Payerset data is retrieved at request time and is not indexed into Microsoft 365. Microsoft processes connector inputs and results and stores the user's prompt and Copilot-generated response as Copilot interaction data. Applicable retention, audit, eDiscovery, access, and sharing controls depend on the tenant's Microsoft 365 and Microsoft Purview licensing and configuration. Payerset separately processes authentication data, tool requests, and results to provide the Payerset service.

### Prerequisites

#### Microsoft 365

Your organization needs:

* A Microsoft 365 Copilot add-on license, or Microsoft 365 E7, for every user who queries Payerset.
* A Global Administrator or AI Administrator to configure the connector.
* Access to [Teams Developer Portal](https://dev.teams.microsoft.com/).
* A Microsoft Entra user group for the initial pilot.
* Permission to enable custom federated connectors in the tenant.
* Security approval for the Microsoft 365 Copilot-to-Payerset data flow.

Federated connectors are not supported through Copilot Studio licensing or Microsoft 365 Copilot pay-as-you-go alone.

#### Payerset

Before installation, confirm with your Payerset implementation contact that:

* Your organization and pilot users are provisioned.
* Your licensed payer, provider, geography, module, and feature entitlements are active.
* The Microsoft 365 Copilot read-only tool profile is configured.
* Your contracted usage and metering terms are active.
* The customer-specific OAuth values have been delivered securely.
* A publicly reachable, approved Payerset terms-of-use URL has been supplied for the connector metadata.

### Step 1: Register the OAuth connection

Payerset configures this Microsoft redirect URI for your customer-specific OAuth client:

```
https://teams.microsoft.com/api/platform/v1.0/oAuthRedirect
```

A customer administrator completes the following steps in Teams Developer Portal:

1. Sign in to [Teams Developer Portal](https://dev.teams.microsoft.com/).
2. Select **Tools**.
3. Select **OAuth Client Registration**.
4. Select **New OAuth connection**.
5. Enter a descriptive name such as `Payerset`.
6. Enter the values supplied securely by Payerset:
   * Client ID
   * Client secret
   * Authorization endpoint
   * Token endpoint
   * Refresh endpoint
   * Scopes
7. Enable Proof Key for Code Exchange only when the Payerset onboarding handoff confirms PKCE support.
8. Select **Save**.
9. Copy the generated OAuth registration ID. You will use it when creating the connector.

### Step 2: Create the Payerset connector

A Global Administrator or AI Administrator completes these steps:

1. Sign in to the [Microsoft 365 admin center](https://admin.microsoft.com/).
2. Go to **Copilot -> Connectors**.
3. Select the **Gallery** tab.
4. Under **Created by your org**, find **Create a new connector** and select **Add**.
5. Under **Connect to MCP server**, select **Add**.
6. Enter:
   * **Display name:** `Payerset`
   * **Base URL:** `https://mcp.tpi.payerset.com/mcp`
   * **OAuth registration ID:** the ID generated in Teams Developer Portal
7. Select **Save**.

The connector now appears in **Your Connections**. Microsoft configuration changes can take up to 15 minutes to appear throughout the tenant.

### Step 3: Stage the pilot

Begin with a small group of licensed, authorized users:

1. Open the Payerset connector from **Your Connections**.
2. Select **Staged rollout**.
3. Choose **Users** or **Groups**.
4. Add the Microsoft Entra pilot group.
5. Save the rollout configuration.
6. Wait for the configuration to propagate.
7. Confirm that every pilot user has active Microsoft 365 Copilot and Payerset access.
8. In the Microsoft 365 admin center, verify the connector's allowed-user scope and review **Agents -> Settings -> Allowed agent types** and **User access** for the pilot population.

Keep the pilot limited to users whose Payerset entitlements are already active.

### Step 4: Connect each user

Every user authenticates with their own Payerset identity.

In Microsoft Copilot Chat:

1. Open Microsoft Copilot Chat.
2. Open the ellipsis menu and select **Settings**.
3. Select **Sources**.
4. Find **Payerset**.
5. Select **Connect**.
6. Complete the Payerset sign-in and consent flow.
7. Return to Copilot Chat and begin a new conversation.

Users may also encounter Payerset in source menus within Researcher and other supported Copilot experiences.

Payerset - not the Microsoft 365 group name or the wording of a prompt - determines which data and operations each authenticated user can access.

### Validate the deployment

Test with data your team already knows and is licensed to use.

#### Suggested validation prompts

> Using Payerset, retrieve source-detail negotiated-rate records for a payer, provider NPI, billing code, and plan included in our subscription. Include the network, service setting, rate type, source date, and data vintage available in the result.

> Using Payerset, compare authorized source-detail rates for the same billing code across several plans or networks. Keep the reported source rates distinct from any computed comparison.

> Using Payerset, find the available source context for a negotiated-rate record and explain which payer, plan, provider, billing code, setting, and file vintage it represents.

> Using Payerset, run an analytical workflow included in our subscription and identify the Payerset data and filters used.

If payer-policy or provider-research capabilities are included in your profile, also test:

> Using Payerset, retrieve the applicable payer policy for a covered scenario and include its source and effective-date information.

#### Acceptance checklist

* [ ] An authorized pilot user can connect and run approved tools.
* [ ] A user without the required entitlement receives an access-denied or appropriately limited result.
* [ ] Results include the expected payer, provider, code, plan/network, service setting, source, and vintage context when available.
* [ ] Source-detail rates are not replaced by synthetic or composite values.
* [ ] Requested calculations are presented as analysis rather than as underlying source records.
* [ ] No persistent-write, administrative, or unapproved export tool is available.
* [ ] Usage is attributed to the correct Payerset organization and user.
* [ ] Token expiration, refresh, revocation, and reconnection behave as expected.
* [ ] Intended Microsoft 365 Copilot experiences work for the pilot group.
* [ ] Query latency and result sizes are acceptable for the intended workflows.

Record the pilot group, connector configuration, tool-profile version, test prompts, date, and acceptance owner.

### Security and data handling

#### User-scoped authorization

Every user signs in to Payerset. Payerset validates the token and enforces the organization's data, feature, and usage entitlements on every call.

Microsoft 365 administration controls who can discover and activate the connector. Payerset authorization controls what those users can retrieve.

#### Read-only connector profile

When Payerset has confirmed the provisioned Microsoft-specific profile is active, that profile exposes only approved read-only Payerset operations. It can provide source-detail negotiated rates, payer policies, provider research, and licensed analytical workflows without exposing persistent mutation or administrative tools.

Payerset must enforce the profile during both tool discovery and tool execution. Treat successful verification of both controls as a production deployment gate.

#### Live retrieval

Payerset data is retrieved at request time and is not indexed into Microsoft 365. Microsoft stores the user's prompt and Copilot-generated response as Copilot interaction data; applicable retention, audit, eDiscovery, access, and sharing controls depend on the tenant's Microsoft 365 and Microsoft Purview licensing and configuration. Payerset separately processes authentication data, tool requests, and results to provide the Payerset service.

#### Encryption and secrets

* The MCP endpoint uses HTTPS.
* OAuth access tokens provide user-scoped access.
* The customer-specific client secret is entered only in the approved Microsoft OAuth registration flow.
* Secrets and production tokens must not be placed in documentation, repositories, support tickets, or connector descriptions.

#### Auditability

Maintain both Payerset usage and authorization records for connector calls and the Microsoft audit records available under your Microsoft 365 and Microsoft Purview configuration.

### Usage and cost governance

Payerset usage remains governed by your Payerset agreement.

Recommended controls:

* Begin with a limited pilot group.
* Grant access only to licensed users with a valid business need.
* Review query volume, latency, authorization failures, and model-assisted usage during the pilot.
* Confirm that usage is attributed to the correct organization and user.
* Contact Payerset before materially increasing the user population or expected query volume.
* Review enabled capabilities and user access whenever responsibilities change.

One natural-language question can invoke several tools. Size your pilot and contracted usage envelope around complete user workflows rather than individual tool calls.

### Production rollout

After pilot approval:

1. Record business, security, and technical acceptance.
2. Confirm every production user is covered by a Microsoft 365 Copilot add-on license or Microsoft 365 E7, as well as the applicable Payerset license.
3. Expand connector availability to the approved Microsoft Entra groups.
4. Notify users that they must complete Payerset authentication individually.
5. Provide approved example prompts and usage guidance.
6. Monitor authentication errors, access denials, latency, volume, and unexpected tool selection.
7. Schedule periodic entitlement and access reviews.

### Troubleshooting

| Issue                                                           | What to check                                                                                                                                                                 |
| --------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| The **Create a new connector** tile is missing from Gallery     | Confirm the administrator has the required role, is using the correct tenant, and that custom federated connectors are permitted. Review the tenant's agent settings.         |
| Payerset does not appear in **Your Connections** after creation | Allow up to 15 minutes, refresh the Microsoft 365 admin center, and verify that the connector was saved in the expected tenant.                                               |
| Pilot users cannot see Payerset                                 | Confirm their Copilot licenses, staged-rollout membership, and Payerset provisioning. Allow up to 15 minutes for propagation.                                                 |
| OAuth setup fails                                               | Verify the client ID, client secret, endpoints, scopes, and PKCE selection against the secure Payerset handoff.                                                               |
| Redirect mismatch                                               | Confirm the OAuth client includes exactly `https://teams.microsoft.com/api/platform/v1.0/oAuthRedirect`.                                                                      |
| A user cannot sign in                                           | Confirm the user is active in the correct Payerset organization and is using the expected identity.                                                                           |
| Access denied                                                   | Confirm the user's dataset, payer, geography, feature, and usage entitlements.                                                                                                |
| No records                                                      | Confirm the requested payer, provider, code, plan, geography, setting, and vintage are in the licensed data. A valid empty result is different from an authorization failure. |
| Expected capability is missing                                  | Confirm the capability is included in the Microsoft tool profile and your Payerset agreement.                                                                                 |
| Result is too large or slow                                     | Narrow the payer, provider, code, plan/network, geography, setting, or date filters.                                                                                          |
| A recent change is not visible                                  | Allow up to 15 minutes, refresh the admin and user experiences, and start a new conversation.                                                                                 |

When opening a support request, include the customer organization, Microsoft tenant ID, event time and time zone, affected user, connector name, attempted workflow, and any displayed request or correlation ID. Never include secrets, tokens, or licensed result data unless Payerset provides an approved secure upload method.

### Disable or remove Payerset

#### Temporarily disable access

1. Open **Microsoft 365 admin center -> Copilot -> Connectors -> Your Connections**.
2. Select Payerset.
3. Choose **Disable**, or remove the affected users or groups from the rollout.
4. If an immediate service-side cutoff is required, revoke the affected Payerset tokens or disable the customer OAuth client.
5. Verify that affected users can no longer execute Payerset tools.

Disabling preserves the connector configuration for later reactivation.

#### Permanently remove the connection

1. Disable the connector.
2. Retain the deployment and audit records required by your organization.
3. Revoke active Payerset grants and retire the OAuth client.
4. Delete the connector from **Your Connections**.
5. Verify removal after the Microsoft configuration change has propagated.

### Get support

For Payerset provisioning, entitlements, authentication, tool availability, data coverage, usage, or results, contact your Payerset account team or [info@payerset.com](mailto:info@payerset.com).

For Microsoft 365 licensing, tenant policy, connector visibility, or Copilot behavior, contact your Microsoft 365 administrator or Microsoft Support.

### Microsoft references

* [Set up custom federated connectors](https://learn.microsoft.com/en-us/microsoft-365/copilot/connectors/set-up-custom-federated-connectors)
* [Federated connectors overview](https://learn.microsoft.com/en-us/microsoft-365/copilot/connectors/federated-connectors-overview)
* [Manage federated connector availability](https://learn.microsoft.com/en-us/microsoft-365/copilot/connectors/manage-federated-connectors)
* [Federated connector prerequisites and licensing](https://learn.microsoft.com/en-us/microsoft-365/copilot/connectors/prerequisites)
* [Manage agents in the Microsoft 365 admin center](https://learn.microsoft.com/en-us/microsoft-365/admin/manage/agent-settings?view=o365-worldwide)

***

Payerset is an independent service provided by Payerset Inc. Microsoft 365 and Microsoft Copilot are trademarks of Microsoft Corporation. A separate Payerset subscription and, for every user who queries Payerset, a Microsoft 365 Copilot add-on license or Microsoft 365 E7 are required.
