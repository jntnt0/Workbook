# 01_Troubleshoot_Cloud_Admin_Login_MFA_CA_PIM_And_Role_Assignment_Issues

## 01_Troubleshoot_Cloud_Admin_Login_MFA_CA_PIM_And_Role_Assignment_Issues_Index

### Purpose

This workbook gives a structured troubleshooting path for cloud administrator access failures involving Microsoft Entra sign-in, MFA, Conditional Access, Privileged Identity Management, admin role assignment, Azure RBAC assignment, portal access, and emergency administrative recovery.

### Scope

Use this workbook when an administrator reports one or more of the following:

- Cannot sign in to Microsoft Entra admin center, Microsoft 365 admin center, Azure portal, Defender portal, Purview portal, or Intune admin center
- MFA prompt fails, loops, times out, or blocks access
- Conditional Access blocks an admin unexpectedly
- PIM activation fails, role does not appear, approval does not arrive, or elevation does not grant expected access
- Admin role exists but permissions do not work
- Azure role assignment exists but resource access is denied
- Portal loads but blades, subscriptions, users, groups, policies, or audit records are missing
- Break-glass account validation fails
- Access worked previously and now fails after a policy, role, license, identity, or tenant change

### Assumptions

- You have at least one working administrator account, break-glass account, or delegated support path.
- You can access at least one of these: Microsoft Entra admin center, Azure portal, Microsoft 365 admin center, Microsoft Graph PowerShell, Azure PowerShell, Azure CLI, or Microsoft Graph Explorer.
- You are troubleshooting access, not resetting tenant ownership through Microsoft Support unless all administrative access is lost.
- All findings should be captured in the incident review and change log workbook after remediation.

### Required Admin Roles

| Task Area | Minimum Role |
|---|---|
| View sign-in logs | Reports Reader, Security Reader, Global Reader, Security Administrator |
| Review Conditional Access | Conditional Access Administrator, Security Administrator, Global Administrator |
| Modify Conditional Access | Conditional Access Administrator, Security Administrator, Global Administrator |
| Review users and authentication methods | Authentication Administrator, Privileged Authentication Administrator, Global Reader |
| Reset admin MFA methods | Privileged Authentication Administrator or Global Administrator |
| Review Microsoft Entra roles | Privileged Role Administrator, Global Administrator, Global Reader |
| Manage PIM | Privileged Role Administrator |
| Review Azure RBAC | Reader at scope, User Access Administrator, Owner |
| Assign Azure RBAC | User Access Administrator or Owner |
| Review audit logs | Reports Reader, Security Reader, Global Reader |
| Emergency recovery | Break-glass Global Administrator |

---

## 01_Troubleshoot_Cloud_Admin_Login_MFA_CA_PIM_And_Role_Assignment_Issues_Mental_Model

Cloud admin access depends on multiple gates. Do not assume a login failure is only a password problem.

Access chain:

1. Identity exists and is enabled.
2. Password or authentication credential is valid.
3. MFA registration and authentication method can satisfy the prompt.
4. Sign-in risk and user risk are not blocking the session.
5. Conditional Access allows the session.
6. Device, location, network, app, and client conditions satisfy policy requirements.
7. The admin has the correct Microsoft Entra role, Microsoft 365 role, workload role, or Azure RBAC assignment.
8. PIM role is eligible, activatable, approved if needed, and active at the right scope.
9. Token and portal session have refreshed after role activation.
10. Licensing and service availability are not hiding features or blocking workflows.
11. The requested resource is in the correct tenant, directory, subscription, admin unit, management group, or workload portal.

Traditional troubleshooting order:

1. Confirm the user.
2. Confirm the tenant.
3. Confirm the exact portal or resource.
4. Read the sign-in log.
5. Read Conditional Access result.
6. Verify MFA method and prompt behavior.
7. Verify role assignment and scope.
8. Verify PIM state.
9. Refresh token or session.
10. Only then change policy or roles.

---

## 01_Troubleshoot_Cloud_Admin_Login_MFA_CA_PIM_And_Role_Assignment_Issues_Symptom_Map

| Symptom | Likely Cause | Primary Evidence | First Check |
|---|---|---|---|
| User cannot sign in at all | Disabled account, password issue, blocked sign-in, risk policy, tenant mismatch | Entra sign-in logs | User account state |
| User signs in but receives MFA loop | Broken MFA method, stale session, unsupported auth method, CA auth strength mismatch | Sign-in log authentication details | Authentication methods |
| MFA prompt never arrives | Phone/app method issue, number changed, app re-registration required, network block | Authentication details | User auth methods |
| Error says access blocked by policy | Conditional Access grant/session control failure | Sign-in log Conditional Access tab | Failed CA policy |
| Azure portal opens but subscription missing | Wrong directory, missing RBAC, disabled subscription, PIM not active | Azure portal directories + role assignments | Directory and subscription filter |
| Entra admin center opens but cannot edit users | Missing Entra admin role, admin unit scope, PIM not active | Role assignments and audit logs | Active role and scope |
| PIM role missing from My roles | Eligibility expired, wrong tenant, role assigned to group not user, PIM group issue | PIM assignment list | Eligible assignments |
| PIM activation succeeds but access still denied | Token not refreshed, wrong scope, role propagation delay, resource provider cache | Access check and role assignments | Sign out/in and check active assignment |
| Role assignment exists but access denied | Wrong scope, deny assignment, management group/subscription mismatch, classic admin confusion | IAM Access check | Scope inheritance |
| Admin cannot manage Conditional Access | Missing Conditional Access Administrator, Security Administrator, or Global Administrator | Entra role assignments | Directory role |
| Break-glass account fails MFA | Bad design: break-glass account included in MFA/CA policy | Sign-in log and CA policy include/exclude | CA exclusions |
| Portal errors across many users | Service incident or portal outage | Service health | Microsoft 365 service health / Azure status |

---

## 01_Troubleshoot_Cloud_Admin_Login_MFA_CA_PIM_And_Role_Assignment_Issues_Evidence_Collection

| Evidence | Where To Get It | Why It Matters |
|---|---|---|
| Exact user principal name | User report, Entra user object | Avoid troubleshooting wrong account |
| Exact portal URL | Browser address bar | Different portals use different apps and policies |
| Exact error message | Screenshot or copied text | Error codes usually point to auth, CA, MFA, or RBAC |
| Timestamp with timezone | User report | Needed for sign-in log correlation |
| Source IP / location | Sign-in logs or user network | CA location policy may be involved |
| Device used | User report / sign-in logs | Device compliance or hybrid join may be required |
| Browser/client/app | Sign-in logs | CA can target client apps |
| Sign-in log result | Entra admin center | Primary source of truth for authentication failures |
| Conditional Access result | Sign-in log CA tab | Shows policy grant/session decision |
| Authentication details | Sign-in log details | Shows MFA method, requirement, and failure stage |
| User risk / sign-in risk | Protection logs | Risk policy may block access |
| Role assignment state | Entra roles, Azure IAM, PIM | Confirms entitlement |
| Active PIM activation | PIM My roles / admin view | Confirms elevation actually happened |
| Audit log entry | Entra audit logs | Shows recent policy or role changes |
| Service health | M365 admin center / Azure Service Health | Avoid chasing tenant config during outage |

---

## 01_Troubleshoot_Cloud_Admin_Login_MFA_CA_PIM_And_Role_Assignment_Issues_Triage_Order

1. Confirm this is not a tenant-wide outage or service incident.
2. Confirm the affected administrator is using the correct account and tenant.
3. Capture the exact timestamp, error, portal, and affected resource.
4. Pull the Microsoft Entra sign-in log for the failed attempt.
5. Review the Conditional Access result from that sign-in event.
6. Review authentication details and MFA method used.
7. Check account state, password status, risk state, and authentication methods.
8. Check Microsoft Entra admin role assignment.
9. Check Azure RBAC assignment at the exact management group, subscription, resource group, or resource scope.
10. Check PIM eligibility, activation, approval, MFA, justification, and active assignment state.
11. Force a clean token refresh after any activation or role change.
12. Apply the smallest safe remediation.
13. Verify with a fresh sign-in and a resource-level access test.
14. Record the incident, root cause, fix, rollback path, and preventive control.

---

## 01_Troubleshoot_Cloud_Admin_Login_MFA_CA_PIM_And_Role_Assignment_Issues_Planning_Table

| Item | Value |
|---|---|
| Tenant name | `<tenant-name>` |
| Tenant ID | `<tenant-id>` |
| Affected admin UPN | `<admin-user@domain.com>` |
| Affected portal | `<portal-url>` |
| Affected workload | `<Entra / Azure / M365 / Defender / Purview / Intune>` |
| Affected resource | `<resource-name-or-scope>` |
| Reported error | `<exact-error>` |
| First failed timestamp | `<yyyy-mm-dd hh:mm timezone>` |
| User location | `<city/state/country>` |
| Source IP | `<public-ip>` |
| Device name | `<device-name>` |
| Browser/client | `<browser-or-client>` |
| Break-glass tested | `<yes-no>` |
| Change window active | `<yes-no>` |
| Recent CA change | `<yes-no>` |
| Recent role change | `<yes-no>` |
| Recent MFA/auth method change | `<yes-no>` |
| Recent license change | `<yes-no>` |
| Incident record | `<ticket-or-change-id>` |

---

## 01_Troubleshoot_Cloud_Admin_Login_MFA_CA_PIM_And_Role_Assignment_Issues_Configuration_Checklist

| Step | Task | Device | Command | PowerShell | Expected Result |
|---|---|---|---|---|---|
| 1 | Confirm service health before changing tenant config | Admin workstation | Open Microsoft 365 admin center > Health > Service health; open Azure portal > Service Health | `Get-MgServiceAnnouncementIssue -Top 10` | No active incident explains the login or portal failure |
| 2 | Confirm correct tenant and account | Admin workstation | Open portal directory selector and verify tenant | `Get-MgOrganization` | Tenant ID matches expected tenant |
| 3 | Confirm user object exists and is enabled | Admin workstation | Entra admin center > Users > affected admin | `Get-MgUser -UserId <admin-user@domain.com> -Property Id,UserPrincipalName,AccountEnabled,UserType,AssignedLicenses` | Account exists, is enabled, and is the expected user |
| 4 | Check whether sign-in is blocked | Admin workstation | Entra admin center > Users > affected admin > Properties | `Get-MgUser -UserId <admin-user@domain.com> -Property AccountEnabled` | AccountEnabled is True |
| 5 | Capture failed sign-in event | Admin workstation | Entra admin center > Monitoring > Sign-in logs > filter UPN and timestamp | `Get-MgAuditLogSignIn -Filter "userPrincipalName eq '<admin-user@domain.com>'" -Top 10` | Failed event is located |
| 6 | Record sign-in error code and failure reason | Admin workstation | Open sign-in event > Basic info | `Get-MgAuditLogSignIn -Filter "userPrincipalName eq '<admin-user@domain.com>'" -Top 10 | Select-Object CreatedDateTime,Status,AppDisplayName,IPAddress,ConditionalAccessStatus` | Error code and failure reason are captured |
| 7 | Review Conditional Access decision | Admin workstation | Open sign-in event > Conditional Access tab | Use Graph sign-in output and portal details | Failed, not applied, report-only, or success state is identified |
| 8 | Identify blocking Conditional Access policy | Admin workstation | In sign-in event, expand failed CA policy | Not reliable by basic Graph output; use portal for policy detail | Exact blocking policy is identified |
| 9 | Check whether the user is directly included, group included, or excluded incorrectly | Admin workstation | Entra admin center > Protection > Conditional Access > Policies > target policy | `Get-MgIdentityConditionalAccessPolicy -All` | Policy targeting path is understood |
| 10 | Check location condition | Admin workstation | Sign-in event > Location and IP; CA policy > Conditions > Locations | `Get-MgAuditLogSignIn -Filter "userPrincipalName eq '<admin-user@domain.com>'" -Top 10 | Select-Object IPAddress,Location` | IP/location matches policy intent |
| 11 | Check device condition | Admin workstation | Sign-in event > Device info; CA policy device filters | `Get-MgAuditLogSignIn -Filter "userPrincipalName eq '<admin-user@domain.com>'" -Top 10 | Select-Object DeviceDetail` | Device compliance/join state is not unexpectedly blocking |
| 12 | Check client app condition | Admin workstation | Sign-in event > Client app | Use portal sign-in event | Browser, mobile app, desktop client, or legacy auth condition is identified |
| 13 | Check authentication strength requirement | Admin workstation | CA policy > Grant controls > Authentication strength | `Get-MgIdentityConditionalAccessPolicy -All | Select-Object DisplayName,GrantControls` | MFA method can satisfy required strength |
| 14 | Review authentication details | Admin workstation | Sign-in event > Authentication details | Portal is preferred for detailed MFA step review | MFA requirement, method, and failure stage are identified |
| 15 | Review registered authentication methods | Admin workstation | Entra admin center > Users > affected admin > Authentication methods | `Get-MgUserAuthenticationMethod -UserId <admin-user@domain.com>` | User has valid, current authentication methods |
| 16 | Reset or update broken MFA method only when verified | Admin workstation | Entra admin center > Users > affected admin > Authentication methods > require re-register MFA or add method | `Remove-MgUserAuthenticationPhoneMethod -UserId <admin-user@domain.com> -PhoneAuthenticationMethodId <method-id>` | Broken method is removed or re-registration is required |
| 17 | Check user risk state | Admin workstation | Entra admin center > Protection > Risky users | `Get-MgIdentityProtectionRiskyUser -Filter "userPrincipalName eq '<admin-user@domain.com>'"` | User risk is none, dismissed, confirmed safe, or remediated |
| 18 | Check risky sign-in | Admin workstation | Entra admin center > Protection > Risky sign-ins | `Get-MgIdentityProtectionRiskDetection -Filter "userPrincipalName eq '<admin-user@domain.com>'"` | Risk policy is confirmed or ruled out |
| 19 | Check Microsoft Entra directory role assignment | Admin workstation | Entra admin center > Roles and administrators > search role > Assignments | `Get-MgRoleManagementDirectoryRoleAssignment -All -ExpandProperty Principal,RoleDefinition | Where-Object {$_.Principal.AdditionalProperties.userPrincipalName -eq '<admin-user@domain.com>'}` | Required admin role is assigned |
| 20 | Check role assignment through group | Admin workstation | Entra admin center > Groups > group > Assigned roles | `Get-MgUserMemberOf -UserId <admin-user@domain.com>` | Role-assignable group membership is confirmed if used |
| 21 | Check administrative unit scope | Admin workstation | Entra admin center > Roles and administrators > assignment scope | `Get-MgRoleManagementDirectoryRoleAssignment -All -ExpandProperty Principal,DirectoryScope,RoleDefinition` | Role scope covers the object being administered |
| 22 | Check PIM eligibility | Admin workstation | Entra admin center > Identity Governance > PIM > Microsoft Entra roles > Assignments | `Get-MgRoleManagementDirectoryRoleEligibilityScheduleInstance -All -ExpandProperty Principal,RoleDefinition | Where-Object {$_.Principal.AdditionalProperties.userPrincipalName -eq '<admin-user@domain.com>'}` | User has eligible assignment if PIM is expected |
| 23 | Check PIM active assignment | Admin workstation | PIM > Microsoft Entra roles > Assignments > Active assignments | `Get-MgRoleManagementDirectoryRoleAssignmentScheduleInstance -All -ExpandProperty Principal,RoleDefinition | Where-Object {$_.Principal.AdditionalProperties.userPrincipalName -eq '<admin-user@domain.com>'}` | Role is currently active if elevation is required |
| 24 | Check PIM activation requirements | Admin workstation | PIM > Microsoft Entra roles > Role settings | `Get-MgPolicyRoleManagementPolicyAssignment -Filter "scopeId eq '/' and scopeType eq 'DirectoryRole'"` | MFA, approval, ticket, justification, and duration settings are understood |
| 25 | Check pending PIM approval | Admin workstation | PIM > Approve requests | Portal preferred | Request is approved, denied, expired, or pending |
| 26 | Refresh token after PIM activation | Admin workstation | Sign out of all portals; close browser; reopen private window; sign in again | `Disconnect-MgGraph; Connect-MgGraph -Scopes "Directory.Read.All","RoleManagement.Read.Directory"` | New session contains active role claims |
| 27 | Check Azure subscription directory | Admin workstation | Azure portal > Directories + subscriptions | `az account tenant list` | User is in correct tenant |
| 28 | Check Azure subscription visibility | Admin workstation | Azure portal > Subscriptions | `az account list --output table` | Expected subscription is visible |
| 29 | Check Azure RBAC assignment at exact scope | Admin workstation | Azure portal > resource/subscription > Access control IAM > Check access | `Get-AzRoleAssignment -SignInName <admin-user@domain.com> -Scope <scope>` | Correct Azure role exists at correct scope |
| 30 | Check inherited Azure RBAC assignment | Admin workstation | IAM > Role assignments > Inherited | `Get-AzRoleAssignment -SignInName <admin-user@domain.com> | Select-Object DisplayName,RoleDefinitionName,Scope` | Assignment is inherited from parent scope if expected |
| 31 | Check Azure deny assignments | Admin workstation | Azure portal > IAM > Deny assignments if visible | `Get-AzDenyAssignment -Scope <scope>` | Deny assignment is not overriding allowed access |
| 32 | Check management group access | Admin workstation | Azure portal > Management groups > IAM | `Get-AzRoleAssignment -SignInName <admin-user@domain.com> -Scope /providers/Microsoft.Management/managementGroups/<mg-name>` | Management group role covers target subscriptions if expected |
| 33 | Check resource provider registration issue masquerading as access issue | Admin workstation | Azure portal > Subscription > Resource providers | `Get-AzResourceProvider -ProviderNamespace Microsoft.<ProviderName>` | Provider is registered when required |
| 34 | Check license dependency for portal feature | Admin workstation | M365 admin center > Users > Licenses and apps | `Get-MgUserLicenseDetail -UserId <admin-user@domain.com>` | Required license exists if feature is license-gated |
| 35 | Check audit logs for recent access-impacting changes | Admin workstation | Entra admin center > Monitoring > Audit logs | `Get-MgAuditLogDirectoryAudit -Top 50 | Select-Object ActivityDateTime,ActivityDisplayName,InitiatedBy,TargetResources` | Recent CA, MFA, role, group, or user changes are identified |
| 36 | Check emergency access account policy exclusion | Admin workstation | Conditional Access policies > Users > Exclude | Review each critical CA policy manually | Break-glass account is excluded from restrictive CA/MFA policies by design |
| 37 | Test break-glass sign-in from known secure location | Admin workstation | Use private browser window and sign in to Entra admin center | Not applicable | Emergency account can access tenant if normal admin path fails |
| 38 | Apply smallest remediation | Admin workstation | Modify only the failed setting, not unrelated policies | Use specific Graph/Az cmdlet only after evidence confirms root cause | Access restored without broad security weakening |
| 39 | Verify with fresh session | Admin workstation | Sign out/in, private browser, repeat failed operation | Reconnect Graph/Az session | Admin can perform intended action |
| 40 | Record incident and rollback notes | Admin workstation | Update incident/change record | Not applicable | Root cause, fix, evidence, and rollback are documented |

---

## 01_Troubleshoot_Cloud_Admin_Login_MFA_CA_PIM_And_Role_Assignment_Issues_Detailed_Workflow

### Phase 1: Confirm Scope And Avoid Blind Changes

| Step | Task | Device | Command | PowerShell | Expected Result |
|---|---|---|---|---|---|
| 1 | Ask for exact failure path | Admin workstation | Capture portal URL, username, resource, and error screenshot | Not applicable | You know exactly where access fails |
| 2 | Confirm whether issue affects one admin or many | Admin workstation | Compare reports from multiple admins | Not applicable | Scope is single-user, group, role, portal, or tenant-wide |
| 3 | Check service health | Admin workstation | Microsoft 365 admin center > Health > Service health; Azure portal > Service Health | `Get-MgServiceAnnouncementIssue -Top 10` | Service incident is ruled in or out |
| 4 | Confirm no emergency change window is active | Admin workstation | Review change calendar or incident bridge | Not applicable | You avoid reversing an intentional security change |
| 5 | Confirm break-glass availability before risky CA edits | Admin workstation | Test emergency admin sign-in from private browser | Not applicable | Tenant recovery path exists |

### Phase 2: Sign-In Log First

| Step | Task | Device | Command | PowerShell | Expected Result |
|---|---|---|---|---|---|
| 1 | Locate failed sign-in | Admin workstation | Entra admin center > Monitoring > Sign-in logs > filter UPN | `Get-MgAuditLogSignIn -Filter "userPrincipalName eq '<admin-user@domain.com>'" -Top 20` | Correct failed sign-in event is found |
| 2 | Capture application | Admin workstation | Open event > Application | `Get-MgAuditLogSignIn -Filter "userPrincipalName eq '<admin-user@domain.com>'" -Top 20 | Select-Object CreatedDateTime,AppDisplayName,ResourceDisplayName` | Target app is known |
| 3 | Capture failure reason | Admin workstation | Open event > Status | `Get-MgAuditLogSignIn -Filter "userPrincipalName eq '<admin-user@domain.com>'" -Top 20 | Select-Object Status` | Error code and failure reason are captured |
| 4 | Capture CA status | Admin workstation | Open event > Conditional Access | `Get-MgAuditLogSignIn -Filter "userPrincipalName eq '<admin-user@domain.com>'" -Top 20 | Select-Object ConditionalAccessStatus` | CA status is success, failure, not applied, or report-only |
| 5 | Capture MFA step | Admin workstation | Open event > Authentication details | Portal preferred | MFA satisfied, failed, interrupted, or not prompted is known |
| 6 | Capture device and network context | Admin workstation | Open event > Device info and Location | `Get-MgAuditLogSignIn -Filter "userPrincipalName eq '<admin-user@domain.com>'" -Top 20 | Select-Object IPAddress,Location,DeviceDetail` | Device, IP, and location are known |

### Phase 3: Account And MFA Checks

| Step | Task | Device | Command | PowerShell | Expected Result |
|---|---|---|---|---|---|
| 1 | Confirm account is enabled | Admin workstation | Entra admin center > Users > affected user > Properties | `Get-MgUser -UserId <admin-user@domain.com> -Property AccountEnabled` | AccountEnabled is True |
| 2 | Confirm user type | Admin workstation | Entra user object > Properties | `Get-MgUser -UserId <admin-user@domain.com> -Property UserType` | User type matches expected Member or Guest |
| 3 | Confirm sign-in identity is not stale guest account | Admin workstation | Check home tenant and resource tenant | `Get-MgUser -UserId <admin-user@domain.com> -Property UserPrincipalName,UserType,ExternalUserState` | User is signing into correct tenant context |
| 4 | Review authentication methods | Admin workstation | User > Authentication methods | `Get-MgUserAuthenticationMethod -UserId <admin-user@domain.com>` | Valid method exists |
| 5 | Check for changed phone or lost Authenticator app | Admin workstation | Compare user report to registered methods | `Get-MgUserAuthenticationPhoneMethod -UserId <admin-user@domain.com>` | Registered method matches user possession |
| 6 | Require MFA re-registration if method is stale | Admin workstation | User > Authentication methods > Require re-register MFA | Not consistently exposed in basic Graph cmdlets; use portal | User must register fresh MFA methods |
| 7 | Add temporary access pass if allowed by policy | Admin workstation | User > Authentication methods > Add Temporary Access Pass | `New-MgUserAuthenticationTemporaryAccessPassMethod -UserId <admin-user@domain.com> -BodyParameter @{lifetimeInMinutes=60;isUsableOnce=$true}` | User can recover sign-in securely |
| 8 | Remove bad method only after confirming identity | Admin workstation | User > Authentication methods > delete stale method | `Remove-MgUserAuthenticationPhoneMethod -UserId <admin-user@domain.com> -PhoneAuthenticationMethodId <method-id>` | Bad method no longer blocks MFA |
| 9 | Retest sign-in | Admin workstation | Private browser window > target portal | Not applicable | MFA succeeds or next failure is visible |

### Phase 4: Conditional Access Checks

| Step | Task | Device | Command | PowerShell | Expected Result |
|---|---|---|---|---|---|
| 1 | Identify failed CA policy from sign-in event | Admin workstation | Sign-in log > Conditional Access tab | Portal preferred | Exact policy name is known |
| 2 | Open failed policy | Admin workstation | Entra admin center > Protection > Conditional Access > Policies | `Get-MgIdentityConditionalAccessPolicy -All | Where-Object {$_.DisplayName -like "*<policy-name>*"}` | Policy settings are visible |
| 3 | Review user targeting | Admin workstation | Policy > Users | `Get-MgIdentityConditionalAccessPolicy -ConditionalAccessPolicyId <policy-id>` | User inclusion/exclusion path is known |
| 4 | Review group targeting | Admin workstation | Check included groups and excluded groups | `Get-MgUserMemberOf -UserId <admin-user@domain.com>` | Group membership explains policy application |
| 5 | Review cloud app targeting | Admin workstation | Policy > Target resources | Inspect policy object | Target portal/app is included or excluded |
| 6 | Review conditions | Admin workstation | Policy > Conditions | Inspect policy object | Device, location, risk, client app, or platform condition is identified |
| 7 | Review grant controls | Admin workstation | Policy > Grant | Inspect GrantControls | MFA, compliant device, hybrid joined, auth strength, or approved app requirement is known |
| 8 | Review session controls | Admin workstation | Policy > Session | Inspect SessionControls | Sign-in frequency, app control, or persistent browser control is known |
| 9 | Use What If tool before changing policy | Admin workstation | Conditional Access > What If | Not applicable | Expected policy result matches observed failure |
| 10 | Apply narrow exclusion only if emergency access is needed | Admin workstation | Policy > Users > Exclude specific emergency group/user | `Update-MgIdentityConditionalAccessPolicy -ConditionalAccessPolicyId <policy-id> -BodyParameter <updated-policy-body>` | Admin access restored without disabling whole policy |
| 11 | Prefer report-only clone for testing permanent fix | Admin workstation | Duplicate policy or create test policy in report-only | Not applicable | Permanent CA correction can be validated safely |
| 12 | Retest sign-in | Admin workstation | Private browser > target portal | Not applicable | CA no longer blocks valid admin access |

### Phase 5: Microsoft Entra Role And PIM Checks

| Step | Task | Device | Command | PowerShell | Expected Result |
|---|---|---|---|---|---|
| 1 | Identify required admin action | Admin workstation | Determine whether action is user admin, role admin, CA admin, app admin, etc. | Not applicable | Required role is known |
| 2 | Check active Entra role assignments | Admin workstation | Entra admin center > Roles and administrators > role > Assignments | `Get-MgRoleManagementDirectoryRoleAssignmentScheduleInstance -All -ExpandProperty Principal,RoleDefinition | Where-Object {$_.Principal.AdditionalProperties.userPrincipalName -eq '<admin-user@domain.com>'}` | Active role exists if needed |
| 3 | Check eligible Entra role assignments | Admin workstation | PIM > Microsoft Entra roles > My roles / Assignments | `Get-MgRoleManagementDirectoryRoleEligibilityScheduleInstance -All -ExpandProperty Principal,RoleDefinition | Where-Object {$_.Principal.AdditionalProperties.userPrincipalName -eq '<admin-user@domain.com>'}` | Eligible role exists if PIM is expected |
| 4 | Check assignment scope | Admin workstation | Role assignment details > Scope | `Get-MgRoleManagementDirectoryRoleAssignmentScheduleInstance -All -ExpandProperty DirectoryScope,AppScope,Principal,RoleDefinition` | Scope includes target object |
| 5 | Check admin unit boundary | Admin workstation | Entra admin center > Administrative units | `Get-MgDirectoryAdministrativeUnit -All` | Role is not limited away from target object |
| 6 | Check role-assignable group path | Admin workstation | Group > Roles and administrators | `Get-MgUserMemberOf -UserId <admin-user@domain.com>` | User is member of expected role-assignable group |
| 7 | Activate PIM role | Admin workstation | PIM > My roles > Activate | Use portal for activation workflow | Role activation completes |
| 8 | Check pending approval | Admin workstation | PIM > Approve requests | Portal preferred | Approval is granted or denied |
| 9 | Check activation duration | Admin workstation | PIM active assignments | Portal preferred | Role remains active during test |
| 10 | Refresh token after activation | Admin workstation | Sign out/in or open private browser | `Disconnect-MgGraph; Connect-MgGraph -Scopes "RoleManagement.Read.Directory","Directory.Read.All"` | Token contains active role |
| 11 | Retest admin action | Admin workstation | Repeat failed admin operation | Not applicable | Permission succeeds or next failure is isolated |

### Phase 6: Azure RBAC Checks

| Step | Task | Device | Command | PowerShell | Expected Result |
|---|---|---|---|---|---|
| 1 | Confirm correct Azure directory | Admin workstation | Azure portal > Settings > Directories + subscriptions | `az account show` | Tenant ID matches target tenant |
| 2 | Confirm subscription visibility | Admin workstation | Azure portal > Subscriptions | `Get-AzSubscription` | Target subscription is listed |
| 3 | Confirm selected subscription filter | Admin workstation | Azure portal > Directories + subscriptions > subscription filter | `az account set --subscription <subscription-id>` | Portal is pointed at correct subscription |
| 4 | Check access at exact scope | Admin workstation | Resource/subscription/resource group > IAM > Check access | `Get-AzRoleAssignment -SignInName <admin-user@domain.com> -Scope <scope>` | Required role exists |
| 5 | Check inherited role assignments | Admin workstation | IAM > Role assignments > Inherited | `Get-AzRoleAssignment -SignInName <admin-user@domain.com> | Select-Object RoleDefinitionName,Scope` | Parent scope assignment explains access |
| 6 | Check group-based Azure RBAC | Admin workstation | IAM > Role assignments > group; Entra group membership | `Get-AzRoleAssignment -Scope <scope> | Where-Object {$_.ObjectType -eq "Group"}` | User has access through expected group |
| 7 | Check deny assignments | Admin workstation | IAM > Deny assignments | `Get-AzDenyAssignment -Scope <scope>` | No deny assignment blocks action |
| 8 | Check role definition actions | Admin workstation | IAM > Roles > role definition | `Get-AzRoleDefinition -Name "<role-name>"` | Role includes required action |
| 9 | Check provider registration | Admin workstation | Subscription > Resource providers | `Get-AzResourceProvider -ProviderNamespace Microsoft.<ProviderName>` | Provider is registered |
| 10 | Check management group inheritance | Admin workstation | Management groups > target MG > IAM | `Get-AzRoleAssignment -Scope /providers/Microsoft.Management/managementGroups/<mg-name>` | Management group assignment applies |
| 11 | Re-authenticate Azure session | Admin workstation | Sign out/in from Azure portal | `Disconnect-AzAccount; Connect-AzAccount -Tenant <tenant-id>; Set-AzContext -Subscription <subscription-id>` | New token has current RBAC claims |
| 12 | Retest target resource action | Admin workstation | Repeat failed Azure operation | Run the originally failed Az command again | Action succeeds or returns a different root cause |

### Phase 7: Emergency Recovery Guardrails

| Step | Task | Device | Command | PowerShell | Expected Result |
|---|---|---|---|---|---|
| 1 | Confirm at least two emergency access accounts exist | Admin workstation | Entra admin center > Users > search break-glass accounts | `Get-MgUser -Filter "startswith(displayName,'Emergency') or startswith(userPrincipalName,'breakglass')"` | Two cloud-only emergency accounts are identified |
| 2 | Confirm emergency accounts are cloud-only | Admin workstation | User properties > On-premises sync | `Get-MgUser -UserId <breakglass@domain.com> -Property OnPremisesSyncEnabled` | OnPremisesSyncEnabled is False or empty |
| 3 | Confirm emergency accounts are excluded from restrictive CA | Admin workstation | Review each critical CA policy exclusions | Manual review required | Emergency accounts are excluded from lockout policies |
| 4 | Confirm emergency accounts are monitored | Admin workstation | Audit/sign-in alert rules | Review alert configuration | Sign-in generates alert |
| 5 | Test emergency account access periodically | Admin workstation | Private browser sign-in to Entra admin center | Not applicable | Emergency account can access tenant |
| 6 | Do not use emergency account for routine admin work | Admin workstation | Review sign-in logs | `Get-MgAuditLogSignIn -Filter "userPrincipalName eq '<breakglass@domain.com>'" -Top 20` | No routine usage appears |

---

## 01_Troubleshoot_Cloud_Admin_Login_MFA_CA_PIM_And_Role_Assignment_Issues_Verification_Commands

### Microsoft Graph PowerShell Baseline

| Purpose | PowerShell |
|---|---|
| Connect to Graph for read-only troubleshooting | `Connect-MgGraph -Scopes "User.Read.All","Directory.Read.All","AuditLog.Read.All","Policy.Read.All","RoleManagement.Read.Directory","IdentityRiskyUser.Read.All","IdentityRiskEvent.Read.All"` |
| Confirm tenant | `Get-MgOrganization | Select-Object DisplayName,Id,VerifiedDomains` |
| Confirm user state | `Get-MgUser -UserId <admin-user@domain.com> -Property Id,UserPrincipalName,DisplayName,AccountEnabled,UserType,AssignedLicenses` |
| Get recent sign-ins | `Get-MgAuditLogSignIn -Filter "userPrincipalName eq '<admin-user@domain.com>'" -Top 20 | Select-Object CreatedDateTime,AppDisplayName,ResourceDisplayName,IPAddress,ConditionalAccessStatus,Status` |
| Get authentication methods | `Get-MgUserAuthenticationMethod -UserId <admin-user@domain.com>` |
| Get risky user state | `Get-MgIdentityProtectionRiskyUser -Filter "userPrincipalName eq '<admin-user@domain.com>'"` |
| Get risk detections | `Get-MgIdentityProtectionRiskDetection -Filter "userPrincipalName eq '<admin-user@domain.com>'"` |
| List Conditional Access policies | `Get-MgIdentityConditionalAccessPolicy -All | Select-Object Id,DisplayName,State` |
| List active Entra role assignment instances | `Get-MgRoleManagementDirectoryRoleAssignmentScheduleInstance -All -ExpandProperty Principal,RoleDefinition | Select-Object AssignmentType,MemberType,StartDateTime,EndDateTime,Principal,RoleDefinition` |
| List eligible Entra role assignment instances | `Get-MgRoleManagementDirectoryRoleEligibilityScheduleInstance -All -ExpandProperty Principal,RoleDefinition | Select-Object MemberType,StartDateTime,EndDateTime,Principal,RoleDefinition` |
| Get recent directory audit events | `Get-MgAuditLogDirectoryAudit -Top 50 | Select-Object ActivityDateTime,ActivityDisplayName,Category,Result,InitiatedBy,TargetResources` |
| Disconnect Graph | `Disconnect-MgGraph` |

### Azure PowerShell Baseline

| Purpose | PowerShell |
|---|---|
| Connect to Azure | `Connect-AzAccount -Tenant <tenant-id>` |
| Confirm context | `Get-AzContext` |
| List subscriptions | `Get-AzSubscription | Select-Object Name,Id,TenantId,State` |
| Set subscription | `Set-AzContext -Subscription <subscription-id>` |
| Check role assignments for user | `Get-AzRoleAssignment -SignInName <admin-user@domain.com> | Select-Object DisplayName,RoleDefinitionName,Scope` |
| Check role assignments at scope | `Get-AzRoleAssignment -SignInName <admin-user@domain.com> -Scope <scope>` |
| Check role definition | `Get-AzRoleDefinition -Name "<role-name>"` |
| Check deny assignments | `Get-AzDenyAssignment -Scope <scope>` |
| Check resource provider | `Get-AzResourceProvider -ProviderNamespace Microsoft.<ProviderName>` |
| Register provider if approved | `Register-AzResourceProvider -ProviderNamespace Microsoft.<ProviderName>` |
| Disconnect Azure | `Disconnect-AzAccount` |

### Azure CLI Baseline

| Purpose | Command |
|---|---|
| Login to tenant | `az login --tenant <tenant-id>` |
| Show active account | `az account show --output table` |
| List subscriptions | `az account list --output table` |
| Set subscription | `az account set --subscription <subscription-id>` |
| List role assignments for assignee | `az role assignment list --assignee <admin-user@domain.com> --all --output table` |
| List role assignments at scope | `az role assignment list --assignee <admin-user@domain.com> --scope <scope> --all --output table` |
| List role definition | `az role definition list --name "<role-name>" --output json` |
| Check provider state | `az provider show --namespace Microsoft.<ProviderName> --query registrationState --output table` |
| Register provider if approved | `az provider register --namespace Microsoft.<ProviderName>` |

---

## 01_Troubleshoot_Cloud_Admin_Login_MFA_CA_PIM_And_Role_Assignment_Issues_Rollback

### Rollback Principles

- Do not leave emergency exclusions broader than needed.
- Do not disable Conditional Access policies tenant-wide unless there is no other recovery path.
- Do not permanently assign Global Administrator to fix a narrow workload issue.
- Do not remove MFA from administrator accounts as a long-term fix.
- Roll back temporary access changes after validation.
- Preserve sign-in logs, audit logs, screenshots, and policy exports before and after the change.

### Rollback Table

| Change Made | Rollback Action | Verification |
|---|---|---|
| Added temporary CA exclusion for admin | Remove user from exclusion after permanent fix | User can sign in under corrected policy |
| Disabled CA policy | Re-enable policy after correcting targeting or grant controls | What If and sign-in logs show expected success |
| Added temporary Global Administrator | Remove assignment or let PIM activation expire | User no longer has unnecessary high privilege |
| Added Azure Owner/User Access Administrator | Remove role assignment after resource access is fixed | IAM no longer shows temporary role |
| Added Temporary Access Pass | Let expire or delete method after MFA recovery | User has strong MFA method registered |
| Removed stale MFA method | Confirm user registers replacement method | Authentication methods show approved method |
| Dismissed user risk | Confirm remediation reason and monitoring | Risky user state remains clean |
| Registered Azure resource provider | Usually no rollback required | Provider state is Registered |
| Changed group membership | Remove temporary membership | Access path returns to intended group model |
| Changed PIM role setting | Restore previous approval/MFA/duration settings | PIM activation behaves as intended |

### Policy Export Before Major CA Change

| Task | PowerShell |
|---|---|
| Export CA policies to JSON | `Get-MgIdentityConditionalAccessPolicy -All | ConvertTo-Json -Depth 20 | Out-File ".\ConditionalAccess_Backup_<yyyyMMdd_HHmm>.json"` |
| Export directory role assignment instances | `Get-MgRoleManagementDirectoryRoleAssignmentScheduleInstance -All -ExpandProperty Principal,RoleDefinition | ConvertTo-Json -Depth 20 | Out-File ".\DirectoryRoleAssignments_<yyyyMMdd_HHmm>.json"` |
| Export Azure role assignments | `Get-AzRoleAssignment | ConvertTo-Json -Depth 20 | Out-File ".\AzureRoleAssignments_<yyyyMMdd_HHmm>.json"` |

---

## 01_Troubleshoot_Cloud_Admin_Login_MFA_CA_PIM_And_Role_Assignment_Issues_Failure_Checks

| Failure | Check | Fix |
|---|---|---|
| Graph command returns insufficient privileges | Connected account lacks read scope or role | Reconnect with required scopes and use Reports Reader, Global Reader, Security Reader, or higher |
| Graph command returns empty sign-in logs | Wrong user, wrong tenant, timestamp too old, log retention exceeded | Confirm tenant and UPN; use portal filters; check retention |
| User still blocked after CA exclusion | Token stale, another CA policy also blocks, wrong user excluded | Recheck sign-in log and open new private session |
| PIM active but access denied | Token stale, wrong role, wrong scope, approval not complete | Sign out/in and check active assignment instance |
| Azure RBAC assigned but no access | Wrong subscription, deny assignment, not enough propagation time, wrong scope | Set correct context, check deny assignments, wait briefly, retest |
| Subscription missing | Directory filter wrong, no RBAC, subscription disabled | Switch directory, check IAM, check subscription state |
| Cannot see Conditional Access blade | Missing role or license | Assign Conditional Access Administrator/Security Administrator/Global Reader as appropriate |
| Cannot reset MFA methods | Missing Privileged Authentication Administrator | Use correct role or break-glass Global Administrator |
| Break-glass also blocked | Break-glass included in CA or MFA enforcement | Use remaining admin path or Microsoft Support recovery; then fix exclusions |
| Portal loads blank or partially | Browser/session/cache issue, service issue, client network issue | Private browser, alternate browser, clear cache, check service health |
| MFA app prompt fails repeatedly | Device lost, app registration broken, number changed | Verify identity, add Temporary Access Pass, require re-registration |
| Role assigned to group but not effective | Group not role-assignable, dynamic membership delay, nested group unsupported for role path | Use proper role-assignable group or direct eligible assignment |
| Admin unit scoped role cannot modify object | Object outside administrative unit | Assign correct scope or move object into correct administrative unit |
| User risk policy blocks admin | Risk not remediated or dismissed | Investigate, reset password if needed, dismiss risk only with evidence |
| Authentication strength blocks method | User method does not satisfy required strength | Register compliant method or adjust policy deliberately |

---

## 01_Troubleshoot_Cloud_Admin_Login_MFA_CA_PIM_And_Role_Assignment_Issues_Escalation_Handoff

### Escalate To Identity / Security Team When

- Conditional Access policy failure affects multiple administrators
- Break-glass access is not working
- Risk policy blocks privileged accounts
- MFA methods appear tampered with
- Privileged role assignments changed unexpectedly
- Audit logs show unauthorized role or policy changes
- PIM approvals or activations behave inconsistently
- There is evidence of account compromise

### Escalate To Azure Platform Team When

- Azure RBAC is correct but resource access still fails
- Deny assignments exist and are not understood
- Management group inheritance is involved
- Subscription is disabled, missing, moved, or under another tenant
- Resource provider registration or quota failures are mixed with access failures

### Escalate To Microsoft Support When

- No Global Administrator or emergency account can access the tenant
- Tenant-wide sign-in failure occurs without local configuration change
- Service health does not show an incident but multiple portals fail
- PIM or role assignment backend state is inconsistent across portal and Graph
- Audit/sign-in logs are unavailable during an active security incident

### Handoff Data To Include

| Field | Value |
|---|---|
| Incident ID | `<ticket-id>` |
| Tenant ID | `<tenant-id>` |
| Affected user | `<admin-user@domain.com>` |
| Affected portal/app | `<portal-or-app>` |
| Affected resource/scope | `<resource-or-scope>` |
| First failure time | `<timestamp-with-timezone>` |
| Sign-in error code | `<error-code>` |
| Conditional Access policy result | `<policy-name-and-result>` |
| MFA method involved | `<method>` |
| Role expected | `<role-name>` |
| Role assignment scope | `<scope>` |
| PIM state | `<eligible-active-pending-none>` |
| Recent changes found | `<changes>` |
| Remediation attempted | `<actions>` |
| Current blocker | `<blocker>` |
| Business impact | `<impact>` |

---

## 01_Troubleshoot_Cloud_Admin_Login_MFA_CA_PIM_And_Role_Assignment_Issues_Incident_Record_Template

| Field | Entry |
|---|---|
| Date opened | `<yyyy-mm-dd>` |
| Opened by | `<name>` |
| Affected admin | `<admin-user@domain.com>` |
| Affected workload | `<workload>` |
| Symptom | `<symptom>` |
| Root cause category | `<Login / MFA / CA / PIM / Entra role / Azure RBAC / Service health / Other>` |
| Root cause | `<confirmed-root-cause>` |
| Evidence | `<sign-in-log-id, audit-log-entry, screenshots, command-output>` |
| Fix applied | `<fix>` |
| Security impact | `<impact>` |
| Rollback action | `<rollback>` |
| Verification performed | `<verification>` |
| Preventive action | `<preventive-action>` |
| Closed by | `<name>` |
| Date closed | `<yyyy-mm-dd>` |

---

## 01_Troubleshoot_Cloud_Admin_Login_MFA_CA_PIM_And_Role_Assignment_Issues_Preventive_Controls

| Control | Target State |
|---|---|
| Emergency access accounts | At least two cloud-only break-glass accounts exist and are monitored |
| Conditional Access exclusions | Break-glass accounts excluded from lockout-prone policies |
| PIM usage | Standing Global Administrator assignments minimized |
| Admin MFA | Strong methods enforced and periodically validated |
| PIM approval | High privilege roles require approval where practical |
| Role reviews | Privileged role assignments reviewed regularly |
| CA change process | CA policies tested with What If and report-only before enforcement |
| Audit monitoring | Alerts for CA policy change, role assignment change, and break-glass sign-in |
| Token refresh process | Admins know to sign out/in after PIM activation |
| Documentation | Incident records capture root cause, fix, rollback, and prevention |

---

## 01_Troubleshoot_Cloud_Admin_Login_MFA_CA_PIM_And_Role_Assignment_Issues_Related_Labs

| Lab                                                  | Purpose                                                |
| ---------------------------------------------------- | ------------------------------------------------------ |
| Configure break-glass cloud admin accounts           | Validate emergency tenant access                       |
| Configure baseline Conditional Access admin policies | Build safe admin access controls                       |
| Test Conditional Access What If                      | Predict policy impact before enforcement               |
| Configure MFA methods and Temporary Access Pass      | Recover user authentication safely                     |
| Configure PIM for Microsoft Entra roles              | Practice eligible, active, and approval workflows      |
| Configure PIM for Azure resources                    | Practice scoped RBAC elevation                         |
| Troubleshoot Azure RBAC access denied                | Isolate scope, role, deny assignment, and token issues |
| Review Microsoft Entra sign-in logs                  | Trace login, MFA, CA, and client failures              |
| Review Microsoft Entra audit logs                    | Trace role, policy, and authentication method changes  |
| Create cloud admin incident review record            | Preserve evidence and close the loop                   |