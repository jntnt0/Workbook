# 06_Configure_SSPR_MFA_Authentication_Methods_And_Password_Protection

# 06_Configure_SSPR_MFA_Authentication_Methods_And_Password_Protection_Index

06_Configure_SSPR_MFA_Authentication_Methods_And_Password_Protection.md  
06_Configure_SSPR_MFA_Authentication_Methods_And_Password_Protection  
06_Configure_SSPR_MFA_Authentication_Methods_And_Password_Protection_Source_Basis  
06_Configure_SSPR_MFA_Authentication_Methods_And_Password_Protection_Mental_Model  
06_Configure_SSPR_MFA_Authentication_Methods_And_Password_Protection_Planning_Table  
06_Configure_SSPR_MFA_Authentication_Methods_And_Password_Protection_Configuration_Checklist  
06_Configure_SSPR_MFA_Authentication_Methods_And_Password_Protection_Authentication_Methods_Decision_Table  
06_Configure_SSPR_MFA_Authentication_Methods_And_Password_Protection_Tenant_Identity_Precheck_Skeleton  
06_Configure_SSPR_MFA_Authentication_Methods_And_Password_Protection_SSPR_Baseline_Skeleton  
06_Configure_SSPR_MFA_Authentication_Methods_And_Password_Protection_MFA_Methods_And_Registration_Skeleton  
06_Configure_SSPR_MFA_Authentication_Methods_And_Password_Protection_Entra_Password_Protection_Skeleton  
06_Configure_SSPR_MFA_Authentication_Methods_And_Password_Protection_On_Prem_Password_Protection_Skeleton  
06_Configure_SSPR_MFA_Authentication_Methods_And_Password_Protection_User_Validation_Skeleton  
06_Configure_SSPR_MFA_Authentication_Methods_And_Password_Protection_Verification_Commands  
06_Configure_SSPR_MFA_Authentication_Methods_And_Password_Protection_Rollback  
06_Configure_SSPR_MFA_Authentication_Methods_And_Password_Protection_Failure_Checks  
06_Configure_SSPR_MFA_Authentication_Methods_And_Password_Protection_Related_Labs  

# 06_Configure_SSPR_MFA_Authentication_Methods_And_Password_Protection_Source_Basis

| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Entra Docs | Self-service password reset | SSPR enablement, selected groups, registration, reset methods, and administrator reset behavior |
| Microsoft Entra Docs | Authentication methods policy | Central management of Microsoft Authenticator, FIDO2 security keys, Temporary Access Pass, SMS, voice, email OTP, and passwordless options |
| Microsoft Entra Docs | Multifactor authentication | MFA registration, strong authentication methods, and user verification |
| Microsoft Entra Docs | Combined security information registration | Unified registration experience for SSPR and MFA methods |
| Microsoft Entra Docs | Microsoft Entra Password Protection | Global banned password list, custom banned password list, lockout settings, audit mode, and enforced mode |
| Microsoft Entra Docs | On-premises Microsoft Entra Password Protection | DC agents, proxy service, forest registration, proxy registration, and on-prem password filtering |
| Microsoft Graph PowerShell | User authentication methods and policy inspection | Reporting and validating registered authentication methods |
| Microsoft 365 Docs | Protect Microsoft 365 user accounts | MFA, secure sign-in, password protection, and identity security baseline |
| Operational practice | Pilot group first, enforce later, exclude break glass from risky policy paths, record rollback | Prevents accidental lockout and bad tenant-wide enforcement |

# 06_Configure_SSPR_MFA_Authentication_Methods_And_Password_Protection_Mental_Model

| Concept | Operational Meaning |
|---|---|
| SSPR | Self-service password reset, lets users reset their own passwords after proving identity |
| Authentication methods | Methods users register for MFA, SSPR, passwordless sign-in, or recovery |
| Combined registration | One registration flow for both MFA and SSPR security information |
| MFA | Additional verification factor required during sign-in or sensitive actions |
| Per-user MFA | Legacy direct MFA enablement on individual users, avoid for modern baseline unless needed |
| Conditional Access MFA | Preferred policy-driven way to require MFA based on user, app, risk, device, or location |
| Security defaults | Microsoft baseline security switch that can require MFA but cannot coexist with Conditional Access policies |
| Authentication methods policy | Central policy area controlling which methods are allowed and for whom |
| Microsoft Authenticator | App-based push, number matching, OTP, and passwordless-capable method |
| FIDO2 security key / passkey | Phishing-resistant authentication method |
| Temporary Access Pass | Time-limited credential used to bootstrap passwordless or recover users |
| SMS / voice | Legacy telephony-based methods, useful for fallback but weaker than phishing-resistant methods |
| Email OTP | Email-based verification method, not the same as strong MFA for privileged users |
| Password Protection | Entra feature that blocks weak, known, or custom banned passwords |
| Custom banned password list | Tenant-specific blocked terms, such as company name, city, product, mascot, or seasonal patterns |
| On-prem Password Protection | DC agent and proxy system that extends Entra password protection to AD DS password changes |
| Audit mode | Logs what would be blocked without enforcing it |
| Enforced mode | Blocks passwords that fail password protection checks |
| Break glass account | Emergency cloud-only account that must not be locked out by normal access policies |
| First rule | Pilot SSPR, MFA methods, and password protection before tenant-wide enforcement |
| Blunt rule | Do not use weak methods for admins unless you are accepting a real phishing and SIM-swap risk |

# 06_Configure_SSPR_MFA_Authentication_Methods_And_Password_Protection_Planning_Table

| Item | Example | Decision |
|---|---|---|
| Tenant domain | `contoso.onmicrosoft.com` | `<tenant-domain>` |
| Verified domain | `contoso.com` | `<verified-domain>` |
| Tenant ID | `00000000-0000-0000-0000-000000000000` | `<tenant-id>` |
| Admin account | `cloudadmin@contoso.com` | `<admin-account>` |
| Required admin role | Authentication Policy Administrator / Global Administrator | `<admin-role>` |
| SSPR pilot group | `GRP-Cloud-SSPR-Pilot` | `<sspr-pilot-group>` |
| MFA pilot group | `GRP-Cloud-MFA-Pilot` | `<mfa-pilot-group>` |
| Passwordless pilot group | `GRP-Cloud-Passwordless-Pilot` | `<passwordless-pilot-group>` |
| Break glass accounts | `breakglass1@contoso.onmicrosoft.com` | `<break-glass-accounts>` |
| SSPR scope | Selected group / All users | `<sspr-scope>` |
| Required SSPR methods | 1 or 2 | `<sspr-method-count>` |
| Allowed recovery methods | Authenticator, email, phone, security questions | `<allowed-recovery-methods>` |
| MFA enforcement model | Conditional Access / security defaults / per-user legacy | `<mfa-model>` |
| Strong admin methods | FIDO2, passkey, Authenticator, TAP | `<admin-methods>` |
| User methods | Authenticator, FIDO2, SMS fallback | `<user-methods>` |
| Disabled methods | Voice, SMS, email OTP, security questions | `<disabled-methods>` |
| Registration campaign | Enabled for pilot group | `<registration-campaign>` |
| Temporary Access Pass lifetime | 1 hour / 8 hours / 1 day | `<tap-lifetime>` |
| Password Protection mode | Audit / Enforced | `<password-protection-mode>` |
| Custom banned password terms | Company name, city, product, mascot | `<custom-banned-terms>` |
| On-prem AD DS forest | `corp.local` | `<ad-forest>` |
| Password Protection proxy server | `PWDPROXY01.corp.local` | `<password-proxy-server>` |
| Domain controllers | `DC1`, `DC2` | `<domain-controllers>` |
| Evidence folder | `C:\IdentitySecurity-Baseline` | `<evidence-path>` |
| Rollback plan | Disable policies, revert groups, return to audit mode | `<rollback-plan>` |

# 06_Configure_SSPR_MFA_Authentication_Methods_And_Password_Protection_Configuration_Checklist

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm hybrid identity sync is healthy | Admin Workstation | `Review task 04 and task 05 evidence` | Users and groups are syncing correctly |
| 2 | Create evidence folder | Admin Workstation | `New-Item -ItemType Directory -Force -Path C:\IdentitySecurity-Baseline` | Evidence folder exists |
| 3 | Connect to Microsoft Graph | Admin Workstation | `Connect-MgGraph -Scopes "Directory.Read.All","User.Read.All","Group.Read.All","Policy.Read.All","Policy.ReadWrite.AuthenticationMethod","UserAuthenticationMethod.Read.All"` | Graph session connects |
| 4 | Export tenant domains | Admin Workstation | `Get-MgDomain \| Export-Csv C:\IdentitySecurity-Baseline\tenant-domains.csv -NoTypeInformation` | Verified domains are documented |
| 5 | Export target users | Admin Workstation | `Get-MgUser -All \| Export-Csv C:\IdentitySecurity-Baseline\users-before.csv -NoTypeInformation` | User baseline exists |
| 6 | Create or confirm pilot SSPR group | Entra Admin Center / Graph | `New-MgGroup` or portal group creation | SSPR pilot group exists |
| 7 | Create or confirm MFA pilot group | Entra Admin Center / Graph | `New-MgGroup` or portal group creation | MFA pilot group exists |
| 8 | Add pilot users to groups | Entra Admin Center / Graph | `New-MgGroupMember` | Pilot group membership is ready |
| 9 | Confirm break glass accounts are cloud-only | Admin Workstation | `Get-MgUser -UserId "<breakglass-upn>" -Property OnPremisesSyncEnabled` | Break glass accounts are not synced from AD |
| 10 | Configure SSPR for selected pilot group | Entra Admin Center | `Protection > Password reset > Properties` | SSPR is enabled for pilot group |
| 11 | Configure SSPR authentication methods | Entra Admin Center | `Protection > Password reset > Authentication methods` | Required number of methods and allowed methods are configured |
| 12 | Configure SSPR registration settings | Entra Admin Center | `Protection > Password reset > Registration` | Registration prompt and reconfirm interval are set |
| 13 | Configure SSPR notifications | Entra Admin Center | `Protection > Password reset > Notifications` | User and admin notifications are set |
| 14 | Configure on-prem integration if hybrid writeback is required | Entra Admin Center | `Protection > Password reset > On-premises integration` | Password writeback behavior is planned or enabled |
| 15 | Review authentication methods policy | Entra Admin Center | `Protection > Authentication methods > Policies` | Current allowed methods are known |
| 16 | Enable Microsoft Authenticator for pilot | Entra Admin Center | `Authentication methods policy` | Authenticator is allowed for pilot |
| 17 | Enable FIDO2 / passkey methods for pilot if available | Entra Admin Center | `Authentication methods policy` | Phishing-resistant method is allowed for pilot |
| 18 | Configure Temporary Access Pass for help desk bootstrap | Entra Admin Center | `Authentication methods policy > Temporary Access Pass` | TAP is available to authorized admins |
| 19 | Disable or limit weak methods where appropriate | Entra Admin Center | `SMS, voice, security questions` | Weak methods are limited or scoped |
| 20 | Configure registration campaign | Entra Admin Center | `Authentication methods > Registration campaign` | Pilot users are prompted to register |
| 21 | Confirm MFA enforcement model | Entra Admin Center | `Security defaults or Conditional Access plan` | MFA enforcement path is documented |
| 22 | Configure Conditional Access MFA in task 07 if used | Entra Admin Center | `Require MFA for pilot group` | MFA requirement is policy-driven |
| 23 | Configure cloud Password Protection custom banned passwords | Entra Admin Center | `Protection > Password protection` | Tenant custom banned password list exists |
| 24 | Set Password Protection to audit first if deploying on-prem | Entra Admin Center | `Mode: Audit` | Password policy impact is observed before enforcement |
| 25 | Install on-prem Password Protection proxy if required | Password Proxy Server | `AzureADPasswordProtectionProxySetup.exe` | Proxy service installs |
| 26 | Register on-prem Password Protection forest | Password Proxy Server | `Register-AzureADPasswordProtectionForest` | Forest registration succeeds |
| 27 | Register on-prem Password Protection proxy | Password Proxy Server | `Register-AzureADPasswordProtectionProxy` | Proxy registration succeeds |
| 28 | Install Password Protection DC agent on each DC | Domain Controllers | `AzureADPasswordProtectionDCAgentSetup.msi` | DC agent installs |
| 29 | Reboot DCs if required by agent install | Domain Controllers | `Restart-Computer` | DC agent loads |
| 30 | Validate Password Protection services | Proxy / DCs | `Get-Service AzureADPasswordProtection*` | Services are running |
| 31 | Test pilot SSPR registration | Pilot User Workstation | `https://aka.ms/ssprsetup` | User can register security info |
| 32 | Test SSPR reset flow | Pilot User Workstation | `https://passwordreset.microsoftonline.com` | User can reset password |
| 33 | Test MFA registration and challenge | Pilot User Workstation | `Sign in to cloud app` | User is prompted and can satisfy MFA |
| 34 | Test admin method registration | Admin Workstation | `Register strong method for privileged account` | Admin has strong authentication method |
| 35 | Export registered methods report | Admin Workstation | `Run User_Validation_Skeleton` | Registered methods are documented |
| 36 | Move from pilot to broader group only after validation | Operator | `Expand group membership or switch SSPR to all users` | Rollout expands in controlled phases |
| 37 | Document final state and exceptions | Operator | `Record scopes, methods, exclusions, password policy, test results` | Baseline is complete |

# 06_Configure_SSPR_MFA_Authentication_Methods_And_Password_Protection_Authentication_Methods_Decision_Table

| Method | Strength | Good Use | Caution | Baseline Decision |
|---|---|---|---|---|
| Microsoft Authenticator push with number matching | Strong | Default user MFA method | User fatigue still possible if users approve blindly | Enable for users and admins |
| Microsoft Authenticator OTP | Strong | Backup when push unavailable | User must understand code entry | Enable |
| FIDO2 security key / passkey | Very strong | Admins, privileged users, passwordless pilot | Requires device procurement and user training | Enable for privileged and pilot users |
| Temporary Access Pass | Bootstrap / recovery | Onboarding passwordless users, recovering lost methods | Must be time-limited and help desk controlled | Enable for admins/help desk only |
| SMS | Weak to moderate | Fallback for low-risk users | SIM swap and phishing risk | Limit or phase out |
| Voice call | Weak | Legacy fallback only | Easy to abuse, poor admin method | Disable or tightly scope |
| Email OTP | Limited use | Guest or recovery scenarios depending on tenant design | Not a strong privileged method | Do not use for admins |
| Security questions | Weak | Legacy SSPR only | Guessable and poor security | Avoid |
| Certificate-based authentication | Very strong | Admins, regulated environments, smart card environments | Requires PKI maturity | Optional advanced method |
| Windows Hello for Business | Strong | Managed Windows device sign-in | Requires device and policy planning | Related endpoint identity baseline |
| Password only | Weak | Initial account setup only | Not acceptable for privileged cloud access | Do not rely on alone |

# 06_Configure_SSPR_MFA_Authentication_Methods_And_Password_Protection_Tenant_Identity_Precheck_Skeleton

```powershell
# Run from admin workstation with Microsoft Graph PowerShell.
# Purpose: capture tenant, domain, user, group, and authentication policy baseline before changes.

$EvidencePath = "C:\IdentitySecurity-Baseline"
New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\01-tenant-identity-precheck-transcript.txt"

Import-Module Microsoft.Graph.Authentication
Import-Module Microsoft.Graph.Identity.DirectoryManagement
Import-Module Microsoft.Graph.Users
Import-Module Microsoft.Graph.Groups

Connect-MgGraph -Scopes `
  "Directory.Read.All",
  "Domain.Read.All",
  "User.Read.All",
  "Group.Read.All",
  "Policy.Read.All",
  "UserAuthenticationMethod.Read.All"

# Tenant context.
Get-MgContext |
  Format-List * |
  Out-File "$EvidencePath\graph-context.txt"

# Domains.
Get-MgDomain |
  Select-Object Id,IsVerified,IsDefault,AuthenticationType |
  Export-Csv "$EvidencePath\tenant-domains.csv" -NoTypeInformation

# Users baseline.
Get-MgUser -All `
  -Property Id,DisplayName,UserPrincipalName,Mail,AccountEnabled,UserType,OnPremisesSyncEnabled,OnPremisesImmutableId |
  Select-Object Id,DisplayName,UserPrincipalName,Mail,AccountEnabled,UserType,OnPremisesSyncEnabled,OnPremisesImmutableId |
  Export-Csv "$EvidencePath\users-before-sspr-mfa.csv" -NoTypeInformation

# Groups baseline.
Get-MgGroup -All `
  -Property Id,DisplayName,Mail,MailEnabled,SecurityEnabled,GroupTypes |
  Select-Object Id,DisplayName,Mail,MailEnabled,SecurityEnabled,GroupTypes |
  Export-Csv "$EvidencePath\groups-before-sspr-mfa.csv" -NoTypeInformation

# Authorization policy, useful for security defaults visibility.
Get-MgPolicyAuthorizationPolicy |
  Format-List * |
  Out-File "$EvidencePath\authorization-policy-before.txt"

# Authentication methods policy.
Get-MgPolicyAuthenticationMethodPolicy |
  Format-List * |
  Out-File "$EvidencePath\authentication-methods-policy-before.txt"

Disconnect-MgGraph

Stop-Transcript
```

# 06_Configure_SSPR_MFA_Authentication_Methods_And_Password_Protection_SSPR_Baseline_Skeleton

```text
# SSPR baseline skeleton, portal-driven.

1. Sign in to Microsoft Entra admin center.
2. Go to:
   Protection > Password reset

3. Open Properties.
4. Set Self service password reset enabled to:
   - Selected
   - Select group: GRP-Cloud-SSPR-Pilot

5. Open Authentication methods.
6. Set number of methods required to reset:
   - 2 for normal baseline
   - 1 only for limited lab or low-risk pilot

7. Select allowed methods based on the pilot decision:
   - Microsoft Authenticator notification
   - Microsoft Authenticator code
   - Email
   - Mobile phone
   - Office phone only if justified
   - Security questions only if legacy requirement exists

8. Open Registration.
9. Set users required to register when signing in:
   - Yes
10. Set reconfirm security info interval:
   - 180 days or approved organization value

11. Open Notifications.
12. Enable user notification on password reset:
   - Yes
13. Enable admin notification on admin password reset:
   - Yes

14. Open Customization.
15. Add help desk contact or support URL if available.

16. Open On-premises integration if hybrid password writeback is required.
17. Confirm password writeback prerequisites are met through Entra Connect Sync.
18. Enable writeback only if approved and tested.

19. Save settings.

20. Pilot test:
   - Go to https://aka.ms/ssprsetup
   - Register methods
   - Go to https://passwordreset.microsoftonline.com
   - Complete reset test

21. Export evidence:
   - screenshots
   - pilot user test result
   - registration confirmation
   - reset confirmation
   - help desk notes
```

```powershell
# Optional group preparation using Microsoft Graph.
# Purpose: create pilot groups and add test users.

$EvidencePath = "C:\IdentitySecurity-Baseline"
$SsprPilotGroupName = "GRP-Cloud-SSPR-Pilot"
$MfaPilotGroupName = "GRP-Cloud-MFA-Pilot"
$PilotUserUpn = "pilot.user@contoso.com"

New-Item -ItemType Directory -Force -Path $EvidencePath

Import-Module Microsoft.Graph.Groups
Import-Module Microsoft.Graph.Users

Connect-MgGraph -Scopes "Group.ReadWrite.All","User.Read.All","Directory.ReadWrite.All"

$SsprGroup = Get-MgGroup -Filter "displayName eq '$SsprPilotGroupName'" -ErrorAction SilentlyContinue

if (-not $SsprGroup) {
  $SsprGroup = New-MgGroup `
    -DisplayName $SsprPilotGroupName `
    -MailEnabled:$false `
    -MailNickname "grp-cloud-sspr-pilot" `
    -SecurityEnabled:$true
}

$MfaGroup = Get-MgGroup -Filter "displayName eq '$MfaPilotGroupName'" -ErrorAction SilentlyContinue

if (-not $MfaGroup) {
  $MfaGroup = New-MgGroup `
    -DisplayName $MfaPilotGroupName `
    -MailEnabled:$false `
    -MailNickname "grp-cloud-mfa-pilot" `
    -SecurityEnabled:$true
}

$PilotUser = Get-MgUser -UserId $PilotUserUpn

# Add pilot user to groups.
New-MgGroupMember -GroupId $SsprGroup.Id -DirectoryObjectId $PilotUser.Id -ErrorAction SilentlyContinue
New-MgGroupMember -GroupId $MfaGroup.Id -DirectoryObjectId $PilotUser.Id -ErrorAction SilentlyContinue

# Export group membership evidence.
Get-MgGroupMember -GroupId $SsprGroup.Id |
  Select-Object Id |
  Export-Csv "$EvidencePath\sspr-pilot-group-members.csv" -NoTypeInformation

Get-MgGroupMember -GroupId $MfaGroup.Id |
  Select-Object Id |
  Export-Csv "$EvidencePath\mfa-pilot-group-members.csv" -NoTypeInformation

Disconnect-MgGraph
```

# 06_Configure_SSPR_MFA_Authentication_Methods_And_Password_Protection_MFA_Methods_And_Registration_Skeleton

```text
# Authentication methods and MFA registration skeleton, portal-driven.

1. Sign in to Microsoft Entra admin center.
2. Go to:
   Protection > Authentication methods > Policies

3. Review all methods before changing anything:
   - Microsoft Authenticator
   - FIDO2 security key
   - Temporary Access Pass
   - SMS
   - Voice call
   - Email OTP
   - Certificate-based authentication if used

4. Configure Microsoft Authenticator:
   - Enable for selected pilot group first
   - Require number matching if available
   - Enable additional context if approved
   - Prefer Authenticator for admins and users

5. Configure FIDO2 security key / passkey:
   - Enable for admin or passwordless pilot group
   - Restrict key types only if the organization has a hardware standard
   - Document allowed AAGUIDs if restricted

6. Configure Temporary Access Pass:
   - Enable for selected admin/help desk scope
   - Require limited lifetime
   - Require one-time use if appropriate
   - Document who can issue TAP

7. Configure SMS:
   - Disable for admins
   - Limit to fallback group if business requires it
   - Plan phaseout

8. Configure voice call:
   - Disable unless a documented legacy requirement exists

9. Configure email OTP:
   - Do not use as privileged admin MFA
   - Enable only for the intended scenario if needed

10. Go to:
    Protection > Authentication methods > Registration campaign

11. Enable registration campaign for pilot users.
12. Exclude break glass accounts.
13. Save settings.

14. Validate pilot:
    - User signs in
    - User is prompted for security info registration
    - User registers Authenticator
    - User can complete MFA challenge
    - User can view methods at https://aka.ms/mysecurityinfo

15. Export evidence:
    - authentication methods settings
    - registration campaign settings
    - pilot user method registration
    - pilot MFA sign-in result
```

```powershell
# User authentication method reporting.
# Purpose: export registered authentication methods for pilot users.

$EvidencePath = "C:\IdentitySecurity-Baseline"
$PilotUserUpn = "pilot.user@contoso.com"

New-Item -ItemType Directory -Force -Path $EvidencePath

Import-Module Microsoft.Graph.Users

Connect-MgGraph -Scopes "User.Read.All","UserAuthenticationMethod.Read.All","Directory.Read.All"

$User = Get-MgUser -UserId $PilotUserUpn

# List authentication methods for one user.
Get-MgUserAuthenticationMethod -UserId $User.Id |
  Select-Object Id,AdditionalProperties |
  Export-Csv "$EvidencePath\$($PilotUserUpn)-authentication-methods.csv" -NoTypeInformation

# Export auth methods for all pilot users from a CSV list if available.
# CSV column required: UserPrincipalName
$PilotCsv = "$EvidencePath\pilot-users.csv"

if (Test-Path $PilotCsv) {
  $PilotUsers = Import-Csv $PilotCsv

  $Results = foreach ($Pilot in $PilotUsers) {
    $MgUser = Get-MgUser -UserId $Pilot.UserPrincipalName -ErrorAction SilentlyContinue

    if ($MgUser) {
      $Methods = Get-MgUserAuthenticationMethod -UserId $MgUser.Id -ErrorAction SilentlyContinue

      [PSCustomObject]@{
        UserPrincipalName = $Pilot.UserPrincipalName
        MethodCount = ($Methods | Measure-Object).Count
        MethodTypes = (($Methods | ForEach-Object { $_.AdditionalProperties["@odata.type"] }) -join ";")
      }
    }
  }

  $Results |
    Export-Csv "$EvidencePath\pilot-users-authentication-method-summary.csv" -NoTypeInformation
}

Disconnect-MgGraph
```

# 06_Configure_SSPR_MFA_Authentication_Methods_And_Password_Protection_Entra_Password_Protection_Skeleton

```text
# Cloud Microsoft Entra Password Protection skeleton.

1. Sign in to Microsoft Entra admin center.
2. Go to:
   Protection > Password protection

3. Review current settings.
4. Configure smart lockout:
   - Lockout threshold: organization approved value
   - Lockout duration: organization approved value

5. Configure custom banned password list:
   - Add company name
   - Add company abbreviations
   - Add brand names
   - Add city or region terms
   - Add product names
   - Add mascot or seasonal terms
   - Do not add obvious passwords only. Microsoft already blocks many common weak passwords.

6. Configure mode:
   - Audit first for new rollout
   - Enforced after pilot review

7. Save settings.

8. Validate with test accounts:
   - Attempt weak password reset
   - Attempt custom banned term
   - Attempt strong password
   - Confirm expected behavior

9. Document:
   - lockout threshold
   - lockout duration
   - custom banned terms
   - mode
   - test result
   - enforcement date
```

```powershell
# Evidence capture for cloud Password Protection related tenant settings.
# Many password protection settings are portal-managed. Use this to capture related tenant policy evidence.

$EvidencePath = "C:\IdentitySecurity-Baseline"
New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\02-password-protection-evidence-transcript.txt"

Import-Module Microsoft.Graph.Identity.DirectoryManagement

Connect-MgGraph -Scopes "Policy.Read.All","Directory.Read.All"

# Capture authorization policy and organization details.
Get-MgPolicyAuthorizationPolicy |
  Format-List * |
  Out-File "$EvidencePath\authorization-policy-password-baseline.txt"

Get-MgOrganization |
  Format-List * |
  Out-File "$EvidencePath\organization-baseline.txt"

Disconnect-MgGraph

Stop-Transcript
```

# 06_Configure_SSPR_MFA_Authentication_Methods_And_Password_Protection_On_Prem_Password_Protection_Skeleton

```text
# On-premises Microsoft Entra Password Protection skeleton.
# Use this only for hybrid AD DS password protection.

1. Confirm cloud Password Protection policy is configured.
2. Start in audit mode.
3. Choose a proxy server:
   - Domain joined
   - Stable network connectivity
   - Can reach Microsoft Entra endpoints
   - Not necessarily a domain controller

4. Download Microsoft Entra Password Protection components from the official Microsoft source:
   - Proxy service installer
   - DC agent installer

5. On the proxy server:
   - Install proxy service
   - Register the forest
   - Register the proxy

6. On each domain controller:
   - Install DC agent
   - Reboot if required
   - Confirm DC agent service is running

7. Confirm SYSVOL and AD replication health.
8. Wait for policy download and propagation.
9. Test password change in audit mode.
10. Review DC agent event logs.
11. Confirm banned password attempts are logged.
12. After pilot and event review, switch from audit to enforced mode.
13. Test blocked password behavior.
14. Document proxy server, DC agent state, event IDs, and enforcement date.
```

```powershell
# Run on Password Protection proxy server after installing proxy components.
# Purpose: register forest and proxy, then capture evidence.

$EvidencePath = "C:\IdentitySecurity-Baseline\PasswordProtection"
New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\01-onprem-password-protection-proxy-registration.txt"

# Import module if installed.
Import-Module AzureADPasswordProtection -ErrorAction SilentlyContinue

# Connect/register with appropriate cloud admin credentials when prompted.
# Register the AD forest.
Register-AzureADPasswordProtectionForest

# Register this proxy.
Register-AzureADPasswordProtectionProxy

# Service evidence.
Get-Service |
  Where-Object {
    $_.Name -like "*AzureADPasswordProtection*" -or
    $_.DisplayName -like "*Password Protection*"
  } |
  Select-Object Name,DisplayName,Status,StartType |
  Export-Csv "$EvidencePath\password-protection-proxy-services.csv" -NoTypeInformation

# Event evidence.
Get-WinEvent -LogName Application -MaxEvents 300 |
  Where-Object {
    $_.ProviderName -like "*PasswordProtection*" -or
    $_.Message -like "*Password Protection*"
  } |
  Select-Object TimeCreated,ProviderName,Id,LevelDisplayName,Message |
  Export-Csv "$EvidencePath\password-protection-proxy-events.csv" -NoTypeInformation

Stop-Transcript
```

```powershell
# Run on each domain controller after installing the DC agent.
# Purpose: validate DC agent services and local event evidence.

$EvidencePath = "C:\IdentitySecurity-Baseline\PasswordProtection"
New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\$env:COMPUTERNAME-dc-agent-validation.txt"

hostname | Out-File "$EvidencePath\$env:COMPUTERNAME-hostname.txt"

Get-Service |
  Where-Object {
    $_.Name -like "*AzureADPasswordProtection*" -or
    $_.DisplayName -like "*Password Protection*"
  } |
  Select-Object Name,DisplayName,Status,StartType |
  Export-Csv "$EvidencePath\$env:COMPUTERNAME-password-protection-services.csv" -NoTypeInformation

# AD health checks.
repadmin /replsummary | Out-File "$EvidencePath\$env:COMPUTERNAME-repadmin-replsummary.txt"
dcdiag /test:dns | Out-File "$EvidencePath\$env:COMPUTERNAME-dcdiag-dns.txt"

# Event evidence, adjust log name if your installed component writes to a dedicated operational log.
Get-WinEvent -LogName Application -MaxEvents 500 |
  Where-Object {
    $_.ProviderName -like "*PasswordProtection*" -or
    $_.Message -like "*Password Protection*" -or
    $_.Message -like "*banned password*"
  } |
  Select-Object TimeCreated,ProviderName,Id,LevelDisplayName,Message |
  Export-Csv "$EvidencePath\$env:COMPUTERNAME-password-protection-events.csv" -NoTypeInformation

Stop-Transcript
```

# 06_Configure_SSPR_MFA_Authentication_Methods_And_Password_Protection_User_Validation_Skeleton

```powershell
# Run from admin workstation.
# Purpose: validate users, registered auth methods, and basic readiness.

$EvidencePath = "C:\IdentitySecurity-Baseline"
$PilotGroupName = "GRP-Cloud-SSPR-Pilot"

New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\03-user-validation-transcript.txt"

Import-Module Microsoft.Graph.Users
Import-Module Microsoft.Graph.Groups

Connect-MgGraph -Scopes "User.Read.All","Group.Read.All","UserAuthenticationMethod.Read.All","Directory.Read.All"

$PilotGroup = Get-MgGroup -Filter "displayName eq '$PilotGroupName'"

$Members = Get-MgGroupMember -GroupId $PilotGroup.Id -All

$Results = foreach ($Member in $Members) {
  $User = Get-MgUser `
    -UserId $Member.Id `
    -Property Id,DisplayName,UserPrincipalName,AccountEnabled,UserType,OnPremisesSyncEnabled `
    -ErrorAction SilentlyContinue

  if ($User) {
    $Methods = Get-MgUserAuthenticationMethod -UserId $User.Id -ErrorAction SilentlyContinue

    [PSCustomObject]@{
      DisplayName = $User.DisplayName
      UserPrincipalName = $User.UserPrincipalName
      AccountEnabled = $User.AccountEnabled
      UserType = $User.UserType
      OnPremisesSyncEnabled = $User.OnPremisesSyncEnabled
      AuthenticationMethodCount = ($Methods | Measure-Object).Count
      AuthenticationMethodTypes = (($Methods | ForEach-Object { $_.AdditionalProperties["@odata.type"] }) -join ";")
    }
  }
}

$Results |
  Export-Csv "$EvidencePath\sspr-pilot-users-auth-method-validation.csv" -NoTypeInformation

Disconnect-MgGraph

Stop-Transcript
```

```text
# Manual pilot validation steps.

1. Pick one synced pilot user.
2. Go to:
   https://aka.ms/mysecurityinfo
3. Register approved methods:
   - Microsoft Authenticator
   - phone only if allowed
   - FIDO2/passkey if in pilot
4. Go to:
   https://passwordreset.microsoftonline.com
5. Start password reset.
6. Complete verification using approved methods.
7. Set a password that complies with password protection.
8. Attempt a blocked weak password in test scope if safe.
9. Confirm expected block or audit event.
10. Sign in to Microsoft 365 app.
11. Complete MFA challenge.
12. Record:
    - user
    - methods registered
    - reset result
    - MFA result
    - password protection result
    - time
    - errors
```

# 06_Configure_SSPR_MFA_Authentication_Methods_And_Password_Protection_Verification_Commands

| Command | Purpose | Healthy Output |
|---|---|---|
| `Get-MgDomain` | Confirms tenant domains | Verified custom domain appears |
| `Get-MgPolicyAuthorizationPolicy` | Reviews tenant authorization policy and security defaults related state | Policy data returns |
| `Get-MgPolicyAuthenticationMethodPolicy` | Reviews authentication methods policy | Policy data returns |
| `Get-MgGroup -Filter "displayName eq 'GRP-Cloud-SSPR-Pilot'"` | Confirms pilot group exists | Group object returns |
| `Get-MgGroupMember -GroupId <group-id>` | Confirms pilot membership | Pilot users appear |
| `Get-MgUserAuthenticationMethod -UserId <user-id>` | Lists registered auth methods for a user | Expected methods appear |
| `Get-MgUser -UserId <upn> -Property OnPremisesSyncEnabled` | Confirms synced or cloud-only state | Synced users show expected value |
| `https://aka.ms/ssprsetup` | User security info registration | User can register methods |
| `https://aka.ms/mysecurityinfo` | User security info management | User can view and manage methods |
| `https://passwordreset.microsoftonline.com` | SSPR test | Pilot user can reset password |
| `Get-Service AzureADPasswordProtection*` | Checks on-prem Password Protection services | Proxy or DC agent services are running |
| `repadmin /replsummary` | Confirms AD replication before on-prem enforcement | No major replication failures |
| `dcdiag /test:dns` | Confirms DC DNS health | DNS tests pass or exceptions are documented |
| `Get-WinEvent -LogName Application -MaxEvents 300` | Reviews password protection events | Audit or enforcement events are visible |
| `Get-MgUser -All -Property UserPrincipalName` | Exports users for registration reporting | Users return |

# 06_Configure_SSPR_MFA_Authentication_Methods_And_Password_Protection_Rollback

```text
# SSPR rollback.

1. Sign in to Microsoft Entra admin center.
2. Go to:
   Protection > Password reset > Properties
3. Change SSPR scope from All to Selected if broad rollout caused issues.
4. Remove affected group from SSPR scope if needed.
5. Do not disable SSPR globally unless there is a real incident.
6. Keep admin notification settings enabled if possible.
7. Record rollback time and reason.
```

```text
# Authentication methods rollback.

1. Go to:
   Protection > Authentication methods > Policies
2. Revert method scope from All users to selected pilot groups.
3. Re-enable a temporary fallback method only if users are locked out.
4. Do not add weak methods to privileged admin scope unless emergency approved.
5. Disable registration campaign if prompts are causing business disruption.
6. Keep break glass accounts validated separately.
7. Record:
   - method changed
   - original scope
   - rollback scope
   - affected users
```

```text
# Password Protection rollback.

Cloud-only:
1. Go to:
   Protection > Password protection
2. Switch from Enforced to Audit if password changes are unexpectedly blocked.
3. Remove custom banned terms that are too broad.
4. Save settings.
5. Retest password change.

On-premises:
1. Switch policy to Audit mode in the portal.
2. Confirm policy reaches DC agents.
3. If needed, stop or uninstall DC agent only during approved rollback.
4. Preserve event logs before uninstall.
5. Do not remove proxy or DC agents until source issue is known.
```

```powershell
# Evidence and service rollback helpers for on-prem Password Protection.

$EvidencePath = "C:\IdentitySecurity-Baseline\Rollback"
New-Item -ItemType Directory -Force -Path $EvidencePath

Start-Transcript -Path "$EvidencePath\password-protection-rollback-evidence.txt"

# Capture current services before any rollback action.
Get-Service |
  Where-Object {
    $_.Name -like "*AzureADPasswordProtection*" -or
    $_.DisplayName -like "*Password Protection*"
  } |
  Select-Object Name,DisplayName,Status,StartType |
  Export-Csv "$EvidencePath\password-protection-services-before-rollback.csv" -NoTypeInformation

# Stop service only if approved.
# Stop-Service -Name "<service-name>"

Stop-Transcript
```

# 06_Configure_SSPR_MFA_Authentication_Methods_And_Password_Protection_Failure_Checks

| Symptom | Likely Cause | Check Command | Fix |
|---|---|---|---|
| User not prompted for SSPR registration | User not in SSPR scope or registration disabled | Check SSPR Properties and Registration settings | Add user to pilot group or enable registration |
| User cannot reset password | Not enough registered methods or SSPR disabled | `Get-MgUserAuthenticationMethod -UserId <id>` | Register required methods |
| User sees old phone number | Stale authentication method data | `https://aka.ms/mysecurityinfo` | User or admin updates methods |
| Synced user password reset does not write back to AD | Password writeback not enabled or Entra Connect issue | Check Entra Connect optional features and sync health | Enable writeback and validate sync |
| Admin locked out by method change | Weak rollback planning or no break glass validation | Check break glass account access | Use break glass, restore method scope, add TAP if approved |
| Break glass account affected by policy | Break glass was included in group or policy scope | Check group membership and policy exclusions | Remove from normal policy scope and validate sign-in |
| MFA prompt never appears | MFA not enforced by CA, security defaults, or per-user MFA | Check enforcement model | Configure CA in task 07 or enable intended model |
| MFA prompt appears too often | CA session controls or sign-in frequency too strict | Review CA and session policy | Adjust task 07 session controls |
| SMS available for admins | Authentication method policy scope too broad | Review SMS method policy | Exclude admins or disable SMS for privileged scope |
| Temporary Access Pass abused or too long-lived | TAP policy lifetime too permissive | Review TAP policy | Shorten lifetime and restrict who can issue TAP |
| Users cannot register Authenticator | Method disabled or blocked by scope | Authentication methods policy | Enable Authenticator for pilot or user group |
| FIDO2 security key registration fails | Method disabled, unsupported key, or restriction mismatch | Authentication methods policy | Enable method or allow approved key type |
| Password with company name is allowed | Custom banned password list missing or not enforced | Password Protection settings | Add custom banned term and enforce after audit |
| Too many passwords blocked | Custom banned terms are too broad | Password Protection settings and event logs | Remove broad terms or return to audit mode |
| On-prem password policy not applying | DC agent missing, proxy missing, or forest not registered | `Get-Service AzureADPasswordProtection*` | Register forest/proxy and install DC agents |
| On-prem password protection events missing | Agent not loaded, DC not rebooted, or wrong log checked | Services and Event Viewer | Reboot DC if required and confirm agent |
| Password Protection proxy unhealthy | Service stopped or outbound connectivity blocked | Proxy service status and event logs | Restart service and fix outbound access |
| Users report method registration loop | Browser/session issue or stale registration state | Sign-in logs and user methods | Clear stale methods or re-register |
| Help desk cannot view methods | Missing role | Entra role assignment | Assign Authentication Administrator or appropriate least-privileged role |
| Graph method export fails | Missing Graph scopes or module | `Get-MgContext` | Reconnect with required scopes |

# 06_Configure_SSPR_MFA_Authentication_Methods_And_Password_Protection_Related_Labs

| Lab                                                                                   | Relationship                                                                  |
| ------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------- |
| 01_Prepare_On_Prem_AD_DS_DNS_UPN_Suffixes_And_Routable_Domains_For_Cloud.md           | Ensures users have correct routable sign-in names before SSPR and MFA rollout |
| 02_Validate_Custom_Domain_DNS_Records_For_Azure_And_M365.md                           | Confirms the custom domain used by user UPNs is verified                      |
| 03_Prepare_Identity_Synchronization_With_IdFix_And_AD_Object_Cleanup.md               | Cleans user attributes before security registration and password reset        |
| 04_Configure_Entra_Connect_Sync_And_Cloud_Sync_Baseline.md                            | Provides synced identities and optional password writeback foundation         |
| 05_Troubleshoot_Identity_Sync_Attribute_Duplicate_Object_And_Connect_Health_Issues.md | Fixes sync problems that block user readiness                                 |
| 07_Configure_Conditional_Access_Baseline_Policies_And_Break_Glass_Exclusions.md       | Enforces MFA and sign-in controls after methods are registered                |
| 08_Map_On_Prem_Groups_To_Cloud_Roles_Licensing_Access_And_App_Assignments.md          | Uses groups to scope SSPR, MFA, passwordless, and method policies             |
| 09_Troubleshoot_Hybrid_SignIn_MFA_CA_Licensing_And_Admin_Access_Issues.md             | Troubleshoots sign-in, MFA, password reset, and access problems after rollout |