# 01_Configure_Defender_XDR_Portal_Secure_Score_Exposure_Management_Incidents_And_Alerts

## Objective

Configure the Microsoft Defender XDR portal baseline, review Microsoft Secure Score, validate Exposure Management, and confirm incident and alert visibility for a Microsoft 365 security operations baseline.

## Lab Context

This workbook assumes a Microsoft 365 tenant with Microsoft Defender XDR licensing available. The goal is to validate that the security portal is usable for daily SOC operations and establish a repeatable Defender XDR administration baseline.

## Prerequisites

| Requirement | Details |
|---|---|
| Admin Access | Security Administrator, Security Operator, Security Reader, or Global Administrator |
| Portal | https://security.microsoft.com |
| Licensing | Microsoft Defender XDR licensing |
| Browser | Edge or Chrome |
| Test Environment | Recommended |
| Internet Access | Required |

## Configuration Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|---:|---|---|---|---|---|
| 1 | Open Microsoft Defender Portal | Admin Workstation | Browse to https://security.microsoft.com | N/A | Defender portal loads successfully |
| 2 | Verify Tenant Context | Admin Workstation | Verify active tenant in profile menu | N/A | Correct tenant displayed |
| 3 | Validate Administrative Permissions | Defender Portal | Settings → Microsoft Defender XDR → Permissions | N/A | Required security roles visible |
| 4 | Review Defender XDR Home Dashboard | Defender Portal | Home | N/A | Dashboard displays security overview |
| 5 | Review Navigation Structure | Defender Portal | Home, Incidents & Alerts, Hunting, Secure Score, Exposure Management, Assets | N/A | Required workloads available |
| 6 | Open Microsoft Secure Score | Defender Portal | Secure Score | N/A | Secure Score dashboard loads |
| 7 | Record Current Secure Score | Defender Portal | Secure Score Overview | N/A | Current score documented |
| 8 | Review Secure Score Trend | Defender Portal | Secure Score History | N/A | Historical trend visible |
| 9 | Review Recommended Actions | Defender Portal | Secure Score → Recommended Actions | N/A | Recommendations visible |
| 10 | Sort Recommendations by Impact | Defender Portal | Sort by Points Achievable | N/A | High value actions identified |
| 11 | Review Identity Recommendations | Defender Portal | Filter by Identity | N/A | Identity actions displayed |
| 12 | Review Endpoint Recommendations | Defender Portal | Filter by Endpoint | N/A | Endpoint actions displayed |
| 13 | Review Email Recommendations | Defender Portal | Filter by Email & Collaboration | N/A | Email actions displayed |
| 14 | Open Recommendation Details | Defender Portal | Select Recommendation | N/A | Implementation guidance visible |
| 15 | Review Exposure Management Dashboard | Defender Portal | Exposure Management | N/A | Dashboard loads |
| 16 | Review Exposure Score | Defender Portal | Exposure Management Overview | N/A | Exposure score visible |
| 17 | Review Security Initiatives | Defender Portal | Exposure Management → Initiatives | N/A | Initiatives displayed |
| 18 | Review Attack Paths | Defender Portal | Exposure Management → Attack Paths | N/A | Attack path data visible |
| 19 | Review Critical Assets | Defender Portal | Exposure Management → Critical Assets | N/A | Critical assets inventory visible |
| 20 | Review Security Recommendations | Defender Portal | Exposure Management → Recommendations | N/A | Recommendations displayed |
| 21 | Review Asset Inventory | Defender Portal | Assets | N/A | Asset inventory available |
| 22 | Review User Inventory | Defender Portal | Assets → Users | N/A | User inventory available |
| 23 | Review Device Inventory | Defender Portal | Assets → Devices | N/A | Device inventory available |
| 24 | Open Incident Queue | Defender Portal | Incidents & Alerts → Incidents | N/A | Incident queue loads |
| 25 | Review Incident Status Filters | Defender Portal | Incident Filters | N/A | Active/New incidents visible |
| 26 | Review Incident Severity Filters | Defender Portal | Severity Filters | N/A | Severity filtering works |
| 27 | Open Incident Details | Defender Portal | Select Incident | N/A | Incident details displayed |
| 28 | Review Incident Story | Defender Portal | Incident Overview | N/A | Incident timeline visible |
| 29 | Review Impacted Assets | Defender Portal | Incident Assets Tab | N/A | Assets identified |
| 30 | Review Related Alerts | Defender Portal | Incident Alerts Tab | N/A | Related alerts visible |
| 31 | Add Incident Comment | Defender Portal | Incident Comments | N/A | Comment saved |
| 32 | Assign Incident Ownership | Defender Portal | Assign To Field | N/A | Incident assigned |
| 33 | Open Alert Queue | Defender Portal | Incidents & Alerts → Alerts | N/A | Alert queue loads |
| 34 | Review Alert Sources | Defender Portal | Alert List | N/A | Alert sources identified |
| 35 | Filter Alerts by Severity | Defender Portal | Alert Filters | N/A | Severity filtering works |
| 36 | Open Alert Details | Defender Portal | Select Alert | N/A | Alert details displayed |
| 37 | Review Alert Evidence | Defender Portal | Alert Evidence Tab | N/A | Evidence visible |
| 38 | Review Alert Timeline | Defender Portal | Alert Timeline | N/A | Timeline available |
| 39 | Review Alert Classification Options | Defender Portal | Classification Menu | N/A | Classification options available |
| 40 | Open Action Center | Defender Portal | Actions & Submissions → Action Center | N/A | Action Center loads |
| 41 | Review Completed Actions | Defender Portal | Action Center History | N/A | Historical actions visible |
| 42 | Review Pending Actions | Defender Portal | Pending Actions | N/A | Pending actions visible |
| 43 | Open Advanced Hunting | Defender Portal | Hunting → Advanced Hunting | N/A | Query editor loads |
| 44 | Execute Device Query | Defender Portal | Advanced Hunting | N/A | Query returns results |
| 45 | Execute Alert Query | Defender Portal | Advanced Hunting | N/A | Query returns results |
| 46 | Review Defender XDR Settings | Defender Portal | Settings → Microsoft Defender XDR | N/A | Settings accessible |
| 47 | Review Endpoint Settings | Defender Portal | Settings → Endpoints | N/A | Endpoint settings visible |
| 48 | Review Email Settings | Defender Portal | Settings → Email & Collaboration | N/A | Email settings visible |
| 49 | Review Cloud Apps Settings | Defender Portal | Settings → Cloud Apps | N/A | Cloud Apps settings visible |
| 50 | Document Baseline Findings | Admin Workstation | N/A | N/A | Baseline documented |

## Advanced Hunting Queries

### Recent Alerts

```kql
AlertInfo
| where Timestamp > ago(7d)
| project Timestamp, Title, Severity, ServiceSource, DetectionSource
| order by Timestamp desc
```

### Recent Incidents

```kql
SecurityIncident
| where CreatedTime > ago(30d)
| project IncidentName, Severity, Status, AssignedTo
| order by CreatedTime desc
```

### Recent Device Events

```kql
DeviceEvents
| where Timestamp > ago(24h)
| take 50
```

### Recent Email Events

```kql
EmailEvents
| where Timestamp > ago(7d)
| project Timestamp, SenderFromAddress, RecipientEmailAddress, Subject
| order by Timestamp desc
```

## Validation

| Check | Expected Result |
|---|---|
| Defender Portal Accessible | Success |
| Secure Score Visible | Success |
| Exposure Management Visible | Success |
| Incident Queue Accessible | Success |
| Alert Queue Accessible | Success |
| Action Center Accessible | Success |
| Advanced Hunting Accessible | Success |
| Asset Inventory Accessible | Success |
| Settings Accessible | Success |

## Troubleshooting

| Symptom | Likely Cause | Resolution |
|---|---|---|
| Portal Access Denied | Missing Role Assignment | Assign Security Administrator or Security Reader |
| Secure Score Missing | Licensing Issue | Verify Defender Licensing |
| Exposure Management Missing | Feature Not Enabled | Verify Defender Exposure Management Availability |
| No Incidents Visible | No Data Sources Connected | Validate Defender Integrations |
| No Alerts Visible | Filters Applied | Clear Filters |
| Advanced Hunting Empty | No Telemetry Available | Verify Defender Data Sources |
| Device Inventory Empty | Endpoint Not Onboarded | Onboard Defender for Endpoint Devices |
| User Inventory Empty | Identity Data Not Available | Verify Entra ID Integration |
| Action Center Empty | No Actions Performed | Generate Test Response Activity |
| Settings Missing | Insufficient Permissions | Assign Proper Security Role |

## Rollback

| Step | Action | Expected Result |
|---:|---|---|
| 1 | Remove Test Incident Assignments | Incident Returned To Original State |
| 2 | Remove Test Comments | Incident Notes Cleaned |
| 3 | Reset Filters | Default Views Restored |
| 4 | Remove Temporary Access | Elevated Permissions Removed |
| 5 | Archive Documentation | Baseline Preserved |

## Documentation Checklist

| Item | Status |
|---|---|
| Tenant Verified | ☐ |
| Secure Score Recorded | ☐ |
| Exposure Score Recorded | ☐ |
| Top Recommendations Identified | ☐ |
| Attack Paths Reviewed | ☐ |
| Critical Assets Reviewed | ☐ |
| Incidents Reviewed | ☐ |
| Alerts Reviewed | ☐ |
| Action Center Reviewed | ☐ |
| Advanced Hunting Tested | ☐ |
| Baseline Documented | ☐ |

## Notes

- Do not modify production incidents unless authorized.
- Record Secure Score before making remediation changes.
- Exposure Management findings improve as additional Defender workloads are onboarded.
- Incident assignments should follow documented SOC procedures.
- Advanced Hunting visibility depends on licensed Defender workloads and telemetry availability.
- Preserve evidence before taking response actions.
- Document all baseline values for future comparison.