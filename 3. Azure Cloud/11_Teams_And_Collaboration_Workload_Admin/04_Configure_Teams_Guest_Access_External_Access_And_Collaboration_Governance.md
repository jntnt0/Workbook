# 04_Configure_Teams_Guest_Access_External_Access_And_Collaboration_Governance

## Objective

Configure and validate Microsoft Teams guest access, external access, shared collaboration settings, and governance handoff points across Teams, Entra ID, Microsoft 365 Groups, and SharePoint.

## Lab Context

This workbook builds on the Teams tenant baseline, team/channel membership, and Teams policy workbooks. The goal is to configure collaboration boundaries for people outside the organization.

This workbook focuses on Teams collaboration governance. Retention, eDiscovery, audit, legal hold, and compliance search are handled in the next workbook.

## Prerequisites

| Requirement | Notes |
|---|---|
| Microsoft 365 tenant | Teams enabled |
| Admin role | Teams Administrator or Global Administrator |
| Entra admin access | Needed for external identities and guest restrictions |
| SharePoint admin access | Needed because Teams files depend on SharePoint/OneDrive sharing |
| Test internal users | At least two licensed users |
| External test account | Personal or external business email account |
| Existing test team | Example: `LAB-Teams-Channel-Membership` |
| Optional PowerShell | MicrosoftTeams and Microsoft Graph modules |

## Target Scope

| Area | Baseline Decision |
|---|---|
| Guest access | Enabled for controlled lab users |
| External access | Allow selected domains where possible |
| Anonymous meeting access | Review only unless required |
| Shared channels | Validate whether enabled and governed |
| Guest permissions | Restrict guest creation/deletion where appropriate |
| Entra B2B | Validate guest invite and collaboration settings |
| SharePoint sharing | Align with Teams guest file access |
| Access reviews | Document handoff for periodic guest review |
| Production impact | Avoid broad allow-all external collaboration without documentation |

## Configuration Steps

| Step | Task | Admin Center / Device | Command / Location | PowerShell | Expected Result |
|---:|---|---|---|---|---|
| 1 | Connect to Teams PowerShell | Admin workstation | PowerShell | `Connect-MicrosoftTeams` | Admin connects successfully |
| 2 | Confirm test team exists | Admin workstation | Teams admin center > **Teams** > **Manage teams** | `$Team = Get-Team -DisplayName "LAB-Teams-Channel-Membership"; $Team.GroupId` | Test team GroupId is available |
| 3 | Review current guest access state | Teams admin center | **Users** > **Guest access** | `Get-CsTeamsClientConfiguration` | Current guest posture is documented |
| 4 | Review guest messaging configuration | Teams admin center | **Users** > **Guest access** | `Get-CsTeamsGuestMessagingConfiguration` | Guest messaging settings are visible |
| 5 | Enable Teams guest access if lab requires it | Teams admin center | **Users** > **Guest access** > turn on guest access | N/A | Teams guest access is enabled |
| 6 | Configure guest calling behavior | Teams admin center | **Users** > **Guest access** > **Calling** | N/A | Guest calling is allowed or blocked according to baseline |
| 7 | Configure guest meeting behavior | Teams admin center | **Users** > **Guest access** > **Meetings** | N/A | Guest meeting permissions are documented |
| 8 | Configure guest messaging behavior | Teams admin center | **Users** > **Guest access** > **Messaging** | `Set-CsTeamsGuestMessagingConfiguration -AllowUserEditMessage $false -AllowUserDeleteMessage $false` | Guest messaging permissions are restricted or documented |
| 9 | Review external access state | Teams admin center | **Users** > **External access** | `Get-CsTenantFederationConfiguration` | External/federation configuration is documented |
| 10 | Decide external access mode | Teams admin center | **Users** > **External access** | N/A | Decision recorded: allow all, allow selected domains, block selected domains, or disable |
| 11 | Configure external access for selected domains | Teams admin center | **Users** > **External access** > domain allow/block list | `New-CsTenantFederatedIdentityPlatform -Identity "contoso.com"` | Selected external domain is allowed where supported |
| 12 | Validate federation/external access settings | Admin workstation | Teams PowerShell | `Get-CsTenantFederationConfiguration` | Federation settings match expected baseline |
| 13 | Review anonymous meeting join posture | Teams admin center | **Meetings** > **Meeting settings** | `Get-CsTeamsMeetingConfiguration` | Anonymous join posture is documented |
| 14 | Configure anonymous users if required | Teams admin center | **Meetings** > **Meeting settings** | `Set-CsTeamsMeetingConfiguration -DisableAnonymousJoin $false` | Anonymous join is allowed only if approved |
| 15 | Review meeting policy guest controls | Teams admin center | **Meetings** > **Meeting policies** > lab/global policy | `Get-CsTeamsMeetingPolicy -Identity "LAB-Teams-Meeting-Baseline"` | Guest lobby and meeting controls are documented |
| 16 | Configure lobby baseline for external participants | Teams admin center | Meeting policy > **Participants & guests** | `Set-CsTeamsMeetingPolicy -Identity "LAB-Teams-Meeting-Baseline" -AutoAdmittedUsers EveryoneInCompanyExcludingGuests` | Guests and external users wait in lobby |
| 17 | Review shared channel policy | Teams admin center | **Teams** > **Teams policies** or channel policy area | `Get-CsTeamsChannelsPolicy -Identity Global` | Shared channel posture is documented |
| 18 | Configure lab shared channel policy if used | Teams admin center | Teams/channel policy settings | `Set-CsTeamsChannelsPolicy -Identity "LAB-Teams-Channels-Baseline" -AllowSharedChannelCreation $true` | Lab user can create shared channels |
| 19 | Review B2B direct connect requirement for shared channels | Entra admin center | **External Identities** > **Cross-tenant access settings** | N/A | Cross-tenant posture is documented |
| 20 | Configure cross-tenant access for lab partner tenant if available | Entra admin center | **External Identities** > **Cross-tenant access settings** > add organization | N/A | Partner tenant settings are configured if available |
| 21 | Review Entra guest invite settings | Entra admin center | **External Identities** > **External collaboration settings** | N/A | Guest invitation restrictions are documented |
| 22 | Configure guest invitation restrictions | Entra admin center | External collaboration settings | N/A | Guest invitations are limited to approved users/admins where required |
| 23 | Review Microsoft 365 Groups guest settings | Microsoft 365 admin center | **Settings** > **Org settings** > **Microsoft 365 Groups** | N/A | Group guest access posture is documented |
| 24 | Validate Microsoft 365 Groups guest membership is allowed if Teams guests are required | Microsoft 365 admin center | Microsoft 365 Groups settings | N/A | Groups allow guest membership only if approved |
| 25 | Review SharePoint external sharing baseline | SharePoint admin center | **Policies** > **Sharing** | N/A | SharePoint sharing level supports expected Teams guest file access |
| 26 | Align SharePoint sharing with Teams guest model | SharePoint admin center | **Policies** > **Sharing** | N/A | Sharing is not more permissive than intended |
| 27 | Review OneDrive external sharing baseline | SharePoint admin center | **Policies** > **Sharing** > OneDrive | N/A | OneDrive sharing is documented |
| 28 | Add guest to test team | Teams admin center or Teams client | Team > **Members** > add external email | N/A | Guest invite is sent |
| 29 | Validate guest object in Entra ID | Entra admin center | **Users** > search external guest | N/A | Guest user exists with guest user type |
| 30 | Validate guest membership in team | Admin workstation | Teams PowerShell | `Get-TeamUser -GroupId $Team.GroupId | Where-Object {$_.Role -eq "Member"}` | Guest appears in team membership after invite acceptance |
| 31 | Validate guest Teams access | Guest browser/client | Sign in with external account | N/A | Guest can access the invited team |
| 32 | Validate guest channel visibility | Guest browser/client | Open test team | N/A | Guest sees only allowed teams/channels |
| 33 | Validate guest file access | Guest browser/client | Open channel **Files** tab | N/A | Guest can access files only as allowed by SharePoint settings |
| 34 | Validate external chat behavior | Teams client | Internal user searches external user by email | N/A | External chat works only if external access allows the domain |
| 35 | Validate guest versus external user difference | Teams client | Compare team guest membership with external chat-only contact | N/A | Guest is a member of tenant resource; external user is federation/chat-only |
| 36 | Review access review handoff | Entra admin center | **Identity Governance** > **Access reviews** | N/A | Guest/team access review handoff is documented |
| 37 | Create or document guest review process | Entra admin center | Access reviews > create review if licensed | N/A | Guest access review process exists or is documented as manual |
| 38 | Review sensitivity label handoff for Teams/M365 Groups | Purview portal | **Information Protection** > **Labels** | N/A | Label-based collaboration governance is documented if used |
| 39 | Review audit handoff for guest activity | Purview portal | **Audit** | N/A | Guest audit validation is assigned to compliance workbook |
| 40 | Document final external collaboration baseline | Workbook notes | Record guest, external, SharePoint, Entra, and review settings | N/A | Collaboration governance baseline is complete |

## Validation

| Check | Command / Location | Expected Result |
|---|---|---|
| Teams PowerShell connects | `Connect-MicrosoftTeams` | Admin session opens |
| Test team exists | `Get-Team -DisplayName "LAB-Teams-Channel-Membership"` | Team returns |
| Guest access reviewed | Teams admin center > Users > Guest access | Guest access state is known |
| Guest messaging config visible | `Get-CsTeamsGuestMessagingConfiguration` | Guest messaging configuration returns |
| External access reviewed | `Get-CsTenantFederationConfiguration` | Federation/external access config returns |
| Anonymous join reviewed | `Get-CsTeamsMeetingConfiguration` | Meeting configuration returns |
| Guest object exists | Entra admin center > Users | Guest user appears |
| Guest appears in team | `Get-TeamUser -GroupId $Team.GroupId` | Guest is listed as member |
| Guest can open team | Teams web or desktop client | Guest sees invited team |
| Guest file access works | Teams > Channel > Files | Guest can open files only if sharing permits |
| External chat works as intended | Teams client | External chat follows domain allow/block configuration |
| Access review process documented | Entra admin center or workbook notes | Review owner and cadence are recorded |

## Troubleshooting

| Symptom | Likely Cause | Fix |
|---|---|---|
| Guest invite cannot be sent | Teams guest access, Entra guest invite, or M365 Groups guest setting blocks it | Review Teams guest access, Entra external collaboration, and M365 Groups guest settings |
| Guest accepts invite but cannot see team | Provisioning delay or guest not added to team | Wait, recheck team membership, resend invite if needed |
| Guest sees team but cannot open files | SharePoint sharing or site permissions block file access | Review SharePoint sharing policy and team-connected SharePoint site permissions |
| External user cannot be found in Teams search | External access disabled or domain blocked | Review Teams external access domain configuration |
| External chat works but guest team access fails | External access and guest access are separate controls | Configure guest access and invite the user to the team |
| Guest access works but shared channel fails | Shared channels require cross-tenant/B2B direct connect configuration | Review Entra cross-tenant access settings |
| Anonymous meeting join unavailable | Meeting configuration or meeting policy blocks anonymous join | Review `Get-CsTeamsMeetingConfiguration` and meeting policy |
| Guest can do more than expected | Guest messaging/meeting settings too permissive | Restrict guest settings in Teams admin center |
| Policy change not taking effect | Teams propagation delay | Wait, sign out/in, retest |
| Guest user remains after removal from team | Guest object remains in Entra ID | Remove guest from team, then delete or review guest account separately if required |

## Rollback / Cleanup

| Change | Rollback |
|---|---|
| Guest added to team | Remove guest from team membership |
| Guest object created | Delete guest user from Entra ID if no longer needed |
| Guest access enabled | Disable guest access in Teams admin center if required |
| External domain allowed | Remove domain from allow list or block it |
| Anonymous join enabled | Disable anonymous join or restore previous meeting configuration |
| Shared channel creation enabled | Disable shared channel creation in Teams/channel policy |
| Cross-tenant settings changed | Restore previous Entra cross-tenant access settings |
| SharePoint sharing changed | Restore previous organization sharing level |
| Lab policy changed | Revert lab policy or remove assignment from test user |

## Notes

- Guest access and external access are not the same thing.
- Guest access adds an external person as a guest in your tenant and into team resources.
- External access allows federation/chat/calling with users outside the organization without adding them as guests.
- Teams file access depends on SharePoint and OneDrive sharing settings.
- Shared channels depend on Teams policy and Entra cross-tenant access settings.
- Microsoft 365 Groups settings can block guest membership even when Teams guest access appears enabled.
- Entra external collaboration settings can block who is allowed to invite guests.
- Access reviews are the right governance handoff for periodic guest membership review.
- Sensitivity labels can govern Teams, Microsoft 365 Groups, privacy, external sharing, and unmanaged device behavior when configured.
- Audit, retention, eDiscovery, and legal compliance are handled in workbook 05.