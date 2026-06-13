14_Configure_DHCP_Policies_User_Classes_And_Vendor_Classes.md
# 14_Configure_DHCP_Policies_User_Classes_And_Vendor_Classes

# Configure_DHCP_Policies_User_Classes_And_Vendor_Classes_Index
14_Configure_DHCP_Policies_User_Classes_And_Vendor_Classes.md
Configure_DHCP_Policies_User_Classes_And_Vendor_Classes
Configure_DHCP_Policies_User_Classes_And_Vendor_Classes_Source_Basis
Configure_DHCP_Policies_User_Classes_And_Vendor_Classes_Mental_Model
Configure_DHCP_Policies_User_Classes_And_Vendor_Classes_Planning_Table
Configure_DHCP_Policies_User_Classes_And_Vendor_Classes_Configuration_Checklist
Configure_DHCP_Policies_User_Classes_And_Vendor_Classes_Precheck_Skeleton
Configure_DHCP_Policies_User_Classes_And_Vendor_Classes_Create_User_Class_Skeleton
Configure_DHCP_Policies_User_Classes_And_Vendor_Classes_Create_Vendor_Class_Skeleton
Configure_DHCP_Policies_User_Classes_And_Vendor_Classes_Create_Scope_Level_User_Class_Policy_Skeleton
Configure_DHCP_Policies_User_Classes_And_Vendor_Classes_Create_Scope_Level_Vendor_Class_Policy_Skeleton
Configure_DHCP_Policies_User_Classes_And_Vendor_Classes_Add_Policy_IP_Range_Skeleton
Configure_DHCP_Policies_User_Classes_And_Vendor_Classes_Set_Policy_Options_Skeleton
Configure_DHCP_Policies_User_Classes_And_Vendor_Classes_Enable_And_Order_Policies_Skeleton
Configure_DHCP_Policies_User_Classes_And_Vendor_Classes_Client_Test_Skeleton
Configure_DHCP_Policies_User_Classes_And_Vendor_Classes_Verification_Commands
Configure_DHCP_Policies_User_Classes_And_Vendor_Classes_Rollback
Configure_DHCP_Policies_User_Classes_And_Vendor_Classes_Failure_Checks
Configure_DHCP_Policies_User_Classes_And_Vendor_Classes_Related_Labs

# Configure_DHCP_Policies_User_Classes_And_Vendor_Classes_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Add-DhcpServerv4Class | Creating IPv4 user classes and vendor classes |
| Microsoft Learn | Get-DhcpServerv4Class | Validating existing user and vendor class definitions |
| Microsoft Learn | Remove-DhcpServerv4Class | Removing incorrect user or vendor class definitions |
| Microsoft Learn | Add-DhcpServerv4Policy | Creating DHCPv4 policies at server or scope level |
| Microsoft Learn | Set-DhcpServerv4Policy | Modifying policy conditions, processing order, lease duration, name, and enabled state |
| Microsoft Learn | Get-DhcpServerv4Policy | Validating server-level and scope-level policies |
| Microsoft Learn | Remove-DhcpServerv4Policy | Removing incorrect or lab policy objects |
| Microsoft Learn | Add-DhcpServerv4PolicyIPRange | Assigning a policy-specific address range inside a scope |
| Microsoft Learn | Get-DhcpServerv4PolicyIPRange | Validating policy-specific address ranges |
| Microsoft Learn | Set-DhcpServerv4OptionValue | Setting DHCP options at server, scope, reservation, user-class, vendor-class, or policy level |
| Microsoft Learn | Get-DhcpServerv4OptionValue | Verifying effective option values |
| Windows client operational practice | `ipconfig /setclassid`, `ipconfig /showclassid`, `ipconfig /renew` | Testing user class behavior from a Windows DHCP client |

# Configure_DHCP_Policies_User_Classes_And_Vendor_Classes_Mental_Model
| Concept | Operational Meaning |
|---|---|
| DHCP policy | Rule evaluated by DHCP Server to treat matching clients differently |
| Server-level policy | Policy evaluated across the DHCP server service |
| Scope-level policy | Policy evaluated inside one IPv4 scope |
| Processing order | Order in which policies are evaluated; lower order processes earlier |
| Policy condition | Match logic based on MAC, client ID, FQDN, relay agent info, user class, vendor class, or similar request data |
| Condition operator | `AND` or `OR` relationship between multiple condition types |
| Comparator | `EQ` or `NE` inside a condition value, meaning equals or not equals |
| User class | Class value intentionally sent by the DHCP client, often for lab, department, or special client grouping |
| Vendor class | Vendor identifier sent by the DHCP client, commonly used by phones, printers, PXE devices, APs, or vendor appliances |
| Class definition | Server-side object that defines a user or vendor class before policy matching can use it |
| Policy IP range | Address range inside a scope reserved for clients that match a policy |
| Policy options | DHCP options delivered only to clients that match a policy |
| Vendor option | Option data delivered only to matching vendor-class clients, often option 43 for vendor-specific data |
| User class client test | Windows clients can be forced into a user class with `ipconfig /setclassid` |
| Vendor class caveat | You usually cannot fake a real vendor class from normal Windows client settings unless the client/device actually sends that vendor class |
| First rule | Build the class first, build the policy disabled, attach range/options, verify, then enable policy |

# Configure_DHCP_Policies_User_Classes_And_Vendor_Classes_Planning_Table
| Item | Example | Decision |
|---|---|---|
| DHCP server | `DHCP1` | `<dhcp-server>` |
| Scope ID | `10.10.20.0` | `<scope-id>` |
| Scope subnet | `10.10.20.0/24` | `<scope-subnet>` |
| Standard scope range | `10.10.20.50-10.10.20.199` | `<standard-range>` |
| Policy address range | `10.10.20.200-10.10.20.225` | `<policy-range>` |
| User class name | `LabComputers` | `<user-class-name>` |
| User class data | `LabComputers` | `<user-class-data>` |
| User class description | `Lab computer class for DHCP policy testing` | `<user-class-description>` |
| Vendor class name | `CorpPhones` | `<vendor-class-name>` |
| Vendor class data | `MS-UC-Client` | `<vendor-class-data>` |
| Vendor class description | `VoIP phone or UC client class` | `<vendor-class-description>` |
| User policy name | `POLICY-LabComputers` | `<user-policy-name>` |
| Vendor policy name | `POLICY-CorpPhones` | `<vendor-policy-name>` |
| Policy condition | `OR` | `<policy-condition>` |
| User class match | `EQ,LabComputers` | `<user-class-match>` |
| Vendor class match | `EQ,MS-UC-Client` | `<vendor-class-match>` |
| Policy processing order | `1`, `2` | `<processing-order>` |
| Policy lease duration | `08:00:00` | `<policy-lease-duration>` |
| Policy gateway | `10.10.20.1` | `<policy-router>` |
| Policy DNS servers | `10.10.10.10`, `10.10.10.11` | `<policy-dns-servers>` |
| Policy DNS suffix | `corp.local` | `<policy-dns-domain>` |
| Vendor-specific option ID | `43` | `<vendor-option-id>` |
| Vendor-specific option value | Vendor-defined hex/string payload | `<vendor-option-value>` |
| Test client | `WIN11-01` | `<test-client>` |
| Test client interface | `Ethernet` | `<client-interface>` |
| Evidence path | `C:\DHCPPrep\dhcp-policies-classes` | `<evidence-path>` |

# Configure_DHCP_Policies_User_Classes_And_Vendor_Classes_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Create evidence folder | DHCP server | `New-Item -ItemType Directory -Force -Path C:\DHCPPrep\dhcp-policies-classes` | Evidence path exists |
| 2 | Import DHCP module | DHCP server | `Import-Module DhcpServer` | DHCP cmdlets are available |
| 3 | Confirm DHCP service | DHCP server | `Get-Service DHCPServer` | Service is running |
| 4 | Confirm server authorization | DHCP server or DC | `Get-DhcpServerInDC` | Expected DHCP server is authorized |
| 5 | Confirm scope exists | DHCP server | `Get-DhcpServerv4Scope -ComputerName <dhcp-server> -ScopeId <scope-id>` | Scope exists |
| 6 | Confirm current classes | DHCP server | `Get-DhcpServerv4Class -ComputerName <dhcp-server>` | Existing user/vendor classes are known |
| 7 | Create user class | DHCP server | `Add-DhcpServerv4Class -ComputerName <dhcp-server> -Name <user-class-name> -Type User -Data <user-class-data>` | User class exists |
| 8 | Create vendor class | DHCP server | `Add-DhcpServerv4Class -ComputerName <dhcp-server> -Name <vendor-class-name> -Type Vendor -Data <vendor-class-data>` | Vendor class exists |
| 9 | Create user-class policy disabled | DHCP server | `Add-DhcpServerv4Policy -ComputerName <dhcp-server> -ScopeId <scope-id> -Name <user-policy-name> -Condition OR -UserClass EQ,<user-class-data> -Enabled $false` | Policy exists but is not active |
| 10 | Create vendor-class policy disabled | DHCP server | `Add-DhcpServerv4Policy -ComputerName <dhcp-server> -ScopeId <scope-id> -Name <vendor-policy-name> -Condition OR -VendorClass EQ,<vendor-class-data> -Enabled $false` | Policy exists but is not active |
| 11 | Add policy IP range if required | DHCP server | `Add-DhcpServerv4PolicyIPRange -ComputerName <dhcp-server> -ScopeId <scope-id> -Name <policy-name> -StartRange <start-ip> -EndRange <end-ip>` | Matching clients receive addresses from policy range |
| 12 | Set user policy options | DHCP server | `Set-DhcpServerv4OptionValue -ComputerName <dhcp-server> -ScopeId <scope-id> -PolicyName <user-policy-name> -Router <router> -DnsServer <dns-servers> -DnsDomain <dns-domain>` | Matching user-class clients receive policy options |
| 13 | Set vendor policy options | DHCP server | `Set-DhcpServerv4OptionValue -ComputerName <dhcp-server> -ScopeId <scope-id> -PolicyName <vendor-policy-name> -OptionId <vendor-option-id> -Value <vendor-option-value>` | Matching vendor-class clients receive vendor-specific option data |
| 14 | Set policy processing order | DHCP server | `Set-DhcpServerv4Policy -ComputerName <dhcp-server> -ScopeId <scope-id> -Name <policy-name> -ProcessingOrder <order>` | Policy evaluates in intended order |
| 15 | Enable policy | DHCP server | `Set-DhcpServerv4Policy -ComputerName <dhcp-server> -ScopeId <scope-id> -Name <policy-name> -Enabled $true` | Policy is active |
| 16 | Validate policies | DHCP server | `Get-DhcpServerv4Policy -ComputerName <dhcp-server> -ScopeId <scope-id>` | Policies show correct state and order |
| 17 | Validate policy IP ranges | DHCP server | `Get-DhcpServerv4PolicyIPRange -ComputerName <dhcp-server> -ScopeId <scope-id> -Name <policy-name>` | Policy ranges match plan |
| 18 | Validate policy options | DHCP server | `Get-DhcpServerv4OptionValue -ComputerName <dhcp-server> -ScopeId <scope-id> -PolicyName <policy-name>` | Policy options match plan |
| 19 | Configure test client user class | Client | `ipconfig /setclassid "<client-interface>" <user-class-data>` | Client sends user class |
| 20 | Renew client lease | Client | `ipconfig /release`; `ipconfig /renew`; `ipconfig /all` | Client receives matching policy lease/options |
| 21 | Validate server lease | DHCP server | `Get-DhcpServerv4Lease -ComputerName <dhcp-server> -ScopeId <scope-id> -ClientId <client-id>` | Lease appears in expected scope/range |
| 22 | Clear test client user class after test | Client | `ipconfig /setclassid "<client-interface>"` | Client no longer sends test user class |

# Configure_DHCP_Policies_User_Classes_And_Vendor_Classes_Precheck_Skeleton
~~~powershell
# Run on DHCP server or management host with RSAT DHCP tools.

$DhcpServer = "DHCP1"
$ScopeId = "10.10.20.0"
$EvidencePath = "C:\DHCPPrep\dhcp-policies-classes"

New-Item -ItemType Directory -Force -Path $EvidencePath

Import-Module DhcpServer

# Service and authorization checks.
Get-Service DHCPServer |
  Tee-Object "$EvidencePath\precheck-dhcp-service.txt"

Get-DhcpServerInDC |
  Tee-Object "$EvidencePath\precheck-authorized-dhcp-servers.txt"

# Scope baseline.
Get-DhcpServerv4Scope `
  -ComputerName $DhcpServer `
  -ScopeId $ScopeId |
  Format-List * |
  Tee-Object "$EvidencePath\precheck-scope-details.txt"

Get-DhcpServerv4ScopeStatistics `
  -ComputerName $DhcpServer `
  -ScopeId $ScopeId `
  -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\precheck-scope-statistics.txt"

# Existing classes.
Get-DhcpServerv4Class `
  -ComputerName $DhcpServer |
  Tee-Object "$EvidencePath\precheck-existing-classes.txt"

# Existing policies.
Get-DhcpServerv4Policy `
  -ComputerName $DhcpServer |
  Tee-Object "$EvidencePath\precheck-server-level-policies.txt"

Get-DhcpServerv4Policy `
  -ComputerName $DhcpServer `
  -ScopeId $ScopeId |
  Tee-Object "$EvidencePath\precheck-scope-level-policies.txt"

# Existing policy IP ranges.
Get-DhcpServerv4PolicyIPRange `
  -ComputerName $DhcpServer `
  -ScopeId $ScopeId `
  -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\precheck-policy-ip-ranges.txt"

# Existing scope options.
Get-DhcpServerv4OptionValue `
  -ComputerName $DhcpServer `
  -ScopeId $ScopeId |
  Tee-Object "$EvidencePath\precheck-scope-options.txt"

# Existing leases.
Get-DhcpServerv4Lease `
  -ComputerName $DhcpServer `
  -ScopeId $ScopeId `
  -AllLeases |
  Tee-Object "$EvidencePath\precheck-scope-leases.txt"
~~~

# Configure_DHCP_Policies_User_Classes_And_Vendor_Classes_Create_User_Class_Skeleton
~~~powershell
# Run on DHCP server or management host with RSAT DHCP tools.
# User classes are useful for lab clients, departments, test devices, or special handling.

$DhcpServer = "DHCP1"
$UserClassName = "LabComputers"
$UserClassData = "LabComputers"
$UserClassDescription = "Lab computer class for DHCP policy testing"

Import-Module DhcpServer

# Create user class.
Add-DhcpServerv4Class `
  -ComputerName $DhcpServer `
  -Name $UserClassName `
  -Type User `
  -Data $UserClassData `
  -Description $UserClassDescription `
  -PassThru

# Validate class.
Get-DhcpServerv4Class `
  -ComputerName $DhcpServer |
  Where-Object {
    $_.Type -eq "User" -and
    $_.Data -eq $UserClassData
  } |
  Format-List *
~~~

# Configure_DHCP_Policies_User_Classes_And_Vendor_Classes_Create_Vendor_Class_Skeleton
~~~powershell
# Run on DHCP server or management host with RSAT DHCP tools.
# Vendor class values must match what the device actually sends in DHCP option 60.
# Example: "MS-UC-Client" is commonly used as an example vendor class value.

$DhcpServer = "DHCP1"
$VendorClassName = "CorpPhones"
$VendorClassData = "MS-UC-Client"
$VendorClassDescription = "Vendor class for UC or VoIP client handling"

Import-Module DhcpServer

# Create vendor class.
Add-DhcpServerv4Class `
  -ComputerName $DhcpServer `
  -Name $VendorClassName `
  -Type Vendor `
  -Data $VendorClassData `
  -Description $VendorClassDescription `
  -PassThru

# Validate class.
Get-DhcpServerv4Class `
  -ComputerName $DhcpServer |
  Where-Object {
    $_.Type -eq "Vendor" -and
    $_.Data -eq $VendorClassData
  } |
  Format-List *
~~~

# Configure_DHCP_Policies_User_Classes_And_Vendor_Classes_Create_Scope_Level_User_Class_Policy_Skeleton
~~~powershell
# Run on DHCP server or management host with RSAT DHCP tools.
# Create disabled first, then add ranges/options, then enable.

$DhcpServer = "DHCP1"
$ScopeId = "10.10.20.0"
$PolicyName = "POLICY-LabComputers"
$UserClassData = "LabComputers"
$PolicyDescription = "Policy for DHCP clients sending the LabComputers user class"
$LeaseDuration = New-TimeSpan -Hours 8

Import-Module DhcpServer

# Create user-class policy disabled.
Add-DhcpServerv4Policy `
  -ComputerName $DhcpServer `
  -ScopeId $ScopeId `
  -Name $PolicyName `
  -Description $PolicyDescription `
  -Condition OR `
  -UserClass EQ,$UserClassData `
  -LeaseDuration $LeaseDuration `
  -ProcessingOrder 1 `
  -Enabled $false `
  -PassThru

# Validate policy.
Get-DhcpServerv4Policy `
  -ComputerName $DhcpServer `
  -ScopeId $ScopeId `
  -Name $PolicyName |
  Format-List *
~~~

# Configure_DHCP_Policies_User_Classes_And_Vendor_Classes_Create_Scope_Level_Vendor_Class_Policy_Skeleton
~~~powershell
# Run on DHCP server or management host with RSAT DHCP tools.
# Create disabled first, then add ranges/options, then enable.
# Vendor class must already exist and must match client DHCP option 60 behavior.

$DhcpServer = "DHCP1"
$ScopeId = "10.10.20.0"
$PolicyName = "POLICY-CorpPhones"
$VendorClassData = "MS-UC-Client"
$PolicyDescription = "Policy for clients sending the MS-UC-Client vendor class"
$LeaseDuration = New-TimeSpan -Days 1

Import-Module DhcpServer

# Create vendor-class policy disabled.
Add-DhcpServerv4Policy `
  -ComputerName $DhcpServer `
  -ScopeId $ScopeId `
  -Name $PolicyName `
  -Description $PolicyDescription `
  -Condition OR `
  -VendorClass EQ,$VendorClassData `
  -LeaseDuration $LeaseDuration `
  -ProcessingOrder 2 `
  -Enabled $false `
  -PassThru

# Validate policy.
Get-DhcpServerv4Policy `
  -ComputerName $DhcpServer `
  -ScopeId $ScopeId `
  -Name $PolicyName |
  Format-List *
~~~

# Configure_DHCP_Policies_User_Classes_And_Vendor_Classes_Add_Policy_IP_Range_Skeleton
~~~powershell
# Run on DHCP server or management host with RSAT DHCP tools.
# Policy IP ranges are scope-level. They assign matching clients from a specific range inside the scope.

$DhcpServer = "DHCP1"
$ScopeId = "10.10.20.0"

$UserPolicyName = "POLICY-LabComputers"
$UserPolicyStart = "10.10.20.200"
$UserPolicyEnd = "10.10.20.209"

$VendorPolicyName = "POLICY-CorpPhones"
$VendorPolicyStart = "10.10.20.210"
$VendorPolicyEnd = "10.10.20.225"

Import-Module DhcpServer

# Add IP range for user-class policy.
Add-DhcpServerv4PolicyIPRange `
  -ComputerName $DhcpServer `
  -ScopeId $ScopeId `
  -Name $UserPolicyName `
  -StartRange $UserPolicyStart `
  -EndRange $UserPolicyEnd `
  -PassThru

# Add IP range for vendor-class policy.
Add-DhcpServerv4PolicyIPRange `
  -ComputerName $DhcpServer `
  -ScopeId $ScopeId `
  -Name $VendorPolicyName `
  -StartRange $VendorPolicyStart `
  -EndRange $VendorPolicyEnd `
  -PassThru

# Validate policy IP ranges.
Get-DhcpServerv4PolicyIPRange `
  -ComputerName $DhcpServer `
  -ScopeId $ScopeId

Get-DhcpServerv4PolicyIPRange `
  -ComputerName $DhcpServer `
  -ScopeId $ScopeId `
  -Name $UserPolicyName

Get-DhcpServerv4PolicyIPRange `
  -ComputerName $DhcpServer `
  -ScopeId $ScopeId `
  -Name $VendorPolicyName
~~~

# Configure_DHCP_Policies_User_Classes_And_Vendor_Classes_Set_Policy_Options_Skeleton
~~~powershell
# Run on DHCP server or management host with RSAT DHCP tools.
# Policy options override normal scope options for matching clients.

$DhcpServer = "DHCP1"
$ScopeId = "10.10.20.0"

$UserPolicyName = "POLICY-LabComputers"
$VendorPolicyName = "POLICY-CorpPhones"

$Router = "10.10.20.1"
$DnsServers = @("10.10.10.10","10.10.10.11")
$DnsDomain = "corp.local"

# Example vendor-specific option.
# Option 43 is vendor-specific. The value format depends on the vendor.
$VendorOptionId = 43
$VendorOptionValue = "0104C0A8010A"

Import-Module DhcpServer

# Set common options for user-class policy.
Set-DhcpServerv4OptionValue `
  -ComputerName $DhcpServer `
  -ScopeId $ScopeId `
  -PolicyName $UserPolicyName `
  -Router $Router `
  -DnsServer $DnsServers `
  -DnsDomain $DnsDomain `
  -PassThru

# Set common options for vendor-class policy.
Set-DhcpServerv4OptionValue `
  -ComputerName $DhcpServer `
  -ScopeId $ScopeId `
  -PolicyName $VendorPolicyName `
  -Router $Router `
  -DnsServer $DnsServers `
  -DnsDomain $DnsDomain `
  -PassThru

# Set vendor-specific option on the vendor policy.
# Only use option 43 when you know the vendor's required payload format.
Set-DhcpServerv4OptionValue `
  -ComputerName $DhcpServer `
  -ScopeId $ScopeId `
  -PolicyName $VendorPolicyName `
  -OptionId $VendorOptionId `
  -Value $VendorOptionValue `
  -Force `
  -PassThru

# Validate policy options.
Get-DhcpServerv4OptionValue `
  -ComputerName $DhcpServer `
  -ScopeId $ScopeId `
  -PolicyName $UserPolicyName

Get-DhcpServerv4OptionValue `
  -ComputerName $DhcpServer `
  -ScopeId $ScopeId `
  -PolicyName $VendorPolicyName
~~~

# Configure_DHCP_Policies_User_Classes_And_Vendor_Classes_Enable_And_Order_Policies_Skeleton
~~~powershell
# Run on DHCP server or management host with RSAT DHCP tools.
# Enable only after classes, conditions, ranges, and options are validated.

$DhcpServer = "DHCP1"
$ScopeId = "10.10.20.0"

$UserPolicyName = "POLICY-LabComputers"
$VendorPolicyName = "POLICY-CorpPhones"

Import-Module DhcpServer

# Set final processing order.
Set-DhcpServerv4Policy `
  -ComputerName $DhcpServer `
  -ScopeId $ScopeId `
  -Name $UserPolicyName `
  -ProcessingOrder 1 `
  -Enabled $true `
  -PassThru

Set-DhcpServerv4Policy `
  -ComputerName $DhcpServer `
  -ScopeId $ScopeId `
  -Name $VendorPolicyName `
  -ProcessingOrder 2 `
  -Enabled $true `
  -PassThru

# Validate final policy state.
Get-DhcpServerv4Policy `
  -ComputerName $DhcpServer `
  -ScopeId $ScopeId |
  Sort-Object ProcessingOrder |
  Format-Table Name,Enabled,ProcessingOrder,Condition,Description -AutoSize
~~~

# Configure_DHCP_Policies_User_Classes_And_Vendor_Classes_Client_Test_Skeleton
~~~powershell
# Run on Windows test client.
# This validates user-class behavior.
# Vendor-class behavior usually requires a real device that actually sends the vendor class.

$EvidencePath = "C:\DHCPPrep\dhcp-policies-classes"
$InterfaceAlias = "Ethernet"
$UserClassData = "LabComputers"

New-Item -ItemType Directory -Force -Path $EvidencePath

# Capture before state.
hostname |
  Tee-Object "$EvidencePath\client-hostname.txt"

ipconfig /all |
  Tee-Object "$EvidencePath\client-ipconfig-before.txt"

ipconfig /showclassid "$InterfaceAlias" |
  Tee-Object "$EvidencePath\client-showclassid-before.txt"

Get-NetIPConfiguration |
  Tee-Object "$EvidencePath\client-netipconfig-before.txt"

# Set user class ID.
ipconfig /setclassid "$InterfaceAlias" $UserClassData |
  Tee-Object "$EvidencePath\client-setclassid.txt"

ipconfig /showclassid "$InterfaceAlias" |
  Tee-Object "$EvidencePath\client-showclassid-after-set.txt"

# Renew lease.
ipconfig /release |
  Tee-Object "$EvidencePath\client-release.txt"

ipconfig /renew |
  Tee-Object "$EvidencePath\client-renew.txt"

# Capture after state.
ipconfig /all |
  Tee-Object "$EvidencePath\client-ipconfig-after-renew.txt"

Get-NetIPConfiguration |
  Tee-Object "$EvidencePath\client-netipconfig-after-renew.txt"

# Optional: clear test class after validation.
# Running setclassid without a class ID clears the class ID for the adapter.
# ipconfig /setclassid "$InterfaceAlias"

# ipconfig /release
# ipconfig /renew
# ipconfig /all
~~~

# Configure_DHCP_Policies_User_Classes_And_Vendor_Classes_Verification_Commands
~~~powershell
# DHCP server verification.

Import-Module DhcpServer

Get-Service DHCPServer

Get-DhcpServerInDC

Get-DhcpServerv4Scope `
  -ComputerName <dhcp-server> `
  -ScopeId <scope-id>

Get-DhcpServerv4Class `
  -ComputerName <dhcp-server>

Get-DhcpServerv4Class `
  -ComputerName <dhcp-server> |
  Where-Object {$_.Type -eq "User"}

Get-DhcpServerv4Class `
  -ComputerName <dhcp-server> |
  Where-Object {$_.Type -eq "Vendor"}

Get-DhcpServerv4Policy `
  -ComputerName <dhcp-server>

Get-DhcpServerv4Policy `
  -ComputerName <dhcp-server> `
  -ScopeId <scope-id>

Get-DhcpServerv4Policy `
  -ComputerName <dhcp-server> `
  -ScopeId <scope-id> `
  -Name <policy-name> |
  Format-List *

Get-DhcpServerv4PolicyIPRange `
  -ComputerName <dhcp-server> `
  -ScopeId <scope-id>

Get-DhcpServerv4PolicyIPRange `
  -ComputerName <dhcp-server> `
  -ScopeId <scope-id> `
  -Name <policy-name>

Get-DhcpServerv4OptionValue `
  -ComputerName <dhcp-server> `
  -ScopeId <scope-id> `
  -PolicyName <policy-name>

Get-DhcpServerv4Lease `
  -ComputerName <dhcp-server> `
  -ScopeId <scope-id> `
  -AllLeases

Get-DhcpServerv4Lease `
  -ComputerName <dhcp-server> `
  -ScopeId <scope-id> `
  -ClientId <client-id>

# Client verification.

ipconfig /all

ipconfig /showclassid "<client-interface>"

ipconfig /setclassid "<client-interface>" <user-class-data>

ipconfig /release

ipconfig /renew

ipconfig /all

Get-NetIPConfiguration

# Event and audit evidence.

Get-WinEvent `
  -LogName "Microsoft-Windows-Dhcp-Server/Operational" `
  -MaxEvents 100 `
  -ErrorAction SilentlyContinue

Get-WinEvent `
  -LogName "Microsoft-Windows-Dhcp-Client/Admin" `
  -MaxEvents 100 `
  -ErrorAction SilentlyContinue

Select-String `
  -Path "C:\Windows\System32\dhcp\DhcpSrvLog-*.log" `
  -Pattern "<client-id>","<client-ip>","<policy-name>" `
  -ErrorAction SilentlyContinue
~~~

# Configure_DHCP_Policies_User_Classes_And_Vendor_Classes_Rollback
~~~powershell
# Run only the rollback matching the change being reversed.
# Capture current state first.

$DhcpServer = "DHCP1"
$ScopeId = "10.10.20.0"
$UserClassName = "LabComputers"
$VendorClassName = "CorpPhones"
$UserPolicyName = "POLICY-LabComputers"
$VendorPolicyName = "POLICY-CorpPhones"
$EvidencePath = "C:\DHCPPrep\dhcp-policies-classes"

Import-Module DhcpServer
New-Item -ItemType Directory -Force -Path $EvidencePath

# Capture current state.
Get-DhcpServerv4Class `
  -ComputerName $DhcpServer |
  Export-Csv "$EvidencePath\rollback-classes-before.csv" -NoTypeInformation

Get-DhcpServerv4Policy `
  -ComputerName $DhcpServer `
  -ScopeId $ScopeId `
  -ErrorAction SilentlyContinue |
  Export-Csv "$EvidencePath\rollback-scope-policies-before.csv" -NoTypeInformation

Get-DhcpServerv4PolicyIPRange `
  -ComputerName $DhcpServer `
  -ScopeId $ScopeId `
  -ErrorAction SilentlyContinue |
  Export-Csv "$EvidencePath\rollback-policy-ip-ranges-before.csv" -NoTypeInformation

Get-DhcpServerv4OptionValue `
  -ComputerName $DhcpServer `
  -ScopeId $ScopeId `
  -PolicyName $UserPolicyName `
  -ErrorAction SilentlyContinue |
  Export-Csv "$EvidencePath\rollback-user-policy-options-before.csv" -NoTypeInformation

Get-DhcpServerv4OptionValue `
  -ComputerName $DhcpServer `
  -ScopeId $ScopeId `
  -PolicyName $VendorPolicyName `
  -ErrorAction SilentlyContinue |
  Export-Csv "$EvidencePath\rollback-vendor-policy-options-before.csv" -NoTypeInformation

# Safest first rollback: disable policies.
Set-DhcpServerv4Policy `
  -ComputerName $DhcpServer `
  -ScopeId $ScopeId `
  -Name $UserPolicyName `
  -Enabled $false `
  -ErrorAction SilentlyContinue `
  -PassThru

Set-DhcpServerv4Policy `
  -ComputerName $DhcpServer `
  -ScopeId $ScopeId `
  -Name $VendorPolicyName `
  -Enabled $false `
  -ErrorAction SilentlyContinue `
  -PassThru

# Optional rollback: remove policy IP ranges.
# Remove-DhcpServerv4PolicyIPRange `
#   -ComputerName $DhcpServer `
#   -ScopeId $ScopeId `
#   -Name $UserPolicyName `
#   -StartRange "10.10.20.200" `
#   -EndRange "10.10.20.209" `
#   -Confirm

# Remove-DhcpServerv4PolicyIPRange `
#   -ComputerName $DhcpServer `
#   -ScopeId $ScopeId `
#   -Name $VendorPolicyName `
#   -StartRange "10.10.20.210" `
#   -EndRange "10.10.20.225" `
#   -Confirm

# Optional rollback: remove policies completely.
# Remove-DhcpServerv4Policy `
#   -ComputerName $DhcpServer `
#   -ScopeId $ScopeId `
#   -Name $UserPolicyName `
#   -Confirm

# Remove-DhcpServerv4Policy `
#   -ComputerName $DhcpServer `
#   -ScopeId $ScopeId `
#   -Name $VendorPolicyName `
#   -Confirm

# Optional rollback: remove classes only after policies using them are removed.
# Remove-DhcpServerv4Class `
#   -ComputerName $DhcpServer `
#   -Name $UserClassName `
#   -Type User `
#   -Confirm

# Remove-DhcpServerv4Class `
#   -ComputerName $DhcpServer `
#   -Name $VendorClassName `
#   -Type Vendor `
#   -Confirm

# Client rollback: clear test user class.
# Run on the test client.
# ipconfig /setclassid "Ethernet"
# ipconfig /release
# ipconfig /renew
# ipconfig /all
~~~

# Configure_DHCP_Policies_User_Classes_And_Vendor_Classes_Failure_Checks
| Symptom | Likely Cause | Check | Corrective Action |
|---|---|---|---|
| Policy cannot be created | Class does not exist or condition syntax is wrong | `Get-DhcpServerv4Class`; review `Add-DhcpServerv4Policy` condition | Create class first and use `EQ,<value>` or `NE,<value>` syntax |
| User-class policy never matches | Client is not sending user class | `ipconfig /showclassid "<adapter>"`; `ipconfig /all` | Set class with `ipconfig /setclassid "<adapter>" <class-data>` |
| Vendor-class policy never matches | Device is not sending expected vendor class | DHCP audit logs, packet capture, vendor docs | Correct vendor class data or test with real device |
| Policy exists but does nothing | Policy disabled | `Get-DhcpServerv4Policy -ScopeId <scope-id>` | Enable policy after validation |
| Wrong policy matches first | Processing order wrong | `Get-DhcpServerv4Policy -ScopeId <scope-id>` | Adjust `-ProcessingOrder` |
| Client gets normal scope range instead of policy range | Policy IP range missing or client does not match policy | `Get-DhcpServerv4PolicyIPRange`; `Get-DhcpServerv4Lease` | Add policy range or fix condition |
| Client gets policy IP but wrong DNS/gateway | Policy-specific options missing | `Get-DhcpServerv4OptionValue -ScopeId <scope-id> -PolicyName <policy-name>` | Set policy-specific options |
| Option 43 does not work | Vendor-specific payload format wrong | Vendor documentation and packet capture | Replace option 43 value with vendor-required payload |
| Policy option command fails | Option definition does not exist or invalid option level | `Get-DhcpServerv4OptionDefinition`; command error | Create/verify option definition and use correct server/scope/policy level |
| UserClass and PolicyName conflict in option command | Invalid parameter combination | Review `Set-DhcpServerv4OptionValue` command | Use `-PolicyName` for policy-specific values; use `-UserClass` only for legacy class-based options |
| Client still has old lease | Client has not renewed | `ipconfig /all`; lease time | Run `ipconfig /release` and `ipconfig /renew` |
| Policy range appears exhausted | Too small policy range or stale leases | `Get-DhcpServerv4Lease -AllLeases`; policy range size | Expand policy range or clean stale leases |
| Multiple conditions behave unexpectedly | `AND` versus `OR` mismatch | `Get-DhcpServerv4Policy | Format-List *` | Rebuild condition logic |
| Server-level policy affects more clients than expected | Policy created at server level instead of scope level | `Get-DhcpServerv4Policy` with and without `-ScopeId` | Remove/recreate policy at intended level |
| Scope-level policy missing | Policy created at wrong scope | `Get-DhcpServerv4Policy -ScopeId <scope-id>` | Recreate policy in correct scope |
| Client receives expected IP but not expected behavior | Not a DHCP policy issue | Confirm IP, options, DNS, gateway, app config | Move to DNS, routing, firewall, or application troubleshooting |

# Configure_DHCP_Policies_User_Classes_And_Vendor_Classes_Related_Labs
| Lab | Relationship |
|---|---|
| 01_Install_DHCP_Server_Role_And_Management_Tools | DHCP policies/classes require DHCP Server role and PowerShell module |
| 02_Create_DHCPv4_Scope | Policies usually attach to existing DHCPv4 scopes |
| 03_Configure_DHCPv4_Scope_Options | Policy options override or specialize normal scope options |
| 04_Activate_Authorize_And_Verify_DHCP_Server | Policy behavior depends on working authorization, service, and bindings |
| 05_Configure_DHCPv4_Exclusions_And_Reservations | Policies are separate from reservations, but both affect which clients receive what configuration |
| 11_Monitor_DHCP_Leases_Events_And_Scope_Utilization | Policy validation depends on leases, events, scope utilization, and audit logs |
| 12_Troubleshoot_DHCP_Client_Lease_Failures | Failed policy matches often look like normal DHCP lease failures |