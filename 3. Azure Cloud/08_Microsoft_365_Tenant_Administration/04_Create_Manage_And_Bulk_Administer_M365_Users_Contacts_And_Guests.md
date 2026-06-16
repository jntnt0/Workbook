04_Create_Manage_And_Bulk_Administer_M365_Users_Contacts_And_Guests.md
# 04_Create_Manage_And_Bulk_Administer_M365_Users_Contacts_And_Guests

# 04_Create_Manage_And_Bulk_Administer_M365_Users_Contacts_And_Guests_Index
04_Create_Manage_And_Bulk_Administer_M365_Users_Contacts_And_Guests.md
04_Create_Manage_And_Bulk_Administer_M365_Users_Contacts_And_Guests
04_Create_Manage_And_Bulk_Administer_M365_Users_Contacts_And_Guests_Source_Basis
04_Create_Manage_And_Bulk_Administer_M365_Users_Contacts_And_Guests_Mental_Model
04_Create_Manage_And_Bulk_Administer_M365_Users_Contacts_And_Guests_Planning_Table
04_Create_Manage_And_Bulk_Administer_M365_Users_Contacts_And_Guests_Configuration_Checklist
04_Create_Manage_And_Bulk_Administer_M365_Users_Contacts_And_Guests_Single_User_Portal_Skeleton
04_Create_Manage_And_Bulk_Administer_M365_Users_Contacts_And_Guests_Graph_User_Admin_Skeleton
04_Create_Manage_And_Bulk_Administer_M365_Users_Contacts_And_Guests_Bulk_User_CSV_Skeleton
04_Create_Manage_And_Bulk_Administer_M365_Users_Contacts_And_Guests_Contact_Administration_Skeleton
04_Create_Manage_And_Bulk_Administer_M365_Users_Contacts_And_Guests_Guest_User_Skeleton
04_Create_Manage_And_Bulk_Administer_M365_Users_Contacts_And_Guests_User_Lifecycle_Skeleton
04_Create_Manage_And_Bulk_Administer_M365_Users_Contacts_And_Guests_Verification_Commands
04_Create_Manage_And_Bulk_Administer_M365_Users_Contacts_And_Guests_Rollback
04_Create_Manage_And_Bulk_Administer_M365_Users_Contacts_And_Guests_Failure_Checks
04_Create_Manage_And_Bulk_Administer_M365_Users_Contacts_And_Guests_Related_Labs

# 04_Create_Manage_And_Bulk_Administer_M365_Users_Contacts_And_Guests_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Add users and assign licenses in Microsoft 365 admin center | Creating users, assigning sign-in identity, password behavior, and basic license path |
| Microsoft Learn | Microsoft 365 admin center Users area | Active users, contacts, guests, deleted users, filters, and user lifecycle administration |
| Microsoft Learn | Microsoft Graph PowerShell user cmdlets | Repeatable creation, update, deletion, restore, password reset, and reporting for cloud users |
| Microsoft Learn | Microsoft Graph PowerShell invitation cmdlets | Guest invitation and B2B user onboarding |
| Microsoft Learn | Exchange Online PowerShell mail contact cmdlets | Creating and managing mail contacts visible to Exchange address lists |
| Microsoft Learn | Microsoft 365 bulk user import | CSV-based user creation and bulk administration |
| Microsoft Learn | Microsoft Entra user administration | Identity object state, guest accounts, deleted users, and account properties |
| Microsoft Learn | Microsoft 365 admin roles | User Administrator, Global Administrator, Guest Inviter, Exchange Administrator, and least-privilege role planning |
| Microsoft 365 operational practice | User lifecycle and support workflow | Standardizes joiner, mover, leaver, contact, guest, and bulk operations |

# 04_Create_Manage_And_Bulk_Administer_M365_Users_Contacts_And_Guests_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Cloud user | Microsoft Entra user object mastered directly in the cloud tenant |
| Synced user | User object mastered on-premises and synchronized to Microsoft 365 by Entra Connect or Cloud Sync |
| Active user | User account visible under Microsoft 365 admin center > Users > Active users |
| Deleted user | Soft-deleted user object that can usually be restored before permanent deletion |
| Contact | External person represented as a directory/contact object, usually without Microsoft 365 sign-in |
| Mail contact | Exchange Online contact object with an external email address visible in address lists |
| Guest user | External identity invited into the tenant for collaboration |
| Member user | Internal organization user account |
| User principal name | Sign-in name, usually `<user>@<domain>` |
| Mail nickname | Alias used for mail-related object naming |
| Usage location | Country/region property required before license assignment |
| Password profile | Initial password and force-change behavior for cloud users |
| Block sign-in | Disables account authentication without deleting the object |
| Account lifecycle | Joiner, mover, leaver, restore, and cleanup process |
| Bulk administration | CSV or scripted process for creating or updating many accounts consistently |
| Source of authority | System where the object must be edited: cloud, on-prem AD, HR system, or another identity source |
| First rule | Determine whether the user is cloud-only or synced before editing |
| Blunt rule | Do not directly edit synced user properties in Microsoft 365 if on-prem AD owns them |

# 04_Create_Manage_And_Bulk_Administer_M365_Users_Contacts_And_Guests_Planning_Table
| Item | Example | Decision |
|---|---|---|
| Tenant name | `contoso.onmicrosoft.com` | `<tenant>.onmicrosoft.com` |
| Tenant ID | `00000000-0000-0000-0000-000000000000` | `<tenant-id>` |
| Admin account | `admin@contoso.com` | `<admin-upn>` |
| User admin role | User Administrator | `<role-name>` |
| Guest inviter role | Guest Inviter | `<guest-role>` |
| Exchange contact admin role | Exchange Administrator | `<exchange-role>` |
| Default user domain | `contoso.com` | `<default-user-domain>` |
| Initial user password standard | Temporary password, force change | `<password-standard>` |
| Usage location | `US` | `<usage-location>` |
| Default department | `IT` | `<department>` |
| Default office | `HQ` | `<office-location>` |
| Default city/state/country | `Atlanta, GA, US` | `<city-state-country>` |
| Bulk user CSV path | `.\Input\m365-users.csv` | `<bulk-user-csv>` |
| Contact CSV path | `.\Input\m365-contacts.csv` | `<contact-csv>` |
| Guest CSV path | `.\Input\m365-guests.csv` | `<guest-csv>` |
| Evidence path | `.\Evidence\M365-Users-Contacts-Guests` | `<evidence-path>` |
| Naming standard | `first.last@contoso.com` | `<naming-standard>` |
| Display name standard | `First Last` | `<display-name-standard>` |
| Mail nickname standard | `first.last` | `<mailnickname-standard>` |
| Guest invitation redirect URL | `https://myapps.microsoft.com` | `<guest-redirect-url>` |
| Restore window standard | Restore soft-deleted users before permanent deletion | `<restore-standard>` |
| Rollback owner | `identityadmin@contoso.com` | `<rollback-owner>` |

# 04_Create_Manage_And_Bulk_Administer_M365_Users_Contacts_And_Guests_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm admin account | Admin Workstation | Portal account menu | Correct admin UPN is signed in |
| 2 | Confirm correct tenant | Admin Workstation | Portal tenant switcher | Correct Microsoft 365 tenant is active |
| 3 | Confirm required admin role | Admin Workstation | Microsoft 365 admin center > Roles | Admin has user administration permissions |
| 4 | Open Active users | Admin Workstation | Microsoft 365 admin center > Users > Active users | Active users page opens |
| 5 | Review existing users | Admin Workstation | Active users list | Existing users are visible |
| 6 | Confirm whether users are cloud-only or synced | Admin Workstation | Active users > user details | Source of authority is known |
| 7 | Open deleted users | Admin Workstation | Users > Deleted users | Soft-deleted users are visible |
| 8 | Open contacts | Admin Workstation | Users > Contacts | Contact management page opens |
| 9 | Open guest users | Admin Workstation | Users > Guest users or Entra admin center > Users | Guest users are visible |
| 10 | Connect Microsoft Graph PowerShell | Admin Workstation | `Connect-MgGraph -Scopes "User.ReadWrite.All","Directory.ReadWrite.All","Organization.Read.All"` | Graph session connects |
| 11 | Confirm Graph context | Admin Workstation | `Get-MgContext` | Tenant ID and scopes are visible |
| 12 | Export current cloud users | Admin Workstation | `Get-MgUser -All` | User inventory is available |
| 13 | Export current guest users | Admin Workstation | `Get-MgUser -Filter "userType eq 'Guest'" -All` | Guest inventory is available |
| 14 | Export current deleted users | Admin Workstation | `Get-MgDirectoryDeletedItemAsUser -All` | Deleted user inventory is available |
| 15 | Install Exchange Online module if needed | Admin Workstation | `Install-Module ExchangeOnlineManagement -Scope CurrentUser` | Exchange Online module is installed |
| 16 | Connect Exchange Online | Admin Workstation | `Connect-ExchangeOnline -UserPrincipalName "<admin-upn>"` | Exchange Online session connects |
| 17 | Export current mail contacts | Admin Workstation | `Get-MailContact` | Mail contact inventory is available |
| 18 | Create a single cloud user in portal if needed | Admin Workstation | Active users > Add a user | New user account is created |
| 19 | Create a single cloud user with Graph if needed | Admin Workstation | `New-MgUser` | New cloud user object is created |
| 20 | Set user usage location | Admin Workstation | `Update-MgUser -UserId "<user-upn>" -UsageLocation "US"` | Usage location is set |
| 21 | Assign license only if approved | Admin Workstation | Defer to licensing workbook | License assignment is handled in workbook 06 |
| 22 | Reset cloud user password if needed | Admin Workstation | `Update-MgUser -UserId "<user-upn>" -PasswordProfile $PasswordProfile` | Password reset is applied |
| 23 | Block user sign-in if needed | Admin Workstation | `Update-MgUser -UserId "<user-upn>" -AccountEnabled:$false` | User can no longer sign in |
| 24 | Unblock user sign-in if needed | Admin Workstation | `Update-MgUser -UserId "<user-upn>" -AccountEnabled:$true` | User sign-in is restored |
| 25 | Update user profile fields | Admin Workstation | `Update-MgUser` | Department, job title, office, and phone values are updated |
| 26 | Create bulk users from CSV | Admin Workstation | `Import-Csv .\Input\m365-users.csv` | Multiple users are created consistently |
| 27 | Validate bulk-created users | Admin Workstation | `Get-MgUser -UserId "<user-upn>"` | Created users are visible |
| 28 | Create mail contact if needed | Admin Workstation | `New-MailContact` | External contact appears in Exchange Online |
| 29 | Update mail contact if needed | Admin Workstation | `Set-MailContact` | Contact attributes are updated |
| 30 | Hide contact from address lists if required | Admin Workstation | `Set-MailContact -HiddenFromAddressListsEnabled $true` | Contact is hidden from address lists |
| 31 | Create bulk contacts from CSV | Admin Workstation | `Import-Csv .\Input\m365-contacts.csv` | Multiple mail contacts are created |
| 32 | Invite a single guest user | Admin Workstation | `New-MgInvitation` | Guest invitation is sent |
| 33 | Invite guests in bulk from CSV | Admin Workstation | `Import-Csv .\Input\m365-guests.csv` | Guest invitations are sent |
| 34 | Validate guest user state | Admin Workstation | `Get-MgUser -Filter "userType eq 'Guest'"` | Guest user exists |
| 35 | Remove test user if needed | Admin Workstation | `Remove-MgUser -UserId "<user-upn>"` | User is soft-deleted |
| 36 | Restore deleted user if needed | Admin Workstation | `Restore-MgDirectoryDeletedItem -DirectoryObjectId "<deleted-user-id>"` | Deleted user is restored |
| 37 | Remove test contact if needed | Admin Workstation | `Remove-MailContact -Identity "<contact-identity>"` | Contact is removed |
| 38 | Remove test guest if needed | Admin Workstation | `Remove-MgUser -UserId "<guest-user-id>"` | Guest user is removed |
| 39 | Export final user inventory | Admin Workstation | Graph export commands | Final user state is saved |
| 40 | Export final contact inventory | Admin Workstation | Exchange export commands | Final contact state is saved |
| 41 | Export final guest inventory | Admin Workstation | Graph export commands | Final guest state is saved |
| 42 | Document user source of authority | Operator | Notes | Cloud-only vs synced ownership is recorded |
| 43 | Document bulk import results | Operator | Notes / CSV outputs | Success and failure records are recorded |
| 44 | Document support process | Operator | Notes | Help desk knows create/reset/block/restore workflow |
| 45 | Document completion state | Operator | Notes | Users, contacts, guests, and bulk workflows are complete |

# 04_Create_Manage_And_Bulk_Administer_M365_Users_Contacts_And_Guests_Single_User_Portal_Skeleton
```text
Portal path:

Microsoft 365 admin center
Users
Active users
Add a user
```

| Step | Task | Expected Result |
|---:|---|---|
| 1 | Open Active users | User administration page opens |
| 2 | Select Add a user | User creation wizard starts |
| 3 | Enter first name and last name | Display name is generated |
| 4 | Set username and domain | UPN uses intended verified domain |
| 5 | Set password option | Temporary password behavior is selected |
| 6 | Set location | Usage location is configured |
| 7 | Assign license only if approved | License state is intentional |
| 8 | Assign optional admin role only if approved | Least-privilege role is assigned |
| 9 | Review user details | User data is accurate |
| 10 | Finish creation | User appears under Active users |

Portal user creation capture:

| Field | Value |
|---|---|
| First name | `<first-name>` |
| Last name | `<last-name>` |
| Display name | `<display-name>` |
| Username / UPN | `<user-upn>` |
| Domain | `<domain>` |
| Usage location | `<usage-location>` |
| Department | `<department>` |
| Job title | `<job-title>` |
| Office | `<office-location>` |
| Password option | `<auto-generated/custom>` |
| Force password change | `<yes-no>` |
| License assigned | `<yes-no>` |
| Admin role assigned | `<none-or-role>` |

Operational note:

```text
Portal creation is acceptable for one-off users.
Bulk or repeatable user creation should use CSV and Microsoft Graph PowerShell.
```

# 04_Create_Manage_And_Bulk_Administer_M365_Users_Contacts_And_Guests_Graph_User_Admin_Skeleton
```powershell
# Run from an admin workstation.
# Purpose: create and manage a cloud-only Microsoft 365 user with Microsoft Graph PowerShell.

$EvidencePath = ".\Evidence\M365-Users-Contacts-Guests"
New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Connect-MgGraph -Scopes "User.ReadWrite.All","Directory.ReadWrite.All","Organization.Read.All"

Get-MgContext |
    Tee-Object "$EvidencePath\graph-context-users.txt"

$UserPrincipalName = "jane.doe@<custom-domain>"
$MailNickname = "jane.doe"

$PasswordProfile = @{
    Password = "ChangeMe!12345"
    ForceChangePasswordNextSignIn = $true
}

New-MgUser `
    -AccountEnabled:$true `
    -DisplayName "Jane Doe" `
    -GivenName "Jane" `
    -Surname "Doe" `
    -UserPrincipalName $UserPrincipalName `
    -MailNickname $MailNickname `
    -PasswordProfile $PasswordProfile `
    -JobTitle "Support Technician" `
    -Department "IT" `
    -OfficeLocation "HQ" `
    -UsageLocation "US"

Get-MgUser -UserId $UserPrincipalName |
    Format-List Id,DisplayName,UserPrincipalName,UserType,AccountEnabled,UsageLocation,Department,JobTitle,OfficeLocation |
    Tee-Object "$EvidencePath\created-user-jane-doe.txt"
```

Update user properties:

```powershell
# Purpose: update common user profile fields.

$UserPrincipalName = "jane.doe@<custom-domain>"

Update-MgUser `
    -UserId $UserPrincipalName `
    -JobTitle "Systems Administrator" `
    -Department "Infrastructure" `
    -OfficeLocation "HQ" `
    -MobilePhone "+1 555 010 1000" `
    -BusinessPhones @("+1 555 010 2000")

Get-MgUser -UserId $UserPrincipalName |
    Format-List DisplayName,UserPrincipalName,JobTitle,Department,OfficeLocation,MobilePhone,BusinessPhones
```

Reset password:

```powershell
# Purpose: reset a cloud user's password and force change at next sign-in.

$UserPrincipalName = "jane.doe@<custom-domain>"

$PasswordProfile = @{
    Password = "NewTempPassword!12345"
    ForceChangePasswordNextSignIn = $true
}

Update-MgUser `
    -UserId $UserPrincipalName `
    -PasswordProfile $PasswordProfile
```

Block and unblock sign-in:

```powershell
# Purpose: block or unblock a cloud user's sign-in.

$UserPrincipalName = "jane.doe@<custom-domain>"

# Block sign-in.
Update-MgUser -UserId $UserPrincipalName -AccountEnabled:$false

# Verify blocked state.
Get-MgUser -UserId $UserPrincipalName |
    Format-List DisplayName,UserPrincipalName,AccountEnabled

# Unblock sign-in.
Update-MgUser -UserId $UserPrincipalName -AccountEnabled:$true
```

Soft-delete and restore user:

```powershell
# Purpose: remove a test user and restore if needed.

$UserPrincipalName = "jane.doe@<custom-domain>"

# Soft-delete user.
Remove-MgUser -UserId $UserPrincipalName

# List deleted users.
Get-MgDirectoryDeletedItemAsUser -All |
    Select-Object Id,DisplayName,UserPrincipalName,DeletedDateTime |
    Format-Table -AutoSize

# Restore by deleted object ID.
$DeletedUser = Get-MgDirectoryDeletedItemAsUser -All |
    Where-Object {$_.UserPrincipalName -eq $UserPrincipalName}

Restore-MgDirectoryDeletedItem -DirectoryObjectId $DeletedUser.Id
```

# 04_Create_Manage_And_Bulk_Administer_M365_Users_Contacts_And_Guests_Bulk_User_CSV_Skeleton
Bulk user CSV format:

```csv
UserPrincipalName,DisplayName,GivenName,Surname,MailNickname,JobTitle,Department,OfficeLocation,UsageLocation,TemporaryPassword
john.smith@contoso.com,John Smith,John,Smith,john.smith,Help Desk Technician,IT,HQ,US,ChangeMe!12345
sara.jones@contoso.com,Sara Jones,Sara,Jones,sara.jones,Systems Analyst,IT,HQ,US,ChangeMe!12345
```

Bulk create users:

```powershell
# Run from an admin workstation.
# Purpose: create cloud-only Microsoft 365 users from a CSV file.

$CsvPath = ".\Input\m365-users.csv"
$EvidencePath = ".\Evidence\M365-Users-Contacts-Guests"

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Connect-MgGraph -Scopes "User.ReadWrite.All","Directory.ReadWrite.All","Organization.Read.All"

$Users = Import-Csv $CsvPath
$Results = @()

foreach ($User in $Users) {
    try {
        $PasswordProfile = @{
            Password = $User.TemporaryPassword
            ForceChangePasswordNextSignIn = $true
        }

        New-MgUser `
            -AccountEnabled:$true `
            -DisplayName $User.DisplayName `
            -GivenName $User.GivenName `
            -Surname $User.Surname `
            -UserPrincipalName $User.UserPrincipalName `
            -MailNickname $User.MailNickname `
            -PasswordProfile $PasswordProfile `
            -JobTitle $User.JobTitle `
            -Department $User.Department `
            -OfficeLocation $User.OfficeLocation `
            -UsageLocation $User.UsageLocation

        $Results += [PSCustomObject]@{
            UserPrincipalName = $User.UserPrincipalName
            Status = "Created"
            Error = ""
        }
    }
    catch {
        $Results += [PSCustomObject]@{
            UserPrincipalName = $User.UserPrincipalName
            Status = "Failed"
            Error = $_.Exception.Message
        }
    }
}

$Results |
    Export-Csv "$EvidencePath\bulk-user-create-results.csv" -NoTypeInformation

$Results |
    Format-Table -AutoSize
```

Bulk validation:

```powershell
# Purpose: validate users from the same CSV.

$CsvPath = ".\Input\m365-users.csv"
$Users = Import-Csv $CsvPath

foreach ($User in $Users) {
    Get-MgUser -UserId $User.UserPrincipalName -ErrorAction SilentlyContinue |
        Select-Object DisplayName,UserPrincipalName,AccountEnabled,UsageLocation,Department,JobTitle
}
```

Bulk update user attributes:

```powershell
# CSV columns:
# UserPrincipalName,JobTitle,Department,OfficeLocation,UsageLocation

$CsvPath = ".\Input\m365-user-updates.csv"
$Users = Import-Csv $CsvPath

foreach ($User in $Users) {
    Update-MgUser `
        -UserId $User.UserPrincipalName `
        -JobTitle $User.JobTitle `
        -Department $User.Department `
        -OfficeLocation $User.OfficeLocation `
        -UsageLocation $User.UsageLocation
}
```

Operational note:

```text
Bulk creation should be tested with two or three users first.
Do not bulk-create production users until naming, domain, usage location, and password behavior are validated.
License assignment is intentionally deferred to the licensing workbook unless the change is explicitly approved.
```

# 04_Create_Manage_And_Bulk_Administer_M365_Users_Contacts_And_Guests_Contact_Administration_Skeleton
```text
Portal path:

Microsoft 365 admin center
Users
Contacts
```

Mail contacts are usually best administered through Exchange Online PowerShell because they are mail-enabled directory objects.

Connect to Exchange Online:

```powershell
# Run from an admin workstation.
# Purpose: connect to Exchange Online for mail contact administration.

Install-Module ExchangeOnlineManagement -Scope CurrentUser

Connect-ExchangeOnline -UserPrincipalName "<admin-upn>"

Get-MailContact |
    Sort-Object DisplayName |
    Format-Table DisplayName,ExternalEmailAddress,PrimarySmtpAddress,HiddenFromAddressListsEnabled -AutoSize
```

Create single mail contact:

```powershell
# Purpose: create a single external mail contact.

New-MailContact `
    -Name "Vendor Support" `
    -DisplayName "Vendor Support" `
    -ExternalEmailAddress "support@vendor.example"

Get-MailContact -Identity "Vendor Support" |
    Format-List Name,DisplayName,ExternalEmailAddress,PrimarySmtpAddress,HiddenFromAddressListsEnabled
```

Update mail contact:

```powershell
# Purpose: update contact display and address list visibility.

Set-MailContact `
    -Identity "Vendor Support" `
    -DisplayName "Vendor Support Desk" `
    -HiddenFromAddressListsEnabled $false

Get-MailContact -Identity "Vendor Support" |
    Format-List Name,DisplayName,ExternalEmailAddress,HiddenFromAddressListsEnabled
```

Bulk contact CSV format:

```csv
Name,DisplayName,ExternalEmailAddress,HiddenFromAddressListsEnabled
Vendor Support,Vendor Support,support@vendor.example,false
Partner Helpdesk,Partner Helpdesk,helpdesk@partner.example,false
Private Vendor,Private Vendor,private@vendor.example,true
```

Bulk create contacts:

```powershell
# Purpose: create mail contacts from CSV.

$CsvPath = ".\Input\m365-contacts.csv"
$EvidencePath = ".\Evidence\M365-Users-Contacts-Guests"

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Connect-ExchangeOnline -UserPrincipalName "<admin-upn>"

$Contacts = Import-Csv $CsvPath
$Results = @()

foreach ($Contact in $Contacts) {
    try {
        New-MailContact `
            -Name $Contact.Name `
            -DisplayName $Contact.DisplayName `
            -ExternalEmailAddress $Contact.ExternalEmailAddress

        Set-MailContact `
            -Identity $Contact.Name `
            -HiddenFromAddressListsEnabled ([System.Convert]::ToBoolean($Contact.HiddenFromAddressListsEnabled))

        $Results += [PSCustomObject]@{
            Contact = $Contact.Name
            Status = "Created"
            Error = ""
        }
    }
    catch {
        $Results += [PSCustomObject]@{
            Contact = $Contact.Name
            Status = "Failed"
            Error = $_.Exception.Message
        }
    }
}

$Results |
    Export-Csv "$EvidencePath\bulk-contact-create-results.csv" -NoTypeInformation

$Results |
    Format-Table -AutoSize
```

Remove test contact:

```powershell
Remove-MailContact -Identity "Vendor Support" -Confirm:$false
```

# 04_Create_Manage_And_Bulk_Administer_M365_Users_Contacts_And_Guests_Guest_User_Skeleton
```text
Portal paths:

Microsoft 365 admin center
Users
Guest users

Microsoft Entra admin center
Identity
Users
All users
New user
Invite external user
```

Guest planning table:

| Item | Example | Decision |
|---|---|---|
| Guest email | `consultant@example.com` | `<guest-email>` |
| Display name | `External Consultant` | `<guest-display-name>` |
| Invitation message | `You have been invited to collaborate with Contoso.` | `<invite-message>` |
| Redirect URL | `https://myapps.microsoft.com` | `<redirect-url>` |
| Sponsor | `manager@contoso.com` | `<sponsor>` |
| Access purpose | `Project collaboration` | `<access-purpose>` |
| Expiration review date | `2026-09-30` | `<review-date>` |

Invite single guest:

```powershell
# Run from an admin workstation.
# Purpose: invite an external guest user with Microsoft Graph PowerShell.

Connect-MgGraph -Scopes "User.Invite.All","User.ReadWrite.All","Directory.ReadWrite.All"

$GuestEmail = "consultant@example.com"
$RedirectUrl = "https://myapps.microsoft.com"

New-MgInvitation `
    -InvitedUserEmailAddress $GuestEmail `
    -InviteRedirectUrl $RedirectUrl `
    -SendInvitationMessage:$true `
    -InvitedUserDisplayName "External Consultant"

Get-MgUser -Filter "mail eq '$GuestEmail' or userPrincipalName eq '$GuestEmail'" -All |
    Format-Table Id,DisplayName,UserPrincipalName,UserType,AccountEnabled -AutoSize
```

Bulk guest CSV format:

```csv
InvitedUserEmailAddress,InvitedUserDisplayName,InviteRedirectUrl,SendInvitationMessage
consultant1@example.com,Consultant One,https://myapps.microsoft.com,true
consultant2@example.com,Consultant Two,https://myapps.microsoft.com,true
```

Bulk invite guests:

```powershell
# Purpose: invite external guest users from CSV.

$CsvPath = ".\Input\m365-guests.csv"
$EvidencePath = ".\Evidence\M365-Users-Contacts-Guests"

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Connect-MgGraph -Scopes "User.Invite.All","User.ReadWrite.All","Directory.ReadWrite.All"

$Guests = Import-Csv $CsvPath
$Results = @()

foreach ($Guest in $Guests) {
    try {
        New-MgInvitation `
            -InvitedUserEmailAddress $Guest.InvitedUserEmailAddress `
            -InvitedUserDisplayName $Guest.InvitedUserDisplayName `
            -InviteRedirectUrl $Guest.InviteRedirectUrl `
            -SendInvitationMessage:([System.Convert]::ToBoolean($Guest.SendInvitationMessage))

        $Results += [PSCustomObject]@{
            Guest = $Guest.InvitedUserEmailAddress
            Status = "Invited"
            Error = ""
        }
    }
    catch {
        $Results += [PSCustomObject]@{
            Guest = $Guest.InvitedUserEmailAddress
            Status = "Failed"
            Error = $_.Exception.Message
        }
    }
}

$Results |
    Export-Csv "$EvidencePath\bulk-guest-invite-results.csv" -NoTypeInformation

$Results |
    Format-Table -AutoSize
```

Guest inventory:

```powershell
Get-MgUser -Filter "userType eq 'Guest'" -All |
    Select-Object Id,DisplayName,UserPrincipalName,Mail,UserType,AccountEnabled |
    Sort-Object DisplayName |
    Format-Table -AutoSize
```

Remove guest:

```powershell
# Use object ID or UPN shown from guest inventory.
Remove-MgUser -UserId "<guest-user-id>"
```

Operational note:

```text
Guest creation is not the same as permanent user onboarding.
Every guest should have a sponsor, business purpose, and review date.
Group and app access for guests belongs in the groups, access, and app assignment workbooks.
```

# 04_Create_Manage_And_Bulk_Administer_M365_Users_Contacts_And_Guests_User_Lifecycle_Skeleton
Joiner workflow:

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm source of authority | Operator | HR / identity notes | Cloud-only or synced path is known |
| 2 | Confirm username and domain | Operator | Naming standard | UPN is approved |
| 3 | Create user | Admin Workstation | Portal or `New-MgUser` | User object exists |
| 4 | Set usage location | Admin Workstation | `Update-MgUser -UsageLocation "US"` | User is license-ready |
| 5 | Assign license if approved | Admin Workstation | Licensing workbook | Required services are enabled |
| 6 | Add to groups if approved | Admin Workstation | Groups workbook | Access baseline is applied |
| 7 | Validate sign-in | Test Client | Browser sign-in | User can authenticate |
| 8 | Document account | Operator | Notes | Account creation is recorded |

Mover workflow:

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm department/job change | Operator | HR / ticket | Requested change is approved |
| 2 | Update profile fields | Admin Workstation | `Update-MgUser` | User profile reflects new role |
| 3 | Review group membership | Admin Workstation | Groups workbook | Access matches new role |
| 4 | Review licensing need | Admin Workstation | Licensing workbook | License matches service need |
| 5 | Document change | Operator | Notes | Mover action is recorded |

Leaver workflow:

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm termination request | Operator | HR / ticket | Leaver action is approved |
| 2 | Block sign-in | Admin Workstation | `Update-MgUser -UserId "<user-upn>" -AccountEnabled:$false` | User cannot sign in |
| 3 | Reset password if required | Admin Workstation | `Update-MgUser -PasswordProfile $PasswordProfile` | Existing access is disrupted |
| 4 | Revoke sessions if required | Admin Workstation | `Revoke-MgUserSignInSession -UserId "<user-upn>"` | Existing sessions are revoked |
| 5 | Preserve mailbox/data as required | Admin Workstation | Exchange/Purview workflow | Data retention path is followed |
| 6 | Remove or convert access later | Admin Workstation | Groups/licensing workflow | Access cleanup is controlled |
| 7 | Delete user only after approval | Admin Workstation | `Remove-MgUser -UserId "<user-upn>"` | User is soft-deleted |
| 8 | Document leaver action | Operator | Notes | Deprovisioning is recorded |

Leaver command skeleton:

```powershell
# Purpose: emergency cloud user disablement.
# Use only after approval.

Connect-MgGraph -Scopes "User.ReadWrite.All","Directory.ReadWrite.All"

$UserPrincipalName = "<user-upn>"

Update-MgUser -UserId $UserPrincipalName -AccountEnabled:$false

Revoke-MgUserSignInSession -UserId $UserPrincipalName

Get-MgUser -UserId $UserPrincipalName |
    Format-List DisplayName,UserPrincipalName,AccountEnabled
```

# 04_Create_Manage_And_Bulk_Administer_M365_Users_Contacts_And_Guests_Verification_Commands
```powershell
# Microsoft Graph user verification.

Connect-MgGraph -Scopes "User.ReadWrite.All","Directory.ReadWrite.All","Organization.Read.All","User.Invite.All"

Get-MgContext

Get-MgUser -All |
    Select-Object DisplayName,UserPrincipalName,UserType,AccountEnabled,UsageLocation |
    Sort-Object UserPrincipalName |
    Format-Table -AutoSize

Get-MgUser -Filter "userType eq 'Member'" -All |
    Select-Object DisplayName,UserPrincipalName,AccountEnabled,UsageLocation |
    Sort-Object UserPrincipalName |
    Format-Table -AutoSize

Get-MgUser -Filter "userType eq 'Guest'" -All |
    Select-Object DisplayName,UserPrincipalName,Mail,AccountEnabled |
    Sort-Object DisplayName |
    Format-Table -AutoSize

Get-MgDirectoryDeletedItemAsUser -All |
    Select-Object Id,DisplayName,UserPrincipalName,DeletedDateTime |
    Sort-Object DeletedDateTime -Descending |
    Format-Table -AutoSize
```

```powershell
# Exchange Online contact verification.

Connect-ExchangeOnline -UserPrincipalName "<admin-upn>"

Get-MailContact |
    Sort-Object DisplayName |
    Format-Table DisplayName,ExternalEmailAddress,PrimarySmtpAddress,HiddenFromAddressListsEnabled -AutoSize
```

```powershell
# Export evidence.

$EvidencePath = ".\Evidence\M365-Users-Contacts-Guests"
New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Get-MgUser -All |
    Select-Object Id,DisplayName,UserPrincipalName,UserType,AccountEnabled,UsageLocation,Department,JobTitle,OfficeLocation |
    Export-Csv "$EvidencePath\mg-users.csv" -NoTypeInformation

Get-MgUser -Filter "userType eq 'Guest'" -All |
    Select-Object Id,DisplayName,UserPrincipalName,Mail,UserType,AccountEnabled |
    Export-Csv "$EvidencePath\mg-guests.csv" -NoTypeInformation

Get-MgDirectoryDeletedItemAsUser -All |
    Select-Object Id,DisplayName,UserPrincipalName,DeletedDateTime |
    Export-Csv "$EvidencePath\mg-deleted-users.csv" -NoTypeInformation

Get-MailContact |
    Select-Object Name,DisplayName,ExternalEmailAddress,PrimarySmtpAddress,HiddenFromAddressListsEnabled |
    Export-Csv "$EvidencePath\exo-mail-contacts.csv" -NoTypeInformation
```

Portal verification:

| Validation Area | Portal Path | Good Result |
|---|---|---|
| Active users | Users > Active users | Member users are visible |
| Deleted users | Users > Deleted users | Soft-deleted users are visible |
| Contacts | Users > Contacts | Contacts are visible |
| Guest users | Users > Guest users or Entra admin center > Users | Guest users are visible |
| Single user details | Active users > select user | Profile, sign-in, roles, groups, and licenses are visible |
| Block sign-in state | User details | Account enabled/blocked state matches expectation |
| Usage location | User details | Usage location is set |
| Bulk import result | Active users / CSV result | Bulk users were created or errors are documented |
| Contact visibility | Exchange address list / contact page | Mail contact appears as expected |
| Guest invitation | Guest user list / invitation result | Guest account exists |

# 04_Create_Manage_And_Bulk_Administer_M365_Users_Contacts_And_Guests_Rollback
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify user created by mistake | Admin Workstation | `Get-MgUser -UserId "<user-upn>"` | Incorrect user is identified |
| 2 | Soft-delete incorrect user | Admin Workstation | `Remove-MgUser -UserId "<user-upn>"` | User is moved to deleted users |
| 3 | Restore user deleted by mistake | Admin Workstation | `Restore-MgDirectoryDeletedItem -DirectoryObjectId "<deleted-user-id>"` | User is restored |
| 4 | Re-enable user blocked by mistake | Admin Workstation | `Update-MgUser -UserId "<user-upn>" -AccountEnabled:$true` | User sign-in is restored |
| 5 | Disable user enabled by mistake | Admin Workstation | `Update-MgUser -UserId "<user-upn>" -AccountEnabled:$false` | User sign-in is blocked |
| 6 | Restore previous user attributes | Admin Workstation | `Update-MgUser` | Department, title, office, and phone are restored |
| 7 | Remove test bulk users | Admin Workstation | `Import-Csv .\Input\rollback-users.csv` and `Remove-MgUser` | Test users are soft-deleted |
| 8 | Restore bulk users if needed | Admin Workstation | Deleted user restore process | Users are restored from deleted users |
| 9 | Remove contact created by mistake | Admin Workstation | `Remove-MailContact -Identity "<contact-name>" -Confirm:$false` | Contact is removed |
| 10 | Recreate contact deleted by mistake | Admin Workstation | `New-MailContact` | Contact is recreated |
| 11 | Remove guest invited by mistake | Admin Workstation | `Remove-MgUser -UserId "<guest-user-id>"` | Guest user is removed |
| 12 | Document rollback | Operator | Notes | Rollback owner, time, and validation are recorded |

Rollback user CSV format:

```csv
UserPrincipalName
john.smith@contoso.com
sara.jones@contoso.com
```

Bulk remove test users:

```powershell
# Purpose: remove users listed in rollback CSV.

$CsvPath = ".\Input\rollback-users.csv"

Connect-MgGraph -Scopes "User.ReadWrite.All","Directory.ReadWrite.All"

$Users = Import-Csv $CsvPath

foreach ($User in $Users) {
    Remove-MgUser -UserId $User.UserPrincipalName -ErrorAction Continue
}
```

Rollback note template:

```text
Change made:
Object changed:
Previous value:
New value:
Rollback action:
Rollback owner:
Rollback time:
Validation performed:
Remaining issue:
```

# 04_Create_Manage_And_Bulk_Administer_M365_Users_Contacts_And_Guests_Failure_Checks
| Symptom | Command | What Usually Broke |
|---|---|---|
| Active users page denied | Microsoft 365 admin center > Users | Missing User Administrator, Global Reader, or Global Administrator role |
| User creation fails in portal | Active users > Add a user | Missing role, invalid domain, duplicate UPN, or license issue |
| User creation fails in Graph | `New-MgUser` | Missing required field, duplicate UPN, invalid password, or missing Graph permission |
| Graph permission denied | `Get-MgContext` | Missing `User.ReadWrite.All` or `Directory.ReadWrite.All` |
| User already exists | `Get-MgUser -UserId "<user-upn>"` | Duplicate UPN or soft-deleted object conflict |
| User domain not available | `Get-MgDomain` | Custom domain not verified or not present in tenant |
| License assignment fails later | `Get-MgUser -UserId "<user-upn>"` | Usage location missing or license unavailable |
| Password reset fails | `Update-MgUser -PasswordProfile` | User is synced from on-prem AD or permission is insufficient |
| Profile edits do not stick | User details / `Get-MgUser` | User is synced and must be edited on-prem |
| Block sign-in fails | `Update-MgUser -AccountEnabled` | Missing permission or synced source conflict |
| Deleted user restore fails | `Get-MgDirectoryDeletedItemAsUser` | Deleted object not found, hard-deleted, or conflicting object exists |
| Bulk CSV import fails | `Import-Csv <path>` | Bad headers, bad encoding, missing required fields, or invalid values |
| Bulk users partially created | Result CSV | Some rows passed and some failed; errors must be reviewed row by row |
| Temporary password rejected | `New-MgUser` error | Password does not meet tenant password policy |
| Mail contact creation fails | `New-MailContact` | Exchange Online session missing, duplicate contact, invalid SMTP address |
| Contact not visible in address list | `Get-MailContact` | Hidden from address lists, address book delay, or wrong object type |
| Contact visible when it should be hidden | `Get-MailContact | FL HiddenFromAddressListsEnabled` | Hidden flag not set |
| Guest invitation fails | `New-MgInvitation` | Guest invitations restricted, missing role, invalid email, or external collaboration policy |
| Guest exists but cannot access resource | `Get-MgUser -Filter "userType eq 'Guest'"` | Guest exists but group/app/site access was not granted |
| Guest receives no invitation | Invitation result / mail trace | Invitation email blocked, suppressed, or sent to wrong address |
| Guest duplicate exists | `Get-MgUser -Filter "userType eq 'Guest'"` | Existing guest object or alternate external identity |
| Exchange Online connection fails | `Connect-ExchangeOnline` | Module missing, MFA/browser issue, role issue, or network issue |
| Microsoft Graph module missing | `Get-Module -ListAvailable Microsoft.Graph` | Module not installed |
| Wrong tenant modified | `Get-MgContext` | Admin connected to wrong tenant |
| Bulk rollback misses users | Rollback CSV | CSV does not include all created users |

# 04_Create_Manage_And_Bulk_Administer_M365_Users_Contacts_And_Guests_Related_Labs
| Lab                                                                                        | Relationship                                                                      |
| ------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------- |
| 01_Create_And_Configure_Microsoft_365_Tenant_Org_Profile_And_Service_Settings              | Tenant baseline and admin center access must exist first                          |
| 02_Configure_M365_Custom_Domains_DNS_And_Network_Connectivity_Insights                     | Custom domain must exist before using branded UPNs and email addresses            |
| 03_Monitor_Service_Health_Message_Center_Usage_Reports_And_M365_Backup                     | Service health and usage reports support user troubleshooting and adoption review |
| 05_Manage_M365_Groups_Security_Groups_Distribution_Groups_Shared_Mailboxes_And_M365_Groups | User, guest, and contact objects are assigned to groups and mailbox access        |
| 06_Manage_M365_Licensing_Service_Plans_Group_Based_Licensing_And_Usage                     | Users require licensing and service plan assignment after creation                |
| 07_Manage_Admin_Roles_Role_Groups_Admin_Units_And_Delegation                               | User administrators, guest inviters, and Exchange admins require correct roles    |
| 08_Troubleshoot_M365_Tenant_User_Group_License_Role_And_Service_Health_Issues              | Troubleshoots user, guest, contact, role, and license failures                    |
| 06_Configure_Entra_Users_Groups_Guests_Licenses_And_SSPR_Baseline                          | Cloud foundation user and guest identity baseline                                 |
| 03_Prepare_Identity_Synchronization_With_IdFix_And_AD_Object_Cleanup                       | Hybrid identity source of authority and sync cleanup                              |
| 04_Configure_Entra_Connect_Sync_And_Cloud_Sync_Baseline                                    | Synced user creation and management path                                          |