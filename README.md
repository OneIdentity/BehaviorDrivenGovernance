# Behavior Driven Governance
__Identity Manager__ & __OneLogin__ Solution Accelerator

## --->  This feature is now included in One Identity Manager  <---

## Overview
This solution accelerator allows you to reduce standing privilege and license cost by evaluating event data from OneLogin to inform Identity Manager of whether a user has used an application on their OneLogin launchpad in a specified period of time, such as 90 days. Then an attestation can be run which will provide a recommendation that the user's manager should revoke the applications which are unused in this time period. As an alternate application of this functionality, rather than an attestation with recommendation, it could be automatically revoked, thus reducing certification fatigue.

## Supportability
This Solution Accelerator is delivered "as is".  Any issues encountered can be reported on Github and contributors will make a best effort to resolve them.

## Basic Functionality
With Identity Manager 9.0, a new OneLogin connector is provided, which includes a number of special OneLogin `OLG*` tables in Identity Manager.  The Behavior Driven Governance solution accelerator integrates Identity Manager and OneLogin to provide full visibility into which entitlements are being used and recommend removal of unused ones to minimize vulnerabilities.  This solution accelerator uses the `OLGUserHasOLGApplication` table. In addition, this solution includes the ability to use System Roles in Identity Manager to assign application access in OneLogin along with other access such as target system accounts and entitlements. The table used for this is the PersonHasESet table.

## Link to demonstration video
This video includes a high level overview of this feature: Behavior Driven Governance

[![Demo video](https://img.youtube.com/vi/tLmQK-zQmV8/0.jpg)](https://www.youtube.com/watch?v=tLmQK-zQmV8)

## Extending and Populating Tables
This solution extends the two tables `OLGUserHasOLGApplication` and `PersonHasESet` to include a `CCC_LastUsedDate` column. Then a script runs after the OneLogin synchronization has completed, which will update this `CCC_LastUsedDate` column for the user/application combination in the `OLGUserHasOLGApplication`, as well as the same column in the `PersonHasESet` column for any system role that contains the OneLogin application and is assigned to the affected user.

## Post-Synchronization Script
Whenever Identity Manager completes a synchronization of the OneLogin system attached, a script is run to go find any updates in the OneLogin event log and match them by user ID and app ID to do the following:
- Update the `OLGUserHasOLGApplication` object's LastUsed date with the event date.
- Find any OneLogin Role (not the same as an Identity Manager role) that includes the OneLogin application, and follow that `OLGRole` object to see if it is included in any System Role in Identity Manager. If so, look up the Person UID, and then update the `PersonHasEset` object's `LastUsed` date with the event date.

## Config Parameters
Several configuration parameters are used by the script to do its updates.

- Custom
  - OneLogin Events
    - Events
      - ClientID - the client ID of the API creds for OneLogin
      - ClientSecret - the client secret of the API creds for OneLogin
      - BaseURL - the URL of your OneLogin tenant
      - DaysToRetrieve - how many days of history to get with each call. This should be less frequent than your OneLogin synchronization, in order to ensure there is no gap in coverage.

## Governance Policies
A series of policies, and attestations are recommended that will ensure the governance of the OneLogin application assignment is effective, and samples are provided as recommendations on how to use this functionality.

### Attestation with Recommendation
An attestation can be run on a predetermined schedule, say every month, to approve users to keep access for applications not used in 90 days. Note: What we are attesting in this process is the System Role Membership, not the app assignment itself.

### Company Policy
A policy uses the same query to deliver an exception report for users who have not used an application it the past 90 days. It triggers on the PersonHasEset.CCC_LastUsedDate.

### Deployment and Configuration
Import Transport Files in Identity Manager
The following transport files should be imported in order, First '1 Transport - Schema' and then '2 Transport - Process & Script' (links below). Use the Database Transporter tool to import these transports.

### Transport files
- [1 Transport - Schema](https://github.com/OneIdentity/BehaviorDrivenGovernance/raw/main/1%20Transport_OneIM_v821_OneLogin_LastUsedDate_Schema.zip)
- [2 Transport - Process & Script](https://github.com/OneIdentity/BehaviorDrivenGovernance/raw/main/2%20Transport_OneIM_OneLogin_LastUsedDate.zip)

This will do the necessary schema extensions to the `PersonHasESet` and `OLGUserHasOLGApplication` tables.  It will also deploy the script and add the extra config parameters. For reference, the script and custom process are attached:

### Process
[CCC Update LastUsed Date Process.xml](CCC%20Update%20LastUsed%20Date%20Process.xml)

### Script
[CCC Update LastUsed Date Script.txt](CCC%20Update%20LastUsed%20Date%20Script.txt)

### OneLogin Sync Project -- Optional and Recommended Settings
Some optional settings changes are recommended for your OneLogin synchronization project. Perform these changes in the Synchronization Editor tool. Open your OneLogin sync project to perform these changes.

__Important:__ *It is required to use the 9.0.0 or later OneLogin connector in Identity Manager for this feature to work. If you are using the Starling Connect connector, you will first have to build a new connection to OneLogin and transition to the new connector.*

### Optional: Disable Event Workflow Step
The Events workflow in the OneLogin connector is enabled by default. The intent is to synchronize the event log from OneLogin into Identity Manager in order to enable OneLogin event driven actions. Depending on your deployment, there is a potential of exceeding the OneLogin API rate limit if you synchronize the OneLogin event log. This solution accelerator uses a script to collect the targeted event data, so synchronizing the OneLogin event log may be unnecessary.

In the Synchronization Editor, navigate to Workflows, choose Initial Synchronization workflow, and edit the Event step. Check the `Disabled` box in the `General` tab.

### Recommended: Scheduled Synchronization of OneLogin
Since the behavior-driven governance capability is triggered by the synchronization of your OneLogin tenant, it is recommended to synchronize your OneLogin data on a regular schedule. Enable the scheduled synchronization in the Synchronization Editor.

OneLogin has an hourly API rate limit imposed for most deployments. For this reason, the most efficient synchronization schedule for OneLogin is to do the sync hourly, which will result in evenly distributing the number of API calls to optimize the rate limit.

In the Synchronization Editor, navigate to `Configuration` > `Start up configurations`, and select `Edit Schedule...` under the `Initial Synchronization`. In the dialog, be sure to check the `Enabled` checkbox, and set the desired schedule, such as hourly. Alternately, you can create a new start up configuration to perform your hourly updates. See the Identity Manager documentation for more information on creating a new start up configuration.

### Governing OneLogin Applications with Identity Manager
Following are recommended conventions and application assignment architecture to make governing applications in OneLogin easy and effective. These recommendations are not required for the Behavior Driven Governance solution accelerator to function, but are best practices that should be considered.

### Naming Convention
It is recommended to utilize a naming convention and assignment architecture for provisioning and deprovisioning access in OneLogin and target systems of associated applications. The following is an example, and will be used in the queries and policies referenced later in this document.

- Identity Manager System Role - "Salesforce Access"
  - OneLogin Role - "Salesforce App"
    - OneLogin Application - "Salesforce"
  - Target System Account Definition - "Salesforce Account"
  - Target System entitlements - as needed

| Application | OneLogin Role | Identity Manager System Role |
| :--- | :--- | :--- |
| Salesforce |Salesforce App | Salesforce Access |
| Concur | Concur App | Concur Access |
| Microsoft 365 | Microsoft 365 App | Microsoft 365 Access |

### Policies & Attestation
Create the following policies and attestation policy to govern your users' OneLogin applications based on their behavior.

### Company Policy: App Governance for OneLogin Roles
Create a Company Policy: App Governance for OneLogin Roles. Use the following SQL for the condition:
```
UID_OLGRole in (
   select orl.UID_OLGRole  --, orl.DisplayName as RoleName, oa.DisplayName as AppName
   from OLGRoleApplication orla
   join OLGRole orl on orla.UID_OLGRole = orl.UID_OLGRole
   join OLGApplication oa on orla.UID_OLGApplication = oa.UID_OLGApplication
   Where 1=1
   And orl.DisplayName like ('% App')
   And   (Select count(*) from OLGRoleApplication ora where ora.UID_OLGRole = orl.UID_OLGRole) > 1
   OR orl.DisplayName not like (oa.DisplayName + '% App')
   OR oa.ProvisioningEnabled = 1
)
```

### Company Policy: App Governance for OneLogin Applications
Create a Company Policy App Governance for OneLogin Applications. Use the following condition:
```
UID_OLGApplication in (
   select oa.UID_OLGApplication  --, orl.DisplayName as RoleName, oa.DisplayName as AppName
   from OLGRoleApplication orla
   join OLGRole orl on orla.UID_OLGRole = orl.UID_OLGRole
   join OLGApplication oa on orla.UID_OLGApplication = oa.UID_OLGApplication
   Where 1=1
   AND orl.DisplayName like (oa.DisplayName + '% App')
   AND  (
       (Select count(*)
       From OLGRoleApplication ora2
       Where ora2.UID_OLGApplication = oa.UID_OLGApplication ) > 1
       OR oa.ProvisioningEnabled = 1
   )
)
```

### Company Policy: Applications not used in 90 days
Create a Company Policy: Applications not used in 90 days. Use the following condition:
```
CCC_LastUsedDate is not null
and
CCC_LastUsedDate < DATEADD(day, -90, GetUTCDate())
```

### Attestation Policy: OneLogin Applications not used in 90 days
Create an Attestation Policy to govern OneLogin applications and other entitlements or accounts based on user behavior. It is recommended to run this policy on a monthly basis. Choose whichever approval workflow and policy that conforms to your company's governance framework.

The key to the policy is in the condition:
```
CCC_LastUsedDate is not null
and
CCC_LastUsedDate < DATEADD(day, -90, GetUTCDate())
```

### OneLogin Configuration
Identity Manager governs application assignments into OneLogin users' dashboards through OneLogin Roles. In order to correctly govern these applications, a specific configuration needs to be employed.
For each app which you intend to govern, you must follow these rules:
- On OneLogin, a role is configured for each application you want to govern, which follows the naming convention above: Some Application App
- Any OneLogin application you would like to govern must only be added to one role, which follows the naming convention as above
- Provisioning is disabled for any application to be governed

The company policies are provided with this solution accelerator that will aid in detecting violations of these rules.
