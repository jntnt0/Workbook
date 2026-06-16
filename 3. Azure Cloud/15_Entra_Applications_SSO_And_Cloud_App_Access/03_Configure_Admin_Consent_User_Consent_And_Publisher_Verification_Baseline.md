# 03_Configure_Admin_Consent_User_Consent_And_Publisher_Verification_Baseline

## Objective

Configure a Microsoft Entra consent baseline for applications by reviewing admin consent, user consent, admin consent workflow, permission classifications, publisher verification, risky permission patterns, and validation logs.

This workbook focuses on tenant consent governance. It does not configure SAML/OIDC SSO or provisioning. Those are handled in the next workbook.

## Lab_Context

| Item | Value |
|---|---|
| Tenant | `<tenant-name>.onmicrosoft.com` |
| Admin portal | `https://entra.microsoft.com` |
| Primary admin center | Microsoft Entra admin center |
| Target app registration | `<app-registration-name>` |
| Target enterprise app | `<enterprise-app-name>` |
| Application/client ID | `<application-client-id>` |
| Service principal object ID | `<service-principal-object-id>` |
| Test user | `<test-user-upn>` |
| Admin approver | `<admin-approver-upn>` |
| App owner | `<app-owner-upn>` |
| Publisher domain | `<verified-publisher-domain>` |
| PowerShell module | Microsoft Graph PowerShell |
| Graph permission scope | `Application.ReadWrite.All`, `Directory.ReadWrite.All`, `DelegatedPermissionGrant.ReadWrite.All`, `AppRoleAssignment.ReadWrite.All`, `Policy.ReadWrite.Authorization`, `AuditLog.Read.All` |

## Mental_Model

Consent controls what permissions an application can use against APIs such as Microsoft Graph.

There are two main consent types:

| Consent Type | Meaning | Typical Use |
|---|---|---|
| User consent | A user grants permissions for themselves | Low-risk delegated permissions |
| Admin consent | An admin grants permissions tenant-wide or for privileged scopes | Application permissions, high-risk delegated permissions, organization-wide access |

Permission types matter:

| Permission Type | Token Claim | Runs As | Example |
|---|---|---|---|
| Delegated permission | `scp` | Signed-in user | `User.Read` |
| Application permission | `roles` | The app itself | `User.Read.All` |

Basic flow:

~~~text
App Registration
    |
    | declares required API permissions
    v
Consent decision
    |
    | user consent or admin consent
    v
Permission grants / app role assignments
    |
    | allow tokens to contain scopes or roles
    v
Application can call API
~~~

Publisher verification is separate from consent.

Publisher verification helps identify that an app publisher has verified ownership of a domain. It does not automatically make an app safe, but lack of verification is a warning sign for third-party applications.

## Planning_Table

| Decision Area | Recommended Baseline | Notes |
|---|---|---|
| User consent | Restrict or disable broad user consent | Do not let users approve risky apps casually |
| Admin consent workflow | Enable | Lets users request approval instead of self-consenting |
| Admin consent reviewers | Use a small admin/security group | Avoid random approvers |
| High-risk permissions | Admin approval required | Especially Graph read/write directory permissions |
| Application permissions | Admin approval required | App-only access has no user context |
| Publisher verification | Require for third-party app trust decisions | Especially for SaaS apps |
| Consent review | Review grants regularly | Look for stale, excessive, or unknown grants |
| Permission classification | Classify low-impact permissions only | Do not classify broad permissions as low impact |
| App owners | Required | Owners must justify permissions |
| Audit logs | Review after consent changes | Consent events should be traceable |
| Emergency exception | Documented and time-bound | Avoid permanent consent exceptions |

## Prerequisites

| Requirement | Details |
|---|---|
| Role | Global Administrator, Privileged Role Administrator, Cloud Application Administrator, or Application Administrator depending action |
| Permission knowledge | Know why each API permission is needed |
| App owner | Identified business and technical owner |
| Test user | Non-admin user for consent behavior testing |
| Admin approver | User or group responsible for consent review |
| Graph PowerShell | Installed and available |
| Change window | Recommended for production tenant consent changes |
| Verified domain | Required for publisher verification scenarios |
| Documentation location | Ticket, repo, or workbook notes |

## Variables

Replace placeholders before running commands.

~~~powershell
$TenantId = "<tenant-id>"
$AppRegistrationName = "<app-registration-name>"
$EnterpriseAppName = "<enterprise-app-name>"
$ApplicationClientId = "<application-client-id>"
$ServicePrincipalObjectId = "<service-principal-object-id>"
$TestUserUpn = "<test-user-upn>"
$AdminApproverUpn = "<admin-approver-upn>"
$PublisherDomain = "<verified-publisher-domain>"
~~~

## Configuration_Checklist

| Step | Task | Admin Center / Portal Path | PowerShell / CLI / Graph | Expected Result |
|---:|---|---|---|---|
| 1 | Sign in to Entra admin center | `https://entra.microsoft.com` | N/A | Admin portal loads |
| 2 | Connect to Microsoft Graph | N/A | `Connect-MgGraph` | Required scopes are active |
| 3 | Review tenant user consent settings | Identity > Applications > Enterprise applications > Consent and permissions | Authorization policy | Current user consent posture is known |
| 4 | Restrict user consent | Consent and permissions > User consent settings | Authorization policy | Users cannot approve risky apps broadly |
| 5 | Enable admin consent workflow | Consent and permissions > Admin consent settings | Admin consent request policy | Users can request admin approval |
| 6 | Configure consent reviewers | Admin consent settings | Admin consent request policy | Reviewers are defined |
| 7 | Review app API permissions | App registrations > `<app>` > API permissions | `RequiredResourceAccess` | Required permissions are known |
| 8 | Identify delegated permissions | API permissions | Graph service principal scopes | Delegated permissions are classified |
| 9 | Identify application permissions | API permissions | Graph app roles | App permissions are classified |
| 10 | Grant admin consent if approved | API permissions > Grant admin consent | Permission grants / app role assignments | Approved permissions are granted |
| 11 | Review existing grants | Enterprise apps > Permissions | OAuth2 permission grants / app role assignments | Excessive grants are identified |
| 12 | Review publisher verification | App registration > Branding & properties | App publisher properties | Publisher status is known |
| 13 | Configure publisher domain if applicable | App registration > Branding & properties | Application publisher domain | Publisher domain is set |
| 14 | Validate user consent behavior | Test user browser session | N/A | User cannot self-consent to blocked permissions |
| 15 | Validate admin consent request | Test user request flow | N/A | Admin consent request is created |
| 16 | Review audit logs | Monitoring & health > Audit logs | `Get-MgAuditLogDirectoryAudit` | Consent changes are logged |
| 17 | Document final state | Workbook notes / repo | N/A | Baseline is documented |

## Step_01_Connect_To_Microsoft_Graph

~~~powershell
Install-Module Microsoft.Graph -Scope CurrentUser -Force

Connect-MgGraph -TenantId $TenantId -Scopes `
  "Application.ReadWrite.All", `
  "Directory.ReadWrite.All", `
  "DelegatedPermissionGrant.ReadWrite.All", `
  "AppRoleAssignment.ReadWrite.All", `
  "Policy.ReadWrite.Authorization", `
  "AuditLog.Read.All"

Get-MgContext
~~~

Expected result:

~~~text
Microsoft Graph connection succeeds.
Tenant ID matches the target tenant.
Required scopes are present.
~~~

## Step_02_Review_Current_User_Consent_Settings

### Portal

| Step | Action |
|---:|---|
| 1 | Go to `https://entra.microsoft.com` |
| 2 | Open Identity |
| 3 | Open Applications |
| 4 | Select Enterprise applications |
| 5 | Select Consent and permissions |
| 6 | Open User consent settings |
| 7 | Record current setting |

### What To Look For

| Setting Pattern | Meaning | Baseline View |
|---|---|---|
| Allow user consent for all apps | Users can approve apps broadly | Too permissive for most tenants |
| Allow user consent from verified publishers for selected permissions | More controlled | Acceptable if low-impact permissions are classified correctly |
| Do not allow user consent | Most restrictive | Strong baseline for managed enterprise tenants |

Recommended baseline:

~~~text
Do not allow broad user consent.
Use admin consent workflow for approvals.
If user consent is allowed, limit it to verified publishers and low-impact permissions only.
~~~

## Step_03_Review_Authorization_Policy

The tenant authorization policy controls consent behavior and other authorization defaults.

~~~powershell
$AuthPolicy = Get-MgPolicyAuthorizationPolicy

$AuthPolicy | Format-List `
  Id,
  DisplayName,
  Description,
  DefaultUserRolePermissions,
  PermissionGrantPolicyIdsAssignedToDefaultUserRole
~~~

Expected result:

~~~text
Current user consent and default user permission posture is visible.
Permission grant policies assigned to the default user role are identified.
~~~

## Step_04_Restrict_User_Consent_Baseline

Use the portal for this step unless you are deliberately managing consent policies through Graph.

### Portal

| Step | Action |
|---:|---|
| 1 | Open Entra admin center |
| 2 | Go to Applications > Enterprise applications |
| 3 | Select Consent and permissions |
| 4 | Select User consent settings |
| 5 | Choose the tenant baseline |
| 6 | Recommended: Do not allow user consent, or allow only verified publishers for selected low-impact permissions |
| 7 | Save |

### Baseline Decision

| Tenant Type | Recommended User Consent Setting |
|---|---|
| Lab tenant | Allow limited user consent for learning if isolated |
| Production tenant | Restrict user consent |
| Regulated tenant | Disable user consent |
| Small business tenant | Restrict user consent and enable admin workflow |
| Developer tenant | Allow only if risk is accepted |

Expected result:

~~~text
Users cannot grant broad application permissions without admin review.
Low-risk exceptions are deliberate and documented.
~~~

## Step_05_Enable_Admin_Consent_Workflow

Admin consent workflow lets users request approval when they cannot consent directly.

### Portal

| Step | Action |
|---:|---|
| 1 | Open Entra admin center |
| 2 | Go to Applications > Enterprise applications |
| 3 | Select Consent and permissions |
| 4 | Select Admin consent settings |
| 5 | Set Users can request admin consent to apps they are unable to consent to: Yes |
| 6 | Select reviewers |
| 7 | Configure email notifications |
| 8 | Configure reminder settings |
| 9 | Set request expiration |
| 10 | Save |

### Recommended Settings

| Setting | Baseline |
|---|---|
| Admin consent workflow | Enabled |
| Reviewers | Security admins, cloud app admins, or app governance group |
| Email notifications | Enabled |
| Request expiration | 30 days |
| User justification required | Yes, operationally required even if not technically enforced |
| Reviewer count | At least two |

Expected result:

~~~text
Users can request admin consent instead of being fully blocked.
Designated reviewers receive requests and can approve or deny.
~~~

## Step_06_Document_Admin_Consent_Reviewers

| Reviewer Type | Example |
|---|---|
| Primary reviewer | `<admin-approver-upn>` |
| Backup reviewer | `<backup-admin-upn>` |
| Security group | `<consent-reviewers-group>` |
| Escalation owner | `<security-owner-upn>` |

Admin consent reviewer baseline:

~~~text
Use a small, accountable reviewer group.
Do not make every admin a consent reviewer.
Reviewers must understand Microsoft Graph permissions, app risk, and publisher trust.
~~~

## Step_07_Review_Target_App_API_Permissions

### Portal

| Step | Action |
|---:|---|
| 1 | Open App registrations |
| 2 | Select `<app-registration-name>` |
| 3 | Select API permissions |
| 4 | Record configured permissions |
| 5 | Identify delegated permissions |
| 6 | Identify application permissions |
| 7 | Confirm whether admin consent is granted |

### PowerShell

~~~powershell
$App = Get-MgApplication -Filter "displayName eq '$AppRegistrationName'"

$App | Select-Object DisplayName, Id, AppId, SignInAudience

$App.RequiredResourceAccess
~~~

Expected result:

~~~text
Configured API permissions are visible.
Delegated and application permissions can be reviewed separately.
~~~

## Step_08_Decode_Microsoft_Graph_Permissions

Microsoft Graph AppId:

~~~text
00000003-0000-0000-c000-000000000000
~~~

### Pull Microsoft Graph Permission Catalog

~~~powershell
$GraphSp = Get-MgServicePrincipal -Filter "appId eq '00000003-0000-0000-c000-000000000000'"

$GraphDelegatedScopes = $GraphSp.Oauth2PermissionScopes |
Select-Object Id, Value, AdminConsentDisplayName, Type, IsEnabled

$GraphApplicationRoles = $GraphSp.AppRoles |
Where-Object { $_.AllowedMemberTypes -contains "Application" } |
Select-Object Id, Value, DisplayName, IsEnabled

$GraphDelegatedScopes | Sort-Object Value | Select-Object -First 10
$GraphApplicationRoles | Sort-Object Value | Select-Object -First 10
~~~

Expected result:

~~~text
Microsoft Graph delegated scopes and application roles are available for permission mapping.
~~~

### Map App Required Permissions To Friendly Names

~~~powershell
$Required = $App.RequiredResourceAccess

foreach ($Resource in $Required) {
    $ResourceSp = Get-MgServicePrincipal -Filter "appId eq '$($Resource.ResourceAppId)'"

    foreach ($Access in $Resource.ResourceAccess) {
        $Delegated = $ResourceSp.Oauth2PermissionScopes | Where-Object { $_.Id -eq $Access.Id }
        $Application = $ResourceSp.AppRoles | Where-Object { $_.Id -eq $Access.Id }

        [PSCustomObject]@{
            ResourceAppId = $Resource.ResourceAppId
            ResourceName = $ResourceSp.DisplayName
            PermissionId = $Access.Id
            PermissionType = $Access.Type
            PermissionValue = if ($Delegated) { $Delegated.Value } elseif ($Application) { $Application.Value } else { "<unknown>" }
            DisplayName = if ($Delegated) { $Delegated.AdminConsentDisplayName } elseif ($Application) { $Application.DisplayName } else { "<unknown>" }
        }
    }
}
~~~

Expected result:

~~~text
Required permissions are shown with readable names.
Unknown or excessive permissions are flagged for review.
~~~

## Step_09_Classify_Permission_Risk

Use a conservative review posture.

| Permission Pattern | Risk Level | Notes |
|---|---|---|
| `User.Read` delegated | Low | Basic sign-in/profile read |
| `openid`, `profile`, `email` | Low | OIDC sign-in basics |
| `offline_access` | Medium | Refresh token capability |
| `User.ReadBasic.All` delegated | Medium | Reads basic profile for all users |
| `User.Read.All` delegated | Medium/High | Reads all users under user context |
| `Group.Read.All` | High | Group visibility can expose structure |
| `Directory.Read.All` | High | Broad directory read |
| `User.ReadWrite.All` | High | Can modify users |
| `Group.ReadWrite.All` | High | Can modify groups |
| `Directory.ReadWrite.All` | Critical | Broad directory write |
| `Application.ReadWrite.All` | Critical | Can manage apps |
| `AppRoleAssignment.ReadWrite.All` | Critical | Can grant app assignments |
| `RoleManagement.ReadWrite.Directory` | Critical | Role management risk |
| `Mail.Read` | High | Mailbox data exposure |
| `Mail.Send` | High | Can send mail |
| `Files.Read.All` | High | File content exposure |
| `Sites.Read.All` | High | SharePoint data exposure |
| Any application permission | High by default | App-only access has no signed-in user |

Baseline:

~~~text
Treat application permissions as high risk unless proven otherwise.
Treat directory write, role management, app management, mail, files, and sites permissions as high risk.
Require documented justification before granting admin consent.
~~~

## Step_10_Review_Existing_Delegated_Permission_Grants

OAuth2 permission grants represent delegated permission consent.

### Find Service Principal

~~~powershell
$Sp = Get-MgServicePrincipal -Filter "appId eq '$($App.AppId)'"

$Sp | Select-Object DisplayName, Id, AppId
~~~

### Review Delegated Grants

~~~powershell
Get-MgOauth2PermissionGrant -All |
Where-Object { $_.ClientId -eq $Sp.Id } |
Select-Object Id, ClientId, ConsentType, PrincipalId, ResourceId, Scope
~~~

Expected result:

~~~text
Existing delegated permission grants for the application are visible.
Tenant-wide delegated grants usually show ConsentType as AllPrincipals.
User-specific grants usually show ConsentType as Principal.
~~~

## Step_11_Review_Existing_Application_Permission_Grants

Application permissions are represented as app role assignments from the client service principal to the resource service principal.

~~~powershell
Get-MgServicePrincipalAppRoleAssignment -ServicePrincipalId $Sp.Id -All |
Select-Object PrincipalDisplayName, ResourceDisplayName, AppRoleId, CreatedDateTime
~~~

Expected result:

~~~text
Existing application permission grants are visible.
Each app role assignment should map to an approved application permission.
~~~

## Step_12_Grant_Admin_Consent_For_Approved_Delegated_Permissions

Prefer portal admin consent for standard admin workflow unless automation is required.

### Portal

| Step | Action |
|---:|---|
| 1 | Open App registrations |
| 2 | Select `<app-registration-name>` |
| 3 | Select API permissions |
| 4 | Review permissions |
| 5 | Confirm approval ticket exists |
| 6 | Select Grant admin consent for `<tenant-name>` |
| 7 | Confirm |
| 8 | Verify green consent status |

Expected result:

~~~text
Admin consent is granted for approved delegated permissions.
The app can receive delegated scopes in tokens when requested.
~~~

### Review After Grant

~~~powershell
Get-MgOauth2PermissionGrant -All |
Where-Object { $_.ClientId -eq $Sp.Id } |
Select-Object Id, ConsentType, Scope, ResourceId
~~~

Expected result:

~~~text
Delegated permission grant exists for the approved scopes.
~~~

## Step_13_Grant_Admin_Consent_For_Approved_Application_Permissions

Application permissions require admin consent.

### Portal

| Step | Action |
|---:|---|
| 1 | Open App registrations |
| 2 | Select `<app-registration-name>` |
| 3 | Select API permissions |
| 4 | Confirm application permissions are approved |
| 5 | Select Grant admin consent |
| 6 | Confirm |
| 7 | Verify status shows granted |

Expected result:

~~~text
Application permission grants are created for approved app roles.
The app can receive roles claim in app-only tokens.
~~~

### Validate Application Permission Assignments

~~~powershell
Get-MgServicePrincipalAppRoleAssignment -ServicePrincipalId $Sp.Id -All |
Select-Object PrincipalDisplayName, ResourceDisplayName, AppRoleId, CreatedDateTime
~~~

Expected result:

~~~text
App role assignments exist for approved application permissions.
~~~

## Step_14_Review_Enterprise_App_Permissions_Page

The Enterprise Application view is often clearer for reviewing what has actually been granted.

### Portal

| Step | Action |
|---:|---|
| 1 | Open Enterprise applications |
| 2 | Search for `<enterprise-app-name>` |
| 3 | Open the app |
| 4 | Select Permissions |
| 5 | Review User consent tab |
| 6 | Review Admin consent tab |
| 7 | Review granted permissions |
| 8 | Remove unauthorized grants if approved |

Expected result:

~~~text
Granted permissions match the approved consent baseline.
No unknown or excessive grants remain.
~~~

## Step_15_Remove_Unauthorized_Delegated_Consent

Do not remove grants without confirming app impact.

### Identify Grant

~~~powershell
$DelegatedGrants = Get-MgOauth2PermissionGrant -All |
Where-Object { $_.ClientId -eq $Sp.Id }

$DelegatedGrants |
Select-Object Id, ConsentType, Scope, PrincipalId, ResourceId
~~~

### Remove Grant

~~~powershell
$GrantIdToRemove = "<oauth2-permission-grant-id>"

Remove-MgOauth2PermissionGrant -OAuth2PermissionGrantId $GrantIdToRemove
~~~

Expected result:

~~~text
Unauthorized delegated grant is removed.
Users or the app may be prompted again or blocked depending tenant consent settings.
~~~

## Step_16_Remove_Unauthorized_Application_Permission

Do not remove app role assignments without confirming app impact.

### Identify App Role Assignment

~~~powershell
$AppRoleAssignments = Get-MgServicePrincipalAppRoleAssignment -ServicePrincipalId $Sp.Id -All

$AppRoleAssignments |
Select-Object Id, PrincipalDisplayName, ResourceDisplayName, AppRoleId
~~~

### Remove App Role Assignment

~~~powershell
$AppRoleAssignmentIdToRemove = "<app-role-assignment-id>"

Remove-MgServicePrincipalAppRoleAssignment `
  -ServicePrincipalId $Sp.Id `
  -AppRoleAssignmentId $AppRoleAssignmentIdToRemove
~~~

Expected result:

~~~text
Unauthorized application permission grant is removed.
App-only token calls depending on that role will fail.
~~~

## Step_17_Review_Publisher_Verification_Status

Publisher verification matters most for third-party apps and multi-tenant apps.

### Portal

| Step | Action |
|---:|---|
| 1 | Open App registrations |
| 2 | Select `<app-registration-name>` |
| 3 | Select Branding & properties |
| 4 | Review Publisher domain |
| 5 | Review Verified publisher status |
| 6 | Review Terms of service and privacy statement URLs if applicable |

### What To Look For

| Field | Good Sign | Warning Sign |
|---|---|---|
| Publisher domain | Verified domain present | Missing or suspicious domain |
| Verified publisher | Verified | Unverified third-party app |
| App logo/name | Matches vendor/business use | Generic or misleading name |
| Terms/privacy URLs | Present for external apps | Missing for SaaS apps |
| Requested permissions | Least privilege | Broad Graph or mailbox/file permissions |

Expected result:

~~~text
Publisher status is known and documented.
Third-party unverified apps with high-risk permissions are escalated.
~~~

## Step_18_Configure_Publisher_Domain_For_Internal_App

For internal apps, configure a verified publisher domain when applicable.

Prerequisite:

~~~text
The domain must already be verified in the tenant.
~~~

### Portal

| Step | Action |
|---:|---|
| 1 | Open App registrations |
| 2 | Select `<app-registration-name>` |
| 3 | Select Branding & properties |
| 4 | Set Publisher domain |
| 5 | Save |

### PowerShell

~~~powershell
Update-MgApplication `
  -ApplicationId $App.Id `
  -PublisherDomain $PublisherDomain

Get-MgApplication -ApplicationId $App.Id |
Select-Object DisplayName, PublisherDomain
~~~

Expected result:

~~~text
Application publisher domain is set to a verified tenant domain.
~~~

## Step_19_Review_Verified_Publisher_For_External_App

For external SaaS apps, do not change the vendor publisher. Review it.

| Review Item | Question |
|---|---|
| Publisher verified | Is the publisher verified? |
| Publisher domain | Does it match the vendor? |
| App name | Is it the expected app? |
| Consent URL | Did the request come from a legitimate vendor workflow? |
| Permissions | Are the requested permissions justified? |
| Admin consent | Is there a ticket or business owner approval? |

Decision baseline:

~~~text
Do not grant high-risk permissions to unverified third-party apps without security review.
Do not approve consent requests from unknown publishers.
Do not approve apps with misleading names or unexplained permissions.
~~~

## Step_20_Test_User_Consent_Behavior

Use a non-admin test user.

| Step | Action | Expected Result |
|---:|---|---|
| 1 | Open private browser session | Clean sign-in state |
| 2 | Sign in as `<test-user-upn>` | User session starts |
| 3 | Attempt app sign-in requiring consent | Consent behavior appears |
| 4 | If user consent is restricted | User cannot approve risky permissions |
| 5 | If admin workflow is enabled | User can request admin approval |
| 6 | Submit request with justification | Request appears for reviewers |

Expected result:

~~~text
User consent behavior matches the tenant baseline.
Users cannot grant high-risk permissions directly.
Admin consent workflow catches blocked consent requests.
~~~

## Step_21_Test_Admin_Consent_Request_Review

### Portal

| Step | Action |
|---:|---|
| 1 | Sign in as consent reviewer |
| 2 | Open Entra admin center |
| 3 | Go to Enterprise applications |
| 4 | Open Admin consent requests |
| 5 | Select pending request |
| 6 | Review app, publisher, permissions, justification |
| 7 | Approve or deny |
| 8 | Document decision |

Expected result:

~~~text
Reviewer can see pending admin consent request.
Approval or denial is logged.
App owner and requester can be notified through the workflow.
~~~

## Step_22_Review_Audit_Logs

### Portal

| Step | Action |
|---:|---|
| 1 | Open Entra admin center |
| 2 | Go to Identity > Monitoring & health |
| 3 | Select Audit logs |
| 4 | Filter for application consent activity |
| 5 | Review consent grants, removals, and policy changes |

### PowerShell

~~~powershell
Get-MgAuditLogDirectoryAudit -Top 100 |
Where-Object {
    $_.ActivityDisplayName -match "consent|permission|application|service principal|policy"
} |
Select-Object ActivityDateTime, ActivityDisplayName, Result, InitiatedBy
~~~

Expected result:

~~~text
Consent changes, app permission changes, service principal changes, and policy changes are visible in audit logs.
~~~

## Step_23_Document_Final_State

| Field | Value |
|---|---|
| Tenant | `<tenant-name>` |
| User consent setting | `<restricted-disabled-low-impact-only>` |
| Admin consent workflow | Enabled / Disabled |
| Consent reviewers | `<reviewer-list>` |
| Request expiration | `<days>` |
| Target app registration | `<app-registration-name>` |
| Target application/client ID | `<application-client-id>` |
| Target service principal ID | `<service-principal-object-id>` |
| Delegated permissions approved | `<delegated-permission-list>` |
| Application permissions approved | `<application-permission-list>` |
| Admin consent granted | Yes / No |
| Publisher domain | `<publisher-domain>` |
| Verified publisher | Yes / No / N/A |
| Unauthorized grants removed | Yes / No |
| Audit log reviewed | Yes / No |
| Validation date | `<yyyy-mm-dd>` |
| Validated by | `<admin-upn>` |

## Verification_Commands

### Confirm App Registration

~~~powershell
Get-MgApplication -ApplicationId $App.Id |
Select-Object DisplayName, Id, AppId, SignInAudience, PublisherDomain
~~~

### Confirm Service Principal

~~~powershell
Get-MgServicePrincipal -Filter "appId eq '$($App.AppId)'" |
Select-Object DisplayName, Id, AppId, AccountEnabled
~~~

### Confirm Required API Permissions

~~~powershell
(Get-MgApplication -ApplicationId $App.Id).RequiredResourceAccess
~~~

### Confirm Delegated Grants

~~~powershell
Get-MgOauth2PermissionGrant -All |
Where-Object { $_.ClientId -eq $Sp.Id } |
Select-Object Id, ConsentType, Scope, PrincipalId, ResourceId
~~~

### Confirm Application Permission Grants

~~~powershell
Get-MgServicePrincipalAppRoleAssignment -ServicePrincipalId $Sp.Id -All |
Select-Object Id, PrincipalDisplayName, ResourceDisplayName, AppRoleId, CreatedDateTime
~~~

### Confirm Authorization Policy

~~~powershell
Get-MgPolicyAuthorizationPolicy |
Select-Object Id, DisplayName, PermissionGrantPolicyIdsAssignedToDefaultUserRole
~~~

### Confirm Consent-Related Audit Events

~~~powershell
Get-MgAuditLogDirectoryAudit -Top 100 |
Where-Object {
    $_.ActivityDisplayName -match "consent|permission grant|app role assignment|authorization policy"
} |
Select-Object ActivityDateTime, ActivityDisplayName, Result
~~~

## Failure_Checks

| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| User can approve risky apps | User consent too permissive | User consent settings | Restrict user consent |
| User cannot request admin consent | Admin consent workflow disabled | Admin consent settings | Enable workflow |
| Reviewer not receiving requests | Reviewer not configured or notification disabled | Admin consent settings | Add reviewer and enable notifications |
| Admin consent button unavailable | Insufficient role | Current admin role | Use Global Admin or proper application admin role |
| App still lacks permission after consent | Wrong permission type or token request | Check token `scp` or `roles` | Request correct scope or role |
| Delegated permission not in token | App did not request scope | Auth request parameters | Add requested scope |
| Application permission not in token | App role assignment missing | Service principal app role assignments | Grant approved application permission |
| Consent granted to wrong app | Similar app names | Compare AppId/client ID | Remove wrong grant and approve correct app |
| Publisher missing | Publisher domain not configured or external app unverified | Branding & properties | Configure internal domain or escalate external app |
| Permission grant still exists after removing required permission | Required permissions and granted permissions are separate | OAuth2 grants / app role assignments | Remove grant explicitly |
| Users see consent prompt repeatedly | Consent missing or app requests dynamic scopes | Enterprise app permissions | Grant approved consent or correct requested scopes |
| Admin consent request stale | Request expired | Admin consent requests | User must resubmit |
| Unknown app has broad grants | Shadow IT or phishing consent | Enterprise apps permissions/audit logs | Remove grants and investigate |

## Rollback

Rollback depends on what changed. Confirm app impact before removing consent grants.

### Disable Admin Consent Workflow

Use only if approved by tenant governance.

~~~text
Portal path:
Entra admin center > Applications > Enterprise applications > Consent and permissions > Admin consent settings
Set Users can request admin consent to apps they are unable to consent to: No
Save
~~~

Expected result:

~~~text
Users can no longer submit admin consent requests through the workflow.
Blocked consent attempts remain blocked.
~~~

### Revert User Consent Setting

Use only if the previous baseline was approved.

~~~text
Portal path:
Entra admin center > Applications > Enterprise applications > Consent and permissions > User consent settings
Restore previous user consent setting
Save
~~~

Expected result:

~~~text
Tenant user consent behavior returns to the approved previous state.
~~~

### Remove Delegated Permission Grant

~~~powershell
$GrantIdToRemove = "<oauth2-permission-grant-id>"

Remove-MgOauth2PermissionGrant -OAuth2PermissionGrantId $GrantIdToRemove
~~~

Expected result:

~~~text
Delegated permission consent grant is removed.
Users or applications depending on the grant may fail or prompt again.
~~~

### Remove Application Permission Grant

~~~powershell
$AppRoleAssignmentIdToRemove = "<app-role-assignment-id>"

Remove-MgServicePrincipalAppRoleAssignment `
  -ServicePrincipalId $Sp.Id `
  -AppRoleAssignmentId $AppRoleAssignmentIdToRemove
~~~

Expected result:

~~~text
Application permission grant is removed.
App-only calls that depend on the permission fail.
~~~

### Clear Required API Permissions From App Registration

This removes the app registration declaration. It does not necessarily remove already-granted consent.

~~~powershell
Update-MgApplication `
  -ApplicationId $App.Id `
  -RequiredResourceAccess @()
~~~

Expected result:

~~~text
Required API permissions are cleared from the app registration.
Existing grants should still be reviewed separately.
~~~

## Security_Baseline

| Control | Baseline |
|---|---|
| User consent | Restricted or disabled |
| Admin consent workflow | Enabled |
| Reviewers | Small accountable group |
| Low-impact permissions | Carefully classified |
| High-risk permissions | Admin approval required |
| Application permissions | Admin approval required |
| Directory write permissions | Security review required |
| Mail/file/site permissions | Security review required |
| Publisher verification | Required review signal |
| Unknown publishers | Do not approve high-risk grants without review |
| Consent grants | Reviewed regularly |
| Audit logs | Reviewed after changes |
| Documentation | Required for each approval |
| Emergency approvals | Time-bound and documented |

## Admin_Notes

~~~text
Tenant:
User consent setting:
Admin consent workflow enabled:
Consent reviewers:
Request expiration:
App registration:
Application/client ID:
Enterprise app:
Service principal object ID:
Delegated permissions requested:
Delegated permissions approved:
Application permissions requested:
Application permissions approved:
Admin consent granted:
Publisher verified:
Publisher domain:
Unauthorized grants removed:
Audit logs reviewed:
Approver:
Validation result:
Known exceptions:
~~~

## Related_Labs

| Workbook                                                                             | Relationship                                                    |
| ------------------------------------------------------------------------------------ | --------------------------------------------------------------- |
| 01_Configure_Enterprise_Applications_Service_Principals_And_Assignments.md           | Covers Enterprise Application assignment controls               |
| 02_Configure_App_Registrations_API_Permissions_Secrets_Certificates_And_Owners.md    | Covers app registration permission declarations and credentials |
| 04_Configure_SAML_OIDC_OAuth_SSO_And_Application_Provisioning.md                     | Covers SSO and provisioning                                     |
| 05_Configure_Conditional_Access_App_Targeting_Session_Controls_And_Access_Reviews.md | Covers app targeting, session controls, and reviews             |
| 06_Troubleshoot_Enterprise_App_SSO_Consent_Provisioning_And_API_Permission_Issues.md | Covers troubleshooting consent and permission failures          |