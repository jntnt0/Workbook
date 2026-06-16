# 02_Configure_Sensitive_Info_Types_Trainable_Classifiers_And_Data_Classification

## Objective

Configure the Microsoft Purview data classification foundation used by sensitivity labels, DLP policies, retention workflows, investigation workflows, and compliance monitoring.

This workbook covers:

- Data classification portal review
- Built-in sensitive information type review
- Custom sensitive information type creation
- Sensitive information type testing
- Trainable classifier review
- Custom trainable classifier planning
- Data classification dashboard validation
- Content Explorer and Activity Explorer readiness
- PowerShell validation
- Troubleshooting and rollback

## Lab Context

This workbook builds on the Purview admin portal, roles, audit, and compliance baseline.

Classification should be configured before enforcement policies.

Do this before configuring:

- Sensitivity labels
- Label policies
- Encryption settings
- Retention labels
- Retention policies
- DLP policies
- Endpoint DLP
- Copilot-related DLP controls
- Content Explorer investigations
- Activity Explorer investigations

The goal is to validate that Purview can identify sensitive content before using that detection in labels, DLP, retention, or alerting.

## Prerequisites

| Requirement | Notes |
|---|---|
| Microsoft Purview portal access | Required |
| Compliance Administrator | Required for broad Purview administration |
| Information Protection Admin | Recommended for classification and labeling |
| DLP Compliance Management | Recommended for DLP-related classification use |
| Audit Reader or Audit Manager | Recommended for validation |
| Microsoft 365 E5 or equivalent | Required for some advanced classification features |
| Test SharePoint site | Recommended |
| Test OneDrive account | Recommended |
| Test Exchange mailbox | Recommended |
| Fake sample data | Required |
| No real sensitive data | Required |
| Exchange Online PowerShell module | Recommended |
| Microsoft Graph PowerShell SDK | Optional |
| Baseline workbook 01 completed | Required |

## Topology / Scope

| Component | Purpose |
|---|---|
| Microsoft Purview portal | Main configuration portal |
| Data classification | Central classification visibility area |
| Sensitive information types | Pattern-based detection |
| Built-in sensitive information types | Microsoft-provided detection templates |
| Custom sensitive information types | Tenant-specific detection patterns |
| Trainable classifiers | Machine-learning based document classification |
| Content Explorer | Location-level content investigation |
| Activity Explorer | Activity-level compliance signal review |
| SharePoint Online | File classification test location |
| OneDrive for Business | User file classification test location |
| Exchange Online | Email classification test location |
| Teams | Later DLP and compliance signal source |

## Naming Standards

| Object Type | Naming Example |
|---|---|
| Custom sensitive information type | SIT-Lab-Employee-ID |
| Custom sensitive information type | SIT-Lab-Contract-ID |
| Trainable classifier | TC-Lab-Finance-Contracts |
| Test SharePoint site | SP-Purview-Test |
| Test document | Purview_Test_EmployeeID.docx |
| Test mailbox subject | PURVIEW-SIT-TEST-001 |
| Test security group | GRP-Purview-Test-Scope |
| Evidence export | Purview_DataClassification_YYYYMMDD.csv |
| Admin evidence folder | Purview/Data-Classification |

## Test Data Rules

| Rule | Requirement |
|---|---|
| Use fake data only | Required |
| Do not use real customer data | Required |
| Do not use real employee data | Required |
| Do not use real financial data | Required |
| Do not use real health data | Required |
| Do not use real legal matter data | Required |
| Mark lab files clearly | Required |
| Delete lab files after validation | Recommended |

## Fake Test Data Examples

| Data Type | Fake Example |
|---|---|
| Lab employee ID | LAB-EMP-10001 |
| Lab employee ID | LAB-EMP-10002 |
| Lab contract ID | LAB-CONTRACT-77881 |
| Lab project code | LAB-PROJ-FIN-2026 |
| Lab keyword | PURVIEW-TEST-DATA-ONLY |
| Lab finance marker | LAB-FINANCE-CONTRACT-SAMPLE |
| Lab HR marker | LAB-HR-SAMPLE-DOCUMENT |

## Portal / GUI Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|---|---|---|---|---|---|
| 1 | Open Microsoft Purview portal | Admin workstation | https://purview.microsoft.com | N/A | Purview portal opens |
| 2 | Confirm correct admin account | Admin workstation | Profile menu | N/A | Dedicated admin account is signed in |
| 3 | Open Data classification | Admin workstation | Solutions > Data classification | N/A | Data classification page opens |
| 4 | Review overview dashboard | Admin workstation | Data classification > Overview | N/A | Classification summary is visible |
| 5 | Open Classifiers area | Admin workstation | Data classification > Classifiers | N/A | Classifier options are visible |
| 6 | Open Sensitive info types | Admin workstation | Classifiers > Sensitive info types | N/A | Sensitive information types list opens |
| 7 | Search for Credit Card Number | Admin workstation | Search sensitive info types | N/A | Built-in credit card SIT appears |
| 8 | Open Credit Card Number SIT | Admin workstation | Select sensitive info type | N/A | SIT details are visible |
| 9 | Review confidence levels | Admin workstation | Pattern details | N/A | Detection logic and confidence levels are visible |
| 10 | Search for U.S. Social Security Number | Admin workstation | Search sensitive info types | N/A | Built-in SSN SIT appears |
| 11 | Search for ABA Routing Number | Admin workstation | Search sensitive info types | N/A | Built-in financial SIT appears |
| 12 | Search for Passport Number | Admin workstation | Search sensitive info types | N/A | Built-in identity SIT appears |
| 13 | Create custom sensitive info type | Admin workstation | Create sensitive info type | N/A | Creation wizard opens |
| 14 | Name custom SIT | Admin workstation | Name: SIT-Lab-Employee-ID | N/A | Name is accepted |
| 15 | Add description | Admin workstation | Detects fake lab employee IDs | N/A | Description is saved |
| 16 | Add primary pattern | Admin workstation | LAB-EMP-[0-9]{5} | N/A | Regex pattern is accepted |
| 17 | Add supporting evidence | Admin workstation | PURVIEW-TEST-DATA-ONLY | N/A | Supporting keyword is accepted |
| 18 | Configure proximity | Admin workstation | Default or lab value | N/A | Proximity is configured |
| 19 | Configure confidence level | Admin workstation | Medium or High | N/A | Confidence level is configured |
| 20 | Review custom SIT | Admin workstation | Review settings | N/A | Settings are ready to create |
| 21 | Create custom SIT | Admin workstation | Create | N/A | Custom SIT is created |
| 22 | Test custom SIT | Admin workstation | Test sensitive info type | N/A | Test pane opens |
| 23 | Paste fake matching text | Admin workstation | LAB-EMP-10001 PURVIEW-TEST-DATA-ONLY | N/A | Match is detected |
| 24 | Paste fake nonmatching text | Admin workstation | EMP-10001 | N/A | No match is detected |
| 25 | Create second custom SIT if needed | Admin workstation | SIT-Lab-Contract-ID | N/A | Optional SIT is created |
| 26 | Open Trainable classifiers | Admin workstation | Classifiers > Trainable classifiers | N/A | Trainable classifiers page opens |
| 27 | Review built-in classifiers | Admin workstation | Browse classifier list | N/A | Built-in classifiers are visible |
| 28 | Open a built-in classifier | Admin workstation | Select classifier | N/A | Classifier details are visible |
| 29 | Review custom classifier option | Admin workstation | Create trainable classifier | N/A | Creation option is visible if licensed |
| 30 | Plan custom classifier | Admin workstation | TC-Lab-Finance-Contracts | N/A | Classifier purpose is documented |
| 31 | Upload positive examples if licensed | Admin workstation | Add matching sample files | N/A | Positive examples are accepted |
| 32 | Upload negative examples if licensed | Admin workstation | Add nonmatching sample files | N/A | Negative examples are accepted |
| 33 | Start training if licensed | Admin workstation | Train classifier | N/A | Training starts or queues |
| 34 | Review classifier status | Admin workstation | Classifier status page | N/A | Status is visible |
| 35 | Publish classifier after validation | Admin workstation | Publish | N/A | Classifier becomes available for policies |
| 36 | Return to Data classification overview | Admin workstation | Data classification > Overview | N/A | Dashboard opens |
| 37 | Open Content Explorer if available | Admin workstation | Data classification > Content Explorer | N/A | Content Explorer opens if licensed and permissioned |
| 38 | Open Activity Explorer if available | Admin workstation | Data classification > Activity Explorer | N/A | Activity Explorer opens if licensed and permissioned |
| 39 | Document findings | Admin workstation | Evidence folder | N/A | Baseline notes are captured |
| 40 | Record created classifiers and SITs | Admin workstation | Workbook notes | N/A | Classification inventory is documented |

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
        $_.Name -match "Compliance|Information|DLP|Records|Audit|eDiscovery|Insider|Communication"
    } | Select-Object Name

Expected result:

    Compliance-related role groups are listed.

## Review Information Protection Role Membership

    Get-RoleGroupMember "Information Protection"

Expected result:

    Members are displayed if the role group exists in the tenant.

If the exact role group name differs, list role groups first and use the matching tenant role group name.

## Review Compliance Management Membership

    Get-RoleGroupMember "Compliance Management"

Expected result:

    Members of Compliance Management are displayed.

## Review DLP Role Groups

    Get-RoleGroup | Where-Object {
        $_.Name -match "DLP|Data Loss"
    } | Select-Object Name

Expected result:

    DLP-related role groups are listed.

## Search Audit Log for Recent Compliance Administration

    Search-UnifiedAuditLog `
        -StartDate (Get-Date).AddDays(-7) `
        -EndDate (Get-Date) `
        -ResultSize 100 |
    Where-Object {
        $_.Operations -match "Sensitive|Classifier|Dlp|Compliance|Label|Protection"
    }

Expected result:

    Recent matching Purview or compliance administration events appear if available.

No results can be normal if the tenant has no matching activity or if audit ingestion has not completed.

## Export Recent Compliance Administration Audit Events

    $Date = Get-Date -Format "yyyyMMdd"
    $OutputPath = ".\Purview_Classification_Admin_Audit_$Date.csv"

    Search-UnifiedAuditLog `
        -StartDate (Get-Date).AddDays(-7) `
        -EndDate (Get-Date) `
        -ResultSize 500 |
    Where-Object {
        $_.Operations -match "Sensitive|Classifier|Dlp|Compliance|Label|Protection"
    } |
    Export-Csv $OutputPath -NoTypeInformation

    Get-Item $OutputPath

Expected result:

    CSV export is created.

## Export Compliance Role Groups for Evidence

    $Date = Get-Date -Format "yyyyMMdd"
    $OutputPath = ".\Purview_Classification_RoleGroups_$Date.csv"

    Get-RoleGroup | Where-Object {
        $_.Name -match "Compliance|Information|DLP|Records|Audit|eDiscovery|Insider|Communication"
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

## Sensitive Information Type Concepts

| Concept | Meaning |
|---|---|
| Sensitive information type | A pattern-based classifier Purview uses to detect sensitive content |
| Built-in SIT | Microsoft-provided sensitive information type |
| Custom SIT | Tenant-created sensitive information type |
| Pattern | Detection rule using regex, keywords, functions, or supporting evidence |
| Primary element | Main item being detected |
| Supporting evidence | Context that strengthens the match |
| Confidence level | How likely the match is true |
| Proximity | Distance between primary match and supporting evidence |
| Instance count | Number of matches required before detection triggers |
| False positive | Incorrect match |
| False negative | Missed match |

## Built-In Sensitive Information Type Review Checklist

| Built-In Sensitive Info Type | Use Case | Reviewed |
|---|---|---|
| Credit Card Number | Payment card data detection |  |
| U.S. Social Security Number | U.S. government identifier detection |  |
| ABA Routing Number | Banking routing number detection |  |
| U.S. Bank Account Number | Financial account detection |  |
| Passport Number | Identity document detection |  |
| Driver License Number | Identity document detection |  |
| Tax Identification Number | Tax identifier detection |  |
| Medical Terms and Conditions | Health-related data detection |  |
| IP Address | Technical and network data detection |  |
| Azure Storage Account Key | Cloud secret detection |  |
| Password patterns | Credential exposure detection |  |
| SWIFT Code | Financial transaction detection |  |
| International Banking Account Number | International bank account detection |  |

## Custom Sensitive Information Type Design

| Field | Lab Value |
|---|---|
| Name | SIT-Lab-Employee-ID |
| Description | Detects fake lab employee identifiers |
| Primary pattern | LAB-EMP-[0-9]{5} |
| Supporting keyword | PURVIEW-TEST-DATA-ONLY |
| Example match | LAB-EMP-10001 |
| Example nonmatch | EMP-10001 |
| Confidence | Medium or High |
| Intended use | DLP and classification testing |
| Production ready | No, lab only |

## Custom SIT Pattern Reference

| Pattern Name | Pattern |
|---|---|
| Lab Employee ID | LAB-EMP-[0-9]{5} |
| Lab Contract ID | LAB-CONTRACT-[0-9]{5} |
| Lab Project Code | LAB-PROJ-[A-Z]{3}-[0-9]{4} |
| Lab Case Number | LAB-CASE-[0-9]{6} |
| Lab Finance Marker | LAB-FINANCE-CONTRACT-SAMPLE |

## Custom SIT Test Strings

| Test String | Expected Result |
|---|---|
| LAB-EMP-10001 PURVIEW-TEST-DATA-ONLY | Match |
| LAB-EMP-99999 PURVIEW-TEST-DATA-ONLY | Match |
| EMP-10001 PURVIEW-TEST-DATA-ONLY | No match |
| LAB-EMP-1001 PURVIEW-TEST-DATA-ONLY | No match |
| LAB-EMP-ABCDE PURVIEW-TEST-DATA-ONLY | No match |
| LAB-CONTRACT-77881 | Match only if contract SIT exists |
| LAB-PROJ-FIN-2026 | Match only if project SIT exists |

## Custom SIT Test Document Content

Use this as plain text inside a test Word document, text file, or email body.

    This document is for Microsoft Purview testing only.

    PURVIEW-TEST-DATA-ONLY

    Employee record reference:
    LAB-EMP-10001

    Contract reference:
    LAB-CONTRACT-77881

    Project reference:
    LAB-PROJ-FIN-2026

    This data is fake and must not be used for production decisions.

Expected result:

    SIT-Lab-Employee-ID detects LAB-EMP-10001 when tested.
    Supporting evidence improves confidence when configured.

## Trainable Classifier Concepts

| Concept | Meaning |
|---|---|
| Trainable classifier | Classifier trained to identify content by examples |
| Built-in classifier | Microsoft-provided classifier |
| Custom classifier | Tenant-trained classifier |
| Positive examples | Documents that represent the target content |
| Negative examples | Documents that should not match |
| Training | Learning process for classifier |
| Testing | Validation against sample content |
| Publishing | Makes classifier available for policy use |
| Retraining | Improves classifier accuracy |
| Confidence | Likelihood the content matches the classifier |
| False positive | Content incorrectly classified |
| False negative | Target content not classified |

## Built-In Trainable Classifier Review Checklist

| Classifier Type | Use Case | Reviewed |
|---|---|---|
| Resume | HR and recruiting content |  |
| Source code | Developer or intellectual property content |  |
| Offensive language | Communication compliance |  |
| Threat language | Communication compliance |  |
| Harassment language | Communication compliance |  |
| Profanity | Communication compliance |  |
| Regulatory collusion | Risk and compliance review |  |
| Customer complaints | Regulatory and service review |  |
| Financial information | Finance-sensitive content review |  |

## Custom Trainable Classifier Planning

| Field | Lab Value |
|---|---|
| Name | TC-Lab-Finance-Contracts |
| Description | Identifies fake finance contract documents |
| Positive examples | Fake finance contract sample files |
| Negative examples | Unrelated fake sample files |
| Minimum quality | Clear difference between matching and nonmatching files |
| Target workload | SharePoint and OneDrive |
| Intended use | Future DLP or retention testing |
| Production ready | No, lab only |

## Positive Example File Guidance

| Requirement | Notes |
|---|---|
| Use fake files | Required |
| Use consistent document type | Required |
| Include realistic structure | Recommended |
| Include repeated markers | Recommended for lab training |
| Avoid real customer names | Required |
| Avoid real financial numbers | Required |
| Avoid real contracts | Required |
| Use enough examples | Required by wizard and feature availability |

## Negative Example File Guidance

| Requirement | Notes |
|---|---|
| Use fake files | Required |
| Use unrelated content | Required |
| Avoid target content markers | Required |
| Use similar file formats | Recommended |
| Include business documents that should not match | Recommended |
| Keep files clean and readable | Recommended |

## Example Positive Classifier Text

    LAB-FINANCE-CONTRACT-SAMPLE

    This fake finance contract document is used only for Microsoft Purview trainable classifier testing.

    Contract Type: Lab Services Agreement
    Contract Owner: Lab Finance Team
    Contract Reference: LAB-CONTRACT-77881
    Project Code: LAB-PROJ-FIN-2026

    This file is not a real agreement and contains no production data.

Expected result:

    This type of file should be used as a positive example for TC-Lab-Finance-Contracts.

## Example Negative Classifier Text

    LAB-GENERAL-OPERATIONS-SAMPLE

    This fake operations document is used only for Microsoft Purview trainable classifier testing.

    Topic: Office supply checklist
    Department: Lab Operations
    Project Code: LAB-PROJ-OPS-2026

    This file is not a finance contract.

Expected result:

    This type of file should be used as a negative example for TC-Lab-Finance-Contracts.

## Data Classification Review Areas

| Area | What To Check |
|---|---|
| Overview | High-level sensitive data summary |
| Sensitive info types | Which SITs are being detected |
| Trainable classifiers | Which classifiers exist and are active |
| Content Explorer | Where sensitive content exists |
| Activity Explorer | Which activities involve classified content |
| Label activity | Later sensitivity label dependency |
| DLP activity | Later DLP dependency |
| Retention activity | Later retention dependency |
| Recommended actions | Suggested improvements or risks |

## Content Explorer Access Review

| Check | Expected Result |
|---|---|
| Content Explorer page opens | Admin has required access |
| Sensitive info type filters appear | Classification data can be filtered |
| Locations are visible | SharePoint, OneDrive, Exchange, or Teams data may appear |
| Item details are restricted | Access follows permission model |
| No unnecessary users have access | Least privilege is maintained |

## Activity Explorer Access Review

| Check | Expected Result |
|---|---|
| Activity Explorer page opens | Admin has required access |
| Activity filters appear | Events can be filtered |
| Sensitive info type activity appears | Classification-related activity may be visible |
| Workload filters appear | Exchange, SharePoint, OneDrive, Teams, endpoint options may appear |
| Export or investigation workflow is understood | Evidence can be captured if permitted |

## Validation Tests

## Test 1: Data Classification Portal Access

| Step | Action | Expected Result |
|---|---|---|
| 1 | Open Microsoft Purview portal | Portal opens |
| 2 | Open Data classification | Data classification page opens |
| 3 | Open Classifiers | Classifier options are visible |
| 4 | Open Sensitive info types | SIT list appears |

## Test 2: Built-In SIT Review

| Step | Action | Expected Result |
|---|---|---|
| 1 | Search for Credit Card Number | Built-in SIT appears |
| 2 | Open SIT details | Pattern information appears |
| 3 | Review confidence levels | Detection logic is understood |
| 4 | Repeat with SSN and ABA Routing Number | Additional built-in SITs are reviewed |

## Test 3: Custom SIT Creation

| Step | Action | Expected Result |
|---|---|---|
| 1 | Create sensitive info type | Wizard opens |
| 2 | Enter SIT-Lab-Employee-ID | Name is accepted |
| 3 | Add pattern LAB-EMP-[0-9]{5} | Pattern is accepted |
| 4 | Add keyword PURVIEW-TEST-DATA-ONLY | Supporting evidence is accepted |
| 5 | Create SIT | Custom SIT is saved |

## Test 4: Custom SIT Detection

| Step | Test Content | Expected Result |
|---|---|---|
| 1 | LAB-EMP-10001 PURVIEW-TEST-DATA-ONLY | Match |
| 2 | LAB-EMP-99999 PURVIEW-TEST-DATA-ONLY | Match |
| 3 | EMP-10001 PURVIEW-TEST-DATA-ONLY | No match |
| 4 | LAB-EMP-1001 PURVIEW-TEST-DATA-ONLY | No match |
| 5 | LAB-EMP-ABCDE PURVIEW-TEST-DATA-ONLY | No match |

## Test 5: Trainable Classifier Review

| Step | Action | Expected Result |
|---|---|---|
| 1 | Open Trainable classifiers | Classifier page opens |
| 2 | Review built-in classifiers | Built-in classifiers are visible |
| 3 | Open classifier details | Description and use cases are visible |
| 4 | Start custom classifier wizard if licensed | Wizard opens |
| 5 | Document licensing limitation if unavailable | Gap is recorded |

## Test 6: Data Classification Dashboard

| Step | Action | Expected Result |
|---|---|---|
| 1 | Open Data classification overview | Dashboard loads |
| 2 | Review sensitive info summary | Summary appears |
| 3 | Open Content Explorer if available | Content view opens |
| 4 | Open Activity Explorer if available | Activity view opens |
| 5 | Document findings | Evidence notes are saved |

## Troubleshooting

| Issue | Likely Cause | Fix |
|---|---|---|
| Data classification page missing | Missing role or license | Confirm Purview role and licensing |
| Sensitive info types page missing | Insufficient permissions | Assign Compliance Administrator or Information Protection role |
| Custom SIT creation blocked | Missing admin role | Use correct Purview role group |
| Regex pattern not accepted | Invalid regex syntax | Simplify and retest pattern |
| SIT test returns no match | Pattern mismatch or supporting evidence missing | Use known-good lab string |
| SIT produces too many matches | Pattern too broad | Add supporting evidence and tighten regex |
| SIT produces too few matches | Pattern too strict | Adjust regex or proximity |
| Trainable classifiers missing | Licensing or feature availability issue | Confirm Microsoft 365 E5 or equivalent features |
| Custom classifier wizard unavailable | Unsupported license or permission | Confirm licensing and role group membership |
| Classifier training fails | Poor samples or unsupported files | Use clean positive and negative examples |
| Content Explorer unavailable | Missing role or license | Assign proper role and confirm licensing |
| Activity Explorer shows no data | Ingestion delay or no activity | Generate test activity and wait |
| Audit export has no records | No matching admin events | Expand date range |
| PowerShell role group name fails | Tenant role group name differs | List role groups and use exact name |

## Common Commands

## Connect to Exchange Online

    Connect-ExchangeOnline

## Connect to Microsoft Graph

    Connect-MgGraph -Scopes "Directory.Read.All","User.Read.All","Group.Read.All","Sites.Read.All","Files.Read.All"

## Review Compliance Role Groups

    Get-RoleGroup | Where-Object {
        $_.Name -match "Compliance|Information|DLP|Records|Audit|eDiscovery|Insider|Communication"
    } | Select-Object Name

## Review Compliance Management Members

    Get-RoleGroupMember "Compliance Management"

## Review Information Protection Role Groups

    Get-RoleGroup | Where-Object {
        $_.Name -match "Information|Protection"
    } | Select-Object Name

## Review DLP Role Groups

    Get-RoleGroup | Where-Object {
        $_.Name -match "DLP|Data Loss"
    } | Select-Object Name

## Search Recent Unified Audit Log Events

    Search-UnifiedAuditLog `
        -StartDate (Get-Date).AddDays(-7) `
        -EndDate (Get-Date) `
        -ResultSize 100

## Search Recent Classification-Related Admin Events

    Search-UnifiedAuditLog `
        -StartDate (Get-Date).AddDays(-7) `
        -EndDate (Get-Date) `
        -ResultSize 500 |
    Where-Object {
        $_.Operations -match "Sensitive|Classifier|Dlp|Compliance|Label|Protection"
    }

## Export Recent Compliance Admin Events

    Search-UnifiedAuditLog `
        -StartDate (Get-Date).AddDays(-7) `
        -EndDate (Get-Date) `
        -ResultSize 500 |
    Where-Object {
        $_.Operations -match "Sensitive|Classifier|Dlp|Compliance|Label|Protection"
    } |
    Export-Csv ".\Purview_Classification_Admin_Audit.csv" -NoTypeInformation

## Disconnect Sessions

    Disconnect-ExchangeOnline -Confirm:$false
    Disconnect-MgGraph

## Rollback / Cleanup

| Change | Rollback |
|---|---|
| Created custom SIT | Delete or retire the custom sensitive information type |
| Created lab contract SIT | Delete or retire the custom SIT |
| Created trainable classifier | Delete, unpublish, or leave unpublished |
| Uploaded positive example files | Delete test files |
| Uploaded negative example files | Delete test files |
| Created SharePoint test documents | Delete test documents |
| Created OneDrive test documents | Delete test documents |
| Created Exchange test messages | Delete test messages |
| Exported audit evidence | Store securely or delete lab-only export |
| Added temporary admin role | Remove temporary role assignment |
| Added temporary test group | Remove group if no longer needed |

## Cleanup Test Files

| Location | Cleanup Action |
|---|---|
| SharePoint document library | Delete Purview test files |
| OneDrive test folder | Delete Purview test files |
| Exchange mailbox | Delete test messages |
| Teams files | Delete test files if used |
| Local workstation | Delete temporary sample files |
| Evidence folder | Retain only required evidence |

## Security Notes

- Never test classification with real confidential data.
- Never upload real customer, patient, employee, legal, or payment data for lab testing.
- Custom regex patterns can cause false positives if too broad.
- Supporting evidence improves detection precision.
- Poor classification design causes poor DLP and labeling outcomes.
- Content Explorer can expose sensitive file locations and details.
- Do not grant Content Explorer access broadly.
- Activity Explorer can expose user activity involving sensitive data.
- Treat classification exports as sensitive evidence.
- Use least privilege for classification administrators.
- Review custom SITs before using them in enforcement-mode DLP policies.
- Validate with positive and negative test cases before production use.

## Production Readiness Review

| Question | Required Answer |
|---|---|
| Has the custom SIT been tested with known-good examples? | Yes |
| Has the custom SIT been tested with known-bad examples? | Yes |
| Has supporting evidence been configured where needed? | Yes |
| Are false positives acceptable? | Yes |
| Are false negatives understood? | Yes |
| Has compliance reviewed the detection purpose? | Yes |
| Has legal reviewed regulated detection use cases if needed? | Yes |
| Has the classifier been tested before enforcement? | Yes |
| Is there an owner for tuning? | Yes |
| Is there a rollback plan? | Yes |

## Evidence Collection

| Evidence Item | Collection Method | File / Location |
|---|---|---|
| Built-in SIT review | Screenshot or workbook notes | Evidence folder |
| Custom SIT settings | Screenshot or documented configuration | Evidence folder |
| Custom SIT test result | Screenshot | Evidence folder |
| Classifier list | Screenshot | Evidence folder |
| Trainable classifier plan | Workbook notes | Evidence folder |
| Content Explorer access | Screenshot if available | Evidence folder |
| Activity Explorer access | Screenshot if available | Evidence folder |
| Compliance admin audit export | PowerShell Export-Csv | Purview_Classification_Admin_Audit_YYYYMMDD.csv |
| Role group export | PowerShell Export-Csv | Purview_Classification_RoleGroups_YYYYMMDD.csv |

## Verification Checklist

| Check | Pass / Fail |
|---|---|
| Microsoft Purview portal opens |  |
| Data classification page opens |  |
| Classifiers area opens |  |
| Sensitive info types list opens |  |
| Built-in SITs reviewed |  |
| Credit Card Number SIT reviewed |  |
| SSN SIT reviewed |  |
| Financial SIT reviewed |  |
| Custom SIT created |  |
| Custom SIT positive test passed |  |
| Custom SIT negative test passed |  |
| Supporting evidence configured |  |
| Trainable classifiers reviewed |  |
| Custom classifier planned |  |
| Custom classifier created if licensed |  |
| Content Explorer reviewed if available |  |
| Activity Explorer reviewed if available |  |
| Role permissions validated |  |
| Audit evidence exported |  |
| No real sensitive data used |  |
| Cleanup plan documented |  |

## Exam / Interview Notes

- Sensitive information types are pattern-based classifiers in Microsoft Purview.
- Built-in sensitive information types detect common regulated data such as credit card numbers, government IDs, financial identifiers, and other sensitive patterns.
- Custom sensitive information types are used for organization-specific identifiers such as employee IDs, customer numbers, project codes, and internal case numbers.
- Confidence levels help determine how strong a match must be before detection occurs.
- Supporting evidence improves detection accuracy by requiring contextual keywords or related patterns.
- Proximity controls how close supporting evidence must be to the primary pattern.
- Trainable classifiers identify content based on examples instead of only fixed patterns.
- Trainable classifiers require positive and negative examples.
- Trainable classifiers should be validated before being used in policies.
- Data classification gives visibility before enforcement.
- Content Explorer shows where sensitive content exists.
- Activity Explorer shows activities involving sensitive content.
- Classification should be validated before DLP blocking policies are enabled.
- Bad classification design leads to noisy alerts, false positives, missed detections, and frustrated users.
- Classification is a foundation for sensitivity labels, DLP, retention, audit, communication compliance, and insider risk workflows.

## Related Workbooks

| Workbook                                                                       | Relationship                                                       |
| ------------------------------------------------------------------------------ | ------------------------------------------------------------------ |
| 01_Configure_Purview_Admin_Portal_Roles_Audit_And_Compliance_Baseline.md       | Provides portal, role, and audit baseline                          |
| 03_Configure_Sensitivity_Labels_Label_Policies_And_Encryption_Settings.md      | Uses classification signals for label planning                     |
| 04_Configure_Retention_Labels_Retention_Policies_And_Data_Lifecycle.md         | Uses classification for retention targeting                        |
| 05_Configure_DLP_For_Exchange_SharePoint_OneDrive_Teams_PowerBI_And_Copilot.md | Uses SITs and classifiers for DLP rules                            |
| 06_Configure_Endpoint_DLP_Alerts_Events_Reports_And_Response.md                | Extends classification detection to endpoint DLP                   |
| 07_Monitor_Content_Explorer_Activity_Explorer_Label_Reports_And_DLP_Alerts.md  | Monitors classification results and activity                       |
| 08_Troubleshoot_Purview_Labeling_Retention_DLP_Audit_And_Compliance_Alerts.md  | Troubleshoots SIT, classifier, explorer, and classification issues |