# 03_Configure_Sensitivity_Labels_Label_Policies_And_Encryption_Settings

## Objective

Configure Microsoft Purview sensitivity labels, label policies, and encryption settings for Microsoft 365 information protection.

This workbook covers:

- Sensitivity label planning
- Label taxonomy design
- Creating sensitivity labels
- Configuring label scope
- Configuring visual markings
- Configuring encryption settings
- Publishing labels with label policies
- Assigning labels to test users and groups
- Validating labels in Office apps
- Validating labels in SharePoint and OneDrive
- Reviewing label activity readiness
- Troubleshooting and rollback

## Lab Context

This workbook builds on the Purview admin baseline and data classification baseline.

Complete these first:

- 01_Configure_Purview_Admin_Portal_Roles_Audit_And_Compliance_Baseline.md
- 02_Configure_Sensitive_Info_Types_Trainable_Classifiers_And_Data_Classification.md

Sensitivity labels classify and protect content. Labels can apply markings, encryption, access restrictions, site and group privacy controls, and later policy-based behavior across Exchange, SharePoint, OneDrive, Teams, Microsoft 365 Groups, Office apps, and supported services.

This workbook focuses on a practical baseline:

- Public
- General
- Confidential
- Highly Confidential

The goal is to create a clean label structure before adding advanced auto-labeling, DLP, retention, or Copilot-related protection workflows.

## Prerequisites

| Requirement | Notes |
|---|---|
| Microsoft Purview portal access | Required |
| Compliance Administrator | Required for broad configuration |
| Information Protection Admin | Recommended |
| Microsoft 365 licensing for sensitivity labels | Required |
| Test user account | Required |
| Test security group | Recommended |
| Office apps signed in with test user | Recommended |
| SharePoint test site | Recommended |
| OneDrive test account | Recommended |
| Exchange Online mailbox | Recommended |
| Audit baseline completed | Required |
| Data classification baseline completed | Required |
| Fake test documents | Required |
| Exchange Online PowerShell module | Recommended |
| Microsoft Graph PowerShell SDK | Optional |

## Topology / Scope

| Component | Purpose |
|---|---|
| Microsoft Purview portal | Label and label policy configuration |
| Information protection | Sensitivity label configuration area |
| Sensitivity labels | Classification and protection objects |
| Label policies | Publish labels to users and groups |
| Microsoft Entra groups | Targeting label policies |
| Office apps | End-user labeling validation |
| SharePoint Online | File and site labeling validation |
| OneDrive for Business | User file labeling validation |
| Exchange Online | Email labeling validation |
| Teams | Later workload validation for group/site labels |
| Audit | Label activity validation |

## Naming Standards

| Object Type | Naming Example |
|---|---|
| Sensitivity label | Public |
| Sensitivity label | General |
| Sensitivity label | Confidential |
| Sensitivity label | Highly Confidential |
| Label policy | POL-Purview-Sensitivity-Baseline |
| Test group | GRP-Purview-Label-Test-Users |
| Test document | Purview_Label_Test.docx |
| Evidence export | Purview_SensitivityLabels_YYYYMMDD.csv |
| Evidence folder | Purview/Sensitivity-Labels |

## Label Taxonomy

| Label | Purpose | Protection |
|---|---|---|
| Public | Approved for public release | No encryption |
| General | Internal business content with low risk | No encryption |
| Confidential | Sensitive internal data | Encryption recommended |
| Highly Confidential | Sensitive restricted data | Encryption required |

## Label Design Principles

| Principle | Requirement |
|---|---|
| Keep labels simple | Required |
| Avoid too many labels | Required |
| Use clear label names | Required |
| Match labels to business risk | Required |
| Test labels before broad rollout | Required |
| Use scoped label policies | Required |
| Avoid encryption until tested | Required |
| Document all labels | Required |
| Use fake test data | Required |
| Validate user experience | Required |

## Portal / GUI Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|---|---|---|---|---|---|
| 1 | Open Microsoft Purview portal | Admin workstation | https://purview.microsoft.com | N/A | Purview portal opens |
| 2 | Confirm correct admin account | Admin workstation | Profile menu | N/A | Dedicated admin account is signed in |
| 3 | Open Information protection | Admin workstation | Solutions > Information protection | N/A | Information protection page opens |
| 4 | Open Sensitivity labels | Admin workstation | Information protection > Sensitivity labels | N/A | Label list opens |
| 5 | Review existing labels | Admin workstation | Sensitivity labels page | N/A | Existing tenant labels are visible |
| 6 | Document current label state | Admin workstation | Workbook notes | N/A | Existing labels are recorded |
| 7 | Create Public label | Admin workstation | Create a label | N/A | Label wizard opens |
| 8 | Name Public label | Admin workstation | Name: Public | N/A | Name is accepted |
| 9 | Add Public description for admins | Admin workstation | Admin description | N/A | Admin description is saved |
| 10 | Add Public description for users | Admin workstation | User description | N/A | User description is saved |
| 11 | Configure Public scope | Admin workstation | Files and emails | N/A | Scope is selected |
| 12 | Configure Public protection | Admin workstation | No encryption | N/A | Label has no encryption |
| 13 | Configure Public markings | Admin workstation | Optional footer/header | N/A | Markings are configured or skipped |
| 14 | Review and create Public label | Admin workstation | Review > Create | N/A | Public label is created |
| 15 | Create General label | Admin workstation | Create a label | N/A | Label wizard opens |
| 16 | Name General label | Admin workstation | Name: General | N/A | Name is accepted |
| 17 | Configure General scope | Admin workstation | Files and emails | N/A | Scope is selected |
| 18 | Configure General protection | Admin workstation | No encryption | N/A | Label has no encryption |
| 19 | Configure General markings | Admin workstation | Optional footer/header | N/A | Markings are configured or skipped |
| 20 | Review and create General label | Admin workstation | Review > Create | N/A | General label is created |
| 21 | Create Confidential label | Admin workstation | Create a label | N/A | Label wizard opens |
| 22 | Name Confidential label | Admin workstation | Name: Confidential | N/A | Name is accepted |
| 23 | Configure Confidential scope | Admin workstation | Files and emails | N/A | Scope is selected |
| 24 | Configure Confidential visual marking | Admin workstation | Footer/header/watermark | N/A | Visual marking is configured |
| 25 | Configure Confidential encryption | Admin workstation | Assign permissions now or let users assign | N/A | Encryption configuration is selected |
| 26 | Configure Confidential user permissions | Admin workstation | Add test group | N/A | Test group permissions are configured |
| 27 | Configure Confidential offline access | Admin workstation | Set offline access days | N/A | Offline access is configured |
| 28 | Review and create Confidential label | Admin workstation | Review > Create | N/A | Confidential label is created |
| 29 | Create Highly Confidential label | Admin workstation | Create a label | N/A | Label wizard opens |
| 30 | Name Highly Confidential label | Admin workstation | Name: Highly Confidential | N/A | Name is accepted |
| 31 | Configure Highly Confidential scope | Admin workstation | Files and emails | N/A | Scope is selected |
| 32 | Configure Highly Confidential visual marking | Admin workstation | Header/footer/watermark | N/A | Strong visual marking is configured |
| 33 | Configure Highly Confidential encryption | Admin workstation | Assign permissions now | N/A | Encryption is enabled |
| 34 | Configure Highly Confidential permissions | Admin workstation | Add restricted test group | N/A | Restricted permissions are configured |
| 35 | Configure content expiration if required | Admin workstation | Optional expiration | N/A | Expiration is configured or skipped |
| 36 | Configure offline access | Admin workstation | Set days or never | N/A | Offline access behavior is configured |
| 37 | Review and create Highly Confidential label | Admin workstation | Review > Create | N/A | Highly Confidential label is created |
| 38 | Review label order | Admin workstation | Sensitivity labels list | N/A | Labels are ordered logically |
| 39 | Open Label policies | Admin workstation | Information protection > Label policies | N/A | Label policies page opens |
| 40 | Create label policy | Admin workstation | Publish labels | N/A | Policy wizard opens |
| 41 | Select labels to publish | Admin workstation | Public, General, Confidential, Highly Confidential | N/A | Labels are selected |
| 42 | Assign policy to test users or group | Admin workstation | GRP-Purview-Label-Test-Users | N/A | Test scope is selected |
| 43 | Configure default label if required | Admin workstation | Optional default label | N/A | Default label is configured or skipped |
| 44 | Configure mandatory labeling if required | Admin workstation | Require users to apply a label | N/A | Mandatory labeling is configured or skipped |
| 45 | Configure justification for downgrade | Admin workstation | Require justification | N/A | Downgrade justification is enabled |
| 46 | Name policy | Admin workstation | POL-Purview-Sensitivity-Baseline | N/A | Policy name is accepted |
| 47 | Review and create label policy | Admin workstation | Submit | N/A | Label policy is created |
| 48 | Wait for policy propagation | Admin workstation | N/A | N/A | Labels become available to scoped users after propagation |
| 49 | Sign in to Office app as test user | Test workstation | Word / Excel / PowerPoint / Outlook | N/A | Test user is signed in |
| 50 | Create test document | Test workstation | Word document | N/A | Test document is created |
| 51 | Apply Public label | Test workstation | Sensitivity button | N/A | Public label applies |
| 52 | Apply Confidential label | Test workstation | Sensitivity button | N/A | Confidential label applies |
| 53 | Confirm markings | Test workstation | Document view | N/A | Header/footer/watermark appears if configured |
| 54 | Save labeled file to OneDrive | Test workstation | Save to OneDrive | N/A | File saves successfully |
| 55 | Upload labeled file to SharePoint | Test workstation | SharePoint test library | N/A | File uploads successfully |
| 56 | Create test email | Test workstation | Outlook | N/A | Email draft is created |
| 57 | Apply Confidential label to email | Test workstation | Sensitivity button | N/A | Email label applies |
| 58 | Send test email to permitted user | Test workstation | Send | N/A | Email sends successfully |
| 59 | Attempt access as unauthorized user if safe | Test workstation | Open protected file/email | N/A | Access is blocked if encryption restricts user |
| 60 | Document validation evidence | Admin workstation | Evidence folder | N/A | Test results are recorded |

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

## Review Compliance Role Groups

    Get-RoleGroup | Where-Object {
        $_.Name -match "Compliance|Information|Protection|DLP|Audit"
    } | Select-Object Name

Expected result:

    Compliance and information protection role groups are listed.

## Review Information Protection Role Membership

    Get-RoleGroup | Where-Object {
        $_.Name -match "Information|Protection"
    } | Select-Object Name

Expected result:

    Information protection-related role groups are listed.

## Review Compliance Management Members

    Get-RoleGroupMember "Compliance Management"

Expected result:

    Members of Compliance Management are displayed.

## Search Audit Log for Label Administration

    Search-UnifiedAuditLog `
        -StartDate (Get-Date).AddDays(-7) `
        -EndDate (Get-Date) `
        -ResultSize 500 |
    Where-Object {
        $_.Operations -match "Label|Sensitivity|Protection|Policy"
    }

Expected result:

    Recent sensitivity label or policy administration events appear if available.

## Export Label Administration Audit Events

    $Date = Get-Date -Format "yyyyMMdd"
    $OutputPath = ".\Purview_SensitivityLabel_Admin_Audit_$Date.csv"

    Search-UnifiedAuditLog `
        -StartDate (Get-Date).AddDays(-7) `
        -EndDate (Get-Date) `
        -ResultSize 500 |
    Where-Object {
        $_.Operations -match "Label|Sensitivity|Protection|Policy"
    } |
    Export-Csv $OutputPath -NoTypeInformation

    Get-Item $OutputPath

Expected result:

    CSV export is created.

## Export Compliance Role Groups for Evidence

    $Date = Get-Date -Format "yyyyMMdd"
    $OutputPath = ".\Purview_SensitivityLabel_RoleGroups_$Date.csv"

    Get-RoleGroup | Where-Object {
        $_.Name -match "Compliance|Information|Protection|DLP|Audit"
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

## Sensitivity Label Concepts

| Concept | Meaning |
|---|---|
| Sensitivity label | Classification and protection label for content |
| Label policy | Publishes labels to users or groups |
| Scope | Defines where the label can apply |
| Visual marking | Header, footer, or watermark added to content |
| Encryption | Rights protection applied to files or emails |
| User-defined permissions | Users choose who can access protected content |
| Admin-defined permissions | Admin predefines who can access protected content |
| Offline access | How long protected content can be accessed without reconnecting |
| Default label | Label automatically applied by policy |
| Mandatory labeling | Users must choose a label |
| Downgrade justification | User must explain when lowering label sensitivity |
| Label priority | Order of labels from lower to higher sensitivity |

## Recommended Baseline Labels

| Label | User Description | Admin Description | Protection |
|---|---|---|---|
| Public | Data approved for public sharing | Lowest sensitivity label | No encryption |
| General | Business data for internal use | Standard internal content | No encryption |
| Confidential | Sensitive business data | Sensitive internal information | Encryption optional or scoped |
| Highly Confidential | Restricted sensitive data | Highest sensitivity baseline label | Encryption required |

## Label Scope Planning

| Scope | Use Case | Baseline Decision |
|---|---|---|
| Files | Word, Excel, PowerPoint, PDF where supported | Enable |
| Emails | Outlook email labeling | Enable |
| Meetings | Meeting invite and meeting protection where licensed | Optional |
| Groups and sites | Teams, Microsoft 365 Groups, SharePoint site controls | Optional for later |
| Schematized data assets | Data map and supported data sources | Optional for later |

## Visual Marking Planning

| Label | Header | Footer | Watermark |
|---|---|---|---|
| Public | Optional | Optional | No |
| General | Optional | Optional | No |
| Confidential | Confidential | Confidential | Optional |
| Highly Confidential | Highly Confidential | Highly Confidential | Recommended |

## Encryption Planning

| Label | Encryption | Permission Model | Notes |
|---|---|---|---|
| Public | No | None | Should not restrict public content |
| General | No | None | Internal classification only |
| Confidential | Optional | Admin-defined or user-defined | Test before broad rollout |
| Highly Confidential | Yes | Admin-defined | Restrict to approved groups |

## Example Encryption Permission Model

| Label | Users / Groups | Permission |
|---|---|---|
| Confidential | GRP-Purview-Label-Test-Users | Co-author or Edit |
| Confidential | Tenant users | View or Reviewer, if needed |
| Highly Confidential | GRP-Purview-HighlyConfidential-Test | Co-author or Edit |
| Highly Confidential | All other users | No access |

## Label Policy Planning

| Policy Setting | Baseline Recommendation |
|---|---|
| Policy name | POL-Purview-Sensitivity-Baseline |
| Published labels | Public, General, Confidential, Highly Confidential |
| Target users | GRP-Purview-Label-Test-Users |
| Default label for documents | General or none for initial lab |
| Default label for emails | General or none for initial lab |
| Mandatory labeling | Optional in lab, test before production |
| Require justification to remove or lower label | Enable |
| Link to help page | Optional |
| User notification text | Optional |
| Advanced settings | Avoid unless required |

## Label Policy Targeting

| Targeting Method | Recommendation |
|---|---|
| Individual users | Avoid except lab testing |
| Security groups | Preferred |
| Microsoft 365 groups | Use when appropriate |
| All users | Use only after pilot validation |
| Admin accounts | Include only if needed for validation |
| Break-glass accounts | Avoid routine policy targeting unless required |

## Validation Tests

## Test 1: Purview Label Configuration Access

| Step | Action | Expected Result |
|---|---|---|
| 1 | Open Purview portal | Portal opens |
| 2 | Open Information protection | Information protection opens |
| 3 | Open Sensitivity labels | Label list appears |
| 4 | Open Label policies | Label policies page appears |

## Test 2: Create Baseline Labels

| Step | Action | Expected Result |
|---|---|---|
| 1 | Create Public label | Public label is created |
| 2 | Create General label | General label is created |
| 3 | Create Confidential label | Confidential label is created |
| 4 | Create Highly Confidential label | Highly Confidential label is created |
| 5 | Review label order | Labels are ordered by sensitivity |

## Test 3: Configure Protection

| Step | Action | Expected Result |
|---|---|---|
| 1 | Confirm Public has no encryption | Public is classification only |
| 2 | Confirm General has no encryption | General is classification only |
| 3 | Confirm Confidential protection | Encryption or user permissions are configured |
| 4 | Confirm Highly Confidential protection | Encryption is configured |
| 5 | Confirm offline access behavior | Offline access is documented |

## Test 4: Publish Labels

| Step | Action | Expected Result |
|---|---|---|
| 1 | Create label policy | Policy wizard opens |
| 2 | Select baseline labels | Labels are selected |
| 3 | Target test group | Test group is selected |
| 4 | Configure policy settings | Defaults and justification are configured |
| 5 | Create policy | Policy is created |

## Test 5: Office App Label Visibility

| Step | Action | Expected Result |
|---|---|---|
| 1 | Sign in to Word as test user | User signs in |
| 2 | Open Sensitivity menu | Published labels appear |
| 3 | Apply General label | Label applies |
| 4 | Apply Confidential label | Label applies |
| 5 | Save document | Labeled document saves |

## Test 6: Visual Marking Validation

| Step | Action | Expected Result |
|---|---|---|
| 1 | Apply Confidential label | Label applies |
| 2 | Check header | Header appears if configured |
| 3 | Check footer | Footer appears if configured |
| 4 | Check watermark | Watermark appears if configured |
| 5 | Print or export test if needed | Markings remain where supported |

## Test 7: Encryption Validation

| Step | Action | Expected Result |
|---|---|---|
| 1 | Apply Highly Confidential label | Label applies |
| 2 | Save protected file | File saves |
| 3 | Open as permitted user | Access succeeds |
| 4 | Open as unauthorized user | Access is blocked |
| 5 | Document result | Evidence is saved |

## Test 8: Outlook Label Validation

| Step | Action | Expected Result |
|---|---|---|
| 1 | Open Outlook as test user | Outlook opens |
| 2 | Create test email | Draft opens |
| 3 | Apply Confidential label | Label applies |
| 4 | Send to permitted user | Message sends |
| 5 | Verify recipient access | Recipient can open if permitted |

## Test 9: SharePoint and OneDrive Validation

| Step | Action | Expected Result |
|---|---|---|
| 1 | Upload labeled file to OneDrive | Upload succeeds |
| 2 | Upload labeled file to SharePoint | Upload succeeds |
| 3 | Open file in browser | Label is recognized where supported |
| 4 | Open file in desktop app | Label persists |
| 5 | Validate unauthorized access if encryption is configured | Unauthorized user is blocked |

## Troubleshooting

| Issue | Likely Cause | Fix |
|---|---|---|
| Sensitivity labels page missing | Missing role or license | Confirm Information Protection Admin or Compliance Administrator |
| Cannot create label | Insufficient permission | Assign proper Purview role |
| Label policy creation blocked | Missing role or license | Confirm role and licensing |
| Labels do not appear in Office apps | Policy propagation delay | Wait and restart Office apps |
| Labels do not appear for user | User not in label policy scope | Confirm group membership and policy target |
| Only some labels appear | Label not selected in policy | Edit policy and publish label |
| Encryption blocks intended users | Permission model too restrictive | Add correct users or groups |
| Unauthorized user can open file | Encryption not enabled or wrong label applied | Confirm label settings and reapply label |
| Visual markings missing | Markings not configured or unsupported app | Confirm label settings and app support |
| Outlook label missing | Policy not applied or Office cache issue | Restart Outlook and confirm user scope |
| SharePoint does not show label immediately | Processing delay | Wait and refresh |
| Label downgrade does not require justification | Policy setting not enabled | Edit label policy |
| Mandatory labeling not enforced | Policy setting not enabled or app unsupported | Confirm policy and client support |
| Audit search has no label events | Ingestion delay or no matching activity | Expand time range and retry |
| PowerShell role group name fails | Tenant role group name differs | List role groups and use exact tenant name |

## Common Commands

## Connect to Exchange Online

    Connect-ExchangeOnline

## Connect to Microsoft Graph

    Connect-MgGraph -Scopes "Directory.Read.All","User.Read.All","Group.Read.All","Sites.Read.All","Files.Read.All"

## Review Compliance and Information Protection Role Groups

    Get-RoleGroup | Where-Object {
        $_.Name -match "Compliance|Information|Protection|DLP|Audit"
    } | Select-Object Name

## Review Compliance Management Members

    Get-RoleGroupMember "Compliance Management"

## Search Recent Label-Related Audit Events

    Search-UnifiedAuditLog `
        -StartDate (Get-Date).AddDays(-7) `
        -EndDate (Get-Date) `
        -ResultSize 500 |
    Where-Object {
        $_.Operations -match "Label|Sensitivity|Protection|Policy"
    }

## Export Recent Label-Related Audit Events

    Search-UnifiedAuditLog `
        -StartDate (Get-Date).AddDays(-7) `
        -EndDate (Get-Date) `
        -ResultSize 500 |
    Where-Object {
        $_.Operations -match "Label|Sensitivity|Protection|Policy"
    } |
    Export-Csv ".\Purview_SensitivityLabel_Admin_Audit.csv" -NoTypeInformation

## Disconnect Sessions

    Disconnect-ExchangeOnline -Confirm:$false
    Disconnect-MgGraph

## Rollback / Cleanup

| Change | Rollback |
|---|---|
| Created Public label | Delete if unused, or leave as baseline |
| Created General label | Delete if unused, or leave as baseline |
| Created Confidential label | Disable, remove from policy, or delete if unused |
| Created Highly Confidential label | Disable, remove from policy, or delete if unused |
| Created label policy | Disable or delete policy |
| Published labels to test group | Remove group from policy |
| Configured mandatory labeling | Disable setting in policy |
| Configured default label | Remove default label setting |
| Configured downgrade justification | Disable if required |
| Configured encryption | Edit label protection settings after testing |
| Uploaded test files | Delete test files |
| Sent test emails | Delete test emails |
| Added temporary groups | Remove groups if no longer required |

## Cleanup Test Files

| Location | Cleanup Action |
|---|---|
| Local workstation | Delete local test documents |
| OneDrive | Delete labeled test files |
| SharePoint | Delete labeled test files |
| Exchange mailbox | Delete test emails |
| Teams files | Delete test files if used |
| Evidence folder | Retain required evidence only |

## Security Notes

- Do not test encryption with production-only data first.
- Do not publish new encryption labels to all users without a pilot.
- Encryption mistakes can lock users out of documents.
- Use test groups for initial label policy rollout.
- Do not target break-glass accounts unless there is a specific reason.
- Keep label names simple.
- Too many labels confuse users.
- Mandatory labeling can disrupt users if deployed without training.
- Default labels can over-classify content if not planned.
- Downgrade justification helps with accountability.
- Treat label activity exports as sensitive evidence.
- Highly Confidential labels should be carefully scoped.
- Review encryption permissions before broad rollout.
- Document label owners and support contacts.

## Production Readiness Review

| Question | Required Answer |
|---|---|
| Are label names simple and clear? | Yes |
| Are label descriptions understandable to users? | Yes |
| Are labels ordered by sensitivity? | Yes |
| Has each label scope been reviewed? | Yes |
| Has encryption been tested with permitted users? | Yes |
| Has encryption been tested with unauthorized users? | Yes |
| Has Office app behavior been validated? | Yes |
| Has Outlook behavior been validated? | Yes |
| Has SharePoint and OneDrive behavior been validated? | Yes |
| Has rollback been documented? | Yes |
| Has help desk support been briefed? | Yes |
| Has user training been prepared? | Yes |
| Has pilot scope been used before broad rollout? | Yes |

## Evidence Collection

| Evidence Item | Collection Method | File / Location |
|---|---|---|
| Sensitivity label list | Screenshot | Evidence folder |
| Public label settings | Screenshot or notes | Evidence folder |
| General label settings | Screenshot or notes | Evidence folder |
| Confidential label settings | Screenshot or notes | Evidence folder |
| Highly Confidential label settings | Screenshot or notes | Evidence folder |
| Encryption settings | Screenshot | Evidence folder |
| Label policy settings | Screenshot | Evidence folder |
| Test user label menu | Screenshot | Evidence folder |
| Marked document | Test file | Evidence folder if safe |
| Protected document access test | Screenshot | Evidence folder |
| Outlook labeled email test | Screenshot | Evidence folder |
| Audit export | PowerShell Export-Csv | Purview_SensitivityLabel_Admin_Audit_YYYYMMDD.csv |

## Verification Checklist

| Check | Pass / Fail |
|---|---|
| Purview portal opens |  |
| Information protection page opens |  |
| Sensitivity labels page opens |  |
| Existing labels reviewed |  |
| Public label created |  |
| General label created |  |
| Confidential label created |  |
| Highly Confidential label created |  |
| Label order reviewed |  |
| Visual markings configured where needed |  |
| Confidential encryption reviewed |  |
| Highly Confidential encryption configured |  |
| Label policy created |  |
| Label policy targeted to test group |  |
| Default label setting reviewed |  |
| Mandatory labeling setting reviewed |  |
| Downgrade justification configured |  |
| Labels appear in Office apps |  |
| Labels appear in Outlook |  |
| Labeled file saves successfully |  |
| Labeled email sends successfully |  |
| Protected content access tested |  |
| Unauthorized access tested if safe |  |
| SharePoint upload tested |  |
| OneDrive upload tested |  |
| Audit evidence exported |  |
| Rollback documented |  |

## Exam / Interview Notes

- Sensitivity labels classify and protect Microsoft 365 content.
- Sensitivity labels are created in Microsoft Purview.
- Label policies publish sensitivity labels to users and groups.
- A label can classify content without encrypting it.
- Encryption should be tested carefully before broad deployment.
- Visual markings include headers, footers, and watermarks.
- Label policies can require users to apply labels.
- Label policies can require justification when users remove or lower a label.
- Default labels can automatically classify new content.
- Label scope determines where labels can apply.
- Labels can apply to files and emails.
- Labels can also apply to groups and sites when configured for those scopes.
- Published labels may take time to appear for users.
- Office app sign-in and licensing affect label visibility.
- Overly complex label taxonomies hurt adoption.
- Start with a pilot group before tenant-wide rollout.
- Audit logs can help validate label and policy administration.
- Sensitivity labels are foundational for information protection, DLP, retention planning, and secure collaboration.

## Related Workbooks

| Workbook | Relationship |
|---|---|
| 01_Configure_Purview_Admin_Portal_Roles_Audit_And_Compliance_Baseline.md | Provides Purview portal, role, and audit baseline |
| 02_Configure_Sensitive_Info_Types_Trainable_Classifiers_And_Data_Classification.md | Provides classification planning and detection foundation |
| 04_Configure_Retention_Labels_Retention_Policies_And_Data_Lifecycle.md | Complements sensitivity labels with lifecycle controls |
| 05_Configure_DLP_For_Exchange_SharePoint_OneDrive_Teams_PowerBI_And_Copilot.md | Uses labels and SITs in DLP policy logic |
| 06_Configure_Endpoint_DLP_Alerts_Events_Reports_And_Response.md | Extends label and classification controls to endpoints |
| 07_Monitor_Content_Explorer_Activity_Explorer_Label_Reports_And_DLP_Alerts.md | Monitors label activity and DLP alerts |
| 08_Troubleshoot_Purview_Labeling_Retention_DLP_Audit_And_Compliance_Alerts.md | Troubleshoots labeling, policy, encryption, and audit issues |