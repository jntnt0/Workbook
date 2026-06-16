07_Manage_Admin_Roles_Role_Groups_Admin_Units_And_Delegation.md
# 07_Manage_Admin_Roles_Role_Groups_Admin_Units_And_Delegation

# 07_Manage_Admin_Roles_Role_Groups_Admin_Units_And_Delegation_Index
07_Manage_Admin_Roles_Role_Groups_Admin_Units_And_Delegation.md
07_Manage_Admin_Roles_Role_Groups_Admin_Units_And_Delegation
07_Manage_Admin_Roles_Role_Groups_Admin_Units_And_Delegation_Source_Basis
07_Manage_Admin_Roles_Role_Groups_Admin_Units_And_Delegation_Mental_Model
07_Manage_Admin_Roles_Role_Groups_Admin_Units_And_Delegation_Planning_Table
07_Manage_Admin_Roles_Role_Groups_Admin_Units_And_Delegation_Configuration_Checklist
07_Manage_Admin_Roles_Role_Groups_Admin_Units_And_Delegation_M365_Admin_Roles_Skeleton
07_Manage_Admin_Roles_Role_Groups_Admin_Units_And_Delegation_Entra_Role_Assignment_Skeleton
07_Manage_Admin_Roles_Role_Groups_Admin_Units_And_Delegation_Role_Assignable_Groups_Skeleton
07_Manage_Admin_Roles_Role_Groups_Admin_Units_And_Delegation_Admin_Units_Skeleton
07_Manage_Admin_Roles_Role_Groups_Admin_Units_And_Delegation_Exchange_Role_Groups_Skeleton
07_Manage_Admin_Roles_Role_Groups_Admin_Units_And_Delegation_PIM_And_Eligible_Access_Skeleton
07_Manage_Admin_Roles_Role_Groups_Admin_Units_And_Delegation_Delegation_Boundary_Skeleton
07_Manage_Admin_Roles_Role_Groups_Admin_Units_And_Delegation_Access_Review_Export_Skeleton
07_Manage_Admin_Roles_Role_Groups_Admin_Units_And_Delegation_Verification_Commands
07_Manage_Admin_Roles_Role_Groups_Admin_Units_And_Delegation_Rollback
07_Manage_Admin_Roles_Role_Groups_Admin_Units_And_Delegation_Failure_Checks
07_Manage_Admin_Roles_Role_Groups_Admin_Units_And_Delegation_Related_Labs

# 07_Manage_Admin_Roles_Role_Groups_Admin_Units_And_Delegation_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Microsoft 365 admin roles | Built-in Microsoft 365 admin roles, role assignment, least privilege, and admin center role management |
| Microsoft Learn | Assign admin roles in Microsoft 365 admin center | Portal-based role assignment, role comparison, multiple user assignment, and export workflow |
| Microsoft Learn | Microsoft Entra built-in roles | Tenant-wide identity roles such as Global Administrator, User Administrator, Groups Administrator, Privileged Role Administrator, and Global Reader |
| Microsoft Learn | Microsoft Entra administrative units | Scoped delegation for users, groups, and devices through administrative unit boundaries |
| Microsoft Learn | Microsoft Graph role management cmdlets | Repeatable role assignment, role assignment inventory, role definition lookup, and delegated directory scopes |
| Microsoft Learn | Microsoft Graph administrative unit cmdlets | Creating admin units, adding members, and assigning roles scoped to admin units |
| Microsoft Learn | Microsoft Entra role-assignable groups | Assigning directory roles through protected role-assignable security groups |
| Microsoft Learn | Exchange Online role groups | Exchange RBAC, role groups, role group members, management roles, and recipient management scopes |
| Microsoft Learn | Privileged Identity Management | Eligible role assignment, activation workflow, approval, justification, MFA, and time-bound privilege |
| Microsoft 365 operational practice | Delegated administration | Separates tenant-wide roles, workload roles, scoped roles, emergency access, and audit evidence |

# 07_Manage_Admin_Roles_Role_Groups_Admin_Units_And_Delegation_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Admin role | Built-in permission set that grants administrative capability in Microsoft 365 or Microsoft Entra |
| Microsoft 365 admin role | Role visible in Microsoft 365 admin center for tenant and workload administration |
| Microsoft Entra role | Identity platform role controlling directory, user, group, app, and security administration |
| Workload admin role | Admin role focused on a service such as Exchange, SharePoint, Teams, Security, Compliance, or Intune |
| Role assignment | Binding between a principal and a role definition |
| Principal | User, group, or service principal receiving a role assignment |
| Directory scope | Boundary where a role assignment applies, such as tenant root or administrative unit |
| Tenant-wide assignment | Role assignment that applies across the full tenant |
| Administrative unit | Microsoft Entra container used to scope delegated administration to selected users, groups, or devices |
| Scoped role assignment | Role assignment limited to an administrative unit instead of the full tenant |
| Role-assignable group | Protected security group that can receive Microsoft Entra role assignments |
| Exchange role group | Exchange Online RBAC group containing management roles and members |
| Management role | Exchange permission definition used inside Exchange RBAC |
| Management scope | Exchange filter or scope that limits where Exchange role assignments apply |
| Privileged Identity Management | Just-in-time administrative access model for eligible role activation |
| Eligible assignment | Role assignment requiring activation before use |
| Active assignment | Role assignment currently usable |
| Permanent assignment | Role assignment with no expiration |
| Time-bound assignment | Role assignment with start and end time |
| Break glass admin | Emergency cloud-only account with high privilege and strict protection |
| Least privilege | Admins receive only the role required for the task |
| First rule | Do not give Global Administrator when a narrower role works |
| Blunt rule | If you cannot explain why an admin has a role, remove or replace the assignment with a scoped one |

# 07_Manage_Admin_Roles_Role_Groups_Admin_Units_And_Delegation_Planning_Table
| Item | Example | Decision |
|---|---|---|
| Tenant name | `contoso.onmicrosoft.com` | `<tenant>.onmicrosoft.com` |
| Tenant ID | `00000000-0000-0000-0000-000000000000` | `<tenant-id>` |
| Admin account used | `admin@contoso.com` | `<admin-upn>` |
| Break glass account 1 | `breakglass01@contoso.onmicrosoft.com` | `<break-glass-upn-1>` |
| Break glass account 2 | `breakglass02@contoso.onmicrosoft.com` | `<break-glass-upn-2>` |
| Privileged Role Administrator | `pra@contoso.com` | `<pra-upn>` |
| Global Reader group | `RA_Global_Reader` | `<global-reader-group>` |
| Helpdesk scoped admin group | `RA_Helpdesk_Admins` | `<helpdesk-admin-group>` |
| License admin group | `RA_License_Admins` | `<license-admin-group>` |
| Exchange admin group | `EXO_RoleGroup_Recipient_Admins` | `<exchange-role-group>` |
| Administrative unit name | `AU_Atlanta_Users` | `<admin-unit-name>` |
| Administrative unit scope | Atlanta users and groups | `<admin-unit-scope>` |
| Admin unit owner | `identityadmin@contoso.com` | `<admin-unit-owner>` |
| Delegated role | User Administrator | `<delegated-role>` |
| Delegated principal | `atl.helpdesk@contoso.com` | `<delegated-admin-upn>` |
| Exchange management scope | `Atlanta Mailboxes` | `<exchange-management-scope>` |
| PIM activation requirement | MFA, justification, approval | `<pim-requirement>` |
| Access review cadence | Monthly or quarterly | `<access-review-cadence>` |
| Role assignment evidence path | `.\Evidence\M365-Roles-Delegation` | `<evidence-path>` |
| Rollback owner | `pra@contoso.com` | `<rollback-owner>` |
| Approval source | Ticket/change request | `<approval-source>` |

# 07_Manage_Admin_Roles_Role_Groups_Admin_Units_And_Delegation_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm admin account | Admin Workstation | Portal account menu | Correct admin UPN is signed in |
| 2 | Confirm correct tenant | Admin Workstation | Portal tenant switcher | Correct Microsoft 365 tenant is active |
| 3 | Confirm privileged role authority | Admin Workstation | Microsoft 365 admin center > Roles | Admin has Privileged Role Administrator, Global Administrator, or approved equivalent |
| 4 | Open Microsoft 365 role assignments | Admin Workstation | Microsoft 365 admin center > Roles > Role assignments | Role assignment page opens |
| 5 | Review common admin roles | Admin Workstation | Roles > Role assignments | Available admin roles are visible |
| 6 | Export role assignment baseline from portal if needed | Admin Workstation | Roles > Role assignments > Export | Role assignment evidence is saved |
| 7 | Open Microsoft Entra roles | Admin Workstation | Microsoft Entra admin center > Identity > Roles and admins | Entra role list opens |
| 8 | Review privileged tenant-wide role assignments | Admin Workstation | Roles and admins | Global Admin, PRA, Security Admin, Exchange Admin, and other privileged assignments are visible |
| 9 | Identify permanent high privilege assignments | Admin Workstation | Role assignment list | Permanent privileged users are documented |
| 10 | Identify role assignments that should be scoped | Operator | Planning table | Candidates for admin unit or workload delegation are known |
| 11 | Connect Microsoft Graph PowerShell | Admin Workstation | `Connect-MgGraph -Scopes "RoleManagement.ReadWrite.Directory","Directory.ReadWrite.All","User.Read.All","Group.ReadWrite.All"` | Graph session connects |
| 12 | Confirm Graph context | Admin Workstation | `Get-MgContext` | Tenant ID and scopes are visible |
| 13 | Export Entra role definitions | Admin Workstation | `Get-MgRoleManagementDirectoryRoleDefinition` | Directory role definitions are exported |
| 14 | Export Entra role assignments | Admin Workstation | `Get-MgRoleManagementDirectoryRoleAssignment` | Directory role assignments are exported |
| 15 | Export Microsoft 365 admin role evidence | Admin Workstation | Graph role assignment export | Admin role baseline is saved |
| 16 | Create role-assignable group if approved | Admin Workstation | `New-MgGroup -IsAssignableToRole:$true` | Protected role-assignable group exists |
| 17 | Add members to role-assignable group | Admin Workstation | `New-MgGroupMemberByRef` | Admins are added to role group |
| 18 | Assign directory role to role-assignable group if approved | Admin Workstation | `New-MgRoleManagementDirectoryRoleAssignment` | Role is assigned to group |
| 19 | Create administrative unit if required | Admin Workstation | `New-MgDirectoryAdministrativeUnit` | Admin unit exists |
| 20 | Add users to administrative unit | Admin Workstation | `New-MgDirectoryAdministrativeUnitMemberByRef` | Scoped objects are members of admin unit |
| 21 | Assign scoped admin role to admin unit | Admin Workstation | `New-MgRoleManagementDirectoryRoleAssignment` with admin unit scope | Delegated admin can manage only scoped objects |
| 22 | Validate admin unit membership | Admin Workstation | `Get-MgDirectoryAdministrativeUnitMember` | Expected users/groups/devices are present |
| 23 | Validate scoped role assignment | Admin Workstation | `Get-MgRoleManagementDirectoryRoleAssignment` | Directory scope points to admin unit |
| 24 | Test delegated admin access | Delegated Admin Workstation | Microsoft 365 or Entra portal test | Delegated admin can manage scoped objects |
| 25 | Confirm delegated admin cannot manage out-of-scope object | Delegated Admin Workstation | Attempt out-of-scope user update | Access is blocked or unavailable |
| 26 | Connect Exchange Online | Admin Workstation | `Connect-ExchangeOnline -UserPrincipalName "<admin-upn>"` | Exchange Online session connects |
| 27 | Export Exchange role groups | Admin Workstation | `Get-RoleGroup` | Exchange RBAC role groups are exported |
| 28 | Export Exchange role group members | Admin Workstation | `Get-RoleGroupMember` | Exchange role group membership is exported |
| 29 | Create Exchange role group if required | Admin Workstation | `New-RoleGroup` | Exchange role group is created |
| 30 | Add Exchange role group members | Admin Workstation | `Add-RoleGroupMember` | Members receive Exchange RBAC permissions |
| 31 | Create Exchange management scope if required | Admin Workstation | `New-ManagementScope` | Exchange recipient scope exists |
| 32 | Assign Exchange role group with management scope if required | Admin Workstation | `New-RoleGroup -CustomRecipientWriteScope` | Exchange role permissions are scoped |
| 33 | Validate Exchange role group permissions | Admin Workstation | `Get-ManagementRoleAssignment` | Role assignments and scopes are visible |
| 34 | Open PIM role management if licensed | Admin Workstation | Entra admin center > Identity Governance > Privileged Identity Management | PIM portal opens |
| 35 | Review eligible and active assignments | Admin Workstation | PIM > Microsoft Entra roles | Eligible and active roles are visible |
| 36 | Convert standing privilege to eligible where approved | Admin Workstation | PIM assignment workflow | User must activate role before use |
| 37 | Configure PIM activation requirements | Admin Workstation | PIM role settings | MFA, justification, approval, and duration are configured |
| 38 | Test PIM activation | Delegated Admin Workstation | PIM > Activate role | Role activates with required controls |
| 39 | Review break glass accounts | Admin Workstation | Users and role assignments | Emergency admin accounts exist and are documented |
| 40 | Confirm break glass exclusions are documented | Operator | Conditional Access and emergency access notes | Emergency access design is documented |
| 41 | Remove unnecessary permanent assignments | Admin Workstation | Portal or Graph removal command | Excess standing privilege is reduced |
| 42 | Confirm each privileged role has owner and reason | Operator | Access review table | Every role assignment has business justification |
| 43 | Export final role assignment evidence | Admin Workstation | Graph and Exchange export commands | Final evidence files are saved |
| 44 | Document admin unit boundaries | Operator | Notes | Scope and delegated admins are documented |
| 45 | Document Exchange role group boundaries | Operator | Notes | Exchange RBAC scope and members are documented |
| 46 | Document PIM policy settings | Operator | Notes / screenshots | Activation rules are documented |
| 47 | Document access review cadence | Operator | Notes | Review rhythm is established |
| 48 | Validate no unexpected Global Administrators | Admin Workstation | Graph role assignment review | Global Admin list is intentional |
| 49 | Validate no orphaned privileged assignments | Admin Workstation | Role assignment export | Former admins and stale users are not privileged |
| 50 | Document completion state | Operator | Notes | Admin roles, role groups, admin units, PIM, and delegation baseline is complete |

# 07_Manage_Admin_Roles_Role_Groups_Admin_Units_And_Delegation_M365_Admin_Roles_Skeleton
```text
Portal path:

Microsoft 365 admin center
Roles
Role assignments
```

Common role planning:

| Role | Intended Use | Standing Assignment Standard |
|---|---|---|
| Global Administrator | Emergency tenant-wide administration | Break glass and very small number of approved admins only |
| Privileged Role Administrator | Manage role assignments and PIM | Restricted to identity leadership |
| Global Reader | Read-only tenant-wide investigation | Safe default for broad visibility |
| User Administrator | User lifecycle administration | Help desk or identity admins |
| Groups Administrator | Group lifecycle administration | Collaboration or identity admins |
| License Administrator | License assignment and license cleanup | Licensing admins |
| Exchange Administrator | Exchange Online administration | Messaging admins |
| SharePoint Administrator | SharePoint and OneDrive administration | Collaboration admins |
| Teams Administrator | Teams administration | Collaboration admins |
| Service Support Administrator | Service health and support tickets | Help desk or service desk lead |
| Security Reader | Read-only security visibility | Security analysts |
| Security Administrator | Security configuration | Security admins |
| Compliance Administrator | Compliance and Purview administration | Compliance admins |
| Billing Administrator | Billing and subscription management | Finance or billing owner |
| Helpdesk Administrator | Password reset and user support tasks | Help desk admins |

Portal checklist:

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Open role assignments | Admin Workstation | Microsoft 365 admin center > Roles > Role assignments | Role list opens |
| 2 | Select target role | Admin Workstation | Select `<role-name>` | Role detail opens |
| 3 | Review assigned admins | Admin Workstation | Assigned tab | Current admins are visible |
| 4 | Review role permissions | Admin Workstation | Permissions tab / description | Admin understands role capability |
| 5 | Add admin only with approval | Admin Workstation | Assign admins | Approved admin is added |
| 6 | Remove stale admin assignment | Admin Workstation | Remove assignment | Stale admin loses role |
| 7 | Export or screenshot evidence | Admin Workstation | Export / screenshot | Role assignment evidence is saved |
| 8 | Document reason | Operator | Notes | Assignment has owner, reason, and review date |

Role assignment capture:

| Role | Principal | Assignment Type | Scope | Reason | Owner | Review Date |
|---|---|---|---|---|---|---|
| `<role-name>` | `<user-or-group>` | `<active/eligible/permanent>` | `<tenant/admin-unit/workload>` | `<reason>` | `<owner>` | `<date>` |

Operational note:

```text
Use the Microsoft 365 admin center for clear role visibility.
Use Microsoft Graph PowerShell for repeatable export, review, assignment, and rollback.
```

# 07_Manage_Admin_Roles_Role_Groups_Admin_Units_And_Delegation_Entra_Role_Assignment_Skeleton
```powershell
# Run from an admin workstation.
# Purpose: inventory and assign Microsoft Entra directory roles with Microsoft Graph PowerShell.

$EvidencePath = ".\Evidence\M365-Roles-Delegation"
New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Connect-MgGraph -Scopes "RoleManagement.ReadWrite.Directory","Directory.ReadWrite.All","User.Read.All","Group.ReadWrite.All"

Get-MgContext |
    Tee-Object "$EvidencePath\graph-context-role-management.txt"

# Export role definitions.
Get-MgRoleManagementDirectoryRoleDefinition -All |
    Sort-Object DisplayName |
    Select-Object Id,DisplayName,Description,IsEnabled |
    Export-Csv "$EvidencePath\entra-role-definitions.csv" -NoTypeInformation

# Export role assignments.
Get-MgRoleManagementDirectoryRoleAssignment -All |
    Select-Object Id,PrincipalId,RoleDefinitionId,DirectoryScopeId,AppScopeId |
    Export-Csv "$EvidencePath\entra-role-assignments.csv" -NoTypeInformation

# Display common privileged roles.
Get-MgRoleManagementDirectoryRoleDefinition -All |
    Where-Object {
        $_.DisplayName -in @(
            "Global Administrator",
            "Privileged Role Administrator",
            "User Administrator",
            "Groups Administrator",
            "Exchange Administrator",
            "SharePoint Administrator",
            "Teams Administrator",
            "License Administrator",
            "Global Reader",
            "Security Administrator",
            "Security Reader",
            "Compliance Administrator",
            "Helpdesk Administrator"
        )
    } |
    Sort-Object DisplayName |
    Format-Table Id,DisplayName,IsEnabled -AutoSize
```

Assign a tenant-wide Entra role to a user:

```powershell
# Purpose: assign a tenant-wide Microsoft Entra role to a user.
# Use tenant-wide scope only when admin unit or workload scope is not enough.

$UserPrincipalName = "<delegated-admin-upn>"
$RoleName = "<role-name>"

$User = Get-MgUser -UserId $UserPrincipalName

$RoleDefinition = Get-MgRoleManagementDirectoryRoleDefinition -All |
    Where-Object {$_.DisplayName -eq $RoleName}

$Assignment = @{
    "@odata.type" = "#microsoft.graph.unifiedRoleAssignment"
    RoleDefinitionId = $RoleDefinition.Id
    PrincipalId = $User.Id
    DirectoryScopeId = "/"
}

New-MgRoleManagementDirectoryRoleAssignment -BodyParameter $Assignment

Get-MgRoleManagementDirectoryRoleAssignment -All |
    Where-Object {
        $_.PrincipalId -eq $User.Id -and
        $_.RoleDefinitionId -eq $RoleDefinition.Id
    } |
    Format-List Id,PrincipalId,RoleDefinitionId,DirectoryScopeId
```

Remove a role assignment:

```powershell
# Purpose: remove a Microsoft Entra role assignment.

$AssignmentId = "<role-assignment-id>"

Remove-MgRoleManagementDirectoryRoleAssignment `
    -UnifiedRoleAssignmentId $AssignmentId
```

Role lookup helper:

```powershell
# Purpose: resolve role assignment IDs into readable role names and principal IDs.

$RoleDefinitions = Get-MgRoleManagementDirectoryRoleDefinition -All
$Assignments = Get-MgRoleManagementDirectoryRoleAssignment -All

foreach ($Assignment in $Assignments) {
    $Role = $RoleDefinitions | Where-Object {$_.Id -eq $Assignment.RoleDefinitionId}

    [PSCustomObject]@{
        AssignmentId = $Assignment.Id
        Role = $Role.DisplayName
        PrincipalId = $Assignment.PrincipalId
        DirectoryScopeId = $Assignment.DirectoryScopeId
    }
} |
Sort-Object Role |
Format-Table -AutoSize
```

Operational note:

```text
Tenant-wide role assignment uses DirectoryScopeId "/".
Administrative unit scoped role assignment uses DirectoryScopeId "/administrativeUnits/<admin-unit-id>".
```

# 07_Manage_Admin_Roles_Role_Groups_Admin_Units_And_Delegation_Role_Assignable_Groups_Skeleton
Role-assignable groups let you assign Microsoft Entra roles to a protected security group instead of directly to individual users.

Planning table:

| Item | Example | Decision |
|---|---|---|
| Group display name | `RA_Global_Reader` | `<role-assignable-group-name>` |
| Mail nickname | `ra-global-reader` | `<mailnickname>` |
| Assigned role | `Global Reader` | `<role-name>` |
| Members | `auditor@contoso.com` | `<members>` |
| Owners | `pra@contoso.com` | `<owners>` |
| Purpose | Read-only tenant investigation | `<purpose>` |

Create role-assignable group:

```powershell
# Run from an admin workstation.
# Purpose: create a role-assignable security group for Entra role assignment.

Connect-MgGraph -Scopes "Group.ReadWrite.All","RoleManagement.ReadWrite.Directory","Directory.ReadWrite.All","User.Read.All"

$GroupDisplayName = "RA_Global_Reader"
$MailNickname = "ra-global-reader"

$Group = New-MgGroup `
    -DisplayName $GroupDisplayName `
    -Description "Role-assignable group for Global Reader delegation" `
    -MailEnabled:$false `
    -MailNickname $MailNickname `
    -SecurityEnabled:$true `
    -IsAssignableToRole:$true `
    -Visibility "Private"

Get-MgGroup -GroupId $Group.Id |
    Format-List Id,DisplayName,MailEnabled,SecurityEnabled,IsAssignableToRole,Visibility
```

Add members:

```powershell
# Purpose: add a user to a role-assignable group.

$GroupDisplayName = "RA_Global_Reader"
$UserPrincipalName = "<admin-upn>"

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

Assign role to group:

```powershell
# Purpose: assign an Entra role to a role-assignable group.

$GroupDisplayName = "RA_Global_Reader"
$RoleName = "Global Reader"

$Group = Get-MgGroup -Filter "displayName eq '$GroupDisplayName'"
$RoleDefinition = Get-MgRoleManagementDirectoryRoleDefinition -All |
    Where-Object {$_.DisplayName -eq $RoleName}

$Assignment = @{
    "@odata.type" = "#microsoft.graph.unifiedRoleAssignment"
    RoleDefinitionId = $RoleDefinition.Id
    PrincipalId = $Group.Id
    DirectoryScopeId = "/"
}

New-MgRoleManagementDirectoryRoleAssignment -BodyParameter $Assignment
```

Operational note:

```text
Role-assignable groups are protected objects.
Use them for cleaner delegation, but do not treat them like normal security groups.
Restrict ownership and membership changes.
```

# 07_Manage_Admin_Roles_Role_Groups_Admin_Units_And_Delegation_Admin_Units_Skeleton
```text
Portal path:

Microsoft Entra admin center
Identity
Roles and admins
Admin units
```

Administrative unit planning:

| Field | Example | Decision |
|---|---|---|
| Admin unit name | `AU_Atlanta_Users` | `<admin-unit-name>` |
| Description | `Scoped administration for Atlanta users` | `<description>` |
| Scope objects | Users and groups | `<users/groups/devices>` |
| Delegated role | User Administrator | `<role-name>` |
| Delegated admin | `atl.helpdesk@contoso.com` | `<delegated-admin-upn>` |
| Business owner | `atl.manager@contoso.com` | `<owner>` |
| Review cadence | Quarterly | `<review-cadence>` |

Create administrative unit:

```powershell
# Run from an admin workstation.
# Purpose: create an administrative unit for scoped delegation.

$EvidencePath = ".\Evidence\M365-Roles-Delegation"
New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Connect-MgGraph -Scopes "Directory.ReadWrite.All","RoleManagement.ReadWrite.Directory","User.Read.All","Group.ReadWrite.All"

$AdminUnitName = "AU_Atlanta_Users"

$AdminUnit = New-MgDirectoryAdministrativeUnit `
    -DisplayName $AdminUnitName `
    -Description "Scoped administration for Atlanta users"

Get-MgDirectoryAdministrativeUnit -AdministrativeUnitId $AdminUnit.Id |
    Format-List Id,DisplayName,Description |
    Tee-Object "$EvidencePath\admin-unit-created.txt"
```

Add user to administrative unit:

```powershell
# Purpose: add a user object into an administrative unit.

$AdminUnitName = "AU_Atlanta_Users"
$UserPrincipalName = "<user-upn>"

$AdminUnit = Get-MgDirectoryAdministrativeUnit -Filter "displayName eq '$AdminUnitName'"
$User = Get-MgUser -UserId $UserPrincipalName

New-MgDirectoryAdministrativeUnitMemberByRef `
    -AdministrativeUnitId $AdminUnit.Id `
    -BodyParameter @{
        "@odata.id" = "https://graph.microsoft.com/v1.0/directoryObjects/$($User.Id)"
    }

Get-MgDirectoryAdministrativeUnitMember -AdministrativeUnitId $AdminUnit.Id |
    Select-Object Id,AdditionalProperties |
    Format-Table -AutoSize
```

Assign scoped role to administrative unit:

```powershell
# Purpose: assign a Microsoft Entra role scoped to an administrative unit.

$AdminUnitName = "AU_Atlanta_Users"
$DelegatedAdminUpn = "<delegated-admin-upn>"
$RoleName = "User Administrator"

$AdminUnit = Get-MgDirectoryAdministrativeUnit -Filter "displayName eq '$AdminUnitName'"
$DelegatedAdmin = Get-MgUser -UserId $DelegatedAdminUpn

$RoleDefinition = Get-MgRoleManagementDirectoryRoleDefinition -All |
    Where-Object {$_.DisplayName -eq $RoleName}

$Assignment = @{
    "@odata.type" = "#microsoft.graph.unifiedRoleAssignment"
    RoleDefinitionId = $RoleDefinition.Id
    PrincipalId = $DelegatedAdmin.Id
    DirectoryScopeId = "/administrativeUnits/$($AdminUnit.Id)"
}

New-MgRoleManagementDirectoryRoleAssignment -BodyParameter $Assignment

Get-MgRoleManagementDirectoryRoleAssignment -All |
    Where-Object {$_.DirectoryScopeId -eq "/administrativeUnits/$($AdminUnit.Id)"} |
    Format-List Id,PrincipalId,RoleDefinitionId,DirectoryScopeId
```

Remove user from administrative unit:

```powershell
# Purpose: remove a user object from an administrative unit.

$AdminUnitName = "AU_Atlanta_Users"
$UserPrincipalName = "<user-upn>"

$AdminUnit = Get-MgDirectoryAdministrativeUnit -Filter "displayName eq '$AdminUnitName'"
$User = Get-MgUser -UserId $UserPrincipalName

Remove-MgDirectoryAdministrativeUnitMemberByRef `
    -AdministrativeUnitId $AdminUnit.Id `
    -DirectoryObjectId $User.Id
```

Operational note:

```text
Administrative units scope supported directory administration.
They are not a replacement for every workload permission model.
Use admin units for identity delegation, then use workload RBAC where the workload has its own permission system.
```

# 07_Manage_Admin_Roles_Role_Groups_Admin_Units_And_Delegation_Exchange_Role_Groups_Skeleton
```text
Portal path:

Exchange admin center
Roles
Admin roles
```

Exchange role group planning:

| Field | Example | Decision |
|---|---|---|
| Role group name | `EXO_RoleGroup_Atlanta_Recipient_Admins` | `<role-group-name>` |
| Management roles | `Mail Recipients`, `View-Only Recipients` | `<roles>` |
| Members | `atl.exchangeadmin@contoso.com` | `<members>` |
| Recipient scope | Atlanta mailboxes only | `<recipient-scope>` |
| Management scope name | `Atlanta Mailboxes` | `<management-scope-name>` |
| Recipient restriction filter | `CustomAttribute1 -eq 'Atlanta'` | `<recipient-filter>` |
| Owner | `exchangeowner@contoso.com` | `<owner>` |

Connect and export Exchange RBAC baseline:

```powershell
# Run from an admin workstation.
# Purpose: inventory Exchange Online role groups and management role assignments.

$EvidencePath = ".\Evidence\M365-Roles-Delegation"
New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Connect-ExchangeOnline -UserPrincipalName "<admin-upn>"

Get-RoleGroup |
    Sort-Object Name |
    Format-Table Name,ManagedBy,Roles -AutoSize |
    Tee-Object "$EvidencePath\exo-role-groups.txt"

Get-RoleGroup |
    ForEach-Object {
        $RoleGroup = $_
        Get-RoleGroupMember -Identity $RoleGroup.Name |
            Select-Object @{Name="RoleGroup";Expression={$RoleGroup.Name}},Name,PrimarySmtpAddress,RecipientType
    } |
    Export-Csv "$EvidencePath\exo-role-group-members.csv" -NoTypeInformation

Get-ManagementRoleAssignment |
    Select-Object Name,Role,RoleAssigneeName,RecipientWriteScope,CustomRecipientWriteScope,ConfigWriteScope |
    Export-Csv "$EvidencePath\exo-management-role-assignments.csv" -NoTypeInformation
```

Create Exchange role group:

```powershell
# Purpose: create an Exchange role group with selected roles and members.

Connect-ExchangeOnline -UserPrincipalName "<admin-upn>"

New-RoleGroup `
    -Name "EXO_RoleGroup_Recipient_Admins" `
    -Roles "Mail Recipients","View-Only Recipients" `
    -Members "recipientadmin@<custom-domain>" `
    -Description "Delegated Exchange recipient administration"

Get-RoleGroup -Identity "EXO_RoleGroup_Recipient_Admins" |
    Format-List Name,Roles,Members,ManagedBy
```

Create management scope and scoped role group:

```powershell
# Purpose: limit Exchange recipient administration to a scoped recipient set.
# Example uses CustomAttribute1 to scope Atlanta mailboxes.

Connect-ExchangeOnline -UserPrincipalName "<admin-upn>"

New-ManagementScope `
    -Name "Atlanta Mailboxes" `
    -RecipientRestrictionFilter "CustomAttribute1 -eq 'Atlanta'"

New-RoleGroup `
    -Name "EXO_RoleGroup_Atlanta_Recipient_Admins" `
    -Roles "Mail Recipients","View-Only Recipients" `
    -Members "atl.exchangeadmin@<custom-domain>" `
    -CustomRecipientWriteScope "Atlanta Mailboxes" `
    -Description "Scoped Exchange recipient administration for Atlanta mailboxes"

Get-ManagementScope -Identity "Atlanta Mailboxes" |
    Format-List *

Get-RoleGroup -Identity "EXO_RoleGroup_Atlanta_Recipient_Admins" |
    Format-List *
```

Add and remove Exchange role group members:

```powershell
# Add member.
Add-RoleGroupMember `
    -Identity "EXO_RoleGroup_Recipient_Admins" `
    -Member "recipientadmin2@<custom-domain>"

# Remove member.
Remove-RoleGroupMember `
    -Identity "EXO_RoleGroup_Recipient_Admins" `
    -Member "recipientadmin2@<custom-domain>" `
    -Confirm:$false
```

Operational note:

```text
Exchange role groups control Exchange permissions.
Do not assume a Microsoft 365 admin role gives the exact Exchange recipient scope you want.
Use Exchange RBAC scopes when recipient administration must be limited.
```

# 07_Manage_Admin_Roles_Role_Groups_Admin_Units_And_Delegation_PIM_And_Eligible_Access_Skeleton
```text
Portal path:

Microsoft Entra admin center
Identity Governance
Privileged Identity Management
Microsoft Entra roles
```

PIM planning:

| Field | Example | Decision |
|---|---|---|
| Role | Privileged Role Administrator | `<role-name>` |
| Eligible principal | `admin@contoso.com` | `<admin-upn>` |
| Assignment type | Eligible | `<eligible/active>` |
| Activation duration | 2 hours | `<duration>` |
| MFA required | Yes | `<yes-no>` |
| Justification required | Yes | `<yes-no>` |
| Approval required | Yes for highly privileged roles | `<yes-no>` |
| Approver | `securitymanager@contoso.com` | `<approver>` |
| Notification recipients | Security operations | `<notification-recipients>` |
| Review cadence | Monthly | `<review-cadence>` |

Portal checklist:

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Open PIM | Admin Workstation | Entra admin center > Identity Governance > Privileged Identity Management | PIM opens |
| 2 | Open Microsoft Entra roles | Admin Workstation | PIM > Microsoft Entra roles | Role management opens |
| 3 | Review active assignments | Admin Workstation | Assignments > Active | Standing privileged assignments are visible |
| 4 | Review eligible assignments | Admin Workstation | Assignments > Eligible | Eligible assignments are visible |
| 5 | Review role settings | Admin Workstation | Settings > select role | Activation requirements are visible |
| 6 | Configure activation duration | Admin Workstation | Role settings | Activation duration matches policy |
| 7 | Require MFA | Admin Workstation | Role settings | MFA required for activation |
| 8 | Require justification | Admin Workstation | Role settings | Admin must enter reason |
| 9 | Require approval for high privilege roles | Admin Workstation | Role settings | Approver required |
| 10 | Assign eligible role | Admin Workstation | Assignments > Add assignments | Admin receives eligible assignment |
| 11 | Test activation | Delegated Admin Workstation | My roles > Activate | Role activates with controls |
| 12 | Validate activation expiration | Admin Workstation | PIM assignment view | Role expires after configured duration |
| 13 | Export or screenshot evidence | Admin Workstation | Screenshots / notes | PIM settings are documented |

Operational note:

```text
Use PIM for standing privilege reduction.
Break glass accounts are usually handled differently and must remain available during identity control failures.
```

# 07_Manage_Admin_Roles_Role_Groups_Admin_Units_And_Delegation_Delegation_Boundary_Skeleton
Delegation decision table:

| Need | Correct Control |
|---|---|
| Read-only tenant-wide visibility | Global Reader |
| Reset passwords and manage basic user lifecycle | Helpdesk Administrator or User Administrator |
| Manage users only in one region or business unit | Administrative unit scoped User Administrator |
| Manage groups tenant-wide | Groups Administrator |
| Manage license assignment | License Administrator or group-based licensing owner |
| Manage Exchange recipients | Exchange Administrator or Exchange role group |
| Manage only a subset of Exchange recipients | Exchange role group plus management scope |
| Manage SharePoint and OneDrive | SharePoint Administrator |
| Manage Teams policies and settings | Teams Administrator |
| Manage security portal settings | Security Administrator |
| Read security data only | Security Reader |
| Manage compliance and Purview | Compliance Administrator or Purview role group |
| Manage role assignments | Privileged Role Administrator |
| Emergency tenant recovery | Break glass Global Administrator |

Boundary capture:

| Delegation Area | Principal | Role | Scope | Approval | Review Date |
|---|---|---|---|---|---|
| Tenant read-only | `<principal>` | Global Reader | Tenant-wide | `<ticket>` | `<date>` |
| Atlanta helpdesk | `<principal>` | User Administrator | `AU_Atlanta_Users` | `<ticket>` | `<date>` |
| Exchange recipient admin | `<principal>` | Mail Recipients | `Atlanta Mailboxes` | `<ticket>` | `<date>` |
| License admin | `<principal>` | License Administrator | Tenant-wide | `<ticket>` | `<date>` |
| Teams admin | `<principal>` | Teams Administrator | Tenant-wide | `<ticket>` | `<date>` |

Operational note:

```text
Delegation is not just assigning roles.
Delegation means defining who can do what, where they can do it, how long they can do it, and how the access is reviewed.
```

# 07_Manage_Admin_Roles_Role_Groups_Admin_Units_And_Delegation_Access_Review_Export_Skeleton
```powershell
# Run from an admin workstation.
# Purpose: export role, admin unit, group, and Exchange RBAC evidence for access review.

$EvidencePath = ".\Evidence\M365-Roles-Delegation"
New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Connect-MgGraph -Scopes "RoleManagement.Read.Directory","Directory.Read.All","User.Read.All","Group.Read.All"

# Export Entra role definitions.
Get-MgRoleManagementDirectoryRoleDefinition -All |
    Select-Object Id,DisplayName,Description,IsEnabled |
    Sort-Object DisplayName |
    Export-Csv "$EvidencePath\entra-role-definitions-review.csv" -NoTypeInformation

# Export Entra role assignments.
Get-MgRoleManagementDirectoryRoleAssignment -All |
    Select-Object Id,PrincipalId,RoleDefinitionId,DirectoryScopeId,AppScopeId |
    Export-Csv "$EvidencePath\entra-role-assignments-review.csv" -NoTypeInformation

# Export admin units.
Get-MgDirectoryAdministrativeUnit -All |
    Select-Object Id,DisplayName,Description |
    Sort-Object DisplayName |
    Export-Csv "$EvidencePath\admin-units-review.csv" -NoTypeInformation

# Export role-assignable groups.
Get-MgGroup -All -Property Id,DisplayName,MailEnabled,SecurityEnabled,IsAssignableToRole |
    Where-Object {$_.IsAssignableToRole -eq $true} |
    Select-Object Id,DisplayName,MailEnabled,SecurityEnabled,IsAssignableToRole |
    Sort-Object DisplayName |
    Export-Csv "$EvidencePath\role-assignable-groups-review.csv" -NoTypeInformation
```

```powershell
# Exchange RBAC export.

Connect-ExchangeOnline -UserPrincipalName "<admin-upn>"

Get-RoleGroup |
    Sort-Object Name |
    Select-Object Name,ManagedBy,Roles |
    Export-Csv "$EvidencePath\exo-role-groups-review.csv" -NoTypeInformation

Get-RoleGroup |
    ForEach-Object {
        $RoleGroup = $_
        Get-RoleGroupMember -Identity $RoleGroup.Name |
            Select-Object @{Name="RoleGroup";Expression={$RoleGroup.Name}},Name,PrimarySmtpAddress,RecipientType
    } |
    Export-Csv "$EvidencePath\exo-role-group-members-review.csv" -NoTypeInformation

Get-ManagementRoleAssignment |
    Select-Object Name,Role,RoleAssigneeName,RecipientWriteScope,CustomRecipientWriteScope,ConfigWriteScope |
    Export-Csv "$EvidencePath\exo-management-role-assignments-review.csv" -NoTypeInformation

Get-ManagementScope |
    Select-Object Name,RecipientRestrictionFilter,RecipientRoot,Exclusive |
    Export-Csv "$EvidencePath\exo-management-scopes-review.csv" -NoTypeInformation
```

Access review table:

| Review Item | Owner | Evidence File | Finding | Action |
|---|---|---|---|---|
| Global Administrators | `<owner>` | `entra-role-assignments-review.csv` | `<finding>` | `<keep/remove/convert-to-eligible>` |
| Privileged Role Administrators | `<owner>` | `entra-role-assignments-review.csv` | `<finding>` | `<keep/remove/convert-to-eligible>` |
| Admin units | `<owner>` | `admin-units-review.csv` | `<finding>` | `<update-membership/remove>` |
| Role-assignable groups | `<owner>` | `role-assignable-groups-review.csv` | `<finding>` | `<keep/remove>` |
| Exchange role groups | `<owner>` | `exo-role-groups-review.csv` | `<finding>` | `<keep/remove/scope>` |

# 07_Manage_Admin_Roles_Role_Groups_Admin_Units_And_Delegation_Verification_Commands
```powershell
# Microsoft Graph role management verification.

Connect-MgGraph -Scopes "RoleManagement.Read.Directory","Directory.Read.All","User.Read.All","Group.Read.All"

Get-MgContext

Get-MgRoleManagementDirectoryRoleDefinition -All |
    Sort-Object DisplayName |
    Format-Table Id,DisplayName,IsEnabled -AutoSize

Get-MgRoleManagementDirectoryRoleAssignment -All |
    Select-Object Id,PrincipalId,RoleDefinitionId,DirectoryScopeId |
    Format-Table -AutoSize
```

```powershell
# Administrative unit verification.

Get-MgDirectoryAdministrativeUnit -All |
    Sort-Object DisplayName |
    Format-Table Id,DisplayName,Description -AutoSize

$AdminUnitName = "<admin-unit-name>"

$AdminUnit = Get-MgDirectoryAdministrativeUnit -Filter "displayName eq '$AdminUnitName'"

Get-MgDirectoryAdministrativeUnitMember -AdministrativeUnitId $AdminUnit.Id |
    Select-Object Id,AdditionalProperties |
    Format-Table -AutoSize

Get-MgRoleManagementDirectoryRoleAssignment -All |
    Where-Object {$_.DirectoryScopeId -eq "/administrativeUnits/$($AdminUnit.Id)"} |
    Format-Table Id,PrincipalId,RoleDefinitionId,DirectoryScopeId -AutoSize
```

```powershell
# Role-assignable group verification.

Get-MgGroup -All -Property Id,DisplayName,MailEnabled,SecurityEnabled,IsAssignableToRole |
    Where-Object {$_.IsAssignableToRole -eq $true} |
    Sort-Object DisplayName |
    Format-Table Id,DisplayName,SecurityEnabled,IsAssignableToRole -AutoSize

$GroupDisplayName = "<role-assignable-group-name>"
$Group = Get-MgGroup -Filter "displayName eq '$GroupDisplayName'"

Get-MgGroupMember -GroupId $Group.Id |
    Select-Object Id,AdditionalProperties |
    Format-Table -AutoSize
```

```powershell
# Exchange RBAC verification.

Connect-ExchangeOnline -UserPrincipalName "<admin-upn>"

Get-RoleGroup |
    Sort-Object Name |
    Format-Table Name,Roles -AutoSize

Get-RoleGroupMember -Identity "<role-group-name>" |
    Format-Table Name,PrimarySmtpAddress,RecipientType -AutoSize

Get-ManagementRoleAssignment |
    Where-Object {$_.RoleAssigneeName -like "*<role-group-name>*"} |
    Format-Table Name,Role,RoleAssigneeName,RecipientWriteScope,CustomRecipientWriteScope -AutoSize

Get-ManagementScope |
    Sort-Object Name |
    Format-Table Name,RecipientRestrictionFilter,Exclusive -AutoSize
```

```powershell
# Evidence export.

$EvidencePath = ".\Evidence\M365-Roles-Delegation"
New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Get-MgRoleManagementDirectoryRoleDefinition -All |
    Export-Csv "$EvidencePath\final-entra-role-definitions.csv" -NoTypeInformation

Get-MgRoleManagementDirectoryRoleAssignment -All |
    Export-Csv "$EvidencePath\final-entra-role-assignments.csv" -NoTypeInformation

Get-MgDirectoryAdministrativeUnit -All |
    Export-Csv "$EvidencePath\final-admin-units.csv" -NoTypeInformation

Get-MgGroup -All -Property Id,DisplayName,MailEnabled,SecurityEnabled,IsAssignableToRole |
    Where-Object {$_.IsAssignableToRole -eq $true} |
    Export-Csv "$EvidencePath\final-role-assignable-groups.csv" -NoTypeInformation

Get-RoleGroup |
    Export-Csv "$EvidencePath\final-exo-role-groups.csv" -NoTypeInformation

Get-ManagementRoleAssignment |
    Export-Csv "$EvidencePath\final-exo-management-role-assignments.csv" -NoTypeInformation
```

Portal verification:

| Validation Area | Portal Path | Good Result |
|---|---|---|
| Microsoft 365 role assignments | Microsoft 365 admin center > Roles > Role assignments | Role assignments are visible |
| Entra roles | Entra admin center > Identity > Roles and admins | Role assignments are visible |
| Global Administrator list | Roles and admins > Global Administrator | Only approved admins are assigned |
| Privileged Role Administrator list | Roles and admins > Privileged Role Administrator | Only approved admins are assigned |
| Admin units | Entra admin center > Identity > Roles and admins > Admin units | Admin units are visible |
| Admin unit members | Admin unit > Members | Scoped objects are present |
| Scoped admins | Admin unit > Roles and administrators | Delegated admins are visible |
| PIM assignments | Entra admin center > Identity Governance > PIM | Eligible and active assignments are visible |
| Exchange role groups | Exchange admin center > Roles > Admin roles | Role groups and members are visible |
| Exchange management scopes | Exchange PowerShell | Scopes match delegation plan |

# 07_Manage_Admin_Roles_Role_Groups_Admin_Units_And_Delegation_Rollback
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify incorrect Entra role assignment | Admin Workstation | `Get-MgRoleManagementDirectoryRoleAssignment` | Incorrect assignment ID is known |
| 2 | Remove incorrect Entra role assignment | Admin Workstation | `Remove-MgRoleManagementDirectoryRoleAssignment` | Role assignment is removed |
| 3 | Recreate removed Entra assignment if needed | Admin Workstation | `New-MgRoleManagementDirectoryRoleAssignment` | Role assignment is restored |
| 4 | Identify incorrect role-assignable group membership | Admin Workstation | `Get-MgGroupMember` | Incorrect member is identified |
| 5 | Remove member from role-assignable group | Admin Workstation | `Remove-MgGroupMemberByRef` | Member loses inherited role path |
| 6 | Re-add role-assignable group member if needed | Admin Workstation | `New-MgGroupMemberByRef` | Member is restored |
| 7 | Identify incorrect admin unit membership | Admin Workstation | `Get-MgDirectoryAdministrativeUnitMember` | Incorrect scoped object is identified |
| 8 | Remove object from admin unit | Admin Workstation | `Remove-MgDirectoryAdministrativeUnitMemberByRef` | Object is no longer in admin unit |
| 9 | Re-add object to admin unit if needed | Admin Workstation | `New-MgDirectoryAdministrativeUnitMemberByRef` | Object is restored to scope |
| 10 | Identify incorrect admin unit scoped role assignment | Admin Workstation | `Get-MgRoleManagementDirectoryRoleAssignment` | Incorrect scoped role assignment is known |
| 11 | Remove incorrect admin unit scoped role assignment | Admin Workstation | `Remove-MgRoleManagementDirectoryRoleAssignment` | Scoped role is removed |
| 12 | Delete incorrect administrative unit if lab-only | Admin Workstation | `Remove-MgDirectoryAdministrativeUnit` | Admin unit is removed |
| 13 | Identify incorrect Exchange role group member | Admin Workstation | `Get-RoleGroupMember` | Incorrect member is known |
| 14 | Remove Exchange role group member | Admin Workstation | `Remove-RoleGroupMember` | Exchange RBAC access is removed |
| 15 | Re-add Exchange role group member if needed | Admin Workstation | `Add-RoleGroupMember` | Exchange RBAC access is restored |
| 16 | Remove incorrect Exchange role group | Admin Workstation | `Remove-RoleGroup` | Role group is removed |
| 17 | Remove incorrect Exchange management scope | Admin Workstation | `Remove-ManagementScope` | Management scope is removed |
| 18 | Restore PIM setting if changed incorrectly | Admin Workstation | PIM role settings | Activation policy returns to baseline |
| 19 | Validate rollback | Admin Workstation | Verification commands | Role state matches rollback plan |
| 20 | Document rollback | Operator | Notes | Rollback owner, time, and validation are recorded |

Rollback command examples:

```powershell
# Remove Entra role assignment.
Remove-MgRoleManagementDirectoryRoleAssignment `
    -UnifiedRoleAssignmentId "<role-assignment-id>"
```

```powershell
# Remove user from role-assignable group.
$Group = Get-MgGroup -Filter "displayName eq '<role-assignable-group-name>'"
$User = Get-MgUser -UserId "<user-upn>"

Remove-MgGroupMemberByRef `
    -GroupId $Group.Id `
    -DirectoryObjectId $User.Id
```

```powershell
# Remove user from administrative unit.
$AdminUnit = Get-MgDirectoryAdministrativeUnit -Filter "displayName eq '<admin-unit-name>'"
$User = Get-MgUser -UserId "<user-upn>"

Remove-MgDirectoryAdministrativeUnitMemberByRef `
    -AdministrativeUnitId $AdminUnit.Id `
    -DirectoryObjectId $User.Id
```

```powershell
# Remove Exchange role group member.
Remove-RoleGroupMember `
    -Identity "<role-group-name>" `
    -Member "<admin-upn>" `
    -Confirm:$false
```

```powershell
# Remove Exchange role group.
Remove-RoleGroup `
    -Identity "<role-group-name>" `
    -Confirm:$false
```

Rollback note template:

```text
Change made:
Role or group changed:
Principal changed:
Previous value:
New value:
Rollback action:
Rollback owner:
Rollback time:
Validation performed:
Remaining issue:
```

# 07_Manage_Admin_Roles_Role_Groups_Admin_Units_And_Delegation_Failure_Checks
| Symptom | Command | What Usually Broke |
|---|---|---|
| Microsoft 365 Roles page denied | Microsoft 365 admin center > Roles | Admin lacks role management permissions |
| Entra Roles page denied | Entra admin center > Roles and admins | Admin lacks Privileged Role Administrator or required read role |
| Cannot assign admin role | Portal role assignment workflow | Missing Privileged Role Administrator or Global Administrator |
| Graph role assignment fails | `Get-MgContext` | Missing `RoleManagement.ReadWrite.Directory` or admin consent |
| Role definition not found | `Get-MgRoleManagementDirectoryRoleDefinition` | Wrong role display name or role not available in tenant |
| Assignment uses wrong scope | `Get-MgRoleManagementDirectoryRoleAssignment` | DirectoryScopeId is `/` instead of admin unit scope |
| Admin has tenant-wide access unexpectedly | Role assignment export | Role assigned at tenant root instead of admin unit |
| Delegated admin cannot manage scoped user | Admin unit membership and role assignment check | User not in admin unit, wrong role, or propagation delay |
| Delegated admin can manage too much | DirectoryScopeId check | Tenant-wide assignment exists somewhere else |
| Admin unit member missing | `Get-MgDirectoryAdministrativeUnitMember` | User/group/device was not added to the admin unit |
| Admin unit command fails | `New-MgDirectoryAdministrativeUnit` | Missing Graph permissions or unsupported object operation |
| Role-assignable group creation fails | `New-MgGroup -IsAssignableToRole:$true` | Missing permissions or invalid group settings |
| Cannot make existing group role-assignable | Group property review | Role-assignable status must be planned at creation |
| Role-assignable group member add fails | `New-MgGroupMemberByRef` | Missing permissions or protected group restrictions |
| PIM not visible | Entra admin center > Identity Governance | License, role, or portal access issue |
| PIM activation fails | PIM activation page | MFA, approval, justification, or eligibility issue |
| PIM role does not activate quickly | PIM assignment and sign-out/sign-in | Propagation delay or stale token |
| Break glass account not visible | User and role assignment review | Emergency account not created or not assigned |
| Exchange role group cmdlet denied | `Get-RoleGroup` | Missing Exchange permissions or not connected to Exchange Online |
| Exchange role group member cannot perform task | `Get-ManagementRoleAssignment` | Role group lacks required management role |
| Exchange role group has too much access | `Get-ManagementRoleAssignment` | Missing custom management scope |
| Management scope filters wrong recipients | `Get-ManagementScope` and test recipient attributes | Recipient filter is wrong or attributes not populated |
| Role assignment export does not show names | Graph role assignment export | Export returns IDs; resolve principals and role definitions separately |
| Old admin still has access | Role assignment and role group export | Stale assignment, role-assignable group membership, or Exchange role group membership remains |
| Removed admin still appears active | Portal or token behavior | Propagation delay or active session token |
| Wrong tenant modified | `Get-MgContext` and Exchange connection banner | Admin connected to wrong tenant |
| Access review incomplete | Evidence folder review | Role assignments, Exchange role groups, PIM, or admin units were not all exported |

# 07_Manage_Admin_Roles_Role_Groups_Admin_Units_And_Delegation_Related_Labs
| Lab                                                                                        | Relationship                                                                                              |
| ------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------- |
| 01_Create_And_Configure_Microsoft_365_Tenant_Org_Profile_And_Service_Settings              | Tenant and admin center baseline must exist first                                                         |
| 02_Configure_M365_Custom_Domains_DNS_And_Network_Connectivity_Insights                     | Domain administration requires proper delegated admin roles                                               |
| 03_Monitor_Service_Health_Message_Center_Usage_Reports_And_M365_Backup                     | Service health, reports, and backup visibility require correct admin roles                                |
| 04_Create_Manage_And_Bulk_Administer_M365_Users_Contacts_And_Guests                        | User, contact, and guest administration requires User Administrator, Guest Inviter, or scoped admin roles |
| 05_Manage_M365_Groups_Security_Groups_Distribution_Groups_Shared_Mailboxes_And_M365_Groups | Group and shared mailbox administration requires Groups Administrator and Exchange role groups            |
| 06_Manage_M365_Licensing_Service_Plans_Group_Based_Licensing_And_Usage                     | License assignment requires License Administrator and group management delegation                         |
| 08_Troubleshoot_M365_Tenant_User_Group_License_Role_And_Service_Health_Issues              | Troubleshoots role, permission, delegation, and service access failures                                   |
| 04_Compare_Azure_RBAC_Entra_Roles_M365_Admin_Roles_And_Workload_Roles                      | Cloud foundation role comparison baseline                                                                 |
| 05_Configure_Cloud_Admin_Accounts_Break_Glass_And_Privileged_Access_Baseline               | Break glass and privileged access baseline                                                                |
| 07_Configure_Conditional_Access_Baseline_Policies_And_Break_Glass_Exclusions               | Ensures admin access and emergency accounts are not locked out                                            |
| 08_Map_On_Prem_Groups_To_Cloud_Roles_Licensing_Access_And_App_Assignments                  | Maps groups to roles, licenses, apps, and access assignments                                              |