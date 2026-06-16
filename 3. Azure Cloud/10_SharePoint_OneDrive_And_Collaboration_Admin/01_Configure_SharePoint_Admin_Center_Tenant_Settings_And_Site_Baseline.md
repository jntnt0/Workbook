# 01_Configure_SharePoint_Admin_Center_Tenant_Settings_And_Site_Baseline

## Objective

Configure a Microsoft 365 SharePoint Online tenant baseline using the SharePoint Admin Center and SharePoint Online PowerShell. Establish foundational settings for sites, storage, sharing, access control, synchronization, and administration before deploying collaboration workloads.

---

## Prerequisites

| Requirement | Details |
|------------|---------|
| Microsoft 365 Tenant | Active tenant with SharePoint Online licensed |
| Role Assignment | SharePoint Administrator or Global Administrator |
| SharePoint Admin Center | Accessible from Microsoft 365 Admin Center |
| PowerShell | PowerShell 7.x or Windows PowerShell 5.1 |
| SPO Module | Microsoft.Online.SharePoint.PowerShell |
| Test User | Standard licensed user account |
| Browser | Edge or Chrome |

---

## Lab Topology / Assumptions

| Component | Value |
|------------|---------|
| Tenant Name | Contoso |
| Tenant URL | https://contoso.sharepoint.com |
| Admin URL | https://contoso-admin.sharepoint.com |
| Admin Account | spadmin@contoso.onmicrosoft.com |
| Test User | user01@contoso.onmicrosoft.com |
| Hub Site | Not yet configured |
| OneDrive | Enabled |

---

# Phase 1 – Connect Administrative Tools

## Configuration Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|--------|--------|---------|------------|----------------|
| 1 | Open Microsoft 365 Admin Center | Admin Workstation | N/A | N/A | Admin portal accessible |
| 2 | Launch SharePoint Admin Center | Admin Workstation | N/A | N/A | SharePoint Admin Center opens |
| 3 | Install SPO Module | Admin Workstation | N/A | Install-Module Microsoft.Online.SharePoint.PowerShell -Scope CurrentUser | Module installed |
| 4 | Import Module | Admin Workstation | N/A | Import-Module Microsoft.Online.SharePoint.PowerShell | Module loaded |
| 5 | Connect to SharePoint Online | Admin Workstation | N/A | Connect-SPOService -Url https://contoso-admin.sharepoint.com | Successful connection |
| 6 | Verify Tenant Connectivity | Admin Workstation | N/A | Get-SPOTenant | Tenant information displayed |

---

# Phase 2 – Configure Tenant General Settings

## Configuration Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|--------|--------|---------|------------|----------------|
| 1 | Open Settings → General | SharePoint Admin Center | N/A | N/A | General settings displayed |
| 2 | Verify Organization Profile | SharePoint Admin Center | N/A | N/A | Tenant information validated |
| 3 | Configure Time Zone | SharePoint Admin Center | N/A | N/A | Correct timezone configured |
| 4 | Verify Site Creation Enabled | SharePoint Admin Center | N/A | N/A | Site creation available |
| 5 | Review Legacy Features | SharePoint Admin Center | N/A | N/A | Legacy features identified |
| 6 | Save Changes | SharePoint Admin Center | N/A | N/A | Configuration applied |

---

# Phase 3 – Configure Site Creation Baseline

## Configuration Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|--------|--------|---------|------------|----------------|
| 1 | Open Active Sites | SharePoint Admin Center | N/A | N/A | Site inventory visible |
| 2 | Review Default Site Types | SharePoint Admin Center | N/A | N/A | Site templates identified |
| 3 | Verify Team Site Availability | SharePoint Admin Center | N/A | N/A | Team Sites available |
| 4 | Verify Communication Site Availability | SharePoint Admin Center | N/A | N/A | Communication Sites available |
| 5 | Confirm Group-Connected Sites | SharePoint Admin Center | N/A | N/A | M365 integration enabled |
| 6 | Document Baseline Standards | Admin Workstation | N/A | N/A | Site governance documented |

---

# Phase 4 – Configure Storage Baseline

## Configuration Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|--------|--------|---------|------------|----------------|
| 1 | Open Settings → Storage Limit | SharePoint Admin Center | N/A | N/A | Storage settings available |
| 2 | Review Tenant Storage Allocation | SharePoint Admin Center | N/A | N/A | Total storage displayed |
| 3 | Select Automatic Site Management | SharePoint Admin Center | N/A | N/A | Auto storage enabled |
| 4 | Verify Site Quotas | SharePoint Admin Center | N/A | N/A | Quotas available |
| 5 | Review Storage Consumption | SharePoint Admin Center | N/A | N/A | Baseline established |
| 6 | Save Configuration | SharePoint Admin Center | N/A | N/A | Settings committed |

---

# Phase 5 – Configure Access Control Baseline

## Configuration Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|--------|--------|---------|------------|----------------|
| 1 | Open Policies → Access Control | SharePoint Admin Center | N/A | N/A | Access Control page displayed |
| 2 | Review Unmanaged Device Access | SharePoint Admin Center | N/A | N/A | Device policy visible |
| 3 | Select Appropriate Baseline | SharePoint Admin Center | N/A | N/A | Access policy configured |
| 4 | Review Network Location Options | SharePoint Admin Center | N/A | N/A | Network restrictions available |
| 5 | Verify Conditional Access Integration | SharePoint Admin Center | N/A | N/A | Entra integration functional |
| 6 | Save Changes | SharePoint Admin Center | N/A | N/A | Policy applied |

---

# Phase 6 – Configure Sync Baseline

## Configuration Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|--------|--------|---------|------------|----------------|
| 1 | Open Settings → Sync | SharePoint Admin Center | N/A | N/A | Sync settings displayed |
| 2 | Verify OneDrive Sync Enabled | SharePoint Admin Center | N/A | N/A | Sync permitted |
| 3 | Review Domain Restrictions | SharePoint Admin Center | N/A | N/A | Domain restrictions available |
| 4 | Configure Sync Warnings | SharePoint Admin Center | N/A | N/A | Warnings configured |
| 5 | Review Offline Access Options | SharePoint Admin Center | N/A | N/A | Offline settings verified |
| 6 | Save Configuration | SharePoint Admin Center | N/A | N/A | Settings applied |

---

# Phase 7 – Configure Site Administration Baseline

## Configuration Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|--------|--------|---------|------------|----------------|
| 1 | Open Active Sites | SharePoint Admin Center | N/A | N/A | Sites visible |
| 2 | Select Root Site | SharePoint Admin Center | N/A | N/A | Root site selected |
| 3 | Review Site Owners | SharePoint Admin Center | N/A | N/A | Ownership verified |
| 4 | Assign Secondary Admin | SharePoint Admin Center | N/A | Set-SPOUser | Backup administrator assigned |
| 5 | Verify Site Administrators | SharePoint Admin Center | N/A | Get-SPOUser | Admins displayed |
| 6 | Document Site Ownership | Admin Workstation | N/A | N/A | Ownership recorded |

---

# Phase 8 – Configure Tenant PowerShell Baseline

## Configuration Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|--------|--------|---------|------------|----------------|
| 1 | Retrieve Tenant Settings | Admin Workstation | N/A | Get-SPOTenant | Settings displayed |
| 2 | Review Sharing Settings | Admin Workstation | N/A | Get-SPOTenant \| Select SharingCapability | Sharing configuration displayed |
| 3 | Review Storage Settings | Admin Workstation | N/A | Get-SPOTenant \| Select StorageQuota | Storage information displayed |
| 4 | Review Sync Settings | Admin Workstation | N/A | Get-SPOTenant | Sync values available |
| 5 | Export Tenant Baseline | Admin Workstation | N/A | Get-SPOTenant \| Out-File .\SPOTenantBaseline.txt | Baseline exported |
| 6 | Archive Configuration | Admin Workstation | N/A | N/A | Baseline preserved |

---

# Validation Steps

| Step | Task | Device | Command | PowerShell | Expected Result |
|------|--------|--------|---------|------------|----------------|
| 1 | Verify SPO Connectivity | Admin Workstation | N/A | Get-SPOTenant | Successful output |
| 2 | Verify Tenant URL | Browser | N/A | N/A | Tenant accessible |
| 3 | Verify Admin Center Access | Browser | N/A | N/A | Admin center functional |
| 4 | Verify Storage Settings | SharePoint Admin Center | N/A | N/A | Storage baseline configured |
| 5 | Verify Access Policies | SharePoint Admin Center | N/A | N/A | Access controls active |
| 6 | Verify Sync Policies | SharePoint Admin Center | N/A | N/A | Sync configuration active |
| 7 | Verify Site Administration | SharePoint Admin Center | N/A | N/A | Admin assignments confirmed |
| 8 | Verify Exported Baseline | Admin Workstation | N/A | Test-Path .\SPOTenantBaseline.txt | File exists |

---

# Troubleshooting

| Issue | Cause | Resolution |
|---------|---------|---------|
| Cannot connect to SPO | Incorrect admin URL | Verify tenant-admin URL |
| Access denied | Missing SharePoint Admin role | Assign SharePoint Administrator role |
| Get-SPOTenant fails | SPO module missing | Install Microsoft.Online.SharePoint.PowerShell |
| Admin Center unavailable | Licensing or service issue | Verify M365 licensing |
| Site administration unavailable | Ownership issue | Reassign site administrators |
| Sync settings missing | OneDrive disabled | Verify OneDrive service health |

---

# Rollback / Cleanup

| Step | Action |
|------|---------|
| 1 | Remove test administrators |
| 2 | Remove test sites |
| 3 | Restore previous access settings |
| 4 | Restore previous sync settings |
| 5 | Remove exported baseline files if required |
| 6 | Disconnect PowerShell session |

```powershell
Disconnect-SPOService
```

---

# Related Workbooks

```text
02_Manage_SharePoint_Sites_Hub_Sites_Permissions_And_Sharing.md
03_Configure_OneDrive_Admin_Settings_Sync_Sharing_And_Retention_Baseline.md
04_Configure_External_Sharing_Guest_Access_Link_Settings_And_Access_Reviews.md
05_Manage_Site_Storage_Versioning_Recycle_Bin_And_Restore.md
06_Configure_Sensitivity_Labels_DLP_And_Information_Protection_For_Files.md
07_Troubleshoot_SharePoint_OneDrive_Sync_Sharing_Permissions_And_Restore_Issues.md
```