# 05_Configure_Defender_For_Cloud_Apps_Connectors_Policies_Activity_Log_And_Discovery

## Objective

Configure Microsoft Defender for Cloud Apps baseline administration by validating portal access, connecting supported apps, reviewing policy types, using the activity log, configuring discovery, and documenting cloud app visibility.

## Lab Context

This workbook assumes a Microsoft 365 tenant with Microsoft Defender for Cloud Apps available through Microsoft Defender XDR. The goal is to establish a practical baseline for SaaS app visibility, app connector status, activity monitoring, app discovery, and policy-based alerting.

## Prerequisites

| Requirement | Details |
|---|---|
| Admin Access | Security Administrator, Cloud App Security Administrator, Global Administrator, Security Operator, or Security Reader depending on task |
| Portal | https://security.microsoft.com |
| Legacy Portal Reference | https://portal.cloudappsecurity.com if still redirected or available |
| Licensing | Microsoft Defender for Cloud Apps |
| Test SaaS App | Microsoft 365, Azure, GitHub, Salesforce, Box, Dropbox, ServiceNow, or other supported connector |
| Test User | At least one test user with app activity |
| Network Log Source | Firewall, proxy, Defender for Endpoint, or supported log collector for discovery |
| Browser | Edge or Chrome |
| Optional PowerShell | Microsoft Graph PowerShell or Exchange Online PowerShell depending on connected workload |

## Configuration Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|---:|---|---|---|---|---|
| 1 | Open Microsoft Defender Portal | Admin Workstation | Browse to https://security.microsoft.com | N/A | Defender portal loads |
| 2 | Verify Tenant Context | Admin Workstation | Confirm tenant from profile menu | N/A | Correct tenant displayed |
| 3 | Open Cloud Apps Settings | Defender Portal | Settings → Cloud Apps | N/A | Cloud Apps settings page opens |
| 4 | Review Cloud Apps Dashboard | Defender Portal | Cloud Apps → Overview | N/A | Defender for Cloud Apps dashboard visible |
| 5 | Review Connected Apps | Defender Portal | Settings → Cloud Apps → Connected apps → App connectors | N/A | Existing app connectors visible |
| 6 | Review Connector Status | Defender Portal | App connectors list | N/A | Connector health and status visible |
| 7 | Start New App Connector | Defender Portal | App connectors → Connect an app | N/A | Connector wizard opens |
| 8 | Select Supported App | Defender Portal | Choose Microsoft 365, Azure, GitHub, Salesforce, Box, Dropbox, or other supported app | N/A | App connector type selected |
| 9 | Review Connector Requirements | Defender Portal | Read permissions and prerequisites | N/A | Required permissions understood |
| 10 | Authorize App Connector | SaaS Admin Portal | Sign in and consent required permissions | N/A | Connector authorization completes |
| 11 | Confirm Connector Status | Defender Portal | Settings → Cloud Apps → App connectors | N/A | Connector status shows connected or pending |
| 12 | Validate Connector Data Flow | Defender Portal | Cloud Apps → Activity log | N/A | Activities from connected app appear after processing |
| 13 | Review App Governance If Available | Defender Portal | Cloud Apps → App governance | N/A | OAuth app governance data visible if licensed and enabled |
| 14 | Review OAuth Apps | Defender Portal | Cloud Apps → OAuth apps | N/A | OAuth applications visible |
| 15 | Filter OAuth Apps By Risk | Defender Portal | OAuth apps → Risk level filter | N/A | High risk OAuth apps identified |
| 16 | Review OAuth App Details | Defender Portal | Select OAuth app | N/A | Publisher, permissions, users, and activity visible |
| 17 | Document OAuth App Risk | Admin Workstation | Record app name, permissions, consented users, and risk | N/A | OAuth risk notes documented |
| 18 | Open Policy Management | Defender Portal | Cloud Apps → Policies → Policy management | N/A | Policy management page opens |
| 19 | Review Built-In Policy Templates | Defender Portal | Policy templates | N/A | Built-in templates visible |
| 20 | Review Activity Policies | Defender Portal | Policy management → Activity policies | N/A | Activity policies visible |
| 21 | Create Activity Policy | Defender Portal | Create policy → Activity policy | N/A | Activity policy wizard opens |
| 22 | Name Activity Policy | Defender Portal | Enter `Baseline Suspicious Cloud Activity Policy` | N/A | Policy named |
| 23 | Configure Activity Policy Filters | Defender Portal | Filter by app, user, activity type, IP address, location, or device tag | N/A | Scope defined |
| 24 | Configure Activity Policy Alerting | Defender Portal | Enable alert and set severity | N/A | Alerts generated when policy matches |
| 25 | Configure Activity Policy Governance Action | Defender Portal | Select allowed governance action if approved | N/A | Governance action configured or left disabled |
| 26 | Save Activity Policy | Defender Portal | Create | N/A | Activity policy created |
| 27 | Review File Policies | Defender Portal | Policy management → File policies | N/A | File policies visible |
| 28 | Create File Policy If Needed | Defender Portal | Create policy → File policy | N/A | File policy wizard opens |
| 29 | Configure File Policy Filters | Defender Portal | Filter by app, owner, sharing level, file type, sensitivity label, or inspection result | N/A | File policy scope configured |
| 30 | Configure File Policy Actions | Defender Portal | Alert, remove external users, make private, or apply governance action if supported | N/A | File governance configured if approved |
| 31 | Save File Policy | Defender Portal | Create | N/A | File policy created |
| 32 | Review Anomaly Detection Policies | Defender Portal | Policy management → Anomaly detection policies | N/A | Anomaly policies visible |
| 33 | Review Impossible Travel Policy | Defender Portal | Open impossible travel policy | N/A | Policy status and sensitivity visible |
| 34 | Review Mass Download Policy | Defender Portal | Open mass download policy | N/A | Policy status and threshold visible |
| 35 | Review Suspicious Inbox Rule Or App Activity Policies | Defender Portal | Review relevant anomaly policies | N/A | Anomaly monitoring understood |
| 36 | Enable Or Tune Anomaly Policies | Defender Portal | Adjust policy state and sensitivity if approved | N/A | Anomaly detection baseline configured |
| 37 | Review Session Policies | Defender Portal | Policy management → Session policies | N/A | Session policies visible if Conditional Access App Control is configured |
| 38 | Review Access Policies | Defender Portal | Policy management → Access policies | N/A | Access policies visible if Conditional Access App Control is configured |
| 39 | Confirm Conditional Access App Control Status | Defender Portal | Settings → Cloud Apps → Conditional Access App Control | N/A | Session control readiness visible |
| 40 | Open Activity Log | Defender Portal | Cloud Apps → Activity log | N/A | Activity log opens |
| 41 | Filter Activity Log By App | Defender Portal | App filter | N/A | Activities from selected app displayed |
| 42 | Filter Activity Log By User | Defender Portal | User filter | N/A | User activity displayed |
| 43 | Filter Activity Log By Activity Type | Defender Portal | Activity type filter | N/A | Specific action types displayed |
| 44 | Filter Activity Log By IP Address | Defender Portal | IP address filter | N/A | Activities from IP displayed |
| 45 | Filter Activity Log By Location | Defender Portal | Location filter | N/A | Geo-based activity visible |
| 46 | Open Activity Details | Defender Portal | Select activity | N/A | Detailed activity record opens |
| 47 | Review Activity Raw Data | Defender Portal | Activity details → Raw data if available | N/A | Raw event data visible |
| 48 | Investigate Related User | Defender Portal | Open user entity from activity | N/A | User activity and risk context visible |
| 49 | Investigate Related IP | Defender Portal | Open IP entity from activity | N/A | IP context visible |
| 50 | Export Activity Results | Defender Portal | Export activity list | N/A | CSV export generated if permitted |
| 51 | Open Cloud Discovery Dashboard | Defender Portal | Cloud Apps → Cloud Discovery | N/A | Discovery dashboard opens |
| 52 | Review Discovered Apps | Defender Portal | Cloud Discovery → Discovered apps | N/A | Discovered SaaS apps listed |
| 53 | Review App Risk Score | Defender Portal | Select discovered app | N/A | Risk score and app details visible |
| 54 | Filter Discovered Apps By Risk | Defender Portal | Risk score filter | N/A | Risky apps identified |
| 55 | Filter Discovered Apps By Category | Defender Portal | Category filter | N/A | App category visibility available |
| 56 | Filter Discovered Apps By Traffic | Defender Portal | Sort by traffic, users, uploads, downloads, transactions | N/A | High usage apps identified |
| 57 | Tag Sanctioned Apps | Defender Portal | Select app → Mark as sanctioned | N/A | Approved apps marked sanctioned |
| 58 | Tag Unsanctioned Apps | Defender Portal | Select app → Mark as unsanctioned | N/A | Risky apps marked unsanctioned |
| 59 | Review Discovered Users | Defender Portal | Cloud Discovery → Discovered users | N/A | Users associated with cloud app usage visible |
| 60 | Review Discovered IP Addresses | Defender Portal | Cloud Discovery → IP addresses | N/A | Source IPs visible |
| 61 | Configure Discovery Data Source | Defender Portal | Settings → Cloud Apps → Cloud Discovery → Automatic log upload | N/A | Discovery upload configuration visible |
| 62 | Add Data Source | Defender Portal | Add data source | N/A | Firewall or proxy source selected |
| 63 | Select Log Format | Defender Portal | Choose supported appliance or custom log parser | N/A | Log format selected |
| 64 | Configure Log Collector If Used | Log Collector Host | Install and configure Docker-based log collector if required | N/A | Log collector configured |
| 65 | Upload Sample Log If Using Manual Upload | Defender Portal | Cloud Discovery → Snapshot report → Upload logs | N/A | Snapshot discovery report created |
| 66 | Validate Discovery Processing | Defender Portal | Cloud Discovery dashboard | N/A | Uploaded or collected logs processed |
| 67 | Review Continuous Reports | Defender Portal | Cloud Discovery → Continuous reports | N/A | Continuous discovery report visible if configured |
| 68 | Review Snapshot Reports | Defender Portal | Cloud Discovery → Snapshot reports | N/A | Snapshot reports visible if used |
| 69 | Configure Discovery Policy | Defender Portal | Cloud Apps → Policies → Create policy → App discovery policy | N/A | Discovery policy wizard opens |
| 70 | Define Discovery Policy Filters | Defender Portal | Filter by risk score, category, users, traffic, or unsanctioned status | N/A | Discovery policy scope configured |
| 71 | Configure Discovery Policy Alert | Defender Portal | Enable alert and severity | N/A | Alerts generated for risky discovered apps |
| 72 | Save Discovery Policy | Defender Portal | Create | N/A | App discovery policy created |
| 73 | Review Alerts | Defender Portal | Incidents & Alerts → Alerts | N/A | Cloud app alerts visible if triggered |
| 74 | Open Cloud App Alert | Defender Portal | Select alert from Defender for Cloud Apps source | N/A | Alert details visible |
| 75 | Review Alert Evidence | Defender Portal | Alert evidence and activity details | N/A | Alert context visible |
| 76 | Classify Test Alert If Authorized | Defender Portal | Classify alert as true positive, false positive, benign positive, or undetermined | N/A | Alert classification recorded |
| 77 | Review Governance Log | Defender Portal | Cloud Apps → Governance log | N/A | Governance actions visible |
| 78 | Document Connector And Discovery Baseline | Admin Workstation | Record connectors, policies, discovery sources, sanctioned apps, unsanctioned apps, and known gaps | N/A | Baseline documented |

## Policy Baseline Examples

| Policy Type | Example Use | Recommended Initial Action |
|---|---|---|
| Activity Policy | Alert on admin activity from risky country | Alert only |
| Activity Policy | Alert on mass failed login attempts | Alert only |
| File Policy | Alert on externally shared sensitive files | Alert first, govern after testing |
| Anomaly Detection Policy | Impossible travel or mass download | Enable and monitor |
| Session Policy | Block download from unmanaged device | Pilot only |
| Access Policy | Block access from risky location | Pilot only |
| App Discovery Policy | Alert on high traffic unsanctioned app | Alert only |
| OAuth App Policy | Alert on high privilege OAuth app consent | Alert only |

## Optional Advanced Hunting Queries

### Recent Cloud App Events

```kql
CloudAppEvents
| where Timestamp > ago(7d)
| project Timestamp, AccountDisplayName, AccountId, Application, ActionType, IPAddress, CountryCode
| order by Timestamp desc
```

### Activities From A Specific User

```kql
CloudAppEvents
| where Timestamp > ago(30d)
| where AccountDisplayName has "user"
| project Timestamp, AccountDisplayName, Application, ActionType, IPAddress, CountryCode
| order by Timestamp desc
```

### Activities From A Specific App

```kql
CloudAppEvents
| where Timestamp > ago(30d)
| where Application has "AppName"
| project Timestamp, AccountDisplayName, Application, ActionType, IPAddress, CountryCode
| order by Timestamp desc
```

### Cloud Activities From Non-US Locations

```kql
CloudAppEvents
| where Timestamp > ago(30d)
| where CountryCode != "US"
| project Timestamp, AccountDisplayName, Application, ActionType, IPAddress, CountryCode
| order by Timestamp desc
```

### Potential Risky File Activities

```kql
CloudAppEvents
| where Timestamp > ago(30d)
| where ActionType has_any ("Download", "Share", "FileDownloaded", "FileShared")
| project Timestamp, AccountDisplayName, Application, ActionType, ObjectName, IPAddress, CountryCode
| order by Timestamp desc
```

### OAuth App Related Events

```kql
CloudAppEvents
| where Timestamp > ago(30d)
| where ActionType has_any ("Consent", "Add service principal", "Add app role assignment", "Update application")
| project Timestamp, AccountDisplayName, Application, ActionType, ObjectName, IPAddress
| order by Timestamp desc
```

## Validation

| Check | Expected Result |
|---|---|
| Defender Portal Opens | Success |
| Cloud Apps Settings Accessible | Success |
| App Connectors Page Opens | Success |
| At Least One Connector Reviewed Or Configured | Success |
| Activity Log Opens | Success |
| Activity Filters Work | Success |
| Activity Details Visible | Success |
| Policy Management Opens | Success |
| Activity Policy Created Or Reviewed | Success |
| File Policy Created Or Reviewed | Success |
| Anomaly Policies Reviewed | Success |
| OAuth Apps Reviewed | Success |
| Cloud Discovery Dashboard Opens | Success |
| Discovered Apps Visible If Data Exists | Success |
| Sanctioned And Unsanctioned Tags Reviewed | Success |
| Discovery Policy Created Or Reviewed | Success |
| Alerts Visible If Policies Trigger | Success |
| Baseline Documented | Success |

## Troubleshooting

| Symptom | Likely Cause | Resolution |
|---|---|---|
| Cloud Apps Menu Missing | Missing license or role | Verify Defender for Cloud Apps licensing and permissions |
| App Connector Fails | Insufficient SaaS permissions or consent blocked | Use correct SaaS admin account and verify consent settings |
| Connector Shows Pending | Data sync delay | Wait and recheck connector health |
| No Activity Log Data | Connector not active or no app activity | Verify connector status and generate test activity |
| OAuth Apps Empty | No supported OAuth app data or connector missing | Connect Microsoft 365 or supported app |
| File Policies Not Triggering | File scanning not available for app or scope mismatch | Validate connector support and policy filters |
| Session Policies Missing | Conditional Access App Control not configured | Configure Entra Conditional Access session control |
| Access Policies Missing | Conditional Access App Control not configured | Configure app onboarding for session control |
| Discovery Dashboard Empty | No discovery data source configured | Configure automatic log upload or manual snapshot report |
| Log Upload Fails | Unsupported format or parsing issue | Confirm log source type and parser format |
| Docker Log Collector Fails | Host, firewall, proxy, or certificate issue | Validate collector network path and container status |
| Discovered Apps Not Updating | Processing delay or stale logs | Confirm current logs are uploaded |
| Alerts Not Created | Policy disabled or filters too narrow | Enable policy and broaden filters |
| Governance Action Failed | App connector lacks required permission | Review connector permissions and supported actions |
| Advanced Hunting CloudAppEvents Empty | No Defender for Cloud Apps telemetry in hunting | Validate connector, activity, and licensing |

## Rollback

| Step | Action | Expected Result |
|---:|---|---|
| 1 | Disable Test Activity Policy | Test alerts stop |
| 2 | Disable Test File Policy | File governance no longer applies |
| 3 | Disable Test Discovery Policy | Discovery alerts stop |
| 4 | Remove Test Sanctioned Or Unsanctioned Tags | App governance labels restored |
| 5 | Remove Test App Connector If Needed | Connector removed |
| 6 | Stop Test Log Collector If Used | Discovery upload stops |
| 7 | Remove Uploaded Snapshot Report If Needed | Test report removed if portal allows |
| 8 | Close Or Reclassify Test Alerts | Alert queue restored |
| 9 | Archive Baseline Notes | Configuration record preserved |

## Documentation Checklist

| Item | Status |
|---|---|
| Tenant Verified | ☐ |
| Defender for Cloud Apps License Verified | ☐ |
| Admin Role Verified | ☐ |
| Cloud Apps Settings Reviewed | ☐ |
| App Connectors Reviewed | ☐ |
| Connector Status Recorded | ☐ |
| OAuth Apps Reviewed | ☐ |
| High Risk OAuth Apps Recorded | ☐ |
| Activity Log Reviewed | ☐ |
| Activity Filters Tested | ☐ |
| Activity Export Tested | ☐ |
| Activity Policies Reviewed | ☐ |
| File Policies Reviewed | ☐ |
| Anomaly Policies Reviewed | ☐ |
| Session Policies Reviewed | ☐ |
| Access Policies Reviewed | ☐ |
| Cloud Discovery Dashboard Reviewed | ☐ |
| Discovery Data Source Recorded | ☐ |
| Discovered Apps Reviewed | ☐ |
| Sanctioned Apps Recorded | ☐ |
| Unsanctioned Apps Recorded | ☐ |
| Discovery Policies Reviewed | ☐ |
| Cloud App Alerts Reviewed | ☐ |
| Governance Log Reviewed | ☐ |
| Rollback Plan Documented | ☐ |

## Notes

- Start with alert-only policies before enabling governance actions.
- App connectors require permissions in the target SaaS application, not only in Microsoft Defender.
- Activity log value depends on connected apps and actual user activity.
- Cloud Discovery depends on network traffic logs or Defender for Endpoint integration.
- Sanctioned and unsanctioned app tags are useful only when the organization agrees on what should be approved or blocked.
- Conditional Access App Control requires Entra Conditional Access integration and should be piloted carefully.
- Governance actions can affect user access and files, so test with pilot users first.
- Document connector ownership because SaaS app permissions and tokens can break later if admin accounts change.