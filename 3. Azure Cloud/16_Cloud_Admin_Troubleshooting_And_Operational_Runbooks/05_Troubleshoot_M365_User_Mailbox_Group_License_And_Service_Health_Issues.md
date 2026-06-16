# 05_Troubleshoot_M365_User_Mailbox_Group_License_And_Service_Health_Issues

## 05_Troubleshoot_M365_User_Mailbox_Group_License_And_Service_Health_Issues_Index

### Purpose

This workbook provides a structured troubleshooting workflow for Microsoft 365 user, mailbox, group, license, service plan, and service health issues across Microsoft 365 admin center, Microsoft Entra admin center, Exchange Online, Microsoft Graph PowerShell, and Exchange Online PowerShell.

### Scope

Use this workbook when Microsoft 365 users or administrators report one or more of the following symptoms:

- User cannot access Microsoft 365 services
- User cannot sign in to Microsoft 365 apps
- User exists but mailbox is missing
- Mailbox exists but user cannot send or receive mail
- Shared mailbox, room mailbox, equipment mailbox, or contact does not behave as expected
- Mailbox permissions, delegation, forwarding, archive, or auditing does not work
- Distribution group, mail-enabled security group, dynamic distribution group, or Microsoft 365 group issue occurs
- Group membership does not apply
- License assignment fails
- Service plan is disabled or missing
- Group-based licensing does not apply
- User has license but app/service is unavailable
- Microsoft 365 admin center shows service degradation or advisory
- Issue appears tenant-wide or workload-wide
- Microsoft 365 usage, reports, or admin portal data appears delayed
- Mail flow, provisioning, or permissions changed unexpectedly

### Assumptions

- You have access to Microsoft 365 admin center, Microsoft Entra admin center, Exchange admin center, and PowerShell where needed.
- You can connect to Microsoft Graph PowerShell and Exchange Online PowerShell.
- You know the affected user's UPN, mailbox address, group name, or license SKU.
- You will check service health before making broad tenant changes.
- You will capture evidence before modifying users, groups, licenses, mailboxes, or policies.
- You will avoid assigning broad admin roles just to resolve a single user or mailbox issue.

### Required Admin Roles

| Task Area | Minimum Role |
|---|---|
| View users and groups | Global Reader, User Administrator, Groups Administrator |
| Create/update users | User Administrator |
| Manage licenses | License Administrator, User Administrator, Global Administrator |
| Manage group-based licensing | Groups Administrator, License Administrator, Global Administrator |
| Manage mailboxes | Exchange Administrator |
| Manage mailbox permissions | Exchange Administrator |
| Run message trace | Exchange Administrator, Global Reader with appropriate Exchange access |
| View service health | Service Support Administrator, Global Reader, Global Administrator |
| View message center | Message Center Reader, Global Reader |
| View audit logs | Reports Reader, Audit Reader, Global Reader |
| Manage Microsoft 365 groups | Groups Administrator, Exchange Administrator |
| Manage distribution groups | Exchange Administrator |
| Emergency recovery | Global Administrator or break-glass Global Administrator |

---

## 05_Troubleshoot_M365_User_Mailbox_Group_License_And_Service_Health_Issues_Mental_Model

Microsoft 365 issues usually pass through several gates:

1. Tenant health gate: service is healthy and no active incident explains the symptom.
2. Identity gate: user exists, is enabled, and has the correct UPN and proxy addresses.
3. Authentication gate: sign-in is allowed and Conditional Access/MFA is not blocking access.
4. License gate: correct product license is assigned.
5. Service plan gate: the specific workload plan is enabled under that license.
6. Provisioning gate: mailbox, OneDrive, Teams, SharePoint, or workload object has actually provisioned.
7. Group gate: group type, membership, ownership, mail settings, and dynamic rules are correct.
8. Permission gate: mailbox/group/resource permissions are assigned at the right object.
9. Mail flow gate: accepted domain, recipient object, connectors, transport rules, quarantine, and message trace are clean.
10. Client/session gate: Outlook, Teams, browser, mobile app, token, or cache is current.
11. Audit/change gate: a recent admin, automation, sync, or policy change caused the symptom.

Traditional troubleshooting order:

1. Confirm service health.
2. Confirm exact user/group/mailbox and symptom.
3. Confirm identity state.
4. Confirm license and service plan state.
5. Confirm workload provisioning.
6. Confirm mailbox/group object state.
7. Confirm permissions and membership.
8. Confirm mail flow or client behavior.
9. Check audit logs and recent changes.
10. Apply the smallest confirmed fix.
11. Verify with a fresh session or test message.
12. Record root cause, evidence, fix, and rollback.

---

## 05_Troubleshoot_M365_User_Mailbox_Group_License_And_Service_Health_Issues_Symptom_Map

| Symptom | Likely Cause | Primary Evidence | First Check |
|---|---|---|---|
| User cannot access Microsoft 365 app | License missing, service plan disabled, sign-in blocked, CA issue, service incident | User license details and sign-in logs | User account and license |
| User exists but mailbox missing | Exchange license missing, provisioning delay, soft-deleted mailbox, sync issue | Exchange recipient lookup | `Get-EXOMailbox` and `Get-Recipient` |
| Mailbox exists but user cannot open it | License, mailbox disabled, permissions, client cache, CA | Mailbox state and permissions | Mailbox recipient type |
| User cannot send mail | License, mailbox quota, transport rule, connector, blocked sender, restricted entity | NDR and message trace | Message trace |
| User cannot receive mail | MX/accepted domain, recipient address missing, transport rule, quarantine, mailbox full | Message trace | Recipient lookup |
| Shared mailbox access fails | FullAccess missing, automapping cache, mailbox hidden, license issue over 50 GB | Mailbox permissions | `Get-MailboxPermission` |
| Send As fails | SendAs permission missing, token cache, wrong object | Recipient permission | `Get-RecipientPermission` |
| Send on Behalf fails | GrantSendOnBehalfTo missing | Mailbox properties | `GrantSendOnBehalfTo` |
| Distribution group mail fails | Moderation, delivery management, sender restriction, external sender block | Group properties and message trace | `Get-DistributionGroup` |
| M365 group missing in Outlook/Teams | Group hidden, provisioning delay, license/service issue, Teams not connected | Group properties | Group visibility |
| Dynamic group membership wrong | Rule syntax, processing delay, user attributes wrong | Dynamic group rule and member list | Group rule |
| License assignment fails | No available license, usage location missing, conflicting service plan, group-based licensing error | License assignment error | User usage location |
| Group-based license not applying | User not in group, group licensing error, conflicting plans | Entra group license blade | Group licensing errors |
| App missing despite license | Service plan disabled, app disabled, propagation delay | License details | Service plan |
| Service appears down for many users | Microsoft 365 incident/advisory | Service health | Service health |
| Admin center data stale | Reporting delay, service advisory, browser cache | Reports and service health | Report timestamp |
| Mailbox permission change not effective | Token/cache delay, Outlook automapping, wrong mailbox | Permission output | Reopen session / Outlook profile |

---

## 05_Troubleshoot_M365_User_Mailbox_Group_License_And_Service_Health_Issues_Evidence_Collection

| Evidence | Where To Get It | Why It Matters |
|---|---|---|
| Affected user UPN | User report / admin center | Primary identity lookup |
| Affected mailbox address | Exchange admin center / user report | Mailbox may differ from UPN |
| Affected group name/address | Admin center / Exchange admin center | Group type determines troubleshooting path |
| Tenant ID | Entra admin center / Graph | Prevents wrong tenant troubleshooting |
| Exact app/workload | User report | License/service plan depends on workload |
| Exact error message | Screenshot / copied text | Error often identifies licensing/auth/mail flow |
| Timestamp with timezone | User report | Needed for logs and message trace |
| Sign-in logs | Entra admin center | Shows authentication and CA failures |
| User account state | Entra admin center / Graph | Confirms enabled, synced, guest/member |
| License details | M365 admin center / Graph | Confirms product and service plans |
| Usage location | Entra user object | Required for license assignment |
| Mailbox state | Exchange Online | Confirms mailbox exists and type |
| Recipient state | Exchange Online | Confirms proxy addresses and recipient type |
| Message trace | Exchange admin center | Confirms send/receive path |
| NDR | User-provided bounce | Shows SMTP enhanced status code |
| Mailbox permissions | Exchange Online | Confirms FullAccess, SendAs, SendOnBehalf |
| Group properties | Exchange / Entra | Confirms group type, restrictions, hidden status |
| Group membership | Entra / Exchange | Confirms access path |
| Group-based license errors | Entra admin center | Identifies licensing conflict |
| Service health | M365 admin center | Confirms known incidents |
| Audit log | Purview / Entra | Finds recent admin or automation changes |
| Client info | User report | Outlook/Teams/browser cache may be involved |

---

## 05_Troubleshoot_M365_User_Mailbox_Group_License_And_Service_Health_Issues_Triage_Order

1. Confirm whether the issue affects one user, one group, one workload, or many users.
2. Check Microsoft 365 service health and advisories.
3. Confirm tenant, user UPN, mailbox address, group name, and timestamp.
4. Check user account state and sign-in status.
5. Check license assignment and usage location.
6. Check service plan state for the affected workload.
7. Check workload object provisioning, especially Exchange mailbox.
8. Check mailbox, recipient, proxy address, and accepted domain state.
9. Check group type, membership, ownership, visibility, delivery restrictions, and moderation.
10. Check message trace or client logs for mail flow issues.
11. Check mailbox permissions and delegation for access issues.
12. Check audit logs for recent changes.
13. Apply the smallest confirmed remediation.
14. Verify with a fresh session, message trace, or permission test.
15. Record root cause, evidence, fix, and rollback.

---

## 05_Troubleshoot_M365_User_Mailbox_Group_License_And_Service_Health_Issues_Planning_Table

| Item | Value |
|---|---|
| Tenant name | `<tenant-name>` |
| Tenant ID | `<tenant-id>` |
| Affected workload | `<Exchange / Teams / SharePoint / OneDrive / Office Apps / M365 Admin / Other>` |
| Affected user UPN | `<user@domain.com>` |
| Affected mailbox | `<mailbox@domain.com>` |
| Affected group | `<group-name-or-address>` |
| Group type | `<Microsoft 365 group / Distribution group / Mail-enabled security group / Dynamic distribution group / Security group>` |
| Affected license SKU | `<sku-part-number>` |
| Affected service plan | `<service-plan-name>` |
| Error message | `<exact-error>` |
| First failure time | `<yyyy-mm-dd hh:mm timezone>` |
| Scope | `<single user / multiple users / group / tenant / workload>` |
| Recent license change | `<yes-no>` |
| Recent group change | `<yes-no>` |
| Recent mailbox change | `<yes-no>` |
| Recent service health incident | `<yes-no>` |
| Change ticket | `<ticket-id>` |
| Business impact | `<impact>` |

---

## 05_Troubleshoot_M365_User_Mailbox_Group_License_And_Service_Health_Issues_Configuration_Checklist

| Step | Task | Device | Command | PowerShell | Expected Result |
|---|---|---|---|---|---|
| 1 | Confirm Microsoft 365 service health | Admin workstation | Microsoft 365 admin center > Health > Service health | `Get-MgServiceAnnouncementIssue -Top 20` | No active incident explains issue or incident is identified |
| 2 | Confirm tenant | Admin workstation | Microsoft 365 admin center > Org settings | `Get-MgOrganization | Select-Object DisplayName,Id,VerifiedDomains` | Correct tenant is selected |
| 3 | Confirm user exists | Admin workstation | Microsoft 365 admin center > Users > Active users | `Get-MgUser -UserId <user@domain.com> -Property Id,UserPrincipalName,DisplayName,AccountEnabled,UserType,UsageLocation` | User object exists |
| 4 | Confirm account enabled | Admin workstation | User > Account status | `Get-MgUser -UserId <user@domain.com> -Property AccountEnabled` | AccountEnabled is True |
| 5 | Check sign-in logs if user access issue | Admin workstation | Entra admin center > Sign-in logs | `Get-MgAuditLogSignIn -Filter "userPrincipalName eq '<user@domain.com>'" -Top 10` | Login/auth/CA result is known |
| 6 | Check usage location | Admin workstation | User > Licenses and apps | `Get-MgUser -UserId <user@domain.com> -Property UsageLocation` | UsageLocation is set before licensing |
| 7 | Check assigned licenses | Admin workstation | User > Licenses and apps | `Get-MgUserLicenseDetail -UserId <user@domain.com>` | Required license is assigned |
| 8 | Check tenant license inventory | Admin workstation | Billing > Licenses | `Get-MgSubscribedSku | Select-Object SkuPartNumber,ConsumedUnits,PrepaidUnits` | Available license capacity is known |
| 9 | Check service plan state | Admin workstation | User > Licenses and apps > Apps | `Get-MgUserLicenseDetail -UserId <user@domain.com> | Select-Object -ExpandProperty ServicePlans` | Required service plan is enabled |
| 10 | Check group-based license errors | Admin workstation | Entra admin center > Groups > group > Licenses | Portal preferred for error details | Group-based licensing error is identified or ruled out |
| 11 | Connect to Exchange Online | Admin workstation | Not applicable | `Connect-ExchangeOnline` | Exchange cmdlets are available |
| 12 | Check recipient object | Admin workstation | Exchange admin center > Recipients | `Get-Recipient <user@domain.com> | Format-List Name,RecipientType,RecipientTypeDetails,PrimarySmtpAddress,EmailAddresses` | Recipient exists and type is known |
| 13 | Check mailbox object | Admin workstation | Exchange admin center > Mailboxes | `Get-EXOMailbox <user@domain.com> -Properties RecipientTypeDetails,PrimarySmtpAddress,EmailAddresses,ArchiveStatus,ForwardingSmtpAddress,HiddenFromAddressListsEnabled` | Mailbox exists and properties are known |
| 14 | Check soft-deleted mailbox | Admin workstation | Exchange admin center or PowerShell | `Get-Mailbox -SoftDeletedMailbox <user@domain.com>` | Soft-deleted mailbox is identified or ruled out |
| 15 | Check mailbox statistics | Admin workstation | Exchange admin center > Mailbox usage | `Get-EXOMailboxStatistics <user@domain.com> | Select-Object DisplayName,TotalItemSize,ItemCount,LastLogonTime` | Mailbox size and access state are known |
| 16 | Check mailbox quota | Admin workstation | Exchange admin center > Mailbox usage | `Get-EXOMailbox <user@domain.com> -Properties ProhibitSendQuota,ProhibitSendReceiveQuota,IssueWarningQuota | Select-Object ProhibitSendQuota,ProhibitSendReceiveQuota,IssueWarningQuota` | Quota is not blocking send/receive |
| 17 | Check mailbox forwarding | Admin workstation | Exchange admin center > Mail flow settings | `Get-EXOMailbox <user@domain.com> -Properties ForwardingAddress,ForwardingSmtpAddress,DeliverToMailboxAndForward | Select-Object ForwardingAddress,ForwardingSmtpAddress,DeliverToMailboxAndForward` | Forwarding is expected or corrected |
| 18 | Check inbox rules | Admin workstation | Outlook on web or PowerShell | `Get-InboxRule -Mailbox <user@domain.com>` | Unexpected rules are identified |
| 19 | Run message trace | Admin workstation | Exchange admin center > Mail flow > Message trace | `Get-MessageTrace -RecipientAddress <user@domain.com> -StartDate (Get-Date).AddHours(-24) -EndDate (Get-Date)` | Delivery path is known |
| 20 | Check mailbox permissions | Admin workstation | Exchange admin center > Mailbox delegation | `Get-MailboxPermission <mailbox@domain.com> | Where-Object {$_.IsInherited -eq $false}` | FullAccess permissions are known |
| 21 | Check Send As permissions | Admin workstation | Exchange admin center > Mailbox delegation | `Get-RecipientPermission <mailbox@domain.com> | Where-Object {$_.Trustee -like "*<user>*"}` | SendAs permission is present or missing |
| 22 | Check Send on Behalf permissions | Admin workstation | Exchange admin center > Mailbox delegation | `Get-EXOMailbox <mailbox@domain.com> -Properties GrantSendOnBehalfTo | Select-Object GrantSendOnBehalfTo` | Send on Behalf state is known |
| 23 | Check group object in Entra | Admin workstation | Entra admin center > Groups | `Get-MgGroup -Filter "displayName eq '<group-name>'"` | Group exists and object ID is known |
| 24 | Check group members | Admin workstation | Entra admin center > Groups > Members | `Get-MgGroupMember -GroupId <group-id> -All` | Membership is correct or not |
| 25 | Check distribution group | Admin workstation | Exchange admin center > Recipients > Groups | `Get-DistributionGroup <group@domain.com> | Format-List Name,PrimarySmtpAddress,RequireSenderAuthenticationEnabled,AcceptMessagesOnlyFromSendersOrMembers,ModerationEnabled,HiddenFromAddressListsEnabled` | Mail delivery restrictions are known |
| 26 | Check distribution group members | Admin workstation | Exchange admin center > Group > Members | `Get-DistributionGroupMember <group@domain.com>` | Exchange group membership is known |
| 27 | Check Microsoft 365 group | Admin workstation | Microsoft 365 admin center > Teams & groups > Active teams & groups | `Get-UnifiedGroup <group@domain.com> | Format-List DisplayName,PrimarySmtpAddress,AccessType,HiddenFromExchangeClientsEnabled,HiddenFromAddressListsEnabled,RequireSenderAuthenticationEnabled` | M365 group settings are known |
| 28 | Check dynamic distribution group | Admin workstation | Exchange admin center > Groups > Dynamic distribution list | `Get-DynamicDistributionGroup <group-name> | Format-List Name,RecipientFilter,IncludedRecipients,Conditional*` | Dynamic rule is known |
| 29 | Preview dynamic distribution group members | Admin workstation | PowerShell | `Get-Recipient -RecipientPreviewFilter (Get-DynamicDistributionGroup <group-name>).RecipientFilter` | Dynamic membership output is known |
| 30 | Check audit logs for recent changes | Admin workstation | Purview audit or Entra audit logs | `Search-UnifiedAuditLog -StartDate (Get-Date).AddDays(-7) -EndDate (Get-Date) -UserIds <admin-or-user@domain.com>` | Recent changes are identified |
| 31 | Apply smallest confirmed fix | Admin workstation | Correct account, license, mailbox, group, or permission setting | Depends on fix | Root cause is remediated |
| 32 | Verify with fresh session or test message | User workstation / Admin workstation | Sign out/in, test app, send test mail, rerun trace | Depends on workload | Original symptom is resolved |
| 33 | Record incident and rollback | Admin workstation | Update incident/change record | Not applicable | Evidence, fix, verification, and rollback are documented |

---

## 05_Troubleshoot_M365_User_Mailbox_Group_License_And_Service_Health_Issues_User_And_License_Workflow

### Phase 1: User State And Sign-In

| Step | Task | Device | Command | PowerShell | Expected Result |
|---|---|---|---|---|---|
| 1 | Confirm exact UPN | Admin workstation | Microsoft 365 admin center > Users > Active users | `Get-MgUser -UserId <user@domain.com>` | Correct user object is located |
| 2 | Check account enabled | Admin workstation | User > Account status | `Get-MgUser -UserId <user@domain.com> -Property AccountEnabled` | Account is enabled |
| 3 | Check user type | Admin workstation | User properties | `Get-MgUser -UserId <user@domain.com> -Property UserType` | User is expected Member or Guest |
| 4 | Check usage location | Admin workstation | User > Licenses and apps | `Get-MgUser -UserId <user@domain.com> -Property UsageLocation` | Usage location is populated |
| 5 | Check recent sign-ins | Admin workstation | Entra admin center > Monitoring > Sign-in logs | `Get-MgAuditLogSignIn -Filter "userPrincipalName eq '<user@domain.com>'" -Top 10` | Sign-in error, CA status, or success is known |
| 6 | Check recent user changes | Admin workstation | Entra admin center > Audit logs | `Get-MgAuditLogDirectoryAudit -Top 50 | Where-Object {$_.TargetResources -match "<user@domain.com>"}` | Recent identity changes are identified |

### Phase 2: License Assignment

| Step | Task | Device | Command | PowerShell | Expected Result |
|---|---|---|---|---|---|
| 1 | Check tenant SKUs | Admin workstation | Microsoft 365 admin center > Billing > Licenses | `Get-MgSubscribedSku | Select-Object SkuPartNumber,ConsumedUnits,PrepaidUnits` | License capacity is known |
| 2 | Check user license details | Admin workstation | User > Licenses and apps | `Get-MgUserLicenseDetail -UserId <user@domain.com>` | Assigned product licenses are known |
| 3 | Check service plans under license | Admin workstation | User > Licenses and apps > Apps | `Get-MgUserLicenseDetail -UserId <user@domain.com> | Select-Object -ExpandProperty ServicePlans` | Workload plan is enabled or disabled |
| 4 | Check direct vs inherited license | Admin workstation | Entra user > Licenses | Portal preferred for direct/inherited distinction | Assignment path is known |
| 5 | Check group-based license group | Admin workstation | Entra admin center > Groups > group > Licenses | `Get-MgGroup -Filter "displayName eq '<group-name>'"` | Group license assignment is known |
| 6 | Check group membership | Admin workstation | Group > Members | `Get-MgGroupMember -GroupId <group-id> -All` | User is or is not member |
| 7 | Fix usage location first | Admin workstation | User > Properties > Usage location | `Update-MgUser -UserId <user@domain.com> -UsageLocation "US"` | Usage location is set |
| 8 | Assign license if approved | Admin workstation | User > Licenses and apps | `Set-MgUserLicense -UserId <user@domain.com> -AddLicenses @{SkuId="<sku-guid>"} -RemoveLicenses @()` | License is assigned |
| 9 | Disable/enable service plan only if intended | Admin workstation | User > Licenses and apps > Apps | Use Graph license assignment with DisabledPlans after planning | Required service plan is enabled |
| 10 | Verify workload access after propagation | User workstation | Sign out/in and open affected app | Not applicable | App/service is available |

---

## 05_Troubleshoot_M365_User_Mailbox_Group_License_And_Service_Health_Issues_Mailbox_Workflow

### Phase 1: Mailbox Provisioning And Recipient State

| Step | Task | Device | Command | PowerShell | Expected Result |
|---|---|---|---|---|---|
| 1 | Connect to Exchange Online | Admin workstation | Not applicable | `Connect-ExchangeOnline` | Exchange session is connected |
| 2 | Check recipient object | Admin workstation | Exchange admin center > Recipients | `Get-Recipient <user@domain.com> | Format-List Name,RecipientType,RecipientTypeDetails,PrimarySmtpAddress,EmailAddresses` | Recipient exists and type is known |
| 3 | Check mailbox | Admin workstation | Exchange admin center > Mailboxes | `Get-EXOMailbox <user@domain.com> -Properties RecipientTypeDetails,PrimarySmtpAddress,EmailAddresses,ArchiveStatus,HiddenFromAddressListsEnabled` | Mailbox exists |
| 4 | Check soft-deleted mailbox | Admin workstation | PowerShell | `Get-Mailbox -SoftDeletedMailbox <user@domain.com>` | Soft-deleted mailbox is found or ruled out |
| 5 | Check mailbox statistics | Admin workstation | Mailbox > Usage | `Get-EXOMailboxStatistics <user@domain.com> | Select-Object DisplayName,TotalItemSize,ItemCount,LastLogonTime` | Mailbox size and access data are visible |
| 6 | Check primary SMTP and aliases | Admin workstation | Mailbox > Email addresses | `Get-EXOMailbox <user@domain.com> -Properties EmailAddresses | Select-Object PrimarySmtpAddress,EmailAddresses` | Address list is correct |
| 7 | Check hidden address list state | Admin workstation | Mailbox properties | `Get-EXOMailbox <user@domain.com> -Properties HiddenFromAddressListsEnabled | Select-Object HiddenFromAddressListsEnabled` | GAL visibility state is known |
| 8 | Check archive mailbox | Admin workstation | Mailbox > Others > Manage mailbox archive | `Get-EXOMailbox <user@domain.com> -Properties ArchiveStatus,ArchiveGuid | Select-Object ArchiveStatus,ArchiveGuid` | Archive state is known |
| 9 | Check litigation hold or retention impact | Admin workstation | Mailbox > Others > Litigation hold | `Get-EXOMailbox <user@domain.com> -Properties LitigationHoldEnabled,RetentionPolicy | Select-Object LitigationHoldEnabled,RetentionPolicy` | Compliance settings are known |
| 10 | Verify mailbox after license/provisioning | Admin workstation | Refresh Exchange admin center | `Get-EXOMailbox <user@domain.com>` | Mailbox exists and is ready |

### Phase 2: Mail Flow And Message Trace

| Step | Task | Device | Command | PowerShell | Expected Result |
|---|---|---|---|---|---|
| 1 | Confirm accepted domain | Admin workstation | Exchange admin center > Mail flow > Accepted domains | `Get-AcceptedDomain | Where-Object {$_.DomainName -eq "<domain.com>"}` | Domain exists and type is correct |
| 2 | Run inbound trace | Admin workstation | Exchange admin center > Mail flow > Message trace | `Get-MessageTrace -RecipientAddress <user@domain.com> -StartDate (Get-Date).AddHours(-24) -EndDate (Get-Date)` | Inbound message status is visible |
| 3 | Run outbound trace | Admin workstation | Exchange admin center > Mail flow > Message trace | `Get-MessageTrace -SenderAddress <user@domain.com> -StartDate (Get-Date).AddHours(-24) -EndDate (Get-Date)` | Outbound message status is visible |
| 4 | Get message trace details | Admin workstation | Message trace > Details | `Get-MessageTraceDetail -MessageTraceId <trace-id> -RecipientAddress <recipient@domain.com>` | Transport events are visible |
| 5 | Check mailbox forwarding | Admin workstation | Mailbox > Mail flow settings | `Get-EXOMailbox <user@domain.com> -Properties ForwardingAddress,ForwardingSmtpAddress,DeliverToMailboxAndForward` | Forwarding state is known |
| 6 | Check inbox rules | Admin workstation | User mailbox / PowerShell | `Get-InboxRule -Mailbox <user@domain.com>` | Rules that move/delete/forward mail are identified |
| 7 | Check transport rules | Admin workstation | Exchange admin center > Mail flow > Rules | `Get-TransportRule | Select-Object Name,State,Mode,Priority` | Mail flow rules are identified |
| 8 | Check quarantine | Admin workstation | Microsoft Defender portal > Email & collaboration > Review > Quarantine | Not applicable | Message is quarantined or not |
| 9 | Check restricted entities | Admin workstation | Defender portal > Email & collaboration > Review > Restricted entities | Not applicable | User is blocked from sending or not |
| 10 | Verify with test mail | User/admin workstation | Send internal and external test messages | `Get-MessageTrace -SenderAddress <user@domain.com> -StartDate (Get-Date).AddMinutes(-30) -EndDate (Get-Date)` | Mail flow succeeds |

### Phase 3: Mailbox Permissions And Delegation

| Step | Task | Device | Command | PowerShell | Expected Result |
|---|---|---|---|---|---|
| 1 | Check FullAccess | Admin workstation | Exchange admin center > Mailbox delegation | `Get-MailboxPermission <mailbox@domain.com> | Where-Object {$_.IsInherited -eq $false}` | FullAccess is present or missing |
| 2 | Add FullAccess if approved | Admin workstation | Mailbox delegation > Read and manage | `Add-MailboxPermission -Identity <mailbox@domain.com> -User <delegate@domain.com> -AccessRights FullAccess -InheritanceType All` | Delegate can open mailbox after propagation |
| 3 | Check SendAs | Admin workstation | Mailbox delegation > Send as | `Get-RecipientPermission <mailbox@domain.com>` | SendAs is present or missing |
| 4 | Add SendAs if approved | Admin workstation | Mailbox delegation > Send as | `Add-RecipientPermission -Identity <mailbox@domain.com> -Trustee <delegate@domain.com> -AccessRights SendAs -Confirm:$false` | Delegate can send as mailbox |
| 5 | Check Send on Behalf | Admin workstation | Mailbox delegation > Send on behalf | `Get-EXOMailbox <mailbox@domain.com> -Properties GrantSendOnBehalfTo | Select-Object GrantSendOnBehalfTo` | Send on Behalf state is known |
| 6 | Add Send on Behalf if approved | Admin workstation | Mailbox delegation > Send on behalf | `Set-Mailbox <mailbox@domain.com> -GrantSendOnBehalfTo @{Add="<delegate@domain.com>"}` | Delegate can send on behalf |
| 7 | Check automapping issue | User workstation | Outlook profile behavior | Not applicable | Automapping cache issue is identified |
| 8 | Re-add FullAccess without automapping if needed | Admin workstation | PowerShell | `Remove-MailboxPermission -Identity <mailbox@domain.com> -User <delegate@domain.com> -AccessRights FullAccess -Confirm:$false; Add-MailboxPermission -Identity <mailbox@domain.com> -User <delegate@domain.com> -AccessRights FullAccess -AutoMapping:$false` | Mailbox can be manually added |
| 9 | Refresh user token/client | User workstation | Sign out/in, restart Outlook, OWA test | Not applicable | Permission becomes effective |
| 10 | Verify delegation | User workstation | OWA open another mailbox; send test message | Not applicable | Delegated access works |

---

## 05_Troubleshoot_M365_User_Mailbox_Group_License_And_Service_Health_Issues_Group_Workflow

### Phase 1: Identify Group Type

| Step | Task | Device | Command | PowerShell | Expected Result |
|---|---|---|---|---|---|
| 1 | Search group in Microsoft 365 admin center | Admin workstation | Teams & groups > Active teams & groups | Not applicable | Group appears or does not |
| 2 | Search group in Entra | Admin workstation | Entra admin center > Groups | `Get-MgGroup -Filter "displayName eq '<group-name>'"` | Entra group object is found |
| 3 | Search Exchange recipient | Admin workstation | Exchange admin center > Recipients > Groups | `Get-Recipient <group@domain.com>` | Exchange-recipient group is found |
| 4 | Identify group type | Admin workstation | Review group properties | `Get-Recipient <group@domain.com> | Format-List RecipientType,RecipientTypeDetails` | Group type is known |
| 5 | Confirm owners | Admin workstation | Group > Owners | `Get-MgGroupOwner -GroupId <group-id> -All` | Owners are known |
| 6 | Confirm members | Admin workstation | Group > Members | `Get-MgGroupMember -GroupId <group-id> -All` | Members are known |

### Phase 2: Distribution Group And Mail-Enabled Group

| Step | Task | Device | Command | PowerShell | Expected Result |
|---|---|---|---|---|---|
| 1 | Check distribution group settings | Admin workstation | Exchange admin center > Recipients > Groups | `Get-DistributionGroup <group@domain.com> | Format-List Name,PrimarySmtpAddress,RequireSenderAuthenticationEnabled,AcceptMessagesOnlyFromSendersOrMembers,RejectMessagesFromSendersOrMembers,ModerationEnabled,HiddenFromAddressListsEnabled` | Delivery restrictions are known |
| 2 | Check members | Admin workstation | Group > Members | `Get-DistributionGroupMember <group@domain.com>` | Membership is correct |
| 3 | Check external sender restriction | Admin workstation | Group delivery management | `Get-DistributionGroup <group@domain.com> | Select-Object RequireSenderAuthenticationEnabled` | External senders allowed or blocked |
| 4 | Check moderation | Admin workstation | Group message approval | `Get-DistributionGroup <group@domain.com> | Select-Object ModerationEnabled,ModeratedBy,BypassModerationFromSendersOrMembers` | Moderation is expected or blocking |
| 5 | Check accepted senders | Admin workstation | Group delivery management | `Get-DistributionGroup <group@domain.com> | Select-Object AcceptMessagesOnlyFromSendersOrMembers` | Sender restrictions are known |
| 6 | Check group hidden from GAL | Admin workstation | Group properties | `Get-DistributionGroup <group@domain.com> | Select-Object HiddenFromAddressListsEnabled` | GAL visibility is known |
| 7 | Run message trace to group | Admin workstation | Exchange admin center > Message trace | `Get-MessageTrace -RecipientAddress <group@domain.com> -StartDate (Get-Date).AddHours(-24) -EndDate (Get-Date)` | Group delivery status is visible |
| 8 | Correct group setting | Admin workstation | Update delivery management or membership | `Set-DistributionGroup <group@domain.com> -RequireSenderAuthenticationEnabled $false` | Group behavior matches design |
| 9 | Verify group mail | User/admin workstation | Send test message | Message trace after test | Group delivery works |

### Phase 3: Microsoft 365 Group

| Step | Task | Device | Command | PowerShell | Expected Result |
|---|---|---|---|---|---|
| 1 | Check unified group | Admin workstation | Microsoft 365 admin center > Teams & groups | `Get-UnifiedGroup <group@domain.com> | Format-List DisplayName,PrimarySmtpAddress,AccessType,HiddenFromExchangeClientsEnabled,HiddenFromAddressListsEnabled,RequireSenderAuthenticationEnabled,AlwaysSubscribeMembersToCalendarEvents` | M365 group settings are known |
| 2 | Check members | Admin workstation | Group > Members | `Get-UnifiedGroupLinks <group@domain.com> -LinkType Members` | Members are correct |
| 3 | Check owners | Admin workstation | Group > Owners | `Get-UnifiedGroupLinks <group@domain.com> -LinkType Owners` | Owners are correct |
| 4 | Check subscribers | Admin workstation | PowerShell | `Get-UnifiedGroupLinks <group@domain.com> -LinkType Subscribers` | Mail subscribers are known |
| 5 | Check hidden from Outlook clients | Admin workstation | Group settings | `Get-UnifiedGroup <group@domain.com> | Select-Object HiddenFromExchangeClientsEnabled,HiddenFromAddressListsEnabled` | Visibility settings are known |
| 6 | Check external sender setting | Admin workstation | Group settings | `Get-UnifiedGroup <group@domain.com> | Select-Object RequireSenderAuthenticationEnabled` | External mail behavior is known |
| 7 | Correct visibility if approved | Admin workstation | Exchange PowerShell | `Set-UnifiedGroup <group@domain.com> -HiddenFromExchangeClientsEnabled:$false -HiddenFromAddressListsEnabled:$false` | Group appears where expected |
| 8 | Add owner/member if needed | Admin workstation | Group membership page | `Add-UnifiedGroupLinks <group@domain.com> -LinkType Members -Links <user@domain.com>` | Membership is corrected |
| 9 | Verify access | User workstation | Outlook/Teams/SharePoint group access | Not applicable | Group access works |

### Phase 4: Dynamic Distribution Group

| Step | Task | Device | Command | PowerShell | Expected Result |
|---|---|---|---|---|---|
| 1 | Check dynamic group filter | Admin workstation | Exchange admin center > Groups | `Get-DynamicDistributionGroup <group-name> | Format-List Name,RecipientFilter,IncludedRecipients,Conditional*` | Recipient filter is known |
| 2 | Preview membership | Admin workstation | PowerShell | `Get-Recipient -RecipientPreviewFilter (Get-DynamicDistributionGroup <group-name>).RecipientFilter` | Previewed members match expected result |
| 3 | Check user attributes | Admin workstation | User properties | `Get-Recipient <user@domain.com> | Format-List Department,Company,Office,CustomAttribute*` | Attributes match rule |
| 4 | Fix user attributes or filter | Admin workstation | Entra/Exchange user properties or group filter | `Set-DynamicDistributionGroup <group-name> -RecipientFilter "<filter>"` | Rule matches intended population |
| 5 | Verify delivery | User/admin workstation | Send test message | `Get-MessageTrace -RecipientAddress <user@domain.com> -StartDate (Get-Date).AddHours(-1) -EndDate (Get-Date)` | Dynamic group delivery works |

---

## 05_Troubleshoot_M365_User_Mailbox_Group_License_And_Service_Health_Issues_Service_Health_Workflow

### Phase 1: Service Health And Advisory Review

| Step | Task | Device | Command | PowerShell | Expected Result |
|---|---|---|---|---|---|
| 1 | Open service health | Admin workstation | Microsoft 365 admin center > Health > Service health | `Get-MgServiceAnnouncementIssue -Top 20` | Incidents/advisories are visible |
| 2 | Filter by workload | Admin workstation | Service health > Exchange Online / Teams / SharePoint / OneDrive / Microsoft 365 suite | Review Graph output | Relevant incident is identified or ruled out |
| 3 | Check affected geography | Admin workstation | Incident details | Review incident details | Tenant geography impact is known |
| 4 | Check incident start time | Admin workstation | Incident details | Review incident details | Timeline matches user reports |
| 5 | Check admin center message center | Admin workstation | Health > Message center | `Get-MgServiceAnnouncementMessage -Top 20` | Planned change/advisory is identified |
| 6 | Check Microsoft 365 reports delay | Admin workstation | Reports > Usage | Not applicable | Report freshness is understood |
| 7 | Communicate known incident | Admin workstation | Send internal update | Not applicable | Users know issue is service-side |
| 8 | Avoid unnecessary tenant changes | Admin workstation | Freeze unrelated changes during incident | Not applicable | No extra config drift is introduced |

### Phase 2: Tenant-Wide Impact Isolation

| Step | Task | Device | Command | PowerShell | Expected Result |
|---|---|---|---|---|---|
| 1 | Compare affected and unaffected users | Admin workstation | Check different license, region, department, app, and device | Not applicable | Pattern emerges |
| 2 | Compare workloads | Admin workstation | Test OWA, Outlook, Teams, SharePoint, OneDrive | Not applicable | Scope is workload-specific or tenant-wide |
| 3 | Compare clients | User workstation | Browser private window, desktop app, mobile app | Not applicable | Client cache issue is ruled in/out |
| 4 | Compare network | User workstation | Test on corporate network and external network if allowed | Not applicable | Network condition is ruled in/out |
| 5 | Check recent admin changes | Admin workstation | Entra audit, Exchange admin audit, Purview audit | `Search-UnifiedAuditLog -StartDate (Get-Date).AddDays(-7) -EndDate (Get-Date)` | Recent changes are identified |
| 6 | Check license availability | Admin workstation | Billing > Licenses | `Get-MgSubscribedSku` | Tenant has license capacity |
| 7 | Check service plan disabled pattern | Admin workstation | Sample affected users | `Get-MgUserLicenseDetail -UserId <user@domain.com>` | Common disabled plan is identified |
| 8 | Verify after service incident resolves | Admin workstation | Retest affected workflow | Not applicable | Issue clears without local change or requires follow-up |

---

## 05_Troubleshoot_M365_User_Mailbox_Group_License_And_Service_Health_Issues_Verification_Commands

### Microsoft Graph PowerShell

| Purpose | PowerShell |
|---|---|
| Connect for M365 user/license checks | `Connect-MgGraph -Scopes "User.Read.All","Group.Read.All","Directory.Read.All","Organization.Read.All","AuditLog.Read.All","LicenseAssignment.Read.All","ServiceHealth.Read.All","ServiceMessage.Read.All"` |
| Confirm tenant | `Get-MgOrganization | Select-Object DisplayName,Id,VerifiedDomains` |
| Check user | `Get-MgUser -UserId <user@domain.com> -Property Id,UserPrincipalName,DisplayName,AccountEnabled,UserType,UsageLocation,AssignedLicenses` |
| Check sign-ins | `Get-MgAuditLogSignIn -Filter "userPrincipalName eq '<user@domain.com>'" -Top 10` |
| Check user license detail | `Get-MgUserLicenseDetail -UserId <user@domain.com>` |
| Check tenant SKUs | `Get-MgSubscribedSku | Select-Object SkuPartNumber,ConsumedUnits,PrepaidUnits` |
| Check group | `Get-MgGroup -Filter "displayName eq '<group-name>'"` |
| Check group members | `Get-MgGroupMember -GroupId <group-id> -All` |
| Check group owners | `Get-MgGroupOwner -GroupId <group-id> -All` |
| Check service health issues | `Get-MgServiceAnnouncementIssue -Top 20` |
| Check message center | `Get-MgServiceAnnouncementMessage -Top 20` |
| Disconnect Graph | `Disconnect-MgGraph` |

### Exchange Online PowerShell

| Purpose | PowerShell |
|---|---|
| Connect to Exchange Online | `Connect-ExchangeOnline` |
| Check recipient | `Get-Recipient <user-or-group@domain.com> | Format-List Name,RecipientType,RecipientTypeDetails,PrimarySmtpAddress,EmailAddresses` |
| Check mailbox | `Get-EXOMailbox <user@domain.com> -Properties RecipientTypeDetails,PrimarySmtpAddress,EmailAddresses,ArchiveStatus,ForwardingAddress,ForwardingSmtpAddress,HiddenFromAddressListsEnabled` |
| Check mailbox statistics | `Get-EXOMailboxStatistics <user@domain.com> | Select-Object DisplayName,TotalItemSize,ItemCount,LastLogonTime` |
| Check mailbox permissions | `Get-MailboxPermission <mailbox@domain.com> | Where-Object {$_.IsInherited -eq $false}` |
| Check SendAs | `Get-RecipientPermission <mailbox@domain.com>` |
| Check Send on Behalf | `Get-EXOMailbox <mailbox@domain.com> -Properties GrantSendOnBehalfTo | Select-Object GrantSendOnBehalfTo` |
| Check inbox rules | `Get-InboxRule -Mailbox <user@domain.com>` |
| Check forwarding | `Get-EXOMailbox <user@domain.com> -Properties ForwardingAddress,ForwardingSmtpAddress,DeliverToMailboxAndForward | Select-Object ForwardingAddress,ForwardingSmtpAddress,DeliverToMailboxAndForward` |
| Check accepted domain | `Get-AcceptedDomain | Select-Object Name,DomainName,DomainType,Default` |
| Trace inbound mail | `Get-MessageTrace -RecipientAddress <user@domain.com> -StartDate (Get-Date).AddHours(-24) -EndDate (Get-Date)` |
| Trace outbound mail | `Get-MessageTrace -SenderAddress <user@domain.com> -StartDate (Get-Date).AddHours(-24) -EndDate (Get-Date)` |
| Trace details | `Get-MessageTraceDetail -MessageTraceId <trace-id> -RecipientAddress <recipient@domain.com>` |
| Check distribution group | `Get-DistributionGroup <group@domain.com> | Format-List` |
| Check distribution group members | `Get-DistributionGroupMember <group@domain.com>` |
| Check Microsoft 365 group | `Get-UnifiedGroup <group@domain.com> | Format-List` |
| Check M365 group members | `Get-UnifiedGroupLinks <group@domain.com> -LinkType Members` |
| Check M365 group owners | `Get-UnifiedGroupLinks <group@domain.com> -LinkType Owners` |
| Check dynamic group | `Get-DynamicDistributionGroup <group-name> | Format-List Name,RecipientFilter,IncludedRecipients,Conditional*` |
| Preview dynamic group | `Get-Recipient -RecipientPreviewFilter (Get-DynamicDistributionGroup <group-name>).RecipientFilter` |
| Disconnect Exchange Online | `Disconnect-ExchangeOnline` |

### Purview / Audit PowerShell

| Purpose | PowerShell |
|---|---|
| Connect to Exchange Online for audit search cmdlets | `Connect-ExchangeOnline` |
| Search unified audit log for user | `Search-UnifiedAuditLog -StartDate (Get-Date).AddDays(-7) -EndDate (Get-Date) -UserIds <user@domain.com>` |
| Search unified audit log broadly | `Search-UnifiedAuditLog -StartDate (Get-Date).AddDays(-7) -EndDate (Get-Date) -Operations "Set-Mailbox","Add-MailboxPermission","Set-User","Set-MsolUserLicense","Update user","Add member to group"` |
| Search mailbox permission changes | `Search-UnifiedAuditLog -StartDate (Get-Date).AddDays(-7) -EndDate (Get-Date) -Operations "Add-MailboxPermission","Remove-MailboxPermission","Add-RecipientPermission","Remove-RecipientPermission"` |
| Search group changes | `Search-UnifiedAuditLog -StartDate (Get-Date).AddDays(-7) -EndDate (Get-Date) -Operations "Add member to group","Remove member from group","Update group"` |

---

## 05_Troubleshoot_M365_User_Mailbox_Group_License_And_Service_Health_Issues_Known_Good_Baselines

### User Baseline

| Setting | Target State |
|---|---|
| Account enabled | True |
| UPN | Matches sign-in address |
| Usage location | Set before license assignment |
| User type | Member for internal employees unless guest is intended |
| Assigned licenses | Matches role/job/service requirement |
| Required service plans | Enabled |
| Sign-in logs | No repeated failure from CA/MFA/license issue |
| Proxy addresses | Correct primary SMTP and aliases |
| Admin role | Only assigned if required |

### Mailbox Baseline

| Setting | Target State |
|---|---|
| Recipient exists | `Get-Recipient` returns expected object |
| Mailbox exists | `Get-EXOMailbox` returns expected mailbox |
| Primary SMTP | Correct domain and address |
| Email aliases | Required aliases present |
| Hidden from GAL | False unless intentionally hidden |
| Forwarding | Disabled unless approved |
| Inbox rules | No suspicious forwarding/delete rules |
| Quota | Not at prohibit send/receive |
| Archive | Enabled only if required |
| Litigation hold/retention | Matches compliance requirement |
| Permissions | Least privilege delegated access |

### Group Baseline

| Group Type | Baseline Checks |
|---|---|
| Microsoft 365 group | Owners present, members correct, visibility expected, external sender setting expected |
| Distribution group | Members correct, delivery restrictions expected, moderation expected, GAL visibility expected |
| Mail-enabled security group | Used where security and mail delivery are both required |
| Security group | Used for access/licensing, not mail delivery unless mail-enabled |
| Dynamic distribution group | Recipient filter documented and previewed |
| Group-based licensing group | License errors monitored and remediated |

### Licensing Baseline

| Setting | Target State |
|---|---|
| Available licenses | Consumed units below purchased/enabled units |
| Usage location | Set for licensed users |
| Direct license assignment | Used intentionally and documented |
| Group-based license assignment | Preferred for repeatable population-based licensing |
| Service plans | Required apps enabled, unnecessary plans disabled only by policy |
| Conflicting plans | Resolved before assignment |
| License errors | Reviewed on group licensing blade |
| Departed users | Licenses reclaimed during offboarding |

### Service Health Baseline

| Setting | Target State |
|---|---|
| Service health access | Service Support Admin / Global Reader can view incidents |
| Message center access | Owners can review planned changes |
| Incident process | Known incident communicated before local changes |
| Tenant-wide symptoms | Service health checked first |
| Advisory tracking | Relevant advisories captured in incident record |
| Post-incident validation | Affected workload retested after resolution |

---

## 05_Troubleshoot_M365_User_Mailbox_Group_License_And_Service_Health_Issues_Rollback

### Rollback Principles

- Capture user, mailbox, group, and license state before modifying.
- Avoid broad license changes across many users until the affected population is confirmed.
- Do not remove group-based licensing without documenting direct/inherited impacts.
- Do not delete mailboxes, groups, aliases, or licenses as a first troubleshooting step.
- Revert temporary mailbox permissions after testing.
- Remove temporary forwarding or transport exceptions after incident.
- Preserve message trace, audit logs, and screenshots.
- For service incidents, avoid making permanent tenant changes to work around temporary platform behavior unless approved.

### Rollback Table

| Change Made | Rollback Action | Verification |
|---|---|---|
| Assigned temporary license | Remove license after permanent license path is fixed | User license detail no longer shows temporary SKU |
| Enabled service plan temporarily | Restore approved service plan state | Required apps still work or change is documented |
| Changed usage location | Restore only if previous value was wrong for compliance/tax reasons | User license remains valid |
| Added mailbox FullAccess | Remove permission after test | `Get-MailboxPermission` no longer shows delegate |
| Added SendAs | Remove recipient permission | `Get-RecipientPermission` no longer shows trustee |
| Added Send on Behalf | Remove delegate from GrantSendOnBehalfTo | Mailbox property no longer lists delegate |
| Changed forwarding | Restore previous forwarding state | `Get-EXOMailbox` forwarding properties match baseline |
| Disabled inbox rule | Re-enable only if legitimate | Rule state matches approved user config |
| Added group member | Remove member if temporary | Group membership returns to baseline |
| Changed group delivery restriction | Restore previous restriction | Group properties match baseline |
| Changed hidden-from-GAL setting | Restore previous visibility if temporary | Address list visibility matches design |
| Created direct license to bypass group error | Remove direct license once group licensing fixed | User license assignment path is clean |
| Changed transport rule | Restore prior rule state | Mail flow rule priority/state matches pre-change |
| Removed quarantine/restriction | Reapply if release was temporary and investigation requires it | Defender state matches security decision |

### Pre-Change Capture Commands

| Purpose | PowerShell |
|---|---|
| Capture user state | `Get-MgUser -UserId <user@domain.com> -Property * | ConvertTo-Json -Depth 10 | Out-File ".\user_before.json"` |
| Capture user license detail | `Get-MgUserLicenseDetail -UserId <user@domain.com> | ConvertTo-Json -Depth 10 | Out-File ".\license_before.json"` |
| Capture mailbox state | `Get-EXOMailbox <user@domain.com> -Properties * | ConvertTo-Json -Depth 10 | Out-File ".\mailbox_before.json"` |
| Capture mailbox permissions | `Get-MailboxPermission <mailbox@domain.com> | ConvertTo-Json -Depth 10 | Out-File ".\mailbox_permissions_before.json"` |
| Capture SendAs permissions | `Get-RecipientPermission <mailbox@domain.com> | ConvertTo-Json -Depth 10 | Out-File ".\recipient_permissions_before.json"` |
| Capture distribution group | `Get-DistributionGroup <group@domain.com> | ConvertTo-Json -Depth 10 | Out-File ".\distribution_group_before.json"` |
| Capture unified group | `Get-UnifiedGroup <group@domain.com> | ConvertTo-Json -Depth 10 | Out-File ".\unified_group_before.json"` |
| Capture group members | `Get-MgGroupMember -GroupId <group-id> -All | ConvertTo-Json -Depth 10 | Out-File ".\group_members_before.json"` |
| Capture message trace | `Get-MessageTrace -RecipientAddress <user@domain.com> -StartDate (Get-Date).AddHours(-24) -EndDate (Get-Date) | Export-Csv ".\message_trace_before.csv" -NoTypeInformation` |

---

## 05_Troubleshoot_M365_User_Mailbox_Group_License_And_Service_Health_Issues_Failure_Checks

| Failure | Check | Fix |
|---|---|---|
| User has license but app missing | Service plan disabled or propagation delay | Enable service plan or wait after assignment |
| License assignment fails | Usage location missing, no available licenses, conflict | Set usage location, free license, resolve conflict |
| Group-based license not applying | User not in group, inherited conflict, licensing error | Check group licensing errors and membership |
| Mailbox missing after license assigned | Provisioning delay, Exchange plan disabled, sync issue | Confirm Exchange service plan and wait/check recipient |
| Get-EXOMailbox fails but Get-Recipient works | Recipient is not mailbox-enabled or object type differs | Check RecipientTypeDetails |
| User cannot receive mail | Address missing, mailbox full, transport rule, quarantine, accepted domain issue | Check recipient, quota, trace, quarantine |
| User cannot send mail | Mailbox quota, restricted entity, outbound spam policy, license issue | Check quota, trace, Defender restricted entities |
| SendAs fails after permission added | Token cache or wrong permission type | Reopen Outlook/OWA and confirm SendAs vs SendOnBehalf |
| FullAccess works in OWA but not Outlook | Outlook profile/automapping cache | Recreate profile or use automapping false |
| Shared mailbox asks for license | Size over limit, archive, hold, or direct sign-in use case | Assign license if required by scenario |
| Distribution group blocks external senders | RequireSenderAuthenticationEnabled is True | Set to False if external senders are approved |
| Group mail moderated unexpectedly | Moderation enabled | Review moderation settings |
| Dynamic group membership wrong | User attribute wrong or recipient filter wrong | Fix attributes or filter |
| M365 group hidden in Outlook | HiddenFromExchangeClientsEnabled True | Set visibility to false if approved |
| Service health shows incident | Platform issue | Communicate and avoid unnecessary tenant changes |
| Admin center stale | Reporting delay or service advisory | Check report timestamp and service health |
| Audit search returns no records | Audit not available, wrong time range, insufficient role | Expand range and check permissions |
| Message trace returns no result | Wrong sender/recipient/time range or message never reached EXO | Expand range and verify MX/source |
| User changes revert | On-premises sync overwrites cloud value | Fix source of authority on-premises |
| Proxy address cannot be changed | Attribute synced from on-premises or duplicate proxy | Fix source object or duplicate address |
| Group membership change delayed | Dynamic group processing or token cache | Wait, refresh token, verify rule |
| User signs in but portal denies access | License/service plan or role missing | Check license, plan, and admin role |

---

## 05_Troubleshoot_M365_User_Mailbox_Group_License_And_Service_Health_Issues_Escalation_Handoff

### Escalate To Identity Team When

- User object is synced from on-premises and cloud changes revert
- Proxy address conflict exists
- UPN, source anchor, duplicate object, or sync error is involved
- Sign-in logs show Conditional Access or MFA failure
- Group membership is controlled by dynamic rules or on-premises sync
- User is a guest/member mismatch

### Escalate To Exchange / Messaging Team When

- Mailbox provisioning fails
- Message trace shows transport failure
- Mailbox permissions, delegation, or shared mailbox behavior is broken
- Distribution group or dynamic distribution group behavior is incorrect
- Mail flow rules, connectors, quarantine, or restricted entities are involved
- Archive, retention, litigation hold, or mailbox restore is involved

### Escalate To Licensing / M365 Admin Team When

- Tenant license pool is exhausted
- Group-based licensing errors affect many users
- Service plans are disabled by policy
- License changes affect multiple workloads
- New SKU mapping or product transition is required

### Escalate To Security / Compliance Team When

- User is blocked as a restricted entity
- Suspicious inbox forwarding or malicious rules are found
- Audit logs show unauthorized mailbox or group changes
- DLP, retention, eDiscovery, or audit configuration affects user experience
- Mail is quarantined due to threat policy

### Escalate To Microsoft Support When

- Service health does not show incident but many users are affected
- Mailbox provisioning is stuck after correct license and service plan are assigned
- Exchange recipient state is inconsistent between portals and PowerShell
- Group-based licensing errors do not clear after fixing membership and conflicts
- Message trace or audit log data is missing unexpectedly
- Admin center reports incorrect state across long periods

### Handoff Data To Include

| Field | Value |
|---|---|
| Incident ID | `<ticket-id>` |
| Tenant ID | `<tenant-id>` |
| Affected workload | `<workload>` |
| Affected user | `<user@domain.com>` |
| Affected mailbox | `<mailbox@domain.com>` |
| Affected group | `<group@domain.com>` |
| Group type | `<type>` |
| License SKU | `<sku>` |
| Service plan | `<plan>` |
| Error message | `<error>` |
| First failure time | `<timestamp>` |
| Service health incident ID | `<incident-id>` |
| Message trace ID | `<trace-id>` |
| Audit log event | `<event>` |
| Recent changes | `<changes>` |
| Remediation attempted | `<actions>` |
| Current blocker | `<blocker>` |
| Business impact | `<impact>` |

---

## 05_Troubleshoot_M365_User_Mailbox_Group_License_And_Service_Health_Issues_Incident_Record_Template

| Field | Entry |
|---|---|
| Date opened | `<yyyy-mm-dd>` |
| Opened by | `<name>` |
| Affected user | `<user@domain.com>` |
| Affected mailbox | `<mailbox@domain.com>` |
| Affected group | `<group@domain.com>` |
| Affected workload | `<workload>` |
| Symptom | `<symptom>` |
| Root cause category | `<User / Mailbox / Group / License / Service Health / Mail Flow / Permissions / Audit / Sync>` |
| Root cause | `<confirmed-root-cause>` |
| Evidence | `<screenshots, cmdlet output, message trace, audit event, service incident>` |
| Fix applied | `<fix>` |
| Security impact | `<impact>` |
| Mail flow impact | `<impact>` |
| Licensing impact | `<impact>` |
| Rollback action | `<rollback>` |
| Verification performed | `<verification>` |
| Preventive action | `<preventive-action>` |
| Closed by | `<name>` |
| Date closed | `<yyyy-mm-dd>` |

---

## 05_Troubleshoot_M365_User_Mailbox_Group_License_And_Service_Health_Issues_Preventive_Controls

| Control | Target State |
|---|---|
| User lifecycle process | Joiner/mover/leaver workflow controls users, licenses, groups, and mailboxes |
| License inventory | License capacity and consumption reviewed regularly |
| Group-based licensing | Repeatable license assignment through controlled groups |
| Usage location baseline | Required users have usage location set before licensing |
| Mailbox provisioning validation | New users validated for mailbox creation after license assignment |
| Proxy address hygiene | Duplicate and stale proxy addresses remediated |
| Group ownership | Groups have at least two owners where appropriate |
| Distribution group restrictions | External sender, moderation, and delivery restrictions documented |
| Mailbox delegation review | Shared mailbox and SendAs permissions reviewed periodically |
| Forwarding monitoring | External forwarding and suspicious inbox rules monitored |
| Service health review | Service health checked before tenant-wide troubleshooting |
| Message center review | Planned changes reviewed and communicated |
| Audit logging | User, mailbox, group, and license changes searchable |
| Incident documentation | Root cause, fix, rollback, and prevention recorded |

---

## 05_Troubleshoot_M365_User_Mailbox_Group_License_And_Service_Health_Issues_Related_Labs

| Lab | Purpose |
|---|---|
| Troubleshoot Microsoft 365 user sign-in and license access | Validate user, license, and service plan gates |
| Troubleshoot mailbox provisioning after license assignment | Confirm Exchange mailbox creation |
| Troubleshoot mailbox send and receive issues | Use message trace, recipient lookup, and mailbox state |
| Troubleshoot shared mailbox access | Validate FullAccess, SendAs, Send on Behalf, and automapping |
| Troubleshoot distribution group delivery | Validate restrictions, moderation, external sender settings, and trace |
| Troubleshoot Microsoft 365 group visibility and membership | Validate owners, members, subscribers, and hidden flags |
| Troubleshoot dynamic distribution group membership | Validate recipient filter and user attributes |
| Troubleshoot group-based licensing | Validate membership, license assignment, and service plan conflicts |
| Review Microsoft 365 service health and message center | Distinguish tenant config from platform issue |
| Search Microsoft 365 audit logs for admin changes | Identify recent mailbox, group, or license changes |
| Create Microsoft 365 incident review record | Document evidence, root cause, fix, and rollback |
