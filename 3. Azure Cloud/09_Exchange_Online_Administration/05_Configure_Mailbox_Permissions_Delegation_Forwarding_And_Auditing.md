# Configure_Mailbox_Permissions_Delegation_Forwarding_And_Auditing

# Configure_Mailbox_Permissions_Delegation_Forwarding_And_Auditing_Index
05_Configure_Mailbox_Permissions_Delegation_Forwarding_And_Auditing.md
Configure_Mailbox_Permissions_Delegation_Forwarding_And_Auditing
Configure_Mailbox_Permissions_Delegation_Forwarding_And_Auditing_Source_Basis
Configure_Mailbox_Permissions_Delegation_Forwarding_And_Auditing_Mental_Model
Configure_Mailbox_Permissions_Delegation_Forwarding_And_Auditing_Planning_Table
Configure_Mailbox_Permissions_Delegation_Forwarding_And_Auditing_Configuration_Checklist
Configure_Mailbox_Permissions_Delegation_Forwarding_And_Auditing_Precheck_Skeleton
Configure_Mailbox_Permissions_Delegation_Forwarding_And_Auditing_Full_Access_Skeleton
Configure_Mailbox_Permissions_Delegation_Forwarding_And_Auditing_Send_As_Skeleton
Configure_Mailbox_Permissions_Delegation_Forwarding_And_Auditing_Send_On_Behalf_Skeleton
Configure_Mailbox_Permissions_Delegation_Forwarding_And_Auditing_Calendar_Folder_Permission_Skeleton
Configure_Mailbox_Permissions_Delegation_Forwarding_And_Auditing_Forwarding_Skeleton
Configure_Mailbox_Permissions_Delegation_Forwarding_And_Auditing_Shared_Mailbox_Sent_Items_Skeleton
Configure_Mailbox_Permissions_Delegation_Forwarding_And_Auditing_Audit_Baseline_Skeleton
Configure_Mailbox_Permissions_Delegation_Forwarding_And_Auditing_Bulk_Inventory_Skeleton
Configure_Mailbox_Permissions_Delegation_Forwarding_And_Auditing_Validation_Skeleton
Configure_Mailbox_Permissions_Delegation_Forwarding_And_Auditing_Verification_Commands
Configure_Mailbox_Permissions_Delegation_Forwarding_And_Auditing_Rollback
Configure_Mailbox_Permissions_Delegation_Forwarding_And_Auditing_Failure_Checks
Configure_Mailbox_Permissions_Delegation_Forwarding_And_Auditing_Related_Labs

# Configure_Mailbox_Permissions_Delegation_Forwarding_And_Auditing_Source_Basis

| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Exchange admin center mailboxes | Managing mailbox delegation, forwarding, and mailbox properties |
| Microsoft Learn | Exchange Online PowerShell | Managing mailbox permissions and auditing from PowerShell |
| Microsoft Learn | Add-MailboxPermission / Remove-MailboxPermission | Full Access mailbox delegation |
| Microsoft Learn | Add-RecipientPermission / Remove-RecipientPermission | Send As delegation |
| Microsoft Learn | Set-Mailbox -GrantSendOnBehalfTo | Send on Behalf delegation |
| Microsoft Learn | Get-MailboxFolderPermission / Add-MailboxFolderPermission / Set-MailboxFolderPermission | Calendar and folder-level permissions |
| Microsoft Learn | Set-Mailbox forwarding properties | ForwardingAddress, ForwardingSmtpAddress, and DeliverToMailboxAndForward |
| Microsoft Learn | Mailbox auditing in Exchange Online | AuditEnabled, AuditAdmin, AuditDelegate, AuditOwner, and mailbox audit bypass checks |
| Microsoft Learn | Search-UnifiedAuditLog | Unified audit log validation for mailbox actions |
| Prior workbook | 01_Configure_Exchange_Admin_Center_PowerShell_And_Org_Baseline.md | Provides Exchange Online PowerShell connection and RBAC baseline |
| Prior workbook | 02_Manage_User_Mailboxes_Shared_Mailboxes_Contacts_And_Resources.md | Provides mailbox and shared mailbox objects |
| Prior workbook | 03_Manage_Distribution_Groups_Dynamic_Groups_Mail_Enabled_Security_Groups_And_M365_Groups.md | Provides groups that may be used for delegation and policy targeting |
| Prior workbook | 04_Configure_Mail_Flow_Accepted_Domains_Connectors_And_Rules.md | Provides mail flow and forwarding awareness before delegation changes |

# Configure_Mailbox_Permissions_Delegation_Forwarding_And_Auditing_Mental_Model

| Concept | Operational Meaning |
|---|---|
| Full Access | Delegate can open the mailbox and read or manage mailbox content |
| Send As | Delegate sends as the mailbox identity, hiding the delegate identity from recipients |
| Send on Behalf | Delegate sends on behalf of the mailbox, exposing both mailbox and delegate identity |
| Auto-mapping | Outlook automatically maps a delegated mailbox when Full Access is granted with auto-mapping enabled |
| Mailbox folder permission | Permission applied to a specific folder, commonly Calendar |
| Calendar delegate | User with folder-level access to another mailbox calendar |
| ForwardingAddress | Forwarding target that is an internal Exchange recipient |
| ForwardingSmtpAddress | Forwarding target that is an external SMTP address |
| DeliverToMailboxAndForward | Controls whether a copy is kept in the original mailbox when forwarding is enabled |
| Shared mailbox sent items copy | Shared mailbox setting that keeps Sent Items copies for Send As and Send on Behalf activity |
| Mailbox auditing | Logs mailbox actions by owner, delegate, or admin |
| AuditAdmin | Audited admin actions against the mailbox |
| AuditDelegate | Audited delegate actions against the mailbox |
| AuditOwner | Audited owner actions inside the mailbox |
| Unified audit log | Central audit log used to search Exchange and Microsoft 365 audit activity |
| Mailbox audit bypass | Setting that can exempt a user from mailbox audit logging |
| First rule | Permission changes are access changes, so capture before and after state every time |
| Second rule | Full Access does not automatically grant Send As or Send on Behalf |
| Third rule | Forwarding can create data leakage, so always document target, mailbox copy behavior, and approval |
| Fourth rule | Folder permissions are not the same as mailbox permissions |
| Fifth rule | Use groups for scalable permission assignment only when the workload supports the intended behavior |

# Configure_Mailbox_Permissions_Delegation_Forwarding_And_Auditing_Planning_Table

| Item | Example | Decision |
|---|---|---|
| Tenant name | `contoso.onmicrosoft.com` | `<tenant-name>` |
| Admin UPN | `admin@contoso.com` | `<admin-upn>` |
| Evidence root | `C:\M365-Exchange-Baseline` | `<evidence-root>` |
| Target user mailbox | `alex.wilber@contoso.com` | `<target-user-mailbox>` |
| Target shared mailbox | `helpdesk@contoso.com` | `<target-shared-mailbox>` |
| Delegate user | `adele.vance@contoso.com` | `<delegate-user>` |
| Delegate group | `mesg-helpdesk-delegates@contoso.com` | `<delegate-group>` |
| Full Access required | Yes / No | `<yes-no>` |
| Full Access auto-mapping | Enabled / Disabled | `<enabled-disabled>` |
| Send As required | Yes / No | `<yes-no>` |
| Send on Behalf required | Yes / No | `<yes-no>` |
| Calendar permission required | Reviewer / Editor / PublishingEditor | `<calendar-role>` |
| Forwarding required | Internal / External / None | `<forwarding-type>` |
| Forwarding target | `manager@contoso.com` or `archive@example.com` | `<forwarding-target>` |
| Keep mailbox copy while forwarding | Yes / No | `<yes-no>` |
| Shared mailbox Sent Items copy required | Yes | `<yes-no>` |
| Audit baseline required before changes | Yes | `<yes-no>` |
| Audit validation window | Last 24 hours / 7 days | `<audit-window>` |
| Change ticket | `LAB-EXO-005` | `<ticket-or-lab-id>` |
| Rollback stance | Remove lab permissions, forwarding, and test audit changes only | `<rollback-scope>` |

# Configure_Mailbox_Permissions_Delegation_Forwarding_And_Auditing_Configuration_Checklist

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm Exchange Online session | Admin Workstation | `Get-ConnectionInformation` | Active Exchange Online session exists |
| 2 | Confirm target mailbox exists | Admin Workstation | `Get-EXOMailbox -Identity "<mailbox>"` | Mailbox exists |
| 3 | Confirm delegate exists | Admin Workstation | `Get-Recipient -Identity "<delegate>"` | Delegate recipient exists |
| 4 | Export existing mailbox permissions | Admin Workstation | `Get-MailboxPermission -Identity "<mailbox>"` | Current Full Access state is saved |
| 5 | Export existing Send As permissions | Admin Workstation | `Get-RecipientPermission -Identity "<mailbox>"` | Current Send As state is saved |
| 6 | Export Send on Behalf state | Admin Workstation | `Get-Mailbox -Identity "<mailbox>" \| Format-List GrantSendOnBehalfTo` | Current Send on Behalf state is saved |
| 7 | Export calendar folder permissions | Admin Workstation | `Get-MailboxFolderPermission -Identity "<mailbox>:\Calendar"` | Current calendar permissions are saved |
| 8 | Export forwarding settings | Admin Workstation | `Get-Mailbox -Identity "<mailbox>" \| Format-List ForwardingAddress,ForwardingSmtpAddress,DeliverToMailboxAndForward` | Current forwarding state is saved |
| 9 | Export shared mailbox Sent Items settings | Admin Workstation | `Get-Mailbox -Identity "<shared-mailbox>" \| Format-List MessageCopyForSentAsEnabled,MessageCopyForSendOnBehalfEnabled` | Current sent item copy settings are saved |
| 10 | Export mailbox audit settings | Admin Workstation | `Get-Mailbox -Identity "<mailbox>" \| Format-List Audit*` | Current audit state is saved |
| 11 | Grant Full Access if required | Admin Workstation | `Add-MailboxPermission -Identity "<mailbox>" -User "<delegate>" -AccessRights FullAccess` | Delegate has Full Access |
| 12 | Validate Full Access | Admin Workstation | `Get-MailboxPermission -Identity "<mailbox>"` | Delegate permission appears |
| 13 | Grant Send As if required | Admin Workstation | `Add-RecipientPermission -Identity "<mailbox>" -Trustee "<delegate>" -AccessRights SendAs` | Delegate has Send As |
| 14 | Validate Send As | Admin Workstation | `Get-RecipientPermission -Identity "<mailbox>"` | Delegate Send As permission appears |
| 15 | Configure Send on Behalf if required | Admin Workstation | `Set-Mailbox -Identity "<mailbox>" -GrantSendOnBehalfTo @{Add="<delegate>"}` | Delegate has Send on Behalf |
| 16 | Validate Send on Behalf | Admin Workstation | `Get-Mailbox -Identity "<mailbox>" \| Format-List GrantSendOnBehalfTo` | Delegate appears |
| 17 | Configure calendar folder permission if required | Admin Workstation | `Add-MailboxFolderPermission -Identity "<mailbox>:\Calendar" -User "<delegate>" -AccessRights Reviewer` | Calendar permission is applied |
| 18 | Configure mailbox forwarding if required | Admin Workstation | `Set-Mailbox -Identity "<mailbox>" -ForwardingAddress "<recipient>" -DeliverToMailboxAndForward $true` | Forwarding is enabled |
| 19 | Configure shared mailbox Sent Items copy if required | Admin Workstation | `Set-Mailbox -Identity "<shared-mailbox>" -MessageCopyForSentAsEnabled $true -MessageCopyForSendOnBehalfEnabled $true` | Shared mailbox keeps sent item copies |
| 20 | Confirm mailbox audit is enabled | Admin Workstation | `Set-Mailbox -Identity "<mailbox>" -AuditEnabled $true` | Mailbox auditing is enabled |
| 21 | Confirm audit bypass is not enabled for delegate | Admin Workstation | `Get-MailboxAuditBypassAssociation -Identity "<delegate>"` | AuditBypassEnabled is false |
| 22 | Validate in Outlook or OWA | Browser / Outlook | Open mailbox, send test messages, verify calendar access | Expected delegated access works |
| 23 | Run audit validation | Admin Workstation | `Search-UnifiedAuditLog` | Relevant mailbox actions appear after audit delay |
| 24 | Export final permission and audit state | Admin Workstation | Run final inventory skeleton | Final evidence is saved |
| 25 | Disconnect Exchange Online session | Admin Workstation | `Disconnect-ExchangeOnline -Confirm:$false` | Session closes cleanly |

# Configure_Mailbox_Permissions_Delegation_Forwarding_And_Auditing_Precheck_Skeleton

```powershell
# Run on the admin workstation.
# Purpose: capture target mailbox, delegate, permission, forwarding, and audit state before changes.

$AdminUpn = "admin@contoso.com"
$EvidenceRoot = "C:\M365-Exchange-Baseline"
$RunStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$EvidencePath = Join-Path $EvidenceRoot "05-Mailbox-Permissions-Precheck-$RunStamp"

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\mailbox-permissions-precheck-transcript.txt"

Import-Module ExchangeOnlineManagement

try {
  $Connection = Get-ConnectionInformation -ErrorAction Stop
}
catch {
  Connect-ExchangeOnline -UserPrincipalName $AdminUpn -ShowBanner:$false
}

# Variables.
$TargetMailbox = "helpdesk@contoso.com"
$DelegateUser = "adele.vance@contoso.com"
$SharedMailbox = "helpdesk@contoso.com"

# Connection state.
Get-ConnectionInformation |
  Tee-Object "$EvidencePath\connection-information.txt"

# Validate target mailbox.
Get-EXOMailbox -Identity $TargetMailbox |
  Format-List DisplayName,UserPrincipalName,PrimarySmtpAddress,RecipientTypeDetails,HiddenFromAddressListsEnabled |
  Out-File "$EvidencePath\target-mailbox.txt"

Get-Mailbox -Identity $TargetMailbox |
  Format-List DisplayName,PrimarySmtpAddress,RecipientTypeDetails,ForwardingAddress,ForwardingSmtpAddress,DeliverToMailboxAndForward,GrantSendOnBehalfTo,MessageCopyForSentAsEnabled,MessageCopyForSendOnBehalfEnabled,AuditEnabled,AuditAdmin,AuditDelegate,AuditOwner |
  Out-File "$EvidencePath\target-mailbox-full-baseline.txt"

# Validate delegate.
Get-Recipient -Identity $DelegateUser |
  Format-List DisplayName,PrimarySmtpAddress,RecipientTypeDetails |
  Out-File "$EvidencePath\delegate-recipient.txt"

# Existing Full Access permissions.
Get-MailboxPermission -Identity $TargetMailbox |
  Where-Object {$_.IsInherited -eq $false} |
  Select-Object Identity,User,AccessRights,Deny,IsInherited |
  Export-Csv "$EvidencePath\mailbox-permissions-before.csv" -NoTypeInformation

# Existing Send As permissions.
Get-RecipientPermission -Identity $TargetMailbox |
  Select-Object Identity,Trustee,AccessRights,IsInherited,Deny |
  Export-Csv "$EvidencePath\recipient-permissions-before.csv" -NoTypeInformation

# Existing calendar permissions.
Get-MailboxFolderPermission -Identity "$TargetMailbox`:\Calendar" |
  Select-Object FolderName,User,AccessRights,SharingPermissionFlags |
  Export-Csv "$EvidencePath\calendar-permissions-before.csv" -NoTypeInformation

# Existing forwarding and sent item copy settings.
Get-Mailbox -Identity $TargetMailbox |
  Select-Object DisplayName,PrimarySmtpAddress,ForwardingAddress,ForwardingSmtpAddress,DeliverToMailboxAndForward,MessageCopyForSentAsEnabled,MessageCopyForSendOnBehalfEnabled |
  Export-Csv "$EvidencePath\forwarding-and-sentitems-before.csv" -NoTypeInformation

# Existing audit settings.
Get-Mailbox -Identity $TargetMailbox |
  Select-Object DisplayName,PrimarySmtpAddress,AuditEnabled,AuditAdmin,AuditDelegate,AuditOwner |
  Export-Csv "$EvidencePath\audit-settings-before.csv" -NoTypeInformation

# Audit bypass check for delegate.
Get-MailboxAuditBypassAssociation -Identity $DelegateUser |
  Format-List Identity,AuditBypassEnabled |
  Out-File "$EvidencePath\delegate-audit-bypass-before.txt"

Stop-Transcript
```

# Configure_Mailbox_Permissions_Delegation_Forwarding_And_Auditing_Full_Access_Skeleton

```powershell
# Run on the admin workstation.
# Purpose: grant and validate Full Access mailbox permission.
# Full Access lets the delegate open and manage mailbox content.
# Full Access does not grant Send As or Send on Behalf.

$EvidenceRoot = "C:\M365-Exchange-Baseline"
$RunStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$EvidencePath = Join-Path $EvidenceRoot "05-Full-Access-$RunStamp"

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\full-access-transcript.txt"

# Variables.
$TargetMailbox = "helpdesk@contoso.com"
$DelegateUser = "adele.vance@contoso.com"
$AutoMapping = $true

# Capture before state.
Get-MailboxPermission -Identity $TargetMailbox |
  Where-Object {$_.IsInherited -eq $false} |
  Select-Object Identity,User,AccessRights,Deny,IsInherited |
  Export-Csv "$EvidencePath\mailbox-permissions-before.csv" -NoTypeInformation

# Grant Full Access.
if ($AutoMapping -eq $true) {
  Add-MailboxPermission `
    -Identity $TargetMailbox `
    -User $DelegateUser `
    -AccessRights FullAccess `
    -InheritanceType All `
    -AutoMapping $true
}
else {
  Add-MailboxPermission `
    -Identity $TargetMailbox `
    -User $DelegateUser `
    -AccessRights FullAccess `
    -InheritanceType All `
    -AutoMapping $false
}

# Validate.
Get-MailboxPermission -Identity $TargetMailbox |
  Where-Object {
    $_.User -like "*$DelegateUser*" -or
    $_.User.ToString() -eq $DelegateUser
  } |
  Select-Object Identity,User,AccessRights,Deny,IsInherited |
  Tee-Object "$EvidencePath\full-access-validation.txt"

# Capture after state.
Get-MailboxPermission -Identity $TargetMailbox |
  Where-Object {$_.IsInherited -eq $false} |
  Select-Object Identity,User,AccessRights,Deny,IsInherited |
  Export-Csv "$EvidencePath\mailbox-permissions-after.csv" -NoTypeInformation

Stop-Transcript
```

# Configure_Mailbox_Permissions_Delegation_Forwarding_And_Auditing_Send_As_Skeleton

```powershell
# Run on the admin workstation.
# Purpose: grant and validate Send As permission.
# Send As lets the delegate send as the mailbox identity.

$EvidenceRoot = "C:\M365-Exchange-Baseline"
$RunStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$EvidencePath = Join-Path $EvidenceRoot "05-Send-As-$RunStamp"

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\send-as-transcript.txt"

# Variables.
$TargetMailbox = "helpdesk@contoso.com"
$DelegateUser = "adele.vance@contoso.com"

# Capture before state.
Get-RecipientPermission -Identity $TargetMailbox |
  Select-Object Identity,Trustee,AccessRights,IsInherited,Deny |
  Export-Csv "$EvidencePath\recipient-permissions-before.csv" -NoTypeInformation

# Grant Send As.
Add-RecipientPermission `
  -Identity $TargetMailbox `
  -Trustee $DelegateUser `
  -AccessRights SendAs `
  -Confirm:$false

# Validate.
Get-RecipientPermission -Identity $TargetMailbox |
  Where-Object {$_.Trustee -like "*$DelegateUser*" -or $_.Trustee.ToString() -eq $DelegateUser} |
  Select-Object Identity,Trustee,AccessRights,IsInherited,Deny |
  Tee-Object "$EvidencePath\send-as-validation.txt"

# Capture after state.
Get-RecipientPermission -Identity $TargetMailbox |
  Select-Object Identity,Trustee,AccessRights,IsInherited,Deny |
  Export-Csv "$EvidencePath\recipient-permissions-after.csv" -NoTypeInformation

Stop-Transcript
```

# Configure_Mailbox_Permissions_Delegation_Forwarding_And_Auditing_Send_On_Behalf_Skeleton

```powershell
# Run on the admin workstation.
# Purpose: grant and validate Send on Behalf permission.
# Send on Behalf shows messages as sent by the delegate on behalf of the mailbox.

$EvidenceRoot = "C:\M365-Exchange-Baseline"
$RunStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$EvidencePath = Join-Path $EvidenceRoot "05-Send-On-Behalf-$RunStamp"

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\send-on-behalf-transcript.txt"

# Variables.
$TargetMailbox = "helpdesk@contoso.com"
$DelegateUser = "adele.vance@contoso.com"

# Capture before state.
Get-Mailbox -Identity $TargetMailbox |
  Format-List DisplayName,PrimarySmtpAddress,GrantSendOnBehalfTo |
  Out-File "$EvidencePath\send-on-behalf-before.txt"

# Add delegate to Send on Behalf list without overwriting existing delegates.
Set-Mailbox -Identity $TargetMailbox `
  -GrantSendOnBehalfTo @{Add=$DelegateUser}

# Validate.
Get-Mailbox -Identity $TargetMailbox |
  Format-List DisplayName,PrimarySmtpAddress,GrantSendOnBehalfTo |
  Out-File "$EvidencePath\send-on-behalf-after.txt"

Stop-Transcript
```

# Configure_Mailbox_Permissions_Delegation_Forwarding_And_Auditing_Calendar_Folder_Permission_Skeleton

```powershell
# Run on the admin workstation.
# Purpose: configure mailbox folder permission for Calendar access.
# Folder permissions are separate from Full Access and Send permissions.

$EvidenceRoot = "C:\M365-Exchange-Baseline"
$RunStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$EvidencePath = Join-Path $EvidenceRoot "05-Calendar-Folder-Permission-$RunStamp"

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\calendar-folder-permission-transcript.txt"

# Variables.
$TargetMailbox = "alex.wilber@contoso.com"
$DelegateUser = "adele.vance@contoso.com"
$CalendarIdentity = "$TargetMailbox`:\Calendar"
$AccessRights = "Reviewer"

# Common calendar AccessRights examples:
# AvailabilityOnly
# LimitedDetails
# Reviewer
# Editor
# PublishingEditor
# Owner

# Capture before state.
Get-MailboxFolderPermission -Identity $CalendarIdentity |
  Select-Object FolderName,User,AccessRights,SharingPermissionFlags |
  Export-Csv "$EvidencePath\calendar-permissions-before.csv" -NoTypeInformation

# Add or update calendar permission.
$ExistingPermission = Get-MailboxFolderPermission -Identity $CalendarIdentity -User $DelegateUser -ErrorAction SilentlyContinue

if ($ExistingPermission) {
  Set-MailboxFolderPermission `
    -Identity $CalendarIdentity `
    -User $DelegateUser `
    -AccessRights $AccessRights
}
else {
  Add-MailboxFolderPermission `
    -Identity $CalendarIdentity `
    -User $DelegateUser `
    -AccessRights $AccessRights
}

# Validate.
Get-MailboxFolderPermission -Identity $CalendarIdentity |
  Select-Object FolderName,User,AccessRights,SharingPermissionFlags |
  Export-Csv "$EvidencePath\calendar-permissions-after.csv" -NoTypeInformation

Get-MailboxFolderPermission -Identity $CalendarIdentity -User $DelegateUser |
  Format-List FolderName,User,AccessRights,SharingPermissionFlags |
  Out-File "$EvidencePath\calendar-permission-delegate-validation.txt"

Stop-Transcript
```

# Configure_Mailbox_Permissions_Delegation_Forwarding_And_Auditing_Forwarding_Skeleton

```powershell
# Run on the admin workstation.
# Purpose: configure and validate mailbox forwarding.
# Treat forwarding as a data movement control. External forwarding should require approval.

$EvidenceRoot = "C:\M365-Exchange-Baseline"
$RunStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$EvidencePath = Join-Path $EvidenceRoot "05-Mailbox-Forwarding-$RunStamp"

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\mailbox-forwarding-transcript.txt"

# Variables.
$TargetMailbox = "alex.wilber@contoso.com"
$InternalForwardingRecipient = "adele.vance@contoso.com"
$ExternalForwardingSmtpAddress = "external.archive@example.com"
$UseInternalForwarding = $true
$UseExternalForwarding = $false
$DeliverToMailboxAndForward = $true

# Capture before state.
Get-Mailbox -Identity $TargetMailbox |
  Format-List DisplayName,PrimarySmtpAddress,ForwardingAddress,ForwardingSmtpAddress,DeliverToMailboxAndForward |
  Out-File "$EvidencePath\forwarding-before.txt"

# Internal forwarding target must be an Exchange recipient.
if ($UseInternalForwarding -eq $true) {
  Get-Recipient -Identity $InternalForwardingRecipient |
    Format-List DisplayName,PrimarySmtpAddress,RecipientTypeDetails |
    Out-File "$EvidencePath\internal-forwarding-recipient-validation.txt"

  Set-Mailbox -Identity $TargetMailbox `
    -ForwardingAddress $InternalForwardingRecipient `
    -ForwardingSmtpAddress $null `
    -DeliverToMailboxAndForward $DeliverToMailboxAndForward
}

# External forwarding target is a raw SMTP address.
# Only enable after approval and after outbound anti-spam policy allows it.
if ($UseExternalForwarding -eq $true) {
  Set-Mailbox -Identity $TargetMailbox `
    -ForwardingAddress $null `
    -ForwardingSmtpAddress $ExternalForwardingSmtpAddress `
    -DeliverToMailboxAndForward $DeliverToMailboxAndForward
}

# Validate after state.
Get-Mailbox -Identity $TargetMailbox |
  Format-List DisplayName,PrimarySmtpAddress,ForwardingAddress,ForwardingSmtpAddress,DeliverToMailboxAndForward |
  Out-File "$EvidencePath\forwarding-after.txt"

Stop-Transcript
```

# Configure_Mailbox_Permissions_Delegation_Forwarding_And_Auditing_Shared_Mailbox_Sent_Items_Skeleton

```powershell
# Run on the admin workstation.
# Purpose: configure shared mailbox Sent Items copy behavior for delegated sending.
# This helps keep records of messages sent as or on behalf of the shared mailbox.

$EvidenceRoot = "C:\M365-Exchange-Baseline"
$RunStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$EvidencePath = Join-Path $EvidenceRoot "05-Shared-Mailbox-Sent-Items-$RunStamp"

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\shared-mailbox-sent-items-transcript.txt"

# Variables.
$SharedMailbox = "helpdesk@contoso.com"

# Validate shared mailbox.
Get-EXOMailbox -Identity $SharedMailbox |
  Format-List DisplayName,PrimarySmtpAddress,RecipientTypeDetails |
  Out-File "$EvidencePath\shared-mailbox-validation.txt"

# Capture before state.
Get-Mailbox -Identity $SharedMailbox |
  Format-List DisplayName,PrimarySmtpAddress,MessageCopyForSentAsEnabled,MessageCopyForSendOnBehalfEnabled |
  Out-File "$EvidencePath\shared-mailbox-sent-items-before.txt"

# Configure Sent Items copy behavior.
Set-Mailbox -Identity $SharedMailbox `
  -MessageCopyForSentAsEnabled $true `
  -MessageCopyForSendOnBehalfEnabled $true

# Validate after state.
Get-Mailbox -Identity $SharedMailbox |
  Format-List DisplayName,PrimarySmtpAddress,MessageCopyForSentAsEnabled,MessageCopyForSendOnBehalfEnabled |
  Out-File "$EvidencePath\shared-mailbox-sent-items-after.txt"

Stop-Transcript
```

# Configure_Mailbox_Permissions_Delegation_Forwarding_And_Auditing_Audit_Baseline_Skeleton

```powershell
# Run on the admin workstation.
# Purpose: capture and configure mailbox auditing baseline.
# Exchange Online mailbox auditing is commonly enabled by default, but this workbook still verifies the explicit mailbox state.

$EvidenceRoot = "C:\M365-Exchange-Baseline"
$RunStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$EvidencePath = Join-Path $EvidenceRoot "05-Mailbox-Audit-Baseline-$RunStamp"

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\mailbox-audit-baseline-transcript.txt"

# Variables.
$TargetMailbox = "helpdesk@contoso.com"
$DelegateUser = "adele.vance@contoso.com"

# Capture audit state before.
Get-Mailbox -Identity $TargetMailbox |
  Format-List DisplayName,PrimarySmtpAddress,AuditEnabled,AuditAdmin,AuditDelegate,AuditOwner,AuditLogAgeLimit |
  Out-File "$EvidencePath\mailbox-audit-before.txt"

# Confirm delegate is not bypassing mailbox audit.
Get-MailboxAuditBypassAssociation -Identity $DelegateUser |
  Format-List Identity,AuditBypassEnabled |
  Out-File "$EvidencePath\delegate-audit-bypass-before.txt"

# Enable mailbox auditing if not already enabled.
Set-Mailbox -Identity $TargetMailbox `
  -AuditEnabled $true

# Optional: define explicit audit action sets.
# Validate tenant policy before overriding defaults.
# Example conservative baseline:
Set-Mailbox -Identity $TargetMailbox `
  -AuditAdmin Update,MoveToDeletedItems,SoftDelete,HardDelete,SendAs,SendOnBehalf,Create,UpdateFolderPermissions `
  -AuditDelegate Update,MoveToDeletedItems,SoftDelete,HardDelete,SendAs,SendOnBehalf,Create,UpdateFolderPermissions `
  -AuditOwner Update,MoveToDeletedItems,SoftDelete,HardDelete,Create,MailboxLogin,UpdateFolderPermissions

# Ensure delegate audit bypass is disabled.
Set-MailboxAuditBypassAssociation -Identity $DelegateUser `
  -AuditBypassEnabled $false

# Capture audit state after.
Get-Mailbox -Identity $TargetMailbox |
  Format-List DisplayName,PrimarySmtpAddress,AuditEnabled,AuditAdmin,AuditDelegate,AuditOwner,AuditLogAgeLimit |
  Out-File "$EvidencePath\mailbox-audit-after.txt"

Get-MailboxAuditBypassAssociation -Identity $DelegateUser |
  Format-List Identity,AuditBypassEnabled |
  Out-File "$EvidencePath\delegate-audit-bypass-after.txt"

Stop-Transcript
```

# Configure_Mailbox_Permissions_Delegation_Forwarding_And_Auditing_Bulk_Inventory_Skeleton

```powershell
# Run on the admin workstation.
# Purpose: export mailbox permissions, forwarding, folder permissions, sent items behavior, and audit baseline.

$EvidenceRoot = "C:\M365-Exchange-Baseline"
$RunStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$EvidencePath = Join-Path $EvidenceRoot "05-Mailbox-Permissions-Inventory-$RunStamp"

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\mailbox-permissions-inventory-transcript.txt"

# Scope. Adjust for full tenant or lab mailbox list.
$MailboxScope = Get-EXOMailbox -ResultSize 200

# Mailbox forwarding and audit summary.
$MailboxScope |
  ForEach-Object {
    Get-Mailbox -Identity $_.PrimarySmtpAddress |
      Select-Object DisplayName,PrimarySmtpAddress,RecipientTypeDetails,ForwardingAddress,ForwardingSmtpAddress,DeliverToMailboxAndForward,GrantSendOnBehalfTo,MessageCopyForSentAsEnabled,MessageCopyForSendOnBehalfEnabled,AuditEnabled
  } |
  Export-Csv "$EvidencePath\mailbox-forwarding-delegation-audit-summary.csv" -NoTypeInformation

# Full Access permissions.
$FullAccessInventory = foreach ($Mailbox in $MailboxScope) {
  Get-MailboxPermission -Identity $Mailbox.PrimarySmtpAddress |
    Where-Object {$_.IsInherited -eq $false} |
    Select-Object @{Name="Mailbox";Expression={$Mailbox.PrimarySmtpAddress}},User,AccessRights,Deny,IsInherited
}

$FullAccessInventory |
  Export-Csv "$EvidencePath\full-access-permissions.csv" -NoTypeInformation

# Send As permissions.
$SendAsInventory = foreach ($Mailbox in $MailboxScope) {
  Get-RecipientPermission -Identity $Mailbox.PrimarySmtpAddress -ErrorAction SilentlyContinue |
    Where-Object {$_.IsInherited -eq $false} |
    Select-Object @{Name="Mailbox";Expression={$Mailbox.PrimarySmtpAddress}},Trustee,AccessRights,Deny,IsInherited
}

$SendAsInventory |
  Export-Csv "$EvidencePath\send-as-permissions.csv" -NoTypeInformation

# Calendar permissions.
$CalendarPermissionInventory = foreach ($Mailbox in $MailboxScope) {
  $CalendarIdentity = "$($Mailbox.PrimarySmtpAddress)`:\Calendar"

  Get-MailboxFolderPermission -Identity $CalendarIdentity -ErrorAction SilentlyContinue |
    Select-Object @{Name="Mailbox";Expression={$Mailbox.PrimarySmtpAddress}},FolderName,User,AccessRights,SharingPermissionFlags
}

$CalendarPermissionInventory |
  Export-Csv "$EvidencePath\calendar-folder-permissions.csv" -NoTypeInformation

# Mailboxes with forwarding.
$MailboxScope |
  ForEach-Object {
    Get-Mailbox -Identity $_.PrimarySmtpAddress |
      Where-Object {$_.ForwardingAddress -or $_.ForwardingSmtpAddress} |
      Select-Object DisplayName,PrimarySmtpAddress,ForwardingAddress,ForwardingSmtpAddress,DeliverToMailboxAndForward
  } |
  Export-Csv "$EvidencePath\mailboxes-with-forwarding.csv" -NoTypeInformation

# Mailboxes with Send on Behalf delegates.
$MailboxScope |
  ForEach-Object {
    Get-Mailbox -Identity $_.PrimarySmtpAddress |
      Where-Object {$_.GrantSendOnBehalfTo -ne $null} |
      Select-Object DisplayName,PrimarySmtpAddress,GrantSendOnBehalfTo
  } |
  Export-Csv "$EvidencePath\mailboxes-with-send-on-behalf.csv" -NoTypeInformation

Stop-Transcript
```

# Configure_Mailbox_Permissions_Delegation_Forwarding_And_Auditing_Validation_Skeleton

```powershell
# Run on the admin workstation.
# Purpose: validate permissions, forwarding, sent item copy behavior, and audit search readiness.

$EvidenceRoot = "C:\M365-Exchange-Baseline"
$RunStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$EvidencePath = Join-Path $EvidenceRoot "05-Mailbox-Permissions-Validation-$RunStamp"

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\mailbox-permissions-validation-transcript.txt"

# Variables.
$TargetMailbox = "helpdesk@contoso.com"
$UserMailbox = "alex.wilber@contoso.com"
$SharedMailbox = "helpdesk@contoso.com"
$DelegateUser = "adele.vance@contoso.com"
$CalendarIdentity = "$UserMailbox`:\Calendar"
$AuditStart = (Get-Date).AddDays(-7)
$AuditEnd = Get-Date

# Validate Full Access.
Get-MailboxPermission -Identity $TargetMailbox |
  Where-Object {
    $_.IsInherited -eq $false -and
    ($_.User -like "*$DelegateUser*" -or $_.User.ToString() -eq $DelegateUser)
  } |
  Select-Object Identity,User,AccessRights,Deny,IsInherited |
  Tee-Object "$EvidencePath\validate-full-access.txt"

# Validate Send As.
Get-RecipientPermission -Identity $TargetMailbox |
  Where-Object {
    $_.IsInherited -eq $false -and
    ($_.Trustee -like "*$DelegateUser*" -or $_.Trustee.ToString() -eq $DelegateUser)
  } |
  Select-Object Identity,Trustee,AccessRights,Deny,IsInherited |
  Tee-Object "$EvidencePath\validate-send-as.txt"

# Validate Send on Behalf.
Get-Mailbox -Identity $TargetMailbox |
  Format-List DisplayName,PrimarySmtpAddress,GrantSendOnBehalfTo |
  Out-File "$EvidencePath\validate-send-on-behalf.txt"

# Validate calendar permission.
Get-MailboxFolderPermission -Identity $CalendarIdentity -User $DelegateUser -ErrorAction SilentlyContinue |
  Format-List FolderName,User,AccessRights,SharingPermissionFlags |
  Out-File "$EvidencePath\validate-calendar-permission.txt"

# Validate forwarding.
Get-Mailbox -Identity $UserMailbox |
  Format-List DisplayName,PrimarySmtpAddress,ForwardingAddress,ForwardingSmtpAddress,DeliverToMailboxAndForward |
  Out-File "$EvidencePath\validate-forwarding.txt"

# Validate shared mailbox Sent Items copy.
Get-Mailbox -Identity $SharedMailbox |
  Format-List DisplayName,PrimarySmtpAddress,MessageCopyForSentAsEnabled,MessageCopyForSendOnBehalfEnabled |
  Out-File "$EvidencePath\validate-shared-mailbox-sent-items.txt"

# Validate auditing.
Get-Mailbox -Identity $TargetMailbox |
  Format-List DisplayName,PrimarySmtpAddress,AuditEnabled,AuditAdmin,AuditDelegate,AuditOwner |
  Out-File "$EvidencePath\validate-mailbox-audit-settings.txt"

Get-MailboxAuditBypassAssociation -Identity $DelegateUser |
  Format-List Identity,AuditBypassEnabled |
  Out-File "$EvidencePath\validate-delegate-audit-bypass.txt"

# Unified audit log sample.
# Results may not appear immediately after test activity.
Search-UnifiedAuditLog `
  -StartDate $AuditStart `
  -EndDate $AuditEnd `
  -UserIds $DelegateUser `
  -ResultSize 100 |
  Select-Object CreationDate,UserIds,Operations,AuditData |
  Export-Csv "$EvidencePath\validate-unified-audit-log-sample.csv" -NoTypeInformation

Stop-Transcript
```

# Configure_Mailbox_Permissions_Delegation_Forwarding_And_Auditing_Verification_Commands

| Check | Device | PowerShell / Command | Healthy Output |
|---|---|---|---|
| Exchange Online session active | Admin Workstation | `Get-ConnectionInformation` | Active session exists |
| Target mailbox exists | Admin Workstation | `Get-EXOMailbox -Identity "<mailbox>"` | Mailbox returns |
| Delegate exists | Admin Workstation | `Get-Recipient -Identity "<delegate>"` | Delegate returns |
| Full Access present | Admin Workstation | `Get-MailboxPermission -Identity "<mailbox>"` | Delegate has FullAccess |
| Send As present | Admin Workstation | `Get-RecipientPermission -Identity "<mailbox>"` | Delegate has SendAs |
| Send on Behalf present | Admin Workstation | `Get-Mailbox -Identity "<mailbox>" \| Format-List GrantSendOnBehalfTo` | Delegate appears in GrantSendOnBehalfTo |
| Calendar permission present | Admin Workstation | `Get-MailboxFolderPermission -Identity "<mailbox>:\Calendar" -User "<delegate>"` | Expected AccessRights appears |
| Forwarding disabled or correct | Admin Workstation | `Get-Mailbox -Identity "<mailbox>" \| Format-List ForwardingAddress,ForwardingSmtpAddress,DeliverToMailboxAndForward` | Forwarding matches approved design |
| Shared mailbox sent item copies enabled | Admin Workstation | `Get-Mailbox -Identity "<shared-mailbox>" \| Format-List MessageCopyForSentAsEnabled,MessageCopyForSendOnBehalfEnabled` | Both values match standard |
| Mailbox audit enabled | Admin Workstation | `Get-Mailbox -Identity "<mailbox>" \| Format-List AuditEnabled` | AuditEnabled is true |
| Audit action sets visible | Admin Workstation | `Get-Mailbox -Identity "<mailbox>" \| Format-List AuditAdmin,AuditDelegate,AuditOwner` | Audit action sets return |
| Audit bypass disabled | Admin Workstation | `Get-MailboxAuditBypassAssociation -Identity "<delegate>"` | AuditBypassEnabled is false |
| Delegate can open mailbox | Outlook / OWA | Open shared mailbox or delegated mailbox | Mailbox opens if Full Access is granted |
| Delegate can Send As | Outlook / OWA | Send message as mailbox | Recipient sees sender as mailbox |
| Delegate can Send on Behalf | Outlook / OWA | Send message on behalf of mailbox | Recipient sees on behalf relationship |
| Calendar access works | Outlook / OWA | Open delegated calendar | Calendar access matches role |
| Forwarded message test works | Outlook / OWA | Send test message to mailbox | Forwarding behavior matches copy setting |
| Audit search returns activity | Admin Workstation | `Search-UnifiedAuditLog -StartDate (Get-Date).AddDays(-7) -EndDate (Get-Date) -UserIds "<delegate>"` | Recent activity appears after audit delay |
| EAC delegation view | Browser | Exchange admin center > Recipients > Mailboxes > Delegation | Delegation matches PowerShell |
| EAC forwarding view | Browser | Exchange admin center > Recipients > Mailboxes > Mailbox features / Mail flow settings | Forwarding matches PowerShell |

# Configure_Mailbox_Permissions_Delegation_Forwarding_And_Auditing_Rollback

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm rollback scope | Operator | Review mailbox, delegate, forwarding, and audit changes | Only lab changes are targeted |
| 2 | Export current state before rollback | Admin Workstation | Run inventory commands | Pre-rollback evidence is saved |
| 3 | Remove Full Access | Admin Workstation | `Remove-MailboxPermission -Identity "<mailbox>" -User "<delegate>" -AccessRights FullAccess -Confirm:$false` | Delegate no longer has Full Access |
| 4 | Remove Send As | Admin Workstation | `Remove-RecipientPermission -Identity "<mailbox>" -Trustee "<delegate>" -AccessRights SendAs -Confirm:$false` | Delegate no longer has Send As |
| 5 | Remove Send on Behalf | Admin Workstation | `Set-Mailbox -Identity "<mailbox>" -GrantSendOnBehalfTo @{Remove="<delegate>"}` | Delegate removed from Send on Behalf |
| 6 | Remove calendar permission | Admin Workstation | `Remove-MailboxFolderPermission -Identity "<mailbox>:\Calendar" -User "<delegate>" -Confirm:$false` | Delegate folder permission removed |
| 7 | Clear forwarding | Admin Workstation | `Set-Mailbox -Identity "<mailbox>" -ForwardingAddress $null -ForwardingSmtpAddress $null -DeliverToMailboxAndForward $false` | Forwarding disabled |
| 8 | Revert shared mailbox sent item copy if required | Admin Workstation | `Set-Mailbox -Identity "<shared-mailbox>" -MessageCopyForSentAsEnabled $false -MessageCopyForSendOnBehalfEnabled $false` | Sent Items copy behavior reverts |
| 9 | Revert audit action customization only if changed for lab | Admin Workstation | Use captured before state | Audit settings return to original state |
| 10 | Validate rollback | Admin Workstation | Run validation commands | Removed permissions no longer appear |
| 11 | Export final state | Admin Workstation | Run inventory commands again | Post-rollback evidence is saved |
| 12 | Disconnect session | Admin Workstation | `Disconnect-ExchangeOnline -Confirm:$false` | Session closes |

# Configure_Mailbox_Permissions_Delegation_Forwarding_And_Auditing_Rollback_Helper

```powershell
# Run on the admin workstation.
# Purpose: rollback lab-created mailbox permissions, forwarding, and shared mailbox sent item changes only.

$EvidenceRoot = "C:\M365-Exchange-Baseline"
$RunStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$EvidencePath = Join-Path $EvidenceRoot "05-Mailbox-Permissions-Rollback-$RunStamp"

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\mailbox-permissions-rollback-transcript.txt"

# Variables.
$TargetMailbox = "helpdesk@contoso.com"
$UserMailbox = "alex.wilber@contoso.com"
$SharedMailbox = "helpdesk@contoso.com"
$DelegateUser = "adele.vance@contoso.com"
$CalendarIdentity = "$UserMailbox`:\Calendar"

# Capture before rollback.
Get-MailboxPermission -Identity $TargetMailbox |
  Where-Object {$_.IsInherited -eq $false} |
  Select-Object Identity,User,AccessRights,Deny,IsInherited |
  Export-Csv "$EvidencePath\full-access-before-rollback.csv" -NoTypeInformation

Get-RecipientPermission -Identity $TargetMailbox |
  Select-Object Identity,Trustee,AccessRights,IsInherited,Deny |
  Export-Csv "$EvidencePath\send-as-before-rollback.csv" -NoTypeInformation

Get-Mailbox -Identity $TargetMailbox |
  Format-List DisplayName,PrimarySmtpAddress,GrantSendOnBehalfTo |
  Out-File "$EvidencePath\send-on-behalf-before-rollback.txt"

Get-MailboxFolderPermission -Identity $CalendarIdentity |
  Select-Object FolderName,User,AccessRights,SharingPermissionFlags |
  Export-Csv "$EvidencePath\calendar-permissions-before-rollback.csv" -NoTypeInformation

Get-Mailbox -Identity $UserMailbox |
  Format-List DisplayName,PrimarySmtpAddress,ForwardingAddress,ForwardingSmtpAddress,DeliverToMailboxAndForward |
  Out-File "$EvidencePath\forwarding-before-rollback.txt"

Get-Mailbox -Identity $SharedMailbox |
  Format-List DisplayName,PrimarySmtpAddress,MessageCopyForSentAsEnabled,MessageCopyForSendOnBehalfEnabled |
  Out-File "$EvidencePath\shared-mailbox-sent-items-before-rollback.txt"

# Remove Full Access.
Remove-MailboxPermission `
  -Identity $TargetMailbox `
  -User $DelegateUser `
  -AccessRights FullAccess `
  -Confirm:$false `
  -ErrorAction SilentlyContinue

# Remove Send As.
Remove-RecipientPermission `
  -Identity $TargetMailbox `
  -Trustee $DelegateUser `
  -AccessRights SendAs `
  -Confirm:$false `
  -ErrorAction SilentlyContinue

# Remove Send on Behalf.
Set-Mailbox -Identity $TargetMailbox `
  -GrantSendOnBehalfTo @{Remove=$DelegateUser} `
  -ErrorAction SilentlyContinue

# Remove calendar permission.
Remove-MailboxFolderPermission `
  -Identity $CalendarIdentity `
  -User $DelegateUser `
  -Confirm:$false `
  -ErrorAction SilentlyContinue

# Clear forwarding on the lab target mailbox.
Set-Mailbox -Identity $UserMailbox `
  -ForwardingAddress $null `
  -ForwardingSmtpAddress $null `
  -DeliverToMailboxAndForward $false `
  -ErrorAction SilentlyContinue

# Revert shared mailbox sent item copy only if the lab changed it.
# Comment out if production standard requires these values to remain true.
Set-Mailbox -Identity $SharedMailbox `
  -MessageCopyForSentAsEnabled $false `
  -MessageCopyForSendOnBehalfEnabled $false `
  -ErrorAction SilentlyContinue

# Capture after rollback.
Get-MailboxPermission -Identity $TargetMailbox |
  Where-Object {$_.IsInherited -eq $false} |
  Select-Object Identity,User,AccessRights,Deny,IsInherited |
  Export-Csv "$EvidencePath\full-access-after-rollback.csv" -NoTypeInformation

Get-RecipientPermission -Identity $TargetMailbox |
  Select-Object Identity,Trustee,AccessRights,IsInherited,Deny |
  Export-Csv "$EvidencePath\send-as-after-rollback.csv" -NoTypeInformation

Get-Mailbox -Identity $TargetMailbox |
  Format-List DisplayName,PrimarySmtpAddress,GrantSendOnBehalfTo |
  Out-File "$EvidencePath\send-on-behalf-after-rollback.txt"

Get-MailboxFolderPermission -Identity $CalendarIdentity |
  Select-Object FolderName,User,AccessRights,SharingPermissionFlags |
  Export-Csv "$EvidencePath\calendar-permissions-after-rollback.csv" -NoTypeInformation

Get-Mailbox -Identity $UserMailbox |
  Format-List DisplayName,PrimarySmtpAddress,ForwardingAddress,ForwardingSmtpAddress,DeliverToMailboxAndForward |
  Out-File "$EvidencePath\forwarding-after-rollback.txt"

Stop-Transcript
```

# Configure_Mailbox_Permissions_Delegation_Forwarding_And_Auditing_Failure_Checks

| Symptom | Likely Cause | Check Command | Fix |
|---|---|---|---|
| Delegate cannot open mailbox | Full Access missing, propagation delay, Outlook cache, or auto-mapping issue | `Get-MailboxPermission -Identity "<mailbox>"` | Confirm Full Access, wait, reopen Outlook, or add mailbox manually |
| Delegate can open mailbox but cannot send as | Full Access does not include Send As | `Get-RecipientPermission -Identity "<mailbox>"` | Add Send As permission |
| Delegate sends on behalf instead of Send As | Send on Behalf configured or client selected wrong From identity | `Get-Mailbox -Identity "<mailbox>" \| Format-List GrantSendOnBehalfTo`; `Get-RecipientPermission` | Remove incorrect delegation or select correct From address |
| Send As fails after permission grant | Permission propagation delay or Outlook cached token | `Get-RecipientPermission -Identity "<mailbox>"` | Wait for propagation and restart client session |
| Send on Behalf overwrote existing delegates | Command used direct assignment instead of hash add syntax | `Get-Mailbox -Identity "<mailbox>" \| Format-List GrantSendOnBehalfTo` | Restore previous delegates from evidence and use `@{Add=...}` |
| Calendar permission fails | Wrong folder identity or localized calendar folder name | `Get-MailboxFolderStatistics -Identity "<mailbox>" -FolderScope Calendar` | Use the correct folder path |
| Calendar delegate sees too much or too little | Wrong AccessRights role | `Get-MailboxFolderPermission -Identity "<mailbox>:\Calendar"` | Set correct folder permission role |
| Forwarding does not work | External forwarding blocked, wrong forwarding target, or mail flow policy blocks it | `Get-Mailbox -Identity "<mailbox>" \| Format-List Forwarding*`; message trace | Use internal forwarding or approve and allow external forwarding policy |
| Forwarding works but mailbox copy missing | DeliverToMailboxAndForward is false | `Get-Mailbox -Identity "<mailbox>" \| Format-List DeliverToMailboxAndForward` | Set `DeliverToMailboxAndForward $true` |
| Forwarding to external address creates security concern | Data exfiltration risk | Review forwarding target and outbound policy | Disable external forwarding or require approved exception |
| Shared mailbox sent messages do not appear in shared Sent Items | Message copy settings disabled | `Get-Mailbox -Identity "<shared-mailbox>" \| Format-List MessageCopyForSentAsEnabled,MessageCopyForSendOnBehalfEnabled` | Enable both sent item copy settings |
| Mailbox audit actions do not appear immediately | Audit pipeline delay | `Search-UnifiedAuditLog` with wider window | Wait and search again |
| No audit entries for delegate | Audit bypass enabled or activity not audited | `Get-MailboxAuditBypassAssociation -Identity "<delegate>"`; `Get-Mailbox -Identity "<mailbox>" \| Format-List Audit*` | Disable audit bypass and validate audit action set |
| Cannot run Search-UnifiedAuditLog | Missing permissions or wrong PowerShell context | `Get-ManagementRoleAssignment -RoleAssignee "<admin-upn>"` | Use proper compliance or audit role |
| Remove Full Access fails | Permission assigned to group, inherited permission, or wrong delegate identity | `Get-MailboxPermission -Identity "<mailbox>"` | Remove the exact identity shown in output |
| Remove Send As fails | Trustee value does not match stored permission identity | `Get-RecipientPermission -Identity "<mailbox>"` | Use the exact Trustee value shown |
| Permission command fails on synced mailbox | Source of authority or RBAC limitation | Check mailbox and directory sync state | Manage cloud-owned permissions in Exchange, source-owned attributes on-premises |
| EAC and PowerShell disagree | Portal cache or propagation delay | Validate with PowerShell | Refresh portal or wait |
| Delegated mailbox appears twice in Outlook | Auto-mapping plus manual mailbox addition | Check Outlook profile and Full Access auto-mapping | Remove manual mapping or re-grant Full Access with AutoMapping false |

# Configure_Mailbox_Permissions_Delegation_Forwarding_And_Auditing_Related_Labs

| Related Lab                                                                                    | Relationship                                                                                            |
| ---------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------- |
| `01_Configure_Exchange_Admin_Center_PowerShell_And_Org_Baseline.md`                            | Provides Exchange admin access, PowerShell connection, and RBAC baseline                                |
| `02_Manage_User_Mailboxes_Shared_Mailboxes_Contacts_And_Resources.md`                          | Provides target mailboxes, shared mailboxes, and contacts used in delegation and forwarding scenarios   |
| `03_Manage_Distribution_Groups_Dynamic_Groups_Mail_Enabled_Security_Groups_And_M365_Groups.md` | Provides mail-enabled security groups and Microsoft 365 groups that may be used for delegation patterns |
| `04_Configure_Mail_Flow_Accepted_Domains_Connectors_And_Rules.md`                              | Provides mail routing awareness needed before forwarding and send permission testing                    |
| `06_Configure_Exchange_Online_Protection_AntiSpam_AntiMalware_And_Quarantine.md`               | Uses mailbox and forwarding context during protection and quarantine tests                              |
| `07_Configure_Retention_Archive_Litigation_Hold_And_eDiscovery_Handoff.md`                     | Depends on mailbox audit, access, and delegation evidence for compliance scenarios                      |
| `08_Troubleshoot_Mail_Flow_Mailbox_Access_Permissions_And_Quarantine_Issues.md`                | Uses delegation, forwarding, and audit evidence for troubleshooting                                     |