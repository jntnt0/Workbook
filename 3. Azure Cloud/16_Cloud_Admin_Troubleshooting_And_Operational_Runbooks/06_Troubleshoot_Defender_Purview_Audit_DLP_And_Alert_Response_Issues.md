# 06_Troubleshoot_Defender_Purview_Audit_DLP_And_Alert_Response_Issues

## 06_Troubleshoot_Defender_Purview_Audit_DLP_And_Alert_Response_Issues_Index

### Purpose

This workbook provides a structured troubleshooting and response workflow for Microsoft Defender, Microsoft Purview, audit logging, DLP, alert triage, incident investigation, evidence collection, containment, remediation, and escalation.

### Scope

Use this workbook when Microsoft 365, Defender, or Purview shows one or more of the following symptoms:

- Defender incident or alert is generated and needs triage
- Alert is missing, delayed, duplicated, noisy, or not assigned correctly
- Defender for Office 365 detects phishing, malware, spam, spoofing, or user compromise indicators
- Defender for Endpoint detects device, vulnerability, malware, lateral movement, or suspicious process activity
- Defender for Cloud Apps detects suspicious activity, impossible travel, mass download, OAuth app abuse, or shadow IT risk
- Advanced Hunting query is needed to validate impact
- Purview audit search returns no data or incomplete results
- Unified audit log does not show expected event
- DLP policy does not trigger, triggers incorrectly, or blocks legitimate activity
- DLP alert, activity explorer, content explorer, or endpoint DLP event needs investigation
- Quarantine, submissions, restricted entities, or threat explorer investigation is required
- Alert response action needs to be performed safely and documented
- Security/compliance incident needs evidence, containment, rollback, and handoff

### Assumptions

- You have access to Microsoft Defender portal and Microsoft Purview portal.
- You can connect to Exchange Online PowerShell and Microsoft Graph PowerShell where needed.
- You know the affected user, device, mailbox, file, alert, incident, or policy.
- You will collect evidence before closing alerts, deleting data, releasing quarantine, disabling policy, or changing DLP actions.
- You will not suppress alerts or weaken DLP policies without a documented reason and rollback plan.
- You will coordinate with security, compliance, legal, and workload owners when data loss, compromise, malware, or regulated data is involved.

### Required Admin Roles

| Task Area | Minimum Role |
|---|---|
| View Defender incidents and alerts | Security Reader, Security Operator, Security Administrator |
| Manage Defender incidents and alerts | Security Operator, Security Administrator |
| Run Advanced Hunting | Security Reader or Security Operator with hunting access |
| Manage Defender for Office 365 policies | Security Administrator, Exchange Administrator |
| Review quarantine | Security Reader, Security Operator, Quarantine Administrator, Exchange Administrator depending action |
| Submit messages/files/URLs | Security Operator, Security Administrator |
| View restricted entities | Security Reader, Security Administrator |
| Manage Defender for Endpoint actions | Security Operator, Security Administrator |
| View Purview audit | Audit Reader, Compliance Administrator, Global Reader |
| Run audit searches | Audit Reader, Compliance Administrator |
| Manage DLP policies | Compliance Administrator, Information Protection Administrator |
| View Activity Explorer / Content Explorer | Compliance Data Administrator, Content Explorer roles |
| View DLP alerts | Compliance Administrator, Security Reader, DLP Compliance Management |
| Manage eDiscovery handoff | eDiscovery Manager, Compliance Administrator |
| Emergency response | Global Administrator, Security Administrator, Compliance Administrator as appropriate |

---

## 06_Troubleshoot_Defender_Purview_Audit_DLP_And_Alert_Response_Issues_Mental_Model

Security and compliance troubleshooting has two parallel tracks:

1. Detection and response: Defender incident, alert, evidence, entity, timeline, hunting, containment, remediation, and closure.
2. Governance and evidence: Purview audit, DLP policy, sensitive data match, activity trail, alert policy, legal/compliance retention, and reporting.

Do not start by closing alerts or disabling policies. Start by proving what happened.

Response chain:

1. Confirm alert or policy signal.
2. Confirm affected entities: user, mailbox, device, IP, app, file, site, policy, or data type.
3. Confirm timeline.
4. Collect raw evidence.
5. Determine whether event is true positive, benign positive, false positive, test, or policy misconfiguration.
6. Scope blast radius using logs, audit, hunting, message trace, and DLP activity.
7. Contain if there is active risk.
8. Remediate root cause.
9. Restore legitimate access or release legitimate content if needed.
10. Tune policy only after proving noise pattern.
11. Document evidence, actions, rollback, and escalation.

Traditional troubleshooting order:

1. Confirm service health and portal availability.
2. Confirm exact incident, alert, DLP policy, audit search, or user report.
3. Capture incident/alert IDs, timestamps, entities, and status.
4. Pull related logs and evidence.
5. Scope impact.
6. Decide severity and response path.
7. Apply containment only when justified.
8. Remediate and validate.
9. Close or tune only after evidence.
10. Record incident review and handoff.

---

## 06_Troubleshoot_Defender_Purview_Audit_DLP_And_Alert_Response_Issues_Symptom_Map

| Symptom | Likely Cause | Primary Evidence | First Check |
|---|---|---|---|
| Defender incident needs triage | True positive, false positive, noisy rule, correlated alerts | Incident page and alert evidence | Incident timeline |
| Alert not firing | Policy disabled, signal missing, license/role issue, ingestion delay, alert suppression | Alert policy and activity logs | Alert rule status |
| Alert fires repeatedly | Noisy detection, unresolved root cause, repeated user behavior, test traffic | Alert history and entity timeline | Alert frequency and entities |
| Phishing alert | Malicious message, spoofing, user submission, campaign | Threat Explorer and message headers | Message trace / Explorer |
| User listed as restricted entity | Outbound spam, compromised account, suspicious sending | Restricted entities page and message trace | User send activity |
| Malware alert on device | Malicious file/process, EDR detection, unsafe remediation | Device timeline | Defender for Endpoint alert evidence |
| Device isolation fails | Device offline, unsupported OS, permission issue, MDE onboarding issue | Device action center | Device health |
| Advanced Hunting returns no data | Wrong table, wrong time range, role/license issue, sensor not onboarded | Hunting query result | Table/time range |
| Audit search missing expected event | Audit not enabled/available, wrong workload, wrong time range, retention limit, permission issue | Purview audit search | Audit scope and time |
| DLP does not trigger | Policy not enabled, wrong location, wrong condition, test mode, content not matched | DLP policy and activity | Policy mode |
| DLP triggers incorrectly | Sensitive info type overmatch, trainable classifier, condition too broad, exception missing | DLP alert details | Matched condition |
| Endpoint DLP missing event | Device not onboarded, endpoint DLP not enabled, policy not assigned | Endpoint DLP settings and device state | Device onboarding |
| Activity Explorer missing data | Role issue, data delay, wrong filters, unsupported workload | Activity Explorer filters | Time range and filters |
| Content Explorer inaccessible | Missing role or role group | Purview permissions | Content Explorer role |
| Quarantine release blocked | Permission, policy, malware verdict, admin review required | Quarantine item details | Quarantine permissions |
| Incident closure rejected by process | Missing classification, required comments, unresolved tasks | Incident status | Closure fields |

---

## 06_Troubleshoot_Defender_Purview_Audit_DLP_And_Alert_Response_Issues_Evidence_Collection

| Evidence | Where To Get It | Why It Matters |
|---|---|---|
| Incident ID | Defender portal | Primary response object |
| Alert ID | Defender portal | Needed for alert-level triage |
| Detection source | Defender alert details | Identifies workload and data source |
| Severity | Defender portal | Drives SLA and escalation |
| Classification | Defender incident/alert | True positive, benign positive, false positive |
| Determination | Defender incident/alert | Malware, phishing, user activity, policy issue, etc. |
| Affected user | Alert entity / audit log | Required for identity scope |
| Affected mailbox | Defender for Office / Exchange | Required for message investigation |
| Affected device | Defender for Endpoint | Required for device timeline |
| Affected file | Defender / Purview | Required for file and DLP investigation |
| Affected URL/domain/IP | Defender evidence | Required for IOC scope |
| Timestamp with timezone | Alert details / user report | Needed for hunting and audit queries |
| Message ID / NetworkMessageId | Threat Explorer / message trace | Required for email investigation |
| Device ID | Defender for Endpoint | Required for hunting and response action |
| DLP policy name | Purview DLP alert | Required for policy review |
| Sensitive info type | DLP alert details | Explains DLP match |
| Audit operation | Purview audit result | Explains user/admin activity |
| KQL query output | Advanced Hunting | Supports scope and conclusion |
| Quarantine item | Defender portal | Supports release/delete decision |
| Service health | M365 admin center | Confirms platform issue or outage |
| Recent policy changes | Audit logs / admin portal | Explains alert/DLP behavior change |
| Response actions | Action center / audit logs | Documents containment and remediation |

---

## 06_Troubleshoot_Defender_Purview_Audit_DLP_And_Alert_Response_Issues_Triage_Order

1. Confirm whether the issue is security detection, compliance detection, missing audit evidence, DLP behavior, or response action failure.
2. Check Microsoft 365 service health if portal, ingestion, or alert delay is suspected.
3. Capture incident ID, alert ID, DLP policy, audit search, affected entities, and timestamps.
4. Open the incident or alert and review evidence, timeline, entities, and related alerts.
5. Classify the issue as true positive, false positive, benign positive, policy misconfiguration, or service/data delay.
6. Scope affected users, mailboxes, devices, files, URLs, IPs, apps, and data locations.
7. Use Advanced Hunting, message trace, audit search, Activity Explorer, and DLP alert details to confirm impact.
8. Apply containment only when active risk is present.
9. Remediate root cause.
10. Restore legitimate access/content if blocked incorrectly.
11. Tune alert or DLP policy only after evidence supports tuning.
12. Verify the alert, audit, DLP, or response behavior.
13. Record evidence, actions, rollback, and escalation notes.

---

## 06_Troubleshoot_Defender_Purview_Audit_DLP_And_Alert_Response_Issues_Planning_Table

| Item | Value |
|---|---|
| Tenant name | `<tenant-name>` |
| Tenant ID | `<tenant-id>` |
| Incident ID | `<incident-id>` |
| Alert ID | `<alert-id>` |
| Workload | `<Defender XDR / Defender for Office 365 / Defender for Endpoint / Defender for Cloud Apps / Purview / DLP / Audit>` |
| Severity | `<Informational / Low / Medium / High>` |
| Status | `<New / In progress / Resolved>` |
| Classification | `<True positive / Benign positive / False positive / Unknown>` |
| Determination | `<Phishing / Malware / Data loss / Suspicious activity / Policy misconfiguration / Other>` |
| Affected user | `<user@domain.com>` |
| Affected mailbox | `<mailbox@domain.com>` |
| Affected device | `<device-name>` |
| Affected file | `<file-name-or-path>` |
| Affected SharePoint/OneDrive site | `<site-url>` |
| Affected message ID | `<message-id-or-network-message-id>` |
| Affected IP/domain/URL | `<indicator>` |
| DLP policy | `<policy-name>` |
| DLP rule | `<rule-name>` |
| Sensitive info type | `<sensitive-info-type>` |
| Audit operation | `<operation>` |
| First event time | `<yyyy-mm-dd hh:mm timezone>` |
| Last event time | `<yyyy-mm-dd hh:mm timezone>` |
| Response owner | `<owner>` |
| Change ticket | `<ticket-id>` |
| Business impact | `<impact>` |

---

## 06_Troubleshoot_Defender_Purview_Audit_DLP_And_Alert_Response_Issues_Configuration_Checklist

| Step | Task | Device | Command | PowerShell | Expected Result |
|---|---|---|---|---|---|
| 1 | Confirm service health | Admin workstation | Microsoft 365 admin center > Health > Service health | `Get-MgServiceAnnouncementIssue -Top 20` | No service incident explains missing alerts, audit, or portal failures |
| 2 | Confirm Defender portal access | Admin workstation | Defender portal > Incidents & alerts | Not applicable | Incidents and alerts are visible |
| 3 | Confirm Purview portal access | Admin workstation | Purview portal > Audit / DLP / Activity Explorer | Not applicable | Audit and DLP areas are visible |
| 4 | Capture incident details | Admin workstation | Defender portal > Incidents > select incident | Not applicable | Incident ID, severity, status, entities, and timeline are recorded |
| 5 | Capture alert details | Admin workstation | Defender portal > Alerts > select alert | Not applicable | Alert ID, detection source, evidence, and entities are recorded |
| 6 | Review related entities | Admin workstation | Incident > Evidence and response | Not applicable | Users, devices, mailboxes, files, URLs, IPs, and apps are identified |
| 7 | Review timeline | Admin workstation | Incident > Timeline | Not applicable | Event sequence is understood |
| 8 | Run initial hunting query | Admin workstation | Defender portal > Hunting > Advanced Hunting | Use KQL from verification section | Impact scope is confirmed |
| 9 | Check user sign-ins if identity involved | Admin workstation | Entra admin center > Sign-in logs | `Get-MgAuditLogSignIn -Filter "userPrincipalName eq '<user@domain.com>'" -Top 20` | Suspicious sign-in activity is confirmed or ruled out |
| 10 | Check mailbox messages if email involved | Admin workstation | Defender portal > Explorer | `Get-MessageTrace -RecipientAddress <user@domain.com> -StartDate (Get-Date).AddDays(-2) -EndDate (Get-Date)` | Message delivery path is known |
| 11 | Check quarantine if email was blocked | Admin workstation | Defender portal > Email & collaboration > Review > Quarantine | Not applicable | Quarantined message state is known |
| 12 | Check restricted entities if outbound mail issue | Admin workstation | Defender portal > Email & collaboration > Review > Restricted entities | Not applicable | User or connector restriction is identified |
| 13 | Check submissions if user/admin submitted item | Admin workstation | Defender portal > Email & collaboration > Submissions | Not applicable | Submission result and verdict are known |
| 14 | Check device timeline if endpoint involved | Admin workstation | Defender portal > Assets > Devices > device > Timeline | Not applicable | Device process/file/network timeline is reviewed |
| 15 | Check device action center | Admin workstation | Defender portal > Action center | Not applicable | Pending/successful/failed response actions are visible |
| 16 | Check Defender for Cloud Apps activity if SaaS/OAuth involved | Admin workstation | Defender portal > Cloud Apps > Activity log | Not applicable | App and activity evidence is visible |
| 17 | Search unified audit log | Admin workstation | Purview portal > Audit | `Search-UnifiedAuditLog -StartDate (Get-Date).AddDays(-7) -EndDate (Get-Date) -UserIds <user@domain.com>` | User/admin activity is visible or absence is documented |
| 18 | Check DLP alerts | Admin workstation | Purview portal > Data loss prevention > Alerts | Not applicable | DLP alert, policy, rule, and matched condition are known |
| 19 | Check DLP policy mode | Admin workstation | Purview portal > DLP > Policies > policy settings | Not applicable | Policy is On, Test, or Test with notifications |
| 20 | Check DLP locations | Admin workstation | DLP policy > Locations | Not applicable | Exchange, SharePoint, OneDrive, Teams, Devices, or Power BI targeting is correct |
| 21 | Check DLP conditions and exceptions | Admin workstation | DLP policy > Rules | Not applicable | Match logic is understood |
| 22 | Check Activity Explorer | Admin workstation | Purview portal > Activity Explorer | Not applicable | DLP-related activity is found |
| 23 | Check Content Explorer access if needed | Admin workstation | Purview portal > Content Explorer | Not applicable | Sensitive content can be reviewed by authorized role |
| 24 | Check endpoint DLP device scope | Admin workstation | Purview portal > Endpoint DLP settings | Not applicable | Device onboarding and policy targeting are correct |
| 25 | Apply containment if active risk exists | Admin workstation | Disable user, revoke sessions, isolate device, block URL/file, quarantine message, remove OAuth grant | Depends on containment action | Active risk is reduced |
| 26 | Apply remediation | Admin workstation | Remove malicious mail, reset password, remediate device, fix policy, release legitimate content | Depends on remediation | Root cause is addressed |
| 27 | Verify with hunting/audit/DLP retest | Admin workstation | Rerun query, audit search, message trace, or DLP test | Depends on validation | Issue is resolved or scoped |
| 28 | Classify and close alert/incident | Admin workstation | Defender portal > Incident > Manage incident | Not applicable | Incident is closed with correct classification and comments |
| 29 | Record response actions and rollback | Admin workstation | Update incident/change record | Not applicable | Evidence, actions, decisions, and rollback are documented |

---

## 06_Troubleshoot_Defender_Purview_Audit_DLP_And_Alert_Response_Issues_Defender_Incident_Workflow

### Phase 1: Incident Triage

| Step | Task | Device | Command | PowerShell | Expected Result |
|---|---|---|---|---|---|
| 1 | Open incident | Admin workstation | Defender portal > Incidents & alerts > Incidents | Not applicable | Incident details are visible |
| 2 | Record incident metadata | Admin workstation | Incident summary | Not applicable | ID, severity, status, detection source, and owner are recorded |
| 3 | Review alert queue | Admin workstation | Incident > Alerts | Not applicable | Related alerts are listed |
| 4 | Review evidence | Admin workstation | Incident > Evidence and response | Not applicable | Entities are identified |
| 5 | Review timeline | Admin workstation | Incident > Timeline | Not applicable | Event order is understood |
| 6 | Review investigation graph | Admin workstation | Incident > Attack story if available | Not applicable | Attack chain is understood |
| 7 | Assign owner | Admin workstation | Incident > Manage incident | Not applicable | Response owner is assigned |
| 8 | Set severity if incorrect | Admin workstation | Incident > Manage incident | Not applicable | Severity reflects business and security risk |
| 9 | Add response notes | Admin workstation | Incident comments | Not applicable | Triage notes are preserved |
| 10 | Decide classification | Admin workstation | Based on evidence | Not applicable | True positive, benign positive, false positive, or unknown is selected |

### Phase 2: Scope And Impact

| Step | Task | Device | Command | PowerShell | Expected Result |
|---|---|---|---|---|---|
| 1 | Identify users | Admin workstation | Incident > Evidence | Advanced Hunting user queries | Affected users are listed |
| 2 | Identify devices | Admin workstation | Incident > Evidence | Advanced Hunting device queries | Affected devices are listed |
| 3 | Identify mailboxes/messages | Admin workstation | Defender Explorer | Exchange message trace / hunting queries | Affected messages are listed |
| 4 | Identify files | Admin workstation | Incident > Evidence > files | Advanced Hunting file queries | Affected files are listed |
| 5 | Identify URLs/domains/IPs | Admin workstation | Incident evidence | Advanced Hunting network queries | Indicators are listed |
| 6 | Check related incidents | Admin workstation | Defender incident queue search | Not applicable | Duplicate or related incident is identified |
| 7 | Check recent audit activity | Admin workstation | Purview audit | `Search-UnifiedAuditLog -StartDate (Get-Date).AddDays(-7) -EndDate (Get-Date) -UserIds <user@domain.com>` | Relevant user/admin activity is captured |
| 8 | Determine active risk | Admin workstation | Evidence, timeline, hunting | Not applicable | Containment decision is justified |
| 9 | Record blast radius | Admin workstation | Incident record | Not applicable | Users/devices/messages/files affected are documented |
| 10 | Escalate if scope exceeds authority | Admin workstation | Notify security/compliance/legal/management | Not applicable | Correct responders are engaged |

### Phase 3: Response Actions

| Step | Task | Device | Command | PowerShell | Expected Result |
|---|---|---|---|---|---|
| 1 | Revoke user sessions if account compromise suspected | Admin workstation | Entra admin center > User > Revoke sessions | `Revoke-MgUserSignInSession -UserId <user@domain.com>` | Existing refresh tokens are invalidated |
| 2 | Reset password if compromise confirmed | Admin workstation | Microsoft 365 admin center > User > Reset password | `Update-MgUser -UserId <user@domain.com> -PasswordProfile @{forceChangePasswordNextSignIn=$true; password="<temporary-password>"}` | User password is reset with approved process |
| 3 | Disable user if active compromise | Admin workstation | Microsoft 365 admin center > Block sign-in | `Update-MgUser -UserId <user@domain.com> -AccountEnabled:$false` | User cannot sign in |
| 4 | Isolate device if active endpoint threat | Admin workstation | Defender portal > Device > Isolate device | Not applicable | Device isolation action is submitted |
| 5 | Run antivirus scan | Admin workstation | Defender portal > Device > Run antivirus scan | Not applicable | Scan action is submitted |
| 6 | Collect investigation package | Admin workstation | Defender portal > Device > Collect investigation package | Not applicable | Package collection starts |
| 7 | Remove malicious email | Admin workstation | Defender portal > Explorer > Take action | Not applicable | Message is soft-deleted, hard-deleted, or moved as approved |
| 8 | Block indicator if approved | Admin workstation | Defender portal > Settings > Endpoints/Email indicators | Not applicable | URL/domain/IP/file indicator is blocked |
| 9 | Remove malicious OAuth consent if confirmed | Admin workstation | Entra admin center > Enterprise applications > permissions | Graph app permission review | Risky app grant is removed |
| 10 | Verify action center | Admin workstation | Defender portal > Action center | Not applicable | Response action succeeded or failed with reason |

---

## 06_Troubleshoot_Defender_Purview_Audit_DLP_And_Alert_Response_Issues_Defender_For_Office_365_Workflow

### Phase 1: Email Threat Investigation

| Step | Task | Device | Command | PowerShell | Expected Result |
|---|---|---|---|---|---|
| 1 | Open Explorer | Admin workstation | Defender portal > Email & collaboration > Explorer | Not applicable | Email events are searchable |
| 2 | Search by sender | Admin workstation | Explorer filter sender | `Get-MessageTrace -SenderAddress <sender@domain.com> -StartDate (Get-Date).AddDays(-2) -EndDate (Get-Date)` | Sender activity is visible |
| 3 | Search by recipient | Admin workstation | Explorer filter recipient | `Get-MessageTrace -RecipientAddress <recipient@domain.com> -StartDate (Get-Date).AddDays(-2) -EndDate (Get-Date)` | Recipient activity is visible |
| 4 | Search by subject | Admin workstation | Explorer filter subject | Message trace and Explorer | Campaign scope is visible |
| 5 | Search by NetworkMessageId | Admin workstation | Explorer advanced filter | Use message details from trace | Exact message is found |
| 6 | Review verdict | Admin workstation | Explorer message details | Not applicable | Malware/phish/spam/good verdict is known |
| 7 | Review delivery location | Admin workstation | Explorer message details | Not applicable | Inbox, junk, quarantine, failed, or deleted state is known |
| 8 | Review URL and attachment details | Admin workstation | Explorer evidence | Not applicable | Malicious indicators are identified |
| 9 | Check quarantine | Admin workstation | Defender portal > Quarantine | Not applicable | Quarantined copy is found if present |
| 10 | Take action if approved | Admin workstation | Explorer > Take action | Not applicable | Message remediation action is submitted |

### Phase 2: Submissions, Quarantine, And Restricted Entities

| Step | Task | Device | Command | PowerShell | Expected Result |
|---|---|---|---|---|---|
| 1 | Review user submissions | Admin workstation | Defender portal > Submissions | Not applicable | Submitted items and verdicts are visible |
| 2 | Submit message/file/URL if needed | Admin workstation | Defender portal > Submissions > Submit to Microsoft | Not applicable | Submission is created |
| 3 | Review quarantine item | Admin workstation | Defender portal > Quarantine | Not applicable | Quarantine reason and policy are known |
| 4 | Release legitimate message if approved | Admin workstation | Quarantine > Release | Not applicable | Message is released to recipient |
| 5 | Delete malicious message if approved | Admin workstation | Quarantine or Explorer action | Not applicable | Message is removed |
| 6 | Check restricted entities | Admin workstation | Defender portal > Restricted entities | Not applicable | Restricted user/connector status is known |
| 7 | Investigate restricted user | Admin workstation | Message trace, sign-in logs, audit, inbox rules | Exchange and Graph checks | Compromise or false positive is assessed |
| 8 | Remove restriction only after remediation | Admin workstation | Restricted entities > Unblock | Not applicable | User can send after remediation |
| 9 | Check outbound spam policy if recurrence | Admin workstation | Defender portal > Policies & rules > Threat policies > Anti-spam | Not applicable | Policy cause is identified |
| 10 | Verify outbound send | User/admin workstation | Send test message | `Get-MessageTrace -SenderAddress <user@domain.com> -StartDate (Get-Date).AddMinutes(-30) -EndDate (Get-Date)` | Send path works |

---

## 06_Troubleshoot_Defender_Purview_Audit_DLP_And_Alert_Response_Issues_Defender_For_Endpoint_Workflow

### Phase 1: Device Alert Investigation

| Step | Task | Device | Command | PowerShell | Expected Result |
|---|---|---|---|---|---|
| 1 | Open device page | Admin workstation | Defender portal > Assets > Devices > device | Not applicable | Device overview is visible |
| 2 | Check device health | Admin workstation | Device page > Health status | Not applicable | Onboarding and sensor health are known |
| 3 | Review alert evidence | Admin workstation | Alert > Evidence | Not applicable | File/process/user/network evidence is visible |
| 4 | Review device timeline | Admin workstation | Device > Timeline | Advanced Hunting device timeline queries | Process and file activity is visible |
| 5 | Review logged-on users | Admin workstation | Device > Logged on users | Advanced Hunting | User context is known |
| 6 | Review file page | Admin workstation | Evidence > file > Open file page | Not applicable | File prevalence, hash, and verdict are known |
| 7 | Review process tree | Admin workstation | Alert > Process tree | Not applicable | Parent/child process chain is known |
| 8 | Check exposure/vulnerability if relevant | Admin workstation | Device > Security recommendations | Not applicable | Vulnerable software/misconfig is known |
| 9 | Decide containment | Admin workstation | Based on active risk | Not applicable | Isolation/scan/package action is justified |
| 10 | Document device action | Admin workstation | Incident record | Not applicable | Action and result are recorded |

### Phase 2: Device Response Actions

| Step | Task | Device | Command | PowerShell | Expected Result |
|---|---|---|---|---|---|
| 1 | Isolate device if active threat | Admin workstation | Device page > Isolate device | Not applicable | Isolation action is queued |
| 2 | Run antivirus scan | Admin workstation | Device page > Run antivirus scan | Not applicable | Scan action is queued |
| 3 | Collect investigation package | Admin workstation | Device page > Collect investigation package | Not applicable | Package collection is queued |
| 4 | Initiate live response if approved | Admin workstation | Device page > Live response | Not applicable | Live response session starts |
| 5 | Stop and quarantine file if available | Admin workstation | File action from evidence | Not applicable | File action is queued |
| 6 | Check action center | Admin workstation | Defender portal > Action center | Not applicable | Action completes or failure is visible |
| 7 | Release device isolation after remediation | Admin workstation | Device page > Release from isolation | Not applicable | Device reconnects normally |
| 8 | Verify endpoint health | Admin workstation | Device page > Health status | Not applicable | Sensor is healthy |
| 9 | Verify no recurrence | Admin workstation | Advanced Hunting and alert queue | Hunting query | No new matching activity |
| 10 | Close alert after evidence | Admin workstation | Alert/incident close workflow | Not applicable | Closure includes classification and notes |

---

## 06_Troubleshoot_Defender_Purview_Audit_DLP_And_Alert_Response_Issues_Purview_Audit_Workflow

### Phase 1: Audit Search Troubleshooting

| Step | Task | Device | Command | PowerShell | Expected Result |
|---|---|---|---|---|---|
| 1 | Confirm audit permissions | Admin workstation | Purview portal > Permissions | Not applicable | User has Audit Reader or appropriate role |
| 2 | Open audit search | Admin workstation | Purview portal > Audit | Not applicable | Audit search is accessible |
| 3 | Define time range | Admin workstation | Audit search date range | Not applicable | Search window covers event |
| 4 | Define user | Admin workstation | Audit search user filter | `Search-UnifiedAuditLog -StartDate <start> -EndDate <end> -UserIds <user@domain.com>` | User activity is returned or not |
| 5 | Define operation | Admin workstation | Audit search activity filter | `Search-UnifiedAuditLog -StartDate <start> -EndDate <end> -Operations "<operation>"` | Operation activity is returned or not |
| 6 | Search broadly first | Admin workstation | Audit search without narrow operation | `Search-UnifiedAuditLog -StartDate <start> -EndDate <end> -UserIds <user@domain.com> -ResultSize 5000` | Activity baseline is established |
| 7 | Check workload-specific logs | Admin workstation | Exchange, SharePoint, Teams, Entra logs | Workload-specific cmdlets or portals | Workload log confirms or supplements audit |
| 8 | Check audit delay | Admin workstation | Compare event time to current time | Not applicable | Missing result may be ingestion delay |
| 9 | Export results | Admin workstation | Audit search > Export | PowerShell output to CSV | Evidence is preserved |
| 10 | Escalate missing audit if required | Admin workstation | Support/security escalation | Not applicable | Missing audit evidence is tracked |

### Phase 2: Audit Scoping Examples

| Step | Task | Device | Command | PowerShell | Expected Result |
|---|---|---|---|---|---|
| 1 | Search mailbox changes | Admin workstation | Purview audit | `Search-UnifiedAuditLog -StartDate (Get-Date).AddDays(-7) -EndDate (Get-Date) -Operations "Set-Mailbox","Add-MailboxPermission","Remove-MailboxPermission"` | Mailbox admin changes are found |
| 2 | Search inbox rule changes | Admin workstation | Purview audit | `Search-UnifiedAuditLog -StartDate (Get-Date).AddDays(-7) -EndDate (Get-Date) -Operations "New-InboxRule","Set-InboxRule","Remove-InboxRule"` | Suspicious inbox rules are found |
| 3 | Search file access | Admin workstation | Purview audit | `Search-UnifiedAuditLog -StartDate (Get-Date).AddDays(-7) -EndDate (Get-Date) -UserIds <user@domain.com> -Operations "FileAccessed","FileDownloaded","FileDeleted"` | File activity is found |
| 4 | Search sharing activity | Admin workstation | Purview audit | `Search-UnifiedAuditLog -StartDate (Get-Date).AddDays(-7) -EndDate (Get-Date) -Operations "SharingSet","AddedToSecureLink","AnonymousLinkCreated"` | Sharing events are found |
| 5 | Search DLP events | Admin workstation | Purview audit / Activity Explorer | Search DLP operations and Activity Explorer | DLP activity is found |
| 6 | Search admin role changes | Admin workstation | Entra audit | `Get-MgAuditLogDirectoryAudit -Top 100 | Where-Object {$_.ActivityDisplayName -match "role|member|assignment"}` | Privileged changes are found |
| 7 | Search app consent changes | Admin workstation | Entra audit | `Get-MgAuditLogDirectoryAudit -Top 100 | Where-Object {$_.ActivityDisplayName -match "Consent|service principal|application"}` | App consent events are found |
| 8 | Export scoped evidence | Admin workstation | Portal export / PowerShell CSV | `Search-UnifiedAuditLog ... | Export-Csv ".\audit_results.csv" -NoTypeInformation` | Evidence is preserved |

---

## 06_Troubleshoot_Defender_Purview_Audit_DLP_And_Alert_Response_Issues_DLP_Workflow

### Phase 1: DLP Alert And Policy Investigation

| Step | Task | Device | Command | PowerShell | Expected Result |
|---|---|---|---|---|---|
| 1 | Open DLP alert | Admin workstation | Purview portal > DLP > Alerts | Not applicable | DLP alert details are visible |
| 2 | Record DLP metadata | Admin workstation | Alert details | Not applicable | Policy, rule, user, location, and matched content type are recorded |
| 3 | Check policy mode | Admin workstation | DLP > Policies > policy | Not applicable | Policy is On, Test, or Test with notifications |
| 4 | Check policy locations | Admin workstation | Policy > Locations | Not applicable | Exchange, SharePoint, OneDrive, Teams, Devices, or other locations are targeted |
| 5 | Check users/groups scope | Admin workstation | Policy > Admin units/users/groups | Not applicable | Affected user is included or excluded |
| 6 | Check rule conditions | Admin workstation | Policy > Rules | Not applicable | Sensitive info type, classifier, label, or sharing condition is understood |
| 7 | Check rule actions | Admin workstation | Policy > Actions | Not applicable | Block, restrict, notify, audit, or override action is known |
| 8 | Check exceptions | Admin workstation | Policy > Exceptions | Not applicable | Exception explains behavior or not |
| 9 | Check Activity Explorer | Admin workstation | Purview portal > Activity Explorer | Not applicable | Related DLP activity is visible |
| 10 | Check Content Explorer if authorized | Admin workstation | Purview portal > Content Explorer | Not applicable | Sensitive content location and match are reviewed by authorized role |

### Phase 2: DLP Does Not Trigger

| Step | Task | Device | Command | PowerShell | Expected Result |
|---|---|---|---|---|---|
| 1 | Confirm policy is enabled | Admin workstation | DLP policy status | `Get-DlpCompliancePolicy | Where-Object {$_.Name -eq "<policy-name>"}` | Policy is enabled or in intended test mode |
| 2 | Confirm rule is enabled | Admin workstation | DLP policy rule | `Get-DlpComplianceRule | Where-Object {$_.Policy -eq "<policy-name>"}` | Rule exists and is enabled |
| 3 | Confirm workload location | Admin workstation | Policy locations | Portal preferred | Target location is included |
| 4 | Confirm user is in scope | Admin workstation | Policy users/groups | Portal preferred | User is included and not excluded |
| 5 | Confirm content matches condition | Admin workstation | Test content against SIT/classifier/label | Portal preferred | Content should match rule |
| 6 | Confirm action threshold | Admin workstation | Rule condition count/confidence | Portal preferred | Test content meets threshold |
| 7 | Check endpoint DLP device onboarding | Admin workstation | Purview endpoint DLP settings and Defender device page | Not applicable | Device is onboarded and policy-capable |
| 8 | Check audit/activity delay | Admin workstation | Activity Explorer time range | Not applicable | Event may be delayed or filtered |
| 9 | Retest with controlled sample | Test workstation | Generate approved test event | Not applicable | DLP event triggers |
| 10 | Update policy if confirmed misconfigured | Admin workstation | Modify policy condition/scope/action | `Set-DlpComplianceRule` only after planning | DLP triggers as expected |

### Phase 3: DLP False Positive Or Overblocking

| Step | Task | Device | Command | PowerShell | Expected Result |
|---|---|---|---|---|---|
| 1 | Identify matched condition | Admin workstation | DLP alert details | Not applicable | Triggering condition is known |
| 2 | Identify matched sensitive info type | Admin workstation | Alert details / Content Explorer | Not applicable | SIT/classifier/label match is known |
| 3 | Review confidence and instance count | Admin workstation | Rule condition | Not applicable | Match threshold is understood |
| 4 | Review business justification | User/owner | User report and file/message context | Not applicable | Legitimate business case is known |
| 5 | Check exceptions | Admin workstation | DLP rule exceptions | Not applicable | Existing exceptions are reviewed |
| 6 | Add narrow exception if approved | Admin workstation | DLP policy > rule exception | `Set-DlpComplianceRule` only after planning | Legitimate workflow is allowed |
| 7 | Adjust threshold if overbroad | Admin workstation | DLP rule condition | `Set-DlpComplianceRule` only after planning | False positives reduce |
| 8 | Keep audit/notification if blocking removed | Admin workstation | DLP rule actions | Not applicable | Visibility remains |
| 9 | Retest false positive scenario | Test workstation | Repeat controlled scenario | Not applicable | Legitimate activity no longer blocked |
| 10 | Retest true positive scenario | Test workstation | Repeat controlled violation scenario | Not applicable | Real violation still triggers |

---

## 06_Troubleshoot_Defender_Purview_Audit_DLP_And_Alert_Response_Issues_Verification_Commands

### Microsoft Graph PowerShell

| Purpose | PowerShell |
|---|---|
| Connect for identity, audit, and service health checks | `Connect-MgGraph -Scopes "User.Read.All","Directory.Read.All","AuditLog.Read.All","SecurityEvents.Read.All","SecurityAlert.Read.All","ServiceHealth.Read.All","ServiceMessage.Read.All"` |
| Confirm tenant | `Get-MgOrganization | Select-Object DisplayName,Id,VerifiedDomains` |
| Check user | `Get-MgUser -UserId <user@domain.com> -Property Id,UserPrincipalName,AccountEnabled,UserType` |
| Check sign-ins | `Get-MgAuditLogSignIn -Filter "userPrincipalName eq '<user@domain.com>'" -Top 20` |
| Revoke user sessions | `Revoke-MgUserSignInSession -UserId <user@domain.com>` |
| Disable user if approved | `Update-MgUser -UserId <user@domain.com> -AccountEnabled:$false` |
| Check directory audit | `Get-MgAuditLogDirectoryAudit -Top 100` |
| Check service health | `Get-MgServiceAnnouncementIssue -Top 20` |
| Check message center | `Get-MgServiceAnnouncementMessage -Top 20` |
| Disconnect Graph | `Disconnect-MgGraph` |

### Exchange Online PowerShell

| Purpose | PowerShell |
|---|---|
| Connect to Exchange Online | `Connect-ExchangeOnline` |
| Trace recipient messages | `Get-MessageTrace -RecipientAddress <recipient@domain.com> -StartDate (Get-Date).AddDays(-2) -EndDate (Get-Date)` |
| Trace sender messages | `Get-MessageTrace -SenderAddress <sender@domain.com> -StartDate (Get-Date).AddDays(-2) -EndDate (Get-Date)` |
| Get trace details | `Get-MessageTraceDetail -MessageTraceId <trace-id> -RecipientAddress <recipient@domain.com>` |
| Check inbox rules | `Get-InboxRule -Mailbox <user@domain.com>` |
| Check forwarding | `Get-EXOMailbox <user@domain.com> -Properties ForwardingAddress,ForwardingSmtpAddress,DeliverToMailboxAndForward | Select-Object ForwardingAddress,ForwardingSmtpAddress,DeliverToMailboxAndForward` |
| Search mailbox audit/admin activity | `Search-UnifiedAuditLog -StartDate (Get-Date).AddDays(-7) -EndDate (Get-Date) -UserIds <user@domain.com>` |
| Search inbox rule changes | `Search-UnifiedAuditLog -StartDate (Get-Date).AddDays(-7) -EndDate (Get-Date) -Operations "New-InboxRule","Set-InboxRule","Remove-InboxRule"` |
| Search mailbox permission changes | `Search-UnifiedAuditLog -StartDate (Get-Date).AddDays(-7) -EndDate (Get-Date) -Operations "Add-MailboxPermission","Remove-MailboxPermission","Add-RecipientPermission","Remove-RecipientPermission"` |
| Disconnect Exchange Online | `Disconnect-ExchangeOnline` |

### Purview Compliance PowerShell

| Purpose | PowerShell |
|---|---|
| Connect to compliance PowerShell | `Connect-IPPSSession` |
| List DLP policies | `Get-DlpCompliancePolicy` |
| List DLP rules | `Get-DlpComplianceRule` |
| Check specific DLP policy | `Get-DlpCompliancePolicy -Identity "<policy-name>" | Format-List` |
| Check specific DLP rule | `Get-DlpComplianceRule -Identity "<rule-name>" | Format-List` |
| Search audit log | `Search-UnifiedAuditLog -StartDate <start-date> -EndDate <end-date> -ResultSize 5000` |
| Export audit log | `Search-UnifiedAuditLog -StartDate <start-date> -EndDate <end-date> -ResultSize 5000 | Export-Csv ".\audit_export.csv" -NoTypeInformation` |

### Advanced Hunting KQL

| Purpose | KQL |
|---|---|
| Email events for recipient | `EmailEvents | where Timestamp > ago(7d) | where RecipientEmailAddress =~ "<user@domain.com>" | sort by Timestamp desc` |
| Email events for sender | `EmailEvents | where Timestamp > ago(7d) | where SenderFromAddress =~ "<sender@domain.com>" or SenderMailFromAddress =~ "<sender@domain.com>" | sort by Timestamp desc` |
| Email by subject keyword | `EmailEvents | where Timestamp > ago(7d) | where Subject has "<subject-keyword>" | project Timestamp, NetworkMessageId, SenderFromAddress, RecipientEmailAddress, Subject, DeliveryAction, ThreatTypes, DetectionMethods` |
| Email URL evidence | `EmailUrlInfo | where Timestamp > ago(7d) | where Url has "<domain-or-url>" | project Timestamp, NetworkMessageId, Url, UrlDomain` |
| Email attachment evidence | `EmailAttachmentInfo | where Timestamp > ago(7d) | where FileName has "<filename>" or SHA256 =~ "<sha256>" | project Timestamp, NetworkMessageId, FileName, SHA256, RecipientEmailAddress` |
| Device alerts | `AlertInfo | where Timestamp > ago(7d) | join AlertEvidence on AlertId | where DeviceName =~ "<device-name>" | project Timestamp, AlertId, Title, Severity, DeviceName, AccountName, EntityType` |
| Device process events | `DeviceProcessEvents | where Timestamp > ago(7d) | where DeviceName =~ "<device-name>" | sort by Timestamp desc | project Timestamp, DeviceName, AccountName, FileName, ProcessCommandLine, InitiatingProcessFileName, InitiatingProcessCommandLine` |
| Device file hash scope | `DeviceFileEvents | where Timestamp > ago(30d) | where SHA256 =~ "<sha256>" | summarize Devices=dcount(DeviceName), Users=dcount(InitiatingProcessAccountName), FirstSeen=min(Timestamp), LastSeen=max(Timestamp) by SHA256, FileName` |
| Device network connections to indicator | `DeviceNetworkEvents | where Timestamp > ago(7d) | where RemoteUrl has "<domain>" or RemoteIP == "<ip-address>" | project Timestamp, DeviceName, InitiatingProcessAccountName, InitiatingProcessFileName, RemoteUrl, RemoteIP, RemotePort` |
| User risky activity across tables | `CloudAppEvents | where Timestamp > ago(7d) | where AccountId =~ "<user@domain.com>" or AccountDisplayName has "<user>" | sort by Timestamp desc` |
| OAuth app activity | `CloudAppEvents | where Timestamp > ago(30d) | where ActionType has "Consent" or ActivityType has "Consent" | sort by Timestamp desc` |
| DLP-related file activity proxy | `CloudAppEvents | where Timestamp > ago(7d) | where AccountId =~ "<user@domain.com>" | where ActivityType has_any ("FileDownloaded","FileUploaded","FileDeleted","FileShared") | sort by Timestamp desc` |
| Alert evidence by alert ID | `AlertEvidence | where AlertId =~ "<alert-id>" | project Timestamp, AlertId, EntityType, EvidenceRole, AccountName, DeviceName, FileName, FolderPath, SHA256, RemoteUrl, RemoteIP` |
| Incident alert list | `AlertInfo | where Timestamp > ago(30d) | where AlertId in ("<alert-id-1>","<alert-id-2>") | project Timestamp, AlertId, Title, Category, Severity, ServiceSource, DetectionSource` |

---

## 06_Troubleshoot_Defender_Purview_Audit_DLP_And_Alert_Response_Issues_Known_Good_Baselines

### Defender Incident Baseline

| Setting | Target State |
|---|---|
| Incident owner | Assigned during triage |
| Severity | Reflects real risk |
| Classification | True positive, benign positive, or false positive before closure |
| Determination | Selected before closure |
| Evidence | User, device, file, mailbox, URL, IP, and app evidence reviewed |
| Timeline | Earliest and latest activity identified |
| Scope | Affected users/devices/messages/files documented |
| Response action | Only performed when justified |
| Closure notes | Include evidence, actions, and reason |
| Related incidents | Linked or merged where appropriate |

### Defender For Office 365 Baseline

| Setting | Target State |
|---|---|
| Explorer search | Message evidence scoped by sender, recipient, subject, or NetworkMessageId |
| Quarantine review | Release/delete decision documented |
| Submissions | Suspicious messages/files/URLs submitted when verdict is uncertain |
| Restricted entities | Users unblocked only after compromise check |
| Message trace | Used to confirm delivery path |
| Headers | Reviewed for SPF, DKIM, DMARC, authentication, and route |
| Remediation | Malicious messages removed only after approval |
| User communication | Impacted recipients notified if required |

### Defender For Endpoint Baseline

| Setting | Target State |
|---|---|
| Device onboarded | Sensor healthy |
| Device timeline | Reviewed before action |
| File/process evidence | Hash, path, parent process, and user captured |
| Isolation | Used only for active risk |
| Investigation package | Collected when forensic evidence needed |
| AV scan | Run when malware suspected |
| Action center | Response action status verified |
| Release from isolation | Done only after remediation |
| Vulnerability follow-up | Security recommendation reviewed |

### Purview Audit Baseline

| Setting | Target State |
|---|---|
| Role access | Audit Reader or equivalent assigned |
| Time range | Covers event time plus buffer |
| Search filters | Start broad, then narrow |
| Workload-specific validation | Used when unified audit is delayed or incomplete |
| Export | Evidence exported for incident record |
| Retention awareness | Search window within audit retention |
| Missing audit | Documented and escalated if required |

### DLP Baseline

| Setting | Target State |
|---|---|
| Policy mode | On only after test validation |
| Locations | Target workloads intentionally included |
| Users/groups | Scope documented |
| Conditions | Sensitive info, label, classifier, or sharing condition documented |
| Exceptions | Narrow and justified |
| Actions | Block/restrict/notify/audit behavior documented |
| Alerting | Severity and alert routing configured |
| Activity Explorer | Used for event context |
| Content Explorer | Restricted to authorized investigators |
| Endpoint DLP | Device onboarding and policy targeting confirmed |

---

## 06_Troubleshoot_Defender_Purview_Audit_DLP_And_Alert_Response_Issues_Rollback

### Rollback Principles

- Preserve evidence before remediation.
- Do not delete or purge messages/files unless approved by security/compliance process.
- Do not release quarantined content unless verdict and business justification are documented.
- Do not suppress alerts without a replacement detection or written reason.
- Do not disable DLP policies tenant-wide to fix one false positive.
- Prefer narrow exceptions over broad policy weakening.
- Remove temporary blocks, indicators, exceptions, and containment actions after resolution.
- Keep legal/compliance hold requirements in mind before data deletion.

### Rollback Table

| Change Made | Rollback Action | Verification |
|---|---|---|
| Disabled user account | Re-enable account after remediation approval | User sign-in works and risk is cleared |
| Revoked sessions | No rollback needed; user signs in again | User can authenticate after reset/remediation |
| Reset password | User completes secure password reset | Sign-in succeeds |
| Isolated device | Release from isolation after remediation | Device connectivity restored |
| Blocked URL/domain/IP/file indicator | Remove indicator if false positive | Indicator no longer blocks legitimate activity |
| Released quarantined message | Recall/delete if release was incorrect and possible | Message state and trace reviewed |
| Deleted/remediated message | Restore only if recoverable and approved | Message recovery path validated |
| Added DLP exception | Remove exception after business process fixed | DLP rule catches expected violations |
| Disabled DLP rule | Re-enable rule after correction | DLP alerting resumes |
| Changed DLP policy mode to test | Restore enforce mode after validation | Policy actions apply |
| Suppressed alert | Remove suppression or alert processing rule | Alert fires for future matching events |
| Closed incident incorrectly | Reopen incident or create linked incident | Investigation continues with corrected state |
| Removed OAuth app consent | Re-grant only if app is approved | App access restored through approved consent |
| Unblocked restricted user | Re-restrict if compromise still active | User send behavior monitored |

### Pre-Change Capture Commands

| Purpose | PowerShell / KQL |
|---|---|
| Export user state | `Get-MgUser -UserId <user@domain.com> -Property * | ConvertTo-Json -Depth 10 | Out-File ".\user_before.json"` |
| Export sign-ins | `Get-MgAuditLogSignIn -Filter "userPrincipalName eq '<user@domain.com>'" -Top 50 | ConvertTo-Json -Depth 10 | Out-File ".\signins_before.json"` |
| Export mailbox state | `Get-EXOMailbox <user@domain.com> -Properties * | ConvertTo-Json -Depth 10 | Out-File ".\mailbox_before.json"` |
| Export message trace | `Get-MessageTrace -RecipientAddress <user@domain.com> -StartDate (Get-Date).AddDays(-2) -EndDate (Get-Date) | Export-Csv ".\message_trace_before.csv" -NoTypeInformation` |
| Export inbox rules | `Get-InboxRule -Mailbox <user@domain.com> | Export-Csv ".\inbox_rules_before.csv" -NoTypeInformation` |
| Export audit search | `Search-UnifiedAuditLog -StartDate <start-date> -EndDate <end-date> -ResultSize 5000 | Export-Csv ".\audit_before.csv" -NoTypeInformation` |
| Capture DLP policy | `Get-DlpCompliancePolicy -Identity "<policy-name>" | Format-List | Out-File ".\dlp_policy_before.txt"` |
| Capture DLP rules | `Get-DlpComplianceRule | Where-Object {$_.Policy -eq "<policy-name>"} | Format-List | Out-File ".\dlp_rules_before.txt"` |
| Export hunting result | Run hunting query and use Export in Defender portal |
| Capture incident evidence | Export incident/alert page screenshots or JSON where available |

---

## 06_Troubleshoot_Defender_Purview_Audit_DLP_And_Alert_Response_Issues_Failure_Checks

| Failure | Check | Fix |
|---|---|---|
| Alert missing | Detection source, policy status, time range, ingestion delay, license | Confirm signal source and wait/check policy |
| Alert noisy | Same entity, same detection, repeated behavior, benign tool | Tune rule only after evidence |
| Advanced Hunting returns no rows | Wrong table, time range, casing, licensing, device not onboarded | Broaden time/table and validate data source |
| Incident evidence incomplete | Data ingestion delay or related alert not correlated | Search entities manually |
| Device action fails | Device offline, unsupported OS, sensor unhealthy | Check device health and retry when online |
| Quarantine release unavailable | Missing role or malware verdict policy | Use correct role or escalate |
| Restricted user returns after unblock | Compromise still active or outbound spam continues | Reset password, revoke sessions, remove rules, investigate device |
| Audit search missing data | Wrong operation, workload delay, retention expired, missing role | Broaden search and check workload logs |
| DLP does not trigger | Policy in test, wrong location, user excluded, content threshold not met | Fix scope/mode/condition |
| DLP overblocks | Sensitive info type overmatch or condition too broad | Add narrow exception or adjust threshold |
| Endpoint DLP absent | Device not onboarded, unsupported device, policy not assigned | Fix endpoint onboarding and targeting |
| Activity Explorer empty | Wrong filters, delay, role issue | Broaden filters and check permissions |
| Content Explorer denied | Missing Content Explorer role | Assign appropriate role through Purview permissions |
| Policy changes not effective | Propagation delay or cached client state | Wait and retest |
| False positive closed without evidence | Missing classification notes | Reopen or create linked review |
| Message trace no result | Message never entered Exchange Online or wrong time range | Expand time and check mail path |
| Header authentication mismatch | SPF/DKIM/DMARC alignment issue | Validate domain authentication records |
| DLP exception too broad | Exception allows real data loss | Narrow exception to user/group/domain/location |
| Legal hold conflict with deletion | Content cannot be purged due to retention/hold | Escalate to compliance/legal |
| Service health incident active | Platform issue, not tenant configuration | Communicate and monitor incident |

---

## 06_Troubleshoot_Defender_Purview_Audit_DLP_And_Alert_Response_Issues_Escalation_Handoff

### Escalate To Security Operations When

- Alert is high severity or active compromise is suspected
- Multiple users, devices, or mailboxes are affected
- Malware, phishing campaign, credential theft, or lateral movement is suspected
- Device isolation, file quarantine, or live response is needed
- OAuth app abuse or suspicious consent is detected
- Incident involves privileged account activity

### Escalate To Messaging / Exchange Team When

- Phishing or malware message remediation is needed
- Quarantine release/delete requires mail admin decision
- Restricted entity or outbound spam issue affects business mail
- Mail flow rules, connectors, transport behavior, or message trace is involved
- Inbox rules, forwarding, or mailbox permissions look suspicious

### Escalate To Compliance / Legal When

- DLP incident involves regulated data
- Content Explorer review is required
- Data deletion conflicts with hold or retention
- eDiscovery case should be created
- External sharing or data exfiltration is suspected
- User activity needs formal investigation

### Escalate To Endpoint Team When

- Defender for Endpoint sensor is unhealthy
- Device isolation or live response is required
- Malware remediation requires hands-on device support
- Device needs reimage, patching, or vulnerability remediation
- Endpoint DLP is not applying to expected devices

### Escalate To Microsoft Support When

- Defender portal data is missing or inconsistent
- Advanced Hunting tables are delayed beyond expected windows
- Audit search fails despite correct permissions and time range
- DLP policy behavior contradicts configuration
- Response actions fail without clear device or permission cause
- Incident correlation appears broken or incomplete

### Handoff Data To Include

| Field | Value |
|---|---|
| Incident ID | `<incident-id>` |
| Alert ID | `<alert-id>` |
| Tenant ID | `<tenant-id>` |
| Workload | `<workload>` |
| Severity | `<severity>` |
| Classification | `<classification>` |
| Determination | `<determination>` |
| Affected user | `<user@domain.com>` |
| Affected mailbox | `<mailbox@domain.com>` |
| Affected device | `<device-name>` |
| Affected file/hash | `<file-or-sha256>` |
| Affected URL/domain/IP | `<indicator>` |
| DLP policy/rule | `<policy-rule>` |
| Sensitive info type | `<sit>` |
| First event time | `<timestamp>` |
| Last event time | `<timestamp>` |
| KQL used | `<query-name-or-summary>` |
| Audit search used | `<search-criteria>` |
| Response actions taken | `<actions>` |
| Current blocker | `<blocker>` |
| Business impact | `<impact>` |

---

## 06_Troubleshoot_Defender_Purview_Audit_DLP_And_Alert_Response_Issues_Incident_Record_Template

| Field | Entry |
|---|---|
| Date opened | `<yyyy-mm-dd>` |
| Opened by | `<name>` |
| Incident ID | `<incident-id>` |
| Alert ID | `<alert-id>` |
| Workload | `<Defender / Purview / DLP / Audit>` |
| Severity | `<severity>` |
| Status | `<status>` |
| Affected users | `<users>` |
| Affected devices | `<devices>` |
| Affected mailboxes | `<mailboxes>` |
| Affected files/sites | `<files-sites>` |
| Affected indicators | `<urls-domains-ips-hashes>` |
| Symptom | `<symptom>` |
| Root cause category | `<Threat / False positive / Policy misconfiguration / Audit gap / DLP match / Service issue / User behavior>` |
| Root cause | `<confirmed-root-cause>` |
| Evidence | `<alerts, audit logs, hunting output, message trace, screenshots>` |
| Containment actions | `<actions>` |
| Remediation actions | `<actions>` |
| Rollback actions | `<rollback>` |
| Classification | `<true-positive-benign-positive-false-positive>` |
| Determination | `<determination>` |
| Compliance impact | `<impact>` |
| Business impact | `<impact>` |
| Verification performed | `<verification>` |
| Preventive action | `<preventive-action>` |
| Closed by | `<name>` |
| Date closed | `<yyyy-mm-dd>` |

---

## 06_Troubleshoot_Defender_Purview_Audit_DLP_And_Alert_Response_Issues_Preventive_Controls

| Control | Target State |
|---|---|
| Incident ownership | Defender incidents assigned and worked through closure |
| Alert classification | Closed alerts include classification and determination |
| Hunting library | Common KQL queries saved for email, endpoint, identity, cloud app, and DLP investigations |
| Evidence retention | Audit, hunting, trace, and portal evidence exported for serious incidents |
| Quarantine process | Release/delete decisions documented |
| Restricted entity process | Users unblocked only after compromise checks |
| Endpoint response process | Isolation, scan, package collection, and release steps documented |
| DLP test process | Policies tested before enforcement |
| DLP exception governance | Exceptions are narrow, justified, time-bound where possible |
| Audit role assignments | Audit Reader and compliance roles assigned to approved investigators |
| Content Explorer governance | Access restricted and reviewed |
| Service health review | Platform incidents checked before policy changes |
| Alert tuning process | Noise tuned with evidence, not by broad suppression |
| Incident review | Root cause, fix, rollback, and prevention documented |

---

## 06_Troubleshoot_Defender_Purview_Audit_DLP_And_Alert_Response_Issues_Related_Labs

| Lab | Purpose |
|---|---|
| Triage Defender XDR incident | Practice incident timeline, evidence, and classification |
| Investigate phishing with Defender Explorer | Scope malicious email, URLs, attachments, and recipients |
| Review quarantine and submissions | Practice release/delete/submit workflows |
| Investigate restricted user | Validate outbound spam and account compromise checks |
| Investigate Defender for Endpoint alert | Review device timeline, process tree, and response actions |
| Run Advanced Hunting queries | Scope users, devices, messages, files, and indicators |
| Search Purview audit logs | Validate user/admin activity trail |
| Troubleshoot missing audit events | Check roles, time range, workload, and ingestion delay |
| Investigate DLP alert | Review policy, rule, sensitive info match, and activity |
| Troubleshoot DLP false positive | Adjust threshold or exception safely |
| Troubleshoot endpoint DLP | Validate device onboarding and policy targeting |
| Create security incident review record | Document evidence, containment, remediation, rollback, and prevention |
