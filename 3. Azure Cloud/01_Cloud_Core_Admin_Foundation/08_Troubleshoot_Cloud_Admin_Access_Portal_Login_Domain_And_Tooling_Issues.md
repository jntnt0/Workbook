# Troubleshoot_Cloud_Admin_Access_Portal_Login_Domain_And_Tooling_Issues

# Troubleshoot_Cloud_Admin_Access_Portal_Login_Domain_And_Tooling_Issues_Index
08_Troubleshoot_Cloud_Admin_Access_Portal_Login_Domain_And_Tooling_Issues.md
Troubleshoot_Cloud_Admin_Access_Portal_Login_Domain_And_Tooling_Issues
Troubleshoot_Cloud_Admin_Access_Portal_Login_Domain_And_Tooling_Issues_Source_Basis
Troubleshoot_Cloud_Admin_Access_Portal_Login_Domain_And_Tooling_Issues_Mental_Model
Troubleshoot_Cloud_Admin_Access_Portal_Login_Domain_And_Tooling_Issues_Planning_Table
Troubleshoot_Cloud_Admin_Access_Portal_Login_Domain_And_Tooling_Issues_Configuration_Checklist
Troubleshoot_Cloud_Admin_Access_Portal_Login_Domain_And_Tooling_Issues_Initial_Triage_Skeleton
Troubleshoot_Cloud_Admin_Access_Portal_Login_Domain_And_Tooling_Issues_Portal_Login_Skeleton
Troubleshoot_Cloud_Admin_Access_Portal_Login_Domain_And_Tooling_Issues_Wrong_Tenant_And_Subscription_Skeleton
Troubleshoot_Cloud_Admin_Access_Portal_Login_Domain_And_Tooling_Issues_Role_And_Scope_Skeleton
Troubleshoot_Cloud_Admin_Access_Portal_Login_Domain_And_Tooling_Issues_Conditional_Access_Skeleton
Troubleshoot_Cloud_Admin_Access_Portal_Login_Domain_And_Tooling_Issues_Domain_And_DNS_Skeleton
Troubleshoot_Cloud_Admin_Access_Portal_Login_Domain_And_Tooling_Issues_Azure_CLI_Skeleton
Troubleshoot_Cloud_Admin_Access_Portal_Login_Domain_And_Tooling_Issues_Az_PowerShell_Skeleton
Troubleshoot_Cloud_Admin_Access_Portal_Login_Domain_And_Tooling_Issues_Graph_PowerShell_Skeleton
Troubleshoot_Cloud_Admin_Access_Portal_Login_Domain_And_Tooling_Issues_Workload_PowerShell_Skeleton
Troubleshoot_Cloud_Admin_Access_Portal_Login_Domain_And_Tooling_Issues_Service_Health_And_Message_Center_Skeleton
Troubleshoot_Cloud_Admin_Access_Portal_Login_Domain_And_Tooling_Issues_Evidence_Capture_Skeleton
Troubleshoot_Cloud_Admin_Access_Portal_Login_Domain_And_Tooling_Issues_Verification_Commands
Troubleshoot_Cloud_Admin_Access_Portal_Login_Domain_And_Tooling_Issues_Rollback
Troubleshoot_Cloud_Admin_Access_Portal_Login_Domain_And_Tooling_Issues_Failure_Checks
Troubleshoot_Cloud_Admin_Access_Portal_Login_Domain_And_Tooling_Issues_Related_Labs

# Troubleshoot_Cloud_Admin_Access_Portal_Login_Domain_And_Tooling_Issues_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Microsoft Entra sign-in logs | Sign-in logs show who signed in, how, what resource was accessed, failure details, and are used for login troubleshooting |
| Microsoft Learn | Conditional Access sign-in troubleshooting | Review the browser error, More Details, correlation ID, sign-in log event, Conditional Access tab, Troubleshooting and support tab, and policy result |
| Microsoft Learn | Microsoft Entra audit logs | Audit logs show tenant changes to users, groups, applications, licenses, roles, and settings |
| Microsoft Learn | Microsoft 365 Health dashboard | Health dashboard and Service health help identify Microsoft 365 service incidents before local troubleshooting |
| Microsoft Learn | Microsoft 365 Message center | Message center tracks upcoming changes, retirements, admin impact, user impact, major updates, and required actions |
| Microsoft Learn | Azure CLI authentication | `az login`, tenant selection, and subscription context are core to Azure CLI access troubleshooting |
| Microsoft Learn | Azure PowerShell authentication | `Connect-AzAccount` and `Get-AzContext` validate Az PowerShell account, tenant, and subscription context |
| Microsoft Learn | Microsoft Graph PowerShell authentication | `Connect-MgGraph`, scopes, `Get-MgContext`, and `Disconnect-MgGraph` validate Graph PowerShell access |
| Operational practice | Cloud admin troubleshooting | Separates service outage, wrong account, wrong tenant, wrong role plane, wrong subscription, DNS/domain failure, CA block, MFA issue, stale token, and module/tooling failure |

# Troubleshoot_Cloud_Admin_Access_Portal_Login_Domain_And_Tooling_Issues_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Portal access failure | Browser-based admin center cannot open, loops, denies access, or lands in the wrong tenant |
| Login failure | Authentication problem before authorization is evaluated |
| Authorization failure | Sign-in succeeds, but the user lacks the role/scope needed for the admin action |
| Wrong tenant | Admin is signed into a different Microsoft Entra tenant than the target tenant |
| Wrong subscription | Azure CLI, Az PowerShell, or Azure portal is pointed at the wrong Azure subscription |
| Wrong role plane | Admin has a role in Azure, Entra, M365, or workload plane, but the task needs a different plane |
| Conditional Access block | CA policy blocks sign-in or requires a condition the admin cannot satisfy |
| MFA/authentication method problem | Admin cannot complete required authentication prompt or method registration |
| Device compliance problem | CA requires compliant, hybrid joined, approved app, or app protection condition that the device/session lacks |
| Domain/namespace problem | UPN suffix, custom domain, DNS verification, or MX/autodiscover/SPF state causes sign-in or service confusion |
| Service health issue | Microsoft service incident affects admin portals, Exchange, Teams, SharePoint, Entra, or other cloud workloads |
| Message center change | Microsoft announced an upcoming or active change that explains new behavior |
| Stale token | Cached browser, WAM, Azure CLI, Az PowerShell, Graph, or workload token still reflects old role/context |
| Graph scope problem | `Connect-MgGraph` succeeded but does not include required delegated scopes |
| Module problem | PowerShell module is missing, old, conflicting, blocked by execution policy, or imported in the wrong shell |
| Context problem | Tool is connected, but not to the intended account, tenant, cloud, or subscription |
| Evidence first | Capture correlation ID, request ID, timestamp, UPN, tenant ID, resource, app, and error code before changing settings |
| First rule | Check service health, then sign-in logs, then tenant/context, then roles, then Conditional Access, then tooling |

# Troubleshoot_Cloud_Admin_Access_Portal_Login_Domain_And_Tooling_Issues_Planning_Table
| Item | Example | Decision |
|---|---|---|
| Tenant display name | `Contoso` | `<tenant-display-name>` |
| Tenant ID | `00000000-0000-0000-0000-000000000000` | `<tenant-id>` |
| Initial domain | `contoso.onmicrosoft.com` | `<initial-domain>` |
| Custom domain | `contoso.com` | `<custom-domain>` |
| Affected admin account | `adm.identity@contoso.com` | `<admin-upn>` |
| Break-glass account | `bg-admin01@contoso.onmicrosoft.com` | `<break-glass-upn>` |
| Affected portal | Azure / Entra / M365 / Exchange / SharePoint / Teams / Purview / Defender | `<portal>` |
| Affected command tool | Azure CLI / Az PowerShell / Graph / EXO / SPO / Teams | `<tool>` |
| Azure subscription name | `Contoso-Prod` | `<subscription-name>` |
| Azure subscription ID | `11111111-1111-1111-1111-111111111111` | `<subscription-id>` |
| Error code | `AADSTS53003`, `DeviceNotCompliant`, `Authorization_RequestDenied` | `<error-code>` |
| Correlation ID | `00000000-0000-0000-0000-000000000000` | `<correlation-id>` |
| Request ID | `00000000-0000-0000-0000-000000000000` | `<request-id>` |
| Failure timestamp | `2026-06-15 10:00 EST` | `<failure-time>` |
| Client/device | Edge / Chrome / PowerShell / iPhone / PAW / Cloud Shell | `<client-device>` |
| Network/IP | Public IP or named location | `<network-ip>` |
| Expected role | Global Reader / User Admin / Exchange Admin / Owner / Reader | `<expected-role>` |
| Current role | Role discovered during troubleshooting | `<current-role>` |
| Conditional Access policy involved | `CA-Require-Compliant-Device-Admins` | `<ca-policy>` |
| Service health status | Healthy / Advisory / Incident | `<service-health-status>` |
| Evidence path | `C:\Cloud-Core-Admin\08-Cloud-Admin-Troubleshooting` | `<evidence-path>` |
| Fix type | Context switch / role assignment / CA change / module update / DNS correction / wait for service incident | `<fix-type>` |

# Troubleshoot_Cloud_Admin_Access_Portal_Login_Domain_And_Tooling_Issues_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Create evidence folder | Admin Workstation | `New-Item -ItemType Directory -Force -Path "C:\Cloud-Core-Admin\08-Cloud-Admin-Troubleshooting"` | Evidence folder exists |
| 2 | Record exact symptom before changing anything | Operator | Fill planning table | Error, portal/tool, account, time, and tenant are documented |
| 3 | Capture screenshot/error text if safe | Admin Workstation | Browser screenshot or copy error details | Error evidence exists |
| 4 | Check Microsoft 365 service health | Browser | `https://admin.microsoft.com` > Health > Service health | Known incidents/advisories are ruled in/out |
| 5 | Check Microsoft 365 Message center | Browser | `https://admin.microsoft.com` > Health > Message center | Recent service changes are ruled in/out |
| 6 | Confirm affected account UPN and tenant | Admin Workstation | `Get-MgContext`; browser account menu | Account and tenant are known |
| 7 | Confirm sign-in logs for affected event | Entra admin center | Entra ID > Monitoring & health > Sign-in logs | Matching sign-in event is found |
| 8 | Record request ID/correlation ID | Browser / Entra | Error page / sign-in event | IDs are captured for troubleshooting |
| 9 | Review Conditional Access tab on sign-in event | Entra admin center | Sign-in logs > event > Conditional Access | Blocking or grant-control policy is identified |
| 10 | Review authentication details | Entra admin center | Sign-in logs > event > Authentication details | MFA/authentication method behavior is known |
| 11 | Review device details | Entra admin center | Sign-in logs > event > Device info | Device compliance/join state is known |
| 12 | Review resource/application pair | Entra admin center | Sign-in logs > event > Basic info | App and resource target are known |
| 13 | Check wrong tenant/directory state | Browser / CLI | Directory switcher, `az account tenant list` | Wrong tenant is ruled in/out |
| 14 | Check wrong Azure subscription state | Azure CLI / Az | `az account show`; `Get-AzContext` | Correct subscription context is confirmed |
| 15 | Check role plane and scope | Entra / Azure / M365 | Role inventory commands | Required role exists in correct plane |
| 16 | Check domain/UPN namespace state | Graph / DNS | `Get-MgDomain`; `Resolve-DnsName` | Domain and DNS state are known |
| 17 | Check Azure CLI authentication | Admin Workstation | `az login`; `az account show` | Azure CLI account context works |
| 18 | Check Az PowerShell authentication | Admin Workstation | `Connect-AzAccount`; `Get-AzContext` | Az PowerShell context works |
| 19 | Check Graph PowerShell authentication | Admin Workstation | `Connect-MgGraph`; `Get-MgContext` | Graph account/scopes are known |
| 20 | Check Exchange Online PowerShell if workload is Exchange | Admin Workstation | `Connect-ExchangeOnline`; `Get-ConnectionInformation` | EXO session works |
| 21 | Check SharePoint Online PowerShell if workload is SharePoint | Admin Workstation | `Connect-SPOService`; `Get-SPOTenant` | SPO session works |
| 22 | Check Teams PowerShell if workload is Teams | Admin Workstation | `Connect-MicrosoftTeams`; `Get-CsTenant` | Teams session works |
| 23 | Check Purview/Defender role plane if security/compliance portal fails | Browser | Purview / Defender permissions | Security/compliance permission gap is identified |
| 24 | Clear stale browser session if context is wrong | Browser | InPrivate / sign out / clear site data | Fresh portal session tests correctly |
| 25 | Clear stale CLI token if context is wrong | Admin Workstation | `az logout`; `az login --tenant "<tenant-id>"` | Azure CLI retests cleanly |
| 26 | Clear stale Az PowerShell token if context is wrong | Admin Workstation | `Disconnect-AzAccount`; `Connect-AzAccount -Tenant "<tenant-id>"` | Az PowerShell retests cleanly |
| 27 | Clear stale Graph token if scopes/context are wrong | Admin Workstation | `Disconnect-MgGraph`; reconnect with required scopes | Graph retests cleanly |
| 28 | Use break-glass account only if tenant lockout is likely | Secure workstation | Sign in with approved emergency process | Tenant control is restored safely |
| 29 | Apply smallest safe fix | Operator | Role/context/CA/DNS/tooling correction | Issue is corrected without broad privilege |
| 30 | Retest original failure path | Browser / Tool | Same portal or command | Original issue is resolved or narrowed |
| 31 | Capture final evidence | Admin Workstation | Run evidence capture skeleton | Before/after evidence is saved |
| 32 | Document root cause and prevention | Operator | Update failure notes | Root cause, fix, and prevention are recorded |

# Troubleshoot_Cloud_Admin_Access_Portal_Login_Domain_And_Tooling_Issues_Initial_Triage_Skeleton
```text
Purpose:
  Identify the troubleshooting lane before changing roles, CA policies, DNS, or tooling.

Triage questions:
  1. Is the issue affecting one user or many users?
  2. Is the issue affecting one portal/tool or all Microsoft portals/tools?
  3. Is the user blocked before sign-in completes?
  4. Does sign-in succeed but the portal says access denied?
  5. Is the admin in the right tenant?
  6. Is the Azure subscription visible?
  7. Does the user have the correct role plane?
  8. Is Conditional Access blocking or requiring a missing condition?
  9. Is MFA/authentication method registration failing?
  10. Is the device compliant/joined if policy requires it?
  11. Is a custom domain or UPN suffix involved?
  12. Is Microsoft reporting a service incident?
  13. Did Message center announce a change related to this behavior?
  14. Is the failure only in CLI/PowerShell and not the portal?
  15. Is the failure only in one workload, such as Exchange, SharePoint, Teams, Defender, or Purview?

Decision tree:
  If many users and many services fail:
    Check Service health first.
    Check network/DNS/proxy next.
    Check broad Conditional Access changes.
    Check tenant-wide role/policy changes.

  If one user fails before sign-in:
    Check sign-in logs.
    Check MFA/auth method.
    Check CA policy result.
    Check account enabled state.
    Check password/reset state.

  If one user signs in but cannot administer:
    Check role plane.
    Check scope.
    Check license/workload availability.
    Check portal-specific role requirements.

  If Azure portal opens but no subscriptions show:
    Check Azure RBAC.
    Check directory switcher.
    Check `az account list --all`.
    Check subscription tenant association.

  If M365 admin opens but workload portal fails:
    Check workload role.
    Check workload license.
    Check workload service health.
    Check workload PowerShell.

  If CLI/PowerShell fails:
    Check module version.
    Check login method.
    Check tenant/subscription context.
    Check token cache.
    Check required Graph scopes.
    Check execution policy/proxy/TLS if module install fails.

Record:
  Most likely lane:
  Evidence collected:
  Next action:
```

# Troubleshoot_Cloud_Admin_Access_Portal_Login_Domain_And_Tooling_Issues_Portal_Login_Skeleton
```text
Purpose:
  Troubleshoot browser-based portal login and admin center access.

Portals:
  Azure portal:
    https://portal.azure.com

  Microsoft Entra admin center:
    https://entra.microsoft.com

  Microsoft 365 admin center:
    https://admin.microsoft.com

  Exchange admin center:
    https://admin.exchange.microsoft.com

  SharePoint admin center:
    https://<tenant>-admin.sharepoint.com

  Teams admin center:
    https://admin.teams.microsoft.com

  Microsoft Defender portal:
    https://security.microsoft.com

  Microsoft Purview portal:
    https://purview.microsoft.com

Steps:
  1. Open the portal in a normal browser session.
  2. Record the signed-in account shown in the account menu.
  3. Record the tenant/directory shown in the portal.
  4. If the issue persists, open the same portal in InPrivate/Private mode.
  5. Sign in with the intended admin account.
  6. If prompted, capture error code, request ID, correlation ID, timestamp, app, and resource.
  7. If access denied after successful sign-in, record exact portal area.
  8. Check whether the admin can open:
       https://myaccount.microsoft.com
       https://myapps.microsoft.com
  9. Check whether other admin portals work.
  10. Check Service health.
  11. Check Message center.
  12. Check Entra sign-in logs for the failed event.
  13. Review Conditional Access tab.
  14. Review Authentication details.
  15. Review Device info.
  16. Review Troubleshooting and support tab.
  17. Compare required role against actual role.

Common fixes:
  Wrong account:
    Sign out and use intended admin account.

  Wrong tenant:
    Switch directory or use tenant-specific login.

  Stale browser token:
    Use InPrivate session, clear site data, or sign out of all Microsoft sessions.

  Missing role:
    Assign least-privilege role in correct role plane.

  Conditional Access block:
    Fix target/exclusion/grant control after emergency access review.

  Service incident:
    Document incident and avoid unnecessary local changes.

Record:
  Portal:
  URL:
  Account:
  Tenant:
  Error:
  Correlation ID:
  Request ID:
  Time:
  CA result:
  Role issue:
  Service health issue:
  Fix:
```

# Troubleshoot_Cloud_Admin_Access_Portal_Login_Domain_And_Tooling_Issues_Wrong_Tenant_And_Subscription_Skeleton
```powershell id="jm43z5"
# Run in PowerShell.
# Purpose: detect wrong tenant, wrong directory, and wrong Azure subscription context.

$EvidencePath = "C:\Cloud-Core-Admin\08-Cloud-Admin-Troubleshooting"
$ExpectedTenantId = "<tenant-id>"
$ExpectedSubscriptionId = "<subscription-id>"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\wrong-tenant-subscription-triage-transcript.txt"

# Azure CLI context.
if (Get-Command az -ErrorAction SilentlyContinue) {
  az account show --output json |
    Out-File "$EvidencePath\az-account-show.json"

  az account show --output table |
    Tee-Object "$EvidencePath\az-account-show.txt"

  az account tenant list --output table |
    Tee-Object "$EvidencePath\az-tenant-list.txt"

  az account list --all --output table |
    Tee-Object "$EvidencePath\az-account-list-all.txt"

  $CurrentAzTenant = az account show --query tenantId --output tsv 2>$null
  $CurrentAzSubscription = az account show --query id --output tsv 2>$null

  Write-Output "Expected tenant: $ExpectedTenantId"
  Write-Output "Current Azure CLI tenant: $CurrentAzTenant"
  Write-Output "Expected subscription: $ExpectedSubscriptionId"
  Write-Output "Current Azure CLI subscription: $CurrentAzSubscription"

  if ($CurrentAzTenant -ne $ExpectedTenantId) {
    Write-Output "Azure CLI tenant mismatch. Re-login with tenant:"
    Write-Output "az logout"
    Write-Output "az login --tenant $ExpectedTenantId"
  }

  if ($CurrentAzSubscription -ne $ExpectedSubscriptionId) {
    Write-Output "Azure CLI subscription mismatch. Set subscription:"
    Write-Output "az account set --subscription $ExpectedSubscriptionId"
  }
}

# Az PowerShell context.
try {
  Get-AzContext |
    Format-List * |
    Out-File "$EvidencePath\get-azcontext.txt"

  Get-AzTenant |
    Select-Object Name,Id |
    Format-Table -AutoSize |
    Out-File "$EvidencePath\get-aztenant.txt"

  Get-AzSubscription |
    Select-Object Name,Id,TenantId,State |
    Format-Table -AutoSize |
    Out-File "$EvidencePath\get-azsubscription.txt"

  $CurrentAzPsContext = Get-AzContext

  Write-Output "Current Az PowerShell tenant:"
  Write-Output $CurrentAzPsContext.Tenant.Id

  Write-Output "Current Az PowerShell subscription:"
  Write-Output $CurrentAzPsContext.Subscription.Id

  if ($CurrentAzPsContext.Tenant.Id -ne $ExpectedTenantId) {
    Write-Output "Az PowerShell tenant mismatch. Reconnect with:"
    Write-Output "Disconnect-AzAccount"
    Write-Output "Connect-AzAccount -Tenant $ExpectedTenantId"
  }

  if ($CurrentAzPsContext.Subscription.Id -ne $ExpectedSubscriptionId) {
    Write-Output "Az PowerShell subscription mismatch. Set context:"
    Write-Output "Set-AzContext -SubscriptionId $ExpectedSubscriptionId"
  }
}
catch {
  $_ | Out-File "$EvidencePath\azps-context-error.txt"
}

# Microsoft Graph context.
try {
  Get-MgContext |
    Format-List * |
    Out-File "$EvidencePath\get-mgcontext.txt"

  $GraphTenantId = (Get-MgContext).TenantId

  Write-Output "Current Graph tenant:"
  Write-Output $GraphTenantId

  if ($GraphTenantId -ne $ExpectedTenantId) {
    Write-Output "Graph tenant mismatch. Reconnect with:"
    Write-Output "Disconnect-MgGraph"
    Write-Output "Connect-MgGraph -TenantId $ExpectedTenantId -Scopes '<required-scopes>'"
  }
}
catch {
  $_ | Out-File "$EvidencePath\graph-context-error.txt"
}

Stop-Transcript
```

# Troubleshoot_Cloud_Admin_Access_Portal_Login_Domain_And_Tooling_Issues_Role_And_Scope_Skeleton
```powershell id="b65sa7"
# Run in PowerShell.
# Purpose: troubleshoot access denied after sign-in by checking role plane and scope.
# This is read-only unless you uncomment assignment/removal commands.

$EvidencePath = "C:\Cloud-Core-Admin\08-Cloud-Admin-Troubleshooting"
$AffectedUserUpn = "<admin-upn>"
$SubscriptionId = "<subscription-id>"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\role-and-scope-triage-transcript.txt"

# Azure RBAC check.
az account set --subscription $SubscriptionId

az role assignment list `
  --assignee $AffectedUserUpn `
  --all `
  --include-inherited `
  --query "[].{PrincipalName:principalName,PrincipalType:principalType,Role:roleDefinitionName,Scope:scope}" `
  --output table |
  Tee-Object "$EvidencePath\azure-rbac-for-affected-user.txt"

# Az PowerShell RBAC check.
Connect-AzAccount
Set-AzContext -SubscriptionId $SubscriptionId

Get-AzRoleAssignment -SignInName $AffectedUserUpn -ErrorAction SilentlyContinue |
  Select-Object DisplayName,SignInName,ObjectType,RoleDefinitionName,Scope |
  Format-Table -AutoSize |
  Out-File "$EvidencePath\get-azroleassignment-affected-user.txt"

# Microsoft Entra / Microsoft 365 directory role check.
Import-Module Microsoft.Graph.Identity.DirectoryManagement
Import-Module Microsoft.Graph.Users

Connect-MgGraph -Scopes "RoleManagement.Read.Directory","Directory.Read.All","User.Read.All"

$User = Get-MgUser -UserId $AffectedUserUpn

Get-MgContext |
  Format-List * |
  Out-File "$EvidencePath\get-mgcontext-role-triage.txt"

# Active directory role memberships.
$DirectoryRoles = Get-MgDirectoryRole -All

$RoleRows = foreach ($Role in $DirectoryRoles) {
  $Members = Get-MgDirectoryRoleMember -DirectoryRoleId $Role.Id -All -ErrorAction SilentlyContinue

  foreach ($Member in $Members) {
    if ($Member.Id -eq $User.Id) {
      [PSCustomObject]@{
        UserPrincipalName = $AffectedUserUpn
        RoleName          = $Role.DisplayName
        RoleId            = $Role.Id
        RoleTemplateId    = $Role.RoleTemplateId
      }
    }
  }
}

$RoleRows |
  Format-Table -AutoSize |
  Tee-Object "$EvidencePath\entra-directory-roles-for-affected-user.txt"

# Raw role assignment inventory for deeper review.
Get-MgRoleManagementDirectoryRoleAssignment -All |
  Select-Object Id,PrincipalId,RoleDefinitionId,DirectoryScopeId,AppScopeId |
  Export-Csv "$EvidencePath\directory-role-assignments-raw.csv" -NoTypeInformation

Disconnect-MgGraph

# Role plane notes:
# Azure resources:
#   Check Azure RBAC at management group, subscription, resource group, or resource scope.
#
# Entra user/group/domain/CA/app tasks:
#   Check Entra directory roles.
#
# Microsoft 365 admin center:
#   Check Microsoft 365 admin roles.
#
# Exchange:
#   Check Exchange admin role and Exchange Online role groups.
#
# SharePoint:
#   Check SharePoint Administrator and site-level permissions.
#
# Teams:
#   Check Teams Administrator or narrower Teams roles.
#
# Defender/Purview:
#   Check security/compliance role plane and workload-specific RBAC.

Stop-Transcript
```

# Troubleshoot_Cloud_Admin_Access_Portal_Login_Domain_And_Tooling_Issues_Conditional_Access_Skeleton
```text id="f4lv14"
Purpose:
  Troubleshoot sign-in failures caused by Conditional Access, MFA, device compliance, app protection, location, or risk controls.

Portal:
  https://entra.microsoft.com

Primary path:
  1. Go to Entra ID.
  2. Go to Monitoring & health.
  3. Go to Sign-in logs.
  4. Filter by:
       UserPrincipalName: <admin-upn>
       Date/time around failure
       Correlation ID if available
       Application if known
       Conditional Access: Failure if needed
  5. Open the matching sign-in event.
  6. Review Basic info:
       User
       Application
       Resource
       IP address
       Location
       Client app
       Browser
       Operating system
       Correlation ID
       Request ID
  7. Review Conditional Access tab:
       Applied policies
       Failed policies
       Not applied policies
       Grant controls
       Session controls
  8. Review Authentication details:
       MFA required
       MFA satisfied
       Method used
       Failure reason
  9. Review Device info:
       Device ID
       Join type
       Compliance state
       Managed state
  10. Review Troubleshooting and support tab:
       Failure reason
       Sign-in diagnostic
       Support details
  11. If policy behavior is unexpected, run:
       Conditional Access What If tool

Common risky CA patterns:
  Block all users and all cloud apps.
  Require compliant device for all users before devices are enrolled.
  Require hybrid joined device for all users without hybrid join ready.
  Require approved app/app protection for all users including admins.
  Target all cloud apps without excluding emergency access accounts.
  Target admin portals but forget Azure Resource Manager dependency.
  Block legacy auth but workload still depends on old client.
  Named location mismatch due to VPN/proxy/mobile network.
  Guest/external user blocked by user type condition.
  Report-only policy misunderstood as enforced policy.

Safe fix sequence:
  1. Use break-glass only if tenant lockout risk exists.
  2. Confirm emergency account is excluded from blocking policies.
  3. Do not disable all CA blindly.
  4. Prefer report-only or targeted exclusion for the affected admin/test group.
  5. Fix the smallest assignment/grant/session condition.
  6. Re-test sign-in.
  7. Review sign-in logs again.
  8. Document policy change and reason.

Record:
  Error code:
  User:
  App:
  Resource:
  Device:
  Location:
  Policy blocked:
  Grant control failed:
  Session control failed:
  Fix:
  Retest result:
```

# Troubleshoot_Cloud_Admin_Access_Portal_Login_Domain_And_Tooling_Issues_Domain_And_DNS_Skeleton
```powershell id="y75sni"
# Run in PowerShell.
# Purpose: troubleshoot domain, UPN suffix, DNS verification, and Microsoft 365 namespace issues.

$EvidencePath = "C:\Cloud-Core-Admin\08-Cloud-Admin-Troubleshooting"
$CustomDomain = "contoso.com"
$AffectedUserUpn = "<admin-upn>"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\domain-dns-triage-transcript.txt"

# Microsoft Graph domain/user state.
Import-Module Microsoft.Graph.Identity.DirectoryManagement
Import-Module Microsoft.Graph.Users

Connect-MgGraph -Scopes "Domain.Read.All","Directory.Read.All","User.Read.All"

Get-MgContext |
  Format-List * |
  Out-File "$EvidencePath\get-mgcontext-domain-triage.txt"

Get-MgDomain |
  Select-Object Id,IsDefault,IsInitial,IsVerified,AuthenticationType,SupportedServices |
  Format-Table -AutoSize |
  Tee-Object "$EvidencePath\mg-domains.txt"

Get-MgDomain -DomainId $CustomDomain -ErrorAction SilentlyContinue |
  Format-List * |
  Out-File "$EvidencePath\mg-domain-$CustomDomain.txt"

Get-MgUser -UserId $AffectedUserUpn -Property Id,DisplayName,UserPrincipalName,Mail,ProxyAddresses,AccountEnabled,UserType,OnPremisesSyncEnabled |
  Select-Object Id,DisplayName,UserPrincipalName,Mail,ProxyAddresses,AccountEnabled,UserType,OnPremisesSyncEnabled |
  Format-List * |
  Out-File "$EvidencePath\affected-user-domain-state.txt"

Disconnect-MgGraph

# Public DNS state.
Resolve-DnsName -Name $CustomDomain -Type NS -ErrorAction SilentlyContinue |
  Format-List * |
  Out-File "$EvidencePath\dns-ns-$CustomDomain.txt"

Resolve-DnsName -Name $CustomDomain -Type TXT -ErrorAction SilentlyContinue |
  Format-List * |
  Out-File "$EvidencePath\dns-txt-$CustomDomain.txt"

Resolve-DnsName -Name $CustomDomain -Type MX -ErrorAction SilentlyContinue |
  Format-List * |
  Out-File "$EvidencePath\dns-mx-$CustomDomain.txt"

Resolve-DnsName -Name "autodiscover.$CustomDomain" -Type CNAME -ErrorAction SilentlyContinue |
  Format-List * |
  Out-File "$EvidencePath\dns-autodiscover-$CustomDomain.txt"

Resolve-DnsName -Name "_dmarc.$CustomDomain" -Type TXT -ErrorAction SilentlyContinue |
  Format-List * |
  Out-File "$EvidencePath\dns-dmarc-$CustomDomain.txt"

# Resolver comparison.
foreach ($Resolver in @("1.1.1.1","8.8.8.8","9.9.9.9")) {
  Resolve-DnsName -Server $Resolver -Name $CustomDomain -Type TXT -ErrorAction SilentlyContinue |
    Out-File "$EvidencePath\dns-txt-$CustomDomain-$Resolver.txt"

  Resolve-DnsName -Server $Resolver -Name $CustomDomain -Type MX -ErrorAction SilentlyContinue |
    Out-File "$EvidencePath\dns-mx-$CustomDomain-$Resolver.txt"
}

Stop-Transcript
```

# Troubleshoot_Cloud_Admin_Access_Portal_Login_Domain_And_Tooling_Issues_Azure_CLI_Skeleton
```powershell id="bfdlqq"
# Run in PowerShell.
# Purpose: troubleshoot Azure CLI install, login, tenant, and subscription context.

$EvidencePath = "C:\Cloud-Core-Admin\08-Cloud-Admin-Troubleshooting"
$ExpectedTenantId = "<tenant-id>"
$ExpectedSubscriptionId = "<subscription-id>"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\azure-cli-troubleshooting-transcript.txt"

# Check Azure CLI exists.
Get-Command az -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\get-command-az.txt"

# Version.
az version |
  Out-File "$EvidencePath\az-version.json"

# Clear token only if stale context is suspected.
# az logout

# Login options:
# Standard interactive:
# az login
#
# Tenant-specific:
# az login --tenant $ExpectedTenantId
#
# Device code for browser/WAM issues:
# az login --tenant $ExpectedTenantId --use-device-code

# Current context.
az account show --output json |
  Out-File "$EvidencePath\az-account-show.json"

az account show --output table |
  Tee-Object "$EvidencePath\az-account-show.txt"

# All tenants/subscriptions visible.
az account tenant list --output table |
  Tee-Object "$EvidencePath\az-tenant-list.txt"

az account list --all --output table |
  Tee-Object "$EvidencePath\az-account-list-all.txt"

# Set expected subscription.
az account set --subscription $ExpectedSubscriptionId

# Confirm final context.
az account show `
  --query "{Name:name,SubscriptionId:id,TenantId:tenantId,State:state,User:user.name}" `
  --output table |
  Tee-Object "$EvidencePath\az-final-context.txt"

# Basic authorization test.
az group list --output table |
  Tee-Object "$EvidencePath\az-group-list.txt"

# Azure RBAC for signed-in user may not resolve cleanly for all account types.
# If access denied, check role assignments at correct scope.

Stop-Transcript
```

# Troubleshoot_Cloud_Admin_Access_Portal_Login_Domain_And_Tooling_Issues_Az_PowerShell_Skeleton
```powershell id="z4bi3z"
# Run in PowerShell 7 when possible.
# Purpose: troubleshoot Az PowerShell install, login, tenant, and subscription context.

$EvidencePath = "C:\Cloud-Core-Admin\08-Cloud-Admin-Troubleshooting"
$ExpectedTenantId = "<tenant-id>"
$ExpectedSubscriptionId = "<subscription-id>"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\az-powershell-troubleshooting-transcript.txt"

# PowerShell and module state.
$PSVersionTable |
  Out-File "$EvidencePath\psversiontable.txt"

Get-ExecutionPolicy -List |
  Out-File "$EvidencePath\execution-policy.txt"

Get-Module Az -ListAvailable |
  Select-Object Name,Version,Path |
  Sort-Object Version -Descending |
  Out-File "$EvidencePath\az-module-list.txt"

Get-Module AzureRM -ListAvailable |
  Select-Object Name,Version,Path |
  Out-File "$EvidencePath\azurerm-module-list.txt"

# Install/update if needed.
# Install-Module Az -Scope CurrentUser -Repository PSGallery -Force

# Clear context only if stale context is suspected.
# Disconnect-AzAccount

# Tenant-specific connection.
Connect-AzAccount -Tenant $ExpectedTenantId

# Context capture.
Get-AzContext |
  Format-List * |
  Tee-Object "$EvidencePath\get-azcontext.txt"

Get-AzTenant |
  Select-Object Name,Id |
  Format-Table -AutoSize |
  Out-File "$EvidencePath\get-aztenant.txt"

Get-AzSubscription |
  Select-Object Name,Id,TenantId,State |
  Format-Table -AutoSize |
  Out-File "$EvidencePath\get-azsubscription.txt"

# Set expected subscription.
Set-AzContext -SubscriptionId $ExpectedSubscriptionId

# Confirm final context.
Get-AzContext |
  Select-Object Account,Tenant,Subscription,Environment |
  Format-List * |
  Tee-Object "$EvidencePath\azps-final-context.txt"

# Basic authorization test.
Get-AzResourceGroup |
  Select-Object ResourceGroupName,Location,ProvisioningState |
  Format-Table -AutoSize |
  Tee-Object "$EvidencePath\get-azresourcegroup.txt"

Stop-Transcript
```

# Troubleshoot_Cloud_Admin_Access_Portal_Login_Domain_And_Tooling_Issues_Graph_PowerShell_Skeleton
```powershell id="w7kfmy"
# Run in PowerShell 7 when possible.
# Purpose: troubleshoot Microsoft Graph PowerShell install, login, scopes, and tenant context.

$EvidencePath = "C:\Cloud-Core-Admin\08-Cloud-Admin-Troubleshooting"
$ExpectedTenantId = "<tenant-id>"

$RequiredScopes = @(
  "Directory.Read.All",
  "User.Read.All",
  "Group.Read.All",
  "Organization.Read.All",
  "AuditLog.Read.All",
  "RoleManagement.Read.Directory"
)

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\graph-powershell-troubleshooting-transcript.txt"

# PowerShell and module state.
$PSVersionTable |
  Out-File "$EvidencePath\graph-psversiontable.txt"

Get-Module Microsoft.Graph* -ListAvailable |
  Select-Object Name,Version,Path |
  Sort-Object Name,Version |
  Out-File "$EvidencePath\microsoft-graph-module-list.txt"

# Install if needed.
# Install-Module Microsoft.Graph -Scope CurrentUser -Repository PSGallery -Force

# Disconnect stale session if needed.
try {
  Disconnect-MgGraph -ErrorAction SilentlyContinue
}
catch {
  Write-Output "No existing Graph session or disconnect failed harmlessly."
}

# Connect to expected tenant with required read scopes.
Connect-MgGraph -TenantId $ExpectedTenantId -Scopes $RequiredScopes

# Capture context.
Get-MgContext |
  Format-List * |
  Tee-Object "$EvidencePath\get-mgcontext.txt"

Get-MgContext |
  Select-Object -ExpandProperty Scopes |
  Out-File "$EvidencePath\mg-scopes.txt"

# Validate tenant access.
Get-MgOrganization |
  Select-Object Id,DisplayName |
  Format-Table -AutoSize |
  Tee-Object "$EvidencePath\mg-organization.txt"

Get-MgDomain |
  Select-Object Id,IsDefault,IsInitial,IsVerified |
  Format-Table -AutoSize |
  Tee-Object "$EvidencePath\mg-domains.txt"

# Validate directory reads.
Get-MgUser -Top 5 |
  Select-Object Id,DisplayName,UserPrincipalName,AccountEnabled |
  Format-Table -AutoSize |
  Out-File "$EvidencePath\mg-users-top5.txt"

Get-MgGroup -Top 5 |
  Select-Object Id,DisplayName,SecurityEnabled,MailEnabled |
  Format-Table -AutoSize |
  Out-File "$EvidencePath\mg-groups-top5.txt"

# Common Graph failures:
# Authorization_RequestDenied:
#   Connected but lacks role or scope.
#
# Insufficient privileges:
#   Scope missing, consent missing, or admin role missing.
#
# Empty result:
#   Wrong tenant, filter issue, or insufficient permission.
#
# Command not found:
#   Missing Graph submodule or module not imported.

Disconnect-MgGraph

Stop-Transcript
```

# Troubleshoot_Cloud_Admin_Access_Portal_Login_Domain_And_Tooling_Issues_Workload_PowerShell_Skeleton
```powershell id="f3axw9"
# Run in PowerShell.
# Purpose: troubleshoot workload-specific PowerShell access for Exchange Online, SharePoint Online, and Teams.

$EvidencePath = "C:\Cloud-Core-Admin\08-Cloud-Admin-Troubleshooting"
$AdminUpn = "<admin-upn>"
$SpoAdminUrl = "https://<tenant>-admin.sharepoint.com"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\workload-powershell-troubleshooting-transcript.txt"

# Module inventory.
Get-Module ExchangeOnlineManagement -ListAvailable |
  Select-Object Name,Version,Path |
  Out-File "$EvidencePath\module-exchangeonlinemanagement.txt"

Get-Module Microsoft.Online.SharePoint.PowerShell -ListAvailable |
  Select-Object Name,Version,Path |
  Out-File "$EvidencePath\module-spo.txt"

Get-Module MicrosoftTeams -ListAvailable |
  Select-Object Name,Version,Path |
  Out-File "$EvidencePath\module-teams.txt"

# Exchange Online test.
try {
  Import-Module ExchangeOnlineManagement
  Connect-ExchangeOnline -UserPrincipalName $AdminUpn

  Get-ConnectionInformation |
    Format-List * |
    Out-File "$EvidencePath\exo-connection-information.txt"

  Get-OrganizationConfig |
    Select-Object Name,Guid |
    Format-List * |
    Out-File "$EvidencePath\exo-organization-config.txt"

  Get-RoleGroupMember -Identity "Organization Management" -ErrorAction SilentlyContinue |
    Select-Object Name,RecipientType,PrimarySmtpAddress |
    Format-Table -AutoSize |
    Out-File "$EvidencePath\exo-organization-management-members.txt"

  Disconnect-ExchangeOnline -Confirm:$false
}
catch {
  $_ | Out-File "$EvidencePath\exo-troubleshooting-error.txt"
}

# SharePoint Online test.
try {
  if ($PSVersionTable.PSVersion.Major -ge 7) {
    Import-Module Microsoft.Online.SharePoint.PowerShell -UseWindowsPowerShell
  }
  else {
    Import-Module Microsoft.Online.SharePoint.PowerShell
  }

  Connect-SPOService -Url $SpoAdminUrl

  Get-SPOTenant |
    Format-List * |
    Out-File "$EvidencePath\spo-tenant.txt"

  Get-SPOSite -Limit 10 |
    Select-Object Url,Owner,Template,Status |
    Format-Table -AutoSize |
    Out-File "$EvidencePath\spo-sites-top10.txt"
}
catch {
  $_ | Out-File "$EvidencePath\spo-troubleshooting-error.txt"
}

# Teams test.
try {
  Import-Module MicrosoftTeams
  Connect-MicrosoftTeams

  Get-CsTenant |
    Format-List * |
    Out-File "$EvidencePath\teams-tenant.txt"

  Get-CsTeamsMeetingPolicy |
    Select-Object Identity,Description |
    Format-Table -AutoSize |
    Out-File "$EvidencePath\teams-meeting-policies.txt"

  Disconnect-MicrosoftTeams
}
catch {
  $_ | Out-File "$EvidencePath\teams-troubleshooting-error.txt"
}

Stop-Transcript
```

# Troubleshoot_Cloud_Admin_Access_Portal_Login_Domain_And_Tooling_Issues_Service_Health_And_Message_Center_Skeleton
```powershell id="vc8fmc"
# Run in PowerShell 7 when possible.
# Purpose: rule out Microsoft service incidents and Message center changes before local fixes.

$EvidencePath = "C:\Cloud-Core-Admin\08-Cloud-Admin-Troubleshooting"
New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\service-health-message-center-triage-transcript.txt"

Import-Module Microsoft.Graph.ServiceAnnouncement

Connect-MgGraph -Scopes "ServiceHealth.Read.All","ServiceMessage.Read.All"

Get-MgContext |
  Format-List * |
  Out-File "$EvidencePath\get-mgcontext-service-health.txt"

# Service health overview.
Get-MgAdminServiceAnnouncementHealthOverview |
  Select-Object Id,Service,Status |
  Sort-Object Service |
  Format-Table -AutoSize |
  Tee-Object "$EvidencePath\service-health-overview.txt"

# Current service issues.
Get-MgAdminServiceAnnouncementIssue |
  Select-Object Id,Title,Service,Status,Classification,StartDateTime,EndDateTime,LastModifiedDateTime |
  Sort-Object LastModifiedDateTime -Descending |
  Format-Table -AutoSize |
  Tee-Object "$EvidencePath\service-issues.txt"

# Message center recent messages.
Get-MgAdminServiceAnnouncementMessage -Top 100 |
  Select-Object Id,Title,Category,Severity,Services,StartDateTime,EndDateTime,LastModifiedDateTime,IsMajorChange |
  Sort-Object LastModifiedDateTime -Descending |
  Export-Csv "$EvidencePath\message-center-top100.csv" -NoTypeInformation

Get-MgAdminServiceAnnouncementMessage -Top 50 |
  Select-Object Id,Title,Category,Severity,LastModifiedDateTime,IsMajorChange |
  Sort-Object LastModifiedDateTime -Descending |
  Format-Table -AutoSize |
  Tee-Object "$EvidencePath\message-center-top50.txt"

Disconnect-MgGraph

Stop-Transcript
```

# Troubleshoot_Cloud_Admin_Access_Portal_Login_Domain_And_Tooling_Issues_Evidence_Capture_Skeleton
```powershell id="x29srh"
# Run in PowerShell.
# Purpose: capture broad troubleshooting evidence for cloud admin access, login, domain, and tooling issues.

$EvidencePath = "C:\Cloud-Core-Admin\08-Cloud-Admin-Troubleshooting"
$AffectedUserUpn = "<admin-upn>"
$CustomDomain = "contoso.com"
$SubscriptionId = "<subscription-id>"
$TenantId = "<tenant-id>"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\final-troubleshooting-evidence-transcript.txt"

# Local workstation context.
hostname | Out-File "$EvidencePath\hostname.txt"
whoami | Out-File "$EvidencePath\whoami.txt"
$PSVersionTable | Out-File "$EvidencePath\psversiontable.txt"
Get-ExecutionPolicy -List | Out-File "$EvidencePath\execution-policy.txt"

# Module inventory.
Get-Module Az,Microsoft.Graph,ExchangeOnlineManagement,Microsoft.Online.SharePoint.PowerShell,MicrosoftTeams -ListAvailable |
  Select-Object Name,Version,Path |
  Sort-Object Name,Version |
  Out-File "$EvidencePath\module-inventory.txt"

# Azure CLI evidence.
if (Get-Command az -ErrorAction SilentlyContinue) {
  az version | Out-File "$EvidencePath\az-version.json"
  az account show --output json | Out-File "$EvidencePath\az-account-show.json"
  az account list --all --output table | Out-File "$EvidencePath\az-account-list-all.txt"
}

# Az PowerShell evidence.
try {
  Get-AzContext | Format-List * | Out-File "$EvidencePath\get-azcontext.txt"
  Get-AzSubscription | Select-Object Name,Id,TenantId,State | Export-Csv "$EvidencePath\get-azsubscription.csv" -NoTypeInformation
}
catch {
  $_ | Out-File "$EvidencePath\azps-evidence-error.txt"
}

# Microsoft Graph evidence.
try {
  Connect-MgGraph -TenantId $TenantId -Scopes "Directory.Read.All","User.Read.All","Group.Read.All","AuditLog.Read.All","RoleManagement.Read.Directory","Domain.Read.All"

  Get-MgContext | Format-List * | Out-File "$EvidencePath\get-mgcontext.txt"

  Get-MgOrganization |
    Select-Object Id,DisplayName |
    Format-List * |
    Out-File "$EvidencePath\mg-organization.txt"

  Get-MgDomain |
    Select-Object Id,IsDefault,IsInitial,IsVerified,AuthenticationType |
    Format-Table -AutoSize |
    Out-File "$EvidencePath\mg-domains.txt"

  Get-MgUser -UserId $AffectedUserUpn -Property Id,DisplayName,UserPrincipalName,AccountEnabled,UserType,OnPremisesSyncEnabled,UsageLocation |
    Select-Object Id,DisplayName,UserPrincipalName,AccountEnabled,UserType,OnPremisesSyncEnabled,UsageLocation |
    Format-List * |
    Out-File "$EvidencePath\affected-user.txt"

  Get-MgAuditLogSignIn -Top 50 |
    Select-Object CreatedDateTime,UserPrincipalName,AppDisplayName,IpAddress,Status,ConditionalAccessStatus,CorrelationId |
    Export-Csv "$EvidencePath\signinlog-top50.csv" -NoTypeInformation

  Get-MgAuditLogDirectoryAudit -Top 50 |
    Select-Object ActivityDateTime,ActivityDisplayName,Category,Result,InitiatedBy,TargetResources,CorrelationId |
    Export-Csv "$EvidencePath\directoryaudit-top50.csv" -NoTypeInformation

  Disconnect-MgGraph
}
catch {
  $_ | Out-File "$EvidencePath\graph-evidence-error.txt"
}

# DNS evidence.
Resolve-DnsName -Name $CustomDomain -Type NS -ErrorAction SilentlyContinue |
  Out-File "$EvidencePath\dns-ns.txt"

Resolve-DnsName -Name $CustomDomain -Type TXT -ErrorAction SilentlyContinue |
  Out-File "$EvidencePath\dns-txt.txt"

Resolve-DnsName -Name $CustomDomain -Type MX -ErrorAction SilentlyContinue |
  Out-File "$EvidencePath\dns-mx.txt"

Resolve-DnsName -Name "autodiscover.$CustomDomain" -Type CNAME -ErrorAction SilentlyContinue |
  Out-File "$EvidencePath\dns-autodiscover.txt"

Stop-Transcript
```

# Troubleshoot_Cloud_Admin_Access_Portal_Login_Domain_And_Tooling_Issues_Verification_Commands
```powershell id="eph6a8"
# Local workstation.
whoami
hostname
$PSVersionTable
Get-ExecutionPolicy -List
Get-Module Az,Microsoft.Graph,ExchangeOnlineManagement,Microsoft.Online.SharePoint.PowerShell,MicrosoftTeams -ListAvailable

# Portal checks:
# https://admin.microsoft.com
# https://entra.microsoft.com
# https://portal.azure.com
# https://admin.exchange.microsoft.com
# https://<tenant>-admin.sharepoint.com
# https://admin.teams.microsoft.com
# https://security.microsoft.com
# https://purview.microsoft.com

# Service health and Message center:
# Microsoft 365 admin center > Health > Service health
# Microsoft 365 admin center > Health > Message center

# Entra sign-in logs:
# Entra admin center > Entra ID > Monitoring & health > Sign-in logs
# Filter by:
#   User
#   Date/time
#   Correlation ID
#   Application
#   Resource
# Review:
#   Conditional Access
#   Authentication details
#   Device info
#   Troubleshooting and support

# Azure CLI.
az version
az login --tenant "<tenant-id>"
az account show -o table
az account tenant list -o table
az account list --all -o table
az account set --subscription "<subscription-id>"
az group list -o table

# Az PowerShell.
Connect-AzAccount -Tenant "<tenant-id>"
Get-AzContext
Get-AzTenant
Get-AzSubscription
Set-AzContext -SubscriptionId "<subscription-id>"
Get-AzResourceGroup

# Microsoft Graph PowerShell.
Connect-MgGraph -TenantId "<tenant-id>" -Scopes "Directory.Read.All","User.Read.All","Group.Read.All","AuditLog.Read.All","RoleManagement.Read.Directory","Domain.Read.All"
Get-MgContext
Get-MgOrganization
Get-MgDomain
Get-MgUser -UserId "<admin-upn>"
Get-MgAuditLogSignIn -Top 10
Disconnect-MgGraph

# Role plane checks.
az role assignment list --assignee "<admin-upn>" --all --include-inherited -o table
Get-AzRoleAssignment -SignInName "<admin-upn>"
Connect-MgGraph -Scopes "RoleManagement.Read.Directory","Directory.Read.All","User.Read.All"
Get-MgRoleManagementDirectoryRoleAssignment -All
Get-MgDirectoryRole -All
Disconnect-MgGraph

# Domain and DNS checks.
Resolve-DnsName -Name "contoso.com" -Type NS
Resolve-DnsName -Name "contoso.com" -Type TXT
Resolve-DnsName -Name "contoso.com" -Type MX
Resolve-DnsName -Name "autodiscover.contoso.com" -Type CNAME

# Exchange Online.
Connect-ExchangeOnline -UserPrincipalName "<admin-upn>"
Get-ConnectionInformation
Get-OrganizationConfig
Disconnect-ExchangeOnline -Confirm:$false

# SharePoint Online.
Connect-SPOService -Url "https://<tenant>-admin.sharepoint.com"
Get-SPOTenant

# Teams.
Connect-MicrosoftTeams
Get-CsTenant
Disconnect-MicrosoftTeams
```

# Troubleshoot_Cloud_Admin_Access_Portal_Login_Domain_And_Tooling_Issues_Rollback
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm what was changed during troubleshooting | Operator | Review transcript/evidence | Changed items are known |
| 2 | Revert temporary Azure RBAC assignment if used | Admin Workstation | `Remove-AzRoleAssignment -ObjectId "<object-id>" -RoleDefinitionName "<role>" -Scope "<scope>"` | Temporary Azure role is removed |
| 3 | Revert temporary Entra role assignment if used | Entra / Graph | Remove role assignment through approved role workflow | Temporary directory role is removed |
| 4 | Revert temporary M365/workload role assignment if used | M365 / Workload portal | Remove role/group membership | Temporary workload role is removed |
| 5 | Re-enable Conditional Access policy if disabled for emergency fix | Entra admin center | Conditional Access > Policy > Enable | Policy state is restored |
| 6 | Remove temporary CA user/group exclusion | Entra admin center | Conditional Access > Users > Exclude | Temporary exclusion is removed |
| 7 | Restore previous DNS record if changed incorrectly | DNS host | Restore TXT/MX/CNAME/SPF from evidence | DNS returns to prior state |
| 8 | Restore previous default domain if changed incorrectly | Graph / Entra | `Update-MgDomain -DomainId "<prior-domain>" -BodyParameter @{ isDefault = $true }` | Default domain returns to previous state |
| 9 | Clear temporary CLI/Az/Graph sessions | Admin Workstation | `az logout`; `Disconnect-AzAccount`; `Disconnect-MgGraph` | Tokens are cleared |
| 10 | Disconnect workload sessions | Admin Workstation | `Disconnect-ExchangeOnline`; `Disconnect-MicrosoftTeams`; close SPO session | Workload sessions close |
| 11 | Rotate emergency credentials if break-glass was used outside drill | Entra / Secure vault | Reset password/key per process | Emergency credentials are protected |
| 12 | Preserve evidence for incident/problem review | Secure location | Move evidence folder to secure storage | Troubleshooting record is retained |
| 13 | Delete local evidence if sensitive and no longer needed | Admin Workstation | `Remove-Item "C:\Cloud-Core-Admin\08-Cloud-Admin-Troubleshooting" -Recurse -Force` | Local evidence is removed |
| 14 | Document final root cause | Operator | Update workbook notes | Root cause and prevention are captured |

# Troubleshoot_Cloud_Admin_Access_Portal_Login_Domain_And_Tooling_Issues_Failure_Checks
| Symptom | Likely Cause | Device | PowerShell / Command | Fix |
|---|---|---|---|---|
| Portal loops between sign-in pages | Stale browser session, third-party cookie issue, wrong account cache, WAM/browser mismatch | Browser | InPrivate session, clear Microsoft site data | Sign out fully, use InPrivate, try Edge, clear cached account |
| Portal says access denied after sign-in | Missing role in correct role plane | Portal / Graph | Check Entra/M365/Azure/workload role assignments | Assign least-privilege role in correct plane |
| Azure portal opens but no subscriptions show | User has tenant access but no Azure RBAC or wrong directory | Azure portal / CLI | `az account list --all -o table` | Switch directory or assign Azure RBAC at correct scope |
| Microsoft 365 admin center unavailable | Missing M365 admin role or service incident | Browser | Service health, M365 roles | Assign correct role or wait/escalate incident |
| Entra admin center unavailable | Missing directory role or CA block | Browser / Entra logs | Sign-in logs and role assignments | Fix role, tenant context, or CA policy |
| Exchange admin center unavailable | Missing Exchange role or license/workload issue | Browser / EXO | `Connect-ExchangeOnline`; `Get-RoleGroupMember` | Assign Exchange Administrator or workload RBAC as needed |
| SharePoint admin center unavailable | Wrong admin URL or missing SharePoint Administrator role | Browser / SPO | `Connect-SPOService -Url "https://<tenant>-admin.sharepoint.com"` | Use correct URL and role |
| Teams admin center unavailable | Missing Teams role or Teams service issue | Browser / Teams PS | `Connect-MicrosoftTeams`; `Get-CsTenant` | Assign Teams role or check service health |
| Defender portal unavailable | Missing Security role or Defender RBAC configuration | Browser | Defender permissions | Assign correct security role or Defender RBAC |
| Purview portal unavailable | Missing compliance role group or Purview permission | Browser | Purview permissions | Add user to correct Purview role group |
| Sign-in blocked by Conditional Access | Policy requires unmet condition or block grant | Entra sign-in logs | Check Conditional Access tab | Adjust policy target/grant/exclusion safely |
| Error shows `AADSTS53003` | Blocked by Conditional Access | Entra sign-in logs | Filter by user/time/correlation ID | Review blocking CA policy |
| Error shows device not compliant | Device compliance required but device not compliant | Entra sign-in logs / Intune | Device info tab | Use compliant device or adjust policy |
| Error shows device not domain joined | CA requires hybrid joined device | Entra sign-in logs | Device info tab | Use compliant/hybrid device or adjust policy |
| MFA prompt fails | Auth method missing, unavailable, blocked, or registration incomplete | Entra / User | Authentication details tab | Register/fix authentication method |
| User cannot register SSPR/MFA | User not in policy scope or method disabled | Entra admin center | Auth method and SSPR settings | Add user to scope and enable approved methods |
| User disabled | Account disabled | Graph / Entra | `Get-MgUser -UserId "<upn>" -Property AccountEnabled` | Re-enable if approved |
| Password expired/unknown | Credential issue | Entra / M365 admin center | Password reset | Reset password or use SSPR |
| Custom domain UPN fails | Domain not verified or user has wrong UPN suffix | Graph / DNS | `Get-MgDomain`; `Get-MgUser` | Verify domain or fix UPN |
| Domain verification fails | TXT/MX record missing or wrong DNS host | DNS / Graph | `Resolve-DnsName -Type TXT`; `Get-MgDomainVerificationDnsRecord` | Add correct verification record at authoritative DNS |
| Mail breaks after domain setup | MX changed before Exchange readiness | DNS / EXO | `Resolve-DnsName -Type MX`; `Get-AcceptedDomain` | Restore previous MX or complete mail cutover |
| Azure CLI `az` not recognized | Azure CLI missing or terminal not reopened | Admin Workstation | `Get-Command az` | Install Azure CLI and reopen terminal |
| Azure CLI logged into wrong tenant | Cached account or default tenant mismatch | Admin Workstation | `az account show`; `az account tenant list` | `az logout`; `az login --tenant "<tenant-id>"` |
| Azure CLI wrong subscription | Default subscription mismatch | Admin Workstation | `az account list --all`; `az account set` | Set intended subscription |
| Az PowerShell wrong tenant | Cached context or wrong login | Admin Workstation | `Get-AzContext` | `Disconnect-AzAccount`; `Connect-AzAccount -Tenant "<tenant-id>"` |
| Az module install fails | PSGallery/proxy/TLS/execution policy issue | Admin Workstation | `Get-PSRepository`; `Get-ExecutionPolicy -List` | Fix repository/proxy/policy or use approved offline install |
| Graph connects but command fails | Missing Graph scope or admin consent | Admin Workstation | `Get-MgContext` | Reconnect with required scopes and authorized account |
| Graph connected to wrong tenant | Missing tenant parameter or cached context | Admin Workstation | `Get-MgContext` | `Disconnect-MgGraph`; `Connect-MgGraph -TenantId "<tenant-id>"` |
| Graph cmdlet missing | Graph module/submodule missing | Admin Workstation | `Get-Module Microsoft.Graph* -ListAvailable` | Install/update Microsoft Graph module |
| Exchange cmdlet missing | Not connected to Exchange Online or module missing | Admin Workstation | `Get-Module ExchangeOnlineManagement` | Install module and `Connect-ExchangeOnline` |
| SPO module fails in PowerShell 7 | SharePoint module compatibility issue | Admin Workstation | Import with `-UseWindowsPowerShell` | Use Windows PowerShell 5.1 or compatibility import |
| Teams cmdlet fails | Old module, missing role, or auth issue | Admin Workstation | `Get-Module MicrosoftTeams -ListAvailable` | Update module and verify Teams role |
| Service health shows incident | Microsoft-side service incident | M365 admin center | Health > Service health | Track incident and avoid unnecessary tenant changes |
| Message center shows breaking change | Microsoft change affects workflow | M365 admin center | Health > Message center | Follow required action and update runbook |
| Evidence missing correlation ID | Error page not captured before retry | Browser / Sign-in logs | Search by user/time/app | Capture details before clearing sessions next time |

# Troubleshoot_Cloud_Admin_Access_Portal_Login_Domain_And_Tooling_Issues_Related_Labs
| Lab                                                                                | Relationship                                                                                                   |
| ---------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------- |
| 01_Compare_Azure_Tenant_M365_Tenant_Subscriptions_And_Directories.md               | Provides the tenant, directory, subscription, and service boundary map used to diagnose wrong-context problems |
| 02_Configure_Admin_Portals_PowerShell_Azure_CLI_And_Graph_Baseline.md              | Provides baseline portal and tool setup used to isolate tooling failures                                       |
| 03_Configure_Custom_Domains_DNS_Verification_And_Tenant_Namespaces.md              | Provides domain and DNS baseline used to diagnose UPN, verification, and mail routing problems                 |
| 04_Compare_Azure_RBAC_Entra_Roles_M365_Admin_Roles_And_Workload_Roles.md           | Provides role plane model used to diagnose access denied after successful sign-in                              |
| 05_Configure_Cloud_Admin_Accounts_Break_Glass_And_Privileged_Access_Baseline.md    | Provides emergency access recovery path for tenant lockout or bad Conditional Access changes                   |
| 06_Configure_Entra_Users_Groups_Guests_Licenses_And_SSPR_Baseline.md               | Provides user, group, guest, license, and SSPR context for identity troubleshooting                            |
| 07_Configure_Cloud_Audit_Logs_Service_Health_Message_Center_And_Change_Tracking.md | Provides sign-in logs, audit logs, service health, Message center, and alerting used during troubleshooting    |
