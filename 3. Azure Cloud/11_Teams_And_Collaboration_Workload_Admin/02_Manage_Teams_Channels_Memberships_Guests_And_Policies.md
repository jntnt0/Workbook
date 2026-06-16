# 02_Manage_Teams_Channels_Memberships_Guests_And_Policies

## Objective

Create and manage Microsoft Teams teams, channels, owners, members, guest users, and team/channel policy controls from the Teams admin center and Teams PowerShell.

## Lab Context

This workbook builds on the Teams tenant baseline. The goal is to manage team structure and membership, not meeting, messaging, calling, app, compliance, or troubleshooting policy depth. Those are handled in later workbooks.

## Prerequisites

| Requirement | Notes |
|---|---|
| Microsoft 365 tenant | Teams enabled |
| Admin role | Teams Administrator or Global Administrator |
| Test users | At least two internal licensed users |
| Guest test account | External email account available |
| Teams admin center access | `https://admin.teams.microsoft.com` |
| Optional PowerShell | MicrosoftTeams module installed |
| Baseline workbook complete | `01_Configure_Teams_Admin_Center_Org_Settings_And_Service_Baseline` |

## Target Scope

| Area | Baseline Decision |
|---|---|
| Teams creation | Create a controlled lab team |
| Owners | Minimum two owners where possible |
| Members | Add internal test users |
| Guests | Add one guest only if guest access is allowed |
| Channels | Validate standard, private, and shared channel behavior where licensed/supported |
| Team policy | Review global Teams policy and create a scoped lab policy if needed |
| Channel policy | Validate private/shared channel controls |
| Lifecycle | Document team ownership and cleanup path |

## Configuration Steps

| Step | Task | Admin Center / Device | Command / Location | PowerShell | Expected Result |
|---:|---|---|---|---|---|
| 1 | Connect to Teams PowerShell | Admin workstation | PowerShell | `Connect-MicrosoftTeams` | Admin connects to Teams PowerShell |
| 2 | Confirm test users are Teams-enabled | Admin workstation | Teams admin center > **Users** > **Manage users** | `Get-CsOnlineUser -Identity user1@domain.com` | Test users return successfully |
| 3 | Review existing teams | Teams admin center | **Teams** > **Manage teams** | `Get-Team` | Existing teams are visible |
| 4 | Create baseline lab team | Teams admin center or PowerShell | **Teams** > **Manage teams** > **Add** | `New-Team -DisplayName "LAB-Teams-Channel-Membership" -Description "Lab team for Teams channel and membership management" -Visibility Private` | Private lab team is created |
| 5 | Capture new team GroupId | Admin workstation | PowerShell | `$Team = Get-Team -DisplayName "LAB-Teams-Channel-Membership"; $Team.GroupId` | Team GroupId is available |
| 6 | Add primary owner | Teams admin center | Open team > **Members** > add owner | `Add-TeamUser -GroupId $Team.GroupId -User owner1@domain.com -Role Owner` | Owner is added |
| 7 | Add secondary owner | Teams admin center | Open team > **Members** > add owner | `Add-TeamUser -GroupId $Team.GroupId -User owner2@domain.com -Role Owner` | Second owner is added |
| 8 | Add internal member | Teams admin center | Open team > **Members** > add member | `Add-TeamUser -GroupId $Team.GroupId -User user1@domain.com -Role Member` | Internal member is added |
| 9 | Add second internal member | Teams admin center | Open team > **Members** > add member | `Add-TeamUser -GroupId $Team.GroupId -User user2@domain.com -Role Member` | Second internal member is added |
| 10 | Validate membership | Admin workstation | Teams admin center > team > members | `Get-TeamUser -GroupId $Team.GroupId` | Owners and members are listed correctly |
| 11 | Review team settings | Teams admin center | **Teams** > **Manage teams** > select team > **Settings** | `Get-Team -GroupId $Team.GroupId` | Team visibility, description, and settings are documented |
| 12 | Configure member permissions | Teams admin center | Team settings > **Member permissions** | N/A | Member permissions are reviewed and adjusted for lab baseline |
| 13 | Configure guest permissions at team level | Teams admin center | Team settings > **Guest permissions** | N/A | Guest permissions are documented |
| 14 | Create standard channel | Teams admin center or Teams client | Team > **Channels** > add channel | `New-TeamChannel -GroupId $Team.GroupId -DisplayName "standard-operations" -MembershipType Standard` | Standard channel is created |
| 15 | Create private channel | Teams admin center or Teams client | Team > **Channels** > add private channel | `New-TeamChannel -GroupId $Team.GroupId -DisplayName "private-admin" -MembershipType Private -Owner owner1@domain.com` | Private channel is created |
| 16 | Add user to private channel | Teams client or PowerShell | Private channel > manage members | `Add-TeamChannelUser -GroupId $Team.GroupId -DisplayName "private-admin" -User user1@domain.com` | User is added to private channel |
| 17 | Validate private channel membership | Admin workstation | Teams client or PowerShell | `Get-TeamChannelUser -GroupId $Team.GroupId -DisplayName "private-admin"` | Private channel membership is listed |
| 18 | Review shared channel availability | Teams admin center | **Teams** > **Teams policies** or channel policy controls | N/A | Shared channel capability is confirmed or documented as unavailable |
| 19 | Create shared channel if enabled | Teams client or PowerShell | Team > **Channels** > add shared channel | `New-TeamChannel -GroupId $Team.GroupId -DisplayName "shared-collaboration" -MembershipType Shared -Owner owner1@domain.com` | Shared channel is created if allowed |
| 20 | Review global Teams policy | Teams admin center | **Teams** > **Teams policies** > **Global** | `Get-CsTeamsChannelsPolicy -Identity Global` | Global channel/team policy posture is documented |
| 21 | Create lab Teams/channel policy if needed | Teams admin center | **Teams** > **Teams policies** > **Add** | `New-CsTeamsChannelsPolicy -Identity "LAB-Teams-Channels-Baseline" -AllowPrivateChannelCreation $true -AllowSharedChannelCreation $true` | Lab policy is created |
| 22 | Assign lab policy to test user | Teams admin center | **Users** > select user > **Policies** | `Grant-CsTeamsChannelsPolicy -Identity user1@domain.com -PolicyName "LAB-Teams-Channels-Baseline"` | User receives lab channel policy |
| 23 | Validate policy assignment | Admin workstation | PowerShell | `Get-CsOnlineUser -Identity user1@domain.com | Select DisplayName,TeamsChannelsPolicy` | Assigned policy is visible |
| 24 | Review guest access tenant state | Teams admin center | **Users** > **Guest access** | `Get-CsTeamsGuestMessagingConfiguration` | Guest access state is documented |
| 25 | Add guest to team if allowed | Teams admin center or Teams client | Team > members > add guest email | `Add-TeamUser -GroupId $Team.GroupId -User guestuser_external#EXT#@tenant.onmicrosoft.com -Role Member` | Guest is added if guest object already exists |
| 26 | Validate guest membership | Teams admin center | Team > **Members** | `Get-TeamUser -GroupId $Team.GroupId | Where-Object {$_.User -like "*#EXT#*"}` | Guest membership is visible |
| 27 | Confirm team appears for test users | Teams client | Sign in as test user | N/A | Team and allowed channels appear |
| 28 | Confirm private channel visibility | Teams client | Sign in as user with and without private channel membership | N/A | Only private channel members see private channel |
| 29 | Confirm shared channel visibility | Teams client | Sign in as assigned user | N/A | Shared channel appears only if enabled and assigned |
| 30 | Document final membership and channel model | Workbook notes | Record owners, members, guests, channels, and policies | N/A | Team structure baseline is documented |

## Validation

| Check | Command / Location | Expected Result |
|---|---|---|
| Teams PowerShell connects | `Connect-MicrosoftTeams` | Admin session opens |
| Lab team exists | `Get-Team -DisplayName "LAB-Teams-Channel-Membership"` | Team returns |
| Owners and members are correct | `Get-TeamUser -GroupId $Team.GroupId` | Expected users and roles appear |
| Standard channel exists | `Get-TeamChannel -GroupId $Team.GroupId` | `standard-operations` appears |
| Private channel exists | `Get-TeamChannel -GroupId $Team.GroupId` | `private-admin` appears |
| Private channel membership correct | `Get-TeamChannelUser -GroupId $Team.GroupId -DisplayName "private-admin"` | Only assigned users appear |
| Teams/channel policy visible | `Get-CsTeamsChannelsPolicy` | Global or lab policy returns |
| User policy assigned | `Get-CsOnlineUser -Identity user1@domain.com | Select TeamsChannelsPolicy` | Lab policy appears if assigned |
| Test user sees team | Teams client | Team appears in client |
| Non-member cannot see private channel | Teams client | Private channel is hidden |

## Troubleshooting

| Symptom | Likely Cause | Fix |
|---|---|---|
| Team creation fails | User lacks role or Teams service not provisioned | Confirm Teams Administrator role and service health |
| User cannot be added | User is unlicensed, disabled, or not found | Assign license, enable account, verify UPN |
| Guest cannot be added | Guest access disabled or guest object missing | Enable guest access and invite guest through Entra ID or Teams |
| Private channel creation unavailable | Policy blocks private channels | Review Teams/channel policy and assign allowed policy |
| Shared channel creation unavailable | Shared channels disabled or unsupported policy state | Enable shared channel settings and validate B2B direct connect requirements |
| User cannot see new team | Client cache or propagation delay | Sign out/in, refresh Teams, wait for provisioning |
| Policy assignment not taking effect | Teams policy propagation delay | Wait and recheck with `Get-CsOnlineUser` |
| PowerShell command fails | MicrosoftTeams module outdated | Run `Update-Module MicrosoftTeams` |
| Guest appears but cannot access resources | SharePoint/M365 group guest settings conflict | Review M365 Groups, SharePoint sharing, and Entra guest restrictions |
| Private channel user cannot access files | SharePoint site permission delay | Wait for private channel site provisioning and retest |

## Rollback / Cleanup

| Change | Rollback |
|---|---|
| Lab team created | Delete team from Teams admin center or run `Remove-Team -GroupId $Team.GroupId` |
| Internal user added | `Remove-TeamUser -GroupId $Team.GroupId -User user1@domain.com` |
| Guest added | Remove guest from team membership |
| Standard channel created | Delete channel from Teams client/admin center |
| Private channel created | Delete private channel from Teams client/admin center |
| Shared channel created | Delete shared channel from Teams client/admin center |
| Lab channel policy assigned | `Grant-CsTeamsChannelsPolicy -Identity user1@domain.com -PolicyName $null` |
| Lab channel policy created | `Remove-CsTeamsChannelsPolicy -Identity "LAB-Teams-Channels-Baseline"` |

## Notes

- Keep at least two owners on production teams where possible.
- Private channels have separate membership from the parent team.
- Shared channels are for collaboration without adding users to the parent team.
- Guest access depends on Teams settings, Entra ID external identities, Microsoft 365 Groups, and SharePoint sharing.
- Teams policy changes can take time to apply.
- Do not use this workbook to fully configure guest/external governance. That belongs in workbook 04.
- Do not use this workbook to configure meetings, messaging, calling, or apps. That belongs in workbook 03.