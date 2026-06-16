# 03_Configure_Meetings_Messaging_Calling_And_App_Policies

## Objective

Configure Microsoft Teams baseline policies for meetings, messaging, calling, and Teams apps, then assign those policies to test users and validate effective behavior from the Teams client.

## Lab Context

This workbook builds on the Teams tenant baseline and team/channel membership workbooks. The goal is to configure workload policies that control what users can do inside Teams meetings, chats, calls, and app surfaces.

This workbook does not fully configure guest access, external access, retention, eDiscovery, or audit handoff. Those are handled in later workbooks.

## Prerequisites

| Requirement | Notes |
|---|---|
| Microsoft 365 tenant | Teams enabled |
| Admin role | Teams Administrator or Global Administrator |
| Test users | At least two licensed internal users |
| Teams baseline complete | Workbook 01 complete |
| Teams/channel baseline complete | Workbook 02 complete |
| Teams admin center access | `https://admin.teams.microsoft.com` |
| Optional PowerShell | MicrosoftTeams module installed |
| Test team | `LAB-Teams-Channel-Membership` or equivalent |

## Target Scope

| Area | Baseline Decision |
|---|---|
| Meeting policy | Create scoped lab meeting policy |
| Messaging policy | Create scoped lab messaging policy |
| Calling policy | Create scoped lab calling policy |
| App permission policy | Create scoped lab app permission policy |
| App setup policy | Create scoped lab app setup policy |
| Assignment method | Assign directly to test user |
| Validation method | Teams admin center, Teams PowerShell, and Teams client |
| Production impact | Avoid changing Global unless explicitly required |

## Configuration Steps

| Step | Task | Admin Center / Device | Command / Location | PowerShell | Expected Result |
|---:|---|---|---|---|---|
| 1 | Connect to Teams PowerShell | Admin workstation | PowerShell | `Connect-MicrosoftTeams` | Admin connects successfully |
| 2 | Confirm test users are Teams-enabled | Admin workstation | Teams admin center > **Users** > **Manage users** | `Get-CsOnlineUser -Identity user1@domain.com` | Test user returns with Teams properties |
| 3 | Review current user policy assignments | Teams admin center | **Users** > select test user > **Policies** | `Get-CsOnlineUser -Identity user1@domain.com | Select DisplayName,TeamsMeetingPolicy,TeamsMessagingPolicy,TeamsCallingPolicy,TeamsAppPermissionPolicy,TeamsAppSetupPolicy` | Current policy baseline is documented |
| 4 | Review global meeting policy | Teams admin center | **Meetings** > **Meeting policies** > **Global** | `Get-CsTeamsMeetingPolicy -Identity Global` | Existing global meeting policy is documented |
| 5 | Create lab meeting policy | Teams admin center | **Meetings** > **Meeting policies** > **Add** | `New-CsTeamsMeetingPolicy -Identity "LAB-Teams-Meeting-Baseline" -Description "Lab meeting policy baseline"` | Lab meeting policy is created |
| 6 | Configure meeting lobby baseline | Teams admin center | Meeting policy > **Participants & guests** | `Set-CsTeamsMeetingPolicy -Identity "LAB-Teams-Meeting-Baseline" -AutoAdmittedUsers "EveryoneInCompanyExcludingGuests"` | Internal users bypass lobby, guests require admission |
| 7 | Configure meeting chat baseline | Teams admin center | Meeting policy > **Meeting engagement** | `Set-CsTeamsMeetingPolicy -Identity "LAB-Teams-Meeting-Baseline" -MeetingChatEnabledType Enabled` | Meeting chat is enabled |
| 8 | Configure cloud recording baseline | Teams admin center | Meeting policy > **Recording & transcription** | `Set-CsTeamsMeetingPolicy -Identity "LAB-Teams-Meeting-Baseline" -AllowCloudRecording $true` | Cloud recording is allowed for lab user |
| 9 | Configure transcription baseline | Teams admin center | Meeting policy > **Recording & transcription** | `Set-CsTeamsMeetingPolicy -Identity "LAB-Teams-Meeting-Baseline" -AllowTranscription $true` | Transcription is allowed if licensed and available |
| 10 | Configure screen sharing baseline | Teams admin center | Meeting policy > **Content sharing** | `Set-CsTeamsMeetingPolicy -Identity "LAB-Teams-Meeting-Baseline" -ScreenSharingMode EntireScreen` | User can share screen |
| 11 | Assign lab meeting policy to test user | Teams admin center | User > **Policies** > **Meeting policy** | `Grant-CsTeamsMeetingPolicy -Identity user1@domain.com -PolicyName "LAB-Teams-Meeting-Baseline"` | Meeting policy is assigned |
| 12 | Review global messaging policy | Teams admin center | **Messaging policies** > **Global** | `Get-CsTeamsMessagingPolicy -Identity Global` | Existing global messaging policy is documented |
| 13 | Create lab messaging policy | Teams admin center | **Messaging policies** > **Add** | `New-CsTeamsMessagingPolicy -Identity "LAB-Teams-Messaging-Baseline" -Description "Lab messaging policy baseline"` | Lab messaging policy is created |
| 14 | Configure user chat baseline | Teams admin center | Messaging policy settings | `Set-CsTeamsMessagingPolicy -Identity "LAB-Teams-Messaging-Baseline" -AllowUserChat $true` | 1:1 and group chat is allowed |
| 15 | Configure message edit/delete baseline | Teams admin center | Messaging policy settings | `Set-CsTeamsMessagingPolicy -Identity "LAB-Teams-Messaging-Baseline" -AllowUserEditMessage $true -AllowUserDeleteMessage $true` | Users can edit and delete their sent messages |
| 16 | Configure owners delete sent messages baseline | Teams admin center | Messaging policy settings | `Set-CsTeamsMessagingPolicy -Identity "LAB-Teams-Messaging-Baseline" -AllowOwnerDeleteMessage $true` | Owners can delete sent messages where supported |
| 17 | Configure URL preview baseline | Teams admin center | Messaging policy settings | `Set-CsTeamsMessagingPolicy -Identity "LAB-Teams-Messaging-Baseline" -AllowUrlPreviews $true` | URL previews are allowed |
| 18 | Assign lab messaging policy to test user | Teams admin center | User > **Policies** > **Messaging policy** | `Grant-CsTeamsMessagingPolicy -Identity user1@domain.com -PolicyName "LAB-Teams-Messaging-Baseline"` | Messaging policy is assigned |
| 19 | Review global calling policy | Teams admin center | **Voice** > **Calling policies** > **Global** | `Get-CsTeamsCallingPolicy -Identity Global` | Existing global calling policy is documented |
| 20 | Create lab calling policy | Teams admin center | **Voice** > **Calling policies** > **Add** | `New-CsTeamsCallingPolicy -Identity "LAB-Teams-Calling-Baseline" -Description "Lab calling policy baseline"` | Lab calling policy is created |
| 21 | Configure private calling baseline | Teams admin center | Calling policy settings | `Set-CsTeamsCallingPolicy -Identity "LAB-Teams-Calling-Baseline" -AllowPrivateCalling $true` | User can place private Teams calls |
| 22 | Configure call forwarding baseline | Teams admin center | Calling policy settings | `Set-CsTeamsCallingPolicy -Identity "LAB-Teams-Calling-Baseline" -AllowCallForwardingToUser $true -AllowCallForwardingToPhone $false` | Forwarding to Teams users is allowed; forwarding to phone is blocked |
| 23 | Configure voicemail baseline | Teams admin center | Calling policy settings | `Set-CsTeamsCallingPolicy -Identity "LAB-Teams-Calling-Baseline" -AllowVoicemail AlwaysEnabled` | Voicemail behavior is set |
| 24 | Assign lab calling policy to test user | Teams admin center | User > **Policies** > **Calling policy** | `Grant-CsTeamsCallingPolicy -Identity user1@domain.com -PolicyName "LAB-Teams-Calling-Baseline"` | Calling policy is assigned |
| 25 | Review global app permission policy | Teams admin center | **Teams apps** > **Permission policies** > **Global** | `Get-CsTeamsAppPermissionPolicy -Identity Global` | Existing app permission posture is documented |
| 26 | Create lab app permission policy | Teams admin center | **Teams apps** > **Permission policies** > **Add** | `New-CsTeamsAppPermissionPolicy -Identity "LAB-Teams-AppPermission-Baseline" -Description "Lab app permission baseline"` | Lab app permission policy is created |
| 27 | Configure Microsoft apps posture | Teams admin center | App permission policy > **Microsoft apps** | N/A | Microsoft apps are allowed or restricted according to lab baseline |
| 28 | Configure third-party apps posture | Teams admin center | App permission policy > **Third-party apps** | N/A | Third-party app posture is documented |
| 29 | Configure custom apps posture | Teams admin center | App permission policy > **Custom apps** | N/A | Custom app posture is documented |
| 30 | Assign lab app permission policy | Teams admin center | User > **Policies** > **App permission policy** | `Grant-CsTeamsAppPermissionPolicy -Identity user1@domain.com -PolicyName "LAB-Teams-AppPermission-Baseline"` | App permission policy is assigned |
| 31 | Review global app setup policy | Teams admin center | **Teams apps** > **Setup policies** > **Global** | `Get-CsTeamsAppSetupPolicy -Identity Global` | Existing pinned apps and app setup behavior are documented |
| 32 | Create lab app setup policy | Teams admin center | **Teams apps** > **Setup policies** > **Add** | `New-CsTeamsAppSetupPolicy -Identity "LAB-Teams-AppSetup-Baseline" -Description "Lab app setup policy baseline"` | Lab app setup policy is created |
| 33 | Configure pinned app baseline | Teams admin center | App setup policy > **Pinned apps** | N/A | Required pinned apps are configured |
| 34 | Configure user pinning baseline | Teams admin center | App setup policy settings | N/A | User pinning is allowed or blocked according to baseline |
| 35 | Assign lab app setup policy | Teams admin center | User > **Policies** > **App setup policy** | `Grant-CsTeamsAppSetupPolicy -Identity user1@domain.com -PolicyName "LAB-Teams-AppSetup-Baseline"` | App setup policy is assigned |
| 36 | Validate assigned policy state | Admin workstation | PowerShell | `Get-CsOnlineUser -Identity user1@domain.com | Select DisplayName,TeamsMeetingPolicy,TeamsMessagingPolicy,TeamsCallingPolicy,TeamsAppPermissionPolicy,TeamsAppSetupPolicy` | All lab policies show on test user |
| 37 | Sign in as test user | Teams client | Desktop or web Teams client | N/A | User can access Teams |
| 38 | Validate meeting behavior | Teams client | Schedule test meeting | N/A | Meeting options match assigned meeting policy |
| 39 | Validate messaging behavior | Teams client | Send, edit, delete test message | N/A | Messaging behavior matches assigned messaging policy |
| 40 | Validate calling behavior | Teams client | Start Teams call with another test user | N/A | Calling behavior matches assigned calling policy |
| 41 | Validate app behavior | Teams client | Open **Apps** and left rail | N/A | App availability and pinned apps match app policies |
| 42 | Document final policy baseline | Workbook notes | Record policies, assignments, and test results | N/A | Policy baseline is documented |

## Validation

| Check | Command / Location | Expected Result |
|---|---|---|
| Teams PowerShell connects | `Connect-MicrosoftTeams` | Admin session opens |
| Lab meeting policy exists | `Get-CsTeamsMeetingPolicy -Identity "LAB-Teams-Meeting-Baseline"` | Policy returns |
| Lab messaging policy exists | `Get-CsTeamsMessagingPolicy -Identity "LAB-Teams-Messaging-Baseline"` | Policy returns |
| Lab calling policy exists | `Get-CsTeamsCallingPolicy -Identity "LAB-Teams-Calling-Baseline"` | Policy returns |
| Lab app permission policy exists | `Get-CsTeamsAppPermissionPolicy -Identity "LAB-Teams-AppPermission-Baseline"` | Policy returns |
| Lab app setup policy exists | `Get-CsTeamsAppSetupPolicy -Identity "LAB-Teams-AppSetup-Baseline"` | Policy returns |
| User has assigned policies | `Get-CsOnlineUser -Identity user1@domain.com | Select TeamsMeetingPolicy,TeamsMessagingPolicy,TeamsCallingPolicy,TeamsAppPermissionPolicy,TeamsAppSetupPolicy` | Expected lab policies are listed |
| Meeting controls work | Teams client > schedule meeting > meeting options | Lobby, chat, recording, transcription, and sharing behavior match policy |
| Messaging controls work | Teams client > chat/channel | Edit, delete, URL preview, and chat behavior match policy |
| Calling controls work | Teams client > Calls | Private calling and forwarding behavior match policy |
| App controls work | Teams client > Apps / left rail | Allowed apps and pinned apps match policy |

## Troubleshooting

| Symptom | Likely Cause | Fix |
|---|---|---|
| Policy does not appear on user immediately | Teams policy propagation delay | Wait, sign out/in, and recheck with `Get-CsOnlineUser` |
| User cannot schedule expected meeting options | Wrong meeting policy or policy not propagated | Confirm assignment and wait for propagation |
| Recording option missing | Meeting policy, license, storage, or compliance setting blocks it | Check meeting policy, license, OneDrive/SharePoint availability, and compliance settings |
| Transcription option missing | Feature disabled, unsupported language/region, license, or policy restriction | Confirm `AllowTranscription`, licensing, and tenant availability |
| User cannot edit or delete messages | Messaging policy blocks action | Confirm assigned messaging policy |
| User cannot start private call | Calling policy blocks private calling | Confirm `AllowPrivateCalling` is enabled |
| Call forwarding to phone unavailable | Calling policy blocks phone forwarding or user lacks Phone System capability | Confirm policy and licensing |
| App unavailable in Teams client | App permission policy blocks app | Review app permission policy and app status |
| Pinned app missing | App setup policy not assigned or not propagated | Confirm app setup policy assignment and wait |
| PowerShell command fails | Module outdated or parameter unsupported in current module | Run `Update-Module MicrosoftTeams` and check command syntax |
| Policy assignment shows blank | User is using Global policy | Blank assignment usually means global policy applies |

## Rollback / Cleanup

| Change | Rollback |
|---|---|
| Lab meeting policy assigned | `Grant-CsTeamsMeetingPolicy -Identity user1@domain.com -PolicyName $null` |
| Lab messaging policy assigned | `Grant-CsTeamsMessagingPolicy -Identity user1@domain.com -PolicyName $null` |
| Lab calling policy assigned | `Grant-CsTeamsCallingPolicy -Identity user1@domain.com -PolicyName $null` |
| Lab app permission policy assigned | `Grant-CsTeamsAppPermissionPolicy -Identity user1@domain.com -PolicyName $null` |
| Lab app setup policy assigned | `Grant-CsTeamsAppSetupPolicy -Identity user1@domain.com -PolicyName $null` |
| Lab meeting policy created | `Remove-CsTeamsMeetingPolicy -Identity "LAB-Teams-Meeting-Baseline"` |
| Lab messaging policy created | `Remove-CsTeamsMessagingPolicy -Identity "LAB-Teams-Messaging-Baseline"` |
| Lab calling policy created | `Remove-CsTeamsCallingPolicy -Identity "LAB-Teams-Calling-Baseline"` |
| Lab app permission policy created | `Remove-CsTeamsAppPermissionPolicy -Identity "LAB-Teams-AppPermission-Baseline"` |
| Lab app setup policy created | `Remove-CsTeamsAppSetupPolicy -Identity "LAB-Teams-AppSetup-Baseline"` |
| PowerShell session opened | `Disconnect-MicrosoftTeams` |

## Notes

- Avoid changing Global policies during lab work unless the goal is tenant-wide behavior.
- Prefer scoped lab policies assigned to test users.
- Teams policy changes can take time to apply.
- A blank policy assignment on a user often means the Global policy is effective.
- Meeting recording and transcription also depend on licensing, storage, compliance, and tenant availability.
- Calling behavior depends on Teams calling policy, user licensing, and whether the tenant uses Teams Phone.
- App behavior depends on app permission policies, app setup policies, app status, and org-wide app settings.
- Guest and external collaboration controls are handled in workbook 04.
- Retention, eDiscovery, audit, and compliance handoff are handled in workbook 05.