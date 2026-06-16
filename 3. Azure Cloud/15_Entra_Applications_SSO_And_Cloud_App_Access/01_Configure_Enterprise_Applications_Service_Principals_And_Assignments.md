# 01_Configure_Enterprise_Applications_Service_Principals_And_Assignments

## Objective

Configure Microsoft Entra Enterprise Applications, service principals, owners, user/group assignments, assignment requirements, visibility, and app access validation.

This workbook focuses on the admin side of an application after it exists in the tenant as an Enterprise Application. It does not focus on building the application registration itself. That is handled in the next workbook.

## Source_Basis

- Project workbook structure baseline from prior Networking, Windows Server, Azure, and Microsoft 365 workbook sets.
- Connected GitHub repository crawl baseline:
  - `jntnt0/Workbook`
  - `jntnt0/microsoft-365-docs`
  - `jntnt0/memdocs`
  - `jntnt0/defender-docs`
- Microsoft Entra administration concepts:
  - Enterprise Applications
  - Service principals
  - Application assignments
  - App roles
  - Owners
  - Assignment required
  - Sign-in logs
  - Audit logs
  - Microsoft Graph validation

## Lab_Context

| Item | Value |
|---|---|
| Tenant | `<tenant-name>.onmicrosoft.com` |
| Admin portal | `https://entra.microsoft.com` |
| Primary admin center | Microsoft Entra admin center |
| Target app name | `<enterprise-app-name>` |
| Test user | `<test-user-upn>` |
| Test group | `<app-access-group-name>` |
| Break-glass exclusion | `<break-glass-user-upn>` |
| PowerShell module | Microsoft Graph PowerShell |
| Graph permission scope | `Application.ReadWrite.All`, `AppRoleAssignment.ReadWrite.All`, `Directory.ReadWrite.All`, `AuditLog.Read.All` |

## Mental_Model

An Enterprise Application is the tenant-local service principal object for an application.

An App Registration is the application definition.

An Enterprise Application is the instance of that application inside a tenant.

For a single-tenant custom app, the app registration and enterprise application usually exist in the same tenant.

For a SaaS app, the app registration usually belongs to the vendor tenant, while your tenant receives a service principal object when the app is added or consented to.

Basic flow:

~~~text
Application registration
        |
        | creates or backs
        v
Service principal / Enterprise application
        |
        | controlled by
        v
Assignments, owners, SSO, provisioning, Conditional Access, logs
~~~

Use Enterprise Applications when managing:

- Who can access the app
- Whether assignment is required
- Which groups or users are assigned
- Whether the app appears in My Apps
- App-specific Conditional Access targeting
- SSO configuration
- Provisioning
- Sign-in and audit review
- Service principal ownership

## Planning_Table

| Decision Area | Recommended Baseline | Notes |
|---|---|---|
| Assignment required | Enabled for business apps | Prevents broad tenant-wide access unless explicitly assigned |
| Assignment method | Group-based assignment | Easier to manage than direct user assignment |
| Group type | Security group or Microsoft 365 group depending app support | Security group is the clean default |
| Owners | At least two named owners | Avoid orphaned apps |
| Break-glass handling | Do not assign break-glass unless required | Break-glass should not depend on SaaS apps |
| App visibility | Hide apps not meant for user launch | Keeps My Apps clean |
| User consent | Do not rely on uncontrolled user consent | Consent is covered in workbook 03 |
| Logging | Review sign-ins and audit logs after configuration | Validate real access behavior |
| Naming | Use clear app and group names | Example: `APP-Salesforce-Users` |
| Review cadence | Quarterly access review for sensitive apps | Covered deeper in workbook 05 |

## Prerequisites

| Requirement | Details |
|---|---|
| Role | Cloud Application Administrator, Application Administrator, Global Administrator, or Privileged Role Administrator depending action |
| License | Basic assignment works broadly; access reviews and advanced governance may require Entra ID P2 |
| Test identity | A non-admin test user |
| Group | A test security group for app assignment |
| Graph PowerShell | Installed and available |
| Change window | Recommended for production apps |
| Existing app | Target Enterprise Application must already exist or be added during the workbook |

## Variables

Replace these placeholders throughout the workbook.

~~~powershell
$TenantId = "<tenant-id>"
$EnterpriseAppName = "<enterprise-app-name>"
$TestUserUpn = "<test-user-upn>"
$AccessGroupName = "<app-access-group-name>"
$OwnerUpn1 = "<owner1-upn>"
$OwnerUpn2 = "<owner2-upn>"
~~~

## Configuration_Checklist

| Step | Task | Admin Center / Portal Path | PowerShell / CLI / Graph | Expected Result |
|---:|---|---|---|---|
| 1 | Sign in to Entra admin center | `https://entra.microsoft.com` | N/A | Admin portal loads successfully |
| 2 | Open Enterprise Applications | Entra admin center > Applications > Enterprise applications | N/A | Enterprise application list appears |
| 3 | Locate target app | Enterprise applications > Search `<enterprise-app-name>` | `Get-MgServicePrincipal` | App service principal is found |
| 4 | Review app overview | Enterprise applications > `<enterprise-app-name>` > Overview | `Get-MgServicePrincipal` | App ID, object ID, homepage, publisher, and sign-in status are reviewed |
| 5 | Confirm whether assignment is required | Enterprise applications > `<enterprise-app-name>` > Properties | `AppRoleAssignmentRequired` | App access behavior is known |
| 6 | Enable assignment required | Properties > Assignment required? > Yes | `Update-MgServicePrincipal` | Only assigned users/groups can access the app |
| 7 | Create or confirm access group | Entra admin center > Identity > Groups | `New-MgGroup` or `Get-MgGroup` | App access group exists |
| 8 | Add test user to access group | Groups > `<app-access-group-name>` > Members | `New-MgGroupMember` | Test user is a member |
| 9 | Assign group to enterprise app | Enterprise app > Users and groups > Add user/group | `New-MgServicePrincipalAppRoleAssignedTo` | Group is assigned to app |
| 10 | Add app owners | Enterprise app > Owners > Add | `New-MgServicePrincipalOwnerByRef` | At least two owners assigned |
| 11 | Configure app visibility | Enterprise app > Properties > Visible to users? | `Tags` or portal setting | App visibility matches intended behavior |
| 12 | Validate user assignment | Enterprise app > Users and groups | `Get-MgServicePrincipalAppRoleAssignedTo` | Assigned users/groups are visible |
| 13 | Test user access | My Apps or app URL | Browser sign-in test | Assigned user can access |
| 14 | Test unassigned user denial | My Apps or app URL | Browser sign-in test | Unassigned user is blocked if assignment required is enabled |
| 15 | Review sign-in logs | Enterprise app > Sign-in logs | `Get-MgAuditLogSignIn` | App sign-ins are recorded |
| 16 | Review audit logs | Enterprise app > Audit logs | `Get-MgAuditLogDirectoryAudit` | Assignment and property changes are recorded |
| 17 | Document final state | Workbook notes / repo | N/A | App configuration is documented |

## Step_01_Connect_To_Microsoft_Graph

Use Microsoft Graph PowerShell for validation and repeatable checks.

~~~powershell
Install-Module Microsoft.Graph -Scope CurrentUser -Force

Connect-MgGraph -TenantId $TenantId -Scopes `
  "Application.ReadWrite.All", `
  "AppRoleAssignment.ReadWrite.All", `
  "Directory.ReadWrite.All", `
  "AuditLog.Read.All"

Get-MgContext
~~~

Expected result:

~~~text
Microsoft Graph connection succeeds.
Tenant ID matches the target tenant.
Required scopes are granted.
~~~

## Step_02_Find_Enterprise_Application

### Portal

| Step | Action |
|---:|---|
| 1 | Go to `https://entra.microsoft.com` |
| 2 | Open Applications |
| 3 | Select Enterprise applications |
| 4 | Search for `<enterprise-app-name>` |
| 5 | Open the target application |

### PowerShell

~~~powershell
$Sp = Get-MgServicePrincipal -Filter "displayName eq '$EnterpriseAppName'"

$Sp | Select-Object DisplayName, Id, AppId, AccountEnabled, AppRoleAssignmentRequired
~~~

Expected result:

~~~text
The target service principal is returned.
The Id value is the Enterprise Application object ID.
The AppId value is the application/client ID.
~~~

If multiple results are returned:

~~~powershell
Get-MgServicePrincipal -Search "displayName:$EnterpriseAppName" -ConsistencyLevel eventual |
Select-Object DisplayName, Id, AppId, PublisherName, AccountEnabled
~~~

## Step_03_Review_Enterprise_Application_Properties

### Portal

| Setting | Portal Path | Baseline |
|---|---|---|
| Enabled for users to sign-in | Enterprise app > Properties | Yes unless intentionally disabled |
| Assignment required | Enterprise app > Properties | Yes for controlled apps |
| Visible to users | Enterprise app > Properties | Yes for user-launched apps, No for backend apps |
| Homepage URL | Enterprise app > Properties | Should point to vendor or internal app |
| Logo | Enterprise app > Properties | Optional |
| Notes | Enterprise app > Properties | Use for ownership / support notes |

### PowerShell

~~~powershell
$Sp | Format-List `
  DisplayName,
  Id,
  AppId,
  AccountEnabled,
  AppRoleAssignmentRequired,
  PublisherName,
  Homepage,
  LoginUrl,
  LogoutUrl,
  Notes,
  Tags
~~~

Expected result:

~~~text
Application properties are visible and match the intended app behavior.
~~~

## Step_04_Enable_Assignment_Required

Assignment required is one of the most important controls on Enterprise Applications.

When enabled, users must be explicitly assigned directly or through group assignment before access is allowed.

### Portal

| Step | Action |
|---:|---|
| 1 | Open Enterprise applications |
| 2 | Select `<enterprise-app-name>` |
| 3 | Select Properties |
| 4 | Set Assignment required? to Yes |
| 5 | Select Save |

### PowerShell

~~~powershell
Update-MgServicePrincipal -ServicePrincipalId $Sp.Id -AppRoleAssignmentRequired:$true

Get-MgServicePrincipal -ServicePrincipalId $Sp.Id |
Select-Object DisplayName, AppRoleAssignmentRequired
~~~

Expected result:

~~~text
AppRoleAssignmentRequired is True.
Only assigned users or groups should be allowed to access the app.
~~~

## Step_05_Create_Or_Confirm_App_Access_Group

Use group-based assignment instead of direct user assignment unless there is a reason not to.

### Naming Examples

| App Type | Group Name |
|---|---|
| Salesforce | `APP-Salesforce-Users` |
| ServiceNow | `APP-ServiceNow-Users` |
| Workday | `APP-Workday-Users` |
| Internal HR app | `APP-HRPortal-Users` |
| Admin-only app | `APP-AdminTool-Users` |

### Portal

| Step | Action |
|---:|---|
| 1 | Go to Identity > Groups |
| 2 | Select New group |
| 3 | Group type: Security |
| 4 | Name: `<app-access-group-name>` |
| 5 | Membership type: Assigned |
| 6 | Create the group |

### PowerShell

~~~powershell
$Group = Get-MgGroup -Filter "displayName eq '$AccessGroupName'"

if (-not $Group) {
    $MailNickName = ($AccessGroupName -replace '[^a-zA-Z0-9]', '').ToLower()

    $Group = New-MgGroup `
      -DisplayName $AccessGroupName `
      -MailEnabled:$false `
      -MailNickname $MailNickName `
      -SecurityEnabled:$true
}

$Group | Select-Object DisplayName, Id, SecurityEnabled, MailEnabled
~~~

Expected result:

~~~text
Security group exists and can be used for app assignment.
~~~

## Step_06_Add_Test_User_To_Access_Group

### Portal

| Step | Action |
|---:|---|
| 1 | Go to Identity > Groups |
| 2 | Open `<app-access-group-name>` |
| 3 | Select Members |
| 4 | Select Add members |
| 5 | Add `<test-user-upn>` |
| 6 | Save |

### PowerShell

~~~powershell
$TestUser = Get-MgUser -UserId $TestUserUpn

New-MgGroupMemberByRef `
  -GroupId $Group.Id `
  -BodyParameter @{
    "@odata.id" = "https://graph.microsoft.com/v1.0/directoryObjects/$($TestUser.Id)"
  }

Get-MgGroupMember -GroupId $Group.Id |
Where-Object { $_.Id -eq $TestUser.Id }
~~~

Expected result:

~~~text
The test user is a member of the app access group.
~~~

## Step_07_Assign_Group_To_Enterprise_Application

Enterprise apps can expose app roles. Some apps have real app roles like Admin, User, Reader, Contributor. Other apps only expose a default access role.

### Find Available App Roles

~~~powershell
$Sp = Get-MgServicePrincipal -ServicePrincipalId $Sp.Id

$Sp.AppRoles | Select-Object DisplayName, Id, Value, IsEnabled, AllowedMemberTypes
~~~

Expected result:

~~~text
Available app roles are listed.
If no meaningful app roles exist, use the default all-zero role ID.
~~~

### Default App Role ID

If the app has no role that applies to users/groups, use:

~~~powershell
$DefaultAppRoleId = "00000000-0000-0000-0000-000000000000"
~~~

### Pick A User/Group Role

~~~powershell
$AppRole = $Sp.AppRoles |
Where-Object {
    $_.IsEnabled -eq $true -and
    $_.AllowedMemberTypes -contains "User"
} |
Select-Object -First 1

if ($AppRole) {
    $AppRoleId = $AppRole.Id
} else {
    $AppRoleId = "00000000-0000-0000-0000-000000000000"
}

$AppRoleId
~~~

### Portal Assignment

| Step | Action |
|---:|---|
| 1 | Open Enterprise applications |
| 2 | Open `<enterprise-app-name>` |
| 3 | Select Users and groups |
| 4 | Select Add user/group |
| 5 | Select Users and groups |
| 6 | Choose `<app-access-group-name>` |
| 7 | Select role if prompted |
| 8 | Select Assign |

### PowerShell Assignment

~~~powershell
New-MgServicePrincipalAppRoleAssignedTo `
  -ServicePrincipalId $Sp.Id `
  -PrincipalId $Group.Id `
  -ResourceId $Sp.Id `
  -AppRoleId $AppRoleId
~~~

Expected result:

~~~text
The access group is assigned to the Enterprise Application.
~~~

## Step_08_Validate_App_Assignments

### Portal

| Step | Action |
|---:|---|
| 1 | Open Enterprise applications |
| 2 | Select `<enterprise-app-name>` |
| 3 | Select Users and groups |
| 4 | Confirm `<app-access-group-name>` appears |
| 5 | Confirm expected role is shown |

### PowerShell

~~~powershell
Get-MgServicePrincipalAppRoleAssignedTo -ServicePrincipalId $Sp.Id -All |
Select-Object PrincipalDisplayName, PrincipalType, AppRoleId, ResourceDisplayName, CreatedDateTime
~~~

Expected result:

~~~text
Assigned group appears in the app role assignment list.
PrincipalType should show Group.
~~~

## Step_09_Add_Enterprise_Application_Owners

Owners are responsible for app lifecycle, assignment review, SSO coordination, vendor changes, and support escalation.

Minimum baseline:

~~~text
At least two owners per business-critical Enterprise Application.
Avoid a single owner.
Avoid ownerless apps.
Use named accounts, not shared admin accounts.
~~~

### Portal

| Step | Action |
|---:|---|
| 1 | Open Enterprise applications |
| 2 | Select `<enterprise-app-name>` |
| 3 | Select Owners |
| 4 | Select Add |
| 5 | Add `<owner1-upn>` and `<owner2-upn>` |
| 6 | Save |

### PowerShell

~~~powershell
$Owner1 = Get-MgUser -UserId $OwnerUpn1
$Owner2 = Get-MgUser -UserId $OwnerUpn2

New-MgServicePrincipalOwnerByRef `
  -ServicePrincipalId $Sp.Id `
  -BodyParameter @{
    "@odata.id" = "https://graph.microsoft.com/v1.0/directoryObjects/$($Owner1.Id)"
  }

New-MgServicePrincipalOwnerByRef `
  -ServicePrincipalId $Sp.Id `
  -BodyParameter @{
    "@odata.id" = "https://graph.microsoft.com/v1.0/directoryObjects/$($Owner2.Id)"
  }
~~~

### Validate Owners

~~~powershell
Get-MgServicePrincipalOwner -ServicePrincipalId $Sp.Id |
Select-Object Id, AdditionalProperties
~~~

Expected result:

~~~text
At least two owners are assigned to the service principal.
~~~

## Step_10_Configure_App_Visibility

Visible to users controls whether the app appears in user-facing launch surfaces such as My Apps.

Use this carefully.

| App Type | Recommended Visibility |
|---|---|
| SaaS user-launched app | Visible |
| SAML app users launch from My Apps | Visible |
| Backend API | Hidden |
| Provisioning-only service principal | Hidden |
| Automation service principal | Hidden |
| Legacy app not directly launched by users | Hidden |

### Portal

| Step | Action |
|---:|---|
| 1 | Open Enterprise applications |
| 2 | Select `<enterprise-app-name>` |
| 3 | Select Properties |
| 4 | Set Visible to users? |
| 5 | Save |

Expected result:

~~~text
The app appears or does not appear in My Apps based on the selected visibility setting.
~~~

## Step_11_Confirm_App_Enabled_State

Disabling an Enterprise Application blocks user sign-in to that application.

### Portal

| Step | Action |
|---:|---|
| 1 | Open Enterprise applications |
| 2 | Select `<enterprise-app-name>` |
| 3 | Select Properties |
| 4 | Confirm Enabled for users to sign-in is Yes |
| 5 | Save if changed |

### PowerShell

~~~powershell
Update-MgServicePrincipal -ServicePrincipalId $Sp.Id -AccountEnabled:$true

Get-MgServicePrincipal -ServicePrincipalId $Sp.Id |
Select-Object DisplayName, AccountEnabled
~~~

Expected result:

~~~text
AccountEnabled is True.
Users can sign in if otherwise authorized.
~~~

## Step_12_Test_Assigned_User_Access

### User Test

| Step | Action | Expected Result |
|---:|---|---|
| 1 | Open private browser window | Clean session |
| 2 | Browse to app URL or My Apps | Sign-in prompt appears |
| 3 | Sign in as `<test-user-upn>` | Authentication succeeds |
| 4 | Launch app | App opens or redirects to configured SSO |
| 5 | Confirm no assignment error appears | Access is allowed |

### Common Successful Result

~~~text
The assigned user can launch or sign in to the application.
The sign-in appears in Entra sign-in logs.
~~~

## Step_13_Test_Unassigned_User_Denial

Use a non-admin user that is not assigned directly and is not in the access group.

| Step | Action | Expected Result |
|---:|---|---|
| 1 | Open private browser window | Clean session |
| 2 | Browse to app URL | Sign-in prompt appears |
| 3 | Sign in as unassigned user | Access is denied |
| 4 | Review error | Error should indicate assignment or access restriction |
| 5 | Review sign-in logs | Failed sign-in should appear |

Expected result:

~~~text
Unassigned user is blocked when Assignment required is enabled.
~~~

If unassigned users are still allowed:

~~~text
Check:
- Assignment required is actually enabled
- User is not in an assigned group
- Dynamic group membership did not include user
- User is not directly assigned
- App is not bypassing Entra authorization through another login path
~~~

## Step_14_Review_Sign_In_Logs

### Portal

| Step | Action |
|---:|---|
| 1 | Open Enterprise applications |
| 2 | Select `<enterprise-app-name>` |
| 3 | Select Sign-in logs |
| 4 | Filter by test user |
| 5 | Review Status, Conditional Access, Authentication Details, Failure Reason |

### PowerShell

~~~powershell
Get-MgAuditLogSignIn -Filter "appDisplayName eq '$EnterpriseAppName'" -Top 10 |
Select-Object CreatedDateTime, UserDisplayName, UserPrincipalName, AppDisplayName, Status, ConditionalAccessStatus
~~~

Expected result:

~~~text
Recent app sign-ins are visible.
Assigned user sign-in should succeed.
Unassigned user sign-in should fail if assignment required is enabled.
~~~

## Step_15_Review_Audit_Logs

### Portal

| Step | Action |
|---:|---|
| 1 | Open Enterprise applications |
| 2 | Select `<enterprise-app-name>` |
| 3 | Select Audit logs |
| 4 | Review property changes |
| 5 | Review assignment changes |
| 6 | Review owner changes |

### PowerShell

~~~powershell
Get-MgAuditLogDirectoryAudit -Top 25 |
Where-Object {
    $_.TargetResources.DisplayName -contains $EnterpriseAppName -or
    $_.ActivityDisplayName -match "application|service principal|app role|owner"
} |
Select-Object ActivityDateTime, ActivityDisplayName, Result, InitiatedBy
~~~

Expected result:

~~~text
Enterprise Application changes are visible in audit logs.
Assignment and owner changes are traceable.
~~~

## Step_16_Document_Final_State

Record the final state in the workbook, ticket, or repo.

| Field | Value |
|---|---|
| Enterprise Application Name | `<enterprise-app-name>` |
| Enterprise Application Object ID | `<service-principal-object-id>` |
| Application ID / Client ID | `<app-id>` |
| Assignment Required | Yes / No |
| Assigned Group | `<app-access-group-name>` |
| App Owners | `<owner1-upn>`, `<owner2-upn>` |
| Visible to Users | Yes / No |
| Enabled for Sign-in | Yes / No |
| Test User | `<test-user-upn>` |
| Validation Date | `<yyyy-mm-dd>` |
| Validated By | `<admin-upn>` |

## Verification_Commands

### Confirm Service Principal

~~~powershell
Get-MgServicePrincipal -ServicePrincipalId $Sp.Id |
Select-Object DisplayName, Id, AppId, AccountEnabled, AppRoleAssignmentRequired
~~~

### Confirm Assignments

~~~powershell
Get-MgServicePrincipalAppRoleAssignedTo -ServicePrincipalId $Sp.Id -All |
Select-Object PrincipalDisplayName, PrincipalType, AppRoleId, ResourceDisplayName
~~~

### Confirm Group Members

~~~powershell
Get-MgGroupMember -GroupId $Group.Id -All |
Select-Object Id, AdditionalProperties
~~~

### Confirm Owners

~~~powershell
Get-MgServicePrincipalOwner -ServicePrincipalId $Sp.Id |
Select-Object Id, AdditionalProperties
~~~

### Confirm Recent Sign-ins

~~~powershell
Get-MgAuditLogSignIn -Filter "appDisplayName eq '$EnterpriseAppName'" -Top 10 |
Select-Object CreatedDateTime, UserPrincipalName, AppDisplayName, Status, ConditionalAccessStatus
~~~

### Confirm Directory Audit Events

~~~powershell
Get-MgAuditLogDirectoryAudit -Top 50 |
Select-Object ActivityDateTime, ActivityDisplayName, Result
~~~

## Failure_Checks

| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| Assigned user cannot access app | Group membership not applied yet | Check group members and sign-in logs | Wait for propagation or assign directly for test |
| User gets assignment required error | User not assigned directly or through group | Enterprise app > Users and groups | Add user/group assignment |
| App does not appear in My Apps | Visible to users is No | Enterprise app > Properties | Set Visible to users to Yes |
| App appears for wrong users | Assignment required is No | Enterprise app > Properties | Set Assignment required to Yes |
| Cannot assign group | Role or license limitation | Check admin role and app role support | Use proper admin role or direct assignment |
| Cannot update service principal | Insufficient permissions | Check Graph scopes and admin role | Reconnect Graph with required scopes |
| Owners missing | App was created without lifecycle ownership | Enterprise app > Owners | Add two owners |
| App sign-in fails after assignment | SSO not configured | Sign-in logs and app SSO settings | Configure SSO in workbook 04 |
| User can access despite no assignment | Assignment required disabled or alternate login path | App properties and vendor login behavior | Enable assignment required and enforce SSO |
| Graph command returns empty result | Wrong display name or object type | Search by AppId or object ID | Use exact service principal ID |
| App role assignment fails | Wrong AppRoleId | Review `$Sp.AppRoles` | Use valid role ID or default zero GUID |
| Duplicate assignment error | Principal already assigned | Get existing assignments | Do not re-add assignment |

## Rollback

Rollback should restore the pre-change state. Do not blindly remove assignments from production apps without confirming impact.

### Remove Group Assignment From Enterprise App

~~~powershell
$Assignments = Get-MgServicePrincipalAppRoleAssignedTo -ServicePrincipalId $Sp.Id -All

$TargetAssignment = $Assignments |
Where-Object {
    $_.PrincipalId -eq $Group.Id
}

foreach ($Assignment in $TargetAssignment) {
    Remove-MgServicePrincipalAppRoleAssignedTo `
      -ServicePrincipalId $Sp.Id `
      -AppRoleAssignmentId $Assignment.Id
}
~~~

Expected result:

~~~text
The group is no longer assigned to the Enterprise Application.
Users who depend on that group lose app access if assignment required is enabled.
~~~

### Disable Assignment Required

Only do this if the intended rollback is to restore open access behavior.

~~~powershell
Update-MgServicePrincipal -ServicePrincipalId $Sp.Id -AppRoleAssignmentRequired:$false

Get-MgServicePrincipal -ServicePrincipalId $Sp.Id |
Select-Object DisplayName, AppRoleAssignmentRequired
~~~

Expected result:

~~~text
AppRoleAssignmentRequired is False.
Users may access the app without explicit assignment, depending on the application and SSO behavior.
~~~

### Disable User Sign-in To App

Use only if the app must be blocked.

~~~powershell
Update-MgServicePrincipal -ServicePrincipalId $Sp.Id -AccountEnabled:$false

Get-MgServicePrincipal -ServicePrincipalId $Sp.Id |
Select-Object DisplayName, AccountEnabled
~~~

Expected result:

~~~text
AccountEnabled is False.
Users cannot sign in to the Enterprise Application.
~~~

### Remove Owner

~~~powershell
$OwnerToRemove = Get-MgUser -UserId $OwnerUpn2

Remove-MgServicePrincipalOwnerByRef `
  -ServicePrincipalId $Sp.Id `
  -DirectoryObjectId $OwnerToRemove.Id
~~~

Expected result:

~~~text
Selected owner is removed.
At least one responsible owner should remain.
~~~

## Security_Baseline

| Control | Baseline |
|---|---|
| Assignment required | Enabled for controlled enterprise apps |
| Assignments | Group-based |
| Direct user assignment | Avoid except for testing or exceptions |
| Owners | Minimum two |
| App enabled state | Enabled only for active apps |
| Visibility | Visible only when users need to launch it |
| Break-glass dependency | Avoid |
| Audit logs | Reviewed after changes |
| Sign-in logs | Reviewed after user test |
| Access review | Required for sensitive apps |
| Privileged apps | Scope tightly and review frequently |

## Admin_Notes

~~~text
Enterprise Application object ID:
Application / Client ID:
Assigned group:
Owner 1:
Owner 2:
Assignment required:
Visible to users:
Enabled for sign-in:
Validated successful assigned-user sign-in:
Validated unassigned-user denial:
Known exceptions:
~~~

## Related_Labs

| Workbook | Relationship |
|---|---|
| 02_Configure_App_Registrations_API_Permissions_Secrets_Certificates_And_Owners.md | Covers the app registration side |
| 03_Configure_Admin_Consent_User_Consent_And_Publisher_Verification_Baseline.md | Covers tenant consent and permission grant baseline |
| 04_Configure_SAML_OIDC_OAuth_SSO_And_Application_Provisioning.md | Covers SSO and provisioning |
| 05_Configure_Conditional_Access_App_Targeting_Session_Controls_And_Access_Reviews.md | Covers CA targeting and access reviews |
| 06_Troubleshoot_Enterprise_App_SSO_Consent_Provisioning_And_API_Permission_Issues.md | Covers troubleshooting workflow |