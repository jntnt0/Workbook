# Configure_Retention_Archive_Litigation_Hold_And_eDiscovery_Handoff

# Configure_Retention_Archive_Litigation_Hold_And_eDiscovery_Handoff_Index
07_Configure_Retention_Archive_Litigation_Hold_And_eDiscovery_Handoff.md
Configure_Retention_Archive_Litigation_Hold_And_eDiscovery_Handoff
Configure_Retention_Archive_Litigation_Hold_And_eDiscovery_Handoff_Source_Basis
Configure_Retention_Archive_Litigation_Hold_And_eDiscovery_Handoff_Mental_Model
Configure_Retention_Archive_Litigation_Hold_And_eDiscovery_Handoff_Planning_Table
Configure_Retention_Archive_Litigation_Hold_And_eDiscovery_Handoff_Configuration_Checklist
Configure_Retention_Archive_Litigation_Hold_And_eDiscovery_Handoff_Precheck_Skeleton
Configure_Retention_Archive_Litigation_Hold_And_eDiscovery_Handoff_Archive_Mailbox_Skeleton
Configure_Retention_Archive_Litigation_Hold_And_eDiscovery_Handoff_Exchange_MRM_Retention_Skeleton
Configure_Retention_Archive_Litigation_Hold_And_eDiscovery_Handoff_Purview_Retention_Handoff_Skeleton
Configure_Retention_Archive_Litigation_Hold_And_eDiscovery_Handoff_Litigation_Hold_Skeleton
Configure_Retention_Archive_Litigation_Hold_And_eDiscovery_Handoff_Recoverable_Items_Skeleton
Configure_Retention_Archive_Litigation_Hold_And_eDiscovery_Handoff_eDiscovery_Handoff_Package_Skeleton
Configure_Retention_Archive_Litigation_Hold_And_eDiscovery_Handoff_Bulk_Inventory_Skeleton
Configure_Retention_Archive_Litigation_Hold_And_eDiscovery_Handoff_Validation_Skeleton
Configure_Retention_Archive_Litigation_Hold_And_eDiscovery_Handoff_Verification_Commands
Configure_Retention_Archive_Litigation_Hold_And_eDiscovery_Handoff_Rollback
Configure_Retention_Archive_Litigation_Hold_And_eDiscovery_Handoff_Failure_Checks
Configure_Retention_Archive_Litigation_Hold_And_eDiscovery_Handoff_Related_Labs

# Configure_Retention_Archive_Litigation_Hold_And_eDiscovery_Handoff_Source_Basis

| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Microsoft Purview retention policies | Retain, delete, or retain then delete content across Microsoft 365 locations |
| Microsoft Learn | Purview retention policy supported locations | Exchange mailboxes, SharePoint sites, OneDrive accounts, Microsoft 365 Group mailboxes and sites, Teams messages, and other workloads |
| Microsoft Learn | Exchange Online archive mailboxes | Enabling archive mailboxes and validating archive status |
| Microsoft Learn | Exchange Online retention tags and retention policies | Messaging records management policy structure for Exchange mailbox retention |
| Microsoft Learn | Litigation hold in Exchange Online | Preserving mailbox content for legal or investigation purposes |
| Microsoft Learn | Recoverable Items folder | Validating preserved, deleted, and held mailbox items |
| Microsoft Learn | Microsoft Purview eDiscovery | Case, search, hold, export, custodian, and review workflow handoff |
| Microsoft Learn | eDiscovery retirement notice | Classic eDiscovery experiences retired August 31, 2025, with new Purview eDiscovery experience as the main path |
| Prior workbook | `01_Configure_Exchange_Admin_Center_PowerShell_And_Org_Baseline.md` | Provides Exchange Online PowerShell and RBAC baseline |
| Prior workbook | `02_Manage_User_Mailboxes_Shared_Mailboxes_Contacts_And_Resources.md` | Provides mailbox targets |
| Prior workbook | `05_Configure_Mailbox_Permissions_Delegation_Forwarding_And_Auditing.md` | Provides mailbox auditing and delegated access evidence |
| Prior workbook | `06_Configure_Exchange_Online_Protection_AntiSpam_AntiMalware_And_Quarantine.md` | Provides quarantine and investigation evidence that can become compliance handoff material |

# Configure_Retention_Archive_Litigation_Hold_And_eDiscovery_Handoff_Mental_Model

| Concept | Operational Meaning |
|---|---|
| Archive mailbox | Secondary mailbox storage used for older mailbox data |
| In-place archive | Exchange Online archive mailbox associated with a primary mailbox |
| Auto-expanding archive | Archive capacity expansion feature for qualifying archive mailbox scenarios |
| Exchange MRM | Messaging records management, Exchange retention tags and retention policies applied to mailboxes |
| Retention tag | Retention action applied to folders or items, such as move to archive or delete |
| Retention policy | Collection of retention tags assigned to a mailbox |
| Managed Folder Assistant | Background process that applies Exchange MRM retention settings |
| Purview retention policy | Compliance retention policy for Microsoft 365 locations such as Exchange mailboxes, SharePoint, OneDrive, and groups |
| Retain | Preserve content for a retention period |
| Delete | Delete content after a retention period |
| Retain then delete | Preserve content first, then delete after the configured period |
| Static scope | Retention policy scope based on selected locations, users, groups, or sites |
| Adaptive scope | Retention policy scope based on attributes and rules |
| Litigation hold | Mailbox hold that preserves mailbox content for legal or investigation needs |
| Hold duration | How long content is preserved under litigation hold |
| Recoverable Items | Hidden mailbox storage area where deleted and held items are preserved |
| Delay hold | Temporary hold behavior that can remain after hold removal |
| eDiscovery case | Purview container for investigation workflow, access, holds, searches, review, and export |
| Custodian | Person whose data is relevant to an investigation |
| Non-custodial data source | Location relevant to a case but not assigned to a person |
| eDiscovery handoff | Admin package containing mailbox, hold, archive, retention, audit, and scope evidence for legal or compliance team |
| First rule | Archive is storage management, hold is preservation, retention is lifecycle policy, eDiscovery is investigation workflow |
| Second rule | Do not remove holds or retention policies without legal/compliance approval |
| Third rule | Exchange MRM and Purview retention can both apply, so document precedence and intended location |
| Fourth rule | Classic eDiscovery should not be your default path in 2026 unless operating in an exception environment |

# Configure_Retention_Archive_Litigation_Hold_And_eDiscovery_Handoff_Planning_Table

| Item | Example | Decision |
|---|---|---|
| Tenant name | `contoso.onmicrosoft.com` | `<tenant-name>` |
| Admin UPN | `admin@contoso.com` | `<admin-upn>` |
| Evidence root | `C:\M365-Exchange-Baseline` | `<evidence-root>` |
| Target mailbox | `alex.wilber@contoso.com` | `<target-mailbox>` |
| Shared mailbox target | `helpdesk@contoso.com` | `<shared-mailbox>` |
| Archive required | Yes / No | `<yes-no>` |
| Auto-expanding archive required | Yes / No | `<yes-no>` |
| Exchange MRM policy required | Yes / No | `<yes-no>` |
| MRM policy name | `LAB - Exchange MRM 7 Year Archive` | `<mrm-policy-name>` |
| Archive tag name | `LAB - Move to Archive after 2 Years` | `<archive-tag-name>` |
| Delete tag name | `LAB - Delete after 7 Years` | `<delete-tag-name>` |
| Purview retention required | Yes / No | `<yes-no>` |
| Purview retention policy name | `LAB - Exchange Retain 7 Years` | `<purview-policy-name>` |
| Retention location | Exchange mailboxes | `<location>` |
| Retention action | Retain only / Delete only / Retain then delete | `<retention-action>` |
| Retention duration | `2555` days | `<duration>` |
| Litigation hold required | Yes / No | `<yes-no>` |
| Litigation hold duration | Unlimited / `2555` days | `<hold-duration>` |
| Litigation hold owner | `legal@contoso.com` | `<hold-owner>` |
| Litigation hold note | `Legal hold for case LAB-CASE-001` | `<hold-note>` |
| eDiscovery case name | `LAB-CASE-001` | `<case-name>` |
| eDiscovery manager | `ediscovery.manager@contoso.com` | `<ediscovery-manager>` |
| Legal contact | `legal@contoso.com` | `<legal-contact>` |
| Investigation date range | `2026-01-01 to 2026-06-15` | `<date-range>` |
| Keywords | `"project phoenix"` | `<keywords>` |
| Export required | Yes / No | `<yes-no>` |
| Change ticket | `LAB-EXO-007` | `<ticket-or-lab-id>` |
| Rollback stance | Remove lab archive, MRM, and hold only when approved | `<rollback-scope>` |

# Configure_Retention_Archive_Litigation_Hold_And_eDiscovery_Handoff_Configuration_Checklist

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm Exchange Online session | Admin Workstation | `Get-ConnectionInformation` | Active Exchange Online session exists |
| 2 | Confirm target mailbox exists | Admin Workstation | `Get-EXOMailbox -Identity "<mailbox>"` | Mailbox returns |
| 3 | Confirm mailbox compliance baseline | Admin Workstation | `Get-Mailbox -Identity "<mailbox>" \| Format-List Archive*,RetentionPolicy,LitigationHold*` | Current archive, retention, and hold state is captured |
| 4 | Export mailbox statistics | Admin Workstation | `Get-MailboxStatistics -Identity "<mailbox>"` | Mailbox size and item count are saved |
| 5 | Export archive statistics if present | Admin Workstation | `Get-MailboxStatistics -Identity "<mailbox>" -Archive` | Archive size and item count are saved if archive exists |
| 6 | Export Recoverable Items state | Admin Workstation | `Get-MailboxFolderStatistics -Identity "<mailbox>" -FolderScope RecoverableItems` | Deleted and held item folders are visible |
| 7 | Enable archive mailbox if required | Admin Workstation | `Enable-Mailbox -Identity "<mailbox>" -Archive` | Archive mailbox is enabled |
| 8 | Validate archive mailbox | Admin Workstation | `Get-Mailbox -Identity "<mailbox>" \| Format-List ArchiveStatus,ArchiveGuid,ArchiveName` | Archive state is active |
| 9 | Configure Exchange MRM tags if required | Admin Workstation | `New-RetentionPolicyTag` | Retention tags are created |
| 10 | Configure Exchange MRM policy if required | Admin Workstation | `New-RetentionPolicy` | MRM policy exists |
| 11 | Assign Exchange MRM policy to mailbox | Admin Workstation | `Set-Mailbox -Identity "<mailbox>" -RetentionPolicy "<policy>"` | Mailbox has assigned retention policy |
| 12 | Start Managed Folder Assistant if appropriate | Admin Workstation | `Start-ManagedFolderAssistant -Identity "<mailbox>"` | MRM processing is queued |
| 13 | Connect to Purview compliance PowerShell if needed | Admin Workstation | `Connect-IPPSSession -UserPrincipalName "<admin-upn>"` | Compliance PowerShell session connects |
| 14 | Create Purview retention policy if required | Admin Workstation / Purview portal | `New-RetentionCompliancePolicy` or portal workflow | Retention policy exists |
| 15 | Create Purview retention rule if required | Admin Workstation / Purview portal | `New-RetentionComplianceRule` or portal workflow | Retention rule exists |
| 16 | Place mailbox on litigation hold if required | Admin Workstation | `Set-Mailbox -Identity "<mailbox>" -LitigationHoldEnabled $true` | Mailbox hold is enabled |
| 17 | Validate litigation hold | Admin Workstation | `Get-Mailbox -Identity "<mailbox>" \| Format-List LitigationHold*` | Hold state, duration, owner, and note match design |
| 18 | Export hold and retention inventory | Admin Workstation | Run bulk inventory skeleton | Compliance evidence is saved |
| 19 | Prepare eDiscovery handoff package | Admin Workstation | Run eDiscovery handoff skeleton | Legal/compliance package is created |
| 20 | Verify Purview eDiscovery access path | Browser | Microsoft Purview portal > eDiscovery | eDiscovery area opens |
| 21 | Hand off case data to eDiscovery manager | Operator | Provide evidence package, target mailbox, dates, keywords, and hold state | Investigation team has required scope |
| 22 | Disconnect sessions | Admin Workstation | `Disconnect-ExchangeOnline -Confirm:$false` | Exchange session closes |

# Configure_Retention_Archive_Litigation_Hold_And_eDiscovery_Handoff_Precheck_Skeleton

```powershell
# Run on the admin workstation.
# Purpose: capture mailbox archive, retention, hold, recoverable items, and compliance baseline before changes.

$AdminUpn = "admin@contoso.com"
$TargetMailbox = "alex.wilber@contoso.com"
$SharedMailbox = "helpdesk@contoso.com"
$EvidenceRoot = "C:\M365-Exchange-Baseline"
$RunStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$EvidencePath = Join-Path $EvidenceRoot "07-Retention-Archive-Hold-Precheck-$RunStamp"

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\retention-archive-hold-precheck-transcript.txt"

Import-Module ExchangeOnlineManagement

try {
  $Connection = Get-ConnectionInformation -ErrorAction Stop
}
catch {
  Connect-ExchangeOnline -UserPrincipalName $AdminUpn -ShowBanner:$false
}

Get-ConnectionInformation |
  Tee-Object "$EvidencePath\connection-information.txt"

# Validate mailbox targets.
Get-EXOMailbox -Identity $TargetMailbox |
  Format-List DisplayName,UserPrincipalName,PrimarySmtpAddress,RecipientTypeDetails,ArchiveStatus,ArchiveGuid |
  Out-File "$EvidencePath\target-mailbox-exo.txt"

Get-Mailbox -Identity $TargetMailbox |
  Format-List DisplayName,PrimarySmtpAddress,RecipientTypeDetails,ArchiveStatus,ArchiveGuid,ArchiveName,RetentionPolicy,LitigationHoldEnabled,LitigationHoldDuration,LitigationHoldDate,LitigationHoldOwner,LitigationHoldDuration,LitigationHoldEnabled,LitigationHoldDate,LitigationHoldOwner,LitigationHoldDuration,InPlaceHolds,DelayHoldApplied,DelayReleaseHoldApplied,SingleItemRecoveryEnabled,RetainDeletedItemsFor |
  Out-File "$EvidencePath\target-mailbox-compliance-baseline.txt"

# Shared mailbox baseline if in scope.
Get-Mailbox -Identity $SharedMailbox -ErrorAction SilentlyContinue |
  Format-List DisplayName,PrimarySmtpAddress,RecipientTypeDetails,ArchiveStatus,ArchiveGuid,RetentionPolicy,LitigationHoldEnabled,LitigationHoldDuration,InPlaceHolds |
  Out-File "$EvidencePath\shared-mailbox-compliance-baseline.txt"

# Mailbox statistics.
Get-MailboxStatistics -Identity $TargetMailbox |
  Format-List DisplayName,ItemCount,TotalItemSize,DeletedItemCount,TotalDeletedItemSize,LastLogonTime |
  Out-File "$EvidencePath\target-mailbox-statistics.txt"

# Archive statistics if archive exists.
Get-MailboxStatistics -Identity $TargetMailbox -Archive -ErrorAction SilentlyContinue |
  Format-List DisplayName,ItemCount,TotalItemSize,DeletedItemCount,TotalDeletedItemSize,LastLogonTime |
  Out-File "$EvidencePath\target-archive-statistics.txt"

# Recoverable Items folder statistics.
Get-MailboxFolderStatistics -Identity $TargetMailbox -FolderScope RecoverableItems |
  Select-Object Name,FolderPath,ItemsInFolder,FolderAndSubfolderSize |
  Export-Csv "$EvidencePath\recoverable-items-folder-statistics-before.csv" -NoTypeInformation

# Existing Exchange MRM retention tags and policies.
Get-RetentionPolicyTag |
  Select-Object Name,Type,RetentionEnabled,AgeLimitForRetention,RetentionAction |
  Export-Csv "$EvidencePath\retention-policy-tags-before.csv" -NoTypeInformation

Get-RetentionPolicy |
  Select-Object Name,RetentionPolicyTagLinks,IsDefault |
  Export-Csv "$EvidencePath\retention-policies-before.csv" -NoTypeInformation

# Existing mailbox retention policy assignments.
Get-EXOMailbox -ResultSize 200 |
  Select-Object DisplayName,PrimarySmtpAddress,RecipientTypeDetails,ArchiveStatus,RetentionPolicy |
  Export-Csv "$EvidencePath\mailbox-retention-assignments-sample-before.csv" -NoTypeInformation

Stop-Transcript
```

# Configure_Retention_Archive_Litigation_Hold_And_eDiscovery_Handoff_Archive_Mailbox_Skeleton

```powershell
# Run on the admin workstation.
# Purpose: enable and validate archive mailbox for a target mailbox.

$TargetMailbox = "alex.wilber@contoso.com"
$EvidenceRoot = "C:\M365-Exchange-Baseline"
$RunStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$EvidencePath = Join-Path $EvidenceRoot "07-Archive-Mailbox-$RunStamp"

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\archive-mailbox-transcript.txt"

# Capture before state.
Get-Mailbox -Identity $TargetMailbox |
  Format-List DisplayName,PrimarySmtpAddress,ArchiveStatus,ArchiveGuid,ArchiveName,ArchiveQuota,ArchiveWarningQuota,AutoExpandingArchiveEnabled |
  Out-File "$EvidencePath\archive-before.txt"

Get-MailboxStatistics -Identity $TargetMailbox |
  Format-List DisplayName,ItemCount,TotalItemSize,DeletedItemCount,TotalDeletedItemSize |
  Out-File "$EvidencePath\primary-mailbox-statistics-before.txt"

Get-MailboxStatistics -Identity $TargetMailbox -Archive -ErrorAction SilentlyContinue |
  Format-List DisplayName,ItemCount,TotalItemSize,DeletedItemCount,TotalDeletedItemSize |
  Out-File "$EvidencePath\archive-statistics-before.txt"

# Enable archive if not already active.
$Mailbox = Get-Mailbox -Identity $TargetMailbox

if ($Mailbox.ArchiveStatus -ne "Active") {
  Enable-Mailbox -Identity $TargetMailbox -Archive
}

# Optional: enable auto-expanding archive if approved and licensed.
# Validate tenant and licensing requirements first.
# Enable-Mailbox -Identity $TargetMailbox -AutoExpandingArchive

# Validate after state.
Get-Mailbox -Identity $TargetMailbox |
  Format-List DisplayName,PrimarySmtpAddress,ArchiveStatus,ArchiveGuid,ArchiveName,ArchiveQuota,ArchiveWarningQuota,AutoExpandingArchiveEnabled |
  Out-File "$EvidencePath\archive-after.txt"

Get-MailboxStatistics -Identity $TargetMailbox -Archive -ErrorAction SilentlyContinue |
  Format-List DisplayName,ItemCount,TotalItemSize,DeletedItemCount,TotalDeletedItemSize |
  Out-File "$EvidencePath\archive-statistics-after.txt"

Stop-Transcript
```

# Configure_Retention_Archive_Litigation_Hold_And_eDiscovery_Handoff_Exchange_MRM_Retention_Skeleton

```powershell
# Run on the admin workstation.
# Purpose: create Exchange MRM retention tags, create a retention policy, assign it to a mailbox, and trigger processing.
# Use this for Exchange mailbox lifecycle management, not legal hold.

$TargetMailbox = "alex.wilber@contoso.com"
$EvidenceRoot = "C:\M365-Exchange-Baseline"
$RunStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$EvidencePath = Join-Path $EvidenceRoot "07-Exchange-MRM-Retention-$RunStamp"

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\exchange-mrm-retention-transcript.txt"

# Variables.
$ArchiveTagName = "LAB - Move to Archive after 2 Years"
$DeleteTagName = "LAB - Delete after 7 Years"
$RetentionPolicyName = "LAB - Exchange MRM 7 Year Archive"

# Capture before state.
Get-RetentionPolicyTag |
  Select-Object Name,Type,RetentionEnabled,AgeLimitForRetention,RetentionAction |
  Export-Csv "$EvidencePath\retention-tags-before.csv" -NoTypeInformation

Get-RetentionPolicy |
  Select-Object Name,RetentionPolicyTagLinks,IsDefault |
  Export-Csv "$EvidencePath\retention-policies-before.csv" -NoTypeInformation

Get-Mailbox -Identity $TargetMailbox |
  Format-List DisplayName,PrimarySmtpAddress,RetentionPolicy,ArchiveStatus |
  Out-File "$EvidencePath\mailbox-retention-before.txt"

# Create archive tag if missing.
if (-not (Get-RetentionPolicyTag -Identity $ArchiveTagName -ErrorAction SilentlyContinue)) {
  New-RetentionPolicyTag `
    -Name $ArchiveTagName `
    -Type All `
    -RetentionEnabled $true `
    -AgeLimitForRetention 730 `
    -RetentionAction MoveToArchive
}

# Create delete tag if missing.
# Use carefully. Delete policies should be approved by governance/legal.
if (-not (Get-RetentionPolicyTag -Identity $DeleteTagName -ErrorAction SilentlyContinue)) {
  New-RetentionPolicyTag `
    -Name $DeleteTagName `
    -Type DeletedItems `
    -RetentionEnabled $true `
    -AgeLimitForRetention 2555 `
    -RetentionAction PermanentlyDelete
}

# Create retention policy if missing.
if (-not (Get-RetentionPolicy -Identity $RetentionPolicyName -ErrorAction SilentlyContinue)) {
  New-RetentionPolicy `
    -Name $RetentionPolicyName `
    -RetentionPolicyTagLinks $ArchiveTagName,$DeleteTagName
}
else {
  Set-RetentionPolicy `
    -Identity $RetentionPolicyName `
    -RetentionPolicyTagLinks $ArchiveTagName,$DeleteTagName
}

# Assign policy to mailbox.
Set-Mailbox -Identity $TargetMailbox `
  -RetentionPolicy $RetentionPolicyName

# Trigger Managed Folder Assistant.
Start-ManagedFolderAssistant -Identity $TargetMailbox

# Validate after state.
Get-RetentionPolicyTag -Identity $ArchiveTagName |
  Format-List Name,Type,RetentionEnabled,AgeLimitForRetention,RetentionAction |
  Out-File "$EvidencePath\archive-tag-after.txt"

Get-RetentionPolicyTag -Identity $DeleteTagName |
  Format-List Name,Type,RetentionEnabled,AgeLimitForRetention,RetentionAction |
  Out-File "$EvidencePath\delete-tag-after.txt"

Get-RetentionPolicy -Identity $RetentionPolicyName |
  Format-List Name,RetentionPolicyTagLinks |
  Out-File "$EvidencePath\retention-policy-after.txt"

Get-Mailbox -Identity $TargetMailbox |
  Format-List DisplayName,PrimarySmtpAddress,RetentionPolicy,ArchiveStatus |
  Out-File "$EvidencePath\mailbox-retention-after.txt"

Stop-Transcript
```

# Configure_Retention_Archive_Litigation_Hold_And_eDiscovery_Handoff_Purview_Retention_Handoff_Skeleton

```powershell
# Run on the admin workstation.
# Purpose: prepare Purview retention policy handoff and optionally create a scoped Exchange mailbox retention policy.
# Purview retention is compliance lifecycle management across Microsoft 365 locations.
# Use the Purview portal for final review when legal/compliance owns policy approval.

$AdminUpn = "admin@contoso.com"
$TargetMailbox = "alex.wilber@contoso.com"
$PolicyName = "LAB - Exchange Retain 7 Years"
$RuleName = "LAB - Exchange Retain 7 Years Rule"
$EvidenceRoot = "C:\M365-Exchange-Baseline"
$RunStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$EvidencePath = Join-Path $EvidenceRoot "07-Purview-Retention-Handoff-$RunStamp"

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\purview-retention-handoff-transcript.txt"

# Connect to Purview compliance PowerShell if required.
# This uses ExchangeOnlineManagement module.
Connect-IPPSSession -UserPrincipalName $AdminUpn

# Capture command metadata because retention compliance cmdlet parameters can vary by service update.
Get-Command New-RetentionCompliancePolicy |
  Out-File "$EvidencePath\new-retention-compliance-policy-command.txt"

Get-Command New-RetentionComplianceRule |
  Out-File "$EvidencePath\new-retention-compliance-rule-command.txt"

Get-Command Get-RetentionCompliancePolicy |
  Out-File "$EvidencePath\get-retention-compliance-policy-command.txt"

Get-Command Get-RetentionComplianceRule |
  Out-File "$EvidencePath\get-retention-compliance-rule-command.txt"

# Capture existing Purview retention compliance policies if cmdlets are available.
Get-RetentionCompliancePolicy -ErrorAction SilentlyContinue |
  Select-Object Name,Enabled,Mode,ExchangeLocation,SharePointLocation,OneDriveLocation,PublicFolderLocation,ModernGroupLocation |
  Export-Csv "$EvidencePath\purview-retention-policies-before.csv" -NoTypeInformation

Get-RetentionComplianceRule -ErrorAction SilentlyContinue |
  Select-Object Name,Policy,RetentionDuration,RetentionComplianceAction,ExpirationDateOption |
  Export-Csv "$EvidencePath\purview-retention-rules-before.csv" -NoTypeInformation

# Optional creation example:
# Static Exchange mailbox retention policy scoped to one mailbox.
# Validate in Purview portal before production use.
if (-not (Get-RetentionCompliancePolicy -Identity $PolicyName -ErrorAction SilentlyContinue)) {
  New-RetentionCompliancePolicy `
    -Name $PolicyName `
    -ExchangeLocation $TargetMailbox `
    -Enabled $true
}

if (-not (Get-RetentionComplianceRule -Identity $RuleName -ErrorAction SilentlyContinue)) {
  New-RetentionComplianceRule `
    -Name $RuleName `
    -Policy $PolicyName `
    -RetentionDuration 2555 `
    -RetentionComplianceAction Keep `
    -ExpirationDateOption ModificationAgeInDays
}

# Validate after state.
Get-RetentionCompliancePolicy -Identity $PolicyName -ErrorAction SilentlyContinue |
  Format-List * |
  Out-File "$EvidencePath\purview-retention-policy-after.txt"

Get-RetentionComplianceRule -Identity $RuleName -ErrorAction SilentlyContinue |
  Format-List * |
  Out-File "$EvidencePath\purview-retention-rule-after.txt"

# Handoff note for portal validation.
@"
Purview portal validation path:
1. Open Microsoft Purview portal.
2. Go to Solutions > Data Lifecycle Management > Policies > Retention policies.
3. Confirm policy name: $PolicyName
4. Confirm location: Exchange mailboxes
5. Confirm scope includes: $TargetMailbox
6. Confirm retention duration and action.
7. Confirm policy status and distribution state.
"@ | Out-File "$EvidencePath\purview-portal-validation-steps.txt"

Stop-Transcript
```

# Configure_Retention_Archive_Litigation_Hold_And_eDiscovery_Handoff_Litigation_Hold_Skeleton

```powershell
# Run on the admin workstation.
# Purpose: place a mailbox on litigation hold and validate preservation state.
# Do not enable or remove litigation hold without legal/compliance approval.

$TargetMailbox = "alex.wilber@contoso.com"
$HoldOwner = "legal@contoso.com"
$HoldDurationDays = 2555
$HoldComment = "LAB-EXO-007 legal hold for LAB-CASE-001. Approved by legal."
$EvidenceRoot = "C:\M365-Exchange-Baseline"
$RunStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$EvidencePath = Join-Path $EvidenceRoot "07-Litigation-Hold-$RunStamp"

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\litigation-hold-transcript.txt"

# Capture before state.
Get-Mailbox -Identity $TargetMailbox |
  Format-List DisplayName,PrimarySmtpAddress,LitigationHoldEnabled,LitigationHoldDuration,LitigationHoldDate,LitigationHoldOwner,LitigationHoldDuration,LitigationHoldEnabled,LitigationHoldDate,LitigationHoldOwner,LitigationHoldDuration,InPlaceHolds,DelayHoldApplied,DelayReleaseHoldApplied,SingleItemRecoveryEnabled,RetainDeletedItemsFor |
  Out-File "$EvidencePath\litigation-hold-before.txt"

Get-MailboxFolderStatistics -Identity $TargetMailbox -FolderScope RecoverableItems |
  Select-Object Name,FolderPath,ItemsInFolder,FolderAndSubfolderSize |
  Export-Csv "$EvidencePath\recoverable-items-before-hold.csv" -NoTypeInformation

# Enable litigation hold.
Set-Mailbox -Identity $TargetMailbox `
  -LitigationHoldEnabled $true `
  -LitigationHoldDuration $HoldDurationDays `
  -LitigationHoldOwner $HoldOwner `
  -LitigationHoldDate (Get-Date) `
  -LitigationHoldComment $HoldComment

# Validate after state.
Get-Mailbox -Identity $TargetMailbox |
  Format-List DisplayName,PrimarySmtpAddress,LitigationHoldEnabled,LitigationHoldDuration,LitigationHoldDate,LitigationHoldOwner,LitigationHoldComment,InPlaceHolds,DelayHoldApplied,DelayReleaseHoldApplied,SingleItemRecoveryEnabled,RetainDeletedItemsFor |
  Out-File "$EvidencePath\litigation-hold-after.txt"

Get-MailboxFolderStatistics -Identity $TargetMailbox -FolderScope RecoverableItems |
  Select-Object Name,FolderPath,ItemsInFolder,FolderAndSubfolderSize |
  Export-Csv "$EvidencePath\recoverable-items-after-hold.csv" -NoTypeInformation

Stop-Transcript
```

# Configure_Retention_Archive_Litigation_Hold_And_eDiscovery_Handoff_Recoverable_Items_Skeleton

```powershell
# Run on the admin workstation.
# Purpose: inspect Recoverable Items folders for hold and retention validation.

$TargetMailbox = "alex.wilber@contoso.com"
$EvidenceRoot = "C:\M365-Exchange-Baseline"
$RunStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$EvidencePath = Join-Path $EvidenceRoot "07-Recoverable-Items-$RunStamp"

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\recoverable-items-transcript.txt"

# Mailbox compliance state.
Get-Mailbox -Identity $TargetMailbox |
  Format-List DisplayName,PrimarySmtpAddress,LitigationHoldEnabled,LitigationHoldDuration,InPlaceHolds,DelayHoldApplied,DelayReleaseHoldApplied,SingleItemRecoveryEnabled,RetainDeletedItemsFor |
  Out-File "$EvidencePath\mailbox-hold-state.txt"

# Recoverable Items folder statistics.
Get-MailboxFolderStatistics -Identity $TargetMailbox -FolderScope RecoverableItems |
  Select-Object Name,FolderPath,ItemsInFolder,FolderAndSubfolderSize,OldestItemReceivedDate,NewestItemReceivedDate |
  Export-Csv "$EvidencePath\recoverable-items-folder-statistics.csv" -NoTypeInformation

# Mailbox statistics.
Get-MailboxStatistics -Identity $TargetMailbox |
  Format-List DisplayName,ItemCount,TotalItemSize,DeletedItemCount,TotalDeletedItemSize |
  Out-File "$EvidencePath\mailbox-statistics.txt"

# Archive statistics if present.
Get-MailboxStatistics -Identity $TargetMailbox -Archive -ErrorAction SilentlyContinue |
  Format-List DisplayName,ItemCount,TotalItemSize,DeletedItemCount,TotalDeletedItemSize |
  Out-File "$EvidencePath\archive-statistics.txt"

Stop-Transcript
```

# Configure_Retention_Archive_Litigation_Hold_And_eDiscovery_Handoff_eDiscovery_Handoff_Package_Skeleton

```powershell
# Run on the admin workstation.
# Purpose: create a technical handoff package for legal/compliance eDiscovery managers.
# This does not replace the legal team's Purview eDiscovery case workflow.
# Use the new Purview eDiscovery experience unless your tenant is in an exception environment.

$TargetMailbox = "alex.wilber@contoso.com"
$SharedMailbox = "helpdesk@contoso.com"
$CaseName = "LAB-CASE-001"
$LegalContact = "legal@contoso.com"
$EDiscoveryManager = "ediscovery.manager@contoso.com"
$DateRangeStart = "2026-01-01"
$DateRangeEnd = "2026-06-15"
$Keywords = '"project phoenix" OR "confidential acquisition"'
$EvidenceRoot = "C:\M365-Exchange-Baseline"
$RunStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$EvidencePath = Join-Path $EvidenceRoot "07-eDiscovery-Handoff-$CaseName-$RunStamp"

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\ediscovery-handoff-transcript.txt"

# Case handoff summary.
@"
Case Name: $CaseName
Legal Contact: $LegalContact
eDiscovery Manager: $EDiscoveryManager
Primary Custodian Mailbox: $TargetMailbox
Additional Mailbox: $SharedMailbox
Date Range Start: $DateRangeStart
Date Range End: $DateRangeEnd
Keyword Query Draft: $Keywords

Recommended Purview path:
1. Open Microsoft Purview portal.
2. Go to eDiscovery.
3. Create or open case: $CaseName
4. Add case members.
5. Add custodians and data sources.
6. Place case hold if approved.
7. Create search query using the mailbox scope, date range, and keyword query.
8. Review search statistics.
9. Export only after legal approval.
"@ | Out-File "$EvidencePath\case-handoff-summary.txt"

# Mailbox identity evidence.
Get-EXOMailbox -Identity $TargetMailbox |
  Format-List DisplayName,UserPrincipalName,PrimarySmtpAddress,ExchangeGuid,ExternalDirectoryObjectId,RecipientTypeDetails,ArchiveStatus |
  Out-File "$EvidencePath\custodian-mailbox-identity.txt"

Get-EXOMailbox -Identity $SharedMailbox -ErrorAction SilentlyContinue |
  Format-List DisplayName,PrimarySmtpAddress,ExchangeGuid,ExternalDirectoryObjectId,RecipientTypeDetails,ArchiveStatus |
  Out-File "$EvidencePath\shared-mailbox-identity.txt"

# Hold and retention state.
Get-Mailbox -Identity $TargetMailbox |
  Format-List DisplayName,PrimarySmtpAddress,RetentionPolicy,ArchiveStatus,ArchiveGuid,LitigationHoldEnabled,LitigationHoldDuration,LitigationHoldDate,LitigationHoldOwner,LitigationHoldComment,InPlaceHolds,DelayHoldApplied,DelayReleaseHoldApplied,SingleItemRecoveryEnabled,RetainDeletedItemsFor |
  Out-File "$EvidencePath\custodian-hold-retention-state.txt"

Get-Mailbox -Identity $SharedMailbox -ErrorAction SilentlyContinue |
  Format-List DisplayName,PrimarySmtpAddress,RetentionPolicy,ArchiveStatus,ArchiveGuid,LitigationHoldEnabled,LitigationHoldDuration,LitigationHoldDate,LitigationHoldOwner,LitigationHoldComment,InPlaceHolds,DelayHoldApplied,DelayReleaseHoldApplied,SingleItemRecoveryEnabled,RetainDeletedItemsFor |
  Out-File "$EvidencePath\shared-mailbox-hold-retention-state.txt"

# Mailbox and Recoverable Items statistics.
Get-MailboxStatistics -Identity $TargetMailbox |
  Format-List DisplayName,ItemCount,TotalItemSize,DeletedItemCount,TotalDeletedItemSize,LastLogonTime |
  Out-File "$EvidencePath\custodian-mailbox-statistics.txt"

Get-MailboxStatistics -Identity $TargetMailbox -Archive -ErrorAction SilentlyContinue |
  Format-List DisplayName,ItemCount,TotalItemSize,DeletedItemCount,TotalDeletedItemSize |
  Out-File "$EvidencePath\custodian-archive-statistics.txt"

Get-MailboxFolderStatistics -Identity $TargetMailbox -FolderScope RecoverableItems |
  Select-Object Name,FolderPath,ItemsInFolder,FolderAndSubfolderSize |
  Export-Csv "$EvidencePath\custodian-recoverable-items.csv" -NoTypeInformation

# Permission and audit context.
Get-MailboxPermission -Identity $TargetMailbox |
  Where-Object {$_.IsInherited -eq $false} |
  Select-Object Identity,User,AccessRights,Deny,IsInherited |
  Export-Csv "$EvidencePath\custodian-mailbox-permissions.csv" -NoTypeInformation

Get-RecipientPermission -Identity $TargetMailbox |
  Select-Object Identity,Trustee,AccessRights,IsInherited,Deny |
  Export-Csv "$EvidencePath\custodian-send-as-permissions.csv" -NoTypeInformation

Get-Mailbox -Identity $TargetMailbox |
  Format-List AuditEnabled,AuditAdmin,AuditDelegate,AuditOwner |
  Out-File "$EvidencePath\custodian-mailbox-audit-settings.txt"

# Optional unified audit search sample for the custodian.
Search-UnifiedAuditLog `
  -StartDate ([datetime]$DateRangeStart) `
  -EndDate ([datetime]$DateRangeEnd) `
  -UserIds $TargetMailbox `
  -ResultSize 100 `
  -ErrorAction SilentlyContinue |
  Select-Object CreationDate,UserIds,Operations,AuditData |
  Export-Csv "$EvidencePath\custodian-unified-audit-sample.csv" -NoTypeInformation

Stop-Transcript
```

# Configure_Retention_Archive_Litigation_Hold_And_eDiscovery_Handoff_Bulk_Inventory_Skeleton

```powershell
# Run on the admin workstation.
# Purpose: export tenant-wide or scoped archive, retention, and litigation hold inventory.

$EvidenceRoot = "C:\M365-Exchange-Baseline"
$RunStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$EvidencePath = Join-Path $EvidenceRoot "07-Archive-Retention-Hold-Inventory-$RunStamp"

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\archive-retention-hold-inventory-transcript.txt"

# Scope sample. Adjust ResultSize for tenant size and permission scope.
$Mailboxes = Get-EXOMailbox -ResultSize 500

# Mailbox compliance summary.
$Mailboxes |
  ForEach-Object {
    Get-Mailbox -Identity $_.PrimarySmtpAddress |
      Select-Object DisplayName,PrimarySmtpAddress,RecipientTypeDetails,ArchiveStatus,ArchiveGuid,AutoExpandingArchiveEnabled,RetentionPolicy,LitigationHoldEnabled,LitigationHoldDuration,LitigationHoldDate,LitigationHoldOwner,InPlaceHolds,DelayHoldApplied,DelayReleaseHoldApplied,SingleItemRecoveryEnabled,RetainDeletedItemsFor
  } |
  Export-Csv "$EvidencePath\mailbox-archive-retention-hold-summary.csv" -NoTypeInformation

# Mailboxes with litigation hold.
$Mailboxes |
  ForEach-Object {
    Get-Mailbox -Identity $_.PrimarySmtpAddress |
      Where-Object {$_.LitigationHoldEnabled -eq $true} |
      Select-Object DisplayName,PrimarySmtpAddress,LitigationHoldEnabled,LitigationHoldDuration,LitigationHoldDate,LitigationHoldOwner,LitigationHoldComment
  } |
  Export-Csv "$EvidencePath\mailboxes-on-litigation-hold.csv" -NoTypeInformation

# Mailboxes with archive enabled.
$Mailboxes |
  ForEach-Object {
    Get-Mailbox -Identity $_.PrimarySmtpAddress |
      Where-Object {$_.ArchiveStatus -eq "Active"} |
      Select-Object DisplayName,PrimarySmtpAddress,ArchiveStatus,ArchiveGuid,ArchiveName,AutoExpandingArchiveEnabled
  } |
  Export-Csv "$EvidencePath\mailboxes-with-archive.csv" -NoTypeInformation

# Mailboxes without archive.
$Mailboxes |
  ForEach-Object {
    Get-Mailbox -Identity $_.PrimarySmtpAddress |
      Where-Object {$_.ArchiveStatus -ne "Active"} |
      Select-Object DisplayName,PrimarySmtpAddress,ArchiveStatus,RetentionPolicy,LitigationHoldEnabled
  } |
  Export-Csv "$EvidencePath\mailboxes-without-archive.csv" -NoTypeInformation

# Exchange MRM policies and tags.
Get-RetentionPolicy |
  Select-Object Name,RetentionPolicyTagLinks,IsDefault |
  Export-Csv "$EvidencePath\exchange-retention-policies.csv" -NoTypeInformation

Get-RetentionPolicyTag |
  Select-Object Name,Type,RetentionEnabled,AgeLimitForRetention,RetentionAction |
  Export-Csv "$EvidencePath\exchange-retention-policy-tags.csv" -NoTypeInformation

# Archive statistics for archive-enabled mailboxes.
$ArchiveEnabledMailboxes = $Mailboxes |
  ForEach-Object { Get-Mailbox -Identity $_.PrimarySmtpAddress } |
  Where-Object {$_.ArchiveStatus -eq "Active"}

$ArchiveStats = foreach ($Mailbox in $ArchiveEnabledMailboxes) {
  Get-MailboxStatistics -Identity $Mailbox.PrimarySmtpAddress -Archive -ErrorAction SilentlyContinue |
    Select-Object @{Name="Mailbox";Expression={$Mailbox.PrimarySmtpAddress}},DisplayName,ItemCount,TotalItemSize,DeletedItemCount,TotalDeletedItemSize
}

$ArchiveStats |
  Export-Csv "$EvidencePath\archive-mailbox-statistics.csv" -NoTypeInformation

Stop-Transcript
```

# Configure_Retention_Archive_Litigation_Hold_And_eDiscovery_Handoff_Validation_Skeleton

```powershell
# Run on the admin workstation.
# Purpose: validate archive, Exchange MRM retention, Purview retention handoff, litigation hold, and Recoverable Items state.

$TargetMailbox = "alex.wilber@contoso.com"
$RetentionPolicyName = "LAB - Exchange MRM 7 Year Archive"
$PurviewPolicyName = "LAB - Exchange Retain 7 Years"
$EvidenceRoot = "C:\M365-Exchange-Baseline"
$RunStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$EvidencePath = Join-Path $EvidenceRoot "07-Retention-Archive-Hold-Validation-$RunStamp"

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\retention-archive-hold-validation-transcript.txt"

# Validate mailbox state.
Get-Mailbox -Identity $TargetMailbox |
  Format-List DisplayName,PrimarySmtpAddress,ArchiveStatus,ArchiveGuid,ArchiveName,AutoExpandingArchiveEnabled,RetentionPolicy,LitigationHoldEnabled,LitigationHoldDuration,LitigationHoldDate,LitigationHoldOwner,LitigationHoldComment,InPlaceHolds,DelayHoldApplied,DelayReleaseHoldApplied,SingleItemRecoveryEnabled,RetainDeletedItemsFor |
  Out-File "$EvidencePath\validate-mailbox-compliance-state.txt"

# Validate archive statistics.
Get-MailboxStatistics -Identity $TargetMailbox -Archive -ErrorAction SilentlyContinue |
  Format-List DisplayName,ItemCount,TotalItemSize,DeletedItemCount,TotalDeletedItemSize |
  Out-File "$EvidencePath\validate-archive-statistics.txt"

# Validate MRM retention policy.
Get-RetentionPolicy -Identity $RetentionPolicyName -ErrorAction SilentlyContinue |
  Format-List Name,RetentionPolicyTagLinks |
  Out-File "$EvidencePath\validate-exchange-mrm-policy.txt"

Get-Mailbox -Identity $TargetMailbox |
  Format-List DisplayName,PrimarySmtpAddress,RetentionPolicy |
  Out-File "$EvidencePath\validate-mailbox-retention-policy-assignment.txt"

# Validate Recoverable Items.
Get-MailboxFolderStatistics -Identity $TargetMailbox -FolderScope RecoverableItems |
  Select-Object Name,FolderPath,ItemsInFolder,FolderAndSubfolderSize |
  Export-Csv "$EvidencePath\validate-recoverable-items.csv" -NoTypeInformation

# Validate Purview retention policy if connected to compliance PowerShell.
Get-RetentionCompliancePolicy -Identity $PurviewPolicyName -ErrorAction SilentlyContinue |
  Format-List * |
  Out-File "$EvidencePath\validate-purview-retention-policy.txt"

Get-RetentionComplianceRule -ErrorAction SilentlyContinue |
  Where-Object {$_.Policy -like "*$PurviewPolicyName*"} |
  Format-List * |
  Out-File "$EvidencePath\validate-purview-retention-rule.txt"

Stop-Transcript
```

# Configure_Retention_Archive_Litigation_Hold_And_eDiscovery_Handoff_Verification_Commands

| Check | Device | PowerShell / Command | Healthy Output |
|---|---|---|---|
| Exchange Online session active | Admin Workstation | `Get-ConnectionInformation` | Active session exists |
| Target mailbox exists | Admin Workstation | `Get-EXOMailbox -Identity "<mailbox>"` | Mailbox returns |
| Archive status visible | Admin Workstation | `Get-Mailbox -Identity "<mailbox>" \| Format-List ArchiveStatus,ArchiveGuid,ArchiveName` | Archive state is visible |
| Archive statistics visible | Admin Workstation | `Get-MailboxStatistics -Identity "<mailbox>" -Archive` | Archive stats return if archive exists |
| MRM tags visible | Admin Workstation | `Get-RetentionPolicyTag` | Retention tags return |
| MRM policies visible | Admin Workstation | `Get-RetentionPolicy` | Retention policies return |
| Mailbox MRM assignment visible | Admin Workstation | `Get-Mailbox -Identity "<mailbox>" \| Format-List RetentionPolicy` | Assigned retention policy returns |
| Managed Folder Assistant triggered | Admin Workstation | `Start-ManagedFolderAssistant -Identity "<mailbox>"` | Command completes without error |
| Litigation hold visible | Admin Workstation | `Get-Mailbox -Identity "<mailbox>" \| Format-List LitigationHoldEnabled,LitigationHoldDuration,LitigationHoldDate,LitigationHoldOwner` | Hold state matches design |
| Recoverable Items visible | Admin Workstation | `Get-MailboxFolderStatistics -Identity "<mailbox>" -FolderScope RecoverableItems` | Recoverable Items folders return |
| Purview retention policy visible | Admin Workstation | `Get-RetentionCompliancePolicy -Identity "<policy>"` | Policy returns if created through compliance PowerShell |
| Purview retention rule visible | Admin Workstation | `Get-RetentionComplianceRule` | Rule returns if created through compliance PowerShell |
| Purview portal opens | Browser | Microsoft Purview portal > Data Lifecycle Management | Retention policy area opens |
| Purview eDiscovery opens | Browser | Microsoft Purview portal > eDiscovery | eDiscovery area opens |
| eDiscovery permissions assigned | Browser / Purview | eDiscovery role group membership | eDiscovery manager can access cases |
| eDiscovery handoff package exists | Admin Workstation | `Get-ChildItem C:\M365-Exchange-Baseline -Recurse` | Case handoff evidence files exist |
| Audit sample available | Admin Workstation | `Search-UnifiedAuditLog` | Audit results return if permissions and activity exist |

# Configure_Retention_Archive_Litigation_Hold_And_eDiscovery_Handoff_Rollback

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm rollback approval | Operator | Written legal/compliance approval | Hold and retention rollback is authorized |
| 2 | Export current state before rollback | Admin Workstation | Run bulk inventory skeleton | Pre-rollback evidence is saved |
| 3 | Remove mailbox MRM policy assignment if lab-only | Admin Workstation | `Set-Mailbox -Identity "<mailbox>" -RetentionPolicy $null` | Mailbox no longer has lab MRM policy |
| 4 | Remove lab MRM retention policy if unused | Admin Workstation | `Remove-RetentionPolicy -Identity "<policy>" -Confirm:$false` | Lab MRM policy is removed |
| 5 | Remove lab MRM tags if unused | Admin Workstation | `Remove-RetentionPolicyTag -Identity "<tag>" -Confirm:$false` | Lab tags are removed |
| 6 | Remove Purview retention policy only with approval | Admin Workstation / Portal | `Remove-RetentionCompliancePolicy` or Purview portal | Lab Purview retention policy is removed |
| 7 | Remove litigation hold only with legal approval | Admin Workstation | `Set-Mailbox -Identity "<mailbox>" -LitigationHoldEnabled $false` | Litigation hold is disabled |
| 8 | Disable archive only if lab-created and safe | Admin Workstation | `Disable-Mailbox -Identity "<mailbox>" -Archive` | Archive mailbox is disabled if allowed |
| 9 | Validate delay hold state | Admin Workstation | `Get-Mailbox -Identity "<mailbox>" \| Format-List DelayHoldApplied,DelayReleaseHoldApplied` | Delay hold state is documented |
| 10 | Export post-rollback state | Admin Workstation | Run validation skeleton | Post-rollback evidence is saved |
| 11 | Notify legal/compliance owner | Operator | Send evidence package | Compliance owner has rollback record |

# Configure_Retention_Archive_Litigation_Hold_And_eDiscovery_Handoff_Rollback_Helper

```powershell
# Run on the admin workstation.
# Purpose: rollback lab-created archive, MRM, Purview retention, and litigation hold settings only.
# Do not run without legal/compliance approval.

$TargetMailbox = "alex.wilber@contoso.com"
$ArchiveTagName = "LAB - Move to Archive after 2 Years"
$DeleteTagName = "LAB - Delete after 7 Years"
$RetentionPolicyName = "LAB - Exchange MRM 7 Year Archive"
$PurviewPolicyName = "LAB - Exchange Retain 7 Years"
$PurviewRuleName = "LAB - Exchange Retain 7 Years Rule"
$EvidenceRoot = "C:\M365-Exchange-Baseline"
$RunStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$EvidencePath = Join-Path $EvidenceRoot "07-Retention-Archive-Hold-Rollback-$RunStamp"

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\rollback-transcript.txt"

# Capture before rollback.
Get-Mailbox -Identity $TargetMailbox |
  Format-List DisplayName,PrimarySmtpAddress,ArchiveStatus,ArchiveGuid,RetentionPolicy,LitigationHoldEnabled,LitigationHoldDuration,LitigationHoldDate,LitigationHoldOwner,LitigationHoldComment,InPlaceHolds,DelayHoldApplied,DelayReleaseHoldApplied |
  Out-File "$EvidencePath\mailbox-state-before-rollback.txt"

Get-RetentionPolicy -Identity $RetentionPolicyName -ErrorAction SilentlyContinue |
  Format-List * |
  Out-File "$EvidencePath\exchange-mrm-policy-before-rollback.txt"

Get-RetentionPolicyTag -Identity $ArchiveTagName -ErrorAction SilentlyContinue |
  Format-List * |
  Out-File "$EvidencePath\archive-tag-before-rollback.txt"

Get-RetentionPolicyTag -Identity $DeleteTagName -ErrorAction SilentlyContinue |
  Format-List * |
  Out-File "$EvidencePath\delete-tag-before-rollback.txt"

# Remove Exchange MRM assignment from target mailbox if lab policy is assigned.
$Mailbox = Get-Mailbox -Identity $TargetMailbox

if ($Mailbox.RetentionPolicy -eq $RetentionPolicyName) {
  Set-Mailbox -Identity $TargetMailbox -RetentionPolicy $null
}

# Remove lab Exchange MRM policy if it exists.
Remove-RetentionPolicy -Identity $RetentionPolicyName -Confirm:$false -ErrorAction SilentlyContinue

# Remove lab Exchange MRM tags if they exist and are no longer linked.
Remove-RetentionPolicyTag -Identity $ArchiveTagName -Confirm:$false -ErrorAction SilentlyContinue
Remove-RetentionPolicyTag -Identity $DeleteTagName -Confirm:$false -ErrorAction SilentlyContinue

# Remove Purview retention rule and policy only if approved.
# Requires compliance PowerShell connection.
Remove-RetentionComplianceRule -Identity $PurviewRuleName -Confirm:$false -ErrorAction SilentlyContinue
Remove-RetentionCompliancePolicy -Identity $PurviewPolicyName -Confirm:$false -ErrorAction SilentlyContinue

# Disable litigation hold only with explicit legal/compliance approval.
# Uncomment only after approval.
# Set-Mailbox -Identity $TargetMailbox -LitigationHoldEnabled $false

# Disable archive only if this was a lab archive and removal is approved.
# This can disconnect archive access. Do not run casually.
# Disable-Mailbox -Identity $TargetMailbox -Archive -Confirm:$false

# Capture after rollback.
Get-Mailbox -Identity $TargetMailbox |
  Format-List DisplayName,PrimarySmtpAddress,ArchiveStatus,ArchiveGuid,RetentionPolicy,LitigationHoldEnabled,LitigationHoldDuration,LitigationHoldDate,LitigationHoldOwner,LitigationHoldComment,InPlaceHolds,DelayHoldApplied,DelayReleaseHoldApplied |
  Out-File "$EvidencePath\mailbox-state-after-rollback.txt"

Get-RetentionPolicy |
  Select-Object Name,RetentionPolicyTagLinks |
  Export-Csv "$EvidencePath\exchange-mrm-policies-after-rollback.csv" -NoTypeInformation

Get-RetentionPolicyTag |
  Select-Object Name,Type,RetentionEnabled,AgeLimitForRetention,RetentionAction |
  Export-Csv "$EvidencePath\exchange-mrm-tags-after-rollback.csv" -NoTypeInformation

Stop-Transcript
```

# Configure_Retention_Archive_Litigation_Hold_And_eDiscovery_Handoff_Failure_Checks

| Symptom | Likely Cause | Check Command | Fix |
|---|---|---|---|
| Archive enable fails | License missing, mailbox type unsupported, or archive already enabled | `Get-Mailbox -Identity "<mailbox>" \| Format-List ArchiveStatus,RecipientTypeDetails` | Assign correct license or validate mailbox type |
| Archive does not appear immediately | Provisioning delay | `Get-Mailbox -Identity "<mailbox>" \| Format-List ArchiveStatus,ArchiveGuid` | Wait and recheck |
| Archive statistics fail | Archive not active or not provisioned yet | `Get-Mailbox -Identity "<mailbox>" \| Format-List ArchiveStatus` | Enable archive and wait for provisioning |
| MRM retention does not process immediately | Managed Folder Assistant has not processed mailbox yet | `Start-ManagedFolderAssistant -Identity "<mailbox>"` | Trigger assistant and allow processing time |
| Retention tag cannot be removed | Tag linked to a retention policy | `Get-RetentionPolicy \| Format-List Name,RetentionPolicyTagLinks` | Unlink tag from policy first |
| Retention policy cannot be removed | Policy still assigned to mailboxes | `Get-EXOMailbox -ResultSize Unlimited \| Where-Object {$_.RetentionPolicy -eq "<policy>"}` | Remove assignments first |
| Purview retention cmdlets not found | Compliance PowerShell session not connected | `Get-Command Connect-IPPSSession` | Connect with `Connect-IPPSSession` |
| Purview retention policy not applying | Scope, distribution, or policy status issue | Purview portal policy status and `Get-RetentionCompliancePolicy` | Validate location and distribution status |
| Litigation hold command fails | Missing role or license issue | `Get-ManagementRoleAssignment -RoleAssignee "<admin-upn>"` | Assign proper compliance or Exchange role |
| Litigation hold appears enabled but deleted items not visible | Hold preserves in Recoverable Items, not normal folders | `Get-MailboxFolderStatistics -FolderScope RecoverableItems` | Validate Recoverable Items folders |
| Hold removal does not immediately clear all hold indicators | Delay hold remains | `Get-Mailbox -Identity "<mailbox>" \| Format-List DelayHoldApplied,DelayReleaseHoldApplied` | Document and allow service-managed delay behavior |
| Search-UnifiedAuditLog fails | Missing audit role or compliance session issue | Check role group membership | Assign Audit Logs or compliance role as required |
| eDiscovery manager cannot access case | User not in correct Purview role group or not case member | Purview portal eDiscovery permissions | Add user to eDiscovery role group and case |
| Classic eDiscovery path missing | Classic experiences retired | Use Purview eDiscovery | Use new Purview portal eDiscovery workflow |
| Retention and hold conflict seems confusing | Multiple retention settings apply | Review mailbox holds, Purview retention policies, MRM policies | Document all applicable controls and consult precedence guidance |
| User expects archive to delete mail | Archive moves mail, it is not a deletion control by itself | Check retention tag action | Use proper retention or deletion policy |
| Shared mailbox archive fails | License or mailbox type requirement issue | `Get-Mailbox -Identity "<shared>" \| Format-List RecipientTypeDetails,ArchiveStatus` | Validate licensing and shared mailbox requirements |
| Removing archive is blocked or unsafe | Archive contains data or legal hold applies | Check archive stats and hold state | Do not remove until approved |
| Export package incomplete | Missing permissions or failed commands | Review transcript errors | Re-run failed skeleton sections with correct role |

# Configure_Retention_Archive_Litigation_Hold_And_eDiscovery_Handoff_Related_Labs

| Related Lab                                                                                    | Relationship                                                                                    |
| ---------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------- |
| `01_Configure_Exchange_Admin_Center_PowerShell_And_Org_Baseline.md`                            | Provides Exchange admin access, PowerShell connection, RBAC, and baseline evidence path         |
| `02_Manage_User_Mailboxes_Shared_Mailboxes_Contacts_And_Resources.md`                          | Provides mailbox targets for archive, retention, hold, and eDiscovery handoff                   |
| `03_Manage_Distribution_Groups_Dynamic_Groups_Mail_Enabled_Security_Groups_And_M365_Groups.md` | Provides Microsoft 365 group mailbox and group scope context for retention and eDiscovery       |
| `04_Configure_Mail_Flow_Accepted_Domains_Connectors_And_Rules.md`                              | Provides mail flow and message trace context for investigation scope                            |
| `05_Configure_Mailbox_Permissions_Delegation_Forwarding_And_Auditing.md`                       | Provides mailbox audit, access, delegation, and forwarding evidence                             |
| `06_Configure_Exchange_Online_Protection_AntiSpam_AntiMalware_And_Quarantine.md`               | Provides protection, quarantine, and submission evidence that can become investigation material |
| `08_Troubleshoot_Mail_Flow_Mailbox_Access_Permissions_And_Quarantine_Issues.md`                | Uses retention, hold, archive, and eDiscovery evidence during escalated troubleshooting         |