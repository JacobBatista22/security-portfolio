# Investigating OAuth Phishing Incident

## Scenario
Reconstructed a five-stage OAuth consent phishing kill chain in a live Azure tenant through forensic analysis of two linked app registrations.

The investigation uncovered an intrusion where the attacker bypassed conventional perimeter defenses by moving entirely across the identity plane. Analysis of the compromised application registration revealed the complete chain of actions.

## Environment
Live multi-user Azure training tenant, Reader access.

## Investigation

Stage one covers entry. An attacker phished a legitimate corporate user. The victim satisfied multifactor authentication during the initial login. The attacker captured the valid session token containing the satisfied multifactor claim. This token bypassed Conditional Access policies and showed up as routine sign-in traffic. Drift over several years had left this compromised user account assigned as an Owner on the legacy connector application registration.

Stage two covers escalation. The attacker leveraged their Owner rights on the legacy application registration to generate a new client secret. The attacker configured this secret with an expiration date set almost one century into the future. With this secret, the attacker switched to the OAuth client credentials flow. They authenticated as the application service principal itself. This action granted them the directory permissions of the application and removed the need to sign in with any human credentials.

Stage three covers pivoting. The attacker knew that a security team would eventually discover and rotate the newly minted secret. To establish reliable operational control, the attacker registered a secondary rogue application registration in the tenant. Standard Microsoft Entra ID tenant settings permit any normal user to register applications by default. The attacker then added the service principal of this rogue application directly to the Owners list of the legacy application. This link ensured the attacker could mint fresh client secrets on the legacy application at any point in the future.

Stage four covers persistence. The attacker created a backup channel using the Expose an API configuration blade on the legacy application. They defined and published a custom application scope. This change transformed the legacy application into an accessible backend resource. The rogue application could now request delegated permissions against this newly exposed resource.

Stage five covers extraction. The attacker configured a redirect URI on their rogue application pointing to an external attacker-controlled server. Combining the client ID of the rogue application, the malicious redirect URI, and the custom backend scope created a functional consent phishing link. When an authenticated employee on a compliant corporate device clicks this link and consents to the prompt, the authorization code routes straight to the attacker's infrastructure.

The core reason an attacker builds this infrastructure instead of phishing credentials repeatedly comes down to defense evasion and persistence durability. Traditional credential theft repeatedly collides with device compliance checks, trusted IP ranges, and multifactor prompts. Consent phishing sidesteps these controls entirely because the victim already possesses an active session on a trusted corporate device. More importantly, an OAuth2PermissionGrant object created through consent survives typical containment actions. Resetting a user password, revoking active refresh tokens, or enforcing new multifactor challenges will not invalidate an established OAuth grant. Standard incident response playbooks routinely leave these grants intact, giving attackers continuous silent access.

## What broke / what surprised me
The most surprising finding during this investigation is the default tenant configuration in Entra ID that allows any standard non-administrative user to register applications. Furthermore, application registration ownership functions as an unlogged, hidden privilege pathway. A routine audit focused strictly on Global Administrators and other built-in directory roles completely misses who holds ownership over critical service principals.

## Findings and recommendations
This attack structure represents a classic confused deputy problem. The trusted legacy application registration acts as the deputy. It executes actions and exposes scopes on behalf of the attacker, serving requests that security controls never should have permitted.

## What I learned
The most surprising finding during this investigation is the default tenant configuration in Entra ID that allows any standard non-administrative user to register applications. Furthermore, application registration ownership functions as an unlogged, hidden privilege pathway. A routine audit focused strictly on Global Administrators and other built in directory roles completely misses who holds ownership over critical service principals.

Findings and recommendations require immediate remediation across the identity footprint.

Revoke the unauthorized client secret on the legacy connector application immediately.

Remove the rogue service principal from the Owners list of the legacy application.

Delete the custom scope created under the Expose an API blade.

Remove the malicious redirect URI configured on the rogue application registration.

Delete the rogue application registration completely.

Revoke the malicious OAuth2PermissionGrant object explicitly using PowerShell or Microsoft Graph API, as basic account reset procedures will not clear it.

Review all existing Microsoft Graph application permissions assigned to the legacy service principal and reduce them to follow least privilege principles.

Update tenant settings to disable the default permission that allows standard users to register applications.

Implement regular access reviews for application registration Owners, treating app ownership with the same rigor as directory administrative roles.

Deploy detection alerts for the creation of new application client secrets, certificate additions, and modifications to application redirect URIs.
