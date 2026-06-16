# 04_Troubleshoot_Azure_Network_Storage_Compute_Monitor_And_Backup_Issues

## 04_Troubleshoot_Azure_Network_Storage_Compute_Monitor_And_Backup_Issues_Index

### Purpose

This workbook provides a structured troubleshooting workflow for Azure operational issues across networking, storage, compute, monitoring, alerts, diagnostics, backup, and restore readiness.

### Scope

Use this workbook when Azure resources show one or more of the following symptoms:

- VM cannot reach another VM, private endpoint, storage account, database, or internet
- NSG, route table, DNS, VNet peering, private endpoint, or firewall behavior blocks traffic
- Storage account access fails with authentication, authorization, firewall, private endpoint, DNS, or performance errors
- VM fails to start, stop, resize, boot, connect by RDP/SSH, or run extensions
- App or workload is reachable internally but not externally
- Azure Monitor metrics, logs, diagnostic settings, alerts, or action groups are missing or not firing
- Log Analytics queries return no data or stale data
- Backup fails, restore point missing, Recovery Services Vault unhealthy, or policy not applying
- Snapshot, soft delete, restore, or vault access behavior does not match expectations
- Issue crosses network, compute, storage, and monitoring at the same time

### Assumptions

- You have at least Reader access to the target subscription and resource group.
- You can access Azure portal, Azure CLI, Azure PowerShell, Network Watcher, Azure Monitor, and Recovery Services Vault where applicable.
- You know the target subscription, resource group, region, and affected resource name.
- You will collect evidence before deleting, redeploying, resizing, or restoring resources.
- You will not bypass security controls without an approved incident or change record.

### Required Admin Roles

| Task Area | Minimum Role |
|---|---|
| View resources | Reader |
| View network topology and diagnostics | Network Contributor or Reader with Network Watcher access |
| Modify NSG, UDR, VNet, peering, DNS | Network Contributor |
| View storage account config | Reader |
| Modify storage networking/auth settings | Storage Account Contributor |
| Access storage data | Storage Blob Data Reader, Storage Blob Data Contributor, Storage File Data SMB Share roles, or account key/SAS path |
| Manage virtual machines | Virtual Machine Contributor |
| Run VM commands/extensions | Virtual Machine Contributor |
| Modify disks | Disk Contributor or VM Contributor |
| View metrics and logs | Monitoring Reader |
| Create alerts and diagnostics | Monitoring Contributor |
| Manage Log Analytics workspace | Log Analytics Contributor |
| Manage backup vaults and backup items | Backup Contributor |
| Restore from backup | Backup Operator, Backup Contributor, or vault-specific permissions |

---

## 04_Troubleshoot_Azure_Network_Storage_Compute_Monitor_And_Backup_Issues_Mental_Model

Most Azure operational failures are dependency failures. A VM, storage account, private endpoint, backup job, or alert rarely fails in isolation.

Use this dependency chain:

1. Subscription and region are healthy.
2. Resource exists and is in the expected provisioning state.
3. Identity and RBAC allow the intended action.
4. Network path allows the traffic.
5. DNS resolves the expected endpoint.
6. Firewall, NSG, UDR, service endpoint, private endpoint, or platform rule allows access.
7. Resource configuration allows the operation.
8. Guest OS, agent, extension, workload, or service inside the resource is healthy.
9. Monitoring pipeline is collecting the correct signal.
10. Backup policy is assigned and jobs are succeeding.
11. Restore points exist and are usable.

Traditional troubleshooting order:

1. Confirm scope and symptom.
2. Confirm resource state.
3. Check platform health.
4. Check recent changes.
5. Check network path and DNS.
6. Check identity and access.
7. Check resource configuration.
8. Check logs, metrics, and activity events.
9. Check backup health and restore point availability.
10. Fix the smallest confirmed blocker.
11. Verify from the same source and destination.
12. Record root cause, evidence, fix, and rollback.

---

## 04_Troubleshoot_Azure_Network_Storage_Compute_Monitor_And_Backup_Issues_Symptom_Map

| Symptom | Likely Cause | Primary Evidence | First Check |
|---|---|---|---|
| VM cannot reach another VM | NSG, UDR, firewall, wrong subnet, guest firewall | Network Watcher IP flow verify | Effective security rules |
| VM cannot reach internet | UDR to NVA, NAT missing, NSG deny, no public path | Effective routes | Route table and outbound path |
| VM cannot resolve private endpoint | Private DNS missing or wrong VNet link | DNS query from VM | Private DNS zone link |
| Private endpoint connects but service fails | DNS still resolves public endpoint, firewall, approval pending | Private endpoint connection state | DNS resolution |
| Storage account access denied | RBAC missing, key disabled, SAS expired, firewall blocked | Storage error and activity log | Storage network/auth settings |
| Storage account unreachable | Firewall, private endpoint DNS, service endpoint, NSG | Connectivity test | DNS and firewall |
| Blob upload/download slow | Large objects, tier, network path, throttling, client issue | Metrics and client logs | Storage metrics |
| VM cannot start | Allocation failure, quota, disk issue, host issue, policy | VM instance view and activity log | VM status |
| VM cannot RDP/SSH | NSG, public IP, Bastion, guest firewall, service down | Connection troubleshoot | NSG and boot diagnostics |
| VM extension fails | Agent unhealthy, bad script, no outbound access, identity issue | Extension status | VM agent status |
| Monitor logs missing | Diagnostic setting absent, wrong workspace, ingestion delay | Diagnostic settings | Workspace and table |
| Alert did not fire | Wrong signal, threshold, scope, action group, evaluation frequency | Alert rule history | Alert processing |
| Backup job failed | Agent, snapshot, vault policy, storage, network, permissions | Backup job details | Vault backup jobs |
| Restore point missing | Policy not assigned, backup never completed, retention expired | Backup item recovery points | Backup item state |
| Backup protected item unhealthy | VM agent, extension, vault communication, snapshot issue | Backup health | Backup item details |
| Resource changed unexpectedly | Manual change, deployment, policy remediation, automation | Activity Log | Change event |

---

## 04_Troubleshoot_Azure_Network_Storage_Compute_Monitor_And_Backup_Issues_Evidence_Collection

| Evidence | Where To Get It | Why It Matters |
|---|---|---|
| Subscription ID | Azure portal / CLI | Confirms target environment |
| Resource group | Azure portal / CLI | Limits search scope |
| Region | Resource overview | Region affects availability and service health |
| Affected resource ID | Resource overview / CLI | Needed for logs, RBAC, and escalation |
| Error message | Portal, CLI, app, job, or client | Root-cause signal |
| Timestamp with timezone | User report / logs | Required for Activity Log and metrics |
| Source resource | VM, subnet, client, app, pipeline | Needed for network path |
| Destination resource | VM, storage, endpoint, service | Needed for connectivity test |
| Protocol and port | User report / app config | Needed for NSG/firewall checks |
| DNS name and resolved IP | VM/client test | Confirms endpoint resolution |
| Effective routes | Network Watcher | Confirms traffic path |
| Effective security rules | Network Watcher | Confirms NSG allow/deny |
| Private endpoint state | Private endpoint blade | Confirms approval and NIC state |
| Storage network settings | Storage account networking | Confirms public/private access |
| VM instance view | VM diagnostics | Confirms power and guest agent state |
| Boot diagnostics screenshot | VM support blade | Confirms guest boot issue |
| Activity Log | Azure Monitor | Captures changes, denies, failures |
| Metrics | Azure Monitor | Captures performance and availability |
| Diagnostic settings | Resource monitoring blade | Confirms log export |
| Log Analytics workspace | Azure Monitor Logs | Confirms data destination |
| Alert rule history | Azure Monitor alerts | Confirms evaluation behavior |
| Backup job details | Recovery Services Vault | Confirms backup failure reason |
| Recovery points | Backup item blade | Confirms restore availability |

---

## 04_Troubleshoot_Azure_Network_Storage_Compute_Monitor_And_Backup_Issues_Triage_Order

1. Confirm exact symptom, affected resource, source, destination, protocol, port, and timestamp.
2. Confirm subscription, resource group, region, and resource ID.
3. Check Azure Service Health and Resource Health.
4. Check Activity Log for recent changes or platform errors.
5. For network issues, check DNS, IP flow, effective security rules, effective routes, and connection troubleshoot.
6. For storage issues, check auth method, firewall, private endpoint, DNS, RBAC, metrics, and logs.
7. For compute issues, check power state, instance view, boot diagnostics, serial console, VM agent, extensions, disks, and NSG.
8. For monitoring issues, check diagnostic settings, workspace, tables, metrics, alert rule, action group, and processing rules.
9. For backup issues, check vault, policy, protected item state, job error, extension, snapshots, and recovery points.
10. Apply the smallest confirmed remediation.
11. Verify from the original source.
12. Record root cause, evidence, fix, and rollback path.

---

## 04_Troubleshoot_Azure_Network_Storage_Compute_Monitor_And_Backup_Issues_Planning_Table

| Item | Value |
|---|---|
| Tenant ID | `<tenant-id>` |
| Subscription name | `<subscription-name>` |
| Subscription ID | `<subscription-id>` |
| Resource group | `<resource-group>` |
| Region | `<region>` |
| Affected resource name | `<resource-name>` |
| Affected resource ID | `<resource-id>` |
| Resource type | `<Microsoft.Provider/type>` |
| Symptom category | `<Network / Storage / Compute / Monitor / Backup>` |
| Source resource/client | `<source>` |
| Destination resource/service | `<destination>` |
| Protocol | `<tcp / udp / icmp / https / smb / nfs / rdp / ssh>` |
| Port | `<port>` |
| DNS name | `<fqdn>` |
| Expected IP | `<ip>` |
| Actual resolved IP | `<ip>` |
| First failure time | `<yyyy-mm-dd hh:mm timezone>` |
| Recent change | `<yes-no>` |
| Change ticket | `<ticket-id>` |
| Backup vault | `<vault-name>` |
| Log Analytics workspace | `<workspace-name>` |
| Alert rule | `<alert-rule-name>` |
| Business impact | `<impact>` |

---

## 04_Troubleshoot_Azure_Network_Storage_Compute_Monitor_And_Backup_Issues_Configuration_Checklist

| Step | Task | Device | Command | PowerShell | Expected Result |
|---|---|---|---|---|---|
| 1 | Confirm Azure context | Admin workstation | `az account show --output table` | `Get-AzContext` | Correct tenant and subscription are selected |
| 2 | Set target subscription | Admin workstation | `az account set --subscription <subscription-id>` | `Set-AzContext -Subscription <subscription-id>` | Commands target correct subscription |
| 3 | Confirm resource exists | Admin workstation | `az resource show --ids <resource-id>` | `Get-AzResource -ResourceId <resource-id>` | Resource exists and is visible |
| 4 | Check resource health | Admin workstation | Azure portal > Resource > Resource health | `Get-AzResource -ResourceId <resource-id>` | Resource health is available, degraded, or unavailable |
| 5 | Check Activity Log | Admin workstation | Azure portal > Monitor > Activity log | `Get-AzActivityLog -ResourceGroupName <resource-group> -StartTime (Get-Date).AddHours(-24)` | Recent change/failure events are identified |
| 6 | Check Azure Service Health | Admin workstation | Azure portal > Service Health | Not applicable | Platform incident is ruled in or out |
| 7 | Identify source and destination | Admin workstation | Record source IP, destination IP/FQDN, protocol, and port | Not applicable | Connectivity test can be scoped correctly |
| 8 | Test DNS from source | Source VM/client | `nslookup <fqdn>` | `Resolve-DnsName <fqdn>` | FQDN resolves to expected IP |
| 9 | Test TCP connectivity from source | Source VM/client | `Test-NetConnection <fqdn-or-ip> -Port <port>` | `Test-NetConnection <fqdn-or-ip> -Port <port>` | TCP path succeeds or fails clearly |
| 10 | Run IP flow verify | Admin workstation | Network Watcher > IP flow verify | `Test-AzNetworkWatcherIPFlow -NetworkWatcher <nw> -TargetVirtualMachineId <vm-id> -Direction Outbound -Protocol TCP -LocalIPAddress <source-ip> -RemoteIPAddress <dest-ip> -LocalPort <local-port> -RemotePort <dest-port>` | NSG allow/deny decision is known |
| 11 | Check effective security rules | Admin workstation | VM NIC > Effective security rules | `Get-AzEffectiveNetworkSecurityGroup -NetworkInterfaceName <nic-name> -ResourceGroupName <resource-group>` | Applied NSG rules are visible |
| 12 | Check effective routes | Admin workstation | VM NIC > Effective routes | `Get-AzEffectiveRouteTable -NetworkInterfaceName <nic-name> -ResourceGroupName <resource-group>` | Route path is visible |
| 13 | Check VNet peering | Admin workstation | VNet > Peerings | `Get-AzVirtualNetworkPeering -VirtualNetworkName <vnet-name> -ResourceGroupName <resource-group>` | Peering is connected and settings are correct |
| 14 | Check private DNS zones | Admin workstation | Private DNS zones > Virtual network links | `Get-AzPrivateDnsVirtualNetworkLink -ZoneName <zone-name> -ResourceGroupName <dns-rg>` | VNet is linked to private DNS zone |
| 15 | Check private endpoint state | Admin workstation | Private endpoint > Connections | `Get-AzPrivateEndpoint -Name <pe-name> -ResourceGroupName <resource-group>` | Private endpoint is approved and healthy |
| 16 | Check storage account network rules | Admin workstation | Storage account > Networking | `Get-AzStorageAccountNetworkRuleSet -ResourceGroupName <resource-group> -Name <storage-account>` | Public access, firewall, bypass, and private endpoint settings are known |
| 17 | Check storage RBAC | Admin workstation | Storage account > Access control IAM | `Get-AzRoleAssignment -Scope <storage-account-resource-id>` | Required management/data role exists |
| 18 | Check storage metrics | Admin workstation | Storage account > Monitoring > Metrics | Not applicable | Availability, latency, transactions, and errors are visible |
| 19 | Check VM power and instance view | Admin workstation | VM > Overview and VM > Support + troubleshooting | `Get-AzVM -ResourceGroupName <resource-group> -Name <vm-name> -Status` | VM power state and agent status are known |
| 20 | Check VM boot diagnostics | Admin workstation | VM > Boot diagnostics | Not applicable | Screenshot or serial log confirms boot state |
| 21 | Check VM extensions | Admin workstation | VM > Extensions + applications | `Get-AzVMExtension -ResourceGroupName <resource-group> -VMName <vm-name>` | Extension state and errors are visible |
| 22 | Check VM serial console if needed | Admin workstation | VM > Serial console | Not applicable | Guest OS can be inspected when network login fails |
| 23 | Check diagnostic settings | Admin workstation | Resource > Diagnostic settings | `Get-AzDiagnosticSetting -ResourceId <resource-id>` | Logs/metrics are routed to expected destination |
| 24 | Check Log Analytics data | Admin workstation | Azure Monitor > Logs | `Search-AzGraph -Query "resources | where id =~ '<resource-id>'"` | Workspace and data path are confirmed |
| 25 | Check alert rule | Admin workstation | Azure Monitor > Alerts > Alert rules | `Get-AzScheduledQueryRule -ResourceGroupName <resource-group>` | Alert scope, condition, frequency, and action group are known |
| 26 | Check action group | Admin workstation | Azure Monitor > Alerts > Action groups | `Get-AzActionGroup -ResourceGroupName <resource-group>` | Notification path is correct |
| 27 | Check backup vault | Admin workstation | Recovery Services Vault > Backup items | `Get-AzRecoveryServicesVault -Name <vault-name>` | Vault exists and is accessible |
| 28 | Check backup jobs | Admin workstation | Recovery Services Vault > Backup jobs | `Get-AzRecoveryServicesBackupJob -VaultId <vault-id> -From (Get-Date).AddDays(-7) -To (Get-Date)` | Backup failures are visible |
| 29 | Check recovery points | Admin workstation | Backup item > Restore points | `Get-AzRecoveryServicesBackupRecoveryPoint -Item <backup-item> -StartDate (Get-Date).AddDays(-30) -EndDate (Get-Date)` | Restore points exist |
| 30 | Apply smallest confirmed fix | Admin workstation | Change only the confirmed blocker | Depends on fix | Issue is corrected without broad side effects |
| 31 | Verify from original source | Source VM/client | Repeat original failed operation | Repeat original failed command | Original symptom is resolved |
| 32 | Record evidence and rollback | Admin workstation | Update incident/change record | Not applicable | Troubleshooting record is complete |

---

## 04_Troubleshoot_Azure_Network_Storage_Compute_Monitor_And_Backup_Issues_Network_Workflow

### Phase 1: Network Path Isolation

| Step | Task | Device | Command | PowerShell | Expected Result |
|---|---|---|---|---|---|
| 1 | Confirm source IP | Source VM/client | `ipconfig` or `ip addr` | `Get-NetIPAddress` | Source IP is known |
| 2 | Confirm destination IP/FQDN | Source VM/client | `nslookup <fqdn>` | `Resolve-DnsName <fqdn>` | Destination resolves |
| 3 | Test TCP port | Source VM/client | `Test-NetConnection <destination> -Port <port>` | `Test-NetConnection <destination> -Port <port>` | TCP success or failure is clear |
| 4 | Trace route | Source VM/client | `tracert <destination>` or `traceroute <destination>` | `Test-NetConnection <destination> -TraceRoute` | Routing path or stop point is visible |
| 5 | Check IP flow verify | Admin workstation | Network Watcher > IP flow verify | `Test-AzNetworkWatcherIPFlow ...` | NSG allow/deny is known |
| 6 | Check effective NSG rules | Admin workstation | NIC > Effective security rules | `Get-AzEffectiveNetworkSecurityGroup -NetworkInterfaceName <nic-name> -ResourceGroupName <resource-group>` | Applied security rules are known |
| 7 | Check effective routes | Admin workstation | NIC > Effective routes | `Get-AzEffectiveRouteTable -NetworkInterfaceName <nic-name> -ResourceGroupName <resource-group>` | UDR/BGP/system route path is known |
| 8 | Check route table association | Admin workstation | Subnet > Route table | `Get-AzVirtualNetworkSubnetConfig -VirtualNetwork <vnet>` | Correct route table is associated |
| 9 | Check NSG association | Admin workstation | NIC and subnet NSG settings | `Get-AzNetworkInterface -Name <nic-name> -ResourceGroupName <resource-group>` | NSG is applied where expected |
| 10 | Check Azure Firewall/NVA path | Admin workstation | Route table next hop and firewall logs | Not applicable | Middlebox path is confirmed |

### Phase 2: VNet, Peering, DNS, And Private Endpoint

| Step | Task | Device | Command | PowerShell | Expected Result |
|---|---|---|---|---|---|
| 1 | Check VNet address space overlap | Admin workstation | VNet > Address space | `Get-AzVirtualNetwork -Name <vnet-name> -ResourceGroupName <resource-group>` | No unexpected overlap |
| 2 | Check subnet route table | Admin workstation | VNet > Subnets > target subnet | `Get-AzVirtualNetworkSubnetConfig -Name <subnet-name> -VirtualNetwork <vnet>` | Subnet config is known |
| 3 | Check VNet peering status | Admin workstation | VNet > Peerings | `Get-AzVirtualNetworkPeering -VirtualNetworkName <vnet-name> -ResourceGroupName <resource-group>` | Peering state is Connected |
| 4 | Check peering allow forwarded traffic | Admin workstation | Peering settings | Review peering object | Forwarded traffic setting matches design |
| 5 | Check gateway transit settings | Admin workstation | Peering settings | Review peering object | Gateway transit settings match design |
| 6 | Check private DNS resolution | Source VM | `nslookup <private-endpoint-fqdn>` | `Resolve-DnsName <private-endpoint-fqdn>` | FQDN resolves to private IP |
| 7 | Check private DNS zone records | Admin workstation | Private DNS zone > Recordsets | `Get-AzPrivateDnsRecordSet -ZoneName <zone-name> -ResourceGroupName <dns-rg>` | A record exists for private endpoint |
| 8 | Check private DNS VNet links | Admin workstation | Private DNS zone > Virtual network links | `Get-AzPrivateDnsVirtualNetworkLink -ZoneName <zone-name> -ResourceGroupName <dns-rg>` | Source VNet is linked |
| 9 | Check private endpoint approval | Admin workstation | Private endpoint > Connections | `Get-AzPrivateEndpoint -Name <pe-name> -ResourceGroupName <resource-group>` | Connection is Approved |
| 10 | Check private endpoint NIC | Admin workstation | Private endpoint > Network interface | `Get-AzNetworkInterface -ResourceId <pe-nic-id>` | NIC has expected private IP |

---

## 04_Troubleshoot_Azure_Network_Storage_Compute_Monitor_And_Backup_Issues_Storage_Workflow

### Phase 1: Storage Access Isolation

| Step | Task | Device | Command | PowerShell | Expected Result |
|---|---|---|---|---|---|
| 1 | Confirm storage account exists | Admin workstation | `az storage account show -g <resource-group> -n <storage-account>` | `Get-AzStorageAccount -ResourceGroupName <resource-group> -Name <storage-account>` | Storage account exists |
| 2 | Check storage account provisioning state | Admin workstation | `az storage account show -g <resource-group> -n <storage-account> --query provisioningState` | `Get-AzStorageAccount -ResourceGroupName <resource-group> -Name <storage-account>` | State is Succeeded |
| 3 | Check public network access | Admin workstation | Storage account > Networking | `Get-AzStorageAccount -ResourceGroupName <resource-group> -Name <storage-account> | Select-Object PublicNetworkAccess` | Public network setting is known |
| 4 | Check network rule set | Admin workstation | Storage account > Networking | `Get-AzStorageAccountNetworkRuleSet -ResourceGroupName <resource-group> -Name <storage-account>` | Firewall rules are known |
| 5 | Check private endpoints | Admin workstation | Storage account > Networking > Private endpoint connections | `Get-AzPrivateEndpointConnection -PrivateLinkResourceId <storage-resource-id>` | Private endpoint connection state is known |
| 6 | Check DNS for blob endpoint | Source VM/client | `nslookup <storage-account>.blob.core.windows.net` | `Resolve-DnsName <storage-account>.blob.core.windows.net` | Endpoint resolves to expected public or private IP |
| 7 | Check DNS for file endpoint | Source VM/client | `nslookup <storage-account>.file.core.windows.net` | `Resolve-DnsName <storage-account>.file.core.windows.net` | Endpoint resolves to expected public or private IP |
| 8 | Test HTTPS to blob endpoint | Source VM/client | `Test-NetConnection <storage-account>.blob.core.windows.net -Port 443` | `Test-NetConnection <storage-account>.blob.core.windows.net -Port 443` | TCP 443 succeeds |
| 9 | Test SMB to file endpoint | Source VM/client | `Test-NetConnection <storage-account>.file.core.windows.net -Port 445` | `Test-NetConnection <storage-account>.file.core.windows.net -Port 445` | TCP 445 succeeds if Azure Files SMB is used |
| 10 | Check RBAC assignment | Admin workstation | Storage account > IAM | `Get-AzRoleAssignment -Scope <storage-resource-id>` | Required data or management role exists |

### Phase 2: Storage Performance And Data Protection

| Step | Task | Device | Command | PowerShell | Expected Result |
|---|---|---|---|---|---|
| 1 | Check storage metrics | Admin workstation | Storage account > Metrics | Not applicable | Latency, availability, ingress, egress, and errors are visible |
| 2 | Check transaction errors | Admin workstation | Metrics > Transactions by response type | Not applicable | Throttling/auth/network errors are visible |
| 3 | Check diagnostic settings | Admin workstation | Storage account > Diagnostic settings | `Get-AzDiagnosticSetting -ResourceId <storage-resource-id>` | Logs are sent to expected workspace |
| 4 | Check blob soft delete | Admin workstation | Storage account > Data protection | `Get-AzStorageBlobServiceProperty -Context <ctx>` | Soft delete state is known |
| 5 | Check container soft delete | Admin workstation | Storage account > Data protection | Review blob service properties | Retention state is known |
| 6 | Check versioning | Admin workstation | Storage account > Data protection | Review blob service properties | Versioning state is known |
| 7 | Check lifecycle rules | Admin workstation | Storage account > Lifecycle management | `Get-AzStorageAccountManagementPolicy -ResourceGroupName <resource-group> -StorageAccountName <storage-account>` | Lifecycle policy is known |
| 8 | Check access keys state | Admin workstation | Storage account > Access keys | `Get-AzStorageAccount -ResourceGroupName <resource-group> -Name <storage-account> | Select-Object AllowSharedKeyAccess` | Shared key setting is known |
| 9 | Check SAS validity if used | Client/app config | Review SAS start, expiry, permissions, IP, protocol | Not applicable | SAS is valid or expired |
| 10 | Verify data operation | Source VM/client | Use Storage Explorer, AzCopy, app test, or CLI | Not applicable | Data path succeeds |

---

## 04_Troubleshoot_Azure_Network_Storage_Compute_Monitor_And_Backup_Issues_Compute_Workflow

### Phase 1: VM Availability And Access

| Step | Task | Device | Command | PowerShell | Expected Result |
|---|---|---|---|---|---|
| 1 | Check VM status | Admin workstation | `az vm get-instance-view -g <resource-group> -n <vm-name> --output table` | `Get-AzVM -ResourceGroupName <resource-group> -Name <vm-name> -Status` | Power and provisioning state are known |
| 2 | Check Resource Health | Admin workstation | VM > Resource health | Not applicable | Platform health is known |
| 3 | Check boot diagnostics | Admin workstation | VM > Boot diagnostics | Not applicable | Boot screenshot/log is visible |
| 4 | Check serial console | Admin workstation | VM > Serial console | Not applicable | Guest can be inspected if enabled |
| 5 | Check VM agent | Admin workstation | VM > Properties or Extensions | `Get-AzVM -ResourceGroupName <resource-group> -Name <vm-name> -Status` | VM agent status is ready or unhealthy |
| 6 | Check RDP/SSH NSG | Admin workstation | Network Watcher IP flow verify | `Test-AzNetworkWatcherIPFlow ...` | Access port is allowed or denied |
| 7 | Test RDP port | Client | `Test-NetConnection <vm-ip-or-fqdn> -Port 3389` | `Test-NetConnection <vm-ip-or-fqdn> -Port 3389` | RDP port is reachable or blocked |
| 8 | Test SSH port | Client | `Test-NetConnection <vm-ip-or-fqdn> -Port 22` | `Test-NetConnection <vm-ip-or-fqdn> -Port 22` | SSH port is reachable or blocked |
| 9 | Reset password or SSH key if needed | Admin workstation | VM > Reset password | `Set-AzVMAccessExtension ...` | Access credential is reset |
| 10 | Restart VM only after evidence capture | Admin workstation | `az vm restart -g <resource-group> -n <vm-name>` | `Restart-AzVM -ResourceGroupName <resource-group> -Name <vm-name>` | VM restarts cleanly |

### Phase 2: VM Extensions, Disks, And Performance

| Step | Task | Device | Command | PowerShell | Expected Result |
|---|---|---|---|---|---|
| 1 | List extensions | Admin workstation | `az vm extension list -g <resource-group> --vm-name <vm-name> --output table` | `Get-AzVMExtension -ResourceGroupName <resource-group> -VMName <vm-name>` | Extension states are visible |
| 2 | Show failed extension | Admin workstation | `az vm extension show -g <resource-group> --vm-name <vm-name> -n <extension-name>` | `Get-AzVMExtension -ResourceGroupName <resource-group> -VMName <vm-name> -Name <extension-name>` | Extension error is visible |
| 3 | Check OS disk | Admin workstation | VM > Disks | `Get-AzDisk -ResourceGroupName <resource-group> -DiskName <disk-name>` | Disk state and SKU are known |
| 4 | Check data disks | Admin workstation | VM > Disks | `Get-AzVM -ResourceGroupName <resource-group> -Name <vm-name>` | Disk attachments are known |
| 5 | Check VM metrics | Admin workstation | VM > Metrics | Not applicable | CPU, disk, network metrics are visible |
| 6 | Run command inside VM | Admin workstation | VM > Run command | `Invoke-AzVMRunCommand -ResourceGroupName <resource-group> -VMName <vm-name> -CommandId RunPowerShellScript -ScriptString "hostname"` | Guest command succeeds |
| 7 | Check Windows event logs | VM | Event Viewer | `Get-EventLog -LogName System -Newest 50` | Guest OS errors are visible |
| 8 | Check Linux system logs | VM | `journalctl -xe` | Not applicable | Guest OS errors are visible |
| 9 | Resize only after quota and SKU check | Admin workstation | VM > Size | `Update-AzVM` with approved size change | VM size changes successfully |
| 10 | Redeploy VM if host issue suspected | Admin workstation | VM > Redeploy | `Set-AzVM -Redeploy -ResourceGroupName <resource-group> -Name <vm-name>` | VM moves to new host |

---

## 04_Troubleshoot_Azure_Network_Storage_Compute_Monitor_And_Backup_Issues_Monitor_Workflow

### Phase 1: Metrics, Logs, And Diagnostic Settings

| Step | Task | Device | Command | PowerShell | Expected Result |
|---|---|---|---|---|---|
| 1 | Confirm resource emits metric | Admin workstation | Resource > Metrics | Not applicable | Metric namespace and metric exist |
| 2 | Check diagnostic settings | Admin workstation | Resource > Diagnostic settings | `Get-AzDiagnosticSetting -ResourceId <resource-id>` | Logs are routed to expected workspace/storage/event hub |
| 3 | Confirm workspace | Admin workstation | Log Analytics workspace > Overview | `Get-AzOperationalInsightsWorkspace -ResourceGroupName <resource-group>` | Correct workspace is identified |
| 4 | Check ingestion delay | Admin workstation | Azure Monitor > Logs | Not applicable | Recent data delay is understood |
| 5 | Query heartbeat for VM | Admin workstation | Logs workspace | `Heartbeat | where Computer =~ "<vm-name>" | sort by TimeGenerated desc | take 20` | VM agent heartbeat is present |
| 6 | Query Azure Activity | Admin workstation | Logs workspace | `AzureActivity | where ResourceGroup =~ "<resource-group>" | sort by TimeGenerated desc | take 50` | Activity data is present |
| 7 | Query storage logs | Admin workstation | Logs workspace | `StorageBlobLogs | where AccountName =~ "<storage-account>" | sort by TimeGenerated desc | take 50` | Storage diagnostics are present if enabled |
| 8 | Query VM perf counters | Admin workstation | Logs workspace | `Perf | where Computer =~ "<vm-name>" | sort by TimeGenerated desc | take 50` | Perf data is present if agent configured |
| 9 | Fix missing diagnostics | Admin workstation | Add diagnostic setting to resource | `New-AzDiagnosticSetting` | Logs begin flowing after ingestion delay |
| 10 | Verify after delay | Admin workstation | Run KQL again | Run KQL again | Expected table receives records |

### Phase 2: Alerts And Action Groups

| Step | Task | Device | Command | PowerShell | Expected Result |
|---|---|---|---|---|---|
| 1 | Check alert rule scope | Admin workstation | Azure Monitor > Alerts > Alert rules | Review alert rule | Scope includes affected resource |
| 2 | Check alert signal | Admin workstation | Alert rule > Condition | Review alert rule | Metric/log signal matches issue |
| 3 | Check threshold | Admin workstation | Alert rule > Condition | Review alert rule | Threshold can actually be crossed |
| 4 | Check evaluation frequency | Admin workstation | Alert rule > Details | Review alert rule | Frequency/window match expectation |
| 5 | Check action group | Admin workstation | Alert rule > Actions | `Get-AzActionGroup -ResourceGroupName <resource-group>` | Action group exists and is enabled |
| 6 | Check alert processing rules | Admin workstation | Azure Monitor > Alerts > Alert processing rules | Not applicable | Suppression rule is not muting alert |
| 7 | Check fired alerts | Admin workstation | Azure Monitor > Alerts | Not applicable | Alert history confirms fired/not fired |
| 8 | Test action group | Admin workstation | Action group > Test action group | Not applicable | Notification path works |
| 9 | Fix alert rule | Admin workstation | Update scope, signal, threshold, or action group | Depends on alert type | Alert rule is corrected |
| 10 | Verify by controlled trigger | Admin workstation | Generate safe test condition | Not applicable | Alert fires and notification is received |

---

## 04_Troubleshoot_Azure_Network_Storage_Compute_Monitor_And_Backup_Issues_Backup_Workflow

### Phase 1: Vault, Policy, And Protected Item

| Step | Task | Device | Command | PowerShell | Expected Result |
|---|---|---|---|---|---|
| 1 | Confirm vault | Admin workstation | Recovery Services Vault > Overview | `Get-AzRecoveryServicesVault -Name <vault-name>` | Vault exists |
| 2 | Set vault context | Admin workstation | Not applicable | `Set-AzRecoveryServicesVaultContext -Vault <vault>` | Backup cmdlets target correct vault |
| 3 | Check backup items | Admin workstation | Vault > Backup items | `Get-AzRecoveryServicesBackupItem -BackupManagementType AzureVM -WorkloadType AzureVM -VaultId <vault-id>` | Protected item is listed |
| 4 | Check backup policy | Admin workstation | Backup item > Backup policy | `Get-AzRecoveryServicesBackupProtectionPolicy -VaultId <vault-id>` | Policy is assigned |
| 5 | Check backup jobs | Admin workstation | Vault > Backup jobs | `Get-AzRecoveryServicesBackupJob -VaultId <vault-id> -From (Get-Date).AddDays(-7) -To (Get-Date)` | Job success/failure is visible |
| 6 | Open failed job details | Admin workstation | Backup job > Error details | `Get-AzRecoveryServicesBackupJobDetail -Job <job>` | Error code and recommendation are visible |
| 7 | Check VM agent | Admin workstation | VM > Properties / Extensions | `Get-AzVM -ResourceGroupName <resource-group> -Name <vm-name> -Status` | Agent is ready |
| 8 | Check snapshot extension | Admin workstation | VM > Extensions | `Get-AzVMExtension -ResourceGroupName <resource-group> -VMName <vm-name>` | Backup extension state is known |
| 9 | Trigger backup if safe | Admin workstation | Backup item > Backup now | `Backup-AzRecoveryServicesBackupItem -Item <backup-item>` | On-demand backup starts |
| 10 | Verify job completion | Admin workstation | Vault > Backup jobs | `Get-AzRecoveryServicesBackupJob -VaultId <vault-id> -Status InProgress` | Job completes successfully |

### Phase 2: Recovery Points And Restore

| Step | Task | Device | Command | PowerShell | Expected Result |
|---|---|---|---|---|---|
| 1 | List recovery points | Admin workstation | Backup item > Restore points | `Get-AzRecoveryServicesBackupRecoveryPoint -Item <backup-item> -StartDate (Get-Date).AddDays(-30) -EndDate (Get-Date)` | Recovery points are visible |
| 2 | Confirm restore point type | Admin workstation | Restore point details | Review recovery point object | App-consistent, file-system-consistent, or crash-consistent type is known |
| 3 | Confirm retention | Admin workstation | Backup policy > Retention range | Review policy | Missing point is not expired by design |
| 4 | Check soft delete | Admin workstation | Vault > Properties > Security settings | Not applicable | Soft delete state is known |
| 5 | Validate restore permissions | Admin workstation | IAM on vault, target RG, storage, network | `Get-AzRoleAssignment -SignInName <admin-user> -Scope <scope>` | Operator has required restore permissions |
| 6 | Choose restore type | Admin workstation | Restore VM, replace disk, create new VM, file recovery | Not applicable | Restore method matches need |
| 7 | Perform test restore if required | Admin workstation | Backup item > Restore VM | `Restore-AzRecoveryServicesBackupItem` | Restore job starts |
| 8 | Monitor restore job | Admin workstation | Vault > Backup jobs | `Get-AzRecoveryServicesBackupJob -VaultId <vault-id>` | Restore completes |
| 9 | Validate restored workload | Restored VM/client | App-specific validation | Not applicable | Restored workload is usable |
| 10 | Record recovery evidence | Admin workstation | Save job output and restore details | Not applicable | Restore proof is documented |

---

## 04_Troubleshoot_Azure_Network_Storage_Compute_Monitor_And_Backup_Issues_Verification_Commands

### Azure Context

| Purpose | Command |
|---|---|
| Show CLI context | `az account show --output table` |
| List subscriptions | `az account list --output table` |
| Set subscription | `az account set --subscription <subscription-id>` |
| Show resource | `az resource show --ids <resource-id>` |
| List resources in RG | `az resource list -g <resource-group> --output table` |

### Network Verification

| Purpose | Command |
|---|---|
| Test DNS | `nslookup <fqdn>` |
| Test TCP port from Windows | `Test-NetConnection <destination> -Port <port>` |
| Trace route from Windows | `tracert <destination>` |
| Show IP config Windows | `ipconfig /all` |
| Show IP config Linux | `ip addr` |
| Trace route Linux | `traceroute <destination>` |
| Test port Linux | `nc -vz <destination> <port>` |
| List NSGs | `az network nsg list -g <resource-group> --output table` |
| List route tables | `az network route-table list -g <resource-group> --output table` |
| Show VNet peerings | `az network vnet peering list -g <resource-group> --vnet-name <vnet-name> --output table` |
| Show private endpoint | `az network private-endpoint show -g <resource-group> -n <private-endpoint-name>` |
| List private DNS links | `az network private-dns link vnet list -g <dns-rg> -z <zone-name> --output table` |

### Storage Verification

| Purpose | Command |
|---|---|
| Show storage account | `az storage account show -g <resource-group> -n <storage-account>` |
| Show storage network rules | `az storage account network-rule list -g <resource-group> --account-name <storage-account>` |
| Check blob DNS | `nslookup <storage-account>.blob.core.windows.net` |
| Check file DNS | `nslookup <storage-account>.file.core.windows.net` |
| Test blob HTTPS | `Test-NetConnection <storage-account>.blob.core.windows.net -Port 443` |
| Test Azure Files SMB | `Test-NetConnection <storage-account>.file.core.windows.net -Port 445` |
| List containers with login auth | `az storage container list --account-name <storage-account> --auth-mode login --output table` |
| List shares | `az storage share-rm list --storage-account <storage-account> -g <resource-group> --output table` |

### Compute Verification

| Purpose | Command |
|---|---|
| Show VM instance view | `az vm get-instance-view -g <resource-group> -n <vm-name> --output table` |
| Start VM | `az vm start -g <resource-group> -n <vm-name>` |
| Restart VM | `az vm restart -g <resource-group> -n <vm-name>` |
| Redeploy VM | `az vm redeploy -g <resource-group> -n <vm-name>` |
| List VM extensions | `az vm extension list -g <resource-group> --vm-name <vm-name> --output table` |
| Run command Windows | `az vm run-command invoke -g <resource-group> -n <vm-name> --command-id RunPowerShellScript --scripts "hostname"` |
| Run command Linux | `az vm run-command invoke -g <resource-group> -n <vm-name> --command-id RunShellScript --scripts "hostname"` |
| Show boot diagnostics setting | `az vm boot-diagnostics get-boot-log -g <resource-group> -n <vm-name>` |

### Monitor Verification

| Purpose | Command |
|---|---|
| List diagnostic settings | `az monitor diagnostic-settings list --resource <resource-id>` |
| List metric definitions | `az monitor metrics list-definitions --resource <resource-id> --output table` |
| List metric values | `az monitor metrics list --resource <resource-id> --metric <metric-name>` |
| List alert rules | `az monitor metrics alert list -g <resource-group> --output table` |
| List action groups | `az monitor action-group list -g <resource-group> --output table` |
| List activity log | `az monitor activity-log list --resource-group <resource-group> --offset 24h` |

### Backup Verification

| Purpose | PowerShell |
|---|---|
| Get vault | `Get-AzRecoveryServicesVault -Name <vault-name>` |
| Set vault context | `Set-AzRecoveryServicesVaultContext -Vault <vault>` |
| List backup containers | `Get-AzRecoveryServicesBackupContainer -ContainerType AzureVM -VaultId <vault-id>` |
| List backup items | `Get-AzRecoveryServicesBackupItem -BackupManagementType AzureVM -WorkloadType AzureVM -VaultId <vault-id>` |
| List backup jobs | `Get-AzRecoveryServicesBackupJob -VaultId <vault-id> -From (Get-Date).AddDays(-7) -To (Get-Date)` |
| Get job details | `Get-AzRecoveryServicesBackupJobDetail -Job <job>` |
| List recovery points | `Get-AzRecoveryServicesBackupRecoveryPoint -Item <backup-item> -StartDate (Get-Date).AddDays(-30) -EndDate (Get-Date)` |
| Trigger backup | `Backup-AzRecoveryServicesBackupItem -Item <backup-item>` |

### Useful KQL

| Purpose | KQL |
|---|---|
| Recent Azure Activity | `AzureActivity | where TimeGenerated > ago(24h) | where ResourceGroup =~ "<resource-group>" | sort by TimeGenerated desc` |
| Failed Azure Activity | `AzureActivity | where TimeGenerated > ago(24h) | where ActivityStatusValue == "Failure" | sort by TimeGenerated desc` |
| VM heartbeat | `Heartbeat | where TimeGenerated > ago(24h) | where Computer =~ "<vm-name>" | sort by TimeGenerated desc` |
| VM performance | `Perf | where TimeGenerated > ago(24h) | where Computer =~ "<vm-name>" | summarize avg(CounterValue) by CounterName, bin(TimeGenerated, 15m)` |
| Storage blob errors | `StorageBlobLogs | where TimeGenerated > ago(24h) | where AccountName =~ "<storage-account>" | where StatusCode >= 400 | sort by TimeGenerated desc` |
| Alert fired events | `AlertsManagementResources | where type =~ "microsoft.alertsmanagement/alerts" | where properties.essentials.startDateTime > ago(24h)` |

---

## 04_Troubleshoot_Azure_Network_Storage_Compute_Monitor_And_Backup_Issues_Rollback

### Rollback Principles

- Capture current config before modifying NSGs, UDRs, DNS, storage firewall, VM state, diagnostic settings, alerts, or backup policy.
- Prefer temporary allow rules with narrow source, destination, port, and expiration.
- Do not broadly open storage public access to fix a private endpoint or RBAC problem.
- Do not delete failed VMs, disks, snapshots, or backup items until recovery evidence is captured.
- Do not disable backup protection unless retention and restore impact are understood.
- Preserve failed backup job details before retrying.
- Restore monitoring rules after testing.

### Rollback Table

| Change Made | Rollback Action | Verification |
|---|---|---|
| Added temporary NSG allow rule | Remove rule after permanent fix | Effective security rules no longer show temporary rule |
| Changed UDR next hop | Restore previous route table entry | Effective routes show expected next hop |
| Added private DNS link | Remove if wrong VNet linked | DNS resolves to intended endpoint |
| Changed storage firewall | Restore previous default action and IP/VNet rules | Storage network rules match approved baseline |
| Enabled public network access temporarily | Disable or restore previous setting | PublicNetworkAccess returns to approved state |
| Changed VM size | Resize back if temporary | VM size matches design |
| Restarted or redeployed VM | No rollback, but document action | VM health and workload status verified |
| Removed VM extension | Reinstall extension if required | Extension state is Succeeded |
| Changed diagnostic setting | Restore previous destination/categories | Logs flow to approved workspace |
| Disabled alert processing rule | Re-enable after test | Suppression state matches design |
| Changed alert threshold | Restore approved threshold | Alert rule matches baseline |
| Triggered test restore | Delete test restored resource after validation | Test restore cleanup complete |
| Changed backup policy | Reassign original policy if temporary | Backup item shows correct policy |
| Stopped backup protection | Re-enable protection if accidental | Backup item protected again |

### Pre-Change Capture Commands

| Purpose | Command |
|---|---|
| Export resource JSON | `az resource show --ids <resource-id> > resource_before.json` |
| Export NSG | `az network nsg show -g <resource-group> -n <nsg-name> > nsg_before.json` |
| Export route table | `az network route-table show -g <resource-group> -n <route-table-name> > route_table_before.json` |
| Export storage account | `az storage account show -g <resource-group> -n <storage-account> > storage_before.json` |
| Export storage network rules | `az storage account network-rule list -g <resource-group> --account-name <storage-account> > storage_network_rules_before.json` |
| Export VM | `az vm show -g <resource-group> -n <vm-name> > vm_before.json` |
| Export diagnostic settings | `az monitor diagnostic-settings list --resource <resource-id> > diagnostics_before.json` |
| Export activity log | `az monitor activity-log list --resource-group <resource-group> --offset 24h > activity_before.json` |

---

## 04_Troubleshoot_Azure_Network_Storage_Compute_Monitor_And_Backup_Issues_Failure_Checks

| Failure | Check | Fix |
|---|---|---|
| Network test fails but NSG allows | UDR, firewall, NVA, guest firewall, DNS, service firewall | Check effective routes and destination firewall |
| IP flow says allow but app fails | Wrong port, app not listening, guest firewall, TLS/app auth | Test local listener and app logs |
| DNS resolves public endpoint instead of private endpoint | Missing private DNS zone link or wrong DNS server | Link zone to VNet or fix conditional forwarding |
| Private endpoint pending | Connection not approved | Approve private endpoint |
| Private endpoint approved but unreachable | DNS wrong or NSG/route path issue | Fix private DNS and source routing |
| Storage 403 | RBAC, SAS, key, firewall, private endpoint, ACL | Determine auth method and check matching control |
| Storage works from portal but app fails | App identity or network path differs from admin | Test as app identity/source |
| Azure Files SMB fails | Port 445 blocked by network or ISP, private endpoint DNS wrong | Use private endpoint/VPN or allow SMB path |
| VM running but cannot RDP/SSH | NSG, public IP, Bastion, guest firewall, service stopped | Use IP flow, boot diagnostics, run command |
| VM extension fails | VM agent unhealthy, no outbound internet, bad script, identity issue | Check extension status and agent |
| VM start fails | Allocation, quota, disk, policy, host issue | Check Activity Log and VM status |
| Logs missing | Diagnostic setting missing, wrong workspace, table not enabled, ingestion delay | Fix diagnostic setting and wait |
| Alert did not fire | Wrong metric, scope, threshold, dimension, action group, processing rule | Review alert history and processing rules |
| Backup failed | VM agent, snapshot, extension, vault, policy, storage, permissions | Open backup job details |
| Restore point missing | Backup never completed, retention expired, wrong vault/item | Check backup item and policy |
| Restore fails | Permission, target RG/VNet/storage issue, quota, resource name conflict | Check restore job details |
| Activity Log lacks expected event | Wrong subscription, time range, resource group, or caller | Expand time range and scope |
| Resource health degraded | Platform issue or host issue | Check Service Health and redeploy if appropriate |
| Metric exists but no logs | Metrics and logs use different pipelines | Enable diagnostic logs to workspace |

---

## 04_Troubleshoot_Azure_Network_Storage_Compute_Monitor_And_Backup_Issues_Escalation_Handoff

### Escalate To Network Team When

- UDR or NVA routing controls the path
- Azure Firewall, third-party firewall, or VPN gateway is in the path
- Peering, gateway transit, ExpressRoute, or VPN is involved
- Private DNS forwarding is managed centrally
- NSG changes require security approval
- Traffic crosses hub-spoke boundaries

### Escalate To Storage Team When

- Storage firewall, private endpoint, or data access roles are unclear
- Production data is inaccessible
- SAS/key policy conflicts with application design
- Lifecycle management deleted or tiered expected data
- Storage latency or throttling affects workload

### Escalate To Compute Team When

- VM guest OS is unhealthy
- VM agent or extensions fail
- Boot diagnostics shows OS failure
- Disk attach, resize, encryption, or snapshot issue exists
- VM scale set or availability zone behavior is involved

### Escalate To Monitoring Team When

- Logs are missing from workspace
- Alerts are not firing or notifications fail
- Diagnostic settings are centrally governed
- Action groups or alert processing rules are managed by platform team
- Workspace retention or table plan is involved

### Escalate To Backup / DR Team When

- Backup jobs fail repeatedly
- Restore points are missing
- Restore fails or RTO/RPO is at risk
- Vault immutability, soft delete, or cross-region restore is involved
- Production restore is requested

### Escalate To Microsoft Support When

- Resource health indicates platform issue
- VM is stuck in provisioning/deleting
- Backup job fails with platform-side error after prerequisites are correct
- Storage availability or data plane issue persists across clients
- Network Watcher results contradict observed traffic
- Azure Monitor ingestion is delayed beyond expected windows
- A restore point exists but cannot be used

### Handoff Data To Include

| Field | Value |
|---|---|
| Incident ID | `<ticket-id>` |
| Subscription ID | `<subscription-id>` |
| Resource group | `<resource-group>` |
| Region | `<region>` |
| Affected resource ID | `<resource-id>` |
| Symptom category | `<Network / Storage / Compute / Monitor / Backup>` |
| Source | `<source-resource-or-client>` |
| Destination | `<destination-resource-or-service>` |
| Protocol/port | `<protocol-port>` |
| DNS result | `<fqdn-to-ip>` |
| Network Watcher result | `<result>` |
| Activity Log event | `<event-id>` |
| Resource Health status | `<status>` |
| Backup job ID | `<job-id>` |
| Alert rule ID | `<alert-rule-id>` |
| Log Analytics workspace | `<workspace-id>` |
| Error message | `<error>` |
| Recent changes | `<changes>` |
| Remediation attempted | `<actions>` |
| Current blocker | `<blocker>` |
| Business impact | `<impact>` |

---

## 04_Troubleshoot_Azure_Network_Storage_Compute_Monitor_And_Backup_Issues_Incident_Record_Template

| Field | Entry |
|---|---|
| Date opened | `<yyyy-mm-dd>` |
| Opened by | `<name>` |
| Subscription | `<subscription-name>` |
| Resource group | `<resource-group>` |
| Resource | `<resource-name>` |
| Resource type | `<resource-type>` |
| Symptom | `<symptom>` |
| Root cause category | `<Network / Storage / Compute / Monitor / Backup / Platform / Change>` |
| Root cause | `<confirmed-root-cause>` |
| Evidence | `<logs, metrics, screenshots, command outputs, job IDs>` |
| Fix applied | `<fix>` |
| Security impact | `<impact>` |
| Availability impact | `<impact>` |
| Data protection impact | `<impact>` |
| Rollback action | `<rollback>` |
| Verification performed | `<verification>` |
| Preventive action | `<preventive-action>` |
| Closed by | `<name>` |
| Date closed | `<yyyy-mm-dd>` |

---

## 04_Troubleshoot_Azure_Network_Storage_Compute_Monitor_And_Backup_Issues_Preventive_Controls

| Control | Target State |
|---|---|
| Network baselines | VNets, subnets, NSGs, UDRs, peerings, and DNS links documented |
| Private endpoint DNS | Private DNS zones linked to required VNets |
| NSG change control | Rule changes require ticket and rollback |
| Route table review | UDRs reviewed before production network changes |
| Storage firewall baseline | Public/private access design documented |
| Storage data protection | Soft delete, versioning, and retention configured where required |
| VM diagnostics | Boot diagnostics and guest monitoring enabled |
| VM backup | Critical VMs protected by backup policy |
| Diagnostic settings | Critical resources send logs to approved workspace |
| Alert coverage | Critical metrics and log signals have tested alerts |
| Action group testing | Notification paths tested periodically |
| Backup testing | Restore tests performed on schedule |
| Activity log review | Change events monitored for critical resources |
| Incident record | Root cause, fix, rollback, and prevention documented |

---

## 04_Troubleshoot_Azure_Network_Storage_Compute_Monitor_And_Backup_Issues_Related_Labs

| Lab                                                       | Purpose                                      |
| --------------------------------------------------------- | -------------------------------------------- |
| Troubleshoot NSG with IP flow verify                      | Identify allow/deny path                     |
| Troubleshoot effective routes                             | Identify UDR and next-hop issues             |
| Configure VNet peering and DNS resolution                 | Validate cross-VNet connectivity             |
| Configure private endpoint and private DNS                | Validate private service access              |
| Troubleshoot storage firewall and private endpoint access | Isolate storage access failures              |
| Troubleshoot Azure Files SMB access                       | Validate port 445, DNS, and identity         |
| Troubleshoot VM RDP and SSH access                        | Isolate NSG, guest firewall, and boot issues |
| Troubleshoot VM extensions                                | Validate agent and extension status          |
| Configure diagnostic settings to Log Analytics            | Ensure logs are collected                    |
| Build Azure Monitor metric alert                          | Validate alert firing and action group       |
| Troubleshoot missing Log Analytics data                   | Confirm ingestion and table routing          |
| Configure Azure VM backup                                 | Protect VM with Recovery Services Vault      |
| Restore Azure VM from backup                              | Validate restore readiness                   |
| Create operational incident record                        | Document evidence, fix, and rollback         |