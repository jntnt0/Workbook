# 06_Troubleshoot_Enterprise_App_SSO_Consent_Provisioning_And_API_Permission_Issues

## Objective

Troubleshoot Microsoft Entra Enterprise Application issues across sign-in, SSO, consent, provisioning, API permissions, Conditional Access, assignments, and app registration configuration.

This workbook is built as an operational troubleshooting runbook.

Use it when an application has symptoms such as:

- User cannot access the app
- SAML SSO fails
- OIDC sign-in fails
- OAuth token does not contain expected scopes or roles
- Admin consent appears granted but the app still fails
- User consent is blocked
- Provisioning fails or skips users
- Enterprise Application assignments are not working
- Conditional Access blocks access unexpectedly
- App registration redirect URI or credential issues occur

## Lab_Context

| Item | Value |
|---|---|
| Tenant | `<tenant-name>.onmicrosoft.com` |
| Admin portal | `https://entra.microsoft.com` |
| Primary admin center | Microsoft Entra admin center |
| Target enterprise app | `<enterprise-app-name>` |
| Target app registration | `<app-registration-name>` |
| Application/client ID | `<application-client-id>` |
| Service principal object ID | `<service-principal-object-id>` |
| Application object ID | `<application-object-id>` |
| Test user | `<test-user-upn>` |
| Affected user | `<affected-user-upn>` |
| Access group | `<app-access-group-name>` |
| Conditional Access policy | `<ca-policy-name>` |
| Error code | `<aadsts-error-code>` |
| Correlation ID | `<correlation-id>` |
| Request ID | `<request-id>` |
| Timestamp | `<yyyy-mm-dd hh:mm:ss timezone>` |
| PowerShell module | Microsoft Graph PowerShell |
| Graph permission scope | `Application.Read.All`, `Directory.Read.All`, `AuditLog.Read.All`, `Policy.Read.All`, `AppRoleAssignment.ReadWrite.All`, `DelegatedPermissionGrant.ReadWrite.All` |

## Mental_Model

Troubleshooting Entra application access requires separating five layers.

~~~text
Layer 1 - User entitlement
    |
    | Enterprise app assignment, group membership, assignment required
    v
Layer 2 - Sign-in control
    |
    | Conditional Access, MFA, device, risk, session controls
    v
Layer 3 - SSO protocol
    |
    | SAML, OIDC, OAuth, redirect URI, claims, certificates
    v
Layer 4 - Consent and API permissions
    |
    | Delegated scopes, application roles, admin consent, user consent
    v
Layer 5 - Provisioning
    |
    | SCIM endpoint, token, mappings, scope, matching, logs
~~~

Do not troubleshoot all layers at once.

Start with the error, the user, the app, and the timestamp.

Then isolate the failed layer.

## Troubleshooting_Decision_Table

| Symptom | Start Here | Most Likely Layer |
|---|---|---|
| User sees assignment required error | Enterprise app Users and groups | Entitlement |
| User blocked by MFA/device/location | Sign-in logs Conditional Access tab | Sign-in control |
| SAML assertion rejected | Enterprise app SAML settings and vendor error | SSO protocol |
| Reply URL mismatch | App registration Authentication | OIDC protocol |
| Token missing `scp` | API permissions and requested scopes | OAuth delegated permission |
| Token missing `roles` | Application permissions and app role assignments | OAuth application permission |
| Admin consent button unavailable | Admin role and permission type | Consent |
| User cannot self-consent | User consent settings | Consent policy |
| Provisioning test connection fails | Provisioning Admin Credentials | SCIM connectivity |
| Provisioning skips user | Provisioning scope or attribute mapping | Provisioning |
| User provisioned wrong | Matching attribute | Provisioning |
| App works for one user but not another | Assignment, group membership, CA, user attributes | Multiple layers |

## Prerequisites

| Requirement | Details |
|---|---|
| Role | Cloud Application Administrator, Application Administrator, Security Reader, Conditional Access Administrator, or Global Reader depending task |
| Logs | Sign-in logs, audit logs, provisioning logs |
| Error details | Error code, correlation ID, timestamp, affected user |
| Test identity | Known-good test user |
| App identifiers | Enterprise Application object ID and App Registration client ID |
| Vendor access | Vendor admin portal access for SAML/SCIM review |
| Graph PowerShell | Installed and available |
| Token inspection | Ability to decode ID/access token safely |
| Change control | Required before modifying production SSO, consent, or provisioning |

## Variables

Replace placeholders before running commands.

~~~powershell
$TenantId = "<tenant-id>"
$EnterpriseAppName = "<enterprise-app-name>"
$AppRegistrationName = "<app-registration-name>"
$ApplicationClientId = "<application-client-id>"
$ServicePrincipalObjectId = "<service-principal-object-id>"
$ApplicationObjectId = "<application-object-id>"
$AffectedUserUpn = "<affected-user-upn>"
$TestUserUpn = "<test-user-upn>"
$AccessGroupName = "<app-access-group-name>"
$ConditionalAccessPolicyName = "<ca-policy-name>"
$AadstsErrorCode = "<aadsts-error-code>"
$CorrelationId = "<correlation-id>"
$RequestId = "<request-id>"
$FailureTimestamp = "<yyyy-mm-dd hh:mm:ss timezone>"
~~~

## Configuration_Checklist

| Step | Task | Admin Center / Portal Path | PowerShell / CLI / Graph | Expected Result |
|---:|---|---|---|---|
| 1 | Capture error details | Browser error / user screenshot | N/A | Error code, timestamp, correlation ID captured |
| 2 | Locate Enterprise Application | Applications > Enterprise applications | `Get-MgServicePrincipal` | Service principal found |
| 3 | Locate App Registration | Applications > App registrations | `Get-MgApplication` | App registration found |
| 4 | Confirm app enabled state | Enterprise app > Properties | `AccountEnabled` | App is enabled unless intentionally blocked |
| 5 | Confirm assignment required | Enterprise app > Properties | `AppRoleAssignmentRequired` | Assignment behavior known |
| 6 | Confirm user/group assignment | Enterprise app > Users and groups | App role assignments | User has entitlement path |
| 7 | Confirm group membership | Identity > Groups | `Get-MgGroupMember` | User is in assigned group |
| 8 | Review sign-in logs | Monitoring & health > Sign-in logs | `Get-MgAuditLogSignIn` | Failure reason and CA result identified |
| 9 | Review Conditional Access result | Sign-in event > Conditional Access | Policy details | Blocking policy identified |
| 10 | Review SAML settings if SAML app | Enterprise app > Single sign-on | Portal | Entity ID, ACS URL, cert, claims reviewed |
| 11 | Review OIDC redirect URI if OIDC app | App registration > Authentication | `Web`, `Spa`, `PublicClient` | Redirect URI matches app |
| 12 | Review API permissions | App registration > API permissions | `RequiredResourceAccess` | Required permissions identified |
| 13 | Review delegated grants | Enterprise app > Permissions | `Get-MgOauth2PermissionGrant` | Delegated consent state known |
| 14 | Review application grants | Enterprise app > Permissions | App role assignments | Application permission state known |
| 15 | Review credentials | App registration > Certificates & secrets | `PasswordCredentials`, `KeyCredentials` | Secret/cert validity known |
| 16 | Review provisioning settings | Enterprise app > Provisioning | Portal | SCIM endpoint, scope, mappings reviewed |
| 17 | Review provisioning logs | Enterprise app > Provisioning logs | Portal | Provisioning failure reason identified |
| 18 | Compare known-good test user | Same app test | Logs | User-specific vs app-wide issue isolated |
| 19 | Apply fix under change control | Relevant blade | Graph / portal | Root cause fixed |
| 20 | Validate and document | Logs / workbook notes | Commands | Evidence captured |

## Step_01_Connect_To_Microsoft_Graph

~~~powershell
Install-Module Microsoft.Graph -Scope CurrentUser -Force

Connect-MgGraph -TenantId $TenantId -Scopes `
  "Application.Read.All", `
  "Directory.Read.All", `
  "AuditLog.Read.All", `
  "Policy.Read.All", `
  "AppRoleAssignment.ReadWrite.All", `
  "DelegatedPermissionGrant.ReadWrite.All"

Get-MgContext
~~~

Expected result:

~~~text
Microsoft Graph connection succeeds.
Tenant ID matches the target tenant.
Required scopes are present.
~~~

## Step_02_Capture_Error_Details

Before changing anything, capture the exact failure.

| Item | Capture |
|---|---|
| Affected user | `<affected-user-upn>` |
| Application name | `<enterprise-app-name>` |
| Error code | `<aadsts-error-code>` |
| Error message | `<full-error-message>` |
| Correlation ID | `<correlation-id>` |
| Request ID | `<request-id>` |
| Timestamp | `<yyyy-mm-dd hh:mm:ss timezone>` |
| Browser/device | `<browser-device>` |
| Network/location | `<source-ip-location>` |
| App launch method | My Apps / direct app URL / vendor login / mobile app |
| Scope | One user / group / all users |
| Recent change | SSO / consent / CA / provisioning / certificate / secret / app update |

Expected result:

~~~text
Troubleshooting evidence is captured before changes are made.
Correlation ID and timestamp can be used to locate sign-in log events.
~~~

## Step_03_Locate_Enterprise_Application

### Portal

| Step | Action |
|---:|---|
| 1 | Go to `https://entra.microsoft.com` |
| 2 | Open Applications |
| 3 | Select Enterprise applications |
| 4 | Search for `<enterprise-app-name>` |
| 5 | Open the app |
| 6 | Record Object ID and Application ID |

### PowerShell

~~~powershell
$Sp = Get-MgServicePrincipal -Filter "displayName eq '$EnterpriseAppName'"

$Sp | Select-Object DisplayName, Id, AppId, AccountEnabled, AppRoleAssignmentRequired
~~~

Expected result:

~~~text
Enterprise Application service principal is found.
Object ID and AppId are known.
~~~

If search by name fails:

~~~powershell
Get-MgServicePrincipal -Filter "appId eq '$ApplicationClientId'" |
Select-Object DisplayName, Id, AppId, AccountEnabled, AppRoleAssignmentRequired
~~~

Expected result:

~~~text
Service principal is found by Application/client ID.
~~~

## Step_04_Locate_App_Registration

### Portal

| Step | Action |
|---:|---|
| 1 | Open Applications |
| 2 | Select App registrations |
| 3 | Search for `<app-registration-name>` |
| 4 | If missing, search by Application/client ID |
| 5 | Record Object ID and Application/client ID |

### PowerShell

~~~powershell
$App = Get-MgApplication -Filter "appId eq '$ApplicationClientId'"

$App | Select-Object DisplayName, Id, AppId, SignInAudience
~~~

Expected result:

~~~text
App registration is found if the tenant owns the application object.
For third-party SaaS apps, the app registration may live in the vendor tenant, while your tenant only has the service principal.
~~~

## Step_05_Check_App_Enabled_State

If the Enterprise Application is disabled, users cannot sign in.

### Portal

| Step | Action |
|---:|---|
| 1 | Open Enterprise applications |
| 2 | Select `<enterprise-app-name>` |
| 3 | Select Properties |
| 4 | Check Enabled for users to sign-in |

### PowerShell

~~~powershell
Get-MgServicePrincipal -ServicePrincipalId $Sp.Id |
Select-Object DisplayName, AccountEnabled
~~~

Expected result:

~~~text
AccountEnabled is True unless the app is intentionally blocked.
~~~

Fix if approved:

~~~powershell
Update-MgServicePrincipal -ServicePrincipalId $Sp.Id -AccountEnabled:$true
~~~

## Step_06_Check_Assignment_Required_And_User_Assignment

If assignment required is enabled, the user must be directly assigned or assigned through a group.

### Portal

| Step | Action |
|---:|---|
| 1 | Open Enterprise applications |
| 2 | Select `<enterprise-app-name>` |
| 3 | Select Properties |
| 4 | Check Assignment required |
| 5 | Select Users and groups |
| 6 | Confirm user or group assignment |

### PowerShell

~~~powershell
Get-MgServicePrincipal -ServicePrincipalId $Sp.Id |
Select-Object DisplayName, AppRoleAssignmentRequired

Get-MgServicePrincipalAppRoleAssignedTo -ServicePrincipalId $Sp.Id -All |
Select-Object PrincipalDisplayName, PrincipalType, PrincipalId, AppRoleId, ResourceDisplayName
~~~

Expected result:

~~~text
Assignment required state is known.
User has a valid assignment path if assignment required is enabled.
~~~

## Step_07_Check_Affected_User_Group_Membership

If the app is assigned to a group, confirm the affected user is actually in the group.

~~~powershell
$AffectedUser = Get-MgUser -UserId $AffectedUserUpn
$AccessGroup = Get-MgGroup -Filter "displayName eq '$AccessGroupName'"

Get-MgGroupMember -GroupId $AccessGroup.Id -All |
Where-Object { $_.Id -eq $AffectedUser.Id }
~~~

Expected result:

~~~text
Affected user appears as a member of the assigned access group.
~~~

If using dynamic groups:

~~~text
Check:
- Dynamic membership rule
- User attributes used by rule
- Membership processing status
- Whether the user is a direct member of another assigned group
~~~

## Step_08_Review_Sign_In_Logs_By_User_And_App

Sign-in logs usually identify the failed layer.

### Portal

| Step | Action |
|---:|---|
| 1 | Open Entra admin center |
| 2 | Go to Identity > Monitoring & health |
| 3 | Select Sign-in logs |
| 4 | Filter User = `<affected-user-upn>` |
| 5 | Filter Application = `<enterprise-app-name>` |
| 6 | Open failed event |
| 7 | Review Status |
| 8 | Review Failure reason |
| 9 | Review Conditional Access |
| 10 | Review Authentication Details |
| 11 | Review Device info |
| 12 | Review Location |

### PowerShell

~~~powershell
Get-MgAuditLogSignIn -Filter "userPrincipalName eq '$AffectedUserUpn'" -Top 20 |
Where-Object { $_.AppDisplayName -eq $EnterpriseAppName } |
Select-Object CreatedDateTime, UserPrincipalName, AppDisplayName, Status, ConditionalAccessStatus, CorrelationId
~~~

Expected result:

~~~text
Failed sign-in event is found.
Failure reason, Conditional Access result, and correlation ID are available.
~~~

## Step_09_Review_Sign_In_By_Correlation_ID

Use this when the browser error includes correlation ID.

~~~powershell
Get-MgAuditLogSignIn -Filter "correlationId eq '$CorrelationId'" -Top 10 |
Select-Object CreatedDateTime, UserPrincipalName, AppDisplayName, Status, ConditionalAccessStatus, CorrelationId
~~~

Expected result:

~~~text
The exact sign-in event matching the user error is found.
~~~

## Step_10_Troubleshoot_Conditional_Access_Block

### Portal

| Step | Action |
|---:|---|
| 1 | Open failed sign-in event |
| 2 | Select Conditional Access tab |
| 3 | Identify policies that applied |
| 4 | Identify policies that failed |
| 5 | Review grant controls |
| 6 | Review session controls |
| 7 | Review device state |
| 8 | Review user/group targeting |

### Common CA Failure Causes

| Failure | Check | Fix |
|---|---|---|
| MFA required but not satisfied | Authentication Details | Register or complete MFA |
| Compliant device required | Device info and Intune compliance | Remediate device or adjust policy |
| Hybrid joined device required | Device join state | Join device or adjust policy |
| Block policy applied | CA tab | Exclude user/app or change policy if approved |
| User in wrong group | Policy users/groups | Correct group membership |
| Break-glass not excluded | Exclusion group | Add break-glass exclusion |
| Location condition applied | Location tab | Update named location or policy |
| Client app blocked | Client app type | Adjust client app condition |

Expected result:

~~~text
Blocking Conditional Access policy is identified.
Policy condition causing the block is known.
~~~

## Step_11_Troubleshoot_Assignment_Required_Error

Typical symptom:

~~~text
AADSTS50105: The signed in user is not assigned to a role for the application.
~~~

Checks:

| Check | Portal Path | Expected Result |
|---|---|---|
| Assignment required | Enterprise app > Properties | Yes or No known |
| User assignment | Enterprise app > Users and groups | User or group assigned |
| Group membership | Identity > Groups | User is member |
| App role | Enterprise app assignment | Correct role assigned |
| Account enabled | User profile | User account enabled |

PowerShell:

~~~powershell
Get-MgServicePrincipal -ServicePrincipalId $Sp.Id |
Select-Object DisplayName, AppRoleAssignmentRequired

Get-MgServicePrincipalAppRoleAssignedTo -ServicePrincipalId $Sp.Id -All |
Select-Object PrincipalDisplayName, PrincipalType, PrincipalId, AppRoleId

$AffectedUser = Get-MgUser -UserId $AffectedUserUpn
$AffectedUser | Select-Object DisplayName, UserPrincipalName, AccountEnabled
~~~

Fix options:

| Situation | Fix |
|---|---|
| User should have access | Add user to assigned access group |
| Group should have access | Assign group to Enterprise Application |
| App should be open to all users | Disable Assignment required if approved |
| User should not have access | No fix, denial is expected |

## Step_12_Troubleshoot_SAML_SSO_Failure

SAML failures usually come from wrong URLs, wrong claims, wrong certificate, or vendor-side mismatch.

### Portal Checks

| Check | Portal Path |
|---|---|
| SSO method | Enterprise app > Single sign-on |
| Identifier / Entity ID | SAML > Basic SAML Configuration |
| Reply URL / ACS URL | SAML > Basic SAML Configuration |
| Sign-on URL | SAML > Basic SAML Configuration |
| Logout URL | SAML > Basic SAML Configuration |
| NameID | SAML > Attributes & Claims |
| Claims | SAML > Attributes & Claims |
| Signing certificate | SAML > SAML Certificates |
| Federation metadata | SAML > SAML Certificates |

### Common SAML Symptoms

| Symptom | Likely Cause | Fix |
|---|---|---|
| Vendor says invalid issuer | Entity ID mismatch | Match vendor SP Entity ID and Entra issuer expectations |
| Vendor says invalid audience | Identifier mismatch | Correct Identifier / Entity ID |
| Vendor says invalid recipient | Reply URL mismatch | Correct ACS / Reply URL |
| Vendor says invalid signature | Wrong cert uploaded to vendor | Upload active Entra SAML cert |
| User maps to wrong account | NameID wrong | Change NameID to vendor-required attribute |
| Missing authorization in app | Group/role claim missing | Add required claim or app role mapping |
| SAML works for one user only | Attribute mismatch | Compare user mail/UPN/immutable ID |
| SAML broke suddenly | Certificate rollover | Update vendor with active certificate |

Expected result:

~~~text
SAML failure is mapped to a specific SAML setting, claim, certificate, or vendor-side trust issue.
~~~

## Step_13_Check_SAML_Certificate_Expiration

### Portal

| Step | Action |
|---:|---|
| 1 | Open Enterprise app |
| 2 | Select Single sign-on |
| 3 | Select SAML |
| 4 | Review SAML Certificates |
| 5 | Confirm active certificate expiration |
| 6 | Confirm vendor has active certificate |

Documentation:

| Field | Value |
|---|---|
| Active certificate thumbprint | `<thumbprint>` |
| Expiration | `<yyyy-mm-dd>` |
| Vendor updated | Yes / No |
| Rollover needed | Yes / No |

Expected result:

~~~text
Certificate expiration and vendor certificate state are known.
Expired or mismatched certificate is identified.
~~~

## Step_14_Troubleshoot_OIDC_Redirect_URI_Failure

Common symptom:

~~~text
AADSTS50011: The redirect URI specified in the request does not match the redirect URIs configured for the application.
~~~

### Portal

| Step | Action |
|---:|---|
| 1 | Open App registrations |
| 2 | Search by Application/client ID |
| 3 | Select Authentication |
| 4 | Review Web redirect URIs |
| 5 | Review SPA redirect URIs |
| 6 | Review Mobile and desktop redirect URIs |
| 7 | Compare exact URI from error/request |
| 8 | Add or correct redirect URI if approved |

### PowerShell

~~~powershell
$App = Get-MgApplication -Filter "appId eq '$ApplicationClientId'"

$App.Web
$App.Spa
$App.PublicClient
~~~

Expected result:

~~~text
Redirect URI configured in Entra exactly matches the redirect URI sent by the app.
Scheme, hostname, path, and trailing slash are correct.
~~~

Fix if approved:

~~~powershell
$CurrentWeb = $App.Web
$CurrentUris = @($CurrentWeb.RedirectUris)
$CorrectRedirectUri = "<correct-redirect-uri>"

if ($CurrentUris -notcontains $CorrectRedirectUri) {
    $CurrentWeb.RedirectUris = $CurrentUris + $CorrectRedirectUri

    Update-MgApplication `
      -ApplicationId $App.Id `
      -Web $CurrentWeb
}
~~~

## Step_15_Troubleshoot_OIDC_Token_Validation_Failure

OIDC token validation failures usually involve issuer, audience, signing keys, or tenant mismatch.

| Claim / Setting | Meaning | Check |
|---|---|---|
| `iss` | Issuer | App expects correct tenant issuer |
| `aud` | Audience | Must match client ID |
| `tid` | Tenant ID | Must match expected tenant |
| `exp` | Expiration | Token not expired |
| `nonce` | Replay protection | App validates expected nonce |
| Signing key | Token signature | App uses current JWKS metadata |
| Redirect URI | Return endpoint | Must match configured URI |

Metadata endpoint:

~~~text
https://login.microsoftonline.com/<tenant-id>/v2.0/.well-known/openid-configuration
~~~

Expected result:

~~~text
OIDC token issuer, audience, tenant, signature, and redirect URI match the app configuration.
~~~

## Step_16_Troubleshoot_OAuth_Delegated_Permission_Issue

Delegated permissions appear in the access token as `scp`.

Common symptom:

~~~text
API returns 403 Forbidden or insufficient privileges.
Token does not contain expected scope.
~~~

### Checks

| Check | Expected Result |
|---|---|
| API permission declared | App registration lists delegated scope |
| Admin/user consent granted | Permission grant exists |
| App requested correct scope | Auth request includes scope |
| Token contains `scp` | Access token has expected scope |
| User has resource access | User is allowed to access target data |

### PowerShell

~~~powershell
$App = Get-MgApplication -Filter "appId eq '$ApplicationClientId'"
$Sp = Get-MgServicePrincipal -Filter "appId eq '$ApplicationClientId'"

$App.RequiredResourceAccess

Get-MgOauth2PermissionGrant -All |
Where-Object { $_.ClientId -eq $Sp.Id } |
Select-Object Id, ConsentType, Scope, PrincipalId, ResourceId
~~~

Expected result:

~~~text
Delegated permission is declared and consented.
Application requests the scope during sign-in.
Access token contains the expected `scp` value.
~~~

Fix options:

| Problem | Fix |
|---|---|
| Permission not declared | Add delegated API permission |
| Permission not consented | Grant admin/user consent if approved |
| App not requesting scope | Update app auth request |
| User lacks data access | Grant resource-side access |
| Wrong token audience | Request token for correct API |

## Step_17_Troubleshoot_OAuth_Application_Permission_Issue

Application permissions appear in the access token as `roles`.

Common symptom:

~~~text
Daemon/service app receives 403 Forbidden.
Token has no `roles` claim.
Application permission appears configured but not effective.
~~~

### Checks

| Check | Expected Result |
|---|---|
| Application permission declared | App registration has permission type Role |
| Admin consent granted | App role assignment exists |
| Token requested with client credentials | No signed-in user required |
| Token contains `roles` | Access token has expected app role |
| Resource supports app-only access | API allows application permission |

### PowerShell

~~~powershell
$Sp = Get-MgServicePrincipal -Filter "appId eq '$ApplicationClientId'"

Get-MgServicePrincipalAppRoleAssignment -ServicePrincipalId $Sp.Id -All |
Select-Object Id, PrincipalDisplayName, ResourceDisplayName, AppRoleId, CreatedDateTime
~~~

Expected result:

~~~text
Application permission grant exists as an app role assignment.
Access token contains expected `roles` claim.
~~~

Fix options:

| Problem | Fix |
|---|---|
| Permission not declared | Add application API permission |
| Permission not consented | Grant admin consent if approved |
| Wrong credential | Use valid secret/certificate |
| Wrong token endpoint | Use tenant token endpoint |
| Resource does not support app-only | Use delegated permission or supported API model |

## Step_18_Troubleshoot_Admin_Consent_Issue

### Symptoms

| Symptom | Likely Cause |
|---|---|
| Grant admin consent button unavailable | Insufficient admin role |
| Admin consent granted but app still fails | App not requesting permission or wrong token |
| User keeps seeing consent prompt | Consent missing for requested dynamic scope |
| Consent request pending | Admin consent workflow waiting for reviewer |
| Consent denied | Reviewer denied request |
| Consent granted to wrong app | Similar app name or wrong AppId |

### Portal Checks

| Check | Portal Path |
|---|---|
| Required permissions | App registrations > API permissions |
| Granted permissions | Enterprise apps > Permissions |
| Admin consent requests | Enterprise apps > Admin consent requests |
| Publisher verification | App registration or Enterprise app properties |
| Audit logs | Monitoring & health > Audit logs |

### PowerShell

~~~powershell
$Sp = Get-MgServicePrincipal -Filter "appId eq '$ApplicationClientId'"

Get-MgOauth2PermissionGrant -All |
Where-Object { $_.ClientId -eq $Sp.Id } |
Select-Object Id, ConsentType, Scope, PrincipalId, ResourceId

Get-MgServicePrincipalAppRoleAssignment -ServicePrincipalId $Sp.Id -All |
Select-Object Id, PrincipalDisplayName, ResourceDisplayName, AppRoleId
~~~

Expected result:

~~~text
Consent state is separated between delegated grants and application app role assignments.
The app with the correct client ID has the expected grants.
~~~

## Step_19_Troubleshoot_User_Consent_Blocked

User consent may be restricted by tenant policy.

### Symptoms

| Symptom | Meaning |
|---|---|
| User cannot consent to app | User consent disabled or restricted |
| User can request admin approval | Admin consent workflow enabled |
| User sees app requires admin approval | Permission requires admin consent |
| User sees unverified publisher warning | Publisher verification issue |

### Portal Checks

| Check | Portal Path |
|---|---|
| User consent settings | Enterprise applications > Consent and permissions |
| Admin consent workflow | Enterprise applications > Consent and permissions |
| Permission classification | Enterprise applications > Consent and permissions |
| Publisher verification | App registration / Enterprise app properties |

Expected result:

~~~text
User consent block is confirmed as tenant policy behavior or high-risk permission behavior.
If access is approved, route through admin consent workflow.
~~~

## Step_20_Troubleshoot_Client_Secret_Or_Certificate_Issue

Credential failures usually affect confidential client apps, daemon apps, automations, and API integrations.

### Symptoms

| Symptom | Likely Cause |
|---|---|
| `invalid_client` | Wrong client secret/certificate |
| Secret stopped working | Secret expired or rotated |
| Certificate auth fails | Wrong private key or uploaded public cert mismatch |
| App-only token fails | Credential invalid or wrong tenant/client |
| Works in dev but not prod | Wrong app registration or secret |

### Portal Checks

| Check | Portal Path |
|---|---|
| Client secret expiration | App registration > Certificates & secrets |
| Certificate expiration | App registration > Certificates & secrets |
| Credential display name | App registration > Certificates & secrets |
| App/client ID | App registration > Overview |
| Tenant ID | App registration > Overview |

### PowerShell

~~~powershell
$App = Get-MgApplication -Filter "appId eq '$ApplicationClientId'"

$App.PasswordCredentials |
Select-Object DisplayName, StartDateTime, EndDateTime, KeyId

$App.KeyCredentials |
Select-Object DisplayName, StartDateTime, EndDateTime, KeyId, Type, Usage
~~~

Expected result:

~~~text
Active non-expired credential exists.
Application is using the correct tenant ID, client ID, and current credential.
~~~

Fix options:

| Problem | Fix |
|---|---|
| Secret expired | Create new secret and update app securely |
| Secret value lost | Create new secret because old value cannot be retrieved |
| Wrong certificate | Upload matching public certificate and use matching private key |
| Certificate expired | Upload new certificate and rotate app configuration |
| Wrong tenant/client | Correct application configuration |

## Step_21_Troubleshoot_SCIM_Test_Connection_Failure

### Portal

| Step | Action |
|---:|---|
| 1 | Open Enterprise applications |
| 2 | Select `<enterprise-app-name>` |
| 3 | Select Provisioning |
| 4 | Select Admin Credentials |
| 5 | Review Tenant URL |
| 6 | Review Secret Token |
| 7 | Select Test Connection |
| 8 | Record error |

### Common Causes

| Failure | Likely Cause | Fix |
|---|---|---|
| Unauthorized | Bad SCIM token | Regenerate token in app and update Entra |
| Not found | Wrong tenant URL | Correct SCIM endpoint URL |
| Timeout | Network/vendor outage | Confirm vendor endpoint availability |
| Forbidden | Token lacks provisioning rights | Create token with correct vendor permissions |
| TLS/certificate error | Vendor endpoint certificate issue | Fix vendor TLS/cert |
| Rate limited | Vendor throttling | Retry later or contact vendor |
| Schema error | Vendor SCIM incompatibility | Review vendor SCIM docs |

Expected result:

~~~text
SCIM test connection succeeds before provisioning is enabled.
If it fails, exact vendor/API issue is identified.
~~~

## Step_22_Troubleshoot_User_Not_Provisioned

### Checks

| Check | Expected Result |
|---|---|
| User assigned to app | User or group assigned |
| Provisioning scope | Sync only assigned users/groups or all users/groups |
| User account enabled | User active |
| Required attributes present | mail, UPN, givenName, surname as needed |
| Attribute mapping valid | Source values map to target |
| Provisioning service running | Provisioning status On |
| Provisioning logs | User has create/update/skip event |

### Portal

| Step | Action |
|---:|---|
| 1 | Open Enterprise app |
| 2 | Select Provisioning |
| 3 | Select Provisioning logs |
| 4 | Filter by `<affected-user-upn>` |
| 5 | Review action |
| 6 | Review status |
| 7 | Review skipped reason or error |
| 8 | Use Provision on demand for test |

Expected result:

~~~text
Reason for missing provisioning is identified.
User is either out of scope, missing attributes, skipped, or failed.
~~~

## Step_23_Troubleshoot_Provisioning_Attribute_Mapping

### Common Mapping Failures

| Symptom | Likely Cause | Fix |
|---|---|---|
| User created with wrong username | Wrong `userName` mapping | Map to correct UPN/mail |
| User matched to wrong account | Bad matching precedence | Use stable unique matching attribute |
| User skipped | Required source attribute empty | Populate source attribute or change mapping |
| Department/title wrong | Attribute source mismatch | Correct mapping |
| User disabled unexpectedly | `active` mapping issue | Review accountEnabled to active mapping |
| Group not provisioned | Group provisioning unsupported or out of scope | Disable group provisioning or correct scope |

### Portal

| Step | Action |
|---:|---|
| 1 | Open Enterprise app |
| 2 | Select Provisioning |
| 3 | Select Mappings |
| 4 | Open user mapping |
| 5 | Review matching precedence |
| 6 | Review required attributes |
| 7 | Review transformations |
| 8 | Save only after impact review |

Expected result:

~~~text
Attribute mappings match the vendor requirements.
Matching attribute is stable and unique.
Required attributes are populated.
~~~

## Step_24_Troubleshoot_Provisioning_Deletes_Or_Disables

Provisioning may disable users when they fall out of scope. This can be correct but must be understood.

### Checks

| Check | Meaning |
|---|---|
| User removed from assignment group | User may be disabled in app |
| User disabled in Entra | User may be disabled in app |
| Provisioning scope changed | Many users may fall out of scope |
| Accidental deletion threshold | Protects against mass changes |
| App delete behavior | Vendor may delete or deactivate |

### Emergency Action

~~~text
If provisioning is disabling too many users:
1. Open Enterprise app.
2. Go to Provisioning.
3. Set Provisioning Status to Off.
4. Save.
5. Review scope, mappings, and recent group changes.
6. Contact vendor if destructive changes already occurred.
~~~

Expected result:

~~~text
Provisioning is stopped before further unwanted changes occur.
Root cause is isolated before re-enabling.
~~~

## Step_25_Compare_Affected_User_To_Known_Good_User

Use comparison when one user fails and another succeeds.

| Check | Affected User | Known Good User |
|---|---|---|
| Account enabled | `<value>` | `<value>` |
| UPN | `<value>` | `<value>` |
| Mail | `<value>` | `<value>` |
| Assigned to app | `<value>` | `<value>` |
| Access group membership | `<value>` | `<value>` |
| MFA registered | `<value>` | `<value>` |
| Device compliant | `<value>` | `<value>` |
| Conditional Access result | `<value>` | `<value>` |
| SAML NameID | `<value>` | `<value>` |
| Provisioned in app | `<value>` | `<value>` |
| App-side role | `<value>` | `<value>` |

PowerShell user check:

~~~powershell
Get-MgUser -UserId $AffectedUserUpn |
Select-Object DisplayName, UserPrincipalName, Mail, AccountEnabled, Id

Get-MgUser -UserId $TestUserUpn |
Select-Object DisplayName, UserPrincipalName, Mail, AccountEnabled, Id
~~~

Expected result:

~~~text
Difference between failing user and working user is identified.
Troubleshooting narrows from app-wide issue to user-specific issue.
~~~

## Step_26_Review_Audit_Logs_For_Recent_Changes

Recent changes often explain sudden failures.

### Portal

| Step | Action |
|---:|---|
| 1 | Open Entra admin center |
| 2 | Go to Identity > Monitoring & health |
| 3 | Select Audit logs |
| 4 | Filter around failure time |
| 5 | Filter by application name |
| 6 | Review app, consent, CA, group, provisioning, and credential changes |

### PowerShell

~~~powershell
Get-MgAuditLogDirectoryAudit -Top 100 |
Where-Object {
    $_.ActivityDisplayName -match "application|service principal|permission|consent|conditional access|group|credential|provisioning"
} |
Select-Object ActivityDateTime, ActivityDisplayName, Result, InitiatedBy
~~~

Expected result:

~~~text
Recent configuration changes are identified or ruled out.
Change initiator and timestamp are documented.
~~~

## Step_27_Root_Cause_Matrix

| Root Cause | Evidence | Corrective Action |
|---|---|---|
| User not assigned | App assignment list lacks user/group | Assign user/group if approved |
| Assignment required misconfigured | AppRoleAssignmentRequired unexpected | Enable/disable based on design |
| CA block | Sign-in log CA tab shows failure | Adjust policy or remediate user/device |
| MFA issue | Authentication Details shows MFA failure | Register or complete MFA |
| Device compliance issue | Device not compliant | Remediate Intune compliance |
| SAML reply URL mismatch | Vendor/Entra ACS mismatch | Correct Reply URL |
| SAML cert mismatch | Vendor has old cert | Upload active cert to vendor |
| SAML claim mismatch | NameID or required claim wrong | Correct claims |
| OIDC redirect URI mismatch | AADSTS50011 | Add exact redirect URI |
| OAuth delegated scope missing | Token lacks `scp` | Add/request/consent delegated scope |
| OAuth app role missing | Token lacks `roles` | Grant app permission |
| Admin consent missing | No grant/app role assignment | Grant consent if approved |
| User consent blocked | Tenant policy restricts consent | Use admin consent workflow |
| Secret expired | Credential expired | Rotate secret |
| Certificate expired | KeyCredential expired | Rotate certificate |
| SCIM token invalid | Test connection unauthorized | Update SCIM token |
| User out of provisioning scope | Provisioning log skipped | Assign user/group |
| Attribute mapping broken | Provisioning log mapping error | Fix mapping/source attribute |

## Step_28_Validate_Fix

After applying a fix, validate from the user path and from logs.

| Validation | Expected Result |
|---|---|
| User signs in again | App access succeeds |
| Sign-in log event appears | Status success |
| Conditional Access tab | Expected policies applied |
| SAML/OIDC redirect | No protocol error |
| Token claims | Expected claims/scopes/roles present |
| API call | Succeeds |
| Provision on demand | Succeeds if provisioning was issue |
| Provisioning log | Success or expected skip |
| Audit log | Fix action recorded |

Expected result:

~~~text
Fix is confirmed by user test and administrative logs.
No unrelated access path was weakened.
~~~

## Step_29_Document_Final_State

| Field | Value |
|---|---|
| Incident / ticket | `<ticket-id>` |
| Affected app | `<enterprise-app-name>` |
| App/client ID | `<application-client-id>` |
| Service principal object ID | `<service-principal-object-id>` |
| Affected user(s) | `<affected-users>` |
| Error code | `<aadsts-error-code>` |
| Correlation ID | `<correlation-id>` |
| Timestamp | `<failure-time>` |
| Failed layer | Assignment / CA / SAML / OIDC / OAuth / Consent / Credential / Provisioning |
| Root cause | `<root-cause>` |
| Fix applied | `<fix>` |
| Validation result | Success / Failed |
| Sign-in logs reviewed | Yes / No |
| Audit logs reviewed | Yes / No |
| Provisioning logs reviewed | Yes / No / N/A |
| Rollback needed | Yes / No |
| Known exceptions | `<exceptions>` |
| Completed by | `<admin-upn>` |
| Completed date | `<yyyy-mm-dd>` |

## Verification_Commands

### Confirm Enterprise App

~~~powershell
Get-MgServicePrincipal -ServicePrincipalId $Sp.Id |
Select-Object DisplayName, Id, AppId, AccountEnabled, AppRoleAssignmentRequired
~~~

### Confirm App Registration

~~~powershell
Get-MgApplication -Filter "appId eq '$ApplicationClientId'" |
Select-Object DisplayName, Id, AppId, SignInAudience
~~~

### Confirm App Assignments

~~~powershell
Get-MgServicePrincipalAppRoleAssignedTo -ServicePrincipalId $Sp.Id -All |
Select-Object PrincipalDisplayName, PrincipalType, PrincipalId, AppRoleId
~~~

### Confirm Affected User

~~~powershell
Get-MgUser -UserId $AffectedUserUpn |
Select-Object DisplayName, UserPrincipalName, Mail, AccountEnabled, Id
~~~

### Confirm Delegated Grants

~~~powershell
Get-MgOauth2PermissionGrant -All |
Where-Object { $_.ClientId -eq $Sp.Id } |
Select-Object Id, ConsentType, Scope, PrincipalId, ResourceId
~~~

### Confirm Application Permission Grants

~~~powershell
Get-MgServicePrincipalAppRoleAssignment -ServicePrincipalId $Sp.Id -All |
Select-Object Id, PrincipalDisplayName, ResourceDisplayName, AppRoleId
~~~

### Confirm App Credentials

~~~powershell
$App = Get-MgApplication -Filter "appId eq '$ApplicationClientId'"

$App.PasswordCredentials |
Select-Object DisplayName, StartDateTime, EndDateTime, KeyId

$App.KeyCredentials |
Select-Object DisplayName, StartDateTime, EndDateTime, KeyId, Type, Usage
~~~

### Confirm Recent Sign-ins

~~~powershell
Get-MgAuditLogSignIn -Filter "userPrincipalName eq '$AffectedUserUpn'" -Top 20 |
Where-Object { $_.AppDisplayName -eq $EnterpriseAppName } |
Select-Object CreatedDateTime, UserPrincipalName, AppDisplayName, Status, ConditionalAccessStatus, CorrelationId
~~~

### Confirm Audit Events

~~~powershell
Get-MgAuditLogDirectoryAudit -Top 100 |
Where-Object {
    $_.ActivityDisplayName -match "application|service principal|permission|consent|conditional access|group|credential|provisioning"
} |
Select-Object ActivityDateTime, ActivityDisplayName, Result
~~~

## Failure_Checks

| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| User gets assignment error | User/group not assigned | Enterprise app Users and groups | Assign approved group/user |
| User assigned but still blocked | Conditional Access | Sign-in log CA tab | Remediate MFA/device or adjust policy |
| App disabled | AccountEnabled false | Enterprise app properties | Enable app if approved |
| SAML invalid audience | Identifier mismatch | SAML basic settings | Correct Entity ID |
| SAML invalid recipient | Reply URL mismatch | ACS/Reply URL | Correct Reply URL |
| SAML invalid signature | Cert mismatch | SAML certificate/vendor cert | Upload active cert |
| Wrong user in app | NameID mismatch | Attributes & Claims | Correct NameID |
| OIDC redirect error | Redirect URI mismatch | App registration Authentication | Add exact URI |
| Token missing scope | Delegated permission missing or not requested | API permissions/token `scp` | Add/request/consent scope |
| Token missing role | Application permission not granted | App role assignments/token `roles` | Grant app permission |
| Admin consent unavailable | Wrong admin role | Admin role assignment | Use proper role |
| User consent blocked | Tenant consent policy | Consent settings | Use admin consent workflow |
| Secret auth fails | Secret expired/wrong | Certificates & secrets | Rotate secret |
| Certificate auth fails | Cert mismatch/expired | Key credentials/private key | Rotate cert |
| SCIM test fails | Bad URL/token | Provisioning Admin Credentials | Correct SCIM settings |
| User not provisioned | Out of scope | Assignments/provisioning scope | Assign user/group |
| User provisioning skipped | Missing required attribute | Provisioning logs | Fix source attribute or mapping |
| Mass disable risk | Scope changed | Provisioning settings/logs | Stop provisioning and review |

## Rollback

Rollback depends on the fix applied. Use the smallest safe rollback.

### Disable Enterprise Application Sign-In

Use only for emergency containment.

~~~powershell
Update-MgServicePrincipal -ServicePrincipalId $Sp.Id -AccountEnabled:$false

Get-MgServicePrincipal -ServicePrincipalId $Sp.Id |
Select-Object DisplayName, AccountEnabled
~~~

Expected result:

~~~text
Enterprise Application is disabled for user sign-in.
Users cannot access the app through Entra.
~~~

### Revert Assignment Change

Remove a user or group assignment only after confirming it should not have access.

~~~powershell
$Assignments = Get-MgServicePrincipalAppRoleAssignedTo -ServicePrincipalId $Sp.Id -All

$Assignments |
Select-Object Id, PrincipalDisplayName, PrincipalType, PrincipalId, AppRoleId
~~~

Remove selected assignment:

~~~powershell
$AssignmentIdToRemove = "<assignment-id>"

Remove-MgServicePrincipalAppRoleAssignedTo `
  -ServicePrincipalId $Sp.Id `
  -AppRoleAssignmentId $AssignmentIdToRemove
~~~

Expected result:

~~~text
Selected assignment is removed.
Access through that assignment path is revoked.
~~~

### Disable Conditional Access Policy

Use when CA policy is causing outage and rollback is approved.

~~~powershell
$Policy = Get-MgIdentityConditionalAccessPolicy |
Where-Object { $_.DisplayName -eq $ConditionalAccessPolicyName }

Update-MgIdentityConditionalAccessPolicy `
  -ConditionalAccessPolicyId $Policy.Id `
  -State "disabled"
~~~

Expected result:

~~~text
Conditional Access policy no longer applies.
User sign-in should be retested immediately.
~~~

### Return Conditional Access Policy To Report-Only

Preferred over deletion when troubleshooting.

~~~powershell
Update-MgIdentityConditionalAccessPolicy `
  -ConditionalAccessPolicyId $Policy.Id `
  -State "enabledForReportingButNotEnforced"
~~~

Expected result:

~~~text
Policy no longer enforces but continues to report impact.
~~~

### Stop Provisioning

Use if provisioning is causing incorrect creates, updates, disables, or deletes.

~~~text
Portal path:
Enterprise applications > <enterprise-app-name> > Provisioning
Set Provisioning Status to Off
Save
~~~

Expected result:

~~~text
Provisioning stops.
No further SCIM changes are sent.
~~~

### Remove Unauthorized Delegated Grant

~~~powershell
$GrantIdToRemove = "<oauth2-permission-grant-id>"

Remove-MgOauth2PermissionGrant -OAuth2PermissionGrantId $GrantIdToRemove
~~~

Expected result:

~~~text
Delegated permission grant is removed.
App may prompt again or fail until correct consent is granted.
~~~

### Remove Unauthorized Application Permission Grant

~~~powershell
$AppRoleAssignmentIdToRemove = "<app-role-assignment-id>"

Remove-MgServicePrincipalAppRoleAssignment `
  -ServicePrincipalId $Sp.Id `
  -AppRoleAssignmentId $AppRoleAssignmentIdToRemove
~~~

Expected result:

~~~text
Application permission grant is removed.
App-only API calls depending on that permission fail.
~~~

### Remove Bad Redirect URI

~~~powershell
$App = Get-MgApplication -Filter "appId eq '$ApplicationClientId'"

$BadRedirectUri = "<bad-redirect-uri>"
$CurrentWeb = $App.Web
$CurrentWeb.RedirectUris = @($CurrentWeb.RedirectUris | Where-Object { $_ -ne $BadRedirectUri })

Update-MgApplication `
  -ApplicationId $App.Id `
  -Web $CurrentWeb
~~~

Expected result:

~~~text
Incorrect redirect URI is removed from the app registration.
~~~

## Security_Baseline

| Control | Baseline |
|---|---|
| Troubleshooting method | Start with logs and correlation ID |
| Assignment required | Validate before changing |
| Conditional Access | Prefer report-only rollback over deletion |
| SAML | Validate URL, claim, and certificate before changing vendor config |
| OIDC | Redirect URI must match exactly |
| OAuth delegated permissions | Confirm `scp` claim |
| OAuth application permissions | Confirm `roles` claim |
| Consent | Separate declared permissions from granted permissions |
| Credentials | Rotate expired secrets/certs securely |
| Provisioning | Stop provisioning before risky mapping/scope changes |
| Audit logs | Review recent changes before applying fix |
| Documentation | Record root cause and exact fix |
| Emergency rollback | Preserve break-glass and vendor local admin access |

## Admin_Notes

~~~text
Ticket:
Affected app:
Enterprise app object ID:
Application/client ID:
App registration object ID:
Affected user:
Known-good test user:
Error code:
Correlation ID:
Request ID:
Timestamp:
Failure layer:
Sign-in log result:
Conditional Access result:
Assignment state:
SAML check result:
OIDC redirect check result:
OAuth delegated grant result:
OAuth app permission result:
Consent state:
Credential state:
Provisioning status:
Provisioning log result:
Root cause:
Fix applied:
Rollback needed:
Validation result:
Known exceptions:
Completed by:
Completed date:
~~~

## Related_Labs

| Workbook                                                                             | Relationship                                                              |
| ------------------------------------------------------------------------------------ | ------------------------------------------------------------------------- |
| 01_Configure_Enterprise_Applications_Service_Principals_And_Assignments.md           | Covers app assignment, owners, and assignment required                    |
| 02_Configure_App_Registrations_API_Permissions_Secrets_Certificates_And_Owners.md    | Covers app registration, credentials, and API permission declarations     |
| 03_Configure_Admin_Consent_User_Consent_And_Publisher_Verification_Baseline.md       | Covers consent governance and publisher verification                      |
| 04_Configure_SAML_OIDC_OAuth_SSO_And_Application_Provisioning.md                     | Covers SSO and provisioning configuration                                 |
| 05_Configure_Conditional_Access_App_Targeting_Session_Controls_And_Access_Reviews.md | Covers Conditional Access targeting, session controls, and access reviews |
