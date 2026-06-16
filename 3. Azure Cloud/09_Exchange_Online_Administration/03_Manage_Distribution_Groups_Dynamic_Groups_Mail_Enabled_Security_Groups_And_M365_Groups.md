# Manage_Distribution_Groups_Dynamic_Groups_Mail_Enabled_Security_Groups_And_M365_Groups

# Manage_Distribution_Groups_Dynamic_Groups_Mail_Enabled_Security_Groups_And_M365_Groups_Index
03_Manage_Distribution_Groups_Dynamic_Groups_Mail_Enabled_Security_Groups_And_M365_Groups.md
Manage_Distribution_Groups_Dynamic_Groups_Mail_Enabled_Security_Groups_And_M365_Groups
Manage_Distribution_Groups_Dynamic_Groups_Mail_Enabled_Security_Groups_And_M365_Groups_Source_Basis
Manage_Distribution_Groups_Dynamic_Groups_Mail_Enabled_Security_Groups_And_M365_Groups_Mental_Model
Manage_Distribution_Groups_Dynamic_Groups_Mail_Enabled_Security_Groups_And_M365_Groups_Planning_Table
Manage_Distribution_Groups_Dynamic_Groups_Mail_Enabled_Security_Groups_And_M365_Groups_Configuration_Checklist
Manage_Distribution_Groups_Dynamic_Groups_Mail_Enabled_Security_Groups_And_M365_Groups_Precheck_Skeleton
Manage_Distribution_Groups_Dynamic_Groups_Mail_Enabled_Security_Groups_And_M365_Groups_Distribution_Group_Skeleton
Manage_Distribution_Groups_Dynamic_Groups_Mail_Enabled_Security_Groups_And_M365_Groups_Dynamic_Distribution_Group_Skeleton
Manage_Distribution_Groups_Dynamic_Groups_Mail_Enabled_Security_Groups_And_M365_Groups_Mail_Enabled_Security_Group_Skeleton
Manage_Distribution_Groups_Dynamic_Groups_Mail_Enabled_Security_Groups_And_M365_Groups_M365_Group_Skeleton
Manage_Distribution_Groups_Dynamic_Groups_Mail_Enabled_Security_Groups_And_M365_Groups_Moderation_And_Delivery_Restrictions_Skeleton
Manage_Distribution_Groups_Dynamic_Groups_Mail_Enabled_Security_Groups_And_M365_Groups_Bulk_Inventory_Skeleton
Manage_Distribution_Groups_Dynamic_Groups_Mail_Enabled_Security_Groups_And_M365_Groups_Validation_Skeleton
Manage_Distribution_Groups_Dynamic_Groups_Mail_Enabled_Security_Groups_And_M365_Groups_Verification_Commands
Manage_Distribution_Groups_Dynamic_Groups_Mail_Enabled_Security_Groups_And_M365_Groups_Rollback
Manage_Distribution_Groups_Dynamic_Groups_Mail_Enabled_Security_Groups_And_M365_Groups_Failure_Checks
Manage_Distribution_Groups_Dynamic_Groups_Mail_Enabled_Security_Groups_And_M365_Groups_Related_Labs

# Manage_Distribution_Groups_Dynamic_Groups_Mail_Enabled_Security_Groups_And_M365_Groups_Source_Basis

| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Exchange admin center groups | Managing distribution lists, mail-enabled security groups, dynamic distribution groups, and Microsoft 365 groups |
| Microsoft Learn | Exchange Online PowerShell | Managing group recipients through ExchangeOnlineManagement |
| Microsoft Learn | New-DistributionGroup / Set-DistributionGroup | Creating and configuring distribution groups and mail-enabled security groups |
| Microsoft Learn | Add-DistributionGroupMember / Remove-DistributionGroupMember | Managing static distribution group and mail-enabled security group membership |
| Microsoft Learn | New-DynamicDistributionGroup / Set-DynamicDistributionGroup | Creating recipient-filtered dynamic distribution groups |
| Microsoft Learn | Get-Recipient -RecipientPreviewFilter | Previewing dynamic distribution group membership |
| Microsoft Learn | New-UnifiedGroup / Set-UnifiedGroup | Creating and configuring Microsoft 365 groups from Exchange Online PowerShell |
| Microsoft Learn | Add-UnifiedGroupLinks / Remove-UnifiedGroupLinks | Managing Microsoft 365 group owners, members, and subscribers |
| Prior workbook | 01_Configure_Exchange_Admin_Center_PowerShell_And_Org_Baseline.md | Provides Exchange Online PowerShell connection and baseline evidence model |
| Prior workbook | 02_Manage_User_Mailboxes_Shared_Mailboxes_Contacts_And_Resources.md | Provides recipient inventory and test mailboxes used for group membership |

# Manage_Distribution_Groups_Dynamic_Groups_Mail_Enabled_Security_Groups_And_M365_Groups_Mental_Model

| Concept | Operational Meaning |
|---|---|
| Distribution group | Mail-enabled group used to send email to multiple recipients |
| Dynamic distribution group | Query-based group where membership is calculated from a recipient filter |
| Mail-enabled security group | Security group that can receive mail and can also be used for access control |
| Microsoft 365 group | Collaboration group backed by mailbox, calendar, SharePoint site, Planner, Teams-ready identity, and membership |
| Static membership | Members are manually added or removed |
| Dynamic membership | Members are selected by filter at message delivery or by Entra group rule, depending on group type |
| Owner | User responsible for group management |
| ManagedBy | Exchange property that stores group owners or managers |
| Members | Recipients that receive mail sent to a static group |
| Moderation | Approval workflow for messages sent to a group |
| Delivery restrictions | Rules controlling who can send to the group |
| RequireSenderAuthenticationEnabled | Controls whether external senders can send to the group |
| HiddenFromAddressListsEnabled | Hides group from address lists |
| HiddenFromExchangeClientsEnabled | Hides Microsoft 365 group from Exchange clients such as Outlook |
| AccessType | Microsoft 365 group public or private membership setting |
| RecipientFilter | OPATH-style filter used by dynamic distribution groups |
| Source of authority | Whether the group is cloud-managed or synced from on-premises AD |
| First rule | Never edit synced group properties in cloud if the property is mastered on-premises |
| Second rule | Distribution groups are for mail delivery, mail-enabled security groups can be used for mail plus permissions, Microsoft 365 groups are collaboration groups |
| Third rule | Preview dynamic distribution group filters before relying on them for mail delivery |

# Manage_Distribution_Groups_Dynamic_Groups_Mail_Enabled_Security_Groups_And_M365_Groups_Planning_Table

| Item | Example | Decision |
|---|---|---|
| Tenant name | `contoso.onmicrosoft.com` | `<tenant-name>` |
| Primary SMTP domain | `contoso.com` | `<primary-smtp-domain>` |
| Admin UPN | `admin@contoso.com` | `<admin-upn>` |
| Evidence root | `C:\M365-Exchange-Baseline` | `<evidence-root>` |
| Group source of authority | Cloud-only / Hybrid synced | `<source-of-authority>` |
| Distribution group naming standard | `DL-Department-Name` | `<dg-standard>` |
| Dynamic distribution group naming standard | `DDG-Department-Name` | `<ddg-standard>` |
| Mail-enabled security group naming standard | `MESG-Access-Name` | `<mesg-standard>` |
| Microsoft 365 group naming standard | `M365-Team-Name` | `<m365-standard>` |
| Default owner | `group.owner@contoso.com` | `<owner-upn>` |
| Test member 1 | `alex.wilber@contoso.com` | `<member-1>` |
| Test member 2 | `adele.vance@contoso.com` | `<member-2>` |
| External sender allowed | No by default | `<yes-no>` |
| Group hidden from address lists | No by default | `<yes-no>` |
| Moderation standard | Optional for broad groups | `<moderation-policy>` |
| Dynamic group filter field | Department / CustomAttribute / Office / State | `<filter-field>` |
| Dynamic group filter value | `Sales` | `<filter-value>` |
| Microsoft 365 group access type | Private | `<public-private>` |
| Change ticket | `LAB-EXO-003` | `<ticket-or-lab-id>` |
| Rollback stance | Remove lab-created groups only | `<rollback-scope>` |

# Manage_Distribution_Groups_Dynamic_Groups_Mail_Enabled_Security_Groups_And_M365_Groups_Configuration_Checklist

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm Exchange Online session | Admin Workstation | `Get-ConnectionInformation` | Active Exchange Online session exists |
| 2 | Confirm accepted domain | Admin Workstation | `Get-AcceptedDomain` | Target SMTP domain exists |
| 3 | Confirm recipient baseline | Admin Workstation | `Get-Recipient -ResultSize 10` | Recipient data returns |
| 4 | Export existing group inventory | Admin Workstation | `Get-DistributionGroup`; `Get-DynamicDistributionGroup`; `Get-UnifiedGroup` | Current groups are captured |
| 5 | Confirm source of authority | Admin Workstation | Check directory sync and group origin | Cloud or synced management path is known |
| 6 | Create static distribution group | Admin Workstation | `New-DistributionGroup -Type Distribution` | Distribution group exists |
| 7 | Add members to distribution group | Admin Workstation | `Add-DistributionGroupMember` | Members are added |
| 8 | Validate distribution group membership | Admin Workstation | `Get-DistributionGroupMember` | Expected members appear |
| 9 | Configure distribution group owner | Admin Workstation | `Set-DistributionGroup -ManagedBy "<owner-upn>"` | Owner is assigned |
| 10 | Configure distribution group delivery restrictions | Admin Workstation | `Set-DistributionGroup -RequireSenderAuthenticationEnabled $true` | External senders are blocked if required |
| 11 | Create dynamic distribution group | Admin Workstation | `New-DynamicDistributionGroup -RecipientFilter "<filter>"` | Dynamic group exists |
| 12 | Preview dynamic distribution group membership | Admin Workstation | `Get-Recipient -RecipientPreviewFilter` | Expected recipients match filter |
| 13 | Create mail-enabled security group | Admin Workstation | `New-DistributionGroup -Type Security` | Mail-enabled security group exists |
| 14 | Add members to mail-enabled security group | Admin Workstation | `Add-DistributionGroupMember` | Members are added |
| 15 | Validate mail-enabled security group | Admin Workstation | `Get-DistributionGroup` | RecipientTypeDetails is MailUniversalSecurityGroup |
| 16 | Create Microsoft 365 group | Admin Workstation | `New-UnifiedGroup` | Microsoft 365 group exists |
| 17 | Add Microsoft 365 group owners | Admin Workstation | `Add-UnifiedGroupLinks -LinkType Owners` | Owners are assigned |
| 18 | Add Microsoft 365 group members | Admin Workstation | `Add-UnifiedGroupLinks -LinkType Members` | Members are assigned |
| 19 | Configure Microsoft 365 group visibility | Admin Workstation | `Set-UnifiedGroup` | Visibility and access type match standard |
| 20 | Configure moderation if required | Admin Workstation | `Set-DistributionGroup -ModerationEnabled $true` | Group moderation is enabled |
| 21 | Export final group inventory | Admin Workstation | Export group and membership data | Final evidence is saved |
| 22 | Compare before and after group counts | Admin Workstation | Compare exported CSV files | Expected group count changes are visible |
| 23 | Validate in Exchange admin center | Browser | Exchange admin center > Recipients > Groups | Groups appear in the correct category |
| 24 | Disconnect Exchange Online session | Admin Workstation | `Disconnect-ExchangeOnline -Confirm:$false` | Session closes cleanly |

# Manage_Distribution_Groups_Dynamic_Groups_Mail_Enabled_Security_Groups_And_M365_Groups_Precheck_Skeleton

```powershell
# Run on the admin workstation.
# Purpose: validate Exchange Online connection, accepted domain, recipient availability, and baseline group state.

$AdminUpn = "admin@contoso.com"
$PrimaryDomain = "contoso.com"
$EvidenceRoot = "C:\M365-Exchange-Baseline"
$RunStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$EvidencePath = Join-Path $EvidenceRoot "03-Groups-Precheck-$RunStamp"

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\groups-precheck-transcript.txt"

Import-Module ExchangeOnlineManagement

try {
  $Connection = Get-ConnectionInformation -ErrorAction Stop
}
catch {
  Connect-ExchangeOnline -UserPrincipalName $AdminUpn -ShowBanner:$false
}

Get-ConnectionInformation |
  Tee-Object "$EvidencePath\connection-information.txt"

Get-AcceptedDomain |
  Select-Object Name,DomainName,DomainType,Default |
  Export-Csv "$EvidencePath\accepted-domains.csv" -NoTypeInformation

Get-AcceptedDomain |
  Where-Object {$_.DomainName -eq $PrimaryDomain} |
  Format-List * |
  Out-File "$EvidencePath\target-domain-check.txt"

# Recipient baseline.
Get-Recipient -ResultSize Unlimited |
  Group-Object RecipientTypeDetails |
  Sort-Object Name |
  Select-Object Name,Count |
  Export-Csv "$EvidencePath\recipient-type-counts-before.csv" -NoTypeInformation

# Existing static distribution groups and mail-enabled security groups.
Get-DistributionGroup -ResultSize Unlimited |
  Select-Object DisplayName,Name,PrimarySmtpAddress,RecipientTypeDetails,ManagedBy,RequireSenderAuthenticationEnabled,HiddenFromAddressListsEnabled |
  Export-Csv "$EvidencePath\distribution-groups-before.csv" -NoTypeInformation

# Existing dynamic distribution groups.
Get-DynamicDistributionGroup -ResultSize Unlimited |
  Select-Object DisplayName,Name,PrimarySmtpAddress,RecipientFilter,RecipientTypeDetails,HiddenFromAddressListsEnabled |
  Export-Csv "$EvidencePath\dynamic-distribution-groups-before.csv" -NoTypeInformation

# Existing Microsoft 365 groups.
Get-UnifiedGroup -ResultSize Unlimited |
  Select-Object DisplayName,PrimarySmtpAddress,AccessType,ManagedBy,HiddenFromAddressListsEnabled,HiddenFromExchangeClientsEnabled |
  Export-Csv "$EvidencePath\m365-groups-before.csv" -NoTypeInformation

# Sample mailboxes for test membership.
Get-EXOMailbox -ResultSize 50 |
  Select-Object DisplayName,UserPrincipalName,PrimarySmtpAddress,RecipientTypeDetails,Department,Office |
  Export-Csv "$EvidencePath\mailbox-sample-for-group-membership.csv" -NoTypeInformation

Stop-Transcript
```

# Manage_Distribution_Groups_Dynamic_Groups_Mail_Enabled_Security_Groups_And_M365_Groups_Distribution_Group_Skeleton

```powershell
# Run on the admin workstation.
# Purpose: create, configure, populate, and validate a static distribution group.

$EvidenceRoot = "C:\M365-Exchange-Baseline"
$RunStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$EvidencePath = Join-Path $EvidenceRoot "03-Distribution-Group-$RunStamp"

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\distribution-group-transcript.txt"

# Variables.
$GroupName = "DL Help Desk Notifications"
$GroupAlias = "dl-helpdesk-notifications"
$GroupSmtp = "dl-helpdesk-notifications@contoso.com"
$GroupDisplayName = "DL Help Desk Notifications"
$OwnerUpn = "alex.wilber@contoso.com"
$Members = @(
  "alex.wilber@contoso.com",
  "adele.vance@contoso.com"
)

# Precheck for existing group or address conflict.
Get-Recipient -Identity $GroupSmtp -ErrorAction SilentlyContinue |
  Format-List Name,PrimarySmtpAddress,RecipientTypeDetails |
  Out-File "$EvidencePath\distribution-group-existing-check.txt"

# Create distribution group if it does not exist.
if (-not (Get-Recipient -Identity $GroupSmtp -ErrorAction SilentlyContinue)) {
  New-DistributionGroup `
    -Name $GroupName `
    -DisplayName $GroupDisplayName `
    -Alias $GroupAlias `
    -PrimarySmtpAddress $GroupSmtp `
    -Type Distribution
}

# Configure baseline properties.
Set-DistributionGroup -Identity $GroupSmtp `
  -ManagedBy $OwnerUpn `
  -RequireSenderAuthenticationEnabled $true `
  -HiddenFromAddressListsEnabled $false `
  -Notes "Lab distribution group for Exchange Online workbook 03"

# Add members.
foreach ($Member in $Members) {
  Add-DistributionGroupMember `
    -Identity $GroupSmtp `
    -Member $Member `
    -BypassSecurityGroupManagerCheck `
    -ErrorAction SilentlyContinue
}

# Validate group object.
Get-DistributionGroup -Identity $GroupSmtp |
  Format-List DisplayName,Name,PrimarySmtpAddress,RecipientTypeDetails,ManagedBy,RequireSenderAuthenticationEnabled,HiddenFromAddressListsEnabled |
  Out-File "$EvidencePath\distribution-group-after.txt"

# Validate membership.
Get-DistributionGroupMember -Identity $GroupSmtp |
  Select-Object Name,PrimarySmtpAddress,RecipientTypeDetails |
  Export-Csv "$EvidencePath\distribution-group-members.csv" -NoTypeInformation

Stop-Transcript
```

# Manage_Distribution_Groups_Dynamic_Groups_Mail_Enabled_Security_Groups_And_M365_Groups_Dynamic_Distribution_Group_Skeleton

```powershell
# Run on the admin workstation.
# Purpose: create and validate a dynamic distribution group.
# Dynamic distribution groups do not have manually added members. Membership is calculated from the recipient filter.

$EvidenceRoot = "C:\M365-Exchange-Baseline"
$RunStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$EvidencePath = Join-Path $EvidenceRoot "03-Dynamic-Distribution-Group-$RunStamp"

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\dynamic-distribution-group-transcript.txt"

# Variables.
$DDGName = "DDG Sales User Mailboxes"
$DDGAlias = "ddg-sales-user-mailboxes"
$DDGSmtp = "ddg-sales-user-mailboxes@contoso.com"
$OwnerUpn = "alex.wilber@contoso.com"

# Example filter:
# User mailboxes where Department equals Sales.
$RecipientFilter = "(RecipientTypeDetails -eq 'UserMailbox') -and (Department -eq 'Sales')"

# Precheck for existing group or address conflict.
Get-Recipient -Identity $DDGSmtp -ErrorAction SilentlyContinue |
  Format-List Name,PrimarySmtpAddress,RecipientTypeDetails |
  Out-File "$EvidencePath\dynamic-group-existing-check.txt"

# Preview filter before creating group.
Get-Recipient -RecipientPreviewFilter $RecipientFilter -ResultSize Unlimited |
  Select-Object DisplayName,PrimarySmtpAddress,RecipientTypeDetails,Department |
  Export-Csv "$EvidencePath\dynamic-group-filter-preview-before-create.csv" -NoTypeInformation

# Create dynamic distribution group if it does not exist.
if (-not (Get-Recipient -Identity $DDGSmtp -ErrorAction SilentlyContinue)) {
  New-DynamicDistributionGroup `
    -Name $DDGName `
    -Alias $DDGAlias `
    -PrimarySmtpAddress $DDGSmtp `
    -RecipientFilter $RecipientFilter
}

# Configure baseline properties.
Set-DynamicDistributionGroup -Identity $DDGSmtp `
  -ManagedBy $OwnerUpn `
  -RequireSenderAuthenticationEnabled $true `
  -HiddenFromAddressListsEnabled $false `
  -Notes "Lab dynamic distribution group for Sales user mailboxes"

# Validate group object.
Get-DynamicDistributionGroup -Identity $DDGSmtp |
  Format-List DisplayName,Name,PrimarySmtpAddress,RecipientTypeDetails,RecipientFilter,ManagedBy,RequireSenderAuthenticationEnabled,HiddenFromAddressListsEnabled |
  Out-File "$EvidencePath\dynamic-distribution-group-after.txt"

# Preview final membership.
$DDG = Get-DynamicDistributionGroup -Identity $DDGSmtp

Get-Recipient -RecipientPreviewFilter $DDG.RecipientFilter -ResultSize Unlimited |
  Select-Object DisplayName,PrimarySmtpAddress,RecipientTypeDetails,Department |
  Export-Csv "$EvidencePath\dynamic-group-filter-preview-after-create.csv" -NoTypeInformation

Stop-Transcript
```

# Manage_Distribution_Groups_Dynamic_Groups_Mail_Enabled_Security_Groups_And_M365_Groups_Mail_Enabled_Security_Group_Skeleton

```powershell
# Run on the admin workstation.
# Purpose: create, configure, populate, and validate a mail-enabled security group.
# Use mail-enabled security groups when the group must receive email and also be usable for permissions.

$EvidenceRoot = "C:\M365-Exchange-Baseline"
$RunStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$EvidencePath = Join-Path $EvidenceRoot "03-Mail-Enabled-Security-Group-$RunStamp"

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\mail-enabled-security-group-transcript.txt"

# Variables.
$SecurityGroupName = "MESG Finance Share Access"
$SecurityGroupAlias = "mesg-finance-share-access"
$SecurityGroupSmtp = "mesg-finance-share-access@contoso.com"
$SecurityGroupDisplayName = "MESG Finance Share Access"
$OwnerUpn = "alex.wilber@contoso.com"
$Members = @(
  "alex.wilber@contoso.com",
  "adele.vance@contoso.com"
)

# Precheck for existing group or address conflict.
Get-Recipient -Identity $SecurityGroupSmtp -ErrorAction SilentlyContinue |
  Format-List Name,PrimarySmtpAddress,RecipientTypeDetails |
  Out-File "$EvidencePath\mail-enabled-security-group-existing-check.txt"

# Create mail-enabled security group if it does not exist.
if (-not (Get-Recipient -Identity $SecurityGroupSmtp -ErrorAction SilentlyContinue)) {
  New-DistributionGroup `
    -Name $SecurityGroupName `
    -DisplayName $SecurityGroupDisplayName `
    -Alias $SecurityGroupAlias `
    -PrimarySmtpAddress $SecurityGroupSmtp `
    -Type Security
}

# Configure baseline properties.
Set-DistributionGroup -Identity $SecurityGroupSmtp `
  -ManagedBy $OwnerUpn `
  -RequireSenderAuthenticationEnabled $true `
  -HiddenFromAddressListsEnabled $false `
  -Notes "Lab mail-enabled security group for Exchange Online workbook 03"

# Add members.
foreach ($Member in $Members) {
  Add-DistributionGroupMember `
    -Identity $SecurityGroupSmtp `
    -Member $Member `
    -BypassSecurityGroupManagerCheck `
    -ErrorAction SilentlyContinue
}

# Validate group object.
Get-DistributionGroup -Identity $SecurityGroupSmtp |
  Format-List DisplayName,Name,PrimarySmtpAddress,RecipientTypeDetails,GroupType,ManagedBy,RequireSenderAuthenticationEnabled,HiddenFromAddressListsEnabled |
  Out-File "$EvidencePath\mail-enabled-security-group-after.txt"

# Validate membership.
Get-DistributionGroupMember -Identity $SecurityGroupSmtp |
  Select-Object Name,PrimarySmtpAddress,RecipientTypeDetails |
  Export-Csv "$EvidencePath\mail-enabled-security-group-members.csv" -NoTypeInformation

Stop-Transcript
```

# Manage_Distribution_Groups_Dynamic_Groups_Mail_Enabled_Security_Groups_And_M365_Groups_M365_Group_Skeleton

```powershell
# Run on the admin workstation.
# Purpose: create, configure, populate, and validate a Microsoft 365 group.
# Microsoft 365 groups are collaboration groups, not simple mail distribution lists.

$EvidenceRoot = "C:\M365-Exchange-Baseline"
$RunStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$EvidencePath = Join-Path $EvidenceRoot "03-M365-Group-$RunStamp"

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\m365-group-transcript.txt"

# Variables.
$M365GroupName = "M365 Project Phoenix"
$M365GroupAlias = "m365-project-phoenix"
$M365GroupSmtp = "m365-project-phoenix@contoso.com"
$AccessType = "Private"
$OwnerUpn = "alex.wilber@contoso.com"
$Members = @(
  "alex.wilber@contoso.com",
  "adele.vance@contoso.com"
)

# Precheck for existing group or address conflict.
Get-Recipient -Identity $M365GroupSmtp -ErrorAction SilentlyContinue |
  Format-List Name,PrimarySmtpAddress,RecipientTypeDetails |
  Out-File "$EvidencePath\m365-group-existing-check.txt"

# Create Microsoft 365 group if it does not exist.
if (-not (Get-Recipient -Identity $M365GroupSmtp -ErrorAction SilentlyContinue)) {
  New-UnifiedGroup `
    -DisplayName $M365GroupName `
    -Alias $M365GroupAlias `
    -PrimarySmtpAddress $M365GroupSmtp `
    -AccessType $AccessType `
    -Owner $OwnerUpn
}

# Configure baseline Exchange-visible group properties.
Set-UnifiedGroup -Identity $M365GroupSmtp `
  -AccessType $AccessType `
  -HiddenFromAddressListsEnabled $false `
  -HiddenFromExchangeClientsEnabled $false `
  -AutoSubscribeNewMembers:$true `
  -UnifiedGroupWelcomeMessageEnabled:$true `
  -Notes "Lab Microsoft 365 group for Exchange Online workbook 03"

# Add owner link.
Add-UnifiedGroupLinks `
  -Identity $M365GroupSmtp `
  -LinkType Owners `
  -Links $OwnerUpn `
  -ErrorAction SilentlyContinue

# Add members.
foreach ($Member in $Members) {
  Add-UnifiedGroupLinks `
    -Identity $M365GroupSmtp `
    -LinkType Members `
    -Links $Member `
    -ErrorAction SilentlyContinue
}

# Optional: subscribe members so they receive group conversations in inbox.
foreach ($Member in $Members) {
  Add-UnifiedGroupLinks `
    -Identity $M365GroupSmtp `
    -LinkType Subscribers `
    -Links $Member `
    -ErrorAction SilentlyContinue
}

# Validate group object.
Get-UnifiedGroup -Identity $M365GroupSmtp |
  Format-List DisplayName,PrimarySmtpAddress,AccessType,ManagedBy,HiddenFromAddressListsEnabled,HiddenFromExchangeClientsEnabled,AutoSubscribeNewMembers,UnifiedGroupWelcomeMessageEnabled |
  Out-File "$EvidencePath\m365-group-after.txt"

# Validate owners.
Get-UnifiedGroupLinks -Identity $M365GroupSmtp -LinkType Owners |
  Select-Object Name,PrimarySmtpAddress,RecipientTypeDetails |
  Export-Csv "$EvidencePath\m365-group-owners.csv" -NoTypeInformation

# Validate members.
Get-UnifiedGroupLinks -Identity $M365GroupSmtp -LinkType Members |
  Select-Object Name,PrimarySmtpAddress,RecipientTypeDetails |
  Export-Csv "$EvidencePath\m365-group-members.csv" -NoTypeInformation

# Validate subscribers.
Get-UnifiedGroupLinks -Identity $M365GroupSmtp -LinkType Subscribers |
  Select-Object Name,PrimarySmtpAddress,RecipientTypeDetails |
  Export-Csv "$EvidencePath\m365-group-subscribers.csv" -NoTypeInformation

Stop-Transcript
```

# Manage_Distribution_Groups_Dynamic_Groups_Mail_Enabled_Security_Groups_And_M365_Groups_Moderation_And_Delivery_Restrictions_Skeleton

```powershell
# Run on the admin workstation.
# Purpose: configure sender restrictions, moderation, and visibility for mail-enabled groups.
# Apply only to groups where this matches the business requirement.

$EvidenceRoot = "C:\M365-Exchange-Baseline"
$RunStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$EvidencePath = Join-Path $EvidenceRoot "03-Group-Moderation-Restrictions-$RunStamp"

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\group-moderation-restrictions-transcript.txt"

# Variables.
$GroupSmtp = "dl-helpdesk-notifications@contoso.com"
$ModeratorUpn = "alex.wilber@contoso.com"
$AllowedSender = "adele.vance@contoso.com"

# Capture before state.
Get-DistributionGroup -Identity $GroupSmtp |
  Format-List DisplayName,PrimarySmtpAddress,RequireSenderAuthenticationEnabled,ModerationEnabled,ModeratedBy,AcceptMessagesOnlyFromSendersOrMembers,RejectMessagesFromSendersOrMembers,HiddenFromAddressListsEnabled |
  Out-File "$EvidencePath\group-restrictions-before.txt"

# Block external unauthenticated senders.
Set-DistributionGroup -Identity $GroupSmtp `
  -RequireSenderAuthenticationEnabled $true

# Enable moderation.
Set-DistributionGroup -Identity $GroupSmtp `
  -ModerationEnabled $true `
  -ModeratedBy $ModeratorUpn `
  -SendModerationNotifications Always

# Optional: restrict delivery to specific sender or group.
# Use carefully. This can block expected mail flow.
# Set-DistributionGroup -Identity $GroupSmtp -AcceptMessagesOnlyFromSendersOrMembers $AllowedSender

# Optional: hide group from address lists.
# Set-DistributionGroup -Identity $GroupSmtp -HiddenFromAddressListsEnabled $true

# Capture after state.
Get-DistributionGroup -Identity $GroupSmtp |
  Format-List DisplayName,PrimarySmtpAddress,RequireSenderAuthenticationEnabled,ModerationEnabled,ModeratedBy,AcceptMessagesOnlyFromSendersOrMembers,RejectMessagesFromSendersOrMembers,HiddenFromAddressListsEnabled |
  Out-File "$EvidencePath\group-restrictions-after.txt"

Stop-Transcript
```

# Manage_Distribution_Groups_Dynamic_Groups_Mail_Enabled_Security_Groups_And_M365_Groups_Bulk_Inventory_Skeleton

```powershell
# Run on the admin workstation.
# Purpose: export static groups, dynamic groups, Microsoft 365 groups, members, owners, and key mail settings.

$EvidenceRoot = "C:\M365-Exchange-Baseline"
$RunStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$EvidencePath = Join-Path $EvidenceRoot "03-Group-Inventory-$RunStamp"

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\group-inventory-transcript.txt"

# Static distribution groups and mail-enabled security groups.
$DistributionGroups = Get-DistributionGroup -ResultSize Unlimited

$DistributionGroups |
  Select-Object DisplayName,Name,PrimarySmtpAddress,RecipientTypeDetails,GroupType,ManagedBy,RequireSenderAuthenticationEnabled,ModerationEnabled,HiddenFromAddressListsEnabled |
  Export-Csv "$EvidencePath\distribution-and-mail-enabled-security-groups.csv" -NoTypeInformation

# Static group members.
$DistributionGroupMembers = foreach ($Group in $DistributionGroups) {
  Get-DistributionGroupMember -Identity $Group.PrimarySmtpAddress -ResultSize Unlimited -ErrorAction SilentlyContinue |
    Select-Object @{Name="Group";Expression={$Group.PrimarySmtpAddress}},Name,PrimarySmtpAddress,RecipientTypeDetails
}

$DistributionGroupMembers |
  Export-Csv "$EvidencePath\distribution-group-members.csv" -NoTypeInformation

# Dynamic distribution groups.
$DynamicGroups = Get-DynamicDistributionGroup -ResultSize Unlimited

$DynamicGroups |
  Select-Object DisplayName,Name,PrimarySmtpAddress,RecipientFilter,ManagedBy,RequireSenderAuthenticationEnabled,ModerationEnabled,HiddenFromAddressListsEnabled |
  Export-Csv "$EvidencePath\dynamic-distribution-groups.csv" -NoTypeInformation

# Dynamic distribution group previews.
foreach ($Group in $DynamicGroups) {
  $SafeName = ($Group.PrimarySmtpAddress.ToString()).Replace("@","_at_").Replace(".","_")

  Get-Recipient -RecipientPreviewFilter $Group.RecipientFilter -ResultSize Unlimited |
    Select-Object @{Name="DynamicGroup";Expression={$Group.PrimarySmtpAddress}},DisplayName,PrimarySmtpAddress,RecipientTypeDetails,Department,Office |
    Export-Csv "$EvidencePath\dynamic-preview-$SafeName.csv" -NoTypeInformation
}

# Microsoft 365 groups.
$UnifiedGroups = Get-UnifiedGroup -ResultSize Unlimited

$UnifiedGroups |
  Select-Object DisplayName,PrimarySmtpAddress,AccessType,ManagedBy,HiddenFromAddressListsEnabled,HiddenFromExchangeClientsEnabled,AutoSubscribeNewMembers,UnifiedGroupWelcomeMessageEnabled |
  Export-Csv "$EvidencePath\m365-groups.csv" -NoTypeInformation

# Microsoft 365 group owners and members.
$UnifiedGroupOwners = foreach ($Group in $UnifiedGroups) {
  Get-UnifiedGroupLinks -Identity $Group.PrimarySmtpAddress -LinkType Owners -ErrorAction SilentlyContinue |
    Select-Object @{Name="Group";Expression={$Group.PrimarySmtpAddress}},Name,PrimarySmtpAddress,RecipientTypeDetails
}

$UnifiedGroupOwners |
  Export-Csv "$EvidencePath\m365-group-owners.csv" -NoTypeInformation

$UnifiedGroupMembers = foreach ($Group in $UnifiedGroups) {
  Get-UnifiedGroupLinks -Identity $Group.PrimarySmtpAddress -LinkType Members -ErrorAction SilentlyContinue |
    Select-Object @{Name="Group";Expression={$Group.PrimarySmtpAddress}},Name,PrimarySmtpAddress,RecipientTypeDetails
}

$UnifiedGroupMembers |
  Export-Csv "$EvidencePath\m365-group-members.csv" -NoTypeInformation

# Recipient type counts.
Get-Recipient -ResultSize Unlimited |
  Group-Object RecipientTypeDetails |
  Sort-Object Name |
  Select-Object Name,Count |
  Export-Csv "$EvidencePath\recipient-type-counts.csv" -NoTypeInformation

Stop-Transcript
```

# Manage_Distribution_Groups_Dynamic_Groups_Mail_Enabled_Security_Groups_And_M365_Groups_Validation_Skeleton

```powershell
# Run on the admin workstation.
# Purpose: validate the created group types, memberships, filters, owners, and visibility.

$EvidenceRoot = "C:\M365-Exchange-Baseline"
$RunStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$EvidencePath = Join-Path $EvidenceRoot "03-Groups-Validation-$RunStamp"

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\groups-validation-transcript.txt"

# Variables.
$DistributionGroupSmtp = "dl-helpdesk-notifications@contoso.com"
$DynamicGroupSmtp = "ddg-sales-user-mailboxes@contoso.com"
$SecurityGroupSmtp = "mesg-finance-share-access@contoso.com"
$M365GroupSmtp = "m365-project-phoenix@contoso.com"

# Validate distribution group.
Get-DistributionGroup -Identity $DistributionGroupSmtp -ErrorAction SilentlyContinue |
  Select-Object DisplayName,PrimarySmtpAddress,RecipientTypeDetails,ManagedBy,RequireSenderAuthenticationEnabled,ModerationEnabled |
  Tee-Object "$EvidencePath\validate-distribution-group.txt"

Get-DistributionGroupMember -Identity $DistributionGroupSmtp -ErrorAction SilentlyContinue |
  Select-Object Name,PrimarySmtpAddress,RecipientTypeDetails |
  Export-Csv "$EvidencePath\validate-distribution-group-members.csv" -NoTypeInformation

# Validate dynamic distribution group.
$DDG = Get-DynamicDistributionGroup -Identity $DynamicGroupSmtp -ErrorAction SilentlyContinue

$DDG |
  Select-Object DisplayName,PrimarySmtpAddress,RecipientTypeDetails,RecipientFilter,ManagedBy |
  Tee-Object "$EvidencePath\validate-dynamic-distribution-group.txt"

if ($DDG) {
  Get-Recipient -RecipientPreviewFilter $DDG.RecipientFilter -ResultSize Unlimited |
    Select-Object DisplayName,PrimarySmtpAddress,RecipientTypeDetails,Department,Office |
    Export-Csv "$EvidencePath\validate-dynamic-distribution-group-preview.csv" -NoTypeInformation
}

# Validate mail-enabled security group.
Get-DistributionGroup -Identity $SecurityGroupSmtp -ErrorAction SilentlyContinue |
  Select-Object DisplayName,PrimarySmtpAddress,RecipientTypeDetails,GroupType,ManagedBy,RequireSenderAuthenticationEnabled |
  Tee-Object "$EvidencePath\validate-mail-enabled-security-group.txt"

Get-DistributionGroupMember -Identity $SecurityGroupSmtp -ErrorAction SilentlyContinue |
  Select-Object Name,PrimarySmtpAddress,RecipientTypeDetails |
  Export-Csv "$EvidencePath\validate-mail-enabled-security-group-members.csv" -NoTypeInformation

# Validate Microsoft 365 group.
Get-UnifiedGroup -Identity $M365GroupSmtp -ErrorAction SilentlyContinue |
  Select-Object DisplayName,PrimarySmtpAddress,AccessType,HiddenFromAddressListsEnabled,HiddenFromExchangeClientsEnabled |
  Tee-Object "$EvidencePath\validate-m365-group.txt"

Get-UnifiedGroupLinks -Identity $M365GroupSmtp -LinkType Owners -ErrorAction SilentlyContinue |
  Select-Object Name,PrimarySmtpAddress,RecipientTypeDetails |
  Export-Csv "$EvidencePath\validate-m365-group-owners.csv" -NoTypeInformation

Get-UnifiedGroupLinks -Identity $M365GroupSmtp -LinkType Members -ErrorAction SilentlyContinue |
  Select-Object Name,PrimarySmtpAddress,RecipientTypeDetails |
  Export-Csv "$EvidencePath\validate-m365-group-members.csv" -NoTypeInformation

# Final counts.
Get-Recipient -ResultSize Unlimited |
  Group-Object RecipientTypeDetails |
  Sort-Object Name |
  Select-Object Name,Count |
  Export-Csv "$EvidencePath\recipient-type-counts-after.csv" -NoTypeInformation

Stop-Transcript
```

# Manage_Distribution_Groups_Dynamic_Groups_Mail_Enabled_Security_Groups_And_M365_Groups_Verification_Commands

| Check | Device | PowerShell / Command | Healthy Output |
|---|---|---|---|
| Exchange Online session active | Admin Workstation | `Get-ConnectionInformation` | Active session exists |
| Accepted domain exists | Admin Workstation | `Get-AcceptedDomain` | Target SMTP domain appears |
| Distribution group exists | Admin Workstation | `Get-DistributionGroup -Identity "<group-smtp>"` | RecipientTypeDetails is MailUniversalDistributionGroup |
| Distribution group members exist | Admin Workstation | `Get-DistributionGroupMember -Identity "<group-smtp>"` | Expected members appear |
| Distribution group owner configured | Admin Workstation | `Get-DistributionGroup -Identity "<group-smtp>" \| Format-List ManagedBy` | Expected owner appears |
| Dynamic distribution group exists | Admin Workstation | `Get-DynamicDistributionGroup -Identity "<ddg-smtp>"` | Dynamic group appears |
| Dynamic distribution group filter previews correctly | Admin Workstation | `$g = Get-DynamicDistributionGroup "<ddg-smtp>"; Get-Recipient -RecipientPreviewFilter $g.RecipientFilter` | Expected recipients match the filter |
| Mail-enabled security group exists | Admin Workstation | `Get-DistributionGroup -Identity "<mesg-smtp>"` | RecipientTypeDetails is MailUniversalSecurityGroup |
| Mail-enabled security group members exist | Admin Workstation | `Get-DistributionGroupMember -Identity "<mesg-smtp>"` | Expected members appear |
| Microsoft 365 group exists | Admin Workstation | `Get-UnifiedGroup -Identity "<m365-group-smtp>"` | Group appears |
| Microsoft 365 group owners exist | Admin Workstation | `Get-UnifiedGroupLinks -Identity "<m365-group-smtp>" -LinkType Owners` | Expected owner appears |
| Microsoft 365 group members exist | Admin Workstation | `Get-UnifiedGroupLinks -Identity "<m365-group-smtp>" -LinkType Members` | Expected members appear |
| External sender restriction configured | Admin Workstation | `Get-DistributionGroup -Identity "<group-smtp>" \| Format-List RequireSenderAuthenticationEnabled` | Value matches standard |
| Moderation configured | Admin Workstation | `Get-DistributionGroup -Identity "<group-smtp>" \| Format-List ModerationEnabled,ModeratedBy` | Moderation matches standard |
| Group visibility configured | Admin Workstation | `Get-DistributionGroup -Identity "<group-smtp>" \| Format-List HiddenFromAddressListsEnabled` | Visibility matches standard |
| M365 group client visibility configured | Admin Workstation | `Get-UnifiedGroup -Identity "<m365-group-smtp>" \| Format-List HiddenFromExchangeClientsEnabled` | Visibility matches standard |
| EAC group view | Browser | Exchange admin center > Recipients > Groups | Groups appear in correct group category |

# Manage_Distribution_Groups_Dynamic_Groups_Mail_Enabled_Security_Groups_And_M365_Groups_Rollback

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm rollback scope | Operator | Review lab-created group names and SMTP addresses | Only lab groups are targeted |
| 2 | Export group state before removal | Admin Workstation | `Get-DistributionGroup`; `Get-DynamicDistributionGroup`; `Get-UnifiedGroup` | Pre-removal evidence is saved |
| 3 | Remove lab static distribution group | Admin Workstation | `Remove-DistributionGroup -Identity "<group-smtp>" -Confirm:$false` | Distribution group is removed |
| 4 | Remove lab dynamic distribution group | Admin Workstation | `Remove-DynamicDistributionGroup -Identity "<ddg-smtp>" -Confirm:$false` | Dynamic distribution group is removed |
| 5 | Remove lab mail-enabled security group | Admin Workstation | `Remove-DistributionGroup -Identity "<mesg-smtp>" -Confirm:$false` | Mail-enabled security group is removed |
| 6 | Remove lab Microsoft 365 group | Admin Workstation | `Remove-UnifiedGroup -Identity "<m365-group-smtp>" -Confirm:$false` | Microsoft 365 group is removed |
| 7 | Validate removal | Admin Workstation | `Get-Recipient -Identity "<smtp>" -ErrorAction SilentlyContinue` | Removed groups no longer appear |
| 8 | Export final group inventory | Admin Workstation | Export group state again | Post-rollback evidence is saved |
| 9 | Disconnect session | Admin Workstation | `Disconnect-ExchangeOnline -Confirm:$false` | Session closes |

# Manage_Distribution_Groups_Dynamic_Groups_Mail_Enabled_Security_Groups_And_M365_Groups_Rollback_Helper

```powershell
# Run on the admin workstation.
# Purpose: remove lab-created Exchange Online groups only.
# Do not run this against production groups.

$EvidenceRoot = "C:\M365-Exchange-Baseline"
$RunStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$EvidencePath = Join-Path $EvidenceRoot "03-Groups-Rollback-$RunStamp"

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\groups-rollback-transcript.txt"

# Lab group identities.
$DistributionGroupSmtp = "dl-helpdesk-notifications@contoso.com"
$DynamicGroupSmtp = "ddg-sales-user-mailboxes@contoso.com"
$SecurityGroupSmtp = "mesg-finance-share-access@contoso.com"
$M365GroupSmtp = "m365-project-phoenix@contoso.com"

$RollbackTargets = @(
  $DistributionGroupSmtp,
  $DynamicGroupSmtp,
  $SecurityGroupSmtp,
  $M365GroupSmtp
)

# Capture before removal.
foreach ($Target in $RollbackTargets) {
  Get-Recipient -Identity $Target -ErrorAction SilentlyContinue |
    Format-List Name,PrimarySmtpAddress,RecipientTypeDetails |
    Out-File "$EvidencePath\$($Target.Replace('@','_at_'))-before-removal.txt"
}

# Remove static distribution group.
Remove-DistributionGroup -Identity $DistributionGroupSmtp -Confirm:$false -ErrorAction SilentlyContinue

# Remove dynamic distribution group.
Remove-DynamicDistributionGroup -Identity $DynamicGroupSmtp -Confirm:$false -ErrorAction SilentlyContinue

# Remove mail-enabled security group.
Remove-DistributionGroup -Identity $SecurityGroupSmtp -Confirm:$false -ErrorAction SilentlyContinue

# Remove Microsoft 365 group.
Remove-UnifiedGroup -Identity $M365GroupSmtp -Confirm:$false -ErrorAction SilentlyContinue

# Validate after removal.
foreach ($Target in $RollbackTargets) {
  Get-Recipient -Identity $Target -ErrorAction SilentlyContinue |
    Format-List Name,PrimarySmtpAddress,RecipientTypeDetails |
    Out-File "$EvidencePath\$($Target.Replace('@','_at_'))-after-removal.txt"
}

# Final group inventory.
Get-DistributionGroup -ResultSize Unlimited |
  Select-Object DisplayName,PrimarySmtpAddress,RecipientTypeDetails |
  Export-Csv "$EvidencePath\distribution-groups-after-rollback.csv" -NoTypeInformation

Get-DynamicDistributionGroup -ResultSize Unlimited |
  Select-Object DisplayName,PrimarySmtpAddress,RecipientTypeDetails,RecipientFilter |
  Export-Csv "$EvidencePath\dynamic-distribution-groups-after-rollback.csv" -NoTypeInformation

Get-UnifiedGroup -ResultSize Unlimited |
  Select-Object DisplayName,PrimarySmtpAddress,AccessType |
  Export-Csv "$EvidencePath\m365-groups-after-rollback.csv" -NoTypeInformation

Stop-Transcript
```

# Manage_Distribution_Groups_Dynamic_Groups_Mail_Enabled_Security_Groups_And_M365_Groups_Failure_Checks

| Symptom | Likely Cause | Check Command | Fix |
|---|---|---|---|
| Group creation fails with duplicate address | Alias or SMTP address already exists on another recipient | `Get-Recipient -ResultSize Unlimited \| Where-Object {$_.EmailAddresses -match "<smtp>"}` | Choose a unique alias and SMTP address |
| `Set-DistributionGroup` fails on synced group | Group is mastered on-premises | `Get-DistributionGroup -Identity "<group>" \| Format-List IsDirSynced` | Modify source object on-premises and allow sync |
| Cannot add group member | Member identity wrong, member not mail-enabled, or RBAC issue | `Get-Recipient -Identity "<member>"` | Use correct member identity and verify admin role |
| `Add-DistributionGroupMember` says insufficient permissions | Group manager restriction or RBAC missing | `Get-DistributionGroup -Identity "<group>" \| Format-List ManagedBy` | Use authorized owner or `-BypassSecurityGroupManagerCheck` as admin |
| Distribution group receives internal mail but not external mail | External sender blocked by `RequireSenderAuthenticationEnabled` | `Get-DistributionGroup -Identity "<group>" \| Format-List RequireSenderAuthenticationEnabled` | Set based on business requirement |
| External senders can mail group unexpectedly | `RequireSenderAuthenticationEnabled` is false | `Get-DistributionGroup -Identity "<group>" \| Format-List RequireSenderAuthenticationEnabled` | Set to `$true` if external mail should be blocked |
| Dynamic group has wrong recipients | Recipient filter is too broad or wrong attribute values | `$g = Get-DynamicDistributionGroup "<group>"; Get-Recipient -RecipientPreviewFilter $g.RecipientFilter` | Fix recipient attributes or narrow the filter |
| Dynamic group has no recipients | Filter field values do not match actual recipient attributes | `Get-Recipient -ResultSize Unlimited \| Select DisplayName,Department,Office` | Correct filter or populate required attributes |
| Mail-enabled security group appears as distribution group | Group created with wrong type | `Get-DistributionGroup -Identity "<group>" \| Format-List RecipientTypeDetails,GroupType` | Recreate as `-Type Security` if lab group |
| Microsoft 365 group does not show in Outlook | Hidden from Exchange clients or provisioning delay | `Get-UnifiedGroup -Identity "<group>" \| Format-List HiddenFromExchangeClientsEnabled` | Set `HiddenFromExchangeClientsEnabled $false` and wait for propagation |
| Microsoft 365 group members do not receive conversations in inbox | Members are not subscribers | `Get-UnifiedGroupLinks -Identity "<group>" -LinkType Subscribers` | Add subscribers or enable subscription behavior |
| Owner cannot manage group | Owner missing, synced group, or portal permission issue | `Get-DistributionGroup -Identity "<group>" \| Format-List ManagedBy` | Add owner or manage from correct source of authority |
| Moderation does not trigger | Moderation disabled or bypass sender configured | `Get-DistributionGroup -Identity "<group>" \| Format-List ModerationEnabled,ModeratedBy,BypassModerationFromSendersOrMembers` | Enable moderation and verify bypass list |
| EAC does not show group immediately | Portal cache or provisioning delay | Validate with PowerShell | Refresh, sign out and back in, or wait |
| Remove group command fails | Wrong cmdlet for group type | `Get-Recipient -Identity "<group>" \| Format-List RecipientTypeDetails` | Use `Remove-DistributionGroup`, `Remove-DynamicDistributionGroup`, or `Remove-UnifiedGroup` as appropriate |
| Command works in PowerShell but portal blocks change | RBAC propagation or role mismatch | Check Microsoft 365 admin roles and Exchange role groups | Reopen portal session or use proper admin role |

# Manage_Distribution_Groups_Dynamic_Groups_Mail_Enabled_Security_Groups_And_M365_Groups_Related_Labs

| Related Lab | Relationship |
|---|---|
| `01_Configure_Exchange_Admin_Center_PowerShell_And_Org_Baseline.md` | Provides admin access, Exchange Online PowerShell, RBAC, and baseline exports |
| `02_Manage_User_Mailboxes_Shared_Mailboxes_Contacts_And_Resources.md` | Provides recipient objects used as group members and owners |
| `04_Configure_Mail_Flow_Accepted_Domains_Connectors_And_Rules.md` | Uses groups as transport rule senders, recipients, exceptions, and targets |
| `05_Configure_Mailbox_Permissions_Delegation_Forwarding_And_Auditing.md` | Uses mail-enabled security groups and M365 groups in permission scenarios |
| `06_Configure_Exchange_Online_Protection_AntiSpam_AntiMalware_And_Quarantine.md` | Uses groups for policy targeting and mail protection testing |
| `07_Configure_Retention_Archive_Litigation_Hold_And_eDiscovery_Handoff.md` | Uses M365 groups and mailboxes for compliance handoff scenarios |
| `08_Troubleshoot_Mail_Flow_Mailbox_Access_Permissions_And_Quarantine_Issues.md` | Uses group membership, moderation, and delivery restriction evidence during troubleshooting |
