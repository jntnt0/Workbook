Azure_Migration_Foundation_Provisioning.md
# Azure_Migration_Foundation_Provisioning

# Azure_Migration_Foundation_Provisioning_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Management Group Hierarchy | The structural container for all subscriptions - governs policy, RBAC, and landing zones before any workload lands |
| Identity Subscription | Hosts domain controllers and Entra Connect - nothing joins the domain or authenticates without this |
| Connectivity Subscription | Hosts the hub VNet, firewall, VPN/ER gateway, Bastion, and DNS - all spoke traffic flows through here |
| Management Subscription | Hosts Log Analytics, Recovery Services Vault, Automation Account, and Azure Migrate - your operational backbone |
| Landing Zone Corp Subscription | Where migrated workloads will eventually land - empty at day one, populated by migration waves |
| Sandbox Subscription | Isolated environment for test VMs and the Azure Migrate appliance - validate before you cut over |
| Azure Migrate Project | Central hub for discovery, dependency mapping, assessment, TCO, and wave planning |
| Azure Migrate Appliance | Lightweight source-side VM deployed on-prem or in sandbox - performs agentless discovery of your estate |
| Recovery Services Vault | Single vault backing up all migrated workloads and hosting ASR replication policies |
| Log Analytics Workspace | Central sink for all diagnostic logs, metrics, and security events across every subscription |
| Automation Account | Hosts runbooks for patching, DR orchestration, and operational tasks from day one |
| Azure Private DNS Resolver | Handles split-brain DNS so private endpoints resolve correctly both from on-prem and within Azure |
| Azure Bastion | PaaS-based RDP/SSH access - eliminates the need for a public-IP jump box |
| Entra Connect | Syncs on-prem AD DS identities to Microsoft Entra ID for hybrid authentication |
| Issuing CA | Internal PKI for certificate issuance to domain-joined servers and services |
| VPN Gateway | Site-to-site IPsec tunnel back to on-prem - required before any domain-joined VM can communicate |
| Azure Firewall Premium | Inspects all north/south and east/west traffic - sits in the hub VNet |
| Wave Gating | Nothing in the "do not deploy yet" list gets provisioned until identity, connectivity, and management are validated |

# Azure_Migration_Foundation_Provisioning_Configuration_Checklist
| Step | Task | Subscription | Resource | Expected Result |
|---:|---|---|---|---|
| 1 | Create Management Group hierarchy | Tenant Root | Management Groups: Platform, Landing Zones, Sandbox, Decommissioned | Hierarchy visible in Azure Portal under Management Groups |
| 2 | Create platform subscriptions | Tenant Root | Subscriptions: connectivity, identity, management | Three subscriptions nested under Platform management group |
| 3 | Create landing zone subscription | Landing Zones MG | Subscription: corp | Corp subscription nested under Landing Zones management group |
| 4 | Create sandbox subscription | Sandbox MG | Subscription: sandbox | Sandbox subscription nested under Sandbox management group |
| 5 | Assign ALZ baseline policies | Management Group level | Azure Policy initiative assignments per ALZ defaults | Policy compliance blade shows assignments propagating |
| 6 | Deploy Log Analytics Workspace | management | Log Analytics Workspace | Workspace active, retention set, all subscriptions sending diagnostics |
| 7 | Deploy Automation Account | management | Automation Account linked to Log Analytics | Automation Account online, linked to workspace |
| 8 | Deploy Recovery Services Vault | management | Recovery Services Vault | Vault active, backup and replication policies configured |
| 9 | Create Azure Migrate Project | management | Azure Migrate Project | Project visible, discovery and assessment tools enabled |
| 10 | Deploy hub VNet | connectivity | VNet with defined address space | Hub VNet created, no peerings yet |
| 11 | Deploy Azure Firewall Premium | connectivity | Azure Firewall in hub VNet with dedicated subnet | Firewall running, policy attached, diagnostic logs flowing to Log Analytics |
| 12 | Deploy VPN Gateway | connectivity | VpnGw2 SKU in GatewaySubnet | Gateway provisioned; allow 30-45 min, status shows succeeded |
| 13 | Configure site-to-site VPN | connectivity | Local Network Gateway + Connection resource | Connection status shows Connected, on-prem can reach hub VNet |
| 14 | Deploy Azure Private DNS Resolver | connectivity | Private DNS Resolver with inbound and outbound endpoints | DNS queries from on-prem resolve Azure private DNS zones |
| 15 | Deploy Azure Bastion | connectivity | Bastion Standard in AzureBastionSubnet | Bastion portal blade shows running, RDP/SSH accessible via browser |
| 16 | Deploy identity spoke VNet | identity | Spoke VNet peered to hub | Peering status: Connected both directions |
| 17 | Deploy Domain Controller VM 1 | identity | Windows Server 2022, D2s_v3 | VM running, AD DS role installed, DNS configured |
| 18 | Deploy Domain Controller VM 2 | identity | Windows Server 2022, D2s_v3 | Replication healthy, DC2 is additional DC in same domain |
| 19 | Deploy Entra Connect VM | identity | Windows Server 2022, D2s_v3 | Entra Connect installed, sync cycle completing, users visible in Entra ID |
| 20 | Deploy Issuing CA VM optional | identity | Windows Server 2022, D2s_v3 | ADCS role installed, issuing CA online, templates published |
| 21 | Deploy corp landing zone spoke VNet | corp | Spoke VNet peered to hub | Peering status: Connected, traffic routes through firewall |
| 22 | Deploy sandbox spoke VNet | sandbox | Spoke VNet peered to hub | Peering status: Connected |
| 23 | Deploy Windows test VM | sandbox | Windows Server 2022, B2s | VM domain-joined, reachable via Bastion, logs flowing to Log Analytics |
| 24 | Deploy Linux test VM | sandbox | Ubuntu 22.04 LTS, B2s | VM reachable via Bastion SSH, Azure Monitor agent installed |
| 25 | Deploy Azure Migrate appliance | sandbox or on-prem | Windows Server 2022, D8s_v4 | Appliance registered to Azure Migrate project, discovery running |
| 26 | Validate end-to-end connectivity | all | N/A | On-prem -> VPN -> hub -> spoke -> DC: all reachable. DNS resolving. Bastion working. |
| 27 | Validate Log Analytics ingestion | management | N/A | All VMs and PaaS resources sending heartbeats and diagnostics to workspace |
| 28 | Validate backup policy | management | Recovery Services Vault | Test VM backup job completes successfully |
| 29 | Confirm Azure Migrate discovery | management | Azure Migrate Project | Discovered servers appear in inventory with dependency data populating |
| 30 | Gate check before wave 1 | all | N/A | Identity validated, connectivity validated, management validated, discovery running; cleared to begin wave planning |

# Azure_Migration_Foundation_Provisioning_Skeleton
| Resource | Subscription | Subnet | SKU / Tier | Notes |
|---|---|---|---|---|
| Log Analytics Workspace | management | N/A - PaaS | PerGB2018 | Central to all monitoring |
| Automation Account | management | N/A - PaaS | N/A | Link to Log Analytics at creation |
| Recovery Services Vault | management | N/A - PaaS | GRS redundancy | Enable soft delete and immutability |
| Azure Migrate Project | management | N/A - PaaS | N/A | One project per migration engagement |
| Hub VNet | connectivity | /22 or larger | N/A | Reserve room for GatewaySubnet, AzureFirewallSubnet, AzureBastionSubnet, DNS |
| Azure Firewall | connectivity | AzureFirewallSubnet /26 | Premium | Attach Firewall Policy, enable IDPS |
| VPN Gateway | connectivity | GatewaySubnet /27 | VpnGw2 | Do not put other resources in GatewaySubnet |
| Private DNS Resolver | connectivity | Inbound /28, Outbound /28 | N/A | Wire on-prem forwarders to inbound endpoint IP |
| Azure Bastion | connectivity | AzureBastionSubnet /26 | Standard | Standard tier required for native client and IP-based connection |
| Identity Spoke VNet | identity | /24 | N/A | Peer to hub, route table forcing traffic through firewall |
| DC VM 1 | identity | identity spoke | D2s_v3 | Static private IP, AD DS + DNS roles |
| DC VM 2 | identity | identity spoke | D2s_v3 | Static private IP, additional DC |
| Entra Connect VM | identity | identity spoke | D2s_v3 | Needs line of sight to DCs and internet for sync |
| Issuing CA VM | identity | identity spoke | D2s_v3 | Optional; deploy if internal certs are required |
| Corp Spoke VNet | corp | /22 | N/A | Peer to hub; workload subnets defined per wave |
| Sandbox Spoke VNet | sandbox | /24 | N/A | Peer to hub |
| Windows Test VM | sandbox | sandbox spoke | B2s | Domain join to validate AD replication and DNS |
| Linux Test VM | sandbox | sandbox spoke | B2s | Azure Monitor agent, validate Linux workload path |
| Azure Migrate Appliance | sandbox or on-prem | sandbox spoke or on-prem | D8s_v4 | Requires outbound internet to Azure Migrate endpoints |

# Azure_Migration_Foundation_Provisioning_Verification_Commands
| Step | What to Verify | Command / Method | Expected Result |
|---:|---|---|---|
| 1 | Management group hierarchy | `az account management-group list --output table` | All MGs listed with correct parent/child relationships |
| 2 | Subscription placement | `az account list --output table` | Each subscription shows correct management group |
| 3 | VNet peering status | `az network vnet peering list --resource-group <rg> --vnet-name <hub-vnet> --output table` | All peerings show Connected state |
| 4 | VPN Gateway connection | `az network vpn-connection show --name <conn> --resource-group <rg> --query connectionStatus` | Returns Connected |
| 5 | DC replication health | Run on DC: `repadmin /replsummary` | No replication failures, all DCs listed |
| 6 | DNS resolution from Azure VM | Run on test VM: `nslookup <domain-name>` | Returns correct DC IP |
| 7 | DNS resolution from on-prem | Run on on-prem host: `nslookup <private-endpoint-fqdn>` | Resolves to private IP via DNS Resolver |
| 8 | Entra Connect sync status | Azure Portal -> Entra ID -> Entra Connect -> Sync status | Last sync within last 30 minutes, no errors |
| 9 | Bastion connectivity | Azure Portal -> VM -> Connect -> Bastion | RDP/SSH session opens without public IP |
| 10 | Log Analytics heartbeat | Portal -> Log Analytics -> Logs: `Heartbeat \| summarize max(TimeGenerated) by Computer` | All VMs reporting within last 5 minutes |
| 11 | Azure Migrate discovery | Portal -> Azure Migrate -> Servers -> Discovered servers | On-prem servers appearing with hardware and dependency data |
| 12 | Backup policy validation | Portal -> Recovery Services Vault -> Backup Jobs | Test VM backup job status: Completed |
| 13 | Firewall traffic flow | Portal -> Azure Firewall -> Logs -> Network rule log | Traffic between spokes flowing through firewall, not bypassing |
| 14 | Defender for Cloud coverage | Portal -> Defender for Cloud -> Inventory | All subscriptions and resources onboarded, no missing coverage |

# Azure_Migration_Foundation_Provisioning_Rollback
| Step | Trigger | Rollback Action | Notes |
|---:|---|---|---|
| 1 | VPN Gateway fails to connect | Delete Connection resource, recreate Local Network Gateway with corrected on-prem IP/ASN | Do not delete the Gateway itself; reprovisioning takes about 45 minutes |
| 2 | DC VM fails to promote | Demote via `dcpromo` or force metadata cleanup: `ntdsutil -> metadata cleanup` | DC2 can serve as sole DC while DC1 is rebuilt |
| 3 | Entra Connect sync errors | Disable sync scheduler: `Set-ADSyncScheduler -SyncCycleEnabled $false`, remediate, re-enable | Do not delete Entra Connect config; reconfiguration is destructive |
| 4 | VNet peering misconfiguration | Delete peering both sides: `az network vnet peering delete`, recreate with correct address spaces | Overlapping address spaces require VNet redesign; plan carefully upfront |
| 5 | Azure Firewall blocking legitimate traffic | Add explicit allow rule in Firewall Policy, check network rule logs first | Default deny stance will block traffic; this is expected during initial cutover |
| 6 | Azure Migrate appliance not registering | Re-run appliance configuration manager, verify outbound internet to migrate.azure.com | Check NSG and firewall rules; appliance needs specific outbound URLs |
| 7 | Log Analytics not receiving data | Reinstall Azure Monitor Agent via VM extension, verify DCR association | Check AMA health extension status on VM |
| 8 | Management group policy blocking deployment | Create policy exemption at resource or subscription scope | Exemptions are auditable; document reason |
| 9 | Recovery Services Vault backup failure | Check VM agent status, verify backup policy assignment, review backup job error code | Most failures are agent or network related |
| 10 | Full foundation rollback | Delete resource groups in reverse order: workloads -> spokes -> hub -> management -> identity | Management groups must be emptied of subscriptions before deletion |

# Azure_Migration_Foundation_Provisioning_Failure_Checks
| Symptom | Likely Cause | Where to Look | Fix |
|---|---|---|---|
| VPN shows Connected but on-prem cannot reach Azure VMs | BGP route not propagating or NSG blocking | Check effective routes on VM NIC, check on-prem router BGP table | Add Azure address space to on-prem routing, check NSG inbound rules |
| Domain join fails on test VM | DNS not resolving DC, or DC unreachable | `nslookup <domain>` from VM, check NSG on identity spoke | Point VM DNS to DC private IP, open port 53/88/389/445 in NSG |
| Entra Connect not syncing | No network path to Entra ID endpoints or AD DS | Check sync errors in Synchronization Service Manager | Open outbound 443 to login.microsoftonline.com and related URLs |
| Azure Migrate appliance showing no discovered servers | Appliance cannot reach on-prem vCenter or Hyper-V host, or credentials wrong | Appliance config manager -> validation results | Verify credentials, firewall rules, and that WMI/vCenter ports are open |
| Bastion not connecting | AzureBastionSubnet too small or NSG misconfigured | Check subnet size /26 minimum, check NSG allows GatewayManager and AzureLoadBalancer inbound | Resize subnet or fix NSG; Bastion has specific required rules |
| Spoke VMs cannot reach internet | Default route not pointing through Azure Firewall, or firewall application rule missing | Check effective routes on VM NIC, check Firewall Policy application rules | Add UDR forcing 0.0.0.0/0 to Firewall private IP, add allow rule in policy |
| Log Analytics shows gaps in heartbeat | Azure Monitor Agent stopped or DCR not associated | Check AMA extension status on VM, check DCR associations | Reinstall AMA extension, reassociate DCR |
| Recovery Services Vault replication failing | VM agent outdated or network path to vault endpoint blocked | Check ASR health in vault, VM agent version | Update VM agent, open outbound to *.backup.windowsazure.com |
| Policy compliance showing non-compliant resources | ALZ policies applied before resources configured to meet them | Defender for Cloud -> Recommendations | Remediate per recommendation or create tracked exemption |
| Azure Firewall IDPS blocking Migrate appliance traffic | IDPS signature flagging Migrate traffic as suspicious | Firewall logs -> IDPS log | Add IDPS bypass rule for Migrate appliance source IP |

# Index
Azure_Migration_Foundation_Provisioning.md
Azure_Migration_Foundation_Provisioning
Azure_Migration_Foundation_Provisioning_Mental_Model
Azure_Migration_Foundation_Provisioning_Configuration_Checklist
Azure_Migration_Foundation_Provisioning_Skeleton
Azure_Migration_Foundation_Provisioning_Verification_Commands
Azure_Migration_Foundation_Provisioning_Rollback
Azure_Migration_Foundation_Provisioning_Failure_Checks