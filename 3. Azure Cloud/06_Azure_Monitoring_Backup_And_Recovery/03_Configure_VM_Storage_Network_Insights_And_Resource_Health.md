03_Configure_VM_Storage_Network_Insights_And_Resource_Health.md

# Configure_VM_Storage_Network_Insights_And_Resource_Health

# Configure_VM_Storage_Network_Insights_And_Resource_Health_Index

03_Configure_VM_Storage_Network_Insights_And_Resource_Health.md
Configure_VM_Storage_Network_Insights_And_Resource_Health
Configure_VM_Storage_Network_Insights_And_Resource_Health_Source_Basis
Configure_VM_Storage_Network_Insights_And_Resource_Health_Mental_Model
Configure_VM_Storage_Network_Insights_And_Resource_Health_Planning_Table
Configure_VM_Storage_Network_Insights_And_Resource_Health_Configuration_Checklist
Configure_VM_Storage_Network_Insights_And_Resource_Health_VM_Insights_Skeleton
Configure_VM_Storage_Network_Insights_And_Resource_Health_Storage_Insights_Skeleton
Configure_VM_Storage_Network_Insights_And_Resource_Health_Network_Insights_Skeleton
Configure_VM_Storage_Network_Insights_And_Resource_Health_Resource_Health_Skeleton
Configure_VM_Storage_Network_Insights_And_Resource_Health_Resource_Health_Alert_Skeleton
Configure_VM_Storage_Network_Insights_And_Resource_Health_KQL_Query_Skeleton
Configure_VM_Storage_Network_Insights_And_Resource_Health_Workbook_View_Skeleton
Configure_VM_Storage_Network_Insights_And_Resource_Health_Evidence_Export_Skeleton
Configure_VM_Storage_Network_Insights_And_Resource_Health_Verification_Commands
Configure_VM_Storage_Network_Insights_And_Resource_Health_Rollback
Configure_VM_Storage_Network_Insights_And_Resource_Health_Failure_Checks
Configure_VM_Storage_Network_Insights_And_Resource_Health_Related_Labs

# Configure_VM_Storage_Network_Insights_And_Resource_Health_Source_Basis

| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Monitor virtual machines in Azure | Host metrics, guest metrics, Azure Monitor Agent, guest logs, VM health, VM alerting |
| Microsoft Learn | Enable VM monitoring in Azure Monitor | VM Insights enablement, Azure Monitor Agent, Data Collection Rules, at-scale configuration |
| Microsoft Learn | Azure Monitor Storage insights | Storage account availability, latency, transactions, failures, capacity, default storage metrics |
| Microsoft Learn | Azure Storage metrics | Supported storage metric namespaces, dimensions, and service-level metrics |
| Microsoft Learn | Azure Monitor Network Insights | Topology, health, metrics, connectivity, traffic, diagnostic toolkit, Network Watcher integration |
| Microsoft Learn | Azure Network Watcher | Connection troubleshoot, IP flow verify, next hop, packet capture, flow logs |
| Microsoft Learn | Azure Resource Health overview | Available, Unavailable, Unknown, Degraded states, platform and non-platform events |
| Microsoft Learn | Create Resource Health alerts | Activity log alerting for resource health state changes |
| Microsoft Learn | Azure Monitor Workbooks | Custom resource insight views across VM, storage, network, and health data |

# Configure_VM_Storage_Network_Insights_And_Resource_Health_Mental_Model

| Concept | Operational Meaning |
|---|---|
| VM host metrics | Metrics collected automatically from Azure infrastructure, such as CPU, disk, and network counters |
| VM guest metrics | Metrics from inside the operating system that require enhanced monitoring, Azure Monitor Agent, and a data collection rule |
| VM guest logs | Windows Event Logs, Syslog, IIS logs, custom logs, and other guest data collected through a DCR |
| Azure Monitor Agent | Current agent used to collect guest monitoring data from Azure VMs, Arc servers, and scale sets |
| Data Collection Rule | Rule that defines what guest data is collected and where it is sent |
| Storage insights | Built-in Azure Monitor view for storage performance, capacity, availability, latency, and failures |
| Storage metrics | Default Azure Storage platform metrics used for transactions, availability, latency, capacity, and throttling analysis |
| Storage diagnostic logs | Resource logs that require diagnostic settings before landing in Log Analytics |
| Network insights | Built-in Azure Monitor view for network topology, health, metrics, alerts, connectivity, traffic, and diagnostics |
| Network Watcher | Azure network diagnostic service used for packet capture, connection troubleshoot, IP flow verify, next hop, and flow logs |
| Connection Monitor | Network Watcher capability that validates reachability, latency, and path behavior between endpoints |
| Flow logs | NSG or VNet traffic records used for traffic analytics and security review |
| Resource Health | Per-resource health signal used to determine whether the issue is caused by Azure platform health, user action, or unsupported state |
| Resource Health alert | Activity Log alert that fires when a resource health event occurs |
| First rule | Insights are views. They are only as useful as the metrics, logs, agents, and diagnostic settings behind them |
| Blunt rule | Host metrics can appear automatically, but guest-level VM insights require agent and data collection configuration |

# Configure_VM_Storage_Network_Insights_And_Resource_Health_Planning_Table

| Item | Example | Decision |
|---|---|---|
| Subscription ID | `11111111-1111-1111-1111-111111111111` | `<subscription-id>` |
| Location | `eastus` | `<location>` |
| Monitoring resource group | `rg-monitor-core-01` | `<monitor-resource-group>` |
| Log Analytics workspace | `law-monitor-core-01` | `<workspace-name>` |
| Log Analytics workspace ID | `/subscriptions/<id>/resourceGroups/<rg>/providers/Microsoft.OperationalInsights/workspaces/<name>` | `<workspace-resource-id>` |
| Target VM resource group | `rg-compute-01` | `<vm-resource-group>` |
| Target VM name | `vm-app-01` | `<vm-name>` |
| Target VM resource ID | `/subscriptions/<id>/resourceGroups/rg-compute-01/providers/Microsoft.Compute/virtualMachines/vm-app-01` | `<vm-resource-id>` |
| VM operating system | `Windows Server 2022` | `<windows / linux>` |
| Data collection rule | `dcr-vm-insights-core-01` | `<dcr-name>` |
| Storage account resource group | `rg-storage-01` | `<storage-resource-group>` |
| Storage account name | `stdiagcore01` | `<storage-account-name>` |
| Network resource group | `rg-network-01` | `<network-resource-group>` |
| Virtual network name | `vnet-core-01` | `<vnet-name>` |
| Network interface name | `nic-vm-app-01` | `<nic-name>` |
| NSG name | `nsg-app-01` | `<nsg-name>` |
| Connection Monitor name | `cm-app-to-db-01` | `<connection-monitor-name>` |
| Resource Health alert name | `ala-resource-health-unavailable` | `<resource-health-alert-name>` |
| Action group name | `ag-core-oncall-email` | `<action-group-name>` |
| Workbook name | `wb-resource-insights-health` | `<workbook-name>` |
| Evidence path | `C:\AzureResourceInsights` | `<evidence-path>` |
| Next workbook | `04_Configure_Recovery_Services_Vault_Backup_Vault_Backup_Policies_And_Restore.md` | `<next-task>` |

# Configure_VM_Storage_Network_Insights_And_Resource_Health_Configuration_Checklist

| Step | Task | Device | PowerShell / Azure CLI / KQL | Expected Result |
|---:|---|---|---|---|
| 1 | Sign in to Azure | Admin Workstation / Cloud Shell | `az login` | Azure CLI is authenticated |
| 2 | Select subscription | Admin Workstation / Cloud Shell | `az account set --subscription "<subscription-id>"` | Correct subscription is active |
| 3 | Confirm required providers | Admin Workstation / Cloud Shell | `az provider show --namespace Microsoft.Insights --query registrationState -o tsv` | Microsoft.Insights is registered |
| 4 | Confirm Operational Insights provider | Admin Workstation / Cloud Shell | `az provider show --namespace Microsoft.OperationalInsights --query registrationState -o tsv` | Microsoft.OperationalInsights is registered |
| 5 | Confirm Network provider | Admin Workstation / Cloud Shell | `az provider show --namespace Microsoft.Network --query registrationState -o tsv` | Microsoft.Network is registered |
| 6 | Confirm target VM | Admin Workstation / Cloud Shell | `az vm show --resource-group "<vm-resource-group>" --name "<vm-name>" --output table` | VM exists |
| 7 | Review VM host metrics | Admin Workstation / Cloud Shell | `az monitor metrics list-definitions --resource "<vm-resource-id>" --output table` | Host-level metric definitions are listed |
| 8 | Enable VM enhanced monitoring | Azure Portal / Admin Workstation | Run VM Insights skeleton | Azure Monitor Agent and DCR are applied |
| 9 | Confirm VM agent extension | Admin Workstation / Cloud Shell | `az vm extension list --resource-group "<vm-resource-group>" --vm-name "<vm-name>" --output table` | Azure Monitor Agent extension appears |
| 10 | Confirm DCR association | Admin Workstation / Cloud Shell | `az resource list --resource-type Microsoft.Insights/dataCollectionRuleAssociations --output table` | VM is associated to DCR |
| 11 | Open VM Insights | Azure Portal | VM > Monitoring > Insights | VM performance, maps, or monitoring panes load |
| 12 | Confirm VM guest data in Log Analytics | Log Analytics | Run VM KQL skeleton | Guest performance or event data appears |
| 13 | Confirm storage account | Admin Workstation / Cloud Shell | `az storage account show --resource-group "<storage-resource-group>" --name "<storage-account-name>" --output table` | Storage account exists |
| 14 | Open Storage insights | Azure Portal | Monitor > Insights > Storage Accounts | Storage account appears in Storage insights |
| 15 | Review storage availability and latency | Azure Portal | Storage insights > Availability / Performance | Storage health, latency, and transaction views appear |
| 16 | Query storage metrics | Admin Workstation / Cloud Shell | `az monitor metrics list --resource "<storage-resource-id>" --metric Transactions --interval PT5M --output table` | Storage transaction metrics return |
| 17 | Confirm storage diagnostic setting if logs are required | Admin Workstation / Cloud Shell | `az monitor diagnostic-settings list --resource "<storage-resource-id>" --output table` | Log Analytics routing is visible if configured |
| 18 | Open Network Insights | Azure Portal | Monitor > Insights > Networks | Network health, topology, metrics, alerts, connectivity, and traffic views appear |
| 19 | Confirm Network Watcher availability | Admin Workstation / Cloud Shell | `az network watcher list --output table` | Network Watcher exists for target region |
| 20 | Review network topology | Azure Portal | Network Insights > Topology | VNet and related resources appear |
| 21 | Review network health and metrics | Azure Portal | Network Insights > Network health | Network resource health and metrics appear |
| 22 | Validate NIC effective security rules | Admin Workstation / Cloud Shell | `az network nic list-effective-nsg --resource-group "<network-resource-group>" --name "<nic-name>" --output table` | Effective NSG rules are visible |
| 23 | Validate NIC effective routes | Admin Workstation / Cloud Shell | `az network nic show-effective-route-table --resource-group "<network-resource-group>" --name "<nic-name>" --output table` | Effective routes are visible |
| 24 | Create or confirm Connection Monitor | Azure Portal / Admin Workstation | Run Network Insights skeleton | Connectivity monitoring exists |
| 25 | Open Resource Health for VM | Azure Portal | VM > Resource health | VM health state and history are visible |
| 26 | Open Resource Health for storage account | Azure Portal | Storage account > Resource health | Storage account health state is visible |
| 27 | Create Resource Health alert | Admin Workstation / Cloud Shell | Run Resource Health alert skeleton | Resource health alert rule exists |
| 28 | Build resource insights workbook view | Azure Portal | Monitor > Workbooks | VM, storage, network, and health view exists |
| 29 | Export evidence | Admin Workstation / Cloud Shell | Run evidence export skeleton | Resource insights evidence is saved |
| 30 | Document baseline | Operator | Record VM, storage, network, Resource Health, and workbook state | Resource insight baseline is documented |

# Configure_VM_Storage_Network_Insights_And_Resource_Health_VM_Insights_Skeleton

```bash
# Run from Azure Cloud Shell or a workstation with Azure CLI.
# Purpose: validate VM monitoring prerequisites and enable Azure Monitor Agent.
# Note: VM Insights guest monitoring is commonly completed through Azure portal, Azure Policy, ARM/Bicep, or Data Collection Rules.
# This skeleton confirms the VM, installs Azure Monitor Agent extension, and validates extension state.

SUBSCRIPTION_ID="<subscription-id>"
LOCATION="<location>"
VM_RG="<vm-resource-group>"
VM_NAME="<vm-name>"
WORKSPACE_RG="<monitor-resource-group>"
WORKSPACE_NAME="<workspace-name>"

az account set \
  --subscription "$SUBSCRIPTION_ID"

VM_ID=$(az vm show \
  --resource-group "$VM_RG" \
  --name "$VM_NAME" \
  --query id \
  --output tsv)

WORKSPACE_ID=$(az monitor log-analytics workspace show \
  --resource-group "$WORKSPACE_RG" \
  --workspace-name "$WORKSPACE_NAME" \
  --query id \
  --output tsv)

echo "VM ID: $VM_ID"
echo "Workspace ID: $WORKSPACE_ID"

az vm show \
  --resource-group "$VM_RG" \
  --name "$VM_NAME" \
  --output table

# Windows Azure Monitor Agent extension.
az vm extension set \
  --resource-group "$VM_RG" \
  --vm-name "$VM_NAME" \
  --name AzureMonitorWindowsAgent \
  --publisher Microsoft.Azure.Monitor \
  --enable-auto-upgrade true

# Linux Azure Monitor Agent extension.
# Use this instead for Linux VMs.
# az vm extension set \
#   --resource-group "$VM_RG" \
#   --vm-name "$VM_NAME" \
#   --name AzureMonitorLinuxAgent \
#   --publisher Microsoft.Azure.Monitor \
#   --enable-auto-upgrade true

az vm extension list \
  --resource-group "$VM_RG" \
  --vm-name "$VM_NAME" \
  --output table

# Portal completion path:
# 1. Open the target VM.
# 2. Go to Monitoring > Insights.
# 3. Select Enable.
# 4. Choose the Log Analytics workspace.
# 5. Allow Azure to create or use the required Data Collection Rule.
# 6. Return after several minutes and confirm Performance, Processes, and Map views.
```

# Configure_VM_Storage_Network_Insights_And_Resource_Health_VM_DCR_Association_Check_Skeleton

```bash
# Run from Azure Cloud Shell or a workstation with Azure CLI.
# Purpose: confirm whether a VM has Data Collection Rule associations.

SUBSCRIPTION_ID="<subscription-id>"
VM_RESOURCE_ID="<vm-resource-id>"

az account set \
  --subscription "$SUBSCRIPTION_ID"

az resource list \
  --resource-type "Microsoft.Insights/dataCollectionRuleAssociations" \
  --query "[?contains(id, '$VM_RESOURCE_ID')]" \
  --output jsonc

# If no association exists, use one of these supported deployment methods:
# 1. Azure portal VM > Insights > Enable
# 2. Azure Policy for VM Insights at scale
# 3. ARM or Bicep deployment for Data Collection Rule and association
# 4. Azure Monitor Agent onboarding workflow
```

# Configure_VM_Storage_Network_Insights_And_Resource_Health_Storage_Insights_Skeleton

```bash
# Run from Azure Cloud Shell or a workstation with Azure CLI.
# Purpose: validate Storage insights metric data and optional diagnostic settings.

SUBSCRIPTION_ID="<subscription-id>"
STORAGE_RG="<storage-resource-group>"
STORAGE_ACCOUNT="<storage-account-name>"

az account set \
  --subscription "$SUBSCRIPTION_ID"

STORAGE_ID=$(az storage account show \
  --resource-group "$STORAGE_RG" \
  --name "$STORAGE_ACCOUNT" \
  --query id \
  --output tsv)

echo "Storage Account ID: $STORAGE_ID"

az storage account show \
  --resource-group "$STORAGE_RG" \
  --name "$STORAGE_ACCOUNT" \
  --output table

# Review metric definitions.
az monitor metrics list-definitions \
  --resource "$STORAGE_ID" \
  --output table

# Query transactions.
az monitor metrics list \
  --resource "$STORAGE_ID" \
  --metric Transactions \
  --interval PT5M \
  --aggregation Total \
  --output table

# Query availability.
az monitor metrics list \
  --resource "$STORAGE_ID" \
  --metric Availability \
  --interval PT5M \
  --aggregation Average \
  --output table

# Query latency.
az monitor metrics list \
  --resource "$STORAGE_ID" \
  --metric SuccessE2ELatency \
  --interval PT5M \
  --aggregation Average \
  --output table

# Confirm diagnostic settings if resource logs are required in Log Analytics.
az monitor diagnostic-settings list \
  --resource "$STORAGE_ID" \
  --output table

# Portal path:
# 1. Go to Monitor.
# 2. Select Insights.
# 3. Select Storage Accounts.
# 4. Review Overview, Capacity, Failures, Performance, and Availability views.
# 5. Save a customized workbook copy if the default view needs operator-specific filters.
```

# Configure_VM_Storage_Network_Insights_And_Resource_Health_Network_Insights_Skeleton

```bash
# Run from Azure Cloud Shell or a workstation with Azure CLI.
# Purpose: validate Network Insights inputs and Network Watcher diagnostic paths.

SUBSCRIPTION_ID="<subscription-id>"
LOCATION="<location>"
NETWORK_RG="<network-resource-group>"
VNET_NAME="<vnet-name>"
NIC_NAME="<nic-name>"
NSG_NAME="<nsg-name>"

az account set \
  --subscription "$SUBSCRIPTION_ID"

# Confirm Network Watcher exists.
az network watcher list \
  --output table

# Confirm VNet exists.
az network vnet show \
  --resource-group "$NETWORK_RG" \
  --name "$VNET_NAME" \
  --output table

# Confirm NIC exists.
az network nic show \
  --resource-group "$NETWORK_RG" \
  --name "$NIC_NAME" \
  --output table

# Review effective NSG rules on NIC.
az network nic list-effective-nsg \
  --resource-group "$NETWORK_RG" \
  --name "$NIC_NAME" \
  --output table

# Review effective routes on NIC.
az network nic show-effective-route-table \
  --resource-group "$NETWORK_RG" \
  --name "$NIC_NAME" \
  --output table

# Confirm NSG exists.
az network nsg show \
  --resource-group "$NETWORK_RG" \
  --name "$NSG_NAME" \
  --output table

# Portal path:
# 1. Go to Monitor.
# 2. Select Insights.
# 3. Select Networks.
# 4. Review Network health.
# 5. Review Topology.
# 6. Review Connectivity.
# 7. Review Traffic.
# 8. Use Diagnostic Toolkit for IP flow verify, next hop, packet capture, and connection troubleshoot.
```

# Configure_VM_Storage_Network_Insights_And_Resource_Health_Connection_Monitor_Skeleton

```bash
# Run from Azure Cloud Shell or a workstation with Azure CLI.
# Purpose: create a basic Connection Monitor test between a source VM and destination endpoint.
# Confirm current CLI syntax in your environment with:
# az network watcher connection-monitor --help

SUBSCRIPTION_ID="<subscription-id>"
LOCATION="<location>"
NETWORK_WATCHER_RG="NetworkWatcherRG"
NETWORK_WATCHER_NAME="NetworkWatcher_<location>"
CONNECTION_MONITOR_NAME="<connection-monitor-name>"
SOURCE_VM_ID="<source-vm-resource-id>"
DESTINATION_ADDRESS="<destination-fqdn-or-ip>"
DESTINATION_PORT="443"

az account set \
  --subscription "$SUBSCRIPTION_ID"

az network watcher connection-monitor create \
  --location "$LOCATION" \
  --name "$CONNECTION_MONITOR_NAME" \
  --source-resource "$SOURCE_VM_ID" \
  --dest-address "$DESTINATION_ADDRESS" \
  --dest-port "$DESTINATION_PORT"

az network watcher connection-monitor show \
  --location "$LOCATION" \
  --name "$CONNECTION_MONITOR_NAME" \
  --output jsonc

# Portal validation path:
# 1. Go to Network Watcher.
# 2. Select Connection Monitor.
# 3. Open the test.
# 4. Confirm reachability, RTT, checks failed percentage, and hop details.
# 5. Return to Monitor > Insights > Networks > Connectivity.
```

# Configure_VM_Storage_Network_Insights_And_Resource_Health_Resource_Health_Skeleton

```bash
# Run from Azure Cloud Shell or a workstation with Azure CLI.
# Purpose: open and record Resource Health state through portal, then correlate with Activity Log.

SUBSCRIPTION_ID="<subscription-id>"
TARGET_RESOURCE_ID="<target-resource-id>"

az account set \
  --subscription "$SUBSCRIPTION_ID"

az resource show \
  --ids "$TARGET_RESOURCE_ID" \
  --output table

# Pull recent activity log records for the resource.
az monitor activity-log list \
  --resource-id "$TARGET_RESOURCE_ID" \
  --offset 7d \
  --output jsonc

# Portal path:
# 1. Open the target resource.
# 2. Select Resource health.
# 3. Record current status:
#    - Available
#    - Unavailable
#    - Unknown
#    - Degraded
# 4. Open Health history.
# 5. Review platform event details.
# 6. Select Diagnose and solve problems if the resource is degraded or unavailable.
# 7. Save screenshot or export evidence for the incident record.
```

# Configure_VM_Storage_Network_Insights_And_Resource_Health_Resource_Health_Alert_Skeleton

```bash
# Run from Azure Cloud Shell or a workstation with Azure CLI.
# Purpose: create an Activity Log alert for Resource Health events.

SUBSCRIPTION_ID="<subscription-id>"
MONITOR_RG="<monitor-resource-group>"
ACTION_GROUP_NAME="<action-group-name>"
RESOURCE_HEALTH_ALERT_NAME="<resource-health-alert-name>"
SCOPE="/subscriptions/<subscription-id>"

az account set \
  --subscription "$SUBSCRIPTION_ID"

ACTION_GROUP_ID=$(az monitor action-group show \
  --resource-group "$MONITOR_RG" \
  --name "$ACTION_GROUP_NAME" \
  --query id \
  --output tsv)

az monitor activity-log alert create \
  --resource-group "$MONITOR_RG" \
  --name "$RESOURCE_HEALTH_ALERT_NAME" \
  --scopes "$SCOPE" \
  --condition category=ResourceHealth \
  --action-groups "$ACTION_GROUP_ID" \
  --description "Alert when Azure Resource Health events are written to the Activity Log"

az monitor activity-log alert show \
  --resource-group "$MONITOR_RG" \
  --name "$RESOURCE_HEALTH_ALERT_NAME" \
  --output jsonc
```

# Configure_VM_Storage_Network_Insights_And_Resource_Health_KQL_Query_Skeleton

```kusto
// Purpose: validate VM, storage, network, and Resource Health data in Log Analytics.
// Run in the Log Analytics workspace used by the monitoring baseline.

//
// 1. Confirm VM heartbeat if collected.
//
Heartbeat
| where TimeGenerated > ago(24h)
| summarize LastHeartbeat=max(TimeGenerated), Count=count() by Computer, OSType, ResourceGroup, _ResourceId
| order by LastHeartbeat desc

//
// 2. Confirm VM performance data if collected.
//
Perf
| where TimeGenerated > ago(24h)
| summarize Count=count(), LastRecord=max(TimeGenerated) by Computer, ObjectName, CounterName
| order by LastRecord desc

//
// 3. Confirm VM insights performance table if present.
//
InsightsMetrics
| where TimeGenerated > ago(24h)
| summarize Count=count(), LastRecord=max(TimeGenerated) by Namespace, Name, Computer
| order by LastRecord desc

//
// 4. Confirm Windows event collection if enabled.
//
Event
| where TimeGenerated > ago(24h)
| summarize Count=count(), LastEvent=max(TimeGenerated) by Computer, EventLog, EventLevelName
| order by Count desc

//
// 5. Confirm Syslog collection if enabled.
//
Syslog
| where TimeGenerated > ago(24h)
| summarize Count=count(), LastEvent=max(TimeGenerated) by Computer, Facility, SeverityLevel
| order by Count desc

//
// 6. Confirm storage metrics routed to Log Analytics.
//
AzureMetrics
| where TimeGenerated > ago(24h)
| where ResourceProvider =~ "MICROSOFT.STORAGE"
| summarize Count=count(), LastRecord=max(TimeGenerated) by ResourceId, MetricName
| order by LastRecord desc

//
// 7. Storage latency trend from AzureMetrics.
//
AzureMetrics
| where TimeGenerated > ago(6h)
| where ResourceProvider =~ "MICROSOFT.STORAGE"
| where MetricName in ("SuccessE2ELatency", "SuccessServerLatency")
| summarize AvgLatency=avg(Average), MaxLatency=max(Maximum) by bin(TimeGenerated, 15m), ResourceId, MetricName
| order by TimeGenerated asc

//
// 8. Storage availability trend.
//
AzureMetrics
| where TimeGenerated > ago(24h)
| where ResourceProvider =~ "MICROSOFT.STORAGE"
| where MetricName == "Availability"
| summarize AvgAvailability=avg(Average), MinAvailability=min(Minimum) by bin(TimeGenerated, 30m), ResourceId
| order by TimeGenerated asc

//
// 9. Network metrics routed to Log Analytics.
//
AzureMetrics
| where TimeGenerated > ago(24h)
| where ResourceProvider has_any ("MICROSOFT.NETWORK")
| summarize Count=count(), LastRecord=max(TimeGenerated) by ResourceId, MetricName
| order by LastRecord desc

//
// 10. Resource Health events from Activity Log.
//
AzureActivity
| where TimeGenerated > ago(30d)
| where CategoryValue == "ResourceHealth"
| project TimeGenerated, ResourceGroup, ResourceProviderValue, Resource, OperationNameValue, ActivityStatusValue, Properties
| order by TimeGenerated desc

//
// 11. Resource unavailable or degraded event hunting.
//
AzureActivity
| where TimeGenerated > ago(30d)
| where CategoryValue == "ResourceHealth"
| where Properties has_any ("Unavailable", "Degraded", "ResourceDegraded", "ResourceUnavailable")
| project TimeGenerated, ResourceGroup, Resource, OperationNameValue, ActivityStatusValue, Properties
| order by TimeGenerated desc

//
// 12. Cross-resource health and performance snapshot.
//
union isfuzzy=true
    Heartbeat,
    Perf,
    InsightsMetrics,
    AzureMetrics,
    AzureActivity
| where TimeGenerated > ago(24h)
| summarize Count=count(), LastRecord=max(TimeGenerated) by Type
| order by LastRecord desc
```

# Configure_VM_Storage_Network_Insights_And_Resource_Health_Workbook_View_Skeleton

```text
Azure Portal workbook build path:

1. Open Azure portal.
2. Go to Monitor.
3. Select Workbooks.
4. Select New.
5. Add text block:
   Title: Resource Insights and Health
   Purpose: show VM, storage, network, and Resource Health signals in one operator view.

6. Add parameters:
   - Subscription
   - Resource Group
   - Time Range
   - Resource Type
   - Target Resource

7. Add VM section:
   Data source: Logs
   Query:
     Heartbeat
     | where TimeGenerated > ago(24h)
     | summarize LastHeartbeat=max(TimeGenerated) by Computer, OSType, _ResourceId
     | order by LastHeartbeat desc

8. Add VM performance section:
   Data source: Logs
   Query:
     InsightsMetrics
     | where TimeGenerated > ago(24h)
     | summarize Count=count(), LastRecord=max(TimeGenerated) by Computer, Namespace, Name
     | order by LastRecord desc

9. Add Storage section:
   Data source: Metrics or Logs
   Metric examples:
     - Transactions
     - Availability
     - SuccessE2ELatency
     - SuccessServerLatency

10. Add Network section:
    Data source: Azure Resource Graph
    Query:
      resources
      | where type startswith "microsoft.network/"
      | project name, type, resourceGroup, location
      | order by type asc, name asc

11. Add Resource Health section:
    Data source: Azure Resource Graph or Logs
    Logs query:
      AzureActivity
      | where TimeGenerated > ago(30d)
      | where CategoryValue == "ResourceHealth"
      | project TimeGenerated, ResourceGroup, Resource, OperationNameValue, ActivityStatusValue, Properties
      | order by TimeGenerated desc

12. Add operator notes:
    - Check Resource Health first when platform outage is suspected.
    - Check VM guest data when the VM is available but workload is unhealthy.
    - Check Storage insights when latency, throttling, availability, or capacity symptoms appear.
    - Check Network Insights and Network Watcher when reachability or routing symptoms appear.

13. Save workbook:
    Name: <workbook-name>
    Resource group: <monitor-resource-group>
    Location: <location>
    Sharing: Shared
```

# Configure_VM_Storage_Network_Insights_And_Resource_Health_Evidence_Export_Skeleton

```bash
# Run from Azure Cloud Shell or a workstation with Azure CLI.
# Purpose: export VM, storage, network, and health evidence.

EVIDENCE_PATH="./AzureResourceInsights"
SUBSCRIPTION_ID="<subscription-id>"
VM_RG="<vm-resource-group>"
VM_NAME="<vm-name>"
STORAGE_RG="<storage-resource-group>"
STORAGE_ACCOUNT="<storage-account-name>"
NETWORK_RG="<network-resource-group>"
VNET_NAME="<vnet-name>"
NIC_NAME="<nic-name>"
NSG_NAME="<nsg-name>"
MONITOR_RG="<monitor-resource-group>"
RESOURCE_HEALTH_ALERT_NAME="<resource-health-alert-name>"

mkdir -p "$EVIDENCE_PATH"

az account set \
  --subscription "$SUBSCRIPTION_ID"

az account show \
  --output jsonc \
  > "$EVIDENCE_PATH/account-context.jsonc"

az vm show \
  --resource-group "$VM_RG" \
  --name "$VM_NAME" \
  --output jsonc \
  > "$EVIDENCE_PATH/vm.jsonc"

az vm extension list \
  --resource-group "$VM_RG" \
  --vm-name "$VM_NAME" \
  --output jsonc \
  > "$EVIDENCE_PATH/vm-extensions.jsonc"

az storage account show \
  --resource-group "$STORAGE_RG" \
  --name "$STORAGE_ACCOUNT" \
  --output jsonc \
  > "$EVIDENCE_PATH/storage-account.jsonc"

STORAGE_ID=$(az storage account show \
  --resource-group "$STORAGE_RG" \
  --name "$STORAGE_ACCOUNT" \
  --query id \
  --output tsv)

az monitor metrics list-definitions \
  --resource "$STORAGE_ID" \
  --output jsonc \
  > "$EVIDENCE_PATH/storage-metric-definitions.jsonc"

az monitor diagnostic-settings list \
  --resource "$STORAGE_ID" \
  --output jsonc \
  > "$EVIDENCE_PATH/storage-diagnostic-settings.jsonc"

az network vnet show \
  --resource-group "$NETWORK_RG" \
  --name "$VNET_NAME" \
  --output jsonc \
  > "$EVIDENCE_PATH/vnet.jsonc"

az network nic show \
  --resource-group "$NETWORK_RG" \
  --name "$NIC_NAME" \
  --output jsonc \
  > "$EVIDENCE_PATH/nic.jsonc"

az network nic list-effective-nsg \
  --resource-group "$NETWORK_RG" \
  --name "$NIC_NAME" \
  --output jsonc \
  > "$EVIDENCE_PATH/nic-effective-nsg.jsonc"

az network nic show-effective-route-table \
  --resource-group "$NETWORK_RG" \
  --name "$NIC_NAME" \
  --output jsonc \
  > "$EVIDENCE_PATH/nic-effective-routes.jsonc"

az network nsg show \
  --resource-group "$NETWORK_RG" \
  --name "$NSG_NAME" \
  --output jsonc \
  > "$EVIDENCE_PATH/nsg.jsonc"

az monitor activity-log alert show \
  --resource-group "$MONITOR_RG" \
  --name "$RESOURCE_HEALTH_ALERT_NAME" \
  --output jsonc \
  > "$EVIDENCE_PATH/resource-health-alert.jsonc"

az monitor activity-log list \
  --offset 30d \
  --output jsonc \
  > "$EVIDENCE_PATH/activity-log-30d.jsonc"

echo "Evidence exported to $EVIDENCE_PATH"
```

# Configure_VM_Storage_Network_Insights_And_Resource_Health_Verification_Commands

```bash
# VM verification.

az vm show \
  --resource-group "<vm-resource-group>" \
  --name "<vm-name>" \
  --output table

az vm extension list \
  --resource-group "<vm-resource-group>" \
  --vm-name "<vm-name>" \
  --output table

az monitor metrics list-definitions \
  --resource "<vm-resource-id>" \
  --output table

az monitor metrics list \
  --resource "<vm-resource-id>" \
  --metric "Percentage CPU" \
  --interval PT5M \
  --output table

# Storage verification.

az storage account show \
  --resource-group "<storage-resource-group>" \
  --name "<storage-account-name>" \
  --output table

az monitor metrics list-definitions \
  --resource "<storage-resource-id>" \
  --output table

az monitor metrics list \
  --resource "<storage-resource-id>" \
  --metric Transactions \
  --interval PT5M \
  --aggregation Total \
  --output table

az monitor diagnostic-settings list \
  --resource "<storage-resource-id>" \
  --output table

# Network verification.

az network watcher list \
  --output table

az network vnet show \
  --resource-group "<network-resource-group>" \
  --name "<vnet-name>" \
  --output table

az network nic list-effective-nsg \
  --resource-group "<network-resource-group>" \
  --name "<nic-name>" \
  --output table

az network nic show-effective-route-table \
  --resource-group "<network-resource-group>" \
  --name "<nic-name>" \
  --output table

# Resource Health alert verification.

az monitor activity-log alert show \
  --resource-group "<monitor-resource-group>" \
  --name "<resource-health-alert-name>" \
  --output table
```

```kusto
// Log Analytics verification.

Heartbeat
| where TimeGenerated > ago(24h)
| summarize LastHeartbeat=max(TimeGenerated) by Computer, OSType, _ResourceId
| order by LastHeartbeat desc

InsightsMetrics
| where TimeGenerated > ago(24h)
| summarize Count=count(), LastRecord=max(TimeGenerated) by Namespace, Name, Computer
| order by LastRecord desc

AzureMetrics
| where TimeGenerated > ago(24h)
| where ResourceProvider has_any ("MICROSOFT.STORAGE", "MICROSOFT.NETWORK", "MICROSOFT.COMPUTE")
| summarize Count=count(), LastRecord=max(TimeGenerated) by ResourceProvider, ResourceId, MetricName
| order by LastRecord desc

AzureActivity
| where TimeGenerated > ago(30d)
| where CategoryValue == "ResourceHealth"
| project TimeGenerated, ResourceGroup, ResourceProviderValue, Resource, OperationNameValue, ActivityStatusValue, Properties
| order by TimeGenerated desc
```

# Configure_VM_Storage_Network_Insights_And_Resource_Health_Rollback

| Change | Rollback Command / Action | Risk |
|---|---|---|
| Azure Monitor Agent installed on wrong VM | `az vm extension delete --resource-group "<vm-resource-group>" --vm-name "<vm-name>" --name AzureMonitorWindowsAgent` | Stops guest telemetry collection for that VM |
| Linux AMA installed on wrong VM | `az vm extension delete --resource-group "<vm-resource-group>" --vm-name "<vm-name>" --name AzureMonitorLinuxAgent` | Stops guest telemetry collection for that Linux VM |
| VM associated to wrong DCR | Remove the incorrect DCR association resource | Guest logs or metrics stop flowing through that DCR |
| Storage diagnostic setting sends too much data | Update diagnostic setting categories or delete setting | Reduces storage log visibility |
| Storage workbook customized incorrectly | Delete copied workbook or revert saved workbook changes | Removes custom view only |
| Connection Monitor test created incorrectly | Delete the connection monitor test | Removes connectivity monitoring |
| NSG flow logs enabled against wrong target | Disable or delete flow log configuration | Stops flow log collection |
| Resource Health alert too broad | Narrow scope or delete alert | May miss broader health events after narrowing |
| Workbook saved in wrong resource group | Delete workbook and save again to correct resource group | Deletes custom workbook view |
| Evidence files exported locally | Delete evidence folder | Removes local evidence only |

# Configure_VM_Storage_Network_Insights_And_Resource_Health_Failure_Checks

| Symptom | Likely Layer | First Check | Do Not Start With |
|---|---|---|---|
| VM host metrics visible but no guest data | AMA or DCR not configured | Check VM extension and DCR association | Recreating the VM |
| VM Insights page shows limited data | Enhanced monitoring not fully enabled | VM > Insights > Enable status | Changing alert rules |
| Heartbeat table empty | Agent not installed, no DCR, or workspace mismatch | `az vm extension list` and DCR association | Recreating workspace |
| Perf table empty | Guest performance counters not collected | Check DCR data sources | Reinstalling OS |
| Event table empty | Windows Event Logs not configured in DCR | Check DCR log data source | Changing diagnostic settings |
| Storage insights missing account | Wrong subscription or portal filter | Check subscription, resource group, and selected accounts | Recreating storage account |
| Storage metrics empty | No recent activity or wrong namespace | Generate test transaction and list metrics | Enabling random logs |
| Storage logs not in Log Analytics | Diagnostic setting not configured | `az monitor diagnostic-settings list` | Blaming Storage insights |
| High E2E latency but normal server latency | Client, network, or application path issue | Compare E2E and server latency | Rebuilding storage account |
| Client throttling errors | Storage workload or limits issue | Review ResponseType dimensions and transaction patterns | Disabling monitoring |
| Network Insights topology missing resource | Unsupported type, wrong subscription, or filter | Check Monitor > Insights > Networks filters | Recreating network |
| Effective NSG command fails | NIC name or resource group wrong | Confirm NIC resource ID | Editing NSG rules |
| Effective routes missing expected route | Route table not associated or propagated route absent | Check subnet route table association | Rebuilding VNet |
| Connection Monitor cannot create | Source VM, extension, or Network Watcher issue | Confirm source VM and region Network Watcher | Deleting VNet |
| Resource Health is Unknown | Resource state unsupported or deallocated | Check resource power/state and supportability | Opening Sev A ticket immediately |
| Resource Health is Unavailable | Platform or non-platform event | Open Resource Health event details | Rebooting without evidence |
| Resource Health alert not firing | Activity Log condition or scope wrong | Check Activity Log for ResourceHealth events | Recreating action group |
| Workbook query fails | Wrong table, wrong workspace, or syntax issue | Run query directly in Logs first | Deleting workbook |
| Alert view empty in Network Insights | No matching alerts or wrong scope | Check Monitor > Alerts scope | Rebuilding Network Insights |
| VM appears healthy but app is down | Guest app telemetry missing | Check guest logs, app logs, and endpoint tests | Assuming Azure platform issue |

# Configure_VM_Storage_Network_Insights_And_Resource_Health_Related_Labs

| Lab                            | Related Workbook                                                                 | Skill Proven                            |
| ------------------------------ | -------------------------------------------------------------------------------- | --------------------------------------- |
| Enable VM Insights on one VM   | `03_Configure_VM_Storage_Network_Insights_And_Resource_Health.md`                | VM guest monitoring                     |
| Validate Azure Monitor Agent   | `03_Configure_VM_Storage_Network_Insights_And_Resource_Health.md`                | Agent verification                      |
| Query VM guest data            | `03_Configure_VM_Storage_Network_Insights_And_Resource_Health.md`                | Log Analytics validation                |
| Review Storage insights        | `03_Configure_VM_Storage_Network_Insights_And_Resource_Health.md`                | Storage performance and capacity review |
| Query storage metrics          | `03_Configure_VM_Storage_Network_Insights_And_Resource_Health.md`                | Metric validation                       |
| Open Network Insights topology | `03_Configure_VM_Storage_Network_Insights_And_Resource_Health.md`                | Network topology visibility             |
| Validate effective NSG rules   | `03_Configure_VM_Storage_Network_Insights_And_Resource_Health.md`                | Network security troubleshooting        |
| Validate effective routes      | `03_Configure_VM_Storage_Network_Insights_And_Resource_Health.md`                | Routing troubleshooting                 |
| Create Resource Health alert   | `03_Configure_VM_Storage_Network_Insights_And_Resource_Health.md`                | Resource health event alerting          |
| Build resource health workbook | `03_Configure_VM_Storage_Network_Insights_And_Resource_Health.md`                | Operator dashboarding                   |
| Create metric alert for VM     | `02_Configure_Alerts_Action_Groups_Alert_Processing_Rules_And_Workbook_Views.md` | Alerting integration                    |
| Configure Backup monitoring    | `06_Configure_Backup_Reports_Alerts_And_Recovery_Validation.md`                  | Recovery monitoring                     |
| Troubleshoot missing telemetry | `07_Troubleshoot_Azure_Monitor_Backup_Restore_ASR_And_Alerting_Issues.md`        | Full observability troubleshooting      |