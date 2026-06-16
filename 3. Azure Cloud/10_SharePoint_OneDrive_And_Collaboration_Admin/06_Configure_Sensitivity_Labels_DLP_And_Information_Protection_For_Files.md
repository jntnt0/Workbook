# 06_Configure_Sensitivity_Labels_DLP_And_Information_Protection_For_Files

## Objective

Configure Microsoft Purview sensitivity labels, Data Loss Prevention policies, and information protection controls for SharePoint Online and OneDrive files. This workbook establishes a baseline for labeling files, protecting sensitive content, warning or blocking risky sharing, validating policy enforcement, and documenting file protection behavior.

---

## Prerequisites

| Requirement | Details |
|------------|---------|
| Microsoft 365 Tenant | Active tenant with SharePoint Online and OneDrive licensed |
| Role Assignment | Compliance Administrator, Security Administrator, or Global Administrator |
| SharePoint Role | SharePoint Administrator for site validation |
| Licensing | Microsoft Purview sensitivity labels and DLP-capable licensing |
| Previous Workbook | 04_Configure_External_Sharing_Guest_Access_Link_Settings_And_Access_Reviews.md |
| Related Workbook | 05_Manage_Site_Storage_Versioning_Recycle_Bin_And_Restores.md |
| Microsoft Purview Portal | Accessible at https://purview.microsoft.com |
| SharePoint Admin Center | Accessible from Microsoft 365 Admin Center |
| Test Site | https://contoso.sharepoint.com/sites/FinanceTeam |
| Test User | user01@contoso.onmicrosoft.com |
| Test External User | guest.user@externaldomain.com |
| Test File | SensitiveDataTest.docx |

---

## Lab Topology / Assumptions

| Component | Value |
|------------|---------|
| Tenant Name | Contoso |
| SharePoint Root URL | https://contoso.sharepoint.com |
| SharePoint Admin URL | https://contoso-admin.sharepoint.com |
| Test Site | Finance Team |
| Test Site URL | https://contoso.sharepoint.com/sites/FinanceTeam |
| Test Library | Documents |
| Label 1 | Public |
| Label 2 | Internal |
| Label 3 | Confidential |
| DLP Test Data | U.S. Social Security Number or Credit Card Number test string |
| Baseline DLP Action | Warn first, then block external sharing for high-confidence matches |
| Policy Scope | SharePoint sites and OneDrive accounts |
| Validation Client | Browser and Office web apps |

---

# Phase 1 - Open Required Admin Centers

## Configuration Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|------|--------|---------|------------|----------------|
| 1 | Open Microsoft Purview portal | Admin Workstation | https://purview.microsoft.com | N/A | Purview portal opens |
| 2 | Open SharePoint Admin Center | Admin Workstation | Microsoft 365 Admin Center → Admin centers → SharePoint | N/A | SharePoint Admin Center opens |
| 3 | Open test SharePoint site | Browser | https://contoso.sharepoint.com/sites/FinanceTeam | N/A | Test site opens |
| 4 | Open test document library | Browser | Finance Team → Documents | N/A | Documents library opens |
| 5 | Confirm test user access | Browser | Sign in as user01@contoso.onmicrosoft.com | N/A | Test user can access the library |
| 6 | Confirm admin role access | Purview Portal | Settings → Roles and scopes | N/A | Admin has required Purview permissions |

---

# Phase 2 - Review Existing Sensitivity Labels

## Configuration Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|------|--------|---------|------------|----------------|
| 1 | Open information protection labels | Purview Portal | Solutions → Information protection → Sensitivity labels | N/A | Sensitivity label list opens |
| 2 | Review existing labels | Purview Portal | Sensitivity labels page | N/A | Existing labels identified |
| 3 | Review label order | Purview Portal | Label list order | N/A | Label priority understood |
| 4 | Review published label policies | Purview Portal | Label policies | N/A | Existing publication scope identified |
| 5 | Identify conflicts | Purview Portal | Compare labels and policies | N/A | Duplicate or overlapping labels noted |
| 6 | Document current baseline | Admin Workstation | N/A | N/A | Current label state recorded |

---

# Phase 3 - Create Baseline Sensitivity Labels

## Configuration Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|------|--------|---------|------------|----------------|
| 1 | Start new label wizard | Purview Portal | Information protection → Sensitivity labels → Create a label | N/A | Label wizard opens |
| 2 | Create Public label | Purview Portal | Name: Public | N/A | Public label created |
| 3 | Configure Public description | Purview Portal | Description: Approved for public release | N/A | Description configured |
| 4 | Leave encryption disabled for Public | Purview Portal | Encryption settings | N/A | Public files are not encrypted |
| 5 | Create Internal label | Purview Portal | Name: Internal | N/A | Internal label created |
| 6 | Configure Internal description | Purview Portal | Description: Internal business data | N/A | Description configured |
| 7 | Create Confidential label | Purview Portal | Name: Confidential | N/A | Confidential label created |
| 8 | Configure Confidential description | Purview Portal | Description: Sensitive business or regulated data | N/A | Description configured |
| 9 | Configure optional content marking | Purview Portal | Header/footer/watermark if required | N/A | Visual marking configured if selected |
| 10 | Save labels | Purview Portal | Finish each label wizard | N/A | Labels available for policy publication |

---

# Phase 4 - Configure Encryption for Confidential Label

## Configuration Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|------|--------|---------|------------|----------------|
| 1 | Edit Confidential label | Purview Portal | Information protection → Sensitivity labels → Confidential | N/A | Confidential label settings open |
| 2 | Open encryption settings | Purview Portal | Protection settings → Encryption | N/A | Encryption configuration opens |
| 3 | Enable encryption if required | Purview Portal | Assign permissions now | N/A | Encryption enabled |
| 4 | Add internal users or groups | Purview Portal | Add users/groups, example All Employees | N/A | Internal users permitted |
| 5 | Configure permissions | Purview Portal | Viewer, Reviewer, Co-author, or Co-owner | N/A | Rights assigned |
| 6 | Set offline access duration | Purview Portal | Configure offline access period | N/A | Offline access controlled |
| 7 | Save encryption settings | Purview Portal | Save | N/A | Confidential label protection configured |
| 8 | Document encryption behavior | Admin Workstation | N/A | N/A | Protection behavior recorded |

---

# Phase 5 - Publish Sensitivity Labels to Users

## Configuration Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|------|--------|---------|------------|----------------|
| 1 | Open label policies | Purview Portal | Information protection → Label policies | N/A | Label policy list opens |
| 2 | Start publish labels wizard | Purview Portal | Publish labels | N/A | Publication wizard opens |
| 3 | Select labels | Purview Portal | Public, Internal, Confidential | N/A | Labels selected |
| 4 | Select users and groups | Purview Portal | Include user01 or pilot group | N/A | Pilot scope configured |
| 5 | Configure default label | Purview Portal | Default label: Internal if required | N/A | Default label configured |
| 6 | Require justification for downgrade | Purview Portal | Enable justification for removing or lowering classification | N/A | Downgrade justification enabled |
| 7 | Configure mandatory labeling if required | Purview Portal | Require users to apply a label | N/A | Mandatory labeling configured if selected |
| 8 | Name policy | Purview Portal | SPO-OneDrive-File-Labels-Baseline | N/A | Policy named |
| 9 | Publish policy | Purview Portal | Submit | N/A | Labels published to scoped users |
| 10 | Allow policy propagation | Admin Workstation | Wait for service propagation | N/A | Labels become available in Office apps |

---

# Phase 6 - Enable Sensitivity Labels for SharePoint and OneDrive Files

## Configuration Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|------|--------|---------|------------|----------------|
| 1 | Open Information protection settings | Purview Portal | Information protection → Settings | N/A | Information protection settings open |
| 2 | Review Office file support | Purview Portal | Sensitivity label settings | N/A | Office file labeling support verified |
| 3 | Review SharePoint and OneDrive label support | Purview Portal | Information protection settings | N/A | Cloud file labeling support identified |
| 4 | Confirm coauthoring and protected file support if available | Purview Portal | Settings related to encrypted file support | N/A | Support settings reviewed |
| 5 | Validate labels are scoped to SharePoint/OneDrive users | Purview Portal | Label policies | N/A | Users can apply labels to cloud files |
| 6 | Document propagation expectation | Admin Workstation | N/A | N/A | Admin notes updated |

---

# Phase 7 - Create DLP Policy for SharePoint and OneDrive

## Configuration Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|------|--------|---------|------------|----------------|
| 1 | Open DLP policies | Purview Portal | Solutions → Data Loss Prevention → Policies | N/A | DLP policy page opens |
| 2 | Start new DLP policy | Purview Portal | Create policy | N/A | DLP policy wizard opens |
| 3 | Select template | Purview Portal | Financial, Privacy, or Custom policy | N/A | Policy template selected |
| 4 | Name policy | Purview Portal | SPO-OneDrive-Sensitive-Data-DLP-Baseline | N/A | Policy named |
| 5 | Select locations | Purview Portal | SharePoint sites and OneDrive accounts | N/A | Cloud file locations selected |
| 6 | Scope included sites | Purview Portal | Include all or selected Finance Team site | N/A | SharePoint scope configured |
| 7 | Scope included OneDrive accounts | Purview Portal | Include all or selected users | N/A | OneDrive scope configured |
| 8 | Configure sensitive info condition | Purview Portal | Sensitive info type: U.S. SSN or Credit Card Number | N/A | Detection condition configured |
| 9 | Set instance count and confidence | Purview Portal | Configure threshold | N/A | Match confidence configured |
| 10 | Configure user notifications | Purview Portal | Notify users and show policy tips | N/A | Users receive policy tip |
| 11 | Configure action | Purview Portal | Restrict external sharing or block access for external users | N/A | Risky sharing controlled |
| 12 | Configure incident report | Purview Portal | Send alert/report to admin mailbox | N/A | Incident reporting configured |
| 13 | Start in test mode first | Purview Portal | Test with policy tips | N/A | Policy runs without full enforcement |
| 14 | Submit policy | Purview Portal | Create | N/A | DLP policy created |

---

# Phase 8 - Configure DLP Rule Enforcement

## Configuration Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|------|--------|---------|------------|----------------|
| 1 | Open created DLP policy | Purview Portal | DLP → Policies → SPO-OneDrive-Sensitive-Data-DLP-Baseline | N/A | Policy opens |
| 2 | Review rule conditions | Purview Portal | Rules | N/A | Sensitive info rule visible |
| 3 | Review actions | Purview Portal | Restrict access or sharing | N/A | Enforcement behavior verified |
| 4 | Review user notification text | Purview Portal | User notifications | N/A | Policy tip text configured |
| 5 | Review admin alerts | Purview Portal | Incident reports and alerts | N/A | Admin notification configured |
| 6 | Change from test to enforce after validation | Purview Portal | Turn policy on | N/A | Policy enforcement enabled |
| 7 | Document enforcement date | Admin Workstation | N/A | N/A | Change record updated |

---

# Phase 9 - Upload and Label Test File

## Configuration Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|------|--------|---------|------------|----------------|
| 1 | Open test library | Browser | Finance Team → Documents | N/A | Documents library opens |
| 2 | Create test document | Browser or Word | SensitiveDataTest.docx | N/A | Test document created |
| 3 | Add test sensitive content | Browser or Word | Add approved fake SSN or credit card test data | N/A | Test content added |
| 4 | Upload file to SharePoint | Browser | Upload → SensitiveDataTest.docx | N/A | File uploaded |
| 5 | Open file in Office web app | Browser | Click file | N/A | File opens |
| 6 | Apply sensitivity label | Browser | Sensitivity → Confidential | N/A | Confidential label applied |
| 7 | Save file | Browser | Save or auto-save | N/A | Label persists |
| 8 | Verify label display | Browser | File information or sensitivity menu | N/A | Confidential label visible |

---

# Phase 10 - Validate DLP Policy Tips and Sharing Protection

## Configuration Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|------|--------|---------|------------|----------------|
| 1 | Wait for DLP scan | Admin Workstation | Allow service processing time | N/A | DLP evaluates file |
| 2 | Open test file as owner | Browser | Open SensitiveDataTest.docx | N/A | File opens |
| 3 | Check for policy tip | Browser | Review policy notification | N/A | DLP policy tip appears if matched |
| 4 | Attempt external share | Browser | Share → Specific people → guest.user@externaldomain.com | N/A | Sharing warning or block occurs |
| 5 | Validate external access restriction | External Browser Session | Open shared link as guest | N/A | Access is blocked or restricted per policy |
| 6 | Confirm internal access still works | Browser | Open as user01 | N/A | Internal user access works |
| 7 | Review DLP alert | Purview Portal | DLP → Alerts or Activity explorer | N/A | DLP event appears |
| 8 | Document result | Admin Workstation | N/A | N/A | Validation evidence recorded |

---

# Phase 11 - Configure Auto-Labeling Review Baseline

## Configuration Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|------|--------|---------|------------|----------------|
| 1 | Open auto-labeling policies | Purview Portal | Information protection → Auto-labeling | N/A | Auto-labeling page opens |
| 2 | Start new auto-labeling policy if licensed | Purview Portal | Create auto-labeling policy | N/A | Wizard opens |
| 3 | Select sensitive information type | Purview Portal | U.S. SSN, credit card, or business-specific type | N/A | Condition selected |
| 4 | Select label to apply | Purview Portal | Confidential | N/A | Label selected |
| 5 | Select locations | Purview Portal | SharePoint sites and OneDrive accounts | N/A | Cloud file locations selected |
| 6 | Use simulation mode first | Purview Portal | Run policy in simulation | N/A | Auto-labeling does not immediately enforce |
| 7 | Review simulation results | Purview Portal | Auto-labeling policy → Simulation results | N/A | Matched files identified |
| 8 | Turn on policy only after approval | Purview Portal | Turn on policy | N/A | Auto-labeling enforcement enabled if approved |
| 9 | Document scope and risk | Admin Workstation | N/A | N/A | Auto-labeling scope recorded |

---

# Phase 12 - Review Activity Explorer and Content Explorer

## Configuration Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|------|--------|---------|------------|----------------|
| 1 | Open Activity explorer | Purview Portal | Data classification → Activity explorer | N/A | Activity explorer opens |
| 2 | Filter by sensitivity label | Purview Portal | Label = Confidential | N/A | Label activities displayed |
| 3 | Filter by SharePoint workload | Purview Portal | Workload = SharePoint | N/A | SharePoint file activity visible |
| 4 | Filter by OneDrive workload | Purview Portal | Workload = OneDrive | N/A | OneDrive file activity visible |
| 5 | Open Content explorer if permitted | Purview Portal | Data classification → Content explorer | N/A | Content explorer opens |
| 6 | Review sensitive info locations | Purview Portal | Filter by sensitive info type | N/A | Sensitive content locations identified |
| 7 | Export evidence if allowed | Purview Portal | Export | N/A | Evidence exported |
| 8 | Document findings | Admin Workstation | N/A | N/A | Findings recorded |

---

# Phase 13 - Optional Purview PowerShell Validation

## Configuration Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|------|--------|---------|------------|----------------|
| 1 | Open PowerShell | Admin Workstation | Start PowerShell | N/A | PowerShell opens |
| 2 | Install Exchange Online module if missing | Admin Workstation | N/A | Install-Module ExchangeOnlineManagement -Scope CurrentUser | Module installed |
| 3 | Import module | Admin Workstation | N/A | Import-Module ExchangeOnlineManagement | Module loaded |
| 4 | Connect to Purview PowerShell | Admin Workstation | N/A | Connect-IPPSSession | Purview PowerShell connected |
| 5 | List sensitivity labels | Admin Workstation | N/A | Get-Label | Labels displayed |
| 6 | List label policies | Admin Workstation | N/A | Get-LabelPolicy | Label policies displayed |
| 7 | List DLP policies | Admin Workstation | N/A | Get-DlpCompliancePolicy | DLP policies displayed |
| 8 | List DLP rules | Admin Workstation | N/A | Get-DlpComplianceRule | DLP rules displayed |
| 9 | Export policy data | Admin Workstation | N/A | Get-DlpCompliancePolicy \| Export-Csv .\Purview-DLP-Policies.csv -NoTypeInformation | DLP policy export created |
| 10 | Disconnect session | Admin Workstation | N/A | Disconnect-ExchangeOnline -Confirm:$false | Session disconnected |

---

# Phase 14 - Export Label and DLP Baseline

## Configuration Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|------|--------|---------|------------|----------------|
| 1 | Export labels | Admin Workstation | N/A | Get-Label \| Select Name, DisplayName, Guid, Disabled \| Export-Csv .\Purview-SensitivityLabels.csv -NoTypeInformation | Labels exported |
| 2 | Export label policies | Admin Workstation | N/A | Get-LabelPolicy \| Select Name, Enabled, Mode \| Export-Csv .\Purview-LabelPolicies.csv -NoTypeInformation | Label policies exported |
| 3 | Export DLP policies | Admin Workstation | N/A | Get-DlpCompliancePolicy \| Select Name, Mode, Workload, Enabled \| Export-Csv .\Purview-DLPPolicies.csv -NoTypeInformation | DLP policies exported |
| 4 | Export DLP rules | Admin Workstation | N/A | Get-DlpComplianceRule \| Select Name, Disabled, Mode \| Export-Csv .\Purview-DLPRules.csv -NoTypeInformation | DLP rules exported |
| 5 | Confirm exported files | Admin Workstation | N/A | Get-ChildItem .\Purview-*.csv | Export files listed |
| 6 | Store baseline evidence | Admin Workstation | N/A | N/A | Evidence archived |

---

# Validation Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|------|--------|---------|------------|----------------|
| 1 | Verify labels exist | Purview Portal | Information protection → Sensitivity labels | N/A | Public, Internal, and Confidential labels visible |
| 2 | Verify label policy exists | Purview Portal | Information protection → Label policies | N/A | SPO-OneDrive label policy visible |
| 3 | Verify DLP policy exists | Purview Portal | Data Loss Prevention → Policies | N/A | SPO-OneDrive DLP policy visible |
| 4 | Verify user can see labels | Browser | Open Office web file → Sensitivity | N/A | Labels available to user |
| 5 | Verify Confidential label applies | Browser | Apply Confidential to test file | N/A | Label shown on file |
| 6 | Verify protected file behavior | Browser | Open labeled file as permitted user | N/A | Authorized user can open file |
| 7 | Verify external sharing protection | Browser | Try sharing sensitive file externally | N/A | Warning or block appears |
| 8 | Verify DLP event appears | Purview Portal | DLP alerts or Activity explorer | N/A | DLP match visible |
| 9 | Verify policy tips | Browser | Open or share matching file | N/A | User sees policy tip |
| 10 | Verify PowerShell policy export | Admin Workstation | N/A | Test-Path .\Purview-DLPPolicies.csv | Export file exists |

---

# Troubleshooting

| Issue | Cause | Resolution |
|------|-------|------------|
| Sensitivity labels not visible to user | Label policy not published or propagation delay | Confirm label policy scope and wait for propagation |
| User cannot apply label | Missing license, unsupported client, or policy not scoped | Verify licensing, use Office web app, and confirm policy assignment |
| Confidential file cannot be opened | Encryption permissions too restrictive | Review label encryption permissions and add required users/groups |
| DLP policy does not trigger | Content not indexed yet, wrong sensitive info type, or policy in test mode | Wait for processing, verify test data, check policy mode |
| External sharing still succeeds | DLP action not configured to restrict sharing | Edit DLP rule action to block or restrict external access |
| Policy tips missing | User notifications disabled or policy not matched | Enable user notifications and verify rule condition |
| Activity explorer shows no events | Data classification delay or permissions missing | Wait for event ingestion and verify role membership |
| Content explorer unavailable | Missing role or licensing | Assign Content Explorer Content Viewer or required Purview role |
| PowerShell Get-Label fails | Not connected to Purview PowerShell | Run Connect-IPPSSession |
| DLP export command fails | Missing compliance role | Assign Compliance Administrator or DLP Compliance Management role |

---

# Rollback / Cleanup

| Step | Action | PowerShell |
|------|--------|------------|
| 1 | Remove test file | Delete SensitiveDataTest.docx from SharePoint library |
| 2 | Remove external sharing test access | Use Manage access on the test file |
| 3 | Disable DLP policy if lab-only | Set-DlpCompliancePolicy -Identity "SPO-OneDrive-Sensitive-Data-DLP-Baseline" -Mode Disable |
| 4 | Remove DLP rule if lab-only | Remove-DlpComplianceRule -Identity "<RuleName>" -Confirm:$false |
| 5 | Remove DLP policy if lab-only | Remove-DlpCompliancePolicy -Identity "SPO-OneDrive-Sensitive-Data-DLP-Baseline" -Confirm:$false |
| 6 | Disable label policy if lab-only | Set-LabelPolicy -Identity "SPO-OneDrive-File-Labels-Baseline" -Enabled $false |
| 7 | Remove lab-only sensitivity labels only after policy cleanup | Remove-Label -Identity "<LabelName>" -Confirm:$false |
| 8 | Remove exported evidence files if required | Remove-Item .\Purview-*.csv |
| 9 | Disconnect Purview PowerShell | Disconnect-ExchangeOnline -Confirm:$false |

---

# Related Workbooks

```text
01_Configure_SharePoint_Admin_Center_Tenant_Settings_And_Site_Baseline.md
02_Manage_SharePoint_Sites_Hub_Sites_Permissions_And_Sharing.md
03_Configure_OneDrive_Admin_Settings_Sync_Sharing_And_Retention_Baseline.md
04_Configure_External_Sharing_Guest_Access_Link_Settings_And_Access_Reviews.md
05_Manage_Site_Storage_Versioning_Recycle_Bin_And_Restores.md
07_Troubleshoot_SharePoint_OneDrive_Sync_Sharing_Permissions_And_Restore_Issues.md
```