---
author: "Zach Stagers"
title: "Changing a Data Factory Integration Runtime Registration Key"
date: "2021-12-14"
#description: "Explaining the steps required to change which Data Factory an Integration Runtime is registered with."
tags: 
    - "Azure"
    - "Data Factory"
    - "Integration Runtime"
    - "PowerShell"
image: banner_change_adf_ir.jpg
---

### Introduction

When you've installed an integration runtime (IR) and connected it to an Azure Data Factory (ADF), it isn't exactly obvious how you change which Data Factory that IR is connected to. This short blog talks you through the process of updating an IR's connection.

### Steps to change the IR's Key

Firstly, confirm the IR is in fact already registered to a Data Factory. Open the Integration Runtime Configuration Manager and you should see a green tick and 'Self-hosted node is connected to the cloud service' with details of the ADF resource it's connected to. If you don't see this, and instead see a text box with 'Register Integration Runtime (Self-hosted)' above it then your IR isn't registered and you can skip all of the steps below and just paste the key into the registration box and hit go.

On to how to change which ADF resource your IR is registered with...

When you install an IR, a PowerShell script is placed alongside the installation which can be used for this task. Note that your folder path may not be identical to mine, as your IR may not have been installed in the default location, or you may have a different IR version.

1. From the machine hosting the IR, open the Windows PowerShell ISE as Administrator (right click, Run as admin...)

2. Within the ISE click File, Open, and navigating to: `C:\Program Files\Microsoft Integration Runtime\5.0\PowerShellScript`

3. Open the `RegisterIntegrationRuntime.ps1` PowerShell script.

4. Populate the parameters:
    * **$gatewayKey** is the registration key of the new ADF resource you want to register the IR with. Remove the $throw so it's just a string of the key.
    * **$IsRegisteringOnRemoteMachine** can be left as it's default, false, as in step 1 we're running this script from the machine hosting the IR.
    * **$NodeName** is the name that'll appear in ADF when it's registered. This is typically populated with the machine name hosting the IR.

5. Your script should look something like the screenshot below. Press run, it may take a minute or so, but once successful you'll see the message 'Integration Runtime registration is successful!'

6. Re-open the Integration Runtime Configuration Manager and confirm it is now registered with the new IR.

![RegisterIntegrationRuntime.ps1 parameters and success message.](powershell_adf_integration_runtime_change.jpg)

If you get either of the errors below, you may need to run `Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass` in PowerShell before you can run the script:
* `C:\Program Files\Microsoft Integration Runtime\5.0\PowerShellScript\RegisterIntegrationRuntime.ps1 cannot be loaded. The contents of file C:\Program Files\Microsoft Integration Runtime\5.0\PowerShellScript\RegisterIntegrationRuntime.ps1 might have been changed by an unauthorized user or process, because the hash of the file does not match the hash stored in the digital signature. The script cannot run on the specified system. For more information, run Get-Help about_Signing..` 
* `Integration Runtime registration has failed with below : Cannot open DIAHostService service on computer`

### Why might I have to do this?

To give a bit of context to those who're simply hosting an IR inside a network for someone else to use, you might need to undertake this process if significant changes have occurred in the Azure environment using the IR. For example, they may be migrating to a different tenancy, subscription, or are setting up a different ADF IR implementation by linking all IR's to a single ADF and sharing and managing them from there. It should be quite rare once in production that this needs to be done.