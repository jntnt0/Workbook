22_Configure_Azure_File_Sync_For_Windows_File_Server.md
# Configure_Azure_File_Sync_For_Windows_File_Server

# Configure_Azure_File_Sync_For_Windows_File_Server_Index
22_Configure_Azure_File_Sync_For_Windows_File_Server.md
Configure_Azure_File_Sync_For_Windows_File_Server
Configure_Azure_File_Sync_For_Windows_File_Server_Source_Basis
Configure_Azure_File_Sync_For_Windows_File_Server_Mental_Model
Configure_Azure_File_Sync_For_Windows_File_Server_Planning_Table
Configure_Azure_File_Sync_For_Windows_File_Server_Configuration_Checklist
Configure_Azure_File_Sync_For_Windows_File_Server_Azure_Prereq_Skeleton
Configure_Azure_File_Sync_For_Windows_File_Server_Storage_Account_And_File_Share_Skeleton
Configure_Azure_File_Sync_For_Windows_File_Server_Storage_Sync_Service_Skeleton
Configure_Azure_File_Sync_For_Windows_File_Server_Agent_Install_Skeleton
Configure_Azure_File_Sync_For_Windows_File_Server_Server_Registration_Skeleton
Configure_Azure_File_Sync_For_Windows_File_Server_Sync_Group_And_Cloud_Endpoint_Skeleton
Configure_Azure_File_Sync_For_Windows_File_Server_Server_Endpoint_Skeleton
Configure_Azure_File_Sync_For_Windows_File_Server_Cloud_Tiering_Skeleton
Configure_Azure_File_Sync_For_Windows_File_Server_SMB_Validation_Skeleton
Configure_Azure_File_Sync_For_Windows_File_Server_Status_And_Event_Review_Skeleton
Configure_Azure_File_Sync_For_Windows_File_Server_Verification_Commands
Configure_Azure_File_Sync_For_Windows_File_Server_Rollback
Configure_Azure_File_Sync_For_Windows_File_Server_Failure_Checks
Configure_Azure_File_Sync_For_Windows_File_Server_Related_Labs

# Configure_Azure_File_Sync_For_Windows_File_Server_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Azure File Sync deployment guide | Deploy Azure File Sync | End-to-end deployment sequence |
| Azure File Sync planning guide | Azure File Sync management concepts | Storage Sync Service, registered servers, sync groups, cloud endpoints, server endpoints |
| Azure File Sync deployment guide | Prerequisites | Azure file share, storage account settings, Windows Server requirements, permissions |
| Azure File Sync deployment guide | Install Azure File Sync agent | Agent download and installation workflow |
| Azure File Sync deployment guide | Register Windows Server | Trust relationship between server and Storage Sync Service |
| Azure File Sync deployment guide | Create sync group and cloud endpoint | Cloud endpoint connection to Azure file share |
| Azure File Sync deployment guide | Create server endpoint | Local server path sync relationship |
| Azure File Sync cloud tiering overview | Cloud tiering policies | Volume free space policy and date policy |
| Azure File Sync planning guide | Networks | HTTPS 443 sync communication |
| Azure File Sync planning guide | Identity | AD-based ACL behavior and optional storage account domain join |
| Operational file server practice | Hybrid file server operations | Validating SMB access, sync status, rollback, and user impact |

# Configure_Azure_File_Sync_For_Windows_File_Server_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Azure File Sync | Hybrid file service that syncs Windows Server file paths with Azure Files |
| Azure file share | Cloud-side file share that acts as the hub of the sync group |
| Storage Sync Service | Azure management resource that registers servers and owns sync topology |
| Registered server | Windows Server trusted by one Storage Sync Service |
| Sync group | Logical sync relationship containing one cloud endpoint and one or more server endpoints |
| Cloud endpoint | Azure file share attached to the sync group |
| Server endpoint | Local Windows Server path attached to the sync group |
| Cloud tiering | Optional feature that keeps namespace local and tiers cold file content to Azure Files |
| Volume free space policy | Cloud tiering policy that keeps a defined amount of local disk free |
| Date policy | Cloud tiering policy that tiers files after a defined number of days without access |
| Initial upload policy | Determines how local server data merges or overwrites cloud data on first sync |
| Initial download policy | Determines how a server downloads namespace and file content from cloud |
| Agent | Azure File Sync software installed on the Windows file server |
| Local cache | Windows file server path users keep accessing over normal SMB |
| Hub model | Azure file share is the cloud hub, servers are edge caches or full copies |
| Port 443 | Azure File Sync agent uses HTTPS-based protocols to Azure |
| Not SMB-to-Azure | The sync agent does not upload or download through SMB port 445 |
| Blunt rule | Users should normally keep accessing the Windows file server share, not bypass the design by mounting the Azure file share directly |
| Design rule | Do not point a server endpoint at the wrong folder. The endpoint path and drive letter are design decisions, not casual settings |

# Configure_Azure_File_Sync_For_Windows_File_Server_Planning_Table
| Item | Example | Decision |
|---|---|---|
| Azure subscription | `Visual Studio Enterprise` | `<subscription-name>` |
| Tenant | `contoso.onmicrosoft.com` | `<tenant-name>` |
| Azure region | `eastus` | `<azure-region>` |
| Resource group | `rg-filesync-lab-eastus-001` | `<resource-group>` |
| Storage Sync Service | `sss-filesync-lab-eastus-001` | `<storage-sync-service-name>` |
| Storage account | `stfilesynclab001` | `<storage-account-name>` |
| Azure file share | `departments` | `<azure-file-share-name>` |
| File share quota | `1024 GiB` | `<share-quota>` |
| Storage redundancy | `LRS` | `<redundancy>` |
| Storage account kind | `StorageV2` or `FileStorage` | `<storage-account-kind>` |
| Windows file server | `FS1` | `<file-server-name>` |
| File server FQDN | `FS1.corp.local` | `<file-server-fqdn>` |
| Existing SMB share | `Departments` | `<smb-share-name>` |
| Existing SMB path | `D:\Shares\Departments` | `<server-endpoint-path>` |
| Server endpoint name | `FS1-Departments` | `<server-endpoint-name>` |
| Sync group | `sg-departments` | `<sync-group-name>` |
| Cloud endpoint name | `ce-departments` | `<cloud-endpoint-name>` |
| Registered server name | `FS1` | `<registered-server-name>` |
| Cloud tiering | Enabled / Disabled | `<cloud-tiering-state>` |
| Volume free space percent | `20` | `<volume-free-space-percent>` |
| Tier files older than days | `30` | `<tier-files-older-than-days>` |
| Initial upload policy | `Merge` | `<initial-upload-policy>` |
| Initial download policy | `NamespaceOnly` | `<initial-download-policy>` |
| Test client | `WIN11-01` | `<client-name>` |
| Test file | `afs-test.txt` | `<test-file>` |
| Report path | `C:\AzureFileSync-Reports` | `<report-path>` |
| Change ticket | `CHG000123` | `<ticket-id>` |

# Configure_Azure_File_Sync_For_Windows_File_Server_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm file server identity | File Server | `hostname` | Correct Windows file server is identified |
| 2 | Confirm server OS | File Server | `Get-ComputerInfo | Select WindowsProductName,WindowsVersion,OsHardwareAbstractionLayer` | Windows Server version is known |
| 3 | Confirm PowerShell version | File Server | `$PSVersionTable.PSVersion` | PowerShell 5.1 or later is present |
| 4 | Confirm local endpoint path | File Server | `Test-Path "D:\Shares\Departments"` | Server endpoint path exists |
| 5 | Confirm local volume health | File Server | `Get-Volume -DriveLetter D` | Volume is healthy and online |
| 6 | Confirm existing SMB share | File Server | `Get-SmbShare -Name "Departments"` | Existing share is present |
| 7 | Confirm SMB access before sync | Client | `Test-Path "\\FS1\Departments"` | Existing SMB path works before Azure File Sync |
| 8 | Confirm Azure PowerShell module | Admin Host | `Get-Module Az -ListAvailable` | Az module is installed |
| 9 | Sign in to Azure | Admin Host | `Connect-AzAccount` | Azure session is authenticated |
| 10 | Select subscription | Admin Host | `Set-AzContext -Subscription "<subscription-name>"` | Correct subscription is active |
| 11 | Confirm provider registration | Admin Host | `Get-AzResourceProvider -ProviderNamespace Microsoft.StorageSync` | StorageSync provider state is visible |
| 12 | Register provider if needed | Admin Host | `Register-AzResourceProvider -ProviderNamespace Microsoft.StorageSync` | Provider registration starts |
| 13 | Create resource group | Admin Host | `New-AzResourceGroup -Name "<resource-group>" -Location "<azure-region>"` | Resource group exists |
| 14 | Create storage account | Admin Host | `New-AzStorageAccount -ResourceGroupName "<resource-group>" -Name "<storage-account-name>" -Location "<azure-region>" -SkuName Standard_LRS -Kind StorageV2 -EnableHttpsTrafficOnly $true` | Storage account exists |
| 15 | Confirm storage account key access | Azure Portal | `Storage account > Configuration > Allow storage account key access: Enabled` | Key access is enabled for Azure File Sync |
| 16 | Confirm SMB security settings | Azure Portal | `Storage account > File shares > SMB security settings` | SMB 3.1.1, NTLMv2, and AES-128-GCM are allowed |
| 17 | Create Azure file share | Admin Host | `New-AzStorageShare -Name "<azure-file-share-name>" -Context $StorageAccount.Context` | Azure file share exists |
| 18 | Create Storage Sync Service | Admin Host | `New-AzStorageSyncService -ResourceGroupName "<resource-group>" -Name "<storage-sync-service-name>" -Location "<azure-region>"` | Storage Sync Service exists |
| 19 | Confirm Storage Sync Service | Admin Host | `Get-AzStorageSyncService -ResourceGroupName "<resource-group>" -Name "<storage-sync-service-name>"` | Service is visible |
| 20 | Confirm IAM for operator | Azure Portal | `Storage Sync Service IAM and Storage Account IAM` | Operator has required registration and cloud endpoint permissions |
| 21 | Download Azure File Sync agent | File Server | `Invoke-WebRequest -Uri https://aka.ms/afs/agent/Server2022 -OutFile C:\Temp\StorageSyncAgent.msi` | Agent MSI is downloaded for matching OS |
| 22 | Install Azure File Sync agent | File Server | `Start-Process msiexec.exe -ArgumentList "/i C:\Temp\StorageSyncAgent.msi /quiet /norestart" -Wait` | Agent installs |
| 23 | Confirm agent services | File Server | `Get-Service | Where-Object DisplayName -like "*Storage Sync*"` | Storage Sync services are visible |
| 24 | Import Az.StorageSync | File Server | `Import-Module Az.StorageSync` | Storage Sync Azure cmdlets are loaded |
| 25 | Register server | File Server | `Register-AzStorageSyncServer -ParentObject $StorageSyncService` | Server is registered with Storage Sync Service |
| 26 | Confirm registered server | Admin Host | `Get-AzStorageSyncServer -ResourceGroupName "<resource-group>" -StorageSyncServiceName "<storage-sync-service-name>"` | Registered server appears |
| 27 | Create sync group | Admin Host | `New-AzStorageSyncGroup -ParentObject $StorageSyncService -Name "<sync-group-name>"` | Sync group exists |
| 28 | Create cloud endpoint | Admin Host | `New-AzStorageSyncCloudEndpoint -Name "<cloud-endpoint-name>" -ParentObject $SyncGroup -StorageAccountResourceId $StorageAccount.Id -AzureFileShareName "<azure-file-share-name>"` | Cloud endpoint exists |
| 29 | Create server endpoint without cloud tiering | Admin Host | `New-AzStorageSyncServerEndpoint -Name "<server-endpoint-name>" -SyncGroup $SyncGroup -ServerResourceId $RegisteredServer.ResourceId -ServerLocalPath "D:\Shares\Departments" -InitialUploadPolicy Merge` | Server endpoint is created |
| 30 | Or create server endpoint with cloud tiering | Admin Host | `New-AzStorageSyncServerEndpoint -Name "<server-endpoint-name>" -SyncGroup $SyncGroup -ServerResourceId $RegisteredServer.ResourceId -ServerLocalPath "D:\Shares\Departments" -CloudTiering -VolumeFreeSpacePercent 20 -TierFilesOlderThanDays 30 -InitialDownloadPolicy NamespaceOnly -InitialUploadPolicy Merge` | Server endpoint is created with cloud tiering |
| 31 | Confirm sync group endpoints | Admin Host | `Get-AzStorageSyncServerEndpoint -ResourceGroupName "<resource-group>" -StorageSyncServiceName "<storage-sync-service-name>" -SyncGroupName "<sync-group-name>"` | Server endpoint is visible |
| 32 | Create local test file | Client | `"afs-test" | Out-File "\\FS1\Departments\afs-test.txt"` | File writes through SMB |
| 33 | Confirm local file on server | File Server | `Get-Item "D:\Shares\Departments\afs-test.txt"` | File exists locally |
| 34 | Confirm file in Azure share | Admin Host | `Get-AzStorageFile -ShareName "<azure-file-share-name>" -Path "." -Context $StorageAccount.Context` | File appears after sync completes |
| 35 | Create cloud-side test file | Admin Host | `New-AzStorageFileContent -ShareName "<azure-file-share-name>" -Source "C:\Temp\cloud-test.txt" -Path "." -Context $StorageAccount.Context` | File is uploaded to Azure file share |
| 36 | Confirm cloud file syncs down | File Server | `Get-ChildItem "D:\Shares\Departments"` | Cloud-side file appears after sync processing |
| 37 | Confirm SMB users still use local share | Client | `Test-Path "\\FS1\Departments"` | Local file server access still works |
| 38 | Review agent events | File Server | `Get-WinEvent -ListLog "*StorageSync*","*FileSync*"` | Azure File Sync logs are discoverable |
| 39 | Export configuration evidence | Admin Host | `Get-AzStorageSyncService -ResourceGroupName "<resource-group>" | Export-Clixml "C:\AzureFileSync-Reports\storage-sync-service.xml"` | Evidence is saved |
| 40 | Document final state | Operator | `Record service, storage account, share, sync group, endpoints, tiering policy, test files, and rollback` | Change record is complete |

# Configure_Azure_File_Sync_For_Windows_File_Server_Azure_Prereq_Skeleton
```powershell
# Run on admin workstation or file server with Azure PowerShell.
# Purpose: authenticate, select subscription, and validate Azure File Sync provider availability.

$SubscriptionName = "<subscription-name>"
$Region = "eastus"
$ResourceGroupName = "rg-filesync-lab-eastus-001"

Write-Host "Signing in to Azure..."
Connect-AzAccount

Write-Host "Selecting subscription..."
Set-AzContext -Subscription $SubscriptionName

Write-Host "Current Azure context:"
Get-AzContext

Write-Host "Checking Microsoft.StorageSync provider..."
Get-AzResourceProvider -ProviderNamespace Microsoft.StorageSync |
  Select-Object ProviderNamespace, RegistrationState

Write-Host "Registering Microsoft.StorageSync provider if required..."
Register-AzResourceProvider -ProviderNamespace Microsoft.StorageSync

Write-Host "Checking whether Azure File Sync is available in selected region..."
$StorageSyncRegionAvailable = (Get-AzLocation | Where-Object {
  $_.Location -eq $Region -and $_.Providers -contains "Microsoft.StorageSync"
})

if (-not $StorageSyncRegionAvailable) {
  throw "Microsoft.StorageSync is not available in region $Region or the region name is wrong."
}

Write-Host "Creating resource group if missing..."
if (-not (Get-AzResourceGroup -Name $ResourceGroupName -ErrorAction SilentlyContinue)) {
  New-AzResourceGroup -Name $ResourceGroupName -Location $Region
}

Get-AzResourceGroup -Name $ResourceGroupName
```

# Configure_Azure_File_Sync_For_Windows_File_Server_Storage_Account_And_File_Share_Skeleton
```powershell
# Run on admin workstation or file server with Azure PowerShell.
# Purpose: create the storage account and Azure file share used as the cloud endpoint.

$ResourceGroupName = "rg-filesync-lab-eastus-001"
$Region = "eastus"
$StorageAccountName = "stfilesynclab001"
$FileShareName = "departments"

Write-Host "Getting or creating storage account..."
$StorageAccount = Get-AzStorageAccount `
  -ResourceGroupName $ResourceGroupName `
  -Name $StorageAccountName `
  -ErrorAction SilentlyContinue

if (-not $StorageAccount) {
  $StorageAccount = New-AzStorageAccount `
    -ResourceGroupName $ResourceGroupName `
    -Name $StorageAccountName `
    -Location $Region `
    -SkuName Standard_LRS `
    -Kind StorageV2 `
    -EnableHttpsTrafficOnly $true
}

Write-Host "Storage account:"
$StorageAccount | Select-Object StorageAccountName, Location, Sku, Kind, EnableHttpsTrafficOnly

Write-Host "Creating Azure file share if missing..."
$ExistingShare = Get-AzStorageShare `
  -Context $StorageAccount.Context `
  -Name $FileShareName `
  -ErrorAction SilentlyContinue

if (-not $ExistingShare) {
  New-AzStorageShare `
    -Context $StorageAccount.Context `
    -Name $FileShareName
}

Write-Host "Azure file shares in storage account:"
Get-AzStorageShare -Context $StorageAccount.Context |
  Select-Object Name, Quota, IsSnapshot
```

# Configure_Azure_File_Sync_For_Windows_File_Server_Storage_Sync_Service_Skeleton
```powershell
# Run on admin workstation or file server with Azure PowerShell.
# Purpose: create the Storage Sync Service management resource.

$ResourceGroupName = "rg-filesync-lab-eastus-001"
$Region = "eastus"
$StorageSyncServiceName = "sss-filesync-lab-eastus-001"

Write-Host "Getting or creating Storage Sync Service..."
$StorageSyncService = Get-AzStorageSyncService `
  -ResourceGroupName $ResourceGroupName `
  -Name $StorageSyncServiceName `
  -ErrorAction SilentlyContinue

if (-not $StorageSyncService) {
  $StorageSyncService = New-AzStorageSyncService `
    -ResourceGroupName $ResourceGroupName `
    -Name $StorageSyncServiceName `
    -Location $Region
}

Write-Host "Storage Sync Service:"
$StorageSyncService | Format-List *
```

# Configure_Azure_File_Sync_For_Windows_File_Server_Agent_Install_Skeleton
```powershell
# Run in elevated PowerShell on the Windows file server.
# Purpose: download and install the correct Azure File Sync agent for the server OS.

$DownloadPath = "C:\Temp"
$MsiPath = Join-Path $DownloadPath "StorageSyncAgent.msi"

New-Item -ItemType Directory -Force -Path $DownloadPath | Out-Null

$OSVersion = [System.Environment]::OSVersion.Version

Write-Host "Detected OS version: $OSVersion"

if ($OSVersion.Equals([System.Version]::new(10, 0, 20348, 0))) {
  $AgentUri = "https://aka.ms/afs/agent/Server2022"
}
elseif ($OSVersion.Equals([System.Version]::new(10, 0, 17763, 0))) {
  $AgentUri = "https://aka.ms/afs/agent/Server2019"
}
elseif ($OSVersion.Equals([System.Version]::new(10, 0, 14393, 0))) {
  $AgentUri = "https://aka.ms/afs/agent/Server2016"
}
elseif ($OSVersion.Equals([System.Version]::new(6, 3, 9600, 0))) {
  $AgentUri = "https://aka.ms/afs/agent/Server2012R2"
}
else {
  throw "This skeleton only handles the common Azure File Sync agent download links for Server 2012 R2, 2016, 2019, and 2022. Confirm the current agent link manually for this OS."
}

Write-Host "Downloading Azure File Sync agent..."
Invoke-WebRequest -Uri $AgentUri -OutFile $MsiPath

Write-Host "Installing Azure File Sync agent..."
Start-Process `
  -FilePath "msiexec.exe" `
  -ArgumentList "/i `"$MsiPath`" /quiet /norestart" `
  -Wait

Write-Host "Checking Storage Sync services..."
Get-Service | Where-Object {
  $_.DisplayName -like "*Storage Sync*" -or $_.Name -like "*StorageSync*"
} | Format-Table Name, DisplayName, Status, StartType -AutoSize

Write-Host "Checking installed agent folder..."
Test-Path "C:\Program Files\Azure\StorageSyncAgent"
```

# Configure_Azure_File_Sync_For_Windows_File_Server_Server_Registration_Skeleton
```powershell
# Run on the Windows file server after the Azure File Sync agent is installed.
# Purpose: register the Windows Server with the Storage Sync Service.

$SubscriptionName = "<subscription-name>"
$ResourceGroupName = "rg-filesync-lab-eastus-001"
$StorageSyncServiceName = "sss-filesync-lab-eastus-001"

Write-Host "Signing in to Azure..."
Connect-AzAccount

Write-Host "Selecting subscription..."
Set-AzContext -Subscription $SubscriptionName

Write-Host "Importing Az.StorageSync..."
Import-Module Az.StorageSync

Write-Host "Getting Storage Sync Service..."
$StorageSyncService = Get-AzStorageSyncService `
  -ResourceGroupName $ResourceGroupName `
  -Name $StorageSyncServiceName

Write-Host "Registering this Windows Server..."
$RegisteredServer = Register-AzStorageSyncServer -ParentObject $StorageSyncService

Write-Host "Registered server:"
$RegisteredServer | Format-List *

Write-Host "Confirming registered servers:"
Get-AzStorageSyncServer `
  -ResourceGroupName $ResourceGroupName `
  -StorageSyncServiceName $StorageSyncServiceName |
  Format-Table FriendlyName, ServerId, ResourceId -AutoSize
```

# Configure_Azure_File_Sync_For_Windows_File_Server_Sync_Group_And_Cloud_Endpoint_Skeleton
```powershell
# Run on admin workstation or file server with Azure PowerShell.
# Purpose: create sync group and cloud endpoint backed by the Azure file share.

$ResourceGroupName = "rg-filesync-lab-eastus-001"
$StorageSyncServiceName = "sss-filesync-lab-eastus-001"
$SyncGroupName = "sg-departments"
$CloudEndpointName = "ce-departments"
$StorageAccountName = "stfilesynclab001"
$FileShareName = "departments"

Write-Host "Getting Storage Sync Service..."
$StorageSyncService = Get-AzStorageSyncService `
  -ResourceGroupName $ResourceGroupName `
  -Name $StorageSyncServiceName

Write-Host "Getting or creating sync group..."
$SyncGroup = Get-AzStorageSyncGroup `
  -ResourceGroupName $ResourceGroupName `
  -StorageSyncServiceName $StorageSyncServiceName `
  -Name $SyncGroupName `
  -ErrorAction SilentlyContinue

if (-not $SyncGroup) {
  $SyncGroup = New-AzStorageSyncGroup `
    -ParentObject $StorageSyncService `
    -Name $SyncGroupName
}

Write-Host "Getting storage account..."
$StorageAccount = Get-AzStorageAccount `
  -ResourceGroupName $ResourceGroupName `
  -Name $StorageAccountName

Write-Host "Creating cloud endpoint..."
$CloudEndpoint = New-AzStorageSyncCloudEndpoint `
  -Name $CloudEndpointName `
  -ParentObject $SyncGroup `
  -StorageAccountResourceId $StorageAccount.Id `
  -AzureFileShareName $FileShareName

Write-Host "Cloud endpoint:"
$CloudEndpoint | Format-List *
```

# Configure_Azure_File_Sync_For_Windows_File_Server_Server_Endpoint_Skeleton
```powershell
# Run on admin workstation or file server with Azure PowerShell.
# Purpose: create the server endpoint for the local Windows file server path.

$ResourceGroupName = "rg-filesync-lab-eastus-001"
$StorageSyncServiceName = "sss-filesync-lab-eastus-001"
$SyncGroupName = "sg-departments"
$ServerEndpointName = "FS1-Departments"
$ServerEndpointPath = "D:\Shares\Departments"
$RegisteredServerName = "FS1"

Write-Host "Getting sync group..."
$SyncGroup = Get-AzStorageSyncGroup `
  -ResourceGroupName $ResourceGroupName `
  -StorageSyncServiceName $StorageSyncServiceName `
  -Name $SyncGroupName

Write-Host "Getting registered server..."
$RegisteredServer = Get-AzStorageSyncServer `
  -ResourceGroupName $ResourceGroupName `
  -StorageSyncServiceName $StorageSyncServiceName |
  Where-Object FriendlyName -eq $RegisteredServerName

if (-not $RegisteredServer) {
  throw "Registered server $RegisteredServerName not found."
}

Write-Host "Creating server endpoint without cloud tiering..."
$ServerEndpoint = New-AzStorageSyncServerEndpoint `
  -Name $ServerEndpointName `
  -SyncGroup $SyncGroup `
  -ServerResourceId $RegisteredServer.ResourceId `
  -ServerLocalPath $ServerEndpointPath `
  -InitialUploadPolicy Merge `
  -InitialDownloadPolicy NamespaceThenModifiedFiles

Write-Host "Server endpoint:"
$ServerEndpoint | Format-List *
```

# Configure_Azure_File_Sync_For_Windows_File_Server_Cloud_Tiering_Skeleton
```powershell
# Run on admin workstation or file server with Azure PowerShell.
# Purpose: create a server endpoint with cloud tiering enabled.
# Use this instead of the non-tiering server endpoint skeleton when local cache size matters.

$ResourceGroupName = "rg-filesync-lab-eastus-001"
$StorageSyncServiceName = "sss-filesync-lab-eastus-001"
$SyncGroupName = "sg-departments"
$ServerEndpointName = "FS1-Departments"
$ServerEndpointPath = "D:\Shares\Departments"
$RegisteredServerName = "FS1"
$VolumeFreeSpacePercent = 20
$TierFilesOlderThanDays = 30

$EndpointRoot = [System.IO.Directory]::GetDirectoryRoot($ServerEndpointPath)
$OSRoot = "$($env:SystemDrive)\"

if ($EndpointRoot -eq $OSRoot) {
  throw "Do not enable cloud tiering on the system volume."
}

$SyncGroup = Get-AzStorageSyncGroup `
  -ResourceGroupName $ResourceGroupName `
  -StorageSyncServiceName $StorageSyncServiceName `
  -Name $SyncGroupName

$RegisteredServer = Get-AzStorageSyncServer `
  -ResourceGroupName $ResourceGroupName `
  -StorageSyncServiceName $StorageSyncServiceName |
  Where-Object FriendlyName -eq $RegisteredServerName

if (-not $RegisteredServer) {
  throw "Registered server $RegisteredServerName not found."
}

Write-Host "Creating server endpoint with cloud tiering..."
$ServerEndpoint = New-AzStorageSyncServerEndpoint `
  -Name $ServerEndpointName `
  -SyncGroup $SyncGroup `
  -ServerResourceId $RegisteredServer.ResourceId `
  -ServerLocalPath $ServerEndpointPath `
  -CloudTiering `
  -VolumeFreeSpacePercent $VolumeFreeSpacePercent `
  -TierFilesOlderThanDays $TierFilesOlderThanDays `
  -InitialUploadPolicy Merge `
  -InitialDownloadPolicy NamespaceOnly

$ServerEndpoint | Format-List *
```

# Configure_Azure_File_Sync_For_Windows_File_Server_SMB_Validation_Skeleton
```powershell
# Run from a domain client that normally accesses the file server.
# Purpose: prove users continue to access the local Windows file server path.

$FileServer = "FS1"
$ShareName = "Departments"
$UncPath = "\\$FileServer\$ShareName"
$TestFile = Join-Path $UncPath "afs-client-test.txt"

Write-Host "Confirming logged-on user..."
whoami

Write-Host "Testing SMB port to local file server..."
Test-NetConnection $FileServer -Port 445

Write-Host "Testing UNC access..."
Test-Path $UncPath

Write-Host "Writing test file through SMB..."
"Azure File Sync SMB test from $env:COMPUTERNAME by $env:USERNAME at $(Get-Date)" |
  Out-File -FilePath $TestFile -Encoding utf8

Write-Host "Reading test file through SMB..."
Get-Content $TestFile

Write-Host "SMB connection details:"
Get-SmbConnection | Where-Object ServerName -like "*$FileServer*"
```

# Configure_Azure_File_Sync_For_Windows_File_Server_Status_And_Event_Review_Skeleton
```powershell
# Run on the file server and admin host as appropriate.
# Purpose: export Azure File Sync resource state, local services, and event evidence.

$ResourceGroupName = "rg-filesync-lab-eastus-001"
$StorageSyncServiceName = "sss-filesync-lab-eastus-001"
$SyncGroupName = "sg-departments"
$ReportRoot = "C:\AzureFileSync-Reports"
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"

New-Item -ItemType Directory -Force -Path $ReportRoot | Out-Null

Write-Host "Export Azure Storage Sync Service..."
Get-AzStorageSyncService `
  -ResourceGroupName $ResourceGroupName `
  -Name $StorageSyncServiceName |
  Export-Clixml "$ReportRoot\storage-sync-service-$Timestamp.xml"

Write-Host "Export registered servers..."
Get-AzStorageSyncServer `
  -ResourceGroupName $ResourceGroupName `
  -StorageSyncServiceName $StorageSyncServiceName |
  Export-Clixml "$ReportRoot\registered-servers-$Timestamp.xml"

Write-Host "Export sync groups..."
Get-AzStorageSyncGroup `
  -ResourceGroupName $ResourceGroupName `
  -StorageSyncServiceName $StorageSyncServiceName |
  Export-Clixml "$ReportRoot\sync-groups-$Timestamp.xml"

Write-Host "Export server endpoints..."
Get-AzStorageSyncServerEndpoint `
  -ResourceGroupName $ResourceGroupName `
  -StorageSyncServiceName $StorageSyncServiceName `
  -SyncGroupName $SyncGroupName |
  Export-Clixml "$ReportRoot\server-endpoints-$Timestamp.xml"

Write-Host "Export local Storage Sync services..."
Get-Service | Where-Object {
  $_.DisplayName -like "*Storage Sync*" -or $_.Name -like "*StorageSync*"
} |
  Select-Object Name, DisplayName, Status, StartType |
  Export-Csv "$ReportRoot\local-storage-sync-services-$Timestamp.csv" -NoTypeInformation

Write-Host "Export Storage Sync event log inventory..."
Get-WinEvent -ListLog "*StorageSync*","*FileSync*" -ErrorAction SilentlyContinue |
  Select-Object LogName, IsEnabled, RecordCount |
  Export-Csv "$ReportRoot\afs-eventlog-inventory-$Timestamp.csv" -NoTypeInformation

Write-Host "Export recent Storage Sync events..."
Get-WinEvent -ListLog "*StorageSync*","*FileSync*" -ErrorAction SilentlyContinue |
  ForEach-Object {
    $LogName = $_.LogName
    Get-WinEvent -LogName $LogName -MaxEvents 200 -ErrorAction SilentlyContinue |
      Select-Object TimeCreated, Id, ProviderName, LevelDisplayName, Message |
      Export-Csv "$ReportRoot\$($LogName.Replace('/','-'))-$Timestamp.csv" -NoTypeInformation
  }

Write-Host "Azure File Sync report data exported to $ReportRoot"
```

# Configure_Azure_File_Sync_For_Windows_File_Server_Verification_Commands
| Task | Command | Expected Result |
|---|---|---|
| Verify local SMB share | `Get-SmbShare -Name "Departments"` | Existing SMB share is present |
| Verify local endpoint path | `Test-Path "D:\Shares\Departments"` | Server endpoint path exists |
| Verify Azure context | `Get-AzContext` | Correct tenant and subscription are active |
| Verify provider | `Get-AzResourceProvider -ProviderNamespace Microsoft.StorageSync` | Provider is registered |
| Verify resource group | `Get-AzResourceGroup -Name "<resource-group>"` | Resource group exists |
| Verify storage account | `Get-AzStorageAccount -ResourceGroupName "<resource-group>" -Name "<storage-account-name>"` | Storage account exists |
| Verify Azure file share | `Get-AzStorageShare -Name "<azure-file-share-name>" -Context $StorageAccount.Context` | Azure file share exists |
| Verify Storage Sync Service | `Get-AzStorageSyncService -ResourceGroupName "<resource-group>" -Name "<storage-sync-service-name>"` | Storage Sync Service exists |
| Verify agent services | `Get-Service | Where-Object DisplayName -like "*Storage Sync*"` | Agent services are visible |
| Verify registered server | `Get-AzStorageSyncServer -ResourceGroupName "<resource-group>" -StorageSyncServiceName "<storage-sync-service-name>"` | File server appears as registered |
| Verify sync group | `Get-AzStorageSyncGroup -ResourceGroupName "<resource-group>" -StorageSyncServiceName "<storage-sync-service-name>"` | Sync group exists |
| Verify server endpoint | `Get-AzStorageSyncServerEndpoint -ResourceGroupName "<resource-group>" -StorageSyncServiceName "<storage-sync-service-name>" -SyncGroupName "<sync-group-name>"` | Server endpoint exists |
| Verify SMB path still works | `Test-Path "\\FS1\Departments"` | Client can access local SMB path |
| Verify local-to-cloud sync | `Get-AzStorageFile -ShareName "<azure-file-share-name>" -Path "." -Context $StorageAccount.Context` | Test file eventually appears in cloud |
| Verify cloud-to-local sync | `Get-ChildItem "D:\Shares\Departments"` | Cloud-side test file eventually appears locally |
| Verify HTTPS to Azure generally | `Test-NetConnection management.azure.com -Port 443` | Outbound HTTPS works |
| Verify event logs | `Get-WinEvent -ListLog "*StorageSync*","*FileSync*"` | Sync-related logs are discoverable |
| Verify report output | `Test-Path "C:\AzureFileSync-Reports"` | Evidence folder exists |

# Configure_Azure_File_Sync_For_Windows_File_Server_Rollback
| Change | Rollback Command | Expected Result |
|---|---|---|
| Client test file created | `Remove-Item "\\FS1\Departments\afs-client-test.txt" -Force` | Test file is removed through SMB |
| Cloud test file uploaded | `Remove-AzStorageFile -ShareName "<azure-file-share-name>" -Path "cloud-test.txt" -Context $StorageAccount.Context` | Cloud test file is removed |
| Server endpoint created | `Remove-AzStorageSyncServerEndpoint -ResourceGroupName "<resource-group>" -StorageSyncServiceName "<storage-sync-service-name>" -SyncGroupName "<sync-group-name>" -Name "<server-endpoint-name>"` | Local path is removed from sync topology |
| Cloud endpoint created | `Remove-AzStorageSyncCloudEndpoint -ResourceGroupName "<resource-group>" -StorageSyncServiceName "<storage-sync-service-name>" -SyncGroupName "<sync-group-name>" -Name "<cloud-endpoint-name>"` | Azure file share is detached from sync group |
| Sync group created | `Remove-AzStorageSyncGroup -ResourceGroupName "<resource-group>" -StorageSyncServiceName "<storage-sync-service-name>" -Name "<sync-group-name>"` | Sync group is removed |
| Server registered | `Unregister-AzStorageSyncServer -ResourceGroupName "<resource-group>" -StorageSyncServiceName "<storage-sync-service-name>" -ServerId "<server-id>"` | Server trust is removed after endpoints are removed |
| Azure File Sync agent installed | `Start-Process msiexec.exe -ArgumentList "/x C:\Temp\StorageSyncAgent.msi /quiet /norestart" -Wait` | Agent is uninstalled if MSI path is available |
| Azure file share created | `Remove-AzStorageShare -Name "<azure-file-share-name>" -Context $StorageAccount.Context -Force` | Azure file share is removed after data is protected |
| Storage account created | `Remove-AzStorageAccount -ResourceGroupName "<resource-group>" -Name "<storage-account-name>" -Force` | Storage account is removed after data is protected |
| Storage Sync Service created | `Remove-AzStorageSyncService -ResourceGroupName "<resource-group>" -Name "<storage-sync-service-name>"` | Storage Sync Service is removed |
| Resource group created | `Remove-AzResourceGroup -Name "<resource-group>" -Force` | Full lab resource group is removed |
| Report folder created | `Remove-Item "C:\AzureFileSync-Reports" -Recurse -Force` | Report folder is removed |

# Configure_Azure_File_Sync_For_Windows_File_Server_Failure_Checks
| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| `New-AzStorageSyncService` not found | Az.StorageSync module missing | `Get-Module Az.StorageSync -ListAvailable` | Install or update Az PowerShell |
| Provider unavailable in region | Region does not support StorageSync or name is wrong | `Get-AzLocation | Where-Object Providers -contains "Microsoft.StorageSync"` | Pick supported region |
| Agent install fails | Unsupported OS, missing updates, blocked download, or old MSI | OS version and Windows Update status | Patch server and use current agent link |
| Server registration fails | Wrong Azure account, missing IAM, browser auth issue, or proxy issue | `Get-AzContext`; IAM role check | Grant Azure File Sync Administrator, Owner, or Contributor as required |
| Server already registered elsewhere | Server can register to only one Storage Sync Service | Registered server list in Azure | Unregister from old service only after removing endpoints |
| Cloud endpoint creation fails | Missing storage account permissions or storage account setting mismatch | Storage account IAM and configuration | Grant required permissions and enable key access |
| Server endpoint creation fails | Path does not exist, path is not local, overlapping endpoint, or system volume tiering issue | `Test-Path`; endpoint list | Use final supported local path |
| Cloud tiering creation fails | System volume or invalid policy | Drive letter and tiering parameters | Use data volume and valid free space/date policy |
| Sync does not start | Agent service stopped, registration issue, endpoint unhealthy, or network blocked | Services, Azure endpoint health, event logs | Start services and fix registration/network |
| Files not appearing in cloud | Sync backlog, conflict, file lock, unsupported name, or endpoint unhealthy | Event logs and endpoint status | Resolve file issue and wait for sync |
| Cloud-side change slow to appear | Cloud change detection is not instant | Endpoint and cloud change timing | Prefer local server path for user changes |
| SMB users lose access | Separate NTFS or SMB issue, not normal Azure File Sync behavior | `icacls`; `Get-SmbShareAccess`; `Test-Path` | Fix local ACL or share permission |
| Users bypass file server and mount Azure share | Direct cloud access not intended or identity not configured | Storage account auth settings | Keep access through local file server unless cloud SMB identity is designed |
| Port 445 blocked to Azure | Direct Azure Files SMB access issue | `Test-NetConnection <storageaccount>.file.core.windows.net -Port 445` | Not required for agent sync. Use local SMB to file server |
| Agent cannot reach Azure | HTTPS 443 blocked, proxy, TLS inspection, or firewall | `Test-NetConnection management.azure.com -Port 443` | Allow required HTTPS/proxy path |
| DFS-R conflict | DFS-R and Azure File Sync both trying to replicate same data | DFS-R memberships and server endpoints | Avoid overlapping replication or migrate carefully |
| Tiered files recall slowly | Cloud tiering recall, WAN bandwidth, or large file | User access pattern and network | Adjust tiering policy or pre-stage hot data |
| Free space not maintained | Tiering policy too conservative or high churn | Volume free space and tiering policy | Lower free-space target carefully or add storage |
| Previous Versions behavior confusing | VSS and cloud tiering compatibility not enabled | `Get-StorageSyncSelfServiceRestore` if local cmdlet exists | Enable self-service restore compatibility if needed |
| Deduplication interaction unexpected | Dedup and tiering order/policy effect | Dedup status and tiering settings | Confirm supported configuration and schedule |
| Event logs missing | Different log names or no activity yet | `Get-WinEvent -ListLog "*StorageSync*","*FileSync*"` | Query discovered logs after generating sync activity |

# Configure_Azure_File_Sync_For_Windows_File_Server_Related_Labs
| Lab | Relationship |
|---|---|
| `01_Install_File_Server_Role_And_Management_Tools.md` | Establishes the Windows file server baseline |
| `02_Create_And_Publish_SMB_Shares.md` | Users continue accessing synced data through SMB shares |
| `03_Configure_NTFS_And_Share_Permissions.md` | Azure File Sync preserves Windows-style ACLs across endpoints |
| `07_Configure_User_Home_Folders_And_Department_Shares.md` | Common data candidates for Azure File Sync |
| `09_Configure_Storage_Quotas_With_FSRM.md` | Local quotas need review before syncing user data |
| `10_Configure_FSRM_Storage_Reports_And_File_Management_Tasks.md` | Reports help identify data size and file types before sync |
| `11_Configure_Shadow_Copies_And_Previous_Versions.md` | Previous Versions can coexist with Azure File Sync when designed correctly |
| `12_Backup_Restore_And_Export_File_Server_Config.md` | Backup plan must exist before attaching sync topology |
| `13_Configure_DFS_Namespaces.md` | DFS-N can provide namespace paths over Azure File Sync server endpoints |
| `14_Configure_DFS_Replication.md` | DFS-R overlap needs careful migration or avoidance |
| `15_Monitor_SMB_Sessions_Open_Files_Events_And_Performance.md` | SMB monitoring validates user impact during hybrid sync deployment |
| `16_Troubleshoot_File_Share_Access_And_SMB_Failures.md` | Separates local SMB failures from Azure File Sync failures |
| `17_Configure_Data_Deduplication_For_File_Shares.md` | Deduplication and cloud tiering can interact on the same volume |
| `21_Configure_Work_Folders.md` | Contrasts file-server sync to user-device sync |