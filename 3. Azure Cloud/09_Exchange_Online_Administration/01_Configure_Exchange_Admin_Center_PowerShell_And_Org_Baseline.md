Built from current Microsoft guidance for Exchange admin center, Exchange Online PowerShell, Exchange Online RBAC, and baseline organization cmdlets.  

# Configure_Exchange_Admin_Center_PowerShell_And_Org_Baseline

  

# Configure_Exchange_Admin_Center_PowerShell_And_Org_Baseline_Index

01_Configure_Exchange_Admin_Center_PowerShell_And_Org_Baseline.md

Configure_Exchange_Admin_Center_PowerShell_And_Org_Baseline

Configure_Exchange_Admin_Center_PowerShell_And_Org_Baseline_Source_Basis

Configure_Exchange_Admin_Center_PowerShell_And_Org_Baseline_Mental_Model

Configure_Exchange_Admin_Center_PowerShell_And_Org_Baseline_Planning_Table

Configure_Exchange_Admin_Center_PowerShell_And_Org_Baseline_Configuration_Checklist

Configure_Exchange_Admin_Center_PowerShell_And_Org_Baseline_Precheck_Skeleton

Configure_Exchange_Admin_Center_PowerShell_And_Org_Baseline_Module_Install_Skeleton

Configure_Exchange_Admin_Center_PowerShell_And_Org_Baseline_Connection_Skeleton

Configure_Exchange_Admin_Center_PowerShell_And_Org_Baseline_Org_Inventory_Skeleton

Configure_Exchange_Admin_Center_PowerShell_And_Org_Baseline_RBAC_Baseline_Skeleton

Configure_Exchange_Admin_Center_PowerShell_And_Org_Baseline_Recipient_Baseline_Skeleton

Configure_Exchange_Admin_Center_PowerShell_And_Org_Baseline_Validation_Skeleton

Configure_Exchange_Admin_Center_PowerShell_And_Org_Baseline_Verification_Commands

Configure_Exchange_Admin_Center_PowerShell_And_Org_Baseline_Rollback

Configure_Exchange_Admin_Center_PowerShell_And_Org_Baseline_Failure_Checks

Configure_Exchange_Admin_Center_PowerShell_And_Org_Baseline_Related_Labs

  

# Configure_Exchange_Admin_Center_PowerShell_And_Org_Baseline_Source_Basis

  

| Source | Relevant Section | What It Supports |

|---|---|---|

| Microsoft Learn | Exchange admin center in Exchange Online | Locating Exchange Online admin portal areas for recipients, mail flow, roles, reports, and settings |

| Microsoft Learn | Connect to Exchange Online PowerShell | Installing, importing, connecting, validating, and disconnecting Exchange Online PowerShell |

| Microsoft Learn | Permissions in Exchange Online | Understanding RBAC, role groups, management roles, and cmdlet access boundaries |

| Microsoft Learn | Get-OrganizationConfig | Capturing tenant-level Exchange organization configuration |

| Microsoft Learn | Get-TransportConfig | Capturing global transport configuration |

| Microsoft Learn | Get-AcceptedDomain | Capturing accepted domain inventory |

| Microsoft Learn | ExchangeOnlineManagement module | Using modern Exchange Online PowerShell cmdlets, including REST-backed EXO cmdlets |

| Microsoft 365 operational practice | Admin evidence capture, baseline export, and least privilege | Producing a reusable Exchange Online admin baseline before mailbox, group, mail flow, security, and retention changes |

  

# Configure_Exchange_Admin_Center_PowerShell_And_Org_Baseline_Mental_Model

  

| Concept | Operational Meaning |

|---|---|

| Exchange admin center | Web console for Exchange Online administration |

| Exchange Online PowerShell | Administrative shell used to inspect and configure Exchange Online at scale |

| ExchangeOnlineManagement module | PowerShell module used to connect to Exchange Online with modern authentication |

| Connect-ExchangeOnline | Cmdlet that establishes an Exchange Online PowerShell session |

| Disconnect-ExchangeOnline | Cmdlet that closes the Exchange Online PowerShell session cleanly |

| RBAC | Exchange role-based access control model that decides which cmdlets and parameters an admin can use |

| Role group | Group that assigns Exchange administrative permissions |

| Management role | Permission bundle that grants access to specific Exchange tasks |

| Organization config | Tenant-level Exchange Online settings returned by `Get-OrganizationConfig` |

| Transport config | Global mail transport settings returned by `Get-TransportConfig` |

| Accepted domain | Domain Exchange Online accepts mail for |

| Recipient baseline | Initial inventory of mailbox, group, contact, and resource recipient types |

| Evidence path | Local folder where command output is saved for later comparison |

| First rule | Do not change Exchange Online until admin access, PowerShell connectivity, RBAC scope, accepted domains, and baseline exports are known |

  

# Configure_Exchange_Admin_Center_PowerShell_And_Org_Baseline_Planning_Table

  

| Item | Example | Decision |

|---|---|---|

| Microsoft 365 tenant | `contoso.onmicrosoft.com` | `<tenant-name>` |

| Primary SMTP domain | `contoso.com` | `<primary-smtp-domain>` |

| Admin UPN | `admin@contoso.com` | `<admin-upn>` |

| Admin role model | Exchange Administrator / Global Administrator / delegated role | `<role-model>` |

| Break-glass account excluded from routine work | Yes | `<yes-no>` |

| Admin workstation | Windows 11 admin VM / PAW / management laptop | `<admin-workstation>` |

| PowerShell version | PowerShell 7.x or Windows PowerShell 5.1 | `<ps-version>` |

| ExchangeOnlineManagement module version | Latest available approved version | `<module-version>` |

| MFA required | Yes | `<yes-no>` |

| Conditional Access impact | Admin workstation allowed | `<allowed-blocked>` |

| Browser portal | `https://admin.exchange.microsoft.com` | `<portal-url>` |

| Evidence root | `C:\M365-Exchange-Baseline` | `<evidence-root>` |

| Baseline date | `2026-06-15` | `<baseline-date>` |

| Change ticket | `LAB-EXO-001` | `<ticket-or-lab-id>` |

| Next workbook | `02_Manage_User_Mailboxes_Shared_Mailboxes_Contacts_And_Resources.md` | `<next-task>` |

| Rollback stance | No destructive change in this workbook | Baseline only |

  

# Configure_Exchange_Admin_Center_PowerShell_And_Org_Baseline_Configuration_Checklist

  

| Step | Task | Device | PowerShell / Command | Expected Result |

|---:|---|---|---|---|

| 1 | Confirm admin workstation identity | Admin Workstation | `hostname` | Workstation name is known |

| 2 | Confirm PowerShell version | Admin Workstation | `$PSVersionTable` | Supported PowerShell version is visible |

| 3 | Confirm internet access to Microsoft 365 admin portals | Admin Workstation | Open `https://admin.microsoft.com` and `https://admin.exchange.microsoft.com` | Portals load |

| 4 | Confirm admin account can sign in | Browser | Sign in as `<admin-upn>` | Admin sign-in succeeds |

| 5 | Confirm Exchange admin center loads | Browser | Open Exchange admin center | Exchange admin center is accessible |

| 6 | Confirm admin role assignment in portal | Browser | Microsoft 365 admin center > Roles | Admin has expected Exchange administrative role |

| 7 | Open elevated or standard PowerShell based on policy | Admin Workstation | `powershell` or `pwsh` | PowerShell opens |

| 8 | Confirm PowerShell Gallery trust path | Admin Workstation | `Get-PSRepository` | PSGallery is visible |

| 9 | Install ExchangeOnlineManagement module | Admin Workstation | `Install-Module ExchangeOnlineManagement -Scope CurrentUser` | Module installs |

| 10 | Import module | Admin Workstation | `Import-Module ExchangeOnlineManagement` | Module imports |

| 11 | Confirm module version | Admin Workstation | `Get-Module ExchangeOnlineManagement -ListAvailable` | Installed version is visible |

| 12 | Connect to Exchange Online | Admin Workstation | `Connect-ExchangeOnline -UserPrincipalName "<admin-upn>" -ShowBanner:$false` | Authentication succeeds |

| 13 | Validate Exchange cmdlet access | Admin Workstation | `Get-AcceptedDomain` | Accepted domains return |

| 14 | Capture connection information | Admin Workstation | `Get-ConnectionInformation` | Active Exchange Online connection is visible |

| 15 | Create evidence directory | Admin Workstation | `New-Item -ItemType Directory -Force -Path C:\M365-Exchange-Baseline` | Evidence directory exists |

| 16 | Capture organization config | Admin Workstation | `Get-OrganizationConfig \| Format-List *` | Org config is exported |

| 17 | Capture transport config | Admin Workstation | `Get-TransportConfig \| Format-List *` | Transport config is exported |

| 18 | Capture accepted domains | Admin Workstation | `Get-AcceptedDomain` | Accepted domain inventory is exported |

| 19 | Capture remote domains | Admin Workstation | `Get-RemoteDomain` | Remote domain inventory is exported |

| 20 | Capture admin role groups | Admin Workstation | `Get-RoleGroup` | Exchange role groups are exported |

| 21 | Capture management role assignments | Admin Workstation | `Get-ManagementRoleAssignment` | RBAC assignments are exported |

| 22 | Capture mailbox baseline sample | Admin Workstation | `Get-EXOMailbox -ResultSize 100` | Mailbox sample is exported |

| 23 | Capture recipient type counts | Admin Workstation | `Get-Recipient -ResultSize Unlimited \| Group-Object RecipientTypeDetails` | Recipient counts are exported |

| 24 | Capture organization sharing baseline | Admin Workstation | `Get-OrganizationRelationship`; `Get-SharingPolicy` | Sharing baseline is exported |

| 25 | Capture outbound/inbound connector baseline | Admin Workstation | `Get-InboundConnector`; `Get-OutboundConnector` | Connector baseline is exported |

| 26 | Capture transport rule baseline | Admin Workstation | `Get-TransportRule` | Mail flow rule baseline is exported |

| 27 | Disconnect session | Admin Workstation | `Disconnect-ExchangeOnline -Confirm:$false` | Session closes |

| 28 | Document baseline | Operator | Record tenant, admin UPN, module version, evidence path, date, and next workbook | Workbook evidence is complete |

  

# Configure_Exchange_Admin_Center_PowerShell_And_Org_Baseline_Precheck_Skeleton

  

```powershell

# Run on the admin workstation.

# Purpose: confirm workstation, PowerShell, network, and baseline folder readiness.

  

$TenantName = "contoso.onmicrosoft.com"

$AdminUpn = "admin@contoso.com"

$EvidenceRoot = "C:\M365-Exchange-Baseline"

$RunStamp = Get-Date -Format "yyyyMMdd-HHmmss"

$EvidencePath = Join-Path $EvidenceRoot "01-EXO-Org-Baseline-$RunStamp"

  

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

  

# Capture workstation and PowerShell context.

hostname | Tee-Object "$EvidencePath\workstation-hostname.txt"

  

$PSVersionTable |

  Out-File "$EvidencePath\powershell-version.txt"

  

whoami /user |

  Out-File "$EvidencePath\whoami-user.txt"

  

whoami /groups |

  Out-File "$EvidencePath\whoami-groups.txt"

  

# Confirm internet name resolution and HTTPS reachability.

Resolve-DnsName outlook.office365.com |

  Tee-Object "$EvidencePath\dns-outlook-office365.txt"

  

Resolve-DnsName admin.exchange.microsoft.com |

  Tee-Object "$EvidencePath\dns-exchange-admin-center.txt"

  

Test-NetConnection outlook.office365.com -Port 443 |

  Tee-Object "$EvidencePath\nettest-outlook-443.txt"

  

Test-NetConnection admin.exchange.microsoft.com -Port 443 |

  Tee-Object "$EvidencePath\nettest-eac-443.txt"

  

# Capture PowerShell repository state.

Get-PSRepository |

  Tee-Object "$EvidencePath\psrepositories.txt"

  

Write-Output "Precheck complete. Evidence path: $EvidencePath"

Configure_Exchange_Admin_Center_PowerShell_And_Org_Baseline_Module_Install_Skeleton

# Run on the admin workstation.

# Purpose: install or update ExchangeOnlineManagement module.

  

$EvidenceRoot = "C:\M365-Exchange-Baseline"

$RunStamp = Get-Date -Format "yyyyMMdd-HHmmss"

$EvidencePath = Join-Path $EvidenceRoot "01-EXO-Module-$RunStamp"

  

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

  

Start-Transcript -Path "$EvidencePath\module-install-transcript.txt"

  

# Optional: set PSGallery trust only if your lab policy allows it.

# Set-PSRepository -Name PSGallery -InstallationPolicy Trusted

  

# Confirm current module state.

Get-Module ExchangeOnlineManagement -ListAvailable |

  Sort-Object Version -Descending |

  Tee-Object "$EvidencePath\existing-exchangeonline-module.txt"

  

# Install for current user if missing.

if (-not (Get-Module ExchangeOnlineManagement -ListAvailable)) {

  Install-Module ExchangeOnlineManagement -Scope CurrentUser -Force

}

else {

  Update-Module ExchangeOnlineManagement -Force -ErrorAction SilentlyContinue

}

  

# Import module.

Import-Module ExchangeOnlineManagement

  

# Capture final module state.

Get-Module ExchangeOnlineManagement -ListAvailable |

  Sort-Object Version -Descending |

  Select-Object Name,Version,Path |

  Tee-Object "$EvidencePath\final-exchangeonline-module.txt"

  

Get-Command Connect-ExchangeOnline |

  Tee-Object "$EvidencePath\connect-exchangeonline-command.txt"

  

Get-Command Disconnect-ExchangeOnline |

  Tee-Object "$EvidencePath\disconnect-exchangeonline-command.txt"

  

Stop-Transcript

Configure_Exchange_Admin_Center_PowerShell_And_Org_Baseline_Connection_Skeleton

# Run on the admin workstation.

# Purpose: connect to Exchange Online PowerShell using modern authentication.

  

$AdminUpn = "admin@contoso.com"

$EvidenceRoot = "C:\M365-Exchange-Baseline"

$RunStamp = Get-Date -Format "yyyyMMdd-HHmmss"

$EvidencePath = Join-Path $EvidenceRoot "01-EXO-Connection-$RunStamp"

  

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

  

Start-Transcript -Path "$EvidencePath\exo-connection-transcript.txt"

  

Import-Module ExchangeOnlineManagement

  

# Standard commercial Microsoft 365 tenant connection.

Connect-ExchangeOnline `

  -UserPrincipalName $AdminUpn `

  -ShowBanner:$false

  

# Confirm connection. If this cmdlet is unavailable in your module version, use Get-AcceptedDomain as the functional test.

Get-ConnectionInformation |

  Tee-Object "$EvidencePath\exo-connection-information.txt"

  

# Functional smoke test.

Get-AcceptedDomain |

  Select-Object Name,DomainName,DomainType,Default |

  Tee-Object "$EvidencePath\accepted-domain-smoke-test.txt"

  

Stop-Transcript

Configure_Exchange_Admin_Center_PowerShell_And_Org_Baseline_Org_Inventory_Skeleton

# Run after Connect-ExchangeOnline succeeds.

# Purpose: capture Exchange Online organization, transport, domain, connector, and rule baseline.

  

$EvidenceRoot = "C:\M365-Exchange-Baseline"

$RunStamp = Get-Date -Format "yyyyMMdd-HHmmss"

$EvidencePath = Join-Path $EvidenceRoot "01-EXO-Org-Inventory-$RunStamp"

  

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

  

Start-Transcript -Path "$EvidencePath\exo-org-inventory-transcript.txt"

  

# Organization configuration.

Get-OrganizationConfig |

  Format-List * |

  Out-File "$EvidencePath\organization-config-full.txt"

  

Get-OrganizationConfig |

  Select-Object Name,Guid,IsDehydrated,OAuth2ClientProfileEnabled |

  Export-Csv "$EvidencePath\organization-config-summary.csv" -NoTypeInformation

  

# Transport configuration.

Get-TransportConfig |

  Format-List * |

  Out-File "$EvidencePath\transport-config-full.txt"

  

# Accepted domains.

Get-AcceptedDomain |

  Select-Object Name,DomainName,DomainType,Default,AddressBookEnabled,PendingRemoval |

  Export-Csv "$EvidencePath\accepted-domains.csv" -NoTypeInformation

  

# Remote domains.

Get-RemoteDomain |

  Select-Object Name,DomainName,AllowedOOFType,AutoReplyEnabled,AutoForwardEnabled,DeliveryReportEnabled,TNEFEnabled |

  Export-Csv "$EvidencePath\remote-domains.csv" -NoTypeInformation

  

# Connectors.

Get-InboundConnector |

  Select-Object Name,Enabled,ConnectorType,SenderDomains,SenderIPAddresses,RequireTls,RestrictDomainsToIPAddresses |

  Export-Csv "$EvidencePath\inbound-connectors.csv" -NoTypeInformation

  

Get-OutboundConnector |

  Select-Object Name,Enabled,ConnectorType,RecipientDomains,SmartHosts,TlsSettings,RouteAllMessagesViaOnPremises |

  Export-Csv "$EvidencePath\outbound-connectors.csv" -NoTypeInformation

  

# Mail flow rules.

Get-TransportRule |

  Select-Object Name,State,Mode,Priority,RuleVersion,Comments |

  Export-Csv "$EvidencePath\transport-rules.csv" -NoTypeInformation

  

Get-TransportRule |

  Format-List * |

  Out-File "$EvidencePath\transport-rules-full.txt"

  

Stop-Transcript

Configure_Exchange_Admin_Center_PowerShell_And_Org_Baseline_RBAC_Baseline_Skeleton

# Run after Connect-ExchangeOnline succeeds.

# Purpose: capture Exchange Online RBAC role groups, assignments, and current admin visibility.

  

$AdminUpn = "admin@contoso.com"

$EvidenceRoot = "C:\M365-Exchange-Baseline"

$RunStamp = Get-Date -Format "yyyyMMdd-HHmmss"

$EvidencePath = Join-Path $EvidenceRoot "01-EXO-RBAC-$RunStamp"

  

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

  

Start-Transcript -Path "$EvidencePath\exo-rbac-transcript.txt"

  

# Role groups.

Get-RoleGroup |

  Select-Object Name,ManagedBy,RoleAssignments,Members |

  Export-Csv "$EvidencePath\role-groups.csv" -NoTypeInformation

  

Get-RoleGroup |

  Format-List * |

  Out-File "$EvidencePath\role-groups-full.txt"

  

# Role group members.

$RoleGroupMembers = foreach ($RoleGroup in Get-RoleGroup) {

  Get-RoleGroupMember -Identity $RoleGroup.Name -ErrorAction SilentlyContinue |

    Select-Object @{Name="RoleGroup";Expression={$RoleGroup.Name}},Name,PrimarySmtpAddress,RecipientType

}

  

$RoleGroupMembers |

  Export-Csv "$EvidencePath\role-group-members.csv" -NoTypeInformation

  

# Management roles.

Get-ManagementRole |

  Select-Object Name,RoleType |

  Export-Csv "$EvidencePath\management-roles.csv" -NoTypeInformation

  

# Role assignments.

Get-ManagementRoleAssignment |

  Select-Object Name,Role,RoleAssigneeName,RoleAssigneeType,AssignmentMethod,EffectiveUserName,CustomRecipientWriteScope,CustomConfigWriteScope |

  Export-Csv "$EvidencePath\management-role-assignments.csv" -NoTypeInformation

  

# Current admin recipient object, if mailbox or mail user exists.

Get-Recipient -Identity $AdminUpn -ErrorAction SilentlyContinue |

  Format-List * |

  Out-File "$EvidencePath\current-admin-recipient.txt"

  

Stop-Transcript

Configure_Exchange_Admin_Center_PowerShell_And_Org_Baseline_Recipient_Baseline_Skeleton

# Run after Connect-ExchangeOnline succeeds.

# Purpose: capture high-level recipient baseline before mailbox and group workbooks.

  

$EvidenceRoot = "C:\M365-Exchange-Baseline"

$RunStamp = Get-Date -Format "yyyyMMdd-HHmmss"

$EvidencePath = Join-Path $EvidenceRoot "01-EXO-Recipient-Baseline-$RunStamp"

  

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

  

Start-Transcript -Path "$EvidencePath\exo-recipient-baseline-transcript.txt"

  

# Recipient type counts.

Get-Recipient -ResultSize Unlimited |

  Group-Object RecipientTypeDetails |

  Sort-Object Name |

  Select-Object Name,Count |

  Export-Csv "$EvidencePath\recipient-type-counts.csv" -NoTypeInformation

  

# Mailbox sample.

Get-EXOMailbox -ResultSize 100 |

  Select-Object DisplayName,UserPrincipalName,PrimarySmtpAddress,RecipientTypeDetails,ExchangeGuid,ArchiveStatus |

  Export-Csv "$EvidencePath\mailbox-sample.csv" -NoTypeInformation

  

# Shared mailbox sample.

Get-EXOMailbox -RecipientTypeDetails SharedMailbox -ResultSize 100 |

  Select-Object DisplayName,PrimarySmtpAddress,RecipientTypeDetails,ExchangeGuid |

  Export-Csv "$EvidencePath\shared-mailbox-sample.csv" -NoTypeInformation

  

# Room and equipment mailbox sample.

Get-EXOMailbox -RecipientTypeDetails RoomMailbox,EquipmentMailbox -ResultSize 100 |

  Select-Object DisplayName,PrimarySmtpAddress,RecipientTypeDetails,ExchangeGuid |

  Export-Csv "$EvidencePath\resource-mailbox-sample.csv" -NoTypeInformation

  

# Group sample.

Get-DistributionGroup -ResultSize 100 |

  Select-Object DisplayName,PrimarySmtpAddress,RecipientTypeDetails,ManagedBy,RequireSenderAuthenticationEnabled |

  Export-Csv "$EvidencePath\distribution-group-sample.csv" -NoTypeInformation

  

# Microsoft 365 group sample.

Get-UnifiedGroup -ResultSize 100 |

  Select-Object DisplayName,PrimarySmtpAddress,AccessType,ManagedBy,HiddenFromExchangeClientsEnabled |

  Export-Csv "$EvidencePath\m365-group-sample.csv" -NoTypeInformation

  

# Contacts sample.

Get-MailContact -ResultSize 100 |

  Select-Object DisplayName,ExternalEmailAddress,PrimarySmtpAddress,RecipientTypeDetails |

  Export-Csv "$EvidencePath\mail-contact-sample.csv" -NoTypeInformation

  

Stop-Transcript

Configure_Exchange_Admin_Center_PowerShell_And_Org_Baseline_Validation_Skeleton

# Run after all baseline exports complete.

# Purpose: validate that required baseline evidence files exist and basic Exchange cmdlets return data.

  

$EvidenceRoot = "C:\M365-Exchange-Baseline"

  

# Find newest baseline folders for this workbook.

Get-ChildItem $EvidenceRoot -Directory |

  Sort-Object LastWriteTime -Descending |

  Select-Object LastWriteTime,Name,FullName |

  Format-Table -AutoSize

  

# Functional cmdlet validation.

$Validation = [ordered]@{}

  

$Validation["AcceptedDomains"] = (Get-AcceptedDomain | Measure-Object).Count

$Validation["RemoteDomains"] = (Get-RemoteDomain | Measure-Object).Count

$Validation["RoleGroups"] = (Get-RoleGroup | Measure-Object).Count

$Validation["RecipientsSample"] = (Get-Recipient -ResultSize 100 | Measure-Object).Count

$Validation["TransportRules"] = (Get-TransportRule | Measure-Object).Count

$Validation["InboundConnectors"] = (Get-InboundConnector | Measure-Object).Count

$Validation["OutboundConnectors"] = (Get-OutboundConnector | Measure-Object).Count

  

$Validation.GetEnumerator() |

  Sort-Object Name |

  Format-Table -AutoSize

  

# Expected result:

# AcceptedDomains should be at least 1.

# RoleGroups should return Exchange role groups.

# Recipient sample should return data in an active tenant.

Configure_Exchange_Admin_Center_PowerShell_And_Org_Baseline_Verification_Commands

|   |   |   |   |
|---|---|---|---|
|Check|Device|PowerShell / Command|Healthy Output|
|Exchange admin center loads|Browser|Open https://admin.exchange.microsoft.com|Admin portal opens without access denied|
|PowerShell module installed|Admin Workstation|Get-Module ExchangeOnlineManagement -ListAvailable|Module version and path appear|
|Connect cmdlet exists|Admin Workstation|Get-Command Connect-ExchangeOnline|Cmdlet is found|
|Exchange session active|Admin Workstation|Get-ConnectionInformation|Active Exchange Online connection appears|
|Accepted domains visible|Admin Workstation|Get-AcceptedDomain|One or more accepted domains return|
|Organization config visible|Admin Workstation|Get-OrganizationConfig|Tenant organization config returns|
|Transport config visible|Admin Workstation|Get-TransportConfig|Transport settings return|
|Remote domains visible|Admin Workstation|Get-RemoteDomain|Default remote domain and any custom remote domains return|
|Role groups visible|Admin Workstation|Get-RoleGroup|Exchange role groups return|
|Role group members visible|Admin Workstation|Get-RoleGroupMember "Organization Management"|Members return if permitted|
|Recipient visibility works|Admin Workstation|Get-Recipient -ResultSize 10|Recipients return|
|EXO mailbox cmdlet works|Admin Workstation|Get-EXOMailbox -ResultSize 10|Mailboxes return|
|Mail flow rules visible|Admin Workstation|Get-TransportRule|Rules return or empty list if none exist|
|Connectors visible|Admin Workstation|Get-InboundConnector; Get-OutboundConnector|Connectors return or empty list if none exist|
|Session disconnects cleanly|Admin Workstation|Disconnect-ExchangeOnline -Confirm:$false|Session closes without prompt|

Configure_Exchange_Admin_Center_PowerShell_And_Org_Baseline_Rollback

|   |   |   |   |   |
|---|---|---|---|---|
|Step|Task|Device|PowerShell / Command|Expected Result|
|1|Disconnect Exchange Online session|Admin Workstation|Disconnect-ExchangeOnline -Confirm:$false|Active Exchange session is closed|
|2|Remove local evidence if lab cleanup is required|Admin Workstation|Remove-Item C:\M365-Exchange-Baseline -Recurse -Force|Local baseline files are removed|
|3|Remove ExchangeOnlineManagement module only if required|Admin Workstation|Uninstall-Module ExchangeOnlineManagement -AllVersions|Local module is removed|
|4|Revert PSGallery trust only if changed|Admin Workstation|Set-PSRepository -Name PSGallery -InstallationPolicy Untrusted|Repository policy is reverted|
|5|Close browser admin sessions|Browser|Sign out of Microsoft 365 admin portals|Browser session is closed|

# Rollback helper.

# This workbook is baseline and evidence capture only.

# It should not modify Exchange Online tenant configuration.

  

Disconnect-ExchangeOnline -Confirm:$false

  

# Optional local cleanup:

# Remove-Item C:\M365-Exchange-Baseline -Recurse -Force

  

# Optional module cleanup:

# Uninstall-Module ExchangeOnlineManagement -AllVersions

Configure_Exchange_Admin_Center_PowerShell_And_Org_Baseline_Failure_Checks

|   |   |   |   |
|---|---|---|---|
|Symptom|Likely Cause|Check Command|Fix|
|Exchange admin center shows access denied|Admin lacks Exchange role or Conditional Access blocks session|Check Microsoft 365 admin roles and sign-in logs|Assign correct role or use approved admin workstation|
|Install-Module fails|PowerShell Gallery blocked, TLS issue, proxy issue, or execution policy problem|Get-PSRepository; Test-NetConnection www.powershellgallery.com -Port 443|Fix proxy/TLS path or install module from approved internal repository|
|Connect-ExchangeOnline not recognized|Module missing or not imported|Get-Module ExchangeOnlineManagement -ListAvailable|Install and import ExchangeOnlineManagement|
|Sign-in prompt loops|MFA, browser, WAM, stale credentials, or Conditional Access issue|Try private browser sign-in and check sign-in logs|Clear cached auth, use approved account, or try PowerShell 7/device flow|
|WAM-related connection error|Web Account Manager issue on workstation|Connect-ExchangeOnline ... -DisableWAM|Use -DisableWAM or use a clean admin workstation|
|Connected but cmdlet is missing|RBAC does not grant that cmdlet|Get-ManagementRoleAssignment -RoleAssignee "<admin-upn>"|Add appropriate Exchange role group or use delegated admin model|
|Get-AcceptedDomain fails after connection|Session did not fully connect or RBAC blocks visibility|Get-ConnectionInformation|Reconnect and confirm admin role|
|Get-EXOMailbox returns nothing|No mailbox visibility or wrong tenant/account|Get-Recipient -ResultSize 10|Confirm tenant, role, and recipient scope|
|Get-RoleGroupMember fails|Admin lacks role management permissions|Get-RoleGroup|Use an admin with Organization Management or appropriate role|
|Connector exports are empty|No connectors configured|Get-InboundConnector; Get-OutboundConnector|Empty output is healthy if tenant does not use connectors|
|Transport rule export is empty|No mail flow rules configured|Get-TransportRule|Empty output is healthy if tenant has no rules|
|Disconnect warning appears|Session already closed or profile path issue|Get-ConnectionInformation|Open new PowerShell session and continue|
|Evidence files missing|Script path typo or transcript interrupted|Get-ChildItem C:\M365-Exchange-Baseline -Recurse|Re-run the relevant skeleton block|

Configure_Exchange_Admin_Center_PowerShell_And_Org_Baseline_Related_Labs

|   |   |
|---|---|
|Related Lab|Relationship|
|02_Manage_User_Mailboxes_Shared_Mailboxes_Contacts_And_Resources.md|Uses this baseline connection and recipient inventory|
|03_Manage_Distribution_Groups_Dynamic_Groups_Mail_Enabled_Security_Groups_And_M365_Groups.md|Uses group and RBAC baseline|
|04_Configure_Mail_Flow_Accepted_Domains_Connectors_And_Rules.md|Uses accepted domain, connector, transport config, and rule baseline|
|05_Configure_Mailbox_Permissions_Delegation_Forwarding_And_Auditing.md|Uses admin role and mailbox baseline|
|06_Configure_Exchange_Online_Protection_AntiSpam_AntiMalware_And_Quarantine.md|Uses Exchange and Defender admin access baseline|
|07_Configure_Retention_Archive_Litigation_Hold_And_eDiscovery_Handoff.md|Uses mailbox, archive, and admin role baseline|
|08_Troubleshoot_Mail_Flow_Mailbox_Access_Permissions_And_Quarantine_Issues.md|Uses all exported baseline evidence as comparison data|