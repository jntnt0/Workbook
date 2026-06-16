01_Create_VNets_Subnets_Public_IPs_And_Address_Space_Baseline.md

# Create_VNets_Subnets_Public_IPs_And_Address_Space_Baseline

# Create_VNets_Subnets_Public_IPs_And_Address_Space_Baseline_Index
01_Create_VNets_Subnets_Public_IPs_And_Address_Space_Baseline.md
Create_VNets_Subnets_Public_IPs_And_Address_Space_Baseline
Create_VNets_Subnets_Public_IPs_And_Address_Space_Baseline_Source_Basis
Create_VNets_Subnets_Public_IPs_And_Address_Space_Baseline_Mental_Model
Create_VNets_Subnets_Public_IPs_And_Address_Space_Baseline_Planning_Table
Create_VNets_Subnets_Public_IPs_And_Address_Space_Baseline_Configuration_Checklist
Create_VNets_Subnets_Public_IPs_And_Address_Space_Baseline_Azure_CLI_Skeleton
Create_VNets_Subnets_Public_IPs_And_Address_Space_Baseline_PowerShell_Skeleton
Create_VNets_Subnets_Public_IPs_And_Address_Space_Baseline_Subnet_Expansion_Skeleton
Create_VNets_Subnets_Public_IPs_And_Address_Space_Baseline_Public_IP_Skeleton
Create_VNets_Subnets_Public_IPs_And_Address_Space_Baseline_Evidence_Export_Skeleton
Create_VNets_Subnets_Public_IPs_And_Address_Space_Baseline_Verification_Commands
Create_VNets_Subnets_Public_IPs_And_Address_Space_Baseline_Rollback
Create_VNets_Subnets_Public_IPs_And_Address_Space_Baseline_Failure_Checks
Create_VNets_Subnets_Public_IPs_And_Address_Space_Baseline_Related_Labs

# Create_VNets_Subnets_Public_IPs_And_Address_Space_Baseline_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Azure Virtual Network quickstart | Creating a resource group, virtual network, subnet, and baseline validation |
| Microsoft Learn | Azure CLI `az network vnet create` | Creating a VNet with address space and initial subnet |
| Microsoft Learn | Azure PowerShell `New-AzVirtualNetwork` | Creating a VNet address space with PowerShell |
| Microsoft Learn | Azure PowerShell `Add-AzVirtualNetworkSubnetConfig` / `Set-AzVirtualNetwork` | Adding subnet configuration to a VNet and committing the change |
| Microsoft Learn | Azure public IP quickstart | Creating Standard SKU public IPv4 addresses |
| Microsoft Learn | Azure Virtual Network FAQ | Non-overlapping CIDR blocks, private address ranges, subnet limits, reserved addresses, and region scope |
| Azure operational practice | Naming, tagging, CIDR planning, subscription/resource group placement | Repeatable baseline network build |
| Azure operational practice | Validation before dependencies | Preventing broken peering, private endpoint, DNS, load balancer, and routing work later |

# Create_VNets_Subnets_Public_IPs_And_Address_Space_Baseline_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Resource group | Logical container for the VNet, subnets, public IPs, and future network dependencies |
| Region | Location boundary for the VNet. A VNet is regional, not global |
| VNet | Private Layer 3 network boundary inside Azure |
| Address space | CIDR block assigned to the VNet, such as `10.40.0.0/16` |
| Subnet | Smaller CIDR range carved from the VNet address space |
| Subnet overlap | Invalid condition where subnets use the same address range or collide with future networks |
| Private IP | IP assigned from a subnet to NICs and private resources |
| Azure reserved IPs | Azure reserves the first four and last IP address inside each subnet |
| Public IP | Azure resource used to expose supported resources publicly, such as load balancers, NAT gateways, Bastion, or VM NICs |
| Standard SKU public IP | Recommended production baseline for public IP resources |
| Zonal public IP | Public IP pinned to a specific availability zone |
| Zone-redundant public IP | Public IP spread across zones in supported regions |
| Tags | Metadata used for ownership, environment, cost, and lifecycle tracking |
| First rule | If the address plan is bad, every downstream networking workbook becomes harder |
| Blunt rule | Do not create peering, private endpoints, UDRs, or load balancers until the VNet/subnet/public IP baseline is clean |

# Create_VNets_Subnets_Public_IPs_And_Address_Space_Baseline_Planning_Table
| Item | Example | Decision |
|---|---|---|
| Subscription | `Visual Studio Enterprise` | `<subscription-name>` |
| Resource group | `rg-net-core-lab-eus2-01` | `<resource-group>` |
| Region | `eastus2` | `<region>` |
| Environment tag | `Lab` | `<environment>` |
| Owner tag | `Jonathan` | `<owner>` |
| Cost center tag | `Training` | `<cost-center>` |
| VNet name | `vnet-core-lab-eus2-01` | `<vnet-name>` |
| VNet address space | `10.40.0.0/16` | `<vnet-address-space>` |
| Workload subnet name | `snet-workload-01` | `<workload-subnet-name>` |
| Workload subnet prefix | `10.40.10.0/24` | `<workload-subnet-prefix>` |
| Management subnet name | `snet-management-01` | `<management-subnet-name>` |
| Management subnet prefix | `10.40.20.0/24` | `<management-subnet-prefix>` |
| Private endpoint subnet name | `snet-private-endpoints-01` | `<private-endpoint-subnet-name>` |
| Private endpoint subnet prefix | `10.40.30.0/24` | `<private-endpoint-subnet-prefix>` |
| Reserved subnet space | `10.40.240.0/20` | `<reserved-subnet-space>` |
| Public IP name | `pip-net-baseline-eus2-01` | `<public-ip-name>` |
| Public IP SKU | `Standard` | `<public-ip-sku>` |
| Public IP allocation | `Static` | `<public-ip-allocation>` |
| Public IP zones | `1 2 3` or none | `<zone-setting>` |
| Evidence path | `C:\Azure-Network-Baseline` | `<evidence-path>` |
| Rollback stance | Delete lab resource group if disposable | `<rollback-plan>` |

# Create_VNets_Subnets_Public_IPs_And_Address_Space_Baseline_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm Azure login | Admin Workstation / Cloud Shell | `az account show` or `Get-AzContext` | Active Azure session is visible |
| 2 | Select target subscription | Admin Workstation / Cloud Shell | `az account set --subscription "<subscription-name>"` or `Set-AzContext -Subscription "<subscription-name>"` | Commands target the correct subscription |
| 3 | Confirm region choice | Admin Workstation / Cloud Shell | `az account list-locations -o table` or `Get-AzLocation` | Target region is available |
| 4 | Confirm naming plan | Operator | Record `<resource-group>`, `<vnet-name>`, subnet names, and public IP name | Names are final before deployment |
| 5 | Confirm address plan | Operator | Record `<vnet-address-space>` and subnet prefixes | Address ranges are known and non-overlapping |
| 6 | Confirm subnet sizing | Operator | Verify each subnet leaves room for Azure reserved addresses and future growth | Subnets are not undersized |
| 7 | Create resource group | Admin Workstation / Cloud Shell | `az group create --name "<resource-group>" --location "<region>"` | Resource group exists |
| 8 | Add baseline tags to resource group | Admin Workstation / Cloud Shell | `az group update --name "<resource-group>" --tags Environment="<environment>" Owner="<owner>" CostCenter="<cost-center>"` | Tags are applied |
| 9 | Create VNet with initial subnet | Admin Workstation / Cloud Shell | `az network vnet create --resource-group "<resource-group>" --name "<vnet-name>" --location "<region>" --address-prefixes "<vnet-address-space>" --subnet-name "<workload-subnet-name>" --subnet-prefixes "<workload-subnet-prefix>"` | VNet exists with workload subnet |
| 10 | Verify VNet address space | Admin Workstation / Cloud Shell | `az network vnet show --resource-group "<resource-group>" --name "<vnet-name>" --query "addressSpace.addressPrefixes" -o table` | Intended VNet CIDR is shown |
| 11 | Verify initial subnet | Admin Workstation / Cloud Shell | `az network vnet subnet show --resource-group "<resource-group>" --vnet-name "<vnet-name>" --name "<workload-subnet-name>" -o table` | Workload subnet exists |
| 12 | Add management subnet | Admin Workstation / Cloud Shell | `az network vnet subnet create --resource-group "<resource-group>" --vnet-name "<vnet-name>" --name "<management-subnet-name>" --address-prefixes "<management-subnet-prefix>"` | Management subnet exists |
| 13 | Add private endpoint subnet | Admin Workstation / Cloud Shell | `az network vnet subnet create --resource-group "<resource-group>" --vnet-name "<vnet-name>" --name "<private-endpoint-subnet-name>" --address-prefixes "<private-endpoint-subnet-prefix>"` | Private endpoint subnet exists |
| 14 | Verify all subnets | Admin Workstation / Cloud Shell | `az network vnet subnet list --resource-group "<resource-group>" --vnet-name "<vnet-name>" -o table` | All planned subnets are listed |
| 15 | Confirm no accidental NSG association yet | Admin Workstation / Cloud Shell | `az network vnet subnet list --resource-group "<resource-group>" --vnet-name "<vnet-name>" --query "[].{Name:name,Nsg:networkSecurityGroup.id}" -o table` | NSG field is empty unless intentionally assigned |
| 16 | Confirm no accidental route table association yet | Admin Workstation / Cloud Shell | `az network vnet subnet list --resource-group "<resource-group>" --vnet-name "<vnet-name>" --query "[].{Name:name,RouteTable:routeTable.id}" -o table` | Route table field is empty unless intentionally assigned |
| 17 | Create Standard public IP | Admin Workstation / Cloud Shell | `az network public-ip create --resource-group "<resource-group>" --name "<public-ip-name>" --location "<region>" --sku Standard --allocation-method Static --version IPv4 --zone 1 2 3` | Standard static public IP exists if region supports zones |
| 18 | Use non-zonal public IP fallback if zones are unsupported | Admin Workstation / Cloud Shell | `az network public-ip create --resource-group "<resource-group>" --name "<public-ip-name>" --location "<region>" --sku Standard --allocation-method Static --version IPv4` | Standard static public IP exists |
| 19 | Verify public IP SKU and allocation | Admin Workstation / Cloud Shell | `az network public-ip show --resource-group "<resource-group>" --name "<public-ip-name>" --query "{Name:name,Sku:sku.name,Allocation:publicIPAllocationMethod,IPAddress:ipAddress}" -o table` | SKU is Standard and allocation is Static |
| 20 | Confirm VNet resource ID | Admin Workstation / Cloud Shell | `az network vnet show --resource-group "<resource-group>" --name "<vnet-name>" --query id -o tsv` | VNet resource ID is known |
| 21 | Confirm public IP resource ID | Admin Workstation / Cloud Shell | `az network public-ip show --resource-group "<resource-group>" --name "<public-ip-name>" --query id -o tsv` | Public IP resource ID is known |
| 22 | Confirm effective resource inventory | Admin Workstation / Cloud Shell | `az resource list --resource-group "<resource-group>" -o table` | Expected network resources are visible |
| 23 | Export VNet JSON | Admin Workstation / Cloud Shell | `az network vnet show --resource-group "<resource-group>" --name "<vnet-name>" -o json > vnet-baseline.json` | VNet evidence file is saved |
| 24 | Export subnet JSON | Admin Workstation / Cloud Shell | `az network vnet subnet list --resource-group "<resource-group>" --vnet-name "<vnet-name>" -o json > subnets-baseline.json` | Subnet evidence file is saved |
| 25 | Export public IP JSON | Admin Workstation / Cloud Shell | `az network public-ip show --resource-group "<resource-group>" --name "<public-ip-name>" -o json > public-ip-baseline.json` | Public IP evidence file is saved |
| 26 | Document baseline | Operator | Record VNet, address space, subnets, public IP, region, tags, and evidence path | Baseline record is complete |

# Create_VNets_Subnets_Public_IPs_And_Address_Space_Baseline_Azure_CLI_Skeleton
```bash
# Purpose: create baseline Azure VNet, subnets, and Standard public IP.
# Run from Azure Cloud Shell Bash or local Azure CLI.

subscriptionName="<subscription-name>"
resourceGroup="<resource-group>"
location="<region>"

environment="<environment>"
owner="<owner>"
costCenter="<cost-center>"

vnetName="<vnet-name>"
vnetAddressSpace="<vnet-address-space>"

workloadSubnetName="<workload-subnet-name>"
workloadSubnetPrefix="<workload-subnet-prefix>"

managementSubnetName="<management-subnet-name>"
managementSubnetPrefix="<management-subnet-prefix>"

privateEndpointSubnetName="<private-endpoint-subnet-name>"
privateEndpointSubnetPrefix="<private-endpoint-subnet-prefix>"

publicIpName="<public-ip-name>"

# Select subscription.
az account set \
  --subscription "$subscriptionName"

# Create resource group.
az group create \
  --name "$resourceGroup" \
  --location "$location"

# Apply baseline tags.
az group update \
  --name "$resourceGroup" \
  --tags Environment="$environment" Owner="$owner" CostCenter="$costCenter"

# Create VNet with first subnet.
az network vnet create \
  --resource-group "$resourceGroup" \
  --name "$vnetName" \
  --location "$location" \
  --address-prefixes "$vnetAddressSpace" \
  --subnet-name "$workloadSubnetName" \
  --subnet-prefixes "$workloadSubnetPrefix"

# Add management subnet.
az network vnet subnet create \
  --resource-group "$resourceGroup" \
  --vnet-name "$vnetName" \
  --name "$managementSubnetName" \
  --address-prefixes "$managementSubnetPrefix"

# Add private endpoint subnet.
az network vnet subnet create \
  --resource-group "$resourceGroup" \
  --vnet-name "$vnetName" \
  --name "$privateEndpointSubnetName" \
  --address-prefixes "$privateEndpointSubnetPrefix"

# Create Standard static public IP.
# Use zones only if the region supports availability zones.
az network public-ip create \
  --resource-group "$resourceGroup" \
  --name "$publicIpName" \
  --location "$location" \
  --sku Standard \
  --allocation-method Static \
  --version IPv4 \
  --zone 1 2 3

# If the zonal command fails because the region does not support zones, use this fallback.
# az network public-ip create \
#   --resource-group "$resourceGroup" \
#   --name "$publicIpName" \
#   --location "$location" \
#   --sku Standard \
#   --allocation-method Static \
#   --version IPv4
```

# Create_VNets_Subnets_Public_IPs_And_Address_Space_Baseline_PowerShell_Skeleton
```powershell
# Purpose: create baseline Azure VNet, subnets, and Standard public IP.
# Run from Azure Cloud Shell PowerShell or local PowerShell with Az module.

$SubscriptionName = "<subscription-name>"
$ResourceGroup = "<resource-group>"
$Location = "<region>"

$Environment = "<environment>"
$Owner = "<owner>"
$CostCenter = "<cost-center>"

$VNetName = "<vnet-name>"
$VNetAddressSpace = "<vnet-address-space>"

$WorkloadSubnetName = "<workload-subnet-name>"
$WorkloadSubnetPrefix = "<workload-subnet-prefix>"

$ManagementSubnetName = "<management-subnet-name>"
$ManagementSubnetPrefix = "<management-subnet-prefix>"

$PrivateEndpointSubnetName = "<private-endpoint-subnet-name>"
$PrivateEndpointSubnetPrefix = "<private-endpoint-subnet-prefix>"

$PublicIpName = "<public-ip-name>"

# Select subscription.
Set-AzContext -Subscription $SubscriptionName

# Create resource group.
New-AzResourceGroup `
  -Name $ResourceGroup `
  -Location $Location `
  -Tag @{
    Environment = $Environment
    Owner = $Owner
    CostCenter = $CostCenter
  }

# Create VNet.
$VirtualNetwork = New-AzVirtualNetwork `
  -Name $VNetName `
  -ResourceGroupName $ResourceGroup `
  -Location $Location `
  -AddressPrefix $VNetAddressSpace

# Add workload subnet.
$VirtualNetwork = Add-AzVirtualNetworkSubnetConfig `
  -Name $WorkloadSubnetName `
  -VirtualNetwork $VirtualNetwork `
  -AddressPrefix $WorkloadSubnetPrefix

# Add management subnet.
$VirtualNetwork = Add-AzVirtualNetworkSubnetConfig `
  -Name $ManagementSubnetName `
  -VirtualNetwork $VirtualNetwork `
  -AddressPrefix $ManagementSubnetPrefix

# Add private endpoint subnet.
$VirtualNetwork = Add-AzVirtualNetworkSubnetConfig `
  -Name $PrivateEndpointSubnetName `
  -VirtualNetwork $VirtualNetwork `
  -AddressPrefix $PrivateEndpointSubnetPrefix

# Commit subnet configuration.
$VirtualNetwork | Set-AzVirtualNetwork

# Create Standard static public IP.
# Use zones only if the target region supports availability zones.
New-AzPublicIpAddress `
  -Name $PublicIpName `
  -ResourceGroupName $ResourceGroup `
  -Location $Location `
  -Sku Standard `
  -AllocationMethod Static `
  -IpAddressVersion IPv4 `
  -Zone 1,2,3

# If the zonal command fails because the region does not support zones, use this fallback.
# New-AzPublicIpAddress `
#   -Name $PublicIpName `
#   -ResourceGroupName $ResourceGroup `
#   -Location $Location `
#   -Sku Standard `
#   -AllocationMethod Static `
#   -IpAddressVersion IPv4
```

# Create_VNets_Subnets_Public_IPs_And_Address_Space_Baseline_Subnet_Expansion_Skeleton
```bash
# Purpose: add an extra subnet after the VNet exists.
# Use only when the new subnet prefix is inside the VNet address space and does not overlap existing subnets.

resourceGroup="<resource-group>"
vnetName="<vnet-name>"
newSubnetName="<new-subnet-name>"
newSubnetPrefix="<new-subnet-prefix>"

# Show current address space and subnets before adding anything.
az network vnet show \
  --resource-group "$resourceGroup" \
  --name "$vnetName" \
  --query "{VNet:name,AddressSpace:addressSpace.addressPrefixes}" \
  -o table

az network vnet subnet list \
  --resource-group "$resourceGroup" \
  --vnet-name "$vnetName" \
  --query "[].{Name:name,Prefix:addressPrefix}" \
  -o table

# Add subnet.
az network vnet subnet create \
  --resource-group "$resourceGroup" \
  --vnet-name "$vnetName" \
  --name "$newSubnetName" \
  --address-prefixes "$newSubnetPrefix"

# Verify subnet exists.
az network vnet subnet show \
  --resource-group "$resourceGroup" \
  --vnet-name "$vnetName" \
  --name "$newSubnetName" \
  -o table
```

# Create_VNets_Subnets_Public_IPs_And_Address_Space_Baseline_Public_IP_Skeleton
```bash
# Purpose: create a standalone Standard public IP baseline for future use by Bastion, Load Balancer, NAT Gateway, or app ingress.
# Do not attach this to anything until the later workbook calls for it.

resourceGroup="<resource-group>"
location="<region>"
publicIpName="<public-ip-name>"

# Standard zone-redundant public IPv4 address.
az network public-ip create \
  --resource-group "$resourceGroup" \
  --name "$publicIpName" \
  --location "$location" \
  --sku Standard \
  --allocation-method Static \
  --version IPv4 \
  --zone 1 2 3

# Verify.
az network public-ip show \
  --resource-group "$resourceGroup" \
  --name "$publicIpName" \
  --query "{Name:name,Sku:sku.name,Allocation:publicIPAllocationMethod,IPAddress:ipAddress,Zones:zones}" \
  -o table
```

# Create_VNets_Subnets_Public_IPs_And_Address_Space_Baseline_Evidence_Export_Skeleton
```powershell
# Purpose: export evidence for the baseline build.
# Run from PowerShell. Azure CLI must be available if using az commands.

$ResourceGroup = "<resource-group>"
$VNetName = "<vnet-name>"
$PublicIpName = "<public-ip-name>"
$EvidencePath = "C:\Azure-Network-Baseline"

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

az account show `
  --output json |
  Out-File "$EvidencePath\account-context.json" -Encoding utf8

az group show `
  --name $ResourceGroup `
  --output json |
  Out-File "$EvidencePath\resource-group.json" -Encoding utf8

az network vnet show `
  --resource-group $ResourceGroup `
  --name $VNetName `
  --output json |
  Out-File "$EvidencePath\vnet-baseline.json" -Encoding utf8

az network vnet subnet list `
  --resource-group $ResourceGroup `
  --vnet-name $VNetName `
  --output json |
  Out-File "$EvidencePath\subnets-baseline.json" -Encoding utf8

az network public-ip show `
  --resource-group $ResourceGroup `
  --name $PublicIpName `
  --output json |
  Out-File "$EvidencePath\public-ip-baseline.json" -Encoding utf8

az resource list `
  --resource-group $ResourceGroup `
  --output table |
  Out-File "$EvidencePath\resource-inventory.txt" -Encoding utf8

az network vnet subnet list `
  --resource-group $ResourceGroup `
  --vnet-name $VNetName `
  --query "[].{Name:name,Prefix:addressPrefix,Nsg:networkSecurityGroup.id,RouteTable:routeTable.id,ServiceEndpoints:serviceEndpoints}" `
  --output table |
  Out-File "$EvidencePath\subnet-associations.txt" -Encoding utf8
```

# Create_VNets_Subnets_Public_IPs_And_Address_Space_Baseline_Verification_Commands
```bash
# Account and subscription context.
az account show -o table
az group show --name "<resource-group>" -o table

# VNet verification.
az network vnet list -g "<resource-group>" -o table
az network vnet show -g "<resource-group>" -n "<vnet-name>" -o table
az network vnet show -g "<resource-group>" -n "<vnet-name>" --query "addressSpace.addressPrefixes" -o table

# Subnet verification.
az network vnet subnet list \
  --resource-group "<resource-group>" \
  --vnet-name "<vnet-name>" \
  -o table

az network vnet subnet list \
  --resource-group "<resource-group>" \
  --vnet-name "<vnet-name>" \
  --query "[].{Name:name,Prefix:addressPrefix,Nsg:networkSecurityGroup.id,RouteTable:routeTable.id}" \
  -o table

# Public IP verification.
az network public-ip list -g "<resource-group>" -o table

az network public-ip show \
  --resource-group "<resource-group>" \
  --name "<public-ip-name>" \
  --query "{Name:name,Sku:sku.name,Allocation:publicIPAllocationMethod,IPAddress:ipAddress,Zones:zones}" \
  -o table

# Resource inventory.
az resource list \
  --resource-group "<resource-group>" \
  -o table
```

```powershell
# PowerShell verification.
Get-AzContext

Get-AzResourceGroup `
  -Name "<resource-group>"

Get-AzVirtualNetwork `
  -ResourceGroupName "<resource-group>" `
  -Name "<vnet-name>"

(Get-AzVirtualNetwork `
  -ResourceGroupName "<resource-group>" `
  -Name "<vnet-name>").Subnets |
  Select-Object Name, AddressPrefix, NetworkSecurityGroup, RouteTable

Get-AzPublicIpAddress `
  -ResourceGroupName "<resource-group>" `
  -Name "<public-ip-name>" |
  Select-Object Name, ResourceGroupName, Location, Sku, PublicIpAllocationMethod, IpAddress, Zones
```

# Create_VNets_Subnets_Public_IPs_And_Address_Space_Baseline_Rollback
```bash
# Remove one subnet.
# Use only if no resources are deployed in the subnet.
az network vnet subnet delete \
  --resource-group "<resource-group>" \
  --vnet-name "<vnet-name>" \
  --name "<subnet-name>"

# Remove public IP.
# Use only if the public IP is not associated with another resource.
az network public-ip delete \
  --resource-group "<resource-group>" \
  --name "<public-ip-name>"

# Remove entire VNet.
# Use only if no dependent resources remain attached.
az network vnet delete \
  --resource-group "<resource-group>" \
  --name "<vnet-name>"

# Lab-only full cleanup.
# This deletes everything in the resource group.
az group delete \
  --name "<resource-group>" \
  --yes
```

```powershell
# Remove one subnet.
$VNet = Get-AzVirtualNetwork `
  -ResourceGroupName "<resource-group>" `
  -Name "<vnet-name>"

Remove-AzVirtualNetworkSubnetConfig `
  -Name "<subnet-name>" `
  -VirtualNetwork $VNet

$VNet | Set-AzVirtualNetwork

# Remove public IP.
Remove-AzPublicIpAddress `
  -ResourceGroupName "<resource-group>" `
  -Name "<public-ip-name>" `
  -Force

# Remove VNet.
Remove-AzVirtualNetwork `
  -ResourceGroupName "<resource-group>" `
  -Name "<vnet-name>" `
  -Force

# Lab-only full cleanup.
Remove-AzResourceGroup `
  -Name "<resource-group>" `
  -Force
```

# Create_VNets_Subnets_Public_IPs_And_Address_Space_Baseline_Failure_Checks
| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| VNet creation fails | CIDR syntax is wrong | Confirm `<vnet-address-space>` uses CIDR notation | Use a valid prefix such as `10.40.0.0/16` |
| Subnet creation fails | Subnet prefix is outside VNet address space | Compare subnet prefix to VNet address space | Choose a subnet inside the VNet CIDR |
| Subnet creation fails | Subnet overlaps another subnet | `az network vnet subnet list ... -o table` | Pick a non-overlapping range |
| Subnet is too small | Azure reserved addresses consume usable IPs | Review subnet prefix size | Use a larger subnet |
| Public IP creation fails with zone error | Region does not support requested zones | Retry without `--zone` | Create non-zonal Standard public IP |
| Public IP is not assigned an address immediately | Allocation or provisioning not finished | `az network public-ip show ...` | Wait and recheck provisioning state |
| Public IP SKU is Basic | Wrong command or portal default was used | Check `sku.name` | Recreate as Standard if baseline requires Standard |
| Resource group is in wrong region | Wrong variable value | `az group show -n "<resource-group>"` | Recreate in intended region if needed |
| VNet is in wrong region | Wrong location parameter | `az network vnet show ... --query location` | Recreate VNet in correct region |
| Tags missing | Tags were not applied or overwritten | `az group show --query tags` | Reapply tags |
| Later peering fails | Address space overlaps another VNet | Compare both VNet CIDR blocks | Readdress before building dependencies |
| Later private endpoints fail | Subnet plan was not reserved | Review private endpoint subnet | Create dedicated subnet before private endpoint work |
| Later route tables cause confusion | UDRs were attached too early | Check subnet route table associations | Remove route table until UDR workbook |
| Later NSG troubleshooting is noisy | NSGs were attached too early | Check subnet NSG associations | Remove NSG until NSG workbook |

# Create_VNets_Subnets_Public_IPs_And_Address_Space_Baseline_Related_Labs
| Lab                                                                                                | Relationship                                                          |
| -------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------- |
| `02_Configure_VNet_Peering_DNS_Resolution_And_Private_DNS_Zones.md`                                | Uses this VNet baseline as the peering and private DNS foundation     |
| `03_Configure_NSGs_ASGs_Effective_Security_Rules_And_Service_Endpoints.md`                         | Adds security controls after subnets exist                            |
| `04_Configure_Private_Endpoints_Private_Link_And_PaaS_Network_Access.md`                           | Uses private endpoint subnet planning from this baseline              |
| `05_Configure_UDRs_Route_Tables_Azure_DNS_And_Name_Resolution.md`                                  | Adds route tables and DNS behavior after the base address plan exists |
| `06_Configure_Azure_Load_Balancer_Bastion_And_Basic_Traffic_Distribution.md`                       | Uses public IP and subnet foundation for traffic distribution         |
| `07_Troubleshoot_Azure_Networking_With_Network_Watcher_Connection_Monitor_And_Effective_Routes.md` | Validates network path after resources are deployed                   |
| `08_Troubleshoot_Private_Endpoint_DNS_Routing_NSG_And_Load_Balancer_Issues.md`                     | Troubleshoots problems built on this baseline                         |