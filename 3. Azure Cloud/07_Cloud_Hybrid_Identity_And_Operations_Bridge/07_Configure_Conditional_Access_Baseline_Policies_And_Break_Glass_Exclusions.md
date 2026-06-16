# 07_Configure_Conditional_Access_Baseline_Policies_And_Break_Glass_Exclusions

# 07_Configure_Conditional_Access_Baseline_Policies_And_Break_Glass_Exclusions_Index

07_Configure_Conditional_Access_Baseline_Policies_And_Break_Glass_Exclusions.md  
07_Configure_Conditional_Access_Baseline_Policies_And_Break_Glass_Exclusions  
07_Configure_Conditional_Access_Baseline_Policies_And_Break_Glass_Exclusions_Source_Basis  
07_Configure_Conditional_Access_Baseline_Policies_And_Break_Glass_Exclusions_Mental_Model  
07_Configure_Conditional_Access_Baseline_Policies_And_Break_Glass_Exclusions_Planning_Table  
07_Configure_Conditional_Access_Baseline_Policies_And_Break_Glass_Exclusions_Configuration_Checklist  
07_Configure_Conditional_Access_Baseline_Policies_And_Break_Glass_Exclusions_Baseline_Policy_Map  
07_Configure_Conditional_Access_Baseline_Policies_And_Break_Glass_Exclusions_Tenant_And_License_Precheck_Skeleton  
07_Configure_Conditional_Access_Baseline_Policies_And_Break_Glass_Exclusions_Break_Glass_Account_Baseline_Skeleton  
07_Configure_Conditional_Access_Baseline_Policies_And_Break_Glass_Exclusions_Named_Locations_Skeleton  
07_Configure_Conditional_Access_Baseline_Policies_And_Break_Glass_Exclusions_Report_Only_Policy_Build_Skeleton  
07_Configure_Conditional_Access_Baseline_Policies_And_Break_Glass_Exclusions_Conditional_Access_Portal_Runbook_Skeleton  
07_Configure_Conditional_Access_Baseline_Policies_And_Break_Glass_Exclusions_Policy_Validation_And_SignIn_Log_Skeleton  
07_Configure_Conditional_Access_Baseline_Policies_And_Break_Glass_Exclusions_Verification_Commands  
07_Configure_Conditional_Access_Baseline_Policies_And_Break_Glass_Exclusions_Rollback  
07_Configure_Conditional_Access_Baseline_Policies_And_Break_Glass_Exclusions_Failure_Checks  
07_Configure_Conditional_Access_Baseline_Policies_And_Break_Glass_Exclusions_Related_Labs  

# 07_Configure_Conditional_Access_Baseline_Policies_And_Break_Glass_Exclusions_Source_Basis

| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Entra Docs | Conditional Access overview | Core policy model, assignments, conditions, access controls, session controls, and report-only rollout |
| Microsoft Entra Docs | Conditional Access common policies | Baseline policies such as require MFA for admins, require MFA for users, block legacy authentication, and require compliant or hybrid joined devices |
| Microsoft Entra Docs | Conditional Access templates | Template-driven policy creation for common security baselines |
| Microsoft Entra Docs | Emergency access accounts | Break glass account design, exclusions, monitoring, and validation |
| Microsoft Entra Docs | Named locations | Trusted locations, country locations, IP ranges, and location-based policy conditions |
| Microsoft Entra Docs | Sign-in logs | Validating Conditional Access result, report-only result, failure reason, and applied policy |
| Microsoft Entra Docs | What If tool | Testing expected Conditional Access decisions before enforcement |
| Microsoft Graph PowerShell | Conditional Access policy inventory | Exporting existing CA policies, named locations, users, groups, roles, and sign-in evidence |
| Microsoft 365 Docs | Protect privileged accounts | Dedicated admin accounts, MFA, emergency access, and least privilege |
| Operational practice | Report-only first, pilot group first, exclude break glass, document every exclusion | Prevents self-lockout and tenant-wide access outages |

# 07_Configure_Conditional_Access_Baseline_Policies_And_Break_Glass_Exclusions_Mental_Model

| Concept | Operational Meaning |
|---|---|
| Conditional Access | Policy engine that decides whether a sign-in is allowed, blocked, challenged, or controlled |
| Assignment | Who and what the policy targets, such as users, groups, roles, apps, and workload identities |
| Condition | Context such as location, device platform, client app, sign-in risk, user risk, or authentication context |
| Grant control | Requirement before access is granted, such as MFA, compliant device, hybrid joined device, or approved client app |
| Session control | Controls after sign-in, such as sign-in frequency, persistent browser session, or app-enforced restrictions |
| Report-only mode | Evaluation mode that logs policy impact without enforcing the control |
| On mode | Enforced policy state that affects real user access |
| Off mode | Disabled policy state |
| Security defaults | Tenant-wide Microsoft baseline security setting that cannot be used together with Conditional Access |
| Legacy authentication | Older protocols that do not support modern MFA, often blocked by baseline CA policy |
| Named location | Defined IP range or country location used by Conditional Access location conditions |
| Trusted location | Named location marked as trusted, often corporate egress IPs |
| Break glass account | Emergency cloud-only account used when normal admin access fails |
| Exclusion | Explicit user, group, role, location, or app exception that prevents a policy from applying |
| Least privilege | Assign only the minimum admin role and access needed |
| Policy collision | Multiple policies apply and produce unexpected combined result |
| What If tool | Conditional Access simulator used to predict policy application |
| Sign-in log | Source of truth for which CA policies applied, failed, or would have applied |
| First rule | Never enforce a tenant-wide block or MFA policy without tested break glass exclusions |
| Blunt rule | Report-only is not optional unless you are deliberately accepting lockout risk |

# 07_Configure_Conditional_Access_Baseline_Policies_And_Break_Glass_Exclusions_Planning_Table

| Item | Example | Decision |
|---|---|---|
| Tenant domain | `contoso.onmicrosoft.com` | `<tenant-domain>` |
| Verified domain | `contoso.com` | `<verified-domain>` |
| Tenant ID | `00000000-0000-0000-0000-000000000000` | `<tenant-id>` |
| Required license | Microsoft Entra ID P1 or P2 | `<license-baseline>` |
| Admin role required | Conditional Access Administrator / Security Administrator / Global Administrator | `<admin-role>` |
| Primary admin account | `cloudadmin@contoso.com` | `<primary-admin>` |
| Break glass account 1 | `breakglass1@contoso.onmicrosoft.com` | `<breakglass-account-1>` |
| Break glass account 2 | `breakglass2@contoso.onmicrosoft.com` | `<breakglass-account-2>` |
| Break glass auth method | FIDO2 / certificate / long password stored offline | `<breakglass-auth-method>` |
| CA pilot group | `GRP-CA-Pilot-Users` | `<ca-pilot-group>` |
| Admin CA group | `GRP-CA-Admins` | `<admin-ca-group>` |
| Exclusion group | `GRP-CA-Excluded-BreakGlass` | `<exclusion-group>` |
| Named trusted location | `Corp-Egress-IPs` | `<trusted-location-name>` |
| Corporate public IP ranges | `203.0.113.10/32` | `<trusted-ip-ranges>` |
| Blocked countries | `<country-list>` | `<blocked-countries>` |
| Target cloud apps | All cloud apps / Office 365 / Azure management | `<target-apps>` |
| Baseline policy state | Report-only first | `<initial-policy-state>` |
| MFA method readiness | Authenticator / FIDO2 / TAP | `<mfa-readiness>` |
| Device compliance dependency | Intune compliant device | `<device-compliance-dependency>` |
| Legacy authentication handling | Block | `<legacy-auth-decision>` |
| Guest access handling | Require MFA or block | `<guest-access-decision>` |
| Sign-in risk policy | Require MFA for medium and high risk | `<sign-in-risk-decision>` |
| User risk policy | Require password change for high risk | `<user-risk-decision>` |
| Evidence folder | `C:\ConditionalAccess-Baseline` | `<evidence-path>` |
| Rollback stance | Set policies to report-only or off | `<rollback-plan>` |

# 07_Configure_Conditional_Access_Baseline_Policies_And_Break_Glass_Exclusions_Configuration_Checklist

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm SSPR and MFA methods are ready | Admin Workstation | `Review task 06 evidence` | Pilot users and admins have registered methods |
| 2 | Create evidence folder | Admin Workstation | `New-Item -ItemType Directory -Force -Path C:\ConditionalAccess-Baseline` | Evidence folder exists |
| 3 | Connect to Microsoft Graph | Admin Workstation | `Connect-MgGraph -Scopes "Directory.Read.All","Policy.Read.All","Policy.ReadWrite.ConditionalAccess","AuditLog.Read.All","User.Read.All","Group.Read.All"` | Graph session connects |
| 4 | Export current Conditional Access policies | Admin Workstation | `Get-MgIdentityConditionalAccessPolicy \| Export-Csv C:\ConditionalAccess-Baseline\ca-policies-before.csv -NoTypeInformation` | Current policy inventory exists |
| 5 | Export named locations | Admin Workstation | `Get-MgIdentityConditionalAccessNamedLocation \| Export-Csv C:\ConditionalAccess-Baseline\named-locations-before.csv -NoTypeInformation` | Existing named locations are documented |
| 6 | Check security defaults state | Admin Workstation | `Get-MgPolicyIdentitySecurityDefaultEnforcementPolicy` | Security defaults state is known |
| 7 | Confirm CA licensing | Entra Admin Center | `Billing / Licenses / Entra admin center` | Required Entra ID license exists |
| 8 | Create pilot group | Entra Admin Center / Graph | `GRP-CA-Pilot-Users` | Pilot group exists |
| 9 | Create admin scope group | Entra Admin Center / Graph | `GRP-CA-Admins` | Admin group exists |
| 10 | Create break glass exclusion group | Entra Admin Center / Graph | `GRP-CA-Excluded-BreakGlass` | Exclusion group exists |
| 11 | Confirm break glass accounts are cloud-only | Admin Workstation | `Get-MgUser -UserId "<breakglass-upn>" -Property OnPremisesSyncEnabled` | Break glass accounts are not synced |
| 12 | Add break glass accounts to exclusion group | Entra Admin Center / Graph | `New-MgGroupMember` | Break glass exclusion membership is set |
| 13 | Validate break glass sign-in before policy creation | Browser / Private Window | `Sign in as breakglass account` | Emergency account can sign in |
| 14 | Configure named trusted locations | Entra Admin Center | `Protection > Conditional Access > Named locations` | Corporate IP ranges are defined |
| 15 | Create baseline policy: require MFA for admins | Entra Admin Center | `Report-only first` | Admin MFA policy exists in report-only |
| 16 | Create baseline policy: require MFA for all users or pilot group | Entra Admin Center | `Report-only first` | User MFA policy exists in report-only |
| 17 | Create baseline policy: block legacy authentication | Entra Admin Center | `Report-only first` | Legacy auth block policy exists in report-only |
| 18 | Create baseline policy: require MFA for Azure management | Entra Admin Center | `Target Microsoft Azure Management` | Azure management MFA policy exists |
| 19 | Create baseline policy: require MFA for guests | Entra Admin Center | `Target guest and external users` | Guest MFA policy exists |
| 20 | Create baseline policy: require compliant device or approved app if ready | Entra Admin Center | `Pilot only` | Device-based policy exists only if dependencies are ready |
| 21 | Create risk-based policies if P2 is available | Entra Admin Center | `Sign-in risk and user risk policies` | Risk-based policies exist in report-only |
| 22 | Exclude break glass accounts from every CA policy | Entra Admin Center | `Users > Exclude > break glass group` | Emergency access remains outside normal policy scope |
| 23 | Exclude service accounts only when documented | Entra Admin Center | `Users > Exclude > approved service account group` | Service account exceptions are intentional |
| 24 | Validate with What If tool | Entra Admin Center | `Conditional Access > What If` | Expected policies apply to pilot users |
| 25 | Validate report-only impact | Entra Admin Center | `Sign-in logs > Report-only tab` | No unexpected lockout appears |
| 26 | Test pilot user sign-in | Browser / Test device | `Sign in to Microsoft 365 app` | User receives expected MFA or control |
| 27 | Test admin sign-in | Admin Workstation | `Sign in to admin center` | Admin receives expected MFA |
| 28 | Test legacy auth block simulation or controlled test | Test Client | `Legacy auth attempt if safe` | Legacy auth is blocked or would be blocked |
| 29 | Move policies from report-only to on in phases | Entra Admin Center | `Policy state: On` | Only validated policies are enforced |
| 30 | Export final policies | Admin Workstation | `Get-MgIdentityConditionalAccessPolicy \| ConvertTo-Json -Depth 20` | Final policy backup exists |
| 31 | Monitor sign-in logs after enforcement | Entra Admin Center / Graph | `Sign-in logs` | No unexpected broad failures |
| 32 | Document exclusions and owner | Operator | `Record every excluded account, group, app, and location` | Exceptions are accountable |
| 33 | Schedule break glass validation | Operator | `Monthly or quarterly validation` | Emergency access is periodically tested |

# 07_Configure_Conditional_Access_Baseline_Policies_And_Break_Glass_Exclusions_Baseline_Policy_Map

| Policy Name | Target | Exclusions | Conditions | Grant / Session Control | Initial State |
|---|---|---|---|---|---|
| `CA001-Require-MFA-For-Admins` | Directory roles or admin group | Break glass group | All cloud apps or admin portals | Require MFA | Report-only |
| `CA002-Require-MFA-For-All-Users` | Pilot group, then all users | Break glass group, service exceptions | All cloud apps | Require MFA | Report-only |
| `CA003-Block-Legacy-Authentication` | All users | Break glass group, documented legacy exceptions | Client apps: legacy auth | Block access | Report-only |
| `CA004-Require-MFA-For-Azure-Management` | Admins and operators | Break glass group | Microsoft Azure Management | Require MFA | Report-only |
| `CA005-Require-MFA-For-Guest-Users` | Guest and external users | Emergency exceptions only | All cloud apps or selected apps | Require MFA | Report-only |
| `CA006-Require-Compliant-Device-For-Sensitive-Apps` | Pilot users | Break glass group | Selected sensitive apps | Require compliant device | Report-only |
| `CA007-Block-High-Risk-SignIns` | Pilot users or all users if P2 ready | Break glass group | Sign-in risk: high | Block or require MFA | Report-only |
| `CA008-Require-Password-Change-For-High-Risk-Users` | Pilot users or all users if P2 ready | Break glass group | User risk: high | Require password change | Report-only |
| `CA009-Block-Unknown-Or-Unsupported-Countries` | All users or pilot group | Break glass group, travel exceptions | Selected countries | Block access | Report-only |
| `CA010-Session-Control-For-Admin-Portals` | Admins | Break glass group | Admin portals | Sign-in frequency and persistent session control | Report-only |

# 07_Configure_Conditional_Access_Baseline_Policies_And_Break_Glass_Exclusions_Tenant_And_License_Precheck_Skeleton

```powershell
# Run from admin workstation with Microsoft Graph PowerShell.
# Purpose: capture tenant state, licensing signals, CA policies, named locations, users, groups, and security defaults before changes.

$EvidencePath = "C:\ConditionalAccess-Baseline"
New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\01-tenant-license-ca-precheck-transcript.txt"

Import-Module Microsoft.Graph.Authentication
Import-Module Microsoft.Graph.Identity.DirectoryManagement
Import-Module Microsoft.Graph.Identity.SignIns
Import-Module Microsoft.Graph.Users
Import-Module Microsoft.Graph.Groups

Connect-MgGraph -Scopes `
  "Directory.Read.All",
  "Organization.Read.All",
  "Policy.Read.All",
  "Policy.ReadWrite.ConditionalAccess",
  "User.Read.All",
  "Group.Read.All",
  "AuditLog.Read.All"

# Context.
Get-MgContext |
  Format-List * |
  Out-File "$EvidencePath\graph-context.txt"

# Tenant domains.
Get-MgDomain |
  Select-Object Id,IsVerified,IsDefault,AuthenticationType |
  Export-Csv "$EvidencePath\tenant-domains.csv" -NoTypeInformation

# Organization and subscribed SKUs.
Get-MgOrganization |
  Format-List * |
  Out-File "$EvidencePath\organization.txt"

Get-MgSubscribedSku |
  Select-Object SkuPartNumber,ConsumedUnits,PrepaidUnits |
  Export-Csv "$EvidencePath\subscribed-skus.csv" -NoTypeInformation

# Security defaults state.
Get-MgPolicyIdentitySecurityDefaultEnforcementPolicy |
  Format-List * |
  Out-File "$EvidencePath\security-defaults-state.txt"

# Existing CA policies.
Get-MgIdentityConditionalAccessPolicy |
  Select-Object Id,DisplayName,State,CreatedDateTime,ModifiedDateTime |
  Export-Csv "$EvidencePath\conditional-access-policies-before.csv" -NoTypeInformation

Get-MgIdentityConditionalAccessPolicy |
  ConvertTo-Json -Depth 30 |
  Out-File "$EvidencePath\conditional-access-policies-before-full.json"

# Existing named locations.
Get-MgIdentityConditionalAccessNamedLocation |
  Select-Object Id,DisplayName,CreatedDateTime,ModifiedDateTime |
  Export-Csv "$EvidencePath\named-locations-before.csv" -NoTypeInformation

Get-MgIdentityConditionalAccessNamedLocation |
  ConvertTo-Json -Depth 20 |
  Out-File "$EvidencePath\named-locations-before-full.json"

# Users and groups.
Get-MgUser -All `
  -Property Id,DisplayName,UserPrincipalName,AccountEnabled,UserType,OnPremisesSyncEnabled |
  Select-Object Id,DisplayName,UserPrincipalName,AccountEnabled,UserType,OnPremisesSyncEnabled |
  Export-Csv "$EvidencePath\users-before-ca.csv" -NoTypeInformation

Get-MgGroup -All `
  -Property Id,DisplayName,MailEnabled,SecurityEnabled,GroupTypes |
  Select-Object Id,DisplayName,MailEnabled,SecurityEnabled,GroupTypes |
  Export-Csv "$EvidencePath\groups-before-ca.csv" -NoTypeInformation

Disconnect-MgGraph

Stop-Transcript
```

# 07_Configure_Conditional_Access_Baseline_Policies_And_Break_Glass_Exclusions_Break_Glass_Account_Baseline_Skeleton

```powershell
# Run from admin workstation with Microsoft Graph PowerShell.
# Purpose: validate break glass accounts and create a CA exclusion group.

$EvidencePath = "C:\ConditionalAccess-Baseline"
$BreakGlassGroupName = "GRP-CA-Excluded-BreakGlass"
$BreakGlassUpns = @(
  "breakglass1@contoso.onmicrosoft.com",
  "breakglass2@contoso.onmicrosoft.com"
)

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\02-break-glass-baseline-transcript.txt"

Import-Module Microsoft.Graph.Users
Import-Module Microsoft.Graph.Groups
Import-Module Microsoft.Graph.Identity.DirectoryManagement

Connect-MgGraph -Scopes `
  "User.Read.All",
  "Group.ReadWrite.All",
  "Directory.ReadWrite.All",
  "UserAuthenticationMethod.Read.All"

# Create or find exclusion group.
$BreakGlassGroup = Get-MgGroup -Filter "displayName eq '$BreakGlassGroupName'" -ErrorAction SilentlyContinue

if (-not $BreakGlassGroup) {
  $BreakGlassGroup = New-MgGroup `
    -DisplayName $BreakGlassGroupName `
    -MailEnabled:$false `
    -MailNickname "grp-ca-excluded-breakglass" `
    -SecurityEnabled:$true
}

# Validate and add accounts.
$Results = foreach ($Upn in $BreakGlassUpns) {
  $User = Get-MgUser `
    -UserId $Upn `
    -Property Id,DisplayName,UserPrincipalName,AccountEnabled,UserType,OnPremisesSyncEnabled,CreatedDateTime `
    -ErrorAction SilentlyContinue

  if ($User) {
    New-MgGroupMember `
      -GroupId $BreakGlassGroup.Id `
      -DirectoryObjectId $User.Id `
      -ErrorAction SilentlyContinue

    $Methods = Get-MgUserAuthenticationMethod -UserId $User.Id -ErrorAction SilentlyContinue

    [PSCustomObject]@{
      UserPrincipalName = $User.UserPrincipalName
      AccountEnabled = $User.AccountEnabled
      UserType = $User.UserType
      OnPremisesSyncEnabled = $User.OnPremisesSyncEnabled
      MethodCount = ($Methods | Measure-Object).Count
      MethodTypes = (($Methods | ForEach-Object { $_.AdditionalProperties["@odata.type"] }) -join ";")
      InBreakGlassExclusionGroup = $true
    }
  }
  else {
    [PSCustomObject]@{
      UserPrincipalName = $Upn
      AccountEnabled = "NotFound"
      UserType = ""
      OnPremisesSyncEnabled = ""
      MethodCount = ""
      MethodTypes = ""
      InBreakGlassExclusionGroup = $false
    }
  }
}

$Results |
  Export-Csv "$EvidencePath\break-glass-account-validation.csv" -NoTypeInformation

# Export group members.
Get-MgGroupMember -GroupId $BreakGlassGroup.Id -All |
  Select-Object Id |
  Export-Csv "$EvidencePath\break-glass-exclusion-group-members.csv" -NoTypeInformation

Disconnect-MgGraph

Stop-Transcript
```

```text
# Manual break glass validation steps.

1. Open a private browser window.
2. Sign in as breakglass1@contoso.onmicrosoft.com.
3. Confirm the account can reach Microsoft Entra admin center or Microsoft 365 admin center.
4. Sign out.
5. Repeat for breakglass2@contoso.onmicrosoft.com.
6. Confirm passwords or credentials are stored in the approved offline vault.
7. Confirm accounts are:
   - cloud-only
   - excluded from normal CA policies
   - not used for daily admin work
   - monitored with alerting
   - tested on a schedule
8. Record test date, result, and tester.
```

# 07_Configure_Conditional_Access_Baseline_Policies_And_Break_Glass_Exclusions_Named_Locations_Skeleton

```text
# Named locations portal skeleton.

1. Sign in to Microsoft Entra admin center.
2. Go to:
   Protection > Conditional Access > Named locations

3. Create corporate trusted location:
   Name: Corp-Egress-IPs
   Type: IP ranges
   IP ranges:
   - 203.0.113.10/32
   - 198.51.100.0/24
   Mark as trusted:
   - Yes, only if these are controlled corporate egress IPs

4. Create country location if needed:
   Name: Allowed-Countries
   Countries:
   - United States
   Include unknown areas:
   - Based on business decision

5. Create blocked country location if needed:
   Name: Blocked-Countries
   Countries:
   - Approved blocked list
   Include unknown areas:
   - Only if approved

6. Document:
   - who owns the IP list
   - renewal/review interval
   - VPN egress dependencies
   - remote worker impact
   - travel exception process
```

```powershell
# Export named locations before and after portal changes.

$EvidencePath = "C:\ConditionalAccess-Baseline"
New-Item -ItemType Directory -Force -Path $EvidencePath

Import-Module Microsoft.Graph.Identity.SignIns

Connect-MgGraph -Scopes "Policy.Read.All","Policy.ReadWrite.ConditionalAccess"

Get-MgIdentityConditionalAccessNamedLocation |
  Select-Object Id,DisplayName,CreatedDateTime,ModifiedDateTime |
  Export-Csv "$EvidencePath\named-locations-current.csv" -NoTypeInformation

Get-MgIdentityConditionalAccessNamedLocation |
  ConvertTo-Json -Depth 20 |
  Out-File "$EvidencePath\named-locations-current-full.json"

Disconnect-MgGraph
```

# 07_Configure_Conditional_Access_Baseline_Policies_And_Break_Glass_Exclusions_Report_Only_Policy_Build_Skeleton

```text
# Report-only build skeleton.
# Use portal or Graph templates. Portal steps are preferred for initial workbook execution.

Policy 1:
Name:
CA001-Require-MFA-For-Admins

Assignments:
Users:
- Include directory roles:
  - Global Administrator
  - Privileged Role Administrator
  - Conditional Access Administrator
  - Security Administrator
  - Exchange Administrator
  - SharePoint Administrator
  - User Administrator
  - Helpdesk Administrator if required
- Exclude:
  - GRP-CA-Excluded-BreakGlass

Target resources:
- All cloud apps
- Or selected admin portals if using a narrower pilot

Conditions:
- None for baseline
- Optionally exclude trusted locations only if approved

Grant:
- Require multifactor authentication

State:
- Report-only first


Policy 2:
Name:
CA002-Require-MFA-For-All-Users

Assignments:
Users:
- Include:
  - GRP-CA-Pilot-Users first
  - Expand to All users later
- Exclude:
  - GRP-CA-Excluded-BreakGlass
  - documented service account group if required

Target resources:
- All cloud apps

Grant:
- Require multifactor authentication

State:
- Report-only first


Policy 3:
Name:
CA003-Block-Legacy-Authentication

Assignments:
Users:
- Include All users
- Exclude:
  - GRP-CA-Excluded-BreakGlass
  - documented legacy exception group only if required

Target resources:
- All cloud apps

Conditions:
Client apps:
- Exchange ActiveSync clients
- Other clients using legacy authentication

Grant:
- Block access

State:
- Report-only first


Policy 4:
Name:
CA004-Require-MFA-For-Azure-Management

Assignments:
Users:
- Admin group or all admins
- Exclude:
  - GRP-CA-Excluded-BreakGlass

Target resources:
- Microsoft Azure Management

Grant:
- Require multifactor authentication

State:
- Report-only first


Policy 5:
Name:
CA005-Require-MFA-For-Guests

Assignments:
Users:
- Guest or external users
- Exclude:
  - documented emergency exceptions only

Target resources:
- All cloud apps or selected apps

Grant:
- Require multifactor authentication

State:
- Report-only first


Policy 6:
Name:
CA006-Require-Compliant-Device-For-Sensitive-Apps

Assignments:
Users:
- GRP-CA-Pilot-Users

Target resources:
- Selected sensitive apps

Grant:
- Require device to be marked as compliant

State:
- Report-only first


Policy 7:
Name:
CA007-High-Risk-SignIn-Require-MFA-Or-Block

Assignments:
Users:
- Pilot group first
- Exclude:
  - GRP-CA-Excluded-BreakGlass

Conditions:
- Sign-in risk: high
- Add medium if approved

Grant:
- Require MFA
- Or block access if approved

State:
- Report-only first


Policy 8:
Name:
CA008-High-Risk-User-Require-Password-Change

Assignments:
Users:
- Pilot group first
- Exclude:
  - GRP-CA-Excluded-BreakGlass

Conditions:
- User risk: high

Grant:
- Require password change

State:
- Report-only first
```

# 07_Configure_Conditional_Access_Baseline_Policies_And_Break_Glass_Exclusions_Conditional_Access_Portal_Runbook_Skeleton

```text
# Conditional Access portal runbook.

1. Sign in to Microsoft Entra admin center.
2. Go to:
   Protection > Conditional Access > Policies

3. Before creating policies:
   - Confirm security defaults state.
   - Confirm break glass accounts.
   - Confirm break glass exclusion group.
   - Confirm pilot users registered MFA methods.
   - Confirm no existing policy already does the same thing.

4. Create policy:
   Name:
   CA001-Require-MFA-For-Admins

5. Assignments:
   Users:
   - Include selected directory roles or admin group.
   - Exclude GRP-CA-Excluded-BreakGlass.

6. Target resources:
   - All cloud apps, unless narrowing for pilot.

7. Conditions:
   - Keep simple for first baseline.
   - Avoid location conditions unless named locations are verified.

8. Access controls:
   Grant:
   - Require multifactor authentication.

9. Enable policy:
   - Report-only.

10. Save policy.

11. Repeat for:
   - CA002-Require-MFA-For-All-Users
   - CA003-Block-Legacy-Authentication
   - CA004-Require-MFA-For-Azure-Management
   - CA005-Require-MFA-For-Guests
   - CA006-Require-Compliant-Device-For-Sensitive-Apps, only if Intune is ready
   - CA007 and CA008, only if Entra ID P2 and Identity Protection are ready

12. Validate using What If:
   - Pilot user
   - Admin user
   - Break glass account
   - Guest user
   - Trusted location
   - Untrusted location
   - Legacy client condition if applicable

13. Validate using sign-in logs:
   - Report-only tab
   - Conditional Access tab
   - Failure reason
   - Applied policies
   - Report-only policies

14. Move policy to On only after:
   - report-only impact is understood
   - pilot user succeeds
   - break glass remains excluded
   - help desk knows expected prompts
   - rollback is documented

15. Export final policy state.
```

# 07_Configure_Conditional_Access_Baseline_Policies_And_Break_Glass_Exclusions_Policy_Validation_And_SignIn_Log_Skeleton

```powershell
# Run from admin workstation with Microsoft Graph PowerShell.
# Purpose: export Conditional Access policies and recent sign-in logs for validation.

$EvidencePath = "C:\ConditionalAccess-Baseline"
$StartTime = (Get-Date).AddDays(-7).ToString("o")

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\03-ca-policy-validation-signin-log-transcript.txt"

Import-Module Microsoft.Graph.Identity.SignIns
Import-Module Microsoft.Graph.Users
Import-Module Microsoft.Graph.Groups

Connect-MgGraph -Scopes `
  "Policy.Read.All",
  "AuditLog.Read.All",
  "Directory.Read.All",
  "User.Read.All",
  "Group.Read.All"

# Export current CA policy summary.
Get-MgIdentityConditionalAccessPolicy |
  Select-Object Id,DisplayName,State,CreatedDateTime,ModifiedDateTime |
  Export-Csv "$EvidencePath\conditional-access-policies-current.csv" -NoTypeInformation

# Export full policy JSON.
Get-MgIdentityConditionalAccessPolicy |
  ConvertTo-Json -Depth 30 |
  Out-File "$EvidencePath\conditional-access-policies-current-full.json"

# Export named locations.
Get-MgIdentityConditionalAccessNamedLocation |
  ConvertTo-Json -Depth 20 |
  Out-File "$EvidencePath\named-locations-current-full.json"

# Export recent sign-ins.
Get-MgAuditLogSignIn -Filter "createdDateTime ge $StartTime" -All |
  Select-Object `
    CreatedDateTime,
    UserDisplayName,
    UserPrincipalName,
    AppDisplayName,
    ClientAppUsed,
    IpAddress,
    ConditionalAccessStatus,
    Status,
    RiskDetail,
    RiskLevelAggregated,
    RiskLevelDuringSignIn |
  Export-Csv "$EvidencePath\signin-logs-last-7-days.csv" -NoTypeInformation

# Export sign-ins where CA failed or interrupted.
Get-MgAuditLogSignIn -Filter "createdDateTime ge $StartTime" -All |
  Where-Object {
    $_.ConditionalAccessStatus -ne "success" -and
    $_.ConditionalAccessStatus -ne "notApplied"
  } |
  Select-Object `
    CreatedDateTime,
    UserDisplayName,
    UserPrincipalName,
    AppDisplayName,
    ClientAppUsed,
    IpAddress,
    ConditionalAccessStatus,
    Status,
    RiskDetail,
    RiskLevelAggregated,
    RiskLevelDuringSignIn |
  Export-Csv "$EvidencePath\signin-logs-ca-non-success.csv" -NoTypeInformation

Disconnect-MgGraph

Stop-Transcript
```

```text
# Manual What If validation matrix.

Test 1:
User: pilot.user@contoso.com
App: Office 365
Location: untrusted
Expected:
- CA002 would require MFA

Test 2:
User: admin.user@contoso.com
App: Microsoft Azure Management
Location: untrusted
Expected:
- CA001 would require MFA
- CA004 would require MFA

Test 3:
User: breakglass1@contoso.onmicrosoft.com
App: Microsoft Entra admin center
Location: untrusted
Expected:
- Normal CA policies do not apply because account is excluded
- Account can still sign in

Test 4:
User: guest.user_external#EXT#@contoso.onmicrosoft.com
App: SharePoint Online
Location: untrusted
Expected:
- CA005 would require MFA

Test 5:
User: legacy.test@contoso.com
Client app: legacy authentication
Expected:
- CA003 would block or report block

Test 6:
User: pilot.user@contoso.com
App: sensitive app
Device: not compliant
Expected:
- CA006 would block or require compliant device if policy is active
```

# 07_Configure_Conditional_Access_Baseline_Policies_And_Break_Glass_Exclusions_Verification_Commands

| Command | Purpose | Healthy Output |
|---|---|---|
| `Get-MgPolicyIdentitySecurityDefaultEnforcementPolicy` | Checks security defaults state | State is known and documented |
| `Get-MgIdentityConditionalAccessPolicy` | Lists CA policies | Expected baseline policies appear |
| `Get-MgIdentityConditionalAccessPolicy \| ConvertTo-Json -Depth 30` | Exports full CA policy JSON | Full backup exists |
| `Get-MgIdentityConditionalAccessNamedLocation` | Lists named locations | Trusted and country locations appear if configured |
| `Get-MgGroup -Filter "displayName eq 'GRP-CA-Excluded-BreakGlass'"` | Confirms break glass exclusion group | Group exists |
| `Get-MgGroupMember -GroupId <group-id>` | Confirms break glass group membership | Break glass accounts appear |
| `Get-MgUser -UserId "breakglass1@contoso.onmicrosoft.com" -Property OnPremisesSyncEnabled` | Confirms break glass account is cloud-only | `OnPremisesSyncEnabled` is empty or false |
| `Get-MgAuditLogSignIn -Top 10` | Confirms sign-in log access | Recent sign-ins return |
| `Get-MgAuditLogSignIn -Filter "userPrincipalName eq 'pilot.user@contoso.com'"` | Checks pilot user CA results | CA status and applied policy details are visible |
| `What If tool` | Simulates CA application | Expected policy set appears |
| `Private browser break glass sign-in` | Confirms emergency access | Break glass account can sign in |
| `Pilot user sign-in test` | Confirms user MFA policy behavior | User receives expected prompt |
| `Admin portal sign-in test` | Confirms admin MFA policy behavior | Admin receives expected prompt |
| `Legacy auth test` | Confirms legacy protocol block | Legacy auth is blocked or report-only block appears |
| `Sign-in logs > Conditional Access tab` | Reviews applied and report-only policies | Expected policies apply, unexpected policies do not |

# 07_Configure_Conditional_Access_Baseline_Policies_And_Break_Glass_Exclusions_Rollback

```powershell
# Conditional Access emergency rollback.
# Use only with approval or during active lockout risk.
# This changes selected policies to report-only or disabled based on names.

$EvidencePath = "C:\ConditionalAccess-Baseline\Rollback"
New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\ca-rollback-transcript.txt"

Import-Module Microsoft.Graph.Identity.SignIns

Connect-MgGraph -Scopes "Policy.ReadWrite.ConditionalAccess","Policy.Read.All"

# Export before rollback.
Get-MgIdentityConditionalAccessPolicy |
  ConvertTo-Json -Depth 30 |
  Out-File "$EvidencePath\ca-policies-before-rollback.json"

# Policy names to move back to report-only.
$PoliciesToReportOnly = @(
  "CA001-Require-MFA-For-Admins",
  "CA002-Require-MFA-For-All-Users",
  "CA003-Block-Legacy-Authentication",
  "CA004-Require-MFA-For-Azure-Management",
  "CA005-Require-MFA-For-Guests",
  "CA006-Require-Compliant-Device-For-Sensitive-Apps",
  "CA007-High-Risk-SignIn-Require-MFA-Or-Block",
  "CA008-High-Risk-User-Require-Password-Change"
)

foreach ($PolicyName in $PoliciesToReportOnly) {
  $Policy = Get-MgIdentityConditionalAccessPolicy |
    Where-Object { $_.DisplayName -eq $PolicyName }

  if ($Policy) {
    Update-MgIdentityConditionalAccessPolicy `
      -ConditionalAccessPolicyId $Policy.Id `
      -State "enabledForReportingButNotEnforced"
  }
}

# Export after rollback.
Get-MgIdentityConditionalAccessPolicy |
  ConvertTo-Json -Depth 30 |
  Out-File "$EvidencePath\ca-policies-after-rollback.json"

Disconnect-MgGraph

Stop-Transcript
```

```text
# Portal rollback procedure.

1. Use break glass account if normal admin account is blocked.
2. Sign in to Microsoft Entra admin center.
3. Go to:
   Protection > Conditional Access > Policies
4. Sort by recently modified.
5. Open the policy causing impact.
6. Change Enable policy to:
   - Report-only, preferred rollback
   - Off, if report-only does not remove impact or active outage exists
7. Save.
8. Validate affected user sign-in.
9. Review sign-in logs.
10. Fix policy assignment, exclusions, named locations, or grant controls.
11. Move back to report-only.
12. Re-enforce only after successful validation.
```

# 07_Configure_Conditional_Access_Baseline_Policies_And_Break_Glass_Exclusions_Failure_Checks

| Symptom | Likely Cause | Check Command | Fix |
|---|---|---|---|
| Admin locked out | CA policy applied to admins without usable MFA or break glass exclusion | Sign in with break glass and review sign-in logs | Move policy to report-only or add proper exclusion |
| Break glass account blocked | Break glass not excluded or account included through group/role | Check group membership and CA sign-in result | Add break glass exclusion to every CA policy |
| Security defaults conflict with CA | Security defaults still enabled while CA is planned | `Get-MgPolicyIdentitySecurityDefaultEnforcementPolicy` | Disable security defaults only after CA baseline is ready |
| Pilot user not prompted for MFA | User not in policy scope or method not registered | What If tool and auth method report | Add user to pilot group and complete MFA registration |
| User prompted too often | Sign-in frequency or session controls too strict | Sign-in logs and policy session controls | Adjust session controls |
| Legacy authentication still works | Client app condition missing or policy not enforced | Sign-in logs ClientAppUsed | Add legacy client condition and move policy to On after testing |
| Service account broken | Service account included in block or MFA policy | Sign-in logs for service account | Exclude only documented service accounts or modernize authentication |
| Named location not matching | Wrong IP range or user exits through different public IP | Sign-in log IP address | Update named location ranges |
| Trusted location overtrusted | Broad IP range marked trusted | Named location config | Narrow IP ranges and review ownership |
| Guests blocked unexpectedly | Guest policy too broad or missing exception | Sign-in logs UserType and CA result | Adjust guest policy scope |
| Device compliance policy blocks users | Intune compliance not ready or device not enrolled | Sign-in logs and device details | Keep in report-only until device baseline is ready |
| Azure portal access blocked | Azure Management policy too broad or missing admin method | Sign-in logs AppDisplayName | Restore report-only or fix admin MFA |
| Risk policy unavailable | Entra ID P2 not licensed | SKU export | Use non-risk baseline or assign correct license |
| What If result differs from real sign-in | Real sign-in has different location, device, app, or client | Compare sign-in log details | Adjust conditions based on real log data |
| Policy collision causes unexpected block | Multiple policies apply, one blocks | Sign-in logs Conditional Access tab | Separate block policies and document precedence |
| Excluded user still affected | User included through another policy | Sign-in logs applied policies | Add exclusion to every relevant policy |
| Report-only looks clean but enforcement fails | Not enough representative pilot tests | Wider sign-in log sample | Extend pilot scope and test more apps/devices |
| Graph export fails | Missing Graph scopes or permissions | `Get-MgContext` | Reconnect with required scopes |
| Conditional Access menu missing | License or role issue | Entra admin center role and license checks | Assign proper role/license |
| Emergency account password unknown | Break glass process not operational | Manual sign-in test | Reset and store credential through approved offline process |

# 07_Configure_Conditional_Access_Baseline_Policies_And_Break_Glass_Exclusions_Related_Labs

| Lab | Relationship |
|---|---|
| 01_Prepare_On_Prem_AD_DS_DNS_UPN_Suffixes_And_Routable_Domains_For_Cloud.md | Ensures synced users have correct sign-in names before CA enforcement |
| 02_Validate_Custom_Domain_DNS_Records_For_Azure_And_M365.md | Confirms verified domain used by users and admin accounts |
| 03_Prepare_Identity_Synchronization_With_IdFix_And_AD_Object_Cleanup.md | Cleans identity attributes before access policies target users |
| 04_Configure_Entra_Connect_Sync_And_Cloud_Sync_Baseline.md | Provides synced users and groups for CA assignment |
| 05_Troubleshoot_Identity_Sync_Attribute_Duplicate_Object_And_Connect_Health_Issues.md | Fixes sync problems before access policy enforcement |
| 06_Configure_SSPR_MFA_Authentication_Methods_And_Password_Protection.md | Registers MFA and recovery methods before CA requires them |
| 08_Map_On_Prem_Groups_To_Cloud_Roles_Licensing_Access_And_App_Assignments.md | Uses groups and roles for scoped CA policy assignments |
| 09_Troubleshoot_Hybrid_SignIn_MFA_CA_Licensing_And_Admin_Access_Issues.md | Troubleshoots sign-in failures caused by CA, MFA, licensing, roles, or sync |
```