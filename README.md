# Enterprise App Provisioning Lab

## Overview

This lab sets up automatic user provisioning for an enterprise application in Microsoft Entra ID, scoped to specific users, using nothing but admin console configuration. No custom code involved. The SCIM Provisioning Lab built the receiving side of a SCIM integration from scratch; this one works from the other direction, using a connector Microsoft already built to automatically create, update, and deactivate accounts in a real target application.

I also built this to settle something that kept tripping me up on SC-300 practice exams: the difference between provisioning, Conditional Access, and self-service access. Provisioning is about whether an account exists in an application at all. Conditional Access governs the conditions under which someone who already has access can sign in, things like requiring MFA or a compliant device. Self-service, through Entra's Entitlement Management, is about how someone gets assigned access in the first place, by requesting it and getting approved instead of an admin dropping them into a group directly. This lab is entirely about the first one.

## Environment

- Identity Provider: Microsoft Entra ID (tenant: fernandotech187.onmicrosoft.com)
- Target application: ServiceNow, Personal Developer Instance (Australia release)
- Provisioning protocol: Automatic provisioning via Entra's built-in ServiceNow connector, Basic Authentication
- Also attempted: Salesforce Developer Edition, documented below as a real platform-level dead end rather than a working path

## What I Built

### Adding an Enterprise Application from the Gallery

I added Salesforce as an Enterprise Application in Entra ID straight from the application gallery. This is a different object than an App Registration. An App Registration is a blueprint for an app you build yourself; an Enterprise Application from the gallery is a connector Microsoft already built for a real SaaS product, so turning on provisioning here means configuring an existing integration rather than writing one.

`[SCREENSHOT: Salesforce enterprise application added from the gallery in Entra ID]`

### Salesforce Automatic Provisioning: A Platform-Level Dead End

I tried configuring automatic provisioning against a free Salesforce Developer Edition org (reused from the Okta SSO and MFA Lab). Entra's Salesforce connector authenticates with a username, password, and security token. Every attempt failed with a generic `CredentialValidationUnavailable` error.

Digging into it, it turns out Salesforce has been retiring this exact login method (the OAuth 2.0 Username-Password Flow) platform-wide since 2023 for security reasons. The toggle to allow it under OAuth and OpenID Connect Settings was greyed out entirely, meaning it can't be re-enabled by an admin anymore, not through a setting, not through a support ticket. The only real fix on Salesforce's side would be building a custom OAuth-based integration by hand, which is exactly the custom-code territory this lab was meant to avoid. I documented this as a genuine, permanent platform limitation rather than a misconfiguration, and moved the lab to a different target application instead of working around it with code.

`[SCREENSHOT: Salesforce OAuth and OpenID Connect Settings showing the Username-Password Flow toggle greyed out]`

`[SCREENSHOT: CredentialValidationUnavailable error in the Entra provisioning logs]`

### Pivoting to ServiceNow

I spun up a free ServiceNow Personal Developer Instance (PDI) on the Australia release and added ServiceNow as an Enterprise Application in Entra from the gallery. ServiceNow's connector uses Basic Authentication with a username and password rather than Salesforce's retired flow, so it wasn't caught by the same platform-level lockdown.

`[SCREENSHOT: ServiceNow enterprise application added in Entra, provisioning configuration screen]`

### Fixing a ServiceNow Basic Authentication Restriction

The first connection test still failed, this time with an `InvalidCredentials` error, despite the credentials being correct. ServiceNow has recently restricted Basic Authentication API access for accounts used for both interactive login and API integrations, a similar security hardening move to Salesforce's, but implemented as a role-based exception instead of a full retirement. Granting the admin account the `snc_basic_auth_api_access` role fixed it, and the connection test passed right after. It's a good contrast to the Salesforce issue: one platform closed the door permanently, the other just gated it behind a specific role.

`[SCREENSHOT: snc_basic_auth_api_access role assigned to the admin account in ServiceNow]`

`[SCREENSHOT: successful connection test in Entra provisioning settings]`

### Scoping Provisioning to Specific Users

I created a security group meant to scope provisioning to a small set of test users, but group-based assignment to an Enterprise Application requires Microsoft Entra ID P1 or P2 licensing. This tenant's premium trials (Entra ID P2, Intune, Business Premium) had all already expired from earlier labs and can't be restarted once a trial ends. Rather than rebuild the whole lab on a new tenant just to chase a one-time trial, I documented this as a licensing tier limitation and assigned the two test users to the application individually instead of through the group. Provisioning is still fully scoped, just to specific users rather than a group object, which are really two variations of the same underlying assignment-based scoping model.

`[SCREENSHOT: two test users assigned individually to the ServiceNow enterprise application]`

### Verifying Automatic Provisioning End to End

I started provisioning and confirmed the initial sync completed successfully, 100 percent, 2 users processed. Then I checked ServiceNow's user list directly, both test accounts existed with matching usernames and creation timestamps, created automatically with no manual account creation involved.

`[SCREENSHOT: Entra provisioning logs showing successful initial sync, 2 users processed]`

`[SCREENSHOT: both test accounts visible in ServiceNow's user list]`

### Demonstrating Deprovisioning

I removed one of the two test users from the application's assignment in Entra, then ran Provision on demand for that user instead of waiting for the next scheduled sync cycle. The result showed a successful attribute change, the account's active status flipped from true to false. I confirmed the same change directly in ServiceNow's user list, the account's Active field now read false with an updated timestamp matching the deprovisioning action, while the second test user stayed untouched and active. This shows the full Joiner and Leaver lifecycle: accounts get created automatically on assignment and deactivated automatically on removal, without deleting the account or its history.

`[SCREENSHOT: Provision on demand result showing the active attribute change to false]`

`[SCREENSHOT: ServiceNow user list showing one account deactivated, the other still active]`

## Key Concepts Demonstrated

- The distinction between provisioning (does an account exist), Conditional Access (under what conditions can someone sign in), and self-service access (how someone gets assigned in the first place)
- Enterprise Application (a pre-built gallery connector) versus App Registration (a blueprint for a custom-built app)
- Automatic user provisioning configured entirely through admin console settings, no custom code
- Assignment-based provisioning scope, accounts are only created for users or groups explicitly assigned to the application, never tenant-wide
- Recognizing the difference between a genuine platform-level dead end (Salesforce's retired OAuth flow) and a fixable configuration gap (ServiceNow's role-gated Basic Authentication restriction)
- Microsoft Entra ID licensing tiers, group-based application assignment requires P1 or P2, documented here as a real tier limitation rather than a design flaw
- Full Joiner and Leaver lifecycle automation, verified through Entra's provisioning logs and independently confirmed inside the target application itself

## Technologies Used

Microsoft Entra ID, Enterprise Applications, Automatic User Provisioning, ServiceNow (Personal Developer Instance), Salesforce Developer Edition, Basic Authentication, Role-Based Access Control
