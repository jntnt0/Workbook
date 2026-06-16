# 07_Create_Cloud_Admin_Incident_Review_Change_Log_And_Rollback_Records

## 07_Create_Cloud_Admin_Incident_Review_Change_Log_And_Rollback_Records_Index

### Purpose

This workbook provides a structured process for creating cloud administration incident reviews, change logs, rollback records, evidence indexes, approval notes, validation records, and post-incident follow-up documentation.

### Scope

Use this workbook after or during any cloud administration event involving:

- Microsoft 365 admin issue
- Microsoft Entra identity issue
- Azure deployment failure
- Azure networking, storage, compute, monitoring, or backup issue
- Defender incident or alert response
- Purview audit or DLP investigation
- Exchange Online mail flow or mailbox issue
- Conditional Access, MFA, PIM, RBAC, or admin role issue
- Service health incident affecting users
- Emergency access / break-glass usage
- Production change with rollback requirement
- Failed change, partial deployment, or service degradation
- Security, compliance, licensing, or data protection impact

### Assumptions

- A ticket, incident, change request, or investigation record exists.
- Evidence is collected before major remediation when possible.
- All temporary access, policy changes, exceptions, exclusions, and bypasses are tracked.
- Rollback is documented even if rollback is not executed.
- Final review includes root cause, fix, validation, business impact, security impact, and preventive action.
- Records are written so another admin can understand what happened without needing the original chat, call, or portal session.

### Required Roles

| Task Area | Minimum Role |
|---|---|
| Create incident record | Helpdesk, service desk, admin, or incident owner |
| Approve production change | Change manager, service owner, platform owner, or security owner |
| Capture Azure evidence | Reader at target scope |
| Capture Microsoft 365 evidence | Global Reader, Reports Reader, workload reader |
| Capture security evidence | Security Reader, Security Operator |
| Capture compliance evidence | Audit Reader, Compliance Administrator |
| Capture mailbox evidence | Exchange Administrator or Global Reader with Exchange access |
| Capture identity evidence | Global Reader, User Administrator, Security Reader |
| Capture backup/restore evidence | Backup Reader, Backup Operator, Backup Contributor |
| Approve rollback | Service owner, change approver, incident commander, or platform owner |

---

## 07_Create_Cloud_Admin_Incident_Review_Change_Log_And_Rollback_Records_Mental_Model

Cloud admin records should answer seven questions:

1. What happened?
2. Who or what was affected?
3. When did it start and end?
4. What evidence proves the root cause?
5. What changed?
6. How was it fixed or contained?
7. How would we roll it back or prevent it next time?

A good incident/change/rollback record is not a journal of random troubleshooting. It is a clean timeline that separates observation, evidence, decision, action, validation, and follow-up.

Record types:

1. Incident record: documents unplanned impact or investigation.
2. Change log: documents deliberate admin actions.
3. Rollback record: documents how to reverse actions safely.
4. Evidence index: tracks screenshots, command output, logs, IDs, and exports.
5. Decision log: explains why an action was chosen.
6. Validation record: proves the issue is resolved.
7. Post-incident review: captures root cause and prevention.

Traditional documentation order:

1. Open record.
2. Define scope and impact.
3. Capture timeline.
4. Capture evidence.
5. Record decisions.
6. Record changes.
7. Record rollback.
8. Validate result.
9. Capture root cause.
10. Record preventive actions.
11. Close with owner and date.

---

## 07_Create_Cloud_Admin_Incident_Review_Change_Log_And_Rollback_Records_Record_Types

| Record Type | Purpose | Created When |
|---|---|---|
| Incident record | Tracks unplanned service disruption, security event, access issue, or operational failure | A user, admin, service, workload, or control fails unexpectedly |
| Change log | Tracks intentional admin modifications | Any production config, policy, RBAC, license, DNS, mailbox, or deployment change is made |
| Rollback record | Defines how to reverse a change | Before or during any risky change |
| Evidence index | Tracks proof used for decision-making | During investigation and before remediation |
| Decision log | Records why a specific action was chosen | During incident bridge or approval |
| Validation record | Proves fix worked | After remediation or rollback |
| Post-incident review | Captures root cause and prevention | After closure |
| Temporary exception register | Tracks temporary bypasses and expiration | When CA/DLP/policy/RBAC/security exceptions are used |
| Approval record | Tracks who approved change, rollback, release, deletion, or exception | Before high-risk action |
| Handoff record | Transfers ownership to another team | When issue leaves current admin scope |

---

## 07_Create_Cloud_Admin_Incident_Review_Change_Log_And_Rollback_Records_Symptom_Map

| Situation | Required Record | Why |
|---|---|---|
| Admin changes Conditional Access to restore access | Incident record, change log, rollback record | CA changes can lock out users |
| Break-glass account used | Incident record, evidence index, decision log | Emergency access must be justified |
| Azure deployment failed due to RBAC or policy | Incident record, evidence index, change log | Captures exact failure and fix |
| DNS/MX/SPF/DKIM/DMARC changed | Change log, rollback record, validation record | DNS changes require rollback and proof |
| Mailbox permissions added temporarily | Change log, rollback record, validation record | Delegation must be removed later if temporary |
| Defender alert remediated | Incident record, evidence index, response log | Security actions need proof |
| Quarantine message released | Decision log, change log, evidence index | Release must be justified |
| DLP exception added | Change log, approval record, rollback record | Exception can create data-loss risk |
| Azure backup restore performed | Incident record, restore record, validation record | Restore must be proven usable |
| User license changed to restore access | Change log, validation record | Licensing affects cost and service access |
| Service health incident confirmed | Incident record, communication log | Local changes may not be needed |
| Temporary RBAC elevation granted | Change log, approval record, rollback record | Privilege must be removed |
| Failed change rolled back | Incident record, rollback record, validation record | Confirms return to known-good state |

---

## 07_Create_Cloud_Admin_Incident_Review_Change_Log_And_Rollback_Records_Evidence_Collection

| Evidence | Source | Record It Supports |
|---|---|---|
| Ticket ID | ITSM / issue tracker | Incident and change traceability |
| Incident ID | Defender / service desk / monitoring platform | Incident record |
| Change ID | Change management system | Change log and approval |
| Tenant ID | Entra / M365 / Azure | Scope validation |
| Subscription ID | Azure portal / CLI | Azure scope validation |
| Resource ID | Azure resource overview / CLI | Azure evidence |
| User UPN | Microsoft 365 / Entra | Identity and mailbox evidence |
| Mailbox address | Exchange Online | Mailbox and mail flow evidence |
| Group object ID | Entra / Graph | Group and license evidence |
| Policy ID | CA / DLP / Azure Policy | Policy evidence |
| Role assignment ID | Azure IAM / Entra roles | RBAC/PIM evidence |
| Alert ID | Defender portal | Security evidence |
| Message trace ID | Exchange Online | Mail flow evidence |
| Audit log event | Purview / Entra | Change and user activity evidence |
| Activity log event | Azure Monitor | Azure change/failure evidence |
| Backup job ID | Recovery Services Vault | Backup/restore evidence |
| Correlation ID | Azure / M365 error | Escalation evidence |
| Screenshot | Portal | Visual state evidence |
| Command output | CLI / PowerShell | Reproducible evidence |
| KQL output | Defender / Log Analytics | Investigation evidence |
| Approval note | Ticket/email/chat | Decision evidence |
| Validation result | Test output / user confirmation | Closure evidence |

---

## 07_Create_Cloud_Admin_Incident_Review_Change_Log_And_Rollback_Records_Triage_Order

1. Create or identify the incident/change ticket.
2. Record the scope: tenant, subscription, workload, resource, user, mailbox, group, policy, or device.
3. Record the impact: who is affected, what is broken, and business severity.
4. Capture the first known failure time and current status.
5. Capture evidence before making changes.
6. Identify the owner and approver.
7. Document the remediation plan.
8. Document the rollback plan before executing risky change.
9. Execute change in controlled steps.
10. Record each action, command, portal path, actor, and timestamp.
11. Validate the outcome with explicit tests.
12. Remove temporary exceptions or schedule cleanup.
13. Record root cause and preventive action.
14. Close the record with evidence attached or indexed.

---

## 07_Create_Cloud_Admin_Incident_Review_Change_Log_And_Rollback_Records_Planning_Table

| Item | Value |
|---|---|
| Record type | `<Incident / Change / Rollback / PIR / Evidence index>` |
| Ticket ID | `<ticket-id>` |
| Incident ID | `<incident-id>` |
| Change ID | `<change-id>` |
| Title | `<short-title>` |
| Opened date/time | `<yyyy-mm-dd hh:mm timezone>` |
| Opened by | `<name>` |
| Incident owner | `<name/team>` |
| Technical owner | `<name/team>` |
| Approver | `<name/team>` |
| Tenant name | `<tenant-name>` |
| Tenant ID | `<tenant-id>` |
| Subscription name | `<subscription-name>` |
| Subscription ID | `<subscription-id>` |
| Resource group | `<resource-group>` |
| Workload | `<Azure / Entra / M365 / Exchange / Defender / Purview / Intune / Teams / SharePoint / OneDrive>` |
| Affected user | `<user@domain.com>` |
| Affected mailbox | `<mailbox@domain.com>` |
| Affected group | `<group-name>` |
| Affected resource | `<resource-name-or-id>` |
| Affected policy | `<policy-name>` |
| Severity | `<Low / Medium / High / Critical>` |
| Business impact | `<impact>` |
| Security impact | `<impact>` |
| Compliance impact | `<impact>` |
| Data loss risk | `<yes-no>` |
| Customer/user impact | `<impact>` |
| Start time | `<yyyy-mm-dd hh:mm timezone>` |
| Detection time | `<yyyy-mm-dd hh:mm timezone>` |
| Mitigation time | `<yyyy-mm-dd hh:mm timezone>` |
| Resolution time | `<yyyy-mm-dd hh:mm timezone>` |
| Current status | `<Open / Monitoring / Mitigated / Resolved / Closed>` |

---

## 07_Create_Cloud_Admin_Incident_Review_Change_Log_And_Rollback_Records_Configuration_Checklist

| Step | Task | Device | Command | PowerShell | Expected Result |
|---|---|---|---|---|---|
| 1 | Create incident or change record | Admin workstation | Open ITSM, issue tracker, or workbook record | Not applicable | Ticket or record ID exists |
| 2 | Record scope | Admin workstation | Capture tenant, subscription, workload, resource, user, mailbox, group, or policy | Not applicable | Scope is clearly defined |
| 3 | Record impact | Admin workstation | Document affected users, services, business process, and severity | Not applicable | Impact statement is complete |
| 4 | Record timeline start | Admin workstation | Add first known failure time and detection time | Not applicable | Timeline has starting point |
| 5 | Capture service health | Admin workstation | Microsoft 365 admin center > Service health; Azure portal > Service Health | `Get-MgServiceAnnouncementIssue -Top 20` | Platform incident is ruled in or out |
| 6 | Capture identity evidence if involved | Admin workstation | Entra admin center > Users / Sign-in logs / Audit logs | `Get-MgUser -UserId <user@domain.com>; Get-MgAuditLogSignIn -Filter "userPrincipalName eq '<user@domain.com>'" -Top 20` | User and sign-in evidence is captured |
| 7 | Capture Azure evidence if involved | Admin workstation | Azure portal > Resource / Activity Log / Deployment / IAM | `Get-AzResource -ResourceId <resource-id>; Get-AzActivityLog -StartTime (Get-Date).AddHours(-24)` | Azure resource and activity evidence is captured |
| 8 | Capture M365 mailbox evidence if involved | Admin workstation | Exchange admin center > Recipients / Message trace | `Get-EXOMailbox <mailbox@domain.com>; Get-MessageTrace -RecipientAddress <mailbox@domain.com> -StartDate (Get-Date).AddHours(-24) -EndDate (Get-Date)` | Mailbox and mail flow evidence is captured |
| 9 | Capture Defender evidence if involved | Admin workstation | Defender portal > Incident / Alert / Evidence / Timeline | Use Advanced Hunting queries as needed | Incident and alert evidence is captured |
| 10 | Capture Purview/DLP evidence if involved | Admin workstation | Purview portal > Audit / DLP alerts / Activity Explorer | `Search-UnifiedAuditLog -StartDate <start> -EndDate <end> -ResultSize 5000` | Audit and DLP evidence is captured |
| 11 | Capture backup evidence if involved | Admin workstation | Recovery Services Vault > Backup jobs / Recovery points | `Get-AzRecoveryServicesBackupJob -VaultId <vault-id> -From (Get-Date).AddDays(-7) -To (Get-Date)` | Backup job or restore evidence is captured |
| 12 | Record suspected root cause | Admin workstation | Add working theory with confidence level | Not applicable | Hypothesis is documented separately from confirmed root cause |
| 13 | Record decision and approval | Admin workstation | Add approver, approval time, and approved action | Not applicable | Approval is traceable |
| 14 | Create rollback plan before change | Admin workstation | Fill rollback table for each planned action | Not applicable | Rollback steps exist before execution |
| 15 | Execute remediation step | Admin workstation | Run portal action, CLI, PowerShell, or pipeline step | Depends on remediation | Change is made |
| 16 | Record exact change | Admin workstation | Add actor, timestamp, object, old value, new value, command, and result | Not applicable | Change log entry is complete |
| 17 | Validate immediately | Admin workstation | Repeat original failed test | Depends on issue | Symptom is resolved or next blocker identified |
| 18 | Record validation evidence | Admin workstation | Attach test output, screenshot, trace, query, or user confirmation | Not applicable | Validation proof exists |
| 19 | Remove or schedule cleanup for temporary changes | Admin workstation | Remove temporary access, exclusions, exceptions, rules, or licenses | Depends on cleanup | Temporary risk is removed or tracked |
| 20 | Confirm monitoring period | Admin workstation | Watch alerts/logs/service for agreed time window | Not applicable | Issue does not recur during monitoring |
| 21 | Record final root cause | Admin workstation | Replace hypothesis with confirmed root cause | Not applicable | Root cause is clear and evidence-backed |
| 22 | Record preventive action | Admin workstation | Add policy/process/automation/documentation improvement | Not applicable | Prevention is assigned |
| 23 | Assign follow-up owners | Admin workstation | Add owner and due date for each action item | Not applicable | Follow-up work is trackable |
| 24 | Close record | Admin workstation | Mark resolved/closed in system | Not applicable | Record has owner, evidence, validation, rollback, and prevention |

---

## 07_Create_Cloud_Admin_Incident_Review_Change_Log_And_Rollback_Records_Incident_Record_Template

### Incident Summary

| Field | Entry |
|---|---|
| Incident ID | `<incident-id>` |
| Ticket ID | `<ticket-id>` |
| Title | `<short-clear-title>` |
| Status | `<Open / Mitigated / Resolved / Closed>` |
| Severity | `<Low / Medium / High / Critical>` |
| Opened date/time | `<yyyy-mm-dd hh:mm timezone>` |
| Detected by | `<user / alert / monitoring / admin / service health>` |
| Incident owner | `<name/team>` |
| Technical owner | `<name/team>` |
| Workload | `<workload>` |
| Tenant ID | `<tenant-id>` |
| Subscription ID | `<subscription-id>` |
| Affected resources | `<resources>` |
| Affected users | `<users/count>` |
| Business impact | `<impact>` |
| Security impact | `<impact>` |
| Compliance impact | `<impact>` |
| Data impact | `<none / potential / confirmed>` |
| Customer impact | `<internal / external / none>` |

### Incident Description

| Field | Entry |
|---|---|
| What happened | `<plain-language-summary>` |
| What was expected | `<expected-state>` |
| What actually happened | `<observed-state>` |
| First known occurrence | `<yyyy-mm-dd hh:mm timezone>` |
| Detection time | `<yyyy-mm-dd hh:mm timezone>` |
| Mitigation time | `<yyyy-mm-dd hh:mm timezone>` |
| Resolution time | `<yyyy-mm-dd hh:mm timezone>` |
| Current state | `<current-state>` |

### Incident Classification

| Field | Entry |
|---|---|
| Category | `<Identity / Access / Deployment / Network / Storage / Compute / Mail / Security / Compliance / Backup / Service Health>` |
| Root cause status | `<Suspected / Confirmed / Unknown>` |
| Root cause | `<confirmed-root-cause>` |
| Trigger | `<change / alert / user action / policy / outage / capacity / expired credential / misconfiguration / other>` |
| Detection source | `<monitoring / user report / alert / audit / service health / pipeline>` |
| Preventability | `<Preventable / Not preventable / Unknown>` |
| Recurrence risk | `<Low / Medium / High>` |

---

## 07_Create_Cloud_Admin_Incident_Review_Change_Log_And_Rollback_Records_Timeline_Template

| Time | Timezone | Actor | Event Type | Details | Evidence ID |
|---|---|---|---|---|---|
| `<yyyy-mm-dd hh:mm>` | `<timezone>` | `<user/system/admin>` | Detection | `<what detected issue>` | `<EVID-001>` |
| `<yyyy-mm-dd hh:mm>` | `<timezone>` | `<admin>` | Evidence | `<evidence captured>` | `<EVID-002>` |
| `<yyyy-mm-dd hh:mm>` | `<timezone>` | `<admin>` | Decision | `<decision made>` | `<DEC-001>` |
| `<yyyy-mm-dd hh:mm>` | `<timezone>` | `<approver>` | Approval | `<approval granted>` | `<APP-001>` |
| `<yyyy-mm-dd hh:mm>` | `<timezone>` | `<admin>` | Change | `<change made>` | `<CHG-001>` |
| `<yyyy-mm-dd hh:mm>` | `<timezone>` | `<admin/user>` | Validation | `<test result>` | `<VAL-001>` |
| `<yyyy-mm-dd hh:mm>` | `<timezone>` | `<admin>` | Cleanup | `<temporary change removed>` | `<CHG-002>` |
| `<yyyy-mm-dd hh:mm>` | `<timezone>` | `<owner>` | Closure | `<incident closed>` | `<VAL-002>` |

---

## 07_Create_Cloud_Admin_Incident_Review_Change_Log_And_Rollback_Records_Evidence_Index_Template

| Evidence ID | Type | Source | Object | Time Captured | Captured By | Storage Location | Notes |
|---|---|---|---|---|---|---|---|
| `EVID-001` | Screenshot | `<portal>` | `<resource/user/policy>` | `<timestamp>` | `<name>` | `<link/path>` | `<notes>` |
| `EVID-002` | Command output | `<PowerShell/CLI>` | `<object>` | `<timestamp>` | `<name>` | `<link/path>` | `<notes>` |
| `EVID-003` | Log export | `<Audit/Activity/Defender/KQL>` | `<query>` | `<timestamp>` | `<name>` | `<link/path>` | `<notes>` |
| `EVID-004` | Error message | `<portal/app/client>` | `<operation>` | `<timestamp>` | `<name>` | `<link/path>` | `<notes>` |
| `EVID-005` | Message trace | `<Exchange Online>` | `<message>` | `<timestamp>` | `<name>` | `<link/path>` | `<notes>` |
| `EVID-006` | Alert export | `<Defender/Purview>` | `<alert-id>` | `<timestamp>` | `<name>` | `<link/path>` | `<notes>` |
| `EVID-007` | Approval | `<ticket/email/chat>` | `<change>` | `<timestamp>` | `<name>` | `<link/path>` | `<notes>` |
| `EVID-008` | Validation result | `<test/query/user-confirmation>` | `<result>` | `<timestamp>` | `<name>` | `<link/path>` | `<notes>` |

---

## 07_Create_Cloud_Admin_Incident_Review_Change_Log_And_Rollback_Records_Decision_Log_Template

| Decision ID | Time | Decision Maker | Decision | Options Considered | Reason | Risk Accepted | Approval Evidence |
|---|---|---|---|---|---|---|---|
| `DEC-001` | `<timestamp>` | `<name>` | `<decision>` | `<option-a / option-b / option-c>` | `<why>` | `<risk>` | `<APP-001>` |
| `DEC-002` | `<timestamp>` | `<name>` | `<decision>` | `<options>` | `<why>` | `<risk>` | `<APP-002>` |
| `DEC-003` | `<timestamp>` | `<name>` | `<decision>` | `<options>` | `<why>` | `<risk>` | `<APP-003>` |

---

## 07_Create_Cloud_Admin_Incident_Review_Change_Log_And_Rollback_Records_Approval_Record_Template

| Approval ID | Time | Approver | Role / Team | Approved Action | Scope | Expiration | Conditions | Evidence |
|---|---|---|---|---|---|---|---|---|
| `APP-001` | `<timestamp>` | `<name>` | `<role/team>` | `<action>` | `<scope>` | `<expiration>` | `<conditions>` | `<link/path>` |
| `APP-002` | `<timestamp>` | `<name>` | `<role/team>` | `<action>` | `<scope>` | `<expiration>` | `<conditions>` | `<link/path>` |
| `APP-003` | `<timestamp>` | `<name>` | `<role/team>` | `<action>` | `<scope>` | `<expiration>` | `<conditions>` | `<link/path>` |

---

## 07_Create_Cloud_Admin_Incident_Review_Change_Log_And_Rollback_Records_Change_Log_Template

| Change ID | Time | Actor | Tool | Object Type | Object Name / ID | Old Value | New Value | Command / Portal Path | Result | Evidence ID | Rollback ID |
|---|---|---|---|---|---|---|---|---|---|---|---|
| `CHG-001` | `<timestamp>` | `<admin>` | `<Portal / PowerShell / CLI / Graph / Pipeline>` | `<object-type>` | `<object-name-or-id>` | `<old-value>` | `<new-value>` | `<command-or-portal-path>` | `<success/failure>` | `<EVID-001>` | `<RB-001>` |
| `CHG-002` | `<timestamp>` | `<admin>` | `<tool>` | `<object-type>` | `<object-name-or-id>` | `<old-value>` | `<new-value>` | `<command-or-portal-path>` | `<result>` | `<EVID-002>` | `<RB-002>` |
| `CHG-003` | `<timestamp>` | `<admin>` | `<tool>` | `<object-type>` | `<object-name-or-id>` | `<old-value>` | `<new-value>` | `<command-or-portal-path>` | `<result>` | `<EVID-003>` | `<RB-003>` |

---

## 07_Create_Cloud_Admin_Incident_Review_Change_Log_And_Rollback_Records_Rollback_Record_Template

| Rollback ID | Related Change ID | Trigger Condition | Rollback Action | Tool | Command / Portal Path | Owner | Approval Needed | Validation | Status |
|---|---|---|---|---|---|---|---|---|---|
| `RB-001` | `CHG-001` | `<when-to-rollback>` | `<rollback-action>` | `<tool>` | `<command-or-path>` | `<owner>` | `<yes-no>` | `<validation-test>` | `<ready / executed / not-needed>` |
| `RB-002` | `CHG-002` | `<trigger>` | `<rollback-action>` | `<tool>` | `<command-or-path>` | `<owner>` | `<yes-no>` | `<validation-test>` | `<status>` |
| `RB-003` | `CHG-003` | `<trigger>` | `<rollback-action>` | `<tool>` | `<command-or-path>` | `<owner>` | `<yes-no>` | `<validation-test>` | `<status>` |

---

## 07_Create_Cloud_Admin_Incident_Review_Change_Log_And_Rollback_Records_Validation_Record_Template

| Validation ID | Time | Validator | Test | Tool | Expected Result | Actual Result | Pass / Fail | Evidence ID |
|---|---|---|---|---|---|---|---|---|
| `VAL-001` | `<timestamp>` | `<name>` | `<test>` | `<tool>` | `<expected>` | `<actual>` | `<pass/fail>` | `<EVID-001>` |
| `VAL-002` | `<timestamp>` | `<name>` | `<test>` | `<tool>` | `<expected>` | `<actual>` | `<pass/fail>` | `<EVID-002>` |
| `VAL-003` | `<timestamp>` | `<name>` | `<test>` | `<tool>` | `<expected>` | `<actual>` | `<pass/fail>` | `<EVID-003>` |

---

## 07_Create_Cloud_Admin_Incident_Review_Change_Log_And_Rollback_Records_Temporary_Exception_Register

| Exception ID | Type | Object | Scope | Reason | Approved By | Start Time | Expiration Time | Owner | Cleanup Action | Status |
|---|---|---|---|---|---|---|---|---|---|---|
| `EXC-001` | `<CA exclusion / DLP exception / RBAC elevation / License / Firewall allow / Quarantine release / Alert suppression>` | `<object>` | `<scope>` | `<reason>` | `<approver>` | `<timestamp>` | `<timestamp>` | `<owner>` | `<cleanup-action>` | `<active / removed / expired>` |
| `EXC-002` | `<type>` | `<object>` | `<scope>` | `<reason>` | `<approver>` | `<timestamp>` | `<timestamp>` | `<owner>` | `<cleanup-action>` | `<status>` |
| `EXC-003` | `<type>` | `<object>` | `<scope>` | `<reason>` | `<approver>` | `<timestamp>` | `<timestamp>` | `<owner>` | `<cleanup-action>` | `<status>` |

---

## 07_Create_Cloud_Admin_Incident_Review_Change_Log_And_Rollback_Records_Root_Cause_Analysis_Template

| Field | Entry |
|---|---|
| Confirmed root cause | `<root-cause>` |
| Root cause category | `<Misconfiguration / Expired credential / Policy conflict / RBAC gap / Quota / Service incident / User error / Process gap / Security event / Unknown>` |
| Triggering event | `<event>` |
| Detection method | `<alert / user report / monitoring / audit / service health / admin>` |
| Why detection worked or failed | `<explanation>` |
| Why prevention worked or failed | `<explanation>` |
| Contributing factors | `<factors>` |
| Systems involved | `<systems>` |
| People/teams involved | `<teams>` |
| Process gaps | `<gaps>` |
| Technical gaps | `<gaps>` |
| Security gaps | `<gaps>` |
| Compliance gaps | `<gaps>` |
| Recurrence likelihood | `<low / medium / high>` |
| Recurrence impact | `<low / medium / high>` |

---

## 07_Create_Cloud_Admin_Incident_Review_Change_Log_And_Rollback_Records_Five_Whys_Template

| Level | Question | Answer |
|---|---|---|
| Why 1 | Why did the user/service/resource fail? | `<answer>` |
| Why 2 | Why did that condition exist? | `<answer>` |
| Why 3 | Why was it not detected or prevented earlier? | `<answer>` |
| Why 4 | Why did the process/control allow it? | `<answer>` |
| Why 5 | Why is this the systemic root cause? | `<answer>` |

---

## 07_Create_Cloud_Admin_Incident_Review_Change_Log_And_Rollback_Records_Action_Items_Template

| Action ID | Action | Owner | Priority | Due Date | Status | Validation Method |
|---|---|---|---|---|---|---|
| `ACT-001` | `<preventive-action>` | `<owner>` | `<Low / Medium / High>` | `<yyyy-mm-dd>` | `<open / in-progress / complete>` | `<how-to-verify>` |
| `ACT-002` | `<action>` | `<owner>` | `<priority>` | `<date>` | `<status>` | `<validation>` |
| `ACT-003` | `<action>` | `<owner>` | `<priority>` | `<date>` | `<status>` | `<validation>` |

---

## 07_Create_Cloud_Admin_Incident_Review_Change_Log_And_Rollback_Records_Detailed_Workflow

### Phase 1: Open And Scope The Record

| Step | Task | Device | Command | PowerShell | Expected Result |
|---|---|---|---|---|---|
| 1 | Create or locate ticket | Admin workstation | ITSM / issue tracker / workbook | Not applicable | Ticket ID exists |
| 2 | Assign owner | Admin workstation | Set incident owner and technical owner | Not applicable | Accountability is clear |
| 3 | Define affected workload | Admin workstation | Select Azure, Entra, M365, Exchange, Defender, Purview, etc. | Not applicable | Workload scope is clear |
| 4 | Define affected objects | Admin workstation | Record users, mailboxes, groups, policies, resources, devices, or alerts | Not applicable | Object scope is clear |
| 5 | Define impact | Admin workstation | Record business, security, compliance, and user impact | Not applicable | Severity can be assigned |
| 6 | Define timeline start | Admin workstation | Record first known failure and detection time | Not applicable | Timeline begins |
| 7 | Define record type | Admin workstation | Incident, change, rollback, PIR, or evidence index | Not applicable | Correct template is used |

### Phase 2: Capture Evidence

| Step | Task | Device | Command | PowerShell | Expected Result |
|---|---|---|---|---|---|
| 1 | Capture portal screenshot | Admin workstation | Screenshot affected portal state | Not applicable | Screenshot evidence is preserved |
| 2 | Capture command output | Admin workstation | Run scoped CLI/PowerShell command | Depends on workload | Reproducible evidence is preserved |
| 3 | Capture log export | Admin workstation | Export Activity Log, audit, sign-in, message trace, KQL, or alert data | Depends on workload | Log evidence is preserved |
| 4 | Assign evidence ID | Admin workstation | Add entry to evidence index | Not applicable | Evidence is traceable |
| 5 | Store evidence | Admin workstation | Save to approved ticket, repo, case, or secure folder | Not applicable | Evidence has stable location |
| 6 | Record capture time | Admin workstation | Add timestamp and collector | Not applicable | Evidence chain is clear |
| 7 | Preserve pre-change state | Admin workstation | Export current config before editing | Depends on object | Rollback is possible |

### Phase 3: Decision And Approval

| Step | Task | Device | Command | PowerShell | Expected Result |
|---|---|---|---|---|---|
| 1 | Identify options | Admin workstation | List possible fixes or containment paths | Not applicable | Options are visible |
| 2 | Assess risk | Admin workstation | Compare security, availability, data, and rollback risk | Not applicable | Risk is understood |
| 3 | Choose action | Admin workstation | Record decision and reason | Not applicable | Decision log is complete |
| 4 | Get approval | Admin workstation | Record approver and approval evidence | Not applicable | Approval is traceable |
| 5 | Define rollback trigger | Admin workstation | Record when rollback will happen | Not applicable | Rollback criteria are clear |
| 6 | Define validation test | Admin workstation | Record exact success test | Not applicable | Fix can be proven |

### Phase 4: Execute And Record Change

| Step | Task | Device | Command | PowerShell | Expected Result |
|---|---|---|---|---|---|
| 1 | Execute one change at a time | Admin workstation | Use portal, CLI, PowerShell, Graph, or pipeline | Depends on change | Change is controlled |
| 2 | Record old value | Admin workstation | Capture value before change | Depends on object | Previous state is known |
| 3 | Record new value | Admin workstation | Capture value after change | Depends on object | New state is known |
| 4 | Record actor | Admin workstation | Add admin or automation identity | Not applicable | Actor is known |
| 5 | Record timestamp | Admin workstation | Use consistent timezone | Not applicable | Timeline is accurate |
| 6 | Record command or portal path | Admin workstation | Paste command or portal navigation | Not applicable | Change is reproducible |
| 7 | Record result | Admin workstation | Success, failure, partial, pending | Not applicable | Change outcome is known |
| 8 | Link rollback ID | Admin workstation | Add rollback entry for change | Not applicable | Change has reversal plan |

### Phase 5: Validate And Monitor

| Step | Task | Device | Command | PowerShell | Expected Result |
|---|---|---|---|---|---|
| 1 | Repeat original failed test | Source workstation or admin workstation | Run same test that failed | Depends on issue | Original symptom is resolved |
| 2 | Validate with independent signal | Admin workstation | Check logs, trace, alert, service health, or user confirmation | Depends on workload | Fix is proven beyond one observation |
| 3 | Record validation evidence | Admin workstation | Add validation output to evidence index | Not applicable | Closure proof exists |
| 4 | Monitor for recurrence | Admin workstation | Watch alerts/logs/service/user reports for agreed time | Depends on workload | No immediate recurrence |
| 5 | Record monitoring result | Admin workstation | Update timeline and validation table | Not applicable | Monitoring is documented |
| 6 | Decide closure or escalation | Admin workstation | Close if resolved or escalate if blocker remains | Not applicable | Record status is accurate |

### Phase 6: Cleanup And Closure

| Step | Task | Device | Command | PowerShell | Expected Result |
|---|---|---|---|---|---|
| 1 | Review temporary exceptions | Admin workstation | Check exception register | Not applicable | Temporary risk is visible |
| 2 | Remove temporary access | Admin workstation | Remove RBAC, role, license, group membership, mailbox permission, or policy exclusion | Depends on object | Temporary access is removed |
| 3 | Restore policy settings | Admin workstation | Re-enable or restore CA, DLP, alert, firewall, or mail rule settings | Depends on object | Baseline policy state returns |
| 4 | Validate cleanup | Admin workstation | Confirm temporary object no longer exists | Depends on object | Cleanup is verified |
| 5 | Record final root cause | Admin workstation | Complete RCA fields | Not applicable | Root cause is documented |
| 6 | Record preventive actions | Admin workstation | Add action items with owners and dates | Not applicable | Follow-up is assigned |
| 7 | Close record | Admin workstation | Mark ticket closed/resolved | Not applicable | Record is complete and auditable |

---

## 07_Create_Cloud_Admin_Incident_Review_Change_Log_And_Rollback_Records_Workload_Evidence_Commands

### Microsoft Entra / Identity Evidence

| Purpose | PowerShell |
|---|---|
| Connect to Graph | `Connect-MgGraph -Scopes "User.Read.All","Group.Read.All","Directory.Read.All","AuditLog.Read.All","RoleManagement.Read.Directory","Policy.Read.All"` |
| Confirm tenant | `Get-MgOrganization | Select-Object DisplayName,Id,VerifiedDomains` |
| Capture user | `Get-MgUser -UserId <user@domain.com> -Property Id,UserPrincipalName,DisplayName,AccountEnabled,UserType,UsageLocation | ConvertTo-Json -Depth 10 | Out-File ".\EVID_user.json"` |
| Capture sign-ins | `Get-MgAuditLogSignIn -Filter "userPrincipalName eq '<user@domain.com>'" -Top 50 | ConvertTo-Json -Depth 10 | Out-File ".\EVID_signins.json"` |
| Capture directory audit | `Get-MgAuditLogDirectoryAudit -Top 100 | ConvertTo-Json -Depth 10 | Out-File ".\EVID_directory_audit.json"` |
| Capture CA policies | `Get-MgIdentityConditionalAccessPolicy -All | ConvertTo-Json -Depth 20 | Out-File ".\EVID_conditional_access.json"` |
| Capture role assignments | `Get-MgRoleManagementDirectoryRoleAssignmentScheduleInstance -All -ExpandProperty Principal,RoleDefinition | ConvertTo-Json -Depth 20 | Out-File ".\EVID_entra_roles.json"` |

### Azure Evidence

| Purpose | Command |
|---|---|
| Capture account context | `az account show > EVID_az_context.json` |
| Capture resource | `az resource show --ids <resource-id> > EVID_resource.json` |
| Capture resource group | `az group show --name <resource-group> > EVID_resource_group.json` |
| Capture activity log | `az monitor activity-log list --resource-group <resource-group> --offset 24h > EVID_activity_log.json` |
| Capture deployments | `az deployment group list -g <resource-group> > EVID_deployments.json` |
| Capture deployment operations | `az deployment operation group list -g <resource-group> -n <deployment-name> > EVID_deployment_operations.json` |
| Capture role assignments | `az role assignment list --scope <scope> --all > EVID_role_assignments.json` |
| Capture locks | `az lock list --scope <scope> > EVID_locks.json` |
| Capture policy assignments | `az policy assignment list --scope <scope> > EVID_policy_assignments.json` |

### Exchange Online / Mailbox Evidence

| Purpose | PowerShell |
|---|---|
| Connect to Exchange Online | `Connect-ExchangeOnline` |
| Capture recipient | `Get-Recipient <user-or-group@domain.com> | Format-List | Out-File ".\EVID_recipient.txt"` |
| Capture mailbox | `Get-EXOMailbox <mailbox@domain.com> -Properties * | ConvertTo-Json -Depth 10 | Out-File ".\EVID_mailbox.json"` |
| Capture mailbox stats | `Get-EXOMailboxStatistics <mailbox@domain.com> | Format-List | Out-File ".\EVID_mailbox_stats.txt"` |
| Capture mailbox permissions | `Get-MailboxPermission <mailbox@domain.com> | Export-Csv ".\EVID_mailbox_permissions.csv" -NoTypeInformation` |
| Capture recipient permissions | `Get-RecipientPermission <mailbox@domain.com> | Export-Csv ".\EVID_recipient_permissions.csv" -NoTypeInformation` |
| Capture message trace | `Get-MessageTrace -RecipientAddress <mailbox@domain.com> -StartDate (Get-Date).AddHours(-24) -EndDate (Get-Date) | Export-Csv ".\EVID_message_trace.csv" -NoTypeInformation` |
| Capture inbox rules | `Get-InboxRule -Mailbox <mailbox@domain.com> | Export-Csv ".\EVID_inbox_rules.csv" -NoTypeInformation` |
| Capture accepted domains | `Get-AcceptedDomain | Export-Csv ".\EVID_accepted_domains.csv" -NoTypeInformation` |

### Defender / Purview Evidence

| Purpose | PowerShell / KQL |
|---|---|
| Connect compliance PowerShell | `Connect-IPPSSession` |
| Capture audit search | `Search-UnifiedAuditLog -StartDate <start-date> -EndDate <end-date> -ResultSize 5000 | Export-Csv ".\EVID_audit.csv" -NoTypeInformation` |
| Capture DLP policies | `Get-DlpCompliancePolicy | Format-List | Out-File ".\EVID_dlp_policies.txt"` |
| Capture DLP rules | `Get-DlpComplianceRule | Format-List | Out-File ".\EVID_dlp_rules.txt"` |
| Email events KQL | `EmailEvents | where Timestamp > ago(7d) | where RecipientEmailAddress =~ "<user@domain.com>" | sort by Timestamp desc` |
| Device events KQL | `DeviceProcessEvents | where Timestamp > ago(7d) | where DeviceName =~ "<device-name>" | sort by Timestamp desc` |
| Alert evidence KQL | `AlertEvidence | where AlertId =~ "<alert-id>" | project Timestamp, AlertId, EntityType, EvidenceRole, AccountName, DeviceName, FileName, SHA256, RemoteUrl, RemoteIP` |

### Backup / Restore Evidence

| Purpose | PowerShell |
|---|---|
| Capture vault | `Get-AzRecoveryServicesVault -Name <vault-name> | ConvertTo-Json -Depth 10 | Out-File ".\EVID_vault.json"` |
| Capture backup items | `Get-AzRecoveryServicesBackupItem -BackupManagementType AzureVM -WorkloadType AzureVM -VaultId <vault-id> | ConvertTo-Json -Depth 10 | Out-File ".\EVID_backup_items.json"` |
| Capture backup jobs | `Get-AzRecoveryServicesBackupJob -VaultId <vault-id> -From (Get-Date).AddDays(-7) -To (Get-Date) | ConvertTo-Json -Depth 10 | Out-File ".\EVID_backup_jobs.json"` |
| Capture recovery points | `Get-AzRecoveryServicesBackupRecoveryPoint -Item <backup-item> -StartDate (Get-Date).AddDays(-30) -EndDate (Get-Date) | ConvertTo-Json -Depth 10 | Out-File ".\EVID_recovery_points.json"` |

---

## 07_Create_Cloud_Admin_Incident_Review_Change_Log_And_Rollback_Records_Rollback_Patterns

### Identity / Access Rollback Patterns

| Change | Rollback |
|---|---|
| Added user to privileged group | Remove user from group |
| Assigned Entra admin role | Remove role assignment or let PIM activation expire |
| Added Azure RBAC assignment | Remove role assignment |
| Added Conditional Access exclusion | Remove exclusion |
| Disabled Conditional Access policy | Re-enable policy |
| Reset user password | User completes secure reset; no direct rollback |
| Disabled user account | Re-enable account only after approval |
| Added Temporary Access Pass | Let expire or delete method |
| Revoke sessions | No rollback; user signs in again |

### Exchange / Microsoft 365 Rollback Patterns

| Change | Rollback |
|---|---|
| Assigned temporary license | Remove license |
| Enabled service plan | Restore previous service plan state |
| Added mailbox FullAccess | Remove mailbox permission |
| Added SendAs | Remove recipient permission |
| Added Send on Behalf | Remove delegate |
| Added forwarding | Remove forwarding or restore previous forwarding |
| Changed group membership | Remove or restore members |
| Changed distribution group restriction | Restore previous restriction |
| Released quarantine item | Remediate if incorrect release caused risk |
| Changed transport rule | Restore previous rule state |

### Azure Rollback Patterns

| Change | Rollback |
|---|---|
| Added NSG rule | Remove rule |
| Changed route table | Restore previous route |
| Changed private DNS link | Remove or restore link |
| Enabled public access | Restore prior network setting |
| Changed VM size | Resize back |
| Removed resource lock | Recreate lock |
| Changed diagnostic setting | Restore previous diagnostic setting |
| Added policy exemption | Remove or expire exemption |
| Changed alert threshold | Restore threshold |
| Registered provider | Usually no rollback required |

### Defender / Purview Rollback Patterns

| Change | Rollback |
|---|---|
| Isolated device | Release from isolation |
| Blocked indicator | Remove indicator |
| Suppressed alert | Remove suppression |
| Closed incident incorrectly | Reopen or create linked incident |
| Added DLP exception | Remove exception |
| Disabled DLP rule | Re-enable rule |
| Changed DLP from enforce to test | Restore enforce mode |
| Removed OAuth consent | Re-grant only through approved consent |
| Deleted message | Restore only if recoverable and approved |

---

## 07_Create_Cloud_Admin_Incident_Review_Change_Log_And_Rollback_Records_Validation_Test_Catalog

| Workload | Validation Test | Success Criteria |
|---|---|---|
| Entra login | User signs in from expected location/device | Sign-in succeeds and CA result is expected |
| MFA | User completes required MFA method | MFA succeeds without loop |
| PIM | User activates role and performs intended action | Role active and action succeeds |
| Azure RBAC | User performs required resource action | No AuthorizationFailed error |
| Azure deployment | Redeploy or validate deployment | Deployment succeeds or validation passes |
| Azure network | Source reaches destination on protocol/port | DNS, route, and TCP test pass |
| Storage | User/app reads or writes required object | Operation succeeds with expected auth |
| VM | VM boots and accepts expected management path | VM running and RDP/SSH/Bastion works |
| Monitor | Alert fires or log appears | Expected signal appears in alert/log |
| Backup | Backup job completes | Job status is Completed |
| Restore | Test restore completes and workload opens | Restored object is usable |
| Exchange mail flow | Send/receive test and message trace | Message delivered |
| Shared mailbox | Delegate opens mailbox and sends approved test | Permission works |
| Group | User receives group mail or access applies | Membership/delivery works |
| License | User opens licensed app/service | Service available |
| Defender | Alert no longer recurs after remediation | No matching new evidence |
| Purview audit | Audit search returns expected event | Event appears in search/export |
| DLP | Controlled test triggers or does not trigger as intended | Policy behavior matches design |

---

## 07_Create_Cloud_Admin_Incident_Review_Change_Log_And_Rollback_Records_Closure_Checklist

| Step | Closure Item | Complete |
|---|---|---|
| 1 | Incident/change/ticket ID recorded | `<yes-no>` |
| 2 | Scope documented | `<yes-no>` |
| 3 | Impact documented | `<yes-no>` |
| 4 | Timeline completed | `<yes-no>` |
| 5 | Evidence index completed | `<yes-no>` |
| 6 | Decision log completed | `<yes-no>` |
| 7 | Approval recorded where needed | `<yes-no>` |
| 8 | Change log completed | `<yes-no>` |
| 9 | Rollback plan documented | `<yes-no>` |
| 10 | Rollback executed or marked not needed | `<yes-no>` |
| 11 | Temporary exceptions removed or tracked | `<yes-no>` |
| 12 | Validation completed | `<yes-no>` |
| 13 | Root cause documented | `<yes-no>` |
| 14 | Preventive actions assigned | `<yes-no>` |
| 15 | Escalations/handoffs documented | `<yes-no>` |
| 16 | User/business owner confirmation captured | `<yes-no>` |
| 17 | Security/compliance review completed if needed | `<yes-no>` |
| 18 | Record closed by owner | `<yes-no>` |

---

## 07_Create_Cloud_Admin_Incident_Review_Change_Log_And_Rollback_Records_Failure_Checks

| Failure | Check | Fix |
|---|---|---|
| Record lacks root cause | Evidence and timeline incomplete | Reconstruct timeline and identify confirmed cause or mark unknown |
| Change log missing old value | Pre-change export not captured | Use audit logs, deployment history, or config history if available |
| Rollback cannot be executed | Dependency, data loss, or missing previous state | Escalate and document compensating action |
| Temporary exception forgotten | Exception register missing owner/expiration | Assign owner and cleanup date |
| Evidence cannot be found | Screenshots/exports stored outside ticket | Add evidence index and stable storage location |
| Approval not documented | Verbal approval only | Add approval note with approver, time, and action |
| Validation too vague | “Looks good” without test | Add exact test and result |
| Incident closed while alerts continue | Monitoring not checked | Reopen and monitor alert/log recurrence |
| Service health incident mistaken for local issue | Service health not checked | Add service health check to triage |
| False root cause recorded | Hypothesis treated as fact | Separate suspected and confirmed root cause |
| Rollback caused worse issue | Rollback not validated or dependencies missed | Use rollback trigger/validation table before executing |
| Compliance impact omitted | DLP/audit/data involved but compliance not engaged | Escalate to compliance/legal |
| Security impact omitted | Defender/identity/privilege involved but security not engaged | Escalate to security |
| User impact not documented | Only technical symptoms recorded | Add affected population and business process |
| No preventive action | Record closes after fix only | Add action item with owner and due date |
| No cleanup verification | Temporary change removed but not verified | Add cleanup validation entry |

---

## 07_Create_Cloud_Admin_Incident_Review_Change_Log_And_Rollback_Records_Escalation_Handoff

### Escalate To Cloud Platform Team When

- Azure subscription, management group, landing zone, policy, networking, backup, or monitoring owner must act
- Resource locks, deny assignments, policy inheritance, or quota blocks remediation
- Production deployment rollback requires platform approval
- Centralized logging or diagnostics are missing

### Escalate To Identity Team When

- Entra roles, PIM, MFA, Conditional Access, account sync, or sign-in issues are involved
- Break-glass access was used
- User or admin access changes have security impact
- Privileged role assignments require review

### Escalate To Messaging / Collaboration Team When

- Exchange, mailbox, mail flow, groups, Teams, SharePoint, or OneDrive are involved
- Message trace, quarantine, shared mailbox delegation, or distribution group delivery must be reviewed
- User-facing productivity issue needs workload owner validation

### Escalate To Security Team When

- Defender incident, suspicious sign-in, malware, phishing, OAuth abuse, data exfiltration, or privileged account activity is involved
- Containment or response actions are required
- Alert classification or closure requires security approval

### Escalate To Compliance / Legal When

- DLP, audit, retention, eDiscovery, legal hold, sensitive data, regulated data, or external sharing is involved
- Data deletion, release, or disclosure decision is needed
- Content Explorer or audit evidence requires formal handling

### Escalate To Microsoft Support When

- Platform state is inconsistent across portal, PowerShell, CLI, and logs
- Service health does not explain tenant-wide issue
- Resource, mailbox, role, alert, audit, or backup object is stuck or corrupted
- Correlation ID points to backend failure
- Tenant recovery or domain ownership recovery is required

### Handoff Data To Include

| Field | Value |
|---|---|
| Ticket ID | `<ticket-id>` |
| Incident ID | `<incident-id>` |
| Change ID | `<change-id>` |
| Tenant ID | `<tenant-id>` |
| Subscription ID | `<subscription-id>` |
| Workload | `<workload>` |
| Affected object | `<object>` |
| Severity | `<severity>` |
| Business impact | `<impact>` |
| Security impact | `<impact>` |
| Compliance impact | `<impact>` |
| Timeline summary | `<summary>` |
| Root cause status | `<suspected / confirmed / unknown>` |
| Evidence index location | `<location>` |
| Actions already taken | `<actions>` |
| Temporary exceptions active | `<exceptions>` |
| Rollback status | `<ready / executed / not-needed / blocked>` |
| Current blocker | `<blocker>` |
| Requested action | `<what-next-team-must-do>` |
| Needed approval | `<approval>` |

---

## 07_Create_Cloud_Admin_Incident_Review_Change_Log_And_Rollback_Records_Post_Incident_Review_Template

### Executive Summary

| Field | Entry |
|---|---|
| Incident title | `<title>` |
| Date | `<yyyy-mm-dd>` |
| Duration | `<duration>` |
| Severity | `<severity>` |
| Owner | `<owner>` |
| Affected services | `<services>` |
| Affected users | `<count/group>` |
| Business impact | `<impact>` |
| Final status | `<closed/resolved/monitoring>` |

### What Happened

| Field | Entry |
|---|---|
| Summary | `<short narrative>` |
| Trigger | `<trigger>` |
| Detection | `<how detected>` |
| Impact | `<impact>` |
| Mitigation | `<what reduced impact>` |
| Resolution | `<what fixed it>` |

### What Went Well

| Item | Notes |
|---|---|
| Detection | `<notes>` |
| Communication | `<notes>` |
| Evidence collection | `<notes>` |
| Remediation | `<notes>` |
| Rollback readiness | `<notes>` |

### What Did Not Go Well

| Item | Notes |
|---|---|
| Detection gaps | `<notes>` |
| Access gaps | `<notes>` |
| Documentation gaps | `<notes>` |
| Tooling gaps | `<notes>` |
| Approval delays | `<notes>` |
| Rollback gaps | `<notes>` |

### Preventive Actions

| Action ID | Action | Owner | Due Date | Success Metric |
|---|---|---|---|---|
| `ACT-001` | `<action>` | `<owner>` | `<date>` | `<metric>` |
| `ACT-002` | `<action>` | `<owner>` | `<date>` | `<metric>` |
| `ACT-003` | `<action>` | `<owner>` | `<date>` | `<metric>` |

---

## 07_Create_Cloud_Admin_Incident_Review_Change_Log_And_Rollback_Records_Related_Labs

| Lab | Purpose |
|---|---|
| Create cloud admin incident record | Practice incident summary, impact, timeline, and owner fields |
| Build evidence index from Azure and Microsoft 365 logs | Practice evidence capture and ID tracking |
| Create rollback record for Conditional Access change | Practice access-policy rollback planning |
| Create rollback record for Azure deployment | Practice deployment rollback and validation |
| Create rollback record for DNS and mail authentication | Practice MX/SPF/DKIM/DMARC rollback |
| Create mailbox permission change log | Practice Exchange delegation tracking |
| Create Defender incident response record | Practice security evidence, response actions, and closure |
| Create DLP exception record | Practice approval, expiration, and cleanup |
| Create backup restore validation record | Practice restore proof and data protection validation |
| Run post-incident review | Practice root cause, five whys, and preventive action assignment |
