00_Group_Policy_Index.md
# 00_Group_Policy_Index

# 00_Group_Policy_Index_Index
00_Group_Policy_Index.md
00_Group_Policy_Index
00_Group_Policy_Index_Source_Basis
00_Group_Policy_Index_Mental_Model
00_Group_Policy_Index_Planning_Table
00_Group_Policy_Index_Configuration_Checklist
00_Group_Policy_Index_GPO_Suite_Map
00_Group_Policy_Index_Build_Order
00_Group_Policy_Index_Dependency_Matrix
00_Group_Policy_Index_Validation_Matrix
00_Group_Policy_Index_Naming_Standards
00_Group_Policy_Index_Operator_Runbook_Skeleton
00_Group_Policy_Index_Verification_Commands
00_Group_Policy_Index_Rollback
00_Group_Policy_Index_Failure_Checks
00_Group_Policy_Index_Related_Labs

# 00_Group_Policy_Index_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Windows Server Group Policy | GPOs, links, inheritance, security filtering, delegation, and processing | Full GPO operational suite |
| Group Policy Management Console | GPMC creation, linking, backup, restore, reports, modeling, and results | GUI-based GPO administration |
| GroupPolicy PowerShell module | `New-GPO`, `New-GPLink`, `Get-GPO`, `Backup-GPO`, `Restore-GPO`, `Get-GPInheritance`, `Get-GPResultantSetOfPolicy` | Repeatable GPO configuration and validation |
| Active Directory Domain Services | OUs, users, computers, security groups, SYSVOL, replication, domain policy | GPO targeting and replication foundation |
| SYSVOL and DFSR | Policy template replication and GPT delivery | Proving GPO files replicate between domain controllers |
| Windows client policy engine | `gpupdate`, `gpresult`, RSOP, GroupPolicy event logs | Client-side policy processing and troubleshooting |
| Windows security baselines | Security settings, audit policy, firewall, Defender, account policy, user rights | Practical baseline hardening through GPO |
| Windows Server operational practice | OU targeting, change control, policy layering, rollback, and reporting | Supportable production-style GPO management |

# 00_Group_Policy_Index_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Group Policy | AD-based configuration system for users and computers |
| GPO | Group Policy Object containing policy settings and preference settings |
| GPO link | Attachment point between a GPO and a site, domain, or OU |
| Scope of Management | Site, domain, or OU where a linked GPO can apply |
| Scope of Application | Actual users or computers that receive the GPO after inheritance, filtering, and WMI evaluation |
| Computer Configuration | Policy half that applies to computer accounts at startup or background refresh |
| User Configuration | Policy half that applies to user accounts at logon or background refresh |
| LSDOU | Processing order: Local, Site, Domain, OU |
| Link order | Order of linked GPO processing at the same container level |
| Enforced | Prevents lower-level block inheritance from blocking the linked GPO |
| Block inheritance | OU setting that blocks normal inherited GPOs from parent containers |
| Security filtering | ACL-based targeting that controls which users or computers can apply a GPO |
| Delegation | Administrative permissions over GPO edit, link, read, and apply rights |
| WMI filter | Conditional targeting based on WMI query results |
| Loopback processing | Computer-side setting that changes how user policy applies on shared or special-purpose machines |
| Group Policy Preferences | Non-enforced preference items such as drive maps, printers, files, registry, scheduled tasks, and shortcuts |
| ADMX Central Store | SYSVOL location where domain-wide policy definition files are stored |
| SYSVOL | Domain share that stores Group Policy templates and scripts |
| GPT | Group Policy Template, file portion of a GPO stored in SYSVOL |
| GPC | Group Policy Container, AD portion of a GPO stored in Active Directory |
| CSE | Client Side Extension that processes a category of GPO settings |
| RSOP | Resultant Set of Policy, the effective policy after processing rules are applied |
| First rule | Do not troubleshoot advanced GPO behavior until DNS, domain join, OU placement, SYSVOL, and replication are clean |
| Blunt rule | A GPO that is not linked, not in scope, denied by filtering, blocked by WMI, or missing from SYSVOL does not matter |

# 00_Group_Policy_Index_Planning_Table
| Item | Example | Decision |
|---|---|---|
| GPO suite folder | `2. Windows Servers/Group Policy` | `<gpo-suite-folder>` |
| Domain FQDN | `corp.local` | `<domain-fqdn>` |
| NetBIOS domain | `CORP` | `<netbios-name>` |
| Primary domain controller | `DC1` | `<primary-dc>` |
| Secondary domain controller | `DC2` | `<secondary-dc>` |
| Management host | `MGMT01` | `<management-host>` |
| Admin account | `CORP\Administrator` or delegated GPO admin | `<admin-account>` |
| Baseline root OU | `OU=Corp,DC=corp,DC=local` | `<root-ou-dn>` |
| Workstations OU | `OU=Workstations,OU=Corp,DC=corp,DC=local` | `<workstations-ou>` |
| Servers OU | `OU=Servers,OU=Corp,DC=corp,DC=local` | `<servers-ou>` |
| Users OU | `OU=Users,OU=Corp,DC=corp,DC=local` | `<users-ou>` |
| Admins OU | `OU=Admins,OU=Corp,DC=corp,DC=local` | `<admins-ou>` |
| Test workstation | `WIN11-01` | `<test-workstation>` |
| Test server | `MEMBER01` | `<test-server>` |
| Test user | `tuser` | `<test-user>` |
| Baseline GPO prefix | `CORP-Baseline` | `<gpo-prefix>` |
| Backup path | `C:\GPO-Backup` | `<gpo-backup-path>` |
| Report path | `C:\GPO-Reports` | `<gpo-report-path>` |
| RSOP path | `C:\GPO-Reports\RSOP` | `<rsop-path>` |
| ADMX Central Store path | `\\corp.local\SYSVOL\corp.local\Policies\PolicyDefinitions` | `<central-store-path>` |
| Change control stance | Backup before edit | `<change-control-standard>` |
| Rollback stance | Restore GPO backup or unlink GPO | `<rollback-plan>` |

# 00_Group_Policy_Index_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Create Group Policy suite folder | Management Host | `New-Item -ItemType Directory -Force -Path ".\2. Windows Servers\Group Policy"` | GPO suite folder exists |
| 2 | Create index workbook | Management Host | `New-Item -ItemType File -Force -Path ".\2. Windows Servers\Group Policy\00_Group_Policy_Index.md"` | GPO index file exists |
| 3 | Confirm domain identity | DC or Management Host | `Get-ADDomain` | Domain object returns expected DNS root and DN |
| 4 | Confirm forest identity | DC or Management Host | `Get-ADForest` | Forest object returns expected forest root |
| 5 | Confirm Group Policy module exists | DC or Management Host | `Get-Module -ListAvailable GroupPolicy` | GroupPolicy module is available |
| 6 | Confirm GPMC tools are installed | Management Host | `Get-WindowsFeature GPMC` | GPMC state is known |
| 7 | Install GPMC if missing | Management Host | `Install-WindowsFeature GPMC` | GPMC tools are installed |
| 8 | Confirm AD module exists | DC or Management Host | `Get-Module -ListAvailable ActiveDirectory` | ActiveDirectory module is available |
| 9 | Confirm OU baseline exists | DC or Management Host | `Get-ADOrganizationalUnit -Filter * -SearchBase "<root-ou-dn>"` | Target OUs are visible |
| 10 | Confirm test computer domain join | Test Client | `Get-ComputerInfo \| Select-Object CsName,CsDomain,CsPartOfDomain` | Client is joined to expected domain |
| 11 | Confirm SYSVOL share exists | DC or Management Host | `Test-Path "\\<domain-fqdn>\SYSVOL\<domain-fqdn>\Policies"` | SYSVOL Policies path is reachable |
| 12 | Confirm NETLOGON share exists | DC or Management Host | `Test-Path "\\<domain-fqdn>\NETLOGON"` | NETLOGON path is reachable |
| 13 | Confirm AD replication health | Domain Controller | `repadmin /replsummary` | Replication failures are absent or known |
| 14 | Confirm SYSVOL health | Domain Controller | `dcdiag /test:sysvolcheck /test:advertising` | DC advertises and SYSVOL is healthy |
| 15 | Inventory existing GPOs | DC or Management Host | `Get-GPO -All` | Existing GPOs are visible |
| 16 | Inventory inheritance on core OUs | DC or Management Host | `Get-GPInheritance -Target "<workstations-ou>"` | Current inherited and linked GPOs are visible |
| 17 | Create report folder | DC or Management Host | `New-Item -ItemType Directory -Force -Path "<gpo-report-path>"` | Report folder exists |
| 18 | Export existing GPO reports | DC or Management Host | `Get-GPO -All \| ForEach-Object { Get-GPOReport -Guid $_.Id -ReportType Html -Path "<gpo-report-path>\$($_.DisplayName).html" }` | Current GPO reports are saved |
| 19 | Create backup folder | DC or Management Host | `New-Item -ItemType Directory -Force -Path "<gpo-backup-path>"` | Backup folder exists |
| 20 | Back up existing GPOs | DC or Management Host | `Backup-GPO -All -Path "<gpo-backup-path>"` | GPO backup exists before changes |
| 21 | Validate client policy baseline | Test Client | `gpupdate /force`; `gpresult /r` | Client can process domain policy |
| 22 | Generate client RSOP report | Test Client | `gpresult /h C:\GPO-Reports\gpresult.html` | HTML policy report is created |
| 23 | Review Group Policy client events | Test Client | `Get-WinEvent -LogName "Microsoft-Windows-GroupPolicy/Operational" -MaxEvents 100` | Recent policy processing events are visible |
| 24 | Build task 01 first | Management Host | `Install-WindowsFeature GPMC` | GPO management tools are ready |
| 25 | Build task 02 second | Management Host | `New-GPO -Name "<baseline-gpo-name>"` | First baseline GPO exists |
| 26 | Build linking and scope tasks before settings-heavy tasks | Management Host | `New-GPLink`; `Get-GPInheritance` | Targeting path is supportable |
| 27 | Document GPO baseline | Operator | `Record domain, OUs, test machines, backup path, report path, and build order` | GPO suite control record is complete |

# 00_Group_Policy_Index_GPO_Suite_Map
| Task | Workbook | Purpose | Depends On | Output |
|---:|---|---|---|---|
| 00 | `00_Group_Policy_Index.md` | Suite map, dependency control, validation matrix, and build order | Existing AD lab context | GPO suite control document |
| 01 | `01_Install_Group_Policy_Management_Tools.md` | Install GPMC and GroupPolicy management tools | Domain-joined admin host or DC | GPO management tools ready |
| 02 | `02_Create_And_Link_A_Baseline_GPO.md` | Create first GPO and link it to a target OU | 01, baseline OU structure | Baseline GPO linked |
| 03 | `03_Configure_GPO_Link_Order_Enforcement_And_Block_Inheritance.md` | Control inheritance, link order, enforced links, and block inheritance | 02 | Predictable GPO precedence |
| 04 | `04_Configure_GPO_Security_Filtering_And_Delegation.md` | Target GPOs using ACLs and delegate GPO management safely | 02, AD groups | Controlled GPO scope and admin delegation |
| 05 | `05_Configure_Computer_Security_Baseline_With_GPO.md` | Configure computer-side hardening baseline | 02 through 04 | Computer security baseline |
| 06 | `06_Configure_User_Desktop_Control_Panel_And_Explorer_Settings.md` | Configure user-side desktop and shell restrictions | 02 through 04 | User environment baseline |
| 07 | `07_Configure_Password_Account_Lockout_And_Kerberos_Policies.md` | Configure domain account policy and Kerberos policy | AD domain baseline | Domain password and Kerberos policy |
| 08 | `08_Configure_Local_Users_Groups_User_Rights_And_Restricted_Groups.md` | Configure local admin control and user rights assignment | 05 | Local privilege baseline |
| 09 | `09_Configure_Windows_Firewall_And_Defender_Settings_With_GPO.md` | Configure firewall and Defender policy through GPO | 05 | Endpoint protection baseline |
| 10 | `10_Configure_Audit_Policy_Advanced_Audit_And_Event_Log_Settings.md` | Configure advanced audit policy and event log sizing | 05 | Audit and log baseline |
| 11 | `11_Configure_Logon_Startup_Scripts_And_PowerShell_Execution.md` | Configure startup, shutdown, logon, logoff scripts, and PowerShell execution controls | 02 through 04 | Script policy baseline |
| 12 | `12_Configure_Mapped_Drives_Printers_Files_And_Shortcuts_With_GPP.md` | Use Group Policy Preferences for user resources | 02 through 04 | Drive, printer, file, and shortcut preferences |
| 13 | `13_Configure_Folder_Redirection_And_Known_Folder_Settings.md` | Configure user data redirection paths | File services plan, 06 | Folder redirection baseline |
| 14 | `14_Configure_Windows_Update_And_WSUS_Client_Settings_With_GPO.md` | Configure Windows Update and WSUS client policy | WSUS or update design | Update client baseline |
| 15 | `15_Backup_Restore_Import_Copy_And_Migrate_GPOs.md` | Back up, restore, copy, import, and migrate GPOs | Existing GPOs | GPO recovery and migration path |
| 16 | `16_Troubleshoot_Group_Policy_Processing_And_Replication.md` | Troubleshoot policy processing, SYSVOL, AD replication, CSEs, and RSOP | 01 through 15 | Core GPO troubleshooting workflow |
| 17 | `17_Configure_ADMX_Central_Store_And_Policy_Definitions.md` | Configure ADMX Central Store and policy definition management | 01, SYSVOL healthy | Centralized policy definitions |
| 18 | `18_Configure_WMI_Filters_For_GPO_Targeting.md` | Configure WMI filters for conditional GPO application | 02 through 04 | WMI-targeted GPOs |
| 19 | `19_Configure_Group_Policy_Preferences_Item_Level_Targeting.md` | Configure item-level targeting for GPP items | 12 | Granular preference targeting |
| 20 | `20_Configure_Loopback_Processing_For_RDS_Kiosk_And_Shared_Computers.md` | Configure loopback processing for special-use computers | 03, 04, user/computer OU plan | Controlled user policy on shared systems |
| 21 | `21_Configure_Starter_GPOs_And_Baseline_Templates.md` | Use starter GPOs and templates for reusable baselines | 15, 17 | Repeatable baseline templates |
| 22 | `22_Configure_GPO_Modeling_RSOP_And_Resultant_Set_Analysis.md` | Model and analyze effective policy before deployment | 02 through 16 | Policy impact reports |
| 23 | `23_Configure_GPO_Change_Control_Backup_And_Reporting_Workflow.md` | Formalize backup, report, change, and review workflow | 15 | GPO operational control process |
| 24 | `24_Troubleshoot_GPO_Security_Filtering_WMI_And_Inheritance.md` | Troubleshoot targeting and precedence failures | 03, 04, 18, 19 | Targeting troubleshooting workflow |
| 25 | `25_Troubleshoot_Group_Policy_Slow_Logon_CSE_And_WMI_Filter_Delays.md` | Troubleshoot slow logons, slow CSEs, and WMI filter delays | 16, 18, 20 | Advanced performance troubleshooting workflow |

# 00_Group_Policy_Index_Build_Order
| Phase | Tasks | Goal | Stop Condition |
|---:|---|---|---|
| 1 | 00, 01 | Establish GPO suite and install management tools | GPMC and GroupPolicy module work |
| 2 | 02, 03, 04 | Build GPO creation, linking, inheritance, filtering, and delegation foundation | A test GPO applies only where intended |
| 3 | 05, 06 | Build basic computer and user configuration baselines | Client receives expected computer and user settings |
| 4 | 07, 08, 09, 10 | Build security, account policy, local rights, firewall, Defender, audit, and event log baselines | Security settings apply and verify through RSOP |
| 5 | 11, 12, 13, 14 | Build scripts, preferences, folder redirection, and update policy behavior | User and computer experience settings apply cleanly |
| 6 | 15, 16 | Build backup, restore, reporting, and troubleshooting workflow | GPO changes can be backed out and diagnosed |
| 7 | 17, 18, 19, 20, 21 | Build advanced targeting, ADMX, loopback, and template behavior | Advanced targeting applies predictably |
| 8 | 22, 23, 24, 25 | Build modeling, change control, targeting troubleshooting, and slow logon troubleshooting | Advanced GPO support workflow is complete |

# 00_Group_Policy_Index_Dependency_Matrix
| Feature Area | Required Beforehand | Do Not Start Until |
|---|---|---|
| GPMC install | Domain-joined admin host or DC | Admin rights and network path to DC are confirmed |
| GPO creation | GroupPolicy module or GPMC installed | Domain connectivity and permissions are confirmed |
| GPO linking | OU, domain, or site target exists | Target DN is known and documented |
| Link order | Multiple GPO links on same target | Existing precedence is inventoried |
| Enforced links | Parent policy must override lower OU blocks | Risk of overriding child OU policy is approved |
| Block inheritance | OU policy isolation is required | Inherited GPO impact is understood |
| Security filtering | AD groups and target principals exist | `Read` and `Apply group policy` effects are understood |
| Delegation | GPO admin roles are defined | Helpdesk or admin group names are known |
| Computer configuration | Computer objects are in target OUs | Test computer can run `gpupdate` and `gpresult` |
| User configuration | User objects are in target OUs | Test user logon path is available |
| Account policy | Domain-level policy design is known | You understand domain policy vs local policy boundary |
| Local users and groups | Security baseline is defined | Break-glass local admin plan exists |
| Firewall and Defender | Endpoint security design is known | Remote admin access requirements are documented |
| Audit policy | Logging and SIEM/event review design exists | Event log sizing and retention are chosen |
| Scripts | Script path and execution context are known | SYSVOL and permissions are confirmed |
| Group Policy Preferences | User or computer resource design exists | Item-level targeting requirements are known |
| Folder redirection | File server, share, NTFS, and backup plan exist | Data path and rollback plan are approved |
| Windows Update / WSUS | Update source design exists | WSUS or Microsoft Update behavior is chosen |
| GPO backup and restore | Backup path exists | Existing GPO state is inventoried |
| Troubleshooting | Test client exists | DNS, domain join, SYSVOL, and AD replication are healthy |
| ADMX Central Store | SYSVOL replication is healthy | Current ADMX source version is chosen |
| WMI filters | Target OS/build criteria are defined | Query performance impact is accepted |
| Loopback processing | Shared computer scenario exists | User/computer OU model is clear |
| RSOP and modeling | GPOs and users/computers exist | Desired policy outcome is documented |

# 00_Group_Policy_Index_Validation_Matrix
| Validation Area | Command | Good Result |
|---|---|---|
| Domain identity | `Get-ADDomain` | Expected domain DNS root and distinguished name |
| Forest identity | `Get-ADForest` | Expected forest root domain |
| GPMC installed | `Get-WindowsFeature GPMC` | Installed |
| GroupPolicy module available | `Get-Module -ListAvailable GroupPolicy` | Module appears |
| Existing GPO inventory | `Get-GPO -All` | Domain GPOs are listed |
| GPO details | `Get-GPO -Name "<gpo-name>"` | Target GPO returns |
| GPO link state | `Get-GPInheritance -Target "<ou-dn>"` | Linked and inherited GPOs are visible |
| GPO report | `Get-GPOReport -Name "<gpo-name>" -ReportType Html -Path "<report-path>\<gpo-name>.html"` | HTML report is created |
| GPO backup | `Backup-GPO -Name "<gpo-name>" -Path "<backup-path>"` | Backup ID is returned |
| SYSVOL path | `Test-Path "\\<domain-fqdn>\SYSVOL\<domain-fqdn>\Policies"` | Returns True |
| NETLOGON path | `Test-Path "\\<domain-fqdn>\NETLOGON"` | Returns True |
| AD replication summary | `repadmin /replsummary` | No unexpected failures |
| DC health | `dcdiag /test:sysvolcheck /test:advertising` | Tests pass |
| DFSR SYSVOL backlog | `dfsrdiag backlog /rgname:"Domain System Volume" /rfname:"SYSVOL Share" /smem:<dc1> /rmem:<dc2>` | No major backlog |
| Client policy refresh | `gpupdate /force` | Computer and user policy update completes |
| Client summary result | `gpresult /r` | Expected applied GPOs appear |
| Client HTML result | `gpresult /h C:\GPO-Reports\gpresult.html` | HTML report is created |
| RSOP report through PowerShell | `Get-GPResultantSetOfPolicy -ReportType Html -Path "<rsop-path>\rsop.html"` | RSOP HTML report is created |
| Group Policy events | `Get-WinEvent -LogName "Microsoft-Windows-GroupPolicy/Operational" -MaxEvents 100` | Recent processing events are visible |
| Applied computer policy | `gpresult /scope computer /r` | Expected computer GPOs apply |
| Applied user policy | `gpresult /scope user /r` | Expected user GPOs apply |

# 00_Group_Policy_Index_Naming_Standards
| Object Type | Naming Pattern | Example |
|---|---|---|
| Baseline computer GPO | `<ORG>-Computer-<Purpose>` | `CORP-Computer-Security-Baseline` |
| Baseline user GPO | `<ORG>-User-<Purpose>` | `CORP-User-Desktop-Baseline` |
| Server GPO | `<ORG>-Server-<Role>-<Purpose>` | `CORP-Server-DNS-Audit-Baseline` |
| Workstation GPO | `<ORG>-Workstation-<Purpose>` | `CORP-Workstation-Defender-Baseline` |
| Admin GPO | `<ORG>-Admin-<Purpose>` | `CORP-Admin-Lockdown-Baseline` |
| Preference GPO | `<ORG>-GPP-<Purpose>` | `CORP-GPP-Drive-Maps` |
| Folder redirection GPO | `<ORG>-User-Folder-Redirection` | `CORP-User-Folder-Redirection` |
| WSUS GPO | `<ORG>-Computer-Windows-Update` | `CORP-Computer-Windows-Update` |
| Test GPO | `TEST-<Purpose>-<Date>` | `TEST-Workstation-Baseline-2026-06-13` |
| Disabled GPO | `DISABLED-<OriginalName>` | `DISABLED-CORP-User-Desktop-Baseline` |
| Backup folder | `<date>-<change-ticket>-<scope>` | `2026-06-13-GPO-Baseline` |
| Report folder | `<date>-GPO-Reports` | `2026-06-13-GPO-Reports` |
| WMI filter | `WMI-<Target>-<Condition>` | `WMI-Windows11-Only` |
| Security group for filtering | `GG_GPO_<Purpose>_Apply` | `GG_GPO_Workstation_Baseline_Apply` |
| Delegation group | `GG_GPO_<Purpose>_Admins` | `GG_GPO_Workstation_GPO_Admins` |

# 00_Group_Policy_Index_Operator_Runbook_Skeleton
```powershell
# Run on a domain controller or domain-joined management workstation.
# Purpose: baseline discovery for the Group Policy suite.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$Domain = Get-ADDomain
$DomainFqdn = $Domain.DNSRoot
$DomainDN = $Domain.DistinguishedName

$PrimaryDC = "DC1"
$SecondaryDC = "DC2"

$RootOU = "OU=Corp,$DomainDN"
$WorkstationsOU = "OU=Workstations,$RootOU"
$ServersOU = "OU=Servers,$RootOU"
$UsersOU = "OU=Users,$RootOU"

$BasePath = "C:\GPOPrep"
$BackupPath = "$BasePath\Backup"
$ReportPath = "$BasePath\Reports"
$RsopPath = "$ReportPath\RSOP"

New-Item -ItemType Directory -Force -Path $BasePath,$BackupPath,$ReportPath,$RsopPath

# Domain and forest baseline.
Get-ADDomain | Tee-Object "$BasePath\domain.txt"
Get-ADForest | Tee-Object "$BasePath\forest.txt"

# Tool baseline.
Get-Module -ListAvailable GroupPolicy | Tee-Object "$BasePath\group-policy-module.txt"
Get-WindowsFeature GPMC | Tee-Object "$BasePath\gpmc-feature.txt"

# OU baseline.
Get-ADOrganizationalUnit -Filter * -SearchBase $RootOU |
  Select-Object Name,DistinguishedName |
  Tee-Object "$BasePath\ou-baseline.txt"

# SYSVOL and NETLOGON baseline.
Test-Path "\\$DomainFqdn\SYSVOL\$DomainFqdn\Policies" |
  Tee-Object "$BasePath\sysvol-path-test.txt"

Test-Path "\\$DomainFqdn\NETLOGON" |
  Tee-Object "$BasePath\netlogon-path-test.txt"

# AD and SYSVOL health.
repadmin /replsummary | Tee-Object "$BasePath\repadmin-replsummary.txt"
dcdiag /test:sysvolcheck /test:advertising | Tee-Object "$BasePath\dcdiag-sysvol-advertising.txt"

# GPO inventory.
Get-GPO -All |
  Sort-Object DisplayName |
  Select-Object DisplayName,Id,Owner,CreationTime,ModificationTime,GpoStatus |
  Export-Csv "$BasePath\gpo-inventory.csv" -NoTypeInformation

# Inheritance inventory for core OUs.
Get-GPInheritance -Target $RootOU | Out-File "$BasePath\inheritance-root-ou.txt"
Get-GPInheritance -Target $WorkstationsOU | Out-File "$BasePath\inheritance-workstations-ou.txt"
Get-GPInheritance -Target $ServersOU | Out-File "$BasePath\inheritance-servers-ou.txt"
Get-GPInheritance -Target $UsersOU | Out-File "$BasePath\inheritance-users-ou.txt"

# Back up current GPO state before changes.
Backup-GPO -All -Path $BackupPath

# Export current GPO reports.
Get-GPO -All | ForEach-Object {
    $SafeName = $_.DisplayName -replace '[\\/:*?"<>|]', '_'
    Get-GPOReport -Guid $_.Id -ReportType Html -Path "$ReportPath\$SafeName.html"
}

Write-Host "GPO baseline discovery complete."
Write-Host "Output path: $BasePath"