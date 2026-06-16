# 04_Configure_SAML_OIDC_OAuth_SSO_And_Application_Provisioning

## Objective

Configure Microsoft Entra application single sign-on and provisioning baselines for enterprise applications.

This workbook covers:

- SAML-based single sign-on
- OpenID Connect sign-in baseline
- OAuth authorization baseline
- Enterprise Application SSO settings
- Claims and attributes
- Certificates and metadata
- SCIM provisioning
- Provisioning scope
- Provisioning logs
- Sign-in validation
- Failure checks and rollback

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
| SAML identifier/entity ID | `<saml-entity-id>` |
| SAML reply URL/ACS URL | `<saml-reply-url>` |
| SAML sign-on URL | `<saml-sign-on-url>` |
| Logout URL | `<logout-url>` |
| OIDC redirect URI | `<oidc-redirect-uri>` |
| SCIM tenant URL | `<scim-tenant-url>` |
| SCIM secret token | `<scim-secret-token>` |
| Test user | `<test-user-upn>` |
| Access group | `<app-access-group-name>` |
| PowerShell module | Microsoft Graph PowerShell |
| Graph permission scope | `Application.ReadWrite.All`, `Directory.ReadWrite.All`, `AppRoleAssignment.ReadWrite.All`, `AuditLog.Read.All` |

## Mental_Model

Microsoft Entra can provide authentication, authorization, SSO, and provisioning for applications.

The application type determines the control plane.

| App Pattern | Main Admin Area | Protocol | Common Use |
|---|---|---|---|
| Gallery SaaS app | Enterprise applications | SAML or OIDC | Vendor SaaS SSO |
| Custom SAML app | Enterprise applications | SAML 2.0 | Legacy or vendor SSO |
| Custom OIDC app | App registrations | OIDC / OAuth 2.0 | Modern web app sign-in |
| API app | App registrations | OAuth 2.0 | API access tokens |
| Provisioned SaaS app | Enterprise applications | SCIM | User/group lifecycle sync |

Basic relationship:

~~~text
User
    |
    | signs in
    v
Microsoft Entra ID
    |
    | issues token/assertion
    v
Application
    |
    | optionally receives users/groups through provisioning
    v
Application account and access state
~~~

SAML issues assertions.

OIDC issues ID tokens.

OAuth issues access tokens.

SCIM provisions users and groups.

Do not treat these as the same thing.

## Protocol_Decision_Table

| Protocol | Used For | Token / Message | Admin Surface |
|---|---|---|---|
| SAML 2.0 | Enterprise SSO | SAML assertion | Enterprise applications > Single sign-on |
| OIDC | User authentication | ID token | App registrations > Authentication |
| OAuth 2.0 | API authorization | Access token | App registrations > API permissions / Expose an API |
| SCIM | Provisioning | REST provisioning calls | Enterprise applications > Provisioning |

## Planning_Table

| Decision Area | Recommended Baseline | Notes |
|---|---|---|
| SSO protocol | Use vendor-supported protocol | Prefer OIDC for modern apps, SAML for many SaaS apps |
| Assignment required | Enabled for controlled apps | Limits access to assigned users/groups |
| Assignment method | Group-based | Easier lifecycle management |
| SAML certificate | Track expiration | Certificate expiration breaks SSO |
| SAML claims | Minimal required claims only | Do not send unnecessary attributes |
| NameID | Match vendor requirement | Usually UPN or email |
| OIDC redirect URI | Exact URI only | No stale or broad redirect URIs |
| OAuth scopes | Least privilege | Avoid broad API permissions |
| Provisioning scope | Assigned users/groups only | Prevents unintended account creation |
| SCIM secret | Stored securely | Rotate if exposed |
| Provisioning mappings | Review before enablement | Wrong mapping can overwrite app attributes |
| Logs | Review sign-in and provisioning logs | Validate behavior before production rollout |
| Rollback | Disable provisioning before deleting mappings | Prevent destructive sync behavior |

## Prerequisites

| Requirement | Details |
|---|---|
| Role | Cloud Application Administrator, Application Administrator, Global Administrator, or Hybrid Identity Administrator depending action |
| App exists | Enterprise Application or App Registration exists |
| Vendor SSO details | Identifier, reply URL, sign-on URL, logout URL |
| Vendor metadata | Metadata URL or XML if available |
| Provisioning details | SCIM tenant URL and token if provisioning is used |
| Test user | Non-admin user assigned to the app |
| Access group | Group used for assignment and provisioning scope |
| Certificate tracking | Expiration date documented |
| Change window | Required for production SSO/provisioning cutover |
| Backout plan | Existing login method and local admin access confirmed |

## Variables

Replace placeholders before running commands.

~~~powershell
$TenantId = "<tenant-id>"
$EnterpriseAppName = "<enterprise-app-name>"
$AppRegistrationName = "<app-registration-name>"
$ApplicationClientId = "<application-client-id>"
$ServicePrincipalObjectId = "<service-principal-object-id>"
$TestUserUpn = "<test-user-upn>"
$AccessGroupName = "<app-access-group-name>"
$SamlEntityId = "<saml-entity-id>"
$SamlReplyUrl = "<saml-reply-url>"
$SamlSignOnUrl = "<saml-sign-on-url>"
$LogoutUrl = "<logout-url>"
$OidcRedirectUri = "<oidc-redirect-uri>"
$ScimTenantUrl = "<scim-tenant-url>"
~~~

## Configuration_Checklist

| Step | Task | Admin Center / Portal Path | PowerShell / CLI / Graph | Expected Result |
|---:|---|---|---|---|
| 1 | Sign in to Entra admin center | `https://entra.microsoft.com` | N/A | Admin portal loads |
| 2 | Connect to Microsoft Graph | N/A | `Connect-MgGraph` | Required scopes are active |
| 3 | Locate enterprise app | Applications > Enterprise applications | `Get-MgServicePrincipal` | Service principal is found |
| 4 | Confirm assignment required | Enterprise app > Properties | `AppRoleAssignmentRequired` | Controlled access is confirmed |
| 5 | Confirm user/group assignment | Enterprise app > Users and groups | App role assignments | Test user/group is assigned |
| 6 | Select SSO method | Enterprise app > Single sign-on | N/A | SAML/OIDC/password/linked option is known |
| 7 | Configure SAML basic settings | Enterprise app > Single sign-on > SAML | Portal | Identifier, reply URL, sign-on URL configured |
| 8 | Configure SAML attributes and claims | SAML > Attributes & Claims | Portal | Required claims are emitted |
| 9 | Download SAML certificate/metadata | SAML > Signing Certificate | Portal | Metadata/certificate available for vendor |
| 10 | Configure vendor with Entra metadata | Vendor admin portal | N/A | Vendor trusts Entra |
| 11 | Configure OIDC redirect URI if custom app | App registrations > Authentication | `Update-MgApplication` | Redirect URI is configured |
| 12 | Configure OAuth permissions if API access required | App registrations > API permissions | `RequiredResourceAccess` | Least privilege permissions configured |
| 13 | Configure provisioning mode | Enterprise app > Provisioning | Portal | Provisioning mode set to Automatic if used |
| 14 | Enter SCIM endpoint and token | Provisioning > Admin Credentials | Portal | SCIM test connection succeeds |
| 15 | Configure provisioning scope | Provisioning > Settings | Portal | Scope limited to assigned users/groups |
| 16 | Review attribute mappings | Provisioning > Mappings | Portal | Mappings match app requirements |
| 17 | Start provisioning | Provisioning > Start provisioning | Portal | Provisioning cycle begins |
| 18 | Validate assigned user SSO | Browser test | N/A | User signs in successfully |
| 19 | Validate provisioning result | Provisioning logs | Graph / portal | User is created or updated in app |
| 20 | Review logs | Sign-in logs and provisioning logs | `Get-MgAuditLogSignIn` | Success/failure visible |
| 21 | Document final state | Workbook notes / repo | N/A | SSO/provisioning baseline documented |

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

## Step_02_Locate_Enterprise_Application

### Portal

| Step | Action |
|---:|---|
| 1 | Go to `https://entra.microsoft.com` |
| 2 | Open Applications |
| 3 | Select Enterprise applications |
| 4 | Search for `<enterprise-app-name>` |
| 5 | Open the app |

### PowerShell

~~~powershell
$Sp = Get-MgServicePrincipal -Filter "displayName eq '$EnterpriseAppName'"

$Sp | Select-Object DisplayName, Id, AppId, AccountEnabled, AppRoleAssignmentRequired
~~~

Expected result:

~~~text
The target Enterprise Application service principal is found.
AccountEnabled is True.
Assignment required is known.
~~~

## Step_03_Confirm_Assignment_Baseline

SSO should normally be scoped to assigned users/groups.

### Portal

| Step | Action |
|---:|---|
| 1 | Open Enterprise applications |
| 2 | Select `<enterprise-app-name>` |
| 3 | Select Properties |
| 4 | Confirm Assignment required is Yes |
| 5 | Select Users and groups |
| 6 | Confirm `<app-access-group-name>` is assigned |

### PowerShell

~~~powershell
Get-MgServicePrincipal -ServicePrincipalId $Sp.Id |
Select-Object DisplayName, AppRoleAssignmentRequired

Get-MgServicePrincipalAppRoleAssignedTo -ServicePrincipalId $Sp.Id -All |
Select-Object PrincipalDisplayName, PrincipalType, AppRoleId, ResourceDisplayName
~~~

Expected result:

~~~text
Assignment required is enabled for controlled applications.
The access group is assigned to the app.
~~~

## Step_04_Select_SSO_Method

### Portal

| Step | Action |
|---:|---|
| 1 | Open Enterprise applications |
| 2 | Select `<enterprise-app-name>` |
| 3 | Select Single sign-on |
| 4 | Choose the correct SSO method |

### SSO Method Selection

| Method | Use When |
|---|---|
| SAML | Vendor requires SAML 2.0 |
| OpenID Connect | App uses modern OIDC sign-in through app registration |
| Password-based | Legacy app needs password vaulting |
| Linked | App is simply linked from My Apps |
| Disabled | App is not ready for SSO |

Expected result:

~~~text
The selected SSO method matches the application design and vendor requirement.
~~~

## Step_05_Configure_SAML_Basic_Settings

Use this section for SAML applications.

### Vendor Values Needed

| Value | Description |
|---|---|
| Identifier / Entity ID | Unique app identifier expected by vendor |
| Reply URL / ACS URL | Vendor endpoint that receives SAML response |
| Sign-on URL | Vendor login URL |
| Logout URL | Optional vendor logout endpoint |
| Relay State | Optional app-specific landing state |

### Portal

| Step | Action |
|---:|---|
| 1 | Open Enterprise applications |
| 2 | Select `<enterprise-app-name>` |
| 3 | Select Single sign-on |
| 4 | Select SAML |
| 5 | Edit Basic SAML Configuration |
| 6 | Set Identifier / Entity ID |
| 7 | Set Reply URL / ACS URL |
| 8 | Set Sign-on URL |
| 9 | Set Logout URL if required |
| 10 | Set Relay State if required |
| 11 | Save |

Expected result:

~~~text
Basic SAML Configuration contains the exact vendor-required values.
Reply URL matches the vendor ACS URL exactly.
Identifier matches the vendor entity ID exactly.
~~~

## Step_06_Configure_SAML_User_Attributes_And_Claims

Claims tell the application who the user is.

### Common SAML Claims

| Claim | Common Value |
|---|---|
| Unique User Identifier / NameID | `user.userprincipalname` or `user.mail` |
| givenname | `user.givenname` |
| surname | `user.surname` |
| emailaddress | `user.mail` |
| name | `user.userprincipalname` |
| groups | Security group IDs or names if required |

### Portal

| Step | Action |
|---:|---|
| 1 | Open Enterprise app |
| 2 | Select Single sign-on |
| 3 | Select SAML |
| 4 | Edit Attributes & Claims |
| 5 | Confirm Unique User Identifier |
| 6 | Add required claims |
| 7 | Remove unnecessary custom claims |
| 8 | Save |

### Claim Baseline

| Requirement | Baseline |
|---|---|
| NameID | Match vendor requirement |
| Email claim | Send only if app needs it |
| Group claim | Avoid unless required |
| Custom claims | Keep minimal |
| Sensitive attributes | Do not send unless required |
| Immutable identifier | Use stable value when vendor requires durable mapping |

Expected result:

~~~text
SAML assertion contains only required claims.
NameID format and value match vendor expectations.
~~~

## Step_07_Configure_SAML_Group_Claims_If_Required

Only configure group claims if the app requires them for authorization.

Group claims can cause token bloat and overexposure of directory structure.

### Portal

| Step | Action |
|---:|---|
| 1 | Open Attributes & Claims |
| 2 | Select Add a group claim |
| 3 | Choose group type |
| 4 | Prefer groups assigned to the application if available |
| 5 | Choose source attribute |
| 6 | Save |

### Group Claim Options

| Option | Baseline View |
|---|---|
| All groups | Avoid unless required |
| Security groups | Use only if needed |
| Groups assigned to the application | Preferred when supported |
| Group ID | Stable but less readable |
| Group display name | Readable but can change |
| sAMAccountName | Hybrid-specific and depends on sync |

Expected result:

~~~text
Group claims are emitted only when required.
Scope is limited to app-assigned groups where possible.
~~~

## Step_08_Download_Or_Copy_SAML_Metadata

Entra provides the SAML values the vendor needs.

### Portal

| Step | Action |
|---:|---|
| 1 | Open Enterprise app |
| 2 | Select Single sign-on |
| 3 | Select SAML |
| 4 | Go to SAML Certificates |
| 5 | Download Federation Metadata XML |
| 6 | Download Certificate Base64 if required |
| 7 | Copy Login URL |
| 8 | Copy Microsoft Entra Identifier |
| 9 | Copy Logout URL if used |

### Vendor Configuration Values

| Entra Value | Vendor Field Usually Called |
|---|---|
| Login URL | IdP SSO URL |
| Microsoft Entra Identifier | IdP Entity ID / Issuer |
| Certificate Base64 | IdP signing certificate |
| Federation Metadata XML | IdP metadata |
| Logout URL | IdP logout URL |

Expected result:

~~~text
Vendor has the correct IdP metadata, login URL, issuer, and signing certificate.
~~~

## Step_09_Track_SAML_Certificate_Expiration

SAML signing certificate expiration can break SSO.

### Portal

| Step | Action |
|---:|---|
| 1 | Open Enterprise app |
| 2 | Select Single sign-on |
| 3 | Select SAML |
| 4 | Review SAML Certificates |
| 5 | Record active certificate expiration date |
| 6 | Create rotation reminder before expiration |

### Documentation

| Field | Value |
|---|---|
| Certificate status | Active |
| Thumbprint | `<thumbprint>` |
| Expiration | `<yyyy-mm-dd>` |
| Vendor updated | Yes / No |
| Rotation owner | `<owner-upn>` |
| Reminder date | `<yyyy-mm-dd>` |

Expected result:

~~~text
SAML signing certificate expiration is documented.
Rotation owner is known.
Vendor update process is known.
~~~

## Step_10_Configure_Vendor_SAML_Settings

This step occurs in the vendor admin portal, not Entra.

| Vendor Field | Entra Source |
|---|---|
| IdP Entity ID | Microsoft Entra Identifier |
| SSO URL | Login URL |
| SLO URL | Logout URL if used |
| Signing certificate | Certificate Base64 |
| Metadata XML | Federation Metadata XML |
| NameID format | Attributes & Claims |
| ACS URL | Vendor provides this to Entra |
| SP Entity ID | Vendor provides this to Entra |

Expected result:

~~~text
Vendor trusts Microsoft Entra as the identity provider.
Vendor SAML settings match Entra SAML settings.
~~~

## Step_11_Test_SAML_SSO

### Portal Test

| Step | Action | Expected Result |
|---:|---|---|
| 1 | Open Enterprise app | App blade opens |
| 2 | Select Single sign-on | SSO page opens |
| 3 | Select Test this application | Test panel opens |
| 4 | Sign in as `<test-user-upn>` | Authentication succeeds |
| 5 | Confirm vendor app opens | User lands in app |

### Browser Test

| Step | Action | Expected Result |
|---:|---|---|
| 1 | Open private browser window | Clean session |
| 2 | Browse to My Apps or vendor app URL | Sign-in begins |
| 3 | Sign in as `<test-user-upn>` | Entra authenticates user |
| 4 | Vendor receives SAML assertion | App grants access |
| 5 | Confirm correct user profile | User identity maps correctly |

Expected result:

~~~text
Assigned test user can sign in using SAML SSO.
The app receives correct NameID and required claims.
~~~

## Step_12_Configure_OIDC_Redirect_URI_For_Custom_App

Use this section for custom OIDC applications.

### Portal

| Step | Action |
|---:|---|
| 1 | Open App registrations |
| 2 | Select `<app-registration-name>` |
| 3 | Select Authentication |
| 4 | Select Add a platform |
| 5 | Choose Web or SPA as appropriate |
| 6 | Add exact redirect URI |
| 7 | Configure logout URL if needed |
| 8 | Save |

### PowerShell

~~~powershell
$App = Get-MgApplication -Filter "displayName eq '$AppRegistrationName'"

$WebConfig = @{
    RedirectUris = @($OidcRedirectUri)
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
Select-Object DisplayName, AppId, Web
~~~

Expected result:

~~~text
OIDC redirect URI is configured exactly.
Implicit grant remains disabled unless legacy app design requires it.
~~~

## Step_13_Validate_OIDC_Metadata

OIDC apps use tenant metadata endpoints.

### Tenant Metadata Pattern

~~~text
https://login.microsoftonline.com/<tenant-id>/v2.0/.well-known/openid-configuration
~~~

### Values To Confirm

| Metadata Item | Purpose |
|---|---|
| authorization_endpoint | Where the app sends users for sign-in |
| token_endpoint | Where the app redeems authorization code |
| jwks_uri | Where the app gets signing keys |
| issuer | Token issuer expected by app |
| end_session_endpoint | Logout endpoint |

Expected result:

~~~text
Application is configured to use the correct tenant metadata endpoint.
Issuer and redirect URI match app configuration.
~~~

## Step_14_Configure_OAuth_API_Permissions

Use OAuth access tokens when the app calls APIs.

### Portal

| Step | Action |
|---:|---|
| 1 | Open App registrations |
| 2 | Select `<app-registration-name>` |
| 3 | Select API permissions |
| 4 | Add required delegated or application permissions |
| 5 | Use least privilege |
| 6 | Grant admin consent only after approval |

### Permission Review

| Permission Type | Use Case | Risk |
|---|---|---|
| Delegated | App acts as signed-in user | Depends on user and scope |
| Application | App acts as itself | Higher risk |
| `openid` | Sign-in | Low |
| `profile` | Basic profile | Low |
| `email` | Email claim | Low |
| `offline_access` | Refresh token | Medium |
| Graph read/write broad permissions | Directory, mail, files, sites | High/Critical |

Expected result:

~~~text
OAuth permissions are configured with least privilege.
Admin consent exists only for approved permissions.
~~~

## Step_15_Test_OIDC_And_OAuth_Sign_In

### Test Flow

| Step | Action | Expected Result |
|---:|---|---|
| 1 | Open private browser window | Clean session |
| 2 | Browse to custom app | App redirects to Entra |
| 3 | Sign in as `<test-user-upn>` | Authentication succeeds |
| 4 | Entra redirects to configured redirect URI | No reply URL mismatch |
| 5 | App validates ID token | User session starts |
| 6 | App requests access token if needed | Access token contains expected scopes/roles |
| 7 | App calls API | API call succeeds if permission is granted |

### Token Validation

| Claim | Meaning |
|---|---|
| `aud` | Audience/client/API expected by token |
| `iss` | Issuer tenant |
| `tid` | Tenant ID |
| `oid` | User or service principal object ID |
| `upn` / `preferred_username` | User sign-in name |
| `scp` | Delegated scopes |
| `roles` | Application permissions or app roles |
| `exp` | Expiration time |

Expected result:

~~~text
OIDC sign-in succeeds.
OAuth access token contains expected permissions.
No redirect URI or consent errors occur.
~~~

## Step_16_Configure_Provisioning_Mode

Use this section for SaaS apps that support automatic provisioning, usually through SCIM.

### Portal

| Step | Action |
|---:|---|
| 1 | Open Enterprise applications |
| 2 | Select `<enterprise-app-name>` |
| 3 | Select Provisioning |
| 4 | Select Get started |
| 5 | Set Provisioning Mode to Automatic |
| 6 | Enter Tenant URL |
| 7 | Enter Secret Token |
| 8 | Select Test Connection |
| 9 | Save |

Expected result:

~~~text
SCIM test connection succeeds.
Entra can connect to the application's provisioning endpoint.
~~~

## Step_17_Configure_Provisioning_Scope

Provisioning should usually be scoped to assigned users and groups.

### Portal

| Step | Action |
|---:|---|
| 1 | Open Enterprise app |
| 2 | Select Provisioning |
| 3 | Select Settings |
| 4 | Set Scope to Sync only assigned users and groups |
| 5 | Confirm assignment group is assigned to the app |
| 6 | Save |

### Scope Decision

| Scope | Use Case | Risk |
|---|---|---|
| Sync only assigned users and groups | Recommended baseline | Controlled |
| Sync all users and groups | Rare tenant-wide apps | High risk |
| Disabled | Provisioning not used | No provisioning |

Expected result:

~~~text
Provisioning scope is limited to assigned users and groups.
Unassigned users are not provisioned to the app.
~~~

## Step_18_Review_Provisioning_Attribute_Mappings

Attribute mappings determine what Entra sends to the application.

### Portal

| Step | Action |
|---:|---|
| 1 | Open Enterprise app |
| 2 | Select Provisioning |
| 3 | Select Mappings |
| 4 | Open user mapping |
| 5 | Review source attributes |
| 6 | Review target attributes |
| 7 | Confirm matching precedence |
| 8 | Disable unnecessary attributes |
| 9 | Save |

### Common SCIM Mappings

| Entra Attribute | SCIM Attribute |
|---|---|
| `userPrincipalName` | `userName` |
| `mail` | `emails[type eq "work"].value` |
| `givenName` | `name.givenName` |
| `surname` | `name.familyName` |
| `displayName` | `displayName` |
| `jobTitle` | `title` |
| `department` | `urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:department` |
| `accountEnabled` | `active` |

Expected result:

~~~text
Mappings match vendor requirements.
Matching attribute is stable and unique.
Unnecessary attributes are not synchronized.
~~~

## Step_19_Configure_Provisioning_Settings

### Portal

| Step | Action |
|---:|---|
| 1 | Open Enterprise app |
| 2 | Select Provisioning |
| 3 | Select Settings |
| 4 | Enter notification email |
| 5 | Configure accidental deletion prevention threshold |
| 6 | Confirm scope |
| 7 | Save |

### Recommended Settings

| Setting | Baseline |
|---|---|
| Notification email | App owner or operations mailbox |
| Accidental deletion prevention | Enabled |
| Deletion threshold | Conservative default unless justified |
| Scope | Assigned users and groups |
| Provisioning status | On only after validation |

Expected result:

~~~text
Provisioning settings protect against accidental mass deletion or disablement.
Operational notifications go to an accountable owner.
~~~

## Step_20_Start_Provisioning

### Portal

| Step | Action |
|---:|---|
| 1 | Open Enterprise app |
| 2 | Select Provisioning |
| 3 | Confirm Admin Credentials test succeeds |
| 4 | Confirm mappings are reviewed |
| 5 | Confirm scope is assigned users/groups |
| 6 | Set Provisioning Status to On |
| 7 | Save |
| 8 | Select Start provisioning if available |

Expected result:

~~~text
Provisioning begins.
Initial cycle starts or is queued.
Only assigned users/groups are in scope.
~~~

## Step_21_Run_Provision_On_Demand

Provision on demand is useful for testing one user before broad rollout.

### Portal

| Step | Action |
|---:|---|
| 1 | Open Enterprise app |
| 2 | Select Provisioning |
| 3 | Select Provision on demand |
| 4 | Search for `<test-user-upn>` |
| 5 | Select user |
| 6 | Run provisioning |
| 7 | Review result |

Expected result:

~~~text
Test user provisions successfully.
Attribute mappings and matching behavior are validated before broad rollout.
~~~

## Step_22_Review_Provisioning_Logs

### Portal

| Step | Action |
|---:|---|
| 1 | Open Enterprise app |
| 2 | Select Provisioning logs |
| 3 | Filter by `<test-user-upn>` |
| 4 | Review action |
| 5 | Review source and target system |
| 6 | Review status |
| 7 | Review modified attributes |
| 8 | Review error details if failed |

Expected result:

~~~text
Provisioning log shows create, update, disable, or skip action.
Failures include actionable error details.
~~~

## Step_23_Review_Sign_In_Logs

### Portal

| Step | Action |
|---:|---|
| 1 | Open Enterprise app |
| 2 | Select Sign-in logs |
| 3 | Filter by `<test-user-upn>` |
| 4 | Review Status |
| 5 | Review Authentication Details |
| 6 | Review Conditional Access tab |
| 7 | Review Failure Reason if applicable |

### PowerShell

~~~powershell
Get-MgAuditLogSignIn -Filter "appDisplayName eq '$EnterpriseAppName'" -Top 20 |
Select-Object CreatedDateTime, UserPrincipalName, AppDisplayName, Status, ConditionalAccessStatus
~~~

Expected result:

~~~text
SSO attempts are visible.
Failures identify SAML, OIDC, assignment, Conditional Access, or provisioning-related issues.
~~~

## Step_24_Document_Final_State

| Field | Value |
|---|---|
| Enterprise app | `<enterprise-app-name>` |
| App registration | `<app-registration-name>` |
| Application/client ID | `<application-client-id>` |
| Service principal object ID | `<service-principal-object-id>` |
| SSO protocol | SAML / OIDC / OAuth / Linked / Password |
| Assignment required | Yes / No |
| Assigned group | `<app-access-group-name>` |
| SAML Entity ID | `<saml-entity-id>` |
| SAML Reply URL | `<saml-reply-url>` |
| SAML Sign-on URL | `<saml-sign-on-url>` |
| SAML certificate expiration | `<yyyy-mm-dd>` |
| NameID value | `<claim-value>` |
| Required claims | `<claim-list>` |
| OIDC Redirect URI | `<oidc-redirect-uri>` |
| OAuth permissions | `<permission-list>` |
| Provisioning mode | Manual / Automatic / Disabled |
| Provisioning scope | Assigned users/groups / All users/groups |
| SCIM test connection | Success / Failed / N/A |
| Provision on demand result | Success / Failed / N/A |
| Test user SSO result | Success / Failed |
| Sign-in logs reviewed | Yes / No |
| Provisioning logs reviewed | Yes / No |
| Validation date | `<yyyy-mm-dd>` |
| Validated by | `<admin-upn>` |

## Verification_Commands

### Confirm Enterprise App

~~~powershell
Get-MgServicePrincipal -ServicePrincipalId $Sp.Id |
Select-Object DisplayName, Id, AppId, AccountEnabled, AppRoleAssignmentRequired
~~~

### Confirm App Assignments

~~~powershell
Get-MgServicePrincipalAppRoleAssignedTo -ServicePrincipalId $Sp.Id -All |
Select-Object PrincipalDisplayName, PrincipalType, AppRoleId, ResourceDisplayName
~~~

### Confirm App Registration

~~~powershell
$App = Get-MgApplication -Filter "displayName eq '$AppRegistrationName'"

$App | Select-Object DisplayName, Id, AppId, SignInAudience
~~~

### Confirm Redirect URIs

~~~powershell
$App.Web
$App.Spa
$App.PublicClient
~~~

### Confirm API Permissions

~~~powershell
$App.RequiredResourceAccess
~~~

### Confirm Recent Sign-ins

~~~powershell
Get-MgAuditLogSignIn -Filter "appDisplayName eq '$EnterpriseAppName'" -Top 20 |
Select-Object CreatedDateTime, UserPrincipalName, AppDisplayName, Status, ConditionalAccessStatus
~~~

### Confirm Directory Audit Events

~~~powershell
Get-MgAuditLogDirectoryAudit -Top 50 |
Where-Object {
    $_.ActivityDisplayName -match "application|service principal|provisioning|single sign-on|claims|credential"
} |
Select-Object ActivityDateTime, ActivityDisplayName, Result
~~~

## Failure_Checks

| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| SAML login fails | Wrong reply URL / ACS URL | SAML basic settings and vendor ACS URL | Correct reply URL exactly |
| SAML issuer mismatch | Wrong entity ID | Identifier / Entity ID | Match vendor requirement |
| User lands in wrong account | Wrong NameID | Attributes & Claims | Change NameID to vendor-required attribute |
| Vendor rejects assertion | Certificate mismatch | Vendor IdP cert and Entra active cert | Upload correct certificate to vendor |
| SSO works for admin but not user | User not assigned | Enterprise app Users and groups | Assign user/group |
| User blocked before app | Conditional Access | Sign-in logs CA tab | Adjust CA policy if approved |
| App not visible in My Apps | App hidden or user not assigned | Enterprise app properties and assignment | Set visible or assign user |
| OIDC reply URL mismatch | Redirect URI mismatch | App registration Authentication | Add exact redirect URI |
| OIDC token rejected | Wrong issuer or audience | Decode ID token | Correct tenant/client configuration |
| OAuth API call fails | Missing permission or consent | API permissions and token claims | Add permission and grant approved consent |
| Access token missing scope | Scope not requested | Auth request | Request correct scope |
| Access token missing roles | App permission not granted | App role assignment | Grant approved application permission |
| SCIM test connection fails | Bad tenant URL or token | Provisioning Admin Credentials | Correct URL/token |
| User not provisioned | User not in scope | Assignment and provisioning scope | Assign user/group |
| Provisioning skipped user | Missing required attribute | Provisioning logs | Fix attribute mapping/source value |
| Provisioning updates wrong user | Bad matching attribute | Mappings matching precedence | Use stable unique matching attribute |
| Provisioning disables too many users | Scope or deletion threshold issue | Provisioning settings | Stop provisioning and review scope |
| Group provisioning fails | App does not support groups | Provisioning logs/vendor docs | Disable group provisioning or adjust app config |

## Rollback

Rollback must preserve emergency access. Confirm vendor local admin access before changing production SSO.

### Disable Provisioning

~~~text
Portal path:
Enterprise applications > <enterprise-app-name> > Provisioning
Set Provisioning Status to Off
Save
~~~

Expected result:

~~~text
Provisioning stops.
No further SCIM create, update, or disable actions are sent.
~~~

### Remove SCIM Credentials

~~~text
Portal path:
Enterprise applications > <enterprise-app-name> > Provisioning > Admin Credentials
Clear or replace Tenant URL and Secret Token if required
Save
~~~

Expected result:

~~~text
Entra can no longer provision to the SCIM endpoint using the old credential.
~~~

### Revert Provisioning Scope

~~~text
Portal path:
Enterprise applications > <enterprise-app-name> > Provisioning > Settings
Set Scope back to previous approved value
Save
~~~

Expected result:

~~~text
Provisioning scope returns to the previous approved state.
~~~

### Disable SAML SSO

Use only if returning to the prior login method.

~~~text
Portal path:
Enterprise applications > <enterprise-app-name> > Single sign-on
Change or clear SSO configuration only after confirming vendor fallback login
~~~

Expected result:

~~~text
Application no longer relies on the broken SAML configuration.
Users must use the approved fallback login method.
~~~

### Remove OIDC Redirect URI

~~~powershell
$App = Get-MgApplication -Filter "displayName eq '$AppRegistrationName'"

$CurrentWeb = $App.Web
$CurrentWeb.RedirectUris = @($CurrentWeb.RedirectUris | Where-Object { $_ -ne $OidcRedirectUri })

Update-MgApplication `
  -ApplicationId $App.Id `
  -Web $CurrentWeb
~~~

Expected result:

~~~text
Specified OIDC redirect URI is removed.
Any app flow using that URI will fail until restored.
~~~

### Disable App For User Sign-In

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

## Security_Baseline

| Control | Baseline |
|---|---|
| Assignment required | Enabled for controlled apps |
| Assignment method | Group-based |
| SAML URLs | Exact vendor-provided values |
| SAML certificate | Expiration tracked |
| SAML claims | Minimal required attributes |
| Group claims | Avoid unless required |
| OIDC redirect URIs | Exact and minimal |
| Implicit grant | Disabled unless legacy requirement exists |
| OAuth permissions | Least privilege |
| Admin consent | Granted only after review |
| Provisioning mode | Automatic only after test connection and mapping review |
| Provisioning scope | Assigned users/groups |
| Attribute mappings | Reviewed before enablement |
| Accidental deletion protection | Enabled |
| Sign-in logs | Reviewed after SSO test |
| Provisioning logs | Reviewed after test provisioning |
| Break-glass | Vendor fallback/admin access confirmed |

## Admin_Notes

~~~text
Enterprise app:
App registration:
Application/client ID:
Service principal object ID:
SSO method:
Assignment required:
Assigned group:
SAML Entity ID:
SAML Reply URL:
SAML Sign-on URL:
SAML certificate thumbprint:
SAML certificate expiration:
NameID:
Claims:
OIDC redirect URI:
OAuth permissions:
Provisioning mode:
Provisioning scope:
SCIM tenant URL:
SCIM test result:
Provision on demand result:
Test user:
SSO test result:
Provisioning log result:
Sign-in log result:
Fallback login confirmed:
Known exceptions:
~~~

## Related_Labs

| Workbook                                                                             | Relationship                                                          |
| ------------------------------------------------------------------------------------ | --------------------------------------------------------------------- |
| 01_Configure_Enterprise_Applications_Service_Principals_And_Assignments.md           | Covers Enterprise Application assignments and owners                  |
| 02_Configure_App_Registrations_API_Permissions_Secrets_Certificates_And_Owners.md    | Covers app registration settings, API permissions, and credentials    |
| 03_Configure_Admin_Consent_User_Consent_And_Publisher_Verification_Baseline.md       | Covers consent governance and publisher trust                         |
| 05_Configure_Conditional_Access_App_Targeting_Session_Controls_And_Access_Reviews.md | Covers Conditional Access and access reviews                          |
| 06_Troubleshoot_Enterprise_App_SSO_Consent_Provisioning_And_API_Permission_Issues.md | Covers SSO, consent, provisioning, and API permission troubleshooting |
