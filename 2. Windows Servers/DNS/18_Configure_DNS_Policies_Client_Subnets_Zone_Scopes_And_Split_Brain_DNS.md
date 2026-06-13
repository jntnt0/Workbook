18_Configure_DNS_Policies_Client_Subnets_Zone_Scopes_And_Split_Brain_DNS.md
# Configure_DNS_Policies_Client_Subnets_Zone_Scopes_And_Split_Brain_DNS

# Configure_DNS_Policies_Client_Subnets_Zone_Scopes_And_Split_Brain_DNS_Index
18_Configure_DNS_Policies_Client_Subnets_Zone_Scopes_And_Split_Brain_DNS.md
Configure_DNS_Policies_Client_Subnets_Zone_Scopes_And_Split_Brain_DNS
Configure_DNS_Policies_Client_Subnets_Zone_Scopes_And_Split_Brain_DNS_Source_Basis
Configure_DNS_Policies_Client_Subnets_Zone_Scopes_And_Split_Brain_DNS_Mental_Model
Configure_DNS_Policies_Client_Subnets_Zone_Scopes_And_Split_Brain_DNS_Planning_Table
Configure_DNS_Policies_Client_Subnets_Zone_Scopes_And_Split_Brain_DNS_Configuration_Checklist
Configure_DNS_Policies_Client_Subnets_Zone_Scopes_And_Split_Brain_DNS_Precheck_Skeleton
Configure_DNS_Policies_Client_Subnets_Zone_Scopes_And_Split_Brain_DNS_Client_Subnet_Skeleton
Configure_DNS_Policies_Client_Subnets_Zone_Scopes_And_Split_Brain_DNS_Zone_Scope_Skeleton
Configure_DNS_Policies_Client_Subnets_Zone_Scopes_And_Split_Brain_DNS_Split_Brain_Record_Skeleton
Configure_DNS_Policies_Client_Subnets_Zone_Scopes_And_Split_Brain_DNS_Query_Resolution_Policy_Skeleton
Configure_DNS_Policies_Client_Subnets_Zone_Scopes_And_Split_Brain_DNS_Geo_Or_Site_Based_Response_Skeleton
Configure_DNS_Policies_Client_Subnets_Zone_Scopes_And_Split_Brain_DNS_Block_Or_Filter_Response_Skeleton
Configure_DNS_Policies_Client_Subnets_Zone_Scopes_And_Split_Brain_DNS_Validation_Skeleton
Configure_DNS_Policies_Client_Subnets_Zone_Scopes_And_Split_Brain_DNS_Event_Log_Skeleton
Configure_DNS_Policies_Client_Subnets_Zone_Scopes_And_Split_Brain_DNS_Verification_Commands
Configure_DNS_Policies_Client_Subnets_Zone_Scopes_And_Split_Brain_DNS_Rollback
Configure_DNS_Policies_Client_Subnets_Zone_Scopes_And_Split_Brain_DNS_Failure_Checks
Configure_DNS_Policies_Client_Subnets_Zone_Scopes_And_Split_Brain_DNS_Related_Labs

# Configure_DNS_Policies_Client_Subnets_Zone_Scopes_And_Split_Brain_DNS_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | DNS Server PowerShell module | Managing DNS policies, client subnets, zone scopes, records, and query responses |
| Microsoft Learn | Add-DnsServerClientSubnet | Defining client subnet objects for policy matching |
| Microsoft Learn | Get-DnsServerClientSubnet | Reviewing configured DNS client subnets |
| Microsoft Learn | Add-DnsServerZoneScope | Creating additional record containers inside the same DNS zone |
| Microsoft Learn | Get-DnsServerZoneScope | Reviewing zone scopes for a DNS zone |
| Microsoft Learn | Add-DnsServerResourceRecordA | Creating records inside specific zone scopes |
| Microsoft Learn | Add-DnsServerQueryResolutionPolicy | Creating DNS policies that choose response behavior |
| Microsoft Learn | Get-DnsServerQueryResolutionPolicy | Reviewing policy order and match criteria |
| Microsoft Learn | Remove-DnsServerQueryResolutionPolicy | Removing incorrect DNS policies |
| Microsoft Learn | Remove-DnsServerZoneScope | Removing incorrect zone scopes |
| Microsoft Learn | Remove-DnsServerClientSubnet | Removing incorrect client subnet definitions |
| Microsoft Learn | Resolve-DnsName | Testing DNS policy response behavior from client or admin hosts |
| Windows Server operational practice | Split-brain DNS, site-based answers, policy-based blocking, controlled DNS responses | Returning different DNS answers based on client subnet, zone scope, or policy rules |

# Configure_DNS_Policies_Client_Subnets_Zone_Scopes_And_Split_Brain_DNS_Mental_Model
| Concept | Operational Meaning |
|---|---|
| DNS policy | Rule that changes DNS server behavior based on conditions such as client subnet, zone, FQDN, query type, or transport |
| Client subnet | Named network range used as a condition in DNS query resolution policies |
| Zone scope | Alternate record set inside the same DNS zone |
| Default zone scope | Normal DNS zone data used when no special policy redirects the query |
| Split-brain DNS | Same DNS name returns different answers depending on whether the client is internal, external, branch, VPN, or site-based |
| Site-based response | DNS answer selected based on the subnet or location of the querying client |
| Policy processing order | Lower processing order rules are evaluated before later rules |
| FQDN filter | Policy condition that matches a specific DNS name or pattern |
| Query type filter | Policy condition that applies only to A, AAAA, SRV, TXT, or other query types |
| ALLOW action | Policy permits a query and can direct it to a selected zone scope |
| DENY action | Policy blocks a matching query |
| IGNORE action | Policy drops a matching query without response |
| Zone scope weight | Value used to distribute answers across multiple scopes when multiple scopes are listed |
| Recursion policy | DNS policy controlling recursive behavior, separate from authoritative zone response design |
| First rule | Build DNS policies like firewall rules: define match conditions, processing order, expected answer, validation source, and rollback before enabling |

# Configure_DNS_Policies_Client_Subnets_Zone_Scopes_And_Split_Brain_DNS_Planning_Table
| Item | Example | Decision |
|---|---|---|
| DNS server | `DC1.corp.local` | `<dns-server-fqdn>` |
| DNS server IP | `10.10.10.10` | `<dns-server-ip>` |
| Zone name | `corp.local` | `<zone-name>` |
| Split-brain FQDN | `app1.corp.local` | `<split-brain-fqdn>` |
| Record label | `app1` | `<record-label>` |
| Internal client subnet name | `Internal-Clients` | `<internal-client-subnet-name>` |
| Internal client subnet | `10.10.0.0/16` | `<internal-client-subnet>` |
| Branch client subnet name | `Branch-Clients` | `<branch-client-subnet-name>` |
| Branch client subnet | `10.20.0.0/16` | `<branch-client-subnet>` |
| VPN client subnet name | `VPN-Clients` | `<vpn-client-subnet-name>` |
| VPN client subnet | `10.30.0.0/16` | `<vpn-client-subnet>` |
| Internal zone scope | `InternalScope` | `<internal-zone-scope>` |
| Branch zone scope | `BranchScope` | `<branch-zone-scope>` |
| Default scope | Existing zone records | `<default-zone-scope>` |
| Internal app IP | `10.10.10.50` | `<internal-app-ip>` |
| Branch app IP | `10.20.10.50` | `<branch-app-ip>` |
| External/public app IP | `203.0.113.50` | `<external-app-ip>` |
| Policy name internal | `Policy-App1-Internal` | `<internal-policy-name>` |
| Policy name branch | `Policy-App1-Branch` | `<branch-policy-name>` |
| Policy processing order | Internal 1, Branch 2, Default normal | `<processing-order>` |
| Blocked name example | `badhost.corp.local` | `<blocked-name>` |
| Evidence path | `C:\DNS-Policies-SplitBrain` | `<evidence-path>` |
| Rollback stance | Remove policies first, then zone-scope records, then zone scopes/client subnet objects | `<rollback-plan>` |
| Next workbook | `19_Troubleshoot_DNS_Policies_And_Split_Brain_Resolution.md` | `<next-task>` |

# Configure_DNS_Policies_Client_Subnets_Zone_Scopes_And_Split_Brain_DNS_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Open elevated PowerShell | DNS Server | `whoami /groups` | Admin context is visible |
| 2 | Confirm DNS role installed | DNS Server | `Get-WindowsFeature DNS` | DNS role is installed |
| 3 | Confirm DNS service running | DNS Server | `Get-Service DNS` | DNS service is Running |
| 4 | Import DNS module | DNS Server | `Import-Module DnsServer` | DNS cmdlets are available |
| 5 | Confirm target zone exists | DNS Server | `Get-DnsServerZone -Name "<zone-name>"` | Zone exists |
| 6 | Export current zone records | DNS Server | `Get-DnsServerResourceRecord -ZoneName "<zone-name>"` | Rollback inventory exists |
| 7 | Export current DNS policies | DNS Server | `Get-DnsServerQueryResolutionPolicy` | Existing policy state is documented |
| 8 | Export current client subnets | DNS Server | `Get-DnsServerClientSubnet` | Existing subnet objects are documented |
| 9 | Export current zone scopes | DNS Server | `Get-DnsServerZoneScope -ZoneName "<zone-name>"` | Existing scopes are documented |
| 10 | Create internal client subnet | DNS Server | `Add-DnsServerClientSubnet -Name "<internal-subnet-name>" -IPv4Subnet "<internal-subnet>"` | Internal client subnet object exists |
| 11 | Create branch client subnet | DNS Server | `Add-DnsServerClientSubnet -Name "<branch-subnet-name>" -IPv4Subnet "<branch-subnet>"` | Branch client subnet object exists |
| 12 | Create internal zone scope | DNS Server | `Add-DnsServerZoneScope -ZoneName "<zone-name>" -Name "<internal-scope>"` | Internal zone scope exists |
| 13 | Create branch zone scope | DNS Server | `Add-DnsServerZoneScope -ZoneName "<zone-name>" -Name "<branch-scope>"` | Branch zone scope exists |
| 14 | Add default record if needed | DNS Server | `Add-DnsServerResourceRecordA -ZoneName "<zone-name>" -Name "<record-label>" -IPv4Address "<default-ip>"` | Default response exists |
| 15 | Add internal scoped record | DNS Server | `Add-DnsServerResourceRecordA -ZoneName "<zone-name>" -ZoneScope "<internal-scope>" -Name "<record-label>" -IPv4Address "<internal-ip>"` | Internal scope response exists |
| 16 | Add branch scoped record | DNS Server | `Add-DnsServerResourceRecordA -ZoneName "<zone-name>" -ZoneScope "<branch-scope>" -Name "<record-label>" -IPv4Address "<branch-ip>"` | Branch scope response exists |
| 17 | Create internal query policy | DNS Server | `Add-DnsServerQueryResolutionPolicy -Name "<policy-name>" -Action ALLOW -ClientSubnet "eq,<internal-subnet-name>" -Fqdn "eq,<fqdn>" -ZoneScope "<internal-scope>,1" -ZoneName "<zone-name>" -ProcessingOrder 1` | Internal clients receive internal scoped answer |
| 18 | Create branch query policy | DNS Server | `Add-DnsServerQueryResolutionPolicy -Name "<policy-name>" -Action ALLOW -ClientSubnet "eq,<branch-subnet-name>" -Fqdn "eq,<fqdn>" -ZoneScope "<branch-scope>,1" -ZoneName "<zone-name>" -ProcessingOrder 2` | Branch clients receive branch scoped answer |
| 19 | Validate policy inventory | DNS Server | `Get-DnsServerQueryResolutionPolicy -ZoneName "<zone-name>"` | Policies are visible in expected order |
| 20 | Validate scoped records | DNS Server | `Get-DnsServerResourceRecord -ZoneName "<zone-name>" -ZoneScope "<scope-name>"` | Scoped records are visible |
| 21 | Test resolution from internal client | Internal Client | `Resolve-DnsName "<fqdn>" -Server "<dns-server-ip>"` | Internal IP is returned |
| 22 | Test resolution from branch client | Branch Client | `Resolve-DnsName "<fqdn>" -Server "<dns-server-ip>"` | Branch IP is returned |
| 23 | Test resolution from unmatched client | Admin Host / Other Client | `Resolve-DnsName "<fqdn>" -Server "<dns-server-ip>"` | Default zone answer is returned |
| 24 | Review DNS Server events | DNS Server | `Get-WinEvent -LogName "DNS Server" -MaxEvents 100` | DNS events are captured |
| 25 | Export final evidence | DNS Server / Clients | Run validation skeleton | Evidence files are saved |
| 26 | Document final design | Operator | `Record client subnets, scopes, records, policies, processing order, and test results` | Split-brain DNS build record is complete |

# Configure_DNS_Policies_Client_Subnets_Zone_Scopes_And_Split_Brain_DNS_Precheck_Skeleton
```powershell
# Run in elevated PowerShell on the DNS server.
# Purpose: capture baseline DNS zone, policy, client subnet, zone scope, record, and event state before changes.

$ZoneName = "corp.local"
$SplitBrainFqdn = "app1.corp.local"
$RecordLabel = "app1"
$DnsServerIp = "10.10.10.10"
$EvidencePath = "C:\DNS-Policies-SplitBrain"

New-Item -ItemType Directory -Force -Path $EvidencePath

# Confirm administrator context.
whoami /groups |
  Tee-Object "$EvidencePath\whoami-groups.txt"

# Capture server identity.
hostname |
  Tee-Object "$EvidencePath\hostname.txt"

Get-ComputerInfo |
  Select-Object CsName,CsDomain,CsPartOfDomain,OsName,OsVersion |
  Tee-Object "$EvidencePath\computer-info.txt"

# Confirm DNS role and service.
Get-WindowsFeature DNS |
  Tee-Object "$EvidencePath\dns-role-state.txt"

Get-Service DNS |
  Tee-Object "$EvidencePath\dns-service-state.txt"

# Capture network state.
Get-NetIPConfiguration |
  Tee-Object "$EvidencePath\net-ip-configuration.txt"

ipconfig /all |
  Tee-Object "$EvidencePath\ipconfig-all.txt"

# Import DNS module.
Import-Module DnsServer

# Capture zone inventory.
Get-DnsServerZone |
  Tee-Object "$EvidencePath\dns-zones-before.txt"

Get-DnsServerZone -Name $ZoneName |
  Select-Object ZoneName,ZoneType,IsDsIntegrated,ReplicationScope,DynamicUpdate,IsPaused,IsShutdown |
  Tee-Object "$EvidencePath\target-zone-before.txt"

# Capture records.
Get-DnsServerResourceRecord -ZoneName $ZoneName |
  Tee-Object "$EvidencePath\zone-records-before.txt"

Get-DnsServerResourceRecord -ZoneName $ZoneName |
  Export-Csv "$EvidencePath\zone-records-before.csv" -NoTypeInformation

# Capture current app record.
Get-DnsServerResourceRecord -ZoneName $ZoneName -Name $RecordLabel -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\split-brain-record-before.txt"

# Capture DNS policy objects.
Get-DnsServerClientSubnet -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dns-client-subnets-before.txt"

Get-DnsServerZoneScope -ZoneName $ZoneName -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dns-zone-scopes-before.txt"

Get-DnsServerQueryResolutionPolicy -ZoneName $ZoneName -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dns-query-resolution-policies-before.txt"

# Baseline query test before policy.
Resolve-DnsName $SplitBrainFqdn -Server $DnsServerIp -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\resolve-split-brain-name-before-policy.txt"

# Capture DNS events.
Get-WinEvent -LogName "DNS Server" -MaxEvents 100 -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dns-server-events-before.txt"
```

# Configure_DNS_Policies_Client_Subnets_Zone_Scopes_And_Split_Brain_DNS_Client_Subnet_Skeleton
```powershell
# Run in elevated PowerShell on the DNS server.
# Purpose: define named client subnet objects for DNS policy matching.

$InternalSubnetName = "Internal-Clients"
$InternalSubnet = "10.10.0.0/16"
$BranchSubnetName = "Branch-Clients"
$BranchSubnet = "10.20.0.0/16"
$VpnSubnetName = "VPN-Clients"
$VpnSubnet = "10.30.0.0/16"
$EvidencePath = "C:\DNS-Policies-SplitBrain"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\client-subnet-config-transcript.txt"

Import-Module DnsServer

# Capture existing client subnets.
Get-DnsServerClientSubnet -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\client-subnets-before.txt"

# Create internal client subnet if missing.
if (-not (Get-DnsServerClientSubnet -Name $InternalSubnetName -ErrorAction SilentlyContinue)) {
  Add-DnsServerClientSubnet `
    -Name $InternalSubnetName `
    -IPv4Subnet $InternalSubnet
}

# Create branch client subnet if missing.
if (-not (Get-DnsServerClientSubnet -Name $BranchSubnetName -ErrorAction SilentlyContinue)) {
  Add-DnsServerClientSubnet `
    -Name $BranchSubnetName `
    -IPv4Subnet $BranchSubnet
}

# Create VPN client subnet if missing.
if (-not (Get-DnsServerClientSubnet -Name $VpnSubnetName -ErrorAction SilentlyContinue)) {
  Add-DnsServerClientSubnet `
    -Name $VpnSubnetName `
    -IPv4Subnet $VpnSubnet
}

# Confirm final client subnets.
Get-DnsServerClientSubnet |
  Tee-Object "$EvidencePath\client-subnets-after.txt"

Stop-Transcript
```

# Configure_DNS_Policies_Client_Subnets_Zone_Scopes_And_Split_Brain_DNS_Zone_Scope_Skeleton
```powershell
# Run in elevated PowerShell on the DNS server.
# Purpose: create zone scopes that hold alternate answers for the same zone.

$ZoneName = "corp.local"
$InternalScope = "InternalScope"
$BranchScope = "BranchScope"
$VpnScope = "VPNScope"
$EvidencePath = "C:\DNS-Policies-SplitBrain"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\zone-scope-config-transcript.txt"

Import-Module DnsServer

# Confirm target zone.
Get-DnsServerZone -Name $ZoneName |
  Tee-Object "$EvidencePath\target-zone-before-zone-scopes.txt"

# Capture existing scopes.
Get-DnsServerZoneScope -ZoneName $ZoneName -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\zone-scopes-before.txt"

# Create internal scope if missing.
if (-not (Get-DnsServerZoneScope -ZoneName $ZoneName -Name $InternalScope -ErrorAction SilentlyContinue)) {
  Add-DnsServerZoneScope `
    -ZoneName $ZoneName `
    -Name $InternalScope
}

# Create branch scope if missing.
if (-not (Get-DnsServerZoneScope -ZoneName $ZoneName -Name $BranchScope -ErrorAction SilentlyContinue)) {
  Add-DnsServerZoneScope `
    -ZoneName $ZoneName `
    -Name $BranchScope
}

# Create VPN scope if missing.
if (-not (Get-DnsServerZoneScope -ZoneName $ZoneName -Name $VpnScope -ErrorAction SilentlyContinue)) {
  Add-DnsServerZoneScope `
    -ZoneName $ZoneName `
    -Name $VpnScope
}

# Confirm final scopes.
Get-DnsServerZoneScope -ZoneName $ZoneName |
  Tee-Object "$EvidencePath\zone-scopes-after.txt"

Stop-Transcript
```

# Configure_DNS_Policies_Client_Subnets_Zone_Scopes_And_Split_Brain_DNS_Split_Brain_Record_Skeleton
```powershell
# Run in elevated PowerShell on the DNS server.
# Purpose: place different A records for the same name into different zone scopes.

$ZoneName = "corp.local"
$RecordLabel = "app1"
$DefaultIp = "203.0.113.50"
$InternalScope = "InternalScope"
$InternalIp = "10.10.10.50"
$BranchScope = "BranchScope"
$BranchIp = "10.20.10.50"
$VpnScope = "VPNScope"
$VpnIp = "10.30.10.50"
$EvidencePath = "C:\DNS-Policies-SplitBrain"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\split-brain-record-config-transcript.txt"

Import-Module DnsServer

# Capture current default-scope record.
Get-DnsServerResourceRecord -ZoneName $ZoneName -Name $RecordLabel -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\default-scope-record-before.txt"

# Add default record if missing.
if (-not (Get-DnsServerResourceRecord -ZoneName $ZoneName -Name $RecordLabel -RRType A -ErrorAction SilentlyContinue)) {
  Add-DnsServerResourceRecordA `
    -ZoneName $ZoneName `
    -Name $RecordLabel `
    -IPv4Address $DefaultIp `
    -TimeToLive 00:05:00
}

# Add internal scoped record.
if (-not (Get-DnsServerResourceRecord -ZoneName $ZoneName -ZoneScope $InternalScope -Name $RecordLabel -RRType A -ErrorAction SilentlyContinue)) {
  Add-DnsServerResourceRecordA `
    -ZoneName $ZoneName `
    -ZoneScope $InternalScope `
    -Name $RecordLabel `
    -IPv4Address $InternalIp `
    -TimeToLive 00:05:00
}

# Add branch scoped record.
if (-not (Get-DnsServerResourceRecord -ZoneName $ZoneName -ZoneScope $BranchScope -Name $RecordLabel -RRType A -ErrorAction SilentlyContinue)) {
  Add-DnsServerResourceRecordA `
    -ZoneName $ZoneName `
    -ZoneScope $BranchScope `
    -Name $RecordLabel `
    -IPv4Address $BranchIp `
    -TimeToLive 00:05:00
}

# Add VPN scoped record.
if (-not (Get-DnsServerResourceRecord -ZoneName $ZoneName -ZoneScope $VpnScope -Name $RecordLabel -RRType A -ErrorAction SilentlyContinue)) {
  Add-DnsServerResourceRecordA `
    -ZoneName $ZoneName `
    -ZoneScope $VpnScope `
    -Name $RecordLabel `
    -IPv4Address $VpnIp `
    -TimeToLive 00:05:00
}

# Confirm default and scoped records.
Get-DnsServerResourceRecord -ZoneName $ZoneName -Name $RecordLabel |
  Tee-Object "$EvidencePath\default-scope-record-after.txt"

Get-DnsServerResourceRecord -ZoneName $ZoneName -ZoneScope $InternalScope -Name $RecordLabel |
  Tee-Object "$EvidencePath\internal-scope-record-after.txt"

Get-DnsServerResourceRecord -ZoneName $ZoneName -ZoneScope $BranchScope -Name $RecordLabel |
  Tee-Object "$EvidencePath\branch-scope-record-after.txt"

Get-DnsServerResourceRecord -ZoneName $ZoneName -ZoneScope $VpnScope -Name $RecordLabel |
  Tee-Object "$EvidencePath\vpn-scope-record-after.txt"

Stop-Transcript
```

# Configure_DNS_Policies_Client_Subnets_Zone_Scopes_And_Split_Brain_DNS_Query_Resolution_Policy_Skeleton
```powershell
# Run in elevated PowerShell on the DNS server.
# Purpose: create query resolution policies that return different zone scopes based on client subnet.

$ZoneName = "corp.local"
$SplitBrainFqdn = "app1.corp.local"
$InternalSubnetName = "Internal-Clients"
$BranchSubnetName = "Branch-Clients"
$VpnSubnetName = "VPN-Clients"
$InternalScope = "InternalScope"
$BranchScope = "BranchScope"
$VpnScope = "VPNScope"
$EvidencePath = "C:\DNS-Policies-SplitBrain"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\query-resolution-policy-config-transcript.txt"

Import-Module DnsServer

# Capture current policies.
Get-DnsServerQueryResolutionPolicy -ZoneName $ZoneName -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\query-policies-before.txt"

# Create policy for internal clients.
if (-not (Get-DnsServerQueryResolutionPolicy -ZoneName $ZoneName -Name "Policy-App1-Internal" -ErrorAction SilentlyContinue)) {
  Add-DnsServerQueryResolutionPolicy `
    -Name "Policy-App1-Internal" `
    -Action ALLOW `
    -ClientSubnet "eq,$InternalSubnetName" `
    -Fqdn "eq,$SplitBrainFqdn" `
    -ZoneScope "$InternalScope,1" `
    -ZoneName $ZoneName `
    -ProcessingOrder 1
}

# Create policy for branch clients.
if (-not (Get-DnsServerQueryResolutionPolicy -ZoneName $ZoneName -Name "Policy-App1-Branch" -ErrorAction SilentlyContinue)) {
  Add-DnsServerQueryResolutionPolicy `
    -Name "Policy-App1-Branch" `
    -Action ALLOW `
    -ClientSubnet "eq,$BranchSubnetName" `
    -Fqdn "eq,$SplitBrainFqdn" `
    -ZoneScope "$BranchScope,1" `
    -ZoneName $ZoneName `
    -ProcessingOrder 2
}

# Create policy for VPN clients.
if (-not (Get-DnsServerQueryResolutionPolicy -ZoneName $ZoneName -Name "Policy-App1-VPN" -ErrorAction SilentlyContinue)) {
  Add-DnsServerQueryResolutionPolicy `
    -Name "Policy-App1-VPN" `
    -Action ALLOW `
    -ClientSubnet "eq,$VpnSubnetName" `
    -Fqdn "eq,$SplitBrainFqdn" `
    -ZoneScope "$VpnScope,1" `
    -ZoneName $ZoneName `
    -ProcessingOrder 3
}

# Confirm policies.
Get-DnsServerQueryResolutionPolicy -ZoneName $ZoneName |
  Tee-Object "$EvidencePath\query-policies-after.txt"

Stop-Transcript
```

# Configure_DNS_Policies_Client_Subnets_Zone_Scopes_And_Split_Brain_DNS_Geo_Or_Site_Based_Response_Skeleton
```powershell
# Run in elevated PowerShell on the DNS server.
# Purpose: create a simple site-based DNS answer pattern using client subnets and zone scopes.

$ZoneName = "corp.local"
$RecordLabel = "files"
$Fqdn = "files.corp.local"
$SiteASubnetName = "SiteA-Clients"
$SiteASubnet = "10.40.0.0/16"
$SiteBSSubnetName = "SiteB-Clients"
$SiteBSubnet = "10.50.0.0/16"
$SiteAScope = "SiteAScope"
$SiteBScope = "SiteBScope"
$SiteAFileServerIp = "10.40.10.20"
$SiteBFileServerIp = "10.50.10.20"
$EvidencePath = "C:\DNS-Policies-SplitBrain"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\site-based-response-config-transcript.txt"

Import-Module DnsServer

# Create site client subnets.
if (-not (Get-DnsServerClientSubnet -Name $SiteASubnetName -ErrorAction SilentlyContinue)) {
  Add-DnsServerClientSubnet -Name $SiteASubnetName -IPv4Subnet $SiteASubnet
}

if (-not (Get-DnsServerClientSubnet -Name $SiteBSSubnetName -ErrorAction SilentlyContinue)) {
  Add-DnsServerClientSubnet -Name $SiteBSSubnetName -IPv4Subnet $SiteBSubnet
}

# Create site scopes.
if (-not (Get-DnsServerZoneScope -ZoneName $ZoneName -Name $SiteAScope -ErrorAction SilentlyContinue)) {
  Add-DnsServerZoneScope -ZoneName $ZoneName -Name $SiteAScope
}

if (-not (Get-DnsServerZoneScope -ZoneName $ZoneName -Name $SiteBScope -ErrorAction SilentlyContinue)) {
  Add-DnsServerZoneScope -ZoneName $ZoneName -Name $SiteBScope
}

# Add site-specific records.
if (-not (Get-DnsServerResourceRecord -ZoneName $ZoneName -ZoneScope $SiteAScope -Name $RecordLabel -RRType A -ErrorAction SilentlyContinue)) {
  Add-DnsServerResourceRecordA `
    -ZoneName $ZoneName `
    -ZoneScope $SiteAScope `
    -Name $RecordLabel `
    -IPv4Address $SiteAFileServerIp `
    -TimeToLive 00:05:00
}

if (-not (Get-DnsServerResourceRecord -ZoneName $ZoneName -ZoneScope $SiteBScope -Name $RecordLabel -RRType A -ErrorAction SilentlyContinue)) {
  Add-DnsServerResourceRecordA `
    -ZoneName $ZoneName `
    -ZoneScope $SiteBScope `
    -Name $RecordLabel `
    -IPv4Address $SiteBFileServerIp `
    -TimeToLive 00:05:00
}

# Create site policies.
if (-not (Get-DnsServerQueryResolutionPolicy -ZoneName $ZoneName -Name "Policy-Files-SiteA" -ErrorAction SilentlyContinue)) {
  Add-DnsServerQueryResolutionPolicy `
    -Name "Policy-Files-SiteA" `
    -Action ALLOW `
    -ClientSubnet "eq,$SiteASubnetName" `
    -Fqdn "eq,$Fqdn" `
    -ZoneScope "$SiteAScope,1" `
    -ZoneName $ZoneName `
    -ProcessingOrder 10
}

if (-not (Get-DnsServerQueryResolutionPolicy -ZoneName $ZoneName -Name "Policy-Files-SiteB" -ErrorAction SilentlyContinue)) {
  Add-DnsServerQueryResolutionPolicy `
    -Name "Policy-Files-SiteB" `
    -Action ALLOW `
    -ClientSubnet "eq,$SiteBSSubnetName" `
    -Fqdn "eq,$Fqdn" `
    -ZoneScope "$SiteBScope,1" `
    -ZoneName $ZoneName `
    -ProcessingOrder 11
}

# Confirm site-based policy set.
Get-DnsServerClientSubnet |
  Tee-Object "$EvidencePath\site-client-subnets-after.txt"

Get-DnsServerZoneScope -ZoneName $ZoneName |
  Tee-Object "$EvidencePath\site-zone-scopes-after.txt"

Get-DnsServerQueryResolutionPolicy -ZoneName $ZoneName |
  Tee-Object "$EvidencePath\site-query-policies-after.txt"

Stop-Transcript
```

# Configure_DNS_Policies_Client_Subnets_Zone_Scopes_And_Split_Brain_DNS_Block_Or_Filter_Response_Skeleton
```powershell
# Run in elevated PowerShell on the DNS server.
# Purpose: create a DNS policy to block or ignore selected queries.
# Use carefully. Blocking internal names can break domain and application behavior.

$ZoneName = "corp.local"
$BlockedFqdn = "badhost.corp.local"
$BlockedSubnetName = "Internal-Clients"
$EvidencePath = "C:\DNS-Policies-SplitBrain"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\block-filter-policy-config-transcript.txt"

Import-Module DnsServer

# Capture current policies.
Get-DnsServerQueryResolutionPolicy -ZoneName $ZoneName -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\query-policies-before-block-filter.txt"

# Example DENY policy for a specific name from a specific client subnet.
if (-not (Get-DnsServerQueryResolutionPolicy -ZoneName $ZoneName -Name "Policy-Block-BadHost" -ErrorAction SilentlyContinue)) {
  Add-DnsServerQueryResolutionPolicy `
    -Name "Policy-Block-BadHost" `
    -Action DENY `
    -ClientSubnet "eq,$BlockedSubnetName" `
    -Fqdn "eq,$BlockedFqdn" `
    -ZoneName $ZoneName `
    -ProcessingOrder 50
}

# Example IGNORE policy pattern, disabled by default in notes.
# IGNORE drops the query and can look like timeout behavior to clients.
# Add-DnsServerQueryResolutionPolicy `
#   -Name "Policy-Ignore-BadHost" `
#   -Action IGNORE `
#   -ClientSubnet "eq,$BlockedSubnetName" `
#   -Fqdn "eq,$BlockedFqdn" `
#   -ZoneName $ZoneName `
#   -ProcessingOrder 51

# Confirm policies.
Get-DnsServerQueryResolutionPolicy -ZoneName $ZoneName |
  Tee-Object "$EvidencePath\query-policies-after-block-filter.txt"

Stop-Transcript
```

# Configure_DNS_Policies_Client_Subnets_Zone_Scopes_And_Split_Brain_DNS_Validation_Skeleton
```powershell
# Run from DNS server and from clients in each matching subnet.
# Purpose: prove policy-based DNS responses return the expected IP for each source network.

$DnsServerIp = "10.10.10.10"
$ZoneName = "corp.local"
$SplitBrainFqdn = "app1.corp.local"
$RecordLabel = "app1"
$InternalScope = "InternalScope"
$BranchScope = "BranchScope"
$VpnScope = "VPNScope"
$EvidencePath = "C:\DNS-Policies-SplitBrain\Validation"

New-Item -ItemType Directory -Force -Path $EvidencePath

Import-Module DnsServer

# Server-side policy inventory.
Get-DnsServerClientSubnet |
  Tee-Object "$EvidencePath\client-subnets.txt"

Get-DnsServerZoneScope -ZoneName $ZoneName |
  Tee-Object "$EvidencePath\zone-scopes.txt"

Get-DnsServerQueryResolutionPolicy -ZoneName $ZoneName |
  Tee-Object "$EvidencePath\query-resolution-policies.txt"

# Server-side scoped record inventory.
Get-DnsServerResourceRecord -ZoneName $ZoneName -Name $RecordLabel -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\default-scope-record.txt"

Get-DnsServerResourceRecord -ZoneName $ZoneName -ZoneScope $InternalScope -Name $RecordLabel -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\internal-scope-record.txt"

Get-DnsServerResourceRecord -ZoneName $ZoneName -ZoneScope $BranchScope -Name $RecordLabel -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\branch-scope-record.txt"

Get-DnsServerResourceRecord -ZoneName $ZoneName -ZoneScope $VpnScope -Name $RecordLabel -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\vpn-scope-record.txt"

# Query from current host perspective.
# For true policy validation, run this command from a host inside each defined client subnet.
Resolve-DnsName $SplitBrainFqdn -Server $DnsServerIp |
  Tee-Object "$EvidencePath\resolve-split-brain-fqdn-from-current-host.txt"

# Capture local source client details for evidence.
hostname |
  Tee-Object "$EvidencePath\validation-client-hostname.txt"

Get-NetIPConfiguration |
  Tee-Object "$EvidencePath\validation-client-ip-config.txt"

ipconfig /all |
  Tee-Object "$EvidencePath\validation-client-ipconfig-all.txt"

# DNS port reachability.
Test-NetConnection $DnsServerIp -Port 53 |
  Tee-Object "$EvidencePath\test-dns-server-tcp-53.txt"

# Optional repeated query test.
1..5 | ForEach-Object {
  Resolve-DnsName $SplitBrainFqdn -Server $DnsServerIp -ErrorAction SilentlyContinue
} |
  Tee-Object "$EvidencePath\repeated-query-test.txt"
```

# Configure_DNS_Policies_Client_Subnets_Zone_Scopes_And_Split_Brain_DNS_Event_Log_Skeleton
```powershell
# Run in elevated PowerShell on the DNS server.
# Purpose: collect DNS events after policy, subnet, scope, and split-brain DNS configuration.

$EvidencePath = "C:\DNS-Policies-SplitBrain\Events"
$Since = (Get-Date).AddHours(-24)

New-Item -ItemType Directory -Force -Path $EvidencePath

# DNS Server events.
Get-WinEvent -FilterHashtable @{
  LogName = "DNS Server"
  StartTime = $Since
} -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dns-server-events-last-24h.txt"

# DNS Server operational events.
Get-WinEvent -FilterHashtable @{
  LogName = "Microsoft-Windows-DNSServer/Operational"
  StartTime = $Since
} -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dns-server-operational-events-last-24h.txt"

# System events related to DNS.
Get-WinEvent -FilterHashtable @{
  LogName = "System"
  StartTime = $Since
} |
  Where-Object {
    $_.ProviderName -like "*DNS*" -or
    $_.Message -like "*DNS*" -or
    $_.Message -like "*policy*" -or
    $_.Message -like "*zone scope*"
  } |
  Tee-Object "$EvidencePath\system-dns-policy-events-last-24h.txt"

# Final DNS service state.
Get-Service DNS |
  Tee-Object "$EvidencePath\dns-service-final.txt"

# Final policy inventory.
Import-Module DnsServer

Get-DnsServerClientSubnet -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\client-subnets-final.txt"

Get-DnsServerQueryResolutionPolicy -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\query-resolution-policies-final.txt"
```

# Configure_DNS_Policies_Client_Subnets_Zone_Scopes_And_Split_Brain_DNS_Verification_Commands
```powershell
# DNS role and service.
Get-WindowsFeature DNS
Get-Service DNS
Import-Module DnsServer

# Zone baseline.
Get-DnsServerZone -Name "<zone-name>"
Get-DnsServerResourceRecord -ZoneName "<zone-name>"

# Client subnet objects.
Get-DnsServerClientSubnet
Get-DnsServerClientSubnet -Name "<client-subnet-name>"

Add-DnsServerClientSubnet `
  -Name "<client-subnet-name>" `
  -IPv4Subnet "<ipv4-subnet-cidr>"

Remove-DnsServerClientSubnet `
  -Name "<client-subnet-name>" `
  -Force

# Zone scopes.
Get-DnsServerZoneScope -ZoneName "<zone-name>"

Add-DnsServerZoneScope `
  -ZoneName "<zone-name>" `
  -Name "<zone-scope-name>"

Remove-DnsServerZoneScope `
  -ZoneName "<zone-name>" `
  -Name "<zone-scope-name>" `
  -Force

# Default-scope record.
Add-DnsServerResourceRecordA `
  -ZoneName "<zone-name>" `
  -Name "<record-label>" `
  -IPv4Address "<default-ip>"

# Scoped A record.
Add-DnsServerResourceRecordA `
  -ZoneName "<zone-name>" `
  -ZoneScope "<zone-scope-name>" `
  -Name "<record-label>" `
  -IPv4Address "<scope-specific-ip>"

# Verify scoped records.
Get-DnsServerResourceRecord `
  -ZoneName "<zone-name>" `
  -ZoneScope "<zone-scope-name>" `
  -Name "<record-label>"

# Query resolution policies.
Get-DnsServerQueryResolutionPolicy
Get-DnsServerQueryResolutionPolicy -ZoneName "<zone-name>"

Add-DnsServerQueryResolutionPolicy `
  -Name "<policy-name>" `
  -Action ALLOW `
  -ClientSubnet "eq,<client-subnet-name>" `
  -Fqdn "eq,<fqdn>" `
  -ZoneScope "<zone-scope-name>,1" `
  -ZoneName "<zone-name>" `
  -ProcessingOrder 1

# DENY policy example.
Add-DnsServerQueryResolutionPolicy `
  -Name "<deny-policy-name>" `
  -Action DENY `
  -ClientSubnet "eq,<client-subnet-name>" `
  -Fqdn "eq,<blocked-fqdn>" `
  -ZoneName "<zone-name>" `
  -ProcessingOrder 50

# Remove policy.
Remove-DnsServerQueryResolutionPolicy `
  -Name "<policy-name>" `
  -ZoneName "<zone-name>" `
  -Force

# Resolution validation.
Resolve-DnsName "<split-brain-fqdn>" -Server "<dns-server-ip>"
Resolve-DnsName "<default-record-fqdn>" -Server "<dns-server-ip>"
Test-NetConnection "<dns-server-ip>" -Port 53

# Client-side validation.
hostname
Get-NetIPConfiguration
ipconfig /all
Clear-DnsClientCache
Resolve-DnsName "<split-brain-fqdn>" -Server "<dns-server-ip>"

# DNS events.
Get-WinEvent -LogName "DNS Server" -MaxEvents 100
Get-WinEvent -LogName "Microsoft-Windows-DNSServer/Operational" -MaxEvents 100 -ErrorAction SilentlyContinue
Get-WinEvent -LogName System -MaxEvents 100 | Where-Object {$_.ProviderName -like "*DNS*" -or $_.Message -like "*policy*"}
```

# Configure_DNS_Policies_Client_Subnets_Zone_Scopes_And_Split_Brain_DNS_Rollback
| Change | Rollback Command / Action | Risk |
|---|---|---|
| Query policy created incorrectly | `Remove-DnsServerQueryResolutionPolicy -Name "<policy-name>" -ZoneName "<zone-name>" -Force` | Clients may continue receiving wrong answers until policy is removed |
| Client subnet object wrong | Remove and recreate with correct subnet | Policy may match wrong clients |
| Zone scope created incorrectly | Remove scope only after removing policies and records that depend on it | Removing scope deletes scoped records |
| Scoped A record wrong | Remove or replace record inside the specific zone scope | Wrong subnet receives wrong IP |
| Default record wrong | Restore original default-scope record from baseline export | Unmatched clients receive wrong IP |
| Policy processing order wrong | Remove and recreate policy with correct processing order | More general policy may override specific policy |
| DENY policy too broad | Remove DENY policy immediately | Legitimate queries may fail |
| IGNORE policy too broad | Remove IGNORE policy immediately | Clients may see timeouts instead of clear DNS errors |
| Split-brain behavior unwanted | Remove query policies first, then scoped records/scopes if no longer needed | Name resolution returns to default zone behavior |
| DNS cache contains old answer | Clear client cache and server cache during controlled window | Temporary query latency |
| Evidence folder created | `Remove-Item C:\DNS-Policies-SplitBrain -Recurse -Force` | Deletes build and troubleshooting evidence |

# Configure_DNS_Policies_Client_Subnets_Zone_Scopes_And_Split_Brain_DNS_Failure_Checks
| Symptom | Likely Layer | First Check | Do Not Start With |
|---|---|---|---|
| Client gets default answer instead of scoped answer | Client subnet mismatch or policy not matching | `Get-DnsServerClientSubnet`; client IP; policy criteria | Editing the default record |
| Client gets wrong scoped answer | Processing order or subnet overlap | Policy order and subnet definitions | Removing all scopes |
| Policy does not appear to work | FQDN condition, subnet condition, or zone name mismatch | `Get-DnsServerQueryResolutionPolicy -ZoneName <zone>` | Reinstalling DNS |
| Scoped record missing | Record not created inside zone scope | `Get-DnsServerResourceRecord -ZoneScope <scope>` | Changing policy first |
| Zone scope missing | Scope not created or wrong zone | `Get-DnsServerZoneScope -ZoneName <zone>` | Editing client DNS settings |
| Branch clients get internal IP | Branch subnet overlaps internal subnet or policy order wrong | Subnet CIDRs and processing order | Changing app server IP |
| VPN clients get default IP | VPN subnet not defined or client source IP differs | Check actual client source IP seen by DNS path | Adding random forwarder |
| DENY policy blocks legitimate query | Policy too broad | FQDN and client subnet criteria | Restarting DNS service |
| IGNORE policy causes timeouts | IGNORE action expected behavior | Policy action | Troubleshooting firewall first |
| One DNS server behaves differently | Policy exists only on one server or AD replication/config drift | Query each DNS server directly | Blaming client cache only |
| AD-integrated zone policy differs by DC | DNS policy/config replication or manual drift | Compare policy inventory per DNS server | Manually recreating records everywhere |
| Split-brain answer works on server but not client | Client using different DNS server or cached result | `ipconfig /all`; `Clear-DnsClientCache` | Editing scoped records |
| Short name gives unexpected response | Suffix search list chooses different FQDN | `Get-DnsClientGlobalSetting` | Changing policy FQDN first |
| External clients receive internal answer | Policy/subnet boundary wrong or public/internal DNS mixed | Confirm resolver path and subnet match | Publishing more records |
| Internal clients receive external answer | Policy not matching internal subnet | Client source IP and DNS policy match | Changing external DNS |
| Query fails after policy | Missing scope record or DENY/IGNORE policy | Policy action and scoped record inventory | Removing DNS role |
| Weighted zone scopes produce unexpected distribution | Multiple scopes and weights configured | ZoneScope values in policy | Assuming round robin only |
| Recursion issue confused with policy issue | Query is outside authoritative zone | Check zone name and policy scope | Editing authoritative records |

# Configure_DNS_Policies_Client_Subnets_Zone_Scopes_And_Split_Brain_DNS_Related_Labs
| Lab                                                                      | Related Workbook                                                              | Skill Proven                                                          |
| ------------------------------------------------------------------------ | ----------------------------------------------------------------------------- | --------------------------------------------------------------------- |
| Configure AD-integrated DNS zones                                        | `07_Configure_AD_Integrated_DNS_Zones.md`                                     | Authoritative zone baseline before policy design                      |
| Validate AD DNS SRV records and DC locator                               | `08_Validate_AD_DNS_SRV_Records_And_DC_Locator.md`                            | Avoid breaking AD DNS while adding policies                           |
| Configure DNS dynamic updates                                            | `09_Configure_DNS_Dynamic_Updates.md`                                         | Understand dynamic records before scoped records                      |
| Configure DNS aging and scavenging                                       | `10_Configure_DNS_Aging_And_Scavenging.md`                                    | Stale record behavior before split-brain design                       |
| Configure DNS zone transfers, secondary, and stub zones                  | `11_Configure_DNS_Zone_Transfers_Secondary_And_Stub_Zones.md`                 | Multi-server DNS behavior awareness                                   |
| Configure DNS delegation                                                 | `12_Configure_DNS_Delegation.md`                                              | Namespace boundary awareness before policies                          |
| Configure DNS client settings and name resolution                        | `13_Configure_DNS_Client_Settings_And_Name_Resolution.md`                     | Client resolver validation for policy testing                         |
| Monitor DNS events, logs, statistics, and cache                          | `14_Monitor_DNS_Events_Logs_Statistics_And_Cache.md`                          | Evidence collection for policy behavior                               |
| Backup, export, restore DNS zones and server config                      | `15_Backup_Export_Restore_DNS_Zones_And_Server_Config.md`                     | Rollback before policy changes                                        |
| Troubleshoot DNS name resolution failures                                | `16_Troubleshoot_DNS_Name_Resolution_Failures.md`                             | Baseline DNS failure isolation before policy-specific triage          |
| Configure DNSSEC zone signing, trust anchors, and key rollover           | `17_Configure_DNSSEC_Zone_Signing_Trust_Anchors_And_Key_Rollover.md`          | DNSSEC-aware policy change risk                                       |
| Configure DNS policies, client subnets, zone scopes, and split-brain DNS | `18_Configure_DNS_Policies_Client_Subnets_Zone_Scopes_And_Split_Brain_DNS.md` | Policy-based DNS responses and split-brain DNS workflow               |
| Troubleshoot DNS policies and split-brain resolution                     | `19_Troubleshoot_DNS_Policies_And_Split_Brain_Resolution.md`                  | Client subnet, zone scope, policy order, and response mismatch triage |