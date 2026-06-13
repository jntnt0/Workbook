20_Configure_DNS_DoH_And_Encryption_Settings.md
# 20_Configure_DNS_DoH_And_Encryption_Settings

# 20_Configure_DNS_DoH_And_Encryption_Settings_Index
20_Configure_DNS_DoH_And_Encryption_Settings.md
20_Configure_DNS_DoH_And_Encryption_Settings
20_Configure_DNS_DoH_And_Encryption_Settings_Source_Basis
20_Configure_DNS_DoH_And_Encryption_Settings_Mental_Model
20_Configure_DNS_DoH_And_Encryption_Settings_Planning_Table
20_Configure_DNS_DoH_And_Encryption_Settings_Configuration_Checklist
20_Configure_DNS_DoH_And_Encryption_Settings_Client_DoH_Strict_Skeleton
20_Configure_DNS_DoH_And_Encryption_Settings_Client_DoH_Fallback_Allowed_Skeleton
20_Configure_DNS_DoH_And_Encryption_Settings_Domain_Client_Baseline_Skeleton
20_Configure_DNS_DoH_And_Encryption_Settings_Verification_Commands
20_Configure_DNS_DoH_And_Encryption_Settings_Rollback
20_Configure_DNS_DoH_And_Encryption_Settings_Failure_Checks
20_Configure_DNS_DoH_And_Encryption_Settings_Related_Labs

# 20_Configure_DNS_DoH_And_Encryption_Settings_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn / MicrosoftDocs | Windows Server DNS DoH client support | Understanding DoH as a Windows DNS client encryption feature |
| Microsoft Learn / MicrosoftDocs | Add-DnsClientDohServerAddress | Adding a DoH server mapping and DoH template |
| Microsoft Learn / MicrosoftDocs | Get-DnsClientDohServerAddress | Verifying configured DoH server mappings |
| Microsoft Learn / MicrosoftDocs | Remove-DnsClientDohServerAddress | Removing DoH server mappings during rollback |
| Microsoft Learn / MicrosoftDocs | Set-DnsClientServerAddress | Pointing the NIC DNS client setting at the intended resolver IP |
| Windows Server operational practice | DNS client testing, packet validation, fallback control | Confirming whether DNS uses encrypted DoH or falls back to classic UDP/TCP 53 |

# 20_Configure_DNS_DoH_And_Encryption_Settings_Mental_Model
| Concept | Operational Meaning |
|---|---|
| DNS over HTTPS | DNS queries sent over HTTPS instead of classic plaintext UDP/TCP 53 |
| DoH server address | The DNS resolver IP configured on the Windows client NIC |
| DoH template | HTTPS URI used by Windows to send encrypted DNS queries to that resolver |
| Auto upgrade | Windows can upgrade traffic to DoH when the configured DNS server has a matching DoH template |
| UDP fallback | Whether Windows may fall back to classic DNS if encrypted DoH fails |
| Strict DoH | DoH enabled with UDP fallback disabled |
| Opportunistic DoH | DoH enabled but UDP fallback allowed |
| DNS client setting | The per-interface resolver list configured with Set-DnsClientServerAddress |
| DoH mapping | Local Windows mapping between resolver IP and HTTPS DoH template |
| Internal Windows DNS Server | Usually still listens on classic UDP/TCP 53 unless a separate DoH proxy or gateway is used |
| External DoH resolver | Public or private resolver that actually supports DoH endpoint access |
| First rule | DoH is configured on the client resolver path; do not confuse it with AD-integrated DNS zone encryption |
| Blunt rule | If the client points to a normal Windows DNS Server with no DoH front end, DoH will not be used to that server |

# 20_Configure_DNS_DoH_And_Encryption_Settings_Planning_Table
| Item | Example | Decision |
|---|---|---|
| Target client | `WIN11-01` | `<client-name>` |
| Interface alias | `Ethernet` | `<interface-alias>` |
| Primary DoH resolver IP | `1.1.1.1` | `<primary-doh-ip>` |
| Primary DoH template | `https://cloudflare-dns.com/dns-query` | `<primary-doh-template>` |
| Secondary DoH resolver IP | `9.9.9.9` | `<secondary-doh-ip>` |
| Secondary DoH template | `https://dns.quad9.net/dns-query` | `<secondary-doh-template>` |
| Internal DNS server IP | `10.10.10.10` | `<internal-dns-ip>` |
| Mode | Strict DoH | `Strict / Opportunistic / Internal-DNS-Only` |
| Allow UDP fallback | No for strict DoH | `<true-false>` |
| Domain joined client | Yes | `<yes-no>` |
| Test name | `www.microsoft.com` | `<test-name>` |
| AD test name | `_ldap._tcp.dc._msdcs.<domain-fqdn>` | `<ad-srv-test-name>` |
| Rollback resolver | Internal DNS | `<rollback-dns-ip>` |

# 20_Configure_DNS_DoH_And_Encryption_Settings_Configuration_Checklist
| Step | Task | Device | PowerShell | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm OS and DNS client module | Client | `$PSVersionTable; Get-Command Get-DnsClientDohServerAddress -ErrorAction SilentlyContinue` | DoH cmdlets exist on the target system |
| 2 | Confirm interface alias | Client | `Get-NetAdapter` | Correct active adapter is known |
| 3 | Record current DNS client settings | Client | `Get-DnsClientServerAddress -InterfaceAlias "<interface-alias>"` | Current DNS servers are documented |
| 4 | Record existing DoH mappings | Client | `Get-DnsClientDohServerAddress` | Existing DoH mappings are documented |
| 5 | Confirm HTTPS reachability to DoH endpoint | Client | `Test-NetConnection <primary-doh-hostname> -Port 443` | TCP 443 to DoH endpoint succeeds |
| 6 | Add primary DoH mapping | Client | `Add-DnsClientDohServerAddress -ServerAddress "<primary-doh-ip>" -DohTemplate "<primary-doh-template>" -AutoUpgrade $true -AllowFallbackToUdp $false` | Resolver IP is mapped to DoH template |
| 7 | Add secondary DoH mapping | Client | `Add-DnsClientDohServerAddress -ServerAddress "<secondary-doh-ip>" -DohTemplate "<secondary-doh-template>" -AutoUpgrade $true -AllowFallbackToUdp $false` | Secondary resolver IP is mapped to DoH template |
| 8 | Set DNS client resolver list | Client | `Set-DnsClientServerAddress -InterfaceAlias "<interface-alias>" -ServerAddresses "<primary-doh-ip>","<secondary-doh-ip>"` | NIC points at DoH-capable resolver IPs |
| 9 | Flush DNS cache | Client | `Clear-DnsClientCache` | Cached answers are cleared |
| 10 | Verify resolver assignment | Client | `Get-DnsClientServerAddress -InterfaceAlias "<interface-alias>"` | Configured resolver IPs appear |
| 11 | Verify DoH mappings | Client | `Get-DnsClientDohServerAddress` | Resolver IPs show DoH templates and desired fallback behavior |
| 12 | Resolve external name | Client | `Resolve-DnsName "<test-name>"` | Name resolves successfully |
| 13 | Check DNS client events if needed | Client | `Get-WinEvent -LogName "Microsoft-Windows-DNS-Client/Operational" -MaxEvents 50` | DNS client activity is visible |
| 14 | Confirm no dependency on internal AD DNS for public-only test | Client | `Resolve-DnsName "<test-name>" -Server "<primary-doh-ip>"` | Public resolution works through selected resolver |
| 15 | For domain clients, validate AD resolution before enforcing strict external DoH | Client | `Resolve-DnsName "_ldap._tcp.dc._msdcs.<domain-fqdn>" -Type SRV` | AD SRV records still resolve if client requires domain services |
| 16 | If AD SRV records fail, restore internal DNS or use split DNS design | Client | `Set-DnsClientServerAddress -InterfaceAlias "<interface-alias>" -ServerAddresses "<internal-dns-ip>"` | Domain discovery works again |
| 17 | Document final DNS encryption mode | Operator | `Record resolver IPs, DoH templates, fallback mode, interface alias, and rollback DNS` | Workbook notes are complete |

# 20_Configure_DNS_DoH_And_Encryption_Settings_Client_DoH_Strict_Skeleton
```powershell
# Run in elevated PowerShell on the Windows client.
# Strict mode means DoH is preferred and UDP fallback is disabled.
# Use this for public DoH resolvers or private resolvers that truly support DoH.

$InterfaceAlias = "Ethernet"

$PrimaryDohIp = "1.1.1.1"
$PrimaryDohTemplate = "https://cloudflare-dns.com/dns-query"

$SecondaryDohIp = "9.9.9.9"
$SecondaryDohTemplate = "https://dns.quad9.net/dns-query"

$PrimaryDohHost = "cloudflare-dns.com"
$SecondaryDohHost = "dns.quad9.net"

# Confirm local capability.
Get-Command Get-DnsClientDohServerAddress
Get-Command Add-DnsClientDohServerAddress
Get-Command Remove-DnsClientDohServerAddress

# Record current state before changing anything.
Get-NetAdapter
Get-DnsClientServerAddress -InterfaceAlias $InterfaceAlias
Get-DnsClientDohServerAddress

# Confirm HTTPS reachability to DoH endpoints.
Test-NetConnection $PrimaryDohHost -Port 443
Test-NetConnection $SecondaryDohHost -Port 443

# Add DoH mappings.
# AllowFallbackToUdp = false is the strict part.
Add-DnsClientDohServerAddress `
  -ServerAddress $PrimaryDohIp `
  -DohTemplate $PrimaryDohTemplate `
  -AutoUpgrade $true `
  -AllowFallbackToUdp $false

Add-DnsClientDohServerAddress `
  -ServerAddress $SecondaryDohIp `
  -DohTemplate $SecondaryDohTemplate `
  -AutoUpgrade $true `
  -AllowFallbackToUdp $false

# Point the interface DNS client setting at the DoH-capable resolver IPs.
Set-DnsClientServerAddress `
  -InterfaceAlias $InterfaceAlias `
  -ServerAddresses $PrimaryDohIp,$SecondaryDohIp

# Flush cache and verify.
Clear-DnsClientCache

Get-DnsClientServerAddress -InterfaceAlias $InterfaceAlias
Get-DnsClientDohServerAddress

Resolve-DnsName "www.microsoft.com"
Resolve-DnsName "www.cloudflare.com"