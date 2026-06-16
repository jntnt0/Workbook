# 08_Troubleshoot_Azure_VM_Container_App_Service_Deployment_And_Runtime_Issues

# 08_Troubleshoot_Azure_VM_Container_App_Service_Deployment_And_Runtime_Issues_Index

08_Troubleshoot_Azure_VM_Container_App_Service_Deployment_And_Runtime_Issues.md  
Troubleshoot_Azure_VM_Container_App_Service_Deployment_And_Runtime_Issues  
Troubleshoot_Azure_VM_Container_App_Service_Deployment_And_Runtime_Issues_Source_Basis  
Troubleshoot_Azure_VM_Container_App_Service_Deployment_And_Runtime_Issues_Mental_Model  
Troubleshoot_Azure_VM_Container_App_Service_Deployment_And_Runtime_Issues_Planning_Table  
Troubleshoot_Azure_VM_Container_App_Service_Deployment_And_Runtime_Issues_Symptom_Map  
Troubleshoot_Azure_VM_Container_App_Service_Deployment_And_Runtime_Issues_Log_And_Diagnostic_Evidence  
Troubleshoot_Azure_VM_Container_App_Service_Deployment_And_Runtime_Issues_Triage_Order  
Troubleshoot_Azure_VM_Container_App_Service_Deployment_And_Runtime_Issues_Configuration_Checklist  
Troubleshoot_Azure_VM_Container_App_Service_Deployment_And_Runtime_Issues_Portal_Skeleton  
Troubleshoot_Azure_VM_Container_App_Service_Deployment_And_Runtime_Issues_AzureCLI_Skeleton  
Troubleshoot_Azure_VM_Container_App_Service_Deployment_And_Runtime_Issues_PowerShell_Skeleton  
Troubleshoot_Azure_VM_Container_App_Service_Deployment_And_Runtime_Issues_VM_Troubleshooting_Skeleton  
Troubleshoot_Azure_VM_Container_App_Service_Deployment_And_Runtime_Issues_Container_Apps_Troubleshooting_Skeleton  
Troubleshoot_Azure_VM_Container_App_Service_Deployment_And_Runtime_Issues_App_Service_Troubleshooting_Skeleton  
Troubleshoot_Azure_VM_Container_App_Service_Deployment_And_Runtime_Issues_ARM_Deployment_Troubleshooting_Skeleton  
Troubleshoot_Azure_VM_Container_App_Service_Deployment_And_Runtime_Issues_Network_And_DNS_Troubleshooting_Skeleton  
Troubleshoot_Azure_VM_Container_App_Service_Deployment_And_Runtime_Issues_Identity_And_Secrets_Troubleshooting_Skeleton  
Troubleshoot_Azure_VM_Container_App_Service_Deployment_And_Runtime_Issues_Verification_Commands  
Troubleshoot_Azure_VM_Container_App_Service_Deployment_And_Runtime_Issues_Rollback  
Troubleshoot_Azure_VM_Container_App_Service_Deployment_And_Runtime_Issues_Failure_Checks  
Troubleshoot_Azure_VM_Container_App_Service_Deployment_And_Runtime_Issues_Related_Labs  

# Troubleshoot_Azure_VM_Container_App_Service_Deployment_And_Runtime_Issues_Source_Basis

| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | ARM deployment troubleshooting | Deployment failures, operation logs, invalid template, invalid reference, quota, policy, SKU, and provider registration errors |
| Microsoft Learn | Azure Virtual Machines troubleshooting | VM provisioning, boot diagnostics, serial console, run command, VM agent, extensions, networking, and access issues |
| Microsoft Learn | Azure VM boot diagnostics | Boot log and screenshot evidence for startup and OS-level failures |
| Microsoft Learn | Azure VM Run Command | Guest command execution when direct SSH or RDP is unavailable |
| Microsoft Learn | Azure VM extensions troubleshooting | Extension provisioning, handler status, guest agent dependency, and script failure analysis |
| Microsoft Learn | Azure Network Watcher | Effective NSG rules, effective routes, IP flow verify, connection troubleshoot, and packet capture |
| Microsoft Learn | Azure Container Apps troubleshooting | Image pull, revision, replica, ingress, target port, container start, probes, logs, and scale behavior |
| Microsoft Learn | Azure Container Apps logging | Log streaming, revision logs, replica logs, system logs, and Log Analytics queries |
| Microsoft Learn | Azure Container Apps managed identity and registry auth | ACR image pull with managed identity and AcrPull role |
| Microsoft Learn | Azure App Service diagnostics | Web app logs, deployment logs, Kudu diagnostics, App Service logs, Diagnose and solve problems, and metrics |
| Microsoft Learn | Azure App Service custom domains and TLS | DNS validation, hostname binding, certificate binding, HTTPS-only, and TLS failures |
| Microsoft Learn | Azure App Service networking | Access restrictions, VNet integration, private endpoint, private DNS, and outbound dependency path |
| Microsoft Learn | Azure Container Registry troubleshooting | Image repository, tag, manifest, digest, authentication, and pull failures |
| Azure operational practice | Incident triage | Identify scope, collect evidence, isolate failing layer, fix smallest safe layer, verify, and document |
| Azure operational practice | Safe rollback | Revert template, image, revision, slot, size, NSG, DNS, or identity changes with evidence |

# Troubleshoot_Azure_VM_Container_App_Service_Deployment_And_Runtime_Issues_Mental_Model

| Concept | Operational Meaning |
|---|---|
| Deployment failure | Azure Resource Manager could not create or update one or more resources |
| Runtime failure | Resource deployed but app or VM workload does not run correctly |
| Control plane | Azure management API layer used by ARM, CLI, portal, Bicep, and PowerShell |
| Data plane | Runtime access path to the workload, such as SSH, RDP, HTTP, HTTPS, ACR pull, or Key Vault secret read |
| Provisioning state | Azure resource create or update result |
| Power state | VM runtime state, such as running, stopped, or deallocated |
| Instance view | VM or VMSS runtime status details from the platform |
| Boot diagnostics | Platform evidence from VM boot process |
| VM agent | Guest agent required for extensions and Run Command |
| Extension handler | Guest-side component that runs VM extension logic |
| Container image | Immutable container package pulled by Container Apps, ACI, or App Service container runtime |
| Image pull | Runtime operation that authenticates to registry and downloads image |
| Revision | Container Apps deployment snapshot |
| Replica | Running instance of a Container Apps revision |
| Ingress | Container Apps or App Service HTTP entry point |
| Target port | Port where the container process listens |
| App Service slot | Separate deployment target for safe release and rollback |
| Kudu | App Service advanced deployment and diagnostics site |
| Effective NSG | Final network security rules applied to a NIC |
| Effective route | Final route table decision applied to a NIC |
| Private endpoint | Inbound private IP access to a PaaS resource |
| VNet integration | App Service outbound path into a VNet |
| Private DNS | DNS zone required for private endpoint name resolution |
| Managed identity | Microsoft Entra identity used for secretless access |
| AcrPull | RBAC role required to pull images from Azure Container Registry |
| First rule | Separate deployment failure from runtime failure before changing anything |
| Blunt rule | Logs first, fixes second; guessing at Azure compute failures burns time and creates drift |

# Troubleshoot_Azure_VM_Container_App_Service_Deployment_And_Runtime_Issues_Planning_Table

| Item | Example | Decision |
|---|---|---|
| Tenant | `contoso.onmicrosoft.com` | `<tenant-name>` |
| Subscription ID | `00000000-0000-0000-0000-000000000000` | `<subscription-id>` |
| Resource group | `rg-compute-lab-01` | `<resource-group-name>` |
| Location | `eastus` | `<azure-region>` |
| Incident ID | `INC-2026-0008` | `<incident-id>` |
| Affected resource type | `VM`, `Container App`, `App Service` | `<resource-type>` |
| Affected VM | `vm-lab-linux-01` | `<vm-name>` |
| Affected Container App | `ca-web-lab-01` | `<container-app-name>` |
| Container Apps environment | `cae-compute-lab-01` | `<container-app-env-name>` |
| Affected App Service | `app-web-lab-01` | `<web-app-name>` |
| App Service Plan | `asp-web-lab-01` | `<app-service-plan-name>` |
| ACR name | `acrcomputelab01` | `<acr-name>` |
| Image | `acrcomputelab01.azurecr.io/webapp:v1` | `<image-name>` |
| Current revision | `ca-web-lab-01--abc123` | `<revision-name>` |
| Deployment name | `dep-compute-baseline-01` | `<deployment-name>` |
| Symptom start time | `2026-06-16T09:00:00-04:00` | `<start-time>` |
| Last known-good change | `slot swap`, `new image`, `VM resize` | `<last-change>` |
| Expected endpoint | `https://app.contoso.com` | `<endpoint>` |
| Approved admin source | `203.0.113.10/32` | `<approved-source-cidr>` |
| Evidence path | `.\evidence\08-compute-troubleshooting` | `<evidence-path>` |
| Rollback candidate | `previous image`, `previous slot`, `known-good Bicep` | `<rollback-candidate>` |
| Business impact | `Lab app unavailable` | `<impact>` |
| Owner | `Cloud Admin` | `<owner>` |

# Troubleshoot_Azure_VM_Container_App_Service_Deployment_And_Runtime_Issues_Symptom_Map

| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| ARM deployment failed immediately | RBAC, provider, policy, invalid template | Deployment operation details | Fix permission, provider, policy, or template |
| `AuthorizationFailed` | Missing Azure role assignment | `az role assignment list` | Assign required role or target correct subscription |
| `MissingSubscriptionRegistration` | Resource provider not registered | `az provider show` | Register provider |
| `InvalidTemplate` | Bad Bicep or ARM schema | Validate deployment | Fix template syntax or properties |
| `InvalidResourceReference` | Missing dependency or wrong resource ID | Deployment operation error | Fix symbolic reference, resourceId, or dependency |
| `SkuNotAvailable` | Region or zone does not support SKU | Error detail and SKU list | Choose supported SKU, region, or zone |
| `QuotaExceeded` | Subscription quota too low | Quota blade or error detail | Request quota or reduce size/count |
| VM stuck creating | Platform provisioning issue | VM instance view and deployment operations | Inspect operation failure and retry or redeploy |
| VM running but unreachable | NSG, route, public IP, Bastion, guest firewall | Effective NSG and route | Fix network path or use Bastion |
| VM boots to error | OS or boot config issue | Boot diagnostics | Use serial console, redeploy, restore, or repair disk |
| Run Command fails | VM agent unavailable | Instance view and guest agent | Restart agent, reboot VM, or repair guest |
| VM extension failed | Script error, agent issue, blocked download | Extension status and logs | Fix script, network, settings, or handler |
| Container App revision inactive | Bad deployment or activation issue | Revision list | Activate known-good revision or fix new revision |
| Container App image pull fails | Wrong image, missing tag, no AcrPull | Replica logs and registry config | Fix image name, tag, identity, or role |
| Container App starts then exits | App crash or bad command | Logs and replica status | Fix image, command, env vars, or secrets |
| Container App returns 502 | Wrong target port or app not listening | Ingress config and logs | Fix target port or listener |
| Container App does not scale | Scale rule or metric issue | Scale config and replicas | Fix min/max, rule, or event source |
| App Service deployment failed | Package, build, or deployment config issue | Deployment logs and Kudu | Fix package, startup, build, or app settings |
| App Service returns 500 | Application runtime failure | App logs | Fix app code, settings, connection string, or identity |
| App Service returns 502 or 503 | Startup, worker, container, or plan issue | Logs, metrics, container logs | Fix app startup, scale, plan, or image |
| Custom domain fails | DNS or hostname binding issue | DNS lookup and hostname list | Fix TXT, CNAME, A record, or binding |
| TLS fails | Missing certificate or wrong binding | SSL list and endpoint test | Bind correct cert and enforce HTTPS |
| Private endpoint app resolves public IP | Missing private DNS | `nslookup` from VNet | Configure private DNS zone and link |
| App cannot reach private dependency | VNet integration, DNS, route, firewall | Console test and network rules | Fix VNet integration, DNS, route, or firewall |
| Key Vault reference fails | Identity or Key Vault permission issue | App settings and Key Vault access | Assign secret read role or fix reference |
| ACR pull fails in App Service | Managed identity or registry config issue | Web app container config | Assign AcrPull and configure registry auth |

# Troubleshoot_Azure_VM_Container_App_Service_Deployment_And_Runtime_Issues_Log_And_Diagnostic_Evidence

| Evidence | Resource | Command / Location | What It Proves |
|---|---|---|---|
| Account context | All | `az account show -o table` | Correct tenant and subscription |
| Resource inventory | All | `az resource list -g "<resource-group-name>" -o table` | Existing resources and types |
| Activity log | All | `az monitor activity-log list -g "<resource-group-name>"` | Azure control-plane events |
| Deployment details | ARM/Bicep | `az deployment group show` | Deployment state and outputs |
| Deployment operations | ARM/Bicep | `az deployment operation group list` | Exact failed resource operation |
| Provider state | ARM/Bicep | `az provider show --namespace <provider>` | Resource provider registration |
| Policy events | ARM/Bicep | Azure Policy compliance and deployment errors | Policy deny or modify impact |
| VM instance view | VM | `az vm get-instance-view` | Power state, agent, extension, provisioning |
| VM boot log | VM | `az vm boot-diagnostics get-boot-log` | Guest startup and boot issues |
| VM Run Command output | VM | `az vm run-command invoke` | Guest OS reachability through control plane |
| Effective NSG | VM | `az network nic list-effective-nsg` | Final inbound and outbound security rules |
| Effective routes | VM | `az network nic show-effective-route-table` | Final routing path |
| VM extension list | VM | `az vm extension list` | Extension installation and status |
| Serial console | VM | Portal > VM > Serial console | Low-level guest recovery path |
| Container App show | Container Apps | `az containerapp show` | Provisioning, ingress, template, revision mode |
| Container App revisions | Container Apps | `az containerapp revision list` | Active and inactive revisions |
| Container App replicas | Container Apps | `az containerapp replica list` | Replica health and running state |
| Container App logs | Container Apps | `az containerapp logs show` | App startup, crash, and request logs |
| Container Apps environment | Container Apps | `az containerapp env show` | Environment and Log Analytics wiring |
| ACR repository tags | ACR | `az acr repository show-tags` | Image tag exists |
| ACR manifests | ACR | `az acr repository show-manifests` | Image digest and timestamps |
| App Service show | App Service | `az webapp show` | Web app state and hostname |
| App Service config | App Service | `az webapp config show` | Runtime, TLS, platform settings |
| App settings | App Service | `az webapp config appsettings list` | Environment variables and Key Vault refs |
| App Service logs | App Service | `az webapp log tail` | Runtime and app errors |
| Deployment logs | App Service | Kudu or deployment center logs | Package or build failure |
| Hostname list | App Service | `az webapp config hostname list` | Custom domain binding |
| SSL list | App Service | `az webapp config ssl list` | Certificate availability and bindings |
| Access restrictions | App Service | `az webapp config access-restriction show` | Inbound allow and deny rules |
| VNet integration | App Service | `az webapp vnet-integration list` | Outbound VNet path |
| Private endpoint list | App Service | `az network private-endpoint list` | Inbound private endpoint exists |

# Troubleshoot_Azure_VM_Container_App_Service_Deployment_And_Runtime_Issues_Triage_Order

| Order | Layer | Question | Evidence |
|---:|---|---|---|
| 1 | Scope | Which resource, region, subscription, and time window are affected? | Account context, resource list, incident notes |
| 2 | Last change | What changed immediately before the failure? | Activity log, deployment history, image tag, slot swap |
| 3 | Deployment | Did ARM/Bicep deployment succeed? | Deployment show and operation list |
| 4 | Provider | Are required resource providers registered? | Provider state |
| 5 | Policy/RBAC | Did policy or permissions block the operation? | Deployment error, role assignment, policy events |
| 6 | Resource state | Does the resource exist and show healthy provisioning state? | Resource show command |
| 7 | Runtime state | Is the VM, replica, revision, or web app running? | Instance view, replica list, app state |
| 8 | Logs | What is the first real error in platform or app logs? | Boot diagnostics, container logs, web app logs |
| 9 | Network | Can traffic reach the resource and can the resource reach dependencies? | NSG, routes, DNS, access restrictions |
| 10 | Identity | Can the runtime identity access ACR, Key Vault, storage, or dependency? | Identity show, role assignments, Key Vault access |
| 11 | Config | Are ports, app settings, runtime stack, image tags, and secrets correct? | App config, container template, env vars |
| 12 | Rollback | Is the fastest safe fix to revert image, slot, template, or setting? | Known-good evidence and rollback table |
| 13 | Verify | Did endpoint, logs, metrics, and health checks return normal? | Curl, logs, metrics, revision state |
| 14 | Document | What failed, what fixed it, and what guardrail prevents recurrence? | Incident notes and workbook evidence |

# Troubleshoot_Azure_VM_Container_App_Service_Deployment_And_Runtime_Issues_Configuration_Checklist

| Step | Task | Device | PowerShell / Command | Expected Result |
|---:|---|---|---|---|
| 1 | Create evidence folder | Admin Workstation | `mkdir -p .\evidence\08-compute-troubleshooting` | Evidence path exists |
| 2 | Confirm Azure context | Admin Workstation | `az account show -o table` | Correct subscription is active |
| 3 | Set subscription | Admin Workstation | `az account set --subscription "<subscription-id>"` | CLI targets correct subscription |
| 4 | Capture resource inventory | Admin Workstation | `az resource list --resource-group "<resource-group-name>" -o table` | Resource list is visible |
| 5 | Capture activity log | Admin Workstation | `az monitor activity-log list --resource-group "<resource-group-name>" --max-events 50 -o table` | Recent control-plane events are visible |
| 6 | List deployments | Admin Workstation | `az deployment group list --resource-group "<resource-group-name>" -o table` | Deployment history is visible |
| 7 | Inspect failed deployment | Admin Workstation | `az deployment group show --resource-group "<resource-group-name>" --name "<deployment-name>" -o json` | Deployment status and error details are visible |
| 8 | Inspect deployment operations | Admin Workstation | `az deployment operation group list --resource-group "<resource-group-name>" --name "<deployment-name>" -o json` | Failed operation details are visible |
| 9 | Check provider registration | Admin Workstation | `az provider show --namespace Microsoft.Compute --query registrationState -o tsv` | Provider state is confirmed |
| 10 | Register missing provider | Admin Workstation | `az provider register --namespace "<provider-namespace>"` | Provider registration starts |
| 11 | Inspect VM state | Admin Workstation | `az vm show --resource-group "<resource-group-name>" --name "<vm-name>" --show-details -o json` | VM configuration and state are visible |
| 12 | Inspect VM instance view | Admin Workstation | `az vm get-instance-view --resource-group "<resource-group-name>" --name "<vm-name>" -o json` | Runtime status is visible |
| 13 | Capture VM boot log | Admin Workstation | `az vm boot-diagnostics get-boot-log --resource-group "<resource-group-name>" --name "<vm-name>"` | Boot evidence is captured |
| 14 | Test Linux guest with Run Command | Admin Workstation | `az vm run-command invoke --resource-group "<resource-group-name>" --name "<linux-vm-name>" --command-id RunShellScript --scripts "hostname; uptime; df -h; ip addr"` | Linux guest responds |
| 15 | Test Windows guest with Run Command | Admin Workstation | `az vm run-command invoke --resource-group "<resource-group-name>" --name "<windows-vm-name>" --command-id RunPowerShellScript --scripts "hostname; Get-NetIPConfiguration; Get-Service WindowsAzureGuestAgent"` | Windows guest responds |
| 16 | Inspect VM effective NSG | Admin Workstation | `az network nic list-effective-nsg --resource-group "<resource-group-name>" --name "<nic-name>" -o table` | Effective rules are visible |
| 17 | Inspect VM effective routes | Admin Workstation | `az network nic show-effective-route-table --resource-group "<resource-group-name>" --name "<nic-name>" -o table` | Effective routes are visible |
| 18 | Inspect VM extensions | Admin Workstation | `az vm extension list --resource-group "<resource-group-name>" --vm-name "<vm-name>" -o table` | Extension states are visible |
| 19 | Inspect Container App | Admin Workstation | `az containerapp show --name "<container-app-name>" --resource-group "<resource-group-name>" -o json` | Container App config is visible |
| 20 | List Container App revisions | Admin Workstation | `az containerapp revision list --name "<container-app-name>" --resource-group "<resource-group-name>" -o table` | Revision status is visible |
| 21 | List Container App replicas | Admin Workstation | `az containerapp replica list --name "<container-app-name>" --resource-group "<resource-group-name>" --revision "<revision-name>" -o table` | Replica status is visible |
| 22 | Capture Container App logs | Admin Workstation | `az containerapp logs show --name "<container-app-name>" --resource-group "<resource-group-name>" --follow false` | Logs are captured |
| 23 | Verify ACR image tag | Admin Workstation | `az acr repository show-tags --name "<acr-name>" --repository "<repository-name>" -o table` | Image tag exists |
| 24 | Verify ACR manifest | Admin Workstation | `az acr repository show-manifests --name "<acr-name>" --repository "<repository-name>" -o table` | Digest evidence is visible |
| 25 | Inspect Container App identity | Admin Workstation | `az containerapp identity show --name "<container-app-name>" --resource-group "<resource-group-name>" -o json` | Principal ID is visible |
| 26 | Inspect Container App registry config | Admin Workstation | `az containerapp registry list --name "<container-app-name>" --resource-group "<resource-group-name>" -o table` | Registry auth config is visible |
| 27 | Inspect App Service | Admin Workstation | `az webapp show --name "<web-app-name>" --resource-group "<resource-group-name>" -o json` | Web app state is visible |
| 28 | Inspect App Service config | Admin Workstation | `az webapp config show --name "<web-app-name>" --resource-group "<resource-group-name>" -o json` | Runtime and platform config are visible |
| 29 | Inspect App Service app settings | Admin Workstation | `az webapp config appsettings list --name "<web-app-name>" --resource-group "<resource-group-name>" -o table` | App settings are visible |
| 30 | Tail App Service logs | Admin Workstation | `az webapp log tail --name "<web-app-name>" --resource-group "<resource-group-name>"` | Runtime logs stream |
| 31 | Inspect hostnames | Admin Workstation | `az webapp config hostname list --webapp-name "<web-app-name>" --resource-group "<resource-group-name>" -o table` | Hostname bindings are visible |
| 32 | Inspect certificates | Admin Workstation | `az webapp config ssl list --resource-group "<resource-group-name>" -o table` | Certificates are visible |
| 33 | Inspect access restrictions | Admin Workstation | `az webapp config access-restriction show --name "<web-app-name>" --resource-group "<resource-group-name>" -o json` | Inbound rules are visible |
| 34 | Inspect VNet integration | Admin Workstation | `az webapp vnet-integration list --name "<web-app-name>" --resource-group "<resource-group-name>" -o table` | Outbound VNet path is visible |
| 35 | Test endpoint | Admin Workstation | `curl -I "https://<endpoint>"` | HTTP status and headers are visible |
| 36 | Perform rollback if required | Admin Workstation | `Use rollback command from rollback table` | Known-good state is restored |
| 37 | Verify after fix | Admin Workstation | `Repeat endpoint, logs, and resource state checks` | Resource returns expected state |
| 38 | Document incident | Operator | `Record root cause, fix, commands, evidence, and prevention` | Incident record is complete |

# Troubleshoot_Azure_VM_Container_App_Service_Deployment_And_Runtime_Issues_Portal_Skeleton

```text
Purpose:
Use the Azure portal to triage Azure compute deployment and runtime failures.

Global triage:
1. Azure portal
2. Open Resource group: <resource-group-name>
3. Review Activity log.
4. Review Deployments.
5. Open failed deployment.
6. Open Operation details.
7. Copy error code, target resource, message, and correlation ID.

VM triage:
1. Azure portal > Virtual machines > <vm-name>.
2. Overview:
   - Status
   - Public IP
   - Private IP
   - Size
   - Location
3. Activity log:
   - Create/update/start/restart/deallocate events
4. Diagnose and solve problems:
   - VM performance
   - Connectivity
   - Boot issues
5. Boot diagnostics:
   - Screenshot
   - Serial log
6. Run command:
   - Linux: shell command
   - Windows: PowerShell command
7. Extensions + applications:
   - Extension provisioning state
   - Handler status
8. Networking:
   - Effective security rules
   - Effective routes
9. Serial console:
   - Use only if boot or OS recovery requires it.

Container Apps triage:
1. Azure portal > Container Apps > <container-app-name>.
2. Overview:
   - Provisioning state
   - Application Url
   - Environment
3. Revisions:
   - Active revision
   - Failed revision
   - Traffic allocation
4. Replicas:
   - Running state
   - Restart count
5. Logs:
   - Console logs
   - System logs
6. Containers:
   - Image
   - CPU
   - Memory
   - Environment variables
   - Secrets
7. Ingress:
   - External or internal
   - Target port
   - Transport
8. Scale:
   - Min replicas
   - Max replicas
   - Rules
9. Identity:
   - System-assigned or user-assigned identity
10. Registry:
   - Server
   - Identity or secret-based auth

App Service triage:
1. Azure portal > App Services > <web-app-name>.
2. Overview:
   - Status
   - URL
   - App Service Plan
3. Diagnose and solve problems:
   - Availability and performance
   - Configuration and management
   - SSL and domains
   - Networking
4. Deployment Center:
   - Deployment status
   - Build logs
5. Log stream:
   - Application logs
   - Web server logs
6. Configuration:
   - Runtime stack
   - App settings
   - Connection strings
   - Startup command
7. Custom domains:
   - Hostname binding
   - Verification state
8. TLS/SSL settings:
   - HTTPS only
   - Certificate binding
   - Minimum TLS
9. Networking:
   - Access restrictions
   - VNet integration
   - Private endpoints
10. Deployment slots:
   - Staging state
   - Swap history
   - Sticky settings

Portal evidence:
1. Save screenshots only when useful.
2. Prefer downloaded JSON, logs, and copied error details.
3. Record correlation IDs and timestamps.
```

# Troubleshoot_Azure_VM_Container_App_Service_Deployment_And_Runtime_Issues_AzureCLI_Skeleton

```bash
# Run from Azure Cloud Shell or local Azure CLI.
# Purpose: collect compute troubleshooting evidence for ARM deployments, VMs, Container Apps, App Service, ACR, networking, identity, and DNS.

# -----------------------------
# Variables
# -----------------------------

export SUBSCRIPTION_ID="<subscription-id>"
export RESOURCE_GROUP="rg-compute-lab-01"
export LOCATION="eastus"

export DEPLOYMENT_NAME="<deployment-name>"

export VM_NAME="vm-lab-linux-01"
export LINUX_VM_NAME="vm-lab-linux-01"
export WINDOWS_VM_NAME="vm-lab-win-01"
export NIC_NAME="<nic-name>"

export ACR_NAME="acrcomputelab01"
export REPOSITORY_NAME="webapp"
export IMAGE_TAG="v1"

export CONTAINERAPP_NAME="ca-web-lab-01"
export CONTAINERAPP_ENV_NAME="cae-compute-lab-01"

export WEBAPP_NAME="app-web-lab-01"
export PLAN_NAME="asp-web-lab-01"

export ENDPOINT="https://app.contoso.com"

export EVIDENCE_PATH="./evidence/08-compute-troubleshooting"

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
# Resource inventory and activity
# -----------------------------

az resource list \
  --resource-group "$RESOURCE_GROUP" \
  --output table |
  tee "$EVIDENCE_PATH/resource-list.txt"

az monitor activity-log list \
  --resource-group "$RESOURCE_GROUP" \
  --max-events 50 \
  --output table |
  tee "$EVIDENCE_PATH/activity-log.txt"

# -----------------------------
# Provider state
# -----------------------------

for provider in \
  Microsoft.Resources \
  Microsoft.Compute \
  Microsoft.Network \
  Microsoft.Storage \
  Microsoft.Web \
  Microsoft.App \
  Microsoft.ContainerRegistry \
  Microsoft.ContainerInstance \
  Microsoft.OperationalInsights \
  Microsoft.Insights
do
  az provider show \
    --namespace "$provider" \
    --query "{namespace:namespace, registrationState:registrationState}" \
    --output table
done | tee "$EVIDENCE_PATH/provider-state.txt"

# -----------------------------
# ARM deployment troubleshooting
# -----------------------------

az deployment group list \
  --resource-group "$RESOURCE_GROUP" \
  --output table |
  tee "$EVIDENCE_PATH/deployment-list.txt"

az deployment group show \
  --resource-group "$RESOURCE_GROUP" \
  --name "$DEPLOYMENT_NAME" \
  --output json |
  tee "$EVIDENCE_PATH/deployment-show.json"

az deployment operation group list \
  --resource-group "$RESOURCE_GROUP" \
  --name "$DEPLOYMENT_NAME" \
  --output json |
  tee "$EVIDENCE_PATH/deployment-operations.json"

az deployment operation group list \
  --resource-group "$RESOURCE_GROUP" \
  --name "$DEPLOYMENT_NAME" \
  --query "[?properties.provisioningState=='Failed']" \
  --output json |
  tee "$EVIDENCE_PATH/deployment-failed-operations.json"

# -----------------------------
# VM troubleshooting
# -----------------------------

az vm show \
  --resource-group "$RESOURCE_GROUP" \
  --name "$VM_NAME" \
  --show-details \
  --output json |
  tee "$EVIDENCE_PATH/vm-show.json"

az vm get-instance-view \
  --resource-group "$RESOURCE_GROUP" \
  --name "$VM_NAME" \
  --output json |
  tee "$EVIDENCE_PATH/vm-instance-view.json"

az vm boot-diagnostics get-boot-log \
  --resource-group "$RESOURCE_GROUP" \
  --name "$VM_NAME" |
  tee "$EVIDENCE_PATH/vm-boot-log.txt"

az vm extension list \
  --resource-group "$RESOURCE_GROUP" \
  --vm-name "$VM_NAME" \
  --output json |
  tee "$EVIDENCE_PATH/vm-extension-list.json"

az vm run-command invoke \
  --resource-group "$RESOURCE_GROUP" \
  --name "$LINUX_VM_NAME" \
  --command-id RunShellScript \
  --scripts "hostname; date; uptime; df -h; ip addr; systemctl status waagent --no-pager || true" \
  --output json |
  tee "$EVIDENCE_PATH/linux-run-command.json"

az vm run-command invoke \
  --resource-group "$RESOURCE_GROUP" \
  --name "$WINDOWS_VM_NAME" \
  --command-id RunPowerShellScript \
  --scripts "hostname; Get-Date; Get-NetIPConfiguration; Get-Service WindowsAzureGuestAgent; Get-EventLog -LogName System -Newest 20" \
  --output json |
  tee "$EVIDENCE_PATH/windows-run-command.json"

az network nic list-effective-nsg \
  --resource-group "$RESOURCE_GROUP" \
  --name "$NIC_NAME" \
  --output table |
  tee "$EVIDENCE_PATH/vm-effective-nsg.txt"

az network nic show-effective-route-table \
  --resource-group "$RESOURCE_GROUP" \
  --name "$NIC_NAME" \
  --output table |
  tee "$EVIDENCE_PATH/vm-effective-routes.txt"

# -----------------------------
# ACR troubleshooting
# -----------------------------

az acr show \
  --name "$ACR_NAME" \
  --output json |
  tee "$EVIDENCE_PATH/acr-show.json"

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
# Container Apps troubleshooting
# -----------------------------

az containerapp env show \
  --name "$CONTAINERAPP_ENV_NAME" \
  --resource-group "$RESOURCE_GROUP" \
  --output json |
  tee "$EVIDENCE_PATH/containerapp-env-show.json"

az containerapp show \
  --name "$CONTAINERAPP_NAME" \
  --resource-group "$RESOURCE_GROUP" \
  --output json |
  tee "$EVIDENCE_PATH/containerapp-show.json"

az containerapp revision list \
  --name "$CONTAINERAPP_NAME" \
  --resource-group "$RESOURCE_GROUP" \
  --output json |
  tee "$EVIDENCE_PATH/containerapp-revisions.json"

ACTIVE_REVISION=$(az containerapp revision list \
  --name "$CONTAINERAPP_NAME" \
  --resource-group "$RESOURCE_GROUP" \
  --query "[?properties.active==\`true\`].name | [0]" \
  --output tsv)

echo "$ACTIVE_REVISION" |
  tee "$EVIDENCE_PATH/containerapp-active-revision.txt"

az containerapp replica list \
  --name "$CONTAINERAPP_NAME" \
  --resource-group "$RESOURCE_GROUP" \
  --revision "$ACTIVE_REVISION" \
  --output json |
  tee "$EVIDENCE_PATH/containerapp-replicas.json"

az containerapp logs show \
  --name "$CONTAINERAPP_NAME" \
  --resource-group "$RESOURCE_GROUP" \
  --follow false |
  tee "$EVIDENCE_PATH/containerapp-logs.txt"

az containerapp identity show \
  --name "$CONTAINERAPP_NAME" \
  --resource-group "$RESOURCE_GROUP" \
  --output json |
  tee "$EVIDENCE_PATH/containerapp-identity.json"

az containerapp registry list \
  --name "$CONTAINERAPP_NAME" \
  --resource-group "$RESOURCE_GROUP" \
  --output table |
  tee "$EVIDENCE_PATH/containerapp-registry.txt"

# -----------------------------
# App Service troubleshooting
# -----------------------------

az appservice plan show \
  --name "$PLAN_NAME" \
  --resource-group "$RESOURCE_GROUP" \
  --output json |
  tee "$EVIDENCE_PATH/appservice-plan-show.json"

az webapp show \
  --name "$WEBAPP_NAME" \
  --resource-group "$RESOURCE_GROUP" \
  --output json |
  tee "$EVIDENCE_PATH/webapp-show.json"

az webapp config show \
  --name "$WEBAPP_NAME" \
  --resource-group "$RESOURCE_GROUP" \
  --output json |
  tee "$EVIDENCE_PATH/webapp-config.json"

az webapp config appsettings list \
  --name "$WEBAPP_NAME" \
  --resource-group "$RESOURCE_GROUP" \
  --output json |
  tee "$EVIDENCE_PATH/webapp-appsettings.json"

az webapp config hostname list \
  --webapp-name "$WEBAPP_NAME" \
  --resource-group "$RESOURCE_GROUP" \
  --output table |
  tee "$EVIDENCE_PATH/webapp-hostnames.txt"

az webapp config ssl list \
  --resource-group "$RESOURCE_GROUP" \
  --output table |
  tee "$EVIDENCE_PATH/webapp-ssl-list.txt"

az webapp config access-restriction show \
  --name "$WEBAPP_NAME" \
  --resource-group "$RESOURCE_GROUP" \
  --output json |
  tee "$EVIDENCE_PATH/webapp-access-restrictions.json"

az webapp vnet-integration list \
  --name "$WEBAPP_NAME" \
  --resource-group "$RESOURCE_GROUP" \
  --output table |
  tee "$EVIDENCE_PATH/webapp-vnet-integration.txt"

az webapp deployment slot list \
  --name "$WEBAPP_NAME" \
  --resource-group "$RESOURCE_GROUP" \
  --output table |
  tee "$EVIDENCE_PATH/webapp-slots.txt"

az webapp log config \
  --name "$WEBAPP_NAME" \
  --resource-group "$RESOURCE_GROUP" |
  tee "$EVIDENCE_PATH/webapp-log-config.txt"

# Do not use --follow for evidence collection unless live observation is needed.
timeout 30 az webapp log tail \
  --name "$WEBAPP_NAME" \
  --resource-group "$RESOURCE_GROUP" |
  tee "$EVIDENCE_PATH/webapp-log-tail.txt"

# -----------------------------
# Endpoint and DNS tests
# -----------------------------

curl -I "$ENDPOINT" |
  tee "$EVIDENCE_PATH/endpoint-curl-headers.txt"

nslookup "$(echo "$ENDPOINT" | sed 's#https://##' | sed 's#http://##' | cut -d/ -f1)" |
  tee "$EVIDENCE_PATH/endpoint-nslookup.txt"

# -----------------------------
# Final triage note
# -----------------------------

echo "Review failed deployment operations, runtime logs, network path, identity permissions, and last change before applying fixes." |
  tee "$EVIDENCE_PATH/triage-note.txt"
```

# Troubleshoot_Azure_VM_Container_App_Service_Deployment_And_Runtime_Issues_PowerShell_Skeleton

```powershell
# Run from PowerShell with Az module installed.
# Purpose: collect Azure compute troubleshooting evidence.

# -----------------------------
# Variables
# -----------------------------

$SubscriptionId = "<subscription-id>"
$ResourceGroupName = "rg-compute-lab-01"

$DeploymentName = "<deployment-name>"

$VmName = "vm-lab-linux-01"
$LinuxVmName = "vm-lab-linux-01"
$WindowsVmName = "vm-lab-win-01"

$WebAppName = "app-web-lab-01"
$PlanName = "asp-web-lab-01"

$EvidencePath = ".\evidence\08-compute-troubleshooting"

New-Item -ItemType Directory -Force -Path $EvidencePath

# -----------------------------
# Context
# -----------------------------

Connect-AzAccount

Set-AzContext -SubscriptionId $SubscriptionId

Get-AzContext |
  Tee-Object (Join-Path $EvidencePath "az-context.txt")

# -----------------------------
# Resource inventory
# -----------------------------

Get-AzResource `
  -ResourceGroupName $ResourceGroupName |
  Tee-Object (Join-Path $EvidencePath "resource-list.txt")

Get-AzLog `
  -ResourceGroupName $ResourceGroupName `
  -MaxRecord 50 |
  Tee-Object (Join-Path $EvidencePath "activity-log.txt")

# -----------------------------
# Deployment troubleshooting
# -----------------------------

Get-AzResourceGroupDeployment `
  -ResourceGroupName $ResourceGroupName |
  Tee-Object (Join-Path $EvidencePath "deployment-list.txt")

Get-AzResourceGroupDeployment `
  -ResourceGroupName $ResourceGroupName `
  -Name $DeploymentName |
  Tee-Object (Join-Path $EvidencePath "deployment-show.txt")

Get-AzResourceGroupDeploymentOperation `
  -ResourceGroupName $ResourceGroupName `
  -DeploymentName $DeploymentName |
  Tee-Object (Join-Path $EvidencePath "deployment-operations.txt")

# -----------------------------
# VM troubleshooting
# -----------------------------

Get-AzVM `
  -ResourceGroupName $ResourceGroupName `
  -Name $VmName `
  -Status |
  Tee-Object (Join-Path $EvidencePath "vm-status.txt")

Invoke-AzVMRunCommand `
  -ResourceGroupName $ResourceGroupName `
  -VMName $LinuxVmName `
  -CommandId "RunShellScript" `
  -ScriptString "hostname; date; uptime; df -h; ip addr; systemctl status waagent --no-pager || true" |
  Tee-Object (Join-Path $EvidencePath "linux-run-command.txt")

Invoke-AzVMRunCommand `
  -ResourceGroupName $ResourceGroupName `
  -VMName $WindowsVmName `
  -CommandId "RunPowerShellScript" `
  -ScriptString "hostname; Get-Date; Get-NetIPConfiguration; Get-Service WindowsAzureGuestAgent; Get-EventLog -LogName System -Newest 20" |
  Tee-Object (Join-Path $EvidencePath "windows-run-command.txt")

# -----------------------------
# App Service troubleshooting
# -----------------------------

Get-AzAppServicePlan `
  -ResourceGroupName $ResourceGroupName `
  -Name $PlanName |
  Tee-Object (Join-Path $EvidencePath "appservice-plan.txt")

Get-AzWebApp `
  -ResourceGroupName $ResourceGroupName `
  -Name $WebAppName |
  Tee-Object (Join-Path $EvidencePath "webapp-show.txt")

# Use Azure CLI from PowerShell for detailed App Service and Container Apps evidence.

az webapp config show `
  --name $WebAppName `
  --resource-group $ResourceGroupName `
  --output json |
  Tee-Object (Join-Path $EvidencePath "webapp-config.json")

az webapp config appsettings list `
  --name $WebAppName `
  --resource-group $ResourceGroupName `
  --output json |
  Tee-Object (Join-Path $EvidencePath "webapp-appsettings.json")
```

# Troubleshoot_Azure_VM_Container_App_Service_Deployment_And_Runtime_Issues_VM_Troubleshooting_Skeleton

```text
Purpose:
Troubleshoot Azure VM provisioning, boot, access, extension, and guest runtime failures.

VM triage order:
1. Confirm VM exists.
2. Confirm provisioning state.
3. Confirm power state.
4. Confirm boot diagnostics.
5. Confirm VM agent.
6. Confirm network path.
7. Confirm extension state.
8. Confirm guest OS health.
9. Confirm last change.
10. Decide repair, redeploy, restore, or recreate.

Core commands:
az vm show -g <rg> -n <vm> --show-details -o json

az vm get-instance-view -g <rg> -n <vm> -o json

az vm boot-diagnostics get-boot-log -g <rg> -n <vm>

az vm run-command invoke \
  -g <rg> \
  -n <linux-vm> \
  --command-id RunShellScript \
  --scripts "hostname; uptime; df -h; ip addr; systemctl status waagent --no-pager || true"

az vm run-command invoke \
  -g <rg> \
  -n <windows-vm> \
  --command-id RunPowerShellScript \
  --scripts "hostname; Get-NetIPConfiguration; Get-Service WindowsAzureGuestAgent"

Access triage:
1. If public IP is used:
   - Confirm public IP exists.
   - Confirm NSG allows only approved source.
   - Confirm guest firewall allows port.
   - Confirm service is listening.

2. If private-only access is used:
   - Confirm Bastion or VPN path.
   - Confirm private IP.
   - Confirm effective NSG.
   - Confirm effective routes.

3. If no direct access:
   - Use Run Command.
   - Use boot diagnostics.
   - Use serial console if enabled and appropriate.

Extension triage:
1. List extensions.
2. Show failed extension.
3. Inspect provisioning message.
4. Confirm VM agent is ready.
5. Confirm script exits with code 0.
6. Confirm script download URI is reachable.
7. Confirm protected settings are valid.
8. Remove and reapply extension only after saving evidence.

Boot triage:
1. Read boot diagnostics.
2. Check recent OS-level change.
3. Check disk, fstab, bootloader, kernel, or Windows update.
4. Use serial console or repair VM workflow if guest cannot boot.
5. Restore from backup or snapshot if repair is not safe.

Safe fixes:
1. Restart VM for transient guest issue.
2. Reapply VM if model drift is suspected.
3. Redeploy VM for host issue.
4. Resize back if size change caused issue.
5. Restore OS disk if boot is corrupted.
6. Recreate lab VM if disposable.
```

# Troubleshoot_Azure_VM_Container_App_Service_Deployment_And_Runtime_Issues_Container_Apps_Troubleshooting_Skeleton

```text
Purpose:
Troubleshoot Azure Container Apps image pull, revision, replica, ingress, startup, probe, and scaling failures.

Container Apps triage order:
1. Confirm environment exists.
2. Confirm container app provisioning state.
3. Confirm current active revision.
4. Confirm replica status.
5. Confirm logs.
6. Confirm image name, tag, and registry auth.
7. Confirm target port and ingress.
8. Confirm CPU and memory.
9. Confirm secrets and environment variables.
10. Confirm scaling rules.

Core commands:
az containerapp show \
  -g <rg> \
  -n <container-app-name> \
  -o json

az containerapp revision list \
  -g <rg> \
  -n <container-app-name> \
  -o table

az containerapp replica list \
  -g <rg> \
  -n <container-app-name> \
  --revision <revision-name> \
  -o table

az containerapp logs show \
  -g <rg> \
  -n <container-app-name> \
  --follow false

az containerapp registry list \
  -g <rg> \
  -n <container-app-name> \
  -o table

az containerapp identity show \
  -g <rg> \
  -n <container-app-name> \
  -o json

Image pull triage:
1. Confirm ACR name.
2. Confirm repository exists.
3. Confirm image tag exists.
4. Confirm manifest exists.
5. Confirm container app registry server matches ACR login server.
6. Confirm identity exists.
7. Confirm identity has AcrPull on ACR.
8. Confirm image reference uses correct registry, repository, and tag.

Startup triage:
1. Check logs for first exception.
2. Confirm entrypoint or command.
3. Confirm environment variables.
4. Confirm secrets.
5. Confirm CPU and memory.
6. Confirm app does not exit immediately.
7. Confirm startup time is not longer than probe tolerance.

Ingress triage:
1. Confirm ingress is enabled.
2. Confirm external versus internal setting.
3. Confirm target port matches container listener.
4. Confirm app listens on 0.0.0.0, not just localhost.
5. Confirm FQDN.
6. Test endpoint with curl.
7. Check logs for request arrival.

Revision rollback:
1. List revisions.
2. Identify known-good active or inactive revision.
3. Activate known-good revision.
4. Adjust traffic if multiple revision mode is enabled.
5. Verify logs and endpoint.

Scaling triage:
1. Confirm min replicas.
2. Confirm max replicas.
3. Confirm scale rule syntax.
4. Confirm HTTP concurrency or event source.
5. Confirm no downstream throttling.
6. Confirm replicas are not crashing after scale-out.
```

# Troubleshoot_Azure_VM_Container_App_Service_Deployment_And_Runtime_Issues_App_Service_Troubleshooting_Skeleton

```text
Purpose:
Troubleshoot Azure App Service deployment, runtime, custom domain, TLS, slot, backup, networking, and container failures.

App Service triage order:
1. Confirm app exists and is running.
2. Confirm App Service Plan state and SKU.
3. Confirm runtime stack or container settings.
4. Confirm app settings and connection strings.
5. Confirm logs.
6. Confirm deployment status.
7. Confirm custom domain and TLS if endpoint issue.
8. Confirm access restrictions and private endpoint if network issue.
9. Confirm VNet integration if outbound dependency issue.
10. Confirm slot state if release issue.

Core commands:
az webapp show \
  -g <rg> \
  -n <web-app-name> \
  -o json

az webapp config show \
  -g <rg> \
  -n <web-app-name> \
  -o json

az webapp config appsettings list \
  -g <rg> \
  -n <web-app-name> \
  -o table

az webapp log config \
  -g <rg> \
  -n <web-app-name>

az webapp log tail \
  -g <rg> \
  -n <web-app-name>

az webapp deployment slot list \
  -g <rg> \
  -n <web-app-name> \
  -o table

Runtime triage:
1. Check HTTP status.
2. Check app logs.
3. Check runtime stack.
4. Check startup command.
5. Check app settings.
6. Check connection strings.
7. Check managed identity permissions.
8. Check dependency DNS and network path.

Deployment triage:
1. Check Deployment Center.
2. Check Kudu deployment logs.
3. Confirm package structure.
4. Confirm build provider.
5. Confirm startup file or command.
6. Redeploy known-good package if needed.

Custom domain triage:
1. Confirm hostname binding.
2. Confirm DNS CNAME or A record.
3. Confirm TXT verification record.
4. Confirm DNS resolves to expected endpoint.
5. Confirm certificate exists.
6. Confirm TLS binding.
7. Confirm HTTPS-only and minimum TLS.

Slot triage:
1. Confirm staging slot exists.
2. Confirm staging hostname responds.
3. Confirm slot app settings.
4. Confirm sticky settings.
5. Confirm recent swap.
6. Swap back if production broke immediately after swap.

Networking triage:
1. Access restrictions affect inbound access.
2. VNet integration affects outbound access.
3. Private endpoint affects inbound private access.
4. Private DNS is required for private endpoint name resolution.
5. SCM site can have separate access restrictions.
6. Confirm route-all if dependencies require forced tunneling.

Container web app triage:
1. Confirm image name and tag.
2. Confirm registry credentials or managed identity.
3. Confirm AcrPull role.
4. Confirm WEBSITES_PORT if container uses nonstandard port.
5. Confirm logs show container start.
6. Confirm app listens on expected port.
```

# Troubleshoot_Azure_VM_Container_App_Service_Deployment_And_Runtime_Issues_ARM_Deployment_Troubleshooting_Skeleton

```text
Purpose:
Troubleshoot Azure Resource Manager, Bicep, and template deployment failures.

Deployment triage order:
1. Confirm deployment scope.
2. Confirm subscription and resource group.
3. Confirm deployment name.
4. Inspect deployment status.
5. Inspect failed operation.
6. Read exact error code and target resource.
7. Check provider registration.
8. Check RBAC.
9. Check Azure Policy.
10. Check SKU, quota, region, name, and dependency.

Commands:
az deployment group show \
  -g <rg> \
  -n <deployment-name> \
  -o json

az deployment operation group list \
  -g <rg> \
  -n <deployment-name> \
  -o json

az deployment operation group list \
  -g <rg> \
  -n <deployment-name> \
  --query "[?properties.provisioningState=='Failed']" \
  -o json

az provider show \
  --namespace <provider-namespace> \
  --query registrationState \
  -o tsv

Common fixes:
1. AuthorizationFailed:
   - Use correct subscription.
   - Assign required RBAC role at the correct scope.

2. MissingSubscriptionRegistration:
   - Register provider.
   - Wait for registration.

3. InvalidTemplate:
   - Validate Bicep build.
   - Fix schema and property names.

4. InvalidResourceReference:
   - Fix resourceId.
   - Fix dependency order.
   - Use symbolic reference in Bicep where possible.

5. SkuNotAvailable:
   - Choose supported SKU.
   - Change region.
   - Change zone.

6. QuotaExceeded:
   - Reduce count or SKU.
   - Request quota increase.

7. RequestDisallowedByPolicy:
   - Read policy assignment.
   - Change template to satisfy policy.
   - Request exemption only if approved.

8. Conflict:
   - Existing resource state conflicts with desired change.
   - Check if property is immutable.
   - Recreate only if safe.

Evidence to save:
1. Deployment JSON.
2. Operation JSON.
3. Failed operation JSON.
4. What-if output.
5. Provider states.
6. Policy error details.
7. Corrected deployment command.
```

# Troubleshoot_Azure_VM_Container_App_Service_Deployment_And_Runtime_Issues_Network_And_DNS_Troubleshooting_Skeleton

```text
Purpose:
Troubleshoot compute network access, private endpoint, DNS, NSG, routes, access restrictions, and outbound dependency failures.

VM network triage:
1. Confirm NIC private IP.
2. Confirm public IP if used.
3. Confirm effective NSG.
4. Confirm effective route.
5. Confirm guest firewall.
6. Confirm service listener.
7. Confirm DNS from inside guest.
8. Confirm Bastion, VPN, ExpressRoute, or jump path.

VM commands:
az vm list-ip-addresses -g <rg> -n <vm> -o table

az network nic list-effective-nsg \
  -g <rg> \
  -n <nic-name> \
  -o table

az network nic show-effective-route-table \
  -g <rg> \
  -n <nic-name> \
  -o table

Container Apps network triage:
1. Confirm ingress external or internal.
2. Confirm target port.
3. Confirm FQDN.
4. Confirm app logs show requests.
5. Confirm environment networking if internal.
6. Confirm private DNS and internal client location.

App Service inbound triage:
1. Confirm public endpoint or private endpoint design.
2. Confirm access restrictions.
3. Confirm custom domain DNS.
4. Confirm TLS binding.
5. Confirm private endpoint DNS.
6. Confirm SCM access restrictions if deployment fails.

App Service outbound triage:
1. Confirm VNet integration.
2. Confirm integration subnet.
3. Confirm DNS resolver.
4. Confirm route all setting if needed.
5. Confirm firewall allows outbound dependency.
6. Confirm private endpoint for dependency.
7. Confirm Key Vault, database, storage, or ACR firewall rules.

DNS commands:
nslookup <hostname>

dig <hostname>

az network private-dns zone list -g <rg> -o table

az network private-dns record-set a list \
  -g <rg> \
  -z privatelink.azurewebsites.net \
  -o table

Endpoint commands:
curl -I https://<hostname>

curl -vk https://<hostname>

Network rule principle:
1. Inbound failure:
   - Check access restrictions, NSG, private endpoint, DNS, TLS.

2. Outbound failure:
   - Check VNet integration, route, DNS, firewall, private endpoint, identity.

3. Registry pull failure:
   - Check identity and AcrPull before assuming network.
```

# Troubleshoot_Azure_VM_Container_App_Service_Deployment_And_Runtime_Issues_Identity_And_Secrets_Troubleshooting_Skeleton

```text
Purpose:
Troubleshoot managed identity, ACR pull, Key Vault references, app settings, secrets, and RBAC failures.

Identity triage:
1. Confirm resource has identity enabled.
2. Capture principal ID.
3. Confirm role assignment scope.
4. Confirm role is correct.
5. Confirm token propagation delay is not the issue.
6. Confirm app or container uses the intended identity.
7. Confirm secret reference syntax.

VM identity:
az vm identity show \
  -g <rg> \
  -n <vm> \
  -o json

Container App identity:
az containerapp identity show \
  -g <rg> \
  -n <container-app-name> \
  -o json

App Service identity:
az webapp identity show \
  -g <rg> \
  -n <web-app-name> \
  -o json

Role assignment:
az role assignment list \
  --assignee <principal-id> \
  --all \
  -o table

ACR pull fix:
1. Get runtime principal ID.
2. Get ACR resource ID.
3. Assign AcrPull.

az role assignment create \
  --assignee <principal-id> \
  --role AcrPull \
  --scope <acr-resource-id>

Key Vault secret fix:
1. Confirm Key Vault permission model.
2. Assign Key Vault Secrets User or equivalent approved role.
3. Confirm secret exists and is enabled.
4. Confirm firewall allows trusted path or private path.
5. Restart app if secret cache needs refresh.

App Service Key Vault reference checks:
1. App setting value starts with:
   @Microsoft.KeyVault(SecretUri=...)

2. Identity is assigned.
3. Identity has secret get permission.
4. App can reach Key Vault if network restricted.
5. App configuration shows reference status in portal.

Container Apps secret checks:
1. Secret exists in container app.
2. Environment variable references correct secret name.
3. New revision created after secret/config change if required.
4. Logs do not expose secret value.
5. App is restarted or revision updated.

Common identity errors:
1. `unauthorized`
2. `forbidden`
3. `managed identity not found`
4. `secret not found`
5. `image pull unauthorized`
6. `KeyVaultReferenceException`

Safe fixes:
1. Do not paste secrets into logs.
2. Do not commit secrets to Git.
3. Prefer managed identity over static credentials.
4. Scope roles to the smallest resource required.
5. Rotate any exposed credential.
```

# Troubleshoot_Azure_VM_Container_App_Service_Deployment_And_Runtime_Issues_Verification_Commands

```bash
# Context
az account show -o table

# Resource inventory
az resource list \
  --resource-group "<resource-group-name>" \
  -o table

# Activity log
az monitor activity-log list \
  --resource-group "<resource-group-name>" \
  --max-events 50 \
  -o table

# Deployments
az deployment group list \
  --resource-group "<resource-group-name>" \
  -o table

az deployment group show \
  --resource-group "<resource-group-name>" \
  --name "<deployment-name>" \
  -o json

az deployment operation group list \
  --resource-group "<resource-group-name>" \
  --name "<deployment-name>" \
  -o table

az deployment operation group list \
  --resource-group "<resource-group-name>" \
  --name "<deployment-name>" \
  --query "[?properties.provisioningState=='Failed']" \
  -o json

# Providers
az provider show --namespace Microsoft.Compute --query registrationState -o tsv
az provider show --namespace Microsoft.Web --query registrationState -o tsv
az provider show --namespace Microsoft.App --query registrationState -o tsv
az provider show --namespace Microsoft.ContainerRegistry --query registrationState -o tsv
az provider show --namespace Microsoft.Network --query registrationState -o tsv

# VM
az vm show \
  --resource-group "<resource-group-name>" \
  --name "<vm-name>" \
  --show-details \
  -o json

az vm get-instance-view \
  --resource-group "<resource-group-name>" \
  --name "<vm-name>" \
  -o json

az vm boot-diagnostics get-boot-log \
  --resource-group "<resource-group-name>" \
  --name "<vm-name>"

az vm extension list \
  --resource-group "<resource-group-name>" \
  --vm-name "<vm-name>" \
  -o table

az vm run-command invoke \
  --resource-group "<resource-group-name>" \
  --name "<linux-vm-name>" \
  --command-id RunShellScript \
  --scripts "hostname; uptime; df -h; ip addr"

az vm run-command invoke \
  --resource-group "<resource-group-name>" \
  --name "<windows-vm-name>" \
  --command-id RunPowerShellScript \
  --scripts "hostname; Get-NetIPConfiguration; Get-Service WindowsAzureGuestAgent"

# VM network
az vm list-ip-addresses \
  --resource-group "<resource-group-name>" \
  --name "<vm-name>" \
  -o table

az network nic list-effective-nsg \
  --resource-group "<resource-group-name>" \
  --name "<nic-name>" \
  -o table

az network nic show-effective-route-table \
  --resource-group "<resource-group-name>" \
  --name "<nic-name>" \
  -o table

# ACR
az acr show \
  --name "<acr-name>" \
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

# Container Apps
az containerapp env show \
  --name "<container-app-env-name>" \
  --resource-group "<resource-group-name>" \
  -o json

az containerapp show \
  --name "<container-app-name>" \
  --resource-group "<resource-group-name>" \
  -o json

az containerapp revision list \
  --name "<container-app-name>" \
  --resource-group "<resource-group-name>" \
  -o table

az containerapp replica list \
  --name "<container-app-name>" \
  --resource-group "<resource-group-name>" \
  --revision "<revision-name>" \
  -o table

az containerapp logs show \
  --name "<container-app-name>" \
  --resource-group "<resource-group-name>" \
  --follow false

az containerapp identity show \
  --name "<container-app-name>" \
  --resource-group "<resource-group-name>" \
  -o json

az containerapp registry list \
  --name "<container-app-name>" \
  --resource-group "<resource-group-name>" \
  -o table

# App Service
az appservice plan show \
  --name "<app-service-plan-name>" \
  --resource-group "<resource-group-name>" \
  -o json

az webapp show \
  --name "<web-app-name>" \
  --resource-group "<resource-group-name>" \
  -o json

az webapp config show \
  --name "<web-app-name>" \
  --resource-group "<resource-group-name>" \
  -o json

az webapp config appsettings list \
  --name "<web-app-name>" \
  --resource-group "<resource-group-name>" \
  -o table

az webapp config hostname list \
  --webapp-name "<web-app-name>" \
  --resource-group "<resource-group-name>" \
  -o table

az webapp config ssl list \
  --resource-group "<resource-group-name>" \
  -o table

az webapp config access-restriction show \
  --name "<web-app-name>" \
  --resource-group "<resource-group-name>" \
  -o json

az webapp vnet-integration list \
  --name "<web-app-name>" \
  --resource-group "<resource-group-name>" \
  -o table

az webapp deployment slot list \
  --name "<web-app-name>" \
  --resource-group "<resource-group-name>" \
  -o table

az webapp log tail \
  --name "<web-app-name>" \
  --resource-group "<resource-group-name>"

# Identity and RBAC
az role assignment list \
  --assignee "<principal-id>" \
  --all \
  -o table

# DNS and endpoint
nslookup "<hostname>"

curl -I "https://<hostname>"

curl -vk "https://<hostname>"
```

# Troubleshoot_Azure_VM_Container_App_Service_Deployment_And_Runtime_Issues_Rollback

| Change | Rollback Command / Action | Risk |
|---|---|---|
| Bad Bicep or ARM deployment | Redeploy last known-good template and parameter file | May not undo destructive changes |
| Wrong resource group deployment | `az group delete --name "<wrong-resource-group-name>" --yes --no-wait` | Deletes all resources in that RG |
| Provider missing | `az provider register --namespace "<provider-namespace>"` | Registration may take time |
| VM bad size change | `az vm resize --resource-group "<resource-group-name>" --name "<vm-name>" --size "<old-size>"` | May require deallocation |
| VM stuck after host issue | `az vm redeploy --resource-group "<resource-group-name>" --name "<vm-name>"` | VM restarts on another host |
| VM model drift | `az vm reapply --resource-group "<resource-group-name>" --name "<vm-name>"` | Reapplies model but does not repair app code |
| VM extension bad | `az vm extension delete --resource-group "<resource-group-name>" --vm-name "<vm-name>" --name "<extension-name>"` | Guest changes made by extension may remain |
| VM public admin exposed | Delete SSH or RDP NSG rule | Admin access path may close |
| VM lab resource disposable | `az vm delete --resource-group "<resource-group-name>" --name "<vm-name>" --yes` | VM removed |
| Bad Container App image | `az containerapp update --name "<container-app-name>" --resource-group "<resource-group-name>" --image "<previous-image>"` | Creates new revision |
| Bad Container App revision | `az containerapp revision activate --name "<container-app-name>" --resource-group "<resource-group-name>" --revision "<known-good-revision>"` | Traffic behavior depends on revision mode |
| Bad Container App scale config | `az containerapp update --name "<container-app-name>" --resource-group "<resource-group-name>" --min-replicas "<old-min>" --max-replicas "<old-max>"` | Capacity changes immediately |
| Bad ACR role assignment | Remove incorrect role assignment and add correct AcrPull scope | Image pull may fail until fixed |
| Bad App Service deployment | Redeploy known-good package or image | Production remains impacted until deployment completes |
| Bad App Service slot swap | Run swap command again between staging and production | Production reverts to prior slot |
| Bad App Service app setting | Set previous value with `az webapp config appsettings set` | App may restart |
| Bad App Service custom domain | Remove hostname binding or fix DNS | Domain may be unavailable during propagation |
| Bad TLS binding | Bind previous certificate thumbprint | HTTPS may fail until cert is correct |
| Bad access restriction | Remove deny rule or add allow rule | Public exposure may temporarily increase |
| Bad VNet integration | `az webapp vnet-integration remove --name "<web-app-name>" --resource-group "<resource-group-name>" --vnet "<vnet-name>"` | Private outbound dependencies may fail |
| Bad private endpoint | Delete private endpoint or fix DNS | Inbound private access changes |
| Full lab cleanup | `az group delete --name "<resource-group-name>" --yes --no-wait` | Deletes all lab resources |

# Troubleshoot_Azure_VM_Container_App_Service_Deployment_And_Runtime_Issues_Failure_Checks

| Symptom | Likely Layer | First Check | Fix |
|---|---|---|---|
| Deployment fails with `AuthorizationFailed` | RBAC | Deployment operation message | Assign required role or target correct subscription |
| Deployment fails with `MissingSubscriptionRegistration` | Provider | Provider state | Register missing provider |
| Deployment fails with `RequestDisallowedByPolicy` | Azure Policy | Policy assignment and error | Change template to comply or request approved exemption |
| Deployment fails with `InvalidTemplate` | Template | Validation output | Fix Bicep or ARM schema |
| Deployment fails with `InvalidResourceReference` | Dependency | Failed operation details | Fix resource ID or dependency order |
| Deployment fails with `SkuNotAvailable` | Region/SKU | Error and SKU availability | Use supported SKU, region, or zone |
| Deployment fails with `QuotaExceeded` | Capacity | Quota limits | Request quota or reduce deployment |
| VM is running but inaccessible | Network | Effective NSG and route | Fix NSG, route, Bastion, or guest firewall |
| VM is stopped | Runtime | VM power state | Start VM |
| VM is deallocated | Runtime | VM power state | Start VM |
| VM boot fails | Guest OS | Boot diagnostics | Use serial console, repair disk, restore, or recreate |
| Run Command fails | VM agent | Instance view | Repair agent, reboot, or redeploy |
| Extension fails | Extension | Extension status | Fix script, settings, download URL, or agent |
| VM cannot resolve DNS | DNS | Guest `nslookup` | Fix VNet DNS or private DNS |
| VM cannot reach dependency | Routing or firewall | Guest network test | Fix route table, NSG, firewall, or private endpoint |
| ACR image tag missing | Registry | ACR tags | Build or push correct tag |
| ACR pull unauthorized | Identity/RBAC | Identity and role assignments | Assign AcrPull at ACR scope |
| Container App revision failed | App template | Revision and logs | Fix image, env vars, secrets, CPU/memory, or port |
| Container App replica restarts | Runtime crash | Replica status and logs | Fix app startup, memory, or dependency |
| Container App returns 502 | Ingress/port | Target port and logs | Set correct target port and listener |
| Container App scales to zero unexpectedly | Scale config | Min replicas | Set min replicas to 1 or more |
| Container App never scales out | Scale rule | Scale configuration | Fix rule and max replicas |
| App Service returns 500 | Application | App logs | Fix code, settings, connection strings, identity |
| App Service returns 502/503 | Startup or platform | Logs and plan metrics | Fix startup, scale plan, image, or runtime |
| App Service deploy fails | Deployment pipeline | Kudu or deployment logs | Fix package, build, or credentials |
| App Service container pull fails | Registry auth | Container config and identity | Assign AcrPull and configure registry |
| Custom domain not working | DNS | Hostname list and DNS lookup | Fix CNAME, A, TXT, or hostname binding |
| TLS certificate not served | SSL binding | SSL list and curl | Bind correct cert with SNI |
| HTTP still allowed | App config | HTTPS-only | Enable HTTPS-only |
| Private endpoint resolves public | DNS | Private DNS zone and nslookup | Link private DNS zone to VNet |
| Access restriction blocks valid traffic | Inbound access rules | Access restriction config | Add correct allow rule or adjust priority |
| App cannot reach Key Vault | Identity/network | Key Vault reference status | Fix identity role and network path |
| Slot swap broke production | Slot config | Slot settings and swap history | Swap back and fix sticky settings |
| Logs missing | Logging disabled | Log config | Enable logs and reproduce issue |
| Cost spike after troubleshooting | Resources left running | Resource list and plan capacity | Stop, scale down, or delete lab resources |

# Troubleshoot_Azure_VM_Container_App_Service_Deployment_And_Runtime_Issues_Related_Labs

| Lab                                                                 | Related Workbook                                                                         | Skill Proven                                                       |
| ------------------------------------------------------------------- | ---------------------------------------------------------------------------------------- | ------------------------------------------------------------------ |
| Deploy ARM and Bicep baseline                                       | `01_Deploy_ARM_Templates_Bicep_And_Exported_Deployments.md`                              | Deployment validation, operation logs, and rollback                |
| Create and manage Azure VMs                                         | `02_Create_Configure_And_Manage_Azure_Virtual_Machines.md`                               | VM lifecycle, identity, diagnostics, and access                    |
| Configure VM disks, sizes, availability, extensions, and encryption | `03_Configure_VM_Disks_Sizes_Availability_Sets_Zones_Extensions_And_Encryption.md`       | VM platform failure isolation                                      |
| Configure VMSS images and scaling                                   | `04_Configure_VMSS_Images_Scaling_And_Automation.md`                                     | Scale set image and instance troubleshooting                       |
| Deploy ACR, ACI, and Azure Container Apps                           | `05_Deploy_ACR_ACI_Azure_Container_Apps_Sizing_And_Scaling.md`                           | Container image, runtime, ingress, and scale diagnostics           |
| Deploy App Service plans and web apps                               | `06_Deploy_App_Service_Plans_Web_Apps_TLS_Custom_Domains_Backup_Networking_And_Slots.md` | App Service runtime, TLS, domain, slot, and networking diagnostics |
| Configure compute managed identities and Key Vault references       | `07_Configure_Compute_Managed_Identities_Key_Vault_Access_And_Secrets_References.md`     | Identity, ACR pull, and secret reference troubleshooting           |
| Troubleshoot compute deployment and runtime issues                  | `08_Troubleshoot_Azure_VM_Container_App_Service_Deployment_And_Runtime_Issues.md`        | End-to-end Azure compute incident triage                           |
