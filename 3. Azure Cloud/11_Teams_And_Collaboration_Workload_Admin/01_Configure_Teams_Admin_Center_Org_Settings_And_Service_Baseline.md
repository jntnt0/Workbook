# 01_Configure_Teams_Admin_Center_Org_Settings_And_Service_Baseline

## Objective

Configure a baseline Microsoft Teams tenant posture from the Teams admin center, including core organization-wide settings, service readiness checks, admin access validation, and baseline documentation for later Teams policy workbooks.

## Lab Context

This workbook assumes a Microsoft 365 tenant with Teams enabled and at least one Teams admin account. The goal is not to configure every Teams workload policy yet. The goal is to establish the tenant-level baseline before moving into teams, channels, meetings, messaging, calling, apps, guests, compliance, and troubleshooting.

## Prerequisites

| Requirement | Notes |
|---|---|
| Microsoft 365 tenant | Teams service available |
| Admin account | Teams Administrator or Global Administrator |
| Test users | At least two licensed users |
| Teams licenses | Assigned to test users |
| Browser access | Microsoft Teams admin center |
| Optional PowerShell | MicrosoftTeams module installed |

## Target Scope

| Area | Baseline Decision |
|---|---|
| Teams admin center access | Confirmed |
| Teams service health | Reviewed |
| Org-wide Teams settings | Documented |
| Teams upgrade/coexistence | Teams-only baseline unless hybrid/Skype exists |
| External/guest posture | Reviewed only, detailed config in later workbook |
| Teams apps posture | Reviewed only, detailed config in later workbook |
| Policy baseline | Default/global policies documented |
| Audit trail | Changes recorded in workbook notes |

## Configuration Steps

| Step | Task | Admin Center / Device | Command / Location | PowerShell | Expected Result |
|---:|---|---|---|---|---|
| 1 | Sign in to Microsoft 365 admin center | Admin workstation | `https://admin.microsoft.com` | N/A | Admin portal opens successfully |
| 2 | Confirm Teams service is available | Microsoft 365 admin center | **Health** > **Service health** > **Microsoft Teams** | N/A | Teams shows healthy or known advisory is documented |
| 3 | Open Teams admin center | Admin workstation | `https://admin.teams.microsoft.com` | N/A | Teams admin center opens |
| 4 | Confirm admin role access | Teams admin center | Verify left navigation loads: **Teams**, **Users**, **Meetings**, **Messaging policies**, **Voice**, **Teams apps** | N/A | Admin has proper Teams management access |
| 5 | Record tenant identity baseline | Microsoft 365 admin center | **Settings** > **Org settings** > **Organization profile** | N/A | Tenant name, default domain, and technical contact are documented |
| 6 | Review Teams service settings landing areas | Teams admin center | Review **Teams**, **Users**, **Meetings**, **Messaging policies**, **Voice**, **Teams apps**, **Analytics & reports** | N/A | Admin confirms major Teams control areas are visible |
| 7 | Review organization-wide Teams settings | Teams admin center | **Org-wide settings** or equivalent tenant settings area, depending on admin center layout | N/A | Existing org-wide Teams configuration is reviewed and documented |
| 8 | Review Teams upgrade/coexistence mode | Teams admin center | **Teams** > **Teams upgrade settings** | `Get-CsTeamsUpgradeConfiguration` | Tenant upgrade mode is identified |
| 9 | Set Teams-only posture if no Skype hybrid requirement exists | Teams admin center | **Teams upgrade settings** > set tenant coexistence to Teams-only where applicable | `Grant-CsTeamsUpgradePolicy -PolicyName UpgradeToTeams -Global` | Tenant is aligned to Teams-only baseline |
| 10 | Review default Teams policy assignments | Teams admin center | **Users** > select test user > **Policies** | `Get-CsOnlineUser -Identity user@domain.com | Select DisplayName,TeamsUpgradeEffectiveMode,TeamsMessagingPolicy,TeamsMeetingPolicy,TeamsCallingPolicy` | Test user policy state is visible |
| 11 | Review global meeting policy baseline | Teams admin center | **Meetings** > **Meeting policies** > **Global** | `Get-CsTeamsMeetingPolicy -Identity Global` | Global meeting policy is documented, not deeply changed yet |
| 12 | Review global messaging policy baseline | Teams admin center | **Messaging policies** > **Global** | `Get-CsTeamsMessagingPolicy -Identity Global` | Global messaging policy is documented |
| 13 | Review global app permission policy baseline | Teams admin center | **Teams apps** > **Permission policies** > **Global** | `Get-CsTeamsAppPermissionPolicy -Identity Global` | Global app permission posture is documented |
| 14 | Review global app setup policy baseline | Teams admin center | **Teams apps** > **Setup policies** > **Global** | `Get-CsTeamsAppSetupPolicy -Identity Global` | Global app setup posture is documented |
| 15 | Review external access baseline only | Teams admin center | **Users** > **External access** | `Get-CsTenantFederationConfiguration` | External access state is documented for later governance workbook |
| 16 | Review guest access baseline only | Teams admin center | **Users** > **Guest access** | `Get-CsTeamsGuestMessagingConfiguration` | Guest access state is documented for later governance workbook |
| 17 | Confirm licensed test users appear in Teams admin center | Teams admin center | **Users** > **Manage users** > search test users | `Get-CsOnlineUser -Identity user@domain.com` | Licensed users are visible and Teams-enabled |
| 18 | Confirm Teams client access for test user | Test workstation / browser | `https://teams.microsoft.com` | N/A | Test user can sign in to Teams |
| 19 | Create a baseline test team if needed | Teams admin center or Teams client | **Teams** > **Manage teams** > **Add** | `New-Team -DisplayName "LAB-Teams-Baseline" -Description "Baseline Teams validation team"` | Test team exists |
| 20 | Add second test user to baseline team | Teams admin center or Teams client | Open team > manage members | `Add-TeamUser -GroupId <GroupId> -User user2@domain.com -Role Member` | Test member is added |
| 21 | Validate Teams admin reporting access | Teams admin center | **Analytics & reports** > **Usage reports** | N/A | Admin can access Teams reporting area |
| 22 | Record baseline decisions | Workbook notes | Document current settings and planned changes | N/A | Tenant baseline is captured before later policy changes |

## Validation

| Check | Command / Location | Expected Result |
|---|---|---|
| Teams admin center access | `https://admin.teams.microsoft.com` | Portal opens without access error |
| Teams service health | Microsoft 365 admin center > Service health | Teams is healthy or advisory is documented |
| Teams PowerShell connectivity | `Connect-MicrosoftTeams` | Admin connects successfully |
| Upgrade mode visible | `Get-CsTeamsUpgradeConfiguration` | Tenant upgrade configuration returns |
| Test user Teams state visible | `Get-CsOnlineUser -Identity user@domain.com` | User object returns with Teams properties |
| Global meeting policy visible | `Get-CsTeamsMeetingPolicy -Identity Global` | Global policy returns |
| Global messaging policy visible | `Get-CsTeamsMessagingPolicy -Identity Global` | Global policy returns |
| Test team exists | `Get-Team -DisplayName "LAB-Teams-Baseline"` | Baseline test team returns |
| Test user can sign in | Teams web client | User reaches Teams interface |

## Troubleshooting

| Symptom | Likely Cause | Fix |
|---|---|---|
| Teams admin center will not open | Missing admin role | Assign Teams Administrator or Global Administrator |
| User missing from Teams admin center | License not assigned or provisioning delay | Assign Teams license and wait for service provisioning |
| PowerShell cmdlets unavailable | MicrosoftTeams module missing | Run `Install-Module MicrosoftTeams -Scope CurrentUser` |
| `Connect-MicrosoftTeams` fails | MFA, stale session, or blocked admin account | Re-authenticate, use compliant browser, verify admin account status |
| Teams client sign-in fails | License, Conditional Access, or service issue | Check license, sign-in logs, Conditional Access, and service health |
| Teams policy changes not visible immediately | Policy propagation delay | Wait, sign out/in, test again later |
| Upgrade mode conflicts with expected Teams-only behavior | Legacy Skype/Teams coexistence setting | Review Teams upgrade settings and confirm no hybrid dependency |
| Reports unavailable | Insufficient role or reporting delay | Confirm role access and allow reporting data to populate |

## Rollback / Cleanup

| Change | Rollback |
|---|---|
| Test team created | Delete `LAB-Teams-Baseline` from Teams admin center or Teams client |
| Test user added to team | Remove user from team membership |
| Tenant upgrade mode changed | Restore previous coexistence mode only if required by legacy Skype/Teams design |
| Policy documentation captured | Keep as baseline record |
| PowerShell session opened | `Disconnect-MicrosoftTeams` |

## Notes

- Do not overconfigure meetings, messaging, calling, apps, guests, external access, retention, or compliance in this workbook.
- This workbook is the tenant baseline.
- Later workbooks handle detailed Teams policy configuration.
- Default/global policies should be reviewed and documented before creating custom policies.
- Teams changes can take time to propagate.
- Always test with licensed non-admin users, not only admin accounts.