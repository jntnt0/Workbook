21_Configure_Certificate_AutoEnrollment_And_PKIGPO_Settings.md
# 21_Configure_Certificate_AutoEnrollment_And_PKIGPO_Settings

# 21_Configure_Certificate_AutoEnrollment_And_PKIGPO_Settings_Index
21_Configure_Certificate_AutoEnrollment_And_PKIGPO_Settings.md
21_Configure_Certificate_AutoEnrollment_And_PKIGPO_Settings
21_Configure_Certificate_AutoEnrollment_And_PKIGPO_Settings_Source_Basis
21_Configure_Certificate_AutoEnrollment_And_PKIGPO_Settings_Mental_Model
21_Configure_Certificate_AutoEnrollment_And_PKIGPO_Settings_Planning_Table
21_Configure_Certificate_AutoEnrollment_And_PKIGPO_Settings_Control_Map
21_Configure_Certificate_AutoEnrollment_And_PKIGPO_Settings_Configuration_Checklist
21_Configure_Certificate_AutoEnrollment_And_PKIGPO_Settings_Precheck_Skeleton
21_Configure_Certificate_AutoEnrollment_And_PKIGPO_Settings_CA_And_Template_Readiness_Skeleton
21_Configure_Certificate_AutoEnrollment_And_PKIGPO_Settings_Template_Permissions_And_Publication_Skeleton
21_Configure_Certificate_AutoEnrollment_And_PKIGPO_Settings_Create_And_Link_GPO_Skeleton
21_Configure_Certificate_AutoEnrollment_And_PKIGPO_Settings_Computer_AutoEnrollment_GPMC_Skeleton
21_Configure_Certificate_AutoEnrollment_And_PKIGPO_Settings_User_AutoEnrollment_GPMC_Skeleton
21_Configure_Certificate_AutoEnrollment_And_PKIGPO_Settings_PKIGPO_Trust_And_Path_Settings_Skeleton
21_Configure_Certificate_AutoEnrollment_And_PKIGPO_Settings_Client_Enrollment_Validation_Skeleton
21_Configure_Certificate_AutoEnrollment_And_PKIGPO_Settings_Report_And_Backup_Skeleton
21_Configure_Certificate_AutoEnrollment_And_PKIGPO_Settings_Verification_Commands
21_Configure_Certificate_AutoEnrollment_And_PKIGPO_Settings_Rollback
21_Configure_Certificate_AutoEnrollment_And_PKIGPO_Settings_Failure_Checks
21_Configure_Certificate_AutoEnrollment_And_PKIGPO_Settings_Related_Labs

# 21_Configure_Certificate_AutoEnrollment_And_PKIGPO_Settings_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Active Directory Certificate Services | Enterprise CA, certificate templates, enrollment, and trust model |
| Microsoft Learn | Certificate auto-enrollment | Computer and user certificate auto-enrollment through Group Policy |
| Microsoft Learn | Certificate templates | Template permissions, enrollment, auto-enrollment, and supersedence |
| Microsoft Learn | Public Key Policies Group Policy | Auto-enrollment, trusted root, enterprise trust, path validation, and certificate services client settings |
| Microsoft Learn | `certutil` | CA validation, template listing, enrollment pulse, root store, NTAuth, CRL, and chain checks |
| Microsoft Learn | Group Policy Management Console | Configuring PKI and auto-enrollment policy |
| Microsoft Learn | GroupPolicy PowerShell module | Creating, linking, reporting, and backing up PKI GPOs |
| Windows client certificate stores | `Cert:\LocalMachine` and `Cert:\CurrentUser` | Validating issued certificates on clients |
| Windows Server operational practice | Pilot certificate auto-enrollment before broad rollout | Preventing certificate sprawl, wrong templates, and broken authentication |

# 21_Configure_Certificate_AutoEnrollment_And_PKIGPO_Settings_Mental_Model
| Concept | Operational Meaning |
|---|---|
| PKI | Public Key Infrastructure used for certificates, trust, authentication, encryption, and signing |
| AD CS | Active Directory Certificate Services role that can issue certificates |
| Enterprise CA | AD-integrated CA that can publish templates and support domain auto-enrollment |
| Root CA | Trust anchor for certificate chains |
| Issuing CA | CA that issues end-entity certificates to users, computers, or services |
| Certificate template | AD object that defines certificate purpose, subject, key usage, enrollment rules, and security |
| Template publication | Making a template available on the CA for enrollment |
| Enroll permission | Template permission allowing manual enrollment |
| Autoenroll permission | Template permission allowing automatic certificate enrollment |
| Computer auto-enrollment | Computer certificate enrollment based on computer policy and computer template permissions |
| User auto-enrollment | User certificate enrollment based on user policy and user template permissions |
| Public Key Policies | GPO area where certificate auto-enrollment and trust settings are configured |
| NTAuth store | AD store used to trust CAs for smart card logon and certain enterprise authentication scenarios |
| Trusted Root store | Local or domain-managed store containing trusted root CA certificates |
| Intermediate store | Store containing intermediate CA certificates |
| CRL | Certificate Revocation List used to check revoked certificates |
| AIA | Authority Information Access location used to find issuing CA certificates |
| CDP | CRL Distribution Point location used to find revocation data |
| Certificate chain | End certificate plus issuing CA chain back to a trusted root |
| `certutil -pulse` | Triggers certificate auto-enrollment processing |
| First rule | Template permissions and GPO auto-enrollment must both be correct |
| Blunt rule | If clients cannot build the chain or reach CRL locations, certificates may enroll but still fail authentication |

# 21_Configure_Certificate_AutoEnrollment_And_PKIGPO_Settings_Planning_Table
| Item | Example | Decision |
|---|---|---|
| Domain FQDN | `corp.local` | `<domain-fqdn>` |
| Domain DN | `DC=corp,DC=local` | `<domain-dn>` |
| Enterprise CA server | `CA01` | `<ca-server>` |
| CA common name | `CORP-CA01-CA` | `<ca-common-name>` |
| CA config string | `CA01\CORP-CA01-CA` | `<ca-config>` |
| Root CA certificate path | `C:\GPOPrep\PKI\RootCA.cer` | `<root-ca-cert-path>` |
| Issuing CA certificate path | `C:\GPOPrep\PKI\IssuingCA.cer` | `<issuing-ca-cert-path>` |
| Pilot computer OU | `OU=Pilot,OU=Workstations,OU=Corp,DC=corp,DC=local` | `<pilot-computer-ou-dn>` |
| Pilot user OU | `OU=Users,OU=Corp,DC=corp,DC=local` | `<pilot-user-ou-dn>` |
| Test computer | `WIN11-01` | `<test-computer>` |
| Test user | `tuser` | `<test-user>` |
| Computer certificate template | `Corp-Workstation-Authentication` | `<computer-template-name>` |
| User certificate template | `Corp-User-Authentication` | `<user-template-name>` |
| Computer auto-enroll group | `GG_Cert_AutoEnroll_Workstations` | `<computer-autoenroll-group>` |
| User auto-enroll group | `GG_Cert_AutoEnroll_Users` | `<user-autoenroll-group>` |
| PKI computer GPO | `CORP-PKI-Computer-AutoEnrollment` | `<computer-pki-gpo>` |
| PKI user GPO | `CORP-PKI-User-AutoEnrollment` | `<user-pki-gpo>` |
| Trust GPO | `CORP-PKI-Trust-And-Path-Validation` | `<trust-gpo>` |
| Auto-enrollment mode | Renew, update, enroll | `<autoenrollment-mode>` |
| Certificate purpose | Client Authentication, Server Authentication, Smart Card Logon, EFS | `<certificate-purpose>` |
| CRL location | `http://pki.corp.local/crl/...` | `<crl-location>` |
| AIA location | `http://pki.corp.local/aia/...` | `<aia-location>` |
| Report path | `C:\GPOPrep\Reports\PKI` | `<report-path>` |
| Backup path | `C:\GPOPrep\Backup\PKI` | `<backup-path>` |
| Rollback method | Disable GPO links, unpublish template, restore GPO backup | `<rollback-plan>` |

# 21_Configure_Certificate_AutoEnrollment_And_PKIGPO_Settings_Control_Map
| Control | Tool / GPO Location | Purpose | Validation |
|---|---|---|---|
| CA health | `certutil -config "<ca-config>" -ping` | Confirms CA RPC availability | Ping succeeds |
| Template list | `certutil -template` and `certutil -catemplates` | Confirms templates exist and are published | Template appears |
| Template permissions | `certtmpl.msc` | Grants Enroll and Autoenroll to approved groups | Security tab shows correct ACEs |
| Template publication | `certsrv.msc` or `certutil -SetCATemplates` | Makes template available on CA | CA template list includes template |
| Computer auto-enrollment | `Computer Configuration > Windows Settings > Security Settings > Public Key Policies` | Enables computer certificate auto-enrollment | `certutil -pulse` and LocalMachine cert appears |
| User auto-enrollment | `User Configuration > Windows Settings > Security Settings > Public Key Policies` | Enables user certificate auto-enrollment | User certificate appears in CurrentUser store |
| Trusted Root GPO | `Public Key Policies > Trusted Root Certification Authorities` | Deploys root CA trust | Root appears in LocalMachine Root store |
| Intermediate CA GPO | `Public Key Policies > Intermediate Certification Authorities` | Deploys issuing/intermediate CA certs | Intermediate appears in CA store |
| NTAuth publication | `certutil -dspublish -f <ca.cer> NTAuthCA` | Trusts CA for enterprise authentication scenarios | `certutil -enterprise -viewstore NTAuth` |
| Path validation policy | `Public Key Policies > Certificate Path Validation Settings` | Controls chain and revocation behavior | GPO report and chain tests |
| Auto-enrollment trigger | `certutil -pulse` | Starts enrollment cycle | CertificateServicesClient events |
| Client cert store | `Cert:\LocalMachine\My`, `Cert:\CurrentUser\My` | Confirms issued certificates | Expected cert exists |
| Event logs | AutoEnrollment, CAPI2, Application | Shows enrollment success/failure | Events show success or actionable error |

# 21_Configure_Certificate_AutoEnrollment_And_PKIGPO_Settings_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Open elevated PowerShell | Management Host / CA | `whoami /groups` | Admin token is visible |
| 2 | Import AD module | Management Host / CA | `Import-Module ActiveDirectory` | AD module imports |
| 3 | Import GroupPolicy module | Management Host / CA | `Import-Module GroupPolicy` | GroupPolicy module imports |
| 4 | Confirm domain identity | Management Host / CA | `Get-ADDomain` | Domain object returns |
| 5 | Confirm SYSVOL access | Management Host / CA | `Test-Path "\\<domain-fqdn>\SYSVOL\<domain-fqdn>\Policies"` | Returns True |
| 6 | Confirm CA server resolves | Management Host | `Resolve-DnsName <ca-server>` | CA DNS record resolves |
| 7 | Confirm CA service responds | Management Host / CA | `certutil -config "<ca-config>" -ping` | CA ping succeeds |
| 8 | Confirm CA role service | CA Server | `Get-Service CertSvc` | Certificate Services service exists and runs |
| 9 | Create report folder | Management Host | `New-Item -ItemType Directory -Force -Path C:\GPOPrep\Reports\PKI` | Report path exists |
| 10 | Create backup folder | Management Host | `New-Item -ItemType Directory -Force -Path C:\GPOPrep\Backup\PKI` | Backup path exists |
| 11 | Back up current GPO state | Management Host | `Backup-GPO -All -Path C:\GPOPrep\Backup\PKI` | Pre-change GPO backup exists |
| 12 | Confirm pilot computer OU exists | Management Host | `Get-ADOrganizationalUnit -Identity "<pilot-computer-ou-dn>"` | OU returns |
| 13 | Confirm pilot user OU exists | Management Host | `Get-ADOrganizationalUnit -Identity "<pilot-user-ou-dn>"` | OU returns |
| 14 | Confirm test computer exists | Management Host | `Get-ADComputer -Identity "<test-computer>" -Properties DistinguishedName,Enabled` | Computer object returns |
| 15 | Confirm test user exists | Management Host | `Get-ADUser -Identity "<test-user>" -Properties DistinguishedName,Enabled` | User object returns |
| 16 | Review existing certificate templates | CA / Management Host | `certutil -template` | Templates are listed |
| 17 | Review CA-published templates | CA / Management Host | `certutil -config "<ca-config>" -catemplates` | Published templates are listed |
| 18 | Open certificate template console | CA / Management Host | `certtmpl.msc` | Certificate Templates console opens |
| 19 | Duplicate or select computer template | CA / Management Host | `certtmpl.msc` | Computer template exists |
| 20 | Grant computer template permissions | CA / Management Host | `Security tab > add <computer-autoenroll-group> > Enroll + Autoenroll` | Group can enroll and autoenroll |
| 21 | Duplicate or select user template | CA / Management Host | `certtmpl.msc` | User template exists |
| 22 | Grant user template permissions | CA / Management Host | `Security tab > add <user-autoenroll-group> > Enroll + Autoenroll` | Group can enroll and autoenroll |
| 23 | Publish computer template on CA | CA / Management Host | `certsrv.msc > Certificate Templates > New > Certificate Template to Issue` | Computer template appears under issued templates |
| 24 | Publish user template on CA | CA / Management Host | `certsrv.msc > Certificate Templates > New > Certificate Template to Issue` | User template appears under issued templates |
| 25 | Confirm templates are published | CA / Management Host | `certutil -config "<ca-config>" -catemplates` | Target templates appear |
| 26 | Publish CA cert to NTAuth if required | CA / Management Host | `certutil -dspublish -f "<issuing-ca-cert-path>" NTAuthCA` | CA appears in NTAuth store |
| 27 | Create computer PKI GPO | Management Host | `New-GPO -Name "<computer-pki-gpo>" -Comment "Computer certificate auto-enrollment policy"` | GPO exists |
| 28 | Link computer PKI GPO | Management Host | `New-GPLink -Name "<computer-pki-gpo>" -Target "<pilot-computer-ou-dn>" -LinkEnabled Yes` | GPO linked to computer OU |
| 29 | Create user PKI GPO | Management Host | `New-GPO -Name "<user-pki-gpo>" -Comment "User certificate auto-enrollment policy"` | GPO exists |
| 30 | Link user PKI GPO | Management Host | `New-GPLink -Name "<user-pki-gpo>" -Target "<pilot-user-ou-dn>" -LinkEnabled Yes` | GPO linked to user OU |
| 31 | Configure computer auto-enrollment | Management Host | `GPMC > Computer Configuration > Windows Settings > Security Settings > Public Key Policies > Certificate Services Client - Auto-Enrollment` | Computer auto-enrollment enabled |
| 32 | Configure user auto-enrollment | Management Host | `GPMC > User Configuration > Windows Settings > Security Settings > Public Key Policies > Certificate Services Client - Auto-Enrollment` | User auto-enrollment enabled |
| 33 | Create PKI trust GPO if needed | Management Host | `New-GPO -Name "<trust-gpo>" -Comment "PKI trust and path validation settings"` | Trust GPO exists |
| 34 | Link trust GPO to pilot scope | Management Host | `New-GPLink -Name "<trust-gpo>" -Target "<pilot-computer-ou-dn>" -LinkEnabled Yes` | Trust GPO linked |
| 35 | Configure root/intermediate trust if needed | Management Host | `GPMC > Public Key Policies > Trusted Root Certification Authorities / Intermediate Certification Authorities` | CA certs are deployed |
| 36 | Configure path validation if required | Management Host | `GPMC > Public Key Policies > Certificate Path Validation Settings` | Path validation settings are configured |
| 37 | Export GPO reports | Management Host | `Get-GPOReport -Name "<computer-pki-gpo>" -ReportType Html -Path C:\GPOPrep\Reports\PKI\<computer-pki-gpo>.html` | Reports exist |
| 38 | Back up configured PKI GPOs | Management Host | `Backup-GPO -Name "<computer-pki-gpo>" -Path C:\GPOPrep\Backup\PKI` | PKI GPO backup exists |
| 39 | Refresh computer policy on test client | Test Client | `gpupdate /target:computer /force` | Computer policy refresh completes |
| 40 | Trigger computer certificate auto-enrollment | Test Client | `certutil -pulse` | Auto-enrollment cycle runs |
| 41 | Validate computer certificate | Test Client | `Get-ChildItem Cert:\LocalMachine\My` | Expected computer certificate appears |
| 42 | Refresh user policy as test user | Test Client | `gpupdate /target:user /force` | User policy refresh completes |
| 43 | Trigger user certificate auto-enrollment | Test Client | `certutil -pulse` | Auto-enrollment cycle runs |
| 44 | Validate user certificate | Test Client | `Get-ChildItem Cert:\CurrentUser\My` | Expected user certificate appears |
| 45 | Validate certificate chain | Test Client | `certutil -verify "<exported-cert.cer>"` | Chain builds and revocation checks succeed |
| 46 | Review auto-enrollment events | Test Client | `Get-WinEvent -LogName "Microsoft-Windows-CertificateServicesClient-AutoEnrollment/Operational" -MaxEvents 100` | Success or actionable failure events appear |
| 47 | Review CAPI2 events if chain fails | Test Client | `Get-WinEvent -LogName "Microsoft-Windows-CAPI2/Operational" -MaxEvents 100` | Chain and revocation details appear |
| 48 | Export GPResult report | Test Client | `gpresult /h C:\GPOPrep\Reports\PKI\gpresult-pki.html` | HTML RSOP report exists |
| 49 | Document result | Operator | `Record CA, templates, permissions, GPOs, certificate thumbprints, chain result, event logs, and rollback path` | PKI auto-enrollment deployment is documented |

# 21_Configure_Certificate_AutoEnrollment_And_PKIGPO_Settings_Precheck_Skeleton
```powershell
# Run on a domain controller, CA, or domain-joined management host.
# Purpose: validate domain, CA, SYSVOL, OU, and object readiness before configuring PKI GPOs.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$Domain = Get-ADDomain
$DomainFqdn = $Domain.DNSRoot
$DomainDN = $Domain.DistinguishedName

$CAServer = "CA01"
$CACommonName = "CORP-CA01-CA"
$CAConfig = "$CAServer\$CACommonName"

$PilotComputerOU = "OU=Pilot,OU=Workstations,OU=Corp,$DomainDN"
$PilotUserOU = "OU=Users,OU=Corp,$DomainDN"

$TestComputer = "WIN11-01"
$TestUser = "tuser"

$BasePath = "C:\GPOPrep"
$ReportPath = "$BasePath\Reports\PKI"
$BackupPath = "$BasePath\Backup\PKI"

New-Item -ItemType Directory -Force -Path $ReportPath,$BackupPath

# Confirm domain.
$Domain |
  Select-Object DNSRoot,NetBIOSName,DistinguishedName,PDCEmulator |
  Out-File "$ReportPath\domain-info.txt"

# Confirm SYSVOL and NETLOGON.
nltest /dsgetdc:$DomainFqdn |
  Out-File "$ReportPath\nltest-dsgetdc.txt"

Test-Path "\\$DomainFqdn\SYSVOL\$DomainFqdn\Policies" |
  Out-File "$ReportPath\sysvol-check.txt"

Test-Path "\\$DomainFqdn\NETLOGON" |
  Out-File "$ReportPath\netlogon-check.txt"

# Confirm CA resolution and service.
Resolve-DnsName $CAServer |
  Out-File "$ReportPath\ca-dns-resolution.txt" `
  -ErrorAction SilentlyContinue

certutil -config $CAConfig -ping |
  Out-File "$ReportPath\ca-ping.txt"

# Confirm pilot OUs and objects.
Get-ADOrganizationalUnit -Identity $PilotComputerOU |
  Out-File "$ReportPath\pilot-computer-ou.txt"

Get-ADOrganizationalUnit -Identity $PilotUserOU |
  Out-File "$ReportPath\pilot-user-ou.txt"

Get-ADComputer `
  -Identity $TestComputer `
  -Properties DNSHostName,Enabled,DistinguishedName,MemberOf |
  Select-Object Name,DNSHostName,Enabled,DistinguishedName,MemberOf |
  Out-File "$ReportPath\test-computer-before-pki.txt"

Get-ADUser `
  -Identity $TestUser `
  -Properties UserPrincipalName,Enabled,DistinguishedName,MemberOf |
  Select-Object SamAccountName,UserPrincipalName,Enabled,DistinguishedName,MemberOf |
  Out-File "$ReportPath\test-user-before-pki.txt"

# Capture current OU inheritance.
Get-GPInheritance `
  -Target $PilotComputerOU |
  Out-File "$ReportPath\pilot-computer-ou-inheritance-before-pki.txt"

Get-GPInheritance `
  -Target $PilotUserOU |
  Out-File "$ReportPath\pilot-user-ou-inheritance-before-pki.txt"

# Capture GPO inventory and backup.
Get-GPO -All |
  Sort-Object DisplayName |
  Select-Object DisplayName,Id,GpoStatus,Owner,CreationTime,ModificationTime |
  Export-Csv "$ReportPath\gpo-inventory-before-pki.csv" -NoTypeInformation

Backup-GPO `
  -All `
  -Path $BackupPath `
  -Comment "Pre PKI auto-enrollment GPO backup"

Write-Host "PKI auto-enrollment precheck complete."
Write-Host "Report path: $ReportPath"
Write-Host "Backup path: $BackupPath"
```

# 21_Configure_Certificate_AutoEnrollment_And_PKIGPO_Settings_CA_And_Template_Readiness_Skeleton
```powershell
# Run on the CA server or management host with CA tools.
# Purpose: verify CA availability, templates, root trust, NTAuth, CRL, and AIA readiness.

$CAServer = "CA01"
$CACommonName = "CORP-CA01-CA"
$CAConfig = "$CAServer\$CACommonName"

$ReportPath = "C:\GPOPrep\Reports\PKI"
New-Item -ItemType Directory -Force -Path $ReportPath

# Confirm Certificate Services.
Get-Service `
  -Name CertSvc `
  -ErrorAction SilentlyContinue |
  Select-Object Name,Status,StartType |
  Out-File "$ReportPath\certsvc-service.txt"

# Confirm CA responds.
certutil -config $CAConfig -ping |
  Out-File "$ReportPath\ca-ping.txt"

# Dump CA configuration.
certutil -config $CAConfig -getconfig |
  Out-File "$ReportPath\ca-getconfig.txt"

# List available certificate templates in AD.
certutil -template |
  Out-File "$ReportPath\certutil-template-list.txt"

# List templates currently published by the CA.
certutil -config $CAConfig -catemplates |
  Out-File "$ReportPath\ca-published-templates.txt"

# View enterprise root store.
certutil -enterprise -viewstore Root |
  Out-File "$ReportPath\enterprise-root-store.txt"

# View NTAuth store.
certutil -enterprise -viewstore NTAuth |
  Out-File "$ReportPath\enterprise-ntauth-store.txt"

# Check CA registry configuration for CRL and AIA publication data.
certutil -config $CAConfig -getreg CA\CRLPublicationURLs |
  Out-File "$ReportPath\ca-crl-publication-urls.txt"

certutil -config $CAConfig -getreg CA\CACertPublicationURLs |
  Out-File "$ReportPath\ca-aia-publication-urls.txt"

Write-Host "CA and template readiness collection complete."
Write-Host "Report path: $ReportPath"
```

# 21_Configure_Certificate_AutoEnrollment_And_PKIGPO_Settings_Template_Permissions_And_Publication_Skeleton
```powershell
# Native GUI workflow for template creation, permissions, and publication.
# Run on the CA server or management host with AD CS management tools.

certtmpl.msc

# Template workflow:
# 1. Open Certificate Templates Console.
# 2. Find a safe source template:
#      Workstation Authentication for computer certs
#      User for basic user certs
#      Smartcard Logon only if smart card design exists
# 3. Right-click source template.
# 4. Select Duplicate Template.
# 5. Name the new template:
#      Corp-Workstation-Authentication
#      or
#      Corp-User-Authentication
#
# Template properties:
# 6. Compatibility tab:
#      Select CA and recipient compatibility appropriate for the lab.
# 7. General tab:
#      Set display name and validity period.
# 8. Request Handling tab:
#      Confirm purpose and private key behavior.
# 9. Cryptography tab:
#      Confirm provider and key size.
# 10. Subject Name tab:
#      Usually build from Active Directory information for auto-enrollment.
# 11. Extensions tab:
#      Confirm EKUs such as Client Authentication if needed.
#
# Security tab:
# 12. Add:
#      CORP\GG_Cert_AutoEnroll_Workstations
#      or
#      CORP\GG_Cert_AutoEnroll_Users
# 13. Grant:
#      Read
#      Enroll
#      Autoenroll
# 14. Avoid granting Autoenroll to broad groups until pilot is validated.
#
# Publish template:
# 15. Open Certification Authority console:
#      certsrv.msc
# 16. Expand CA.
# 17. Right-click Certificate Templates.
# 18. Select:
#      New > Certificate Template to Issue
# 19. Select:
#      Corp-Workstation-Authentication
#      Corp-User-Authentication
# 20. Confirm templates appear under Certificate Templates.
```

```powershell
# Optional command-line checks after GUI publication.

$CAServer = "CA01"
$CACommonName = "CORP-CA01-CA"
$CAConfig = "$CAServer\$CACommonName"

$ReportPath = "C:\GPOPrep\Reports\PKI"
New-Item -ItemType Directory -Force -Path $ReportPath

# Confirm templates are published on CA.
certutil -config $CAConfig -catemplates |
  Out-File "$ReportPath\ca-published-templates-after-publication.txt"

# Optional:
# Publish CA certificate to NTAuth if required for enterprise authentication.
# certutil -dspublish -f "C:\GPOPrep\PKI\IssuingCA.cer" NTAuthCA
```

# 21_Configure_Certificate_AutoEnrollment_And_PKIGPO_Settings_Create_And_Link_GPO_Skeleton
```powershell
# Run on a domain controller or domain-joined management host.
# Purpose: create and link PKI auto-enrollment GPOs.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$DomainDN = (Get-ADDomain).DistinguishedName

$PilotComputerOU = "OU=Pilot,OU=Workstations,OU=Corp,$DomainDN"
$PilotUserOU = "OU=Users,OU=Corp,$DomainDN"

$ComputerPkiGpo = "CORP-PKI-Computer-AutoEnrollment"
$UserPkiGpo = "CORP-PKI-User-AutoEnrollment"
$TrustGpo = "CORP-PKI-Trust-And-Path-Validation"

# Confirm OUs.
Get-ADOrganizationalUnit -Identity $PilotComputerOU
Get-ADOrganizationalUnit -Identity $PilotUserOU

# Create GPOs.
foreach ($GpoName in @($ComputerPkiGpo,$UserPkiGpo,$TrustGpo)) {
    if (-not (Get-GPO -Name $GpoName -ErrorAction SilentlyContinue)) {
        New-GPO `
          -Name $GpoName `
          -Comment "PKI certificate auto-enrollment and trust policy"
    }
}

# Link computer auto-enrollment and trust GPOs to computer OU.
foreach ($GpoName in @($ComputerPkiGpo,$TrustGpo)) {
    $Inheritance = Get-GPInheritance -Target $PilotComputerOU

    if (-not ($Inheritance.GpoLinks | Where-Object DisplayName -eq $GpoName)) {
        New-GPLink `
          -Name $GpoName `
          -Target $PilotComputerOU `
          -LinkEnabled Yes
    }

    Set-GPLink `
      -Name $GpoName `
      -Target $PilotComputerOU `
      -LinkEnabled Yes `
      -Enforced No
}

# Link user auto-enrollment GPO to user OU.
$Inheritance = Get-GPInheritance -Target $PilotUserOU

if (-not ($Inheritance.GpoLinks | Where-Object DisplayName -eq $UserPkiGpo)) {
    New-GPLink `
      -Name $UserPkiGpo `
      -Target $PilotUserOU `
      -LinkEnabled Yes
}

Set-GPLink `
  -Name $UserPkiGpo `
  -Target $PilotUserOU `
  -LinkEnabled Yes `
  -Enforced No

# Confirm link state.
Get-GPInheritance -Target $PilotComputerOU
Get-GPInheritance -Target $PilotUserOU
```

# 21_Configure_Certificate_AutoEnrollment_And_PKIGPO_Settings_Computer_AutoEnrollment_GPMC_Skeleton
```powershell
# Native GUI workflow for computer certificate auto-enrollment.
# Run on the management host.

gpmc.msc

# GUI workflow:
# 1. Open Group Policy Management Console.
# 2. Edit:
#      CORP-PKI-Computer-AutoEnrollment
# 3. Go to:
#      Computer Configuration
#      > Policies
#      > Windows Settings
#      > Security Settings
#      > Public Key Policies
# 4. Open:
#      Certificate Services Client - Auto-Enrollment
# 5. Set:
#      Enabled
# 6. Select:
#      Renew expired certificates, update pending certificates, and remove revoked certificates
#      Update certificates that use certificate templates
#
# Optional:
# 7. Open:
#      Certificate Services Client - Certificate Enrollment Policy
# 8. Confirm AD enrollment policy is available if configured.
#
# Validation expectation:
# - Computer must be in linked OU.
# - Computer or computer group must have template Read, Enroll, and Autoenroll.
# - CA must publish the template.
# - Client must process computer policy.
```

# 21_Configure_Certificate_AutoEnrollment_And_PKIGPO_Settings_User_AutoEnrollment_GPMC_Skeleton
```powershell
# Native GUI workflow for user certificate auto-enrollment.
# Run on the management host.

gpmc.msc

# GUI workflow:
# 1. Open Group Policy Management Console.
# 2. Edit:
#      CORP-PKI-User-AutoEnrollment
# 3. Go to:
#      User Configuration
#      > Policies
#      > Windows Settings
#      > Security Settings
#      > Public Key Policies
# 4. Open:
#      Certificate Services Client - Auto-Enrollment
# 5. Set:
#      Enabled
# 6. Select:
#      Renew expired certificates, update pending certificates, and remove revoked certificates
#      Update certificates that use certificate templates
#
# Validation expectation:
# - User must be in linked OU.
# - User or user group must have template Read, Enroll, and Autoenroll.
# - CA must publish the template.
# - User must sign in and process user policy.
```

# 21_Configure_Certificate_AutoEnrollment_And_PKIGPO_Settings_PKIGPO_Trust_And_Path_Settings_Skeleton
```powershell
# Native GUI workflow for PKI trust and path validation GPO settings.
# Run on the management host.

gpmc.msc

# Trust GPO workflow:
# 1. Edit:
#      CORP-PKI-Trust-And-Path-Validation
# 2. Go to:
#      Computer Configuration
#      > Policies
#      > Windows Settings
#      > Security Settings
#      > Public Key Policies
#
# Trusted root:
# 3. Right-click:
#      Trusted Root Certification Authorities
# 4. Select Import.
# 5. Import root CA certificate:
#      C:\GPOPrep\PKI\RootCA.cer
#
# Intermediate CA:
# 6. Right-click:
#      Intermediate Certification Authorities
# 7. Select Import.
# 8. Import issuing or intermediate CA certificate if required:
#      C:\GPOPrep\PKI\IssuingCA.cer
#
# Certificate Path Validation:
# 9. Open:
#      Certificate Path Validation Settings
# 10. Review:
#      Stores
#      Trusted Publishers
#      Network Retrieval
# 11. Configure only if the environment requires explicit path validation behavior.
#
# Enterprise Trust:
# 12. Review Enterprise Trust only when using certificate trust lists.
#
# NTAuth:
# 13. NTAuth is normally handled through AD publication, not directly as a normal GPO root import.
# 14. Validate with:
#      certutil -enterprise -viewstore NTAuth
```

```powershell
# Optional command-line trust publication examples.
# Run from a management host with proper permissions.

$RootCA = "C:\GPOPrep\PKI\RootCA.cer"
$IssuingCA = "C:\GPOPrep\PKI\IssuingCA.cer"

# Publish root CA certificate to AD root store if required.
# certutil -dspublish -f $RootCA RootCA

# Publish issuing CA certificate to NTAuth if required.
# certutil -dspublish -f $IssuingCA NTAuthCA

# Validate stores.
certutil -enterprise -viewstore Root
certutil -enterprise -viewstore NTAuth
```

# 21_Configure_Certificate_AutoEnrollment_And_PKIGPO_Settings_Client_Enrollment_Validation_Skeleton
```powershell
# Run on the pilot client.
# Purpose: refresh policy, trigger auto-enrollment, validate certificates, chain, and logs.

$DomainFqdn = "corp.local"
$ReportPath = "C:\GPOPrep\Reports\PKI"

New-Item -ItemType Directory -Force -Path $ReportPath

# Confirm identity.
hostname |
  Out-File "$ReportPath\client-hostname.txt"

whoami /all |
  Out-File "$ReportPath\client-whoami-all.txt"

Get-ComputerInfo |
  Select-Object CsName,CsDomain,CsPartOfDomain,OsName,OsVersion,OsBuildNumber |
  Out-File "$ReportPath\client-computer-info.txt"

# Confirm DC locator and SYSVOL.
nltest /dsgetdc:$DomainFqdn |
  Out-File "$ReportPath\client-nltest-dsgetdc.txt"

Test-Path "\\$DomainFqdn\SYSVOL\$DomainFqdn\Policies" |
  Out-File "$ReportPath\client-sysvol-check.txt"

# Refresh computer and user policy.
gpupdate /target:computer /force |
  Tee-Object "$ReportPath\client-gpupdate-computer.txt"

gpupdate /target:user /force |
  Tee-Object "$ReportPath\client-gpupdate-user.txt"

# Trigger certificate auto-enrollment.
certutil -pulse |
  Tee-Object "$ReportPath\client-certutil-pulse.txt"

# Export GPResult.
gpresult /scope computer /r |
  Tee-Object "$ReportPath\client-gpresult-computer-r.txt"

gpresult /scope user /r |
  Tee-Object "$ReportPath\client-gpresult-user-r.txt"

gpresult /h "$ReportPath\client-gpresult-pki.html"
gpresult /x "$ReportPath\client-gpresult-pki.xml"

# Validate LocalMachine certificates.
Get-ChildItem `
  -Path Cert:\LocalMachine\My |
  Select-Object Subject,Issuer,Thumbprint,NotBefore,NotAfter,EnhancedKeyUsageList |
  Out-File "$ReportPath\localmachine-my-certificates.txt"

# Validate CurrentUser certificates.
Get-ChildItem `
  -Path Cert:\CurrentUser\My |
  Select-Object Subject,Issuer,Thumbprint,NotBefore,NotAfter,EnhancedKeyUsageList |
  Out-File "$ReportPath\currentuser-my-certificates.txt"

# Validate root and intermediate stores.
Get-ChildItem Cert:\LocalMachine\Root |
  Select-Object Subject,Issuer,Thumbprint,NotAfter |
  Out-File "$ReportPath\localmachine-root-store.txt"

Get-ChildItem Cert:\LocalMachine\CA |
  Select-Object Subject,Issuer,Thumbprint,NotAfter |
  Out-File "$ReportPath\localmachine-intermediate-store.txt"

# Export one computer certificate for chain validation if needed.
$ComputerCert = Get-ChildItem Cert:\LocalMachine\My |
  Where-Object { $_.Subject -like "*$env:COMPUTERNAME*" } |
  Sort-Object NotAfter -Descending |
  Select-Object -First 1

if ($ComputerCert) {
    Export-Certificate `
      -Cert $ComputerCert `
      -FilePath "$ReportPath\computer-cert.cer" `
      -Force

    certutil -verify "$ReportPath\computer-cert.cer" |
      Out-File "$ReportPath\computer-cert-chain-verify.txt"
}

# Review auto-enrollment events.
Get-WinEvent `
  -LogName "Microsoft-Windows-CertificateServicesClient-AutoEnrollment/Operational" `
  -MaxEvents 150 `
  -ErrorAction SilentlyContinue |
  Select-Object TimeCreated,Id,LevelDisplayName,ProviderName,Message |
  Out-File "$ReportPath\autoenrollment-operational-events.txt"

# Review lifecycle events.
Get-WinEvent `
  -LogName "Microsoft-Windows-CertificateServicesClient-Lifecycle-System/Operational" `
  -MaxEvents 150 `
  -ErrorAction SilentlyContinue |
  Select-Object TimeCreated,Id,LevelDisplayName,ProviderName,Message |
  Out-File "$ReportPath\certificate-lifecycle-system-events.txt"

# Review CAPI2 for chain/revocation troubleshooting.
Get-WinEvent `
  -LogName "Microsoft-Windows-CAPI2/Operational" `
  -MaxEvents 150 `
  -ErrorAction SilentlyContinue |
  Select-Object TimeCreated,Id,LevelDisplayName,ProviderName,Message |
  Out-File "$ReportPath\capi2-operational-events.txt"

Write-Host "PKI client enrollment validation complete."
Write-Host "Report path: $ReportPath"
```

# 21_Configure_Certificate_AutoEnrollment_And_PKIGPO_Settings_Report_And_Backup_Skeleton
```powershell
# Run on a domain controller, CA, or domain-joined management host.
# Purpose: capture final PKI GPO reports, CA template state, trust state, and GPO backups.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$Domain = Get-ADDomain
$DomainFqdn = $Domain.DNSRoot
$DomainDN = $Domain.DistinguishedName

$CAServer = "CA01"
$CACommonName = "CORP-CA01-CA"
$CAConfig = "$CAServer\$CACommonName"

$PilotComputerOU = "OU=Pilot,OU=Workstations,OU=Corp,$DomainDN"
$PilotUserOU = "OU=Users,OU=Corp,$DomainDN"

$GpoNames = @(
  "CORP-PKI-Computer-AutoEnrollment",
  "CORP-PKI-User-AutoEnrollment",
  "CORP-PKI-Trust-And-Path-Validation"
)

$ReportPath = "C:\GPOPrep\Reports\PKI"
$BackupPath = "C:\GPOPrep\Backup\PKI"

New-Item -ItemType Directory -Force -Path $ReportPath,$BackupPath

# Capture OU inheritance.
Get-GPInheritance `
  -Target $PilotComputerOU |
  Out-File "$ReportPath\pilot-computer-ou-inheritance-final-pki.txt"

Get-GPInheritance `
  -Target $PilotUserOU |
  Out-File "$ReportPath\pilot-user-ou-inheritance-final-pki.txt"

# Capture CA and template state.
certutil -config $CAConfig -ping |
  Out-File "$ReportPath\ca-ping-final.txt"

certutil -config $CAConfig -catemplates |
  Out-File "$ReportPath\ca-published-templates-final.txt"

certutil -enterprise -viewstore Root |
  Out-File "$ReportPath\enterprise-root-store-final.txt"

certutil -enterprise -viewstore NTAuth |
  Out-File "$ReportPath\enterprise-ntauth-store-final.txt"

# Export GPO reports and backups.
foreach ($GpoName in $GpoNames) {
    if (Get-GPO -Name $GpoName -ErrorAction SilentlyContinue) {
        Get-GPPermission `
          -Name $GpoName `
          -All |
          Out-File "$ReportPath\$GpoName-permissions.txt"

        Get-GPOReport `
          -Name $GpoName `
          -ReportType Html `
          -Path "$ReportPath\$GpoName-final.html"

        Get-GPOReport `
          -Name $GpoName `
          -ReportType Xml `
          -Path "$ReportPath\$GpoName-final.xml"

        Backup-GPO `
          -Name $GpoName `
          -Path $BackupPath `
          -Comment "Final PKI GPO backup"
    }
}

# Search reports for PKI policy indicators.
Select-String `
  -Path "$ReportPath\*.xml" `
  -Pattern "Auto-Enrollment","AutoEnrollment","Public Key","Certificate Services Client","Trusted Root","Certificate Path Validation","Root","Intermediate" |
  Out-File "$ReportPath\pki-gpo-report-search.txt"

# Replication checks.
repadmin /replsummary |
  Out-File "$ReportPath\repadmin-replsummary-after-pki.txt"

dcdiag /test:sysvolcheck /test:advertising |
  Out-File "$ReportPath\dcdiag-sysvol-advertising-after-pki.txt"

Write-Host "PKI GPO reports and backups complete."
Write-Host "Report path: $ReportPath"
Write-Host "Backup path: $BackupPath"
```

# 21_Configure_Certificate_AutoEnrollment_And_PKIGPO_Settings_Verification_Commands
| Command | Purpose | Healthy Output |
|---|---|---|
| `certutil -config "<ca-config>" -ping` | Confirms CA service responds | CA ping succeeds |
| `Get-Service CertSvc` | Confirms Certificate Services service state | Service is running |
| `certutil -template` | Lists AD certificate templates | Templates return |
| `certutil -config "<ca-config>" -catemplates` | Lists templates published by CA | Target templates appear |
| `certtmpl.msc` | Opens certificate template console | Console opens |
| `certsrv.msc` | Opens Certification Authority console | Console opens |
| `certutil -enterprise -viewstore Root` | Views enterprise root store | Root CA appears |
| `certutil -enterprise -viewstore NTAuth` | Views NTAuth store | Issuing CA appears if required |
| `certutil -dspublish -f "<ca-cert-path>" NTAuthCA` | Publishes CA cert to NTAuth | Publication succeeds |
| `Get-GPO -Name "<computer-pki-gpo>"` | Confirms computer PKI GPO exists | GPO object returns |
| `Get-GPO -Name "<user-pki-gpo>"` | Confirms user PKI GPO exists | GPO object returns |
| `Get-GPInheritance -Target "<pilot-computer-ou-dn>"` | Confirms computer PKI GPO link | GPO appears |
| `Get-GPInheritance -Target "<pilot-user-ou-dn>"` | Confirms user PKI GPO link | GPO appears |
| `Get-GPOReport -Name "<gpo-name>" -ReportType Html -Path "<path>"` | Exports readable GPO report | HTML report exists |
| `Backup-GPO -Name "<gpo-name>" -Path "<backup-path>"` | Backs up PKI GPO | Backup completes |
| `gpupdate /target:computer /force` | Refreshes computer PKI policy | Policy refresh completes |
| `gpupdate /target:user /force` | Refreshes user PKI policy | Policy refresh completes |
| `certutil -pulse` | Triggers certificate auto-enrollment | Enrollment pulse runs |
| `Get-ChildItem Cert:\LocalMachine\My` | Validates computer certificates | Expected cert appears |
| `Get-ChildItem Cert:\CurrentUser\My` | Validates user certificates | Expected cert appears |
| `Get-ChildItem Cert:\LocalMachine\Root` | Validates root trust | Root CA cert appears |
| `Get-ChildItem Cert:\LocalMachine\CA` | Validates intermediate trust | Issuing CA cert appears if deployed |
| `certutil -verify "<cert.cer>"` | Validates certificate chain and revocation | Chain verifies |
| `Get-WinEvent -LogName "Microsoft-Windows-CertificateServicesClient-AutoEnrollment/Operational" -MaxEvents 100` | Reviews auto-enrollment events | Success or actionable errors appear |
| `Get-WinEvent -LogName "Microsoft-Windows-CAPI2/Operational" -MaxEvents 100` | Reviews chain and revocation detail | Chain diagnostics appear |
| `gpresult /h C:\GPOPrep\Reports\PKI\gpresult-pki.html` | Exports client effective policy report | HTML report exists |
| `repadmin /replsummary` | Checks AD replication | No unexpected failures |
| `dcdiag /test:sysvolcheck /test:advertising` | Checks SYSVOL and DC health | Tests pass |

# 21_Configure_Certificate_AutoEnrollment_And_PKIGPO_Settings_Rollback
```powershell
# Run on a domain controller, CA, or management host.
# Purpose: safely roll back PKI auto-enrollment GPO deployment from pilot scope.
# Do not remove CA templates or trust anchors from production without change approval.

Import-Module ActiveDirectory
Import-Module GroupPolicy

$DomainDN = (Get-ADDomain).DistinguishedName

$PilotComputerOU = "OU=Pilot,OU=Workstations,OU=Corp,$DomainDN"
$PilotUserOU = "OU=Users,OU=Corp,$DomainDN"

$GpoNames = @(
  "CORP-PKI-Computer-AutoEnrollment",
  "CORP-PKI-User-AutoEnrollment",
  "CORP-PKI-Trust-And-Path-Validation"
)

$ReportPath = "C:\GPOPrep\Reports\PKI-Rollback"
$BackupPath = "C:\GPOPrep\Backup\PKI-Rollback"

New-Item -ItemType Directory -Force -Path $ReportPath,$BackupPath

# Capture current state before rollback.
foreach ($GpoName in $GpoNames) {
    if (Get-GPO -Name $GpoName -ErrorAction SilentlyContinue) {
        Get-GPOReport `
          -Name $GpoName `
          -ReportType Html `
          -Path "$ReportPath\$GpoName-before-rollback.html"

        Backup-GPO `
          -Name $GpoName `
          -Path $BackupPath `
          -Comment "Before PKI GPO rollback"
    }
}

Get-GPInheritance `
  -Target $PilotComputerOU |
  Out-File "$ReportPath\pilot-computer-ou-inheritance-before-pki-rollback.txt"

Get-GPInheritance `
  -Target $PilotUserOU |
  Out-File "$ReportPath\pilot-user-ou-inheritance-before-pki-rollback.txt"

# Rollback option 1:
# Disable computer and trust GPO links from computer OU.
foreach ($GpoName in @("CORP-PKI-Computer-AutoEnrollment","CORP-PKI-Trust-And-Path-Validation")) {
    Set-GPLink `
      -Name $GpoName `
      -Target $PilotComputerOU `
      -LinkEnabled No `
      -ErrorAction SilentlyContinue
}

# Rollback option 2:
# Disable user GPO link from user OU.
Set-GPLink `
  -Name "CORP-PKI-User-AutoEnrollment" `
  -Target $PilotUserOU `
  -LinkEnabled No `
  -ErrorAction SilentlyContinue

# Rollback option 3:
# Use GPMC to set auto-enrollment policies back to Not Configured.
gpmc.msc

# GUI rollback workflow:
# 1. Edit computer PKI GPO.
# 2. Set Computer Configuration > Public Key Policies > Certificate Services Client - Auto-Enrollment to Not Configured.
# 3. Edit user PKI GPO.
# 4. Set User Configuration > Public Key Policies > Certificate Services Client - Auto-Enrollment to Not Configured.
# 5. Do not remove root or intermediate trust from broad scopes without validating dependency.
# 6. Export reports.
# 7. Run gpupdate and certutil -pulse on pilot client.
#
# Rollback option 4:
# Unpublish test certificate templates from CA if they were lab-only.
# certsrv.msc > CA > Certificate Templates > Delete selected test template from issued templates
#
# Rollback option 5:
# Remove issued test certificates from pilot client if lab-only.
# Use certlm.msc or certmgr.msc carefully and only for test certificates.

# Capture after rollback.
Get-GPInheritance `
  -Target $PilotComputerOU |
  Out-File "$ReportPath\pilot-computer-ou-inheritance-after-pki-rollback.txt"

Get-GPInheritance `
  -Target $PilotUserOU |
  Out-File "$ReportPath\pilot-user-ou-inheritance-after-pki-rollback.txt"

Write-Host "PKI GPO rollback workflow complete."
Write-Host "On pilot client: run gpupdate /force, certutil -pulse, and validate certificate stores."
```

# 21_Configure_Certificate_AutoEnrollment_And_PKIGPO_Settings_Failure_Checks
| Symptom | Likely Cause | Check Command | Fix |
|---|---|---|---|
| CA ping fails | CA offline, RPC blocked, wrong config string, or DNS issue | `certutil -config "<ca-config>" -ping`; `Resolve-DnsName <ca-server>` | Start CA service, fix DNS, firewall, or config string |
| Template does not appear on CA | Template not published | `certutil -config "<ca-config>" -catemplates` | Publish template in `certsrv.msc` |
| Template exists but client does not enroll | Missing Enroll or Autoenroll permission | Template Security tab | Grant Read, Enroll, and Autoenroll to correct group |
| Computer certificate does not enroll | Computer GPO not applied or computer lacks template permission | `gpresult /scope computer /r`; template permissions | Fix GPO scope or computer group permission |
| User certificate does not enroll | User GPO not applied or user lacks template permission | `gpresult /scope user /r`; template permissions | Fix GPO scope or user group permission |
| Auto-enrollment GPO missing from GPResult | Wrong OU link, filtering, WMI, or disabled link | `Get-GPInheritance`; `Get-GPPermission` | Fix link or filtering |
| Auto-enrollment events show access denied | Template, CA, or AD permission issue | AutoEnrollment log and template permissions | Correct template ACL or CA permissions |
| Auto-enrollment events show no template | Template not published or incompatible | `certutil -catemplates`; template compatibility | Publish compatible template |
| Certificate appears but authentication fails | Chain, EKU, subject, SAN, or NTAuth issue | `certutil -verify`; certificate details | Fix template EKU, subject, trust, or NTAuth |
| Chain fails | Missing root or intermediate trust | `certutil -verify`; Root and CA stores | Deploy root/intermediate certs |
| Revocation check fails | CRL/CDP unreachable or stale | `certutil -url <cert.cer>`; CAPI2 log | Fix CRL publication and client access |
| NTAuth missing | Issuing CA not published to NTAuth where required | `certutil -enterprise -viewstore NTAuth` | Publish CA cert to NTAuth |
| GPMC lacks PKI settings | RSAT/GPMC issue or ADMX problem | GPMC and Central Store | Fix management tools or ADMX Central Store |
| Root cert deployed to wrong store | Imported to CurrentUser or wrong GPO store | `Get-ChildItem Cert:\LocalMachine\Root` | Deploy through computer Public Key Policies |
| User cert appears on wrong user | Testing under wrong sign-in session | `whoami`; `Cert:\CurrentUser\My` | Sign in as intended test user |
| Computer cert subject wrong | Template subject name setting wrong | Certificate details and template Subject Name tab | Fix subject name template setting |
| Duplicate certs appear | Auto-enrollment renewal or template changes triggered new certs | Certificate store and template version | Clean up test certs and tune renewal settings |
| Old certificate remains after rollback | Disabling auto-enrollment does not remove existing certs automatically | Certificate store check | Remove test cert manually if lab-only |
| CAPI2 log empty | CAPI2 Operational log disabled | Event Viewer | Enable log and reproduce chain validation |
| AutoEnrollment log missing | Log disabled or no event generated yet | `Get-WinEvent -ListLog "*CertificateServicesClient*"` | Enable log or trigger `certutil -pulse` |
| Group membership not reflected | User or computer token stale | `whoami /groups`; `gpresult /r` | Sign out user or reboot computer |
| AD replication delay | Template, permission, or NTAuth change not on all DCs | `repadmin /replsummary` | Wait or fix AD replication |
| SYSVOL replication issue | GPO policy files inconsistent across DCs | `dcdiag /test:sysvolcheck`; `gpresult` | Fix SYSVOL/DFSR replication |
| Rollback does not stop enrollment | Another GPO still enables auto-enrollment | `gpresult /h` | Find winning GPO and disable there |

# 21_Configure_Certificate_AutoEnrollment_And_PKIGPO_Settings_Related_Labs
| Related Lab                                                               | Relationship                                                                   |
| ------------------------------------------------------------------------- | ------------------------------------------------------------------------------ |
| `00_Group_Policy_Index.md`                                                | Defines suite order and dependency path                                        |
| `01_Install_Group_Policy_Management_Tools.md`                             | Provides GPMC and GroupPolicy module                                           |
| `02_Create_And_Link_Baseline_GPO.md`                                      | Establishes GPO creation and linking workflow                                  |
| `03_Configure_GPO_Link_Order_Inheritance_Enforcement_And_Blocking.md`     | Controls precedence before PKI GPO rollout                                     |
| `04_Configure_GPO_Security_Filtering_And_Delegation.md`                   | Controls who can apply and edit PKI GPOs                                       |
| `06_Configure_Computer_Administrative_Template_Settings.md`               | Provides computer-side policy foundation                                       |
| `07_Configure_User_Administrative_Template_Settings.md`                   | Provides user-side policy foundation                                           |
| `11_Configure_Security_Baseline_GPO_Settings.md`                          | Security baseline companion workbook                                           |
| `14_Backup_Restore_Import_And_Report_GPOs.md`                             | Provides backup and restore workflow before PKI GPO changes                    |
| `15_Validate_Group_Policy_Application_With_GPUpdate_GPResult_And_RSoP.md` | Validates PKI GPO application on clients                                       |
| `16_Troubleshoot_Group_Policy_Processing_And_Replication.md`              | Diagnoses GPO processing, AD replication, and SYSVOL issues                    |
| `17_Configure_ADMX_Central_Store_And_Policy_Definitions.md`               | Ensures GPMC policy definitions are available                                  |
| `20_Configure_LAPS_And_Local_Administrator_Password_Policy.md`            | Related security policy deployment requiring AD permissions and GPO validation |
| `Verify_SYSVOL_NETLOGON_DC_Locator_And_FSMO_Roles.md`                     | Confirms domain infrastructure before PKI GPO rollout                          |
| `22_Configure_IPSec_Connection_Security_And_Advanced_Firewall_GPOs.md`    | Next workbook using certificates for secure network policy scenarios           |