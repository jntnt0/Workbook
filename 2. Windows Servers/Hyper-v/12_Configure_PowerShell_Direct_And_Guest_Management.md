# 12_Configure_PowerShell_Direct_And_Guest_Management

# 12_Configure_PowerShell_Direct_And_Guest_Management_Index
12_Configure_PowerShell_Direct_And_Guest_Management.md
12_Configure_PowerShell_Direct_And_Guest_Management
12_Configure_PowerShell_Direct_And_Guest_Management_Source_Basis
12_Configure_PowerShell_Direct_And_Guest_Management_Mental_Model
12_Configure_PowerShell_Direct_And_Guest_Management_Planning_Table
12_Configure_PowerShell_Direct_And_Guest_Management_Configuration_Checklist
12_Configure_PowerShell_Direct_And_Guest_Management_PreChange_Capture_Skeleton
12_Configure_PowerShell_Direct_And_Guest_Management_PowerShell_Direct_Test_Skeleton
12_Configure_PowerShell_Direct_And_Guest_Management_Windows_Guest_Management_Skeleton
12_Configure_PowerShell_Direct_And_Guest_Management_Guest_File_Copy_Skeleton
12_Configure_PowerShell_Direct_And_Guest_Management_Network_Based_Guest_Management_Skeleton
12_Configure_PowerShell_Direct_And_Guest_Management_PostChange_Verification_Skeleton
12_Configure_PowerShell_Direct_And_Guest_Management_Verification_Commands
12_Configure_PowerShell_Direct_And_Guest_Management_Rollback
12_Configure_PowerShell_Direct_And_Guest_Management_Failure_Checks
12_Configure_PowerShell_Direct_And_Guest_Management_Related_Labs

# 12_Configure_PowerShell_Direct_And_Guest_Management_Source_Basis

| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | PowerShell Direct | Supports `Enter-PSSession -VMName` and `Invoke-Command -VMName` guest management without network connectivity |
| Microsoft Learn | Hyper-V Integration Services | Supports heartbeat, shutdown, time sync, data exchange, VSS, and guest service interface |
| Microsoft Learn | Hyper-V PowerShell `Get-VMIntegrationService` | Supports integration service verification |
| Microsoft Learn | Hyper-V PowerShell `Copy-VMFile` | Supports copying files from host to guest when Guest Service Interface is enabled |
| Microsoft Learn | PowerShell Remoting | Supports network-based management with WinRM after guest networking is configured |
| Microsoft Learn | OpenSSH on Windows and Linux | Supports SSH based guest management where WinRM is not used |
| Internal Workbook Standard | Windows Server workbook structure | Keeps this workbook aligned with the Hyper-V, AD, DNS, DHCP, GPO, and File Services format |

# 12_Configure_PowerShell_Direct_And_Guest_Management_Mental_Model

| Concept | Operational Meaning |
|---|---|
| PowerShell Direct | Hyper-V host feature that runs PowerShell inside a supported Windows guest through the VM bus |
| Guest management | Administrative control of the guest OS through PowerShell Direct, WinRM, SSH, VM console, or file copy |
| VMName targeting | PowerShell Direct can target a guest by VM name from the Hyper-V host |
| VMId targeting | PowerShell Direct can target a guest by VM ID if names are duplicated |
| Guest credentials | Local or domain credentials inside the guest OS, not host credentials |
| VM bus | Hyper-V communication path that allows host to guest operations without guest network connectivity |
| Integration services | Guest components required for heartbeat, shutdown, time sync, data exchange, VSS, and file copy behavior |
| Guest Service Interface | Integration service required for `Copy-VMFile` from host to guest |
| Heartbeat | Integration signal showing whether the guest OS is responsive |
| WinRM | Network-based Windows remote management path after guest networking works |
| SSH | Network-based management path commonly used for Linux guests and optionally Windows guests |
| Console access | VMConnect access used when remoting is unavailable |
| Break-glass management | Using PowerShell Direct when guest networking, firewall, DNS, or IP configuration is broken |
| Rollback boundary | This workbook configures management access paths, not application deployment or domain join policy |

# 12_Configure_PowerShell_Direct_And_Guest_Management_Planning_Table

| Item | Example | Decision |
|---|---|---|
| Hyper-V host name | `HV01` | `<host-name>` |
| VM name | `LAB-WIN-SRV01` | `<vm-name>` |
| VM ID | `GUID from Get-VM` | `<vm-id>` |
| Guest OS | Windows Server 2022, Windows 11, Ubuntu Server | `<guest-os>` |
| Guest hostname | `LAB-WIN-SRV01` | `<guest-hostname>` |
| Guest local admin | `Administrator`, `localadmin` | `<guest-admin>` |
| Guest domain account | `CORP\AdminUser` | `<domain-admin>` |
| PowerShell Direct required | Yes or No | `<yes-no>` |
| WinRM required | Yes or No | `<yes-no>` |
| SSH required | Yes or No | `<yes-no>` |
| Guest Service Interface required | Yes or No | `<yes-no>` |
| File copy source path | `C:\Admin\Tools\script.ps1` | `<host-source-path>` |
| File copy destination path | `C:\Temp\script.ps1` | `<guest-destination-path>` |
| Guest IP mode | DHCP or static | `<dhcp-static>` |
| Guest IP address | `192.168.10.51` | `<guest-ip>` |
| Guest DNS servers | `192.168.10.10`, `192.168.10.11` | `<guest-dns>` |
| Guest firewall profile | Domain, Private, Public | `<guest-firewall-profile>` |
| Evidence path | `C:\Admin\HyperV-GuestManagement` | `<evidence-path>` |
| Rollback required | Yes or No | `<rollback-required>` |

# 12_Configure_PowerShell_Direct_And_Guest_Management_Configuration_Checklist

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm Hyper-V role is installed | Hyper-V host | `Get-WindowsFeature Hyper-V` | Hyper-V shows installed |
| 2 | Confirm target VM exists | Hyper-V host | `Get-VM -Name '<vm-name>'` | Target VM is found |
| 3 | Confirm VM is running | Hyper-V host | `Get-VM -Name '<vm-name>' \| Select Name, State` | VM state is running |
| 4 | Confirm guest OS is installed | Hyper-V host | `vmconnect localhost '<vm-name>'` | Guest OS reaches logon or shell |
| 5 | Capture current integration services | Hyper-V host | `Get-VMIntegrationService -VMName '<vm-name>'` | Integration services are documented |
| 6 | Confirm heartbeat | Hyper-V host | `Get-VMIntegrationService -VMName '<vm-name>' -Name Heartbeat` | Heartbeat reports healthy if guest is responsive |
| 7 | Test PowerShell Direct with guest credentials | Hyper-V host | `Invoke-Command -VMName '<vm-name>' -Credential (Get-Credential) -ScriptBlock { hostname }` | Host can run command inside guest |
| 8 | Open interactive PowerShell Direct session if needed | Hyper-V host | `Enter-PSSession -VMName '<vm-name>' -Credential (Get-Credential)` | Interactive guest shell opens |
| 9 | Confirm guest OS identity | Hyper-V host using PowerShell Direct | `Get-ComputerInfo \| Select CsName, WindowsProductName, OsBuildNumber` | Guest identity is returned |
| 10 | Confirm guest network state | Hyper-V host using PowerShell Direct | `Get-NetIPConfiguration` | Guest IP, DNS, and gateway are visible |
| 11 | Enable Guest Service Interface if file copy is required | Hyper-V host | `Enable-VMIntegrationService -VMName '<vm-name>' -Name 'Guest Service Interface'` | Guest file copy integration is enabled |
| 12 | Copy file from host to guest if required | Hyper-V host | `Copy-VMFile -Name '<vm-name>' -SourcePath '<host-source-path>' -DestinationPath '<guest-destination-path>' -FileSource Host -CreateFullPath` | File copies into guest |
| 13 | Enable WinRM inside Windows guest if network-based management is required | Guest VM or PowerShell Direct | `Enable-PSRemoting -Force` | WinRM is enabled in guest |
| 14 | Confirm WinRM reachability by IP or DNS | Hyper-V host or admin workstation | `Test-WSMan <guest-ip-or-name>` | Guest responds to WinRM |
| 15 | Configure guest firewall for management | Guest VM or PowerShell Direct | `Get-NetFirewallRule -DisplayGroup 'Windows Remote Management'` | WinRM firewall rules are enabled |
| 16 | Confirm SSH service if Linux guest is used | Linux guest | `systemctl status ssh` | SSH service is installed and running |
| 17 | Test SSH to Linux guest | Hyper-V host or admin workstation | `ssh <user>@<guest-ip>` | SSH session opens |
| 18 | Capture post-change guest management evidence | Hyper-V host | Run post-change verification skeleton | Guest management paths are documented |

# 12_Configure_PowerShell_Direct_And_Guest_Management_PreChange_Capture_Skeleton

~~~powershell
# 12_Configure_PowerShell_Direct_And_Guest_Management_PreChange_Capture_Skeleton
# Purpose:
# Capture VM state, integration services, network adapters, heartbeat, and current management readiness.

$EvidenceRoot = "C:\Admin\HyperV-GuestManagement"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$VMName = "LAB-WIN-SRV01"
$OutputPath = Join-Path $EvidenceRoot "PreChange-$VMName-$Timestamp"

New-Item -Path $OutputPath -ItemType Directory -Force | Out-Null

Write-Host "Validating target VM..." -ForegroundColor Cyan

$VM = Get-VM -Name $VMName -ErrorAction Stop

Write-Host "Capturing VM summary..." -ForegroundColor Cyan

$VM |
    Select-Object Name, Id, State, Status, Generation, Uptime, Path |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "01-VM-Summary-Before.txt")

$VM |
    ConvertTo-Json -Depth 8 |
    Out-File -FilePath (Join-Path $OutputPath "01-VM-Summary-Before.json") -Encoding UTF8

Write-Host "Capturing integration services..." -ForegroundColor Cyan

Get-VMIntegrationService -VMName $VMName |
    Select-Object VMName, Name, Enabled, PrimaryStatusDescription, SecondaryStatusDescription |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "02-IntegrationServices-Before.txt")

Write-Host "Capturing heartbeat state..." -ForegroundColor Cyan

Get-VMIntegrationService -VMName $VMName -Name Heartbeat |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "03-Heartbeat-Before.txt")

Write-Host "Capturing VM network adapter state..." -ForegroundColor Cyan

Get-VMNetworkAdapter -VMName $VMName |
    Select-Object VMName, Name, SwitchName, MacAddress, Status, Connected, IPAddresses |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "04-VMNetworkAdapter-Before.txt")

Write-Host "Capturing VLAN state..." -ForegroundColor Cyan

Get-VMNetworkAdapterVlan -VMName $VMName |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "05-VMNetworkAdapterVlan-Before.txt")

Write-Host "Capturing VM console and worker event context..." -ForegroundColor Cyan

Get-WinEvent -LogName "Microsoft-Windows-Hyper-V-Worker-Admin" -MaxEvents 50 -ErrorAction SilentlyContinue |
    Select-Object TimeCreated, Id, LevelDisplayName, ProviderName, Message |
    Format-List |
    Out-File -FilePath (Join-Path $OutputPath "06-HyperV-Worker-RecentEvents-Before.txt") -Encoding UTF8

Get-WinEvent -LogName "Microsoft-Windows-Hyper-V-VMMS-Admin" -MaxEvents 50 -ErrorAction SilentlyContinue |
    Select-Object TimeCreated, Id, LevelDisplayName, ProviderName, Message |
    Format-List |
    Out-File -FilePath (Join-Path $OutputPath "07-HyperV-VMMS-RecentEvents-Before.txt") -Encoding UTF8

Write-Host "Pre-change guest management evidence exported to: $OutputPath" -ForegroundColor Green
~~~

# 12_Configure_PowerShell_Direct_And_Guest_Management_PowerShell_Direct_Test_Skeleton

~~~powershell
# 12_Configure_PowerShell_Direct_And_Guest_Management_PowerShell_Direct_Test_Skeleton
# Purpose:
# Test PowerShell Direct into a supported Windows guest from the Hyper-V host.

$VMName = "LAB-WIN-SRV01"
$GuestUser = "Administrator"

Write-Host "Validating target VM state..." -ForegroundColor Cyan

$VM = Get-VM -Name $VMName -ErrorAction Stop

if ($VM.State -ne "Running") {
    throw "PowerShell Direct requires the VM to be running. Current state: $($VM.State)"
}

Write-Host "Collecting guest credentials..." -ForegroundColor Cyan

$Credential = Get-Credential -UserName $GuestUser -Message "Enter credentials for the guest OS"

Write-Host "Testing basic PowerShell Direct command..." -ForegroundColor Cyan

Invoke-Command -VMName $VMName -Credential $Credential -ScriptBlock {
    [PSCustomObject]@{
        GuestComputerName = $env:COMPUTERNAME
        UserContext = whoami
        PowerShellVersion = $PSVersionTable.PSVersion.ToString()
        OS = (Get-CimInstance Win32_OperatingSystem).Caption
        Build = (Get-CimInstance Win32_OperatingSystem).BuildNumber
    }
} |
Format-List

Write-Host "Testing guest services and networking through PowerShell Direct..." -ForegroundColor Cyan

Invoke-Command -VMName $VMName -Credential $Credential -ScriptBlock {
    Get-Service |
        Where-Object {
            $_.Name -like "vmic*" -or
            $_.DisplayName -like "*Hyper-V*"
        } |
        Select-Object Name, DisplayName, Status, StartType |
        Sort-Object Name

    Get-NetIPConfiguration
}

Write-Host "PowerShell Direct test completed." -ForegroundColor Green
~~~

# 12_Configure_PowerShell_Direct_And_Guest_Management_Windows_Guest_Management_Skeleton

~~~powershell
# 12_Configure_PowerShell_Direct_And_Guest_Management_Windows_Guest_Management_Skeleton
# Purpose:
# Configure basic Windows guest management from the Hyper-V host using PowerShell Direct.

$VMName = "LAB-WIN-SRV01"
$GuestUser = "Administrator"

$DesiredComputerName = "LAB-WIN-SRV01"
$EnableWinRM = $true
$SetPrivateFirewallProfile = $true

Write-Host "Collecting guest credentials..." -ForegroundColor Cyan

$Credential = Get-Credential -UserName $GuestUser -Message "Enter guest local or domain administrator credentials"

Write-Host "Running Windows guest management configuration..." -ForegroundColor Cyan

Invoke-Command -VMName $VMName -Credential $Credential -ScriptBlock {
    param(
        $DesiredComputerName,
        $EnableWinRM,
        $SetPrivateFirewallProfile
    )

    Write-Host "Guest identity before changes:"
    hostname
    whoami

    if ($env:COMPUTERNAME -ne $DesiredComputerName) {
        Rename-Computer -NewName $DesiredComputerName -Force
        Write-Host "Computer rename staged. Reboot required."
    }
    else {
        Write-Host "Computer name already matches desired value."
    }

    if ($SetPrivateFirewallProfile) {
        Get-NetConnectionProfile |
            Where-Object { $_.NetworkCategory -eq "Public" } |
            Set-NetConnectionProfile -NetworkCategory Private
    }

    if ($EnableWinRM) {
        Enable-PSRemoting -Force
    }

    Write-Host "Guest OS summary:"
    Get-ComputerInfo |
        Select-Object CsName, CsDomain, CsPartOfDomain, WindowsProductName, WindowsVersion, OsBuildNumber

    Write-Host "Guest network configuration:"
    Get-NetIPConfiguration

    Write-Host "Guest firewall profiles:"
    Get-NetFirewallProfile |
        Select-Object Name, Enabled, DefaultInboundAction, DefaultOutboundAction

    Write-Host "Guest WinRM service:"
    Get-Service WinRM |
        Select-Object Name, Status, StartType

} -ArgumentList $DesiredComputerName, $EnableWinRM, $SetPrivateFirewallProfile

Write-Host "Windows guest management configuration completed." -ForegroundColor Green
Write-Host "If the guest was renamed, restart it with:" -ForegroundColor Yellow
Write-Host "Restart-VM -Name `"$VMName`"" -ForegroundColor Yellow
~~~

# 12_Configure_PowerShell_Direct_And_Guest_Management_Guest_File_Copy_Skeleton

~~~powershell
# 12_Configure_PowerShell_Direct_And_Guest_Management_Guest_File_Copy_Skeleton
# Purpose:
# Enable Guest Service Interface and copy a file from the Hyper-V host into a Windows guest.

$VMName = "LAB-WIN-SRV01"
$SourcePath = "C:\Admin\Tools\TestScript.ps1"
$DestinationPath = "C:\Temp\TestScript.ps1"

Write-Host "Validating source file..." -ForegroundColor Cyan

if (-not (Test-Path $SourcePath)) {
    throw "Source file does not exist: $SourcePath"
}

Write-Host "Validating target VM..." -ForegroundColor Cyan

$VM = Get-VM -Name $VMName -ErrorAction Stop

if ($VM.State -ne "Running") {
    throw "Copy-VMFile requires the VM to be running. Current state: $($VM.State)"
}

Write-Host "Enabling Guest Service Interface..." -ForegroundColor Cyan

Enable-VMIntegrationService `
    -VMName $VMName `
    -Name "Guest Service Interface"

Write-Host "Confirming Guest Service Interface state..." -ForegroundColor Cyan

Get-VMIntegrationService -VMName $VMName -Name "Guest Service Interface" |
    Format-List

Write-Host "Copying file from host to guest..." -ForegroundColor Cyan

Copy-VMFile `
    -Name $VMName `
    -SourcePath $SourcePath `
    -DestinationPath $DestinationPath `
    -FileSource Host `
    -CreateFullPath

Write-Host "File copy command completed." -ForegroundColor Green

Write-Host "Optional verification through PowerShell Direct:" -ForegroundColor Yellow
Write-Host "Invoke-Command -VMName `"$VMName`" -Credential (Get-Credential) -ScriptBlock { Test-Path `"$DestinationPath`" }" -ForegroundColor Yellow
~~~

# 12_Configure_PowerShell_Direct_And_Guest_Management_Network_Based_Guest_Management_Skeleton

~~~powershell
# 12_Configure_PowerShell_Direct_And_Guest_Management_Network_Based_Guest_Management_Skeleton
# Purpose:
# Validate network-based guest management after VM networking is working.
# Use WinRM for Windows guests and SSH for Linux guests or Windows guests with OpenSSH.

$VMName = "LAB-WIN-SRV01"
$GuestAddress = "192.168.10.51"
$TestWinRM = $true
$TestSSH = $false
$SSHUser = "localadmin"

Write-Host "Capturing VM network adapter IP reporting from host..." -ForegroundColor Cyan

Get-VMNetworkAdapter -VMName $VMName |
    Select-Object VMName, Name, SwitchName, MacAddress, Status, IPAddresses |
    Format-List

Write-Host "Testing ICMP or TCP reachability..." -ForegroundColor Cyan

Test-NetConnection $GuestAddress |
    Format-List

if ($TestWinRM) {
    Write-Host "Testing WinRM to guest..." -ForegroundColor Cyan

    Test-WSMan $GuestAddress

    Write-Host "Testing remote PowerShell command over WinRM..." -ForegroundColor Cyan

    Invoke-Command -ComputerName $GuestAddress -Credential (Get-Credential) -ScriptBlock {
        hostname
        whoami
        Get-ComputerInfo | Select-Object CsName, WindowsProductName, OsBuildNumber
    }
}

if ($TestSSH) {
    Write-Host "Testing SSH TCP port 22..." -ForegroundColor Cyan

    Test-NetConnection $GuestAddress -Port 22

    Write-Host "Use this command from a shell if SSH is required:" -ForegroundColor Yellow
    Write-Host "ssh $SSHUser@$GuestAddress" -ForegroundColor Yellow
}

Write-Host "Network-based guest management validation completed." -ForegroundColor Green
~~~

# 12_Configure_PowerShell_Direct_And_Guest_Management_PostChange_Verification_Skeleton

~~~powershell
# 12_Configure_PowerShell_Direct_And_Guest_Management_PostChange_Verification_Skeleton
# Purpose:
# Capture final PowerShell Direct, integration services, guest network, file copy, WinRM, and event evidence.

$EvidenceRoot = "C:\Admin\HyperV-GuestManagement"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$VMName = "LAB-WIN-SRV01"
$GuestUser = "Administrator"
$OutputPath = Join-Path $EvidenceRoot "PostChange-$VMName-$Timestamp"

New-Item -Path $OutputPath -ItemType Directory -Force | Out-Null

Write-Host "Capturing VM summary after guest management configuration..." -ForegroundColor Cyan

Get-VM -Name $VMName |
    Select-Object Name, Id, State, Status, Generation, Uptime |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "01-VM-Summary-After.txt")

Write-Host "Capturing integration services after changes..." -ForegroundColor Cyan

Get-VMIntegrationService -VMName $VMName |
    Select-Object VMName, Name, Enabled, PrimaryStatusDescription, SecondaryStatusDescription |
    Format-Table -AutoSize |
    Tee-Object -FilePath (Join-Path $OutputPath "02-IntegrationServices-After.txt")

Write-Host "Capturing VM network adapter IP reporting..." -ForegroundColor Cyan

Get-VMNetworkAdapter -VMName $VMName |
    Select-Object VMName, Name, SwitchName, MacAddress, Status, Connected, IPAddresses |
    Format-List |
    Tee-Object -FilePath (Join-Path $OutputPath "03-VMNetworkAdapter-After.txt")

Write-Host "Testing PowerShell Direct if guest credentials are available..." -ForegroundColor Cyan

try {
    $Credential = Get-Credential -UserName $GuestUser -Message "Enter guest credentials for final PowerShell Direct verification"

    Invoke-Command -VMName $VMName -Credential $Credential -ScriptBlock {
        [PSCustomObject]@{
            Hostname = hostname
            User = whoami
            OS = (Get-CimInstance Win32_OperatingSystem).Caption
            Build = (Get-CimInstance Win32_OperatingSystem).BuildNumber
            PowerShellVersion = $PSVersionTable.PSVersion.ToString()
        }

        Get-NetIPConfiguration

        Get-Service WinRM |
            Select-Object Name, Status, StartType
    } |
    Out-File -FilePath (Join-Path $OutputPath "04-PowerShellDirect-GuestVerification.txt") -Encoding UTF8
}
catch {
    $_ |
        Out-File -FilePath (Join-Path $OutputPath "04-PowerShellDirect-GuestVerification-Error.txt") -Encoding UTF8
}

Write-Host "Capturing recent Hyper-V Worker and VMMS events..." -ForegroundColor Cyan

Get-WinEvent -LogName "Microsoft-Windows-Hyper-V-Worker-Admin" -MaxEvents 75 -ErrorAction SilentlyContinue |
    Select-Object TimeCreated, Id, LevelDisplayName, ProviderName, Message |
    Format-List |
    Out-File -FilePath (Join-Path $OutputPath "05-HyperV-Worker-RecentEvents-After.txt") -Encoding UTF8

Get-WinEvent -LogName "Microsoft-Windows-Hyper-V-VMMS-Admin" -MaxEvents 75 -ErrorAction SilentlyContinue |
    Select-Object TimeCreated, Id, LevelDisplayName, ProviderName, Message |
    Format-List |
    Out-File -FilePath (Join-Path $OutputPath "06-HyperV-VMMS-RecentEvents-After.txt") -Encoding UTF8

Write-Host "Post-change guest management evidence exported to: $OutputPath" -ForegroundColor Green
~~~

# 12_Configure_PowerShell_Direct_And_Guest_Management_Verification_Commands

| Command | Where to Run | What It Confirms |
|---|---|---|
| `Get-VM -Name '<vm-name>'` | Hyper-V host | Confirms VM exists and shows state |
| `Get-VMIntegrationService -VMName '<vm-name>'` | Hyper-V host | Shows integration service status |
| `Get-VMIntegrationService -VMName '<vm-name>' -Name Heartbeat` | Hyper-V host | Confirms guest heartbeat state |
| `Invoke-Command -VMName '<vm-name>' -Credential (Get-Credential) -ScriptBlock { hostname }` | Hyper-V host | Confirms PowerShell Direct works |
| `Enter-PSSession -VMName '<vm-name>' -Credential (Get-Credential)` | Hyper-V host | Opens an interactive PowerShell Direct session |
| `Get-VMNetworkAdapter -VMName '<vm-name>'` | Hyper-V host | Shows VM adapter state and reported IPs |
| `Enable-VMIntegrationService -VMName '<vm-name>' -Name 'Guest Service Interface'` | Hyper-V host | Enables host to guest file copy |
| `Copy-VMFile -Name '<vm-name>' -SourcePath '<source>' -DestinationPath '<destination>' -FileSource Host -CreateFullPath` | Hyper-V host | Copies file from host to guest |
| `Test-WSMan <guest-ip-or-name>` | Hyper-V host or admin workstation | Confirms WinRM network management path |
| `Invoke-Command -ComputerName '<guest-ip-or-name>' -Credential (Get-Credential) -ScriptBlock { hostname }` | Hyper-V host or admin workstation | Confirms network PowerShell remoting |
| `Test-NetConnection <guest-ip> -Port 22` | Hyper-V host or admin workstation | Confirms SSH TCP reachability |
| `ssh <user>@<guest-ip>` | Hyper-V host or admin workstation | Confirms SSH login to Linux or OpenSSH-enabled Windows guest |
| `Get-NetIPConfiguration` | Windows guest | Confirms guest IP, gateway, and DNS |
| `Get-Service WinRM` | Windows guest | Confirms WinRM service state |
| `Get-NetFirewallRule -DisplayGroup 'Windows Remote Management'` | Windows guest | Confirms WinRM firewall rules |
| `hostnamectl` | Linux guest | Confirms Linux guest identity |
| `ip addr` | Linux guest | Confirms Linux guest IP configuration |
| `systemctl status ssh` | Linux guest | Confirms SSH service state |

# 12_Configure_PowerShell_Direct_And_Guest_Management_Rollback

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Disable Guest Service Interface if no longer required | Hyper-V host | `Disable-VMIntegrationService -VMName '<vm-name>' -Name 'Guest Service Interface'` | Host to guest file copy is disabled |
| 2 | Disable optional integration service if it was enabled by mistake | Hyper-V host | `Disable-VMIntegrationService -VMName '<vm-name>' -Name '<service-name>'` | Integration service returns to prior state |
| 3 | Re-enable required integration service if disabled by mistake | Hyper-V host | `Enable-VMIntegrationService -VMName '<vm-name>' -Name '<service-name>'` | Required service is enabled |
| 4 | Remove copied guest file if not needed | Hyper-V host using PowerShell Direct | `Invoke-Command -VMName '<vm-name>' -Credential (Get-Credential) -ScriptBlock { Remove-Item '<guest-file-path>' -Force }` | Copied file is removed |
| 5 | Disable WinRM if it was enabled only for testing | Windows guest | `Disable-PSRemoting -Force` | WinRM listener and firewall rules are disabled where applicable |
| 6 | Restore firewall profile if changed by mistake | Windows guest | `Set-NetConnectionProfile -InterfaceAlias '<adapter-name>' -NetworkCategory Public` | Network profile returns to prior state |
| 7 | Remove temporary local admin account if created | Windows guest | `Remove-LocalUser -Name '<temp-user>'` | Temporary account is removed |
| 8 | Stop SSH service if it was enabled only for testing | Linux guest | `sudo systemctl disable --now ssh` | SSH service is disabled |
| 9 | Reboot guest after identity or management changes if needed | Hyper-V host | `Restart-VM -Name '<vm-name>'` | Guest restarts cleanly |
| 10 | Capture rollback evidence | Hyper-V host | `Get-VMIntegrationService -VMName '<vm-name>'; Get-VMNetworkAdapter -VMName '<vm-name>'` | Rollback state is documented |

# 12_Configure_PowerShell_Direct_And_Guest_Management_Failure_Checks

| Symptom | Likely Cause | Fix |
|---|---|---|
| `Invoke-Command -VMName` fails | VM is not running, unsupported guest, wrong credentials, or guest not booted | Start VM, confirm supported Windows guest, verify credentials, wait for guest boot |
| `Enter-PSSession -VMName` prompts but rejects credentials | Guest credentials are wrong or account lacks rights | Use valid local or domain admin inside the guest |
| PowerShell Direct works but WinRM fails | Guest network, firewall, DNS, or WinRM listener issue | Use PowerShell Direct to enable WinRM and fix guest network settings |
| VM has no reported IP address | Guest integration not ready, no DHCP, static IP issue, or unsupported guest reporting | Check guest network from console or PowerShell Direct |
| Heartbeat unhealthy | Guest OS hung, integration service disabled, or guest still booting | Open console, check guest state, restart guest if required |
| `Copy-VMFile` fails | Guest Service Interface disabled | Enable `Guest Service Interface` |
| `Copy-VMFile` destination path fails | Destination folder missing or permission issue | Use `-CreateFullPath` and confirm guest permissions |
| Guest file copy succeeds but file not found | Wrong destination path or different guest drive layout | Verify with PowerShell Direct inside guest |
| WinRM test fails by hostname | DNS missing or wrong | Test by IP, then fix DNS registration or guest DNS settings |
| WinRM test fails by IP | Firewall or WinRM service issue | Enable PSRemoting and confirm firewall rules |
| SSH to Linux guest fails | SSH service missing, firewall blocked, wrong IP, or bad credentials | Install and start SSH, confirm IP, open firewall, use correct account |
| Guest rename does not apply | Reboot required | Restart the guest OS |
| Guest firewall profile stays Public | Network profile not changed or domain not reachable | Set profile manually or fix domain and DNS connectivity |
| PowerShell Direct hangs | Guest is overloaded, booting, or PowerShell in guest is broken | Wait, check VM console, restart guest if needed |
| Guest Service Interface is enabled but file copy still fails | Guest integration component issue | Reboot guest and confirm integration service status |
| Linux guest cannot use PowerShell Direct | PowerShell Direct is for supported Windows guests | Use SSH or VM console for Linux |
| Domain account fails inside guest | Guest is not domain joined or cannot contact domain controller | Use local admin or fix guest DNS and domain connectivity |
| Management works from host but not admin workstation | Network route, firewall, DNS, or permissions issue | Validate from workstation with `Test-NetConnection`, `Test-WSMan`, and DNS lookup |
| VMConnect required every time | Guest remote management path is not configured | Configure WinRM or SSH after basic OS install |

# 12_Configure_PowerShell_Direct_And_Guest_Management_Related_Labs

| Lab | Relationship |
|---|---|
| 00_Hyper-V_Index.md | Places this guest management task in the full Hyper-V suite |
| 01_Confirm_Hyper-V_Host_Baseline.md | Confirms host readiness for Hyper-V management |
| 02_Install_Hyper-V_Role_And_Management_Tools.md | Required before PowerShell Direct and VM cmdlets are available |
| 04_Create_And_Configure_Virtual_Switches.md | Provides guest network path for WinRM and SSH management |
| 05_Create_Generation_1_And_Generation_2_VMs.md | Creates the VM shell managed here |
| 08_Configure_VM_Network_Adapters_VLANs_MAC_And_Bandwidth.md | Provides adapter and VLAN settings needed for network-based guest management |
| 09_Install_Guest_OS_And_Configure_Integration_Services.md | Required before guest management is useful |
| 10_Configure_Checkpoints_Production_Checkpoints_And_Restore.md | Useful before risky guest management changes |
| 11_Manage_VM_Lifecycle_Start_Stop_Save_Reset_Export_Import.md | Provides VM start, restart, and console access workflows |
| 13_Configure_VM_Backup_And_Recovery_Workflows.md | Protects guests before larger configuration changes |
| 18_Troubleshoot_Hyper-V_VM_Network_Storage_Checkpoint_And_Startup_Failures.md | Uses PowerShell Direct as a break-glass troubleshooting path |