# Investigating an Identity Attack in Entra ID

## Scenario
A team member deployed an unapproved test environment over the weekend that bypassed organizational naming standards. I received a ticket on Monday to locate the rogue deployment, identify the creator, and establish the exact timeline using reader-level permissions. This investigation determined how the deployment bypassed governance controls and answered why existing platform policies allowed the creation. 

## Environment
**Platform:** Azure Cloud \
**Services:** Azure Resource Manager, Azure Policy, Resource Groups \
**Tools:** Azure Portal \
**Access Level:** Reader 

## Investigation

### Stage 1
I signed into the Azure portal and navigated to Resource Manager to search for the "test environment" among the many resource groups shown. 
To find the test environment as fast as possible, I began filtering the resource groups to see if the deployer had ignored the prefix naming convention. 
Starting with resource groups where the name did not begin with -rg. I quickly found the "test environment" as the only result. 


<img width="645" height="331" alt="image" src="https://github.com/user-attachments/assets/47d7a6b8-2fdb-47a4-8d30-1d6413849449" />
<br><br>

### Stage 2
I continued the investigation by inspecting the testing environment resource group. 
Inside this resource group was only a single resource that had been deployed.
That resource contained useful tags such as the owner, cost center, environment, and a flag the deployer was instructed to use.


<img width="1192" height="792" alt="image" src="https://github.com/user-attachments/assets/ebdb340d-8742-472a-b520-209ed5e371f0" />
<br><br>

<img width="1717" height="445" alt="image" src="https://github.com/user-attachments/assets/317544b4-9852-42d9-a837-e70d266f1266" />
<br><br>

### Stage 3
To learn more about the creation of this resource, I had to trace its deployment. Navigating back to Resource Group Overview -> Settings -> Deployments.
Under the deployment blade was only a single deployment result whose naming gave me the answer to who ran it and roughly when, which is the start of any incident timeline.


<img width="1897" height="645" alt="image" src="https://github.com/user-attachments/assets/2045faa0-0dba-4473-95e8-d4139339318a" />
<br><br>

<img width="1252" height="412" alt="image" src="https://github.com/user-attachments/assets/6e9ebe22-c61d-4972-b948-c4aa72f822ab" />
<br><br>


### Stage 4
The lingering question of this investigation was "Why didn't the policy in place prevent this?" To solve this question, I navigated to the policies section: Resource Group Overview -> Settings -> Policies. The naming convention policy flagged the resource as noncompliant, but the misnamed resource group was deployed. To understand why, I looked further into the details of the policy assignment by navigating to Authoring-> Assignments -> Naming Convention Assignment. 
Under the assignment details, I found the parameters effect of this assignment to be set to audit, which only passively logs violations, unlike deny, which blocks them from being created. 


<img width="1852" height="737" alt="image" src="https://github.com/user-attachments/assets/0c080eae-8c97-440f-b5ee-30857bed1fec" />
<br><br>


<img width="1897" height="747" alt="image" src="https://github.com/user-attachments/assets/3c80350e-57c6-4639-bce8-e0a83e7cfbc7" />
<br><br>



## What broke / what surprised me
I expected the platform policy to block any resource group that lacked a proper naming prefix. However, discovering that the policy ran in audit mode explained why the portal permitted the operation without an outright error. 

## Findings and recommendations
I determined that an intern had ignored the company's governance standards and deployed an unauthorized resource group in a live Azure subscription. In creating and deploying this noncompliant resource group, the company's naming convention policy failed to prevent the violation because of a technical misconfiguration in the policy's assignment parameters. My recommendations moving forward are to properly configure the policy assignment parameters from audit to deny across all active subscriptions and to implement required training for all new personnel before granting any access/permissions to live platforms. 

## What I learned
* Reader-level access still permits full timeline reconstruction through resource deployment history.

- An Azure Policy with an audit effect logs non-compliance without preventing unauthorized deployments.

+ I need to document incident steps in real time during the investigation rather than writing the entire report from memory at the end.

