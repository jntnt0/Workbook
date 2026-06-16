05_Configure_UDRs_Route_Tables_Azure_DNS_And_Name_Resolution.md

# Configure_UDRs_Route_Tables_Azure_DNS_And_Name_Resolution

# Configure_UDRs_Route_Tables_Azure_DNS_And_Name_Resolution_Index
05_Configure_UDRs_Route_Tables_Azure_DNS_And_Name_Resolution.md
Configure_UDRs_Route_Tables_Azure_DNS_And_Name_Resolution
Configure_UDRs_Route_Tables_Azure_DNS_And_Name_Resolution_Source_Basis
Configure_UDRs_Route_Tables_Azure_DNS_And_Name_Resolution_Mental_Model
Configure_UDRs_Route_Tables_Azure_DNS_And_Name_Resolution_Planning_Table
Configure_UDRs_Route_Tables_Azure_DNS_And_Name_Resolution_Configuration_Checklist
Configure_UDRs_Route_Tables_Azure_DNS_And_Name_Resolution_Azure_CLI_Route_Table_Skeleton
Configure_UDRs_Route_Tables_Azure_DNS_And_Name_Resolution_PowerShell_Route_Table_Skeleton
Configure_UDRs_Route_Tables_Azure_DNS_And_Name_Resolution_Custom_DNS_Skeleton
Configure_UDRs_Route_Tables_Azure_DNS_And_Name_Resolution_Private_DNS_Skeleton
Configure_UDRs_Route_Tables_Azure_DNS_And_Name_Resolution_DNS_Private_Resolver_Skeleton
Configure_UDRs_Route_Tables_Azure_DNS_And_Name_Resolution_Effective_Route_Validation_Skeleton
Configure_UDRs_Route_Tables_Azure_DNS_And_Name_Resolution_DNS_Validation_Skeleton
Configure_UDRs_Route_Tables_Azure_DNS_And_Name_Resolution_Evidence_Export_Skeleton
Configure_UDRs_Route_Tables_Azure_DNS_And_Name_Resolution_Verification_Commands
Configure_UDRs_Route_Tables_Azure_DNS_And_Name_Resolution_Rollback
Configure_UDRs_Route_Tables_Azure_DNS_And_Name_Resolution_Failure_Checks
Configure_UDRs_Route_Tables_Azure_DNS_And_Name_Resolution_Related_Labs

# Configure_UDRs_Route_Tables_Azure_DNS_And_Name_Resolution_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Azure virtual network traffic routing | System routes, UDRs, BGP routes, service endpoint routes, next hop types, route priority, and longest prefix match |
| Microsoft Learn | Create, change, or delete an Azure route table | Creating route tables, adding routes, associating route tables to subnets, and removing route tables |
| Microsoft Learn | Configure DNS name resolution for Azure virtual networks | Azure-provided DNS, Private DNS zones, customer-managed DNS servers, and DNS Private Resolver selection |
| Microsoft Learn | Azure DNS Private Resolver overview | Inbound endpoints, outbound endpoints, forwarding rulesets, virtual network links, and hybrid DNS forwarding |
| Microsoft Learn | Azure Private DNS zones | Private DNS zones, records, VNet links, registration links, and resolution links |
| Azure operational practice | Hub-spoke route control | Route table association per subnet and UDR-based traffic steering |
| Azure operational practice | Forced tunneling caution | Handling `0.0.0.0/0` carefully to avoid breaking platform, gateway, or internet paths |
| Azure operational practice | DNS validation before application testing | Confirming name resolution before blaming routing, NSGs, private endpoints, or load balancers |

# Configure_UDRs_Route_Tables_Azure_DNS_And_Name_Resolution_Mental_Model
| Concept | Operational Meaning |
|---|---|
| System route | Route Azure creates automatically for VNet, internet, peering, gateway, and service endpoint behavior |
| User-defined route | Static custom route created in a route table |
| Route table | Azure resource that contains UDRs and is associated to one or more subnets |
| Subnet association | Route table attachment point. Route tables attach to subnets, not directly to VNets |
| Longest prefix match | Most specific matching route wins, such as `/24` before `/16` |
| Route source priority | For same prefix matches, Azure prefers UDR, then BGP, then system route |
| Next hop type | Where Azure sends matching traffic next |
| Virtual appliance next hop | Sends traffic to a firewall, NVA, or appliance private IP |
| Virtual network gateway next hop | Sends traffic to a VPN gateway path when valid |
| Internet next hop | Sends traffic toward Azure internet egress handling |
| None next hop | Drops traffic for the matching prefix |
| VNetLocal route | System route for address spaces inside the local VNet |
| VNet peering route | System route added for peered VNet prefixes |
| Service endpoint route | System route added when service endpoints are enabled |
| Gateway route propagation | Routes learned from VPN or ExpressRoute gateway into subnet effective routes |
| Effective routes | Final route set evaluated for a NIC |
| Azure-provided DNS | Default DNS service at `168.63.129.16` scoped to the VNet |
| Custom DNS server | DNS server IP configured on VNet or NIC |
| Private DNS zone | Azure-hosted private namespace linked to VNets |
| DNS Private Resolver | Managed Azure DNS forwarding service for hybrid and cross-network resolution |
| Inbound resolver endpoint | Receives DNS queries from on-premises or other connected networks |
| Outbound resolver endpoint | Sends DNS queries to target DNS servers according to forwarding rules |
| Forwarding ruleset | DNS suffix routing policy linked to VNets |
| First rule | Route tables control packet path, DNS controls destination lookup |
| Blunt rule | A good route to the wrong IP is still broken, and good DNS with a bad route is still broken |

# Configure_UDRs_Route_Tables_Azure_DNS_And_Name_Resolution_Planning_Table
| Item | Example | Decision |
|---|---|---|
| Subscription | `Visual Studio Enterprise` | `<subscription-name>` |
| Resource group | `rg-net-core-lab-eus2-01` | `<resource-group>` |
| DNS resource group | `rg-dns-private-eus2-01` | `<dns-resource-group>` |
| Region | `eastus2` | `<region>` |
| VNet name | `vnet-core-lab-eus2-01` | `<vnet-name>` |
| Workload subnet | `snet-workload-01` | `<workload-subnet-name>` |
| Management subnet | `snet-management-01` | `<management-subnet-name>` |
| NVA subnet | `snet-nva-01` | `<nva-subnet-name>` |
| DNS resolver inbound subnet | `snet-dnspr-inbound-01` | `<dnspr-inbound-subnet-name>` |
| DNS resolver outbound subnet | `snet-dnspr-outbound-01` | `<dnspr-outbound-subnet-name>` |
| Route table name | `rt-workload-eus2-01` | `<route-table-name>` |
| Route name | `route-to-shared-services` | `<route-name>` |
| Destination prefix | `10.60.0.0/16` | `<destination-prefix>` |
| Next hop type | `VirtualAppliance` | `<next-hop-type>` |
| Next hop IP | `10.40.100.4` | `<next-hop-ip>` |
| Disable gateway route propagation | `false` baseline | `<true-false>` |
| Test NIC | `nic-vm-workload-01` | `<test-nic-name>` |
| Test destination IP | `10.60.10.10` | `<test-destination-ip>` |
| Azure DNS IP | `168.63.129.16` | Fixed platform value |
| Custom DNS server 1 | `10.40.20.10` | `<custom-dns-ip-1>` |
| Custom DNS server 2 | `10.40.20.11` | `<custom-dns-ip-2>` |
| Private DNS zone | `corp.internal` | `<private-zone-name>` |
| Private DNS record name | `app01` | `<private-record-name>` |
| Private DNS record IP | `10.60.10.10` | `<private-record-ip>` |
| DNS Private Resolver name | `dnspr-core-eus2-01` | `<dns-private-resolver-name>` |
| Inbound endpoint name | `in-dnspr-core-eus2-01` | `<inbound-endpoint-name>` |
| Outbound endpoint name | `out-dnspr-core-eus2-01` | `<outbound-endpoint-name>` |
| Forwarding ruleset name | `frs-core-eus2-01` | `<forwarding-ruleset-name>` |
| Forwarding rule name | `rule-onprem-contoso` | `<forwarding-rule-name>` |
| Forwarded domain | `onprem.contoso.com.` | `<forwarded-domain>` |
| Target DNS server | `192.168.10.10` | `<target-dns-ip>` |
| Evidence path | `C:\Azure-UDR-DNS` | `<evidence-path>` |
| Rollback stance | Remove subnet route table association, delete routes, restore DNS settings, delete resolver pieces if lab-only | `<rollback-plan>` |

# Configure_UDRs_Route_Tables_Azure_DNS_And_Name_Resolution_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm Azure login | Admin Workstation / Cloud Shell | `az account show` or `Get-AzContext` | Active Azure session is visible |
| 2 | Select target subscription | Admin Workstation / Cloud Shell | `az account set --subscription "<subscription-name>"` or `Set-AzContext -Subscription "<subscription-name>"` | Correct subscription is active |
| 3 | Confirm VNet exists | Admin Workstation / Cloud Shell | `az network vnet show -g "<resource-group>" -n "<vnet-name>"` | VNet exists |
| 4 | Confirm target subnets exist | Admin Workstation / Cloud Shell | `az network vnet subnet list -g "<resource-group>" --vnet-name "<vnet-name>" -o table` | Workload, management, NVA, and DNS subnets are known |
| 5 | Confirm current route table associations | Admin Workstation / Cloud Shell | `az network vnet subnet list -g "<resource-group>" --vnet-name "<vnet-name>" --query "[].{Name:name,RouteTable:routeTable.id}" -o table` | Existing subnet route table state is known |
| 6 | Confirm current DNS settings on VNet | Admin Workstation / Cloud Shell | `az network vnet show -g "<resource-group>" -n "<vnet-name>" --query "dhcpOptions.dnsServers" -o json` | Current VNet DNS setting is known |
| 7 | Create route table | Admin Workstation / Cloud Shell | `az network route-table create -g "<resource-group>" -n "<route-table-name>" -l "<region>"` | Route table exists |
| 8 | Decide gateway route propagation | Operator | Set `<true-false>` for gateway route propagation | Propagation choice is intentional |
| 9 | Disable gateway propagation if required | Admin Workstation / Cloud Shell | `az network route-table update -g "<resource-group>" -n "<route-table-name>" --disable-bgp-route-propagation true` | Gateway propagated routes are not added to associated subnets |
| 10 | Create UDR to virtual appliance | Admin Workstation / Cloud Shell | `az network route-table route create -g "<resource-group>" --route-table-name "<route-table-name>" -n "<route-name>" --address-prefix "<destination-prefix>" --next-hop-type VirtualAppliance --next-hop-ip-address "<next-hop-ip>"` | Route exists in table |
| 11 | Create optional default route through appliance | Admin Workstation / Cloud Shell | `az network route-table route create -g "<resource-group>" --route-table-name "<route-table-name>" -n "default-to-nva" --address-prefix "0.0.0.0/0" --next-hop-type VirtualAppliance --next-hop-ip-address "<next-hop-ip>"` | Forced tunnel route exists only if intended |
| 12 | Create optional blackhole route | Admin Workstation / Cloud Shell | `az network route-table route create -g "<resource-group>" --route-table-name "<route-table-name>" -n "deny-unapproved-prefix" --address-prefix "<blocked-prefix>" --next-hop-type None` | Traffic to prefix is dropped |
| 13 | Associate route table to workload subnet | Admin Workstation / Cloud Shell | `az network vnet subnet update -g "<resource-group>" --vnet-name "<vnet-name>" -n "<workload-subnet-name>" --route-table "<route-table-name>"` | Workload subnet uses route table |
| 14 | Confirm route table association | Admin Workstation / Cloud Shell | `az network vnet subnet show -g "<resource-group>" --vnet-name "<vnet-name>" -n "<workload-subnet-name>" --query "routeTable.id" -o tsv` | Route table ID is attached |
| 15 | Confirm route entries | Admin Workstation / Cloud Shell | `az network route-table route list -g "<resource-group>" --route-table-name "<route-table-name>" -o table` | Expected routes are listed |
| 16 | Confirm NVA NIC IP forwarding if using virtual appliance | Admin Workstation / Cloud Shell | `az network nic show -g "<resource-group>" -n "<nva-nic-name>" --query "enableIPForwarding" -o tsv` | IP forwarding is `true` if NVA forwarding is required |
| 17 | Enable NVA NIC IP forwarding if required | Admin Workstation / Cloud Shell | `az network nic update -g "<resource-group>" -n "<nva-nic-name>" --ip-forwarding true` | Azure NIC allows forwarded traffic |
| 18 | Confirm OS or appliance routing is enabled | NVA | Vendor or OS-specific command | NVA forwards packets if it is part of the design |
| 19 | Check effective routes on test NIC | Admin Workstation / Cloud Shell | `az network nic show-effective-route-table -g "<resource-group>" -n "<test-nic-name>" -o table` | UDR appears in effective route table |
| 20 | Test next hop from Network Watcher if available | Admin Workstation / Cloud Shell | `az network watcher show-next-hop -g "<resource-group>" --vm "<test-vm-name>" --source-ip "<source-ip>" --dest-ip "<test-destination-ip>"` | Next hop matches expected route |
| 21 | Create or verify Private DNS zone | Admin Workstation / Cloud Shell | `az network private-dns zone create -g "<dns-resource-group>" -n "<private-zone-name>"` | Private DNS zone exists |
| 22 | Link VNet to Private DNS zone | Admin Workstation / Cloud Shell | `az network private-dns link vnet create -g "<dns-resource-group>" -z "<private-zone-name>" -n "<private-dns-link-name>" -v "<vnet-id>" -e false` | VNet can resolve records in zone |
| 23 | Add private A record | Admin Workstation / Cloud Shell | `az network private-dns record-set a add-record -g "<dns-resource-group>" -z "<private-zone-name>" -n "<private-record-name>" -a "<private-record-ip>"` | Private record exists |
| 24 | Validate private DNS from workload VM | Workload VM | `Resolve-DnsName "<private-record-name>.<private-zone-name>"` | Name resolves to expected private IP |
| 25 | Configure VNet custom DNS only if required | Admin Workstation / Cloud Shell | `az network vnet update -g "<resource-group>" -n "<vnet-name>" --dns-servers "<custom-dns-ip-1>" "<custom-dns-ip-2>"` | VNet DHCP DNS server list is updated |
| 26 | Restart VM or renew DNS client after DNS change | Workload VM | `ipconfig /renew` or VM restart | VM receives updated DNS settings |
| 27 | Deploy DNS Private Resolver only if hybrid or forwarding is required | Admin Workstation / Cloud Shell | Run DNS Private Resolver skeleton | Resolver, endpoints, and ruleset exist |
| 28 | Validate forwarded DNS name | Workload VM | `Resolve-DnsName "<host>.<forwarded-domain>"` | Query returns answer from forwarded DNS path |
| 29 | Validate internet and Azure service DNS | Workload VM | `Resolve-DnsName login.microsoftonline.com` and service FQDN tests | Public and Azure service DNS still works |
| 30 | Export evidence | Admin Workstation / Cloud Shell | Run evidence export skeleton | Route, DNS, and effective route evidence is saved |
| 31 | Document final state | Operator | Record route tables, UDRs, subnet associations, DNS settings, zone links, and validation results | Workbook is complete |

# Configure_UDRs_Route_Tables_Azure_DNS_And_Name_Resolution_Azure_CLI_Route_Table_Skeleton
```bash
# Purpose: create route table, add UDRs, and associate route table to a subnet.
# Run from Azure Cloud Shell Bash or local Azure CLI.

subscriptionName="<subscription-name>"
resourceGroup="<resource-group>"
location="<region>"

vnetName="<vnet-name>"
workloadSubnetName="<workload-subnet-name>"

routeTableName="<route-table-name>"
routeName="<route-name>"
destinationPrefix="<destination-prefix>"
nextHopIp="<next-hop-ip>"

az account set \
  --subscription "$subscriptionName"

# Create route table.
az network route-table create \
  --resource-group "$resourceGroup" \
  --name "$routeTableName" \
  --location "$location"

# Optional: disable gateway route propagation.
# Do this only when you intentionally do not want VPN or ExpressRoute propagated routes on this subnet.
# Do not disable gateway propagation on GatewaySubnet.
az network route-table update \
  --resource-group "$resourceGroup" \
  --name "$routeTableName" \
  --disable-bgp-route-propagation false

# Create UDR toward a virtual appliance.
az network route-table route create \
  --resource-group "$resourceGroup" \
  --route-table-name "$routeTableName" \
  --name "$routeName" \
  --address-prefix "$destinationPrefix" \
  --next-hop-type VirtualAppliance \
  --next-hop-ip-address "$nextHopIp"

# Optional: default route through NVA.
# Use carefully. This changes internet and public Azure service egress path for associated subnet.
# az network route-table route create \
#   --resource-group "$resourceGroup" \
#   --route-table-name "$routeTableName" \
#   --name default-to-nva \
#   --address-prefix 0.0.0.0/0 \
#   --next-hop-type VirtualAppliance \
#   --next-hop-ip-address "$nextHopIp"

# Optional: drop route.
# az network route-table route create \
#   --resource-group "$resourceGroup" \
#   --route-table-name "$routeTableName" \
#   --name deny-unapproved-prefix \
#   --address-prefix "<blocked-prefix>" \
#   --next-hop-type None

# Associate route table to subnet.
az network vnet subnet update \
  --resource-group "$resourceGroup" \
  --vnet-name "$vnetName" \
  --name "$workloadSubnetName" \
  --route-table "$routeTableName"

# Verify route table.
az network route-table show \
  --resource-group "$resourceGroup" \
  --name "$routeTableName" \
  --output table

az network route-table route list \
  --resource-group "$resourceGroup" \
  --route-table-name "$routeTableName" \
  --output table

# Verify subnet association.
az network vnet subnet show \
  --resource-group "$resourceGroup" \
  --vnet-name "$vnetName" \
  --name "$workloadSubnetName" \
  --query "{Name:name,RouteTable:routeTable.id}" \
  --output table
```

# Configure_UDRs_Route_Tables_Azure_DNS_And_Name_Resolution_PowerShell_Route_Table_Skeleton
```powershell
# Purpose: create route table, add UDRs, and associate route table to a subnet.
# Run from Azure Cloud Shell PowerShell or local PowerShell with Az.Network.

$SubscriptionName = "<subscription-name>"
$ResourceGroup = "<resource-group>"
$Location = "<region>"

$VNetName = "<vnet-name>"
$WorkloadSubnetName = "<workload-subnet-name>"

$RouteTableName = "<route-table-name>"
$RouteName = "<route-name>"
$DestinationPrefix = "<destination-prefix>"
$NextHopIp = "<next-hop-ip>"

Set-AzContext -Subscription $SubscriptionName

# Create route config.
$RouteConfig = New-AzRouteConfig `
  -Name $RouteName `
  -AddressPrefix $DestinationPrefix `
  -NextHopType VirtualAppliance `
  -NextHopIpAddress $NextHopIp

# Create route table.
$RouteTable = New-AzRouteTable `
  -ResourceGroupName $ResourceGroup `
  -Location $Location `
  -Name $RouteTableName `
  -Route $RouteConfig

# Optional: disable gateway route propagation if required.
# Do not apply this setting to GatewaySubnet route table use cases.
# $RouteTable.DisableBgpRoutePropagation = $true
# $RouteTable | Set-AzRouteTable

# Associate route table to subnet.
$VNet = Get-AzVirtualNetwork `
  -ResourceGroupName $ResourceGroup `
  -Name $VNetName

$Subnet = $VNet.Subnets | Where-Object Name -eq $WorkloadSubnetName

Set-AzVirtualNetworkSubnetConfig `
  -VirtualNetwork $VNet `
  -Name $WorkloadSubnetName `
  -AddressPrefix $Subnet.AddressPrefix `
  -RouteTable $RouteTable

$VNet | Set-AzVirtualNetwork

# Verify.
Get-AzRouteTable `
  -ResourceGroupName $ResourceGroup `
  -Name $RouteTableName

(Get-AzVirtualNetwork `
  -ResourceGroupName $ResourceGroup `
  -Name $VNetName).Subnets |
  Select-Object Name, AddressPrefix, RouteTable
```

# Configure_UDRs_Route_Tables_Azure_DNS_And_Name_Resolution_Custom_DNS_Skeleton
```bash
# Purpose: configure custom DNS servers on a VNet.
# Use only when you have a real DNS server or DNS forwarding design.
# VMs must renew DHCP or restart to pick up changed DNS settings.

subscriptionName="<subscription-name>"
resourceGroup="<resource-group>"
vnetName="<vnet-name>"

customDnsIp1="<custom-dns-ip-1>"
customDnsIp2="<custom-dns-ip-2>"

az account set \
  --subscription "$subscriptionName"

# Show current DNS settings.
az network vnet show \
  --resource-group "$resourceGroup" \
  --name "$vnetName" \
  --query "dhcpOptions.dnsServers" \
  --output json

# Set custom DNS servers.
az network vnet update \
  --resource-group "$resourceGroup" \
  --name "$vnetName" \
  --dns-servers "$customDnsIp1" "$customDnsIp2"

# Verify custom DNS settings.
az network vnet show \
  --resource-group "$resourceGroup" \
  --name "$vnetName" \
  --query "dhcpOptions.dnsServers" \
  --output table

# Reset to Azure-provided DNS.
# Use only for rollback or when no custom DNS is required.
# az network vnet update \
#   --resource-group "$resourceGroup" \
#   --name "$vnetName" \
#   --dns-servers ""
```

```powershell
# Run inside a Windows VM after VNet DNS settings change.
# Restart is safest, but renew can work depending on adapter state.

ipconfig /all
ipconfig /renew
ipconfig /flushdns
ipconfig /displaydns

Resolve-DnsName "login.microsoftonline.com"
Resolve-DnsName "<private-record-name>.<private-zone-name>"
```

# Configure_UDRs_Route_Tables_Azure_DNS_And_Name_Resolution_Private_DNS_Skeleton
```bash
# Purpose: create private DNS zone, link it to a VNet, and add a test A record.

subscriptionName="<subscription-name>"

resourceGroup="<resource-group>"
dnsResourceGroup="<dns-resource-group>"
location="<region>"

vnetName="<vnet-name>"
privateZoneName="<private-zone-name>"
privateDnsLinkName="<private-dns-link-name>"

privateRecordName="<private-record-name>"
privateRecordIp="<private-record-ip>"

az account set \
  --subscription "$subscriptionName"

# Create DNS resource group if required.
az group create \
  --name "$dnsResourceGroup" \
  --location "$location"

# Capture VNet ID.
vnetId=$(az network vnet show \
  --resource-group "$resourceGroup" \
  --name "$vnetName" \
  --query id \
  --output tsv)

# Create private DNS zone.
az network private-dns zone create \
  --resource-group "$dnsResourceGroup" \
  --name "$privateZoneName"

# Link VNet for resolution.
az network private-dns link vnet create \
  --resource-group "$dnsResourceGroup" \
  --zone-name "$privateZoneName" \
  --name "$privateDnsLinkName" \
  --virtual-network "$vnetId" \
  --registration-enabled false

# Add A record.
az network private-dns record-set a add-record \
  --resource-group "$dnsResourceGroup" \
  --zone-name "$privateZoneName" \
  --record-set-name "$privateRecordName" \
  --ipv4-address "$privateRecordIp"

# Verify zone, link, and records.
az network private-dns zone show \
  --resource-group "$dnsResourceGroup" \
  --name "$privateZoneName" \
  --output table

az network private-dns link vnet list \
  --resource-group "$dnsResourceGroup" \
  --zone-name "$privateZoneName" \
  --output table

az network private-dns record-set a list \
  --resource-group "$dnsResourceGroup" \
  --zone-name "$privateZoneName" \
  --output table
```

# Configure_UDRs_Route_Tables_Azure_DNS_And_Name_Resolution_DNS_Private_Resolver_Skeleton
```bash
# Purpose: create Azure DNS Private Resolver for hybrid or cross-network forwarding.
# This is only needed when Private DNS zones, Azure-provided DNS, or simple custom DNS are not enough.
# Verify exact CLI help with:
# az dns-resolver -h

subscriptionName="<subscription-name>"
resourceGroup="<resource-group>"
location="<region>"

vnetName="<vnet-name>"
dnsPrivateResolverName="<dns-private-resolver-name>"

inboundSubnetName="<dnspr-inbound-subnet-name>"
outboundSubnetName="<dnspr-outbound-subnet-name>"

inboundEndpointName="<inbound-endpoint-name>"
outboundEndpointName="<outbound-endpoint-name>"

forwardingRulesetName="<forwarding-ruleset-name>"
forwardingRuleName="<forwarding-rule-name>"
forwardedDomain="<forwarded-domain>"
targetDnsIp="<target-dns-ip>"

az account set \
  --subscription "$subscriptionName"

# Capture VNet and subnet IDs.
vnetId=$(az network vnet show \
  --resource-group "$resourceGroup" \
  --name "$vnetName" \
  --query id \
  --output tsv)

inboundSubnetId=$(az network vnet subnet show \
  --resource-group "$resourceGroup" \
  --vnet-name "$vnetName" \
  --name "$inboundSubnetName" \
  --query id \
  --output tsv)

outboundSubnetId=$(az network vnet subnet show \
  --resource-group "$resourceGroup" \
  --vnet-name "$vnetName" \
  --name "$outboundSubnetName" \
  --query id \
  --output tsv)

# Delegate resolver subnets if not already delegated.
az network vnet subnet update \
  --resource-group "$resourceGroup" \
  --vnet-name "$vnetName" \
  --name "$inboundSubnetName" \
  --delegations Microsoft.Network/dnsResolvers

az network vnet subnet update \
  --resource-group "$resourceGroup" \
  --vnet-name "$vnetName" \
  --name "$outboundSubnetName" \
  --delegations Microsoft.Network/dnsResolvers

# Create DNS Private Resolver.
az dns-resolver create \
  --resource-group "$resourceGroup" \
  --name "$dnsPrivateResolverName" \
  --location "$location" \
  --virtual-network "$vnetId"

# Create inbound endpoint.
az dns-resolver inbound-endpoint create \
  --resource-group "$resourceGroup" \
  --dns-resolver-name "$dnsPrivateResolverName" \
  --name "$inboundEndpointName" \
  --location "$location" \
  --ip-configurations "[{\"privateIpAllocationMethod\":\"Dynamic\",\"subnet\":{\"id\":\"$inboundSubnetId\"}}]"

# Create outbound endpoint.
az dns-resolver outbound-endpoint create \
  --resource-group "$resourceGroup" \
  --dns-resolver-name "$dnsPrivateResolverName" \
  --name "$outboundEndpointName" \
  --location "$location" \
  --subnet "$outboundSubnetId"

# Capture outbound endpoint ID.
outboundEndpointId=$(az dns-resolver outbound-endpoint show \
  --resource-group "$resourceGroup" \
  --dns-resolver-name "$dnsPrivateResolverName" \
  --name "$outboundEndpointName" \
  --query id \
  --output tsv)

# Create forwarding ruleset.
az dns-resolver forwarding-ruleset create \
  --resource-group "$resourceGroup" \
  --name "$forwardingRulesetName" \
  --location "$location" \
  --outbound-endpoints "[{\"id\":\"$outboundEndpointId\"}]"

# Create forwarding rule.
az dns-resolver forwarding-ruleset rule create \
  --resource-group "$resourceGroup" \
  --ruleset-name "$forwardingRulesetName" \
  --name "$forwardingRuleName" \
  --domain-name "$forwardedDomain" \
  --forwarding-rule-state Enabled \
  --target-dns-servers "[{\"ipAddress\":\"$targetDnsIp\",\"port\":53}]"

# Link forwarding ruleset to VNet.
az dns-resolver forwarding-ruleset virtual-network-link create \
  --resource-group "$resourceGroup" \
  --ruleset-name "$forwardingRulesetName" \
  --name "link-$vnetName" \
  --virtual-network "$vnetId"

# Verify resolver resources.
az dns-resolver show \
  --resource-group "$resourceGroup" \
  --name "$dnsPrivateResolverName" \
  --output table

az dns-resolver inbound-endpoint list \
  --resource-group "$resourceGroup" \
  --dns-resolver-name "$dnsPrivateResolverName" \
  --output table

az dns-resolver outbound-endpoint list \
  --resource-group "$resourceGroup" \
  --dns-resolver-name "$dnsPrivateResolverName" \
  --output table

az dns-resolver forwarding-ruleset rule list \
  --resource-group "$resourceGroup" \
  --ruleset-name "$forwardingRulesetName" \
  --output table
```

# Configure_UDRs_Route_Tables_Azure_DNS_And_Name_Resolution_Effective_Route_Validation_Skeleton
```bash
# Purpose: validate the effective route table on a NIC.
# This proves whether the subnet is actually using the intended UDR.

resourceGroup="<resource-group>"
testNicName="<test-nic-name>"

# Show effective routes.
az network nic show-effective-route-table \
  --resource-group "$resourceGroup" \
  --name "$testNicName" \
  --output table

# Export effective routes to JSON.
az network nic show-effective-route-table \
  --resource-group "$resourceGroup" \
  --name "$testNicName" \
  --output json > effective-routes-$testNicName.json

# Confirm effective NSG separately if route path still fails after routes look correct.
az network nic list-effective-nsg \
  --resource-group "$resourceGroup" \
  --name "$testNicName" \
  --output table
```

```powershell
# Inside Windows VM route and DNS client checks.

route print
ipconfig /all
Resolve-DnsName "<private-record-name>.<private-zone-name>"
Test-NetConnection "<test-destination-ip>" -InformationLevel Detailed
Test-NetConnection "<test-destination-fqdn>" -Port 443 -InformationLevel Detailed
```

# Configure_UDRs_Route_Tables_Azure_DNS_And_Name_Resolution_DNS_Validation_Skeleton
```powershell
# Purpose: validate Azure-provided DNS, custom DNS, Private DNS, and forwarded DNS behavior.
# Run inside workload VM.

$PrivateFqdn = "<private-record-name>.<private-zone-name>"
$ForwardedFqdn = "<host>.<forwarded-domain>"
$PublicFqdn = "login.microsoftonline.com"
$ExpectedPrivateIp = "<private-record-ip>"

# Show DNS client configuration.
ipconfig /all

# Flush local resolver cache.
ipconfig /flushdns

# Test public DNS resolution.
Resolve-DnsName $PublicFqdn

# Test private DNS zone resolution.
$PrivateResult = Resolve-DnsName $PrivateFqdn -Type A

$PrivateResult |
  Select-Object Name, Type, IPAddress

if ($PrivateResult.IPAddress -contains $ExpectedPrivateIp) {
  Write-Host "PASS: $PrivateFqdn resolves to $ExpectedPrivateIp"
}
else {
  Write-Host "FAIL: $PrivateFqdn did not resolve to $ExpectedPrivateIp"
}

# Test forwarded DNS suffix if DNS Private Resolver or custom forwarding is configured.
Resolve-DnsName $ForwardedFqdn

# Test network path to resolved private IP.
Test-NetConnection `
  -ComputerName $PrivateFqdn `
  -Port 443 `
  -InformationLevel Detailed
```

```bash
# Linux VM validation.

privateFqdn="<private-record-name>.<private-zone-name>"
forwardedFqdn="<host>.<forwarded-domain>"
publicFqdn="login.microsoftonline.com"

cat /etc/resolv.conf

dig "$publicFqdn"
dig "$privateFqdn"
dig "$forwardedFqdn"

nslookup "$privateFqdn"
nc -vz "$privateFqdn" 443
ip route
```

# Configure_UDRs_Route_Tables_Azure_DNS_And_Name_Resolution_Evidence_Export_Skeleton
```powershell
# Purpose: export route table, UDR, subnet association, VNet DNS, Private DNS, and effective route evidence.

$ResourceGroup = "<resource-group>"
$DnsResourceGroup = "<dns-resource-group>"
$VNetName = "<vnet-name>"
$RouteTableName = "<route-table-name>"
$WorkloadSubnetName = "<workload-subnet-name>"
$TestNicName = "<test-nic-name>"
$PrivateZoneName = "<private-zone-name>"
$DnsPrivateResolverName = "<dns-private-resolver-name>"
$ForwardingRulesetName = "<forwarding-ruleset-name>"

$EvidencePath = "C:\Azure-UDR-DNS"

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

az account show `
  --output json |
  Out-File "$EvidencePath\account-context.json" -Encoding utf8

az network vnet show `
  --resource-group $ResourceGroup `
  --name $VNetName `
  --output json |
  Out-File "$EvidencePath\vnet.json" -Encoding utf8

az network vnet show `
  --resource-group $ResourceGroup `
  --name $VNetName `
  --query "dhcpOptions.dnsServers" `
  --output json |
  Out-File "$EvidencePath\vnet-dns-settings.json" -Encoding utf8

az network vnet subnet show `
  --resource-group $ResourceGroup `
  --vnet-name $VNetName `
  --name $WorkloadSubnetName `
  --output json |
  Out-File "$EvidencePath\workload-subnet.json" -Encoding utf8

az network route-table show `
  --resource-group $ResourceGroup `
  --name $RouteTableName `
  --output json |
  Out-File "$EvidencePath\route-table.json" -Encoding utf8

az network route-table route list `
  --resource-group $ResourceGroup `
  --route-table-name $RouteTableName `
  --output json |
  Out-File "$EvidencePath\route-table-routes.json" -Encoding utf8

az network nic show-effective-route-table `
  --resource-group $ResourceGroup `
  --name $TestNicName `
  --output json |
  Out-File "$EvidencePath\effective-routes-$TestNicName.json" -Encoding utf8

az network nic list-effective-nsg `
  --resource-group $ResourceGroup `
  --name $TestNicName `
  --output json |
  Out-File "$EvidencePath\effective-nsg-$TestNicName.json" -Encoding utf8

az network private-dns zone show `
  --resource-group $DnsResourceGroup `
  --name $PrivateZoneName `
  --output json |
  Out-File "$EvidencePath\private-dns-zone.json" -Encoding utf8

az network private-dns link vnet list `
  --resource-group $DnsResourceGroup `
  --zone-name $PrivateZoneName `
  --output json |
  Out-File "$EvidencePath\private-dns-vnet-links.json" -Encoding utf8

az network private-dns record-set list `
  --resource-group $DnsResourceGroup `
  --zone-name $PrivateZoneName `
  --output json |
  Out-File "$EvidencePath\private-dns-records.json" -Encoding utf8

# Optional DNS Private Resolver evidence.
az dns-resolver show `
  --resource-group $ResourceGroup `
  --name $DnsPrivateResolverName `
  --output json |
  Out-File "$EvidencePath\dns-private-resolver.json" -Encoding utf8

az dns-resolver forwarding-ruleset rule list `
  --resource-group $ResourceGroup `
  --ruleset-name $ForwardingRulesetName `
  --output json |
  Out-File "$EvidencePath\dns-forwarding-rules.json" -Encoding utf8
```

# Configure_UDRs_Route_Tables_Azure_DNS_And_Name_Resolution_Verification_Commands
```bash
# Context.
az account show -o table

# VNet and subnet route table associations.
az network vnet subnet list \
  --resource-group "<resource-group>" \
  --vnet-name "<vnet-name>" \
  --query "[].{Name:name,Prefix:addressPrefix,RouteTable:routeTable.id,Nsg:networkSecurityGroup.id}" \
  --output table

# Route table inventory.
az network route-table list \
  --resource-group "<resource-group>" \
  --output table

az network route-table show \
  --resource-group "<resource-group>" \
  --name "<route-table-name>" \
  --query "{Name:name,DisableBgpRoutePropagation:disableBgpRoutePropagation,Routes:routes}" \
  --output json

# UDR list.
az network route-table route list \
  --resource-group "<resource-group>" \
  --route-table-name "<route-table-name>" \
  --output table

# Individual route.
az network route-table route show \
  --resource-group "<resource-group>" \
  --route-table-name "<route-table-name>" \
  --name "<route-name>" \
  --output table

# NIC effective routes.
az network nic show-effective-route-table \
  --resource-group "<resource-group>" \
  --name "<test-nic-name>" \
  --output table

# NVA NIC IP forwarding.
az network nic show \
  --resource-group "<resource-group>" \
  --name "<nva-nic-name>" \
  --query "{Name:name,EnableIPForwarding:enableIPForwarding,PrivateIp:ipConfigurations[0].privateIPAddress}" \
  --output table

# VNet DNS settings.
az network vnet show \
  --resource-group "<resource-group>" \
  --name "<vnet-name>" \
  --query "dhcpOptions.dnsServers" \
  --output json

# Private DNS.
az network private-dns zone list \
  --resource-group "<dns-resource-group>" \
  --output table

az network private-dns link vnet list \
  --resource-group "<dns-resource-group>" \
  --zone-name "<private-zone-name>" \
  --output table

az network private-dns record-set list \
  --resource-group "<dns-resource-group>" \
  --zone-name "<private-zone-name>" \
  --output table

# DNS Private Resolver if used.
az dns-resolver list \
  --resource-group "<resource-group>" \
  --output table

az dns-resolver forwarding-ruleset rule list \
  --resource-group "<resource-group>" \
  --ruleset-name "<forwarding-ruleset-name>" \
  --output table
```

```powershell
# Inside Windows VM.

ipconfig /all
route print
Resolve-DnsName "login.microsoftonline.com"
Resolve-DnsName "<private-record-name>.<private-zone-name>"
Resolve-DnsName "<host>.<forwarded-domain>"

Test-NetConnection "<test-destination-ip>" -InformationLevel Detailed
Test-NetConnection "<private-record-name>.<private-zone-name>" -Port 443 -InformationLevel Detailed
```

# Configure_UDRs_Route_Tables_Azure_DNS_And_Name_Resolution_Rollback
```bash
# Dissociate route table from subnet.
az network vnet subnet update \
  --resource-group "<resource-group>" \
  --vnet-name "<vnet-name>" \
  --name "<workload-subnet-name>" \
  --route-table ""

# Delete a route from route table.
az network route-table route delete \
  --resource-group "<resource-group>" \
  --route-table-name "<route-table-name>" \
  --name "<route-name>"

# Delete default route if created.
az network route-table route delete \
  --resource-group "<resource-group>" \
  --route-table-name "<route-table-name>" \
  --name "default-to-nva"

# Delete route table after all subnet associations are removed.
az network route-table delete \
  --resource-group "<resource-group>" \
  --name "<route-table-name>"

# Reset VNet DNS to Azure-provided DNS.
az network vnet update \
  --resource-group "<resource-group>" \
  --name "<vnet-name>" \
  --dns-servers ""

# Remove Private DNS record.
az network private-dns record-set a remove-record \
  --resource-group "<dns-resource-group>" \
  --zone-name "<private-zone-name>" \
  --record-set-name "<private-record-name>" \
  --ipv4-address "<private-record-ip>"

# Delete Private DNS VNet link.
az network private-dns link vnet delete \
  --resource-group "<dns-resource-group>" \
  --zone-name "<private-zone-name>" \
  --name "<private-dns-link-name>" \
  --yes

# Delete Private DNS zone if lab-only and unused.
az network private-dns zone delete \
  --resource-group "<dns-resource-group>" \
  --name "<private-zone-name>" \
  --yes
```

```bash
# DNS Private Resolver rollback.
# Delete forwarding ruleset VNet link.
az dns-resolver forwarding-ruleset virtual-network-link delete \
  --resource-group "<resource-group>" \
  --ruleset-name "<forwarding-ruleset-name>" \
  --name "link-<vnet-name>" \
  --yes

# Delete forwarding rule.
az dns-resolver forwarding-ruleset rule delete \
  --resource-group "<resource-group>" \
  --ruleset-name "<forwarding-ruleset-name>" \
  --name "<forwarding-rule-name>" \
  --yes

# Delete forwarding ruleset.
az dns-resolver forwarding-ruleset delete \
  --resource-group "<resource-group>" \
  --name "<forwarding-ruleset-name>" \
  --yes

# Delete outbound endpoint.
az dns-resolver outbound-endpoint delete \
  --resource-group "<resource-group>" \
  --dns-resolver-name "<dns-private-resolver-name>" \
  --name "<outbound-endpoint-name>" \
  --yes

# Delete inbound endpoint.
az dns-resolver inbound-endpoint delete \
  --resource-group "<resource-group>" \
  --dns-resolver-name "<dns-private-resolver-name>" \
  --name "<inbound-endpoint-name>" \
  --yes

# Delete DNS Private Resolver.
az dns-resolver delete \
  --resource-group "<resource-group>" \
  --name "<dns-private-resolver-name>" \
  --yes
```

```powershell
# PowerShell route rollback.

$ResourceGroup = "<resource-group>"
$VNetName = "<vnet-name>"
$WorkloadSubnetName = "<workload-subnet-name>"
$RouteTableName = "<route-table-name>"

$VNet = Get-AzVirtualNetwork `
  -ResourceGroupName $ResourceGroup `
  -Name $VNetName

$Subnet = $VNet.Subnets | Where-Object Name -eq $WorkloadSubnetName

Set-AzVirtualNetworkSubnetConfig `
  -VirtualNetwork $VNet `
  -Name $WorkloadSubnetName `
  -AddressPrefix $Subnet.AddressPrefix `
  -RouteTable $null

$VNet | Set-AzVirtualNetwork

Remove-AzRouteTable `
  -ResourceGroupName $ResourceGroup `
  -Name $RouteTableName `
  -Force
```

# Configure_UDRs_Route_Tables_Azure_DNS_And_Name_Resolution_Failure_Checks
| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| Route table exists but traffic ignores it | Route table not associated to subnet | Check subnet `routeTable.id` | Associate route table to correct subnet |
| Route table cannot be deleted | Still associated to subnet | List subnet associations | Dissociate route table first |
| Traffic goes to wrong next hop | More specific route wins | Check effective routes and prefix length | Adjust destination prefix |
| UDR does not beat route | Service endpoint or platform preferred route | Check effective routes for `VirtualNetworkServiceEndpoint` | Redesign path or remove service endpoint if appropriate |
| Forced tunnel breaks internet | `0.0.0.0/0` points to NVA without egress | Effective route table and NVA NAT policy | Add NVA egress/NAT or remove default UDR |
| Gateway stops working | Bad route table or propagation setting on `GatewaySubnet` | Check route table association on `GatewaySubnet` | Remove route table or fix gateway design |
| NVA does not forward traffic | Azure NIC IP forwarding disabled | Check `enableIPForwarding` | Enable NIC IP forwarding |
| NVA still does not forward traffic | OS or appliance forwarding disabled | Check appliance config | Enable routing, NAT, and security policy on appliance |
| Return traffic fails | Asymmetric routing | Check effective routes on both source and destination NICs | Add matching return path routes |
| Peered VNet route missing | Peering not connected or address space changed | Check peering state and sync | Sync or recreate peering |
| BGP routes missing | Gateway route propagation disabled | Check route table setting | Enable propagation unless intentionally disabled |
| Private DNS name does not resolve | VNet not linked to private DNS zone | List private DNS VNet links | Link VNet to zone |
| Private DNS record missing | A record not created or zone group failed | List record sets | Add record or attach correct zone group |
| Azure-provided DNS does not resolve across VNets | Azure-provided DNS is scoped to a VNet | Test from each VNet | Use Private DNS zones or DNS Private Resolver |
| Custom DNS not used by VM | VM has not renewed DHCP or restarted | `ipconfig /all` | Restart VM or renew DHCP |
| Custom DNS breaks Azure private zones | DNS server does not forward correctly | Query private zone and Azure DNS | Configure conditional forwarders or DNS Private Resolver |
| DNS Private Resolver inbound endpoint not reachable | No network path from source to inbound endpoint | Check routing and NSGs | Add route, firewall rule, or VPN/ExpressRoute path |
| DNS Private Resolver outbound forwarding fails | Ruleset not linked to VNet | List forwarding ruleset VNet links | Link ruleset to VNet |
| Wrong DNS forwarding rule matches | Longer suffix or wrong domain format | List forwarding rules | Fix suffix and include trailing dot if required |
| Public DNS works but private endpoint fails | Private endpoint DNS zone missing or wrong | Resolve service FQDN | Use correct `privatelink` zone and VNet link |
| Route test passes but app fails | NSG, firewall, or identity issue | Check effective NSGs and app auth | Fix security rule or app permission |
| DNS test passes but TCP fails | Route or NSG path issue | Test effective routes and NSGs | Fix routing or NSG |
| TCP test passes but app login fails | Network is fine, identity is broken | Check RBAC, keys, token, or service auth | Fix application authorization |

# Configure_UDRs_Route_Tables_Azure_DNS_And_Name_Resolution_Related_Labs
| Lab | Relationship |
|---|---|
| `01_Create_VNets_Subnets_Public_IPs_And_Address_Space_Baseline.md` | Provides VNet and subnet base used by route tables and DNS links |
| `02_Configure_VNet_Peering_DNS_Resolution_And_Private_DNS_Zones.md` | Provides peering and Private DNS zone concepts extended here |
| `03_Configure_NSGs_ASGs_Effective_Security_Rules_And_Service_Endpoints.md` | Adds security controls that must be separated from route failures |
| `04_Configure_Private_Endpoints_Private_Link_And_PaaS_Network_Access.md` | Depends on private DNS and correct route path to private endpoint IPs |
| `06_Configure_Azure_Load_Balancer_Bastion_And_Basic_Traffic_Distribution.md` | Requires route and DNS clarity before traffic distribution work |
| `07_Troubleshoot_Azure_Networking_With_Network_Watcher_Connection_Monitor_And_Effective_Routes.md` | Uses Network Watcher for deeper validation of routes and connection paths |
| `08_Troubleshoot_Private_Endpoint_DNS_Routing_NSG_And_Load_Balancer_Issues.md` | Combines DNS, route, NSG, and load balancer troubleshooting |