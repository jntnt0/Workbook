# 02_Configure_App_Registrations_API_Permissions_Secrets_Certificates_And_Owners

## Objective

Configure Microsoft Entra App Registrations, API permissions, redirect URIs, authentication platform settings, client secrets, certificates, owners, and validation checks.

This workbook focuses on the application object side of Microsoft Entra ID.

The prior workbook focused on Enterprise Applications and service principals. This workbook focuses on App Registrations.

## Lab_Context

| Item | Value |
|---|---|
| Tenant | `<tenant-name>.onmicrosoft.com` |
| Admin portal | `https://entra.microsoft.com` |
| Primary admin center | Microsoft Entra admin center |
| App registration name | `<app-registration-name>` |
| Application/client ID | `<application-client-id>` |
| Directory/tenant ID | `<tenant-id>` |
| Redirect URI | `<redirect-uri>` |
| Logout URL | `<logout-url>` |
| Owner 1 | `<owner1-upn>` |
| Owner 2 | `<owner2-upn>` |
| Test API permission | Microsoft Graph `User.Read` |
| PowerShell module | Microsoft Graph PowerShell |
| Graph permission scope | `Application.ReadWrite.All`, `Directory.ReadWrite.All`, `AppRoleAssignment.ReadWrite.All`, `AuditLog.Read.All` |

## Mental_Model

An App Registration is the application definition in Microsoft Entra ID.

An Enterprise Application is the tenant-local service principal instance used for assignments, sign-ins, SSO, provisioning, and Conditional Access targeting.

Basic relationship:

~~~text
App Registration
    |
    | Defines
    | - Client ID
    | - Redirect URIs
    | - API permissions
    | - Secrets
    | - Certificates
    | - Owners
    | - App roles
    | - Exposed APIs
    v
Service Principal / Enterprise Application
    |
    | Used for
    | - Tenant-local sign-in
    | - Assignments
    | - Conditional Access targeting
    | - Provisioning
    | - SSO
~~~

Use App Registrations when configuring:

- OAuth / OIDC application identity
- Redirect URIs
- Client secrets
- Certificate credentials
- API permissions
- Token settings
- Owners
- Supported account types
- Public client flows
- Exposed APIs
- App roles

Use Enterprise Applications when configuring:

- Users and groups
- Assignment required
- SAML SSO
- Provisioning
- Sign-in logs
- App-specific Conditional Access targeting

## Planning_Table

| Decision Area | Recommended Baseline | Notes |
|---|---|---|
| Supported account type | Single tenant unless there is a real multi-tenant need | Do not make apps multi-tenant casually |
| Redirect URIs | Exact URIs only | Avoid broad or unused redirect URIs |
| Implicit grant | Disabled unless legacy app requires it | Auth code with PKCE is the modern default |
| Public client flows | Disabled unless app is native/mobile/CLI and requires it | Do not enable for confidential web apps |
| API permissions | Least privilege | Avoid broad Graph permissions unless required |
| Permission type | Delegated for user-context apps, Application for daemon/service apps | Application permissions are higher risk |
| Admin consent | Required for high-risk permissions | Covered deeper in workbook 03 |
| Client secrets | Avoid when certificate auth is possible | If used, set short expiration and track rotation |
| Certificates | Preferred for confidential automation apps | Better than shared secret strings |
| Owners | Minimum two owners | Prevent orphaned registrations |
| Naming | Clear and environment-specific | Example: `APP-CRM-Prod-OIDC` |
| Notes / documentation | Required | Track business owner, technical owner, rotation date |

## Prerequisites

| Requirement | Details |
|---|---|
| Role | Application Administrator, Cloud Application Administrator, or Global Administrator |
| Graph PowerShell | Installed and available |
| Test owner accounts | Two valid users |
| Redirect URI | Known before configuration |
| Permission list | Known before admin consent |
| Change window | Required for production apps |
| Secret storage | Use Key Vault, password manager, or approved vault |
| Certificate source | Valid certificate if using cert auth |

## Variables

Replace placeholders before running commands.

~~~powershell
$TenantId = "<tenant-id>"
$AppRegistrationName = "<app-registration-name>"
$RedirectUri = "<redirect-uri>"
$LogoutUrl = "<logout-url>"
$OwnerUpn1 = "<owner1-upn>"
$OwnerUpn2 = "<owner2-upn>"
$SecretDisplayName = "client-secret-<yyyy-mm-dd>"
$CertificatePath = "C:\Path\To\certificate.cer"
~~~

## Configuration_Checklist

| Step | Task | Admin Center / Portal Path | PowerShell / CLI / Graph | Expected Result |
|---:|---|---|---|---|
| 1 | Sign in to Entra admin center | `https://entra.microsoft.com` | N/A | Admin portal loads |
| 2 | Open App registrations | Entra admin center > Applications > App registrations | N/A | App registration list appears |
| 3 | Create or locate app registration | App registrations > New registration | `New-MgApplication` or `Get-MgApplication` | App object exists |
| 4 | Review supported account type | App registration > Authentication | `SignInAudience` | Audience matches design |
| 5 | Configure redirect URI | App registration > Authentication > Platform configuration | `Update-MgApplication` | Redirect URI is configured |
| 6 | Configure logout URL | App registration > Authentication | `Web.LogoutUrl` | Logout URL is configured if required |
| 7 | Disable unnecessary implicit grants | App registration > Authentication | `Web.ImplicitGrantSettings` | Tokens not exposed unless required |
| 8 | Configure API permissions | App registration > API permissions | `RequiredResourceAccess` | Required permissions appear |
| 9 | Grant admin consent if approved | API permissions > Grant admin consent | Graph or portal | Consent granted where appropriate |
| 10 | Create client secret if needed | App registration > Certificates & secrets | `Add-MgApplicationPassword` | Secret is created and captured once |
| 11 | Upload certificate if needed | App registration > Certificates & secrets | `Add-MgApplicationKey` | Certificate credential appears |
| 12 | Add app owners | App registration > Owners | `New-MgApplicationOwnerByRef` | At least two owners assigned |
| 13 | Confirm enterprise app exists | Enterprise applications | `Get-MgServicePrincipal` | Service principal exists |
| 14 | Validate app registration state | App registration > Overview | `Get-MgApplication` | Client ID, object ID, tenant ID documented |
| 15 | Review audit logs | Entra admin center > Audit logs | `Get-MgAuditLogDirectoryAudit` | Changes are logged |
| 16 | Document final state | Workbook notes / repo | N/A | App registration configuration documented |

## Step_01_Connect_To_Microsoft_Graph

~~~powershell
Install-Module Microsoft.Graph -Scope CurrentUser -Force

Connect-MgGraph -TenantId $TenantId -Scopes `
  "Application.ReadWrite.All", `
  "Directory.ReadWrite.All", `
  "AppRoleAssignment.ReadWrite.All", `
  "AuditLog.Read.All"

Get-MgContext
~~~

Expected result:

~~~text
Microsoft Graph connection succeeds.
Tenant ID matches the target tenant.
Required scopes are present.
~~~

## Step_02_Create_Or_Locate_App_Registration

### Portal

| Step | Action |
|---:|---|
| 1 | Go to `https://entra.microsoft.com` |
| 2 | Open Applications |
| 3 | Select App registrations |
| 4 | Search for `<app-registration-name>` |
| 5 | If missing, select New registration |
| 6 | Enter app name |
| 7 | Select supported account type |
| 8 | Add redirect URI if known |
| 9 | Select Register |

### PowerShell - Locate Existing App

~~~powershell
$App = Get-MgApplication -Filter "displayName eq '$AppRegistrationName'"

$App | Select-Object DisplayName, Id, AppId, SignInAudience
~~~

### PowerShell - Create New Single-Tenant App

~~~powershell
if (-not $App) {
    $App = New-MgApplication `
      -DisplayName $AppRegistrationName `
      -SignInAudience "AzureADMyOrg"
}

$App | Select-Object DisplayName, Id, AppId, SignInAudience
~~~

Expected result:

~~~text
App registration exists.
Application object ID and client ID are visible.
SignInAudience matches the intended account type.
~~~

## Step_03_Record_App_Identifiers

The Overview page contains the identifiers most admins need.

### Portal

| Field | Portal Location |
|---|---|
| Display name | App registration > Overview |
| Application/client ID | App registration > Overview |
| Directory/tenant ID | App registration > Overview |
| Object ID | App registration > Overview |

### PowerShell

~~~powershell
$App = Get-MgApplication -ApplicationId $App.Id

$App | Select-Object `
  DisplayName,
  Id,
  AppId,
  SignInAudience,
  CreatedDateTime
~~~

Document:

~~~text
Application display name:
Application object ID:
Application/client ID:
Directory/tenant ID:
Supported account type:
Created date:
~~~

## Step_04_Configure_Supported_Account_Type

Supported account type determines who can sign in.

| SignInAudience Value | Meaning | Use Case |
|---|---|---|
| `AzureADMyOrg` | Single tenant | Internal line-of-business apps |
| `AzureADMultipleOrgs` | Any Microsoft Entra tenant | Multi-tenant SaaS apps |
| `AzureADandPersonalMicrosoftAccount` | Work/school plus personal Microsoft accounts | Consumer-facing apps |
| `PersonalMicrosoftAccount` | Personal Microsoft accounts only | Consumer apps |

Baseline:

~~~text
Use AzureADMyOrg unless there is a documented business and security requirement for multi-tenant access.
~~~

### PowerShell

~~~powershell
Update-MgApplication `
  -ApplicationId $App.Id `
  -SignInAudience "AzureADMyOrg"

Get-MgApplication -ApplicationId $App.Id |
Select-Object DisplayName, SignInAudience
~~~

Expected result:

~~~text
SignInAudience is AzureADMyOrg for a single-tenant internal app.
~~~

## Step_05_Configure_Redirect_URI

Redirect URIs must be exact. Do not leave stale redirect URIs in production app registrations.

### Common Redirect URI Examples

| App Type | Redirect URI Example |
|---|---|
| Local development | `http://localhost:3000/auth/callback` |
| Web app | `https://app.contoso.com/signin-oidc` |
| SPA | `https://app.contoso.com/auth/callback` |
| Postman test | `https://oauth.pstmn.io/v1/callback` |
| Azure App Service | `https://<app-name>.azurewebsites.net/.auth/login/aad/callback` |

### Portal

| Step | Action |
|---:|---|
| 1 | Open App registrations |
| 2 | Select `<app-registration-name>` |
| 3 | Select Authentication |
| 4 | Select Add a platform |
| 5 | Choose Web, SPA, Mobile and desktop, or Public client |
| 6 | Add exact redirect URI |
| 7 | Save |

### PowerShell - Web Redirect URI

~~~powershell
$WebConfig = @{
    RedirectUris = @($RedirectUri)
    LogoutUrl = $LogoutUrl
    ImplicitGrantSettings = @{
        EnableAccessTokenIssuance = $false
        EnableIdTokenIssuance = $false
    }
}

Update-MgApplication `
  -ApplicationId $App.Id `
  -Web $WebConfig

Get-MgApplication -ApplicationId $App.Id |
Select-Object -ExpandProperty Web
~~~

Expected result:

~~~text
Redirect URI is configured.
Implicit grant is disabled unless specifically required.
Logout URL is set if required.
~~~

## Step_06_Configure_SPA_Redirect_URI_If_Required

Use SPA platform settings only for browser-based single-page apps.

~~~powershell
$SpaConfig = @{
    RedirectUris = @("https://<spa-app-fqdn>/auth/callback")
}

Update-MgApplication `
  -ApplicationId $App.Id `
  -Spa $SpaConfig

Get-MgApplication -ApplicationId $App.Id |
Select-Object -ExpandProperty Spa
~~~

Expected result:

~~~text
SPA redirect URI is configured only if the app is actually a SPA.
~~~

Security note:

~~~text
Do not configure SPA redirect URIs for confidential server-side web apps.
Do not use wildcard redirect URIs.
Remove localhost redirect URIs from production apps unless explicitly required.
~~~

## Step_07_Configure_Public_Client_Flows_If_Required

Public client flows are used by native apps, mobile apps, device code flows, and some CLI scenarios.

Baseline:

~~~text
Disable public client flows unless the app design requires native/mobile/CLI authentication.
~~~

### PowerShell - Disable Public Client Flow

~~~powershell
Update-MgApplication `
  -ApplicationId $App.Id `
  -IsFallbackPublicClient:$false

Get-MgApplication -ApplicationId $App.Id |
Select-Object DisplayName, IsFallbackPublicClient
~~~

Expected result:

~~~text
IsFallbackPublicClient is False unless public client flows are required.
~~~

### PowerShell - Configure Native Redirect URI If Required

~~~powershell
$PublicClientConfig = @{
    RedirectUris = @("http://localhost")
}

Update-MgApplication `
  -ApplicationId $App.Id `
  -PublicClient $PublicClientConfig
~~~

Expected result:

~~~text
Native redirect URI is present only for approved public client use cases.
~~~

## Step_08_Review_API_Permission_Types

API permissions come in two major types.

| Permission Type | Runs As | Example | Risk |
|---|---|---|---|
| Delegated | Signed-in user | User reads own profile | Depends on user and permission |
| Application | App/service itself | Daemon reads all users | Higher risk because no user context |

Baseline:

~~~text
Use the least privileged permission.
Prefer delegated permissions for user-context apps.
Use application permissions only for daemon/background/service workloads.
Require admin approval for high-risk Graph permissions.
~~~

## Step_09_Add_Basic_Microsoft_Graph_Delegated_Permission

This example adds Microsoft Graph delegated `User.Read`.

Microsoft Graph service principal AppId:

~~~text
00000003-0000-0000-c000-000000000000
~~~

### PowerShell

~~~powershell
$GraphSp = Get-MgServicePrincipal -Filter "appId eq '00000003-0000-0000-c000-000000000000'"

$UserReadScope = $GraphSp.Oauth2PermissionScopes |
Where-Object { $_.Value -eq "User.Read" }

$RequiredResourceAccess = @(
    @{
        ResourceAppId = $GraphSp.AppId
        ResourceAccess = @(
            @{
                Id = $UserReadScope.Id
                Type = "Scope"
            }
        )
    }
)

Update-MgApplication `
  -ApplicationId $App.Id `
  -RequiredResourceAccess $RequiredResourceAccess

Get-MgApplication -ApplicationId $App.Id |
Select-Object -ExpandProperty RequiredResourceAccess
~~~

Expected result:

~~~text
Microsoft Graph delegated User.Read appears under configured API permissions.
~~~

## Step_10_Add_Microsoft_Graph_Application_Permission_If_Required

Example: `User.Read.All` as an application permission.

Do not add this casually. Application permissions can grant broad directory access without a signed-in user.

~~~powershell
$GraphSp = Get-MgServicePrincipal -Filter "appId eq '00000003-0000-0000-c000-000000000000'"

$UserReadAllAppRole = $GraphSp.AppRoles |
Where-Object {
    $_.Value -eq "User.Read.All" -and
    $_.AllowedMemberTypes -contains "Application"
}

$RequiredResourceAccess = @(
    @{
        ResourceAppId = $GraphSp.AppId
        ResourceAccess = @(
            @{
                Id = $UserReadAllAppRole.Id
                Type = "Role"
            }
        )
    }
)

Update-MgApplication `
  -ApplicationId $App.Id `
  -RequiredResourceAccess $RequiredResourceAccess
~~~

Expected result:

~~~text
Microsoft Graph application permission User.Read.All appears on the app registration.
Admin consent is still required before the app can use it.
~~~

## Step_11_Add_Multiple_API_Permissions

When adding multiple permissions, preserve the existing list or deliberately replace it. A careless update can wipe existing permissions.

### Read Existing Permissions

~~~powershell
$App = Get-MgApplication -ApplicationId $App.Id

$App.RequiredResourceAccess
~~~

### Build Combined Permission Set

Example with delegated `User.Read` and application `User.Read.All`:

~~~powershell
$GraphSp = Get-MgServicePrincipal -Filter "appId eq '00000003-0000-0000-c000-000000000000'"

$UserReadScope = $GraphSp.Oauth2PermissionScopes |
Where-Object { $_.Value -eq "User.Read" }

$UserReadAllAppRole = $GraphSp.AppRoles |
Where-Object {
    $_.Value -eq "User.Read.All" -and
    $_.AllowedMemberTypes -contains "Application"
}

$CombinedGraphAccess = @{
    ResourceAppId = $GraphSp.AppId
    ResourceAccess = @(
        @{
            Id = $UserReadScope.Id
            Type = "Scope"
        },
        @{
            Id = $UserReadAllAppRole.Id
            Type = "Role"
        }
    )
}

Update-MgApplication `
  -ApplicationId $App.Id `
  -RequiredResourceAccess @($CombinedGraphAccess)
~~~

Expected result:

~~~text
Configured API permissions reflect the intended full permission list.
Existing required permissions are not accidentally removed unless that was intended.
~~~

## Step_12_Grant_Admin_Consent_When_Approved

Admin consent should follow approval. Do not use admin consent as a substitute for review.

### Portal

| Step | Action |
|---:|---|
| 1 | Open App registrations |
| 2 | Select `<app-registration-name>` |
| 3 | Select API permissions |
| 4 | Review configured permissions |
| 5 | Select Grant admin consent for `<tenant-name>` |
| 6 | Confirm |
| 7 | Verify status shows granted |

### Graph Concept

Admin consent creates OAuth2 permission grants or app role assignments against the service principal.

The deeper governance baseline is handled in:

~~~text
03_Configure_Admin_Consent_User_Consent_And_Publisher_Verification_Baseline.md
~~~

Expected result:

~~~text
Approved permissions show as granted for the tenant.
~~~

## Step_13_Create_Client_Secret_If_Required

Client secrets are sensitive. Capture the secret value immediately. You cannot retrieve it later.

Baseline:

~~~text
Prefer certificates or managed identities where possible.
If a secret is required, use short expiration and store it in an approved vault.
Do not paste secrets into tickets, chat, email, or source code.
~~~

### Portal

| Step | Action |
|---:|---|
| 1 | Open App registrations |
| 2 | Select `<app-registration-name>` |
| 3 | Select Certificates & secrets |
| 4 | Select Client secrets |
| 5 | Select New client secret |
| 6 | Enter description |
| 7 | Choose expiration |
| 8 | Select Add |
| 9 | Copy secret Value immediately |
| 10 | Store in approved vault |

### PowerShell

~~~powershell
$PasswordCredential = @{
    DisplayName = $SecretDisplayName
    EndDateTime = (Get-Date).AddMonths(6)
}

$Secret = Add-MgApplicationPassword `
  -ApplicationId $App.Id `
  -PasswordCredential $PasswordCredential

$Secret | Select-Object DisplayName, SecretText, EndDateTime
~~~

Expected result:

~~~text
Client secret is created.
SecretText is copied immediately and stored securely.
Expiration date is documented.
~~~

## Step_14_Upload_Certificate_If_Required

Certificates are preferred over client secrets for many confidential client scenarios.

### Certificate Guidance

| Item | Baseline |
|---|---|
| Key length | 2048-bit RSA or stronger |
| Private key | Stored securely, not uploaded to Entra |
| Uploaded file | Public certificate `.cer` |
| Rotation | Track expiration and rotate before outage |
| Ownership | Technical owner must know where private key lives |

### PowerShell - Add Certificate Credential

~~~powershell
$CertBytes = [System.IO.File]::ReadAllBytes($CertificatePath)
$CertBase64 = [System.Convert]::ToBase64String($CertBytes)

$KeyCredential = @{
    Type = "AsymmetricX509Cert"
    Usage = "Verify"
    Key = [System.Convert]::FromBase64String($CertBase64)
    DisplayName = "certificate-<yyyy-mm-dd>"
    StartDateTime = Get-Date
    EndDateTime = (Get-Date).AddYears(1)
}

Add-MgApplicationKey `
  -ApplicationId $App.Id `
  -KeyCredential $KeyCredential
~~~

Expected result:

~~~text
Certificate credential is added to the app registration.
The private key remains outside Entra and is protected by the application owner.
~~~

## Step_15_Validate_Secrets_And_Certificates

~~~powershell
$App = Get-MgApplication -ApplicationId $App.Id

$App.PasswordCredentials |
Select-Object DisplayName, StartDateTime, EndDateTime, KeyId

$App.KeyCredentials |
Select-Object DisplayName, StartDateTime, EndDateTime, KeyId, Type, Usage
~~~

Expected result:

~~~text
Only approved active secrets and certificates are present.
Expired or unknown credentials are removed after confirming they are unused.
~~~

## Step_16_Add_App_Owners

Owners are required for lifecycle, permission review, secret rotation, and incident response.

Baseline:

~~~text
At least two owners.
Use named accounts.
Avoid shared accounts.
Avoid ownerless app registrations.
~~~

### Portal

| Step | Action |
|---:|---|
| 1 | Open App registrations |
| 2 | Select `<app-registration-name>` |
| 3 | Select Owners |
| 4 | Select Add owners |
| 5 | Add `<owner1-upn>` |
| 6 | Add `<owner2-upn>` |
| 7 | Save |

### PowerShell

~~~powershell
$Owner1 = Get-MgUser -UserId $OwnerUpn1
$Owner2 = Get-MgUser -UserId $OwnerUpn2

New-MgApplicationOwnerByRef `
  -ApplicationId $App.Id `
  -BodyParameter @{
    "@odata.id" = "https://graph.microsoft.com/v1.0/directoryObjects/$($Owner1.Id)"
  }

New-MgApplicationOwnerByRef `
  -ApplicationId $App.Id `
  -BodyParameter @{
    "@odata.id" = "https://graph.microsoft.com/v1.0/directoryObjects/$($Owner2.Id)"
  }
~~~

### Validate Owners

~~~powershell
Get-MgApplicationOwner -ApplicationId $App.Id |
Select-Object Id, AdditionalProperties
~~~

Expected result:

~~~text
At least two owners are assigned to the app registration.
~~~

## Step_17_Confirm_Service_Principal_Exists

App registrations often have a corresponding service principal. Enterprise Application workflows depend on it.

~~~powershell
$ServicePrincipal = Get-MgServicePrincipal -Filter "appId eq '$($App.AppId)'"

if (-not $ServicePrincipal) {
    $ServicePrincipal = New-MgServicePrincipal -AppId $App.AppId
}

$ServicePrincipal | Select-Object DisplayName, Id, AppId, AccountEnabled, AppRoleAssignmentRequired
~~~

Expected result:

~~~text
A service principal exists for the app registration.
The service principal can be managed under Enterprise Applications.
~~~

## Step_18_Review_Application_Manifest_Baseline

The manifest exposes advanced app settings. Use it carefully.

### Portal

| Step | Action |
|---:|---|
| 1 | Open App registrations |
| 2 | Select `<app-registration-name>` |
| 3 | Select Manifest |
| 4 | Review settings |
| 5 | Do not edit casually |

### High-Value Manifest Fields

| Field | Meaning |
|---|---|
| `appId` | Client ID |
| `signInAudience` | Supported account type |
| `requiredResourceAccess` | API permissions requested |
| `passwordCredentials` | Client secrets metadata |
| `keyCredentials` | Certificate metadata |
| `web` | Web redirect URIs and implicit grant settings |
| `spa` | SPA redirect URIs |
| `publicClient` | Native/public client redirect URIs |
| `appRoles` | App roles exposed by the app |
| `identifierUris` | App ID URI for exposed API |
| `optionalClaims` | Additional token claims |

Expected result:

~~~text
Manifest configuration matches the approved design.
No stale redirect URIs, unused credentials, or excessive permissions are present.
~~~

## Step_19_Review_Audit_Logs

### Portal

| Step | Action |
|---:|---|
| 1 | Go to Entra admin center |
| 2 | Open Identity > Monitoring & health |
| 3 | Select Audit logs |
| 4 | Filter activity by application name |
| 5 | Review app creation, permission updates, owner changes, and secret/cert changes |

### PowerShell

~~~powershell
Get-MgAuditLogDirectoryAudit -Top 50 |
Where-Object {
    $_.TargetResources.DisplayName -contains $AppRegistrationName -or
    $_.ActivityDisplayName -match "application|credential|owner|permission"
} |
Select-Object ActivityDateTime, ActivityDisplayName, Result, InitiatedBy
~~~

Expected result:

~~~text
Application registration changes are visible in audit logs.
~~~

## Step_20_Document_Final_State

| Field | Value |
|---|---|
| App registration name | `<app-registration-name>` |
| Application object ID | `<application-object-id>` |
| Application/client ID | `<application-client-id>` |
| Tenant ID | `<tenant-id>` |
| Supported account type | `<sign-in-audience>` |
| Redirect URI list | `<redirect-uri-list>` |
| Logout URL | `<logout-url>` |
| API permissions | `<permission-list>` |
| Admin consent granted | Yes / No |
| Secret configured | Yes / No |
| Secret expiration | `<yyyy-mm-dd>` |
| Certificate configured | Yes / No |
| Certificate expiration | `<yyyy-mm-dd>` |
| Owners | `<owner1-upn>`, `<owner2-upn>` |
| Enterprise app exists | Yes / No |
| Validated by | `<admin-upn>` |
| Validation date | `<yyyy-mm-dd>` |

## Verification_Commands

### Confirm App Registration

~~~powershell
Get-MgApplication -ApplicationId $App.Id |
Select-Object DisplayName, Id, AppId, SignInAudience
~~~

### Confirm Redirect URIs

~~~powershell
$App = Get-MgApplication -ApplicationId $App.Id

$App.Web
$App.Spa
$App.PublicClient
~~~

### Confirm API Permissions

~~~powershell
$App.RequiredResourceAccess
~~~

### Confirm Secrets

~~~powershell
$App.PasswordCredentials |
Select-Object DisplayName, StartDateTime, EndDateTime, KeyId
~~~

### Confirm Certificates

~~~powershell
$App.KeyCredentials |
Select-Object DisplayName, StartDateTime, EndDateTime, KeyId, Type, Usage
~~~

### Confirm Owners

~~~powershell
Get-MgApplicationOwner -ApplicationId $App.Id |
Select-Object Id, AdditionalProperties
~~~

### Confirm Service Principal

~~~powershell
Get-MgServicePrincipal -Filter "appId eq '$($App.AppId)'" |
Select-Object DisplayName, Id, AppId, AccountEnabled
~~~

## Failure_Checks

| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| App cannot sign in users | Wrong redirect URI | App registration > Authentication | Add exact redirect URI |
| AADSTS reply URL mismatch | Redirect URI mismatch | Error details and app Authentication blade | Correct redirect URI |
| Token missing expected permission | Permission not configured or not consented | API permissions blade | Add permission and grant consent if approved |
| App gets insufficient privileges | Using delegated token without required scope | Decode token and check `scp` claim | Request correct delegated scope |
| Daemon app fails permission check | Application permission not consented | Check `roles` claim and admin consent | Grant approved app permission |
| Secret value lost | Secret copied only once | Certificates & secrets | Create new secret and update app config |
| Certificate auth fails | Wrong certificate or private key mismatch | Compare thumbprint and private key | Upload matching public cert and use matching private key |
| Owner cannot manage app | Owner missing or lacks directory permissions | App owners and role assignment | Add owner or proper admin role |
| Graph update wipes permissions | RequiredResourceAccess overwritten | Compare previous config | Rebuild full permission list |
| Multi-tenant app unexpected | Wrong SignInAudience | App manifest or Overview | Change to single tenant if appropriate |
| Public client flow risk | IsFallbackPublicClient enabled unnecessarily | App properties | Disable unless required |
| Stale localhost URI in prod | Dev URI left behind | Authentication redirect list | Remove unused redirect URI |

## Rollback

Rollback depends on what changed. Confirm production impact before removing credentials or permissions.

### Remove Client Secret

~~~powershell
$App = Get-MgApplication -ApplicationId $App.Id

$SecretToRemove = $App.PasswordCredentials |
Where-Object { $_.DisplayName -eq $SecretDisplayName } |
Select-Object -First 1

if ($SecretToRemove) {
    Remove-MgApplicationPassword `
      -ApplicationId $App.Id `
      -KeyId $SecretToRemove.KeyId
}
~~~

Expected result:

~~~text
Selected client secret is removed.
Any app using that secret will fail until reconfigured.
~~~

### Remove Certificate Credential

~~~powershell
$App = Get-MgApplication -ApplicationId $App.Id

$CertToRemove = $App.KeyCredentials |
Where-Object { $_.DisplayName -eq "certificate-<yyyy-mm-dd>" } |
Select-Object -First 1

if ($CertToRemove) {
    Remove-MgApplicationKey `
      -ApplicationId $App.Id `
      -KeyId $CertToRemove.KeyId `
      -Proof "<proof-token-if-required>"
}
~~~

Expected result:

~~~text
Selected certificate credential is removed.
Any app using that certificate will fail until reconfigured.
~~~

### Reset Required API Permissions

Use this only if the approved rollback is to clear required API permission declarations.

~~~powershell
Update-MgApplication `
  -ApplicationId $App.Id `
  -RequiredResourceAccess @()
~~~

Expected result:

~~~text
Required API permissions are cleared from the app registration.
Existing grants may still need separate cleanup.
~~~

### Remove Owner

~~~powershell
$OwnerToRemove = Get-MgUser -UserId $OwnerUpn2

Remove-MgApplicationOwnerByRef `
  -ApplicationId $App.Id `
  -DirectoryObjectId $OwnerToRemove.Id
~~~

Expected result:

~~~text
Selected owner is removed.
At least one responsible owner should remain.
~~~

### Disable Public Client Flow

~~~powershell
Update-MgApplication `
  -ApplicationId $App.Id `
  -IsFallbackPublicClient:$false
~~~

Expected result:

~~~text
Public client fallback is disabled.
~~~

## Security_Baseline

| Control | Baseline |
|---|---|
| Supported account type | Single tenant unless multi-tenant is approved |
| Redirect URIs | Exact, minimal, no stale values |
| Localhost redirect URIs | Dev only |
| Implicit grant | Disabled unless legacy requirement exists |
| Public client flow | Disabled unless native/mobile/CLI requirement exists |
| API permissions | Least privilege |
| Application permissions | Approved and reviewed |
| Admin consent | Granted only after review |
| Secrets | Avoid when certificate auth is possible |
| Secret lifetime | Short and tracked |
| Certificates | Preferred for confidential app auth |
| Owners | Minimum two |
| Audit logs | Reviewed after changes |
| Documentation | Required |

## Admin_Notes

~~~text
App registration name:
Application object ID:
Application/client ID:
Tenant ID:
Supported account type:
Redirect URI list:
Logout URL:
Delegated permissions:
Application permissions:
Admin consent granted:
Client secret name:
Client secret expiration:
Certificate thumbprint:
Certificate expiration:
Owner 1:
Owner 2:
Service principal object ID:
Validation result:
Known exceptions:
~~~

## Related_Labs

| Workbook                                                                             | Relationship                                               |
| ------------------------------------------------------------------------------------ | ---------------------------------------------------------- |
| 01_Configure_Enterprise_Applications_Service_Principals_And_Assignments.md           | Covers the Enterprise Application / service principal side |
| 03_Configure_Admin_Consent_User_Consent_And_Publisher_Verification_Baseline.md       | Covers consent governance and publisher verification       |
| 04_Configure_SAML_OIDC_OAuth_SSO_And_Application_Provisioning.md                     | Covers SSO and provisioning                                |
| 05_Configure_Conditional_Access_App_Targeting_Session_Controls_And_Access_Reviews.md | Covers Conditional Access and access reviews               |
| 06_Troubleshoot_Enterprise_App_SSO_Consent_Provisioning_And_API_Permission_Issues.md | Covers troubleshooting                                     |