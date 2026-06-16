03_Monitor_Service_Health_Message_Center_Usage_Reports_And_M365_Backup.md
# 03_Monitor_Service_Health_Message_Center_Usage_Reports_And_M365_Backup

# 03_Monitor_Service_Health_Message_Center_Usage_Reports_And_M365_Backup_Index
03_Monitor_Service_Health_Message_Center_Usage_Reports_And_M365_Backup.md
03_Monitor_Service_Health_Message_Center_Usage_Reports_And_M365_Backup
03_Monitor_Service_Health_Message_Center_Usage_Reports_And_M365_Backup_Source_Basis
03_Monitor_Service_Health_Message_Center_Usage_Reports_And_M365_Backup_Mental_Model
03_Monitor_Service_Health_Message_Center_Usage_Reports_And_M365_Backup_Planning_Table
03_Monitor_Service_Health_Message_Center_Usage_Reports_And_M365_Backup_Configuration_Checklist
03_Monitor_Service_Health_Message_Center_Usage_Reports_And_M365_Backup_Service_Health_Skeleton
03_Monitor_Service_Health_Message_Center_Usage_Reports_And_M365_Backup_Message_Center_Skeleton
03_Monitor_Service_Health_Message_Center_Usage_Reports_And_M365_Backup_Usage_Reports_Skeleton
03_Monitor_Service_Health_Message_Center_Usage_Reports_And_M365_Backup_M365_Backup_Skeleton
03_Monitor_Service_Health_Message_Center_Usage_Reports_And_M365_Backup_Graph_Service_Announcements_Skeleton
03_Monitor_Service_Health_Message_Center_Usage_Reports_And_M365_Backup_Verification_Commands
03_Monitor_Service_Health_Message_Center_Usage_Reports_And_M365_Backup_Rollback
03_Monitor_Service_Health_Message_Center_Usage_Reports_And_M365_Backup_Failure_Checks
03_Monitor_Service_Health_Message_Center_Usage_Reports_And_M365_Backup_Related_Labs

# 03_Monitor_Service_Health_Message_Center_Usage_Reports_And_M365_Backup_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | How to check Microsoft 365 service health | Service health dashboard, active issues, incident details, advisories, history, and notifications |
| Microsoft Learn | Microsoft 365 Message center | Upcoming changes, major updates, relevance, filtering, preferences, and email digest |
| Microsoft Learn | Microsoft 365 activity reports | Usage reporting, report dashboard, active user reports, workload reports, export behavior, and privacy settings |
| Microsoft Learn | Microsoft 365 Backup overview | Backup scope, SharePoint, OneDrive, Exchange Online, restore scenarios, retention, audit, and billing concepts |
| Microsoft Learn | Microsoft 365 admin center overview | Health, Reports, Support, and Admin center navigation |
| Microsoft Graph PowerShell | Service announcements and reports cmdlets | Repeatable service health, Message center, and usage reporting inventory |
| Microsoft 365 operational practice | Incident monitoring and change review | Establishes tenant monitoring rhythm and response ownership |
| Backup operational practice | Recovery readiness | Validates whether Microsoft 365 Backup is configured, licensed, tested, and owned |

# 03_Monitor_Service_Health_Message_Center_Usage_Reports_And_M365_Backup_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Service health | Microsoft 365 tenant-specific status view for active incidents, advisories, and service degradation |
| Incident | Service issue affecting availability or functionality |
| Advisory | Service issue or change notice that may affect users but is usually less severe than an incident |
| Health history | Past service incidents and advisories for the tenant |
| Message center | Change notification center for Microsoft 365 features, deprecations, retirement notices, admin action requirements, and major updates |
| Major update | Message center item likely to affect users, admins, compliance, service behavior, or business process |
| Message center relevance | Microsoft recommendation about whether a message applies to the tenant |
| Email digest | Scheduled Message center summary sent to selected recipients |
| Usage reports | Admin center reports showing adoption and activity across Microsoft 365 services |
| Active users report | Usage report showing users active across Microsoft 365 services over a reporting window |
| Report privacy | Tenant setting that can hide or show identifiable user, group, and site names in reports |
| Microsoft 365 Backup | Microsoft service for backup and restore of supported Microsoft 365 data such as Exchange Online, SharePoint, and OneDrive |
| Restore point | Available backup state used to recover protected data |
| Backup policy | Configuration that defines what is protected and how backup behavior applies |
| Recovery validation | Test proving that protected data can actually be restored |
| First rule | Service health tells you if Microsoft is broken before you blame your tenant |
| Blunt rule | Message center is where Microsoft tells you future breakage before it becomes your ticket queue |

# 03_Monitor_Service_Health_Message_Center_Usage_Reports_And_M365_Backup_Planning_Table
| Item | Example | Decision |
|---|---|---|
| Tenant name | `contoso.onmicrosoft.com` | `<tenant>.onmicrosoft.com` |
| Tenant ID | `00000000-0000-0000-0000-000000000000` | `<tenant-id>` |
| Admin account | `admin@contoso.com` | `<admin-upn>` |
| Service health owner | `m365ops@contoso.com` | `<service-health-owner>` |
| Message center owner | `m365change@contoso.com` | `<message-center-owner>` |
| Usage reports owner | `reportingadmin@contoso.com` | `<reports-owner>` |
| Backup owner | `backupadmin@contoso.com` | `<backup-owner>` |
| Security owner | `securityadmin@contoso.com` | `<security-owner>` |
| Compliance owner | `complianceadmin@contoso.com` | `<compliance-owner>` |
| Help desk distribution list | `helpdesk@contoso.com` | `<helpdesk-dl>` |
| Change review cadence | Weekly | `<change-review-cadence>` |
| Service health review cadence | Daily | `<health-review-cadence>` |
| Usage report review cadence | Monthly | `<usage-review-cadence>` |
| Backup validation cadence | Quarterly | `<backup-test-cadence>` |
| Message center digest recipients | IT admins and service owners | `<digest-recipients>` |
| Target workloads for usage reports | Exchange, Teams, SharePoint, OneDrive, M365 Apps | `<workloads>` |
| Report privacy stance | Hide user identifiable info unless approved | `<privacy-setting>` |
| Backup scope | Exchange, SharePoint, OneDrive | `<backup-scope>` |
| Evidence path | `.\Evidence\M365-Health-Reports-Backup` | `<evidence-path>` |
| Rollback standard | Revert preferences and notification settings | `<rollback-plan>` |

# 03_Monitor_Service_Health_Message_Center_Usage_Reports_And_M365_Backup_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm admin account | Admin Workstation | Portal account menu | Correct admin UPN is signed in |
| 2 | Confirm correct tenant | Admin Workstation | Portal tenant switcher | Correct Microsoft 365 tenant is active |
| 3 | Confirm Health menu access | Admin Workstation | Microsoft 365 admin center > Health | Health section opens |
| 4 | Open Service health | Admin Workstation | Health > Service health | Service health dashboard opens |
| 5 | Review active incidents | Admin Workstation | Health > Service health > Active issues | Active incidents and advisories are visible |
| 6 | Review health history | Admin Workstation | Health > Service health > History | Recent resolved issues are visible |
| 7 | Open issue details | Admin Workstation | Select active issue | Scope, impact, status, and latest update are visible |
| 8 | Identify affected workloads | Admin Workstation | Issue details | Affected services are documented |
| 9 | Configure service health notifications if available | Admin Workstation | Service health notification settings | Service owners receive outage notifications |
| 10 | Document service health owner | Operator | Notes | Incident review owner is recorded |
| 11 | Open Message center | Admin Workstation | Health > Message center | Message center opens |
| 12 | Review major updates | Admin Workstation | Message center > Major updates | Major tenant-impacting changes are visible |
| 13 | Review all active messages | Admin Workstation | Message center list | Active change notifications are visible |
| 14 | Filter Message center by workload | Admin Workstation | Message center filters | Relevant service changes are grouped |
| 15 | Review high impact messages | Admin Workstation | Message center sorting/filtering | Required admin actions are identified |
| 16 | Configure Message center preferences | Admin Workstation | Message center > Preferences | Preferred services and email digest behavior are configured |
| 17 | Add digest recipients | Admin Workstation | Message center > Preferences | Change owners receive digest emails |
| 18 | Document Message center review cadence | Operator | Notes | Change review rhythm is recorded |
| 19 | Open Reports dashboard | Admin Workstation | Reports > Usage | Usage dashboard opens |
| 20 | Review active users report | Admin Workstation | Reports > Usage > Active users | Tenant active user trend is visible |
| 21 | Review Exchange usage report | Admin Workstation | Reports > Usage > Exchange | Mailbox usage and activity are visible |
| 22 | Review Teams usage report | Admin Workstation | Reports > Usage > Teams | Teams usage and activity are visible |
| 23 | Review SharePoint usage report | Admin Workstation | Reports > Usage > SharePoint | Site usage is visible |
| 24 | Review OneDrive usage report | Admin Workstation | Reports > Usage > OneDrive | OneDrive activity and storage are visible |
| 25 | Review Microsoft 365 Apps usage report | Admin Workstation | Reports > Usage > Microsoft 365 Apps | App activation and usage are visible |
| 26 | Confirm reporting time window | Admin Workstation | Reports filter | 7, 30, 90, or 180 day period is selected |
| 27 | Export report evidence if needed | Admin Workstation | Reports export option | CSV/report export is saved |
| 28 | Review report privacy setting | Admin Workstation | Settings > Org settings > Reports | User identifiable report data setting is known |
| 29 | Set report privacy stance | Admin Workstation | Settings > Org settings > Reports | Privacy setting matches organization policy |
| 30 | Document usage report owner | Operator | Notes | Reporting owner and cadence are recorded |
| 31 | Open Microsoft 365 Backup area | Admin Workstation | Microsoft 365 admin center > Backup or Setup > Backup | Backup management entry point is identified |
| 32 | Confirm Microsoft 365 Backup availability | Admin Workstation | Backup page / service setup | Tenant backup feature availability is known |
| 33 | Review supported workloads | Admin Workstation | Backup page | Exchange, SharePoint, and OneDrive backup scope is understood |
| 34 | Confirm billing prerequisites | Admin Workstation | Backup setup / Billing | Backup billing readiness is documented |
| 35 | Configure backup if approved | Admin Workstation | Backup setup wizard | Backup protection is enabled for intended workloads |
| 36 | Record backup policy scope | Admin Workstation | Backup policy page | Protected users/sites/mailboxes are documented |
| 37 | Validate restore workflow | Admin Workstation | Backup restore flow | Restore path is understood and documented |
| 38 | Run recovery test if approved | Admin Workstation | Backup restore test | Test restore succeeds or issue is documented |
| 39 | Connect Microsoft Graph PowerShell | Admin Workstation | `Connect-MgGraph -Scopes "ServiceHealth.Read.All","ServiceMessage.Read.All","Reports.Read.All","Organization.Read.All"` | Graph session connects |
| 40 | Export service health overview | Admin Workstation | `Get-MgAdminServiceAnnouncementHealthOverview` | Service health overview is exported |
| 41 | Export service health issues | Admin Workstation | `Get-MgAdminServiceAnnouncementIssue` | Active and recent issues are exported |
| 42 | Export Message center messages | Admin Workstation | `Get-MgAdminServiceAnnouncementMessage` | Message center inventory is exported |
| 43 | Export active user report | Admin Workstation | Graph report cmdlet or portal export | Active user report evidence is saved |
| 44 | Save screenshots and exports | Admin Workstation | `<evidence-path>` | Evidence folder contains baseline records |
| 45 | Document operational runbook owner | Operator | Notes | Health, Message center, reports, and backup owners are recorded |
| 46 | Document escalation process | Operator | Notes | Incident and change escalation path is known |
| 47 | Document review cadence | Operator | Notes | Daily, weekly, monthly, and quarterly cadences are recorded |
| 48 | Review dashboard cards | Admin Workstation | Microsoft 365 admin center > Home | Service health and Message center cards are pinned if useful |
| 49 | Add Health dashboard card if missing | Admin Workstation | Home > Add card | Service health is visible from home dashboard |
| 50 | Document completion state | Operator | Notes | Monitoring, change visibility, reporting, and backup baseline is complete |

# 03_Monitor_Service_Health_Message_Center_Usage_Reports_And_M365_Backup_Service_Health_Skeleton
```text
Portal path:

Microsoft 365 admin center
Health
Service health
```

| Area | What To Review | Expected Result |
|---|---|---|
| Overview | Current service status | Active issues are visible |
| Active issues | Incidents and advisories | Impact, affected service, and latest update are visible |
| Issue details | Scope and progress | Admin understands if the issue affects the tenant |
| History | Resolved incidents and advisories | Recent resolved issues are available |
| Notifications | Service health alert delivery | Service owners receive notifications if configured |
| Report an issue | Tenant issue reporting path | Admin knows how to submit support context |

Service health review capture:

| Field | Value |
|---|---|
| Review date | `<date>` |
| Reviewed by | `<admin-upn>` |
| Active incidents | `<count>` |
| Active advisories | `<count>` |
| Affected workloads | `<workloads>` |
| User impact | `<impact-summary>` |
| Microsoft status | `<investigating/restoring/resolved>` |
| Tenant action required | `<yes-no>` |
| Escalated to help desk | `<yes-no>` |
| Evidence path | `<evidence-path>` |

Service health operating rule:

```text
Check Service health before troubleshooting a broad tenant issue.
If Microsoft reports an active incident for the affected workload, document it and adjust the incident response.
Do not waste time rebuilding local settings when the service itself is degraded.
```

# 03_Monitor_Service_Health_Message_Center_Usage_Reports_And_M365_Backup_Message_Center_Skeleton
```text
Portal path:

Microsoft 365 admin center
Health
Message center
```

| Area | What To Review | Expected Result |
|---|---|---|
| Major updates | High-impact Microsoft 365 changes | Admin action items are visible |
| All messages | Full change notification inventory | Upcoming changes are visible |
| Favorites | Tracked messages | Important changes are easy to revisit |
| Preferences | Services, digest, and email recipients | Relevant owners receive messages |
| Filters | Workload, impact, action required, date | Messages can be grouped operationally |
| Message detail | Timing, affected service, impact, action | Change can be assigned to an owner |

Message center triage table:

| Message ID | Workload | Title | Impact | Action Required | Owner | Due Date | Status |
|---|---|---|---|---|---|---|---|
| `<message-id>` | `<service>` | `<title>` | `<high/medium/low>` | `<yes-no>` | `<owner>` | `<date>` | `<new/reviewing/done>` |

Configuration checklist:

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Open Message center | Admin Workstation | Health > Message center | Message center opens |
| 2 | Review major updates | Admin Workstation | Message center > Major updates | High-impact changes are visible |
| 3 | Filter by workload | Admin Workstation | Message center filters | Relevant service changes are grouped |
| 4 | Identify action-required items | Admin Workstation | Filter: Action required | Required admin changes are visible |
| 5 | Assign message owner | Operator | Notes / tracker | Each high-impact message has an owner |
| 6 | Configure digest recipients | Admin Workstation | Message center > Preferences | Digest goes to correct owners |
| 7 | Save review evidence | Operator | Screenshot / export / notes | Message center review is documented |

Operational rule:

```text
Message center is the tenant change radar.
Treat major updates like planned change tickets, not random FYI emails.
```

# 03_Monitor_Service_Health_Message_Center_Usage_Reports_And_M365_Backup_Usage_Reports_Skeleton
```text
Portal path:

Microsoft 365 admin center
Reports
Usage
```

| Report Area | What It Shows | Operational Use |
|---|---|---|
| Active users | User activity across Microsoft 365 services | Adoption and license utilization review |
| Email activity | Exchange usage | Mailbox activity and service adoption |
| Teams activity | Teams usage | Collaboration adoption |
| SharePoint activity | Site usage | Collaboration and storage review |
| OneDrive activity | User file activity and storage | Storage and adoption review |
| Microsoft 365 Apps | App activation and usage | Desktop app deployment/adoption |
| Groups activity | Group usage | Group lifecycle and cleanup |
| License activity | License usage | Licensing optimization |

Usage report review capture:

| Field | Value |
|---|---|
| Review date | `<date>` |
| Reviewed by | `<admin-upn>` |
| Reporting window | `<7/30/90/180-days>` |
| Active user count | `<count>` |
| Exchange active users | `<count>` |
| Teams active users | `<count>` |
| SharePoint active users | `<count>` |
| OneDrive active users | `<count>` |
| Microsoft 365 Apps active users | `<count>` |
| Unused licenses flagged | `<yes-no>` |
| Adoption issue flagged | `<yes-no>` |
| Evidence path | `<evidence-path>` |

Configuration checklist:

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Open usage reports | Admin Workstation | Reports > Usage | Reports dashboard opens |
| 2 | Select report window | Admin Workstation | 7, 30, 90, or 180 days | Reporting window is intentional |
| 3 | Review active users | Admin Workstation | Reports > Active users | Active user trend is visible |
| 4 | Review workload reports | Admin Workstation | Exchange, Teams, SharePoint, OneDrive, Apps | Workload usage is visible |
| 5 | Export reports if required | Admin Workstation | Export option | Report evidence is saved |
| 6 | Review privacy setting | Admin Workstation | Settings > Org settings > Reports | User identifiable data stance is known |
| 7 | Document findings | Operator | Notes | Usage trends and license risks are recorded |

Graph report starter:

```powershell
# Run from an admin workstation.
# Purpose: connect with reporting permissions for Microsoft 365 usage reports.

$EvidencePath = ".\Evidence\M365-Health-Reports-Backup"
New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Connect-MgGraph -Scopes "Reports.Read.All","Organization.Read.All"

Get-MgContext |
    Tee-Object "$EvidencePath\graph-context-reports.txt"

# Microsoft Graph Reports cmdlets can export report content depending on module version and report endpoint support.
# If a report cmdlet is unavailable in the installed module, use the Microsoft 365 admin center export button.

Get-Command -Module Microsoft.Graph.Reports |
    Sort-Object Name |
    Tee-Object "$EvidencePath\available-graph-report-cmdlets.txt"
```

Operational rule:

```text
Usage reports are not just adoption charts.
Use them to find dead licenses, unused services, abnormal activity drops, and training gaps.
```

# 03_Monitor_Service_Health_Message_Center_Usage_Reports_And_M365_Backup_M365_Backup_Skeleton
```text
Portal path:

Microsoft 365 admin center
Backup

Alternative discovery path:
Microsoft 365 admin center
Setup
Search: Backup
```

| Backup Area | What To Review | Expected Result |
|---|---|---|
| Availability | Whether Microsoft 365 Backup is available in tenant | Feature availability is known |
| Billing | Pay-as-you-go or required billing setup | Billing prerequisite is documented |
| Workload scope | Exchange, SharePoint, OneDrive | Supported protection scope is understood |
| Backup policy | What users, sites, or mailboxes are protected | Protection scope is documented |
| Restore flow | How restore requests are initiated | Admin knows the recovery workflow |
| Audit | Backup and restore operation visibility | Recovery actions are traceable |
| Test restore | Recovery validation | Restore is proven, not assumed |

Backup planning table:

| Item | Example | Decision |
|---|---|---|
| Backup owner | `backupadmin@contoso.com` | `<backup-owner>` |
| Billing owner | `billingadmin@contoso.com` | `<billing-owner>` |
| Protected Exchange scope | Executive mailboxes | `<exchange-scope>` |
| Protected SharePoint scope | Critical sites | `<sharepoint-scope>` |
| Protected OneDrive scope | Executive OneDrives | `<onedrive-scope>` |
| Restore approver | Compliance owner | `<restore-approver>` |
| Recovery test cadence | Quarterly | `<backup-test-cadence>` |
| Restore evidence path | `.\Evidence\M365-Backup-Restore` | `<restore-evidence-path>` |
| Business risk | Accidental deletion, ransomware, user error | `<risk>` |

Configuration checklist:

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Open Microsoft 365 Backup | Admin Workstation | Admin center > Backup | Backup page opens or availability issue is documented |
| 2 | Confirm billing prerequisites | Admin Workstation | Backup setup / Billing | Billing readiness is known |
| 3 | Review supported workloads | Admin Workstation | Backup page | Exchange, SharePoint, and OneDrive support is understood |
| 4 | Identify critical data scope | Operator | Planning table | Protected scope is defined |
| 5 | Configure backup if approved | Admin Workstation | Backup setup wizard | Backup protection is enabled |
| 6 | Record protected objects | Admin Workstation | Backup policy page | Protected mailboxes/sites/users are documented |
| 7 | Review restore flow | Admin Workstation | Backup restore option | Restore path is understood |
| 8 | Run test restore if approved | Admin Workstation | Restore test | Restore succeeds or issue is documented |
| 9 | Document restore evidence | Operator | Screenshots / notes | Recovery proof is stored |
| 10 | Assign backup review owner | Operator | Notes | Backup ownership is recorded |

Operational rule:

```text
A backup product is not real until a restore has been tested.
Do not list Microsoft 365 Backup as complete unless recovery evidence exists.
```

# 03_Monitor_Service_Health_Message_Center_Usage_Reports_And_M365_Backup_Graph_Service_Announcements_Skeleton
```powershell
# Run from an admin workstation.
# Purpose: export Service health and Message center data with Microsoft Graph PowerShell.

$EvidencePath = ".\Evidence\M365-Health-Reports-Backup"
New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Connect-MgGraph -Scopes "ServiceHealth.Read.All","ServiceMessage.Read.All","Reports.Read.All","Organization.Read.All"

Get-MgContext |
    Tee-Object "$EvidencePath\graph-context-service-announcements.txt"

# Service health overview.
Get-MgAdminServiceAnnouncementHealthOverview |
    Sort-Object Service |
    Format-Table Service,Status,Id -AutoSize |
    Tee-Object "$EvidencePath\service-health-overview.txt"

# Service health issues.
Get-MgAdminServiceAnnouncementIssue -All |
    Sort-Object LastModifiedDateTime -Descending |
    Select-Object Id,Title,Service,Status,Classification,StartDateTime,LastModifiedDateTime |
    Format-Table -AutoSize |
    Tee-Object "$EvidencePath\service-health-issues.txt"

# Message center messages.
Get-MgAdminServiceAnnouncementMessage -All |
    Sort-Object LastModifiedDateTime -Descending |
    Select-Object Id,Title,Category,Severity,StartDateTime,EndDateTime,LastModifiedDateTime |
    Format-Table -AutoSize |
    Tee-Object "$EvidencePath\message-center-messages.txt"

# Preserve raw JSON for deeper review.
Get-MgAdminServiceAnnouncementHealthOverview |
    ConvertTo-Json -Depth 10 |
    Out-File "$EvidencePath\service-health-overview.json"

Get-MgAdminServiceAnnouncementIssue -All |
    ConvertTo-Json -Depth 10 |
    Out-File "$EvidencePath\service-health-issues.json"

Get-MgAdminServiceAnnouncementMessage -All |
    ConvertTo-Json -Depth 10 |
    Out-File "$EvidencePath\message-center-messages.json"
```

Expected result:

| Check | Expected Result |
|---|---|
| Graph context | Correct tenant ID and scopes are visible |
| Health overview export | Services and statuses are exported |
| Issue export | Incidents and advisories are exported |
| Message export | Message center items are exported |
| JSON files | Raw evidence exists for review |

# 03_Monitor_Service_Health_Message_Center_Usage_Reports_And_M365_Backup_Verification_Commands
```powershell
# Service health and Message center verification.

Connect-MgGraph -Scopes "ServiceHealth.Read.All","ServiceMessage.Read.All","Reports.Read.All","Organization.Read.All"

Get-MgContext

Get-MgAdminServiceAnnouncementHealthOverview |
    Format-Table Service,Status,Id -AutoSize

Get-MgAdminServiceAnnouncementIssue -All |
    Select-Object Id,Title,Service,Status,Classification,StartDateTime,LastModifiedDateTime |
    Format-Table -AutoSize

Get-MgAdminServiceAnnouncementMessage -All |
    Select-Object Id,Title,Category,Severity,StartDateTime,EndDateTime,LastModifiedDateTime |
    Format-Table -AutoSize
```

```powershell
# Report cmdlet inventory and export readiness.

Connect-MgGraph -Scopes "Reports.Read.All","Organization.Read.All"

Get-Command -Module Microsoft.Graph.Reports |
    Sort-Object Name |
    Format-Table Name,Source -AutoSize
```

Portal verification:

| Validation Area | Portal Path | Good Result |
|---|---|---|
| Service health opens | Health > Service health | Active issues and service status are visible |
| Service health history opens | Health > Service health > History | Resolved issues are visible |
| Message center opens | Health > Message center | Active messages are visible |
| Major updates visible | Message center > Major updates | High-impact changes are visible |
| Message preferences configured | Message center > Preferences | Digest and service preferences are configured |
| Reports dashboard opens | Reports > Usage | Usage report dashboard loads |
| Active users report opens | Reports > Usage > Active users | Active user trends are visible |
| Report privacy setting known | Settings > Org settings > Reports | Privacy stance is known |
| Backup page opens | Admin center > Backup | Backup availability is known |
| Restore flow reviewed | Backup > Restore | Restore path is understood |
| Evidence saved | `<evidence-path>` | Screenshots, exports, and notes exist |

# 03_Monitor_Service_Health_Message_Center_Usage_Reports_And_M365_Backup_Rollback
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify changed Message center preferences | Admin Workstation | Message center > Preferences | Changed digest or service preferences are visible |
| 2 | Restore previous digest recipients | Admin Workstation | Message center > Preferences | Original recipients are restored |
| 3 | Restore previous service preferences | Admin Workstation | Message center > Preferences | Original workload preferences are restored |
| 4 | Identify changed report privacy setting | Admin Workstation | Settings > Org settings > Reports | Current privacy setting is visible |
| 5 | Restore previous report privacy setting | Admin Workstation | Settings > Org settings > Reports | Previous privacy stance is restored |
| 6 | Identify changed dashboard cards | Admin Workstation | Home dashboard | Added or removed cards are known |
| 7 | Restore dashboard card layout | Admin Workstation | Home > Add card / Remove | Prior dashboard layout is restored |
| 8 | Identify changed service health notification settings | Admin Workstation | Service health notification settings | Notification changes are visible |
| 9 | Restore service health notification recipients | Admin Workstation | Service health notification settings | Original notification behavior is restored |
| 10 | Identify changed backup policy | Admin Workstation | Backup policy page | Changed backup scope is visible |
| 11 | Restore previous backup policy only with approval | Admin Workstation | Backup policy page | Backup scope returns to approved state |
| 12 | Document rollback | Operator | Notes | Rollback owner, time, and validation are recorded |

Rollback note template:

```text
Change made:
Previous value:
New value:
Rollback action:
Rollback owner:
Rollback time:
Validation performed:
Remaining issue:
```

Backup rollback warning:

```text
Do not disable or reduce Microsoft 365 Backup protection without business approval.
Changing backup scope can create recovery gaps.
For backup changes, rollback must be approved by the data owner, compliance owner, or backup owner.
```

# 03_Monitor_Service_Health_Message_Center_Usage_Reports_And_M365_Backup_Failure_Checks
| Symptom | Command | What Usually Broke |
|---|---|---|
| Service health page does not open | Admin center > Health > Service health | Missing admin role, tenant limitation, or portal issue |
| Service health shows no data | Refresh portal / try another admin | Tenant has no current issues or admin lacks visibility |
| Broad user issue but no tenant change found | Check Service health first | Microsoft service incident may be active |
| Issue details unclear | Open incident details | Admin did not review scope, impact, or latest update |
| Message center missing | Admin center > Health > Message center | Missing role or navigation limitation |
| Digest emails not received | Message center > Preferences | Recipient not added, mail filtering, or digest not enabled |
| Major update missed | Message center filters | Preferences too narrow or no review cadence |
| Message center flooded with noise | Message center filters/preferences | Services are not filtered by owned workload |
| Reports page denied | Reports > Usage | Missing Reports Reader, Global Reader, or admin role |
| Reports show anonymized users | Settings > Org settings > Reports | Report privacy hides identifiable names |
| Reports show too little data | Report time filter | Wrong reporting window selected |
| Usage data looks delayed | Reports > Usage | Report data can lag behind current activity |
| Graph service health cmdlets fail | `Get-MgContext` | Missing `ServiceHealth.Read.All` or `ServiceMessage.Read.All` |
| Graph report cmdlets fail | `Get-MgContext` | Missing `Reports.Read.All` or module command not available |
| Graph command not found | `Get-Command -Module Microsoft.Graph*` | Microsoft Graph module missing or outdated |
| `Get-MgAdminServiceAnnouncementIssue` returns nothing | Graph query | No issues match or permissions are insufficient |
| Backup page missing | Admin center search for Backup | Feature unavailable, license/billing not ready, or tenant not enabled |
| Backup setup blocked | Backup setup page | Billing prerequisite, role, or service availability issue |
| Backup scope unclear | Backup policy page | Protection policy not documented |
| Restore test not possible | Backup restore page | No protected data, no restore point, or permission issue |
| Restore test fails | Backup restore workflow | Wrong scope, missing permissions, unsupported object, or service issue |
| Help desk unaware of Microsoft incident | Incident notes / ticket | Service health review is not integrated into support workflow |
| Adoption report ignored | Reports > Usage | No owner assigned to report review |
| Backup assumed but not tested | Restore evidence folder | No recovery validation exists |

# 03_Monitor_Service_Health_Message_Center_Usage_Reports_And_M365_Backup_Related_Labs
| Lab | Relationship |
|---|---|
| 01_Create_And_Configure_Microsoft_365_Tenant_Org_Profile_And_Service_Settings | Tenant baseline and admin center access must exist first |
| 02_Configure_M365_Custom_Domains_DNS_And_Network_Connectivity_Insights | Network connectivity insights depend on domain and network baseline |
| 04_Create_Manage_And_Bulk_Administer_M365_Users_Contacts_And_Guests | Usage reports and service incidents affect user support |
| 05_Manage_M365_Groups_Security_Groups_Distribution_Groups_Shared_Mailboxes_And_M365_Groups | Group and shared mailbox health can appear in service incidents and reports |
| 06_Manage_M365_Licensing_Service_Plans_Group_Based_Licensing_And_Usage | Usage reports support license cleanup and adoption review |
| 07_Manage_Admin_Roles_Role_Groups_Admin_Units_And_Delegation | Health, Message center, reports, and backup require correct admin roles |
| 08_Troubleshoot_M365_Tenant_User_Group_License_Role_And_Service_Health_Issues | Service health and reports feed troubleshooting workflow |
| 07_Configure_Cloud_Audit_Logs_Service_Health_Message_Center_And_Change_Tracking | Cloud foundation monitoring workbook |
| 06_Configure_Backup_Reports_Alerts_And_Recovery_Validation | Azure backup reporting and recovery validation pattern |
| 03_Configure_Azure_Monitor_Metrics_Diagnostic_Settings_Log_Analytics_And_KQL | Azure monitoring pattern for alerting, logs, and evidence review |