# 06_Deploy_App_Service_Plans_Web_Apps_TLS_Custom_Domains_Backup_Networking_And_Slots

# 06_Deploy_App_Service_Plans_Web_Apps_TLS_Custom_Domains_Backup_Networking_And_Slots_Index

06_Deploy_App_Service_Plans_Web_Apps_TLS_Custom_Domains_Backup_Networking_And_Slots.md  
Deploy_App_Service_Plans_Web_Apps_TLS_Custom_Domains_Backup_Networking_And_Slots  
Deploy_App_Service_Plans_Web_Apps_TLS_Custom_Domains_Backup_Networking_And_Slots_Source_Basis  
Deploy_App_Service_Plans_Web_Apps_TLS_Custom_Domains_Backup_Networking_And_Slots_Mental_Model  
Deploy_App_Service_Plans_Web_Apps_TLS_Custom_Domains_Backup_Networking_And_Slots_Planning_Table  
Deploy_App_Service_Plans_Web_Apps_TLS_Custom_Domains_Backup_Networking_And_Slots_Configuration_Checklist  
Deploy_App_Service_Plans_Web_Apps_TLS_Custom_Domains_Backup_Networking_And_Slots_Portal_Skeleton  
Deploy_App_Service_Plans_Web_Apps_TLS_Custom_Domains_Backup_Networking_And_Slots_AzureCLI_Skeleton  
Deploy_App_Service_Plans_Web_Apps_TLS_Custom_Domains_Backup_Networking_And_Slots_PowerShell_Skeleton  
Deploy_App_Service_Plans_Web_Apps_TLS_Custom_Domains_Backup_Networking_And_Slots_Bicep_Skeleton  
Deploy_App_Service_Plans_Web_Apps_TLS_Custom_Domains_Backup_Networking_And_Slots_App_Service_Plan_Skeleton  
Deploy_App_Service_Plans_Web_Apps_TLS_Custom_Domains_Backup_Networking_And_Slots_Custom_Domain_TLS_Skeleton  
Deploy_App_Service_Plans_Web_Apps_TLS_Custom_Domains_Backup_Networking_And_Slots_Backup_Skeleton  
Deploy_App_Service_Plans_Web_Apps_TLS_Custom_Domains_Backup_Networking_And_Slots_Networking_Skeleton  
Deploy_App_Service_Plans_Web_Apps_TLS_Custom_Domains_Backup_Networking_And_Slots_Slots_Skeleton  
Deploy_App_Service_Plans_Web_Apps_TLS_Custom_Domains_Backup_Networking_And_Slots_Verification_Commands  
Deploy_App_Service_Plans_Web_Apps_TLS_Custom_Domains_Backup_Networking_And_Slots_Rollback  
Deploy_App_Service_Plans_Web_Apps_TLS_Custom_Domains_Backup_Networking_And_Slots_Failure_Checks  
Deploy_App_Service_Plans_Web_Apps_TLS_Custom_Domains_Backup_Networking_And_Slots_Related_Labs  

# Deploy_App_Service_Plans_Web_Apps_TLS_Custom_Domains_Backup_Networking_And_Slots_Source_Basis

| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Azure App Service overview | PaaS web app hosting, runtime stacks, deployment options, scaling, and management |
| Microsoft Learn | App Service plans | Pricing tier, compute SKU, worker size, scale-up, scale-out, and app-to-plan relationship |
| Microsoft Learn | Create web apps with Azure CLI, PowerShell, portal, Bicep, and ARM | Repeatable App Service deployment workflows |
| Microsoft Learn | App Service configuration | App settings, connection strings, runtime stack, platform settings, and deployment settings |
| Microsoft Learn | App Service custom domains | Domain verification, CNAME, A record, TXT records, and hostname binding |
| Microsoft Learn | App Service TLS and certificates | Managed certificates, uploaded certificates, certificate bindings, and HTTPS-only |
| Microsoft Learn | App Service deployment slots | Staging slots, swap, sticky settings, warm-up, rollback, and production-safe release workflow |
| Microsoft Learn | App Service backup and restore | Storage-backed backup configuration, on-demand backup, scheduled backup, and restore |
| Microsoft Learn | App Service networking | Access restrictions, VNet integration, private endpoints, service endpoints, and outbound routing |
| Microsoft Learn | App Service diagnostics and logs | Application logs, web server logs, log streaming, metrics, alerts, and troubleshooting |
| Microsoft Learn | Key Vault references for App Service | Secret references in app settings and managed identity dependency |
| Azure operational practice | PaaS app release workflow | Build, deploy to staging, test, swap, monitor, rollback |
| Azure operational practice | Secure web app baseline | HTTPS-only, minimum TLS, no broad SCM access, least privilege identity, private dependencies |
| Azure operational practice | Evidence capture | Plan SKU, runtime, hostnames, TLS binding, backup config, network path, slots, logs, and revision evidence |

# Deploy_App_Service_Plans_Web_Apps_TLS_Custom_Domains_Backup_Networking_And_Slots_Mental_Model

| Concept | Operational Meaning |
|---|---|
| App Service | Azure PaaS platform for hosting web apps, APIs, and background web workloads |
| Web App | App Service app resource that runs code or a custom container |
| App Service Plan | Compute container that defines region, SKU, worker size, and scale capacity for one or more apps |
| Runtime stack | Language/runtime hosted by App Service, such as .NET, Node, Python, Java, PHP, or custom container |
| SKU | Pricing and capability tier, such as Free, Basic, Standard, Premium v3, or Isolated |
| Scale up | Move App Service Plan to a larger SKU or worker size |
| Scale out | Increase worker instance count |
| App settings | Environment variables injected into the app |
| Connection strings | App-specific connection settings, often used by frameworks |
| Custom domain | User-owned DNS name bound to a web app |
| Domain verification | TXT or CNAME proof that the operator controls the hostname |
| TLS binding | Certificate binding that protects a hostname with HTTPS |
| Managed certificate | Free App Service managed certificate for supported custom domains |
| Uploaded certificate | PFX certificate uploaded by operator |
| HTTPS-only | App setting that rejects plain HTTP |
| Minimum TLS version | Lowest TLS protocol version accepted by app |
| Deployment slot | Separate live app instance tied to the same app, commonly `staging` |
| Slot swap | Promotes one slot into another, usually staging into production |
| Sticky setting | App setting or connection string that stays with a slot during swap |
| Backup | App Service backup of app content and optionally database configuration into storage |
| VNet integration | Outbound private network integration from the app to resources in a VNet |
| Private endpoint | Inbound private access path to the app over a private IP |
| Access restriction | Allow or deny inbound access by IP, service tag, or VNet source |
| SCM site | Kudu/deployment endpoint for the app |
| First rule | App Service networking has separate inbound and outbound controls |
| Blunt rule | A web app is not production-ready just because it returns a page; prove TLS, domain, backup, slot release, logging, and network paths |

# Deploy_App_Service_Plans_Web_Apps_TLS_Custom_Domains_Backup_Networking_And_Slots_Planning_Table

| Item | Example | Decision |
|---|---|---|
| Tenant | `contoso.onmicrosoft.com` | `<tenant-name>` |
| Subscription ID | `00000000-0000-0000-0000-000000000000` | `<subscription-id>` |
| Resource group | `rg-compute-lab-01` | `<resource-group-name>` |
| Location | `eastus` | `<azure-region>` |
| App Service Plan | `asp-web-lab-01` | `<app-service-plan-name>` |
| Plan OS | `Linux` | `<Linux / Windows>` |
| Plan SKU | `B1` | `<sku>` |
| Worker count | `1` | `<worker-count>` |
| Web app name | `app-web-lab-01` | `<web-app-name>` |
| Default hostname | `app-web-lab-01.azurewebsites.net` | `<default-hostname>` |
| Runtime stack | `PYTHON:3.11` | `<runtime-stack>` |
| App type | `Code` | `<Code / Container>` |
| Container image | `acrcomputelab01.azurecr.io/webapp:v1` | `<container-image>` |
| Custom domain | `app.contoso.com` | `<custom-domain>` |
| Domain verification TXT | `asuid.app` | `<verification-record>` |
| CNAME target | `<web-app-name>.azurewebsites.net` | `<cname-target>` |
| Certificate type | `App Service Managed Certificate` | `<managed / uploaded / key-vault>` |
| Minimum TLS | `1.2` | `<minimum-tls-version>` |
| HTTPS-only | `Enabled` | `<enabled-disabled>` |
| Backup storage account | `stapplabbackup01` | `<storage-account-name>` |
| Backup container | `appservice-backups` | `<backup-container>` |
| Backup schedule | `Daily` | `<backup-schedule>` |
| VNet name | `vnet-compute-lab-01` | `<vnet-name>` |
| Integration subnet | `snet-appsvc-integration-01` | `<integration-subnet-name>` |
| Private endpoint subnet | `snet-private-endpoints-01` | `<private-endpoint-subnet-name>` |
| Private endpoint name | `pe-app-web-lab-01` | `<private-endpoint-name>` |
| Private DNS zone | `privatelink.azurewebsites.net` | `<private-dns-zone>` |
| Access restriction posture | `Allow approved source only` | `<access-restriction-design>` |
| Staging slot | `staging` | `<slot-name>` |
| Slot swap method | `Preview then swap` | `<direct / preview>` |
| Sticky settings | `ASPNETCORE_ENVIRONMENT`, `DATABASE_URL` | `<sticky-settings>` |
| Managed identity | `System-assigned` | `<system-assigned / user-assigned / none>` |
| Evidence path | `.\evidence\06-appservice-webapps` | `<evidence-path>` |
| Rollback posture | `Swap back or delete lab RG` | `<rollback-plan>` |

# Deploy_App_Service_Plans_Web_Apps_TLS_Custom_Domains_Backup_Networking_And_Slots_Configuration_Checklist

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm Azure context | Admin Workstation | `az account show -o table` | Correct tenant and subscription are active |
| 2 | Set subscription | Admin Workstation | `az account set --subscription "<subscription-id>"` | CLI targets correct subscription |
| 3 | Register required providers | Admin Workstation | `az provider register --namespace Microsoft.Web && az provider register --namespace Microsoft.Storage && az provider register --namespace Microsoft.Network && az provider register --namespace Microsoft.Insights` | Required providers register |
| 4 | Confirm provider state | Admin Workstation | `az provider show --namespace Microsoft.Web --query registrationState -o tsv` | Provider shows `Registered` |
| 5 | Create resource group | Admin Workstation | `az group create --name "<resource-group-name>" --location "<azure-region>"` | Resource group exists |
| 6 | Create App Service Plan | Admin Workstation | `az appservice plan create --name "<app-service-plan-name>" --resource-group "<resource-group-name>" --location "<azure-region>" --sku "<sku>" --is-linux` | Linux App Service Plan exists |
| 7 | Verify plan | Admin Workstation | `az appservice plan show --name "<app-service-plan-name>" --resource-group "<resource-group-name>" -o table` | Plan SKU and region are visible |
| 8 | Create code-based web app | Admin Workstation | `az webapp create --name "<web-app-name>" --resource-group "<resource-group-name>" --plan "<app-service-plan-name>" --runtime "<runtime-stack>"` | Web app exists |
| 9 | Create container-based web app if needed | Admin Workstation | `az webapp create --name "<web-app-name>" --resource-group "<resource-group-name>" --plan "<app-service-plan-name>" --deployment-container-image-name "<container-image>"` | Container web app exists |
| 10 | Confirm default hostname | Admin Workstation | `az webapp show --name "<web-app-name>" --resource-group "<resource-group-name>" --query defaultHostName -o tsv` | Azure websites hostname is returned |
| 11 | Configure app settings | Admin Workstation | `az webapp config appsettings set --name "<web-app-name>" --resource-group "<resource-group-name>" --settings ENVIRONMENT=lab APPROLE=web` | App settings are applied |
| 12 | Configure HTTPS-only | Admin Workstation | `az webapp update --name "<web-app-name>" --resource-group "<resource-group-name>" --https-only true` | HTTP is redirected or blocked |
| 13 | Configure minimum TLS version | Admin Workstation | `az webapp config set --name "<web-app-name>" --resource-group "<resource-group-name>" --min-tls-version 1.2` | Minimum TLS is set |
| 14 | Enable application logging | Admin Workstation | `az webapp log config --name "<web-app-name>" --resource-group "<resource-group-name>" --application-logging filesystem --level information` | App logging is enabled |
| 15 | Tail logs | Admin Workstation | `az webapp log tail --name "<web-app-name>" --resource-group "<resource-group-name>"` | Logs stream from app |
| 16 | Add custom domain DNS verification record | DNS Provider | `Create TXT asuid.<host> with value from App Service custom domain verification` | Domain ownership can be verified |
| 17 | Add CNAME for subdomain | DNS Provider | `Create CNAME <host> -> <web-app-name>.azurewebsites.net` | Hostname points to App Service |
| 18 | Bind custom hostname | Admin Workstation | `az webapp config hostname add --webapp-name "<web-app-name>" --resource-group "<resource-group-name>" --hostname "<custom-domain>"` | Custom domain is bound |
| 19 | Create App Service managed certificate | Admin Workstation | `az webapp config ssl create --resource-group "<resource-group-name>" --name "<web-app-name>" --hostname "<custom-domain>"` | Managed certificate is created |
| 20 | Bind certificate to hostname | Admin Workstation | `az webapp config ssl bind --resource-group "<resource-group-name>" --name "<web-app-name>" --certificate-thumbprint "<thumbprint>" --ssl-type SNI` | Hostname is protected by TLS |
| 21 | Create backup storage account | Admin Workstation | `az storage account create --name "<storage-account-name>" --resource-group "<resource-group-name>" --location "<azure-region>" --sku Standard_LRS --kind StorageV2` | Backup storage exists |
| 22 | Create backup container | Admin Workstation | `az storage container create --name "<backup-container>" --account-name "<storage-account-name>"` | Backup container exists |
| 23 | Generate backup SAS URL | Admin Workstation | `Generate container SAS with write/list/create permissions and expiration` | Backup URL is available |
| 24 | Create on-demand backup | Admin Workstation | `az webapp config backup create --resource-group "<resource-group-name>" --webapp-name "<web-app-name>" --backup-name "<backup-name>" --container-url "<container-sas-url>"` | Backup job starts |
| 25 | Configure VNet integration subnet | Admin Workstation | `az network vnet subnet create --resource-group "<resource-group-name>" --vnet-name "<vnet-name>" --name "<integration-subnet-name>" --address-prefixes "<integration-subnet-prefix>"` | Integration subnet exists |
| 26 | Add VNet integration | Admin Workstation | `az webapp vnet-integration add --name "<web-app-name>" --resource-group "<resource-group-name>" --vnet "<vnet-name>" --subnet "<integration-subnet-name>"` | App has outbound VNet integration |
| 27 | Configure access restriction | Admin Workstation | `az webapp config access-restriction add --resource-group "<resource-group-name>" --name "<web-app-name>" --rule-name "Allow-Admin-IP" --action Allow --ip-address "<approved-public-ip-cidr>" --priority 100` | Approved source is allowed |
| 28 | Add deny-all rule after allow rules | Admin Workstation | `az webapp config access-restriction add --resource-group "<resource-group-name>" --name "<web-app-name>" --rule-name "Deny-All" --action Deny --ip-address "0.0.0.0/0" --priority 65000` | Unapproved sources are blocked |
| 29 | Create staging slot | Admin Workstation | `az webapp deployment slot create --name "<web-app-name>" --resource-group "<resource-group-name>" --slot staging` | Staging slot exists |
| 30 | Configure slot app settings | Admin Workstation | `az webapp config appsettings set --name "<web-app-name>" --resource-group "<resource-group-name>" --slot staging --settings ENVIRONMENT=staging` | Staging settings are applied |
| 31 | Mark sticky settings | Admin Workstation | `az webapp config appsettings set --name "<web-app-name>" --resource-group "<resource-group-name>" --slot-settings ENVIRONMENT=staging` | Setting sticks to slot |
| 32 | Deploy code or image to staging | Admin Workstation | `az webapp deployment source config-zip --name "<web-app-name>" --resource-group "<resource-group-name>" --slot staging --src "<package.zip>"` | Staging slot has new build |
| 33 | Test staging slot | Admin Workstation | `curl https://<web-app-name>-staging.azurewebsites.net` | Staging app responds |
| 34 | Swap staging to production | Admin Workstation | `az webapp deployment slot swap --name "<web-app-name>" --resource-group "<resource-group-name>" --slot staging --target-slot production` | Staging becomes production |
| 35 | Verify production endpoint | Admin Workstation | `curl https://<custom-domain>` | Production endpoint returns expected app |
| 36 | Capture final plan evidence | Admin Workstation | `az appservice plan show --name "<app-service-plan-name>" --resource-group "<resource-group-name>" -o json > .\evidence\plan-final.json` | Plan evidence is saved |
| 37 | Capture final web app evidence | Admin Workstation | `az webapp show --name "<web-app-name>" --resource-group "<resource-group-name>" -o json > .\evidence\webapp-final.json` | App evidence is saved |
| 38 | Capture final slot evidence | Admin Workstation | `az webapp deployment slot list --name "<web-app-name>" --resource-group "<resource-group-name>" -o json > .\evidence\slots-final.json` | Slot evidence is saved |
| 39 | Document final state | Operator | `Record plan SKU, hostname, TLS, backup, network controls, slots, and rollback` | Workbook evidence is complete |

# Deploy_App_Service_Plans_Web_Apps_TLS_Custom_Domains_Backup_Networking_And_Slots_Portal_Skeleton

```text
Purpose:
Deploy and configure App Service Plan, Web App, TLS, custom domains, backups, networking, and deployment slots through Azure portal.

App Service Plan path:
Azure portal
> App Service plans
> Create

Plan basics:
1. Subscription: <subscription-name>
2. Resource group: <resource-group-name>
3. Name: <app-service-plan-name>
4. Operating system:
   - Linux for Linux runtime or Linux containers
   - Windows for Windows runtime or Windows containers
5. Region: <azure-region>
6. Pricing plan:
   - Free or Shared only for basic tests
   - Basic for simple dev/test
   - Standard for slots and common production-like labs
   - Premium v3 for stronger production-style features
7. Zone redundancy:
   - Enable only if SKU and region support it and workload requires it
8. Review and create.

Web App path:
Azure portal
> App Services
> Create
> Web App

Web app basics:
1. Subscription: <subscription-name>
2. Resource group: <resource-group-name>
3. Name: <web-app-name>
4. Publish:
   - Code
   - Docker Container
5. Runtime stack: <runtime-stack>
6. Operating system: match plan OS
7. Region: <azure-region>
8. App Service Plan: <app-service-plan-name>

Web app configuration:
1. App Service > <web-app-name>
2. Configuration > Application settings
3. Add settings:
   - ENVIRONMENT=lab
   - APPROLE=web
4. Save and restart when prompted.
5. TLS/SSL settings:
   - HTTPS Only: On
   - Minimum TLS Version: 1.2 or higher

Custom domain:
1. App Service > Custom domains.
2. Add custom domain.
3. Enter <custom-domain>.
4. Create required DNS records:
   - CNAME for subdomain
   - TXT record for verification
5. Validate domain.
6. Add hostname binding.

TLS:
1. App Service > Certificates.
2. Create App Service Managed Certificate if eligible.
3. Go to TLS/SSL settings.
4. Add TLS binding.
5. Choose custom domain.
6. Choose certificate.
7. Select SNI SSL.
8. Confirm HTTPS works.

Backup:
1. App Service > Backups.
2. Configure custom backup.
3. Select storage account and container.
4. Create on-demand backup.
5. Configure schedule if required.
6. Confirm backup completes.

Networking:
1. App Service > Networking.
2. Access restrictions:
   - Add approved allow rules.
   - Add deny-all rule after allow rules if locking down public access.
3. VNet integration:
   - Select VNet and delegated integration subnet.
   - Confirm outbound private access path.
4. Private endpoint:
   - Create private endpoint if inbound private access is required.
   - Configure private DNS zone: privatelink.azurewebsites.net.
5. Validate public and private endpoint behavior.

Deployment slots:
1. App Service > Deployment slots.
2. Add slot:
   - Name: staging
   - Clone settings from production if appropriate.
3. Configure staging app settings.
4. Mark environment-specific settings as deployment slot settings.
5. Deploy app to staging.
6. Test staging URL.
7. Swap staging into production.
8. Validate production.
9. Swap back if rollback is needed.
```

# Deploy_App_Service_Plans_Web_Apps_TLS_Custom_Domains_Backup_Networking_And_Slots_AzureCLI_Skeleton

```bash
# Run from Azure Cloud Shell or local Azure CLI.
# Purpose: create App Service Plan, Web App, TLS/domain baseline, backup, networking, and slots.

# -----------------------------
# Variables
# -----------------------------

export SUBSCRIPTION_ID="<subscription-id>"
export LOCATION="eastus"
export RESOURCE_GROUP="rg-compute-lab-01"

export PLAN_NAME="asp-web-lab-01"
export PLAN_SKU="B1"

export WEBAPP_NAME="app-web-lab-01"
export RUNTIME_STACK="PYTHON:3.11"

export CUSTOM_DOMAIN="app.contoso.com"
export CERT_THUMBPRINT="<certificate-thumbprint>"

export STORAGE_ACCOUNT="stapplabbackup01"
export BACKUP_CONTAINER="appservice-backups"
export BACKUP_NAME="backup-app-web-lab-01"

export VNET_NAME="vnet-compute-lab-01"
export VNET_PREFIX="10.20.0.0/16"
export INTEGRATION_SUBNET="snet-appsvc-integration-01"
export INTEGRATION_SUBNET_PREFIX="10.20.3.0/27"

export PRIVATE_ENDPOINT_SUBNET="snet-private-endpoints-01"
export PRIVATE_ENDPOINT_SUBNET_PREFIX="10.20.4.0/24"
export PRIVATE_ENDPOINT_NAME="pe-app-web-lab-01"
export PRIVATE_DNS_ZONE="privatelink.azurewebsites.net"

export APPROVED_PUBLIC_IP_CIDR="<approved-public-ip-cidr>"

export SLOT_NAME="staging"
export PACKAGE_ZIP="./package.zip"

export EVIDENCE_PATH="./evidence/06-appservice-webapps"

mkdir -p "$EVIDENCE_PATH"

# -----------------------------
# Login and context
# -----------------------------

az login

az account set \
  --subscription "$SUBSCRIPTION_ID"

az account show \
  --output table |
  tee "$EVIDENCE_PATH/account-context.txt"

# -----------------------------
# Providers
# -----------------------------

for provider in Microsoft.Web Microsoft.Storage Microsoft.Network Microsoft.Insights Microsoft.Resources
do
  az provider register --namespace "$provider"
done

for provider in Microsoft.Web Microsoft.Storage Microsoft.Network Microsoft.Insights Microsoft.Resources
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
# App Service Plan
# -----------------------------

az appservice plan create \
  --name "$PLAN_NAME" \
  --resource-group "$RESOURCE_GROUP" \
  --location "$LOCATION" \
  --sku "$PLAN_SKU" \
  --is-linux |
  tee "$EVIDENCE_PATH/plan-create.json"

az appservice plan show \
  --name "$PLAN_NAME" \
  --resource-group "$RESOURCE_GROUP" \
  --output json |
  tee "$EVIDENCE_PATH/plan-show.json"

# -----------------------------
# Web App, code-based Linux runtime
# -----------------------------

az webapp create \
  --name "$WEBAPP_NAME" \
  --resource-group "$RESOURCE_GROUP" \
  --plan "$PLAN_NAME" \
  --runtime "$RUNTIME_STACK" |
  tee "$EVIDENCE_PATH/webapp-create.json"

az webapp show \
  --name "$WEBAPP_NAME" \
  --resource-group "$RESOURCE_GROUP" \
  --query "{name:name,defaultHostName:defaultHostName,state:state,httpsOnly:httpsOnly}" \
  --output table |
  tee "$EVIDENCE_PATH/webapp-show-basic.txt"

# -----------------------------
# App settings and platform config
# -----------------------------

az webapp config appsettings set \
  --name "$WEBAPP_NAME" \
  --resource-group "$RESOURCE_GROUP" \
  --settings ENVIRONMENT=lab APPROLE=web |
  tee "$EVIDENCE_PATH/appsettings-set.json"

az webapp update \
  --name "$WEBAPP_NAME" \
  --resource-group "$RESOURCE_GROUP" \
  --https-only true |
  tee "$EVIDENCE_PATH/https-only.json"

az webapp config set \
  --name "$WEBAPP_NAME" \
  --resource-group "$RESOURCE_GROUP" \
  --min-tls-version 1.2 \
  --ftps-state Disabled |
  tee "$EVIDENCE_PATH/webapp-config-security.json"

# -----------------------------
# Logging
# -----------------------------

az webapp log config \
  --name "$WEBAPP_NAME" \
  --resource-group "$RESOURCE_GROUP" \
  --application-logging filesystem \
  --level information \
  --web-server-logging filesystem |
  tee "$EVIDENCE_PATH/log-config.json"

# -----------------------------
# Custom domain
# DNS records must exist before hostname binding.
# For subdomain:
# CNAME: app.contoso.com -> app-web-lab-01.azurewebsites.net
# TXT: asuid.app.contoso.com -> custom domain verification ID
# -----------------------------

az webapp show \
  --name "$WEBAPP_NAME" \
  --resource-group "$RESOURCE_GROUP" \
  --query "customDomainVerificationId" \
  --output tsv |
  tee "$EVIDENCE_PATH/custom-domain-verification-id.txt"

# Run after DNS records are created.
az webapp config hostname add \
  --webapp-name "$WEBAPP_NAME" \
  --resource-group "$RESOURCE_GROUP" \
  --hostname "$CUSTOM_DOMAIN" |
  tee "$EVIDENCE_PATH/custom-hostname-add.json"

az webapp config hostname list \
  --webapp-name "$WEBAPP_NAME" \
  --resource-group "$RESOURCE_GROUP" \
  --output table |
  tee "$EVIDENCE_PATH/custom-hostnames.txt"

# -----------------------------
# Managed certificate and TLS binding
# -----------------------------
# Managed certificate eligibility depends on hostname type and DNS validation.
# After certificate creation, capture thumbprint and bind.

az webapp config ssl create \
  --resource-group "$RESOURCE_GROUP" \
  --name "$WEBAPP_NAME" \
  --hostname "$CUSTOM_DOMAIN" |
  tee "$EVIDENCE_PATH/managed-cert-create.json"

CERT_THUMBPRINT=$(az webapp config ssl list \
  --resource-group "$RESOURCE_GROUP" \
  --query "[?hostNames[?@=='$CUSTOM_DOMAIN']].thumbprint | [0]" \
  --output tsv)

echo "$CERT_THUMBPRINT" | tee "$EVIDENCE_PATH/cert-thumbprint.txt"

az webapp config ssl bind \
  --resource-group "$RESOURCE_GROUP" \
  --name "$WEBAPP_NAME" \
  --certificate-thumbprint "$CERT_THUMBPRINT" \
  --ssl-type SNI |
  tee "$EVIDENCE_PATH/ssl-bind.json"

# -----------------------------
# Backup storage
# -----------------------------

az storage account create \
  --name "$STORAGE_ACCOUNT" \
  --resource-group "$RESOURCE_GROUP" \
  --location "$LOCATION" \
  --sku Standard_LRS \
  --kind StorageV2 |
  tee "$EVIDENCE_PATH/backup-storage-create.json"

az storage container create \
  --name "$BACKUP_CONTAINER" \
  --account-name "$STORAGE_ACCOUNT" |
  tee "$EVIDENCE_PATH/backup-container-create.json"

STORAGE_KEY=$(az storage account keys list \
  --account-name "$STORAGE_ACCOUNT" \
  --resource-group "$RESOURCE_GROUP" \
  --query "[0].value" \
  --output tsv)

SAS_TOKEN=$(az storage container generate-sas \
  --account-name "$STORAGE_ACCOUNT" \
  --account-key "$STORAGE_KEY" \
  --name "$BACKUP_CONTAINER" \
  --permissions acdlrw \
  --expiry "2030-01-01T00:00:00Z" \
  --output tsv)

CONTAINER_SAS_URL="https://$STORAGE_ACCOUNT.blob.core.windows.net/$BACKUP_CONTAINER?$SAS_TOKEN"

echo "$CONTAINER_SAS_URL" > "$EVIDENCE_PATH/backup-container-sas-url.txt"

az webapp config backup create \
  --resource-group "$RESOURCE_GROUP" \
  --webapp-name "$WEBAPP_NAME" \
  --backup-name "$BACKUP_NAME" \
  --container-url "$CONTAINER_SAS_URL" |
  tee "$EVIDENCE_PATH/webapp-backup-create.json"

az webapp config backup list \
  --resource-group "$RESOURCE_GROUP" \
  --webapp-name "$WEBAPP_NAME" \
  --output table |
  tee "$EVIDENCE_PATH/webapp-backup-list.txt"

# -----------------------------
# Network baseline, VNet and integration subnet
# -----------------------------

az network vnet create \
  --resource-group "$RESOURCE_GROUP" \
  --location "$LOCATION" \
  --name "$VNET_NAME" \
  --address-prefixes "$VNET_PREFIX" |
  tee "$EVIDENCE_PATH/vnet-create.json"

az network vnet subnet create \
  --resource-group "$RESOURCE_GROUP" \
  --vnet-name "$VNET_NAME" \
  --name "$INTEGRATION_SUBNET" \
  --address-prefixes "$INTEGRATION_SUBNET_PREFIX" |
  tee "$EVIDENCE_PATH/integration-subnet-create.json"

az webapp vnet-integration add \
  --name "$WEBAPP_NAME" \
  --resource-group "$RESOURCE_GROUP" \
  --vnet "$VNET_NAME" \
  --subnet "$INTEGRATION_SUBNET" |
  tee "$EVIDENCE_PATH/vnet-integration-add.json"

az webapp vnet-integration list \
  --name "$WEBAPP_NAME" \
  --resource-group "$RESOURCE_GROUP" \
  --output table |
  tee "$EVIDENCE_PATH/vnet-integration-list.txt"

# -----------------------------
# Access restrictions
# -----------------------------

az webapp config access-restriction add \
  --resource-group "$RESOURCE_GROUP" \
  --name "$WEBAPP_NAME" \
  --rule-name "Allow-Admin-IP" \
  --action Allow \
  --ip-address "$APPROVED_PUBLIC_IP_CIDR" \
  --priority 100 |
  tee "$EVIDENCE_PATH/access-allow-admin-ip.json"

az webapp config access-restriction add \
  --resource-group "$RESOURCE_GROUP" \
  --name "$WEBAPP_NAME" \
  --rule-name "Deny-All" \
  --action Deny \
  --ip-address "0.0.0.0/0" \
  --priority 65000 |
  tee "$EVIDENCE_PATH/access-deny-all.json"

az webapp config access-restriction show \
  --name "$WEBAPP_NAME" \
  --resource-group "$RESOURCE_GROUP" \
  --output json |
  tee "$EVIDENCE_PATH/access-restrictions.json"

# -----------------------------
# Optional private endpoint
# -----------------------------
# Use when inbound private access is required.
# Requires DNS validation through privatelink.azurewebsites.net.

az network vnet subnet create \
  --resource-group "$RESOURCE_GROUP" \
  --vnet-name "$VNET_NAME" \
  --name "$PRIVATE_ENDPOINT_SUBNET" \
  --address-prefixes "$PRIVATE_ENDPOINT_SUBNET_PREFIX" |
  tee "$EVIDENCE_PATH/private-endpoint-subnet-create.json"

WEBAPP_ID=$(az webapp show \
  --name "$WEBAPP_NAME" \
  --resource-group "$RESOURCE_GROUP" \
  --query id \
  --output tsv)

az network private-endpoint create \
  --name "$PRIVATE_ENDPOINT_NAME" \
  --resource-group "$RESOURCE_GROUP" \
  --vnet-name "$VNET_NAME" \
  --subnet "$PRIVATE_ENDPOINT_SUBNET" \
  --private-connection-resource-id "$WEBAPP_ID" \
  --group-id sites \
  --connection-name "pec-$WEBAPP_NAME" |
  tee "$EVIDENCE_PATH/private-endpoint-create.json"

az network private-dns zone create \
  --resource-group "$RESOURCE_GROUP" \
  --name "$PRIVATE_DNS_ZONE" |
  tee "$EVIDENCE_PATH/private-dns-zone-create.json"

az network private-dns link vnet create \
  --resource-group "$RESOURCE_GROUP" \
  --zone-name "$PRIVATE_DNS_ZONE" \
  --name "link-$VNET_NAME" \
  --virtual-network "$VNET_NAME" \
  --registration-enabled false |
  tee "$EVIDENCE_PATH/private-dns-link.json"

# -----------------------------
# Deployment slot
# -----------------------------

az webapp deployment slot create \
  --name "$WEBAPP_NAME" \
  --resource-group "$RESOURCE_GROUP" \
  --slot "$SLOT_NAME" |
  tee "$EVIDENCE_PATH/slot-create.json"

az webapp config appsettings set \
  --name "$WEBAPP_NAME" \
  --resource-group "$RESOURCE_GROUP" \
  --slot "$SLOT_NAME" \
  --slot-settings ENVIRONMENT=staging APPROLE=web |
  tee "$EVIDENCE_PATH/slot-appsettings.json"

# Optional package deployment to staging slot.
# az webapp deployment source config-zip \
#   --name "$WEBAPP_NAME" \
#   --resource-group "$RESOURCE_GROUP" \
#   --slot "$SLOT_NAME" \
#   --src "$PACKAGE_ZIP" |
#   tee "$EVIDENCE_PATH/slot-zip-deploy.json"

az webapp deployment slot list \
  --name "$WEBAPP_NAME" \
  --resource-group "$RESOURCE_GROUP" \
  --output table |
  tee "$EVIDENCE_PATH/slot-list.txt"

# Swap staging into production after validation.
# az webapp deployment slot swap \
#   --name "$WEBAPP_NAME" \
#   --resource-group "$RESOURCE_GROUP" \
#   --slot "$SLOT_NAME" \
#   --target-slot production |
#   tee "$EVIDENCE_PATH/slot-swap.json"

# -----------------------------
# Final evidence
# -----------------------------

az appservice plan show \
  --name "$PLAN_NAME" \
  --resource-group "$RESOURCE_GROUP" \
  --output json |
  tee "$EVIDENCE_PATH/plan-final.json"

az webapp show \
  --name "$WEBAPP_NAME" \
  --resource-group "$RESOURCE_GROUP" \
  --output json |
  tee "$EVIDENCE_PATH/webapp-final.json"

az webapp config show \
  --name "$WEBAPP_NAME" \
  --resource-group "$RESOURCE_GROUP" \
  --output json |
  tee "$EVIDENCE_PATH/webapp-config-final.json"

az resource list \
  --resource-group "$RESOURCE_GROUP" \
  --output table |
  tee "$EVIDENCE_PATH/resource-list-final.txt"
```

# Deploy_App_Service_Plans_Web_Apps_TLS_Custom_Domains_Backup_Networking_And_Slots_AzureCLI_Container_WebApp_Skeleton

```bash
# Purpose:
# Create App Service web app from a container image instead of runtime code.

export RESOURCE_GROUP="rg-compute-lab-01"
export LOCATION="eastus"
export PLAN_NAME="asp-web-container-lab-01"
export WEBAPP_NAME="app-container-lab-01"
export PLAN_SKU="B1"

export ACR_LOGIN_SERVER="acrcomputelab01.azurecr.io"
export IMAGE_NAME="acrcomputelab01.azurecr.io/webapp:v1"

# Create Linux plan.
az appservice plan create \
  --name "$PLAN_NAME" \
  --resource-group "$RESOURCE_GROUP" \
  --location "$LOCATION" \
  --sku "$PLAN_SKU" \
  --is-linux

# Create web app from container image.
az webapp create \
  --name "$WEBAPP_NAME" \
  --resource-group "$RESOURCE_GROUP" \
  --plan "$PLAN_NAME" \
  --deployment-container-image-name "$IMAGE_NAME"

# Assign system identity.
az webapp identity assign \
  --name "$WEBAPP_NAME" \
  --resource-group "$RESOURCE_GROUP"

WEBAPP_PRINCIPAL_ID=$(az webapp identity show \
  --name "$WEBAPP_NAME" \
  --resource-group "$RESOURCE_GROUP" \
  --query principalId \
  --output tsv)

ACR_ID=$(az acr show \
  --name "acrcomputelab01" \
  --query id \
  --output tsv)

# Grant AcrPull.
az role assignment create \
  --assignee "$WEBAPP_PRINCIPAL_ID" \
  --role AcrPull \
  --scope "$ACR_ID"

# Configure registry identity-based pull where supported.
az webapp config set \
  --name "$WEBAPP_NAME" \
  --resource-group "$RESOURCE_GROUP" \
  --generic-configurations '{"acrUseManagedIdentityCreds": true}'

# Configure image.
az webapp config container set \
  --name "$WEBAPP_NAME" \
  --resource-group "$RESOURCE_GROUP" \
  --docker-custom-image-name "$IMAGE_NAME" \
  --docker-registry-server-url "https://$ACR_LOGIN_SERVER"

# Verify.
az webapp show \
  --name "$WEBAPP_NAME" \
  --resource-group "$RESOURCE_GROUP" \
  --query "{name:name,defaultHostName:defaultHostName,state:state}" \
  -o table

az webapp log tail \
  --name "$WEBAPP_NAME" \
  --resource-group "$RESOURCE_GROUP"
```

# Deploy_App_Service_Plans_Web_Apps_TLS_Custom_Domains_Backup_Networking_And_Slots_PowerShell_Skeleton

```powershell
# Run from PowerShell with Az module installed.
# Purpose: create App Service Plan and Web App baseline, then use Azure CLI for operations where CLI is cleaner.

# -----------------------------
# Variables
# -----------------------------

$SubscriptionId = "<subscription-id>"
$Location = "eastus"
$ResourceGroupName = "rg-compute-lab-01"
$PlanName = "asp-web-lab-01"
$WebAppName = "app-web-lab-01"
$PlanSku = "B1"
$RuntimeStack = "PYTHON:3.11"

$EvidencePath = ".\evidence\06-appservice-webapps"

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
# App Service Plan
# -----------------------------

New-AzAppServicePlan `
  -ResourceGroupName $ResourceGroupName `
  -Name $PlanName `
  -Location $Location `
  -Tier Basic `
  -NumberofWorkers 1 `
  -WorkerSize Small `
  -Linux |
  Tee-Object (Join-Path $EvidencePath "plan-create.txt")

Get-AzAppServicePlan `
  -ResourceGroupName $ResourceGroupName `
  -Name $PlanName |
  Tee-Object (Join-Path $EvidencePath "plan-show.txt")

# -----------------------------
# Web App
# -----------------------------
# Azure CLI runtime syntax is usually cleaner for Linux stack creation.

az webapp create `
  --name $WebAppName `
  --resource-group $ResourceGroupName `
  --plan $PlanName `
  --runtime $RuntimeStack |
  Tee-Object (Join-Path $EvidencePath "webapp-create.txt")

az webapp config appsettings set `
  --name $WebAppName `
  --resource-group $ResourceGroupName `
  --settings ENVIRONMENT=lab APPROLE=web |
  Tee-Object (Join-Path $EvidencePath "appsettings-set.txt")

az webapp update `
  --name $WebAppName `
  --resource-group $ResourceGroupName `
  --https-only true |
  Tee-Object (Join-Path $EvidencePath "https-only.txt")

az webapp config set `
  --name $WebAppName `
  --resource-group $ResourceGroupName `
  --min-tls-version 1.2 `
  --ftps-state Disabled |
  Tee-Object (Join-Path $EvidencePath "webapp-config-security.txt")

# -----------------------------
# Slots
# -----------------------------

az webapp deployment slot create `
  --name $WebAppName `
  --resource-group $ResourceGroupName `
  --slot staging |
  Tee-Object (Join-Path $EvidencePath "slot-create.txt")

az webapp deployment slot list `
  --name $WebAppName `
  --resource-group $ResourceGroupName `
  --output table |
  Tee-Object (Join-Path $EvidencePath "slot-list.txt")

# -----------------------------
# Verify
# -----------------------------

Get-AzWebApp `
  -ResourceGroupName $ResourceGroupName `
  -Name $WebAppName |
  Tee-Object (Join-Path $EvidencePath "webapp-show.txt")

az webapp show `
  --name $WebAppName `
  --resource-group $ResourceGroupName `
  --output json |
  Tee-Object (Join-Path $EvidencePath "webapp-show.json")
```

# Deploy_App_Service_Plans_Web_Apps_TLS_Custom_Domains_Backup_Networking_And_Slots_Bicep_Skeleton

```bicep
// File: .\infra\06-appservice-webapps\main.bicep
// Purpose: Deploy App Service Plan, Linux Web App, staging slot, HTTPS-only, app settings, and basic access restriction.
// Custom domain DNS and certificate binding often require external DNS readiness, so those are handled in CLI/portal workflow.

targetScope = 'resourceGroup'

@description('Azure region.')
param location string = resourceGroup().location

@description('App Service Plan name.')
param planName string = 'asp-web-lab-01'

@description('Web app name. Must be globally unique.')
param webAppName string = 'app-web-lab-01'

@description('Linux runtime stack.')
param linuxFxVersion string = 'PYTHON|3.11'

@description('App Service Plan SKU.')
param skuName string = 'B1'

@description('App Service Plan tier.')
param skuTier string = 'Basic'

@description('Worker count.')
param workerCount int = 1

@description('Approved public source CIDR for access restriction.')
param approvedPublicIpCidr string = '<approved-public-ip-cidr>'

@description('Staging slot name.')
param slotName string = 'staging'

var tags = {
  Environment: 'Lab'
  Workload: 'Compute'
  ManagedBy: 'Bicep'
  Workbook: '06_Deploy_App_Service_Plans_Web_Apps_TLS_Custom_Domains_Backup_Networking_And_Slots'
}

resource plan 'Microsoft.Web/serverfarms@2023-12-01' = {
  name: planName
  location: location
  tags: tags
  sku: {
    name: skuName
    tier: skuTier
    capacity: workerCount
  }
  kind: 'linux'
  properties: {
    reserved: true
  }
}

resource webApp 'Microsoft.Web/sites@2023-12-01' = {
  name: webAppName
  location: location
  tags: tags
  kind: 'app,linux'
  properties: {
    serverFarmId: plan.id
    httpsOnly: true
    siteConfig: {
      linuxFxVersion: linuxFxVersion
      minTlsVersion: '1.2'
      ftpsState: 'Disabled'
      alwaysOn: false
      appSettings: [
        {
          name: 'ENVIRONMENT'
          value: 'lab'
        }
        {
          name: 'APPROLE'
          value: 'web'
        }
      ]
      ipSecurityRestrictions: [
        {
          name: 'Allow-Admin-IP'
          action: 'Allow'
          priority: 100
          ipAddress: approvedPublicIpCidr
        }
        {
          name: 'Deny-All'
          action: 'Deny'
          priority: 65000
          ipAddress: '0.0.0.0/0'
        }
      ]
      scmIpSecurityRestrictionsUseMain: true
    }
  }
}

resource stagingSlot 'Microsoft.Web/sites/slots@2023-12-01' = {
  name: '${webApp.name}/${slotName}'
  location: location
  tags: tags
  kind: 'app,linux'
  properties: {
    serverFarmId: plan.id
    httpsOnly: true
    siteConfig: {
      linuxFxVersion: linuxFxVersion
      minTlsVersion: '1.2'
      ftpsState: 'Disabled'
      appSettings: [
        {
          name: 'ENVIRONMENT'
          value: 'staging'
        }
        {
          name: 'APPROLE'
          value: 'web'
        }
      ]
    }
  }
}

resource stickySlotSettings 'Microsoft.Web/sites/config@2023-12-01' = {
  name: '${webApp.name}/slotConfigNames'
  properties: {
    appSettingNames: [
      'ENVIRONMENT'
    ]
  }
}

output webAppId string = webApp.id
output defaultHostName string = webApp.properties.defaultHostName
output stagingSlotName string = stagingSlot.name
```

# Deploy_App_Service_Plans_Web_Apps_TLS_Custom_Domains_Backup_Networking_And_Slots_Bicep_Parameters_Skeleton

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "location": {
      "value": "eastus"
    },
    "planName": {
      "value": "asp-web-lab-01"
    },
    "webAppName": {
      "value": "app-web-lab-01"
    },
    "linuxFxVersion": {
      "value": "PYTHON|3.11"
    },
    "skuName": {
      "value": "B1"
    },
    "skuTier": {
      "value": "Basic"
    },
    "workerCount": {
      "value": 1
    },
    "approvedPublicIpCidr": {
      "value": "<approved-public-ip-cidr>"
    },
    "slotName": {
      "value": "staging"
    }
  }
}
```

# Deploy_App_Service_Plans_Web_Apps_TLS_Custom_Domains_Backup_Networking_And_Slots_App_Service_Plan_Skeleton

```text
Purpose:
Choose and operate the App Service Plan correctly.

Plan controls:
1. Region:
   - Apps in the plan run in the plan region.
   - Moving regions usually means redeploying to another plan.

2. OS:
   - Linux plan for Linux runtimes and Linux containers.
   - Windows plan for Windows runtimes and Windows containers.

3. SKU:
   - Free/Shared:
     - Basic tests only.
     - Not suitable for slots, custom production-like features, or steady workloads.
   - Basic:
     - Simple lab and dev/test.
   - Standard:
     - Common baseline for deployment slots and production-like labs.
   - Premium v3:
     - Better performance and production-style hosting.
   - Isolated:
     - App Service Environment scenarios.

4. Scale up:
   - Change SKU or worker size.
   - Used when app needs more CPU, memory, or platform features.

5. Scale out:
   - Increase worker count.
   - Used when app needs more parallel capacity.

CLI:
az appservice plan create \
  --name <plan-name> \
  --resource-group <rg> \
  --location <region> \
  --sku B1 \
  --is-linux

az appservice plan update \
  --name <plan-name> \
  --resource-group <rg> \
  --sku S1

az appservice plan update \
  --name <plan-name> \
  --resource-group <rg> \
  --number-of-workers 2

Evidence:
1. Plan name
2. Region
3. OS
4. SKU
5. Worker count
6. Apps hosted on plan
7. Cost posture
```

# Deploy_App_Service_Plans_Web_Apps_TLS_Custom_Domains_Backup_Networking_And_Slots_Custom_Domain_TLS_Skeleton

```text
Purpose:
Bind a custom domain and protect it with TLS.

Custom domain workflow:
1. Confirm app default hostname:
   <web-app-name>.azurewebsites.net

2. Get verification ID:
   az webapp show \
     --name <web-app-name> \
     --resource-group <rg> \
     --query customDomainVerificationId -o tsv

3. Create DNS records:
   For subdomain:
   - CNAME:
     app.contoso.com -> <web-app-name>.azurewebsites.net

   - TXT:
     asuid.app.contoso.com -> <custom-domain-verification-id>

   For apex domain:
   - A record:
     contoso.com -> App Service inbound IP

   - TXT:
     asuid.contoso.com -> <custom-domain-verification-id>

4. Bind hostname:
   az webapp config hostname add \
     --webapp-name <web-app-name> \
     --resource-group <rg> \
     --hostname app.contoso.com

5. Create managed certificate:
   az webapp config ssl create \
     --resource-group <rg> \
     --name <web-app-name> \
     --hostname app.contoso.com

6. Bind certificate:
   az webapp config ssl bind \
     --resource-group <rg> \
     --name <web-app-name> \
     --certificate-thumbprint <thumbprint> \
     --ssl-type SNI

7. Enforce HTTPS:
   az webapp update \
     --name <web-app-name> \
     --resource-group <rg> \
     --https-only true

8. Set minimum TLS:
   az webapp config set \
     --name <web-app-name> \
     --resource-group <rg> \
     --min-tls-version 1.2

Validation:
1. nslookup app.contoso.com
2. curl -I https://app.contoso.com
3. Confirm certificate CN/SAN.
4. Confirm HTTP does not remain open.
5. Confirm TLS binding in portal.

Common issues:
1. DNS has not propagated.
2. TXT record name is wrong.
3. CNAME points to the wrong app.
4. Managed certificate is not eligible for the hostname.
5. Certificate thumbprint not bound.
6. HTTPS-only disabled.
```

# Deploy_App_Service_Plans_Web_Apps_TLS_Custom_Domains_Backup_Networking_And_Slots_Backup_Skeleton

```text
Purpose:
Configure App Service backup and restore.

Backup requirements:
1. Storage account.
2. Blob container.
3. SAS URL with required permissions.
4. App Service plan tier that supports backup.
5. App content and optional database connection settings.

Create storage:
az storage account create \
  --name <storage-account-name> \
  --resource-group <rg> \
  --location <region> \
  --sku Standard_LRS \
  --kind StorageV2

Create container:
az storage container create \
  --name <backup-container> \
  --account-name <storage-account-name>

Create backup:
az webapp config backup create \
  --resource-group <rg> \
  --webapp-name <web-app-name> \
  --backup-name <backup-name> \
  --container-url <container-sas-url>

List backups:
az webapp config backup list \
  --resource-group <rg> \
  --webapp-name <web-app-name> \
  -o table

Scheduled backup:
1. Use portal or automation to configure schedule.
2. Keep SAS expiry aligned with retention plan.
3. Store backup storage separately from app runtime configuration.
4. Test restore before relying on backup.

Restore planning:
1. Restore to staging slot first when possible.
2. Validate app before production restore.
3. Keep database restore process separate unless included and tested.
4. Document backup ID, timestamp, storage container, and restore target.

Evidence:
1. Backup configuration
2. Backup job status
3. Storage account
4. Container
5. SAS expiration
6. Restore test result
```

# Deploy_App_Service_Plans_Web_Apps_TLS_Custom_Domains_Backup_Networking_And_Slots_Networking_Skeleton

```text
Purpose:
Separate App Service inbound and outbound networking.

Inbound controls:
1. Public endpoint:
   - Default azurewebsites.net endpoint.
   - Custom domain endpoint.
   - Controlled by access restrictions, private endpoint, and TLS.

2. Access restrictions:
   - IP allow rules.
   - Service tag rules.
   - VNet/subnet rules.
   - Deny-all fallback.

3. Private endpoint:
   - Gives the app a private IP inside a VNet.
   - Requires private DNS for reliable name resolution.
   - Typical private DNS zone:
     privatelink.azurewebsites.net

Outbound controls:
1. VNet integration:
   - Allows app outbound traffic into a VNet.
   - Used to reach private databases, storage, APIs, or internal services.
   - Does not make inbound traffic private by itself.

2. Route all:
   - Sends outbound traffic through VNet routes when required.
   - Useful with firewall/NVA designs.

3. NAT gateway:
   - Can provide stable outbound public IP through integration subnet.

Access restriction example:
az webapp config access-restriction add \
  --resource-group <rg> \
  --name <web-app-name> \
  --rule-name Allow-Admin-IP \
  --action Allow \
  --ip-address <approved-public-ip-cidr> \
  --priority 100

az webapp config access-restriction add \
  --resource-group <rg> \
  --name <web-app-name> \
  --rule-name Deny-All \
  --action Deny \
  --ip-address 0.0.0.0/0 \
  --priority 65000

VNet integration example:
az webapp vnet-integration add \
  --name <web-app-name> \
  --resource-group <rg> \
  --vnet <vnet-name> \
  --subnet <integration-subnet-name>

Private endpoint flow:
1. Create private endpoint subnet.
2. Create private endpoint targeting App Service `sites`.
3. Create private DNS zone:
   privatelink.azurewebsites.net
4. Link private DNS zone to VNet.
5. Confirm app name resolves to private IP from inside VNet.
6. Disable or restrict public access if required.

Validation:
1. curl from approved source.
2. curl from unapproved source.
3. nslookup from private network.
4. App logs show requests.
5. Access restriction logs or HTTP status confirm block behavior.
```

# Deploy_App_Service_Plans_Web_Apps_TLS_Custom_Domains_Backup_Networking_And_Slots_Slots_Skeleton

```text
Purpose:
Use deployment slots to reduce production release risk.

Slot model:
1. Production slot:
   - Main app endpoint.
   - Custom domain usually points here.

2. Staging slot:
   - Separate live app instance.
   - Has its own hostname:
     <web-app-name>-staging.azurewebsites.net

3. Swap:
   - Exchanges staging and production.
   - Warms staging before production traffic moves.
   - Can be reversed.

Create staging slot:
az webapp deployment slot create \
  --name <web-app-name> \
  --resource-group <rg> \
  --slot staging

Configure staging settings:
az webapp config appsettings set \
  --name <web-app-name> \
  --resource-group <rg> \
  --slot staging \
  --settings ENVIRONMENT=staging

Set sticky slot settings:
az webapp config appsettings set \
  --name <web-app-name> \
  --resource-group <rg> \
  --slot-settings ENVIRONMENT=staging

Deploy to staging:
az webapp deployment source config-zip \
  --name <web-app-name> \
  --resource-group <rg> \
  --slot staging \
  --src <package.zip>

Validate staging:
1. Browse:
   https://<web-app-name>-staging.azurewebsites.net

2. Check logs:
   az webapp log tail -g <rg> -n <web-app-name> --slot staging

3. Confirm app settings.
4. Confirm database or dependency path.
5. Confirm health endpoint.

Swap:
az webapp deployment slot swap \
  --name <web-app-name> \
  --resource-group <rg> \
  --slot staging \
  --target-slot production

Rollback:
az webapp deployment slot swap \
  --name <web-app-name> \
  --resource-group <rg> \
  --slot staging \
  --target-slot production

Rules:
1. Keep environment-specific settings sticky.
2. Test staging before swap.
3. Use preview swap when dependencies are sensitive.
4. Keep old production in staging after swap until validation is complete.
5. Swap back fast if production validation fails.
```

# Deploy_App_Service_Plans_Web_Apps_TLS_Custom_Domains_Backup_Networking_And_Slots_Verification_Commands

```bash
# Context
az account show -o table

# Providers
az provider show --namespace Microsoft.Web --query registrationState -o tsv
az provider show --namespace Microsoft.Storage --query registrationState -o tsv
az provider show --namespace Microsoft.Network --query registrationState -o tsv

# Resource group
az group show \
  --name "<resource-group-name>" \
  -o table

# App Service Plan
az appservice plan show \
  --name "<app-service-plan-name>" \
  --resource-group "<resource-group-name>" \
  -o json

az appservice plan list \
  --resource-group "<resource-group-name>" \
  -o table

# Web App
az webapp show \
  --name "<web-app-name>" \
  --resource-group "<resource-group-name>" \
  -o json

az webapp show \
  --name "<web-app-name>" \
  --resource-group "<resource-group-name>" \
  --query "{name:name,state:state,defaultHostName:defaultHostName,httpsOnly:httpsOnly,enabled:enabled}" \
  -o table

# App settings
az webapp config appsettings list \
  --name "<web-app-name>" \
  --resource-group "<resource-group-name>" \
  -o table

# App config
az webapp config show \
  --name "<web-app-name>" \
  --resource-group "<resource-group-name>" \
  --query "{linuxFxVersion:linuxFxVersion,minTlsVersion:minTlsVersion,ftpsState:ftpsState,alwaysOn:alwaysOn}" \
  -o table

# Hostnames
az webapp config hostname list \
  --webapp-name "<web-app-name>" \
  --resource-group "<resource-group-name>" \
  -o table

# TLS certificates
az webapp config ssl list \
  --resource-group "<resource-group-name>" \
  -o table

# Access restrictions
az webapp config access-restriction show \
  --name "<web-app-name>" \
  --resource-group "<resource-group-name>" \
  -o json

# VNet integration
az webapp vnet-integration list \
  --name "<web-app-name>" \
  --resource-group "<resource-group-name>" \
  -o table

# Private endpoint
az network private-endpoint list \
  --resource-group "<resource-group-name>" \
  -o table

# Backup
az webapp config backup list \
  --resource-group "<resource-group-name>" \
  --webapp-name "<web-app-name>" \
  -o table

# Slots
az webapp deployment slot list \
  --name "<web-app-name>" \
  --resource-group "<resource-group-name>" \
  -o table

az webapp show \
  --name "<web-app-name>" \
  --resource-group "<resource-group-name>" \
  --slot staging \
  -o json

# Logs
az webapp log config \
  --name "<web-app-name>" \
  --resource-group "<resource-group-name>"

az webapp log tail \
  --name "<web-app-name>" \
  --resource-group "<resource-group-name>"

# Endpoint checks
curl -I "https://<web-app-name>.azurewebsites.net"
curl -I "https://<custom-domain>"

# Activity log
az monitor activity-log list \
  --resource-group "<resource-group-name>" \
  --max-events 30 \
  -o table

# Resource inventory
az resource list \
  --resource-group "<resource-group-name>" \
  -o table
```

```powershell
# Context
Get-AzContext

# Resource group
Get-AzResourceGroup -Name "<resource-group-name>"

# App Service Plan
Get-AzAppServicePlan `
  -ResourceGroupName "<resource-group-name>" `
  -Name "<app-service-plan-name>"

# Web App
Get-AzWebApp `
  -ResourceGroupName "<resource-group-name>" `
  -Name "<web-app-name>"

# Use Azure CLI from PowerShell for detailed App Service checks
az webapp show `
  --name "<web-app-name>" `
  --resource-group "<resource-group-name>" `
  --output table

az webapp config hostname list `
  --webapp-name "<web-app-name>" `
  --resource-group "<resource-group-name>" `
  --output table

az webapp deployment slot list `
  --name "<web-app-name>" `
  --resource-group "<resource-group-name>" `
  --output table
```

# Deploy_App_Service_Plans_Web_Apps_TLS_Custom_Domains_Backup_Networking_And_Slots_Rollback

| Change | Rollback Command / Action | Risk |
|---|---|---|
| Web app created for lab | `az webapp delete --name "<web-app-name>" --resource-group "<resource-group-name>"` | Deletes app and production configuration |
| App Service Plan created for lab | `az appservice plan delete --name "<app-service-plan-name>" --resource-group "<resource-group-name>" --yes` | Deletes plan if no apps depend on it |
| Full lab resource group created | `az group delete --name "<resource-group-name>" --yes --no-wait` | Deletes all resources in the RG |
| App setting changed incorrectly | `az webapp config appsettings set --name "<web-app-name>" --resource-group "<resource-group-name>" --settings KEY=oldvalue` | App restart may occur |
| HTTPS-only enabled | `az webapp update --name "<web-app-name>" --resource-group "<resource-group-name>" --https-only false` | Plain HTTP becomes available |
| TLS minimum changed | `az webapp config set --name "<web-app-name>" --resource-group "<resource-group-name>" --min-tls-version "<old-version>"` | Lower TLS versions may be less secure |
| Custom hostname added | `az webapp config hostname delete --webapp-name "<web-app-name>" --resource-group "<resource-group-name>" --hostname "<custom-domain>"` | Custom domain stops resolving to app |
| TLS binding added | Remove binding in portal or replace with correct cert | HTTPS for custom domain may break |
| Managed certificate created | Delete certificate if unused | Hostname binding may depend on it |
| DNS record changed | Revert CNAME, A, or TXT at DNS provider | DNS propagation delay applies |
| Access restriction blocks admins | Remove deny rule or add correct allow rule | Public exposure may temporarily increase |
| VNet integration added | `az webapp vnet-integration remove --name "<web-app-name>" --resource-group "<resource-group-name>" --vnet "<vnet-name>"` | Private outbound dependencies may fail |
| Private endpoint added | `az network private-endpoint delete --name "<private-endpoint-name>" --resource-group "<resource-group-name>"` | Private inbound path breaks |
| Backup storage created | Delete storage account after backup review | Deletes backup data |
| Bad staging deployment | Redeploy known-good package to staging | Staging only if not swapped |
| Bad slot swap | Run the same swap command again to swap back | Production changes revert to previous slot state |
| Slot setting not sticky | Mark setting sticky and swap back if needed | Production may receive staging config |
| Plan scaled up | `az appservice plan update --name "<app-service-plan-name>" --resource-group "<resource-group-name>" --sku "<old-sku>"` | Feature loss if lower SKU lacks required capabilities |
| Worker count increased | `az appservice plan update --name "<app-service-plan-name>" --resource-group "<resource-group-name>" --number-of-workers "<old-count>"` | Capacity decreases |

# Deploy_App_Service_Plans_Web_Apps_TLS_Custom_Domains_Backup_Networking_And_Slots_Failure_Checks

| Symptom | Likely Layer | First Check | Fix |
|---|---|---|---|
| App Service Plan creation fails | Provider, region, SKU, or RBAC | Provider state and error detail | Register provider, change region/SKU, or fix RBAC |
| Web app name rejected | Global name conflict or invalid name | Error detail | Use globally unique valid app name |
| Runtime stack rejected | OS or runtime mismatch | Plan OS and runtime list | Match Linux/Windows plan and valid runtime |
| Web app returns default page | App not deployed | Deployment logs and app content | Deploy package or container |
| App returns 500 | App runtime/config issue | App logs | Fix code, app settings, connection strings, or runtime |
| App returns 502/503 | Startup failure or container not listening | Logs and container port | Fix startup command, target port, or dependencies |
| Logs not showing | Logging disabled | `az webapp log config` | Enable application logging |
| Custom domain add fails | DNS validation failed | CNAME/TXT lookup | Fix DNS record and wait for propagation |
| TXT record not found | Wrong record name | DNS provider record | Use `asuid.<host>` with verification ID |
| CNAME points wrong target | DNS misconfiguration | `nslookup <custom-domain>` | Point CNAME to `<web-app-name>.azurewebsites.net` |
| Managed certificate creation fails | Hostname not eligible or DNS issue | Hostname binding and DNS | Fix domain binding or use uploaded certificate |
| TLS binding fails | Wrong thumbprint or cert missing | `az webapp config ssl list` | Use correct cert thumbprint |
| HTTP still accessible | HTTPS-only disabled | Web app config | Enable HTTPS-only |
| Old TLS allowed | Minimum TLS too low | Web app config | Set minimum TLS 1.2 or higher |
| Backup create fails | SAS permissions or plan tier | Backup error and storage container | Fix SAS permissions, expiry, or plan tier |
| Backup succeeds but restore not tested | Operational gap | Backup list only | Restore to slot and validate |
| VNet integration fails | Subnet issue or unsupported plan | Subnet and plan SKU | Use valid delegated subnet and supported plan |
| App cannot reach private resource | DNS, route, firewall, or integration issue | App console, DNS, route, firewall logs | Fix VNet integration, DNS, route, or firewall rule |
| Private endpoint created but name resolves public | DNS missing | `nslookup` from VNet | Configure private DNS zone and VNet link |
| Public app still reachable after private endpoint | Public access not restricted | Access restrictions | Add restrictions or disable public access where supported |
| Access restriction blocks staging tests | Rules apply unexpectedly | Access restriction list | Add approved test source or slot-specific rules |
| Slot creation fails | Plan SKU does not support slots | Plan SKU | Move to Standard or higher |
| Swap fails | Slot config, app warm-up, or setting conflict | Swap error details | Fix slot settings and retry |
| Production gets staging config | Sticky settings missing | Slot setting list | Mark environment-specific settings as slot settings |
| Zip deploy fails | Package structure or auth | Deployment logs | Fix zip root, deployment method, or permissions |
| Container web app cannot pull image | Registry auth | Identity and AcrPull | Assign AcrPull and configure registry identity |
| Container web app starts then exits | App process issue | Container logs | Fix image entrypoint, app settings, or port |
| Scale-up breaks feature expectation | Lower SKU missing feature | Plan SKU | Move to supported SKU |
| Cost higher than expected | SKU, worker count, slots, storage | Cost analysis and resource list | Scale down, delete slots, delete backup storage if not needed |

# Deploy_App_Service_Plans_Web_Apps_TLS_Custom_Domains_Backup_Networking_And_Slots_Related_Labs

| Lab                                                                 | Related Workbook                                                                         | Skill Proven                                                     |
| ------------------------------------------------------------------- | ---------------------------------------------------------------------------------------- | ---------------------------------------------------------------- |
| Deploy ARM and Bicep baseline                                       | `01_Deploy_ARM_Templates_Bicep_And_Exported_Deployments.md`                              | Repeatable deployment foundation                                 |
| Create and manage Azure VMs                                         | `02_Create_Configure_And_Manage_Azure_Virtual_Machines.md`                               | IaaS compute lifecycle                                           |
| Configure VM disks, sizes, availability, extensions, and encryption | `03_Configure_VM_Disks_Sizes_Availability_Sets_Zones_Extensions_And_Encryption.md`       | VM platform configuration                                        |
| Configure VMSS images and scaling                                   | `04_Configure_VMSS_Images_Scaling_And_Automation.md`                                     | Scale set management                                             |
| Deploy ACR, ACI, and Azure Container Apps                           | `05_Deploy_ACR_ACI_Azure_Container_Apps_Sizing_And_Scaling.md`                           | Container hosting and scaling                                    |
| Deploy App Service plans and web apps                               | `06_Deploy_App_Service_Plans_Web_Apps_TLS_Custom_Domains_Backup_Networking_And_Slots.md` | PaaS web app hosting, TLS, backup, networking, and release slots |
| Configure compute managed identities and Key Vault references       | `07_Configure_Compute_Managed_Identities_Key_Vault_Access_And_Secrets_References.md`     | Secretless app access                                            |
| Troubleshoot compute deployment and runtime issues                  | `08_Troubleshoot_Azure_VM_Container_App_Service_Deployment_And_Runtime_Issues.md`        | App Service and compute troubleshooting                          |