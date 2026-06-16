# 04_Configure_Device_Configuration_Profiles_Security_Baselines_And_Settings_Catalog

## Objective

Configure baseline Microsoft Intune device configuration using Windows configuration profiles, Intune security baselines, Endpoint security policy areas, and the Settings catalog. This workbook establishes controlled device hardening after enrollment, compliance, and update management are in place.

## Lab Context

This workbook assumes Windows devices are already enrolled in Intune and are receiving compliance and update policies. The goal is to configure core device settings using the correct Intune policy type for each job:

- Use security baselines for Microsoft-recommended security bundles.
- Use Endpoint security policies for focused security areas like antivirus, firewall, disk encryption, and attack surface reduction.
- Use Settings catalog profiles for granular Windows settings.
- Use templates only where they are still the clearest fit.
- Assign everything to pilot groups first before broad production rollout.

## Prerequisites

| Requirement | Details |
|---|---|
| Admin role | Intune Administrator, Endpoint Security Manager, Security Administrator, or Global Administrator |
| Portal access | Microsoft Intune admin center |
| Existing baseline | Task 01 enrollment baseline, Task 02 compliance baseline, and Task 03 update baseline should be complete |
| Managed devices | At least one enrolled Windows 10/11 test device |
| Device state | Microsoft Entra joined or hybrid joined and Intune managed |
| Pilot group | Dedicated Windows pilot device group |
| Licensing | Microsoft Intune licensing for managed Windows devices |
| Test user | Licensed Intune pilot user |
| Change control | Pilot deployment window and rollback plan |
| Exclusions | Break-glass, kiosk, shared, or fragile devices excluded where needed |

## Naming / Variables

| Variable | Example Value | Purpose |
|---|---|---|
| Pilot device group | GRP-INTUNE-Windows-Pilot-Devices | Target group for test Windows devices |
| Pilot user group | GRP-INTUNE-Pilot-Users | Optional user-targeted settings |
| Exclusion group | GRP-INTUNE-Exclude-Critical-Devices | Devices excluded from hardening tests |
| Security baseline | INTUNE-WIN-SecurityBaseline-Pilot | Windows security baseline profile |
| Defender baseline | INTUNE-MDE-SecurityBaseline-Pilot | Defender for Endpoint security baseline profile |
| Settings catalog profile | INTUNE-WIN-SettingsCatalog-CoreBaseline-Pilot | Granular Windows settings |
| Device restrictions profile | INTUNE-WIN-DeviceRestrictions-Pilot | Windows device restrictions if template is used |
| Firewall policy | INTUNE-WIN-Firewall-Pilot | Endpoint security firewall policy |
| Antivirus policy | INTUNE-WIN-Antivirus-Pilot | Endpoint security antivirus policy |
| Disk encryption policy | INTUNE-WIN-BitLocker-Pilot | Endpoint security disk encryption policy |
| ASR policy | INTUNE-WIN-ASR-Pilot | Attack surface reduction policy |
| Account protection policy | INTUNE-WIN-AccountProtection-Pilot | Local admin and identity protection |
| Test device | WIN11-INTUNE-01 | Pilot validation endpoint |

## Configuration Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|---:|---|---|---|---|---|
| 1 | Sign in to Microsoft Intune admin center | Admin workstation | Browser: `https://intune.microsoft.com` | N/A | Intune admin center opens successfully |
| 2 | Confirm pilot Windows device exists | Admin workstation | Devices > All devices | `Get-MgDeviceManagementManagedDevice -Filter "contains(deviceName,'WIN11-INTUNE-01')"` | Pilot Windows device appears as Intune managed |
| 3 | Confirm pilot device group exists | Admin workstation | Groups > search `GRP-INTUNE-Windows-Pilot-Devices` | `Get-MgGroup -Filter "displayName eq 'GRP-INTUNE-Windows-Pilot-Devices'"` | Pilot device group is available |
| 4 | Confirm exclusion group exists if needed | Admin workstation | Groups > search `GRP-INTUNE-Exclude-Critical-Devices` | `Get-MgGroup -Filter "displayName eq 'GRP-INTUNE-Exclude-Critical-Devices'"` | Critical device exclusion group is available |
| 5 | Confirm test device is in pilot group | Admin workstation | Group > Members | `Get-MgGroupMember -GroupId <GroupId>` | Test device is targeted |
| 6 | Review existing device configuration profiles | Admin workstation | Devices > Configuration profiles | N/A | Current profile inventory is known |
| 7 | Review existing Endpoint security policies | Admin workstation | Endpoint security | N/A | Existing security policies are known |
| 8 | Review existing security baselines | Admin workstation | Endpoint security > Security baselines | N/A | Existing baseline assignments are known |
| 9 | Check for policy overlap before creating new profiles | Admin workstation | Devices > Configuration profiles and Endpoint security | N/A | Duplicate settings are identified before deployment |
| 10 | Open Windows security baseline area | Admin workstation | Endpoint security > Security baselines | N/A | Security baseline templates are visible |
| 11 | Create Windows security baseline profile | Admin workstation | Security baselines > Windows security baseline > Create profile | N/A | Windows security baseline wizard opens |
| 12 | Name Windows security baseline | Admin workstation | Name: `INTUNE-WIN-SecurityBaseline-Pilot` | N/A | Baseline profile has clear pilot name |
| 13 | Review baseline categories | Admin workstation | Configuration settings | N/A | Security setting groups are visible |
| 14 | Keep Microsoft recommended values unless lab requires exception | Admin workstation | Configuration settings | N/A | Baseline remains close to Microsoft defaults |
| 15 | Adjust only known-impact settings | Admin workstation | Configuration settings | N/A | High-impact settings are documented if changed |
| 16 | Assign Windows security baseline to pilot devices | Admin workstation | Assignments > Include > `GRP-INTUNE-Windows-Pilot-Devices` | N/A | Pilot devices receive security baseline |
| 17 | Exclude critical devices if needed | Admin workstation | Assignments > Exclude > `GRP-INTUNE-Exclude-Critical-Devices` | N/A | Fragile devices are protected from pilot hardening |
| 18 | Review and create Windows security baseline | Admin workstation | Review + create > Create | N/A | Windows security baseline is created |
| 19 | Create Microsoft Defender for Endpoint baseline if available | Admin workstation | Endpoint security > Security baselines > Microsoft Defender for Endpoint baseline > Create profile | N/A | Defender baseline wizard opens |
| 20 | Name Defender baseline | Admin workstation | Name: `INTUNE-MDE-SecurityBaseline-Pilot` | N/A | Defender baseline has clear pilot name |
| 21 | Configure Defender baseline settings | Admin workstation | Configuration settings | N/A | Defender settings are reviewed and accepted or documented |
| 22 | Assign Defender baseline to pilot devices | Admin workstation | Assignments > Include > `GRP-INTUNE-Windows-Pilot-Devices` | N/A | Pilot devices receive Defender baseline |
| 23 | Review and create Defender baseline | Admin workstation | Review + create > Create | N/A | Defender baseline is created |
| 24 | Open Endpoint security antivirus area | Admin workstation | Endpoint security > Antivirus | N/A | Antivirus policy blade opens |
| 25 | Create antivirus policy | Admin workstation | Create policy > Platform: Windows 10, Windows 11, and Windows Server > Profile: Microsoft Defender Antivirus | N/A | Antivirus policy wizard opens |
| 26 | Name antivirus policy | Admin workstation | Name: `INTUNE-WIN-Antivirus-Pilot` | N/A | Antivirus policy has clear pilot name |
| 27 | Configure Defender real-time protection | Admin workstation | Configuration settings > Real-time protection | N/A | Defender real-time protection is enabled |
| 28 | Configure cloud-delivered protection | Admin workstation | Configuration settings > Cloud protection | N/A | Cloud-delivered protection is enabled |
| 29 | Configure sample submission | Admin workstation | Configuration settings > Submit samples consent | N/A | Sample submission behavior is defined |
| 30 | Configure scan settings | Admin workstation | Configuration settings > Scheduled scan | N/A | Scan schedule and scan type are defined |
| 31 | Assign antivirus policy to pilot devices | Admin workstation | Assignments > Include > `GRP-INTUNE-Windows-Pilot-Devices` | N/A | Pilot devices receive antivirus policy |
| 32 | Review and create antivirus policy | Admin workstation | Review + create > Create | N/A | Antivirus policy is created |
| 33 | Open Endpoint security firewall area | Admin workstation | Endpoint security > Firewall | N/A | Firewall policy blade opens |
| 34 | Create firewall policy | Admin workstation | Create policy > Platform: Windows 10, Windows 11, and Windows Server > Profile: Microsoft Defender Firewall | N/A | Firewall policy wizard opens |
| 35 | Name firewall policy | Admin workstation | Name: `INTUNE-WIN-Firewall-Pilot` | N/A | Firewall policy has clear pilot name |
| 36 | Configure domain firewall profile | Admin workstation | Configuration settings > Domain profile | N/A | Domain firewall is enabled |
| 37 | Configure private firewall profile | Admin workstation | Configuration settings > Private profile | N/A | Private firewall is enabled |
| 38 | Configure public firewall profile | Admin workstation | Configuration settings > Public profile | N/A | Public firewall is enabled |
| 39 | Configure inbound default behavior | Admin workstation | Configuration settings > Default inbound action | N/A | Inbound blocking behavior is defined |
| 40 | Assign firewall policy to pilot devices | Admin workstation | Assignments > Include > `GRP-INTUNE-Windows-Pilot-Devices` | N/A | Pilot devices receive firewall policy |
| 41 | Review and create firewall policy | Admin workstation | Review + create > Create | N/A | Firewall policy is created |
| 42 | Open Endpoint security disk encryption area | Admin workstation | Endpoint security > Disk encryption | N/A | Disk encryption policy blade opens |
| 43 | Create BitLocker policy | Admin workstation | Create policy > Platform: Windows 10 and later > Profile: BitLocker | N/A | BitLocker policy wizard opens |
| 44 | Name BitLocker policy | Admin workstation | Name: `INTUNE-WIN-BitLocker-Pilot` | N/A | BitLocker policy has clear pilot name |
| 45 | Configure OS drive encryption | Admin workstation | Configuration settings > Operating system drive | N/A | OS drive encryption is required |
| 46 | Configure recovery key escrow | Admin workstation | Configuration settings > Recovery options | N/A | Recovery key escrow to Entra ID is enabled |
| 47 | Configure silent enablement if supported | Admin workstation | Configuration settings > Silent BitLocker enablement | N/A | BitLocker can enable silently on supported devices |
| 48 | Assign BitLocker policy to pilot devices | Admin workstation | Assignments > Include > `GRP-INTUNE-Windows-Pilot-Devices` | N/A | Pilot devices receive BitLocker policy |
| 49 | Review and create BitLocker policy | Admin workstation | Review + create > Create | N/A | BitLocker policy is created |
| 50 | Open Endpoint security attack surface reduction area | Admin workstation | Endpoint security > Attack surface reduction | N/A | ASR policy blade opens |
| 51 | Create ASR policy | Admin workstation | Create policy > Platform: Windows 10, Windows 11, and Windows Server > Profile: Attack surface reduction rules | N/A | ASR policy wizard opens |
| 52 | Name ASR policy | Admin workstation | Name: `INTUNE-WIN-ASR-Pilot` | N/A | ASR policy has clear pilot name |
| 53 | Configure ASR rules in Audit mode first | Admin workstation | Configuration settings > ASR rules | N/A | ASR impact is monitored before blocking |
| 54 | Configure controlled folder access if required | Admin workstation | Configuration settings > Controlled folder access | N/A | Controlled folder access is audited or enabled intentionally |
| 55 | Assign ASR policy to pilot devices | Admin workstation | Assignments > Include > `GRP-INTUNE-Windows-Pilot-Devices` | N/A | Pilot devices receive ASR policy |
| 56 | Review and create ASR policy | Admin workstation | Review + create > Create | N/A | ASR policy is created |
| 57 | Open Endpoint security account protection area | Admin workstation | Endpoint security > Account protection | N/A | Account protection blade opens |
| 58 | Create account protection policy if needed | Admin workstation | Create policy > Platform: Windows 10 and later | N/A | Account protection wizard opens |
| 59 | Name account protection policy | Admin workstation | Name: `INTUNE-WIN-AccountProtection-Pilot` | N/A | Account protection policy has clear name |
| 60 | Configure local administrator controls if using LAPS or local user group membership | Admin workstation | Configuration settings | N/A | Local admin control is defined |
| 61 | Assign account protection policy to pilot devices | Admin workstation | Assignments > Include > `GRP-INTUNE-Windows-Pilot-Devices` | N/A | Pilot devices receive account protection policy |
| 62 | Review and create account protection policy | Admin workstation | Review + create > Create | N/A | Account protection policy is created |
| 63 | Open Settings catalog | Admin workstation | Devices > Configuration profiles > Create profile > Windows 10 and later > Settings catalog | N/A | Settings catalog wizard opens |
| 64 | Name Settings catalog profile | Admin workstation | Name: `INTUNE-WIN-SettingsCatalog-CoreBaseline-Pilot` | N/A | Settings catalog profile has clear name |
| 65 | Add password or sign-in settings if not already controlled elsewhere | Admin workstation | Add settings > search relevant setting | N/A | Granular setting is added only if not duplicated |
| 66 | Add privacy or experience settings if required | Admin workstation | Add settings > search relevant setting | N/A | User/device experience setting is defined |
| 67 | Add Windows Update UX settings only if not already handled by update rings | Admin workstation | Add settings > Windows Update settings | N/A | No duplicate update management conflict is created |
| 68 | Add Microsoft Edge settings if part of core device baseline | Admin workstation | Add settings > Microsoft Edge | N/A | Browser baseline settings are configured |
| 69 | Add OneDrive known folder move settings if required | Admin workstation | Add settings > OneDrive | N/A | OneDrive baseline is configured if in scope |
| 70 | Review Settings catalog profile for duplicate settings | Admin workstation | Configuration settings summary | N/A | Settings catalog does not duplicate baseline/Endpoint security settings unnecessarily |
| 71 | Assign Settings catalog profile to pilot devices | Admin workstation | Assignments > Include > `GRP-INTUNE-Windows-Pilot-Devices` | N/A | Pilot devices receive Settings catalog profile |
| 72 | Exclude critical devices if needed | Admin workstation | Assignments > Exclude > `GRP-INTUNE-Exclude-Critical-Devices` | N/A | Sensitive devices are excluded |
| 73 | Review and create Settings catalog profile | Admin workstation | Review + create > Create | N/A | Settings catalog profile is created |
| 74 | Create device restrictions template only if needed | Admin workstation | Devices > Configuration profiles > Create profile > Templates > Device restrictions | N/A | Device restrictions profile is used only for template-specific needs |
| 75 | Name device restrictions profile | Admin workstation | Name: `INTUNE-WIN-DeviceRestrictions-Pilot` | N/A | Device restrictions profile has clear name |
| 76 | Configure required device restriction settings | Admin workstation | Configuration settings | N/A | Restrictions are defined and documented |
| 77 | Assign device restrictions profile to pilot devices | Admin workstation | Assignments > Include > `GRP-INTUNE-Windows-Pilot-Devices` | N/A | Pilot devices receive device restrictions profile |
| 78 | Review and create device restrictions profile | Admin workstation | Review + create > Create | N/A | Device restrictions profile is created |
| 79 | Sync pilot Windows device | Windows test device | Settings > Accounts > Access work or school > Info > Sync | N/A | Device checks in with Intune |
| 80 | Trigger sync from Intune | Admin workstation | Devices > All devices > select `WIN11-INTUNE-01` > Sync | `Sync-MgDeviceManagementManagedDevice -ManagedDeviceId <ManagedDeviceId>` | Sync command is queued |
| 81 | Review device configuration status | Admin workstation | Devices > All devices > select device > Device configuration | N/A | Assigned profiles show status |
| 82 | Review Endpoint security policy status | Admin workstation | Endpoint security > policy area > select policy > Device status | N/A | Endpoint security status is visible |
| 83 | Review security baseline status | Admin workstation | Endpoint security > Security baselines > profile > Device status | N/A | Security baseline status is visible |
| 84 | Document policy values and assignments | Admin workstation | Workbook notes / repo documentation | N/A | Configuration is reproducible |

## Validation Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|---:|---|---|---|---|---|
| 1 | Confirm Windows security baseline exists | Admin workstation | Endpoint security > Security baselines | N/A | `INTUNE-WIN-SecurityBaseline-Pilot` is present |
| 2 | Confirm security baseline assignment | Admin workstation | Baseline profile > Assignments | N/A | Pilot device group is assigned |
| 3 | Confirm Defender baseline exists if configured | Admin workstation | Endpoint security > Security baselines | N/A | `INTUNE-MDE-SecurityBaseline-Pilot` is present |
| 4 | Confirm antivirus policy exists | Admin workstation | Endpoint security > Antivirus | N/A | `INTUNE-WIN-Antivirus-Pilot` is present |
| 5 | Confirm firewall policy exists | Admin workstation | Endpoint security > Firewall | N/A | `INTUNE-WIN-Firewall-Pilot` is present |
| 6 | Confirm BitLocker policy exists | Admin workstation | Endpoint security > Disk encryption | N/A | `INTUNE-WIN-BitLocker-Pilot` is present |
| 7 | Confirm ASR policy exists | Admin workstation | Endpoint security > Attack surface reduction | N/A | `INTUNE-WIN-ASR-Pilot` is present |
| 8 | Confirm Settings catalog profile exists | Admin workstation | Devices > Configuration profiles | N/A | `INTUNE-WIN-SettingsCatalog-CoreBaseline-Pilot` is present |
| 9 | Confirm profile assignments | Admin workstation | Each profile > Properties > Assignments | N/A | Pilot group is assigned and exclusions are correct |
| 10 | Confirm device check-in is recent | Admin workstation | Devices > All devices > test device > Overview | N/A | Last check-in time is recent |
| 11 | Confirm device configuration status | Admin workstation | Devices > All devices > test device > Device configuration | N/A | Profiles show Succeeded, Pending, or explainable status |
| 12 | Confirm policy setting status | Admin workstation | Profile > Per-setting status | N/A | Individual setting results are visible |
| 13 | Check local MDM diagnostic report | Windows test device | Settings > Accounts > Access work or school > Info > Create report | N/A | MDM diagnostics report is generated |
| 14 | Export MDM diagnostics from command line | Windows test device | `mdmdiagnosticstool.exe -area DeviceEnrollment;DeviceProvisioning;Autopilot;Policy -cab C:\Temp\MDMDiag.cab` | `Start-Process mdmdiagnosticstool.exe -ArgumentList "-area DeviceEnrollment;DeviceProvisioning;Autopilot;Policy -cab C:\Temp\MDMDiag.cab" -Wait` | MDM diagnostics CAB is created |
| 15 | Check applied MDM policies in registry | Windows test device | `reg query HKLM\SOFTWARE\Microsoft\PolicyManager\current\device /s` | `Get-ChildItem "HKLM:\SOFTWARE\Microsoft\PolicyManager\current\device" -Recurse -ErrorAction SilentlyContinue` | MDM policy values are present |
| 16 | Check Defender Antivirus status | Windows test device | N/A | `Get-MpComputerStatus` | Defender reports real-time protection and engine status |
| 17 | Check Defender preferences | Windows test device | N/A | `Get-MpPreference` | Defender policy preferences reflect Intune configuration |
| 18 | Check firewall profile status | Windows test device | `netsh advfirewall show allprofiles` | `Get-NetFirewallProfile` | Domain, private, and public firewall profiles are enabled as configured |
| 19 | Check BitLocker status | Windows test device | `manage-bde -status` | `Get-BitLockerVolume` | OS drive encryption status matches policy |
| 20 | Confirm BitLocker recovery key escrow | Admin workstation | Entra admin center > Devices > select device > BitLocker keys | N/A | Recovery key is available if encryption completed |
| 21 | Check ASR rule status | Windows test device | N/A | `Get-MpPreference | Select-Object AttackSurfaceReductionRules_Ids,AttackSurfaceReductionRules_Actions` | ASR rules and actions are visible |
| 22 | Check Windows security events for ASR audit | Windows test device | Event Viewer > Applications and Services Logs > Microsoft > Windows > Windows Defender > Operational | `Get-WinEvent -LogName "Microsoft-Windows-Windows Defender/Operational" -MaxEvents 50` | Defender operational events are visible |
| 23 | Confirm local admin membership if account protection was configured | Windows test device | `net localgroup administrators` | `Get-LocalGroupMember -Group Administrators` | Local admin membership matches policy |
| 24 | Confirm user experience settings if configured | Windows test device | Settings app / Edge / OneDrive / Start menu | N/A | User-facing settings match policy |
| 25 | Confirm compliance still succeeds | Admin workstation | Devices > Compliance policies > Device status | N/A | Device remains compliant or has explainable gaps |
| 26 | Confirm Conditional Access still succeeds from compliant device | Windows test device | Sign in to Microsoft 365 app | N/A | Access succeeds from compliant managed device |
| 27 | Review Intune reports | Admin workstation | Reports > Device configuration | N/A | Policy status is visible in reporting |
| 28 | Record validation result | Admin workstation | Workbook notes | N/A | Baseline validation is documented |

## Troubleshooting Checks

| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| Profile remains Pending | Device has not checked in or assignment has not evaluated | Device last check-in and profile assignment | Sync device and wait for policy evaluation |
| Profile shows Error | Unsupported setting, wrong OS version, or policy conflict | Per-setting status and MDM diagnostics | Remove unsupported setting or correct assignment |
| Settings conflict appears | Same CSP setting configured by multiple profiles | Intune profile conflict report and setting path | Pick one policy source for the setting |
| Security baseline conflicts with Settings catalog | Duplicate hardening setting configured in both | Per-setting conflict details | Leave setting in baseline or Settings catalog, not both |
| Endpoint security policy conflicts with security baseline | Antivirus, firewall, ASR, or BitLocker configured twice | Endpoint security reports | Use Endpoint security for focused security areas and remove duplicate baseline override |
| BitLocker does not enable silently | TPM, Secure Boot, user interaction, or hardware readiness issue | `Get-BitLockerVolume`, Event Viewer, device hardware state | Fix hardware readiness or adjust BitLocker policy |
| Recovery key not visible in Entra | Key escrow failed or encryption started outside expected flow | Entra device object and BitLocker event logs | Rotate or back up recovery key after policy correction |
| Defender policy not applying | Third-party AV, disabled Defender, or conflicting policy | `Get-MpComputerStatus` and security center | Remove conflict or adjust Defender configuration |
| Firewall blocks required app | Inbound/outbound rule missing | Firewall logs and app connectivity test | Create scoped firewall rule policy |
| ASR breaks business app | ASR rule set to Block too early | Defender operational log and ASR report | Start with Audit mode, add exclusions, then phase to Block |
| User loses local admin unexpectedly | Account protection policy too aggressive | Local administrators group and policy assignment | Adjust local group membership policy |
| Device restrictions affect kiosk/shared device | Broad assignment or missing exclusion | Profile assignment and group membership | Exclude kiosk/shared device group |
| Setting does not exist on client | OS build does not support the setting | Windows version and policy CSP support | Update OS or remove setting |
| Compliance fails after hardening | Required setting not successfully applied | Compliance details and configuration status | Remediate setting or adjust compliance policy |
| Conditional Access blocks user after profile rollout | Device became noncompliant due to new baseline | Entra sign-in logs and Intune compliance report | Fix device compliance or roll back problematic setting |
| Reports show Not applicable | Wrong platform, edition, or device type | Device OS and policy platform | Assign only supported devices |
| GPO overrides Intune setting | On-prem Group Policy wins or conflicts | `gpresult /h C:\Temp\gp.html` | Remove GPO conflict or define MDM wins over GP where appropriate |
| Co-management conflict exists | Configuration Manager owns endpoint security workload | Co-management workload settings | Move relevant workload to Intune or manage in ConfigMgr |

## Rollback / Cleanup

| Step | Task | Device | Command | PowerShell | Expected Result |
|---:|---|---|---|---|---|
| 1 | Change ASR rules from Block to Audit if user impact occurs | Admin workstation | Endpoint security > Attack surface reduction > policy > Properties | N/A | ASR impact is reduced |
| 2 | Remove pilot assignment from ASR policy | Admin workstation | ASR policy > Assignments | N/A | Pilot devices no longer receive ASR policy |
| 3 | Remove pilot assignment from BitLocker policy if encryption rollout must stop | Admin workstation | Disk encryption policy > Assignments | N/A | New BitLocker enforcement stops |
| 4 | Remove pilot assignment from firewall policy if connectivity breaks | Admin workstation | Firewall policy > Assignments | N/A | Firewall policy no longer targets pilot devices |
| 5 | Remove pilot assignment from antivirus policy if Defender issue occurs | Admin workstation | Antivirus policy > Assignments | N/A | Antivirus policy no longer targets pilot devices |
| 6 | Remove pilot assignment from account protection policy if admin access breaks | Admin workstation | Account protection policy > Assignments | N/A | Local admin changes stop applying |
| 7 | Remove pilot assignment from Settings catalog profile | Admin workstation | Configuration profiles > Settings catalog profile > Assignments | N/A | Granular settings stop applying |
| 8 | Remove pilot assignment from security baseline | Admin workstation | Security baseline profile > Assignments | N/A | Baseline no longer targets pilot devices |
| 9 | Delete test profiles only after assignment removal is confirmed | Admin workstation | Select profile > Delete | N/A | Test policies are removed |
| 10 | Sync pilot device after rollback | Windows test device | Settings > Accounts > Access work or school > Info > Sync | `Sync-MgDeviceManagementManagedDevice -ManagedDeviceId <ManagedDeviceId>` | Device checks in after rollback |
| 11 | Validate local policy state after rollback | Windows test device | `reg query HKLM\SOFTWARE\Microsoft\PolicyManager\current\device /s` | `Get-ChildItem "HKLM:\SOFTWARE\Microsoft\PolicyManager\current\device" -Recurse -ErrorAction SilentlyContinue` | Removed settings eventually clear or are replaced by remaining policy |
| 12 | Check Defender and firewall after rollback | Windows test device | `netsh advfirewall show allprofiles` | `Get-MpComputerStatus; Get-NetFirewallProfile` | Security state is known after rollback |
| 13 | Confirm access still works | Windows test device | Sign in to Microsoft 365 target app | N/A | User can access required services |
| 14 | Document rollback state | Admin workstation | Workbook notes | N/A | Rollback is recorded |

## Notes

- Do not configure the same setting in multiple places unless the conflict is deliberate and documented.
- Use security baselines for broad Microsoft-recommended hardening.
- Use Endpoint security for focused security workloads like Defender Antivirus, Firewall, BitLocker, ASR, and Account protection.
- Use Settings catalog for granular settings that are not cleanly handled by a baseline or Endpoint security policy.
- Deploy ASR rules in Audit mode first. Moving straight to Block is how business apps get broken.
- BitLocker rollout must be handled carefully. Confirm recovery key escrow before broad deployment.
- Avoid assigning hardening policies to all devices until pilot results are clean.
- For hybrid or co-managed devices, check Group Policy and Configuration Manager workload ownership before assuming Intune is the only policy source.
- Security baselines change over time. Review new baseline versions before upgrading existing profiles.
- Configuration success does not always mean the device is secure. Validate locally and through reports.

## Related Workbooks

| Workbook                                                                              | Relationship                                                             |
| ------------------------------------------------------------------------------------- | ------------------------------------------------------------------------ |
| 01_Configure_Intune_Tenant_Enrollment_Baseline_And_Device_Platform_Restrictions.md    | Provides enrollment foundation before configuration profiles can apply   |
| 02_Configure_Device_Compliance_Conditional_Access_And_Enrollment_Status.md            | Uses configuration state to support compliance and access enforcement    |
| 03_Configure_Windows_Update_Rings_Feature_Updates_And_Driver_Update_Policies.md       | Keeps Windows devices patched before and after hardening                 |
| 05_Configure_App_Deployment_Assignment_Detection_And_Remediation.md                   | Deploys required apps after device configuration targeting is stable     |
| 06_Configure_Defender_For_Endpoint_Integration_And_Device_Risk_Signals.md             | Extends Defender policy into MDE integration and risk-based signals      |
| 07_Troubleshoot_Intune_Enrollment_Compliance_App_Deployment_And_Device_Sync_Issues.md | Troubleshoots policy conflicts, profile failures, and device sync issues |