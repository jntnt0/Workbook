# 09_Install_Guest_OS_And_Configure_Integration_Services

# 09_Install_Guest_OS_And_Configure_Integration_Services_Index
09_Install_Guest_OS_And_Configure_Integration_Services.md
09_Install_Guest_OS_And_Configure_Integration_Services
09_Install_Guest_OS_And_Configure_Integration_Services_Source_Basis
09_Install_Guest_OS_And_Configure_Integration_Services_Mental_Model
09_Install_Guest_OS_And_Configure_Integration_Services_Planning_Table
09_Install_Guest_OS_And_Configure_Integration_Services_Configuration_Checklist
09_Install_Guest_OS_And_Configure_Integration_Services_PreInstall_Capture_Skeleton
09_Install_Guest_OS_And_Configure_Integration_Services_Windows_Guest_Install_Skeleton
09_Install_Guest_OS_And_Configure_Integration_Services_Linux_Guest_Install_Skeleton
09_Install_Guest_OS_And_Configure_Integration_Services_Windows_Guest_PostInstall_Skeleton
09_Install_Guest_OS_And_Configure_Integration_Services_Linux_Guest_PostInstall_Skeleton
09_Install_Guest_OS_And_Configure_Integration_Services_Integration_Services_Verification_Skeleton
09_Install_Guest_OS_And_Configure_Integration_Services_PostInstall_Verification_Skeleton
09_Install_Guest_OS_And_Configure_Integration_Services_Verification_Commands
09_Install_Guest_OS_And_Configure_Integration_Services_Rollback
09_Install_Guest_OS_And_Configure_Integration_Services_Failure_Checks
09_Install_Guest_OS_And_Configure_Integration_Services_Related_Labs

# 09_Install_Guest_OS_And_Configure_Integration_Services_Source_Basis

| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Hyper-V supported Windows guest operating systems | Supports guest OS selection and compatibility validation |
| Microsoft Learn | Hyper-V supported Linux and FreeBSD virtual machines | Supports Linux guest compatibility and Linux integration services behavior |
| Microsoft Learn | Hyper-V Integration Services | Supports heartbeat, time sync, data exchange, shutdown, VSS, and guest service interface configuration |
| Microsoft Learn | Hyper-V PowerShell `Get-VMIntegrationService` and `Enable-VMIntegrationService` | Supports verifying and enabling integration services |
| Microsoft Learn | Hyper-V PowerShell `Add-VMDvdDrive`, `Set-VMFirmware`, and `Set-VMBios` | Supports ISO attachment and boot order configuration |
| Microsoft Learn | PowerShell Direct | Supports managing supported Windows guests without network connectivity |
| Internal Workbook Standard | Windows Server workbook structure | Keeps this workbook aligned with the Hyper-V, AD, DNS, DHCP, GPO, and File Services format |

# 09_Install_Guest_OS_And_Configure_Integration_Services_Mental_Model

| Concept | Operational Meaning |
|---|---|
| Guest OS | The operating system installed inside the VM |
| VM shell | The VM object, CPU, memory, disk, and network adapter created before OS installation |
| Installer ISO | Bootable OS media attached to the VM DVD drive |
| Boot order | Firmware or BIOS setting that determines whether the VM boots from DVD, disk, or network |
| Generation 1 guest | BIOS based VM that normally boots from IDE or legacy DVD devices |
| Generation 2 guest | UEFI based VM that supports Secure Boot and SCSI boot disks |
| Secure Boot template | Generation 2 setting that must match Windows or supported Linux guests |
| Integration Services | Hyper-V guest components that improve management, time sync, shutdown, heartbeat, backup, and data exchange |
| Heartbeat | Integration component that reports whether the guest OS is responsive |
| Time synchronization | Keeps guest time aligned with the Hyper-V host unless disabled for domain controller or special timing scenarios |
| Guest service interface | Allows file copy operations from host to guest when enabled |
| Data exchange | Allows key-value metadata exchange between host and guest |
| VSS integration | Supports coordinated backup behavior for supported guests |
| PowerShell Direct | Allows host to run PowerShell inside supported Windows guests without guest network connectivity |
| Guest tools state | Windows guests usually include modern integration components, while Linux integration is kernel and distribution dependent |
| Rollback boundary | This workbook installs and validates a guest OS and integration services, not production application configuration |

# 09_Install_Guest_OS_And_Configure_Integration_Services_Planning_Table

| Item | Example | Decision |
|---|---|---|
| Hyper-V host name | `HV01` | `<host-name>` |
| VM name | `LAB-WIN-SRV01` | `<vm-name>` |
| VM generation | Generation 1 or Generation 2 | `<generation>` |
| Guest OS | Windows Server 2022, Windows 11, Ubuntu Server | `<guest-os>` |
| ISO path | `D:\Hyper-V\ISO\WindowsServer.iso` | `<iso-path>` |
| VM switch | `vSwitch-External`, `vSwitch-Internal-Lab` | `<switch-name>` |
| Boot type | UEFI, BIOS | `<boot-type>` |
| Secure Boot | Enabled or disabled | `<enabled-disabled>` |
| Secure Boot template | `MicrosoftWindows`, `MicrosoftUEFICertificateAuthority` | `<secure-boot-template>` |
| OS disk path | `D:\Hyper-V\Virtual Hard Disks\LAB-WIN-SRV01.vhdx` | `<os-vhdx-path>` |
| Guest hostname | `LAB-WIN-SRV01` | `<guest-hostname>` |
| Guest local admin | `Administrator`, `localadmin` | `<guest-admin>` |
| Guest IP mode | DHCP or static | `<dhcp-static>` |
| Guest IP address | `192.168.10.51/24` | `<guest-ip>` |
| Guest default gateway | `192.168.10.1` | `<guest-gateway>` |
| Guest DNS servers | `192.168.10.10`, `192.168.10.11` | `<guest-dns>` |
| Time sync policy | Enabled, disabled for special roles | `<time-sync-policy>` |
| Guest service interface | Enabled or disabled | `<enabled-disabled>` |
| PowerShell Direct required | Yes or No | `<yes-no>` |
| Evidence path | `C:\Admin\HyperV-GuestInstall` | `<evidence-path>` |
| Rollback required | Yes or No | `<rollback-required>` |

# 09_Install_Guest_OS_And_Configure_Integration_Services_Configuration_Checklist

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm Hyper-V role is installed | Hyper-V host | `Get-WindowsFeature Hyper-V` | Hyper-V shows installed |
| 2 | Confirm target VM exists | Hyper-V host | `Get-VM -Name '<vm-name>'` | VM object exists |
| 3 | Confirm VM generation and state | Hyper-V host | `Get-VM -Name '<vm-name>' \| Select Name, Generation, State` | VM generation and power state are known |
| 4 | Confirm VM has a boot disk | Hyper-V host | `Get-VMHardDiskDrive -VMName '<vm-name>'` | OS VHDX is attached |
| 5 | Confirm VM has a network adapter | Hyper-V host | `Get-VMNetworkAdapter -VMName '<vm-name>'` | VM network adapter exists |
| 6 | Confirm installer ISO exists | Hyper-V host | `Test-Path '<iso-path>'` | ISO file exists |
| 7 | Capture pre-install VM state | Hyper-V host | Run pre-install capture skeleton | VM, disk, firmware, DVD, and network state are documented |
| 8 | Attach installer ISO | Hyper-V host | `Add-VMDvdDrive -VMName '<vm-name>' -Path '<iso-path>'` | Installer ISO is attached |
| 9 | Configure Generation 2 DVD boot | Hyper-V host | `$dvd = Get-VMDvdDrive -VMName '<vm-name>'; Set-VMFirmware -VMName '<vm-name>' -FirstBootDevice $dvd` | VM boots from ISO first |
| 10 | Configure Generation 1 DVD boot | Hyper-V host | `Set-VMBios -VMName '<vm-name>' -StartupOrder CD,IDE,LegacyNetworkAdapter,Floppy` | VM boots from CD first |
| 11 | Configure Windows Secure Boot if applicable | Hyper-V host | `Set-VMFirmware -VMName '<vm-name>' -EnableSecureBoot On -SecureBootTemplate MicrosoftWindows` | Windows Generation 2 Secure Boot is enabled |
| 12 | Configure Linux Secure Boot if applicable | Hyper-V host | `Set-VMFirmware -VMName '<vm-name>' -EnableSecureBoot On -SecureBootTemplate MicrosoftUEFICertificateAuthority` | Linux Generation 2 Secure Boot template is set |
| 13 | Start VM | Hyper-V host | `Start-VM -Name '<vm-name>'` | VM powers on |
| 14 | Open console and install OS | Hyper-V host | `vmconnect localhost '<vm-name>'` | Guest OS installer launches |
| 15 | Complete guest OS setup | Guest VM console | Manual installer steps | Guest OS installs to VHDX |
| 16 | Eject ISO after installation | Hyper-V host | `Set-VMDvdDrive -VMName '<vm-name>' -Path $null` | VM no longer boots back into installer |
| 17 | Set boot disk first after OS install | Hyper-V host | `Set-VMFirmware` or `Set-VMBios` | VM boots from installed OS disk |
| 18 | Confirm integration services state | Hyper-V host | `Get-VMIntegrationService -VMName '<vm-name>'` | Integration services are visible |
| 19 | Enable required integration services | Hyper-V host | `Enable-VMIntegrationService -VMName '<vm-name>' -Name '<service-name>'` | Required services are enabled |
| 20 | Configure guest hostname and basic IP settings | Guest VM | Run post-install guest skeleton | Guest identity and network are configured |
| 21 | Confirm heartbeat and guest responsiveness | Hyper-V host | `Get-VMIntegrationService -VMName '<vm-name>' -Name Heartbeat` | Heartbeat reports healthy state |
| 22 | Capture post-install evidence | Hyper-V host | Run post-install verification skeleton | Guest installation and integration evidence is saved |

# 09_Install_Guest_OS_And_Configure_Integration_Services_PreInstall_Capture_Skeleton

~~~powershell
# 09_Install_Guest_OS_And_Configure_Integration_Services_PreInstall_Capture_Skeleton
# Purpose:
# Capture VM hardware, firmware, disk, DVD, network, and integration state before guest OS installation.

$EvidenceRoot = "C:\Admin\HyperV-GuestInstall"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$VMName = "LAB-WIN-SRV01"
$OutputPath = Join-Path $EvidenceRoot "PreInstall-$VMName-$Timestamp"

New-Item -Path $OutputPath -ItemType Directory -Force | Out-Null

Write-Host "Validating target VM..." -ForegroundColor Cyan

$VM = Get-VM -Name $VMName -ErrorAction Stop

Write-Host "Capturing VM summary..." -ForegroundColor Cyan

$VM |
    Select-Object Name, State, Generation, ProcessorCount, MemoryStartup, DynamicMemoryEnabled, Path |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "01-VM-Summary-BeforeInstall.txt")

Write-Host "Capturing disk configuration..." -ForegroundColor Cyan

Get-VMHardDiskDrive -VMName $VMName |
    Select-Object VMName, ControllerType, ControllerNumber, ControllerLocation, Path |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "02-VMHardDiskDrive-BeforeInstall.txt")

Write-Host "Capturing VHDX details..." -ForegroundColor Cyan

foreach ($Disk in Get-VMHardDiskDrive -VMName $VMName) {
    if ($Disk.Path -and (Test-Path $Disk.Path)) {
        Get-VHD -Path $Disk.Path |
            Format-List |
            Out-File -FilePath (Join-Path $OutputPath ("03-VHD-" + [IO.Path]::GetFileNameWithoutExtension($Disk.Path) + ".txt")) -Encoding UTF8
    }
}

Write-Host "Capturing DVD drive configuration..." -ForegroundColor Cyan

Get-VMDvdDrive -VMName $VMName -ErrorAction SilentlyContinue |
    Select-Object VMName, ControllerType, ControllerNumber, ControllerLocation, Path |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "04-VMDvdDrive-BeforeInstall.txt")

Write-Host "Capturing network adapter configuration..." -ForegroundColor Cyan

Get-VMNetworkAdapter -VMName $VMName |
    Select-Object VMName, Name, SwitchName, MacAddress, Status, Connected |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "05-VMNetworkAdapter-BeforeInstall.txt")

Get-VMNetworkAdapterVlan -VMName $VMName |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "06-VMNetworkAdapterVlan-BeforeInstall.txt")

Write-Host "Capturing firmware or BIOS configuration..." -ForegroundColor Cyan

if ($VM.Generation -eq 2) {
    Get-VMFirmware -VMName $VMName |
        Format-List |
        Tee-Object -FilePath (Join-Path $OutputPath "07-VMFirmware-BeforeInstall.txt")
}
else {
    Get-VMBios -VMName $VMName |
        Format-List |
        Tee-Object -FilePath (Join-Path $OutputPath "07-VMBios-BeforeInstall.txt")
}

Write-Host "Capturing integration services state..." -ForegroundColor Cyan

Get-VMIntegrationService -VMName $VMName |
    Select-Object VMName, Name, Enabled, PrimaryStatusDescription, SecondaryStatusDescription |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "08-IntegrationServices-BeforeInstall.txt")

Write-Host "Pre-install guest evidence exported to: $OutputPath" -ForegroundColor Green
~~~

# 09_Install_Guest_OS_And_Configure_Integration_Services_Windows_Guest_Install_Skeleton

~~~powershell
# 09_Install_Guest_OS_And_Configure_Integration_Services_Windows_Guest_Install_Skeleton
# Purpose:
# Prepare a Generation 2 or Generation 1 VM to boot from a Windows installer ISO.

$VMName = "LAB-WIN-SRV01"
$ISOPath = "D:\Hyper-V\ISO\WindowsServer.iso"

Write-Host "Validating VM and ISO..." -ForegroundColor Cyan

$VM = Get-VM -Name $VMName -ErrorAction Stop

if (-not (Test-Path $ISOPath)) {
    throw "Installer ISO does not exist: $ISOPath"
}

Write-Host "Stopping VM if currently running..." -ForegroundColor Cyan

if ($VM.State -ne "Off") {
    Stop-VM -Name $VMName
}

Write-Host "Attaching or updating DVD drive ISO..." -ForegroundColor Cyan

$DVD = Get-VMDvdDrive -VMName $VMName -ErrorAction SilentlyContinue | Select-Object -First 1

if ($DVD) {
    Set-VMDvdDrive `
        -VMName $VMName `
        -ControllerNumber $DVD.ControllerNumber `
        -ControllerLocation $DVD.ControllerLocation `
        -Path $ISOPath
}
else {
    Add-VMDvdDrive `
        -VMName $VMName `
        -Path $ISOPath
}

$DVD = Get-VMDvdDrive -VMName $VMName | Select-Object -First 1

if ($VM.Generation -eq 2) {
    Write-Host "Configuring Generation 2 Windows firmware..." -ForegroundColor Cyan

    Set-VMFirmware `
        -VMName $VMName `
        -EnableSecureBoot On `
        -SecureBootTemplate MicrosoftWindows `
        -FirstBootDevice $DVD

    Get-VMFirmware -VMName $VMName |
        Format-List
}
else {
    Write-Host "Configuring Generation 1 BIOS boot order..." -ForegroundColor Cyan

    Set-VMBios `
        -VMName $VMName `
        -StartupOrder CD,IDE,LegacyNetworkAdapter,Floppy

    Get-VMBios -VMName $VMName |
        Format-List
}

Write-Host "Starting VM for Windows installation..." -ForegroundColor Green

Start-VM -Name $VMName

Write-Host "Open the VM console with:" -ForegroundColor Yellow
Write-Host "vmconnect localhost `"$VMName`"" -ForegroundColor Yellow
~~~

# 09_Install_Guest_OS_And_Configure_Integration_Services_Linux_Guest_Install_Skeleton

~~~powershell
# 09_Install_Guest_OS_And_Configure_Integration_Services_Linux_Guest_Install_Skeleton
# Purpose:
# Prepare a Generation 2 or Generation 1 VM to boot from a Linux installer ISO.

$VMName = "LAB-LINUX01"
$ISOPath = "D:\Hyper-V\ISO\UbuntuServer.iso"
$EnableSecureBootForLinux = $true

Write-Host "Validating VM and ISO..." -ForegroundColor Cyan

$VM = Get-VM -Name $VMName -ErrorAction Stop

if (-not (Test-Path $ISOPath)) {
    throw "Installer ISO does not exist: $ISOPath"
}

Write-Host "Stopping VM if currently running..." -ForegroundColor Cyan

if ($VM.State -ne "Off") {
    Stop-VM -Name $VMName
}

Write-Host "Attaching or updating DVD drive ISO..." -ForegroundColor Cyan

$DVD = Get-VMDvdDrive -VMName $VMName -ErrorAction SilentlyContinue | Select-Object -First 1

if ($DVD) {
    Set-VMDvdDrive `
        -VMName $VMName `
        -ControllerNumber $DVD.ControllerNumber `
        -ControllerLocation $DVD.ControllerLocation `
        -Path $ISOPath
}
else {
    Add-VMDvdDrive `
        -VMName $VMName `
        -Path $ISOPath
}

$DVD = Get-VMDvdDrive -VMName $VMName | Select-Object -First 1

if ($VM.Generation -eq 2) {
    Write-Host "Configuring Generation 2 Linux firmware..." -ForegroundColor Cyan

    if ($EnableSecureBootForLinux) {
        Set-VMFirmware `
            -VMName $VMName `
            -EnableSecureBoot On `
            -SecureBootTemplate MicrosoftUEFICertificateAuthority `
            -FirstBootDevice $DVD
    }
    else {
        Set-VMFirmware `
            -VMName $VMName `
            -EnableSecureBoot Off `
            -FirstBootDevice $DVD
    }

    Get-VMFirmware -VMName $VMName |
        Format-List
}
else {
    Write-Host "Configuring Generation 1 BIOS boot order..." -ForegroundColor Cyan

    Set-VMBios `
        -VMName $VMName `
        -StartupOrder CD,IDE,LegacyNetworkAdapter,Floppy

    Get-VMBios -VMName $VMName |
        Format-List
}

Write-Host "Starting VM for Linux installation..." -ForegroundColor Green

Start-VM -Name $VMName

Write-Host "Open the VM console with:" -ForegroundColor Yellow
Write-Host "vmconnect localhost `"$VMName`"" -ForegroundColor Yellow
~~~

# 09_Install_Guest_OS_And_Configure_Integration_Services_Windows_Guest_PostInstall_Skeleton

~~~powershell
# 09_Install_Guest_OS_And_Configure_Integration_Services_Windows_Guest_PostInstall_Skeleton
# Purpose:
# Run basic post-install Windows guest configuration through PowerShell Direct.
# Requires a supported Windows guest and valid local administrator credentials.

$VMName = "LAB-WIN-SRV01"
$GuestComputerName = "LAB-WIN-SRV01"
$GuestAdminUser = "Administrator"

Write-Host "Collecting guest credentials..." -ForegroundColor Cyan

$Credential = Get-Credential -UserName $GuestAdminUser -Message "Enter guest local administrator credentials"

Write-Host "Testing PowerShell Direct..." -ForegroundColor Cyan

Invoke-Command -VMName $VMName -Credential $Credential -ScriptBlock {
    hostname
    Get-ComputerInfo | Select-Object CsName, WindowsProductName, WindowsVersion, OsBuildNumber
}

Write-Host "Renaming guest computer if needed..." -ForegroundColor Cyan

Invoke-Command -VMName $VMName -Credential $Credential -ScriptBlock {
    param($GuestComputerName)

    if ($env:COMPUTERNAME -ne $GuestComputerName) {
        Rename-Computer -NewName $GuestComputerName -Force
        "Rename pending. Reboot required."
    }
    else {
        "Computer name already correct."
    }
} -ArgumentList $GuestComputerName

Write-Host "Configuring Windows Update service startup type..." -ForegroundColor Cyan

Invoke-Command -VMName $VMName -Credential $Credential -ScriptBlock {
    Set-Service -Name wuauserv -StartupType Manual
    Get-Service -Name wuauserv | Select-Object Name, Status, StartType
}

Write-Host "Confirming network configuration inside guest..." -ForegroundColor Cyan

Invoke-Command -VMName $VMName -Credential $Credential -ScriptBlock {
    Get-NetAdapter |
        Select-Object Name, Status, LinkSpeed, MacAddress

    Get-NetIPConfiguration |
        Format-List
}

Write-Host "Confirming guest integration related services..." -ForegroundColor Cyan

Invoke-Command -VMName $VMName -Credential $Credential -ScriptBlock {
    Get-Service |
        Where-Object {
            $_.Name -like "vmic*" -or
            $_.DisplayName -like "*Hyper-V*"
        } |
        Select-Object Name, DisplayName, Status, StartType |
        Sort-Object Name
}

Write-Host "Windows guest post-install checks completed." -ForegroundColor Green
Write-Host "Reboot the guest if rename was performed:" -ForegroundColor Yellow
Write-Host "Restart-VM -Name `"$VMName`"" -ForegroundColor Yellow
~~~

# 09_Install_Guest_OS_And_Configure_Integration_Services_Linux_Guest_PostInstall_Skeleton

~~~powershell
# 09_Install_Guest_OS_And_Configure_Integration_Services_Linux_Guest_PostInstall_Skeleton
# Purpose:
# Document host-side checks and Linux guest-side commands after Linux installation.
# Run Linux commands inside the VM console or SSH session after the guest OS is installed.

$VMName = "LAB-LINUX01"

Write-Host "Host-side Linux VM summary..." -ForegroundColor Cyan

Get-VM -Name $VMName |
    Select-Object Name, State, Generation, Uptime, Status |
    Format-List

Write-Host "Host-side integration service state..." -ForegroundColor Cyan

Get-VMIntegrationService -VMName $VMName |
    Select-Object VMName, Name, Enabled, PrimaryStatusDescription, SecondaryStatusDescription |
    Format-Table -AutoSize

Write-Host "Host-side VM network adapter state..." -ForegroundColor Cyan

Get-VMNetworkAdapter -VMName $VMName |
    Select-Object VMName, Name, SwitchName, MacAddress, Status, IPAddresses |
    Format-List

Write-Host "Run these inside the Linux guest:" -ForegroundColor Yellow

@'
# Linux guest commands

hostnamectl

uname -a

ip addr

ip route

cat /etc/os-release

lsmod | grep hv_

systemctl list-units | grep -i hyper

dmesg | grep -i hyper-v

# Ubuntu or Debian style update example
sudo apt update
sudo apt upgrade -y

# RHEL or Fedora style update example
sudo dnf update -y

# Confirm time sync status where systemd-timesyncd or chrony is used
timedatectl
'@
~~~

# 09_Install_Guest_OS_And_Configure_Integration_Services_Integration_Services_Verification_Skeleton

~~~powershell
# 09_Install_Guest_OS_And_Configure_Integration_Services_Integration_Services_Verification_Skeleton
# Purpose:
# Verify and configure Hyper-V integration services after the guest OS is installed.

$VMName = "LAB-WIN-SRV01"

Write-Host "Capturing integration service state..." -ForegroundColor Cyan

Get-VMIntegrationService -VMName $VMName |
    Select-Object VMName, Name, Enabled, PrimaryStatusDescription, SecondaryStatusDescription |
    Format-Table -AutoSize

Write-Host "Enabling common integration services..." -ForegroundColor Cyan

$ServicesToEnable = @(
    "Heartbeat",
    "Key-Value Pair Exchange",
    "Shutdown",
    "Time Synchronization",
    "VSS"
)

foreach ($ServiceName in $ServicesToEnable) {
    $Service = Get-VMIntegrationService -VMName $VMName -Name $ServiceName -ErrorAction SilentlyContinue

    if ($Service) {
        Enable-VMIntegrationService -VMName $VMName -Name $ServiceName
    }
    else {
        Write-Host "Integration service not found: $ServiceName" -ForegroundColor Yellow
    }
}

Write-Host "Optional: enable Guest Service Interface for host to guest file copy." -ForegroundColor Cyan

$EnableGuestServiceInterface = $false

if ($EnableGuestServiceInterface) {
    Enable-VMIntegrationService -VMName $VMName -Name "Guest Service Interface"
}
else {
    Disable-VMIntegrationService -VMName $VMName -Name "Guest Service Interface" -ErrorAction SilentlyContinue
}

Write-Host "Final integration service state:" -ForegroundColor Green

Get-VMIntegrationService -VMName $VMName |
    Select-Object VMName, Name, Enabled, PrimaryStatusDescription, SecondaryStatusDescription |
    Format-Table -AutoSize
~~~

# 09_Install_Guest_OS_And_Configure_Integration_Services_PostInstall_Verification_Skeleton

~~~powershell
# 09_Install_Guest_OS_And_Configure_Integration_Services_PostInstall_Verification_Skeleton
# Purpose:
# Capture final guest OS install, boot, disk, network, firmware, and integration services evidence.

$EvidenceRoot = "C:\Admin\HyperV-GuestInstall"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$VMName = "LAB-WIN-SRV01"
$OutputPath = Join-Path $EvidenceRoot "PostInstall-$VMName-$Timestamp"

New-Item -Path $OutputPath -ItemType Directory -Force | Out-Null

Write-Host "Capturing VM summary after guest install..." -ForegroundColor Cyan

Get-VM -Name $VMName |
    Select-Object Name, State, Generation, Uptime, Status, ProcessorCount, MemoryAssigned, MemoryDemand, Path |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "01-VM-Summary-AfterInstall.txt")

Write-Host "Capturing integration services after guest install..." -ForegroundColor Cyan

Get-VMIntegrationService -VMName $VMName |
    Select-Object VMName, Name, Enabled, PrimaryStatusDescription, SecondaryStatusDescription |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "02-IntegrationServices-AfterInstall.txt")

Write-Host "Capturing network adapter state after guest install..." -ForegroundColor Cyan

Get-VMNetworkAdapter -VMName $VMName |
    Select-Object VMName, Name, SwitchName, MacAddress, Status, Connected, IPAddresses |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "03-VMNetworkAdapter-AfterInstall.txt")

Write-Host "Capturing disk and DVD state after guest install..." -ForegroundColor Cyan

Get-VMHardDiskDrive -VMName $VMName |
    Select-Object VMName, ControllerType, ControllerNumber, ControllerLocation, Path |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "04-VMHardDiskDrive-AfterInstall.txt")

Get-VMDvdDrive -VMName $VMName |
    Select-Object VMName, ControllerType, ControllerNumber, ControllerLocation, Path |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "05-VMDvdDrive-AfterInstall.txt")

Write-Host "Capturing firmware or BIOS after guest install..." -ForegroundColor Cyan

$VM = Get-VM -Name $VMName

if ($VM.Generation -eq 2) {
    Get-VMFirmware -VMName $VMName |
        Format-List |
        Tee-Object -FilePath (Join-Path $OutputPath "06-VMFirmware-AfterInstall.txt")
}
else {
    Get-VMBios -VMName $VMName |
        Format-List |
        Tee-Object -FilePath (Join-Path $OutputPath "06-VMBios-AfterInstall.txt")
}

Write-Host "Capturing recent VMMS and worker events..." -ForegroundColor Cyan

Get-WinEvent -LogName "Microsoft-Windows-Hyper-V-VMMS-Admin" -MaxEvents 50 -ErrorAction SilentlyContinue |
    Select-Object TimeCreated, Id, LevelDisplayName, ProviderName, Message |
    Format-List |
    Out-File -FilePath (Join-Path $OutputPath "07-HyperV-VMMS-RecentEvents.txt") -Encoding UTF8

Get-WinEvent -LogName "Microsoft-Windows-Hyper-V-Worker-Admin" -MaxEvents 50 -ErrorAction SilentlyContinue |
    Select-Object TimeCreated, Id, LevelDisplayName, ProviderName, Message |
    Format-List |
    Out-File -FilePath (Join-Path $OutputPath "08-HyperV-Worker-RecentEvents.txt") -Encoding UTF8

Write-Host "Testing heartbeat state..." -ForegroundColor Cyan

Get-VMIntegrationService -VMName $VMName -Name Heartbeat |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "09-Heartbeat-State.txt")

Write-Host "Post-install guest evidence exported to: $OutputPath" -ForegroundColor Green
~~~

# 09_Install_Guest_OS_And_Configure_Integration_Services_Verification_Commands

| Command | Where to Run | What It Confirms |
|---|---|---|
| `Get-VM -Name '<vm-name>'` | Hyper-V host | Confirms VM exists and shows runtime state |
| `Get-VM -Name '<vm-name>' \| Select Name, State, Generation, Uptime, Status` | Hyper-V host | Confirms guest VM is running after install |
| `Get-VMHardDiskDrive -VMName '<vm-name>'` | Hyper-V host | Confirms OS disk is attached |
| `Get-VMDvdDrive -VMName '<vm-name>'` | Hyper-V host | Confirms ISO is attached or ejected |
| `Get-VMFirmware -VMName '<vm-name>'` | Hyper-V host | Confirms Generation 2 boot order and Secure Boot |
| `Get-VMBios -VMName '<vm-name>'` | Hyper-V host | Confirms Generation 1 boot order |
| `Get-VMNetworkAdapter -VMName '<vm-name>'` | Hyper-V host | Confirms VM adapter, switch, MAC, and IP reporting |
| `Get-VMIntegrationService -VMName '<vm-name>'` | Hyper-V host | Lists all integration services and status |
| `Get-VMIntegrationService -VMName '<vm-name>' -Name Heartbeat` | Hyper-V host | Confirms guest heartbeat state |
| `Enable-VMIntegrationService -VMName '<vm-name>' -Name '<service-name>'` | Hyper-V host | Enables a selected integration service |
| `Disable-VMIntegrationService -VMName '<vm-name>' -Name '<service-name>'` | Hyper-V host | Disables a selected integration service |
| `Invoke-Command -VMName '<vm-name>' -Credential (Get-Credential) -ScriptBlock { hostname }` | Hyper-V host | Confirms PowerShell Direct into Windows guest |
| `vmconnect localhost '<vm-name>'` | Hyper-V host | Opens guest console |
| `hostnamectl` | Linux guest | Confirms Linux guest hostname |
| `lsmod \| grep hv_` | Linux guest | Confirms Hyper-V related Linux kernel modules are loaded |
| `ip addr` | Linux guest | Confirms Linux guest network configuration |
| `Get-WinEvent -LogName 'Microsoft-Windows-Hyper-V-VMMS-Admin' -MaxEvents 20` | Hyper-V host | Checks recent VM management errors |
| `Get-WinEvent -LogName 'Microsoft-Windows-Hyper-V-Worker-Admin' -MaxEvents 20` | Hyper-V host | Checks guest worker process errors |

# 09_Install_Guest_OS_And_Configure_Integration_Services_Rollback

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Stop failed installation VM | Hyper-V host | `Stop-VM -Name '<vm-name>' -TurnOff` | VM is powered off |
| 2 | Eject installer ISO | Hyper-V host | `Set-VMDvdDrive -VMName '<vm-name>' -Path $null` | ISO is detached |
| 3 | Reset Generation 2 boot order to disk | Hyper-V host | `$disk = Get-VMHardDiskDrive -VMName '<vm-name>' \| Select-Object -First 1; Set-VMFirmware -VMName '<vm-name>' -FirstBootDevice $disk` | VM boots from disk first |
| 4 | Reset Generation 1 boot order to disk | Hyper-V host | `Set-VMBios -VMName '<vm-name>' -StartupOrder IDE,CD,LegacyNetworkAdapter,Floppy` | VM boots from IDE disk first |
| 5 | Disable Guest Service Interface if not approved | Hyper-V host | `Disable-VMIntegrationService -VMName '<vm-name>' -Name 'Guest Service Interface'` | Host to guest file copy service is disabled |
| 6 | Restore integration service state | Hyper-V host | `Enable-VMIntegrationService` or `Disable-VMIntegrationService` | Integration service state matches baseline |
| 7 | Remove broken VM if rebuild is required | Hyper-V host | `Remove-VM -Name '<vm-name>' -Force` | VM registration is removed |
| 8 | Delete OS VHDX only if approved | Hyper-V host | `Remove-Item '<os-vhdx-path>' -Force` | Failed OS disk is removed |
| 9 | Recreate VM from prior workbook if needed | Hyper-V host | Use task `05_Create_Generation_1_And_Generation_2_VMs` | Clean VM shell exists |
| 10 | Capture rollback state | Hyper-V host | `Get-VM -Name '<vm-name>'; Get-VMIntegrationService -VMName '<vm-name>'` | Rollback evidence is documented |

# 09_Install_Guest_OS_And_Configure_Integration_Services_Failure_Checks

| Symptom | Likely Cause | Fix |
|---|---|---|
| VM boots back to installer repeatedly | ISO still attached or DVD remains first boot device | Eject ISO and set hard disk as first boot device |
| Generation 2 VM does not boot ISO | ISO is not UEFI bootable or boot order is wrong | Use UEFI capable ISO and set DVD as first boot device |
| Windows Generation 2 install fails with Secure Boot issue | Wrong Secure Boot template or unsupported media | Use `MicrosoftWindows` template or disable Secure Boot temporarily |
| Linux Generation 2 install fails with Secure Boot issue | Linux ISO does not support configured Secure Boot template | Use `MicrosoftUEFICertificateAuthority` or disable Secure Boot |
| Generation 1 VM cannot find boot disk | Boot disk is on wrong controller or BIOS order is wrong | Attach boot disk to IDE and set BIOS startup order |
| Guest installer cannot see disk | VHDX not attached or unsupported controller scenario | Confirm `Get-VMHardDiskDrive` and generation specific controller requirements |
| Guest has no network during install | Adapter disconnected, wrong switch, VLAN mismatch, or DHCP unavailable | Confirm adapter, switch, VLAN, and DHCP scope |
| VMConnect console does not open | Hyper-V tools missing or permission issue | Run as administrator and confirm Hyper-V tools are installed |
| PowerShell Direct fails | Guest is unsupported, not running, credentials wrong, or integration not ready | Confirm supported Windows guest, heartbeat, credentials, and VM state |
| Heartbeat shows no contact | Guest OS not booted, integration component unavailable, or guest hung | Open console, confirm OS boot, restart guest if needed |
| Time inside guest is wrong | Time sync disabled or guest time service misconfigured | Review Hyper-V time sync and guest time service |
| Linux `hv_` modules not loaded | Unsupported kernel or missing Hyper-V drivers | Update guest kernel or use supported distribution |
| Guest Service Interface file copy fails | Guest Service Interface disabled | Enable `Guest Service Interface` if approved |
| Windows integration services appear missing | Older unsupported guest or damaged guest services | Patch guest OS and confirm Hyper-V guest service components |
| Backup integration fails | VSS integration disabled or guest VSS issue | Enable VSS integration and check guest VSS writers |
| Guest shutdown from host fails | Shutdown integration service disabled or guest hung | Enable Shutdown integration or use guest console |
| Guest rename does not apply | Reboot required | Restart guest OS |
| Guest static IP breaks connectivity | Wrong VLAN, gateway, DNS, or subnet | Confirm VM adapter VLAN and guest IP settings |
| Event logs show worker errors | Guest boot, device, firmware, or storage issue | Review Hyper-V Worker Admin and VMMS Admin logs |

# 09_Install_Guest_OS_And_Configure_Integration_Services_Related_Labs

| Lab | Relationship |
|---|---|
| 00_Hyper-V_Index.md | Places this guest install task in the full Hyper-V suite |
| 01_Confirm_Hyper-V_Host_Baseline.md | Confirms host hardware, storage, and networking readiness |
| 02_Install_Hyper-V_Role_And_Management_Tools.md | Required before VM install and integration cmdlets are available |
| 03_Configure_Hyper-V_Host_Default_Paths_And_Settings.md | Provides the default VM and VHDX paths used by guest VMs |
| 04_Create_And_Configure_Virtual_Switches.md | Provides the virtual switches used by guest networking |
| 05_Create_Generation_1_And_Generation_2_VMs.md | Creates the VM shell used for OS installation |
| 06_Configure_VM_CPU_Memory_Dynamic_Memory_And_NUMA.md | Provides CPU and memory sizing before OS install |
| 07_Create_Attach_Resize_And_Compact_VHDX_Disks.md | Provides the OS and data disks used by the guest |
| 08_Configure_VM_Network_Adapters_VLANs_MAC_And_Bandwidth.md | Provides adapter, VLAN, MAC, and bandwidth settings before OS install |
| 10_Configure_Checkpoints_Production_Checkpoints_And_Restore.md | Adds restore points after a clean OS installation |
| 12_Configure_PowerShell_Direct_And_Guest_Management.md | Expands guest management after OS installation |
| 13_Configure_VM_Backup_And_Recovery_Workflows.md | Protects installed guest VMs |
| 18_Troubleshoot_Hyper-V_VM_Network_Storage_Checkpoint_And_Startup_Failures.md | Uses guest install and integration state during troubleshooting |