## Overview

This guide covers deploying Cisco Modeling Labs (CML) on Microsoft Azure using the official CiscoDevNet Terraform module. It includes infrastructure provisioning, blob storage setup for CML images, daily lifecycle management, and teardown procedure.

## Environment Reference

|Component|Value|
|---|---|
|CML Version|2.9.1|
|Cloud|Microsoft Azure|
|VM SKU|Standard_D8ds_v4 (8 vCPU, 32 GB RAM)|
|Terraform Module|github.com/CiscoDevNet/cloud-cml|
|CML SSH Port|1122 (not 22)|
|CML SSH User|sysadmin|

---

## Prerequisites

Install the following on your local Windows machine before starting:

- [Azure CLI](https://aka.ms/installazurecliwindows)
- [Terraform](https://developer.hashicorp.com/terraform/install)
- [Git](https://git-scm.com/)
- [AzCopy](https://aka.ms/downloadazcopy-v10-windows)

---

## Deployment Procedure

### Step 1 — Azure Login

```powershell
az login
```

### Step 2 — Create Resource Group

```powershell
az group create --name <resource-group> --location <region>
```

### Step 3 — Create Storage Account

```powershell
az storage account create `
  --name <storage-account-name> `
  --resource-group <resource-group> `
  --location <region> `
  --sku Standard_LRS `
  --encryption-services blob
```

### Step 4 — Assign Blob Contributor Role

```powershell
az ad signed-in-user show --query id -o tsv | `
  az role assignment create `
    --role "Storage Blob Data Contributor" `
    --assignee @- `
    --scope "/subscriptions/<subscription-id>/resourceGroups/<resource-group>/providers/Microsoft.Storage/storageAccounts/<storage-account-name>"
```

### Step 5 — Create Blob Container

```powershell
az storage container create `
  --account-name <storage-account-name> `
  --name <container-name> `
  --auth-mode login
```

### Step 6 — Create SSH Key

```powershell
az sshkey create `
  --name <key-name> `
  --resource-group <resource-group> `
  --encryption-type Ed25519
```

In Cloud Shell, rename the generated key and set permissions:

```bash
mv ~/.ssh/<generated-name> ~/.ssh/cml-key
mv ~/.ssh/<generated-name>.pub ~/.ssh/cml-key.pub
chmod 600 ~/.ssh/cml-key
cat ~/.ssh/cml-key
```

Save the private key output to your local machine with no file extension.

### Step 7 — Upload CML Files via AzCopy

Login AzCopy:

```powershell
.\azcopy login
```

Upload the CML pkg file:

```powershell
.\azcopy copy "<path-to-pkg-file>" "https://<storage-account-name>.blob.core.windows.net/<container-name>/<pkg-filename>"
```

Mount the refplat ISO and upload its contents:

```powershell
.\azcopy copy "<drive-letter>:\*" "https://<storage-account-name>.blob.core.windows.net/<container-name>/refplat" --recursive
```

> **Note:** Upload only the pkg file and refplat ISO contents. Bare metal ISOs and VMware OVAs are not needed and will waste storage.

### Step 8 — Clone and Prepare the CiscoDevNet Repo

```powershell
git clone https://github.com/CiscoDevNet/cloud-cml.git <local-path>
cd <local-path>
.\prepare.bat
```

When prompted: AWS=no, Azure=yes, Conjur=no, Vault=no

### Step 9 — Edit variables.tf

```powershell
notepad <local-path>\variables.tf
```

Set the following:

- `azure_subscription_id` — your Azure subscription ID
- `azure_tenant_id` — your Azure tenant ID

> **Note:** Default values in `variables.tf` are `notset` and will cause deployment to fail if not replaced.

### Step 10 — Configure config.yml

Copy your `config.yml` into the repo root and verify these key fields:

```yaml
target: azure
resource_group: <resource-group>
storage_account: <storage-account-name>
container_name: <container-name>
key_name: <key-name>
software: <pkg-filename>          # Must exactly match the blob filename
flavor: CML_Personal
smartlicense_token: <your-token>
```

> **Note:** The `software` field must exactly match what is in blob storage, including filename casing. Refplat image entries in `config.yml` must also exactly match what was uploaded.

### Step 11 — Deploy

```powershell
cd <local-path>
terraform init
terraform apply
```

Type `yes` when prompted. Deployment takes 20-30 minutes. Do not close the terminal or lose internet connectivity during this process.

### Step 12 — Retrieve Credentials

```powershell
terraform output cml2secrets
```

### Step 13 — Access CML

Navigate to `https://<ip-from-terraform-output>` in a browser and log in with the credentials from the previous step.

---

## Daily Lifecycle Management

CML on Azure incurs compute costs while running. Deallocate the VM when not in use.

**Stop (deallocate):**

```powershell
az vm deallocate --resource-group <resource-group> --name <vm-name>
```

**Start:**

```powershell
az vm start --resource-group <resource-group> --name <vm-name>
```

**Get current public IP after start** (IP may change on each start):

```powershell
az vm show -d `
  --resource-group <resource-group> `
  --name <vm-name> `
  --query publicIps -o tsv
```

> **Note:** Use `az vm deallocate` for daily stop/start. Only use `terraform destroy` when permanently tearing down the environment.

---

## Teardown Procedure

### Step 1 — Deregister CML License

This must be done before destroying infrastructure or the license seat will not be released:

```powershell
ssh -p1122 -i <path-to-ssh-key> sysadmin@<cml-public-ip> /provision/del.sh
```

### Step 2 — Destroy Infrastructure

```powershell
cd <local-path>
terraform destroy
```

### Step 3 — Optional: Delete Resource Group

If you want to preserve blob storage (avoids re-uploading images on next deployment), do not delete the resource group. If a full wipe is needed:

```powershell
az group delete --name <resource-group> --yes
```

---

## Key Lessons Learned

|Issue|Resolution|
|---|---|
|Azure Cloud Shell unreliable for long-running jobs|Run Terraform from local Windows PowerShell instead|
|Cloud-init completes silently without running CML install script|If browser times out post-deploy, the install script did not run — check cloud-init logs|
|`variables.tf` defaults to `notset`|Must set real subscription and tenant IDs before `terraform init`|
|`prepare.bat` must run before `terraform init`|It wires up the Azure provider modules|
|CML SSH uses port 1122, not 22|Always specify `-p1122` and `sysadmin` as the user|
|`software` field mismatch causes deployment failure|Filename in `config.yml` must exactly match the blob filename|
|Refplat image mismatch causes deployment failure|Only list images in `config.yml` that are actually present in blob storage|
|License not released after destroy|Always run `/provision/del.sh` via SSH before `terraform destroy`|