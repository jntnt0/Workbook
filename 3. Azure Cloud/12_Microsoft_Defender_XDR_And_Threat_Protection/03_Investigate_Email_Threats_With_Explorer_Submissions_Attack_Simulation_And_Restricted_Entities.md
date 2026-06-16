# 03_Investigate_Email_Threats_With_Explorer_Submissions_Attack_Simulation_And_Restricted_Entities

## Objective

Investigate email threats in Microsoft Defender for Office 365 using Explorer, Submissions, Attack Simulation Training, and Restricted Entities. Validate how an administrator reviews phishing, malware, spoofing, user reports, simulation activity, and blocked senders or users.

## Lab Context

This workbook assumes a Microsoft 365 tenant using Exchange Online and Microsoft Defender for Office 365. The goal is to build a repeatable investigation workflow for email threats and post-delivery response.

## Prerequisites

| Requirement | Details |
|---|---|
| Admin Access | Security Administrator, Security Operator, Exchange Administrator, or Global Administrator |
| Portal | https://security.microsoft.com |
| Licensing | Microsoft Defender for Office 365 Plan 2 recommended for Explorer and Attack Simulation Training |
| Mailboxes | At least one test mailbox |
| Test Messages | Safe test phishing/spam samples or user-reported test messages |
| Browser | Edge or Chrome |
| Optional PowerShell | Exchange Online PowerShell module |

## Configuration Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|---:|---|---|---|---|---|
| 1 | Open Microsoft Defender Portal | Admin Workstation | Browse to https://security.microsoft.com | N/A | Defender portal loads |
| 2 | Verify Tenant Context | Admin Workstation | Confirm tenant from profile menu | N/A | Correct tenant displayed |
| 3 | Open Explorer | Defender Portal | Email & Collaboration → Explorer | N/A | Explorer opens |
| 4 | Set Investigation Time Range | Defender Portal | Select last 7 days or last 30 days | N/A | Explorer scoped to investigation window |
| 5 | Review All Email View | Defender Portal | Explorer → All email | N/A | Email events visible |
| 6 | Search By Recipient | Defender Portal | Filter recipient address | N/A | Messages to recipient displayed |
| 7 | Search By Sender | Defender Portal | Filter sender address | N/A | Messages from sender displayed |
| 8 | Search By Subject | Defender Portal | Filter subject or keyword | N/A | Matching messages displayed |
| 9 | Search By Internet Message ID | Defender Portal | Filter by Message ID if available | N/A | Specific message located |
| 10 | Search By Network Message ID | Defender Portal | Filter by Network Message ID if available | N/A | Message trace group located |
| 11 | Filter By Threat Type | Defender Portal | Threat type → Phish, Malware, Spam, or None | N/A | Threat-specific messages displayed |
| 12 | Filter By Delivery Action | Defender Portal | Delivery action → Delivered, Junked, Blocked, Quarantined, Removed | N/A | Delivery outcomes visible |
| 13 | Filter By Detection Technology | Defender Portal | Detection technology filter | N/A | Detection source identified |
| 14 | Open Message Details | Defender Portal | Select message | N/A | Message details flyout opens |
| 15 | Review Message Summary | Defender Portal | Message Details → Summary | N/A | Sender, recipient, subject, direction, delivery action visible |
| 16 | Review Authentication Results | Defender Portal | Message Details → Authentication | N/A | SPF, DKIM, DMARC, composite authentication visible |
| 17 | Review URLs | Defender Portal | Message Details → URLs | N/A | URLs listed if present |
| 18 | Review Attachments | Defender Portal | Message Details → Attachments | N/A | Attachments listed if present |
| 19 | Review Timeline | Defender Portal | Message Details → Timeline | N/A | Delivery and post-delivery actions visible |
| 20 | Review Similar Emails | Defender Portal | Use Explorer filters from sender, URL, subject, or campaign | N/A | Related messages identified |
| 21 | Export Investigation Results | Defender Portal | Export from Explorer | N/A | CSV export downloaded if permitted |
| 22 | Open Campaigns View | Defender Portal | Email & Collaboration → Campaigns | N/A | Campaign view opens if available |
| 23 | Review Campaign Details | Defender Portal | Select campaign | N/A | Campaign sender, recipients, payload, and delivery details visible |
| 24 | Open URL Clicks | Defender Portal | Explorer → URL Clicks | N/A | Safe Links click events visible |
| 25 | Search URL Clicks By User | Defender Portal | Filter Account or Recipient | N/A | User click activity visible |
| 26 | Search URL Clicks By URL | Defender Portal | Filter URL or domain | N/A | URL click timeline visible |
| 27 | Determine User Exposure | Defender Portal | Review clicked vs blocked actions | N/A | User exposure status determined |
| 28 | Open Submissions | Defender Portal | Actions & Submissions → Submissions | N/A | Submissions page opens |
| 29 | Review User Reported Messages | Defender Portal | Submissions → User reported | N/A | User reported email list visible |
| 30 | Review Admin Submitted Messages | Defender Portal | Submissions → Submitted for analysis | N/A | Admin submissions visible |
| 31 | Submit Email To Microsoft | Defender Portal | Submissions → Submit to Microsoft | N/A | Submission wizard opens |
| 32 | Choose Submission Type | Defender Portal | Select Email, URL, Attachment, or Teams message if available | N/A | Correct submission type selected |
| 33 | Upload Or Select Message | Defender Portal | Select message from Explorer or upload .eml/.msg if allowed | N/A | Message attached to submission |
| 34 | Mark Submission Category | Defender Portal | Choose Phish, Malware, Spam, or Not junk | N/A | Submission category selected |
| 35 | Submit For Analysis | Defender Portal | Review → Submit | N/A | Submission recorded |
| 36 | Track Submission Result | Defender Portal | Submissions → Microsoft results | N/A | Analysis result visible when complete |
| 37 | Open Quarantine | Defender Portal | Email & Collaboration → Review → Quarantine | N/A | Quarantine queue opens |
| 38 | Search Quarantine By Recipient | Defender Portal | Filter recipient | N/A | Quarantined messages for recipient visible |
| 39 | Search Quarantine By Reason | Defender Portal | Filter reason Phish, Malware, Spam, High Confidence Phish | N/A | Quarantine reason identified |
| 40 | Review Quarantined Message | Defender Portal | Select message | N/A | Message details visible |
| 41 | Preview Quarantined Message Safely | Defender Portal | Preview message if allowed | N/A | Message safely reviewed |
| 42 | Release Known Safe Test Message | Defender Portal | Release message only if safe and authorized | N/A | Message released |
| 43 | Delete Malicious Message From Quarantine | Defender Portal | Delete message if confirmed malicious | N/A | Message removed from quarantine |
| 44 | Open Restricted Entities | Defender Portal | Email & Collaboration → Review → Restricted Entities | N/A | Restricted entities page opens |
| 45 | Review Restricted Users | Defender Portal | Restricted Entities → Restricted Users | N/A | Restricted sending users visible |
| 46 | Review Restricted Connectors | Defender Portal | Restricted Entities → Restricted Connectors | N/A | Restricted connectors visible |
| 47 | Investigate Restricted User | Defender Portal | Select restricted user | N/A | Restriction reason and details visible |
| 48 | Validate Compromise Indicators | Defender Portal | Review sent volume, message samples, sign-in risk, inbox rules | N/A | Compromise indicators documented |
| 49 | Remove Restriction Only After Remediation | Defender Portal | Unblock user if authorized | N/A | User sending restored |
| 50 | Open Attack Simulation Training | Defender Portal | Email & Collaboration → Attack Simulation Training | N/A | Attack Simulation Training opens |
| 51 | Review Simulation Dashboard | Defender Portal | Attack Simulation Training → Overview | N/A | Simulation metrics visible |
| 52 | Review Existing Simulations | Defender Portal | Simulations tab | N/A | Existing simulation campaigns visible |
| 53 | Create Test Simulation | Defender Portal | Launch a simulation | N/A | Simulation wizard starts |
| 54 | Select Payload Type | Defender Portal | Credential harvest, malware attachment, link in attachment, drive-by URL, or OAuth consent grant where available | N/A | Payload technique selected |
| 55 | Select Training Payload | Defender Portal | Choose Microsoft or tenant payload | N/A | Payload selected |
| 56 | Select Target Users | Defender Portal | Choose pilot test users only | N/A | Test users scoped |
| 57 | Configure Landing Page | Defender Portal | Select training landing page | N/A | Landing page configured |
| 58 | Configure End User Notifications | Defender Portal | Select notification template | N/A | Notifications configured |
| 59 | Schedule Simulation | Defender Portal | Launch now or schedule | N/A | Simulation scheduled |
| 60 | Review Simulation Results | Defender Portal | Open simulation results | N/A | Opens, clicks, credentials submitted, training completion visible |
| 61 | Correlate Simulation With Explorer | Defender Portal | Search simulation sender, subject, or payload | N/A | Simulation messages visible in Explorer if logged |
| 62 | Document Investigation Findings | Admin Workstation | Record sender, recipient, subject, threat type, delivery action, user clicks, submissions, quarantine action, restricted entity status | N/A | Investigation record completed |

## Investigation Workflow

| Phase | Action | Expected Result |
|---|---|---|
| Scope | Identify sender, recipient, subject, Message ID, URL, attachment hash, or time range | Search criteria defined |
| Search | Use Explorer filters | Related email events identified |
| Analyze | Review authentication, URLs, attachments, delivery action, and timeline | Threat behavior understood |
| Determine Exposure | Check delivered messages, user clicks, and post-delivery actions | Impacted users identified |
| Contain | Quarantine, delete, block, or restrict only if authorized | Threat contained |
| Submit | Submit email, URL, or attachment to Microsoft | Verdict requested or corrected |
| Review | Check submissions, quarantine, and restricted entities | Response status confirmed |
| Document | Record indicators, affected users, actions, and final classification | Investigation closed cleanly |

## Optional Exchange Online PowerShell Checks

| Step | Task | Device | Command | PowerShell | Expected Result |
|---:|---|---|---|---|---|
| 1 | Install Exchange Online Module | Admin Workstation | N/A | `Install-Module ExchangeOnlineManagement -Scope CurrentUser` | Module installed |
| 2 | Connect To Exchange Online | Admin Workstation | N/A | `Connect-ExchangeOnline` | Connected to Exchange Online |
| 3 | Review Restricted Users | Admin Workstation | N/A | `Get-BlockedSenderAddress` | Blocked sender addresses listed if available |
| 4 | Review Tenant Allow Block List Items | Admin Workstation | N/A | `Get-TenantAllowBlockListItems` | Tenant allow/block entries listed |
| 5 | Review Hosted Content Filter Policy | Admin Workstation | N/A | `Get-HostedContentFilterPolicy` | Anti-spam policies listed |
| 6 | Review Anti-Phish Policy | Admin Workstation | N/A | `Get-AntiPhishPolicy` | Anti-phishing policies listed |
| 7 | Review Safe Links Policy | Admin Workstation | N/A | `Get-SafeLinksPolicy` | Safe Links policies listed |
| 8 | Review Safe Attachments Policy | Admin Workstation | N/A | `Get-SafeAttachmentPolicy` | Safe Attachments policies listed |
| 9 | Disconnect Exchange Online | Admin Workstation | N/A | `Disconnect-ExchangeOnline -Confirm:$false` | Session disconnected |

## Optional Advanced Hunting Queries

### Recent Email Threats

```kql
EmailEvents
| where Timestamp > ago(7d)
| where isnotempty(ThreatTypes)
| project Timestamp, SenderFromAddress, RecipientEmailAddress, Subject, ThreatTypes, DetectionMethods, DeliveryAction, DeliveryLocation, NetworkMessageId
| order by Timestamp desc
```

### Search By Sender

```kql
EmailEvents
| where Timestamp > ago(30d)
| where SenderFromAddress =~ "sender@example.com"
| project Timestamp, SenderFromAddress, RecipientEmailAddress, Subject, ThreatTypes, DeliveryAction, NetworkMessageId
| order by Timestamp desc
```

### Search By Recipient

```kql
EmailEvents
| where Timestamp > ago(30d)
| where RecipientEmailAddress =~ "user@example.com"
| project Timestamp, SenderFromAddress, RecipientEmailAddress, Subject, ThreatTypes, DeliveryAction, NetworkMessageId
| order by Timestamp desc
```

### URL Click Investigation

```kql
UrlClickEvents
| where Timestamp > ago(30d)
| project Timestamp, AccountUpn, Url, ActionType, Workload, NetworkMessageId
| order by Timestamp desc
```

### Attachment Investigation

```kql
EmailAttachmentInfo
| where Timestamp > ago(30d)
| project Timestamp, SenderFromAddress, RecipientEmailAddress, FileName, FileType, SHA256, NetworkMessageId
| order by Timestamp desc
```

### Join Email And Attachment Events

```kql
EmailEvents
| where Timestamp > ago(30d)
| project Timestamp, SenderFromAddress, RecipientEmailAddress, Subject, ThreatTypes, DeliveryAction, NetworkMessageId
| join kind=leftouter (
    EmailAttachmentInfo
    | where Timestamp > ago(30d)
    | project NetworkMessageId, FileName, FileType, SHA256
) on NetworkMessageId
| project Timestamp, SenderFromAddress, RecipientEmailAddress, Subject, ThreatTypes, DeliveryAction, FileName, FileType, SHA256, NetworkMessageId
| order by Timestamp desc
```

### Find Potential Phishing Messages

```kql
EmailEvents
| where Timestamp > ago(30d)
| where ThreatTypes has "Phish" or DetectionMethods has "Phish"
| project Timestamp, SenderFromAddress, RecipientEmailAddress, Subject, ThreatTypes, DetectionMethods, DeliveryAction, DeliveryLocation
| order by Timestamp desc
```

## Validation

| Check | Expected Result |
|---|---|
| Explorer Opens | Success |
| All Email View Loads | Success |
| Recipient Search Works | Success |
| Sender Search Works | Success |
| Threat Type Filtering Works | Success |
| Message Details Visible | Success |
| URL Clicks Visible If Safe Links Data Exists | Success |
| Submissions Page Opens | Success |
| User Reported Messages Visible If Configured | Success |
| Admin Submission Can Be Created | Success |
| Quarantine Queue Opens | Success |
| Restricted Entities Page Opens | Success |
| Attack Simulation Training Opens If Licensed | Success |
| Simulation Results Visible If Simulations Exist | Success |
| Advanced Hunting Queries Run If Data Exists | Success |

## Troubleshooting

| Symptom | Likely Cause | Resolution |
|---|---|---|
| Explorer Missing | Defender for Office 365 Plan 2 not licensed | Verify licensing |
| Explorer Shows No Data | Time range too narrow or no mail flow | Expand time range and verify Exchange Online mail flow |
| Message Not Found | Wrong Message ID, recipient, sender, or time range | Search by multiple indicators |
| URL Clicks Empty | Safe Links not enabled or no clicks occurred | Validate Safe Links policy and search wider range |
| Submissions Missing | Insufficient permissions | Assign Security Operator, Security Administrator, or appropriate submission role |
| Cannot Submit Message | Unsupported file type or missing permission | Use .eml/.msg or submit from Explorer |
| Quarantine Empty | No quarantined mail or filters applied | Clear filters and expand time range |
| Cannot Preview Quarantine | Quarantine policy blocks preview | Review quarantine policy permissions |
| Cannot Release Message | Quarantine policy or role restriction | Use authorized admin role or adjust policy |
| Restricted Entities Empty | No users or connectors restricted | This is normal if no outbound abuse detected |
| Cannot Unblock Restricted User | Remediation incomplete or insufficient role | Complete compromise remediation and verify admin role |
| Attack Simulation Missing | License not available | Verify Defender for Office 365 Plan 2 |
| Simulation Cannot Launch | Missing permissions or payload configuration | Validate Attack Simulation role and target scope |
| Advanced Hunting Table Missing | Workload telemetry unavailable | Validate Defender workload licensing and onboarding |

## Rollback

| Step | Action | Expected Result |
|---:|---|---|
| 1 | Clear Explorer Filters | Default Explorer view restored |
| 2 | Cancel Test Simulation If Scheduled | Simulation does not run |
| 3 | Delete Draft Simulation If Not Needed | Simulation removed |
| 4 | Revert Any Test Quarantine Release | Confirm no malicious production mail was released |
| 5 | Remove Temporary Tenant Allow Or Block Entries | Allow/block list restored |
| 6 | Restore Restricted User Only After Verification | User sending restored safely |
| 7 | Disconnect PowerShell Session | Admin session closed |
| 8 | Archive Investigation Notes | Evidence retained for audit |

## Documentation Checklist

| Item | Status |
|---|---|
| Tenant Verified | ☐ |
| Investigator Account Recorded | ☐ |
| Time Range Recorded | ☐ |
| Sender Address Recorded | ☐ |
| Recipient Address Recorded | ☐ |
| Subject Recorded | ☐ |
| Internet Message ID Recorded | ☐ |
| Network Message ID Recorded | ☐ |
| Threat Type Recorded | ☐ |
| Detection Method Recorded | ☐ |
| Delivery Action Recorded | ☐ |
| Delivery Location Recorded | ☐ |
| URLs Reviewed | ☐ |
| Attachments Reviewed | ☐ |
| User Clicks Reviewed | ☐ |
| Quarantine Status Reviewed | ☐ |
| Submission Created If Needed | ☐ |
| Restricted Entity Status Reviewed | ☐ |
| Attack Simulation Results Reviewed | ☐ |
| Final Classification Recorded | ☐ |
| Response Actions Recorded | ☐ |

## Notes

- Explorer is the main investigation surface for Defender for Office 365 Plan 2.
- Use multiple indicators during investigation because sender, subject, and message IDs may not tell the full story alone.
- Do not release quarantined messages unless they are confirmed safe.
- User click data is critical for determining exposure after phishing delivery.
- Submissions help correct verdicts and improve Microsoft detection.
- Restricted entities usually indicate outbound abuse or suspected compromise and should not be unblocked until remediation is complete.
- Attack Simulation Training should be scoped to pilot users before broader campaigns.
- Always preserve investigation notes before deleting, releasing, or remediating messages.