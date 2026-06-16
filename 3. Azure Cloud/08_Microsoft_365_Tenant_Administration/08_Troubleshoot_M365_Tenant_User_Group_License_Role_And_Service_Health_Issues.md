08_Troubleshoot_M365_Tenant_User_Group_License_Role_And_Service_Health_Issues.md
# 08_Troubleshoot_M365_Tenant_User_Group_License_Role_And_Service_Health_Issues

# 08_Troubleshoot_M365_Tenant_User_Group_License_Role_And_Service_Health_Issues_Index
08_Troubleshoot_M365_Tenant_User_Group_License_Role_And_Service_Health_Issues.md
08_Troubleshoot_M365_Tenant_User_Group_License_Role_And_Service_Health_Issues
08_Troubleshoot_M365_Tenant_User_Group_License_Role_And_Service_Health_Issues_Source_Basis
08_Troubleshoot_M365_Tenant_User_Group_License_Role_And_Service_Health_Issues_Mental_Model
08_Troubleshoot_M365_Tenant_User_Group_License_Role_And_Service_Health_Issues_Planning_Table
08_Troubleshoot_M365_Tenant_User_Group_License_Role_And_Service_Health_Issues_Configuration_Checklist
08_Troubleshoot_M365_Tenant_User_Group_License_Role_And_Service_Health_Issues_Triage_Intake_Skeleton
08_Troubleshoot_M365_Tenant_User_Group_License_Role_And_Service_Health_Issues_Service_Health_Skeleton
08_Troubleshoot_M365_Tenant_User_Group_License_Role_And_Service_Health_Issues_User_Sign_In_And_Profile_Skeleton
08_Troubleshoot_M365_Tenant_User_Group_License_Role_And_Service_Health_Issues_Domain_UPN_And_DNS_Skeleton
08_Troubleshoot_M365_Tenant_User_Group_License_Role_And_Service_Health_Issues_Group_And_Shared_Mailbox_Skeleton
08_Troubleshoot_M365_Tenant_User_Group_License_Role_And_Service_Health_Issues_License_And_Service_Plan_Skeleton
08_Troubleshoot_M365_Tenant_User_Group_License_Role_And_Service_Health_Issues_Role_And_Delegation_Skeleton
08_Troubleshoot_M365_Tenant_User_Group_License_Role_And_Service_Health_Issues_Graph_Export_Skeleton
08_Troubleshoot_M365_Tenant_User_Group_License_Role_And_Service_Health_Issues_Escalation_Skeleton
08_Troubleshoot_M365_Tenant_User_Group_License_Role_And_Service_Health_Issues_Verification_Commands
08_Troubleshoot_M365_Tenant_User_Group_License_Role_And_Service_Health_Issues_Rollback
08_Troubleshoot_M365_Tenant_User_Group_License_Role_And_Service_Health_Issues_Failure_Checks
08_Troubleshoot_M365_Tenant_User_Group_License_Role_And_Service_Health_Issues_Related_Labs

# 08_Troubleshoot_M365_Tenant_User_Group_License_Role_And_Service_Health_Issues_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Microsoft 365 service health | Confirming whether an issue is caused by Microsoft service incidents or advisories |
| Microsoft Learn | Microsoft 365 Message center | Identifying planned changes, retirements, major updates, and tenant-impacting changes |
| Microsoft Learn | Microsoft 365 admin center Users area | Troubleshooting active users, deleted users, guest users, sign-in state, profile state, and user filters |
| Microsoft Learn | Add users and assign licenses | User provisioning, usage location, license assignment, and common license errors |
| Microsoft Learn | Manage groups and shared mailboxes | Troubleshooting Microsoft 365 groups, distribution groups, mail-enabled security groups, and shared mailboxes |
| Microsoft Learn | Microsoft 365 licensing and group-based licensing | License inventory, direct license assignment, group license assignment, reprocessing, and group license errors |
| Microsoft Learn | Microsoft 365 admin roles and Microsoft Entra roles | Troubleshooting missing permissions, role assignments, scoped admin units, and delegation failures |
| Microsoft Learn | Microsoft Graph PowerShell | Repeatable tenant, user, group, license, role, service health, and report investigation |
| Microsoft Learn | Exchange Online PowerShell | Troubleshooting distribution groups, mail contacts, shared mailboxes, accepted domains, and Exchange RBAC |
| Microsoft 365 operational practice | Incident triage and evidence collection | Separates Microsoft outage, tenant configuration, identity, licensing, DNS, group, and role root causes |

# 08_Troubleshoot_M365_Tenant_User_Group_License_Role_And_Service_Health_Issues_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Tenant issue | Problem affecting broad Microsoft 365 administration, workload access, service state, or multiple users |
| User issue | Problem tied to one user object, sign-in state, profile, license, mailbox, group membership, or source of authority |
| Group issue | Problem tied to membership, ownership, group type, mail flow, access assignment, or group-based licensing |
| License issue | Problem tied to missing usage location, no available SKU, disabled service plan, conflicting SKU, or bad group scope |
| Role issue | Problem tied to missing admin role, wrong role, scoped role, stale role assignment, or workload-specific RBAC |
| Service health issue | Microsoft-side incident or advisory affecting the tenant or workload |
| Message center issue | Planned Microsoft change or retirement that explains changed behavior |
| Source of authority | Where the object must be fixed: cloud, on-prem AD, Entra Connect, HR system, Exchange, or another workload |
| Direct assignment | License, role, permission, or access assigned directly to a user |
| Inherited assignment | License, role, permission, or access inherited from group membership |
| Scoped delegation | Admin permission limited by administrative unit or workload scope |
| Service plan | Workload feature inside a license SKU, such as Exchange, Teams, SharePoint, Intune, or Entra feature |
| Provisioning delay | Delay between assignment and service availability |
| Soft-deleted object | Deleted user, group, or mailbox that may still be recoverable |
| Token/session issue | User access appears wrong because old tokens or sessions have not refreshed |
| First rule | Check Service health before tearing apart tenant configuration |
| Blunt rule | If multiple unrelated users break at once, assume service health, license exhaustion, Conditional Access, DNS, or role/config change before blaming one account |

# 08_Troubleshoot_M365_Tenant_User_Group_License_Role_And_Service_Health_Issues_Planning_Table
| Item | Example | Decision |
|---|---|---|
| Tenant name | `contoso.onmicrosoft.com` | `<tenant>.onmicrosoft.com` |
| Tenant ID | `00000000-0000-0000-0000-000000000000` | `<tenant-id>` |
| Admin account | `admin@contoso.com` | `<admin-upn>` |
| Affected user | `user@contoso.com` | `<affected-user-upn>` |
| Affected guest | `guest_example.com#EXT#@contoso.onmicrosoft.com` | `<affected-guest-upn>` |
| Affected group | `M365_Project_Alpha` | `<affected-group-name>` |
| Affected shared mailbox | `helpdesk@contoso.com` | `<shared-mailbox-email>` |
| Affected license SKU | `SPE_E3` | `<sku-part-number>` |
| Affected workload | Exchange, Teams, SharePoint, OneDrive, Admin center | `<workload>` |
| Symptom summary | User cannot access Teams | `<symptom>` |
| Impact scope | One user / group / department / whole tenant | `<impact-scope>` |
| Start time | `2026-06-16 09:00` | `<start-time>` |
| Last known good | `2026-06-15 17:00` | `<last-known-good>` |
| Recent change | License group update | `<recent-change>` |
| Service health checked | Yes | `<yes-no>` |
| Message center checked | Yes | `<yes-no>` |
| Source of authority | Cloud-only / Synced / Hybrid | `<source-of-authority>` |
| Evidence path | `.\Evidence\M365-Troubleshooting` | `<evidence-path>` |
| Escalation owner | `m365ops@contoso.com` | `<escalation-owner>` |
| Rollback owner | `identityadmin@contoso.com` | `<rollback-owner>` |
| Microsoft support case | `Case #123456789` | `<support-case>` |

# 08_Troubleshoot_M365_Tenant_User_Group_License_Role_And_Service_Health_Issues_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm correct admin account | Admin Workstation | Portal account menu | Correct admin UPN is signed in |
| 2 | Confirm correct tenant | Admin Workstation | Portal tenant switcher | Correct tenant is active |
| 3 | Record symptom and impact | Operator | Intake notes | Issue scope and affected objects are documented |
| 4 | Record start time and last known good | Operator | Ticket notes | Timeline is known |
| 5 | Check Microsoft 365 Service health | Admin Workstation | Admin center > Health > Service health | Active incidents and advisories are reviewed |
| 6 | Check Message center | Admin Workstation | Admin center > Health > Message center | Planned changes are reviewed |
| 7 | Check network connectivity insights if access or performance issue | Admin Workstation | Admin center > Health > Network connectivity | Network recommendations are reviewed |
| 8 | Confirm whether issue affects one user or many | Operator | User reports / help desk ticket count | Scope is classified |
| 9 | Connect Microsoft Graph PowerShell | Admin Workstation | `Connect-MgGraph -Scopes "User.ReadWrite.All","Group.ReadWrite.All","Directory.ReadWrite.All","Organization.Read.All","RoleManagement.Read.Directory","ServiceHealth.Read.All","ServiceMessage.Read.All","Reports.Read.All"` | Graph session connects |
| 10 | Confirm Graph tenant context | Admin Workstation | `Get-MgContext` | Correct tenant ID and scopes are visible |
| 11 | Export tenant SKU inventory | Admin Workstation | `Get-MgSubscribedSku` | License availability is known |
| 12 | Export service health issues | Admin Workstation | `Get-MgAdminServiceAnnouncementIssue -All` | Microsoft incidents are available for evidence |
| 13 | Export Message center messages | Admin Workstation | `Get-MgAdminServiceAnnouncementMessage -All` | Recent Microsoft changes are available |
| 14 | Confirm affected user exists | Admin Workstation | `Get-MgUser -UserId "<affected-user-upn>"` | User object is found |
| 15 | Confirm affected user type | Admin Workstation | `Get-MgUser -UserId "<affected-user-upn>" -Property UserType` | Member or Guest state is known |
| 16 | Confirm affected user account enabled state | Admin Workstation | `Get-MgUser -UserId "<affected-user-upn>" -Property AccountEnabled` | Sign-in block state is known |
| 17 | Confirm affected user usage location | Admin Workstation | `Get-MgUser -UserId "<affected-user-upn>" -Property UsageLocation` | Usage location is populated or missing |
| 18 | Confirm affected user license detail | Admin Workstation | `Get-MgUserLicenseDetail -UserId "<affected-user-upn>"` | Assigned SKUs and service plans are visible |
| 19 | Confirm affected user group memberships | Admin Workstation | `Get-MgUserMemberOf -UserId "<affected-user-upn>"` | Group-based access path is visible |
| 20 | Check if affected user is soft-deleted | Admin Workstation | `Get-MgDirectoryDeletedItemAsUser -All` | Deleted user conflict is ruled in or out |
| 21 | Confirm domain verification state | Admin Workstation | `Get-MgDomain` | Custom domain state is visible |
| 22 | Validate DNS records if sign-in/mail/domain issue | Admin Workstation | `nslookup -type=TXT <custom-domain>` and `nslookup -type=MX <custom-domain>` | Public DNS state is known |
| 23 | Confirm affected group exists | Admin Workstation | `Get-MgGroup -Filter "displayName eq '<affected-group-name>'"` | Group object is found |
| 24 | Confirm affected group type | Admin Workstation | `Get-MgGroup` | Microsoft 365, security, or mail-enabled state is known |
| 25 | Confirm group owners | Admin Workstation | `Get-MgGroupOwner -GroupId "<group-id>"` | Group owner state is known |
| 26 | Confirm group members | Admin Workstation | `Get-MgGroupMember -GroupId "<group-id>"` | Group membership is visible |
| 27 | Confirm group license assignment if licensing issue | Admin Workstation | `Get-MgGroupLicenseDetail -GroupId "<group-id>"` | Group-based license state is visible |
| 28 | Connect Exchange Online if mail/group/shared mailbox issue | Admin Workstation | `Connect-ExchangeOnline -UserPrincipalName "<admin-upn>"` | Exchange Online session connects |
| 29 | Check accepted domain if mail issue | Admin Workstation | `Get-AcceptedDomain` | Mail domain acceptance is known |
| 30 | Check distribution group if mail list issue | Admin Workstation | `Get-DistributionGroup -Identity "<group-name>"` | Distribution group state is visible |
| 31 | Check distribution group members | Admin Workstation | `Get-DistributionGroupMember -Identity "<group-name>"` | Recipient membership is visible |
| 32 | Check shared mailbox if delegated mailbox issue | Admin Workstation | `Get-EXOMailbox -Identity "<shared-mailbox-email>"` | Shared mailbox exists |
| 33 | Check shared mailbox Full Access | Admin Workstation | `Get-MailboxPermission -Identity "<shared-mailbox-email>"` | Full Access delegates are visible |
| 34 | Check shared mailbox Send As | Admin Workstation | `Get-RecipientPermission -Identity "<shared-mailbox-email>"` | Send As delegates are visible |
| 35 | Check role assignment if admin permission issue | Admin Workstation | `Get-MgRoleManagementDirectoryRoleAssignment -All` | Role assignment evidence is available |
| 36 | Check admin unit scope if delegated admin issue | Admin Workstation | `Get-MgDirectoryAdministrativeUnit -All` | Admin unit scope is visible |
| 37 | Check Exchange RBAC if Exchange permission issue | Admin Workstation | `Get-RoleGroupMember` and `Get-ManagementRoleAssignment` | Exchange role group and scope are known |
| 38 | Identify recent change | Operator | Change records / Message center / audit notes | Probable trigger is documented |
| 39 | Classify root cause area | Operator | Triage notes | Issue is mapped to service health, user, group, license, role, DNS, or workload |
| 40 | Apply lowest-risk fix | Admin Workstation | Relevant skeleton below | Issue is corrected without unnecessary tenant-wide change |
| 41 | Validate fix with affected user or admin | Test Client | Portal sign-in, workload access, mail test, or admin action | Symptom is resolved |
| 42 | Capture final evidence | Admin Workstation | Export commands | Before/after state is saved |
| 43 | Document rollback path | Operator | Notes | Undo plan is known |
| 44 | Escalate if Microsoft-side or unresolved | Operator | Help and support / support case | Support path is initiated |
| 45 | Document closure | Operator | Notes | Root cause, fix, validation, and follow-up are recorded |

# 08_Troubleshoot_M365_Tenant_User_Group_License_Role_And_Service_Health_Issues_Triage_Intake_Skeleton
Use this before changing anything.

| Field | Value |
|---|---|
| Ticket number | `<ticket-id>` |
| Reported by | `<reporter>` |
| Affected user(s) | `<affected-users>` |
| Affected group(s) | `<affected-groups>` |
| Affected workload | `<workload>` |
| Symptom | `<symptom>` |
| Error message | `<exact-error>` |
| Start time | `<start-time>` |
| Last known good | `<last-known-good>` |
| Scope | `<one-user/multiple-users/tenant-wide>` |
| Recent change | `<change-summary>` |
| Device/browser/client | `<client-info>` |
| Network path | `<office/home/vpn/proxy>` |
| Service health checked | `<yes-no>` |
| Message center checked | `<yes-no>` |
| Evidence path | `<evidence-path>` |
| Initial severity | `<sev>` |

Triage decision table:

| Symptom Pattern | Start Here |
|---|---|
| Many users affected at once | Service health, Message center, license availability, Conditional Access, DNS |
| One user affected | User object, license, group membership, sign-in, mailbox, usage location |
| One group affected | Group type, membership, owner, mail settings, access assignment |
| Users cannot access one app | License service plan, app assignment, Conditional Access, service health |
| Admin cannot perform task | Admin role, admin unit scope, Exchange RBAC, PIM activation |
| Mail not flowing | Service health, accepted domain, MX, SPF, Exchange recipient, distribution group settings |
| Shared mailbox not opening | Full Access permission, automapping, mailbox existence, Outlook cache |
| User cannot send as mailbox | Send As permission, propagation delay, Outlook session |
| License assignment fails | Usage location, license availability, group license error, conflicting plan |
| Portal looks different or feature missing | Role, license, release ring, Message center change, service availability |

Operational note:

```text
Do not fix before scoping.
A one-user issue and a tenant-wide incident have completely different root-cause paths.
```

# 08_Troubleshoot_M365_Tenant_User_Group_License_Role_And_Service_Health_Issues_Service_Health_Skeleton
```text
Portal path:

Microsoft 365 admin center
Health
Service health
```

Service health checklist:

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Open Service health | Admin Workstation | Admin center > Health > Service health | Service health page opens |
| 2 | Review active incidents | Admin Workstation | Active issues | Current incidents are visible |
| 3 | Review active advisories | Admin Workstation | Active advisories | Current advisories are visible |
| 4 | Search affected workload | Admin Workstation | Service health filter/search | Workload status is known |
| 5 | Open issue details | Admin Workstation | Select issue | Scope, impact, start time, and latest update are visible |
| 6 | Review history | Admin Workstation | Service health > History | Recent resolved issues are visible |
| 7 | Compare issue timeline with ticket timeline | Operator | Notes | Microsoft incident correlation is known |
| 8 | Export service health evidence | Admin Workstation | Graph commands below | Incident evidence is saved |
| 9 | Notify help desk if Microsoft-side issue | Operator | Internal comms | Help desk stops treating it as isolated user issue |
| 10 | Escalate to Microsoft support if needed | Admin Workstation | Help and support | Support case is opened if appropriate |

Graph service health export:

```powershell
# Run from an admin workstation.
# Purpose: export Microsoft 365 service health and Message center evidence.

$EvidencePath = ".\Evidence\M365-Troubleshooting\ServiceHealth"
New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Connect-MgGraph -Scopes "ServiceHealth.Read.All","ServiceMessage.Read.All","Organization.Read.All"

Get-MgContext |
    Tee-Object "$EvidencePath\graph-context-service-health.txt"

Get-MgAdminServiceAnnouncementHealthOverview |
    Sort-Object Service |
    Format-Table Service,Status,Id -AutoSize |
    Tee-Object "$EvidencePath\service-health-overview.txt"

Get-MgAdminServiceAnnouncementIssue -All |
    Sort-Object LastModifiedDateTime -Descending |
    Select-Object Id,Title,Service,Status,Classification,StartDateTime,LastModifiedDateTime |
    Format-Table -AutoSize |
    Tee-Object "$EvidencePath\service-health-issues.txt"

Get-MgAdminServiceAnnouncementMessage -All |
    Sort-Object LastModifiedDateTime -Descending |
    Select-Object Id,Title,Category,Severity,StartDateTime,EndDateTime,LastModifiedDateTime |
    Format-Table -AutoSize |
    Tee-Object "$EvidencePath\message-center-messages.txt"
```

Service health finding table:

| Workload | Incident ID | Status | Tenant Impact | User Impact | Action |
|---|---|---|---|---|---|
| `<service>` | `<incident-id>` | `<status>` | `<yes-no>` | `<impact>` | `<monitor/escalate/workaround>` |

Operational note:

```text
If Microsoft confirms an active service incident matching the symptom and timeline, do not keep making tenant changes.
Communicate, monitor, collect evidence, and use workarounds only if safe.
```

# 08_Troubleshoot_M365_Tenant_User_Group_License_Role_And_Service_Health_Issues_User_Sign_In_And_Profile_Skeleton
User investigation checklist:

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm user exists | Admin Workstation | `Get-MgUser -UserId "<affected-user-upn>"` | User object is returned |
| 2 | Confirm user type | Admin Workstation | `Get-MgUser -UserId "<affected-user-upn>" -Property UserType` | User is Member or Guest |
| 3 | Confirm account enabled state | Admin Workstation | `Get-MgUser -UserId "<affected-user-upn>" -Property AccountEnabled` | Sign-in block state is visible |
| 4 | Confirm usage location | Admin Workstation | `Get-MgUser -UserId "<affected-user-upn>" -Property UsageLocation` | Usage location is present if licensing is needed |
| 5 | Confirm UPN and mail values | Admin Workstation | `Get-MgUser -UserId "<affected-user-upn>" -Property UserPrincipalName,Mail,ProxyAddresses` | UPN and mail values are visible |
| 6 | Confirm license details | Admin Workstation | `Get-MgUserLicenseDetail -UserId "<affected-user-upn>"` | Assigned SKUs and service plans are visible |
| 7 | Confirm group memberships | Admin Workstation | `Get-MgUserMemberOf -UserId "<affected-user-upn>"` | Access and licensing groups are visible |
| 8 | Check deleted users for conflict | Admin Workstation | `Get-MgDirectoryDeletedItemAsUser -All` | Soft-deleted duplicate is ruled in or out |
| 9 | Confirm source of authority | Admin Workstation | Portal user details / sync status | Cloud-only or synced path is known |
| 10 | Apply fix only at source | Operator | Cloud or on-prem workflow | Object is corrected at the right authority |

User investigation commands:

```powershell
# Run from an admin workstation.
# Purpose: collect user troubleshooting evidence.

$UserPrincipalName = "<affected-user-upn>"
$EvidencePath = ".\Evidence\M365-Troubleshooting\User-$($UserPrincipalName.Replace('@','_'))"
New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Connect-MgGraph -Scopes "User.ReadWrite.All","Directory.ReadWrite.All","Group.Read.All","Organization.Read.All"

Get-MgContext |
    Tee-Object "$EvidencePath\graph-context.txt"

Get-MgUser `
    -UserId $UserPrincipalName `
    -Property Id,DisplayName,UserPrincipalName,Mail,ProxyAddresses,UserType,AccountEnabled,UsageLocation,Department,JobTitle,OfficeLocation,CreatedDateTime |
    Format-List Id,DisplayName,UserPrincipalName,Mail,ProxyAddresses,UserType,AccountEnabled,UsageLocation,Department,JobTitle,OfficeLocation,CreatedDateTime |
    Tee-Object "$EvidencePath\user-profile.txt"

Get-MgUserLicenseDetail -UserId $UserPrincipalName |
    Format-List SkuPartNumber,SkuId,ServicePlans |
    Tee-Object "$EvidencePath\user-license-details.txt"

Get-MgUserMemberOf -UserId $UserPrincipalName |
    ConvertTo-Json -Depth 10 |
    Out-File "$EvidencePath\user-memberof.json"

Get-MgDirectoryDeletedItemAsUser -All |
    Where-Object {$_.UserPrincipalName -eq $UserPrincipalName -or $_.Mail -eq $UserPrincipalName} |
    Format-List * |
    Tee-Object "$EvidencePath\deleted-user-conflicts.txt"
```

Common user fixes:

```powershell
# Unblock sign-in.
Update-MgUser -UserId "<affected-user-upn>" -AccountEnabled:$true

# Block sign-in.
Update-MgUser -UserId "<affected-user-upn>" -AccountEnabled:$false

# Set usage location.
Update-MgUser -UserId "<affected-user-upn>" -UsageLocation "US"

# Reset password for cloud-only user.
$PasswordProfile = @{
    Password = "NewTempPassword!12345"
    ForceChangePasswordNextSignIn = $true
}

Update-MgUser `
    -UserId "<affected-user-upn>" `
    -PasswordProfile $PasswordProfile

# Revoke sign-in sessions.
Revoke-MgUserSignInSession -UserId "<affected-user-upn>"
```

Operational note:

```text
If the user is synced from on-prem AD, fix the owned attributes on-prem and let sync update Microsoft 365.
Do not fight directory synchronization from the cloud portal.
```

# 08_Troubleshoot_M365_Tenant_User_Group_License_Role_And_Service_Health_Issues_Domain_UPN_And_DNS_Skeleton
Use this when the symptom involves sign-in name, custom domain, mail routing, Autodiscover, DNS verification, or domain health.

Domain and DNS checklist:

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm tenant domains | Admin Workstation | `Get-MgDomain` | Domains and verification state are visible |
| 2 | Confirm affected user UPN domain | Admin Workstation | `Get-MgUser -UserId "<affected-user-upn>"` | UPN uses expected verified domain |
| 3 | Confirm custom domain verified | Admin Workstation | `Get-MgDomain -DomainId "<custom-domain>"` | Domain is verified |
| 4 | Check public TXT records | Admin Workstation | `nslookup -type=TXT <custom-domain> 8.8.8.8` | Verification, SPF, and DMARC TXT records are visible |
| 5 | Check public MX records | Admin Workstation | `nslookup -type=MX <custom-domain> 8.8.8.8` | MX destination is visible |
| 6 | Check Autodiscover | Admin Workstation | `nslookup autodiscover.<custom-domain> 8.8.8.8` | Autodiscover target is visible |
| 7 | Check internal DNS if split DNS exists | Internal Client | `nslookup autodiscover.<custom-domain> <internal-dns>` | Internal and external answers are understood |
| 8 | Confirm accepted domain | Admin Workstation | `Get-AcceptedDomain` | Exchange accepts the mail domain |
| 9 | Confirm recipient address | Admin Workstation | `Get-EXORecipient "<affected-user-upn>"` | Recipient has expected addresses |
| 10 | Map DNS issue to rollback or correction | Operator | DNS evidence | Correct DNS record change is known |

Domain and DNS commands:

```powershell
# Graph domain checks.

$Domain = "<custom-domain>"
$EvidencePath = ".\Evidence\M365-Troubleshooting\Domain-$Domain"
New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Connect-MgGraph -Scopes "Domain.Read.All","Organization.Read.All","Directory.Read.All"

Get-MgDomain |
    Sort-Object Id |
    Format-Table Id,IsDefault,IsInitial,IsVerified,AuthenticationType -AutoSize |
    Tee-Object "$EvidencePath\tenant-domains.txt"

Get-MgDomain -DomainId $Domain |
    Format-List * |
    Tee-Object "$EvidencePath\custom-domain-detail.txt"
```

```powershell
# Public DNS checks.

$Domain = "<custom-domain>"
$DnsServer = "8.8.8.8"
$EvidencePath = ".\Evidence\M365-Troubleshooting\Domain-$Domain"

nslookup -type=NS $Domain $DnsServer | Tee-Object "$EvidencePath\ns-records.txt"
nslookup -type=TXT $Domain $DnsServer | Tee-Object "$EvidencePath\txt-records.txt"
nslookup -type=MX $Domain $DnsServer | Tee-Object "$EvidencePath\mx-records.txt"
nslookup "autodiscover.$Domain" $DnsServer | Tee-Object "$EvidencePath\autodiscover.txt"
nslookup "enterpriseregistration.$Domain" $DnsServer | Tee-Object "$EvidencePath\enterpriseregistration.txt"
nslookup "enterpriseenrollment.$Domain" $DnsServer | Tee-Object "$EvidencePath\enterpriseenrollment.txt"
```

```powershell
# Exchange accepted domain and recipient checks.

Connect-ExchangeOnline -UserPrincipalName "<admin-upn>"

Get-AcceptedDomain |
    Sort-Object DomainName |
    Format-Table Name,DomainName,DomainType,Default -AutoSize

Get-EXORecipient -Identity "<affected-user-upn>" |
    Format-List DisplayName,PrimarySmtpAddress,RecipientType,RecipientTypeDetails,EmailAddresses
```

Operational note:

```text
If public DNS is correct but internal clients fail, check split DNS.
If internal DNS hosts the same zone, internal records must also support the Microsoft 365 path.
```

# 08_Troubleshoot_M365_Tenant_User_Group_License_Role_And_Service_Health_Issues_Group_And_Shared_Mailbox_Skeleton
Use this when the issue involves group membership, distribution lists, Microsoft 365 groups, security groups, shared mailboxes, or mailbox delegation.

Group troubleshooting checklist:

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Find group by display name | Admin Workstation | `Get-MgGroup -Filter "displayName eq '<group-name>'"` | Group object is found |
| 2 | Confirm group type | Admin Workstation | `Get-MgGroup` | MailEnabled, SecurityEnabled, and GroupTypes are known |
| 3 | Confirm group owners | Admin Workstation | `Get-MgGroupOwner -GroupId "<group-id>"` | Owners are visible |
| 4 | Confirm group members | Admin Workstation | `Get-MgGroupMember -GroupId "<group-id>"` | Members are visible |
| 5 | Confirm affected user membership | Admin Workstation | Group member export | User is present or missing |
| 6 | Check group-based license state if applicable | Admin Workstation | `Get-MgGroupLicenseDetail -GroupId "<group-id>"` | Group license state is visible |
| 7 | Check distribution group if mail issue | Admin Workstation | `Get-DistributionGroup` | Distribution group state is visible |
| 8 | Check distribution group members | Admin Workstation | `Get-DistributionGroupMember` | Mail list members are visible |
| 9 | Check sender restrictions | Admin Workstation | `Get-DistributionGroup | Format-List RequireSenderAuthenticationEnabled` | External sender state is known |
| 10 | Check shared mailbox permissions | Admin Workstation | `Get-MailboxPermission` and `Get-RecipientPermission` | Full Access and Send As are visible |

Graph group investigation:

```powershell
# Run from an admin workstation.
# Purpose: collect group troubleshooting evidence.

$GroupDisplayName = "<affected-group-name>"
$EvidencePath = ".\Evidence\M365-Troubleshooting\Group-$($GroupDisplayName.Replace(' ','_'))"
New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Connect-MgGraph -Scopes "Group.ReadWrite.All","User.Read.All","Directory.Read.All"

$Group = Get-MgGroup -Filter "displayName eq '$GroupDisplayName'"

$Group |
    Format-List Id,DisplayName,Mail,MailNickname,MailEnabled,SecurityEnabled,GroupTypes,Visibility |
    Tee-Object "$EvidencePath\group-detail.txt"

Get-MgGroupOwner -GroupId $Group.Id |
    ConvertTo-Json -Depth 10 |
    Out-File "$EvidencePath\group-owners.json"

Get-MgGroupMember -GroupId $Group.Id |
    ConvertTo-Json -Depth 10 |
    Out-File "$EvidencePath\group-members.json"

Get-MgGroupLicenseDetail -GroupId $Group.Id -ErrorAction SilentlyContinue |
    Format-List * |
    Tee-Object "$EvidencePath\group-license-detail.txt"
```

Exchange group and shared mailbox investigation:

```powershell
# Run from an admin workstation.
# Purpose: troubleshoot distribution groups, mail-enabled security groups, and shared mailboxes.

$GroupIdentity = "<distribution-group-name>"
$SharedMailbox = "<shared-mailbox-email>"
$EvidencePath = ".\Evidence\M365-Troubleshooting\Exchange-Groups-Mailboxes"

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Connect-ExchangeOnline -UserPrincipalName "<admin-upn>"

Get-DistributionGroup -Identity $GroupIdentity -ErrorAction SilentlyContinue |
    Format-List * |
    Tee-Object "$EvidencePath\distribution-group-detail.txt"

Get-DistributionGroupMember -Identity $GroupIdentity -ErrorAction SilentlyContinue |
    Format-Table Name,PrimarySmtpAddress,RecipientType -AutoSize |
    Tee-Object "$EvidencePath\distribution-group-members.txt"

Get-EXOMailbox -Identity $SharedMailbox -ErrorAction SilentlyContinue |
    Format-List DisplayName,PrimarySmtpAddress,RecipientTypeDetails,HiddenFromAddressListsEnabled |
    Tee-Object "$EvidencePath\shared-mailbox-detail.txt"

Get-MailboxPermission -Identity $SharedMailbox -ErrorAction SilentlyContinue |
    Where-Object {
        $_.User -notlike "NT AUTHORITY\SELF" -and
        $_.IsInherited -eq $false
    } |
    Format-Table User,AccessRights,Deny,IsInherited -AutoSize |
    Tee-Object "$EvidencePath\shared-mailbox-full-access.txt"

Get-RecipientPermission -Identity $SharedMailbox -ErrorAction SilentlyContinue |
    Format-Table Trustee,AccessRights,IsInherited -AutoSize |
    Tee-Object "$EvidencePath\shared-mailbox-send-as.txt"

Get-Mailbox -Identity $SharedMailbox -ErrorAction SilentlyContinue |
    Format-List GrantSendOnBehalfTo |
    Tee-Object "$EvidencePath\shared-mailbox-send-on-behalf.txt"
```

Common group fixes:

```powershell
# Add user to Microsoft 365 or security group.
$Group = Get-MgGroup -Filter "displayName eq '<group-name>'"
$User = Get-MgUser -UserId "<user-upn>"

New-MgGroupMemberByRef `
    -GroupId $Group.Id `
    -BodyParameter @{
        "@odata.id" = "https://graph.microsoft.com/v1.0/directoryObjects/$($User.Id)"
    }

# Add user to distribution group.
Add-DistributionGroupMember `
    -Identity "<distribution-group-name>" `
    -Member "<user-upn>"

# Grant shared mailbox Full Access.
Add-MailboxPermission `
    -Identity "<shared-mailbox-email>" `
    -User "<user-upn>" `
    -AccessRights FullAccess `
    -InheritanceType All `
    -AutoMapping $true

# Grant shared mailbox Send As.
Add-RecipientPermission `
    -Identity "<shared-mailbox-email>" `
    -Trustee "<user-upn>" `
    -AccessRights SendAs `
    -Confirm:$false
```

Operational note:

```text
Do not troubleshoot every group like it is the same object.
Microsoft 365 groups, security groups, distribution groups, mail-enabled security groups, and shared mailboxes have different control planes.
```

# 08_Troubleshoot_M365_Tenant_User_Group_License_Role_And_Service_Health_Issues_License_And_Service_Plan_Skeleton
Use this when the symptom is missing app access, provisioning failure, disabled service, mailbox not created, Teams not available, OneDrive unavailable, or license assignment error.

License troubleshooting checklist:

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Check tenant SKU availability | Admin Workstation | `Get-MgSubscribedSku` | Consumed and enabled units are visible |
| 2 | Check user usage location | Admin Workstation | `Get-MgUser -UserId "<user-upn>" -Property UsageLocation` | Usage location is populated |
| 3 | Check direct user licenses | Admin Workstation | `Get-MgUserLicenseDetail -UserId "<user-upn>"` | Assigned SKUs and plans are visible |
| 4 | Check group memberships | Admin Workstation | `Get-MgUserMemberOf -UserId "<user-upn>"` | Licensing groups are visible or missing |
| 5 | Check group license assignment | Admin Workstation | `Get-MgGroupLicenseDetail -GroupId "<group-id>"` | Inherited license path is visible |
| 6 | Check disabled service plans | Admin Workstation | License detail service plans | Disabled plans are identified |
| 7 | Check license exhaustion | Admin Workstation | `Get-MgSubscribedSku` | Available licenses are known |
| 8 | Check provisioning delay | Admin Workstation | Wait/recheck service workload | License may still be processing |
| 9 | Confirm workload-specific object | Admin Workstation | Exchange/Teams/OneDrive portal | Service object exists or is still provisioning |
| 10 | Apply lowest-risk correction | Admin Workstation | Usage location, group membership, direct license, or service plan correction | Access is restored |

License investigation commands:

```powershell
# Run from an admin workstation.
# Purpose: collect license troubleshooting evidence for one affected user.

$UserPrincipalName = "<affected-user-upn>"
$EvidencePath = ".\Evidence\M365-Troubleshooting\License-$($UserPrincipalName.Replace('@','_'))"
New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Connect-MgGraph -Scopes "User.ReadWrite.All","Group.Read.All","Directory.Read.All","Organization.Read.All"

Get-MgSubscribedSku |
    Select-Object `
        SkuPartNumber,
        SkuId,
        ConsumedUnits,
        @{Name="EnabledUnits";Expression={$_.PrepaidUnits.Enabled}} |
    Sort-Object SkuPartNumber |
    Format-Table -AutoSize |
    Tee-Object "$EvidencePath\tenant-skus.txt"

Get-MgUser `
    -UserId $UserPrincipalName `
    -Property Id,DisplayName,UserPrincipalName,AccountEnabled,UsageLocation,AssignedLicenses |
    Format-List Id,DisplayName,UserPrincipalName,AccountEnabled,UsageLocation,AssignedLicenses |
    Tee-Object "$EvidencePath\user-license-summary.txt"

Get-MgUserLicenseDetail -UserId $UserPrincipalName |
    Format-List SkuPartNumber,SkuId,ServicePlans |
    Tee-Object "$EvidencePath\user-license-detail.txt"

Get-MgUserMemberOf -UserId $UserPrincipalName |
    ConvertTo-Json -Depth 10 |
    Out-File "$EvidencePath\user-memberof.json"
```

Common license fixes:

```powershell
# Set missing usage location.
Update-MgUser -UserId "<affected-user-upn>" -UsageLocation "US"

# Add user to licensing group.
$Group = Get-MgGroup -Filter "displayName eq '<license-group-name>'"
$User = Get-MgUser -UserId "<affected-user-upn>"

New-MgGroupMemberByRef `
    -GroupId $Group.Id `
    -BodyParameter @{
        "@odata.id" = "https://graph.microsoft.com/v1.0/directoryObjects/$($User.Id)"
    }

# Assign direct license as approved exception.
$Sku = Get-MgSubscribedSku |
    Where-Object {$_.SkuPartNumber -eq "<sku-part-number>"}

Set-MgUserLicense `
    -UserId "<affected-user-upn>" `
    -AddLicenses @(
        @{
            SkuId = $Sku.SkuId
            DisabledPlans = @()
        }
    ) `
    -RemoveLicenses @()

# Remove direct license if it was assigned incorrectly.
Set-MgUserLicense `
    -UserId "<affected-user-upn>" `
    -AddLicenses @() `
    -RemoveLicenses @($Sku.SkuId)
```

Service plan check:

```powershell
# Find service plans inside a SKU.

$SkuPartNumber = "<sku-part-number>"

$Sku = Get-MgSubscribedSku |
    Where-Object {$_.SkuPartNumber -eq $SkuPartNumber}

$Sku.ServicePlans |
    Sort-Object ServicePlanName |
    Format-Table ServicePlanName,ServicePlanId,ProvisioningStatus,AppliesTo -AutoSize
```

Operational note:

```text
A user can have the right SKU but still miss the needed service if the service plan is disabled.
Check the service plan, not just the license name.
```

# 08_Troubleshoot_M365_Tenant_User_Group_License_Role_And_Service_Health_Issues_Role_And_Delegation_Skeleton
Use this when an admin cannot open a portal, cannot perform a task, sees only part of the tenant, or can manage some objects but not others.

Role troubleshooting checklist:

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm admin user exists | Admin Workstation | `Get-MgUser -UserId "<admin-upn>"` | Admin user object exists |
| 2 | Confirm admin account enabled | Admin Workstation | `Get-MgUser -UserId "<admin-upn>" -Property AccountEnabled` | Admin sign-in is enabled |
| 3 | Export Entra role assignments | Admin Workstation | `Get-MgRoleManagementDirectoryRoleAssignment -All` | Role assignments are visible |
| 4 | Resolve target role definition | Admin Workstation | `Get-MgRoleManagementDirectoryRoleDefinition -All` | Role ID is known |
| 5 | Check direct role assignment | Admin Workstation | Role assignment export | Admin has or lacks expected role |
| 6 | Check role-assignable group membership | Admin Workstation | `Get-MgGroupMember -GroupId "<group-id>"` | Admin inherits role through group or not |
| 7 | Check admin unit scope | Admin Workstation | `Get-MgDirectoryAdministrativeUnit` | Scoped delegation boundary is known |
| 8 | Check scoped role assignment | Admin Workstation | Role assignment DirectoryScopeId | Assignment is tenant-wide or admin-unit scoped |
| 9 | Check PIM activation if applicable | Admin Workstation | Entra admin center > PIM | Admin may need to activate role |
| 10 | Check Exchange RBAC if Exchange task fails | Admin Workstation | `Get-RoleGroupMember` and `Get-ManagementRoleAssignment` | Exchange role group access is known |

Role investigation commands:

```powershell
# Run from an admin workstation.
# Purpose: collect role and delegation troubleshooting evidence.

$AdminUpn = "<admin-upn>"
$EvidencePath = ".\Evidence\M365-Troubleshooting\Roles-$($AdminUpn.Replace('@','_'))"
New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Connect-MgGraph -Scopes "RoleManagement.Read.Directory","Directory.Read.All","User.Read.All","Group.Read.All"

$Admin = Get-MgUser -UserId $AdminUpn

Get-MgUser `
    -UserId $AdminUpn `
    -Property Id,DisplayName,UserPrincipalName,AccountEnabled,UserType |
    Format-List Id,DisplayName,UserPrincipalName,AccountEnabled,UserType |
    Tee-Object "$EvidencePath\admin-user.txt"

Get-MgRoleManagementDirectoryRoleDefinition -All |
    Select-Object Id,DisplayName,Description,IsEnabled |
    Sort-Object DisplayName |
    Export-Csv "$EvidencePath\role-definitions.csv" -NoTypeInformation

Get-MgRoleManagementDirectoryRoleAssignment -All |
    Select-Object Id,PrincipalId,RoleDefinitionId,DirectoryScopeId,AppScopeId |
    Export-Csv "$EvidencePath\role-assignments.csv" -NoTypeInformation

Get-MgUserMemberOf -UserId $AdminUpn |
    ConvertTo-Json -Depth 10 |
    Out-File "$EvidencePath\admin-memberof.json"

Get-MgDirectoryAdministrativeUnit -All |
    Select-Object Id,DisplayName,Description |
    Export-Csv "$EvidencePath\admin-units.csv" -NoTypeInformation
```

Exchange RBAC investigation:

```powershell
# Run from an admin workstation.
# Purpose: collect Exchange role group and RBAC evidence.

$AdminUpn = "<admin-upn>"
$EvidencePath = ".\Evidence\M365-Troubleshooting\Exchange-RBAC-$($AdminUpn.Replace('@','_'))"
New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Connect-ExchangeOnline -UserPrincipalName "<admin-upn>"

Get-RoleGroup |
    Sort-Object Name |
    Format-Table Name,Roles -AutoSize |
    Tee-Object "$EvidencePath\exo-role-groups.txt"

Get-RoleGroup |
    ForEach-Object {
        $RoleGroup = $_
        Get-RoleGroupMember -Identity $RoleGroup.Name |
            Select-Object @{Name="RoleGroup";Expression={$RoleGroup.Name}},Name,PrimarySmtpAddress,RecipientType
    } |
    Export-Csv "$EvidencePath\exo-role-group-members.csv" -NoTypeInformation

Get-ManagementRoleAssignment |
    Select-Object Name,Role,RoleAssigneeName,RecipientWriteScope,CustomRecipientWriteScope,ConfigWriteScope |
    Export-Csv "$EvidencePath\exo-management-role-assignments.csv" -NoTypeInformation
```

Common role fixes:

```powershell
# Assign tenant-wide Entra role only if approved.

$AdminUpn = "<admin-upn>"
$RoleName = "<role-name>"

$Admin = Get-MgUser -UserId $AdminUpn
$RoleDefinition = Get-MgRoleManagementDirectoryRoleDefinition -All |
    Where-Object {$_.DisplayName -eq $RoleName}

$Assignment = @{
    "@odata.type" = "#microsoft.graph.unifiedRoleAssignment"
    RoleDefinitionId = $RoleDefinition.Id
    PrincipalId = $Admin.Id
    DirectoryScopeId = "/"
}

New-MgRoleManagementDirectoryRoleAssignment -BodyParameter $Assignment
```

```powershell
# Add Exchange role group member only if approved.

Add-RoleGroupMember `
    -Identity "<exchange-role-group-name>" `
    -Member "<admin-upn>"
```

Operational note:

```text
If an admin can manage some users but not others, check administrative unit scope.
If an admin can manage Microsoft 365 users but not Exchange recipients, check Exchange RBAC.
```

# 08_Troubleshoot_M365_Tenant_User_Group_License_Role_And_Service_Health_Issues_Graph_Export_Skeleton
Use this to capture broad evidence before escalation or major changes.

```powershell
# Run from an admin workstation.
# Purpose: export tenant troubleshooting evidence across users, groups, licenses, roles, and service health.

$EvidencePath = ".\Evidence\M365-Troubleshooting\Full-Tenant-Export"
New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Connect-MgGraph -Scopes `
    "User.Read.All",
    "Group.Read.All",
    "Directory.Read.All",
    "Organization.Read.All",
    "RoleManagement.Read.Directory",
    "ServiceHealth.Read.All",
    "ServiceMessage.Read.All",
    "Reports.Read.All"

Get-MgContext |
    Tee-Object "$EvidencePath\graph-context.txt"

Get-MgOrganization |
    ConvertTo-Json -Depth 10 |
    Out-File "$EvidencePath\organization.json"

Get-MgDomain |
    ConvertTo-Json -Depth 10 |
    Out-File "$EvidencePath\domains.json"

Get-MgSubscribedSku |
    ConvertTo-Json -Depth 20 |
    Out-File "$EvidencePath\subscribed-skus.json"

Get-MgUser -All -Property Id,DisplayName,UserPrincipalName,Mail,UserType,AccountEnabled,UsageLocation,AssignedLicenses,CreatedDateTime |
    Select-Object Id,DisplayName,UserPrincipalName,Mail,UserType,AccountEnabled,UsageLocation,AssignedLicenses,CreatedDateTime |
    Export-Csv "$EvidencePath\users.csv" -NoTypeInformation

Get-MgGroup -All -Property Id,DisplayName,Mail,MailNickname,MailEnabled,SecurityEnabled,GroupTypes,AssignedLicenses,Visibility |
    Select-Object Id,DisplayName,Mail,MailNickname,MailEnabled,SecurityEnabled,GroupTypes,AssignedLicenses,Visibility |
    Export-Csv "$EvidencePath\groups.csv" -NoTypeInformation

Get-MgRoleManagementDirectoryRoleDefinition -All |
    Select-Object Id,DisplayName,Description,IsEnabled |
    Export-Csv "$EvidencePath\role-definitions.csv" -NoTypeInformation

Get-MgRoleManagementDirectoryRoleAssignment -All |
    Select-Object Id,PrincipalId,RoleDefinitionId,DirectoryScopeId,AppScopeId |
    Export-Csv "$EvidencePath\role-assignments.csv" -NoTypeInformation

Get-MgDirectoryAdministrativeUnit -All |
    Select-Object Id,DisplayName,Description |
    Export-Csv "$EvidencePath\admin-units.csv" -NoTypeInformation

Get-MgAdminServiceAnnouncementHealthOverview |
    ConvertTo-Json -Depth 10 |
    Out-File "$EvidencePath\service-health-overview.json"

Get-MgAdminServiceAnnouncementIssue -All |
    ConvertTo-Json -Depth 10 |
    Out-File "$EvidencePath\service-health-issues.json"

Get-MgAdminServiceAnnouncementMessage -All |
    ConvertTo-Json -Depth 10 |
    Out-File "$EvidencePath\message-center-messages.json"
```

Exchange full export:

```powershell
# Run from an admin workstation.
# Purpose: export Exchange Online evidence for mail, groups, shared mailboxes, and Exchange RBAC.

$EvidencePath = ".\Evidence\M365-Troubleshooting\Full-Tenant-Export"
New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Connect-ExchangeOnline -UserPrincipalName "<admin-upn>"

Get-AcceptedDomain |
    Export-Csv "$EvidencePath\exo-accepted-domains.csv" -NoTypeInformation

Get-EXOMailbox -ResultSize Unlimited |
    Select-Object DisplayName,PrimarySmtpAddress,RecipientTypeDetails |
    Export-Csv "$EvidencePath\exo-mailboxes.csv" -NoTypeInformation

Get-DistributionGroup |
    Select-Object DisplayName,PrimarySmtpAddress,RecipientTypeDetails,RequireSenderAuthenticationEnabled,HiddenFromAddressListsEnabled |
    Export-Csv "$EvidencePath\exo-distribution-groups.csv" -NoTypeInformation

Get-EXOMailbox -RecipientTypeDetails SharedMailbox -ResultSize Unlimited |
    Select-Object DisplayName,PrimarySmtpAddress,RecipientTypeDetails,HiddenFromAddressListsEnabled |
    Export-Csv "$EvidencePath\exo-shared-mailboxes.csv" -NoTypeInformation

Get-RoleGroup |
    Export-Csv "$EvidencePath\exo-role-groups.csv" -NoTypeInformation

Get-ManagementRoleAssignment |
    Export-Csv "$EvidencePath\exo-management-role-assignments.csv" -NoTypeInformation
```

Operational note:

```text
Full exports are evidence, not the fix.
Use them before escalation, broad rollback, or when the root cause is unclear.
```

# 08_Troubleshoot_M365_Tenant_User_Group_License_Role_And_Service_Health_Issues_Escalation_Skeleton
Escalate when the issue is Microsoft-side, tenant-wide, high impact, or not resolved after local evidence collection.

Escalation checklist:

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm service health status | Admin Workstation | Service health export | Microsoft incident state is known |
| 2 | Confirm affected scope | Operator | Ticket count / evidence | User count and workload impact are known |
| 3 | Capture exact error messages | Affected Client | Screenshot / text copy | Error is preserved |
| 4 | Capture timestamps | Operator | Notes | Start time and test times are documented |
| 5 | Capture tenant ID | Admin Workstation | `Get-MgContext` | Tenant ID is available |
| 6 | Capture affected users/groups | Operator | Evidence table | Affected objects are listed |
| 7 | Capture recent changes | Operator | Change log / Message center | Change correlation is known |
| 8 | Run Graph export | Admin Workstation | Export skeleton | Tenant evidence is saved |
| 9 | Run Exchange export if mail issue | Admin Workstation | Exchange export skeleton | Mail evidence is saved |
| 10 | Open Microsoft support case if needed | Admin Workstation | Admin center > Help and support | Support case exists |

Support case template:

| Field | Value |
|---|---|
| Tenant ID | `<tenant-id>` |
| Affected workload | `<workload>` |
| Impact summary | `<impact-summary>` |
| Number of affected users | `<count>` |
| Example affected user | `<user-upn>` |
| Start time | `<start-time>` |
| Last known good | `<last-known-good>` |
| Error message | `<exact-error>` |
| Service health incident | `<incident-id-or-none>` |
| Recent change | `<change-summary>` |
| Evidence path | `<evidence-path>` |
| Tests performed | `<tests>` |
| Business impact | `<business-impact>` |
| Requested help | `<request>` |

Operational note:

```text
A good escalation includes tenant ID, timestamps, exact errors, affected objects, what changed, what was tested, and what evidence was captured.
Bad escalation is just "Microsoft 365 is broken."
```

# 08_Troubleshoot_M365_Tenant_User_Group_License_Role_And_Service_Health_Issues_Verification_Commands
```powershell
# Baseline Microsoft Graph connection.

Connect-MgGraph -Scopes `
    "User.ReadWrite.All",
    "Group.ReadWrite.All",
    "Directory.ReadWrite.All",
    "Organization.Read.All",
    "RoleManagement.Read.Directory",
    "ServiceHealth.Read.All",
    "ServiceMessage.Read.All",
    "Reports.Read.All"

Get-MgContext
```

```powershell
# Service health and Message center.

Get-MgAdminServiceAnnouncementHealthOverview |
    Format-Table Service,Status,Id -AutoSize

Get-MgAdminServiceAnnouncementIssue -All |
    Select-Object Id,Title,Service,Status,Classification,StartDateTime,LastModifiedDateTime |
    Format-Table -AutoSize

Get-MgAdminServiceAnnouncementMessage -All |
    Select-Object Id,Title,Category,Severity,StartDateTime,EndDateTime,LastModifiedDateTime |
    Format-Table -AutoSize
```

```powershell
# User checks.

$UserPrincipalName = "<affected-user-upn>"

Get-MgUser `
    -UserId $UserPrincipalName `
    -Property Id,DisplayName,UserPrincipalName,Mail,UserType,AccountEnabled,UsageLocation,AssignedLicenses |
    Format-List Id,DisplayName,UserPrincipalName,Mail,UserType,AccountEnabled,UsageLocation,AssignedLicenses

Get-MgUserLicenseDetail -UserId $UserPrincipalName |
    Format-List SkuPartNumber,SkuId,ServicePlans

Get-MgUserMemberOf -UserId $UserPrincipalName |
    ConvertTo-Json -Depth 10
```

```powershell
# Group checks.

$GroupDisplayName = "<affected-group-name>"

$Group = Get-MgGroup -Filter "displayName eq '$GroupDisplayName'"

Get-MgGroup -GroupId $Group.Id |
    Format-List Id,DisplayName,Mail,MailEnabled,SecurityEnabled,GroupTypes,Visibility

Get-MgGroupOwner -GroupId $Group.Id |
    ConvertTo-Json -Depth 10

Get-MgGroupMember -GroupId $Group.Id |
    ConvertTo-Json -Depth 10

Get-MgGroupLicenseDetail -GroupId $Group.Id -ErrorAction SilentlyContinue |
    Format-List *
```

```powershell
# License checks.

Get-MgSubscribedSku |
    Select-Object `
        SkuPartNumber,
        SkuId,
        ConsumedUnits,
        @{Name="EnabledUnits";Expression={$_.PrepaidUnits.Enabled}} |
    Sort-Object SkuPartNumber |
    Format-Table -AutoSize
```

```powershell
# Role checks.

Get-MgRoleManagementDirectoryRoleDefinition -All |
    Sort-Object DisplayName |
    Format-Table Id,DisplayName,IsEnabled -AutoSize

Get-MgRoleManagementDirectoryRoleAssignment -All |
    Select-Object Id,PrincipalId,RoleDefinitionId,DirectoryScopeId |
    Format-Table -AutoSize

Get-MgDirectoryAdministrativeUnit -All |
    Format-Table Id,DisplayName,Description -AutoSize
```

```powershell
# Exchange Online checks.

Connect-ExchangeOnline -UserPrincipalName "<admin-upn>"

Get-AcceptedDomain |
    Format-Table Name,DomainName,DomainType,Default -AutoSize

Get-DistributionGroup -Identity "<distribution-group-name>" -ErrorAction SilentlyContinue |
    Format-List DisplayName,PrimarySmtpAddress,RecipientTypeDetails,RequireSenderAuthenticationEnabled,HiddenFromAddressListsEnabled

Get-DistributionGroupMember -Identity "<distribution-group-name>" -ErrorAction SilentlyContinue |
    Format-Table Name,PrimarySmtpAddress,RecipientType -AutoSize

Get-EXOMailbox -Identity "<shared-mailbox-email>" -ErrorAction SilentlyContinue |
    Format-List DisplayName,PrimarySmtpAddress,RecipientTypeDetails,HiddenFromAddressListsEnabled

Get-MailboxPermission -Identity "<shared-mailbox-email>" -ErrorAction SilentlyContinue |
    Where-Object {
        $_.User -notlike "NT AUTHORITY\SELF" -and
        $_.IsInherited -eq $false
    } |
    Format-Table User,AccessRights,Deny,IsInherited -AutoSize

Get-RecipientPermission -Identity "<shared-mailbox-email>" -ErrorAction SilentlyContinue |
    Format-Table Trustee,AccessRights,IsInherited -AutoSize
```

```powershell
# DNS checks.

$Domain = "<custom-domain>"
$DnsServer = "8.8.8.8"

nslookup -type=NS $Domain $DnsServer
nslookup -type=TXT $Domain $DnsServer
nslookup -type=MX $Domain $DnsServer
nslookup "autodiscover.$Domain" $DnsServer
```

Portal verification:

| Validation Area | Portal Path | Good Result |
|---|---|---|
| Service health | Health > Service health | No matching incident, or incident explains issue |
| Message center | Health > Message center | No relevant planned change, or change explains issue |
| User state | Users > Active users > affected user | User exists and account state is correct |
| Deleted users | Users > Deleted users | No conflicting deleted user, or deleted user restored if needed |
| Licenses and apps | User > Licenses and apps | Required SKU and service plans are enabled |
| Groups | Teams and groups | Group exists with expected type, owners, and members |
| Shared mailbox | Teams and groups > Shared mailboxes | Mailbox and permissions are correct |
| Roles | Roles > Role assignments | Admin has expected role |
| Admin units | Entra admin center > Admin units | Scoped delegation is correct |
| Exchange RBAC | Exchange admin center > Roles | Exchange role group access is correct |
| DNS | Settings > Domains and external DNS checks | Domain is verified and records are correct |

# 08_Troubleshoot_M365_Tenant_User_Group_License_Role_And_Service_Health_Issues_Rollback
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify change made during troubleshooting | Operator | Notes / command history | Changed object and value are known |
| 2 | Revert user enabled state if changed incorrectly | Admin Workstation | `Update-MgUser -AccountEnabled` | Account enabled state returns to prior value |
| 3 | Restore previous usage location if changed incorrectly | Admin Workstation | `Update-MgUser -UsageLocation "<previous-location>"` | Usage location returns to prior value |
| 4 | Remove direct license assigned by mistake | Admin Workstation | `Set-MgUserLicense -RemoveLicenses` | Incorrect direct license is removed |
| 5 | Reassign direct license removed by mistake | Admin Workstation | `Set-MgUserLicense -AddLicenses` | License is restored |
| 6 | Remove user from group added by mistake | Admin Workstation | `Remove-MgGroupMemberByRef` | Incorrect group membership is removed |
| 7 | Re-add user to group removed by mistake | Admin Workstation | `New-MgGroupMemberByRef` | Group membership is restored |
| 8 | Remove distribution group member added by mistake | Admin Workstation | `Remove-DistributionGroupMember` | Incorrect distribution membership is removed |
| 9 | Re-add distribution group member removed by mistake | Admin Workstation | `Add-DistributionGroupMember` | Distribution membership is restored |
| 10 | Remove shared mailbox Full Access added by mistake | Admin Workstation | `Remove-MailboxPermission` | Delegate access is removed |
| 11 | Remove shared mailbox Send As added by mistake | Admin Workstation | `Remove-RecipientPermission` | Send As access is removed |
| 12 | Restore shared mailbox permission removed by mistake | Admin Workstation | `Add-MailboxPermission` or `Add-RecipientPermission` | Delegate permission is restored |
| 13 | Remove role assignment added by mistake | Admin Workstation | `Remove-MgRoleManagementDirectoryRoleAssignment` | Incorrect role is removed |
| 14 | Recreate role assignment removed by mistake | Admin Workstation | `New-MgRoleManagementDirectoryRoleAssignment` | Role assignment is restored |
| 15 | Restore DNS record changed by mistake | DNS Provider Portal | Recreate previous DNS value | Public DNS returns to prior state |
| 16 | Validate rollback | Admin Workstation | Verification commands | Object state matches rollback plan |
| 17 | Document rollback | Operator | Notes | Rollback owner, time, and validation are recorded |

Rollback command examples:

```powershell
# Revert user sign-in state.
Update-MgUser -UserId "<affected-user-upn>" -AccountEnabled:$true
```

```powershell
# Remove user from Graph group.
$Group = Get-MgGroup -Filter "displayName eq '<group-name>'"
$User = Get-MgUser -UserId "<user-upn>"

Remove-MgGroupMemberByRef `
    -GroupId $Group.Id `
    -DirectoryObjectId $User.Id
```

```powershell
# Remove direct license.
$Sku = Get-MgSubscribedSku |
    Where-Object {$_.SkuPartNumber -eq "<sku-part-number>"}

Set-MgUserLicense `
    -UserId "<user-upn>" `
    -AddLicenses @() `
    -RemoveLicenses @($Sku.SkuId)
```

```powershell
# Remove shared mailbox Full Access.
Remove-MailboxPermission `
    -Identity "<shared-mailbox-email>" `
    -User "<user-upn>" `
    -AccessRights FullAccess `
    -InheritanceType All `
    -Confirm:$false
```

```powershell
# Remove Send As.
Remove-RecipientPermission `
    -Identity "<shared-mailbox-email>" `
    -Trustee "<user-upn>" `
    -AccessRights SendAs `
    -Confirm:$false
```

Rollback note template:

```text
Issue:
Change made:
Object changed:
Previous value:
New value:
Rollback action:
Rollback owner:
Rollback time:
Validation performed:
Remaining issue:
```

# 08_Troubleshoot_M365_Tenant_User_Group_License_Role_And_Service_Health_Issues_Failure_Checks
| Symptom | Command | What Usually Broke |
|---|---|---|
| Many users affected at once | Service health and Message center checks | Microsoft incident, tenant-wide change, license exhaustion, Conditional Access, DNS, or proxy issue |
| One user cannot sign in | `Get-MgUser -UserId "<user-upn>"` | Account disabled, wrong UPN, password/auth issue, Conditional Access, or deleted/conflicting user |
| User cannot access Exchange | `Get-MgUserLicenseDetail`; `Get-EXOMailbox` | Exchange service plan disabled, license missing, mailbox not provisioned, or service incident |
| User cannot access Teams | `Get-MgUserLicenseDetail` | Teams service plan disabled, license missing, policy issue, or service incident |
| User cannot access OneDrive | `Get-MgUserLicenseDetail` | SharePoint/OneDrive plan disabled, provisioning delay, or service incident |
| User license assignment fails | `Get-MgUser -Property UsageLocation`; `Get-MgSubscribedSku` | Missing usage location, no available licenses, or conflicting SKU |
| Group-based license not applying | `Get-MgGroupLicenseDetail`; group members | Processing delay, missing usage location, nested group, or group license error |
| User has license but wrong service disabled | `Get-MgUserLicenseDetail` | Service plan disabled inside assigned SKU |
| User removed from group but still has license | `Get-MgUserLicenseDetail` | User has direct license or another licensing group |
| Direct license removal fails | `Set-MgUserLicense` error | License is inherited from group or wrong SKU ID used |
| Group membership looks right but access fails | `Get-MgUserMemberOf`; workload access test | Propagation delay, token/session cache, or workload-specific permission issue |
| Distribution group does not receive external mail | `Get-DistributionGroup | FL RequireSenderAuthenticationEnabled` | External senders are blocked |
| Distribution group member does not receive mail | `Get-DistributionGroupMember` | User missing from group, moderation, mail flow, or recipient issue |
| Shared mailbox does not appear in Outlook | `Get-MailboxPermission` | Full Access missing, automapping delay, Outlook cache, or wrong account |
| User can open shared mailbox but cannot send | `Get-RecipientPermission` | Send As missing or propagation delay |
| Send on behalf happens instead of Send As | `Get-Mailbox | FL GrantSendOnBehalfTo`; `Get-RecipientPermission` | Wrong permission model configured |
| Admin cannot open admin center page | Role assignment export | Missing admin role, PIM not activated, or scoped role limitation |
| Admin can manage some users but not others | Admin unit role assignment check | Administrative unit scope is limiting access |
| Admin can manage users but not Exchange recipients | Exchange RBAC export | Exchange role group missing or scoped too narrowly |
| Role assignment was added but still no access | Token/session refresh | User needs sign out/in, role activation, or propagation time |
| Graph command denied | `Get-MgContext` | Missing Graph scope or admin consent |
| Exchange command denied | `Connect-ExchangeOnline`; `Get-RoleGroupMember` | Missing Exchange role or RBAC membership |
| Custom domain user cannot receive mail | `Get-AcceptedDomain`; `nslookup -type=MX` | Accepted domain missing, MX wrong, mailbox missing, or recipient address missing |
| Outlook Autodiscover fails | `nslookup autodiscover.<domain>` | Autodiscover record missing, split DNS issue, or old provider DNS |
| SPF/authentication issue | `nslookup -type=TXT <domain>` | SPF missing Microsoft 365, duplicate SPF records, or old mail sender retained |
| Guest cannot access resource | `Get-MgUser -Filter "userType eq 'Guest'"`; group membership check | Guest exists but is not assigned group/app/site access |
| Deleted user cannot be recreated | `Get-MgDirectoryDeletedItemAsUser -All` | Soft-deleted object conflict |
| Wrong tenant modified | `Get-MgContext` | Admin connected to wrong tenant |
| Service health says healthy but issue persists | Full export and workload checks | Tenant configuration, identity, license, group, role, DNS, or client/network issue |
| Support case lacks useful detail | Evidence folder review | Missing tenant ID, timestamps, error message, affected objects, or test results |

# 08_Troubleshoot_M365_Tenant_User_Group_License_Role_And_Service_Health_Issues_Related_Labs
| Lab | Relationship |
|---|---|
| 01_Create_And_Configure_Microsoft_365_Tenant_Org_Profile_And_Service_Settings | Provides tenant baseline, admin center navigation, and org settings context |
| 02_Configure_M365_Custom_Domains_DNS_And_Network_Connectivity_Insights | Provides domain, DNS, MX, Autodiscover, and network connectivity baseline |
| 03_Monitor_Service_Health_Message_Center_Usage_Reports_And_M365_Backup | Provides Service health, Message center, usage reports, and backup/recovery visibility |
| 04_Create_Manage_And_Bulk_Administer_M365_Users_Contacts_And_Guests | Provides user, contact, guest, deleted user, and lifecycle administration baseline |
| 05_Manage_M365_Groups_Security_Groups_Distribution_Groups_Shared_Mailboxes_And_M365_Groups | Provides group, distribution group, mail-enabled security group, and shared mailbox baseline |
| 06_Manage_M365_Licensing_Service_Plans_Group_Based_Licensing_And_Usage | Provides direct licensing, group-based licensing, service plan, and usage cleanup baseline |
| 07_Manage_Admin_Roles_Role_Groups_Admin_Units_And_Delegation | Provides admin role, role group, admin unit, PIM, and delegation baseline |
| 08_Troubleshoot_Cloud_Admin_Access_Portal_Login_Domain_And_Tooling_Issues | Cloud admin troubleshooting pattern for portal, login, domain, and tooling failures |
| 05_Troubleshoot_Identity_Sync_Attribute_Duplicate_Object_And_Connect_Health_Issues | Hybrid identity troubleshooting for synced users and duplicate attributes |
| 06_Configure_SSPR_MFA_Authentication_Methods_And_Password_Protection | Authentication and password reset troubleshooting dependency |
| 07_Configure_Conditional_Access_Baseline_Policies_And_Break_Glass_Exclusions | Conditional Access and break glass troubleshooting dependency |
| 08_Map_On_Prem_Groups_To_Cloud_Roles_Licensing_Access_And_App_Assignments | Hybrid group mapping dependency for roles, licenses, apps, and access |