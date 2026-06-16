02_Configure_VNet_Peering_DNS_Resolution_And_Private_DNS_Zones.md

# Configure_VNet_Peering_DNS_Resolution_And_Private_DNS_Zones

# Configure_VNet_Peering_DNS_Resolution_And_Private_DNS_Zones_Index
02_Configure_VNet_Peering_DNS_Resolution_And_Private_DNS_Zones.md
Configure_VNet_Peering_DNS_Resolution_And_Private_DNS_Zones
Configure_VNet_Peering_DNS_Resolution_And_Private_DNS_Zones_Source_Basis
Configure_VNet_Peering_DNS_Resolution_And_Private_DNS_Zones_Mental_Model
Configure_VNet_Peering_DNS_Resolution_And_Private_DNS_Zones_Planning_Table
Configure_VNet_Peering_DNS_Resolution_And_Private_DNS_Zones_Configuration_Checklist
Configure_VNet_Peering_DNS_Resolution_And_Private_DNS_Zones_Azure_CLI_Peering_Skeleton
Configure_VNet_Peering_DNS_Resolution_And_Private_DNS_Zones_PowerShell_Peering_Skeleton
Configure_VNet_Peering_DNS_Resolution_And_Private_DNS_Zones_Private_DNS_Zone_Skeleton
Configure_VNet_Peering_DNS_Resolution_And_Private_DNS_Zones_Private_DNS_Record_Skeleton
Configure_VNet_Peering_DNS_Resolution_And_Private_DNS_Zones_DNS_Validation_Skeleton
Configure_VNet_Peering_DNS_Resolution_And_Private_DNS_Zones_Evidence_Export_Skeleton
Configure_VNet_Peering_DNS_Resolution_And_Private_DNS_Zones_Verification_Commands
Configure_VNet_Peering_DNS_Resolution_And_Private_DNS_Zones_Rollback
Configure_VNet_Peering_DNS_Resolution_And_Private_DNS_Zones_Failure_Checks
Configure_VNet_Peering_DNS_Resolution_And_Private_DNS_Zones_Related_Labs

# Configure_VNet_Peering_DNS_Resolution_And_Private_DNS_Zones_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Azure virtual network peering overview | VNet peering behavior, local peering, global peering, Azure backbone traffic, and hub-spoke patterns |
| Microsoft Learn | Create, change, or delete Azure virtual network peering | Peering creation, peering settings, bidirectional peering, forwarded traffic, gateway transit, and deletion |
| Microsoft Learn | Virtual network peering requirements and constraints | Non-overlapping address spaces, non-transitive peering, same-cloud limitations, and global peering constraints |
| Microsoft Learn | Azure Private DNS overview | Private DNS zone behavior, linked VNets, split-horizon naming, and private name resolution |
| Microsoft Learn | Private DNS virtual network links | Registration links, resolution links, link status, autoregistration, and VNet link limits |
| Microsoft Learn | Azure CLI Private DNS quickstart | `az network private-dns zone create`, `az network private-dns link vnet create`, and private A record creation |
| Microsoft Learn | Azure PowerShell Private DNS quickstart | `New-AzPrivateDnsZone` and `New-AzPrivateDnsVirtualNetworkLink` |
| Azure operational practice | Hub-spoke network design | Pairing peering with central DNS and later routing/security workbooks |
| Azure operational practice | DNS validation before private endpoints | Proving custom private name resolution before PaaS private endpoint DNS is introduced |

# Configure_VNet_Peering_DNS_Resolution_And_Private_DNS_Zones_Mental_Model
| Concept | Operational Meaning |
|---|---|
| VNet peering | Direct private connectivity between two Azure VNets over the Microsoft backbone |
| Local peering | Peering between VNets in the same Azure region |
| Global peering | Peering between VNets in different Azure regions |
| Bidirectional peering | Peering must exist from VNet A to VNet B and from VNet B to VNet A |
| Peering status | `Connected` means both sides of the peering are complete |
| Allow virtual network access | Lets resources in peered VNets communicate through the peering path |
| Allow forwarded traffic | Allows traffic forwarded by an appliance or routing device to cross the peering |
| Gateway transit | Allows a spoke VNet to use a gateway in a peered hub VNet |
| Use remote gateway | Spoke setting that uses the hub gateway when hub gateway transit is enabled |
| Non-transitive peering | If A peers with B and B peers with C, A does not automatically reach C |
| Address overlap | Peering fails if VNet address spaces overlap |
| Azure default DNS | Good inside a VNet, but not enough for name resolution across peered VNets |
| Private DNS zone | Azure-managed private DNS namespace linked to one or more VNets |
| Registration VNet link | Link that allows VM records from that VNet to be automatically registered in the zone |
| Resolution VNet link | Link that allows VMs in that VNet to resolve zone records but does not auto-register records |
| Split-horizon DNS | Same zone name can exist publicly and privately with different answers |
| First rule | Peering gives IP reachability, not complete DNS design |
| Blunt rule | If DNS resolution is not validated now, private endpoints and app dependencies will look broken later |

# Configure_VNet_Peering_DNS_Resolution_And_Private_DNS_Zones_Planning_Table
| Item | Example | Decision |
|---|---|---|
| Subscription | `Visual Studio Enterprise` | `<subscription-name>` |
| Hub resource group | `rg-net-hub-eus2-01` | `<hub-resource-group>` |
| Spoke resource group | `rg-net-spoke-eus2-01` | `<spoke-resource-group>` |
| DNS resource group | `rg-dns-private-eus2-01` | `<dns-resource-group>` |
| Hub region | `eastus2` | `<hub-region>` |
| Spoke region | `eastus2` or `centralus` | `<spoke-region>` |
| Hub VNet name | `vnet-hub-eus2-01` | `<hub-vnet-name>` |
| Hub VNet address space | `10.40.0.0/16` | `<hub-address-space>` |
| Spoke VNet name | `vnet-spoke-app-eus2-01` | `<spoke-vnet-name>` |
| Spoke VNet address space | `10.50.0.0/16` | `<spoke-address-space>` |
| Hub-to-spoke peering name | `peer-hub-to-spoke-app-01` | `<hub-to-spoke-peering>` |
| Spoke-to-hub peering name | `peer-spoke-app-to-hub-01` | `<spoke-to-hub-peering>` |
| Allow VNet access | `true` | `<true-false>` |
| Allow forwarded traffic | `false` baseline, `true` with NVA | `<true-false>` |
| Allow gateway transit | `false` baseline | `<true-false>` |
| Use remote gateway | `false` baseline | `<true-false>` |
| Private DNS zone | `corp.internal` | `<private-zone-name>` |
| Hub DNS link name | `link-hub-vnet` | `<hub-dns-link-name>` |
| Spoke DNS link name | `link-spoke-vnet` | `<spoke-dns-link-name>` |
| Registration link VNet | Hub or spoke, not accidental | `<registration-vnet>` |
| Resolution link VNet | Hub and spoke as needed | `<resolution-vnet>` |
| Test record name | `app01` | `<test-record-name>` |
| Test record IP | `10.50.10.10` | `<test-record-ip>` |
| Test FQDN | `app01.corp.internal` | `<test-fqdn>` |
| Test VM in hub | `vm-hub-test-01` | `<hub-test-vm>` |
| Test VM in spoke | `vm-spoke-test-01` | `<spoke-test-vm>` |
| Evidence path | `C:\Azure-VNet-Peering-DNS` | `<evidence-path>` |
| Rollback stance | Remove links, peerings, and test zone records | `<rollback-plan>` |

# Configure_VNet_Peering_DNS_Resolution_And_Private_DNS_Zones_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm Azure login | Admin Workstation / Cloud Shell | `az account show` or `Get-AzContext` | Active Azure session is visible |
| 2 | Select target subscription | Admin Workstation / Cloud Shell | `az account set --subscription "<subscription-name>"` or `Set-AzContext -Subscription "<subscription-name>"` | Correct subscription is active |
| 3 | Confirm hub VNet exists | Admin Workstation / Cloud Shell | `az network vnet show -g "<hub-resource-group>" -n "<hub-vnet-name>"` | Hub VNet exists |
| 4 | Confirm spoke VNet exists | Admin Workstation / Cloud Shell | `az network vnet show -g "<spoke-resource-group>" -n "<spoke-vnet-name>"` | Spoke VNet exists |
| 5 | Confirm address spaces do not overlap | Admin Workstation / Cloud Shell | `az network vnet list --query "[].{Name:name,RG:resourceGroup,AddressSpace:addressSpace.addressPrefixes}" -o table` | Hub and spoke CIDRs are unique |
| 6 | Capture hub VNet ID | Admin Workstation / Cloud Shell | `az network vnet show -g "<hub-resource-group>" -n "<hub-vnet-name>" --query id -o tsv` | Hub VNet resource ID is known |
| 7 | Capture spoke VNet ID | Admin Workstation / Cloud Shell | `az network vnet show -g "<spoke-resource-group>" -n "<spoke-vnet-name>" --query id -o tsv` | Spoke VNet resource ID is known |
| 8 | Create hub-to-spoke peering | Admin Workstation / Cloud Shell | `az network vnet peering create -g "<hub-resource-group>" --vnet-name "<hub-vnet-name>" -n "<hub-to-spoke-peering>" --remote-vnet "<spoke-vnet-id>" --allow-vnet-access` | First peering direction exists |
| 9 | Create spoke-to-hub peering | Admin Workstation / Cloud Shell | `az network vnet peering create -g "<spoke-resource-group>" --vnet-name "<spoke-vnet-name>" -n "<spoke-to-hub-peering>" --remote-vnet "<hub-vnet-id>" --allow-vnet-access` | Second peering direction exists |
| 10 | Verify hub peering status | Admin Workstation / Cloud Shell | `az network vnet peering show -g "<hub-resource-group>" --vnet-name "<hub-vnet-name>" -n "<hub-to-spoke-peering>" --query "{Name:name,State:peeringState,Sync:peeringSyncLevel,AllowVnet:allowVirtualNetworkAccess}" -o table` | Peering state is `Connected` |
| 11 | Verify spoke peering status | Admin Workstation / Cloud Shell | `az network vnet peering show -g "<spoke-resource-group>" --vnet-name "<spoke-vnet-name>" -n "<spoke-to-hub-peering>" --query "{Name:name,State:peeringState,Sync:peeringSyncLevel,AllowVnet:allowVirtualNetworkAccess}" -o table` | Peering state is `Connected` |
| 12 | Enable forwarded traffic only if required | Admin Workstation / Cloud Shell | `az network vnet peering update -g "<hub-resource-group>" --vnet-name "<hub-vnet-name>" -n "<hub-to-spoke-peering>" --set allowForwardedTraffic=true` | Forwarded traffic is allowed only by design |
| 13 | Enable opposite forwarded traffic only if required | Admin Workstation / Cloud Shell | `az network vnet peering update -g "<spoke-resource-group>" --vnet-name "<spoke-vnet-name>" -n "<spoke-to-hub-peering>" --set allowForwardedTraffic=true` | Opposite direction is allowed only by design |
| 14 | Create DNS resource group if separate | Admin Workstation / Cloud Shell | `az group create --name "<dns-resource-group>" --location "<hub-region>"` | DNS resource group exists |
| 15 | Create private DNS zone | Admin Workstation / Cloud Shell | `az network private-dns zone create -g "<dns-resource-group>" -n "<private-zone-name>"` | Private DNS zone exists |
| 16 | Link hub VNet to private DNS zone | Admin Workstation / Cloud Shell | `az network private-dns link vnet create -g "<dns-resource-group>" -n "<hub-dns-link-name>" -z "<private-zone-name>" -v "<hub-vnet-id>" -e false` | Hub VNet can resolve private zone records |
| 17 | Link spoke VNet to private DNS zone | Admin Workstation / Cloud Shell | `az network private-dns link vnet create -g "<dns-resource-group>" -n "<spoke-dns-link-name>" -z "<private-zone-name>" -v "<spoke-vnet-id>" -e false` | Spoke VNet can resolve private zone records |
| 18 | Enable registration only where intended | Admin Workstation / Cloud Shell | `az network private-dns link vnet update -g "<dns-resource-group>" -n "<registration-link-name>" -z "<private-zone-name>" --registration-enabled true` | Only selected VNet auto-registers VM records |
| 19 | Create manual private A record | Admin Workstation / Cloud Shell | `az network private-dns record-set a add-record -g "<dns-resource-group>" -z "<private-zone-name>" -n "<test-record-name>" -a "<test-record-ip>"` | Test record exists |
| 20 | Verify private DNS zone records | Admin Workstation / Cloud Shell | `az network private-dns record-set list -g "<dns-resource-group>" -z "<private-zone-name>" -o table` | Expected record sets are visible |
| 21 | Verify DNS VNet links | Admin Workstation / Cloud Shell | `az network private-dns link vnet list -g "<dns-resource-group>" -z "<private-zone-name>" -o table` | Hub and spoke links are visible |
| 22 | Test IP connectivity across peering | Hub or Spoke VM | `Test-NetConnection <remote-private-ip>` | Remote private IP is reachable if NSGs and firewall allow it |
| 23 | Test private DNS name resolution from hub | Hub VM | `Resolve-DnsName "<test-fqdn>"` | FQDN resolves to expected private IP |
| 24 | Test private DNS name resolution from spoke | Spoke VM | `Resolve-DnsName "<test-fqdn>"` | FQDN resolves to expected private IP |
| 25 | Validate that default Azure DNS alone is not assumed | Operator | Compare DNS results before and after private zone link | DNS design is explicit |
| 26 | Export peering and DNS evidence | Admin Workstation / Cloud Shell | Run evidence export skeleton | Evidence files are saved |
| 27 | Document final state | Operator | Record peerings, DNS zone, VNet links, test records, and validation results | Workbook is complete |

# Configure_VNet_Peering_DNS_Resolution_And_Private_DNS_Zones_Azure_CLI_Peering_Skeleton
```bash
# Purpose: create bidirectional VNet peering between hub and spoke VNets.
# Run from Azure Cloud Shell Bash or local Azure CLI.

subscriptionName="<subscription-name>"

hubResourceGroup="<hub-resource-group>"
spokeResourceGroup="<spoke-resource-group>"

hubVnetName="<hub-vnet-name>"
spokeVnetName="<spoke-vnet-name>"

hubToSpokePeering="<hub-to-spoke-peering>"
spokeToHubPeering="<spoke-to-hub-peering>"

az account set \
  --subscription "$subscriptionName"

# Capture resource IDs.
hubVnetId=$(az network vnet show \
  --resource-group "$hubResourceGroup" \
  --name "$hubVnetName" \
  --query id \
  --output tsv)

spokeVnetId=$(az network vnet show \
  --resource-group "$spokeResourceGroup" \
  --name "$spokeVnetName" \
  --query id \
  --output tsv)

# Create hub to spoke peering.
az network vnet peering create \
  --resource-group "$hubResourceGroup" \
  --vnet-name "$hubVnetName" \
  --name "$hubToSpokePeering" \
  --remote-vnet "$spokeVnetId" \
  --allow-vnet-access

# Create spoke to hub peering.
az network vnet peering create \
  --resource-group "$spokeResourceGroup" \
  --vnet-name "$spokeVnetName" \
  --name "$spokeToHubPeering" \
  --remote-vnet "$hubVnetId" \
  --allow-vnet-access

# Optional: allow forwarded traffic only if an NVA, firewall, or routing appliance design requires it.
# az network vnet peering update \
#   --resource-group "$hubResourceGroup" \
#   --vnet-name "$hubVnetName" \
#   --name "$hubToSpokePeering" \
#   --set allowForwardedTraffic=true

# az network vnet peering update \
#   --resource-group "$spokeResourceGroup" \
#   --vnet-name "$spokeVnetName" \
#   --name "$spokeToHubPeering" \
#   --set allowForwardedTraffic=true

# Verify.
az network vnet peering list \
  --resource-group "$hubResourceGroup" \
  --vnet-name "$hubVnetName" \
  --output table

az network vnet peering list \
  --resource-group "$spokeResourceGroup" \
  --vnet-name "$spokeVnetName" \
  --output table
```

# Configure_VNet_Peering_DNS_Resolution_And_Private_DNS_Zones_PowerShell_Peering_Skeleton
```powershell
# Purpose: create bidirectional VNet peering between hub and spoke VNets.
# Run from Azure Cloud Shell PowerShell or local PowerShell with Az.Network.

$SubscriptionName = "<subscription-name>"

$HubResourceGroup = "<hub-resource-group>"
$SpokeResourceGroup = "<spoke-resource-group>"

$HubVNetName = "<hub-vnet-name>"
$SpokeVNetName = "<spoke-vnet-name>"

$HubToSpokePeering = "<hub-to-spoke-peering>"
$SpokeToHubPeering = "<spoke-to-hub-peering>"

Set-AzContext -Subscription $SubscriptionName

$HubVNet = Get-AzVirtualNetwork `
  -ResourceGroupName $HubResourceGroup `
  -Name $HubVNetName

$SpokeVNet = Get-AzVirtualNetwork `
  -ResourceGroupName $SpokeResourceGroup `
  -Name $SpokeVNetName

# Create hub to spoke peering.
Add-AzVirtualNetworkPeering `
  -Name $HubToSpokePeering `
  -VirtualNetwork $HubVNet `
  -RemoteVirtualNetworkId $SpokeVNet.Id `
  -AllowVirtualNetworkAccess

# Create spoke to hub peering.
Add-AzVirtualNetworkPeering `
  -Name $SpokeToHubPeering `
  -VirtualNetwork $SpokeVNet `
  -RemoteVirtualNetworkId $HubVNet.Id `
  -AllowVirtualNetworkAccess

# Optional: allow forwarded traffic only if appliance routing requires it.
# $HubPeer = Get-AzVirtualNetworkPeering `
#   -ResourceGroupName $HubResourceGroup `
#   -VirtualNetworkName $HubVNetName `
#   -Name $HubToSpokePeering
#
# $HubPeer.AllowForwardedTraffic = $true
# Set-AzVirtualNetworkPeering -VirtualNetworkPeering $HubPeer

# Verify.
Get-AzVirtualNetworkPeering `
  -ResourceGroupName $HubResourceGroup `
  -VirtualNetworkName $HubVNetName

Get-AzVirtualNetworkPeering `
  -ResourceGroupName $SpokeResourceGroup `
  -VirtualNetworkName $SpokeVNetName
```

# Configure_VNet_Peering_DNS_Resolution_And_Private_DNS_Zones_Private_DNS_Zone_Skeleton
```bash
# Purpose: create private DNS zone and link hub/spoke VNets for name resolution.
# Keep private endpoint service zones for the private endpoint workbook.
# This workbook uses a custom private zone such as corp.internal.

subscriptionName="<subscription-name>"

dnsResourceGroup="<dns-resource-group>"
location="<hub-region>"

hubResourceGroup="<hub-resource-group>"
spokeResourceGroup="<spoke-resource-group>"

hubVnetName="<hub-vnet-name>"
spokeVnetName="<spoke-vnet-name>"

privateZoneName="<private-zone-name>"

hubDnsLinkName="<hub-dns-link-name>"
spokeDnsLinkName="<spoke-dns-link-name>"

az account set \
  --subscription "$subscriptionName"

# Create DNS resource group if needed.
az group create \
  --name "$dnsResourceGroup" \
  --location "$location"

# Capture VNet IDs.
hubVnetId=$(az network vnet show \
  --resource-group "$hubResourceGroup" \
  --name "$hubVnetName" \
  --query id \
  --output tsv)

spokeVnetId=$(az network vnet show \
  --resource-group "$spokeResourceGroup" \
  --name "$spokeVnetName" \
  --query id \
  --output tsv)

# Create private DNS zone.
az network private-dns zone create \
  --resource-group "$dnsResourceGroup" \
  --name "$privateZoneName"

# Link hub VNet for resolution only.
az network private-dns link vnet create \
  --resource-group "$dnsResourceGroup" \
  --zone-name "$privateZoneName" \
  --name "$hubDnsLinkName" \
  --virtual-network "$hubVnetId" \
  --registration-enabled false

# Link spoke VNet for resolution only.
az network private-dns link vnet create \
  --resource-group "$dnsResourceGroup" \
  --zone-name "$privateZoneName" \
  --name "$spokeDnsLinkName" \
  --virtual-network "$spokeVnetId" \
  --registration-enabled false

# Optional: enable registration on one intended link only if VM autoregistration is part of the design.
# az network private-dns link vnet update \
#   --resource-group "$dnsResourceGroup" \
#   --zone-name "$privateZoneName" \
#   --name "$spokeDnsLinkName" \
#   --registration-enabled true

# Verify links.
az network private-dns link vnet list \
  --resource-group "$dnsResourceGroup" \
  --zone-name "$privateZoneName" \
  --output table
```

# Configure_VNet_Peering_DNS_Resolution_And_Private_DNS_Zones_Private_DNS_Record_Skeleton
```bash
# Purpose: create a manual private A record for cross-VNet DNS validation.

dnsResourceGroup="<dns-resource-group>"
privateZoneName="<private-zone-name>"

recordName="<test-record-name>"
recordIp="<test-record-ip>"

# Add A record.
az network private-dns record-set a add-record \
  --resource-group "$dnsResourceGroup" \
  --zone-name "$privateZoneName" \
  --record-set-name "$recordName" \
  --ipv4-address "$recordIp"

# Verify record set.
az network private-dns record-set a show \
  --resource-group "$dnsResourceGroup" \
  --zone-name "$privateZoneName" \
  --name "$recordName" \
  --output table

# List all records.
az network private-dns record-set list \
  --resource-group "$dnsResourceGroup" \
  --zone-name "$privateZoneName" \
  --output table
```

# Configure_VNet_Peering_DNS_Resolution_And_Private_DNS_Zones_DNS_Validation_Skeleton
```powershell
# Purpose: validate IP reachability and private DNS resolution from test VMs.
# Run inside hub and spoke test VMs.

$RemotePrivateIp = "<remote-private-ip>"
$TestFqdn = "<test-fqdn>"
$ExpectedIp = "<test-record-ip>"

# Confirm local IP configuration.
ipconfig /all

# Test raw private IP path across peering.
Test-NetConnection `
  -ComputerName $RemotePrivateIp `
  -InformationLevel Detailed

# Resolve private DNS record.
Resolve-DnsName `
  -Name $TestFqdn

# Confirm expected answer.
$Result = Resolve-DnsName -Name $TestFqdn -Type A

$Result |
  Select-Object Name, IPAddress, Type

if ($Result.IPAddress -contains $ExpectedIp) {
  Write-Host "PASS: $TestFqdn resolves to $ExpectedIp"
}
else {
  Write-Host "FAIL: $TestFqdn did not resolve to $ExpectedIp"
}

# Optional ICMP test if guest firewall allows ICMP.
ping $TestFqdn
```

# Configure_VNet_Peering_DNS_Resolution_And_Private_DNS_Zones_Evidence_Export_Skeleton
```powershell
# Purpose: export evidence for peering and Private DNS configuration.

$HubResourceGroup = "<hub-resource-group>"
$SpokeResourceGroup = "<spoke-resource-group>"
$DnsResourceGroup = "<dns-resource-group>"

$HubVNetName = "<hub-vnet-name>"
$SpokeVNetName = "<spoke-vnet-name>"

$PrivateZoneName = "<private-zone-name>"
$EvidencePath = "C:\Azure-VNet-Peering-DNS"

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

az account show `
  --output json |
  Out-File "$EvidencePath\account-context.json" -Encoding utf8

az network vnet show `
  --resource-group $HubResourceGroup `
  --name $HubVNetName `
  --output json |
  Out-File "$EvidencePath\hub-vnet.json" -Encoding utf8

az network vnet show `
  --resource-group $SpokeResourceGroup `
  --name $SpokeVNetName `
  --output json |
  Out-File "$EvidencePath\spoke-vnet.json" -Encoding utf8

az network vnet peering list `
  --resource-group $HubResourceGroup `
  --vnet-name $HubVNetName `
  --output json |
  Out-File "$EvidencePath\hub-peerings.json" -Encoding utf8

az network vnet peering list `
  --resource-group $SpokeResourceGroup `
  --vnet-name $SpokeVNetName `
  --output json |
  Out-File "$EvidencePath\spoke-peerings.json" -Encoding utf8

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
```

# Configure_VNet_Peering_DNS_Resolution_And_Private_DNS_Zones_Verification_Commands
```bash
# VNet address space verification.
az network vnet list \
  --query "[].{Name:name,RG:resourceGroup,Location:location,AddressSpace:addressSpace.addressPrefixes}" \
  --output table

# Hub peering verification.
az network vnet peering list \
  --resource-group "<hub-resource-group>" \
  --vnet-name "<hub-vnet-name>" \
  --output table

az network vnet peering show \
  --resource-group "<hub-resource-group>" \
  --vnet-name "<hub-vnet-name>" \
  --name "<hub-to-spoke-peering>" \
  --query "{Name:name,State:peeringState,Sync:peeringSyncLevel,AllowVnet:allowVirtualNetworkAccess,AllowForwarded:allowForwardedTraffic,AllowGatewayTransit:allowGatewayTransit,UseRemoteGateway:useRemoteGateways}" \
  --output table

# Spoke peering verification.
az network vnet peering list \
  --resource-group "<spoke-resource-group>" \
  --vnet-name "<spoke-vnet-name>" \
  --output table

az network vnet peering show \
  --resource-group "<spoke-resource-group>" \
  --vnet-name "<spoke-vnet-name>" \
  --name "<spoke-to-hub-peering>" \
  --query "{Name:name,State:peeringState,Sync:peeringSyncLevel,AllowVnet:allowVirtualNetworkAccess,AllowForwarded:allowForwardedTraffic,AllowGatewayTransit:allowGatewayTransit,UseRemoteGateway:useRemoteGateways}" \
  --output table

# Private DNS zone verification.
az network private-dns zone list \
  --resource-group "<dns-resource-group>" \
  --output table

az network private-dns zone show \
  --resource-group "<dns-resource-group>" \
  --name "<private-zone-name>" \
  --output table

# Private DNS VNet links.
az network private-dns link vnet list \
  --resource-group "<dns-resource-group>" \
  --zone-name "<private-zone-name>" \
  --output table

az network private-dns link vnet show \
  --resource-group "<dns-resource-group>" \
  --zone-name "<private-zone-name>" \
  --name "<hub-dns-link-name>" \
  --query "{Name:name,Registration:registrationEnabled,Status:virtualNetworkLinkState,VNet:virtualNetwork.id}" \
  --output table

az network private-dns link vnet show \
  --resource-group "<dns-resource-group>" \
  --zone-name "<private-zone-name>" \
  --name "<spoke-dns-link-name>" \
  --query "{Name:name,Registration:registrationEnabled,Status:virtualNetworkLinkState,VNet:virtualNetwork.id}" \
  --output table

# Private DNS records.
az network private-dns record-set list \
  --resource-group "<dns-resource-group>" \
  --zone-name "<private-zone-name>" \
  --output table
```

```powershell
# Run inside test VM.
ipconfig /all
Resolve-DnsName "<test-fqdn>"
Test-NetConnection "<remote-private-ip>" -InformationLevel Detailed
Test-NetConnection "<test-fqdn>" -InformationLevel Detailed
```

# Configure_VNet_Peering_DNS_Resolution_And_Private_DNS_Zones_Rollback
```bash
# Remove test A record.
az network private-dns record-set a remove-record \
  --resource-group "<dns-resource-group>" \
  --zone-name "<private-zone-name>" \
  --record-set-name "<test-record-name>" \
  --ipv4-address "<test-record-ip>"

# Delete hub DNS link.
az network private-dns link vnet delete \
  --resource-group "<dns-resource-group>" \
  --zone-name "<private-zone-name>" \
  --name "<hub-dns-link-name>" \
  --yes

# Delete spoke DNS link.
az network private-dns link vnet delete \
  --resource-group "<dns-resource-group>" \
  --zone-name "<private-zone-name>" \
  --name "<spoke-dns-link-name>" \
  --yes

# Delete private DNS zone.
# Only do this if no production records or private endpoints depend on it.
az network private-dns zone delete \
  --resource-group "<dns-resource-group>" \
  --name "<private-zone-name>" \
  --yes

# Delete hub to spoke peering.
az network vnet peering delete \
  --resource-group "<hub-resource-group>" \
  --vnet-name "<hub-vnet-name>" \
  --name "<hub-to-spoke-peering>"

# Delete spoke to hub peering.
az network vnet peering delete \
  --resource-group "<spoke-resource-group>" \
  --vnet-name "<spoke-vnet-name>" \
  --name "<spoke-to-hub-peering>"
```

```powershell
# PowerShell rollback.

Remove-AzPrivateDnsRecordSet `
  -ResourceGroupName "<dns-resource-group>" `
  -ZoneName "<private-zone-name>" `
  -Name "<test-record-name>" `
  -RecordType A

Remove-AzPrivateDnsVirtualNetworkLink `
  -ResourceGroupName "<dns-resource-group>" `
  -ZoneName "<private-zone-name>" `
  -Name "<hub-dns-link-name>"

Remove-AzPrivateDnsVirtualNetworkLink `
  -ResourceGroupName "<dns-resource-group>" `
  -ZoneName "<private-zone-name>" `
  -Name "<spoke-dns-link-name>"

Remove-AzPrivateDnsZone `
  -ResourceGroupName "<dns-resource-group>" `
  -Name "<private-zone-name>"

Remove-AzVirtualNetworkPeering `
  -ResourceGroupName "<hub-resource-group>" `
  -VirtualNetworkName "<hub-vnet-name>" `
  -Name "<hub-to-spoke-peering>"

Remove-AzVirtualNetworkPeering `
  -ResourceGroupName "<spoke-resource-group>" `
  -VirtualNetworkName "<spoke-vnet-name>" `
  -Name "<spoke-to-hub-peering>"
```

# Configure_VNet_Peering_DNS_Resolution_And_Private_DNS_Zones_Failure_Checks
| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| Peering creation fails | VNet address spaces overlap | `az network vnet list --query "[].{Name:name,AddressSpace:addressSpace.addressPrefixes}" -o table` | Readdress one VNet before peering |
| Peering remains `Initiated` | Only one direction was created | Check peerings on both VNets | Create the reverse peering |
| Peering fails across subscriptions | Remote VNet ID not used or permissions missing | Confirm full remote VNet resource ID and RBAC | Use full resource ID and assign Network Contributor or custom role |
| VM cannot reach remote private IP | NSG or guest firewall blocks traffic | `Test-NetConnection <remote-private-ip>` and NSG effective rules | Allow required traffic in NSG and guest firewall |
| VM cannot reach remote private IP | Route table forces traffic somewhere else | Check effective routes on NIC | Fix UDRs in the UDR workbook |
| Traffic through NVA fails | Forwarded traffic disabled | Show peering settings | Enable `allowForwardedTraffic` on required direction |
| Spoke cannot use hub gateway | Gateway transit not configured correctly | Show both peering settings | Enable `allowGatewayTransit` on hub side and `useRemoteGateways` on spoke side |
| Remote gateway setting fails | Spoke already has its own gateway | Check spoke gateway resources | Remove local gateway or do not use remote gateway |
| DNS name does not resolve across peering | Azure default DNS was assumed | Try `Resolve-DnsName <fqdn>` from both VNets | Create Private DNS zone and VNet links |
| Private DNS link creation fails | Duplicate link exists | `az network private-dns link vnet list ...` | Reuse or delete duplicate link |
| Private DNS link status not completed | Link still provisioning | Show link state | Wait and recheck |
| Autoregistration does not work | Link is resolution-only | Show `registrationEnabled` | Enable registration on intended VNet link |
| Autoregistration conflicts | VNet already has another registration zone | List links and zones | Keep only one registration zone per VNet |
| Manual A record resolves wrong IP | Wrong record value | List record set | Remove bad record and add correct private IP |
| Split-horizon name resolves public answer | Client not using Azure-provided DNS path or private zone link missing | `ipconfig /all`; check VNet DNS settings and zone links | Link VNet to private zone or fix custom DNS forwarding |
| Private endpoint DNS later breaks | Private endpoint zone was mixed into generic custom zone design | Check zone name | Use proper `privatelink.*` zone in private endpoint workbook |

# Configure_VNet_Peering_DNS_Resolution_And_Private_DNS_Zones_Related_Labs
| Lab | Relationship |
|---|---|
| `01_Create_VNets_Subnets_Public_IPs_And_Address_Space_Baseline.md` | Provides the hub and spoke VNet baseline |
| `03_Configure_NSGs_ASGs_Effective_Security_Rules_And_Service_Endpoints.md` | Adds security filtering after peering and DNS basics are clean |
| `04_Configure_Private_Endpoints_Private_Link_And_PaaS_Network_Access.md` | Uses Private DNS mechanics for private endpoint service zones |
| `05_Configure_UDRs_Route_Tables_Azure_DNS_And_Name_Resolution.md` | Adds routing and deeper DNS resolution behavior |
| `06_Configure_Azure_Load_Balancer_Bastion_And_Basic_Traffic_Distribution.md` | Uses peered networks for management and traffic distribution paths |
| `07_Troubleshoot_Azure_Networking_With_Network_Watcher_Connection_Monitor_And_Effective_Routes.md` | Validates peering path and routing with Network Watcher |
| `08_Troubleshoot_Private_Endpoint_DNS_Routing_NSG_And_Load_Balancer_Issues.md` | Troubleshoots the DNS and routing failures that build on this workbook |