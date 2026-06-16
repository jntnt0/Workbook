03_Configure_NSGs_ASGs_Effective_Security_Rules_And_Service_Endpoints.md

# Configure_NSGs_ASGs_Effective_Security_Rules_And_Service_Endpoints

# Configure_NSGs_ASGs_Effective_Security_Rules_And_Service_Endpoints_Index
03_Configure_NSGs_ASGs_Effective_Security_Rules_And_Service_Endpoints.md
Configure_NSGs_ASGs_Effective_Security_Rules_And_Service_Endpoints
Configure_NSGs_ASGs_Effective_Security_Rules_And_Service_Endpoints_Source_Basis
Configure_NSGs_ASGs_Effective_Security_Rules_And_Service_Endpoints_Mental_Model
Configure_NSGs_ASGs_Effective_Security_Rules_And_Service_Endpoints_Planning_Table
Configure_NSGs_ASGs_Effective_Security_Rules_And_Service_Endpoints_Configuration_Checklist
Configure_NSGs_ASGs_Effective_Security_Rules_And_Service_Endpoints_Azure_CLI_NSG_Skeleton
Configure_NSGs_ASGs_Effective_Security_Rules_And_Service_Endpoints_PowerShell_NSG_Skeleton
Configure_NSGs_ASGs_Effective_Security_Rules_And_Service_Endpoints_ASG_Skeleton
Configure_NSGs_ASGs_Effective_Security_Rules_And_Service_Endpoints_Service_Endpoint_Skeleton
Configure_NSGs_ASGs_Effective_Security_Rules_And_Service_Endpoints_Effective_Security_Rules_Skeleton
Configure_NSGs_ASGs_Effective_Security_Rules_And_Service_Endpoints_Evidence_Export_Skeleton
Configure_NSGs_ASGs_Effective_Security_Rules_And_Service_Endpoints_Verification_Commands
Configure_NSGs_ASGs_Effective_Security_Rules_And_Service_Endpoints_Rollback
Configure_NSGs_ASGs_Effective_Security_Rules_And_Service_Endpoints_Failure_Checks
Configure_NSGs_ASGs_Effective_Security_Rules_And_Service_Endpoints_Related_Labs

# Configure_NSGs_ASGs_Effective_Security_Rules_And_Service_Endpoints_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Azure network security groups overview | NSG rule structure, priorities, default rules, stateful behavior, service tags, and augmented rules |
| Microsoft Learn | Application security groups | ASG purpose, NSG rule use, NIC membership behavior, and same-VNet constraints |
| Microsoft Learn | Azure virtual network service endpoints | Service endpoint purpose, supported service model, subnet enablement, routing behavior, and limitations |
| Microsoft Learn | Network Watcher effective security rules | Viewing aggregated NIC security rules from subnet NSGs, NIC NSGs, and security admin rules |
| Azure operational practice | Subnet-level NSG baseline | Prefer subnet NSG association for broad control and simpler operations |
| Azure operational practice | ASG-based workload segmentation | Use ASGs for app role rules instead of hardcoding VM IP addresses |
| Azure operational practice | Effective rule validation | Validate what Azure is actually enforcing before troubleshooting application traffic |
| Azure operational practice | Service endpoint caution | Use service endpoints for supported PaaS firewall integration, but use private endpoints when private IP access is required |

# Configure_NSGs_ASGs_Effective_Security_Rules_And_Service_Endpoints_Mental_Model
| Concept | Operational Meaning |
|---|---|
| NSG | Rule set that allows or denies traffic to Azure resources at subnet or NIC scope |
| NSG rule | Direction, priority, protocol, source, destination, port, and action |
| Rule priority | Lower number wins. A priority `100` rule is processed before a priority `200` rule |
| Stateful filtering | Return traffic for an allowed flow does not need a mirror rule |
| Default inbound rules | Allow VNet inbound, allow Azure Load Balancer inbound, deny all other inbound |
| Default outbound rules | Allow VNet outbound, allow Internet outbound, deny all other outbound |
| Custom rule | Your explicit allow or deny rule that overrides defaults when priority is higher |
| Service tag | Microsoft-maintained group of IP prefixes for an Azure service or platform target |
| ASG | Logical application group for NICs, used as source or destination in NSG rules |
| ASG scope | ASG NIC membership must stay in the same VNet boundary |
| Subnet NSG | Applies to resources in the subnet |
| NIC NSG | Applies directly to a NIC and combines with subnet NSG evaluation |
| Effective security rules | What Azure actually applies to a NIC after combining relevant security rules |
| Service endpoint | Subnet setting that gives supported Azure PaaS services the identity of the VNet/subnet |
| Service endpoint route | More specific route to the Azure service with next hop type `VirtualNetworkServiceEndpoint` |
| Service endpoint DNS | DNS name still resolves to public service endpoint names, not a private endpoint IP |
| First rule | Build NSG and ASG policy after the VNet and subnet plan is stable |
| Blunt rule | Do not call a service endpoint a private endpoint. They solve different problems |

# Configure_NSGs_ASGs_Effective_Security_Rules_And_Service_Endpoints_Planning_Table
| Item | Example | Decision |
|---|---|---|
| Subscription | `Visual Studio Enterprise` | `<subscription-name>` |
| Resource group | `rg-net-core-lab-eus2-01` | `<resource-group>` |
| Region | `eastus2` | `<region>` |
| VNet name | `vnet-core-lab-eus2-01` | `<vnet-name>` |
| Workload subnet | `snet-workload-01` | `<workload-subnet-name>` |
| Management subnet | `snet-management-01` | `<management-subnet-name>` |
| NSG name | `nsg-workload-eus2-01` | `<nsg-name>` |
| Management NSG name | `nsg-management-eus2-01` | `<management-nsg-name>` |
| Web ASG | `asg-web` | `<asg-web-name>` |
| App ASG | `asg-app` | `<asg-app-name>` |
| DB ASG | `asg-db` | `<asg-db-name>` |
| Web VM NIC | `nic-vm-web-01` | `<web-nic-name>` |
| App VM NIC | `nic-vm-app-01` | `<app-nic-name>` |
| DB VM NIC | `nic-vm-db-01` | `<db-nic-name>` |
| Admin source CIDR | `203.0.113.10/32` | `<admin-source-cidr>` |
| Web inbound port | `443` | `<web-port>` |
| App inbound port | `8443` | `<app-port>` |
| DB inbound port | `1433` | `<db-port>` |
| Service endpoint target | `Microsoft.Storage` | `<service-endpoint-service>` |
| Storage account for test | `stnetlab001` | `<storage-account-name>` |
| Evidence path | `C:\Azure-NSG-ASG-ServiceEndpoints` | `<evidence-path>` |
| Rollback stance | Remove service endpoints, disassociate NSGs, delete rules, then delete NSGs/ASGs | `<rollback-plan>` |

# Configure_NSGs_ASGs_Effective_Security_Rules_And_Service_Endpoints_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm Azure login | Admin Workstation / Cloud Shell | `az account show` or `Get-AzContext` | Active Azure session is visible |
| 2 | Select target subscription | Admin Workstation / Cloud Shell | `az account set --subscription "<subscription-name>"` or `Set-AzContext -Subscription "<subscription-name>"` | Correct subscription is active |
| 3 | Confirm VNet exists | Admin Workstation / Cloud Shell | `az network vnet show -g "<resource-group>" -n "<vnet-name>"` | VNet exists |
| 4 | Confirm target subnets exist | Admin Workstation / Cloud Shell | `az network vnet subnet list -g "<resource-group>" --vnet-name "<vnet-name>" -o table` | Workload and management subnets exist |
| 5 | Confirm existing subnet NSG associations | Admin Workstation / Cloud Shell | `az network vnet subnet list -g "<resource-group>" --vnet-name "<vnet-name>" --query "[].{Name:name,Nsg:networkSecurityGroup.id}" -o table` | Current NSG state is known |
| 6 | Create workload NSG | Admin Workstation / Cloud Shell | `az network nsg create -g "<resource-group>" -n "<nsg-name>" -l "<region>"` | Workload NSG exists |
| 7 | Create management NSG | Admin Workstation / Cloud Shell | `az network nsg create -g "<resource-group>" -n "<management-nsg-name>" -l "<region>"` | Management NSG exists |
| 8 | Create ASG for web role | Admin Workstation / Cloud Shell | `az network asg create -g "<resource-group>" -n "<asg-web-name>" -l "<region>"` | Web ASG exists |
| 9 | Create ASG for app role | Admin Workstation / Cloud Shell | `az network asg create -g "<resource-group>" -n "<asg-app-name>" -l "<region>"` | App ASG exists |
| 10 | Create ASG for database role | Admin Workstation / Cloud Shell | `az network asg create -g "<resource-group>" -n "<asg-db-name>" -l "<region>"` | DB ASG exists |
| 11 | Add web NIC to web ASG | Admin Workstation / Cloud Shell | `az network nic ip-config update -g "<resource-group>" --nic-name "<web-nic-name>" -n ipconfig1 --application-security-groups "<asg-web-id>"` | Web NIC is an ASG member |
| 12 | Add app NIC to app ASG | Admin Workstation / Cloud Shell | `az network nic ip-config update -g "<resource-group>" --nic-name "<app-nic-name>" -n ipconfig1 --application-security-groups "<asg-app-id>"` | App NIC is an ASG member |
| 13 | Add DB NIC to DB ASG | Admin Workstation / Cloud Shell | `az network nic ip-config update -g "<resource-group>" --nic-name "<db-nic-name>" -n ipconfig1 --application-security-groups "<asg-db-id>"` | DB NIC is an ASG member |
| 14 | Allow internet HTTPS to web ASG | Admin Workstation / Cloud Shell | `az network nsg rule create -g "<resource-group>" --nsg-name "<nsg-name>" -n Allow-HTTPS-Internet-To-Web --priority 100 --direction Inbound --access Allow --protocol Tcp --source-address-prefixes Internet --source-port-ranges "*" --destination-asgs "<asg-web-id>" --destination-port-ranges 443` | HTTPS inbound to web ASG is allowed |
| 15 | Allow web ASG to app ASG | Admin Workstation / Cloud Shell | `az network nsg rule create -g "<resource-group>" --nsg-name "<nsg-name>" -n Allow-Web-To-App --priority 110 --direction Inbound --access Allow --protocol Tcp --source-asgs "<asg-web-id>" --source-port-ranges "*" --destination-asgs "<asg-app-id>" --destination-port-ranges "<app-port>"` | Web tier can reach app tier |
| 16 | Allow app ASG to DB ASG | Admin Workstation / Cloud Shell | `az network nsg rule create -g "<resource-group>" --nsg-name "<nsg-name>" -n Allow-App-To-DB --priority 120 --direction Inbound --access Allow --protocol Tcp --source-asgs "<asg-app-id>" --source-port-ranges "*" --destination-asgs "<asg-db-id>" --destination-port-ranges "<db-port>"` | App tier can reach DB tier |
| 17 | Deny direct traffic to DB ASG | Admin Workstation / Cloud Shell | `az network nsg rule create -g "<resource-group>" --nsg-name "<nsg-name>" -n Deny-Direct-To-DB --priority 130 --direction Inbound --access Deny --protocol "*" --source-address-prefixes "*" --source-port-ranges "*" --destination-asgs "<asg-db-id>" --destination-port-ranges "*"` | DB tier is protected from non-app sources |
| 18 | Allow admin management to management subnet | Admin Workstation / Cloud Shell | `az network nsg rule create -g "<resource-group>" --nsg-name "<management-nsg-name>" -n Allow-Admin-RDP-SSH --priority 100 --direction Inbound --access Allow --protocol "*" --source-address-prefixes "<admin-source-cidr>" --source-port-ranges "*" --destination-address-prefixes VirtualNetwork --destination-port-ranges 22 3389` | Admin source can manage systems |
| 19 | Associate workload NSG to workload subnet | Admin Workstation / Cloud Shell | `az network vnet subnet update -g "<resource-group>" --vnet-name "<vnet-name>" -n "<workload-subnet-name>" --network-security-group "<nsg-name>"` | Workload subnet uses workload NSG |
| 20 | Associate management NSG to management subnet | Admin Workstation / Cloud Shell | `az network vnet subnet update -g "<resource-group>" --vnet-name "<vnet-name>" -n "<management-subnet-name>" --network-security-group "<management-nsg-name>"` | Management subnet uses management NSG |
| 21 | Enable service endpoint on workload subnet | Admin Workstation / Cloud Shell | `az network vnet subnet update -g "<resource-group>" --vnet-name "<vnet-name>" -n "<workload-subnet-name>" --service-endpoints "<service-endpoint-service>"` | Service endpoint is enabled on subnet |
| 22 | Add service endpoint network rule to PaaS resource | Admin Workstation / Cloud Shell | `az storage account network-rule add -g "<resource-group>" --account-name "<storage-account-name>" --vnet-name "<vnet-name>" --subnet "<workload-subnet-name>"` | Storage account allows selected subnet |
| 23 | Confirm NSG rules | Admin Workstation / Cloud Shell | `az network nsg rule list -g "<resource-group>" --nsg-name "<nsg-name>" -o table` | Expected rules are listed |
| 24 | Confirm subnet NSG and service endpoint state | Admin Workstation / Cloud Shell | `az network vnet subnet show -g "<resource-group>" --vnet-name "<vnet-name>" -n "<workload-subnet-name>" --query "{Nsg:networkSecurityGroup.id,ServiceEndpoints:serviceEndpoints}"` | NSG and service endpoint are visible |
| 25 | Confirm ASG membership on NICs | Admin Workstation / Cloud Shell | `az network nic show -g "<resource-group>" -n "<web-nic-name>" --query "ipConfigurations[].applicationSecurityGroups[].id" -o table` | NIC shows ASG membership |
| 26 | Check effective security rules on web NIC | Admin Workstation / Cloud Shell | `az network nic list-effective-nsg -g "<resource-group>" -n "<web-nic-name>" -o table` | Effective rules include expected workload NSG rules |
| 27 | Check effective security rules on app NIC | Admin Workstation / Cloud Shell | `az network nic list-effective-nsg -g "<resource-group>" -n "<app-nic-name>" -o table` | Effective rules include expected app flow |
| 28 | Check effective security rules on DB NIC | Admin Workstation / Cloud Shell | `az network nic list-effective-nsg -g "<resource-group>" -n "<db-nic-name>" -o table` | Effective rules show DB allow and deny logic |
| 29 | Validate application path | Test VM / Workload VM | `Test-NetConnection <destination-private-ip> -Port <port>` | Intended ports succeed and denied ports fail |
| 30 | Export evidence | Admin Workstation / Cloud Shell | Run evidence export skeleton | NSG, ASG, service endpoint, and effective rule evidence is saved |
| 31 | Document final state | Operator | Record NSG associations, ASG mappings, service endpoint target, and test results | Workbook is complete |

# Configure_NSGs_ASGs_Effective_Security_Rules_And_Service_Endpoints_Azure_CLI_NSG_Skeleton
```bash
# Purpose: create NSGs, baseline rules, and subnet associations.
# Run from Azure Cloud Shell Bash or local Azure CLI.

subscriptionName="<subscription-name>"
resourceGroup="<resource-group>"
location="<region>"

vnetName="<vnet-name>"
workloadSubnetName="<workload-subnet-name>"
managementSubnetName="<management-subnet-name>"

nsgName="<nsg-name>"
managementNsgName="<management-nsg-name>"

adminSourceCidr="<admin-source-cidr>"

az account set \
  --subscription "$subscriptionName"

# Create workload NSG.
az network nsg create \
  --resource-group "$resourceGroup" \
  --name "$nsgName" \
  --location "$location"

# Create management NSG.
az network nsg create \
  --resource-group "$resourceGroup" \
  --name "$managementNsgName" \
  --location "$location"

# Management rule for admin source only.
az network nsg rule create \
  --resource-group "$resourceGroup" \
  --nsg-name "$managementNsgName" \
  --name Allow-Admin-RDP-SSH \
  --priority 100 \
  --direction Inbound \
  --access Allow \
  --protocol "*" \
  --source-address-prefixes "$adminSourceCidr" \
  --source-port-ranges "*" \
  --destination-address-prefixes VirtualNetwork \
  --destination-port-ranges 22 3389

# Associate workload NSG to workload subnet.
az network vnet subnet update \
  --resource-group "$resourceGroup" \
  --vnet-name "$vnetName" \
  --name "$workloadSubnetName" \
  --network-security-group "$nsgName"

# Associate management NSG to management subnet.
az network vnet subnet update \
  --resource-group "$resourceGroup" \
  --vnet-name "$vnetName" \
  --name "$managementSubnetName" \
  --network-security-group "$managementNsgName"

# Verify associations.
az network vnet subnet list \
  --resource-group "$resourceGroup" \
  --vnet-name "$vnetName" \
  --query "[].{Name:name,Nsg:networkSecurityGroup.id}" \
  --output table
```

# Configure_NSGs_ASGs_Effective_Security_Rules_And_Service_Endpoints_PowerShell_NSG_Skeleton
```powershell
# Purpose: create NSGs and associate them to subnets.
# Run from Azure Cloud Shell PowerShell or local PowerShell with Az.Network.

$SubscriptionName = "<subscription-name>"
$ResourceGroup = "<resource-group>"
$Location = "<region>"

$VNetName = "<vnet-name>"
$WorkloadSubnetName = "<workload-subnet-name>"
$ManagementSubnetName = "<management-subnet-name>"

$NsgName = "<nsg-name>"
$ManagementNsgName = "<management-nsg-name>"
$AdminSourceCidr = "<admin-source-cidr>"

Set-AzContext -Subscription $SubscriptionName

# Create workload NSG.
$WorkloadNsg = New-AzNetworkSecurityGroup `
  -ResourceGroupName $ResourceGroup `
  -Location $Location `
  -Name $NsgName

# Create management NSG with admin rule.
$AdminRule = New-AzNetworkSecurityRuleConfig `
  -Name "Allow-Admin-RDP-SSH" `
  -Description "Allow admin management from approved source" `
  -Access Allow `
  -Protocol * `
  -Direction Inbound `
  -Priority 100 `
  -SourceAddressPrefix $AdminSourceCidr `
  -SourcePortRange * `
  -DestinationAddressPrefix VirtualNetwork `
  -DestinationPortRange 22,3389

$ManagementNsg = New-AzNetworkSecurityGroup `
  -ResourceGroupName $ResourceGroup `
  -Location $Location `
  -Name $ManagementNsgName `
  -SecurityRules $AdminRule

# Associate NSGs to subnets.
$VNet = Get-AzVirtualNetwork `
  -ResourceGroupName $ResourceGroup `
  -Name $VNetName

Set-AzVirtualNetworkSubnetConfig `
  -VirtualNetwork $VNet `
  -Name $WorkloadSubnetName `
  -AddressPrefix (($VNet.Subnets | Where-Object Name -eq $WorkloadSubnetName).AddressPrefix) `
  -NetworkSecurityGroup $WorkloadNsg

Set-AzVirtualNetworkSubnetConfig `
  -VirtualNetwork $VNet `
  -Name $ManagementSubnetName `
  -AddressPrefix (($VNet.Subnets | Where-Object Name -eq $ManagementSubnetName).AddressPrefix) `
  -NetworkSecurityGroup $ManagementNsg

$VNet | Set-AzVirtualNetwork
```

# Configure_NSGs_ASGs_Effective_Security_Rules_And_Service_Endpoints_ASG_Skeleton
```bash
# Purpose: create ASGs, associate NICs, and create ASG-based NSG rules.

resourceGroup="<resource-group>"
location="<region>"
nsgName="<nsg-name>"

asgWebName="<asg-web-name>"
asgAppName="<asg-app-name>"
asgDbName="<asg-db-name>"

webNicName="<web-nic-name>"
appNicName="<app-nic-name>"
dbNicName="<db-nic-name>"

appPort="<app-port>"
dbPort="<db-port>"

# Create ASGs.
az network asg create \
  --resource-group "$resourceGroup" \
  --name "$asgWebName" \
  --location "$location"

az network asg create \
  --resource-group "$resourceGroup" \
  --name "$asgAppName" \
  --location "$location"

az network asg create \
  --resource-group "$resourceGroup" \
  --name "$asgDbName" \
  --location "$location"

# Capture ASG IDs.
asgWebId=$(az network asg show \
  --resource-group "$resourceGroup" \
  --name "$asgWebName" \
  --query id \
  --output tsv)

asgAppId=$(az network asg show \
  --resource-group "$resourceGroup" \
  --name "$asgAppName" \
  --query id \
  --output tsv)

asgDbId=$(az network asg show \
  --resource-group "$resourceGroup" \
  --name "$asgDbName" \
  --query id \
  --output tsv)

# Associate NIC IP configurations to ASGs.
az network nic ip-config update \
  --resource-group "$resourceGroup" \
  --nic-name "$webNicName" \
  --name ipconfig1 \
  --application-security-groups "$asgWebId"

az network nic ip-config update \
  --resource-group "$resourceGroup" \
  --nic-name "$appNicName" \
  --name ipconfig1 \
  --application-security-groups "$asgAppId"

az network nic ip-config update \
  --resource-group "$resourceGroup" \
  --nic-name "$dbNicName" \
  --name ipconfig1 \
  --application-security-groups "$asgDbId"

# Allow HTTPS from Internet to web ASG.
az network nsg rule create \
  --resource-group "$resourceGroup" \
  --nsg-name "$nsgName" \
  --name Allow-HTTPS-Internet-To-Web \
  --priority 100 \
  --direction Inbound \
  --access Allow \
  --protocol Tcp \
  --source-address-prefixes Internet \
  --source-port-ranges "*" \
  --destination-asgs "$asgWebId" \
  --destination-port-ranges 443

# Allow web ASG to app ASG.
az network nsg rule create \
  --resource-group "$resourceGroup" \
  --nsg-name "$nsgName" \
  --name Allow-Web-To-App \
  --priority 110 \
  --direction Inbound \
  --access Allow \
  --protocol Tcp \
  --source-asgs "$asgWebId" \
  --source-port-ranges "*" \
  --destination-asgs "$asgAppId" \
  --destination-port-ranges "$appPort"

# Allow app ASG to DB ASG.
az network nsg rule create \
  --resource-group "$resourceGroup" \
  --nsg-name "$nsgName" \
  --name Allow-App-To-DB \
  --priority 120 \
  --direction Inbound \
  --access Allow \
  --protocol Tcp \
  --source-asgs "$asgAppId" \
  --source-port-ranges "*" \
  --destination-asgs "$asgDbId" \
  --destination-port-ranges "$dbPort"

# Deny all other direct inbound traffic to DB ASG.
az network nsg rule create \
  --resource-group "$resourceGroup" \
  --nsg-name "$nsgName" \
  --name Deny-Direct-To-DB \
  --priority 130 \
  --direction Inbound \
  --access Deny \
  --protocol "*" \
  --source-address-prefixes "*" \
  --source-port-ranges "*" \
  --destination-asgs "$asgDbId" \
  --destination-port-ranges "*"

# Verify NSG rules.
az network nsg rule list \
  --resource-group "$resourceGroup" \
  --nsg-name "$nsgName" \
  --output table
```

# Configure_NSGs_ASGs_Effective_Security_Rules_And_Service_Endpoints_Service_Endpoint_Skeleton
```bash
# Purpose: enable a service endpoint on a subnet and secure a PaaS resource to that subnet.
# Example uses Microsoft.Storage.
# This is not Private Link and does not create a private IP for the service.

resourceGroup="<resource-group>"
vnetName="<vnet-name>"
subnetName="<workload-subnet-name>"
serviceEndpointService="<service-endpoint-service>"
storageAccountName="<storage-account-name>"

# Enable service endpoint on subnet.
az network vnet subnet update \
  --resource-group "$resourceGroup" \
  --vnet-name "$vnetName" \
  --name "$subnetName" \
  --service-endpoints "$serviceEndpointService"

# Confirm service endpoint state.
az network vnet subnet show \
  --resource-group "$resourceGroup" \
  --vnet-name "$vnetName" \
  --name "$subnetName" \
  --query "{Name:name,ServiceEndpoints:serviceEndpoints}" \
  --output json

# Lock down Storage account default network access.
# Use carefully. This can break access if rules are incomplete.
az storage account update \
  --resource-group "$resourceGroup" \
  --name "$storageAccountName" \
  --default-action Deny

# Add virtual network rule to Storage account.
az storage account network-rule add \
  --resource-group "$resourceGroup" \
  --account-name "$storageAccountName" \
  --vnet-name "$vnetName" \
  --subnet "$subnetName"

# Verify Storage account network rules.
az storage account network-rule list \
  --resource-group "$resourceGroup" \
  --account-name "$storageAccountName" \
  --output table
```

# Configure_NSGs_ASGs_Effective_Security_Rules_And_Service_Endpoints_Effective_Security_Rules_Skeleton
```bash
# Purpose: view effective security rules on NICs.
# This validates what Azure actually applies after subnet NSGs, NIC NSGs, and admin rules are combined.

resourceGroup="<resource-group>"

webNicName="<web-nic-name>"
appNicName="<app-nic-name>"
dbNicName="<db-nic-name>"

# Web NIC effective security rules.
az network nic list-effective-nsg \
  --resource-group "$resourceGroup" \
  --name "$webNicName" \
  --output table

# App NIC effective security rules.
az network nic list-effective-nsg \
  --resource-group "$resourceGroup" \
  --name "$appNicName" \
  --output table

# DB NIC effective security rules.
az network nic list-effective-nsg \
  --resource-group "$resourceGroup" \
  --name "$dbNicName" \
  --output table

# JSON output for evidence capture.
az network nic list-effective-nsg \
  --resource-group "$resourceGroup" \
  --name "$webNicName" \
  --output json > web-effective-security-rules.json

az network nic list-effective-nsg \
  --resource-group "$resourceGroup" \
  --name "$appNicName" \
  --output json > app-effective-security-rules.json

az network nic list-effective-nsg \
  --resource-group "$resourceGroup" \
  --name "$dbNicName" \
  --output json > db-effective-security-rules.json
```

# Configure_NSGs_ASGs_Effective_Security_Rules_And_Service_Endpoints_Evidence_Export_Skeleton
```powershell
# Purpose: export evidence for NSG, ASG, service endpoint, and effective security rule configuration.

$ResourceGroup = "<resource-group>"
$VNetName = "<vnet-name>"
$WorkloadSubnetName = "<workload-subnet-name>"
$ManagementSubnetName = "<management-subnet-name>"

$NsgName = "<nsg-name>"
$ManagementNsgName = "<management-nsg-name>"

$WebNicName = "<web-nic-name>"
$AppNicName = "<app-nic-name>"
$DbNicName = "<db-nic-name>"

$StorageAccountName = "<storage-account-name>"
$EvidencePath = "C:\Azure-NSG-ASG-ServiceEndpoints"

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

az account show `
  --output json |
  Out-File "$EvidencePath\account-context.json" -Encoding utf8

az network nsg show `
  --resource-group $ResourceGroup `
  --name $NsgName `
  --output json |
  Out-File "$EvidencePath\workload-nsg.json" -Encoding utf8

az network nsg rule list `
  --resource-group $ResourceGroup `
  --nsg-name $NsgName `
  --output json |
  Out-File "$EvidencePath\workload-nsg-rules.json" -Encoding utf8

az network nsg show `
  --resource-group $ResourceGroup `
  --name $ManagementNsgName `
  --output json |
  Out-File "$EvidencePath\management-nsg.json" -Encoding utf8

az network asg list `
  --resource-group $ResourceGroup `
  --output json |
  Out-File "$EvidencePath\application-security-groups.json" -Encoding utf8

az network vnet subnet show `
  --resource-group $ResourceGroup `
  --vnet-name $VNetName `
  --name $WorkloadSubnetName `
  --output json |
  Out-File "$EvidencePath\workload-subnet.json" -Encoding utf8

az network vnet subnet show `
  --resource-group $ResourceGroup `
  --vnet-name $VNetName `
  --name $ManagementSubnetName `
  --output json |
  Out-File "$EvidencePath\management-subnet.json" -Encoding utf8

az network nic show `
  --resource-group $ResourceGroup `
  --name $WebNicName `
  --output json |
  Out-File "$EvidencePath\web-nic.json" -Encoding utf8

az network nic show `
  --resource-group $ResourceGroup `
  --name $AppNicName `
  --output json |
  Out-File "$EvidencePath\app-nic.json" -Encoding utf8

az network nic show `
  --resource-group $ResourceGroup `
  --name $DbNicName `
  --output json |
  Out-File "$EvidencePath\db-nic.json" -Encoding utf8

az network nic list-effective-nsg `
  --resource-group $ResourceGroup `
  --name $WebNicName `
  --output json |
  Out-File "$EvidencePath\web-effective-security-rules.json" -Encoding utf8

az network nic list-effective-nsg `
  --resource-group $ResourceGroup `
  --name $AppNicName `
  --output json |
  Out-File "$EvidencePath\app-effective-security-rules.json" -Encoding utf8

az network nic list-effective-nsg `
  --resource-group $ResourceGroup `
  --name $DbNicName `
  --output json |
  Out-File "$EvidencePath\db-effective-security-rules.json" -Encoding utf8

az storage account network-rule list `
  --resource-group $ResourceGroup `
  --account-name $StorageAccountName `
  --output json |
  Out-File "$EvidencePath\storage-network-rules.json" -Encoding utf8
```

# Configure_NSGs_ASGs_Effective_Security_Rules_And_Service_Endpoints_Verification_Commands
```bash
# Subscription context.
az account show -o table

# NSG inventory.
az network nsg list \
  --resource-group "<resource-group>" \
  --output table

# NSG rules.
az network nsg rule list \
  --resource-group "<resource-group>" \
  --nsg-name "<nsg-name>" \
  --output table

az network nsg rule list \
  --resource-group "<resource-group>" \
  --nsg-name "<management-nsg-name>" \
  --output table

# Subnet NSG associations.
az network vnet subnet list \
  --resource-group "<resource-group>" \
  --vnet-name "<vnet-name>" \
  --query "[].{Name:name,Prefix:addressPrefix,Nsg:networkSecurityGroup.id,ServiceEndpoints:serviceEndpoints}" \
  --output table

# ASG inventory.
az network asg list \
  --resource-group "<resource-group>" \
  --output table

# NIC ASG membership.
az network nic show \
  --resource-group "<resource-group>" \
  --name "<web-nic-name>" \
  --query "ipConfigurations[].applicationSecurityGroups[].id" \
  --output table

az network nic show \
  --resource-group "<resource-group>" \
  --name "<app-nic-name>" \
  --query "ipConfigurations[].applicationSecurityGroups[].id" \
  --output table

az network nic show \
  --resource-group "<resource-group>" \
  --name "<db-nic-name>" \
  --query "ipConfigurations[].applicationSecurityGroups[].id" \
  --output table

# Effective security rules.
az network nic list-effective-nsg \
  --resource-group "<resource-group>" \
  --name "<web-nic-name>" \
  --output table

az network nic list-effective-nsg \
  --resource-group "<resource-group>" \
  --name "<app-nic-name>" \
  --output table

az network nic list-effective-nsg \
  --resource-group "<resource-group>" \
  --name "<db-nic-name>" \
  --output table

# Service endpoint verification.
az network vnet subnet show \
  --resource-group "<resource-group>" \
  --vnet-name "<vnet-name>" \
  --name "<workload-subnet-name>" \
  --query "{Name:name,ServiceEndpoints:serviceEndpoints}" \
  --output json

# Storage network rule verification if Microsoft.Storage was used.
az storage account network-rule list \
  --resource-group "<resource-group>" \
  --account-name "<storage-account-name>" \
  --output table
```

```powershell
# Run inside test VMs to validate expected flow behavior.

# Web to app.
Test-NetConnection `
  -ComputerName "<app-private-ip>" `
  -Port "<app-port>" `
  -InformationLevel Detailed

# App to DB.
Test-NetConnection `
  -ComputerName "<db-private-ip>" `
  -Port "<db-port>" `
  -InformationLevel Detailed

# Direct web to DB should fail if deny rule is working.
Test-NetConnection `
  -ComputerName "<db-private-ip>" `
  -Port "<db-port>" `
  -InformationLevel Detailed

# Management source to management target.
Test-NetConnection `
  -ComputerName "<management-private-ip>" `
  -Port 3389 `
  -InformationLevel Detailed

Test-NetConnection `
  -ComputerName "<management-private-ip>" `
  -Port 22 `
  -InformationLevel Detailed
```

# Configure_NSGs_ASGs_Effective_Security_Rules_And_Service_Endpoints_Rollback
```bash
# Remove service endpoint from subnet.
az network vnet subnet update \
  --resource-group "<resource-group>" \
  --vnet-name "<vnet-name>" \
  --name "<workload-subnet-name>" \
  --service-endpoints ""

# Remove Storage account VNet rule if used.
az storage account network-rule remove \
  --resource-group "<resource-group>" \
  --account-name "<storage-account-name>" \
  --vnet-name "<vnet-name>" \
  --subnet "<workload-subnet-name>"

# Optional: restore Storage account default network action.
az storage account update \
  --resource-group "<resource-group>" \
  --name "<storage-account-name>" \
  --default-action Allow

# Disassociate NSG from workload subnet.
az network vnet subnet update \
  --resource-group "<resource-group>" \
  --vnet-name "<vnet-name>" \
  --name "<workload-subnet-name>" \
  --network-security-group ""

# Disassociate NSG from management subnet.
az network vnet subnet update \
  --resource-group "<resource-group>" \
  --vnet-name "<vnet-name>" \
  --name "<management-subnet-name>" \
  --network-security-group ""

# Remove ASG membership from NICs.
az network nic ip-config update \
  --resource-group "<resource-group>" \
  --nic-name "<web-nic-name>" \
  --name ipconfig1 \
  --application-security-groups ""

az network nic ip-config update \
  --resource-group "<resource-group>" \
  --nic-name "<app-nic-name>" \
  --name ipconfig1 \
  --application-security-groups ""

az network nic ip-config update \
  --resource-group "<resource-group>" \
  --nic-name "<db-nic-name>" \
  --name ipconfig1 \
  --application-security-groups ""

# Delete NSG rules.
az network nsg rule delete \
  --resource-group "<resource-group>" \
  --nsg-name "<nsg-name>" \
  --name Allow-HTTPS-Internet-To-Web

az network nsg rule delete \
  --resource-group "<resource-group>" \
  --nsg-name "<nsg-name>" \
  --name Allow-Web-To-App

az network nsg rule delete \
  --resource-group "<resource-group>" \
  --nsg-name "<nsg-name>" \
  --name Allow-App-To-DB

az network nsg rule delete \
  --resource-group "<resource-group>" \
  --nsg-name "<nsg-name>" \
  --name Deny-Direct-To-DB

# Delete NSGs.
az network nsg delete \
  --resource-group "<resource-group>" \
  --name "<nsg-name>"

az network nsg delete \
  --resource-group "<resource-group>" \
  --name "<management-nsg-name>"

# Delete ASGs.
az network asg delete \
  --resource-group "<resource-group>" \
  --name "<asg-web-name>"

az network asg delete \
  --resource-group "<resource-group>" \
  --name "<asg-app-name>"

az network asg delete \
  --resource-group "<resource-group>" \
  --name "<asg-db-name>"
```

```powershell
# PowerShell rollback skeleton.

$ResourceGroup = "<resource-group>"
$VNetName = "<vnet-name>"
$WorkloadSubnetName = "<workload-subnet-name>"
$ManagementSubnetName = "<management-subnet-name>"

$NsgName = "<nsg-name>"
$ManagementNsgName = "<management-nsg-name>"

$VNet = Get-AzVirtualNetwork `
  -ResourceGroupName $ResourceGroup `
  -Name $VNetName

# Remove NSG association from subnets.
$WorkloadSubnet = $VNet.Subnets | Where-Object Name -eq $WorkloadSubnetName
$ManagementSubnet = $VNet.Subnets | Where-Object Name -eq $ManagementSubnetName

Set-AzVirtualNetworkSubnetConfig `
  -VirtualNetwork $VNet `
  -Name $WorkloadSubnetName `
  -AddressPrefix $WorkloadSubnet.AddressPrefix `
  -NetworkSecurityGroup $null

Set-AzVirtualNetworkSubnetConfig `
  -VirtualNetwork $VNet `
  -Name $ManagementSubnetName `
  -AddressPrefix $ManagementSubnet.AddressPrefix `
  -NetworkSecurityGroup $null

$VNet | Set-AzVirtualNetwork

# Delete NSGs.
Remove-AzNetworkSecurityGroup `
  -ResourceGroupName $ResourceGroup `
  -Name $NsgName `
  -Force

Remove-AzNetworkSecurityGroup `
  -ResourceGroupName $ResourceGroup `
  -Name $ManagementNsgName `
  -Force

# Delete ASGs.
Remove-AzApplicationSecurityGroup `
  -ResourceGroupName $ResourceGroup `
  -Name "<asg-web-name>" `
  -Force

Remove-AzApplicationSecurityGroup `
  -ResourceGroupName $ResourceGroup `
  -Name "<asg-app-name>" `
  -Force

Remove-AzApplicationSecurityGroup `
  -ResourceGroupName $ResourceGroup `
  -Name "<asg-db-name>" `
  -Force
```

# Configure_NSGs_ASGs_Effective_Security_Rules_And_Service_Endpoints_Failure_Checks
| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| Rule does not behave as expected | Priority is wrong | `az network nsg rule list ... -o table` | Move specific allow or deny rule to lower priority number |
| Rule never matches | Source or destination field is wrong | Check effective security rules on NIC | Correct source, destination, protocol, or port |
| Traffic still allowed after deny rule | Higher priority allow rule wins | Review NSG rule order | Put deny rule before broad allow where intended |
| Traffic still allowed inside VNet | Default `AllowVNetInBound` is still active | Check default effective rules | Add explicit deny with higher priority |
| Internet inbound fails | Default inbound denies internet traffic | Effective security rules on destination NIC | Add explicit inbound allow rule |
| Return traffic missing from rule set | Misunderstanding stateful behavior | Review flow direction | Do not add mirror return rule unless new flow starts opposite direction |
| Existing session stays connected after rule removal | Existing flow record remains active | Start a new connection test | Validate with new connection, not old session |
| ASG rule does not apply | NIC is not a member of ASG | `az network nic show ... applicationSecurityGroups` | Add NIC IP config to correct ASG |
| ASG association fails | NIC belongs to different VNet scope | Check NIC VNet and ASG usage | Use ASGs only within same VNet boundary |
| ASG source and destination rule fails | ASGs span different VNets | Check ASG member NIC locations | Use CIDR or separate rule strategy across VNets |
| Service endpoint does not restrict PaaS access | Subnet endpoint enabled but resource firewall not updated | Check PaaS network rules | Add VNet/subnet rule on the PaaS resource |
| Service endpoint breaks existing firewall rule | Source changes from public IP to private IP/VNet identity | Check service diagnostics and resource firewall | Add correct VNet rule before enforcing default deny |
| Service endpoint traffic still uses public DNS name | This is normal for service endpoints | Resolve service FQDN | Use Private Endpoint if private IP DNS answer is required |
| On-premises cannot access PaaS resource through service endpoint | Service endpoints do not apply to on-premises source traffic | Check source path | Add public/NAT IP rule or use Private Link design |
| Storage endpoint route ignores forced tunnel | Service endpoint route is more specific | Check effective routes | Accept behavior or redesign with private endpoints/NVA pattern |
| Effective security rules command returns missing data | Network Watcher provider or region state issue | Check Network Watcher availability and NIC state | Register provider or retry after NIC exists |
| NIC shows both subnet and NIC NSGs | Dual association complicates troubleshooting | Effective security rules | Prefer subnet NSG unless NIC-specific control is intentional |
| Load balancer health probes fail after NSG | Azure Load Balancer probe traffic blocked | Check for `AzureLoadBalancer` default or custom deny | Allow required probe traffic |
| Bastion access fails later | Management NSG blocks Bastion subnet or target VM ports | Check effective rules | Add Bastion-required rules in Bastion workbook |
| Private endpoint troubleshooting gets confusing | Service endpoint was used instead of private endpoint | Check subnet endpoint type and DNS result | Use private endpoint workbook for private IP PaaS access |

# Configure_NSGs_ASGs_Effective_Security_Rules_And_Service_Endpoints_Related_Labs
| Lab                                                                                                | Relationship                                                             |
| -------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------ |
| `01_Create_VNets_Subnets_Public_IPs_And_Address_Space_Baseline.md`                                 | Provides the VNet and subnet baseline that NSGs attach to                |
| `02_Configure_VNet_Peering_DNS_Resolution_And_Private_DNS_Zones.md`                                | Provides peered VNet and DNS context before security filtering           |
| `04_Configure_Private_Endpoints_Private_Link_And_PaaS_Network_Access.md`                           | Uses private endpoints when service endpoints are not private enough     |
| `05_Configure_UDRs_Route_Tables_Azure_DNS_And_Name_Resolution.md`                                  | Adds route tables and DNS behavior that may change path troubleshooting  |
| `06_Configure_Azure_Load_Balancer_Bastion_And_Basic_Traffic_Distribution.md`                       | Requires correct NSG rules for probes, backend ports, and Bastion access |
| `07_Troubleshoot_Azure_Networking_With_Network_Watcher_Connection_Monitor_And_Effective_Routes.md` | Uses Network Watcher to validate effective routes and connection paths   |
| `08_Troubleshoot_Private_Endpoint_DNS_Routing_NSG_And_Load_Balancer_Issues.md`                     | Troubleshoots combined DNS, routing, NSG, and load balancer failures     |