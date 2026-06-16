01_Create_And_Configure_Microsoft_365_Tenant_Org_Profile_And_Service_Settings.md
# 01_Create_And_Configure_Microsoft_365_Tenant_Org_Profile_And_Service_Settings

# 01_Create_And_Configure_Microsoft_365_Tenant_Org_Profile_And_Service_Settings_Index
01_Create_And_Configure_Microsoft_365_Tenant_Org_Profile_And_Service_Settings.md
01_Create_And_Configure_Microsoft_365_Tenant_Org_Profile_And_Service_Settings
01_Create_And_Configure_Microsoft_365_Tenant_Org_Profile_And_Service_Settings_Source_Basis
01_Create_And_Configure_Microsoft_365_Tenant_Org_Profile_And_Service_Settings_Mental_Model
01_Create_And_Configure_Microsoft_365_Tenant_Org_Profile_And_Service_Settings_Planning_Table
01_Create_And_Configure_Microsoft_365_Tenant_Org_Profile_And_Service_Settings_Configuration_Checklist
01_Create_And_Configure_Microsoft_365_Tenant_Org_Profile_And_Service_Settings_Tenant_Inventory_Skeleton
01_Create_And_Configure_Microsoft_365_Tenant_Org_Profile_And_Service_Settings_Org_Profile_Skeleton
01_Create_And_Configure_Microsoft_365_Tenant_Org_Profile_And_Service_Settings_Release_Preferences_Skeleton
01_Create_And_Configure_Microsoft_365_Tenant_Org_Profile_And_Service_Settings_Service_Settings_Skeleton
01_Create_And_Configure_Microsoft_365_Tenant_Org_Profile_And_Service_Settings_Admin_Center_Baseline_Skeleton
01_Create_And_Configure_Microsoft_365_Tenant_Org_Profile_And_Service_Settings_Verification_Commands
01_Create_And_Configure_Microsoft_365_Tenant_Org_Profile_And_Service_Settings_Rollback
01_Create_And_Configure_Microsoft_365_Tenant_Org_Profile_And_Service_Settings_Failure_Checks
01_Create_And_Configure_Microsoft_365_Tenant_Org_Profile_And_Service_Settings_Related_Labs

# 01_Create_And_Configure_Microsoft_365_Tenant_Org_Profile_And_Service_Settings_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Microsoft 365 admin center overview | Main admin center navigation, Dashboard view, Users, Teams and groups, Roles, Billing, Settings, Setup, Reports, Health, and Admin centers |
| Microsoft Learn | Set up Microsoft 365 for business | First-run Microsoft 365 setup flow and subscription setup path |
| Microsoft Learn | Organization profile | Tenant-wide organization identity, profile data, and release preferences |
| Microsoft Learn | Org settings | Tenant-wide Microsoft 365 service settings |
| Microsoft Learn | Setup | Guided setup prompts for domains, sign-in security, migration, app installs, and feature enablement |
| Microsoft Learn | Microsoft Graph PowerShell | Tenant inventory, organization object, verified domains, subscribed SKUs, and read-only baseline checks |
| Microsoft Learn | Microsoft 365 admin roles | Admin access validation and least-privilege administration |
| Microsoft 365 operational practice | Tenant baseline before workload configuration | Prevents random service changes before identity, domain, licensing, and workload ownership are understood |

# 01_Create_And_Configure_Microsoft_365_Tenant_Org_Profile_And_Service_Settings_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Microsoft 365 tenant | Organization boundary for Microsoft 365 identities, domains, licenses, workload settings, service health, and admin roles |
| Initial tenant namespace | The original `<tenant>.onmicrosoft.com` namespace created during signup |
| Custom domain | Public DNS domain verified into Microsoft 365 for sign-in, email, Teams, SharePoint, and branding |
| Microsoft 365 admin center | Primary admin surface for common Microsoft 365 tenant administration |
| Dashboard view | Full admin view for complex settings and operational administration |
| Simplified view | Reduced view for smaller environments and basic tasks |
| Organization profile | Tenant-wide business identity, contact information, and release preference location |
| Org settings | Tenant-wide settings for Microsoft 365 apps and services |
| Setup | Guided Microsoft 365 setup actions, including domains, sign-in security, app deployment, and migration prompts |
| Service settings | Tenant or workload-level controls exposed through Microsoft 365 admin center |
| Admin centers | Links to workload-specific admin portals such as Entra, Exchange, SharePoint, Teams, Security, Compliance, Defender, and Intune |
| Release preferences | Controls whether the tenant receives standard feature release or targeted release |
| Standard release | Safer default for production tenants |
| Targeted release | Early feature exposure for selected users or the whole tenant |
| Tenant baseline | Documented starting state before changing domains, users, licenses, roles, groups, or workload settings |
| Least privilege | Admins should use the smallest role needed instead of defaulting to Global Administrator |
| Break glass account | Emergency cloud-only admin account protected from normal lockout dependencies |
| First rule | Inventory the tenant before changing the tenant |
| Blunt rule | Do not click every Microsoft 365 setup prompt just because it appears |

# 01_Create_And_Configure_Microsoft_365_Tenant_Org_Profile_And_Service_Settings_Planning_Table
| Item | Example | Decision |
|---|---|---|
| Tenant display name | `Contoso` | `<organization-name>` |
| Initial tenant namespace | `contoso.onmicrosoft.com` | `<tenant>.onmicrosoft.com` |
| Tenant ID | `00000000-0000-0000-0000-000000000000` | `<tenant-id>` |
| Primary custom domain | `contoso.com` | `<domain.com>` |
| Admin account used | `admin@contoso.onmicrosoft.com` | `<admin-upn>` |
| Admin role used | `Global Administrator` for setup only | `<role-name>` |
| Emergency admin account | `breakglass01@contoso.onmicrosoft.com` | `<break-glass-upn>` |
| Preferred admin center view | Dashboard view | `Dashboard view` |
| Release preference | Standard release | `<Standard / Targeted selected / Targeted everyone>` |
| Targeted release pilot users | IT admins only | `<pilot-users>` |
| Billing owner | `billingadmin@contoso.com` | `<billing-owner>` |
| Technical support owner | `supportadmin@contoso.com` | `<support-owner>` |
| Security owner | `securityadmin@contoso.com` | `<security-owner>` |
| Identity owner | `identityadmin@contoso.com` | `<identity-owner>` |
| Exchange owner | `exchangeadmin@contoso.com` | `<exchange-owner>` |
| SharePoint owner | `spadmin@contoso.com` | `<sharepoint-owner>` |
| Teams owner | `teamsadmin@contoso.com` | `<teams-owner>` |
| Evidence path | `.\Evidence\M365-Tenant-Baseline` | `<evidence-path>` |
| Change window | `After-hours lab window` | `<change-window>` |
| Rollback owner | `admin@contoso.com` | `<rollback-owner>` |

# 01_Create_And_Configure_Microsoft_365_Tenant_Org_Profile_And_Service_Settings_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Open Microsoft 365 admin center | Admin Workstation | `https://admin.cloud.microsoft` | Microsoft 365 admin center opens |
| 2 | Confirm correct signed-in account | Admin Workstation | Portal account menu | Expected admin UPN is signed in |
| 3 | Confirm correct tenant | Admin Workstation | Portal account menu / tenant switcher | Correct tenant name is visible |
| 4 | Confirm admin center access | Admin Workstation | App launcher > Admin | Admin center is available |
| 5 | Switch to Dashboard view | Admin Workstation | Portal view selector | Dashboard view is active |
| 6 | Expand full navigation | Admin Workstation | Portal: Show all | Full admin navigation is visible |
| 7 | Record navigation baseline | Admin Workstation | Screenshot / notes | Current admin center layout is documented |
| 8 | Open organization profile | Admin Workstation | Settings > Org settings > Organization profile | Organization profile opens |
| 9 | Record organization display name | Admin Workstation | Portal: Organization profile | Current organization name is documented |
| 10 | Record organization contact fields | Admin Workstation | Portal: Organization profile | Address, phone, and contact fields are documented |
| 11 | Update organization profile if required | Admin Workstation | Portal: Organization profile > Edit | Profile reflects intended organization details |
| 12 | Record original Microsoft 365 namespace | Admin Workstation | Settings > Domains | `<tenant>.onmicrosoft.com` namespace is documented |
| 13 | Confirm custom domain work is deferred | Admin Workstation | Notes | Custom domain task is mapped to workbook 02 |
| 14 | Open release preferences | Admin Workstation | Settings > Org settings > Organization profile > Release preferences | Release preference settings are visible |
| 15 | Record current release preference | Admin Workstation | Screenshot / notes | Existing release state is documented |
| 16 | Set safe production baseline | Admin Workstation | Select `Standard release for everyone` | Tenant receives stable release cadence |
| 17 | Configure targeted release only for pilot users if required | Admin Workstation | Select `Targeted release for selected users` | Only intended pilot users receive early features |
| 18 | Save release preference | Admin Workstation | Portal: Save | Release preference is saved |
| 19 | Record release preference decision | Admin Workstation | Notes | Reason for release setting is documented |
| 20 | Open Org settings | Admin Workstation | Settings > Org settings | Tenant-wide settings list is visible |
| 21 | Inventory available setting categories | Admin Workstation | Portal list / notes | Service setting categories are documented |
| 22 | Review Microsoft 365 Apps settings | Admin Workstation | Settings > Org settings > Microsoft 365 Apps | Apps settings are visible |
| 23 | Review email-related service settings | Admin Workstation | Settings > Org settings | Email settings are identified |
| 24 | Review Teams-related service settings | Admin Workstation | Settings > Org settings | Teams settings are identified |
| 25 | Review SharePoint and OneDrive service settings | Admin Workstation | Settings > Org settings | SharePoint and OneDrive settings are identified |
| 26 | Review security and privacy related settings | Admin Workstation | Settings > Org settings | Security, privacy, and consent areas are identified |
| 27 | Avoid deep workload changes | Admin Workstation | Notes | Exchange, SharePoint, Teams, Defender, Purview, and Intune changes are deferred |
| 28 | Open Setup page | Admin Workstation | Setup | Guided setup prompts are visible |
| 29 | Document active setup prompts | Admin Workstation | Screenshot / notes | Open setup actions are recorded |
| 30 | Map setup prompts to correct workbooks | Admin Workstation | Notes | Domain, MFA, migration, app install, and DLP prompts are mapped |
| 31 | Review domain setup prompt | Admin Workstation | Setup > Domains | Domain prompt is identified and deferred to workbook 02 |
| 32 | Review sign-in security prompt | Admin Workstation | Setup > Sign-in and security | Security prompt is identified and deferred to identity workbook |
| 33 | Review app install prompt | Admin Workstation | Setup > Install apps | App deployment prompt is documented |
| 34 | Review migration prompt | Admin Workstation | Setup > Migration | Migration prompt is documented if present |
| 35 | Review mobile app protection prompt | Admin Workstation | Setup | Intune-related prompt is documented if present |
| 36 | Review DLP or compliance prompt | Admin Workstation | Setup | Purview-related prompt is documented if present |
| 37 | Open Admin centers menu | Admin Workstation | Admin centers | Workload admin centers are listed |
| 38 | Validate Microsoft Entra admin center access | Admin Workstation | Admin centers > Microsoft Entra | Entra admin center opens or access issue is documented |
| 39 | Validate Exchange admin center access | Admin Workstation | Admin centers > Exchange | Exchange admin center opens or access issue is documented |
| 40 | Validate SharePoint admin center access | Admin Workstation | Admin centers > SharePoint | SharePoint admin center opens or access issue is documented |
| 41 | Validate Teams admin center access | Admin Workstation | Admin centers > Teams | Teams admin center opens or access issue is documented |
| 42 | Validate Security admin center access | Admin Workstation | Admin centers > Security | Security admin center opens or access issue is documented |
| 43 | Validate Compliance or Purview access | Admin Workstation | Admin centers > Compliance / Purview | Compliance portal opens or access issue is documented |
| 44 | Record unavailable admin centers | Admin Workstation | Notes | Missing portals are mapped to role, license, or plan issue |
| 45 | Review Home dashboard cards | Admin Workstation | Home | Dashboard cards are visible |
| 46 | Add useful dashboard cards | Admin Workstation | Home > Add card | Operational shortcuts are pinned |
| 47 | Pin Service health card | Admin Workstation | Home > Add card | Service health is visible from dashboard |
| 48 | Pin Users or Active users card | Admin Workstation | Home > Add card | User admin shortcut is available |
| 49 | Pin Billing or Licenses card | Admin Workstation | Home > Add card | Subscription/license shortcut is available |
| 50 | Remove noisy cards if needed | Admin Workstation | Card menu > Remove | Dashboard remains clean |
| 51 | Open Help and support | Admin Workstation | Help and support | Support workflow opens |
| 52 | Confirm support path | Admin Workstation | Help and support search | Admin can search and open support if permitted |
| 53 | Install Microsoft Graph PowerShell if needed | Admin Workstation | `Install-Module Microsoft.Graph -Scope CurrentUser` | Microsoft Graph PowerShell is installed |
| 54 | Connect Microsoft Graph PowerShell | Admin Workstation | `Connect-MgGraph -Scopes "Organization.Read.All","Directory.Read.All"` | Graph session connects |
| 55 | Confirm Graph context | Admin Workstation | `Get-MgContext` | Tenant ID and scopes are visible |
| 56 | Export organization object | Admin Workstation | `Get-MgOrganization` | Tenant organization object is returned |
| 57 | Export verified domains | Admin Workstation | `(Get-MgOrganization).VerifiedDomains` | Verified domain inventory is visible |
| 58 | Export subscribed SKUs | Admin Workstation | `Get-MgSubscribedSku` | Subscription/license inventory is visible |
| 59 | Save tenant baseline output | Admin Workstation | Export commands in skeleton below | Baseline files are stored in evidence path |
| 60 | Document completion state | Operator | Record tenant name, tenant ID, org profile, release ring, setup prompts, admin centers, and evidence path | Tenant baseline workbook is complete |

# 01_Create_And_Configure_Microsoft_365_Tenant_Org_Profile_And_Service_Settings_Tenant_Inventory_Skeleton
```powershell
# Run from an admin workstation with Microsoft Graph PowerShell installed.
# Purpose: collect a read-only Microsoft 365 tenant baseline.

$EvidencePath = ".\Evidence\M365-Tenant-Baseline"
New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Connect-MgGraph -Scopes "Organization.Read.All","Directory.Read.All"

Get-MgContext |
    Tee-Object "$EvidencePath\graph-context.txt"

Get-MgOrganization |
    Select-Object Id,DisplayName,VerifiedDomains |
    Format-List |
    Tee-Object "$EvidencePath\organization.txt"

(Get-MgOrganization).VerifiedDomains |
    Format-Table Name,Type,IsDefault,IsInitial,IsVerified -AutoSize |
    Tee-Object "$EvidencePath\verified-domains.txt"

Get-MgSubscribedSku |
    Select-Object SkuPartNumber,ConsumedUnits,PrepaidUnits |
    Format-Table -AutoSize |
    Tee-Object "$EvidencePath\subscribed-skus.txt"
```

Expected output:

| Check | Expected Result |
|---|---|
| Graph context | Correct tenant ID is visible |
| Organization object | Tenant display name and verified domain list return |
| Verified domains | Initial and verified domains are visible |
| Subscribed SKUs | License inventory returns |
| Evidence folder | Baseline text files exist |

# 01_Create_And_Configure_Microsoft_365_Tenant_Org_Profile_And_Service_Settings_Org_Profile_Skeleton
```text
Portal path:

Microsoft 365 admin center
Settings
Org settings
Organization profile
```

| Setting | Baseline Action | Expected Result |
|---|---|---|
| Organization name | Confirm or update | Correct business name is displayed |
| Business address | Confirm or update | Address is accurate |
| Phone number | Confirm or update | Supportable contact number is recorded |
| Technical contact | Confirm or update if available | Correct technical owner is documented |
| Release preferences | Configure in release skeleton | Release ring is intentional |
| Partner relationships | Document only unless approved | Partner access is not changed blindly |
| Tenant branding or theme | Document only unless required | Branding changes are deferred if not part of this task |

Operational note:

```text
Do not use the organization profile page as a dumping ground for random tenant changes.
Record the current profile first, then change only the fields with an actual business reason.
```

# 01_Create_And_Configure_Microsoft_365_Tenant_Org_Profile_And_Service_Settings_Release_Preferences_Skeleton
```text
Portal path:

Microsoft 365 admin center
Settings
Org settings
Organization profile
Release preferences
```

| Release Option | Use Case | Risk |
|---|---|---|
| Standard release for everyone | Production-safe default | Slower feature arrival |
| Targeted release for selected users | IT pilot or admin testing | Requires pilot list maintenance |
| Targeted release for everyone | Lab or training tenant | Highest change noise and user confusion |

Recommended baseline:

| Tenant Type | Recommended Setting |
|---|---|
| Production tenant | Standard release for everyone |
| Lab tenant | Targeted release for selected users |
| Training tenant | Targeted release for selected users |
| Small business tenant | Standard release for everyone |
| IT pilot tenant | Targeted release for selected users |

Checklist:

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Open release preferences | Admin Workstation | Portal path above | Release preferences pane opens |
| 2 | Screenshot current setting | Admin Workstation | Screenshot / notes | Before state is captured |
| 3 | Select release model | Admin Workstation | Portal selection | Intended release model is selected |
| 4 | Add selected users if required | Admin Workstation | Portal user picker | Only pilot users are added |
| 5 | Save change | Admin Workstation | Portal: Save | Release preference is saved |
| 6 | Record final setting | Admin Workstation | Notes | Release ring is documented |

# 01_Create_And_Configure_Microsoft_365_Tenant_Org_Profile_And_Service_Settings_Service_Settings_Skeleton
```text
Portal path:

Microsoft 365 admin center
Settings
Org settings
Services
```

| Service Area | Baseline Action | Dedicated Workbook |
|---|---|---|
| Domains | Identify only | 02_Configure_M365_Custom_Domains_DNS_And_Network_Connectivity_Insights |
| Microsoft 365 Apps | Review only | Endpoint or app deployment workbook |
| Exchange | Identify only | Exchange Online workbook |
| SharePoint | Identify only | SharePoint and OneDrive workbook |
| OneDrive | Identify only | SharePoint and OneDrive workbook |
| Teams | Identify only | Teams workbook |
| Viva Engage | Identify only | Viva workload workbook if needed |
| Bookings | Identify only | Optional workload workbook |
| Copilot | Identify only | Copilot admin workbook if needed |
| Security defaults or MFA prompts | Identify only | Identity and security workbook |
| DLP or compliance prompts | Identify only | Purview workbook |
| Mobile app protection | Identify only | Intune workbook |

Service settings capture template:

| Service | Current State | Change Needed | Deferred To | Owner |
|---|---|---|---|---|
| `<service-name>` | `<current-state>` | `<yes-no>` | `<workbook-name>` | `<owner>` |

Operational note:

```text
This workbook inventories service settings.
It does not harden Exchange, SharePoint, Teams, Defender, Purview, or Intune.
Those get separate workbooks so tenant setup stays clean and supportable.
```

# 01_Create_And_Configure_Microsoft_365_Tenant_Org_Profile_And_Service_Settings_Admin_Center_Baseline_Skeleton
```text
Portal path:

Microsoft 365 admin center
Admin centers
```

| Admin Center | Opens Successfully | Required Role | Notes |
|---|---:|---|---|
| Microsoft Entra | `<yes-no>` | Global Reader / Identity role | `<notes>` |
| Exchange | `<yes-no>` | Exchange Administrator | `<notes>` |
| SharePoint | `<yes-no>` | SharePoint Administrator | `<notes>` |
| Teams | `<yes-no>` | Teams Administrator | `<notes>` |
| Security | `<yes-no>` | Security Reader / Security Administrator | `<notes>` |
| Compliance / Purview | `<yes-no>` | Compliance role | `<notes>` |
| Defender | `<yes-no>` | Security role | `<notes>` |
| Intune | `<yes-no>` | Intune Administrator | `<notes>` |
| Viva Engage | `<yes-no>` | Workload role | `<notes>` |

Checklist:

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Open Admin centers menu | Admin Workstation | Portal: Admin centers | Workload portals are listed |
| 2 | Open Entra admin center | Admin Workstation | Portal link | Entra portal opens |
| 3 | Open Exchange admin center | Admin Workstation | Portal link | Exchange portal opens or issue is documented |
| 4 | Open SharePoint admin center | Admin Workstation | Portal link | SharePoint portal opens or issue is documented |
| 5 | Open Teams admin center | Admin Workstation | Portal link | Teams portal opens or issue is documented |
| 6 | Open Security admin center | Admin Workstation | Portal link | Security portal opens or issue is documented |
| 7 | Open Compliance or Purview portal | Admin Workstation | Portal link | Compliance portal opens or issue is documented |
| 8 | Record missing portals | Admin Workstation | Notes | Missing access is mapped to role, license, or plan |
| 9 | Defer role changes | Admin Workstation | Notes | No random Global Administrator assignment is made |

# 01_Create_And_Configure_Microsoft_365_Tenant_Org_Profile_And_Service_Settings_Verification_Commands
```powershell
# Run from an admin workstation.
# Purpose: verify tenant baseline after profile and settings review.

Connect-MgGraph -Scopes "Organization.Read.All","Directory.Read.All"

Get-MgContext

Get-MgOrganization |
    Select-Object Id,DisplayName,VerifiedDomains |
    Format-List

(Get-MgOrganization).VerifiedDomains |
    Format-Table Name,Type,IsDefault,IsInitial,IsVerified -AutoSize

Get-MgSubscribedSku |
    Select-Object SkuPartNumber,ConsumedUnits,PrepaidUnits |
    Format-Table -AutoSize
```

Optional export:

```powershell
$OutputPath = ".\Evidence\M365-Tenant-Baseline"
New-Item -ItemType Directory -Force -Path $OutputPath | Out-Null

Get-MgOrganization |
    Select-Object Id,DisplayName,VerifiedDomains |
    ConvertTo-Json -Depth 10 |
    Out-File "$OutputPath\organization.json"

(Get-MgOrganization).VerifiedDomains |
    ConvertTo-Json -Depth 10 |
    Out-File "$OutputPath\verified-domains.json"

Get-MgSubscribedSku |
    Select-Object SkuPartNumber,ConsumedUnits,PrepaidUnits |
    ConvertTo-Json -Depth 10 |
    Out-File "$OutputPath\subscribed-skus.json"
```

| Check | Device | PowerShell / Command | Good Output |
|---|---|---|---|
| Admin center opens | Admin Workstation | `https://admin.cloud.microsoft` | Microsoft 365 admin center loads |
| Correct admin signed in | Admin Workstation | Portal account menu | Expected admin UPN appears |
| Dashboard view active | Admin Workstation | Portal view selector | Dashboard view is enabled |
| Org profile opens | Admin Workstation | Settings > Org settings > Organization profile | Organization profile is accessible |
| Release preferences visible | Admin Workstation | Organization profile > Release preferences | Release state is visible |
| Setup prompts visible | Admin Workstation | Setup | Current setup prompts are visible |
| Admin centers visible | Admin Workstation | Admin centers | Workload portals are listed |
| Support opens | Admin Workstation | Help and support | Support assistant opens |
| Graph context valid | Admin Workstation | `Get-MgContext` | Tenant ID and scopes are visible |
| Organization object readable | Admin Workstation | `Get-MgOrganization` | Tenant organization details return |
| Domains readable | Admin Workstation | `(Get-MgOrganization).VerifiedDomains` | Initial and verified domains return |
| SKUs readable | Admin Workstation | `Get-MgSubscribedSku` | License inventory returns |

# 01_Create_And_Configure_Microsoft_365_Tenant_Org_Profile_And_Service_Settings_Rollback
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify changed organization profile fields | Admin Workstation | Compare notes/screenshots | Changed profile fields are known |
| 2 | Restore organization name if changed incorrectly | Admin Workstation | Settings > Org settings > Organization profile > Edit | Original organization name is restored |
| 3 | Restore business address if changed incorrectly | Admin Workstation | Settings > Org settings > Organization profile > Edit | Original address is restored |
| 4 | Restore contact details if changed incorrectly | Admin Workstation | Settings > Org settings > Organization profile > Edit | Original contact values are restored |
| 5 | Identify changed release preference | Admin Workstation | Organization profile > Release preferences | Current release preference is visible |
| 6 | Restore previous release preference | Admin Workstation | Select prior release option | Release ring returns to previous state |
| 7 | Restore targeted release pilot list | Admin Workstation | Release preferences user picker | Pilot list matches previous baseline |
| 8 | Restore dashboard cards if needed | Admin Workstation | Home > Add card / Remove card | Dashboard layout returns to previous state |
| 9 | Review accidental setup prompt actions | Admin Workstation | Setup and relevant workload admin center | Unintended changes are identified |
| 10 | Restore accidental service setting change | Admin Workstation | Settings > Org settings | Service setting returns to previous baseline |
| 11 | Validate tenant inventory after rollback | Admin Workstation | `Get-MgOrganization`; `Get-MgSubscribedSku` | Tenant baseline is readable after rollback |
| 12 | Document rollback | Operator | Notes | Rollback owner, time, and validation are recorded |

Rollback note template:

```text
Change made:
Previous value:
New value:
Rollback action:
Rollback owner:
Rollback time:
Validation performed:
Remaining issue:
```

# 01_Create_And_Configure_Microsoft_365_Tenant_Org_Profile_And_Service_Settings_Failure_Checks
| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| Admin app is not visible | Signed-in user is not an admin | App launcher and account menu | Sign in with an admin account |
| Microsoft 365 admin center denies access | User lacks required admin role | User role assignment | Use proper admin account or assign least-privileged role |
| Dashboard view is not visible | UI state, tenant type, or view difference | Admin center view selector | Use Show all and search |
| Settings menu is missing options | Role, license, plan, or cloud limitation | Roles and subscription | Confirm role and license coverage |
| Organization profile cannot be edited | Missing admin privilege | Role assignment | Use account with correct tenant admin role |
| Release preferences cannot be changed | Missing admin privilege | Role assignment | Use appropriate admin account |
| Setup prompts differ from expected | Tenant state or subscription plan differs | Setup page | Document actual prompts |
| Domain prompt appears | Tenant still needs custom domain work | Settings > Domains | Defer to workbook 02 |
| Sign-in security prompt appears | Tenant security baseline incomplete | Setup > Sign-in security | Defer to identity and security workbook |
| Workload admin center missing | Workload not licensed or not available | Billing and licenses | Confirm subscription includes workload |
| Exchange admin center blocked | Missing Exchange role or license | Admin centers > Exchange | Assign Exchange admin role if required |
| SharePoint admin center blocked | Missing SharePoint role or license | Admin centers > SharePoint | Assign SharePoint admin role if required |
| Teams admin center blocked | Missing Teams role or license | Admin centers > Teams | Assign Teams admin role if required |
| Security or Compliance portal blocked | Missing security/compliance role | Admin centers menu | Assign least-privileged role |
| Help and support cannot open case | Role, support plan, partner, or subscription limitation | Help and support | Confirm support path |
| Graph module missing | Microsoft Graph PowerShell not installed | `Get-Module -ListAvailable Microsoft.Graph` | Install Microsoft Graph module |
| Graph connect fails | Consent, scope, browser, or network issue | `Connect-MgGraph` error | Reconnect with required scopes |
| Graph returns insufficient privileges | Missing scope or admin consent | `Get-MgContext` | Reconnect with `Organization.Read.All` and `Directory.Read.All` |
| Verified domains look wrong | Domain not added or not verified | Settings > Domains | Defer to domain workbook |
| License inventory looks wrong | Wrong tenant or billing issue | `Get-MgContext`; Billing | Confirm tenant and subscription |
| Dashboard cards missing | Cards not pinned | Home > Add card | Add required cards |

# 01_Create_And_Configure_Microsoft_365_Tenant_Org_Profile_And_Service_Settings_Related_Labs
| Lab                                                                                        | Relationship                                                                                     |
| ------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------ |
| 02_Configure_M365_Custom_Domains_DNS_And_Network_Connectivity_Insights                     | Handles custom domains, DNS verification, and M365 network connectivity visibility               |
| 03_Monitor_Service_Health_Message_Center_Usage_Reports_And_M365_Backup                     | Handles service health, Message center, usage reports, and Microsoft 365 Backup                  |
| 04_Create_Manage_And_Bulk_Administer_M365_Users_Contacts_And_Guests                        | Handles users, contacts, guests, and bulk user administration                                    |
| 05_Manage_M365_Groups_Security_Groups_Distribution_Groups_Shared_Mailboxes_And_M365_Groups | Handles groups, distribution groups, security groups, Microsoft 365 groups, and shared mailboxes |
| 06_Manage_M365_Licensing_Service_Plans_Group_Based_Licensing_And_Usage                     | Handles licensing, service plans, and group-based licensing                                      |
| 07_Manage_Admin_Roles_Role_Groups_Admin_Units_And_Delegation                               | Handles admin roles, role groups, admin units, and delegation                                    |
| 08_Troubleshoot_M365_Tenant_User_Group_License_Role_And_Service_Health_Issues              | Handles tenant, user, group, license, role, and service health troubleshooting                   |
| 01_Cloud_Core_Admin_Foundation                                                             | Provides baseline cloud tenant and admin account mental model                                    |
| 07_Cloud_Hybrid_Identity_And_Operations_Bridge                                             | Handles hybrid identity, Entra Connect, SSPR, MFA, and Conditional Access dependencies           |