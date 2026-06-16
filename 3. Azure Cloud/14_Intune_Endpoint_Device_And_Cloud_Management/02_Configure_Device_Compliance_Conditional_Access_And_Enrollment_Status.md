# 02_Configure_Device_Compliance_Conditional_Access_And_Enrollment_Status

## Objective

Configure baseline Microsoft Intune device compliance policies, connect compliance state to Microsoft Entra Conditional Access, and validate enrollment status reporting. This workbook builds on the tenant enrollment baseline by enforcing minimum security posture before managed devices are trusted for access.

## Lab Context

This workbook assumes Intune enrollment has already been prepared and pilot devices can appear in Intune. The goal is to create controlled compliance policies first, assign them to pilot users/devices, review enrollment status, and then connect compliance state to Conditional Access without locking out administrators or emergency access accounts.

## Prerequisites

| Requirement | Details |
|---|---|
| Admin role | Intune Administrator, Conditional Access Administrator, Security Administrator, or Global Administrator |
| Portal access | Microsoft Intune admin center and Microsoft Entra admin center |
| Licensing | Microsoft Intune plus Microsoft Entra ID P1 or higher for Conditional Access |
| Existing baseline | Task 01 Intune tenant enrollment baseline should be complete |
| Pilot users | At least one test user licensed for Intune |
| Pilot devices | At least one enrolled Windows 10/11 device preferred |
| Pilot groups | Pilot user group and pilot device group |
| Emergency access | Break-glass accounts must be excluded from enforcement policies |
| Test app | Microsoft 365 cloud app or Office 365 cloud app target for pilot Conditional Access testing |

## Naming / Variables

| Variable | Example Value | Purpose |
|---|---|---|
| Pilot user group | GRP-INTUNE-Pilot-Users | Users targeted for compliance and CA testing |
| Pilot device group | GRP-INTUNE-Pilot-Devices | Devices used for pilot validation |
| Exclusion group | GRP-CA-Exclude-BreakGlass | Emergency accounts excluded from CA |
| Windows compliance policy | INTUNE-WIN-Compliance-Baseline | Windows compliance rule set |
| macOS compliance policy | INTUNE-macOS-Compliance-Baseline | macOS compliance rule set |
| iOS compliance policy | INTUNE-iOS-Compliance-Baseline | iOS/iPadOS compliance rule set |
| Android compliance policy | INTUNE-Android-Compliance-Baseline | Android compliance rule set |
| Conditional Access policy | CA-PILOT-Require-Compliant-Device-M365 | Pilot CA policy requiring compliant devices |
| Grace period | 1 day | Delay before marking new devices noncompliant |
| Test user | user01@contoso.com | Pilot sign-in validation account |
| Test device | WIN11-INTUNE-01 | Enrolled validation device |

## Configuration Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|---:|---|---|---|---|---|
| 1 | Sign in to Microsoft Intune admin center | Admin workstation | Browser: `https://intune.microsoft.com` | N/A | Intune admin center opens successfully |
| 2 | Confirm enrolled pilot device exists | Admin workstation | Devices > All devices | Optional Graph PowerShell: `Get-MgDeviceManagementManagedDevice -Filter "contains(deviceName,'WIN11-INTUNE-01')"` | Pilot device appears in Intune |
| 3 | Confirm pilot user is licensed | Admin workstation | Microsoft 365 admin center > Users > Licenses and apps | `Get-MgUserLicenseDetail -UserId user01@contoso.com` | User has Intune-capable license |
| 4 | Confirm pilot user group exists | Admin workstation | Intune admin center > Groups | Optional: `Get-MgGroup -Filter "displayName eq 'GRP-INTUNE-Pilot-Users'"` | Pilot user group is available |
| 5 | Confirm break-glass exclusion group exists | Admin workstation | Entra admin center > Groups | Optional: `Get-MgGroup -Filter "displayName eq 'GRP-CA-Exclude-BreakGlass'"` | Exclusion group is available |
| 6 | Add emergency accounts to exclusion group | Admin workstation | Entra admin center > Groups > GRP-CA-Exclude-BreakGlass > Members | `New-MgGroupMember -GroupId <GroupId> -DirectoryObjectId <BreakGlassUserId>` | Emergency accounts are excluded from CA enforcement |
| 7 | Open compliance policy area | Admin workstation | Intune admin center > Devices > Compliance policies | N/A | Compliance policy blade opens |
| 8 | Review compliance policy settings | Admin workstation | Devices > Compliance policies > Compliance policy settings | N/A | Tenant-wide compliance behavior is visible |
| 9 | Configure mark devices with no compliance policy | Admin workstation | Compliance policy settings > Mark devices with no compliance policy assigned | N/A | Setting is intentionally configured, preferably Not compliant for stronger posture after pilot planning |
| 10 | Configure enhanced jailbreak/root detection if available | Admin workstation | Compliance policy settings | N/A | Mobile compliance detection behavior is reviewed |
| 11 | Create Windows compliance policy | Admin workstation | Devices > Compliance policies > Create policy > Platform: Windows 10 and later | N/A | Windows compliance wizard opens |
| 12 | Name Windows compliance policy | Admin workstation | Name: `INTUNE-WIN-Compliance-Baseline` | N/A | Policy has clear workbook-aligned name |
| 13 | Configure Windows device health requirements | Admin workstation | Compliance settings > Device Health | N/A | Require settings such as BitLocker, Secure Boot, and code integrity where supported |
| 14 | Configure Windows minimum OS version | Admin workstation | Compliance settings > Device Properties | N/A | Minimum supported Windows version is defined |
| 15 | Configure Windows system security password requirements | Admin workstation | Compliance settings > System Security | N/A | Password or local security requirements are defined |
| 16 | Configure Windows Defender security requirements | Admin workstation | Compliance settings > Microsoft Defender for Endpoint / Security | N/A | Antivirus, antispyware, real-time protection, or risk rules are configured as available |
| 17 | Configure actions for noncompliance | Admin workstation | Actions for noncompliance | N/A | Mark device noncompliant after defined grace period |
| 18 | Assign Windows compliance policy | Admin workstation | Assignments > Add groups > `GRP-INTUNE-Pilot-Users` or pilot device group | N/A | Pilot scope receives Windows compliance policy |
| 19 | Create iOS/iPadOS compliance policy if required | Admin workstation | Create policy > Platform: iOS/iPadOS | N/A | iOS/iPadOS compliance wizard opens |
| 20 | Configure iOS/iPadOS baseline requirements | Admin workstation | Device properties, system security, device health | N/A | Minimum OS, passcode, jailbreak detection, and threat level settings are defined |
| 21 | Assign iOS/iPadOS compliance policy | Admin workstation | Assignments > Add groups > `GRP-INTUNE-Pilot-Users` | N/A | Pilot users receive iOS/iPadOS compliance policy |
| 22 | Create Android compliance policy if required | Admin workstation | Create policy > Platform: Android Enterprise | N/A | Android compliance wizard opens |
| 23 | Configure Android baseline requirements | Admin workstation | Device health, device properties, system security | N/A | Root detection, minimum OS, passcode, encryption, and threat level settings are defined |
| 24 | Assign Android compliance policy | Admin workstation | Assignments > Add groups > `GRP-INTUNE-Pilot-Users` | N/A | Pilot users receive Android compliance policy |
| 25 | Create macOS compliance policy if required | Admin workstation | Create policy > Platform: macOS | N/A | macOS compliance wizard opens |
| 26 | Configure macOS baseline requirements | Admin workstation | Device properties, system security | N/A | Minimum OS, password, firewall, encryption, and gatekeeper settings are defined where available |
| 27 | Assign macOS compliance policy | Admin workstation | Assignments > Add groups > `GRP-INTUNE-Pilot-Users` | N/A | Pilot users receive macOS compliance policy |
| 28 | Review compliance assignment targeting | Admin workstation | Devices > Compliance policies > each policy > Properties | N/A | No accidental all-user assignment exists |
| 29 | Sync pilot Windows device | Windows test device | Settings > Accounts > Access work or school > Info > Sync | N/A | Device checks in with Intune |
| 30 | Trigger sync from Intune if available | Admin workstation | Devices > All devices > select pilot device > Sync | Optional: `Sync-MgDeviceManagementManagedDevice -ManagedDeviceId <ManagedDeviceId>` | Sync command is queued |
| 31 | Review device compliance state | Admin workstation | Devices > All devices > select device > Device compliance | N/A | Device compliance policy state appears |
| 32 | Review compliance policy report | Admin workstation | Devices > Compliance policies > Monitor > Device compliance | N/A | Policy reports show targeted device status |
| 33 | Open Microsoft Entra admin center | Admin workstation | Browser: `https://entra.microsoft.com` | N/A | Entra admin center opens |
| 34 | Open Conditional Access policies | Admin workstation | Protection > Conditional Access > Policies | N/A | Conditional Access policy list opens |
| 35 | Create pilot Conditional Access policy | Admin workstation | New policy | N/A | New policy wizard opens |
| 36 | Name CA policy | Admin workstation | Name: `CA-PILOT-Require-Compliant-Device-M365` | N/A | CA policy has clear pilot name |
| 37 | Assign CA policy to pilot users | Admin workstation | Users > Include > Select users and groups > `GRP-INTUNE-Pilot-Users` | N/A | Only pilot users are included |
| 38 | Exclude break-glass accounts | Admin workstation | Users > Exclude > `GRP-CA-Exclude-BreakGlass` | N/A | Emergency accounts are excluded |
| 39 | Select target cloud apps | Admin workstation | Target resources > Cloud apps > Office 365 or selected test app | N/A | CA targets intended pilot cloud app |
| 40 | Configure conditions if needed | Admin workstation | Conditions > Client apps / Device platforms / Locations | N/A | Conditions are intentionally scoped or left broad for pilot |
| 41 | Configure grant control | Admin workstation | Grant > Require device to be marked as compliant | N/A | Access requires compliant device |
| 42 | Set policy to Report-only first | Admin workstation | Enable policy > Report-only | N/A | Policy evaluates without blocking access |
| 43 | Save Conditional Access policy | Admin workstation | Create | N/A | CA policy exists in report-only mode |
| 44 | Test sign-in from compliant pilot device | Windows test device | Browser to `https://portal.office.com` | N/A | Sign-in succeeds and CA report-only logs are generated |
| 45 | Test sign-in from unmanaged device if safe | Unmanaged test device | Browser to target app as pilot user | N/A | Report-only CA result indicates access would be blocked or require compliance |
| 46 | Review sign-in logs | Admin workstation | Entra admin center > Monitoring > Sign-in logs | N/A | CA policy result appears for pilot sign-ins |
| 47 | Review Conditional Access insights | Admin workstation | Entra admin center > Conditional Access > Insights and reporting | N/A | Report-only impact is visible |
| 48 | Move CA policy to On after validation | Admin workstation | Conditional Access policy > Enable policy > On | N/A | Compliant device requirement is enforced for pilot users |
| 49 | Document final compliance and CA scope | Admin workstation | Workbook notes / repo documentation | N/A | Configuration is reproducible and auditable |

## Validation Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|---:|---|---|---|---|---|
| 1 | Confirm Windows compliance policy exists | Admin workstation | Intune > Devices > Compliance policies | N/A | `INTUNE-WIN-Compliance-Baseline` is present |
| 2 | Confirm Windows compliance assignment | Admin workstation | Policy > Properties > Assignments | N/A | Pilot group is assigned |
| 3 | Confirm mobile/macOS compliance policies exist if configured | Admin workstation | Compliance policies list | N/A | Platform-specific policies are present |
| 4 | Confirm no accidental all-user targeting | Admin workstation | Each compliance policy > Assignments | N/A | Only intended pilot groups are assigned |
| 5 | Confirm pilot device synced recently | Admin workstation | Devices > All devices > pilot device > Overview | N/A | Last check-in time is recent |
| 6 | Confirm device compliance state | Admin workstation | Devices > All devices > pilot device | `Get-MgDeviceManagementManagedDevice -ManagedDeviceId <ManagedDeviceId>` | Device shows Compliant or expected Noncompliant reason |
| 7 | Confirm compliance details | Admin workstation | Device > Device compliance | N/A | Policy setting-by-setting result is visible |
| 8 | Confirm noncompliance action status | Admin workstation | Device compliance report | N/A | Grace period and action behavior are visible |
| 9 | Confirm CA policy exists | Admin workstation | Entra admin center > Conditional Access > Policies | N/A | `CA-PILOT-Require-Compliant-Device-M365` is present |
| 10 | Confirm CA policy includes only pilot users | Admin workstation | CA policy > Users | N/A | Pilot group included |
| 11 | Confirm break-glass exclusion | Admin workstation | CA policy > Users > Exclude | N/A | Break-glass exclusion group is listed |
| 12 | Confirm grant control | Admin workstation | CA policy > Grant | N/A | Require device to be marked as compliant is enabled |
| 13 | Confirm policy mode | Admin workstation | CA policy list | N/A | Policy is Report-only during test or On after approval |
| 14 | Test compliant device access | Windows test device | Sign in to Microsoft 365 target app | N/A | Access succeeds |
| 15 | Test unmanaged device behavior | Unmanaged test device | Sign in as pilot user to target app | N/A | Access is blocked after enforcement or logged as blocked in report-only |
| 16 | Review Entra sign-in logs | Admin workstation | Entra > Sign-in logs > select event > Conditional Access | N/A | CA result matches expected policy behavior |
| 17 | Review Intune compliance reports | Admin workstation | Intune > Reports > Device compliance | N/A | Compliance reporting reflects device state |
| 18 | Confirm enrollment status for pilot device | Admin workstation | Intune > Devices > Monitor > Enrollment | N/A | Enrollment status is successful or errors are explainable |
| 19 | Confirm user device list | Admin workstation | Entra > Users > pilot user > Devices | N/A | Device is associated with pilot user |
| 20 | Record pass/fail results | Admin workstation | Workbook notes | N/A | Baseline validation is documented |

## Troubleshooting Checks

| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| Device shows Not compliant immediately | Compliance grace period too strict or device missing required setting | Intune > Device > Device compliance | Adjust policy or remediate device setting |
| Device shows Not evaluated | Device has not checked in or policy assignment has not applied | Device last check-in and policy assignment | Sync device and wait for policy evaluation |
| Device has no compliance policy | Assignment group mismatch | Compliance policy > Assignments | Assign correct pilot user or device group |
| User is blocked unexpectedly | Conditional Access policy enabled too broadly | Entra > Conditional Access > Policy assignments | Limit to pilot group and exclude emergency accounts |
| Break-glass account is affected | Exclusion missing or wrong group | CA policy > Users > Exclude | Add emergency account or group to exclusion |
| Compliant device is still blocked | Device not recognized as compliant in Entra token claims | Sign-in logs > Conditional Access details | Re-sync device, re-authenticate, verify device registration |
| Device appears compliant in Intune but not in Entra | Device state synchronization delay | Entra device object and Intune managed device object | Wait for propagation, verify device join/registration state |
| Windows device fails BitLocker requirement | BitLocker disabled or recovery state not escrowed | Device > Encryption report / Compliance details | Enable BitLocker and confirm recovery key escrow |
| Secure Boot requirement fails | Secure Boot disabled in firmware | Device compliance details and BIOS/UEFI | Enable Secure Boot if hardware supports it |
| Defender requirement fails | Defender disabled, third-party AV conflict, stale definitions | Windows Security and Intune compliance details | Enable Defender components or adjust policy intentionally |
| Minimum OS requirement fails | Device OS version below baseline | Device hardware properties | Update OS or lower minimum only if justified |
| CA report-only logs do not appear | Wrong app targeted or no test sign-in occurred | Entra sign-in logs and CA insights | Test target app again with pilot user |
| Enrollment status shows failure | Enrollment restriction, license, MDM scope, or platform issue | Intune > Devices > Monitor > Enrollment failures | Fix underlying enrollment baseline issue |
| Noncompliant device still accesses app | CA policy still in report-only or wrong cloud app selected | CA policy mode and target resources | Enable policy On and verify app targeting |

## Rollback / Cleanup

| Step | Task | Device | Command | PowerShell | Expected Result |
|---:|---|---|---|---|---|
| 1 | Set CA policy back to Report-only | Admin workstation | Entra > Conditional Access > policy > Enable policy > Report-only | N/A | Access enforcement is paused but logging continues |
| 2 | Disable pilot CA policy if needed | Admin workstation | Conditional Access > policy > Enable policy > Off | N/A | Policy no longer evaluates |
| 3 | Remove pilot group from CA policy | Admin workstation | CA policy > Users > Include | N/A | Pilot users are no longer targeted |
| 4 | Remove pilot assignments from Windows compliance policy | Admin workstation | Intune > Compliance policy > Assignments | N/A | Policy no longer targets pilot users/devices |
| 5 | Remove pilot assignments from mobile/macOS policies | Admin workstation | Intune > Compliance policies > Assignments | N/A | Platform policies no longer target pilot users |
| 6 | Delete test compliance policies if no longer needed | Admin workstation | Compliance policies > select policy > Delete | N/A | Test compliance policies are removed |
| 7 | Reset mark devices with no compliance policy if changed for test | Admin workstation | Compliance policy settings | N/A | Tenant-wide compliance behavior returns to intended state |
| 8 | Sync pilot device after rollback | Windows test device | Settings > Accounts > Access work or school > Info > Sync | N/A | Device receives updated policy state |
| 9 | Retest access as pilot user | Windows test device | Sign in to target cloud app | N/A | Access behavior reflects rollback |
| 10 | Document rollback state | Admin workstation | Workbook notes | N/A | Rollback is recorded |

## Notes

- Conditional Access should be staged in Report-only before enforcement.
- Never test compliance-based Conditional Access without excluding emergency accounts.
- Compliance policies define whether a device is trusted enough for access. Configuration profiles and security baselines actually apply many of the settings that make a device compliant.
- A device can be enrolled but not compliant. Enrollment alone should not be treated as trust.
- User targeting is usually cleaner for compliance policy assignment in small pilots, but device groups may be useful for lab-specific targeting.
- Device compliance state can take time to propagate into Conditional Access decisions.
- If you enforce compliance before update rings, security baselines, and configuration profiles exist, you may create avoidable failures.
- Treat Report-only logs as mandatory evidence before turning a CA policy On.

## Related Workbooks

| Workbook                                                                              | Relationship                                                                  |
| ------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------- |
| 01_Configure_Intune_Tenant_Enrollment_Baseline_And_Device_Platform_Restrictions.md    | Provides the enrollment foundation required before compliance policies matter |
| 03_Configure_Windows_Update_Rings_Feature_Updates_And_Driver_Update_Policies.md       | Helps devices meet OS version and update compliance requirements              |
| 04_Configure_Device_Configuration_Profiles_Security_Baselines_And_Settings_Catalog.md | Applies the device settings that support compliance success                   |
| 05_Configure_App_Deployment_Assignment_Detection_And_Remediation.md                   | Deploys apps after device compliance and targeting are stable                 |
| 06_Configure_Defender_For_Endpoint_Integration_And_Device_Risk_Signals.md             | Adds Defender risk-based compliance and CA signals                            |
| 07_Troubleshoot_Intune_Enrollment_Compliance_App_Deployment_And_Device_Sync_Issues.md | Troubleshoots enrollment, compliance, CA, and sync failures                   |