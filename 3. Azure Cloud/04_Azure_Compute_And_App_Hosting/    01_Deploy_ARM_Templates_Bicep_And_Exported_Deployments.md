# 01_Deploy_ARM_Templates_Bicep_And_Exported_Deployments

# 01_Deploy_ARM_Templates_Bicep_And_Exported_Deployments_Index

01_Deploy_ARM_Templates_Bicep_And_Exported_Deployments.md  
Deploy_ARM_Templates_Bicep_And_Exported_Deployments  
Deploy_ARM_Templates_Bicep_And_Exported_Deployments_Source_Basis  
Deploy_ARM_Templates_Bicep_And_Exported_Deployments_Mental_Model  
Deploy_ARM_Templates_Bicep_And_Exported_Deployments_Planning_Table  
Deploy_ARM_Templates_Bicep_And_Exported_Deployments_Configuration_Checklist  
Deploy_ARM_Templates_Bicep_And_Exported_Deployments_Portal_Skeleton  
Deploy_ARM_Templates_Bicep_And_Exported_Deployments_AzureCLI_Skeleton  
Deploy_ARM_Templates_Bicep_And_Exported_Deployments_PowerShell_Skeleton  
Deploy_ARM_Templates_Bicep_And_Exported_Deployments_Bicep_Skeleton  
Deploy_ARM_Templates_Bicep_And_Exported_Deployments_ARM_Template_Skeleton  
Deploy_ARM_Templates_Bicep_And_Exported_Deployments_Exported_Deployment_Skeleton  
Deploy_ARM_Templates_Bicep_And_Exported_Deployments_WhatIf_And_Validation_Skeleton  
Deploy_ARM_Templates_Bicep_And_Exported_Deployments_Verification_Commands  
Deploy_ARM_Templates_Bicep_And_Exported_Deployments_Rollback  
Deploy_ARM_Templates_Bicep_And_Exported_Deployments_Failure_Checks  
Deploy_ARM_Templates_Bicep_And_Exported_Deployments_Related_Labs  

# Deploy_ARM_Templates_Bicep_And_Exported_Deployments_Source_Basis

| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Azure Resource Manager templates overview | ARM template deployment model, template files, parameter files, resource declarations, and deployment scopes |
| Microsoft Learn | Bicep overview | Bicep as declarative Azure infrastructure language compiled to ARM JSON |
| Microsoft Learn | Export templates from Azure portal | Exporting templates from resource groups and deployment history |
| Microsoft Learn | Export ARM template and convert to Bicep | Exported ARM JSON, Bicep decompile workflow, and cleanup requirements |
| Microsoft Learn | Azure CLI deployment commands | `az deployment group validate`, `what-if`, `create`, `export`, and deployment operation inspection |
| Microsoft Learn | Azure PowerShell deployment commands | `Test-AzResourceGroupDeployment`, `New-AzResourceGroupDeployment`, and export workflows |
| Microsoft Learn | ARM deployment troubleshooting | Deployment operation logs, authorization failures, provider registration, invalid template, invalid reference, quota, and SKU errors |
| Microsoft Learn | Resource provider registration | Provider registration requirements before deployment |
| Microsoft Learn | Azure Policy effects | Deny, audit, modify, append, and remediation impact on deployments |
| Azure operational practice | Infrastructure as Code workflow | Author, parameterize, validate, preview, deploy, verify, export, document, and roll back |
| Azure operational practice | Evidence capture | Save template, parameter file, what-if output, deployment result, operation logs, and final resource state |

# Deploy_ARM_Templates_Bicep_And_Exported_Deployments_Mental_Model

| Concept | Operational Meaning |
|---|---|
| ARM | Azure Resource Manager, the Azure control plane that accepts declarative deployments |
| ARM template | JSON file that describes Azure resources and desired properties |
| Bicep | Higher-level declarative language that compiles into ARM JSON |
| Parameter file | Separate values file used to keep environment-specific inputs out of the main template |
| Resource group deployment | Deployment scoped to one resource group |
| Subscription deployment | Deployment scoped to a subscription, often used to create resource groups, policy assignments, or role assignments |
| Management group deployment | Deployment scoped above subscriptions for governance resources |
| Tenant deployment | Deployment scoped to Microsoft Entra tenant-level resources |
| Deployment name | Control-plane record name for a deployment run |
| Validation | Syntax and control-plane check before deployment |
| What-if | Preview of create, modify, ignore, delete, or no-change effects before deployment |
| Incremental mode | Deploys declared resources and leaves undeclared resources alone |
| Complete mode | Deletes resources not declared in the template at the deployment scope, use carefully |
| Deployment operation | Individual resource operation inside a deployment |
| Exported template | ARM JSON generated from an existing resource group or deployment history |
| Decompile | Convert ARM JSON to Bicep syntax |
| Provider registration | Subscription must have required resource providers registered before using resource types |
| Idempotency | Re-running the same deployment should converge to the same desired state |
| Drift | Actual Azure resource configuration differs from template-defined desired state |
| Rollback | Return to known-good template, parameters, or resource state |
| First rule | Never trust exported templates as clean source code without review |
| Blunt rule | Exported templates are reference material; authored Bicep is the clean long-term IaC source |

# Deploy_ARM_Templates_Bicep_And_Exported_Deployments_Planning_Table

| Item | Example | Decision |
|---|---|---|
| Tenant | `contoso.onmicrosoft.com` | `<tenant-name>` |
| Subscription name | `Azure Lab Subscription` | `<subscription-name>` |
| Subscription ID | `00000000-0000-0000-0000-000000000000` | `<subscription-id>` |
| Deployment scope | `Resource group` | `<tenant / management-group / subscription / resource-group>` |
| Resource group | `rg-compute-lab-01` | `<resource-group-name>` |
| Location | `eastus` | `<azure-region>` |
| Deployment name | `dep-compute-baseline-01` | `<deployment-name>` |
| Template language | `Bicep` | `<Bicep / ARM JSON>` |
| Bicep file | `main.bicep` | `<bicep-file>` |
| ARM template file | `azuredeploy.json` | `<arm-template-file>` |
| Parameter file | `main.parameters.json` | `<parameter-file>` |
| Output folder | `.\out` | `<output-folder>` |
| Evidence folder | `.\evidence\01-arm-bicep-deployments` | `<evidence-folder>` |
| Resource naming prefix | `labcompute` | `<naming-prefix>` |
| Target workload | `Compute baseline` | `<workload-name>` |
| Resource provider list | `Microsoft.Compute`, `Microsoft.Network`, `Microsoft.Storage`, `Microsoft.Web` | `<provider-list>` |
| Validation command | `az deployment group validate` | `<validation-method>` |
| Preview command | `az deployment group what-if` | `<what-if-method>` |
| Deployment command | `az deployment group create` | `<deployment-method>` |
| Export method | `Portal export`, `az group export`, `az deployment group export` | `<export-method>` |
| Deployment mode | `Incremental` | `<Incremental / Complete>` |
| Rollback posture | `Redeploy last known-good template` | `<rollback-plan>` |
| Approval checkpoint | `Review what-if before create` | `<approval-step>` |

# Deploy_ARM_Templates_Bicep_And_Exported_Deployments_Configuration_Checklist

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm Azure CLI login | Admin Workstation | `az account show -o table` | Current account and subscription are visible |
| 2 | Set correct subscription | Admin Workstation | `az account set --subscription "<subscription-id>"` | CLI targets correct subscription |
| 3 | Confirm resource group scope | Admin Workstation | `az group show --name "<resource-group-name>" -o table` | Resource group exists or not found is confirmed |
| 4 | Create resource group if needed | Admin Workstation | `az group create --name "<resource-group-name>" --location "<azure-region>"` | Resource group exists |
| 5 | Confirm provider registration | Admin Workstation | `az provider show --namespace Microsoft.Resources --query registrationState -o tsv` | Provider shows `Registered` |
| 6 | Register required providers | Admin Workstation | `az provider register --namespace Microsoft.Compute` | Provider registration starts |
| 7 | Create project folders | Admin Workstation | `mkdir -p infra/01-arm-bicep out evidence/01-arm-bicep-deployments` | Folder structure exists |
| 8 | Create Bicep file | Admin Workstation | `code infra/01-arm-bicep/main.bicep` | Bicep file is created |
| 9 | Create parameter file | Admin Workstation | `code infra/01-arm-bicep/main.parameters.json` | Parameter file is created |
| 10 | Build Bicep into ARM JSON | Admin Workstation | `az bicep build --file infra/01-arm-bicep/main.bicep --outfile out/main.json` | ARM JSON file is generated |
| 11 | Validate deployment | Admin Workstation | `az deployment group validate --resource-group "<resource-group-name>" --template-file infra/01-arm-bicep/main.bicep --parameters infra/01-arm-bicep/main.parameters.json` | Template validation succeeds |
| 12 | Run what-if preview | Admin Workstation | `az deployment group what-if --resource-group "<resource-group-name>" --template-file infra/01-arm-bicep/main.bicep --parameters infra/01-arm-bicep/main.parameters.json` | Planned changes are displayed |
| 13 | Save what-if evidence | Admin Workstation | `az deployment group what-if --resource-group "<resource-group-name>" --template-file infra/01-arm-bicep/main.bicep --parameters infra/01-arm-bicep/main.parameters.json > evidence/01-arm-bicep-deployments/what-if.txt` | Preview evidence is saved |
| 14 | Deploy Bicep | Admin Workstation | `az deployment group create --name "<deployment-name>" --resource-group "<resource-group-name>" --template-file infra/01-arm-bicep/main.bicep --parameters infra/01-arm-bicep/main.parameters.json` | Deployment succeeds |
| 15 | Save deployment result | Admin Workstation | `az deployment group show --name "<deployment-name>" --resource-group "<resource-group-name>" -o json > evidence/01-arm-bicep-deployments/deployment-result.json` | Deployment result is saved |
| 16 | Review deployment operations | Admin Workstation | `az deployment operation group list --name "<deployment-name>" --resource-group "<resource-group-name>" -o table` | Per-resource operations are visible |
| 17 | Save operation evidence | Admin Workstation | `az deployment operation group list --name "<deployment-name>" --resource-group "<resource-group-name>" -o json > evidence/01-arm-bicep-deployments/deployment-operations.json` | Operation evidence is saved |
| 18 | Verify resources in resource group | Admin Workstation | `az resource list --resource-group "<resource-group-name>" -o table` | Expected resources exist |
| 19 | Export current resource group template | Admin Workstation | `az group export --name "<resource-group-name>" > out/exported-resource-group.json` | Resource group export file is created |
| 20 | Export deployment template | Admin Workstation | `az deployment group export --name "<deployment-name>" --resource-group "<resource-group-name>" > out/exported-deployment.json` | Deployment export file is created |
| 21 | Decompile exported ARM JSON | Admin Workstation | `az bicep decompile --file out/exported-deployment.json --outfile out/exported-deployment.bicep` | Bicep reference file is generated |
| 22 | Review decompiled Bicep | Admin Workstation | `code out/exported-deployment.bicep` | Generated code is inspected |
| 23 | Compare authored and exported code | Admin Workstation | `code --diff infra/01-arm-bicep/main.bicep out/exported-deployment.bicep` | Differences are visible |
| 24 | Clean exported template noise | Admin Workstation | `Remove runtime-only properties, generated names, read-only values, and null defaults` | Exported template becomes usable reference |
| 25 | Run validation after cleanup | Admin Workstation | `az deployment group validate --resource-group "<resource-group-name>" --template-file "<cleaned-template-file>" --parameters "<parameter-file>"` | Cleaned template validates |
| 26 | Check activity log | Admin Workstation | `az monitor activity-log list --resource-group "<resource-group-name>" --max-events 20 -o table` | Deployment events are visible |
| 27 | Record outputs | Admin Workstation | `az deployment group show --resource-group "<resource-group-name>" --name "<deployment-name>" --query properties.outputs -o json` | Template outputs are captured |
| 28 | Commit IaC files | Admin Workstation | `git add infra/01-arm-bicep && git commit -m "Add ARM Bicep deployment workbook lab"` | IaC baseline is versioned |
| 29 | Tag known-good version | Admin Workstation | `git tag lab-01-arm-bicep-known-good` | Known-good state is marked |
| 30 | Document final state | Operator | `Record deployment name, scope, outputs, resource IDs, and rollback plan` | Workbook evidence is complete |

# Deploy_ARM_Templates_Bicep_And_Exported_Deployments_Portal_Skeleton

```text
Purpose:
Deploy and export ARM templates through the Azure portal.

Portal deployment path:
Azure portal
> Search: Deploy a custom template
> Build your own template in the editor
> Load file or paste template
> Save
> Choose subscription
> Choose resource group
> Choose region
> Enter parameters
> Review + create
> Create

Portal export from resource group:
Azure portal
> Resource groups
> <resource-group-name>
> Automation
> Export template
> Download

Portal export from deployment history:
Azure portal
> Resource groups
> <resource-group-name>
> Deployments
> <deployment-name>
> Template
> Download

Portal review checklist:
1. Confirm deployment scope.
2. Confirm subscription.
3. Confirm resource group.
4. Confirm region.
5. Confirm parameters.
6. Confirm no accidental production names.
7. Confirm no secrets are stored in plain text.
8. Confirm validation succeeds.
9. Deploy.
10. Open deployment.
11. Review inputs.
12. Review outputs.
13. Review deployment operations.
14. Save deployment evidence.
15. Export template only as reference material.

Portal cleanup:
1. Remove exported template properties that are read-only.
2. Remove generated runtime values.
3. Parameterize names, locations, SKUs, and tags.
4. Convert exported JSON to Bicep if using Bicep.
5. Validate before reuse.
```

# Deploy_ARM_Templates_Bicep_And_Exported_Deployments_AzureCLI_Skeleton

```bash
# Run from Azure Cloud Shell or local Azure CLI.
# Purpose: validate, preview, deploy, inspect, export, and decompile ARM/Bicep deployments.

# -----------------------------
# Variables
# -----------------------------

export SUBSCRIPTION_ID="<subscription-id>"
export LOCATION="eastus"
export RESOURCE_GROUP="rg-compute-lab-01"
export DEPLOYMENT_NAME="dep-compute-baseline-01"

export TEMPLATE_FILE="infra/01-arm-bicep/main.bicep"
export PARAMETER_FILE="infra/01-arm-bicep/main.parameters.json"

export OUTPUT_FOLDER="out"
export EVIDENCE_FOLDER="evidence/01-arm-bicep-deployments"

mkdir -p "$OUTPUT_FOLDER"
mkdir -p "$EVIDENCE_FOLDER"

# -----------------------------
# Context
# -----------------------------

az login

az account set \
  --subscription "$SUBSCRIPTION_ID"

az account show \
  --query "{user:user.name, tenant:tenantId, subscription:name, subscriptionId:id}" \
  --output table |
  tee "$EVIDENCE_FOLDER/account-context.txt"

# -----------------------------
# Providers
# -----------------------------

for provider in Microsoft.Resources Microsoft.Compute Microsoft.Network Microsoft.Storage Microsoft.Web Microsoft.Insights
do
  az provider register --namespace "$provider"
done

for provider in Microsoft.Resources Microsoft.Compute Microsoft.Network Microsoft.Storage Microsoft.Web Microsoft.Insights
do
  az provider show \
    --namespace "$provider" \
    --query "{namespace:namespace, registrationState:registrationState}" \
    --output table
done | tee "$EVIDENCE_FOLDER/provider-registration.txt"

# -----------------------------
# Resource group
# -----------------------------

az group create \
  --name "$RESOURCE_GROUP" \
  --location "$LOCATION" |
  tee "$EVIDENCE_FOLDER/resource-group-create.json"

# -----------------------------
# Bicep build
# -----------------------------

az bicep version |
  tee "$EVIDENCE_FOLDER/bicep-version.txt"

az bicep build \
  --file "$TEMPLATE_FILE" \
  --outfile "$OUTPUT_FOLDER/main.json"

# -----------------------------
# Validate
# -----------------------------

az deployment group validate \
  --name "$DEPLOYMENT_NAME" \
  --resource-group "$RESOURCE_GROUP" \
  --template-file "$TEMPLATE_FILE" \
  --parameters "$PARAMETER_FILE" |
  tee "$EVIDENCE_FOLDER/validate.json"

# -----------------------------
# What-if
# -----------------------------

az deployment group what-if \
  --name "$DEPLOYMENT_NAME" \
  --resource-group "$RESOURCE_GROUP" \
  --template-file "$TEMPLATE_FILE" \
  --parameters "$PARAMETER_FILE" |
  tee "$EVIDENCE_FOLDER/what-if.txt"

# -----------------------------
# Deploy
# -----------------------------

az deployment group create \
  --name "$DEPLOYMENT_NAME" \
  --resource-group "$RESOURCE_GROUP" \
  --template-file "$TEMPLATE_FILE" \
  --parameters "$PARAMETER_FILE" |
  tee "$EVIDENCE_FOLDER/deployment-create.json"

# -----------------------------
# Inspect deployment
# -----------------------------

az deployment group show \
  --name "$DEPLOYMENT_NAME" \
  --resource-group "$RESOURCE_GROUP" \
  --output json |
  tee "$EVIDENCE_FOLDER/deployment-show.json"

az deployment operation group list \
  --name "$DEPLOYMENT_NAME" \
  --resource-group "$RESOURCE_GROUP" \
  --output json |
  tee "$EVIDENCE_FOLDER/deployment-operations.json"

az deployment group show \
  --name "$DEPLOYMENT_NAME" \
  --resource-group "$RESOURCE_GROUP" \
  --query "properties.outputs" \
  --output json |
  tee "$EVIDENCE_FOLDER/deployment-outputs.json"

# -----------------------------
# Verify resources
# -----------------------------

az resource list \
  --resource-group "$RESOURCE_GROUP" \
  --output table |
  tee "$EVIDENCE_FOLDER/resource-list.txt"

az monitor activity-log list \
  --resource-group "$RESOURCE_GROUP" \
  --max-events 25 \
  --output table |
  tee "$EVIDENCE_FOLDER/activity-log.txt"

# -----------------------------
# Export from resource group
# -----------------------------

az group export \
  --name "$RESOURCE_GROUP" \
  > "$OUTPUT_FOLDER/exported-resource-group.json"

# -----------------------------
# Export from deployment history
# -----------------------------

az deployment group export \
  --name "$DEPLOYMENT_NAME" \
  --resource-group "$RESOURCE_GROUP" \
  > "$OUTPUT_FOLDER/exported-deployment.json"

# -----------------------------
# Decompile exported ARM JSON to Bicep
# -----------------------------
# Treat decompiled Bicep as reference material, not clean source.

az bicep decompile \
  --file "$OUTPUT_FOLDER/exported-deployment.json" \
  --outfile "$OUTPUT_FOLDER/exported-deployment.bicep" \
  --force

# -----------------------------
# Final note
# -----------------------------

echo "Review exported and decompiled files manually before reusing them." |
  tee "$EVIDENCE_FOLDER/export-review-note.txt"
```

# Deploy_ARM_Templates_Bicep_And_Exported_Deployments_PowerShell_Skeleton

```powershell
# Run from PowerShell with Az module installed.
# Purpose: validate, deploy, inspect, and export ARM/Bicep deployments.

# -----------------------------
# Variables
# -----------------------------

$SubscriptionId = "<subscription-id>"
$Location = "eastus"
$ResourceGroupName = "rg-compute-lab-01"
$DeploymentName = "dep-compute-baseline-01"

$TemplateFile = "infra/01-arm-bicep/main.bicep"
$ParameterFile = "infra/01-arm-bicep/main.parameters.json"

$OutputFolder = "out"
$EvidenceFolder = "evidence/01-arm-bicep-deployments"

New-Item -ItemType Directory -Force -Path $OutputFolder
New-Item -ItemType Directory -Force -Path $EvidenceFolder

# -----------------------------
# Context
# -----------------------------

Connect-AzAccount

Set-AzContext -SubscriptionId $SubscriptionId

Get-AzContext |
  Tee-Object (Join-Path $EvidenceFolder "az-context.txt")

# -----------------------------
# Resource group
# -----------------------------

New-AzResourceGroup `
  -Name $ResourceGroupName `
  -Location $Location `
  -Force |
  Tee-Object (Join-Path $EvidenceFolder "resource-group.txt")

# -----------------------------
# Validate
# -----------------------------

Test-AzResourceGroupDeployment `
  -ResourceGroupName $ResourceGroupName `
  -TemplateFile $TemplateFile `
  -TemplateParameterFile $ParameterFile |
  Tee-Object (Join-Path $EvidenceFolder "validate.txt")

# -----------------------------
# Deploy
# -----------------------------

New-AzResourceGroupDeployment `
  -Name $DeploymentName `
  -ResourceGroupName $ResourceGroupName `
  -TemplateFile $TemplateFile `
  -TemplateParameterFile $ParameterFile `
  -Mode Incremental |
  Tee-Object (Join-Path $EvidenceFolder "deployment-create.txt")

# -----------------------------
# Inspect
# -----------------------------

Get-AzResourceGroupDeployment `
  -ResourceGroupName $ResourceGroupName `
  -Name $DeploymentName |
  Tee-Object (Join-Path $EvidenceFolder "deployment-show.txt")

Get-AzResourceGroupDeploymentOperation `
  -ResourceGroupName $ResourceGroupName `
  -DeploymentName $DeploymentName |
  Tee-Object (Join-Path $EvidenceFolder "deployment-operations.txt")

Get-AzResource `
  -ResourceGroupName $ResourceGroupName |
  Tee-Object (Join-Path $EvidenceFolder "resource-list.txt")

# -----------------------------
# Export resource group
# -----------------------------

Export-AzResourceGroup `
  -ResourceGroupName $ResourceGroupName `
  -Path (Join-Path $OutputFolder "exported-resource-group.json") `
  -Force

# -----------------------------
# Bicep build through Azure CLI if desired
# -----------------------------

az bicep build `
  --file $TemplateFile `
  --outfile "$OutputFolder/main.json"

az bicep decompile `
  --file "$OutputFolder/exported-resource-group.json" `
  --outfile "$OutputFolder/exported-resource-group.bicep" `
  --force
```

# Deploy_ARM_Templates_Bicep_And_Exported_Deployments_Bicep_Skeleton

```bicep
// File: infra/01-arm-bicep/main.bicep
// Purpose: baseline resource group deployment proving Bicep parameters, tags, outputs, and repeatable deployment.

targetScope = 'resourceGroup'

@description('Azure region for resources.')
param location string = resourceGroup().location

@description('Environment name.')
@allowed([
  'lab'
  'dev'
  'test'
  'prod'
])
param environment string = 'lab'

@description('Workload name.')
param workloadName string = 'compute'

@description('Storage account name. Must be globally unique, lowercase, and 3 to 24 characters.')
param storageAccountName string

@description('Storage account SKU.')
@allowed([
  'Standard_LRS'
  'Standard_GRS'
  'Standard_ZRS'
])
param storageSku string = 'Standard_LRS'

@description('VNet name.')
param vnetName string = 'vnet-compute-lab-01'

@description('VNet address prefix.')
param vnetAddressPrefix string = '10.20.0.0/16'

@description('Subnet name.')
param subnetName string = 'snet-workload-01'

@description('Subnet prefix.')
param subnetPrefix string = '10.20.1.0/24'

@description('Network security group name.')
param nsgName string = 'nsg-snet-workload-01'

var commonTags = {
  Environment: environment
  Workload: workloadName
  ManagedBy: 'Bicep'
  Workbook: '01_Deploy_ARM_Templates_Bicep_And_Exported_Deployments'
}

resource storage 'Microsoft.Storage/storageAccounts@2023-05-01' = {
  name: storageAccountName
  location: location
  tags: commonTags
  sku: {
    name: storageSku
  }
  kind: 'StorageV2'
  properties: {
    allowBlobPublicAccess: false
    minimumTlsVersion: 'TLS1_2'
    supportsHttpsTrafficOnly: true
  }
}

resource nsg 'Microsoft.Network/networkSecurityGroups@2023-11-01' = {
  name: nsgName
  location: location
  tags: commonTags
  properties: {
    securityRules: []
  }
}

resource vnet 'Microsoft.Network/virtualNetworks@2023-11-01' = {
  name: vnetName
  location: location
  tags: commonTags
  properties: {
    addressSpace: {
      addressPrefixes: [
        vnetAddressPrefix
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

output storageAccountId string = storage.id
output storageAccountName string = storage.name
output vnetId string = vnet.id
output subnetId string = resourceId('Microsoft.Network/virtualNetworks/subnets', vnet.name, subnetName)
output nsgId string = nsg.id
```

# Deploy_ARM_Templates_Bicep_And_Exported_Deployments_Bicep_Parameters_Skeleton

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "location": {
      "value": "eastus"
    },
    "environment": {
      "value": "lab"
    },
    "workloadName": {
      "value": "compute"
    },
    "storageAccountName": {
      "value": "<globally-unique-storage-account-name>"
    },
    "storageSku": {
      "value": "Standard_LRS"
    },
    "vnetName": {
      "value": "vnet-compute-lab-01"
    },
    "vnetAddressPrefix": {
      "value": "10.20.0.0/16"
    },
    "subnetName": {
      "value": "snet-workload-01"
    },
    "subnetPrefix": {
      "value": "10.20.1.0/24"
    },
    "nsgName": {
      "value": "nsg-snet-workload-01"
    }
  }
}
```

# Deploy_ARM_Templates_Bicep_And_Exported_Deployments_ARM_Template_Skeleton

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "location": {
      "type": "string",
      "defaultValue": "[resourceGroup().location]"
    },
    "environment": {
      "type": "string",
      "defaultValue": "lab",
      "allowedValues": [
        "lab",
        "dev",
        "test",
        "prod"
      ]
    },
    "workloadName": {
      "type": "string",
      "defaultValue": "compute"
    },
    "storageAccountName": {
      "type": "string"
    },
    "storageSku": {
      "type": "string",
      "defaultValue": "Standard_LRS",
      "allowedValues": [
        "Standard_LRS",
        "Standard_GRS",
        "Standard_ZRS"
      ]
    },
    "vnetName": {
      "type": "string",
      "defaultValue": "vnet-compute-lab-01"
    },
    "vnetAddressPrefix": {
      "type": "string",
      "defaultValue": "10.20.0.0/16"
    },
    "subnetName": {
      "type": "string",
      "defaultValue": "snet-workload-01"
    },
    "subnetPrefix": {
      "type": "string",
      "defaultValue": "10.20.1.0/24"
    },
    "nsgName": {
      "type": "string",
      "defaultValue": "nsg-snet-workload-01"
    }
  },
  "variables": {
    "tags": {
      "Environment": "[parameters('environment')]",
      "Workload": "[parameters('workloadName')]",
      "ManagedBy": "ARM",
      "Workbook": "01_Deploy_ARM_Templates_Bicep_And_Exported_Deployments"
    }
  },
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2023-05-01",
      "name": "[parameters('storageAccountName')]",
      "location": "[parameters('location')]",
      "tags": "[variables('tags')]",
      "sku": {
        "name": "[parameters('storageSku')]"
      },
      "kind": "StorageV2",
      "properties": {
        "allowBlobPublicAccess": false,
        "minimumTlsVersion": "TLS1_2",
        "supportsHttpsTrafficOnly": true
      }
    },
    {
      "type": "Microsoft.Network/networkSecurityGroups",
      "apiVersion": "2023-11-01",
      "name": "[parameters('nsgName')]",
      "location": "[parameters('location')]",
      "tags": "[variables('tags')]",
      "properties": {
        "securityRules": []
      }
    },
    {
      "type": "Microsoft.Network/virtualNetworks",
      "apiVersion": "2023-11-01",
      "name": "[parameters('vnetName')]",
      "location": "[parameters('location')]",
      "tags": "[variables('tags')]",
      "dependsOn": [
        "[resourceId('Microsoft.Network/networkSecurityGroups', parameters('nsgName'))]"
      ],
      "properties": {
        "addressSpace": {
          "addressPrefixes": [
            "[parameters('vnetAddressPrefix')]"
          ]
        },
        "subnets": [
          {
            "name": "[parameters('subnetName')]",
            "properties": {
              "addressPrefix": "[parameters('subnetPrefix')]",
              "networkSecurityGroup": {
                "id": "[resourceId('Microsoft.Network/networkSecurityGroups', parameters('nsgName'))]"
              }
            }
          }
        ]
      }
    }
  ],
  "outputs": {
    "storageAccountId": {
      "type": "string",
      "value": "[resourceId('Microsoft.Storage/storageAccounts', parameters('storageAccountName'))]"
    },
    "vnetId": {
      "type": "string",
      "value": "[resourceId('Microsoft.Network/virtualNetworks', parameters('vnetName'))]"
    },
    "nsgId": {
      "type": "string",
      "value": "[resourceId('Microsoft.Network/networkSecurityGroups', parameters('nsgName'))]"
    }
  }
}
```

# Deploy_ARM_Templates_Bicep_And_Exported_Deployments_Exported_Deployment_Skeleton

```text
Purpose:
Use exported templates as reference material, not final source code.

Export paths:
1. Resource group export:
   az group export --name <resource-group-name> > out/exported-resource-group.json

2. Deployment history export:
   az deployment group export \
     --name <deployment-name> \
     --resource-group <resource-group-name> \
     > out/exported-deployment.json

3. Portal export:
   Resource group > Export template
   Resource group > Deployments > <deployment-name> > Template

Decompile:
az bicep decompile \
  --file out/exported-deployment.json \
  --outfile out/exported-deployment.bicep

Review exported template:
1. Remove read-only properties.
2. Remove null properties that add noise.
3. Remove generated names where naming should be parameterized.
4. Replace hardcoded region with parameter.
5. Replace hardcoded tags with parameter or variable.
6. Replace embedded IDs with resourceId expressions where possible.
7. Remove secrets and sensitive outputs.
8. Replace one-off values with parameters.
9. Add outputs needed by dependent deployments.
10. Validate before redeploying.

Good use cases:
1. Learn existing resource shape.
2. Recover a rough template from a portal-created resource.
3. Compare actual resource state with authored IaC.
4. Start a migration from click-built resources to Bicep.

Bad use cases:
1. Treat exported template as clean production IaC.
2. Commit secret values.
3. Deploy complete mode blindly.
4. Reuse exported templates without validation.
```

# Deploy_ARM_Templates_Bicep_And_Exported_Deployments_WhatIf_And_Validation_Skeleton

```text
Purpose:
Prevent unsafe deployments before touching resources.

Validation:
az deployment group validate \
  --resource-group <rg> \
  --template-file main.bicep \
  --parameters main.parameters.json

What-if:
az deployment group what-if \
  --resource-group <rg> \
  --template-file main.bicep \
  --parameters main.parameters.json

What-if output categories:
1. Create:
   Resource does not exist and will be created.

2. Modify:
   Resource exists and will be changed.

3. Delete:
   Resource may be removed, especially in complete mode.

4. Ignore:
   Resource exists but is not affected.

5. NoChange:
   Resource matches desired state.

Operator checks before deployment:
1. Confirm subscription.
2. Confirm scope.
3. Confirm resource group.
4. Confirm mode is Incremental unless Complete is intentional.
5. Confirm no unexpected deletes.
6. Confirm names and regions.
7. Confirm SKUs and costs.
8. Confirm tags.
9. Confirm networking address ranges.
10. Confirm no plain-text secrets.
11. Confirm policy denies are understood.
12. Save what-if output.

Approval rule:
If what-if shows unexpected delete, unexpected SKU change, wrong region, or wrong resource group, stop and fix template or parameters before deployment.
```

# Deploy_ARM_Templates_Bicep_And_Exported_Deployments_Verification_Commands

```bash
# Context
az account show -o table

# Resource group
az group show \
  --name "<resource-group-name>" \
  -o table

# Provider state
az provider show \
  --namespace Microsoft.Compute \
  --query "{namespace:namespace, registrationState:registrationState}" \
  -o table

az provider show \
  --namespace Microsoft.Network \
  --query "{namespace:namespace, registrationState:registrationState}" \
  -o table

az provider show \
  --namespace Microsoft.Storage \
  --query "{namespace:namespace, registrationState:registrationState}" \
  -o table

# Bicep version
az bicep version

# Build Bicep
az bicep build \
  --file "infra/01-arm-bicep/main.bicep" \
  --outfile "out/main.json"

# Validate deployment
az deployment group validate \
  --name "<deployment-name>" \
  --resource-group "<resource-group-name>" \
  --template-file "infra/01-arm-bicep/main.bicep" \
  --parameters "infra/01-arm-bicep/main.parameters.json"

# What-if deployment
az deployment group what-if \
  --name "<deployment-name>" \
  --resource-group "<resource-group-name>" \
  --template-file "infra/01-arm-bicep/main.bicep" \
  --parameters "infra/01-arm-bicep/main.parameters.json"

# Show deployment
az deployment group show \
  --resource-group "<resource-group-name>" \
  --name "<deployment-name>" \
  -o json

# Deployment outputs
az deployment group show \
  --resource-group "<resource-group-name>" \
  --name "<deployment-name>" \
  --query "properties.outputs" \
  -o json

# Deployment operations
az deployment operation group list \
  --resource-group "<resource-group-name>" \
  --name "<deployment-name>" \
  -o table

# Failed operations only
az deployment operation group list \
  --resource-group "<resource-group-name>" \
  --name "<deployment-name>" \
  --query "[?properties.provisioningState=='Failed']" \
  -o json

# Resource inventory
az resource list \
  --resource-group "<resource-group-name>" \
  -o table

# Export resource group
az group export \
  --name "<resource-group-name>" \
  > "out/exported-resource-group.json"

# Export deployment
az deployment group export \
  --resource-group "<resource-group-name>" \
  --name "<deployment-name>" \
  > "out/exported-deployment.json"

# Decompile ARM JSON to Bicep
az bicep decompile \
  --file "out/exported-deployment.json" \
  --outfile "out/exported-deployment.bicep" \
  --force

# Activity log
az monitor activity-log list \
  --resource-group "<resource-group-name>" \
  --max-events 25 \
  -o table
```

```powershell
# Context
Get-AzContext

# Resource group
Get-AzResourceGroup -Name "<resource-group-name>"

# Validate
Test-AzResourceGroupDeployment `
  -ResourceGroupName "<resource-group-name>" `
  -TemplateFile "infra/01-arm-bicep/main.bicep" `
  -TemplateParameterFile "infra/01-arm-bicep/main.parameters.json"

# Deploy
New-AzResourceGroupDeployment `
  -Name "<deployment-name>" `
  -ResourceGroupName "<resource-group-name>" `
  -TemplateFile "infra/01-arm-bicep/main.bicep" `
  -TemplateParameterFile "infra/01-arm-bicep/main.parameters.json" `
  -Mode Incremental

# Show deployment
Get-AzResourceGroupDeployment `
  -ResourceGroupName "<resource-group-name>" `
  -Name "<deployment-name>"

# Show operations
Get-AzResourceGroupDeploymentOperation `
  -ResourceGroupName "<resource-group-name>" `
  -DeploymentName "<deployment-name>"

# Export resource group
Export-AzResourceGroup `
  -ResourceGroupName "<resource-group-name>" `
  -Path "out/exported-resource-group.json" `
  -Force
```

# Deploy_ARM_Templates_Bicep_And_Exported_Deployments_Rollback

| Change | Rollback Command / Action | Risk |
|---|---|---|
| Bad deployment changed resource config | Redeploy last known-good Bicep and parameter file | May not undo destructive changes |
| Wrong resource group created | `az group delete --name "<wrong-resource-group-name>" --yes --no-wait` | Deletes all resources inside that RG |
| Wrong resource created | `az resource delete --ids "<resource-id>"` | Deletes selected resource |
| Wrong storage account created | `az storage account delete --name "<storage-account-name>" --resource-group "<resource-group-name>" --yes` | Data loss if used |
| Wrong VNet created | `az network vnet delete --name "<vnet-name>" --resource-group "<resource-group-name>"` | Breaks dependent subnets and NICs |
| Wrong NSG created | `az network nsg delete --name "<nsg-name>" --resource-group "<resource-group-name>"` | Breaks subnet or NIC association if still attached |
| Incorrect tags applied | Redeploy with corrected tags or run `az resource tag` | Cost and ownership reporting may remain wrong until fixed |
| Bad parameter file committed | Revert Git commit or restore known-good parameter file | Future deployments may repeat bad values if not corrected |
| Exported template committed as source | Remove exported file or move to reference folder | Keeps noisy or unsafe generated IaC out of source path |
| Complete mode deleted resources | Restore from backup or redeploy missing resources | May be unrecoverable without backup |
| Provider registered unnecessarily | Leave registered unless policy requires cleanup | Low operational risk |
| Deployment failed halfway | Inspect operations, fix failed dependency, rerun in incremental mode | Partial resources may remain |
| Secret exposed in parameter file | Rotate secret, remove from Git history if required, use Key Vault reference | Exposure may persist in history |
| Wrong SKU deployed | Redeploy with correct SKU if supported | Some SKU changes require recreation |
| Wrong region deployed | Recreate resources in correct region | Many resources cannot move cleanly |

# Deploy_ARM_Templates_Bicep_And_Exported_Deployments_Failure_Checks

| Symptom | Likely Layer | First Check | Fix |
|---|---|---|---|
| `AuthorizationFailed` | RBAC | `az account show`; role assignment at deployment scope | Use correct subscription or assign Contributor or required role |
| `MissingSubscriptionRegistration` | Provider registration | `az provider show --namespace <provider>` | Register provider and retry |
| `InvalidTemplate` | Template syntax or schema | Deployment error details | Fix JSON/Bicep syntax or resource schema |
| Bicep build fails | Bicep compile issue | `az bicep build --file main.bicep` | Fix type, symbol, module, or property error |
| Parameter file rejected | Parameter mismatch | Validate output | Match parameter names and types |
| `InvalidResourceReference` | Dependency or resource ID issue | Deployment operations | Add `dependsOn`, fix symbolic reference, or correct resourceId |
| `ResourceGroupNotFound` | Scope issue | `az group show -n <rg>` | Create RG or target correct scope |
| `LocationNotAvailableForResourceType` | Region support | Error details and SKU location | Choose supported region |
| `SkuNotAvailable` | SKU or region | Deployment error | Pick supported SKU or region |
| `QuotaExceeded` | Subscription quota | Quota blade or deployment details | Request quota or reduce deployment |
| Policy deny | Azure Policy | Deployment operation status message | Change template or request exemption |
| What-if shows delete | Deployment mode or scope | What-if output | Use incremental mode or correct template |
| Deployment succeeds but resource missing | Conditional deployment or wrong scope | Resource list and outputs | Fix condition, scope, or parameter |
| Exported template fails redeploy | Read-only exported properties | Validation errors | Clean exported template |
| Decompile produces ugly Bicep | Exported JSON not authored source | Decompiled output | Refactor into clean Bicep |
| Secret appears in exported file | Export captured sensitive value | Template and parameters | Remove, rotate, and use secure parameters or Key Vault |
| Deployment operation list is empty | Wrong deployment name or scope | Deployment list | Query correct scope and deployment name |
| Template works in portal but not CLI | Parameter quoting or file path | CLI command and shell | Fix parameter syntax and path |
| PowerShell deploy fails but CLI works | Module version or parameter syntax | `Get-Module Az` | Update Az module or adjust command |
| Resource already exists | Name collision or global uniqueness | Error detail | Use unique name or import/reference existing resource |
| `DeploymentActiveAndUneditable` | Concurrent deployment | Deployment list | Wait or use unique deployment name |
| Tags overwritten unexpectedly | Template tag model | Resource tags after deployment | Merge tags intentionally or update template |
| Complete mode unsupported at scope | Deployment mode issue | Error detail | Use incremental mode |
| Private endpoint or DNS resource fails | Missing provider or dependency | Provider state and operations | Register provider and correct dependency order |
| Rollback does not remove created resource | Incremental redeploy only modifies desired resources | Resource inventory | Delete unwanted resources explicitly |

# Deploy_ARM_Templates_Bicep_And_Exported_Deployments_Related_Labs

| Lab | Related Workbook | Skill Proven |
|---|---|---|
| Deploy ARM and Bicep baseline | `01_Deploy_ARM_Templates_Bicep_And_Exported_Deployments.md` | IaC authoring, validation, what-if, deployment, export, and cleanup |
| Create and manage Azure VMs | `02_Create_Configure_And_Manage_Azure_Virtual_Machines.md` | VM deployment using validated IaC patterns |
| Configure VM disks, sizes, availability, extensions, and encryption | `03_Configure_VM_Disks_Sizes_Availability_Sets_Zones_Extensions_And_Encryption.md` | Declarative and operational VM platform configuration |
| Configure VMSS images and scaling | `04_Configure_VMSS_Images_Scaling_And_Automation.md` | Repeatable scale set deployment and model updates |
| Deploy ACR, ACI, and Azure Container Apps | `05_Deploy_ACR_ACI_Azure_Container_Apps_Sizing_And_Scaling.md` | Container registry and app platform deployment |
| Deploy App Service plans and web apps | `06_Deploy_App_Service_Plans_Web_Apps_TLS_Custom_Domains_Backup_Networking_And_Slots.md` | PaaS web deployment using templates and CLI |
| Configure compute managed identities and Key Vault references | `07_Configure_Compute_Managed_Identities_Key_Vault_Access_And_Secrets_References.md` | Secretless IaC integration |
| Troubleshoot compute deployment and runtime issues | `08_Troubleshoot_Azure_VM_Container_App_Service_Deployment_And_Runtime_Issues.md` | Deployment operation diagnostics and failure triage |