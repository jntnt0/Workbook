06_Manage_M365_Licensing_Service_Plans_Group_Based_Licensing_And_Usage.md
# 06_Manage_M365_Licensing_Service_Plans_Group_Based_Licensing_And_Usage

# 06_Manage_M365_Licensing_Service_Plans_Group_Based_Licensing_And_Usage_Index
06_Manage_M365_Licensing_Service_Plans_Group_Based_Licensing_And_Usage.md
06_Manage_M365_Licensing_Service_Plans_Group_Based_Licensing_And_Usage
06_Manage_M365_Licensing_Service_Plans_Group_Based_Licensing_And_Usage_Source_Basis
06_Manage_M365_Licensing_Service_Plans_Group_Based_Licensing_And_Usage_Mental_Model
06_Manage_M365_Licensing_Service_Plans_Group_Based_Licensing_And_Usage_Planning_Table
06_Manage_M365_Licensing_Service_Plans_Group_Based_Licensing_And_Usage_Configuration_Checklist
06_Manage_M365_Licensing_Service_Plans_Group_Based_Licensing_And_Usage_Tenant_License_Inventory_Skeleton
06_Manage_M365_Licensing_Service_Plans_Group_Based_Licensing_And_Usage_Direct_User_Licensing_Skeleton
06_Manage_M365_Licensing_Service_Plans_Group_Based_Licensing_And_Usage_Service_Plan_Control_Skeleton
06_Manage_M365_Licensing_Service_Plans_Group_Based_Licensing_And_Usage_Group_Based_Licensing_Skeleton
06_Manage_M365_Licensing_Service_Plans_Group_Based_Licensing_And_Usage_Bulk_Licensing_Skeleton
06_Manage_M365_Licensing_Service_Plans_Group_Based_Licensing_And_Usage_Usage_And_Cleanup_Skeleton
06_Manage_M365_Licensing_Service_Plans_Group_Based_Licensing_And_Usage_Verification_Commands
06_Manage_M365_Licensing_Service_Plans_Group_Based_Licensing_And_Usage_Rollback
06_Manage_M365_Licensing_Service_Plans_Group_Based_Licensing_And_Usage_Failure_Checks
06_Manage_M365_Licensing_Service_Plans_Group_Based_Licensing_And_Usage_Related_Labs

# 06_Manage_M365_Licensing_Service_Plans_Group_Based_Licensing_And_Usage_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Assign licenses to users in Microsoft 365 admin center | Direct license assignment, unassignment, service app toggles, reprocessing, and user license state |
| Microsoft Learn | Manage group-based licenses in Microsoft 365 admin center | Assigning licenses to groups, group license errors, usage location, nested group limitation, and membership processing |
| Microsoft Learn | Microsoft Graph PowerShell licensing cmdlets | Repeatable tenant SKU inventory, direct user licensing, group licensing, service plan control, and license detail checks |
| Microsoft Learn | Microsoft 365 usage reports | Active user and workload usage review for license cleanup |
| Microsoft Learn | Microsoft 365 admin roles | License Administrator, User Administrator, Groups Administrator, and least-privilege role planning |
| Microsoft Learn | Microsoft 365 admin center Billing area | Subscription inventory, license availability, purchased units, assigned units, and billing ownership |
| Microsoft 365 operational practice | License governance | Standardizes license request, assignment, removal, group-based licensing, and cost cleanup workflow |
| Identity operational practice | Usage location and source of authority | Prevents failed license assignment caused by missing usage location or wrong object management path |

# 06_Manage_M365_Licensing_Service_Plans_Group_Based_Licensing_And_Usage_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Subscription | Purchased Microsoft 365 product such as Business Premium, E3, E5, Exchange Online, Teams, or add-on license |
| SKU | Technical product identifier for a subscription, such as `SPE_E3`, `SPE_E5`, or `O365_BUSINESS_PREMIUM` |
| Service plan | Individual workload feature inside a SKU, such as Exchange Online, SharePoint, Teams, Intune, or Entra features |
| License assignment | Applying a SKU to a user or group |
| Direct licensing | License assigned directly to a user object |
| Group-based licensing | License assigned to a group; users inherit the license through group membership |
| Disabled plan | Service plan intentionally turned off inside an assigned SKU |
| Usage location | User property required before assigning many Microsoft cloud licenses |
| Consumed units | Number of licenses currently assigned or consumed |
| Prepaid units | Number of purchased license units available in the subscription |
| License error | Assignment failure caused by missing usage location, conflicting plans, no available licenses, unsupported group type, or service dependency |
| Reprocess | Admin action to retry failed group-based license processing |
| License cleanup | Review and removal of unused, duplicate, stale, or over-assigned licenses |
| Source of authority | Whether user/group properties are cloud-managed or synced from on-premises |
| First rule | Set usage location before assigning licenses |
| Blunt rule | Use group-based licensing for scale; use direct licensing only for exceptions or small one-off cases |

# 06_Manage_M365_Licensing_Service_Plans_Group_Based_Licensing_And_Usage_Planning_Table
| Item | Example | Decision |
|---|---|---|
| Tenant name | `contoso.onmicrosoft.com` | `<tenant>.onmicrosoft.com` |
| Tenant ID | `00000000-0000-0000-0000-000000000000` | `<tenant-id>` |
| Admin account | `admin@contoso.com` | `<admin-upn>` |
| Licensing admin role | License Administrator | `<license-admin-role>` |
| User admin role | User Administrator | `<user-admin-role>` |
| Group admin role | Groups Administrator | `<group-admin-role>` |
| Billing owner | `billingadmin@contoso.com` | `<billing-owner>` |
| Default usage location | `US` | `<usage-location>` |
| Primary license SKU | `SPE_E3` | `<primary-sku-part-number>` |
| Add-on license SKU | `POWER_BI_PRO` | `<addon-sku-part-number>` |
| Baseline license group | `SG_LIC_M365_E3` | `<baseline-license-group>` |
| Add-on license group | `SG_LIC_PowerBI_Pro` | `<addon-license-group>` |
| Exception direct-license users | `executive@contoso.com` | `<exception-users>` |
| License assignment model | Group-based licensing | `<direct/group-based/hybrid>` |
| Service plans disabled by default | Yammer, Sway, or other unused service | `<disabled-service-plans>` |
| License request process | Help desk ticket with manager approval | `<request-process>` |
| License cleanup cadence | Monthly | `<cleanup-cadence>` |
| Usage report window | 90 days | `<usage-report-window>` |
| Evidence path | `.\Evidence\M365-Licensing` | `<evidence-path>` |
| Bulk user CSV path | `.\Input\license-users.csv` | `<bulk-user-csv>` |
| Bulk group membership CSV path | `.\Input\license-group-members.csv` | `<bulk-membership-csv>` |
| Rollback owner | `licenseadmin@contoso.com` | `<rollback-owner>` |

# 06_Manage_M365_Licensing_Service_Plans_Group_Based_Licensing_And_Usage_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm admin account | Admin Workstation | Portal account menu | Correct admin UPN is signed in |
| 2 | Confirm correct tenant | Admin Workstation | Portal tenant switcher | Correct Microsoft 365 tenant is active |
| 3 | Confirm licensing permissions | Admin Workstation | Microsoft 365 admin center > Roles | Admin has License Administrator or equivalent role |
| 4 | Open licensing area | Admin Workstation | Microsoft 365 admin center > Billing > Licenses | Tenant subscription and license inventory is visible |
| 5 | Record purchased license SKUs | Admin Workstation | Billing > Licenses | Purchased products are documented |
| 6 | Record assigned and available license counts | Admin Workstation | Billing > Licenses | Consumed and available units are known |
| 7 | Open Active users license view | Admin Workstation | Users > Active users | User license state is visible |
| 8 | Review unlicensed users | Admin Workstation | Users > Active users > Filter unlicensed users | Users missing licenses are identified |
| 9 | Review licensed users | Admin Workstation | Users > Active users | Licensed users are identified |
| 10 | Review usage reports before cleanup | Admin Workstation | Reports > Usage | Activity context is available before removing licenses |
| 11 | Connect Microsoft Graph PowerShell | Admin Workstation | `Connect-MgGraph -Scopes "User.ReadWrite.All","Group.ReadWrite.All","Directory.ReadWrite.All","Organization.Read.All"` | Graph session connects |
| 12 | Confirm Graph context | Admin Workstation | `Get-MgContext` | Tenant ID and scopes are visible |
| 13 | Export subscribed SKUs | Admin Workstation | `Get-MgSubscribedSku` | SKU inventory is exported |
| 14 | Export user license details | Admin Workstation | `Get-MgUserLicenseDetail` | User license details are available |
| 15 | Confirm user usage location | Admin Workstation | `Get-MgUser -Property UsageLocation` | Usage location is populated |
| 16 | Set usage location if missing | Admin Workstation | `Update-MgUser -UsageLocation "<usage-location>"` | User becomes license-ready |
| 17 | Assign direct license only if approved | Admin Workstation | `Set-MgUserLicense` | User receives license directly |
| 18 | Disable service plans if required | Admin Workstation | `Set-MgUserLicense -AddLicenses @{DisabledPlans=...}` | SKU is assigned with selected plans disabled |
| 19 | Validate direct license | Admin Workstation | `Get-MgUserLicenseDetail -UserId "<user-upn>"` | License appears on user |
| 20 | Remove direct license if no longer needed | Admin Workstation | `Set-MgUserLicense -RemoveLicenses` | Direct license is removed |
| 21 | Create license security group | Admin Workstation | `New-MgGroup` | License assignment group exists |
| 22 | Add users to license group | Admin Workstation | `New-MgGroupMemberByRef` | Users are members of license group |
| 23 | Assign license to group | Admin Workstation | `Set-MgGroupLicense` | Group receives target license |
| 24 | Validate group license detail | Admin Workstation | `Get-MgGroupLicenseDetail` | Group license assignment is visible |
| 25 | Validate inherited user license | Admin Workstation | `Get-MgUserLicenseDetail` | User receives license through group membership |
| 26 | Review group-based license errors | Admin Workstation | Microsoft 365 admin center > Billing > Licenses or group license view | Assignment errors are visible |
| 27 | Fix missing usage location errors | Admin Workstation | `Update-MgUser -UsageLocation "<usage-location>"` | User can receive license |
| 28 | Fix no available licenses issue | Admin Workstation | Billing > Licenses | License availability issue is escalated to billing owner |
| 29 | Fix conflicting plan issue | Admin Workstation | License details / service plans | Conflicting service plan assignments are corrected |
| 30 | Reprocess group license assignment if needed | Admin Workstation | Admin center license group error view | Group license processing is retried |
| 31 | Bulk assign usage location | Admin Workstation | Import CSV skeleton | Multiple users receive usage location |
| 32 | Bulk add users to license groups | Admin Workstation | Import CSV skeleton | Multiple users inherit group-based licenses |
| 33 | Bulk remove users from license groups | Admin Workstation | Import CSV skeleton | Multiple users lose inherited license after processing |
| 34 | Export final SKU inventory | Admin Workstation | `Get-MgSubscribedSku` | Final license counts are saved |
| 35 | Export final user license inventory | Admin Workstation | Graph export commands | Final user license state is saved |
| 36 | Export final group license inventory | Admin Workstation | Graph export commands | Group license state is saved |
| 37 | Review unused license candidates | Admin Workstation | Reports > Usage / Graph exports | Cleanup candidates are identified |
| 38 | Confirm business approval before license removal | Operator | Ticket / approval notes | Removal is authorized |
| 39 | Remove unused direct licenses | Admin Workstation | `Set-MgUserLicense -RemoveLicenses` | Unused direct licenses are reclaimed |
| 40 | Remove users from license groups if approved | Admin Workstation | `Remove-MgGroupMemberByRef` | Group-based license is removed through membership |
| 41 | Validate service access after assignment | Test User | Sign in to Microsoft 365 services | User can access expected services |
| 42 | Validate disabled service plans | Test User | Service portal access test | Disabled workloads are unavailable |
| 43 | Validate Exchange mailbox provisioning if Exchange plan enabled | Admin Workstation | Exchange admin center / `Get-EXOMailbox` | Mailbox exists after provisioning |
| 44 | Validate Teams access if Teams plan enabled | Test User | Teams web or desktop | Teams access works |
| 45 | Validate OneDrive provisioning if SharePoint plan enabled | Test User | OneDrive web | OneDrive access works after provisioning |
| 46 | Document license model | Operator | Notes | Direct, group-based, or hybrid model is recorded |
| 47 | Document service plan decisions | Operator | Notes | Disabled service plans and reason are recorded |
| 48 | Document cleanup cadence | Operator | Notes | License review rhythm is recorded |
| 49 | Save evidence | Admin Workstation | `<evidence-path>` | License exports and screenshots are saved |
| 50 | Document completion state | Operator | Notes | Licensing, service plans, group licensing, and usage review baseline is complete |

# 06_Manage_M365_Licensing_Service_Plans_Group_Based_Licensing_And_Usage_Tenant_License_Inventory_Skeleton
```powershell
# Run from an admin workstation.
# Purpose: inventory tenant SKUs, available license counts, and service plans.

$EvidencePath = ".\Evidence\M365-Licensing"
New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Connect-MgGraph -Scopes "Organization.Read.All","Directory.Read.All","User.Read.All","Group.Read.All"

Get-MgContext |
    Tee-Object "$EvidencePath\graph-context-licensing.txt"

# Tenant SKU summary.
Get-MgSubscribedSku |
    Select-Object `
        SkuPartNumber,
        SkuId,
        ConsumedUnits,
        @{Name="EnabledUnits";Expression={$_.PrepaidUnits.Enabled}},
        @{Name="SuspendedUnits";Expression={$_.PrepaidUnits.Suspended}},
        @{Name="WarningUnits";Expression={$_.PrepaidUnits.Warning}} |
    Sort-Object SkuPartNumber |
    Format-Table -AutoSize |
    Tee-Object "$EvidencePath\subscribed-skus-summary.txt"

# Full SKU and service plan detail.
Get-MgSubscribedSku |
    ConvertTo-Json -Depth 20 |
    Out-File "$EvidencePath\subscribed-skus-full.json"

# Service plan list by SKU.
$SkuPlanRows = foreach ($Sku in Get-MgSubscribedSku) {
    foreach ($Plan in $Sku.ServicePlans) {
        [PSCustomObject]@{
            SkuPartNumber = $Sku.SkuPartNumber
            SkuId = $Sku.SkuId
            ServicePlanName = $Plan.ServicePlanName
            ServicePlanId = $Plan.ServicePlanId
            ProvisioningStatus = $Plan.ProvisioningStatus
            AppliesTo = $Plan.AppliesTo
        }
    }
}

$SkuPlanRows |
    Sort-Object SkuPartNumber,ServicePlanName |
    Export-Csv "$EvidencePath\sku-service-plans.csv" -NoTypeInformation

$SkuPlanRows |
    Sort-Object SkuPartNumber,ServicePlanName |
    Format-Table -AutoSize
```

License inventory interpretation:

| Field | Meaning |
|---|---|
| `SkuPartNumber` | Friendly product SKU identifier used in scripts and reports |
| `SkuId` | GUID required when assigning or removing licenses |
| `ConsumedUnits` | Licenses currently consumed |
| `PrepaidUnits.Enabled` | Purchased usable license quantity |
| `ServicePlanName` | Individual workload feature inside the SKU |
| `ServicePlanId` | GUID used when disabling service plans |
| `ProvisioningStatus` | Service plan availability state |
| `AppliesTo` | Whether the service plan applies to users or company level features |

Operational note:

```text
Always capture SKU inventory before assigning or removing licenses.
Do not assume the tenant has the same SKU names as another tenant.
```

# 06_Manage_M365_Licensing_Service_Plans_Group_Based_Licensing_And_Usage_Direct_User_Licensing_Skeleton
Direct license assignment should be used for small exceptions, testing, or one-off users.
For standard access patterns, prefer group-based licensing.

Set usage location:

```powershell
# Run from an admin workstation.
# Purpose: set usage location before direct license assignment.

Connect-MgGraph -Scopes "User.ReadWrite.All","Directory.ReadWrite.All","Organization.Read.All"

$UserPrincipalName = "<user-upn>"
$UsageLocation = "US"

Update-MgUser `
    -UserId $UserPrincipalName `
    -UsageLocation $UsageLocation

Get-MgUser `
    -UserId $UserPrincipalName `
    -Property Id,DisplayName,UserPrincipalName,UsageLocation |
    Format-List Id,DisplayName,UserPrincipalName,UsageLocation
```

Assign direct license:

```powershell
# Purpose: assign a license directly to a user.

$UserPrincipalName = "<user-upn>"
$SkuPartNumber = "<sku-part-number>"

$Sku = Get-MgSubscribedSku |
    Where-Object {$_.SkuPartNumber -eq $SkuPartNumber}

Set-MgUserLicense `
    -UserId $UserPrincipalName `
    -AddLicenses @(
        @{
            SkuId = $Sku.SkuId
            DisabledPlans = @()
        }
    ) `
    -RemoveLicenses @()

Get-MgUserLicenseDetail -UserId $UserPrincipalName |
    Select-Object SkuPartNumber,SkuId |
    Format-Table -AutoSize
```

Remove direct license:

```powershell
# Purpose: remove a directly assigned license from a user.

$UserPrincipalName = "<user-upn>"
$SkuPartNumber = "<sku-part-number>"

$Sku = Get-MgSubscribedSku |
    Where-Object {$_.SkuPartNumber -eq $SkuPartNumber}

Set-MgUserLicense `
    -UserId $UserPrincipalName `
    -AddLicenses @() `
    -RemoveLicenses @($Sku.SkuId)

Get-MgUserLicenseDetail -UserId $UserPrincipalName |
    Select-Object SkuPartNumber,SkuId |
    Format-Table -AutoSize
```

Validate user license detail:

```powershell
$UserPrincipalName = "<user-upn>"

Get-MgUser -UserId $UserPrincipalName -Property Id,DisplayName,UserPrincipalName,AccountEnabled,UsageLocation,AssignedLicenses |
    Format-List Id,DisplayName,UserPrincipalName,AccountEnabled,UsageLocation,AssignedLicenses

Get-MgUserLicenseDetail -UserId $UserPrincipalName |
    Format-List SkuPartNumber,SkuId,ServicePlans
```

Operational note:

```text
Direct licensing is easy to apply but hard to govern at scale.
Use it for exceptions, then move standard users into group-based licensing.
```

# 06_Manage_M365_Licensing_Service_Plans_Group_Based_Licensing_And_Usage_Service_Plan_Control_Skeleton
Use this when assigning a SKU but disabling selected workloads inside that SKU.

Find service plan IDs:

```powershell
# Purpose: find service plan IDs inside a SKU.

$SkuPartNumber = "<sku-part-number>"

$Sku = Get-MgSubscribedSku |
    Where-Object {$_.SkuPartNumber -eq $SkuPartNumber}

$Sku.ServicePlans |
    Sort-Object ServicePlanName |
    Format-Table ServicePlanName,ServicePlanId,ProvisioningStatus,AppliesTo -AutoSize
```

Assign license with disabled service plans:

```powershell
# Purpose: assign a SKU while disabling selected service plans.

$UserPrincipalName = "<user-upn>"
$SkuPartNumber = "<sku-part-number>"
$DisabledServicePlanNames = @(
    "<service-plan-name-1>",
    "<service-plan-name-2>"
)

$Sku = Get-MgSubscribedSku |
    Where-Object {$_.SkuPartNumber -eq $SkuPartNumber}

$DisabledPlanIds = $Sku.ServicePlans |
    Where-Object {$_.ServicePlanName -in $DisabledServicePlanNames} |
    Select-Object -ExpandProperty ServicePlanId

Set-MgUserLicense `
    -UserId $UserPrincipalName `
    -AddLicenses @(
        @{
            SkuId = $Sku.SkuId
            DisabledPlans = $DisabledPlanIds
        }
    ) `
    -RemoveLicenses @()

Get-MgUserLicenseDetail -UserId $UserPrincipalName |
    Where-Object {$_.SkuId -eq $Sku.SkuId} |
    Format-List SkuPartNumber,ServicePlans
```

Update disabled plans for an already assigned license:

```powershell
# Purpose: replace the disabled plan list for a user's assigned SKU.

$UserPrincipalName = "<user-upn>"
$SkuPartNumber = "<sku-part-number>"
$DisabledServicePlanNames = @(
    "<service-plan-name-1>",
    "<service-plan-name-2>"
)

$Sku = Get-MgSubscribedSku |
    Where-Object {$_.SkuPartNumber -eq $SkuPartNumber}

$DisabledPlanIds = $Sku.ServicePlans |
    Where-Object {$_.ServicePlanName -in $DisabledServicePlanNames} |
    Select-Object -ExpandProperty ServicePlanId

Set-MgUserLicense `
    -UserId $UserPrincipalName `
    -AddLicenses @(
        @{
            SkuId = $Sku.SkuId
            DisabledPlans = $DisabledPlanIds
        }
    ) `
    -RemoveLicenses @()
```

Service plan decision table:

| SKU | Service Plan | Enabled | Reason |
|---|---|---:|---|
| `<sku-part-number>` | `<service-plan-name>` | `<yes-no>` | `<business-reason>` |
| `<sku-part-number>` | `<service-plan-name>` | `<yes-no>` | `<business-reason>` |

Operational note:

```text
Disabling service plans is not the same as removing the license.
The user still consumes the SKU, but selected workloads inside the SKU are disabled.
```

# 06_Manage_M365_Licensing_Service_Plans_Group_Based_Licensing_And_Usage_Group_Based_Licensing_Skeleton
Group-based licensing should be the default model for repeatable license assignment.

License group planning:

| Field | Example | Decision |
|---|---|---|
| Group display name | `SG_LIC_M365_E3` | `<license-group-name>` |
| Group mail nickname | `sg-lic-m365-e3` | `<mailnickname>` |
| SKU assigned | `SPE_E3` | `<sku-part-number>` |
| Disabled service plans | `<none>` | `<disabled-service-plans>` |
| Membership source | Manual / dynamic / synced | `<membership-source>` |
| Group owner | `licenseadmin@contoso.com` | `<owner>` |
| Users in scope | Standard office workers | `<scope>` |
| Excluded users | Contractors, service accounts | `<excluded-users>` |

Create license security group:

```powershell
# Run from an admin workstation.
# Purpose: create a cloud security group used for group-based licensing.

Connect-MgGraph -Scopes "Group.ReadWrite.All","User.Read.All","Directory.ReadWrite.All","Organization.Read.All"

$GroupDisplayName = "SG_LIC_M365_E3"
$MailNickname = "sg-lic-m365-e3"

$Group = New-MgGroup `
    -DisplayName $GroupDisplayName `
    -Description "Group-based licensing assignment for Microsoft 365 E3" `
    -MailEnabled:$false `
    -MailNickname $MailNickname `
    -SecurityEnabled:$true

Get-MgGroup -GroupId $Group.Id |
    Format-List Id,DisplayName,MailEnabled,SecurityEnabled,MailNickname
```

Add user to license group:

```powershell
# Purpose: add a user to a license group.

$GroupDisplayName = "SG_LIC_M365_E3"
$UserPrincipalName = "<user-upn>"

$Group = Get-MgGroup -Filter "displayName eq '$GroupDisplayName'"
$User = Get-MgUser -UserId $UserPrincipalName

New-MgGroupMemberByRef `
    -GroupId $Group.Id `
    -BodyParameter @{
        "@odata.id" = "https://graph.microsoft.com/v1.0/directoryObjects/$($User.Id)"
    }

Get-MgGroupMember -GroupId $Group.Id |
    Select-Object Id,AdditionalProperties |
    Format-Table -AutoSize
```

Assign license to group:

```powershell
# Purpose: assign a SKU to a licensing group.

$GroupDisplayName = "SG_LIC_M365_E3"
$SkuPartNumber = "<sku-part-number>"

$Group = Get-MgGroup -Filter "displayName eq '$GroupDisplayName'"
$Sku = Get-MgSubscribedSku |
    Where-Object {$_.SkuPartNumber -eq $SkuPartNumber}

Set-MgGroupLicense `
    -GroupId $Group.Id `
    -AddLicenses @(
        @{
            SkuId = $Sku.SkuId
            DisabledPlans = @()
        }
    ) `
    -RemoveLicenses @()

Get-MgGroupLicenseDetail -GroupId $Group.Id |
    Format-List SkuPartNumber,SkuId,ServicePlans
```

Assign group license with disabled service plans:

```powershell
# Purpose: assign a SKU to a group while disabling selected service plans.

$GroupDisplayName = "SG_LIC_M365_E3"
$SkuPartNumber = "<sku-part-number>"
$DisabledServicePlanNames = @(
    "<service-plan-name-1>",
    "<service-plan-name-2>"
)

$Group = Get-MgGroup -Filter "displayName eq '$GroupDisplayName'"
$Sku = Get-MgSubscribedSku |
    Where-Object {$_.SkuPartNumber -eq $SkuPartNumber}

$DisabledPlanIds = $Sku.ServicePlans |
    Where-Object {$_.ServicePlanName -in $DisabledServicePlanNames} |
    Select-Object -ExpandProperty ServicePlanId

Set-MgGroupLicense `
    -GroupId $Group.Id `
    -AddLicenses @(
        @{
            SkuId = $Sku.SkuId
            DisabledPlans = $DisabledPlanIds
        }
    ) `
    -RemoveLicenses @()

Get-MgGroupLicenseDetail -GroupId $Group.Id |
    Format-List SkuPartNumber,SkuId,ServicePlans
```

Remove group license:

```powershell
# Purpose: remove a SKU assignment from a licensing group.
# This removes inherited licensing from users that only receive the SKU through this group.

$GroupDisplayName = "SG_LIC_M365_E3"
$SkuPartNumber = "<sku-part-number>"

$Group = Get-MgGroup -Filter "displayName eq '$GroupDisplayName'"
$Sku = Get-MgSubscribedSku |
    Where-Object {$_.SkuPartNumber -eq $SkuPartNumber}

Set-MgGroupLicense `
    -GroupId $Group.Id `
    -AddLicenses @() `
    -RemoveLicenses @($Sku.SkuId)

Get-MgGroupLicenseDetail -GroupId $Group.Id
```

Validate inherited license on user:

```powershell
$UserPrincipalName = "<user-upn>"

Get-MgUserLicenseDetail -UserId $UserPrincipalName |
    Select-Object SkuPartNumber,SkuId |
    Format-Table -AutoSize

Get-MgUser -UserId $UserPrincipalName -Property Id,DisplayName,UserPrincipalName,AssignedLicenses,UsageLocation |
    Format-List Id,DisplayName,UserPrincipalName,UsageLocation,AssignedLicenses
```

Operational note:

```text
Group-based licensing does not process nested group membership.
Put users directly in the licensing group or use a supported dynamic group design.
```

# 06_Manage_M365_Licensing_Service_Plans_Group_Based_Licensing_And_Usage_Bulk_Licensing_Skeleton
Bulk user usage location CSV format:

```csv
UserPrincipalName,UsageLocation
john.smith@contoso.com,US
sara.jones@contoso.com,US
```

Bulk set usage location:

```powershell
# Run from an admin workstation.
# Purpose: set usage location for multiple users before license assignment.

$CsvPath = ".\Input\license-users.csv"
$EvidencePath = ".\Evidence\M365-Licensing"

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Connect-MgGraph -Scopes "User.ReadWrite.All","Directory.ReadWrite.All"

$Rows = Import-Csv $CsvPath
$Results = @()

foreach ($Row in $Rows) {
    try {
        Update-MgUser `
            -UserId $Row.UserPrincipalName `
            -UsageLocation $Row.UsageLocation

        $Results += [PSCustomObject]@{
            UserPrincipalName = $Row.UserPrincipalName
            UsageLocation = $Row.UsageLocation
            Status = "Updated"
            Error = ""
        }
    }
    catch {
        $Results += [PSCustomObject]@{
            UserPrincipalName = $Row.UserPrincipalName
            UsageLocation = $Row.UsageLocation
            Status = "Failed"
            Error = $_.Exception.Message
        }
    }
}

$Results |
    Export-Csv "$EvidencePath\bulk-usage-location-results.csv" -NoTypeInformation

$Results |
    Format-Table -AutoSize
```

Bulk license group membership CSV format:

```csv
GroupDisplayName,UserPrincipalName,Action
SG_LIC_M365_E3,john.smith@contoso.com,Add
SG_LIC_M365_E3,sara.jones@contoso.com,Add
SG_LIC_M365_E3,old.user@contoso.com,Remove
```

Bulk add or remove users from license groups:

```powershell
# Purpose: bulk add or remove users from licensing groups.

$CsvPath = ".\Input\license-group-members.csv"
$EvidencePath = ".\Evidence\M365-Licensing"

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Connect-MgGraph -Scopes "Group.ReadWrite.All","User.Read.All","Directory.ReadWrite.All"

$Rows = Import-Csv $CsvPath
$Results = @()

foreach ($Row in $Rows) {
    try {
        $Group = Get-MgGroup -Filter "displayName eq '$($Row.GroupDisplayName)'"
        $User = Get-MgUser -UserId $Row.UserPrincipalName

        if ($Row.Action -eq "Add") {
            New-MgGroupMemberByRef `
                -GroupId $Group.Id `
                -BodyParameter @{
                    "@odata.id" = "https://graph.microsoft.com/v1.0/directoryObjects/$($User.Id)"
                }

            $Status = "Added"
        }

        if ($Row.Action -eq "Remove") {
            Remove-MgGroupMemberByRef `
                -GroupId $Group.Id `
                -DirectoryObjectId $User.Id

            $Status = "Removed"
        }

        $Results += [PSCustomObject]@{
            GroupDisplayName = $Row.GroupDisplayName
            UserPrincipalName = $Row.UserPrincipalName
            Action = $Row.Action
            Status = $Status
            Error = ""
        }
    }
    catch {
        $Results += [PSCustomObject]@{
            GroupDisplayName = $Row.GroupDisplayName
            UserPrincipalName = $Row.UserPrincipalName
            Action = $Row.Action
            Status = "Failed"
            Error = $_.Exception.Message
        }
    }
}

$Results |
    Export-Csv "$EvidencePath\bulk-license-group-membership-results.csv" -NoTypeInformation

$Results |
    Format-Table -AutoSize
```

Bulk direct license assignment CSV format:

```csv
UserPrincipalName,SkuPartNumber,Action
john.smith@contoso.com,SPE_E3,Add
sara.jones@contoso.com,SPE_E3,Add
old.user@contoso.com,SPE_E3,Remove
```

Bulk direct licensing:

```powershell
# Purpose: bulk assign or remove direct user licenses.
# Use for exception handling, not preferred baseline.

$CsvPath = ".\Input\direct-license-users.csv"
$EvidencePath = ".\Evidence\M365-Licensing"

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Connect-MgGraph -Scopes "User.ReadWrite.All","Directory.ReadWrite.All","Organization.Read.All"

$Rows = Import-Csv $CsvPath
$Results = @()

foreach ($Row in $Rows) {
    try {
        $Sku = Get-MgSubscribedSku |
            Where-Object {$_.SkuPartNumber -eq $Row.SkuPartNumber}

        if ($Row.Action -eq "Add") {
            Set-MgUserLicense `
                -UserId $Row.UserPrincipalName `
                -AddLicenses @(
                    @{
                        SkuId = $Sku.SkuId
                        DisabledPlans = @()
                    }
                ) `
                -RemoveLicenses @()

            $Status = "Assigned"
        }

        if ($Row.Action -eq "Remove") {
            Set-MgUserLicense `
                -UserId $Row.UserPrincipalName `
                -AddLicenses @() `
                -RemoveLicenses @($Sku.SkuId)

            $Status = "Removed"
        }

        $Results += [PSCustomObject]@{
            UserPrincipalName = $Row.UserPrincipalName
            SkuPartNumber = $Row.SkuPartNumber
            Action = $Row.Action
            Status = $Status
            Error = ""
        }
    }
    catch {
        $Results += [PSCustomObject]@{
            UserPrincipalName = $Row.UserPrincipalName
            SkuPartNumber = $Row.SkuPartNumber
            Action = $Row.Action
            Status = "Failed"
            Error = $_.Exception.Message
        }
    }
}

$Results |
    Export-Csv "$EvidencePath\bulk-direct-license-results.csv" -NoTypeInformation

$Results |
    Format-Table -AutoSize
```

Operational note:

```text
For production, bulk add users to licensing groups instead of bulk assigning direct licenses.
Direct license CSVs are for exceptions, migrations, and controlled cleanup.
```

# 06_Manage_M365_Licensing_Service_Plans_Group_Based_Licensing_And_Usage_Usage_And_Cleanup_Skeleton
```text
Portal path:

Microsoft 365 admin center
Reports
Usage
```

License cleanup review areas:

| Area | What To Review | Action |
|---|---|---|
| Active users | Users with no recent activity | Review whether license is still needed |
| Exchange activity | Mailbox activity | Identify inactive mail users |
| Teams activity | Teams usage | Identify users not using Teams |
| OneDrive activity | File activity and storage | Identify inactive OneDrive users |
| SharePoint activity | Site activity | Identify adoption and stale use |
| Microsoft 365 Apps | App activation and usage | Identify users not using desktop apps |
| License counts | Purchased vs consumed | Identify shortage or overbuy risk |
| Group licensing | Membership and inherited assignments | Identify incorrect group scope |
| Direct licensing | Exceptions | Reduce direct assignments where possible |

Export user license and usage cleanup baseline:

```powershell
# Run from an admin workstation.
# Purpose: export user licensing data for cleanup review.

$EvidencePath = ".\Evidence\M365-Licensing"
New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Connect-MgGraph -Scopes "User.Read.All","Directory.Read.All","Reports.Read.All","Organization.Read.All"

$Users = Get-MgUser -All -Property Id,DisplayName,UserPrincipalName,AccountEnabled,UsageLocation,AssignedLicenses,CreatedDateTime

$UserLicenseRows = foreach ($User in $Users) {
    $LicenseDetails = Get-MgUserLicenseDetail -UserId $User.Id -ErrorAction SilentlyContinue

    foreach ($License in $LicenseDetails) {
        [PSCustomObject]@{
            DisplayName = $User.DisplayName
            UserPrincipalName = $User.UserPrincipalName
            AccountEnabled = $User.AccountEnabled
            UsageLocation = $User.UsageLocation
            SkuPartNumber = $License.SkuPartNumber
            SkuId = $License.SkuId
        }
    }
}

$UserLicenseRows |
    Sort-Object UserPrincipalName,SkuPartNumber |
    Export-Csv "$EvidencePath\user-license-inventory.csv" -NoTypeInformation

$UserLicenseRows |
    Sort-Object UserPrincipalName,SkuPartNumber |
    Format-Table -AutoSize
```

Cleanup candidate table:

| User | License | Reason Flagged | Last Activity Evidence | Approved For Removal | Removal Method |
|---|---|---|---|---:|---|
| `<user-upn>` | `<sku-part-number>` | `<inactive/no-longer-needed/duplicate>` | `<report-evidence>` | `<yes-no>` | `<direct-remove/group-remove>` |

License cleanup rule:

```text
Never remove licenses only because a report looks quiet.
Confirm user status, business owner approval, mailbox/data impact, and whether the license is inherited from a group.
```

# 06_Manage_M365_Licensing_Service_Plans_Group_Based_Licensing_And_Usage_Verification_Commands
```powershell
# Tenant SKU verification.

Connect-MgGraph -Scopes "Organization.Read.All","Directory.Read.All","User.Read.All","Group.Read.All"

Get-MgContext

Get-MgSubscribedSku |
    Select-Object `
        SkuPartNumber,
        SkuId,
        ConsumedUnits,
        @{Name="EnabledUnits";Expression={$_.PrepaidUnits.Enabled}} |
    Sort-Object SkuPartNumber |
    Format-Table -AutoSize
```

```powershell
# User license verification.

$UserPrincipalName = "<user-upn>"

Get-MgUser `
    -UserId $UserPrincipalName `
    -Property Id,DisplayName,UserPrincipalName,AccountEnabled,UsageLocation,AssignedLicenses |
    Format-List Id,DisplayName,UserPrincipalName,AccountEnabled,UsageLocation,AssignedLicenses

Get-MgUserLicenseDetail -UserId $UserPrincipalName |
    Format-List SkuPartNumber,SkuId,ServicePlans
```

```powershell
# Group license verification.

$GroupDisplayName = "<license-group-name>"

$Group = Get-MgGroup -Filter "displayName eq '$GroupDisplayName'"

Get-MgGroup -GroupId $Group.Id |
    Format-List Id,DisplayName,MailEnabled,SecurityEnabled,GroupTypes

Get-MgGroupLicenseDetail -GroupId $Group.Id |
    Format-List SkuPartNumber,SkuId,ServicePlans

Get-MgGroupMember -GroupId $Group.Id |
    Select-Object Id,AdditionalProperties |
    Format-Table -AutoSize
```

```powershell
# Export evidence.

$EvidencePath = ".\Evidence\M365-Licensing"
New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Get-MgSubscribedSku |
    ConvertTo-Json -Depth 20 |
    Out-File "$EvidencePath\subscribed-skus-final.json"

Get-MgUser -All -Property Id,DisplayName,UserPrincipalName,AccountEnabled,UsageLocation,AssignedLicenses |
    Select-Object Id,DisplayName,UserPrincipalName,AccountEnabled,UsageLocation,AssignedLicenses |
    Export-Csv "$EvidencePath\users-assigned-licenses-final.csv" -NoTypeInformation

Get-MgGroup -All -Property Id,DisplayName,MailEnabled,SecurityEnabled,GroupTypes,AssignedLicenses |
    Select-Object Id,DisplayName,MailEnabled,SecurityEnabled,GroupTypes,AssignedLicenses |
    Export-Csv "$EvidencePath\groups-assigned-licenses-final.csv" -NoTypeInformation
```

Portal verification:

| Validation Area | Portal Path | Good Result |
|---|---|---|
| License inventory | Billing > Licenses | Purchased, assigned, and available counts are visible |
| User license state | Users > Active users > select user > Licenses and apps | User license and service app state is visible |
| Unlicensed users | Users > Active users > filter unlicensed users | Unlicensed users are known |
| Group license assignment | Billing > Licenses or group license view | License group assignment is visible |
| Group license errors | License group details / error view | Errors are visible or no errors are present |
| Usage reports | Reports > Usage | Workload usage is visible |
| Service plan state | User > Licenses and apps | Enabled and disabled apps match plan |
| Exchange mailbox provisioning | Exchange admin center | Mailbox appears when Exchange plan is enabled |
| Teams access | Teams web client | User can access Teams when plan is enabled |
| OneDrive access | OneDrive web | User can access OneDrive when SharePoint/OneDrive plan is enabled |

# 06_Manage_M365_Licensing_Service_Plans_Group_Based_Licensing_And_Usage_Rollback
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify direct license assigned by mistake | Admin Workstation | `Get-MgUserLicenseDetail -UserId "<user-upn>"` | Incorrect direct license is identified |
| 2 | Remove incorrect direct license | Admin Workstation | `Set-MgUserLicense -RemoveLicenses` | Direct license is removed |
| 3 | Reassign direct license removed by mistake | Admin Workstation | `Set-MgUserLicense -AddLicenses` | Direct license is restored |
| 4 | Identify user added to wrong license group | Admin Workstation | `Get-MgGroupMember -GroupId "<group-id>"` | Incorrect membership is identified |
| 5 | Remove user from wrong license group | Admin Workstation | `Remove-MgGroupMemberByRef` | User no longer inherits license from that group |
| 6 | Re-add user removed from license group by mistake | Admin Workstation | `New-MgGroupMemberByRef` | User inherits license again |
| 7 | Identify license assigned to wrong group | Admin Workstation | `Get-MgGroupLicenseDetail -GroupId "<group-id>"` | Incorrect group license is identified |
| 8 | Remove wrong group license assignment | Admin Workstation | `Set-MgGroupLicense -RemoveLicenses` | Group no longer assigns the wrong SKU |
| 9 | Restore group license assignment removed by mistake | Admin Workstation | `Set-MgGroupLicense -AddLicenses` | Group license assignment is restored |
| 10 | Restore disabled service plan state | Admin Workstation | `Set-MgUserLicense` or `Set-MgGroupLicense` with previous `DisabledPlans` | Service plan state returns to baseline |
| 11 | Restore previous usage location if changed incorrectly | Admin Workstation | `Update-MgUser -UsageLocation "<previous-location>"` | Usage location is restored |
| 12 | Validate rollback on user | Admin Workstation | `Get-MgUserLicenseDetail` | User license state matches rollback plan |
| 13 | Validate rollback on group | Admin Workstation | `Get-MgGroupLicenseDetail` | Group license state matches rollback plan |
| 14 | Confirm service access impact | Test User | Sign in to affected service | Access state matches expected result |
| 15 | Document rollback | Operator | Notes | Rollback owner, time, and validation are recorded |

Rollback command examples:

```powershell
# Remove direct user license.

$UserPrincipalName = "<user-upn>"
$SkuPartNumber = "<sku-part-number>"

$Sku = Get-MgSubscribedSku |
    Where-Object {$_.SkuPartNumber -eq $SkuPartNumber}

Set-MgUserLicense `
    -UserId $UserPrincipalName `
    -AddLicenses @() `
    -RemoveLicenses @($Sku.SkuId)
```

```powershell
# Remove user from license group.

$GroupDisplayName = "<license-group-name>"
$UserPrincipalName = "<user-upn>"

$Group = Get-MgGroup -Filter "displayName eq '$GroupDisplayName'"
$User = Get-MgUser -UserId $UserPrincipalName

Remove-MgGroupMemberByRef `
    -GroupId $Group.Id `
    -DirectoryObjectId $User.Id
```

```powershell
# Remove license from group.

$GroupDisplayName = "<license-group-name>"
$SkuPartNumber = "<sku-part-number>"

$Group = Get-MgGroup -Filter "displayName eq '$GroupDisplayName'"
$Sku = Get-MgSubscribedSku |
    Where-Object {$_.SkuPartNumber -eq $SkuPartNumber}

Set-MgGroupLicense `
    -GroupId $Group.Id `
    -AddLicenses @() `
    -RemoveLicenses @($Sku.SkuId)
```

Rollback note template:

```text
Change made:
Object changed:
Previous value:
New value:
Rollback action:
Rollback owner:
Rollback time:
Validation performed:
Remaining issue:
```

# 06_Manage_M365_Licensing_Service_Plans_Group_Based_Licensing_And_Usage_Failure_Checks
| Symptom | Command | What Usually Broke |
|---|---|---|
| Billing licenses page denied | Microsoft 365 admin center > Billing > Licenses | Missing License Administrator, Billing role, or Global Reader permissions |
| License assignment button unavailable | Users > Active users > Licenses and apps | Missing admin role or user source/state issue |
| Graph license assignment fails | `Get-MgContext` | Missing `User.ReadWrite.All`, `Group.ReadWrite.All`, or `Directory.ReadWrite.All` |
| User license assignment fails | `Get-MgUser -UserId "<user-upn>" -Property UsageLocation` | Usage location is missing |
| No licenses available | `Get-MgSubscribedSku` | Consumed units equal or exceed enabled units |
| Wrong SKU selected | `Get-MgSubscribedSku` | SKU part number confusion |
| Service plan cannot be disabled | SKU service plan list | Plan dependency, unsupported plan, or wrong plan ID |
| User still has service after license removal | `Get-MgUserLicenseDetail` | User also inherits license from another group |
| User lost service unexpectedly | `Get-MgUserLicenseDetail` and group membership | Removed from license group or direct license removed |
| Group license does not apply | `Get-MgGroupLicenseDetail` | License not assigned to group or processing delay |
| Group-based licensing error appears | Admin center group license error view | Missing usage location, no licenses, conflict, or invalid service plan |
| Nested group members not licensed | `Get-MgGroupMember` | Group-based licensing does not process nested group membership |
| User added to group but not licensed yet | `Get-MgUserLicenseDetail` | Processing delay or assignment error |
| Dynamic group licensing does not update | Group membership view | Dynamic membership rule issue or processing delay |
| Synced group cannot be edited in cloud | Group details | Group membership is mastered on-prem |
| User usage location keeps reverting | User properties | Synced identity or upstream provisioning process overwrites cloud value |
| Direct license removal fails | `Set-MgUserLicense` error | License is inherited from group or wrong SKU ID used |
| Group license removal affects too many users | `Get-MgGroupMember` | Group scope was broader than expected |
| License cleanup candidate still active | Reports > Usage | User is active in workload despite stale assumption |
| Exchange mailbox not created after license assignment | Exchange admin center / `Get-EXOMailbox` | Provisioning delay, Exchange plan disabled, or license error |
| Teams access fails after license assignment | Teams web client | Teams service plan disabled or provisioning delay |
| OneDrive not available after license assignment | OneDrive web | SharePoint/OneDrive plan disabled or provisioning delay |
| Multiple conflicting licenses assigned | `Get-MgUserLicenseDetail` | User has direct and group-based overlapping SKUs |
| Bulk CSV fails | Bulk result CSV | Bad headers, wrong UPN, missing usage location, or invalid SKU |
| Wrong tenant modified | `Get-MgContext` | Admin connected to wrong tenant |
| License report counts do not match expectation | `Get-MgSubscribedSku` | Assignment delay, disabled subscription, or different SKU counted |

# 06_Manage_M365_Licensing_Service_Plans_Group_Based_Licensing_And_Usage_Related_Labs
| Lab                                                                                        | Relationship                                                                              |
| ------------------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------- |
| 01_Create_And_Configure_Microsoft_365_Tenant_Org_Profile_And_Service_Settings              | Tenant and admin center baseline must exist first                                         |
| 02_Configure_M365_Custom_Domains_DNS_And_Network_Connectivity_Insights                     | Verified domains are required before assigning branded UPN/mail workloads at scale        |
| 03_Monitor_Service_Health_Message_Center_Usage_Reports_And_M365_Backup                     | Usage reports support license cleanup and adoption review                                 |
| 04_Create_Manage_And_Bulk_Administer_M365_Users_Contacts_And_Guests                        | Users require usage location and account readiness before licensing                       |
| 05_Manage_M365_Groups_Security_Groups_Distribution_Groups_Shared_Mailboxes_And_M365_Groups | Security groups are used for group-based licensing                                        |
| 07_Manage_Admin_Roles_Role_Groups_Admin_Units_And_Delegation                               | Licensing requires delegated roles such as License Administrator and Groups Administrator |
| 08_Troubleshoot_M365_Tenant_User_Group_License_Role_And_Service_Health_Issues              | Troubleshoots failed assignments, group licensing errors, and service access issues       |
| 06_Configure_Entra_Users_Groups_Guests_Licenses_And_SSPR_Baseline                          | Cloud foundation user, group, guest, and license baseline                                 |
| 08_Map_On_Prem_Groups_To_Cloud_Roles_Licensing_Access_And_App_Assignments                  | Hybrid group-to-cloud licensing and access mapping                                        |
| 05_Configure_Cost_Management_Budgets_Alerts_And_Advisor_Recommendations                    | Cost governance pattern for license cleanup and waste reduction                           |