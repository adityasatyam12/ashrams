CAPSTONE PROJECT 3 – DEVOPS
Project 3: Automated DevOps Pipeline for Infrastructure Deployment
Case Study Scenario: A software development firm wants to adopt DevOps practices to accelerate deployments while minimizing risks. The pipeline must support infrastructure as code and advanced release strategies, emulating a production environment for continuous delivery.
Objectives:
• Automate infrastructure provisioning and application deployment.
• Incorporate security scans and phased rollouts.
• Foster a DevOps culture with collaborative roles.
Key Technologies/Topics Covered:
• DevOps, DevOps Culture Role, Terraform, Jenkins, CI/CD, Canary, Blue/Green Pipelines.
High-Level Architecture:
• Use Terraform for IaC to define Azure resources (VMs, networks, storage).
• Integrate Jenkins for CI builds and Azure DevOps for CD pipelines.
• Implement blue/green deployments for zero-downtime updates and canary releases for testing subsets of traffic.
• Include security gates with tools like Azure Security Center.
• Version control in Azure Repos with branching strategies.
Implementation Outline:
1. Write Terraform scripts for Azure infrastructure and store state remotely.
2. Set up Jenkins to trigger builds on code commits, running tests and scans.
3. Configure Azure Pipelines for CD, including blue/green swaps and canary traffic routing.
4. Define DevOps roles (e.g., developers for code, ops for monitoring) and use collaboration tools like Azure Boards.
5. Deploy and test in staged environments (dev/test/prod) to ensure seamless operation.
Why It's Production-Ready: Simulates enterprise DevOps workflows with automated, secure pipelines that reduce human error and support scalable, resilient deployments.
STEPS:
1.	Create a folder directory ex- capstone
2.	Write Terraform  code for infrastructure creation like vnet, subnet, vm. –main.tf and variables.tf
3.	Created Jenkins file in capstone folder for terraform init+ terraform validate.
4.	Uploaded folder to github repo. Must create gitignore.
5.	Go to Jenkins and run the pipeline for CI( Build+Test). Used github repo
6.	For CD we will use Azure DevOps.
7.	In azure devops, create a pipeline and choose github repository.
8.	Choose starter pipeline and write yaml file for terraform plan+ apply.

Must add credentials by going to pipeline	> edit pipeline > variables> add all credentials like like 
AZURE_CLIENT_ID
AZURE_CLIENT_SECRET
AZURE_TENANT_ID
AZURE_SUBSCRIPTION_ID
This will be used to perform operation to Azure portal.
 

yaml file:


trigger:
  enabled: false
pool:
  name: 'Default'
stages:
- stage: Terraform_Plan
  displayName: Terraform Init & Plan
  jobs:
  - job: Plan
    displayName: Terraform Plan
    steps:
    - task: 6d15af64-176c-496d-b583-fd2ae21d4df4@1
      inputs:
        repository: self
    - task: PowerShell@2
      displayName: Terraform Init
      inputs:
        targetType: 'inline'
        script: |
          terraform init
    - task: PowerShell@2
      displayName: Terraform Plan
      inputs:
        targetType: 'inline'
        script: |
          terraform plan `
            -var "subscription_id=$env:ARM_SUBSCRIPTION_ID" `
            -var "client_id=$env:ARM_CLIENT_ID" `
            -var "client_secret=$env:ARM_CLIENT_SECRET" `
            -var "tenant_id=$env:ARM_TENANT_ID"
- stage: Approval
  displayName: Manual Approval
  dependsOn:
  - Terraform_Plan
  jobs:
  - job: ApprovalJob
    pool:
      name: server
    steps:
    - task: ManualValidation@0
      inputs:
        instructions: 'Approve Terraform Apply?'
        timeoutInMinutes: 1440
- stage: Terraform_Apply
  displayName: Terraform Apply
  dependsOn:
  - Approval
  jobs:
  - job: Apply
    displayName: Terraform Apply
    steps:
    - task: 6d15af64-176c-496d-b583-fd2ae21d4df4@1
      inputs:
        repository: self
    - task: PowerShell@2
      displayName: Terraform Init
      inputs:
        targetType: 'inline'
        script: |
          terraform init
    - task: PowerShell@2
      displayName: Terraform Apply
      inputs:
        targetType: 'inline'
        script: |
          terraform apply -auto-approve `
            -var "subscription_id=$env:ARM_SUBSCRIPTION_ID" `
            -var "client_id=$env:ARM_CLIENT_ID" `
            -var "client_secret=$env:ARM_CLIENT_SECRET" `
            -var "tenant_id=$env:ARM_TENANT_ID"

Now we will perform Blue/Green deployment strategies and validate canary release.
9. Create 2 dummy applications named app1blue and app2green.

9.	Create app service in azure e.g. appblue- this will be used as our production environment, deploy app1blue to appblue app service. By going to Deployment Centre, choose github repo to deploy.

now go to Deployment Slot add additional slot and name it green.

10.	Go to created Deployment slot inside it go to Deployment Centre and deploy app2green to green environment.

11.	Now, Go to appblue > Deployment slot> Swap Slot

Source- Green 
Target- Production
 


12. Now Test the website, blue environment would show app2green as entire traffic has been moved to production from green.

For Canary Release, change the traffic configuration like add custom traffic like 50 % for blue and 50% for green.

STEP 1 — Set the GREEN slot traffic to 10%
In your screenshot, the Traffic % box is showing:
Copy code

100
This means 100% of traffic is going to green slot right now, which is why you can’t validate.
We need to reduce that.
✔ In the “Traffic %” field, enter:
Copy code

10
(Meaning:
10% traffic → GREEN (new version)
90% traffic → BLUE (production)
)
Then click Save at the top.
⭐ STEP 2 — Validate the Canary (IMPORTANT)
Use the main App URL:
Copy code

https://app1blue.azurewebsites.net
Now do the following:
✔ Open Incognito browser (fresh session)
✔ Refresh 10–15 times
You should see:
➡ MOST of the time:
Copy code

Version 1 - BLUE
➡ SOMETIMES (10%):
Copy code

Version 2 - GREEN
That means Canary is WORKING.
⭐ STEP 3 — Increase Canary gradually
Go back to the same traffic screen (the one in your screenshot):
Copy code

Traffic %: 10 → Change to 30
Click Save.
Validate again.
Then:
Copy code

Traffic %: 30 → Change to 50
Click Save.
Validate again.
Finally:
Copy code

Traffic %: 50 → Change to 100
Click Save.
🎉 Now 100% traffic goes to GREEN.
⚠ IMPORTANT NOTE
The Traffic % you are changing is ONLY FOR THE GREEN SLOT (staging).
Azure automatically calculates the rest of the traffic to BLUE.
Example:
Green Slot Traffic
Blue Slot Traffic
10%
90%
30%
70%
50%
50%
100%
0%
⭐ STEP 4 — OPTIONAL FINAL SWAP
Once GREEN is stable, you can make it the permanent production slot:
Copy code

App Service → Deployment Slots → Swap
Source: green
Target: production
This completes Blue/Green promotion.
🎉 GOOD NEWS
You are now using the latest, correct Azure interface for canary.
You don’t need Traffic Manager.
You don’t need old traffic routing screens.
Your screen is the NEW 2024 UI for slot canary.
📣 Satyam, if you want I can:
✔ Create the FINAL EXAM ANSWER for Canary
✔ Give validation screenshots guide
✔ Provide architecture diagram
✔ Provide viva questions + answers
Just say:
“Give me final exam answer for canary.”

After refreshing a lot my environment is not switching from blue to green like in my phone when I am opening link it's showing green always even after refreshing but in my laptop it is showing only blue


Satyam, this exact behaviour is 100% NORMAL and happens because of ARR Affinity Cookies + Browser Caching + IP Sticky Routing in Azure App Service.
You did NOT do anything wrong.
Azure is doing what it is designed to do.
Let me explain clearly and fix it step-by-step.
💥 WHY YOUR PHONE SHOWS GREEN ALWAYS
Your phone hit the app for the first time, so Traffic Routing gave it a random slot (Green).
After that, Azure added a special cookie:
ARRAffinity
or
ARRAffinitySameSite
This cookie forces your phone to KEEP going to the same slot
(either Blue or Green) EVERY TIME.
This prevents users from jumping between versions randomly.
So your phone is stuck to GREEN because Azure assigned it.
💥 WHY YOUR LAPTOP ONLY SHOWS BLUE
Your laptop had already visited the site before.
Azure gave your laptop the BLUE slot and saved the cookie.
So the laptop will ALWAYS stay on Blue,
no matter how many times you refresh.
🎯 THIS IS EXACTLY HOW CANARY IS MEANT TO WORK
Canary is NOT “change every refresh.”
It is:
“Each user gets a version and stays there.
Only new users go to the canary depending on percentage.”
This is REAL production behaviour.
Refreshing does nothing because of “Sticky Sessions.”
✔ HOW TO TEST CANARY PROPERLY (WORKING METHODS)
METHOD 1 — Private / Incognito Window
Each incognito window = new user.
To test:
On Laptop:
Open Chrome → New Incognito
Visit app URL
Refresh 2–3 times (should stay same)
Close window
Open a new incognito window
Visit URL again
→ You may now see Green, depending on percentage.
Do this 10 times:
You should see some windows show:
Copy code

Version 2 - GREEN
and some show:
Copy code

Version 1 - BLUE
METHOD 2 — Different Browsers
Use:
Chrome Normal
Chrome Incognito
Firefox
Firefox Private
Edge
Edge InPrivate
Safari (if available)
Phone Browser
Phone Incognito
Each behaves as a different user.
You will observe:
✔ Some browsers show BLUE
✔ Some show GREEN
THIS CONFIRMS CANARY WORKS.



