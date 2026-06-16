06_Configure_Azure_Load_Balancer_Bastion_And_Basic_Traffic_Distribution.md

# Configure_Azure_Load_Balancer_Bastion_And_Basic_Traffic_Distribution

# Configure_Azure_Load_Balancer_Bastion_And_Basic_Traffic_Distribution_Index
06_Configure_Azure_Load_Balancer_Bastion_And_Basic_Traffic_Distribution.md
Configure_Azure_Load_Balancer_Bastion_And_Basic_Traffic_Distribution
Configure_Azure_Load_Balancer_Bastion_And_Basic_Traffic_Distribution_Source_Basis
Configure_Azure_Load_Balancer_Bastion_And_Basic_Traffic_Distribution_Mental_Model
Configure_Azure_Load_Balancer_Bastion_And_Basic_Traffic_Distribution_Planning_Table
Configure_Azure_Load_Balancer_Bastion_And_Basic_Traffic_Distribution_Configuration_Checklist
Configure_Azure_Load_Balancer_Bastion_And_Basic_Traffic_Distribution_Azure_CLI_Public_Load_Balancer_Skeleton
Configure_Azure_Load_Balancer_Bastion_And_Basic_Traffic_Distribution_PowerShell_Public_Load_Balancer_Skeleton
Configure_Azure_Load_Balancer_Bastion_And_Basic_Traffic_Distribution_Backend_Pool_Association_Skeleton
Configure_Azure_Load_Balancer_Bastion_And_Basic_Traffic_Distribution_Load_Balancer_NSG_Skeleton
Configure_Azure_Load_Balancer_Bastion_And_Basic_Traffic_Distribution_Bastion_Skeleton
Configure_Azure_Load_Balancer_Bastion_And_Basic_Traffic_Distribution_Bastion_Connection_Validation_Skeleton
Configure_Azure_Load_Balancer_Bastion_And_Basic_Traffic_Distribution_Traffic_Validation_Skeleton
Configure_Azure_Load_Balancer_Bastion_And_Basic_Traffic_Distribution_Evidence_Export_Skeleton
Configure_Azure_Load_Balancer_Bastion_And_Basic_Traffic_Distribution_Verification_Commands
Configure_Azure_Load_Balancer_Bastion_And_Basic_Traffic_Distribution_Rollback
Configure_Azure_Load_Balancer_Bastion_And_Basic_Traffic_Distribution_Failure_Checks
Configure_Azure_Load_Balancer_Bastion_And_Basic_Traffic_Distribution_Related_Labs

# Configure_Azure_Load_Balancer_Bastion_And_Basic_Traffic_Distribution_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Azure Load Balancer overview | Layer 4 load balancing, frontend IPs, backend pools, health probes, rules, public load balancers, internal load balancers, and Standard Load Balancer behavior |
| Microsoft Learn | Public Load Balancer Azure CLI quickstart | Public IP creation, Standard Load Balancer creation, frontend configuration, backend pool, health probe, rule, NSG rule, and backend VM association |
| Microsoft Learn | Azure Load Balancer components | Frontend IP configuration, backend pool, health probe, load-balancing rule, inbound NAT rule, and outbound rule concepts |
| Microsoft Learn | Azure Bastion overview | Bastion managed RDP/SSH access over TLS, private VM IP access, SKU behavior, and no public IP requirement on managed VMs |
| Microsoft Learn | Deploy Azure Bastion with Azure CLI | `AzureBastionSubnet`, `/26` or larger subnet sizing, Standard public IP, `az network bastion create`, and portal connection flow |
| Azure operational practice | Standard Load Balancer baseline | Use Standard SKU public IPs and Standard Load Balancer for current builds |
| Azure operational practice | Bastion management baseline | Manage VMs over private IP and remove unnecessary public IPs from backend VMs |
| Azure operational practice | Health probe first validation | Validate backend health before blaming application routing or DNS |
| Azure operational practice | NSG alignment | Allow load-balanced frontend ports, probe behavior, and Bastion management traffic intentionally |

# Configure_Azure_Load_Balancer_Bastion_And_Basic_Traffic_Distribution_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Azure Load Balancer | Layer 4 TCP/UDP load distribution service |
| Public Load Balancer | Exposes a frontend public IP and distributes inbound internet traffic to backend resources |
| Internal Load Balancer | Exposes a private frontend IP and distributes private network traffic |
| Standard Load Balancer | Current baseline SKU for production and lab builds |
| Basic Load Balancer | Retired legacy SKU. Do not use for new builds |
| Frontend IP configuration | Listener IP on the load balancer |
| Backend pool | Group of VM NICs, VM IP configs, or VMSS instances that receive traffic |
| Health probe | Probe that decides whether a backend instance is healthy enough to receive traffic |
| Load-balancing rule | Maps frontend protocol and port to backend protocol and port using a health probe |
| Inbound NAT rule | Maps a frontend port to a specific backend VM port for direct access |
| Outbound rule | Controls outbound SNAT behavior for backend pool members |
| Floating IP | Direct server return behavior, only used for specific advanced designs |
| Idle timeout | How long idle TCP or UDP flows remain active |
| TCP reset | Sends TCP reset when idle timeout occurs, useful for clean connection handling |
| NSG alignment | Standard Load Balancer traffic still needs NSG rules permitting backend traffic |
| AzureBastionSubnet | Required subnet name for dedicated Bastion deployments |
| Bastion host | Managed RDP/SSH jump service deployed into a VNet |
| Bastion public IP | Public IP used to reach the Bastion service endpoint, not the target VMs |
| Target VM private IP | Address Bastion uses to connect to the VM |
| No VM public IP | Correct Bastion posture for management access |
| Basic traffic distribution | Client hits frontend IP, load balancer probes backend, healthy backend receives flow |
| First rule | Health probe state controls whether backend receives traffic |
| Blunt rule | If the backend port is blocked by NSG or guest firewall, the load balancer is not the real problem |

# Configure_Azure_Load_Balancer_Bastion_And_Basic_Traffic_Distribution_Planning_Table
| Item | Example | Decision |
|---|---|---|
| Subscription | `Visual Studio Enterprise` | `<subscription-name>` |
| Resource group | `rg-net-core-lab-eus2-01` | `<resource-group>` |
| Region | `eastus2` | `<region>` |
| VNet name | `vnet-core-lab-eus2-01` | `<vnet-name>` |
| Backend subnet | `snet-workload-01` | `<backend-subnet-name>` |
| Bastion subnet | `AzureBastionSubnet` | `AzureBastionSubnet` |
| Bastion subnet prefix | `10.40.250.0/26` | `<bastion-subnet-prefix>` |
| Public IP for load balancer | `pip-lb-web-eus2-01` | `<lb-public-ip-name>` |
| Public IP for Bastion | `pip-bas-eus2-01` | `<bastion-public-ip-name>` |
| Load balancer name | `lb-web-eus2-01` | `<load-balancer-name>` |
| Load balancer SKU | `Standard` | `Standard` |
| Frontend name | `fe-web-public-01` | `<frontend-name>` |
| Backend pool name | `be-web-pool-01` | `<backend-pool-name>` |
| Health probe name | `probe-http-80` | `<probe-name>` |
| Health probe protocol | `Tcp` or `Http` | `<probe-protocol>` |
| Health probe port | `80` | `<probe-port>` |
| Load balancer rule name | `rule-http-80` | `<lb-rule-name>` |
| Frontend port | `80` | `<frontend-port>` |
| Backend port | `80` | `<backend-port>` |
| Protocol | `Tcp` | `<protocol>` |
| Backend VM 1 | `vm-web-01` | `<backend-vm-1>` |
| Backend VM 2 | `vm-web-02` | `<backend-vm-2>` |
| Backend NIC 1 | `nic-vm-web-01` | `<backend-nic-1>` |
| Backend NIC 2 | `nic-vm-web-02` | `<backend-nic-2>` |
| Backend NIC IP config | `ipconfig1` | `<ipconfig-name>` |
| Workload NSG | `nsg-workload-eus2-01` | `<workload-nsg-name>` |
| Bastion name | `bas-core-eus2-01` | `<bastion-name>` |
| Bastion SKU | `Basic` or `Standard` | `<bastion-sku>` |
| Test URL | `http://<lb-public-ip>` | `<test-url>` |
| Evidence path | `C:\Azure-LB-Bastion` | `<evidence-path>` |
| Rollback stance | Delete Bastion first if cost-sensitive, remove backend associations, delete LB and public IPs | `<rollback-plan>` |

# Configure_Azure_Load_Balancer_Bastion_And_Basic_Traffic_Distribution_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm Azure login | Admin Workstation / Cloud Shell | `az account show` or `Get-AzContext` | Active Azure session is visible |
| 2 | Select target subscription | Admin Workstation / Cloud Shell | `az account set --subscription "<subscription-name>"` or `Set-AzContext -Subscription "<subscription-name>"` | Correct subscription is active |
| 3 | Confirm resource group exists | Admin Workstation / Cloud Shell | `az group show -n "<resource-group>" -o table` | Resource group exists |
| 4 | Confirm VNet exists | Admin Workstation / Cloud Shell | `az network vnet show -g "<resource-group>" -n "<vnet-name>"` | VNet exists |
| 5 | Confirm backend subnet exists | Admin Workstation / Cloud Shell | `az network vnet subnet show -g "<resource-group>" --vnet-name "<vnet-name>" -n "<backend-subnet-name>"` | Backend subnet exists |
| 6 | Confirm backend VMs exist | Admin Workstation / Cloud Shell | `az vm list -g "<resource-group>" -o table` | Backend VMs exist |
| 7 | Confirm backend NICs exist | Admin Workstation / Cloud Shell | `az network nic list -g "<resource-group>" -o table` | Backend NICs exist |
| 8 | Confirm backend services listen locally | Backend VMs | `Test-NetConnection localhost -Port <backend-port>` | Service is listening on backend port |
| 9 | Confirm guest firewall allows backend port | Backend VMs | Windows Firewall or Linux firewall check | Backend OS allows probe and data port |
| 10 | Create Standard public IP for load balancer | Admin Workstation / Cloud Shell | `az network public-ip create -g "<resource-group>" -n "<lb-public-ip-name>" --sku Standard --zone 1 2 3` | Public IP exists |
| 11 | Create Standard public load balancer | Admin Workstation / Cloud Shell | `az network lb create -g "<resource-group>" -n "<load-balancer-name>" --sku Standard --public-ip-address "<lb-public-ip-name>" --frontend-ip-name "<frontend-name>" --backend-pool-name "<backend-pool-name>"` | Load balancer exists |
| 12 | Create health probe | Admin Workstation / Cloud Shell | `az network lb probe create -g "<resource-group>" --lb-name "<load-balancer-name>" -n "<probe-name>" --protocol tcp --port "<probe-port>"` | Probe exists |
| 13 | Create load-balancing rule | Admin Workstation / Cloud Shell | `az network lb rule create -g "<resource-group>" --lb-name "<load-balancer-name>" -n "<lb-rule-name>" --protocol Tcp --frontend-port "<frontend-port>" --backend-port "<backend-port>" --frontend-ip-name "<frontend-name>" --backend-pool-name "<backend-pool-name>" --probe-name "<probe-name>" --disable-outbound-snat true --idle-timeout 15 --enable-tcp-reset true` | Rule exists |
| 14 | Associate backend NIC 1 with backend pool | Admin Workstation / Cloud Shell | `az network nic ip-config address-pool add -g "<resource-group>" --nic-name "<backend-nic-1>" --ip-config-name "<ipconfig-name>" --lb-name "<load-balancer-name>" --address-pool "<backend-pool-name>"` | NIC 1 is in backend pool |
| 15 | Associate backend NIC 2 with backend pool | Admin Workstation / Cloud Shell | `az network nic ip-config address-pool add -g "<resource-group>" --nic-name "<backend-nic-2>" --ip-config-name "<ipconfig-name>" --lb-name "<load-balancer-name>" --address-pool "<backend-pool-name>"` | NIC 2 is in backend pool |
| 16 | Create or update workload NSG rule for frontend traffic | Admin Workstation / Cloud Shell | `az network nsg rule create -g "<resource-group>" --nsg-name "<workload-nsg-name>" -n "Allow-LB-HTTP" --priority 200 --direction Inbound --access Allow --protocol Tcp --source-address-prefixes Internet --source-port-ranges "*" --destination-address-prefixes "*" --destination-port-ranges "<backend-port>"` | Backend traffic is allowed |
| 17 | Verify backend pool membership | Admin Workstation / Cloud Shell | `az network lb address-pool show -g "<resource-group>" --lb-name "<load-balancer-name>" -n "<backend-pool-name>" -o json` | Backend NICs appear in pool |
| 18 | Verify public IP address | Admin Workstation / Cloud Shell | `az network public-ip show -g "<resource-group>" -n "<lb-public-ip-name>" --query ipAddress -o tsv` | Public frontend IP is known |
| 19 | Test frontend traffic | External or test client | `curl http://<lb-public-ip>` | Traffic reaches a healthy backend |
| 20 | Confirm load distribution | External or test client | Refresh `curl` multiple times or use backend hostname output | Responses rotate or fail over according to flow behavior |
| 21 | Create Bastion subnet | Admin Workstation / Cloud Shell | `az network vnet subnet create -g "<resource-group>" --vnet-name "<vnet-name>" -n AzureBastionSubnet --address-prefixes "<bastion-subnet-prefix>"` | `AzureBastionSubnet` exists with `/26` or larger prefix |
| 22 | Create Standard public IP for Bastion | Admin Workstation / Cloud Shell | `az network public-ip create -g "<resource-group>" -n "<bastion-public-ip-name>" --sku Standard --zone 1 2 3` | Bastion public IP exists |
| 23 | Create Bastion host | Admin Workstation / Cloud Shell | `az network bastion create -g "<resource-group>" -n "<bastion-name>" --public-ip-address "<bastion-public-ip-name>" --vnet-name "<vnet-name>" --location "<region>" --sku "<bastion-sku>"` | Bastion deploys successfully |
| 24 | Remove VM public IPs if no longer needed | Admin Workstation / Cloud Shell | Check NIC IP configs and public IP association | Backend VMs no longer require direct public management |
| 25 | Connect to backend VM through Bastion | Azure Portal | VM > Connect > Bastion | RDP or SSH opens through Bastion |
| 26 | Validate management path over private IP | Bastion Session | `hostname`, `ipconfig`, `Test-NetConnection localhost -Port <backend-port>` | VM is manageable without public IP |
| 27 | Export evidence | Admin Workstation / Cloud Shell | Run evidence export skeleton | Load balancer and Bastion evidence files are saved |
| 28 | Document final state | Operator | Record public IPs, LB rules, backend pool members, health probe, Bastion subnet, Bastion SKU, and test results | Workbook is complete |

# Configure_Azure_Load_Balancer_Bastion_And_Basic_Traffic_Distribution_Azure_CLI_Public_Load_Balancer_Skeleton
```bash
# Purpose: create a Standard public Load Balancer with frontend IP, backend pool, health probe, and rule.
# Run from Azure Cloud Shell Bash or local Azure CLI.

subscriptionName="<subscription-name>"
resourceGroup="<resource-group>"
location="<region>"

lbPublicIpName="<lb-public-ip-name>"
loadBalancerName="<load-balancer-name>"
frontendName="<frontend-name>"
backendPoolName="<backend-pool-name>"
probeName="<probe-name>"
lbRuleName="<lb-rule-name>"

frontendPort="<frontend-port>"
backendPort="<backend-port>"
probePort="<probe-port>"

az account set \
  --subscription "$subscriptionName"

# Create Standard public IP for Load Balancer frontend.
az network public-ip create \
  --resource-group "$resourceGroup" \
  --name "$lbPublicIpName" \
  --location "$location" \
  --sku Standard \
  --allocation-method Static \
  --zone 1 2 3

# Create Standard public Load Balancer.
az network lb create \
  --resource-group "$resourceGroup" \
  --name "$loadBalancerName" \
  --location "$location" \
  --sku Standard \
  --public-ip-address "$lbPublicIpName" \
  --frontend-ip-name "$frontendName" \
  --backend-pool-name "$backendPoolName"

# Create health probe.
az network lb probe create \
  --resource-group "$resourceGroup" \
  --lb-name "$loadBalancerName" \
  --name "$probeName" \
  --protocol Tcp \
  --port "$probePort"

# Create load-balancing rule.
az network lb rule create \
  --resource-group "$resourceGroup" \
  --lb-name "$loadBalancerName" \
  --name "$lbRuleName" \
  --protocol Tcp \
  --frontend-port "$frontendPort" \
  --backend-port "$backendPort" \
  --frontend-ip-name "$frontendName" \
  --backend-pool-name "$backendPoolName" \
  --probe-name "$probeName" \
  --disable-outbound-snat true \
  --idle-timeout 15 \
  --enable-tcp-reset true

# Verify Load Balancer.
az network lb show \
  --resource-group "$resourceGroup" \
  --name "$loadBalancerName" \
  --output table

az network lb rule list \
  --resource-group "$resourceGroup" \
  --lb-name "$loadBalancerName" \
  --output table

az network lb probe list \
  --resource-group "$resourceGroup" \
  --lb-name "$loadBalancerName" \
  --output table
```

# Configure_Azure_Load_Balancer_Bastion_And_Basic_Traffic_Distribution_PowerShell_Public_Load_Balancer_Skeleton
```powershell
# Purpose: create a Standard public Load Balancer with frontend IP, backend pool, health probe, and rule.
# Run from Azure Cloud Shell PowerShell or local PowerShell with Az.Network.

$SubscriptionName = "<subscription-name>"
$ResourceGroup = "<resource-group>"
$Location = "<region>"

$LbPublicIpName = "<lb-public-ip-name>"
$LoadBalancerName = "<load-balancer-name>"
$FrontendName = "<frontend-name>"
$BackendPoolName = "<backend-pool-name>"
$ProbeName = "<probe-name>"
$LbRuleName = "<lb-rule-name>"

$FrontendPort = <frontend-port>
$BackendPort = <backend-port>
$ProbePort = <probe-port>

Set-AzContext -Subscription $SubscriptionName

# Create Standard public IP.
$PublicIp = New-AzPublicIpAddress `
  -ResourceGroupName $ResourceGroup `
  -Name $LbPublicIpName `
  -Location $Location `
  -Sku Standard `
  -AllocationMethod Static `
  -Zone 1,2,3

# Create frontend configuration.
$FrontendConfig = New-AzLoadBalancerFrontendIpConfig `
  -Name $FrontendName `
  -PublicIpAddress $PublicIp

# Create backend pool.
$BackendPool = New-AzLoadBalancerBackendAddressPoolConfig `
  -Name $BackendPoolName

# Create health probe.
$Probe = New-AzLoadBalancerProbeConfig `
  -Name $ProbeName `
  -Protocol Tcp `
  -Port $ProbePort `
  -IntervalInSeconds 5 `
  -ProbeCount 2

# Create load-balancing rule.
$Rule = New-AzLoadBalancerRuleConfig `
  -Name $LbRuleName `
  -Protocol Tcp `
  -FrontendPort $FrontendPort `
  -BackendPort $BackendPort `
  -FrontendIpConfiguration $FrontendConfig `
  -BackendAddressPool $BackendPool `
  -Probe $Probe `
  -IdleTimeoutInMinutes 15 `
  -EnableTcpReset `
  -DisableOutboundSNAT

# Create Load Balancer.
New-AzLoadBalancer `
  -ResourceGroupName $ResourceGroup `
  -Name $LoadBalancerName `
  -Location $Location `
  -Sku Standard `
  -FrontendIpConfiguration $FrontendConfig `
  -BackendAddressPool $BackendPool `
  -Probe $Probe `
  -LoadBalancingRule $Rule

# Verify.
Get-AzLoadBalancer `
  -ResourceGroupName $ResourceGroup `
  -Name $LoadBalancerName
```

# Configure_Azure_Load_Balancer_Bastion_And_Basic_Traffic_Distribution_Backend_Pool_Association_Skeleton
```bash
# Purpose: associate existing backend VM NIC IP configurations with the Load Balancer backend pool.

resourceGroup="<resource-group>"
loadBalancerName="<load-balancer-name>"
backendPoolName="<backend-pool-name>"
ipconfigName="<ipconfig-name>"

backendNic1="<backend-nic-1>"
backendNic2="<backend-nic-2>"

# Add backend NIC 1 to backend pool.
az network nic ip-config address-pool add \
  --resource-group "$resourceGroup" \
  --nic-name "$backendNic1" \
  --ip-config-name "$ipconfigName" \
  --lb-name "$loadBalancerName" \
  --address-pool "$backendPoolName"

# Add backend NIC 2 to backend pool.
az network nic ip-config address-pool add \
  --resource-group "$resourceGroup" \
  --nic-name "$backendNic2" \
  --ip-config-name "$ipconfigName" \
  --lb-name "$loadBalancerName" \
  --address-pool "$backendPoolName"

# Verify backend pool.
az network lb address-pool show \
  --resource-group "$resourceGroup" \
  --lb-name "$loadBalancerName" \
  --name "$backendPoolName" \
  --output json

# Verify NIC backend pool references.
az network nic show \
  --resource-group "$resourceGroup" \
  --name "$backendNic1" \
  --query "ipConfigurations[].loadBalancerBackendAddressPools[].id" \
  --output table

az network nic show \
  --resource-group "$resourceGroup" \
  --name "$backendNic2" \
  --query "ipConfigurations[].loadBalancerBackendAddressPools[].id" \
  --output table
```

```powershell
# PowerShell backend pool association.

$ResourceGroup = "<resource-group>"
$LoadBalancerName = "<load-balancer-name>"
$BackendPoolName = "<backend-pool-name>"
$IpConfigName = "<ipconfig-name>"

$BackendNic1 = "<backend-nic-1>"
$BackendNic2 = "<backend-nic-2>"

$LoadBalancer = Get-AzLoadBalancer `
  -ResourceGroupName $ResourceGroup `
  -Name $LoadBalancerName

$BackendPool = Get-AzLoadBalancerBackendAddressPoolConfig `
  -LoadBalancer $LoadBalancer `
  -Name $BackendPoolName

$Nic1 = Get-AzNetworkInterface `
  -ResourceGroupName $ResourceGroup `
  -Name $BackendNic1

$Nic1.IpConfigurations |
  Where-Object Name -eq $IpConfigName |
  ForEach-Object {
    $_.LoadBalancerBackendAddressPools = $BackendPool
  }

$Nic1 | Set-AzNetworkInterface

$Nic2 = Get-AzNetworkInterface `
  -ResourceGroupName $ResourceGroup `
  -Name $BackendNic2

$Nic2.IpConfigurations |
  Where-Object Name -eq $IpConfigName |
  ForEach-Object {
    $_.LoadBalancerBackendAddressPools = $BackendPool
  }

$Nic2 | Set-AzNetworkInterface
```

# Configure_Azure_Load_Balancer_Bastion_And_Basic_Traffic_Distribution_Load_Balancer_NSG_Skeleton
```bash
# Purpose: allow frontend traffic and probe-relevant backend traffic through the workload NSG.
# Apply this to the NSG associated with the backend subnet or backend NICs.

resourceGroup="<resource-group>"
workloadNsgName="<workload-nsg-name>"
backendPort="<backend-port>"

# Allow client traffic to backend service port.
az network nsg rule create \
  --resource-group "$resourceGroup" \
  --nsg-name "$workloadNsgName" \
  --name Allow-LB-Backend-Port \
  --priority 200 \
  --direction Inbound \
  --access Allow \
  --protocol Tcp \
  --source-address-prefixes Internet \
  --source-port-ranges "*" \
  --destination-address-prefixes "*" \
  --destination-port-ranges "$backendPort"

# Optional explicit Azure Load Balancer probe allowance.
# Default NSG rules usually include AllowAzureLoadBalancerInBound, but explicit rules are useful when custom deny rules exist.
az network nsg rule create \
  --resource-group "$resourceGroup" \
  --nsg-name "$workloadNsgName" \
  --name Allow-AzureLoadBalancer-Probe \
  --priority 210 \
  --direction Inbound \
  --access Allow \
  --protocol Tcp \
  --source-address-prefixes AzureLoadBalancer \
  --source-port-ranges "*" \
  --destination-address-prefixes "*" \
  --destination-port-ranges "$backendPort"

# Verify rules.
az network nsg rule list \
  --resource-group "$resourceGroup" \
  --nsg-name "$workloadNsgName" \
  --output table
```

# Configure_Azure_Load_Balancer_Bastion_And_Basic_Traffic_Distribution_Bastion_Skeleton
```bash
# Purpose: deploy Azure Bastion into an existing VNet.
# Dedicated Bastion requires a subnet named AzureBastionSubnet.
# Use /26 or larger for the Bastion subnet.

subscriptionName="<subscription-name>"
resourceGroup="<resource-group>"
location="<region>"

vnetName="<vnet-name>"
bastionSubnetPrefix="<bastion-subnet-prefix>"

bastionPublicIpName="<bastion-public-ip-name>"
bastionName="<bastion-name>"
bastionSku="<bastion-sku>"

az account set \
  --subscription "$subscriptionName"

# Create AzureBastionSubnet.
az network vnet subnet create \
  --resource-group "$resourceGroup" \
  --vnet-name "$vnetName" \
  --name AzureBastionSubnet \
  --address-prefixes "$bastionSubnetPrefix"

# Create Standard public IP for Bastion.
az network public-ip create \
  --resource-group "$resourceGroup" \
  --name "$bastionPublicIpName" \
  --location "$location" \
  --sku Standard \
  --allocation-method Static \
  --zone 1 2 3

# Create Bastion host.
az network bastion create \
  --resource-group "$resourceGroup" \
  --name "$bastionName" \
  --public-ip-address "$bastionPublicIpName" \
  --vnet-name "$vnetName" \
  --location "$location" \
  --sku "$bastionSku"

# Verify Bastion.
az network bastion show \
  --resource-group "$resourceGroup" \
  --name "$bastionName" \
  --output table

az network vnet subnet show \
  --resource-group "$resourceGroup" \
  --vnet-name "$vnetName" \
  --name AzureBastionSubnet \
  --query "{Name:name,Prefix:addressPrefix,RouteTable:routeTable.id,Delegations:delegations}" \
  --output json
```

```powershell
# PowerShell Bastion skeleton.

$SubscriptionName = "<subscription-name>"
$ResourceGroup = "<resource-group>"
$Location = "<region>"

$VNetName = "<vnet-name>"
$BastionSubnetPrefix = "<bastion-subnet-prefix>"

$BastionPublicIpName = "<bastion-public-ip-name>"
$BastionName = "<bastion-name>"
$BastionSku = "<bastion-sku>"

Set-AzContext -Subscription $SubscriptionName

$VNet = Get-AzVirtualNetwork `
  -ResourceGroupName $ResourceGroup `
  -Name $VNetName

Add-AzVirtualNetworkSubnetConfig `
  -Name "AzureBastionSubnet" `
  -VirtualNetwork $VNet `
  -AddressPrefix $BastionSubnetPrefix

$VNet | Set-AzVirtualNetwork

$BastionPublicIp = New-AzPublicIpAddress `
  -ResourceGroupName $ResourceGroup `
  -Name $BastionPublicIpName `
  -Location $Location `
  -Sku Standard `
  -AllocationMethod Static `
  -Zone 1,2,3

New-AzBastion `
  -ResourceGroupName $ResourceGroup `
  -Name $BastionName `
  -PublicIpAddress $BastionPublicIp `
  -VirtualNetworkId $VNet.Id `
  -Sku $BastionSku
```

# Configure_Azure_Load_Balancer_Bastion_And_Basic_Traffic_Distribution_Bastion_Connection_Validation_Skeleton
```powershell
# Purpose: validate VM management access after Bastion deployment.
# Run these checks inside the Bastion session after connecting to a VM.

hostname
whoami
ipconfig /all

# Confirm backend service is listening locally.
Test-NetConnection `
  -ComputerName localhost `
  -Port <backend-port> `
  -InformationLevel Detailed

# Confirm backend VM can reach Azure platform DNS.
Resolve-DnsName login.microsoftonline.com

# Confirm no direct public management dependency is required.
Get-NetTCPConnection |
  Where-Object {
    $_.LocalPort -in 22,3389,80,443
  } |
  Select-Object LocalAddress, LocalPort, State, OwningProcess

# Confirm Windows firewall profile and rules if Windows VM.
Get-NetFirewallProfile |
  Select-Object Name, Enabled, DefaultInboundAction, DefaultOutboundAction
```

```bash
# Linux Bastion session validation.

hostname
whoami
ip addr
ip route

# Confirm backend service is listening.
sudo ss -tulpen

# Test local service.
curl -I http://localhost:<backend-port>

# DNS validation.
nslookup login.microsoftonline.com
```

# Configure_Azure_Load_Balancer_Bastion_And_Basic_Traffic_Distribution_Traffic_Validation_Skeleton
```powershell
# Purpose: validate frontend traffic distribution and backend health behavior.

$LoadBalancerPublicIp = "<lb-public-ip>"
$FrontendPort = <frontend-port>
$TestUrl = "http://$LoadBalancerPublicIp"

# Basic TCP test.
Test-NetConnection `
  -ComputerName $LoadBalancerPublicIp `
  -Port $FrontendPort `
  -InformationLevel Detailed

# HTTP test.
Invoke-WebRequest `
  -Uri $TestUrl `
  -UseBasicParsing

# Repeat requests to observe backend response behavior.
1..10 | ForEach-Object {
  try {
    $Response = Invoke-WebRequest -Uri $TestUrl -UseBasicParsing
    [PSCustomObject]@{
      Attempt = $_
      StatusCode = $Response.StatusCode
      Content = ($Response.Content -replace "`r|`n"," ").Substring(0, [Math]::Min(120, $Response.Content.Length))
    }
  }
  catch {
    [PSCustomObject]@{
      Attempt = $_
      StatusCode = "FAILED"
      Content = $_.Exception.Message
    }
  }
  Start-Sleep -Seconds 2
}
```

```bash
# Linux or Cloud Shell traffic validation.

lbPublicIp="<lb-public-ip>"
frontendPort="<frontend-port>"

# TCP check.
nc -vz "$lbPublicIp" "$frontendPort"

# HTTP check.
curl -v "http://$lbPublicIp:$frontendPort"

# Repeat requests.
for i in {1..10}; do
  echo "Attempt $i"
  curl -s "http://$lbPublicIp:$frontendPort"
  echo
  sleep 2
done
```

# Configure_Azure_Load_Balancer_Bastion_And_Basic_Traffic_Distribution_Evidence_Export_Skeleton
```powershell
# Purpose: export evidence for Load Balancer, backend pool, public IPs, NSGs, Bastion, and subnet state.

$ResourceGroup = "<resource-group>"
$VNetName = "<vnet-name>"

$LoadBalancerName = "<load-balancer-name>"
$LbPublicIpName = "<lb-public-ip-name>"
$BastionName = "<bastion-name>"
$BastionPublicIpName = "<bastion-public-ip-name>"
$WorkloadNsgName = "<workload-nsg-name>"

$BackendNic1 = "<backend-nic-1>"
$BackendNic2 = "<backend-nic-2>"

$EvidencePath = "C:\Azure-LB-Bastion"

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

az account show `
  --output json |
  Out-File "$EvidencePath\account-context.json" -Encoding utf8

az network public-ip show `
  --resource-group $ResourceGroup `
  --name $LbPublicIpName `
  --output json |
  Out-File "$EvidencePath\load-balancer-public-ip.json" -Encoding utf8

az network lb show `
  --resource-group $ResourceGroup `
  --name $LoadBalancerName `
  --output json |
  Out-File "$EvidencePath\load-balancer.json" -Encoding utf8

az network lb frontend-ip list `
  --resource-group $ResourceGroup `
  --lb-name $LoadBalancerName `
  --output json |
  Out-File "$EvidencePath\load-balancer-frontend-ip.json" -Encoding utf8

az network lb address-pool list `
  --resource-group $ResourceGroup `
  --lb-name $LoadBalancerName `
  --output json |
  Out-File "$EvidencePath\load-balancer-backend-pools.json" -Encoding utf8

az network lb probe list `
  --resource-group $ResourceGroup `
  --lb-name $LoadBalancerName `
  --output json |
  Out-File "$EvidencePath\load-balancer-probes.json" -Encoding utf8

az network lb rule list `
  --resource-group $ResourceGroup `
  --lb-name $LoadBalancerName `
  --output json |
  Out-File "$EvidencePath\load-balancer-rules.json" -Encoding utf8

az network nic show `
  --resource-group $ResourceGroup `
  --name $BackendNic1 `
  --output json |
  Out-File "$EvidencePath\backend-nic-1.json" -Encoding utf8

az network nic show `
  --resource-group $ResourceGroup `
  --name $BackendNic2 `
  --output json |
  Out-File "$EvidencePath\backend-nic-2.json" -Encoding utf8

az network nsg show `
  --resource-group $ResourceGroup `
  --name $WorkloadNsgName `
  --output json |
  Out-File "$EvidencePath\workload-nsg.json" -Encoding utf8

az network nsg rule list `
  --resource-group $ResourceGroup `
  --nsg-name $WorkloadNsgName `
  --output json |
  Out-File "$EvidencePath\workload-nsg-rules.json" -Encoding utf8

az network public-ip show `
  --resource-group $ResourceGroup `
  --name $BastionPublicIpName `
  --output json |
  Out-File "$EvidencePath\bastion-public-ip.json" -Encoding utf8

az network bastion show `
  --resource-group $ResourceGroup `
  --name $BastionName `
  --output json |
  Out-File "$EvidencePath\bastion.json" -Encoding utf8

az network vnet subnet show `
  --resource-group $ResourceGroup `
  --vnet-name $VNetName `
  --name AzureBastionSubnet `
  --output json |
  Out-File "$EvidencePath\azure-bastion-subnet.json" -Encoding utf8
```

# Configure_Azure_Load_Balancer_Bastion_And_Basic_Traffic_Distribution_Verification_Commands
```bash
# Context.
az account show -o table

# Public IPs.
az network public-ip show \
  --resource-group "<resource-group>" \
  --name "<lb-public-ip-name>" \
  --query "{Name:name,Sku:sku.name,IPAddress:ipAddress,Zones:zones}" \
  --output table

az network public-ip show \
  --resource-group "<resource-group>" \
  --name "<bastion-public-ip-name>" \
  --query "{Name:name,Sku:sku.name,IPAddress:ipAddress,Zones:zones}" \
  --output table

# Load Balancer.
az network lb show \
  --resource-group "<resource-group>" \
  --name "<load-balancer-name>" \
  --query "{Name:name,Sku:sku.name,Location:location,FrontendIpConfigs:frontendIPConfigurations[].name,BackendPools:backendAddressPools[].name}" \
  --output json

# Frontend IP.
az network lb frontend-ip list \
  --resource-group "<resource-group>" \
  --lb-name "<load-balancer-name>" \
  --output table

# Backend pool.
az network lb address-pool list \
  --resource-group "<resource-group>" \
  --lb-name "<load-balancer-name>" \
  --output table

az network lb address-pool show \
  --resource-group "<resource-group>" \
  --lb-name "<load-balancer-name>" \
  --name "<backend-pool-name>" \
  --output json

# Health probes.
az network lb probe list \
  --resource-group "<resource-group>" \
  --lb-name "<load-balancer-name>" \
  --output table

# Load-balancing rules.
az network lb rule list \
  --resource-group "<resource-group>" \
  --lb-name "<load-balancer-name>" \
  --output table

# Backend NIC pool references.
az network nic show \
  --resource-group "<resource-group>" \
  --name "<backend-nic-1>" \
  --query "ipConfigurations[].loadBalancerBackendAddressPools[].id" \
  --output table

az network nic show \
  --resource-group "<resource-group>" \
  --name "<backend-nic-2>" \
  --query "ipConfigurations[].loadBalancerBackendAddressPools[].id" \
  --output table

# Workload NSG rules.
az network nsg rule list \
  --resource-group "<resource-group>" \
  --nsg-name "<workload-nsg-name>" \
  --output table

# Bastion subnet.
az network vnet subnet show \
  --resource-group "<resource-group>" \
  --vnet-name "<vnet-name>" \
  --name AzureBastionSubnet \
  --query "{Name:name,Prefix:addressPrefix,RouteTable:routeTable.id,Delegations:delegations}" \
  --output json

# Bastion host.
az network bastion show \
  --resource-group "<resource-group>" \
  --name "<bastion-name>" \
  --output table
```

```powershell
# Client-side traffic validation.

Test-NetConnection `
  -ComputerName "<lb-public-ip>" `
  -Port <frontend-port> `
  -InformationLevel Detailed

Invoke-WebRequest `
  -Uri "http://<lb-public-ip>:<frontend-port>" `
  -UseBasicParsing

# Backend VM validation from Bastion session.

hostname
ipconfig /all

Test-NetConnection `
  -ComputerName localhost `
  -Port <backend-port> `
  -InformationLevel Detailed
```

# Configure_Azure_Load_Balancer_Bastion_And_Basic_Traffic_Distribution_Rollback
```bash
# Remove backend NIC 1 from backend pool.
az network nic ip-config address-pool remove \
  --resource-group "<resource-group>" \
  --nic-name "<backend-nic-1>" \
  --ip-config-name "<ipconfig-name>" \
  --lb-name "<load-balancer-name>" \
  --address-pool "<backend-pool-name>"

# Remove backend NIC 2 from backend pool.
az network nic ip-config address-pool remove \
  --resource-group "<resource-group>" \
  --nic-name "<backend-nic-2>" \
  --ip-config-name "<ipconfig-name>" \
  --lb-name "<load-balancer-name>" \
  --address-pool "<backend-pool-name>"

# Delete load-balancing rule.
az network lb rule delete \
  --resource-group "<resource-group>" \
  --lb-name "<load-balancer-name>" \
  --name "<lb-rule-name>"

# Delete health probe.
az network lb probe delete \
  --resource-group "<resource-group>" \
  --lb-name "<load-balancer-name>" \
  --name "<probe-name>"

# Delete Load Balancer.
az network lb delete \
  --resource-group "<resource-group>" \
  --name "<load-balancer-name>"

# Delete Load Balancer public IP.
az network public-ip delete \
  --resource-group "<resource-group>" \
  --name "<lb-public-ip-name>"

# Delete Bastion host.
# Bastion has hourly cost, so delete it if this is a short lab.
az network bastion delete \
  --resource-group "<resource-group>" \
  --name "<bastion-name>"

# Delete Bastion public IP.
az network public-ip delete \
  --resource-group "<resource-group>" \
  --name "<bastion-public-ip-name>"

# Delete AzureBastionSubnet if lab-only.
az network vnet subnet delete \
  --resource-group "<resource-group>" \
  --vnet-name "<vnet-name>" \
  --name AzureBastionSubnet

# Delete NSG rules created by this workbook.
az network nsg rule delete \
  --resource-group "<resource-group>" \
  --nsg-name "<workload-nsg-name>" \
  --name Allow-LB-Backend-Port

az network nsg rule delete \
  --resource-group "<resource-group>" \
  --nsg-name "<workload-nsg-name>" \
  --name Allow-AzureLoadBalancer-Probe
```

```powershell
# PowerShell rollback skeleton.

$ResourceGroup = "<resource-group>"
$LoadBalancerName = "<load-balancer-name>"
$LbPublicIpName = "<lb-public-ip-name>"
$BastionName = "<bastion-name>"
$BastionPublicIpName = "<bastion-public-ip-name>"

Remove-AzLoadBalancer `
  -ResourceGroupName $ResourceGroup `
  -Name $LoadBalancerName `
  -Force

Remove-AzPublicIpAddress `
  -ResourceGroupName $ResourceGroup `
  -Name $LbPublicIpName `
  -Force

Remove-AzBastion `
  -ResourceGroupName $ResourceGroup `
  -Name $BastionName `
  -Force

Remove-AzPublicIpAddress `
  -ResourceGroupName $ResourceGroup `
  -Name $BastionPublicIpName `
  -Force
```

# Configure_Azure_Load_Balancer_Bastion_And_Basic_Traffic_Distribution_Failure_Checks
| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| Load Balancer creation fails | Basic SKU or mismatched resource type was attempted | Check public IP SKU and LB SKU | Use Standard public IP and Standard Load Balancer |
| Public IP creation fails with zone error | Region does not support requested zones | Retry without zones or with supported zone | Create supported Standard public IP |
| Frontend IP has no address | Public IP provisioning incomplete | `az network public-ip show` | Wait and verify provisioning state |
| Backend pool is empty | NICs were not associated | Show backend pool and NIC IP configs | Add NIC IP configs to backend pool |
| Backend VM never receives traffic | Health probe fails | Check probe port locally on backend VM | Start service or fix guest firewall |
| Health probe succeeds locally but LB still fails | NSG blocks probe or backend port | Effective NSG on backend NIC | Add required NSG allow rule |
| Frontend TCP test fails | NSG blocks backend port | Check NSG rules and effective NSGs | Allow backend service port |
| HTTP returns only one backend | Load balancing is flow based, not round robin per request | Test from different clients or clear connections | Validate with multiple flows |
| HTTP fails but TCP succeeds | Application service issue | Check web server logs and local response | Fix web service configuration |
| Probe port differs from backend port | Probe checks different service than data port | Review probe and rule config | Align probe with intended health signal |
| Outbound access breaks after adding Standard LB | Default outbound access changed | Check outbound rules, NAT Gateway, or public IP | Add NAT Gateway or outbound rule design |
| Backend VM cannot be reached through Bastion | Bastion not deployed or wrong VNet | Check Bastion resource and VNet | Deploy Bastion into correct VNet |
| Bastion deployment fails | Missing required subnet name | Check subnet name | Create subnet named exactly `AzureBastionSubnet` |
| Bastion deployment fails | Bastion subnet is too small | Check subnet prefix | Use `/26` or larger |
| Bastion deployment fails | Bastion public IP is not Standard | Check public IP SKU | Recreate public IP as Standard |
| Bastion deployment fails | Bastion public IP is in wrong region | Check public IP location | Create public IP in same region as Bastion |
| Bastion connection opens but login fails | Wrong VM credentials or identity | Check authentication method | Use correct local, domain, or Entra login path |
| Bastion connection to Linux fails | SSH service not running or blocked | Check port 22 on VM | Start SSH and allow guest firewall |
| Bastion connection to Windows fails | RDP not enabled or blocked | Check port 3389 on VM | Enable RDP and allow guest firewall |
| VM still has public IP | Public IP was not removed from NIC | Check NIC IP configuration | Disassociate public IP if Bastion is the intended management path |
| Load Balancer and Bastion both fail | Underlying subnet, NSG, or route issue | Check subnet NSG and route table | Fix NSG or UDR before retesting |
| Portal metrics show no data | No traffic has crossed resource yet | Generate test traffic | Retest after traffic generation |

# Configure_Azure_Load_Balancer_Bastion_And_Basic_Traffic_Distribution_Related_Labs
| Lab | Relationship |
|---|---|
| `01_Create_VNets_Subnets_Public_IPs_And_Address_Space_Baseline.md` | Provides VNet, subnet, and public IP baseline |
| `02_Configure_VNet_Peering_DNS_Resolution_And_Private_DNS_Zones.md` | Allows Bastion and traffic paths across peered networks when designed |
| `03_Configure_NSGs_ASGs_Effective_Security_Rules_And_Service_Endpoints.md` | Provides NSG rules required for backend traffic and management paths |
| `04_Configure_Private_Endpoints_Private_Link_And_PaaS_Network_Access.md` | Separates private PaaS access from public traffic distribution |
| `05_Configure_UDRs_Route_Tables_Azure_DNS_And_Name_Resolution.md` | Provides route and DNS checks that affect backend reachability |
| `07_Troubleshoot_Azure_Networking_With_Network_Watcher_Connection_Monitor_And_Effective_Routes.md` | Uses Network Watcher to prove route, NSG, and connection behavior |
| `08_Troubleshoot_Private_Endpoint_DNS_Routing_NSG_And_Load_Balancer_Issues.md` | Troubleshoots combined private endpoint, DNS, NSG, route, and load balancer failures |