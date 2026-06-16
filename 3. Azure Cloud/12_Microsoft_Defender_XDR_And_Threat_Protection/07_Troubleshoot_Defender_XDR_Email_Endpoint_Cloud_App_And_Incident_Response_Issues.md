# 07_Troubleshoot_Defender_XDR_Email_Endpoint_Cloud_App_And_Incident_Response_Issues

## Objective

Troubleshoot common Microsoft Defender XDR issues across email protection, endpoint onboarding and telemetry, Defender for Cloud Apps visibility, Advanced Hunting, incidents, alerts, and response actions.

## Lab Context

This workbook assumes Microsoft Defender XDR is deployed or being deployed across Microsoft Defender for Office 365, Microsoft Defender for Endpoint, Microsoft Defender for Cloud Apps, and incident response workflows. The goal is to isolate whether a problem is caused by licensing, permissions, policy scope, telemetry delay, connector health, device state, mail flow, or response action failure.

## Prerequisites

| Requirement | Details |
|---|---|
| Admin Access | Security Administrator, Security Operator, Security Reader, Exchange Administrator, Cloud App Security Administrator, Endpoint Administrator, or Global Administrator |
| Portal | https://security.microsoft.com |
| Exchange Admin Center | https://admin.exchange.microsoft.com |
| Licensing | Defender XDR workloads licensed as needed |
| Test Mailbox | Exchange Online mailbox for email testing |
| Test Device | Defender for Endpoint onboarded endpoint |
| Test SaaS App | Connected app for Defender for Cloud Apps testing |
| Browser | Edge or Chrome |
| Optional PowerShell | Exchange Online PowerShell and local endpoint PowerShell |
| Change Control | Required before production response actions |

## Configuration Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|---:|---|---|---|---|---|
| 1 | Confirm Defender Portal Access | Admin Workstation | Browse to https://security.microsoft.com | N/A | Defender portal loads |
| 2 | Confirm Tenant Context | Admin Workstation | Check profile menu | N/A | Correct tenant displayed |
| 3 | Confirm Admin Role | Defender Portal | Settings → Microsoft Defender XDR → Permissions | N/A | Correct security role assigned |
| 4 | Confirm Licensing | Microsoft 365 Admin Center | Billing → Licenses | N/A | Required Defender licenses assigned |
| 5 | Confirm Workload Availability | Defender Portal | Review Email, Endpoints, Cloud Apps, Hunting, Incidents | N/A | Expected workload blades visible |
| 6 | Check Service Health | Microsoft 365 Admin Center | Health → Service health | N/A | No active Microsoft service incident affecting workload |
| 7 | Review Message Center If Needed | Microsoft 365 Admin Center | Health → Message center | N/A | No announced feature retirement or portal change causing issue |
| 8 | Confirm Browser Session Health | Admin Workstation | Sign out and sign back in | N/A | Portal reloads cleanly |
| 9 | Test Alternate Browser | Admin Workstation | Open portal in Edge or Chrome private window | N/A | Browser cache or extension issue ruled out |
| 10 | Check Email Protection Policy Access | Defender Portal | Email & Collaboration → Policies & Rules → Threat Policies | N/A | Threat policies visible |
| 11 | Check Safe Links Policy | Defender Portal | Threat Policies → Safe Links | N/A | Safe Links policies visible |
| 12 | Check Safe Attachments Policy | Defender Portal | Threat Policies → Safe Attachments | N/A | Safe Attachments policies visible |
| 13 | Check Anti-Phishing Policy | Defender Portal | Threat Policies → Anti-phishing | N/A | Anti-phishing policies visible |
| 14 | Check Anti-Spam Policy | Defender Portal | Threat Policies → Anti-spam | N/A | Anti-spam policies visible |
| 15 | Validate Email Policy Scope | Defender Portal | Open policy → Users, groups, domains | N/A | Affected user or domain is in scope |
| 16 | Validate Email Policy Priority | Defender Portal | Review policy order | N/A | Intended policy has correct priority |
| 17 | Check Quarantine Queue | Defender Portal | Email & Collaboration → Review → Quarantine | N/A | Quarantine queue opens |
| 18 | Search Quarantine By Recipient | Defender Portal | Filter by recipient | N/A | Quarantined messages found or absence confirmed |
| 19 | Search Explorer By Sender | Defender Portal | Email & Collaboration → Explorer → Sender filter | N/A | Sender activity visible if Plan 2 and data exists |
| 20 | Search Explorer By Recipient | Defender Portal | Explorer → Recipient filter | N/A | Recipient activity visible |
| 21 | Search Explorer By Subject | Defender Portal | Explorer → Subject filter | N/A | Matching email events found |
| 22 | Check Email Delivery Action | Defender Portal | Message details → Delivery action/location | N/A | Delivered, Junked, Quarantined, Blocked, or Removed status identified |
| 23 | Check Email Authentication Results | Defender Portal | Message details → Authentication | N/A | SPF, DKIM, DMARC, and composite authentication reviewed |
| 24 | Check URL Click Events | Defender Portal | Explorer → URL clicks | N/A | Safe Links click data visible if present |
| 25 | Check Submissions | Defender Portal | Actions & Submissions → Submissions | N/A | User and admin submissions visible |
| 26 | Validate User Reported Message Flow | Defender Portal | Submissions → User reported | N/A | User reports arrive if configured |
| 27 | Check Restricted Entities | Defender Portal | Email & Collaboration → Review → Restricted entities | N/A | Restricted users or connectors visible |
| 28 | Investigate Restricted User | Defender Portal | Select restricted user | N/A | Restriction reason reviewed |
| 29 | Check Endpoint Settings | Defender Portal | Settings → Endpoints | N/A | Endpoint settings page opens |
| 30 | Check Endpoint Onboarding Page | Defender Portal | Settings → Endpoints → Device management → Onboarding | N/A | Onboarding package available |
| 31 | Check Device Inventory | Defender Portal | Assets → Devices | N/A | Device inventory loads |
| 32 | Search Affected Device | Defender Portal | Search hostname | N/A | Device found or absence confirmed |
| 33 | Open Device Page | Defender Portal | Select device | N/A | Device details open |
| 34 | Check Device Health State | Defender Portal | Device overview | N/A | Sensor health, risk, and exposure visible |
| 35 | Check Device Timeline | Defender Portal | Device page → Timeline | N/A | Recent telemetry visible |
| 36 | Check Local Defender Services | Affected Endpoint | Services console | `Get-Service Sense, WinDefend` | Services present and running where applicable |
| 37 | Check Defender AV Status | Affected Endpoint | Windows Security | `Get-MpComputerStatus` | Defender status visible |
| 38 | Check Defender Preferences | Affected Endpoint | Windows Security | `Get-MpPreference` | Policy preferences visible |
| 39 | Check Network Connectivity | Affected Endpoint | Browser or PowerShell | `Test-NetConnection security.microsoft.com -Port 443` | HTTPS connectivity works |
| 40 | Run Defender Signature Update | Affected Endpoint | Windows Security | `Update-MpSignature` | Signatures update or error captured |
| 41 | Run Quick Scan | Affected Endpoint | Windows Security | `Start-MpScan -ScanType QuickScan` | Scan starts |
| 42 | Check Endpoint Alerts | Defender Portal | Incidents & Alerts → Alerts → filter device | N/A | Device alerts visible if generated |
| 43 | Check Advanced Hunting Endpoint Tables | Defender Portal | Hunting → Advanced Hunting | N/A | Device tables return telemetry |
| 44 | Check Vulnerability Management | Defender Portal | Endpoints or Exposure Management → Recommendations | N/A | Device recommendations visible if available |
| 45 | Check Cloud Apps Settings | Defender Portal | Settings → Cloud Apps | N/A | Cloud Apps settings open |
| 46 | Check App Connectors | Defender Portal | Settings → Cloud Apps → Connected apps → App connectors | N/A | Connector status visible |
| 47 | Review Failed Connector | Defender Portal | Select connector | N/A | Error or permission issue identified |
| 48 | Check Cloud Apps Activity Log | Defender Portal | Cloud Apps → Activity log | N/A | Activity log opens |
| 49 | Filter Activity Log By User | Defender Portal | User filter | N/A | User activity visible if connector has data |
| 50 | Filter Activity Log By App | Defender Portal | App filter | N/A | App activity visible |
| 51 | Check OAuth Apps | Defender Portal | Cloud Apps → OAuth apps | N/A | OAuth app inventory visible |
| 52 | Check Cloud Discovery | Defender Portal | Cloud Apps → Cloud Discovery | N/A | Discovery dashboard opens |
| 53 | Validate Discovery Data Source | Defender Portal | Settings → Cloud Apps → Cloud Discovery → Automatic log upload | N/A | Data source or collector configured |
| 54 | Review Discovery Reports | Defender Portal | Cloud Discovery → Continuous reports or Snapshot reports | N/A | Reports visible if logs processed |
| 55 | Check Cloud App Policies | Defender Portal | Cloud Apps → Policies → Policy management | N/A | Cloud app policies visible |
| 56 | Check Policy Match Conditions | Defender Portal | Open affected policy | N/A | Filters and scope match expected event |
| 57 | Check Incident Queue | Defender Portal | Incidents & Alerts → Incidents | N/A | Incident queue loads |
| 58 | Clear Incident Filters | Defender Portal | Clear all filters | N/A | Hidden incidents ruled out |
| 59 | Search Incident By User Or Device | Defender Portal | Search entity name | N/A | Related incidents found if present |
| 60 | Open Incident Details | Defender Portal | Select incident | N/A | Incident details open |
| 61 | Review Incident Evidence | Defender Portal | Evidence and Response tab | N/A | Evidence visible |
| 62 | Review Incident Alerts | Defender Portal | Alerts tab | N/A | Alerts visible |
| 63 | Check Alert Queue | Defender Portal | Incidents & Alerts → Alerts | N/A | Alert queue loads |
| 64 | Clear Alert Filters | Defender Portal | Clear all filters | N/A | Hidden alerts ruled out |
| 65 | Search Alert By Title Or Entity | Defender Portal | Search alert | N/A | Alert located if present |
| 66 | Check Action Center | Defender Portal | Actions & Submissions → Action Center | N/A | Action Center opens |
| 67 | Review Pending Actions | Defender Portal | Action Center → Pending | N/A | Pending actions visible |
| 68 | Review Failed Actions | Defender Portal | Action Center → Failed or filters | N/A | Failed response actions identified |
| 69 | Review Completed Actions | Defender Portal | Action Center → History | N/A | Completed actions visible |
| 70 | Check Device Online State Before Response | Defender Portal | Device page → Last seen | N/A | Device online or stale status known |
| 71 | Check Response Action Permissions | Defender Portal | Settings → Permissions | N/A | Admin has action permission |
| 72 | Retry Test Response Action Only If Safe | Defender Portal | Use test device or test message | N/A | Action completes or produces useful error |
| 73 | Capture Screenshots Or Export Logs | Admin Workstation | Export portal results where available | N/A | Evidence captured |
| 74 | Document Root Cause | Admin Workstation | Record cause, affected scope, fix, validation, and rollback | N/A | Troubleshooting record completed |

## Email Troubleshooting Checks

| Issue | Check | Expected Result |
|---|---|---|
| Safe Links not applying | Policy scope, priority, workload, user/domain inclusion | Affected user is in correct policy |
| Safe Attachments not applying | Policy scope, action, recipient inclusion | Attachment action matches expected behavior |
| Anti-phishing misses | Protected users, protected domains, mailbox intelligence, spoof intelligence | Impersonation controls enabled and scoped |
| Message not in quarantine | Explorer delivery action, transport rules, allow list, policy action | Actual delivery action identified |
| User cannot release quarantine | Quarantine policy permissions | User release behavior matches quarantine policy |
| User reports not visible | Report Message add-in or user reported settings | Reports appear under Submissions |
| Explorer empty | Licensing, time range, filters, mail flow | Email telemetry visible |
| URL clicks missing | Safe Links policy, click activity, time range | URL click events visible when clicks occur |
| Restricted user cannot send | Restricted entities status and compromise remediation | Restriction reason identified before unblock |

## Endpoint Troubleshooting Checks

| Issue | Check | Expected Result |
|---|---|---|
| Device not onboarded | Onboarding package, Sense service, OS support | Device appears in inventory |
| Device not reporting | Sensor health, last seen, connectivity, proxy | Device timeline updates |
| Defender AV disabled | AV provider, GPO, Intune policy, tamper protection | Defender status understood |
| No endpoint alerts | Test detection, telemetry delay, alert filters | Alert visible or cause documented |
| Live Response unavailable | Feature toggle, role, OS support, device online | Live Response starts on supported device |
| Isolation fails | Device offline or unsupported | Action failure reason documented |
| Vulnerability data missing | Telemetry delay, unsupported platform, licensing | Recommendations appear when supported |
| Advanced Hunting empty | Device telemetry missing | DeviceInfo or DeviceEvents return rows |

## Cloud Apps Troubleshooting Checks

| Issue | Check | Expected Result |
|---|---|---|
| Cloud Apps missing | License or role | Cloud Apps blade visible |
| App connector failed | SaaS admin consent, permissions, token health | Connector connected |
| Activity log empty | Connector status, app activity, processing delay | Activities appear |
| OAuth app data missing | Microsoft 365 connector and app governance support | OAuth apps visible |
| Discovery empty | Log source, Defender for Endpoint integration, collector, log parser | Discovered apps appear |
| Discovery stale | Latest log upload and report timestamp | Current logs processed |
| Policy not alerting | Policy enabled, filters, severity, threshold | Matching events trigger alerts |
| Governance action failed | Connector permission or unsupported action | Action failure reason documented |

## Incident And Response Troubleshooting Checks

| Issue | Check | Expected Result |
|---|---|---|
| Incident missing | Filters, time range, alert grouping, suppression | Incident found or no incident created |
| Alert missing | Filters, detection source, telemetry delay | Alert found or cause documented |
| Evidence missing | Alert source, entity type, telemetry coverage | Evidence limitations understood |
| Cannot assign incident | Role missing | Assignment works with correct role |
| Cannot classify alert | Role missing or portal issue | Classification available |
| Action stuck pending | Device offline, backend delay, unsupported action | Action eventually completes or fails with reason |
| Action failed | Permissions, entity state, device state | Failure reason documented |
| Email remediation failed | Message already remediated or inaccessible | Message state verified |
| Indicator not working | Indicator feature disabled, wrong indicator format, policy conflict | Indicator applied and validated |
| Live Response command fails | Session permissions or command support | Supported command succeeds |

## Optional PowerShell Checks

### Connect To Exchange Online

```powershell
Install-Module ExchangeOnlineManagement -Scope CurrentUser
Connect-ExchangeOnline
```

### Review Safe Links Policies

```powershell
Get-SafeLinksPolicy
Get-SafeLinksRule
```

### Review Safe Attachments Policies

```powershell
Get-SafeAttachmentPolicy
Get-SafeAttachmentRule
```

### Review Anti-Phishing Policies

```powershell
Get-AntiPhishPolicy
Get-AntiPhishRule
```

### Review Anti-Spam Policies

```powershell
Get-HostedContentFilterPolicy
Get-HostedContentFilterRule
```

### Review Anti-Malware Policies

```powershell
Get-MalwareFilterPolicy
Get-MalwareFilterRule
```

### Review Quarantine Policies

```powershell
Get-QuarantinePolicy
```

### Review Tenant Allow Block List

```powershell
Get-TenantAllowBlockListItems
```

### Disconnect Exchange Online

```powershell
Disconnect-ExchangeOnline -Confirm:$false
```

## Local Endpoint PowerShell Checks

### Check Defender Services

```powershell
Get-Service Sense, WinDefend
```

### Check Defender Status

```powershell
Get-MpComputerStatus
```

### Check Defender Preferences

```powershell
Get-MpPreference
```

### Check Real-Time Protection

```powershell
(Get-MpComputerStatus).RealTimeProtectionEnabled
```

### Check Cloud Protection

```powershell
(Get-MpPreference).MAPSReporting
```

### Update Signatures

```powershell
Update-MpSignature
```

### Run Quick Scan

```powershell
Start-MpScan -ScanType QuickScan
```

### Check Connectivity To Defender Portal

```powershell
Test-NetConnection security.microsoft.com -Port 443
```

### Check Device Identity

```powershell
hostname
Get-ComputerInfo | Select-Object CsName, WindowsProductName, WindowsVersion, OsBuildNumber
```

## Advanced Hunting Troubleshooting Queries

### Confirm Recent Alerts

```kql
AlertInfo
| where Timestamp > ago(7d)
| project Timestamp, AlertId, Title, Severity, ServiceSource, DetectionSource
| order by Timestamp desc
```

### Confirm Recent Incidents

```kql
SecurityIncident
| where CreatedTime > ago(30d)
| project CreatedTime, IncidentName, Severity, Status, Classification, AssignedTo
| order by CreatedTime desc
```

### Confirm Endpoint Telemetry

```kql
DeviceInfo
| where Timestamp > ago(7d)
| summarize LastSeen=max(Timestamp) by DeviceName, DeviceId, OSPlatform, OnboardingStatus, SensorHealthState
| order by LastSeen desc
```

### Confirm Device Events

```kql
DeviceEvents
| where Timestamp > ago(24h)
| project Timestamp, DeviceName, ActionType, InitiatingProcessFileName, InitiatingProcessAccountName
| order by Timestamp desc
```

### Confirm Email Telemetry

```kql
EmailEvents
| where Timestamp > ago(7d)
| project Timestamp, SenderFromAddress, RecipientEmailAddress, Subject, ThreatTypes, DeliveryAction, DeliveryLocation, NetworkMessageId
| order by Timestamp desc
```

### Confirm URL Click Telemetry

```kql
UrlClickEvents
| where Timestamp > ago(7d)
| project Timestamp, AccountUpn, Url, ActionType, Workload, NetworkMessageId
| order by Timestamp desc
```

### Confirm Cloud App Telemetry

```kql
CloudAppEvents
| where Timestamp > ago(7d)
| project Timestamp, AccountDisplayName, Application, ActionType, IPAddress, CountryCode
| order by Timestamp desc
```

### Find Alerts For A Device

```kql
let TargetDevice = "DEVICE-NAME";
AlertEvidence
| where Timestamp > ago(30d)
| where DeviceName =~ TargetDevice
| join kind=leftouter AlertInfo on AlertId
| project Timestamp, Title, Severity, ServiceSource, DeviceName, EntityType
| order by Timestamp desc
```

### Find Email For A Recipient

```kql
let TargetRecipient = "user@example.com";
EmailEvents
| where Timestamp > ago(30d)
| where RecipientEmailAddress =~ TargetRecipient
| project Timestamp, SenderFromAddress, RecipientEmailAddress, Subject, ThreatTypes, DeliveryAction, DeliveryLocation, NetworkMessageId
| order by Timestamp desc
```

### Find Cloud App Activity For A User

```kql
let TargetUser = "user@example.com";
CloudAppEvents
| where Timestamp > ago(30d)
| where AccountId =~ TargetUser or AccountDisplayName has TargetUser
| project Timestamp, AccountDisplayName, Application, ActionType, IPAddress, CountryCode
| order by Timestamp desc
```

## Validation

| Check | Expected Result |
|---|---|
| Defender Portal Loads | Success |
| Correct Tenant Confirmed | Success |
| Admin Role Confirmed | Success |
| Licensing Confirmed | Success |
| Service Health Checked | Success |
| Email Threat Policies Accessible | Success |
| Quarantine Accessible | Success |
| Explorer Search Works If Licensed | Success |
| Submissions Accessible | Success |
| Restricted Entities Accessible | Success |
| Endpoint Settings Accessible | Success |
| Device Inventory Accessible | Success |
| Affected Device Status Known | Success |
| Local Defender Services Checked | Success |
| Cloud Apps Settings Accessible | Success |
| App Connector Status Reviewed | Success |
| Activity Log Reviewed | Success |
| Cloud Discovery Reviewed | Success |
| Incident Queue Reviewed | Success |
| Alert Queue Reviewed | Success |
| Action Center Reviewed | Success |
| Advanced Hunting Queries Run | Success |
| Root Cause Documented | Success |

## Troubleshooting Matrix

| Symptom | Likely Cause | Fix |
|---|---|---|
| Defender portal access denied | Missing role assignment | Assign appropriate security role |
| Workload blade missing | Missing license or permission | Verify assigned license and role |
| Data missing everywhere | Service issue or telemetry not onboarded | Check service health and workload onboarding |
| Email data missing | Defender for Office 365 licensing or no mail flow | Verify licensing, mail flow, Explorer availability |
| Endpoint data missing | Device not onboarded or not reporting | Check Sense service, connectivity, and device inventory |
| Cloud app data missing | Connector or discovery source not configured | Check app connectors and discovery data sources |
| Hunting table missing | Workload data source unavailable | Onboard workload and wait for telemetry |
| Incident missing | No alert correlation or filters hide it | Clear filters and check alert queue |
| Alert missing | Detection not generated or filters hide it | Clear filters and check telemetry source |
| Response action failed | Device offline, permission issue, or unsupported action | Review Action Center failure details |
| Email remediation failed | Message moved, deleted, or out of scope | Confirm message state in Explorer |
| User remains restricted | Compromise not remediated or unblock not performed | Review Restricted Entities and complete remediation |
| Safe Links not rewriting | Policy scope or priority wrong | Fix policy inclusion and order |
| Safe Attachments not scanning | Policy scope or attachment action wrong | Fix policy and verify supported attachment flow |
| Cloud discovery stale | Log upload stopped | Restart collector or upload current logs |
| Live Response unavailable | Feature disabled or device unsupported | Enable feature and verify support |
| Vulnerability data stale | Telemetry delay or unsupported device | Wait and verify device health |

## Rollback

| Step | Action | Expected Result |
|---:|---|---|
| 1 | Remove Temporary Test Indicators | Custom blocks or allows removed |
| 2 | Release Test Device From Isolation | Network restored |
| 3 | Remove Test App Restrictions | Application execution restored |
| 4 | Close Live Response Session | Interactive session ended |
| 5 | Disable Test Cloud App Policies | Test alerts stop |
| 6 | Remove Test Sanctioned Or Unsanctioned Tags | Discovery labels restored |
| 7 | Restore Email From Test Remediation If Possible | Test message restored if recoverable |
| 8 | Revert Test Quarantine Changes | Quarantine state corrected |
| 9 | Revert Temporary Role Assignments | Least privilege restored |
| 10 | Archive Troubleshooting Notes | Evidence preserved |

## Documentation Checklist

| Item | Status |
|---|---|
| Tenant Verified | ☐ |
| Admin Account Recorded | ☐ |
| Role Assignment Verified | ☐ |
| Licensing Verified | ☐ |
| Service Health Checked | ☐ |
| Affected Workload Identified | ☐ |
| Affected User Recorded | ☐ |
| Affected Device Recorded | ☐ |
| Affected App Recorded | ☐ |
| Affected Email Recorded | ☐ |
| Incident ID Recorded | ☐ |
| Alert ID Recorded | ☐ |
| Message ID Recorded | ☐ |
| Network Message ID Recorded | ☐ |
| Device Sensor Health Recorded | ☐ |
| Email Delivery Action Recorded | ☐ |
| Cloud App Connector Status Recorded | ☐ |
| Discovery Source Status Recorded | ☐ |
| Action Center Result Recorded | ☐ |
| Hunting Query Used Recorded | ☐ |
| Root Cause Recorded | ☐ |
| Fix Applied | ☐ |
| Validation Completed | ☐ |
| Rollback Completed If Needed | ☐ |

## Notes

- Start with licensing, role, service health, and filters before assuming the product is broken.
- Defender portal data often has processing delay, especially after onboarding devices, connecting apps, or uploading discovery logs.
- Advanced Hunting is the fastest way to confirm whether telemetry exists.
- Action Center is the source of truth for response action status.
- Do not unblock restricted users until compromise checks are complete.
- Do not release quarantined email unless the message is confirmed safe.
- Device isolation, file quarantine, and custom indicators can disrupt production. Use test assets first.
- Always document the difference between no data, no detection, and no permission.