# Troubleshoot_Mail_Flow_Mailbox_Access_Permissions_And_Quarantine_Issues

# Troubleshoot_Mail_Flow_Mailbox_Access_Permissions_And_Quarantine_Issues_Index
08_Troubleshoot_Mail_Flow_Mailbox_Access_Permissions_And_Quarantine_Issues.md
Troubleshoot_Mail_Flow_Mailbox_Access_Permissions_And_Quarantine_Issues
Troubleshoot_Mail_Flow_Mailbox_Access_Permissions_And_Quarantine_Issues_Source_Basis
Troubleshoot_Mail_Flow_Mailbox_Access_Permissions_And_Quarantine_Issues_Mental_Model
Troubleshoot_Mail_Flow_Mailbox_Access_Permissions_And_Quarantine_Issues_Planning_Table
Troubleshoot_Mail_Flow_Mailbox_Access_Permissions_And_Quarantine_Issues_Symptom_Map
Troubleshoot_Mail_Flow_Mailbox_Access_Permissions_And_Quarantine_Issues_Configuration_Checklist
Troubleshoot_Mail_Flow_Mailbox_Access_Permissions_And_Quarantine_Issues_Intake_Skeleton
Troubleshoot_Mail_Flow_Mailbox_Access_Permissions_And_Quarantine_Issues_Mail_Flow_Trace_Skeleton
Troubleshoot_Mail_Flow_Mailbox_Access_Permissions_And_Quarantine_Issues_Mailbox_Access_Skeleton
Troubleshoot_Mail_Flow_Mailbox_Access_Permissions_And_Quarantine_Issues_Permissions_Delegation_Skeleton
Troubleshoot_Mail_Flow_Mailbox_Access_Permissions_And_Quarantine_Issues_Forwarding_And_Rule_Skeleton
Troubleshoot_Mail_Flow_Mailbox_Access_Permissions_And_Quarantine_Issues_Quarantine_Skeleton
Troubleshoot_Mail_Flow_Mailbox_Access_Permissions_And_Quarantine_Issues_EOP_Policy_Skeleton
Troubleshoot_Mail_Flow_Mailbox_Access_Permissions_And_Quarantine_Issues_Connector_And_Domain_Skeleton
Troubleshoot_Mail_Flow_Mailbox_Access_Permissions_And_Quarantine_Issues_Audit_And_Evidence_Skeleton
Troubleshoot_Mail_Flow_Mailbox_Access_Permissions_And_Quarantine_Issues_Remediation_Skeleton
Troubleshoot_Mail_Flow_Mailbox_Access_Permissions_And_Quarantine_Issues_Escalation_Package_Skeleton
Troubleshoot_Mail_Flow_Mailbox_Access_Permissions_And_Quarantine_Issues_Verification_Commands
Troubleshoot_Mail_Flow_Mailbox_Access_Permissions_And_Quarantine_Issues_Rollback
Troubleshoot_Mail_Flow_Mailbox_Access_Permissions_And_Quarantine_Issues_Failure_Checks
Troubleshoot_Mail_Flow_Mailbox_Access_Permissions_And_Quarantine_Issues_Related_Labs

# Troubleshoot_Mail_Flow_Mailbox_Access_Permissions_And_Quarantine_Issues_Source_Basis

| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Message trace in Exchange admin center | Tracing received, rejected, deferred, delivered, quarantined, and policy-acted messages |
| Microsoft Learn | Get-MessageTrace / Get-MessageTraceDetail / Get-MessageTraceV2 | PowerShell mail flow investigation |
| Microsoft Learn | Message trace report limits | Summary trace, enhanced summary, extended report, 10-day instant window, and 90-day historical search considerations |
| Microsoft Learn | Manage quarantined messages and files as an admin | Admin search, preview, release, delete, and submission workflow for quarantine |
| Microsoft Learn | Manage recipient permissions in Exchange Online | Full Access, Send As, Send on Behalf, and recipient permission validation |
| Microsoft Learn | Mail flow rules in Exchange Online | Transport rule condition, exception, action, mode, and priority troubleshooting |
| Microsoft Learn | Connectors in Exchange Online | Inbound and outbound connector validation |
| Microsoft Learn | Accepted domains in Exchange Online | Authoritative and internal relay domain troubleshooting |
| Microsoft Learn | Exchange Online mailbox auditing | Audit and Unified Audit Log checks for access, delegation, send, and admin operations |
| Prior workbook | `01_Configure_Exchange_Admin_Center_PowerShell_And_Org_Baseline.md` | Provides admin connection and baseline organization evidence |
| Prior workbook | `02_Manage_User_Mailboxes_Shared_Mailboxes_Contacts_And_Resources.md` | Provides recipient and mailbox baseline |
| Prior workbook | `03_Manage_Distribution_Groups_Dynamic_Groups_Mail_Enabled_Security_Groups_And_M365_Groups.md` | Provides group, ownership, delivery restriction, and membership baseline |
| Prior workbook | `04_Configure_Mail_Flow_Accepted_Domains_Connectors_And_Rules.md` | Provides accepted domain, connector, remote domain, and transport rule baseline |
| Prior workbook | `05_Configure_Mailbox_Permissions_Delegation_Forwarding_And_Auditing.md` | Provides mailbox delegation, forwarding, and audit baseline |
| Prior workbook | `06_Configure_Exchange_Online_Protection_AntiSpam_AntiMalware_And_Quarantine.md` | Provides EOP, anti-spam, anti-malware, quarantine, and allow/block baseline |
| Prior workbook | `07_Configure_Retention_Archive_Litigation_Hold_And_eDiscovery_Handoff.md` | Provides retention, hold, archive, and eDiscovery handoff baseline |

# Troubleshoot_Mail_Flow_Mailbox_Access_Permissions_And_Quarantine_Issues_Mental_Model

| Concept | Operational Meaning |
|---|---|
| Mail flow troubleshooting | Determine whether Exchange Online received, rejected, deferred, delivered, expanded, filtered, or quarantined a message |
| Message trace | Primary tool for tracing message status through Exchange Online |
| Message trace detail | Event-level message detail that explains transport actions |
| Network Message ID | Message identifier that can persist across message copies and transport processing |
| Message ID | Internet message ID from the message header |
| Delivery status | Delivered, failed, pending, expanded, quarantined, filtered as spam, or getting status |
| Deferred | Message delivery is delayed and retry is occurring |
| Rejected | Message was refused or blocked before successful delivery |
| Delivered | Message reached the destination mailbox or route |
| Quarantined | Message was held by EOP or Defender policy |
| Filtered as spam | Message was blocked or rejected by spam filtering instead of quarantined |
| Connector issue | Routing problem caused by inbound or outbound connector scope, smart host, TLS, sender IP, or recipient domain |
| Accepted domain issue | Mail domain does not exist, is wrong type, is unverified, or has DNS mismatch |
| Transport rule issue | Mail flow rule condition, exception, action, mode, or priority changes expected behavior |
| Mailbox access issue | User cannot open mailbox, shared mailbox, delegated mailbox, or calendar |
| Full Access | Permission to open mailbox content |
| Send As | Permission to send as the mailbox identity |
| Send on Behalf | Permission to send on behalf of the mailbox |
| Calendar folder permission | Folder-specific permission separate from mailbox-level delegation |
| Forwarding issue | Mailbox forwarding or inbox rules send mail to another target |
| Inbox rule issue | Client or server-side mailbox rule moves, deletes, forwards, or redirects messages |
| Quarantine issue | Message is held and user/admin cannot find, release, preview, or request release |
| EOP policy issue | Anti-spam, anti-malware, outbound spam, or quarantine policy causes expected or unexpected action |
| Audit evidence | Unified Audit Log and mailbox audit evidence proving access, send, forwarding, or admin changes |
| First rule | Establish message status before changing configuration |
| Second rule | Never assume the mailbox is the problem until message trace confirms delivery |
| Third rule | Never assume mail flow is the problem until access and folder placement are checked |
| Fourth rule | For quarantine, identify detection type and policy before release decisions |
| Fifth rule | Build an evidence package before remediation, then capture after-state proof |

# Troubleshoot_Mail_Flow_Mailbox_Access_Permissions_And_Quarantine_Issues_Planning_Table

| Item | Example | Decision |
|---|---|---|
| Tenant name | `contoso.onmicrosoft.com` | `<tenant-name>` |
| Admin UPN | `admin@contoso.com` | `<admin-upn>` |
| Evidence root | `C:\M365-Exchange-Baseline` | `<evidence-root>` |
| Incident ID | `INC-EXO-008` | `<incident-id>` |
| Reported issue type | Mail flow / Access / Permission / Quarantine | `<issue-type>` |
| Reported user | `alex.wilber@contoso.com` | `<reported-user>` |
| Sender | `sender@example.com` | `<sender>` |
| Recipient | `alex.wilber@contoso.com` | `<recipient>` |
| Shared mailbox | `helpdesk@contoso.com` | `<shared-mailbox>` |
| Delegate | `adele.vance@contoso.com` | `<delegate>` |
| Message subject | `Quarterly report` | `<subject>` |
| Message ID | `<message-id-header>` | `<message-id>` |
| Network Message ID | `<network-message-id>` | `<network-message-id>` |
| Approximate sent time | `2026-06-15 09:30` | `<sent-time>` |
| Time zone | `America/New_York` | `<time-zone>` |
| Search window start | `2026-06-15 00:00` | `<start-time>` |
| Search window end | `2026-06-15 23:59` | `<end-time>` |
| Direction | Inbound / Outbound / Internal | `<direction>` |
| Expected result | Delivered to Inbox / Open mailbox / Release quarantine | `<expected-result>` |
| Actual result | Missing / NDR / Access denied / Quarantined | `<actual-result>` |
| Priority | Low / Medium / High / Critical | `<priority>` |
| Business impact | One user / Team / Tenant-wide | `<impact>` |
| Change correlation | Recent policy, DNS, connector, license, permission, or role change | `<change-correlation>` |
| Remediation approved by | `manager@contoso.com` | `<approver>` |
| Rollback stance | Revert only the specific fix applied | `<rollback-scope>` |

# Troubleshoot_Mail_Flow_Mailbox_Access_Permissions_And_Quarantine_Issues_Symptom_Map

| Symptom | Primary Investigation Path | First Evidence Command | Likely Root Areas |
|---|---|---|---|
| Sender says message was sent but recipient never got it | Message trace first | `Get-MessageTrace -SenderAddress "<sender>" -RecipientAddress "<recipient>"` | Delivery failure, quarantine, rule, connector, DNS, spam filtering |
| Message delivered but user cannot find it | Mailbox folder and rule checks | `Get-MessageTrace`; `Get-InboxRule`; mailbox search in OWA | Inbox rule, junk folder, focused inbox, deleted items, client issue |
| Message bounced with NDR | NDR and trace detail | `Get-MessageTraceDetail` | Invalid recipient, connector failure, policy rejection, DNS |
| Message stuck pending or deferred | Message trace detail and connector check | `Get-MessageTraceDetail`; `Get-OutboundConnector` | Remote server, smart host, TLS, throttling, transient service issue |
| Distribution group did not deliver to member | Group expansion and membership check | `Get-DistributionGroupMember`; trace status Expanded | Membership, moderation, restrictions, dynamic filter |
| User cannot open shared mailbox | Full Access and client mapping check | `Get-MailboxPermission` | Missing Full Access, auto-mapping, Outlook cache, license/session |
| User can open mailbox but cannot send as | Send As check | `Get-RecipientPermission` | Missing Send As, propagation delay, wrong From address |
| User sends on behalf instead of as | Send on Behalf check | `Get-Mailbox -Identity "<mailbox>" \| Format-List GrantSendOnBehalfTo` | Wrong delegation type |
| Calendar access fails | Folder permission check | `Get-MailboxFolderPermission -Identity "<mailbox>:\Calendar"` | Folder permissions, localized folder name, client cache |
| Mail forwards unexpectedly | Forwarding and inbox rule check | `Get-Mailbox`; `Get-InboxRule` | Mailbox forwarding, inbox rule, transport rule |
| Message is quarantined | Quarantine search and policy check | `Get-QuarantineMessage`; `Get-HostedContentFilterPolicy` | Anti-spam, phishing, malware, policy action, quarantine policy |
| User cannot release quarantine | Quarantine policy and verdict check | `Get-QuarantinePolicy`; quarantine details | Admin-only policy, malware, high confidence phishing, missing permissions |
| External sender blocked | Accepted domain, connector, EOP check | `Get-MessageTrace`; `Get-InboundConnector`; `Get-HostedContentFilterPolicy` | Spam/phish verdict, inbound connector, sender authentication |
| Outbound mail blocked | Outbound spam and trace check | `Get-HostedOutboundSpamFilterPolicy`; `Get-MessageTrace` | Compromised account protection, forwarding block, connector route |
| Mail rule applied unexpectedly | Transport rule order check | `Get-TransportRule \| Sort-Object Priority` | Rule priority, broad condition, missing exception |
| Admin changed something but nobody knows what | Audit search | `Search-UnifiedAuditLog` | Admin change, mailbox delegation, transport rule, EOP policy |

# Troubleshoot_Mail_Flow_Mailbox_Access_Permissions_And_Quarantine_Issues_Configuration_Checklist

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Open incident record | Operator | Record sender, recipient, subject, time, direction, error, and impact | Scope is defined |
| 2 | Confirm Exchange Online connection | Admin Workstation | `Get-ConnectionInformation` | Active session exists |
| 3 | Confirm recipient exists | Admin Workstation | `Get-Recipient -Identity "<recipient>"` | Recipient object returns |
| 4 | Confirm mailbox exists | Admin Workstation | `Get-EXOMailbox -Identity "<mailbox>"` | Mailbox object returns |
| 5 | Run message trace | Admin Workstation | `Get-MessageTrace` or EAC message trace | Message status is known |
| 6 | Run message trace detail | Admin Workstation | `Get-MessageTraceDetail` or V2 detail cmdlet | Transport events are visible |
| 7 | Check quarantine | Admin Workstation / Defender portal | `Get-QuarantineMessage` | Quarantine state is known |
| 8 | Check EOP policy match | Admin Workstation | Export anti-spam, anti-malware, outbound policy and rules | Protection policy state is known |
| 9 | Check accepted domains | Admin Workstation | `Get-AcceptedDomain` | Domain type and default state are known |
| 10 | Check connectors | Admin Workstation | `Get-InboundConnector`; `Get-OutboundConnector` | Connector route and TLS state are known |
| 11 | Check transport rules | Admin Workstation | `Get-TransportRule` | Rule order and actions are known |
| 12 | Check mailbox forwarding | Admin Workstation | `Get-Mailbox -Identity "<mailbox>" \| Format-List Forwarding*` | Forwarding state is known |
| 13 | Check inbox rules | Admin Workstation | `Get-InboxRule -Mailbox "<mailbox>"` | Mailbox rules are known |
| 14 | Check mailbox permissions | Admin Workstation | `Get-MailboxPermission` | Full Access state is known |
| 15 | Check Send As | Admin Workstation | `Get-RecipientPermission` | Send As state is known |
| 16 | Check Send on Behalf | Admin Workstation | `Get-Mailbox \| Format-List GrantSendOnBehalfTo` | Send on Behalf state is known |
| 17 | Check folder permissions | Admin Workstation | `Get-MailboxFolderPermission` | Calendar or folder access state is known |
| 18 | Check mailbox audit | Admin Workstation | `Search-UnifiedAuditLog`; `Get-MailboxAuditBypassAssociation` | Access or admin activity is known |
| 19 | Apply scoped remediation | Admin Workstation | Use relevant remediation skeleton | Specific issue is corrected |
| 20 | Validate with retest | User / Admin Workstation | Send test mail, open mailbox, release quarantine, or rerun trace | Issue is resolved |
| 21 | Capture after-state evidence | Admin Workstation | Export relevant settings again | Evidence package is complete |
| 22 | Document root cause | Operator | Update ticket with cause, fix, validation, and rollback | Incident is closed cleanly |

# Troubleshoot_Mail_Flow_Mailbox_Access_Permissions_And_Quarantine_Issues_Intake_Skeleton

```powershell
# Run on the admin workstation.
# Purpose: create an incident evidence folder and collect initial facts for Exchange Online troubleshooting.

$AdminUpn = "admin@contoso.com"
$IncidentId = "INC-EXO-008"
$ReportedUser = "alex.wilber@contoso.com"
$SenderAddress = "sender@example.com"
$RecipientAddress = "alex.wilber@contoso.com"
$SubjectContains = "Quarterly report"
$StartDate = (Get-Date).AddDays(-2)
$EndDate = Get-Date
$EvidenceRoot = "C:\M365-Exchange-Baseline"
$RunStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$EvidencePath = Join-Path $EvidenceRoot "08-Troubleshooting-$IncidentId-$RunStamp"

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\intake-transcript.txt"

Import-Module ExchangeOnlineManagement

try {
  $Connection = Get-ConnectionInformation -ErrorAction Stop
}
catch {
  Connect-ExchangeOnline -UserPrincipalName $AdminUpn -ShowBanner:$false
}

@"
Incident ID: $IncidentId
Reported User: $ReportedUser
Sender: $SenderAddress
Recipient: $RecipientAddress
Subject Contains: $SubjectContains
Start Date: $StartDate
End Date: $EndDate
Evidence Path: $EvidencePath

Required user-provided facts:
- Sender address
- Recipient address
- Approximate sent time and time zone
- Subject or Message ID
- Screenshot or NDR text if available
- Whether this affects one user, multiple users, or all users
- Whether any recent mail flow, DNS, connector, EOP, mailbox, or permission change occurred
"@ | Out-File "$EvidencePath\incident-intake-summary.txt"

Get-ConnectionInformation |
  Tee-Object "$EvidencePath\connection-information.txt"

Get-Recipient -Identity $ReportedUser -ErrorAction SilentlyContinue |
  Format-List DisplayName,PrimarySmtpAddress,RecipientTypeDetails,ExternalDirectoryObjectId |
  Out-File "$EvidencePath\reported-user-recipient.txt"

Get-EXOMailbox -Identity $ReportedUser -ErrorAction SilentlyContinue |
  Format-List DisplayName,UserPrincipalName,PrimarySmtpAddress,RecipientTypeDetails,ArchiveStatus |
  Out-File "$EvidencePath\reported-user-mailbox.txt"

Get-Mailbox -Identity $ReportedUser -ErrorAction SilentlyContinue |
  Format-List DisplayName,PrimarySmtpAddress,ForwardingAddress,ForwardingSmtpAddress,DeliverToMailboxAndForward,GrantSendOnBehalfTo,RetentionPolicy,LitigationHoldEnabled,AuditEnabled |
  Out-File "$EvidencePath\reported-user-mailbox-control-state.txt"

Stop-Transcript
```

# Troubleshoot_Mail_Flow_Mailbox_Access_Permissions_And_Quarantine_Issues_Mail_Flow_Trace_Skeleton

```powershell
# Run on the admin workstation.
# Purpose: trace mail delivery status and transport events for sender, recipient, time window, and optional message identifiers.

$IncidentId = "INC-EXO-008"
$SenderAddress = "sender@example.com"
$RecipientAddress = "alex.wilber@contoso.com"
$StartDate = (Get-Date).AddDays(-2)
$EndDate = Get-Date
$MessageId = ""
$NetworkMessageId = ""
$EvidenceRoot = "C:\M365-Exchange-Baseline"
$RunStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$EvidencePath = Join-Path $EvidenceRoot "08-Mail-Flow-Trace-$IncidentId-$RunStamp"

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\mail-flow-trace-transcript.txt"

# Basic message trace by sender and recipient.
$TraceResults = Get-MessageTrace `
  -SenderAddress $SenderAddress `
  -RecipientAddress $RecipientAddress `
  -StartDate $StartDate `
  -EndDate $EndDate `
  -ErrorAction SilentlyContinue

$TraceResults |
  Select-Object Received,SenderAddress,RecipientAddress,Subject,Status,MessageTraceId,MessageId,FromIP,ToIP,Size |
  Export-Csv "$EvidencePath\message-trace-summary.csv" -NoTypeInformation

$TraceResults |
  Format-List * |
  Out-File "$EvidencePath\message-trace-summary-full.txt"

# Detail for each message trace result.
$TraceDetails = foreach ($Trace in $TraceResults) {
  Get-MessageTraceDetail `
    -MessageTraceId $Trace.MessageTraceId `
    -RecipientAddress $Trace.RecipientAddress `
    -ErrorAction SilentlyContinue |
    Select-Object @{Name="TraceReceived";Expression={$Trace.Received}},Event,Action,Detail,Data,Date,MessageTraceId,RecipientAddress
}

$TraceDetails |
  Export-Csv "$EvidencePath\message-trace-detail.csv" -NoTypeInformation

$TraceDetails |
  Format-Table -AutoSize |
  Out-File "$EvidencePath\message-trace-detail-table.txt"

# Optional Message ID filter if provided.
if ($MessageId -ne "") {
  Get-MessageTrace `
    -MessageId $MessageId `
    -StartDate $StartDate `
    -EndDate $EndDate `
    -ErrorAction SilentlyContinue |
    Select-Object Received,SenderAddress,RecipientAddress,Subject,Status,MessageTraceId,MessageId |
    Export-Csv "$EvidencePath\message-trace-by-message-id.csv" -NoTypeInformation
}

# Optional V2 trace path if available in tenant/module.
if (Get-Command Get-MessageTraceV2 -ErrorAction SilentlyContinue) {
  if ($NetworkMessageId -ne "") {
    Get-MessageTraceV2 `
      -MessageTraceId $NetworkMessageId `
      -SenderAddress $SenderAddress `
      -StartDate $StartDate `
      -EndDate $EndDate `
      -ErrorAction SilentlyContinue |
      Export-Csv "$EvidencePath\message-trace-v2-by-network-message-id.csv" -NoTypeInformation
  }
}

# Basic interpretation helper.
$TraceResults |
  Group-Object Status |
  Select-Object Name,Count |
  Export-Csv "$EvidencePath\message-trace-status-counts.csv" -NoTypeInformation

Stop-Transcript
```

# Troubleshoot_Mail_Flow_Mailbox_Access_Permissions_And_Quarantine_Issues_Mailbox_Access_Skeleton

```powershell
# Run on the admin workstation.
# Purpose: troubleshoot mailbox access, mailbox existence, sign-in relevance, license-adjacent indicators, and mailbox state.

$IncidentId = "INC-EXO-008"
$Mailbox = "alex.wilber@contoso.com"
$EvidenceRoot = "C:\M365-Exchange-Baseline"
$RunStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$EvidencePath = Join-Path $EvidenceRoot "08-Mailbox-Access-$IncidentId-$RunStamp"

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\mailbox-access-transcript.txt"

# Mailbox identity and type.
Get-Recipient -Identity $Mailbox -ErrorAction SilentlyContinue |
  Format-List DisplayName,Name,PrimarySmtpAddress,RecipientType,RecipientTypeDetails,ExternalDirectoryObjectId |
  Out-File "$EvidencePath\recipient-identity.txt"

Get-EXOMailbox -Identity $Mailbox -ErrorAction SilentlyContinue |
  Format-List DisplayName,UserPrincipalName,PrimarySmtpAddress,RecipientTypeDetails,ExchangeGuid,ArchiveStatus,HiddenFromAddressListsEnabled |
  Out-File "$EvidencePath\exo-mailbox-identity.txt"

Get-Mailbox -Identity $Mailbox -ErrorAction SilentlyContinue |
  Format-List DisplayName,PrimarySmtpAddress,RecipientTypeDetails,AccountDisabled,HiddenFromAddressListsEnabled,ForwardingAddress,ForwardingSmtpAddress,DeliverToMailboxAndForward,RetentionPolicy,LitigationHoldEnabled,AuditEnabled |
  Out-File "$EvidencePath\mailbox-control-state.txt"

# Mailbox statistics.
Get-MailboxStatistics -Identity $Mailbox -ErrorAction SilentlyContinue |
  Format-List DisplayName,ItemCount,TotalItemSize,DeletedItemCount,TotalDeletedItemSize,LastLogonTime,DisconnectDate |
  Out-File "$EvidencePath\mailbox-statistics.txt"

# Folder statistics for common hiding places.
Get-MailboxFolderStatistics -Identity $Mailbox -FolderScope Inbox -ErrorAction SilentlyContinue |
  Select-Object Name,FolderPath,ItemsInFolder,FolderAndSubfolderSize |
  Export-Csv "$EvidencePath\inbox-folder-statistics.csv" -NoTypeInformation

Get-MailboxFolderStatistics -Identity $Mailbox -FolderScope RecoverableItems -ErrorAction SilentlyContinue |
  Select-Object Name,FolderPath,ItemsInFolder,FolderAndSubfolderSize |
  Export-Csv "$EvidencePath\recoverable-items-statistics.csv" -NoTypeInformation

# Inbox rules.
Get-InboxRule -Mailbox $Mailbox -ErrorAction SilentlyContinue |
  Select-Object Name,Enabled,Priority,Description,ForwardTo,ForwardAsAttachmentTo,RedirectTo,DeleteMessage,MoveToFolder,MarkAsRead,StopProcessingRules |
  Export-Csv "$EvidencePath\inbox-rules.csv" -NoTypeInformation

Get-InboxRule -Mailbox $Mailbox -ErrorAction SilentlyContinue |
  Format-List * |
  Out-File "$EvidencePath\inbox-rules-full.txt"

# Calendar folder name discovery.
Get-MailboxFolderStatistics -Identity $Mailbox -FolderScope Calendar -ErrorAction SilentlyContinue |
  Select-Object Name,FolderPath,FolderType,ItemsInFolder |
  Export-Csv "$EvidencePath\calendar-folder-statistics.csv" -NoTypeInformation

Stop-Transcript
```

# Troubleshoot_Mail_Flow_Mailbox_Access_Permissions_And_Quarantine_Issues_Permissions_Delegation_Skeleton

```powershell
# Run on the admin workstation.
# Purpose: troubleshoot Full Access, Send As, Send on Behalf, shared mailbox access, and calendar delegation.

$IncidentId = "INC-EXO-008"
$TargetMailbox = "helpdesk@contoso.com"
$DelegateUser = "adele.vance@contoso.com"
$CalendarMailbox = "alex.wilber@contoso.com"
$CalendarIdentity = "$CalendarMailbox`:\Calendar"
$EvidenceRoot = "C:\M365-Exchange-Baseline"
$RunStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$EvidencePath = Join-Path $EvidenceRoot "08-Permissions-Delegation-$IncidentId-$RunStamp"

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\permissions-delegation-transcript.txt"

# Validate target and delegate objects.
Get-Recipient -Identity $TargetMailbox -ErrorAction SilentlyContinue |
  Format-List DisplayName,PrimarySmtpAddress,RecipientTypeDetails |
  Out-File "$EvidencePath\target-recipient.txt"

Get-Recipient -Identity $DelegateUser -ErrorAction SilentlyContinue |
  Format-List DisplayName,PrimarySmtpAddress,RecipientTypeDetails |
  Out-File "$EvidencePath\delegate-recipient.txt"

# Full Access.
Get-MailboxPermission -Identity $TargetMailbox -ErrorAction SilentlyContinue |
  Where-Object {$_.IsInherited -eq $false} |
  Select-Object Identity,User,AccessRights,Deny,IsInherited |
  Export-Csv "$EvidencePath\full-access-permissions.csv" -NoTypeInformation

Get-MailboxPermission -Identity $TargetMailbox -ErrorAction SilentlyContinue |
  Where-Object {
    $_.IsInherited -eq $false -and
    ($_.User -like "*$DelegateUser*" -or $_.User.ToString() -eq $DelegateUser)
  } |
  Format-List Identity,User,AccessRights,Deny,IsInherited |
  Out-File "$EvidencePath\delegate-full-access-check.txt"

# Send As.
Get-RecipientPermission -Identity $TargetMailbox -ErrorAction SilentlyContinue |
  Select-Object Identity,Trustee,AccessRights,Deny,IsInherited |
  Export-Csv "$EvidencePath\send-as-permissions.csv" -NoTypeInformation

Get-RecipientPermission -Identity $TargetMailbox -ErrorAction SilentlyContinue |
  Where-Object {
    $_.Trustee -like "*$DelegateUser*" -or $_.Trustee.ToString() -eq $DelegateUser
  } |
  Format-List Identity,Trustee,AccessRights,Deny,IsInherited |
  Out-File "$EvidencePath\delegate-send-as-check.txt"

# Send on Behalf.
Get-Mailbox -Identity $TargetMailbox -ErrorAction SilentlyContinue |
  Format-List DisplayName,PrimarySmtpAddress,GrantSendOnBehalfTo |
  Out-File "$EvidencePath\send-on-behalf-check.txt"

# Shared mailbox sent item behavior.
Get-Mailbox -Identity $TargetMailbox -ErrorAction SilentlyContinue |
  Format-List DisplayName,PrimarySmtpAddress,MessageCopyForSentAsEnabled,MessageCopyForSendOnBehalfEnabled |
  Out-File "$EvidencePath\shared-mailbox-sent-items-check.txt"

# Calendar permission.
Get-MailboxFolderPermission -Identity $CalendarIdentity -ErrorAction SilentlyContinue |
  Select-Object FolderName,User,AccessRights,SharingPermissionFlags |
  Export-Csv "$EvidencePath\calendar-permissions.csv" -NoTypeInformation

Get-MailboxFolderPermission -Identity $CalendarIdentity -User $DelegateUser -ErrorAction SilentlyContinue |
  Format-List FolderName,User,AccessRights,SharingPermissionFlags |
  Out-File "$EvidencePath\delegate-calendar-permission-check.txt"

Stop-Transcript
```

# Troubleshoot_Mail_Flow_Mailbox_Access_Permissions_And_Quarantine_Issues_Forwarding_And_Rule_Skeleton

```powershell
# Run on the admin workstation.
# Purpose: identify mailbox forwarding, inbox rules, transport rules, and outbound forwarding policy behavior.

$IncidentId = "INC-EXO-008"
$Mailbox = "alex.wilber@contoso.com"
$EvidenceRoot = "C:\M365-Exchange-Baseline"
$RunStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$EvidencePath = Join-Path $EvidenceRoot "08-Forwarding-And-Rules-$IncidentId-$RunStamp"

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\forwarding-and-rules-transcript.txt"

# Mailbox-level forwarding.
Get-Mailbox -Identity $Mailbox -ErrorAction SilentlyContinue |
  Format-List DisplayName,PrimarySmtpAddress,ForwardingAddress,ForwardingSmtpAddress,DeliverToMailboxAndForward |
  Out-File "$EvidencePath\mailbox-forwarding.txt"

# Inbox rules.
Get-InboxRule -Mailbox $Mailbox -ErrorAction SilentlyContinue |
  Select-Object Name,Enabled,Priority,Description,ForwardTo,ForwardAsAttachmentTo,RedirectTo,DeleteMessage,MoveToFolder,MarkAsRead,StopProcessingRules |
  Export-Csv "$EvidencePath\inbox-rules-summary.csv" -NoTypeInformation

Get-InboxRule -Mailbox $Mailbox -ErrorAction SilentlyContinue |
  Format-List * |
  Out-File "$EvidencePath\inbox-rules-full.txt"

# Rules with forwarding, redirect, delete, move, or mark read actions.
Get-InboxRule -Mailbox $Mailbox -ErrorAction SilentlyContinue |
  Where-Object {
    $_.ForwardTo -or
    $_.ForwardAsAttachmentTo -or
    $_.RedirectTo -or
    $_.DeleteMessage -eq $true -or
    $_.MoveToFolder -or
    $_.MarkAsRead -eq $true
  } |
  Format-List * |
  Out-File "$EvidencePath\suspicious-or-impactful-inbox-rules.txt"

# Transport rules.
Get-TransportRule |
  Sort-Object Priority |
  Select-Object Priority,Name,State,Mode,RuleVersion,Comments |
  Export-Csv "$EvidencePath\transport-rule-order.csv" -NoTypeInformation

Get-TransportRule |
  Format-List * |
  Out-File "$EvidencePath\transport-rules-full.txt"

# Outbound forwarding policy posture.
Get-HostedOutboundSpamFilterPolicy |
  Format-List * |
  Out-File "$EvidencePath\outbound-spam-policies-full.txt"

Get-HostedOutboundSpamFilterRule |
  Format-List * |
  Out-File "$EvidencePath\outbound-spam-rules-full.txt"

Stop-Transcript
```

# Troubleshoot_Mail_Flow_Mailbox_Access_Permissions_And_Quarantine_Issues_Quarantine_Skeleton

```powershell
# Run on the admin workstation.
# Purpose: search quarantine, identify verdict/policy, and capture safe release or escalation evidence.
# Do not release malware or high confidence phishing without formal approval.

$IncidentId = "INC-EXO-008"
$RecipientAddress = "alex.wilber@contoso.com"
$SenderAddress = "sender@example.com"
$StartReceivedDate = (Get-Date).AddDays(-7)
$EndReceivedDate = Get-Date
$EvidenceRoot = "C:\M365-Exchange-Baseline"
$RunStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$EvidencePath = Join-Path $EvidenceRoot "08-Quarantine-$IncidentId-$RunStamp"

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\quarantine-transcript.txt"

# Quarantine by recipient.
$RecipientQuarantine = Get-QuarantineMessage `
  -RecipientAddress $RecipientAddress `
  -StartReceivedDate $StartReceivedDate `
  -EndReceivedDate $EndReceivedDate `
  -PageSize 100 `
  -ErrorAction SilentlyContinue

$RecipientQuarantine |
  Select-Object ReceivedTime,SenderAddress,RecipientAddress,Subject,Type,PolicyName,Expires,Released,Identity |
  Export-Csv "$EvidencePath\quarantine-by-recipient.csv" -NoTypeInformation

# Quarantine by sender.
Get-QuarantineMessage `
  -SenderAddress $SenderAddress `
  -StartReceivedDate $StartReceivedDate `
  -EndReceivedDate $EndReceivedDate `
  -PageSize 100 `
  -ErrorAction SilentlyContinue |
  Select-Object ReceivedTime,SenderAddress,RecipientAddress,Subject,Type,PolicyName,Expires,Released,Identity |
  Export-Csv "$EvidencePath\quarantine-by-sender.csv" -NoTypeInformation

# First candidate for review.
$Candidate = $RecipientQuarantine |
  Where-Object {$_.Released -eq $false} |
  Select-Object -First 1

if ($Candidate) {
  $Candidate |
    Format-List * |
    Out-File "$EvidencePath\quarantine-candidate-for-review.txt"

  # Release only after review and approval.
  # Release-QuarantineMessage -Identity $Candidate.Identity -ReleaseToAll -Confirm:$false

  # Delete only if confirmed malicious or unwanted.
  # Delete-QuarantineMessage -Identity $Candidate.Identity -Confirm:$false
}

# Quarantine policies.
Get-QuarantinePolicy |
  Select-Object Name,EndUserQuarantinePermissionsValue,ESNEnabled,AdminDisplayName |
  Export-Csv "$EvidencePath\quarantine-policies.csv" -NoTypeInformation

Get-QuarantinePolicy |
  Format-List * |
  Out-File "$EvidencePath\quarantine-policies-full.txt"

Stop-Transcript
```

# Troubleshoot_Mail_Flow_Mailbox_Access_Permissions_And_Quarantine_Issues_EOP_Policy_Skeleton

```powershell
# Run on the admin workstation.
# Purpose: capture EOP and Defender policy state that could explain spam, phishing, malware, outbound block, or quarantine behavior.

$IncidentId = "INC-EXO-008"
$RecipientAddress = "alex.wilber@contoso.com"
$EvidenceRoot = "C:\M365-Exchange-Baseline"
$RunStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$EvidencePath = Join-Path $EvidenceRoot "08-EOP-Policy-$IncidentId-$RunStamp"

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\eop-policy-transcript.txt"

# Anti-spam policies and rules.
Get-HostedContentFilterPolicy |
  Select-Object Name,IsDefault,SpamAction,HighConfidenceSpamAction,PhishSpamAction,HighConfidencePhishAction,BulkSpamAction,QuarantineRetentionPeriod,EnableSpamZap,EnablePhishZap |
  Export-Csv "$EvidencePath\hosted-content-filter-policies.csv" -NoTypeInformation

Get-HostedContentFilterPolicy |
  Format-List * |
  Out-File "$EvidencePath\hosted-content-filter-policies-full.txt"

Get-HostedContentFilterRule |
  Select-Object Name,State,Priority,HostedContentFilterPolicy,RecipientDomainIs,SentTo,SentToMemberOf,ExceptIfSentTo,ExceptIfRecipientDomainIs |
  Export-Csv "$EvidencePath\hosted-content-filter-rules.csv" -NoTypeInformation

Get-HostedContentFilterRule |
  Format-List * |
  Out-File "$EvidencePath\hosted-content-filter-rules-full.txt"

# Anti-malware policies and rules.
Get-MalwareFilterPolicy |
  Select-Object Name,IsDefault,EnableFileFilter,FileTypeAction,ZapEnabled |
  Export-Csv "$EvidencePath\malware-filter-policies.csv" -NoTypeInformation

Get-MalwareFilterPolicy |
  Format-List * |
  Out-File "$EvidencePath\malware-filter-policies-full.txt"

Get-MalwareFilterRule |
  Select-Object Name,State,Priority,MalwareFilterPolicy,RecipientDomainIs,SentTo,SentToMemberOf,ExceptIfSentTo,ExceptIfRecipientDomainIs |
  Export-Csv "$EvidencePath\malware-filter-rules.csv" -NoTypeInformation

Get-MalwareFilterRule |
  Format-List * |
  Out-File "$EvidencePath\malware-filter-rules-full.txt"

# Outbound spam policies and rules.
Get-HostedOutboundSpamFilterPolicy |
  Format-List * |
  Out-File "$EvidencePath\hosted-outbound-spam-filter-policies-full.txt"

Get-HostedOutboundSpamFilterRule |
  Format-List * |
  Out-File "$EvidencePath\hosted-outbound-spam-filter-rules-full.txt"

# Tenant Allow/Block List.
foreach ($ListType in @("Sender","Url","FileHash")) {
  Get-TenantAllowBlockListItems -ListType $ListType -ErrorAction SilentlyContinue |
    Select-Object ListType,Value,Action,ExpirationDate,Notes |
    Export-Csv "$EvidencePath\tenant-allow-block-$ListType.csv" -NoTypeInformation
}

# Recipient object for policy scope matching.
Get-Recipient -Identity $RecipientAddress -ErrorAction SilentlyContinue |
  Format-List DisplayName,PrimarySmtpAddress,RecipientTypeDetails,MemberOfGroup |
  Out-File "$EvidencePath\recipient-policy-scope-context.txt"

Stop-Transcript
```

# Troubleshoot_Mail_Flow_Mailbox_Access_Permissions_And_Quarantine_Issues_Connector_And_Domain_Skeleton

```powershell
# Run on the admin workstation.
# Purpose: troubleshoot accepted domains, remote domains, connectors, DNS, and routing path.

$IncidentId = "INC-EXO-008"
$PrimaryDomain = "contoso.com"
$PartnerDomain = "partner.example"
$SmartHost = "mail.partner.example"
$EvidenceRoot = "C:\M365-Exchange-Baseline"
$RunStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$EvidencePath = Join-Path $EvidenceRoot "08-Connector-Domain-$IncidentId-$RunStamp"

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\connector-domain-transcript.txt"

# Accepted domains.
Get-AcceptedDomain |
  Select-Object Name,DomainName,DomainType,Default,AddressBookEnabled,PendingRemoval |
  Export-Csv "$EvidencePath\accepted-domains.csv" -NoTypeInformation

Get-AcceptedDomain |
  Format-List * |
  Out-File "$EvidencePath\accepted-domains-full.txt"

# Remote domains.
Get-RemoteDomain |
  Select-Object Name,DomainName,AllowedOOFType,AutoReplyEnabled,AutoForwardEnabled,DeliveryReportEnabled,TNEFEnabled |
  Export-Csv "$EvidencePath\remote-domains.csv" -NoTypeInformation

Get-RemoteDomain |
  Format-List * |
  Out-File "$EvidencePath\remote-domains-full.txt"

# Connectors.
Get-InboundConnector |
  Select-Object Name,Enabled,ConnectorType,SenderDomains,SenderIPAddresses,RequireTls,RestrictDomainsToIPAddresses,TlsSenderCertificateName,CloudServicesMailEnabled |
  Export-Csv "$EvidencePath\inbound-connectors.csv" -NoTypeInformation

Get-InboundConnector |
  Format-List * |
  Out-File "$EvidencePath\inbound-connectors-full.txt"

Get-OutboundConnector |
  Select-Object Name,Enabled,ConnectorType,RecipientDomains,SmartHosts,TlsSettings,TlsDomain,UseMXRecord,RouteAllMessagesViaOnPremises,CloudServicesMailEnabled |
  Export-Csv "$EvidencePath\outbound-connectors.csv" -NoTypeInformation

Get-OutboundConnector |
  Format-List * |
  Out-File "$EvidencePath\outbound-connectors-full.txt"

# DNS checks.
Resolve-DnsName -Name $PrimaryDomain -Type MX -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dns-mx-$PrimaryDomain.txt"

Resolve-DnsName -Name $PrimaryDomain -Type TXT -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dns-txt-$PrimaryDomain.txt"

Resolve-DnsName -Name $PartnerDomain -Type MX -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dns-mx-$PartnerDomain.txt"

Resolve-DnsName -Name $SmartHost -Type A -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dns-a-$SmartHost.txt"

Resolve-DnsName -Name $SmartHost -Type AAAA -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dns-aaaa-$SmartHost.txt"

Stop-Transcript
```

# Troubleshoot_Mail_Flow_Mailbox_Access_Permissions_And_Quarantine_Issues_Audit_And_Evidence_Skeleton

```powershell
# Run on the admin workstation.
# Purpose: capture audit evidence for mailbox access, send actions, transport rule changes, policy changes, and admin operations.

$IncidentId = "INC-EXO-008"
$AdminUpn = "admin@contoso.com"
$ReportedUser = "alex.wilber@contoso.com"
$DelegateUser = "adele.vance@contoso.com"
$StartDate = (Get-Date).AddDays(-7)
$EndDate = Get-Date
$EvidenceRoot = "C:\M365-Exchange-Baseline"
$RunStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$EvidencePath = Join-Path $EvidenceRoot "08-Audit-Evidence-$IncidentId-$RunStamp"

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\audit-evidence-transcript.txt"

# Mailbox audit settings.
Get-Mailbox -Identity $ReportedUser -ErrorAction SilentlyContinue |
  Format-List DisplayName,PrimarySmtpAddress,AuditEnabled,AuditAdmin,AuditDelegate,AuditOwner |
  Out-File "$EvidencePath\reported-user-mailbox-audit-settings.txt"

Get-MailboxAuditBypassAssociation -Identity $ReportedUser -ErrorAction SilentlyContinue |
  Format-List Identity,AuditBypassEnabled |
  Out-File "$EvidencePath\reported-user-audit-bypass.txt"

Get-MailboxAuditBypassAssociation -Identity $DelegateUser -ErrorAction SilentlyContinue |
  Format-List Identity,AuditBypassEnabled |
  Out-File "$EvidencePath\delegate-audit-bypass.txt"

# Unified audit log for reported user.
Search-UnifiedAuditLog `
  -StartDate $StartDate `
  -EndDate $EndDate `
  -UserIds $ReportedUser `
  -ResultSize 500 `
  -ErrorAction SilentlyContinue |
  Select-Object CreationDate,UserIds,Operations,AuditData |
  Export-Csv "$EvidencePath\unified-audit-reported-user.csv" -NoTypeInformation

# Unified audit log for delegate.
Search-UnifiedAuditLog `
  -StartDate $StartDate `
  -EndDate $EndDate `
  -UserIds $DelegateUser `
  -ResultSize 500 `
  -ErrorAction SilentlyContinue |
  Select-Object CreationDate,UserIds,Operations,AuditData |
  Export-Csv "$EvidencePath\unified-audit-delegate-user.csv" -NoTypeInformation

# Admin operations sample.
Search-UnifiedAuditLog `
  -StartDate $StartDate `
  -EndDate $EndDate `
  -UserIds $AdminUpn `
  -ResultSize 500 `
  -ErrorAction SilentlyContinue |
  Select-Object CreationDate,UserIds,Operations,AuditData |
  Export-Csv "$EvidencePath\unified-audit-admin-user.csv" -NoTypeInformation

Stop-Transcript
```

# Troubleshoot_Mail_Flow_Mailbox_Access_Permissions_And_Quarantine_Issues_Remediation_Skeleton

```powershell
# Run only after root cause is known.
# Purpose: apply scoped remediation for common Exchange Online issues.
# Keep all remediation blocks commented until the specific root cause is confirmed.

$IncidentId = "INC-EXO-008"
$Mailbox = "alex.wilber@contoso.com"
$SharedMailbox = "helpdesk@contoso.com"
$DelegateUser = "adele.vance@contoso.com"
$TransportRuleName = "LAB - Prepend EXTERNAL To External Mail"
$InboundConnectorName = "Inbound from Partner Example"
$OutboundConnectorName = "Outbound to Partner Example"
$QuarantineMessageIdentity = ""
$EvidenceRoot = "C:\M365-Exchange-Baseline"
$RunStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$EvidencePath = Join-Path $EvidenceRoot "08-Remediation-$IncidentId-$RunStamp"

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\remediation-transcript.txt"

@"
Remediation guardrail:
1. Confirm root cause.
2. Export before state.
3. Apply the smallest possible fix.
4. Validate with retest.
5. Export after state.
6. Record rollback command.
"@ | Out-File "$EvidencePath\remediation-guardrail.txt"

# Example fix: Grant missing Full Access.
# Add-MailboxPermission -Identity $SharedMailbox -User $DelegateUser -AccessRights FullAccess -InheritanceType All -AutoMapping $true

# Example fix: Grant missing Send As.
# Add-RecipientPermission -Identity $SharedMailbox -Trustee $DelegateUser -AccessRights SendAs -Confirm:$false

# Example fix: Add Send on Behalf without overwriting existing delegates.
# Set-Mailbox -Identity $SharedMailbox -GrantSendOnBehalfTo @{Add=$DelegateUser}

# Example fix: Clear unauthorized mailbox forwarding.
# Set-Mailbox -Identity $Mailbox -ForwardingAddress $null -ForwardingSmtpAddress $null -DeliverToMailboxAndForward $false

# Example fix: Disable malicious or mistaken inbox rule.
# Disable-InboxRule -Mailbox $Mailbox -Identity "<rule-name>"

# Example fix: Disable bad transport rule.
# Disable-TransportRule -Identity $TransportRuleName

# Example fix: Disable mis-scoped connector.
# Set-InboundConnector -Identity $InboundConnectorName -Enabled $false
# Set-OutboundConnector -Identity $OutboundConnectorName -Enabled $false

# Example fix: Release safe quarantined message after review.
# Release-QuarantineMessage -Identity $QuarantineMessageIdentity -ReleaseToAll -Confirm:$false

# Capture after-state snapshots.
Get-Mailbox -Identity $Mailbox -ErrorAction SilentlyContinue |
  Format-List DisplayName,PrimarySmtpAddress,ForwardingAddress,ForwardingSmtpAddress,DeliverToMailboxAndForward |
  Out-File "$EvidencePath\mailbox-forwarding-after-remediation.txt"

Get-MailboxPermission -Identity $SharedMailbox -ErrorAction SilentlyContinue |
  Where-Object {$_.IsInherited -eq $false} |
  Select-Object Identity,User,AccessRights,Deny,IsInherited |
  Export-Csv "$EvidencePath\shared-mailbox-full-access-after-remediation.csv" -NoTypeInformation

Get-RecipientPermission -Identity $SharedMailbox -ErrorAction SilentlyContinue |
  Select-Object Identity,Trustee,AccessRights,Deny,IsInherited |
  Export-Csv "$EvidencePath\shared-mailbox-send-as-after-remediation.csv" -NoTypeInformation

Stop-Transcript
```

# Troubleshoot_Mail_Flow_Mailbox_Access_Permissions_And_Quarantine_Issues_Escalation_Package_Skeleton

```powershell
# Run when escalation is required to Microsoft, security, legal, messaging engineering, or another team.
# Purpose: create a clean evidence package with facts, command outputs, and timeline.

$IncidentId = "INC-EXO-008"
$ReportedUser = "alex.wilber@contoso.com"
$SenderAddress = "sender@example.com"
$RecipientAddress = "alex.wilber@contoso.com"
$SubjectContains = "Quarterly report"
$BusinessImpact = "Single user unable to locate external message"
$CurrentStatus = "Pending root cause validation"
$EvidenceRoot = "C:\M365-Exchange-Baseline"
$RunStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$EvidencePath = Join-Path $EvidenceRoot "08-Escalation-Package-$IncidentId-$RunStamp"

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\escalation-package-transcript.txt"

@"
Incident ID: $IncidentId
Reported User: $ReportedUser
Sender: $SenderAddress
Recipient: $RecipientAddress
Subject Contains: $SubjectContains
Business Impact: $BusinessImpact
Current Status: $CurrentStatus

Escalation checklist:
- Include message trace summary and details.
- Include quarantine search result.
- Include mailbox forwarding and inbox rule export.
- Include transport rule order.
- Include accepted domain and connector exports.
- Include EOP policy exports.
- Include mailbox permission exports if access issue.
- Include Unified Audit Log export if admin or user action is suspected.
- Include exact UTC and local time windows.
- Include screenshots only if they add context not captured by command output.
"@ | Out-File "$EvidencePath\escalation-summary.txt"

# Core exports.
Get-ConnectionInformation |
  Out-File "$EvidencePath\connection-information.txt"

Get-Recipient -Identity $RecipientAddress -ErrorAction SilentlyContinue |
  Format-List * |
  Out-File "$EvidencePath\recipient-full.txt"

Get-Mailbox -Identity $RecipientAddress -ErrorAction SilentlyContinue |
  Format-List * |
  Out-File "$EvidencePath\mailbox-full.txt"

Get-MessageTrace `
  -SenderAddress $SenderAddress `
  -RecipientAddress $RecipientAddress `
  -StartDate (Get-Date).AddDays(-7) `
  -EndDate (Get-Date) `
  -ErrorAction SilentlyContinue |
  Export-Csv "$EvidencePath\message-trace.csv" -NoTypeInformation

Get-QuarantineMessage `
  -RecipientAddress $RecipientAddress `
  -StartReceivedDate (Get-Date).AddDays(-7) `
  -EndReceivedDate (Get-Date) `
  -PageSize 100 `
  -ErrorAction SilentlyContinue |
  Export-Csv "$EvidencePath\quarantine-search.csv" -NoTypeInformation

Get-InboxRule -Mailbox $RecipientAddress -ErrorAction SilentlyContinue |
  Format-List * |
  Out-File "$EvidencePath\inbox-rules-full.txt"

Get-TransportRule |
  Sort-Object Priority |
  Format-List * |
  Out-File "$EvidencePath\transport-rules-full.txt"

Get-InboundConnector |
  Format-List * |
  Out-File "$EvidencePath\inbound-connectors-full.txt"

Get-OutboundConnector |
  Format-List * |
  Out-File "$EvidencePath\outbound-connectors-full.txt"

Get-HostedContentFilterPolicy |
  Format-List * |
  Out-File "$EvidencePath\hosted-content-filter-policies-full.txt"

Get-MalwareFilterPolicy |
  Format-List * |
  Out-File "$EvidencePath\malware-filter-policies-full.txt"

Get-QuarantinePolicy |
  Format-List * |
  Out-File "$EvidencePath\quarantine-policies-full.txt"

Stop-Transcript
```

# Troubleshoot_Mail_Flow_Mailbox_Access_Permissions_And_Quarantine_Issues_Verification_Commands

| Check | Device | PowerShell / Command | Healthy Output |
|---|---|---|---|
| Exchange session active | Admin Workstation | `Get-ConnectionInformation` | Active Exchange Online session appears |
| Recipient exists | Admin Workstation | `Get-Recipient -Identity "<recipient>"` | Recipient returns |
| Mailbox exists | Admin Workstation | `Get-EXOMailbox -Identity "<mailbox>"` | Mailbox returns |
| Message trace by sender and recipient | Admin Workstation | `Get-MessageTrace -SenderAddress "<sender>" -RecipientAddress "<recipient>" -StartDate <start> -EndDate <end>` | Message status returns |
| Message trace details | Admin Workstation | `Get-MessageTraceDetail -MessageTraceId "<trace-id>" -RecipientAddress "<recipient>"` | Event detail returns |
| Quarantine by recipient | Admin Workstation | `Get-QuarantineMessage -RecipientAddress "<recipient>" -PageSize 100` | Quarantined messages return if present |
| Mailbox forwarding | Admin Workstation | `Get-Mailbox -Identity "<mailbox>" \| Format-List ForwardingAddress,ForwardingSmtpAddress,DeliverToMailboxAndForward` | Forwarding state is known |
| Inbox rules | Admin Workstation | `Get-InboxRule -Mailbox "<mailbox>"` | Rules return |
| Full Access | Admin Workstation | `Get-MailboxPermission -Identity "<mailbox>"` | Delegate permissions visible |
| Send As | Admin Workstation | `Get-RecipientPermission -Identity "<mailbox>"` | Send As permissions visible |
| Send on Behalf | Admin Workstation | `Get-Mailbox -Identity "<mailbox>" \| Format-List GrantSendOnBehalfTo` | Delegates visible |
| Calendar permission | Admin Workstation | `Get-MailboxFolderPermission -Identity "<mailbox>:\Calendar"` | Folder permissions visible |
| Transport rules | Admin Workstation | `Get-TransportRule \| Sort-Object Priority` | Rule order visible |
| Accepted domains | Admin Workstation | `Get-AcceptedDomain` | Domains return |
| Connectors | Admin Workstation | `Get-InboundConnector`; `Get-OutboundConnector` | Connector state visible |
| Anti-spam policies | Admin Workstation | `Get-HostedContentFilterPolicy`; `Get-HostedContentFilterRule` | Policy and rule state visible |
| Anti-malware policies | Admin Workstation | `Get-MalwareFilterPolicy`; `Get-MalwareFilterRule` | Policy and rule state visible |
| Outbound spam policy | Admin Workstation | `Get-HostedOutboundSpamFilterPolicy`; `Get-HostedOutboundSpamFilterRule` | Outbound policy state visible |
| Quarantine policies | Admin Workstation | `Get-QuarantinePolicy` | Policies return |
| Allow/block list | Admin Workstation | `Get-TenantAllowBlockListItems -ListType Sender` | Entries return if present |
| Unified audit log | Admin Workstation | `Search-UnifiedAuditLog -StartDate <start> -EndDate <end> -UserIds "<user>"` | Audit results return if activity exists |
| EAC message trace | Browser | Exchange admin center > Mail flow > Message trace | Trace UI opens |
| Defender quarantine | Browser | Defender portal > Email & collaboration > Review > Quarantine | Quarantine UI opens |

# Troubleshoot_Mail_Flow_Mailbox_Access_Permissions_And_Quarantine_Issues_Rollback

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify the fix applied | Operator | Review incident notes and remediation transcript | Specific changed object is known |
| 2 | Export current state before rollback | Admin Workstation | Run matching evidence skeleton | Pre-rollback state is saved |
| 3 | Roll back mailbox forwarding fix if needed | Admin Workstation | Restore previous `ForwardingAddress`, `ForwardingSmtpAddress`, and `DeliverToMailboxAndForward` values | Forwarding returns to approved prior state |
| 4 | Roll back inbox rule change if needed | Admin Workstation | `Enable-InboxRule` or recreate rule from evidence | Inbox rule state is restored |
| 5 | Roll back Full Access if needed | Admin Workstation | `Remove-MailboxPermission` or `Add-MailboxPermission` | Full Access returns to prior state |
| 6 | Roll back Send As if needed | Admin Workstation | `Remove-RecipientPermission` or `Add-RecipientPermission` | Send As returns to prior state |
| 7 | Roll back Send on Behalf if needed | Admin Workstation | `Set-Mailbox -GrantSendOnBehalfTo @{Add=...}` or `@{Remove=...}` | Send on Behalf returns to prior state |
| 8 | Roll back calendar permission if needed | Admin Workstation | `Set-MailboxFolderPermission` or `Remove-MailboxFolderPermission` | Calendar permission returns to prior state |
| 9 | Roll back transport rule fix if needed | Admin Workstation | `Enable-TransportRule`, `Disable-TransportRule`, or `Set-TransportRule` | Rule behavior returns to prior state |
| 10 | Roll back connector fix if needed | Admin Workstation | `Set-InboundConnector`; `Set-OutboundConnector` | Connector state returns to prior state |
| 11 | Roll back EOP policy fix if needed | Admin Workstation | `Set-HostedContentFilterPolicy`; `Set-MalwareFilterPolicy`; `Set-QuarantinePolicy` | Protection policy returns to prior state |
| 12 | Roll back Tenant Allow/Block entry if needed | Admin Workstation | `Remove-TenantAllowBlockListItems` | Temporary entry is removed |
| 13 | Validate after rollback | Admin Workstation / User | Re-run trace, mailbox access test, or quarantine check | Prior expected behavior is restored |
| 14 | Close with evidence | Operator | Update incident with rollback proof | Ticket has closure evidence |

# Troubleshoot_Mail_Flow_Mailbox_Access_Permissions_And_Quarantine_Issues_Rollback_Helper

```powershell
# Run only when a remediation must be reversed.
# Purpose: provide safe examples for targeted rollback.
# Do not run every block. Uncomment only the block that matches the specific change applied.

$IncidentId = "INC-EXO-008"
$Mailbox = "alex.wilber@contoso.com"
$SharedMailbox = "helpdesk@contoso.com"
$DelegateUser = "adele.vance@contoso.com"
$TransportRuleName = "LAB - Prepend EXTERNAL To External Mail"
$InboundConnectorName = "Inbound from Partner Example"
$OutboundConnectorName = "Outbound to Partner Example"
$EvidenceRoot = "C:\M365-Exchange-Baseline"
$RunStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$EvidencePath = Join-Path $EvidenceRoot "08-Rollback-$IncidentId-$RunStamp"

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\rollback-transcript.txt"

# Capture current state.
Get-Mailbox -Identity $Mailbox -ErrorAction SilentlyContinue |
  Format-List DisplayName,PrimarySmtpAddress,ForwardingAddress,ForwardingSmtpAddress,DeliverToMailboxAndForward |
  Out-File "$EvidencePath\mailbox-forwarding-before-rollback.txt"

Get-MailboxPermission -Identity $SharedMailbox -ErrorAction SilentlyContinue |
  Where-Object {$_.IsInherited -eq $false} |
  Select-Object Identity,User,AccessRights,Deny,IsInherited |
  Export-Csv "$EvidencePath\full-access-before-rollback.csv" -NoTypeInformation

Get-RecipientPermission -Identity $SharedMailbox -ErrorAction SilentlyContinue |
  Select-Object Identity,Trustee,AccessRights,Deny,IsInherited |
  Export-Csv "$EvidencePath\send-as-before-rollback.csv" -NoTypeInformation

# Example rollback: remove Full Access added during troubleshooting.
# Remove-MailboxPermission -Identity $SharedMailbox -User $DelegateUser -AccessRights FullAccess -Confirm:$false

# Example rollback: remove Send As added during troubleshooting.
# Remove-RecipientPermission -Identity $SharedMailbox -Trustee $DelegateUser -AccessRights SendAs -Confirm:$false

# Example rollback: remove Send on Behalf added during troubleshooting.
# Set-Mailbox -Identity $SharedMailbox -GrantSendOnBehalfTo @{Remove=$DelegateUser}

# Example rollback: clear forwarding set during troubleshooting.
# Set-Mailbox -Identity $Mailbox -ForwardingAddress $null -ForwardingSmtpAddress $null -DeliverToMailboxAndForward $false

# Example rollback: re-enable a transport rule disabled during troubleshooting.
# Enable-TransportRule -Identity $TransportRuleName

# Example rollback: re-enable connectors disabled during troubleshooting.
# Set-InboundConnector -Identity $InboundConnectorName -Enabled $true
# Set-OutboundConnector -Identity $OutboundConnectorName -Enabled $true

# Capture after state.
Get-Mailbox -Identity $Mailbox -ErrorAction SilentlyContinue |
  Format-List DisplayName,PrimarySmtpAddress,ForwardingAddress,ForwardingSmtpAddress,DeliverToMailboxAndForward |
  Out-File "$EvidencePath\mailbox-forwarding-after-rollback.txt"

Stop-Transcript
```

# Troubleshoot_Mail_Flow_Mailbox_Access_Permissions_And_Quarantine_Issues_Failure_Checks

| Symptom | Likely Cause | Check Command | Fix |
|---|---|---|---|
| Message trace finds no result | Wrong sender, recipient, time zone, time window, or message not received by Microsoft 365 | `Get-MessageTrace -StartDate <start> -EndDate <end>` | Widen time range, verify sender/recipient, use Message ID |
| Message trace says Delivered but user cannot find mail | Inbox rule, junk folder, deleted items, focused inbox, client cache, or delegate moved it | `Get-InboxRule`; mailbox audit; user OWA search | Disable bad rule, search mailbox, validate audit |
| Message trace says Quarantined | EOP or Defender policy held the message | `Get-QuarantineMessage`; `Get-HostedContentFilterPolicy`; `Get-MalwareFilterPolicy` | Review verdict, policy, and release path |
| Message trace says Filtered as spam | Spam filtering rejected or blocked the message | EOP policy and allow/block list export | Submit false positive or adjust scoped policy |
| Message trace says Failed | NDR, invalid recipient, connector, DNS, or policy failure | Trace detail and NDR text | Fix root cause identified by trace detail |
| Message trace says Pending | Retry in progress or remote endpoint unavailable | Trace detail, connector, smart host DNS | Validate remote server and connector route |
| Delivery group expands but member does not receive | Group membership, moderation, delivery restriction, or dynamic filter issue | `Get-DistributionGroupMember`; `Get-DynamicDistributionGroup`; trace details | Fix group membership, filter, moderation, or restrictions |
| Shared mailbox access denied | Missing Full Access, stale token, or Outlook cache | `Get-MailboxPermission` | Grant Full Access or refresh client |
| Auto-mapped mailbox missing | AutoMapping false, propagation delay, or Outlook profile issue | `Get-MailboxPermission` | Add mailbox manually or re-grant with AutoMapping true |
| User can open shared mailbox but cannot Send As | Send As missing | `Get-RecipientPermission` | Add Send As |
| User sends on behalf unexpectedly | Send on Behalf configured instead of Send As | `Get-Mailbox \| Format-List GrantSendOnBehalfTo` | Configure correct send permission |
| Calendar access denied | Folder permission missing or wrong folder path | `Get-MailboxFolderPermission`; `Get-MailboxFolderStatistics -FolderScope Calendar` | Use correct calendar folder identity and permission role |
| Mail forwards externally unexpectedly | Mailbox forwarding or inbox rule present | `Get-Mailbox \| Format-List Forwarding*`; `Get-InboxRule` | Remove unauthorized forwarding or rule |
| External forwarding blocked | Outbound spam policy blocks automatic forwarding | `Get-HostedOutboundSpamFilterPolicy` | Use approved policy exception or internal forwarding |
| Quarantine release unavailable | Verdict or quarantine policy prevents user release | `Get-QuarantinePolicy`; quarantine message type | Admin review or request release workflow |
| Quarantine message missing | Message expired, released, deleted, wrong recipient, or outside date range | `Get-QuarantineMessage -PageSize 100` | Widen search and verify recipient |
| Transport rule applied unexpectedly | Rule condition broad, exception missing, or priority wrong | `Get-TransportRule \| Sort Priority` | Add exception, change priority, or disable rule |
| Connector routes wrong mail | Recipient domain or sender domain scope too broad | `Get-InboundConnector`; `Get-OutboundConnector` | Narrow connector scope |
| TLS connector failure | Certificate, TLS domain, or smart host mismatch | `Get-OutboundConnector \| Format-List TlsSettings,TlsDomain,SmartHosts` | Correct TLS settings and certificate match |
| Accepted domain failure | Domain missing, wrong type, or DNS mismatch | `Get-AcceptedDomain`; DNS lookup | Fix accepted domain or DNS |
| Search-UnifiedAuditLog returns nothing | No audit event yet, wrong date, missing role, or audit delay | `Search-UnifiedAuditLog` with wider window | Wait, widen range, or assign audit role |
| PowerShell cmdlet unavailable | Not connected to Exchange Online or wrong role | `Get-ConnectionInformation`; `Get-Command <cmdlet>` | Reconnect and verify RBAC |
| EAC differs from PowerShell | Portal cache or service delay | Validate with PowerShell | Refresh browser or wait |
| Remediation fixed issue but created side effect | Fix was too broad | Compare before and after evidence | Roll back and apply narrower fix |

# Troubleshoot_Mail_Flow_Mailbox_Access_Permissions_And_Quarantine_Issues_Related_Labs

| Related Lab                                                                                    | Relationship                                                                                          |
| ---------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------- |
| `01_Configure_Exchange_Admin_Center_PowerShell_And_Org_Baseline.md`                            | Provides admin access, Exchange Online PowerShell connection, RBAC, and baseline evidence method      |
| `02_Manage_User_Mailboxes_Shared_Mailboxes_Contacts_And_Resources.md`                          | Provides recipient and mailbox object knowledge needed for troubleshooting                            |
| `03_Manage_Distribution_Groups_Dynamic_Groups_Mail_Enabled_Security_Groups_And_M365_Groups.md` | Provides group membership, delivery restriction, moderation, and dynamic group context                |
| `04_Configure_Mail_Flow_Accepted_Domains_Connectors_And_Rules.md`                              | Provides accepted domain, connector, remote domain, transport rule, and message trace baseline        |
| `05_Configure_Mailbox_Permissions_Delegation_Forwarding_And_Auditing.md`                       | Provides Full Access, Send As, Send on Behalf, forwarding, calendar permission, and audit context     |
| `06_Configure_Exchange_Online_Protection_AntiSpam_AntiMalware_And_Quarantine.md`               | Provides EOP, anti-spam, anti-malware, outbound spam, Tenant Allow/Block List, and quarantine context |
| `07_Configure_Retention_Archive_Litigation_Hold_And_eDiscovery_Handoff.md`                     | Provides retention, litigation hold, Recoverable Items, audit, and eDiscovery handoff context         |