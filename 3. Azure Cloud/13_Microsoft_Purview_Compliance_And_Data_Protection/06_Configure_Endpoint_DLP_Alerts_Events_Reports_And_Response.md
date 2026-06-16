# 06_Configure_Endpoint_DLP_Alerts_Events_Reports_And_Response

## Objective

Configure Microsoft Purview Endpoint DLP for endpoint activity monitoring, policy enforcement, alerts, events, reports, and response workflows.

This workbook covers:

- Endpoint DLP prerequisites
- Device onboarding dependencies
- Endpoint DLP settings
- Device group planning
- Policy scoping
- Endpoint DLP rule creation
- Removable media controls
- Clipboard controls
- Print controls
- Network share controls
- Cloud egress controls
- Browser and domain restrictions
- Endpoint DLP alerts
- Activity Explorer events
- Reports and evidence collection
- Response actions
- Troubleshooting and rollback

## Lab Context

This workbook builds on the Microsoft Purview DLP baseline.

Complete these first:

- 01_Configure_Purview_Admin_Portal_Roles_Audit_And_Compliance_Baseline.md
- 02_Configure_Sensitive_Info_Types_Trainable_Classifiers_And_Data_Classification.md
- 03_Configure_Sensitivity_Labels_Label_Policies_And_Encryption_Settings.md
- 04_Configure_Retention_Labels_Retention_Policies_And_Data_Lifecycle.md
- 05_Configure_DLP_For_Exchange_SharePoint_OneDrive_Teams_PowerBI_And_Copilot.md

Endpoint DLP extends DLP controls to Windows endpoints and supported endpoint activity.

This workbook focuses on a practical baseline:

- Confirm device onboarding
- Enable endpoint DLP settings
- Scope to pilot devices and users
- Use test mode or audit mode first
- Validate endpoint events
- Review alerts and reports
- Tune before enforcement
- Respond to risky activity

## Prerequisites

| Requirement | Notes |
|---|---|
| Microsoft Purview portal access | Required |
| Microsoft Defender portal access | Recommended |
| Compliance Administrator | Required for broad Purview configuration |
| DLP Compliance Management role | Recommended |
| Security Administrator | Recommended for Defender device visibility |
| Audit Reader or Audit Manager | Recommended |
| Microsoft 365 licensing for Endpoint DLP | Required |
| Microsoft Defender for Endpoint onboarding | Required for supported endpoint visibility |
| Windows test endpoint | Required |
| Supported endpoint operating system | Required |
| Test user account | Required |
| Test security group | Recommended |
| Test device group | Recommended |
| Fake sample data | Required |
| No real sensitive data | Required |
| Exchange Online PowerShell module | Recommended |
| Microsoft Graph PowerShell SDK | Optional |

## Topology / Scope

| Component | Purpose |
|---|---|
| Microsoft Purview portal | Endpoint DLP policy and activity review |
| Microsoft Defender portal | Device onboarding and device inventory validation |
| Endpoint DLP settings | Endpoint control baseline |
| Windows endpoint | DLP activity test device |
| Microsoft Defender for Endpoint | Endpoint onboarding and device signal dependency |
| Data loss prevention policy | DLP rule enforcement |
| Sensitive information types | Endpoint DLP detection signals |
| Sensitivity labels | Endpoint DLP detection and policy conditions |
| Activity Explorer | Endpoint DLP event review |
| Alerts | DLP incident triage |
| Audit | Administrative and activity validation |
| Removable media | USB and external storage test path |
| Clipboard | Copy/paste control test path |
| Print | Printing control test path |
| Network share | Copy to SMB/network location test path |
| Cloud upload | Browser and cloud egress control test path |

## Naming Standards

| Object Type | Naming Example |
|---|---|
| Endpoint DLP policy | DLP-POL-Endpoint-SensitiveData-Test |
| Endpoint DLP rule | DLP-RULE-Endpoint-LabEmployeeID |
| Test security group | GRP-Purview-EndpointDLP-Test-Users |
| Test device group | DG-Purview-EndpointDLP-Test-Devices |
| Test endpoint | WIN11-DLP-TEST01 |
| Test document | EndpointDLP_Test_LabEmployeeID.docx |
| Evidence export | Purview_EndpointDLP_YYYYMMDD.csv |
| Evidence folder | Purview/Endpoint-DLP |
| Incident mailbox | compliance-admin@contoso.com |

## Endpoint DLP Strategy

| Phase | Purpose | Action |
|---|---|---|
| Phase 1 | Readiness | Confirm device onboarding and licensing |
| Phase 2 | Visibility | Enable endpoint DLP in audit/test mode |
| Phase 3 | Pilot | Scope to test users and devices |
| Phase 4 | Notify | Enable user notifications and policy tips |
| Phase 5 | Alert | Enable alerts and incident reports |
| Phase 6 | Restrict | Block or warn on risky endpoint actions |
| Phase 7 | Tune | Reduce false positives |
| Phase 8 | Enforce | Expand only after validation |

## Endpoint DLP Design Principles

| Principle | Requirement |
|---|---|
| Start with pilot devices | Required |
| Start with test/audit mode | Required |
| Use fake test data | Required |
| Avoid tenant-wide blocking first | Required |
| Validate device onboarding | Required |
| Validate each activity type separately | Required |
| Document user experience | Required |
| Document alert behavior | Required |
| Tune false positives before enforcement | Required |
| Coordinate with help desk | Required |
| Coordinate with endpoint/security team | Required |
| Keep rollback plan ready | Required |

## Test Data Rules

| Rule | Requirement |
|---|---|
| Use fake data only | Required |
| Do not use real employee data | Required |
| Do not use real customer data | Required |
| Do not use real health data | Required |
| Do not use real financial data | Required |
| Do not test blocking on production devices first | Required |
| Use dedicated test device | Required |
| Use dedicated test user | Required |
| Document cleanup | Required |

## Fake Test Data Examples

| Data Type | Fake Example |
|---|---|
| Lab employee ID | LAB-EMP-10001 |
| Lab contract ID | LAB-CONTRACT-77881 |
| Lab project code | LAB-PROJ-FIN-2026 |
| Lab DLP marker | PURVIEW-ENDPOINT-DLP-TEST-DATA-ONLY |
| Lab endpoint marker | LAB-ENDPOINT-DLP-TEST |
| Test file name | EndpointDLP_Test_LabEmployeeID.docx |

## Endpoint DLP Activity Types

| Activity | Purpose |
|---|---|
| Copy to removable USB | Detect or block sensitive data copied to removable media |
| Copy to network share | Detect or block sensitive data copied to SMB/network locations |
| Print | Detect or block printing of sensitive content |
| Copy to clipboard | Detect or block copy/paste of sensitive content |
| Upload to cloud service | Detect or block upload to restricted domains |
| Access from unallowed browser | Detect or block activity from unsupported or restricted browser flows |
| Copy to remote desktop session | Detect or block movement through remote session paths where supported |
| App access | Control sensitive content interaction by application where supported |

## Portal / GUI Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|---|---|---|---|---|---|
| 1 | Open Microsoft Purview portal | Admin workstation | https://purview.microsoft.com | N/A | Purview portal opens |
| 2 | Confirm correct admin account | Admin workstation | Profile menu | N/A | Dedicated admin account is signed in |
| 3 | Open Data loss prevention | Admin workstation | Solutions > Data loss prevention | N/A | DLP page opens |
| 4 | Open Endpoint DLP settings | Admin workstation | Data loss prevention > Endpoint DLP settings | N/A | Endpoint DLP settings page opens |
| 5 | Review device onboarding dependency | Admin workstation | Endpoint DLP settings | N/A | Device onboarding requirement is understood |
| 6 | Open Microsoft Defender portal | Admin workstation | https://security.microsoft.com | N/A | Defender portal opens |
| 7 | Review device inventory | Admin workstation | Assets > Devices | N/A | Test endpoint appears if onboarded |
| 8 | Search for test endpoint | Admin workstation | WIN11-DLP-TEST01 | N/A | Test device is found |
| 9 | Confirm device health | Admin workstation | Device page | N/A | Device is active and reporting |
| 10 | Return to Purview Endpoint DLP settings | Admin workstation | Purview > DLP settings | N/A | Endpoint settings are available |
| 11 | Review browser and domain settings | Admin workstation | Endpoint DLP settings | N/A | Browser/domain controls are visible |
| 12 | Configure restricted service domains if needed | Admin workstation | Add cloud service domain | N/A | Domain list is configured |
| 13 | Configure allowed browser behavior | Admin workstation | Browser settings | N/A | Browser behavior is configured |
| 14 | Review file path exclusions if needed | Admin workstation | Endpoint settings | N/A | Exclusions are documented |
| 15 | Review app restrictions if needed | Admin workstation | Restricted apps / app groups | N/A | App control planning is documented |
| 16 | Create endpoint DLP policy | Admin workstation | DLP > Policies > Create policy | N/A | Policy wizard opens |
| 17 | Select custom policy | Admin workstation | Custom policy | N/A | Custom policy is selected |
| 18 | Name endpoint policy | Admin workstation | DLP-POL-Endpoint-SensitiveData-Test | N/A | Policy name is accepted |
| 19 | Add policy description | Admin workstation | Endpoint DLP pilot for fake lab sensitive data | N/A | Description is saved |
| 20 | Choose policy scope | Admin workstation | Admin units or full directory depending tenant | N/A | Scope is selected |
| 21 | Select endpoint devices location | Admin workstation | Devices | N/A | Endpoint location is selected |
| 22 | Scope users or groups | Admin workstation | GRP-Purview-EndpointDLP-Test-Users | N/A | Test users are scoped |
| 23 | Scope devices if supported | Admin workstation | DG-Purview-EndpointDLP-Test-Devices | N/A | Test devices are scoped |
| 24 | Create rule | Admin workstation | New rule | N/A | Rule wizard opens |
| 25 | Name rule | Admin workstation | DLP-RULE-Endpoint-LabEmployeeID | N/A | Rule name is accepted |
| 26 | Add sensitive info type condition | Admin workstation | SIT-Lab-Employee-ID | N/A | Custom SIT is selected |
| 27 | Add sensitivity label condition if needed | Admin workstation | Highly Confidential | N/A | Label condition is selected if needed |
| 28 | Configure instance count | Admin workstation | 1 or more | N/A | Threshold is configured |
| 29 | Configure confidence threshold | Admin workstation | Medium or High | N/A | Confidence is configured |
| 30 | Configure audit action | Admin workstation | Audit or test mode | N/A | Audit-only behavior is selected |
| 31 | Configure user notification | Admin workstation | Notify user | N/A | User notification is enabled |
| 32 | Configure policy tip text | Admin workstation | Explain endpoint data handling | N/A | Policy tip is configured |
| 33 | Configure user override | Admin workstation | Allow override with justification if needed | N/A | Override behavior is configured |
| 34 | Configure USB action | Admin workstation | Audit, warn, or block copy to removable media | N/A | Removable media behavior is configured |
| 35 | Configure print action | Admin workstation | Audit, warn, or block printing | N/A | Print behavior is configured |
| 36 | Configure clipboard action | Admin workstation | Audit, warn, or block clipboard copy | N/A | Clipboard behavior is configured |
| 37 | Configure network share action | Admin workstation | Audit, warn, or block copy to network share | N/A | Network share behavior is configured |
| 38 | Configure cloud upload action | Admin workstation | Audit, warn, or block upload to restricted domain | N/A | Cloud upload behavior is configured |
| 39 | Configure alert | Admin workstation | Alert on policy match | N/A | Alert is enabled |
| 40 | Configure incident report | Admin workstation | Send report to compliance admin | N/A | Incident report is enabled |
| 41 | Review rule | Admin workstation | Review settings | N/A | Rule configuration is correct |
| 42 | Save rule | Admin workstation | Save | N/A | Rule is saved |
| 43 | Select policy mode | Admin workstation | Test mode or audit mode | N/A | Enforcement is not broadly enabled |
| 44 | Review and create policy | Admin workstation | Submit | N/A | Endpoint DLP policy is created |
| 45 | Wait for policy propagation | Admin workstation | N/A | N/A | Policy starts applying after propagation |
| 46 | Sign in to test endpoint | Test endpoint | WIN11-DLP-TEST01 | N/A | Test user signs in |
| 47 | Create fake test document | Test endpoint | Word or Notepad | N/A | Test file is created |
| 48 | Add fake sensitive data | Test endpoint | LAB-EMP-10001 PURVIEW-ENDPOINT-DLP-TEST-DATA-ONLY | N/A | Test content is added |
| 49 | Save test file locally | Test endpoint | Documents folder | N/A | File saves |
| 50 | Test copy to USB | Test endpoint | Copy file to removable media | N/A | Audit/warn/block behavior occurs |
| 51 | Test print action | Test endpoint | Print to PDF or printer if safe | N/A | Audit/warn/block behavior occurs |
| 52 | Test clipboard copy | Test endpoint | Copy sensitive text | N/A | Audit/warn/block behavior occurs |
| 53 | Test network share copy | Test endpoint | Copy to test SMB share | N/A | Audit/warn/block behavior occurs |
| 54 | Test browser upload if safe | Test endpoint | Upload to restricted test domain | N/A | Audit/warn/block behavior occurs |
| 55 | Review Activity Explorer | Admin workstation | Purview > Activity Explorer | N/A | Endpoint events appear after processing |
| 56 | Review DLP alerts | Admin workstation | Purview DLP alerts or Microsoft 365 Defender alerts | N/A | Alert appears if configured |
| 57 | Review DLP reports | Admin workstation | DLP reports | N/A | Endpoint DLP reporting is visible |
| 58 | Review audit search | Admin workstation | Audit > Search | N/A | Endpoint DLP activity appears if available |
| 59 | Tune policy if needed | Admin workstation | Edit DLP policy/rule | N/A | False positives are reduced |
| 60 | Document validation evidence | Admin workstation | Evidence folder | N/A | Evidence is saved |

## PowerShell / CLI Validation

## Install Required Modules

    Install-Module ExchangeOnlineManagement -Scope CurrentUser
    Install-Module Microsoft.Graph -Scope CurrentUser

Expected result:

    Required modules install successfully or already exist.

## Connect to Exchange Online

    Connect-ExchangeOnline

Expected result:

    Exchange Online PowerShell connects successfully.

## Connect to Microsoft Graph

    Connect-MgGraph -Scopes "Directory.Read.All","User.Read.All","Group.Read.All","Device.Read.All","AuditLog.Read.All"

Expected result:

    Microsoft Graph connects successfully with requested scopes.

## Validate Graph Context

    Get-MgContext

Expected result:

    Connected tenant, account, and scopes are displayed.

## Review Endpoint DLP and Compliance Role Groups

    Get-RoleGroup | Where-Object {
        $_.Name -match "Compliance|DLP|Data Loss|Information|Protection|Audit"
    } | Select-Object Name

Expected result:

    DLP, compliance, and information protection role groups are listed.

## Review Compliance Management Members

    Get-RoleGroupMember "Compliance Management"

Expected result:

    Members of Compliance Management are displayed.

## Review DLP Role Groups

    Get-RoleGroup | Where-Object {
        $_.Name -match "DLP|Data Loss"
    } | Select-Object Name

Expected result:

    DLP-related role groups are listed.

## Review Devices in Microsoft Graph

    Get-MgDevice -Top 20 | Select-Object DisplayName,Id,OperatingSystem,ApproximateLastSignInDateTime

Expected result:

    Tenant devices are listed if Graph permissions and device objects are available.

## Search for Test Device

    Get-MgDevice -Filter "displayName eq 'WIN11-DLP-TEST01'" | Select-Object DisplayName,Id,OperatingSystem,ApproximateLastSignInDateTime

Expected result:

    Test device is returned if it exists in Entra ID device inventory.

## Search Audit Log for Endpoint DLP Administration

    Search-UnifiedAuditLog `
        -StartDate (Get-Date).AddDays(-7) `
        -EndDate (Get-Date) `
        -ResultSize 500 |
    Where-Object {
        $_.Operations -match "Dlp|DLP|Endpoint|Device|DataLoss|Policy|Compliance|Rule"
    }

Expected result:

    Recent Endpoint DLP or compliance administration events appear if available.

## Export Endpoint DLP Administration Audit Events

    $Date = Get-Date -Format "yyyyMMdd"
    $OutputPath = ".\Purview_EndpointDLP_Admin_Audit_$Date.csv"

    Search-UnifiedAuditLog `
        -StartDate (Get-Date).AddDays(-7) `
        -EndDate (Get-Date) `
        -ResultSize 500 |
    Where-Object {
        $_.Operations -match "Dlp|DLP|Endpoint|Device|DataLoss|Policy|Compliance|Rule"
    } |
    Export-Csv $OutputPath -NoTypeInformation

    Get-Item $OutputPath

Expected result:

    CSV export is created.

## Search Audit Log for Endpoint DLP Test Activity

    Search-UnifiedAuditLog `
        -StartDate (Get-Date).AddDays(-2) `
        -EndDate (Get-Date) `
        -ResultSize 500 |
    Where-Object {
        $_.AuditData -match "PURVIEW-ENDPOINT-DLP-TEST|LAB-EMP|Endpoint|DLP|DataLoss"
    }

Expected result:

    Matching endpoint DLP test activity appears if audit records are available.

## Export Endpoint DLP Role Groups for Evidence

    $Date = Get-Date -Format "yyyyMMdd"
    $OutputPath = ".\Purview_EndpointDLP_RoleGroups_$Date.csv"

    Get-RoleGroup | Where-Object {
        $_.Name -match "Compliance|DLP|Data Loss|Information|Protection|Audit"
    } | Select-Object Name,ManagedBy,Roles |
    Export-Csv $OutputPath -NoTypeInformation

    Get-Item $OutputPath

Expected result:

    CSV export is created.

## Disconnect Sessions

    Disconnect-ExchangeOnline -Confirm:$false
    Disconnect-MgGraph

Expected result:

    PowerShell sessions disconnect cleanly.

## Endpoint DLP Concepts

| Concept | Meaning |
|---|---|
| Endpoint DLP | DLP controls applied to supported endpoint activities |
| Onboarded device | Endpoint reporting into required Microsoft security services |
| Device location | DLP policy location for endpoint devices |
| Device group | Logical set of devices used for targeting or reporting |
| Activity Explorer | Purview view of user and content activity |
| DLP alert | Alert generated by DLP policy match |
| User notification | End-user message shown during policy match |
| Policy tip | Contextual guidance shown to user |
| Override | Allows user to continue with justification if configured |
| Removable media control | USB or external storage control |
| Print control | Printing sensitive content control |
| Clipboard control | Copy/paste sensitive content control |
| Network share control | Copy to SMB/network share control |
| Cloud upload control | Upload to restricted service/domain control |
| Audit mode | Records activity without blocking |
| Warn mode | Warns user before action |
| Block mode | Prevents user action |
| False positive | Non-sensitive action incorrectly matched |
| False negative | Sensitive action missed |

## Endpoint DLP Policy Baseline

| Setting | Lab Value |
|---|---|
| Policy name | DLP-POL-Endpoint-SensitiveData-Test |
| Rule name | DLP-RULE-Endpoint-LabEmployeeID |
| Location | Devices |
| User scope | GRP-Purview-EndpointDLP-Test-Users |
| Device scope | DG-Purview-EndpointDLP-Test-Devices |
| Condition | SIT-Lab-Employee-ID |
| Supporting marker | PURVIEW-ENDPOINT-DLP-TEST-DATA-ONLY |
| Mode | Test/audit first |
| USB action | Audit or warn first |
| Print action | Audit or warn first |
| Clipboard action | Audit or warn first |
| Network share action | Audit or warn first |
| Cloud upload action | Audit or warn first |
| Alerts | Enabled |
| Incident reports | Enabled |

## Endpoint Activity Validation Matrix

| Activity | Test Action | Expected Result |
|---|---|---|
| Removable media | Copy fake sensitive file to USB | Audit, warn, or block |
| Print | Print fake sensitive document | Audit, warn, or block |
| Clipboard | Copy fake sensitive text | Audit, warn, or block |
| Network share | Copy fake sensitive file to SMB share | Audit, warn, or block |
| Cloud upload | Upload fake sensitive file to restricted domain | Audit, warn, or block |
| Local save | Save fake sensitive file locally | Usually allowed unless policy restricts |
| App access | Open file in test app | Activity may be audited depending configuration |

## Browser and Domain Planning

| Area | Planning Item |
|---|---|
| Restricted domains | Identify cloud services where upload should be audited or blocked |
| Allowed domains | Identify trusted business services |
| Browser support | Validate supported browsers and behavior |
| Unallowed browser behavior | Decide audit, warn, or block |
| Business exceptions | Document required exceptions |
| Pilot validation | Test with real user workflow using fake data |

## Device Readiness Checklist

| Check | Expected Result |
|---|---|
| Device appears in Defender portal | Yes |
| Device appears active | Yes |
| Device user is test user | Yes |
| Device is in pilot group | Yes |
| Required endpoint security components healthy | Yes |
| User can receive policy tips | Yes |
| Activity appears in Activity Explorer | Yes |
| Alerts appear after test activity | Yes |

## Validation Tests

## Test 1: Endpoint DLP Portal Access

| Step | Action | Expected Result |
|---|---|---|
| 1 | Open Purview portal | Portal opens |
| 2 | Open Data loss prevention | DLP page opens |
| 3 | Open Endpoint DLP settings | Settings page opens |
| 4 | Open Policies | Policy list appears |

## Test 2: Device Onboarding Validation

| Step | Action | Expected Result |
|---|---|---|
| 1 | Open Defender portal | Portal opens |
| 2 | Open device inventory | Devices list appears |
| 3 | Search WIN11-DLP-TEST01 | Test endpoint appears |
| 4 | Open device page | Device details appear |
| 5 | Confirm active status | Device is reporting |

## Test 3: Create Endpoint DLP Policy

| Step | Action | Expected Result |
|---|---|---|
| 1 | Create custom DLP policy | Wizard opens |
| 2 | Select Devices location | Endpoint location is selected |
| 3 | Scope test user group | Test group is selected |
| 4 | Add SIT-Lab-Employee-ID condition | Condition is configured |
| 5 | Save policy in test mode | Policy is created |

## Test 4: Removable Media Validation

| Step | Action | Expected Result |
|---|---|---|
| 1 | Create fake sensitive file | File is created |
| 2 | Insert LAB-EMP-10001 marker | Sensitive test data is present |
| 3 | Copy file to USB | Audit/warn/block behavior occurs |
| 4 | Review user notification | Notification appears if configured |
| 5 | Review Activity Explorer | Event appears after processing |

## Test 5: Print Validation

| Step | Action | Expected Result |
|---|---|---|
| 1 | Open fake sensitive file | File opens |
| 2 | Attempt print or print to PDF | Print action is tested |
| 3 | Observe policy behavior | Audit/warn/block behavior occurs |
| 4 | Review alert | Alert appears if configured |
| 5 | Document evidence | Evidence is saved |

## Test 6: Clipboard Validation

| Step | Action | Expected Result |
|---|---|---|
| 1 | Open fake sensitive file | File opens |
| 2 | Copy LAB-EMP-10001 text | Clipboard action is tested |
| 3 | Paste into another app | Audit/warn/block behavior occurs |
| 4 | Review Activity Explorer | Event appears if available |
| 5 | Document evidence | Evidence is saved |

## Test 7: Network Share Validation

| Step | Action | Expected Result |
|---|---|---|
| 1 | Prepare test SMB share | Share is available |
| 2 | Copy fake sensitive file to share | Network copy is attempted |
| 3 | Observe policy behavior | Audit/warn/block behavior occurs |
| 4 | Review alerts | Alert appears if configured |
| 5 | Document evidence | Evidence is saved |

## Test 8: Cloud Upload Validation

| Step | Action | Expected Result |
|---|---|---|
| 1 | Configure restricted test domain if safe | Domain setting is configured |
| 2 | Open supported browser | Browser opens |
| 3 | Attempt upload of fake sensitive file | Upload is tested |
| 4 | Observe policy behavior | Audit/warn/block behavior occurs |
| 5 | Review Activity Explorer | Event appears if available |

## Test 9: Alert Validation

| Step | Action | Expected Result |
|---|---|---|
| 1 | Trigger endpoint DLP test activity | Activity occurs |
| 2 | Open DLP alerts | Alerts page opens |
| 3 | Search recent alerts | Test alert appears if configured |
| 4 | Open alert details | User, device, file, and activity metadata appear |
| 5 | Document alert evidence | Evidence is saved |

## Test 10: Report Validation

| Step | Action | Expected Result |
|---|---|---|
| 1 | Open DLP reports | Reports page opens |
| 2 | Filter by endpoint workload | Endpoint activity is filtered |
| 3 | Filter by policy | DLP-POL-Endpoint-SensitiveData-Test appears |
| 4 | Review trend and event details | Report data appears after processing |
| 5 | Export or document | Evidence is captured |

## Test 11: Response Workflow Validation

| Step | Action | Expected Result |
|---|---|---|
| 1 | Open endpoint DLP alert | Alert details open |
| 2 | Identify user and device | User/device are clear |
| 3 | Identify activity type | USB, print, clipboard, upload, or network copy is visible |
| 4 | Determine severity | Triage decision is documented |
| 5 | Record response action | Response notes are saved |

## Troubleshooting

| Issue | Likely Cause | Fix |
|---|---|---|
| Endpoint DLP settings missing | Missing role or license | Confirm DLP Compliance Management and licensing |
| Devices location unavailable | Endpoint DLP not licensed or not enabled | Confirm tenant licensing and endpoint DLP support |
| Device not visible | Not onboarded or not reporting | Confirm Defender onboarding and device health |
| Device visible but no DLP events | Policy not scoped or propagation delay | Confirm user/device scope and wait |
| Test user not affected | User not in policy scope | Confirm group membership |
| Policy does not match content | SIT mismatch or threshold too high | Validate SIT and lower threshold for lab |
| USB activity not detected | Control not configured or device unsupported | Confirm removable media settings |
| Print activity not detected | Print action not configured or unsupported workflow | Confirm print control settings |
| Clipboard activity not detected | Clipboard action not configured | Confirm clipboard settings |
| Browser upload not detected | Browser/domain settings missing | Configure restricted domains and browser behavior |
| Network share copy not detected | Network share action not configured | Confirm network share control |
| User notification missing | Notification not enabled or client behavior unsupported | Confirm rule notification settings |
| Alert not created | Alert not enabled or delay | Confirm alert settings and wait |
| Activity Explorer empty | Delay, no matching events, or missing role | Wait and confirm permissions |
| Incident report not received | Mail flow or recipient issue | Confirm recipient and Exchange delivery |
| Too many false positives | Rule too broad | Add exceptions and refine conditions |
| Legitimate workflow blocked | Enforcement too aggressive | Move to warn/audit and tune |
| PowerShell search returns no results | No matching audit events or delay | Expand time range |
| Role group command fails | Tenant role group name differs | List role groups and use exact name |

## Common Commands

## Connect to Exchange Online

    Connect-ExchangeOnline

## Connect to Microsoft Graph

    Connect-MgGraph -Scopes "Directory.Read.All","User.Read.All","Group.Read.All","Device.Read.All","AuditLog.Read.All"

## Review Endpoint DLP Role Groups

    Get-RoleGroup | Where-Object {
        $_.Name -match "Compliance|DLP|Data Loss|Information|Protection|Audit"
    } | Select-Object Name

## Review Compliance Management Members

    Get-RoleGroupMember "Compliance Management"

## Search for Test Device

    Get-MgDevice -Filter "displayName eq 'WIN11-DLP-TEST01'" | Select-Object DisplayName,Id,OperatingSystem,ApproximateLastSignInDateTime

## Search Recent Endpoint DLP Audit Events

    Search-UnifiedAuditLog `
        -StartDate (Get-Date).AddDays(-7) `
        -EndDate (Get-Date) `
        -ResultSize 500 |
    Where-Object {
        $_.Operations -match "Dlp|DLP|Endpoint|Device|DataLoss|Policy|Compliance|Rule"
    }

## Search Recent Endpoint DLP Test Activity

    Search-UnifiedAuditLog `
        -StartDate (Get-Date).AddDays(-2) `
        -EndDate (Get-Date) `
        -ResultSize 500 |
    Where-Object {
        $_.AuditData -match "PURVIEW-ENDPOINT-DLP-TEST|LAB-EMP|Endpoint|DLP|DataLoss"
    }

## Export Endpoint DLP Audit Evidence

    Search-UnifiedAuditLog `
        -StartDate (Get-Date).AddDays(-7) `
        -EndDate (Get-Date) `
        -ResultSize 500 |
    Where-Object {
        $_.Operations -match "Dlp|DLP|Endpoint|Device|DataLoss|Policy|Compliance|Rule"
    } |
    Export-Csv ".\Purview_EndpointDLP_Admin_Audit.csv" -NoTypeInformation

## Disconnect Sessions

    Disconnect-ExchangeOnline -Confirm:$false
    Disconnect-MgGraph

## Response Workflow

| Phase | Action |
|---|---|
| Detect | Endpoint DLP policy generates event or alert |
| Triage | Review user, device, file, activity, policy, and rule |
| Validate | Determine if event is true positive or false positive |
| Contain | Block, warn, or coach user depending severity |
| Investigate | Review Activity Explorer, audit, and Defender device context |
| Escalate | Notify security, compliance, legal, or HR if required |
| Remediate | Remove risky file, fix sharing path, update user guidance |
| Tune | Adjust DLP rule, exceptions, or thresholds |
| Document | Save evidence and response notes |

## Response Severity Guide

| Severity | Example | Response |
|---|---|---|
| Low | User copies fake sensitive file locally in test | Document and monitor |
| Medium | User attempts copy to USB | Warn, review business need, tune policy |
| High | User attempts upload to unapproved cloud service | Block, investigate, notify security/compliance |
| Critical | Repeated exfiltration attempts across device and cloud paths | Escalate to security incident workflow |

## Rollback / Cleanup

| Change | Rollback |
|---|---|
| Created Endpoint DLP policy | Disable or delete policy |
| Created Endpoint DLP rule | Disable or delete rule |
| Enabled USB control | Return to audit mode or disable |
| Enabled print control | Return to audit mode or disable |
| Enabled clipboard control | Return to audit mode or disable |
| Enabled network share control | Return to audit mode or disable |
| Enabled cloud upload control | Return to audit mode or disable |
| Enabled restricted domains | Remove or adjust domain entries |
| Enabled alerts | Disable alert setting |
| Enabled incident reports | Remove recipients or disable |
| Scoped test group | Remove group from policy |
| Scoped test device | Remove device group from policy |
| Uploaded test files | Delete fake test files |
| Exported audit evidence | Store securely or delete lab-only export |

## Rollback Warnings

| Warning | Explanation |
|---|---|
| Policy changes may take time | Endpoint DLP policy updates require propagation |
| Blocking can disrupt users | Start with audit or warn mode |
| Endpoint events may still appear after policy changes | Activity and audit history remain |
| Device onboarding is separate from DLP policy | Removing policy does not offboard device |
| False positives can reduce trust | Tune before enforcement |
| Domain restrictions can affect business apps | Validate exceptions before rollout |

## Cleanup Test Data

| Location | Cleanup Action |
|---|---|
| Test endpoint local folder | Delete fake sensitive files |
| USB test device | Delete fake test files |
| Test SMB share | Delete fake test files |
| Browser upload destination | Delete test files if uploaded |
| Print queue / PDF output | Delete fake print output |
| Evidence folder | Retain only approved evidence |
| Audit export folder | Secure or delete lab-only CSV files |

## Security Notes

- Do not test Endpoint DLP with real sensitive data.
- Do not enable block mode broadly without pilot validation.
- Endpoint DLP depends on device readiness and reporting health.
- User notifications should clearly explain what happened.
- Incident reports may contain sensitive metadata.
- Activity Explorer and alert details should be restricted to authorized staff.
- USB blocking can affect legitimate workflows.
- Print blocking can affect business operations.
- Cloud upload blocking requires business exception planning.
- Browser and domain restrictions require careful testing.
- Coordinate with endpoint, security, help desk, legal, and compliance teams.
- Document every enforcement decision.
- Use audit and warn modes before blocking.

## Production Readiness Review

| Question | Required Answer |
|---|---|
| Are devices onboarded and healthy? | Yes |
| Has a pilot device group been created? | Yes |
| Has a pilot user group been created? | Yes |
| Has policy been tested in audit mode? | Yes |
| Has USB behavior been tested? | Yes |
| Has print behavior been tested? | Yes |
| Has clipboard behavior been tested? | Yes |
| Has network share behavior been tested? | Yes |
| Has cloud upload behavior been tested? | Yes |
| Are user notifications clear? | Yes |
| Are incident report recipients correct? | Yes |
| Are alerts tuned? | Yes |
| Are false positives documented? | Yes |
| Are business exceptions documented? | Yes |
| Has rollback been documented? | Yes |
| Has help desk guidance been prepared? | Yes |
| Has enforcement been approved? | Yes |

## Evidence Collection

| Evidence Item | Collection Method | File / Location |
|---|---|---|
| Endpoint DLP settings | Screenshot | Evidence folder |
| Defender device inventory | Screenshot | Evidence folder |
| Test device page | Screenshot | Evidence folder |
| Endpoint DLP policy settings | Screenshot | Evidence folder |
| Endpoint DLP rule settings | Screenshot | Evidence folder |
| USB test result | Screenshot or notes | Evidence folder |
| Print test result | Screenshot or notes | Evidence folder |
| Clipboard test result | Screenshot or notes | Evidence folder |
| Network share test result | Screenshot or notes | Evidence folder |
| Cloud upload test result | Screenshot or notes | Evidence folder |
| Activity Explorer endpoint event | Screenshot | Evidence folder |
| DLP alert details | Screenshot | Evidence folder |
| DLP report view | Screenshot | Evidence folder |
| Audit export | PowerShell Export-Csv | Purview_EndpointDLP_Admin_Audit_YYYYMMDD.csv |
| Response notes | Workbook notes | Evidence folder |

## Verification Checklist

| Check | Pass / Fail |
|---|---|
| Purview portal opens |  |
| Data loss prevention page opens |  |
| Endpoint DLP settings page opens |  |
| Defender portal opens |  |
| Test device appears in device inventory |  |
| Test device is active and reporting |  |
| Endpoint DLP policy created |  |
| Device location selected |  |
| Test user group scoped |  |
| Test device group scoped if supported |  |
| SIT-Lab-Employee-ID condition selected |  |
| User notification enabled |  |
| Policy tip enabled |  |
| Incident report enabled |  |
| Alert enabled |  |
| USB action configured |  |
| Print action configured |  |
| Clipboard action configured |  |
| Network share action configured |  |
| Cloud upload action configured |  |
| Policy set to audit/test mode first |  |
| Fake sensitive test file created |  |
| USB test completed |  |
| Print test completed |  |
| Clipboard test completed |  |
| Network share test completed |  |
| Cloud upload test completed |  |
| Activity Explorer reviewed |  |
| Endpoint DLP alert reviewed |  |
| DLP reports reviewed |  |
| Audit search completed |  |
| Audit export created |  |
| Response workflow documented |  |
| Rollback documented |  |
| No real sensitive data used |  |
| No broad enforcement enabled without approval |  |

## Exam / Interview Notes

- Endpoint DLP extends Microsoft Purview DLP controls to supported endpoint activities.
- Endpoint DLP depends on supported device onboarding and reporting.
- Endpoint DLP policies use the Devices location.
- Endpoint DLP can detect sensitive data movement to removable media, printers, clipboard, network shares, cloud services, and other supported endpoint paths.
- Sensitive information types and sensitivity labels can be used as Endpoint DLP detection conditions.
- Endpoint DLP should start in audit or test mode.
- Warn mode helps coach users before full blocking.
- Block mode should be piloted carefully.
- Activity Explorer is used to review endpoint DLP activity.
- Alerts and incident reports support compliance triage.
- Device health matters when troubleshooting missing events.
- Policy scope must include the correct users and devices.
- Endpoint DLP behavior can vary by activity type, browser, app, OS support, and policy mode.
- False positives should be tuned before broad enforcement.
- Endpoint DLP response workflows should involve compliance, security, endpoint operations, and help desk teams.
- Endpoint DLP is part of a broader data protection model that includes labels, DLP, retention, permissions, and monitoring.

## Related Workbooks

| Workbook                                                                           | Relationship                                                           |
| ---------------------------------------------------------------------------------- | ---------------------------------------------------------------------- |
| 01_Configure_Purview_Admin_Portal_Roles_Audit_And_Compliance_Baseline.md           | Provides Purview portal, role, and audit baseline                      |
| 02_Configure_Sensitive_Info_Types_Trainable_Classifiers_And_Data_Classification.md | Provides SIT and classifier detection foundation                       |
| 03_Configure_Sensitivity_Labels_Label_Policies_And_Encryption_Settings.md          | Provides sensitivity label protection context                          |
| 04_Configure_Retention_Labels_Retention_Policies_And_Data_Lifecycle.md             | Provides lifecycle governance context                                  |
| 05_Configure_DLP_For_Exchange_SharePoint_OneDrive_Teams_PowerBI_And_Copilot.md     | Provides Microsoft 365 DLP baseline                                    |
| 07_Monitor_Content_Explorer_Activity_Explorer_Label_Reports_And_DLP_Alerts.md      | Monitors endpoint DLP events, alerts, and reports                      |
| 08_Troubleshoot_Purview_Labeling_Retention_DLP_Audit_And_Compliance_Alerts.md      | Troubleshoots Endpoint DLP, policy, alert, audit, and reporting issues |