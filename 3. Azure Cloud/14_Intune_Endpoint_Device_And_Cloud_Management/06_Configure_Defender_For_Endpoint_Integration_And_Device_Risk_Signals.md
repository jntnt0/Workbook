# 06_Configure_Defender_For_Endpoint_Integration_And_Device_Risk_Signals

## Objective

Configure Microsoft Defender for Endpoint integration with Microsoft Intune so managed devices can onboard to Defender for Endpoint, report endpoint risk signals, and use machine risk in Intune compliance and Microsoft Entra Conditional Access decisions.

## Lab Context

This workbook assumes Intune enrollment, compliance, update rings, device configuration, security baselines, and app deployment are already configured. The goal is to connect Intune and Microsoft Defender for Endpoint, onboard pilot Windows devices, validate sensor health, confirm device risk reporting, and use Defender risk signals in compliance and Conditional Access policy design.

This workbook focuses mainly on Windows 10/11 because Windows Defender for Endpoint onboarding and risk-based compliance are the most common endpoint administration tasks.

## Prerequisites

| Requirement | Details |
|---|---|
| Admin role | Intune Administrator, Security Administrator, Endpoint Security Manager, Global Administrator, or Security Operator for validation |
| Portal access | Microsoft Intune admin center, Microsoft Defender portal, Microsoft Entra admin center |
| Existing baseline | Tasks 01 through 05 should be complete |
| Licensing | Microsoft Defender for Endpoint Plan 1 or Plan 2, Microsoft Intune, and Microsoft Entra ID P1 or higher for Conditional Access |
| Managed devices | At least one enrolled Windows 10/11 pilot device |
| Device state | Microsoft Entra joined or hybrid joined and Intune managed |
| Pilot group | Dedicated Windows pilot device group |
| Compliance policy | Existing Windows compliance policy or permission to create a new one |
| Conditional Access | Existing report-only CA practice and break-glass exclusions |
| Network access | Devices can reach Microsoft Defender for Endpoint service URLs |
| Test device | Windows test device with Microsoft Defender Antivirus active or compatible EDR configuration |
| Change control | Pilot deployment window and rollback plan |

## Naming / Variables

| Variable | Example Value | Purpose |
|---|---|---|
| Pilot device group | GRP-INTUNE-Windows-Pilot-Devices | Target group for MDE onboarding |
| Pilot user group | GRP-INTUNE-Pilot-Users | Conditional Access and compliance test users |
| Exclusion group | GRP-CA-Exclude-BreakGlass | Emergency access exclusion group |
| Critical device exclusion group | GRP-INTUNE-Exclude-Critical-Devices | Devices excluded from pilot MDE changes |
| MDE connector | Microsoft Defender for Endpoint connector | Intune-to-MDE service connection |
| MDE onboarding profile | INTUNE-WIN-MDE-Onboarding-Pilot | Endpoint detection and response onboarding policy |
| MDE security baseline | INTUNE-MDE-SecurityBaseline-Pilot | Defender security baseline profile |
| Defender AV policy | INTUNE-WIN-DefenderAV-Pilot | Defender Antivirus endpoint security policy |
| ASR policy | INTUNE-WIN-ASR-Pilot | Attack surface reduction policy |
| Risk compliance policy | INTUNE-WIN-Compliance-MDE-Risk-Pilot | Compliance policy using MDE machine risk |
| Risk CA policy | CA-PILOT-Require-Compliant-Device-MDE-Risk | Conditional Access policy using compliance state |
| Test device | WIN11-INTUNE-01 | Pilot validation endpoint |
| Risk threshold | Low or Medium | Maximum allowed MDE machine risk for compliance |
| Defender portal | `https://security.microsoft.com` | Microsoft Defender portal |
| Intune portal | `https://intune.microsoft.com` | Microsoft Intune admin center |
| Entra portal | `https://entra.microsoft.com` | Microsoft Entra admin center |

## Configuration Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|---:|---|---|---|---|---|
| 1 | Sign in to Microsoft Intune admin center | Admin workstation | Browser: `https://intune.microsoft.com` | N/A | Intune admin center opens successfully |
| 2 | Sign in to Microsoft Defender portal | Admin workstation | Browser: `https://security.microsoft.com` | N/A | Defender portal opens successfully |
| 3 | Confirm pilot Windows device exists | Admin workstation | Intune > Devices > All devices | `Get-MgDeviceManagementManagedDevice -Filter "contains(deviceName,'WIN11-INTUNE-01')"` | Pilot Windows device appears as Intune managed |
| 4 | Confirm pilot device group exists | Admin workstation | Intune > Groups > search `GRP-INTUNE-Windows-Pilot-Devices` | `Get-MgGroup -Filter "displayName eq 'GRP-INTUNE-Windows-Pilot-Devices'"` | Pilot device group is available |
| 5 | Confirm test device is in pilot device group | Admin workstation | Group > Members | `Get-MgGroupMember -GroupId <GroupId>` | Test device is targeted |
| 6 | Confirm pilot user group exists | Admin workstation | Groups > search `GRP-INTUNE-Pilot-Users` | `Get-MgGroup -Filter "displayName eq 'GRP-INTUNE-Pilot-Users'"` | Pilot user group is available |
| 7 | Confirm break-glass exclusion group exists | Admin workstation | Entra > Groups > search `GRP-CA-Exclude-BreakGlass` | `Get-MgGroup -Filter "displayName eq 'GRP-CA-Exclude-BreakGlass'"` | CA exclusion group is available |
| 8 | Confirm Defender for Endpoint tenant is provisioned | Admin workstation | Defender portal > Settings > Endpoints | N/A | Endpoint settings are available |
| 9 | Review Defender licensing | Admin workstation | Microsoft 365 admin center > Billing > Licenses | N/A | Defender for Endpoint licensing is present |
| 10 | Review Defender endpoint advanced features | Admin workstation | Defender portal > Settings > Endpoints > Advanced features | N/A | Advanced feature settings are visible |
| 11 | Enable Microsoft Intune connection in Defender if required | Admin workstation | Defender portal > Settings > Endpoints > Advanced features > Microsoft Intune connection | N/A | Defender allows Intune integration |
| 12 | Enable device discovery if required | Admin workstation | Defender portal > Settings > Endpoints > Advanced features > Device discovery | N/A | Defender discovery is enabled if part of tenant standard |
| 13 | Enable live response if required | Admin workstation | Defender portal > Settings > Endpoints > Advanced features > Live response | N/A | Live response is enabled if authorized |
| 14 | Enable automated investigation if licensed and approved | Admin workstation | Defender portal > Settings > Endpoints > Advanced features | N/A | Automated investigation behavior is configured |
| 15 | Open Intune Defender connector | Admin workstation | Intune > Endpoint security > Microsoft Defender for Endpoint | N/A | Defender connector page opens |
| 16 | Connect Intune to Defender for Endpoint | Admin workstation | Endpoint security > Microsoft Defender for Endpoint > Connect | N/A | Intune connector status changes to connected |
| 17 | Enable connecting Windows devices to Defender for Endpoint | Admin workstation | Microsoft Defender for Endpoint connector settings | N/A | Windows device onboarding through Intune is enabled |
| 18 | Enable compliance policy evaluation with MDE signals | Admin workstation | Connector settings > Compliance policy evaluation | N/A | Intune can use MDE risk signals for compliance |
| 19 | Enable app protection or mobile threat defense integration only if in scope | Admin workstation | Connector settings | N/A | Non-Windows integration is intentionally enabled or left disabled |
| 20 | Save connector settings | Admin workstation | Review/save connector configuration | N/A | Connector settings are committed |
| 21 | Open Endpoint detection and response policy area | Admin workstation | Intune > Endpoint security > Endpoint detection and response | N/A | EDR policy blade opens |
| 22 | Create EDR onboarding policy | Admin workstation | Create policy > Platform: Windows 10, Windows 11, and Windows Server > Profile: Endpoint detection and response | N/A | EDR policy wizard opens |
| 23 | Name EDR onboarding policy | Admin workstation | Name: `INTUNE-WIN-MDE-Onboarding-Pilot` | N/A | EDR policy has clear pilot name |
| 24 | Configure Microsoft Defender for Endpoint client configuration package type | Admin workstation | Configuration settings | N/A | Auto from connector or onboarding blob is configured depending portal experience |
| 25 | Configure sample sharing if shown | Admin workstation | Configuration settings | N/A | Sample sharing setting matches tenant policy |
| 26 | Configure telemetry reporting frequency if shown | Admin workstation | Configuration settings | N/A | Reporting behavior is set |
| 27 | Assign EDR onboarding policy to pilot devices | Admin workstation | Assignments > Include > `GRP-INTUNE-Windows-Pilot-Devices` | N/A | Pilot devices receive MDE onboarding policy |
| 28 | Exclude critical devices if needed | Admin workstation | Assignments > Exclude > `GRP-INTUNE-Exclude-Critical-Devices` | N/A | Sensitive devices are excluded |
| 29 | Review and create EDR onboarding policy | Admin workstation | Review + create > Create | N/A | MDE onboarding policy is created |
| 30 | Open Antivirus policy area | Admin workstation | Intune > Endpoint security > Antivirus | N/A | Antivirus policy blade opens |
| 31 | Create or update Defender Antivirus policy | Admin workstation | Create policy or select existing `INTUNE-WIN-DefenderAV-Pilot` | N/A | Defender AV settings are managed |
| 32 | Enable real-time protection | Admin workstation | Defender Antivirus policy > Configuration settings | N/A | Real-time protection is enabled |
| 33 | Enable cloud-delivered protection | Admin workstation | Defender Antivirus policy > Configuration settings | N/A | Cloud protection is enabled |
| 34 | Enable behavior monitoring | Admin workstation | Defender Antivirus policy > Configuration settings | N/A | Behavior monitoring is enabled |
| 35 | Configure PUA protection | Admin workstation | Defender Antivirus policy > Potentially unwanted app protection | N/A | PUA protection is enabled or audited |
| 36 | Configure Defender exclusions only if required | Admin workstation | Defender Antivirus policy > Exclusions | N/A | Exclusions are minimized and documented |
| 37 | Assign Defender Antivirus policy to pilot devices | Admin workstation | Assignments > Include > `GRP-INTUNE-Windows-Pilot-Devices` | N/A | Pilot devices receive Defender AV policy |
| 38 | Save Defender Antivirus policy | Admin workstation | Review + create or Review + save | N/A | Defender AV policy is active |
| 39 | Open Attack surface reduction policy area | Admin workstation | Intune > Endpoint security > Attack surface reduction | N/A | ASR policy blade opens |
| 40 | Create or update ASR policy | Admin workstation | Create policy or select existing `INTUNE-WIN-ASR-Pilot` | N/A | ASR settings are managed |
| 41 | Configure ASR rules in Audit first | Admin workstation | ASR rules > set initial rules to Audit | N/A | Impact can be measured before blocking |
| 42 | Configure selected high-confidence ASR rules to Block after validation | Admin workstation | ASR rules | N/A | Validated ASR rules can block risky behavior |
| 43 | Configure controlled folder access in Audit first if used | Admin workstation | Controlled folder access | N/A | Ransomware protection impact is monitored |
| 44 | Assign ASR policy to pilot devices | Admin workstation | Assignments > Include > `GRP-INTUNE-Windows-Pilot-Devices` | N/A | Pilot devices receive ASR policy |
| 45 | Save ASR policy | Admin workstation | Review + create or Review + save | N/A | ASR policy is active |
| 46 | Open Security baselines | Admin workstation | Intune > Endpoint security > Security baselines | N/A | Security baseline blade opens |
| 47 | Create Defender for Endpoint security baseline if not already created | Admin workstation | Microsoft Defender for Endpoint baseline > Create profile | N/A | MDE baseline wizard opens |
| 48 | Name MDE security baseline | Admin workstation | Name: `INTUNE-MDE-SecurityBaseline-Pilot` | N/A | MDE baseline has clear pilot name |
| 49 | Review MDE baseline settings | Admin workstation | Configuration settings | N/A | Baseline values are reviewed |
| 50 | Avoid duplicating settings already controlled by AV or ASR policies | Admin workstation | Compare baseline settings to Endpoint security policies | N/A | Policy conflicts are avoided |
| 51 | Assign MDE baseline to pilot devices | Admin workstation | Assignments > Include > `GRP-INTUNE-Windows-Pilot-Devices` | N/A | Pilot devices receive MDE security baseline |
| 52 | Review and create MDE baseline | Admin workstation | Review + create > Create | N/A | MDE security baseline is created |
| 53 | Sync pilot Windows device | Windows test device | Settings > Accounts > Access work or school > Info > Sync | N/A | Device checks in with Intune |
| 54 | Trigger sync from Intune | Admin workstation | Devices > All devices > select `WIN11-INTUNE-01` > Sync | `Sync-MgDeviceManagementManagedDevice -ManagedDeviceId <ManagedDeviceId>` | Sync command is queued |
| 55 | Confirm Defender services locally | Windows test device | `sc query Sense` | `Get-Service Sense,WinDefend,WdNisSvc -ErrorAction SilentlyContinue` | Sense and Defender services are present and expected services are running |
| 56 | Confirm onboarding state locally | Windows test device | `reg query "HKLM\SOFTWARE\Microsoft\Windows Advanced Threat Protection\Status"` | `Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows Advanced Threat Protection\Status" -ErrorAction SilentlyContinue` | MDE status registry keys are present |
| 57 | Run local MDE connectivity analyzer if available | Windows test device | Use Microsoft Defender for Endpoint Client Analyzer if staged | N/A | Connectivity and sensor diagnostics are collected |
| 58 | Check Defender sensor health in Defender portal | Admin workstation | Defender portal > Assets > Devices > search `WIN11-INTUNE-01` | N/A | Device appears in Defender device inventory |
| 59 | Review device page in Defender | Admin workstation | Defender portal > Device inventory > select device | N/A | Device risk, exposure level, sensor health, and timeline are visible |
| 60 | Review Intune MDE onboarding status | Admin workstation | Intune > Endpoint security > Endpoint detection and response > policy > Device status | N/A | Device reports policy success or explainable state |
| 61 | Review security recommendations | Admin workstation | Defender portal > Exposure management or Vulnerability management > Recommendations | N/A | Device recommendations are visible if licensed |
| 62 | Review device risk level | Admin workstation | Defender portal > Device inventory > Risk level | N/A | Device risk level is visible |
| 63 | Open Windows compliance policy | Admin workstation | Intune > Devices > Compliance policies | N/A | Compliance policy list opens |
| 64 | Create or edit MDE risk compliance policy | Admin workstation | Create policy > Windows 10 and later or edit existing pilot compliance policy | N/A | Compliance policy wizard/editor opens |
| 65 | Name risk compliance policy | Admin workstation | Name: `INTUNE-WIN-Compliance-MDE-Risk-Pilot` | N/A | Risk compliance policy has clear name |
| 66 | Configure Microsoft Defender for Endpoint machine risk requirement | Admin workstation | Compliance settings > Microsoft Defender for Endpoint > Require device to be at or under machine risk score | N/A | Maximum allowed risk is set to Low or Medium |
| 67 | Configure actions for noncompliance | Admin workstation | Actions for noncompliance | N/A | Device is marked noncompliant after defined grace period |
| 68 | Assign risk compliance policy to pilot devices or users | Admin workstation | Assignments > Include > pilot group | N/A | Pilot scope receives risk-based compliance |
| 69 | Review and create or save compliance policy | Admin workstation | Review + create or Review + save | N/A | Compliance policy uses MDE risk signals |
| 70 | Open Conditional Access policies | Admin workstation | Entra admin center > Protection > Conditional Access > Policies | N/A | CA policy list opens |
| 71 | Create pilot CA policy for compliant device access | Admin workstation | New policy | N/A | CA wizard opens |
| 72 | Name risk-aware CA policy | Admin workstation | Name: `CA-PILOT-Require-Compliant-Device-MDE-Risk` | N/A | CA policy has clear pilot name |
| 73 | Include pilot users | Admin workstation | Users > Include > `GRP-INTUNE-Pilot-Users` | N/A | Pilot users are included |
| 74 | Exclude break-glass accounts | Admin workstation | Users > Exclude > `GRP-CA-Exclude-BreakGlass` | N/A | Emergency accounts are excluded |
| 75 | Select target cloud apps | Admin workstation | Target resources > Cloud apps > Office 365 or selected test app | N/A | CA targets intended pilot app |
| 76 | Configure grant control | Admin workstation | Grant > Require device to be marked as compliant | N/A | Compliance state is required for access |
| 77 | Set CA policy to Report-only first | Admin workstation | Enable policy > Report-only | N/A | Policy evaluates without blocking access |
| 78 | Save CA policy | Admin workstation | Create | N/A | Risk-aware CA policy exists in report-only mode |
| 79 | Test sign-in from pilot device | Windows test device | Browser to `https://portal.office.com` | N/A | Sign-in succeeds and CA result is logged |
| 80 | Review sign-in logs for CA result | Admin workstation | Entra admin center > Sign-in logs > select event > Conditional Access | N/A | CA policy result appears |
| 81 | Move CA policy to On after validation | Admin workstation | Conditional Access policy > Enable policy > On | N/A | Compliance requirement is enforced for pilot users |
| 82 | Document final connector, onboarding, compliance, and CA settings | Admin workstation | Workbook notes / repo documentation | N/A | Configuration is reproducible |

## Validation Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|---:|---|---|---|---|---|
| 1 | Confirm Intune MDE connector is connected | Admin workstation | Intune > Endpoint security > Microsoft Defender for Endpoint | N/A | Connector status shows connected |
| 2 | Confirm Windows device onboarding is enabled | Admin workstation | MDE connector settings | N/A | Windows connection/onboarding option is enabled |
| 3 | Confirm compliance evaluation with MDE is enabled | Admin workstation | MDE connector settings | N/A | Compliance policy evaluation is enabled |
| 4 | Confirm EDR onboarding policy exists | Admin workstation | Endpoint security > Endpoint detection and response | N/A | `INTUNE-WIN-MDE-Onboarding-Pilot` is present |
| 5 | Confirm EDR policy assignment | Admin workstation | EDR policy > Properties > Assignments | N/A | Pilot device group is assigned |
| 6 | Confirm EDR policy device status | Admin workstation | EDR policy > Device status | N/A | Test device shows Succeeded, Pending, or explainable state |
| 7 | Confirm Defender Antivirus policy status | Admin workstation | Endpoint security > Antivirus > policy > Device status | N/A | Test device reports policy status |
| 8 | Confirm ASR policy status | Admin workstation | Endpoint security > Attack surface reduction > policy > Device status | N/A | Test device reports policy status |
| 9 | Confirm MDE baseline status if configured | Admin workstation | Security baselines > MDE baseline > Device status | N/A | Test device reports baseline status |
| 10 | Confirm Sense service exists | Windows test device | `sc query Sense` | `Get-Service Sense -ErrorAction SilentlyContinue` | Sense service is present |
| 11 | Confirm Defender service state | Windows test device | `sc query WinDefend` | `Get-Service WinDefend,WdNisSvc -ErrorAction SilentlyContinue` | Defender services are running or state is explainable |
| 12 | Confirm Defender computer status | Windows test device | N/A | `Get-MpComputerStatus` | Defender status and protection components are visible |
| 13 | Confirm Defender preferences | Windows test device | N/A | `Get-MpPreference` | Defender preferences reflect policy |
| 14 | Confirm MDE onboarding registry status | Windows test device | `reg query "HKLM\SOFTWARE\Microsoft\Windows Advanced Threat Protection\Status"` | `Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows Advanced Threat Protection\Status" -ErrorAction SilentlyContinue` | Onboarding-related values exist |
| 15 | Confirm device appears in Defender portal | Admin workstation | Defender portal > Assets > Devices > search test device | N/A | Device appears in Defender inventory |
| 16 | Confirm sensor health | Admin workstation | Defender portal > Device page | N/A | Sensor health is active or explainable |
| 17 | Confirm device risk level | Admin workstation | Defender portal > Device page > Risk level | N/A | Risk level is visible |
| 18 | Confirm exposure level | Admin workstation | Defender portal > Device page > Exposure level | N/A | Exposure level is visible |
| 19 | Confirm timeline events | Admin workstation | Defender portal > Device page > Timeline | N/A | Device telemetry appears |
| 20 | Confirm security recommendations | Admin workstation | Defender portal > Vulnerability management > Recommendations | N/A | Recommendations are available if licensed |
| 21 | Confirm MDE risk compliance policy exists | Admin workstation | Intune > Devices > Compliance policies | N/A | `INTUNE-WIN-Compliance-MDE-Risk-Pilot` is present |
| 22 | Confirm risk threshold in compliance policy | Admin workstation | Compliance policy > Properties > Compliance settings | N/A | Machine risk threshold is configured |
| 23 | Confirm compliance policy assignment | Admin workstation | Compliance policy > Assignments | N/A | Pilot group is assigned |
| 24 | Confirm device compliance state | Admin workstation | Intune > Devices > All devices > test device > Device compliance | N/A | Device shows compliant or risk-based noncompliance reason |
| 25 | Confirm CA policy exists | Admin workstation | Entra > Conditional Access > Policies | N/A | `CA-PILOT-Require-Compliant-Device-MDE-Risk` is present |
| 26 | Confirm CA exclusions | Admin workstation | CA policy > Users > Exclude | N/A | Break-glass exclusion group is listed |
| 27 | Confirm CA grant control | Admin workstation | CA policy > Grant | N/A | Require compliant device is configured |
| 28 | Confirm CA report-only logs | Admin workstation | Entra > Sign-in logs > Conditional Access tab | N/A | Policy result is visible |
| 29 | Confirm access from compliant low-risk device | Windows test device | Sign in to target Microsoft 365 app | N/A | Access succeeds |
| 30 | Record validation result | Admin workstation | Workbook notes | N/A | MDE integration validation is documented |

## Optional Safe Test Signals

| Test | Device | Command | PowerShell | Expected Result |
|---|---|---|---|---|
| Generate EICAR test file for antivirus validation | Windows test device | Use standard EICAR antivirus test string in a `.txt` file | N/A | Defender Antivirus detects or blocks the test file |
| Trigger Defender quick scan | Windows test device | N/A | `Start-MpScan -ScanType QuickScan` | Quick scan starts |
| Update Defender signatures | Windows test device | N/A | `Update-MpSignature` | Defender signatures update |
| Confirm recent Defender events | Windows test device | Event Viewer > Microsoft > Windows > Windows Defender > Operational | `Get-WinEvent -LogName "Microsoft-Windows-Windows Defender/Operational" -MaxEvents 50` | Defender events are visible |
| Confirm ASR audit events if configured | Windows test device | Defender operational log | `Get-WinEvent -LogName "Microsoft-Windows-Windows Defender/Operational" -MaxEvents 100` | ASR audit events appear if triggered |

## Troubleshooting Checks

| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| Intune connector will not connect to Defender | Licensing, role, or tenant provisioning issue | Intune connector page and Defender endpoint settings | Confirm licensing, admin role, and Defender tenant provisioning |
| MDE onboarding policy not applying | Device not targeted or sync delay | EDR policy assignment and device last check-in | Add device to pilot group and sync |
| Device does not appear in Defender portal | Onboarding failed, sensor unhealthy, or telemetry delay | Sense service, registry status, EDR policy status | Confirm onboarding policy and wait for portal inventory |
| Sense service missing | Unsupported OS, onboarding not applied, or Defender components missing | `Get-Service Sense` | Confirm supported Windows version and reapply onboarding policy |
| Sense service stopped | Sensor issue or device policy problem | Event logs and service state | Restart device, check onboarding, run client analyzer |
| Defender AV policy fails | Third-party AV, policy conflict, or unsupported setting | `Get-MpComputerStatus`, Intune per-setting status | Remove conflict or adjust AV policy |
| Cloud protection not enabled | Policy setting not applied or user tampering allowed | `Get-MpPreference` | Reapply Defender AV policy and block tampering where appropriate |
| ASR rules break apps | Rules moved to Block too early | Defender operational logs and user reports | Move to Audit, add exclusions, then re-test |
| Controlled folder access blocks app writes | CFA enabled without allow list | Defender events and blocked app path | Add controlled folder access allowed app or keep Audit |
| Device risk not visible in Intune compliance | MDE compliance connector setting disabled or delay | Intune MDE connector and compliance policy | Enable compliance evaluation and wait for sync |
| Device marked noncompliant due to risk | MDE machine risk exceeds threshold | Defender device page and compliance details | Investigate alerts, remediate device, or tune threshold |
| Compliance remains stale after remediation | Signal propagation delay | Defender device page and Intune device compliance | Wait for sync, trigger device check-in, confirm alert resolved |
| CA blocks pilot user unexpectedly | Policy enabled too soon or missing exclusion | Entra sign-in logs and CA policy assignments | Set policy to Report-only, add exclusions, fix compliance |
| Break-glass account affected | Exclusion missing | CA policy > Users > Exclude | Add break-glass group to exclusion |
| Defender recommendations missing | Vulnerability management license or data delay | Defender portal vulnerability management | Confirm licensing and device onboarding age |
| Duplicate security settings conflict | Baseline, Endpoint security, and Settings catalog overlap | Intune per-setting conflict report | Choose one policy source per setting |
| GPO overrides Defender settings | Existing domain policy controls Defender | `gpresult /h C:\Temp\gp.html` | Remove or reconcile GPO settings |
| Co-management conflict exists | Configuration Manager owns endpoint security workload | ConfigMgr co-management workload settings | Move workload to Intune or manage in ConfigMgr |
| Network blocks MDE telemetry | Firewall/proxy blocks Defender endpoints | MDE client analyzer and network logs | Allow required Microsoft Defender endpoints |

## Rollback / Cleanup

| Step | Task | Device | Command | PowerShell | Expected Result |
|---:|---|---|---|---|---|
| 1 | Set CA policy back to Report-only | Admin workstation | Entra > Conditional Access > policy > Enable policy > Report-only | N/A | Access enforcement is paused |
| 2 | Disable CA policy if needed | Admin workstation | Conditional Access > policy > Enable policy > Off | N/A | Policy no longer evaluates |
| 3 | Remove pilot assignment from MDE risk compliance policy | Admin workstation | Intune > Compliance policies > policy > Assignments | N/A | Risk compliance no longer targets pilot devices |
| 4 | Remove MDE risk setting from compliance policy if required | Admin workstation | Compliance policy > Properties > Compliance settings | N/A | Compliance no longer depends on MDE risk |
| 5 | Remove pilot assignment from EDR onboarding policy | Admin workstation | Endpoint security > Endpoint detection and response > policy > Assignments | N/A | New onboarding pressure is removed |
| 6 | Create offboarding policy only if formally required | Admin workstation | Endpoint security > Endpoint detection and response > Create offboarding policy if available | N/A | Devices can be offboarded intentionally |
| 7 | Remove pilot assignment from Defender AV policy | Admin workstation | Endpoint security > Antivirus > policy > Assignments | N/A | Defender AV policy no longer targets pilot group |
| 8 | Move ASR rules back to Audit if app impact occurs | Admin workstation | Endpoint security > Attack surface reduction > policy | N/A | ASR enforcement impact is reduced |
| 9 | Remove pilot assignment from ASR policy | Admin workstation | ASR policy > Assignments | N/A | ASR policy no longer targets pilot group |
| 10 | Remove pilot assignment from MDE security baseline | Admin workstation | Security baselines > MDE baseline > Assignments | N/A | MDE baseline no longer targets pilot devices |
| 11 | Sync pilot device after rollback | Windows test device | Settings > Accounts > Access work or school > Info > Sync | `Sync-MgDeviceManagementManagedDevice -ManagedDeviceId <ManagedDeviceId>` | Device checks in after rollback |
| 12 | Validate local Defender state after rollback | Windows test device | `sc query Sense` | `Get-Service Sense,WinDefend,WdNisSvc -ErrorAction SilentlyContinue; Get-MpComputerStatus` | Defender state is known after rollback |
| 13 | Confirm access behavior after rollback | Windows test device | Sign in to Microsoft 365 target app | N/A | Access behavior matches rollback design |
| 14 | Document rollback state | Admin workstation | Workbook notes | N/A | Rollback is recorded |

## Notes

- Defender for Endpoint integration should be piloted before using risk in Conditional Access.
- MDE onboarding, Defender AV policy, ASR policy, and MDE security baseline can overlap. Do not configure the same setting in three different places.
- Start ASR rules in Audit mode. Move to Block only after reviewing impact.
- Device risk is not instant. Defender risk, Intune compliance, and Conditional Access can have propagation delays.
- Conditional Access should use the Intune compliance result rather than trying to directly target raw Defender risk in most baseline designs.
- Break-glass accounts must be excluded from CA enforcement.
- Third-party antivirus can affect Defender Antivirus state, but MDE EDR can still be used depending on mode and licensing.
- Co-managed environments need clear workload ownership. Do not assume Intune endpoint security policy wins if Configuration Manager still owns the workload.
- MDE device inventory is not the same as Intune device inventory. Validate both.
- If a device becomes risky, the fix is not to lower the compliance threshold blindly. Investigate and remediate the Defender alert.

## Related Workbooks

| Workbook                                                                              | Relationship                                                                    |
| ------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------- |
| 01_Configure_Intune_Tenant_Enrollment_Baseline_And_Device_Platform_Restrictions.md    | Provides enrollment foundation before MDE onboarding                            |
| 02_Configure_Device_Compliance_Conditional_Access_And_Enrollment_Status.md            | Provides compliance and CA structure used by MDE risk signals                   |
| 03_Configure_Windows_Update_Rings_Feature_Updates_And_Driver_Update_Policies.md       | Keeps devices patched so Defender risk and exposure stay lower                  |
| 04_Configure_Device_Configuration_Profiles_Security_Baselines_And_Settings_Catalog.md | Provides Defender AV, firewall, ASR, BitLocker, and hardening policy foundation |
| 05_Configure_App_Deployment_Assignment_Detection_And_Remediation.md                   | Deploys supporting tools and remediation scripts for managed endpoints          |
| 07_Troubleshoot_Intune_Enrollment_Compliance_App_Deployment_And_Device_Sync_Issues.md | Troubleshoots MDE onboarding, compliance signal, sync, and policy failures      |