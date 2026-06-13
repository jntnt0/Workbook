03_Create_DHCPv4_Scope.md
# Create_DHCPv4_Scope

# Create_DHCPv4_Scope_Index
03_Create_DHCPv4_Scope.md
Create_DHCPv4_Scope
Create_DHCPv4_Scope_Source_Basis
Create_DHCPv4_Scope_Mental_Model
Create_DHCPv4_Scope_Planning_Table
Create_DHCPv4_Scope_Configuration_Checklist
Create_DHCPv4_Scope_Precheck_Skeleton
Create_DHCPv4_Scope_Creation_Skeleton
Create_DHCPv4_Scope_State_Management_Skeleton
Create_DHCPv4_Scope_Validation_Skeleton
Create_DHCPv4_Scope_Event_Log_Skeleton
Create_DHCPv4_Scope_Verification_Commands
Create_DHCPv4_Scope_Rollback
Create_DHCPv4_Scope_Failure_Checks
Create_DHCPv4_Scope_Related_Labs

# Create_DHCPv4_Scope_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Add-DhcpServerv4Scope | Creating a DHCPv4 scope with start range, end range, subnet mask, state, and lease duration |
| Microsoft Learn | Get-DhcpServerv4Scope | Listing and validating DHCPv4 scopes |
| Microsoft Learn | Set-DhcpServerv4Scope | Modifying scope state, name, description, or lease duration |
| Microsoft Learn | Remove-DhcpServerv4Scope | Removing an incorrectly created DHCPv4 scope |
| Microsoft Learn | Get-DhcpServerv4ScopeStatistics | Checking utilization and scope statistics |
| Microsoft Learn | Get-DhcpServerv4Lease | Validating leases after clients begin using the scope |
| Microsoft Learn | Get-DhcpServerInDC | Confirming DHCP server authorization in Active Directory |
| Microsoft Learn | Get-Service | Confirming DHCP Server service state |
| Microsoft Learn | DHCP Server event logs | Reviewing scope creation, service, and authorization events |
| Windows Server operational practice | Subnet planning, gateway reservation, exclusions, and options sequencing | Creating a clean scope before adding options, exclusions, reservations, relay, or failover |

# Create_DHCPv4_Scope_Mental_Model
| Concept | Operational Meaning |
|---|---|
| DHCPv4 scope | IPv4 address pool that DHCP can lease to clients on a specific subnet |
| Scope ID | Network address of the subnet, such as `10.10.10.0` |
| Start range | First DHCP-leasable address in the scope |
| End range | Last DHCP-leasable address in the scope |
| Subnet mask | Defines the subnet boundary for the scope |
| Lease duration | How long a client can keep a DHCP lease before renewal |
| Scope state | Active scopes can issue leases; inactive scopes exist but do not lease addresses |
| Address pool | Total leasable range inside the scope before exclusions and reservations are applied |
| Exclusions | Addresses inside the scope range that DHCP should not lease |
| Reservations | Fixed client-to-IP mappings configured after the scope exists |
| Scope options | Gateway, DNS, DNS suffix, and other settings configured after scope creation |
| Authorization dependency | In AD environments, DHCP server should be authorized before clients rely on the scope |
| Relay dependency | Clients on remote subnets need DHCP relay or IP helper configured on the router or L3 switch |
| First rule | A DHCP scope should match exactly one subnet; do not create a scope until subnet, gateway, static ranges, and lease range are known |

# Create_DHCPv4_Scope_Planning_Table
| Item | Example | Decision |
|---|---|---|
| DHCP server name | `DHCP1` | `<dhcp-server-name>` |
| DHCP server FQDN | `DHCP1.corp.local` | `<dhcp-server-fqdn>` |
| DHCP server IP | `10.10.10.20` | `<dhcp-server-ip>` |
| Domain FQDN | `corp.local` | `<domain-fqdn>` |
| Scope name | `Corp-Users-VLAN10` | `<scope-name>` |
| Scope description | `User workstation DHCP scope for VLAN 10` | `<scope-description>` |
| Scope ID / subnet network | `10.10.10.0` | `<scope-id>` |
| Subnet mask | `255.255.255.0` | `<subnet-mask>` |
| CIDR prefix | `/24` | `<prefix-length>` |
| Start range | `10.10.10.50` | `<start-range>` |
| End range | `10.10.10.200` | `<end-range>` |
| Default gateway to reserve later | `10.10.10.1` | `<gateway-ip>` |
| Static infrastructure range | `10.10.10.1` to `10.10.10.49` | `<static-range>` |
| Printer/reservation range | `10.10.10.201` to `10.10.10.230` | `<reservation-range>` |
| Broadcast address | `10.10.10.255` | `<broadcast-address>` |
| Lease duration | `8 days` | `<lease-duration>` |
| Initial scope state | `Inactive until options are configured` | `<active-inactive>` |
| Expected client VLAN | `VLAN 10` | `<client-vlan>` |
| Expected client subnet | `10.10.10.0/24` | `<client-subnet>` |
| Relay required | No for same subnet, yes for remote subnet | `<yes-no>` |
| Evidence path | `C:\DHCP-Scope-Build` | `<evidence-path>` |
| Next workbook | `04_Configure_DHCPv4_Scope_Options.md` | `<next-task>` |
| Rollback stance | Remove scope if network range is wrong | `<rollback-plan>` |

# Create_DHCPv4_Scope_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Open elevated PowerShell | DHCP Server | `whoami /groups` | Local Administrators or DHCP Administrators context is visible |
| 2 | Confirm DHCP server hostname | DHCP Server | `hostname` | Expected DHCP server name appears |
| 3 | Confirm DHCP role installed | DHCP Server | `Get-WindowsFeature DHCP` | DHCP role is installed |
| 4 | Confirm DHCP management tools installed | DHCP Server | `Get-WindowsFeature RSAT-DHCP` | DHCP tools are installed |
| 5 | Confirm DHCP service running | DHCP Server | `Get-Service DHCPServer` | DHCP service is Running |
| 6 | Confirm DHCP server authorization | DHCP Server / DC | `Get-DhcpServerInDC` | DHCP server appears in AD authorized list |
| 7 | Import DHCP module | DHCP Server | `Import-Module DhcpServer` | DhcpServer module imports |
| 8 | Confirm existing scopes | DHCP Server | `Get-DhcpServerv4Scope` | Existing scopes are known |
| 9 | Confirm target scope does not already exist | DHCP Server | `Get-DhcpServerv4Scope -ScopeId "<scope-id>"` | Command fails or returns no duplicate scope |
| 10 | Confirm subnet plan | Operator | `Review scope ID, start, end, mask, gateway, static ranges` | Scope design is approved |
| 11 | Create DHCPv4 scope inactive first | DHCP Server | `Add-DhcpServerv4Scope -Name "<scope-name>" -StartRange "<start-range>" -EndRange "<end-range>" -SubnetMask "<subnet-mask>" -Description "<description>" -LeaseDuration 8.00:00:00 -State InActive` | Scope is created but does not lease yet |
| 12 | Confirm scope exists | DHCP Server | `Get-DhcpServerv4Scope -ScopeId "<scope-id>"` | Scope appears with expected range and mask |
| 13 | Confirm scope state | DHCP Server | `Get-DhcpServerv4Scope -ScopeId "<scope-id>" \| Select-Object ScopeId,Name,State` | Scope state is visible |
| 14 | Keep scope inactive until options are configured | DHCP Server | `Set-DhcpServerv4Scope -ScopeId "<scope-id>" -State InActive` | Scope remains inactive |
| 15 | Export scope inventory | DHCP Server | `Get-DhcpServerv4Scope \| Export-Csv C:\DHCP-Scope-Build\dhcp-scopes.csv -NoTypeInformation` | Scope list is saved |
| 16 | Review DHCP events | DHCP Server | `Get-WinEvent -LogName "Microsoft-Windows-Dhcp-Server/Operational" -MaxEvents 50` | Recent DHCP events are visible |
| 17 | Document scope build | Operator | `Record scope ID, name, range, mask, lease duration, intended gateway, and next option task` | Scope build record is complete |
| 18 | Proceed to options workbook | Operator | `04_Configure_DHCPv4_Scope_Options.md` | Scope options are configured before client validation |

# Create_DHCPv4_Scope_Precheck_Skeleton
```powershell
# Run in elevated PowerShell on the DHCP server.
# Purpose: confirm DHCP server state and prevent duplicate or wrong-scope creation.

$DhcpServerName = "DHCP1"
$ScopeName = "Corp-Users-VLAN10"
$ScopeDescription = "User workstation DHCP scope for VLAN 10"
$ScopeId = "10.10.10.0"
$StartRange = "10.10.10.50"
$EndRange = "10.10.10.200"
$SubnetMask = "255.255.255.0"
$LeaseDuration = "8.00:00:00"
$EvidencePath = "C:\DHCP-Scope-Build"

New-Item -ItemType Directory -Force -Path $EvidencePath

# Confirm administrator context.
whoami /groups |
  Tee-Object "$EvidencePath\whoami-groups.txt"

# Confirm server identity.
hostname |
  Tee-Object "$EvidencePath\hostname.txt"

Get-ComputerInfo |
  Select-Object CsName,CsDomain,CsPartOfDomain |
  Tee-Object "$EvidencePath\computer-domain-state.txt"

# Confirm DHCP role and tools.
Get-WindowsFeature DHCP,RSAT-DHCP |
  Tee-Object "$EvidencePath\dhcp-feature-state.txt"

# Confirm DHCP service.
Get-Service DHCPServer |
  Tee-Object "$EvidencePath\dhcp-service-state.txt"

# Confirm DHCP module.
Import-Module DhcpServer

Get-Command -Module DhcpServer |
  Select-Object Name,CommandType,Source |
  Tee-Object "$EvidencePath\dhcpserver-module-commands.txt"

# Confirm AD authorization state.
Get-DhcpServerInDC |
  Tee-Object "$EvidencePath\authorized-dhcp-servers.txt"

# Capture existing scope inventory.
Get-DhcpServerv4Scope |
  Tee-Object "$EvidencePath\existing-dhcpv4-scopes.txt"

# Check whether target scope already exists.
Get-DhcpServerv4Scope -ScopeId $ScopeId -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\target-scope-existing-check.txt"

# Stop if the target scope already exists.
$ExistingScope = Get-DhcpServerv4Scope -ScopeId $ScopeId -ErrorAction SilentlyContinue

if ($ExistingScope) {
  Write-Warning "Scope $ScopeId already exists. Review before creating another scope."
}
else {
  Write-Output "Scope $ScopeId does not exist yet. Safe to proceed if subnet plan is correct."
}
```

# Create_DHCPv4_Scope_Creation_Skeleton
```powershell
# Run in elevated PowerShell on the DHCP server.
# Purpose: create the DHCPv4 scope.
# Recommended stance: create inactive first, then activate after options/exclusions are configured.

$ScopeName = "Corp-Users-VLAN10"
$ScopeDescription = "User workstation DHCP scope for VLAN 10"
$ScopeId = "10.10.10.0"
$StartRange = "10.10.10.50"
$EndRange = "10.10.10.200"
$SubnetMask = "255.255.255.0"
$LeaseDuration = "8.00:00:00"
$EvidencePath = "C:\DHCP-Scope-Build"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\dhcpv4-scope-create-transcript.txt"

Import-Module DhcpServer

# Final duplicate check.
$ExistingScope = Get-DhcpServerv4Scope -ScopeId $ScopeId -ErrorAction SilentlyContinue

if ($ExistingScope) {
  Write-Warning "Scope $ScopeId already exists. No new scope was created."
  $ExistingScope
}
else {
  # Create the scope inactive first.
  Add-DhcpServerv4Scope `
    -Name $ScopeName `
    -StartRange $StartRange `
    -EndRange $EndRange `
    -SubnetMask $SubnetMask `
    -Description $ScopeDescription `
    -LeaseDuration $LeaseDuration `
    -State InActive

  # Confirm target scope after creation.
  Get-DhcpServerv4Scope -ScopeId $ScopeId |
    Tee-Object "$EvidencePath\target-scope-after-create.txt"
}

# Export all scopes after change.
Get-DhcpServerv4Scope |
  Export-Csv "$EvidencePath\dhcpv4-scopes-after-create.csv" -NoTypeInformation

Stop-Transcript
```

# Create_DHCPv4_Scope_State_Management_Skeleton
```powershell
# Run in elevated PowerShell on the DHCP server.
# Purpose: activate or deactivate the DHCPv4 scope at the correct time.
# Keep inactive until options, exclusions, and reservations are ready.

$ScopeId = "10.10.10.0"
$EvidencePath = "C:\DHCP-Scope-Build"

New-Item -ItemType Directory -Force -Path $EvidencePath

Import-Module DhcpServer

# Check current scope state.
Get-DhcpServerv4Scope -ScopeId $ScopeId |
  Select-Object ScopeId,Name,StartRange,EndRange,SubnetMask,State,LeaseDuration |
  Tee-Object "$EvidencePath\scope-state-before.txt"

# Keep inactive during initial build.
Set-DhcpServerv4Scope `
  -ScopeId $ScopeId `
  -State InActive

# Later, after options, exclusions, and reservations are configured, activate the scope.
# Set-DhcpServerv4Scope `
#   -ScopeId $ScopeId `
#   -State Active

# Confirm state after change.
Get-DhcpServerv4Scope -ScopeId $ScopeId |
  Select-Object ScopeId,Name,StartRange,EndRange,SubnetMask,State,LeaseDuration |
  Tee-Object "$EvidencePath\scope-state-after.txt"
```

# Create_DHCPv4_Scope_Validation_Skeleton
```powershell
# Run in elevated PowerShell on the DHCP server.
# Purpose: validate that the DHCPv4 scope exists and has the intended core properties.

$ScopeId = "10.10.10.0"
$EvidencePath = "C:\DHCP-Scope-Build"

New-Item -ItemType Directory -Force -Path $EvidencePath

Import-Module DhcpServer

# Confirm scope exists.
Get-DhcpServerv4Scope -ScopeId $ScopeId |
  Tee-Object "$EvidencePath\target-scope-validation.txt"

# Confirm all scopes.
Get-DhcpServerv4Scope |
  Tee-Object "$EvidencePath\all-dhcpv4-scopes.txt"

# Confirm scope statistics.
# New inactive scopes may show no leases yet.
Get-DhcpServerv4ScopeStatistics -ScopeId $ScopeId |
  Tee-Object "$EvidencePath\target-scope-statistics.txt"

# Confirm no leases yet if scope has not been activated or clients have not renewed.
Get-DhcpServerv4Lease -ScopeId $ScopeId -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\target-scope-leases.txt"

# Confirm scope options are not the focus of this workbook.
# Options are configured in 04_Configure_DHCPv4_Scope_Options.md.
Get-DhcpServerv4OptionValue -ScopeId $ScopeId -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\target-scope-options-current.txt"

# Export readable scope inventory.
Get-DhcpServerv4Scope |
  Select-Object ScopeId,Name,StartRange,EndRange,SubnetMask,State,LeaseDuration,Description |
  Export-Csv "$EvidencePath\dhcpv4-scope-inventory.csv" -NoTypeInformation
```

# Create_DHCPv4_Scope_Event_Log_Skeleton
```powershell
# Run in elevated PowerShell on the DHCP server.
# Purpose: collect DHCP Server events after scope creation.

$EvidencePath = "C:\DHCP-Scope-Build"

New-Item -ItemType Directory -Force -Path $EvidencePath

# DHCP Server operational events.
Get-WinEvent -LogName "Microsoft-Windows-Dhcp-Server/Operational" -MaxEvents 100 |
  Tee-Object "$EvidencePath\dhcp-server-operational-events.txt"

# DHCP Server admin events if present.
Get-WinEvent -LogName "Microsoft-Windows-Dhcp-Server/Admin" -MaxEvents 100 -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\dhcp-server-admin-events.txt"

# System log DHCP-related events.
Get-WinEvent -LogName System -MaxEvents 200 |
  Where-Object {
    $_.ProviderName -like "*DHCP*" -or
    $_.Message -like "*DHCP*"
  } |
  Tee-Object "$EvidencePath\system-dhcp-events.txt"

# Confirm service state after scope creation.
Get-Service DHCPServer |
  Tee-Object "$EvidencePath\dhcp-service-final.txt"
```

# Create_DHCPv4_Scope_Verification_Commands
```powershell
# Confirm DHCP role and tools.
Get-WindowsFeature DHCP,RSAT-DHCP

# Confirm DHCP service.
Get-Service DHCPServer

# Confirm DHCP server authorization.
Get-DhcpServerInDC

# Import DHCP module.
Import-Module DhcpServer

# List all DHCPv4 scopes.
Get-DhcpServerv4Scope

# Check one target scope.
Get-DhcpServerv4Scope -ScopeId "<scope-id>"

# Check scope state and range.
Get-DhcpServerv4Scope -ScopeId "<scope-id>" |
  Select-Object ScopeId,Name,StartRange,EndRange,SubnetMask,State,LeaseDuration,Description

# Create DHCPv4 scope.
Add-DhcpServerv4Scope `
  -Name "<scope-name>" `
  -StartRange "<start-range>" `
  -EndRange "<end-range>" `
  -SubnetMask "<subnet-mask>" `
  -Description "<scope-description>" `
  -LeaseDuration 8.00:00:00 `
  -State InActive

# Activate scope only after options, exclusions, and reservations are ready.
Set-DhcpServerv4Scope -ScopeId "<scope-id>" -State Active

# Deactivate scope if it should not lease yet.
Set-DhcpServerv4Scope -ScopeId "<scope-id>" -State InActive

# Check utilization.
Get-DhcpServerv4ScopeStatistics -ScopeId "<scope-id>"

# Check leases.
Get-DhcpServerv4Lease -ScopeId "<scope-id>"

# Export scope inventory.
Get-DhcpServerv4Scope |
  Export-Csv "C:\DHCP-Scope-Build\dhcpv4-scopes.csv" -NoTypeInformation

# Review DHCP Server events.
Get-WinEvent -LogName "Microsoft-Windows-Dhcp-Server/Operational" -MaxEvents 50
Get-WinEvent -LogName System -MaxEvents 100 | Where-Object {$_.ProviderName -like "*DHCP*" -or $_.Message -like "*DHCP*"}
```

# Create_DHCPv4_Scope_Rollback
| Change | Rollback Command / Action | Risk |
|---|---|---|
| Scope created with wrong subnet | `Remove-DhcpServerv4Scope -ScopeId "<scope-id>" -Force` | Deletes the scope and all related scope data |
| Scope created with wrong range | Remove and recreate scope, or adjust if safe with change control | Can affect clients if leases already exist |
| Scope created active too early | `Set-DhcpServerv4Scope -ScopeId "<scope-id>" -State InActive` | Stops new leases from that scope |
| Scope name or description wrong | `Set-DhcpServerv4Scope -ScopeId "<scope-id>" -Name "<correct-name>" -Description "<correct-description>"` | Low risk |
| Lease duration wrong | `Set-DhcpServerv4Scope -ScopeId "<scope-id>" -LeaseDuration <duration>` | Clients apply new duration on renew |
| Evidence folder created | `Remove-Item C:\DHCP-Scope-Build -Recurse -Force` | Deletes build evidence |
| Duplicate scope attempted | Stop and review existing scope | Avoid deleting until confirming it is safe |
| Wrong server selected | Remove scope from wrong server only after confirming no production use | Can cause client lease outage if active |

# Create_DHCPv4_Scope_Failure_Checks
| Symptom | Likely Layer | First Check | Do Not Start With |
|---|---|---|---|
| `Add-DhcpServerv4Scope` not recognized | DHCP tools/module missing | `Import-Module DhcpServer`; `Get-WindowsFeature RSAT-DHCP` | Reinstalling Windows |
| Scope creation fails with duplicate scope | Scope already exists | `Get-DhcpServerv4Scope -ScopeId <scope-id>` | Creating a different random subnet |
| Scope ID looks wrong | Subnet/range mismatch | Verify network address, mask, start, and end range | Activating the scope |
| Scope created but clients get no lease | Scope inactive, no options, no relay, or no client path | `Get-DhcpServerv4Scope`; check State | Deleting AD authorization |
| Scope is active but still no leases | Client network path, relay, VLAN, firewall, or no scope options | Validate client VLAN and server events | Recreating scope first |
| DHCP server not authorized | AD authorization missing | `Get-DhcpServerInDC` | Creating more scopes |
| DHCP service stopped | Service issue | `Get-Service DHCPServer` | Troubleshooting client NICs |
| StartRange or EndRange rejected | Invalid address range or mask mismatch | Confirm range is inside subnet | Changing lease duration |
| Wrong subnet mask used | Planning error | Compare ScopeId, mask, start, end, gateway | Adding exclusions |
| Scope has no gateway or DNS | Expected at this stage if workbook 04 not done yet | Proceed to scope options workbook | Testing production clients |
| Client receives APIPA | Scope not active, no relay, wrong VLAN, service issue | Check scope state, relay, DHCP service | Rebuilding DHCP server |
| Scope utilization unavailable | Scope not fully created or wrong ScopeId | `Get-DhcpServerv4Scope` | Assuming lease exhaustion |
| Event log shows authorization warning | DHCP server not authorized or cannot contact AD | `Get-DhcpServerInDC`; DNS/DC connectivity | Editing the scope range |
| Multiple overlapping scopes exist | Design conflict | List all scopes and compare ranges | Activating all scopes |

# Create_DHCPv4_Scope_Related_Labs
| Lab | Related Workbook | Skill Proven |
|---|---|---|
| Install DHCP Server role | `01_Install_DHCP_Server_Role_And_Management_Tools.md` | DHCP role and tools installation |
| Authorize DHCP server in AD | `02_Authorize_DHCP_Server_In_Active_Directory.md` | AD-integrated DHCP authorization |
| Create DHCPv4 scope | `03_Create_DHCPv4_Scope.md` | IPv4 DHCP lease pool creation |
| Configure DHCPv4 scope options | `04_Configure_DHCPv4_Scope_Options.md` | Gateway, DNS, and DNS suffix assignment |
| Configure DHCPv4 exclusions and reservations | `05_Configure_DHCPv4_Exclusions_And_Reservations.md` | Protecting static ranges and fixed lease mappings |
| Validate DHCP client lease assignment | `06_Validate_DHCP_Client_Lease_Assignment.md` | End-to-end DHCP lease verification |
| Configure DHCP relay and IP helper | `07_Configure_DHCP_Relay_And_IP_Helper.md` | Routed subnet DHCP support |
| Troubleshoot DHCP client lease failures | `12_Troubleshoot_DHCP_Client_Lease_Failures.md` | DHCP scope and client failure triage |