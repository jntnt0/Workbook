16_Configure_PXE_Boot_DHCP_Options.md
# 16_Configure_PXE_Boot_DHCP_Options

# Configure_PXE_Boot_DHCP_Options_Index
16_Configure_PXE_Boot_DHCP_Options.md
Configure_PXE_Boot_DHCP_Options
Configure_PXE_Boot_DHCP_Options_Source_Basis
Configure_PXE_Boot_DHCP_Options_Mental_Model
Configure_PXE_Boot_DHCP_Options_Design_Decision_Table
Configure_PXE_Boot_DHCP_Options_Planning_Table
Configure_PXE_Boot_DHCP_Options_Configuration_Checklist
Configure_PXE_Boot_DHCP_Options_Precheck_Skeleton
Configure_PXE_Boot_DHCP_Options_Basic_Scope_Options_66_67_Skeleton
Configure_PXE_Boot_DHCP_Options_Option_60_Skeleton
Configure_PXE_Boot_DHCP_Options_UEFI_BIOS_Policy_Skeleton
Configure_PXE_Boot_DHCP_Options_IP_Helper_Design_Skeleton
Configure_PXE_Boot_DHCP_Options_Client_Test_Skeleton
Configure_PXE_Boot_DHCP_Options_Verification_Commands
Configure_PXE_Boot_DHCP_Options_Rollback
Configure_PXE_Boot_DHCP_Options_Failure_Checks
Configure_PXE_Boot_DHCP_Options_Related_Labs

# Configure_PXE_Boot_DHCP_Options_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Set-DhcpServerv4OptionValue | Setting DHCPv4 option values at server, scope, reservation, vendor class, user class, and policy level |
| Microsoft Learn | Get-DhcpServerv4OptionValue | Validating DHCPv4 option values at server, scope, reservation, class, and policy level |
| Microsoft Learn | Add-DhcpServerv4OptionDefinition | Creating missing DHCP option definitions before assigning values |
| Microsoft Learn | Get-DhcpServerv4OptionDefinition | Verifying standard and vendor-specific option definitions |
| Microsoft Learn | Remove-DhcpServerv4OptionValue | Removing incorrect PXE option values during rollback |
| Windows Server DHCP operational practice | DHCP options 60, 66, 67 | PXE vendor class identifier, boot server host name, and bootfile name |
| PXE/WDS/MDT/SCCM operational practice | BIOS and UEFI boot file selection | Avoiding one static bootfile for mixed firmware environments |
| Network design practice | DHCP relay / IP helper to DHCP and PXE servers | Cleaner routed-network PXE design when DHCP and PXE services are separate |

# Configure_PXE_Boot_DHCP_Options_Mental_Model
| Concept | Operational Meaning |
|---|---|
| PXE boot | Client firmware uses DHCP-like discovery to locate a boot server and network boot file |
| Option 60 | Vendor Class Identifier, commonly set to `PXEClient` when DHCP and PXE/WDS are on the same server |
| Option 66 | Boot Server Host Name, usually the PXE/TFTP/WDS server name or IP |
| Option 67 | Bootfile Name, the boot program path returned to the PXE client |
| BIOS boot file | Legacy BIOS clients usually need a different boot program than UEFI clients |
| UEFI boot file | UEFI clients usually need an EFI boot program, for example `wdsmgfw.efi` in WDS environments |
| Static 66/67 risk | One static option 67 can break mixed BIOS and UEFI booting |
| DHCP policy design | Use DHCP policies or vendor-class matching when different client firmware types need different bootfiles |
| IP helper design | On routed networks, relay DHCP/PXE broadcasts to the DHCP server and PXE server instead of forcing all logic into options |
| Same-server DHCP + PXE | Option 60 may be needed so PXE clients know the DHCP server also provides PXE service |
| Separate DHCP and PXE servers | Usually avoid option 60 on the DHCP server and relay traffic to the PXE server |
| TFTP | PXE boot files are commonly retrieved over TFTP after DHCP/PXE discovery |
| First rule | Do not set global option 67 unless every PXE client uses the same boot program |
| Second rule | For mixed BIOS/UEFI, use DHCP policies or IP helper design instead of a single static bootfile |
| Third rule | For routed PXE, verify relay/IP helper, firewall, DHCP scope, PXE service, and TFTP before changing boot options |

# Configure_PXE_Boot_DHCP_Options_Design_Decision_Table
| Scenario | Recommended Design | Use Option 60? | Use Option 66/67? | Notes |
|---|---|---:|---:|---|
| DHCP and WDS/PXE on same Windows server | Same-server PXE design | Yes, often | Maybe | Option 60 tells PXE clients the DHCP server is also a PXE service |
| DHCP server separate from PXE server on same VLAN | PXE server listens directly | No | Maybe | PXE server can answer PXE discovery directly |
| DHCP server separate from PXE server across routed VLANs | IP helper to DHCP and PXE server | No | Usually no | Cleaner than hardcoding 66/67 |
| Simple lab with one firmware type | Scope-level 66/67 | No if separate server | Yes | Acceptable for homogeneous lab clients |
| Mixed BIOS and UEFI clients | DHCP policies or PXE server logic | Depends | Only by policy | Avoid one static option 67 |
| SCCM/MECM PXE-enabled DP | IP helper to DHCP and PXE-enabled DP | Usually no | Usually no | Let PXE service decide boot file |
| WDS with mixed firmware | IP helper or policy split | Depends | Policy only | Static option 67 can send wrong boot file |
| Unknown vendor devices | Packet capture first | No | Not yet | Confirm what vendor class and architecture the device sends |
| PXE only failing on remote VLAN | Relay/firewall issue likely | No | Not first | Fix network path before DHCP option changes |

# Configure_PXE_Boot_DHCP_Options_Planning_Table
| Item | Example | Decision |
|---|---|---|
| DHCP server | `DHCP1` | `<dhcp-server>` |
| PXE/WDS server | `PXE1` | `<pxe-server>` |
| PXE/WDS server IP | `10.10.10.30` | `<pxe-server-ip>` |
| PXE/WDS server FQDN | `pxe1.corp.local` | `<pxe-server-fqdn>` |
| DHCP scope ID | `10.10.20.0` | `<scope-id>` |
| Client subnet | `10.10.20.0/24` | `<client-subnet>` |
| Client VLAN | `VLAN20` | `<client-vlan>` |
| DHCP and PXE same server? | `No` | `<same-server-yes-no>` |
| Routed PXE? | `Yes` | `<routed-pxe-yes-no>` |
| Firmware mix | BIOS only / UEFI only / mixed | `<firmware-mix>` |
| BIOS bootfile | `boot\x64\wdsnbp.com` | `<bios-bootfile>` |
| UEFI x64 bootfile | `boot\x64\wdsmgfw.efi` | `<uefi-x64-bootfile>` |
| Option 60 value | `PXEClient` | `<option-60-value>` |
| Option 66 value | `pxe1.corp.local` | `<option-66-value>` |
| Option 67 value | `boot\x64\wdsmgfw.efi` | `<option-67-value>` |
| BIOS policy name | `PXE-BIOS` | `<bios-policy-name>` |
| UEFI x64 policy name | `PXE-UEFI-x64` | `<uefi-policy-name>` |
| BIOS vendor/arch match | `PXEClient:Arch:00000` | `<bios-vendor-match>` |
| UEFI x64 vendor/arch match | `PXEClient:Arch:00007` or `PXEClient:Arch:00009` | `<uefi-vendor-match>` |
| Relay device | `CoreSW1` | `<relay-device>` |
| Relay interface | `Vlan20` | `<relay-interface>` |
| DHCP relay target | `10.10.10.20` | `<dhcp-server-ip>` |
| PXE relay target | `10.10.10.30` | `<pxe-server-ip>` |
| Test client | `PXE-TEST-01` | `<test-client>` |
| Evidence path | `C:\DHCPPrep\pxe-dhcp-options` | `<evidence-path>` |

# Configure_PXE_Boot_DHCP_Options_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify PXE design | Operator | Confirm same-server, separate-server, same-VLAN, or routed design | Correct configuration path is chosen |
| 2 | Confirm firmware mix | Operator | BIOS only, UEFI only, or mixed | Option 67 strategy is known |
| 3 | Create evidence folder | DHCP server | `New-Item -ItemType Directory -Force -Path C:\DHCPPrep\pxe-dhcp-options` | Evidence path exists |
| 4 | Import DHCP module | DHCP server | `Import-Module DhcpServer` | DHCP cmdlets are available |
| 5 | Confirm DHCP service | DHCP server | `Get-Service DHCPServer` | DHCP service is running |
| 6 | Confirm DHCP authorization | DHCP server or DC | `Get-DhcpServerInDC` | Expected DHCP server is authorized |
| 7 | Confirm scope exists | DHCP server | `Get-DhcpServerv4Scope -ComputerName <dhcp-server> -ScopeId <scope-id>` | Scope exists |
| 8 | Capture existing options | DHCP server | `Get-DhcpServerv4OptionValue -ComputerName <dhcp-server> -ScopeId <scope-id> -All` | Existing option state is preserved |
| 9 | Verify PXE option definitions | DHCP server | `Get-DhcpServerv4OptionDefinition -ComputerName <dhcp-server> -OptionId 60,66,67` | Option definitions are present or missing definitions are identified |
| 10 | Add option 60 definition if missing | DHCP server | `Add-DhcpServerv4OptionDefinition -ComputerName <dhcp-server> -OptionId 60 -Name PXEClient -Type String` | Option 60 can be assigned |
| 11 | Configure option 60 only if DHCP and PXE are same server | DHCP server | `Set-DhcpServerv4OptionValue -ComputerName <dhcp-server> -OptionId 60 -Value PXEClient` | Server-wide option 60 is set only for correct design |
| 12 | Configure basic scope option 66 | DHCP server | `Set-DhcpServerv4OptionValue -ComputerName <dhcp-server> -ScopeId <scope-id> -OptionId 66 -Value <pxe-server-fqdn>` | Clients receive PXE boot server value |
| 13 | Configure basic scope option 67 | DHCP server | `Set-DhcpServerv4OptionValue -ComputerName <dhcp-server> -ScopeId <scope-id> -OptionId 67 -Value <bootfile>` | Clients receive bootfile value |
| 14 | For mixed firmware, avoid one static 67 | DHCP server | Use policy-specific option 67 values | BIOS and UEFI clients receive different boot files |
| 15 | Configure DHCP policy for BIOS PXE if required | DHCP server | Match `PXEClient:Arch:00000` or confirmed vendor-class string | BIOS clients receive BIOS bootfile |
| 16 | Configure DHCP policy for UEFI PXE if required | DHCP server | Match `PXEClient:Arch:00007` or confirmed vendor-class string | UEFI clients receive EFI bootfile |
| 17 | For routed PXE, verify IP helper | Router / L3 switch | `ip helper-address <dhcp-server-ip>` and `ip helper-address <pxe-server-ip>` | DHCP and PXE discovery reaches correct services |
| 18 | Verify firewall path | Firewall / server | UDP 67, 68, 69, and PXE service requirements | DHCP/PXE/TFTP traffic is not blocked |
| 19 | Renew or reboot PXE client | Client | PXE boot test from firmware boot menu | Client receives DHCP lease and PXE boot response |
| 20 | Validate DHCP lease | DHCP server | `Get-DhcpServerv4Lease -ComputerName <dhcp-server> -ScopeId <scope-id>` | PXE client lease appears |
| 21 | Validate effective options | DHCP server | `Get-DhcpServerv4OptionValue -ComputerName <dhcp-server> -ScopeId <scope-id> -All` | PXE options match design |
| 22 | Capture DHCP/PXE events | DHCP/PXE server | Event Viewer and DHCP audit logs | Boot attempt evidence exists |
| 23 | Remove lab-only PXE options if needed | DHCP server | `Remove-DhcpServerv4OptionValue -ScopeId <scope-id> -OptionId 66,67` | Test-only options are removed |

# Configure_PXE_Boot_DHCP_Options_Precheck_Skeleton
~~~powershell
# Run on the DHCP server or a management host with RSAT DHCP tools.

$DhcpServer = "DHCP1"
$ScopeId = "10.10.20.0"
$EvidencePath = "C:\DHCPPrep\pxe-dhcp-options"

New-Item -ItemType Directory -Force -Path $EvidencePath

Import-Module DhcpServer

# Confirm DHCP service.
Get-Service DHCPServer |
  Tee-Object "$EvidencePath\precheck-dhcp-service.txt"

# Confirm DHCP authorization.
Get-DhcpServerInDC |
  Tee-Object "$EvidencePath\precheck-authorized-dhcp-servers.txt"

# Confirm scope exists.
Get-DhcpServerv4Scope `
  -ComputerName $DhcpServer `
  -ScopeId $ScopeId |
  Format-List * |
  Tee-Object "$EvidencePath\precheck-scope-details.txt"

# Capture current scope options.
Get-DhcpServerv4OptionValue `
  -ComputerName $DhcpServer `
  -ScopeId $ScopeId `
  -All `
  -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\precheck-scope-options-all.txt"

# Capture server-level options.
Get-DhcpServerv4OptionValue `
  -ComputerName $DhcpServer `
  -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\precheck-server-options.txt"

# Verify option definitions.
Get-DhcpServerv4OptionDefinition `
  -ComputerName $DhcpServer `
  -OptionId 60,66,67 `
  -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\precheck-option-definitions-60-66-67.txt"

# Capture existing DHCP policies in the scope.
Get-DhcpServerv4Policy `
  -ComputerName $DhcpServer `
  -ScopeId $ScopeId `
  -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\precheck-scope-policies.txt"

# Capture current leases.
Get-DhcpServerv4Lease `
  -ComputerName $DhcpServer `
  -ScopeId $ScopeId `
  -AllLeases `
  -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\precheck-scope-leases.txt"

# DHCP server event logs.
Get-WinEvent `
  -LogName "Microsoft-Windows-Dhcp-Server/Operational" `
  -MaxEvents 100 `
  -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\precheck-dhcp-server-operational-events.txt"
~~~

# Configure_PXE_Boot_DHCP_Options_Basic_Scope_Options_66_67_Skeleton
~~~powershell
# Use this only for a simple lab or homogeneous PXE client firmware type.
# Do not use one static option 67 for mixed BIOS and UEFI clients.

$DhcpServer = "DHCP1"
$ScopeId = "10.10.20.0"
$PxeServer = "pxe1.corp.local"

# Choose one bootfile based on the actual firmware design.
# BIOS example:
$BiosBootFile = "boot\x64\wdsnbp.com"

# UEFI x64 example:
$UefiX64BootFile = "boot\x64\wdsmgfw.efi"

# Select the bootfile for this scope.
$BootFile = $UefiX64BootFile

Import-Module DhcpServer

# Verify option definitions.
Get-DhcpServerv4OptionDefinition `
  -ComputerName $DhcpServer `
  -OptionId 66,67

# Set option 66, boot server host name.
Set-DhcpServerv4OptionValue `
  -ComputerName $DhcpServer `
  -ScopeId $ScopeId `
  -OptionId 66 `
  -Value $PxeServer `
  -PassThru

# Set option 67, bootfile name.
Set-DhcpServerv4OptionValue `
  -ComputerName $DhcpServer `
  -ScopeId $ScopeId `
  -OptionId 67 `
  -Value $BootFile `
  -PassThru

# Validate options.
Get-DhcpServerv4OptionValue `
  -ComputerName $DhcpServer `
  -ScopeId $ScopeId `
  -OptionId 66,67

Get-DhcpServerv4OptionValue `
  -ComputerName $DhcpServer `
  -ScopeId $ScopeId `
  -All
~~~

# Configure_PXE_Boot_DHCP_Options_Option_60_Skeleton
~~~powershell
# Use option 60 only when the DHCP server also provides PXE service.
# Do not set option 60 on a separate DHCP server just because PXE exists somewhere else.

$DhcpServer = "DHCP1"
$Option60Value = "PXEClient"

Import-Module DhcpServer

# Check whether option 60 definition exists.
$Option60 = Get-DhcpServerv4OptionDefinition `
  -ComputerName $DhcpServer `
  -OptionId 60 `
  -ErrorAction SilentlyContinue

# Add option 60 definition only if missing.
if (-not $Option60) {
  Add-DhcpServerv4OptionDefinition `
    -ComputerName $DhcpServer `
    -Name "PXEClient" `
    -OptionId 60 `
    -Type String `
    -Description "PXE Client Vendor Class Identifier" `
    -PassThru
}

# Set server-level option 60.
Set-DhcpServerv4OptionValue `
  -ComputerName $DhcpServer `
  -OptionId 60 `
  -Value $Option60Value `
  -PassThru

# Validate server-level option 60.
Get-DhcpServerv4OptionValue `
  -ComputerName $DhcpServer `
  -OptionId 60
~~~

# Configure_PXE_Boot_DHCP_Options_UEFI_BIOS_Policy_Skeleton
~~~powershell
# Use this when BIOS and UEFI clients need different boot files.
# Confirm the exact vendor class / architecture strings in your environment before relying on these examples.
# Common examples:
# BIOS: PXEClient:Arch:00000
# UEFI x64 examples seen in the field: PXEClient:Arch:00007 or PXEClient:Arch:00009

$DhcpServer = "DHCP1"
$ScopeId = "10.10.20.0"
$PxeServer = "pxe1.corp.local"

$BiosPolicyName = "PXE-BIOS"
$UefiPolicyName = "PXE-UEFI-x64"

$BiosVendorMatch = "PXEClient:Arch:00000"
$UefiVendorMatch = "PXEClient:Arch:00007"

$BiosBootFile = "boot\x64\wdsnbp.com"
$UefiBootFile = "boot\x64\wdsmgfw.efi"

Import-Module DhcpServer

# Create BIOS policy disabled first.
Add-DhcpServerv4Policy `
  -ComputerName $DhcpServer `
  -ScopeId $ScopeId `
  -Name $BiosPolicyName `
  -Description "PXE BIOS clients receive BIOS boot program" `
  -Condition OR `
  -VendorClass EQ,$BiosVendorMatch `
  -ProcessingOrder 1 `
  -Enabled $false `
  -PassThru

# Create UEFI policy disabled first.
Add-DhcpServerv4Policy `
  -ComputerName $DhcpServer `
  -ScopeId $ScopeId `
  -Name $UefiPolicyName `
  -Description "PXE UEFI x64 clients receive EFI boot program" `
  -Condition OR `
  -VendorClass EQ,$UefiVendorMatch `
  -ProcessingOrder 2 `
  -Enabled $false `
  -PassThru

# Set BIOS PXE options on BIOS policy.
Set-DhcpServerv4OptionValue `
  -ComputerName $DhcpServer `
  -ScopeId $ScopeId `
  -PolicyName $BiosPolicyName `
  -OptionId 66 `
  -Value $PxeServer `
  -PassThru

Set-DhcpServerv4OptionValue `
  -ComputerName $DhcpServer `
  -ScopeId $ScopeId `
  -PolicyName $BiosPolicyName `
  -OptionId 67 `
  -Value $BiosBootFile `
  -PassThru

# Set UEFI PXE options on UEFI policy.
Set-DhcpServerv4OptionValue `
  -ComputerName $DhcpServer `
  -ScopeId $ScopeId `
  -PolicyName $UefiPolicyName `
  -OptionId 66 `
  -Value $PxeServer `
  -PassThru

Set-DhcpServerv4OptionValue `
  -ComputerName $DhcpServer `
  -ScopeId $ScopeId `
  -PolicyName $UefiPolicyName `
  -OptionId 67 `
  -Value $UefiBootFile `
  -PassThru

# Enable policies after validation.
Set-DhcpServerv4Policy `
  -ComputerName $DhcpServer `
  -ScopeId $ScopeId `
  -Name $BiosPolicyName `
  -Enabled $true `
  -ProcessingOrder 1 `
  -PassThru

Set-DhcpServerv4Policy `
  -ComputerName $DhcpServer `
  -ScopeId $ScopeId `
  -Name $UefiPolicyName `
  -Enabled $true `
  -ProcessingOrder 2 `
  -PassThru

# Validate policies and policy option values.
Get-DhcpServerv4Policy `
  -ComputerName $DhcpServer `
  -ScopeId $ScopeId |
  Sort-Object ProcessingOrder |
  Format-Table Name,Enabled,ProcessingOrder,Condition,Description -AutoSize

Get-DhcpServerv4OptionValue `
  -ComputerName $DhcpServer `
  -ScopeId $ScopeId `
  -PolicyName $BiosPolicyName `
  -OptionId 66,67

Get-DhcpServerv4OptionValue `
  -ComputerName $DhcpServer `
  -ScopeId $ScopeId `
  -PolicyName $UefiPolicyName `
  -OptionId 66,67
~~~

# Configure_PXE_Boot_DHCP_Options_IP_Helper_Design_Skeleton
~~~text
# Use this design when the PXE client VLAN is routed and DHCP/PXE servers are not on that VLAN.
# This is often cleaner than static DHCP options 66/67.
# Example shown with Cisco IOS-style syntax.

# =========================================================
# DHCP and PXE server are separate systems.
# Forward client broadcast traffic to both.
# =========================================================

configure terminal
interface Vlan20
 description Client PXE subnet
 ip address 10.10.20.1 255.255.255.0
 ip helper-address 10.10.10.20
 ip helper-address 10.10.10.30
 no shutdown
end
write memory

# 10.10.10.20 = DHCP server
# 10.10.10.30 = PXE/WDS/SCCM/MDT PXE server

# =========================================================
# Verification.
# =========================================================

show running-config interface Vlan20
show ip interface Vlan20
show ip route 10.10.10.20
show ip route 10.10.10.30
show access-lists
show logging | include DHCP|dhcp|PXE|pxe|helper|UDP|67|68|69

# =========================================================
# Firewall/path checks to validate outside router config.
# =========================================================

# DHCP client/server uses UDP 67 and 68.
# PXE/TFTP commonly needs UDP 69 plus dynamic data flow behavior.
# WDS/SCCM environments may require additional service-specific firewall allowances.
~~~

# Configure_PXE_Boot_DHCP_Options_Client_Test_Skeleton
~~~powershell
# Run Windows-side checks before rebooting into PXE.
# PXE boot itself occurs before Windows loads, so final validation happens at firmware/network boot screen.

$EvidencePath = "C:\DHCPPrep\pxe-dhcp-options"
$DhcpServer = "DHCP1"
$ScopeId = "10.10.20.0"
$PxeServer = "pxe1.corp.local"

New-Item -ItemType Directory -Force -Path $EvidencePath

# Capture current Windows client DHCP state.
hostname |
  Tee-Object "$EvidencePath\client-hostname.txt"

ipconfig /all |
  Tee-Object "$EvidencePath\client-ipconfig-before.txt"

Get-NetIPConfiguration |
  Tee-Object "$EvidencePath\client-netipconfiguration-before.txt"

# Renew the client lease to verify normal DHCP still works.
ipconfig /release |
  Tee-Object "$EvidencePath\client-release.txt"

ipconfig /renew |
  Tee-Object "$EvidencePath\client-renew.txt"

ipconfig /all |
  Tee-Object "$EvidencePath\client-ipconfig-after-renew.txt"

# Resolve PXE server name.
Resolve-DnsName $PxeServer -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\client-resolve-pxe-server.txt"

# Basic reachability to PXE server by name.
Test-NetConnection $PxeServer |
  Tee-Object "$EvidencePath\client-test-pxe-server-name.txt"

# Optional TFTP test if the Windows TFTP Client feature is installed.
# This is environment-dependent and the boot file path must exist on the PXE/TFTP server.
# Enable-WindowsOptionalFeature -Online -FeatureName TFTP -NoRestart
# tftp -i $PxeServer GET boot\x64\wdsmgfw.efi C:\DHCPPrep\pxe-dhcp-options\wdsmgfw.efi

# Final test:
# 1. Reboot physical or VM client.
# 2. Select network/PXE boot.
# 3. Confirm DHCP address is received.
# 4. Confirm PXE server responds.
# 5. Confirm bootfile downloads.
# 6. Confirm WDS/MDT/SCCM boot menu or task sequence starts.
~~~

# Configure_PXE_Boot_DHCP_Options_Verification_Commands
~~~powershell
# DHCP server verification.

Import-Module DhcpServer

Get-Service DHCPServer

Get-DhcpServerInDC

Get-DhcpServerv4Scope `
  -ComputerName <dhcp-server> `
  -ScopeId <scope-id>

Get-DhcpServerv4OptionDefinition `
  -ComputerName <dhcp-server> `
  -OptionId 60,66,67

Get-DhcpServerv4OptionValue `
  -ComputerName <dhcp-server>

Get-DhcpServerv4OptionValue `
  -ComputerName <dhcp-server> `
  -ScopeId <scope-id> `
  -OptionId 66,67

Get-DhcpServerv4OptionValue `
  -ComputerName <dhcp-server> `
  -ScopeId <scope-id> `
  -All

Get-DhcpServerv4Policy `
  -ComputerName <dhcp-server> `
  -ScopeId <scope-id>

Get-DhcpServerv4OptionValue `
  -ComputerName <dhcp-server> `
  -ScopeId <scope-id> `
  -PolicyName <policy-name> `
  -OptionId 66,67

Get-DhcpServerv4Lease `
  -ComputerName <dhcp-server> `
  -ScopeId <scope-id> `
  -AllLeases

Get-WinEvent `
  -LogName "Microsoft-Windows-Dhcp-Server/Operational" `
  -MaxEvents 100 `
  -ErrorAction SilentlyContinue

Select-String `
  -Path "C:\Windows\System32\dhcp\DhcpSrvLog-*.log" `
  -Pattern "PXE","PXEClient","<client-mac>","<client-ip>" `
  -ErrorAction SilentlyContinue

# Client-side Windows checks before PXE reboot.

ipconfig /all
ipconfig /release
ipconfig /renew
ipconfig /all
Resolve-DnsName <pxe-server-fqdn>
Test-NetConnection <pxe-server-fqdn>

# Network device checks.

show running-config interface <client-vlan-svi>
show ip interface <client-vlan-svi>
show ip route <dhcp-server-ip>
show ip route <pxe-server-ip>
show access-lists
show logging | include DHCP|dhcp|PXE|pxe|helper|UDP|67|68|69
~~~

# Configure_PXE_Boot_DHCP_Options_Rollback
~~~powershell
# Run only the rollback matching the change being reversed.
# Capture current state first.

$DhcpServer = "DHCP1"
$ScopeId = "10.10.20.0"
$BiosPolicyName = "PXE-BIOS"
$UefiPolicyName = "PXE-UEFI-x64"
$EvidencePath = "C:\DHCPPrep\pxe-dhcp-options"

Import-Module DhcpServer
New-Item -ItemType Directory -Force -Path $EvidencePath

# Capture current server-level and scope-level option state.
Get-DhcpServerv4OptionValue `
  -ComputerName $DhcpServer `
  -ErrorAction SilentlyContinue |
  Export-Csv "$EvidencePath\rollback-server-options-before.csv" -NoTypeInformation

Get-DhcpServerv4OptionValue `
  -ComputerName $DhcpServer `
  -ScopeId $ScopeId `
  -All `
  -ErrorAction SilentlyContinue |
  Export-Csv "$EvidencePath\rollback-scope-options-before.csv" -NoTypeInformation

Get-DhcpServerv4Policy `
  -ComputerName $DhcpServer `
  -ScopeId $ScopeId `
  -ErrorAction SilentlyContinue |
  Export-Csv "$EvidencePath\rollback-policies-before.csv" -NoTypeInformation

# Safest rollback: remove scope-level PXE static options 66 and 67.
Remove-DhcpServerv4OptionValue `
  -ComputerName $DhcpServer `
  -ScopeId $ScopeId `
  -OptionId 66,67 `
  -ErrorAction SilentlyContinue `
  -PassThru

# Remove server-level option 60 only if it was incorrectly added.
# Do not remove if DHCP and PXE/WDS are intentionally on the same server.
# Remove-DhcpServerv4OptionValue `
#   -ComputerName $DhcpServer `
#   -OptionId 60 `
#   -ErrorAction SilentlyContinue `
#   -PassThru

# Disable PXE policies first.
Set-DhcpServerv4Policy `
  -ComputerName $DhcpServer `
  -ScopeId $ScopeId `
  -Name $BiosPolicyName `
  -Enabled $false `
  -ErrorAction SilentlyContinue `
  -PassThru

Set-DhcpServerv4Policy `
  -ComputerName $DhcpServer `
  -ScopeId $ScopeId `
  -Name $UefiPolicyName `
  -Enabled $false `
  -ErrorAction SilentlyContinue `
  -PassThru

# Optional: remove PXE policies completely after disabling.
# Remove-DhcpServerv4Policy `
#   -ComputerName $DhcpServer `
#   -ScopeId $ScopeId `
#   -Name $BiosPolicyName `
#   -Confirm

# Remove-DhcpServerv4Policy `
#   -ComputerName $DhcpServer `
#   -ScopeId $ScopeId `
#   -Name $UefiPolicyName `
#   -Confirm

# Validate rollback state.
Get-DhcpServerv4OptionValue `
  -ComputerName $DhcpServer `
  -ScopeId $ScopeId `
  -All `
  -ErrorAction SilentlyContinue

Get-DhcpServerv4Policy `
  -ComputerName $DhcpServer `
  -ScopeId $ScopeId `
  -ErrorAction SilentlyContinue
~~~

# Configure_PXE_Boot_DHCP_Options_Failure_Checks
| Symptom | Likely Cause | Check | Corrective Action |
|---|---|---|---|
| PXE client gets no IP | DHCP scope, relay, VLAN, or firewall issue | DHCP lease table, relay config, client VLAN | Fix DHCP reachability before PXE options |
| PXE client gets IP but no PXE response | PXE server not reachable or relay missing | IP helper, firewall, PXE service logs | Add helper to PXE server or fix PXE service |
| PXE client gets wrong bootfile | Static option 67 wrong for firmware type | Scope/policy option 67 values | Use BIOS/UEFI policy split or PXE server logic |
| BIOS client fails after UEFI option set | Option 67 points to EFI boot program | `Get-DhcpServerv4OptionValue -OptionId 67` | Use BIOS bootfile for BIOS clients |
| UEFI client fails after BIOS option set | Option 67 points to BIOS boot program | Scope/policy option 67 | Use `wdsmgfw.efi` or correct EFI bootfile |
| Mixed BIOS/UEFI clients behave inconsistently | One static option 67 is being used | DHCP options and client firmware inventory | Use DHCP policies or IP helper design |
| DHCP and WDS are on same server but PXE ignored | Option 60 missing or WDS DHCP port configuration wrong | Server-level option 60 and WDS/PXE config | Set option 60 only for same-server design |
| DHCP and PXE are separate but option 60 set | DHCP server is incorrectly advertising PXE | Server-level option 60 | Remove option 60 from DHCP server |
| Option value command fails | Option definition missing | `Get-DhcpServerv4OptionDefinition -OptionId 60,66,67` | Add missing option definition |
| Option 66 uses wrong format | UNC path or wrong host value used | Option 66 value | Use PXE/TFTP server host name or IP, not `\\server\share` |
| Option 67 uses wrong path | Boot file path not valid on PXE/TFTP server | PXE server boot folder and logs | Correct bootfile path |
| Remote VLAN PXE fails only | Relay/IP helper missing for PXE server | Router interface config | Add helper to both DHCP and PXE server when separate |
| Same VLAN PXE works but routed VLAN fails | Firewall or relay issue | Firewall, ACL, IP helper, logs | Permit DHCP/PXE/TFTP path |
| TFTP timeout | UDP 69 or dynamic TFTP flow blocked | Firewall and PXE server logs | Permit TFTP and required PXE server traffic |
| PXE starts but task sequence fails | Not a DHCP option issue | WDS/MDT/SCCM logs | Move to deployment service troubleshooting |
| Policy never matches | Vendor class string does not match real client | Packet capture or DHCP logs | Confirm actual vendor class/architecture and update policy |
| UEFI policy does not match some UEFI clients | Wrong architecture code | PXE packet capture | Add separate policies for observed architecture values |
| Removing options breaks lab boot | Rollback removed required static values | Rollback evidence and option values | Restore saved 66/67 values or use policy design |
| PXE boot loops | Wrong NBP or deployment server config | PXE server logs and bootfile | Correct boot program or PXE service configuration |

# Configure_PXE_Boot_DHCP_Options_Related_Labs
| Lab                                                        | Relationship                                                                    |
| ---------------------------------------------------------- | ------------------------------------------------------------------------------- |
| 02_Create_DHCPv4_Scope                                     | PXE options are usually configured on an existing IPv4 scope                    |
| 03_Configure_DHCPv4_Scope_Options                          | PXE boot options are specialized DHCP scope options                             |
| 04_Activate_Authorize_And_Verify_DHCP_Server               | PXE cannot work if DHCP service, authorization, or bindings are broken          |
| 07_Configure_DHCP_Relay_And_IP_Helper                      | Routed PXE often depends more on relay than on static 66/67                     |
| 11_Monitor_DHCP_Leases_Events_And_Scope_Utilization        | PXE troubleshooting uses leases, events, and DHCP audit logs                    |
| 12_Troubleshoot_DHCP_Client_Lease_Failures                 | PXE failures often start as normal DHCP lease failures                          |
| 14_Configure_DHCP_Policies_User_Classes_And_Vendor_Classes | Mixed BIOS/UEFI PXE designs may require DHCP policies and vendor-class matching |