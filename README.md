# Enterprise App Provisioning Lab

## Overview

This lab configures automatic user provisioning for an enterprise application in Microsoft Entra ID, scoped to specific users, entirely through admin console configuration, no custom code involved. Unlike the SCIM Provisioning Lab, which built the receiving side of a SCIM integration from scratch, this lab works from the other direction, using a pre-built connector already provided by Microsoft to automatically create, update, and deactivate accounts in a real target application.

This lab was also built to clear up a specific point of confusion from SC-300 practice exams: the difference between provisioning, Conditional Access, and self-service access. Provisioning controls whether an account exists in an application at all. Conditional Access controls the conditions under which someone who already has access can sign in, things like requiring MFA or a compliant device. Self-service, through Entra's Entitlement Management, controls how someone gets assigned access in the first place, by requesting it and getting approved, rather than being placed into a group directly by an admin. This lab focuses entirely on the first one.

## Environment

- Identity Provider: Microsoft Entra ID (tenant: fernandotech187.onmicrosoft.com)
- Target application: ServiceNow, Personal Developer Instance (Australia release)
- Provisioning protocol: Automatic provisioning via Entra's built-in ServiceNow connector, Basic Authentication
- Also attempted: Salesforce Developer Edition, documented below as a real platform-level dead end rather than a working path

## What I Built

### Adding an Enterprise Application from the Gallery

Added Salesforce as an Enterprise Application in Entra ID directly from the application gallery. This is a different object than an App Registration. An App Registration is a blueprint for an app you build yourself. An Enterprise Application from the gallery is a connector Microsoft already built for a real SaaS product, so turning on provisioning means configuring an existing integration, not writing one.

### Salesforce Automatic Provisioning: A Platform-Level Dead End

Attempted to configure automatic provisioning against a free Salesforce Developer Edition org (reused from the Okta SSO and MFA Lab). Entra's Salesforce connector authenticates using a username, password, and security token. Every attempt failed with a generic CredentialValidationUnavailable error.

Investigation showed Salesforce has been retiring this exact login method (the OAuth 2.0 Username-Password Flow) platform-wide since 2023 for security reasons. The toggle to allow it under OAuth and OpenID Connect Settings was greyed out entirely, meaning it can no longer be re-enabled by an admin, not through a setting, not through a support ticket. The only real fix on Salesforce's side would be building a custom OAuth-based integration by hand, which is exactly the custom-code territory this lab was meant to avoid. This was documented as a genuine, permanent platform limitation rather than a misconfiguration, and the lab moved to a different target application instead of working around it with code.

### Pivoting to ServiceNow

Created a free ServiceNow Personal Developer Instance (PDI) on the Australia release and added ServiceNow as an Enterprise Application in Entra from the gallery. ServiceNow's connector uses Basic Authentication with a username and password rather than Salesforce's retired flow, so it was not affected by the same platform-level lockdown.

### Fixing a ServiceNow Basic Authentication Restriction

The first connection test also failed, with an InvalidCredentials error, despite correct credentials. ServiceNow has recently restricted Basic Authentication API access for accounts used for both interactive login and API integrations, a similar security hardening move to Salesforce's, but implemented as a role-based exception instead of a full retirement. Granting the admin account the snc_basic_auth_api_access role resolved this, and the connection test succeeded immediately after. This is a useful contrast to the Salesforce issue: one platform closed the door permanently, the other gated it behind a specific role.

### Scoping Provisioning to Specific Users

Created a security group intended to scope provisioning to a small set of test users. Group-based assignment to an Enterprise Application requires Microsoft Entra ID P1 or P2 licensing. This tenant's premium trials (Entra ID P2, Intune, Business Premium) had all already expired from earlier labs and cannot be restarted once a trial ends. Rather than rebuild the entire lab on a new tenant to chase a one-time trial, this was documented as a licensing tier limitation, and the two test users were assigned to the application individually instead of through the group. Provisioning is still fully scoped, just to specific users rather than a group object, which are two variations of the same underlying assignment-based scoping model.

### Verifying Automatic Provisioning End to End

Started provisioning and confirmed the initial sync completed successfully, 100 percent, 2 users processed. Verified this directly in ServiceNow's user list, both test accounts existed with matching usernames and creation timestamps, created automatically with no manual account creation involved.

### Demonstrating Deprovisioning

Removed one of the two test users from the application's assignment in Entra, then ran Provision on demand for that user instead of waiting for the next scheduled sync cycle. The result showed a successful attribute change, the account's active status flipped from true to false. Confirmed the same change directly in ServiceNow's user list, the account's Active field now read false, with an updated timestamp matching the deprovisioning action, while the second test user remained untouched and active. This demonstrates the full Joiner and Leaver lifecycle, accounts are automatically created on assignment and automatically deactivated on removal, without deleting the account or its history.

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
