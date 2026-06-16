# 05_Configure_Teams_Compliance_Retention_eDiscovery_And_Audit_Handoff

## Objective

Configure and validate the Microsoft Purview handoff points for Microsoft Teams retention, eDiscovery, audit, legal/compliance investigation readiness, and administrative evidence collection.

## Lab Context

This workbook builds on the Teams tenant baseline, Teams membership, Teams policy, and Teams external collaboration workbooks. The goal is to understand where Teams compliance controls actually live and how Teams content is retained, searched, audited, and handed off to compliance administrators.

Teams compliance is not managed only from the Teams admin center. Teams messages, channel messages, chat messages, files, meeting recordings, transcripts, and audit events cross Teams, Exchange, SharePoint, OneDrive, Entra ID, and Microsoft Purview.

## Prerequisites

| Requirement | Notes |
|---|---|
| Microsoft 365 tenant | Teams enabled |
| Admin role for Teams | Teams Administrator or Global Administrator |
| Compliance role | Compliance Administrator, eDiscovery Manager, or Purview role group membership |
| Audit access | Audit Reader or Audit Manager role where required |
| Test users | At least two licensed Teams users |
| Test team | Example: `LAB-Teams-Channel-Membership` |
| Test content | Channel posts, chats, meeting, file upload |
| Microsoft Purview access | `https://purview.microsoft.com` |
| Optional PowerShell | MicrosoftTeams, ExchangeOnlineManagement, Microsoft Graph modules |

## Target Scope

| Area | Baseline Decision |
|---|---|
| Teams retention | Configure or document retention for Teams chats and channel messages |
| Files retention | Document SharePoint/OneDrive retention dependency |
| Meeting recordings | Document OneDrive/SharePoint storage and retention handoff |
| eDiscovery | Create or validate a standard case workflow |
| Audit | Validate Teams admin/user activity search |
| Legal hold | Document handoff to Purview eDiscovery |
| DLP/compliance | Document handoff only unless licensing supports deeper configuration |
| Production impact | Use lab users/team and avoid broad production retention changes |

## Teams Compliance Data Map

| Teams Data Type | Compliance Location | Notes |
|---|---|---|
| 1:1 chat messages | Purview retention/eDiscovery, Exchange substrate mailbox storage | Search through Purview |
| Group chat messages | Purview retention/eDiscovery, Exchange substrate mailbox storage | Search through Purview |
| Standard channel messages | Purview retention/eDiscovery, group mailbox substrate | Associated with Microsoft 365 Group |
| Private channel messages | Purview retention/eDiscovery, private channel mailbox substrate | Separate compliance boundary from standard channel |
| Shared channel messages | Purview retention/eDiscovery, shared channel storage model | External collaboration may complicate ownership |
| Teams files | SharePoint team site or OneDrive | Governed by SharePoint/OneDrive retention and sharing |
| Meeting recordings | OneDrive or SharePoint | Retention depends on storage location and policies |
| Meeting transcripts | Teams/Purview supported locations | Validate tenant feature availability |
| Audit events | Microsoft Purview Audit | Requires proper audit role and licensing |

## Configuration Steps

| Step | Task | Admin Center / Device | Command / Location | PowerShell | Expected Result |
|---:|---|---|---|---|---|
| 1 | Confirm Teams admin access | Admin workstation | `https://admin.teams.microsoft.com` | `Connect-MicrosoftTeams` | Teams admin access works |
| 2 | Confirm Purview portal access | Admin workstation | `https://purview.microsoft.com` | N/A | Purview portal opens |
| 3 | Confirm compliance role membership | Microsoft Purview portal | **Settings** > **Roles and scopes** > **Role groups** | N/A | Admin is in appropriate compliance role group |
| 4 | Confirm audit role access | Microsoft Purview portal | **Audit** | N/A | Audit search page is accessible |
| 5 | Confirm test team exists | Admin workstation | Teams admin center > **Teams** > **Manage teams** | `$Team = Get-Team -DisplayName "LAB-Teams-Channel-Membership"; $Team.GroupId` | Test team is available |
| 6 | Generate Teams channel test message | Teams client | Post message in standard channel | N/A | Channel message exists for retention/eDiscovery validation |
| 7 | Generate Teams chat test message | Teams client | Send 1:1 chat between test users | N/A | Chat message exists |
| 8 | Generate meeting artifact | Teams client | Schedule and join test meeting | N/A | Meeting activity exists |
| 9 | Upload test file to channel | Teams client | Channel > **Files** > upload test file | N/A | File exists in team-connected SharePoint site |
| 10 | Review existing retention policies | Microsoft Purview portal | **Data lifecycle management** > **Microsoft 365** > **Retention policies** | N/A | Existing retention policy baseline is documented |
| 11 | Create lab Teams message retention policy | Microsoft Purview portal | **Data lifecycle management** > **Retention policies** > **New retention policy** | N/A | Lab retention policy wizard starts |
| 12 | Name retention policy | Microsoft Purview portal | Policy name: `LAB-Teams-Messages-Retention-Baseline` | N/A | Policy has clear lab name |
| 13 | Select Teams locations | Microsoft Purview portal | Choose **Teams channel messages** and **Teams chats** | N/A | Teams message locations are selected |
| 14 | Scope retention policy to lab users/team where possible | Microsoft Purview portal | Adaptive/static scope depending on license and tenant capability | N/A | Policy scope avoids broad tenant-wide impact |
| 15 | Configure retention duration | Microsoft Purview portal | Retain for selected lab duration, then delete or do nothing based on lab decision | N/A | Retention behavior is configured |
| 16 | Review before enabling | Microsoft Purview portal | Review policy summary | N/A | Admin confirms policy does not target unintended production users |
| 17 | Create policy | Microsoft Purview portal | Submit retention policy | N/A | Retention policy is created |
| 18 | Document Teams file retention dependency | Microsoft Purview portal | Data lifecycle management > retention policies | N/A | File retention noted as SharePoint/OneDrive policy area |
| 19 | Review SharePoint/OneDrive retention policies | Microsoft Purview portal | Retention policies > SharePoint sites / OneDrive accounts | N/A | File retention baseline is documented |
| 20 | Identify team SharePoint site | Teams client or SharePoint admin center | Team channel > Files > Open in SharePoint | N/A | Team-connected SharePoint site URL is identified |
| 21 | Confirm meeting recording storage model | Teams admin center / Teams client | Meeting policy and recording location behavior | N/A | Recording storage handoff to OneDrive/SharePoint is documented |
| 22 | Review meeting recording policy dependency | Teams admin center | **Meetings** > **Meeting policies** > lab/global policy | `Get-CsTeamsMeetingPolicy -Identity "LAB-Teams-Meeting-Baseline"` | Recording/transcription policy state is known |
| 23 | Review eDiscovery permissions | Microsoft Purview portal | **eDiscovery** > role access | N/A | Admin can access eDiscovery area |
| 24 | Create lab eDiscovery case | Microsoft Purview portal | **eDiscovery** > **Cases** > create case | N/A | Case exists, for example `LAB-Teams-eDiscovery-Case` |
| 25 | Add case members | Microsoft Purview portal | Case > **Settings** > members | N/A | Compliance admin/test reviewer is added |
| 26 | Create eDiscovery search | Microsoft Purview portal | Case > **Searches** > create search | N/A | Search wizard opens |
| 27 | Select Teams-related custodians or locations | Microsoft Purview portal | Add test users and/or team-related locations | N/A | Search scope includes test Teams data |
| 28 | Add keyword query for test content | Microsoft Purview portal | Query known phrase from test chat/channel message | N/A | Query targets known lab evidence |
| 29 | Run eDiscovery search estimate | Microsoft Purview portal | Run search / estimate results | N/A | Search returns estimated Teams results |
| 30 | Review search results | Microsoft Purview portal | Search results preview if available | N/A | Matching test content appears or result count is documented |
| 31 | Document export handoff | Microsoft Purview portal | eDiscovery case > export options | N/A | Export path and role requirements are documented |
| 32 | Review legal hold capability | Microsoft Purview portal | eDiscovery case > holds | N/A | Hold workflow is identified |
| 33 | Create lab hold only if approved | Microsoft Purview portal | Case > **Holds** > create hold for test users/locations | N/A | Hold is created only for lab scope |
| 34 | Validate hold status | Microsoft Purview portal | Case > Holds | N/A | Hold shows active or pending status |
| 35 | Search Teams audit activity | Microsoft Purview portal | **Audit** > search activities related to Teams | N/A | Teams activity search is configured |
| 36 | Filter audit by test user | Microsoft Purview portal | Audit search > user = test user | N/A | Audit search targets lab user |
| 37 | Filter audit by time window | Microsoft Purview portal | Audit search > date/time covering lab actions | N/A | Audit window includes generated activity |
| 38 | Run audit search | Microsoft Purview portal | Submit search | N/A | Audit search runs |
| 39 | Review Teams audit events | Microsoft Purview portal | Audit results | N/A | Teams-related activity appears if available |
| 40 | Review Teams admin change audit | Microsoft Purview portal | Audit search for policy/admin changes | N/A | Admin changes are searchable |
| 41 | Document DLP handoff | Microsoft Purview portal | **Data loss prevention** > Policies | N/A | DLP location and owner are documented |
| 42 | Review Teams DLP availability | Microsoft Purview portal | DLP policy locations | N/A | Teams chat/channel DLP availability is confirmed or documented |
| 43 | Document Communication Compliance handoff | Microsoft Purview portal | **Communication compliance** | N/A | Communication compliance owner and licensing dependency documented |
| 44 | Review sensitivity label handoff | Microsoft Purview portal | **Information Protection** > **Sensitivity labels** | N/A | Teams/M365 Groups label governance is documented |
| 45 | Review access review handoff | Entra admin center | **Identity Governance** > **Access reviews** | N/A | Guest/team membership review handoff is documented |
| 46 | Build evidence record | Workbook notes | Record policy names, case name, search query, audit search, and findings | N/A | Compliance handoff evidence is complete |
| 47 | Validate no broad production impact | Microsoft Purview portal | Review retention policies, holds, and searches | N/A | Only intended lab scope is affected |
| 48 | Disconnect PowerShell sessions | Admin workstation | PowerShell | `Disconnect-MicrosoftTeams` | Admin session is closed |

## Validation

| Check | Command / Location | Expected Result |
|---|---|---|
| Purview portal accessible | `https://purview.microsoft.com` | Portal opens |
| Compliance role available | Purview > Roles and scopes | Admin has needed compliance role |
| Audit search accessible | Purview > Audit | Audit page opens |
| Test team exists | `Get-Team -DisplayName "LAB-Teams-Channel-Membership"` | Team returns |
| Test content created | Teams client | Chat/channel/file/meeting activity exists |
| Retention policy created | Purview > Data lifecycle management > Retention policies | `LAB-Teams-Messages-Retention-Baseline` exists |
| Teams locations selected | Retention policy details | Teams chats and/or Teams channel messages are selected |
| SharePoint file location identified | Teams > Files > Open in SharePoint | Team site opens |
| eDiscovery case exists | Purview > eDiscovery > Cases | `LAB-Teams-eDiscovery-Case` exists |
| eDiscovery search runs | Case > Searches | Search completes or estimate returns |
| Audit search runs | Purview > Audit | Audit results return or no-result finding is documented |
| Hold status reviewed | eDiscovery case > Holds | Hold is active, pending, or not created by design |
| DLP handoff documented | Purview > DLP | Teams DLP governance owner is recorded |
| Sensitivity label handoff documented | Purview > Information Protection | Label governance owner is recorded |

## Troubleshooting

| Symptom | Likely Cause | Fix |
|---|---|---|
| Purview portal opens but compliance features are missing | Admin lacks Purview role group membership | Add admin to Compliance Administrator, eDiscovery Manager, Audit Reader, or required role group |
| Audit search unavailable | Audit role missing or audit not available for tenant/licensing | Assign audit role and verify licensing |
| eDiscovery case creation unavailable | eDiscovery role missing | Add admin to eDiscovery Manager role group |
| Teams messages do not appear in search immediately | Indexing delay | Wait and rerun search |
| Search returns no Teams content | Wrong location, wrong custodian, wrong keyword, or content not indexed yet | Confirm user/team scope, keyword, and date range |
| Retention policy targets too much content | Scope too broad | Stop and revise policy before production use |
| Hold affects unintended users | Hold scope too broad | Disable/remove hold and recreate with narrow scope |
| Meeting recording not retained as expected | Recording stored in OneDrive/SharePoint, not as normal chat message | Apply SharePoint/OneDrive retention policy to recording storage location |
| Files not governed by Teams message retention | Teams files live in SharePoint/OneDrive | Use SharePoint/OneDrive retention locations |
| Guest activity missing from expected view | Guest activity may appear under guest identity or audit delay | Search by guest UPN/object and broaden time range |
| Admin policy changes not visible immediately in audit | Audit ingestion delay | Wait and rerun audit search |
| DLP options unavailable for Teams | Licensing or role limitation | Confirm licensing and Purview permissions |

## Rollback / Cleanup

| Change | Rollback |
|---|---|
| Lab retention policy created | Disable or delete `LAB-Teams-Messages-Retention-Baseline` after lab validation |
| Lab eDiscovery case created | Close or delete case if no longer needed |
| Lab eDiscovery search created | Delete search or keep as evidence according to lab policy |
| Lab hold created | Release hold before deleting the case |
| Test Teams messages created | Delete test content if allowed by retention policy |
| Test file uploaded | Delete file from Teams/SharePoint if allowed |
| Meeting recording created | Delete recording from OneDrive/SharePoint if allowed |
| Audit search run | No rollback needed |
| Role assignment changed | Remove temporary compliance role membership after validation |

## Notes

- Teams compliance controls are split across Teams, Purview, Exchange, SharePoint, OneDrive, and Entra ID.
- Teams chat and channel message retention is configured in Microsoft Purview, not only in Teams admin center.
- Teams files are retained through SharePoint and OneDrive retention policies.
- Meeting recordings are stored in OneDrive or SharePoint and should be governed there.
- eDiscovery is a Purview workflow. Teams admin center is not the investigation workspace.
- Legal hold should be handled carefully and scoped tightly.
- Do not create broad tenant-wide holds or retention changes during lab work.
- Audit search results can take time to appear.
- DLP, Communication Compliance, Insider Risk, sensitivity labels, and access reviews are separate governance controls that may require additional licensing.
- Compliance workbooks should document the handoff owner, scope, policy names, and test evidence.