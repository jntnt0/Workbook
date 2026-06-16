# 02_Create_Configure_And_Manage_Azure_Virtual_Machines

# 02_Create_Configure_And_Manage_Azure_Virtual_Machines_Index

02_Create_Configure_And_Manage_Azure_Virtual_Machines.md  
Create_Configure_And_Manage_Azure_Virtual_Machines  
Create_Configure_And_Manage_Azure_Virtual_Machines_Source_Basis  
Create_Configure_And_Manage_Azure_Virtual_Machines_Mental_Model  
Create_Configure_And_Manage_Azure_Virtual_Machines_Planning_Table  
Create_Configure_And_Manage_Azure_Virtual_Machines_Configuration_Checklist  
Create_Configure_And_Manage_Azure_Virtual_Machines_Portal_Skeleton  
Create_Configure_And_Manage_Azure_Virtual_Machines_AzureCLI_Skeleton  
Create_Configure_And_Manage_Azure_Virtual_Machines_PowerShell_Skeleton  
Create_Configure_And_Manage_Azure_Virtual_Machines_Bicep_Skeleton  
Create_Configure_And_Manage_Azure_Virtual_Machines_Linux_VM_Skeleton  
Create_Configure_And_Manage_Azure_Virtual_Machines_Windows_VM_Skeleton  
Create_Configure_And_Manage_Azure_Virtual_Machines_Lifecycle_Operations_Skeleton  
Create_Configure_And_Manage_Azure_Virtual_Machines_Verification_Commands  
Create_Configure_And_Manage_Azure_Virtual_Machines_Rollback  
Create_Configure_And_Manage_Azure_Virtual_Machines_Failure_Checks  
Create_Configure_And_Manage_Azure_Virtual_Machines_Related_Labs  

# Create_Configure_And_Manage_Azure_Virtual_Machines_Source_Basis

| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Azure Virtual Machines overview | VM purpose, IaaS responsibility model, VM lifecycle, OS images, and sizing |
| Microsoft Learn | Linux VM quickstart with Azure CLI | Resource group, VM creation, SSH key authentication, public IP, and connection validation |
| Microsoft Learn | Windows VM quickstart with Azure CLI | Windows VM creation, admin credentials, RDP access, and VM validation |
| Microsoft Learn | Azure VM sizes | Size selection, CPU, memory, disk, NIC limits, and resize considerations |
| Microsoft Learn | Azure VM images | Marketplace images, publisher, offer, SKU, version, and image selection |
| Microsoft Learn | Azure VM networking | NIC, subnet, public IP, NSG, inbound rules, private IP, and connectivity |
| Microsoft Learn | Azure Bastion | Private administrative access to VMs without exposing RDP or SSH directly |
| Microsoft Learn | Azure VM managed identities | System-assigned and user-assigned identity for secretless Azure resource access |
| Microsoft Learn | Azure VM Run Command | Control-plane command execution inside Linux or Windows VM guests |
| Microsoft Learn | Azure VM boot diagnostics | Boot screenshot, serial log, and guest startup evidence |
| Microsoft Learn | Azure VM extensions | Post-deployment configuration, agent-based operations, and extension health |
| Microsoft Learn | Azure Resource Manager and Bicep | Declarative VM, NIC, VNet, NSG, public IP, and identity deployment |
| Azure operational practice | Secure VM baseline | Prefer private admin path, least privilege RBAC, limited inbound rules, tags, diagnostics, and evidence capture |
| Azure operational practice | VM lifecycle operations | Create, start, stop, deallocate, restart, resize, redeploy, reapply, run command, tag, inspect, and delete |

# Create_Configure_And_Manage_Azure_Virtual_Machines_Mental_Model

| Concept | Operational Meaning |
|---|---|
| Virtual machine | Azure IaaS compute resource that runs a full guest operating system |
| Resource group | Management container for VM, NIC, disk, public IP, NSG, and related resources |
| Region | Azure location where VM resources are deployed |
| VM size | Hardware profile defining vCPU, memory, disk limits, NIC limits, and cost |
| Image | Source OS used to create the VM |
| Marketplace image | Microsoft or publisher-provided image, such as Ubuntu or Windows Server |
| OS disk | Managed disk that contains the guest operating system |
| NIC | Network interface card attached to the VM |
| VNet | Private Azure network where the VM NIC is placed |
| Subnet | IP range inside a VNet used for VM placement |
| NSG | Network security group that filters inbound and outbound traffic |
| Public IP | Internet-reachable address attached to NIC or load balancer |
| Private IP | VNet-routable address assigned to the NIC |
| SSH | Linux remote administration protocol |
| RDP | Windows remote administration protocol |
| Bastion | Azure-managed private admin access path using browser-based SSH or RDP |
| Boot diagnostics | Platform capture of boot logs and screenshots for troubleshooting |
| VM agent | Guest agent that supports extensions, Run Command, and platform operations |
| Run Command | Azure control-plane method to run scripts inside a VM guest |
| Managed identity | Microsoft Entra identity attached to the VM for Azure resource access |
| System-assigned identity | Identity lifecycle tied to one VM |
| User-assigned identity | Standalone identity that can be attached to one or more resources |
| Tags | Metadata used for ownership, cost, environment, and lifecycle tracking |
| Stop | Guest shutdown action |
| Deallocate | Releases compute allocation and stops compute billing for the VM |
| Restart | Reboots the VM guest |
| Redeploy | Moves VM to a new Azure host, used for host-level issues |
| Reapply | Reapplies VM model state to the running VM |
| First rule | Build VMs from repeatable config, not undocumented clicks |
| Blunt rule | A VM with open 22 or 3389 to the internet is a lab shortcut, not a safe baseline |

# Create_Configure_And_Manage_Azure_Virtual_Machines_Planning_Table

| Item | Example | Decision |
|---|---|---|
| Tenant | `contoso.onmicrosoft.com` | `<tenant-name>` |
| Subscription ID | `00000000-0000-0000-0000-000000000000` | `<subscription-id>` |
| Resource group | `rg-compute-lab-01` | `<resource-group-name>` |
| Location | `eastus` | `<azure-region>` |
| VM name | `vm-lab-linux-01` | `<vm-name>` |
| OS type | `Linux` | `<Linux / Windows>` |
| Image | `Ubuntu2204` | `<image>` |
| VM size | `Standard_B2s` | `<vm-size>` |
| Admin username | `azureuser` | `<admin-username>` |
| Authentication | `SSH key` | `<SSH key / password / Entra login>` |
| SSH public key path | `~/.ssh/id_rsa.pub` | `<ssh-public-key-path>` |
| Windows admin password | Stored securely | `<secure-password-source>` |
| VNet name | `vnet-compute-lab-01` | `<vnet-name>` |
| VNet prefix | `10.20.0.0/16` | `<vnet-prefix>` |
| Subnet name | `snet-vm-01` | `<subnet-name>` |
| Subnet prefix | `10.20.1.0/24` | `<subnet-prefix>` |
| NSG name | `nsg-snet-vm-01` | `<nsg-name>` |
| NIC name | `nic-vm-lab-linux-01` | `<nic-name>` |
| Public IP required | `No for secure baseline` | `<yes-no>` |
| Public IP name | `pip-vm-lab-linux-01` | `<public-ip-name>` |
| Bastion required | `Yes` | `<yes-no>` |
| Bastion subnet | `AzureBastionSubnet` | `<bastion-subnet-name>` |
| Bastion public IP | `pip-bas-compute-lab-01` | `<bastion-public-ip-name>` |
| Bastion host | `bas-compute-lab-01` | `<bastion-name>` |
| Inbound admin source | `203.0.113.10/32` | `<approved-source-cidr>` |
| Boot diagnostics | `Enabled` | `<enabled-disabled>` |
| Managed identity | `System-assigned` | `<system-assigned / user-assigned / none>` |
| Tags | `Environment=Lab Workload=Compute` | `<tag-set>` |
| Evidence path | `.\evidence\02-azure-vms` | `<evidence-path>` |
| Rollback posture | `Delete lab RG or delete VM resource set` | `<rollback-plan>` |

# Create_Configure_And_Manage_Azure_Virtual_Machines_Configuration_Checklist

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm Azure CLI context | Admin Workstation | `az account show -o table` | Correct tenant and subscription are visible |
| 2 | Set subscription | Admin Workstation | `az account set --subscription "<subscription-id>"` | CLI targets correct subscription |
| 3 | Register compute provider | Admin Workstation | `az provider register --namespace Microsoft.Compute` | Compute provider registration starts |
| 4 | Register network provider | Admin Workstation | `az provider register --namespace Microsoft.Network` | Network provider registration starts |
| 5 | Register storage provider | Admin Workstation | `az provider register --namespace Microsoft.Storage` | Storage provider registration starts |
| 6 | Confirm provider state | Admin Workstation | `az provider show --namespace Microsoft.Compute --query registrationState -o tsv` | Provider shows `Registered` |
| 7 | Create resource group | Admin Workstation | `az group create --name "<resource-group-name>" --location "<azure-region>"` | Resource group exists |
| 8 | Create VNet and subnet | Admin Workstation | `az network vnet create --resource-group "<resource-group-name>" --location "<azure-region>" --name "<vnet-name>" --address-prefixes "<vnet-prefix>" --subnet-name "<subnet-name>" --subnet-prefixes "<subnet-prefix>"` | VNet and subnet exist |
| 9 | Create NSG | Admin Workstation | `az network nsg create --resource-group "<resource-group-name>" --location "<azure-region>" --name "<nsg-name>"` | NSG exists |
| 10 | Associate NSG to subnet | Admin Workstation | `az network vnet subnet update --resource-group "<resource-group-name>" --vnet-name "<vnet-name>" --name "<subnet-name>" --network-security-group "<nsg-name>"` | Subnet is protected by NSG |
| 11 | Add scoped SSH rule only if public SSH is required | Admin Workstation | `az network nsg rule create --resource-group "<resource-group-name>" --nsg-name "<nsg-name>" --name Allow-SSH-Scoped --priority 1000 --access Allow --direction Inbound --protocol Tcp --source-address-prefixes "<approved-source-cidr>" --source-port-ranges "*" --destination-address-prefixes "*" --destination-port-ranges 22` | SSH allowed only from approved source |
| 12 | Add scoped RDP rule only if public RDP is required | Admin Workstation | `az network nsg rule create --resource-group "<resource-group-name>" --nsg-name "<nsg-name>" --name Allow-RDP-Scoped --priority 1010 --access Allow --direction Inbound --protocol Tcp --source-address-prefixes "<approved-source-cidr>" --source-port-ranges "*" --destination-address-prefixes "*" --destination-port-ranges 3389` | RDP allowed only from approved source |
| 13 | Create Linux VM baseline | Admin Workstation | `az vm create --resource-group "<resource-group-name>" --name "<linux-vm-name>" --image Ubuntu2204 --size "<vm-size>" --admin-username "<admin-username>" --generate-ssh-keys --vnet-name "<vnet-name>" --subnet "<subnet-name>" --nsg "" --public-ip-address "" --tags Environment=Lab Workload=Compute ManagedBy=AzureCLI` | Linux VM is created with private-only NIC |
| 14 | Create Windows VM baseline | Admin Workstation | `az vm create --resource-group "<resource-group-name>" --name "<windows-vm-name>" --image Win2022Datacenter --size "<vm-size>" --admin-username "<admin-username>" --admin-password "<secure-password>" --vnet-name "<vnet-name>" --subnet "<subnet-name>" --nsg "" --public-ip-address "" --tags Environment=Lab Workload=Compute ManagedBy=AzureCLI` | Windows VM is created with private-only NIC |
| 15 | Confirm VM state | Admin Workstation | `az vm list --resource-group "<resource-group-name>" -d -o table` | VM appears with power state and private IP |
| 16 | Enable system-assigned identity | Admin Workstation | `az vm identity assign --resource-group "<resource-group-name>" --name "<vm-name>"` | VM has managed identity |
| 17 | Confirm VM identity | Admin Workstation | `az vm identity show --resource-group "<resource-group-name>" --name "<vm-name>" -o json` | Principal ID is visible |
| 18 | Enable boot diagnostics | Admin Workstation | `az vm boot-diagnostics enable --resource-group "<resource-group-name>" --name "<vm-name>"` | Boot diagnostics enabled |
| 19 | Confirm boot diagnostics | Admin Workstation | `az vm boot-diagnostics get-boot-log --resource-group "<resource-group-name>" --name "<vm-name>"` | Boot log is returned if guest supports it |
| 20 | Run Linux command | Admin Workstation | `az vm run-command invoke --resource-group "<resource-group-name>" --name "<linux-vm-name>" --command-id RunShellScript --scripts "hostname; uname -a; uptime"` | Linux guest returns command output |
| 21 | Run Windows command | Admin Workstation | `az vm run-command invoke --resource-group "<resource-group-name>" --name "<windows-vm-name>" --command-id RunPowerShellScript --scripts "hostname; Get-ComputerInfo | Select-Object CsName,WindowsProductName,WindowsVersion"` | Windows guest returns command output |
| 22 | Inspect VM instance view | Admin Workstation | `az vm get-instance-view --resource-group "<resource-group-name>" --name "<vm-name>" -o json` | Runtime and provisioning state are visible |
| 23 | Stop VM | Admin Workstation | `az vm stop --resource-group "<resource-group-name>" --name "<vm-name>"` | Guest shuts down |
| 24 | Start VM | Admin Workstation | `az vm start --resource-group "<resource-group-name>" --name "<vm-name>"` | VM starts |
| 25 | Restart VM | Admin Workstation | `az vm restart --resource-group "<resource-group-name>" --name "<vm-name>"` | VM reboots |
| 26 | Deallocate VM | Admin Workstation | `az vm deallocate --resource-group "<resource-group-name>" --name "<vm-name>"` | Compute allocation is released |
| 27 | Start deallocated VM | Admin Workstation | `az vm start --resource-group "<resource-group-name>" --name "<vm-name>"` | VM is allocated and started again |
| 28 | Add or update tags | Admin Workstation | `az resource tag --ids "$(az vm show --resource-group "<resource-group-name>" --name "<vm-name>" --query id -o tsv)" --tags Environment=Lab Workload=Compute Owner="<owner>"` | VM tags are updated |
| 29 | List VM sizes in region | Admin Workstation | `az vm list-sizes --location "<azure-region>" -o table` | Available regional VM sizes are listed |
| 30 | List resize options for VM | Admin Workstation | `az vm list-vm-resize-options --resource-group "<resource-group-name>" --name "<vm-name>" -o table` | Compatible resize options are listed |
| 31 | Capture VM evidence | Admin Workstation | `az vm show --resource-group "<resource-group-name>" --name "<vm-name>" --show-details -o json > .\evidence\02-azure-vms\vm-final.json` | VM config evidence is saved |
| 32 | Capture NIC evidence | Admin Workstation | `az network nic list --resource-group "<resource-group-name>" -o json > .\evidence\02-azure-vms\nics-final.json` | NIC evidence is saved |
| 33 | Capture NSG evidence | Admin Workstation | `az network nsg show --resource-group "<resource-group-name>" --name "<nsg-name>" -o json > .\evidence\02-azure-vms\nsg-final.json` | NSG evidence is saved |
| 34 | Capture resource inventory | Admin Workstation | `az resource list --resource-group "<resource-group-name>" -o table > .\evidence\02-azure-vms\resource-list.txt` | Resource inventory is saved |
| 35 | Document final access path | Operator | `Record Bastion, private IP, or scoped public IP path` | Admin access method is documented |

# Create_Configure_And_Manage_Azure_Virtual_Machines_Portal_Skeleton

```text
Purpose:
Create, configure, and manage Azure virtual machines through the Azure portal.

Portal path:
Azure portal
> Virtual machines
> Create
> Azure virtual machine

Basics:
1. Subscription: <subscription-name>
2. Resource group: <resource-group-name>
3. Virtual machine name: <vm-name>
4. Region: <azure-region>
5. Availability options:
   - None for basic lab
   - Availability zone if single-VM zonal placement is required
   - Availability set for grouped regional VMs
6. Security type:
   - Standard for broad compatibility
   - Trusted launch where supported and desired
7. Image:
   - Ubuntu Server 22.04 LTS
   - Windows Server 2022 Datacenter
   - Approved custom image if used
8. Size: <vm-size>
9. Authentication:
   - Linux: SSH public key preferred
   - Windows: password or approved enterprise method

Disks:
1. OS disk type:
   - Standard SSD for lab cost control
   - Premium SSD for production-like performance
2. Delete with VM:
   - Enabled for disposable lab VMs
   - Disabled when OS disk retention is required
3. Data disks:
   - Keep deeper disk work in task 03

Networking:
1. Virtual network: <vnet-name>
2. Subnet: <subnet-name>
3. Public IP:
   - None for secure private-only baseline
   - Public only for scoped lab access
4. NIC network security group:
   - Prefer subnet NSG for consistent rules
5. Public inbound ports:
   - None for secure baseline
   - SSH or RDP only from approved source for lab
6. Bastion:
   - Preferred for browser-based private admin access

Management:
1. Boot diagnostics: Enable
2. System-assigned managed identity: Enable if VM needs Azure resource access
3. Auto-shutdown: Optional for lab cost control
4. Patch orchestration:
   - Configure according to OS and workload requirements
5. Monitoring:
   - Enable later through Azure Monitor workbook if required

Tags:
1. Environment: Lab
2. Workload: Compute
3. Owner: <owner>
4. ManagedBy: Portal
5. Workbook: 02_Create_Configure_And_Manage_Azure_Virtual_Machines

Post-create validation:
1. Open VM Overview.
2. Confirm Status is Running.
3. Confirm private IP.
4. Confirm public IP is absent unless required.
5. Confirm NSG rules.
6. Confirm boot diagnostics.
7. Confirm identity state.
8. Use Run Command to validate guest OS.
9. Record final evidence.
```

# Create_Configure_And_Manage_Azure_Virtual_Machines_AzureCLI_Skeleton

```bash
# Run from Azure Cloud Shell or local Azure CLI.
# Purpose: create, configure, inspect, and manage Azure VMs.

# -----------------------------
# Variables
# -----------------------------

export SUBSCRIPTION_ID="<subscription-id>"
export LOCATION="eastus"
export RESOURCE_GROUP="rg-compute-lab-01"

export VNET_NAME="vnet-compute-lab-01"
export VNET_PREFIX="10.20.0.0/16"
export SUBNET_NAME="snet-vm-01"
export SUBNET_PREFIX="10.20.1.0/24"
export NSG_NAME="nsg-snet-vm-01"

export LINUX_VM_NAME="vm-lab-linux-01"
export WINDOWS_VM_NAME="vm-lab-win-01"
export VM_SIZE="Standard_B2s"
export ADMIN_USERNAME="azureuser"

export APPROVED_SOURCE_CIDR="<approved-source-cidr>"

export EVIDENCE_PATH="./evidence/02-azure-vms"

mkdir -p "$EVIDENCE_PATH"

# -----------------------------
# Login and context
# -----------------------------

az login

az account set \
  --subscription "$SUBSCRIPTION_ID"

az account show \
  --query "{user:user.name, tenant:tenantId, subscription:name, subscriptionId:id}" \
  --output table |
  tee "$EVIDENCE_PATH/account-context.txt"

# -----------------------------
# Providers
# -----------------------------

for provider in Microsoft.Compute Microsoft.Network Microsoft.Storage Microsoft.Insights Microsoft.Resources
do
  az provider register --namespace "$provider"
done

for provider in Microsoft.Compute Microsoft.Network Microsoft.Storage Microsoft.Insights Microsoft.Resources
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

# Optional scoped SSH rule for lab access.
# Prefer Bastion or private admin path instead of public SSH.

az network nsg rule create \
  --resource-group "$RESOURCE_GROUP" \
  --nsg-name "$NSG_NAME" \
  --name "Allow-SSH-Scoped" \
  --priority 1000 \
  --access Allow \
  --direction Inbound \
  --protocol Tcp \
  --source-address-prefixes "$APPROVED_SOURCE_CIDR" \
  --source-port-ranges "*" \
  --destination-address-prefixes "*" \
  --destination-port-ranges 22 |
  tee "$EVIDENCE_PATH/nsg-ssh-scoped-rule.json"

# Optional scoped RDP rule for lab access.
# Prefer Bastion or private admin path instead of public RDP.

az network nsg rule create \
  --resource-group "$RESOURCE_GROUP" \
  --nsg-name "$NSG_NAME" \
  --name "Allow-RDP-Scoped" \
  --priority 1010 \
  --access Allow \
  --direction Inbound \
  --protocol Tcp \
  --source-address-prefixes "$APPROVED_SOURCE_CIDR" \
  --source-port-ranges "*" \
  --destination-address-prefixes "*" \
  --destination-port-ranges 3389 |
  tee "$EVIDENCE_PATH/nsg-rdp-scoped-rule.json"

# -----------------------------
# Linux VM, private-only baseline
# -----------------------------

az vm create \
  --resource-group "$RESOURCE_GROUP" \
  --name "$LINUX_VM_NAME" \
  --location "$LOCATION" \
  --image Ubuntu2204 \
  --size "$VM_SIZE" \
  --admin-username "$ADMIN_USERNAME" \
  --generate-ssh-keys \
  --vnet-name "$VNET_NAME" \
  --subnet "$SUBNET_NAME" \
  --nsg "" \
  --public-ip-address "" \
  --tags Environment=Lab Workload=Compute OS=Linux ManagedBy=AzureCLI Workbook=02_Create_Configure_And_Manage_Azure_Virtual_Machines |
  tee "$EVIDENCE_PATH/linux-vm-create.json"

# -----------------------------
# Windows VM, private-only baseline
# -----------------------------
# Read password securely if running interactively.

read -s WINDOWS_ADMIN_PASSWORD

az vm create \
  --resource-group "$RESOURCE_GROUP" \
  --name "$WINDOWS_VM_NAME" \
  --location "$LOCATION" \
  --image Win2022Datacenter \
  --size "$VM_SIZE" \
  --admin-username "$ADMIN_USERNAME" \
  --admin-password "$WINDOWS_ADMIN_PASSWORD" \
  --vnet-name "$VNET_NAME" \
  --subnet "$SUBNET_NAME" \
  --nsg "" \
  --public-ip-address "" \
  --tags Environment=Lab Workload=Compute OS=Windows ManagedBy=AzureCLI Workbook=02_Create_Configure_And_Manage_Azure_Virtual_Machines |
  tee "$EVIDENCE_PATH/windows-vm-create.json"

# -----------------------------
# Managed identity
# -----------------------------

az vm identity assign \
  --resource-group "$RESOURCE_GROUP" \
  --name "$LINUX_VM_NAME" |
  tee "$EVIDENCE_PATH/linux-vm-identity.json"

az vm identity assign \
  --resource-group "$RESOURCE_GROUP" \
  --name "$WINDOWS_VM_NAME" |
  tee "$EVIDENCE_PATH/windows-vm-identity.json"

# -----------------------------
# Boot diagnostics
# -----------------------------

az vm boot-diagnostics enable \
  --resource-group "$RESOURCE_GROUP" \
  --name "$LINUX_VM_NAME" |
  tee "$EVIDENCE_PATH/linux-boot-diagnostics-enable.json"

az vm boot-diagnostics enable \
  --resource-group "$RESOURCE_GROUP" \
  --name "$WINDOWS_VM_NAME" |
  tee "$EVIDENCE_PATH/windows-boot-diagnostics-enable.json"

# -----------------------------
# Guest validation through Run Command
# -----------------------------

az vm run-command invoke \
  --resource-group "$RESOURCE_GROUP" \
  --name "$LINUX_VM_NAME" \
  --command-id RunShellScript \
  --scripts "hostname; uname -a; uptime; ip addr" |
  tee "$EVIDENCE_PATH/linux-run-command.json"

az vm run-command invoke \
  --resource-group "$RESOURCE_GROUP" \
  --name "$WINDOWS_VM_NAME" \
  --command-id RunPowerShellScript \
  --scripts "hostname; Get-ComputerInfo | Select-Object CsName,WindowsProductName,WindowsVersion; Get-NetIPConfiguration" |
  tee "$EVIDENCE_PATH/windows-run-command.json"

# -----------------------------
# Lifecycle operations
# -----------------------------

az vm get-instance-view \
  --resource-group "$RESOURCE_GROUP" \
  --name "$LINUX_VM_NAME" \
  --output json |
  tee "$EVIDENCE_PATH/linux-instance-view.json"

az vm stop \
  --resource-group "$RESOURCE_GROUP" \
  --name "$LINUX_VM_NAME"

az vm start \
  --resource-group "$RESOURCE_GROUP" \
  --name "$LINUX_VM_NAME"

az vm restart \
  --resource-group "$RESOURCE_GROUP" \
  --name "$LINUX_VM_NAME"

# Deallocate to release compute allocation.
az vm deallocate \
  --resource-group "$RESOURCE_GROUP" \
  --name "$LINUX_VM_NAME"

az vm start \
  --resource-group "$RESOURCE_GROUP" \
  --name "$LINUX_VM_NAME"

# -----------------------------
# Tag update
# -----------------------------

LINUX_VM_ID=$(az vm show \
  --resource-group "$RESOURCE_GROUP" \
  --name "$LINUX_VM_NAME" \
  --query id \
  --output tsv)

az resource tag \
  --ids "$LINUX_VM_ID" \
  --tags Environment=Lab Workload=Compute OS=Linux Owner="<owner>" ManagedBy=AzureCLI |
  tee "$EVIDENCE_PATH/linux-tag-update.json"

# -----------------------------
# Size inspection
# -----------------------------

az vm list-sizes \
  --location "$LOCATION" \
  --output table |
  tee "$EVIDENCE_PATH/regional-vm-sizes.txt"

az vm list-vm-resize-options \
  --resource-group "$RESOURCE_GROUP" \
  --name "$LINUX_VM_NAME" \
  --output table |
  tee "$EVIDENCE_PATH/linux-resize-options.txt"

# -----------------------------
# Final evidence
# -----------------------------

az vm list \
  --resource-group "$RESOURCE_GROUP" \
  --show-details \
  --output table |
  tee "$EVIDENCE_PATH/vm-list-final.txt"

az vm show \
  --resource-group "$RESOURCE_GROUP" \
  --name "$LINUX_VM_NAME" \
  --show-details \
  --output json |
  tee "$EVIDENCE_PATH/linux-vm-final.json"

az vm show \
  --resource-group "$RESOURCE_GROUP" \
  --name "$WINDOWS_VM_NAME" \
  --show-details \
  --output json |
  tee "$EVIDENCE_PATH/windows-vm-final.json"

az network nic list \
  --resource-group "$RESOURCE_GROUP" \
  --output json |
  tee "$EVIDENCE_PATH/nics-final.json"

az network nsg show \
  --resource-group "$RESOURCE_GROUP" \
  --name "$NSG_NAME" \
  --output json |
  tee "$EVIDENCE_PATH/nsg-final.json"

az resource list \
  --resource-group "$RESOURCE_GROUP" \
  --output table |
  tee "$EVIDENCE_PATH/resource-list-final.txt"
```

# Create_Configure_And_Manage_Azure_Virtual_Machines_PowerShell_Skeleton

```powershell
# Run from PowerShell with Az module installed.
# Purpose: create, configure, inspect, and manage Azure VMs.

# -----------------------------
# Variables
# -----------------------------

$SubscriptionId = "<subscription-id>"
$Location = "eastus"
$ResourceGroupName = "rg-compute-lab-01"

$VNetName = "vnet-compute-lab-01"
$VNetPrefix = "10.20.0.0/16"
$SubnetName = "snet-vm-01"
$SubnetPrefix = "10.20.1.0/24"
$NsgName = "nsg-snet-vm-01"

$LinuxVmName = "vm-lab-linux-01"
$WindowsVmName = "vm-lab-win-01"
$VmSize = "Standard_B2s"
$AdminUsername = "azureuser"

$EvidencePath = ".\evidence\02-azure-vms"

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

$Nsg = New-AzNetworkSecurityGroup `
  -ResourceGroupName $ResourceGroupName `
  -Location $Location `
  -Name $NsgName

$SubnetConfig = New-AzVirtualNetworkSubnetConfig `
  -Name $SubnetName `
  -AddressPrefix $SubnetPrefix `
  -NetworkSecurityGroup $Nsg

$VNet = New-AzVirtualNetwork `
  -ResourceGroupName $ResourceGroupName `
  -Location $Location `
  -Name $VNetName `
  -AddressPrefix $VNetPrefix `
  -Subnet $SubnetConfig

# -----------------------------
# Linux VM through Azure CLI for clean SSH key flow
# -----------------------------

az vm create `
  --resource-group $ResourceGroupName `
  --name $LinuxVmName `
  --location $Location `
  --image Ubuntu2204 `
  --size $VmSize `
  --admin-username $AdminUsername `
  --generate-ssh-keys `
  --vnet-name $VNetName `
  --subnet $SubnetName `
  --nsg "" `
  --public-ip-address "" `
  --tags Environment=Lab Workload=Compute OS=Linux ManagedBy=AzureCLI |
  Tee-Object (Join-Path $EvidencePath "linux-vm-create.txt")

# -----------------------------
# Windows VM through Azure CLI from PowerShell
# -----------------------------

$WindowsAdminPassword = Read-Host "Enter Windows admin password" -AsSecureString
$WindowsAdminPasswordPlain = [Runtime.InteropServices.Marshal]::PtrToStringAuto([Runtime.InteropServices.Marshal]::SecureStringToBSTR($WindowsAdminPassword))

az vm create `
  --resource-group $ResourceGroupName `
  --name $WindowsVmName `
  --location $Location `
  --image Win2022Datacenter `
  --size $VmSize `
  --admin-username $AdminUsername `
  --admin-password $WindowsAdminPasswordPlain `
  --vnet-name $VNetName `
  --subnet $SubnetName `
  --nsg "" `
  --public-ip-address "" `
  --tags Environment=Lab Workload=Compute OS=Windows ManagedBy=AzureCLI |
  Tee-Object (Join-Path $EvidencePath "windows-vm-create.txt")

# -----------------------------
# Identity and diagnostics
# -----------------------------

az vm identity assign `
  --resource-group $ResourceGroupName `
  --name $LinuxVmName |
  Tee-Object (Join-Path $EvidencePath "linux-identity.txt")

az vm identity assign `
  --resource-group $ResourceGroupName `
  --name $WindowsVmName |
  Tee-Object (Join-Path $EvidencePath "windows-identity.txt")

az vm boot-diagnostics enable `
  --resource-group $ResourceGroupName `
  --name $LinuxVmName |
  Tee-Object (Join-Path $EvidencePath "linux-boot-diagnostics.txt")

az vm boot-diagnostics enable `
  --resource-group $ResourceGroupName `
  --name $WindowsVmName |
  Tee-Object (Join-Path $EvidencePath "windows-boot-diagnostics.txt")

# -----------------------------
# Run command validation
# -----------------------------

Invoke-AzVMRunCommand `
  -ResourceGroupName $ResourceGroupName `
  -VMName $LinuxVmName `
  -CommandId "RunShellScript" `
  -ScriptString "hostname; uname -a; uptime" |
  Tee-Object (Join-Path $EvidencePath "linux-run-command.txt")

Invoke-AzVMRunCommand `
  -ResourceGroupName $ResourceGroupName `
  -VMName $WindowsVmName `
  -CommandId "RunPowerShellScript" `
  -ScriptString "hostname; Get-ComputerInfo | Select-Object CsName,WindowsProductName,WindowsVersion" |
  Tee-Object (Join-Path $EvidencePath "windows-run-command.txt")

# -----------------------------
# Lifecycle
# -----------------------------

Get-AzVM `
  -ResourceGroupName $ResourceGroupName `
  -Name $LinuxVmName `
  -Status |
  Tee-Object (Join-Path $EvidencePath "linux-status-before.txt")

Stop-AzVM `
  -ResourceGroupName $ResourceGroupName `
  -Name $LinuxVmName `
  -Force

Start-AzVM `
  -ResourceGroupName $ResourceGroupName `
  -Name $LinuxVmName

Restart-AzVM `
  -ResourceGroupName $ResourceGroupName `
  -Name $LinuxVmName

# -----------------------------
# Final evidence
# -----------------------------

Get-AzVM `
  -ResourceGroupName $ResourceGroupName `
  -Status |
  Tee-Object (Join-Path $EvidencePath "vm-status-final.txt")

Get-AzNetworkInterface `
  -ResourceGroupName $ResourceGroupName |
  Tee-Object (Join-Path $EvidencePath "nic-list-final.txt")

Get-AzNetworkSecurityGroup `
  -ResourceGroupName $ResourceGroupName `
  -Name $NsgName |
  Tee-Object (Join-Path $EvidencePath "nsg-final.txt")

Get-AzResource `
  -ResourceGroupName $ResourceGroupName |
  Tee-Object (Join-Path $EvidencePath "resource-list-final.txt")
```

# Create_Configure_And_Manage_Azure_Virtual_Machines_Bicep_Skeleton

```bicep
// File: .\infra\02-azure-vms\main.bicep
// Purpose: deploy a secure private-only Linux VM baseline with VNet, subnet, NSG, NIC, boot diagnostics, and managed identity.

targetScope = 'resourceGroup'

@description('Azure region.')
param location string = resourceGroup().location

@description('VM name.')
param vmName string = 'vm-lab-linux-01'

@description('Admin username.')
param adminUsername string = 'azureuser'

@description('SSH public key.')
param sshPublicKey string

@description('VM size.')
param vmSize string = 'Standard_B2s'

@description('VNet name.')
param vnetName string = 'vnet-compute-lab-01'

@description('VNet address prefix.')
param vnetPrefix string = '10.20.0.0/16'

@description('Subnet name.')
param subnetName string = 'snet-vm-01'

@description('Subnet address prefix.')
param subnetPrefix string = '10.20.1.0/24'

@description('NSG name.')
param nsgName string = 'nsg-snet-vm-01'

@description('NIC name.')
param nicName string = 'nic-vm-lab-linux-01'

@description('OS disk SKU.')
param osDiskSku string = 'StandardSSD_LRS'

var tags = {
  Environment: 'Lab'
  Workload: 'Compute'
  OS: 'Linux'
  ManagedBy: 'Bicep'
  Workbook: '02_Create_Configure_And_Manage_Azure_Virtual_Machines'
}

resource nsg 'Microsoft.Network/networkSecurityGroups@2023-11-01' = {
  name: nsgName
  location: location
  tags: tags
  properties: {
    securityRules: []
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

resource nic 'Microsoft.Network/networkInterfaces@2023-11-01' = {
  name: nicName
  location: location
  tags: tags
  properties: {
    ipConfigurations: [
      {
        name: 'ipconfig1'
        properties: {
          privateIPAllocationMethod: 'Dynamic'
          subnet: {
            id: resourceId('Microsoft.Network/virtualNetworks/subnets', vnet.name, subnetName)
          }
        }
      }
    ]
  }
}

resource vm 'Microsoft.Compute/virtualMachines@2024-03-01' = {
  name: vmName
  location: location
  tags: tags
  identity: {
    type: 'SystemAssigned'
  }
  properties: {
    hardwareProfile: {
      vmSize: vmSize
    }
    osProfile: {
      computerName: vmName
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
        publisher: 'Canonical'
        offer: '0001-com-ubuntu-server-jammy'
        sku: '22_04-lts-gen2'
        version: 'latest'
      }
      osDisk: {
        createOption: 'FromImage'
        deleteOption: 'Delete'
        managedDisk: {
          storageAccountType: osDiskSku
        }
      }
    }
    networkProfile: {
      networkInterfaces: [
        {
          id: nic.id
          properties: {
            deleteOption: 'Delete'
          }
        }
      ]
    }
    diagnosticsProfile: {
      bootDiagnostics: {
        enabled: true
      }
    }
  }
}

output vmId string = vm.id
output vmName string = vm.name
output vmPrivateIp string = nic.properties.ipConfigurations[0].properties.privateIPAddress
output principalId string = vm.identity.principalId
```

# Create_Configure_And_Manage_Azure_Virtual_Machines_Bicep_Parameters_Skeleton

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "location": {
      "value": "eastus"
    },
    "vmName": {
      "value": "vm-lab-linux-01"
    },
    "adminUsername": {
      "value": "azureuser"
    },
    "sshPublicKey": {
      "value": "<paste-ssh-public-key-here>"
    },
    "vmSize": {
      "value": "Standard_B2s"
    },
    "vnetName": {
      "value": "vnet-compute-lab-01"
    },
    "vnetPrefix": {
      "value": "10.20.0.0/16"
    },
    "subnetName": {
      "value": "snet-vm-01"
    },
    "subnetPrefix": {
      "value": "10.20.1.0/24"
    },
    "nsgName": {
      "value": "nsg-snet-vm-01"
    },
    "nicName": {
      "value": "nic-vm-lab-linux-01"
    },
    "osDiskSku": {
      "value": "StandardSSD_LRS"
    }
  }
}
```

# Create_Configure_And_Manage_Azure_Virtual_Machines_Linux_VM_Skeleton

```text
Purpose:
Create and validate a Linux VM baseline.

Recommended baseline:
1. Private IP only.
2. No public SSH unless scoped lab rule is explicitly required.
3. SSH key authentication.
4. Subnet-level NSG.
5. Boot diagnostics enabled.
6. System-assigned managed identity enabled if Azure access is required.
7. Tags applied.
8. Run Command validated.

CLI creation:
az vm create \
  --resource-group <rg> \
  --name <linux-vm-name> \
  --image Ubuntu2204 \
  --size <vm-size> \
  --admin-username <admin-username> \
  --generate-ssh-keys \
  --vnet-name <vnet-name> \
  --subnet <subnet-name> \
  --nsg "" \
  --public-ip-address "" \
  --tags Environment=Lab Workload=Compute OS=Linux

Guest validation:
az vm run-command invoke \
  --resource-group <rg> \
  --name <linux-vm-name> \
  --command-id RunShellScript \
  --scripts "hostname; uname -a; uptime; df -h; ip addr"

SSH validation if private path exists:
ssh <admin-username>@<private-ip>

Common Linux checks:
1. hostname
2. uname -a
3. uptime
4. df -h
5. ip addr
6. systemctl status waagent
7. cloud-init status
8. journalctl -u waagent --no-pager
```

# Create_Configure_And_Manage_Azure_Virtual_Machines_Windows_VM_Skeleton

```text
Purpose:
Create and validate a Windows VM baseline.

Recommended baseline:
1. Private IP only.
2. No public RDP unless scoped lab rule is explicitly required.
3. Strong password stored securely.
4. Subnet-level NSG.
5. Boot diagnostics enabled.
6. System-assigned managed identity enabled if Azure access is required.
7. Tags applied.
8. Run Command validated.

CLI creation:
az vm create \
  --resource-group <rg> \
  --name <windows-vm-name> \
  --image Win2022Datacenter \
  --size <vm-size> \
  --admin-username <admin-username> \
  --admin-password <secure-password> \
  --vnet-name <vnet-name> \
  --subnet <subnet-name> \
  --nsg "" \
  --public-ip-address "" \
  --tags Environment=Lab Workload=Compute OS=Windows

Guest validation:
az vm run-command invoke \
  --resource-group <rg> \
  --name <windows-vm-name> \
  --command-id RunPowerShellScript \
  --scripts "hostname; Get-ComputerInfo | Select-Object CsName,WindowsProductName,WindowsVersion; Get-NetIPConfiguration"

RDP validation if private path exists:
1. Use Bastion.
2. Use private IP.
3. Use approved admin account.
4. Confirm no broad public 3389 exposure.

Common Windows checks:
1. hostname
2. Get-ComputerInfo
3. Get-NetIPConfiguration
4. Get-Service WindowsAzureGuestAgent
5. Get-EventLog -LogName System -Newest 20
6. Test-NetConnection <target> -Port <port>
```

# Create_Configure_And_Manage_Azure_Virtual_Machines_Lifecycle_Operations_Skeleton

```text
Purpose:
Manage VM state safely after deployment.

Inspect:
az vm show -g <rg> -n <vm> --show-details -o json
az vm get-instance-view -g <rg> -n <vm> -o json
az vm list -g <rg> -d -o table

Start:
az vm start -g <rg> -n <vm>

Stop guest:
az vm stop -g <rg> -n <vm>

Deallocate:
az vm deallocate -g <rg> -n <vm>

Restart:
az vm restart -g <rg> -n <vm>

Redeploy:
az vm redeploy -g <rg> -n <vm>

Reapply:
az vm reapply -g <rg> -n <vm>

Run Linux command:
az vm run-command invoke \
  -g <rg> \
  -n <linux-vm> \
  --command-id RunShellScript \
  --scripts "hostname; uptime"

Run Windows command:
az vm run-command invoke \
  -g <rg> \
  -n <windows-vm> \
  --command-id RunPowerShellScript \
  --scripts "hostname; Get-Date"

Resize check:
az vm list-vm-resize-options -g <rg> -n <vm> -o table

Resize:
az vm resize -g <rg> -n <vm> --size <target-size>

Tag:
az resource tag --ids <vm-resource-id> --tags Environment=Lab Workload=Compute Owner=<owner>

Delete VM:
az vm delete -g <rg> -n <vm> --yes

Delete VM and related resources carefully:
1. Identify NIC.
2. Identify OS disk.
3. Identify public IP if any.
4. Identify NSG only if not shared.
5. Delete only the resources meant to be removed.
```

# Create_Configure_And_Manage_Azure_Virtual_Machines_Verification_Commands

```bash
# Context
az account show -o table

# Providers
az provider show --namespace Microsoft.Compute --query registrationState -o tsv
az provider show --namespace Microsoft.Network --query registrationState -o tsv
az provider show --namespace Microsoft.Storage --query registrationState -o tsv

# Resource group
az group show \
  --name "<resource-group-name>" \
  -o table

# VM inventory
az vm list \
  --resource-group "<resource-group-name>" \
  -d \
  -o table

# Specific VM
az vm show \
  --resource-group "<resource-group-name>" \
  --name "<vm-name>" \
  --show-details \
  -o json

# Instance view
az vm get-instance-view \
  --resource-group "<resource-group-name>" \
  --name "<vm-name>" \
  -o json

# VM power state
az vm get-instance-view \
  --resource-group "<resource-group-name>" \
  --name "<vm-name>" \
  --query "instanceView.statuses[?starts_with(code, 'PowerState/')].displayStatus" \
  -o tsv

# NICs
az network nic list \
  --resource-group "<resource-group-name>" \
  -o table

az network nic show \
  --resource-group "<resource-group-name>" \
  --name "<nic-name>" \
  -o json

# Private IP
az vm list-ip-addresses \
  --resource-group "<resource-group-name>" \
  --name "<vm-name>" \
  -o table

# NSG
az network nsg show \
  --resource-group "<resource-group-name>" \
  --name "<nsg-name>" \
  -o json

az network nsg rule list \
  --resource-group "<resource-group-name>" \
  --nsg-name "<nsg-name>" \
  -o table

# Effective NSG rules for NIC
az network nic list-effective-nsg \
  --resource-group "<resource-group-name>" \
  --name "<nic-name>" \
  -o table

# Boot diagnostics
az vm boot-diagnostics get-boot-log \
  --resource-group "<resource-group-name>" \
  --name "<vm-name>"

# Identity
az vm identity show \
  --resource-group "<resource-group-name>" \
  --name "<vm-name>" \
  -o json

# Linux Run Command
az vm run-command invoke \
  --resource-group "<resource-group-name>" \
  --name "<linux-vm-name>" \
  --command-id RunShellScript \
  --scripts "hostname; uname -a; uptime"

# Windows Run Command
az vm run-command invoke \
  --resource-group "<resource-group-name>" \
  --name "<windows-vm-name>" \
  --command-id RunPowerShellScript \
  --scripts "hostname; Get-ComputerInfo | Select-Object CsName,WindowsProductName,WindowsVersion"

# Sizes
az vm list-sizes \
  --location "<azure-region>" \
  -o table

az vm list-vm-resize-options \
  --resource-group "<resource-group-name>" \
  --name "<vm-name>" \
  -o table

# Activity log
az monitor activity-log list \
  --resource-group "<resource-group-name>" \
  --max-events 30 \
  -o table

# Final inventory
az resource list \
  --resource-group "<resource-group-name>" \
  -o table
```

```powershell
# Context
Get-AzContext

# Resource group
Get-AzResourceGroup -Name "<resource-group-name>"

# VM list
Get-AzVM `
  -ResourceGroupName "<resource-group-name>" `
  -Status

# Specific VM
Get-AzVM `
  -ResourceGroupName "<resource-group-name>" `
  -Name "<vm-name>" `
  -Status

# NICs
Get-AzNetworkInterface `
  -ResourceGroupName "<resource-group-name>"

# NSG
Get-AzNetworkSecurityGroup `
  -ResourceGroupName "<resource-group-name>" `
  -Name "<nsg-name>"

# Linux command
Invoke-AzVMRunCommand `
  -ResourceGroupName "<resource-group-name>" `
  -VMName "<linux-vm-name>" `
  -CommandId "RunShellScript" `
  -ScriptString "hostname; uname -a; uptime"

# Windows command
Invoke-AzVMRunCommand `
  -ResourceGroupName "<resource-group-name>" `
  -VMName "<windows-vm-name>" `
  -CommandId "RunPowerShellScript" `
  -ScriptString "hostname; Get-ComputerInfo | Select-Object CsName,WindowsProductName,WindowsVersion"

# Start, stop, restart
Start-AzVM `
  -ResourceGroupName "<resource-group-name>" `
  -Name "<vm-name>"

Stop-AzVM `
  -ResourceGroupName "<resource-group-name>" `
  -Name "<vm-name>" `
  -Force

Restart-AzVM `
  -ResourceGroupName "<resource-group-name>" `
  -Name "<vm-name>"
```

# Create_Configure_And_Manage_Azure_Virtual_Machines_Rollback

| Change | Rollback Command / Action | Risk |
|---|---|---|
| VM created for lab | `az vm delete --resource-group "<resource-group-name>" --name "<vm-name>" --yes` | VM is deleted but some related resources may remain |
| Resource group created for lab | `az group delete --name "<resource-group-name>" --yes --no-wait` | Deletes all resources in the group |
| Public IP created | `az network public-ip delete --resource-group "<resource-group-name>" --name "<public-ip-name>"` | Public access breaks |
| NIC created | `az network nic delete --resource-group "<resource-group-name>" --name "<nic-name>"` | NIC must be detached or VM deleted first |
| OS disk retained after VM delete | `az disk delete --resource-group "<resource-group-name>" --name "<os-disk-name>" --yes` | Disk data is lost |
| NSG rule opened SSH | `az network nsg rule delete --resource-group "<resource-group-name>" --nsg-name "<nsg-name>" --name Allow-SSH-Scoped` | SSH access removed |
| NSG rule opened RDP | `az network nsg rule delete --resource-group "<resource-group-name>" --nsg-name "<nsg-name>" --name Allow-RDP-Scoped` | RDP access removed |
| NSG associated to wrong subnet | `az network vnet subnet update --resource-group "<resource-group-name>" --vnet-name "<vnet-name>" --name "<subnet-name>" --network-security-group ""` | Subnet loses NSG protection |
| Managed identity enabled | `az vm identity remove --resource-group "<resource-group-name>" --name "<vm-name>"` | Azure resource access using identity fails |
| Role assigned to VM identity | `az role assignment delete --assignee "<principal-id>" --role "<role-name>" --scope "<scope>"` | App or script access to Azure resource fails |
| Boot diagnostics enabled | Leave enabled or disable in portal if required | Low risk, storage or diagnostics behavior changes |
| VM stopped | `az vm start --resource-group "<resource-group-name>" --name "<vm-name>"` | VM starts billing again |
| VM deallocated | `az vm start --resource-group "<resource-group-name>" --name "<vm-name>"` | Dynamic public IP may change if one was used |
| VM restarted | Wait for service recovery | Guest workload outage may occur |
| VM resized | Resize back to original size | May require deallocation |
| Tags changed | Reapply previous tags | Cost reporting may be temporarily inaccurate |
| Bad guest change via Run Command | Restore from backup, redeploy VM, or manually undo change | Run Command changes are inside guest and not automatically reversible |
| Wrong VM image used | Recreate VM with correct image | Existing guest state is lost unless backed up |

# Create_Configure_And_Manage_Azure_Virtual_Machines_Failure_Checks

| Symptom | Likely Layer | First Check | Fix |
|---|---|---|---|
| VM create fails with `AuthorizationFailed` | RBAC | `az account show`; role assignment at RG or subscription | Use correct subscription or assign required role |
| VM create fails with provider error | Provider registration | `az provider show --namespace Microsoft.Compute` | Register provider and retry |
| VM create fails with `SkuNotAvailable` | Region or size | Error detail and `az vm list-sizes` | Choose supported size or region |
| VM create fails with quota error | Subscription quota | Error detail and quota blade | Request quota or choose smaller size |
| VM name rejected | Naming | Error detail | Use valid Azure VM name |
| Windows password rejected | Credential policy | Error detail | Use compliant strong password |
| SSH key not accepted | SSH key format | Public key content | Use valid OpenSSH public key |
| VM created with public IP accidentally | Network config | `az vm list-ip-addresses` | Remove public IP or recreate private-only |
| Cannot SSH | NSG, route, public IP, private path, or guest SSH | Effective NSG and Run Command | Use Bastion, fix NSG, or start SSH service |
| Cannot RDP | NSG, route, public IP, private path, or guest RDP | Effective NSG and Run Command | Use Bastion, enable RDP, or fix NSG |
| Run Command fails | VM agent or guest issue | Instance view and agent status | Restart agent, restart VM, or troubleshoot guest |
| VM stuck creating | Platform provisioning | `az vm get-instance-view` | Inspect deployment operations and activity log |
| VM stuck starting | Guest boot issue | Boot diagnostics | Inspect boot log, serial console, or redeploy |
| Boot diagnostics empty | Diagnostics unsupported or not ready | Boot diagnostics state | Enable diagnostics and retry after boot |
| VM agent not ready | Guest agent issue | Instance view | Repair agent or redeploy |
| Deallocate did not stop billing for disks | Billing model | Resource list | Disks continue billing until deleted |
| Public IP changed after deallocate | Dynamic public IP | Public IP SKU and allocation method | Use static public IP if public IP is required |
| Identity principal ID missing | Identity not assigned | `az vm identity show` | Assign identity |
| Role assignment fails | RBAC or identity propagation | Principal ID and scope | Wait for identity propagation and retry |
| NSG rule has no effect | Wrong NSG or wrong subnet/NIC | Effective NSG | Associate correct NSG or edit priority |
| Deny rule overrides allow | NSG priority | Effective rules | Adjust priority and source |
| VM cannot reach internet | Route, NSG, firewall, or DNS | Run Command network tests | Fix route, DNS, NAT, or firewall path |
| VM cannot resolve DNS | DNS settings | `nslookup` or `Resolve-DnsName` | Fix VNet DNS or guest DNS |
| Resize option missing | Host cluster availability | `az vm list-vm-resize-options` | Deallocate or choose another size |
| Resize fails | SKU, quota, or region | Error detail | Fix quota or choose supported size |
| Delete leaves disks or NICs | Delete option or dependency | Resource list | Delete orphaned related resources carefully |
| Portal and CLI show different state | Cache or wrong context | Subscription and RG | Refresh portal and verify CLI context |
| Tags missing from child resources | Tags not inherited automatically | Resource tags | Apply tags to NIC, disk, and public IP as needed |

# Create_Configure_And_Manage_Azure_Virtual_Machines_Related_Labs

| Lab | Related Workbook | Skill Proven |
|---|---|---|
| Deploy ARM and Bicep baseline | `01_Deploy_ARM_Templates_Bicep_And_Exported_Deployments.md` | IaC deployment foundation |
| Create and manage Azure VMs | `02_Create_Configure_And_Manage_Azure_Virtual_Machines.md` | VM creation, configuration, identity, networking, diagnostics, and lifecycle |
| Configure VM disks, sizes, availability, extensions, and encryption | `03_Configure_VM_Disks_Sizes_Availability_Sets_Zones_Extensions_And_Encryption.md` | VM platform configuration |
| Configure VMSS images and scaling | `04_Configure_VMSS_Images_Scaling_And_Automation.md` | Scale set management |
| Deploy ACR, ACI, and Azure Container Apps | `05_Deploy_ACR_ACI_Azure_Container_Apps_Sizing_And_Scaling.md` | Container hosting and scaling |
| Deploy App Service plans and web apps | `06_Deploy_App_Service_Plans_Web_Apps_TLS_Custom_Domains_Backup_Networking_And_Slots.md` | PaaS web hosting |
| Configure compute managed identities and Key Vault references | `07_Configure_Compute_Managed_Identities_Key_Vault_Access_And_Secrets_References.md` | Secretless Azure resource access |
| Troubleshoot compute deployment and runtime issues | `08_Troubleshoot_Azure_VM_Container_App_Service_Deployment_And_Runtime_Issues.md` | VM and compute troubleshooting |