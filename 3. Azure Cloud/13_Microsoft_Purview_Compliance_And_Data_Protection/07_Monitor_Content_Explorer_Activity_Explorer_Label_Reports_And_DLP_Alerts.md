# 07_Monitor_Content_Explorer_Activity_Explorer_Label_Reports_And_DLP_Alerts

## Objective

Monitor Microsoft Purview Content Explorer, Activity Explorer, sensitivity label reports, DLP reports, alerts, audit records, and compliance signals.

This workbook covers:

- Content Explorer access validation
- Activity Explorer access validation
- Sensitivity label report review
- DLP alert review
- DLP event investigation
- Endpoint DLP event review
- Retention and label activity monitoring
- Workload filtering
- Evidence export
- Alert triage
- Investigation workflow
- Reporting baseline
- Troubleshooting and rollback

## Lab Context

This workbook builds on the Purview configuration workbooks that created classification, labels, retention, DLP, and Endpoint DLP baselines.

Complete these first:

- 01_Configure_Purview_Admin_Portal_Roles_Audit_And_Compliance_Baseline.md
- 02_Configure_Sensitive_Info_Types_Trainable_Classifiers_And_Data_Classification.md
- 03_Configure_Sensitivity_Labels_Label_Policies_And_Encryption_Settings.md
- 04_Configure_Retention_Labels_Retention_Policies_And_Data_Lifecycle.md
- 05_Configure_DLP_For_Exchange_SharePoint_OneDrive_Teams_PowerBI_And_Copilot.md
- 06_Configure_Endpoint_DLP_Alerts_Events_Reports_And_Response.md

The goal is to validate that Purview monitoring shows sensitive content, user activity, policy matches, label usage, alerts, and endpoint DLP events after policies are configured.

This workbook focuses on:

- What sensitive content exists
- Where sensitive content is stored
- Which users interacted with sensitive data
- Which DLP policies matched
- Which labels were applied or changed
- Which alerts need triage
- Which false positives require tuning
- Which evidence should be exported

## Prerequisites

| Requirement | Notes |
|---|---|
| Microsoft Purview portal access | Required |
| Compliance Administrator | Required for broad monitoring |
| DLP Compliance Management role | Recommended |
| Information Protection Admin | Recommended |
| Content Explorer role | Required for Content Explorer access |
| Activity Explorer access | Required for activity review |
| Audit Reader or Audit Manager | Recommended |
| Microsoft 365 licensing for explorer/reporting features | Required |
| DLP policies configured | Required |
| Sensitivity labels configured | Required |
| Endpoint DLP configured | Recommended |
| Test user activity generated | Required |
| Fake sample data | Required |
| Exchange Online PowerShell module | Recommended |
| Microsoft Graph PowerShell SDK | Optional |

## Topology / Scope

| Component | Purpose |
|---|---|
| Microsoft Purview portal | Monitoring and investigation portal |
| Content Explorer | Sensitive content location review |
| Activity Explorer | User and system activity review |
| Data classification dashboard | Sensitive data summary |
| Information protection reports | Sensitivity label usage review |
| DLP alerts | DLP policy match triage |
| DLP reports | DLP policy trend and event review |
| Endpoint DLP reports | Endpoint activity review |
| Audit | Unified audit log validation |
| Exchange Online | Email DLP and label activity source |
| SharePoint Online | File DLP, label, and content source |
| OneDrive for Business | User file DLP and label source |
| Teams | Message DLP activity source |
| Power BI | Label and DLP planning source where supported |
| Microsoft 365 Defender portal | Alert and device context handoff |

## Naming Standards

| Object Type | Naming Example |
|---|---|
| Monitoring evidence folder | Purview/Monitoring-Reports-Alerts |
| DLP alert export | Purview_DLP_Alerts_YYYYMMDD.csv |
| Activity export | Purview_ActivityExplorer_YYYYMMDD.csv |
| Audit export | Purview_Audit_Monitoring_YYYYMMDD.csv |
| Label report export | Purview_Label_Report_YYYYMMDD.csv |
| Incident notes | Purview_DLP_Alert_Triage_YYYYMMDD.md |
| Test user | user-purview-test01 |
| Test endpoint | WIN11-DLP-TEST01 |
| Test policy | DLP-POL-M365-SensitiveData-Test |
| Test endpoint policy | DLP-POL-Endpoint-SensitiveData-Test |

## Monitoring Strategy

| Phase | Purpose | Action |
|---|---|---|
| Phase 1 | Confirm access | Validate explorer and report access |
| Phase 2 | Generate test data | Create DLP, label, and endpoint activity |
| Phase 3 | Review dashboards | Check summary reporting |
| Phase 4 | Investigate events | Use Activity Explorer and alerts |
| Phase 5 | Inspect content | Use Content Explorer with least privilege |
| Phase 6 | Export evidence | Save CSV/screenshots/notes |
| Phase 7 | Triage alerts | Determine true positive or false positive |
| Phase 8 | Tune controls | Adjust policies and labels |
| Phase 9 | Document response | Record decision and next action |

## Monitoring Design Principles

| Principle | Requirement |
|---|---|
| Use least privilege | Required |
| Restrict Content Explorer access | Required |
| Use fake test data | Required |
| Review alerts regularly | Required |
| Separate alert triage from policy tuning | Recommended |
| Export evidence securely | Required |
| Document false positives | Required |
| Correlate across workloads | Required |
| Validate audit search | Required |
| Validate Activity Explorer | Required |
| Validate DLP reports | Required |
| Validate label reports | Required |
| Tune policies based on evidence | Required |

## Test Data Rules

| Rule | Requirement |
|---|---|
| Use fake data only | Required |
| Do not use real employee data | Required |
| Do not use real customer data | Required |
| Do not use real health data | Required |
| Do not use real financial data | Required |
| Use test users and test locations | Required |
| Store evidence securely | Required |
| Delete lab files after validation if allowed | Recommended |

## Fake Test Data Examples

| Data Type | Fake Example |
|---|---|
| Lab employee ID | LAB-EMP-10001 |
| Lab contract ID | LAB-CONTRACT-77881 |
| Lab DLP marker | PURVIEW-DLP-TEST-DATA-ONLY |
| Endpoint DLP marker | PURVIEW-ENDPOINT-DLP-TEST-DATA-ONLY |
| Label marker | LAB-HIGHLY-CONFIDENTIAL-SAMPLE |
| Retention marker | LAB-RET-10001 |
| Test email subject | PURVIEW-MONITORING-TEST-001 |
| Test document | Purview_Monitoring_Test.docx |

## Portal / GUI Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|---|---|---|---|---|---|
| 1 | Open Microsoft Purview portal | Admin workstation | https://purview.microsoft.com | N/A | Purview portal opens |
| 2 | Confirm correct admin account | Admin workstation | Profile menu | N/A | Dedicated admin account is signed in |
| 3 | Open Data classification | Admin workstation | Solutions > Data classification | N/A | Data classification page opens |
| 4 | Review overview dashboard | Admin workstation | Data classification > Overview | N/A | Sensitive data summary appears |
| 5 | Open Content Explorer | Admin workstation | Data classification > Content Explorer | N/A | Content Explorer opens if licensed and permissioned |
| 6 | Confirm Content Explorer permissions | Admin workstation | Content Explorer page | N/A | Content metadata is visible according to permissions |
| 7 | Filter by sensitive info type | Admin workstation | SIT-Lab-Employee-ID | N/A | Matching content locations appear if detected |
| 8 | Filter by workload | Admin workstation | SharePoint / OneDrive / Exchange | N/A | Workload-specific content appears |
| 9 | Filter by label | Admin workstation | Confidential or Highly Confidential | N/A | Labeled content appears if available |
| 10 | Document Content Explorer findings | Admin workstation | Evidence notes | N/A | Findings are recorded |
| 11 | Open Activity Explorer | Admin workstation | Data classification > Activity Explorer | N/A | Activity Explorer opens |
| 12 | Set time range | Admin workstation | Last 7 days | N/A | Time range is applied |
| 13 | Filter by activity type | Admin workstation | DLP policy match or label applied | N/A | Matching activity appears if available |
| 14 | Filter by workload | Admin workstation | Exchange, SharePoint, OneDrive, Teams, Endpoint | N/A | Workload-specific activity appears |
| 15 | Filter by user | Admin workstation | user-purview-test01 | N/A | Test user activity appears if available |
| 16 | Filter by policy | Admin workstation | DLP-POL-M365-SensitiveData-Test | N/A | Policy match activity appears |
| 17 | Filter by endpoint activity | Admin workstation | Device activity | N/A | Endpoint events appear if configured |
| 18 | Open event details | Admin workstation | Select event | N/A | Event metadata is visible |
| 19 | Document Activity Explorer findings | Admin workstation | Evidence notes | N/A | Findings are recorded |
| 20 | Open Data loss prevention | Admin workstation | Solutions > Data loss prevention | N/A | DLP page opens |
| 21 | Open DLP alerts | Admin workstation | Alerts | N/A | DLP alerts page opens |
| 22 | Filter DLP alerts by policy | Admin workstation | DLP-POL-M365-SensitiveData-Test | N/A | Matching alerts appear |
| 23 | Filter DLP alerts by severity | Admin workstation | Low / Medium / High | N/A | Alert severity filter applies |
| 24 | Open alert details | Admin workstation | Select alert | N/A | Alert metadata opens |
| 25 | Review impacted user | Admin workstation | Alert details | N/A | User identity is visible |
| 26 | Review impacted content | Admin workstation | Alert details | N/A | File, email, message, or endpoint activity metadata appears |
| 27 | Review matched rule | Admin workstation | Alert details | N/A | Policy and rule are visible |
| 28 | Review matched sensitive info type | Admin workstation | Alert details | N/A | SIT or label condition is visible if available |
| 29 | Assign alert if supported | Admin workstation | Alert workflow | N/A | Owner is assigned if feature is available |
| 30 | Update alert status if supported | Admin workstation | Alert workflow | N/A | Status is updated |
| 31 | Document triage decision | Admin workstation | Incident notes | N/A | True positive or false positive is recorded |
| 32 | Open DLP reports | Admin workstation | DLP > Reports | N/A | Reports page opens |
| 33 | Review policy matches | Admin workstation | Reports filter by policy | N/A | Policy trend appears |
| 34 | Review workload trend | Admin workstation | Reports filter by workload | N/A | Workload activity appears |
| 35 | Review endpoint DLP reports | Admin workstation | Endpoint DLP report | N/A | Endpoint activity trend appears if available |
| 36 | Open Information protection reports | Admin workstation | Information protection > Reports | N/A | Label report area opens if available |
| 37 | Review sensitivity label usage | Admin workstation | Label usage report | N/A | Label application trend appears |
| 38 | Review label changes | Admin workstation | Activity Explorer label filter | N/A | Label apply/change/remove events appear |
| 39 | Review retention activity if needed | Admin workstation | Activity Explorer retention filter | N/A | Retention-related activity appears if available |
| 40 | Open Audit solution | Admin workstation | Solutions > Audit | N/A | Audit page opens |
| 41 | Run audit search | Admin workstation | Search last 7 days | N/A | Audit search completes |
| 42 | Search DLP-related audit activity | Admin workstation | DLP / Label / Retention operations | N/A | Matching audit records appear if available |
| 43 | Export audit evidence | Admin workstation | Export results | N/A | Audit export is created |
| 44 | Open Microsoft 365 Defender portal | Admin workstation | https://security.microsoft.com | N/A | Defender portal opens |
| 45 | Review alert handoff if needed | Admin workstation | Incidents and alerts | N/A | Related alerts appear if integrated |
| 46 | Review device context if endpoint event | Admin workstation | Assets > Devices | N/A | Device details are visible |
| 47 | Correlate user, device, content, and policy | Admin workstation | Cross-portal review | N/A | Investigation context is documented |
| 48 | Identify false positives | Admin workstation | Review event and content | N/A | False positives are documented |
| 49 | Identify policy tuning action | Admin workstation | Rule or policy notes | N/A | Tuning action is recorded |
| 50 | Save evidence package | Admin workstation | Evidence folder | N/A | Evidence is stored securely |

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

## Review Monitoring and Compliance Role Groups

    Get-RoleGroup | Where-Object {
        $_.Name -match "Compliance|DLP|Data Loss|Information|Protection|Audit|Explorer|Content|Activity|Records"
    } | Select-Object Name

Expected result:

    Compliance, audit, DLP, and explorer-related role groups are listed.

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

## Search Audit Log for DLP and Label Activity

    Search-UnifiedAuditLog `
        -StartDate (Get-Date).AddDays(-7) `
        -EndDate (Get-Date) `
        -ResultSize 500 |
    Where-Object {
        $_.Operations -match "Dlp|DLP|DataLoss|Label|Sensitivity|Protection|Retention|Policy|Compliance"
    }

Expected result:

    Recent DLP, label, retention, or compliance activity appears if available.

## Export Monitoring Audit Events

    $Date = Get-Date -Format "yyyyMMdd"
    $OutputPath = ".\Purview_Monitoring_Audit_$Date.csv"

    Search-UnifiedAuditLog `
        -StartDate (Get-Date).AddDays(-7) `
        -EndDate (Get-Date) `
        -ResultSize 500 |
    Where-Object {
        $_.Operations -match "Dlp|DLP|DataLoss|Label|Sensitivity|Protection|Retention|Policy|Compliance"
    } |
    Export-Csv $OutputPath -NoTypeInformation

    Get-Item $OutputPath

Expected result:

    CSV export is created.

## Search Audit Log for Lab Monitoring Test Activity

    Search-UnifiedAuditLog `
        -StartDate (Get-Date).AddDays(-2) `
        -EndDate (Get-Date) `
        -ResultSize 500 |
    Where-Object {
        $_.AuditData -match "PURVIEW-DLP-TEST|PURVIEW-ENDPOINT-DLP-TEST|LAB-EMP|LAB-HIGHLY-CONFIDENTIAL|PURVIEW-MONITORING"
    }

Expected result:

    Matching lab monitoring activity appears if audit records are available.

## Export Monitoring Role Groups for Evidence

    $Date = Get-Date -Format "yyyyMMdd"
    $OutputPath = ".\Purview_Monitoring_RoleGroups_$Date.csv"

    Get-RoleGroup | Where-Object {
        $_.Name -match "Compliance|DLP|Data Loss|Information|Protection|Audit|Explorer|Content|Activity|Records"
    } | Select-Object Name,ManagedBy,Roles |
    Export-Csv $OutputPath -NoTypeInformation

    Get-Item $OutputPath

Expected result:

    CSV export is created.

## Review Test Device from Graph

    Get-MgDevice -Filter "displayName eq 'WIN11-DLP-TEST01'" | Select-Object DisplayName,Id,OperatingSystem,ApproximateLastSignInDateTime

Expected result:

    Test device is returned if it exists in Entra ID device inventory.

## Disconnect Sessions

    Disconnect-ExchangeOnline -Confirm:$false
    Disconnect-MgGraph

Expected result:

    PowerShell sessions disconnect cleanly.

## Monitoring Concepts

| Concept | Meaning |
|---|---|
| Content Explorer | Shows where sensitive content is located |
| Activity Explorer | Shows user and system activity involving sensitive data |
| DLP alert | Alert generated by DLP policy match |
| DLP report | Summary and trend view of DLP matches |
| Label report | Summary and trend view of sensitivity label usage |
| Audit log | Searchable record of user and admin activity |
| Event detail | Metadata for a single activity or policy match |
| Alert triage | Review process to decide true positive or false positive |
| False positive | Event matched policy but is not actually risky |
| True positive | Event matched policy and represents real risk |
| Severity | Alert priority or risk impact |
| Incident report | Admin notification generated by DLP policy |
| Workload filter | Filter events by Exchange, SharePoint, OneDrive, Teams, Endpoint |
| Sensitive info type filter | Filter events by detected sensitive data type |
| Label filter | Filter events by sensitivity label |
| User filter | Filter events by user account |
| Device filter | Filter events by endpoint device |

## Content Explorer Review Areas

| Area | What To Check |
|---|---|
| Sensitive info type | Which SITs are found |
| Workload | Where sensitive content exists |
| SharePoint sites | Which sites contain sensitive files |
| OneDrive accounts | Which users have sensitive files |
| Exchange mailboxes | Which mailboxes contain sensitive data |
| Teams content | Where Teams-related content appears |
| Labels | Which labels are applied |
| Item count | Volume of sensitive content |
| Access restrictions | Who can view content details |
| Investigation need | Whether content requires remediation |

## Activity Explorer Review Areas

| Area | What To Check |
|---|---|
| DLP policy matches | Which policies are matching |
| DLP rules | Which rules are firing |
| User activity | Which users trigger events |
| Workload | Exchange, SharePoint, OneDrive, Teams, Endpoint |
| Endpoint activity | USB, print, clipboard, network share, cloud upload |
| Label activity | Applied, changed, removed, downgraded labels |
| Retention activity | Label and policy-related retention actions |
| Sharing activity | External sharing or risky sharing |
| Time range | Event timing and trend |
| False positives | Events requiring policy tuning |

## DLP Alert Triage Fields

| Field | Why It Matters |
|---|---|
| Alert title | Identifies policy match |
| Severity | Helps prioritize response |
| Status | Tracks investigation state |
| User | Identifies actor |
| Workload | Shows source system |
| Policy | Identifies DLP policy |
| Rule | Identifies matching rule |
| Sensitive info type | Shows detected data |
| Sensitivity label | Shows applied label condition |
| File or message | Shows impacted content |
| Device | Shows endpoint context |
| Action | Shows user action |
| Timestamp | Supports timeline |
| Recipients or sharing target | Shows exposure path |
| Override justification | Shows user reason if override allowed |

## Report Review Matrix

| Report | Purpose |
|---|---|
| DLP policy matches | Review policy match trends |
| DLP false positives | Identify tuning needs |
| DLP workload report | Compare Exchange, SharePoint, OneDrive, Teams, Endpoint |
| Endpoint DLP report | Review endpoint activity by action type |
| Sensitivity label report | Review label adoption |
| Label activity report | Review label apply/change/remove behavior |
| Data classification overview | Review sensitive data discovery |
| Audit report/export | Preserve investigation evidence |
| Alert queue | Triage active incidents |

## Alert Severity Guide

| Severity | Example | Response |
|---|---|---|
| Low | Test policy match in pilot scope | Document and monitor |
| Medium | Sensitive data shared internally against policy guidance | Notify user and review business need |
| High | Sensitive file shared externally or copied to USB | Investigate and restrict if needed |
| Critical | Repeated exfiltration attempts across multiple workloads | Escalate to security incident workflow |

## Triage Decision Matrix

| Decision | Meaning | Next Action |
|---|---|---|
| True positive | Policy correctly detected risky behavior | Respond and document |
| False positive | Policy matched benign content | Tune policy |
| Expected test | Lab or pilot activity | Record validation evidence |
| Duplicate | Same event already investigated | Link to existing record |
| Needs escalation | Possible security incident | Escalate to security/legal/compliance |
| Needs user coaching | User made risky but non-malicious action | Notify user or manager based on policy |
| Needs policy tuning | Policy too broad or too narrow | Adjust rule conditions |

## Validation Tests

## Test 1: Content Explorer Access

| Step | Action | Expected Result |
|---|---|---|
| 1 | Open Purview portal | Portal opens |
| 2 | Open Data classification | Dashboard opens |
| 3 | Open Content Explorer | Content Explorer opens |
| 4 | Filter by SIT-Lab-Employee-ID | Matching content appears if detected |
| 5 | Document access level | Permissions are recorded |

## Test 2: Activity Explorer Access

| Step | Action | Expected Result |
|---|---|---|
| 1 | Open Activity Explorer | Activity Explorer opens |
| 2 | Set time range to last 7 days | Time filter applies |
| 3 | Filter by test user | Test user activity appears if available |
| 4 | Filter by DLP policy | Policy match activity appears |
| 5 | Open event details | Event metadata appears |

## Test 3: DLP Alert Review

| Step | Action | Expected Result |
|---|---|---|
| 1 | Open DLP alerts | Alert page opens |
| 2 | Filter by DLP-POL-M365-SensitiveData-Test | Test policy alerts appear |
| 3 | Open alert | Alert details open |
| 4 | Review user, workload, policy, rule | Key context is visible |
| 5 | Assign triage decision | Alert decision is documented |

## Test 4: Endpoint DLP Event Review

| Step | Action | Expected Result |
|---|---|---|
| 1 | Trigger endpoint DLP test event | Event occurs on test endpoint |
| 2 | Open Activity Explorer | Activity Explorer opens |
| 3 | Filter by endpoint activity | Endpoint event appears |
| 4 | Filter by device WIN11-DLP-TEST01 | Device activity appears if available |
| 5 | Document event details | Evidence is saved |

## Test 5: Sensitivity Label Report Review

| Step | Action | Expected Result |
|---|---|---|
| 1 | Open Information protection reports | Label report page opens if available |
| 2 | Review label usage | Label adoption appears |
| 3 | Filter by Confidential label | Confidential activity appears if available |
| 4 | Review label changes in Activity Explorer | Label activity appears |
| 5 | Document label report evidence | Evidence is saved |

## Test 6: DLP Report Review

| Step | Action | Expected Result |
|---|---|---|
| 1 | Open DLP reports | Report page opens |
| 2 | Filter by policy | Policy trend appears |
| 3 | Filter by workload | Workload trend appears |
| 4 | Review false positive indicators | Tuning candidates are identified |
| 5 | Document report evidence | Evidence is saved |

## Test 7: Audit Search Validation

| Step | Action | Expected Result |
|---|---|---|
| 1 | Open Audit | Audit page opens |
| 2 | Search last 7 days | Search completes |
| 3 | Filter for DLP or label activity | Matching records appear if available |
| 4 | Export results | CSV is created |
| 5 | Store export securely | Evidence is retained |

## Test 8: Alert Response Workflow

| Step | Action | Expected Result |
|---|---|---|
| 1 | Open DLP alert | Alert details open |
| 2 | Identify user and content | Actor and content are clear |
| 3 | Identify policy and rule | Triggering control is clear |
| 4 | Determine true or false positive | Triage decision is made |
| 5 | Record response action | Notes are saved |

## Test 9: Cross-Workload Correlation

| Step | Action | Expected Result |
|---|---|---|
| 1 | Identify user from DLP alert | User is documented |
| 2 | Search Activity Explorer for same user | Related activity appears |
| 3 | Search audit for same user | Related audit events appear |
| 4 | Review endpoint device if applicable | Device context appears |
| 5 | Build timeline | Investigation timeline is documented |

## Test 10: Policy Tuning Review

| Step | Action | Expected Result |
|---|---|---|
| 1 | Identify noisy policy | Policy is documented |
| 2 | Review matched conditions | SIT, label, or workload trigger is understood |
| 3 | Review false positive examples | Examples are recorded |
| 4 | Propose tuning | Threshold, exception, or scope change is defined |
| 5 | Document change request | Tuning action is ready |

## Troubleshooting

| Issue | Likely Cause | Fix |
|---|---|---|
| Content Explorer missing | Missing role or license | Assign Content Explorer role and confirm licensing |
| Content details hidden | User lacks content viewer permission | Assign correct least-privilege role |
| Activity Explorer missing | Missing role or license | Confirm role and licensing |
| Activity Explorer empty | No events, processing delay, or wrong filter | Expand time range and clear filters |
| DLP alerts missing | Alert not enabled or processing delay | Confirm DLP rule alert settings |
| DLP report empty | No policy matches or processing delay | Generate test activity and wait |
| Label report empty | Labels not applied or reporting delay | Apply labels and wait |
| Endpoint events missing | Device not onboarded or policy not scoped | Confirm device health and policy scope |
| Audit search empty | No matching events or ingestion delay | Expand time range |
| Audit search permission error | Missing audit role | Assign Audit Reader or Audit Manager |
| Content Explorer shows too much data | Permissions too broad | Review and reduce role membership |
| Too many false positives | Policy too broad | Tune SITs, labels, thresholds, or exceptions |
| Alert severity too noisy | Alert configuration too broad | Adjust alert thresholds |
| Cannot export evidence | Browser, permission, or feature limitation | Use screenshot or PowerShell audit export |
| PowerShell role group command fails | Tenant role group name differs | List role groups and use exact tenant name |
| Defender device context missing | Device not onboarded or role missing | Confirm Defender role and device inventory |

## Common Commands

## Connect to Exchange Online

    Connect-ExchangeOnline

## Connect to Microsoft Graph

    Connect-MgGraph -Scopes "Directory.Read.All","User.Read.All","Group.Read.All","AuditLog.Read.All","Device.Read.All","Sites.Read.All","Files.Read.All"

## Review Monitoring Role Groups

    Get-RoleGroup | Where-Object {
        $_.Name -match "Compliance|DLP|Data Loss|Information|Protection|Audit|Explorer|Content|Activity|Records"
    } | Select-Object Name

## Review Compliance Management Members

    Get-RoleGroupMember "Compliance Management"

## Search Recent Monitoring Events

    Search-UnifiedAuditLog `
        -StartDate (Get-Date).AddDays(-7) `
        -EndDate (Get-Date) `
        -ResultSize 500 |
    Where-Object {
        $_.Operations -match "Dlp|DLP|DataLoss|Label|Sensitivity|Protection|Retention|Policy|Compliance"
    }

## Search Recent Lab Test Events

    Search-UnifiedAuditLog `
        -StartDate (Get-Date).AddDays(-2) `
        -EndDate (Get-Date) `
        -ResultSize 500 |
    Where-Object {
        $_.AuditData -match "PURVIEW-DLP-TEST|PURVIEW-ENDPOINT-DLP-TEST|LAB-EMP|LAB-HIGHLY-CONFIDENTIAL|PURVIEW-MONITORING"
    }

## Export Monitoring Audit Evidence

    Search-UnifiedAuditLog `
        -StartDate (Get-Date).AddDays(-7) `
        -EndDate (Get-Date) `
        -ResultSize 500 |
    Where-Object {
        $_.Operations -match "Dlp|DLP|DataLoss|Label|Sensitivity|Protection|Retention|Policy|Compliance"
    } |
    Export-Csv ".\Purview_Monitoring_Audit.csv" -NoTypeInformation

## Search for Test Device

    Get-MgDevice -Filter "displayName eq 'WIN11-DLP-TEST01'" | Select-Object DisplayName,Id,OperatingSystem,ApproximateLastSignInDateTime

## Disconnect Sessions

    Disconnect-ExchangeOnline -Confirm:$false
    Disconnect-MgGraph

## Investigation Workflow

| Phase | Action |
|---|---|
| Detect | Alert, report, Activity Explorer event, or audit search identifies activity |
| Scope | Identify user, content, workload, device, policy, and rule |
| Validate | Determine whether event is true positive, false positive, or test |
| Correlate | Search related activity across audit, Activity Explorer, alerts, and device context |
| Contain | Restrict sharing, notify user, disable risky access, or escalate |
| Tune | Adjust DLP rule, label policy, SIT threshold, exception, or scope |
| Document | Save evidence, decision, and follow-up |
| Review | Confirm improvement in reports and alert volume |

## Response Actions

| Scenario | Response |
|---|---|
| Test event | Mark as expected test and document evidence |
| False positive | Tune policy condition, threshold, exception, or scope |
| Sensitive file externally shared | Remove sharing link and notify owner |
| Sensitive email sent externally | Review recipient, notify user, escalate if needed |
| Sensitive data copied to USB | Investigate user/device and consider block mode |
| Sensitive Teams message detected | Review message context and notify user if needed |
| Highly Confidential label removed | Review justification and user activity |
| Repeated risky activity | Escalate to security incident workflow |
| Overshared Copilot-accessible content | Fix SharePoint/OneDrive/Teams permissions and labels |

## Rollback / Cleanup

| Change | Rollback |
|---|---|
| Added broad Content Explorer access | Remove unnecessary users from role |
| Added broad Activity Explorer access | Remove unnecessary users from role |
| Created evidence exports | Store securely or delete lab-only exports |
| Created test files | Delete test files if allowed |
| Created test emails | Delete test messages if allowed |
| Created test Teams messages | Delete messages if allowed |
| Created noisy DLP alerts | Tune policy and document test alerts |
| Changed alert status | Record original state if needed |
| Assigned alert owner | Reassign or clear if supported |
| Created investigation notes | Archive or delete based on evidence policy |

## Rollback Warnings

| Warning | Explanation |
|---|---|
| Content Explorer access is sensitive | It can expose where sensitive data exists |
| Activity Explorer access is sensitive | It can expose user behavior |
| Audit exports are sensitive | They can contain user, admin, file, and policy metadata |
| Alert history remains | Closing or tuning alerts does not erase audit history |
| Removing monitoring roles may take time | Role propagation can delay access changes |
| Deleting test files may not remove retained copies | Retention policies can preserve content |

## Cleanup Test Data

| Location | Cleanup Action |
|---|---|
| SharePoint test site | Delete fake monitoring test files |
| OneDrive test folder | Delete fake monitoring test files |
| Exchange mailbox | Delete fake monitoring test emails |
| Teams test channel | Delete test messages if allowed |
| Endpoint test device | Delete fake endpoint DLP files |
| USB test media | Delete fake files |
| Evidence folder | Retain only approved evidence |
| Local export folder | Secure or delete CSV exports |

## Security Notes

- Content Explorer access should be tightly restricted.
- Activity Explorer access should be limited to compliance and investigation staff.
- DLP alerts can contain sensitive metadata.
- Audit exports can contain sensitive user and content activity.
- Store evidence in a restricted location.
- Do not use real sensitive data for monitoring tests.
- False positives should be tuned before enforcement.
- Repeated high-risk alerts should be escalated.
- Monitoring should include Exchange, SharePoint, OneDrive, Teams, Endpoint, and label activity.
- Copilot risk review depends on underlying content permissions and labels.
- Overshared content must be remediated at the source.
- Reporting delays are normal in compliance systems.
- Do not assume no events means no risk. Validate filters, scope, and ingestion delay.

## Production Readiness Review

| Question | Required Answer |
|---|---|
| Is Content Explorer access limited? | Yes |
| Is Activity Explorer access limited? | Yes |
| Are DLP alerts enabled? | Yes |
| Are DLP reports reviewed regularly? | Yes |
| Are label reports reviewed regularly? | Yes |
| Is endpoint DLP reporting validated? | Yes |
| Are audit searches validated? | Yes |
| Are alert triage owners assigned? | Yes |
| Is an escalation path documented? | Yes |
| Are false positives tracked? | Yes |
| Are policy tuning actions documented? | Yes |
| Is evidence storage restricted? | Yes |
| Are test events distinguished from real events? | Yes |
| Are Copilot content risks reviewed? | Yes |
| Is help desk guidance prepared? | Yes |

## Evidence Collection

| Evidence Item | Collection Method | File / Location |
|---|---|---|
| Content Explorer filtered view | Screenshot | Evidence folder |
| Activity Explorer filtered view | Screenshot | Evidence folder |
| DLP alert details | Screenshot | Evidence folder |
| DLP report summary | Screenshot | Evidence folder |
| Endpoint DLP report | Screenshot | Evidence folder |
| Sensitivity label report | Screenshot | Evidence folder |
| Audit search result | Export or screenshot | Purview_Monitoring_Audit_YYYYMMDD.csv |
| Test user activity timeline | Workbook notes | Incident notes |
| False positive example | Screenshot or notes | Tuning folder |
| Policy tuning recommendation | Workbook notes | Tuning folder |
| Alert triage decision | Incident notes | Purview_DLP_Alert_Triage_YYYYMMDD.md |
| Role group export | PowerShell Export-Csv | Purview_Monitoring_RoleGroups_YYYYMMDD.csv |

## Verification Checklist

| Check | Pass / Fail |
|---|---|
| Purview portal opens |  |
| Data classification dashboard opens |  |
| Content Explorer opens |  |
| Content Explorer permissions reviewed |  |
| Activity Explorer opens |  |
| Activity Explorer filters work |  |
| Test user activity appears |  |
| DLP policy match activity appears |  |
| Endpoint DLP activity appears if configured |  |
| DLP alerts page opens |  |
| DLP alert details reviewed |  |
| DLP report opens |  |
| Endpoint DLP report opens if configured |  |
| Sensitivity label report reviewed |  |
| Label activity reviewed |  |
| Retention activity reviewed if available |  |
| Audit search completed |  |
| Audit export created |  |
| Defender device context reviewed if endpoint alert |  |
| Alert triage decision documented |  |
| False positives documented |  |
| Policy tuning action documented |  |
| Evidence saved securely |  |
| Test data cleanup documented |  |
| No real sensitive data used |  |

## Exam / Interview Notes

- Content Explorer shows where sensitive content exists.
- Content Explorer access should be tightly restricted because it can expose sensitive data locations and details.
- Activity Explorer shows user and system activity involving sensitive content.
- Activity Explorer can be filtered by activity, user, workload, sensitive information type, label, and policy.
- DLP alerts are used to triage policy matches.
- DLP reports show trends and policy match volume.
- Sensitivity label reports show label adoption and label activity.
- Endpoint DLP events can show actions such as USB copy, print, clipboard, network share, and cloud upload.
- Audit search provides another validation path for compliance activity.
- Reporting and audit data can be delayed.
- False positives should be documented and used for policy tuning.
- A good triage workflow identifies user, content, workload, policy, rule, action, timestamp, and severity.
- True positive events may require user coaching, sharing remediation, or incident escalation.
- Monitoring is not just alert review. It includes explorer review, report review, audit validation, and policy tuning.
- Copilot risk monitoring depends on understanding underlying content permissions, sensitivity labels, DLP, retention, and oversharing.

## Related Workbooks

| Workbook                                                                           | Relationship                                                                  |
| ---------------------------------------------------------------------------------- | ----------------------------------------------------------------------------- |
| 01_Configure_Purview_Admin_Portal_Roles_Audit_And_Compliance_Baseline.md           | Provides Purview portal, role, and audit baseline                             |
| 02_Configure_Sensitive_Info_Types_Trainable_Classifiers_And_Data_Classification.md | Provides classification detection foundation                                  |
| 03_Configure_Sensitivity_Labels_Label_Policies_And_Encryption_Settings.md          | Provides label activity and report source                                     |
| 04_Configure_Retention_Labels_Retention_Policies_And_Data_Lifecycle.md             | Provides retention activity context                                           |
| 05_Configure_DLP_For_Exchange_SharePoint_OneDrive_Teams_PowerBI_And_Copilot.md     | Provides DLP policy and alert source                                          |
| 06_Configure_Endpoint_DLP_Alerts_Events_Reports_And_Response.md                    | Provides endpoint DLP event and alert source                                  |
| 08_Troubleshoot_Purview_Labeling_Retention_DLP_Audit_And_Compliance_Alerts.md      | Troubleshoots explorer, report, audit, DLP alert, and label monitoring issues |