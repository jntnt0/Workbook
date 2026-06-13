17_Configure_DNSSEC_Zone_Signing_Trust_Anchors_And_Key_Rollover.md
# Configure_DNSSEC_Zone_Signing_Trust_Anchors_And_Key_Rollover

# Configure_DNSSEC_Zone_Signing_Trust_Anchors_And_Key_Rollover_Index
17_Configure_DNSSEC_Zone_Signing_Trust_Anchors_And_Key_Rollover.md
Configure_DNSSEC_Zone_Signing_Trust_Anchors_And_Key_Rollover
Configure_DNSSEC_Zone_Signing_Trust_Anchors_And_Key_Rollover_Source_Basis
Configure_DNSSEC_Zone_Signing_Trust_Anchors_And_Key_Rollover_Mental_Model
Configure_DNSSEC_Zone_Signing_Trust_Anchors_And_Key_Rollover_Planning_Table
Configure_DNSSEC_Zone_Signing_Trust_Anchors_And_Key_Rollover_Configuration_Checklist
Configure_DNSSEC_Zone_Signing_Trust_Anchors_And_Key_Rollover_Precheck_Skeleton
Configure_DNSSEC_Zone_Signing_Trust_Anchors_And_Key_Rollover_Signing_Key_Inventory_Skeleton
Configure_DNSSEC_Zone_Signing_Trust_Anchors_And_Key_Rollover_Zone_Signing_Skeleton
Configure_DNSSEC_Zone_Signing_Trust_Anchors_And_Key_Rollover_DS_Record_Export_Skeleton
Configure_DNSSEC_Zone_Signing_Trust_Anchors_And_Key_Rollover_Trust_Anchor_Skeleton
Configure_DNSSEC_Zone_Signing_Trust_Anchors_And_Key_Rollover_Validation_Skeleton
Configure_DNSSEC_Zone_Signing_Trust_Anchors_And_Key_Rollover_Key_Rollover_Skeleton
Configure_DNSSEC_Zone_Signing_Trust_Anchors_And_Key_Rollover_Unsign_And_Rollback_Skeleton
Configure_DNSSEC_Zone_Signing_Trust_Anchors_And_Key_Rollover_Event_Log_Skeleton
Configure_DNSSEC_Zone_Signing_Trust_Anchors_And_Key_Rollover_Verification_Commands
Configure_DNSSEC_Zone_Signing_Trust_Anchors_And_Key_Rollover_Rollback
Configure_DNSSEC_Zone_Signing_Trust_Anchors_And_Key_Rollover_Failure_Checks
Configure_DNSSEC_Zone_Signing_Trust_Anchors_And_Key_Rollover_Related_Labs

# Configure_DNSSEC_Zone_Signing_Trust_Anchors_And_Key_Rollover_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | DNS Server PowerShell module | DNSSEC zone signing, key inventory, trust anchors, validation, and DNSSEC record review |
| Microsoft Learn | Add-DnsServerSigningKey | Creating Key Signing Keys and Zone Signing Keys |
| Microsoft Learn | Get-DnsServerSigningKey | Reviewing DNSSEC signing keys |
| Microsoft Learn | Invoke-DnsServerZoneSign | Signing a DNS zone |
| Microsoft Learn | Invoke-DnsServerZoneUnsign | Removing DNSSEC signing from a zone |
| Microsoft Learn | Get-DnsServerDnsSecZoneSetting | Reviewing DNSSEC settings for a signed zone |
| Microsoft Learn | Set-DnsServerDnsSecZoneSetting | Configuring DNSSEC zone settings |
| Microsoft Learn | Get-DnsServerTrustAnchor | Reviewing configured DNSSEC trust anchors |
| Microsoft Learn | Add-DnsServerTrustAnchor | Adding trust anchors for DNSSEC validation |
| Microsoft Learn | Remove-DnsServerTrustAnchor | Removing incorrect trust anchors |
| Microsoft Learn | Update-DnsServerTrustAnchor | Refreshing trust anchor state when supported |
| Microsoft Learn | Resolve-DnsName | Testing DNSKEY, DS, RRSIG, NSEC, and DNSSEC-aware resolution |
| Microsoft Learn | Get-DnsServerResourceRecord | Reviewing DNSKEY, DS, RRSIG, NSEC, NSEC3PARAM, SOA, and NS records |
| Windows Server operational practice | DNSSEC planning, KSK/ZSK separation, DS publication, trust anchor validation, key rollover change control | Signing internal zones and validating DNSSEC without breaking name resolution |

# Configure_DNSSEC_Zone_Signing_Trust_Anchors_And_Key_Rollover_Mental_Model
| Concept | Operational Meaning |
|---|---|
| DNSSEC | DNS Security Extensions that add cryptographic validation to DNS records |
| Signed zone | DNS zone containing DNSSEC records such as DNSKEY, RRSIG, NSEC/NSEC3, and DS-related material |
| DNSKEY record | Public key record used to validate DNSSEC signatures |
| RRSIG record | Signature record proving a DNS record set was signed |
| KSK | Key Signing Key that signs DNSKEY records |
| ZSK | Zone Signing Key that signs ordinary zone record sets |
| DS record | Delegation Signer record placed in the parent zone to establish chain of trust |
| Trust anchor | Trusted DNSKEY or DS material used by a resolver to validate a signed zone |
| Chain of trust | Validation path from trust anchor or parent DS record to child DNSKEY and signed records |
| NSEC/NSEC3 | Authenticated denial-of-existence records proving a name does not exist |
| Key rollover | Controlled replacement of DNSSEC signing keys without breaking validation |
| Pre-publish rollover | Safer approach where new key material is published before old key material is retired |
| Validation failure | Condition where DNSSEC-aware resolvers reject answers because signatures, trust anchors, DS records, or time are wrong |
| Time dependency | DNSSEC validation depends on correct system time because signatures have validity windows |
| First rule | DNSSEC should be configured only after normal DNS is healthy, backed up, and monitored because DNSSEC adds validation failure modes on top of normal DNS failure modes |

# Configure_DNSSEC_Zone_Signing_Trust_Anchors_And_Key_Rollover_Planning_Table
| Item | Example | Decision |
|---|---|---|
| DNS server | `DC1.corp.local` | `<dns-server-fqdn>` |
| DNS server IP | `10.10.10.10` | `<dns-server-ip>` |
| Zone to sign | `corp.local` | `<zone-name>` |
| Zone type | AD-integrated primary | `<zone-type>` |
| Parent zone | `local` or parent internal namespace | `<parent-zone-name>` |
| Parent DNS server | `ROOTDNS1` or parent internal DNS | `<parent-dns-server>` |
| DNSSEC scope | Lab / internal production / public zone | `<dnssec-scope>` |
| KSK algorithm | `RsaSha256` | `<ksk-algorithm>` |
| KSK length | `2048` | `<ksk-length>` |
| ZSK algorithm | `RsaSha256` | `<zsk-algorithm>` |
| ZSK length | `1024` or `2048` | `<zsk-length>` |
| NSEC mode | NSEC or NSEC3 | `<nsec-mode>` |
| DS digest algorithm | SHA256 | `<ds-digest-algorithm>` |
| Trust anchor target | `corp.local` | `<trust-anchor-zone>` |
| Resolver validation enabled | Yes / No | `<yes-no>` |
| Key rollover method | Pre-publish / manual lab rollover | `<rollover-method>` |
| Signature validity window | Default / custom | `<signature-validity>` |
| Evidence path | `C:\DNSSEC-Zone-Signing` | `<evidence-path>` |
| Backup path | `D:\DNS-Backups` | `<backup-path>` |
| Rollback stance | Export current zone and config before signing; unsign only during approved rollback window | `<rollback-plan>` |
| Next workbook | `18_Troubleshoot_DNSSEC_Validation_And_Signing_Failures.md` | `<next-task>` |

# Configure_DNSSEC_Zone_Signing_Trust_Anchors_And_Key_Rollover_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Open elevated PowerShell | DNS Server | `whoami /groups` | Admin context is visible |
| 2 | Confirm DNS role installed | DNS Server | `Get-WindowsFeature DNS` | DNS role is installed |
| 3 | Confirm DNS service running | DNS Server | `Get-Service DNS` | DNS service is Running |
| 4 | Import DNS module | DNS Server | `Import-Module DnsServer` | DNS cmdlets are available |
| 5 | Confirm zone exists | DNS Server | `Get-DnsServerZone -Name "<zone-name>"` | Target zone exists |
| 6 | Confirm normal zone resolution first | DNS Server / Client | `Resolve-DnsName "<known-record>" -Server "<dns-server-ip>"` | Normal DNS works before DNSSEC |
| 7 | Export zone and server config backup | DNS Server | Use backup workbook commands | Rollback evidence exists |
| 8 | Capture current DNSSEC state | DNS Server | `Get-DnsServerSigningKey -ZoneName "<zone-name>"`; `Get-DnsServerDnsSecZoneSetting -ZoneName "<zone-name>"` | Existing DNSSEC state is documented |
| 9 | Create or confirm KSK | DNS Server | `Add-DnsServerSigningKey -ZoneName "<zone-name>" -KeyType Ksk ...` | KSK exists |
| 10 | Create or confirm ZSK | DNS Server | `Add-DnsServerSigningKey -ZoneName "<zone-name>" -KeyType Zsk ...` | ZSK exists |
| 11 | Sign the zone | DNS Server | `Invoke-DnsServerZoneSign -ZoneName "<zone-name>"` | Zone signing begins or completes |
| 12 | Confirm signed zone settings | DNS Server | `Get-DnsServerDnsSecZoneSetting -ZoneName "<zone-name>"` | DNSSEC settings are visible |
| 13 | Confirm DNSKEY records | DNS Server | `Get-DnsServerResourceRecord -ZoneName "<zone-name>" -RRType DNSKEY` | DNSKEY records exist |
| 14 | Confirm RRSIG records | DNS Server | `Get-DnsServerResourceRecord -ZoneName "<zone-name>" -RRType RRSIG` | Signatures exist |
| 15 | Confirm NSEC/NSEC3 records | DNS Server | `Get-DnsServerResourceRecord -ZoneName "<zone-name>" -RRType NSEC` or `NSEC3PARAM` | Authenticated denial records exist |
| 16 | Export DS material for parent zone | DNS Server | Use DS export skeleton | DS material is captured for parent publication |
| 17 | Publish DS record in parent zone if required | Parent DNS Server | Add DS record through approved parent-zone method | Chain of trust can be established |
| 18 | Configure trust anchor if internal validation uses trust anchors | Validating Resolver / DNS Server | `Add-DnsServerTrustAnchor` | Trust anchor exists |
| 19 | Validate trust anchor inventory | Validating Resolver / DNS Server | `Get-DnsServerTrustAnchor` | Trust anchor state is visible |
| 20 | Test DNSSEC-aware lookup | Client / Resolver | `Resolve-DnsName "<record>" -DnssecOk -Server "<dns-server-ip>"` | DNSSEC-aware answer is returned |
| 21 | Test DNSKEY lookup | Client / Resolver | `Resolve-DnsName "<zone-name>" -Type DNSKEY -DnssecOk -Server "<dns-server-ip>"` | DNSKEY records resolve |
| 22 | Test DS lookup through parent path | Client / Resolver | `Resolve-DnsName "<zone-name>" -Type DS -DnssecOk -Server "<parent-dns-ip>"` | DS record resolves if published |
| 23 | Review DNS Server events | DNS Server | `Get-WinEvent -LogName "DNS Server" -MaxEvents 100` | Signing and validation events are visible |
| 24 | Document key IDs and rollover plan | Operator | `Record KSK, ZSK, algorithms, DS, trust anchors, and rollover dates` | DNSSEC build record is complete |
| 25 | Perform lab rollover only after validation | DNS Server | Use rollover skeleton | New key material is introduced safely |
| 26 | Retest after rollover | Client / Resolver | Repeat DNSKEY, DS, RRSIG, and normal lookups | Validation still succeeds |

# Configure_DNSSEC_Zone_Signing_Trust_Anchors_And_Key_Rollover_Precheck_Skeleton
```powershell
# Run in elevated PowerShell on the DNS server.
# Purpose: capture DNS health, zone state, existing DNSSEC state, and backup evidence before signing.

$ZoneName = "corp.local"
$KnownRecord = "dc1.corp.local"
$DnsServerIp = "10.10.10.10"
$EvidencePath = "C:\DNSSEC-Zone-Signing"

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

# Confirm zone exists and capture properties.
Get-DnsServerZone -Name $ZoneName |
  Select-Object ZoneName,ZoneType,IsDsIntegrated,ReplicationScope,DynamicUpdate,IsPaused,IsShutdown |
  Tee-Object "$EvidencePath\zone-properties-before-dnssec.txt"

# Export current records.
Get-DnsServerResourceRecord -ZoneName $ZoneName |
  Tee-Object "$EvidencePath\zone-records-before-dnssec.txt"

Get-DnsServerResourceRecord -ZoneName $ZoneName |
  Export-Csv "$EvidencePath\zone-records-before-dnssec.csv" -NoTypeInformation

# Export current zone file if supported.
Export-DnsServerZone -Name $ZoneName -FileName "$ZoneName.pre-dnssec.dns" -ErrorAction SilentlyContinue

# Confirm normal DNS resolution before DNSSEC.
Resolve-DnsName $KnownRecord -Server $DnsServerIp |
  Tee-Object "$EvidencePath\resolve-known-record-before-dnssec.txt"

# Capture current DNSSEC signing keys if present.
Get-DnsServerSigningKey -ZoneName $ZoneName -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\signing-keys-before.txt"

# Capture DNSSEC zone settings if present.
Get-DnsServerDnsSecZoneSetting -ZoneName $ZoneName -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dnssec-zone-settings-before.txt"

# Capture trust anchors.
Get-DnsServerTrustAnchor -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\trust-anchors-before.txt"

# Capture DNS events before change.
Get-WinEvent -LogName "DNS Server" -MaxEvents 100 -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dns-server-events-before-dnssec.txt"
```

# Configure_DNSSEC_Zone_Signing_Trust_Anchors_And_Key_Rollover_Signing_Key_Inventory_Skeleton
```powershell
# Run in elevated PowerShell on the DNS server.
# Purpose: inspect and document DNSSEC signing keys before and after signing.

$ZoneName = "corp.local"
$EvidencePath = "C:\DNSSEC-Zone-Signing"

New-Item -ItemType Directory -Force -Path $EvidencePath

Import-Module DnsServer

# Capture existing signing keys.
Get-DnsServerSigningKey -ZoneName $ZoneName -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\signing-keys-current.txt"

Get-DnsServerSigningKey -ZoneName $ZoneName -ErrorAction SilentlyContinue |
  Select-Object * |
  Export-Csv "$EvidencePath\signing-keys-current.csv" -NoTypeInformation

# Capture current DNSSEC zone settings.
Get-DnsServerDnsSecZoneSetting -ZoneName $ZoneName -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dnssec-zone-settings-current.txt"

# Capture DNSSEC-related records if present.
foreach ($Type in @("DNSKEY","RRSIG","NSEC","NSEC3PARAM","DS")) {
  Get-DnsServerResourceRecord -ZoneName $ZoneName -RRType $Type -ErrorAction SilentlyContinue |
    Tee-Object "$EvidencePath\records-$Type-current.txt"
}
```

# Configure_DNSSEC_Zone_Signing_Trust_Anchors_And_Key_Rollover_Zone_Signing_Skeleton
```powershell
# Run in elevated PowerShell on the authoritative DNS server for the zone.
# Purpose: create DNSSEC signing keys and sign the zone.
# Validate in lab first. DNSSEC can break validation if trust anchors, DS records, time, or signatures are wrong.

$ZoneName = "corp.local"
$KskLength = 2048
$ZskLength = 1024
$CryptoAlgorithm = "RsaSha256"
$EvidencePath = "C:\DNSSEC-Zone-Signing"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\dnssec-zone-signing-transcript.txt"

Import-Module DnsServer

# Capture pre-signing state.
Get-DnsServerZone -Name $ZoneName |
  Tee-Object "$EvidencePath\zone-before-signing.txt"

Get-DnsServerSigningKey -ZoneName $ZoneName -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\signing-keys-before-signing.txt"

# Add Key Signing Key if missing.
# Use lab-approved algorithm and length. Production standards should follow organizational cryptographic policy.
Add-DnsServerSigningKey `
  -ZoneName $ZoneName `
  -KeyType Ksk `
  -CryptoAlgorithm $CryptoAlgorithm `
  -KeyLength $KskLength `
  -KeyProtocol DnsSec `
  -PassThru |
  Tee-Object "$EvidencePath\add-ksk-output.txt"

# Add Zone Signing Key if missing.
Add-DnsServerSigningKey `
  -ZoneName $ZoneName `
  -KeyType Zsk `
  -CryptoAlgorithm $CryptoAlgorithm `
  -KeyLength $ZskLength `
  -KeyProtocol DnsSec `
  -PassThru |
  Tee-Object "$EvidencePath\add-zsk-output.txt"

# Confirm keys before signing.
Get-DnsServerSigningKey -ZoneName $ZoneName |
  Tee-Object "$EvidencePath\signing-keys-after-add.txt"

# Sign the zone.
Invoke-DnsServerZoneSign `
  -ZoneName $ZoneName

# Confirm DNSSEC settings after signing.
Get-DnsServerDnsSecZoneSetting -ZoneName $ZoneName |
  Tee-Object "$EvidencePath\dnssec-zone-settings-after-signing.txt"

# Confirm DNSSEC records.
foreach ($Type in @("DNSKEY","RRSIG","NSEC","NSEC3PARAM")) {
  Get-DnsServerResourceRecord -ZoneName $ZoneName -RRType $Type -ErrorAction SilentlyContinue |
    Tee-Object "$EvidencePath\records-$Type-after-signing.txt"
}

Stop-Transcript
```

# Configure_DNSSEC_Zone_Signing_Trust_Anchors_And_Key_Rollover_DS_Record_Export_Skeleton
```powershell
# Run in elevated PowerShell on the authoritative DNS server.
# Purpose: collect DNSKEY and DS-related material for parent-zone publication.
# Parent DS publication is required for a normal delegated chain of trust.

$ZoneName = "corp.local"
$ParentDnsServerIp = "10.10.0.10"
$EvidencePath = "C:\DNSSEC-Zone-Signing"

New-Item -ItemType Directory -Force -Path $EvidencePath

Import-Module DnsServer

# Export DNSKEY records.
Get-DnsServerResourceRecord -ZoneName $ZoneName -RRType DNSKEY -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dnskey-records-for-ds-publication.txt"

Get-DnsServerResourceRecord -ZoneName $ZoneName -RRType DNSKEY -ErrorAction SilentlyContinue |
  Export-Csv "$EvidencePath\dnskey-records-for-ds-publication.csv" -NoTypeInformation

# Query DNSKEY through resolver path.
Resolve-DnsName $ZoneName -Type DNSKEY -DnssecOk -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\resolve-dnskey-default.txt"

# Query DS through parent path if parent already has DS.
Resolve-DnsName $ZoneName -Type DS -DnssecOk -Server $ParentDnsServerIp -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\resolve-ds-through-parent-before-publication.txt"

# Capture DNSSEC settings and keys for handoff.
Get-DnsServerDnsSecZoneSetting -ZoneName $ZoneName -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dnssec-zone-settings-for-parent-handoff.txt"

Get-DnsServerSigningKey -ZoneName $ZoneName -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\signing-key-inventory-for-parent-handoff.txt"

# Manual parent-zone action note:
# Publish the correct DS record in the parent zone according to the parent DNS platform.
# For internal Windows DNS parent zones, use approved record creation/change workflow.
# Validate DS publication before enabling validation enforcement for dependent resolvers.
```

# Configure_DNSSEC_Zone_Signing_Trust_Anchors_And_Key_Rollover_Trust_Anchor_Skeleton
```powershell
# Run in elevated PowerShell on validating DNS server/resolver.
# Purpose: add or validate DNSSEC trust anchors for an internally signed zone.
# Use trust anchors when no parent DS chain is available or for controlled internal validation.

$ZoneName = "corp.local"
$TrustAnchorFile = "C:\DNSSEC-Zone-Signing\trust-anchor-input.txt"
$EvidencePath = "C:\DNSSEC-Zone-Signing"

New-Item -ItemType Directory -Force -Path $EvidencePath

Import-Module DnsServer

# Capture existing trust anchors.
Get-DnsServerTrustAnchor -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\trust-anchors-before-config.txt"

# Optional trust anchor add example.
# The exact method depends on whether the trust anchor is entered from DNSKEY/DS material or imported.
# Validate syntax in lab and use the approved trust anchor source.
# Add-DnsServerTrustAnchor `
#   -Name $ZoneName `
#   -CryptoAlgorithm RsaSha256 `
#   -KeyProtocol DnsSec `
#   -KeyTag <key-tag> `
#   -DigestType Sha256 `
#   -Digest "<digest-value>"

# If supported in your environment, update trust anchors.
Update-DnsServerTrustAnchor -Name $ZoneName -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\trust-anchor-update-output.txt"

# Confirm trust anchors after configuration.
Get-DnsServerTrustAnchor -Name $ZoneName -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\trust-anchor-after-config.txt"

Get-DnsServerTrustAnchor -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\trust-anchors-all-after-config.txt"
```

# Configure_DNSSEC_Zone_Signing_Trust_Anchors_And_Key_Rollover_Validation_Skeleton
```powershell
# Run from DNS server and from at least one client/resolver.
# Purpose: validate signed zone records, DNSSEC-aware lookups, and normal name resolution.

$ZoneName = "corp.local"
$DnsServerIp = "10.10.10.10"
$KnownRecord = "dc1.corp.local"
$NegativeTestName = "does-not-exist.corp.local"
$EvidencePath = "C:\DNSSEC-Zone-Signing\Validation"

New-Item -ItemType Directory -Force -Path $EvidencePath

Import-Module DnsServer

# Confirm signed DNSSEC zone settings.
Get-DnsServerDnsSecZoneSetting -ZoneName $ZoneName -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dnssec-zone-settings.txt"

# Confirm signing keys.
Get-DnsServerSigningKey -ZoneName $ZoneName -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\signing-keys.txt"

# Confirm DNSSEC record presence.
foreach ($Type in @("DNSKEY","RRSIG","NSEC","NSEC3PARAM")) {
  Get-DnsServerResourceRecord -ZoneName $ZoneName -RRType $Type -ErrorAction SilentlyContinue |
    Tee-Object "$EvidencePath\zone-records-$Type.txt"
}

# Normal lookup still works.
Resolve-DnsName $KnownRecord -Server $DnsServerIp -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\resolve-known-record-normal.txt"

# DNSSEC-aware A lookup.
Resolve-DnsName $KnownRecord -Server $DnsServerIp -DnssecOk -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\resolve-known-record-dnssec-ok.txt"

# DNSKEY lookup.
Resolve-DnsName $ZoneName -Type DNSKEY -Server $DnsServerIp -DnssecOk -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\resolve-zone-dnskey.txt"

# RRSIG lookup.
Resolve-DnsName $KnownRecord -Type RRSIG -Server $DnsServerIp -DnssecOk -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\resolve-known-record-rrsig.txt"

# Negative lookup should return authenticated denial data when DNSSEC-aware.
Resolve-DnsName $NegativeTestName -Server $DnsServerIp -DnssecOk -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\resolve-negative-test-dnssec-ok.txt"

# Trust anchor validation inventory.
Get-DnsServerTrustAnchor -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\trust-anchors.txt"

# DNS Server events.
Get-WinEvent -LogName "DNS Server" -MaxEvents 200 -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dns-server-events-after-validation.txt"
```

# Configure_DNSSEC_Zone_Signing_Trust_Anchors_And_Key_Rollover_Key_Rollover_Skeleton
```powershell
# Run in elevated PowerShell on authoritative DNS server during approved rollover window.
# Purpose: perform controlled DNSSEC key rollover in lab or approved change window.
# Do not remove old keys until caches, DS records, and trust anchors have safely converged.

$ZoneName = "corp.local"
$NewKskLength = 2048
$NewZskLength = 1024
$CryptoAlgorithm = "RsaSha256"
$EvidencePath = "C:\DNSSEC-Zone-Signing\Key-Rollover"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\dnssec-key-rollover-transcript.txt"

Import-Module DnsServer

# Capture keys before rollover.
Get-DnsServerSigningKey -ZoneName $ZoneName |
  Tee-Object "$EvidencePath\signing-keys-before-rollover.txt"

# Capture DNSKEY before rollover.
Get-DnsServerResourceRecord -ZoneName $ZoneName -RRType DNSKEY -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dnskey-before-rollover.txt"

# Pre-publish new ZSK.
Add-DnsServerSigningKey `
  -ZoneName $ZoneName `
  -KeyType Zsk `
  -CryptoAlgorithm $CryptoAlgorithm `
  -KeyLength $NewZskLength `
  -KeyProtocol DnsSec `
  -PassThru |
  Tee-Object "$EvidencePath\new-zsk-add-output.txt"

# Optional pre-publish new KSK.
# Use only when KSK rollover is planned and parent DS/trust anchor process is ready.
Add-DnsServerSigningKey `
  -ZoneName $ZoneName `
  -KeyType Ksk `
  -CryptoAlgorithm $CryptoAlgorithm `
  -KeyLength $NewKskLength `
  -KeyProtocol DnsSec `
  -PassThru |
  Tee-Object "$EvidencePath\new-ksk-add-output.txt"

# Re-sign zone after adding new keys.
Invoke-DnsServerZoneSign `
  -ZoneName $ZoneName

# Capture keys and DNSKEY after rollover publish phase.
Get-DnsServerSigningKey -ZoneName $ZoneName |
  Tee-Object "$EvidencePath\signing-keys-after-new-key-publish.txt"

Get-DnsServerResourceRecord -ZoneName $ZoneName -RRType DNSKEY -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dnskey-after-new-key-publish.txt"

Get-DnsServerResourceRecord -ZoneName $ZoneName -RRType RRSIG -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\rrsig-after-new-key-publish.txt"

# Manual change-control note:
# 1. Publish new DS in parent zone if KSK rollover is happening.
# 2. Wait for TTL/cache convergence.
# 3. Update trust anchors if internal trust anchors are used.
# 4. Validate DNSSEC-aware resolution from multiple resolvers.
# 5. Only then retire old key material.

Stop-Transcript
```

# Configure_DNSSEC_Zone_Signing_Trust_Anchors_And_Key_Rollover_Unsign_And_Rollback_Skeleton
```powershell
# Run in elevated PowerShell on authoritative DNS server only during approved rollback.
# Purpose: remove DNSSEC signing from a zone if validation failure cannot be safely fixed.
# This is disruptive for DNSSEC validators if DS/trust anchors remain published.

$ZoneName = "corp.local"
$EvidencePath = "C:\DNSSEC-Zone-Signing\Rollback"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\dnssec-unsign-rollback-transcript.txt"

Import-Module DnsServer

# Capture DNSSEC state before rollback.
Get-DnsServerSigningKey -ZoneName $ZoneName -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\signing-keys-before-unsign.txt"

Get-DnsServerDnsSecZoneSetting -ZoneName $ZoneName -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dnssec-zone-settings-before-unsign.txt"

foreach ($Type in @("DNSKEY","RRSIG","NSEC","NSEC3PARAM","DS")) {
  Get-DnsServerResourceRecord -ZoneName $ZoneName -RRType $Type -ErrorAction SilentlyContinue |
    Tee-Object "$EvidencePath\records-$Type-before-unsign.txt"
}

# Unsign the zone.
Invoke-DnsServerZoneUnsign `
  -ZoneName $ZoneName

# Confirm DNSSEC records are removed or no longer active.
foreach ($Type in @("DNSKEY","RRSIG","NSEC","NSEC3PARAM","DS")) {
  Get-DnsServerResourceRecord -ZoneName $ZoneName -RRType $Type -ErrorAction SilentlyContinue |
    Tee-Object "$EvidencePath\records-$Type-after-unsign.txt"
}

# Manual parent/trust-anchor rollback note:
# Remove DS record from parent zone if published.
# Remove or update internal trust anchors that point to old key material.
# Clear resolver caches only during approved troubleshooting window.

Stop-Transcript
```

# Configure_DNSSEC_Zone_Signing_Trust_Anchors_And_Key_Rollover_Event_Log_Skeleton
```powershell
# Run on authoritative DNS server and validating resolver.
# Purpose: collect DNSSEC signing, validation, service, and DNS query event evidence.

$EvidencePath = "C:\DNSSEC-Zone-Signing\Events"
$Since = (Get-Date).AddHours(-24)

New-Item -ItemType Directory -Force -Path $EvidencePath

# DNS Server classic events.
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

# System DNS-related events.
Get-WinEvent -FilterHashtable @{
  LogName = "System"
  StartTime = $Since
} |
  Where-Object {
    $_.ProviderName -like "*DNS*" -or
    $_.Message -like "*DNSSEC*" -or
    $_.Message -like "*signature*" -or
    $_.Message -like "*trust anchor*" -or
    $_.Message -like "*validation*"
  } |
  Tee-Object "$EvidencePath\system-dnssec-events-last-24h.txt"

# Directory Service events for AD-integrated DNS zones.
Get-WinEvent -FilterHashtable @{
  LogName = "Directory Service"
  StartTime = $Since
} -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\directory-service-events-last-24h.txt"

# Final DNS service state.
Get-Service DNS |
  Tee-Object "$EvidencePath\dns-service-final.txt"
```

# Configure_DNSSEC_Zone_Signing_Trust_Anchors_And_Key_Rollover_Verification_Commands
```powershell
# DNS role and service.
Get-WindowsFeature DNS
Get-Service DNS
Import-Module DnsServer

# Confirm zone.
Get-DnsServerZone -Name "<zone-name>"
Get-DnsServerZone -Name "<zone-name>" |
  Select-Object ZoneName,ZoneType,IsDsIntegrated,ReplicationScope,DynamicUpdate

# Export zone before signing.
Export-DnsServerZone -Name "<zone-name>" -FileName "<zone-name>.pre-dnssec.dns"
Get-DnsServerResourceRecord -ZoneName "<zone-name>" |
  Export-Csv "<evidence-path>\<zone-name>-records-before-dnssec.csv" -NoTypeInformation

# Existing DNSSEC state.
Get-DnsServerSigningKey -ZoneName "<zone-name>"
Get-DnsServerDnsSecZoneSetting -ZoneName "<zone-name>"
Get-DnsServerTrustAnchor

# Add KSK.
Add-DnsServerSigningKey `
  -ZoneName "<zone-name>" `
  -KeyType Ksk `
  -CryptoAlgorithm RsaSha256 `
  -KeyLength 2048 `
  -KeyProtocol DnsSec `
  -PassThru

# Add ZSK.
Add-DnsServerSigningKey `
  -ZoneName "<zone-name>" `
  -KeyType Zsk `
  -CryptoAlgorithm RsaSha256 `
  -KeyLength 1024 `
  -KeyProtocol DnsSec `
  -PassThru

# Sign zone.
Invoke-DnsServerZoneSign -ZoneName "<zone-name>"

# Review DNSSEC settings and records.
Get-DnsServerDnsSecZoneSetting -ZoneName "<zone-name>"
Get-DnsServerSigningKey -ZoneName "<zone-name>"
Get-DnsServerResourceRecord -ZoneName "<zone-name>" -RRType DNSKEY
Get-DnsServerResourceRecord -ZoneName "<zone-name>" -RRType RRSIG
Get-DnsServerResourceRecord -ZoneName "<zone-name>" -RRType NSEC
Get-DnsServerResourceRecord -ZoneName "<zone-name>" -RRType NSEC3PARAM

# DNSSEC-aware lookups.
Resolve-DnsName "<known-record-fqdn>" -Server "<dns-server-ip>" -DnssecOk
Resolve-DnsName "<zone-name>" -Type DNSKEY -Server "<dns-server-ip>" -DnssecOk
Resolve-DnsName "<zone-name>" -Type DS -Server "<parent-dns-ip>" -DnssecOk
Resolve-DnsName "<known-record-fqdn>" -Type RRSIG -Server "<dns-server-ip>" -DnssecOk

# Trust anchors.
Get-DnsServerTrustAnchor
Get-DnsServerTrustAnchor -Name "<zone-name>"
Update-DnsServerTrustAnchor -Name "<zone-name>"

# Add trust anchor example placeholder.
# Add-DnsServerTrustAnchor -Name "<zone-name>" <approved-trust-anchor-parameters>

# Key rollover inventory.
Get-DnsServerSigningKey -ZoneName "<zone-name>" |
  Select-Object *

# Add new rollover key examples.
Add-DnsServerSigningKey `
  -ZoneName "<zone-name>" `
  -KeyType Zsk `
  -CryptoAlgorithm RsaSha256 `
  -KeyLength 1024 `
  -KeyProtocol DnsSec `
  -PassThru

Invoke-DnsServerZoneSign -ZoneName "<zone-name>"

# Unsign rollback.
Invoke-DnsServerZoneUnsign -ZoneName "<zone-name>"

# Events.
Get-WinEvent -LogName "DNS Server" -MaxEvents 100
Get-WinEvent -LogName "Microsoft-Windows-DNSServer/Operational" -MaxEvents 100 -ErrorAction SilentlyContinue
Get-WinEvent -LogName System -MaxEvents 100 | Where-Object {$_.ProviderName -like "*DNS*" -or $_.Message -like "*DNSSEC*" -or $_.Message -like "*validation*"}
```

# Configure_DNSSEC_Zone_Signing_Trust_Anchors_And_Key_Rollover_Rollback
| Change | Rollback Command / Action | Risk |
|---|---|---|
| Zone signed incorrectly | `Invoke-DnsServerZoneUnsign -ZoneName "<zone-name>"` | DNSSEC validators may still fail if DS or trust anchors remain |
| KSK created incorrectly | Remove incorrect signing key only after confirming it is not active or needed | Removing active key can break validation |
| ZSK created incorrectly | Remove incorrect signing key only after re-signing with valid key material | Signed records may fail validation |
| DS published incorrectly | Remove or replace DS record in parent zone | Chain of trust breaks until corrected |
| Trust anchor added incorrectly | `Remove-DnsServerTrustAnchor -Name "<zone-name>"` or replace with correct anchor | Validating resolver may reject valid answers |
| Key rollover started incorrectly | Pause, validate DNSKEY/DS/trust anchors, keep old key until safe | Removing old key too soon breaks validation |
| DNSSEC validation failing | Fix trust anchor, DS, signature, time, or key state before unsigning | Random cache clearing will not fix bad signatures |
| Resolver cache contains bad validation state | Clear resolver cache during maintenance window | Temporary query latency |
| Zone export only | No rollback required | Export operation does not change zone |
| Event evidence folder created | `Remove-Item C:\DNSSEC-Zone-Signing -Recurse -Force` | Deletes validation evidence |
| Diagnostic logging enabled during DNSSEC troubleshooting | Disable after troubleshooting | Excessive log volume |

# Configure_DNSSEC_Zone_Signing_Trust_Anchors_And_Key_Rollover_Failure_Checks
| Symptom | Likely Layer | First Check | Do Not Start With |
|---|---|---|---|
| Normal lookup fails after signing | Zone signing, records, or DNS service issue | Query same record with and without `-DnssecOk` | Removing all keys immediately |
| DNSSEC-aware lookup fails only | Trust anchor, DS, signature, or validation issue | DNSKEY, DS, RRSIG, trust anchor inventory | Recreating the zone |
| DNSKEY records missing | Zone not signed or signing failed | `Get-DnsServerSigningKey`; DNS Server events | Publishing DS first |
| RRSIG records missing | Zone not signed or signing incomplete | `Invoke-DnsServerZoneSign`; DNS events | Adding more trust anchors |
| DS lookup fails | DS not published in parent zone | Parent zone DS records | Re-signing child zone blindly |
| Trust anchor mismatch | Resolver has wrong key/digest | `Get-DnsServerTrustAnchor` and DNSKEY inventory | Clearing client cache only |
| Validation fails after KSK rollover | Parent DS or trust anchor not updated | DS record and trust anchor state | Removing old KSK immediately |
| Validation fails after ZSK rollover | Old ZSK retired too early or signatures not refreshed | DNSKEY and RRSIG records | Changing parent DS |
| Negative lookups fail validation | NSEC/NSEC3 issue | NSEC/NSEC3PARAM records and DNSSEC zone settings | Editing A records |
| Some resolvers validate and others fail | Trust anchor/cache/config differs by resolver | Query each resolver directly | Assuming authoritative zone is broken |
| Time-related validation errors | Clock skew or expired signatures | Server time and signature validity | Recreating keys first |
| AD-integrated signed zone differs by DC | AD replication issue | `repadmin /replsummary`; query each DC | Manually editing each DC |
| Public clients fail but internal works | Parent/public DS chain issue | Parent DS and public authoritative path | Changing internal trust anchor only |
| Internal clients fail but public works | Internal resolver trust anchor issue | Internal resolver trust anchors | Changing parent registrar DS |
| Unsign did not fix validation | DS/trust anchors still point to signed expectation | Remove DS/trust anchors or wait for TTL | Rebooting all clients |
| DNSSEC event logs noisy | Diagnostic logging or validation failures | DNS Server events and diagnostics | Ignoring validation errors |
| Signature expiration warnings | Signing schedule or key state issue | DNSSEC zone settings and signing key status | Clearing server cache |
| Record changed but signature stale | Zone not re-signed after change | RRSIG timing and zone signing status | Recreating record repeatedly |

# Configure_DNSSEC_Zone_Signing_Trust_Anchors_And_Key_Rollover_Related_Labs
| Lab                                                            | Related Workbook                                                     | Skill Proven                                                            |
| -------------------------------------------------------------- | -------------------------------------------------------------------- | ----------------------------------------------------------------------- |
| Configure AD-integrated DNS zones                              | `07_Configure_AD_Integrated_DNS_Zones.md`                            | Authoritative zone baseline before DNSSEC                               |
| Validate AD DNS SRV records and DC locator                     | `08_Validate_AD_DNS_SRV_Records_And_DC_Locator.md`                   | AD DNS validation before signing                                        |
| Configure DNS dynamic updates                                  | `09_Configure_DNS_Dynamic_Updates.md`                                | Dynamic record behavior before DNSSEC signing                           |
| Configure DNS aging and scavenging                             | `10_Configure_DNS_Aging_And_Scavenging.md`                           | Timestamp and stale record awareness before signing                     |
| Configure DNS zone transfers, secondary, and stub zones        | `11_Configure_DNS_Zone_Transfers_Secondary_And_Stub_Zones.md`        | DNSSEC implications for secondary/stub design                           |
| Configure DNS delegation                                       | `12_Configure_DNS_Delegation.md`                                     | Parent/child namespace chain-of-trust awareness                         |
| Configure DNS client settings and name resolution              | `13_Configure_DNS_Client_Settings_And_Name_Resolution.md`            | Resolver behavior validation                                            |
| Monitor DNS events, logs, statistics, and cache                | `14_Monitor_DNS_Events_Logs_Statistics_And_Cache.md`                 | DNSSEC event and cache evidence collection                              |
| Backup, export, restore DNS zones and server config            | `15_Backup_Export_Restore_DNS_Zones_And_Server_Config.md`            | Pre-signing and rollback backup workflow                                |
| Troubleshoot DNS name resolution failures                      | `16_Troubleshoot_DNS_Name_Resolution_Failures.md`                    | Baseline DNS troubleshooting before DNSSEC-specific triage              |
| Configure DNSSEC zone signing, trust anchors, and key rollover | `17_Configure_DNSSEC_Zone_Signing_Trust_Anchors_And_Key_Rollover.md` | DNSSEC signing, trust anchor, DS, validation, and key rollover workflow |
| Troubleshoot DNSSEC validation and signing failures            | `18_Troubleshoot_DNSSEC_Validation_And_Signing_Failures.md`          | DNSSEC-specific failure isolation                                       |