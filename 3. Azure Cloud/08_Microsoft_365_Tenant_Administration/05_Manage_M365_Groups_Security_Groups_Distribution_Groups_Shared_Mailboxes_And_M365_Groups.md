05_Manage_M365_Groups_Security_Groups_Distribution_Groups_Shared_Mailboxes_And_M365_Groups.md
# 05_Manage_M365_Groups_Security_Groups_Distribution_Groups_Shared_Mailboxes_And_M365_Groups

# 05_Manage_M365_Groups_Security_Groups_Distribution_Groups_Shared_Mailboxes_And_M365_Groups_Index
05_Manage_M365_Groups_Security_Groups_Distribution_Groups_Shared_Mailboxes_And_M365_Groups.md
05_Manage_M365_Groups_Security_Groups_Distribution_Groups_Shared_Mailboxes_And_M365_Groups
05_Manage_M365_Groups_Security_Groups_Distribution_Groups_Shared_Mailboxes_And_M365_Groups_Source_Basis
05_Manage_M365_Groups_Security_Groups_Distribution_Groups_Shared_Mailboxes_And_M365_Groups_Mental_Model
05_Manage_M365_Groups_Security_Groups_Distribution_Groups_Shared_Mailboxes_And_M365_Groups_Planning_Table
05_Manage_M365_Groups_Security_Groups_Distribution_Groups_Shared_Mailboxes_And_M365_Groups_Configuration_Checklist
05_Manage_M365_Groups_Security_Groups_Distribution_Groups_Shared_Mailboxes_And_M365_Groups_Microsoft_365_Group_Skeleton
05_Manage_M365_Groups_Security_Groups_Distribution_Groups_Shared_Mailboxes_And_M365_Groups_Security_Group_Skeleton
05_Manage_M365_Groups_Security_Groups_Distribution_Groups_Shared_Mailboxes_And_M365_Groups_Distribution_Group_Skeleton
05_Manage_M365_Groups_Security_Groups_Distribution_Groups_Shared_Mailboxes_And_M365_Groups_Mail_Enabled_Security_Group_Skeleton
05_Manage_M365_Groups_Security_Groups_Distribution_Groups_Shared_Mailboxes_And_M365_Groups_Shared_Mailbox_Skeleton
05_Manage_M365_Groups_Security_Groups_Distribution_Groups_Shared_Mailboxes_And_M365_Groups_Bulk_Admin_Skeleton
05_Manage_M365_Groups_Security_Groups_Distribution_Groups_Shared_Mailboxes_And_M365_Groups_Verification_Commands
05_Manage_M365_Groups_Security_Groups_Distribution_Groups_Shared_Mailboxes_And_M365_Groups_Rollback
05_Manage_M365_Groups_Security_Groups_Distribution_Groups_Shared_Mailboxes_And_M365_Groups_Failure_Checks
05_Manage_M365_Groups_Security_Groups_Distribution_Groups_Shared_Mailboxes_And_M365_Groups_Related_Labs

# 05_Manage_M365_Groups_Security_Groups_Distribution_Groups_Shared_Mailboxes_And_M365_Groups_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Create Microsoft 365 groups | Creating Microsoft 365 groups, adding owners, and adding members |
| Microsoft Learn | Manage Microsoft 365 groups | Updating group settings, membership, email address, and deletion behavior |
| Microsoft Learn | Microsoft Graph PowerShell group cmdlets | Repeatable management of Microsoft 365 groups and security groups |
| Microsoft Learn | Exchange Online PowerShell distribution group cmdlets | Creating and managing distribution groups and mail-enabled security groups |
| Microsoft Learn | Create a shared mailbox | Creating shared mailboxes and assigning Full Access, Send As, and Send on Behalf permissions |
| Microsoft Learn | About shared mailboxes | Shared mailbox behavior, licensing limits, sign-in state, permissions, and storage limits |
| Microsoft Learn | Microsoft 365 admin center Teams and groups area | Portal management for active teams/groups, distribution lists, security groups, and shared mailboxes |
| Microsoft Learn | Microsoft 365 admin roles | Group Administrator, Exchange Administrator, User Administrator, and least-privilege delegation |
| Microsoft 365 operational practice | Access and collaboration design | Standardizes group ownership, membership, mail flow, mailbox delegation, and rollback |

# 05_Manage_M365_Groups_Security_Groups_Distribution_Groups_Shared_Mailboxes_And_M365_Groups_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Microsoft 365 group | Collaboration group that can back Teams, SharePoint site access, mailbox, calendar, Planner, and other Microsoft 365 workloads |
| Security group | Microsoft Entra group used for access control, app assignment, conditional access targeting, and license assignment |
| Mail-enabled security group | Security group that can receive email and also be used for permissions in some workloads |
| Distribution group | Mail-only group used to distribute email to members |
| Dynamic distribution group | Exchange recipient filter that calculates recipients dynamically instead of storing static members |
| Shared mailbox | Mailbox accessed by multiple users without direct mailbox sign-in as the normal access method |
| Group owner | Person responsible for group membership, lifecycle, and operational ownership |
| Group member | User or guest who receives access or email through the group |
| Full Access | Shared mailbox permission allowing a user to open and read mailbox contents |
| Send As | Permission allowing a user to send as the shared mailbox identity |
| Send on Behalf | Permission allowing a user to send on behalf of the shared mailbox |
| Automapping | Outlook behavior that can automatically add a shared mailbox when Full Access is granted |
| Hidden from address lists | Mail-enabled object property that hides it from the global address list |
| Group lifecycle | Creation, ownership, membership review, naming, expiration, deletion, and restore |
| Source of authority | System where the group should be managed, such as cloud, on-prem AD, Exchange Online, or synced directory |
| First rule | Pick the correct object type before creating anything |
| Blunt rule | Do not create a Microsoft 365 group when all you needed was an email distribution list |

# 05_Manage_M365_Groups_Security_Groups_Distribution_Groups_Shared_Mailboxes_And_M365_Groups_Planning_Table
| Item | Example | Decision |
|---|---|---|
| Tenant name | `contoso.onmicrosoft.com` | `<tenant>.onmicrosoft.com` |
| Tenant ID | `00000000-0000-0000-0000-000000000000` | `<tenant-id>` |
| Admin account | `admin@contoso.com` | `<admin-upn>` |
| Group admin role | Groups Administrator | `<group-admin-role>` |
| Exchange admin role | Exchange Administrator | `<exchange-admin-role>` |
| Default group domain | `contoso.com` | `<group-domain>` |
| Microsoft 365 group naming prefix | `M365_` | `<m365-group-prefix>` |
| Security group naming prefix | `SG_` | `<security-group-prefix>` |
| Distribution group naming prefix | `DL_` | `<distribution-group-prefix>` |
| Mail-enabled security group prefix | `MESG_` | `<mail-enabled-security-group-prefix>` |
| Shared mailbox naming prefix | `SMB_` | `<shared-mailbox-prefix>` |
| Group owner standard | Minimum two owners | `<owner-standard>` |
| Membership approval standard | Ticket or owner approval | `<membership-approval-standard>` |
| External sender standard | Block unless approved | `<external-sender-standard>` |
| Guest membership standard | Allowed only with owner approval | `<guest-standard>` |
| Shared mailbox permission standard | Full Access plus Send As only when approved | `<shared-mailbox-permission-standard>` |
| Shared mailbox license threshold | License if archive, litigation hold, or over 50 GB needed | `<license-threshold>` |
| Bulk group CSV path | `.\Input\m365-groups.csv` | `<group-csv>` |
| Bulk membership CSV path | `.\Input\m365-group-members.csv` | `<membership-csv>` |
| Bulk shared mailbox CSV path | `.\Input\shared-mailboxes.csv` | `<shared-mailbox-csv>` |
| Evidence path | `.\Evidence\M365-Groups-SharedMailboxes` | `<evidence-path>` |
| Rollback owner | `identityadmin@contoso.com` | `<rollback-owner>` |

# 05_Manage_M365_Groups_Security_Groups_Distribution_Groups_Shared_Mailboxes_And_M365_Groups_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm admin account | Admin Workstation | Portal account menu | Correct admin UPN is signed in |
| 2 | Confirm correct tenant | Admin Workstation | Portal tenant switcher | Correct Microsoft 365 tenant is active |
| 3 | Confirm group admin role | Admin Workstation | Microsoft 365 admin center > Roles | Admin has group management permissions |
| 4 | Confirm Exchange admin role if managing mail-enabled groups or shared mailboxes | Admin Workstation | Microsoft 365 admin center > Roles | Admin has Exchange permissions |
| 5 | Open groups area | Admin Workstation | Microsoft 365 admin center > Teams and groups | Teams and groups area opens |
| 6 | Review active teams and groups | Admin Workstation | Teams and groups > Active teams and groups | Existing Microsoft 365 groups are visible |
| 7 | Review security groups | Admin Workstation | Teams and groups > Security groups | Existing security groups are visible |
| 8 | Review distribution lists | Admin Workstation | Teams and groups > Distribution lists | Existing distribution groups are visible |
| 9 | Review shared mailboxes | Admin Workstation | Teams and groups > Shared mailboxes | Existing shared mailboxes are visible |
| 10 | Determine correct object type | Operator | Planning table | Object type is chosen before creation |
| 11 | Confirm naming standard | Operator | Notes | Prefix, alias, and display name format are known |
| 12 | Confirm owner standard | Operator | Notes | Group or mailbox owner is documented |
| 13 | Confirm membership approval | Operator | Notes | Membership request has approval |
| 14 | Connect Microsoft Graph PowerShell | Admin Workstation | `Connect-MgGraph -Scopes "Group.ReadWrite.All","User.Read.All","Directory.ReadWrite.All"` | Graph session connects |
| 15 | Confirm Graph context | Admin Workstation | `Get-MgContext` | Tenant ID and scopes are visible |
| 16 | Install Exchange Online module if needed | Admin Workstation | `Install-Module ExchangeOnlineManagement -Scope CurrentUser` | Exchange Online module is installed |
| 17 | Connect Exchange Online | Admin Workstation | `Connect-ExchangeOnline -UserPrincipalName "<admin-upn>"` | Exchange Online session connects |
| 18 | Export existing Microsoft 365 groups | Admin Workstation | `Get-MgGroup -Filter "groupTypes/any(c:c eq 'Unified')"` | Microsoft 365 group inventory is available |
| 19 | Export existing security groups | Admin Workstation | `Get-MgGroup -Filter "securityEnabled eq true"` | Security group inventory is available |
| 20 | Export existing distribution groups | Admin Workstation | `Get-DistributionGroup` | Distribution group inventory is available |
| 21 | Export existing shared mailboxes | Admin Workstation | `Get-EXOMailbox -RecipientTypeDetails SharedMailbox` | Shared mailbox inventory is available |
| 22 | Create Microsoft 365 group if needed | Admin Workstation | `New-MgGroup` with `groupTypes = Unified` | Microsoft 365 group is created |
| 23 | Add Microsoft 365 group owner | Admin Workstation | `New-MgGroupOwnerByRef` | Owner is assigned |
| 24 | Add Microsoft 365 group members | Admin Workstation | `New-MgGroupMemberByRef` | Members are assigned |
| 25 | Create security group if needed | Admin Workstation | `New-MgGroup -SecurityEnabled:$true -MailEnabled:$false` | Security group is created |
| 26 | Add security group members | Admin Workstation | `New-MgGroupMemberByRef` | Security group membership is assigned |
| 27 | Create distribution group if needed | Admin Workstation | `New-DistributionGroup` | Distribution group is created |
| 28 | Add distribution group members | Admin Workstation | `Add-DistributionGroupMember` | Members receive group email |
| 29 | Configure distribution group sender restrictions | Admin Workstation | `Set-DistributionGroup` | External sender and moderation posture is set |
| 30 | Create mail-enabled security group if needed | Admin Workstation | `New-DistributionGroup -Type Security` | Mail-enabled security group is created |
| 31 | Create shared mailbox if needed | Admin Workstation | `New-Mailbox -Shared` | Shared mailbox is created |
| 32 | Add shared mailbox Full Access permissions | Admin Workstation | `Add-MailboxPermission` | Delegate can open mailbox |
| 33 | Add shared mailbox Send As permissions | Admin Workstation | `Add-RecipientPermission` | Delegate can send as mailbox |
| 34 | Add shared mailbox Send on Behalf if required | Admin Workstation | `Set-Mailbox -GrantSendOnBehalfTo` | Delegate can send on behalf |
| 35 | Confirm shared mailbox sign-in state | Admin Workstation | Microsoft 365 admin center / Entra user details | Direct sign-in is blocked unless intentionally allowed |
| 36 | Hide group or mailbox from address lists if required | Admin Workstation | `Set-DistributionGroup` or `Set-Mailbox` | Object is hidden from address lists |
| 37 | Configure external sender behavior | Admin Workstation | `Set-DistributionGroup -RequireSenderAuthenticationEnabled` | External sender behavior matches policy |
| 38 | Configure moderation if needed | Admin Workstation | `Set-DistributionGroup -ModerationEnabled` | Moderation behavior matches requirement |
| 39 | Validate group visibility | Admin Workstation | Portal / PowerShell lookup | Group appears in expected admin surface |
| 40 | Validate group membership | Admin Workstation | Graph or Exchange membership commands | Expected members are present |
| 41 | Validate distribution group mail flow | Test Mailbox | Send test email to group | Members receive email |
| 42 | Validate shared mailbox open access | Outlook / OWA | Open shared mailbox | Delegate can access mailbox |
| 43 | Validate shared mailbox Send As | Outlook / OWA | Send as shared mailbox | Message sends as shared mailbox |
| 44 | Validate shared mailbox Send on Behalf if configured | Outlook / OWA | Send on behalf | Message sends on behalf |
| 45 | Run bulk group creation if approved | Admin Workstation | Import CSV skeleton | Bulk groups are created |
| 46 | Run bulk membership update if approved | Admin Workstation | Import CSV skeleton | Group memberships are updated |
| 47 | Run bulk shared mailbox creation if approved | Admin Workstation | Import CSV skeleton | Shared mailboxes and permissions are created |
| 48 | Export final group inventory | Admin Workstation | Verification export commands | Final inventory is saved |
| 49 | Document ownership and rollback | Operator | Notes | Object owner and rollback path are recorded |
| 50 | Document completion state | Operator | Notes | Groups, memberships, and shared mailbox state are supportable |

# 05_Manage_M365_Groups_Security_Groups_Distribution_Groups_Shared_Mailboxes_And_M365_Groups_Microsoft_365_Group_Skeleton
```text
Portal path:

Microsoft 365 admin center
Teams and groups
Active teams and groups
Add a group
Microsoft 365
```

Microsoft 365 group planning:

| Field | Example | Decision |
|---|---|---|
| Display name | `M365_Project_Alpha` | `<group-display-name>` |
| Mail nickname | `project.alpha` | `<mailnickname>` |
| Primary SMTP | `project.alpha@contoso.com` | `<group-email>` |
| Privacy | Private | `<Private/Public>` |
| Owner 1 | `owner1@contoso.com` | `<owner1-upn>` |
| Owner 2 | `owner2@contoso.com` | `<owner2-upn>` |
| Members | `user1@contoso.com`, `user2@contoso.com` | `<members>` |
| Guest allowed | No | `<yes-no>` |
| Teams needed | No | `<yes-no>` |
| SharePoint site expected | Yes | `<yes-no>` |

Create Microsoft 365 group with Microsoft Graph:

```powershell
# Run from an admin workstation.
# Purpose: create a Microsoft 365 group and assign owners/members.

$EvidencePath = ".\Evidence\M365-Groups-SharedMailboxes"
New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Connect-MgGraph -Scopes "Group.ReadWrite.All","User.Read.All","Directory.ReadWrite.All"

$GroupDisplayName = "M365_Project_Alpha"
$MailNickname = "project.alpha"
$OwnerUpns = @("owner1@<custom-domain>","owner2@<custom-domain>")
$MemberUpns = @("user1@<custom-domain>","user2@<custom-domain>")

$GroupParams = @{
    DisplayName = $GroupDisplayName
    Description = "Microsoft 365 group for Project Alpha collaboration"
    MailEnabled = $true
    MailNickname = $MailNickname
    SecurityEnabled = $false
    GroupTypes = @("Unified")
    Visibility = "Private"
}

$Group = New-MgGroup -BodyParameter $GroupParams

foreach ($OwnerUpn in $OwnerUpns) {
    $Owner = Get-MgUser -UserId $OwnerUpn
    New-MgGroupOwnerByRef `
        -GroupId $Group.Id `
        -BodyParameter @{
            "@odata.id" = "https://graph.microsoft.com/v1.0/directoryObjects/$($Owner.Id)"
        }
}

foreach ($MemberUpn in $MemberUpns) {
    $Member = Get-MgUser -UserId $MemberUpn
    New-MgGroupMemberByRef `
        -GroupId $Group.Id `
        -BodyParameter @{
            "@odata.id" = "https://graph.microsoft.com/v1.0/directoryObjects/$($Member.Id)"
        }
}

Get-MgGroup -GroupId $Group.Id |
    Format-List Id,DisplayName,Mail,MailNickname,GroupTypes,SecurityEnabled,MailEnabled,Visibility |
    Tee-Object "$EvidencePath\m365-group-created.txt"
```

Validate Microsoft 365 group:

```powershell
$Group = Get-MgGroup -Filter "displayName eq 'M365_Project_Alpha'"

Get-MgGroup -GroupId $Group.Id |
    Format-List Id,DisplayName,Mail,MailNickname,GroupTypes,SecurityEnabled,MailEnabled,Visibility

Get-MgGroupOwner -GroupId $Group.Id |
    Select-Object Id,DisplayName,UserPrincipalName |
    Format-Table -AutoSize

Get-MgGroupMember -GroupId $Group.Id |
    Select-Object Id,AdditionalProperties |
    Format-Table -AutoSize
```

Remove member or owner:

```powershell
# Remove a member by object ID.
Remove-MgGroupMemberByRef `
    -GroupId "<group-id>" `
    -DirectoryObjectId "<member-object-id>"

# Remove an owner by object ID.
Remove-MgGroupOwnerByRef `
    -GroupId "<group-id>" `
    -DirectoryObjectId "<owner-object-id>"
```

Operational note:

```text
Use a Microsoft 365 group when the object needs collaboration services.
If the object only needs email distribution, use a distribution group.
If the object only needs access control, use a security group.
```

# 05_Manage_M365_Groups_Security_Groups_Distribution_Groups_Shared_Mailboxes_And_M365_Groups_Security_Group_Skeleton
```text
Portal path:

Microsoft 365 admin center
Teams and groups
Security groups
Add a group
```

Security group planning:

| Field | Example | Decision |
|---|---|---|
| Display name | `SG_App_Finance_Read` | `<security-group-name>` |
| Mail nickname | `sg-app-finance-read` | `<mailnickname>` |
| Purpose | Finance app read-only access | `<purpose>` |
| Owners | `identityadmin@contoso.com` | `<owners>` |
| Members | `finance.user@contoso.com` | `<members>` |
| Used for licensing | No | `<yes-no>` |
| Used for app assignment | Yes | `<yes-no>` |
| Used for Conditional Access | No | `<yes-no>` |
| Dynamic membership | No | `<yes-no>` |

Create security group with Microsoft Graph:

```powershell
# Run from an admin workstation.
# Purpose: create a cloud security group and add members.

Connect-MgGraph -Scopes "Group.ReadWrite.All","User.Read.All","Directory.ReadWrite.All"

$GroupDisplayName = "SG_App_Finance_Read"
$MailNickname = "sg-app-finance-read"
$MemberUpns = @("finance.user1@<custom-domain>","finance.user2@<custom-domain>")

$Group = New-MgGroup `
    -DisplayName $GroupDisplayName `
    -Description "Security group for Finance app read-only access" `
    -MailEnabled:$false `
    -MailNickname $MailNickname `
    -SecurityEnabled:$true

foreach ($MemberUpn in $MemberUpns) {
    $Member = Get-MgUser -UserId $MemberUpn
    New-MgGroupMemberByRef `
        -GroupId $Group.Id `
        -BodyParameter @{
            "@odata.id" = "https://graph.microsoft.com/v1.0/directoryObjects/$($Member.Id)"
        }
}

Get-MgGroup -GroupId $Group.Id |
    Format-List Id,DisplayName,MailEnabled,SecurityEnabled,MailNickname
```

Validate security group:

```powershell
$Group = Get-MgGroup -Filter "displayName eq 'SG_App_Finance_Read'"

Get-MgGroup -GroupId $Group.Id |
    Format-List Id,DisplayName,MailEnabled,SecurityEnabled

Get-MgGroupMember -GroupId $Group.Id |
    Select-Object Id,AdditionalProperties |
    Format-Table -AutoSize
```

Operational note:

```text
Security groups should have a clear access purpose.
Do not use one giant security group for unrelated app access, licensing, Conditional Access, and file permissions.
```

# 05_Manage_M365_Groups_Security_Groups_Distribution_Groups_Shared_Mailboxes_And_M365_Groups_Distribution_Group_Skeleton
```text
Portal path:

Microsoft 365 admin center
Teams and groups
Distribution lists
Add a group
Distribution
```

Distribution group planning:

| Field | Example | Decision |
|---|---|---|
| Display name | `DL_Helpdesk` | `<distribution-group-name>` |
| Alias | `helpdesk` | `<alias>` |
| Primary SMTP | `helpdesk@contoso.com` | `<primary-smtp>` |
| Owners / managed by | `itmanager@contoso.com` | `<managed-by>` |
| Members | `tech1@contoso.com`, `tech2@contoso.com` | `<members>` |
| Accept external senders | No | `<yes-no>` |
| Moderation required | No | `<yes-no>` |
| Hidden from address lists | No | `<yes-no>` |
| Joining restrictions | Closed | `<open/closed/approval>` |

Create distribution group:

```powershell
# Run from an admin workstation.
# Purpose: create a distribution group and add members.

Connect-ExchangeOnline -UserPrincipalName "<admin-upn>"

$GroupName = "DL_Helpdesk"
$Alias = "helpdesk"
$PrimarySmtpAddress = "helpdesk@<custom-domain>"
$Members = @("tech1@<custom-domain>","tech2@<custom-domain>")

New-DistributionGroup `
    -Name $GroupName `
    -Alias $Alias `
    -PrimarySmtpAddress $PrimarySmtpAddress `
    -Type Distribution

foreach ($Member in $Members) {
    Add-DistributionGroupMember `
        -Identity $GroupName `
        -Member $Member
}

Set-DistributionGroup `
    -Identity $GroupName `
    -RequireSenderAuthenticationEnabled $true `
    -HiddenFromAddressListsEnabled $false

Get-DistributionGroup -Identity $GroupName |
    Format-List Name,DisplayName,PrimarySmtpAddress,RequireSenderAuthenticationEnabled,HiddenFromAddressListsEnabled
```

Validate distribution group:

```powershell
Get-DistributionGroup -Identity "DL_Helpdesk" |
    Format-List *

Get-DistributionGroupMember -Identity "DL_Helpdesk" |
    Format-Table Name,PrimarySmtpAddress,RecipientType -AutoSize
```

Allow or block external senders:

```powershell
# Block external senders.
Set-DistributionGroup `
    -Identity "DL_Helpdesk" `
    -RequireSenderAuthenticationEnabled $true

# Allow external senders.
Set-DistributionGroup `
    -Identity "DL_Helpdesk" `
    -RequireSenderAuthenticationEnabled $false
```

Configure moderation:

```powershell
Set-DistributionGroup `
    -Identity "DL_Helpdesk" `
    -ModerationEnabled $true `
    -ModeratedBy "manager@<custom-domain>" `
    -SendModerationNotifications "Always"
```

Operational note:

```text
Distribution groups are for email distribution.
They do not provide the collaboration stack that Microsoft 365 groups provide.
```

# 05_Manage_M365_Groups_Security_Groups_Distribution_Groups_Shared_Mailboxes_And_M365_Groups_Mail_Enabled_Security_Group_Skeleton
```text
Portal path:

Exchange admin center
Recipients
Groups
Mail-enabled security
```

Mail-enabled security group planning:

| Field | Example | Decision |
|---|---|---|
| Display name | `MESG_Invoice_Approvers` | `<mesg-name>` |
| Alias | `invoice.approvers` | `<alias>` |
| Primary SMTP | `invoice.approvers@contoso.com` | `<primary-smtp>` |
| Purpose | Receive mail and assign access | `<purpose>` |
| Members | `approver1@contoso.com` | `<members>` |
| External senders allowed | No | `<yes-no>` |
| Hidden from GAL | No | `<yes-no>` |

Create mail-enabled security group:

```powershell
# Run from an admin workstation.
# Purpose: create a mail-enabled security group and add members.

Connect-ExchangeOnline -UserPrincipalName "<admin-upn>"

$GroupName = "MESG_Invoice_Approvers"
$Alias = "invoice.approvers"
$PrimarySmtpAddress = "invoice.approvers@<custom-domain>"
$Members = @("approver1@<custom-domain>","approver2@<custom-domain>")

New-DistributionGroup `
    -Name $GroupName `
    -Alias $Alias `
    -PrimarySmtpAddress $PrimarySmtpAddress `
    -Type Security

foreach ($Member in $Members) {
    Add-DistributionGroupMember `
        -Identity $GroupName `
        -Member $Member
}

Set-DistributionGroup `
    -Identity $GroupName `
    -RequireSenderAuthenticationEnabled $true `
    -HiddenFromAddressListsEnabled $false

Get-DistributionGroup -Identity $GroupName |
    Format-List Name,DisplayName,PrimarySmtpAddress,RecipientTypeDetails,GroupType,RequireSenderAuthenticationEnabled
```

Validate mail-enabled security group:

```powershell
Get-DistributionGroup -Identity "MESG_Invoice_Approvers" |
    Format-List *

Get-DistributionGroupMember -Identity "MESG_Invoice_Approvers" |
    Format-Table Name,PrimarySmtpAddress,RecipientType -AutoSize
```

Operational note:

```text
Use a mail-enabled security group only when the object must both receive mail and be usable for access control.
If it only needs mail, use a distribution group.
If it only needs access, use a security group.
```

# 05_Manage_M365_Groups_Security_Groups_Distribution_Groups_Shared_Mailboxes_And_M365_Groups_Shared_Mailbox_Skeleton
```text
Portal path:

Microsoft 365 admin center
Teams and groups
Shared mailboxes
Add a shared mailbox
```

Shared mailbox planning:

| Field | Example | Decision |
|---|---|---|
| Shared mailbox name | `SMB_Helpdesk` | `<shared-mailbox-name>` |
| Display name | `Helpdesk` | `<display-name>` |
| Alias | `helpdesk` | `<alias>` |
| Primary SMTP | `helpdesk@contoso.com` | `<shared-mailbox-email>` |
| Full Access delegates | `tech1@contoso.com`, `tech2@contoso.com` | `<full-access-users>` |
| Send As delegates | `tech1@contoso.com` | `<send-as-users>` |
| Send on Behalf delegates | `<none>` | `<send-on-behalf-users>` |
| Automapping | Enabled | `<enabled-disabled>` |
| Hidden from GAL | No | `<yes-no>` |
| License required | No unless over 50 GB, archive, litigation hold, or advanced features needed | `<yes-no>` |
| Owner | `itmanager@contoso.com` | `<owner>` |

Create shared mailbox:

```powershell
# Run from an admin workstation.
# Purpose: create a shared mailbox and assign common permissions.

Connect-ExchangeOnline -UserPrincipalName "<admin-upn>"

$SharedMailboxName = "SMB_Helpdesk"
$Alias = "helpdesk"
$PrimarySmtpAddress = "helpdesk@<custom-domain>"
$FullAccessUsers = @("tech1@<custom-domain>","tech2@<custom-domain>")
$SendAsUsers = @("tech1@<custom-domain>")

New-Mailbox `
    -Shared `
    -Name $SharedMailboxName `
    -DisplayName "Helpdesk" `
    -Alias $Alias `
    -PrimarySmtpAddress $PrimarySmtpAddress

foreach ($User in $FullAccessUsers) {
    Add-MailboxPermission `
        -Identity $PrimarySmtpAddress `
        -User $User `
        -AccessRights FullAccess `
        -InheritanceType All `
        -AutoMapping $true
}

foreach ($User in $SendAsUsers) {
    Add-RecipientPermission `
        -Identity $PrimarySmtpAddress `
        -Trustee $User `
        -AccessRights SendAs `
        -Confirm:$false
}

Get-EXOMailbox -Identity $PrimarySmtpAddress |
    Format-List DisplayName,PrimarySmtpAddress,RecipientTypeDetails,HiddenFromAddressListsEnabled
```

Configure Send on Behalf:

```powershell
# Purpose: grant Send on Behalf.
# Use Send As and Send on Behalf intentionally. Do not grant both unless there is a reason.

Set-Mailbox `
    -Identity "helpdesk@<custom-domain>" `
    -GrantSendOnBehalfTo "manager@<custom-domain>"

Get-Mailbox -Identity "helpdesk@<custom-domain>" |
    Format-List DisplayName,PrimarySmtpAddress,GrantSendOnBehalfTo
```

Validate shared mailbox permissions:

```powershell
$SharedMailbox = "helpdesk@<custom-domain>"

Get-EXOMailbox -Identity $SharedMailbox |
    Format-List DisplayName,PrimarySmtpAddress,RecipientTypeDetails

Get-MailboxPermission -Identity $SharedMailbox |
    Where-Object {
        $_.User -notlike "NT AUTHORITY\SELF" -and
        $_.IsInherited -eq $false
    } |
    Format-Table User,AccessRights,Deny,IsInherited -AutoSize

Get-RecipientPermission -Identity $SharedMailbox |
    Format-Table Trustee,AccessRights,IsInherited -AutoSize

Get-Mailbox -Identity $SharedMailbox |
    Format-List GrantSendOnBehalfTo
```

Remove shared mailbox permissions:

```powershell
$SharedMailbox = "helpdesk@<custom-domain>"
$User = "tech1@<custom-domain>"

Remove-MailboxPermission `
    -Identity $SharedMailbox `
    -User $User `
    -AccessRights FullAccess `
    -InheritanceType All `
    -Confirm:$false

Remove-RecipientPermission `
    -Identity $SharedMailbox `
    -Trustee $User `
    -AccessRights SendAs `
    -Confirm:$false
```

Operational note:

```text
Shared mailboxes should normally have direct sign-in blocked.
Users should access them through delegated permissions.
License the shared mailbox only when feature requirements or storage thresholds require it.
```

# 05_Manage_M365_Groups_Security_Groups_Distribution_Groups_Shared_Mailboxes_And_M365_Groups_Bulk_Admin_Skeleton
Bulk group CSV format:

```csv
GroupType,DisplayName,Alias,PrimarySmtpAddress,Visibility,Description
Microsoft365,M365_Project_Beta,project.beta,project.beta@contoso.com,Private,Project Beta collaboration group
Security,SG_App_HR_Read,sg-app-hr-read,,Private,HR app read access
Distribution,DL_All_Atlanta,all.atlanta,all.atlanta@contoso.com,,Atlanta distribution list
MailEnabledSecurity,MESG_Approvers,approvers,approvers@contoso.com,,Approvers mail-enabled security group
```

Bulk membership CSV format:

```csv
GroupType,GroupIdentity,MemberUpn
Microsoft365,M365_Project_Beta,user1@contoso.com
Microsoft365,M365_Project_Beta,user2@contoso.com
Security,SG_App_HR_Read,user3@contoso.com
Distribution,DL_All_Atlanta,user4@contoso.com
MailEnabledSecurity,MESG_Approvers,user5@contoso.com
```

Bulk create groups:

```powershell
# Run from an admin workstation.
# Purpose: create multiple Microsoft 365, security, distribution, and mail-enabled security groups.

$GroupCsv = ".\Input\m365-groups.csv"
$EvidencePath = ".\Evidence\M365-Groups-SharedMailboxes"

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Connect-MgGraph -Scopes "Group.ReadWrite.All","User.Read.All","Directory.ReadWrite.All"
Connect-ExchangeOnline -UserPrincipalName "<admin-upn>"

$Groups = Import-Csv $GroupCsv
$Results = @()

foreach ($Group in $Groups) {
    try {
        switch ($Group.GroupType) {
            "Microsoft365" {
                $Params = @{
                    DisplayName = $Group.DisplayName
                    Description = $Group.Description
                    MailEnabled = $true
                    MailNickname = $Group.Alias
                    SecurityEnabled = $false
                    GroupTypes = @("Unified")
                    Visibility = $Group.Visibility
                }

                New-MgGroup -BodyParameter $Params | Out-Null
            }

            "Security" {
                New-MgGroup `
                    -DisplayName $Group.DisplayName `
                    -Description $Group.Description `
                    -MailEnabled:$false `
                    -MailNickname $Group.Alias `
                    -SecurityEnabled:$true | Out-Null
            }

            "Distribution" {
                New-DistributionGroup `
                    -Name $Group.DisplayName `
                    -Alias $Group.Alias `
                    -PrimarySmtpAddress $Group.PrimarySmtpAddress `
                    -Type Distribution | Out-Null
            }

            "MailEnabledSecurity" {
                New-DistributionGroup `
                    -Name $Group.DisplayName `
                    -Alias $Group.Alias `
                    -PrimarySmtpAddress $Group.PrimarySmtpAddress `
                    -Type Security | Out-Null
            }
        }

        $Results += [PSCustomObject]@{
            GroupType = $Group.GroupType
            DisplayName = $Group.DisplayName
            Status = "Created"
            Error = ""
        }
    }
    catch {
        $Results += [PSCustomObject]@{
            GroupType = $Group.GroupType
            DisplayName = $Group.DisplayName
            Status = "Failed"
            Error = $_.Exception.Message
        }
    }
}

$Results |
    Export-Csv "$EvidencePath\bulk-group-create-results.csv" -NoTypeInformation

$Results |
    Format-Table -AutoSize
```

Bulk add members:

```powershell
# Purpose: add members to groups from CSV.

$MembershipCsv = ".\Input\m365-group-members.csv"
$EvidencePath = ".\Evidence\M365-Groups-SharedMailboxes"

Connect-MgGraph -Scopes "Group.ReadWrite.All","User.Read.All","Directory.ReadWrite.All"
Connect-ExchangeOnline -UserPrincipalName "<admin-upn>"

$Rows = Import-Csv $MembershipCsv
$Results = @()

foreach ($Row in $Rows) {
    try {
        switch ($Row.GroupType) {
            "Microsoft365" {
                $Group = Get-MgGroup -Filter "displayName eq '$($Row.GroupIdentity)'"
                $Member = Get-MgUser -UserId $Row.MemberUpn

                New-MgGroupMemberByRef `
                    -GroupId $Group.Id `
                    -BodyParameter @{
                        "@odata.id" = "https://graph.microsoft.com/v1.0/directoryObjects/$($Member.Id)"
                    }
            }

            "Security" {
                $Group = Get-MgGroup -Filter "displayName eq '$($Row.GroupIdentity)'"
                $Member = Get-MgUser -UserId $Row.MemberUpn

                New-MgGroupMemberByRef `
                    -GroupId $Group.Id `
                    -BodyParameter @{
                        "@odata.id" = "https://graph.microsoft.com/v1.0/directoryObjects/$($Member.Id)"
                    }
            }

            "Distribution" {
                Add-DistributionGroupMember `
                    -Identity $Row.GroupIdentity `
                    -Member $Row.MemberUpn
            }

            "MailEnabledSecurity" {
                Add-DistributionGroupMember `
                    -Identity $Row.GroupIdentity `
                    -Member $Row.MemberUpn
            }
        }

        $Results += [PSCustomObject]@{
            GroupType = $Row.GroupType
            GroupIdentity = $Row.GroupIdentity
            MemberUpn = $Row.MemberUpn
            Status = "Added"
            Error = ""
        }
    }
    catch {
        $Results += [PSCustomObject]@{
            GroupType = $Row.GroupType
            GroupIdentity = $Row.GroupIdentity
            MemberUpn = $Row.MemberUpn
            Status = "Failed"
            Error = $_.Exception.Message
        }
    }
}

$Results |
    Export-Csv "$EvidencePath\bulk-group-membership-results.csv" -NoTypeInformation

$Results |
    Format-Table -AutoSize
```

Bulk shared mailbox CSV format:

```csv
Name,DisplayName,Alias,PrimarySmtpAddress,FullAccessUsers,SendAsUsers
SMB_Helpdesk,Helpdesk,helpdesk,helpdesk@contoso.com,"tech1@contoso.com;tech2@contoso.com","tech1@contoso.com"
SMB_AP,Accounts Payable,accounts.payable,accounts.payable@contoso.com,"ap1@contoso.com;ap2@contoso.com","ap1@contoso.com"
```

Bulk create shared mailboxes:

```powershell
# Purpose: create shared mailboxes and delegate permissions from CSV.

$MailboxCsv = ".\Input\shared-mailboxes.csv"
$EvidencePath = ".\Evidence\M365-Groups-SharedMailboxes"

Connect-ExchangeOnline -UserPrincipalName "<admin-upn>"

$Mailboxes = Import-Csv $MailboxCsv
$Results = @()

foreach ($Mailbox in $Mailboxes) {
    try {
        New-Mailbox `
            -Shared `
            -Name $Mailbox.Name `
            -DisplayName $Mailbox.DisplayName `
            -Alias $Mailbox.Alias `
            -PrimarySmtpAddress $Mailbox.PrimarySmtpAddress

        $FullAccessUsers = $Mailbox.FullAccessUsers -split ";"
        foreach ($User in $FullAccessUsers) {
            if ($User.Trim()) {
                Add-MailboxPermission `
                    -Identity $Mailbox.PrimarySmtpAddress `
                    -User $User.Trim() `
                    -AccessRights FullAccess `
                    -InheritanceType All `
                    -AutoMapping $true
            }
        }

        $SendAsUsers = $Mailbox.SendAsUsers -split ";"
        foreach ($User in $SendAsUsers) {
            if ($User.Trim()) {
                Add-RecipientPermission `
                    -Identity $Mailbox.PrimarySmtpAddress `
                    -Trustee $User.Trim() `
                    -AccessRights SendAs `
                    -Confirm:$false
            }
        }

        $Results += [PSCustomObject]@{
            SharedMailbox = $Mailbox.PrimarySmtpAddress
            Status = "Created"
            Error = ""
        }
    }
    catch {
        $Results += [PSCustomObject]@{
            SharedMailbox = $Mailbox.PrimarySmtpAddress
            Status = "Failed"
            Error = $_.Exception.Message
        }
    }
}

$Results |
    Export-Csv "$EvidencePath\bulk-shared-mailbox-results.csv" -NoTypeInformation

$Results |
    Format-Table -AutoSize
```

Operational note:

```text
Bulk group and mailbox work must be tested with a small CSV first.
Do not bulk-create production groups without naming, owner, membership, mail flow, and rollback values documented.
```

# 05_Manage_M365_Groups_Security_Groups_Distribution_Groups_Shared_Mailboxes_And_M365_Groups_Verification_Commands
```powershell
# Microsoft Graph group verification.

Connect-MgGraph -Scopes "Group.ReadWrite.All","User.Read.All","Directory.ReadWrite.All"

Get-MgContext

# Microsoft 365 groups.
Get-MgGroup -Filter "groupTypes/any(c:c eq 'Unified')" -All |
    Select-Object Id,DisplayName,Mail,MailNickname,Visibility |
    Sort-Object DisplayName |
    Format-Table -AutoSize

# Security groups.
Get-MgGroup -Filter "securityEnabled eq true and mailEnabled eq false" -All |
    Select-Object Id,DisplayName,MailEnabled,SecurityEnabled,MailNickname |
    Sort-Object DisplayName |
    Format-Table -AutoSize

# Group details by display name.
$Group = Get-MgGroup -Filter "displayName eq '<group-display-name>'"
Get-MgGroup -GroupId $Group.Id | Format-List *
Get-MgGroupMember -GroupId $Group.Id | Format-Table Id,AdditionalProperties -AutoSize
Get-MgGroupOwner -GroupId $Group.Id | Format-Table Id,AdditionalProperties -AutoSize
```

```powershell
# Exchange Online group and shared mailbox verification.

Connect-ExchangeOnline -UserPrincipalName "<admin-upn>"

# Distribution groups and mail-enabled security groups.
Get-DistributionGroup |
    Sort-Object DisplayName |
    Format-Table DisplayName,PrimarySmtpAddress,RecipientTypeDetails,RequireSenderAuthenticationEnabled,HiddenFromAddressListsEnabled -AutoSize

Get-DistributionGroupMember -Identity "<distribution-group-name>" |
    Format-Table Name,PrimarySmtpAddress,RecipientType -AutoSize

# Shared mailboxes.
Get-EXOMailbox -RecipientTypeDetails SharedMailbox |
    Sort-Object DisplayName |
    Format-Table DisplayName,PrimarySmtpAddress,RecipientTypeDetails -AutoSize

# Shared mailbox permissions.
$SharedMailbox = "<shared-mailbox-email>"

Get-MailboxPermission -Identity $SharedMailbox |
    Where-Object {
        $_.User -notlike "NT AUTHORITY\SELF" -and
        $_.IsInherited -eq $false
    } |
    Format-Table User,AccessRights,Deny,IsInherited -AutoSize

Get-RecipientPermission -Identity $SharedMailbox |
    Format-Table Trustee,AccessRights,IsInherited -AutoSize

Get-Mailbox -Identity $SharedMailbox |
    Format-List DisplayName,PrimarySmtpAddress,GrantSendOnBehalfTo,HiddenFromAddressListsEnabled
```

```powershell
# Export evidence.

$EvidencePath = ".\Evidence\M365-Groups-SharedMailboxes"
New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Get-MgGroup -All |
    Select-Object Id,DisplayName,Mail,MailNickname,MailEnabled,SecurityEnabled,GroupTypes,Visibility |
    Export-Csv "$EvidencePath\mg-groups.csv" -NoTypeInformation

Get-DistributionGroup |
    Select-Object DisplayName,PrimarySmtpAddress,RecipientTypeDetails,RequireSenderAuthenticationEnabled,HiddenFromAddressListsEnabled |
    Export-Csv "$EvidencePath\exo-distribution-groups.csv" -NoTypeInformation

Get-EXOMailbox -RecipientTypeDetails SharedMailbox |
    Select-Object DisplayName,PrimarySmtpAddress,RecipientTypeDetails |
    Export-Csv "$EvidencePath\exo-shared-mailboxes.csv" -NoTypeInformation
```

Portal verification:

| Validation Area | Portal Path | Good Result |
|---|---|---|
| Microsoft 365 groups | Teams and groups > Active teams and groups | Microsoft 365 groups are visible |
| Security groups | Teams and groups > Security groups | Security groups are visible |
| Distribution groups | Teams and groups > Distribution lists | Distribution lists are visible |
| Shared mailboxes | Teams and groups > Shared mailboxes | Shared mailboxes are visible |
| Group owners | Group details > Owners | Owners match plan |
| Group members | Group details > Members | Members match plan |
| Shared mailbox delegates | Shared mailbox details > Members / permissions | Delegates match plan |
| Distribution group mail flow | Send test email | Members receive message |
| Shared mailbox access | OWA / Outlook | Delegate can open mailbox |
| Shared mailbox send permissions | OWA / Outlook | Delegate can Send As or Send on Behalf as configured |

# 05_Manage_M365_Groups_Security_Groups_Distribution_Groups_Shared_Mailboxes_And_M365_Groups_Rollback
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify group created by mistake | Admin Workstation | `Get-MgGroup -Filter "displayName eq '<group-name>'"` | Incorrect Graph group is identified |
| 2 | Delete incorrect Microsoft 365 or security group | Admin Workstation | `Remove-MgGroup -GroupId "<group-id>"` | Group is deleted |
| 3 | Restore deleted Microsoft 365 group if needed | Admin Workstation | Restore process / deleted group workflow | Group is restored if supported and within retention |
| 4 | Identify distribution group created by mistake | Admin Workstation | `Get-DistributionGroup -Identity "<group-name>"` | Incorrect distribution group is identified |
| 5 | Delete incorrect distribution group | Admin Workstation | `Remove-DistributionGroup -Identity "<group-name>" -Confirm:$false` | Distribution group is removed |
| 6 | Identify mail-enabled security group created by mistake | Admin Workstation | `Get-DistributionGroup -Identity "<group-name>"` | Incorrect mail-enabled security group is identified |
| 7 | Delete incorrect mail-enabled security group | Admin Workstation | `Remove-DistributionGroup -Identity "<group-name>" -Confirm:$false` | Mail-enabled security group is removed |
| 8 | Remove incorrect Graph group member | Admin Workstation | `Remove-MgGroupMemberByRef` | Incorrect member is removed |
| 9 | Remove incorrect distribution group member | Admin Workstation | `Remove-DistributionGroupMember` | Incorrect member is removed |
| 10 | Re-add member removed by mistake | Admin Workstation | `New-MgGroupMemberByRef` or `Add-DistributionGroupMember` | Member is restored |
| 11 | Identify shared mailbox created by mistake | Admin Workstation | `Get-EXOMailbox -Identity "<shared-mailbox-email>"` | Incorrect shared mailbox is identified |
| 12 | Remove incorrect shared mailbox | Admin Workstation | `Remove-Mailbox -Identity "<shared-mailbox-email>" -Confirm:$false` | Shared mailbox is removed |
| 13 | Remove incorrect Full Access permission | Admin Workstation | `Remove-MailboxPermission` | Delegate can no longer open mailbox |
| 14 | Remove incorrect Send As permission | Admin Workstation | `Remove-RecipientPermission` | Delegate can no longer send as mailbox |
| 15 | Restore prior sender restrictions | Admin Workstation | `Set-DistributionGroup` | External sender or moderation state is restored |
| 16 | Restore address list visibility | Admin Workstation | `Set-DistributionGroup` or `Set-Mailbox` | Hidden or visible state is restored |
| 17 | Validate rollback | Admin Workstation | Verification commands | Object state matches rollback plan |
| 18 | Document rollback | Operator | Notes | Rollback owner, time, and validation are recorded |

Rollback command examples:

```powershell
# Delete Microsoft 365 group or security group.
$Group = Get-MgGroup -Filter "displayName eq '<group-display-name>'"
Remove-MgGroup -GroupId $Group.Id

# Remove distribution group.
Remove-DistributionGroup -Identity "<distribution-group-name>" -Confirm:$false

# Remove shared mailbox.
Remove-Mailbox -Identity "<shared-mailbox-email>" -Confirm:$false

# Remove Full Access.
Remove-MailboxPermission `
    -Identity "<shared-mailbox-email>" `
    -User "<user-upn>" `
    -AccessRights FullAccess `
    -InheritanceType All `
    -Confirm:$false

# Remove Send As.
Remove-RecipientPermission `
    -Identity "<shared-mailbox-email>" `
    -Trustee "<user-upn>" `
    -AccessRights SendAs `
    -Confirm:$false
```

Rollback note template:

```text
Change made:
Object type:
Object name:
Previous value:
New value:
Rollback action:
Rollback owner:
Rollback time:
Validation performed:
Remaining issue:
```

# 05_Manage_M365_Groups_Security_Groups_Distribution_Groups_Shared_Mailboxes_And_M365_Groups_Failure_Checks
| Symptom | Command | What Usually Broke |
|---|---|---|
| Groups page denied | Microsoft 365 admin center > Teams and groups | Missing Groups Administrator, Global Reader, or Global Administrator role |
| Distribution group cmdlet denied | `Get-DistributionGroup` | Missing Exchange Administrator or Exchange role assignment |
| Shared mailbox cmdlet denied | `New-Mailbox -Shared` | Missing Exchange Administrator role |
| Graph group creation fails | `Get-MgContext` | Missing `Group.ReadWrite.All` or `Directory.ReadWrite.All` |
| Group already exists | `Get-MgGroup -Filter "displayName eq '<name>'"` | Duplicate display name, alias, or mail nickname |
| Mail nickname rejected | `New-MgGroup` error | Alias contains invalid characters or is already used |
| Microsoft 365 group not mail-enabled | `Get-MgGroup` | Wrong group type was created |
| Security group appears in wrong place | `Get-MgGroup` | MailEnabled/SecurityEnabled flags do not match intended object type |
| Distribution group not receiving mail | `Get-DistributionGroup`; message trace | Wrong SMTP address, sender restrictions, moderation, or membership issue |
| External senders blocked | `Get-DistributionGroup | FL RequireSenderAuthenticationEnabled` | Group requires authenticated senders |
| External senders allowed by mistake | `Get-DistributionGroup | FL RequireSenderAuthenticationEnabled` | Sender authentication requirement disabled |
| Group hidden from GAL unexpectedly | `Get-DistributionGroup | FL HiddenFromAddressListsEnabled` | Hidden flag enabled |
| Member cannot access resource | `Get-MgGroupMember` | User is not in security group or access assignment is elsewhere |
| Guest member cannot access resource | `Get-MgGroupMember` | Guest exists but resource-specific access is not granted |
| Owner cannot manage group | `Get-MgGroupOwner` | Owner missing or group ownership not assigned |
| Group has no owner | `Get-MgGroupOwner` | Owner was not set during creation |
| Shared mailbox does not appear in Outlook | `Get-MailboxPermission` | Full Access missing, automapping delay, Outlook cache, or wrong mailbox |
| Delegate can open mailbox but cannot send | `Get-RecipientPermission` | Send As missing |
| Delegate sends on behalf instead of as | `Get-Mailbox | FL GrantSendOnBehalfTo`; `Get-RecipientPermission` | Send on Behalf configured instead of Send As |
| Send As still fails after permission added | `Get-RecipientPermission` | Permission propagation delay or Outlook session cache |
| Shared mailbox asks for password | Entra user / mailbox object | User is attempting direct sign-in instead of delegated access |
| Shared mailbox needs license | `Get-EXOMailbox` and mailbox feature review | Mailbox requires archive, litigation hold, advanced features, or storage over threshold |
| Bulk group creation partially fails | Bulk result CSV | CSV row errors, duplicate aliases, bad object type, or permission issue |
| Bulk membership update fails | Bulk result CSV | User not found, group not found, duplicate membership, or wrong group type |
| Distribution group member add fails | `Add-DistributionGroupMember` | Member object not mail-enabled or group identity wrong |
| Shared mailbox bulk permissions fail | Bulk result CSV | Mailbox not provisioned yet, user not found, or permission already exists |
| Wrong tenant modified | `Get-MgContext` and Exchange connection banner | Admin connected to wrong tenant |
| Deleted group cannot be restored | Deleted group workflow | Retention expired, unsupported object type, or conflict with recreated object |

# 05_Manage_M365_Groups_Security_Groups_Distribution_Groups_Shared_Mailboxes_And_M365_Groups_Related_Labs
| Lab | Relationship |
|---|---|
| 01_Create_And_Configure_Microsoft_365_Tenant_Org_Profile_And_Service_Settings | Tenant and admin center baseline must exist first |
| 02_Configure_M365_Custom_Domains_DNS_And_Network_Connectivity_Insights | Verified domain is needed for branded group and mailbox email addresses |
| 03_Monitor_Service_Health_Message_Center_Usage_Reports_And_M365_Backup | Service health and reports help troubleshoot group and mailbox issues |
| 04_Create_Manage_And_Bulk_Administer_M365_Users_Contacts_And_Guests | Users and guests must exist before group membership and mailbox delegation |
| 06_Manage_M365_Licensing_Service_Plans_Group_Based_Licensing_And_Usage | Security groups can be used for group-based licensing |
| 07_Manage_Admin_Roles_Role_Groups_Admin_Units_And_Delegation | Group, Exchange, and role group administration requires correct delegation |
| 08_Troubleshoot_M365_Tenant_User_Group_License_Role_And_Service_Health_Issues | Troubleshoots group, shared mailbox, membership, and mail flow issues |
| 06_Configure_Entra_Users_Groups_Guests_Licenses_And_SSPR_Baseline | Cloud foundation user and group baseline |
| 08_Map_On_Prem_Groups_To_Cloud_Roles_Licensing_Access_And_App_Assignments | Hybrid group mapping and cloud access planning |
| Exchange Online administration workbook | Deep Exchange recipient, mailbox, and mail flow configuration |