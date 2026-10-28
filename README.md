# WINDOWS-AUTOPILOT-MICROSOFT-INTUNE
A step-by-step reference for registering devices, creating deployment profiles, and assigning them to PCs through Entra ID groups.
Table of Contents 
•	1. Overview 
•	2. Prerequisites 
•	3. Key Concepts 
•	4. Step-by-Step: Registering and Assigning a PC 
•	5. Troubleshooting Common Issues 
•	Appendix A: Useful PowerShell Commands 
•	Appendix B: Reference Links 
 	 
1. Overview 
 
Windows Autopilot lets you deploy new PCs directly to end users — the device sets itself up out of the box, applies your organization's branding, and enrolls into Microsoft Intune without IT ever imaging the machine. "Assigning a PC" in this context means making sure a specific device is registered, is a member of the right group, and that group has the deployment profile, Enrollment Status Page, apps, and policies you want that PC to receive. 
This guide covers the full lifecycle end to end and then focuses in on the steps people usually mean when they ask "how do I assign a PC": Step 3 (group) and Step 4 (deployment profile assignment) in Section 4. 
<img width="969" height="372" alt="image" src="https://github.com/user-attachments/assets/e8c981c7-d7cf-425f-851c-49713933e843" />
. Prerequisites 
 
2.1 Licensing 
Any of the following licenses (assigned to the users who will sign in on the devices) enables Autopilot and Intune enrollment: 
•	Microsoft 365 E3/E5 or Business Premium 
•	Enterprise Mobility + Security (EMS) E3/E5 
•	Intune Plan 1 (standalone) 
<img width="990" height="354" alt="image" src="https://github.com/user-attachments/assets/af18ca46-281d-4fe6-aaa7-7de70015bf0f" />
<img width="1050" height="399" alt="image" src="https://github.com/user-attachments/assets/0abff811-5977-4873-9c13-19d2c600412f" />

Roles and Permissions 
You need one of the following Microsoft Entra ID / Intune roles to register devices and manage assignments: 
•	Intune Administrator, or 
•	Global Administrator, or 
•	A custom role with the "Enrollment Programs" and "Managed Devices" permissions granted 
<img width="1050" height="349" alt="image" src="https://github.com/user-attachments/assets/7995c4af-d3c2-4390-b9c4-7f1cec4b45b4" />

Device Requirements 
•	Windows 10/11 Pro, Enterprise, or Education (Home does not support Autopilot) 
•	Internet access during first boot (Ethernet, or Wi-Fi if the OEM supports it at OOBE) 
•	TPM 2.0 recommended for full feature support 
•	Device not already associated with another tenant's Autopilot program Need this code to find device hardware ID: 
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12 
New-Item -Type Directory -Path "C:\HWID" 
Set-Location -Path "C:\HWID" 
$env:Path += ";C:\Program Files\WindowsPowerShell\Scripts" 
Set-ExecutionPolicy -Scope Process -ExecutionPolicy RemoteSigned 
Install-Script -Name Get-WindowsAutopilotInfo 
Get-WindowsAutopilotInfo -OutputFile AutopilotHWID.csv 
<img width="1050" height="508" alt="image" src="https://github.com/user-attachments/assets/9199fd45-b3cc-406d-8842-93351e9d4879" />
<img width="1050" height="379" alt="image" src="https://github.com/user-attachments/assets/ac927a66-4a79-4f7c-a7d1-316b1c0e6291" />
<img width="1050" height="621" alt="image" src="https://github.com/user-attachments/assets/6bc51d84-cd77-48e9-a3bd-28dcc54aecc3" />
OEM/Reseller registration (recommended for new purchases) 
Ask your PC vendor or reseller to register the device's hardware hash to your tenant at time of purchase using your Autopilot registration ID. No action needed on your end once this is set up — new devices simply appear in Intune. 
Option B: Partner Center / Microsoft 365 admin center bulk import 
For devices bought without OEM registration, use the CSV/hash upload process in the Microsoft 365 admin center or Partner Center if you're an MSP managing a customer tenant. 
Option C: Manual capture with PowerShell (existing or test devices) 
Run this from an elevated PowerShell prompt on the device itself to generate a CSV containing the hardware hash: 
Install-Script -Name Get-WindowsAutoPilotInfo -Force 
Set-ExecutionPolicy -ExecutionPolicy Bypass -Scope Process -Force 
Get-WindowsAutoPilotInfo.ps1 -OutputFile C:\Autopilot\device.csv -GroupTag "Sales-Laptops" 

The -GroupTag parameter is what lets you sort this device into a dynamic group automatically — set it to something meaningful like a department, site, or model name. 
Step 2 — Import the device into Intune 
1.	Sign in to the Intune admin center (intune.microsoft.com). 
2.	Go to Devices > Windows > Windows enrollment > Devices. 
3.	Select Import, browse to the CSV from Step 1, and upload it. 
4.	Wait a few minutes, then Sync confirms the device appears with its Serial Number and Group Tag. 
<img width="1050" height="468" alt="image" src="https://github.com/user-attachments/assets/755d965c-d988-4f24-9593-9b0108261a0c" />
<img width="1050" height="429" alt="image" src="https://github.com/user-attachments/assets/50454812-c371-4da9-bfa6-409a74461147" />
<img width="1050" height="362" alt="image" src="https://github.com/user-attachments/assets/7699f65f-90c1-409b-b11c-8aa31847452c" />
Create (or confirm) an Entra ID group for the device 
This is the core of "assigning" the PC. Everything downstream (profile, ESP, apps, policies) attaches to this group, not to the device itself. 
Dynamic group (scales automatically — recommended) 
Create a Microsoft Entra ID > Groups > New group, type = Security, Membership type = Dynamic Device, and use a rule based on the Group Tag you set in Step 1, for example: 
(device.devicePhysicalIds -any (_ -contains "[OrderID]:Sales-Laptops")) 
Any device you assign this Group Tag to — now or in the future — automatically becomes a member of this group and inherits everything assigned to it. 

Static/assigned group (manual, precise control) 
Create a regular Security group with Membership type = Assigned, then manually add the specific device (search by device name/serial in the group's Members blade) or add the target user if you're doing user-driven, per-user assignment instead of per-device. 
<img width="1050" height="501" alt="image" src="https://github.com/user-attachments/assets/680272d1-769e-4bb6-9553-030d26b93b79" />
<img width="1050" height="709" alt="image" src="https://github.com/user-attachments/assets/bed53d60-fa5c-4f89-b364-b9a389cb94d8" />

Create and assign the Deployment Profile 
1.	In Intune, go to Devices > Windows > Windows enrollment > Deployment Profiles. 
2.	Select Create profile > Windows PC. 
3.	Configure Out-of-box experience (OOBE): deployment mode (User-Driven or Self-Deploying+), Join type (Entra ID joined or Hybrid), and whether to hide the privacy/EULA/account screens. 
4.	On the Assignments tab, click Add groups and select the Entra ID group from Step 3. 
5.	Review + create. 
Assignment rule 
A Deployment Profile, once assigned to a group, applies to every current and future member of that group. To assign the profile to one specific PC, that PC's device record must be the only (or intended) member of the group you selected in this step. 

<img width="968" height="420" alt="image" src="https://github.com/user-attachments/assets/44386baf-737e-44b9-b148-cb53a788f5ad" />
<img width="1050" height="463" alt="image" src="https://github.com/user-attachments/assets/2638f92f-6ae6-45af-bb14-3b586af3313c" />
<img width="845" height="807" alt="image" src="https://github.com/user-attachments/assets/9d47065d-1015-424d-81a4-da32afa7d8aa" />
<img width="831" height="589" alt="image" src="https://github.com/user-attachments/assets/b057cf9b-54e6-4825-a807-cf712811d26e" />

Assign apps, configuration, and compliance policies (optional but typical) 
Repeat the same pattern for anything else you want the PC to receive automatically: go to the app, configuration profile, or compliance policy in Intune, open its Assignments tab, and add the same Entra ID group. Because everything points at one group, you only manage membership in one place
<img width="1050" height="541" alt="image" src="https://github.com/user-attachments/assets/b5e1a3d4-85b2-4154-92ad-5fc8bb369bf9" />
<img width="1050" height="545" alt="image" src="https://github.com/user-attachments/assets/d1b63498-5b73-4d2f-aa3e-e3f0cdb36306" />
<img width="1050" height="653" alt="image" src="https://github.com/user-attachments/assets/a2bd9780-0e95-4b30-8ea1-a8b4e3479308" />
<img width="1050" height="634" alt="image" src="https://github.com/user-attachments/assets/823b4b8b-dba8-40bf-9065-cb4ef01475e7" />
<img width="1050" height="499" alt="image" src="https://github.com/user-attachments/assets/cea8fa4a-1e74-4d74-b223-58b0b74525f1" />

— Configure the Enrollment Status Page (recommended) 
Go to Devices > Windows > Windows enrollment > Enrollment Status Page, create or edit a profile, set "Show app and profile installation progress" to Yes, choose which apps/policies block completion, and assign it to the same group. 
<img width="1050" height="539" alt="image" src="https://github.com/user-attachments/assets/455604ab-d39a-4473-8890-31aa423b49ed" />
<img width="1050" height="451" alt="image" src="https://github.com/user-attachments/assets/c27337a7-5dd3-44be-8d8b-8549eacda847" />
<img width="1050" height="374" alt="image" src="https://github.com/user-attachments/assets/1a9de712-1c53-42ed-bbcb-1a1798b92200" />

Push App (google, ChatGPT and Microsoft office to assign pc 
<img width="1050" height="408" alt="image" src="https://github.com/user-attachments/assets/85f0ac0e-80e4-4e8e-aed1-0d661179fe23" />
<img width="1050" height="447" alt="image" src="https://github.com/user-attachments/assets/0be670bc-9f7f-4c4c-8e4e-6a3def322a58" />
<img width="1050" height="428" alt="image" src="https://github.com/user-attachments/assets/2464c29a-e9d1-48c7-b588-6116a582bdfb" />
<img width="1050" height="359" alt="image" src="https://github.com/user-attachments/assets/05d2cdc7-a6ef-4213-b54c-a050ed5eb782" />

<img width="1050" height="484" alt="image" src="https://github.com/user-attachments/assets/16566247-8b83-4622-baaa-8fc238f07e60" />

Install Google line-of-business-app 
<img width="1050" height="367" alt="image" src="https://github.com/user-attachments/assets/b584b18b-9dc7-4af3-93c9-d8fc60a0ffd7" />
<img width="1050" height="432" alt="image" src="https://github.com/user-attachments/assets/6511923d-5ab8-4d04-b860-329693465fcd" />


Verify the assignment 
•	Devices > Windows > Windows enrollment > Devices — confirm the device shows the correct Group Tag and Profile status = Assigned. 
•	Groups > [your group] > Devices — confirm the target device is listed as a member. 
•	Deployment Profile > Devices tab — confirm the device is listed with Deployment status. 
Step 8 — Test by powering on the device 
Power on (or reset) the PC and connect to the internet. It should show your organization's branding, skip the screens you configured, sign the user in, and — if you enabled, show installation progress before handing control to the user. 
<img width="1050" height="405" alt="image" src="https://github.com/user-attachments/assets/25d17706-8f30-434b-92da-0bef6eb706b0" />
<img width="1041" height="502" alt="image" src="https://github.com/user-attachments/assets/038a3fda-2232-47b4-bdbc-c16740503233" />
<img width="1028" height="459" alt="image" src="https://github.com/user-attachments/assets/dc4faf52-b2e3-4bd0-ab03-1bd2a51f81f1" />
<img width="1028" height="544" alt="image" src="https://github.com/user-attachments/assets/57cffbd4-9b9f-41f9-a66b-37f4f3d76138" />

Troubleshooting Common Issues 
Symptom 	Likely cause / fix 
Device doesn't show Autopilot branding at OOBE 	Device not yet synced, or profile not assigned to the group the device is in. Run a manual sync and re-check group membership. 
Device stuck on "Preparing your device" (ESP) 	A required app/policy is failing to install. Check Devices > Monitor > Autopilot deployment (Preview) or the app's Device install status. 
Symptom 	Likely cause / fix 
Dynamic group not picking up the device 	Confirm the exact Group Tag spelling matches the rule; dynamic group membership updates can take up to a few hours. 
"This device is already enrolled" error 	Device was previously enrolled/wiped without a proper retire. Delete the device record from both Intune and Entra ID, then re-import. 
Autopilot profile shows "Not assigned" 	The Entra ID group has zero members, or the device's hash import hasn't completed. Re-check Step 2 and Step 3. 
Appendix A: Useful PowerShell Commands 
 
Retrieve hardware hash and register directly from a reference device without leaving PowerShell: 
Install-Module -Name WindowsAutoPilotIntune -Force 
Connect-MSGraph 
Get-WindowsAutoPilotInfo.ps1 -Online -GroupTag "Finance-Desktops" -
AssignedComputerName "FIN-%SERIAL%" 
Check enrollment status remotely once a device is assigned: 
Get-AutopilotDevice | Where-Object {$_.serialNumber -eq "<SerialNumber>"} | Select * 
Appendix B: Reference Links 
 
•	Microsoft Learn: Windows Autopilot overview and deployment scenarios 
•	Microsoft Learn: Create an Autopilot deployment profile 
•	Microsoft Learn: Assign devices to Autopilot Deployment Profiles using Entra ID groups 
•	Microsoft Learn: Enrollment Status Page settings reference 
(Search these titles on learn.microsoft.com — links change periodically as Microsoft reorganizes documentation.) 




