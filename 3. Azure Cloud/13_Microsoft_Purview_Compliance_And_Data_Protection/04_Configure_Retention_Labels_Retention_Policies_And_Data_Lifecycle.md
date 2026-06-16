# 04_Configure_Retention_Labels_Retention_Policies_And_Data_Lifecycle

## Objective

Configure Microsoft Purview retention labels, retention label policies, retention policies, and data lifecycle controls for Microsoft 365 workloads.

This workbook covers:

- Retention planning
- Retention label creation
- Retention label policy publishing
- Retention policy creation
- Workload scoping
- Data lifecycle review
- Disposition review planning
- SharePoint and OneDrive validation
- Exchange mailbox validation
- Teams retention planning
- Audit validation
- Troubleshooting and rollback

## Lab Context

This workbook builds on the Purview admin, classification, and sensitivity label baselines.

Complete these first:

- 01_Configure_Purview_Admin_Portal_Roles_Audit_And_Compliance_Baseline.md
- 02_Configure_Sensitive_Info_Types_Trainable_Classifiers_And_Data_Classification.md
- 03_Configure_Sensitivity_Labels_Label_Policies_And_Encryption_Settings.md

Retention controls are used to keep, delete, or retain-then-delete Microsoft 365 content according to business, legal, regulatory, and operational requirements.

This workbook focuses on a practical baseline:

- Retain business records for a defined period
- Delete low-value content after a defined period
- Preserve content where required
- Test retention in a limited scope before broad deployment

## Prerequisites

| Requirement | Notes |
|---|---|
| Microsoft Purview portal access | Required |
| Compliance Administrator | Required for broad configuration |
| Records Management role | Recommended for retention labels and records workflows |
| Information Governance or Data Lifecycle Management role | Recommended where available |
| Audit Reader or Audit Manager | Recommended |
| Microsoft 365 licensing for retention features | Required |
| Test SharePoint site | Required |
| Test OneDrive account | Recommended |
| Test Exchange mailbox | Recommended |
| Test Microsoft 365 Group or Team | Optional |
| Test security group | Recommended |
| Fake sample files | Required |
| No real regulated data | Required |
| Exchange Online PowerShell module | Recommended |
| Microsoft Graph PowerShell SDK | Optional |
| Baseline workbook 01 completed | Required |
| Classification workbook 02 completed | Recommended |
| Sensitivity labels workbook 03 completed | Recommended |

## Topology / Scope

| Component | Purpose |
|---|---|
| Microsoft Purview portal | Retention and data lifecycle configuration |
| Data lifecycle management | Retention labels and retention policies |
| Records management | Records, regulatory records, disposition review |
| Retention labels | Item-level retention controls |
| Retention label policies | Publish labels to locations |
| Retention policies | Location-level retention controls |
| SharePoint Online | Site and document retention validation |
| OneDrive for Business | User file retention validation |
| Exchange Online | Mailbox retention validation |
| Microsoft Teams | Chat and channel message retention planning |
| Microsoft 365 Groups | Group mailbox and site retention planning |
| Audit | Retention configuration and activity validation |

## Naming Standards

| Object Type | Naming Example |
|---|---|
| Retention label | RET-Business-Record-7Y |
| Retention label | RET-Finance-Record-7Y |
| Retention label | RET-Delete-After-3Y |
| Retention label | RET-Project-Close-5Y |
| Retention label policy | POL-Retention-Labels-Test-Publish |
| Retention policy | POL-Retention-Exchange-Delete-3Y |
| Retention policy | POL-Retention-SPO-OneDrive-7Y |
| Test group | GRP-Purview-Retention-Test-Users |
| Test SharePoint site | SP-Purview-Retention-Test |
| Test document | Purview_Retention_Test.docx |
| Evidence export | Purview_Retention_YYYYMMDD.csv |
| Evidence folder | Purview/Retention-Data-Lifecycle |

## Retention Strategy

| Retention Control | Purpose | Example |
|---|---|---|
| Retention label | Applies retention at item level | Label a document as a business record |
| Retention label policy | Publishes retention labels to users or locations | Make labels available in SharePoint |
| Retention policy | Applies retention broadly to locations | Retain all Exchange mail for 7 years |
| Adaptive scope | Dynamic scope based on attributes | Target users by department |
| Static scope | Fixed selected locations | Target a specific SharePoint site |
| Disposition review | Manual review before deletion | Review records before final deletion |
| Retain only | Keep content for a period | Keep for 7 years |
| Delete only | Delete content after a period | Delete after 3 years |
| Retain then delete | Keep content, then delete | Keep 7 years, then delete |
| Record declaration | Lock item as record | Mark final business record |

## Retention Design Principles

| Principle | Requirement |
|---|---|
| Start with business requirements | Required |
| Test in a limited scope | Required |
| Avoid tenant-wide deletion first | Required |
| Separate labels from policies | Required |
| Document retention triggers | Required |
| Document deletion behavior | Required |
| Include legal/compliance review | Required |
| Avoid real regulated data in lab | Required |
| Use clear label names | Required |
| Validate user experience | Required |
| Validate admin visibility | Required |
| Confirm rollback limitations | Required |

## Retention Label Baseline

| Label | Purpose | Action |
|---|---|---|
| RET-Business-Record-7Y | Retain business records for 7 years | Retain for 7 years |
| RET-Finance-Record-7Y | Retain finance records for 7 years | Retain for 7 years |
| RET-Delete-After-3Y | Delete low-value content after 3 years | Delete after 3 years |
| RET-Project-Close-5Y | Retain project records after close | Retain for 5 years based on event if configured |

## Retention Policy Baseline

| Policy | Workload | Purpose |
|---|---|---|
| POL-Retention-Labels-Test-Publish | SharePoint / OneDrive | Publish retention labels to test locations |
| POL-Retention-SPO-OneDrive-7Y | SharePoint / OneDrive | Retain files in scoped locations |
| POL-Retention-Exchange-Delete-3Y | Exchange | Delete old low-value mail after test period |
| POL-Retention-Teams-Chat-Planning | Teams | Plan Teams chat/channel retention |
| POL-Retention-M365Groups-Planning | Microsoft 365 Groups | Plan group mailbox and site retention |

## Test Data Rules

| Rule | Requirement |
|---|---|
| Use fake files only | Required |
| Use fake mail only | Required |
| Do not use real legal records | Required |
| Do not use real financial records | Required |
| Do not use real HR records | Required |
| Do not test deletion broadly | Required |
| Use dedicated test locations | Required |
| Document cleanup expectations | Required |

## Fake Test Data Examples

| Data Type | Fake Example |
|---|---|
| Business record marker | LAB-BUSINESS-RECORD |
| Finance record marker | LAB-FINANCE-RECORD |
| Project record marker | LAB-PROJECT-CLOSE-RECORD |
| Delete test marker | LAB-DELETE-AFTER-TEST |
| Retention test ID | LAB-RET-10001 |
| Test subject | PURVIEW-RETENTION-TEST-001 |

## Portal / GUI Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|---|---|---|---|---|---|
| 1 | Open Microsoft Purview portal | Admin workstation | https://purview.microsoft.com | N/A | Purview portal opens |
| 2 | Confirm correct admin account | Admin workstation | Profile menu | N/A | Dedicated admin account is signed in |
| 3 | Open Data lifecycle management | Admin workstation | Solutions > Data lifecycle management | N/A | Data lifecycle page opens |
| 4 | Open Retention labels | Admin workstation | Data lifecycle management > Retention labels | N/A | Retention labels page opens |
| 5 | Review existing retention labels | Admin workstation | Retention labels list | N/A | Existing labels are visible |
| 6 | Document current label state | Admin workstation | Workbook notes | N/A | Existing labels are recorded |
| 7 | Create business record label | Admin workstation | Create a label | N/A | Label wizard opens |
| 8 | Name business record label | Admin workstation | RET-Business-Record-7Y | N/A | Name is accepted |
| 9 | Add label description | Admin workstation | Business record retention label | N/A | Description is saved |
| 10 | Configure retention period | Admin workstation | Retain items for 7 years | N/A | Retention period is configured |
| 11 | Configure retention trigger | Admin workstation | Based on when item was created or modified | N/A | Trigger is configured |
| 12 | Configure action after period | Admin workstation | Do nothing or delete after review depending lab design | N/A | End action is configured |
| 13 | Configure record behavior if needed | Admin workstation | Mark items as record | N/A | Record behavior is enabled or skipped |
| 14 | Review and create label | Admin workstation | Review > Create | N/A | Label is created |
| 15 | Create finance record label | Admin workstation | Create a label | N/A | Label wizard opens |
| 16 | Name finance record label | Admin workstation | RET-Finance-Record-7Y | N/A | Name is accepted |
| 17 | Configure finance retention | Admin workstation | Retain 7 years | N/A | Retention is configured |
| 18 | Configure disposition review if available | Admin workstation | Start disposition review | N/A | Disposition review is configured or skipped |
| 19 | Review and create finance label | Admin workstation | Review > Create | N/A | Label is created |
| 20 | Create delete-only label | Admin workstation | Create a label | N/A | Label wizard opens |
| 21 | Name delete-only label | Admin workstation | RET-Delete-After-3Y | N/A | Name is accepted |
| 22 | Configure delete action | Admin workstation | Delete items after 3 years | N/A | Deletion behavior is configured |
| 23 | Review and create delete label | Admin workstation | Review > Create | N/A | Label is created |
| 24 | Open Label policies | Admin workstation | Data lifecycle management > Label policies | N/A | Label policies page opens |
| 25 | Create retention label policy | Admin workstation | Publish labels | N/A | Policy wizard opens |
| 26 | Select retention labels | Admin workstation | Select RET-Business-Record-7Y, RET-Finance-Record-7Y, RET-Delete-After-3Y | N/A | Labels are selected |
| 27 | Select policy type | Admin workstation | Static or adaptive scope | N/A | Scope type is selected |
| 28 | Select locations | Admin workstation | SharePoint sites and OneDrive accounts | N/A | Test locations are selected |
| 29 | Add test SharePoint site | Admin workstation | SP-Purview-Retention-Test | N/A | Test site is selected |
| 30 | Add test OneDrive user | Admin workstation | user-purview-test01 | N/A | Test OneDrive is selected |
| 31 | Name label policy | Admin workstation | POL-Retention-Labels-Test-Publish | N/A | Policy name is accepted |
| 32 | Review and create label policy | Admin workstation | Submit | N/A | Label policy is created |
| 33 | Open Retention policies | Admin workstation | Data lifecycle management > Retention policies | N/A | Retention policies page opens |
| 34 | Create SharePoint retention policy | Admin workstation | New retention policy | N/A | Policy wizard opens |
| 35 | Name SharePoint policy | Admin workstation | POL-Retention-SPO-OneDrive-7Y | N/A | Name is accepted |
| 36 | Select static or adaptive scope | Admin workstation | Static for lab | N/A | Scope type is selected |
| 37 | Select locations | Admin workstation | SharePoint sites and OneDrive accounts | N/A | Locations are selected |
| 38 | Add test SharePoint site | Admin workstation | SP-Purview-Retention-Test | N/A | Site is scoped |
| 39 | Add test OneDrive account | Admin workstation | user-purview-test01 | N/A | OneDrive is scoped |
| 40 | Configure retain action | Admin workstation | Retain items for 7 years | N/A | Retention is configured |
| 41 | Configure delete after retain if needed | Admin workstation | Optional delete after retention period | N/A | End action is configured |
| 42 | Review and create policy | Admin workstation | Submit | N/A | Retention policy is created |
| 43 | Create Exchange retention policy if needed | Admin workstation | New retention policy | N/A | Policy wizard opens |
| 44 | Name Exchange policy | Admin workstation | POL-Retention-Exchange-Delete-3Y | N/A | Name is accepted |
| 45 | Select Exchange location | Admin workstation | Exchange mailboxes | N/A | Exchange is selected |
| 46 | Scope test mailbox | Admin workstation | user-purview-test01 | N/A | Test mailbox is selected |
| 47 | Configure delete action carefully | Admin workstation | Delete content older than configured period | N/A | Delete behavior is configured |
| 48 | Review and create Exchange policy | Admin workstation | Submit | N/A | Exchange retention policy is created |
| 49 | Open Records management if available | Admin workstation | Solutions > Records management | N/A | Records management opens if licensed |
| 50 | Review file plan area | Admin workstation | Records management > File plan | N/A | File plan area is visible |
| 51 | Review disposition area | Admin workstation | Records management > Disposition | N/A | Disposition area is visible if available |
| 52 | Upload fake test file to SharePoint | Test workstation | SharePoint test library | N/A | Test file uploads |
| 53 | Apply retention label manually | Test workstation | File details > Apply label | N/A | Retention label applies if published |
| 54 | Upload fake test file to OneDrive | Test workstation | OneDrive test folder | N/A | Test file uploads |
| 55 | Apply retention label manually | Test workstation | File details > Apply label | N/A | Retention label applies if published |
| 56 | Send fake retention test email | Test workstation | Outlook | N/A | Test email sends |
| 57 | Review retention label availability | Test workstation | Outlook or web where supported | N/A | Label availability is validated |
| 58 | Wait for policy propagation | Admin workstation | N/A | N/A | Retention controls apply after propagation |
| 59 | Document test results | Admin workstation | Evidence folder | N/A | Results are recorded |
| 60 | Review audit for retention activity | Admin workstation | Audit search | N/A | Relevant retention activity appears if available |

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

    Connect-MgGraph -Scopes "Directory.Read.All","User.Read.All","Group.Read.All","Sites.Read.All"

Expected result:

    Microsoft Graph connects successfully with requested scopes.

## Validate Graph Context

    Get-MgContext

Expected result:

    Connected tenant, account, and scopes are displayed.

## Review Retention and Records Role Groups

    Get-RoleGroup | Where-Object {
        $_.Name -match "Compliance|Records|Retention|Information|Lifecycle|Audit|eDiscovery"
    } | Select-Object Name

Expected result:

    Retention, records, and compliance-related role groups are listed.

## Review Compliance Management Members

    Get-RoleGroupMember "Compliance Management"

Expected result:

    Members of Compliance Management are displayed.

## Review Records Management Role Groups

    Get-RoleGroup | Where-Object {
        $_.Name -match "Records"
    } | Select-Object Name

Expected result:

    Records-related role groups are listed.

## Search Audit Log for Retention Administration

    Search-UnifiedAuditLog `
        -StartDate (Get-Date).AddDays(-7) `
        -EndDate (Get-Date) `
        -ResultSize 500 |
    Where-Object {
        $_.Operations -match "Retention|Label|Policy|Record|Disposition|Compliance"
    }

Expected result:

    Recent retention, label, records, or compliance administration events appear if available.

## Export Retention Administration Audit Events

    $Date = Get-Date -Format "yyyyMMdd"
    $OutputPath = ".\Purview_Retention_Admin_Audit_$Date.csv"

    Search-UnifiedAuditLog `
        -StartDate (Get-Date).AddDays(-7) `
        -EndDate (Get-Date) `
        -ResultSize 500 |
    Where-Object {
        $_.Operations -match "Retention|Label|Policy|Record|Disposition|Compliance"
    } |
    Export-Csv $OutputPath -NoTypeInformation

    Get-Item $OutputPath

Expected result:

    CSV export is created.

## Export Retention Role Groups for Evidence

    $Date = Get-Date -Format "yyyyMMdd"
    $OutputPath = ".\Purview_Retention_RoleGroups_$Date.csv"

    Get-RoleGroup | Where-Object {
        $_.Name -match "Compliance|Records|Retention|Information|Lifecycle|Audit|eDiscovery"
    } | Select-Object Name,ManagedBy,Roles |
    Export-Csv $OutputPath -NoTypeInformation

    Get-Item $OutputPath

Expected result:

    CSV export is created.

## Search Recent Audit Log for Retention Test Activity

    Search-UnifiedAuditLog `
        -StartDate (Get-Date).AddDays(-2) `
        -EndDate (Get-Date) `
        -ResultSize 500 |
    Where-Object {
        $_.AuditData -match "LAB-RET|PURVIEW-RETENTION-TEST|Retention"
    }

Expected result:

    Matching test activity appears if audit records are available.

## Disconnect Sessions

    Disconnect-ExchangeOnline -Confirm:$false
    Disconnect-MgGraph

Expected result:

    PowerShell sessions disconnect cleanly.

## Retention Concepts

| Concept | Meaning |
|---|---|
| Retention label | Item-level retention setting users or policies can apply |
| Retention label policy | Publishes retention labels to locations |
| Retention policy | Location-level retention setting |
| Static scope | Fixed selected users, groups, sites, or workloads |
| Adaptive scope | Dynamic scope based on attributes |
| Retain action | Preserves content for a configured period |
| Delete action | Deletes content after a configured period |
| Retain then delete | Preserves content, then deletes it |
| Retention period | Time content is retained or deleted after |
| Retention trigger | Date used to calculate retention |
| Disposition review | Review before final deletion |
| Record label | Label that declares content as a record |
| Regulatory record | Stronger record control for regulated scenarios |
| Preservation Hold Library | SharePoint location used for preserved versions |
| Recoverable Items | Exchange mailbox location used for retention and holds |

## Retention Label Planning

| Label | Retention Period | Trigger | End Action | Record Behavior |
|---|---|---|---|---|
| RET-Business-Record-7Y | 7 years | Created or modified | Do nothing or delete after review | Optional |
| RET-Finance-Record-7Y | 7 years | Created or modified | Disposition review if available | Optional record |
| RET-Delete-After-3Y | 3 years | Created or modified | Delete | No |
| RET-Project-Close-5Y | 5 years | Event-based if configured | Disposition review | Optional |

## Retention Policy Planning

| Policy | Locations | Action | Scope |
|---|---|---|---|
| POL-Retention-Labels-Test-Publish | SharePoint, OneDrive | Publish labels | Test locations |
| POL-Retention-SPO-OneDrive-7Y | SharePoint, OneDrive | Retain 7 years | Test locations |
| POL-Retention-Exchange-Delete-3Y | Exchange | Delete after 3 years | Test mailbox |
| POL-Retention-Teams-Chat-Planning | Teams chats/channels | Planning only | Pilot users |
| POL-Retention-M365Groups-Planning | Group mailbox and site | Planning only | Pilot groups |

## Workload Behavior Planning

| Workload | Retention Consideration |
|---|---|
| Exchange Online | Mail can be retained in mailbox recoverable locations |
| SharePoint Online | Deleted or changed content may be preserved depending on policy |
| OneDrive for Business | User files can be retained and preserved |
| Teams chat | Chat retention differs from files shared in Teams |
| Teams channel messages | Channel messages have separate retention behavior |
| Microsoft 365 Groups | Group mailbox and SharePoint site may need separate consideration |
| Viva Engage | Requires separate workload support review |
| Power BI | Check current Purview and workload support before planning |
| Copilot | Retention relies on underlying Microsoft 365 content and interaction storage support |

## Data Lifecycle Review Areas

| Area | What To Check |
|---|---|
| Retention labels | Item-level lifecycle controls |
| Label policies | Where labels are published |
| Retention policies | Location-level lifecycle controls |
| Records management | Record declaration and file plan |
| Disposition | Manual review before final deletion |
| Event-based retention | Retention based on business event |
| Adaptive scopes | Dynamic policy targeting |
| Simulation or test scope | Validate before rollout |
| Audit | Track admin and user activity |
| Service health | Confirm no active service issue |

## Validation Tests

## Test 1: Retention Portal Access

| Step | Action | Expected Result |
|---|---|---|
| 1 | Open Purview portal | Portal opens |
| 2 | Open Data lifecycle management | Page opens |
| 3 | Open Retention labels | Retention label list appears |
| 4 | Open Retention policies | Retention policy list appears |
| 5 | Open Records management if available | Records area opens if licensed |

## Test 2: Create Retention Labels

| Step | Action | Expected Result |
|---|---|---|
| 1 | Create RET-Business-Record-7Y | Label is created |
| 2 | Create RET-Finance-Record-7Y | Label is created |
| 3 | Create RET-Delete-After-3Y | Label is created |
| 4 | Review label list | Labels appear |
| 5 | Document label settings | Evidence is saved |

## Test 3: Publish Retention Labels

| Step | Action | Expected Result |
|---|---|---|
| 1 | Create label policy | Wizard opens |
| 2 | Select test retention labels | Labels are selected |
| 3 | Select test SharePoint site | Site is scoped |
| 4 | Select test OneDrive account | OneDrive is scoped |
| 5 | Create policy | Policy is created |

## Test 4: Create Retention Policy

| Step | Action | Expected Result |
|---|---|---|
| 1 | Create SharePoint/OneDrive policy | Wizard opens |
| 2 | Select test locations | Locations are scoped |
| 3 | Configure retain action | Retention action is configured |
| 4 | Review settings | Settings are correct |
| 5 | Submit policy | Policy is created |

## Test 5: SharePoint Label Validation

| Step | Action | Expected Result |
|---|---|---|
| 1 | Upload fake test file | File uploads |
| 2 | Open file details | Details pane opens |
| 3 | Apply retention label | Label applies if published |
| 4 | Confirm label visibility | Label is shown on item |
| 5 | Document evidence | Screenshot or notes saved |

## Test 6: OneDrive Label Validation

| Step | Action | Expected Result |
|---|---|---|
| 1 | Upload fake test file | File uploads |
| 2 | Open file details | Details pane opens |
| 3 | Apply retention label | Label applies if published |
| 4 | Confirm label visibility | Label is shown on item |
| 5 | Document evidence | Screenshot or notes saved |

## Test 7: Exchange Retention Validation

| Step | Action | Expected Result |
|---|---|---|
| 1 | Send fake test email | Email sends |
| 2 | Scope test mailbox to policy | Mailbox is included |
| 3 | Wait for policy processing | Policy applies after propagation |
| 4 | Search audit for activity | Matching activity may appear |
| 5 | Document behavior | Evidence is saved |

## Test 8: Disposition Review Planning

| Step | Action | Expected Result |
|---|---|---|
| 1 | Open Records management | Records area opens if available |
| 2 | Open Disposition | Disposition area opens if available |
| 3 | Review reviewers | Reviewers are identified |
| 4 | Confirm no production deletion test | No real content is deleted |
| 5 | Document plan | Disposition process is documented |

## Test 9: Audit Validation

| Step | Action | Expected Result |
|---|---|---|
| 1 | Run audit search for last 7 days | Search completes |
| 2 | Filter retention-related activity | Matching activity appears if available |
| 3 | Export results | CSV is created |
| 4 | Store evidence securely | Evidence retained |
| 5 | Confirm no permission error | Audit role is valid |

## Troubleshooting

| Issue | Likely Cause | Fix |
|---|---|---|
| Data lifecycle page missing | Missing role or license | Confirm Compliance Administrator, Records Management, or lifecycle role |
| Retention labels page missing | Missing role | Assign Records Management or Compliance role |
| Cannot create retention label | Insufficient permission | Use proper Purview role group |
| Retention label policy creation blocked | Missing permissions or unsupported license | Confirm role and licensing |
| Labels do not appear in SharePoint | Policy propagation delay | Wait and retest |
| Labels do not appear in OneDrive | User/location not scoped | Confirm policy scope |
| Label applies but behavior unclear | Retention processing delay | Document and wait for processing |
| Retention policy not applying | Location not included or propagation delay | Confirm policy scope and status |
| Exchange policy not applying | Mailbox not scoped or processing delay | Confirm mailbox inclusion |
| Teams retention unclear | Wrong workload assumption | Separate Teams chat, channel, files, and group retention planning |
| Disposition review unavailable | Feature/license limitation | Confirm records management licensing |
| Cannot delete label | Label in use by policy or content | Remove from policies and confirm usage |
| Audit search has no results | No matching events or ingestion delay | Expand time range |
| PowerShell role group name fails | Tenant role name differs | List role groups and use exact name |
| Test content deleted unexpectedly | Delete policy mis-scoped | Disable policy and review scope immediately |
| Users confused by labels | Poor naming or training | Update descriptions and user guidance |

## Common Commands

## Connect to Exchange Online

    Connect-ExchangeOnline

## Connect to Microsoft Graph

    Connect-MgGraph -Scopes "Directory.Read.All","User.Read.All","Group.Read.All","Sites.Read.All"

## Review Retention and Records Role Groups

    Get-RoleGroup | Where-Object {
        $_.Name -match "Compliance|Records|Retention|Information|Lifecycle|Audit|eDiscovery"
    } | Select-Object Name

## Review Compliance Management Members

    Get-RoleGroupMember "Compliance Management"

## Review Records Role Groups

    Get-RoleGroup | Where-Object {
        $_.Name -match "Records"
    } | Select-Object Name

## Search Recent Retention-Related Audit Events

    Search-UnifiedAuditLog `
        -StartDate (Get-Date).AddDays(-7) `
        -EndDate (Get-Date) `
        -ResultSize 500 |
    Where-Object {
        $_.Operations -match "Retention|Label|Policy|Record|Disposition|Compliance"
    }

## Export Recent Retention-Related Audit Events

    Search-UnifiedAuditLog `
        -StartDate (Get-Date).AddDays(-7) `
        -EndDate (Get-Date) `
        -ResultSize 500 |
    Where-Object {
        $_.Operations -match "Retention|Label|Policy|Record|Disposition|Compliance"
    } |
    Export-Csv ".\Purview_Retention_Admin_Audit.csv" -NoTypeInformation

## Disconnect Sessions

    Disconnect-ExchangeOnline -Confirm:$false
    Disconnect-MgGraph

## Rollback / Cleanup

| Change | Rollback |
|---|---|
| Created retention label | Delete if unused, or leave disabled/unpublished |
| Created record label | Remove from policies if not needed |
| Created delete-only label | Remove from policy immediately if unsafe |
| Created retention label policy | Disable or delete policy |
| Published labels to test location | Remove location from policy |
| Created SharePoint retention policy | Disable or delete policy |
| Created OneDrive retention policy | Disable or delete policy |
| Created Exchange retention policy | Disable or delete policy |
| Scoped wrong location | Remove location from policy |
| Uploaded test files | Delete test files if allowed |
| Sent test emails | Delete test messages if allowed |
| Exported audit evidence | Store securely or delete lab-only export |

## Rollback Warnings

| Warning | Explanation |
|---|---|
| Retention rollback may not be immediate | Policy removal can take time to process |
| Retained content may remain preserved | Preservation behavior can continue until processing completes |
| Deleted content may not be recoverable after retention actions complete | Avoid testing deletion on real data |
| Labels in use may not delete cleanly | Remove from policies and wait before deletion |
| Record labels can restrict editing/deletion | Test record behavior only in lab locations |
| Disposition workflows should not be tested on real data first | Use fake files only |

## Cleanup Test Files

| Location | Cleanup Action |
|---|---|
| SharePoint test library | Delete lab files if policy allows |
| OneDrive test folder | Delete lab files if policy allows |
| Exchange test mailbox | Delete lab emails if policy allows |
| Teams files | Delete lab files if used |
| Local workstation | Delete temporary fake data |
| Evidence folder | Keep only approved evidence |

## Security Notes

- Do not test deletion policies against production locations.
- Do not create tenant-wide delete policies without legal/compliance approval.
- Retention policies can preserve content users believe they deleted.
- Retention policies can delete content if misconfigured.
- Record labels can restrict edits and deletion.
- Regulatory record behavior can be difficult or impossible to reverse in some scenarios.
- Use limited test scopes first.
- Use fake data only.
- Treat audit exports as sensitive evidence.
- Document every retention policy and label.
- Include legal, compliance, records, and business owners in production planning.
- Validate workload-specific behavior before rollout.

## Production Readiness Review

| Question | Required Answer |
|---|---|
| Are retention requirements documented? | Yes |
| Has legal/compliance approved the retention design? | Yes |
| Are labels clearly named? | Yes |
| Are retention triggers documented? | Yes |
| Are delete actions reviewed carefully? | Yes |
| Has testing been done in limited scope? | Yes |
| Are SharePoint locations validated? | Yes |
| Are OneDrive locations validated? | Yes |
| Are Exchange mailboxes validated? | Yes |
| Are Teams workloads separately planned? | Yes |
| Is disposition review planned if needed? | Yes |
| Is rollback documented? | Yes |
| Is help desk/user guidance prepared? | Yes |
| Is audit validation complete? | Yes |

## Evidence Collection

| Evidence Item | Collection Method | File / Location |
|---|---|---|
| Retention label list | Screenshot | Evidence folder |
| RET-Business-Record-7Y settings | Screenshot or notes | Evidence folder |
| RET-Finance-Record-7Y settings | Screenshot or notes | Evidence folder |
| RET-Delete-After-3Y settings | Screenshot or notes | Evidence folder |
| Retention label policy settings | Screenshot | Evidence folder |
| Retention policy settings | Screenshot | Evidence folder |
| SharePoint test file label | Screenshot | Evidence folder |
| OneDrive test file label | Screenshot | Evidence folder |
| Exchange test email | Screenshot or notes | Evidence folder |
| Records management review | Screenshot or notes | Evidence folder |
| Disposition review plan | Notes | Evidence folder |
| Audit export | PowerShell Export-Csv | Purview_Retention_Admin_Audit_YYYYMMDD.csv |

## Verification Checklist

| Check | Pass / Fail |
|---|---|
| Purview portal opens |  |
| Data lifecycle management opens |  |
| Retention labels page opens |  |
| Existing retention labels reviewed |  |
| RET-Business-Record-7Y created |  |
| RET-Finance-Record-7Y created |  |
| RET-Delete-After-3Y created |  |
| Retention label policy created |  |
| Test SharePoint site scoped |  |
| Test OneDrive account scoped |  |
| SharePoint retention policy created |  |
| OneDrive retention policy created |  |
| Exchange retention policy planned or created |  |
| Records management reviewed |  |
| Disposition review reviewed |  |
| Test SharePoint file uploaded |  |
| Test OneDrive file uploaded |  |
| Retention label applied to SharePoint file |  |
| Retention label applied to OneDrive file |  |
| Test email sent |  |
| Audit search completed |  |
| Audit export created |  |
| Rollback documented |  |
| No real sensitive data used |  |
| No broad deletion policy created without review |  |

## Exam / Interview Notes

- Retention labels apply retention at the item level.
- Retention label policies publish retention labels to users and locations.
- Retention policies apply retention at the location level.
- Retention can retain, delete, or retain and then delete content.
- Retention triggers determine when the retention period starts.
- Common triggers include creation date, modification date, label date, or event date depending on configuration.
- Records management extends retention with record declaration and disposition review.
- Disposition review supports human review before final deletion.
- SharePoint and OneDrive retention can preserve content even after users delete it.
- Exchange retention can preserve mailbox content in recoverable areas.
- Teams retention must distinguish chats, channel messages, files, and group content.
- Retention policy changes can take time to apply.
- Retention rollback is not always immediate.
- Deletion policies must be tested carefully.
- Never test deletion policies broadly with production data.
- Retention labels and sensitivity labels are different controls.
- Sensitivity labels classify and protect.
- Retention labels manage lifecycle.
- Both can be used together in a mature compliance design.

## Related Workbooks

| Workbook | Relationship |
|---|---|
| 01_Configure_Purview_Admin_Portal_Roles_Audit_And_Compliance_Baseline.md | Provides Purview portal, role, and audit baseline |
| 02_Configure_Sensitive_Info_Types_Trainable_Classifiers_And_Data_Classification.md | Provides classification foundation |
| 03_Configure_Sensitivity_Labels_Label_Policies_And_Encryption_Settings.md | Provides information protection labeling baseline |
| 05_Configure_DLP_For_Exchange_SharePoint_OneDrive_Teams_PowerBI_And_Copilot.md | Uses classification and retention context for DLP planning |
| 06_Configure_Endpoint_DLP_Alerts_Events_Reports_And_Response.md | Extends compliance controls to endpoint activity |
| 07_Monitor_Content_Explorer_Activity_Explorer_Label_Reports_And_DLP_Alerts.md | Monitors retention, label, and DLP activity |
| 08_Troubleshoot_Purview_Labeling_Retention_DLP_Audit_And_Compliance_Alerts.md | Troubleshoots retention labels, policies, audit, and compliance alerts |