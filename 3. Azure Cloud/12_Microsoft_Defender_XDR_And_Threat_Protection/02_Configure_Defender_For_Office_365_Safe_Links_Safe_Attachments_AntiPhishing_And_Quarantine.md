# 02_Configure_Defender_For_Office_365_Safe_Links_Safe_Attachments_AntiPhishing_And_Quarantine

## Objective

Configure Microsoft Defender for Office 365 baseline protections for Safe Links, Safe Attachments, anti-phishing, and quarantine so Exchange Online mail flow has a practical email threat protection baseline.

## Lab Context

This workbook assumes a Microsoft 365 tenant using Exchange Online and Microsoft Defender for Office 365. The goal is to configure and validate common email protection controls used by administrators and SOC teams.

## Prerequisites

| Requirement | Details |
|---|---|
| Admin Access | Security Administrator, Exchange Administrator, Global Administrator, or appropriate Defender role |
| Portal | https://security.microsoft.com |
| Exchange Admin Center | https://admin.exchange.microsoft.com |
| Licensing | Microsoft Defender for Office 365 Plan 1 or Plan 2 |
| Mailboxes | At least one test mailbox |
| Test Domain | Accepted domain configured in Microsoft 365 |
| Browser | Edge or Chrome |
| PowerShell Optional | Exchange Online PowerShell module |

## Configuration Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|---:|---|---|---|---|---|
| 1 | Open Microsoft Defender Portal | Admin Workstation | Browse to https://security.microsoft.com | N/A | Defender portal loads |
| 2 | Verify Tenant Context | Admin Workstation | Confirm tenant from profile menu | N/A | Correct tenant displayed |
| 3 | Open Email Collaboration Policies | Defender Portal | Email & Collaboration → Policies & Rules → Threat Policies | N/A | Threat policies page opens |
| 4 | Review Preset Security Policies | Defender Portal | Threat Policies → Preset Security Policies | N/A | Standard and Strict preset policies visible |
| 5 | Decide Baseline Policy Model | Defender Portal | Choose Preset or Custom | N/A | Configuration approach selected |
| 6 | Open Safe Links Policies | Defender Portal | Threat Policies → Safe Links | N/A | Safe Links policy page opens |
| 7 | Create Safe Links Policy | Defender Portal | Safe Links → Create | N/A | Safe Links policy wizard starts |
| 8 | Name Safe Links Policy | Defender Portal | Enter `Baseline Safe Links Policy` | N/A | Policy named |
| 9 | Configure Users and Domains Scope | Defender Portal | Add test users, groups, or domains | N/A | Policy scoped correctly |
| 10 | Enable URL Rewriting and Detonation | Defender Portal | Enable Safe Links protection for email | N/A | Links are checked at click time |
| 11 | Enable Internal Message Protection | Defender Portal | Enable protection for messages sent within organization | N/A | Internal mail links protected |
| 12 | Enable Teams Protection If Available | Defender Portal | Enable Safe Links for Microsoft Teams | N/A | Teams links protected if licensed |
| 13 | Enable Office App Protection | Defender Portal | Enable Safe Links for Office apps | N/A | Office document links protected |
| 14 | Disable User Click Through For Malicious Links | Defender Portal | Prevent users from clicking through to original URL | N/A | Users blocked from unsafe links |
| 15 | Enable Click Tracking | Defender Portal | Track user clicks | N/A | Click activity logged |
| 16 | Save Safe Links Policy | Defender Portal | Review → Submit | N/A | Safe Links policy created |
| 17 | Open Safe Attachments Policies | Defender Portal | Threat Policies → Safe Attachments | N/A | Safe Attachments policy page opens |
| 18 | Create Safe Attachments Policy | Defender Portal | Safe Attachments → Create | N/A | Safe Attachments policy wizard starts |
| 19 | Name Safe Attachments Policy | Defender Portal | Enter `Baseline Safe Attachments Policy` | N/A | Policy named |
| 20 | Configure Safe Attachments Scope | Defender Portal | Add test users, groups, or domains | N/A | Policy scoped correctly |
| 21 | Select Safe Attachments Action | Defender Portal | Choose Dynamic Delivery or Block | N/A | Attachment scanning behavior configured |
| 22 | Enable Redirect If Needed | Defender Portal | Configure redirect address for detected malware if used | N/A | Redirect address configured or left disabled |
| 23 | Enable Safe Attachments for SharePoint OneDrive Teams | Defender Portal | Turn on protection for files in SharePoint, OneDrive, and Teams if available | N/A | Collaboration files protected |
| 24 | Save Safe Attachments Policy | Defender Portal | Review → Submit | N/A | Safe Attachments policy created |
| 25 | Open Anti-Phishing Policies | Defender Portal | Threat Policies → Anti-phishing | N/A | Anti-phishing policy page opens |
| 26 | Create Anti-Phishing Policy | Defender Portal | Anti-phishing → Create | N/A | Anti-phishing wizard starts |
| 27 | Name Anti-Phishing Policy | Defender Portal | Enter `Baseline Anti-Phishing Policy` | N/A | Policy named |
| 28 | Configure Anti-Phishing Scope | Defender Portal | Add test users, groups, or domains | N/A | Policy scoped correctly |
| 29 | Add Users To Protect | Defender Portal | Add executives, admins, finance users, or test VIPs | N/A | Protected users added |
| 30 | Add Domains To Protect | Defender Portal | Add owned domains and common spoofing targets | N/A | Protected domains added |
| 31 | Enable Mailbox Intelligence | Defender Portal | Turn on mailbox intelligence | N/A | User communication patterns used for detection |
| 32 | Enable Intelligence Based Impersonation Protection | Defender Portal | Turn on impersonation protection | N/A | User/domain impersonation protection active |
| 33 | Configure User Impersonation Action | Defender Portal | Select Quarantine or Move to Junk | N/A | User impersonation action configured |
| 34 | Configure Domain Impersonation Action | Defender Portal | Select Quarantine or Move to Junk | N/A | Domain impersonation action configured |
| 35 | Configure Spoof Intelligence Action | Defender Portal | Select quarantine or reject based on policy | N/A | Spoof handling configured |
| 36 | Configure Safety Tips | Defender Portal | Enable first contact, impersonation, or suspicious safety tips where available | N/A | User warning banners enabled |
| 37 | Save Anti-Phishing Policy | Defender Portal | Review → Submit | N/A | Anti-phishing policy created |
| 38 | Open Anti-Spam Policies | Defender Portal | Threat Policies → Anti-spam | N/A | Anti-spam policy page opens |
| 39 | Review Inbound Anti-Spam Policy | Defender Portal | Open default or custom inbound policy | N/A | Inbound spam controls visible |
| 40 | Configure Phishing Action | Defender Portal | Set phishing to Quarantine | N/A | Phishing messages quarantined |
| 41 | Configure High Confidence Phishing Action | Defender Portal | Set high confidence phishing to Quarantine | N/A | High confidence phishing quarantined |
| 42 | Configure Spam Action | Defender Portal | Move to Junk or Quarantine | N/A | Spam action configured |
| 43 | Configure Bulk Email Threshold | Defender Portal | Set BCL threshold according to tenant standard | N/A | Bulk mail handling configured |
| 44 | Save Anti-Spam Policy | Defender Portal | Review → Submit | N/A | Anti-spam baseline saved |
| 45 | Open Anti-Malware Policies | Defender Portal | Threat Policies → Anti-malware | N/A | Anti-malware policy page opens |
| 46 | Review Malware Detection Action | Defender Portal | Confirm malware attachments are blocked | N/A | Malware block behavior confirmed |
| 47 | Review Common Attachment Filter | Defender Portal | Enable common attachment filtering if required | N/A | Risky attachment types blocked |
| 48 | Save Anti-Malware Settings | Defender Portal | Review → Submit | N/A | Anti-malware baseline saved |
| 49 | Open Quarantine Policies | Defender Portal | Threat Policies → Quarantine Policies | N/A | Quarantine policy page opens |
| 50 | Review Default Quarantine Policies | Defender Portal | Review AdminOnlyAccessPolicy, DefaultFullAccessPolicy, DefaultFullAccessWithNotificationPolicy | N/A | Default quarantine behavior understood |
| 51 | Create Custom Quarantine Policy If Needed | Defender Portal | Quarantine Policies → Add custom policy | N/A | Custom quarantine behavior created |
| 52 | Configure User Access To Quarantine | Defender Portal | Allow or deny preview, release request, release, delete based on security standard | N/A | User quarantine access controlled |
| 53 | Configure Quarantine Notifications | Defender Portal | Enable notifications if allowed | N/A | Users receive quarantine notifications if configured |
| 54 | Apply Quarantine Policy To Threat Policies | Defender Portal | Assign quarantine policy inside anti-phishing, anti-spam, or Safe Attachments policy | N/A | Quarantine behavior linked to threat policy |
| 55 | Open Quarantine Queue | Defender Portal | Email & Collaboration → Review → Quarantine | N/A | Quarantine queue loads |
| 56 | Filter Quarantine By Reason | Defender Portal | Filter by Phish, Malware, Spam, Transport Rule, or High Confidence Phish | N/A | Quarantined messages grouped |
| 57 | Review Quarantined Message Details | Defender Portal | Select quarantined message | N/A | Sender, recipient, subject, detection reason visible |
| 58 | Preview Message Safely | Defender Portal | Use preview if permitted | N/A | Message preview available without release |
| 59 | Release Test Message If Appropriate | Defender Portal | Release only known safe test message | N/A | Message released to recipient |
| 60 | Submit False Positive If Needed | Defender Portal | Submit message to Microsoft from quarantine or submissions | N/A | Submission recorded |
| 61 | Open Threat Explorer | Defender Portal | Email & Collaboration → Explorer | N/A | Explorer opens if Plan 2 available |
| 62 | Search Recent Phishing Detections | Defender Portal | Filter detection technology or threat type | N/A | Email threat data visible |
| 63 | Search URL Click Activity | Defender Portal | Use URL clicks tab if available | N/A | Safe Links click activity visible |
| 64 | Search Malware Attachment Events | Defender Portal | Filter by malware or attachment detections | N/A | Attachment detections visible |
| 65 | Document Policy Baseline | Admin Workstation | Record configured policies, scope, actions, quarantine behavior | N/A | Baseline documented |

## Optional Exchange Online PowerShell Checks

| Step | Task | Device | Command | PowerShell | Expected Result |
|---:|---|---|---|---|---|
| 1 | Install Exchange Online Module | Admin Workstation | N/A | `Install-Module ExchangeOnlineManagement -Scope CurrentUser` | Module installed |
| 2 | Connect To Exchange Online | Admin Workstation | N/A | `Connect-ExchangeOnline` | Connected to Exchange Online |
| 3 | Review Safe Links Policies | Admin Workstation | N/A | `Get-SafeLinksPolicy` | Safe Links policies listed |
| 4 | Review Safe Links Rules | Admin Workstation | N/A | `Get-SafeLinksRule` | Safe Links rules listed |
| 5 | Review Safe Attachments Policies | Admin Workstation | N/A | `Get-SafeAttachmentPolicy` | Safe Attachments policies listed |
| 6 | Review Safe Attachments Rules | Admin Workstation | N/A | `Get-SafeAttachmentRule` | Safe Attachments rules listed |
| 7 | Review Anti-Phishing Policies | Admin Workstation | N/A | `Get-AntiPhishPolicy` | Anti-phishing policies listed |
| 8 | Review Anti-Phishing Rules | Admin Workstation | N/A | `Get-AntiPhishRule` | Anti-phishing rules listed |
| 9 | Review Hosted Content Filter Policies | Admin Workstation | N/A | `Get-HostedContentFilterPolicy` | Anti-spam policies listed |
| 10 | Review Malware Filter Policies | Admin Workstation | N/A | `Get-MalwareFilterPolicy` | Anti-malware policies listed |
| 11 | Review Quarantine Policies | Admin Workstation | N/A | `Get-QuarantinePolicy` | Quarantine policies listed |
| 12 | Disconnect Exchange Online | Admin Workstation | N/A | `Disconnect-ExchangeOnline -Confirm:$false` | Session disconnected |

## Optional Advanced Hunting Queries

### Recent Email Detections

```kql
EmailEvents
| where Timestamp > ago(7d)
| project Timestamp, SenderFromAddress, RecipientEmailAddress, Subject, ThreatTypes, DetectionMethods, DeliveryAction
| order by Timestamp desc
```

### Safe Links URL Clicks

```kql
UrlClickEvents
| where Timestamp > ago(7d)
| project Timestamp, AccountUpn, Url, ActionType, Workload
| order by Timestamp desc
```

### Email Attachment Events

```kql
EmailAttachmentInfo
| where Timestamp > ago(7d)
| project Timestamp, SenderFromAddress, RecipientEmailAddress, FileName, FileType, SHA256
| order by Timestamp desc
```

### Quarantined Email Events

```kql
EmailEvents
| where Timestamp > ago(7d)
| where DeliveryLocation has "Quarantine" or DeliveryAction has "Quarantine"
| project Timestamp, SenderFromAddress, RecipientEmailAddress, Subject, ThreatTypes, DeliveryAction, DeliveryLocation
| order by Timestamp desc
```

### Phishing Detections

```kql
EmailEvents
| where Timestamp > ago(7d)
| where ThreatTypes has "Phish"
| project Timestamp, SenderFromAddress, RecipientEmailAddress, Subject, DetectionMethods, DeliveryAction
| order by Timestamp desc
```

## Validation

| Check | Expected Result |
|---|---|
| Defender Portal Opens | Success |
| Threat Policies Page Opens | Success |
| Safe Links Policy Created | Success |
| Safe Attachments Policy Created | Success |
| Anti-Phishing Policy Created | Success |
| Anti-Spam Baseline Reviewed | Success |
| Anti-Malware Baseline Reviewed | Success |
| Quarantine Policies Reviewed | Success |
| Quarantine Queue Accessible | Success |
| Explorer Accessible If Licensed | Success |
| PowerShell Policy Queries Return Results | Success |
| Advanced Hunting Queries Run If Data Exists | Success |

## Troubleshooting

| Symptom | Likely Cause | Resolution |
|---|---|---|
| Safe Links Policy Missing | Defender for Office 365 not licensed | Verify licensing |
| Safe Attachments Policy Missing | Defender for Office 365 not licensed | Verify Plan 1 or Plan 2 |
| Cannot Edit Threat Policies | Insufficient role | Assign Security Administrator or Exchange Administrator |
| Policy Does Not Apply | Scope mismatch | Verify users, groups, domains, and priority |
| User Still Clicks Malicious URL | Click-through allowed | Disable user click-through |
| Attachments Delayed | Dynamic Delivery or detonation delay | Confirm Safe Attachments action |
| Phishing Delivered To Inbox | Policy priority or action too weak | Review anti-phishing and anti-spam policy order |
| Quarantine Empty | No detections or filters applied | Clear filters and expand time range |
| User Cannot Release Quarantine | Quarantine policy restricts release | Adjust quarantine policy permissions |
| Users Not Getting Quarantine Notifications | Notifications disabled | Enable quarantine notifications |
| Explorer Missing | Defender for Office 365 Plan 2 not available | Confirm licensing |
| Advanced Hunting Tables Empty | No telemetry or wrong table | Validate mail flow and Defender data availability |
| PowerShell Command Fails | Not connected to Exchange Online | Run Connect-ExchangeOnline |

## Rollback

| Step | Action | Expected Result |
|---:|---|---|
| 1 | Disable Custom Safe Links Policy | Safe Links policy no longer applies |
| 2 | Disable Custom Safe Attachments Policy | Safe Attachments policy no longer applies |
| 3 | Disable Custom Anti-Phishing Policy | Anti-phishing policy no longer applies |
| 4 | Restore Previous Anti-Spam Settings | Previous spam handling restored |
| 5 | Restore Previous Anti-Malware Settings | Previous malware handling restored |
| 6 | Remove Custom Quarantine Policy If Created | Quarantine behavior returns to prior baseline |
| 7 | Clear Test Filters | Default portal views restored |
| 8 | Disconnect PowerShell Session | Admin session closed |

## Documentation Checklist

| Item | Status |
|---|---|
| Tenant Verified | ☐ |
| Defender for Office 365 License Verified | ☐ |
| Safe Links Policy Name Recorded | ☐ |
| Safe Links Scope Recorded | ☐ |
| Safe Attachments Policy Name Recorded | ☐ |
| Safe Attachments Scope Recorded | ☐ |
| Anti-Phishing Policy Name Recorded | ☐ |
| Protected Users Recorded | ☐ |
| Protected Domains Recorded | ☐ |
| Anti-Spam Actions Recorded | ☐ |
| Anti-Malware Actions Recorded | ☐ |
| Quarantine Policies Reviewed | ☐ |
| Quarantine Notification Behavior Recorded | ☐ |
| Explorer Tested | ☐ |
| Advanced Hunting Tested | ☐ |
| Rollback Plan Documented | ☐ |

## Notes

- Use preset security policies when you want a Microsoft-managed baseline with less manual tuning.
- Use custom policies when you need explicit scope, priority, and quarantine behavior.
- Safe Links protects users at time of click, not only at time of mail delivery.
- Safe Attachments can delay attachment availability while detonation completes.
- Anti-phishing protection is stronger when protected users, protected domains, mailbox intelligence, and spoof intelligence are all reviewed.
- Quarantine policy controls what users can see, request, release, delete, or preview.
- Do not release real suspicious email during a lab.
- Always test policy scope with pilot users before applying tenant-wide.
- Document policy priority because overlapping mail protection policies can produce confusing results.