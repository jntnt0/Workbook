# 05_Configure_DLP_For_Exchange_SharePoint_OneDrive_Teams_PowerBI_And_Copilot

## Objective

Configure Microsoft Purview Data Loss Prevention policies for Exchange, SharePoint, OneDrive, Teams, Power BI, and Copilot-related Microsoft 365 content workflows.

This workbook covers:

- DLP planning
- DLP policy creation
- Workload scoping
- Sensitive information type selection
- Sensitivity label condition use
- Policy rule configuration
- User notifications
- Policy tips
- Incident reports
- Alerting
- Test mode validation
- Exchange validation
- SharePoint validation
- OneDrive validation
- Teams validation
- Power BI planning
- Copilot data protection planning
- Audit validation
- Troubleshooting and rollback

## Lab Context

This workbook builds on the Purview admin, classification, sensitivity label, and retention baselines.

Complete these first:

- 01_Configure_Purview_Admin_Portal_Roles_Audit_And_Compliance_Baseline.md
- 02_Configure_Sensitive_Info_Types_Trainable_Classifiers_And_Data_Classification.md
- 03_Configure_Sensitivity_Labels_Label_Policies_And_Encryption_Settings.md
- 04_Configure_Retention_Labels_Retention_Policies_And_Data_Lifecycle.md

DLP policies use classification signals, sensitive information types, sensitivity labels, workload locations, user activity, and policy rules to detect or restrict sensitive data movement.

This workbook focuses on a practical baseline:

- Detect sensitive content first
- Use test mode before enforcement
- Scope to pilot users and locations
- Validate workload behavior
- Add user notifications and incident reports
- Move to enforcement only after evidence review

## Prerequisites

| Requirement | Notes |
|---|---|
| Microsoft Purview portal access | Required |
| Compliance Administrator | Required for broad configuration |
| DLP Compliance Management role | Recommended |
| Information Protection Admin | Recommended |
| Audit Reader or Audit Manager | Recommended |
| Microsoft 365 licensing for DLP | Required |
| Exchange Online mailbox | Required |
| SharePoint test site | Required |
| OneDrive test account | Required |
| Teams test team/channel | Recommended |
| Power BI tenant/workspace | Optional depending licensing |
| Microsoft 365 Copilot licensing | Optional depending tenant |
| Test security group | Recommended |
| Fake test data | Required |
| No real sensitive data | Required |
| Exchange Online PowerShell module | Recommended |
| Microsoft Graph PowerShell SDK | Optional |

## Topology / Scope

| Component | Purpose |
|---|---|
| Microsoft Purview portal | DLP configuration |
| Data loss prevention | DLP policy creation and management |
| Exchange Online | Email DLP detection and controls |
| SharePoint Online | File DLP detection and controls |
| OneDrive for Business | User file DLP detection and controls |
| Microsoft Teams | Chat and channel message DLP detection |
| Power BI | Sensitive data and sharing DLP planning |
| Microsoft 365 Copilot | Depends on underlying indexed and permissioned Microsoft 365 content |
| Sensitive information types | DLP detection conditions |
| Sensitivity labels | DLP detection and policy conditions |
| Activity Explorer | DLP and label activity review |
| Content Explorer | Sensitive content investigation |
| Alerts | DLP policy match and incident triage |
| Audit | Admin and user activity validation |

## Naming Standards

| Object Type | Naming Example |
|---|---|
| DLP policy | DLP-POL-M365-SensitiveData-Test |
| DLP policy | DLP-POL-Exchange-PII-Test |
| DLP policy | DLP-POL-SPO-OneDrive-Confidential-Test |
| DLP policy | DLP-POL-Teams-SensitiveData-Test |
| DLP rule | DLP-RULE-Detect-LabEmployeeID |
| DLP rule | DLP-RULE-Block-HighlyConfidential-External |
| Test group | GRP-Purview-DLP-Test-Users |
| Test SharePoint site | SP-Purview-DLP-Test |
| Test Team | Team-Purview-DLP-Test |
| Test Power BI workspace | PBI-Purview-DLP-Test |
| Evidence export | Purview_DLP_YYYYMMDD.csv |
| Evidence folder | Purview/DLP |

## DLP Strategy

| Phase | Purpose | Action |
|---|---|---|
| Phase 1 | Visibility | Create policy in test mode |
| Phase 2 | Pilot | Scope to test users and locations |
| Phase 3 | Notify | Enable policy tips and user notifications |
| Phase 4 | Alert | Enable incident reports and admin alerts |
| Phase 5 | Restrict | Block or restrict risky sharing |
| Phase 6 | Tune | Reduce false positives |
| Phase 7 | Expand | Roll out to more users and workloads |
| Phase 8 | Enforce | Enable blocking only after validation |

## DLP Design Principles

| Principle | Requirement |
|---|---|
| Start in test mode | Required |
| Use fake test data | Required |
| Scope pilot locations first | Required |
| Avoid tenant-wide blocking first | Required |
| Use clear policy names | Required |
| Use clear user notification text | Required |
| Include business owner review | Required |
| Include help desk review | Required |
| Validate each workload separately | Required |
| Document false positives | Required |
| Tune before enforcement | Required |
| Monitor after rollout | Required |

## Baseline DLP Policies

| Policy | Workloads | Purpose |
|---|---|---|
| DLP-POL-M365-SensitiveData-Test | Exchange, SharePoint, OneDrive, Teams | Detect lab sensitive data |
| DLP-POL-Exchange-PII-Test | Exchange | Detect sensitive email sharing |
| DLP-POL-SPO-OneDrive-Confidential-Test | SharePoint, OneDrive | Detect protected or sensitive files |
| DLP-POL-Teams-SensitiveData-Test | Teams | Detect sensitive data in messages |
| DLP-POL-PowerBI-SensitiveData-Planning | Power BI | Plan dataset/report DLP controls |
| DLP-POL-Copilot-SensitiveData-Planning | Microsoft 365 content | Plan Copilot risk reduction through permissions, labels, and DLP |

## Detection Signals

| Signal | Example |
|---|---|
| Built-in sensitive information type | Credit Card Number |
| Built-in sensitive information type | U.S. Social Security Number |
| Custom sensitive information type | SIT-Lab-Employee-ID |
| Custom sensitive information type | SIT-Lab-Contract-ID |
| Trainable classifier | TC-Lab-Finance-Contracts |
| Sensitivity label | Confidential |
| Sensitivity label | Highly Confidential |
| Sharing condition | Shared externally |
| Location condition | SharePoint, OneDrive, Exchange, Teams |
| User group scope | GRP-Purview-DLP-Test-Users |

## Test Data Rules

| Rule | Requirement |
|---|---|
| Use fake data only | Required |
| Do not use real financial data | Required |
| Do not use real health data | Required |
| Do not use real employee data | Required |
| Do not use real customer data | Required |
| Do not test blocking broadly | Required |
| Use dedicated test users | Required |
| Use dedicated test sites | Required |
| Use test mode first | Required |
| Document cleanup | Required |

## Fake Test Data Examples

| Data Type | Fake Example |
|---|---|
| Lab employee ID | LAB-EMP-10001 |
| Lab contract ID | LAB-CONTRACT-77881 |
| Lab project code | LAB-PROJ-FIN-2026 |
| Lab DLP marker | PURVIEW-DLP-TEST-DATA-ONLY |
| Lab retention marker | LAB-RET-10001 |
| Lab sensitivity marker | LAB-HIGHLY-CONFIDENTIAL-SAMPLE |
| Test email subject | PURVIEW-DLP-TEST-001 |

## Portal / GUI Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|---|---|---|---|---|---|
| 1 | Open Microsoft Purview portal | Admin workstation | https://purview.microsoft.com | N/A | Purview portal opens |
| 2 | Confirm correct admin account | Admin workstation | Profile menu | N/A | Dedicated admin account is signed in |
| 3 | Open Data loss prevention | Admin workstation | Solutions > Data loss prevention | N/A | DLP page opens |
| 4 | Review existing policies | Admin workstation | Policies | N/A | Existing DLP policies are visible |
| 5 | Document current DLP state | Admin workstation | Workbook notes | N/A | Existing DLP policies are recorded |
| 6 | Create new DLP policy | Admin workstation | Policies > Create policy | N/A | DLP policy wizard opens |
| 7 | Select template or custom policy | Admin workstation | Custom policy | N/A | Custom policy is selected |
| 8 | Name policy | Admin workstation | DLP-POL-M365-SensitiveData-Test | N/A | Policy name is accepted |
| 9 | Add description | Admin workstation | Detects fake lab sensitive data across M365 workloads | N/A | Description is saved |
| 10 | Choose policy scope | Admin workstation | Admin units or full directory depending tenant | N/A | Scope is selected |
| 11 | Select locations | Admin workstation | Exchange, SharePoint, OneDrive, Teams | N/A | Workloads are selected |
| 12 | Scope Exchange location | Admin workstation | Include test mailbox or group | N/A | Exchange test scope is selected |
| 13 | Scope SharePoint location | Admin workstation | Include SP-Purview-DLP-Test | N/A | SharePoint test site is selected |
| 14 | Scope OneDrive location | Admin workstation | Include user-purview-test01 | N/A | OneDrive test account is selected |
| 15 | Scope Teams location | Admin workstation | Include test users or all Teams for pilot | N/A | Teams scope is selected |
| 16 | Create rule | Admin workstation | New rule | N/A | Rule wizard opens |
| 17 | Name rule | Admin workstation | DLP-RULE-Detect-LabEmployeeID | N/A | Rule name is accepted |
| 18 | Add sensitive info type condition | Admin workstation | Content contains SIT-Lab-Employee-ID | N/A | Custom SIT is selected |
| 19 | Add supporting built-in SIT condition if needed | Admin workstation | Credit Card Number or SSN for lab review only | N/A | Built-in SIT is selected if needed |
| 20 | Add sensitivity label condition if needed | Admin workstation | Content has Highly Confidential label | N/A | Label condition is selected |
| 21 | Configure instance count | Admin workstation | 1 or more | N/A | Match threshold is configured |
| 22 | Configure confidence threshold | Admin workstation | Medium or High confidence | N/A | Confidence threshold is configured |
| 23 | Configure user notification | Admin workstation | Notify users with policy tips | N/A | User notification is enabled |
| 24 | Configure policy tip | Admin workstation | Explain sensitive data handling | N/A | Policy tip text is configured |
| 25 | Configure user override if needed | Admin workstation | Allow override with business justification | N/A | Override behavior is configured |
| 26 | Configure incident report | Admin workstation | Send report to compliance admin mailbox | N/A | Incident report is enabled |
| 27 | Configure alert | Admin workstation | Alert on policy match | N/A | Alert behavior is configured |
| 28 | Configure rule action | Admin workstation | Test mode with policy tips | N/A | Rule does not block yet |
| 29 | Review rule | Admin workstation | Review settings | N/A | Rule is ready |
| 30 | Save rule | Admin workstation | Save | N/A | Rule is saved |
| 31 | Choose policy mode | Admin workstation | Test it out first | N/A | Test mode is selected |
| 32 | Review policy | Admin workstation | Review and finish | N/A | Policy settings are correct |
| 33 | Create policy | Admin workstation | Submit | N/A | DLP policy is created |
| 34 | Wait for propagation | Admin workstation | N/A | N/A | Policy begins applying after propagation |
| 35 | Create Exchange test email | Test workstation | Outlook / Outlook on the web | N/A | Test email draft is created |
| 36 | Insert fake DLP data | Test workstation | LAB-EMP-10001 PURVIEW-DLP-TEST-DATA-ONLY | N/A | Test content is added |
| 37 | Send email to internal recipient | Test workstation | Send | N/A | Internal test message sends |
| 38 | Send email to external test recipient if allowed | Test workstation | Send | N/A | Policy tip or test event appears if configured |
| 39 | Upload fake file to SharePoint | Test workstation | SP-Purview-DLP-Test | N/A | File uploads |
| 40 | Upload fake file to OneDrive | Test workstation | user-purview-test01 OneDrive | N/A | File uploads |
| 41 | Share test file externally if safe | Test workstation | Share link | N/A | DLP test event or policy tip appears if configured |
| 42 | Post fake test data in Teams test channel | Test workstation | Team-Purview-DLP-Test | N/A | Teams test message posts or is flagged |
| 43 | Review DLP alerts | Admin workstation | DLP > Alerts or Alerts portal | N/A | Alerts appear after processing if configured |
| 44 | Review Activity Explorer | Admin workstation | Data classification > Activity Explorer | N/A | DLP activity appears if available |
| 45 | Review Content Explorer | Admin workstation | Data classification > Content Explorer | N/A | Sensitive content appears if available |
| 46 | Review audit search | Admin workstation | Audit > Search | N/A | DLP-related activity appears if available |
| 47 | Review Power BI DLP planning | Admin workstation | Purview / Power BI admin integration area depending tenant | N/A | Power BI DLP requirements are documented |
| 48 | Review Copilot risk reduction planning | Admin workstation | Microsoft 365 content permissions, labels, DLP | N/A | Copilot protection dependencies are documented |
| 49 | Tune rule if false positives occur | Admin workstation | Edit DLP rule | N/A | Conditions are refined |
| 50 | Move to enforcement only after validation | Admin workstation | Turn policy on | N/A | Enforcement is enabled only after approval |

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

    Connect-MgGraph -Scopes "Directory.Read.All","User.Read.All","Group.Read.All","Sites.Read.All","Files.Read.All"

Expected result:

    Microsoft Graph connects successfully with requested scopes.

## Validate Graph Context

    Get-MgContext

Expected result:

    Connected tenant, account, and scopes are displayed.

## Review DLP and Compliance Role Groups

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

## Search Audit Log for DLP Administration

    Search-UnifiedAuditLog `
        -StartDate (Get-Date).AddDays(-7) `
        -EndDate (Get-Date) `
        -ResultSize 500 |
    Where-Object {
        $_.Operations -match "Dlp|DLP|DataLoss|Policy|Compliance|Rule"
    }

Expected result:

    Recent DLP policy or compliance administration events appear if available.

## Export DLP Administration Audit Events

    $Date = Get-Date -Format "yyyyMMdd"
    $OutputPath = ".\Purview_DLP_Admin_Audit_$Date.csv"

    Search-UnifiedAuditLog `
        -StartDate (Get-Date).AddDays(-7) `
        -EndDate (Get-Date) `
        -ResultSize 500 |
    Where-Object {
        $_.Operations -match "Dlp|DLP|DataLoss|Policy|Compliance|Rule"
    } |
    Export-Csv $OutputPath -NoTypeInformation

    Get-Item $OutputPath

Expected result:

    CSV export is created.

## Search Audit Log for DLP Test Activity

    Search-UnifiedAuditLog `
        -StartDate (Get-Date).AddDays(-2) `
        -EndDate (Get-Date) `
        -ResultSize 500 |
    Where-Object {
        $_.AuditData -match "PURVIEW-DLP-TEST|LAB-EMP|DLP|DataLoss"
    }

Expected result:

    Matching DLP test activity appears if audit records are available.

## Export DLP Role Groups for Evidence

    $Date = Get-Date -Format "yyyyMMdd"
    $OutputPath = ".\Purview_DLP_RoleGroups_$Date.csv"

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

## DLP Concepts

| Concept | Meaning |
|---|---|
| DLP policy | Container for DLP workload locations and rules |
| DLP rule | Conditions, actions, exceptions, and notifications |
| Location | Workload where DLP applies |
| Sensitive information type | Pattern-based detection signal |
| Trainable classifier | Example-trained detection signal |
| Sensitivity label | Classification/protection signal used in policy conditions |
| Policy tip | End-user notice shown during activity |
| User notification | Email or in-app notification explaining policy match |
| User override | Allows user to bypass with business justification where configured |
| Incident report | Admin report sent when rule matches |
| Alert | Compliance alert generated by DLP match |
| Test mode | Detects and reports without enforcing block |
| Enforcement mode | Applies configured restrictions |
| False positive | Content incorrectly matched |
| False negative | Sensitive content missed by policy |

## DLP Workload Planning

| Workload | What DLP Can Help Protect |
|---|---|
| Exchange Online | Sensitive email content and attachments |
| SharePoint Online | Sensitive files in sites |
| OneDrive for Business | Sensitive files in user storage |
| Microsoft Teams | Sensitive data in chats and channel messages |
| Power BI | Sensitive data in datasets and reports where supported |
| Microsoft 365 Copilot | Depends on permissions, labels, content governance, and underlying data controls |

## Exchange DLP Planning

| Setting | Baseline |
|---|---|
| Scope | Test mailbox or test group |
| Condition | SIT-Lab-Employee-ID or built-in SIT |
| Action in test mode | Policy tip and incident report |
| Enforcement action later | Block external sharing or restrict message |
| User override | Optional with business justification |
| Incident report | Send to compliance admin |
| Audit | Validate policy match and message activity |

## SharePoint DLP Planning

| Setting | Baseline |
|---|---|
| Scope | SP-Purview-DLP-Test |
| Condition | SIT-Lab-Employee-ID or Highly Confidential label |
| Action in test mode | Detect and notify |
| Enforcement action later | Restrict external sharing |
| User override | Optional |
| Incident report | Enabled |
| Content Explorer | Review sensitive file location |
| Activity Explorer | Review file activity |

## OneDrive DLP Planning

| Setting | Baseline |
|---|---|
| Scope | user-purview-test01 OneDrive |
| Condition | SIT-Lab-Employee-ID or Confidential label |
| Action in test mode | Detect and notify |
| Enforcement action later | Restrict sharing |
| User override | Optional |
| Incident report | Enabled |
| Audit | Review sharing and file activity |

## Teams DLP Planning

| Setting | Baseline |
|---|---|
| Scope | Test users or pilot users |
| Condition | SIT-Lab-Employee-ID |
| Action in test mode | Detect and notify |
| Enforcement action later | Block sensitive message sharing |
| User notification | Enabled |
| Incident report | Enabled |
| Pilot requirement | Required |

## Power BI DLP Planning

| Area | Planning Item |
|---|---|
| Tenant settings | Confirm Power BI and Purview integration support |
| Sensitivity labels | Confirm labels are available in Power BI |
| DLP policies | Confirm Power BI DLP policy support in tenant |
| Workspace scope | Start with test workspace |
| Dataset sensitivity | Validate labeled datasets and reports |
| Alerts | Review DLP matches where supported |
| Governance | Coordinate Power BI admin and Purview admin roles |

## Copilot DLP Planning

| Area | Planning Item |
|---|---|
| Permissions | Copilot can surface content users already have access to |
| Oversharing | Review SharePoint, OneDrive, and Teams permissions |
| Sensitivity labels | Use labels to classify and protect sensitive content |
| DLP | Use DLP to reduce risky sharing and exfiltration |
| Retention | Use retention to govern lifecycle of underlying content |
| Content Explorer | Identify sensitive content locations |
| Activity Explorer | Review activity involving sensitive data |
| Search/indexing | Understand that Copilot relies on Microsoft 365 content access and governance |
| Pilot | Test with limited users before broad rollout |

## Baseline DLP Rule Design

| Rule Setting | Lab Value |
|---|---|
| Rule name | DLP-RULE-Detect-LabEmployeeID |
| Condition | Content contains SIT-Lab-Employee-ID |
| Supporting marker | PURVIEW-DLP-TEST-DATA-ONLY |
| Locations | Exchange, SharePoint, OneDrive, Teams |
| Mode | Test mode |
| User notification | Enabled |
| Policy tips | Enabled |
| User override | Optional |
| Incident report | Enabled |
| Alert | Enabled |
| Enforcement | Disabled until validation |

## Example User Notification Text

| Field | Example |
|---|---|
| Policy tip | This content appears to contain sensitive lab data. Review handling requirements before sharing. |
| Email notification subject | DLP policy matched test sensitive data |
| User guidance | Remove sensitive data, apply the correct label, or provide a business justification if override is allowed. |
| Help link | Internal data handling policy or help desk page |

## Example Incident Report Recipients

| Recipient | Purpose |
|---|---|
| compliance-admin@contoso.com | Compliance investigation |
| security-operations@contoso.com | Security triage |
| dlp-owner@contoso.com | Policy owner review |
| helpdesk-lead@contoso.com | User support awareness |

## Validation Tests

## Test 1: DLP Portal Access

| Step | Action | Expected Result |
|---|---|---|
| 1 | Open Purview portal | Portal opens |
| 2 | Open Data loss prevention | DLP page opens |
| 3 | Open Policies | Policy list appears |
| 4 | Open Alerts or Activity Explorer | Monitoring area opens |

## Test 2: Create Baseline DLP Policy

| Step | Action | Expected Result |
|---|---|---|
| 1 | Create custom DLP policy | Wizard opens |
| 2 | Name policy DLP-POL-M365-SensitiveData-Test | Name is accepted |
| 3 | Select workloads | Exchange, SharePoint, OneDrive, Teams are selected |
| 4 | Scope test users and locations | Test scope is selected |
| 5 | Save policy in test mode | Policy is created |

## Test 3: Configure DLP Rule

| Step | Action | Expected Result |
|---|---|---|
| 1 | Add sensitive info type condition | SIT-Lab-Employee-ID is selected |
| 2 | Configure instance count | Threshold is configured |
| 3 | Enable user notification | Notification is enabled |
| 4 | Enable policy tip | Policy tip is enabled |
| 5 | Enable incident report | Incident report is enabled |
| 6 | Save rule | Rule is saved |

## Test 4: Exchange DLP Validation

| Step | Action | Expected Result |
|---|---|---|
| 1 | Create test email | Draft opens |
| 2 | Add LAB-EMP-10001 PURVIEW-DLP-TEST-DATA-ONLY | Test content is inserted |
| 3 | Send internally | Message sends and match may be logged |
| 4 | Send externally if safe | Policy tip or test alert may trigger |
| 5 | Review alerts and audit | DLP activity appears after processing |

## Test 5: SharePoint DLP Validation

| Step | Action | Expected Result |
|---|---|---|
| 1 | Create fake test document | Document is created |
| 2 | Add LAB-EMP-10001 PURVIEW-DLP-TEST-DATA-ONLY | Test content is inserted |
| 3 | Upload to test SharePoint site | File uploads |
| 4 | Attempt external share if safe | DLP detects or policy tip appears depending settings |
| 5 | Review Activity Explorer | File activity appears after processing |

## Test 6: OneDrive DLP Validation

| Step | Action | Expected Result |
|---|---|---|
| 1 | Create fake test document | Document is created |
| 2 | Upload to test OneDrive | File uploads |
| 3 | Apply Confidential label if needed | Label applies |
| 4 | Attempt sharing if safe | DLP test event appears |
| 5 | Review alerts | Alert appears if configured |

## Test 7: Teams DLP Validation

| Step | Action | Expected Result |
|---|---|---|
| 1 | Open test Team | Team opens |
| 2 | Post LAB-EMP-10001 PURVIEW-DLP-TEST-DATA-ONLY | Message posts or is flagged depending mode |
| 3 | Review user experience | Policy tip or notification appears if supported |
| 4 | Review DLP alerts | Alert appears after processing |
| 5 | Document behavior | Evidence saved |

## Test 8: Sensitivity Label Condition Validation

| Step | Action | Expected Result |
|---|---|---|
| 1 | Create document | Document opens |
| 2 | Apply Highly Confidential label | Label applies |
| 3 | Upload or share in scoped location | Action occurs |
| 4 | DLP rule detects label condition | Match appears if configured |
| 5 | Review activity | Label/DLP activity appears |

## Test 9: Power BI Planning Validation

| Step | Action | Expected Result |
|---|---|---|
| 1 | Confirm Power BI tenant and workspace availability | Workspace is identified |
| 2 | Confirm sensitivity label availability in Power BI | Labels are available if integrated |
| 3 | Identify test dataset/report | Test content is identified |
| 4 | Review DLP support in tenant | Support is documented |
| 5 | Record next actions | Power BI DLP plan is documented |

## Test 10: Copilot Risk Planning Validation

| Step | Action | Expected Result |
|---|---|---|
| 1 | Identify Copilot pilot users | Pilot scope is documented |
| 2 | Review SharePoint oversharing | Risk areas are identified |
| 3 | Review sensitive content locations | Content Explorer findings documented |
| 4 | Confirm labels and DLP are in place | Controls are documented |
| 5 | Record governance gaps | Gaps are captured |

## Test 11: Audit Validation

| Step | Action | Expected Result |
|---|---|---|
| 1 | Run audit search for last 7 days | Search completes |
| 2 | Filter DLP-related events | Matching events appear if available |
| 3 | Export audit results | CSV is created |
| 4 | Store evidence securely | Evidence retained |
| 5 | Confirm no permission errors | Audit access is valid |

## Troubleshooting

| Issue | Likely Cause | Fix |
|---|---|---|
| DLP page missing | Missing role or license | Confirm DLP Compliance Management or Compliance Administrator |
| Cannot create DLP policy | Insufficient permission | Assign proper Purview role |
| Workload location unavailable | License or workload support issue | Confirm tenant licensing and service availability |
| Custom SIT not visible | SIT not created or not published/available | Validate custom SIT from workbook 02 |
| Sensitivity label condition unavailable | Label not created or unsupported in condition | Validate workbook 03 |
| Policy does not match test content | Wrong SIT, confidence, or instance count | Lower threshold and retest |
| Too many matches | Condition too broad | Add exceptions or supporting evidence |
| No policy tip appears | Policy in wrong mode or unsupported client | Confirm mode, client, and user scope |
| User not in policy scope | Group membership issue | Confirm user/group targeting |
| SharePoint file not detected | Processing delay or location not scoped | Wait and verify site scope |
| OneDrive file not detected | OneDrive user not scoped | Confirm account scope |
| Teams message not detected | Teams scope or DLP processing delay | Confirm Teams location and wait |
| Alert not created | Alert not enabled or delay | Confirm alert settings |
| Incident report not received | Recipient or mail flow issue | Confirm incident report address and Exchange delivery |
| External sharing not blocked | Policy in test mode or action not configured | Confirm enforcement settings |
| Legitimate content blocked | False positive | Add exception or tune conditions |
| Power BI DLP not available | Licensing or tenant integration limitation | Confirm Power BI and Purview support |
| Copilot still sees sensitive content | Underlying permissions allow access | Fix SharePoint, OneDrive, Teams permissions and labels |
| Audit has no events | Ingestion delay or no matching events | Expand time range |
| PowerShell role group name fails | Tenant role group name differs | List role groups and use exact name |

## Common Commands

## Connect to Exchange Online

    Connect-ExchangeOnline

## Connect to Microsoft Graph

    Connect-MgGraph -Scopes "Directory.Read.All","User.Read.All","Group.Read.All","Sites.Read.All","Files.Read.All"

## Review DLP and Compliance Role Groups

    Get-RoleGroup | Where-Object {
        $_.Name -match "Compliance|DLP|Data Loss|Information|Protection|Audit"
    } | Select-Object Name

## Review Compliance Management Members

    Get-RoleGroupMember "Compliance Management"

## Search Recent DLP-Related Audit Events

    Search-UnifiedAuditLog `
        -StartDate (Get-Date).AddDays(-7) `
        -EndDate (Get-Date) `
        -ResultSize 500 |
    Where-Object {
        $_.Operations -match "Dlp|DLP|DataLoss|Policy|Compliance|Rule"
    }

## Search Recent DLP Test Activity

    Search-UnifiedAuditLog `
        -StartDate (Get-Date).AddDays(-2) `
        -EndDate (Get-Date) `
        -ResultSize 500 |
    Where-Object {
        $_.AuditData -match "PURVIEW-DLP-TEST|LAB-EMP|DLP|DataLoss"
    }

## Export DLP Audit Evidence

    Search-UnifiedAuditLog `
        -StartDate (Get-Date).AddDays(-7) `
        -EndDate (Get-Date) `
        -ResultSize 500 |
    Where-Object {
        $_.Operations -match "Dlp|DLP|DataLoss|Policy|Compliance|Rule"
    } |
    Export-Csv ".\Purview_DLP_Admin_Audit.csv" -NoTypeInformation

## Disconnect Sessions

    Disconnect-ExchangeOnline -Confirm:$false
    Disconnect-MgGraph

## Rollback / Cleanup

| Change | Rollback |
|---|---|
| Created DLP policy | Disable or delete policy |
| Created DLP rule | Disable or delete rule |
| Enabled user notification | Disable notification |
| Enabled policy tips | Disable policy tips |
| Enabled user override | Disable override |
| Enabled incident reports | Remove recipients or disable |
| Enabled alerts | Disable alert setting |
| Scoped Exchange mailbox | Remove mailbox or group from scope |
| Scoped SharePoint site | Remove site from scope |
| Scoped OneDrive account | Remove account from scope |
| Scoped Teams users | Remove users from scope |
| Enabled enforcement | Return policy to test mode |
| Uploaded test files | Delete test files |
| Sent test emails | Delete test messages |
| Posted Teams test messages | Delete test messages if allowed |
| Exported audit evidence | Store securely or delete lab-only export |

## Rollback Warnings

| Warning | Explanation |
|---|---|
| DLP rollback may take time | Policy changes can require propagation |
| Test mode is safest first step | Enforcement can block business workflows |
| Removing a policy does not remove audit history | Audit evidence remains available based on retention |
| External sharing restrictions can affect users quickly | Scope carefully before enforcement |
| Teams DLP behavior may differ from file DLP | Test separately |
| Power BI DLP behavior depends on tenant support and licensing | Validate before planning enforcement |
| Copilot risk is not solved by DLP alone | Permissions, labels, lifecycle, and oversharing cleanup matter |

## Cleanup Test Data

| Location | Cleanup Action |
|---|---|
| Exchange test mailbox | Delete DLP test messages |
| SharePoint test site | Delete DLP test files |
| OneDrive test folder | Delete DLP test files |
| Teams test channel | Delete test posts if allowed |
| Power BI test workspace | Remove test reports/datasets if created |
| Local workstation | Delete fake sample files |
| Evidence folder | Retain only approved evidence |

## Security Notes

- Do not test DLP with real sensitive data.
- Do not enable blocking tenant-wide without a pilot.
- Do not assume one DLP rule behaves the same in every workload.
- Exchange, SharePoint, OneDrive, Teams, Power BI, and Copilot-related workflows need separate validation.
- DLP can interrupt business workflows if poorly scoped.
- User notifications should clearly explain what happened.
- User override should be reviewed before enabling.
- Incident reports may contain sensitive metadata.
- Store DLP evidence securely.
- Tune false positives before enforcement.
- Coordinate DLP rollout with help desk and business owners.
- Copilot protection depends heavily on underlying permissions and content governance.
- Overshared SharePoint and OneDrive content should be remediated before broad Copilot rollout.

## Production Readiness Review

| Question | Required Answer |
|---|---|
| Has policy been tested in test mode? | Yes |
| Has each workload been validated separately? | Yes |
| Has Exchange behavior been tested? | Yes |
| Has SharePoint behavior been tested? | Yes |
| Has OneDrive behavior been tested? | Yes |
| Has Teams behavior been tested? | Yes |
| Has Power BI support been reviewed? | Yes |
| Has Copilot governance impact been reviewed? | Yes |
| Are policy tips clear? | Yes |
| Are incident report recipients correct? | Yes |
| Are alerts tuned? | Yes |
| Are false positives documented? | Yes |
| Are exceptions documented? | Yes |
| Has enforcement been approved? | Yes |
| Has rollback been documented? | Yes |
| Has help desk guidance been prepared? | Yes |

## Evidence Collection

| Evidence Item | Collection Method | File / Location |
|---|---|---|
| DLP policy settings | Screenshot | Evidence folder |
| DLP rule settings | Screenshot | Evidence folder |
| Workload scope | Screenshot or notes | Evidence folder |
| Sensitive info type condition | Screenshot | Evidence folder |
| Sensitivity label condition | Screenshot | Evidence folder |
| Policy tip configuration | Screenshot | Evidence folder |
| Incident report configuration | Screenshot | Evidence folder |
| Exchange test email result | Screenshot or notes | Evidence folder |
| SharePoint test file result | Screenshot or notes | Evidence folder |
| OneDrive test file result | Screenshot or notes | Evidence folder |
| Teams test message result | Screenshot or notes | Evidence folder |
| Activity Explorer result | Screenshot | Evidence folder |
| Content Explorer result | Screenshot | Evidence folder |
| DLP alert result | Screenshot | Evidence folder |
| Audit export | PowerShell Export-Csv | Purview_DLP_Admin_Audit_YYYYMMDD.csv |
| Copilot governance notes | Workbook notes | Evidence folder |
| Power BI planning notes | Workbook notes | Evidence folder |

## Verification Checklist

| Check | Pass / Fail |
|---|---|
| Purview portal opens |  |
| Data loss prevention page opens |  |
| Existing DLP policies reviewed |  |
| DLP-POL-M365-SensitiveData-Test created |  |
| Exchange location scoped |  |
| SharePoint location scoped |  |
| OneDrive location scoped |  |
| Teams location scoped |  |
| SIT-Lab-Employee-ID condition selected |  |
| Sensitivity label condition reviewed |  |
| Policy tips enabled |  |
| User notifications enabled |  |
| Incident reports enabled |  |
| Alerts enabled |  |
| Policy set to test mode |  |
| Exchange test completed |  |
| SharePoint test completed |  |
| OneDrive test completed |  |
| Teams test completed |  |
| Power BI planning completed |  |
| Copilot governance planning completed |  |
| Activity Explorer reviewed |  |
| Content Explorer reviewed |  |
| DLP alerts reviewed |  |
| Audit search completed |  |
| Audit export created |  |
| False positives documented |  |
| Rollback documented |  |
| No real sensitive data used |  |
| No broad enforcement enabled without approval |  |

## Exam / Interview Notes

- Microsoft Purview DLP policies detect and help prevent risky sharing of sensitive data.
- DLP policies contain locations and rules.
- DLP rules contain conditions, actions, exceptions, user notifications, incident reports, and alerts.
- Sensitive information types are common DLP detection signals.
- Sensitivity labels can also be used as DLP conditions.
- DLP should start in test mode before enforcement.
- Policy tips notify users during risky activity.
- Incident reports notify administrators or compliance owners.
- User overrides can allow users to proceed with justification when configured.
- Exchange DLP protects email and attachments.
- SharePoint DLP protects files in sites.
- OneDrive DLP protects user files.
- Teams DLP protects chat and channel message scenarios where supported.
- Power BI DLP depends on licensing and tenant integration support.
- Copilot risk reduction depends on permissions, labels, DLP, retention, and oversharing cleanup.
- DLP alerts and Activity Explorer support investigation and tuning.
- False positives must be tuned before enforcement.
- Broad blocking policies can disrupt business operations.
- Pilot groups and test locations are the correct starting point.

## Related Workbooks

| Workbook | Relationship |
|---|---|
| 01_Configure_Purview_Admin_Portal_Roles_Audit_And_Compliance_Baseline.md | Provides Purview portal, role, and audit baseline |
| 02_Configure_Sensitive_Info_Types_Trainable_Classifiers_And_Data_Classification.md | Provides SIT and classifier detection foundation |
| 03_Configure_Sensitivity_Labels_Label_Policies_And_Encryption_Settings.md | Provides sensitivity label conditions and protection context |
| 04_Configure_Retention_Labels_Retention_Policies_And_Data_Lifecycle.md | Provides lifecycle governance context |
| 06_Configure_Endpoint_DLP_Alerts_Events_Reports_And_Response.md | Extends DLP controls to endpoint activity |
| 07_Monitor_Content_Explorer_Activity_Explorer_Label_Reports_And_DLP_Alerts.md | Monitors DLP matches, label activity, and alerts |
| 08_Troubleshoot_Purview_Labeling_Retention_DLP_Audit_And_Compliance_Alerts.md | Troubleshoots DLP, labeling, retention, audit, and alert issues |