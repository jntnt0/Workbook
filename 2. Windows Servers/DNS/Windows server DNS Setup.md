Windows_Server_DNS_Setup.md
# Windows_Server_DNS_Setup

# Windows_Server_DNS_Setup_Index
Windows_Server_DNS_Setup.md
Windows_Server_DNS_Setup
Windows_Server_DNS_Setup_Source_Basis
Windows_Server_DNS_Setup_Mental_Model
Windows_Server_DNS_Setup_Planning_Table
Windows_Server_DNS_Setup_Configuration_Checklist
Windows_Server_DNS_Setup_AD_Integrated_Skeleton
Windows_Server_DNS_Setup_File_Backed_Skeleton
Windows_Server_DNS_Setup_Record_Creation_Skeleton
Windows_Server_DNS_Setup_Client_DNS_Skeleton
Windows_Server_DNS_Setup_Verification_Commands
Windows_Server_DNS_Setup_Rollback
Windows_Server_DNS_Setup_Failure_Checks
Windows_Server_DNS_Setup_Related_Labs

# Windows_Server_DNS_Setup_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Install-WindowsFeature | Installing DNS Server role and management tools |
| Microsoft Learn | Add-DnsServerPrimaryZone | Creating forward lookup zones and reverse lookup zones |
| Microsoft Learn | Add-DnsServerResourceRecordA | Creating A records and optional PTR records |
| Microsoft Learn | Add-DnsServerForwarder | Configuring upstream DNS forwarders |
| Microsoft Learn | Set-DnsClientServerAddress | Pointing Windows clients or servers at the DNS server |

# Windows_Server_DNS_Setup_Mental_Model
| Concept | Operational Meaning |
|---|---|
| DNS Server role | Windows Server service that answers DNS queries and hosts DNS zones |
| DNS zone | Administrative container for DNS records, such as `corp.local` |
| Forward lookup zone | Resolves names to IP addresses |
| Reverse lookup zone | Resolves IP addresses back to names through PTR records |
| A record | Maps hostname to IPv4 address |
| PTR record | Maps IPv4 address back to hostname |
| AD-integrated zone | DNS zone stored in Active Directory and replicated through AD |
| File-backed zone | DNS zone stored as a local `.dns` file on the DNS server |
| Dynamic update | Allows DNS clients or domain members to register records automatically |
| Secure dynamic update | AD-integrated mode where only authorized domain members update records |
| Forwarder | Upstream DNS server used when local DNS cannot answer a query |
| Root hints | Public root DNS fallback path if no forwarder handles the query |
| Client DNS setting | Interface-level setting that tells a host which DNS server to query |
| Split brain risk | Same internal and external DNS name can cause wrong answers if zones are not planned |
| First rule | DNS must point to the internal DNS server before domain join, AD lookup, or internal name resolution will work reliably |

# Windows_Server_DNS_Setup_Planning_Table
| Item | Example | Decision |
|---|---|---|
| DNS server name | `DC1` or `DNS1` | `<server-name>` |
| DNS server IP | `10.10.10.10` | `<dns-server-ip>` |
| Interface alias | `Ethernet` | `<interface-alias>` |
| Forward lookup zone | `corp.local` | `<zone-name>` |
| Reverse lookup network | `10.10.10.0/24` | `<reverse-network-id>` |
| First test host | `srv1.corp.local` | `<host-record-name>` |
| First test host IP | `10.10.10.20` | `<host-ip>` |
| Upstream forwarder 1 | `1.1.1.1` or internal upstream | `<forwarder-1>` |
| Upstream forwarder 2 | `8.8.8.8` or internal upstream | `<forwarder-2>` |
| AD integrated? | Yes if domain controller | `Yes/No` |
| Dynamic updates | Secure for AD zones, none/manual for standalone zones | `<dynamic-update-mode>` |

# Windows_Server_DNS_Setup_Configuration_Checklist
| Step | Task | Device | PowerShell | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm server identity | DNS Server | `hostname` | Correct server name is shown |
| 2 | Confirm IP configuration | DNS Server | `Get-NetIPConfiguration` | Static IP, gateway, and current DNS settings are visible |
| 3 | Confirm interface alias | DNS Server | `Get-NetAdapter` | Correct interface alias is known |
| 4 | Confirm server has static IP | DNS Server | `Get-NetIPAddress -AddressFamily IPv4` | DNS server IP is stable and not DHCP dependent |
| 5 | Install DNS Server role | DNS Server | `Install-WindowsFeature -Name DNS -IncludeManagementTools` | DNS role and management tools install |
| 6 | Confirm DNS role installed | DNS Server | `Get-WindowsFeature DNS` | DNS shows installed |
| 7 | Confirm DNS service state | DNS Server | `Get-Service DNS` | DNS service exists and is running |
| 8 | Set server to use itself for DNS | DNS Server | `Set-DnsClientServerAddress -InterfaceAlias "<interface-alias>" -ServerAddresses "<dns-server-ip>"` | Server queries its own DNS service |
| 9 | Create AD-integrated forward zone if domain DNS | DNS Server/DC | `Add-DnsServerPrimaryZone -Name "<zone-name>" -ReplicationScope "Forest" -DynamicUpdate Secure -PassThru` | AD-integrated forward zone exists |
| 10 | Create file-backed forward zone if standalone DNS | DNS Server | `Add-DnsServerPrimaryZone -Name "<zone-name>" -ZoneFile "<zone-name>.dns" -PassThru` | File-backed forward zone exists |
| 11 | Create reverse lookup zone | DNS Server | `Add-DnsServerPrimaryZone -NetworkID "<reverse-network-id>" -ReplicationScope "Forest"` | Reverse zone exists |
| 12 | Add server-level forwarders | DNS Server | `Add-DnsServerForwarder -IPAddress "<forwarder-1>","<forwarder-2>" -PassThru` | Unknown external queries can forward upstream |
| 13 | Add test A record | DNS Server | `Add-DnsServerResourceRecordA -Name "<host-record-name>" -ZoneName "<zone-name>" -IPv4Address "<host-ip>" -CreatePtr` | A record and PTR record are created |
| 14 | Verify zones | DNS Server | `Get-DnsServerZone` | Forward and reverse zones are listed |
| 15 | Verify records | DNS Server | `Get-DnsServerResourceRecord -ZoneName "<zone-name>"` | Test records are visible |
| 16 | Test local forward lookup | DNS Server | `Resolve-DnsName <host-record-name>.<zone-name> -Server <dns-server-ip>` | Name resolves to expected IP |
| 17 | Test local reverse lookup | DNS Server | `Resolve-DnsName <host-ip> -Server <dns-server-ip>` | IP resolves to expected hostname |
| 18 | Point a client at DNS server | Client | `Set-DnsClientServerAddress -InterfaceAlias "<client-interface>" -ServerAddresses "<dns-server-ip>"` | Client uses new DNS server |
| 19 | Test client lookup | Client | `Resolve-DnsName <host-record-name>.<zone-name>` | Client resolves internal hostname |
| 20 | Test external lookup through forwarder | Client | `Resolve-DnsName microsoft.com` | External name resolves |
| 21 | Flush stale client cache if needed | Client | `Clear-DnsClientCache` | Client cache is cleared |
| 22 | Confirm DNS event logs | DNS Server | `Get-WinEvent -LogName "DNS Server" -MaxEvents 20` | Recent DNS service events are visible |

# Windows_Server_DNS_Setup_AD_Integrated_Skeleton
```powershell
# Run in elevated PowerShell on the DNS server or domain controller.

$ZoneName = "corp.local"
$ReverseNetworkId = "10.10.10.0/24"
$DnsServerIp = "10.10.10.10"
$InterfaceAlias = "Ethernet"
$Forwarders = @("1.1.1.1", "8.8.8.8")

Install-WindowsFeature -Name DNS -IncludeManagementTools

Set-DnsClientServerAddress `
  -InterfaceAlias $InterfaceAlias `
  -ServerAddresses $DnsServerIp

Add-DnsServerPrimaryZone `
  -Name $ZoneName `
  -ReplicationScope "Forest" `
  -DynamicUpdate Secure `
  -PassThru

Add-DnsServerPrimaryZone `
  -NetworkID $ReverseNetworkId `
  -ReplicationScope "Forest" `
  -PassThru

Add-DnsServerForwarder `
  -IPAddress $Forwarders `
  -PassThru
```

# Windows_Server_DNS_Setup_File_Backed_Skeleton
```powershell
# Use this for standalone DNS where the zone is not stored in Active Directory.

$ZoneName = "lab.local"
$ReverseNetworkId = "10.10.10.0/24"
$DnsServerIp = "10.10.10.10"
$InterfaceAlias = "Ethernet"
$Forwarders = @("1.1.1.1", "8.8.8.8")

Install-WindowsFeature -Name DNS -IncludeManagementTools

Set-DnsClientServerAddress `
  -InterfaceAlias $InterfaceAlias `
  -ServerAddresses $DnsServerIp

Add-DnsServerPrimaryZone `
  -Name $ZoneName `
  -ZoneFile "$ZoneName.dns" `
  -PassThru

Add-DnsServerPrimaryZone `
  -NetworkID $ReverseNetworkId `
  -ZoneFile "10.10.10.rev.dns" `
  -PassThru

Add-DnsServerForwarder `
  -IPAddress $Forwarders `
  -PassThru
```

# Windows_Server_DNS_Setup_Record_Creation_Skeleton
```powershell
$ZoneName = "corp.local"
$HostName = "srv1"
$HostIp = "10.10.10.20"

Add-DnsServerResourceRecordA `
  -Name $HostName `
  -ZoneName $ZoneName `
  -IPv4Address $HostIp `
  -CreatePtr `
  -TimeToLive 01:00:00 `
  -PassThru

Get-DnsServerResourceRecord -ZoneName $ZoneName -Name $HostName
Resolve-DnsName "$HostName.$ZoneName"
Resolve-DnsName $HostIp
```

# Windows_Server_DNS_Setup_Client_DNS_Skeleton
```powershell
# Run on Windows client or member server.

$ClientInterfaceAlias = "Ethernet"
$DnsServerIp = "10.10.10.10"

Set-DnsClientServerAddress `
  -InterfaceAlias $ClientInterfaceAlias `
  -ServerAddresses $DnsServerIp

Clear-DnsClientCache

Get-DnsClientServerAddress -InterfaceAlias $ClientInterfaceAlias
Resolve-DnsName srv1.corp.local
Resolve-DnsName microsoft.com
```

# Windows_Server_DNS_Setup_Verification_Commands
| Check | Device | Command | Good Output |
|---|---|---|---|
| DNS role installed | DNS Server | `Get-WindowsFeature DNS` | Install state is installed |
| DNS service running | DNS Server | `Get-Service DNS` | Status is Running |
| DNS server listening | DNS Server | `Get-NetTCPConnection -LocalPort 53 -ErrorAction SilentlyContinue` | DNS listener is visible where applicable |
| UDP 53 check | DNS Server | `Get-NetUDPEndpoint -LocalPort 53 -ErrorAction SilentlyContinue` | UDP endpoint exists |
| Server DNS client points to itself | DNS Server | `Get-DnsClientServerAddress -InterfaceAlias "<interface-alias>"` | DNS server IP is listed |
| Zones visible | DNS Server | `Get-DnsServerZone` | Forward and reverse zones appear |
| Forward records visible | DNS Server | `Get-DnsServerResourceRecord -ZoneName "<zone-name>"` | A, SOA, NS, and other records appear |
| A record lookup | DNS Server or Client | `Resolve-DnsName <host>.<zone-name> -Server <dns-server-ip>` | Expected IPv4 address returned |
| PTR lookup | DNS Server or Client | `Resolve-DnsName <host-ip> -Server <dns-server-ip>` | Expected hostname returned |
| External forwarding | DNS Server or Client | `Resolve-DnsName microsoft.com -Server <dns-server-ip>` | External record resolves |
| Forwarders visible | DNS Server | `Get-DnsServerForwarder` | Configured upstream DNS servers listed |
| DNS logs readable | DNS Server | `Get-WinEvent -LogName "DNS Server" -MaxEvents 20` | DNS service events appear |
| Client DNS setting | Client | `Get-DnsClientServerAddress` | Client points to internal DNS server |
| Client cache clear | Client | `Clear-DnsClientCache` | Command completes without error |
| Name resolution path | Client | `nslookup <host>.<zone-name> <dns-server-ip>` | DNS server returns expected answer |

# Windows_Server_DNS_Setup_Rollback
| Step | Task | Device | PowerShell | Expected Result |
|---:|---|---|---|---|
| 1 | Export visible zone records before destructive rollback | DNS Server | `Get-DnsServerResourceRecord -ZoneName "<zone-name>" | Export-Csv .\dns-records-backup.csv -NoTypeInformation` | Basic record backup exists |
| 2 | Remove test A record | DNS Server | `Remove-DnsServerResourceRecord -ZoneName "<zone-name>" -Name "<host-record-name>" -RRType "A"` | Test A record removed |
| 3 | Remove forward lookup zone if lab-only | DNS Server | `Remove-DnsServerZone -Name "<zone-name>" -Force` | Forward zone removed |
| 4 | Remove reverse lookup zone if lab-only | DNS Server | `Remove-DnsServerZone -Name "<reverse-zone-name>" -Force` | Reverse zone removed |
| 5 | Remove forwarders if lab-only | DNS Server | `Remove-DnsServerForwarder -IPAddress "<forwarder-1>","<forwarder-2>" -Force` | Forwarders removed |
| 6 | Reset client DNS back to DHCP | Client | `Set-DnsClientServerAddress -InterfaceAlias "<client-interface>" -ResetServerAddresses` | Client no longer pins manual DNS servers |
| 7 | Reset server DNS client if appropriate | DNS Server | `Set-DnsClientServerAddress -InterfaceAlias "<interface-alias>" -ResetServerAddresses` | Interface DNS settings reset |
| 8 | Remove DNS role only if server is not needed for AD/domain DNS | DNS Server | `Uninstall-WindowsFeature -Name DNS` | DNS role removed |
| 9 | Reboot only if required | DNS Server | `Restart-Computer` | Server restarts if uninstall or dependency requires it |

# Windows_Server_DNS_Setup_Failure_Checks
| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| DNS role install fails | PowerShell not elevated or missing source files | Confirm admin shell and run `Get-WindowsFeature DNS` | Reopen PowerShell as admin and rerun install |
| DNS Manager missing | Management tools not installed | Check install command | Reinstall with `Install-WindowsFeature -Name DNS -IncludeManagementTools` |
| DNS service not running | Service stopped or role install incomplete | `Get-Service DNS` | `Start-Service DNS` or reinstall role |
| Server cannot resolve its own zone | Zone missing or server DNS client points elsewhere | `Get-DnsServerZone`; `Get-DnsClientServerAddress` | Create zone and point server DNS to itself |
| Client cannot resolve internal names | Client points to public DNS instead of internal DNS | `Get-DnsClientServerAddress` on client | Set client DNS to internal DNS server |
| A record resolves but PTR fails | Reverse zone missing or PTR not created | `Get-DnsServerZone`; `Resolve-DnsName <ip>` | Create reverse zone and PTR record |
| `-CreatePtr` does not create PTR | Reverse zone missing | Check reverse lookup zone exists | Create reverse zone first, then recreate PTR |
| External names fail | No forwarder or root-hint path blocked | `Get-DnsServerForwarder`; test external lookup | Add valid forwarders |
| Zone created but dynamic clients do not register | Dynamic update mode wrong or client not domain joined | Check zone dynamic update settings | Use secure updates for AD-integrated domain DNS |
| Domain join fails by DNS name | Client cannot locate domain controller SRV records | `Resolve-DnsName _ldap._tcp.dc._msdcs.<domain>` | Point client to AD DNS server, not public DNS |
| Name resolves to old IP | Stale DNS record or client cache | `Resolve-DnsName`; `Clear-DnsClientCache` | Update record and clear DNS cache |
| Duplicate records exist | Old static record plus dynamic record | `Get-DnsServerResourceRecord` | Remove stale duplicate |
| Queries timeout | Firewall blocks TCP/UDP 53 | Test from client; check Windows Firewall | Permit DNS traffic to server |
| DNS answers wrong zone | Split DNS or overlapping zone issue | Check zone names and search suffix | Correct client suffix or zone design |
| Public domain breaks internally | Internal zone shadows public DNS zone | Check whether internal zone matches public domain | Add required internal records or redesign split DNS |
| Forwarder order wrong | Preferred forwarder not first | `Get-DnsServerForwarder` | Remove/re-add or set forwarder order intentionally |

# Windows_Server_DNS_Setup_Related_Labs
| Lab               | Relationship                                  |
| ----------------- | --------------------------------------------- |
| `windows-dns-001` | Install DNS Server role and management tools  |
| `windows-dns-002` | Create forward lookup zone                    |
| `windows-dns-003` | Create reverse lookup zone and PTR records    |
| `windows-dns-004` | Add A records and validate resolution         |
| `windows-dns-005` | Configure DNS forwarders                      |
| `windows-dns-006` | Point Windows clients at internal DNS         |
| `windows-dns-007` | Troubleshoot internal name resolution         |
| `windows-dns-008` | Troubleshoot reverse lookup and stale records |