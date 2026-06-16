13_Configure_Local_Admin_RDP_Firewall_And_Remote_Management_GPOs.md
# 13_Configure_Local_Admin_RDP_Firewall_And_Remote_Management_GPOs

# 13_Configure_Local_Admin_RDP_Firewall_And_Remote_Management_GPOs_Index
13_Configure_Local_Admin_RDP_Firewall_And_Remote_Management_GPOs.md
13_Configure_Local_Admin_RDP_Firewall_And_Remote_Management_GPOs
13_Configure_Local_Admin_RDP_Firewall_And_Remote_Management_GPOs_Source_Basis
13_Configure_Local_Admin_RDP_Firewall_And_Remote_Management_GPOs_Mental_Model
13_Configure_Local_Admin_RDP_Firewall_And_Remote_Management_GPOs_Planning_Table
13_Configure_Local_Admin_RDP_Firewall_And_Remote_Management_GPOs_Control_Map
13_Configure_Local_Admin_RDP_Firewall_And_Remote_Management_GPOs_Configuration_Checklist
13_Configure_Local_Admin_RDP_Firewall_And_Remote_Management_GPOs_Precheck_Skeleton
13_Configure_Local_Admin_RDP_Firewall_And_Remote_Management_GPOs_Create_And_Link_GPO_Skeleton
13_Configure_Local_Admin_RDP_Firewall_And_Remote_Management_GPOs_Local_Admin_Group_GPP_Skeleton
13_Configure_Local_Admin_RDP_Firewall_And_Remote_Management_GPOs_RDP_User_Rights_And_RDP_Service_Skeleton
13_Configure_Local_Admin_RDP_Firewall_And_Remote_Management_GPOs_Firewall_And_Remote_Management_GPMC_Skeleton
13_Configure_Local_Admin_RDP_Firewall_And_Remote_Management_GPOs_PowerShell_Firewall_Policy_Skeleton
13_Configure_Local_Admin_RDP_Firewall_And_Remote_Management_GPOs_Client_RSOP_Validation_Skeleton
13_Configure_Local_Admin_RDP_Firewall_And_Remote_Management_GPOs_Report_And_Backup_Skeleton
13_Configure_Local_Admin_RDP_Firewall_And_Remote_Management_GPOs_Verification_Commands
13_Configure_Local_Admin_RDP_Firewall_And_Remote_Management_GPOs_Rollback
13_Configure_Local_Admin_RDP_Firewall_And_Remote_Management_GPOs_Failure_Checks
13_Configure_Local_Admin_RDP_Firewall_And_Remote_Management_GPOs_Related_Labs

# 13_Configure_Local_Admin_RDP_Firewall_And_Remote_Management_GPOs_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Group Policy Management Console | Configuring local admin, RDP, firewall, and remote management policies |
| Microsoft Learn | Group Policy Preferences Local Users and Groups | Managing local Administrators and Remote Desktop Users group membership |
| Microsoft Learn | User Rights Assignment | Controlling who can log on locally and through Remote Desktop Services |
| Microsoft Learn | Remote Desktop Services Group Policy | Enabling Remote Desktop and NLA-related policy |
| Microsoft Learn | Windows Defender Firewall with Advanced Security | Managing inbound firewall rules through GPO |
| Microsoft Learn | Windows Remote Management Group Policy | Allowing WinRM remote management through GPO |
| Microsoft Learn | NetSecurity PowerShell module | Managing firewall policy stores and GPO firewall rules |
| Microsoft Learn | GroupPolicy PowerShell module | Creating, linking, reporting, and backing up the GPO |
| Windows client policy processing | `gpupdate`, `gpresult`, RSOP, local group checks, firewall checks, WinRM tests | Proving remote administration settings apply safely |

# 13_Configure_Local_Admin_RDP_Firewall_And_Remote_Management_GPOs_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Local Administrators group | Local machine group that grants full admin rights on a workstation or server |
| Approved admin group | Domain security group intentionally added to the local Administrators group |
| Remote Desktop Users group | Local machine group normally used to allow RDP sign-in when paired with the correct user right |
| Allow log on through Remote Desktop Services | User right that permits RDP logon |
| Deny log on through Remote Desktop Services | User right that blocks RDP logon even if other permissions allow it |
| RDP service | Remote Desktop Services service on the endpoint |
| NLA | Network Level Authentication requirement for RDP |
| Firewall profile | Domain, Private, or Public firewall profile |
| Firewall inbound rule | Rule that permits or blocks inbound traffic to the endpoint |
| Remote management | Administrative access by WinRM, PowerShell remoting, WMI, Event Viewer, Services MMC, or firewall management |
| WinRM | Windows Remote Management service used by PowerShell remoting |
| GPP Local Users and Groups | Preference method for adding or updating local group membership |
| Restricted Groups | Older Security Settings method for enforcing local group membership |
| Replace action risk | GPP action that can wipe local group membership if misused |
| Update action | Safer GPP action for adding approved groups without deleting existing members |
| Pilot OU | First OU where remote access controls are tested |
| First rule | Never manage local Administrators broadly until a break-glass access path is confirmed |
| Blunt rule | Bad local admin, RDP, or firewall GPOs can lock you out of every endpoint in scope |

# 13_Configure_Local_Admin_RDP_Firewall_And_Remote_Management_GPOs_Planning_Table
| Item | Example | Decision |
|---|---|---|
| Domain FQDN | `corp.local` | `<domain-fqdn>` |
| Domain DN | `DC=corp,DC=local` | `<domain-dn>` |
| NetBIOS domain | `CORP` | `<netbios-domain>` |
| Pilot computer OU | `OU=Pilot,OU=Workstations,OU=Corp,DC=corp,DC=local` | `<pilot-ou-dn>` |
| Target computer OU | `OU=Workstations,OU=Corp,DC=corp,DC=local` | `<target-computer-ou-dn>` |
| Test computer | `WIN11-01` | `<test-computer>` |
| Management host | `MGMT01` | `<management-host>` |
| Management subnet | `10.10.10.0/24` | `<management-subnet>` |
| Access GPO | `CORP-Workstation-Remote-Admin-Access` | `<access-gpo-name>` |
| Approved local admin group | `GG_Workstation_Local_Admins` | `<local-admin-group>` |
| Approved RDP group | `GG_Workstation_RDP_Users` | `<rdp-group>` |
| Approved remote management group | `GG_Workstation_Remote_Management` | `<remote-management-group>` |
| Break-glass group | `GG_Workstation_BreakGlass_Admins` | `<breakglass-group>` |
| Security filtering group | `GG_GPO_Workstation_Remote_Admin_Apply` | `<filtering-group>` |
| RDP enabled | Yes for pilot only | `<yes-no>` |
| NLA required | Yes | `<yes-no>` |
| WinRM enabled | Yes for management subnet | `<yes-no>` |
| Firewall profile target | Domain profile | `<profile>` |
| Allow RDP from | Management subnet only | `<source-scope>` |
| Allow WinRM from | Management subnet only | `<source-scope>` |
| Allow remote event log | Management subnet only | `<source-scope>` |
| Allow WMI remote management | Management subnet only | `<source-scope>` |
| Report path | `C:\GPOPrep\Reports` | `<report-path>` |
| Backup path | `C:\GPOPrep\Backup` | `<backup-path>` |
| Rollback method | Disable link, remove GPP items, restore backup | `<rollback-plan>` |

# 13_Configure_Local_Admin_RDP_Firewall_And_Remote_Management_GPOs_Control_Map
| Control Area | GPO Location | Example Setting | Recommended Lab Stance | Validation |
|---|---|---|---|---|
| Local Administrators | `Computer Configuration > Preferences > Control Panel Settings > Local Users and Groups` | Add `CORP\GG_Workstation_Local_Admins` to local Administrators | Use Update, not Replace | `Get-LocalGroupMember Administrators` |
| Remote Desktop Users | `Computer Configuration > Preferences > Control Panel Settings > Local Users and Groups` | Add `CORP\GG_Workstation_RDP_Users` to local Remote Desktop Users | Use Update | `Get-LocalGroupMember "Remote Desktop Users"` |
| RDP User Right | `Computer Configuration > Policies > Windows Settings > Security Settings > Local Policies > User Rights Assignment` | Allow log on through Remote Desktop Services | Approved RDP group and admins | `secedit /export` |
| RDP Enablement | `Computer Configuration > Policies > Administrative Templates > Windows Components > Remote Desktop Services > Remote Desktop Session Host > Connections` | Allow users to connect remotely by using Remote Desktop Services | Enabled for pilot | Registry and connection test |
| RDP NLA | `Remote Desktop Session Host > Security` | Require user authentication for remote connections by using NLA | Enabled | RDP client behavior |
| RDP Firewall | `Windows Defender Firewall with Advanced Security > Inbound Rules` | Remote Desktop inbound rules | Allow from management subnet | `Get-NetFirewallRule` |
| WinRM Policy | `Administrative Templates > Windows Components > Windows Remote Management > WinRM Service` | Allow remote server management through WinRM | Enabled for management subnet | `Test-WSMan` |
| WinRM Firewall | `Windows Defender Firewall with Advanced Security > Inbound Rules` | Windows Remote Management inbound | Allow from management subnet | `Test-NetConnection -Port 5985` |
| Remote Event Log | Firewall inbound rules | Remote Event Log Management group | Allow from management subnet if required | Event Viewer remote connection |
| Remote Service Management | Firewall inbound rules | Remote Service Management group | Allow from management subnet if required | Services MMC or PowerShell |
| WMI Remote Management | Firewall inbound rules | Windows Management Instrumentation group | Allow from management subnet if required | `Get-CimInstance -ComputerName` |
| Deny RDP | User Rights Assignment | Deny log on through Remote Desktop Services | Use only with caution | `secedit /export` |

# 13_Configure_Local_Admin_RDP_Firewall_And_Remote_Management_GPOs_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Open elevated PowerShell | Management Host | `whoami /groups` | Admin token is visible |
| 2 | Import AD module | Management Host | `Import-Module ActiveDirectory` | AD module imports |
| 3 | Import GroupPolicy module | Management Host | `Import-Module GroupPolicy` | GroupPolicy module imports |
| 4 | Import NetSecurity module | Management Host | `Import-Module NetSecurity` | NetSecurity module imports |
| 5 | Confirm domain identity | Management Host | `Get-ADDomain` | Domain object returns |
| 6 | Confirm pilot OU exists | Management Host | `Get-ADOrganizationalUnit -Identity "<pilot-ou-dn>"` | Pilot OU returns |
| 7 | Confirm test computer exists | Management Host | `Get-ADComputer -Identity "<test-computer>" -Properties DistinguishedName,Enabled` | Test computer returns |
| 8 | Confirm test computer is in pilot scope | Management Host | `Get-ADComputer -Identity "<test-computer>" -Properties DistinguishedName` | Computer DN is under pilot OU |
| 9 | Confirm admin groups exist | Management Host | `Get-ADGroup -Identity "<local-admin-group>"; Get-ADGroup -Identity "<rdp-group>"` | Groups return |
| 10 | Confirm SYSVOL access | Management Host | `Test-Path "\\<domain-fqdn>\SYSVOL\<domain-fqdn>\Policies"` | Returns True |
| 11 | Create report folder | Management Host | `New-Item -ItemType Directory -Force -Path C:\GPOPrep\Reports` | Report path exists |
| 12 | Create backup folder | Management Host | `New-Item -ItemType Directory -Force -Path C:\GPOPrep\Backup` | Backup path exists |
| 13 | Back up existing GPOs before changes | Management Host | `Backup-GPO -All -Path C:\GPOPrep\Backup` | Pre-change backup exists |
| 14 | Create remote admin access GPO | Management Host | `New-GPO -Name "<access-gpo-name>" -Comment "Local admin, RDP, firewall, and remote management access"` | GPO exists |
| 15 | Link GPO to pilot OU | Management Host | `New-GPLink -Name "<access-gpo-name>" -Target "<pilot-ou-dn>" -LinkEnabled Yes` | GPO is linked |
| 16 | Keep GPO unenforced by default | Management Host | `Set-GPLink -Name "<access-gpo-name>" -Target "<pilot-ou-dn>" -Enforced No` | Link is not enforced |
| 17 | Confirm link state | Management Host | `Get-GPInheritance -Target "<pilot-ou-dn>"` | Access GPO appears in pilot OU link list |
| 18 | Confirm GPO permissions | Management Host | `Get-GPPermission -Name "<access-gpo-name>" -All` | Read and Apply permissions are known |
| 19 | Apply security filtering if used | Management Host | `Set-GPPermission -Name "<access-gpo-name>" -TargetName "<filtering-group>" -TargetType Group -PermissionLevel GpoApply` | Filtering group can apply GPO |
| 20 | Preserve required read access | Management Host | `Set-GPPermission -Name "<access-gpo-name>" -TargetName "Authenticated Users" -TargetType Group -PermissionLevel GpoRead` | Authenticated Users can read GPO |
| 21 | Configure local Administrators membership | Management Host | `GPMC > Computer Configuration > Preferences > Control Panel Settings > Local Users and Groups` | Approved admin group is added |
| 22 | Configure Remote Desktop Users membership | Management Host | `GPMC > Computer Configuration > Preferences > Control Panel Settings > Local Users and Groups` | Approved RDP group is added |
| 23 | Configure RDP user right | Management Host | `GPMC > Computer Configuration > Policies > Windows Settings > Security Settings > Local Policies > User Rights Assignment` | Approved groups can log on through RDP |
| 24 | Configure RDP enablement | Management Host | `GPMC > Computer Configuration > Policies > Administrative Templates > Windows Components > Remote Desktop Services` | RDP policy is configured |
| 25 | Configure NLA requirement | Management Host | `GPMC > Remote Desktop Session Host > Security` | NLA is required |
| 26 | Configure WinRM service policy | Management Host | `GPMC > Administrative Templates > Windows Components > Windows Remote Management > WinRM Service` | WinRM remote management is configured |
| 27 | Configure firewall profile baseline | Management Host | `GPMC > Windows Defender Firewall with Advanced Security` | Firewall profile is enabled |
| 28 | Configure RDP firewall inbound rule | Management Host | `GPMC > Windows Defender Firewall with Advanced Security > Inbound Rules` | RDP allowed from management source |
| 29 | Configure WinRM firewall inbound rule | Management Host | `GPMC > Windows Defender Firewall with Advanced Security > Inbound Rules` | WinRM allowed from management source |
| 30 | Configure WMI and remote event log firewall rules if required | Management Host | `GPMC > Windows Defender Firewall with Advanced Security > Inbound Rules` | Required remote admin rule groups are allowed |
| 31 | Export GPO HTML report | Management Host | `Get-GPOReport -Name "<access-gpo-name>" -ReportType Html -Path C:\GPOPrep\Reports\<access-gpo-name>.html` | Report shows configured settings |
| 32 | Export GPO XML report | Management Host | `Get-GPOReport -Name "<access-gpo-name>" -ReportType Xml -Path C:\GPOPrep\Reports\<access-gpo-name>.xml` | XML report exists |
| 33 | Back up configured GPO | Management Host | `Backup-GPO -Name "<access-gpo-name>" -Path C:\GPOPrep\Backup` | Post-change backup exists |
| 34 | Force client policy refresh | Test Client | `gpupdate /force` | Computer policy refresh completes |
| 35 | Reboot test client if required | Test Client | `Restart-Computer` | Computer-side access settings refresh |
| 36 | Validate applied computer GPOs | Test Client | `gpresult /scope computer /r` | Access GPO appears under Applied GPOs |
| 37 | Export GPResult report | Test Client | `gpresult /h C:\GPOPrep\Reports\gpresult-remote-admin-access.html` | HTML RSOP report exists |
| 38 | Validate local Administrators group | Test Client | `Get-LocalGroupMember -Group Administrators` | Approved admin group appears |
| 39 | Validate Remote Desktop Users group | Test Client | `Get-LocalGroupMember -Group "Remote Desktop Users"` | Approved RDP group appears |
| 40 | Validate RDP registry state | Test Client | `Get-ItemProperty "HKLM:\System\CurrentControlSet\Control\Terminal Server"` | `fDenyTSConnections` is `0` if enabled |
| 41 | Validate firewall rules | Test Client | `Get-NetFirewallRule | Where-Object DisplayGroup -match "Remote Desktop|Windows Remote Management"` | Expected rules are enabled |
| 42 | Validate WinRM listener | Test Client | `winrm enumerate winrm/config/listener` | Listener exists if WinRM is enabled |
| 43 | Test WinRM from management host | Management Host | `Test-WSMan <test-computer>` | WinRM responds |
| 44 | Test RDP port from management host | Management Host | `Test-NetConnection <test-computer> -Port 3389` | TCP test succeeds if RDP is enabled |
| 45 | Review Group Policy events | Test Client | `Get-WinEvent -LogName "Microsoft-Windows-GroupPolicy/Operational" -MaxEvents 150` | Policy processing events are visible |
| 46 | Document result | Operator | `Record GPO, pilot OU, approved groups, firewall source scopes, reports, backup, and validation results` | Remote admin access deployment is documented |

# 13_Configure_Local_Admin_RDP_Firewall_And_Remote_Management_GPOs_Precheck_Skeleton
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: validate readiness before configuring local admin, RDP, firewall, and remote management GPOs.

Import-Module ActiveDirectory
Import-Module GroupPolicy
Import-Module NetSecurity

$Domain = Get-ADDomain
$DomainFqdn = $Domain.DNSRoot
$DomainDN = $Domain.DistinguishedName
$DomainNetBIOS = $Domain.NetBIOSName

$TargetComputerOU = "OU=Workstations,OU=Corp,$DomainDN"
$PilotOU = "OU=Pilot,OU=Workstations,OU=Corp,$DomainDN"

$TestComputer = "WIN11-01"

$GpoName = "CORP-Workstation-Remote-Admin-Access"

$LocalAdminGroup = "GG_Workstation_Local_Admins"
$RdpGroup = "GG_Workstation_RDP_Users"
$RemoteManagementGroup = "GG_Workstation_Remote_Management"
$FilteringGroup = "GG_GPO_Workstation_Remote_Admin_Apply"

$BasePath = "C:\GPOPrep"
$ReportPath = "$BasePath\Reports"
$BackupPath = "$BasePath\Backup"

New-Item -ItemType Directory -Force -Path $BasePath,$ReportPath,$BackupPath

# Confirm domain and tool readiness.
$Domain | Select-Object DNSRoot,NetBIOSName,DistinguishedName
Get-Module -ListAvailable GroupPolicy
Get-Module -ListAvailable NetSecurity

# Confirm DC locator and SYSVOL.
nltest /dsgetdc:$DomainFqdn
Test-Path "\\$DomainFqdn\SYSVOL\$DomainFqdn\Policies"
Test-Path "\\$DomainFqdn\NETLOGON"

# Confirm OUs.
Get-ADOrganizationalUnit -Identity $TargetComputerOU
Get-ADOrganizationalUnit -Identity $PilotOU

# Confirm test computer.
Get-ADComputer `
  -Identity $TestComputer `
  -Properties DNSHostName,DistinguishedName,Enabled |
  Select-Object Name,DNSHostName,Enabled,DistinguishedName |
  Out-File "$ReportPath\test-computer-before-remote-admin-access.txt"

# Confirm groups.
foreach ($Group in @($LocalAdminGroup,$RdpGroup,$RemoteManagementGroup,$FilteringGroup)) {
    Get-ADGroup -Identity $Group -ErrorAction SilentlyContinue |
      Select-Object Name,SamAccountName,GroupCategory,GroupScope,DistinguishedName
}

# Capture inheritance state.
Get-GPInheritance `
  -Target $PilotOU |
  Out-File "$ReportPath\pilot-ou-inheritance-before-remote-admin-access.txt"

# Capture GPO inventory.
Get-GPO -All |
  Sort-Object DisplayName |
  Select-Object DisplayName,Id,Owner,GpoStatus,CreationTime,ModificationTime |
  Export-Csv "$ReportPath\gpo-inventory-before-remote-admin-access.csv" -NoTypeInformation

# Back up current GPO state before changes.
Backup-GPO `
  -All `
  -Path $BackupPath
```

# 13_Configure_Local_Admin_RDP_Firewall_And_Remote_Management_GPOs_Create_And_Link_GPO_Skeleton
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: create and link the remote administration access GPO to the pilot OU.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$DomainDN = (Get-ADDomain).DistinguishedName
$PilotOU = "OU=Pilot,OU=Workstations,OU=Corp,$DomainDN"

$GpoName = "CORP-Workstation-Remote-Admin-Access"
$GpoComment = "Pilot GPO for local admin group, RDP access, firewall rules, and remote management."

# Confirm pilot OU exists.
Get-ADOrganizationalUnit -Identity $PilotOU

# Create the GPO if missing.
$ExistingGpo = Get-GPO `
  -Name $GpoName `
  -ErrorAction SilentlyContinue

if ($ExistingGpo) {
    Write-Host "GPO already exists: $GpoName"
}
else {
    New-GPO `
      -Name $GpoName `
      -Comment $GpoComment

    Write-Host "Created GPO: $GpoName"
}

# Link the GPO to pilot OU if not already linked.
$Inheritance = Get-GPInheritance -Target $PilotOU

$ExistingLink = $Inheritance.GpoLinks |
  Where-Object { $_.DisplayName -eq $GpoName }

if ($ExistingLink) {
    Write-Host "GPO link already exists on pilot OU."
}
else {
    New-GPLink `
      -Name $GpoName `
      -Target $PilotOU `
      -LinkEnabled Yes

    Write-Host "Linked $GpoName to $PilotOU"
}

# Safe default: enabled, not enforced.
Set-GPLink `
  -Name $GpoName `
  -Target $PilotOU `
  -LinkEnabled Yes `
  -Enforced No

# Optional link order.
Set-GPLink `
  -Name $GpoName `
  -Target $PilotOU `
  -Order 1

# Confirm final link state.
Get-GPInheritance `
  -Target $PilotOU |
  Format-List
```

# 13_Configure_Local_Admin_RDP_Firewall_And_Remote_Management_GPOs_Local_Admin_Group_GPP_Skeleton
```powershell
# Native GUI workflow for local group membership through Group Policy Preferences.
# Run on the management host.

gpmc.msc

# GUI workflow:
# 1. Open Group Policy Management Console.
# 2. Expand Forest.
# 3. Expand Domains.
# 4. Expand the target domain.
# 5. Expand Group Policy Objects.
# 6. Right-click:
#      CORP-Workstation-Remote-Admin-Access
# 7. Select Edit.
# 8. Go to:
#      Computer Configuration
#      > Preferences
#      > Control Panel Settings
#      > Local Users and Groups
#
# Add approved domain group to local Administrators:
# 9. Right-click Local Users and Groups.
# 10. Select:
#      New > Local Group
# 11. Action:
#      Update
# 12. Group name:
#      Administrators (built-in)
# 13. Members section:
#      Add:
#      CORP\GG_Workstation_Local_Admins
#      CORP\GG_Workstation_BreakGlass_Admins
# 14. Do not select delete all member users.
# 15. Do not select delete all member groups unless this is an intentional controlled lock-down.
#
# Add approved domain group to local Remote Desktop Users:
# 16. Right-click Local Users and Groups.
# 17. Select:
#      New > Local Group
# 18. Action:
#      Update
# 19. Group name:
#      Remote Desktop Users (built-in)
# 20. Members section:
#      Add:
#      CORP\GG_Workstation_RDP_Users
#
# Common tab:
# 21. Item-level targeting:
#      Optional
# 22. Remove this item when it is no longer applied:
#      Usually not selected for local group membership unless rollback behavior is fully tested.
#
# Notes:
# - Use Update for pilot.
# - Avoid Replace until break-glass access has been proven.
# - Validate on one machine before expanding scope.
```

# 13_Configure_Local_Admin_RDP_Firewall_And_Remote_Management_GPOs_RDP_User_Rights_And_RDP_Service_Skeleton
```powershell
# Native GUI workflow for RDP user rights and RDP enablement.
# Run on the management host.

gpmc.msc

# RDP user rights:
# 1. Edit:
#      CORP-Workstation-Remote-Admin-Access
# 2. Go to:
#      Computer Configuration
#      > Policies
#      > Windows Settings
#      > Security Settings
#      > Local Policies
#      > User Rights Assignment
# 3. Open:
#      Allow log on through Remote Desktop Services
# 4. Add approved groups:
#      Administrators
#      CORP\GG_Workstation_RDP_Users
# 5. Review:
#      Deny log on through Remote Desktop Services
# 6. Do not add broad groups to Deny unless intentionally blocking them.
#
# RDP service enablement:
# 7. Go to:
#      Computer Configuration
#      > Policies
#      > Administrative Templates
#      > Windows Components
#      > Remote Desktop Services
#      > Remote Desktop Session Host
#      > Connections
# 8. Open:
#      Allow users to connect remotely by using Remote Desktop Services
# 9. Set:
#      Enabled
#
# RDP NLA:
# 10. Go to:
#      Remote Desktop Session Host
#      > Security
# 11. Open:
#      Require user authentication for remote connections by using Network Level Authentication
# 12. Set:
#      Enabled
#
# Optional registry-policy equivalent for RDP enablement:
# This can be configured through Set-GPRegistryValue if using PowerShell-backed policy.
```

```powershell
# Optional PowerShell registry policy for RDP enablement.
# Run on management host.

Import-Module GroupPolicy

$GpoName = "CORP-Workstation-Remote-Admin-Access"

# Allow Remote Desktop connections.
Set-GPRegistryValue `
  -Name $GpoName `
  -Key "HKLM\System\CurrentControlSet\Control\Terminal Server" `
  -ValueName "fDenyTSConnections" `
  -Type DWord `
  -Value 0

# Require NLA.
Set-GPRegistryValue `
  -Name $GpoName `
  -Key "HKLM\System\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" `
  -ValueName "UserAuthentication" `
  -Type DWord `
  -Value 1
```

# 13_Configure_Local_Admin_RDP_Firewall_And_Remote_Management_GPOs_Firewall_And_Remote_Management_GPMC_Skeleton
```powershell
# Native GUI workflow for firewall and remote management policy.
# Run on the management host.

gpmc.msc

# Firewall policy:
# 1. Edit:
#      CORP-Workstation-Remote-Admin-Access
# 2. Go to:
#      Computer Configuration
#      > Policies
#      > Windows Settings
#      > Security Settings
#      > Windows Defender Firewall with Advanced Security
#
# Firewall profile:
# 3. Right-click Windows Defender Firewall with Advanced Security.
# 4. Select Properties.
# 5. Domain Profile:
#      Firewall state: On
#      Inbound connections: Block default unless allowed
#      Outbound connections: Allow default
#
# RDP inbound rule:
# 6. Go to Inbound Rules.
# 7. Create or enable rule for:
#      Remote Desktop - User Mode (TCP-In)
# 8. Profile:
#      Domain
# 9. Scope:
#      Remote IP address = management subnet, example 10.10.10.0/24
#
# WinRM inbound rule:
# 10. Enable or create rule for:
#      Windows Remote Management (HTTP-In)
# 11. Profile:
#      Domain
# 12. Scope:
#      Remote IP address = management subnet
#
# Remote Event Log Management:
# 13. Enable only if needed:
#      Remote Event Log Management rule group
# 14. Scope:
#      Management subnet only
#
# WMI Remote Management:
# 15. Enable only if needed:
#      Windows Management Instrumentation rule group
# 16. Scope:
#      Management subnet only
#
# Remote Service Management:
# 17. Enable only if needed:
#      Remote Service Management rule group
# 18. Scope:
#      Management subnet only
#
# WinRM service policy:
# 19. Go to:
#      Computer Configuration
#      > Policies
#      > Administrative Templates
#      > Windows Components
#      > Windows Remote Management
#      > WinRM Service
# 20. Open:
#      Allow remote server management through WinRM
# 21. Set:
#      Enabled
# 22. IPv4 filter:
#      10.10.10.0/24
# 23. IPv6 filter:
#      Leave blank or configure approved IPv6 management range
#
# Notes:
# - Avoid opening remote management to Any unless this is a disposable lab.
# - Prefer management subnet scope.
# - Validate RDP and WinRM from the management host.
```

# 13_Configure_Local_Admin_RDP_Firewall_And_Remote_Management_GPOs_PowerShell_Firewall_Policy_Skeleton
```powershell
# Run on a domain-joined management host with the NetSecurity module.
# Purpose: configure firewall policy inside a GPO policy store.
# Use this only after testing in a lab because firewall GPO mistakes can break remote access.

Import-Module ActiveDirectory
Import-Module GroupPolicy
Import-Module NetSecurity

$Domain = Get-ADDomain
$DomainFqdn = $Domain.DNSRoot

$GpoName = "CORP-Workstation-Remote-Admin-Access"
$ManagementSubnet = "10.10.10.0/24"

# Confirm GPO exists.
Get-GPO -Name $GpoName

# Open the GPO firewall policy session.
$GpoSession = Open-NetGPO `
  -PolicyStore "$DomainFqdn\$GpoName"

# Configure Domain firewall profile baseline.
Set-NetFirewallProfile `
  -GPOSession $GpoSession `
  -Profile Domain `
  -Enabled True `
  -DefaultInboundAction Block `
  -DefaultOutboundAction Allow

# Create scoped RDP inbound rule.
New-NetFirewallRule `
  -GPOSession $GpoSession `
  -DisplayName "CORP Allow RDP From Management Subnet" `
  -Direction Inbound `
  -Action Allow `
  -Protocol TCP `
  -LocalPort 3389 `
  -RemoteAddress $ManagementSubnet `
  -Profile Domain `
  -Enabled True

# Create scoped WinRM inbound rule.
New-NetFirewallRule `
  -GPOSession $GpoSession `
  -DisplayName "CORP Allow WinRM HTTP From Management Subnet" `
  -Direction Inbound `
  -Action Allow `
  -Protocol TCP `
  -LocalPort 5985 `
  -RemoteAddress $ManagementSubnet `
  -Profile Domain `
  -Enabled True

# Optional WMI/DCOM style remote management rule.
# Use only if your management tools require it.
# New-NetFirewallRule `
#   -GPOSession $GpoSession `
#   -DisplayName "CORP Allow RPC Endpoint Mapper From Management Subnet" `
#   -Direction Inbound `
#   -Action Allow `
#   -Protocol TCP `
#   -LocalPort 135 `
#   -RemoteAddress $ManagementSubnet `
#   -Profile Domain `
#   -Enabled True

# Save the GPO firewall policy session.
Save-NetGPO `
  -GPOSession $GpoSession

# Export GPO report after firewall policy changes.
Get-GPOReport `
  -Name $GpoName `
  -ReportType Html `
  -Path "C:\GPOPrep\Reports\$GpoName-firewall-policy.html"
```

# 13_Configure_Local_Admin_RDP_Firewall_And_Remote_Management_GPOs_Client_RSOP_Validation_Skeleton
```powershell
# Run on the test Windows client after policy refresh.
# Purpose: prove local admin, RDP, firewall, and remote management settings apply.

$DomainFqdn = "corp.local"
$ReportPath = "C:\GPOPrep\Reports"

New-Item -ItemType Directory -Force -Path $ReportPath

# Confirm client identity.
hostname
whoami
Get-ComputerInfo | Select-Object CsName,CsDomain,CsPartOfDomain

# Confirm DC locator.
nltest /dsgetdc:$DomainFqdn

# Force computer policy refresh.
gpupdate /force

# Reboot if local group, firewall, or RDP behavior does not update immediately.
# Restart-Computer

# Show applied computer-side GPOs.
gpresult /scope computer /r |
  Tee-Object "$ReportPath\gpresult-computer-remote-admin-access.txt"

# Export full HTML policy report.
gpresult /h "$ReportPath\gpresult-remote-admin-access.html"

# Optional RSOP report.
Get-GPResultantSetOfPolicy `
  -ReportType Html `
  -Path "$ReportPath\rsop-remote-admin-access.html" `
  -ErrorAction SilentlyContinue

# Validate local Administrators group.
Get-LocalGroupMember `
  -Group Administrators `
  -ErrorAction SilentlyContinue |
  Select-Object Name,ObjectClass,PrincipalSource |
  Tee-Object "$ReportPath\local-administrators-after-gpo.txt"

# Validate local Remote Desktop Users group.
Get-LocalGroupMember `
  -Group "Remote Desktop Users" `
  -ErrorAction SilentlyContinue |
  Select-Object Name,ObjectClass,PrincipalSource |
  Tee-Object "$ReportPath\remote-desktop-users-after-gpo.txt"

# Validate RDP registry state.
Get-ItemProperty `
  -Path "HKLM:\System\CurrentControlSet\Control\Terminal Server" `
  -ErrorAction SilentlyContinue |
  Select-Object fDenyTSConnections |
  Tee-Object "$ReportPath\rdp-terminal-server-policy.txt"

Get-ItemProperty `
  -Path "HKLM:\System\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" `
  -ErrorAction SilentlyContinue |
  Select-Object UserAuthentication |
  Tee-Object "$ReportPath\rdp-nla-policy.txt"

# Validate firewall profile.
Get-NetFirewallProfile |
  Select-Object Name,Enabled,DefaultInboundAction,DefaultOutboundAction |
  Tee-Object "$ReportPath\firewall-profiles-remote-admin.txt"

# Validate scoped firewall rules.
Get-NetFirewallRule |
  Where-Object {
    $_.DisplayName -like "*RDP*" -or
    $_.DisplayName -like "*WinRM*" -or
    $_.DisplayGroup -like "*Remote Desktop*" -or
    $_.DisplayGroup -like "*Windows Remote Management*"
  } |
  Select-Object DisplayName,DisplayGroup,Enabled,Direction,Action,Profile |
  Tee-Object "$ReportPath\firewall-rdp-winrm-rules.txt"

# Validate WinRM config.
winrm enumerate winrm/config/listener |
  Out-File "$ReportPath\winrm-listeners.txt"

winrm get winrm/config/service |
  Out-File "$ReportPath\winrm-service-config.txt"

# Validate services.
Get-Service `
  -Name TermService,WinRM `
  -ErrorAction SilentlyContinue |
  Select-Object Name,Status,StartType |
  Tee-Object "$ReportPath\remote-admin-services.txt"

# Export effective security policy for user rights review.
secedit /export /cfg "$ReportPath\effective-security-policy-remote-admin.inf"

# Review Group Policy events.
Get-WinEvent `
  -LogName "Microsoft-Windows-GroupPolicy/Operational" `
  -MaxEvents 150 |
  Select-Object TimeCreated,Id,LevelDisplayName,Message |
  Out-File "$ReportPath\group-policy-events-remote-admin-access.txt"
```

# 13_Configure_Local_Admin_RDP_Firewall_And_Remote_Management_GPOs_Report_And_Backup_Skeleton
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: capture final remote admin access GPO report, inheritance, permissions, and backup state.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$DomainDN = (Get-ADDomain).DistinguishedName

$PilotOU = "OU=Pilot,OU=Workstations,OU=Corp,$DomainDN"
$GpoName = "CORP-Workstation-Remote-Admin-Access"

$BasePath = "C:\GPOPrep"
$ReportPath = "$BasePath\Reports"
$BackupPath = "$BasePath\Backup"

New-Item -ItemType Directory -Force -Path $ReportPath,$BackupPath

# Capture final inheritance.
Get-GPInheritance `
  -Target $PilotOU |
  Out-File "$ReportPath\pilot-ou-inheritance-after-remote-admin-access.txt"

# Capture final GPO permissions.
Get-GPPermission `
  -Name $GpoName `
  -All |
  Out-File "$ReportPath\$GpoName-permissions-after.txt"

# Export final reports.
Get-GPOReport `
  -Name $GpoName `
  -ReportType Html `
  -Path "$ReportPath\$GpoName-final.html"

Get-GPOReport `
  -Name $GpoName `
  -ReportType Xml `
  -Path "$ReportPath\$GpoName-final.xml"

# Search XML report for remote admin content.
Select-String `
  -Path "$ReportPath\$GpoName-final.xml" `
  -Pattern "Administrators","Remote Desktop Users","Remote Desktop","Terminal Server","WinRM","Firewall","Windows Remote Management","UserAuthentication","fDenyTSConnections" |
  Out-File "$ReportPath\$GpoName-remote-admin-report-search.txt"

# Back up final GPO state.
Backup-GPO `
  -Name $GpoName `
  -Path $BackupPath

# Inventory all current GPOs after change.
Get-GPO -All |
  Sort-Object DisplayName |
  Select-Object DisplayName,Id,Owner,GpoStatus,CreationTime,ModificationTime |
  Export-Csv "$ReportPath\gpo-inventory-after-remote-admin-access.csv" -NoTypeInformation

Write-Host "Remote admin access GPO report and backup complete."
Write-Host "Report path: $ReportPath"
Write-Host "Backup path: $BackupPath"
```

# 13_Configure_Local_Admin_RDP_Firewall_And_Remote_Management_GPOs_Verification_Commands
| Command | Purpose | Healthy Output |
|---|---|---|
| `Get-GPO -Name "<access-gpo-name>"` | Confirms access GPO exists | GPO object returns |
| `Get-GPInheritance -Target "<pilot-ou-dn>"` | Confirms GPO is linked to pilot OU | GPO appears in link list |
| `Get-GPPermission -Name "<access-gpo-name>" -All` | Confirms target computers can read and apply GPO | Expected permissions appear |
| `Get-ADComputer -Identity "<test-computer>" -Properties DistinguishedName` | Confirms computer is in pilot scope | Computer DN is under pilot OU |
| `Get-ADGroup -Identity "<local-admin-group>"` | Confirms approved local admin group exists | Group object returns |
| `Get-ADGroup -Identity "<rdp-group>"` | Confirms approved RDP group exists | Group object returns |
| `gpupdate /force` | Forces computer policy refresh | Policy update completes |
| `gpresult /scope computer /r` | Shows applied computer GPOs | Access GPO appears under Applied GPOs |
| `gpresult /h C:\GPOPrep\Reports\gpresult-remote-admin-access.html` | Exports full RSOP report | HTML report exists |
| `Get-LocalGroupMember -Group Administrators` | Validates local admin membership | Approved group appears |
| `Get-LocalGroupMember -Group "Remote Desktop Users"` | Validates RDP local group membership | Approved group appears |
| `secedit /export /cfg C:\GPOPrep\Reports\effective-security-policy.inf` | Exports effective user rights | INF export exists |
| `Get-ItemProperty "HKLM:\System\CurrentControlSet\Control\Terminal Server"` | Validates RDP enabled state | `fDenyTSConnections = 0` if RDP enabled |
| `Get-ItemProperty "HKLM:\System\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp"` | Validates NLA setting | `UserAuthentication = 1` |
| `Get-Service TermService,WinRM` | Validates remote access services | Services are visible and expected state is known |
| `Get-NetFirewallProfile` | Validates firewall profiles | Domain profile enabled |
| `Get-NetFirewallRule | Where-Object DisplayGroup -like "*Remote Desktop*"` | Validates RDP firewall rules | Expected rules enabled |
| `Get-NetFirewallRule | Where-Object DisplayGroup -like "*Windows Remote Management*"` | Validates WinRM firewall rules | Expected rules enabled |
| `Test-NetConnection <test-computer> -Port 3389` | Tests RDP TCP reachability from management host | TCP succeeds if RDP allowed |
| `Test-WSMan <test-computer>` | Tests WinRM from management host | WSMan response returns |
| `Get-CimInstance -ClassName Win32_OperatingSystem -ComputerName <test-computer>` | Tests CIM or WMI remote management | Remote OS object returns if allowed |
| `Get-WinEvent -LogName "Microsoft-Windows-GroupPolicy/Operational" -MaxEvents 150` | Reviews policy processing events | Recent policy processing events appear |
| `Get-GPOReport -Name "<access-gpo-name>" -ReportType Html -Path "<report-path>\<access-gpo-name>.html"` | Exports readable GPO report | HTML report exists |
| `Backup-GPO -Name "<access-gpo-name>" -Path "<backup-path>"` | Backs up configured GPO | Backup completes |

# 13_Configure_Local_Admin_RDP_Firewall_And_Remote_Management_GPOs_Rollback
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: safely rollback local admin, RDP, firewall, and remote management GPO settings.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$DomainDN = (Get-ADDomain).DistinguishedName
$PilotOU = "OU=Pilot,OU=Workstations,OU=Corp,$DomainDN"

$GpoName = "CORP-Workstation-Remote-Admin-Access"

$ReportPath = "C:\GPOPrep\Reports"
$BackupPath = "C:\GPOPrep\Backup"

New-Item -ItemType Directory -Force -Path $ReportPath,$BackupPath

# Capture state before rollback.
Get-GPOReport `
  -Name $GpoName `
  -ReportType Html `
  -Path "$ReportPath\$GpoName-before-rollback.html"

Backup-GPO `
  -Name $GpoName `
  -Path $BackupPath

# Rollback option 1:
# Disable the GPO link while preserving the GPO for review.
Set-GPLink `
  -Name $GpoName `
  -Target $PilotOU `
  -LinkEnabled No `
  -ErrorAction SilentlyContinue

# Rollback option 2:
# Remove known RDP registry policy values if they were configured through Set-GPRegistryValue.

Remove-GPRegistryValue `
  -Name $GpoName `
  -Key "HKLM\System\CurrentControlSet\Control\Terminal Server" `
  -ValueName "fDenyTSConnections" `
  -ErrorAction SilentlyContinue

Remove-GPRegistryValue `
  -Name $GpoName `
  -Key "HKLM\System\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" `
  -ValueName "UserAuthentication" `
  -ErrorAction SilentlyContinue

# Rollback option 3:
# Use GPMC to remove GPP local group membership items and firewall rules.
gpmc.msc

# GUI rollback workflow:
# 1. Edit the GPO.
# 2. Remove or disable Local Users and Groups preference items.
# 3. Review local Administrators membership rollback carefully.
# 4. Remove RDP user right changes if they are not required.
# 5. Set RDP Administrative Template settings back to Not Configured if required.
# 6. Remove or disable custom RDP and WinRM firewall rules.
# 7. Set WinRM service policy back to Not Configured if required.
# 8. Export a fresh GPO report.
# 9. Run gpupdate /force on test client.
# 10. Reboot if required.
# 11. Validate local groups, RDP, firewall, and WinRM state.

# Rollback option 4:
# Restore from known GPO backup if target and backup are confirmed.
# Restore-GPO `
#   -Name $GpoName `
#   -Path $BackupPath

# Rollback option 5:
# Remove the GPO only in a disposable lab.
# Remove-GPO `
#   -Name $GpoName

# Capture after rollback.
Get-GPOReport `
  -Name $GpoName `
  -ReportType Html `
  -Path "$ReportPath\$GpoName-after-rollback.html" `
  -ErrorAction SilentlyContinue

Get-GPInheritance `
  -Target $PilotOU |
  Out-File "$ReportPath\pilot-ou-inheritance-after-remote-admin-rollback.txt"

Write-Host "Rollback action complete."
Write-Host "On test client: run gpupdate /force, reboot if required, then validate local groups, RDP, firewall, and WinRM."
```

# 13_Configure_Local_Admin_RDP_Firewall_And_Remote_Management_GPOs_Failure_Checks
| Symptom | Likely Cause | Check Command | Fix |
|---|---|---|---|
| Access GPO does not apply | Computer object not in linked pilot OU | `Get-ADComputer -Identity "<test-computer>" -Properties DistinguishedName` | Move computer to pilot OU or link GPO to correct OU |
| GPO appears denied | Security filtering or WMI filter blocks computer | `gpresult /scope computer /r`; `Get-GPPermission` | Grant Apply permission or fix WMI filter |
| Approved admin group not in local Administrators | GPP item missing, wrong action, targeting failed, or policy not refreshed | `Get-LocalGroupMember Administrators`; `gpresult /h` | Fix GPP local group item and refresh policy |
| Local Administrators membership wiped | GPP Replace or delete all members option misused | GPO report and local group membership | Disable link, restore break-glass access, correct GPP action |
| RDP group not in local Remote Desktop Users | GPP item missing or wrong local group name | `Get-LocalGroupMember "Remote Desktop Users"` | Correct group preference item |
| User is in RDP group but cannot sign in | User right missing or Deny right overrides Allow | `secedit /export`; `gpresult /h` | Configure Allow log on through Remote Desktop Services and remove incorrect Deny |
| RDP port closed | Firewall rule missing, RDP disabled, or service not listening | `Test-NetConnection <client> -Port 3389`; `Get-Service TermService` | Enable RDP policy and firewall rule |
| RDP works from anywhere | Firewall rule source scope too broad | `Get-NetFirewallRule`; `Get-NetFirewallAddressFilter` | Scope rule to management subnet |
| WinRM fails | WinRM policy missing, service disabled, firewall blocked, or network profile issue | `Test-WSMan <client>`; `Get-Service WinRM` | Enable WinRM policy and firewall rule |
| WinRM open to too many systems | IPv4 filter or firewall scope too broad | GPO report and firewall address filters | Restrict to management subnet |
| WMI remote query fails | Firewall rule missing, DCOM/RPC blocked, permissions issue | `Get-CimInstance -ComputerName <client>` | Enable required WMI rules or use WinRM/CIM over WSMan |
| Remote Event Viewer fails | Remote Event Log firewall group disabled | Firewall rule check | Enable Remote Event Log Management for management subnet |
| Services MMC fails remotely | Remote Service Management firewall group disabled | Firewall rule check | Enable Remote Service Management if required |
| NLA prevents RDP sign-in | Client lacks proper credentials or NLA setting mismatch | RDP client error and event logs | Confirm user rights, group membership, and NLA support |
| Firewall policy blocks management tools | Default inbound block without required allow rules | `Get-NetFirewallProfile`; firewall logs | Add scoped allow rules or disable pilot link |
| GPO report does not show settings | Wrong GPO edited or stale report | `Get-GPOReport -Name "<access-gpo-name>"` | Edit correct GPO and export fresh report |
| Local group membership changes not immediate | Policy refresh, reboot, or GPP processing delay | `gpupdate /force`; event logs | Refresh policy and reboot if needed |
| Computer group filtering not reflected | Computer token stale | `gpresult /scope computer /r` | Reboot computer |
| Domain group membership not reflected for user test | User token stale | `whoami /groups` | Sign out and sign back in |
| GPO differs between DCs | AD or SYSVOL replication issue | `repadmin /replsummary`; `dcdiag /test:sysvolcheck` | Fix replication before expanding scope |
| Remote access rollback does not work immediately | Policy refresh or reboot required | `gpupdate /force`; `Restart-Computer` | Refresh and reboot test client |
| Locked out of pilot machine | Local admin membership or firewall policy removed access | Console access, break-glass account, disable GPO link | Disable link, force refresh, restore local admin group |

# 13_Configure_Local_Admin_RDP_Firewall_And_Remote_Management_GPOs_Related_Labs
| Related Lab | Relationship |
|---|---|
| `00_Group_Policy_Index.md` | Defines suite order and dependency path |
| `01_Install_Group_Policy_Management_Tools.md` | Provides GPMC and GroupPolicy PowerShell module |
| `02_Create_And_Link_Baseline_GPO.md` | Establishes GPO creation and linking workflow |
| `03_Configure_GPO_Link_Order_Inheritance_Enforcement_And_Blocking.md` | Controls precedence before remote access policy rollout |
| `04_Configure_GPO_Security_Filtering_And_Delegation.md` | Controls which computers can apply the remote admin GPO |
| `05_Configure_WMI_Filtering_For_GPO_Targeting.md` | Adds optional OS targeting before access policy application |
| `06_Configure_Computer_Administrative_Template_Settings.md` | Provides computer ADMX policy foundation |
| `08_Configure_Group_Policy_Preferences.md` | Provides GPP Local Users and Groups foundation |
| `11_Configure_Security_Baseline_GPO_Settings.md` | Security baseline companion workbook |
| `12_Configure_Windows_Update_And_Defender_GPO_Settings.md` | Related endpoint management and hardening GPO |
| `Create_Baseline_OU_Structure.md` | Provides pilot and workstation OU targets |
| `Create_Baseline_Groups_And_Test_Users.md` | Provides local admin, RDP, and filtering groups |
| `Join_Windows_Client_To_Domain.md` | Provides domain-joined test client for validation |
| `Verify_SYSVOL_NETLOGON_DC_Locator_And_FSMO_Roles.md` | Confirms SYSVOL and DC locator before troubleshooting remote admin GPO |
| `14_Backup_Restore_Import_Copy_And_Migrate_GPOs.md` | Next workbook for managing GPO backup, restore, and migration workflows |