# 05_Configure_App_Deployment_Assignment_Detection_And_Remediation

## Objective

Configure Microsoft Intune application deployment using app assignments, install intent, detection rules, requirement rules, dependencies, supersedence, and remediation options. This workbook establishes a practical app deployment baseline for Windows pilot devices before expanding app delivery to production users and devices.

## Lab Context

This workbook assumes Intune enrollment, compliance, update rings, and configuration/security baselines are already in place. The goal is to deploy required and available applications in a controlled way, prove detection logic works, validate install status, and define remediation steps for failed app deployments.

This workbook focuses mainly on Windows app deployment because Windows app detection and remediation are the most operationally important pieces for endpoint administration.

## Prerequisites

| Requirement | Details |
|---|---|
| Admin role | Intune Administrator, Application Manager, Endpoint Security Manager, or Global Administrator |
| Portal access | Microsoft Intune admin center |
| Existing baseline | Tasks 01 through 04 should be complete |
| Managed devices | At least one enrolled Windows 10/11 test device |
| Device state | Microsoft Entra joined or hybrid joined and Intune managed |
| Pilot group | Dedicated Windows pilot device group |
| Pilot user group | Dedicated pilot user group for available apps |
| Licensing | Microsoft Intune licensing for managed devices/users |
| App source files | MSI, EXE, Microsoft Store app, Microsoft 365 Apps, or Win32 `.intunewin` package |
| Packaging tool | Microsoft Win32 Content Prep Tool for Win32 app packaging |
| Test device | Windows test device with Company Portal installed or available |
| Change control | Pilot deployment window and rollback plan |

## Naming / Variables

| Variable | Example Value | Purpose |
|---|---|---|
| Pilot device group | GRP-INTUNE-Windows-Pilot-Devices | Required app deployment target |
| Pilot user group | GRP-INTUNE-Pilot-Users | Available app deployment target |
| Exclusion group | GRP-INTUNE-Exclude-Critical-Devices | Devices excluded from app deployment |
| Required app name | APP-WIN-7Zip-Required-Pilot | Example required Win32 app |
| Available app name | APP-WIN-VLC-Available-Pilot | Example available Company Portal app |
| Microsoft 365 app name | APP-WIN-M365Apps-Pilot | Microsoft 365 Apps deployment |
| Store app name | APP-WIN-CompanyPortal-Pilot | Microsoft Store app deployment |
| Remediation script package | REM-WIN-AppInstall-HealthCheck-Pilot | Proactive remediation or script package |
| Test device | WIN11-INTUNE-01 | Pilot validation endpoint |
| App source folder | C:\IntuneApps\Source\7Zip | Source installer folder |
| Output folder | C:\IntuneApps\Output | IntuneWin output folder |
| Installer file | 7z2409-x64.exe | Example installer |
| IntuneWin file | 7z2409-x64.intunewin | Packaged Win32 app |
| Install command | 7z2409-x64.exe /S | Silent install command |
| Uninstall command | "%ProgramFiles%\7-Zip\Uninstall.exe" /S | Silent uninstall command |
| Detection path | C:\Program Files\7-Zip\7z.exe | File detection rule |
| Detection version | 24.09 | Example version check |

## Configuration Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|---:|---|---|---|---|---|
| 1 | Sign in to Microsoft Intune admin center | Admin workstation | Browser: `https://intune.microsoft.com` | N/A | Intune admin center opens successfully |
| 2 | Confirm pilot Windows device exists | Admin workstation | Devices > All devices | `Get-MgDeviceManagementManagedDevice -Filter "contains(deviceName,'WIN11-INTUNE-01')"` | Pilot Windows device appears as Intune managed |
| 3 | Confirm pilot device group exists | Admin workstation | Groups > search `GRP-INTUNE-Windows-Pilot-Devices` | `Get-MgGroup -Filter "displayName eq 'GRP-INTUNE-Windows-Pilot-Devices'"` | Pilot device group is available |
| 4 | Confirm pilot user group exists | Admin workstation | Groups > search `GRP-INTUNE-Pilot-Users` | `Get-MgGroup -Filter "displayName eq 'GRP-INTUNE-Pilot-Users'"` | Pilot user group is available |
| 5 | Confirm exclusion group exists if needed | Admin workstation | Groups > search `GRP-INTUNE-Exclude-Critical-Devices` | `Get-MgGroup -Filter "displayName eq 'GRP-INTUNE-Exclude-Critical-Devices'"` | Critical device exclusion group is available |
| 6 | Confirm test device is in pilot group | Admin workstation | Pilot device group > Members | `Get-MgGroupMember -GroupId <GroupId>` | Test device is targeted for required app deployment |
| 7 | Review existing Intune apps | Admin workstation | Apps > All apps | N/A | Existing app inventory is known |
| 8 | Review existing app assignments | Admin workstation | Apps > All apps > select existing apps > Properties > Assignments | N/A | Existing required, available, uninstall, and exclusion assignments are known |
| 9 | Create local app packaging folders | Packaging workstation | `mkdir C:\IntuneApps\Source\7Zip C:\IntuneApps\Output C:\IntuneApps\Tools` | `New-Item -ItemType Directory -Path "C:\IntuneApps\Source\7Zip","C:\IntuneApps\Output","C:\IntuneApps\Tools" -Force` | Packaging folders exist |
| 10 | Download or stage installer source | Packaging workstation | Copy installer into `C:\IntuneApps\Source\7Zip` | N/A | Installer file is staged |
| 11 | Confirm silent install command | Packaging workstation | Test command locally in elevated shell | `Start-Process ".\7z2409-x64.exe" -ArgumentList "/S" -Wait` | Application installs silently |
| 12 | Confirm silent uninstall command | Packaging workstation | Test uninstall command locally | `Start-Process "$env:ProgramFiles\7-Zip\Uninstall.exe" -ArgumentList "/S" -Wait` | Application uninstalls silently |
| 13 | Confirm detection artifact after install | Packaging workstation | `dir "C:\Program Files\7-Zip\7z.exe"` | `Test-Path "C:\Program Files\7-Zip\7z.exe"` | Detection file exists after install |
| 14 | Package Win32 app with IntuneWinAppUtil | Packaging workstation | `IntuneWinAppUtil.exe -c C:\IntuneApps\Source\7Zip -s 7z2409-x64.exe -o C:\IntuneApps\Output` | `Start-Process "C:\IntuneApps\Tools\IntuneWinAppUtil.exe" -ArgumentList "-c C:\IntuneApps\Source\7Zip -s 7z2409-x64.exe -o C:\IntuneApps\Output" -Wait` | `.intunewin` file is created |
| 15 | Open Intune app area | Admin workstation | Apps > All apps | N/A | App management blade opens |
| 16 | Add new Win32 app | Admin workstation | Apps > All apps > Add > Windows app (Win32) | N/A | Win32 app creation wizard opens |
| 17 | Upload `.intunewin` package | Admin workstation | App package file > Select app package file | N/A | Package metadata loads successfully |
| 18 | Configure app information | Admin workstation | Name: `APP-WIN-7Zip-Required-Pilot`; Publisher: 7-Zip; Category: Productivity or Utilities | N/A | App information is complete |
| 19 | Configure app logo if desired | Admin workstation | App information > Logo | N/A | Company Portal presentation is clean |
| 20 | Configure install command | Admin workstation | Program > Install command: `7z2409-x64.exe /S` | N/A | Silent install command is set |
| 21 | Configure uninstall command | Admin workstation | Program > Uninstall command: `"%ProgramFiles%\7-Zip\Uninstall.exe" /S` | N/A | Silent uninstall command is set |
| 22 | Configure install behavior | Admin workstation | Program > Install behavior: System | N/A | App installs in system context |
| 23 | Configure device restart behavior | Admin workstation | Program > Device restart behavior: No specific action or determine based on return codes | N/A | Restart behavior is controlled |
| 24 | Configure return codes | Admin workstation | Program > Return codes | N/A | Success, soft reboot, hard reboot, retry, and failed codes are defined |
| 25 | Configure operating system requirement | Admin workstation | Requirements > Operating system architecture: 64-bit; Minimum OS: Windows 10/11 baseline | N/A | App only targets supported Windows devices |
| 26 | Configure disk or memory requirements if needed | Admin workstation | Requirements | N/A | App requirements are defined if needed |
| 27 | Configure detection rule type | Admin workstation | Detection rules > Manually configure detection rules | N/A | Detection rule configuration opens |
| 28 | Configure file detection rule | Admin workstation | Rule type: File; Path: `C:\Program Files\7-Zip`; File or folder: `7z.exe`; Detection method: File or folder exists | N/A | Intune can detect successful install |
| 29 | Configure version detection if required | Admin workstation | Detection method: String/version comparison if using file version | N/A | Version-specific detection is available |
| 30 | Configure dependencies if required | Admin workstation | Dependencies > Add | N/A | Prerequisite app relationship is defined if needed |
| 31 | Configure supersedence if replacing older app | Admin workstation | Supersedence > Add | N/A | Old app can be replaced by new app if required |
| 32 | Assign app as Required to pilot devices | Admin workstation | Assignments > Required > Add group > `GRP-INTUNE-Windows-Pilot-Devices` | N/A | Pilot devices receive required install assignment |
| 33 | Add exclusion group if needed | Admin workstation | Required assignment > Exclude > `GRP-INTUNE-Exclude-Critical-Devices` | N/A | Critical devices are excluded |
| 34 | Configure assignment availability | Admin workstation | Assignment settings > Availability and deadline | N/A | Required install schedule is defined |
| 35 | Review and create Win32 app | Admin workstation | Review + create > Create | N/A | Required Win32 app is uploaded and created |
| 36 | Create available app assignment if needed | Admin workstation | Apps > All apps > select app > Properties > Assignments | N/A | Assignment editor opens |
| 37 | Assign app as Available for enrolled devices | Admin workstation | Available for enrolled devices > Add group > `GRP-INTUNE-Pilot-Users` | N/A | Pilot users can install from Company Portal |
| 38 | Save available assignment | Admin workstation | Review + save | N/A | Available assignment is saved |
| 39 | Add Microsoft Store app if required | Admin workstation | Apps > All apps > Add > Microsoft Store app (new) | N/A | Store app wizard opens |
| 40 | Search and select Company Portal or approved Store app | Admin workstation | Select app > Search | N/A | Store app metadata is loaded |
| 41 | Configure Store app information | Admin workstation | App information | N/A | Store app details are complete |
| 42 | Assign Store app | Admin workstation | Required or Available assignment to pilot group | N/A | Store app deployment is targeted |
| 43 | Create Microsoft 365 Apps deployment if required | Admin workstation | Apps > All apps > Add > Microsoft 365 Apps > Windows 10 and later | N/A | Microsoft 365 Apps wizard opens |
| 44 | Configure Microsoft 365 Apps suite | Admin workstation | Select Office apps, architecture, update channel, language | N/A | Microsoft 365 app package is configured |
| 45 | Configure Microsoft 365 Apps assignment | Admin workstation | Assignments > Required or Available > pilot group | N/A | M365 Apps deployment is targeted |
| 46 | Add MSI line-of-business app if required | Admin workstation | Apps > All apps > Add > Line-of-business app | N/A | MSI app wizard opens |
| 47 | Upload MSI package | Admin workstation | App package file > select MSI | N/A | MSI metadata loads |
| 48 | Configure MSI app information | Admin workstation | Name, publisher, category, command-line arguments if needed | N/A | MSI app metadata is complete |
| 49 | Assign MSI app to pilot group | Admin workstation | Assignments > Required or Available | N/A | MSI app deployment is targeted |
| 50 | Configure app categories | Admin workstation | Apps > App categories | N/A | Apps are organized for Company Portal users |
| 51 | Configure Company Portal branding if needed | Admin workstation | Tenant administration > Customization | N/A | Company Portal presentation is tenant-branded |
| 52 | Sync pilot Windows device | Windows test device | Settings > Accounts > Access work or school > Info > Sync | N/A | Device checks in with Intune |
| 53 | Trigger sync from Intune | Admin workstation | Devices > All devices > select `WIN11-INTUNE-01` > Sync | `Sync-MgDeviceManagementManagedDevice -ManagedDeviceId <ManagedDeviceId>` | Sync command is queued |
| 54 | Open Company Portal on test device | Windows test device | Start > Company Portal | N/A | Company Portal opens |
| 55 | Check available app visibility | Windows test device | Company Portal > Apps | N/A | Available app appears for pilot user |
| 56 | Install available app from Company Portal | Windows test device | Select app > Install | N/A | App install begins |
| 57 | Monitor required app install locally | Windows test device | Check installed programs and target path | `Test-Path "C:\Program Files\7-Zip\7z.exe"` | Required app installs |
| 58 | Review app install status in Intune | Admin workstation | Apps > All apps > select app > Device install status | N/A | Device status appears |
| 59 | Review user install status in Intune | Admin workstation | Apps > All apps > select app > User install status | N/A | User status appears |
| 60 | Review managed app reports | Admin workstation | Apps > Monitor > App install status | N/A | App deployment reporting is visible |
| 61 | Review Intune Management Extension logs | Windows test device | `C:\ProgramData\Microsoft\IntuneManagementExtension\Logs` | `Get-Content "C:\ProgramData\Microsoft\IntuneManagementExtension\Logs\IntuneManagementExtension.log" -Tail 100` | Win32 app processing logs are available |
| 62 | Create remediation detection script if required | Admin workstation | Devices > Scripts and remediations > Remediations > Create | N/A | Remediation wizard opens |
| 63 | Name remediation package | Admin workstation | Name: `REM-WIN-AppInstall-HealthCheck-Pilot` | N/A | Remediation package has clear name |
| 64 | Add detection script | Admin workstation | Detection script checks app presence/version | Example: `if (Test-Path "C:\Program Files\7-Zip\7z.exe") { exit 0 } else { exit 1 }` | Script detects app health |
| 65 | Add remediation script | Admin workstation | Remediation script triggers repair/install action or cleanup | Example: `Start-Process "C:\Program Files\CompanyPackage\Repair.cmd" -Wait` | Script can remediate app issue |
| 66 | Configure remediation settings | Admin workstation | Run script in 64-bit PowerShell: Yes; Run as logged-on credentials: No | N/A | Remediation runs in correct context |
| 67 | Assign remediation to pilot devices | Admin workstation | Assignments > Add group > `GRP-INTUNE-Windows-Pilot-Devices` | N/A | Pilot devices receive remediation |
| 68 | Configure remediation schedule | Admin workstation | Schedule: Once or recurring as required | N/A | Remediation runs at defined cadence |
| 69 | Review and create remediation | Admin workstation | Review + create > Create | N/A | Remediation package is created |
| 70 | Document app package source, commands, detection, and assignments | Admin workstation | Workbook notes / repo documentation | N/A | App deployment is reproducible |

## Detection Rule Examples

| Detection Type | Use Case | Example | Expected Result |
|---|---|---|---|
| File exists | Simple EXE install | `C:\Program Files\7-Zip\7z.exe` exists | Intune marks app installed when file exists |
| File version | Version-specific install | `7z.exe` version greater than or equal to `24.09` | Intune only marks app installed when correct version exists |
| Registry key exists | Apps that write uninstall keys | `HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\<AppKey>` exists | Intune detects app from registry |
| Registry value | Apps with DisplayVersion | `DisplayVersion` equals target version | Intune detects exact version |
| MSI product code | MSI apps | Product code from MSI metadata | Intune detects MSI install |
| Custom script | Complex detection | PowerShell exits `0` for installed, `1` for not installed | Intune uses script exit code for detection |

## Example Win32 Detection Script

| Purpose | Script |
|---|---|
| Detect 7-Zip install by file path and minimum version | `if (Test-Path "C:\Program Files\7-Zip\7z.exe") { $v = (Get-Item "C:\Program Files\7-Zip\7z.exe").VersionInfo.FileVersion; if ([version]$v -ge [version]"24.09") { exit 0 } else { exit 1 } } else { exit 1 }` |

## Example Win32 Install / Uninstall Commands

| App Type | Install Command | Uninstall Command |
|---|---|---|
| EXE silent install | `7z2409-x64.exe /S` | `"%ProgramFiles%\7-Zip\Uninstall.exe" /S` |
| MSI silent install | `msiexec /i app.msi /qn /norestart` | `msiexec /x {PRODUCT-CODE-GUID} /qn /norestart` |
| PowerShell wrapper | `powershell.exe -ExecutionPolicy Bypass -File .\Install.ps1` | `powershell.exe -ExecutionPolicy Bypass -File .\Uninstall.ps1` |
| CMD wrapper | `install.cmd` | `uninstall.cmd` |

## Validation Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|---:|---|---|---|---|---|
| 1 | Confirm required Win32 app exists | Admin workstation | Intune > Apps > All apps | N/A | `APP-WIN-7Zip-Required-Pilot` is present |
| 2 | Confirm app assignment | Admin workstation | App > Properties > Assignments | N/A | Pilot device group is assigned as Required |
| 3 | Confirm exclusions | Admin workstation | App > Properties > Assignments | N/A | Critical device exclusion group is excluded if required |
| 4 | Confirm install command | Admin workstation | App > Properties > Program | N/A | Silent install command is correct |
| 5 | Confirm uninstall command | Admin workstation | App > Properties > Program | N/A | Silent uninstall command is correct |
| 6 | Confirm detection rule | Admin workstation | App > Properties > Detection rules | N/A | Detection rule matches actual installed artifact |
| 7 | Confirm requirement rules | Admin workstation | App > Properties > Requirements | N/A | OS architecture and minimum OS are correct |
| 8 | Confirm dependencies if configured | Admin workstation | App > Properties > Dependencies | N/A | Dependency chain is correct |
| 9 | Confirm supersedence if configured | Admin workstation | App > Properties > Supersedence | N/A | Replacement behavior is correct |
| 10 | Confirm pilot device synced recently | Admin workstation | Devices > All devices > test device > Overview | N/A | Last check-in time is recent |
| 11 | Confirm Intune Management Extension installed | Windows test device | `dir "C:\Program Files (x86)\Microsoft Intune Management Extension"` | `Test-Path "C:\Program Files (x86)\Microsoft Intune Management Extension"` | IME is installed |
| 12 | Check IME service status | Windows test device | `sc query IntuneManagementExtension` | `Get-Service IntuneManagementExtension` | Intune Management Extension service is running |
| 13 | Confirm app installed locally | Windows test device | `dir "C:\Program Files\7-Zip\7z.exe"` | `Test-Path "C:\Program Files\7-Zip\7z.exe"` | App install artifact exists |
| 14 | Confirm app version locally | Windows test device | N/A | `(Get-Item "C:\Program Files\7-Zip\7z.exe").VersionInfo.FileVersion` | Installed version matches target |
| 15 | Check installed app registry | Windows test device | `reg query "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall" /s /f "7-Zip"` | `Get-ChildItem "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall" | Get-ItemProperty | Where-Object DisplayName -like "*7-Zip*"` | App appears in uninstall registry |
| 16 | Review IME logs | Windows test device | Open `C:\ProgramData\Microsoft\IntuneManagementExtension\Logs\IntuneManagementExtension.log` | `Get-Content "C:\ProgramData\Microsoft\IntuneManagementExtension\Logs\IntuneManagementExtension.log" -Tail 200` | App detection/install events are visible |
| 17 | Review app workload logs | Windows test device | Check IME log folder | `Select-String -Path "C:\ProgramData\Microsoft\IntuneManagementExtension\Logs\*.log" -Pattern "7-Zip","APP-WIN-7Zip"` | Relevant app processing entries are found |
| 18 | Confirm Intune device install status | Admin workstation | App > Device install status | N/A | Test device shows Installed or explainable status |
| 19 | Confirm Intune user install status | Admin workstation | App > User install status | N/A | Pilot user status appears if user-targeted |
| 20 | Confirm available app appears in Company Portal | Windows test device | Company Portal > Apps | N/A | Available app is visible |
| 21 | Install available app from Company Portal | Windows test device | Company Portal > select app > Install | N/A | User-initiated install succeeds |
| 22 | Confirm Store app install status if configured | Admin workstation | Store app > Device install status | N/A | Store app status is reported |
| 23 | Confirm Microsoft 365 Apps status if configured | Admin workstation | M365 Apps deployment > Device install status | N/A | M365 Apps status is reported |
| 24 | Confirm remediation package exists if configured | Admin workstation | Devices > Scripts and remediations > Remediations | N/A | Remediation package is present |
| 25 | Confirm remediation results | Admin workstation | Remediation package > Device status | N/A | Detection and remediation result appears |
| 26 | Confirm app deployment does not break compliance | Admin workstation | Devices > Compliance policies > Device status | N/A | Device remains compliant |
| 27 | Confirm Conditional Access still succeeds from test device | Windows test device | Sign in to Microsoft 365 app | N/A | Managed compliant device still has access |
| 28 | Record validation result | Admin workstation | Workbook notes | N/A | App deployment validation is documented |

## Troubleshooting Checks

| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| App stuck Waiting for install status | Device has not checked in or IME has not processed assignment | Device last check-in and IME logs | Sync device and restart Intune Management Extension |
| App not offered in Company Portal | Assigned to device instead of user, wrong group, or Company Portal cache | App assignment and user group membership | Assign Available to pilot user group |
| Required app does not install | Device not in required assignment group | App assignment and device group membership | Add device to pilot group and sync |
| Win32 app never starts install | Intune Management Extension missing or stopped | `Get-Service IntuneManagementExtension` | Restart service or re-enroll device if missing |
| Install fails with command error | Silent command is wrong | IME logs and local manual test | Correct install command and retest locally |
| App installs manually but fails in Intune | System context differs from user context | Install behavior and permissions | Use system-safe install command or change behavior intentionally |
| Detection fails after successful install | Detection rule does not match actual install path/version | Detection path, registry, file version | Correct detection rule |
| App repeatedly reinstalls | Detection rule always returns not detected | IME logs and detection script exit code | Fix detection logic to return success when installed |
| App shows Installed but app is broken | Detection checks only file presence, not health | Detection rule quality | Use version, registry, service, or script detection |
| MSI deployment fails | MSI parameters or product code incorrect | MSI logs and Intune app metadata | Use correct MSI command and product code |
| EXE deployment fails | EXE does not support chosen silent switch | Vendor documentation or local test | Use correct silent switch or wrapper script |
| App requires reboot but reboot behavior suppressed | Restart behavior or return codes wrong | Return code mapping | Map reboot codes correctly and define restart behavior |
| Dependency app not installed | Dependency not assigned or failed | Dependency status | Fix dependency app first |
| Supersedence removes wrong app | Supersedence relationship wrong | App > Supersedence | Correct old/new app relationship |
| Store app does not install | Store service blocked, unsupported app type, or licensing issue | Store app report and device logs | Validate Store app availability and network access |
| Microsoft 365 Apps install fails | Existing Office conflict, wrong architecture, or CDN/network issue | Office install logs and app status | Remove conflicting Office version or align architecture/channel |
| App assignment hits critical device | Missing exclusion group | App assignment exclusions | Add critical device exclusion group |
| Remediation script fails | Script context, execution policy, or 32-bit/64-bit mismatch | Remediation output and local script test | Run as system, enable 64-bit PowerShell, fix script |
| Detection script works locally but not in Intune | Script depends on user profile or mapped drive | Script path/context assumptions | Use system paths and machine-wide detection |
| Reports lag behind device state | Intune reporting delay | Local install state and IME logs | Wait for reporting refresh and force sync |

## Rollback / Cleanup

| Step | Task | Device | Command | PowerShell | Expected Result |
|---:|---|---|---|---|---|
| 1 | Remove required app assignment first | Admin workstation | Apps > select app > Properties > Assignments | N/A | Required install pressure is removed |
| 2 | Add uninstall assignment if app must be removed | Admin workstation | App > Assignments > Uninstall > Add group > pilot device group | N/A | App uninstall is targeted |
| 3 | Exclude devices that should retain app | Admin workstation | Uninstall assignment > Exclude group | N/A | Excluded devices keep app |
| 4 | Save assignment changes | Admin workstation | Review + save | N/A | Assignment update is committed |
| 5 | Sync pilot device | Windows test device | Settings > Accounts > Access work or school > Info > Sync | N/A | Device checks in after rollback assignment |
| 6 | Trigger sync from Intune | Admin workstation | Devices > All devices > select test device > Sync | `Sync-MgDeviceManagementManagedDevice -ManagedDeviceId <ManagedDeviceId>` | Sync command is queued |
| 7 | Confirm uninstall locally | Windows test device | `dir "C:\Program Files\7-Zip\7z.exe"` | `Test-Path "C:\Program Files\7-Zip\7z.exe"` | App file is removed or detection no longer succeeds |
| 8 | Review uninstall status | Admin workstation | App > Device install status | N/A | Device reflects uninstall or not installed state |
| 9 | Remove available assignment if needed | Admin workstation | App > Assignments > Available for enrolled devices | N/A | App no longer appears in Company Portal |
| 10 | Remove remediation assignment | Admin workstation | Devices > Scripts and remediations > Remediation > Assignments | N/A | Remediation no longer runs |
| 11 | Delete remediation package if no longer needed | Admin workstation | Remediations > select package > Delete | N/A | Test remediation is removed |
| 12 | Delete test app if no longer needed | Admin workstation | Apps > All apps > select app > Delete | N/A | Test app object is removed |
| 13 | Archive package source and commands | Packaging workstation | Copy package folder to archive | N/A | Source, output, and documentation are preserved |
| 14 | Document rollback state | Admin workstation | Workbook notes | N/A | Rollback is recorded |

## Notes

- Do not trust an app deployment until detection logic is proven.
- A bad detection rule is worse than no deployment. It causes false success or endless reinstall loops.
- Win32 apps depend on the Intune Management Extension. If IME is broken, Win32 app deployment is broken.
- Required apps are best targeted to device groups for baseline software.
- Available apps are usually best targeted to user groups so users can see them in Company Portal.
- Use system install behavior for machine-wide apps unless there is a clear user-context requirement.
- Package EXE installers as Win32 apps when you need custom commands, detection, dependencies, or supersedence.
- MSI line-of-business apps are simpler, but Win32 packaging usually gives better operational control.
- Do not mix line-of-business MSI and Win32 deployments for the same app on the same device.
- Dependencies should be used for true prerequisites, not as a lazy way to chain unrelated apps.
- Supersedence should be tested carefully. It can uninstall or replace applications at scale.
- Start remediation scripts in a small pilot group. Bad remediation scripts can break devices faster than bad app assignments.
- Store app and Microsoft 365 Apps deployments can have reporting delays. Validate locally and in Intune reports.

## Related Workbooks

| Workbook                                                                              | Relationship                                                                     |
| ------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------- |
| 01_Configure_Intune_Tenant_Enrollment_Baseline_And_Device_Platform_Restrictions.md    | Provides enrollment foundation before app deployment                             |
| 02_Configure_Device_Compliance_Conditional_Access_And_Enrollment_Status.md            | Confirms app deployment does not break compliance and access                     |
| 03_Configure_Windows_Update_Rings_Feature_Updates_And_Driver_Update_Policies.md       | Keeps Windows devices updated before app deployment                              |
| 04_Configure_Device_Configuration_Profiles_Security_Baselines_And_Settings_Catalog.md | Provides configuration and security baseline before app rollout                  |
| 06_Configure_Defender_For_Endpoint_Integration_And_Device_Risk_Signals.md             | Adds device risk and endpoint health signals that may affect app-managed devices |
| 07_Troubleshoot_Intune_Enrollment_Compliance_App_Deployment_And_Device_Sync_Issues.md | Troubleshoots app installation, detection, IME, sync, and remediation issues     |