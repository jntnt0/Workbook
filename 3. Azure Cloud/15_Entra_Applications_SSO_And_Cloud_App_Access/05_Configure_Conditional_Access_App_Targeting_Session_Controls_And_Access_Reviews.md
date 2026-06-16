# 05_Configure_Conditional_Access_App_Targeting_Session_Controls_And_Access_Reviews

## Objective

Configure a Microsoft Entra Conditional Access and governance baseline for Enterprise Applications.

This workbook covers:

- Targeting specific cloud apps in Conditional Access
- Including and excluding users/groups
- Excluding break-glass accounts
- App assignment alignment
- Grant controls
- Session controls
- Sign-in frequency
- Persistent browser session behavior
- Conditional Access report-only validation
- Access reviews for application assignments
- Sign-in log validation
- Failure checks and rollback

## Lab_Context

| Item | Value |
|---|---|
| Tenant | `<tenant-name>.onmicrosoft.com` |
| Admin portal | `https://entra.microsoft.com` |
| Primary admin center | Microsoft Entra admin center |
| Target enterprise app | `<enterprise-app-name>` |
| Application/client ID | `<application-client-id>` |
| Service principal object ID | `<service-principal-object-id>` |
| Access group | `<app-access-group-name>` |
| Pilot group | `<ca-pilot-group-name>` |
| Exclusion group | `<ca-exclusion-group-name>` |
| Break-glass account 1 | `<break-glass-user1-upn>` |
| Break-glass account 2 | `<break-glass-user2-upn>` |
| Test user | `<test-user-upn>` |
| Access review name | `<access-review-name>` |
| Access review reviewer | `<reviewer-upn>` |
| Conditional Access policy name | `<ca-policy-name>` |
| PowerShell module | Microsoft Graph PowerShell |
| Graph permission scope | `Policy.ReadWrite.ConditionalAccess`, `Application.Read.All`, `Directory.ReadWrite.All`, `Group.ReadWrite.All`, `AuditLog.Read.All`, `AccessReview.ReadWrite.All` |

## Mental_Model

Conditional Access controls access decisions at sign-in.

Enterprise Application assignments control who is allowed to use an app.

Access reviews control whether assignments remain valid over time.

These controls should work together.

~~~text
User / Group Assignment
    |
    | determines whether user is allowed to use app
    v
Enterprise Application
    |
    | targeted by Conditional Access
    v
Conditional Access Policy
    |
    | evaluates user, app, device, location, risk, client, session
    v
Grant Controls / Session Controls
    |
    | require MFA, compliant device, approved app, sign-in frequency, app control
    v
Access Review
    |
    | periodically confirms who should keep access
~~~

Do not rely on Conditional Access alone for application entitlement.

Do not rely on app assignment alone for sign-in risk control.

Use both.

## Planning_Table

| Decision Area | Recommended Baseline | Notes |
|---|---|---|
| Policy mode | Start in Report-only | Validate impact before enforcement |
| App targeting | Target specific Enterprise Application | Avoid tenant-wide blast radius unless intended |
| User scope | Pilot group first | Expand after sign-in log validation |
| Break-glass accounts | Always excluded | Exclusion must be documented |
| Assignment required | Enabled on enterprise app | Keeps access entitlement explicit |
| Grant control | Require MFA minimum | Stronger controls for sensitive apps |
| Device control | Require compliant or hybrid joined device where appropriate | Do not apply if unmanaged access is required |
| Session control | Use sign-in frequency for sensitive apps | Avoid excessive prompts |
| Persistent browser | Never persistent for sensitive apps | Balance security and usability |
| App enforced restrictions | Use for supported Microsoft cloud apps | Not all apps support this |
| Defender for Cloud Apps session control | Use for sanctioned SaaS apps when licensed | Enables real-time session controls |
| Access reviews | Required for sensitive apps | Review group or app assignment |
| Review cadence | Quarterly for sensitive apps | More frequent for privileged apps |
| Rollback | Disable policy or return to report-only | Do not delete until evidence is retained |

## Prerequisites

| Requirement | Details |
|---|---|
| Role | Conditional Access Administrator, Security Administrator, Global Administrator, or Identity Governance Administrator depending task |
| License | Entra ID P1 for Conditional Access, Entra ID P2 for access reviews and identity governance features |
| Enterprise app | Target app exists |
| Assignment baseline | Assignment required configured where appropriate |
| Access group | Group assigned to Enterprise Application |
| Pilot group | Small group used for initial CA testing |
| Exclusion group | Group used for emergency/exception exclusions |
| Break-glass accounts | At least two known and excluded |
| Test user | Non-admin test user |
| Sign-in logs | Available for validation |
| Change window | Recommended before enforcement |

## Variables

Replace placeholders before running commands.

~~~powershell
$TenantId = "<tenant-id>"
$EnterpriseAppName = "<enterprise-app-name>"
$ApplicationClientId = "<application-client-id>"
$ServicePrincipalObjectId = "<service-principal-object-id>"
$AccessGroupName = "<app-access-group-name>"
$PilotGroupName = "<ca-pilot-group-name>"
$ExclusionGroupName = "<ca-exclusion-group-name>"
$BreakGlassUser1 = "<break-glass-user1-upn>"
$BreakGlassUser2 = "<break-glass-user2-upn>"
$TestUserUpn = "<test-user-upn>"
$ConditionalAccessPolicyName = "<ca-policy-name>"
$AccessReviewName = "<access-review-name>"
$ReviewerUpn = "<reviewer-upn>"
~~~

## Configuration_Checklist

| Step | Task | Admin Center / Portal Path | PowerShell / CLI / Graph | Expected Result |
|---:|---|---|---|---|
| 1 | Sign in to Entra admin center | `https://entra.microsoft.com` | N/A | Admin portal loads |
| 2 | Connect to Microsoft Graph | N/A | `Connect-MgGraph` | Required scopes are active |
| 3 | Locate Enterprise Application | Applications > Enterprise applications | `Get-MgServicePrincipal` | Target app is found |
| 4 | Confirm assignment required | Enterprise app > Properties | `AppRoleAssignmentRequired` | Assignment behavior is known |
| 5 | Confirm app assignments | Enterprise app > Users and groups | App role assignments | Access group is assigned |
| 6 | Create or confirm pilot group | Identity > Groups | `Get-MgGroup` / `New-MgGroup` | Pilot group exists |
| 7 | Create or confirm exclusion group | Identity > Groups | `Get-MgGroup` / `New-MgGroup` | Exclusion group exists |
| 8 | Add break-glass accounts to exclusion group | Groups > Members | `New-MgGroupMemberByRef` | Break-glass accounts excluded |
| 9 | Create Conditional Access policy in report-only | Protection > Conditional Access | `New-MgIdentityConditionalAccessPolicy` | Policy exists in report-only |
| 10 | Target specific cloud app | CA policy > Target resources | IncludedApplications | Enterprise app is targeted |
| 11 | Include pilot group | CA policy > Users | IncludedUsers / IncludedGroups | Pilot users are included |
| 12 | Exclude break-glass group | CA policy > Users > Exclude | ExcludedGroups | Emergency accounts excluded |
| 13 | Configure grant controls | CA policy > Grant | Built-in controls | MFA/compliant device requirement set |
| 14 | Configure session controls | CA policy > Session | Session controls | Sign-in/session behavior set |
| 15 | Validate report-only results | Sign-in logs > Report-only | Sign-in logs | Expected result shown without enforcement |
| 16 | Enable policy after validation | CA policy > Enable policy | `Update-MgIdentityConditionalAccessPolicy` | Policy enforced |
| 17 | Create access review | Identity Governance > Access reviews | Access review API | Review created for app/group access |
| 18 | Validate reviewer workflow | Access reviews | N/A | Reviewer can approve/deny access |
| 19 | Review sign-in logs | Enterprise app > Sign-in logs | `Get-MgAuditLogSignIn` | CA result visible |
| 20 | Document final state | Workbook notes / repo | N/A | CA and review baseline documented |

## Step_01_Connect_To_Microsoft_Graph

~~~powershell
Install-Module Microsoft.Graph -Scope CurrentUser -Force

Connect-MgGraph -TenantId $TenantId -Scopes `
  "Policy.ReadWrite.ConditionalAccess", `
  "Application.Read.All", `
  "Directory.ReadWrite.All", `
  "Group.ReadWrite.All", `
  "AuditLog.Read.All", `
  "AccessReview.ReadWrite.All"

Get-MgContext
~~~

Expected result:

~~~text
Microsoft Graph connection succeeds.
Tenant ID matches the target tenant.
Required scopes are present.
~~~

## Step_02_Locate_Target_Enterprise_Application

### Portal

| Step | Action |
|---:|---|
| 1 | Go to `https://entra.microsoft.com` |
| 2 | Open Applications |
| 3 | Select Enterprise applications |
| 4 | Search for `<enterprise-app-name>` |
| 5 | Open the app |
| 6 | Record Application ID and Object ID |

### PowerShell

~~~powershell
$Sp = Get-MgServicePrincipal -Filter "displayName eq '$EnterpriseAppName'"

$Sp | Select-Object DisplayName, Id, AppId, AccountEnabled, AppRoleAssignmentRequired
~~~

Expected result:

~~~text
Enterprise Application service principal is found.
Application/client ID and service principal object ID are known.
~~~

## Step_03_Confirm_App_Assignment_Baseline

Conditional Access controls sign-in conditions. Enterprise Application assignments control entitlement.

### Portal

| Step | Action |
|---:|---|
| 1 | Open Enterprise applications |
| 2 | Select `<enterprise-app-name>` |
| 3 | Select Properties |
| 4 | Confirm Assignment required is Yes for controlled app |
| 5 | Select Users and groups |
| 6 | Confirm `<app-access-group-name>` is assigned |

### PowerShell

~~~powershell
Get-MgServicePrincipal -ServicePrincipalId $Sp.Id |
Select-Object DisplayName, AppRoleAssignmentRequired

Get-MgServicePrincipalAppRoleAssignedTo -ServicePrincipalId $Sp.Id -All |
Select-Object PrincipalDisplayName, PrincipalType, AppRoleId, ResourceDisplayName
~~~

Expected result:

~~~text
Assignment required is enabled where appropriate.
Assigned users and groups are known before Conditional Access enforcement.
~~~

## Step_04_Create_Or_Confirm_Pilot_Group

Use a pilot group before enforcing Conditional Access broadly.

### Portal

| Step | Action |
|---:|---|
| 1 | Open Identity > Groups |
| 2 | Search for `<ca-pilot-group-name>` |
| 3 | Create the group if missing |
| 4 | Add test users |
| 5 | Confirm membership |

### PowerShell

~~~powershell
$PilotGroup = Get-MgGroup -Filter "displayName eq '$PilotGroupName'"

if (-not $PilotGroup) {
    $PilotGroup = New-MgGroup `
      -DisplayName $PilotGroupName `
      -MailEnabled:$false `
      -MailNickname (($PilotGroupName -replace '[^a-zA-Z0-9]', '').ToLower()) `
      -SecurityEnabled:$true
}

$PilotGroup | Select-Object DisplayName, Id
~~~

Expected result:

~~~text
Pilot group exists.
Pilot group contains only test users or a small controlled audience.
~~~

## Step_05_Create_Or_Confirm_Exclusion_Group

Use an exclusion group for break-glass and approved emergency exceptions.

### Portal

| Step | Action |
|---:|---|
| 1 | Open Identity > Groups |
| 2 | Search for `<ca-exclusion-group-name>` |
| 3 | Create the group if missing |
| 4 | Add break-glass accounts |
| 5 | Confirm membership |

### PowerShell

~~~powershell
$ExclusionGroup = Get-MgGroup -Filter "displayName eq '$ExclusionGroupName'"

if (-not $ExclusionGroup) {
    $ExclusionGroup = New-MgGroup `
      -DisplayName $ExclusionGroupName `
      -MailEnabled:$false `
      -MailNickname (($ExclusionGroupName -replace '[^a-zA-Z0-9]', '').ToLower()) `
      -SecurityEnabled:$true
}

$ExclusionGroup | Select-Object DisplayName, Id
~~~

Expected result:

~~~text
Exclusion group exists.
Break-glass accounts and approved exceptions can be excluded from the Conditional Access policy.
~~~

## Step_06_Add_Break_Glass_Accounts_To_Exclusion_Group

Break-glass accounts should be excluded from Conditional Access policies that could block emergency access.

~~~powershell
$Bg1 = Get-MgUser -UserId $BreakGlassUser1
$Bg2 = Get-MgUser -UserId $BreakGlassUser2

New-MgGroupMemberByRef `
  -GroupId $ExclusionGroup.Id `
  -BodyParameter @{
    "@odata.id" = "https://graph.microsoft.com/v1.0/directoryObjects/$($Bg1.Id)"
  }

New-MgGroupMemberByRef `
  -GroupId $ExclusionGroup.Id `
  -BodyParameter @{
    "@odata.id" = "https://graph.microsoft.com/v1.0/directoryObjects/$($Bg2.Id)"
  }

Get-MgGroupMember -GroupId $ExclusionGroup.Id -All |
Select-Object Id, AdditionalProperties
~~~

Expected result:

~~~text
Break-glass accounts are members of the exclusion group.
Exclusion group membership is documented and reviewed.
~~~

## Step_07_Add_Test_User_To_Pilot_Group

~~~powershell
$TestUser = Get-MgUser -UserId $TestUserUpn

New-MgGroupMemberByRef `
  -GroupId $PilotGroup.Id `
  -BodyParameter @{
    "@odata.id" = "https://graph.microsoft.com/v1.0/directoryObjects/$($TestUser.Id)"
  }

Get-MgGroupMember -GroupId $PilotGroup.Id -All |
Select-Object Id, AdditionalProperties
~~~

Expected result:

~~~text
Test user is a member of the Conditional Access pilot group.
~~~

## Step_08_Create_Report_Only_Conditional_Access_Policy

Start in report-only mode to validate effect before enforcement.

### Portal

| Step | Action |
|---:|---|
| 1 | Open Entra admin center |
| 2 | Go to Protection |
| 3 | Select Conditional Access |
| 4 | Select Policies |
| 5 | Select New policy |
| 6 | Name policy `<ca-policy-name>` |
| 7 | Include pilot group |
| 8 | Exclude break-glass/exclusion group |
| 9 | Target selected cloud app `<enterprise-app-name>` |
| 10 | Configure grant/session controls |
| 11 | Set Enable policy to Report-only |
| 12 | Create |

### PowerShell Example - Require MFA For Target App

~~~powershell
$PolicyBody = @{
    displayName = $ConditionalAccessPolicyName
    state = "enabledForReportingButNotEnforced"
    conditions = @{
        users = @{
            includeGroups = @($PilotGroup.Id)
            excludeGroups = @($ExclusionGroup.Id)
        }
        applications = @{
            includeApplications = @($Sp.AppId)
        }
        clientAppTypes = @("browser", "mobileAppsAndDesktopClients")
    }
    grantControls = @{
        operator = "OR"
        builtInControls = @("mfa")
    }
}

$Policy = New-MgIdentityConditionalAccessPolicy -BodyParameter $PolicyBody

$Policy | Select-Object Id, DisplayName, State
~~~

Expected result:

~~~text
Conditional Access policy is created in report-only mode.
Policy targets the selected Enterprise Application.
Pilot group is included.
Exclusion group is excluded.
MFA is required in report-only evaluation.
~~~

## Step_09_Target_Specific_Cloud_App

Target the specific Enterprise Application, not all cloud apps, unless the policy is intentionally tenant-wide.

### Portal

| Step | Action |
|---:|---|
| 1 | Open Conditional Access policy |
| 2 | Select Target resources |
| 3 | Choose Cloud apps |
| 4 | Include Select apps |
| 5 | Search for `<enterprise-app-name>` |
| 6 | Select the app |
| 7 | Save |

### PowerShell Validation

~~~powershell
Get-MgIdentityConditionalAccessPolicy -ConditionalAccessPolicyId $Policy.Id |
Select-Object DisplayName, State -ExpandProperty Conditions
~~~

Expected result:

~~~text
Policy conditions include the target application AppId.
Policy does not accidentally target all cloud apps unless intended.
~~~

## Step_10_Configure_User_And_Group_Targeting

### Recommended Pattern

| Target | Baseline |
|---|---|
| Include | Pilot group first |
| Exclude | Break-glass group |
| Avoid | Directly targeting all users during first build |
| Avoid | Excluding broad groups like all admins unless justified |
| Later rollout | Expand from pilot group to access group or all assigned users |

### Portal

| Step | Action |
|---:|---|
| 1 | Open Conditional Access policy |
| 2 | Select Users |
| 3 | Include pilot group |
| 4 | Exclude break-glass group |
| 5 | Save |

Expected result:

~~~text
Only pilot users are evaluated initially.
Break-glass accounts are excluded.
~~~

## Step_11_Configure_Grant_Controls

Grant controls define what must be satisfied before access is granted.

### Common Grant Controls

| Control | Use Case |
|---|---|
| Require multifactor authentication | Baseline for sensitive apps |
| Require authentication strength | Stronger MFA such as phishing-resistant methods |
| Require device to be marked as compliant | Managed device requirement |
| Require Microsoft Entra hybrid joined device | Domain/hybrid managed device requirement |
| Require approved client app | Mobile app control where supported |
| Require app protection policy | Mobile app protection |
| Require password change | Risk-based scenarios |
| Block access | High-risk blocking policies |

### Portal

| Step | Action |
|---:|---|
| 1 | Open Conditional Access policy |
| 2 | Select Grant |
| 3 | Choose Grant access |
| 4 | Select Require multifactor authentication |
| 5 | Add device requirement if appropriate |
| 6 | Choose Require all selected controls or Require one of the selected controls |
| 7 | Save |

### Baseline Choices

| App Sensitivity | Recommended Grant Control |
|---|---|
| Standard SaaS | Require MFA |
| Sensitive business app | Require MFA and compliant device |
| Privileged app | Require phishing-resistant MFA or authentication strength |
| Unknown/risky app | Block until reviewed |
| Mobile-only app | Require approved app or app protection policy where supported |

Expected result:

~~~text
Grant controls match the risk of the application.
MFA is required at minimum for sensitive app access.
~~~

## Step_12_Configure_Authentication_Strength_If_Required

Authentication strength allows targeting specific MFA methods.

### Portal

| Step | Action |
|---:|---|
| 1 | Open Conditional Access policy |
| 2 | Select Grant |
| 3 | Select Require authentication strength |
| 4 | Choose built-in or custom authentication strength |
| 5 | Save |

### Recommended Use

| Scenario | Authentication Strength |
|---|---|
| Standard app | Multifactor authentication |
| Sensitive app | Passwordless MFA |
| Admin app | Phishing-resistant MFA |
| High-risk app | Phishing-resistant MFA plus device controls |

Expected result:

~~~text
Policy requires the selected authentication strength for the target app.
Users without acceptable methods are blocked or prompted to register depending tenant configuration.
~~~

## Step_13_Configure_Device_Based_Controls_If_Required

Use device controls only when the business requirement supports managed-device access.

### Portal

| Step | Action |
|---:|---|
| 1 | Open Conditional Access policy |
| 2 | Select Grant |
| 3 | Select Require device to be marked as compliant |
| 4 | Optionally select Require Microsoft Entra hybrid joined device |
| 5 | Choose operator |
| 6 | Save |

### Device Control Decision

| Control | Use When | Caution |
|---|---|---|
| Require compliant device | Intune-managed access required | Blocks unmanaged devices |
| Require hybrid joined device | On-prem AD joined device requirement | Can block cloud-only/mobile users |
| Require one selected control | MFA OR device | Weaker but more flexible |
| Require all selected controls | MFA AND device | Stronger but higher support impact |

Expected result:

~~~text
Device-based access requirement matches the access model.
Pilot users can satisfy the requirement before enforcement.
~~~

## Step_14_Configure_Session_Controls

Session controls affect what happens after authentication.

### Common Session Controls

| Session Control | Use Case |
|---|---|
| Sign-in frequency | Reauthentication interval |
| Persistent browser session | Controls stay-signed-in behavior |
| Use app enforced restrictions | Supported Microsoft cloud app restrictions |
| Use Conditional Access App Control | Defender for Cloud Apps session proxy |
| Disable resilience defaults | Rare high-security scenario |

### Portal

| Step | Action |
|---:|---|
| 1 | Open Conditional Access policy |
| 2 | Select Session |
| 3 | Configure Sign-in frequency if required |
| 4 | Configure Persistent browser session if required |
| 5 | Configure app enforced restrictions if supported |
| 6 | Configure Conditional Access App Control if licensed and required |
| 7 | Save |

Expected result:

~~~text
Session controls match the app sensitivity and user experience requirements.
~~~

## Step_15_Configure_Sign_In_Frequency

Sign-in frequency controls how often users must reauthenticate.

### Recommended Baseline

| App Type | Sign-in Frequency |
|---|---|
| Standard business app | Default or 7 to 30 days |
| Sensitive app | 12 to 24 hours |
| Privileged app | Every time or short interval |
| Kiosk/shared device | Every time or short interval |
| Low-risk app | Default session behavior |

### Portal

| Step | Action |
|---:|---|
| 1 | Open Conditional Access policy |
| 2 | Select Session |
| 3 | Select Sign-in frequency |
| 4 | Choose Periodic reauthentication or Every time |
| 5 | Set hours/days |
| 6 | Save |

Expected result:

~~~text
Users must reauthenticate based on the configured frequency.
Prompt frequency is appropriate for app sensitivity.
~~~

## Step_16_Configure_Persistent_Browser_Session

Persistent browser session controls whether users stay signed in after browser close.

### Recommended Baseline

| Scenario | Setting |
|---|---|
| Sensitive app | Never persistent |
| Shared device | Never persistent |
| Managed dedicated device | Persistent may be acceptable |
| Standard app | Use tenant default unless risk requires otherwise |

### Portal

| Step | Action |
|---:|---|
| 1 | Open Conditional Access policy |
| 2 | Select Session |
| 3 | Select Persistent browser session |
| 4 | Choose Never persistent for sensitive apps |
| 5 | Save |

Expected result:

~~~text
Persistent browser behavior is controlled for the target app.
Sensitive app sessions do not remain persistent unnecessarily.
~~~

## Step_17_Configure_Conditional_Access_App_Control_If_Required

Conditional Access App Control requires Microsoft Defender for Cloud Apps integration and supported app/session routing.

Use this for monitored or restricted browser sessions.

### Portal

| Step | Action |
|---:|---|
| 1 | Open Conditional Access policy |
| 2 | Select Session |
| 3 | Select Use Conditional Access App Control |
| 4 | Choose Monitor only, Block downloads, or Use custom policy |
| 5 | Save |
| 6 | Validate session routes through Defender for Cloud Apps if applicable |

### App Control Decision

| Mode | Use Case |
|---|---|
| Monitor only | Visibility pilot |
| Block downloads | Prevent data exfiltration |
| Use custom policy | Advanced MCAS session policy |
| Disabled | No session proxy requirement |

Expected result:

~~~text
Supported sessions are monitored or controlled through Defender for Cloud Apps.
Unsupported apps are not assumed to be controlled.
~~~

## Step_18_Validate_Report_Only_Results

Do not enforce until report-only results are reviewed.

### Portal

| Step | Action |
|---:|---|
| 1 | Open Entra admin center |
| 2 | Go to Monitoring & health |
| 3 | Select Sign-in logs |
| 4 | Filter by `<test-user-upn>` |
| 5 | Open sign-in event |
| 6 | Select Report-only tab |
| 7 | Review target policy |
| 8 | Confirm result is expected |

### Expected Report-Only Outcomes

| Result | Meaning |
|---|---|
| Report-only: Success | User would satisfy policy |
| Report-only: Failure | User would be blocked or challenged unsuccessfully |
| Report-only: User action required | User would need MFA/device action |
| Not applied | Conditions did not match |
| Interrupted | Additional interaction required |

Expected result:

~~~text
Report-only evaluation shows the policy applies only to intended users and apps.
Unexpected blocks are corrected before enforcement.
~~~

## Step_19_Enable_Conditional_Access_Policy

Enable only after report-only validation.

### Portal

| Step | Action |
|---:|---|
| 1 | Open Conditional Access |
| 2 | Select Policies |
| 3 | Open `<ca-policy-name>` |
| 4 | Confirm users/groups |
| 5 | Confirm exclusions |
| 6 | Confirm target app |
| 7 | Confirm grant controls |
| 8 | Confirm session controls |
| 9 | Set Enable policy to On |
| 10 | Save |

### PowerShell

~~~powershell
Update-MgIdentityConditionalAccessPolicy `
  -ConditionalAccessPolicyId $Policy.Id `
  -State "enabled"

Get-MgIdentityConditionalAccessPolicy -ConditionalAccessPolicyId $Policy.Id |
Select-Object DisplayName, State
~~~

Expected result:

~~~text
Conditional Access policy is enabled.
Target app access is enforced for included users except excluded accounts.
~~~

## Step_20_Test_Enforced_User_Access

Use a test user in the pilot group.

| Step | Action | Expected Result |
|---:|---|---|
| 1 | Open private browser session | Clean session |
| 2 | Browse to target app | Entra sign-in starts |
| 3 | Sign in as `<test-user-upn>` | User authenticates |
| 4 | Complete MFA/device requirement | Grant control satisfied |
| 5 | Access app | App opens |
| 6 | Review session behavior | Sign-in frequency/persistence matches policy |
| 7 | Review sign-in logs | CA policy shows success |

Expected result:

~~~text
Included test user can access the app only after satisfying the Conditional Access controls.
Sign-in logs show policy applied successfully.
~~~

## Step_21_Test_Excluded_Break_Glass_Access

Use caution. Do not expose break-glass credentials casually.

| Step | Action | Expected Result |
|---:|---|---|
| 1 | Confirm break-glass account is in exclusion group | Account is excluded |
| 2 | Attempt sign-in only if approved | Sign-in proceeds without this policy applying |
| 3 | Review sign-in logs | Policy shows not applied due to exclusion |
| 4 | Document validation | Break-glass exclusion confirmed |

Expected result:

~~~text
Break-glass account is excluded from the policy.
Emergency access path is preserved.
~~~

## Step_22_Create_Access_Review_For_App_Access_Group

For most apps, review the group that grants app access.

### Portal

| Step | Action |
|---:|---|
| 1 | Open Entra admin center |
| 2 | Go to Identity Governance |
| 3 | Select Access reviews |
| 4 | Select New access review |
| 5 | Select Teams + Groups or Applications depending scenario |
| 6 | Choose `<app-access-group-name>` or `<enterprise-app-name>` |
| 7 | Select reviewers |
| 8 | Configure recurrence |
| 9 | Configure auto-apply if approved by policy |
| 10 | Start review |

### Recommended Access Review Settings

| Setting | Baseline |
|---|---|
| Review target | App access group |
| Reviewers | Group owners or selected reviewers |
| Recurrence | Quarterly for sensitive apps |
| Duration | 7 to 14 days |
| Require reason on approval | Yes |
| Mail notifications | Enabled |
| Reminders | Enabled |
| Auto-apply results | Use carefully |
| If reviewers do not respond | Remove access for high-risk apps, no change for lower-risk apps depending policy |
| Justification | Required |

Expected result:

~~~text
Access review exists for application access.
Reviewers can approve or deny continued access.
~~~

## Step_23_Create_Access_Review_For_Enterprise_Application_Assignments

Use this when reviewing direct application assignments instead of only group membership.

### Portal

| Step | Action |
|---:|---|
| 1 | Open Identity Governance |
| 2 | Select Access reviews |
| 3 | Select New access review |
| 4 | Select Applications if available in tenant/license |
| 5 | Choose `<enterprise-app-name>` |
| 6 | Select review scope |
| 7 | Select reviewers |
| 8 | Configure recurrence and duration |
| 9 | Start review |

Expected result:

~~~text
Application assignment review is created where licensing and app support allow it.
Direct assignments and app access can be reviewed.
~~~

## Step_24_Validate_Access_Review_Workflow

| Step | Action | Expected Result |
|---:|---|---|
| 1 | Sign in as reviewer | Reviewer has review access |
| 2 | Open My Access or Entra access reviews | Review appears |
| 3 | Review test user | User access decision visible |
| 4 | Approve or deny with justification | Decision saved |
| 5 | Review final result | Access result is recorded |
| 6 | Apply result if configured | Access is changed or retained |

Expected result:

~~~text
Reviewer can complete the access review.
Review decisions are recorded and auditable.
~~~

## Step_25_Review_Sign_In_Logs

### Portal

| Step | Action |
|---:|---|
| 1 | Open Entra admin center |
| 2 | Go to Monitoring & health |
| 3 | Select Sign-in logs |
| 4 | Filter by `<enterprise-app-name>` |
| 5 | Filter by `<test-user-upn>` |
| 6 | Open sign-in event |
| 7 | Review Conditional Access tab |
| 8 | Review Session controls |
| 9 | Review Failure reason if failed |

### PowerShell

~~~powershell
Get-MgAuditLogSignIn -Filter "appDisplayName eq '$EnterpriseAppName'" -Top 20 |
Select-Object CreatedDateTime, UserPrincipalName, AppDisplayName, Status, ConditionalAccessStatus
~~~

Expected result:

~~~text
Sign-in logs show whether the policy applied, succeeded, failed, or did not apply.
Session behavior and grant controls are visible in the event details.
~~~

## Step_26_Review_Conditional_Access_Policies

~~~powershell
Get-MgIdentityConditionalAccessPolicy |
Select-Object Id, DisplayName, State, CreatedDateTime, ModifiedDateTime
~~~

Expected result:

~~~text
Conditional Access policy exists and is in the expected state.
Report-only and enabled policies are distinguishable.
~~~

## Step_27_Document_Final_State

| Field | Value |
|---|---|
| Conditional Access policy name | `<ca-policy-name>` |
| Conditional Access policy ID | `<ca-policy-id>` |
| Policy state | Report-only / Enabled / Disabled |
| Target app | `<enterprise-app-name>` |
| Target AppId | `<application-client-id>` |
| Included users/groups | `<included-users-groups>` |
| Excluded users/groups | `<excluded-users-groups>` |
| Break-glass excluded | Yes / No |
| Grant controls | `<grant-controls>` |
| Authentication strength | `<authentication-strength>` |
| Device controls | `<device-controls>` |
| Session controls | `<session-controls>` |
| Sign-in frequency | `<frequency>` |
| Persistent browser setting | `<persistent-browser-setting>` |
| Conditional Access App Control | `<enabled-mode-or-disabled>` |
| Report-only validation | Success / Needs change |
| Enforced validation | Success / Failed |
| Access review name | `<access-review-name>` |
| Access review scope | `<group-or-app>` |
| Access review recurrence | `<recurrence>` |
| Reviewer | `<reviewer-upn>` |
| Sign-in logs reviewed | Yes / No |
| Validation date | `<yyyy-mm-dd>` |
| Validated by | `<admin-upn>` |

## Verification_Commands

### Confirm Enterprise App

~~~powershell
Get-MgServicePrincipal -ServicePrincipalId $Sp.Id |
Select-Object DisplayName, Id, AppId, AccountEnabled, AppRoleAssignmentRequired
~~~

### Confirm App Assignments

~~~powershell
Get-MgServicePrincipalAppRoleAssignedTo -ServicePrincipalId $Sp.Id -All |
Select-Object PrincipalDisplayName, PrincipalType, AppRoleId, ResourceDisplayName
~~~

### Confirm Pilot Group

~~~powershell
Get-MgGroup -GroupId $PilotGroup.Id |
Select-Object DisplayName, Id
~~~

### Confirm Exclusion Group

~~~powershell
Get-MgGroup -GroupId $ExclusionGroup.Id |
Select-Object DisplayName, Id
~~~

### Confirm Exclusion Group Members

~~~powershell
Get-MgGroupMember -GroupId $ExclusionGroup.Id -All |
Select-Object Id, AdditionalProperties
~~~

### Confirm Conditional Access Policy

~~~powershell
Get-MgIdentityConditionalAccessPolicy -ConditionalAccessPolicyId $Policy.Id |
Format-List Id, DisplayName, State, CreatedDateTime, ModifiedDateTime
~~~

### Confirm Recent App Sign-ins

~~~powershell
Get-MgAuditLogSignIn -Filter "appDisplayName eq '$EnterpriseAppName'" -Top 20 |
Select-Object CreatedDateTime, UserPrincipalName, AppDisplayName, Status, ConditionalAccessStatus
~~~

### Confirm Conditional Access Policies List

~~~powershell
Get-MgIdentityConditionalAccessPolicy |
Select-Object DisplayName, State, Id
~~~

## Failure_Checks

| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| Policy does not apply | Wrong app target | CA policy target resources | Select correct Enterprise Application |
| Policy does not apply | User not in included group | User/group conditions | Add user to pilot/included group |
| Policy does not apply | User is excluded | Exclusion group membership | Remove accidental exclusion |
| Break-glass blocked | Break-glass not excluded | Exclusion group and sign-in logs | Add break-glass to exclusion group |
| Everyone blocked | Targeted all users/all apps too early | Policy conditions | Disable or revert to report-only |
| MFA not prompted | Existing MFA session or sign-in frequency | Sign-in logs/auth details | Test with private session or adjust session controls |
| Device requirement blocks user | Device not compliant or not hybrid joined | Device details in sign-in log | Fix device compliance or adjust policy |
| Report-only shows unexpected failure | Grant/session controls too strict | Report-only tab | Adjust policy before enabling |
| App not visible to user | Enterprise app assignment/visibility issue | Enterprise app users/groups and properties | Assign user or set visible |
| User assigned but blocked | Conditional Access grant failed | Sign-in log CA tab | Satisfy MFA/device or adjust policy |
| CA App Control not active | App unsupported or Defender integration missing | Session details | Confirm Defender for Cloud Apps integration |
| Access review unavailable | Licensing or role issue | Entra license and admin role | Assign proper license/role |
| Reviewer cannot see review | Reviewer not assigned | Access review settings | Add correct reviewer |
| Review result not applied | Auto-apply disabled | Review settings | Manually apply or enable auto-apply in future |
| Removed user still has app access | Direct assignment or another group | Enterprise app assignments | Remove all entitlement paths |

## Rollback

Rollback should preserve evidence and restore access without deleting useful audit history.

### Set Conditional Access Policy Back To Report-Only

~~~powershell
Update-MgIdentityConditionalAccessPolicy `
  -ConditionalAccessPolicyId $Policy.Id `
  -State "enabledForReportingButNotEnforced"

Get-MgIdentityConditionalAccessPolicy -ConditionalAccessPolicyId $Policy.Id |
Select-Object DisplayName, State
~~~

Expected result:

~~~text
Policy no longer enforces access decisions.
Policy still evaluates in report-only mode for troubleshooting.
~~~

### Disable Conditional Access Policy

~~~powershell
Update-MgIdentityConditionalAccessPolicy `
  -ConditionalAccessPolicyId $Policy.Id `
  -State "disabled"

Get-MgIdentityConditionalAccessPolicy -ConditionalAccessPolicyId $Policy.Id |
Select-Object DisplayName, State
~~~

Expected result:

~~~text
Policy is disabled.
It no longer evaluates or enforces sign-ins.
~~~

### Remove Target App From Policy

Use this only if the policy should remain for other apps.

~~~powershell
$ExistingPolicy = Get-MgIdentityConditionalAccessPolicy -ConditionalAccessPolicyId $Policy.Id

$Conditions = $ExistingPolicy.Conditions

$Conditions.Applications.IncludeApplications = @(
    $Conditions.Applications.IncludeApplications |
    Where-Object { $_ -ne $Sp.AppId }
)

Update-MgIdentityConditionalAccessPolicy `
  -ConditionalAccessPolicyId $Policy.Id `
  -Conditions $Conditions
~~~

Expected result:

~~~text
Target app is no longer included in the Conditional Access policy.
Policy may still apply to other configured apps.
~~~

### Remove Test User From Pilot Group

~~~powershell
$TestUser = Get-MgUser -UserId $TestUserUpn

Remove-MgGroupMemberByRef `
  -GroupId $PilotGroup.Id `
  -DirectoryObjectId $TestUser.Id
~~~

Expected result:

~~~text
Test user is removed from pilot targeting group.
Policy no longer applies through that group unless user is included another way.
~~~

### Stop Or Close Access Review

Use the portal unless automating identity governance.

~~~text
Portal path:
Entra admin center > Identity Governance > Access reviews
Open <access-review-name>
Stop, delete, or let review expire based on governance decision
~~~

Expected result:

~~~text
Access review no longer collects decisions if stopped.
Existing review history should be retained where required.
~~~

## Security_Baseline

| Control | Baseline |
|---|---|
| Policy deployment | Report-only first |
| App targeting | Specific cloud app |
| Included users | Pilot group before broad rollout |
| Exclusions | Break-glass group required |
| Assignment required | Enabled for controlled Enterprise Applications |
| Grant controls | MFA minimum |
| Sensitive apps | MFA plus compliant device or authentication strength |
| Privileged apps | Phishing-resistant authentication strength |
| Session controls | Use sign-in frequency for sensitive apps |
| Persistent browser | Never persistent for sensitive/shared scenarios |
| CA App Control | Use where licensed and supported |
| Access reviews | Required for sensitive app access |
| Review cadence | Quarterly or stricter |
| Audit validation | Sign-in logs reviewed before and after enforcement |
| Rollback | Report-only or disabled state preferred over deletion |

## Admin_Notes

~~~text
Conditional Access policy name:
Policy ID:
Policy state:
Target Enterprise Application:
Target AppId:
Included users/groups:
Excluded users/groups:
Break-glass accounts excluded:
Grant controls:
Authentication strength:
Device controls:
Session controls:
Sign-in frequency:
Persistent browser session:
Conditional Access App Control:
Pilot validation result:
Enforcement validation result:
Access review name:
Access review target:
Access review reviewers:
Access review recurrence:
Review result application:
Sign-in logs reviewed:
Known exceptions:
Rollback tested:
~~~

## Related_Labs

| Workbook | Relationship |
|---|---|
| 01_Configure_Enterprise_Applications_Service_Principals_And_Assignments.md | Covers app assignments and assignment required |
| 02_Configure_App_Registrations_API_Permissions_Secrets_Certificates_And_Owners.md | Covers app registration and permissions |
| 03_Configure_Admin_Consent_User_Consent_And_Publisher_Verification_Baseline.md | Covers consent governance |
| 04_Configure_SAML_OIDC_OAuth_SSO_And_Application_Provisioning.md | Covers SSO and provisioning |
| 06_Troubleshoot_Enterprise_App_SSO_Consent_Provisioning_And_API_Permission_Issues.md | Covers troubleshooting app access, consent, SSO, and provisioning |
