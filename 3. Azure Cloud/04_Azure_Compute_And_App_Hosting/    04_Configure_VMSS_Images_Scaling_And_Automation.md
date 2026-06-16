# 04_Configure_VMSS_Images_Scaling_And_Automation

# 04_Configure_VMSS_Images_Scaling_And_Automation_Index

04_Configure_VMSS_Images_Scaling_And_Automation.md  
Configure_VMSS_Images_Scaling_And_Automation  
Configure_VMSS_Images_Scaling_And_Automation_Source_Basis  
Configure_VMSS_Images_Scaling_And_Automation_Mental_Model  
Configure_VMSS_Images_Scaling_And_Automation_Planning_Table  
Configure_VMSS_Images_Scaling_And_Automation_Configuration_Checklist  
Configure_VMSS_Images_Scaling_And_Automation_Portal_Skeleton  
Configure_VMSS_Images_Scaling_And_Automation_AzureCLI_Skeleton  
Configure_VMSS_Images_Scaling_And_Automation_PowerShell_Skeleton  
Configure_VMSS_Images_Scaling_And_Automation_Bicep_Skeleton  
Configure_VMSS_Images_Scaling_And_Automation_Image_Source_Skeleton  
Configure_VMSS_Images_Scaling_And_Automation_Manual_And_Autoscale_Skeleton  
Configure_VMSS_Images_Scaling_And_Automation_Instance_Operations_Skeleton  
Configure_VMSS_Images_Scaling_And_Automation_Upgrade_And_Repair_Skeleton  
Configure_VMSS_Images_Scaling_And_Automation_Verification_Commands  
Configure_VMSS_Images_Scaling_And_Automation_Rollback  
Configure_VMSS_Images_Scaling_And_Automation_Failure_Checks  
Configure_VMSS_Images_Scaling_And_Automation_Related_Labs  

# Configure_VMSS_Images_Scaling_And_Automation_Source_Basis

| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Azure Virtual Machine Scale Sets overview | VMSS purpose, scale-out compute, instance management, and orchestration models |
| Microsoft Learn | VMSS orchestration modes | Flexible and Uniform orchestration behavior |
| Microsoft Learn | Create VMSS with portal, Azure CLI, PowerShell, Bicep, and ARM | Deployment paths for repeatable VMSS buildout |
| Microsoft Learn | VMSS autoscale | Azure Monitor autoscale rules, minimum capacity, maximum capacity, default capacity, scale out, and scale in |
| Microsoft Learn | VMSS custom images | Azure Compute Gallery image definitions and image versions |
| Microsoft Learn | VMSS application deployment | Extensions, Custom Script Extension, and VM applications |
| Microsoft Learn | VMSS upgrade policies | Manual, automatic, rolling upgrades, reimage, and model update behavior |
| Microsoft Learn | VMSS instance protection | Protect specific instances from scale-in or scale-set actions |
| Microsoft Learn | VMSS automatic repairs | Health-based instance repair behavior |
| Microsoft Learn | VMSS availability zones | Zonal and zone-spread scale set placement |
| Microsoft Learn | Azure Load Balancer | Backend pool and inbound access patterns for VMSS workloads |
| Microsoft Learn | Azure Monitor metrics | CPU, network, disk, and platform metrics used by autoscale rules |
| Azure operational practice | Golden image lifecycle | Build image, publish image version, deploy VMSS from image, roll forward, and retire old image |
| Azure operational practice | VMSS operations | Create, inspect, scale, run command, update model, upgrade instances, reimage, restart, and delete |
| Azure operational practice | Evidence capture | Record capacity, orchestration mode, image reference, autoscale settings, instance view, upgrade policy, and repair settings |

# Configure_VMSS_Images_Scaling_And_Automation_Mental_Model

| Concept | Operational Meaning |
|---|---|
| VMSS | Virtual Machine Scale Set, a group of VM instances managed as a scalable compute pool |
| Scale set model | Desired configuration used to create or update scale set instances |
| Instance | One VM created and managed inside the scale set |
| Capacity | Number of instances the scale set should run |
| Manual scale | Operator directly sets instance count |
| Autoscale | Azure Monitor changes instance count based on rules or schedule |
| Minimum capacity | Lowest number of instances autoscale keeps running |
| Maximum capacity | Highest number of instances autoscale can create |
| Default capacity | Starting or fallback instance count |
| Scale out | Add instances |
| Scale in | Remove instances |
| Flexible orchestration | VMSS mode that behaves more like grouped independent VMs, useful for broader VM compatibility |
| Uniform orchestration | VMSS mode where instances are more identical and model-driven |
| Image | OS and baseline application source used to create instances |
| Marketplace image | Microsoft or vendor-provided image such as Ubuntu or Windows Server |
| Custom image | Organization-built image captured from a prepared VM |
| Azure Compute Gallery | Image library for sharing, versioning, and replicating custom images |
| Image definition | Logical image family in Azure Compute Gallery |
| Image version | Specific immutable build of an image definition |
| Upgrade policy | Controls how model or image changes reach existing instances |
| Manual upgrade | Operator chooses when to update instances |
| Automatic upgrade | Platform updates instances automatically according to policy |
| Rolling upgrade | Updates instances in batches to reduce downtime |
| Reimage | Rebuilds an instance from the scale set image while keeping the instance identity in the set |
| Instance protection | Prevents important instances from scale-in or certain scale-set operations |
| Automatic repairs | Platform replaces unhealthy instances after health checks fail |
| Custom Script Extension | Post-deployment script execution on scale set instances |
| Run command | Control-plane script execution against a specific VMSS instance |
| First rule | VMSS is for repeatable cattle-style compute, not hand-crafted pet servers |
| Blunt rule | If the image, extension, or bootstrap process is weak, scaling just creates broken machines faster |

# Configure_VMSS_Images_Scaling_And_Automation_Planning_Table

| Item | Example | Decision |
|---|---|---|
| Tenant | `contoso.onmicrosoft.com` | `<tenant-name>` |
| Subscription ID | `00000000-0000-0000-0000-000000000000` | `<subscription-id>` |
| Resource group | `rg-compute-lab-01` | `<resource-group-name>` |
| Location | `eastus` | `<azure-region>` |
| VMSS name | `vmss-web-lab-01` | `<vmss-name>` |
| Orchestration mode | `Flexible` | `<Flexible / Uniform>` |
| Instance count | `2` | `<instance-count>` |
| Minimum capacity | `2` | `<autoscale-min>` |
| Default capacity | `2` | `<autoscale-default>` |
| Maximum capacity | `5` | `<autoscale-max>` |
| VM size | `Standard_B2s` | `<vm-size>` |
| OS type | `Linux` | `<Linux / Windows>` |
| Image source | `Marketplace` | `<Marketplace / Azure Compute Gallery / managed image>` |
| Marketplace image | `Ubuntu2204` | `<marketplace-image>` |
| Compute Gallery name | `galcompute01` | `<gallery-name>` |
| Image definition | `imgdef-ubuntu-web` | `<image-definition>` |
| Image version | `1.0.0` | `<image-version>` |
| Admin username | `azureuser` | `<admin-username>` |
| Authentication | `SSH key` | `<ssh-key / password / Entra login>` |
| VNet name | `vnet-compute-lab-01` | `<vnet-name>` |
| Subnet name | `snet-vmss-01` | `<subnet-name>` |
| Subnet prefix | `10.20.2.0/24` | `<subnet-prefix>` |
| NSG name | `nsg-snet-vmss-01` | `<nsg-name>` |
| Load balancer required | `Yes` | `<yes-no>` |
| Load balancer name | `lb-vmss-web-01` | `<load-balancer-name>` |
| Backend pool | `be-vmss-web-01` | `<backend-pool-name>` |
| Health probe | `tcp-80` | `<probe-name>` |
| Application port | `80` | `<app-port>` |
| Upgrade policy | `Manual for lab` | `<manual / automatic / rolling>` |
| Autoscale metric | `Percentage CPU` | `<metric-name>` |
| Scale-out rule | `CPU > 70 for 5 minutes, add 1` | `<scale-out-rule>` |
| Scale-in rule | `CPU < 30 for 10 minutes, remove 1` | `<scale-in-rule>` |
| Extension | `CustomScript` | `<extension-name>` |
| Bootstrap command | `install nginx` | `<bootstrap-command>` |
| Automatic repairs | `Disabled in lab` | `<enabled-disabled>` |
| Evidence path | `.\evidence\04-vmss-images-scaling-automation` | `<evidence-path>` |
| Rollback posture | `Scale to 0 or delete lab RG` | `<rollback-plan>` |

# Configure_VMSS_Images_Scaling_And_Automation_Configuration_Checklist

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm Azure context | Admin Workstation | `az account show -o table` | Correct tenant and subscription are active |
| 2 | Set subscription | Admin Workstation | `az account set --subscription "<subscription-id>"` | CLI targets correct subscription |
| 3 | Confirm required providers | Admin Workstation | `az provider show --namespace Microsoft.Compute --query registrationState -o tsv` | Compute provider is registered |
| 4 | Register required providers if needed | Admin Workstation | `az provider register --namespace Microsoft.Compute` | Provider registration begins |
| 5 | Confirm resource group | Admin Workstation | `az group show --name "<resource-group-name>" -o table` | Resource group exists |
| 6 | Create VMSS subnet if needed | Admin Workstation | `az network vnet subnet create --resource-group "<resource-group-name>" --vnet-name "<vnet-name>" --name "<subnet-name>" --address-prefixes "<subnet-prefix>"` | VMSS subnet exists |
| 7 | Create NSG for VMSS subnet | Admin Workstation | `az network nsg create --resource-group "<resource-group-name>" --name "<nsg-name>" --location "<azure-region>"` | NSG exists |
| 8 | Associate NSG to subnet | Admin Workstation | `az network vnet subnet update --resource-group "<resource-group-name>" --vnet-name "<vnet-name>" --name "<subnet-name>" --network-security-group "<nsg-name>"` | Subnet has NSG |
| 9 | Decide image source | Operator | `Record Marketplace, Azure Compute Gallery, or managed image` | Image source is documented |
| 10 | Confirm image availability | Admin Workstation | `az vm image list --offer UbuntuServer --all -o table` | Candidate image is visible |
| 11 | Create Azure Compute Gallery if using custom images | Admin Workstation | `az sig create --resource-group "<resource-group-name>" --gallery-name "<gallery-name>" --location "<azure-region>"` | Gallery exists |
| 12 | Create image definition if using custom image | Admin Workstation | `az sig image-definition create --resource-group "<resource-group-name>" --gallery-name "<gallery-name>" --gallery-image-definition "<image-definition>" --publisher "<publisher>" --offer "<offer>" --sku "<sku>" --os-type Linux --hyper-v-generation V2` | Image definition exists |
| 13 | Create image version if using custom image | Admin Workstation | `az sig image-version create --resource-group "<resource-group-name>" --gallery-name "<gallery-name>" --gallery-image-definition "<image-definition>" --gallery-image-version "<image-version>" --target-regions "<azure-region>" --managed-image "<managed-image-id>"` | Image version exists |
| 14 | Create VMSS from marketplace image | Admin Workstation | `az vmss create --resource-group "<resource-group-name>" --name "<vmss-name>" --orchestration-mode Flexible --image Ubuntu2204 --vm-sku "<vm-size>" --instance-count 2 --admin-username "<admin-username>" --generate-ssh-keys --vnet-name "<vnet-name>" --subnet "<subnet-name>"` | VMSS is created |
| 15 | Create VMSS from gallery image if needed | Admin Workstation | `az vmss create --resource-group "<resource-group-name>" --name "<vmss-name>" --orchestration-mode Flexible --image "<gallery-image-version-id>" --vm-sku "<vm-size>" --instance-count 2 --admin-username "<admin-username>" --generate-ssh-keys --vnet-name "<vnet-name>" --subnet "<subnet-name>"` | VMSS uses gallery image |
| 16 | Confirm VMSS model | Admin Workstation | `az vmss show --resource-group "<resource-group-name>" --name "<vmss-name>" -o json` | VMSS model is visible |
| 17 | Confirm VMSS instances | Admin Workstation | `az vmss list-instances --resource-group "<resource-group-name>" --name "<vmss-name>" -o table` | Instances are listed |
| 18 | Confirm instance view | Admin Workstation | `az vmss get-instance-view --resource-group "<resource-group-name>" --name "<vmss-name>" -o json` | VMSS runtime state is visible |
| 19 | Manually scale out | Admin Workstation | `az vmss scale --resource-group "<resource-group-name>" --name "<vmss-name>" --new-capacity 3` | Instance count increases |
| 20 | Manually scale in | Admin Workstation | `az vmss scale --resource-group "<resource-group-name>" --name "<vmss-name>" --new-capacity 2` | Instance count decreases |
| 21 | Create autoscale profile | Admin Workstation | `az monitor autoscale create --resource-group "<resource-group-name>" --resource "<vmss-name>" --resource-type Microsoft.Compute/virtualMachineScaleSets --name "<autoscale-name>" --min-count 2 --max-count 5 --count 2` | Autoscale setting exists |
| 22 | Add CPU scale-out rule | Admin Workstation | `az monitor autoscale rule create --resource-group "<resource-group-name>" --autoscale-name "<autoscale-name>" --condition "Percentage CPU > 70 avg 5m" --scale out 1` | Scale-out rule exists |
| 23 | Add CPU scale-in rule | Admin Workstation | `az monitor autoscale rule create --resource-group "<resource-group-name>" --autoscale-name "<autoscale-name>" --condition "Percentage CPU < 30 avg 10m" --scale in 1` | Scale-in rule exists |
| 24 | Confirm autoscale settings | Admin Workstation | `az monitor autoscale show --resource-group "<resource-group-name>" --name "<autoscale-name>" -o json` | Autoscale profile and rules are visible |
| 25 | Install VMSS Custom Script Extension | Admin Workstation | `az vmss extension set --resource-group "<resource-group-name>" --vmss-name "<vmss-name>" --publisher Microsoft.Azure.Extensions --name CustomScript --settings '{"commandToExecute":"echo vmss-extension-test"}'` | Extension is added to model |
| 26 | Apply model changes to instances if needed | Admin Workstation | `az vmss update-instances --resource-group "<resource-group-name>" --name "<vmss-name>" --instance-ids "*"` | Instances receive model change when supported |
| 27 | Run command on an instance | Admin Workstation | `az vmss run-command invoke --resource-group "<resource-group-name>" --name "<vmss-name>" --instance-id "<instance-id>" --command-id RunShellScript --scripts "hostname; uptime"` | Instance responds |
| 28 | Reimage one instance | Admin Workstation | `az vmss reimage --resource-group "<resource-group-name>" --name "<vmss-name>" --instance-id "<instance-id>"` | Instance is rebuilt from image |
| 29 | Restart one instance | Admin Workstation | `az vmss restart --resource-group "<resource-group-name>" --name "<vmss-name>" --instance-ids "<instance-id>"` | Instance restarts |
| 30 | Configure upgrade policy | Admin Workstation | `az vmss update --resource-group "<resource-group-name>" --name "<vmss-name>" --set upgradePolicy.mode=Manual` | Upgrade mode is set |
| 31 | Confirm upgrade policy | Admin Workstation | `az vmss show --resource-group "<resource-group-name>" --name "<vmss-name>" --query "upgradePolicy" -o json` | Upgrade policy is visible |
| 32 | Confirm health and instance status | Admin Workstation | `az vmss list-instances --resource-group "<resource-group-name>" --name "<vmss-name>" --expand instanceView -o table` | Instance status is visible |
| 33 | Capture final VMSS evidence | Admin Workstation | `az vmss show --resource-group "<resource-group-name>" --name "<vmss-name>" -o json > .\evidence\vmss-final.json` | VMSS evidence is saved |
| 34 | Capture final autoscale evidence | Admin Workstation | `az monitor autoscale show --resource-group "<resource-group-name>" --name "<autoscale-name>" -o json > .\evidence\autoscale-final.json` | Autoscale evidence is saved |
| 35 | Document image, capacity, scale rules, extensions, and upgrade policy | Operator | `Record final build state` | Workbook evidence is complete |

# Configure_VMSS_Images_Scaling_And_Automation_Portal_Skeleton

```text
Purpose:
Create and operate a VM scale set through the Azure portal.

Portal path:
Azure portal
> Virtual machine scale sets
> Create

Basics:
1. Subscription: <subscription-name>
2. Resource group: <resource-group-name>
3. Scale set name: <vmss-name>
4. Region: <azure-region>
5. Orchestration mode:
   - Flexible for most modern admin lab use
   - Uniform when strict identical instance model is required
6. Availability zone:
   - None for basic lab
   - Zone 1, 2, or 3 for zonal placement
   - Multiple zones for zone-spread design if supported
7. Image:
   - Marketplace image for baseline lab
   - Azure Compute Gallery image for golden-image workflow
8. Size: <vm-size>
9. Authentication:
   - Linux: SSH key preferred
   - Windows: secure password or managed login path

Instances:
1. Initial instance count: <instance-count>
2. Scaling mode:
   - Manual for first validation
   - Autoscale after health and app path are proven

Networking:
1. Virtual network: <vnet-name>
2. Subnet: <subnet-name>
3. Public IP behavior: avoid direct public IP per instance unless lab requires it
4. Load balancing: configure if inbound app traffic is required
5. NSG: use subnet NSG or explicit inbound rules
6. Avoid broad SSH or RDP exposure

Management:
1. Boot diagnostics: enable
2. Identity: enable only if workload requires it
3. Automatic repairs: enable only after health probe is valid
4. Upgrade policy: manual for first lab

Advanced:
1. Add custom data or cloud-init if needed.
2. Add extensions if needed.
3. Keep bootstrap simple and repeatable.

Tags:
1. Environment: Lab
2. Workload: Compute
3. ManagedBy: Portal
4. Workbook: 04_Configure_VMSS_Images_Scaling_And_Automation

Post-create validation:
1. Open the VMSS resource.
2. Review Instances.
3. Review Scaling.
4. Review Extensions + applications.
5. Review Availability and upgrade settings.
6. Review Activity log.
7. Confirm app path if load balanced.
```

# Configure_VMSS_Images_Scaling_And_Automation_AzureCLI_Skeleton

```bash
# Run from Azure Cloud Shell or local Azure CLI.
# Purpose: create VMSS, configure images, scale rules, extensions, and instance operations.

# -----------------------------
# Variables
# -----------------------------

export SUBSCRIPTION_ID="<subscription-id>"
export LOCATION="eastus"
export RESOURCE_GROUP="rg-compute-lab-01"

export VNET_NAME="vnet-compute-lab-01"
export VNET_PREFIX="10.20.0.0/16"
export SUBNET_NAME="snet-vmss-01"
export SUBNET_PREFIX="10.20.2.0/24"
export NSG_NAME="nsg-snet-vmss-01"

export VMSS_NAME="vmss-web-lab-01"
export VM_SIZE="Standard_B2s"
export INSTANCE_COUNT="2"
export ADMIN_USERNAME="azureuser"
export MARKETPLACE_IMAGE="Ubuntu2204"

export AUTOSCALE_NAME="autoscale-vmss-web-lab-01"
export AUTOSCALE_MIN="2"
export AUTOSCALE_DEFAULT="2"
export AUTOSCALE_MAX="5"

export EVIDENCE_PATH="./evidence/04-vmss-images-scaling-automation"

mkdir -p "$EVIDENCE_PATH"

# -----------------------------
# Context
# -----------------------------

az login

az account set \
  --subscription "$SUBSCRIPTION_ID"

az account show \
  --output table |
  tee "$EVIDENCE_PATH/account-context.txt"

# -----------------------------
# Provider registration
# -----------------------------

for provider in Microsoft.Compute Microsoft.Network Microsoft.Insights Microsoft.Storage Microsoft.Resources
do
  az provider register --namespace "$provider"
done

for provider in Microsoft.Compute Microsoft.Network Microsoft.Insights Microsoft.Storage Microsoft.Resources
do
  az provider show \
    --namespace "$provider" \
    --query "{namespace:namespace, registrationState:registrationState}" \
    --output table
done | tee "$EVIDENCE_PATH/provider-registration.txt"

# -----------------------------
# Resource group
# -----------------------------

az group create \
  --name "$RESOURCE_GROUP" \
  --location "$LOCATION" |
  tee "$EVIDENCE_PATH/resource-group-create.json"

# -----------------------------
# Network baseline
# -----------------------------

az network vnet create \
  --resource-group "$RESOURCE_GROUP" \
  --location "$LOCATION" \
  --name "$VNET_NAME" \
  --address-prefixes "$VNET_PREFIX" \
  --subnet-name "$SUBNET_NAME" \
  --subnet-prefixes "$SUBNET_PREFIX" |
  tee "$EVIDENCE_PATH/vnet-create.json"

az network nsg create \
  --resource-group "$RESOURCE_GROUP" \
  --location "$LOCATION" \
  --name "$NSG_NAME" |
  tee "$EVIDENCE_PATH/nsg-create.json"

az network vnet subnet update \
  --resource-group "$RESOURCE_GROUP" \
  --vnet-name "$VNET_NAME" \
  --name "$SUBNET_NAME" \
  --network-security-group "$NSG_NAME" |
  tee "$EVIDENCE_PATH/subnet-nsg-association.json"

# Optional application inbound rule for HTTP lab traffic.
# Scope this to approved source ranges in real environments.

az network nsg rule create \
  --resource-group "$RESOURCE_GROUP" \
  --nsg-name "$NSG_NAME" \
  --name "Allow-HTTP-Lab" \
  --priority 1000 \
  --access Allow \
  --direction Inbound \
  --protocol Tcp \
  --source-address-prefixes "<approved-source-prefix>" \
  --source-port-ranges "*" \
  --destination-address-prefixes "*" \
  --destination-port-ranges 80 |
  tee "$EVIDENCE_PATH/nsg-http-rule.json"

# -----------------------------
# Create VMSS from marketplace image
# -----------------------------

az vmss create \
  --resource-group "$RESOURCE_GROUP" \
  --location "$LOCATION" \
  --name "$VMSS_NAME" \
  --orchestration-mode Flexible \
  --image "$MARKETPLACE_IMAGE" \
  --vm-sku "$VM_SIZE" \
  --instance-count "$INSTANCE_COUNT" \
  --admin-username "$ADMIN_USERNAME" \
  --generate-ssh-keys \
  --vnet-name "$VNET_NAME" \
  --subnet "$SUBNET_NAME" \
  --upgrade-policy-mode Manual \
  --tags Environment=Lab Workload=Compute ManagedBy=AzureCLI Workbook=04_Configure_VMSS_Images_Scaling_And_Automation |
  tee "$EVIDENCE_PATH/vmss-create.json"

# -----------------------------
# Inspect VMSS
# -----------------------------

az vmss show \
  --resource-group "$RESOURCE_GROUP" \
  --name "$VMSS_NAME" \
  --output json |
  tee "$EVIDENCE_PATH/vmss-show.json"

az vmss get-instance-view \
  --resource-group "$RESOURCE_GROUP" \
  --name "$VMSS_NAME" \
  --output json |
  tee "$EVIDENCE_PATH/vmss-instance-view.json"

az vmss list-instances \
  --resource-group "$RESOURCE_GROUP" \
  --name "$VMSS_NAME" \
  --output table |
  tee "$EVIDENCE_PATH/vmss-instances.txt"

# -----------------------------
# Manual scale test
# -----------------------------

az vmss scale \
  --resource-group "$RESOURCE_GROUP" \
  --name "$VMSS_NAME" \
  --new-capacity 3 |
  tee "$EVIDENCE_PATH/vmss-scale-out.json"

az vmss list-instances \
  --resource-group "$RESOURCE_GROUP" \
  --name "$VMSS_NAME" \
  --output table |
  tee "$EVIDENCE_PATH/vmss-instances-after-scale-out.txt"

az vmss scale \
  --resource-group "$RESOURCE_GROUP" \
  --name "$VMSS_NAME" \
  --new-capacity "$INSTANCE_COUNT" |
  tee "$EVIDENCE_PATH/vmss-scale-in.json"

# -----------------------------
# Autoscale baseline
# -----------------------------

az monitor autoscale create \
  --resource-group "$RESOURCE_GROUP" \
  --resource "$VMSS_NAME" \
  --resource-type Microsoft.Compute/virtualMachineScaleSets \
  --name "$AUTOSCALE_NAME" \
  --min-count "$AUTOSCALE_MIN" \
  --max-count "$AUTOSCALE_MAX" \
  --count "$AUTOSCALE_DEFAULT" |
  tee "$EVIDENCE_PATH/autoscale-create.json"

az monitor autoscale rule create \
  --resource-group "$RESOURCE_GROUP" \
  --autoscale-name "$AUTOSCALE_NAME" \
  --condition "Percentage CPU > 70 avg 5m" \
  --scale out 1 |
  tee "$EVIDENCE_PATH/autoscale-rule-scale-out.json"

az monitor autoscale rule create \
  --resource-group "$RESOURCE_GROUP" \
  --autoscale-name "$AUTOSCALE_NAME" \
  --condition "Percentage CPU < 30 avg 10m" \
  --scale in 1 |
  tee "$EVIDENCE_PATH/autoscale-rule-scale-in.json"

az monitor autoscale show \
  --resource-group "$RESOURCE_GROUP" \
  --name "$AUTOSCALE_NAME" \
  --output json |
  tee "$EVIDENCE_PATH/autoscale-show.json"

# -----------------------------
# Extension, Linux Custom Script
# -----------------------------

az vmss extension set \
  --resource-group "$RESOURCE_GROUP" \
  --vmss-name "$VMSS_NAME" \
  --publisher Microsoft.Azure.Extensions \
  --name CustomScript \
  --settings '{"commandToExecute":"sudo apt-get update && sudo apt-get install -y nginx && echo vmss-lab | sudo tee /var/www/html/index.html"}' |
  tee "$EVIDENCE_PATH/vmss-custom-script-extension.json"

az vmss extension list \
  --resource-group "$RESOURCE_GROUP" \
  --vmss-name "$VMSS_NAME" \
  --output table |
  tee "$EVIDENCE_PATH/vmss-extension-list.txt"

# Apply model change to instances when required.
az vmss update-instances \
  --resource-group "$RESOURCE_GROUP" \
  --name "$VMSS_NAME" \
  --instance-ids "*" |
  tee "$EVIDENCE_PATH/vmss-update-instances.json"

# -----------------------------
# Instance operations
# -----------------------------

INSTANCE_ID=$(az vmss list-instances \
  --resource-group "$RESOURCE_GROUP" \
  --name "$VMSS_NAME" \
  --query "[0].instanceId" \
  --output tsv)

echo "$INSTANCE_ID" | tee "$EVIDENCE_PATH/sample-instance-id.txt"

az vmss run-command invoke \
  --resource-group "$RESOURCE_GROUP" \
  --name "$VMSS_NAME" \
  --instance-id "$INSTANCE_ID" \
  --command-id RunShellScript \
  --scripts "hostname; uptime; systemctl status nginx --no-pager || true" |
  tee "$EVIDENCE_PATH/vmss-run-command.json"

az vmss restart \
  --resource-group "$RESOURCE_GROUP" \
  --name "$VMSS_NAME" \
  --instance-ids "$INSTANCE_ID" |
  tee "$EVIDENCE_PATH/vmss-instance-restart.json"

# Reimage only if you intend to rebuild the instance from the scale set image.
# az vmss reimage \
#   --resource-group "$RESOURCE_GROUP" \
#   --name "$VMSS_NAME" \
#   --instance-id "$INSTANCE_ID"

# -----------------------------
# Upgrade policy evidence
# -----------------------------

az vmss show \
  --resource-group "$RESOURCE_GROUP" \
  --name "$VMSS_NAME" \
  --query "{orchestrationMode:orchestrationMode,upgradePolicy:upgradePolicy,sku:sku,virtualMachineProfile:virtualMachineProfile.storageProfile.imageReference}" \
  --output json |
  tee "$EVIDENCE_PATH/vmss-upgrade-image-policy.json"

# -----------------------------
# Metrics and activity evidence
# -----------------------------

VMSS_ID=$(az vmss show \
  --resource-group "$RESOURCE_GROUP" \
  --name "$VMSS_NAME" \
  --query id \
  --output tsv)

az monitor metrics list \
  --resource "$VMSS_ID" \
  --metric "Percentage CPU" \
  --interval PT1M \
  --output table |
  tee "$EVIDENCE_PATH/vmss-cpu-metric.txt"

az monitor activity-log list \
  --resource-group "$RESOURCE_GROUP" \
  --max-events 25 \
  --output table |
  tee "$EVIDENCE_PATH/activity-log.txt"

# -----------------------------
# Final evidence
# -----------------------------

az vmss show \
  --resource-group "$RESOURCE_GROUP" \
  --name "$VMSS_NAME" \
  --output json |
  tee "$EVIDENCE_PATH/vmss-final.json"

az vmss list-instances \
  --resource-group "$RESOURCE_GROUP" \
  --name "$VMSS_NAME" \
  --expand instanceView \
  --output json |
  tee "$EVIDENCE_PATH/vmss-instances-final.json"
```

# Configure_VMSS_Images_Scaling_And_Automation_AzureCLI_Compute_Gallery_Image_Skeleton

```bash
# Purpose:
# Create Azure Compute Gallery image references for VMSS golden-image deployment.
# This assumes a managed image already exists.

export RESOURCE_GROUP="rg-compute-lab-01"
export LOCATION="eastus"

export GALLERY_NAME="galcompute01"
export IMAGE_DEFINITION="imgdef-ubuntu-web"
export IMAGE_VERSION="1.0.0"

export PUBLISHER="Contoso"
export OFFER="UbuntuWeb"
export SKU="22_04"

export MANAGED_IMAGE_ID="<managed-image-resource-id>"

export EVIDENCE_PATH="./evidence/04-vmss-images-scaling-automation"

mkdir -p "$EVIDENCE_PATH"

# Create gallery.
az sig create \
  --resource-group "$RESOURCE_GROUP" \
  --gallery-name "$GALLERY_NAME" \
  --location "$LOCATION" |
  tee "$EVIDENCE_PATH/gallery-create.json"

# Create image definition.
az sig image-definition create \
  --resource-group "$RESOURCE_GROUP" \
  --gallery-name "$GALLERY_NAME" \
  --gallery-image-definition "$IMAGE_DEFINITION" \
  --publisher "$PUBLISHER" \
  --offer "$OFFER" \
  --sku "$SKU" \
  --os-type Linux \
  --hyper-v-generation V2 |
  tee "$EVIDENCE_PATH/gallery-image-definition-create.json"

# Create image version.
az sig image-version create \
  --resource-group "$RESOURCE_GROUP" \
  --gallery-name "$GALLERY_NAME" \
  --gallery-image-definition "$IMAGE_DEFINITION" \
  --gallery-image-version "$IMAGE_VERSION" \
  --target-regions "$LOCATION" \
  --managed-image "$MANAGED_IMAGE_ID" |
  tee "$EVIDENCE_PATH/gallery-image-version-create.json"

# Get image version ID for VMSS deployment.
GALLERY_IMAGE_VERSION_ID=$(az sig image-version show \
  --resource-group "$RESOURCE_GROUP" \
  --gallery-name "$GALLERY_NAME" \
  --gallery-image-definition "$IMAGE_DEFINITION" \
  --gallery-image-version "$IMAGE_VERSION" \
  --query id \
  --output tsv)

echo "$GALLERY_IMAGE_VERSION_ID" | tee "$EVIDENCE_PATH/gallery-image-version-id.txt"

# Use this value as:
# az vmss create --image "$GALLERY_IMAGE_VERSION_ID" ...
```

# Configure_VMSS_Images_Scaling_And_Automation_AzureCLI_Windows_VMSS_Skeleton

```bash
# Purpose:
# Create a Windows VMSS baseline.
# Use private access or Bastion. Avoid broad public RDP exposure.

export SUBSCRIPTION_ID="<subscription-id>"
export LOCATION="eastus"
export RESOURCE_GROUP="rg-compute-lab-01"

export VNET_NAME="vnet-compute-lab-01"
export SUBNET_NAME="snet-vmss-01"

export VMSS_NAME="vmss-win-lab-01"
export VM_SIZE="Standard_B2s"
export INSTANCE_COUNT="2"
export ADMIN_USERNAME="azureadmin"
export IMAGE="Win2022Datacenter"

read -s ADMIN_PASSWORD

az account set \
  --subscription "$SUBSCRIPTION_ID"

az vmss create \
  --resource-group "$RESOURCE_GROUP" \
  --location "$LOCATION" \
  --name "$VMSS_NAME" \
  --orchestration-mode Flexible \
  --image "$IMAGE" \
  --vm-sku "$VM_SIZE" \
  --instance-count "$INSTANCE_COUNT" \
  --admin-username "$ADMIN_USERNAME" \
  --admin-password "$ADMIN_PASSWORD" \
  --vnet-name "$VNET_NAME" \
  --subnet "$SUBNET_NAME" \
  --upgrade-policy-mode Manual \
  --tags Environment=Lab Workload=Compute ManagedBy=AzureCLI Workbook=04_Configure_VMSS_Images_Scaling_And_Automation

INSTANCE_ID=$(az vmss list-instances \
  --resource-group "$RESOURCE_GROUP" \
  --name "$VMSS_NAME" \
  --query "[0].instanceId" \
  --output tsv)

az vmss run-command invoke \
  --resource-group "$RESOURCE_GROUP" \
  --name "$VMSS_NAME" \
  --instance-id "$INSTANCE_ID" \
  --command-id RunPowerShellScript \
  --scripts "hostname; Get-ComputerInfo | Select-Object CsName,WindowsProductName,WindowsVersion"
```

# Configure_VMSS_Images_Scaling_And_Automation_PowerShell_Skeleton

```powershell
# Run from PowerShell with Az module installed.
# Purpose: create and operate a VMSS baseline with scale and automation evidence.

# -----------------------------
# Variables
# -----------------------------

$SubscriptionId = "<subscription-id>"
$Location = "eastus"
$ResourceGroupName = "rg-compute-lab-01"

$VNetName = "vnet-compute-lab-01"
$VNetPrefix = "10.20.0.0/16"
$SubnetName = "snet-vmss-01"
$SubnetPrefix = "10.20.2.0/24"
$NsgName = "nsg-snet-vmss-01"

$VmssName = "vmss-web-lab-01"
$VmSize = "Standard_B2s"
$InstanceCount = 2
$AdminUsername = "azureuser"
$SshPublicKeyPath = "$HOME\.ssh\id_rsa.pub"

$EvidencePath = ".\evidence\04-vmss-images-scaling-automation"

New-Item -ItemType Directory -Force -Path $EvidencePath

# -----------------------------
# Context
# -----------------------------

Connect-AzAccount

Set-AzContext -SubscriptionId $SubscriptionId

Get-AzContext |
  Tee-Object (Join-Path $EvidencePath "az-context.txt")

# -----------------------------
# Resource group
# -----------------------------

New-AzResourceGroup `
  -Name $ResourceGroupName `
  -Location $Location `
  -Force |
  Tee-Object (Join-Path $EvidencePath "resource-group.txt")

# -----------------------------
# Network baseline
# -----------------------------

$SubnetConfig = New-AzVirtualNetworkSubnetConfig `
  -Name $SubnetName `
  -AddressPrefix $SubnetPrefix

$VNet = New-AzVirtualNetwork `
  -ResourceGroupName $ResourceGroupName `
  -Location $Location `
  -Name $VNetName `
  -AddressPrefix $VNetPrefix `
  -Subnet $SubnetConfig

$Nsg = New-AzNetworkSecurityGroup `
  -ResourceGroupName $ResourceGroupName `
  -Location $Location `
  -Name $NsgName

$Subnet = Get-AzVirtualNetworkSubnetConfig `
  -VirtualNetwork $VNet `
  -Name $SubnetName

$Subnet.NetworkSecurityGroup = $Nsg

$VNet | Set-AzVirtualNetwork

# -----------------------------
# SSH key
# -----------------------------

if (-not (Test-Path $SshPublicKeyPath)) {
  ssh-keygen -t rsa -b 4096 -f "$HOME\.ssh\id_rsa" -N '""'
}

$SshPublicKey = Get-Content $SshPublicKeyPath -Raw

# -----------------------------
# VMSS creation, simplified baseline
# -----------------------------
# Az PowerShell VMSS build flows are more verbose than CLI.
# Use CLI or Bicep for clean repeatable lab deployment when possible.

$VmssConfig = New-AzVmssConfig `
  -Location $Location `
  -SkuCapacity $InstanceCount `
  -SkuName $VmSize `
  -UpgradePolicyMode Manual `
  -OrchestrationMode Flexible

$VmssConfig = Set-AzVmssStorageProfile `
  -VirtualMachineScaleSet $VmssConfig `
  -ImageReferencePublisher "Canonical" `
  -ImageReferenceOffer "0001-com-ubuntu-server-jammy" `
  -ImageReferenceSku "22_04-lts-gen2" `
  -ImageReferenceVersion "latest" `
  -OsDiskCreateOption FromImage `
  -OsDiskCaching ReadWrite

$VmssConfig = Set-AzVmssOsProfile `
  -VirtualMachineScaleSet $VmssConfig `
  -ComputerNamePrefix "vmssweb" `
  -AdminUsername $AdminUsername `
  -LinuxConfigurationDisablePasswordAuthentication $true

$VmssConfig = Add-AzVmssSshPublicKey `
  -VirtualMachineScaleSet $VmssConfig `
  -Path "/home/$AdminUsername/.ssh/authorized_keys" `
  -KeyData $SshPublicKey

$VNet = Get-AzVirtualNetwork `
  -ResourceGroupName $ResourceGroupName `
  -Name $VNetName

$Subnet = Get-AzVirtualNetworkSubnetConfig `
  -VirtualNetwork $VNet `
  -Name $SubnetName

$IpConfig = New-AzVmssIpConfig `
  -Name "ipconfig1" `
  -SubnetId $Subnet.Id

$NetworkConfig = Add-AzVmssNetworkInterfaceConfiguration `
  -VirtualMachineScaleSet $VmssConfig `
  -Name "nicconfig1" `
  -Primary $true `
  -IPConfiguration $IpConfig

New-AzVmss `
  -ResourceGroupName $ResourceGroupName `
  -Name $VmssName `
  -VirtualMachineScaleSet $VmssConfig |
  Tee-Object (Join-Path $EvidencePath "vmss-create.txt")

# -----------------------------
# Inspect VMSS
# -----------------------------

Get-AzVmss `
  -ResourceGroupName $ResourceGroupName `
  -VMScaleSetName $VmssName |
  Tee-Object (Join-Path $EvidencePath "vmss-show.txt")

Get-AzVmssVM `
  -ResourceGroupName $ResourceGroupName `
  -VMScaleSetName $VmssName |
  Tee-Object (Join-Path $EvidencePath "vmss-instances.txt")

# -----------------------------
# Manual scale
# -----------------------------

Update-AzVmss `
  -ResourceGroupName $ResourceGroupName `
  -VMScaleSetName $VmssName `
  -SkuCapacity 3 |
  Tee-Object (Join-Path $EvidencePath "vmss-scale-out.txt")

Update-AzVmss `
  -ResourceGroupName $ResourceGroupName `
  -VMScaleSetName $VmssName `
  -SkuCapacity $InstanceCount |
  Tee-Object (Join-Path $EvidencePath "vmss-scale-in.txt")

# -----------------------------
# Instance operation
# -----------------------------

$Instance = Get-AzVmssVM `
  -ResourceGroupName $ResourceGroupName `
  -VMScaleSetName $VmssName |
  Select-Object -First 1

Restart-AzVmss `
  -ResourceGroupName $ResourceGroupName `
  -VMScaleSetName $VmssName `
  -InstanceId $Instance.InstanceId |
  Tee-Object (Join-Path $EvidencePath "vmss-instance-restart.txt")
```

# Configure_VMSS_Images_Scaling_And_Automation_Bicep_Skeleton

```bicep
// File: .\infra\04-vmss-images-scaling-automation\main.bicep
// Purpose: Deploy a baseline Linux VMSS with flexible orchestration, private subnet, NSG, manual upgrade policy, and bootstrap extension.

targetScope = 'resourceGroup'

@description('Azure region.')
param location string = resourceGroup().location

@description('VMSS name.')
param vmssName string = 'vmss-web-lab-01'

@description('VM size.')
param vmSize string = 'Standard_B2s'

@description('Initial instance count.')
param instanceCount int = 2

@description('Admin username.')
param adminUsername string = 'azureuser'

@description('SSH public key.')
param sshPublicKey string

@description('Marketplace image publisher.')
param imagePublisher string = 'Canonical'

@description('Marketplace image offer.')
param imageOffer string = '0001-com-ubuntu-server-jammy'

@description('Marketplace image SKU.')
param imageSku string = '22_04-lts-gen2'

@description('Marketplace image version.')
param imageVersion string = 'latest'

@description('VNet name.')
param vnetName string = 'vnet-compute-lab-01'

@description('VNet address prefix.')
param vnetPrefix string = '10.20.0.0/16'

@description('Subnet name.')
param subnetName string = 'snet-vmss-01'

@description('Subnet prefix.')
param subnetPrefix string = '10.20.2.0/24'

@description('NSG name.')
param nsgName string = 'nsg-snet-vmss-01'

@description('Autoscale setting name.')
param autoscaleName string = 'autoscale-vmss-web-lab-01'

@description('Autoscale minimum capacity.')
param autoscaleMin int = 2

@description('Autoscale default capacity.')
param autoscaleDefault int = 2

@description('Autoscale maximum capacity.')
param autoscaleMax int = 5

var tags = {
  Environment: 'Lab'
  Workload: 'Compute'
  ManagedBy: 'Bicep'
  Workbook: '04_Configure_VMSS_Images_Scaling_And_Automation'
}

resource nsg 'Microsoft.Network/networkSecurityGroups@2023-11-01' = {
  name: nsgName
  location: location
  tags: tags
  properties: {
    securityRules: [
      {
        name: 'Allow-HTTP-Lab'
        properties: {
          priority: 1000
          access: 'Allow'
          direction: 'Inbound'
          protocol: 'Tcp'
          sourceAddressPrefix: '<approved-source-prefix>'
          sourcePortRange: '*'
          destinationAddressPrefix: '*'
          destinationPortRange: '80'
        }
      }
    ]
  }
}

resource vnet 'Microsoft.Network/virtualNetworks@2023-11-01' = {
  name: vnetName
  location: location
  tags: tags
  properties: {
    addressSpace: {
      addressPrefixes: [
        vnetPrefix
      ]
    }
    subnets: [
      {
        name: subnetName
        properties: {
          addressPrefix: subnetPrefix
          networkSecurityGroup: {
            id: nsg.id
          }
        }
      }
    ]
  }
}

resource vmss 'Microsoft.Compute/virtualMachineScaleSets@2024-03-01' = {
  name: vmssName
  location: location
  tags: tags
  sku: {
    name: vmSize
    capacity: instanceCount
  }
  properties: {
    orchestrationMode: 'Flexible'
    upgradePolicy: {
      mode: 'Manual'
    }
    virtualMachineProfile: {
      osProfile: {
        computerNamePrefix: 'vmssweb'
        adminUsername: adminUsername
        linuxConfiguration: {
          disablePasswordAuthentication: true
          ssh: {
            publicKeys: [
              {
                path: '/home/${adminUsername}/.ssh/authorized_keys'
                keyData: sshPublicKey
              }
            ]
          }
        }
      }
      storageProfile: {
        imageReference: {
          publisher: imagePublisher
          offer: imageOffer
          sku: imageSku
          version: imageVersion
        }
        osDisk: {
          createOption: 'FromImage'
          managedDisk: {
            storageAccountType: 'Premium_LRS'
          }
          deleteOption: 'Delete'
        }
      }
      networkProfile: {
        networkInterfaceConfigurations: [
          {
            name: 'nicconfig1'
            properties: {
              primary: true
              deleteOption: 'Delete'
              ipConfigurations: [
                {
                  name: 'ipconfig1'
                  properties: {
                    subnet: {
                      id: resourceId('Microsoft.Network/virtualNetworks/subnets', vnet.name, subnetName)
                    }
                    primary: true
                  }
                }
              ]
            }
          }
        ]
      }
      diagnosticsProfile: {
        bootDiagnostics: {
          enabled: true
        }
      }
      extensionProfile: {
        extensions: [
          {
            name: 'CustomScript'
            properties: {
              publisher: 'Microsoft.Azure.Extensions'
              type: 'CustomScript'
              typeHandlerVersion: '2.1'
              autoUpgradeMinorVersion: true
              settings: {
                commandToExecute: 'sudo apt-get update && sudo apt-get install -y nginx && echo vmss-lab | sudo tee /var/www/html/index.html'
              }
            }
          }
        ]
      }
    }
  }
}

resource autoscale 'Microsoft.Insights/autoscalesettings@2022-10-01' = {
  name: autoscaleName
  location: location
  tags: tags
  properties: {
    name: autoscaleName
    enabled: true
    targetResourceUri: vmss.id
    profiles: [
      {
        name: 'cpu-autoscale-profile'
        capacity: {
          minimum: string(autoscaleMin)
          maximum: string(autoscaleMax)
          default: string(autoscaleDefault)
        }
        rules: [
          {
            metricTrigger: {
              metricName: 'Percentage CPU'
              metricResourceUri: vmss.id
              timeGrain: 'PT1M'
              statistic: 'Average'
              timeWindow: 'PT5M'
              timeAggregation: 'Average'
              operator: 'GreaterThan'
              threshold: 70
            }
            scaleAction: {
              direction: 'Increase'
              type: 'ChangeCount'
              value: '1'
              cooldown: 'PT5M'
            }
          }
          {
            metricTrigger: {
              metricName: 'Percentage CPU'
              metricResourceUri: vmss.id
              timeGrain: 'PT1M'
              statistic: 'Average'
              timeWindow: 'PT10M'
              timeAggregation: 'Average'
              operator: 'LessThan'
              threshold: 30
            }
            scaleAction: {
              direction: 'Decrease'
              type: 'ChangeCount'
              value: '1'
              cooldown: 'PT10M'
            }
          }
        ]
      }
    ]
  }
}

output vmssId string = vmss.id
output vmssName string = vmss.name
output autoscaleId string = autoscale.id
output subnetId string = resourceId('Microsoft.Network/virtualNetworks/subnets', vnet.name, subnetName)
```

# Configure_VMSS_Images_Scaling_And_Automation_Bicep_Parameters_Skeleton

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "location": {
      "value": "eastus"
    },
    "vmssName": {
      "value": "vmss-web-lab-01"
    },
    "vmSize": {
      "value": "Standard_B2s"
    },
    "instanceCount": {
      "value": 2
    },
    "adminUsername": {
      "value": "azureuser"
    },
    "sshPublicKey": {
      "value": "<paste-ssh-public-key-here>"
    },
    "vnetName": {
      "value": "vnet-compute-lab-01"
    },
    "vnetPrefix": {
      "value": "10.20.0.0/16"
    },
    "subnetName": {
      "value": "snet-vmss-01"
    },
    "subnetPrefix": {
      "value": "10.20.2.0/24"
    },
    "nsgName": {
      "value": "nsg-snet-vmss-01"
    },
    "autoscaleName": {
      "value": "autoscale-vmss-web-lab-01"
    },
    "autoscaleMin": {
      "value": 2
    },
    "autoscaleDefault": {
      "value": 2
    },
    "autoscaleMax": {
      "value": 5
    }
  }
}
```

# Configure_VMSS_Images_Scaling_And_Automation_Image_Source_Skeleton

```text
Purpose:
Choose and manage the image source used by VMSS instances.

Marketplace image:
Use for:
- Fast lab deployment
- Generic Windows Server or Linux baseline
- Minimal custom image management

Example:
az vmss create \
  --image Ubuntu2204 \
  --orchestration-mode Flexible \
  ...

Managed image:
Use for:
- Simple captured image workflow
- Single-region lab
- Small image lifecycle

Limitations:
- Less flexible than Azure Compute Gallery
- Not ideal for versioning and replication

Azure Compute Gallery:
Use for:
- Golden images
- Versioned image rollout
- Multi-region replication
- Controlled image promotion

Recommended image lifecycle:
1. Build source VM.
2. Install baseline packages and agents.
3. Clean machine-specific data.
4. Generalize if required by OS workflow.
5. Capture managed image or gallery image version.
6. Deploy VMSS from image version.
7. Validate application and extensions.
8. Publish next image version when updates are ready.
9. Update VMSS model to new image.
10. Roll instances forward in controlled batches.
11. Keep rollback image version available.
12. Retire old image versions after validation window.

Image evidence to capture:
- Publisher
- Offer
- SKU
- Version
- Gallery name
- Image definition
- Image version
- Source VM or managed image ID
- Replication regions
- VMSS model image reference
```

# Configure_VMSS_Images_Scaling_And_Automation_Manual_And_Autoscale_Skeleton

```text
Purpose:
Control VMSS capacity manually and with Azure Monitor autoscale.

Manual scale:
1. Use for first validation.
2. Scale to 1, 2, or 3 instances.
3. Confirm instance creation.
4. Confirm application bootstrap works.
5. Confirm load-balanced path works if applicable.

Manual scale commands:
az vmss scale -g <rg> -n <vmss-name> --new-capacity 3
az vmss list-instances -g <rg> -n <vmss-name> -o table

Autoscale design:
1. Minimum capacity protects baseline availability.
2. Maximum capacity protects cost and quota.
3. Default capacity is steady state.
4. Scale-out rule should react quickly enough for load.
5. Scale-in rule should be slower to avoid flapping.
6. Cooldown should prevent rapid oscillation.
7. Metric must exist and represent real pressure.
8. Test with safe load before production use.

CPU autoscale example:
- Minimum: 2
- Default: 2
- Maximum: 5
- Scale out: CPU average greater than 70 percent for 5 minutes, add 1
- Scale in: CPU average less than 30 percent for 10 minutes, remove 1

Scale rule evidence:
1. Autoscale profile JSON
2. Rule names and thresholds
3. Capacity bounds
4. VMSS metrics
5. Activity log scale events
6. Instance count before and after scale
```

# Configure_VMSS_Images_Scaling_And_Automation_Instance_Operations_Skeleton

```text
Purpose:
Operate individual VMSS instances without treating them as permanent standalone servers.

Common instance commands:
1. List instances:
   az vmss list-instances -g <rg> -n <vmss-name> -o table

2. Get instance IDs:
   az vmss list-instances -g <rg> -n <vmss-name> --query "[].instanceId" -o tsv

3. Run command on Linux instance:
   az vmss run-command invoke \
     -g <rg> \
     -n <vmss-name> \
     --instance-id <instance-id> \
     --command-id RunShellScript \
     --scripts "hostname; uptime"

4. Run command on Windows instance:
   az vmss run-command invoke \
     -g <rg> \
     -n <vmss-name> \
     --instance-id <instance-id> \
     --command-id RunPowerShellScript \
     --scripts "hostname; whoami"

5. Restart instance:
   az vmss restart -g <rg> -n <vmss-name> --instance-ids <instance-id>

6. Reimage instance:
   az vmss reimage -g <rg> -n <vmss-name> --instance-id <instance-id>

7. Delete instance:
   az vmss delete-instances -g <rg> -n <vmss-name> --instance-ids <instance-id>

Rules:
- Use reimage when instance drift is suspected.
- Use image update for persistent baseline changes.
- Use extensions or VM applications for repeatable bootstrap.
- Avoid manual one-off changes unless troubleshooting.
- Never rely on a hand-edited VMSS instance as the desired state.
```

# Configure_VMSS_Images_Scaling_And_Automation_Upgrade_And_Repair_Skeleton

```text
Purpose:
Control how VMSS model changes reach instances and how unhealthy instances recover.

Upgrade policy options:
1. Manual:
   - Best for labs and controlled validation
   - Operator updates instances
   - Lower surprise factor

2. Automatic:
   - Platform applies model changes automatically
   - Useful when risk is low and app is stateless

3. Rolling:
   - Updates instances in batches
   - Better for production-like workloads
   - Needs health checks and rollback planning

Manual model update flow:
1. Change VMSS model:
   - image
   - extension
   - size
   - tags
   - OS profile setting
   - data disk setting

2. Validate model:
   az vmss show -g <rg> -n <vmss-name>

3. Apply to instances:
   az vmss update-instances -g <rg> -n <vmss-name> --instance-ids "*"

4. Validate instances:
   az vmss list-instances -g <rg> -n <vmss-name> --expand instanceView -o table

Image rollout flow:
1. Publish new image version.
2. Update VMSS model image reference.
3. Update one test instance if supported by mode and policy.
4. Validate boot, app, logs, and metrics.
5. Roll forward remaining instances.
6. Monitor errors and scale events.
7. Keep old image version until validation window closes.

Automatic repair flow:
1. Configure application health signal.
2. Enable automatic repairs.
3. Set grace period.
4. Confirm unhealthy instance detection.
5. Confirm repair action behavior.
6. Monitor repair events in Activity Log.

Rollback:
1. Revert VMSS image reference to previous image version.
2. Reapply model to instances.
3. Reimage broken instances if required.
4. Disable autoscale during unstable rollout if needed.
5. Scale down failed new capacity if required.
```

# Configure_VMSS_Images_Scaling_And_Automation_Verification_Commands

```bash
# Account and resource group
az account show -o table

az group show \
  --name "<resource-group-name>" \
  -o table

# VMSS model
az vmss show \
  --resource-group "<resource-group-name>" \
  --name "<vmss-name>" \
  -o json

# VMSS summary
az vmss show \
  --resource-group "<resource-group-name>" \
  --name "<vmss-name>" \
  --query "{name:name,location:location,orchestrationMode:orchestrationMode,sku:sku,upgradePolicy:upgradePolicy.mode}" \
  -o table

# Image reference
az vmss show \
  --resource-group "<resource-group-name>" \
  --name "<vmss-name>" \
  --query "virtualMachineProfile.storageProfile.imageReference" \
  -o json

# Instance list
az vmss list-instances \
  --resource-group "<resource-group-name>" \
  --name "<vmss-name>" \
  -o table

# Instance view
az vmss get-instance-view \
  --resource-group "<resource-group-name>" \
  --name "<vmss-name>" \
  -o json

# Instance status with expanded view
az vmss list-instances \
  --resource-group "<resource-group-name>" \
  --name "<vmss-name>" \
  --expand instanceView \
  -o table

# Capacity
az vmss show \
  --resource-group "<resource-group-name>" \
  --name "<vmss-name>" \
  --query "sku.capacity" \
  -o tsv

# Autoscale settings
az monitor autoscale list \
  --resource-group "<resource-group-name>" \
  -o table

az monitor autoscale show \
  --resource-group "<resource-group-name>" \
  --name "<autoscale-name>" \
  -o json

# Metrics
VMSS_ID=$(az vmss show \
  --resource-group "<resource-group-name>" \
  --name "<vmss-name>" \
  --query id \
  -o tsv)

az monitor metrics list \
  --resource "$VMSS_ID" \
  --metric "Percentage CPU" \
  --interval PT1M \
  -o table

# Extensions
az vmss extension list \
  --resource-group "<resource-group-name>" \
  --vmss-name "<vmss-name>" \
  -o table

az vmss extension show \
  --resource-group "<resource-group-name>" \
  --vmss-name "<vmss-name>" \
  --name "<extension-name>" \
  -o json

# Run command test
az vmss run-command invoke \
  --resource-group "<resource-group-name>" \
  --name "<vmss-name>" \
  --instance-id "<instance-id>" \
  --command-id RunShellScript \
  --scripts "hostname; uptime"

# Upgrade policy
az vmss show \
  --resource-group "<resource-group-name>" \
  --name "<vmss-name>" \
  --query "upgradePolicy" \
  -o json

# Placement
az vmss show \
  --resource-group "<resource-group-name>" \
  --name "<vmss-name>" \
  --query "{zones:zones,platformFaultDomainCount:platformFaultDomainCount,orchestrationMode:orchestrationMode}" \
  -o json

# Activity log
az monitor activity-log list \
  --resource-group "<resource-group-name>" \
  --max-events 30 \
  -o table

# Azure Compute Gallery
az sig list \
  --resource-group "<resource-group-name>" \
  -o table

az sig image-definition list \
  --resource-group "<resource-group-name>" \
  --gallery-name "<gallery-name>" \
  -o table

az sig image-version list \
  --resource-group "<resource-group-name>" \
  --gallery-name "<gallery-name>" \
  --gallery-image-definition "<image-definition>" \
  -o table
```

```powershell
# Context
Get-AzContext

# VMSS model
Get-AzVmss `
  -ResourceGroupName "<resource-group-name>" `
  -VMScaleSetName "<vmss-name>"

# VMSS instances
Get-AzVmssVM `
  -ResourceGroupName "<resource-group-name>" `
  -VMScaleSetName "<vmss-name>"

# Instance view
Get-AzVmssVM `
  -ResourceGroupName "<resource-group-name>" `
  -VMScaleSetName "<vmss-name>" `
  -InstanceView

# Restart an instance
Restart-AzVmss `
  -ResourceGroupName "<resource-group-name>" `
  -VMScaleSetName "<vmss-name>" `
  -InstanceId "<instance-id>"

# Autoscale settings
Get-AzAutoscaleSetting `
  -ResourceGroupName "<resource-group-name>"

# Azure Compute Gallery
Get-AzGallery `
  -ResourceGroupName "<resource-group-name>"

Get-AzGalleryImageDefinition `
  -ResourceGroupName "<resource-group-name>" `
  -GalleryName "<gallery-name>"

Get-AzGalleryImageVersion `
  -ResourceGroupName "<resource-group-name>" `
  -GalleryName "<gallery-name>" `
  -GalleryImageDefinitionName "<image-definition>"
```

# Configure_VMSS_Images_Scaling_And_Automation_Rollback

| Change | Rollback Command / Action | Risk |
|---|---|---|
| VMSS created for lab | `az vmss delete --resource-group "<resource-group-name>" --name "<vmss-name>"` | Deletes scale set instances |
| Full lab resource group created | `az group delete --name "<resource-group-name>" --yes --no-wait` | Deletes all resources in the resource group |
| Manual scale out increased cost | `az vmss scale --resource-group "<resource-group-name>" --name "<vmss-name>" --new-capacity "<previous-count>"` | Removes instances if scaling down |
| Autoscale profile created incorrectly | `az monitor autoscale delete --resource-group "<resource-group-name>" --name "<autoscale-name>"` | Capacity will no longer autoscale |
| Scale-out rule too aggressive | Edit autoscale rule or delete autoscale setting | Cost spike may continue until fixed |
| Scale-in rule too aggressive | Raise minimum count or remove scale-in rule | App capacity may drop too low |
| Extension installed | `az vmss extension delete --resource-group "<resource-group-name>" --vmss-name "<vmss-name>" --name "<extension-name>"` | Guest changes made by script may remain |
| Bad extension applied to instances | Remove extension, fix script, reimage broken instances | Reimage destroys instance-local changes |
| Bad image version deployed | Revert VMSS model to previous image version and update instances | Existing instances may need reimage |
| Instance unhealthy after update | `az vmss reimage --resource-group "<resource-group-name>" --name "<vmss-name>" --instance-id "<instance-id>"` | Instance-local data is lost |
| Instance stuck after restart | Reimage instance or delete instance and let scale set replace it | App capacity temporarily drops |
| Upgrade policy changed | `az vmss update --resource-group "<resource-group-name>" --name "<vmss-name>" --set upgradePolicy.mode=Manual` | May pause expected rollout |
| Gallery image version published incorrectly | Mark version excluded from latest or create corrected version | Existing deployments may already reference it |
| NSG HTTP rule opened too broadly | Delete or restrict NSG rule | Exposure continues until rule is fixed |
| VMSS subnet created incorrectly | Recreate VMSS in correct subnet | Subnet changes may require redeploy |
| Wrong orchestration mode selected | Recreate VMSS with correct orchestration mode | Orchestration mode is not a casual in-place switch |
| Wrong VM size selected | Update VMSS size and apply to instances | May require instance update or replacement |

# Configure_VMSS_Images_Scaling_And_Automation_Failure_Checks

| Symptom | Likely Layer | First Check | Fix |
|---|---|---|---|
| VMSS deployment fails with `AuthorizationFailed` | RBAC | `az account show`; role assignments | Use correct subscription or assign required role |
| VMSS deployment fails with provider error | Resource provider | `az provider show --namespace Microsoft.Compute` | Register provider |
| VMSS deployment fails with `SkuNotAvailable` | Region or zone | VM size availability | Pick another size, region, or zone |
| VMSS deployment fails with quota error | Subscription quota | Quota blade or error details | Reduce capacity or request quota |
| VMSS deployment fails with image error | Image reference | Image publisher, offer, SKU, version, or gallery ID | Correct image reference |
| VMSS creates but instances fail provisioning | Image, extension, network, or quota | `az vmss list-instances --expand instanceView` | Check instance errors and activity log |
| Instances exist but app is not running | Bootstrap failure | Extension state and run command | Fix Custom Script or image |
| Extension fails | Script, VM agent, or network | `az vmss extension show` | Fix script, outbound path, or extension settings |
| Extension succeeds but app missing | Script did not do expected install | Run command to inspect guest | Fix script and reapply model |
| Autoscale does not trigger | Metric rule or no metric data | Autoscale setting and metrics | Correct metric, threshold, time window, or capacity bounds |
| Autoscale scales too often | Cooldown too short or threshold too sensitive | Autoscale history and rules | Increase cooldown or adjust thresholds |
| Autoscale never scales in | Minimum capacity or rule condition | Autoscale profile | Lower minimum or adjust scale-in threshold |
| Manual scale fails | Quota or model issue | Activity log and VMSS operation | Fix quota, model, or capacity |
| Scale-in removed important instance | No instance protection | Instance list and app state | Use instance protection or externalize state |
| New image version fails | Bad image build | Image version and run command | Revert to previous image version |
| Gallery image not visible | Replication or permission issue | Gallery image version status | Wait for replication or fix RBAC |
| Run command fails on instance | VM agent issue | Instance view | Restart, reimage, or repair guest agent |
| Instance stuck updating | Upgrade or extension issue | Instance view and extension status | Reimage instance or fix model |
| Rolling upgrade stalls | Health probe or batch settings | Upgrade status and health signal | Fix health probe, app startup, or upgrade policy |
| Public app endpoint fails | Load balancer, NSG, app, or health probe | NSG rules, backend pool, app service | Fix probe, app listener, or network rule |
| SSH or RDP fails | Network path | Effective NSG and instance IP | Use Bastion or scoped inbound rule |
| Cost spikes | Autoscale max too high or scale-in broken | Autoscale profile and capacity | Lower max, fix scale-in, or manual scale down |
| Instances drift from baseline | Manual changes inside guest | Compare image and extension state | Reimage instances and update image pipeline |
| VMSS cannot delete | Locks or dependent resources | Resource locks and dependencies | Remove lock or dependency first |
| Bicep deployment fails for autoscale | Bad target resource URI or metric rule | Deployment operation details | Fix `targetResourceUri`, metric name, or profile syntax |

# Configure_VMSS_Images_Scaling_And_Automation_Related_Labs

| Lab                                                                 | Related Workbook                                                                         | Skill Proven                                                    |
| ------------------------------------------------------------------- | ---------------------------------------------------------------------------------------- | --------------------------------------------------------------- |
| Deploy ARM and Bicep baseline                                       | `01_Deploy_ARM_Templates_Bicep_And_Exported_Deployments.md`                              | Repeatable deployment foundation                                |
| Create and manage Azure VMs                                         | `02_Create_Configure_And_Manage_Azure_Virtual_Machines.md`                               | Individual VM lifecycle                                         |
| Configure VM disks, sizes, availability, extensions, and encryption | `03_Configure_VM_Disks_Sizes_Availability_Sets_Zones_Extensions_And_Encryption.md`       | VM platform configuration                                       |
| Configure VMSS images and scaling                                   | `04_Configure_VMSS_Images_Scaling_And_Automation.md`                                     | Scale set image, capacity, autoscale, and automation management |
| Deploy ACR, ACI, and Container Apps                                 | `05_Deploy_ACR_ACI_Azure_Container_Apps_Sizing_And_Scaling.md`                           | Container compute hosting                                       |
| Deploy App Service plans and web apps                               | `06_Deploy_App_Service_Plans_Web_Apps_TLS_Custom_Domains_Backup_Networking_And_Slots.md` | PaaS app hosting                                                |
| Configure compute managed identities and Key Vault references       | `07_Configure_Compute_Managed_Identities_Key_Vault_Access_And_Secrets_References.md`     | Secretless compute access                                       |
| Troubleshoot compute deployment and runtime issues                  | `08_Troubleshoot_Azure_VM_Container_App_Service_Deployment_And_Runtime_Issues.md`        | VMSS, VM, container, and App Service troubleshooting            |