# 06_Use_Advanced_Hunting_Threat_Intelligence_Incidents_And_Response_Actions

## Objective

Use Microsoft Defender XDR Advanced Hunting, Threat Intelligence, incidents, alerts, evidence, and response actions to investigate suspicious activity and perform controlled security response tasks.

## Lab Context

This workbook assumes Microsoft Defender XDR is active and receiving telemetry from one or more workloads such as Defender for Endpoint, Defender for Office 365, Defender for Cloud Apps, or Microsoft Entra ID. The goal is to practice investigation and response without damaging production evidence.

## Prerequisites

| Requirement | Details |
|---|---|
| Admin Access | Security Reader, Security Operator, Security Administrator, Global Administrator, or Defender role with hunting and response permissions |
| Portal | https://security.microsoft.com |
| Licensing | Microsoft Defender XDR with relevant Defender workload licenses |
| Data Sources | Endpoint, email, identity, and cloud app telemetry preferred |
| Test Device | Onboarded Defender for Endpoint test device recommended |
| Test Mailbox | Exchange Online mailbox recommended |
| Browser | Edge or Chrome |
| Change Control | Required before using real response actions |

## Configuration Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|---:|---|---|---|---|---|
| 1 | Open Microsoft Defender Portal | Admin Workstation | Browse to https://security.microsoft.com | N/A | Defender portal loads |
| 2 | Verify Tenant Context | Admin Workstation | Confirm tenant from profile menu | N/A | Correct tenant displayed |
| 3 | Open Advanced Hunting | Defender Portal | Hunting → Advanced Hunting | N/A | Advanced Hunting query editor opens |
| 4 | Review Schema Pane | Defender Portal | Advanced Hunting → Schema | N/A | Available tables visible |
| 5 | Review Endpoint Tables | Defender Portal | Search schema for `Device` | N/A | Device tables visible if endpoint telemetry exists |
| 6 | Review Email Tables | Defender Portal | Search schema for `Email` | N/A | Email tables visible if email telemetry exists |
| 7 | Review Alert Tables | Defender Portal | Search schema for `Alert` | N/A | Alert tables visible |
| 8 | Review Cloud App Tables | Defender Portal | Search schema for `CloudAppEvents` | N/A | Cloud app table visible if telemetry exists |
| 9 | Run Basic Alert Query | Defender Portal | Run query from hunting section | N/A | Recent alerts returned if data exists |
| 10 | Run Basic Device Query | Defender Portal | Run query from hunting section | N/A | Recent endpoint events returned if data exists |
| 11 | Run Basic Email Query | Defender Portal | Run query from hunting section | N/A | Recent email events returned if data exists |
| 12 | Run Basic Cloud App Query | Defender Portal | Run query from hunting section | N/A | Recent cloud app events returned if data exists |
| 13 | Save A Hunting Query | Defender Portal | Advanced Hunting → Save query | N/A | Query saved |
| 14 | Create Query Folder | Defender Portal | Save query into folder if available | N/A | Query organized |
| 15 | Export Query Results | Defender Portal | Export results | N/A | CSV exported if permitted |
| 16 | Open Query Result Entity | Defender Portal | Select device, user, alert, URL, IP, or file entity | N/A | Entity page opens |
| 17 | Pivot From Hunting To Device Page | Defender Portal | Select DeviceName or DeviceId | N/A | Device investigation page opens |
| 18 | Pivot From Hunting To User Page | Defender Portal | Select AccountUpn or user entity | N/A | User investigation page opens |
| 19 | Pivot From Hunting To Alert Page | Defender Portal | Select AlertId | N/A | Alert details open |
| 20 | Open Threat Intelligence | Defender Portal | Threat Intelligence | N/A | Threat intelligence page opens |
| 21 | Search Indicator | Defender Portal | Search IP, URL, domain, file hash, or CVE | N/A | Indicator results displayed if known |
| 22 | Review Indicator Details | Defender Portal | Select indicator | N/A | Indicator reputation and related evidence visible |
| 23 | Review Related Alerts | Defender Portal | Indicator details → Related alerts | N/A | Related alerts visible if present |
| 24 | Review Related Incidents | Defender Portal | Indicator details → Related incidents | N/A | Related incidents visible if present |
| 25 | Open Incidents Queue | Defender Portal | Incidents & Alerts → Incidents | N/A | Incidents queue opens |
| 26 | Filter Incidents By Severity | Defender Portal | Severity filter | N/A | High priority incidents visible |
| 27 | Filter Incidents By Status | Defender Portal | Status filter | N/A | New or active incidents visible |
| 28 | Filter Incidents By Service Source | Defender Portal | Service source filter | N/A | Incidents grouped by Defender workload |
| 29 | Open Incident | Defender Portal | Select incident | N/A | Incident details open |
| 30 | Review Incident Summary | Defender Portal | Incident overview | N/A | Severity, status, category, and impacted entities visible |
| 31 | Review Incident Story | Defender Portal | Incident story tab | N/A | Timeline and attack chain visible |
| 32 | Review Alerts In Incident | Defender Portal | Alerts tab | N/A | Related alerts visible |
| 33 | Review Evidence And Response | Defender Portal | Evidence and Response tab | N/A | Evidence entities and response options visible |
| 34 | Review Impacted Devices | Defender Portal | Devices or evidence entities | N/A | Impacted endpoints identified |
| 35 | Review Impacted Users | Defender Portal | Users or evidence entities | N/A | Impacted users identified |
| 36 | Review Email Evidence | Defender Portal | Mailbox, message, URL, attachment evidence | N/A | Email artifacts visible if present |
| 37 | Assign Incident Owner | Defender Portal | Assigned to field | N/A | Owner recorded |
| 38 | Add Incident Comment | Defender Portal | Comments | N/A | Investigation note saved |
| 39 | Change Incident Status If Authorized | Defender Portal | Set New, Active, or Resolved | N/A | Incident status updated |
| 40 | Classify Incident If Authorized | Defender Portal | Classification field | N/A | Incident classification recorded |
| 41 | Open Alert Details | Defender Portal | Incident → Alerts → Select alert | N/A | Alert details visible |
| 42 | Review Alert Timeline | Defender Portal | Alert timeline | N/A | Detection timeline visible |
| 43 | Review Alert Evidence | Defender Portal | Evidence tab | N/A | Alert evidence visible |
| 44 | Review Alert Recommended Actions | Defender Portal | Alert details | N/A | Suggested remediation visible |
| 45 | Open Action Center | Defender Portal | Actions & Submissions → Action Center | N/A | Action center opens |
| 46 | Review Pending Actions | Defender Portal | Action Center → Pending | N/A | Pending response actions visible |
| 47 | Review Completed Actions | Defender Portal | Action Center → History | N/A | Completed actions visible |
| 48 | Review Failed Actions | Defender Portal | Action Center filters | N/A | Failed actions identified if present |
| 49 | Open Device Response Actions | Defender Portal | Device page → Actions | N/A | Device response actions visible |
| 50 | Collect Investigation Package If Approved | Defender Portal | Device → Collect investigation package | N/A | Collection action submitted |
| 51 | Run Antivirus Scan If Approved | Defender Portal | Device → Run antivirus scan | N/A | Scan action submitted |
| 52 | Isolate Test Device If Approved | Defender Portal | Device → Isolate device | N/A | Test device isolated |
| 53 | Release Test Device From Isolation | Defender Portal | Device → Release from isolation | N/A | Test device network access restored |
| 54 | Restrict App Execution If Approved | Defender Portal | Device → Restrict app execution | N/A | Restriction action submitted |
| 55 | Remove App Restriction | Defender Portal | Device → Remove app restriction | N/A | App restriction removed |
| 56 | Open File Response Actions | Defender Portal | File entity page → Actions | N/A | File response actions visible |
| 57 | Stop And Quarantine File If Approved | Defender Portal | File → Stop and quarantine | N/A | File quarantine action submitted |
| 58 | Add Indicator For File Hash If Approved | Defender Portal | Settings → Endpoints → Indicators → Files | N/A | File indicator created |
| 59 | Add Indicator For URL Or Domain If Approved | Defender Portal | Settings → Endpoints → Indicators → URLs/Domains | N/A | URL or domain indicator created |
| 60 | Add Indicator For IP If Approved | Defender Portal | Settings → Endpoints → Indicators → IP addresses | N/A | IP indicator created |
| 61 | Open Email Response Actions | Defender Portal | Explorer or incident evidence → Email actions | N/A | Email response actions visible |
| 62 | Soft Delete Test Email If Approved | Defender Portal | Email action → Soft delete | N/A | Test message removed to recoverable deleted items |
| 63 | Move Test Email To Junk If Approved | Defender Portal | Email action → Move to junk | N/A | Test message moved to Junk |
| 64 | Submit Email To Microsoft If Needed | Defender Portal | Actions & Submissions → Submissions | N/A | Email submitted |
| 65 | Open Live Response If Approved | Defender Portal | Device page → Live response | N/A | Live response session starts if enabled |
| 66 | Run Read-Only Live Response Command | Defender Portal | Live response command window | N/A | Basic command output displayed |
| 67 | Close Live Response Session | Defender Portal | End session | N/A | Session closed |
| 68 | Confirm Response Action Status | Defender Portal | Action Center | N/A | Response action completed, pending, or failed |
| 69 | Document Investigation And Response | Admin Workstation | Record query, indicators, entities, incident status, response action, and outcome | N/A | Investigation record completed |

## Advanced Hunting Queries

### Recent Alerts

```kql
AlertInfo
| where Timestamp > ago(7d)
| project Timestamp, AlertId, Title, Severity, Category, ServiceSource, DetectionSource
| order by Timestamp desc
```

### Recent Incidents

```kql
SecurityIncident
| where CreatedTime > ago(30d)
| project IncidentName, CreatedTime, Severity, Status, Classification, AssignedTo
| order by CreatedTime desc
```

### Alerts Joined With Evidence

```kql
AlertInfo
| where Timestamp > ago(7d)
| join kind=leftouter AlertEvidence on AlertId
| project Timestamp, Title, Severity, ServiceSource, EntityType, DeviceName, AccountName, FileName, RemoteUrl, RemoteIP
| order by Timestamp desc
```

### Recent Device Process Events

```kql
DeviceProcessEvents
| where Timestamp > ago(24h)
| project Timestamp, DeviceName, AccountName, FileName, ProcessCommandLine, InitiatingProcessFileName, InitiatingProcessCommandLine
| order by Timestamp desc
```

### Recent Device Network Events

```kql
DeviceNetworkEvents
| where Timestamp > ago(24h)
| project Timestamp, DeviceName, InitiatingProcessFileName, RemoteUrl, RemoteIP, RemotePort, Protocol
| order by Timestamp desc
```

### Recent Device File Events

```kql
DeviceFileEvents
| where Timestamp > ago(24h)
| project Timestamp, DeviceName, ActionType, FileName, FolderPath, SHA256, InitiatingProcessFileName
| order by Timestamp desc
```

### Search For A File Hash

```kql
let TargetHash = "PASTE_SHA256_HERE";
DeviceFileEvents
| where Timestamp > ago(30d)
| where SHA256 =~ TargetHash
| project Timestamp, DeviceName, ActionType, FileName, FolderPath, SHA256, InitiatingProcessFileName
| order by Timestamp desc
```

### Search For A Domain Or URL

```kql
let TargetDomain = "example.com";
DeviceNetworkEvents
| where Timestamp > ago(30d)
| where RemoteUrl has TargetDomain
| project Timestamp, DeviceName, InitiatingProcessFileName, RemoteUrl, RemoteIP, RemotePort
| order by Timestamp desc
```

### Recent Email Threats

```kql
EmailEvents
| where Timestamp > ago(7d)
| where isnotempty(ThreatTypes)
| project Timestamp, SenderFromAddress, RecipientEmailAddress, Subject, ThreatTypes, DetectionMethods, DeliveryAction, NetworkMessageId
| order by Timestamp desc
```

### Recent URL Click Events

```kql
UrlClickEvents
| where Timestamp > ago(7d)
| project Timestamp, AccountUpn, Url, ActionType, Workload, NetworkMessageId
| order by Timestamp desc
```

### Recent Cloud App Events

```kql
CloudAppEvents
| where Timestamp > ago(7d)
| project Timestamp, AccountDisplayName, Application, ActionType, IPAddress, CountryCode
| order by Timestamp desc
```

### Suspicious PowerShell Activity

```kql
DeviceProcessEvents
| where Timestamp > ago(7d)
| where FileName in~ ("powershell.exe", "pwsh.exe")
| where ProcessCommandLine has_any ("-enc", "EncodedCommand", "Invoke-WebRequest", "IEX", "DownloadString", "FromBase64String")
| project Timestamp, DeviceName, AccountName, FileName, ProcessCommandLine, InitiatingProcessFileName
| order by Timestamp desc
```

### Suspicious Office Child Processes

```kql
DeviceProcessEvents
| where Timestamp > ago(7d)
| where InitiatingProcessFileName in~ ("winword.exe", "excel.exe", "powerpnt.exe", "outlook.exe")
| where FileName in~ ("cmd.exe", "powershell.exe", "wscript.exe", "cscript.exe", "mshta.exe", "rundll32.exe")
| project Timestamp, DeviceName, AccountName, InitiatingProcessFileName, FileName, ProcessCommandLine
| order by Timestamp desc
```

### Rare Remote IP Connections

```kql
DeviceNetworkEvents
| where Timestamp > ago(7d)
| where isnotempty(RemoteIP)
| summarize DeviceCount=dcount(DeviceName), EventCount=count() by RemoteIP
| where DeviceCount <= 2 and EventCount < 10
| order by EventCount asc
```

## Response Action Reference

| Response Action | Entity | Use Case | Risk |
|---|---|---|---|
| Assign incident | Incident | Set ownership | Low |
| Add comment | Incident | Preserve investigation notes | Low |
| Classify alert or incident | Alert or Incident | Record verdict | Low |
| Run antivirus scan | Device | Check suspected endpoint | Medium |
| Collect investigation package | Device | Gather triage artifacts | Medium |
| Isolate device | Device | Contain active compromise | High |
| Release from isolation | Device | Restore connectivity | High |
| Restrict app execution | Device | Prevent untrusted apps from running | High |
| Stop and quarantine file | File | Remove malicious file | High |
| Add file indicator | File hash | Block or allow file hash | High |
| Add URL or domain indicator | URL or domain | Block or allow web indicator | High |
| Add IP indicator | IP address | Block or allow IP indicator | High |
| Soft delete email | Email | Remove malicious message | Medium |
| Move email to junk | Email | Reduce user exposure | Medium |
| Submit to Microsoft | Email, URL, file | Request verdict review | Low |
| Live Response | Device | Interactive investigation | High |

## Validation

| Check | Expected Result |
|---|---|
| Advanced Hunting Opens | Success |
| Schema Pane Shows Tables | Success |
| Alert Query Runs | Success |
| Device Query Runs If Endpoint Data Exists | Success |
| Email Query Runs If Mail Data Exists | Success |
| Cloud App Query Runs If Cloud App Data Exists | Success |
| Threat Intelligence Search Opens | Success |
| Incident Queue Opens | Success |
| Incident Details Open | Success |
| Alert Details Open | Success |
| Evidence And Response Tab Opens | Success |
| Action Center Opens | Success |
| Pending And Completed Actions Visible | Success |
| Device Response Actions Visible If Authorized | Success |
| Email Response Actions Visible If Authorized | Success |
| Indicators Page Accessible If Authorized | Success |
| Investigation Notes Documented | Success |

## Troubleshooting

| Symptom | Likely Cause | Resolution |
|---|---|---|
| Advanced Hunting Missing | Missing role or license | Verify Defender XDR licensing and security role |
| Hunting Table Missing | Workload not onboarded | Onboard endpoint, email, identity, or cloud app data source |
| Query Returns No Results | Time range too narrow or no telemetry | Expand time range and verify data source |
| Query Fails | KQL syntax error | Check table names, columns, quotes, and pipes |
| Threat Intelligence Page Missing | Missing license or permission | Verify security role and Defender capabilities |
| Incident Queue Empty | No incidents or filters too narrow | Clear filters and expand time range |
| Evidence Tab Empty | Alert has limited evidence | Review alert source and related entities |
| Response Action Missing | Insufficient permission or unsupported entity | Verify role and supported action |
| Device Isolation Fails | Device offline or unsupported OS | Confirm device is online and onboarded |
| Live Response Missing | Feature disabled | Enable Live Response in endpoint advanced features |
| Live Response Fails | Device offline, unsupported, or role missing | Check device status, OS support, and permissions |
| Email Action Fails | Message already remediated or insufficient permission | Check message status and role |
| Indicator Creation Fails | Indicator feature disabled or invalid value | Enable custom indicators and validate hash, URL, domain, or IP |
| Action Center Shows Failed | Device offline or backend action failed | Review action details and retry only if appropriate |

## Rollback

| Step | Action | Expected Result |
|---:|---|---|
| 1 | Release Test Device From Isolation | Device network access restored |
| 2 | Remove App Execution Restriction | Device can run normal applications |
| 3 | Remove Test Indicators | Test blocks or allows removed |
| 4 | Close Live Response Session | Interactive session ended |
| 5 | Restore Test Email If Needed | Message restored from recoverable location if possible |
| 6 | Revert Test Incident Classification If Needed | Incident metadata corrected |
| 7 | Clear Hunting Filters | Default hunting view restored |
| 8 | Archive Investigation Notes | Evidence and decisions preserved |

## Documentation Checklist

| Item | Status |
|---|---|
| Tenant Verified | ☐ |
| Investigator Account Recorded | ☐ |
| Defender Roles Verified | ☐ |
| Hunting Query Used Recorded | ☐ |
| Time Range Recorded | ☐ |
| Alert ID Recorded | ☐ |
| Incident Name Recorded | ☐ |
| Device Name Recorded | ☐ |
| User Account Recorded | ☐ |
| File Hash Recorded | ☐ |
| URL Or Domain Recorded | ☐ |
| IP Address Recorded | ☐ |
| Email Network Message ID Recorded | ☐ |
| Threat Intelligence Result Recorded | ☐ |
| Incident Severity Recorded | ☐ |
| Incident Status Recorded | ☐ |
| Evidence Reviewed | ☐ |
| Response Action Taken | ☐ |
| Action Center Result Recorded | ☐ |
| Rollback Completed If Needed | ☐ |
| Final Classification Recorded | ☐ |

## Notes

- Advanced Hunting is only as useful as the telemetry sources connected to Defender XDR.
- Always start with read-only investigation before taking response actions.
- Do not isolate production devices unless approved through incident response procedures.
- Do not create block indicators broadly without validating business impact.
- Live Response should be limited to trained operators.
- Action Center is the place to verify whether response actions completed, failed, or are still pending.
- Incident comments should explain why actions were taken, not just what button was clicked.
- Preserve evidence before deleting email, quarantining files, or closing incidents.