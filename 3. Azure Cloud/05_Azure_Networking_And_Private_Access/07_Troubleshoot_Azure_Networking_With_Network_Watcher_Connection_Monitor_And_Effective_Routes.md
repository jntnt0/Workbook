07_Troubleshoot_Azure_Networking_With_Network_Watcher_Connection_Monitor_And_Effective_Routes.md

# Troubleshoot_Azure_Networking_With_Network_Watcher_Connection_Monitor_And_Effective_Routes

# Troubleshoot_Azure_Networking_With_Network_Watcher_Connection_Monitor_And_Effective_Routes_Index
07_Troubleshoot_Azure_Networking_With_Network_Watcher_Connection_Monitor_And_Effective_Routes.md
Troubleshoot_Azure_Networking_With_Network_Watcher_Connection_Monitor_And_Effective_Routes
Troubleshoot_Azure_Networking_With_Network_Watcher_Connection_Monitor_And_Effective_Routes_Source_Basis
Troubleshoot_Azure_Networking_With_Network_Watcher_Connection_Monitor_And_Effective_Routes_Mental_Model
Troubleshoot_Azure_Networking_With_Network_Watcher_Connection_Monitor_And_Effective_Routes_Planning_Table
Troubleshoot_Azure_Networking_With_Network_Watcher_Connection_Monitor_And_Effective_Routes_Symptom_Map
Troubleshoot_Azure_Networking_With_Network_Watcher_Connection_Monitor_And_Effective_Routes_Triage_Order
Troubleshoot_Azure_Networking_With_Network_Watcher_Connection_Monitor_And_Effective_Routes_Configuration_Checklist
Troubleshoot_Azure_Networking_With_Network_Watcher_Connection_Monitor_And_Effective_Routes_Azure_CLI_Network_Watcher_Skeleton
Troubleshoot_Azure_Networking_With_Network_Watcher_Connection_Monitor_And_Effective_Routes_Connection_Monitor_Skeleton
Troubleshoot_Azure_Networking_With_Network_Watcher_Connection_Monitor_And_Effective_Routes_Connection_Troubleshoot_Skeleton
Troubleshoot_Azure_Networking_With_Network_Watcher_Connection_Monitor_And_Effective_Routes_IP_Flow_Verify_Skeleton
Troubleshoot_Azure_Networking_With_Network_Watcher_Connection_Monitor_And_Effective_Routes_Effective_Routes_Skeleton
Troubleshoot_Azure_Networking_With_Network_Watcher_Connection_Monitor_And_Effective_Routes_Effective_Security_Rules_Skeleton
Troubleshoot_Azure_Networking_With_Network_Watcher_Connection_Monitor_And_Effective_Routes_Next_Hop_Skeleton
Troubleshoot_Azure_Networking_With_Network_Watcher_Connection_Monitor_And_Effective_Routes_Flow_Logs_Skeleton
Troubleshoot_Azure_Networking_With_Network_Watcher_Connection_Monitor_And_Effective_Routes_Evidence_Export_Skeleton
Troubleshoot_Azure_Networking_With_Network_Watcher_Connection_Monitor_And_Effective_Routes_Verification_Commands
Troubleshoot_Azure_Networking_With_Network_Watcher_Connection_Monitor_And_Effective_Routes_Rollback
Troubleshoot_Azure_Networking_With_Network_Watcher_Connection_Monitor_And_Effective_Routes_Failure_Checks
Troubleshoot_Azure_Networking_With_Network_Watcher_Connection_Monitor_And_Effective_Routes_Related_Labs

# Troubleshoot_Azure_Networking_With_Network_Watcher_Connection_Monitor_And_Effective_Routes_Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Azure Network Watcher overview | Network Watcher scope, monitoring tools, diagnostic tools, traffic tools, Connection Monitor, IP flow verify, NSG diagnostics, Next hop, effective security rules, connection troubleshoot, packet capture, and flow logs |
| Microsoft Learn | Connection Monitor overview | Continuous connectivity monitoring, source and destination endpoints, TCP/ICMP/HTTP tests, latency, packet loss, topology, alerts, and Log Analytics integration |
| Microsoft Learn | Azure CLI `az network watcher connection-monitor` | Creating, listing, showing, querying, starting, stopping, and deleting connection monitors |
| Microsoft Learn | Connection troubleshoot overview | Point-in-time connection tests, reachability status, latency, hop path, NSG issues, UDR issues, DNS failures, guest firewall blocks, and destination port listener problems |
| Microsoft Learn | IP flow verify overview | Testing whether traffic to or from a VM is allowed or denied and identifying the matched security rule |
| Microsoft Learn | Next hop overview | Checking the next hop type, next hop IP, and route table used for a destination |
| Microsoft Learn | Effective security rules overview | Viewing aggregate inbound and outbound security rules applied to a NIC from subnet NSGs, NIC NSGs, and admin rules |
| Microsoft Learn | Route table management | Route table creation, route creation, subnet association, dissociation, deletion, and subnet-scoped route behavior |
| Microsoft Learn | Virtual network flow logs | Logging Layer 4 flow data for traffic through a virtual network |
| Azure operational practice | Evidence-driven troubleshooting | Separate DNS, routing, NSG, guest firewall, service listener, and platform health before changing configuration |
| Azure operational practice | Triage before fix | Run diagnostic commands first, collect evidence, then apply the smallest correction |

# Troubleshoot_Azure_Networking_With_Network_Watcher_Connection_Monitor_And_Effective_Routes_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Network Watcher | Azure diagnostic service for IaaS network visibility and troubleshooting |
| Connection Monitor | Continuous test that tracks reachability, latency, loss, and path behavior over time |
| Connection troubleshoot | Point-in-time connectivity test from a source to a destination |
| IP flow verify | Security rule test that says whether a packet is allowed or denied |
| NSG diagnostics | Security rule diagnosis across VM, VMSS, or Application Gateway context |
| Effective security rules | Final security rule view on a NIC after subnet NSG, NIC NSG, and admin rules are combined |
| Effective routes | Final route table view on a NIC after system routes, UDRs, BGP routes, peering routes, and service endpoint routes are combined |
| Next hop | Diagnostic output showing where Azure sends traffic for a specific destination IP |
| Flow logs | Recorded Layer 4 traffic data used for historical traffic validation |
| Packet capture | On-demand packet capture from a VM or VMSS for deeper inspection |
| Source | VM, VMSS, Bastion, Application Gateway, or agent-backed endpoint that initiates a test |
| Destination | VM, IP, FQDN, URI, or external endpoint receiving the test |
| Reachable | Diagnostic test reached the destination successfully |
| Unreachable | Diagnostic test failed and needs path, rule, DNS, or service listener analysis |
| UserDefinedRoute fault | UDR is steering, blackholing, looping, or misrouting traffic |
| NetworkSecurityRule fault | NSG or security admin rule is denying the flow |
| DNSResolution fault | Name lookup failed before network path could be tested correctly |
| GuestFirewall fault | Azure path may be correct, but the OS firewall blocks traffic |
| NoListenerOnDestination | Destination host is reachable, but no process is listening on the tested port |
| Blunt rule | Do not change routes or NSGs until the diagnostic output proves which layer failed |

# Troubleshoot_Azure_Networking_With_Network_Watcher_Connection_Monitor_And_Effective_Routes_Planning_Table
| Item | Example | Decision |
|---|---|---|
| Subscription | `Visual Studio Enterprise` | `<subscription-name>` |
| Resource group | `rg-net-core-lab-eus2-01` | `<resource-group>` |
| Region | `eastus2` | `<region>` |
| Network Watcher resource group | `NetworkWatcherRG` | `<network-watcher-resource-group>` |
| Network Watcher name | `NetworkWatcher_eastus2` | `<network-watcher-name>` |
| Source VM | `vm-workload-01` | `<source-vm-name>` |
| Source VM resource group | `rg-net-core-lab-eus2-01` | `<source-vm-resource-group>` |
| Source NIC | `nic-vm-workload-01` | `<source-nic-name>` |
| Source private IP | `10.40.10.4` | `<source-private-ip>` |
| Destination VM | `vm-app-01` | `<destination-vm-name>` |
| Destination private IP | `10.50.10.4` | `<destination-private-ip>` |
| Destination FQDN | `app01.corp.internal` | `<destination-fqdn>` |
| Destination URI | `https://app01.corp.internal/health` | `<destination-uri>` |
| Destination port | `443` | `<destination-port>` |
| Protocol | `Tcp` | `<protocol>` |
| Direction | `Outbound` | `<direction>` |
| Test source port | `50000` | `<source-port>` |
| Connection monitor name | `cm-workload-to-app-443` | `<connection-monitor-name>` |
| Source endpoint name | `ep-vm-workload-01` | `<source-endpoint-name>` |
| Destination endpoint name | `ep-app-private` | `<destination-endpoint-name>` |
| Test config name | `tc-tcp-443` | `<test-config-name>` |
| Test group name | `tg-workload-to-app` | `<test-group-name>` |
| Log Analytics workspace | `law-net-eus2-01` | `<workspace-name>` |
| Log Analytics workspace ID | `/subscriptions/.../workspaces/law-net-eus2-01` | `<workspace-resource-id>` |
| Route table | `rt-workload-eus2-01` | `<route-table-name>` |
| NSG | `nsg-workload-eus2-01` | `<nsg-name>` |
| Evidence path | `C:\Azure-NetworkWatcher-Troubleshooting` | `<evidence-path>` |
| Rollback stance | Delete temporary monitors, stop captures, remove temporary rules only after evidence is saved | `<rollback-plan>` |

# Troubleshoot_Azure_Networking_With_Network_Watcher_Connection_Monitor_And_Effective_Routes_Symptom_Map
| Symptom | First Tool | Supporting Tool | Likely Area |
|---|---|---|---|
| VM cannot reach private IP | Connection troubleshoot | Next hop, effective routes | Route table, peering, gateway, NVA |
| VM cannot reach FQDN | DNS tools | Connection troubleshoot | DNS resolution or Private DNS link |
| VM can resolve FQDN but TCP fails | Connection troubleshoot | IP flow verify, effective security rules | NSG, guest firewall, listener |
| VM can reach IP but not port | Connection troubleshoot | Guest OS check | Destination service listener or firewall |
| Traffic should use NVA but does not | Effective routes | Next hop | UDR, route table association, BGP propagation |
| Traffic disappears through NVA | Next hop | NVA OS or firewall logs | IP forwarding, NVA route policy, return route |
| Peered VNet unreachable | Connection troubleshoot | Effective routes, peering status | Peering, NSG, UDR, VNet access setting |
| Load balancer frontend fails | Connection troubleshoot | Effective security rules, backend health | Probe, backend NSG, listener |
| Bastion connection fails | Connection troubleshoot | NSG and VM guest checks | Bastion path, RDP/SSH service, VM state |
| Intermittent packet loss | Connection Monitor | Flow logs | Latency, packet loss, path instability |
| Historical proof needed | VNet flow logs | Traffic analytics or SIEM | Flow evidence and trend analysis |
| Portal says reachable but app fails | App test | Guest OS logs | Application auth, TLS, service configuration |

# Troubleshoot_Azure_Networking_With_Network_Watcher_Connection_Monitor_And_Effective_Routes_Triage_Order
| Order | Check | Tool | Stop Condition |
|---:|---|---|---|
| 1 | Confirm source and destination resource state | `az vm get-instance-view`, `az resource show` | Source and destination are running and provisioned |
| 2 | Confirm IP and DNS inputs | `ipconfig`, `Resolve-DnsName`, `nslookup` | Destination IP and name are correct |
| 3 | Confirm service listener | `Test-NetConnection localhost`, `ss`, `netstat` | Destination process listens on expected port |
| 4 | Check point-in-time reachability | Connection troubleshoot | Reachable or clear fault category returned |
| 5 | Check NSG allow or deny | IP flow verify and effective security rules | Matched allow or deny rule is identified |
| 6 | Check routing path | Effective routes and Next hop | Next hop matches expected design |
| 7 | Check route table association | Route table and subnet commands | Correct route table is attached to source subnet |
| 8 | Check peering or gateway path | Peering, gateway, route propagation | Cross-network dependency is healthy |
| 9 | Create continuous monitor if recurring | Connection Monitor | Latency, loss, and reachability are tracked over time |
| 10 | Enable flow evidence if historical proof is needed | VNet flow logs | Flow records are being captured |
| 11 | Change only the proven broken layer | NSG, UDR, DNS, guest firewall, or service listener | Retest shows recovery |
| 12 | Export evidence | Evidence skeleton | Before and after state is saved |

# Troubleshoot_Azure_Networking_With_Network_Watcher_Connection_Monitor_And_Effective_Routes_Configuration_Checklist
| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm Azure login | Admin Workstation / Cloud Shell | `az account show` or `Get-AzContext` | Active Azure session is visible |
| 2 | Select target subscription | Admin Workstation / Cloud Shell | `az account set --subscription "<subscription-name>"` or `Set-AzContext -Subscription "<subscription-name>"` | Correct subscription is active |
| 3 | Confirm Network Watcher state | Admin Workstation / Cloud Shell | `az network watcher list -o table` | Network Watcher exists in target region |
| 4 | Enable Network Watcher if missing | Admin Workstation / Cloud Shell | `az network watcher configure --locations "<region>" --enabled true` | Network Watcher is enabled |
| 5 | Confirm source VM state | Admin Workstation / Cloud Shell | `az vm get-instance-view -g "<source-vm-resource-group>" -n "<source-vm-name>" --query "instanceView.statuses[].displayStatus" -o table` | VM is running |
| 6 | Confirm destination VM or endpoint | Admin Workstation / Cloud Shell | `az resource show --ids "<destination-resource-id>"` or DNS/IP validation | Destination exists or endpoint is valid |
| 7 | Confirm source NIC and private IP | Admin Workstation / Cloud Shell | `az network nic show -g "<resource-group>" -n "<source-nic-name>" --query "ipConfigurations[].privateIPAddress" -o table` | Source IP is known |
| 8 | Confirm destination DNS result | Source VM | `Resolve-DnsName "<destination-fqdn>"` | FQDN resolves to expected IP |
| 9 | Confirm destination listener locally | Destination VM | `Test-NetConnection localhost -Port <destination-port>` | Service listens locally |
| 10 | Run connection troubleshoot | Admin Workstation / Cloud Shell | `az network watcher test-connectivity --source-resource "<source-vm-id>" --dest-address "<destination-private-ip>" --dest-port "<destination-port>" --protocol Tcp` | Reachability and issue details are returned |
| 11 | Run IP flow verify | Admin Workstation / Cloud Shell | `az network watcher test-ip-flow -g "<source-vm-resource-group>" --vm "<source-vm-name>" --direction Outbound --protocol TCP --local "<source-private-ip>:<source-port>" --remote "<destination-private-ip>:<destination-port>"` | Allow or Deny and matched rule are returned |
| 12 | View effective security rules | Admin Workstation / Cloud Shell | `az network nic list-effective-nsg -g "<resource-group>" -n "<source-nic-name>" -o table` | Effective NSG rules are visible |
| 13 | View effective routes | Admin Workstation / Cloud Shell | `az network nic show-effective-route-table -g "<resource-group>" -n "<source-nic-name>" -o table` | Effective routes are visible |
| 14 | Run next hop check | Admin Workstation / Cloud Shell | `az network watcher show-next-hop -g "<source-vm-resource-group>" --vm "<source-vm-name>" --source-ip "<source-private-ip>" --dest-ip "<destination-private-ip>"` | Next hop matches expected design |
| 15 | Confirm route table association | Admin Workstation / Cloud Shell | `az network vnet subnet list -g "<resource-group>" --vnet-name "<vnet-name>" --query "[].{Name:name,RouteTable:routeTable.id,Nsg:networkSecurityGroup.id}" -o table` | Route and NSG associations are known |
| 16 | Confirm peering state if cross-VNet | Admin Workstation / Cloud Shell | `az network vnet peering list -g "<resource-group>" --vnet-name "<vnet-name>" -o table` | Peering is connected if needed |
| 17 | Create Connection Monitor for recurring issue | Admin Workstation / Cloud Shell | Run connection monitor skeleton | Monitor exists and starts testing |
| 18 | Query Connection Monitor | Admin Workstation / Cloud Shell | `az network watcher connection-monitor query -l "<region>" -n "<connection-monitor-name>"` | Current connection state is returned |
| 19 | Enable VNet flow logs if historical evidence is required | Admin Workstation / Cloud Shell | Run flow logs skeleton | VNet flow logs are enabled |
| 20 | Save before-change evidence | Admin Workstation / Cloud Shell | Run evidence export skeleton | Diagnostic evidence is saved |
| 21 | Apply smallest fix | Admin Workstation / Cloud Shell | NSG, UDR, DNS, route table, guest firewall, or service change | Only proven broken layer is changed |
| 22 | Rerun same diagnostic tests | Admin Workstation / Cloud Shell | Repeat steps 10 through 14 | Test results show recovery |
| 23 | Save after-change evidence | Admin Workstation / Cloud Shell | Run evidence export skeleton again with `after` folder | Before and after evidence exists |
| 24 | Document root cause | Operator | Record failed layer, evidence, fix, and retest result | Troubleshooting record is complete |

# Troubleshoot_Azure_Networking_With_Network_Watcher_Connection_Monitor_And_Effective_Routes_Azure_CLI_Network_Watcher_Skeleton
```bash
# Purpose: confirm or enable Network Watcher in the target region.
# Run from Azure Cloud Shell Bash or local Azure CLI.

subscriptionName="<subscription-name>"
region="<region>"

az account set \
  --subscription "$subscriptionName"

# List Network Watcher instances.
az network watcher list \
  --output table

# Enable Network Watcher in target region if missing.
az network watcher configure \
  --locations "$region" \
  --enabled true

# Verify.
az network watcher list \
  --query "[].{Name:name,Location:location,ResourceGroup:resourceGroup,ProvisioningState:provisioningState}" \
  --output table
```

# Troubleshoot_Azure_Networking_With_Network_Watcher_Connection_Monitor_And_Effective_Routes_Connection_Monitor_Skeleton
```bash
# Purpose: create a continuous Connection Monitor for recurring reachability, latency, or packet-loss issues.
# Connection Monitor classic is deprecated. Use the current connection-monitor command set.

subscriptionName="<subscription-name>"
resourceGroup="<resource-group>"
region="<region>"

connectionMonitorName="<connection-monitor-name>"
sourceEndpointName="<source-endpoint-name>"
destinationEndpointName="<destination-endpoint-name>"
testConfigName="<test-config-name>"
testGroupName="<test-group-name>"

sourceVmId="<source-vm-resource-id>"
destinationAddress="<destination-fqdn-or-ip>"
destinationPort="<destination-port>"
workspaceId="<workspace-resource-id>"

az account set \
  --subscription "$subscriptionName"

# Create TCP connection monitor.
az network watcher connection-monitor create \
  --location "$region" \
  --name "$connectionMonitorName" \
  --resource-group "$resourceGroup" \
  --endpoint-source-name "$sourceEndpointName" \
  --endpoint-source-resource-id "$sourceVmId" \
  --endpoint-dest-name "$destinationEndpointName" \
  --endpoint-dest-address "$destinationAddress" \
  --test-config-name "$testConfigName" \
  --test-group-name "$testGroupName" \
  --protocol Tcp \
  --tcp-port "$destinationPort" \
  --frequency 60 \
  --threshold-failed-percent 5 \
  --threshold-round-trip-time 200 \
  --output-type Workspace \
  --workspace-ids "$workspaceId"

# Start monitor if needed.
az network watcher connection-monitor start \
  --location "$region" \
  --name "$connectionMonitorName"

# Show monitor.
az network watcher connection-monitor show \
  --location "$region" \
  --name "$connectionMonitorName" \
  --output json

# Query latest state.
az network watcher connection-monitor query \
  --location "$region" \
  --name "$connectionMonitorName" \
  --output json
```

# Troubleshoot_Azure_Networking_With_Network_Watcher_Connection_Monitor_And_Effective_Routes_Connection_Troubleshoot_Skeleton
```bash
# Purpose: run a point-in-time reachability test from a VM to an IP, FQDN, or URI.
# This is faster than creating continuous Connection Monitor when you need immediate triage.

subscriptionName="<subscription-name>"
sourceVmResourceGroup="<source-vm-resource-group>"
sourceVmName="<source-vm-name>"
destinationAddress="<destination-private-ip-or-fqdn>"
destinationPort="<destination-port>"

az account set \
  --subscription "$subscriptionName"

# Capture source VM ID.
sourceVmId=$(az vm show \
  --resource-group "$sourceVmResourceGroup" \
  --name "$sourceVmName" \
  --query id \
  --output tsv)

# TCP test.
az network watcher test-connectivity \
  --source-resource "$sourceVmId" \
  --dest-address "$destinationAddress" \
  --dest-port "$destinationPort" \
  --protocol Tcp \
  --output json

# Save output.
az network watcher test-connectivity \
  --source-resource "$sourceVmId" \
  --dest-address "$destinationAddress" \
  --dest-port "$destinationPort" \
  --protocol Tcp \
  --output json > connection-troubleshoot.json
```

# Troubleshoot_Azure_Networking_With_Network_Watcher_Connection_Monitor_And_Effective_Routes_IP_Flow_Verify_Skeleton
```bash
# Purpose: test whether a specific packet is allowed or denied by NSG evaluation.
# Use this before editing NSG rules.

subscriptionName="<subscription-name>"
sourceVmResourceGroup="<source-vm-resource-group>"
sourceVmName="<source-vm-name>"

direction="<direction>"                 # Inbound or Outbound
protocol="<protocol>"                   # TCP or UDP
localIp="<source-private-ip>"
localPort="<source-port>"
remoteIp="<destination-private-ip>"
remotePort="<destination-port>"

az account set \
  --subscription "$subscriptionName"

az network watcher test-ip-flow \
  --resource-group "$sourceVmResourceGroup" \
  --vm "$sourceVmName" \
  --direction "$direction" \
  --protocol "$protocol" \
  --local "$localIp:$localPort" \
  --remote "$remoteIp:$remotePort" \
  --output json

# Example outbound TCP 443 check.
# az network watcher test-ip-flow \
#   -g "$sourceVmResourceGroup" \
#   --vm "$sourceVmName" \
#   --direction Outbound \
#   --protocol TCP \
#   --local "$localIp:50000" \
#   --remote "$remoteIp:443" \
#   --output table
```

# Troubleshoot_Azure_Networking_With_Network_Watcher_Connection_Monitor_And_Effective_Routes_Effective_Routes_Skeleton
```bash
# Purpose: show final route evaluation on a NIC.
# Use this to prove whether UDRs, peering, BGP, or system routes are active.

subscriptionName="<subscription-name>"
resourceGroup="<resource-group>"
sourceNicName="<source-nic-name>"

az account set \
  --subscription "$subscriptionName"

# Show effective route table.
az network nic show-effective-route-table \
  --resource-group "$resourceGroup" \
  --name "$sourceNicName" \
  --output table

# Export JSON for evidence.
az network nic show-effective-route-table \
  --resource-group "$resourceGroup" \
  --name "$sourceNicName" \
  --output json > effective-routes-$sourceNicName.json

# Optional focused query.
az network nic show-effective-route-table \
  --resource-group "$resourceGroup" \
  --name "$sourceNicName" \
  --query "value[].{Name:name,State:state,Source:source,AddressPrefix:addressPrefix,nextHopType:nextHopType,nextHopIpAddress:nextHopIpAddress}" \
  --output table
```

# Troubleshoot_Azure_Networking_With_Network_Watcher_Connection_Monitor_And_Effective_Routes_Effective_Security_Rules_Skeleton
```bash
# Purpose: show final security rule evaluation on a NIC.
# Use this to distinguish subnet NSG, NIC NSG, and admin rules.

subscriptionName="<subscription-name>"
resourceGroup="<resource-group>"
sourceNicName="<source-nic-name>"

az account set \
  --subscription "$subscriptionName"

# Show effective NSG rules.
az network nic list-effective-nsg \
  --resource-group "$resourceGroup" \
  --name "$sourceNicName" \
  --output table

# Export JSON for evidence.
az network nic list-effective-nsg \
  --resource-group "$resourceGroup" \
  --name "$sourceNicName" \
  --output json > effective-security-rules-$sourceNicName.json
```

# Troubleshoot_Azure_Networking_With_Network_Watcher_Connection_Monitor_And_Effective_Routes_Next_Hop_Skeleton
```bash
# Purpose: confirm the next hop Azure selected for a destination IP.
# Use this when routing looks wrong or traffic should go through an NVA, peering, gateway, or internet.

subscriptionName="<subscription-name>"
sourceVmResourceGroup="<source-vm-resource-group>"
sourceVmName="<source-vm-name>"
sourceIp="<source-private-ip>"
destinationIp="<destination-private-ip>"

az account set \
  --subscription "$subscriptionName"

az network watcher show-next-hop \
  --resource-group "$sourceVmResourceGroup" \
  --vm "$sourceVmName" \
  --source-ip "$sourceIp" \
  --dest-ip "$destinationIp" \
  --output json

# Export evidence.
az network watcher show-next-hop \
  --resource-group "$sourceVmResourceGroup" \
  --vm "$sourceVmName" \
  --source-ip "$sourceIp" \
  --dest-ip "$destinationIp" \
  --output json > next-hop-$sourceVmName-to-$destinationIp.json
```

# Troubleshoot_Azure_Networking_With_Network_Watcher_Connection_Monitor_And_Effective_Routes_Flow_Logs_Skeleton
```bash
# Purpose: enable VNet flow logs for historical Layer 4 traffic evidence.
# Use this when you need traffic records, trend evidence, SIEM export, or proof over time.

subscriptionName="<subscription-name>"
resourceGroup="<resource-group>"
region="<region>"

vnetName="<vnet-name>"
storageAccountId="<storage-account-resource-id>"
workspaceId="<workspace-resource-id>"
flowLogName="<vnet-flow-log-name>"

az account set \
  --subscription "$subscriptionName"

# Capture VNet ID.
vnetId=$(az network vnet show \
  --resource-group "$resourceGroup" \
  --name "$vnetName" \
  --query id \
  --output tsv)

# Create VNet flow log.
# Confirm exact CLI extension support in your shell with:
# az network watcher flow-log -h
az network watcher flow-log create \
  --resource-group "$resourceGroup" \
  --name "$flowLogName" \
  --location "$region" \
  --target-resource "$vnetId" \
  --storage-account "$storageAccountId" \
  --enabled true \
  --retention 7 \
  --traffic-analytics true \
  --workspace "$workspaceId" \
  --interval 10

# Verify flow logs.
az network watcher flow-log show \
  --resource-group "$resourceGroup" \
  --name "$flowLogName" \
  --output json
```

# Troubleshoot_Azure_Networking_With_Network_Watcher_Connection_Monitor_And_Effective_Routes_Evidence_Export_Skeleton
```powershell
# Purpose: export troubleshooting evidence before and after remediation.

$ResourceGroup = "<resource-group>"
$SourceVmResourceGroup = "<source-vm-resource-group>"
$SourceVmName = "<source-vm-name>"
$SourceNicName = "<source-nic-name>"
$VNetName = "<vnet-name>"
$RouteTableName = "<route-table-name>"
$NsgName = "<nsg-name>"
$Region = "<region>"
$ConnectionMonitorName = "<connection-monitor-name>"
$DestinationAddress = "<destination-private-ip-or-fqdn>"
$DestinationPort = "<destination-port>"
$EvidencePath = "C:\Azure-NetworkWatcher-Troubleshooting\before"

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

$SourceVmId = az vm show `
  --resource-group $SourceVmResourceGroup `
  --name $SourceVmName `
  --query id `
  --output tsv

az account show `
  --output json |
  Out-File "$EvidencePath\account-context.json" -Encoding utf8

az network watcher list `
  --output json |
  Out-File "$EvidencePath\network-watchers.json" -Encoding utf8

az vm get-instance-view `
  --resource-group $SourceVmResourceGroup `
  --name $SourceVmName `
  --output json |
  Out-File "$EvidencePath\source-vm-instance-view.json" -Encoding utf8

az network nic show `
  --resource-group $ResourceGroup `
  --name $SourceNicName `
  --output json |
  Out-File "$EvidencePath\source-nic.json" -Encoding utf8

az network nic show-effective-route-table `
  --resource-group $ResourceGroup `
  --name $SourceNicName `
  --output json |
  Out-File "$EvidencePath\effective-routes.json" -Encoding utf8

az network nic list-effective-nsg `
  --resource-group $ResourceGroup `
  --name $SourceNicName `
  --output json |
  Out-File "$EvidencePath\effective-security-rules.json" -Encoding utf8

az network vnet show `
  --resource-group $ResourceGroup `
  --name $VNetName `
  --output json |
  Out-File "$EvidencePath\vnet.json" -Encoding utf8

az network vnet subnet list `
  --resource-group $ResourceGroup `
  --vnet-name $VNetName `
  --output json |
  Out-File "$EvidencePath\vnet-subnets.json" -Encoding utf8

az network route-table show `
  --resource-group $ResourceGroup `
  --name $RouteTableName `
  --output json |
  Out-File "$EvidencePath\route-table.json" -Encoding utf8

az network nsg show `
  --resource-group $ResourceGroup `
  --name $NsgName `
  --output json |
  Out-File "$EvidencePath\nsg.json" -Encoding utf8

az network nsg rule list `
  --resource-group $ResourceGroup `
  --nsg-name $NsgName `
  --output json |
  Out-File "$EvidencePath\nsg-rules.json" -Encoding utf8

az network watcher test-connectivity `
  --source-resource $SourceVmId `
  --dest-address $DestinationAddress `
  --dest-port $DestinationPort `
  --protocol Tcp `
  --output json |
  Out-File "$EvidencePath\connection-troubleshoot.json" -Encoding utf8

az network watcher connection-monitor show `
  --location $Region `
  --name $ConnectionMonitorName `
  --output json |
  Out-File "$EvidencePath\connection-monitor.json" -Encoding utf8

az network watcher connection-monitor query `
  --location $Region `
  --name $ConnectionMonitorName `
  --output json |
  Out-File "$EvidencePath\connection-monitor-query.json" -Encoding utf8
```

# Troubleshoot_Azure_Networking_With_Network_Watcher_Connection_Monitor_And_Effective_Routes_Verification_Commands
```bash
# Network Watcher.
az network watcher list -o table

# Source VM state.
az vm get-instance-view \
  --resource-group "<source-vm-resource-group>" \
  --name "<source-vm-name>" \
  --query "instanceView.statuses[].displayStatus" \
  --output table

# Source NIC private IP.
az network nic show \
  --resource-group "<resource-group>" \
  --name "<source-nic-name>" \
  --query "{Name:name,PrivateIps:ipConfigurations[].privateIPAddress,Subnet:ipConfigurations[].subnet.id}" \
  --output json

# Connection troubleshoot.
az network watcher test-connectivity \
  --source-resource "<source-vm-resource-id>" \
  --dest-address "<destination-private-ip-or-fqdn>" \
  --dest-port "<destination-port>" \
  --protocol Tcp \
  --output json

# IP flow verify.
az network watcher test-ip-flow \
  --resource-group "<source-vm-resource-group>" \
  --vm "<source-vm-name>" \
  --direction Outbound \
  --protocol TCP \
  --local "<source-private-ip>:<source-port>" \
  --remote "<destination-private-ip>:<destination-port>" \
  --output json

# Effective routes.
az network nic show-effective-route-table \
  --resource-group "<resource-group>" \
  --name "<source-nic-name>" \
  --output table

# Effective NSGs.
az network nic list-effective-nsg \
  --resource-group "<resource-group>" \
  --name "<source-nic-name>" \
  --output table

# Next hop.
az network watcher show-next-hop \
  --resource-group "<source-vm-resource-group>" \
  --vm "<source-vm-name>" \
  --source-ip "<source-private-ip>" \
  --dest-ip "<destination-private-ip>" \
  --output json

# Subnet route and NSG associations.
az network vnet subnet list \
  --resource-group "<resource-group>" \
  --vnet-name "<vnet-name>" \
  --query "[].{Name:name,Prefix:addressPrefix,Nsg:networkSecurityGroup.id,RouteTable:routeTable.id,ServiceEndpoints:serviceEndpoints}" \
  --output table

# Connection monitor.
az network watcher connection-monitor list \
  --location "<region>" \
  --output table

az network watcher connection-monitor query \
  --location "<region>" \
  --name "<connection-monitor-name>" \
  --output json
```

```powershell
# VM-side validation.

ipconfig /all
route print
Resolve-DnsName "<destination-fqdn>"

Test-NetConnection `
  -ComputerName "<destination-private-ip-or-fqdn>" `
  -Port <destination-port> `
  -InformationLevel Detailed

# Destination VM listener check.
Get-NetTCPConnection |
  Where-Object {
    $_.LocalPort -eq <destination-port>
  } |
  Select-Object LocalAddress, LocalPort, State, OwningProcess

# Windows firewall quick view.
Get-NetFirewallProfile |
  Select-Object Name, Enabled, DefaultInboundAction, DefaultOutboundAction
```

# Troubleshoot_Azure_Networking_With_Network_Watcher_Connection_Monitor_And_Effective_Routes_Rollback
```bash
# Stop a temporary Connection Monitor.
az network watcher connection-monitor stop \
  --location "<region>" \
  --name "<connection-monitor-name>"

# Delete a temporary Connection Monitor.
az network watcher connection-monitor delete \
  --location "<region>" \
  --name "<connection-monitor-name>"

# Delete temporary VNet flow log if created for lab troubleshooting.
az network watcher flow-log delete \
  --resource-group "<resource-group>" \
  --name "<vnet-flow-log-name>"

# Remove temporary NSG allow rule after permanent rule is designed.
az network nsg rule delete \
  --resource-group "<resource-group>" \
  --nsg-name "<nsg-name>" \
  --name "<temporary-rule-name>"

# Remove temporary UDR after permanent route is designed.
az network route-table route delete \
  --resource-group "<resource-group>" \
  --route-table-name "<route-table-name>" \
  --name "<temporary-route-name>"

# Reassociate original route table if changed during troubleshooting.
az network vnet subnet update \
  --resource-group "<resource-group>" \
  --vnet-name "<vnet-name>" \
  --name "<subnet-name>" \
  --route-table "<original-route-table-name>"

# Reassociate original NSG if changed during troubleshooting.
az network vnet subnet update \
  --resource-group "<resource-group>" \
  --vnet-name "<vnet-name>" \
  --name "<subnet-name>" \
  --network-security-group "<original-nsg-name>"
```

# Troubleshoot_Azure_Networking_With_Network_Watcher_Connection_Monitor_And_Effective_Routes_Failure_Checks
| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| Network Watcher command fails | Network Watcher not enabled in region | `az network watcher list -o table` | Enable Network Watcher in target region |
| Connection troubleshoot cannot start | Source VM ID is wrong or VM is stopped | `az vm get-instance-view` | Start VM or use correct resource ID |
| Connection troubleshoot says `DNSResolution` | FQDN does not resolve from source | `Resolve-DnsName <fqdn>` on source VM | Fix Private DNS zone, custom DNS, VNet link, or forwarding |
| Connection troubleshoot says `NetworkSecurityRule` | NSG or admin rule blocks flow | IP flow verify and effective security rules | Add or correct the exact NSG rule |
| Connection troubleshoot says `UserDefinedRoute` | UDR misroutes or drops flow | Effective routes and next hop | Correct route prefix, next hop type, or route table association |
| Connection troubleshoot says `GuestFirewall` | OS firewall blocks traffic | Destination VM firewall check | Allow required port in guest firewall |
| Connection troubleshoot says `NoListenerOnDestination` | App is not listening on tested port | `netstat`, `ss`, `Test-NetConnection localhost` | Start service or use correct port |
| Connection troubleshoot says `RouteMissing` | No valid route exists | Effective routes | Add peering, gateway, route table, or fix address plan |
| Connection troubleshoot says `UDRLoop` | Next hop loops back into same path | Next hop and route table review | Fix next hop IP or route prefix |
| Connection troubleshoot says `IPForwardingNotEnabled` | NVA NIC does not allow forwarding | NIC `enableIPForwarding` | Enable IP forwarding on NVA NIC |
| IP flow verify says Deny | NSG rule denies packet | Matched rule output | Change the correct rule, not random rules |
| IP flow verify says Allow but connection fails | Route, guest firewall, or listener issue | Next hop, Test-NetConnection, service check | Continue triage beyond NSG |
| Effective routes do not show UDR | Route table not associated to source subnet | Subnet route table field | Associate route table to correct subnet |
| Effective routes show wrong next hop | More specific prefix or BGP route wins | Effective routes sorted by prefix | Adjust prefix or route propagation |
| Next hop is Internet unexpectedly | Missing private route, peering, or private DNS resolution | Next hop and DNS result | Fix route or DNS target |
| Next hop is None | Blackhole route exists | Effective routes | Remove or change drop route |
| Connection Monitor shows indeterminate | Source endpoint unavailable or deallocated | VM state and monitor source endpoint | Start VM or update monitor endpoint |
| Connection Monitor missing data | Agent, workspace, or endpoint issue | Monitor query and VM extension state | Fix extension or recreate monitor |
| Flow logs show no traffic | Wrong target resource or no traffic generated | Flow log target and test traffic | Generate traffic and verify target |
| Flow logs duplicate cost | NSG flow logs and VNet flow logs overlap | Flow log inventory | Avoid duplicate logging on same workload |
| Diagnostics point to multiple issues | More than one layer is broken | Work through triage order | Fix one proven failure layer at a time |
| Fix does not work after change | Old connection state or cached DNS | New TCP test, flush DNS, retry | Test fresh connection after cache clear |

# Troubleshoot_Azure_Networking_With_Network_Watcher_Connection_Monitor_And_Effective_Routes_Related_Labs
| Lab | Relationship |
|---|---|
| `01_Create_VNets_Subnets_Public_IPs_And_Address_Space_Baseline.md` | Provides the VNet, subnet, and IP foundation that diagnostics inspect |
| `02_Configure_VNet_Peering_DNS_Resolution_And_Private_DNS_Zones.md` | Supplies peering and Private DNS dependencies for cross-VNet troubleshooting |
| `03_Configure_NSGs_ASGs_Effective_Security_Rules_And_Service_Endpoints.md` | Supplies NSG, ASG, and service endpoint behavior inspected by IP flow verify and effective security rules |
| `04_Configure_Private_Endpoints_Private_Link_And_PaaS_Network_Access.md` | Provides private endpoint paths that depend on DNS, routes, and NSGs |
| `05_Configure_UDRs_Route_Tables_Azure_DNS_And_Name_Resolution.md` | Supplies UDR and DNS design that effective routes and next hop validate |
| `06_Configure_Azure_Load_Balancer_Bastion_And_Basic_Traffic_Distribution.md` | Supplies load balancer and Bastion paths that connection troubleshoot can validate |
| `08_Troubleshoot_Private_Endpoint_DNS_Routing_NSG_And_Load_Balancer_Issues.md` | Extends this workbook into combined failure scenarios across Private Endpoint, DNS, routing, NSG, and Load Balancer |