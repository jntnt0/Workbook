# 08_Troubleshoot_Purview_Labeling_Retention_DLP_Audit_And_Compliance_Alerts

## Objective

Troubleshoot Microsoft Purview labeling, retention, DLP, Endpoint DLP, audit, Content Explorer, Activity Explorer, reports, and compliance alert issues.

This workbook covers:

- Purview portal access troubleshooting
- Role and permission troubleshooting
- Sensitivity label troubleshooting
- Label policy troubleshooting
- Encryption troubleshooting
- Retention label troubleshooting
- Retention policy troubleshooting
- DLP policy troubleshooting
- Endpoint DLP troubleshooting
- Content Explorer troubleshooting
- Activity Explorer troubleshooting
- Audit search troubleshooting
- DLP alert troubleshooting
- Compliance report troubleshooting
- Service health validation
- Evidence collection
- Rollback and escalation workflow

## Lab Context

This workbook is the troubleshooting workbook for the Purview compliance section.

Complete or reference these workbooks first:

- 01_Configure_Purview_Admin_Portal_Roles_Audit_And_Compliance_Baseline.md
- 02_Configure_Sensitive_Info_Types_Trainable_Classifiers_And_Data_Classification.md
- 03_Configure_Sensitivity_Labels_Label_Policies_And_Encryption_Settings.md
- 04_Configure_Retention_Labels_Retention_Policies_And_Data_Lifecycle.md
- 05_Configure_DLP_For_Exchange_SharePoint_OneDrive_Teams_PowerBI_And_Copilot.md
- 06_Configure_Endpoint_DLP_Alerts_Events_Reports_And_Response.md
- 07_Monitor_Content_Explorer_Activity_Explorer_Label_Reports_And_DLP_Alerts.md

The goal is to isolate whether a Purview issue is caused by:

- Licensing
- Role assignment
- Policy scope
- Policy mode
- Client support
- Workload support
- Device onboarding
- Data classification mismatch
- Label publishing delay
- Retention processing delay
- DLP rule design
- Audit ingestion delay
- Alert configuration
- Service health issue
- User misunderstanding
- Mis-scoped test data

## Prerequisites

| Requirement | Notes |
|---|---|
| Microsoft Purview portal access | Required |
| Microsoft 365 admin center access | Required |
| Microsoft Entra admin center access | Recommended |
| Microsoft Defender portal access | Recommended for Endpoint DLP |
| Compliance Administrator | Recommended |
| DLP Compliance Management role | Recommended |
| Information Protection Admin | Recommended |
| Records Management role | Recommended |
| Audit Reader or Audit Manager | Recommended |
| Content Explorer role | Required for Content Explorer troubleshooting |
| Activity Explorer access | Required for activity troubleshooting |
| Test user account | Required |
| Test endpoint | Required for Endpoint DLP troubleshooting |
| Exchange Online PowerShell module | Required |
| Microsoft Graph PowerShell SDK | Recommended |
| Fake test data | Required |
| Evidence folder | Required |

## Topology / Scope

| Component | Troubleshooting Purpose |
|---|---|
| Microsoft Purview portal | Main compliance configuration and monitoring portal |
| Microsoft 365 admin center | Licensing, service health, and message center |
| Microsoft Entra admin center | User, group, role, and device validation |
| Microsoft Defender portal | Endpoint DLP device and alert handoff |
| Exchange Online PowerShell | Audit and role validation |
| Microsoft Graph PowerShell | User, group, device, and directory validation |
| Office apps | Sensitivity label and policy tip validation |
| Outlook | Email label and DLP validation |
| SharePoint Online | File labels, retention, and DLP validation |
| OneDrive for Business | User file labels, retention, and DLP validation |
| Teams | Message DLP and collaboration validation |
| Endpoint device | Endpoint DLP validation |
| Content Explorer | Sensitive content location validation |
| Activity Explorer | Activity and event validation |
| Audit | Admin and user activity validation |
| Alerts | DLP and compliance triage validation |

## Naming Standards

| Object Type | Naming Example |
|---|---|
| Evidence folder | Purview/Troubleshooting |
| Troubleshooting notes | Purview_Troubleshooting_YYYYMMDD.md |
| Audit export | Purview_Troubleshooting_Audit_YYYYMMDD.csv |
| Role export | Purview_Troubleshooting_RoleGroups_YYYYMMDD.csv |
| Test user | user-purview-test01 |
| Test endpoint | WIN11-DLP-TEST01 |
| Test SharePoint site | SP-Purview-DLP-Test |
| Test policy | DLP-POL-M365-SensitiveData-Test |
| Test endpoint policy | DLP-POL-Endpoint-SensitiveData-Test |
| Test sensitivity label | Highly Confidential |
| Test retention label | RET-Business-Record-7Y |

## Troubleshooting Method

| Phase | Purpose | Action |
|---|---|---|
| Phase 1 | Confirm service health | Check Microsoft 365 service health and message center |
| Phase 2 | Confirm licensing | Verify required compliance features are licensed |
| Phase 3 | Confirm access | Validate role groups and portal access |
| Phase 4 | Confirm scope | Validate users, groups, sites, mailboxes, devices, and policies |
| Phase 5 | Confirm policy mode | Check test mode, audit mode, warn mode, or enforce mode |
| Phase 6 | Confirm signal | Validate SIT, classifier, label, or condition match |
| Phase 7 | Confirm client/workload support | Validate Office, Outlook, Teams, browser, endpoint, or workload behavior |
| Phase 8 | Confirm processing delay | Allow for policy, audit, reporting, and alert propagation |
| Phase 9 | Collect evidence | Export audit, screenshots, and event details |
| Phase 10 | Tune or rollback | Adjust policy, scope, rule, or role assignment |

## Troubleshooting Principles

| Principle | Requirement |
|---|---|
| Check service health before deep troubleshooting | Required |
| Verify licensing before policy troubleshooting | Required |
| Verify roles before assuming portal defect | Required |
| Validate policy scope before changing rule logic | Required |
| Validate test data before changing conditions | Required |
| Validate policy mode before assuming enforcement failure | Required |
| Validate workload support before assuming policy failure | Required |
| Allow for propagation delay | Required |
| Use fake test data only | Required |
| Capture evidence before making changes | Required |
| Change one variable at a time | Required |
| Document all findings | Required |

## Quick Triage Matrix

| Symptom | Most Likely Cause | First Check |
|---|---|---|
| Purview page missing | Role or license issue | Role group membership |
| Sensitivity label not visible | Label policy scope or propagation | Label policy target |
| Label applies but encryption fails | Permission model or client issue | Label encryption settings |
| DLP policy not matching | Condition, scope, or mode issue | Policy rule and test data |
| Endpoint DLP not firing | Device onboarding or scope issue | Defender device inventory |
| Retention label not visible | Label policy scope or propagation | Retention label policy |
| Audit search empty | No activity, wrong filter, or delay | Broaden time range |
| Alert not generated | Alert setting disabled or delay | Rule alert configuration |
| Content Explorer empty | No indexed content or role issue | Explorer permissions and filters |
| Activity Explorer empty | No activity or reporting delay | Filters and time range |
| User cannot access protected file | Encryption permissions | Label permission model |
| User can access protected file unexpectedly | Label not applied or encryption not active | File label details |

## Portal / GUI Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|---|---|---|---|---|---|
| 1 | Open Microsoft 365 admin center | Admin workstation | https://admin.microsoft.com | N/A | Admin center opens |
| 2 | Check service health | Admin workstation | Health > Service health | N/A | No active issue explains symptom |
| 3 | Check message center | Admin workstation | Health > Message center | N/A | No relevant change explains symptom |
| 4 | Open Microsoft Purview portal | Admin workstation | https://purview.microsoft.com | N/A | Purview portal opens |
| 5 | Confirm signed-in admin | Admin workstation | Profile menu | N/A | Correct admin account is used |
| 6 | Open Settings | Admin workstation | Settings | N/A | Settings page opens |
| 7 | Open Roles and scopes | Admin workstation | Settings > Roles and scopes | N/A | Role groups are visible |
| 8 | Review role group membership | Admin workstation | Role groups | N/A | Correct admin groups are assigned |
| 9 | Open Data classification | Admin workstation | Solutions > Data classification | N/A | Data classification opens |
| 10 | Open Content Explorer | Admin workstation | Data classification > Content Explorer | N/A | Content Explorer opens or permission issue is confirmed |
| 11 | Open Activity Explorer | Admin workstation | Data classification > Activity Explorer | N/A | Activity Explorer opens or permission issue is confirmed |
| 12 | Open Information protection | Admin workstation | Solutions > Information protection | N/A | Sensitivity label area opens |
| 13 | Review sensitivity labels | Admin workstation | Sensitivity labels | N/A | Labels are visible |
| 14 | Review label policies | Admin workstation | Label policies | N/A | Label policies are visible |
| 15 | Confirm label published to test user | Admin workstation | Label policy target | N/A | Test user or group is in scope |
| 16 | Review label encryption settings | Admin workstation | Label > Edit protection settings | N/A | Permission model is understood |
| 17 | Open Data lifecycle management | Admin workstation | Solutions > Data lifecycle management | N/A | Retention area opens |
| 18 | Review retention labels | Admin workstation | Retention labels | N/A | Retention labels are visible |
| 19 | Review retention label policies | Admin workstation | Label policies | N/A | Retention label publishing is visible |
| 20 | Review retention policies | Admin workstation | Retention policies | N/A | Retention policies are visible |
| 21 | Confirm retention scope | Admin workstation | Policy locations | N/A | Test site, mailbox, or OneDrive is scoped |
| 22 | Open Data loss prevention | Admin workstation | Solutions > Data loss prevention | N/A | DLP area opens |
| 23 | Review DLP policies | Admin workstation | Policies | N/A | DLP policies are visible |
| 24 | Open affected DLP policy | Admin workstation | Policy details | N/A | Policy settings open |
| 25 | Confirm policy mode | Admin workstation | Policy mode | N/A | Test, audit, warn, or enforce mode is known |
| 26 | Confirm workload locations | Admin workstation | Locations | N/A | Exchange, SharePoint, OneDrive, Teams, or Devices are scoped |
| 27 | Review DLP rules | Admin workstation | Rules | N/A | Rule conditions/actions are visible |
| 28 | Confirm SIT or label condition | Admin workstation | Rule condition | N/A | Detection signal matches test design |
| 29 | Confirm actions | Admin workstation | Rule actions | N/A | Audit, notify, block, or restrict actions are visible |
| 30 | Confirm alert settings | Admin workstation | Rule alerts | N/A | Alert behavior is configured |
| 31 | Confirm incident report settings | Admin workstation | Incident report | N/A | Incident report recipients are configured |
| 32 | Open Endpoint DLP settings | Admin workstation | DLP > Endpoint DLP settings | N/A | Endpoint settings open |
| 33 | Open Microsoft Defender portal | Admin workstation | https://security.microsoft.com | N/A | Defender portal opens |
| 34 | Search test endpoint | Admin workstation | Assets > Devices > WIN11-DLP-TEST01 | N/A | Test endpoint appears if onboarded |
| 35 | Confirm device active state | Admin workstation | Device page | N/A | Device is active and reporting |
| 36 | Open DLP alerts | Admin workstation | DLP > Alerts | N/A | Alerts page opens |
| 37 | Filter by affected policy | Admin workstation | Policy filter | N/A | Relevant alerts appear or absence is confirmed |
| 38 | Open Activity Explorer | Admin workstation | Activity Explorer | N/A | Activity view opens |
| 39 | Clear filters | Admin workstation | Reset filters | N/A | Filter bias is removed |
| 40 | Set wide time range | Admin workstation | Last 7 to 30 days | N/A | Time range expands |
| 41 | Filter by affected user | Admin workstation | User filter | N/A | User activity appears if available |
| 42 | Filter by affected workload | Admin workstation | Workload filter | N/A | Workload activity appears |
| 43 | Open Audit solution | Admin workstation | Solutions > Audit | N/A | Audit search opens |
| 44 | Run broad audit search | Admin workstation | Last 7 days | N/A | Search completes |
| 45 | Export evidence | Admin workstation | Export results | N/A | Evidence is saved |
| 46 | Reproduce issue with fake data | Test workstation | Repeat controlled test | N/A | Issue is reproduced or cleared |
| 47 | Change one setting if needed | Admin workstation | Policy or scope edit | N/A | Single controlled change is made |
| 48 | Retest | Test workstation | Repeat test | N/A | Result is compared |
| 49 | Document root cause | Admin workstation | Troubleshooting notes | N/A | Root cause is recorded |
| 50 | Apply rollback or fix | Admin workstation | Policy edit or role update | N/A | Issue is resolved or escalated |

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

    Connect-MgGraph -Scopes "Directory.Read.All","User.Read.All","Group.Read.All","AuditLog.Read.All","Device.Read.All","Sites.Read.All","Files.Read.All"

Expected result:

    Microsoft Graph connects successfully with requested scopes.

## Validate Graph Context

    Get-MgContext

Expected result:

    Connected tenant, account, and scopes are displayed.

## Validate Organization Audit Setting

    Get-OrganizationConfig | Select-Object DisplayName,AuditDisabled

Expected result:

    AuditDisabled should be False.

## Review Compliance Role Groups

    Get-RoleGroup | Where-Object {
        $_.Name -match "Compliance|DLP|Data Loss|Information|Protection|Audit|Explorer|Content|Activity|Records|eDiscovery"
    } | Select-Object Name

Expected result:

    Compliance, DLP, information protection, audit, explorer, and records role groups are listed.

## Review Compliance Management Members

    Get-RoleGroupMember "Compliance Management"

Expected result:

    Members of Compliance Management are displayed.

## Review Audit Role Groups

    Get-RoleGroup | Where-Object {
        $_.Name -match "Audit"
    } | Select-Object Name

Expected result:

    Audit-related role groups are listed.

## Review Information Protection Role Groups

    Get-RoleGroup | Where-Object {
        $_.Name -match "Information|Protection"
    } | Select-Object Name

Expected result:

    Information protection role groups are listed.

## Review DLP Role Groups

    Get-RoleGroup | Where-Object {
        $_.Name -match "DLP|Data Loss"
    } | Select-Object Name

Expected result:

    DLP-related role groups are listed.

## Review Records Role Groups

    Get-RoleGroup | Where-Object {
        $_.Name -match "Records|Retention"
    } | Select-Object Name

Expected result:

    Records and retention-related role groups are listed.

## Search for Test User

    Get-MgUser -Filter "userPrincipalName eq 'user-purview-test01@contoso.com'" | Select-Object DisplayName,UserPrincipalName,Id,AccountEnabled

Expected result:

    Test user is returned if the placeholder is replaced with the real test user UPN.

## Review Test Group Membership

    Get-MgGroup -Filter "displayName eq 'GRP-Purview-DLP-Test-Users'" | Select-Object DisplayName,Id

Expected result:

    Test group is returned if it exists.

## Search for Test Device

    Get-MgDevice -Filter "displayName eq 'WIN11-DLP-TEST01'" | Select-Object DisplayName,Id,OperatingSystem,ApproximateLastSignInDateTime

Expected result:

    Test device is returned if it exists in Entra ID device inventory.

## Search Unified Audit Log Broadly

    Search-UnifiedAuditLog `
        -StartDate (Get-Date).AddDays(-7) `
        -EndDate (Get-Date) `
        -ResultSize 100

Expected result:

    Search completes without permission error.

## Search Audit for Label Activity

    Search-UnifiedAuditLog `
        -StartDate (Get-Date).AddDays(-7) `
        -EndDate (Get-Date) `
        -ResultSize 500 |
    Where-Object {
        $_.Operations -match "Label|Sensitivity|Protection"
    }

Expected result:

    Recent label-related audit events appear if available.

## Search Audit for Retention Activity

    Search-UnifiedAuditLog `
        -StartDate (Get-Date).AddDays(-7) `
        -EndDate (Get-Date) `
        -ResultSize 500 |
    Where-Object {
        $_.Operations -match "Retention|Record|Disposition|Compliance"
    }

Expected result:

    Recent retention or records-related audit events appear if available.

## Search Audit for DLP Activity

    Search-UnifiedAuditLog `
        -StartDate (Get-Date).AddDays(-7) `
        -EndDate (Get-Date) `
        -ResultSize 500 |
    Where-Object {
        $_.Operations -match "Dlp|DLP|DataLoss|Policy|Rule"
    }

Expected result:

    Recent DLP-related audit events appear if available.

## Search Audit for Endpoint DLP Test Activity

    Search-UnifiedAuditLog `
        -StartDate (Get-Date).AddDays(-7) `
        -EndDate (Get-Date) `
        -ResultSize 500 |
    Where-Object {
        $_.AuditData -match "PURVIEW-ENDPOINT-DLP-TEST|LAB-EMP|Endpoint|DLP|DataLoss"
    }

Expected result:

    Recent Endpoint DLP test events appear if available.

## Export Troubleshooting Audit Evidence

    $Date = Get-Date -Format "yyyyMMdd"
    $OutputPath = ".\Purview_Troubleshooting_Audit_$Date.csv"

    Search-UnifiedAuditLog `
        -StartDate (Get-Date).AddDays(-14) `
        -EndDate (Get-Date) `
        -ResultSize 1000 |
    Where-Object {
        $_.Operations -match "Dlp|DLP|DataLoss|Label|Sensitivity|Protection|Retention|Record|Disposition|Compliance|Policy|Rule"
    } |
    Export-Csv $OutputPath -NoTypeInformation

    Get-Item $OutputPath

Expected result:

    CSV export is created.

## Export Troubleshooting Role Group Evidence

    $Date = Get-Date -Format "yyyyMMdd"
    $OutputPath = ".\Purview_Troubleshooting_RoleGroups_$Date.csv"

    Get-RoleGroup | Where-Object {
        $_.Name -match "Compliance|DLP|Data Loss|Information|Protection|Audit|Explorer|Content|Activity|Records|eDiscovery"
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

## Troubleshooting Area 1: Purview Portal Access

| Symptom | Likely Cause | Validation | Fix |
|---|---|---|---|
| Purview portal access denied | Missing compliance role | Check role group membership | Assign appropriate Purview role |
| Settings missing | Insufficient role | Open Roles and scopes | Use Compliance Administrator for setup |
| Solutions missing | License or role issue | Check license and role | Assign role and verify license |
| Admin account cannot view policies | Wrong account or role | Confirm signed-in profile | Sign in with correct admin |
| Feature appears missing | Feature not licensed | Check tenant licenses | Confirm licensing or use supported feature |
| Page loads slowly or fails | Service issue or browser issue | Check service health and browser | Retry, clear cache, check health |

## Troubleshooting Area 2: Role and Permission Issues

| Symptom | Likely Cause | Validation | Fix |
|---|---|---|---|
| Cannot create label | Missing Information Protection role | Review role group membership | Add user/group to correct role |
| Cannot create retention label | Missing Records role | Review records role groups | Add Records Management role |
| Cannot create DLP policy | Missing DLP role | Review DLP role groups | Add DLP Compliance Management |
| Cannot search audit | Missing Audit role | Run Search-UnifiedAuditLog | Add Audit Reader or Audit Manager |
| Cannot view Content Explorer | Missing explorer role | Open Content Explorer | Add least-privilege Content Explorer role |
| Cannot view Activity Explorer | Missing compliance role | Open Activity Explorer | Assign required role |
| Role added but still no access | Propagation delay | Wait and retry | Allow time and sign out/in |
| Too many admins have access | Role sprawl | Export role groups | Remove unnecessary members |

## Troubleshooting Area 3: Sensitivity Labels

| Symptom | Likely Cause | Validation | Fix |
|---|---|---|---|
| Label not visible in Office apps | Label policy not published to user | Check label policy target | Add user/group to label policy |
| Label not visible after policy update | Propagation delay or Office cache | Wait, restart Office apps | Sign out/in and retry |
| Only some labels visible | Policy does not include all labels | Review label policy labels | Add missing labels |
| Label cannot be applied | Client or file type issue | Test in supported Office app | Use supported client/file |
| Visual marking missing | Marking not configured | Review label settings | Add header/footer/watermark |
| Label applied but no encryption | Label has no encryption | Review label protection | Configure encryption |
| User cannot open encrypted file | User not granted permission | Review label permissions | Add user/group permission |
| Unauthorized user can open file | Label not applied or encryption not active | Check file label details | Reapply correct label |
| Downgrade justification not required | Policy setting disabled | Review label policy | Enable justification |
| Mandatory labeling not enforced | Policy setting disabled or unsupported client | Review policy and client | Enable and validate client support |

## Troubleshooting Area 4: Label Encryption

| Symptom | Likely Cause | Validation | Fix |
|---|---|---|---|
| Protected file blocks intended user | Missing permission | Review label encryption users/groups | Add intended user/group |
| External recipient cannot open | External identity not allowed | Review permissions and identity | Add recipient or use proper sharing model |
| Offline access fails | Offline access setting too restrictive | Review label encryption settings | Adjust offline access |
| User can print when not expected | Permission grants print rights | Review rights template | Remove print permission |
| User can copy content when not expected | Permission grants copy/export | Review rights template | Remove copy/export right |
| File remains protected after label change | Protection persists or cache delay | Check current label and permissions | Reapply correct label and wait |
| Outlook protected mail issue | Recipient permission issue | Test with permitted recipient | Adjust label permissions |

## Troubleshooting Area 5: Retention Labels

| Symptom | Likely Cause | Validation | Fix |
|---|---|---|---|
| Retention label not visible in SharePoint | Label policy not published to site | Check retention label policy location | Add site to policy |
| Retention label not visible in OneDrive | User OneDrive not scoped | Check policy locations | Add user OneDrive |
| Label applies but behavior unclear | Retention processing delay | Wait and check policy | Document expected delay |
| Label cannot be deleted | Label in use by policy/content | Check policies using label | Remove from policy and wait |
| Record label prevents edit/delete | Record behavior enabled | Review label record settings | Use non-record label for lab |
| Disposition not appearing | Not enough time or no eligible content | Review retention period/end action | Use correct test design |
| Event-based retention not working | Event type not configured | Review event setup | Configure event-based retention correctly |
| Wrong retention trigger | Label configured with wrong trigger | Review label settings | Create corrected label if needed |

## Troubleshooting Area 6: Retention Policies

| Symptom | Likely Cause | Validation | Fix |
|---|---|---|---|
| Policy not applying to SharePoint | Site not scoped or delay | Review policy locations | Add correct site and wait |
| Policy not applying to OneDrive | User not scoped or OneDrive not provisioned | Review locations | Scope correct OneDrive |
| Policy not applying to Exchange | Mailbox not scoped | Review Exchange locations | Add mailbox/group |
| Teams retention not behaving as expected | Wrong Teams content type | Separate chat, channel, file, and group content | Configure workload-specific retention |
| Content still visible after delete | Retention preserving content | Review policy action | Expected if retain policy applies |
| Content deleted unexpectedly | Delete policy mis-scoped | Review policy scope immediately | Disable policy and escalate |
| Policy removal did not immediately stop behavior | Propagation delay | Check policy status | Wait and verify |
| Cannot reverse record behavior | Record/regulatory record controls | Review label type | Escalate to records owner |

## Troubleshooting Area 7: DLP Policies

| Symptom | Likely Cause | Validation | Fix |
|---|---|---|---|
| DLP policy not matching | Wrong condition, scope, or mode | Review rule, scope, test data | Correct condition and retest |
| Policy tip missing | Notification disabled or unsupported client | Review rule notification | Enable policy tip and test supported client |
| User not blocked | Policy in test mode | Review policy mode | Turn on enforcement after approval |
| User blocked unexpectedly | Rule too broad or wrong scope | Review matched rule | Add exception or narrow scope |
| Alert not created | Alert disabled or delay | Review alert settings | Enable alert and wait |
| Incident report not received | Recipient or mail flow issue | Review report recipient | Correct address and test mail flow |
| Override not available | Override disabled | Review user override settings | Enable override if allowed |
| Too many false positives | SIT or rule too broad | Review match details | Add supporting evidence/exceptions |
| False negatives | Threshold too strict | Test SIT and instance count | Lower threshold or improve condition |
| External sharing not restricted | Action not configured | Review DLP action | Configure restrict/block action |

## Troubleshooting Area 8: Endpoint DLP

| Symptom | Likely Cause | Validation | Fix |
|---|---|---|---|
| Device not showing | Not onboarded or unhealthy | Check Defender device inventory | Fix onboarding |
| Endpoint events missing | Device not scoped or policy delay | Check policy scope and device | Scope user/device and wait |
| USB copy not detected | USB action not configured | Review endpoint rule action | Enable removable media control |
| Print not detected | Print action not configured | Review endpoint rule action | Enable print control |
| Clipboard not detected | Clipboard action not configured | Review endpoint rule action | Enable clipboard control |
| Network share copy not detected | Network share action not configured | Review endpoint rule action | Enable network share control |
| Browser upload not detected | Domain/browser settings missing | Review Endpoint DLP settings | Configure restricted domains/browser behavior |
| User notification missing | Notification disabled | Review rule settings | Enable notification |
| Device active but no events | Policy not applied or unsupported workflow | Check scope, client, and activity type | Retest with supported workflow |
| Endpoint alert missing | Alert disabled or processing delay | Review alert settings | Enable and wait |
| Too much endpoint noise | Policy too broad | Review event details | Tune rule and exceptions |

## Troubleshooting Area 9: Content Explorer

| Symptom | Likely Cause | Validation | Fix |
|---|---|---|---|
| Content Explorer missing | Missing role or license | Check role and licensing | Assign proper role |
| Content metadata visible but details hidden | Limited permission | Review Content Explorer role | Assign correct viewer role only if justified |
| No content appears | No indexed matching content or filter issue | Clear filters and expand time | Generate test content and wait |
| Wrong workload missing | Workload not supported or not scanned yet | Filter by workload | Validate workload support |
| Sensitive info type not appearing | SIT not detected | Test SIT with known content | Fix SIT or wait for processing |
| Too much content visible | Overbroad access | Review role membership | Remove unnecessary access |
| Content appears stale | Processing delay | Compare timestamps | Wait and retest |

## Troubleshooting Area 10: Activity Explorer

| Symptom | Likely Cause | Validation | Fix |
|---|---|---|---|
| Activity Explorer missing | Missing role or license | Check role/licensing | Assign correct role |
| No events appear | Filter issue or no activity | Clear filters and expand range | Retest activity |
| DLP events missing | DLP not matched or delay | Check DLP alerts/policy | Confirm policy match |
| Label events missing | Labels not applied or reporting delay | Apply label and wait | Retest |
| Endpoint events missing | Endpoint DLP not generating events | Check device and policy | Fix endpoint DLP |
| User filter returns no data | Wrong user or UPN | Confirm user identity | Use correct UPN |
| Workload filter hides events | Wrong filter | Clear filters | Reapply carefully |
| Events delayed | Normal processing delay | Check later | Document delay |

## Troubleshooting Area 11: Audit Search

| Symptom | Likely Cause | Validation | Fix |
|---|---|---|---|
| Audit search access denied | Missing audit role | Run Search-UnifiedAuditLog | Add Audit Reader or Audit Manager |
| Search returns no results | No matching activity or filter too narrow | Run broad search | Expand time range and filters |
| AuditDisabled is True | Organization audit disabled | Get-OrganizationConfig | Investigate and enable auditing through supported process |
| Export fails | Browser or permission issue | Try PowerShell export | Export via Search-UnifiedAuditLog |
| Specific event missing | Ingestion delay or unsupported event | Search later and broaden terms | Confirm event type support |
| Too many results | Query too broad | Narrow by date/user/activity | Use smaller time windows |
| PowerShell command fails | Not connected or wrong module | Connect-ExchangeOnline | Reconnect and retry |

## Troubleshooting Area 12: Compliance Alerts

| Symptom | Likely Cause | Validation | Fix |
|---|---|---|---|
| Alert not generated | Alert disabled or policy not matched | Review rule alert settings | Enable alert and retest |
| Alert delayed | Processing delay | Wait and refresh | Document delay |
| Alert has wrong severity | Severity configured incorrectly | Review alert settings | Adjust severity |
| Alert assigned to wrong person | Assignment workflow issue | Review alert details | Reassign if supported |
| Alert lacks context | Policy/rule metadata limited | Review Activity Explorer and audit | Correlate manually |
| Too many alerts | Rule too broad | Review false positives | Tune condition or thresholds |
| Alert not in Defender portal | Integration/workload limitation | Check Purview alerts | Use Purview alert queue |
| Closed alert still appears in reports | Historical data remains | Check status and report period | Filter status/time range |

## Common Root Causes

| Root Cause | Description |
|---|---|
| Missing license | Feature not available in tenant |
| Missing role | Admin cannot access or configure feature |
| Policy not published | User never receives label or policy |
| User not in group | Targeting issue |
| Group membership delay | Scope has not propagated |
| Policy propagation delay | Setting has not reached workloads yet |
| Office cache | Office apps need restart or sign-in refresh |
| Unsupported client | Feature not supported in app/browser/version |
| Wrong workload | Policy not scoped to workload being tested |
| Wrong policy mode | Policy in test mode instead of enforce mode |
| Rule too broad | False positives |
| Rule too narrow | False negatives |
| SIT mismatch | Test data does not match sensitive info type |
| Label mismatch | Expected label was not applied |
| Device not onboarded | Endpoint DLP cannot evaluate endpoint activity |
| Audit delay | Events not yet available |
| Service incident | Microsoft 365 service health issue |

## Controlled Reproduction Procedure

| Step | Action | Expected Result |
|---|---|---|
| 1 | Pick one user | Test scope is controlled |
| 2 | Pick one workload | Test is isolated |
| 3 | Pick one policy | Policy behavior is clear |
| 4 | Pick one rule | Rule logic is clear |
| 5 | Pick one fake data marker | Detection is predictable |
| 6 | Run one test action | Result is easy to analyze |
| 7 | Wait for processing | Delay is accounted for |
| 8 | Check user experience | Policy tip/block/label behavior is documented |
| 9 | Check Activity Explorer | Activity result is documented |
| 10 | Check audit | Audit evidence is exported |
| 11 | Check alerts | Alert behavior is documented |
| 12 | Change one setting only | Troubleshooting remains controlled |

## Troubleshooting Decision Tree

| Question | If Yes | If No |
|---|---|---|
| Is there an active service health issue? | Pause and monitor service health | Continue |
| Is required licensing present? | Continue | Fix licensing |
| Does admin have correct role? | Continue | Fix role assignment |
| Is target user in policy scope? | Continue | Fix policy scope |
| Is target workload in policy scope? | Continue | Fix workload location |
| Is policy in expected mode? | Continue | Fix mode |
| Does test data match detection condition? | Continue | Fix SIT/label/classifier/test data |
| Is client/workload supported? | Continue | Use supported client/workload |
| Has enough time passed for propagation? | Continue | Wait and retest |
| Does Activity Explorer show event? | Investigate event | Check policy and workload |
| Does audit show event? | Export evidence | Expand audit search |
| Does alert show event? | Triage alert | Check alert configuration |

## Rollback / Cleanup

| Change | Rollback |
|---|---|
| Added broad compliance role | Remove unnecessary role member |
| Added Content Explorer access | Remove user/group from role |
| Added Activity Explorer access | Remove user/group from role |
| Changed label policy | Restore prior label policy settings |
| Changed sensitivity label | Restore prior label settings or create corrected label |
| Changed encryption permissions | Restore intended permissions |
| Changed retention policy | Restore prior policy scope/action |
| Changed retention label | Restore prior label design if allowed |
| Changed DLP policy | Restore prior policy mode/scope/rule |
| Enabled blocking | Return to test/audit/warn mode |
| Changed Endpoint DLP action | Return to audit/warn mode |
| Added restricted domain | Remove or adjust domain |
| Created test files | Delete lab files if allowed |
| Created test emails | Delete test emails if allowed |
| Exported evidence | Store securely or delete lab-only export |

## Rollback Warnings

| Warning | Explanation |
|---|---|
| Retention rollback may not be immediate | Retention policy changes require processing |
| Record labels can be hard to reverse | Record/regulatory record controls are intentionally strict |
| Encryption changes may not fix already distributed copies immediately | Recipients may have cached or protected copies |
| DLP changes require propagation | Policy changes can take time |
| Endpoint DLP changes require device policy refresh | Endpoint behavior may lag |
| Audit history remains | Rollback does not erase audit records |
| Deleting test files may not delete preserved copies | Retention may preserve content |
| Removing roles may not immediately remove access | Role propagation can delay |

## Evidence Collection

| Evidence Item | Collection Method | File / Location |
|---|---|---|
| Service health screenshot | Microsoft 365 admin center | Evidence folder |
| Message center screenshot | Microsoft 365 admin center | Evidence folder |
| Role group export | PowerShell Export-Csv | Purview_Troubleshooting_RoleGroups_YYYYMMDD.csv |
| Audit export | PowerShell Export-Csv | Purview_Troubleshooting_Audit_YYYYMMDD.csv |
| Label policy screenshot | Purview portal | Evidence folder |
| Retention policy screenshot | Purview portal | Evidence folder |
| DLP policy screenshot | Purview portal | Evidence folder |
| Endpoint DLP settings screenshot | Purview portal | Evidence folder |
| Device inventory screenshot | Defender portal | Evidence folder |
| Content Explorer screenshot | Purview portal | Evidence folder |
| Activity Explorer screenshot | Purview portal | Evidence folder |
| Alert details screenshot | Purview or Defender portal | Evidence folder |
| Test user experience screenshot | Test workstation | Evidence folder |
| Root cause notes | Markdown notes | Purview_Troubleshooting_YYYYMMDD.md |

## Escalation Checklist

| Escalation Item | Required |
|---|---|
| Tenant ID documented | Yes |
| User UPN documented | Yes |
| Workload documented | Yes |
| Policy name documented | Yes |
| Rule name documented | Yes |
| Label name documented if applicable | Yes |
| Retention label/policy documented if applicable | Yes |
| Device name documented if endpoint issue | Yes |
| Browser/client version documented if client issue | Yes |
| Exact timestamp documented | Yes |
| Correlation ID documented if shown | Yes |
| Screenshots collected | Yes |
| Audit export collected | Yes |
| Reproduction steps documented | Yes |
| Service health checked | Yes |
| Recent changes checked | Yes |

## Verification Checklist

| Check | Pass / Fail |
|---|---|
| Microsoft 365 service health checked |  |
| Message center checked |  |
| Purview portal opens |  |
| Correct admin account confirmed |  |
| Licensing reviewed |  |
| Compliance role groups reviewed |  |
| Audit role reviewed |  |
| Information Protection role reviewed |  |
| DLP role reviewed |  |
| Records role reviewed |  |
| Content Explorer access reviewed |  |
| Activity Explorer access reviewed |  |
| Sensitivity label policy scope reviewed |  |
| Label encryption settings reviewed |  |
| Retention label policy scope reviewed |  |
| Retention policy scope reviewed |  |
| DLP policy scope reviewed |  |
| DLP policy mode reviewed |  |
| DLP rule conditions reviewed |  |
| DLP alert settings reviewed |  |
| Endpoint DLP device onboarding reviewed |  |
| Endpoint DLP actions reviewed |  |
| Controlled reproduction completed |  |
| Activity Explorer checked |  |
| Content Explorer checked |  |
| Audit search completed |  |
| Audit export created |  |
| Root cause documented |  |
| Rollback or fix documented |  |
| Escalation package prepared if needed |  |

## Exam / Interview Notes

- Purview troubleshooting starts with licensing, roles, scope, mode, and service health.
- Sensitivity label visibility usually depends on label policy publishing, user scope, propagation, and Office client support.
- Label encryption issues usually come from permission configuration, external identity mismatch, or client support.
- Retention issues usually require careful review of label policy scope, retention policy location, trigger, action, and processing delay.
- DLP issues usually come from workload scope, policy mode, rule condition, sensitive information type matching, or alert configuration.
- Endpoint DLP issues usually start with device onboarding, device health, user/device scope, endpoint action configuration, and processing delay.
- Content Explorer issues usually involve role permissions, filters, indexing, or licensing.
- Activity Explorer issues usually involve filters, lack of activity, ingestion delay, or missing role.
- Audit search issues usually involve permissions, narrow filters, ingestion delay, or disabled audit.
- Alerts can be delayed and depend on rule alert configuration.
- Always distinguish test mode, audit mode, warn mode, and enforcement mode.
- Always change one troubleshooting variable at a time.
- Evidence collection matters before rollback or escalation.
- Broad deletion, encryption, and blocking controls should never be tested first in production scope.

## Related Workbooks

| Workbook | Relationship |
|---|---|
| 01_Configure_Purview_Admin_Portal_Roles_Audit_And_Compliance_Baseline.md | Baseline for portal, roles, and audit |
| 02_Configure_Sensitive_Info_Types_Trainable_Classifiers_And_Data_Classification.md | Troubleshooting SIT, classifier, and data classification issues |
| 03_Configure_Sensitivity_Labels_Label_Policies_And_Encryption_Settings.md | Troubleshooting sensitivity label and encryption issues |
| 04_Configure_Retention_Labels_Retention_Policies_And_Data_Lifecycle.md | Troubleshooting retention and lifecycle issues |
| 05_Configure_DLP_For_Exchange_SharePoint_OneDrive_Teams_PowerBI_And_Copilot.md | Troubleshooting Microsoft 365 DLP issues |
| 06_Configure_Endpoint_DLP_Alerts_Events_Reports_And_Response.md | Troubleshooting Endpoint DLP issues |
| 07_Monitor_Content_Explorer_Activity_Explorer_Label_Reports_And_DLP_Alerts.md | Monitoring and alert validation source for troubleshooting |