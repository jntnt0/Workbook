# Manage_User_Mailboxes_Shared_Mailboxes_Contacts_And_Resources

# Manage_User_Mailboxes_Shared_Mailboxes_Contacts_And_Resources_Index
02_Manage_User_Mailboxes_Shared_Mailboxes_Contacts_And_Resources.md
Manage_User_Mailboxes_Shared_Mailboxes_Contacts_And_Resources
Manage_User_Mailboxes_Shared_Mailboxes_Contacts_And_Resources_Source_Basis
Manage_User_Mailboxes_Shared_Mailboxes_Contacts_And_Resources_Mental_Model
Manage_User_Mailboxes_Shared_Mailboxes_Contacts_And_Resources_Planning_Table
Manage_User_Mailboxes_Shared_Mailboxes_Contacts_And_Resources_Configuration_Checklist
Manage_User_Mailboxes_Shared_Mailboxes_Contacts_And_Resources_Precheck_Skeleton
Manage_User_Mailboxes_Shared_Mailboxes_Contacts_And_Resources_User_Mailbox_Skeleton
Manage_User_Mailboxes_Shared_Mailboxes_Contacts_And_Resources_Shared_Mailbox_Skeleton
Manage_User_Mailboxes_Shared_Mailboxes_Contacts_And_Resources_Mail_Contact_Skeleton
Manage_User_Mailboxes_Shared_Mailboxes_Contacts_And_Resources_Room_Mailbox_Skeleton
Manage_User_Mailboxes_Shared_Mailboxes_Contacts_And_Resources_Equipment_Mailbox_Skeleton
Manage_User_Mailboxes_Shared_Mailboxes_Contacts_And_Resources_Bulk_Inventory_Skeleton
Manage_User_Mailboxes_Shared_Mailboxes_Contacts_And_Resources_Validation_Skeleton
Manage_User_Mailboxes_Shared_Mailboxes_Contacts_And_Resources_Verification_Commands
Manage_User_Mailboxes_Shared_Mailboxes_Contacts_And_Resources_Rollback
Manage_User_Mailboxes_Shared_Mailboxes_Contacts_And_Resources_Failure_Checks
Manage_User_Mailboxes_Shared_Mailboxes_Contacts_And_Resources_Related_Labs

# Manage_User_Mailboxes_Shared_Mailboxes_Contacts_And_Resources_Source_Basis

| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Exchange admin center recipients | Managing user mailboxes, shared mailboxes, contacts, room mailboxes, and equipment mailboxes |
| Microsoft Learn | Exchange Online PowerShell | Managing recipients with ExchangeOnlineManagement module |
| Microsoft Learn | Get-EXOMailbox / Get-Mailbox | Mailbox inventory and validation |
| Microsoft Learn | New-Mailbox / Set-Mailbox / Remove-Mailbox | Creating, updating, and removing mailbox objects |
| Microsoft Learn | New-MailContact / Set-MailContact / Remove-MailContact | Creating, updating, and removing mail contacts |
| Microsoft Learn | Set-CalendarProcessing | Configuring room and equipment booking behavior |
| Microsoft Learn | Recipient filters and RecipientTypeDetails | Distinguishing user mailboxes, shared mailboxes, room mailboxes, equipment mailboxes, and contacts |
| Prior workbook | 01_Configure_Exchange_Admin_Center_PowerShell_And_Org_Baseline.md | Uses the existing Exchange Online PowerShell connection and baseline evidence model |

# Manage_User_Mailboxes_Shared_Mailboxes_Contacts_And_Resources_Mental_Model

| Concept | Operational Meaning |
|---|---|
| User mailbox | Mailbox tied to a licensed user account |
| Shared mailbox | Mailbox used by multiple people, usually not directly signed into |
| Mail contact | Mail-enabled address book object that points to an external email address |
| Room mailbox | Resource mailbox used to book physical rooms |
| Equipment mailbox | Resource mailbox used to book equipment, vehicles, carts, projectors, or other shared assets |
| RecipientTypeDetails | Exchange field that identifies the exact recipient type |
| Primary SMTP address | Main reply address for the recipient |
| Alias | Mail nickname used in address creation and recipient identity |
| Display name | Human-readable name shown in address lists and admin tools |
| HiddenFromAddressListsEnabled | Controls whether a recipient appears in address lists |
| Calendar processing | Resource mailbox booking behavior controlled by `Set-CalendarProcessing` |
| Cloud-only recipient | Recipient mastered in Microsoft 365 / Exchange Online |
| Synced recipient | Recipient mastered from on-premises Active Directory through Entra Connect |
| First rule | Do not modify synced recipients directly in cloud unless the property is cloud-owned |
| Second rule | Validate recipient type before applying mailbox, contact, or resource commands |

# Manage_User_Mailboxes_Shared_Mailboxes_Contacts_And_Resources_Planning_Table

| Item | Example | Decision |
|---|---|---|
| Tenant name | `contoso.onmicrosoft.com` | `<tenant-name>` |
| Primary SMTP domain | `contoso.com` | `<primary-smtp-domain>` |
| Admin UPN | `admin@contoso.com` | `<admin-upn>` |
| Evidence root | `C:\M365-Exchange-Baseline` | `<evidence-root>` |
| Mailbox naming standard | `First Last` | `<naming-standard>` |
| Shared mailbox naming standard | `Department Name` | `<shared-mailbox-standard>` |
| Contact naming standard | `External Company - Contact Name` | `<contact-standard>` |
| Room naming standard | `Building-Floor-Room` | `<room-standard>` |
| Equipment naming standard | `Equipment-Type-AssetTag` | `<equipment-standard>` |
| Default mailbox domain | `contoso.com` | `<default-domain>` |
| User mailbox source of authority | Cloud-only / Hybrid synced | `<source-of-authority>` |
| Address list visibility standard | Visible by default | `<visible-hidden>` |
| Shared mailbox sign-in blocked | Yes | `<yes-no>` |
| Resource auto-accept standard | AutoAccept | `<resource-processing>` |
| Room booking window | `180` days | `<booking-window>` |
| Max room booking duration | `120` minutes | `<duration-minutes>` |
| Change ticket | `LAB-EXO-002` | `<ticket-or-lab-id>` |
| Rollback stance | Remove lab-created recipients only | `<rollback-scope>` |

# Manage_User_Mailboxes_Shared_Mailboxes_Contacts_And_Resources_Configuration_Checklist

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm Exchange Online connection | Admin Workstation | `Get-ConnectionInformation` | Active Exchange Online connection exists |
| 2 | Confirm accepted domain | Admin Workstation | `Get-AcceptedDomain` | Target SMTP domain exists |
| 3 | Confirm recipient baseline | Admin Workstation | `Get-Recipient -ResultSize 10` | Recipient data returns |
| 4 | Confirm mailbox visibility | Admin Workstation | `Get-EXOMailbox -ResultSize 10` | Mailbox data returns |
| 5 | Confirm whether tenant is cloud-only or hybrid | Admin Workstation | Review Entra Connect / directory sync state | Source of authority is known |
| 6 | Create evidence directory | Admin Workstation | `New-Item -ItemType Directory -Force -Path C:\M365-Exchange-Baseline` | Evidence path exists |
| 7 | Export current recipient counts | Admin Workstation | `Get-Recipient -ResultSize Unlimited \| Group-Object RecipientTypeDetails` | Recipient baseline is saved |
| 8 | Review existing user mailbox | Admin Workstation | `Get-EXOMailbox -Identity "<user-upn>"` | Target mailbox state is known |
| 9 | Update user mailbox display/address list settings if required | Admin Workstation | `Set-Mailbox -Identity "<user-upn>" -HiddenFromAddressListsEnabled $false` | Mailbox property updates |
| 10 | Create shared mailbox | Admin Workstation | `New-Mailbox -Shared -Name "<name>" -DisplayName "<display-name>" -Alias "<alias>" -PrimarySmtpAddress "<smtp>"` | Shared mailbox exists |
| 11 | Validate shared mailbox | Admin Workstation | `Get-EXOMailbox -Identity "<smtp>"` | RecipientTypeDetails is SharedMailbox |
| 12 | Create mail contact | Admin Workstation | `New-MailContact -Name "<name>" -ExternalEmailAddress "<external-email>"` | Mail contact exists |
| 13 | Validate mail contact | Admin Workstation | `Get-MailContact -Identity "<name>"` | External email address is correct |
| 14 | Create room mailbox | Admin Workstation | `New-Mailbox -Room -Name "<room-name>" -Alias "<alias>" -PrimarySmtpAddress "<smtp>"` | Room mailbox exists |
| 15 | Configure room booking processing | Admin Workstation | `Set-CalendarProcessing -Identity "<room-smtp>" -AutomateProcessing AutoAccept` | Room auto-accept behavior is configured |
| 16 | Validate room mailbox | Admin Workstation | `Get-EXOMailbox -Identity "<room-smtp>"` | RecipientTypeDetails is RoomMailbox |
| 17 | Create equipment mailbox | Admin Workstation | `New-Mailbox -Equipment -Name "<equipment-name>" -Alias "<alias>" -PrimarySmtpAddress "<smtp>"` | Equipment mailbox exists |
| 18 | Configure equipment booking processing | Admin Workstation | `Set-CalendarProcessing -Identity "<equipment-smtp>" -AutomateProcessing AutoAccept` | Equipment auto-accept behavior is configured |
| 19 | Validate equipment mailbox | Admin Workstation | `Get-EXOMailbox -Identity "<equipment-smtp>"` | RecipientTypeDetails is EquipmentMailbox |
| 20 | Export final recipient inventory | Admin Workstation | `Get-Recipient -ResultSize Unlimited` | Final inventory is saved |
| 21 | Compare before and after counts | Admin Workstation | Compare exported CSV files | Expected object count changes are visible |
| 22 | Disconnect Exchange Online session | Admin Workstation | `Disconnect-ExchangeOnline -Confirm:$false` | Session closes cleanly |

# Manage_User_Mailboxes_Shared_Mailboxes_Contacts_And_Resources_Precheck_Skeleton

```powershell
# Run on the admin workstation.
# Purpose: validate Exchange Online connection, accepted domain, source of authority, and evidence path.

$AdminUpn = "admin@contoso.com"
$PrimaryDomain = "contoso.com"
$EvidenceRoot = "C:\M365-Exchange-Baseline"
$RunStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$EvidencePath = Join-Path $EvidenceRoot "02-Recipients-Precheck-$RunStamp"

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\precheck-transcript.txt"

Import-Module ExchangeOnlineManagement

# Connect if not already connected.
try {
  $Connection = Get-ConnectionInformation -ErrorAction Stop
}
catch {
  Connect-ExchangeOnline -UserPrincipalName $AdminUpn -ShowBanner:$false
}

# Capture connection state.
Get-ConnectionInformation |
  Tee-Object "$EvidencePath\connection-information.txt"

# Validate accepted domain.
Get-AcceptedDomain |
  Select-Object Name,DomainName,DomainType,Default |
  Export-Csv "$EvidencePath\accepted-domains.csv" -NoTypeInformation

Get-AcceptedDomain |
  Where-Object {$_.DomainName -eq $PrimaryDomain} |
  Format-List * |
  Out-File "$EvidencePath\target-domain-check.txt"

# Recipient baseline counts.
Get-Recipient -ResultSize Unlimited |
  Group-Object RecipientTypeDetails |
  Sort-Object Name |
  Select-Object Name,Count |
  Export-Csv "$EvidencePath\recipient-type-counts-before.csv" -NoTypeInformation

# Mailbox samples.
Get-EXOMailbox -ResultSize 100 |
  Select-Object DisplayName,UserPrincipalName,PrimarySmtpAddress,RecipientTypeDetails,ArchiveStatus |
  Export-Csv "$EvidencePath\mailbox-sample-before.csv" -NoTypeInformation

# Existing shared mailboxes.
Get-EXOMailbox -RecipientTypeDetails SharedMailbox -ResultSize Unlimited |
  Select-Object DisplayName,PrimarySmtpAddress,HiddenFromAddressListsEnabled |
  Export-Csv "$EvidencePath\shared-mailboxes-before.csv" -NoTypeInformation

# Existing contacts.
Get-MailContact -ResultSize Unlimited |
  Select-Object DisplayName,Name,ExternalEmailAddress,PrimarySmtpAddress,HiddenFromAddressListsEnabled |
  Export-Csv "$EvidencePath\mail-contacts-before.csv" -NoTypeInformation

# Existing resource mailboxes.
Get-EXOMailbox -RecipientTypeDetails RoomMailbox,EquipmentMailbox -ResultSize Unlimited |
  Select-Object DisplayName,PrimarySmtpAddress,RecipientTypeDetails,HiddenFromAddressListsEnabled |
  Export-Csv "$EvidencePath\resource-mailboxes-before.csv" -NoTypeInformation

Stop-Transcript
```

# Manage_User_Mailboxes_Shared_Mailboxes_Contacts_And_Resources_User_Mailbox_Skeleton

```powershell
# Run on the admin workstation.
# Purpose: inspect and manage existing user mailboxes.
# Note: In many Microsoft 365 tenants, user mailboxes are created by licensing users in Microsoft 365 admin center or Entra ID.
# In hybrid environments, create and modify source-owned identity attributes on-premises.

$EvidenceRoot = "C:\M365-Exchange-Baseline"
$RunStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$EvidencePath = Join-Path $EvidenceRoot "02-User-Mailboxes-$RunStamp"

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\user-mailbox-transcript.txt"

# Variables.
$UserUpn = "alex.wilber@contoso.com"
$NewDisplayName = "Alex Wilber"
$HideFromAddressLists = $false

# Inspect mailbox.
Get-EXOMailbox -Identity $UserUpn |
  Format-List DisplayName,UserPrincipalName,PrimarySmtpAddress,RecipientTypeDetails,HiddenFromAddressListsEnabled,ArchiveStatus |
  Out-File "$EvidencePath\user-mailbox-before.txt"

# Inspect full mailbox object if standard Get-Mailbox cmdlet is needed for properties not returned by Get-EXOMailbox.
Get-Mailbox -Identity $UserUpn |
  Format-List * |
  Out-File "$EvidencePath\user-mailbox-full-before.txt"

# Example safe mailbox metadata update.
# Use only if the tenant source of authority allows cloud-side edits.
Set-Mailbox -Identity $UserUpn `
  -DisplayName $NewDisplayName `
  -HiddenFromAddressListsEnabled $HideFromAddressLists

# Validate update.
Get-EXOMailbox -Identity $UserUpn |
  Format-List DisplayName,UserPrincipalName,PrimarySmtpAddress,RecipientTypeDetails,HiddenFromAddressListsEnabled,ArchiveStatus |
  Out-File "$EvidencePath\user-mailbox-after.txt"

Stop-Transcript
```

# Manage_User_Mailboxes_Shared_Mailboxes_Contacts_And_Resources_User_Mailbox_Create_Lab_Skeleton

```powershell
# Run on the admin workstation.
# Purpose: create a cloud-only user mailbox for lab use when cloud-side mailbox creation is allowed.
# Do not use this for synced users mastered by on-premises Active Directory.

$EvidenceRoot = "C:\M365-Exchange-Baseline"
$RunStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$EvidencePath = Join-Path $EvidenceRoot "02-User-Mailbox-Create-Lab-$RunStamp"

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\user-mailbox-create-lab-transcript.txt"

# Variables.
$Name = "Lab User One"
$Alias = "lab.user.one"
$UserPrincipalName = "lab.user.one@contoso.com"
$PrimarySmtpAddress = "lab.user.one@contoso.com"
$Password = Read-Host "Enter temporary password" -AsSecureString

# Create lab mailbox and cloud user.
New-Mailbox `
  -Name $Name `
  -Alias $Alias `
  -UserPrincipalName $UserPrincipalName `
  -PrimarySmtpAddress $PrimarySmtpAddress `
  -Password $Password `
  -ResetPasswordOnNextLogon $true

# Validate.
Get-EXOMailbox -Identity $UserPrincipalName |
  Format-List DisplayName,UserPrincipalName,PrimarySmtpAddress,RecipientTypeDetails |
  Out-File "$EvidencePath\created-user-mailbox.txt"

Stop-Transcript
```

# Manage_User_Mailboxes_Shared_Mailboxes_Contacts_And_Resources_Shared_Mailbox_Skeleton

```powershell
# Run on the admin workstation.
# Purpose: create and validate a shared mailbox.
# Deep delegation permissions belong in workbook 05. This workbook only creates the shared mailbox object.

$EvidenceRoot = "C:\M365-Exchange-Baseline"
$RunStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$EvidencePath = Join-Path $EvidenceRoot "02-Shared-Mailbox-$RunStamp"

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\shared-mailbox-transcript.txt"

# Variables.
$SharedName = "Help Desk"
$SharedDisplayName = "Help Desk"
$SharedAlias = "helpdesk"
$SharedSmtp = "helpdesk@contoso.com"

# Precheck for existing recipient.
Get-Recipient -Identity $SharedSmtp -ErrorAction SilentlyContinue |
  Format-List Name,PrimarySmtpAddress,RecipientTypeDetails |
  Out-File "$EvidencePath\shared-mailbox-existing-check.txt"

# Create shared mailbox if it does not exist.
if (-not (Get-Recipient -Identity $SharedSmtp -ErrorAction SilentlyContinue)) {
  New-Mailbox `
    -Shared `
    -Name $SharedName `
    -DisplayName $SharedDisplayName `
    -Alias $SharedAlias `
    -PrimarySmtpAddress $SharedSmtp
}

# Set basic shared mailbox properties.
Set-Mailbox -Identity $SharedSmtp `
  -HiddenFromAddressListsEnabled $false

# Optional: keep copies of sent messages in shared mailbox Sent Items when delegated send is configured later.
Set-Mailbox -Identity $SharedSmtp `
  -MessageCopyForSentAsEnabled $true `
  -MessageCopyForSendOnBehalfEnabled $true

# Validate.
Get-EXOMailbox -Identity $SharedSmtp |
  Format-List DisplayName,PrimarySmtpAddress,RecipientTypeDetails,HiddenFromAddressListsEnabled |
  Out-File "$EvidencePath\shared-mailbox-after.txt"

Get-Mailbox -Identity $SharedSmtp |
  Format-List MessageCopyForSentAsEnabled,MessageCopyForSendOnBehalfEnabled |
  Out-File "$EvidencePath\shared-mailbox-sent-items-settings.txt"

Stop-Transcript
```

# Manage_User_Mailboxes_Shared_Mailboxes_Contacts_And_Resources_Mail_Contact_Skeleton

```powershell
# Run on the admin workstation.
# Purpose: create and validate an external mail contact.

$EvidenceRoot = "C:\M365-Exchange-Baseline"
$RunStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$EvidencePath = Join-Path $EvidenceRoot "02-Mail-Contact-$RunStamp"

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\mail-contact-transcript.txt"

# Variables.
$ContactName = "Vendor Support"
$ContactDisplayName = "Vendor Support"
$ContactAlias = "vendor.support"
$ExternalEmail = "support@vendor.example"
$ContactPrimarySmtp = "vendor.support@contoso.com"

# Precheck.
Get-Recipient -Identity $ContactName -ErrorAction SilentlyContinue |
  Format-List Name,PrimarySmtpAddress,RecipientTypeDetails |
  Out-File "$EvidencePath\mail-contact-existing-check.txt"

# Create mail contact if it does not exist.
if (-not (Get-MailContact -Identity $ContactName -ErrorAction SilentlyContinue)) {
  New-MailContact `
    -Name $ContactName `
    -DisplayName $ContactDisplayName `
    -Alias $ContactAlias `
    -ExternalEmailAddress $ExternalEmail
}

# Optional: set address list visibility and custom primary SMTP if required by tenant standard.
Set-MailContact -Identity $ContactName `
  -HiddenFromAddressListsEnabled $false

# Validate.
Get-MailContact -Identity $ContactName |
  Format-List DisplayName,Name,Alias,ExternalEmailAddress,PrimarySmtpAddress,HiddenFromAddressListsEnabled |
  Out-File "$EvidencePath\mail-contact-after.txt"

Stop-Transcript
```

# Manage_User_Mailboxes_Shared_Mailboxes_Contacts_And_Resources_Room_Mailbox_Skeleton

```powershell
# Run on the admin workstation.
# Purpose: create and configure a room mailbox.

$EvidenceRoot = "C:\M365-Exchange-Baseline"
$RunStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$EvidencePath = Join-Path $EvidenceRoot "02-Room-Mailbox-$RunStamp"

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\room-mailbox-transcript.txt"

# Variables.
$RoomName = "HQ 2nd Floor Conference Room 201"
$RoomDisplayName = "HQ-2F-Conference-201"
$RoomAlias = "hq-2f-conf-201"
$RoomSmtp = "hq-2f-conf-201@contoso.com"
$Capacity = 12
$BookingWindowInDays = 180
$MaximumDurationInMinutes = 120

# Precheck.
Get-Recipient -Identity $RoomSmtp -ErrorAction SilentlyContinue |
  Format-List Name,PrimarySmtpAddress,RecipientTypeDetails |
  Out-File "$EvidencePath\room-existing-check.txt"

# Create room mailbox if it does not exist.
if (-not (Get-Recipient -Identity $RoomSmtp -ErrorAction SilentlyContinue)) {
  New-Mailbox `
    -Room `
    -Name $RoomName `
    -DisplayName $RoomDisplayName `
    -Alias $RoomAlias `
    -PrimarySmtpAddress $RoomSmtp
}

# Configure basic room properties.
Set-Mailbox -Identity $RoomSmtp `
  -ResourceCapacity $Capacity `
  -HiddenFromAddressListsEnabled $false

# Configure booking behavior.
Set-CalendarProcessing -Identity $RoomSmtp `
  -AutomateProcessing AutoAccept `
  -BookingWindowInDays $BookingWindowInDays `
  -MaximumDurationInMinutes $MaximumDurationInMinutes `
  -AllowConflicts $false `
  -DeleteComments $false `
  -DeleteSubject $false `
  -AddOrganizerToSubject $true

# Validate.
Get-EXOMailbox -Identity $RoomSmtp |
  Format-List DisplayName,PrimarySmtpAddress,RecipientTypeDetails,ResourceCapacity,HiddenFromAddressListsEnabled |
  Out-File "$EvidencePath\room-mailbox-after.txt"

Get-CalendarProcessing -Identity $RoomSmtp |
  Format-List AutomateProcessing,BookingWindowInDays,MaximumDurationInMinutes,AllowConflicts,DeleteComments,DeleteSubject,AddOrganizerToSubject |
  Out-File "$EvidencePath\room-calendar-processing-after.txt"

Stop-Transcript
```

# Manage_User_Mailboxes_Shared_Mailboxes_Contacts_And_Resources_Equipment_Mailbox_Skeleton

```powershell
# Run on the admin workstation.
# Purpose: create and configure an equipment mailbox.

$EvidenceRoot = "C:\M365-Exchange-Baseline"
$RunStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$EvidencePath = Join-Path $EvidenceRoot "02-Equipment-Mailbox-$RunStamp"

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\equipment-mailbox-transcript.txt"

# Variables.
$EquipmentName = "Projector Cart 01"
$EquipmentDisplayName = "Projector Cart 01"
$EquipmentAlias = "projector-cart-01"
$EquipmentSmtp = "projector-cart-01@contoso.com"
$BookingWindowInDays = 180
$MaximumDurationInMinutes = 480

# Precheck.
Get-Recipient -Identity $EquipmentSmtp -ErrorAction SilentlyContinue |
  Format-List Name,PrimarySmtpAddress,RecipientTypeDetails |
  Out-File "$EvidencePath\equipment-existing-check.txt"

# Create equipment mailbox if it does not exist.
if (-not (Get-Recipient -Identity $EquipmentSmtp -ErrorAction SilentlyContinue)) {
  New-Mailbox `
    -Equipment `
    -Name $EquipmentName `
    -DisplayName $EquipmentDisplayName `
    -Alias $EquipmentAlias `
    -PrimarySmtpAddress $EquipmentSmtp
}

# Configure basic equipment properties.
Set-Mailbox -Identity $EquipmentSmtp `
  -HiddenFromAddressListsEnabled $false

# Configure booking behavior.
Set-CalendarProcessing -Identity $EquipmentSmtp `
  -AutomateProcessing AutoAccept `
  -BookingWindowInDays $BookingWindowInDays `
  -MaximumDurationInMinutes $MaximumDurationInMinutes `
  -AllowConflicts $false `
  -DeleteComments $false `
  -DeleteSubject $false

# Validate.
Get-EXOMailbox -Identity $EquipmentSmtp |
  Format-List DisplayName,PrimarySmtpAddress,RecipientTypeDetails,HiddenFromAddressListsEnabled |
  Out-File "$EvidencePath\equipment-mailbox-after.txt"

Get-CalendarProcessing -Identity $EquipmentSmtp |
  Format-List AutomateProcessing,BookingWindowInDays,MaximumDurationInMinutes,AllowConflicts,DeleteComments,DeleteSubject |
  Out-File "$EvidencePath\equipment-calendar-processing-after.txt"

Stop-Transcript
```

# Manage_User_Mailboxes_Shared_Mailboxes_Contacts_And_Resources_Bulk_Inventory_Skeleton

```powershell
# Run on the admin workstation.
# Purpose: export recipient inventory for user mailboxes, shared mailboxes, contacts, rooms, and equipment.

$EvidenceRoot = "C:\M365-Exchange-Baseline"
$RunStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$EvidencePath = Join-Path $EvidenceRoot "02-Recipient-Inventory-$RunStamp"

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\recipient-inventory-transcript.txt"

# All recipient type counts.
Get-Recipient -ResultSize Unlimited |
  Group-Object RecipientTypeDetails |
  Sort-Object Name |
  Select-Object Name,Count |
  Export-Csv "$EvidencePath\recipient-type-counts.csv" -NoTypeInformation

# User mailboxes.
Get-EXOMailbox -RecipientTypeDetails UserMailbox -ResultSize Unlimited |
  Select-Object DisplayName,UserPrincipalName,PrimarySmtpAddress,RecipientTypeDetails,ArchiveStatus,HiddenFromAddressListsEnabled |
  Export-Csv "$EvidencePath\user-mailboxes.csv" -NoTypeInformation

# Shared mailboxes.
Get-EXOMailbox -RecipientTypeDetails SharedMailbox -ResultSize Unlimited |
  Select-Object DisplayName,PrimarySmtpAddress,RecipientTypeDetails,HiddenFromAddressListsEnabled |
  Export-Csv "$EvidencePath\shared-mailboxes.csv" -NoTypeInformation

# Room mailboxes.
Get-EXOMailbox -RecipientTypeDetails RoomMailbox -ResultSize Unlimited |
  Select-Object DisplayName,PrimarySmtpAddress,RecipientTypeDetails,ResourceCapacity,HiddenFromAddressListsEnabled |
  Export-Csv "$EvidencePath\room-mailboxes.csv" -NoTypeInformation

# Equipment mailboxes.
Get-EXOMailbox -RecipientTypeDetails EquipmentMailbox -ResultSize Unlimited |
  Select-Object DisplayName,PrimarySmtpAddress,RecipientTypeDetails,HiddenFromAddressListsEnabled |
  Export-Csv "$EvidencePath\equipment-mailboxes.csv" -NoTypeInformation

# Contacts.
Get-MailContact -ResultSize Unlimited |
  Select-Object DisplayName,Name,Alias,ExternalEmailAddress,PrimarySmtpAddress,HiddenFromAddressListsEnabled |
  Export-Csv "$EvidencePath\mail-contacts.csv" -NoTypeInformation

# Resource calendar processing.
$Resources = Get-EXOMailbox -RecipientTypeDetails RoomMailbox,EquipmentMailbox -ResultSize Unlimited

$ResourceCalendarProcessing = foreach ($Resource in $Resources) {
  Get-CalendarProcessing -Identity $Resource.PrimarySmtpAddress |
    Select-Object Identity,AutomateProcessing,BookingWindowInDays,MaximumDurationInMinutes,AllowConflicts,DeleteComments,DeleteSubject
}

$ResourceCalendarProcessing |
  Export-Csv "$EvidencePath\resource-calendar-processing.csv" -NoTypeInformation

Stop-Transcript
```

# Manage_User_Mailboxes_Shared_Mailboxes_Contacts_And_Resources_Validation_Skeleton

```powershell
# Run on the admin workstation.
# Purpose: validate expected recipients exist and are the correct recipient type.

$EvidenceRoot = "C:\M365-Exchange-Baseline"
$RunStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$EvidencePath = Join-Path $EvidenceRoot "02-Recipients-Validation-$RunStamp"

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\recipient-validation-transcript.txt"

# Variables.
$UserUpn = "alex.wilber@contoso.com"
$SharedSmtp = "helpdesk@contoso.com"
$ContactName = "Vendor Support"
$RoomSmtp = "hq-2f-conf-201@contoso.com"
$EquipmentSmtp = "projector-cart-01@contoso.com"

# Validate user mailbox.
Get-EXOMailbox -Identity $UserUpn -ErrorAction SilentlyContinue |
  Select-Object DisplayName,UserPrincipalName,PrimarySmtpAddress,RecipientTypeDetails |
  Tee-Object "$EvidencePath\validate-user-mailbox.txt"

# Validate shared mailbox.
Get-EXOMailbox -Identity $SharedSmtp -ErrorAction SilentlyContinue |
  Select-Object DisplayName,PrimarySmtpAddress,RecipientTypeDetails |
  Tee-Object "$EvidencePath\validate-shared-mailbox.txt"

# Validate contact.
Get-MailContact -Identity $ContactName -ErrorAction SilentlyContinue |
  Select-Object DisplayName,ExternalEmailAddress,PrimarySmtpAddress,RecipientTypeDetails |
  Tee-Object "$EvidencePath\validate-mail-contact.txt"

# Validate room.
Get-EXOMailbox -Identity $RoomSmtp -ErrorAction SilentlyContinue |
  Select-Object DisplayName,PrimarySmtpAddress,RecipientTypeDetails,ResourceCapacity |
  Tee-Object "$EvidencePath\validate-room-mailbox.txt"

Get-CalendarProcessing -Identity $RoomSmtp -ErrorAction SilentlyContinue |
  Select-Object Identity,AutomateProcessing,BookingWindowInDays,MaximumDurationInMinutes,AllowConflicts |
  Tee-Object "$EvidencePath\validate-room-calendar-processing.txt"

# Validate equipment.
Get-EXOMailbox -Identity $EquipmentSmtp -ErrorAction SilentlyContinue |
  Select-Object DisplayName,PrimarySmtpAddress,RecipientTypeDetails |
  Tee-Object "$EvidencePath\validate-equipment-mailbox.txt"

Get-CalendarProcessing -Identity $EquipmentSmtp -ErrorAction SilentlyContinue |
  Select-Object Identity,AutomateProcessing,BookingWindowInDays,MaximumDurationInMinutes,AllowConflicts |
  Tee-Object "$EvidencePath\validate-equipment-calendar-processing.txt"

# Final count.
Get-Recipient -ResultSize Unlimited |
  Group-Object RecipientTypeDetails |
  Sort-Object Name |
  Select-Object Name,Count |
  Export-Csv "$EvidencePath\recipient-type-counts-after.csv" -NoTypeInformation

Stop-Transcript
```

# Manage_User_Mailboxes_Shared_Mailboxes_Contacts_And_Resources_Verification_Commands

| Check | Device | PowerShell / Command | Healthy Output |
|---|---|---|---|
| Exchange Online session active | Admin Workstation | `Get-ConnectionInformation` | Active connection exists |
| Accepted domain exists | Admin Workstation | `Get-AcceptedDomain` | Target SMTP domain appears |
| User mailbox exists | Admin Workstation | `Get-EXOMailbox -Identity "<user-upn>"` | RecipientTypeDetails is UserMailbox |
| Shared mailbox exists | Admin Workstation | `Get-EXOMailbox -Identity "<shared-smtp>"` | RecipientTypeDetails is SharedMailbox |
| Mail contact exists | Admin Workstation | `Get-MailContact -Identity "<contact-name>"` | ExternalEmailAddress is correct |
| Room mailbox exists | Admin Workstation | `Get-EXOMailbox -Identity "<room-smtp>"` | RecipientTypeDetails is RoomMailbox |
| Room booking settings exist | Admin Workstation | `Get-CalendarProcessing -Identity "<room-smtp>"` | AutomateProcessing and booking limits are visible |
| Equipment mailbox exists | Admin Workstation | `Get-EXOMailbox -Identity "<equipment-smtp>"` | RecipientTypeDetails is EquipmentMailbox |
| Equipment booking settings exist | Admin Workstation | `Get-CalendarProcessing -Identity "<equipment-smtp>"` | AutomateProcessing and booking limits are visible |
| Recipient address list visibility | Admin Workstation | `Get-Recipient -Identity "<recipient>" \| Format-List HiddenFromAddressListsEnabled` | Visibility matches standard |
| Recipient count baseline | Admin Workstation | `Get-Recipient -ResultSize Unlimited \| Group-Object RecipientTypeDetails` | Counts match expected changes |
| EAC user mailbox view | Browser | Exchange admin center > Recipients > Mailboxes | User mailbox appears |
| EAC shared mailbox view | Browser | Exchange admin center > Recipients > Mailboxes > Shared | Shared mailbox appears |
| EAC resource view | Browser | Exchange admin center > Recipients > Resources | Room and equipment appear |
| EAC contact view | Browser | Exchange admin center > Recipients > Contacts | Mail contact appears |

# Manage_User_Mailboxes_Shared_Mailboxes_Contacts_And_Resources_Rollback

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm rollback scope | Operator | Review lab-created recipients | Only intended lab objects are removed |
| 2 | Export pre-removal state | Admin Workstation | `Get-Recipient -Identity "<recipient>"` | Object state is captured |
| 3 | Remove lab shared mailbox | Admin Workstation | `Remove-Mailbox -Identity "<shared-smtp>" -Confirm:$false` | Shared mailbox is removed or soft-deleted |
| 4 | Remove lab contact | Admin Workstation | `Remove-MailContact -Identity "<contact-name>" -Confirm:$false` | Mail contact is removed |
| 5 | Remove lab room mailbox | Admin Workstation | `Remove-Mailbox -Identity "<room-smtp>" -Confirm:$false` | Room mailbox is removed or soft-deleted |
| 6 | Remove lab equipment mailbox | Admin Workstation | `Remove-Mailbox -Identity "<equipment-smtp>" -Confirm:$false` | Equipment mailbox is removed or soft-deleted |
| 7 | Remove lab cloud-only user mailbox only if approved | Admin Workstation | `Remove-Mailbox -Identity "<user-upn>" -Confirm:$false` | Lab mailbox is removed or soft-deleted |
| 8 | Validate removal | Admin Workstation | `Get-Recipient -Identity "<recipient>" -ErrorAction SilentlyContinue` | Removed object no longer appears as active recipient |
| 9 | Export final recipient counts | Admin Workstation | `Get-Recipient -ResultSize Unlimited \| Group-Object RecipientTypeDetails` | Post-rollback counts are captured |
| 10 | Disconnect session | Admin Workstation | `Disconnect-ExchangeOnline -Confirm:$false` | Session closes |

# Manage_User_Mailboxes_Shared_Mailboxes_Contacts_And_Resources_Rollback_Helper

```powershell
# Run on the admin workstation.
# Purpose: remove lab-created recipients only.
# Do not run this against production recipients.

$EvidenceRoot = "C:\M365-Exchange-Baseline"
$RunStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$EvidencePath = Join-Path $EvidenceRoot "02-Recipients-Rollback-$RunStamp"

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\rollback-transcript.txt"

# Lab objects.
$LabSharedMailbox = "helpdesk@contoso.com"
$LabContact = "Vendor Support"
$LabRoom = "hq-2f-conf-201@contoso.com"
$LabEquipment = "projector-cart-01@contoso.com"
$LabUserMailbox = "lab.user.one@contoso.com"

# Capture before removal.
$RollbackTargets = @(
  $LabSharedMailbox,
  $LabContact,
  $LabRoom,
  $LabEquipment,
  $LabUserMailbox
)

foreach ($Target in $RollbackTargets) {
  Get-Recipient -Identity $Target -ErrorAction SilentlyContinue |
    Format-List Name,PrimarySmtpAddress,RecipientTypeDetails |
    Out-File "$EvidencePath\$($Target.Replace('@','_at_'))-before-removal.txt"
}

# Remove only lab-created objects.
Remove-Mailbox -Identity $LabSharedMailbox -Confirm:$false -ErrorAction SilentlyContinue
Remove-MailContact -Identity $LabContact -Confirm:$false -ErrorAction SilentlyContinue
Remove-Mailbox -Identity $LabRoom -Confirm:$false -ErrorAction SilentlyContinue
Remove-Mailbox -Identity $LabEquipment -Confirm:$false -ErrorAction SilentlyContinue

# Only remove lab user mailbox if it was created specifically for this lab.
# Remove-Mailbox -Identity $LabUserMailbox -Confirm:$false -ErrorAction SilentlyContinue

# Validate after removal.
foreach ($Target in $RollbackTargets) {
  Get-Recipient -Identity $Target -ErrorAction SilentlyContinue |
    Format-List Name,PrimarySmtpAddress,RecipientTypeDetails |
    Out-File "$EvidencePath\$($Target.Replace('@','_at_'))-after-removal.txt"
}

Get-Recipient -ResultSize Unlimited |
  Group-Object RecipientTypeDetails |
  Sort-Object Name |
  Select-Object Name,Count |
  Export-Csv "$EvidencePath\recipient-type-counts-after-rollback.csv" -NoTypeInformation

Stop-Transcript
```

# Manage_User_Mailboxes_Shared_Mailboxes_Contacts_And_Resources_Failure_Checks

| Symptom | Likely Cause | Check Command | Fix |
|---|---|---|---|
| `New-Mailbox` fails for user mailbox | Tenant uses synced identity or user already exists | `Get-Recipient -Identity "<upn>"` | Create user from correct source of authority or license existing user |
| User mailbox does not appear after licensing | License not assigned, Exchange service plan disabled, or provisioning delay | Microsoft 365 admin center license page and `Get-EXOMailbox -Identity "<upn>"` | Enable Exchange Online service plan and wait for provisioning |
| `Set-Mailbox` fails on synced object | Attribute is mastered on-premises | Check directory sync state and recipient source | Modify object on-premises and allow sync |
| Shared mailbox creation fails | Alias, SMTP address, or name already exists | `Get-Recipient -Identity "<smtp>"` | Choose unique alias and SMTP address |
| Shared mailbox appears as user mailbox | Wrong creation command used | `Get-EXOMailbox -Identity "<smtp>" \| Format-List RecipientTypeDetails` | Remove lab object and recreate with `-Shared` if appropriate |
| Contact creation fails | External email address or alias conflict | `Get-Recipient -ResultSize Unlimited \| Where-Object {$_.PrimarySmtpAddress -like "*vendor*"}` | Use unique alias and verify external address |
| Room mailbox does not auto-accept | Calendar processing not configured | `Get-CalendarProcessing -Identity "<room-smtp>"` | Set `-AutomateProcessing AutoAccept` |
| Room accepts conflicting meetings | `AllowConflicts` is enabled or booking policy allows conflict | `Get-CalendarProcessing -Identity "<room-smtp>" \| Format-List AllowConflicts` | Set `-AllowConflicts $false` |
| Resource mailbox hidden unexpectedly | HiddenFromAddressListsEnabled is true | `Get-Mailbox -Identity "<resource>" \| Format-List HiddenFromAddressListsEnabled` | Set visibility to tenant standard |
| EAC does not show object immediately | Portal cache or provisioning delay | Validate with PowerShell | Wait, refresh portal, or sign out and back in |
| `Get-EXOMailbox` does not show expected property | REST-backed cmdlet property set is limited | Use `Get-Mailbox -Identity "<recipient>" \| Format-List *` | Use the cmdlet that exposes the required property |
| Remove command fails | Object is protected, synced, or wrong recipient type | `Get-Recipient -Identity "<recipient>" \| Format-List RecipientTypeDetails` | Use the correct remove cmdlet and correct source of authority |
| Mail contact receives no mail internally | Address book object exists but mail flow route or external address is wrong | `Get-MailContact -Identity "<contact>" \| Format-List ExternalEmailAddress` | Correct external address and test mail flow |
| Duplicate SMTP address error | Proxy address already exists on another recipient | `Get-Recipient -ResultSize Unlimited \| Where-Object {$_.EmailAddresses -match "<smtp>"}` | Remove or change duplicate address |
| Command works in PowerShell but not portal | RBAC or portal role delay | Check admin role and sign-in session | Reopen browser session or wait for role propagation |

# Manage_User_Mailboxes_Shared_Mailboxes_Contacts_And_Resources_Related_Labs

| Related Lab                                                                                    | Relationship                                                                   |
| ---------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------ |
| `01_Configure_Exchange_Admin_Center_PowerShell_And_Org_Baseline.md`                            | Provides admin access, PowerShell module, connection, and initial baseline     |
| `03_Manage_Distribution_Groups_Dynamic_Groups_Mail_Enabled_Security_Groups_And_M365_Groups.md` | Continues recipient management into group objects                              |
| `04_Configure_Mail_Flow_Accepted_Domains_Connectors_And_Rules.md`                              | Uses recipients as senders, recipients, and rule targets                       |
| `05_Configure_Mailbox_Permissions_Delegation_Forwarding_And_Auditing.md`                       | Adds Full Access, Send As, Send on Behalf, forwarding, and audit configuration |
| `06_Configure_Exchange_Online_Protection_AntiSpam_AntiMalware_And_Quarantine.md`               | Uses mailboxes and contacts for protection testing                             |
| `07_Configure_Retention_Archive_Litigation_Hold_And_eDiscovery_Handoff.md`                     | Applies retention, archive, and hold concepts to mailbox objects               |
| `08_Troubleshoot_Mail_Flow_Mailbox_Access_Permissions_And_Quarantine_Issues.md`                | Uses recipient baseline evidence for troubleshooting                           |