03_Configure_GPO_Link_Order_Inheritance_Enforcement_And_Blocking.md
# 03_Configure_GPO_Link_Order_Inheritance_Enforcement_And_Blocking

# 03_Configure_GPO_Link_Order_Inheritance_Enforcement_And_Blocking_Index
03_Configure_GPO_Link_Order_Inheritance_Enforcement_And_Blocking.md
03_Configure_GPO_Link_Order_Inheritance_Enforcement_And_Blocking
03_Configure_GPO_Link_Order_Inheritance_Enforcement_And_Blocking_Source_Basis
03_Configure_GPO_Link_Order_Inheritance_Enforcement_And_Blocking_Mental_Model
03_Configure_GPO_Link_Order_Inheritance_Enforcement_And_Blocking_Planning_Table
03_Configure_GPO_Link_Order_Inheritance_Enforcement_And_Blocking_Configuration_Checklist
03_Configure_GPO_Link_Order_Inheritance_Enforcement_And_Blocking_Precheck_Skeleton
03_Configure_GPO_Link_Order_Inheritance_Enforcement_And_Blocking_Link_Order_Skeleton
03_Configure_GPO_Link_Order_Inheritance_Enforcement_And_Blocking_Block_Inheritance_Skeleton
03_Configure_GPO_Link_Order_Inheritance_Enforcement_And_Blocking_Enforcement_Skeleton
03_Configure_GPO_Link_Order_Inheritance_Enforcement_And_Blocking_Client_RSOP_Validation_Skeleton
03_Configure_GPO_Link_Order_Inheritance_Enforcement_And_Blocking_Report_And_Backup_Skeleton
03_Configure_GPO_Link_Order_Inheritance_Enforcement_And_Blocking_Verification_Commands
03_Configure_GPO_Link_Order_Inheritance_Enforcement_And_Blocking_Rollback
03_Configure_GPO_Link_Order_Inheritance_Enforcement_And_Blocking_Failure_Checks
03_Configure_GPO_Link_Order_Inheritance_Enforcement_And_Blocking_Related_Labs

# 03_Configure_GPO_Link_Order_Inheritance_Enforcement_And_Blocking_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Group Policy Management Console | Viewing and configuring GPO links, link order, enforced links, and block inheritance |
| Microsoft Learn | `Get-GPInheritance` | Reading linked, inherited, blocked, and enforced GPO behavior on a target container |
| Microsoft Learn | `Set-GPLink` | Configuring link enabled state, link order, and enforced state |
| Microsoft Learn | `Set-GPInheritance` | Configuring block inheritance on a domain or OU |
| Microsoft Learn | `Get-GPOReport` | Exporting GPO configuration and link reports |
| Microsoft Learn | `Backup-GPO` | Backing up GPOs before changing precedence behavior |
| Windows client policy processing | `gpupdate`, `gpresult`, RSOP, and GroupPolicy event logs | Proving effective policy after link and inheritance changes |
| Windows Server operational practice | Least surprise GPO layering | Keeping inheritance and enforcement predictable and supportable |

# 03_Configure_GPO_Link_Order_Inheritance_Enforcement_And_Blocking_Mental_Model
| Concept | Operational Meaning |
|---|---|
| GPO link | The attachment between a GPO and a site, domain, or OU |
| Link enabled | Whether the linked GPO is active for that target |
| Link order | Precedence order for GPOs linked to the same site, domain, or OU |
| Lower number | Higher precedence at the same link location |
| Order 1 | Highest precedence among GPOs linked at that same location |
| LSDOU | Processing path: Local, Site, Domain, then OU from parent to child |
| Child OU policy | Usually wins over parent OU policy when settings conflict |
| Enforced link | Forces a parent linked GPO to win and prevents block inheritance from blocking it |
| Block inheritance | OU or domain setting that blocks normal inherited GPO links from parent containers |
| Enforced beats block | Enforced GPO links still apply even when inheritance is blocked |
| GPO status | Enables or disables user and computer halves of the GPO |
| Inherited GPO | GPO linked above the current OU that still applies through inheritance |
| Direct linked GPO | GPO linked directly to the current OU, domain, or site |
| Conflict | Two or more GPOs configure the same setting differently |
| Winning setting | The effective setting after link order, OU depth, enforcement, block inheritance, security filtering, and WMI filtering |
| RSOP | Effective policy result on a target user or computer |
| First rule | Do not enforce or block inheritance unless you can explain the exact policy conflict being solved |
| Blunt rule | Link order is a scalpel; enforced and block inheritance are chainsaws |

# 03_Configure_GPO_Link_Order_Inheritance_Enforcement_And_Blocking_Planning_Table
| Item | Example | Decision |
|---|---|---|
| Domain FQDN | `corp.local` | `<domain-fqdn>` |
| Domain DN | `DC=corp,DC=local` | `<domain-dn>` |
| Parent OU | `OU=Corp,DC=corp,DC=local` | `<parent-ou-dn>` |
| Target child OU | `OU=Workstations,OU=Corp,DC=corp,DC=local` | `<target-ou-dn>` |
| Test computer | `WIN11-01` | `<test-computer>` |
| Baseline GPO | `CORP-Workstation-Baseline` | `<baseline-gpo-name>` |
| Parent baseline GPO | `CORP-Domain-Computer-Baseline` | `<parent-gpo-name>` |
| Child baseline GPO | `CORP-Workstation-Baseline` | `<child-gpo-name>` |
| Test conflict setting | Harmless registry marker | `<test-setting>` |
| Desired highest precedence GPO | `CORP-Workstation-Baseline` | `<winning-gpo>` |
| Desired link order | `1` | `<link-order>` |
| Link enabled state | `Yes` | `<yes-no>` |
| Enforced required | `No` | `<yes-no>` |
| Block inheritance required | `No` | `<yes-no>` |
| Report path | `C:\GPOPrep\Reports` | `<report-path>` |
| Backup path | `C:\GPOPrep\Backup` | `<backup-path>` |
| Rollback method | Restore link order, disable enforced, unblock inheritance | `<rollback-plan>` |

# 03_Configure_GPO_Link_Order_Inheritance_Enforcement_And_Blocking_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Open elevated PowerShell | Management Host | `whoami /groups` | Admin token is visible |
| 2 | Import AD module | Management Host | `Import-Module ActiveDirectory` | AD module imports |
| 3 | Import GroupPolicy module | Management Host | `Import-Module GroupPolicy` | GroupPolicy module imports |
| 4 | Confirm domain identity | Management Host | `Get-ADDomain` | Domain object returns |
| 5 | Confirm target OU exists | Management Host | `Get-ADOrganizationalUnit -Identity "<target-ou-dn>"` | Target OU returns |
| 6 | Confirm parent OU exists | Management Host | `Get-ADOrganizationalUnit -Identity "<parent-ou-dn>"` | Parent OU returns |
| 7 | Confirm baseline GPO exists | Management Host | `Get-GPO -Name "<baseline-gpo-name>"` | Baseline GPO returns |
| 8 | Confirm SYSVOL path | Management Host | `Test-Path "\\<domain-fqdn>\SYSVOL\<domain-fqdn>\Policies"` | Returns True |
| 9 | Create report folder | Management Host | `New-Item -ItemType Directory -Force -Path C:\GPOPrep\Reports` | Report path exists |
| 10 | Create backup folder | Management Host | `New-Item -ItemType Directory -Force -Path C:\GPOPrep\Backup` | Backup path exists |
| 11 | Capture current inheritance | Management Host | `Get-GPInheritance -Target "<target-ou-dn>"` | Current linked and inherited GPOs are visible |
| 12 | Capture current parent inheritance | Management Host | `Get-GPInheritance -Target "<parent-ou-dn>"` | Parent OU link state is visible |
| 13 | Back up all current GPOs | Management Host | `Backup-GPO -All -Path C:\GPOPrep\Backup` | Pre-change backup exists |
| 14 | Export current baseline GPO report | Management Host | `Get-GPOReport -Name "<baseline-gpo-name>" -ReportType Html -Path C:\GPOPrep\Reports\<baseline-gpo-name>-before.html` | Before report exists |
| 15 | Confirm current link state | Management Host | `Get-GPInheritance -Target "<target-ou-dn>"` | Baseline GPO link status is known |
| 16 | Enable the baseline GPO link | Management Host | `Set-GPLink -Name "<baseline-gpo-name>" -Target "<target-ou-dn>" -LinkEnabled Yes` | Link is enabled |
| 17 | Set baseline GPO to desired link order | Management Host | `Set-GPLink -Name "<baseline-gpo-name>" -Target "<target-ou-dn>" -Order 1` | Baseline GPO has desired precedence at that OU |
| 18 | Keep baseline link unenforced by default | Management Host | `Set-GPLink -Name "<baseline-gpo-name>" -Target "<target-ou-dn>" -Enforced No` | Link is not enforced |
| 19 | Confirm target OU inheritance is not blocked by default | Management Host | `Set-GPInheritance -Target "<target-ou-dn>" -IsBlocked No` | OU inherits normal parent GPOs |
| 20 | Confirm final link and inheritance state | Management Host | `Get-GPInheritance -Target "<target-ou-dn>"` | Order, enabled state, enforced state, and inheritance state are visible |
| 21 | Force client policy refresh | Test Client | `gpupdate /force` | Computer and user policy refresh completes |
| 22 | Validate applied computer GPOs | Test Client | `gpresult /scope computer /r` | Expected GPOs apply in expected order |
| 23 | Generate HTML client report | Test Client | `gpresult /h C:\GPOPrep\Reports\gpresult-link-order.html` | HTML result report exists |
| 24 | Review client processing events | Test Client | `Get-WinEvent -LogName "Microsoft-Windows-GroupPolicy/Operational" -MaxEvents 80` | Recent Group Policy events are visible |
| 25 | Export final inheritance state | Management Host | `Get-GPInheritance -Target "<target-ou-dn>" | Out-File C:\GPOPrep\Reports\inheritance-after.txt` | Final inheritance report exists |
| 26 | Export final GPO report | Management Host | `Get-GPOReport -Name "<baseline-gpo-name>" -ReportType Html -Path C:\GPOPrep\Reports\<baseline-gpo-name>-after.html` | After report exists |
| 27 | Back up baseline GPO after change | Management Host | `Backup-GPO -Name "<baseline-gpo-name>" -Path C:\GPOPrep\Backup` | Post-change backup exists |
| 28 | Document decision | Operator | `Record target OU, GPO link order, enforced state, block inheritance state, and validation client` | Precedence decision is documented |

# 03_Configure_GPO_Link_Order_Inheritance_Enforcement_And_Blocking_Precheck_Skeleton
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: capture current GPO link, inheritance, and policy state before making precedence changes.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$Domain = Get-ADDomain
$DomainFqdn = $Domain.DNSRoot
$DomainDN = $Domain.DistinguishedName

$ParentOU = "OU=Corp,$DomainDN"
$TargetOU = "OU=Workstations,$ParentOU"

$BaselineGpo = "CORP-Workstation-Baseline"

$BasePath = "C:\GPOPrep"
$ReportPath = "$BasePath\Reports"
$BackupPath = "$BasePath\Backup"

New-Item -ItemType Directory -Force -Path $BasePath,$ReportPath,$BackupPath

# Confirm domain and tool readiness.
$Domain | Select-Object DNSRoot,NetBIOSName,DistinguishedName
Get-Module -ListAvailable GroupPolicy
Get-Command -Module GroupPolicy | Select-Object Name | Sort-Object Name

# Confirm DC locator and SYSVOL.
nltest /dsgetdc:$DomainFqdn
Test-Path "\\$DomainFqdn\SYSVOL\$DomainFqdn\Policies"
Test-Path "\\$DomainFqdn\NETLOGON"

# Confirm OU targets.
Get-ADOrganizationalUnit -Identity $ParentOU
Get-ADOrganizationalUnit -Identity $TargetOU

# Confirm GPO exists.
Get-GPO -Name $BaselineGpo

# Capture before state.
Get-GPInheritance -Target $ParentOU |
  Out-File "$ReportPath\parent-ou-inheritance-before.txt"

Get-GPInheritance -Target $TargetOU |
  Out-File "$ReportPath\target-ou-inheritance-before.txt"

Get-GPO -All |
  Sort-Object DisplayName |
  Select-Object DisplayName,Id,Owner,GpoStatus,CreationTime,ModificationTime |
  Export-Csv "$ReportPath\gpo-inventory-before-link-order.csv" -NoTypeInformation

# Export GPO report before changes.
Get-GPOReport `
  -Name $BaselineGpo `
  -ReportType Html `
  -Path "$ReportPath\$BaselineGpo-before-link-order.html"

# Back up GPOs before precedence changes.
Backup-GPO -All -Path $BackupPath
```

# 03_Configure_GPO_Link_Order_Inheritance_Enforcement_And_Blocking_Link_Order_Skeleton
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: set controlled link order for a baseline GPO on a target OU.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$DomainDN = (Get-ADDomain).DistinguishedName
$TargetOU = "OU=Workstations,OU=Corp,$DomainDN"

$BaselineGpo = "CORP-Workstation-Baseline"
$DesiredOrder = 1

# Confirm OU and GPO exist.
Get-ADOrganizationalUnit -Identity $TargetOU
Get-GPO -Name $BaselineGpo

# Confirm current inheritance and links.
$Before = Get-GPInheritance -Target $TargetOU
$Before | Format-List

# Confirm link exists. Create it if missing.
$ExistingLink = $Before.GpoLinks |
  Where-Object { $_.DisplayName -eq $BaselineGpo }

if (-not $ExistingLink) {
    New-GPLink `
      -Name $BaselineGpo `
      -Target $TargetOU `
      -LinkEnabled Yes

    Write-Host "Created missing link for $BaselineGpo on $TargetOU"
}

# Ensure the link is enabled.
Set-GPLink `
  -Name $BaselineGpo `
  -Target $TargetOU `
  -LinkEnabled Yes

# Set link order.
Set-GPLink `
  -Name $BaselineGpo `
  -Target $TargetOU `
  -Order $DesiredOrder

# Keep unenforced unless the design specifically requires enforcement.
Set-GPLink `
  -Name $BaselineGpo `
  -Target $TargetOU `
  -Enforced No

# Confirm final state.
Get-GPInheritance -Target $TargetOU |
  Format-List
```

# 03_Configure_GPO_Link_Order_Inheritance_Enforcement_And_Blocking_Block_Inheritance_Skeleton
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: demonstrate block inheritance control on an OU.
# Default operational stance: leave inheritance unblocked unless isolation is required.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$DomainDN = (Get-ADDomain).DistinguishedName
$TargetOU = "OU=Workstations,OU=Corp,$DomainDN"

$ReportPath = "C:\GPOPrep\Reports"
New-Item -ItemType Directory -Force -Path $ReportPath

# Confirm target OU exists.
Get-ADOrganizationalUnit -Identity $TargetOU

# Capture current state.
Get-GPInheritance -Target $TargetOU |
  Out-File "$ReportPath\target-ou-before-block-inheritance-change.txt"

# Default safe configuration: do not block inheritance.
Set-GPInheritance `
  -Target $TargetOU `
  -IsBlocked No

# Lab-only example: block inheritance when intentionally testing isolated OU policy.
# Set-GPInheritance `
#   -Target $TargetOU `
#   -IsBlocked Yes

# Confirm final state.
Get-GPInheritance -Target $TargetOU |
  Out-File "$ReportPath\target-ou-after-block-inheritance-change.txt"

Get-GPInheritance -Target $TargetOU |
  Format-List
```

# 03_Configure_GPO_Link_Order_Inheritance_Enforcement_And_Blocking_Enforcement_Skeleton
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: control GPO link enforcement.
# Default operational stance: do not enforce links unless a parent policy must override lower-level settings.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$DomainDN = (Get-ADDomain).DistinguishedName

$ParentOU = "OU=Corp,$DomainDN"
$TargetOU = "OU=Workstations,$ParentOU"

$ParentGpo = "CORP-Domain-Computer-Baseline"
$ChildGpo = "CORP-Workstation-Baseline"

$ReportPath = "C:\GPOPrep\Reports"
New-Item -ItemType Directory -Force -Path $ReportPath

# Confirm targets.
Get-ADOrganizationalUnit -Identity $ParentOU
Get-ADOrganizationalUnit -Identity $TargetOU

# Capture before state.
Get-GPInheritance -Target $ParentOU |
  Out-File "$ReportPath\parent-ou-before-enforcement.txt"

Get-GPInheritance -Target $TargetOU |
  Out-File "$ReportPath\target-ou-before-enforcement.txt"

# Safe default: keep child baseline GPO not enforced.
Set-GPLink `
  -Name $ChildGpo `
  -Target $TargetOU `
  -Enforced No `
  -ErrorAction SilentlyContinue

# Safe default: keep parent baseline GPO not enforced unless there is a documented reason.
Set-GPLink `
  -Name $ParentGpo `
  -Target $ParentOU `
  -Enforced No `
  -ErrorAction SilentlyContinue

# Lab-only example: enforce parent GPO.
# Use only if the parent GPO must override child OU settings or bypass block inheritance.
# Set-GPLink `
#   -Name $ParentGpo `
#   -Target $ParentOU `
#   -Enforced Yes

# Confirm after state.
Get-GPInheritance -Target $ParentOU |
  Out-File "$ReportPath\parent-ou-after-enforcement.txt"

Get-GPInheritance -Target $TargetOU |
  Out-File "$ReportPath\target-ou-after-enforcement.txt"

Get-GPInheritance -Target $TargetOU |
  Format-List
```

# 03_Configure_GPO_Link_Order_Inheritance_Enforcement_And_Blocking_Client_RSOP_Validation_Skeleton
```powershell
# Run on the test Windows client.
# Purpose: prove effective policy after link order, inheritance, enforcement, or block inheritance changes.

$DomainFqdn = "corp.local"
$ReportPath = "C:\GPOPrep\Reports"

New-Item -ItemType Directory -Force -Path $ReportPath

# Confirm client identity.
hostname
whoami
Get-ComputerInfo | Select-Object CsName,CsDomain,CsPartOfDomain

# Confirm DC locator.
nltest /dsgetdc:$DomainFqdn

# Force policy refresh.
gpupdate /force

# Confirm applied and denied computer-side GPOs.
gpresult /scope computer /r

# Confirm applied and denied user-side GPOs.
gpresult /scope user /r

# Export full HTML result.
gpresult /h "$ReportPath\gpresult-link-order-inheritance.html"

# Optional RSOP report through PowerShell.
Get-GPResultantSetOfPolicy `
  -ReportType Html `
  -Path "$ReportPath\rsop-link-order-inheritance.html" `
  -ErrorAction SilentlyContinue

# Review recent Group Policy processing events.
Get-WinEvent `
  -LogName "Microsoft-Windows-GroupPolicy/Operational" `
  -MaxEvents 80 |
  Select-Object TimeCreated,Id,LevelDisplayName,Message
```

# 03_Configure_GPO_Link_Order_Inheritance_Enforcement_And_Blocking_Report_And_Backup_Skeleton
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: capture final GPO link, inheritance, report, and backup state.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$DomainDN = (Get-ADDomain).DistinguishedName

$ParentOU = "OU=Corp,$DomainDN"
$TargetOU = "OU=Workstations,$ParentOU"

$BaselineGpo = "CORP-Workstation-Baseline"

$BasePath = "C:\GPOPrep"
$ReportPath = "$BasePath\Reports"
$BackupPath = "$BasePath\Backup"

New-Item -ItemType Directory -Force -Path $ReportPath,$BackupPath

# Capture final inheritance.
Get-GPInheritance -Target $ParentOU |
  Out-File "$ReportPath\parent-ou-inheritance-after.txt"

Get-GPInheritance -Target $TargetOU |
  Out-File "$ReportPath\target-ou-inheritance-after.txt"

# Export final GPO report.
Get-GPOReport `
  -Name $BaselineGpo `
  -ReportType Html `
  -Path "$ReportPath\$BaselineGpo-after-link-order.html"

Get-GPOReport `
  -Name $BaselineGpo `
  -ReportType Xml `
  -Path "$ReportPath\$BaselineGpo-after-link-order.xml"

# Backup final GPO state.
Backup-GPO `
  -Name $BaselineGpo `
  -Path $BackupPath

# Capture full GPO inventory after change.
Get-GPO -All |
  Sort-Object DisplayName |
  Select-Object DisplayName,Id,Owner,GpoStatus,CreationTime,ModificationTime |
  Export-Csv "$ReportPath\gpo-inventory-after-link-order.csv" -NoTypeInformation

Write-Host "GPO link order, inheritance, enforcement, and blocking report complete."
Write-Host "Report path: $ReportPath"
Write-Host "Backup path: $BackupPath"
```

# 03_Configure_GPO_Link_Order_Inheritance_Enforcement_And_Blocking_Verification_Commands
| Command | Purpose | Healthy Output |
|---|---|---|
| `Get-GPO -Name "<gpo-name>"` | Confirms GPO exists | GPO object returns |
| `Get-ADOrganizationalUnit -Identity "<target-ou-dn>"` | Confirms target OU exists | OU object returns |
| `Get-GPInheritance -Target "<target-ou-dn>"` | Shows linked GPOs, inherited GPOs, order, enforced state, and block inheritance | Expected link state appears |
| `Set-GPLink -Name "<gpo-name>" -Target "<target-ou-dn>" -LinkEnabled Yes` | Enables GPO link | Link is enabled |
| `Set-GPLink -Name "<gpo-name>" -Target "<target-ou-dn>" -Order 1` | Sets highest link precedence at that target | GPO appears at desired link order |
| `Set-GPLink -Name "<gpo-name>" -Target "<target-ou-dn>" -Enforced No` | Confirms safe default unenforced state | GPO is not enforced |
| `Set-GPInheritance -Target "<target-ou-dn>" -IsBlocked No` | Confirms safe default inheritance state | OU inherits normal parent GPOs |
| `Get-GPOReport -Name "<gpo-name>" -ReportType Html -Path "<path>"` | Exports readable GPO report | HTML report exists |
| `Backup-GPO -Name "<gpo-name>" -Path "<backup-path>"` | Backs up GPO after precedence changes | Backup completes |
| `gpupdate /force` | Forces client policy refresh | Policy update completes |
| `gpresult /scope computer /r` | Shows effective computer-side GPOs | Expected GPOs appear under Applied Group Policy Objects |
| `gpresult /scope user /r` | Shows effective user-side GPOs | Expected GPOs appear under Applied Group Policy Objects |
| `gpresult /h C:\GPOPrep\Reports\gpresult-link-order.html` | Creates full client policy report | HTML report exists |
| `Get-WinEvent -LogName "Microsoft-Windows-GroupPolicy/Operational" -MaxEvents 80` | Reviews processing events | Recent policy processing events appear |
| `repadmin /replsummary` | Checks AD replication when GPO state differs by DC | No unexpected failures |
| `dcdiag /test:sysvolcheck /test:advertising` | Checks SYSVOL and DC advertising | Tests pass |

# 03_Configure_GPO_Link_Order_Inheritance_Enforcement_And_Blocking_Rollback
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: rollback GPO precedence changes to safe defaults.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$DomainDN = (Get-ADDomain).DistinguishedName

$ParentOU = "OU=Corp,$DomainDN"
$TargetOU = "OU=Workstations,$ParentOU"

$ParentGpo = "CORP-Domain-Computer-Baseline"
$ChildGpo = "CORP-Workstation-Baseline"

$ReportPath = "C:\GPOPrep\Reports"
New-Item -ItemType Directory -Force -Path $ReportPath

# Capture state before rollback.
Get-GPInheritance -Target $ParentOU |
  Out-File "$ReportPath\parent-ou-before-precedence-rollback.txt"

Get-GPInheritance -Target $TargetOU |
  Out-File "$ReportPath\target-ou-before-precedence-rollback.txt"

# Safe rollback default 1: disable enforcement on parent and child links.
Set-GPLink `
  -Name $ParentGpo `
  -Target $ParentOU `
  -Enforced No `
  -ErrorAction SilentlyContinue

Set-GPLink `
  -Name $ChildGpo `
  -Target $TargetOU `
  -Enforced No `
  -ErrorAction SilentlyContinue

# Safe rollback default 2: unblock inheritance on target OU.
Set-GPInheritance `
  -Target $TargetOU `
  -IsBlocked No `
  -ErrorAction SilentlyContinue

# Safe rollback default 3: keep baseline child link enabled and set it to order 1 if this is the intended lab state.
Set-GPLink `
  -Name $ChildGpo `
  -Target $TargetOU `
  -LinkEnabled Yes `
  -ErrorAction SilentlyContinue

Set-GPLink `
  -Name $ChildGpo `
  -Target $TargetOU `
  -Order 1 `
  -ErrorAction SilentlyContinue

# Optional rollback: disable the child link entirely if the baseline must stop applying.
# Set-GPLink `
#   -Name $ChildGpo `
#   -Target $TargetOU `
#   -LinkEnabled No

# Capture after rollback.
Get-GPInheritance -Target $ParentOU |
  Out-File "$ReportPath\parent-ou-after-precedence-rollback.txt"

Get-GPInheritance -Target $TargetOU |
  Out-File "$ReportPath\target-ou-after-precedence-rollback.txt"

Get-GPInheritance -Target $TargetOU |
  Format-List
```

# 03_Configure_GPO_Link_Order_Inheritance_Enforcement_And_Blocking_Failure_Checks
| Symptom | Likely Cause | Check Command | Fix |
|---|---|---|---|
| `Set-GPLink` is not recognized | GroupPolicy module missing | `Get-Module -ListAvailable GroupPolicy` | Complete `01_Install_Group_Policy_Management_Tools.md` |
| `Set-GPInheritance` is not recognized | GroupPolicy module missing or not imported | `Import-Module GroupPolicy`; `Get-Command Set-GPInheritance` | Install or import GroupPolicy module |
| `Set-GPLink` cannot find GPO | GPO name typo or GPO missing | `Get-GPO -All | Select DisplayName` | Correct GPO name or create GPO |
| `Set-GPLink` cannot find target | OU distinguished name typo | `Get-ADOrganizationalUnit -Filter *` | Correct target OU DN |
| Link order did not change | Multiple links or wrong target container | `Get-GPInheritance -Target "<target-ou-dn>"` | Set order on the exact linked target |
| GPO link appears but does not apply | Security filtering, WMI filter, disabled link, disabled GPO half, or wrong OU | `gpresult /r`; `Get-GPInheritance` | Fix filtering, WMI, link state, GPO status, or object location |
| Parent GPO still applies after block inheritance | Parent GPO link is enforced | `Get-GPInheritance -Target "<target-ou-dn>"` | Remove enforced state or accept enforced behavior |
| Child OU setting does not win | Parent GPO is enforced | `Get-GPInheritance -Target "<target-ou-dn>"` | Disable enforcement on parent link if override is not intended |
| Too many GPOs stopped applying | Block inheritance enabled too broadly | `Get-GPInheritance -Target "<target-ou-dn>"` | Set `Set-GPInheritance -IsBlocked No` |
| GPO applies to unexpected computers | Link target too broad | `Get-GPInheritance`; `Get-ADComputer -Properties DistinguishedName` | Link lower in OU tree or add security filtering |
| `gpresult` shows GPO denied | Security filtering, WMI filter, or empty GPO | `gpresult /r`; GPO report | Fix security filtering, WMI query, or add actual setting |
| GPO not visible from all DCs | AD or SYSVOL replication issue | `repadmin /replsummary`; `dcdiag /test:sysvolcheck` | Fix replication before changing more policy |
| Client still sees old policy | Background refresh delay, DC cache, or stale logon session | `gpupdate /force`; reboot client | Force refresh, reboot, or wait for replication |
| GPO report fails | Path missing or invalid filename | `Test-Path "<report-path>"` | Create report folder and sanitize file name |
| Backup fails | Backup path missing or access denied | `Test-Path "<backup-path>"` | Create backup folder and grant write permissions |
| User policy behaves differently than computer policy | User and computer objects are in different OU paths | `gpresult /scope user /r`; `gpresult /scope computer /r` | Link user and computer GPOs to the correct OU paths |
| Precedence is confusing in GPMC | Enforced, blocked inheritance, and link order are mixed together | `Get-GPInheritance -Target "<target-ou-dn>"` | Simplify: remove enforcement, unblock inheritance, then set link order |

# 03_Configure_GPO_Link_Order_Inheritance_Enforcement_And_Blocking_Related_Labs
| Related Lab | Relationship |
|---|---|
| `00_Group_Policy_Index.md` | Defines the full GPO suite and build order |
| `01_Install_Group_Policy_Management_Tools.md` | Provides GPMC and GroupPolicy PowerShell module |
| `02_Create_And_Link_Baseline_GPO.md` | Creates the baseline GPO and first OU link |
| `Create_Baseline_OU_Structure.md` | Provides OU hierarchy where inheritance and link order are tested |
| `Create_Baseline_Groups_And_Test_Users.md` | Provides users and computers for RSOP validation |
| `Join_Windows_Client_To_Domain.md` | Provides domain-joined client for `gpupdate` and `gpresult` |
| `Verify_SYSVOL_NETLOGON_DC_Locator_And_FSMO_Roles.md` | Confirms SYSVOL, NETLOGON, DC locator, and domain health |
| `04_Configure_GPO_Security_Filtering_And_Delegation.md` | Next workbook for controlling who can apply or administer GPOs |