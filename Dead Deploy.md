# Dead Deploy Lab: Investigating Intern Audit Trail

## Scenario
An intern with temporary Contributor access deployed a "test environment" over a weekend, cut every corner, and left. 
I came in on Monday as the on-call engineer with Reader access and had to reconstruct what happened and why governance did not stop it. 

## Environment
Live multi-user Azure training tenant with Reader Level Access, Azure Policy, Resource Manager, Resource Groups, Locks. 

## Investigation
The core. Numbered steps IN YOUR OWN WORDS: what you looked at, what you found, what you concluded at each step. 
6 to 12 screenshots of meaningful moments (portal views, query results, before/after).

In stage 1, I began by signing into the Azure portal and navigating to the Resource Manager to begin searching for this "test environment" amongst the many resource groups shown. 
To find the test environement as fast as possible I began with filtering the resource groups out by naming convention. 
Starting with resource group where the name did not begin with -rg. I found the test environment as the only result of this filter.
<img width="645" height="331" alt="image" src="https://github.com/user-attachments/assets/47d7a6b8-2fdb-47a4-8d30-1d6413849449" />

In stage 2, I continued the investigation by inspecting the resource group. Inside the resource group was a single resource that the intern had deployed.
That resource contained tags such as the owner, cost center, environment, and a flag the intern was instructed to use.
<img width="1192" height="792" alt="image" src="https://github.com/user-attachments/assets/ebdb340d-8742-472a-b520-209ed5e371f0" />
<img width="1717" height="445" alt="image" src="https://github.com/user-attachments/assets/317544b4-9852-42d9-a837-e70d266f1266" />

In stage 3, I had to trace the deployment to find more information about the resource's creation. Navigating back to Resource Group Overview -> Settings -> Deployments.
After looking through the details of this deployment and its naming, I can confirm who deployed this and when.
<img width="1897" height="645" alt="image" src="https://github.com/user-attachments/assets/2045faa0-0dba-4473-95e8-d4139339318a" />
<img width="1517" height="507" alt="image" src="https://github.com/user-attachments/assets/fb5531bc-1ed6-4163-b115-3adeadb80b6c" />










## What broke / what surprised me
The most credible section in the document. Dead ends, wrong guesses, the thing that took an hour. Employers know real work is messy. This section separates you from certificate collectors.

## Findings and recommendations
What you determined, plus 2 or 3 recommendations as if you were reporting to the resource owner.

## What I learned
3 to 5 bullets. At least one technical, one "what I'd do differently."
