04_Configure_GPO_Security_Filtering_And_Delegation.md
# 04_Configure_GPO_Security_Filtering_And_Delegation

# 04_Configure_GPO_Security_Filtering_And_Delegation_Index
04_Configure_GPO_Security_Filtering_And_Delegation.md
04_Configure_GPO_Security_Filtering_And_Delegation
04_Configure_GPO_Security_Filtering_And_Delegation_Source_Basis
04_Configure_GPO_Security_Filtering_And_Delegation_Mental_Model
04_Configure_GPO_Security_Filtering_And_Delegation_Planning_Table
04_Configure_GPO_Security_Filtering_And_Delegation_Configuration_Checklist
04_Configure_GPO_Security_Filtering_And_Delegation_Precheck_Skeleton
04_Configure_GPO_Security_Filtering_And_Delegation_Create_Filtering_Groups_Skeleton
04_Configure_GPO_Security_Filtering_And_Delegation_Apply_Security_Filtering_Skeleton
04_Configure_GPO_Security_Filtering_And_Delegation_Delegate_GPO_Administration_Skeleton
04_Configure_GPO_Security_Filtering_And_Delegation_Client_RSOP_Validation_Skeleton
04_Configure_GPO_Security_Filtering_And_Delegation_Report_And_Backup_Skeleton
04_Configure_GPO_Security_Filtering_And_Delegation_Verification_Commands
04_Configure_GPO_Security_Filtering_And_Delegation_Rollback
04_Configure_GPO_Security_Filtering_And_Delegation_Failure_Checks
04_Configure_GPO_Security_Filtering_And_Delegation_Related_Labs

# 04_Configure_GPO_Security_Filtering_And_Delegation_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Group Policy Management Console Delegation tab | Viewing and modifying GPO permissions |
| Microsoft Learn | Group Policy Security Filtering | Controlling which users, groups, or computers can apply a GPO |
| Microsoft Learn | `Set-GPPermission` | Granting read, apply, edit, and delegated GPO permissions |
| Microsoft Learn | `Get-GPPermission` | Auditing GPO security permissions |
| Microsoft Learn | `Get-GPOReport` | Exporting GPO configuration and permission reports |
| Microsoft Learn | `Backup-GPO` | Preserving GPO state before changing ACLs |
| Active Directory PowerShell | `New-ADGroup`, `Add-ADGroupMember`, `Get-ADGroupMember` | Creating and managing GPO filtering and delegation groups |
| Windows client policy processing | `gpupdate`, `gpresult`, RSOP, GroupPolicy event logs | Proving that filtering allows or denies GPO application correctly |
| Windows Server operational practice | Separate apply groups from admin delegation groups | Preventing accidental broad policy scope or overdelegation |

# 04_Configure_GPO_Security_Filtering_And_Delegation_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Security filtering | ACL-based control over who can apply a linked GPO |
| Apply Group Policy | Permission that allows a principal to receive the GPO |
| Read | Permission that allows the GPO to be read during processing |
| GpoApply | PowerShell permission level that grants Read plus Apply Group Policy |
| GpoRead | PowerShell permission level that grants read-only access without applying the GPO |
| GpoEdit | Allows editing the GPO settings but not deleting or changing security |
| GpoEditDeleteModifySecurity | Allows full control over the GPO |
| Authenticated Users | Default broad principal often used for read and apply unless filtering is narrowed |
| Domain Computers | Important read principal for user-side GPO processing in modern Windows environments |
| Filtering group | AD security group used to control who receives the GPO |
| Delegation group | AD security group used to control who can manage the GPO |
| Scope | A GPO must be linked and the target must pass filtering for it to apply |
| Denied GPO | GPO visible to the client but not applied due to security filtering, WMI, disabled link, or disabled GPO side |
| User filtering | Usually filters user-side GPOs by user or user group |
| Computer filtering | Usually filters computer-side GPOs by computer account or computer group |
| First rule | Link controls where the GPO can apply; filtering controls who within that path can apply |
| Blunt rule | Do not remove broad read permissions unless you know what account must still read the GPO |

# 04_Configure_GPO_Security_Filtering_And_Delegation_Planning_Table
| Item | Example | Decision |
|---|---|---|
| Domain FQDN | `corp.local` | `<domain-fqdn>` |
| Domain DN | `DC=corp,DC=local` | `<domain-dn>` |
| Target OU | `OU=Workstations,OU=Corp,DC=corp,DC=local` | `<target-ou-dn>` |
| Baseline GPO | `CORP-Workstation-Baseline` | `<baseline-gpo-name>` |
| Policy side | Computer | `<computer-or-user>` |
| Test computer | `WIN11-01` | `<test-computer>` |
| Test user | `tuser` | `<test-user>` |
| Filtering group | `GG_GPO_Workstation_Baseline_Apply` | `<filtering-group>` |
| Delegation group | `GG_GPO_Workstation_GPO_Admins` | `<delegation-group>` |
| Filtering group OU | `OU=Groups,OU=Corp,DC=corp,DC=local` | `<groups-ou-dn>` |
| Computer to add | `WIN11-01$` | `<computer-sam-account>` |
| User to add | `tuser` | `<user-sam-account>` |
| Keep Authenticated Users read | Yes | `<yes-no>` |
| Add Domain Computers read | Yes | `<yes-no>` |
| Delegation permission level | `GpoEdit` | `<delegation-level>` |
| Report path | `C:\GPOPrep\Reports` | `<report-path>` |
| Backup path | `C:\GPOPrep\Backup` | `<backup-path>` |
| Rollback method | Restore broad apply or restore backup | `<rollback-plan>` |

# 04_Configure_GPO_Security_Filtering_And_Delegation_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Open elevated PowerShell | Management Host | `whoami /groups` | Admin token is visible |
| 2 | Import AD module | Management Host | `Import-Module ActiveDirectory` | AD module imports |
| 3 | Import GroupPolicy module | Management Host | `Import-Module GroupPolicy` | GroupPolicy module imports |
| 4 | Confirm domain identity | Management Host | `Get-ADDomain` | Domain object returns |
| 5 | Confirm target OU exists | Management Host | `Get-ADOrganizationalUnit -Identity "<target-ou-dn>"` | Target OU returns |
| 6 | Confirm baseline GPO exists | Management Host | `Get-GPO -Name "<baseline-gpo-name>"` | GPO returns |
| 7 | Confirm GPO link exists | Management Host | `Get-GPInheritance -Target "<target-ou-dn>"` | Baseline GPO appears in link list |
| 8 | Confirm SYSVOL is reachable | Management Host | `Test-Path "\\<domain-fqdn>\SYSVOL\<domain-fqdn>\Policies"` | Returns True |
| 9 | Create report folder | Management Host | `New-Item -ItemType Directory -Force -Path C:\GPOPrep\Reports` | Report folder exists |
| 10 | Create backup folder | Management Host | `New-Item -ItemType Directory -Force -Path C:\GPOPrep\Backup` | Backup folder exists |
| 11 | Back up current GPO before ACL change | Management Host | `Backup-GPO -Name "<baseline-gpo-name>" -Path C:\GPOPrep\Backup` | Pre-change backup exists |
| 12 | Export current GPO report | Management Host | `Get-GPOReport -Name "<baseline-gpo-name>" -ReportType Html -Path C:\GPOPrep\Reports\<baseline-gpo-name>-before-filtering.html` | Before report exists |
| 13 | Capture current GPO permissions | Management Host | `Get-GPPermission -Name "<baseline-gpo-name>" -All` | Current ACL is visible |
| 14 | Create filtering group if missing | Management Host | `New-ADGroup -Name "<filtering-group>" -SamAccountName "<filtering-group>" -GroupScope Global -GroupCategory Security -Path "<groups-ou-dn>"` | Filtering group exists |
| 15 | Create delegation group if missing | Management Host | `New-ADGroup -Name "<delegation-group>" -SamAccountName "<delegation-group>" -GroupScope Global -GroupCategory Security -Path "<groups-ou-dn>"` | Delegation group exists |
| 16 | Add test computer to filtering group for computer GPO | Management Host | `Add-ADGroupMember -Identity "<filtering-group>" -Members "<test-computer>$"` | Test computer is a member |
| 17 | Add test user to filtering group for user GPO if required | Management Host | `Add-ADGroupMember -Identity "<filtering-group>" -Members "<test-user>"` | Test user is a member |
| 18 | Grant filtering group Apply permission | Management Host | `Set-GPPermission -Name "<baseline-gpo-name>" -TargetName "<filtering-group>" -TargetType Group -PermissionLevel GpoApply` | Filtering group can apply GPO |
| 19 | Preserve Authenticated Users read access | Management Host | `Set-GPPermission -Name "<baseline-gpo-name>" -TargetName "Authenticated Users" -TargetType Group -PermissionLevel GpoRead` | Authenticated Users can read but not apply |
| 20 | Add Domain Computers read access for user-side processing safety | Management Host | `Set-GPPermission -Name "<baseline-gpo-name>" -TargetName "Domain Computers" -TargetType Group -PermissionLevel GpoRead` | Domain Computers can read GPO |
| 21 | Delegate edit permission to GPO admin group | Management Host | `Set-GPPermission -Name "<baseline-gpo-name>" -TargetName "<delegation-group>" -TargetType Group -PermissionLevel GpoEdit` | Delegation group can edit GPO |
| 22 | Confirm updated GPO permissions | Management Host | `Get-GPPermission -Name "<baseline-gpo-name>" -All` | Apply, read, and edit permissions match plan |
| 23 | Confirm filtering group membership | Management Host | `Get-ADGroupMember -Identity "<filtering-group>"` | Intended users or computers are members |
| 24 | Confirm delegation group membership | Management Host | `Get-ADGroupMember -Identity "<delegation-group>"` | Intended admins are members |
| 25 | Force AD replication if multi-DC lab requires it | Domain Controller | `repadmin /syncall /AdeP` | ACL and group membership replicate |
| 26 | Force policy refresh on test client | Test Client | `gpupdate /force` | Policy refresh completes |
| 27 | Confirm applied computer GPOs | Test Client | `gpresult /scope computer /r` | Baseline GPO appears under Applied GPOs when computer is in filtering group |
| 28 | Confirm denied GPOs on out-of-scope client | Test Client | `gpresult /scope computer /r` | GPO appears denied or absent for non-member target |
| 29 | Export GPResult HTML report | Test Client | `gpresult /h C:\GPOPrep\Reports\gpresult-security-filtering.html` | HTML report exists |
| 30 | Export final GPO report | Management Host | `Get-GPOReport -Name "<baseline-gpo-name>" -ReportType Html -Path C:\GPOPrep\Reports\<baseline-gpo-name>-after-filtering.html` | After report exists |
| 31 | Back up final GPO state | Management Host | `Backup-GPO -Name "<baseline-gpo-name>" -Path C:\GPOPrep\Backup` | Post-change backup exists |
| 32 | Document filtering and delegation design | Operator | `Record GPO, linked OU, filtering group, delegation group, permissions, test client, and validation result` | Security filtering and delegation decision is documented |

# 04_Configure_GPO_Security_Filtering_And_Delegation_Precheck_Skeleton
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: capture GPO, OU, group, and permission state before security filtering changes.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$Domain = Get-ADDomain
$DomainFqdn = $Domain.DNSRoot
$DomainDN = $Domain.DistinguishedName

$TargetOU = "OU=Workstations,OU=Corp,$DomainDN"
$GroupsOU = "OU=Groups,OU=Corp,$DomainDN"

$GpoName = "CORP-Workstation-Baseline"
$FilteringGroup = "GG_GPO_Workstation_Baseline_Apply"
$DelegationGroup = "GG_GPO_Workstation_GPO_Admins"

$TestComputer = "WIN11-01"
$TestUser = "tuser"

$BasePath = "C:\GPOPrep"
$ReportPath = "$BasePath\Reports"
$BackupPath = "$BasePath\Backup"

New-Item -ItemType Directory -Force -Path $BasePath,$ReportPath,$BackupPath

# Confirm domain and tools.
$Domain | Select-Object DNSRoot,NetBIOSName,DistinguishedName
Get-Module -ListAvailable GroupPolicy
Get-Command -Module GroupPolicy | Select-Object Name | Sort-Object Name

# Confirm DC locator and SYSVOL.
nltest /dsgetdc:$DomainFqdn
Test-Path "\\$DomainFqdn\SYSVOL\$DomainFqdn\Policies"
Test-Path "\\$DomainFqdn\NETLOGON"

# Confirm OU and GPO.
Get-ADOrganizationalUnit -Identity $TargetOU
Get-ADOrganizationalUnit -Identity $GroupsOU
Get-GPO -Name $GpoName

# Confirm current link state.
Get-GPInheritance -Target $TargetOU |
  Out-File "$ReportPath\inheritance-before-security-filtering.txt"

# Confirm test principals.
Get-ADComputer -Identity $TestComputer -Properties DistinguishedName,Enabled |
  Select-Object Name,Enabled,DistinguishedName |
  Out-File "$ReportPath\test-computer-before-security-filtering.txt"

Get-ADUser -Identity $TestUser -Properties DistinguishedName,Enabled |
  Select-Object SamAccountName,Enabled,DistinguishedName |
  Out-File "$ReportPath\test-user-before-security-filtering.txt"

# Capture current GPO permissions.
Get-GPPermission -Name $GpoName -All |
  Out-File "$ReportPath\$GpoName-permissions-before.txt"

# Export current report and backup.
Get-GPOReport `
  -Name $GpoName `
  -ReportType Html `
  -Path "$ReportPath\$GpoName-before-security-filtering.html"

Backup-GPO `
  -Name $GpoName `
  -Path $BackupPath
```

# 04_Configure_GPO_Security_Filtering_And_Delegation_Create_Filtering_Groups_Skeleton
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: create security groups used for GPO application and GPO administration delegation.

Import-Module ActiveDirectory

$DomainDN = (Get-ADDomain).DistinguishedName
$GroupsOU = "OU=Groups,OU=Corp,$DomainDN"

$FilteringGroup = "GG_GPO_Workstation_Baseline_Apply"
$DelegationGroup = "GG_GPO_Workstation_GPO_Admins"

$TestComputer = "WIN11-01"
$TestUser = "tuser"

# Confirm Groups OU exists.
Get-ADOrganizationalUnit -Identity $GroupsOU

# Create filtering group if missing.
if (-not (Get-ADGroup -Identity $FilteringGroup -ErrorAction SilentlyContinue)) {
    New-ADGroup `
      -Name $FilteringGroup `
      -SamAccountName $FilteringGroup `
      -GroupScope Global `
      -GroupCategory Security `
      -Path $GroupsOU `
      -Description "Security filtering group used to apply workstation baseline GPO"
}

# Create delegation group if missing.
if (-not (Get-ADGroup -Identity $DelegationGroup -ErrorAction SilentlyContinue)) {
    New-ADGroup `
      -Name $DelegationGroup `
      -SamAccountName $DelegationGroup `
      -GroupScope Global `
      -GroupCategory Security `
      -Path $GroupsOU `
      -Description "Delegated administrators for workstation baseline GPO"
}

# Add test computer to filtering group for computer-side GPO validation.
# Computer accounts use trailing dollar sign.
Add-ADGroupMember `
  -Identity $FilteringGroup `
  -Members "$TestComputer$" `
  -ErrorAction SilentlyContinue

# Optional: add test user if the same GPO includes user-side settings.
Add-ADGroupMember `
  -Identity $FilteringGroup `
  -Members $TestUser `
  -ErrorAction SilentlyContinue

# Confirm membership.
Get-ADGroupMember -Identity $FilteringGroup |
  Select-Object Name,SamAccountName,ObjectClass

Get-ADGroupMember -Identity $DelegationGroup |
  Select-Object Name,SamAccountName,ObjectClass
```

# 04_Configure_GPO_Security_Filtering_And_Delegation_Apply_Security_Filtering_Skeleton
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: narrow GPO application to a filtering group while preserving read access.

Import-Module GroupPolicy

$GpoName = "CORP-Workstation-Baseline"
$FilteringGroup = "GG_GPO_Workstation_Baseline_Apply"

# Confirm GPO exists.
Get-GPO -Name $GpoName

# Capture current permissions.
Get-GPPermission -Name $GpoName -All

# Grant the filtering group permission to read and apply the GPO.
Set-GPPermission `
  -Name $GpoName `
  -TargetName $FilteringGroup `
  -TargetType Group `
  -PermissionLevel GpoApply

# Safe default:
# Preserve Authenticated Users read access but remove broad apply behavior.
# This keeps the GPO readable while preventing every authenticated principal from applying it.
Set-GPPermission `
  -Name $GpoName `
  -TargetName "Authenticated Users" `
  -TargetType Group `
  -PermissionLevel GpoRead

# Add Domain Computers read access.
# This is especially important for user-side GPO processing in modern Windows environments.
Set-GPPermission `
  -Name $GpoName `
  -TargetName "Domain Computers" `
  -TargetType Group `
  -PermissionLevel GpoRead

# Confirm final permissions.
Get-GPPermission `
  -Name $GpoName `
  -All |
  Format-Table Trustee,TrusteeType,Permission -AutoSize
```

# 04_Configure_GPO_Security_Filtering_And_Delegation_Delegate_GPO_Administration_Skeleton
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: delegate GPO edit rights without granting full Domain Admin access.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$GpoName = "CORP-Workstation-Baseline"
$DelegationGroup = "GG_GPO_Workstation_GPO_Admins"
$DelegatedAdminUser = "gpo.admin"

# Confirm GPO and group.
Get-GPO -Name $GpoName
Get-ADGroup -Identity $DelegationGroup

# Optional: add delegated admin user to delegation group.
# Replace gpo.admin with the intended account.
Add-ADGroupMember `
  -Identity $DelegationGroup `
  -Members $DelegatedAdminUser `
  -ErrorAction SilentlyContinue

# Grant edit permission.
# GpoEdit allows editing settings but not deleting the GPO or modifying security.
Set-GPPermission `
  -Name $GpoName `
  -TargetName $DelegationGroup `
  -TargetType Group `
  -PermissionLevel GpoEdit

# Alternative full-control delegation:
# Use only for highly trusted GPO administrators.
# Set-GPPermission `
#   -Name $GpoName `
#   -TargetName $DelegationGroup `
#   -TargetType Group `
#   -PermissionLevel GpoEditDeleteModifySecurity

# Confirm final permissions.
Get-GPPermission `
  -Name $GpoName `
  -All |
  Sort-Object Trustee |
  Format-Table Trustee,TrusteeType,Permission -AutoSize

# Confirm delegation group membership.
Get-ADGroupMember -Identity $DelegationGroup |
  Select-Object Name,SamAccountName,ObjectClass
```

# 04_Configure_GPO_Security_Filtering_And_Delegation_Client_RSOP_Validation_Skeleton
```powershell
# Run on the test Windows client.
# Purpose: prove security filtering allows the intended target to apply the GPO.

$DomainFqdn = "corp.local"
$ReportPath = "C:\GPOPrep\Reports"

New-Item -ItemType Directory -Force -Path $ReportPath

# Confirm client identity and domain state.
hostname
whoami
Get-ComputerInfo | Select-Object CsName,CsDomain,CsPartOfDomain

# Confirm DC locator.
nltest /dsgetdc:$DomainFqdn

# Purge and refresh Kerberos tickets if testing group membership quickly.
# A reboot is often cleaner for computer group membership changes.
klist purge

# Force policy refresh.
gpupdate /force

# Validate applied and denied computer GPOs.
gpresult /scope computer /r

# Validate applied and denied user GPOs.
gpresult /scope user /r

# Export full HTML report.
gpresult /h "$ReportPath\gpresult-security-filtering.html"

# Review recent Group Policy processing events.
Get-WinEvent `
  -LogName "Microsoft-Windows-GroupPolicy/Operational" `
  -MaxEvents 100 |
  Select-Object TimeCreated,Id,LevelDisplayName,Message
```

# 04_Configure_GPO_Security_Filtering_And_Delegation_Report_And_Backup_Skeleton
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: capture final permission, report, inheritance, and backup state.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$DomainDN = (Get-ADDomain).DistinguishedName

$TargetOU = "OU=Workstations,OU=Corp,$DomainDN"
$GpoName = "CORP-Workstation-Baseline"
$FilteringGroup = "GG_GPO_Workstation_Baseline_Apply"
$DelegationGroup = "GG_GPO_Workstation_GPO_Admins"

$BasePath = "C:\GPOPrep"
$ReportPath = "$BasePath\Reports"
$BackupPath = "$BasePath\Backup"

New-Item -ItemType Directory -Force -Path $ReportPath,$BackupPath

# Capture final inheritance.
Get-GPInheritance `
  -Target $TargetOU |
  Out-File "$ReportPath\inheritance-after-security-filtering.txt"

# Capture final permissions.
Get-GPPermission `
  -Name $GpoName `
  -All |
  Out-File "$ReportPath\$GpoName-permissions-after.txt"

# Capture group membership.
Get-ADGroupMember `
  -Identity $FilteringGroup |
  Select-Object Name,SamAccountName,ObjectClass |
  Out-File "$ReportPath\$FilteringGroup-members.txt"

Get-ADGroupMember `
  -Identity $DelegationGroup |
  Select-Object Name,SamAccountName,ObjectClass |
  Out-File "$ReportPath\$DelegationGroup-members.txt"

# Export GPO report.
Get-GPOReport `
  -Name $GpoName `
  -ReportType Html `
  -Path "$ReportPath\$GpoName-after-security-filtering.html"

Get-GPOReport `
  -Name $GpoName `
  -ReportType Xml `
  -Path "$ReportPath\$GpoName-after-security-filtering.xml"

# Back up final GPO state.
Backup-GPO `
  -Name $GpoName `
  -Path $BackupPath

Write-Host "Security filtering and delegation reporting complete."
Write-Host "Report path: $ReportPath"
Write-Host "Backup path: $BackupPath"
```

# 04_Configure_GPO_Security_Filtering_And_Delegation_Verification_Commands
| Command | Purpose | Healthy Output |
|---|---|---|
| `Get-GPO -Name "<baseline-gpo-name>"` | Confirms GPO exists | GPO object returns |
| `Get-GPPermission -Name "<baseline-gpo-name>" -All` | Shows GPO ACL permissions | Filtering and delegation permissions match design |
| `Get-ADGroup -Identity "<filtering-group>"` | Confirms filtering group exists | Group object returns |
| `Get-ADGroupMember -Identity "<filtering-group>"` | Confirms target users or computers are members | Intended members appear |
| `Get-ADGroup -Identity "<delegation-group>"` | Confirms delegation group exists | Group object returns |
| `Get-ADGroupMember -Identity "<delegation-group>"` | Confirms delegated admins are members | Intended admins appear |
| `Set-GPPermission -Name "<gpo-name>" -TargetName "<filtering-group>" -TargetType Group -PermissionLevel GpoApply` | Grants Apply permission to filtering group | Filtering group can apply GPO |
| `Set-GPPermission -Name "<gpo-name>" -TargetName "Authenticated Users" -TargetType Group -PermissionLevel GpoRead` | Preserves read without broad apply | Authenticated Users has read permission |
| `Set-GPPermission -Name "<gpo-name>" -TargetName "Domain Computers" -TargetType Group -PermissionLevel GpoRead` | Preserves computer read access | Domain Computers has read permission |
| `Set-GPPermission -Name "<gpo-name>" -TargetName "<delegation-group>" -TargetType Group -PermissionLevel GpoEdit` | Delegates GPO editing | Delegation group can edit GPO |
| `Get-GPInheritance -Target "<target-ou-dn>"` | Confirms GPO is still linked to target OU | GPO link appears |
| `gpupdate /force` | Refreshes client policy | Policy update completes |
| `gpresult /scope computer /r` | Confirms computer-side applied and denied GPOs | Intended GPO applies only to allowed computer |
| `gpresult /scope user /r` | Confirms user-side applied and denied GPOs | Intended GPO applies only to allowed user |
| `gpresult /h C:\GPOPrep\Reports\gpresult-security-filtering.html` | Exports client RSOP report | HTML report exists |
| `Get-WinEvent -LogName "Microsoft-Windows-GroupPolicy/Operational" -MaxEvents 100` | Reviews processing events | Successful processing or clear denial reason appears |
| `repadmin /replsummary` | Checks replication after ACL/group changes | No unexpected failures |

# 04_Configure_GPO_Security_Filtering_And_Delegation_Rollback
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: rollback security filtering and delegation to broad lab-safe defaults.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$GpoName = "CORP-Workstation-Baseline"
$FilteringGroup = "GG_GPO_Workstation_Baseline_Apply"
$DelegationGroup = "GG_GPO_Workstation_GPO_Admins"

$ReportPath = "C:\GPOPrep\Reports"
New-Item -ItemType Directory -Force -Path $ReportPath

# Capture before rollback.
Get-GPPermission -Name $GpoName -All |
  Out-File "$ReportPath\$GpoName-permissions-before-rollback.txt"

# Restore broad default lab behavior:
# Authenticated Users can read and apply the GPO.
Set-GPPermission `
  -Name $GpoName `
  -TargetName "Authenticated Users" `
  -TargetType Group `
  -PermissionLevel GpoApply `
  -ErrorAction SilentlyContinue

# Keep Domain Computers read access.
Set-GPPermission `
  -Name $GpoName `
  -TargetName "Domain Computers" `
  -TargetType Group `
  -PermissionLevel GpoRead `
  -ErrorAction SilentlyContinue

# Remove filtering group permission if needed.
Set-GPPermission `
  -Name $GpoName `
  -TargetName $FilteringGroup `
  -TargetType Group `
  -PermissionLevel None `
  -ErrorAction SilentlyContinue

# Remove delegation group permission if needed.
Set-GPPermission `
  -Name $GpoName `
  -TargetName $DelegationGroup `
  -TargetType Group `
  -PermissionLevel None `
  -ErrorAction SilentlyContinue

# Optional lab cleanup:
# Remove test principals from filtering and delegation groups.
# Remove-ADGroupMember -Identity $FilteringGroup -Members "WIN11-01$" -Confirm:$false
# Remove-ADGroupMember -Identity $DelegationGroup -Members "gpo.admin" -Confirm:$false

# Optional lab cleanup:
# Remove groups only if they were created solely for this lab.
# Remove-ADGroup -Identity $FilteringGroup -Confirm:$false
# Remove-ADGroup -Identity $DelegationGroup -Confirm:$false

# Capture after rollback.
Get-GPPermission -Name $GpoName -All |
  Out-File "$ReportPath\$GpoName-permissions-after-rollback.txt"

Get-GPPermission -Name $GpoName -All |
  Format-Table Trustee,TrusteeType,Permission -AutoSize
```

# 04_Configure_GPO_Security_Filtering_And_Delegation_Failure_Checks
| Symptom | Likely Cause | Check Command | Fix |
|---|---|---|---|
| `Set-GPPermission` is not recognized | GroupPolicy module missing | `Get-Module -ListAvailable GroupPolicy` | Complete `01_Install_Group_Policy_Management_Tools.md` |
| `Get-GPPermission` fails | GPO name typo or permission issue | `Get-GPO -All | Select DisplayName` | Correct GPO name or use authorized account |
| Filtering group cannot be found | Group missing or wrong SamAccountName | `Get-ADGroup -Identity "<filtering-group>"` | Create group or correct name |
| Computer cannot be added to group | Missing trailing dollar sign or wrong computer name | `Get-ADComputer -Identity "<test-computer>"` | Use `<computer-name>$` for group membership |
| GPO stops applying to everyone | Removed Authenticated Users apply but did not add filtering group apply | `Get-GPPermission -Name "<gpo-name>" -All` | Grant filtering group `GpoApply` |
| User-side GPO does not apply after filtering | GPO not readable by computer account | `Get-GPPermission -Name "<gpo-name>" -All` | Add `Domain Computers` or Authenticated Users with `GpoRead` |
| GPO appears under denied list | Target lacks Apply Group Policy permission | `gpresult /r` | Add user/computer/group to filtering group and grant `GpoApply` |
| Group membership change not reflected | Kerberos token stale or computer needs reboot | `whoami /groups`; `klist`; `gpresult /r` | Reboot client or purge tickets and refresh policy |
| Computer group filtering still fails after membership change | Computer token has not refreshed | `gpresult /scope computer /r` | Reboot computer after adding it to group |
| Delegated admin cannot edit GPO | Delegation group lacks edit permission or user token stale | `Get-GPPermission -Name "<gpo-name>" -All`; `whoami /groups` | Grant `GpoEdit` and sign out or refresh token |
| Delegated admin can edit too much | Granted `GpoEditDeleteModifySecurity` unnecessarily | `Get-GPPermission -Name "<gpo-name>" -All` | Reduce to `GpoEdit` |
| GPO applies outside intended OU | Link target too broad | `Get-GPInheritance -Target "<target-ou-dn>"` | Move link lower or combine OU targeting with filtering |
| GPO still does not apply after permissions look correct | WMI filter, disabled link, disabled GPO side, or wrong OU | `gpresult /r`; `Get-GPInheritance`; `Get-GPOReport` | Fix WMI, link, status, or object location |
| Permission report looks different in GPMC and PowerShell | Replication delay or opened different domain controller | `repadmin /replsummary`; refresh GPMC | Wait or force replication |
| `Backup-GPO` fails before ACL change | Backup path missing or access denied | `Test-Path "<backup-path>"` | Create backup path and grant write permissions |
| `Get-GPOReport` fails | Report path missing or bad filename | `Test-Path "<report-path>"` | Create report path and sanitize filename |

# 04_Configure_GPO_Security_Filtering_And_Delegation_Related_Labs
| Related Lab | Relationship |
|---|---|
| `00_Group_Policy_Index.md` | Defines GPO suite order and dependency map |
| `01_Install_Group_Policy_Management_Tools.md` | Provides GroupPolicy module and GPMC tools |
| `02_Create_And_Link_Baseline_GPO.md` | Creates and links the baseline GPO being filtered |
| `03_Configure_GPO_Link_Order_Inheritance_Enforcement_And_Blocking.md` | Confirms the GPO link and inheritance path before filtering |
| `Create_Baseline_OU_Structure.md` | Provides target OU structure for GPO scope |
| `Create_Baseline_Groups_And_Test_Users.md` | Provides AD groups, users, and test principals for filtering |
| `Join_Windows_Client_To_Domain.md` | Provides test client for RSOP and `gpresult` validation |
| `Verify_SYSVOL_NETLOGON_DC_Locator_And_FSMO_Roles.md` | Confirms policy infrastructure before troubleshooting filtering |
| `05_Configure_Computer_Security_Baseline_With_GPO.md` | Next workbook that applies computer hardening through the filtered baseline path |