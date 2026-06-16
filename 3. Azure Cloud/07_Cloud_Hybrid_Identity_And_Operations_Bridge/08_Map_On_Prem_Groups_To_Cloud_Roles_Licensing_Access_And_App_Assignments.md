# 08_Map_On_Prem_Groups_To_Cloud_Roles_Licensing_Access_And_App_Assignments

# 08_Map_On_Prem_Groups_To_Cloud_Roles_Licensing_Access_And_App_Assignments_Index

08_Map_On_Prem_Groups_To_Cloud_Roles_Licensing_Access_And_App_Assignments.md  
08_Map_On_Prem_Groups_To_Cloud_Roles_Licensing_Access_And_App_Assignments  
08_Map_On_Prem_Groups_To_Cloud_Roles_Licensing_Access_And_App_Assignments_Source_Basis  
08_Map_On_Prem_Groups_To_Cloud_Roles_Licensing_Access_And_App_Assignments_Mental_Model  
08_Map_On_Prem_Groups_To_Cloud_Roles_Licensing_Access_And_App_Assignments_Planning_Table  
08_Map_On_Prem_Groups_To_Cloud_Roles_Licensing_Access_And_App_Assignments_Configuration_Checklist  
08_Map_On_Prem_Groups_To_Cloud_Roles_Licensing_Access_And_App_Assignments_Group_Design_Map  
08_Map_On_Prem_Groups_To_Cloud_Roles_Licensing_Access_And_App_Assignments_Precheck_Skeleton  
08_Map_On_Prem_Groups_To_Cloud_Roles_Licensing_Access_And_App_Assignments_On_Prem_Group_Inventory_Skeleton  
08_Map_On_Prem_Groups_To_Cloud_Roles_Licensing_Access_And_App_Assignments_Cloud_Group_Baseline_Skeleton  
08_Map_On_Prem_Groups_To_Cloud_Roles_Licensing_Access_And_App_Assignments_Group_Based_Licensing_Skeleton  
08_Map_On_Prem_Groups_To_Cloud_Roles_Licensing_Access_And_App_Assignments_Role_Assignable_Group_Skeleton  
08_Map_On_Prem_Groups_To_Cloud_Roles_Licensing_Access_And_App_Assignments_Enterprise_App_Assignment_Skeleton  
08_Map_On_Prem_Groups_To_Cloud_Roles_Licensing_Access_And_App_Assignments_Access_Validation_Skeleton  
08_Map_On_Prem_Groups_To_Cloud_Roles_Licensing_Access_And_App_Assignments_Verification_Commands  
08_Map_On_Prem_Groups_To_Cloud_Roles_Licensing_Access_And_App_Assignments_Rollback  
08_Map_On_Prem_Groups_To_Cloud_Roles_Licensing_Access_And_App_Assignments_Failure_Checks  
08_Map_On_Prem_Groups_To_Cloud_Roles_Licensing_Access_And_App_Assignments_Related_Labs  

# 08_Map_On_Prem_Groups_To_Cloud_Roles_Licensing_Access_And_App_Assignments_Source_Basis

| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Entra Docs | Group-based licensing | Assigning product licenses to groups, synced security group support, service plan control, membership-driven license assignment |
| Microsoft 365 Admin Docs | Assign or unassign licenses in Microsoft 365 admin center | License page workflow, group assignment, service plan toggles, and license error handling |
| Microsoft Entra Docs | Enterprise app user and group assignment | Assigning users and groups to enterprise apps, app roles, and default access |
| Microsoft Entra Docs | Role-assignable groups | Cloud role groups, `isAssignableToRole`, role assignment restrictions, P1/P2 requirements, and PIM relationship |
| Microsoft Entra Docs | Microsoft Graph directory, group, role management, and app role assignment APIs | Scripted inventory, group creation, app assignment, and role assignment validation |
| Microsoft 365 Docs | Assign licenses to user accounts | Usage location and license assignment readiness |
| Windows Server AD DS | ActiveDirectory PowerShell module | On-prem group inventory, membership review, and sync scope validation |
| Microsoft Entra Connect Sync / Cloud Sync | Group sync baseline | Makes on-prem security groups available in Microsoft Entra ID |
| Operational practice | Separate groups by purpose | Prevents one group from accidentally controlling licensing, app access, and privileged roles at the same time |

# 08_Map_On_Prem_Groups_To_Cloud_Roles_Licensing_Access_And_App_Assignments_Mental_Model

| Concept | Operational Meaning |
|---|---|
| On-prem AD group | Group mastered in AD DS and synchronized to Microsoft Entra ID |
| Cloud-only group | Group created directly in Microsoft Entra ID |
| Synced group | AD DS group represented in Microsoft Entra ID through Entra Connect Sync or Cloud Sync |
| Security group | Group that can control access, app assignments, Conditional Access scoping, and license assignment |
| Microsoft 365 group | Collaboration group with mailbox, calendar, SharePoint, Teams-backed use cases |
| Dynamic group | Cloud group with membership rules, useful for automation but not supported for role-assignable groups |
| Role-assignable group | Cloud group created with `isAssignableToRole=true`, used to assign Microsoft Entra roles |
| App assignment | Assigning a user or group to an enterprise application and optionally to an app role |
| App role | Permission role exposed by a service principal, such as `User`, `Reader`, `Admin`, or custom SaaS role |
| Group-based licensing | Product license assignment to a group so members receive licenses automatically |
| Usage location | Required user property for license assignment in many Microsoft cloud services |
| Service plan | Individual workload inside a product license, such as Exchange, SharePoint, Teams, or Intune |
| Direct license assignment | License assigned directly to a user, harder to manage at scale |
| Inherited license assignment | License received through group membership |
| Nested group | Group inside another group, not safe to assume for app assignment or role-assignable group behavior |
| Least privilege group | Group scoped to one access purpose and one owner path |
| First rule | Do not reuse the same group for roles, licenses, apps, and CA unless that coupling is intentional |
| Blunt rule | On-prem groups are fine for normal license and app access, but not for Microsoft Entra role assignment |

# 08_Map_On_Prem_Groups_To_Cloud_Roles_Licensing_Access_And_App_Assignments_Planning_Table

| Item | Example | Decision |
|---|---|---|
| Tenant domain | `contoso.onmicrosoft.com` | `<tenant-domain>` |
| Verified domain | `contoso.com` | `<verified-domain>` |
| Tenant ID | `00000000-0000-0000-0000-000000000000` | `<tenant-id>` |
| AD forest | `corp.local` | `<ad-forest>` |
| AD domain | `corp.local` | `<ad-domain>` |
| Sync method | Entra Connect Sync / Cloud Sync | `<sync-method>` |
| Group source | On-prem synced / cloud-only / dynamic | `<group-source>` |
| On-prem groups OU | `OU=Cloud Groups,OU=Groups,DC=corp,DC=local` | `<cloud-groups-ou>` |
| License group prefix | `LIC-M365-` | `<license-group-prefix>` |
| App group prefix | `APP-` | `<app-group-prefix>` |
| Role group prefix | `ROLE-` | `<role-group-prefix>` |
| CA group prefix | `CA-` | `<ca-group-prefix>` |
| Pilot license group | `LIC-M365-BusinessPremium-Pilot` | `<pilot-license-group>` |
| Pilot app group | `APP-Salesforce-Users-Pilot` | `<pilot-app-group>` |
| Role-assignable group | `ROLE-Entra-HelpdeskAdministrator` | `<role-assignable-group>` |
| License SKU | `Microsoft 365 Business Premium` | `<license-sku>` |
| License SKU part number | `SPB` / `ENTERPRISEPACK` | `<sku-part-number>` |
| Service plans disabled | Yammer / Power BI / other | `<disabled-service-plans>` |
| Enterprise app | `Salesforce` | `<enterprise-app-name>` |
| Enterprise app role | `Default Access` / `User` / `Admin` | `<enterprise-app-role>` |
| Required admin role | License Administrator / Cloud Application Administrator / Privileged Role Administrator | `<required-admin-role>` |
| Group owner | IAM team / app owner / license owner | `<group-owner>` |
| Evidence folder | `C:\CloudAccess-Mapping` | `<evidence-path>` |
| Rollback plan | Remove assignments, restore group membership, remove licenses | `<rollback-plan>` |

# 08_Map_On_Prem_Groups_To_Cloud_Roles_Licensing_Access_And_App_Assignments_Configuration_Checklist

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm sync baseline is healthy | Admin Workstation | `Review task 04 and task 05 evidence` | Users and normal groups sync correctly |
| 2 | Create evidence folder | Admin Workstation | `New-Item -ItemType Directory -Force -Path C:\CloudAccess-Mapping` | Evidence folder exists |
| 3 | Connect to Microsoft Graph | Admin Workstation | `Connect-MgGraph -Scopes "Directory.ReadWrite.All","Group.ReadWrite.All","RoleManagement.ReadWrite.Directory","AppRoleAssignment.ReadWrite.All","Application.ReadWrite.All","User.ReadWrite.All","Organization.Read.All"` | Graph session connects |
| 4 | Export tenant domains | Admin Workstation | `Get-MgDomain \| Export-Csv C:\CloudAccess-Mapping\tenant-domains.csv -NoTypeInformation` | Tenant domains are documented |
| 5 | Export subscribed SKUs | Admin Workstation | `Get-MgSubscribedSku \| Export-Csv C:\CloudAccess-Mapping\subscribed-skus.csv -NoTypeInformation` | Available license products are documented |
| 6 | Export cloud users and usage location | Admin Workstation | `Get-MgUser -All -Property UserPrincipalName,UsageLocation` | Usage location readiness is known |
| 7 | Export existing groups | Admin Workstation | `Get-MgGroup -All \| Export-Csv C:\CloudAccess-Mapping\groups-before.csv -NoTypeInformation` | Existing cloud and synced groups are documented |
| 8 | Export on-prem group inventory | Domain Controller / Admin Workstation | `Run On_Prem_Group_Inventory_Skeleton` | AD group source state is documented |
| 9 | Confirm group sync scope | Sync Server / Admin Workstation | `Review Entra Connect OU filtering or Cloud Sync scope` | Intended groups are in sync scope |
| 10 | Create on-prem license groups if using synced groups | Domain Controller | `New-ADGroup -Name "LIC-M365-BusinessPremium-Pilot" -GroupScope Global -GroupCategory Security` | License group exists in AD DS |
| 11 | Create on-prem app access groups if using synced groups | Domain Controller | `New-ADGroup -Name "APP-Salesforce-Users-Pilot" -GroupScope Global -GroupCategory Security` | App group exists in AD DS |
| 12 | Add pilot users to on-prem groups | Domain Controller | `Add-ADGroupMember -Identity "<group>" -Members "<user>"` | Pilot membership exists |
| 13 | Run sync cycle if Entra Connect Sync is used | Sync Server | `Start-ADSyncSyncCycle -PolicyType Delta` | New groups sync to cloud |
| 14 | Confirm synced groups in Microsoft Entra ID | Admin Workstation | `Get-MgGroup -Filter "displayName eq 'LIC-M365-BusinessPremium-Pilot'"` | Synced group appears in cloud |
| 15 | Create cloud-only groups where needed | Admin Workstation | `New-MgGroup -MailEnabled:$false -SecurityEnabled:$true` | Cloud group exists |
| 16 | Create role-assignable groups as cloud-only groups | Admin Workstation | `New-MgGroup -IsAssignableToRole:$true` | Role-assignable group exists |
| 17 | Assign Microsoft Entra roles to role-assignable group | Admin Workstation | `New-MgRoleManagementDirectoryRoleAssignment` | Group receives intended Entra role |
| 18 | Configure PIM for privileged group if available | Entra Admin Center | `Identity Governance > Privileged Identity Management > Groups` | Privileged group can be eligible instead of standing |
| 19 | Configure group-based licensing in Microsoft 365 admin center | Microsoft 365 Admin Center | `Billing > Licenses > Product > Assign licenses` | License is assigned to group |
| 20 | Disable unwanted service plans if required | Microsoft 365 Admin Center | `Turn apps and services on or off` | Product service plan scope matches rollout |
| 21 | Reprocess licensing errors if needed | Microsoft 365 Admin Center | `Licenses > Errors & issues > Reprocess` | License errors are retried after correction |
| 22 | Assign enterprise app access to group | Entra Admin Center | `Enterprise apps > App > Users and groups > Add user/group` | Group is assigned to application |
| 23 | Select app role if exposed | Entra Admin Center | `Select role` | Group receives intended app role |
| 24 | Validate app requires assignment if intended | Entra Admin Center | `Enterprise app > Properties > Assignment required` | Only assigned users can access app |
| 25 | Validate pilot user license | Admin Workstation | `Get-MgUserLicenseDetail -UserId "<upn>"` | User receives expected inherited license |
| 26 | Validate pilot user app assignment | Admin Workstation | `Check service principal app role assignments` | User or group has expected app assignment |
| 27 | Validate role membership | Admin Workstation | `Get-MgRoleManagementDirectoryRoleAssignment` | Role assignment exists for role group |
| 28 | Validate user sign-in and app launch | Pilot User Workstation | `https://myapps.microsoft.com` | Assigned app appears and launches |
| 29 | Validate Conditional Access scope alignment | Entra Admin Center | `Review CA groups from task 07` | CA groups are separate from license/app groups unless intentional |
| 30 | Document group owners and purpose | Operator | `Record owner, source, assignment type, approval path, rollback` | Access model is accountable |
| 31 | Expand from pilot to production | Operator | `Add production users or groups in batches` | Access expands safely |

# 08_Map_On_Prem_Groups_To_Cloud_Roles_Licensing_Access_And_App_Assignments_Group_Design_Map

| Group Type | Example | Source | Used For | Avoid Using For |
|---|---|---|---|---|
| License group | `LIC-M365-BusinessPremium-Users` | Synced or cloud-only security group | Microsoft 365 product license assignment | Privileged roles |
| App access group | `APP-Salesforce-Users` | Synced or cloud-only security group | Enterprise app access and app roles | Tenant admin roles |
| Privileged role group | `ROLE-Entra-HelpdeskAdministrator` | Cloud-only role-assignable group | Microsoft Entra admin role assignment | Regular licensing or broad app access |
| Conditional Access group | `CA-MFA-Pilot-Users` | Synced or cloud-only security group | CA policy targeting | Licensing unless intentional |
| Exclusion group | `CA-Excluded-BreakGlass` | Cloud-only security group | Emergency access exclusions | Daily user access |
| Dynamic license group | `DYN-LIC-M365-Department-Sales` | Cloud-only dynamic group | Automated licensing by attributes | Role-assignable group |
| App admin group | `APP-Salesforce-Admins` | Cloud-only or synced security group | SaaS app admin app role | Microsoft Entra directory roles unless role-assignable |
| Department group | `DEPT-Finance-Users` | Synced from AD DS | Reference group or app access input | Direct admin role assignment |
| Mail-enabled group | `M365-Team-Finance` | Microsoft 365 group | Collaboration and Teams | Security-sensitive role assignment |
| Break glass group | `CA-Excluded-BreakGlass` | Cloud-only assigned group | CA exclusions and emergency handling | Licenses, apps, normal user access |

# 08_Map_On_Prem_Groups_To_Cloud_Roles_Licensing_Access_And_App_Assignments_Precheck_Skeleton

```powershell
# Run from admin workstation with Microsoft Graph PowerShell.
# Purpose: capture tenant, license, user, group, app, and role baseline before assignments.

$EvidencePath = "C:\CloudAccess-Mapping"
New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\01-precheck-transcript.txt"

Import-Module Microsoft.Graph.Authentication
Import-Module Microsoft.Graph.Identity.DirectoryManagement
Import-Module Microsoft.Graph.Users
Import-Module Microsoft.Graph.Groups
Import-Module Microsoft.Graph.Applications
Import-Module Microsoft.Graph.Identity.Governance

Connect-MgGraph -Scopes `
  "Directory.Read.All",
  "Organization.Read.All",
  "User.Read.All",
  "Group.Read.All",
  "Application.Read.All",
  "AppRoleAssignment.ReadWrite.All",
  "RoleManagement.Read.Directory"

Get-MgContext |
  Format-List * |
  Out-File "$EvidencePath\graph-context.txt"

# Tenant domains.
Get-MgDomain |
  Select-Object Id,IsVerified,IsDefault,AuthenticationType |
  Export-Csv "$EvidencePath\tenant-domains.csv" -NoTypeInformation

# Subscribed SKUs.
Get-MgSubscribedSku |
  Select-Object SkuId,SkuPartNumber,ConsumedUnits,PrepaidUnits,AppliesTo |
  Export-Csv "$EvidencePath\subscribed-skus.csv" -NoTypeInformation

Get-MgSubscribedSku |
  ConvertTo-Json -Depth 10 |
  Out-File "$EvidencePath\subscribed-skus-full.json"

# Users and usage location.
Get-MgUser -All `
  -Property Id,DisplayName,UserPrincipalName,AccountEnabled,UserType,UsageLocation,OnPremisesSyncEnabled,OnPremisesImmutableId |
  Select-Object Id,DisplayName,UserPrincipalName,AccountEnabled,UserType,UsageLocation,OnPremisesSyncEnabled,OnPremisesImmutableId |
  Export-Csv "$EvidencePath\users-usage-location-before.csv" -NoTypeInformation

# Groups.
Get-MgGroup -All `
  -Property Id,DisplayName,Mail,MailEnabled,SecurityEnabled,GroupTypes,OnPremisesSyncEnabled,IsAssignableToRole,MembershipRule |
  Select-Object Id,DisplayName,Mail,MailEnabled,SecurityEnabled,GroupTypes,OnPremisesSyncEnabled,IsAssignableToRole,MembershipRule |
  Export-Csv "$EvidencePath\groups-before.csv" -NoTypeInformation

# Service principals.
Get-MgServicePrincipal -All `
  -Property Id,AppId,DisplayName,AccountEnabled,AppRoles |
  Select-Object Id,AppId,DisplayName,AccountEnabled |
  Export-Csv "$EvidencePath\service-principals-before.csv" -NoTypeInformation

# Directory role definitions.
Get-MgRoleManagementDirectoryRoleDefinition -All |
  Select-Object Id,DisplayName,Description,IsEnabled |
  Export-Csv "$EvidencePath\directory-role-definitions.csv" -NoTypeInformation

# Directory role assignments.
Get-MgRoleManagementDirectoryRoleAssignment -All |
  Select-Object Id,PrincipalId,RoleDefinitionId,DirectoryScopeId |
  Export-Csv "$EvidencePath\directory-role-assignments-before.csv" -NoTypeInformation

Disconnect-MgGraph

Stop-Transcript
```

# 08_Map_On_Prem_Groups_To_Cloud_Roles_Licensing_Access_And_App_Assignments_On_Prem_Group_Inventory_Skeleton

```powershell
# Run from a domain controller or admin workstation with RSAT AD tools.
# Purpose: inventory on-prem groups intended for sync, licensing, app assignment, or CA targeting.

$EvidencePath = "C:\CloudAccess-Mapping"
$CloudGroupsOu = "OU=Cloud Groups,OU=Groups,DC=corp,DC=local"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\02-on-prem-group-inventory-transcript.txt"

Import-Module ActiveDirectory

# Export target cloud access groups.
Get-ADGroup -SearchBase $CloudGroupsOu -Filter * `
  -Properties Description,ManagedBy,mail,proxyAddresses,GroupCategory,GroupScope,whenCreated,whenChanged |
  Select-Object `
    Name,
    SamAccountName,
    DistinguishedName,
    GroupCategory,
    GroupScope,
    Description,
    ManagedBy,
    mail,
    whenCreated,
    whenChanged,
    @{Name="proxyAddresses";Expression={($_.proxyAddresses -join ";")}} |
  Export-Csv "$EvidencePath\ad-cloud-access-groups.csv" -NoTypeInformation

# Export group membership detail.
$Groups = Get-ADGroup -SearchBase $CloudGroupsOu -Filter * -Properties Members

$MembershipRows = foreach ($Group in $Groups) {
  $Members = Get-ADGroupMember -Identity $Group.DistinguishedName -Recursive -ErrorAction SilentlyContinue

  foreach ($Member in $Members) {
    [PSCustomObject]@{
      GroupName = $Group.Name
      GroupSamAccountName = $Group.SamAccountName
      GroupDistinguishedName = $Group.DistinguishedName
      MemberName = $Member.Name
      MemberSamAccountName = $Member.SamAccountName
      MemberObjectClass = $Member.objectClass
      MemberDistinguishedName = $Member.DistinguishedName
    }
  }
}

$MembershipRows |
  Export-Csv "$EvidencePath\ad-cloud-access-group-membership-recursive.csv" -NoTypeInformation

# Detect nested groups.
$NestedRows = foreach ($Group in $Groups) {
  $DirectMembers = Get-ADGroupMember -Identity $Group.DistinguishedName -ErrorAction SilentlyContinue

  foreach ($Member in $DirectMembers | Where-Object { $_.objectClass -eq "group" }) {
    [PSCustomObject]@{
      ParentGroup = $Group.Name
      ParentGroupDn = $Group.DistinguishedName
      NestedGroup = $Member.Name
      NestedGroupDn = $Member.DistinguishedName
      Issue = "Nested group detected"
    }
  }
}

$NestedRows |
  Export-Csv "$EvidencePath\ad-cloud-access-nested-groups.csv" -NoTypeInformation

# Export AD users with usage-relevant identity values.
Get-ADUser -Filter * -Properties UserPrincipalName,mail,Enabled,Department,Title,Company |
  Select-Object SamAccountName,Name,Enabled,UserPrincipalName,mail,Department,Title,Company,DistinguishedName |
  Export-Csv "$EvidencePath\ad-users-for-group-mapping.csv" -NoTypeInformation

Stop-Transcript
```

# 08_Map_On_Prem_Groups_To_Cloud_Roles_Licensing_Access_And_App_Assignments_Cloud_Group_Baseline_Skeleton

```powershell
# Run from admin workstation with Microsoft Graph PowerShell.
# Purpose: create or validate cloud-only security groups used for cloud assignments.

$EvidencePath = "C:\CloudAccess-Mapping"

$GroupsToCreate = @(
  @{
    DisplayName = "LIC-M365-BusinessPremium-Pilot"
    MailNickname = "lic-m365-businesspremium-pilot"
    Description = "Pilot group for Microsoft 365 Business Premium group-based licensing"
  },
  @{
    DisplayName = "APP-Salesforce-Users-Pilot"
    MailNickname = "app-salesforce-users-pilot"
    Description = "Pilot group for Salesforce enterprise app access"
  },
  @{
    DisplayName = "CA-MFA-Pilot-Users"
    MailNickname = "ca-mfa-pilot-users"
    Description = "Pilot group for Conditional Access MFA policy targeting"
  }
)

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\03-cloud-group-baseline-transcript.txt"

Import-Module Microsoft.Graph.Groups
Import-Module Microsoft.Graph.Users

Connect-MgGraph -Scopes "Group.ReadWrite.All","Directory.ReadWrite.All","User.Read.All"

foreach ($GroupSpec in $GroupsToCreate) {
  $ExistingGroup = Get-MgGroup -Filter "displayName eq '$($GroupSpec.DisplayName)'" -ErrorAction SilentlyContinue

  if (-not $ExistingGroup) {
    New-MgGroup `
      -DisplayName $GroupSpec.DisplayName `
      -Description $GroupSpec.Description `
      -MailEnabled:$false `
      -MailNickname $GroupSpec.MailNickname `
      -SecurityEnabled:$true
  }
}

# Example pilot user membership.
$PilotUserUpn = "pilot.user@contoso.com"
$PilotUser = Get-MgUser -UserId $PilotUserUpn -ErrorAction SilentlyContinue

if ($PilotUser) {
  foreach ($GroupSpec in $GroupsToCreate) {
    $Group = Get-MgGroup -Filter "displayName eq '$($GroupSpec.DisplayName)'"

    New-MgGroupMember `
      -GroupId $Group.Id `
      -DirectoryObjectId $PilotUser.Id `
      -ErrorAction SilentlyContinue
  }
}

# Export created groups.
foreach ($GroupSpec in $GroupsToCreate) {
  Get-MgGroup -Filter "displayName eq '$($GroupSpec.DisplayName)'" `
    -Property Id,DisplayName,Description,MailEnabled,SecurityEnabled,GroupTypes,OnPremisesSyncEnabled,IsAssignableToRole |
    Select-Object Id,DisplayName,Description,MailEnabled,SecurityEnabled,GroupTypes,OnPremisesSyncEnabled,IsAssignableToRole |
    Export-Csv "$EvidencePath\$($GroupSpec.DisplayName)-group-validation.csv" -NoTypeInformation
}

Disconnect-MgGraph

Stop-Transcript
```

# 08_Map_On_Prem_Groups_To_Cloud_Roles_Licensing_Access_And_App_Assignments_Group_Based_Licensing_Skeleton

```text
# Microsoft 365 admin center group-based licensing runbook.

1. Confirm target group exists:
   - LIC-M365-BusinessPremium-Pilot

2. Confirm users have Usage location populated.
   - Missing usage location commonly causes license assignment errors.

3. Sign in to Microsoft 365 admin center.

4. Go to:
   Billing > Licenses

5. Select the target product:
   - Microsoft 365 Business Premium
   - Office 365 E3
   - Enterprise Mobility + Security
   - Other approved product

6. Select:
   Assign licenses

7. Search for and select the target group:
   LIC-M365-BusinessPremium-Pilot

8. Select:
   Turn apps and services on or off

9. Disable service plans that should not be enabled yet.
   Example:
   - Disable Yammer/Viva Engage if not ready
   - Disable Power BI if not licensed or governed
   - Disable Intune if endpoint rollout is not ready

10. Select:
    Assign

11. Open:
    Errors & issues

12. Review failures:
    - No available licenses
    - Conflicting service plans
    - Invalid usage location

13. Correct failures.

14. Reprocess failed users after correction.

15. Validate a pilot user:
    - User appears licensed
    - Correct service plans are enabled
    - Microsoft 365 apps activate as expected
    - Exchange/Teams/SharePoint access behaves as planned

16. Document:
    - product
    - group
    - service plans enabled
    - service plans disabled
    - users with errors
    - owner
    - rollback steps
```

```powershell
# Graph validation for group-based license readiness and user license state.
# This skeleton validates inventory. License assignment itself may be handled through Microsoft 365 admin center.

$EvidencePath = "C:\CloudAccess-Mapping"
$LicenseGroupName = "LIC-M365-BusinessPremium-Pilot"
$PilotUserUpn = "pilot.user@contoso.com"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\04-group-based-licensing-validation-transcript.txt"

Import-Module Microsoft.Graph.Users
Import-Module Microsoft.Graph.Groups
Import-Module Microsoft.Graph.Identity.DirectoryManagement

Connect-MgGraph -Scopes "User.ReadWrite.All","Group.Read.All","Directory.Read.All","Organization.Read.All"

# Export subscribed SKUs.
Get-MgSubscribedSku |
  Select-Object SkuId,SkuPartNumber,ConsumedUnits,PrepaidUnits |
  Export-Csv "$EvidencePath\subscribed-skus-before-license-assignment.csv" -NoTypeInformation

# Validate group.
$LicenseGroup = Get-MgGroup -Filter "displayName eq '$LicenseGroupName'" `
  -Property Id,DisplayName,SecurityEnabled,OnPremisesSyncEnabled

$LicenseGroup |
  Select-Object Id,DisplayName,SecurityEnabled,OnPremisesSyncEnabled |
  Export-Csv "$EvidencePath\$LicenseGroupName-validation.csv" -NoTypeInformation

# Validate group members.
Get-MgGroupMember -GroupId $LicenseGroup.Id -All |
  Select-Object Id,AdditionalProperties |
  Export-Csv "$EvidencePath\$LicenseGroupName-members.csv" -NoTypeInformation

# Validate usage location for pilot user.
$PilotUser = Get-MgUser `
  -UserId $PilotUserUpn `
  -Property Id,DisplayName,UserPrincipalName,UsageLocation,AccountEnabled,OnPremisesSyncEnabled

$PilotUser |
  Select-Object Id,DisplayName,UserPrincipalName,UsageLocation,AccountEnabled,OnPremisesSyncEnabled |
  Export-Csv "$EvidencePath\$PilotUserUpn-usage-location.csv" -NoTypeInformation

# Set usage location only if missing and approved.
if (-not $PilotUser.UsageLocation) {
  Update-MgUser -UserId $PilotUser.Id -UsageLocation "US"
}

# Validate assigned licenses and service plans after group-based assignment.
Get-MgUserLicenseDetail -UserId $PilotUser.Id |
  ConvertTo-Json -Depth 10 |
  Out-File "$EvidencePath\$PilotUserUpn-license-details-after.json"

Disconnect-MgGraph

Stop-Transcript
```

# 08_Map_On_Prem_Groups_To_Cloud_Roles_Licensing_Access_And_App_Assignments_Role_Assignable_Group_Skeleton

```text
# Role-assignable group runbook.

Important rules:
1. Role-assignable groups must be created as role-assignable when they are created.
2. You cannot convert an existing normal group into a role-assignable group.
3. On-premises groups cannot be assigned Microsoft Entra roles.
4. Membership must be assigned, not dynamic.
5. Use PIM for eligible access when Entra ID P2 is available.
6. Do not use the same group for licensing or normal app access.

Portal steps:
1. Sign in to Microsoft Entra admin center.
2. Go to:
   Identity > Groups > All groups > New group
3. Group type:
   Security
4. Group name:
   ROLE-Entra-HelpdeskAdministrator
5. Microsoft Entra roles can be assigned to the group:
   Yes
6. Membership type:
   Assigned
7. Add owners:
   IAM owner group or approved owner
8. Add members:
   Approved pilot admin only
9. Create group.
10. Go to:
    Identity > Roles & admins
11. Select role:
    Helpdesk Administrator
12. Add assignment:
    ROLE-Entra-HelpdeskAdministrator
13. If PIM is available:
    Make group or role assignment eligible instead of permanent.
14. Validate:
    Member can perform intended task.
    Member cannot perform unrelated privileged task.
```

```powershell
# Create a role-assignable group and assign a Microsoft Entra role.
# Run with Privileged Role Administrator or equivalent.
# Use carefully. This creates privileged access plumbing.

$EvidencePath = "C:\CloudAccess-Mapping"
$RoleGroupName = "ROLE-Entra-HelpdeskAdministrator"
$RoleGroupMailNickname = "role-entra-helpdeskadministrator"
$RoleDisplayName = "Helpdesk Administrator"
$PilotAdminUpn = "pilot.admin@contoso.com"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\05-role-assignable-group-transcript.txt"

Import-Module Microsoft.Graph.Groups
Import-Module Microsoft.Graph.Users
Import-Module Microsoft.Graph.Identity.Governance

Connect-MgGraph -Scopes `
  "Group.ReadWrite.All",
  "Directory.ReadWrite.All",
  "RoleManagement.ReadWrite.Directory",
  "User.Read.All"

# Create role-assignable group if it does not exist.
$RoleGroup = Get-MgGroup -Filter "displayName eq '$RoleGroupName'" -ErrorAction SilentlyContinue

if (-not $RoleGroup) {
  $RoleGroup = New-MgGroup `
    -DisplayName $RoleGroupName `
    -Description "Role-assignable group for Helpdesk Administrator role" `
    -MailEnabled:$false `
    -MailNickname $RoleGroupMailNickname `
    -SecurityEnabled:$true `
    -IsAssignableToRole:$true
}

# Add pilot admin as member.
$PilotAdmin = Get-MgUser -UserId $PilotAdminUpn -ErrorAction SilentlyContinue

if ($PilotAdmin) {
  New-MgGroupMember `
    -GroupId $RoleGroup.Id `
    -DirectoryObjectId $PilotAdmin.Id `
    -ErrorAction SilentlyContinue
}

# Find role definition.
$RoleDefinition = Get-MgRoleManagementDirectoryRoleDefinition -All |
  Where-Object { $_.DisplayName -eq $RoleDisplayName }

# Assign role to group at tenant scope if not already assigned.
$ExistingAssignments = Get-MgRoleManagementDirectoryRoleAssignment -All |
  Where-Object {
    $_.PrincipalId -eq $RoleGroup.Id -and
    $_.RoleDefinitionId -eq $RoleDefinition.Id
  }

if (-not $ExistingAssignments) {
  New-MgRoleManagementDirectoryRoleAssignment `
    -PrincipalId $RoleGroup.Id `
    -RoleDefinitionId $RoleDefinition.Id `
    -DirectoryScopeId "/"
}

# Export validation.
Get-MgGroup -GroupId $RoleGroup.Id -Property Id,DisplayName,IsAssignableToRole,SecurityEnabled,GroupTypes |
  Select-Object Id,DisplayName,IsAssignableToRole,SecurityEnabled,GroupTypes |
  Export-Csv "$EvidencePath\$RoleGroupName-validation.csv" -NoTypeInformation

Get-MgRoleManagementDirectoryRoleAssignment -All |
  Where-Object { $_.PrincipalId -eq $RoleGroup.Id } |
  Select-Object Id,PrincipalId,RoleDefinitionId,DirectoryScopeId |
  Export-Csv "$EvidencePath\$RoleGroupName-role-assignments.csv" -NoTypeInformation

Disconnect-MgGraph

Stop-Transcript
```

# 08_Map_On_Prem_Groups_To_Cloud_Roles_Licensing_Access_And_App_Assignments_Enterprise_App_Assignment_Skeleton

```text
# Enterprise app group assignment portal runbook.

1. Confirm app exists in Microsoft Entra Enterprise applications.
2. Confirm target group exists:
   APP-Salesforce-Users-Pilot

3. Confirm group membership is direct.
   Nested group membership should not be assumed for enterprise app assignment.

4. Sign in to Microsoft Entra admin center.

5. Go to:
   Identity > Applications > Enterprise applications > All applications

6. Open target app:
   Salesforce

7. Open:
   Users and groups

8. Select:
   Add user/group

9. Select:
   APP-Salesforce-Users-Pilot

10. Select app role:
    Default Access
    Or a specific app role such as User, Reader, Analyst, or Admin

11. Select:
    Assign

12. Optional:
    Open Properties.
    Set Assignment required?
    Yes, if only assigned users should access the app.

13. Validate:
    - Pilot user can see app in My Apps
    - Pilot user can launch app
    - Unassigned user cannot access app if assignment is required
    - App role maps correctly in the SaaS app

14. Document:
    - app name
    - service principal object ID
    - group assigned
    - app role
    - assignment required state
    - owner
    - rollback
```

```powershell
# Assign a group to an enterprise app app role with Microsoft Graph.
# Use after validating app roles and target group.

$EvidencePath = "C:\CloudAccess-Mapping"
$AppDisplayName = "Salesforce"
$AppGroupName = "APP-Salesforce-Users-Pilot"
$AppRoleDisplayName = "Default Access"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\06-enterprise-app-assignment-transcript.txt"

Import-Module Microsoft.Graph.Applications
Import-Module Microsoft.Graph.Groups

Connect-MgGraph -Scopes `
  "Application.ReadWrite.All",
  "AppRoleAssignment.ReadWrite.All",
  "Directory.Read.All",
  "Group.Read.All"

# Get service principal.
$ServicePrincipal = Get-MgServicePrincipal -Filter "displayName eq '$AppDisplayName'" -Property Id,DisplayName,AppRoles

# Get target group.
$AppGroup = Get-MgGroup -Filter "displayName eq '$AppGroupName'" -Property Id,DisplayName

# Determine app role.
if ($AppRoleDisplayName -eq "Default Access") {
  $AppRoleId = [Guid]"00000000-0000-0000-0000-000000000000"
}
else {
  $AppRole = $ServicePrincipal.AppRoles |
    Where-Object { $_.DisplayName -eq $AppRoleDisplayName -and $_.AllowedMemberTypes -contains "User" }

  $AppRoleId = $AppRole.Id
}

# Create group app role assignment.
New-MgGroupAppRoleAssignment `
  -GroupId $AppGroup.Id `
  -PrincipalId $AppGroup.Id `
  -ResourceId $ServicePrincipal.Id `
  -AppRoleId $AppRoleId `
  -ErrorAction SilentlyContinue

# Export app assignments.
Get-MgServicePrincipalAppRoleAssignedTo -ServicePrincipalId $ServicePrincipal.Id -All |
  Select-Object Id,PrincipalDisplayName,PrincipalId,PrincipalType,ResourceDisplayName,AppRoleId |
  Export-Csv "$EvidencePath\$AppDisplayName-app-role-assignments.csv" -NoTypeInformation

Disconnect-MgGraph

Stop-Transcript
```

# 08_Map_On_Prem_Groups_To_Cloud_Roles_Licensing_Access_And_App_Assignments_Access_Validation_Skeleton

```powershell
# Run from admin workstation with Microsoft Graph PowerShell.
# Purpose: validate final group, license, app, and role state for a pilot user.

$EvidencePath = "C:\CloudAccess-Mapping"
$PilotUserUpn = "pilot.user@contoso.com"
$PilotAdminUpn = "pilot.admin@contoso.com"
$LicenseGroupName = "LIC-M365-BusinessPremium-Pilot"
$AppGroupName = "APP-Salesforce-Users-Pilot"
$RoleGroupName = "ROLE-Entra-HelpdeskAdministrator"
$AppDisplayName = "Salesforce"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\07-access-validation-transcript.txt"

Import-Module Microsoft.Graph.Users
Import-Module Microsoft.Graph.Groups
Import-Module Microsoft.Graph.Applications
Import-Module Microsoft.Graph.Identity.Governance

Connect-MgGraph -Scopes `
  "User.Read.All",
  "Group.Read.All",
  "Directory.Read.All",
  "Application.Read.All",
  "RoleManagement.Read.Directory"

# Users.
$PilotUser = Get-MgUser -UserId $PilotUserUpn -Property Id,DisplayName,UserPrincipalName,UsageLocation,AccountEnabled
$PilotAdmin = Get-MgUser -UserId $PilotAdminUpn -Property Id,DisplayName,UserPrincipalName,UsageLocation,AccountEnabled

# Groups.
$LicenseGroup = Get-MgGroup -Filter "displayName eq '$LicenseGroupName'"
$AppGroup = Get-MgGroup -Filter "displayName eq '$AppGroupName'"
$RoleGroup = Get-MgGroup -Filter "displayName eq '$RoleGroupName'" -Property Id,DisplayName,IsAssignableToRole

# Membership validation.
$ValidationRows = @()

foreach ($Group in @($LicenseGroup,$AppGroup,$RoleGroup)) {
  if ($Group) {
    $Members = Get-MgGroupMember -GroupId $Group.Id -All

    $ValidationRows += [PSCustomObject]@{
      GroupDisplayName = $Group.DisplayName
      GroupId = $Group.Id
      MemberCount = ($Members | Measure-Object).Count
      PilotUserMember = [bool]($Members | Where-Object { $_.Id -eq $PilotUser.Id })
      PilotAdminMember = [bool]($Members | Where-Object { $_.Id -eq $PilotAdmin.Id })
    }
  }
}

$ValidationRows |
  Export-Csv "$EvidencePath\pilot-group-membership-validation.csv" -NoTypeInformation

# License validation.
Get-MgUserLicenseDetail -UserId $PilotUser.Id |
  ConvertTo-Json -Depth 10 |
  Out-File "$EvidencePath\$PilotUserUpn-license-details.json"

# Enterprise app assignment validation.
$ServicePrincipal = Get-MgServicePrincipal -Filter "displayName eq '$AppDisplayName'" -Property Id,DisplayName,AppRoles

Get-MgServicePrincipalAppRoleAssignedTo -ServicePrincipalId $ServicePrincipal.Id -All |
  Select-Object Id,PrincipalDisplayName,PrincipalId,PrincipalType,ResourceDisplayName,AppRoleId |
  Export-Csv "$EvidencePath\$AppDisplayName-app-assignments-validation.csv" -NoTypeInformation

# Role assignment validation.
Get-MgRoleManagementDirectoryRoleAssignment -All |
  Where-Object { $_.PrincipalId -eq $RoleGroup.Id } |
  Select-Object Id,PrincipalId,RoleDefinitionId,DirectoryScopeId |
  Export-Csv "$EvidencePath\$RoleGroupName-role-assignment-validation.csv" -NoTypeInformation

Disconnect-MgGraph

Stop-Transcript
```

```text
# Manual access validation.

License validation:
1. Sign in as pilot.user@contoso.com.
2. Open Microsoft 365 portal.
3. Confirm assigned apps are available.
4. Confirm disabled service plans are not available.
5. Confirm Office activation if Microsoft 365 Apps license is included.

App validation:
1. Go to https://myapps.microsoft.com.
2. Confirm assigned enterprise app appears.
3. Launch the app.
4. Confirm assigned role inside the SaaS app.
5. Test unassigned user access if assignment is required.

Role validation:
1. Sign in as pilot.admin@contoso.com.
2. Open Microsoft Entra admin center.
3. Confirm the admin can perform only the intended role tasks.
4. Confirm the admin cannot perform unrelated privileged tasks.
5. If PIM is used, activate the group or role assignment and retest.

Documentation:
1. Record group name.
2. Record source, synced or cloud-only.
3. Record assignment target.
4. Record owner.
5. Record validation result.
6. Record rollback command or portal path.
```

# 08_Map_On_Prem_Groups_To_Cloud_Roles_Licensing_Access_And_App_Assignments_Verification_Commands

| Command | Purpose | Healthy Output |
|---|---|---|
| `Get-ADGroup -SearchBase "<cloud-groups-ou>" -Filter *` | Lists on-prem cloud access groups | Intended AD groups appear |
| `Get-ADGroupMember -Identity "<group>" -Recursive` | Validates AD group membership | Intended members appear |
| `Start-ADSyncSyncCycle -PolicyType Delta` | Syncs new on-prem groups to cloud | Delta sync starts |
| `Get-MgGroup -Filter "displayName eq '<group-name>'"` | Confirms cloud group exists | Group object returns |
| `Get-MgGroupMember -GroupId <group-id>` | Confirms cloud group membership | Intended direct members appear |
| `Get-MgSubscribedSku` | Lists available license SKUs | Product SKU and available count are visible |
| `Get-MgUser -UserId "<upn>" -Property UsageLocation` | Confirms usage location | Usage location is populated |
| `Get-MgUserLicenseDetail -UserId "<user-id>"` | Validates user license assignment | Expected license appears |
| `Get-MgServicePrincipal -Filter "displayName eq '<app-name>'"` | Finds enterprise app service principal | Service principal returns |
| `Get-MgServicePrincipalAppRoleAssignedTo -ServicePrincipalId <sp-id>` | Lists app role assignments | Target group assignment appears |
| `Get-MgRoleManagementDirectoryRoleDefinition` | Lists Entra role definitions | Intended role definition appears |
| `Get-MgRoleManagementDirectoryRoleAssignment` | Lists role assignments | Role group assignment appears |
| `Get-MgGroup -GroupId <id> -Property IsAssignableToRole` | Confirms role-assignable group state | `IsAssignableToRole` is `True` for role group |
| `https://myapps.microsoft.com` | Validates app launch for user | Assigned app appears |
| `Microsoft 365 admin center > Billing > Licenses` | Validates group-based license assignment | Product assigned to group |
| `Microsoft 365 admin center > Licenses > Errors & issues` | Checks license assignment failures | No unresolved errors |

# 08_Map_On_Prem_Groups_To_Cloud_Roles_Licensing_Access_And_App_Assignments_Rollback

```text
# Group-based licensing rollback.

1. Go to Microsoft 365 admin center.
2. Open:
   Billing > Licenses
3. Select the affected product.
4. Select the group assignment.
5. Remove the group license assignment or turn off affected service plans.
6. Review Errors & issues.
7. Validate pilot user license state.
8. Do not delete the group until all downstream assignments are removed.
9. Document removed product, group, time, and affected users.
```

```powershell
# Enterprise app assignment rollback.
# Removes a group app role assignment from an enterprise application.

$EvidencePath = "C:\CloudAccess-Mapping\Rollback"
$AppDisplayName = "Salesforce"
$AppGroupName = "APP-Salesforce-Users-Pilot"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\rollback-enterprise-app-assignment.txt"

Import-Module Microsoft.Graph.Applications
Import-Module Microsoft.Graph.Groups

Connect-MgGraph -Scopes "Application.ReadWrite.All","AppRoleAssignment.ReadWrite.All","Directory.Read.All","Group.Read.All"

$ServicePrincipal = Get-MgServicePrincipal -Filter "displayName eq '$AppDisplayName'"
$AppGroup = Get-MgGroup -Filter "displayName eq '$AppGroupName'"

$Assignments = Get-MgGroupAppRoleAssignment -GroupId $AppGroup.Id -All |
  Where-Object { $_.ResourceId -eq $ServicePrincipal.Id }

$Assignments |
  Select-Object Id,PrincipalId,ResourceId,AppRoleId |
  Export-Csv "$EvidencePath\$AppDisplayName-$AppGroupName-assignments-before-removal.csv" -NoTypeInformation

foreach ($Assignment in $Assignments) {
  Remove-MgGroupAppRoleAssignment `
    -GroupId $AppGroup.Id `
    -AppRoleAssignmentId $Assignment.Id
}

Disconnect-MgGraph

Stop-Transcript
```

```powershell
# Role assignment rollback.
# Removes Microsoft Entra role assignment from a role-assignable group.
# Does not delete the group.

$EvidencePath = "C:\CloudAccess-Mapping\Rollback"
$RoleGroupName = "ROLE-Entra-HelpdeskAdministrator"
$RoleDisplayName = "Helpdesk Administrator"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\rollback-role-assignment.txt"

Import-Module Microsoft.Graph.Groups
Import-Module Microsoft.Graph.Identity.Governance

Connect-MgGraph -Scopes "RoleManagement.ReadWrite.Directory","Group.Read.All","Directory.Read.All"

$RoleGroup = Get-MgGroup -Filter "displayName eq '$RoleGroupName'"
$RoleDefinition = Get-MgRoleManagementDirectoryRoleDefinition -All |
  Where-Object { $_.DisplayName -eq $RoleDisplayName }

$Assignments = Get-MgRoleManagementDirectoryRoleAssignment -All |
  Where-Object {
    $_.PrincipalId -eq $RoleGroup.Id -and
    $_.RoleDefinitionId -eq $RoleDefinition.Id
  }

$Assignments |
  Select-Object Id,PrincipalId,RoleDefinitionId,DirectoryScopeId |
  Export-Csv "$EvidencePath\$RoleGroupName-role-assignments-before-removal.csv" -NoTypeInformation

foreach ($Assignment in $Assignments) {
  Remove-MgRoleManagementDirectoryRoleAssignment `
    -UnifiedRoleAssignmentId $Assignment.Id
}

Disconnect-MgGraph

Stop-Transcript
```

```powershell
# Group membership rollback.
# Removes a pilot user from access groups.

$EvidencePath = "C:\CloudAccess-Mapping\Rollback"
$PilotUserUpn = "pilot.user@contoso.com"
$GroupsToRemove = @(
  "LIC-M365-BusinessPremium-Pilot",
  "APP-Salesforce-Users-Pilot",
  "CA-MFA-Pilot-Users"
)

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\rollback-group-membership.txt"

Import-Module Microsoft.Graph.Users
Import-Module Microsoft.Graph.Groups

Connect-MgGraph -Scopes "Group.ReadWrite.All","User.Read.All","Directory.ReadWrite.All"

$PilotUser = Get-MgUser -UserId $PilotUserUpn

foreach ($GroupName in $GroupsToRemove) {
  $Group = Get-MgGroup -Filter "displayName eq '$GroupName'" -ErrorAction SilentlyContinue

  if ($Group) {
    Remove-MgGroupMemberByRef `
      -GroupId $Group.Id `
      -DirectoryObjectId $PilotUser.Id `
      -ErrorAction SilentlyContinue
  }
}

Disconnect-MgGraph

Stop-Transcript
```

# 08_Map_On_Prem_Groups_To_Cloud_Roles_Licensing_Access_And_App_Assignments_Failure_Checks

| Symptom | Likely Cause | Check Command | Fix |
|---|---|---|---|
| Synced group does not appear in Entra ID | Group OU not in sync scope or sync has not run | `Get-ADGroup`; `Start-ADSyncSyncCycle -PolicyType Delta`; `Get-MgGroup` | Add OU to sync scope or run delta sync |
| Group appears but user not included | Membership not synced yet or nested group assumption | `Get-ADGroupMember`; `Get-MgGroupMember` | Use direct membership where required |
| App access fails for group member | Enterprise app assignment does not support nested groups | `Get-MgGroupMember -GroupId <id>` | Add user directly to assigned group |
| License not assigned to user | Usage location missing, no available licenses, or conflicting service plan | `Get-MgUserLicenseDetail`; M365 Licenses Errors & issues | Set usage location, free license, fix conflict, reprocess |
| License group assignment not visible in Entra admin center UI | License assignments are managed in Microsoft 365 admin center UI | Microsoft 365 admin center > Billing > Licenses | Manage group licensing in M365 admin center or API/PowerShell |
| User has wrong service plan | Product service plans not toggled correctly | User license details | Adjust apps and services for group assignment |
| Direct and group licenses conflict | User has direct license plus inherited group license | `Get-MgUserLicenseDetail` | Standardize on group-based assignment where possible |
| Role assignment to synced group fails | On-prem groups cannot be assigned Microsoft Entra roles | Check group `OnPremisesSyncEnabled` | Create cloud-only role-assignable group |
| Existing group cannot become role-assignable | `isAssignableToRole` must be set at group creation | `Get-MgGroup -Property IsAssignableToRole` | Create a new role-assignable group |
| Dynamic group cannot be role-assignable | Role-assignable groups require assigned membership | Group properties | Use assigned membership for role groups |
| Role-assignable group creation fails | Missing Privileged Role Administrator role or Graph permission | Check admin role and Graph scopes | Use proper role and `RoleManagement.ReadWrite.Directory` |
| Privileged access is standing permanently | Role group assigned directly without PIM | Role assignment review | Configure PIM eligible assignment if P2 is available |
| App role assignment fails | Wrong service principal or app role ID | `Get-MgServicePrincipal`; inspect `AppRoles` | Select correct enterprise app and app role |
| App shows Default Access only | Application exposes no custom app roles | Service principal `AppRoles` | Use Default Access or configure app roles in app registration if applicable |
| User cannot see app in My Apps | Assignment missing, app hidden, or user not direct member | App assignments and group membership | Assign correct group and validate direct membership |
| Unassigned user can access app | Assignment required is disabled | Enterprise app Properties | Set Assignment required to Yes if intended |
| Service account gets user license | Broad synced group contains service account | Group membership export | Remove service account or use cleaner group design |
| Conditional Access targets wrong users | CA group reused for licensing or app access | Group design map | Separate CA groups from license and app groups |
| Group owner unknown | No owner governance | Group export | Assign business or IAM owner |
| Rollback removes access too broadly | Shared group used for multiple purposes | Group design review | Use purpose-specific groups before production rollout |

# 08_Map_On_Prem_Groups_To_Cloud_Roles_Licensing_Access_And_App_Assignments_Related_Labs

| Lab                                                                                   | Relationship                                                                                   |
| ------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------- |
| 01_Prepare_On_Prem_AD_DS_DNS_UPN_Suffixes_And_Routable_Domains_For_Cloud.md           | Ensures users have routable sign-in names before group-based assignment                        |
| 02_Validate_Custom_Domain_DNS_Records_For_Azure_And_M365.md                           | Confirms the verified domain used by users receiving licenses and app access                   |
| 03_Prepare_Identity_Synchronization_With_IdFix_And_AD_Object_Cleanup.md               | Cleans users and groups before sync-driven assignment                                          |
| 04_Configure_Entra_Connect_Sync_And_Cloud_Sync_Baseline.md                            | Syncs on-prem users and groups to Microsoft Entra ID                                           |
| 05_Troubleshoot_Identity_Sync_Attribute_Duplicate_Object_And_Connect_Health_Issues.md | Fixes group or user sync problems before access mapping                                        |
| 06_Configure_SSPR_MFA_Authentication_Methods_And_Password_Protection.md               | Ensures users and admins can authenticate after access is assigned                             |
| 07_Configure_Conditional_Access_Baseline_Policies_And_Break_Glass_Exclusions.md       | Uses separate groups for Conditional Access targeting and exclusions                           |
| 09_Troubleshoot_Hybrid_SignIn_MFA_CA_Licensing_And_Admin_Access_Issues.md             | Troubleshoots user access issues caused by licensing, app assignments, roles, sync, MFA, or CA |