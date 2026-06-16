07_Troubleshoot_Azure_Storage_Access_Networking_Performance_And_Data_Recovery.md

# Troubleshoot_Azure_Storage_Access_Networking_Performance_And_Data_Recovery

# Troubleshoot_Azure_Storage_Access_Networking_Performance_And_Data_Recovery_Index
07_Troubleshoot_Azure_Storage_Access_Networking_Performance_And_Data_Recovery.md
Troubleshoot_Azure_Storage_Access_Networking_Performance_And_Data_Recovery
Troubleshoot_Azure_Storage_Access_Networking_Performance_And_Data_Recovery_Source_Basis
Troubleshoot_Azure_Storage_Access_Networking_Performance_And_Data_Recovery_Mental_Model
Troubleshoot_Azure_Storage_Access_Networking_Performance_And_Data_Recovery_Triage_Map
Troubleshoot_Azure_Storage_Access_Networking_Performance_And_Data_Recovery_Planning_Table
Troubleshoot_Azure_Storage_Access_Networking_Performance_And_Data_Recovery_Configuration_Checklist
Troubleshoot_Azure_Storage_Access_Networking_Performance_And_Data_Recovery_Initial_Triage_Skeleton
Troubleshoot_Azure_Storage_Access_Networking_Performance_And_Data_Recovery_Access_Auth_RBAC_SAS_Key_Skeleton
Troubleshoot_Azure_Storage_Access_Networking_Performance_And_Data_Recovery_Network_Firewall_Private_Endpoint_DNS_Skeleton
Troubleshoot_Azure_Storage_Access_Networking_Performance_And_Data_Recovery_Blob_Performance_And_Throttling_Skeleton
Troubleshoot_Azure_Storage_Access_Networking_Performance_And_Data_Recovery_Azure_Files_SMB_Skeleton
Troubleshoot_Azure_Storage_Access_Networking_Performance_And_Data_Recovery_AzCopy_Storage_Explorer_Skeleton
Troubleshoot_Azure_Storage_Access_Networking_Performance_And_Data_Recovery_Blob_Data_Recovery_Skeleton
Troubleshoot_Azure_Storage_Access_Networking_Performance_And_Data_Recovery_Azure_Files_Data_Recovery_Skeleton
Troubleshoot_Azure_Storage_Access_Networking_Performance_And_Data_Recovery_KQL_Investigation_Skeleton
Troubleshoot_Azure_Storage_Access_Networking_Performance_And_Data_Recovery_Evidence_Export_Skeleton
Troubleshoot_Azure_Storage_Access_Networking_Performance_And_Data_Recovery_Verification_Commands
Troubleshoot_Azure_Storage_Access_Networking_Performance_And_Data_Recovery_Rollback
Troubleshoot_Azure_Storage_Access_Networking_Performance_And_Data_Recovery_Failure_Checks
Troubleshoot_Azure_Storage_Access_Networking_Performance_And_Data_Recovery_Related_Labs

# Troubleshoot_Azure_Storage_Access_Networking_Performance_And_Data_Recovery_Source_Basis

| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Configure Azure Storage firewalls and virtual networks | Public endpoint, default action, IP rules, VNet rules, resource instance rules, trusted service exceptions |
| Microsoft Learn | Use private endpoints for Azure Storage | Private Link, private endpoint sub-resources, private DNS, VNet access |
| Microsoft Learn | Manage storage account access keys | Key listing, regeneration, connection string troubleshooting |
| Microsoft Learn | Shared access signatures overview | User delegation SAS, service SAS, account SAS, permissions, expiry, signed resources |
| Microsoft Learn | Blob versioning | Previous version recovery and delete/overwrite protection |
| Microsoft Learn | Blob soft delete | Recovering deleted blobs, snapshots, and versions during retention |
| Microsoft Learn | Azure Files data protection overview | Soft delete, share snapshots, Azure Backup, Azure File Sync |
| Microsoft Learn | Mount SMB Azure file share on Windows | TCP 445, SMB mount path, identity-based access, key-based mount behavior |
| Microsoft Learn | Monitor Azure Blob Storage | Blob metrics, logs, Metrics Explorer, diagnostic settings, KQL investigation |
| Microsoft Learn | Monitor Azure Files | File metrics, SMB logs, REST logs, authenticated request logging, KQL investigation |
| Azure operational practice | Storage troubleshooting workflow | Incident triage, evidence capture, rollback, recovery, and root cause documentation |

# Troubleshoot_Azure_Storage_Access_Networking_Performance_And_Data_Recovery_Mental_Model

| Layer | What Breaks | What To Check |
|---|---|---|
| Subscription context | Commands target wrong tenant or subscription | `az account show`, `Get-AzContext` |
| Resource existence | Wrong account, container, share, or resource group | `az storage account show`, `az storage container list`, `az storage share-rm list` |
| Name resolution | Endpoint resolves public IP instead of private endpoint IP | `Resolve-DnsName`, `nslookup` |
| Network path | TCP 443 or TCP 445 blocked | `Test-NetConnection`, firewall rules, NSG, route table |
| Storage firewall | Public endpoint denies traffic | `networkRuleSet`, IP rules, VNet rules, trusted services |
| Private endpoint | Wrong sub-resource, pending connection, missing DNS zone group | `az network private-endpoint show` |
| Authorization | RBAC, SAS, key, or Shared Key mismatch | Role assignments, SAS fields, key state, `allowSharedKeyAccess` |
| Blob data plane | Container missing, blob missing, deleted, archived, tiered, or versioned | Blob list with deleted and versions included |
| Azure Files SMB | Port 445, identity source, share RBAC, NTFS ACL, cached credentials | `Test-NetConnection`, `net use`, `cmdkey`, `icacls` |
| Performance | Latency, throttling, transaction bursts, hot partitions, client distance | Metrics, logs, AzCopy job logs, KQL |
| Recovery | No soft delete, versioning, snapshots, or backup | Blob service properties, file service properties, snapshots, backup vault |
| Monitoring | No logs because diagnostic settings missing | Diagnostic settings, workspace tables, activity generation |
| First rule | Prove identity, network, and resource scope separately before blaming the service |
| Blunt rule | Most Azure Storage failures are not storage failures; they are wrong context, DNS, firewall, auth, or client path failures |

# Troubleshoot_Azure_Storage_Access_Networking_Performance_And_Data_Recovery_Triage_Map

| Symptom | Primary Path | First Checks |
|---|---|---|
| `AuthorizationPermissionMismatch` | RBAC or SAS | Role assignment scope, SAS permissions, auth mode |
| `AuthenticationFailed` | SAS, key, token, clock, or signature | SAS expiry, key validity, system clock, token tenant |
| `PublicAccessNotPermitted` | Anonymous access blocked | Account public blob access and container ACL |
| `This request is not authorized to perform this operation` | Data-plane role missing | Storage Blob Data Contributor, Storage File Data SMB Share Contributor |
| `NetworkAclsValidationFailure` | Firewall or network rule issue | Default action, allowed IP, subnet service endpoint |
| Timeout to Blob endpoint | DNS, firewall, private endpoint, route | `Resolve-DnsName`, `Test-NetConnection -Port 443` |
| SMB mount fails | TCP 445, identity, cached credentials | `Test-NetConnection -Port 445`, `cmdkey`, `net use` |
| AzCopy fails | Auth, path, SAS, network, or job issue | AzCopy log, job status, endpoint reachability |
| Storage Explorer fails | Sign-in, subscription filter, auth, network | Reauthenticate, check tenant, check firewall path |
| Blob deleted | Soft delete or versioning recovery | List deleted blobs and versions |
| Container deleted | Container soft delete | List deleted containers |
| File deleted | Share snapshot, soft delete, or Azure Backup | Snapshot list, share soft delete, backup vault |
| Slow transfer | Throughput, latency, client location, throttling | Metrics, logs, AzCopy concurrency, private endpoint path |
| Replication not working | Object replication prerequisites or filters | Versioning, change feed, policy on both accounts, prefix |
| Logs missing | Diagnostic setting not configured or no activity | Diagnostic settings and KQL tables |

# Troubleshoot_Azure_Storage_Access_Networking_Performance_And_Data_Recovery_Planning_Table

| Item | Example | Decision |
|---|---|---|
| Incident ID | `INC-STG-0001` | `<incident-id>` |
| Subscription | `Visual Studio Enterprise Subscription` | `<subscription-name-or-id>` |
| Resource group | `rg-storage-core-lab-01` | `<resource-group>` |
| Storage account | `stcorelab001` | `<storage-account-name>` |
| Service affected | `Blob` | `<Blob/File/Queue/Table>` |
| Container | `labdata` | `<container-name>` |
| File share | `deptshare` | `<share-name>` |
| Endpoint tested | `blob.core.windows.net` | `<blob/file/queue/table>` |
| Access method | `Entra ID` | `<entra/sas/key/anonymous/managed-identity>` |
| Network path | `Private endpoint` | `<public/private/service-endpoint>` |
| Client device | `vm-storage-test-01` | `<client-host>` |
| Client network | `vnet-storage-lab-01` | `<network>` |
| Expected private IP | `10.60.10.4` | `<private-endpoint-ip>` |
| Required port | `443` or `445` | `<port>` |
| Diagnostic workspace | `law-storage-core-lab-01` | `<workspace-name>` |
| Recovery feature expected | `Blob soft delete and versioning` | `<recovery-feature>` |
| Recovery point needed | `2026-06-15 14:30 UTC` | `<timestamp>` |
| Evidence path | `C:\Azure-Storage-Evidence\07-storage-troubleshooting` | `<evidence-path>` |
| Rollback stance | `Restore data, revert access exception, preserve evidence` | `<rollback-plan>` |

# Troubleshoot_Azure_Storage_Access_Networking_Performance_And_Data_Recovery_Configuration_Checklist

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm Azure context | Admin workstation / Cloud Shell | `az account show` or `Get-AzContext` | Correct tenant and subscription are active |
| 2 | Confirm storage account exists | Admin workstation / Cloud Shell | `az storage account show` | Account exists and properties return |
| 3 | Capture storage account baseline | Admin workstation / Cloud Shell | Query kind, SKU, endpoints, network, public access, Shared Key, TLS | Account posture is known |
| 4 | Identify affected service | Operator | Blob, File, Queue, or Table | Correct child service is targeted |
| 5 | Confirm resource exists | Admin workstation / Cloud Shell | Container, blob, file share, queue, or table list command | Target resource exists or deletion is confirmed |
| 6 | Confirm DNS resolution | Client device | `Resolve-DnsName <account>.<service>.core.windows.net` | Endpoint resolves to expected public or private IP |
| 7 | Confirm TCP reachability | Client device | `Test-NetConnection <endpoint> -Port 443` or `-Port 445` | Required port succeeds |
| 8 | Confirm firewall posture | Admin workstation / Cloud Shell | `az storage account show --query networkRuleSet` | Default action and rules are known |
| 9 | Confirm private endpoint posture | Admin workstation / Cloud Shell | `az network private-endpoint list` | Correct sub-resource private endpoint exists |
| 10 | Confirm private DNS posture | Admin workstation / Cloud Shell | Private DNS zone and VNet link list | Correct DNS zone is linked |
| 11 | Confirm RBAC assignments | Admin workstation / Cloud Shell | `az role assignment list --scope "<scope>"` | Required data-plane role exists |
| 12 | Confirm SAS if used | Operator | Decode SAS fields and expiry | SAS has correct service, resource, permissions, and time |
| 13 | Confirm Shared Key if used | Admin workstation / Cloud Shell | `allowSharedKeyAccess` and key list | Key auth is allowed only if intended |
| 14 | Confirm Blob service protection | Admin workstation / Cloud Shell | Blob service properties | Versioning, soft delete, change feed state known |
| 15 | Confirm File service protection | Admin workstation / Cloud Shell | File service properties and share snapshots | Soft delete and snapshots known |
| 16 | Pull metrics | Admin workstation / Cloud Shell | `az monitor metrics list` | Availability, latency, transactions, capacity known |
| 17 | Query logs | Log Analytics | KQL for Blob, File, AzureActivity, AzureMetrics | Requests and errors are visible |
| 18 | Test client operation | Client device | Upload, list, download, mount, read, write | Failure is reproduced or cleared |
| 19 | Recover data if needed | Admin workstation / Cloud Shell | Undelete, restore version, restore snapshot, restore backup | Data restored or recovery gap documented |
| 20 | Export evidence | Admin workstation | Run evidence export skeleton | Evidence bundle saved |
| 21 | Document root cause | Operator | Record cause, fix, validation, prevention | Incident notes are complete |

# Troubleshoot_Azure_Storage_Access_Networking_Performance_And_Data_Recovery_Initial_Triage_Skeleton

```powershell
# Run from admin workstation.
# Purpose: collect first-pass incident facts.

$SubscriptionId = "<subscription-id>"
$ResourceGroupName = "rg-storage-core-lab-01"
$StorageAccountName = "stcorelab001"
$EvidencePath = "C:\Azure-Storage-Evidence\07-storage-troubleshooting"

New-Item `
  -ItemType Directory `
  -Force `
  -Path $EvidencePath

az account set `
  --subscription $SubscriptionId

az account show `
  --output json |
  Out-File "$EvidencePath\az-account-context.json"

az storage account show `
  --resource-group $ResourceGroupName `
  --name $StorageAccountName `
  --output json |
  Out-File "$EvidencePath\storage-account-full.json"

az storage account show `
  --resource-group $ResourceGroupName `
  --name $StorageAccountName `
  --query "{
    name:name,
    id:id,
    location:location,
    kind:kind,
    sku:sku.name,
    hns:isHnsEnabled,
    httpsOnly:enableHttpsTrafficOnly,
    minTls:minimumTlsVersion,
    publicNetworkAccess:publicNetworkAccess,
    defaultAction:networkRuleSet.defaultAction,
    bypass:networkRuleSet.bypass,
    ipRules:networkRuleSet.ipRules,
    vnetRules:networkRuleSet.virtualNetworkRules,
    allowBlobPublicAccess:allowBlobPublicAccess,
    allowSharedKeyAccess:allowSharedKeyAccess,
    endpoints:primaryEndpoints
  }" `
  --output json |
  Out-File "$EvidencePath\storage-account-triage-summary.json"

$StorageId = az storage account show `
  --resource-group $ResourceGroupName `
  --name $StorageAccountName `
  --query id `
  -o tsv

az role assignment list `
  --scope $StorageId `
  --output json |
  Out-File "$EvidencePath\storage-account-rbac.json"

az storage account network-rule list `
  --resource-group $ResourceGroupName `
  --account-name $StorageAccountName `
  --output json |
  Out-File "$EvidencePath\storage-network-rules.json"
```

# Troubleshoot_Azure_Storage_Access_Networking_Performance_And_Data_Recovery_Access_Auth_RBAC_SAS_Key_Skeleton

```bash
# Purpose: isolate auth failures across Entra ID, RBAC, SAS, keys, and public access.

SUBSCRIPTION_ID="<subscription-id>"
RESOURCE_GROUP="rg-storage-core-lab-01"
STORAGE_ACCOUNT="stcorelab001"
CONTAINER_NAME="labdata"
SHARE_NAME="deptshare"
PRINCIPAL_OBJECT_ID="<principal-object-id>"

az account set \
  --subscription "$SUBSCRIPTION_ID"

STORAGE_ID=$(az storage account show \
  --resource-group "$RESOURCE_GROUP" \
  --name "$STORAGE_ACCOUNT" \
  --query id \
  -o tsv)

BLOB_SCOPE="${STORAGE_ID}/blobServices/default/containers/${CONTAINER_NAME}"
FILE_SHARE_SCOPE="${STORAGE_ID}/fileServices/default/shares/${SHARE_NAME}"

# Check account-level access posture.
az storage account show \
  --resource-group "$RESOURCE_GROUP" \
  --name "$STORAGE_ACCOUNT" \
  --query "{
    allowSharedKeyAccess:allowSharedKeyAccess,
    allowBlobPublicAccess:allowBlobPublicAccess,
    publicNetworkAccess:publicNetworkAccess,
    defaultAction:networkRuleSet.defaultAction
  }" \
  -o json

# Check RBAC at account, container, and file share scopes.
az role assignment list \
  --scope "$STORAGE_ID" \
  -o table

az role assignment list \
  --scope "$BLOB_SCOPE" \
  -o table

az role assignment list \
  --scope "$FILE_SHARE_SCOPE" \
  -o table

# Test Blob access with Entra ID.
az storage container list \
  --account-name "$STORAGE_ACCOUNT" \
  --auth-mode login \
  -o table

az storage blob list \
  --account-name "$STORAGE_ACCOUNT" \
  --container-name "$CONTAINER_NAME" \
  --auth-mode login \
  -o table

# Check keys only if Shared Key is allowed by policy.
az storage account keys list \
  --resource-group "$RESOURCE_GROUP" \
  --account-name "$STORAGE_ACCOUNT" \
  --query "[].{keyName:keyName,permissions:permissions,creationTime:creationTime}" \
  -o table

# Common SAS triage:
# Verify these fields in the SAS token:
# sv = service version
# ss = services for account SAS
# srt = resource types for account SAS
# sp = permissions
# st = start time
# se = expiry time
# spr = protocol
# sig = signature
# sr = signed resource for service SAS
# sip = signed IP range if present

# Controlled user delegation SAS test for Blob.
EXPIRY_UTC="$(date -u -d '+2 hours' '+%Y-%m-%dT%H:%MZ' 2>/dev/null || echo '2026-06-15T23:59Z')"

az storage container generate-sas \
  --account-name "$STORAGE_ACCOUNT" \
  --name "$CONTAINER_NAME" \
  --permissions rl \
  --expiry "$EXPIRY_UTC" \
  --https-only \
  --auth-mode login \
  --as-user \
  -o tsv
```

# Troubleshoot_Azure_Storage_Access_Networking_Performance_And_Data_Recovery_Network_Firewall_Private_Endpoint_DNS_Skeleton

```powershell
# Run from the affected client and from a known-good client.
# Purpose: isolate DNS, TCP, private endpoint, and firewall issues.

$StorageAccountName = "stcorelab001"
$EvidencePath = "C:\Azure-Storage-Evidence\07-storage-troubleshooting"

New-Item `
  -ItemType Directory `
  -Force `
  -Path $EvidencePath

$Endpoints = @(
    "$StorageAccountName.blob.core.windows.net",
    "$StorageAccountName.file.core.windows.net",
    "$StorageAccountName.queue.core.windows.net",
    "$StorageAccountName.table.core.windows.net"
)

foreach ($Endpoint in $Endpoints) {
    Resolve-DnsName $Endpoint |
      Out-File "$EvidencePath\resolve-$($Endpoint.Replace('.','-')).txt"

    nslookup $Endpoint |
      Out-File "$EvidencePath\nslookup-$($Endpoint.Replace('.','-')).txt"

    Test-NetConnection `
      -ComputerName $Endpoint `
      -Port 443 |
      Out-File "$EvidencePath\tcp443-$($Endpoint.Replace('.','-')).txt"
}

# SMB requires TCP 445.
Test-NetConnection `
  -ComputerName "$StorageAccountName.file.core.windows.net" `
  -Port 445 |
  Out-File "$EvidencePath\tcp445-file-endpoint.txt"

# Interpret results:
# Private endpoint expected:
#   Blob/File endpoint should resolve to private endpoint IP.
# Public endpoint expected:
#   Endpoint resolves public IP and storage firewall must allow client source.
# Failure:
#   DNS wrong, VNet link missing, private DNS zone missing, firewall denied, NSG route issue, or port blocked.
```

```bash
# Admin-side private endpoint and DNS evidence.

RESOURCE_GROUP="rg-storage-core-lab-01"
STORAGE_ACCOUNT="stcorelab001"

STORAGE_ID=$(az storage account show \
  --resource-group "$RESOURCE_GROUP" \
  --name "$STORAGE_ACCOUNT" \
  --query id \
  -o tsv)

az storage account show \
  --resource-group "$RESOURCE_GROUP" \
  --name "$STORAGE_ACCOUNT" \
  --query "{
    publicNetworkAccess:publicNetworkAccess,
    defaultAction:networkRuleSet.defaultAction,
    bypass:networkRuleSet.bypass,
    ipRules:networkRuleSet.ipRules,
    virtualNetworkRules:networkRuleSet.virtualNetworkRules,
    privateEndpointConnections:privateEndpointConnections
  }" \
  -o json

az network private-endpoint list \
  --resource-group "$RESOURCE_GROUP" \
  -o table

az network private-dns zone list \
  --resource-group "$RESOURCE_GROUP" \
  -o table

az network private-dns link vnet list \
  --resource-group "$RESOURCE_GROUP" \
  --zone-name "privatelink.blob.core.windows.net" \
  -o table

az network private-dns link vnet list \
  --resource-group "$RESOURCE_GROUP" \
  --zone-name "privatelink.file.core.windows.net" \
  -o table
```

# Troubleshoot_Azure_Storage_Access_Networking_Performance_And_Data_Recovery_Blob_Performance_And_Throttling_Skeleton

```bash
# Purpose: inspect Blob performance, capacity, latency, availability, and transaction behavior.

RESOURCE_GROUP="rg-storage-core-lab-01"
STORAGE_ACCOUNT="stcorelab001"

STORAGE_ID=$(az storage account show \
  --resource-group "$RESOURCE_GROUP" \
  --name "$STORAGE_ACCOUNT" \
  --query id \
  -o tsv)

BLOB_SERVICE_ID="${STORAGE_ID}/blobServices/default"

# List metrics.
az monitor metrics list-definitions \
  --resource "$BLOB_SERVICE_ID" \
  -o table

# Availability.
az monitor metrics list \
  --resource "$BLOB_SERVICE_ID" \
  --metric "Availability" \
  --interval PT1H \
  --aggregation Average \
  -o table

# End-to-end latency.
az monitor metrics list \
  --resource "$BLOB_SERVICE_ID" \
  --metric "SuccessE2ELatency" \
  --interval PT1H \
  --aggregation Average \
  -o table

# Server latency.
az monitor metrics list \
  --resource "$BLOB_SERVICE_ID" \
  --metric "SuccessServerLatency" \
  --interval PT1H \
  --aggregation Average \
  -o table

# Transactions.
az monitor metrics list \
  --resource "$BLOB_SERVICE_ID" \
  --metric "Transactions" \
  --interval PT1H \
  --aggregation Total \
  -o table

# Ingress and egress.
az monitor metrics list \
  --resource "$BLOB_SERVICE_ID" \
  --metric "Ingress" \
  --interval PT1H \
  --aggregation Total \
  -o table

az monitor metrics list \
  --resource "$BLOB_SERVICE_ID" \
  --metric "Egress" \
  --interval PT1H \
  --aggregation Total \
  -o table

# Capacity at account scope.
az monitor metrics list \
  --resource "$STORAGE_ID" \
  --metric "UsedCapacity" \
  --interval PT1H \
  --aggregation Average \
  -o table
```

# Troubleshoot_Azure_Storage_Access_Networking_Performance_And_Data_Recovery_Azure_Files_SMB_Skeleton

```powershell
# Run from Windows client.
# Purpose: troubleshoot Azure Files SMB mount, identity, cached credentials, ACLs, and port 445.

$StorageAccountName = "stcorelab001"
$ShareName = "deptshare"
$DriveLetter = "Z"
$EvidencePath = "C:\Azure-Storage-Evidence\07-storage-troubleshooting"

New-Item `
  -ItemType Directory `
  -Force `
  -Path $EvidencePath

# DNS and SMB reachability.
Resolve-DnsName "$StorageAccountName.file.core.windows.net" |
  Tee-Object "$EvidencePath\files-resolve-dns.txt"

Test-NetConnection `
  -ComputerName "$StorageAccountName.file.core.windows.net" `
  -Port 445 |
  Tee-Object "$EvidencePath\files-tcp445.txt"

# Existing mappings.
net use |
  Out-File "$EvidencePath\net-use-before.txt"

# Cached credentials.
cmdkey /list |
  Out-File "$EvidencePath\cmdkey-before.txt"

# Remove old mapping and cached key credential if key-based auth is interfering with identity auth.
net use "$DriveLetter`:" /delete /y 2>$null
cmd.exe /C "cmdkey /delete:`"$StorageAccountName.file.core.windows.net`""

# Try identity mount.
New-PSDrive `
  -Name $DriveLetter `
  -PSProvider FileSystem `
  -Root "\\$StorageAccountName.file.core.windows.net\$ShareName" `
  -Persist `
  -Scope Global

# Validate read/write and ACL.
Get-PSDrive `
  -Name $DriveLetter |
  Tee-Object "$EvidencePath\psdrive-after.txt"

icacls "$DriveLetter`:\" |
  Tee-Object "$EvidencePath\share-root-icacls.txt"

"SMB write test $(Get-Date)" |
  Out-File "$DriveLetter`:\smb-write-test.txt"

Get-Content "$DriveLetter`:\smb-write-test.txt" |
  Tee-Object "$EvidencePath\smb-write-test-result.txt"

# If this fails:
# Check port 445, private DNS, identity source, share-level RBAC, NTFS ACL, and cached credentials.
```

# Troubleshoot_Azure_Storage_Access_Networking_Performance_And_Data_Recovery_AzCopy_Storage_Explorer_Skeleton

```powershell
# Run from transfer workstation.
# Purpose: troubleshoot AzCopy and Storage Explorer access path.

$StorageAccountName = "stcorelab001"
$ContainerName = "labdata"
$LocalPath = "C:\StorageTransferTroubleshoot"
$EvidencePath = "C:\Azure-Storage-Evidence\07-storage-troubleshooting"

New-Item `
  -ItemType Directory `
  -Force `
  -Path $LocalPath, $EvidencePath

azcopy --version |
  Tee-Object "$EvidencePath\azcopy-version.txt"

azcopy env |
  Tee-Object "$EvidencePath\azcopy-env.txt"

# Confirm endpoint reachability.
Resolve-DnsName "$StorageAccountName.blob.core.windows.net" |
  Tee-Object "$EvidencePath\azcopy-blob-dns.txt"

Test-NetConnection `
  -ComputerName "$StorageAccountName.blob.core.windows.net" `
  -Port 443 |
  Tee-Object "$EvidencePath\azcopy-blob-tcp443.txt"

# Login if using Entra ID.
azcopy login `
  --tenant-id "<tenant-id>"

# Test upload.
"AzCopy troubleshoot $(Get-Date)" |
  Out-File "$LocalPath\azcopy-troubleshoot.txt"

azcopy copy `
  "$LocalPath\azcopy-troubleshoot.txt" `
  "https://$StorageAccountName.blob.core.windows.net/$ContainerName/troubleshoot/azcopy-troubleshoot.txt"

# List jobs and logs.
azcopy jobs list |
  Tee-Object "$EvidencePath\azcopy-jobs-list.txt"

# Storage Explorer operator checks:
# 1. Confirm correct tenant and subscription filter.
# 2. Confirm access mode: Entra ID, SAS, account key, or connection string.
# 3. Confirm the same endpoint works from the machine running Storage Explorer.
# 4. Confirm firewall allows that network path.
# 5. Confirm operation matches the role or SAS permissions.
```

# Troubleshoot_Azure_Storage_Access_Networking_Performance_And_Data_Recovery_Blob_Data_Recovery_Skeleton

```bash
# Purpose: recover deleted or overwritten Blob data using soft delete and versioning.

RESOURCE_GROUP="rg-storage-core-lab-01"
STORAGE_ACCOUNT="stcorelab001"
CONTAINER_NAME="labdata"
BLOB_PREFIX="recovery/"
BLOB_NAME="recovery/test-blob.txt"

# Check Blob service protection.
az storage account blob-service-properties show \
  --resource-group "$RESOURCE_GROUP" \
  --account-name "$STORAGE_ACCOUNT" \
  --query "{
    versioning:isVersioningEnabled,
    deleteRetentionPolicy:deleteRetentionPolicy,
    containerDeleteRetentionPolicy:containerDeleteRetentionPolicy,
    changeFeed:changeFeed,
    restorePolicy:restorePolicy
  }" \
  -o json

# List active, deleted, and versioned blobs.
az storage blob list \
  --account-name "$STORAGE_ACCOUNT" \
  --container-name "$CONTAINER_NAME" \
  --prefix "$BLOB_PREFIX" \
  --include d v s m t \
  --auth-mode login \
  --query "[].{
    name:name,
    deleted:deleted,
    versionId:versionId,
    isCurrentVersion:isCurrentVersion,
    snapshot:snapshot,
    tier:properties.blobTier,
    lastModified:properties.lastModified,
    remainingRetentionDays:properties.remainingRetentionDays
  }" \
  -o table

# Undelete a soft-deleted blob.
az storage blob undelete \
  --account-name "$STORAGE_ACCOUNT" \
  --container-name "$CONTAINER_NAME" \
  --name "$BLOB_NAME" \
  --auth-mode login

# Download a specific previous version for validation.
VERSION_ID="<version-id>"

az storage blob download \
  --account-name "$STORAGE_ACCOUNT" \
  --container-name "$CONTAINER_NAME" \
  --name "$BLOB_NAME" \
  --version-id "$VERSION_ID" \
  --file "./restored-version-test-blob.txt" \
  --auth-mode login

# Promote previous version by uploading it back over current blob.
az storage blob upload \
  --account-name "$STORAGE_ACCOUNT" \
  --container-name "$CONTAINER_NAME" \
  --name "$BLOB_NAME" \
  --file "./restored-version-test-blob.txt" \
  --auth-mode login \
  --overwrite true

# Container soft delete recovery.
az storage container list \
  --account-name "$STORAGE_ACCOUNT" \
  --include-deleted \
  --auth-mode login \
  -o table

# Restore deleted container if deleted-version is available.
DELETED_CONTAINER="<deleted-container-name>"
DELETED_VERSION="<deleted-version>"

az storage container restore \
  --account-name "$STORAGE_ACCOUNT" \
  --name "$DELETED_CONTAINER" \
  --deleted-version "$DELETED_VERSION" \
  --auth-mode login
```

# Troubleshoot_Azure_Storage_Access_Networking_Performance_And_Data_Recovery_Azure_Files_Data_Recovery_Skeleton

```bash
# Purpose: recover Azure Files data through share snapshots and share soft delete.

RESOURCE_GROUP="rg-storage-core-lab-01"
STORAGE_ACCOUNT="stcorelab001"
SHARE_NAME="deptshare"
REMOTE_PATH="snapshots"
REMOTE_FILE="snapshot-test.txt"

# Check file service protection.
az storage account file-service-properties show \
  --resource-group "$RESOURCE_GROUP" \
  --account-name "$STORAGE_ACCOUNT" \
  -o json

# List file shares through ARM.
az storage share-rm list \
  --resource-group "$RESOURCE_GROUP" \
  --storage-account "$STORAGE_ACCOUNT" \
  --include-deleted \
  -o table

# Snapshot commands often use data-plane key auth.
# Use only if Shared Key is permitted by policy.
ACCOUNT_KEY=$(az storage account keys list \
  --resource-group "$RESOURCE_GROUP" \
  --account-name "$STORAGE_ACCOUNT" \
  --query "[0].value" \
  -o tsv)

# List share snapshots.
az storage share list \
  --account-name "$STORAGE_ACCOUNT" \
  --account-key "$ACCOUNT_KEY" \
  --include-snapshots \
  --query "[].{name:name,snapshot:snapshot,quota:properties.quota,deleted:deleted}" \
  -o table

# Download file from a known snapshot.
SNAPSHOT_TIME="<snapshot-time>"

az storage file download \
  --account-name "$STORAGE_ACCOUNT" \
  --account-key "$ACCOUNT_KEY" \
  --share-name "$SHARE_NAME" \
  --path "$REMOTE_PATH" \
  --name "$REMOTE_FILE" \
  --snapshot "$SNAPSHOT_TIME" \
  --dest "./restored-$REMOTE_FILE"

# Upload restored copy to active share.
az storage file upload \
  --account-name "$STORAGE_ACCOUNT" \
  --account-key "$ACCOUNT_KEY" \
  --share-name "$SHARE_NAME" \
  --source "./restored-$REMOTE_FILE" \
  --path "$REMOTE_PATH" \
  --name "restored-$REMOTE_FILE"

# If Azure Backup protects the share:
# Check Recovery Services vault backup items and restore from selected recovery point.
# Document vault name, policy, recovery point, and restored path.
```

# Troubleshoot_Azure_Storage_Access_Networking_Performance_And_Data_Recovery_KQL_Investigation_Skeleton

```kusto
// Blob failures by status over the last 24 hours.
StorageBlobLogs
| where TimeGenerated >= ago(24h)
| where StatusText !contains "Success"
| summarize Count = count() by StatusCode, StatusText, OperationName
| order by Count desc
```

```kusto
// Blob auth failures.
StorageBlobLogs
| where TimeGenerated >= ago(7d)
| where StatusText has_any ("Authorization", "Authentication", "Permission", "Forbidden")
| project TimeGenerated, AccountName, OperationName, StatusCode, StatusText, AuthenticationType, CallerIpAddress, Uri
| order by TimeGenerated desc
```

```kusto
// Blob delete and overwrite investigation.
StorageBlobLogs
| where TimeGenerated >= ago(7d)
| where OperationName has_any ("Delete", "PutBlob", "PutBlockList", "CopyBlob")
| project TimeGenerated, AccountName, OperationName, StatusCode, StatusText, AuthenticationType, CallerIpAddress, Uri
| order by TimeGenerated desc
```

```kusto
// High-latency Blob requests.
StorageBlobLogs
| where TimeGenerated >= ago(24h)
| where DurationMs > 1000
| project TimeGenerated, AccountName, OperationName, DurationMs, ServerLatencyMs, StatusText, CallerIpAddress, Uri
| order by DurationMs desc
```

```kusto
// Azure Files SMB failures.
StorageFileLogs
| where TimeGenerated >= ago(7d)
| where Protocol == "SMB"
| where StatusCode contains "-" or StatusText !contains "Success"
| project TimeGenerated, AccountName, ShareName, OperationName, StatusCode, StatusText, AuthenticationType, CallerIpAddress, Uri
| order by TimeGenerated desc
```

```kusto
// Azure Files REST failures.
StorageFileLogs
| where TimeGenerated >= ago(7d)
| where Protocol == "HTTPS"
| where StatusText !contains "Success"
| project TimeGenerated, AccountName, ShareName, OperationName, StatusCode, StatusText, AuthenticationType, CallerIpAddress, Uri
| order by TimeGenerated desc
```

```kusto
// Storage account management-plane changes.
AzureActivity
| where TimeGenerated >= ago(14d)
| where ResourceProviderValue =~ "Microsoft.Storage"
| project TimeGenerated, Caller, OperationNameValue, ActivityStatusValue, ResourceGroup, Resource, Properties
| order by TimeGenerated desc
```

```kusto
// Metrics routed to Log Analytics.
AzureMetrics
| where TimeGenerated >= ago(24h)
| where ResourceProvider == "MICROSOFT.STORAGE"
| summarize AvgValue = avg(Average), TotalValue = sum(Total) by Resource, MetricName, bin(TimeGenerated, 1h)
| order by TimeGenerated desc
```

# Troubleshoot_Azure_Storage_Access_Networking_Performance_And_Data_Recovery_Evidence_Export_Skeleton

```bash
# Purpose: export a complete evidence bundle for support, RCA, and workbook validation.

SUBSCRIPTION_ID="<subscription-id>"
RESOURCE_GROUP="rg-storage-core-lab-01"
MONITOR_RG="rg-monitoring-core-lab-01"
STORAGE_ACCOUNT="stcorelab001"
CONTAINER_NAME="labdata"
SHARE_NAME="deptshare"
WORKSPACE_NAME="law-storage-core-lab-01"
EVIDENCE_DIR="./azure-storage-evidence/07-storage-troubleshooting"

mkdir -p "$EVIDENCE_DIR"

az account set \
  --subscription "$SUBSCRIPTION_ID"

STORAGE_ID=$(az storage account show \
  --resource-group "$RESOURCE_GROUP" \
  --name "$STORAGE_ACCOUNT" \
  --query id \
  -o tsv)

BLOB_SERVICE_ID="${STORAGE_ID}/blobServices/default"
FILE_SERVICE_ID="${STORAGE_ID}/fileServices/default"
SHARE_SCOPE="${STORAGE_ID}/fileServices/default/shares/${SHARE_NAME}"

# Account and posture.
az account show -o json > "$EVIDENCE_DIR/account-context.json"

az storage account show \
  --resource-group "$RESOURCE_GROUP" \
  --name "$STORAGE_ACCOUNT" \
  -o json > "$EVIDENCE_DIR/storage-account-full.json"

az storage account network-rule list \
  --resource-group "$RESOURCE_GROUP" \
  --account-name "$STORAGE_ACCOUNT" \
  -o json > "$EVIDENCE_DIR/storage-network-rules.json"

az storage account blob-service-properties show \
  --resource-group "$RESOURCE_GROUP" \
  --account-name "$STORAGE_ACCOUNT" \
  -o json > "$EVIDENCE_DIR/blob-service-properties.json"

az storage account file-service-properties show \
  --resource-group "$RESOURCE_GROUP" \
  --account-name "$STORAGE_ACCOUNT" \
  -o json > "$EVIDENCE_DIR/file-service-properties.json"

# RBAC.
az role assignment list \
  --scope "$STORAGE_ID" \
  -o json > "$EVIDENCE_DIR/storage-account-rbac.json"

az role assignment list \
  --scope "$SHARE_SCOPE" \
  -o json > "$EVIDENCE_DIR/file-share-rbac.json"

# Private endpoint and DNS.
az network private-endpoint list \
  --resource-group "$RESOURCE_GROUP" \
  -o json > "$EVIDENCE_DIR/private-endpoints.json"

az network private-dns zone list \
  --resource-group "$RESOURCE_GROUP" \
  -o json > "$EVIDENCE_DIR/private-dns-zones.json"

# Blob and file inventory.
az storage container list \
  --account-name "$STORAGE_ACCOUNT" \
  --include-deleted \
  --auth-mode login \
  -o json > "$EVIDENCE_DIR/blob-containers.json"

az storage blob list \
  --account-name "$STORAGE_ACCOUNT" \
  --container-name "$CONTAINER_NAME" \
  --include d v s m t \
  --auth-mode login \
  -o json > "$EVIDENCE_DIR/blob-list-deleted-versions-snapshots.json"

az storage share-rm list \
  --resource-group "$RESOURCE_GROUP" \
  --storage-account "$STORAGE_ACCOUNT" \
  --include-deleted \
  -o json > "$EVIDENCE_DIR/file-shares.json"

# Monitoring.
az monitor diagnostic-settings list \
  --resource "$BLOB_SERVICE_ID" \
  -o json > "$EVIDENCE_DIR/blob-diagnostic-settings.json"

az monitor diagnostic-settings list \
  --resource "$FILE_SERVICE_ID" \
  -o json > "$EVIDENCE_DIR/file-diagnostic-settings.json"

az monitor metrics list \
  --resource "$BLOB_SERVICE_ID" \
  --metric "Availability" \
  --interval PT1H \
  --aggregation Average \
  -o json > "$EVIDENCE_DIR/blob-availability-metric.json"

az monitor metrics list \
  --resource "$BLOB_SERVICE_ID" \
  --metric "SuccessE2ELatency" \
  --interval PT1H \
  --aggregation Average \
  -o json > "$EVIDENCE_DIR/blob-e2e-latency-metric.json"

az monitor metrics list \
  --resource "$BLOB_SERVICE_ID" \
  --metric "Transactions" \
  --interval PT1H \
  --aggregation Total \
  -o json > "$EVIDENCE_DIR/blob-transactions-metric.json"

# Summary note.
cat > "$EVIDENCE_DIR/incident-summary.md" <<'EOF'
# Storage Troubleshooting Incident Summary

## Incident
- Incident ID:
- Date:
- User or workload affected:
- Service affected:
- Symptom:

## Scope
- Subscription:
- Resource group:
- Storage account:
- Container:
- File share:
- Client host:
- Network path:

## Findings
- DNS:
- TCP:
- Firewall:
- Private endpoint:
- RBAC:
- SAS:
- Key:
- Logs:
- Metrics:
- Recovery:

## Fix
-

## Validation
-

## Prevention
-
EOF
```

# Troubleshoot_Azure_Storage_Access_Networking_Performance_And_Data_Recovery_Verification_Commands

```bash
# Azure CLI verification.

RESOURCE_GROUP="rg-storage-core-lab-01"
STORAGE_ACCOUNT="stcorelab001"
CONTAINER_NAME="labdata"
SHARE_NAME="deptshare"

az account show -o table

az storage account show \
  --resource-group "$RESOURCE_GROUP" \
  --name "$STORAGE_ACCOUNT" \
  --query "{
    name:name,
    location:location,
    kind:kind,
    sku:sku.name,
    hns:isHnsEnabled,
    publicNetworkAccess:publicNetworkAccess,
    defaultAction:networkRuleSet.defaultAction,
    allowSharedKeyAccess:allowSharedKeyAccess,
    allowBlobPublicAccess:allowBlobPublicAccess,
    endpoints:primaryEndpoints
  }" \
  -o json

az storage account network-rule list \
  --resource-group "$RESOURCE_GROUP" \
  --account-name "$STORAGE_ACCOUNT" \
  -o table

az storage container list \
  --account-name "$STORAGE_ACCOUNT" \
  --include-deleted \
  --auth-mode login \
  -o table

az storage blob list \
  --account-name "$STORAGE_ACCOUNT" \
  --container-name "$CONTAINER_NAME" \
  --include d v s \
  --auth-mode login \
  -o table

az storage share-rm list \
  --resource-group "$RESOURCE_GROUP" \
  --storage-account "$STORAGE_ACCOUNT" \
  --include-deleted \
  -o table

az network private-endpoint list \
  --resource-group "$RESOURCE_GROUP" \
  -o table

az network private-dns zone list \
  --resource-group "$RESOURCE_GROUP" \
  -o table
```

```powershell
# Client verification.

$StorageAccountName = "stcorelab001"

Resolve-DnsName "$StorageAccountName.blob.core.windows.net"
Resolve-DnsName "$StorageAccountName.file.core.windows.net"

Test-NetConnection `
  -ComputerName "$StorageAccountName.blob.core.windows.net" `
  -Port 443

Test-NetConnection `
  -ComputerName "$StorageAccountName.file.core.windows.net" `
  -Port 445

net use

cmdkey /list
```

# Troubleshoot_Azure_Storage_Access_Networking_Performance_And_Data_Recovery_Rollback

| Scenario | Rollback Action | Validation |
|---|---|---|
| Public access enabled as temporary fix | Disable public access again after private path is fixed | `publicNetworkAccess` returns expected value |
| Temporary IP rule added | Remove IP rule after incident | Network rule list no longer shows temporary IP |
| Shared Key re-enabled for emergency | Migrate client to Entra ID or SAS and disable Shared Key again | `allowSharedKeyAccess` returns false |
| Broad SAS issued | Revoke stored access policy or rotate key if key-signed SAS was exposed | SAS no longer works |
| Wrong RBAC assignment added | Remove incorrect role assignment | Role assignment list shows least privilege |
| Private DNS changed incorrectly | Restore correct private DNS zone and VNet link | Endpoint resolves private IP |
| Private endpoint created wrong | Delete wrong endpoint and recreate for correct sub-resource | Private endpoint list shows correct `blob` or `file` |
| Deleted blob restored incorrectly | Restore intended version and preserve restored copy | Blob content matches expected version |
| File restored to wrong path | Move file to correct path and document change | User validates file location |
| Alert disabled during troubleshooting | Re-enable or replace alert after incident | Alert rule enabled |
| Diagnostic setting removed | Recreate diagnostic setting | Logs flow to workspace |
| AzCopy copied wrong test data | Delete test prefix only | Blob list confirms cleanup |
| Storage Explorer made unintended change | Use logs and soft delete/versioning/snapshot to revert | Resource state matches expected baseline |

# Troubleshoot_Azure_Storage_Access_Networking_Performance_And_Data_Recovery_Failure_Checks

| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| Blob access denied with Entra ID | Missing data-plane role | `az role assignment list --scope <account-or-container>` | Assign Storage Blob Data Reader or Contributor |
| Blob access denied with SAS | SAS missing permission, expired, wrong resource, wrong IP, or wrong protocol | Decode SAS fields | Generate correct scoped SAS |
| Key-based access fails | Shared Key disabled | `az storage account show --query allowSharedKeyAccess` | Use Entra ID or re-enable by exception |
| Anonymous read fails | Public blob access disabled | `allowBlobPublicAccess` and container ACL | Keep disabled unless explicit public design exists |
| Client times out to Blob | DNS, firewall, route, or private endpoint issue | DNS and TCP 443 test | Fix private DNS, firewall, route, or endpoint |
| Client reaches public IP instead of private IP | Private DNS missing or not linked | `Resolve-DnsName` | Link `privatelink.blob.core.windows.net` or `privatelink.file.core.windows.net` |
| SMB mount fails | TCP 445 blocked | `Test-NetConnection -Port 445` | Use private endpoint, VPN, ExpressRoute, or allowed network |
| SMB identity mount fails | Identity source, RBAC, ACL, or cached key credential issue | `net use`, `cmdkey`, RBAC, `icacls` | Remove cached credentials, fix identity, role, and ACL |
| AzCopy fails immediately | Login, SAS, endpoint, or firewall issue | AzCopy logs and endpoint tests | Fix auth and network path |
| Storage Explorer cannot browse | Tenant filter, RBAC, SAS, key, or network issue | Compare with CLI test | Reauth, adjust role/SAS, fix firewall |
| Slow Blob operations | Client distance, latency, throttling, or transaction burst | Metrics and logs | Tune client, use closer region/private path, retry/backoff |
| High E2E latency but low server latency | Client or network path is problem | Compare `SuccessE2ELatency` and `SuccessServerLatency` | Fix network/client path |
| High server latency | Storage service workload pressure or operation pattern | Metrics and logs by operation | Reduce hot paths, spread load, optimize transfer |
| Object replication not moving data | Missing versioning/change feed, policy only on one account, prefix mismatch | OR policy and service properties | Fix prerequisites and policy |
| Deleted blob not visible | Soft delete disabled or retention expired | Blob service properties | Restore from backup or other copy |
| Previous version missing | Versioning enabled after overwrite or lifecycle deleted versions | Version list and lifecycle policy | Restore from backup or other copy |
| Deleted container not restorable | Container soft delete disabled or retention expired | Container list include deleted | Recreate and restore from backup |
| Deleted file not recoverable | No snapshot, soft delete, or backup | Share snapshots and file service properties | Restore from backup or alternate source |
| Logs missing | Diagnostics not configured before event or no activity occurred | Diagnostic settings and KQL tables | Enable diagnostics and reproduce |
| Evidence incomplete | Commands run from wrong client or wrong subscription | Account context and client hostname | Rerun evidence from affected and known-good clients |

# Troubleshoot_Azure_Storage_Access_Networking_Performance_And_Data_Recovery_Related_Labs

| Lab | Relationship |
|---|---|
| 01_Create_And_Configure_Storage_Accounts_Redundancy_Encryption_And_Access.md | Provides baseline settings that can cause or prevent access issues |
| 02_Configure_Storage_Firewalls_Private_Endpoints_Access_Keys_And_SAS.md | Provides firewall, private endpoint, key, and SAS controls that are commonly misconfigured |
| 03_Configure_Blob_Containers_Tiers_Lifecycle_Versioning_And_Soft_Delete.md | Provides Blob tiering, lifecycle, versioning, and soft delete recovery controls |
| 04_Configure_Azure_Files_Shares_Snapshots_Identity_Access_And_SMB.md | Provides Azure Files SMB, identity, ACL, snapshot, and soft delete controls |
| 05_Manage_Storage_Data_With_AzCopy_Storage_Explorer_And_Object_Replication.md | Provides transfer and object replication workflows that commonly surface auth and network issues |
| 06_Configure_Storage_Diagnostic_Settings_Metrics_Logs_And_Alerts.md | Provides logs, metrics, alerts, and KQL used during troubleshooting |
| Azure Networking Private Endpoint Workbook | Supports private DNS, routing, endpoint, and firewall validation |
| Azure Identity RBAC Workbook | Supports role assignment, least privilege, Entra ID auth, and managed identity troubleshooting |
| Azure Monitor Workbook | Supports metric alerts, log queries, workbook dashboards, and RCA evidence |
| Azure Backup Workbook | Supports recovery when soft delete, versioning, or snapshots are insufficient |