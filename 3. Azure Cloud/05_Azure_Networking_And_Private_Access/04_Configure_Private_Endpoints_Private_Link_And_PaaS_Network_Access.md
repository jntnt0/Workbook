04_Configure_Private_Endpoints_Private_Link_And_PaaS_Network_Access.md

# Configure_Private_Endpoints_Private_Link_And_PaaS_Network_Access

# Configure_Private_Endpoints_Private_Link_And_PaaS_Network_Access_Index
04_Configure_Private_Endpoints_Private_Link_And_PaaS_Network_Access.md
Configure_Private_Endpoints_Private_Link_And_PaaS_Network_Access
Configure_Private_Endpoints_Private_Link_And_PaaS_Network_Access_Source_Basis
Configure_Private_Endpoints_Private_Link_And_PaaS_Network_Access_Mental_Model
Configure_Private_Endpoints_Private_Link_And_PaaS_Network_Access_Planning_Table
Configure_Private_Endpoints_Private_Link_And_PaaS_Network_Access_Configuration_Checklist
Configure_Private_Endpoints_Private_Link_And_PaaS_Network_Access_Azure_CLI_Private_Endpoint_Skeleton
Configure_Private_Endpoints_Private_Link_And_PaaS_Network_Access_PowerShell_Private_Endpoint_Skeleton
Configure_Private_Endpoints_Private_Link_And_PaaS_Network_Access_Private_DNS_Zone_Group_Skeleton
Configure_Private_Endpoints_Private_Link_And_PaaS_Network_Access_PaaS_Public_Access_Lockdown_Skeleton
Configure_Private_Endpoints_Private_Link_And_PaaS_Network_Access_Approval_Workflow_Skeleton
Configure_Private_Endpoints_Private_Link_And_PaaS_Network_Access_DNS_Validation_Skeleton
Configure_Private_Endpoints_Private_Link_And_PaaS_Network_Access_Evidence_Export_Skeleton
Configure_Private_Endpoints_Private_Link_And_PaaS_Network_Access_Verification_Commands
Configure_Private_Endpoints_Private_Link_And_PaaS_Network_Access_Rollback
Configure_Private_Endpoints_Private_Link_And_PaaS_Network_Access_Failure_Checks
Configure_Private_Endpoints_Private_Link_And_PaaS_Network_Access_Related_Labs

# Configure_Private_Endpoints_Private_Link_And_PaaS_Network_Access_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Azure Private Endpoint overview | Private endpoint purpose, private IP NIC behavior, target subresource, connection approval states, and DNS requirement |
| Microsoft Learn | Azure Private Endpoint DNS zone values | Private endpoint DNS zone naming, CNAME behavior, private DNS zones, and custom DNS considerations |
| Microsoft Learn | Use private endpoints for Azure Storage | Storage private endpoint behavior, separate subresources, same connection string, storage firewall relationship, and DNS changes |
| Microsoft Learn | Manage Azure private endpoints | Private endpoint connection states, approve, reject, remove, and manual approval workflow |
| Azure operational practice | Private endpoint subnet planning | Isolating private endpoint NICs in a dedicated subnet |
| Azure operational practice | Public access lockdown | Validating private access before disabling public network access |
| Azure operational practice | DNS first troubleshooting | Proving FQDN resolution to private IP before blaming routing or NSGs |
| Azure operational practice | PaaS access model | Treating private endpoint access, identity authorization, and service firewall settings as separate controls |

# Configure_Private_Endpoints_Private_Link_And_PaaS_Network_Access_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Private Endpoint | Azure NIC with a private IP from your VNet that connects to a Private Link resource |
| Private Link | Platform feature that privately exposes supported Azure services over private IP reachability |
| Private Link resource | The destination service, such as Storage, SQL, Key Vault, App Service, Cosmos DB, or another supported PaaS resource |
| Target subresource | The service component being reached, such as `blob`, `file`, `queue`, `table`, `dfs`, `vault`, `sqlServer`, or `sites` |
| Private endpoint NIC | Read-only NIC created and managed by Azure for the private endpoint lifecycle |
| Private endpoint subnet | Subnet where private endpoint NICs receive private IPs |
| Connection name | Logical name of the private endpoint connection request to the PaaS resource |
| Connection state | `Approved`, `Pending`, `Rejected`, or `Disconnected` |
| Automatic approval | Approval occurs when the creating identity has required permissions on the target resource |
| Manual approval | Connection remains pending until the resource owner approves it |
| Private DNS zone | Zone such as `privatelink.blob.core.windows.net` linked to VNets for private endpoint resolution |
| DNS zone group | Association between the private endpoint and one or more Private DNS zones |
| A record | Record that maps the service private link FQDN to the private endpoint private IP |
| CNAME chain | Public service FQDN redirects toward the `privatelink` name, and private DNS overrides that answer inside linked VNets |
| Public network access | Service setting that controls access through the public service endpoint |
| Service firewall | Service-level network control, such as Storage firewall, SQL firewall, or Key Vault networking |
| Identity authorization | RBAC, keys, secrets, SAS, SQL auth, or managed identity authorization still required after network path works |
| First rule | Private endpoint fixes network path, not application permissions |
| Blunt rule | If DNS does not return the private endpoint IP, the private endpoint is not being used |

# Configure_Private_Endpoints_Private_Link_And_PaaS_Network_Access_Planning_Table
| Item | Example | Decision |
|---|---|---|
| Subscription | `Visual Studio Enterprise` | `<subscription-name>` |
| Resource group | `rg-net-core-lab-eus2-01` | `<resource-group>` |
| DNS resource group | `rg-dns-private-eus2-01` | `<dns-resource-group>` |
| Region | `eastus2` | `<region>` |
| VNet name | `vnet-core-lab-eus2-01` | `<vnet-name>` |
| Private endpoint subnet | `snet-private-endpoints-01` | `<private-endpoint-subnet-name>` |
| Workload subnet | `snet-workload-01` | `<workload-subnet-name>` |
| Private endpoint name | `pe-stblob-lab-eus2-01` | `<private-endpoint-name>` |
| Private endpoint NIC name | `nic-pe-stblob-lab-eus2-01` | `<private-endpoint-nic-name>` |
| Connection name | `pec-stblob-lab-eus2-01` | `<private-endpoint-connection-name>` |
| PaaS service type | `Storage` | `<paas-service-type>` |
| PaaS resource group | `rg-storage-lab-eus2-01` | `<paas-resource-group>` |
| PaaS resource name | `stlabprivate001` | `<paas-resource-name>` |
| PaaS resource ID | `/subscriptions/.../storageAccounts/stlabprivate001` | `<paas-resource-id>` |
| Target subresource | `blob` | `<group-id>` |
| Private DNS zone | `privatelink.blob.core.windows.net` | `<private-dns-zone-name>` |
| DNS link name | `link-vnet-core-lab-eus2-01` | `<private-dns-link-name>` |
| DNS zone group name | `default` | `<dns-zone-group-name>` |
| Test FQDN | `stlabprivate001.blob.core.windows.net` | `<test-fqdn>` |
| Expected private IP | `10.40.30.4` | `<expected-private-ip>` |
| Test VM | `vm-workload-test-01` | `<test-vm-name>` |
| Public access setting | `Disabled` after private validation | `<public-access-setting>` |
| Evidence path | `C:\Azure-PrivateEndpoint-PaaS` | `<evidence-path>` |
| Rollback stance | Re-enable public access if required, delete private endpoint, delete DNS records and links | `<rollback-plan>` |

# Configure_Private_Endpoints_Private_Link_And_PaaS_Network_Access_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm Azure login | Admin Workstation / Cloud Shell | `az account show` or `Get-AzContext` | Active Azure session is visible |
| 2 | Select target subscription | Admin Workstation / Cloud Shell | `az account set --subscription "<subscription-name>"` or `Set-AzContext -Subscription "<subscription-name>"` | Correct subscription is active |
| 3 | Confirm Microsoft.Network provider registration | Admin Workstation / Cloud Shell | `az provider show -n Microsoft.Network --query registrationState -o tsv` | Provider is registered |
| 4 | Confirm PaaS resource provider registration | Admin Workstation / Cloud Shell | `az provider show -n Microsoft.Storage --query registrationState -o tsv` | Provider is registered for selected service |
| 5 | Confirm VNet exists | Admin Workstation / Cloud Shell | `az network vnet show -g "<resource-group>" -n "<vnet-name>"` | VNet exists |
| 6 | Confirm private endpoint subnet exists | Admin Workstation / Cloud Shell | `az network vnet subnet show -g "<resource-group>" --vnet-name "<vnet-name>" -n "<private-endpoint-subnet-name>"` | Private endpoint subnet exists |
| 7 | Confirm subnet has enough available IPs | Admin Workstation / Cloud Shell | `az network vnet subnet show -g "<resource-group>" --vnet-name "<vnet-name>" -n "<private-endpoint-subnet-name>" --query "{Name:name,Prefix:addressPrefix,IpConfigs:ipConfigurations}"` | Subnet has available private IP space |
| 8 | Confirm target PaaS resource exists | Admin Workstation / Cloud Shell | `az resource show --ids "<paas-resource-id>"` | PaaS resource exists |
| 9 | Confirm target private link group IDs | Admin Workstation / Cloud Shell | `az network private-link-resource list --id "<paas-resource-id>" -o table` | Expected group ID is visible |
| 10 | Create Private DNS zone | Admin Workstation / Cloud Shell | `az network private-dns zone create -g "<dns-resource-group>" -n "<private-dns-zone-name>"` | Private DNS zone exists |
| 11 | Link VNet to Private DNS zone | Admin Workstation / Cloud Shell | `az network private-dns link vnet create -g "<dns-resource-group>" -z "<private-dns-zone-name>" -n "<private-dns-link-name>" -v "<vnet-id>" -e false` | VNet can resolve private zone records |
| 12 | Create private endpoint | Admin Workstation / Cloud Shell | `az network private-endpoint create -g "<resource-group>" -n "<private-endpoint-name>" --vnet-name "<vnet-name>" --subnet "<private-endpoint-subnet-name>" --private-connection-resource-id "<paas-resource-id>" --group-id "<group-id>" --connection-name "<private-endpoint-connection-name>" --nic-name "<private-endpoint-nic-name>"` | Private endpoint is created |
| 13 | Associate private endpoint with DNS zone group | Admin Workstation / Cloud Shell | `az network private-endpoint dns-zone-group create -g "<resource-group>" --endpoint-name "<private-endpoint-name>" -n "<dns-zone-group-name>" --private-dns-zone "<private-dns-zone-id>" --zone-name "<private-dns-zone-name>"` | DNS zone group is attached |
| 14 | Confirm private endpoint connection state | Admin Workstation / Cloud Shell | `az network private-endpoint show -g "<resource-group>" -n "<private-endpoint-name>" --query "privateLinkServiceConnections[].privateLinkServiceConnectionState.status" -o table` | State is `Approved` or known if pending |
| 15 | Approve pending connection if required | PaaS Resource Owner | Use approval workflow skeleton | Connection state becomes `Approved` |
| 16 | Confirm private endpoint private IP | Admin Workstation / Cloud Shell | `az network private-endpoint show -g "<resource-group>" -n "<private-endpoint-name>" --query "customDnsConfigs"` | Private IP and FQDN data are visible |
| 17 | Confirm Private DNS record exists | Admin Workstation / Cloud Shell | `az network private-dns record-set a list -g "<dns-resource-group>" -z "<private-dns-zone-name>" -o table` | A record exists for the PaaS resource |
| 18 | Test DNS from workload VM | Workload VM | `Resolve-DnsName "<test-fqdn>"` | FQDN resolves to private endpoint IP |
| 19 | Test TCP access from workload VM | Workload VM | `Test-NetConnection "<test-fqdn>" -Port 443` | TCP test succeeds |
| 20 | Test PaaS data-plane access | Workload VM | Storage example: `az storage blob list --account-name "<paas-resource-name>" --auth-mode login` | Data-plane test succeeds if identity authorization is correct |
| 21 | Confirm public endpoint behavior before lock down | Admin Workstation / External Client | Resolve and test from outside VNet | Public behavior is known before change |
| 22 | Disable or restrict public network access | Admin Workstation / Cloud Shell | Use PaaS public access lockdown skeleton | PaaS service is no longer open through public endpoint |
| 23 | Re-test private DNS after public lockdown | Workload VM | `Resolve-DnsName "<test-fqdn>"` | FQDN still resolves to private IP |
| 24 | Re-test private data-plane access after public lockdown | Workload VM | Service-specific client test | Private access still works |
| 25 | Validate no direct public path remains | External Client | Service-specific public access test | Public access fails as intended |
| 26 | Export evidence | Admin Workstation / Cloud Shell | Run evidence export skeleton | Evidence files are saved |
| 27 | Document final state | Operator | Record private endpoint, NIC, private IP, DNS zone, A record, connection state, and public access setting | Workbook is complete |

# Configure_Private_Endpoints_Private_Link_And_PaaS_Network_Access_Azure_CLI_Private_Endpoint_Skeleton
```bash
# Purpose: create a private endpoint for a PaaS resource.
# Example defaults to Azure Storage Blob.
# Run from Azure Cloud Shell Bash or local Azure CLI.

subscriptionName="<subscription-name>"
resourceGroup="<resource-group>"
dnsResourceGroup="<dns-resource-group>"
location="<region>"

vnetName="<vnet-name>"
privateEndpointSubnetName="<private-endpoint-subnet-name>"

paasResourceGroup="<paas-resource-group>"
paasResourceName="<paas-resource-name>"
groupId="<group-id>"

privateEndpointName="<private-endpoint-name>"
privateEndpointNicName="<private-endpoint-nic-name>"
privateEndpointConnectionName="<private-endpoint-connection-name>"

privateDnsZoneName="<private-dns-zone-name>"
privateDnsLinkName="<private-dns-link-name>"
dnsZoneGroupName="<dns-zone-group-name>"

az account set \
  --subscription "$subscriptionName"

# Capture VNet ID.
vnetId=$(az network vnet show \
  --resource-group "$resourceGroup" \
  --name "$vnetName" \
  --query id \
  --output tsv)

# Capture PaaS resource ID.
# Storage example.
paasResourceId=$(az storage account show \
  --resource-group "$paasResourceGroup" \
  --name "$paasResourceName" \
  --query id \
  --output tsv)

# Confirm available private link group IDs.
az network private-link-resource list \
  --id "$paasResourceId" \
  --output table

# Create Private DNS zone.
az network private-dns zone create \
  --resource-group "$dnsResourceGroup" \
  --name "$privateDnsZoneName"

# Link VNet to Private DNS zone for resolution.
az network private-dns link vnet create \
  --resource-group "$dnsResourceGroup" \
  --zone-name "$privateDnsZoneName" \
  --name "$privateDnsLinkName" \
  --virtual-network "$vnetId" \
  --registration-enabled false

# Create private endpoint.
az network private-endpoint create \
  --resource-group "$resourceGroup" \
  --name "$privateEndpointName" \
  --location "$location" \
  --vnet-name "$vnetName" \
  --subnet "$privateEndpointSubnetName" \
  --private-connection-resource-id "$paasResourceId" \
  --group-id "$groupId" \
  --connection-name "$privateEndpointConnectionName" \
  --nic-name "$privateEndpointNicName"

# Capture Private DNS zone ID.
privateDnsZoneId=$(az network private-dns zone show \
  --resource-group "$dnsResourceGroup" \
  --name "$privateDnsZoneName" \
  --query id \
  --output tsv)

# Attach DNS zone group to private endpoint.
az network private-endpoint dns-zone-group create \
  --resource-group "$resourceGroup" \
  --endpoint-name "$privateEndpointName" \
  --name "$dnsZoneGroupName" \
  --private-dns-zone "$privateDnsZoneId" \
  --zone-name "$privateDnsZoneName"

# Verify private endpoint.
az network private-endpoint show \
  --resource-group "$resourceGroup" \
  --name "$privateEndpointName" \
  --query "{Name:name,ProvisioningState:provisioningState,CustomDnsConfigs:customDnsConfigs}" \
  --output json

# Verify connection state.
az network private-endpoint show \
  --resource-group "$resourceGroup" \
  --name "$privateEndpointName" \
  --query "privateLinkServiceConnections[].privateLinkServiceConnectionState" \
  --output table
```

# Configure_Private_Endpoints_Private_Link_And_PaaS_Network_Access_PowerShell_Private_Endpoint_Skeleton
```powershell
# Purpose: create a private endpoint for a PaaS resource.
# Example defaults to Azure Storage Blob.
# Run from Azure Cloud Shell PowerShell or local PowerShell with Az.Network.

$SubscriptionName = "<subscription-name>"
$ResourceGroup = "<resource-group>"
$DnsResourceGroup = "<dns-resource-group>"
$Location = "<region>"

$VNetName = "<vnet-name>"
$PrivateEndpointSubnetName = "<private-endpoint-subnet-name>"

$PaaSResourceGroup = "<paas-resource-group>"
$PaaSResourceName = "<paas-resource-name>"
$GroupId = "<group-id>"

$PrivateEndpointName = "<private-endpoint-name>"
$PrivateEndpointNicName = "<private-endpoint-nic-name>"
$PrivateEndpointConnectionName = "<private-endpoint-connection-name>"

$PrivateDnsZoneName = "<private-dns-zone-name>"
$PrivateDnsLinkName = "<private-dns-link-name>"
$DnsZoneGroupName = "<dns-zone-group-name>"

Set-AzContext -Subscription $SubscriptionName

# Get VNet and subnet.
$VNet = Get-AzVirtualNetwork `
  -ResourceGroupName $ResourceGroup `
  -Name $VNetName

$Subnet = $VNet.Subnets | Where-Object Name -eq $PrivateEndpointSubnetName

# Get PaaS resource.
# Storage example.
$StorageAccount = Get-AzStorageAccount `
  -ResourceGroupName $PaaSResourceGroup `
  -Name $PaaSResourceName

# Create private link service connection.
$PrivateLinkServiceConnection = New-AzPrivateLinkServiceConnection `
  -Name $PrivateEndpointConnectionName `
  -PrivateLinkServiceId $StorageAccount.Id `
  -GroupId $GroupId

# Create private endpoint.
New-AzPrivateEndpoint `
  -ResourceGroupName $ResourceGroup `
  -Name $PrivateEndpointName `
  -Location $Location `
  -Subnet $Subnet `
  -PrivateLinkServiceConnection $PrivateLinkServiceConnection `
  -CustomNetworkInterfaceName $PrivateEndpointNicName

# Create Private DNS zone.
$PrivateDnsZone = New-AzPrivateDnsZone `
  -ResourceGroupName $DnsResourceGroup `
  -Name $PrivateDnsZoneName

# Link VNet to Private DNS zone.
New-AzPrivateDnsVirtualNetworkLink `
  -ResourceGroupName $DnsResourceGroup `
  -ZoneName $PrivateDnsZoneName `
  -Name $PrivateDnsLinkName `
  -VirtualNetworkId $VNet.Id `
  -EnableRegistration:$false

# Create DNS zone group.
$PrivateEndpoint = Get-AzPrivateEndpoint `
  -ResourceGroupName $ResourceGroup `
  -Name $PrivateEndpointName

$ZoneConfig = New-AzPrivateDnsZoneConfig `
  -Name $PrivateDnsZoneName `
  -PrivateDnsZoneId $PrivateDnsZone.ResourceId

New-AzPrivateDnsZoneGroup `
  -ResourceGroupName $ResourceGroup `
  -PrivateEndpointName $PrivateEndpointName `
  -Name $DnsZoneGroupName `
  -PrivateDnsZoneConfig $ZoneConfig

# Verify connection state.
Get-AzPrivateEndpoint `
  -ResourceGroupName $ResourceGroup `
  -Name $PrivateEndpointName
```

# Configure_Private_Endpoints_Private_Link_And_PaaS_Network_Access_Private_DNS_Zone_Group_Skeleton
```bash
# Purpose: attach an existing private endpoint to an existing Private DNS zone.
# Use when the private endpoint already exists but DNS integration was not configured.

resourceGroup="<resource-group>"
dnsResourceGroup="<dns-resource-group>"

privateEndpointName="<private-endpoint-name>"
privateDnsZoneName="<private-dns-zone-name>"
dnsZoneGroupName="<dns-zone-group-name>"

# Capture Private DNS zone ID.
privateDnsZoneId=$(az network private-dns zone show \
  --resource-group "$dnsResourceGroup" \
  --name "$privateDnsZoneName" \
  --query id \
  --output tsv)

# Attach DNS zone group.
az network private-endpoint dns-zone-group create \
  --resource-group "$resourceGroup" \
  --endpoint-name "$privateEndpointName" \
  --name "$dnsZoneGroupName" \
  --private-dns-zone "$privateDnsZoneId" \
  --zone-name "$privateDnsZoneName"

# Verify DNS zone group.
az network private-endpoint dns-zone-group list \
  --resource-group "$resourceGroup" \
  --endpoint-name "$privateEndpointName" \
  --output table

# Verify A records created in zone.
az network private-dns record-set a list \
  --resource-group "$dnsResourceGroup" \
  --zone-name "$privateDnsZoneName" \
  --output table
```

# Configure_Private_Endpoints_Private_Link_And_PaaS_Network_Access_PaaS_Public_Access_Lockdown_Skeleton
```bash
# Purpose: lock down public network access after private endpoint validation.
# Do not run this until private DNS and private data-plane access have been validated.

resourceGroup="<resource-group>"
paasResourceGroup="<paas-resource-group>"
paasResourceName="<paas-resource-name>"

# Storage account example.
# Disable public network access.
az storage account update \
  --resource-group "$paasResourceGroup" \
  --name "$paasResourceName" \
  --public-network-access Disabled

# Optional: also set default firewall action to Deny.
az storage account update \
  --resource-group "$paasResourceGroup" \
  --name "$paasResourceName" \
  --default-action Deny

# Verify Storage network state.
az storage account show \
  --resource-group "$paasResourceGroup" \
  --name "$paasResourceName" \
  --query "{Name:name,PublicNetworkAccess:publicNetworkAccess,DefaultAction:networkRuleSet.defaultAction}" \
  --output table

# Key Vault example.
# az keyvault update \
#   --resource-group "$paasResourceGroup" \
#   --name "$paasResourceName" \
#   --public-network-access Disabled

# SQL logical server example.
# az sql server update \
#   --resource-group "$paasResourceGroup" \
#   --name "$paasResourceName" \
#   --enable-public-network false

# App Service example.
# az webapp update \
#   --resource-group "$paasResourceGroup" \
#   --name "$paasResourceName" \
#   --set publicNetworkAccess=Disabled
```

# Configure_Private_Endpoints_Private_Link_And_PaaS_Network_Access_Approval_Workflow_Skeleton
```bash
# Purpose: inspect and approve or reject private endpoint connections.
# Commands vary by PaaS resource type. This skeleton uses generic resource IDs where possible.

paasResourceId="<paas-resource-id>"

# List private endpoint connections on a resource.
az network private-endpoint-connection list \
  --id "$paasResourceId" \
  --output table

# Show private endpoint connection details.
privateEndpointConnectionName="<private-endpoint-connection-name>"

az network private-endpoint-connection show \
  --id "$paasResourceId" \
  --name "$privateEndpointConnectionName" \
  --output json

# Approve pending connection.
az network private-endpoint-connection approve \
  --id "$paasResourceId" \
  --name "$privateEndpointConnectionName" \
  --description "Approved for private endpoint lab access"

# Reject pending connection.
# Use only when the connection request is not valid.
# az network private-endpoint-connection reject \
#   --id "$paasResourceId" \
#   --name "$privateEndpointConnectionName" \
#   --description "Rejected because request was not approved"

# Verify connection state after action.
az network private-endpoint-connection show \
  --id "$paasResourceId" \
  --name "$privateEndpointConnectionName" \
  --query "privateLinkServiceConnectionState" \
  --output table
```

```powershell
# PowerShell approval workflow.
# Use when managing connection approval from PowerShell.

$PaaSResourceId = "<paas-resource-id>"
$PrivateEndpointConnectionName = "<private-endpoint-connection-name>"

# Generic lookup pattern.
Get-AzPrivateEndpointConnection `
  -PrivateLinkResourceId $PaaSResourceId

# Approve connection.
Approve-AzPrivateEndpointConnection `
  -ResourceId "$PaaSResourceId/privateEndpointConnections/$PrivateEndpointConnectionName" `
  -Description "Approved for private endpoint lab access"

# Reject connection.
# Denied connections cannot simply be approved later. Remove and recreate the request if needed.
# Deny-AzPrivateEndpointConnection `
#   -ResourceId "$PaaSResourceId/privateEndpointConnections/$PrivateEndpointConnectionName" `
#   -Description "Rejected because request was not approved"
```

# Configure_Private_Endpoints_Private_Link_And_PaaS_Network_Access_DNS_Validation_Skeleton
```powershell
# Purpose: validate that the normal public service FQDN resolves to the private endpoint IP from inside the VNet.
# Run from a VM in a VNet linked to the Private DNS zone.

$TestFqdn = "<test-fqdn>"
$ExpectedPrivateIp = "<expected-private-ip>"
$Port = 443

# Show DNS client settings.
ipconfig /all

# Resolve normal service FQDN.
$Result = Resolve-DnsName `
  -Name $TestFqdn `
  -Type A

$Result |
  Select-Object Name, Type, IPAddress, NameHost

# Validate expected private IP.
if ($Result.IPAddress -contains $ExpectedPrivateIp) {
  Write-Host "PASS: $TestFqdn resolves to private endpoint IP $ExpectedPrivateIp"
}
else {
  Write-Host "FAIL: $TestFqdn does not resolve to expected private endpoint IP $ExpectedPrivateIp"
}

# Test TCP reachability.
Test-NetConnection `
  -ComputerName $TestFqdn `
  -Port $Port `
  -InformationLevel Detailed

# Storage example.
# This tests identity and data-plane access, not just network reachability.
# Requires Azure CLI login and correct RBAC.
az storage blob list `
  --account-name "<paas-resource-name>" `
  --container-name "<container-name>" `
  --auth-mode login `
  --output table
```

```bash
# Linux DNS validation from inside linked VNet.

testFqdn="<test-fqdn>"
expectedPrivateIp="<expected-private-ip>"

nslookup "$testFqdn"

dig "$testFqdn"

resolvedIp=$(dig +short "$testFqdn" | tail -n 1)

if [ "$resolvedIp" = "$expectedPrivateIp" ]; then
  echo "PASS: $testFqdn resolves to $expectedPrivateIp"
else
  echo "FAIL: $testFqdn resolved to $resolvedIp, expected $expectedPrivateIp"
fi

nc -vz "$testFqdn" 443
```

# Configure_Private_Endpoints_Private_Link_And_PaaS_Network_Access_Evidence_Export_Skeleton
```powershell
# Purpose: export evidence for Private Endpoint, Private Link, DNS, and PaaS network access.

$ResourceGroup = "<resource-group>"
$DnsResourceGroup = "<dns-resource-group>"
$PaaSResourceGroup = "<paas-resource-group>"

$VNetName = "<vnet-name>"
$PrivateEndpointSubnetName = "<private-endpoint-subnet-name>"
$PrivateEndpointName = "<private-endpoint-name>"
$PrivateEndpointNicName = "<private-endpoint-nic-name>"
$PrivateDnsZoneName = "<private-dns-zone-name>"
$PaaSResourceName = "<paas-resource-name>"

$EvidencePath = "C:\Azure-PrivateEndpoint-PaaS"

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

az account show `
  --output json |
  Out-File "$EvidencePath\account-context.json" -Encoding utf8

az network vnet show `
  --resource-group $ResourceGroup `
  --name $VNetName `
  --output json |
  Out-File "$EvidencePath\vnet.json" -Encoding utf8

az network vnet subnet show `
  --resource-group $ResourceGroup `
  --vnet-name $VNetName `
  --name $PrivateEndpointSubnetName `
  --output json |
  Out-File "$EvidencePath\private-endpoint-subnet.json" -Encoding utf8

az network private-endpoint show `
  --resource-group $ResourceGroup `
  --name $PrivateEndpointName `
  --output json |
  Out-File "$EvidencePath\private-endpoint.json" -Encoding utf8

az network private-endpoint dns-zone-group list `
  --resource-group $ResourceGroup `
  --endpoint-name $PrivateEndpointName `
  --output json |
  Out-File "$EvidencePath\dns-zone-groups.json" -Encoding utf8

az network nic show `
  --resource-group $ResourceGroup `
  --name $PrivateEndpointNicName `
  --output json |
  Out-File "$EvidencePath\private-endpoint-nic.json" -Encoding utf8

az network private-dns zone show `
  --resource-group $DnsResourceGroup `
  --name $PrivateDnsZoneName `
  --output json |
  Out-File "$EvidencePath\private-dns-zone.json" -Encoding utf8

az network private-dns link vnet list `
  --resource-group $DnsResourceGroup `
  --zone-name $PrivateDnsZoneName `
  --output json |
  Out-File "$EvidencePath\private-dns-vnet-links.json" -Encoding utf8

az network private-dns record-set a list `
  --resource-group $DnsResourceGroup `
  --zone-name $PrivateDnsZoneName `
  --output json |
  Out-File "$EvidencePath\private-dns-a-records.json" -Encoding utf8

# Storage account example.
az storage account show `
  --resource-group $PaaSResourceGroup `
  --name $PaaSResourceName `
  --output json |
  Out-File "$EvidencePath\storage-account-network-state.json" -Encoding utf8

az storage account network-rule list `
  --resource-group $PaaSResourceGroup `
  --account-name $PaaSResourceName `
  --output json |
  Out-File "$EvidencePath\storage-network-rules.json" -Encoding utf8
```

# Configure_Private_Endpoints_Private_Link_And_PaaS_Network_Access_Verification_Commands
```bash
# Subscription and resource context.
az account show -o table

# VNet and private endpoint subnet.
az network vnet subnet show \
  --resource-group "<resource-group>" \
  --vnet-name "<vnet-name>" \
  --name "<private-endpoint-subnet-name>" \
  --query "{Name:name,Prefix:addressPrefix,PrivateEndpointPolicies:privateEndpointNetworkPolicies,IpConfigs:ipConfigurations}" \
  --output json

# Private link group IDs for target PaaS resource.
az network private-link-resource list \
  --id "<paas-resource-id>" \
  --output table

# Private endpoint inventory.
az network private-endpoint list \
  --resource-group "<resource-group>" \
  --output table

az network private-endpoint show \
  --resource-group "<resource-group>" \
  --name "<private-endpoint-name>" \
  --query "{Name:name,ProvisioningState:provisioningState,Subnet:subnet.id,CustomDnsConfigs:customDnsConfigs}" \
  --output json

# Private endpoint connection state.
az network private-endpoint show \
  --resource-group "<resource-group>" \
  --name "<private-endpoint-name>" \
  --query "privateLinkServiceConnections[].privateLinkServiceConnectionState" \
  --output table

# Private endpoint NIC.
az network nic show \
  --resource-group "<resource-group>" \
  --name "<private-endpoint-nic-name>" \
  --query "{Name:name,PrivateIp:ipConfigurations[0].privateIPAddress,Subnet:ipConfigurations[0].subnet.id}" \
  --output table

# Private DNS zone.
az network private-dns zone show \
  --resource-group "<dns-resource-group>" \
  --name "<private-dns-zone-name>" \
  --output table

# Private DNS VNet links.
az network private-dns link vnet list \
  --resource-group "<dns-resource-group>" \
  --zone-name "<private-dns-zone-name>" \
  --output table

# Private DNS A records.
az network private-dns record-set a list \
  --resource-group "<dns-resource-group>" \
  --zone-name "<private-dns-zone-name>" \
  --output table

# DNS zone group.
az network private-endpoint dns-zone-group list \
  --resource-group "<resource-group>" \
  --endpoint-name "<private-endpoint-name>" \
  --output table

# Storage public network access state.
az storage account show \
  --resource-group "<paas-resource-group>" \
  --name "<paas-resource-name>" \
  --query "{Name:name,PublicNetworkAccess:publicNetworkAccess,DefaultAction:networkRuleSet.defaultAction}" \
  --output table
```

```powershell
# Run inside workload VM.

Resolve-DnsName "<test-fqdn>"

Test-NetConnection `
  -ComputerName "<test-fqdn>" `
  -Port 443 `
  -InformationLevel Detailed

# Confirm expected private answer.
$Result = Resolve-DnsName "<test-fqdn>" -Type A
$Result | Select-Object Name, IPAddress
```

# Configure_Private_Endpoints_Private_Link_And_PaaS_Network_Access_Rollback
```bash
# Re-enable public access if needed before removing private endpoint path.
# Storage example.
az storage account update \
  --resource-group "<paas-resource-group>" \
  --name "<paas-resource-name>" \
  --public-network-access Enabled

az storage account update \
  --resource-group "<paas-resource-group>" \
  --name "<paas-resource-name>" \
  --default-action Allow

# Delete DNS zone group from private endpoint.
az network private-endpoint dns-zone-group delete \
  --resource-group "<resource-group>" \
  --endpoint-name "<private-endpoint-name>" \
  --name "<dns-zone-group-name>" \
  --yes

# Delete private endpoint.
az network private-endpoint delete \
  --resource-group "<resource-group>" \
  --name "<private-endpoint-name>"

# Delete Private DNS A record if it remains.
az network private-dns record-set a delete \
  --resource-group "<dns-resource-group>" \
  --zone-name "<private-dns-zone-name>" \
  --name "<record-set-name>" \
  --yes

# Delete Private DNS VNet link.
az network private-dns link vnet delete \
  --resource-group "<dns-resource-group>" \
  --zone-name "<private-dns-zone-name>" \
  --name "<private-dns-link-name>" \
  --yes

# Delete Private DNS zone.
# Only do this if no other private endpoints depend on it.
az network private-dns zone delete \
  --resource-group "<dns-resource-group>" \
  --name "<private-dns-zone-name>" \
  --yes
```

```powershell
# PowerShell rollback skeleton.

$ResourceGroup = "<resource-group>"
$DnsResourceGroup = "<dns-resource-group>"
$PaaSResourceGroup = "<paas-resource-group>"

$PrivateEndpointName = "<private-endpoint-name>"
$DnsZoneGroupName = "<dns-zone-group-name>"
$PrivateDnsZoneName = "<private-dns-zone-name>"
$PrivateDnsLinkName = "<private-dns-link-name>"

# Delete DNS zone group.
Remove-AzPrivateDnsZoneGroup `
  -ResourceGroupName $ResourceGroup `
  -PrivateEndpointName $PrivateEndpointName `
  -Name $DnsZoneGroupName `
  -Force

# Delete private endpoint.
Remove-AzPrivateEndpoint `
  -ResourceGroupName $ResourceGroup `
  -Name $PrivateEndpointName `
  -Force

# Delete DNS VNet link.
Remove-AzPrivateDnsVirtualNetworkLink `
  -ResourceGroupName $DnsResourceGroup `
  -ZoneName $PrivateDnsZoneName `
  -Name $PrivateDnsLinkName

# Delete Private DNS zone if disposable.
Remove-AzPrivateDnsZone `
  -ResourceGroupName $DnsResourceGroup `
  -Name $PrivateDnsZoneName
```

# Configure_Private_Endpoints_Private_Link_And_PaaS_Network_Access_Failure_Checks
| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| Private endpoint creation fails | Wrong group ID | `az network private-link-resource list --id "<paas-resource-id>" -o table` | Use the correct target subresource, such as `blob`, `file`, `vault`, `sqlServer`, or `sites` |
| Private endpoint creation fails | PaaS resource ID is wrong | `az resource show --ids "<paas-resource-id>"` | Use the correct resource ID |
| Private endpoint remains pending | Manual approval required | Show private endpoint connection state | Resource owner must approve connection |
| Private endpoint is rejected | Request was denied | Show connection state | Delete rejected connection and recreate valid request |
| Private endpoint is disconnected | Provider removed connection | Show private endpoint and provider connection | Delete stale private endpoint and recreate if needed |
| DNS resolves to public IP | Private DNS zone missing or VNet not linked | `Resolve-DnsName "<test-fqdn>"` and list DNS links | Create correct zone and link VNet |
| DNS returns NXDOMAIN | Private DNS zone exists but missing A record | List A records in private zone | Attach DNS zone group or create correct A record |
| DNS resolves to private IP from one VM but not another | VM VNet is not linked to Private DNS zone | Check VNet links | Link the second VNet to the private DNS zone |
| DNS works in Azure but not on-premises | Custom DNS or forwarding path missing | Check DNS server and conditional forwarder | Configure DNS forwarding to Azure Private DNS Resolver or equivalent |
| Application still uses public path | Client uses hardcoded public IP or custom endpoint | Check application config and DNS | Use normal service FQDN and fix DNS |
| Storage connection fails after public access disabled | Private DNS not resolving to private IP | Resolve storage FQDN from client | Fix Private DNS zone and VNet links |
| Storage connection fails with authorization error | Network path works but identity does not | Check RBAC, keys, SAS, or account permissions | Assign correct data-plane permissions |
| Storage Blob works but DFS operations fail | Missing `dfs` private endpoint | Check required storage subresources | Add private endpoint for `dfs` when ADLS Gen2 DFS endpoint is required |
| File share access fails | Missing `file` private endpoint or SMB blocked | Check group ID and port 445 path | Add `file` private endpoint and validate SMB path |
| Key Vault access fails | Public access disabled before private DNS worked | Resolve vault FQDN and check network setting | Fix DNS and private endpoint connection state |
| SQL access fails | SQL private endpoint exists but public FQDN not resolving privately | Resolve SQL server FQDN | Create or link correct `privatelink.database.windows.net` zone |
| Portal shows private endpoint NIC but it cannot be edited | Private endpoint NIC is managed by Azure | Check NIC owner and private endpoint | Change private endpoint settings, not NIC directly |
| Static private IP needs to change | Static IP must be set at creation | Check private endpoint IP config | Recreate private endpoint with correct static IP |
| NSG troubleshooting is confusing | Private endpoint network policies are not planned | Check subnet policy and NSG association | Handle in NSG and UDR troubleshooting workbook |
| Multiple services share one wrong private zone | Wrong zone design | Review `privatelink` zone name by service | Use the correct zone per service type |
| Public access still succeeds after private endpoint | Private endpoint alone does not disable public endpoint | Check PaaS public network access setting | Disable or restrict public network access after private validation |

# Configure_Private_Endpoints_Private_Link_And_PaaS_Network_Access_Related_Labs
| Lab                                                                                                | Relationship                                                                     |
| -------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------- |
| `01_Create_VNets_Subnets_Public_IPs_And_Address_Space_Baseline.md`                                 | Provides the VNet and private endpoint subnet foundation                         |
| `02_Configure_VNet_Peering_DNS_Resolution_And_Private_DNS_Zones.md`                                | Provides Private DNS zone and VNet link concepts used here                       |
| `03_Configure_NSGs_ASGs_Effective_Security_Rules_And_Service_Endpoints.md`                         | Contrasts service endpoints with private endpoints and prepares NSG context      |
| `05_Configure_UDRs_Route_Tables_Azure_DNS_And_Name_Resolution.md`                                  | Extends DNS and routing behavior after private endpoint creation                 |
| `06_Configure_Azure_Load_Balancer_Bastion_And_Basic_Traffic_Distribution.md`                       | Uses private access paths alongside management and traffic distribution patterns |
| `07_Troubleshoot_Azure_Networking_With_Network_Watcher_Connection_Monitor_And_Effective_Routes.md` | Validates network path and routing evidence                                      |
| `08_Troubleshoot_Private_Endpoint_DNS_Routing_NSG_And_Load_Balancer_Issues.md`                     | Troubleshoots DNS, routing, NSG, and private endpoint failures                   |