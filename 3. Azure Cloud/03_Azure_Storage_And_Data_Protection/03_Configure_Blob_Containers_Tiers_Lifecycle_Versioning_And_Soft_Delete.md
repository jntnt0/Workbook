# 03_Configure_VM_Disks_Sizes_Availability_Sets_Zones_Extensions_And_Encryption

# 03_Configure_VM_Disks_Sizes_Availability_Sets_Zones_Extensions_And_Encryption_Index

03_Configure_VM_Disks_Sizes_Availability_Sets_Zones_Extensions_And_Encryption.md  
Configure_VM_Disks_Sizes_Availability_Sets_Zones_Extensions_And_Encryption  
Configure_VM_Disks_Sizes_Availability_Sets_Zones_Extensions_And_Encryption_Source_Basis  
Configure_VM_Disks_Sizes_Availability_Sets_Zones_Extensions_And_Encryption_Mental_Model  
Configure_VM_Disks_Sizes_Availability_Sets_Zones_Extensions_And_Encryption_Planning_Table  
Configure_VM_Disks_Sizes_Availability_Sets_Zones_Extensions_And_Encryption_Configuration_Checklist  
Configure_VM_Disks_Sizes_Availability_Sets_Zones_Extensions_And_Encryption_Portal_Skeleton  
Configure_VM_Disks_Sizes_Availability_Sets_Zones_Extensions_And_Encryption_AzureCLI_Skeleton  
Configure_VM_Disks_Sizes_Availability_Sets_Zones_Extensions_And_Encryption_PowerShell_Skeleton  
Configure_VM_Disks_Sizes_Availability_Sets_Zones_Extensions_And_Encryption_Bicep_Skeleton  
Configure_VM_Disks_Sizes_Availability_Sets_Zones_Extensions_And_Encryption_Disk_Operations_Skeleton  
Configure_VM_Disks_Sizes_Availability_Sets_Zones_Extensions_And_Encryption_Size_And_Placement_Skeleton  
Configure_VM_Disks_Sizes_Availability_Sets_Zones_Extensions_And_Encryption_Extension_Skeleton  
Configure_VM_Disks_Sizes_Availability_Sets_Zones_Extensions_And_Encryption_Encryption_Skeleton  
Configure_VM_Disks_Sizes_Availability_Sets_Zones_Extensions_And_Encryption_Verification_Commands  
Configure_VM_Disks_Sizes_Availability_Sets_Zones_Extensions_And_Encryption_Rollback  
Configure_VM_Disks_Sizes_Availability_Sets_Zones_Extensions_And_Encryption_Failure_Checks  
Configure_VM_Disks_Sizes_Availability_Sets_Zones_Extensions_And_Encryption_Related_Labs  

# Configure_VM_Disks_Sizes_Availability_Sets_Zones_Extensions_And_Encryption_Source_Basis

| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Azure managed disks | OS disks, data disks, disk SKUs, performance tiers, snapshots, resizing, and attachment |
| Microsoft Learn | Azure VM sizes | VM size selection, resizing, supported capabilities, vCPU, memory, disk limits, and NIC limits |
| Microsoft Learn | Availability sets | Fault domains, update domains, single-region VM resiliency, and planned maintenance separation |
| Microsoft Learn | Availability zones | Zone placement, zonal VMs, zone resiliency, and zone-aware deployment |
| Microsoft Learn | Azure VM extensions | Post-deployment configuration, Custom Script Extension, monitoring agents, security agents, and extension troubleshooting |
| Microsoft Learn | Azure Disk Encryption | BitLocker for Windows, DM-Crypt for Linux, Key Vault dependency, and encryption status |
| Microsoft Learn | Encryption at host | Host-level encryption for VM temporary disks, cache, and disk data flowing between VM and storage |
| Microsoft Learn | Disk encryption sets | Customer-managed keys for managed disks using Key Vault and managed identity |
| Microsoft Learn | Azure Resource Manager and Bicep | Declarative VM, disk, extension, availability set, and encryption configuration |
| Azure operational practice | VM storage lifecycle | Attach, initialize, format, mount, resize, detach, snapshot, and delete |
| Azure operational practice | VM platform lifecycle | Resize, deallocate, restart, redeploy, reapply, inspect, and document |
| Azure operational practice | Secure VM baseline | Prefer private access, no broad public admin ports, least privilege Key Vault access, and evidence capture |

# Configure_VM_Disks_Sizes_Availability_Sets_Zones_Extensions_And_Encryption_Mental_Model

| Concept | Operational Meaning |
|---|---|
| OS disk | Managed disk that contains the VM operating system |
| Data disk | Additional managed disk attached to a VM for application or data storage |
| Temporary disk | Local host storage that is not durable and should not hold persistent data |
| Managed disk | Azure-managed block storage object attached to a VM |
| Disk SKU | Storage performance and redundancy choice such as Standard HDD, Standard SSD, Premium SSD, Premium SSD v2, or Ultra Disk |
| Disk caching | Host cache behavior for disk reads and writes |
| LUN | Logical unit number used by Azure to attach data disks to the VM |
| Snapshot | Point-in-time copy of a managed disk |
| Resize disk | Increase disk size at Azure disk layer, then extend partition and filesystem inside the guest OS |
| Resize VM | Change VM hardware profile, usually requiring a compatible size and sometimes deallocation |
| Availability set | Places VMs across fault and update domains inside a region |
| Fault domain | Physical separation inside an availability set |
| Update domain | Planned maintenance grouping inside an availability set |
| Availability zone | Physically separate datacenter zone inside supported regions |
| Zonal VM | VM pinned to a specific availability zone |
| Zone-redundant design | Application pattern that deploys multiple VMs across zones |
| VM extension | Azure control-plane delivered agent package for post-deployment configuration or monitoring |
| Custom Script Extension | Extension used to run a script inside the VM after deployment |
| Azure VM Agent | Guest agent that enables extensions and run command |
| Platform-managed key | Default Microsoft-managed encryption for managed disks at rest |
| Customer-managed key | Key Vault backed encryption key controlled by the customer |
| Disk encryption set | Azure resource that connects managed disks to a customer-managed key |
| Azure Disk Encryption | Guest-level OS/data disk encryption using BitLocker or DM-Crypt |
| Encryption at host | Host-level encryption that protects VM data before it reaches Azure Storage |
| First rule | Disk, size, zone, and encryption decisions should be made before production VM creation |
| Blunt rule | You cannot cleanly retrofit every placement choice after deployment; some changes require redeploy or migration |

# Configure_VM_Disks_Sizes_Availability_Sets_Zones_Extensions_And_Encryption_Planning_Table

| Item | Example | Decision |
|---|---|---|
| Tenant | `contoso.onmicrosoft.com` | `<tenant-name>` |
| Subscription ID | `00000000-0000-0000-0000-000000000000` | `<subscription-id>` |
| Resource group | `rg-compute-lab-01` | `<resource-group-name>` |
| Location | `eastus` | `<azure-region>` |
| VM name | `vm-lab-linux-01` | `<vm-name>` |
| OS type | `Linux` | `<Linux / Windows>` |
| Current VM size | `Standard_B2s` | `<current-vm-size>` |
| Target VM size | `Standard_D2s_v5` | `<target-vm-size>` |
| Availability model | `Zone` | `<none / availability-set / zone>` |
| Availability set name | `avset-compute-lab-01` | `<availability-set-name>` |
| Availability zone | `1` | `<zone-number>` |
| Fault domains | `2` | `<fault-domain-count>` |
| Update domains | `5` | `<update-domain-count>` |
| OS disk name | `vm-lab-linux-01_OsDisk_1` | `<os-disk-name>` |
| OS disk SKU | `Premium_LRS` | `<os-disk-sku>` |
| Data disk name | `disk-vm-lab-linux-01-data01` | `<data-disk-name>` |
| Data disk size | `128` | `<data-disk-size-gb>` |
| Data disk SKU | `Premium_LRS` | `<data-disk-sku>` |
| Data disk LUN | `0` | `<lun-number>` |
| Disk caching | `ReadWrite` | `<None / ReadOnly / ReadWrite>` |
| Filesystem | `xfs` | `<xfs / ext4 / ntfs>` |
| Linux mount point | `/data` | `<mount-point>` |
| Windows drive letter | `F` | `<drive-letter>` |
| Snapshot name | `snap-vm-lab-linux-01-data01-prechange` | `<snapshot-name>` |
| Extension name | `CustomScript` | `<extension-name>` |
| Extension publisher | `Microsoft.Azure.Extensions` | `<extension-publisher>` |
| Extension purpose | `Bootstrap package install` | `<extension-purpose>` |
| Encryption baseline | `PMK plus encryption at host` | `<PMK / CMK / ADE / encryption-at-host>` |
| Key Vault name | `kv-compute-lab-01` | `<key-vault-name>` |
| Disk encryption set name | `des-compute-lab-01` | `<disk-encryption-set-name>` |
| Key name | `cmk-disk-01` | `<key-name>` |
| Evidence path | `.\evidence\03-vm-platform-config` | `<evidence-path>` |
| Rollback posture | `Snapshot before disk change; delete lab RG if needed` | `<rollback-plan>` |

# Configure_VM_Disks_Sizes_Availability_Sets_Zones_Extensions_And_Encryption_Configuration_Checklist

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm Azure context | Admin Workstation | `az account show -o table` | Correct subscription is active |
| 2 | Set subscription | Admin Workstation | `az account set --subscription "<subscription-id>"` | CLI targets correct subscription |
| 3 | Confirm resource group | Admin Workstation | `az group show --name "<resource-group-name>" -o table` | Resource group exists |
| 4 | Confirm VM exists | Admin Workstation | `az vm show --resource-group "<resource-group-name>" --name "<vm-name>" -o table` | VM is found |
| 5 | Capture current VM state | Admin Workstation | `az vm get-instance-view --resource-group "<resource-group-name>" --name "<vm-name>" -o json > .\evidence\vm-instance-before.json` | Baseline state is saved |
| 6 | Capture current disk inventory | Admin Workstation | `az vm show --resource-group "<resource-group-name>" --name "<vm-name>" --query "storageProfile" -o json > .\evidence\vm-storage-before.json` | Disk baseline is saved |
| 7 | Capture current size | Admin Workstation | `az vm show --resource-group "<resource-group-name>" --name "<vm-name>" --query "hardwareProfile.vmSize" -o tsv` | Current VM size is known |
| 8 | List available resize options | Admin Workstation | `az vm list-vm-resize-options --resource-group "<resource-group-name>" --name "<vm-name>" -o table` | Compatible sizes are visible |
| 9 | Deallocate VM before resize if required | Admin Workstation | `az vm deallocate --resource-group "<resource-group-name>" --name "<vm-name>"` | VM is deallocated |
| 10 | Resize VM | Admin Workstation | `az vm resize --resource-group "<resource-group-name>" --name "<vm-name>" --size "<target-vm-size>"` | VM hardware profile changes |
| 11 | Start VM after resize | Admin Workstation | `az vm start --resource-group "<resource-group-name>" --name "<vm-name>"` | VM starts |
| 12 | Verify new size | Admin Workstation | `az vm show --resource-group "<resource-group-name>" --name "<vm-name>" --query "hardwareProfile.vmSize" -o tsv` | Target size is shown |
| 13 | Create data disk | Admin Workstation | `az disk create --resource-group "<resource-group-name>" --name "<data-disk-name>" --size-gb "<data-disk-size-gb>" --sku "<data-disk-sku>" --location "<azure-region>"` | Managed data disk exists |
| 14 | Attach data disk | Admin Workstation | `az vm disk attach --resource-group "<resource-group-name>" --vm-name "<vm-name>" --name "<data-disk-name>" --lun "<lun-number>" --caching "<disk-caching>"` | Disk is attached to VM |
| 15 | Verify data disk attachment | Admin Workstation | `az vm show --resource-group "<resource-group-name>" --name "<vm-name>" --query "storageProfile.dataDisks" -o table` | Data disk appears |
| 16 | Initialize Linux data disk | Admin Workstation | `az vm run-command invoke --resource-group "<resource-group-name>" --name "<vm-name>" --command-id RunShellScript --scripts "lsblk"` | Guest sees attached disk |
| 17 | Initialize Windows data disk | Admin Workstation | `az vm run-command invoke --resource-group "<resource-group-name>" --name "<vm-name>" --command-id RunPowerShellScript --scripts "Get-Disk"` | Guest sees attached disk |
| 18 | Create snapshot before disk resize | Admin Workstation | `az snapshot create --resource-group "<resource-group-name>" --name "<snapshot-name>" --source "<data-disk-name>"` | Disk snapshot exists |
| 19 | Resize data disk at Azure layer | Admin Workstation | `az disk update --resource-group "<resource-group-name>" --name "<data-disk-name>" --size-gb "<new-size-gb>"` | Managed disk size increases |
| 20 | Extend filesystem inside Linux | Admin Workstation | `az vm run-command invoke --resource-group "<resource-group-name>" --name "<vm-name>" --command-id RunShellScript --scripts "df -h; lsblk"` | Guest-side expansion can be validated |
| 21 | Extend filesystem inside Windows | Admin Workstation | `az vm run-command invoke --resource-group "<resource-group-name>" --name "<vm-name>" --command-id RunPowerShellScript --scripts "Get-Partition; Get-Volume"` | Guest-side expansion can be validated |
| 22 | Create availability set for new VMs if required | Admin Workstation | `az vm availability-set create --resource-group "<resource-group-name>" --name "<availability-set-name>" --platform-fault-domain-count 2 --platform-update-domain-count 5` | Availability set exists |
| 23 | Confirm VM placement | Admin Workstation | `az vm show --resource-group "<resource-group-name>" --name "<vm-name>" --query "{zones:zones,availabilitySet:availabilitySet.id}" -o json` | Zone or availability set state is known |
| 24 | Install Linux Custom Script Extension | Admin Workstation | `az vm extension set --resource-group "<resource-group-name>" --vm-name "<vm-name>" --publisher Microsoft.Azure.Extensions --name CustomScript --settings '{"commandToExecute":"echo extension-test"}'` | Extension installs and runs |
| 25 | Install Windows Custom Script Extension | Admin Workstation | `az vm extension set --resource-group "<resource-group-name>" --vm-name "<vm-name>" --publisher Microsoft.Compute --name CustomScriptExtension --settings '{"commandToExecute":"powershell -ExecutionPolicy Bypass -Command \"hostname\""}'` | Extension installs and runs |
| 26 | Verify extension state | Admin Workstation | `az vm extension list --resource-group "<resource-group-name>" --vm-name "<vm-name>" -o table` | Extension state is visible |
| 27 | Confirm encryption at rest | Admin Workstation | `az disk show --resource-group "<resource-group-name>" --name "<os-disk-name>" --query "{name:name,encryption:encryption}" -o json` | Disk encryption settings are visible |
| 28 | Enable encryption at host if supported | Admin Workstation | `az vm update --resource-group "<resource-group-name>" --name "<vm-name>" --set securityProfile.encryptionAtHost=true` | Host encryption is set if supported |
| 29 | Create Key Vault for disk encryption set if using CMK | Admin Workstation | `az keyvault create --resource-group "<resource-group-name>" --name "<key-vault-name>" --location "<azure-region>" --enable-purge-protection true` | Key Vault exists with purge protection |
| 30 | Create Key Vault key | Admin Workstation | `az keyvault key create --vault-name "<key-vault-name>" --name "<key-name>" --kty RSA --size 2048` | Key exists |
| 31 | Create disk encryption set | Admin Workstation | `az disk-encryption-set create --resource-group "<resource-group-name>" --name "<disk-encryption-set-name>" --location "<azure-region>" --source-vault "<key-vault-name>" --key-url "<key-url>"` | Disk encryption set exists |
| 32 | Verify encryption set identity | Admin Workstation | `az disk-encryption-set show --resource-group "<resource-group-name>" --name "<disk-encryption-set-name>" --query identity.principalId -o tsv` | Managed identity principal ID is visible |
| 33 | Assign Key Vault key access for DES identity | Admin Workstation | `az role assignment create --assignee "<des-principal-id>" --role "Key Vault Crypto Service Encryption User" --scope "<key-vault-resource-id>"` | DES identity can use key |
| 34 | Apply CMK to new managed disk if required | Admin Workstation | `az disk update --resource-group "<resource-group-name>" --name "<data-disk-name>" --disk-encryption-set "<disk-encryption-set-id>"` | Disk uses disk encryption set if supported |
| 35 | Verify final VM platform state | Admin Workstation | `az vm show --resource-group "<resource-group-name>" --name "<vm-name>" --show-details -o json > .\evidence\vm-final.json` | Final VM evidence is saved |
| 36 | Document completion | Operator | `Record VM size, disk list, placement model, extensions, and encryption state` | Workbook evidence is complete |

# Configure_VM_Disks_Sizes_Availability_Sets_Zones_Extensions_And_Encryption_Portal_Skeleton

```text
Purpose:
Configure the VM platform layer from the Azure portal.

Portal path:
Azure portal
> Virtual machines
> <vm-name>

Disk inspection:
1. Open the VM.
2. Go to Settings > Disks.
3. Record:
   - OS disk name
   - OS disk size
   - OS disk SKU
   - Data disks
   - LUNs
   - Caching
   - Encryption type
   - Delete with VM setting

Attach data disk:
1. VM > Disks.
2. Select Create and attach a new disk.
3. Name: <data-disk-name>
4. Size: <data-disk-size-gb>
5. Disk SKU: <data-disk-sku>
6. Host caching: <None / ReadOnly / ReadWrite>
7. Select Save.
8. Sign in to the guest OS and initialize the disk.

Resize data disk:
1. Create a snapshot first if the disk has data.
2. VM > Disks.
3. Select the disk name.
4. Go to Size + performance.
5. Increase disk size.
6. Save.
7. Extend the partition and filesystem inside the guest OS.

Resize VM:
1. VM > Size.
2. Review available sizes.
3. Pick target size.
4. Resize.
5. If the desired size is unavailable, deallocate VM and retry.
6. Start VM and verify the new size.

Availability set:
1. Availability set selection is normally made during VM creation.
2. Existing VMs cannot be casually added to an availability set from the portal after creation.
3. If availability set placement is required, plan it before deployment or redeploy.

Availability zone:
1. Zone placement is normally selected during VM creation.
2. Existing VM zone changes require migration or redeployment planning.
3. For resilient designs, deploy multiple VMs across zones behind a load balancer or application tier.

Extensions:
1. VM > Extensions + applications.
2. Select Add.
3. Choose extension, such as Custom Script Extension.
4. Provide required settings.
5. Install extension.
6. Verify provisioning status.

Encryption:
1. VM > Disks.
2. Review encryption type on each managed disk.
3. Confirm platform-managed key or customer-managed key.
4. For customer-managed keys, confirm Disk Encryption Set and Key Vault configuration.
5. For Azure Disk Encryption, confirm guest-level encryption status.
6. For encryption at host, confirm VM security profile if supported.

Evidence:
1. Screenshot or record VM size.
2. Record disk inventory.
3. Record placement model.
4. Record extension status.
5. Record encryption state.
```

# Configure_VM_Disks_Sizes_Availability_Sets_Zones_Extensions_And_Encryption_AzureCLI_Skeleton

```bash
# Run from Azure Cloud Shell or local Azure CLI.
# Purpose: configure VM disks, resize, extensions, placement evidence, and encryption evidence.

# -----------------------------
# Variables
# -----------------------------

export SUBSCRIPTION_ID="<subscription-id>"
export LOCATION="eastus"
export RESOURCE_GROUP="rg-compute-lab-01"
export VM_NAME="vm-lab-linux-01"

export TARGET_VM_SIZE="Standard_D2s_v5"

export DATA_DISK_NAME="disk-vm-lab-linux-01-data01"
export DATA_DISK_SIZE_GB="128"
export DATA_DISK_NEW_SIZE_GB="256"
export DATA_DISK_SKU="Premium_LRS"
export DATA_DISK_LUN="0"
export DATA_DISK_CACHING="ReadWrite"

export SNAPSHOT_NAME="snap-vm-lab-linux-01-data01-prechange"

export AVSET_NAME="avset-compute-lab-01"

export KEYVAULT_NAME="kv-compute-lab-01"
export KEY_NAME="cmk-disk-01"
export DES_NAME="des-compute-lab-01"

export EVIDENCE_PATH="./evidence/03-vm-platform-config"

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
# Baseline evidence
# -----------------------------

az vm show \
  --resource-group "$RESOURCE_GROUP" \
  --name "$VM_NAME" \
  --show-details \
  --output json |
  tee "$EVIDENCE_PATH/vm-before.json"

az vm get-instance-view \
  --resource-group "$RESOURCE_GROUP" \
  --name "$VM_NAME" \
  --output json |
  tee "$EVIDENCE_PATH/vm-instance-before.json"

az vm show \
  --resource-group "$RESOURCE_GROUP" \
  --name "$VM_NAME" \
  --query "storageProfile" \
  --output json |
  tee "$EVIDENCE_PATH/vm-storage-before.json"

# -----------------------------
# VM resize
# -----------------------------

az vm list-vm-resize-options \
  --resource-group "$RESOURCE_GROUP" \
  --name "$VM_NAME" \
  --output table |
  tee "$EVIDENCE_PATH/vm-resize-options.txt"

az vm deallocate \
  --resource-group "$RESOURCE_GROUP" \
  --name "$VM_NAME"

az vm resize \
  --resource-group "$RESOURCE_GROUP" \
  --name "$VM_NAME" \
  --size "$TARGET_VM_SIZE"

az vm start \
  --resource-group "$RESOURCE_GROUP" \
  --name "$VM_NAME"

az vm show \
  --resource-group "$RESOURCE_GROUP" \
  --name "$VM_NAME" \
  --query "{name:name,size:hardwareProfile.vmSize,location:location}" \
  --output table |
  tee "$EVIDENCE_PATH/vm-size-after.txt"

# -----------------------------
# Create and attach data disk
# -----------------------------

az disk create \
  --resource-group "$RESOURCE_GROUP" \
  --name "$DATA_DISK_NAME" \
  --location "$LOCATION" \
  --size-gb "$DATA_DISK_SIZE_GB" \
  --sku "$DATA_DISK_SKU" |
  tee "$EVIDENCE_PATH/data-disk-create.json"

az vm disk attach \
  --resource-group "$RESOURCE_GROUP" \
  --vm-name "$VM_NAME" \
  --name "$DATA_DISK_NAME" \
  --lun "$DATA_DISK_LUN" \
  --caching "$DATA_DISK_CACHING" |
  tee "$EVIDENCE_PATH/data-disk-attach.json"

az vm show \
  --resource-group "$RESOURCE_GROUP" \
  --name "$VM_NAME" \
  --query "storageProfile.dataDisks" \
  --output table |
  tee "$EVIDENCE_PATH/data-disk-list-after-attach.txt"

# -----------------------------
# Linux guest disk initialization
# -----------------------------
# Review device names before running destructive disk commands.
# This example assumes /dev/sdc is the new disk.

az vm run-command invoke \
  --resource-group "$RESOURCE_GROUP" \
  --name "$VM_NAME" \
  --command-id RunShellScript \
  --scripts "lsblk; df -h" |
  tee "$EVIDENCE_PATH/linux-disk-before-format.json"

az vm run-command invoke \
  --resource-group "$RESOURCE_GROUP" \
  --name "$VM_NAME" \
  --command-id RunShellScript \
  --scripts "sudo parted /dev/sdc --script mklabel gpt mkpart xfspart xfs 0% 100%; sudo mkfs.xfs -f /dev/sdc1; sudo mkdir -p /data; sudo mount /dev/sdc1 /data; df -h; lsblk" |
  tee "$EVIDENCE_PATH/linux-disk-format-mount.json"

# -----------------------------
# Snapshot before disk resize
# -----------------------------

az snapshot create \
  --resource-group "$RESOURCE_GROUP" \
  --name "$SNAPSHOT_NAME" \
  --source "$DATA_DISK_NAME" |
  tee "$EVIDENCE_PATH/data-disk-snapshot.json"

# -----------------------------
# Resize managed disk
# -----------------------------
# Some disk changes require VM deallocation depending on disk type, attachment state, and SKU.

az vm deallocate \
  --resource-group "$RESOURCE_GROUP" \
  --name "$VM_NAME"

az disk update \
  --resource-group "$RESOURCE_GROUP" \
  --name "$DATA_DISK_NAME" \
  --size-gb "$DATA_DISK_NEW_SIZE_GB" |
  tee "$EVIDENCE_PATH/data-disk-resize.json"

az vm start \
  --resource-group "$RESOURCE_GROUP" \
  --name "$VM_NAME"

# Linux guest filesystem expansion example.
# Validate device and partition before running this in a real system.

az vm run-command invoke \
  --resource-group "$RESOURCE_GROUP" \
  --name "$VM_NAME" \
  --command-id RunShellScript \
  --scripts "sudo growpart /dev/sdc 1 || true; sudo xfs_growfs /data || true; df -h; lsblk" |
  tee "$EVIDENCE_PATH/linux-disk-grow.json"

# -----------------------------
# Availability set for future VMs
# -----------------------------
# Existing VMs are not casually moved into an availability set.
# Create this before deploying grouped VMs.

az vm availability-set create \
  --resource-group "$RESOURCE_GROUP" \
  --name "$AVSET_NAME" \
  --location "$LOCATION" \
  --platform-fault-domain-count 2 \
  --platform-update-domain-count 5 |
  tee "$EVIDENCE_PATH/availability-set-create.json"

az vm availability-set show \
  --resource-group "$RESOURCE_GROUP" \
  --name "$AVSET_NAME" \
  --output json |
  tee "$EVIDENCE_PATH/availability-set-show.json"

# -----------------------------
# Placement evidence
# -----------------------------

az vm show \
  --resource-group "$RESOURCE_GROUP" \
  --name "$VM_NAME" \
  --query "{name:name,zones:zones,availabilitySet:availabilitySet.id,proximityPlacementGroup:proximityPlacementGroup.id}" \
  --output json |
  tee "$EVIDENCE_PATH/vm-placement.json"

# -----------------------------
# Custom Script Extension, Linux
# -----------------------------

az vm extension set \
  --resource-group "$RESOURCE_GROUP" \
  --vm-name "$VM_NAME" \
  --publisher Microsoft.Azure.Extensions \
  --name CustomScript \
  --settings '{"commandToExecute":"echo extension-test | sudo tee /var/tmp/extension-test.txt"}' |
  tee "$EVIDENCE_PATH/linux-custom-script-extension.json"

az vm extension list \
  --resource-group "$RESOURCE_GROUP" \
  --vm-name "$VM_NAME" \
  --output table |
  tee "$EVIDENCE_PATH/vm-extension-list.txt"

# -----------------------------
# Encryption at host
# -----------------------------
# Requires supported VM size, region, and subscription feature state.

az vm update \
  --resource-group "$RESOURCE_GROUP" \
  --name "$VM_NAME" \
  --set securityProfile.encryptionAtHost=true |
  tee "$EVIDENCE_PATH/vm-encryption-at-host-update.json"

az vm show \
  --resource-group "$RESOURCE_GROUP" \
  --name "$VM_NAME" \
  --query "securityProfile" \
  --output json |
  tee "$EVIDENCE_PATH/vm-security-profile.json"

# -----------------------------
# Disk encryption evidence
# -----------------------------

OS_DISK_NAME=$(az vm show \
  --resource-group "$RESOURCE_GROUP" \
  --name "$VM_NAME" \
  --query "storageProfile.osDisk.name" \
  --output tsv)

az disk show \
  --resource-group "$RESOURCE_GROUP" \
  --name "$OS_DISK_NAME" \
  --query "{name:name,sku:sku.name,diskSizeGB:diskSizeGB,encryption:encryption}" \
  --output json |
  tee "$EVIDENCE_PATH/os-disk-encryption.json"

az disk show \
  --resource-group "$RESOURCE_GROUP" \
  --name "$DATA_DISK_NAME" \
  --query "{name:name,sku:sku.name,diskSizeGB:diskSizeGB,encryption:encryption}" \
  --output json |
  tee "$EVIDENCE_PATH/data-disk-encryption.json"

# -----------------------------
# Final inventory
# -----------------------------

az vm show \
  --resource-group "$RESOURCE_GROUP" \
  --name "$VM_NAME" \
  --show-details \
  --output json |
  tee "$EVIDENCE_PATH/vm-final.json"

az resource list \
  --resource-group "$RESOURCE_GROUP" \
  --output table |
  tee "$EVIDENCE_PATH/resource-list-final.txt"
```

# Configure_VM_Disks_Sizes_Availability_Sets_Zones_Extensions_And_Encryption_AzureCLI_Windows_Disk_Skeleton

```bash
# Purpose:
# Initialize, format, and resize a Windows data disk through Azure Run Command.

export RESOURCE_GROUP="rg-compute-lab-01"
export VM_NAME="vm-lab-win-01"

# Inspect disks.
az vm run-command invoke \
  --resource-group "$RESOURCE_GROUP" \
  --name "$VM_NAME" \
  --command-id RunPowerShellScript \
  --scripts "Get-Disk | Sort-Object Number | Format-Table Number,FriendlyName,PartitionStyle,OperationalStatus,Size" \
  --output json

# Initialize the first RAW disk and assign drive F.
az vm run-command invoke \
  --resource-group "$RESOURCE_GROUP" \
  --name "$VM_NAME" \
  --command-id RunPowerShellScript \
  --scripts "$disk = Get-Disk | Where-Object PartitionStyle -eq 'RAW' | Select-Object -First 1; Initialize-Disk -Number $disk.Number -PartitionStyle GPT; New-Partition -DiskNumber $disk.Number -UseMaximumSize -DriveLetter F | Format-Volume -FileSystem NTFS -NewFileSystemLabel 'Data01' -Confirm:$false; Get-Volume" \
  --output json

# Extend drive F after Azure disk size was increased.
az vm run-command invoke \
  --resource-group "$RESOURCE_GROUP" \
  --name "$VM_NAME" \
  --command-id RunPowerShellScript \
  --scripts "$size = Get-PartitionSupportedSize -DriveLetter F; Resize-Partition -DriveLetter F -Size $size.SizeMax; Get-Volume -DriveLetter F" \
  --output json
```

# Configure_VM_Disks_Sizes_Availability_Sets_Zones_Extensions_And_Encryption_PowerShell_Skeleton

```powershell
# Run from PowerShell with Az module installed.
# Purpose: configure VM disk, size, extension, placement evidence, and encryption evidence.

# -----------------------------
# Variables
# -----------------------------

$SubscriptionId = "<subscription-id>"
$Location = "eastus"
$ResourceGroupName = "rg-compute-lab-01"
$VmName = "vm-lab-linux-01"

$TargetVmSize = "Standard_D2s_v5"

$DataDiskName = "disk-vm-lab-linux-01-data01"
$DataDiskSizeGb = 128
$DataDiskNewSizeGb = 256
$DataDiskSku = "Premium_LRS"
$DataDiskLun = 0
$DataDiskCaching = "ReadWrite"

$SnapshotName = "snap-vm-lab-linux-01-data01-prechange"
$AvailabilitySetName = "avset-compute-lab-01"

$EvidencePath = ".\evidence\03-vm-platform-config"

New-Item -ItemType Directory -Force -Path $EvidencePath

# -----------------------------
# Context
# -----------------------------

Connect-AzAccount

Set-AzContext -SubscriptionId $SubscriptionId

Get-AzContext |
  Tee-Object (Join-Path $EvidencePath "az-context.txt")

# -----------------------------
# Baseline VM evidence
# -----------------------------

Get-AzVM `
  -ResourceGroupName $ResourceGroupName `
  -Name $VmName `
  -Status |
  Tee-Object (Join-Path $EvidencePath "vm-status-before.txt")

Get-AzVM `
  -ResourceGroupName $ResourceGroupName `
  -Name $VmName |
  Tee-Object (Join-Path $EvidencePath "vm-model-before.txt")

# -----------------------------
# Resize VM
# -----------------------------

Get-AzVMSize `
  -ResourceGroupName $ResourceGroupName `
  -VMName $VmName |
  Tee-Object (Join-Path $EvidencePath "vm-resize-options.txt")

Stop-AzVM `
  -ResourceGroupName $ResourceGroupName `
  -Name $VmName `
  -Force

$Vm = Get-AzVM `
  -ResourceGroupName $ResourceGroupName `
  -Name $VmName

$Vm.HardwareProfile.VmSize = $TargetVmSize

Update-AzVM `
  -ResourceGroupName $ResourceGroupName `
  -VM $Vm

Start-AzVM `
  -ResourceGroupName $ResourceGroupName `
  -Name $VmName

Get-AzVM `
  -ResourceGroupName $ResourceGroupName `
  -Name $VmName |
  Select-Object Name, Location, @{Name="Size";Expression={$_.HardwareProfile.VmSize}} |
  Tee-Object (Join-Path $EvidencePath "vm-size-after.txt")

# -----------------------------
# Create and attach data disk
# -----------------------------

$DiskConfig = New-AzDiskConfig `
  -Location $Location `
  -CreateOption Empty `
  -DiskSizeGB $DataDiskSizeGb `
  -SkuName $DataDiskSku

$DataDisk = New-AzDisk `
  -ResourceGroupName $ResourceGroupName `
  -DiskName $DataDiskName `
  -Disk $DiskConfig

$Vm = Get-AzVM `
  -ResourceGroupName $ResourceGroupName `
  -Name $VmName

$Vm = Add-AzVMDataDisk `
  -VM $Vm `
  -Name $DataDiskName `
  -CreateOption Attach `
  -ManagedDiskId $DataDisk.Id `
  -Lun $DataDiskLun `
  -Caching $DataDiskCaching

Update-AzVM `
  -ResourceGroupName $ResourceGroupName `
  -VM $Vm

Get-AzVM `
  -ResourceGroupName $ResourceGroupName `
  -Name $VmName |
  Select-Object -ExpandProperty StorageProfile |
  Tee-Object (Join-Path $EvidencePath "vm-storage-after-disk-attach.txt")

# -----------------------------
# Snapshot before resize
# -----------------------------

$SnapshotConfig = New-AzSnapshotConfig `
  -SourceUri $DataDisk.Id `
  -Location $Location `
  -CreateOption Copy

New-AzSnapshot `
  -ResourceGroupName $ResourceGroupName `
  -SnapshotName $SnapshotName `
  -Snapshot $SnapshotConfig |
  Tee-Object (Join-Path $EvidencePath "snapshot-create.txt")

# -----------------------------
# Resize managed disk
# -----------------------------

Stop-AzVM `
  -ResourceGroupName $ResourceGroupName `
  -Name $VmName `
  -Force

$Disk = Get-AzDisk `
  -ResourceGroupName $ResourceGroupName `
  -DiskName $DataDiskName

$Disk.DiskSizeGB = $DataDiskNewSizeGb

Update-AzDisk `
  -ResourceGroupName $ResourceGroupName `
  -DiskName $DataDiskName `
  -Disk $Disk |
  Tee-Object (Join-Path $EvidencePath "data-disk-resize.txt")

Start-AzVM `
  -ResourceGroupName $ResourceGroupName `
  -Name $VmName

# -----------------------------
# Availability set for new grouped VMs
# -----------------------------

New-AzAvailabilitySet `
  -ResourceGroupName $ResourceGroupName `
  -Name $AvailabilitySetName `
  -Location $Location `
  -PlatformFaultDomainCount 2 `
  -PlatformUpdateDomainCount 5 `
  -Sku aligned |
  Tee-Object (Join-Path $EvidencePath "availability-set-create.txt")

Get-AzAvailabilitySet `
  -ResourceGroupName $ResourceGroupName `
  -Name $AvailabilitySetName |
  Tee-Object (Join-Path $EvidencePath "availability-set-show.txt")

# -----------------------------
# Extension example
# -----------------------------

Set-AzVMExtension `
  -ResourceGroupName $ResourceGroupName `
  -VMName $VmName `
  -Location $Location `
  -Publisher "Microsoft.Azure.Extensions" `
  -ExtensionType "CustomScript" `
  -Name "CustomScript" `
  -TypeHandlerVersion "2.1" `
  -SettingString '{"commandToExecute":"echo extension-test | sudo tee /var/tmp/extension-test.txt"}' |
  Tee-Object (Join-Path $EvidencePath "custom-script-extension.txt")

Get-AzVMExtension `
  -ResourceGroupName $ResourceGroupName `
  -VMName $VmName |
  Tee-Object (Join-Path $EvidencePath "vm-extensions.txt")

# -----------------------------
# Encryption evidence
# -----------------------------

$Vm = Get-AzVM `
  -ResourceGroupName $ResourceGroupName `
  -Name $VmName

$OsDiskName = $Vm.StorageProfile.OsDisk.Name

Get-AzDisk `
  -ResourceGroupName $ResourceGroupName `
  -DiskName $OsDiskName |
  Select-Object Name, DiskSizeGB, Sku, Encryption |
  Tee-Object (Join-Path $EvidencePath "os-disk-encryption.txt")

Get-AzDisk `
  -ResourceGroupName $ResourceGroupName `
  -DiskName $DataDiskName |
  Select-Object Name, DiskSizeGB, Sku, Encryption |
  Tee-Object (Join-Path $EvidencePath "data-disk-encryption.txt")
```

# Configure_VM_Disks_Sizes_Availability_Sets_Zones_Extensions_And_Encryption_Bicep_Skeleton

```bicep
// File: .\infra\03-vm-platform-config\main.bicep
// Purpose: create a VM with explicit disk, size, availability zone, extension, and encryption-at-host posture.
// Customer-managed key and Azure Disk Encryption are handled in separate skeletons because they require Key Vault sequencing.

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
param vmSize string = 'Standard_D2s_v5'

@description('Availability zone for the VM. Use an empty array if deploying regionally.')
param zones array = [
  '1'
]

@description('VNet name.')
param vnetName string = 'vnet-compute-lab-01'

@description('Subnet name.')
param subnetName string = 'snet-vm-01'

@description('VNet prefix.')
param vnetPrefix string = '10.20.0.0/16'

@description('Subnet prefix.')
param subnetPrefix string = '10.20.1.0/24'

@description('NSG name.')
param nsgName string = 'nsg-snet-vm-01'

@description('NIC name.')
param nicName string = 'nic-vm-lab-linux-01'

@description('OS disk type.')
param osDiskSku string = 'Premium_LRS'

@description('Data disk name.')
param dataDiskName string = 'disk-vm-lab-linux-01-data01'

@description('Data disk size in GB.')
param dataDiskSizeGB int = 128

@description('Data disk SKU.')
param dataDiskSku string = 'Premium_LRS'

@description('Enable encryption at host if supported.')
param encryptionAtHost bool = true

var tags = {
  Environment: 'Lab'
  Workload: 'Compute'
  ManagedBy: 'Bicep'
  Workbook: '03_Configure_VM_Disks_Sizes_Availability_Sets_Zones_Extensions_And_Encryption'
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

resource dataDisk 'Microsoft.Compute/disks@2023-10-02' = {
  name: dataDiskName
  location: location
  tags: tags
  sku: {
    name: dataDiskSku
  }
  zones: zones
  properties: {
    creationData: {
      createOption: 'Empty'
    }
    diskSizeGB: dataDiskSizeGB
  }
}

resource vm 'Microsoft.Compute/virtualMachines@2024-03-01' = {
  name: vmName
  location: location
  zones: zones
  tags: tags
  properties: {
    hardwareProfile: {
      vmSize: vmSize
    }
    securityProfile: {
      encryptionAtHost: encryptionAtHost
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
        managedDisk: {
          storageAccountType: osDiskSku
        }
        deleteOption: 'Delete'
      }
      dataDisks: [
        {
          lun: 0
          name: dataDisk.name
          createOption: 'Attach'
          managedDisk: {
            id: dataDisk.id
          }
          caching: 'ReadWrite'
          deleteOption: 'Delete'
        }
      ]
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

resource customScript 'Microsoft.Compute/virtualMachines/extensions@2024-03-01' = {
  name: '${vm.name}/CustomScript'
  location: location
  properties: {
    publisher: 'Microsoft.Azure.Extensions'
    type: 'CustomScript'
    typeHandlerVersion: '2.1'
    autoUpgradeMinorVersion: true
    settings: {
      commandToExecute: 'echo extension-test | sudo tee /var/tmp/extension-test.txt'
    }
  }
  dependsOn: [
    vm
  ]
}

output vmId string = vm.id
output vmName string = vm.name
output dataDiskId string = dataDisk.id
output zone array = zones
```

# Configure_VM_Disks_Sizes_Availability_Sets_Zones_Extensions_And_Encryption_Disk_Operations_Skeleton

```text
Purpose:
Operate managed disks safely.

Attach data disk:
1. Create managed disk.
2. Attach disk to VM at a specific LUN.
3. Confirm Azure shows disk attached.
4. Confirm guest OS sees disk.
5. Initialize partition table.
6. Create filesystem.
7. Mount or assign drive letter.
8. Persist mount if Linux.
9. Document final path.

Linux disk initialization:
1. Inspect disks:
   lsblk
   df -h

2. Partition new disk:
   sudo parted /dev/sdc --script mklabel gpt mkpart xfspart xfs 0% 100%

3. Create filesystem:
   sudo mkfs.xfs -f /dev/sdc1

4. Create mount point:
   sudo mkdir -p /data

5. Mount:
   sudo mount /dev/sdc1 /data

6. Persist with UUID:
   sudo blkid /dev/sdc1
   sudo vi /etc/fstab

7. Validate:
   df -h
   mount -a

Windows disk initialization:
1. Inspect:
   Get-Disk

2. Initialize:
   Initialize-Disk -Number <disk-number> -PartitionStyle GPT

3. Partition and format:
   New-Partition -DiskNumber <disk-number> -UseMaximumSize -DriveLetter F |
     Format-Volume -FileSystem NTFS -NewFileSystemLabel Data01 -Confirm:$false

4. Validate:
   Get-Volume
   Get-Partition

Resize disk:
1. Create snapshot first.
2. Increase managed disk size in Azure.
3. Start VM if it was deallocated.
4. Extend partition inside guest.
5. Extend filesystem.
6. Validate free space.
7. Document old size, new size, and snapshot.

Detach disk:
1. Stop services using disk.
2. Unmount or remove drive letter inside guest.
3. Detach disk in Azure.
4. Confirm disk is unattached.
5. Keep disk or delete disk based on rollback plan.
```

# Configure_VM_Disks_Sizes_Availability_Sets_Zones_Extensions_And_Encryption_Size_And_Placement_Skeleton

```text
Purpose:
Control VM hardware size and resiliency placement.

VM resize:
1. Capture current size:
   az vm show -g <rg> -n <vm> --query "hardwareProfile.vmSize" -o tsv

2. List resize options:
   az vm list-vm-resize-options -g <rg> -n <vm> -o table

3. If target size is listed, resize directly:
   az vm resize -g <rg> -n <vm> --size <target-size>

4. If target size is not listed:
   az vm deallocate -g <rg> -n <vm>
   az vm resize -g <rg> -n <vm> --size <target-size>
   az vm start -g <rg> -n <vm>

5. Verify size:
   az vm show -g <rg> -n <vm> --query "hardwareProfile.vmSize" -o tsv

Availability set:
1. Use for multiple VMs in the same region when you need fault and update domain separation.
2. Select availability set at VM creation.
3. Do not assume an existing VM can be casually moved into an availability set.
4. For existing production VMs, plan redeploy or migration.

Availability zone:
1. Use when region supports zones.
2. Select zone at VM creation.
3. Deploy at least two application instances across zones for resiliency.
4. Pair with zone-aware load balancing or application routing.
5. Do not assume a single zonal VM is highly available.

Decision:
- Single lab VM: no availability set or zone required.
- Two or more regional VMs: availability set.
- Zone-capable resilient workload: multiple zonal VMs across zones.
- App-tier scale design: consider VMSS in task 04.
```

# Configure_VM_Disks_Sizes_Availability_Sets_Zones_Extensions_And_Encryption_Extension_Skeleton

```text
Purpose:
Install and validate VM extensions.

Extension prerequisites:
1. VM is running.
2. Azure VM Agent is healthy.
3. Guest OS is supported.
4. Outbound network path exists if extension downloads content.
5. Script URL or command is correct.
6. Secrets are not hardcoded in public settings.

Linux Custom Script Extension:
az vm extension set \
  --resource-group <rg> \
  --vm-name <vm> \
  --publisher Microsoft.Azure.Extensions \
  --name CustomScript \
  --settings '{"commandToExecute":"echo extension-test | sudo tee /var/tmp/extension-test.txt"}'

Windows Custom Script Extension:
az vm extension set \
  --resource-group <rg> \
  --vm-name <vm> \
  --publisher Microsoft.Compute \
  --name CustomScriptExtension \
  --settings '{"commandToExecute":"powershell -ExecutionPolicy Bypass -Command \"hostname\""}'

Verify:
1. az vm extension list -g <rg> --vm-name <vm> -o table
2. az vm get-instance-view -g <rg> -n <vm> -o json
3. Check guest logs if extension fails.

Common extension failure causes:
1. VM agent not ready.
2. Script exits with nonzero code.
3. Script cannot reach package repository.
4. File URI is blocked or wrong.
5. Protected settings are malformed.
6. Extension handler version issue.
7. Guest OS unsupported.
```

# Configure_VM_Disks_Sizes_Availability_Sets_Zones_Extensions_And_Encryption_Encryption_Skeleton

```text
Purpose:
Understand and configure VM disk encryption layers.

Layer 1:
Platform-managed key encryption at rest.
- Default for Azure managed disks.
- Microsoft manages the keys.
- Good baseline for lab and many standard workloads.

Layer 2:
Customer-managed key with Disk Encryption Set.
- Uses Key Vault key.
- Disk Encryption Set has managed identity.
- DES identity needs access to Key Vault key.
- Useful when customer controls key lifecycle.

Layer 3:
Encryption at host.
- Encrypts VM data on the host before it reaches Azure Storage.
- Protects temp disk and disk cache path where supported.
- Requires supported VM size, region, and subscription capability.

Layer 4:
Azure Disk Encryption.
- Guest-level encryption with BitLocker or DM-Crypt.
- Requires Key Vault.
- More operational complexity.
- Validate OS support before using.

Customer-managed key flow:
1. Create Key Vault with purge protection.
2. Create or import key.
3. Create Disk Encryption Set using Key Vault key.
4. Assign Key Vault crypto permissions to DES identity.
5. Create new disks with DES or update eligible disks.
6. Verify disk encryption property.
7. Document key, DES, disk IDs, and recovery process.

Encryption at host flow:
1. Confirm VM size and region support.
2. Enable encryption at host at VM creation or update if supported.
3. Verify security profile.
4. Restart or redeploy if required by platform behavior.
5. Document state.

Azure Disk Encryption flow:
1. Create Key Vault enabled for disk encryption.
2. Confirm VM OS support.
3. Enable encryption.
4. Monitor encryption status.
5. Record recovery key and Key Vault dependency.
6. Test reboot and access.
```

# Configure_VM_Disks_Sizes_Availability_Sets_Zones_Extensions_And_Encryption_Verification_Commands

```bash
# Account and VM
az account show -o table

az vm show \
  --resource-group "<resource-group-name>" \
  --name "<vm-name>" \
  --show-details \
  -o json

az vm get-instance-view \
  --resource-group "<resource-group-name>" \
  --name "<vm-name>" \
  -o json

# Size
az vm show \
  --resource-group "<resource-group-name>" \
  --name "<vm-name>" \
  --query "hardwareProfile.vmSize" \
  -o tsv

az vm list-vm-resize-options \
  --resource-group "<resource-group-name>" \
  --name "<vm-name>" \
  -o table

# Placement
az vm show \
  --resource-group "<resource-group-name>" \
  --name "<vm-name>" \
  --query "{name:name,location:location,zones:zones,availabilitySet:availabilitySet.id}" \
  -o json

az vm availability-set list \
  --resource-group "<resource-group-name>" \
  -o table

# Disks
az vm show \
  --resource-group "<resource-group-name>" \
  --name "<vm-name>" \
  --query "storageProfile.osDisk" \
  -o json

az vm show \
  --resource-group "<resource-group-name>" \
  --name "<vm-name>" \
  --query "storageProfile.dataDisks" \
  -o table

az disk list \
  --resource-group "<resource-group-name>" \
  -o table

az disk show \
  --resource-group "<resource-group-name>" \
  --name "<disk-name>" \
  -o json

# Snapshot
az snapshot list \
  --resource-group "<resource-group-name>" \
  -o table

# Linux guest disk state
az vm run-command invoke \
  --resource-group "<resource-group-name>" \
  --name "<linux-vm-name>" \
  --command-id RunShellScript \
  --scripts "lsblk; df -h; mount | grep data || true"

# Windows guest disk state
az vm run-command invoke \
  --resource-group "<resource-group-name>" \
  --name "<windows-vm-name>" \
  --command-id RunPowerShellScript \
  --scripts "Get-Disk; Get-Volume; Get-Partition"

# Extensions
az vm extension list \
  --resource-group "<resource-group-name>" \
  --vm-name "<vm-name>" \
  -o table

az vm extension show \
  --resource-group "<resource-group-name>" \
  --vm-name "<vm-name>" \
  --name "<extension-name>" \
  -o json

# Encryption at host
az vm show \
  --resource-group "<resource-group-name>" \
  --name "<vm-name>" \
  --query "securityProfile" \
  -o json

# Managed disk encryption
az disk show \
  --resource-group "<resource-group-name>" \
  --name "<disk-name>" \
  --query "{name:name,encryption:encryption}" \
  -o json

# Azure Disk Encryption status
az vm encryption show \
  --resource-group "<resource-group-name>" \
  --name "<vm-name>"

# Activity log
az monitor activity-log list \
  --resource-group "<resource-group-name>" \
  --max-events 25 \
  -o table
```

```powershell
# Context
Get-AzContext

# VM status
Get-AzVM `
  -ResourceGroupName "<resource-group-name>" `
  -Name "<vm-name>" `
  -Status

# VM size
(Get-AzVM `
  -ResourceGroupName "<resource-group-name>" `
  -Name "<vm-name>").HardwareProfile.VmSize

# Available sizes
Get-AzVMSize `
  -ResourceGroupName "<resource-group-name>" `
  -VMName "<vm-name>"

# Disks
$Vm = Get-AzVM `
  -ResourceGroupName "<resource-group-name>" `
  -Name "<vm-name>"

$Vm.StorageProfile.OsDisk
$Vm.StorageProfile.DataDisks

Get-AzDisk `
  -ResourceGroupName "<resource-group-name>"

# Extensions
Get-AzVMExtension `
  -ResourceGroupName "<resource-group-name>" `
  -VMName "<vm-name>"

# Placement
$Vm.AvailabilitySetReference
$Vm.Zones

# Availability sets
Get-AzAvailabilitySet `
  -ResourceGroupName "<resource-group-name>"

# Guest disk inspection, Windows
Invoke-AzVMRunCommand `
  -ResourceGroupName "<resource-group-name>" `
  -VMName "<windows-vm-name>" `
  -CommandId "RunPowerShellScript" `
  -ScriptString "Get-Disk; Get-Volume"

# Guest disk inspection, Linux
Invoke-AzVMRunCommand `
  -ResourceGroupName "<resource-group-name>" `
  -VMName "<linux-vm-name>" `
  -CommandId "RunShellScript" `
  -ScriptString "lsblk; df -h"
```

# Configure_VM_Disks_Sizes_Availability_Sets_Zones_Extensions_And_Encryption_Rollback

| Change | Rollback Command / Action | Risk |
|---|---|---|
| VM resized | Resize back to original VM size | May require deallocation |
| VM deallocated | `az vm start --resource-group "<resource-group-name>" --name "<vm-name>"` | Public dynamic IP may change if used |
| Data disk attached | `az vm disk detach --resource-group "<resource-group-name>" --vm-name "<vm-name>" --name "<data-disk-name>"` | Guest mount may break |
| Data disk formatted | Restore from snapshot or backup | Formatting destroys previous contents |
| Data disk resized larger | No simple shrink path | Restore to smaller disk from snapshot if needed |
| Snapshot created | `az snapshot delete --resource-group "<resource-group-name>" --name "<snapshot-name>"` | Removes rollback point |
| Disk SKU changed | Change back if supported | Performance or cost changes |
| Disk caching changed | Revert caching value | App performance may change |
| Availability set created | `az vm availability-set delete --resource-group "<resource-group-name>" --name "<availability-set-name>"` | Only delete if no VMs depend on it |
| Zone placement chosen incorrectly | Redeploy or migrate VM | Existing VM zone changes are not casual |
| Extension installed | `az vm extension delete --resource-group "<resource-group-name>" --vm-name "<vm-name>" --name "<extension-name>"` | Extension changes inside guest may remain |
| Extension script changed guest OS | Use backup, snapshot, or manual remediation | Extension removal does not always undo script actions |
| Encryption at host enabled | Disable if supported or recreate VM | Support depends on VM and region |
| Disk encryption set applied | Revert to allowed encryption configuration if supported | CMK dependency may remain |
| Azure Disk Encryption enabled | Disable ADE carefully and validate guest boot | Incorrect handling can cause boot or data access issues |
| Key Vault key disabled | Re-enable key immediately | CMK-protected disks may become inaccessible |
| Key Vault access removed | Restore DES identity access | Disk operations may fail |
| Lab RG created | `az group delete --name "<resource-group-name>" --yes --no-wait` | Deletes all lab resources |

# Configure_VM_Disks_Sizes_Availability_Sets_Zones_Extensions_And_Encryption_Failure_Checks

| Symptom | Likely Layer | First Check | Fix |
|---|---|---|---|
| VM resize option missing | Size availability or cluster constraint | `az vm list-vm-resize-options` | Deallocate VM or choose available size |
| Resize fails with `SkuNotAvailable` | Region or zone does not support size | Error detail and size list | Pick another size, region, or zone |
| Resize fails with quota error | Subscription quota | Quotas blade or error detail | Request quota or choose smaller size |
| VM will not start after resize | Size, host, or guest issue | Instance view and boot diagnostics | Resize back, redeploy, or inspect guest |
| Disk attach fails | Disk region mismatch or VM limit | Disk location and VM size limits | Use same region and supported disk count |
| Disk attach succeeds but guest does not see disk | Guest rescan needed | `lsblk` or `Get-Disk` | Rescan, reboot, or check VM agent |
| Wrong disk was formatted | Operator error | Snapshot and disk IDs | Restore from snapshot if available |
| Linux mount missing after reboot | `/etc/fstab` not configured | `cat /etc/fstab` | Add UUID-based mount entry |
| Linux VM fails boot after fstab edit | Bad fstab entry | Boot diagnostics or serial console | Fix fstab from recovery path |
| Windows disk has no drive letter | Partition missing or offline | `Get-Disk`, `Get-Partition` | Initialize, online, partition, format |
| Disk resize does not show in guest | Partition not extended | Guest disk tools | Extend partition and filesystem |
| Cannot shrink disk | Managed disks do not support simple shrink | Disk size | Restore to smaller disk from backup or snapshot |
| Snapshot fails | Disk name or permission issue | Disk ID and RBAC | Correct source and permissions |
| Availability set not selectable for existing VM | Placement chosen at creation | VM placement properties | Redeploy VM into availability set |
| Zone not selectable | Region or SKU does not support zones | Region and size support | Choose zone-capable region and size |
| Extension stuck creating | VM agent or script issue | Extension instance view | Check agent, logs, and script exit code |
| Extension failed with nonzero exit | Script error | Extension message | Fix script and rerun extension |
| Extension cannot download script | Network or URL issue | Guest outbound and URL | Fix outbound path or use accessible URI |
| VM extension list empty after install | Wrong VM or RG | CLI context and VM name | Correct context and rerun |
| Encryption at host update fails | Unsupported size, region, or state | Error detail | Use supported size or recreate with setting |
| Disk encryption set creation fails | Key Vault or key issue | Key URL and vault settings | Enable purge protection, correct key URL |
| CMK disk access fails | DES identity lacks Key Vault role | Role assignments | Assign Key Vault Crypto Service Encryption User |
| ADE enable fails | Key Vault, OS support, or extension issue | `az vm encryption show` and extension logs | Fix Key Vault access and OS prerequisites |
| VM fails after encryption | Guest encryption issue | Boot diagnostics | Use recovery procedure and Key Vault validation |
| Disk shows PMK when CMK expected | Disk not associated with DES | `az disk show --query encryption` | Apply correct disk encryption set |
| Costs increased after resize | Larger size or disk SKU | Cost analysis and resource list | Resize down, deallocate, or delete unused disks |

# Configure_VM_Disks_Sizes_Availability_Sets_Zones_Extensions_And_Encryption_Related_Labs

| Lab | Related Workbook | Skill Proven |
|---|---|---|
| Deploy ARM and Bicep baseline | `01_Deploy_ARM_Templates_Bicep_And_Exported_Deployments.md` | Declarative deployment foundation |
| Create and manage Azure VMs | `02_Create_Configure_And_Manage_Azure_Virtual_Machines.md` | VM deployment and lifecycle operations |
| Configure VM disks, sizes, availability, extensions, and encryption | `03_Configure_VM_Disks_Sizes_Availability_Sets_Zones_Extensions_And_Encryption.md` | VM platform configuration |
| Configure VMSS images and scaling | `04_Configure_VMSS_Images_Scaling_And_Automation.md` | Scaled VM compute |
| Deploy ACR, ACI, and Container Apps | `05_Deploy_ACR_ACI_Azure_Container_Apps_Sizing_And_Scaling.md` | Container compute hosting |
| Deploy App Service plans and web apps | `06_Deploy_App_Service_Plans_Web_Apps_TLS_Custom_Domains_Backup_Networking_And_Slots.md` | PaaS app hosting |
| Configure managed identities and Key Vault references | `07_Configure_Compute_Managed_Identities_Key_Vault_Access_And_Secrets_References.md` | Secretless access and Key Vault integration |
| Troubleshoot compute deployment and runtime issues | `08_Troubleshoot_Azure_VM_Container_App_Service_Deployment_And_Runtime_Issues.md` | Azure compute troubleshooting |