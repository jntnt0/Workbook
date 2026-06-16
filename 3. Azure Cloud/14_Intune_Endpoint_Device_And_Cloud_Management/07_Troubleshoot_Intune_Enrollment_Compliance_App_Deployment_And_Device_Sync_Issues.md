# 07_Troubleshoot_Intune_Enrollment_Compliance_App_Deployment_And_Device_Sync_Issues

## Objective

Troubleshoot Microsoft Intune issues across device enrollment, device compliance, Conditional Access impact, app deployment, policy application, Microsoft Defender for Endpoint onboarding, and device sync. This workbook provides a structured workflow for isolating whether a problem is caused by licensing, enrollment scope, device platform restrictions, group assignment, policy conflict, client health, Intune Management Extension, Defender integration, or reporting delay.

## Lab Context

This workbook assumes the previous Intune workbooks have been configured:

- Tenant enrollment baseline and device platform restrictions
- Device compliance and Conditional Access
- Windows update rings, feature updates, and driver update policies
- Device configuration profiles, security baselines, and Settings catalog
- App deployment, assignment, detection, and remediation
- Defender for Endpoint integration and device risk signals

The goal is to troubleshoot from the outside in:

1. Confirm user, license, group, and tenant scope.
2. Confirm device enrollment and check-in state.
3. Confirm assignment targeting.
4. Confirm policy delivery.
5. Confirm local client state.
6. Confirm service-side reports.
7. Confirm rollback or remediation path.

## Prerequisites

| Requirement | Details |
|---|---|
| Admin role | Intune Administrator, Help Desk Operator, Endpoint Security Manager, Security Reader, Conditional Access Administrator, or Global Administrator depending issue type |
| Portal access | Microsoft Intune admin center, Microsoft Entra admin center, Microsoft Defender portal |
| Test user | Licensed pilot user affected by issue |
| Test device | Enrolled or enrollment-targeted Windows 10/11 device |
| Device access | Local admin access or remote support session preferred |
| Logs access | Ability to collect MDM diagnostics, Event Viewer logs, Intune Management Extension logs, and Defender logs |
| Existing groups | Pilot user group, pilot device group, CA exclusion group, and critical device exclusion group |
| Baseline workbooks | Tasks 01 through 06 should be available for comparison |
| Change control | Approval to sync, restart, retire, wipe, delete records, or modify assignments |

## Naming / Variables

| Variable | Example Value | Purpose |
|---|---|---|
| Affected user | user01@contoso.com | User reporting issue |
| Affected device | WIN11-INTUNE-01 | Device being investigated |
| Pilot user group | GRP-INTUNE-Pilot-Users | User targeting group |
| Pilot device group | GRP-INTUNE-Windows-Pilot-Devices | Device targeting group |
| Break-glass exclusion group | GRP-CA-Exclude-BreakGlass | Emergency access exclusion group |
| Critical device exclusion group | GRP-INTUNE-Exclude-Critical-Devices | Devices excluded from policy/app tests |
| Intune portal | `https://intune.microsoft.com` | Intune admin center |
| Entra portal | `https://entra.microsoft.com` | Microsoft Entra admin center |
| Defender portal | `https://security.microsoft.com` | Microsoft Defender portal |
| IME log path | `C:\ProgramData\Microsoft\IntuneManagementExtension\Logs` | Win32 app and remediation logs |
| MDM diagnostics output | `C:\Temp\MDMDiag.cab` | Local MDM diagnostic package |
| GPResult output | `C:\Temp\gpresult.html` | Group Policy conflict evidence |
| Event export folder | `C:\Temp\IntuneTroubleshooting` | Evidence collection folder |

## Triage Flow

| Stage | Question | If Yes | If No |
|---|---|---|---|
| Identity | Is the user licensed and in scope? | Continue to device checks | Fix license, group, or MDM scope |
| Enrollment | Is the device enrolled and Intune managed? | Continue to check-in | Fix enrollment, platform restriction, or device registration |
| Sync | Has the device checked in recently? | Continue to assignment | Force sync, check network, restart services |
| Assignment | Is the policy/app assigned to the correct user or device group? | Continue to local state | Fix group membership or assignment filter |
| Conflict | Is there a policy conflict? | Resolve duplicate setting source | Continue to local logs |
| Local client | Do local logs show policy/app processing? | Use log error to remediate | Check client health and management extension |
| Reporting | Does portal reporting match local state? | Close or escalate with evidence | Wait for reporting delay or force sync |
| Access | Is Conditional Access involved? | Review sign-in logs | Focus on Intune/device/app layer |

## Configuration / Troubleshooting Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|---:|---|---|---|---|---|
| 1 | Create local evidence folder | Windows test device | `mkdir C:\Temp\IntuneTroubleshooting` | `New-Item -ItemType Directory -Path "C:\Temp\IntuneTroubleshooting" -Force` | Troubleshooting folder exists |
| 2 | Record affected user and device | Admin workstation | Open ticket or notes | N/A | User, device name, timestamp, and issue type are documented |
| 3 | Sign in to Intune admin center | Admin workstation | Browser: `https://intune.microsoft.com` | N/A | Intune admin center opens |
| 4 | Sign in to Entra admin center | Admin workstation | Browser: `https://entra.microsoft.com` | N/A | Entra admin center opens |
| 5 | Search for affected user | Admin workstation | Entra > Users > search `user01@contoso.com` | `Get-MgUser -UserId user01@contoso.com` | User object exists |
| 6 | Confirm user account is enabled | Admin workstation | User > Properties | `(Get-MgUser -UserId user01@contoso.com -Property AccountEnabled).AccountEnabled` | User account is enabled |
| 7 | Confirm user has Intune-capable license | Admin workstation | Microsoft 365 admin center > User > Licenses and apps | `Get-MgUserLicenseDetail -UserId user01@contoso.com` | Required Intune license exists |
| 8 | Confirm user is in pilot scope | Admin workstation | Entra > Groups > `GRP-INTUNE-Pilot-Users` > Members | `Get-MgUserMemberOf -UserId user01@contoso.com` | User is in expected targeting group |
| 9 | Confirm MDM user scope | Admin workstation | Entra > Identity > Mobility > Microsoft Intune | N/A | User or group is included in MDM scope |
| 10 | Confirm MDM URLs are populated | Admin workstation | Entra > Mobility > Microsoft Intune | N/A | MDM discovery and compliance URLs are populated |
| 11 | Search for device in Intune | Admin workstation | Intune > Devices > All devices > search `WIN11-INTUNE-01` | `Get-MgDeviceManagementManagedDevice -Filter "contains(deviceName,'WIN11-INTUNE-01')"` | Device record exists in Intune |
| 12 | Search for device in Entra | Admin workstation | Entra > Devices > All devices > search device name | `Get-MgDevice -Filter "displayName eq 'WIN11-INTUNE-01'"` | Entra device object exists |
| 13 | Compare Intune and Entra device state | Admin workstation | Review Intune device ID and Entra device object | N/A | Device records correspond to the same endpoint |
| 14 | Confirm device ownership | Admin workstation | Intune device > Properties | N/A | Ownership is expected: Corporate or Personal |
| 15 | Confirm primary user | Admin workstation | Intune device > Properties > Primary user | N/A | Primary user matches expected user |
| 16 | Confirm device last check-in | Admin workstation | Intune device > Overview | N/A | Last check-in is recent or stale state is identified |
| 17 | Confirm device compliance state | Admin workstation | Intune device > Overview and Device compliance | N/A | Compliance state is visible |
| 18 | Confirm device group membership | Admin workstation | Entra device > Groups or target group > Members | `Get-MgDeviceMemberOf -DeviceId <DeviceObjectId>` | Device is in expected pilot device group |
| 19 | Confirm assignment filters if used | Admin workstation | Intune > Tenant administration > Filters | N/A | Filter rules include or exclude device as intended |
| 20 | Trigger device sync from Intune | Admin workstation | Intune device > Sync | `Sync-MgDeviceManagementManagedDevice -ManagedDeviceId <ManagedDeviceId>` | Sync command is queued |
| 21 | Trigger manual sync on client | Windows test device | Settings > Accounts > Access work or school > Info > Sync | N/A | Client starts MDM sync |
| 22 | Check work or school account state | Windows test device | Settings > Accounts > Access work or school | N/A | Connected work account appears |
| 23 | Check device registration state | Windows test device | `dsregcmd /status` | `dsregcmd /status` | Join type, MDM URL, tenant ID, and SSO state are visible |
| 24 | Save device registration output | Windows test device | `dsregcmd /status > C:\Temp\IntuneTroubleshooting\dsregcmd_status.txt` | `dsregcmd /status | Out-File C:\Temp\IntuneTroubleshooting\dsregcmd_status.txt` | Registration evidence is saved |
| 25 | Create MDM diagnostics report | Windows test device | Settings > Accounts > Access work or school > Info > Create report | N/A | HTML MDM diagnostic report is generated |
| 26 | Export MDM diagnostics CAB | Windows test device | `mdmdiagnosticstool.exe -area DeviceEnrollment;DeviceProvisioning;Autopilot;Policy -cab C:\Temp\IntuneTroubleshooting\MDMDiag.cab` | `Start-Process mdmdiagnosticstool.exe -ArgumentList "-area DeviceEnrollment;DeviceProvisioning;Autopilot;Policy -cab C:\Temp\IntuneTroubleshooting\MDMDiag.cab" -Wait` | MDM diagnostics CAB is created |
| 27 | Check MDM event logs | Windows test device | Event Viewer > Applications and Services Logs > Microsoft > Windows > DeviceManagement-Enterprise-Diagnostics-Provider > Admin | `Get-WinEvent -LogName "Microsoft-Windows-DeviceManagement-Enterprise-Diagnostics-Provider/Admin" -MaxEvents 100` | Recent MDM events are visible |
| 28 | Export MDM Admin event log | Windows test device | `wevtutil epl Microsoft-Windows-DeviceManagement-Enterprise-Diagnostics-Provider/Admin C:\Temp\IntuneTroubleshooting\MDM-Admin.evtx` | `wevtutil epl Microsoft-Windows-DeviceManagement-Enterprise-Diagnostics-Provider/Admin C:\Temp\IntuneTroubleshooting\MDM-Admin.evtx` | MDM Admin log is exported |
| 29 | Check MDM Operational event log | Windows test device | Event Viewer > DeviceManagement-Enterprise-Diagnostics-Provider > Operational | `Get-WinEvent -LogName "Microsoft-Windows-DeviceManagement-Enterprise-Diagnostics-Provider/Operational" -MaxEvents 100` | Operational MDM events are visible |
| 30 | Check PolicyManager registry | Windows test device | `reg query HKLM\SOFTWARE\Microsoft\PolicyManager\current\device /s` | `Get-ChildItem "HKLM:\SOFTWARE\Microsoft\PolicyManager\current\device" -Recurse -ErrorAction SilentlyContinue` | Applied MDM policy values are visible |
| 31 | Check enrollment registry | Windows test device | `reg query HKLM\SOFTWARE\Microsoft\Enrollments /s` | `Get-ChildItem "HKLM:\SOFTWARE\Microsoft\Enrollments" -Recurse -ErrorAction SilentlyContinue` | Enrollment entries exist |
| 32 | Check scheduled tasks for MDM enrollment | Windows test device | Task Scheduler > Microsoft > Windows > EnterpriseMgmt | `Get-ScheduledTask -TaskPath "\Microsoft\Windows\EnterpriseMgmt\*" -ErrorAction SilentlyContinue` | MDM scheduled tasks exist |
| 33 | Force scheduled MDM task if appropriate | Windows test device | Run EnterpriseMgmt schedule task from Task Scheduler | N/A | MDM sync task runs |
| 34 | Check for Group Policy conflicts | Windows test device | `gpresult /h C:\Temp\IntuneTroubleshooting\gpresult.html` | `gpresult /h C:\Temp\IntuneTroubleshooting\gpresult.html` | GPO report is created |
| 35 | Review GPO report for MDM conflicts | Windows test device | Open `gpresult.html` | N/A | Conflicting GPO settings are identified |
| 36 | Check ConfigMgr co-management state if applicable | Windows test device | Control Panel > Configuration Manager or Company portal workload status | N/A | Co-management conflict is identified if present |
| 37 | Review Intune device configuration status | Admin workstation | Intune device > Device configuration | N/A | Configuration profile results are visible |
| 38 | Review per-setting status for failed profile | Admin workstation | Configuration profile > Device status > Per-setting status | N/A | Failed or conflicting setting is identified |
| 39 | Review Endpoint security policy status | Admin workstation | Endpoint security > policy > Device status | N/A | Security policy status is visible |
| 40 | Review Security baseline status | Admin workstation | Endpoint security > Security baselines > profile > Device status | N/A | Baseline setting status is visible |
| 41 | Review compliance policy status | Admin workstation | Devices > Compliance policies > policy > Device status | N/A | Compliance result is visible |
| 42 | Review noncompliant setting details | Admin workstation | Device > Device compliance > select policy | N/A | Specific noncompliant setting is identified |
| 43 | Review compliance policy assignment | Admin workstation | Compliance policy > Properties > Assignments | N/A | User or device is correctly targeted |
| 44 | Review compliance policy actions | Admin workstation | Compliance policy > Actions for noncompliance | N/A | Grace period and actions are understood |
| 45 | Review tenant compliance settings | Admin workstation | Devices > Compliance policies > Compliance policy settings | N/A | Devices with no policy behavior is known |
| 46 | Review Conditional Access sign-in impact | Admin workstation | Entra > Monitoring > Sign-in logs > select affected sign-in > Conditional Access | N/A | CA policy causing block is identified |
| 47 | Confirm CA policy excludes break-glass accounts | Admin workstation | CA policy > Users > Exclude | N/A | Break-glass group is excluded |
| 48 | Set problematic CA policy to Report-only if emergency rollback is approved | Admin workstation | CA policy > Enable policy > Report-only | N/A | Access enforcement is paused |
| 49 | Review enrollment failures | Admin workstation | Intune > Devices > Monitor > Enrollment failures | N/A | Enrollment error codes and affected users are visible |
| 50 | Review device platform restrictions | Admin workstation | Intune > Devices > Enrollment > Device platform restrictions | N/A | Platform is allowed or blocked intentionally |
| 51 | Review enrollment device limit restrictions | Admin workstation | Devices > Enrollment > Enrollment device limit restrictions | N/A | User has not exceeded allowed device count |
| 52 | Review enrollment status page if Autopilot is involved | Admin workstation | Devices > Enrollment > Enrollment Status Page | N/A | ESP policy state is visible |
| 53 | Review Windows Autopilot device record if involved | Admin workstation | Devices > Enrollment > Windows Autopilot devices | N/A | Autopilot record exists and is assigned |
| 54 | Check Autopilot diagnostics locally if involved | Windows test device | `mdmdiagnosticstool.exe -area Autopilot -cab C:\Temp\IntuneTroubleshooting\Autopilot.cab` | `Start-Process mdmdiagnosticstool.exe -ArgumentList "-area Autopilot -cab C:\Temp\IntuneTroubleshooting\Autopilot.cab" -Wait` | Autopilot diagnostics are collected |
| 55 | Review app install status | Admin workstation | Intune > Apps > All apps > select app > Device install status | N/A | App deployment status is visible |
| 56 | Review user app install status | Admin workstation | App > User install status | N/A | User-targeted status is visible |
| 57 | Confirm app assignment | Admin workstation | App > Properties > Assignments | N/A | Required, Available, or Uninstall assignment is correct |
| 58 | Confirm app assignment intent | Admin workstation | App > Assignments | N/A | Install or uninstall intent matches goal |
| 59 | Confirm app requirement rules | Admin workstation | App > Properties > Requirements | N/A | Device meets OS, architecture, disk, and other requirements |
| 60 | Confirm app detection rules | Admin workstation | App > Properties > Detection rules | N/A | Detection rule matches installed artifact |
| 61 | Check Intune Management Extension path | Windows test device | `dir "C:\Program Files (x86)\Microsoft Intune Management Extension"` | `Test-Path "C:\Program Files (x86)\Microsoft Intune Management Extension"` | IME folder exists |
| 62 | Check IME service status | Windows test device | `sc query IntuneManagementExtension` | `Get-Service IntuneManagementExtension -ErrorAction SilentlyContinue` | IME service is running |
| 63 | Restart IME service if appropriate | Windows test device | `net stop IntuneManagementExtension && net start IntuneManagementExtension` | `Restart-Service IntuneManagementExtension -Force` | IME restarts |
| 64 | Review IME logs | Windows test device | Open `C:\ProgramData\Microsoft\IntuneManagementExtension\Logs\IntuneManagementExtension.log` | `Get-Content "C:\ProgramData\Microsoft\IntuneManagementExtension\Logs\IntuneManagementExtension.log" -Tail 200` | Win32 app processing is visible |
| 65 | Search IME logs for app name | Windows test device | `findstr /i "7-Zip APP-WIN" C:\ProgramData\Microsoft\IntuneManagementExtension\Logs\*.log` | `Select-String -Path "C:\ProgramData\Microsoft\IntuneManagementExtension\Logs\*.log" -Pattern "7-Zip","APP-WIN"` | Relevant app log lines are found |
| 66 | Check local app detection path | Windows test device | `dir "C:\Program Files\7-Zip\7z.exe"` | `Test-Path "C:\Program Files\7-Zip\7z.exe"` | Detection artifact state is known |
| 67 | Check installed app registry | Windows test device | `reg query "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall" /s /f "7-Zip"` | `Get-ChildItem "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall" | Get-ItemProperty | Where-Object DisplayName -like "*7-Zip*"` | App registry evidence is found or absent |
| 68 | Test install command manually | Windows test device | Run exact Intune install command from elevated prompt | `Start-Process ".\installer.exe" -ArgumentList "<silent switches>" -Wait` | Silent command succeeds or returns useful error |
| 69 | Test uninstall command manually | Windows test device | Run exact Intune uninstall command from elevated prompt | `Start-Process ".\uninstall.exe" -ArgumentList "<silent switches>" -Wait` | Silent uninstall succeeds or returns useful error |
| 70 | Review remediation status | Admin workstation | Devices > Scripts and remediations > Remediations > select package > Device status | N/A | Detection/remediation results are visible |
| 71 | Review remediation local logs | Windows test device | IME logs folder | `Select-String -Path "C:\ProgramData\Microsoft\IntuneManagementExtension\Logs\*.log" -Pattern "remediation","detect","script"` | Script activity is visible |
| 72 | Review Windows Update policy status | Admin workstation | Devices > Windows > Update rings > policy > Device status | N/A | Update ring status is visible |
| 73 | Check Windows Update event logs | Windows test device | Event Viewer > Microsoft > Windows > WindowsUpdateClient > Operational | `Get-WinEvent -LogName "Microsoft-Windows-WindowsUpdateClient/Operational" -MaxEvents 100` | Update scan/install events are visible |
| 74 | Generate Windows Update log | Windows test device | `Get-WindowsUpdateLog` | `Get-WindowsUpdateLog` | WindowsUpdate.log is generated |
| 75 | Check Windows Update service | Windows test device | `sc query wuauserv` | `Get-Service wuauserv` | Windows Update service state is known |
| 76 | Check Update Orchestrator service | Windows test device | `sc query UsoSvc` | `Get-Service UsoSvc` | Update Orchestrator service state is known |
| 77 | Start update scan if appropriate | Windows test device | `UsoClient StartScan` | `Start-Process UsoClient.exe -ArgumentList "StartScan"` | Update scan is triggered |
| 78 | Review Defender onboarding status | Admin workstation | Intune > Endpoint security > Endpoint detection and response > policy > Device status | N/A | MDE onboarding policy status is visible |
| 79 | Check Defender services | Windows test device | `sc query Sense` | `Get-Service Sense,WinDefend,WdNisSvc -ErrorAction SilentlyContinue` | Defender services state is known |
| 80 | Check Defender status | Windows test device | N/A | `Get-MpComputerStatus` | Defender health is visible |
| 81 | Check MDE onboarding registry | Windows test device | `reg query "HKLM\SOFTWARE\Microsoft\Windows Advanced Threat Protection\Status"` | `Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows Advanced Threat Protection\Status" -ErrorAction SilentlyContinue` | MDE onboarding status values are visible |
| 82 | Confirm device in Defender portal | Admin workstation | Defender portal > Assets > Devices > search device | N/A | Device appears or missing state is confirmed |
| 83 | Review Defender device risk | Admin workstation | Defender portal > Device page > Risk level | N/A | Device risk level is visible |
| 84 | Review MDE risk compliance impact | Admin workstation | Intune > Device > Device compliance | N/A | Risk-based compliance result is visible |
| 85 | Collect Defender operational events | Windows test device | Event Viewer > Microsoft > Windows > Windows Defender > Operational | `Get-WinEvent -LogName "Microsoft-Windows-Windows Defender/Operational" -MaxEvents 100` | Defender events are visible |
| 86 | Export Defender event log | Windows test device | `wevtutil epl "Microsoft-Windows-Windows Defender/Operational" C:\Temp\IntuneTroubleshooting\Defender-Operational.evtx` | `wevtutil epl "Microsoft-Windows-Windows Defender/Operational" C:\Temp\IntuneTroubleshooting\Defender-Operational.evtx` | Defender log is exported |
| 87 | Capture installed apps list | Windows test device | `wmic product get name,version` | `Get-ItemProperty HKLM:\Software\Microsoft\Windows\CurrentVersion\Uninstall\*,HKLM:\Software\WOW6432Node\Microsoft\Windows\CurrentVersion\Uninstall\* | Select-Object DisplayName,DisplayVersion,Publisher,InstallDate | Sort-Object DisplayName | Export-Csv C:\Temp\IntuneTroubleshooting\InstalledApps.csv -NoTypeInformation` | Installed app inventory is exported |
| 88 | Capture services list | Windows test device | `sc query type= service state= all > C:\Temp\IntuneTroubleshooting\Services.txt` | `Get-Service | Sort-Object Name | Export-Csv C:\Temp\IntuneTroubleshooting\Services.csv -NoTypeInformation` | Service inventory is exported |
| 89 | Capture device info | Windows test device | `systeminfo > C:\Temp\IntuneTroubleshooting\SystemInfo.txt` | `Get-ComputerInfo | Out-File C:\Temp\IntuneTroubleshooting\ComputerInfo.txt` | Device inventory evidence is exported |
| 90 | Compress evidence folder | Windows test device | N/A | `Compress-Archive -Path C:\Temp\IntuneTroubleshooting\* -DestinationPath C:\Temp\IntuneTroubleshooting.zip -Force` | Evidence package is created |
| 91 | Decide remediation path | Admin workstation | Review findings | N/A | Root cause category is identified |
| 92 | Apply least disruptive remediation first | Admin workstation / Windows device | Sync, restart service, fix assignment, fix detection, or adjust policy | Depends on issue | Issue is corrected without unnecessary re-enrollment |
| 93 | Escalate with evidence if unresolved | Admin workstation | Attach exported logs and screenshots | N/A | Support package is ready |

## Enrollment Troubleshooting Matrix

| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| User cannot enroll device | User not in MDM scope | Entra > Mobility > Microsoft Intune | Add user/group to MDM scope |
| User cannot enroll device | User lacks Intune license | User > Licenses and apps | Assign Intune-capable license |
| Enrollment blocked by platform | Device platform restriction blocks OS or ownership | Intune > Devices > Enrollment > Device platform restrictions | Adjust platform restriction or use correct ownership path |
| Enrollment blocked after several devices | Device limit reached | Enrollment device limit restrictions and user device list | Retire stale devices or increase pilot limit |
| Device registers in Entra but not Intune | MDM auto-enrollment failed | `dsregcmd /status`, MDM event logs | Fix MDM scope/license and re-trigger enrollment |
| Duplicate device records | Re-enrollment, reset, Autopilot reuse, or stale record | Intune and Entra device lists | Clean stale records carefully |
| Autopilot hangs at ESP | App, policy, or account setup timeout | ESP reports and MDM diagnostics | Identify blocking app/policy and adjust ESP |
| Hybrid join enrollment fails | AD Connect, SCP, line-of-sight, or DNS issue | `dsregcmd /status` and event logs | Fix hybrid join prerequisites |
| Personal device blocked | Personal ownership not allowed | Platform restriction policy | Allow personal devices only if design permits |
| Device appears with wrong primary user | Shared device, enrollment account, or reassignment issue | Device properties | Change primary user if appropriate |

## Compliance / Conditional Access Troubleshooting Matrix

| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| Device is Not compliant | Required setting failed | Device > Device compliance | Remediate specific failed setting |
| Device is Not evaluated | Device has not checked in or policy not assigned | Last check-in and assignment | Sync device and fix assignment |
| Device has no compliance policy | Assignment gap or filter exclusion | Compliance policy assignments and filters | Assign correct group or adjust filter |
| User blocked by CA | Device not compliant or CA policy too broad | Entra sign-in logs > Conditional Access | Fix compliance or narrow CA policy |
| Break-glass blocked | Exclusion missing | CA policy exclusions | Add break-glass group to exclusion |
| Compliant in Intune but blocked by CA | Compliance claim not refreshed or device sign-in state issue | Sign-in logs and `dsregcmd /status` | Re-authenticate, sync device, wait for propagation |
| MDE risk causes noncompliance | Defender machine risk above threshold | Defender device page and Intune compliance details | Investigate and remediate Defender alerts |
| Compliance changed after baseline | New security setting failed | Compliance and configuration reports | Fix policy conflict or remediate device |
| Device marked noncompliant after grace period | Action for noncompliance elapsed | Compliance policy actions | Remediate setting or adjust grace period |
| CA report-only looks wrong | Wrong app/user/platform condition | CA policy conditions and sign-in logs | Correct policy scope |

## App Deployment Troubleshooting Matrix

| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| Required app never installs | Device not targeted | App assignment and group membership | Add device to required assignment group |
| Available app not visible | User not targeted or Company Portal cache | App assignment and user group | Assign available app to user group |
| Win32 app fails immediately | Bad install command | IME logs and local command test | Correct silent install command |
| App installs but Intune says failed | Bad detection rule | Detection rule and local artifact | Fix file, registry, MSI, or script detection |
| App reinstalls repeatedly | Detection always returns false | IME logs and detection script | Fix detection logic |
| App says installed but app is broken | Detection too weak | Detection artifact | Use stronger version or health detection |
| Dependency not installing | Dependency app failed or not applicable | Dependency status | Fix dependency first |
| Supersedence removes wrong app | Incorrect supersedence chain | App supersedence settings | Correct old/new relationship |
| Remediation script fails | Wrong script context or 32-bit PowerShell issue | Remediation output and IME logs | Use system context and 64-bit PowerShell |
| Store app fails | Store blocked or app unavailable | Store app report and network | Validate Store access and app availability |
| Microsoft 365 Apps fails | Office conflict, architecture mismatch, or CDN issue | Office logs and app report | Align architecture/channel and remove conflicting Office |

## Policy / Configuration Troubleshooting Matrix

| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| Profile shows Conflict | Same setting configured in multiple policies | Per-setting status | Choose one policy source |
| Profile shows Error | Unsupported CSP, OS version, or bad value | MDM event logs and per-setting status | Remove or correct setting |
| Profile stuck Pending | Device offline or not synced | Last check-in | Sync device and confirm network |
| BitLocker fails | TPM, recovery escrow, or silent enablement issue | BitLocker report and local status | Fix hardware/readiness and escrow |
| Firewall breaks app | Rule missing | Firewall logs and connectivity test | Add scoped allow rule |
| ASR breaks app | Rule in Block without testing | Defender operational log | Move to Audit or add exclusion |
| GPO overrides MDM | Domain GPO conflict | `gpresult.html` | Remove/reconcile GPO |
| ConfigMgr overrides Intune | Co-management workload not moved | ConfigMgr workload settings | Move workload or manage in ConfigMgr |
| Setting not applicable | Wrong OS edition/build | Device OS and policy support | Adjust targeting or OS minimum |
| Reports lag | Intune reporting delay | Local state and check-in time | Wait and force sync |

## Device Sync Troubleshooting Matrix

| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| Sync button does nothing | Device offline or stale object | Device last check-in | Bring device online or validate enrollment |
| Manual sync fails | MDM enrollment broken | MDM event logs and enrollment registry | Re-enroll only after lighter fixes fail |
| Policies delayed | Normal MDM polling interval | Last check-in | Force sync and wait |
| IME not processing | IME service stopped | `Get-Service IntuneManagementExtension` | Restart IME service |
| IME missing | Device not managed for Win32 app support | IME folder and service | Confirm Intune management and app assignment |
| Device has wrong time | Time skew breaks auth | Windows time settings | Sync time and retry |
| Network blocks endpoints | Proxy/firewall issue | Client logs and network tests | Allow Microsoft Intune/MDE endpoints |
| Stale device record | Device reset or duplicate object | Intune/Entra records | Clean stale record carefully |
| User changed password/MFA | Token issue | `dsregcmd /status` SSO state | Re-authenticate work account |
| Hybrid device lacks line-of-sight | Cannot complete hybrid registration | `dsregcmd /status` | Restore domain/DC connectivity |

## Validation Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|---:|---|---|---|---|---|
| 1 | Confirm root cause category | Admin workstation | Review troubleshooting notes | N/A | Issue is classified as identity, enrollment, assignment, policy, app, sync, CA, or reporting |
| 2 | Confirm user license is fixed | Admin workstation | User > Licenses and apps | `Get-MgUserLicenseDetail -UserId user01@contoso.com` | Intune-capable license exists |
| 3 | Confirm user group membership is fixed | Admin workstation | Group > Members | `Get-MgUserMemberOf -UserId user01@contoso.com` | User is in required groups |
| 4 | Confirm device group membership is fixed | Admin workstation | Group > Members | `Get-MgDeviceMemberOf -DeviceId <DeviceObjectId>` | Device is in required groups |
| 5 | Confirm device check-in is recent | Admin workstation | Intune device > Overview | N/A | Last check-in is recent |
| 6 | Confirm local sync succeeds | Windows test device | Settings > Accounts > Access work or school > Info > Sync | N/A | Sync completes without visible error |
| 7 | Confirm `dsregcmd` state is healthy | Windows test device | `dsregcmd /status` | `dsregcmd /status` | Join, tenant, and MDM state are expected |
| 8 | Confirm MDM events no longer show active failure | Windows test device | Event Viewer MDM Admin log | `Get-WinEvent -LogName "Microsoft-Windows-DeviceManagement-Enterprise-Diagnostics-Provider/Admin" -MaxEvents 50` | No new blocking errors appear |
| 9 | Confirm configuration profile status | Admin workstation | Device > Device configuration | N/A | Target profile shows Succeeded or expected state |
| 10 | Confirm compliance state | Admin workstation | Device > Device compliance | N/A | Device is Compliant or has known remaining issue |
| 11 | Confirm Conditional Access result | Admin workstation | Entra > Sign-in logs | N/A | CA policy result matches intended design |
| 12 | Confirm app status | Admin workstation | App > Device install status | N/A | App shows Installed, Not installed, or explainable state |
| 13 | Confirm app locally | Windows test device | Check install path | `Test-Path "C:\Program Files\7-Zip\7z.exe"` | Local app state matches Intune report |
| 14 | Confirm IME logs show successful processing | Windows test device | IME log folder | `Select-String -Path "C:\ProgramData\Microsoft\IntuneManagementExtension\Logs\*.log" -Pattern "success","installed"` | App processing success is visible |
| 15 | Confirm Defender onboarding if relevant | Admin workstation | Defender portal > Device inventory | N/A | Device appears with healthy sensor |
| 16 | Confirm update policy if relevant | Admin workstation | Update ring or feature update reports | N/A | Device reports expected update state |
| 17 | Confirm no broad production impact | Admin workstation | Policy/app assignment review | N/A | Only intended pilot/prod groups are targeted |
| 18 | Confirm rollback not needed | Admin workstation | Review user impact | N/A | Service is restored |
| 19 | Attach evidence to ticket | Admin workstation | Upload logs/screenshots | N/A | Ticket has proof of root cause and fix |
| 20 | Update workbook notes | Admin workstation | Repo or Obsidian notes | N/A | Known issue and fix are documented |

## Rollback / Cleanup

| Step | Task | Device | Command | PowerShell | Expected Result |
|---:|---|---|---|---|---|
| 1 | Pause CA enforcement if blocking users | Admin workstation | Entra > Conditional Access > policy > Report-only or Off | N/A | Access block is stopped |
| 2 | Remove bad app assignment | Admin workstation | Intune > Apps > App > Assignments | N/A | App no longer installs or uninstalls unintentionally |
| 3 | Add app uninstall assignment if cleanup required | Admin workstation | App > Assignments > Uninstall | N/A | App removal is targeted |
| 4 | Remove problematic configuration profile assignment | Admin workstation | Configuration profile > Assignments | N/A | Bad policy stops applying |
| 5 | Remove problematic security baseline assignment | Admin workstation | Security baseline > Assignments | N/A | Baseline stops applying |
| 6 | Move ASR from Block to Audit if app impact occurs | Admin workstation | Endpoint security > ASR policy | N/A | ASR impact is reduced |
| 7 | Remove update expedite policy if too aggressive | Admin workstation | Devices > Windows > Quality updates > policy > Assignments | N/A | Forced update pressure is removed |
| 8 | Remove MDE risk compliance assignment if access issue occurs | Admin workstation | Compliance policy > Assignments | N/A | Risk-based noncompliance stops affecting pilot |
| 9 | Sync affected device after rollback | Windows test device | Settings > Accounts > Access work or school > Info > Sync | `Sync-MgDeviceManagementManagedDevice -ManagedDeviceId <ManagedDeviceId>` | Device receives rollback |
| 10 | Restart IME after app rollback if required | Windows test device | `net stop IntuneManagementExtension && net start IntuneManagementExtension` | `Restart-Service IntuneManagementExtension -Force` | IME reprocesses assignments |
| 11 | Restart device if policy requires reboot | Windows test device | `shutdown /r /t 0` | `Restart-Computer` | Device restarts |
| 12 | Retire device only when appropriate | Admin workstation | Intune > Device > Retire | N/A | Company data and management are removed without wiping personal data |
| 13 | Delete stale Intune device record only after confirming duplicate/stale state | Admin workstation | Intune > Device > Delete | N/A | Stale record is removed |
| 14 | Delete stale Entra device record only after confirming it is safe | Admin workstation | Entra > Devices > Delete | `Remove-MgDevice -DeviceId <DeviceObjectId>` | Stale Entra object is removed |
| 15 | Re-enroll only after lighter fixes fail | Windows test device | Disconnect work account, reboot, enroll again | N/A | Device is cleanly re-enrolled |
| 16 | Remove troubleshooting files if no longer needed | Windows test device | `rmdir /s /q C:\Temp\IntuneTroubleshooting` | `Remove-Item C:\Temp\IntuneTroubleshooting -Recurse -Force` | Local evidence folder is cleaned up |
| 17 | Preserve evidence ZIP if issue was escalated | Windows test device | Copy `C:\Temp\IntuneTroubleshooting.zip` to approved location | N/A | Evidence is retained for ticket |
| 18 | Document rollback state | Admin workstation | Workbook notes | N/A | Rollback is recorded |

## Escalation Evidence Checklist

| Evidence | Location | Purpose |
|---|---|---|
| User UPN and license screenshot | Microsoft 365 admin center or Entra | Proves user scope and licensing |
| User group membership | Entra group membership | Proves assignment targeting |
| Device Intune overview screenshot | Intune device page | Proves management/check-in state |
| Device Entra object screenshot | Entra device page | Proves registration/join state |
| `dsregcmd /status` output | `C:\Temp\IntuneTroubleshooting\dsregcmd_status.txt` | Proves join, tenant, MDM, and SSO state |
| MDM diagnostics CAB | `C:\Temp\IntuneTroubleshooting\MDMDiag.cab` | Captures MDM client diagnostics |
| MDM Admin event log | `C:\Temp\IntuneTroubleshooting\MDM-Admin.evtx` | Captures MDM policy/enrollment errors |
| IME logs | `C:\ProgramData\Microsoft\IntuneManagementExtension\Logs` | Captures Win32 app and remediation processing |
| App install command and detection rule | Intune app properties | Proves deployment logic |
| Compliance policy details | Intune compliance policy | Proves compliance requirements |
| CA sign-in log | Entra sign-in logs | Proves access policy impact |
| Defender device page | Defender portal | Proves MDE sensor/risk state |
| Windows Update logs | WindowsUpdate.log and event logs | Proves update scan/install behavior |
| `gpresult.html` | `C:\Temp\IntuneTroubleshooting\gpresult.html` | Proves GPO conflicts |
| Screenshots of failed status | Intune/Entra/Defender portals | Useful for escalation context |

## Common Root Cause Patterns

| Pattern | What It Usually Means | Fix |
|---|---|---|
| User not licensed | Intune policy cannot apply properly | Assign correct license |
| User not in MDM scope | Auto-enrollment does not trigger | Add user/group to MDM scope |
| Device not in assignment group | Policy/app never targets device | Correct group membership |
| Filter excludes device | Assignment exists but does not apply | Fix assignment filter |
| Detection rule wrong | App install state is misreported | Correct detection rule |
| IME service broken | Win32 apps and remediations fail | Restart or repair IME |
| Policy conflict | Multiple policies configure same setting | Remove duplicate setting source |
| GPO conflict | Domain GPO overrides or conflicts with MDM | Reconcile GPO and Intune policy |
| Co-management conflict | ConfigMgr owns workload | Move workload or manage in ConfigMgr |
| CA too broad | Users blocked beyond pilot | Narrow policy and exclude break-glass |
| Reporting delay | Portal lags behind device state | Validate locally and wait |
| Stale object | Duplicate Intune/Entra records confuse targeting | Clean stale record carefully |

## Notes

- Do not re-enroll or wipe first. Start with license, scope, group, assignment, sync, and logs.
- Intune reporting can lag behind local state. Always compare portal status with local evidence.
- Win32 app issues almost always require Intune Management Extension logs.
- Compliance and Conditional Access issues require Entra sign-in logs, not just Intune compliance reports.
- Policy conflicts require per-setting status. A profile-level failure is usually not enough detail.
- Device sync is not magic. If the device has no valid enrollment, stale tokens, broken network, or wrong assignment, sync will not fix the root cause.
- Group Policy and Configuration Manager can still control settings in hybrid environments.
- Break-glass accounts must stay excluded from Conditional Access enforcement.
- Keep troubleshooting evidence before deleting device records.
- Clean stale Intune and Entra device objects carefully. Deleting the wrong object can create more problems.
- Re-enrollment is a valid fix only after lighter fixes fail or the device record is clearly broken.

## Related Workbooks

| Workbook                                                                              | Relationship                                                                                   |
| ------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------- |
| 01_Configure_Intune_Tenant_Enrollment_Baseline_And_Device_Platform_Restrictions.md    | Source baseline for enrollment, MDM scope, platform restrictions, and device limits            |
| 02_Configure_Device_Compliance_Conditional_Access_And_Enrollment_Status.md            | Source baseline for compliance state and Conditional Access behavior                           |
| 03_Configure_Windows_Update_Rings_Feature_Updates_And_Driver_Update_Policies.md       | Source baseline for Windows update policy troubleshooting                                      |
| 04_Configure_Device_Configuration_Profiles_Security_Baselines_And_Settings_Catalog.md | Source baseline for configuration profile and security baseline troubleshooting                |
| 05_Configure_App_Deployment_Assignment_Detection_And_Remediation.md                   | Source baseline for app deployment, detection, IME, and remediation troubleshooting            |
| 06_Configure_Defender_For_Endpoint_Integration_And_Device_Risk_Signals.md             | Source baseline for MDE onboarding, Defender health, and risk-based compliance troubleshooting |