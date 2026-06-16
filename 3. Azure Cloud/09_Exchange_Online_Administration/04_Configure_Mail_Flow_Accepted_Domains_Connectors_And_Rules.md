# Configure_Mail_Flow_Accepted_Domains_Connectors_And_Rules

# Configure_Mail_Flow_Accepted_Domains_Connectors_And_Rules_Index
04_Configure_Mail_Flow_Accepted_Domains_Connectors_And_Rules.md
Configure_Mail_Flow_Accepted_Domains_Connectors_And_Rules
Configure_Mail_Flow_Accepted_Domains_Connectors_And_Rules_Source_Basis
Configure_Mail_Flow_Accepted_Domains_Connectors_And_Rules_Mental_Model
Configure_Mail_Flow_Accepted_Domains_Connectors_And_Rules_Planning_Table
Configure_Mail_Flow_Accepted_Domains_Connectors_And_Rules_Configuration_Checklist
Configure_Mail_Flow_Accepted_Domains_Connectors_And_Rules_Precheck_Skeleton
Configure_Mail_Flow_Accepted_Domains_Connectors_And_Rules_Accepted_Domain_Skeleton
Configure_Mail_Flow_Accepted_Domains_Connectors_And_Rules_Remote_Domain_Skeleton
Configure_Mail_Flow_Accepted_Domains_Connectors_And_Rules_Inbound_Connector_Skeleton
Configure_Mail_Flow_Accepted_Domains_Connectors_And_Rules_Outbound_Connector_Skeleton
Configure_Mail_Flow_Accepted_Domains_Connectors_And_Rules_Mail_Flow_Rule_Skeleton
Configure_Mail_Flow_Accepted_Domains_Connectors_And_Rules_Bulk_Inventory_Skeleton
Configure_Mail_Flow_Accepted_Domains_Connectors_And_Rules_Validation_Skeleton
Configure_Mail_Flow_Accepted_Domains_Connectors_And_Rules_Verification_Commands
Configure_Mail_Flow_Accepted_Domains_Connectors_And_Rules_Rollback
Configure_Mail_Flow_Accepted_Domains_Connectors_And_Rules_Failure_Checks
Configure_Mail_Flow_Accepted_Domains_Connectors_And_Rules_Related_Labs

# Configure_Mail_Flow_Accepted_Domains_Connectors_And_Rules_Source_Basis

| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Exchange admin center mail flow | Managing accepted domains, connectors, and mail flow rules |
| Microsoft Learn | Accepted domains in Exchange Online | Adding, validating, setting default, and reviewing authoritative or internal relay domains |
| Microsoft Learn | Connectors in Exchange Online | Configuring inbound and outbound connectors for partner, on-premises, and Microsoft 365 routing scenarios |
| Microsoft Learn | Mail flow rules in Exchange Online | Creating transport rules with conditions, exceptions, actions, priority, and test or enforce mode |
| Microsoft Learn | Get-AcceptedDomain / New-AcceptedDomain / Set-AcceptedDomain | Accepted domain inspection and configuration |
| Microsoft Learn | Get-InboundConnector / New-InboundConnector / Set-InboundConnector | Inbound connector inspection and configuration |
| Microsoft Learn | Get-OutboundConnector / New-OutboundConnector / Set-OutboundConnector | Outbound connector inspection and configuration |
| Microsoft Learn | Get-TransportRule / New-TransportRule / Set-TransportRule | Mail flow rule inspection and configuration |
| Microsoft Learn | Message trace | Testing and validating message routing and rule application |
| Prior workbook | 01_Configure_Exchange_Admin_Center_PowerShell_And_Org_Baseline.md | Provides Exchange Online PowerShell connection and mail flow baseline |
| Prior workbook | 02_Manage_User_Mailboxes_Shared_Mailboxes_Contacts_And_Resources.md | Provides recipients used for test messages and rule conditions |
| Prior workbook | 03_Manage_Distribution_Groups_Dynamic_Groups_Mail_Enabled_Security_Groups_And_M365_Groups.md | Provides groups used for mail flow rule targets and exceptions |

# Configure_Mail_Flow_Accepted_Domains_Connectors_And_Rules_Mental_Model

| Concept | Operational Meaning |
|---|---|
| Mail flow | End-to-end path of messages through Exchange Online |
| Accepted domain | SMTP domain Exchange Online accepts mail for |
| Authoritative domain | Exchange Online is responsible for final delivery for the domain |
| Internal relay domain | Exchange Online accepts mail and relays unresolved recipients to another mail system |
| Default accepted domain | Domain used as the default address domain in the tenant |
| Remote domain | Policy object that controls mail behavior to external domains |
| Connector | Controlled routing path for mail entering or leaving Exchange Online |
| Inbound connector | Defines trusted inbound mail path from partner, on-premises, or another service into Exchange Online |
| Outbound connector | Defines outbound routing path from Exchange Online to partner, on-premises, or another mail system |
| Smart host | Mail server Exchange Online routes outbound mail to |
| TLS requirement | Connector setting requiring encrypted authenticated transport |
| Certificate validation | Connector trust method based on certificate subject or issuer match |
| Sender domain validation | Inbound connector control based on sending domain |
| Sender IP validation | Inbound connector control based on sender IP address |
| Transport rule | Mail flow rule that evaluates messages in transit |
| Rule condition | Match logic for a mail flow rule |
| Rule exception | Logic that bypasses a mail flow rule |
| Rule action | Enforcement action, such as prepend subject, redirect, reject, quarantine, apply disclaimer, or modify message |
| Rule priority | Processing order for mail flow rules |
| Rule mode | Enforce, test with policy tips, or test without policy tips |
| First rule | Do not change accepted domains or connectors without DNS, ownership, routing, and rollback evidence |
| Second rule | Test mail flow rules in test mode before enforcing if the rule can interrupt mail delivery |
| Third rule | Connector mistakes break mail routing faster than most Exchange admin changes |

# Configure_Mail_Flow_Accepted_Domains_Connectors_And_Rules_Planning_Table

| Item | Example | Decision |
|---|---|---|
| Tenant name | `contoso.onmicrosoft.com` | `<tenant-name>` |
| Admin UPN | `admin@contoso.com` | `<admin-upn>` |
| Evidence root | `C:\M365-Exchange-Baseline` | `<evidence-root>` |
| Primary accepted domain | `contoso.com` | `<primary-domain>` |
| Additional accepted domain | `subsidiary.contoso.com` | `<additional-domain>` |
| Domain type | Authoritative / InternalRelay | `<domain-type>` |
| Default domain change required | No | `<yes-no>` |
| Domain verified in Microsoft 365 | Yes | `<yes-no>` |
| MX record target | `contoso-com.mail.protection.outlook.com` | `<mx-target>` |
| SPF record | `v=spf1 include:spf.protection.outlook.com -all` | `<spf-record>` |
| DKIM configured elsewhere | Yes / No | `<yes-no>` |
| DMARC configured elsewhere | Yes / No | `<yes-no>` |
| Inbound connector purpose | Partner relay / On-prem relay / Third-party filter | `<inbound-purpose>` |
| Inbound connector sender domains | `partner.example` | `<sender-domains>` |
| Inbound connector sender IPs | `203.0.113.10` | `<sender-ip-list>` |
| Inbound connector TLS requirement | Required | `<required-optional>` |
| Outbound connector purpose | Partner smart host / On-prem hybrid / Third-party gateway | `<outbound-purpose>` |
| Outbound connector recipient domains | `partner.example` | `<recipient-domains>` |
| Outbound smart host | `mail.partner.example` | `<smart-host>` |
| Outbound TLS requirement | Required | `<required-optional>` |
| Mail flow rule purpose | Tag external mail / block attachment / route messages / apply disclaimer | `<rule-purpose>` |
| Rule test mode first | Yes | `<yes-no>` |
| Rule priority | `0` or after existing critical rules | `<priority>` |
| Test sender | `alex.wilber@contoso.com` | `<test-sender>` |
| Test recipient | `adele.vance@contoso.com` | `<test-recipient>` |
| Change ticket | `LAB-EXO-004` | `<ticket-or-lab-id>` |
| Rollback stance | Disable new connectors and rules before removal | `<rollback-scope>` |

# Configure_Mail_Flow_Accepted_Domains_Connectors_And_Rules_Configuration_Checklist

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm Exchange Online session | Admin Workstation | `Get-ConnectionInformation` | Active Exchange Online session exists |
| 2 | Confirm current accepted domains | Admin Workstation | `Get-AcceptedDomain` | Existing accepted domains are visible |
| 3 | Confirm current transport config | Admin Workstation | `Get-TransportConfig` | Tenant mail transport settings are visible |
| 4 | Confirm current remote domains | Admin Workstation | `Get-RemoteDomain` | Remote domain policies are visible |
| 5 | Export current inbound connectors | Admin Workstation | `Get-InboundConnector` | Existing inbound connectors are saved |
| 6 | Export current outbound connectors | Admin Workstation | `Get-OutboundConnector` | Existing outbound connectors are saved |
| 7 | Export current transport rules | Admin Workstation | `Get-TransportRule` | Existing mail flow rules are saved |
| 8 | Validate domain ownership outside Exchange | Admin Workstation / DNS portal | Confirm Microsoft 365 domain verification | Domain is verified before accepted domain changes |
| 9 | Add accepted domain if required | Admin Workstation | `New-AcceptedDomain` | Accepted domain exists |
| 10 | Configure accepted domain type | Admin Workstation | `Set-AcceptedDomain` | Domain type matches routing design |
| 11 | Validate MX and SPF records | Admin Workstation | `Resolve-DnsName` | Public DNS records match design |
| 12 | Configure remote domain policy if required | Admin Workstation | `Set-RemoteDomain` or `New-RemoteDomain` | Remote domain behavior matches design |
| 13 | Create inbound connector if required | Admin Workstation | `New-InboundConnector` | Inbound connector exists |
| 14 | Validate inbound connector settings | Admin Workstation | `Get-InboundConnector -Identity "<name>"` | Sender domains, IPs, TLS, and enabled state match design |
| 15 | Create outbound connector if required | Admin Workstation | `New-OutboundConnector` | Outbound connector exists |
| 16 | Validate outbound connector settings | Admin Workstation | `Get-OutboundConnector -Identity "<name>"` | Recipient domains, smart hosts, TLS, and enabled state match design |
| 17 | Create mail flow rule in test mode | Admin Workstation | `New-TransportRule -Mode TestWithoutPolicyTips` | Rule exists but does not enforce |
| 18 | Validate rule conditions, exceptions, and actions | Admin Workstation | `Get-TransportRule -Identity "<rule-name>"` | Rule logic matches design |
| 19 | Adjust rule priority | Admin Workstation | `Set-TransportRule -Priority <number>` | Rule order is correct |
| 20 | Send test messages | Outlook / OWA | Send messages matching and not matching the rule | Expected rule behavior is observed |
| 21 | Run message trace | Admin Workstation / EAC | `Get-MessageTrace` or EAC message trace | Message path and status are visible |
| 22 | Enforce rule only after test validation | Admin Workstation | `Set-TransportRule -Mode Enforce` | Rule begins enforcing |
| 23 | Export final mail flow configuration | Admin Workstation | Export accepted domains, connectors, remote domains, and rules | Final evidence is saved |
| 24 | Disconnect Exchange Online session | Admin Workstation | `Disconnect-ExchangeOnline -Confirm:$false` | Session closes cleanly |

# Configure_Mail_Flow_Accepted_Domains_Connectors_And_Rules_Precheck_Skeleton

```powershell
# Run on the admin workstation.
# Purpose: capture current Exchange Online mail flow baseline before accepted domain, connector, or rule changes.

$AdminUpn = "admin@contoso.com"
$PrimaryDomain = "contoso.com"
$EvidenceRoot = "C:\M365-Exchange-Baseline"
$RunStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$EvidencePath = Join-Path $EvidenceRoot "04-Mail-Flow-Precheck-$RunStamp"

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\mail-flow-precheck-transcript.txt"

Import-Module ExchangeOnlineManagement

try {
  $Connection = Get-ConnectionInformation -ErrorAction Stop
}
catch {
  Connect-ExchangeOnline -UserPrincipalName $AdminUpn -ShowBanner:$false
}

Get-ConnectionInformation |
  Tee-Object "$EvidencePath\connection-information.txt"

# Accepted domains.
Get-AcceptedDomain |
  Select-Object Name,DomainName,DomainType,Default,AddressBookEnabled,PendingRemoval |
  Export-Csv "$EvidencePath\accepted-domains-before.csv" -NoTypeInformation

Get-AcceptedDomain |
  Format-List * |
  Out-File "$EvidencePath\accepted-domains-before-full.txt"

# Transport configuration.
Get-TransportConfig |
  Format-List * |
  Out-File "$EvidencePath\transport-config-before-full.txt"

# Remote domains.
Get-RemoteDomain |
  Select-Object Name,DomainName,AllowedOOFType,AutoReplyEnabled,AutoForwardEnabled,DeliveryReportEnabled,TNEFEnabled |
  Export-Csv "$EvidencePath\remote-domains-before.csv" -NoTypeInformation

Get-RemoteDomain |
  Format-List * |
  Out-File "$EvidencePath\remote-domains-before-full.txt"

# Connectors.
Get-InboundConnector |
  Select-Object Name,Enabled,ConnectorType,SenderDomains,SenderIPAddresses,RequireTls,CloudServicesMailEnabled,RestrictDomainsToIPAddresses,TlsSenderCertificateName |
  Export-Csv "$EvidencePath\inbound-connectors-before.csv" -NoTypeInformation

Get-InboundConnector |
  Format-List * |
  Out-File "$EvidencePath\inbound-connectors-before-full.txt"

Get-OutboundConnector |
  Select-Object Name,Enabled,ConnectorType,RecipientDomains,SmartHosts,TlsSettings,RouteAllMessagesViaOnPremises,CloudServicesMailEnabled,TlsDomain |
  Export-Csv "$EvidencePath\outbound-connectors-before.csv" -NoTypeInformation

Get-OutboundConnector |
  Format-List * |
  Out-File "$EvidencePath\outbound-connectors-before-full.txt"

# Mail flow rules.
Get-TransportRule |
  Select-Object Name,State,Mode,Priority,RuleVersion,Comments |
  Export-Csv "$EvidencePath\transport-rules-before.csv" -NoTypeInformation

Get-TransportRule |
  Format-List * |
  Out-File "$EvidencePath\transport-rules-before-full.txt"

# DNS evidence for primary domain.
Resolve-DnsName -Name $PrimaryDomain -Type MX -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dns-mx-$PrimaryDomain.txt"

Resolve-DnsName -Name $PrimaryDomain -Type TXT -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dns-txt-$PrimaryDomain.txt"

# Test recipient visibility.
Get-Recipient -ResultSize 50 |
  Select-Object DisplayName,PrimarySmtpAddress,RecipientTypeDetails |
  Export-Csv "$EvidencePath\recipient-sample-before.csv" -NoTypeInformation

Stop-Transcript
```

# Configure_Mail_Flow_Accepted_Domains_Connectors_And_Rules_Accepted_Domain_Skeleton

```powershell
# Run on the admin workstation.
# Purpose: add or configure an accepted domain.
# Important: Domain ownership should be verified in Microsoft 365 before using the domain for production mail.

$EvidenceRoot = "C:\M365-Exchange-Baseline"
$RunStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$EvidencePath = Join-Path $EvidenceRoot "04-Accepted-Domain-$RunStamp"

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\accepted-domain-transcript.txt"

# Variables.
$DomainName = "subsidiary.contoso.com"
$AcceptedDomainName = "Subsidiary Contoso"
$DomainType = "Authoritative"
$MakeDefault = $false

# Valid DomainType examples:
# Authoritative
# InternalRelay

# Capture before state.
Get-AcceptedDomain |
  Select-Object Name,DomainName,DomainType,Default,AddressBookEnabled |
  Export-Csv "$EvidencePath\accepted-domains-before.csv" -NoTypeInformation

# Precheck for existing domain.
$ExistingDomain = Get-AcceptedDomain -Identity $DomainName -ErrorAction SilentlyContinue

if (-not $ExistingDomain) {
  New-AcceptedDomain `
    -Name $AcceptedDomainName `
    -DomainName $DomainName `
    -DomainType $DomainType
}
else {
  Set-AcceptedDomain `
    -Identity $DomainName `
    -DomainType $DomainType
}

# Optional: make the domain default only if planned.
if ($MakeDefault -eq $true) {
  Set-AcceptedDomain `
    -Identity $DomainName `
    -MakeDefault $true
}

# Validate.
Get-AcceptedDomain -Identity $DomainName |
  Format-List Name,DomainName,DomainType,Default,AddressBookEnabled,PendingRemoval |
  Out-File "$EvidencePath\accepted-domain-after.txt"

Get-AcceptedDomain |
  Select-Object Name,DomainName,DomainType,Default,AddressBookEnabled |
  Export-Csv "$EvidencePath\accepted-domains-after.csv" -NoTypeInformation

# DNS evidence.
Resolve-DnsName -Name $DomainName -Type MX -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dns-mx-$DomainName.txt"

Resolve-DnsName -Name $DomainName -Type TXT -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dns-txt-$DomainName.txt"

Stop-Transcript
```

# Configure_Mail_Flow_Accepted_Domains_Connectors_And_Rules_Remote_Domain_Skeleton

```powershell
# Run on the admin workstation.
# Purpose: configure remote domain behavior for a specific external domain.
# Remote domains control external mail behavior such as auto-forwarding, auto-replies, delivery reports, and message formatting.

$EvidenceRoot = "C:\M365-Exchange-Baseline"
$RunStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$EvidencePath = Join-Path $EvidenceRoot "04-Remote-Domain-$RunStamp"

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\remote-domain-transcript.txt"

# Variables.
$RemoteDomainName = "Partner Example"
$RemoteDomain = "partner.example"

# Capture before state.
Get-RemoteDomain |
  Select-Object Name,DomainName,AllowedOOFType,AutoReplyEnabled,AutoForwardEnabled,DeliveryReportEnabled,TNEFEnabled |
  Export-Csv "$EvidencePath\remote-domains-before.csv" -NoTypeInformation

# Create remote domain if it does not exist.
if (-not (Get-RemoteDomain -Identity $RemoteDomainName -ErrorAction SilentlyContinue)) {
  New-RemoteDomain `
    -Name $RemoteDomainName `
    -DomainName $RemoteDomain
}

# Configure behavior.
Set-RemoteDomain -Identity $RemoteDomainName `
  -AllowedOOFType External `
  -AutoReplyEnabled $true `
  -AutoForwardEnabled $false `
  -DeliveryReportEnabled $true `
  -TNEFEnabled $false

# Validate.
Get-RemoteDomain -Identity $RemoteDomainName |
  Format-List Name,DomainName,AllowedOOFType,AutoReplyEnabled,AutoForwardEnabled,DeliveryReportEnabled,TNEFEnabled |
  Out-File "$EvidencePath\remote-domain-after.txt"

Get-RemoteDomain |
  Select-Object Name,DomainName,AllowedOOFType,AutoReplyEnabled,AutoForwardEnabled,DeliveryReportEnabled,TNEFEnabled |
  Export-Csv "$EvidencePath\remote-domains-after.csv" -NoTypeInformation

Stop-Transcript
```

# Configure_Mail_Flow_Accepted_Domains_Connectors_And_Rules_Inbound_Connector_Skeleton

```powershell
# Run on the admin workstation.
# Purpose: create and validate an inbound connector.
# Example scenario: trusted partner or mail gateway sends mail into Exchange Online from fixed public IP addresses.

$EvidenceRoot = "C:\M365-Exchange-Baseline"
$RunStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$EvidencePath = Join-Path $EvidenceRoot "04-Inbound-Connector-$RunStamp"

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\inbound-connector-transcript.txt"

# Variables.
$ConnectorName = "Inbound from Partner Example"
$SenderDomains = @("partner.example")
$SenderIPAddresses = @("203.0.113.10","203.0.113.11")
$RequireTls = $true
$ConnectorType = "Partner"

# Valid ConnectorType examples:
# Partner
# OnPremises

# Capture before state.
Get-InboundConnector |
  Select-Object Name,Enabled,ConnectorType,SenderDomains,SenderIPAddresses,RequireTls,RestrictDomainsToIPAddresses,TlsSenderCertificateName |
  Export-Csv "$EvidencePath\inbound-connectors-before.csv" -NoTypeInformation

# Precheck.
Get-InboundConnector -Identity $ConnectorName -ErrorAction SilentlyContinue |
  Format-List * |
  Out-File "$EvidencePath\inbound-connector-existing-check.txt"

# Create connector if it does not exist.
if (-not (Get-InboundConnector -Identity $ConnectorName -ErrorAction SilentlyContinue)) {
  New-InboundConnector `
    -Name $ConnectorName `
    -ConnectorType $ConnectorType `
    -SenderDomains $SenderDomains `
    -SenderIPAddresses $SenderIPAddresses `
    -RequireTls $RequireTls `
    -RestrictDomainsToIPAddresses $true `
    -Enabled $true
}
else {
  Set-InboundConnector `
    -Identity $ConnectorName `
    -SenderDomains $SenderDomains `
    -SenderIPAddresses $SenderIPAddresses `
    -RequireTls $RequireTls `
    -RestrictDomainsToIPAddresses $true `
    -Enabled $true
}

# Validate.
Get-InboundConnector -Identity $ConnectorName |
  Format-List Name,Enabled,ConnectorType,SenderDomains,SenderIPAddresses,RequireTls,RestrictDomainsToIPAddresses,TlsSenderCertificateName,CloudServicesMailEnabled |
  Out-File "$EvidencePath\inbound-connector-after.txt"

Get-InboundConnector |
  Select-Object Name,Enabled,ConnectorType,SenderDomains,SenderIPAddresses,RequireTls,RestrictDomainsToIPAddresses,TlsSenderCertificateName |
  Export-Csv "$EvidencePath\inbound-connectors-after.csv" -NoTypeInformation

Stop-Transcript
```

# Configure_Mail_Flow_Accepted_Domains_Connectors_And_Rules_Outbound_Connector_Skeleton

```powershell
# Run on the admin workstation.
# Purpose: create and validate an outbound connector.
# Example scenario: route mail for partner.example through partner smart host using TLS.

$EvidenceRoot = "C:\M365-Exchange-Baseline"
$RunStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$EvidencePath = Join-Path $EvidenceRoot "04-Outbound-Connector-$RunStamp"

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\outbound-connector-transcript.txt"

# Variables.
$ConnectorName = "Outbound to Partner Example"
$RecipientDomains = @("partner.example")
$SmartHosts = @("mail.partner.example")
$ConnectorType = "Partner"
$TlsSettings = "DomainValidation"
$TlsDomain = "partner.example"

# Valid ConnectorType examples:
# Partner
# OnPremises

# Valid TlsSettings examples commonly used:
# EncryptionOnly
# CertificateValidation
# DomainValidation

# Capture before state.
Get-OutboundConnector |
  Select-Object Name,Enabled,ConnectorType,RecipientDomains,SmartHosts,TlsSettings,TlsDomain,RouteAllMessagesViaOnPremises,CloudServicesMailEnabled |
  Export-Csv "$EvidencePath\outbound-connectors-before.csv" -NoTypeInformation

# DNS evidence for smart host.
foreach ($SmartHost in $SmartHosts) {
  Resolve-DnsName -Name $SmartHost -Type A -ErrorAction SilentlyContinue |
    Tee-Object "$EvidencePath\dns-a-$SmartHost.txt"

  Resolve-DnsName -Name $SmartHost -Type AAAA -ErrorAction SilentlyContinue |
    Tee-Object "$EvidencePath\dns-aaaa-$SmartHost.txt"
}

# Precheck.
Get-OutboundConnector -Identity $ConnectorName -ErrorAction SilentlyContinue |
  Format-List * |
  Out-File "$EvidencePath\outbound-connector-existing-check.txt"

# Create connector if it does not exist.
if (-not (Get-OutboundConnector -Identity $ConnectorName -ErrorAction SilentlyContinue)) {
  New-OutboundConnector `
    -Name $ConnectorName `
    -ConnectorType $ConnectorType `
    -RecipientDomains $RecipientDomains `
    -SmartHosts $SmartHosts `
    -TlsSettings $TlsSettings `
    -TlsDomain $TlsDomain `
    -UseMXRecord $false `
    -Enabled $true
}
else {
  Set-OutboundConnector `
    -Identity $ConnectorName `
    -RecipientDomains $RecipientDomains `
    -SmartHosts $SmartHosts `
    -TlsSettings $TlsSettings `
    -TlsDomain $TlsDomain `
    -UseMXRecord $false `
    -Enabled $true
}

# Validate.
Get-OutboundConnector -Identity $ConnectorName |
  Format-List Name,Enabled,ConnectorType,RecipientDomains,SmartHosts,TlsSettings,TlsDomain,UseMXRecord,RouteAllMessagesViaOnPremises,CloudServicesMailEnabled |
  Out-File "$EvidencePath\outbound-connector-after.txt"

Get-OutboundConnector |
  Select-Object Name,Enabled,ConnectorType,RecipientDomains,SmartHosts,TlsSettings,TlsDomain,UseMXRecord,RouteAllMessagesViaOnPremises |
  Export-Csv "$EvidencePath\outbound-connectors-after.csv" -NoTypeInformation

Stop-Transcript
```

# Configure_Mail_Flow_Accepted_Domains_Connectors_And_Rules_Mail_Flow_Rule_Skeleton

```powershell
# Run on the admin workstation.
# Purpose: create a mail flow rule in test mode, validate it, then optionally enforce it.
# Example rule: prepend EXTERNAL to messages from outside the organization.

$EvidenceRoot = "C:\M365-Exchange-Baseline"
$RunStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$EvidencePath = Join-Path $EvidenceRoot "04-Mail-Flow-Rule-$RunStamp"

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\mail-flow-rule-transcript.txt"

# Variables.
$RuleName = "LAB - Prepend EXTERNAL To External Mail"
$RuleComments = "Lab mail flow rule for Exchange Online workbook 04. Created in test mode first."
$RuleMode = "TestWithoutPolicyTips"
$SubjectPrefix = "[EXTERNAL] "

# Capture before state.
Get-TransportRule |
  Select-Object Name,State,Mode,Priority,RuleVersion,Comments |
  Export-Csv "$EvidencePath\transport-rules-before.csv" -NoTypeInformation

Get-TransportRule -Identity $RuleName -ErrorAction SilentlyContinue |
  Format-List * |
  Out-File "$EvidencePath\transport-rule-existing-check.txt"

# Create rule if it does not exist.
if (-not (Get-TransportRule -Identity $RuleName -ErrorAction SilentlyContinue)) {
  New-TransportRule `
    -Name $RuleName `
    -FromScope NotInOrganization `
    -PrependSubject $SubjectPrefix `
    -Mode $RuleMode `
    -Comments $RuleComments `
    -Enabled $true
}
else {
  Set-TransportRule `
    -Identity $RuleName `
    -FromScope NotInOrganization `
    -PrependSubject $SubjectPrefix `
    -Mode $RuleMode `
    -Comments $RuleComments `
    -Enabled $true
}

# Optional: add exception so repeated subject stamping is avoided.
# Some environments use rule predicates or exceptions for this. Validate before production use.
# Example:
# Set-TransportRule -Identity $RuleName -ExceptIfSubjectOrBodyMatchesPatterns "\[EXTERNAL\]"

# Set priority if required.
# Lower number means earlier processing.
Set-TransportRule -Identity $RuleName -Priority 0

# Validate rule.
Get-TransportRule -Identity $RuleName |
  Format-List Name,State,Mode,Priority,FromScope,PrependSubject,Comments |
  Out-File "$EvidencePath\transport-rule-after.txt"

Get-TransportRule |
  Select-Object Name,State,Mode,Priority,RuleVersion,Comments |
  Export-Csv "$EvidencePath\transport-rules-after.csv" -NoTypeInformation

# Enforcement step:
# Only run after validation with test messages and message trace.
# Set-TransportRule -Identity $RuleName -Mode Enforce

Stop-Transcript
```

# Configure_Mail_Flow_Accepted_Domains_Connectors_And_Rules_Mail_Flow_Rule_Attachment_Block_Skeleton

```powershell
# Run on the admin workstation.
# Purpose: example of a safer staged rule for blocking risky attachment types.
# Create in test mode first. Enforce only after business approval.

$EvidenceRoot = "C:\M365-Exchange-Baseline"
$RunStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$EvidencePath = Join-Path $EvidenceRoot "04-Mail-Flow-Rule-Attachment-Block-$RunStamp"

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\attachment-block-rule-transcript.txt"

# Variables.
$RuleName = "LAB - Block Executable Attachments"
$RuleMode = "TestWithoutPolicyTips"
$RejectText = "Executable attachments are not accepted by this organization."

# Create in test mode.
if (-not (Get-TransportRule -Identity $RuleName -ErrorAction SilentlyContinue)) {
  New-TransportRule `
    -Name $RuleName `
    -AttachmentExtensionMatchesWords @("exe","scr","bat","cmd","js","vbs") `
    -RejectMessageReasonText $RejectText `
    -Mode $RuleMode `
    -Comments "Lab rule created in test mode for workbook 04." `
    -Enabled $true
}
else {
  Set-TransportRule `
    -Identity $RuleName `
    -AttachmentExtensionMatchesWords @("exe","scr","bat","cmd","js","vbs") `
    -RejectMessageReasonText $RejectText `
    -Mode $RuleMode `
    -Enabled $true
}

# Validate.
Get-TransportRule -Identity $RuleName |
  Format-List Name,State,Mode,Priority,AttachmentExtensionMatchesWords,RejectMessageReasonText,Comments |
  Out-File "$EvidencePath\attachment-block-rule-after.txt"

# Enforcement step:
# Only after validation.
# Set-TransportRule -Identity $RuleName -Mode Enforce

Stop-Transcript
```

# Configure_Mail_Flow_Accepted_Domains_Connectors_And_Rules_Bulk_Inventory_Skeleton

```powershell
# Run on the admin workstation.
# Purpose: export full mail flow configuration after changes.

$EvidenceRoot = "C:\M365-Exchange-Baseline"
$RunStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$EvidencePath = Join-Path $EvidenceRoot "04-Mail-Flow-Inventory-$RunStamp"

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\mail-flow-inventory-transcript.txt"

# Accepted domains.
Get-AcceptedDomain |
  Select-Object Name,DomainName,DomainType,Default,AddressBookEnabled,PendingRemoval |
  Export-Csv "$EvidencePath\accepted-domains.csv" -NoTypeInformation

Get-AcceptedDomain |
  Format-List * |
  Out-File "$EvidencePath\accepted-domains-full.txt"

# Transport config.
Get-TransportConfig |
  Format-List * |
  Out-File "$EvidencePath\transport-config-full.txt"

# Remote domains.
Get-RemoteDomain |
  Select-Object Name,DomainName,AllowedOOFType,AutoReplyEnabled,AutoForwardEnabled,DeliveryReportEnabled,TNEFEnabled |
  Export-Csv "$EvidencePath\remote-domains.csv" -NoTypeInformation

Get-RemoteDomain |
  Format-List * |
  Out-File "$EvidencePath\remote-domains-full.txt"

# Inbound connectors.
Get-InboundConnector |
  Select-Object Name,Enabled,ConnectorType,SenderDomains,SenderIPAddresses,RequireTls,RestrictDomainsToIPAddresses,TlsSenderCertificateName,CloudServicesMailEnabled |
  Export-Csv "$EvidencePath\inbound-connectors.csv" -NoTypeInformation

Get-InboundConnector |
  Format-List * |
  Out-File "$EvidencePath\inbound-connectors-full.txt"

# Outbound connectors.
Get-OutboundConnector |
  Select-Object Name,Enabled,ConnectorType,RecipientDomains,SmartHosts,TlsSettings,TlsDomain,UseMXRecord,RouteAllMessagesViaOnPremises,CloudServicesMailEnabled |
  Export-Csv "$EvidencePath\outbound-connectors.csv" -NoTypeInformation

Get-OutboundConnector |
  Format-List * |
  Out-File "$EvidencePath\outbound-connectors-full.txt"

# Mail flow rules.
Get-TransportRule |
  Select-Object Name,State,Mode,Priority,RuleVersion,Comments |
  Export-Csv "$EvidencePath\transport-rules.csv" -NoTypeInformation

Get-TransportRule |
  Format-List * |
  Out-File "$EvidencePath\transport-rules-full.txt"

# Rule order view.
Get-TransportRule |
  Sort-Object Priority |
  Select-Object Priority,Name,State,Mode |
  Format-Table -AutoSize |
  Out-File "$EvidencePath\transport-rule-order.txt"

Stop-Transcript
```

# Configure_Mail_Flow_Accepted_Domains_Connectors_And_Rules_Validation_Skeleton

```powershell
# Run on the admin workstation.
# Purpose: validate accepted domains, connectors, rules, DNS, and recent message trace.

$EvidenceRoot = "C:\M365-Exchange-Baseline"
$RunStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$EvidencePath = Join-Path $EvidenceRoot "04-Mail-Flow-Validation-$RunStamp"

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\mail-flow-validation-transcript.txt"

# Variables.
$PrimaryDomain = "contoso.com"
$AcceptedDomain = "subsidiary.contoso.com"
$InboundConnectorName = "Inbound from Partner Example"
$OutboundConnectorName = "Outbound to Partner Example"
$TransportRuleName = "LAB - Prepend EXTERNAL To External Mail"
$TestSender = "alex.wilber@contoso.com"
$TestRecipient = "adele.vance@contoso.com"
$TraceStart = (Get-Date).AddHours(-24)
$TraceEnd = Get-Date

# Validate accepted domains.
Get-AcceptedDomain -Identity $PrimaryDomain -ErrorAction SilentlyContinue |
  Format-List Name,DomainName,DomainType,Default |
  Out-File "$EvidencePath\validate-primary-accepted-domain.txt"

Get-AcceptedDomain -Identity $AcceptedDomain -ErrorAction SilentlyContinue |
  Format-List Name,DomainName,DomainType,Default |
  Out-File "$EvidencePath\validate-added-accepted-domain.txt"

# Validate DNS.
Resolve-DnsName -Name $PrimaryDomain -Type MX -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\validate-primary-domain-mx.txt"

Resolve-DnsName -Name $PrimaryDomain -Type TXT -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\validate-primary-domain-txt.txt"

Resolve-DnsName -Name $AcceptedDomain -Type MX -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\validate-added-domain-mx.txt"

Resolve-DnsName -Name $AcceptedDomain -Type TXT -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\validate-added-domain-txt.txt"

# Validate remote domains.
Get-RemoteDomain |
  Select-Object Name,DomainName,AllowedOOFType,AutoReplyEnabled,AutoForwardEnabled,TNEFEnabled |
  Export-Csv "$EvidencePath\validate-remote-domains.csv" -NoTypeInformation

# Validate connectors.
Get-InboundConnector -Identity $InboundConnectorName -ErrorAction SilentlyContinue |
  Format-List Name,Enabled,ConnectorType,SenderDomains,SenderIPAddresses,RequireTls,RestrictDomainsToIPAddresses,TlsSenderCertificateName |
  Out-File "$EvidencePath\validate-inbound-connector.txt"

Get-OutboundConnector -Identity $OutboundConnectorName -ErrorAction SilentlyContinue |
  Format-List Name,Enabled,ConnectorType,RecipientDomains,SmartHosts,TlsSettings,TlsDomain,UseMXRecord |
  Out-File "$EvidencePath\validate-outbound-connector.txt"

# Validate transport rule.
Get-TransportRule -Identity $TransportRuleName -ErrorAction SilentlyContinue |
  Format-List Name,State,Mode,Priority,FromScope,PrependSubject,Comments |
  Out-File "$EvidencePath\validate-transport-rule.txt"

# Validate rule order.
Get-TransportRule |
  Sort-Object Priority |
  Select-Object Priority,Name,State,Mode |
  Export-Csv "$EvidencePath\validate-transport-rule-order.csv" -NoTypeInformation

# Message trace validation.
# Requires message activity in the last 10 days depending on tenant availability.
Get-MessageTrace `
  -StartDate $TraceStart `
  -EndDate $TraceEnd `
  -SenderAddress $TestSender `
  -RecipientAddress $TestRecipient |
  Select-Object Received,SenderAddress,RecipientAddress,Subject,Status,MessageTraceId |
  Export-Csv "$EvidencePath\validate-message-trace.csv" -NoTypeInformation

Stop-Transcript
```

# Configure_Mail_Flow_Accepted_Domains_Connectors_And_Rules_Verification_Commands

| Check | Device | PowerShell / Command | Healthy Output |
|---|---|---|---|
| Exchange Online session active | Admin Workstation | `Get-ConnectionInformation` | Active session exists |
| Accepted domains visible | Admin Workstation | `Get-AcceptedDomain` | Domains return |
| Target domain exists | Admin Workstation | `Get-AcceptedDomain -Identity "<domain>"` | Domain object returns |
| Target domain type correct | Admin Workstation | `Get-AcceptedDomain -Identity "<domain>" \| Format-List DomainType` | Authoritative or InternalRelay matches design |
| Default accepted domain known | Admin Workstation | `Get-AcceptedDomain \| Where-Object {$_.Default -eq $true}` | One default domain appears |
| MX record resolves | Admin Workstation | `Resolve-DnsName <domain> -Type MX` | MX target is visible |
| SPF TXT resolves | Admin Workstation | `Resolve-DnsName <domain> -Type TXT` | SPF record is visible if configured |
| Transport config visible | Admin Workstation | `Get-TransportConfig` | Transport settings return |
| Remote domains visible | Admin Workstation | `Get-RemoteDomain` | Default and custom remote domains return |
| Inbound connector exists | Admin Workstation | `Get-InboundConnector -Identity "<name>"` | Connector returns |
| Inbound connector enabled | Admin Workstation | `Get-InboundConnector -Identity "<name>" \| Format-List Enabled` | Enabled matches design |
| Inbound sender IPs correct | Admin Workstation | `Get-InboundConnector -Identity "<name>" \| Format-List SenderIPAddresses` | Expected IPs appear |
| Inbound TLS correct | Admin Workstation | `Get-InboundConnector -Identity "<name>" \| Format-List RequireTls` | TLS requirement matches design |
| Outbound connector exists | Admin Workstation | `Get-OutboundConnector -Identity "<name>"` | Connector returns |
| Outbound recipient domains correct | Admin Workstation | `Get-OutboundConnector -Identity "<name>" \| Format-List RecipientDomains` | Expected recipient domains appear |
| Outbound smart host correct | Admin Workstation | `Get-OutboundConnector -Identity "<name>" \| Format-List SmartHosts` | Expected smart host appears |
| Outbound TLS correct | Admin Workstation | `Get-OutboundConnector -Identity "<name>" \| Format-List TlsSettings,TlsDomain` | TLS settings match design |
| Transport rule exists | Admin Workstation | `Get-TransportRule -Identity "<rule-name>"` | Rule returns |
| Transport rule mode safe | Admin Workstation | `Get-TransportRule -Identity "<rule-name>" \| Format-List Mode` | Test mode before enforcement |
| Transport rule priority correct | Admin Workstation | `Get-TransportRule \| Sort Priority \| Select Priority,Name,Mode` | Rule order is expected |
| Message trace shows test message | Admin Workstation | `Get-MessageTrace -StartDate (Get-Date).AddHours(-24) -EndDate (Get-Date)` | Test message appears |
| EAC accepted domains view | Browser | Exchange admin center > Mail flow > Accepted domains | Accepted domains appear |
| EAC connectors view | Browser | Exchange admin center > Mail flow > Connectors | Connectors appear |
| EAC rules view | Browser | Exchange admin center > Mail flow > Rules | Rules appear |

# Configure_Mail_Flow_Accepted_Domains_Connectors_And_Rules_Rollback

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm rollback scope | Operator | Review lab-created domain, connectors, remote domain, and rules | Only lab objects are targeted |
| 2 | Export current state before rollback | Admin Workstation | Export accepted domains, connectors, remote domains, rules | Pre-rollback evidence is saved |
| 3 | Disable lab transport rule | Admin Workstation | `Disable-TransportRule -Identity "<rule-name>"` | Rule stops processing |
| 4 | Remove lab transport rule if approved | Admin Workstation | `Remove-TransportRule -Identity "<rule-name>" -Confirm:$false` | Rule is removed |
| 5 | Disable lab inbound connector | Admin Workstation | `Set-InboundConnector -Identity "<name>" -Enabled $false` | Inbound connector disabled |
| 6 | Remove lab inbound connector if approved | Admin Workstation | `Remove-InboundConnector -Identity "<name>" -Confirm:$false` | Inbound connector removed |
| 7 | Disable lab outbound connector | Admin Workstation | `Set-OutboundConnector -Identity "<name>" -Enabled $false` | Outbound connector disabled |
| 8 | Remove lab outbound connector if approved | Admin Workstation | `Remove-OutboundConnector -Identity "<name>" -Confirm:$false` | Outbound connector removed |
| 9 | Revert remote domain policy | Admin Workstation | `Set-RemoteDomain` or `Remove-RemoteDomain` | Remote domain behavior is restored |
| 10 | Revert accepted domain type if changed | Admin Workstation | `Set-AcceptedDomain -Identity "<domain>" -DomainType "<previous-type>"` | Domain type returns to previous state |
| 11 | Remove lab accepted domain only if safe | Admin Workstation | `Remove-AcceptedDomain -Identity "<domain>" -Confirm:$false` | Domain is removed if no recipients depend on it |
| 12 | Export final state | Admin Workstation | Export accepted domains, connectors, remote domains, rules | Post-rollback evidence is saved |
| 13 | Disconnect session | Admin Workstation | `Disconnect-ExchangeOnline -Confirm:$false` | Session closes |

# Configure_Mail_Flow_Accepted_Domains_Connectors_And_Rules_Rollback_Helper

```powershell
# Run on the admin workstation.
# Purpose: rollback lab-created mail flow objects only.
# Do not run this against production accepted domains, connectors, or rules.

$EvidenceRoot = "C:\M365-Exchange-Baseline"
$RunStamp = Get-Date -Format "yyyyMMdd-HHmmss"
$EvidencePath = Join-Path $EvidenceRoot "04-Mail-Flow-Rollback-$RunStamp"

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\mail-flow-rollback-transcript.txt"

# Lab identities.
$AcceptedDomain = "subsidiary.contoso.com"
$RemoteDomainName = "Partner Example"
$InboundConnectorName = "Inbound from Partner Example"
$OutboundConnectorName = "Outbound to Partner Example"
$TransportRuleName = "LAB - Prepend EXTERNAL To External Mail"
$AttachmentRuleName = "LAB - Block Executable Attachments"

# Capture before rollback.
Get-AcceptedDomain |
  Select-Object Name,DomainName,DomainType,Default |
  Export-Csv "$EvidencePath\accepted-domains-before-rollback.csv" -NoTypeInformation

Get-RemoteDomain |
  Select-Object Name,DomainName,AutoForwardEnabled,AutoReplyEnabled,TNEFEnabled |
  Export-Csv "$EvidencePath\remote-domains-before-rollback.csv" -NoTypeInformation

Get-InboundConnector |
  Select-Object Name,Enabled,ConnectorType,SenderDomains,SenderIPAddresses |
  Export-Csv "$EvidencePath\inbound-connectors-before-rollback.csv" -NoTypeInformation

Get-OutboundConnector |
  Select-Object Name,Enabled,ConnectorType,RecipientDomains,SmartHosts |
  Export-Csv "$EvidencePath\outbound-connectors-before-rollback.csv" -NoTypeInformation

Get-TransportRule |
  Select-Object Name,State,Mode,Priority |
  Export-Csv "$EvidencePath\transport-rules-before-rollback.csv" -NoTypeInformation

# Disable rules first.
Disable-TransportRule -Identity $TransportRuleName -Confirm:$false -ErrorAction SilentlyContinue
Disable-TransportRule -Identity $AttachmentRuleName -Confirm:$false -ErrorAction SilentlyContinue

# Remove lab rules only if approved.
Remove-TransportRule -Identity $TransportRuleName -Confirm:$false -ErrorAction SilentlyContinue
Remove-TransportRule -Identity $AttachmentRuleName -Confirm:$false -ErrorAction SilentlyContinue

# Disable connectors before removal.
Set-InboundConnector -Identity $InboundConnectorName -Enabled $false -ErrorAction SilentlyContinue
Set-OutboundConnector -Identity $OutboundConnectorName -Enabled $false -ErrorAction SilentlyContinue

# Remove lab connectors only if approved.
Remove-InboundConnector -Identity $InboundConnectorName -Confirm:$false -ErrorAction SilentlyContinue
Remove-OutboundConnector -Identity $OutboundConnectorName -Confirm:$false -ErrorAction SilentlyContinue

# Remove lab remote domain only if it was created for this workbook.
Remove-RemoteDomain -Identity $RemoteDomainName -Confirm:$false -ErrorAction SilentlyContinue

# Remove lab accepted domain only if no recipients use it and it is not the default domain.
# Validate first.
Get-Recipient -ResultSize Unlimited |
  Where-Object {$_.PrimarySmtpAddress -like "*@$AcceptedDomain"} |
  Select-Object DisplayName,PrimarySmtpAddress,RecipientTypeDetails |
  Export-Csv "$EvidencePath\recipients-using-lab-domain-before-domain-removal.csv" -NoTypeInformation

$DomainObject = Get-AcceptedDomain -Identity $AcceptedDomain -ErrorAction SilentlyContinue

if ($DomainObject -and $DomainObject.Default -eq $false) {
  $RecipientsUsingDomain = Get-Recipient -ResultSize Unlimited |
    Where-Object {$_.PrimarySmtpAddress -like "*@$AcceptedDomain"}

  if (($RecipientsUsingDomain | Measure-Object).Count -eq 0) {
    Remove-AcceptedDomain -Identity $AcceptedDomain -Confirm:$false -ErrorAction SilentlyContinue
  }
}

# Capture after rollback.
Get-AcceptedDomain |
  Select-Object Name,DomainName,DomainType,Default |
  Export-Csv "$EvidencePath\accepted-domains-after-rollback.csv" -NoTypeInformation

Get-RemoteDomain |
  Select-Object Name,DomainName,AutoForwardEnabled,AutoReplyEnabled,TNEFEnabled |
  Export-Csv "$EvidencePath\remote-domains-after-rollback.csv" -NoTypeInformation

Get-InboundConnector |
  Select-Object Name,Enabled,ConnectorType,SenderDomains,SenderIPAddresses |
  Export-Csv "$EvidencePath\inbound-connectors-after-rollback.csv" -NoTypeInformation

Get-OutboundConnector |
  Select-Object Name,Enabled,ConnectorType,RecipientDomains,SmartHosts |
  Export-Csv "$EvidencePath\outbound-connectors-after-rollback.csv" -NoTypeInformation

Get-TransportRule |
  Select-Object Name,State,Mode,Priority |
  Export-Csv "$EvidencePath\transport-rules-after-rollback.csv" -NoTypeInformation

Stop-Transcript
```

# Configure_Mail_Flow_Accepted_Domains_Connectors_And_Rules_Failure_Checks

| Symptom | Likely Cause | Check Command | Fix |
|---|---|---|---|
| Accepted domain cannot be added | Domain is not verified in Microsoft 365 or already exists | `Get-AcceptedDomain -Identity "<domain>"` | Verify domain in Microsoft 365 admin center or use existing accepted domain |
| Accepted domain removal fails | Domain is default or recipients still use it | `Get-AcceptedDomain -Identity "<domain>"`; `Get-Recipient -ResultSize Unlimited \| Where-Object {$_.EmailAddresses -match "<domain>"}` | Change default domain or remove proxy addresses first |
| Mail for domain bounces | MX record points wrong place or accepted domain type is wrong | `Resolve-DnsName <domain> -Type MX`; `Get-AcceptedDomain <domain>` | Correct DNS and set proper domain type |
| Internal relay domain does not route unresolved recipients | No connector or route exists to downstream mail system | `Get-OutboundConnector`; message trace | Add or fix outbound connector |
| Partner mail rejected inbound | Inbound connector sender IP, sender domain, or TLS does not match | `Get-InboundConnector -Identity "<name>" \| Format-List *` | Correct sender IP, sender domain, TLS, or certificate settings |
| Inbound connector bypasses too broadly | Sender domain or IP scope is too wide | `Get-InboundConnector -Identity "<name>"` | Restrict sender domains and sender IPs |
| Outbound mail to partner fails | Smart host DNS, TLS, or recipient domain mismatch | `Get-OutboundConnector -Identity "<name>"`; `Resolve-DnsName <smart-host>` | Correct smart host, recipient domain, or TLS settings |
| Outbound mail routes unexpectedly | Connector recipient domain pattern too broad | `Get-OutboundConnector \| Format-List Name,RecipientDomains` | Narrow recipient domains |
| TLS failure to partner | Certificate name or TLS domain mismatch | `Get-OutboundConnector -Identity "<name>" \| Format-List TlsSettings,TlsDomain` | Align TLS domain with partner certificate and connector design |
| Mail flow rule does not trigger | Condition does not match or rule priority overridden by earlier rule | `Get-TransportRule -Identity "<rule>" \| Format-List *` | Fix condition or rule priority |
| Mail flow rule affects too many messages | Condition is too broad or missing exceptions | `Get-TransportRule -Identity "<rule>" \| Format-List Conditions,Exceptions` | Add exceptions and test before enforcing |
| Mail flow rule not enforcing | Rule is disabled or in test mode | `Get-TransportRule -Identity "<rule>" \| Format-List State,Mode` | Enable rule and set `Mode Enforce` after validation |
| Rule priority wrong | Rule placed after conflicting rule | `Get-TransportRule \| Sort Priority \| Select Priority,Name` | Move priority with `Set-TransportRule -Priority` |
| Message trace does not show test message | Delay, wrong sender, wrong recipient, or outside trace window | `Get-MessageTrace -StartDate (Get-Date).AddHours(-24) -EndDate (Get-Date)` | Widen time window and confirm sender or recipient |
| EAC shows stale data | Portal cache or propagation delay | Validate with PowerShell | Refresh session or wait |
| PowerShell command not found | Missing ExchangeOnlineManagement connection or RBAC | `Get-ConnectionInformation`; `Get-Command <cmdlet>` | Reconnect and confirm Exchange admin role |
| Connector command fails due to permissions | Admin role lacks mail flow permissions | `Get-ManagementRoleAssignment -RoleAssignee "<admin-upn>"` | Assign correct Exchange role group |
| DNS records look correct but mail still fails | External DNS propagation, SPF fail, connector mismatch, or protection policy | DNS lookup from external network and message trace | Validate public DNS, message trace, and connector settings |
| Production mail outage after connector change | Connector scope or route captured production mail unexpectedly | Disable connector immediately | Disable connector, validate message trace, restore previous connector settings |

# Configure_Mail_Flow_Accepted_Domains_Connectors_And_Rules_Related_Labs

| Related Lab                                                                                    | Relationship                                                                                      |
| ---------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------- |
| `01_Configure_Exchange_Admin_Center_PowerShell_And_Org_Baseline.md`                            | Provides Exchange admin center access, Exchange Online PowerShell, and initial transport baseline |
| `02_Manage_User_Mailboxes_Shared_Mailboxes_Contacts_And_Resources.md`                          | Provides mailbox and contact recipients used for mail flow testing                                |
| `03_Manage_Distribution_Groups_Dynamic_Groups_Mail_Enabled_Security_Groups_And_M365_Groups.md` | Provides groups used as transport rule targets, exceptions, and test recipients                   |
| `05_Configure_Mailbox_Permissions_Delegation_Forwarding_And_Auditing.md`                       | Depends on mail routing and forwarding behavior                                                   |
| `06_Configure_Exchange_Online_Protection_AntiSpam_AntiMalware_And_Quarantine.md`               | Builds on mail flow paths, connectors, and routing behavior                                       |
| `07_Configure_Retention_Archive_Litigation_Hold_And_eDiscovery_Handoff.md`                     | Depends on mailbox and group mail flow visibility                                                 |
| `08_Troubleshoot_Mail_Flow_Mailbox_Access_Permissions_And_Quarantine_Issues.md`                | Uses accepted domain, connector, transport rule, DNS, and message trace evidence                  |
