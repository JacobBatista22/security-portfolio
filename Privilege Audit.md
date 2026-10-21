# Auditing Resource-Based Access Privileges

## Scope and Methodology

As a Junior Cloud Engineer, I received a task to audit resource access across our entire Azure cloud tenant. I started the audit with standard Reader permissions and activated an eligible Privileged Identity Management role when I needed deeper visibility into the environment.

I used four native Azure tools to collect data and verify permissions across the tenant:
* Identity and Access Management blade
* Azure Command Line Interface
* Azure Resource Graph Explorer with Kusto Query Language
* Privileged Identity Management

Using these four tools together helped me build a complete baseline of all role assignments. I located orphaned accounts, identified lingering permissions across subscriptions, and audited both eligible and active admin roles.

## Tool Progression and Blind Spots

A major lesson I learned during this audit is that no single Azure tool shows the complete picture of access control. Each tool has a specific blind spot that the next tool in the workflow solves. I had to combine all four methods to get full visibility across the tenant.

First, I checked the Identity and Access Management blade in the portal and exported the role assignments. This gave me a baseline of every identity with access at a single scope, including inherited permissions. The blind spot here is that the portal lists security groups without showing their individual members. It also hides orphaned role assignments by mixing them in as unresolvable identities without showing their full status.
<br><br>
<img width="1422" height="607" alt="image" src="https://github.com/user-attachments/assets/6831a1ff-ba2b-47d0-8d32-7eecb0d5f669" />


<br><br>
Second, I moved to the Azure Command Line Interface to run repeatable checks. The command line exposed what the web portal hid. When a role assignment returned an empty principal name in my terminal output, it proved that an admin deleted the user account from Entra ID without revoking its resource permissions first. The blind spot of the command line is scale, because it only checks one scope per command.

<br><br>
<img width="1641" height="852" alt="Screenshot 2026-10-20 011238" src="https://github.com/user-attachments/assets/74fa8191-7c8a-44ab-a6c6-61b991fa2dec" />
<br><br>
Third, I used Azure Resource Graph Explorer and wrote Kusto Query Language queries. This solved the scale problem by letting me check every subscription and resource group across the entire tenant in a single query. However, Resource Graph has its own blind spot because it only returns active role assignments and misses dormant privileges waiting for activation.
<br><br>
<img width="1755" height="727" alt="Screenshot 2026-10-20 012010" src="https://github.com/user-attachments/assets/16775922-ea97-42d3-9412-162848763778" />
<br><br>
Fourth, I exported the role data from Privileged Identity Management. This was the only tool that showed eligible role assignments alongside active ones. It also gave me the activation history so I could verify who elevated their access, when they did it, and what business justification they entered. The blind spot of Privileged Identity Management is that it completely ignores standing role assignments created outside of its management scope.
<br><br>
<img width="1907" height="556" alt="Screenshot 2026-10-20 012538" src="https://github.com/user-attachments/assets/b0a95998-8765-48fc-aec4-fb9e3b41d9e9" />
<br><br>
<img width="1421" height="511" alt="image" src="https://github.com/user-attachments/assets/52b61e3f-38c4-4573-a227-ae2798dba129" />

<br><br>

## The Hunt

For the final phase of the audit, I combined data from all four tools to hunt down an overprovisioned account on my own. I activated my eligible Privileged Identity Management role and cross referenced the Kusto Query Language tenant export against the Privileged Identity Management activation logs. By comparing active standing assignments against the list of approved eligible roles, I isolated a user account holding high level standing permissions across multiple scopes without any business justification or time limits.

| Method | Sees | Misses |
| :--- | :--- | :--- |
| IAM blade and export | Active assignments at a single scope and inherited roles | Group members and clear visibility into orphaned principals |
| Azure CLI | Active assignments at a single scope and null principal names | Tenant-wide visibility since it runs one scope at a time |
| Resource Graph with KQL | Active role assignments across the entire tenant in one query | Eligible assignments managed in Privileged Identity Management |
| PIM export | Eligible versus active roles and full activation history | Standing role assignments created outside of PIM |

## Findings by Severity

During the audit, I documented four main classes of access control issues.

The orphaned role assignments stood out as a critical operational gap. When an admin deletes a user or service principal without revoking its role assignments first, Azure keeps the role assignment tied to the unique identifier of that deleted account. These permissions sit dormant on the resource with nobody monitoring them. If a threat actor finds a way to recreate a principal with that same identifier or exploits the lingering configuration during an automated deployment, those permissions are already waiting for them. It also proves that our offboarding process is incomplete.

| Severity | Finding | Description |
| :--- | :--- | :--- |
| Very High | Standing Privileged Access | High-level admin roles are assigned permanently to accounts instead of using Privileged Identity Management. If an attacker compromises one of these accounts, they get immediate control of the tenant without triggering elevation alerts. |
| High | Redundant Owner Grants | Broad Owner permissions appear repeatedly across multiple scopes, likely created by an automated deployment script. This breaks the principle of least privilege and lets accounts alter access controls on resources they do not manage. |
| Moderate | Orphaned Role Assignments | Role assignments remain attached to deleted principals where the principal name returns null. These unmonitored grants clutter the access control list and show that our offboarding workflow deletes accounts before revoking their access. |
| Low | Direct User Assignments | Permissions are assigned directly to individual users instead of security groups. This makes regular access reviews difficult for the engineering team and increases the chance of leftover permissions when employees switch roles. |

## Remediation Plan

1. Revoke all orphaned role assignments where the principal name returns null. I also recommend updating our offboarding runbook so engineers or automated scripts remove all role assignments before deleting an identity.
2. Remove redundant Owner grants across the tenant and replace them with specific job function roles scoped only to the exact resource groups needed for daily engineering work.
3. Convert all standing admin access to eligible assignments in Privileged Identity Management. Enforce multifactor authentication, a written justification, and strict time limits on every activation.
4. Move direct user role assignments over to security groups so the team manages resource access through group membership rather than individual grants.
5. Schedule this four step audit to run every quarter and turn the exported findings directly into Jira tickets so the engineering team tracks every fix to completion.
