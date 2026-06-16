# 03_Configure_Windows_Update_Rings_Feature_Updates_And_Driver_Update_Policies

## Objective

Configure baseline Windows update management in Microsoft Intune using Windows Update rings, Feature update policies, Quality update behavior, and Driver update policies. This workbook establishes controlled Windows servicing for pilot devices before expanding update enforcement across production groups.

## Lab Context

This workbook assumes Microsoft Intune enrollment and compliance baselines are already configured. The goal is to control how enrolled Windows 10/11 devices receive updates, how quickly they install updates, how restarts are handled, how feature updates are held to a target release, and how drivers are approved or managed.

## Prerequisites

| Requirement | Details |
|---|---|
| Admin role | Intune Administrator, Windows Update for Business Deployment Service administrator, or Global Administrator |
| Portal access | Microsoft Intune admin center |
| Existing baseline | Task 01 enrollment baseline and Task 02 compliance baseline should be complete |
| Managed devices | At least one enrolled Windows 10/11 test device |
| Device state | Device should be Microsoft Entra joined or hybrid joined and Intune managed |
| Pilot group | Dedicated Windows pilot device group |
| Licensing | Microsoft Intune licensing for managed Windows devices |
| Network access | Devices must reach Microsoft update services |
| Test window | Schedule testing where forced reboots will not disrupt users |

## Naming / Variables

| Variable | Example Value | Purpose |
|---|---|---|
| Pilot device group | GRP-INTUNE-Windows-Pilot-Devices | Target group for update policies |
| Broad deployment group | GRP-INTUNE-Windows-Production-Devices | Later production rollout group |
| Update ring policy | INTUNE-WIN-UpdateRing-Pilot-Baseline | Windows Update ring pilot policy |
| Feature update policy | INTUNE-WIN-FeatureUpdate-Win11-23H2-Pilot | Target Windows feature release |
| Driver update policy | INTUNE-WIN-DriverUpdate-Pilot-ManualApproval | Driver control baseline |
| Expedite policy | INTUNE-WIN-QualityUpdate-Expedite-Pilot | Emergency quality update policy |
| Test device | WIN11-INTUNE-01 | Pilot validation endpoint |
| Active hours start | 08:00 | Start of business hours |
| Active hours end | 17:00 | End of business hours |
| Quality update deferral | 3 days | Pilot quality update deferral |
| Feature update deferral | 7 days | Pilot feature update deferral |
| Deadline | 7 days | Update install deadline |
| Grace period | 2 days | Restart grace period |

## Configuration Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|---:|---|---|---|---|---|
| 1 | Sign in to Microsoft Intune admin center | Admin workstation | Browser: `https://intune.microsoft.com` | N/A | Intune admin center opens successfully |
| 2 | Confirm pilot Windows device exists | Admin workstation | Devices > All devices | `Get-MgDeviceManagementManagedDevice -Filter "contains(deviceName,'WIN11-INTUNE-01')"` | Pilot Windows device appears as Intune managed |
| 3 | Confirm pilot device group exists | Admin workstation | Groups > search `GRP-INTUNE-Windows-Pilot-Devices` | `Get-MgGroup -Filter "displayName eq 'GRP-INTUNE-Windows-Pilot-Devices'"` | Pilot device group exists |
| 4 | Add test device to pilot device group | Admin workstation | Groups > pilot device group > Members > Add members | `New-MgGroupMember -GroupId <GroupId> -DirectoryObjectId <DeviceObjectId>` | Test device is targeted for Windows update policies |
| 5 | Open Windows update policy area | Admin workstation | Devices > Windows > Update rings for Windows 10 and later | N/A | Update ring policy blade opens |
| 6 | Create pilot update ring | Admin workstation | Create profile | N/A | Update ring wizard opens |
| 7 | Name update ring | Admin workstation | Name: `INTUNE-WIN-UpdateRing-Pilot-Baseline` | N/A | Policy has clear pilot name |
| 8 | Configure Microsoft product updates | Admin workstation | Update settings > Microsoft product updates: Allow | N/A | Microsoft product updates are included |
| 9 | Configure Windows drivers setting | Admin workstation | Update settings > Windows drivers: Allow or Block depending driver policy model | N/A | Driver behavior is intentionally controlled |
| 10 | Configure quality update deferral | Admin workstation | Quality update deferral period: `3` days | N/A | Pilot quality updates are deferred briefly |
| 11 | Configure feature update deferral | Admin workstation | Feature update deferral period: `7` days | N/A | Feature updates are delayed for pilot control |
| 12 | Configure upgrade to Windows 11 behavior | Admin workstation | Upgrade Windows 10 devices to Latest Windows 11 release: based on tenant standard | N/A | Windows 11 upgrade behavior is intentional |
| 13 | Configure automatic update behavior | Admin workstation | Automatic update behavior: Auto install and restart at maintenance time or Auto install and restart without end-user control | N/A | Devices install updates without relying on manual user action |
| 14 | Configure active hours | Admin workstation | Active hours start: `08:00`; Active hours end: `17:00` | N/A | Restarts avoid normal working hours |
| 15 | Configure restart checks | Admin workstation | Restart checks: Allow | N/A | Windows checks battery, user presence, and other restart blockers |
| 16 | Configure option to pause updates | Admin workstation | Option to pause Windows updates: Disable | N/A | Users cannot pause managed update flow |
| 17 | Configure option to check for updates | Admin workstation | Option to check for Windows updates: Enable or Use default | N/A | User update check behavior matches admin preference |
| 18 | Configure deadline for feature updates | Admin workstation | Deadline for feature updates: `7` days | N/A | Feature updates have enforced deadline |
| 19 | Configure deadline for quality updates | Admin workstation | Deadline for quality updates: `7` days | N/A | Quality updates have enforced deadline |
| 20 | Configure grace period | Admin workstation | Grace period: `2` days | N/A | Users receive short restart grace period |
| 21 | Configure auto-reboot before deadline | Admin workstation | Auto reboot before deadline: No or Yes based on policy | N/A | Restart enforcement behavior is documented |
| 22 | Assign update ring to pilot device group | Admin workstation | Assignments > Add groups > `GRP-INTUNE-Windows-Pilot-Devices` | N/A | Pilot Windows devices receive the update ring |
| 23 | Exclude sensitive devices if needed | Admin workstation | Assignments > Excluded groups | N/A | Break-glass, kiosk, or fragile lab devices are excluded if needed |
| 24 | Review and create update ring | Admin workstation | Review + create > Create | N/A | Pilot update ring is created |
| 25 | Open Feature updates policy area | Admin workstation | Devices > Windows > Feature updates for Windows 10 and later | N/A | Feature updates blade opens |
| 26 | Create feature update policy | Admin workstation | Create profile | N/A | Feature update wizard opens |
| 27 | Name feature update policy | Admin workstation | Name: `INTUNE-WIN-FeatureUpdate-Win11-23H2-Pilot` | N/A | Policy has clear target release name |
| 28 | Select feature update target version | Admin workstation | Feature update to deploy: select approved Windows release | N/A | Devices are held to selected feature update version |
| 29 | Configure rollout options | Admin workstation | Make update available as soon as possible or phased rollout | N/A | Feature update rollout behavior is defined |
| 30 | Assign feature update policy | Admin workstation | Assignments > Add groups > `GRP-INTUNE-Windows-Pilot-Devices` | N/A | Pilot devices receive target feature update policy |
| 31 | Review and create feature update policy | Admin workstation | Review + create > Create | N/A | Feature update policy is created |
| 32 | Open Driver updates policy area | Admin workstation | Devices > Windows > Driver updates for Windows 10 and later | N/A | Driver updates blade opens |
| 33 | Create driver update policy | Admin workstation | Create profile | N/A | Driver update wizard opens |
| 34 | Name driver update policy | Admin workstation | Name: `INTUNE-WIN-DriverUpdate-Pilot-ManualApproval` | N/A | Driver update policy has clear name |
| 35 | Select driver approval method | Admin workstation | Approval method: Manually approve and deploy driver updates | N/A | Drivers require admin approval before deployment |
| 36 | Configure driver policy assignment | Admin workstation | Assignments > Add groups > `GRP-INTUNE-Windows-Pilot-Devices` | N/A | Pilot devices are targeted for driver management |
| 37 | Review and create driver update policy | Admin workstation | Review + create > Create | N/A | Driver update policy is created |
| 38 | Allow driver inventory to populate | Admin workstation | Driver update policy > Recommended drivers / Other drivers | N/A | Available driver list begins populating after device scan |
| 39 | Review recommended driver list | Admin workstation | Driver update policy > Recommended drivers | N/A | Driver candidates are visible |
| 40 | Approve selected pilot drivers | Admin workstation | Select driver > Approve | N/A | Approved drivers are made available to pilot devices |
| 41 | Decline or leave risky drivers unapproved | Admin workstation | Driver policy > driver list | N/A | Unwanted drivers are not deployed |
| 42 | Open Quality updates expedite area if emergency update is required | Admin workstation | Devices > Windows > Quality updates for Windows 10 and later | N/A | Quality update expedite blade opens |
| 43 | Create expedite policy only for emergency patch test | Admin workstation | Create profile | N/A | Expedite wizard opens |
| 44 | Name expedite policy | Admin workstation | Name: `INTUNE-WIN-QualityUpdate-Expedite-Pilot` | N/A | Expedite policy has clear emergency-use name |
| 45 | Select target quality update | Admin workstation | Expedite installation of quality updates if device OS version is less than selected update | N/A | Selected minimum quality update is enforced |
| 46 | Configure expedite restart grace period | Admin workstation | Number of days to wait before forced restart: `1` or approved emergency value | N/A | Emergency restart behavior is defined |
| 47 | Assign expedite policy to pilot only | Admin workstation | Assignments > Add groups > `GRP-INTUNE-Windows-Pilot-Devices` | N/A | Expedite policy targets pilot devices only |
| 48 | Review and create expedite policy | Admin workstation | Review + create > Create | N/A | Expedite policy is created only if needed |
| 49 | Sync pilot Windows device | Windows test device | Settings > Accounts > Access work or school > Info > Sync | N/A | Device checks in with Intune |
| 50 | Trigger device sync from Intune | Admin workstation | Devices > All devices > select device > Sync | `Sync-MgDeviceManagementManagedDevice -ManagedDeviceId <ManagedDeviceId>` | Sync command is queued |
| 51 | Review Windows update reports | Admin workstation | Reports > Windows updates | N/A | Update reporting area is available |
| 52 | Review update ring report | Admin workstation | Devices > Windows > Update rings > select policy > Device status | N/A | Pilot device status appears |
| 53 | Review feature update report | Admin workstation | Devices > Windows > Feature updates > select policy > Reports | N/A | Feature update policy status appears |
| 54 | Review driver update report | Admin workstation | Devices > Windows > Driver updates > select policy | N/A | Driver update status appears |
| 55 | Document final policy values | Admin workstation | Workbook notes / repo documentation | N/A | Policy settings are reproducible |

## Validation Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|---:|---|---|---|---|---|
| 1 | Confirm update ring exists | Admin workstation | Intune > Devices > Windows > Update rings | N/A | `INTUNE-WIN-UpdateRing-Pilot-Baseline` is present |
| 2 | Confirm update ring assignment | Admin workstation | Update ring > Properties > Assignments | N/A | Pilot device group is assigned |
| 3 | Confirm feature update policy exists | Admin workstation | Devices > Windows > Feature updates | N/A | Feature update policy is present |
| 4 | Confirm feature update assignment | Admin workstation | Feature update policy > Assignments | N/A | Pilot device group is assigned |
| 5 | Confirm driver update policy exists | Admin workstation | Devices > Windows > Driver updates | N/A | Driver update policy is present |
| 6 | Confirm driver update assignment | Admin workstation | Driver update policy > Assignments | N/A | Pilot device group is assigned |
| 7 | Confirm expedite policy scope if created | Admin workstation | Devices > Windows > Quality updates | N/A | Expedite policy exists only if intentionally created |
| 8 | Confirm pilot device received policy | Admin workstation | Devices > All devices > test device > Device configuration | N/A | Update policy assignment appears for device |
| 9 | Confirm device check-in time | Admin workstation | Devices > All devices > test device > Overview | N/A | Last check-in time is recent |
| 10 | Confirm Windows Update settings on client | Windows test device | Settings > Windows Update > Advanced options | N/A | Settings reflect organizational management |
| 11 | Generate Windows update report locally | Windows test device | `Get-WindowsUpdateLog` | `Get-WindowsUpdateLog` | WindowsUpdate.log is generated on desktop |
| 12 | Check update policy registry area | Windows test device | `reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate /s` | `Get-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\*" -ErrorAction SilentlyContinue` | Windows Update policy values are present |
| 13 | Check update service status | Windows test device | `sc query wuauserv` | `Get-Service wuauserv` | Windows Update service is available |
| 14 | Check Update Orchestrator service status | Windows test device | `sc query UsoSvc` | `Get-Service UsoSvc` | Update Orchestrator service is available |
| 15 | Force update scan from client | Windows test device | `UsoClient StartScan` | `Start-Process UsoClient.exe -ArgumentList "StartScan"` | Device initiates update scan |
| 16 | Check Windows Update event logs | Windows test device | Event Viewer > Applications and Services Logs > Microsoft > Windows > WindowsUpdateClient > Operational | `Get-WinEvent -LogName "Microsoft-Windows-WindowsUpdateClient/Operational" -MaxEvents 25` | Recent scan/install events appear |
| 17 | Confirm update ring device status | Admin workstation | Update ring > Device status | N/A | Device shows Succeeded, Pending, or explainable status |
| 18 | Confirm feature update status | Admin workstation | Feature update policy > Reports | N/A | Device is targeted to correct feature release |
| 19 | Confirm driver update inventory | Admin workstation | Driver update policy > Drivers | N/A | Recommended or other drivers appear after scan |
| 20 | Confirm approved driver status | Admin workstation | Driver update policy > Approved drivers | N/A | Approved drivers show deployment state |
| 21 | Confirm no production devices targeted | Admin workstation | Each update policy > Assignments | N/A | Only pilot groups are assigned |
| 22 | Confirm compliance impact | Admin workstation | Devices > Compliance policies > Device status | N/A | Device remains compliant or shows update-related reason |
| 23 | Confirm Conditional Access still works | Windows test device | Sign in to Microsoft 365 target app | N/A | Compliant managed device still receives access |
| 24 | Record validation result | Admin workstation | Workbook notes | N/A | Update baseline validation is documented |

## Troubleshooting Checks

| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| Device does not receive update policy | Device not in assignment group or has not synced | Intune device assignment and last check-in | Add device to pilot group and sync |
| Policy shows pending for long period | Intune check-in delay or device offline | Device overview and local network state | Bring device online and trigger sync |
| Windows Update page does not show managed settings | Policy not applied or conflicting policy exists | Registry and Intune device configuration status | Remove conflicting GPO/MDM policy and re-sync |
| Updates do not scan | Windows Update service disabled or blocked network | `Get-Service wuauserv`; WindowsUpdateClient logs | Start service and allow Microsoft update endpoints |
| Feature update does not offer | Safeguard hold, unsupported hardware, policy conflict, or target version already met | Feature update report and Windows Update logs | Review safeguard/reporting and confirm device eligibility |
| Feature update installs newer version than expected | Feature update policy missing or not assigned | Feature update policy assignment | Assign target feature update policy correctly |
| Quality update installs too soon | Deferral period too short or expedite policy active | Update ring and quality update policies | Adjust deferral or remove expedite policy |
| Device restarts during business hours | Active hours or deadline settings too aggressive | Update ring restart settings | Adjust active hours, deadline, and grace period |
| Users can pause updates | Pause option not disabled | Update ring user experience settings | Disable pause option in update ring |
| Drivers are not listed | Driver policy needs time and device scan data | Driver update policy inventory | Sync device and wait for inventory population |
| Approved driver does not install | Driver not applicable, blocked by safeguard, or device offline | Driver policy report and Windows Update logs | Confirm applicability and client scan state |
| Duplicate update behavior from GPO | Existing Group Policy controls Windows Update | `gpresult /h C:\Temp\gp.html` | Remove or reconcile GPO with MDM policy |
| Co-management conflict exists | Configuration Manager manages Windows Update workload | Configuration Manager co-management workload settings | Move Windows Update workload to Intune if that is the design |
| Reports are empty | Reporting delay or no targeted devices | Policy assignment and report refresh time | Wait for data and confirm target devices |
| Compliance fails after update baseline | Minimum OS or reboot requirement not satisfied | Intune compliance details | Complete update/restart or adjust compliance baseline |
| Emergency expedite policy affects too many devices | Assignment too broad | Quality update policy assignments | Limit expedite policy to pilot or emergency group only |

## Rollback / Cleanup

| Step | Task | Device | Command | PowerShell | Expected Result |
|---:|---|---|---|---|---|
| 1 | Disable or unassign expedite policy first | Admin workstation | Devices > Windows > Quality updates > policy > Assignments | N/A | Emergency expedited update pressure is removed |
| 2 | Remove pilot group from driver update policy | Admin workstation | Driver updates > policy > Assignments | N/A | Pilot devices no longer receive driver policy |
| 3 | Remove approved driver if needed | Admin workstation | Driver update policy > Approved drivers | N/A | Driver deployment is stopped for uninstalled devices |
| 4 | Remove pilot group from feature update policy | Admin workstation | Feature updates > policy > Assignments | N/A | Feature target hold no longer applies |
| 5 | Remove pilot group from update ring | Admin workstation | Update rings > policy > Assignments | N/A | Update ring no longer targets pilot devices |
| 6 | Delete test expedite policy if no longer needed | Admin workstation | Quality updates > select policy > Delete | N/A | Expedite policy is removed |
| 7 | Delete test driver update policy if no longer needed | Admin workstation | Driver updates > select policy > Delete | N/A | Driver update policy is removed |
| 8 | Delete test feature update policy if no longer needed | Admin workstation | Feature updates > select policy > Delete | N/A | Feature update policy is removed |
| 9 | Delete test update ring if no longer needed | Admin workstation | Update rings > select policy > Delete | N/A | Update ring is removed |
| 10 | Sync pilot device after rollback | Windows test device | Settings > Accounts > Access work or school > Info > Sync | `Start-Process deviceenroller.exe -ArgumentList "/c /AutoEnrollMDM"` | Device checks in after assignment removal |
| 11 | Check local Windows Update policy after rollback | Windows test device | `reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate /s` | `Get-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\*" -ErrorAction SilentlyContinue` | Removed policy values eventually clear or are replaced by remaining policy source |
| 12 | Document rollback state | Admin workstation | Workbook notes | N/A | Rollback is recorded |

## Notes

- Use update rings for ongoing Windows Update behavior.
- Use Feature update policies to hold devices to a specific Windows release.
- Use Quality update expedite policies only for urgent security or emergency patch scenarios.
- Use Driver update policies to control driver rollout instead of letting every driver land automatically.
- Keep pilot and production update groups separate.
- Do not mix unmanaged Group Policy Windows Update settings with Intune Windows Update for Business settings unless the conflict is intentional and documented.
- Co-managed devices require clear workload ownership. If Configuration Manager owns Windows Update policy, Intune update rings may not behave as expected.
- Feature updates and driver updates may take time to appear in reports because the device must scan and report applicability.
- A forced restart policy is powerful. Keep pilot settings firm but not reckless.
- Update policies should be validated before they are tied to compliance and Conditional Access enforcement.

## Related Workbooks

| Workbook                                                                              | Relationship                                                                       |
| ------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------- |
| 01_Configure_Intune_Tenant_Enrollment_Baseline_And_Device_Platform_Restrictions.md    | Provides the enrollment foundation required before Windows update policy targeting |
| 02_Configure_Device_Compliance_Conditional_Access_And_Enrollment_Status.md            | Uses update state as part of device compliance and access enforcement              |
| 04_Configure_Device_Configuration_Profiles_Security_Baselines_And_Settings_Catalog.md | Applies Windows security settings that complement update management                |
| 05_Configure_App_Deployment_Assignment_Detection_And_Remediation.md                   | Deploys apps after device update posture is controlled                             |
| 06_Configure_Defender_For_Endpoint_Integration_And_Device_Risk_Signals.md             | Adds security risk signals that depend on updated endpoint health                  |
| 07_Troubleshoot_Intune_Enrollment_Compliance_App_Deployment_And_Device_Sync_Issues.md | Troubleshoots update policy sync, reporting, and compliance failures               |