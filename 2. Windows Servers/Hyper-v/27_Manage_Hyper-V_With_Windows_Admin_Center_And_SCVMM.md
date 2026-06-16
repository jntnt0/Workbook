# 27_Manage_Hyper-V_With_Windows_Admin_Center_And_SCVMM

# 27_Manage_Hyper-V_With_Windows_Admin_Center_And_SCVMM_Index
27_Manage_Hyper-V_With_Windows_Admin_Center_And_SCVMM.md
27_Manage_Hyper-V_With_Windows_Admin_Center_And_SCVMM
27_Manage_Hyper-V_With_Windows_Admin_Center_And_SCVMM_Source_Basis
27_Manage_Hyper-V_With_Windows_Admin_Center_And_SCVMM_Mental_Model
27_Manage_Hyper-V_With_Windows_Admin_Center_And_SCVMM_Planning_Table
27_Manage_Hyper-V_With_Windows_Admin_Center_And_SCVMM_Configuration_Checklist
27_Manage_Hyper-V_With_Windows_Admin_Center_And_SCVMM_PreChange_Capture_Skeleton
27_Manage_Hyper-V_With_Windows_Admin_Center_And_SCVMM_WAC_Gateway_Readiness_Skeleton
27_Manage_Hyper-V_With_Windows_Admin_Center_And_SCVMM_WAC_Hyper-V_Connection_And_Management_Skeleton
27_Manage_Hyper-V_With_Windows_Admin_Center_And_SCVMM_WAC_Cluster_And_Azure_Integration_Skeleton
27_Manage_Hyper-V_With_Windows_Admin_Center_And_SCVMM_SCVMM_Server_And_Console_Readiness_Skeleton
27_Manage_Hyper-V_With_Windows_Admin_Center_And_SCVMM_SCVMM_Fabric_Host_Group_And_Host_Onboarding_Skeleton
27_Manage_Hyper-V_With_Windows_Admin_Center_And_SCVMM_SCVMM_Library_Network_And_VM_Management_Skeleton
27_Manage_Hyper-V_With_Windows_Admin_Center_And_SCVMM_PostChange_Verification_Skeleton
27_Manage_Hyper-V_With_Windows_Admin_Center_And_SCVMM_Verification_Commands
27_Manage_Hyper-V_With_Windows_Admin_Center_And_SCVMM_Rollback
27_Manage_Hyper-V_With_Windows_Admin_Center_And_SCVMM_Failure_Checks
27_Manage_Hyper-V_With_Windows_Admin_Center_And_SCVMM_Related_Labs

# 27_Manage_Hyper-V_With_Windows_Admin_Center_And_SCVMM_Source_Basis

| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Windows Admin Center overview | Supports browser-based management of Windows Server, Hyper-V hosts, clusters, VMs, storage, networking, and extensions |
| Microsoft Learn | Windows Admin Center gateway deployment | Supports WAC gateway installation, certificate, port, and access model |
| Microsoft Learn | Windows Admin Center Hyper-V and failover cluster tools | Supports VM, virtual switch, storage, event, and cluster management from WAC |
| Microsoft Learn | System Center Virtual Machine Manager overview | Supports centralized fabric, Hyper-V host, cluster, library, network, storage, and VM management |
| Microsoft Learn | VMM host and cluster management | Supports host groups, adding Hyper-V hosts, adding clusters, and fabric organization |
| Microsoft Learn | VMM networking and library concepts | Supports logical networks, VM networks, port profiles, templates, ISO/VHDX libraries, and VM deployment |
| Microsoft Learn | VMM PowerShell module | Supports `Get-SCVMMServer`, `Get-SCVMHost`, `Get-SCVirtualMachine`, `Add-SCVMHost`, and fabric inventory |
| Internal Workbook Standard | Windows Server workbook structure | Keeps this workbook aligned with the Hyper-V, AD, DNS, DHCP, GPO, File Services, and Hyper-V suite format |

# 27_Manage_Hyper-V_With_Windows_Admin_Center_And_SCVMM_Mental_Model

| Concept | Operational Meaning |
|---|---|
| Windows Admin Center | Web-based server management gateway for Windows Server and Hyper-V administration |
| WAC gateway | Server or workstation hosting the Windows Admin Center web service |
| WAC connection | Managed target such as a Windows Server, Hyper-V host, failover cluster, or Azure resource |
| WAC extension | Add-on tool inside Windows Admin Center for specific management features |
| WAC RBAC | Role-based access control model used to limit what users can do through WAC |
| WinRM | Remote management protocol used by WAC and PowerShell to manage Windows hosts |
| CredSSP or delegation | Authentication handling sometimes required for second-hop operations such as remote cluster or storage tasks |
| SCVMM | System Center Virtual Machine Manager, centralized virtualization fabric manager |
| VMM management server | Server running the VMM service and database-backed management plane |
| VMM console | GUI used by admins to manage fabric, VMs, services, jobs, and library |
| VMM fabric | Managed infrastructure layer including hosts, clusters, networks, storage, and library servers |
| Host group | Logical container in VMM used to organize Hyper-V hosts and apply placement or management scope |
| VMM library | File-based repository for VHDX files, ISOs, scripts, profiles, and templates |
| Logical network | VMM abstraction for physical network topology and VLAN/IP pools |
| VM network | VMM abstraction presented to VMs for tenant or workload connectivity |
| Run As account | Stored credential object used by VMM for host, service, or fabric operations |
| Job history | VMM task and operation history used for troubleshooting |
| Rollback boundary | This workbook configures management-plane access through WAC and VMM, not full enterprise private cloud automation |

# 27_Manage_Hyper-V_With_Windows_Admin_Center_And_SCVMM_Planning_Table

| Item | Example | Decision |
|---|---|---|
| Management model | WAC only, SCVMM only, both | `<management-model>` |
| WAC gateway server | `WAC01` | `<wac-server>` |
| WAC gateway FQDN | `wac01.corp.contoso.com` | `<wac-fqdn>` |
| WAC port | `443`, `6516` | `<wac-port>` |
| WAC certificate | Public CA, internal CA, self-signed lab | `<certificate-type>` |
| WAC admin group | `GG-WAC-Admins` | `<wac-admin-group>` |
| Hyper-V host 1 | `HV01` | `<host-1>` |
| Hyper-V host 2 | `HV02` | `<host-2>` |
| Hyper-V cluster | `HVCL01` | `<cluster-name>` |
| WinRM state | Enabled | `<winrm-state>` |
| Firewall profile | Domain | `<firewall-profile>` |
| Azure integration required | Yes or No | `<yes-no>` |
| SCVMM management server | `VMM01` | `<vmm-server>` |
| SCVMM database server | `SQL01` or local SQL | `<sql-server>` |
| SCVMM service account | `svc-scvmm` | `<vmm-service-account>` |
| SCVMM console admin group | `GG-VMM-Admins` | `<vmm-admin-group>` |
| VMM library server | `LIB01` | `<library-server>` |
| VMM library share | `\\LIB01\MSSCVMMLibrary` | `<library-share>` |
| VMM host group | `All Hosts\Lab\Hyper-V` | `<host-group>` |
| Logical network | `LN-Production` | `<logical-network>` |
| VM network | `VMN-Production` | `<vm-network>` |
| IP pool required | Yes or No | `<yes-no>` |
| VM template required | Yes or No | `<yes-no>` |
| Evidence path | `C:\Admin\HyperV-WAC-SCVMM` | `<evidence-path>` |
| Rollback required | Yes or No | `<rollback-required>` |

# 27_Manage_Hyper-V_With_Windows_Admin_Center_And_SCVMM_Configuration_Checklist

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm Hyper-V hosts are reachable | Admin workstation | `Test-NetConnection <host-name>` | Hosts resolve and respond |
| 2 | Confirm Hyper-V role on hosts | Hyper-V host | `Get-WindowsFeature Hyper-V` | Hyper-V installed |
| 3 | Confirm WinRM is enabled | Hyper-V host | `winrm quickconfig` or `Enable-PSRemoting -Force` | Remote management enabled |
| 4 | Confirm firewall rules for remote management | Hyper-V host | `Get-NetFirewallRule -DisplayGroup 'Windows Remote Management'` | WinRM rules enabled |
| 5 | Capture host and VM baseline | Hyper-V host | Run pre-change capture skeleton | Current state documented |
| 6 | Install Windows Admin Center gateway | WAC server | Run WAC installer or install from approved package | WAC gateway installed |
| 7 | Configure WAC certificate and port | WAC server | Installer options or gateway settings | Browser endpoint available |
| 8 | Add Hyper-V hosts to WAC | WAC portal | Add > Server > `<host-name>` | Host appears in WAC connections |
| 9 | Add Hyper-V cluster to WAC | WAC portal | Add > Windows Server cluster > `<cluster-name>` | Cluster appears in WAC connections |
| 10 | Validate WAC Hyper-V tools | WAC portal | Host > Virtual machines | VM inventory visible |
| 11 | Validate WAC events and performance | WAC portal | Host > Events / Performance Monitor | Host monitoring works |
| 12 | Validate WAC virtual switch management | WAC portal | Host > Virtual switches | vSwitches visible |
| 13 | Validate WAC cluster management if clustered | WAC portal | Cluster > Roles / Nodes / Storage | Cluster state visible |
| 14 | Prepare SCVMM server prerequisites | VMM server | AD, DNS, service account, SQL, Windows updates | VMM server ready |
| 15 | Install SCVMM management server and console | VMM server | Approved SCVMM installer | VMM service and console installed |
| 16 | Connect to VMM server | VMM console or PowerShell | `Get-SCVMMServer -ComputerName '<vmm-server>'` | VMM connection works |
| 17 | Create host group | VMM console or PowerShell | `New-SCVMHostGroup` | Host group exists |
| 18 | Add Hyper-V host or cluster to VMM | VMM console or PowerShell | `Add-SCVMHost` | Host appears under fabric |
| 19 | Add VMM library share | VMM console or PowerShell | `Add-SCLibraryShare` | Library content visible |
| 20 | Configure logical network and VM network | VMM console | Fabric > Networking | VM networking abstraction exists |
| 21 | Validate VM inventory in VMM | VMM console or PowerShell | `Get-SCVirtualMachine` | VMs visible |
| 22 | Validate VMM job history | VMM console or PowerShell | `Get-SCJob` | Recent jobs visible |
| 23 | Perform safe management action | WAC or VMM | Refresh host, view VM, or create test checkpoint | Management path works |
| 24 | Capture post-change evidence | WAC server, VMM server, Hyper-V host | Run post-change verification skeleton | Management configuration documented |

# 27_Manage_Hyper-V_With_Windows_Admin_Center_And_SCVMM_PreChange_Capture_Skeleton

~~~powershell
# 27_Manage_Hyper-V_With_Windows_Admin_Center_And_SCVMM_PreChange_Capture_Skeleton
# Purpose:
# Capture Hyper-V, WinRM, firewall, VM, cluster, WAC, SCVMM, and management baseline before onboarding.

$EvidenceRoot = "C:\Admin\HyperV-WAC-SCVMM"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$ClusterName = "HVCL01"
$Hosts = @("HV01", "HV02")
$OutputPath = Join-Path $EvidenceRoot "PreChange-$Timestamp"

New-Item -Path $OutputPath -ItemType Directory -Force | Out-Null

Write-Host "Capturing admin workstation context..." -ForegroundColor Cyan

whoami |
    Out-File -FilePath (Join-Path $OutputPath "00-AdminContext.txt") -Encoding UTF8

Get-ComputerInfo |
    Select-Object CsName, CsDomain, CsPartOfDomain, WindowsProductName, OsBuildNumber |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "01-AdminWorkstation-ComputerInfo.txt")

Write-Host "Capturing reachability to Hyper-V hosts..." -ForegroundColor Cyan

foreach ($HostName in $Hosts) {
    Test-NetConnection $HostName |
        Format-List |
        Out-File -FilePath (Join-Path $OutputPath "02-TestNetConnection-$HostName.txt") -Encoding UTF8
}

foreach ($HostName in $Hosts) {
    $HostPath = Join-Path $OutputPath $HostName
    New-Item -Path $HostPath -ItemType Directory -Force | Out-Null

    Write-Host "Capturing Hyper-V host state from $HostName..." -ForegroundColor Yellow

    Invoke-Command -ComputerName $HostName -ScriptBlock {
        "ComputerInfo"
        Get-ComputerInfo |
            Select-Object CsName, CsDomain, CsPartOfDomain, WindowsProductName, OsBuildNumber |
            Format-List

        "Hyper-V and clustering features"
        Get-WindowsFeature Hyper-V, Hyper-V-PowerShell, Failover-Clustering |
            Select-Object Name, DisplayName, InstallState |
            Format-Table -AutoSize

        "Hyper-V services"
        Get-Service vmms, vmcompute -ErrorAction SilentlyContinue |
            Select-Object Name, DisplayName, Status, StartType |
            Format-Table -AutoSize

        "WinRM service"
        Get-Service WinRM |
            Select-Object Name, DisplayName, Status, StartType |
            Format-Table -AutoSize

        "WinRM listeners"
        winrm enumerate winrm/config/listener

        "WinRM firewall rules"
        Get-NetFirewallRule -DisplayGroup "Windows Remote Management" -ErrorAction SilentlyContinue |
            Select-Object DisplayName, Enabled, Direction, Action, Profile |
            Format-Table -AutoSize

        "Remote Event Log firewall rules"
        Get-NetFirewallRule -DisplayGroup "Remote Event Log Management" -ErrorAction SilentlyContinue |
            Select-Object DisplayName, Enabled, Direction, Action, Profile |
            Format-Table -AutoSize

        "Hyper-V host settings"
        Get-VMHost -ErrorAction SilentlyContinue |
            Select-Object ComputerName, VirtualMachinePath, VirtualHardDiskPath, EnableEnhancedSessionMode, VirtualMachineMigrationEnabled |
            Format-List

        "VM inventory"
        Get-VM -ErrorAction SilentlyContinue |
            Sort-Object Name |
            Select-Object Name, State, Status, Generation, ProcessorCount, MemoryAssigned, Path |
            Format-Table -AutoSize

        "Virtual switches"
        Get-VMSwitch -ErrorAction SilentlyContinue |
            Select-Object Name, SwitchType, NetAdapterInterfaceDescription, AllowManagementOS |
            Format-Table -AutoSize

        "Recent Hyper-V events"
        Get-WinEvent -LogName "Microsoft-Windows-Hyper-V-VMMS-Admin" -MaxEvents 50 -ErrorAction SilentlyContinue |
            Select-Object TimeCreated, Id, LevelDisplayName, ProviderName, Message |
            Format-List
    } |
    Out-File -FilePath (Join-Path $HostPath "Host-PreChange-State.txt") -Encoding UTF8
}

Write-Host "Capturing cluster state if available..." -ForegroundColor Cyan

try {
    Get-Cluster -Name $ClusterName -ErrorAction Stop |
        Format-List * |
        Tee-Object -FilePath (Join-Path $OutputPath "03-Cluster-Summary-Before.txt")

    Get-ClusterNode -Cluster $ClusterName |
        Select-Object Name, State, NodeWeight |
        Format-Table -AutoSize |
        Tee-Object -FilePath (Join-Path $OutputPath "04-Cluster-Nodes-Before.txt")

    Get-ClusterGroup -Cluster $ClusterName |
        Select-Object Name, OwnerNode, State, GroupType |
        Format-Table -AutoSize |
        Tee-Object -FilePath (Join-Path $OutputPath "05-Cluster-Groups-Before.txt")
}
catch {
    $_ |
        Out-File -FilePath (Join-Path $OutputPath "03-Cluster-State-Error.txt") -Encoding UTF8
}

Write-Host "Capturing WAC and SCVMM local install evidence if present..." -ForegroundColor Cyan

Get-Service |
    Where-Object { $_.DisplayName -match "Windows Admin Center|Server Management|Virtual Machine Manager|System Center" -or $_.Name -match "ServerManagementGateway|SCVMM|VMM" } |
    Select-Object Name, DisplayName, Status, StartType |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "06-WAC-SCVMM-Services-Before.txt")

Get-ItemProperty "HKLM:\Software\Microsoft\Windows\CurrentVersion\Uninstall\*" -ErrorAction SilentlyContinue |
    Where-Object { $_.DisplayName -match "Windows Admin Center|Virtual Machine Manager|System Center" } |
    Select-Object DisplayName, DisplayVersion, Publisher, InstallDate |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "07-WAC-SCVMM-InstalledPrograms-Before.txt")

Write-Host "Pre-change WAC and SCVMM evidence exported to: $OutputPath" -ForegroundColor Green
~~~

# 27_Manage_Hyper-V_With_Windows_Admin_Center_And_SCVMM_WAC_Gateway_Readiness_Skeleton

~~~powershell
# 27_Manage_Hyper-V_With_Windows_Admin_Center_And_SCVMM_WAC_Gateway_Readiness_Skeleton
# Purpose:
# Validate Windows Admin Center gateway readiness before adding Hyper-V hosts or clusters.

$WacServer = $env:COMPUTERNAME
$WacPort = 6516
$ExpectedFqdn = "$env:COMPUTERNAME.$env:USERDNSDOMAIN"

Write-Host "Checking WAC gateway server identity..." -ForegroundColor Cyan

Get-ComputerInfo |
    Select-Object CsName, CsDomain, CsPartOfDomain, WindowsProductName, OsBuildNumber |
    Format-List

Write-Host "Checking local WAC-related services..." -ForegroundColor Cyan

Get-Service |
    Where-Object { $_.DisplayName -match "Windows Admin Center|Server Management" -or $_.Name -match "ServerManagementGateway" } |
    Select-Object Name, DisplayName, Status, StartType |
    Format-Table -AutoSize

Write-Host "Checking WAC installation evidence..." -ForegroundColor Cyan

Get-ItemProperty "HKLM:\Software\Microsoft\Windows\CurrentVersion\Uninstall\*" -ErrorAction SilentlyContinue |
    Where-Object { $_.DisplayName -match "Windows Admin Center" } |
    Select-Object DisplayName, DisplayVersion, Publisher, InstallDate |
    Format-Table -AutoSize

Write-Host "Checking listener port..." -ForegroundColor Cyan

Get-NetTCPConnection -LocalPort $WacPort -ErrorAction SilentlyContinue |
    Select-Object LocalAddress, LocalPort, State, OwningProcess |
    Format-Table -AutoSize

Write-Host "Checking firewall rules for WAC port..." -ForegroundColor Cyan

Get-NetFirewallRule -ErrorAction SilentlyContinue |
    Where-Object { $_.DisplayName -match "Windows Admin Center|Server Management" } |
    Select-Object DisplayName, Enabled, Direction, Action, Profile |
    Format-Table -AutoSize

Write-Host "Testing local WAC endpoint..." -ForegroundColor Cyan

Test-NetConnection `
    -ComputerName $WacServer `
    -Port $WacPort |
    Format-List

Write-Host "Checking certificate store for possible WAC certificate..." -ForegroundColor Cyan

Get-ChildItem Cert:\LocalMachine\My |
    Select-Object Subject, Issuer, NotBefore, NotAfter, Thumbprint, EnhancedKeyUsageList |
    Sort-Object NotAfter -Descending |
    Format-List

Write-Host "Expected WAC browser endpoint example:" -ForegroundColor Green
Write-Host "https://$ExpectedFqdn`:$WacPort" -ForegroundColor Yellow
~~~

# 27_Manage_Hyper-V_With_Windows_Admin_Center_And_SCVMM_WAC_Hyper-V_Connection_And_Management_Skeleton

~~~powershell
# 27_Manage_Hyper-V_With_Windows_Admin_Center_And_SCVMM_WAC_Hyper-V_Connection_And_Management_Skeleton
# Purpose:
# Prepare Hyper-V hosts for Windows Admin Center management and document portal workflow.

$Hosts = @("HV01", "HV02")
$WacFqdn = "wac01.corp.contoso.com"
$WacPort = 6516

foreach ($HostName in $Hosts) {
    Write-Host "Preparing host for WAC management: $HostName" -ForegroundColor Cyan

    Invoke-Command -ComputerName $HostName -ScriptBlock {
        Write-Host "Enabling PowerShell remoting..." -ForegroundColor Cyan
        Enable-PSRemoting -Force

        Write-Host "Starting WinRM..." -ForegroundColor Cyan
        Set-Service WinRM -StartupType Automatic
        Start-Service WinRM

        Write-Host "Enabling remote management firewall rules..." -ForegroundColor Cyan

        Get-NetFirewallRule -DisplayGroup "Windows Remote Management" -ErrorAction SilentlyContinue |
            Enable-NetFirewallRule

        Get-NetFirewallRule -DisplayGroup "Remote Event Log Management" -ErrorAction SilentlyContinue |
            Enable-NetFirewallRule

        Get-NetFirewallRule -DisplayGroup "Remote Service Management" -ErrorAction SilentlyContinue |
            Enable-NetFirewallRule

        Write-Host "Validating Hyper-V tools locally..." -ForegroundColor Cyan

        Get-WindowsFeature Hyper-V, Hyper-V-PowerShell |
            Select-Object Name, InstallState |
            Format-Table -AutoSize

        Write-Host "Validating VM inventory..." -ForegroundColor Cyan

        Get-VM -ErrorAction SilentlyContinue |
            Select-Object Name, State, Generation, Uptime |
            Format-Table -AutoSize
    }
}

Write-Host "Windows Admin Center portal workflow:" -ForegroundColor Cyan

@"
1. Open https://$WacFqdn`:$WacPort.
2. Sign in with an account authorized to manage the Hyper-V hosts.
3. Select Add.
4. Choose Servers.
5. Enter each Hyper-V host:
   - HV01
   - HV02
6. Add credentials if prompted.
7. Open each host connection.
8. Validate:
   - Overview loads.
   - Virtual machines page loads.
   - Virtual switches page loads.
   - Events page loads.
   - PowerShell page opens if enabled.
9. Use a safe action:
   - Refresh VM inventory.
   - View VM settings.
   - View host events.
   - Do not change production VM state unless approved.
"@
~~~

# 27_Manage_Hyper-V_With_Windows_Admin_Center_And_SCVMM_WAC_Cluster_And_Azure_Integration_Skeleton

~~~powershell
# 27_Manage_Hyper-V_With_Windows_Admin_Center_And_SCVMM_WAC_Cluster_And_Azure_Integration_Skeleton
# Purpose:
# Prepare failover cluster and optional Azure integrations for Windows Admin Center.

$ClusterName = "HVCL01"
$WacFqdn = "wac01.corp.contoso.com"
$WacPort = 6516

Write-Host "Checking cluster from current admin session..." -ForegroundColor Cyan

try {
    Get-Cluster -Name $ClusterName -ErrorAction Stop |
        Format-List *

    Get-ClusterNode -Cluster $ClusterName |
        Select-Object Name, State, NodeWeight |
        Format-Table -AutoSize

    Get-ClusterGroup -Cluster $ClusterName |
        Select-Object Name, OwnerNode, State, GroupType |
        Format-Table -AutoSize

    Get-ClusterSharedVolume -Cluster $ClusterName -ErrorAction SilentlyContinue |
        Format-List *
}
catch {
    Write-Warning "Cluster check failed. Verify Failover Clustering tools, permissions, and cluster name."
    $_
}

Write-Host "Windows Admin Center cluster onboarding workflow:" -ForegroundColor Cyan

@"
1. Open https://$WacFqdn`:$WacPort.
2. Select Add.
3. Choose Windows Server cluster.
4. Enter cluster name: $ClusterName.
5. Add connection.
6. Open cluster connection.
7. Validate:
   - Dashboard loads.
   - Nodes are visible.
   - Roles are visible.
   - Virtual machines are visible.
   - Storage and CSVs are visible if configured.
   - Networks are visible.
   - Events load.
8. Use a safe action:
   - Refresh cluster roles.
   - View node state.
   - View CSV state.
   - Do not move roles or change storage unless approved.
"@

Write-Host "Optional Azure integration planning notes:" -ForegroundColor Cyan

@"
WAC can integrate with Azure services depending on installed extensions and environment:
- Azure Arc onboarding
- Azure Monitor
- Azure Update Management or update workflows
- Azure Backup
- Azure Site Recovery
- Azure Network Adapter
- Azure hybrid services

Record:
- Subscription used
- Tenant used
- Resource group naming
- Azure region
- Identity used for registration
- Extension installed
- Scope of servers or clusters onboarded
"@
~~~

# 27_Manage_Hyper-V_With_Windows_Admin_Center_And_SCVMM_SCVMM_Server_And_Console_Readiness_Skeleton

~~~powershell
# 27_Manage_Hyper-V_With_Windows_Admin_Center_And_SCVMM_SCVMM_Server_And_Console_Readiness_Skeleton
# Purpose:
# Validate SCVMM management server, console, service, database reachability, and PowerShell module readiness.

$VMMServer = "VMM01"
$SqlServer = "SQL01"
$VMMPort = 8100

Write-Host "Checking VMM server reachability..." -ForegroundColor Cyan

Test-NetConnection $VMMServer |
    Format-List

Test-NetConnection $VMMServer -Port $VMMPort |
    Format-List

Write-Host "Checking SQL reachability if remote SQL is used..." -ForegroundColor Cyan

Test-NetConnection $SqlServer -Port 1433 |
    Format-List

Write-Host "Checking local VMM service state if run on VMM server..." -ForegroundColor Cyan

if ($env:COMPUTERNAME -eq $VMMServer) {
    Get-Service |
        Where-Object { $_.DisplayName -match "Virtual Machine Manager|System Center" -or $_.Name -match "SCVMM|VMM" } |
        Select-Object Name, DisplayName, Status, StartType |
        Format-Table -AutoSize
}

Write-Host "Checking installed VMM programs..." -ForegroundColor Cyan

Get-ItemProperty "HKLM:\Software\Microsoft\Windows\CurrentVersion\Uninstall\*" -ErrorAction SilentlyContinue |
    Where-Object { $_.DisplayName -match "Virtual Machine Manager|System Center" } |
    Select-Object DisplayName, DisplayVersion, Publisher, InstallDate |
    Format-Table -AutoSize

Write-Host "Checking VMM PowerShell module..." -ForegroundColor Cyan

Get-Module -ListAvailable VirtualMachineManager |
    Select-Object Name, Version, Path |
    Format-Table -AutoSize

Write-Host "Connecting to VMM server if module is available..." -ForegroundColor Cyan

try {
    Import-Module VirtualMachineManager -ErrorAction Stop

    $VMM = Get-SCVMMServer -ComputerName $VMMServer -ErrorAction Stop

    $VMM |
        Select-Object Name, Version, ProductVersion, FQDN |
        Format-List

    Write-Host "VMM connection succeeded." -ForegroundColor Green
}
catch {
    Write-Warning "VMM connection failed. Confirm console/module install, VMM service, permissions, and server name."
    $_
}
~~~

# 27_Manage_Hyper-V_With_Windows_Admin_Center_And_SCVMM_SCVMM_Fabric_Host_Group_And_Host_Onboarding_Skeleton

~~~powershell
# 27_Manage_Hyper-V_With_Windows_Admin_Center_And_SCVMM_SCVMM_Fabric_Host_Group_And_Host_Onboarding_Skeleton
# Purpose:
# Create VMM host group and onboard Hyper-V hosts into SCVMM fabric.
# Run from VMM console machine with VirtualMachineManager module installed.

$VMMServer = "VMM01"
$HostGroupPath = "All Hosts\Lab\Hyper-V"
$HyperVHosts = @("HV01.corp.contoso.com", "HV02.corp.contoso.com")

Write-Host "Connecting to VMM server..." -ForegroundColor Cyan

Import-Module VirtualMachineManager

$VMM = Get-SCVMMServer -ComputerName $VMMServer

Write-Host "Checking existing host groups..." -ForegroundColor Cyan

Get-SCVMHostGroup -VMMServer $VMM |
    Select-Object Name, Path |
    Format-Table -AutoSize

Write-Host "Creating host group path if needed..." -ForegroundColor Cyan

$PathParts = $HostGroupPath -split "\\"
$CurrentGroup = Get-SCVMHostGroup -VMMServer $VMM | Where-Object { $_.Path -eq "All Hosts" }

for ($i = 1; $i -lt $PathParts.Count; $i++) {
    $TargetName = $PathParts[$i]
    $TargetPath = ($PathParts[0..$i] -join "\")

    $Existing = Get-SCVMHostGroup -VMMServer $VMM |
        Where-Object { $_.Path -eq $TargetPath }

    if (-not $Existing) {
        $CurrentGroup = New-SCVMHostGroup `
            -Name $TargetName `
            -ParentHostGroup $CurrentGroup
    }
    else {
        $CurrentGroup = $Existing
    }
}

$TargetHostGroup = Get-SCVMHostGroup -VMMServer $VMM |
    Where-Object { $_.Path -eq $HostGroupPath }

Write-Host "Target host group:" -ForegroundColor Green

$TargetHostGroup |
    Select-Object Name, Path |
    Format-List

Write-Host "Prompting for host onboarding credential..." -ForegroundColor Cyan

$Credential = Get-Credential -Message "Enter credential with rights to add Hyper-V hosts to VMM"

foreach ($HostName in $HyperVHosts) {
    Write-Host "Checking whether host is already in VMM: $HostName" -ForegroundColor Cyan

    $ExistingHost = Get-SCVMHost -VMMServer $VMM -ComputerName $HostName -ErrorAction SilentlyContinue

    if ($ExistingHost) {
        Write-Host "Host already managed by VMM: $HostName" -ForegroundColor Yellow
        continue
    }

    Write-Host "Adding Hyper-V host to VMM: $HostName" -ForegroundColor Cyan

    Add-SCVMHost `
        -VMMServer $VMM `
        -ComputerName $HostName `
        -VMHostGroup $TargetHostGroup `
        -Credential $Credential
}

Write-Host "VMM host inventory after onboarding:" -ForegroundColor Green

Get-SCVMHost -VMMServer $VMM |
    Select-Object Name, HostCluster, VMHostGroup, OverallState, ConnectionState, OperatingSystem, TotalMemory, LogicalCPUCount |
    Format-Table -AutoSize
~~~

# 27_Manage_Hyper-V_With_Windows_Admin_Center_And_SCVMM_SCVMM_Library_Network_And_VM_Management_Skeleton

~~~powershell
# 27_Manage_Hyper-V_With_Windows_Admin_Center_And_SCVMM_SCVMM_Library_Network_And_VM_Management_Skeleton
# Purpose:
# Validate VMM library, network abstractions, VM inventory, and safe VM management operations.

$VMMServer = "VMM01"
$LibraryServer = "LIB01.corp.contoso.com"
$LibrarySharePath = "\\LIB01\MSSCVMMLibrary"
$VMName = "LAB-WIN-SRV01"

Write-Host "Connecting to VMM server..." -ForegroundColor Cyan

Import-Module VirtualMachineManager

$VMM = Get-SCVMMServer -ComputerName $VMMServer

Write-Host "Checking VMM library servers and shares..." -ForegroundColor Cyan

Get-SCLibraryServer -VMMServer $VMM |
    Select-Object Name, LibraryShares, OverallState |
    Format-List

Get-SCLibraryShare -VMMServer $VMM |
    Select-Object Name, Path, LibraryServer, SharePath |
    Format-Table -AutoSize

Write-Host "Portal or console workflow to add library share if missing:" -ForegroundColor Cyan

@"
1. Open VMM Console.
2. Go to Fabric.
3. Expand Library.
4. Add Library Server if LIB01 is not present.
5. Add library share: $LibrarySharePath.
6. Refresh library.
7. Confirm ISO, VHDX, scripts, and templates appear.
"@

Write-Host "Checking VMM logical networks..." -ForegroundColor Cyan

Get-SCLogicalNetwork -VMMServer $VMM -ErrorAction SilentlyContinue |
    Select-Object Name, Description, NetworkVirtualizationEnabled |
    Format-Table -AutoSize

Write-Host "Checking VMM VM networks..." -ForegroundColor Cyan

Get-SCVMNetwork -VMMServer $VMM -ErrorAction SilentlyContinue |
    Select-Object Name, LogicalNetwork, IsolationType |
    Format-Table -AutoSize

Write-Host "Checking VMM host inventory..." -ForegroundColor Cyan

Get-SCVMHost -VMMServer $VMM |
    Select-Object Name, VMHostGroup, OverallState, ConnectionState, HostCluster |
    Format-Table -AutoSize

Write-Host "Checking VMM VM inventory..." -ForegroundColor Cyan

Get-SCVirtualMachine -VMMServer $VMM |
    Select-Object Name, Status, HostName, VMHost, OperatingSystem, CPUCount, Memory |
    Format-Table -AutoSize

Write-Host "Checking selected VM if present in VMM..." -ForegroundColor Cyan

$VM = Get-SCVirtualMachine -VMMServer $VMM -Name $VMName -ErrorAction SilentlyContinue

if ($VM) {
    $VM |
        Format-List *

    Write-Host "Safe actions:" -ForegroundColor Yellow
    Write-Host "Refresh VM, view properties, inspect checkpoints, inspect hardware profile. Avoid state changes without approval." -ForegroundColor Yellow
}
else {
    Write-Warning "VM not found in VMM inventory: $VMName"
}

Write-Host "Checking recent VMM jobs..." -ForegroundColor Cyan

Get-SCJob -VMMServer $VMM |
    Sort-Object StartTime -Descending |
    Select-Object -First 20 Name, Status, StartTime, EndTime, Owner, TargetObject |
    Format-Table -AutoSize
~~~

# 27_Manage_Hyper-V_With_Windows_Admin_Center_And_SCVMM_PostChange_Verification_Skeleton

~~~powershell
# 27_Manage_Hyper-V_With_Windows_Admin_Center_And_SCVMM_PostChange_Verification_Skeleton
# Purpose:
# Capture WAC, SCVMM, Hyper-V host, cluster, VM, library, fabric, job, service, and event state after configuration.

$EvidenceRoot = "C:\Admin\HyperV-WAC-SCVMM"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$ClusterName = "HVCL01"
$Hosts = @("HV01", "HV02")
$VMMServer = "VMM01"
$WacServer = "WAC01"
$OutputPath = Join-Path $EvidenceRoot "PostChange-$Timestamp"

New-Item -Path $OutputPath -ItemType Directory -Force | Out-Null

Write-Host "Capturing WAC server state if reachable..." -ForegroundColor Cyan

try {
    Invoke-Command -ComputerName $WacServer -ScriptBlock {
        "WAC Server Info"
        Get-ComputerInfo |
            Select-Object CsName, CsDomain, WindowsProductName, OsBuildNumber |
            Format-List

        "WAC Services"
        Get-Service |
            Where-Object { $_.DisplayName -match "Windows Admin Center|Server Management" -or $_.Name -match "ServerManagementGateway" } |
            Select-Object Name, DisplayName, Status, StartType |
            Format-Table -AutoSize

        "WAC Installed Programs"
        Get-ItemProperty "HKLM:\Software\Microsoft\Windows\CurrentVersion\Uninstall\*" -ErrorAction SilentlyContinue |
            Where-Object { $_.DisplayName -match "Windows Admin Center" } |
            Select-Object DisplayName, DisplayVersion, Publisher, InstallDate |
            Format-Table -AutoSize

        "WAC Listener Ports"
        Get-NetTCPConnection -State Listen |
            Where-Object { $_.LocalPort -in 443,6516 } |
            Select-Object LocalAddress, LocalPort, State, OwningProcess |
            Format-Table -AutoSize
    } |
    Out-File -FilePath (Join-Path $OutputPath "01-WAC-Server-State.txt") -Encoding UTF8
}
catch {
    $_ |
        Out-File -FilePath (Join-Path $OutputPath "01-WAC-Server-State-Error.txt") -Encoding UTF8
}

Write-Host "Capturing Hyper-V host state after management onboarding..." -ForegroundColor Cyan

foreach ($HostName in $Hosts) {
    $HostPath = Join-Path $OutputPath $HostName
    New-Item -Path $HostPath -ItemType Directory -Force | Out-Null

    Invoke-Command -ComputerName $HostName -ScriptBlock {
        "Hyper-V Host"
        Get-VMHost |
            Select-Object ComputerName, VirtualMachinePath, VirtualHardDiskPath, EnableEnhancedSessionMode, VirtualMachineMigrationEnabled |
            Format-List

        "VM Inventory"
        Get-VM |
            Sort-Object Name |
            Select-Object Name, State, Status, Generation, ProcessorCount, MemoryAssigned, Path |
            Format-Table -AutoSize

        "Virtual Switches"
        Get-VMSwitch |
            Select-Object Name, SwitchType, NetAdapterInterfaceDescription, AllowManagementOS |
            Format-Table -AutoSize

        "WinRM"
        Get-Service WinRM |
            Select-Object Name, Status, StartType |
            Format-Table -AutoSize

        "Remote Management Firewall"
        Get-NetFirewallRule -DisplayGroup "Windows Remote Management" -ErrorAction SilentlyContinue |
            Select-Object DisplayName, Enabled, Direction, Action, Profile |
            Format-Table -AutoSize

        "Recent Hyper-V Events"
        Get-WinEvent -LogName "Microsoft-Windows-Hyper-V-VMMS-Admin" -MaxEvents 75 -ErrorAction SilentlyContinue |
            Select-Object TimeCreated, Id, LevelDisplayName, ProviderName, Message |
            Format-List
    } |
    Out-File -FilePath (Join-Path $HostPath "Host-PostChange-State.txt") -Encoding UTF8
}

Write-Host "Capturing cluster state after management onboarding..." -ForegroundColor Cyan

try {
    Get-Cluster -Name $ClusterName -ErrorAction Stop |
        Format-List * |
        Tee-Object -FilePath (Join-Path $OutputPath "02-Cluster-Summary-After.txt")

    Get-ClusterNode -Cluster $ClusterName |
        Select-Object Name, State, NodeWeight |
        Format-Table -AutoSize |
        Tee-Object -FilePath (Join-Path $OutputPath "03-Cluster-Nodes-After.txt")

    Get-ClusterGroup -Cluster $ClusterName |
        Select-Object Name, OwnerNode, State, GroupType |
        Format-Table -AutoSize |
        Tee-Object -FilePath (Join-Path $OutputPath "04-Cluster-Groups-After.txt")
}
catch {
    $_ |
        Out-File -FilePath (Join-Path $OutputPath "02-Cluster-State-After-Error.txt") -Encoding UTF8
}

Write-Host "Capturing SCVMM state if available..." -ForegroundColor Cyan

try {
    Import-Module VirtualMachineManager -ErrorAction Stop

    $VMM = Get-SCVMMServer -ComputerName $VMMServer -ErrorAction Stop

    $VMM |
        Select-Object Name, Version, ProductVersion, FQDN |
        Format-List |
        Tee-Object -FilePath (Join-Path $OutputPath "05-SCVMM-Server-After.txt")

    Get-SCVMHostGroup -VMMServer $VMM |
        Select-Object Name, Path |
        Format-Table -AutoSize |
        Tee-Object -FilePath (Join-Path $OutputPath "06-SCVMM-HostGroups-After.txt")

    Get-SCVMHost -VMMServer $VMM |
        Select-Object Name, VMHostGroup, OverallState, ConnectionState, HostCluster, OperatingSystem |
        Format-Table -AutoSize |
        Tee-Object -FilePath (Join-Path $OutputPath "07-SCVMM-Hosts-After.txt")

    Get-SCVirtualMachine -VMMServer $VMM |
        Select-Object Name, Status, HostName, VMHost, OperatingSystem, CPUCount, Memory |
        Format-Table -AutoSize |
        Tee-Object -FilePath (Join-Path $OutputPath "08-SCVMM-VMs-After.txt")

    Get-SCLibraryServer -VMMServer $VMM |
        Format-List * |
        Out-File -FilePath (Join-Path $OutputPath "09-SCVMM-LibraryServers-After.txt") -Encoding UTF8

    Get-SCLibraryShare -VMMServer $VMM |
        Select-Object Name, Path, LibraryServer, SharePath |
        Format-Table -AutoSize |
        Tee-Object -FilePath (Join-Path $OutputPath "10-SCVMM-LibraryShares-After.txt")

    Get-SCLogicalNetwork -VMMServer $VMM -ErrorAction SilentlyContinue |
        Select-Object Name, Description, NetworkVirtualizationEnabled |
        Format-Table -AutoSize |
        Tee-Object -FilePath (Join-Path $OutputPath "11-SCVMM-LogicalNetworks-After.txt")

    Get-SCVMNetwork -VMMServer $VMM -ErrorAction SilentlyContinue |
        Select-Object Name, LogicalNetwork, IsolationType |
        Format-Table -AutoSize |
        Tee-Object -FilePath (Join-Path $OutputPath "12-SCVMM-VMNetworks-After.txt")

    Get-SCJob -VMMServer $VMM |
        Sort-Object StartTime -Descending |
        Select-Object -First 30 Name, Status, StartTime, EndTime, Owner, TargetObject |
        Format-Table -AutoSize |
        Tee-Object -FilePath (Join-Path $OutputPath "13-SCVMM-Jobs-After.txt")
}
catch {
    $_ |
        Out-File -FilePath (Join-Path $OutputPath "05-SCVMM-State-After-Error.txt") -Encoding UTF8
}

Write-Host "Post-change WAC and SCVMM evidence exported to: $OutputPath" -ForegroundColor Green
~~~

# 27_Manage_Hyper-V_With_Windows_Admin_Center_And_SCVMM_Verification_Commands

| Command | Where to Run | What It Confirms |
|---|---|---|
| `Test-NetConnection <host-name>` | Admin workstation | Host DNS and network reachability |
| `Enable-PSRemoting -Force` | Hyper-V host | Enables PowerShell remoting for WAC and admin tools |
| `Get-Service WinRM` | Hyper-V host | WinRM service state |
| `winrm enumerate winrm/config/listener` | Hyper-V host | WinRM listeners |
| `Get-NetFirewallRule -DisplayGroup 'Windows Remote Management'` | Hyper-V host | WinRM firewall rules |
| `Get-WindowsFeature Hyper-V` | Hyper-V host | Hyper-V role state |
| `Get-VMHost` | Hyper-V host | Hyper-V host settings |
| `Get-VM` | Hyper-V host | VM inventory |
| `Get-VMSwitch` | Hyper-V host | vSwitch inventory |
| `Get-ClusterNode` | Cluster node | Cluster node state |
| `Get-ClusterGroup` | Cluster node | Cluster roles, including clustered VMs |
| `Get-ClusterSharedVolume` | Cluster node | CSV state |
| `Test-NetConnection <wac-server> -Port <wac-port>` | Admin workstation | WAC gateway port reachability |
| `Get-Service \| Where-Object DisplayName -match 'Windows Admin Center'` | WAC server | WAC service state |
| `Get-Module -ListAvailable VirtualMachineManager` | VMM console or server | VMM PowerShell module availability |
| `Import-Module VirtualMachineManager` | VMM console or server | Loads VMM cmdlets |
| `Get-SCVMMServer -ComputerName '<vmm-server>'` | VMM console or server | Connects to VMM management server |
| `Get-SCVMHostGroup` | VMM console or server | Shows VMM host groups |
| `New-SCVMHostGroup` | VMM console or server | Creates VMM host group |
| `Add-SCVMHost -ComputerName '<host-fqdn>' -VMHostGroup '<group>' -Credential '<credential>'` | VMM console or server | Adds Hyper-V host to VMM |
| `Get-SCVMHost` | VMM console or server | Shows managed Hyper-V hosts |
| `Get-SCVirtualMachine` | VMM console or server | Shows VM inventory in VMM |
| `Get-SCLibraryServer` | VMM console or server | Shows VMM library servers |
| `Get-SCLibraryShare` | VMM console or server | Shows VMM library shares |
| `Get-SCLogicalNetwork` | VMM console or server | Shows VMM logical networks |
| `Get-SCVMNetwork` | VMM console or server | Shows VMM VM networks |
| `Get-SCJob` | VMM console or server | Shows VMM job history |
| `Get-WinEvent -LogName 'Microsoft-Windows-Hyper-V-VMMS-Admin' -MaxEvents 20` | Hyper-V host | Hyper-V management events |
| `Get-WinEvent -LogName 'Microsoft-Windows-FailoverClustering/Operational' -MaxEvents 20` | Cluster node | Cluster operational events |

# 27_Manage_Hyper-V_With_Windows_Admin_Center_And_SCVMM_Rollback

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Capture current management state before rollback | WAC server, VMM server, Hyper-V hosts | Run post-change verification skeleton | Current state documented |
| 2 | Remove test WAC connection only | WAC portal | WAC > All connections > Remove connection | WAC no longer lists target connection |
| 3 | Disable WAC extension if test-only | WAC portal | Settings > Extensions | Extension disabled or removed |
| 4 | Stop WAC gateway only if lab cleanup requires it | WAC server | `Stop-Service ServerManagementGateway` | WAC gateway stopped |
| 5 | Uninstall WAC only if no longer required | WAC server | Programs and Features or approved installer uninstall | WAC removed |
| 6 | Remove test host from VMM if rollback requires it | VMM console | Fabric > Servers > Remove host | Host removed from VMM management |
| 7 | Remove test host through VMM PowerShell if approved | VMM console/server | `Remove-SCVMHost -VMHost '<host-object>'` | Host removed from VMM |
| 8 | Remove test host group if empty | VMM console/server | `Remove-SCVMHostGroup '<host-group-object>'` | Empty host group removed |
| 9 | Remove test library share if created | VMM console/server | Remove library share from Fabric > Library | Library share removed from VMM |
| 10 | Remove test logical network or VM network if unused | VMM console/server | Remove network object after dependency check | Test network abstraction removed |
| 11 | Revert WinRM firewall changes only if baseline required it | Hyper-V host | Disable specific firewall rules from baseline diff | Firewall returns to baseline |
| 12 | Leave Hyper-V hosts and VMs unchanged unless explicitly approved | Hyper-V host | No action | Workloads remain intact |
| 13 | Capture rollback evidence | All management endpoints | Run post-change verification skeleton | Rollback documented |

# 27_Manage_Hyper-V_With_Windows_Admin_Center_And_SCVMM_Failure_Checks

| Symptom | Likely Cause | Fix |
|---|---|---|
| WAC cannot connect to host | WinRM disabled, firewall blocked, DNS issue, or permissions | Enable WinRM, open firewall, fix DNS, use authorized account |
| WAC shows access denied | User lacks local admin or delegated permissions | Add user to correct admin group or configure WAC RBAC |
| WAC host overview loads but VM tool fails | Hyper-V PowerShell missing or VMMS issue | Install Hyper-V tools and check VMMS service |
| WAC cluster connection fails | Cluster name DNS, permissions, or Failover Clustering tools issue | Validate cluster access with `Get-Cluster` |
| WAC second-hop storage operation fails | Credential delegation or CredSSP issue | Configure approved delegation path or run action locally |
| WAC browser certificate warning | Self-signed or untrusted certificate | Install trusted certificate from internal or public CA |
| WAC gateway unreachable | Service stopped, port blocked, or wrong URL | Check service, port, firewall, and FQDN |
| VMM console cannot connect | VMM service down, port blocked, wrong server, or permissions | Check VMM service, firewall, DNS, and admin role |
| VMM PowerShell module missing | Console not installed on admin machine | Install VMM console or run from VMM server |
| `Get-SCVMMServer` fails | VMM server unreachable or module/version mismatch | Confirm VMM server, service, console version, and network |
| Add Hyper-V host to VMM fails | Credential, firewall, WinRM, domain trust, or existing management conflict | Fix credentials, firewall, WinRM, DNS, and duplicate host state |
| Host appears in VMM with needs attention | Agent issue, version mismatch, or refresh failure | Update VMM agent and refresh host |
| Host not showing VMs in VMM | Host refresh pending or agent issue | Refresh host and check VMM jobs |
| VMM library share missing content | Library not refreshed or share permissions wrong | Refresh library and fix NTFS/share permissions |
| Logical network not available to VM | Network not associated with host adapter or host group | Map logical network to host NIC and host group |
| VM network assignment fails | Missing logical network, IP pool, or port profile | Fix VMM networking fabric configuration |
| VMM job stuck or failed | Provider, agent, permission, or host communication issue | Open job details and review host/VMM events |
| WAC and VMM show different VM state | Refresh delay or different management scope | Refresh both tools and verify directly on host |
| VMM action conflicts with cluster | Cluster ownership, maintenance mode, or role state issue | Use cluster-aware operation and verify owner node |
| Removing host from VMM risks losing templates or placement data | Host has dependencies in VMM | Review dependencies before removing host |

# 27_Manage_Hyper-V_With_Windows_Admin_Center_And_SCVMM_Related_Labs

| Lab                                                                           | Relationship                                                                  |
| ----------------------------------------------------------------------------- | ----------------------------------------------------------------------------- |
| 00_Hyper-V_Index.md                                                           | Places WAC and SCVMM management in the full Hyper-V suite                     |
| 01_Confirm_Hyper-V_Host_Baseline.md                                           | Confirms host readiness before onboarding to management tools                 |
| 02_Install_Hyper-V_Role_And_Management_Tools.md                               | Required before VM and host management through WAC or VMM                     |
| 03_Configure_Hyper-V_Host_Default_Paths_And_Settings.md                       | WAC and VMM expose and manage host default paths                              |
| 04_Create_And_Configure_Virtual_Switches.md                                   | WAC and VMM can inspect and manage virtual switch configuration               |
| 05_Create_Generation_1_And_Generation_2_VMs.md                                | WAC and VMM both manage VM lifecycle and hardware settings                    |
| 07_Create_Attach_Resize_And_Compact_VHDX_Disks.md                             | Disk inventory and VM storage are visible through management tools            |
| 08_Configure_VM_Network_Adapters_VLANs_MAC_And_Bandwidth.md                   | VM network settings are manageable through WAC and VMM                        |
| 10_Configure_Checkpoints_Production_Checkpoints_And_Restore.md                | WAC and VMM expose checkpoint operations and history                          |
| 11_Manage_VM_Lifecycle_Start_Stop_Save_Reset_Export_Import.md                 | Management tools centralize VM lifecycle workflows                            |
| 13_Configure_VM_Backup_And_Recovery_Workflows.md                              | WAC and VMM support visibility into backup-adjacent operational state         |
| 14_Configure_Live_Migration_And_Storage_Migration.md                          | VMM and WAC can manage or observe migration scenarios                         |
| 16_Configure_Hyper-V_Failover_Clustering_And_Clustered_VMs.md                 | WAC and VMM both manage clustered hosts and highly available VMs              |
| 17_Monitor_Hyper-V_Hosts_VMs_Events_Performance_And_Resource_Usage.md         | WAC and VMM provide operational monitoring views                              |
| 18_Troubleshoot_Hyper-V_VM_Network_Storage_Checkpoint_And_Startup_Failures.md | WAC and VMM evidence helps troubleshoot host, VM, cluster, and job failures   |
| 23_Create_VM_Templates_Differencing_Disks_And_Golden_Images.md                | VMM library and templates expand template-based VM deployment                 |
| 24_Configure_Hyper-V_With_Storage_Spaces_Direct_And_CSV.md                    | WAC and VMM can manage or observe cluster, CSV, and S2D-backed VM state       |
| 25_Configure_Hyper-V_Converged_Networking_For_Cluster_Hosts.md                | WAC and VMM help manage and validate converged cluster networking             |
| 26_Configure_Azure_Site_Recovery_For_Hyper-V.md                               | WAC and Azure integration can complement ASR visibility and hybrid management |