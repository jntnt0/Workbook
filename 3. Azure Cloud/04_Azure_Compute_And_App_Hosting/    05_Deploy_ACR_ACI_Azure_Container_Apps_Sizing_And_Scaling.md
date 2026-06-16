# 05_Deploy_ACR_ACI_Azure_Container_Apps_Sizing_And_Scaling

# 05_Deploy_ACR_ACI_Azure_Container_Apps_Sizing_And_Scaling_Index

05_Deploy_ACR_ACI_Azure_Container_Apps_Sizing_And_Scaling.md  
Deploy_ACR_ACI_Azure_Container_Apps_Sizing_And_Scaling  
Deploy_ACR_ACI_Azure_Container_Apps_Sizing_And_Scaling_Source_Basis  
Deploy_ACR_ACI_Azure_Container_Apps_Sizing_And_Scaling_Mental_Model  
Deploy_ACR_ACI_Azure_Container_Apps_Sizing_And_Scaling_Planning_Table  
Deploy_ACR_ACI_Azure_Container_Apps_Sizing_And_Scaling_Configuration_Checklist  
Deploy_ACR_ACI_Azure_Container_Apps_Sizing_And_Scaling_Portal_Skeleton  
Deploy_ACR_ACI_Azure_Container_Apps_Sizing_And_Scaling_AzureCLI_Skeleton  
Deploy_ACR_ACI_Azure_Container_Apps_Sizing_And_Scaling_PowerShell_Skeleton  
Deploy_ACR_ACI_Azure_Container_Apps_Sizing_And_Scaling_Bicep_Skeleton  
Deploy_ACR_ACI_Azure_Container_Apps_Sizing_And_Scaling_ACR_Build_Push_Pull_Skeleton  
Deploy_ACR_ACI_Azure_Container_Apps_Sizing_And_Scaling_ACI_Runtime_Skeleton  
Deploy_ACR_ACI_Azure_Container_Apps_Sizing_And_Scaling_Container_Apps_Runtime_Skeleton  
Deploy_ACR_ACI_Azure_Container_Apps_Sizing_And_Scaling_Sizing_And_Scaling_Skeleton  
Deploy_ACR_ACI_Azure_Container_Apps_Sizing_And_Scaling_Verification_Commands  
Deploy_ACR_ACI_Azure_Container_Apps_Sizing_And_Scaling_Rollback  
Deploy_ACR_ACI_Azure_Container_Apps_Sizing_And_Scaling_Failure_Checks  
Deploy_ACR_ACI_Azure_Container_Apps_Sizing_And_Scaling_Related_Labs  

# Deploy_ACR_ACI_Azure_Container_Apps_Sizing_And_Scaling_Source_Basis

| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Azure Container Registry overview | Private image registry, repositories, tags, SKUs, tasks, authentication, and image pull workflow |
| Microsoft Learn | ACR quickstart with Azure CLI | Create registry, build image, push image, list repositories, and pull image |
| Microsoft Learn | ACR Tasks | Cloud-side container image build without local Docker dependency |
| Microsoft Learn | Azure Container Instances overview | Fast single-container and container-group execution without orchestrator management |
| Microsoft Learn | ACI quickstarts | Deploy public images, expose ports, set environment variables, collect logs, and delete container groups |
| Microsoft Learn | ACI with Azure Container Registry | Pull private images from ACR |
| Microsoft Learn | Azure Container Apps overview | Serverless container app hosting, environments, revisions, ingress, scaling, logs, and workload profiles |
| Microsoft Learn | Container Apps quickstarts | Deploy from existing image, deploy from code, deploy jobs, configure ingress, and manage revisions |
| Microsoft Learn | Container Apps scaling | Min replicas, max replicas, HTTP scale rules, event scale rules, and KEDA-based scaling behavior |
| Microsoft Learn | Container Apps managed identity image pull | Pull private ACR images using system-assigned or user-assigned managed identity |
| Microsoft Learn | Container Apps secrets | Store registry credentials, app secrets, and environment variables |
| Microsoft Learn | Container Apps logs and monitoring | Log streaming, console access, metrics, alerts, and Log Analytics integration |
| Microsoft Learn | Container Apps troubleshooting | Image pull, target port, health probe, storage mount, container start, and container exit failures |
| Azure operational practice | Container build lifecycle | Build image, tag image, push image, scan image, deploy image, verify runtime, and promote tag |
| Azure operational practice | Runtime selection | ACR stores images, ACI runs quick containers, Container Apps hosts scalable applications |
| Azure operational practice | Evidence capture | Record registry, image tag, digest, container runtime state, ingress FQDN, replica count, revision, logs, and scaling settings |

# Deploy_ACR_ACI_Azure_Container_Apps_Sizing_And_Scaling_Mental_Model

| Concept | Operational Meaning |
|---|---|
| Container image | Packaged application and runtime dependencies |
| Container registry | Image storage service where container images are pushed and pulled |
| ACR | Azure Container Registry, private Azure-hosted container registry |
| Repository | Logical image collection inside ACR, such as `webapp` |
| Tag | Human-readable image version marker, such as `v1` or `2026.06.15` |
| Digest | Immutable image content hash |
| ACR SKU | Registry tier controlling features, throughput, storage, geo-replication, and private endpoint capability |
| ACR Tasks | Azure-side image build and automation service |
| Image pull | Runtime downloads image from registry |
| AcrPull | Azure RBAC role that allows a managed identity or service principal to pull images from ACR |
| ACI | Azure Container Instances, fast container runtime for simple jobs, tests, and short-lived workloads |
| Container group | ACI scheduling boundary for one or more containers sharing lifecycle, network, and volumes |
| Container Apps | Azure app platform for containerized services with ingress, revisions, scaling, secrets, and managed environment |
| Container Apps environment | Boundary where container apps run, share networking, logging, and workload profile settings |
| Revision | Immutable Container Apps deployment snapshot |
| Active revision | Revision currently receiving traffic |
| Ingress | HTTP or TCP entry point into a container app |
| Target port | Port exposed by the container process inside the app |
| External ingress | Publicly reachable endpoint |
| Internal ingress | Private environment-only endpoint |
| Min replicas | Lowest number of app replicas kept ready |
| Max replicas | Highest number of replicas Container Apps can create |
| HTTP scale rule | Scaling based on concurrent HTTP requests |
| KEDA | Event-driven scaler behind Container Apps scale rules |
| Workload profile | Dedicated compute profile in a Container Apps environment |
| Consumption profile | Serverless scale-to-zero capable hosting model |
| CPU allocation | Compute share assigned to container |
| Memory allocation | Memory assigned to container |
| First rule | Registry, image, identity, ingress, port, and scaling must all line up or deployment will fail |
| Blunt rule | A container that runs locally is not proven in Azure until image pull, startup, port binding, logs, and scale behavior are verified |

# Deploy_ACR_ACI_Azure_Container_Apps_Sizing_And_Scaling_Planning_Table

| Item | Example | Decision |
|---|---|---|
| Tenant | `contoso.onmicrosoft.com` | `<tenant-name>` |
| Subscription ID | `00000000-0000-0000-0000-000000000000` | `<subscription-id>` |
| Resource group | `rg-compute-lab-01` | `<resource-group-name>` |
| Location | `eastus` | `<azure-region>` |
| ACR name | `acrcomputelab01` | `<acr-name>` |
| ACR SKU | `Basic` | `<Basic / Standard / Premium>` |
| Repository name | `webapp` | `<repository-name>` |
| Image tag | `v1` | `<image-tag>` |
| Full image name | `acrcomputelab01.azurecr.io/webapp:v1` | `<full-image-name>` |
| Dockerfile path | `./src/Dockerfile` | `<dockerfile-path>` |
| Build context | `./src` | `<build-context>` |
| ACI container group | `aci-web-lab-01` | `<aci-name>` |
| ACI CPU | `1` | `<aci-cpu>` |
| ACI memory | `1.5` | `<aci-memory-gb>` |
| ACI restart policy | `Never` | `<Always / Never / OnFailure>` |
| ACI exposed port | `80` | `<aci-port>` |
| ACI DNS label | `aci-web-lab-01` | `<dns-label>` |
| Container Apps environment | `cae-compute-lab-01` | `<container-app-env-name>` |
| Log Analytics workspace | `law-compute-lab-01` | `<workspace-name>` |
| Container app name | `ca-web-lab-01` | `<container-app-name>` |
| App CPU | `0.5` | `<container-app-cpu>` |
| App memory | `1.0Gi` | `<container-app-memory>` |
| App target port | `80` | `<target-port>` |
| Ingress | `external` | `<external / internal / disabled>` |
| Min replicas | `1` | `<min-replicas>` |
| Max replicas | `5` | `<max-replicas>` |
| HTTP concurrency | `50` | `<http-concurrency>` |
| Managed identity | `System-assigned` | `<system-assigned / user-assigned / none>` |
| ACR pull auth | `Managed identity with AcrPull` | `<managed-identity / admin-enabled-lab / service-principal>` |
| Secrets needed | `APPSETTING_VALUE` | `<secret-list>` |
| Environment variables | `ENVIRONMENT=lab` | `<env-vars>` |
| Revision mode | `Single` | `<Single / Multiple>` |
| Evidence path | `.\evidence\05-acr-aci-containerapps` | `<evidence-path>` |
| Rollback posture | `Delete lab RG or remove ACI/app/registry` | `<rollback-plan>` |

# Deploy_ACR_ACI_Azure_Container_Apps_Sizing_And_Scaling_Configuration_Checklist

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm Azure context | Admin Workstation | `az account show -o table` | Correct tenant and subscription are active |
| 2 | Set subscription | Admin Workstation | `az account set --subscription "<subscription-id>"` | CLI targets correct subscription |
| 3 | Register container providers | Admin Workstation | `az provider register --namespace Microsoft.ContainerRegistry && az provider register --namespace Microsoft.ContainerInstance && az provider register --namespace Microsoft.App && az provider register --namespace Microsoft.OperationalInsights` | Required providers register |
| 4 | Confirm provider states | Admin Workstation | `az provider show --namespace Microsoft.App --query registrationState -o tsv` | Provider shows `Registered` |
| 5 | Create resource group | Admin Workstation | `az group create --name "<resource-group-name>" --location "<azure-region>"` | Resource group exists |
| 6 | Create ACR | Admin Workstation | `az acr create --resource-group "<resource-group-name>" --name "<acr-name>" --sku "<acr-sku>" --location "<azure-region>"` | Registry is created |
| 7 | Confirm ACR login server | Admin Workstation | `az acr show --name "<acr-name>" --query loginServer -o tsv` | Login server is returned |
| 8 | Build image in ACR | Admin Workstation | `az acr build --registry "<acr-name>" --image "<repository-name>:<image-tag>" "<build-context>"` | Image builds and pushes to ACR |
| 9 | List ACR repositories | Admin Workstation | `az acr repository list --name "<acr-name>" -o table` | Repository is visible |
| 10 | List image tags | Admin Workstation | `az acr repository show-tags --name "<acr-name>" --repository "<repository-name>" -o table` | Image tag is visible |
| 11 | Capture image digest | Admin Workstation | `az acr repository show-manifests --name "<acr-name>" --repository "<repository-name>" -o table` | Digest and timestamp are visible |
| 12 | Deploy ACI with public test image first | Admin Workstation | `az container create --resource-group "<resource-group-name>" --name "<aci-name>" --image mcr.microsoft.com/azuredocs/aci-helloworld --cpu 1 --memory 1 --ports 80 --dns-name-label "<dns-label>" --location "<azure-region>"` | ACI container group starts |
| 13 | Verify ACI state | Admin Workstation | `az container show --resource-group "<resource-group-name>" --name "<aci-name>" --query "{state:instanceView.state,ip:ipAddress.ip,fqdn:ipAddress.fqdn}" -o table` | ACI shows running and endpoint |
| 14 | View ACI logs | Admin Workstation | `az container logs --resource-group "<resource-group-name>" --name "<aci-name>"` | Container logs are returned |
| 15 | Delete public-image ACI test if moving to ACR image | Admin Workstation | `az container delete --resource-group "<resource-group-name>" --name "<aci-name>" --yes` | ACI group is removed |
| 16 | Deploy ACI from ACR image using approved auth path | Admin Workstation | `az container create --resource-group "<resource-group-name>" --name "<aci-name>" --image "<full-image-name>" --cpu "<aci-cpu>" --memory "<aci-memory-gb>" --ports "<aci-port>" --restart-policy "<restart-policy>"` | ACI pulls private image when auth is configured |
| 17 | Add Container Apps extension | Admin Workstation | `az extension add --name containerapp --upgrade` | Container Apps CLI extension is installed |
| 18 | Create Log Analytics workspace | Admin Workstation | `az monitor log-analytics workspace create --resource-group "<resource-group-name>" --workspace-name "<workspace-name>" --location "<azure-region>"` | Workspace exists |
| 19 | Get workspace customer ID | Admin Workstation | `az monitor log-analytics workspace show --resource-group "<resource-group-name>" --workspace-name "<workspace-name>" --query customerId -o tsv` | Workspace customer ID is returned |
| 20 | Get workspace shared key | Admin Workstation | `az monitor log-analytics workspace get-shared-keys --resource-group "<resource-group-name>" --workspace-name "<workspace-name>" --query primarySharedKey -o tsv` | Workspace key is returned |
| 21 | Create Container Apps environment | Admin Workstation | `az containerapp env create --name "<container-app-env-name>" --resource-group "<resource-group-name>" --location "<azure-region>" --logs-workspace-id "<workspace-customer-id>" --logs-workspace-key "<workspace-key>"` | Managed environment is created |
| 22 | Create container app from public image | Admin Workstation | `az containerapp create --name "<container-app-name>" --resource-group "<resource-group-name>" --environment "<container-app-env-name>" --image mcr.microsoft.com/azuredocs/containerapps-helloworld:latest --target-port 80 --ingress external --min-replicas 1 --max-replicas 5 --cpu 0.5 --memory 1.0Gi` | Container app is created |
| 23 | Verify container app FQDN | Admin Workstation | `az containerapp show --name "<container-app-name>" --resource-group "<resource-group-name>" --query properties.configuration.ingress.fqdn -o tsv` | App endpoint is returned |
| 24 | Enable managed identity on container app | Admin Workstation | `az containerapp identity assign --name "<container-app-name>" --resource-group "<resource-group-name>" --system-assigned` | System-assigned identity exists |
| 25 | Assign AcrPull to container app identity | Admin Workstation | `az role assignment create --assignee "<principal-id>" --role AcrPull --scope "<acr-resource-id>"` | Container app identity can pull from ACR |
| 26 | Configure container app registry auth with identity | Admin Workstation | `az containerapp registry set --name "<container-app-name>" --resource-group "<resource-group-name>" --server "<acr-login-server>" --identity system` | App can authenticate to ACR |
| 27 | Update container app to ACR image | Admin Workstation | `az containerapp update --name "<container-app-name>" --resource-group "<resource-group-name>" --image "<full-image-name>"` | New revision is created from ACR image |
| 28 | Configure container app resources | Admin Workstation | `az containerapp update --name "<container-app-name>" --resource-group "<resource-group-name>" --cpu "<container-app-cpu>" --memory "<container-app-memory>"` | CPU and memory update is applied |
| 29 | Configure replica bounds | Admin Workstation | `az containerapp update --name "<container-app-name>" --resource-group "<resource-group-name>" --min-replicas "<min-replicas>" --max-replicas "<max-replicas>"` | Replica bounds are set |
| 30 | Add HTTP scale rule at create or update time | Admin Workstation | `az containerapp update --name "<container-app-name>" --resource-group "<resource-group-name>" --scale-rule-name http-concurrency --scale-rule-http-concurrency "<http-concurrency>"` | HTTP scale rule is configured when supported |
| 31 | Review revisions | Admin Workstation | `az containerapp revision list --name "<container-app-name>" --resource-group "<resource-group-name>" -o table` | Revisions are visible |
| 32 | View app logs | Admin Workstation | `az containerapp logs show --name "<container-app-name>" --resource-group "<resource-group-name>" --follow false` | Container logs are returned |
| 33 | View replica status | Admin Workstation | `az containerapp replica list --name "<container-app-name>" --resource-group "<resource-group-name>" --revision "<revision-name>" -o table` | Replica status is visible |
| 34 | Capture final ACR evidence | Admin Workstation | `az acr show --name "<acr-name>" -o json > .\evidence\acr-final.json` | Registry evidence is saved |
| 35 | Capture final ACI evidence | Admin Workstation | `az container show --resource-group "<resource-group-name>" --name "<aci-name>" -o json > .\evidence\aci-final.json` | ACI evidence is saved |
| 36 | Capture final Container Apps evidence | Admin Workstation | `az containerapp show --name "<container-app-name>" --resource-group "<resource-group-name>" -o json > .\evidence\containerapp-final.json` | App evidence is saved |
| 37 | Document image, runtime, ingress, identity, scaling, and rollback | Operator | `Record final state` | Workbook evidence is complete |

# Deploy_ACR_ACI_Azure_Container_Apps_Sizing_And_Scaling_Portal_Skeleton

```text
Purpose:
Deploy and validate ACR, ACI, and Azure Container Apps through the Azure portal.

ACR portal path:
Azure portal
> Container registries
> Create

ACR basics:
1. Subscription: <subscription-name>
2. Resource group: <resource-group-name>
3. Registry name: <acr-name>
4. Location: <azure-region>
5. SKU:
   - Basic for lab and low-volume dev/test
   - Standard for more throughput and storage
   - Premium for private endpoint, geo-replication, and advanced scenarios
6. Networking:
   - Public endpoint for basic lab
   - Private endpoint for production-style private registry path
7. Admin user:
   - Disabled preferred
   - Enable only for short lab testing if explicitly needed
8. Tags:
   - Environment: Lab
   - Workload: Compute
   - ManagedBy: Portal

ACR validation:
1. Open registry.
2. Review Repositories.
3. Confirm image repository and tags.
4. Confirm Access keys admin state.
5. Confirm IAM if identities need AcrPull.

ACI portal path:
Azure portal
> Container instances
> Create

ACI basics:
1. Resource group: <resource-group-name>
2. Container name: <aci-name>
3. Region: <azure-region>
4. Image source:
   - Quickstart image for first runtime test
   - Azure Container Registry for private image test
5. Image: <image-name>
6. OS type: Linux or Windows
7. Size:
   - CPU: <aci-cpu>
   - Memory: <aci-memory-gb>
8. Restart policy:
   - Always for service-like test
   - OnFailure for retry jobs
   - Never for one-shot jobs
9. Networking:
   - Public for simple lab endpoint
   - Private VNet when testing private container patterns
10. DNS label: <dns-label>
11. Ports: <aci-port>

ACI validation:
1. Confirm container group state is Running or Succeeded.
2. Open FQDN if public.
3. Review Logs.
4. Review Events.
5. Confirm restart count.

Container Apps portal path:
Azure portal
> Container Apps
> Create

Container Apps basics:
1. Resource group: <resource-group-name>
2. Container app name: <container-app-name>
3. Region: <azure-region>
4. Container Apps environment:
   - Create new: <container-app-env-name>
   - Existing: select existing environment
5. Log Analytics workspace: <workspace-name>
6. Image source:
   - Quickstart/public image for first validation
   - ACR image for private registry workflow
7. Container name: <container-name>
8. Image and tag: <full-image-name>
9. CPU and memory:
   - CPU: <container-app-cpu>
   - Memory: <container-app-memory>
10. Ingress:
   - External for public test endpoint
   - Internal for private environment-only endpoint
   - Disabled for worker apps
11. Target port: <target-port>
12. Min replicas: <min-replicas>
13. Max replicas: <max-replicas>
14. Scale rule:
   - HTTP concurrency for web apps
   - Event rules for queue or broker workloads
15. Identity:
   - Enable system-assigned or user-assigned identity when pulling private images or accessing Azure services

Container Apps validation:
1. Open container app.
2. Confirm Provisioning state.
3. Confirm Application Url.
4. Review Revisions.
5. Review Replicas.
6. Review Logs.
7. Review Scale settings.
8. Confirm image pull succeeded.
```

# Deploy_ACR_ACI_Azure_Container_Apps_Sizing_And_Scaling_AzureCLI_Skeleton

```bash
# Run from Azure Cloud Shell or local Azure CLI.
# Purpose: deploy ACR, build image, deploy ACI, deploy Azure Container Apps, configure identity, ingress, and scaling.

# -----------------------------
# Variables
# -----------------------------

export SUBSCRIPTION_ID="<subscription-id>"
export LOCATION="eastus"
export RESOURCE_GROUP="rg-compute-lab-01"

export ACR_NAME="acrcomputelab01"
export ACR_SKU="Basic"
export REPOSITORY_NAME="webapp"
export IMAGE_TAG="v1"

export BUILD_CONTEXT="./src"

export ACI_NAME="aci-web-lab-01"
export ACI_CPU="1"
export ACI_MEMORY="1.5"
export ACI_PORT="80"
export ACI_DNS_LABEL="aci-web-lab-01"
export ACI_RESTART_POLICY="Always"

export LAW_NAME="law-compute-lab-01"
export ACA_ENV_NAME="cae-compute-lab-01"
export ACA_APP_NAME="ca-web-lab-01"
export ACA_CONTAINER_NAME="web"
export ACA_TARGET_PORT="80"
export ACA_CPU="0.5"
export ACA_MEMORY="1.0Gi"
export ACA_MIN_REPLICAS="1"
export ACA_MAX_REPLICAS="5"
export ACA_HTTP_CONCURRENCY="50"

export EVIDENCE_PATH="./evidence/05-acr-aci-containerapps"

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
# Providers and extension
# -----------------------------

az extension add --name containerapp --upgrade

for provider in \
  Microsoft.ContainerRegistry \
  Microsoft.ContainerInstance \
  Microsoft.App \
  Microsoft.OperationalInsights \
  Microsoft.Insights \
  Microsoft.ManagedIdentity \
  Microsoft.Resources
do
  az provider register --namespace "$provider"
done

for provider in \
  Microsoft.ContainerRegistry \
  Microsoft.ContainerInstance \
  Microsoft.App \
  Microsoft.OperationalInsights \
  Microsoft.Insights \
  Microsoft.ManagedIdentity \
  Microsoft.Resources
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
# ACR create and build
# -----------------------------

az acr create \
  --resource-group "$RESOURCE_GROUP" \
  --name "$ACR_NAME" \
  --sku "$ACR_SKU" \
  --location "$LOCATION" |
  tee "$EVIDENCE_PATH/acr-create.json"

ACR_LOGIN_SERVER=$(az acr show \
  --name "$ACR_NAME" \
  --query loginServer \
  --output tsv)

export FULL_IMAGE_NAME="$ACR_LOGIN_SERVER/$REPOSITORY_NAME:$IMAGE_TAG"

echo "$FULL_IMAGE_NAME" | tee "$EVIDENCE_PATH/full-image-name.txt"

# Build in ACR. This avoids needing local Docker on the admin workstation.
az acr build \
  --registry "$ACR_NAME" \
  --image "$REPOSITORY_NAME:$IMAGE_TAG" \
  "$BUILD_CONTEXT" |
  tee "$EVIDENCE_PATH/acr-build.txt"

az acr repository list \
  --name "$ACR_NAME" \
  --output table |
  tee "$EVIDENCE_PATH/acr-repositories.txt"

az acr repository show-tags \
  --name "$ACR_NAME" \
  --repository "$REPOSITORY_NAME" \
  --output table |
  tee "$EVIDENCE_PATH/acr-tags.txt"

az acr repository show-manifests \
  --name "$ACR_NAME" \
  --repository "$REPOSITORY_NAME" \
  --output table |
  tee "$EVIDENCE_PATH/acr-manifests.txt"

# -----------------------------
# ACI public quick test
# -----------------------------

az container create \
  --resource-group "$RESOURCE_GROUP" \
  --name "$ACI_NAME" \
  --location "$LOCATION" \
  --image mcr.microsoft.com/azuredocs/aci-helloworld \
  --cpu "$ACI_CPU" \
  --memory "$ACI_MEMORY" \
  --ports "$ACI_PORT" \
  --dns-name-label "$ACI_DNS_LABEL" \
  --restart-policy "$ACI_RESTART_POLICY" |
  tee "$EVIDENCE_PATH/aci-public-create.json"

az container show \
  --resource-group "$RESOURCE_GROUP" \
  --name "$ACI_NAME" \
  --query "{name:name,state:instanceView.state,ip:ipAddress.ip,fqdn:ipAddress.fqdn}" \
  --output table |
  tee "$EVIDENCE_PATH/aci-public-show.txt"

az container logs \
  --resource-group "$RESOURCE_GROUP" \
  --name "$ACI_NAME" |
  tee "$EVIDENCE_PATH/aci-public-logs.txt"

# Remove public test container group before private image test.
az container delete \
  --resource-group "$RESOURCE_GROUP" \
  --name "$ACI_NAME" \
  --yes

# -----------------------------
# Optional ACI from ACR, quick lab method
# -----------------------------
# Preferred production-style approach is managed identity where supported.
# Admin-enabled ACR credentials are not preferred, but useful for isolated quick labs.

# az acr update \
#   --name "$ACR_NAME" \
#   --admin-enabled true
#
# ACR_USERNAME=$(az acr credential show --name "$ACR_NAME" --query username -o tsv)
# ACR_PASSWORD=$(az acr credential show --name "$ACR_NAME" --query "passwords[0].value" -o tsv)
#
# az container create \
#   --resource-group "$RESOURCE_GROUP" \
#   --name "$ACI_NAME" \
#   --location "$LOCATION" \
#   --image "$FULL_IMAGE_NAME" \
#   --registry-login-server "$ACR_LOGIN_SERVER" \
#   --registry-username "$ACR_USERNAME" \
#   --registry-password "$ACR_PASSWORD" \
#   --cpu "$ACI_CPU" \
#   --memory "$ACI_MEMORY" \
#   --ports "$ACI_PORT" \
#   --dns-name-label "$ACI_DNS_LABEL" \
#   --restart-policy "$ACI_RESTART_POLICY" |
#   tee "$EVIDENCE_PATH/aci-acr-create.json"

# -----------------------------
# Log Analytics workspace for Container Apps
# -----------------------------

az monitor log-analytics workspace create \
  --resource-group "$RESOURCE_GROUP" \
  --workspace-name "$LAW_NAME" \
  --location "$LOCATION" |
  tee "$EVIDENCE_PATH/law-create.json"

LAW_CUSTOMER_ID=$(az monitor log-analytics workspace show \
  --resource-group "$RESOURCE_GROUP" \
  --workspace-name "$LAW_NAME" \
  --query customerId \
  --output tsv)

LAW_SHARED_KEY=$(az monitor log-analytics workspace get-shared-keys \
  --resource-group "$RESOURCE_GROUP" \
  --workspace-name "$LAW_NAME" \
  --query primarySharedKey \
  --output tsv)

# -----------------------------
# Container Apps environment
# -----------------------------

az containerapp env create \
  --name "$ACA_ENV_NAME" \
  --resource-group "$RESOURCE_GROUP" \
  --location "$LOCATION" \
  --logs-workspace-id "$LAW_CUSTOMER_ID" \
  --logs-workspace-key "$LAW_SHARED_KEY" |
  tee "$EVIDENCE_PATH/aca-env-create.json"

az containerapp env show \
  --name "$ACA_ENV_NAME" \
  --resource-group "$RESOURCE_GROUP" \
  --output json |
  tee "$EVIDENCE_PATH/aca-env-show.json"

# -----------------------------
# Container App from public image first
# -----------------------------

az containerapp create \
  --name "$ACA_APP_NAME" \
  --resource-group "$RESOURCE_GROUP" \
  --environment "$ACA_ENV_NAME" \
  --image mcr.microsoft.com/azuredocs/containerapps-helloworld:latest \
  --target-port "$ACA_TARGET_PORT" \
  --ingress external \
  --min-replicas "$ACA_MIN_REPLICAS" \
  --max-replicas "$ACA_MAX_REPLICAS" \
  --cpu "$ACA_CPU" \
  --memory "$ACA_MEMORY" \
  --env-vars ENVIRONMENT=lab APPROLE=web |
  tee "$EVIDENCE_PATH/aca-public-create.json"

az containerapp show \
  --name "$ACA_APP_NAME" \
  --resource-group "$RESOURCE_GROUP" \
  --query "{name:name,provisioningState:properties.provisioningState,fqdn:properties.configuration.ingress.fqdn}" \
  --output table |
  tee "$EVIDENCE_PATH/aca-public-show.txt"

# -----------------------------
# Configure identity and ACR pull
# -----------------------------

az containerapp identity assign \
  --name "$ACA_APP_NAME" \
  --resource-group "$RESOURCE_GROUP" \
  --system-assigned |
  tee "$EVIDENCE_PATH/aca-identity-assign.json"

ACA_PRINCIPAL_ID=$(az containerapp identity show \
  --name "$ACA_APP_NAME" \
  --resource-group "$RESOURCE_GROUP" \
  --query principalId \
  --output tsv)

ACR_ID=$(az acr show \
  --name "$ACR_NAME" \
  --query id \
  --output tsv)

az role assignment create \
  --assignee "$ACA_PRINCIPAL_ID" \
  --role AcrPull \
  --scope "$ACR_ID" |
  tee "$EVIDENCE_PATH/acrpull-role-assignment.json"

az containerapp registry set \
  --name "$ACA_APP_NAME" \
  --resource-group "$RESOURCE_GROUP" \
  --server "$ACR_LOGIN_SERVER" \
  --identity system |
  tee "$EVIDENCE_PATH/aca-registry-set.json"

# -----------------------------
# Update app to private ACR image
# -----------------------------

az containerapp update \
  --name "$ACA_APP_NAME" \
  --resource-group "$RESOURCE_GROUP" \
  --image "$FULL_IMAGE_NAME" \
  --cpu "$ACA_CPU" \
  --memory "$ACA_MEMORY" \
  --min-replicas "$ACA_MIN_REPLICAS" \
  --max-replicas "$ACA_MAX_REPLICAS" |
  tee "$EVIDENCE_PATH/aca-update-acr-image.json"

# Optional HTTP scale rule. Syntax may vary by installed containerapp extension version.
az containerapp update \
  --name "$ACA_APP_NAME" \
  --resource-group "$RESOURCE_GROUP" \
  --scale-rule-name http-concurrency \
  --scale-rule-http-concurrency "$ACA_HTTP_CONCURRENCY" |
  tee "$EVIDENCE_PATH/aca-http-scale-rule.json"

# -----------------------------
# Container Apps validation
# -----------------------------

az containerapp revision list \
  --name "$ACA_APP_NAME" \
  --resource-group "$RESOURCE_GROUP" \
  --output table |
  tee "$EVIDENCE_PATH/aca-revisions.txt"

ACTIVE_REVISION=$(az containerapp revision list \
  --name "$ACA_APP_NAME" \
  --resource-group "$RESOURCE_GROUP" \
  --query "[?properties.active==\`true\`].name | [0]" \
  --output tsv)

echo "$ACTIVE_REVISION" | tee "$EVIDENCE_PATH/aca-active-revision.txt"

az containerapp replica list \
  --name "$ACA_APP_NAME" \
  --resource-group "$RESOURCE_GROUP" \
  --revision "$ACTIVE_REVISION" \
  --output table |
  tee "$EVIDENCE_PATH/aca-replicas.txt"

az containerapp logs show \
  --name "$ACA_APP_NAME" \
  --resource-group "$RESOURCE_GROUP" \
  --follow false |
  tee "$EVIDENCE_PATH/aca-logs.txt"

az containerapp show \
  --name "$ACA_APP_NAME" \
  --resource-group "$RESOURCE_GROUP" \
  --output json |
  tee "$EVIDENCE_PATH/aca-final.json"

# -----------------------------
# Final inventory
# -----------------------------

az resource list \
  --resource-group "$RESOURCE_GROUP" \
  --output table |
  tee "$EVIDENCE_PATH/resource-list-final.txt"
```

# Deploy_ACR_ACI_Azure_Container_Apps_Sizing_And_Scaling_PowerShell_Skeleton

```powershell
# Run from PowerShell with Az module installed.
# Purpose: create ACR, validate registry, and inspect Azure container hosting resources.
# Note: Azure CLI is usually cleaner for Container Apps operational labs.

# -----------------------------
# Variables
# -----------------------------

$SubscriptionId = "<subscription-id>"
$Location = "eastus"
$ResourceGroupName = "rg-compute-lab-01"

$AcrName = "acrcomputelab01"
$AcrSku = "Basic"
$RepositoryName = "webapp"
$ImageTag = "v1"

$AciName = "aci-web-lab-01"
$LawName = "law-compute-lab-01"
$AcaEnvName = "cae-compute-lab-01"
$AcaAppName = "ca-web-lab-01"

$EvidencePath = ".\evidence\05-acr-aci-containerapps"

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
# ACR
# -----------------------------

New-AzContainerRegistry `
  -ResourceGroupName $ResourceGroupName `
  -Name $AcrName `
  -Sku $AcrSku `
  -Location $Location `
  -EnableAdminUser:$false |
  Tee-Object (Join-Path $EvidencePath "acr-create.txt")

Get-AzContainerRegistry `
  -ResourceGroupName $ResourceGroupName `
  -Name $AcrName |
  Tee-Object (Join-Path $EvidencePath "acr-show.txt")

# -----------------------------
# Use Azure CLI from PowerShell for build and Container Apps operations
# -----------------------------

az acr build `
  --registry $AcrName `
  --image "$RepositoryName`:$ImageTag" `
  "./src" |
  Tee-Object (Join-Path $EvidencePath "acr-build.txt")

az acr repository list `
  --name $AcrName `
  --output table |
  Tee-Object (Join-Path $EvidencePath "acr-repositories.txt")

az acr repository show-tags `
  --name $AcrName `
  --repository $RepositoryName `
  --output table |
  Tee-Object (Join-Path $EvidencePath "acr-tags.txt")

# -----------------------------
# ACI public image quick validation
# -----------------------------

az container create `
  --resource-group $ResourceGroupName `
  --name $AciName `
  --location $Location `
  --image "mcr.microsoft.com/azuredocs/aci-helloworld" `
  --cpu 1 `
  --memory 1 `
  --ports 80 `
  --dns-name-label $AciName `
  --restart-policy Always |
  Tee-Object (Join-Path $EvidencePath "aci-create.txt")

az container show `
  --resource-group $ResourceGroupName `
  --name $AciName `
  --query "{name:name,state:instanceView.state,ip:ipAddress.ip,fqdn:ipAddress.fqdn}" `
  --output table |
  Tee-Object (Join-Path $EvidencePath "aci-show.txt")

az container logs `
  --resource-group $ResourceGroupName `
  --name $AciName |
  Tee-Object (Join-Path $EvidencePath "aci-logs.txt")

# -----------------------------
# Container Apps validation through Azure CLI
# -----------------------------

az extension add --name containerapp --upgrade

az monitor log-analytics workspace create `
  --resource-group $ResourceGroupName `
  --workspace-name $LawName `
  --location $Location |
  Tee-Object (Join-Path $EvidencePath "law-create.txt")

$LawCustomerId = az monitor log-analytics workspace show `
  --resource-group $ResourceGroupName `
  --workspace-name $LawName `
  --query customerId `
  --output tsv

$LawSharedKey = az monitor log-analytics workspace get-shared-keys `
  --resource-group $ResourceGroupName `
  --workspace-name $LawName `
  --query primarySharedKey `
  --output tsv

az containerapp env create `
  --name $AcaEnvName `
  --resource-group $ResourceGroupName `
  --location $Location `
  --logs-workspace-id $LawCustomerId `
  --logs-workspace-key $LawSharedKey |
  Tee-Object (Join-Path $EvidencePath "aca-env-create.txt")

az containerapp create `
  --name $AcaAppName `
  --resource-group $ResourceGroupName `
  --environment $AcaEnvName `
  --image "mcr.microsoft.com/azuredocs/containerapps-helloworld:latest" `
  --target-port 80 `
  --ingress external `
  --min-replicas 1 `
  --max-replicas 5 `
  --cpu 0.5 `
  --memory "1.0Gi" |
  Tee-Object (Join-Path $EvidencePath "aca-create.txt")

az containerapp show `
  --name $AcaAppName `
  --resource-group $ResourceGroupName `
  --output json |
  Tee-Object (Join-Path $EvidencePath "aca-show.json")
```

# Deploy_ACR_ACI_Azure_Container_Apps_Sizing_And_Scaling_Bicep_Skeleton

```bicep
// File: .\infra\05-acr-aci-containerapps\main.bicep
// Purpose: Deploy ACR, Log Analytics, Container Apps environment, and a baseline container app.
// ACI is usually simpler through CLI for lab validation, but can also be represented in ARM/Bicep.

targetScope = 'resourceGroup'

@description('Azure region.')
param location string = resourceGroup().location

@description('ACR name. Must be globally unique, lowercase, and alphanumeric.')
param acrName string = 'acrcomputelab01'

@description('ACR SKU.')
@allowed([
  'Basic'
  'Standard'
  'Premium'
])
param acrSku string = 'Basic'

@description('Log Analytics workspace name.')
param workspaceName string = 'law-compute-lab-01'

@description('Container Apps environment name.')
param containerAppsEnvName string = 'cae-compute-lab-01'

@description('Container app name.')
param containerAppName string = 'ca-web-lab-01'

@description('Container image.')
param containerImage string = 'mcr.microsoft.com/azuredocs/containerapps-helloworld:latest'

@description('Target container port.')
param targetPort int = 80

@description('Minimum replicas.')
param minReplicas int = 1

@description('Maximum replicas.')
param maxReplicas int = 5

@description('Container CPU.')
param containerCpu string = '0.5'

@description('Container memory.')
param containerMemory string = '1.0Gi'

var tags = {
  Environment: 'Lab'
  Workload: 'Compute'
  ManagedBy: 'Bicep'
  Workbook: '05_Deploy_ACR_ACI_Azure_Container_Apps_Sizing_And_Scaling'
}

resource acr 'Microsoft.ContainerRegistry/registries@2023-07-01' = {
  name: acrName
  location: location
  tags: tags
  sku: {
    name: acrSku
  }
  properties: {
    adminUserEnabled: false
    publicNetworkAccess: 'Enabled'
  }
}

resource workspace 'Microsoft.OperationalInsights/workspaces@2022-10-01' = {
  name: workspaceName
  location: location
  tags: tags
  properties: {
    sku: {
      name: 'PerGB2018'
    }
    retentionInDays: 30
  }
}

resource containerAppsEnv 'Microsoft.App/managedEnvironments@2024-03-01' = {
  name: containerAppsEnvName
  location: location
  tags: tags
  properties: {
    appLogsConfiguration: {
      destination: 'log-analytics'
      logAnalyticsConfiguration: {
        customerId: workspace.properties.customerId
        sharedKey: workspace.listKeys().primarySharedKey
      }
    }
  }
}

resource containerApp 'Microsoft.App/containerApps@2024-03-01' = {
  name: containerAppName
  location: location
  tags: tags
  properties: {
    managedEnvironmentId: containerAppsEnv.id
    configuration: {
      activeRevisionsMode: 'Single'
      ingress: {
        external: true
        targetPort: targetPort
        transport: 'auto'
        allowInsecure: false
      }
    }
    template: {
      containers: [
        {
          name: 'web'
          image: containerImage
          env: [
            {
              name: 'ENVIRONMENT'
              value: 'lab'
            }
            {
              name: 'APPROLE'
              value: 'web'
            }
          ]
          resources: {
            cpu: json(containerCpu)
            memory: containerMemory
          }
        }
      ]
      scale: {
        minReplicas: minReplicas
        maxReplicas: maxReplicas
        rules: [
          {
            name: 'http-concurrency'
            http: {
              metadata: {
                concurrentRequests: '50'
              }
            }
          }
        ]
      }
    }
  }
}

output acrLoginServer string = acr.properties.loginServer
output containerAppsEnvironmentId string = containerAppsEnv.id
output containerAppId string = containerApp.id
output containerAppFqdn string = containerApp.properties.configuration.ingress.fqdn
```

# Deploy_ACR_ACI_Azure_Container_Apps_Sizing_And_Scaling_Bicep_Parameters_Skeleton

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "location": {
      "value": "eastus"
    },
    "acrName": {
      "value": "acrcomputelab01"
    },
    "acrSku": {
      "value": "Basic"
    },
    "workspaceName": {
      "value": "law-compute-lab-01"
    },
    "containerAppsEnvName": {
      "value": "cae-compute-lab-01"
    },
    "containerAppName": {
      "value": "ca-web-lab-01"
    },
    "containerImage": {
      "value": "mcr.microsoft.com/azuredocs/containerapps-helloworld:latest"
    },
    "targetPort": {
      "value": 80
    },
    "minReplicas": {
      "value": 1
    },
    "maxReplicas": {
      "value": 5
    },
    "containerCpu": {
      "value": "0.5"
    },
    "containerMemory": {
      "value": "1.0Gi"
    }
  }
}
```

# Deploy_ACR_ACI_Azure_Container_Apps_Sizing_And_Scaling_ACR_Build_Push_Pull_Skeleton

```text
Purpose:
Create ACR, build image, tag image, validate repository, and prepare pull identity.

ACR naming:
1. Must be globally unique.
2. Use lowercase letters and numbers.
3. Example:
   acrcomputelab01

SKU selection:
1. Basic:
   - Lab
   - Dev/test
   - Low throughput
2. Standard:
   - More storage and throughput
   - Common non-production baseline
3. Premium:
   - Private endpoints
   - Geo-replication
   - Advanced network controls

Build workflow:
1. Store Dockerfile under:
   ./src/Dockerfile

2. Build in ACR:
   az acr build \
     --registry <acr-name> \
     --image <repository-name>:<tag> \
     ./src

3. List repositories:
   az acr repository list --name <acr-name> -o table

4. List tags:
   az acr repository show-tags \
     --name <acr-name> \
     --repository <repository-name> \
     -o table

5. Capture digest:
   az acr repository show-manifests \
     --name <acr-name> \
     --repository <repository-name> \
     -o table

Image promotion model:
1. Build unique tag:
   webapp:2026.06.15.1

2. Test in ACI or Container Apps.

3. Promote tag:
   webapp:dev
   webapp:test
   webapp:prod

4. Prefer digest pinning for strict release control.

Authentication:
1. Preferred:
   - Managed identity with AcrPull

2. Acceptable in automation:
   - Service principal scoped to AcrPull

3. Lab-only:
   - ACR admin user temporarily enabled

Do not:
1. Hardcode registry passwords in workbook files.
2. Commit image pull secrets to Git.
3. Use broad Owner role when AcrPull is enough.
```

# Deploy_ACR_ACI_Azure_Container_Apps_Sizing_And_Scaling_ACI_Runtime_Skeleton

```text
Purpose:
Run quick containers with Azure Container Instances.

Best uses:
1. Quick container test
2. Short-running job
3. Simple single-container app
4. One-off troubleshooting container
5. No orchestrator requirement

Not best for:
1. Complex multi-service platform
2. Advanced rolling deployments
3. Event-driven autoscale
4. Revision traffic splitting
5. Production microservice hosting where Container Apps fits better

ACI public image test:
az container create \
  --resource-group <rg> \
  --name <aci-name> \
  --image mcr.microsoft.com/azuredocs/aci-helloworld \
  --cpu 1 \
  --memory 1 \
  --ports 80 \
  --dns-name-label <dns-label> \
  --restart-policy Always

Validate:
1. State:
   az container show -g <rg> -n <aci-name> --query "instanceView.state" -o tsv

2. FQDN:
   az container show -g <rg> -n <aci-name> --query "ipAddress.fqdn" -o tsv

3. Logs:
   az container logs -g <rg> -n <aci-name>

4. Events:
   az container attach -g <rg> -n <aci-name>

Restart policy:
1. Always:
   - Service-like test container

2. OnFailure:
   - Batch jobs that should retry

3. Never:
   - One-shot job where exit code matters

Sizing:
1. Start small:
   - 1 CPU
   - 1 GB to 1.5 GB memory

2. Increase based on:
   - App startup failure
   - Out-of-memory behavior
   - CPU throttling
   - Runtime performance
```

# Deploy_ACR_ACI_Azure_Container_Apps_Sizing_And_Scaling_Container_Apps_Runtime_Skeleton

```text
Purpose:
Deploy scalable containerized web apps and workers with Azure Container Apps.

Best uses:
1. HTTP APIs
2. Microservices
3. Containerized web apps
4. Event-driven workers
5. Jobs
6. Scale-to-zero workloads
7. Apps that need revisions and traffic splitting

Core deployment objects:
1. Log Analytics workspace
2. Container Apps environment
3. Container app
4. Revision
5. Replica
6. Scale rule
7. Ingress config
8. Secrets
9. Managed identity
10. Registry auth

Public image first validation:
az containerapp create \
  --name <app-name> \
  --resource-group <rg> \
  --environment <env-name> \
  --image mcr.microsoft.com/azuredocs/containerapps-helloworld:latest \
  --target-port 80 \
  --ingress external \
  --min-replicas 1 \
  --max-replicas 5 \
  --cpu 0.5 \
  --memory 1.0Gi

Private ACR image flow:
1. Assign managed identity:
   az containerapp identity assign \
     --name <app-name> \
     --resource-group <rg> \
     --system-assigned

2. Get principal ID:
   az containerapp identity show \
     --name <app-name> \
     --resource-group <rg> \
     --query principalId -o tsv

3. Assign AcrPull:
   az role assignment create \
     --assignee <principal-id> \
     --role AcrPull \
     --scope <acr-resource-id>

4. Set registry auth:
   az containerapp registry set \
     --name <app-name> \
     --resource-group <rg> \
     --server <acr-login-server> \
     --identity system

5. Update image:
   az containerapp update \
     --name <app-name> \
     --resource-group <rg> \
     --image <acr-login-server>/<repository>:<tag>

Revision validation:
1. List revisions:
   az containerapp revision list -n <app-name> -g <rg> -o table

2. List replicas:
   az containerapp replica list -n <app-name> -g <rg> --revision <revision-name> -o table

3. Show logs:
   az containerapp logs show -n <app-name> -g <rg> --follow false

Ingress validation:
1. Confirm target port matches the container listener.
2. Confirm external or internal setting is correct.
3. Confirm FQDN returns expected app.
4. Confirm logs show requests.
```

# Deploy_ACR_ACI_Azure_Container_Apps_Sizing_And_Scaling_Sizing_And_Scaling_Skeleton

```text
Purpose:
Choose CPU, memory, replica bounds, and scale rules.

ACI sizing:
1. CPU:
   - Start with 1 CPU for simple lab container.
   - Increase for CPU-bound jobs.

2. Memory:
   - Start with 1 GB or 1.5 GB.
   - Increase if container exits with memory pressure.

3. Restart policy:
   - Always for service-style test.
   - OnFailure for jobs.
   - Never for one-shot validation.

Container Apps sizing:
1. Small web/API lab:
   - CPU: 0.25 to 0.5
   - Memory: 0.5Gi to 1.0Gi
   - Min replicas: 0 or 1
   - Max replicas: 3 to 5

2. Production-like web/API:
   - CPU: 0.5 to 1.0
   - Memory: 1.0Gi to 2.0Gi
   - Min replicas: 2
   - Max replicas: based on load and quota

3. Worker app:
   - Min replicas: 0 if event-driven and scale-to-zero is acceptable
   - Max replicas: based on queue depth, downstream throttling, and cost

HTTP scaling:
1. Use for web apps and APIs.
2. Scale signal is concurrent HTTP requests.
3. Lower concurrency scales out earlier.
4. Higher concurrency keeps fewer replicas.

Example:
- Min replicas: 1
- Max replicas: 5
- HTTP concurrency: 50

Event scaling:
1. Use for queues, service bus, Kafka, Redis, or other event sources.
2. Store auth values as secrets.
3. Avoid scaling faster than downstream services can handle.

Scale-to-zero decision:
1. Good for:
   - Dev/test apps
   - Low traffic tools
   - Event-driven workers
   - Cost optimization

2. Avoid when:
   - Cold start is unacceptable
   - App must always be warm
   - Health probes require constant availability

Operational checks:
1. Watch replica count.
2. Watch logs during scale-out.
3. Watch startup time.
4. Watch failed pulls.
5. Watch target port and health probe failures.
6. Watch 429 or downstream throttling.
```

# Deploy_ACR_ACI_Azure_Container_Apps_Sizing_And_Scaling_Verification_Commands

```bash
# Context
az account show -o table

# Providers
az provider show --namespace Microsoft.ContainerRegistry --query registrationState -o tsv
az provider show --namespace Microsoft.ContainerInstance --query registrationState -o tsv
az provider show --namespace Microsoft.App --query registrationState -o tsv
az provider show --namespace Microsoft.OperationalInsights --query registrationState -o tsv

# Resource group
az group show \
  --name "<resource-group-name>" \
  -o table

# ACR
az acr show \
  --name "<acr-name>" \
  -o table

az acr show \
  --name "<acr-name>" \
  --query "{name:name,loginServer:loginServer,sku:sku.name,adminUserEnabled:adminUserEnabled}" \
  -o table

az acr repository list \
  --name "<acr-name>" \
  -o table

az acr repository show-tags \
  --name "<acr-name>" \
  --repository "<repository-name>" \
  -o table

az acr repository show-manifests \
  --name "<acr-name>" \
  --repository "<repository-name>" \
  -o table

# ACI
az container list \
  --resource-group "<resource-group-name>" \
  -o table

az container show \
  --resource-group "<resource-group-name>" \
  --name "<aci-name>" \
  --query "{name:name,state:instanceView.state,restartCount:containers[0].instanceView.restartCount,ip:ipAddress.ip,fqdn:ipAddress.fqdn}" \
  -o table

az container logs \
  --resource-group "<resource-group-name>" \
  --name "<aci-name>"

# Container Apps environment
az containerapp env show \
  --name "<container-app-env-name>" \
  --resource-group "<resource-group-name>" \
  -o json

az containerapp env list \
  --resource-group "<resource-group-name>" \
  -o table

# Container app
az containerapp show \
  --name "<container-app-name>" \
  --resource-group "<resource-group-name>" \
  -o json

az containerapp show \
  --name "<container-app-name>" \
  --resource-group "<resource-group-name>" \
  --query "{name:name,provisioningState:properties.provisioningState,fqdn:properties.configuration.ingress.fqdn,activeRevisionsMode:properties.configuration.activeRevisionsMode}" \
  -o table

# Identity
az containerapp identity show \
  --name "<container-app-name>" \
  --resource-group "<resource-group-name>" \
  -o json

# Registry configuration
az containerapp registry list \
  --name "<container-app-name>" \
  --resource-group "<resource-group-name>" \
  -o table

# Revisions and replicas
az containerapp revision list \
  --name "<container-app-name>" \
  --resource-group "<resource-group-name>" \
  -o table

az containerapp replica list \
  --name "<container-app-name>" \
  --resource-group "<resource-group-name>" \
  --revision "<revision-name>" \
  -o table

# Logs
az containerapp logs show \
  --name "<container-app-name>" \
  --resource-group "<resource-group-name>" \
  --follow false

# Scaling settings
az containerapp show \
  --name "<container-app-name>" \
  --resource-group "<resource-group-name>" \
  --query "properties.template.scale" \
  -o json

# Ingress settings
az containerapp show \
  --name "<container-app-name>" \
  --resource-group "<resource-group-name>" \
  --query "properties.configuration.ingress" \
  -o json

# Resource inventory
az resource list \
  --resource-group "<resource-group-name>" \
  -o table

# Activity log
az monitor activity-log list \
  --resource-group "<resource-group-name>" \
  --max-events 30 \
  -o table
```

```powershell
# Context
Get-AzContext

# Resource group
Get-AzResourceGroup -Name "<resource-group-name>"

# ACR
Get-AzContainerRegistry `
  -ResourceGroupName "<resource-group-name>" `
  -Name "<acr-name>"

# Azure CLI from PowerShell for Container Apps checks
az containerapp show `
  --name "<container-app-name>" `
  --resource-group "<resource-group-name>" `
  --output table

az containerapp revision list `
  --name "<container-app-name>" `
  --resource-group "<resource-group-name>" `
  --output table

az containerapp logs show `
  --name "<container-app-name>" `
  --resource-group "<resource-group-name>" `
  --follow false
```

# Deploy_ACR_ACI_Azure_Container_Apps_Sizing_And_Scaling_Rollback

| Change | Rollback Command / Action | Risk |
|---|---|---|
| ACR created for lab | `az acr delete --name "<acr-name>" --resource-group "<resource-group-name>" --yes` | Deletes registry and images |
| ACR image pushed | `az acr repository delete --name "<acr-name>" --repository "<repository-name>" --yes` | Deletes repository and tags |
| ACR admin user enabled | `az acr update --name "<acr-name>" --admin-enabled false` | Clients using admin credentials fail |
| AcrPull role assigned | `az role assignment delete --assignee "<principal-id>" --role AcrPull --scope "<acr-resource-id>"` | Image pulls fail for that identity |
| ACI container group created | `az container delete --resource-group "<resource-group-name>" --name "<aci-name>" --yes` | Deletes running container group |
| Container App created | `az containerapp delete --name "<container-app-name>" --resource-group "<resource-group-name>" --yes` | Deletes app and revisions |
| Container Apps environment created | `az containerapp env delete --name "<container-app-env-name>" --resource-group "<resource-group-name>" --yes` | Deletes environment if apps are removed first |
| Log Analytics workspace created | `az monitor log-analytics workspace delete --resource-group "<resource-group-name>" --workspace-name "<workspace-name>" --yes` | Deletes workspace and collected logs after retention behavior |
| Container app updated to bad image | `az containerapp update --name "<container-app-name>" --resource-group "<resource-group-name>" --image "<previous-image>"` | Creates another revision |
| Bad revision active | `az containerapp revision activate --name "<container-app-name>" --resource-group "<resource-group-name>" --revision "<known-good-revision>"` | Traffic must be validated |
| External ingress enabled by mistake | `az containerapp ingress disable --name "<container-app-name>" --resource-group "<resource-group-name>"` | Public endpoint is removed |
| Min replicas too high | `az containerapp update --name "<container-app-name>" --resource-group "<resource-group-name>" --min-replicas "<lower-count>"` | Availability may decrease |
| Max replicas too high | `az containerapp update --name "<container-app-name>" --resource-group "<resource-group-name>" --max-replicas "<lower-count>"` | App may stop scaling above new cap |
| HTTP scale rule too aggressive | Update or remove scale rule | Cost or replica churn continues until fixed |
| Secret added incorrectly | Update secret value or remove unused secret | Existing revisions may need restart or new revision |
| Full lab resource group created | `az group delete --name "<resource-group-name>" --yes --no-wait` | Deletes all resources in the lab RG |

# Deploy_ACR_ACI_Azure_Container_Apps_Sizing_And_Scaling_Failure_Checks

| Symptom | Likely Layer | First Check | Fix |
|---|---|---|---|
| ACR name rejected | Registry naming | Name format and global uniqueness | Use lowercase alphanumeric globally unique name |
| ACR creation fails | Provider or RBAC | Provider state and role assignment | Register provider or fix permissions |
| `az acr build` fails | Dockerfile or build context | Build output | Fix Dockerfile path, context, or base image |
| ACR repository missing | Build did not push | `az acr repository list` | Rebuild with correct `--image` |
| Image tag missing | Wrong tag or repository | `az acr repository show-tags` | Push or build correct tag |
| Image pull fails | Registry auth | ACR role, registry config, image name | Assign AcrPull or fix registry credentials |
| `unauthorized` image pull | Identity lacks AcrPull | Role assignments | Assign AcrPull at ACR scope |
| `manifest unknown` | Wrong image tag | Repository tags | Use valid tag or digest |
| ACI stuck Waiting | Image pull or resource issue | `az container show` events | Fix image, auth, CPU/memory, or port |
| ACI exits immediately | App process exits | `az container logs` | Fix command, entrypoint, env vars, or restart policy |
| ACI public endpoint unreachable | Port or DNS issue | `ipAddress`, ports, logs | Fix exposed port and app listener |
| ACI logs empty | Container never started | Container events | Fix image pull/startup failure |
| Container Apps environment fails | Provider, workspace, region | Activity log and environment state | Register providers or recreate with valid workspace |
| Container app provisioning fails | Image, ingress, CPU/memory, or env issue | `az containerapp show` | Fix image and template settings |
| Container app has no FQDN | Ingress disabled or internal | Ingress config | Enable external ingress if public endpoint is intended |
| App returns 404 or 502 | Target port mismatch or app not listening | Ingress target port and logs | Set correct target port |
| App starts then crashes | Runtime error | Container logs and replica state | Fix image, command, env vars, or secrets |
| Replica stuck pending | Image pull, quota, or environment issue | Replica list and logs | Fix pull auth, quota, or resource request |
| Scale rule not firing | Rule syntax or no metric pressure | Scale config and metrics | Fix scale rule and generate test load |
| Too many replicas | Aggressive scaling or max too high | Scale rule and replica list | Lower max replicas or raise threshold |
| Scale to zero causes cold starts | Min replicas set to 0 | Scale config | Set min replicas to 1 or higher |
| New revision bad | Image or config regression | Revision list and logs | Activate previous revision or update image |
| Secret not available | Secret name mismatch | App secrets and env var references | Fix secret reference and create new revision |
| ACR admin credentials exposed | Unsafe lab shortcut | Credential usage and repo history | Disable admin user and rotate credentials |
| Log Analytics missing logs | Workspace config or delay | Environment logs configuration | Verify workspace ID/key and wait for ingestion |
| Private image works in ACI but not Container Apps | Different auth path | Registry settings and identity | Configure Container App registry with managed identity |
| Container App update does nothing visible | New revision not active or same image tag reused | Revision list and image digest | Use unique image tags or activate revision |
| Cost higher than expected | Replicas, SKU, logs, or ACR tier | Cost analysis and resource list | Lower replicas, delete ACI, reduce SKU, or delete lab RG |

# Deploy_ACR_ACI_Azure_Container_Apps_Sizing_And_Scaling_Related_Labs

| Lab                                                                 | Related Workbook                                                                         | Skill Proven                                          |
| ------------------------------------------------------------------- | ---------------------------------------------------------------------------------------- | ----------------------------------------------------- |
| Deploy ARM and Bicep baseline                                       | `01_Deploy_ARM_Templates_Bicep_And_Exported_Deployments.md`                              | Declarative deployment foundation                     |
| Create and manage Azure VMs                                         | `02_Create_Configure_And_Manage_Azure_Virtual_Machines.md`                               | Individual VM compute lifecycle                       |
| Configure VM disks, sizes, availability, extensions, and encryption | `03_Configure_VM_Disks_Sizes_Availability_Sets_Zones_Extensions_And_Encryption.md`       | VM platform configuration                             |
| Configure VMSS images and scaling                                   | `04_Configure_VMSS_Images_Scaling_And_Automation.md`                                     | Scale set image and autoscale management              |
| Deploy ACR, ACI, and Azure Container Apps                           | `05_Deploy_ACR_ACI_Azure_Container_Apps_Sizing_And_Scaling.md`                           | Container registry, runtime, app hosting, and scaling |
| Deploy App Service plans and web apps                               | `06_Deploy_App_Service_Plans_Web_Apps_TLS_Custom_Domains_Backup_Networking_And_Slots.md` | PaaS web hosting                                      |
| Configure compute managed identities and Key Vault references       | `07_Configure_Compute_Managed_Identities_Key_Vault_Access_And_Secrets_References.md`     | Managed identity and secretless access                |
| Troubleshoot compute deployment and runtime issues                  | `08_Troubleshoot_Azure_VM_Container_App_Service_Deployment_And_Runtime_Issues.md`        | Container and compute troubleshooting                 |