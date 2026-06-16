# 26_Configure_Azure_Site_Recovery_For_Hyper-V

# 26_Configure_Azure_Site_Recovery_For_Hyper-V_Index
26_Configure_Azure_Site_Recovery_For_Hyper-V.md
26_Configure_Azure_Site_Recovery_For_Hyper-V
26_Configure_Azure_Site_Recovery_For_Hyper-V_Source_Basis
26_Configure_Azure_Site_Recovery_For_Hyper-V_Mental_Model
26_Configure_Azure_Site_Recovery_For_Hyper-V_Planning_Table
26_Configure_Azure_Site_Recovery_For_Hyper-V_Configuration_Checklist
26_Configure_Azure_Site_Recovery_For_Hyper-V_PreChange_Capture_Skeleton
26_Configure_Azure_Site_Recovery_For_Hyper-V_Azure_Resource_Preparation_Skeleton
26_Configure_Azure_Site_Recovery_For_Hyper-V_Hyper-V_Host_Readiness_Skeleton
26_Configure_Azure_Site_Recovery_For_Hyper-V_ASR_Provider_And_Registration_Skeleton
26_Configure_Azure_Site_Recovery_For_Hyper-V_Replication_Policy_And_Enablement_Skeleton
26_Configure_Azure_Site_Recovery_For_Hyper-V_Compute_Network_And_Disk_Settings_Skeleton
26_Configure_Azure_Site_Recovery_For_Hyper-V_Test_Failover_Drill_Skeleton
26_Configure_Azure_Site_Recovery_For_Hyper-V_Planned_Failover_And_Failback_Skeleton
26_Configure_Azure_Site_Recovery_For_Hyper-V_PostChange_Verification_Skeleton
26_Configure_Azure_Site_Recovery_For_Hyper-V_Verification_Commands
26_Configure_Azure_Site_Recovery_For_Hyper-V_Rollback
26_Configure_Azure_Site_Recovery_For_Hyper-V_Failure_Checks
26_Configure_Azure_Site_Recovery_For_Hyper-V_Related_Labs

# 26_Configure_Azure_Site_Recovery_For_Hyper-V_Source_Basis

| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Prepare Azure resources for Hyper-V disaster recovery | Supports Azure resource group, Recovery Services vault, storage, and target virtual network planning |
| Microsoft Learn | Prepare on-premises Hyper-V servers for disaster recovery | Supports Hyper-V host requirements, internet access, VM access after failover, and optional VMM planning |
| Microsoft Learn | Set up disaster recovery of on-premises Hyper-V VMs to Azure | Supports Hyper-V site, ASR provider, MARS agent, vault registration key, replication policy, and enable replication workflow |
| Microsoft Learn | Run a disaster recovery drill to Azure | Supports isolated test network, test failover, validation, and cleanup workflow |
| Microsoft Learn | Azure Site Recovery supported scenarios | Supports compatibility review for Hyper-V host, VM, storage, compute, network, and failover targets |
| Microsoft Learn | Azure PowerShell Az.RecoveryServices | Supports vault discovery and ASR context inspection where automation is used |
| Internal Workbook Standard | Windows Server workbook structure | Keeps this workbook aligned with the Hyper-V, AD, DNS, DHCP, GPO, File Services, and Hyper-V suite format |

# 26_Configure_Azure_Site_Recovery_For_Hyper-V_Mental_Model

| Concept | Operational Meaning |
|---|---|
| Azure Site Recovery | Azure disaster recovery orchestration service for replication, failover, and failback |
| Recovery Services vault | Azure control plane object that stores ASR configuration, metadata, policies, jobs, and protected item state |
| Hyper-V site | ASR grouping object for one or more standalone Hyper-V hosts |
| Hyper-V host registration | Process of installing the ASR provider and MARS agent, then registering the host with a vault key |
| Vault registration key | Downloaded credential file used to register the on-premises Hyper-V host with the Recovery Services vault |
| ASR provider | Software installed on Hyper-V hosts so ASR can discover and replicate VMs |
| MARS agent | Microsoft Azure Recovery Services agent installed with the ASR provider for Hyper-V-to-Azure scenarios |
| Replication policy | Defines copy frequency, recovery point retention, app-consistent snapshot frequency, and initial replication timing |
| Protected item | VM that has replication enabled in Azure Site Recovery |
| Initial replication | First transfer of VM disk data to Azure |
| Recovery point | Point-in-time copy used for test failover or actual failover |
| App-consistent recovery point | Recovery point coordinated with guest VSS where supported |
| Crash-consistent recovery point | Recovery point equivalent to VM power-loss consistency |
| Test failover | Non-disruptive DR drill that creates an Azure VM in an isolated test network |
| Planned failover | Controlled failover when on-premises source is available |
| Unplanned failover | Failover when the source site is unavailable |
| Commit | Action that accepts a failover recovery point as final |
| Re-protect | Reverses protection direction after failover so the VM can fail back |
| Failback | Return workload from Azure back to Hyper-V/on-premises after recovery |
| Rollback boundary | This workbook configures ASR for Hyper-V DR, not full application dependency mapping, DNS cutover automation, or Azure landing zone design |

# 26_Configure_Azure_Site_Recovery_For_Hyper-V_Planning_Table

| Item | Example | Decision |
|---|---|---|
| Azure subscription | `Prod-Subscription` | `<subscription-name>` |
| Azure tenant | `contoso.onmicrosoft.com` | `<tenant-name>` |
| Azure region | `East US`, `West Europe` | `<azure-region>` |
| Resource group | `rg-asr-hyperv-lab` | `<resource-group>` |
| Recovery Services vault | `rsv-hyperv-asr-lab` | `<vault-name>` |
| Hyper-V site name | `HV-Site-01` | `<hyperv-site-name>` |
| Hyper-V host | `HV01` | `<hyperv-host>` |
| Additional Hyper-V hosts | `HV02`, `HV03` | `<host-list>` |
| Source VM | `LAB-WIN-SRV01` | `<vm-name>` |
| Source VM OS | Windows Server 2022, Windows 11, Linux | `<guest-os>` |
| Source VM generation | Gen1, Gen2 | `<generation>` |
| Source VM disk count | `1`, `2`, `4` | `<disk-count>` |
| Target Azure VNet | `vnet-asr-prod` | `<target-vnet>` |
| Target Azure subnet | `snet-asr-prod` | `<target-subnet>` |
| Test failover VNet | `vnet-asr-test` | `<test-vnet>` |
| Test failover subnet | `snet-asr-test` | `<test-subnet>` |
| Target resource group after failover | `rg-asr-failover-vms` | `<target-rg>` |
| Cache or replica storage account | `stasrreplica001` | `<storage-account>` |
| Target disk mode | Managed disks | `<target-disk-mode>` |
| Target VM size | `Standard_D2s_v5` | `<target-vm-size>` |
| Availability option | None, availability set, zone | `<availability-option>` |
| Replication policy name | `pol-hyperv-5min` | `<policy-name>` |
| Copy frequency | `5 minutes` | `<copy-frequency>` |
| Recovery point retention | `2 hours`, `24 hours` | `<retention-hours>` |
| App-consistent snapshot frequency | `1 hour` | `<app-consistent-frequency>` |
| Initial replication start | Immediately, scheduled | `<initial-replication-start>` |
| RPO target | `15 minutes` | `<rpo-target>` |
| RTO target | `1 hour` | `<rto-target>` |
| DNS failover method | Manual, Azure DNS, internal DNS update | `<dns-method>` |
| Evidence path | `C:\Admin\HyperV-ASR` | `<evidence-path>` |
| Rollback required | Yes or No | `<rollback-required>` |

# 26_Configure_Azure_Site_Recovery_For_Hyper-V_Configuration_Checklist

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm Azure permissions | Admin workstation | Confirm Virtual Machine Contributor and Site Recovery Contributor | Account can create ASR resources and manage vault operations |
| 2 | Install or update Az PowerShell | Admin workstation | `Install-Module Az -Scope CurrentUser -Force` | Azure PowerShell available |
| 3 | Sign in to Azure | Admin workstation | `Connect-AzAccount` | Session connected to tenant |
| 4 | Select subscription | Admin workstation | `Set-AzContext -Subscription '<subscription-name>'` | Correct subscription selected |
| 5 | Create resource group | Admin workstation | `New-AzResourceGroup -Name '<rg>' -Location '<region>'` | Resource group exists |
| 6 | Create Recovery Services vault | Admin workstation | `New-AzRecoveryServicesVault -Name '<vault>' -ResourceGroupName '<rg>' -Location '<region>'` | Vault exists |
| 7 | Create target VNet and subnet | Admin workstation | `New-AzVirtualNetwork` | Target network exists |
| 8 | Create isolated test failover VNet | Admin workstation | `New-AzVirtualNetwork` | Isolated test network exists |
| 9 | Create storage account if required | Admin workstation | `New-AzStorageAccount` | Storage account exists |
| 10 | Confirm Hyper-V host readiness | Hyper-V host | `Get-WindowsFeature Hyper-V; Get-VM` | Host and source VMs are visible |
| 11 | Confirm VM meets Azure compatibility | Hyper-V host | Run readiness skeleton | Disk, OS, network, and generation are documented |
| 12 | Prepare ASR infrastructure in vault | Azure portal | Vault > Enable Site Recovery > Hyper-V machines to Azure > Prepare infrastructure | Hyper-V site workflow begins |
| 13 | Create Hyper-V site | Azure portal | Add Hyper-V site | Hyper-V site exists |
| 14 | Download ASR provider installer | Azure portal | Add Hyper-V server > Download installer | `AzureSiteRecoveryProvider.exe` downloaded |
| 15 | Download vault registration key | Azure portal | Add Hyper-V server > Download registration key | Vault key downloaded |
| 16 | Install ASR provider on Hyper-V host | Hyper-V host | Run provider installer | ASR provider and MARS agent installed |
| 17 | Register Hyper-V host to vault | Hyper-V host | Registration wizard or `DRConfigurator.exe` | Host appears under ASR infrastructure |
| 18 | Create or associate replication policy | Azure portal | Create policy with copy frequency and retention | Policy associated with Hyper-V site |
| 19 | Enable replication for VM | Azure portal | Vault > Enable replication > select VM | VM becomes protected item |
| 20 | Confirm initial replication progress | Azure portal | Vault > Replicated items > VM | Initial replication completes |
| 21 | Configure compute and network properties | Azure portal | Replicated item > Compute and Network | Target VM size, VNet, subnet, disk mode configured |
| 22 | Run isolated test failover | Azure portal | Replicated item > Test Failover | Azure test VM created in isolated network |
| 23 | Validate test VM boot and access | Azure portal / Azure VM | RDP, SSH, serial console, boot diagnostics | Test VM is usable |
| 24 | Cleanup test failover | Azure portal | Replicated item > Cleanup test failover | Test VM removed and notes saved |
| 25 | Document planned failover runbook | Azure portal / workbook | Planned failover and failback skeleton | Recovery process is operationally defined |
| 26 | Capture post-change evidence | Admin workstation and Hyper-V host | Run post-change verification skeleton | ASR state is documented |

# 26_Configure_Azure_Site_Recovery_For_Hyper-V_PreChange_Capture_Skeleton

~~~powershell
# 26_Configure_Azure_Site_Recovery_For_Hyper-V_PreChange_Capture_Skeleton
# Purpose:
# Capture Hyper-V host, VM, network, disk, checkpoints, provider, Azure module, and ASR local state before configuration.

$EvidenceRoot = "C:\Admin\HyperV-ASR"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$VMName = "LAB-WIN-SRV01"
$OutputPath = Join-Path $EvidenceRoot "PreChange-$VMName-$Timestamp"

New-Item -Path $OutputPath -ItemType Directory -Force | Out-Null

Write-Host "Capturing Hyper-V host identity..." -ForegroundColor Cyan

Get-ComputerInfo |
    Select-Object CsName, CsDomain, CsPartOfDomain, WindowsProductName, OsBuildNumber, OsArchitecture |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "01-Host-ComputerInfo.txt")

Write-Host "Capturing Hyper-V feature and services..." -ForegroundColor Cyan

Get-WindowsFeature Hyper-V, Hyper-V-PowerShell |
    Select-Object Name, DisplayName, InstallState |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "02-HyperV-Features.txt")

Get-Service vmms, vmcompute -ErrorAction SilentlyContinue |
    Select-Object Name, DisplayName, Status, StartType |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "03-HyperV-Services.txt")

Write-Host "Capturing source VM summary..." -ForegroundColor Cyan

$VM = Get-VM -Name $VMName -ErrorAction SilentlyContinue

if ($VM) {
    $VM |
        Select-Object Name, Id, State, Status, Generation, Uptime, ProcessorCount, DynamicMemoryEnabled, MemoryStartup, MemoryAssigned, Path, ConfigurationLocation, CheckpointType |
        Format-List |
        Tee-Object -FilePath (Join-Path $OutputPath "04-VM-Summary.txt")

    $VM |
        ConvertTo-Json -Depth 10 |
        Out-File -FilePath (Join-Path $OutputPath "04-VM-Summary.json") -Encoding UTF8

    Write-Host "Capturing VM disks..." -ForegroundColor Cyan

    $HardDisks = Get-VMHardDiskDrive -VMName $VMName -ErrorAction SilentlyContinue

    $HardDisks |
        Select-Object VMName, ControllerType, ControllerNumber, ControllerLocation, Path |
        Format-Table -AutoSize |
        Tee-Object -FilePath (Join-Path $OutputPath "05-VM-HardDisks.txt")

    foreach ($Disk in $HardDisks) {
        if ($Disk.Path -and (Test-Path $Disk.Path)) {
            Get-VHD -Path $Disk.Path |
                Select-Object Path, VhdFormat, VhdType, FileSize, Size, ParentPath, Attached |
                Format-List |
                Out-File -FilePath (Join-Path $OutputPath ("06-VHD-" + [IO.Path]::GetFileNameWithoutExtension($Disk.Path) + ".txt")) -Encoding UTF8
        }
    }

    Write-Host "Capturing VM network adapters..." -ForegroundColor Cyan

    Get-VMNetworkAdapter -VMName $VMName |
        Select-Object VMName, Name, SwitchName, MacAddress, Status, Connected, IPAddresses |
        Format-List |
        Tee-Object -FilePath (Join-Path $OutputPath "07-VM-NetworkAdapters.txt")

    Get-VMNetworkAdapterVlan -VMName $VMName |
        Format-List |
        Tee-Object -FilePath (Join-Path $OutputPath "08-VM-NetworkAdapterVLAN.txt")

    Write-Host "Capturing checkpoints..." -ForegroundColor Cyan

    Get-VMCheckpoint -VMName $VMName -ErrorAction SilentlyContinue |
        Select-Object VMName, Name, Id, CreationTime, SnapshotType, ParentSnapshotName |
        Format-Table -AutoSize |
        Tee-Object -FilePath (Join-Path $OutputPath "09-VM-Checkpoints.txt")

    Write-Host "Capturing integration services..." -ForegroundColor Cyan

    Get-VMIntegrationService -VMName $VMName |
        Select-Object VMName, Name, Enabled, PrimaryStatusDescription, SecondaryStatusDescription |
        Format-Table -AutoSize |
        Tee-Object -FilePath (Join-Path $OutputPath "10-VM-IntegrationServices.txt")
}
else {
    "VM not found: $VMName" |
        Out-File -FilePath (Join-Path $OutputPath "04-VM-NotFound.txt") -Encoding UTF8
}

Write-Host "Capturing host virtual switch and IP state..." -ForegroundColor Cyan

Get-VMSwitch |
    Select-Object Name, SwitchType, NetAdapterInterfaceDescription, AllowManagementOS |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "11-VMSwitches.txt")

Get-NetIPConfiguration |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "12-NetIPConfiguration.txt")

Write-Host "Capturing ASR and MARS installation evidence if present..." -ForegroundColor Cyan

Get-Service |
    Where-Object { $_.Name -match "Azure|ASR|MARS|obengine|dra" -or $_.DisplayName -match "Azure|Site Recovery|Recovery Services" } |
    Select-Object Name, DisplayName, Status, StartType |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "13-ASR-MARS-Services.txt")

Get-ItemProperty "HKLM:\Software\Microsoft\Windows\CurrentVersion\Uninstall\*" -ErrorAction SilentlyContinue |
    Where-Object { $_.DisplayName -match "Azure Site Recovery|Microsoft Azure Recovery Services|MARS|Recovery Services" } |
    Select-Object DisplayName, DisplayVersion, Publisher, InstallDate |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "14-ASR-MARS-InstalledPrograms.txt")

Write-Host "Capturing Azure PowerShell module state..." -ForegroundColor Cyan

Get-Module -ListAvailable Az, Az.RecoveryServices, Az.Compute, Az.Network, Az.Storage |
    Select-Object Name, Version, Path |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "15-AzModules.txt")

Write-Host "Capturing Hyper-V and system events..." -ForegroundColor Cyan

$Logs = @(
    "Microsoft-Windows-Hyper-V-VMMS-Admin",
    "Microsoft-Windows-Hyper-V-Worker-Admin",
    "Microsoft-Windows-Hyper-V-Config-Admin",
    "System",
    "Application"
)

foreach ($Log in $Logs) {
    $SafeLog = $Log -replace '[\\\/]', "-"

    Get-WinEvent -LogName $Log -MaxEvents 100 -ErrorAction SilentlyContinue |
        Select-Object TimeCreated, Id, LevelDisplayName, ProviderName, Message |
        Format-List |
        Out-File -FilePath (Join-Path $OutputPath "16-$SafeLog-Before.txt") -Encoding UTF8
}

Write-Host "Pre-change ASR evidence exported to: $OutputPath" -ForegroundColor Green
~~~

# 26_Configure_Azure_Site_Recovery_For_Hyper-V_Azure_Resource_Preparation_Skeleton

~~~powershell
# 26_Configure_Azure_Site_Recovery_For_Hyper-V_Azure_Resource_Preparation_Skeleton
# Purpose:
# Prepare Azure resource group, Recovery Services vault, storage account, target VNet, and isolated test failover VNet.

$SubscriptionName = "Prod-Subscription"
$Location = "eastus"
$ResourceGroupName = "rg-asr-hyperv-lab"
$VaultName = "rsv-hyperv-asr-lab"
$StorageAccountName = "stasrhvreplica001"

$TargetVNetName = "vnet-asr-prod"
$TargetSubnetName = "snet-asr-prod"
$TargetAddressPrefix = "10.50.0.0/16"
$TargetSubnetPrefix = "10.50.1.0/24"

$TestVNetName = "vnet-asr-test"
$TestSubnetName = "snet-asr-test"
$TestAddressPrefix = "10.60.0.0/16"
$TestSubnetPrefix = "10.60.1.0/24"

Write-Host "Installing required Az modules if missing..." -ForegroundColor Cyan

$RequiredModules = @(
    "Az.Accounts",
    "Az.Resources",
    "Az.RecoveryServices",
    "Az.Network",
    "Az.Storage",
    "Az.Compute"
)

foreach ($Module in $RequiredModules) {
    if (-not (Get-Module -ListAvailable -Name $Module)) {
        Install-Module $Module -Scope CurrentUser -Force
    }
}

Write-Host "Connecting to Azure..." -ForegroundColor Cyan

Connect-AzAccount

Set-AzContext -Subscription $SubscriptionName

Write-Host "Creating resource group if missing..." -ForegroundColor Cyan

$RG = Get-AzResourceGroup -Name $ResourceGroupName -ErrorAction SilentlyContinue

if (-not $RG) {
    $RG = New-AzResourceGroup `
        -Name $ResourceGroupName `
        -Location $Location
}

Write-Host "Creating Recovery Services vault if missing..." -ForegroundColor Cyan

$Vault = Get-AzRecoveryServicesVault `
    -ResourceGroupName $ResourceGroupName `
    -Name $VaultName `
    -ErrorAction SilentlyContinue

if (-not $Vault) {
    $Vault = New-AzRecoveryServicesVault `
        -Name $VaultName `
        -ResourceGroupName $ResourceGroupName `
        -Location $Location
}

Set-AzRecoveryServicesAsrVaultContext -Vault $Vault

Write-Host "Creating storage account if missing..." -ForegroundColor Cyan

$StorageAccount = Get-AzStorageAccount `
    -ResourceGroupName $ResourceGroupName `
    -Name $StorageAccountName `
    -ErrorAction SilentlyContinue

if (-not $StorageAccount) {
    $StorageAccount = New-AzStorageAccount `
        -ResourceGroupName $ResourceGroupName `
        -Name $StorageAccountName `
        -Location $Location `
        -SkuName Standard_GRS `
        -Kind StorageV2
}

Write-Host "Creating target VNet if missing..." -ForegroundColor Cyan

$TargetVNet = Get-AzVirtualNetwork `
    -ResourceGroupName $ResourceGroupName `
    -Name $TargetVNetName `
    -ErrorAction SilentlyContinue

if (-not $TargetVNet) {
    $TargetSubnet = New-AzVirtualNetworkSubnetConfig `
        -Name $TargetSubnetName `
        -AddressPrefix $TargetSubnetPrefix

    $TargetVNet = New-AzVirtualNetwork `
        -Name $TargetVNetName `
        -ResourceGroupName $ResourceGroupName `
        -Location $Location `
        -AddressPrefix $TargetAddressPrefix `
        -Subnet $TargetSubnet
}

Write-Host "Creating isolated test failover VNet if missing..." -ForegroundColor Cyan

$TestVNet = Get-AzVirtualNetwork `
    -ResourceGroupName $ResourceGroupName `
    -Name $TestVNetName `
    -ErrorAction SilentlyContinue

if (-not $TestVNet) {
    $TestSubnet = New-AzVirtualNetworkSubnetConfig `
        -Name $TestSubnetName `
        -AddressPrefix $TestSubnetPrefix

    $TestVNet = New-AzVirtualNetwork `
        -Name $TestVNetName `
        -ResourceGroupName $ResourceGroupName `
        -Location $Location `
        -AddressPrefix $TestAddressPrefix `
        -Subnet $TestSubnet
}

Write-Host "Azure ASR resource preparation summary:" -ForegroundColor Green

Get-AzRecoveryServicesVault -ResourceGroupName $ResourceGroupName |
    Select-Object Name, Location, ResourceGroupName, ID |
    Format-List

Get-AzVirtualNetwork -ResourceGroupName $ResourceGroupName |
    Select-Object Name, Location, ResourceGroupName, AddressSpace |
    Format-List

Get-AzStorageAccount -ResourceGroupName $ResourceGroupName |
    Select-Object StorageAccountName, Location, Sku, Kind, PrimaryLocation |
    Format-Table -AutoSize
~~~

# 26_Configure_Azure_Site_Recovery_For_Hyper-V_Hyper-V_Host_Readiness_Skeleton

~~~powershell
# 26_Configure_Azure_Site_Recovery_For_Hyper-V_Hyper-V_Host_Readiness_Skeleton
# Purpose:
# Validate Hyper-V host and VM readiness before ASR provider registration and replication enablement.

$VMName = "LAB-WIN-SRV01"

Write-Host "Checking Hyper-V host role and services..." -ForegroundColor Cyan

Get-WindowsFeature Hyper-V, Hyper-V-PowerShell |
    Select-Object Name, DisplayName, InstallState |
    Format-Table -AutoSize

Get-Service vmms, vmcompute -ErrorAction SilentlyContinue |
    Select-Object Name, DisplayName, Status, StartType |
    Format-Table -AutoSize

Write-Host "Checking source VM state..." -ForegroundColor Cyan

$VM = Get-VM -Name $VMName -ErrorAction Stop

$VM |
    Select-Object Name, State, Status, Generation, ProcessorCount, DynamicMemoryEnabled, MemoryStartup, MemoryAssigned, Path |
    Format-List

Write-Host "Checking VM disks..." -ForegroundColor Cyan

$HardDisks = Get-VMHardDiskDrive -VMName $VMName

$HardDisks |
    Select-Object VMName, ControllerType, ControllerNumber, ControllerLocation, Path |
    Format-Table -AutoSize

foreach ($Disk in $HardDisks) {
    if (-not (Test-Path $Disk.Path)) {
        Write-Warning "Disk path missing: $($Disk.Path)"
    }
    else {
        Get-VHD -Path $Disk.Path |
            Select-Object Path, VhdFormat, VhdType, FileSize, Size, ParentPath, Attached |
            Format-List
    }
}

Write-Host "Checking checkpoint state..." -ForegroundColor Cyan

$Checkpoints = Get-VMCheckpoint -VMName $VMName -ErrorAction SilentlyContinue

if ($Checkpoints) {
    Write-Warning "Checkpoints exist. Review before enabling ASR replication."
    $Checkpoints |
        Select-Object VMName, Name, CreationTime, SnapshotType, ParentSnapshotName |
        Format-Table -AutoSize
}
else {
    Write-Host "No checkpoints found." -ForegroundColor Green
}

Write-Host "Checking VM network state..." -ForegroundColor Cyan

Get-VMNetworkAdapter -VMName $VMName |
    Select-Object VMName, Name, SwitchName, MacAddress, Status, Connected, IPAddresses |
    Format-List

Write-Host "Checking integration services..." -ForegroundColor Cyan

Get-VMIntegrationService -VMName $VMName |
    Select-Object VMName, Name, Enabled, PrimaryStatusDescription, SecondaryStatusDescription |
    Format-Table -AutoSize

Write-Host "Checking host internet connectivity to Azure endpoints by generic HTTPS test..." -ForegroundColor Cyan

$ConnectivityTargets = @(
    "login.microsoftonline.com",
    "management.azure.com",
    "portal.azure.com"
)

foreach ($Target in $ConnectivityTargets) {
    Test-NetConnection $Target -Port 443 |
        Select-Object ComputerName, RemotePort, TcpTestSucceeded, ResolvedAddresses |
        Format-List
}

Write-Host "Checking local ASR/MARS installation state..." -ForegroundColor Cyan

Get-Service |
    Where-Object { $_.Name -match "Azure|ASR|MARS|obengine|dra" -or $_.DisplayName -match "Azure|Site Recovery|Recovery Services" } |
    Select-Object Name, DisplayName, Status, StartType |
    Format-Table -AutoSize

Get-ItemProperty "HKLM:\Software\Microsoft\Windows\CurrentVersion\Uninstall\*" -ErrorAction SilentlyContinue |
    Where-Object { $_.DisplayName -match "Azure Site Recovery|Microsoft Azure Recovery Services|MARS|Recovery Services" } |
    Select-Object DisplayName, DisplayVersion, Publisher, InstallDate |
    Format-Table -AutoSize

Write-Host "Hyper-V ASR readiness check completed." -ForegroundColor Green
~~~

# 26_Configure_Azure_Site_Recovery_For_Hyper-V_ASR_Provider_And_Registration_Skeleton

~~~powershell
# 26_Configure_Azure_Site_Recovery_For_Hyper-V_ASR_Provider_And_Registration_Skeleton
# Purpose:
# Install the Azure Site Recovery provider and register the Hyper-V host to a Recovery Services vault.
# Download the provider installer and vault registration key from:
# Recovery Services vault > Site Recovery > Hyper-V machines to Azure > Prepare infrastructure > Add Hyper-V server.

$InstallerPath = "C:\Install\AzureSiteRecoveryProvider.exe"
$ExtractPath = "C:\Install\ASRProvider"
$VaultKeyPath = "C:\Install\VaultCredentials.VaultCredentials"
$FriendlyName = $env:COMPUTERNAME

Write-Host "Validating ASR provider installer and vault key..." -ForegroundColor Cyan

if (-not (Test-Path $InstallerPath)) {
    throw "ASR provider installer not found: $InstallerPath"
}

if (-not (Test-Path $VaultKeyPath)) {
    throw "Vault registration key not found: $VaultKeyPath"
}

Write-Host "Creating extract path..." -ForegroundColor Cyan

New-Item -Path $ExtractPath -ItemType Directory -Force | Out-Null

Write-Host "Extracting ASR provider installer..." -ForegroundColor Cyan

Start-Process `
    -FilePath $InstallerPath `
    -ArgumentList "/x:$ExtractPath /q" `
    -Wait

Write-Host "Installing ASR provider silently if setupdr.exe exists..." -ForegroundColor Cyan

$SetupDr = Join-Path $ExtractPath "setupdr.exe"

if (Test-Path $SetupDr) {
    Start-Process `
        -FilePath $SetupDr `
        -ArgumentList "/i" `
        -Wait
}
else {
    Write-Warning "setupdr.exe not found after extraction. Run the GUI installer manually if needed."
}

Write-Host "Registering Hyper-V host with Recovery Services vault..." -ForegroundColor Cyan

$Configurator = "C:\Program Files\Microsoft Azure Site Recovery Provider\DRConfigurator.exe"

if (Test-Path $Configurator) {
    Start-Process `
        -FilePath $Configurator `
        -ArgumentList "/r /Friendlyname `"$FriendlyName`" /Credentials `"$VaultKeyPath`"" `
        -Wait
}
else {
    Write-Warning "DRConfigurator.exe not found. Complete registration through the Azure Site Recovery Registration Wizard."
}

Write-Host "Checking ASR/MARS services after install and registration..." -ForegroundColor Cyan

Get-Service |
    Where-Object { $_.Name -match "Azure|ASR|MARS|obengine|dra" -or $_.DisplayName -match "Azure|Site Recovery|Recovery Services" } |
    Select-Object Name, DisplayName, Status, StartType |
    Format-Table -AutoSize

Write-Host "Checking installed ASR/MARS programs..." -ForegroundColor Cyan

Get-ItemProperty "HKLM:\Software\Microsoft\Windows\CurrentVersion\Uninstall\*" -ErrorAction SilentlyContinue |
    Where-Object { $_.DisplayName -match "Azure Site Recovery|Microsoft Azure Recovery Services|MARS|Recovery Services" } |
    Select-Object DisplayName, DisplayVersion, Publisher, InstallDate |
    Format-Table -AutoSize

Write-Host "Review Azure portal: Recovery Services vault > Site Recovery Infrastructure > Hyper-V Hosts." -ForegroundColor Yellow
~~~

# 26_Configure_Azure_Site_Recovery_For_Hyper-V_Replication_Policy_And_Enablement_Skeleton

~~~powershell
# 26_Configure_Azure_Site_Recovery_For_Hyper-V_Replication_Policy_And_Enablement_Skeleton
# Purpose:
# Document the portal-driven replication policy and VM enablement workflow.
# Hyper-V-to-Azure ASR enablement is commonly performed from the Recovery Services vault wizard.

$VaultName = "rsv-hyperv-asr-lab"
$ResourceGroupName = "rg-asr-hyperv-lab"
$HyperVSiteName = "HV-Site-01"
$PolicyName = "pol-hyperv-5min"
$VMName = "LAB-WIN-SRV01"

Write-Host "Azure portal workflow to create or associate replication policy:" -ForegroundColor Cyan

@"
1. Go to Azure portal.
2. Open Recovery Services vault: $VaultName.
3. Select Enable Site Recovery.
4. Under Hyper-V machines to Azure, select Prepare infrastructure.
5. Source settings:
   - Are you using System Center VMM? No, unless your hosts are VMM managed.
   - Hyper-V site: $HyperVSiteName.
   - Add registered Hyper-V server if not already listed.
6. Target settings:
   - Subscription: planned subscription.
   - Target resource group: planned failover VM resource group.
   - Deployment model: Resource Manager.
7. Replication policy:
   - Create new policy if needed.
   - Name: $PolicyName.
   - Copy frequency: 5 minutes unless design requires otherwise.
   - Recovery point retention: match RPO and storage plan.
   - App-consistent snapshot frequency: match application support.
   - Initial replication start: Immediately or scheduled.
8. Review and create.
"@

Write-Host "Azure portal workflow to enable VM replication:" -ForegroundColor Cyan

@"
1. Open Recovery Services vault: $VaultName.
2. Select Enable Site Recovery.
3. Under Hyper-V machines to Azure, select Enable replication.
4. Source environment:
   - Source location / Hyper-V site: $HyperVSiteName.
5. Target environment:
   - Target subscription.
   - Post-failover resource group.
   - Target storage / managed disk setting.
   - Target virtual network and subnet.
6. Virtual machine selection:
   - Select VM: $VMName.
7. Replication settings:
   - Review OS disk and data disks.
   - Exclude disks only if approved.
8. Replication policy:
   - Select policy: $PolicyName.
9. Review and select Enable Replication.
10. Track progress in vault jobs and replicated item health.
"@

Write-Host "Optional Azure PowerShell vault context check:" -ForegroundColor Cyan

try {
    $Vault = Get-AzRecoveryServicesVault -ResourceGroupName $ResourceGroupName -Name $VaultName -ErrorAction Stop
    Set-AzRecoveryServicesAsrVaultContext -Vault $Vault

    Get-AzRecoveryServicesAsrFabric -ErrorAction SilentlyContinue |
        Select-Object Name, FriendlyName, FabricType, FabricSpecificDetails |
        Format-List

    Get-AzRecoveryServicesAsrPolicy -ErrorAction SilentlyContinue |
        Select-Object Name, ReplicationProvider, ReplicationFrequencyInSeconds, RecoveryPointRetentionInHours, ApplicationConsistentSnapshotFrequencyInHours |
        Format-List
}
catch {
    Write-Warning "Az.RecoveryServices context check failed. Confirm sign-in, subscription, vault, and module version."
    $_
}
~~~

# 26_Configure_Azure_Site_Recovery_For_Hyper-V_Compute_Network_And_Disk_Settings_Skeleton

~~~powershell
# 26_Configure_Azure_Site_Recovery_For_Hyper-V_Compute_Network_And_Disk_Settings_Skeleton
# Purpose:
# Review and document post-failover compute, network, disk, and access settings for a replicated Hyper-V VM.

$VaultName = "rsv-hyperv-asr-lab"
$VMName = "LAB-WIN-SRV01"

$TargetSettings = [ordered]@{
    ReplicatedItem = $VMName
    TargetResourceGroup = "rg-asr-failover-vms"
    TargetVMName = "az-$VMName"
    TargetVMSize = "Standard_D2s_v5"
    UseManagedDisks = $true
    TargetVNet = "vnet-asr-prod"
    TargetSubnet = "snet-asr-prod"
    TestFailoverVNet = "vnet-asr-test"
    TestFailoverSubnet = "snet-asr-test"
    PrivateIPPlan = "Dynamic or static per DR design"
    NSG = "nsg-asr-test-or-prod"
    BootDiagnostics = "Enabled"
    AvailabilityOption = "None or Availability Set or Zone"
    DNSCutover = "Manual lab validation"
}

Write-Host "ASR Compute and Network review checklist for $VMName:" -ForegroundColor Cyan

$TargetSettings.GetEnumerator() |
    ForEach-Object {
        "{0}: {1}" -f $_.Key, $_.Value
    }

Write-Host "Portal workflow:" -ForegroundColor Cyan

@"
1. Open Recovery Services vault: $VaultName.
2. Go to Protected items > Replicated items.
3. Select replicated VM: $VMName.
4. Open Compute and Network.
5. Review or configure:
   - Azure VM name.
   - Resource group.
   - Target VM size.
   - Managed disk setting.
   - Target virtual network and subnet.
   - Test failover network.
   - Network interface IP settings.
   - Availability set or zone if used.
6. Open Disks.
7. Review:
   - OS disk classification.
   - Data disks.
   - Disk exclusion decisions.
   - Target disk type.
8. Save settings and capture screenshot or export notes.
"@

Write-Host "Optional Azure resource validation:" -ForegroundColor Cyan

Get-AzResourceGroup -Name $TargetSettings.TargetResourceGroup -ErrorAction SilentlyContinue |
    Select-Object ResourceGroupName, Location, ProvisioningState |
    Format-List

Get-AzVirtualNetwork -Name $TargetSettings.TargetVNet -ErrorAction SilentlyContinue |
    Select-Object Name, ResourceGroupName, Location, AddressSpace |
    Format-List

Get-AzVirtualNetwork -Name $TargetSettings.TestFailoverVNet -ErrorAction SilentlyContinue |
    Select-Object Name, ResourceGroupName, Location, AddressSpace |
    Format-List
~~~

# 26_Configure_Azure_Site_Recovery_For_Hyper-V_Test_Failover_Drill_Skeleton

~~~powershell
# 26_Configure_Azure_Site_Recovery_For_Hyper-V_Test_Failover_Drill_Skeleton
# Purpose:
# Document and execute an isolated ASR test failover drill.
# Test failover should use an isolated Azure VNet, not the production recovery VNet.

$VaultName = "rsv-hyperv-asr-lab"
$VMName = "LAB-WIN-SRV01"
$TestVNetName = "vnet-asr-test"
$TestSubnetName = "snet-asr-test"
$DrillNotesPath = "C:\Admin\HyperV-ASR\TestFailover-Notes.txt"

New-Item -Path (Split-Path $DrillNotesPath -Parent) -ItemType Directory -Force | Out-Null

Write-Host "Pre-drill validation checklist:" -ForegroundColor Cyan

@"
1. Confirm replicated item health is Healthy.
2. Confirm at least one recovery point exists.
3. Confirm test VNet is isolated from production.
4. Confirm NSG allows only approved admin access.
5. Confirm DNS behavior for test network.
6. Confirm no production IP conflict will occur.
7. Confirm application owner approved test.
8. Confirm cleanup owner and rollback decision.
"@ |
Tee-Object -FilePath $DrillNotesPath

Write-Host "Azure portal test failover workflow:" -ForegroundColor Cyan

@"
1. Open Recovery Services vault: $VaultName.
2. Go to Protected items > Replicated items.
3. Select VM: $VMName.
4. Select Test Failover.
5. Recovery point:
   - Latest processed for lowest drill RTO, or
   - Latest app-consistent if application validation requires it.
6. Azure virtual network:
   - Select isolated test VNet: $TestVNetName.
   - Confirm subnet: $TestSubnetName.
7. Select OK.
8. Track job:
   - Vault > Site Recovery jobs.
9. After VM is created:
   - Confirm Azure VM size.
   - Confirm network and subnet.
   - Confirm boot diagnostics.
   - Connect by RDP, SSH, Bastion, or serial console.
10. Validate application or OS boot.
11. Record observations.
12. Select Cleanup test failover.
13. Enter test notes and confirm cleanup.
"@ |
Tee-Object -FilePath $DrillNotesPath -Append

Write-Host "Optional Azure PowerShell observation commands after test VM creation:" -ForegroundColor Cyan

@"
Get-AzVM | Where-Object Name -like '*$VMName*' | Select Name, ResourceGroupName, Location, ProvisioningState
Get-AzNetworkInterface | Where-Object { `$_.VirtualMachine -ne `$null } | Select Name, ResourceGroupName, Location
Get-AzPublicIpAddress | Select Name, ResourceGroupName, IpAddress, PublicIpAllocationMethod
"@ |
Tee-Object -FilePath $DrillNotesPath -Append

Write-Host "Test failover drill notes written to: $DrillNotesPath" -ForegroundColor Green
~~~

# 26_Configure_Azure_Site_Recovery_For_Hyper-V_Planned_Failover_And_Failback_Skeleton

~~~powershell
# 26_Configure_Azure_Site_Recovery_For_Hyper-V_Planned_Failover_And_Failback_Skeleton
# Purpose:
# Document planned failover, commit, re-protect, and failback workflow.
# Do not run planned failover without a maintenance window and application owner approval.

$VaultName = "rsv-hyperv-asr-lab"
$VMName = "LAB-WIN-SRV01"
$RunbookPath = "C:\Admin\HyperV-ASR\PlannedFailover-Failback-Runbook.txt"

New-Item -Path (Split-Path $RunbookPath -Parent) -ItemType Directory -Force | Out-Null

@"
# Planned Failover Runbook - $VMName

Pre-checks:
1. Confirm business approval and maintenance window.
2. Confirm replicated item health is Healthy.
3. Confirm latest recovery point age meets RPO.
4. Confirm source VM application quiescence plan.
5. Confirm on-premises backup status.
6. Confirm Azure target VNet, subnet, NSG, DNS, and VM size.
7. Confirm credentials and access path to Azure VM.
8. Confirm rollback decision point and owner.

Planned failover workflow:
1. Open Recovery Services vault: $VaultName.
2. Go to Protected items > Replicated items.
3. Select VM: $VMName.
4. Select Failover.
5. Choose planned failover if source is available.
6. Choose recovery point based on DR runbook.
7. Start failover job.
8. Monitor Site Recovery jobs.
9. Validate Azure VM boot, network, identity, and application state.
10. If accepted, Commit failover.
11. Update DNS, routing, firewall, and application dependencies as required.

After failover:
1. Confirm source VM shutdown or access state.
2. Confirm Azure VM is protected by backup or operational monitoring.
3. Confirm application owner acceptance.
4. Document recovery point and actual RTO/RPO.

Re-protect and failback planning:
1. After primary site is healthy, select Re-protect.
2. Configure reverse replication from Azure back to Hyper-V.
3. Allow reverse replication to complete.
4. Schedule failback maintenance window.
5. Run planned failback.
6. Validate on-premises VM.
7. Commit failback.
8. Re-enable protection from Hyper-V to Azure.
"@ |
Out-File -FilePath $RunbookPath -Encoding UTF8

Write-Host "Planned failover and failback runbook written to: $RunbookPath" -ForegroundColor Green

Get-Content $RunbookPath
~~~

# 26_Configure_Azure_Site_Recovery_For_Hyper-V_PostChange_Verification_Skeleton

~~~powershell
# 26_Configure_Azure_Site_Recovery_For_Hyper-V_PostChange_Verification_Skeleton
# Purpose:
# Capture ASR, Azure, Hyper-V host, provider, VM, replication, event, and test failover evidence after configuration.

$EvidenceRoot = "C:\Admin\HyperV-ASR"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$VMName = "LAB-WIN-SRV01"
$VaultName = "rsv-hyperv-asr-lab"
$ResourceGroupName = "rg-asr-hyperv-lab"
$OutputPath = Join-Path $EvidenceRoot "PostChange-$VMName-$Timestamp"

New-Item -Path $OutputPath -ItemType Directory -Force | Out-Null

Write-Host "Capturing Hyper-V host and VM state..." -ForegroundColor Cyan

Get-ComputerInfo |
    Select-Object CsName, CsDomain, CsPartOfDomain, WindowsProductName, OsBuildNumber |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "01-Host-ComputerInfo-After.txt")

Get-VM -Name $VMName -ErrorAction SilentlyContinue |
    Select-Object Name, Id, State, Status, Generation, Uptime, ProcessorCount, MemoryAssigned, Path |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "02-VM-Summary-After.txt")

Get-VMHardDiskDrive -VMName $VMName -ErrorAction SilentlyContinue |
    Select-Object VMName, ControllerType, ControllerNumber, ControllerLocation, Path |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "03-VM-Disks-After.txt")

Get-VMNetworkAdapter -VMName $VMName -ErrorAction SilentlyContinue |
    Select-Object VMName, Name, SwitchName, MacAddress, Status, Connected, IPAddresses |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "04-VM-Network-After.txt")

Write-Host "Capturing ASR and MARS local install state..." -ForegroundColor Cyan

Get-Service |
    Where-Object { $_.Name -match "Azure|ASR|MARS|obengine|dra" -or $_.DisplayName -match "Azure|Site Recovery|Recovery Services" } |
    Select-Object Name, DisplayName, Status, StartType |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "05-ASR-MARS-Services-After.txt")

Get-ItemProperty "HKLM:\Software\Microsoft\Windows\CurrentVersion\Uninstall\*" -ErrorAction SilentlyContinue |
    Where-Object { $_.DisplayName -match "Azure Site Recovery|Microsoft Azure Recovery Services|MARS|Recovery Services" } |
    Select-Object DisplayName, DisplayVersion, Publisher, InstallDate |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "06-ASR-MARS-InstalledPrograms-After.txt")

Write-Host "Capturing Azure vault and ASR state if Az context is available..." -ForegroundColor Cyan

try {
    $Vault = Get-AzRecoveryServicesVault -ResourceGroupName $ResourceGroupName -Name $VaultName -ErrorAction Stop

    $Vault |
        Select-Object Name, Location, ResourceGroupName, ID |
        Format-List |
        Tee-Object -FilePath (Join-Path $OutputPath "07-RecoveryServicesVault-After.txt")

    Set-AzRecoveryServicesAsrVaultContext -Vault $Vault

    Get-AzRecoveryServicesAsrFabric -ErrorAction SilentlyContinue |
        Format-List * |
        Out-File -FilePath (Join-Path $OutputPath "08-ASR-Fabrics-After.txt") -Encoding UTF8

    Get-AzRecoveryServicesAsrPolicy -ErrorAction SilentlyContinue |
        Format-List * |
        Out-File -FilePath (Join-Path $OutputPath "09-ASR-Policies-After.txt") -Encoding UTF8

    Get-AzRecoveryServicesAsrJob -ErrorAction SilentlyContinue |
        Select-Object Name, DisplayName, State, StateDescription, StartTime, EndTime, TargetObjectName |
        Format-Table -AutoSize |
        Out-File -FilePath (Join-Path $OutputPath "10-ASR-Jobs-After.txt") -Encoding UTF8

    Get-AzVirtualNetwork -ResourceGroupName $ResourceGroupName -ErrorAction SilentlyContinue |
        Select-Object Name, Location, ResourceGroupName, AddressSpace |
        Format-List |
        Out-File -FilePath (Join-Path $OutputPath "11-Azure-VNets-After.txt") -Encoding UTF8

    Get-AzVM -ErrorAction SilentlyContinue |
        Where-Object { $_.Name -like "*$VMName*" } |
        Select-Object Name, ResourceGroupName, Location, ProvisioningState, VmId |
        Format-Table -AutoSize |
        Out-File -FilePath (Join-Path $OutputPath "12-Azure-VMs-Matching-After.txt") -Encoding UTF8
}
catch {
    $_ |
        Out-File -FilePath (Join-Path $OutputPath "07-Azure-ASR-State-Error.txt") -Encoding UTF8
}

Write-Host "Capturing Hyper-V and ASR-related event logs..." -ForegroundColor Cyan

$Logs = @(
    "Microsoft-Windows-Hyper-V-VMMS-Admin",
    "Microsoft-Windows-Hyper-V-Worker-Admin",
    "System",
    "Application"
)

foreach ($Log in $Logs) {
    $SafeLog = $Log -replace '[\\\/]', "-"

    Get-WinEvent -LogName $Log -MaxEvents 150 -ErrorAction SilentlyContinue |
        Select-Object TimeCreated, Id, LevelDisplayName, ProviderName, Message |
        Format-List |
        Out-File -FilePath (Join-Path $OutputPath "13-$SafeLog-After.txt") -Encoding UTF8
}

Write-Host "Post-change ASR evidence exported to: $OutputPath" -ForegroundColor Green
~~~

# 26_Configure_Azure_Site_Recovery_For_Hyper-V_Verification_Commands

| Command | Where to Run | What It Confirms |
|---|---|---|
| `Connect-AzAccount` | Admin workstation | Authenticates to Azure |
| `Set-AzContext -Subscription '<subscription>'` | Admin workstation | Selects target subscription |
| `Get-AzRecoveryServicesVault` | Admin workstation | Lists Recovery Services vaults |
| `Set-AzRecoveryServicesAsrVaultContext -Vault '<vault-object>'` | Admin workstation | Sets ASR vault context |
| `Get-AzRecoveryServicesAsrFabric` | Admin workstation | Lists ASR fabrics such as Hyper-V sites |
| `Get-AzRecoveryServicesAsrPolicy` | Admin workstation | Lists ASR replication policies |
| `Get-AzRecoveryServicesAsrJob` | Admin workstation | Shows ASR job state |
| `Get-AzVirtualNetwork` | Admin workstation | Confirms target and test VNets |
| `Get-AzStorageAccount` | Admin workstation | Confirms storage account state |
| `Get-AzVM` | Admin workstation | Shows Azure VMs created after failover or test failover |
| `Get-WindowsFeature Hyper-V` | Hyper-V host | Confirms Hyper-V role |
| `Get-VM -Name '<vm-name>'` | Hyper-V host | Confirms source VM state |
| `Get-VMHardDiskDrive -VMName '<vm-name>'` | Hyper-V host | Shows VM disk paths |
| `Get-VHD -Path '<vhdx-path>'` | Hyper-V host | Shows VHDX type, size, and parent chain |
| `Get-VMNetworkAdapter -VMName '<vm-name>'` | Hyper-V host | Shows source VM network adapter state |
| `Get-VMIntegrationService -VMName '<vm-name>'` | Hyper-V host | Shows guest integration service state |
| `Get-VMCheckpoint -VMName '<vm-name>'` | Hyper-V host | Shows checkpoints that may affect replication or consistency |
| `Test-NetConnection login.microsoftonline.com -Port 443` | Hyper-V host | Confirms generic Azure auth endpoint reachability |
| `Test-NetConnection management.azure.com -Port 443` | Hyper-V host | Confirms generic Azure management endpoint reachability |
| `AzureSiteRecoveryProvider.exe /x:. /q` | Hyper-V Core host | Extracts ASR provider installer |
| `.\setupdr.exe /i` | Hyper-V Core host | Installs ASR provider from extracted files |
| `"C:\Program Files\Microsoft Azure Site Recovery Provider\DRConfigurator.exe" /r /Friendlyname '<host>' /Credentials '<vault-key-path>'` | Hyper-V host | Registers host with vault by credential file |
| `Get-Service \| Where-Object DisplayName -match 'Azure|Site Recovery|Recovery Services'` | Hyper-V host | Shows ASR and MARS service state |
| `Get-WinEvent -LogName 'Microsoft-Windows-Hyper-V-VMMS-Admin' -MaxEvents 20` | Hyper-V host | Checks Hyper-V management events |
| `Get-WinEvent -LogName 'Application' -MaxEvents 50` | Hyper-V host | Checks application-level ASR provider events where logged |

# 26_Configure_Azure_Site_Recovery_For_Hyper-V_Rollback

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Capture ASR state before rollback | Admin workstation and Hyper-V host | Run post-change verification skeleton | Current state is documented |
| 2 | Cleanup test failover if active | Azure portal | Replicated item > Cleanup test failover | Test Azure VM removed |
| 3 | Disable replication for lab VM if rollback requires it | Azure portal | Replicated item > Disable replication | VM no longer protected by ASR |
| 4 | Remove stale Azure test VM if cleanup failed | Admin workstation | `Remove-AzVM -Name '<test-vm>' -ResourceGroupName '<rg>' -Force` | Test VM removed |
| 5 | Remove test NICs if orphaned | Admin workstation | `Remove-AzNetworkInterface -Name '<nic>' -ResourceGroupName '<rg>' -Force` | Orphaned NIC removed |
| 6 | Remove test disks if orphaned | Admin workstation | `Remove-AzDisk -DiskName '<disk>' -ResourceGroupName '<rg>' -Force` | Orphaned managed disk removed |
| 7 | Remove test public IPs if orphaned | Admin workstation | `Remove-AzPublicIpAddress -Name '<pip>' -ResourceGroupName '<rg>' -Force` | Orphaned public IP removed |
| 8 | Remove ASR provider only if host is no longer protected | Hyper-V host | Programs and Features or approved uninstall command | Provider removed |
| 9 | Remove MARS agent only if no longer used | Hyper-V host | Programs and Features or approved uninstall command | Agent removed |
| 10 | Remove Azure test VNet if lab cleanup requires it | Admin workstation | `Remove-AzVirtualNetwork -Name '<test-vnet>' -ResourceGroupName '<rg>' -Force` | Test network removed |
| 11 | Remove Recovery Services vault only after all protected items are removed | Admin workstation | `Remove-AzRecoveryServicesVault -Vault '<vault-object>'` | Vault removed |
| 12 | Remove resource group only if all lab resources are disposable | Admin workstation | `Remove-AzResourceGroup -Name '<rg>' -Force` | Lab resource group removed |
| 13 | Capture rollback evidence | Admin workstation and Hyper-V host | Run post-change verification skeleton | Rollback state documented |

# 26_Configure_Azure_Site_Recovery_For_Hyper-V_Failure_Checks

| Symptom | Likely Cause | Fix |
|---|---|---|
| Hyper-V host does not appear in vault | Provider not installed, registration failed, wrong vault key, or metadata delay | Recheck provider install, vault key, site selection, and wait for discovery |
| Vault key rejected | Expired key or wrong vault/site | Download a fresh registration key from the correct vault |
| Provider install fails | Missing prerequisites, old version, permissions, or pending reboot | Reboot host, run as administrator, update OS, reinstall provider |
| MARS agent outdated | Old bundled agent or failed update | Update MARS agent without re-registering host |
| Host cannot reach Azure | Firewall, proxy, DNS, TLS inspection, or no internet access | Validate outbound HTTPS and required Azure URLs |
| VM not listed for replication | Host not registered, metadata not refreshed, unsupported VM, or VM off/in bad state | Refresh discovery, check VM compatibility, and review host state |
| Enable replication fails | Unsupported disk, unsupported OS, storage issue, or target configuration issue | Review ASR error details, disk layout, and Azure target settings |
| Initial replication stalls | Bandwidth, large disks, throttling, provider issue, or host connectivity | Review ASR jobs, host logs, bandwidth, and provider services |
| Replication health warning | RPO breach, app-consistent failure, network issue, or provider issue | Review replicated item health, RPO, and latest job details |
| App-consistent snapshots fail | Guest VSS problem or integration service issue | Check guest VSS writers, integration services, and application logs |
| Recovery point missing | Replication not complete or policy retention too short | Wait for initial replication and verify policy |
| Test failover VM has no network | Wrong test VNet, subnet, NSG, or IP settings | Review Compute and Network settings and test failover VNet |
| Test failover connects to production network | Wrong VNet selected | Cleanup immediately and rerun with isolated test VNet |
| Test VM will not boot | Azure VM compatibility, boot driver, disk, or OS issue | Review boot diagnostics, serial console, and ASR job details |
| RDP or SSH fails after failover | NSG, firewall, guest service, IP, or credentials issue | Validate NSG, guest firewall, NIC IP, and access method |
| Failover creates wrong VM size | Compute and Network settings not reviewed | Edit target VM size before failover |
| Failover disk type wrong | Managed disk setting or replication storage selection not reviewed | Update protected item Compute and Network disk setting |
| Planned failover unavailable | Source VM or provider state not healthy | Use proper failover mode and fix source health |
| Commit selected too early | Application validation incomplete | Validate application before commit |
| Failback blocked | MARS/provider version, network, Hyper-V capacity, or reverse replication issue | Update agent/provider and validate failback prerequisites |
| Cleanup test failover leaves orphaned resources | Job failed or manual deletion interrupted cleanup | Remove orphaned VM, NIC, disk, public IP, and NSG resources carefully |
| Vault cannot be deleted | Protected items, fabrics, policies, or jobs still exist | Remove protection and dependencies first |

# 26_Configure_Azure_Site_Recovery_For_Hyper-V_Related_Labs

| Lab | Relationship |
|---|---|
| 00_Hyper-V_Index.md | Places Azure Site Recovery in the full Hyper-V suite |
| 01_Confirm_Hyper-V_Host_Baseline.md | Confirms Hyper-V host readiness before ASR registration |
| 02_Install_Hyper-V_Role_And_Management_Tools.md | Required before Hyper-V VM discovery and provider use |
| 03_Configure_Hyper-V_Host_Default_Paths_And_Settings.md | Host paths and VHDX location affect ASR discovery and replication |
| 04_Create_And_Configure_Virtual_Switches.md | Source VM networking must map cleanly to Azure target networks |
| 05_Create_Generation_1_And_Generation_2_VMs.md | VM generation affects Azure compatibility and boot behavior |
| 07_Create_Attach_Resize_And_Compact_VHDX_Disks.md | Disk layout and VHDX health affect replication and failover |
| 08_Configure_VM_Network_Adapters_VLANs_MAC_And_Bandwidth.md | NIC and VLAN settings inform target network mapping |
| 09_Install_Guest_OS_And_Configure_Integration_Services.md | Guest integration and VSS affect app-consistent recovery points |
| 10_Configure_Checkpoints_Production_Checkpoints_And_Restore.md | Checkpoint state should be reviewed before replication |
| 11_Manage_VM_Lifecycle_Start_Stop_Save_Reset_Export_Import.md | VM lifecycle control is required during planned failover and failback |
| 12_Configure_PowerShell_Direct_And_Guest_Management.md | PowerShell Direct helps prepare source guest access before failover |
| 13_Configure_VM_Backup_And_Recovery_Workflows.md | Backup remains required because ASR is DR replication, not backup replacement |
| 14_Configure_Live_Migration_And_Storage_Migration.md | Migration may be needed before protecting workloads |
| 15_Configure_Hyper-V_Replica_And_Failover_Testing.md | Hyper-V Replica concepts map to ASR RPO, recovery point, failover, and test failover thinking |
| 16_Configure_Hyper-V_Failover_Clustering_And_Clustered_VMs.md | Clustered Hyper-V workloads require careful ASR support and ownership review |
| 17_Monitor_Hyper-V_Hosts_VMs_Events_Performance_And_Resource_Usage.md | Monitoring validates host, VM, provider, and replication health |
| 18_Troubleshoot_Hyper-V_VM_Network_Storage_Checkpoint_And_Startup_Failures.md | Troubleshooting supports failed boot, failed test failover, and network issues |
| 23_Create_VM_Templates_Differencing_Disks_And_Golden_Images.md | Template consistency helps create ASR-ready lab VMs |
| 24_Configure_Hyper-V_With_Storage_Spaces_Direct_And_CSV.md | S2D and CSV-backed VMs require extra storage and cluster planning before ASR |
| 25_Configure_Hyper-V_Converged_Networking_For_Cluster_Hosts.md | Reliable host networking is required for replication, failover validation, and failback |