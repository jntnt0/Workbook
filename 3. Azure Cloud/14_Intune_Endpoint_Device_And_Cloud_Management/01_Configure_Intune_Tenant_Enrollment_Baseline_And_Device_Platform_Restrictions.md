# 01_Configure_Intune_Tenant_Enrollment_Baseline_And_Device_Platform_Restrictions

## Objective

Configure a baseline Microsoft Intune tenant enrollment posture for a lab or small enterprise tenant. This workbook establishes tenant enrollment readiness, MDM authority validation, automatic enrollment, device platform restrictions, enrollment limit controls, enrollment notifications, and baseline validation checks.

## Lab Context

This workbook assumes a Microsoft Intune tenant integrated with Microsoft Entra ID and Microsoft 365. The goal is not to enroll every device type yet. The goal is to prepare the tenant so Windows, iOS/iPadOS, Android, and macOS enrollment behavior is controlled before production devices are allowed in.

## Prerequisites

| Requirement | Details |
|---|---|
| Admin role | Intune Administrator, Global Administrator, or a custom role with enrollment and device configuration permissions |
| Portal access | Microsoft Intune admin center |
| Entra access | Microsoft Entra admin center |
| Licensing | Microsoft Intune, Enterprise Mobility + Security, Microsoft 365 Business Premium, or Microsoft 365 E3/E5 licensing |
| Test users | At least one pilot user and one break-glass/admin account |
| Test groups | Pilot user group and pilot device group |
| Test devices | At least one Windows 10/11 test device preferred |
| Naming convention | Use clear prefix such as LAB, PILOT, INTUNE, or CORP |

## Naming / Variables

| Variable | Example Value | Purpose |
|---|---|---|
| Tenant name | contoso.onmicrosoft.com | Tenant identity |
| Pilot user group | GRP-INTUNE-Pilot-Users | Users allowed to test enrollment |
| Pilot device group | GRP-INTUNE-Pilot-Devices | Devices used for testing |
| Windows restriction policy | INTUNE-WIN-Enrollment-Restriction-Baseline | Windows platform enrollment baseline |
| iOS/iPadOS restriction policy | INTUNE-iOS-Enrollment-Restriction-Baseline | Apple mobile enrollment baseline |
| Android restriction policy | INTUNE-Android-Enrollment-Restriction-Baseline | Android enrollment baseline |
| macOS restriction policy | INTUNE-macOS-Enrollment-Restriction-Baseline | macOS enrollment baseline |
| Enrollment limit policy | INTUNE-Device-Limit-Restriction-Baseline | Maximum device count baseline |
| Enrollment notification | INTUNE-Enrollment-Notification-Pilot | Pilot enrollment notification |

## Configuration Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|---:|---|---|---|---|---|
| 1 | Sign in to the Microsoft Intune admin center | Admin workstation | Browser: `https://intune.microsoft.com` | N/A | Intune admin center opens successfully |
| 2 | Confirm Intune tenant administration blade is available | Admin workstation | Intune admin center > Tenant administration | N/A | Tenant administration options are visible |
| 3 | Confirm MDM authority is set to Microsoft Intune | Admin workstation | Tenant administration > Tenant status | N/A | MDM authority shows Microsoft Intune |
| 4 | Review tenant status health | Admin workstation | Tenant administration > Tenant status > Service health and message center | N/A | No tenant-wide Intune outage blocks configuration |
| 5 | Confirm administrator account permissions | Admin workstation | Tenant administration > Roles > All roles | N/A | Admin account has rights to configure enrollment |
| 6 | Create or verify pilot user group | Admin workstation | Intune admin center > Groups > New group | Optional Graph PowerShell: `New-MgGroup -DisplayName "GRP-INTUNE-Pilot-Users" -MailEnabled:$false -MailNickname "GRP-INTUNE-Pilot-Users" -SecurityEnabled:$true` | Pilot security group exists |
| 7 | Create or verify pilot device group | Admin workstation | Intune admin center > Groups > New group | Optional Graph PowerShell: `New-MgGroup -DisplayName "GRP-INTUNE-Pilot-Devices" -MailEnabled:$false -MailNickname "GRP-INTUNE-Pilot-Devices" -SecurityEnabled:$true` | Pilot device group exists |
| 8 | Add pilot users to pilot group | Admin workstation | Groups > GRP-INTUNE-Pilot-Users > Members > Add members | Optional Graph PowerShell after collecting IDs: `New-MgGroupMember -GroupId <GroupId> -DirectoryObjectId <UserId>` | Pilot users are members |
| 9 | Review automatic MDM enrollment settings | Admin workstation | Entra admin center > Identity > Mobility > Microsoft Intune | N/A | MDM user scope and URLs are visible |
| 10 | Set MDM user scope to pilot group first | Admin workstation | Mobility > Microsoft Intune > MDM user scope > Some > select `GRP-INTUNE-Pilot-Users` | N/A | Only pilot users are targeted for automatic MDM enrollment |
| 11 | Confirm MDM discovery URL | Admin workstation | Mobility > Microsoft Intune | N/A | MDM discovery URL is populated |
| 12 | Confirm MDM terms of use URL if required | Admin workstation | Mobility > Microsoft Intune | N/A | Terms of use URL is populated or intentionally blank |
| 13 | Confirm MDM compliance URL | Admin workstation | Mobility > Microsoft Intune | N/A | Compliance URL is populated |
| 14 | Open Intune enrollment platform restrictions | Admin workstation | Intune admin center > Devices > Enrollment > Device platform restrictions | N/A | Platform restriction policies are visible |
| 15 | Review default platform restrictions before editing | Admin workstation | Device platform restrictions > All users default policies | N/A | Current defaults are known before changes |
| 16 | Create Windows enrollment restriction policy | Admin workstation | Devices > Enrollment > Device platform restrictions > Create restriction | N/A | Windows restriction policy creation wizard opens |
| 17 | Configure Windows platform allowance | Admin workstation | Platform: Windows; Allow personally owned: based on lab target; Minimum version: define supported baseline | N/A | Windows enrollment behavior is explicitly controlled |
| 18 | Assign Windows restriction to pilot group | Admin workstation | Assignments > Add groups > `GRP-INTUNE-Pilot-Users` | N/A | Windows restriction targets pilot users |
| 19 | Create iOS/iPadOS enrollment restriction policy | Admin workstation | Device platform restrictions > Create restriction | N/A | iOS/iPadOS policy creation wizard opens |
| 20 | Configure iOS/iPadOS platform allowance | Admin workstation | Platform: iOS/iPadOS; Block personally owned if corporate-only baseline is desired | N/A | Apple mobile enrollment is intentionally allowed or blocked |
| 21 | Assign iOS/iPadOS restriction to pilot group | Admin workstation | Assignments > Add groups > `GRP-INTUNE-Pilot-Users` | N/A | iOS/iPadOS restriction targets pilot users |
| 22 | Create Android enrollment restriction policy | Admin workstation | Device platform restrictions > Create restriction | N/A | Android policy creation wizard opens |
| 23 | Configure Android platform allowance | Admin workstation | Platform: Android device administrator / Android Enterprise as required | N/A | Android enrollment is intentionally allowed or blocked |
| 24 | Assign Android restriction to pilot group | Admin workstation | Assignments > Add groups > `GRP-INTUNE-Pilot-Users` | N/A | Android restriction targets pilot users |
| 25 | Create macOS enrollment restriction policy | Admin workstation | Device platform restrictions > Create restriction | N/A | macOS policy creation wizard opens |
| 26 | Configure macOS platform allowance | Admin workstation | Platform: macOS; define allowed ownership and minimum version | N/A | macOS enrollment behavior is explicitly controlled |
| 27 | Assign macOS restriction to pilot group | Admin workstation | Assignments > Add groups > `GRP-INTUNE-Pilot-Users` | N/A | macOS restriction targets pilot users |
| 28 | Configure device type restriction priority | Admin workstation | Device platform restrictions > Reorder priority | N/A | Pilot restriction policies are evaluated before broad defaults |
| 29 | Open enrollment device limit restrictions | Admin workstation | Devices > Enrollment > Enrollment device limit restrictions | N/A | Device limit policies are visible |
| 30 | Create device limit restriction policy | Admin workstation | Create restriction > Name: `INTUNE-Device-Limit-Restriction-Baseline` | N/A | Device limit policy exists |
| 31 | Set device limit for pilot users | Admin workstation | Configure limit such as 3 to 5 devices per user | N/A | Users cannot enroll unlimited devices |
| 32 | Assign device limit policy to pilot group | Admin workstation | Assignments > Add groups > `GRP-INTUNE-Pilot-Users` | N/A | Pilot users receive the device limit |
| 33 | Confirm device limit restriction priority | Admin workstation | Enrollment device limit restrictions > Reorder priority | N/A | Pilot device limit policy has correct priority |
| 34 | Open enrollment notifications | Admin workstation | Devices > Enrollment > Enrollment notifications | N/A | Enrollment notification configuration is visible |
| 35 | Create enrollment notification for pilot users | Admin workstation | Create notification > Name: `INTUNE-Enrollment-Notification-Pilot` | N/A | Notification policy exists |
| 36 | Configure notification message | Admin workstation | Add helpdesk or admin contact message | N/A | Users receive clear post-enrollment guidance |
| 37 | Assign enrollment notification to pilot group | Admin workstation | Assignments > Add groups > `GRP-INTUNE-Pilot-Users` | N/A | Pilot users receive enrollment notification |
| 38 | Review device cleanup rules | Admin workstation | Devices > Device cleanup rules | N/A | Cleanup behavior is reviewed before enabling |
| 39 | Leave cleanup disabled unless required | Admin workstation | Device cleanup rules | N/A | Devices are not unexpectedly removed during early testing |
| 40 | Review enrollment failure report area | Admin workstation | Devices > Monitor > Enrollment failures | N/A | Admin knows where failed enrollments will appear |
| 41 | Review all enrollment policies for scope overlap | Admin workstation | Devices > Enrollment | N/A | No accidental all-user production assignment exists |
| 42 | Document policy names and assignments | Admin workstation | Internal notes / Obsidian / repo workbook | N/A | Baseline configuration is reproducible |

## Validation Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|---:|---|---|---|---|---|
| 1 | Confirm MDM user scope targets pilot group | Admin workstation | Entra admin center > Mobility > Microsoft Intune | N/A | MDM user scope is set to Some and pilot group is selected |
| 2 | Confirm pilot user is licensed | Admin workstation | Microsoft 365 admin center > Users > Active users > Licenses and apps | Optional Graph PowerShell: `Get-MgUserLicenseDetail -UserId <UserPrincipalName>` | Pilot user has Intune-capable license |
| 3 | Confirm Windows platform restriction exists | Admin workstation | Intune admin center > Devices > Enrollment > Device platform restrictions | N/A | Windows restriction policy is present |
| 4 | Confirm iOS/iPadOS platform restriction exists | Admin workstation | Device platform restrictions | N/A | iOS/iPadOS restriction policy is present |
| 5 | Confirm Android platform restriction exists | Admin workstation | Device platform restrictions | N/A | Android restriction policy is present |
| 6 | Confirm macOS platform restriction exists | Admin workstation | Device platform restrictions | N/A | macOS restriction policy is present |
| 7 | Confirm platform restriction assignments | Admin workstation | Each restriction policy > Assignments | N/A | Each baseline policy targets the pilot user group |
| 8 | Confirm restriction priority | Admin workstation | Device platform restrictions > Priority order | N/A | Pilot policies have intended priority |
| 9 | Confirm device limit policy exists | Admin workstation | Enrollment device limit restrictions | N/A | Device limit baseline exists |
| 10 | Confirm device limit policy assignment | Admin workstation | Device limit policy > Assignments | N/A | Device limit targets pilot user group |
| 11 | Confirm notification policy exists | Admin workstation | Enrollment notifications | N/A | Enrollment notification exists |
| 12 | Confirm notification assignment | Admin workstation | Enrollment notification > Assignments | N/A | Notification targets pilot user group |
| 13 | Test Windows enrollment as pilot user | Windows test device | Settings > Accounts > Access work or school > Connect | N/A | Device begins Entra/Intune enrollment |
| 14 | Confirm Windows device appears in Intune | Admin workstation | Devices > All devices | Optional Graph PowerShell: `Get-MgDeviceManagementManagedDevice -Filter "contains(deviceName,'<DeviceName>')"` | Test device appears as managed |
| 15 | Confirm device ownership | Admin workstation | Devices > All devices > select device > Properties | N/A | Ownership matches expected baseline |
| 16 | Confirm primary user | Admin workstation | Device record > Properties | N/A | Primary user shows the pilot user |
| 17 | Confirm enrollment profile details | Admin workstation | Device record > Hardware / Overview | N/A | Enrollment type and platform are populated |
| 18 | Confirm no unexpected blocked platforms | Admin workstation | Devices > Monitor > Enrollment failures | N/A | No unexpected policy failures appear |
| 19 | Confirm expected block behavior using non-pilot user if safe | Test device | Attempt enrollment with non-pilot user | N/A | Non-pilot user is blocked or not auto-enrolled based on scope |
| 20 | Record validation result | Admin workstation | Workbook notes | N/A | Baseline has pass/fail evidence |

## Troubleshooting Checks

| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| User cannot enroll any device | User is not in MDM user scope | Entra admin center > Mobility > Microsoft Intune | Add user to `GRP-INTUNE-Pilot-Users` or change MDM scope |
| User cannot enroll despite group membership | User lacks Intune license | Microsoft 365 admin center > Licenses and apps | Assign Intune-capable license |
| Device enrollment blocked by platform restriction | Platform is blocked or minimum version too high | Intune > Devices > Enrollment > Device platform restrictions | Adjust restriction policy |
| Personal Windows device is blocked | Personally owned enrollment is disabled | Windows enrollment restriction policy | Allow personal ownership only if policy permits |
| iOS or Android enrollment blocked unexpectedly | Platform-specific restriction has higher priority | Device platform restrictions > Priority | Reorder policies |
| Device limit reached | User already enrolled maximum allowed devices | Enrollment device limit restrictions and user device list | Retire stale devices or raise pilot limit |
| Enrollment succeeds but device does not appear quickly | Sync delay | Devices > All devices and Enrollment failures | Wait, then trigger device sync if available |
| Device appears as Entra registered but not Intune managed | MDM auto-enrollment did not trigger | Entra Mobility settings and license | Fix MDM scope, license, or re-enroll |
| User receives no notification | Notification not assigned or platform not supported for the scenario | Enrollment notifications > Assignments | Assign notification to pilot group |
| Policy changed but behavior did not update immediately | Policy propagation delay | Intune admin center monitoring | Wait for propagation and retry |
| Wrong policy applies to user | Restriction priority conflict | Restriction priority order | Move intended policy above default/broader policy |
| Enrollment failure report is empty but user reports failure | Failure happened before Intune enrollment handoff | Check Entra sign-in logs and device registration state | Validate user auth, MFA, CA, and device registration |

## Rollback / Cleanup

| Step | Task | Device | Command | PowerShell | Expected Result |
|---:|---|---|---|---|---|
| 1 | Remove pilot user from MDM scope if rollback is required | Admin workstation | Entra admin center > Mobility > Microsoft Intune > MDM user scope | N/A | User no longer receives automatic MDM enrollment |
| 2 | Remove pilot assignments from platform restrictions | Admin workstation | Intune > Devices > Enrollment > Device platform restrictions > Assignments | N/A | Restriction policies no longer target pilot group |
| 3 | Delete test platform restriction policies if no longer needed | Admin workstation | Device platform restrictions > select policy > Delete | N/A | Test restrictions are removed |
| 4 | Remove pilot assignments from device limit policy | Admin workstation | Enrollment device limit restrictions > policy > Assignments | N/A | Device limit no longer targets pilot group |
| 5 | Delete device limit policy if no longer needed | Admin workstation | Enrollment device limit restrictions > select policy > Delete | N/A | Test device limit policy is removed |
| 6 | Remove enrollment notification assignment | Admin workstation | Enrollment notifications > policy > Assignments | N/A | Pilot users no longer receive notification |
| 7 | Delete enrollment notification if no longer needed | Admin workstation | Enrollment notifications > select notification > Delete | N/A | Notification policy is removed |
| 8 | Retire or delete test enrolled devices | Admin workstation | Devices > All devices > select test device > Retire or Delete | Optional Graph PowerShell: `Remove-MgDeviceManagementManagedDevice -ManagedDeviceId <ManagedDeviceId>` | Test device records are cleaned up |
| 9 | Remove pilot groups if created only for test | Admin workstation | Entra admin center > Groups | Optional Graph PowerShell: `Remove-MgGroup -GroupId <GroupId>` | Test groups are removed |
| 10 | Document rollback state | Admin workstation | Workbook notes | N/A | Tenant returns to pre-test enrollment posture |

## Notes

- Do not enable broad all-user enrollment until pilot enrollment behavior is verified.
- Device platform restrictions should be treated as guardrails. They prevent unmanaged sprawl before compliance and configuration baselines are mature.
- Keep break-glass accounts out of aggressive enrollment, Conditional Access, and device restriction experiments unless a formal emergency access design is already in place.
- Start with a pilot user group. Expand to departments or all users only after validation.
- Platform restriction priority matters. A broad default policy can override your intended behavior if ordering is wrong.
- Device limit restrictions are useful for stopping uncontrolled personal-device enrollment.
- Enrollment baseline should be completed before compliance policy and Conditional Access enforcement are turned on.

## Related Workbooks

| Workbook                                                                              | Relationship                                                            |
| ------------------------------------------------------------------------------------- | ----------------------------------------------------------------------- |
| 02_Configure_Device_Compliance_Conditional_Access_And_Enrollment_Status.md            | Builds compliance and CA enforcement on top of this enrollment baseline |
| 03_Configure_Windows_Update_Rings_Feature_Updates_And_Driver_Update_Policies.md       | Applies Windows servicing policy after devices can enroll               |
| 04_Configure_Device_Configuration_Profiles_Security_Baselines_And_Settings_Catalog.md | Applies configuration and hardening after enrollment is stable          |
| 05_Configure_App_Deployment_Assignment_Detection_And_Remediation.md                   | Deploys apps after device enrollment and targeting are validated        |
| 06_Configure_Defender_For_Endpoint_Integration_And_Device_Risk_Signals.md             | Adds Defender device risk integration after managed device state exists |
| 07_Troubleshoot_Intune_Enrollment_Compliance_App_Deployment_And_Device_Sync_Issues.md | Troubleshoots enrollment and device sync issues from this baseline      |