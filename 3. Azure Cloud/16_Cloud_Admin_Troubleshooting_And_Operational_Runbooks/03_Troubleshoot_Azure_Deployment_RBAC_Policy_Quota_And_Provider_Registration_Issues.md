# 03_Troubleshoot_Azure_Deployment_RBAC_Policy_Quota_And_Provider_Registration_Issues

## 03_Troubleshoot_Azure_Deployment_RBAC_Policy_Quota_And_Provider_Registration_Issues_Index

### Purpose

This workbook provides a structured troubleshooting workflow for Azure deployment failures involving Azure RBAC, Azure Policy, subscription quota, SKU availability, region availability, resource provider registration, deployment validation, ARM/Bicep/Terraform errors, and failed resource provisioning.

### Scope

Use this workbook when an Azure deployment fails with symptoms such as:

- Deployment failed in Azure portal, Azure CLI, Azure PowerShell, Bicep, ARM template, Terraform, GitHub Actions, or Azure DevOps
- Error says authorization failed, forbidden, denied, or missing permission
- Error says request disallowed by policy
- Error says SKU is unavailable in the selected region
- Error says quota exceeded
- Error says resource provider is not registered
- Error says namespace not found or provider not available
- Deployment appears stuck, canceled, timed out, or partially completed
- Resource group deployment succeeds but some resources are missing
- Template validation succeeds but deployment fails
- Deployment works for one admin but fails for another
- Deployment works in one subscription or region but not another
- Terraform plan succeeds but apply fails
- Azure Policy denies a deployment even though RBAC looks correct

### Assumptions

- You have read access to the subscription, resource group, and deployment operations.
- You can access Azure portal, Azure CLI, Azure PowerShell, or the IaC pipeline logs.
- You know the target subscription, resource group, region, and deployment name.
- You will not bypass policy or elevate RBAC until the exact failure gate is identified.
- You will record deployment evidence before retrying or deleting failed resources.

### Required Admin Roles

| Task Area | Minimum Role |
|---|---|
| View resource groups and deployments | Reader |
| View deployment operations | Reader |
| Deploy resources | Contributor or workload-specific role |
| Assign Azure RBAC | Owner or User Access Administrator |
| Register resource providers | Contributor or Owner at subscription scope, depending provider |
| View Azure Policy assignments | Reader |
| Create or modify policy assignments | Resource Policy Contributor or Owner |
| Exempt resource from policy | Resource Policy Contributor or Owner |
| View quotas | Reader usually enough for many quota views |
| Request quota increase | Contributor, Owner, or support request permissions depending subscription |
| Manage subscription | Owner or Account Administrator path depending subscription type |
| Troubleshoot pipelines | Repository and pipeline read access |

---

## 03_Troubleshoot_Azure_Deployment_RBAC_Policy_Quota_And_Provider_Registration_Issues_Mental_Model

Azure deployment failures usually come from one of six gates:

1. Identity gate: who or what is deploying.
2. Authorization gate: Azure RBAC at the target scope.
3. Governance gate: Azure Policy, deny assignments, locks, or blueprints/landing zone controls.
4. Platform gate: resource provider registration and supported API versions.
5. Capacity gate: quota, SKU, region, subscription offer, or availability zone support.
6. Template gate: invalid parameters, dependencies, naming, API schema, or resource state.

Do not troubleshoot deployment failures by randomly retrying. Every Azure deployment records operation-level failures. Read the deployment operations first.

Traditional troubleshooting order:

1. Confirm subscription, tenant, resource group, region, and deployment identity.
2. Read the failed deployment operation.
3. Capture the exact error code and target resource.
4. Check RBAC for the deploying principal at the exact scope.
5. Check deny assignments and locks.
6. Check Azure Policy result and assignment path.
7. Check provider registration.
8. Check quota and SKU availability.
9. Check template parameters, dependencies, names, and API versions.
10. Apply the smallest fix.
11. Redeploy with validation.
12. Record root cause and rollback.

---

## 03_Troubleshoot_Azure_Deployment_RBAC_Policy_Quota_And_Provider_Registration_Issues_Symptom_Map

| Symptom | Likely Cause | Primary Evidence | First Check |
|---|---|---|---|
| AuthorizationFailed | Missing Azure RBAC role at target scope | Deployment operation error | IAM Check access |
| Forbidden | RBAC missing, provider-level permission missing, deny assignment | Activity log / deployment operation | Principal role assignment |
| RequestDisallowedByPolicy | Azure Policy denies resource property, location, SKU, tags, public access, or naming | Policy evaluation details | Failed policy assignment |
| DeploymentFailed with nested error | Child resource failed inside parent deployment | Deployment operations | Expand failed operation |
| ResourceProviderNotRegistered | Provider namespace not registered | Deployment error | `az provider show` |
| MissingSubscriptionRegistration | Resource provider not registered | Deployment operation error | Register provider |
| QuotaExceeded | Subscription or regional quota limit hit | Deployment operation / quota blade | Usage + quotas |
| SkuNotAvailable | SKU unavailable in region or subscription offer | Deployment error | SKU list by location |
| InvalidTemplate | Template syntax, dependency, variable, or function error | Validation output | ARM/Bicep validation |
| InvalidResourceName | Name violates resource naming rules | Deployment operation | Target resource name |
| LocationNotAvailableForResourceType | Resource type not supported in region | Provider resource type locations | Provider info |
| Conflict | Resource already exists, name taken, state lock, or duplicate deployment | Deployment operation | Existing resource |
| ScopeLocked | Resource group/subscription/resource has CanNotDelete or ReadOnly lock | Activity log / lock blade | Locks |
| Role assignment creation fails | Missing `Microsoft.Authorization/roleAssignments/write` or bad principal ID | Deployment operation | RBAC of deployer |
| Key Vault secret access fails during deployment | Deploying principal lacks data-plane access or RBAC | Deployment operation | Key Vault access model |
| Terraform apply fails after plan | Plan account and apply account differ, policy denies, provider unregistered | Terraform log | Effective identity |
| GitHub Actions deployment fails | Federated identity, service principal role, secret, or subscription context issue | Workflow log | `az account show` |

---

## 03_Troubleshoot_Azure_Deployment_RBAC_Policy_Quota_And_Provider_Registration_Issues_Evidence_Collection

| Evidence | Where To Get It | Why It Matters |
|---|---|---|
| Tenant ID | Azure portal / CLI | Prevents wrong directory troubleshooting |
| Subscription ID | Azure portal / CLI | RBAC, quotas, and providers are subscription scoped |
| Resource group | Deployment scope | Most deployments target RG scope |
| Region | Deployment parameters | Region affects SKU, quota, policy, and provider support |
| Deployment name | Azure deployments blade | Needed to inspect operations |
| Correlation ID | Deployment error / Activity log | Needed for escalation |
| Error code | Deployment operation | Primary root-cause signal |
| Error message | Deployment operation | Shows denied action, policy, quota, or provider |
| Target resource | Deployment operation | Identifies failing resource, not just parent template |
| Deploying principal | Pipeline logs / Activity log | RBAC checks must target this identity |
| Principal object ID | Entra / Azure CLI | Role assignments use object IDs |
| Role assignments | IAM / CLI / PowerShell | Confirms authorization |
| Deny assignments | IAM / PowerShell | Deny beats allow |
| Locks | Resource / RG / subscription | Locks block updates/deletes |
| Policy assignments | Azure Policy | Denial source |
| Policy compliance details | Azure Policy / deployment error | Shows which rule denied |
| Provider registration state | Subscription resource providers | Confirms provider gate |
| Quota usage | Usage + quotas | Confirms capacity limit |
| SKU availability | CLI / portal | Confirms regional SKU support |
| Template file | Repo / deployment source | Needed to identify schema/parameter issue |
| Parameter file | Repo / pipeline logs | Bad parameters cause many failures |
| Pipeline log | GitHub Actions / Azure DevOps | Shows actual identity and command |
| Activity log | Azure Monitor | Captures authorization and policy events |

---

## 03_Troubleshoot_Azure_Deployment_RBAC_Policy_Quota_And_Provider_Registration_Issues_Triage_Order

1. Confirm deployment scope: tenant, management group, subscription, resource group, or resource.
2. Confirm the deploying identity: user, service principal, managed identity, federated workload identity, or pipeline connection.
3. Open the failed deployment and expand deployment operations.
4. Capture exact error code, message, target resource, and correlation ID.
5. Check whether the error is RBAC, policy, quota, provider registration, SKU, lock, or template validation.
6. Check RBAC at exact scope for the deploying principal.
7. Check deny assignments and locks.
8. Check Azure Policy denial and policy assignment path.
9. Check provider namespace registration state.
10. Check quota, SKU, region, and subscription offer limitations.
11. Check template parameters, dependency order, naming rules, and API versions.
12. Fix the smallest confirmed blocker.
13. Validate deployment before retrying where possible.
14. Redeploy.
15. Verify resource state, activity log, and policy compliance.
16. Record the incident, root cause, fix, and rollback.

---

## 03_Troubleshoot_Azure_Deployment_RBAC_Policy_Quota_And_Provider_Registration_Issues_Planning_Table

| Item | Value |
|---|---|
| Tenant name | `<tenant-name>` |
| Tenant ID | `<tenant-id>` |
| Subscription name | `<subscription-name>` |
| Subscription ID | `<subscription-id>` |
| Resource group | `<resource-group>` |
| Deployment scope | `<tenant / management group / subscription / resource group / resource>` |
| Deployment name | `<deployment-name>` |
| Region | `<region>` |
| Deploying identity | `<user / service principal / managed identity / workload identity>` |
| Deploying identity UPN/app name | `<name>` |
| Deploying identity object ID | `<object-id>` |
| IaC tool | `<portal / ARM / Bicep / Terraform / Azure CLI / Azure PowerShell / GitHub Actions / Azure DevOps>` |
| Template path | `<path>` |
| Parameter path | `<path>` |
| Error code | `<error-code>` |
| Correlation ID | `<correlation-id>` |
| Failed resource type | `<Microsoft.Provider/type>` |
| Failed resource name | `<resource-name>` |
| Policy assignment involved | `<policy-assignment-name>` |
| Provider namespace involved | `<Microsoft.Provider>` |
| Quota involved | `<quota-name>` |
| SKU involved | `<sku-name>` |
| Change ticket | `<ticket-id>` |

---

## 03_Troubleshoot_Azure_Deployment_RBAC_Policy_Quota_And_Provider_Registration_Issues_Configuration_Checklist

| Step | Task | Device | Command | PowerShell | Expected Result |
|---|---|---|---|---|---|
| 1 | Confirm active tenant and subscription | Admin workstation | `az account show --output table` | `Get-AzContext` | Context matches target tenant and subscription |
| 2 | Set correct subscription | Admin workstation | `az account set --subscription <subscription-id>` | `Set-AzContext -Subscription <subscription-id>` | Commands target correct subscription |
| 3 | Locate failed deployment | Admin workstation | `az deployment group list -g <resource-group> --output table` | `Get-AzResourceGroupDeployment -ResourceGroupName <resource-group>` | Failed deployment is identified |
| 4 | Show deployment error | Admin workstation | `az deployment group show -g <resource-group> -n <deployment-name>` | `Get-AzResourceGroupDeployment -ResourceGroupName <resource-group> -Name <deployment-name>` | Deployment state and error are visible |
| 5 | Expand deployment operations | Admin workstation | `az deployment operation group list -g <resource-group> -n <deployment-name> --output table` | `Get-AzResourceGroupDeploymentOperation -ResourceGroupName <resource-group> -DeploymentName <deployment-name>` | Failed child operation is identified |
| 6 | Capture full failed operation | Admin workstation | `az deployment operation group list -g <resource-group> -n <deployment-name> --query "[?properties.provisioningState=='Failed']"` | `Get-AzResourceGroupDeploymentOperation -ResourceGroupName <resource-group> -DeploymentName <deployment-name> | Where-Object {$_.ProvisioningState -eq "Failed"}` | Error code, message, and target resource are captured |
| 7 | Confirm deploying identity | Admin workstation | `az account show --query user` | `Get-AzContext | Select-Object Account,Tenant,Subscription` | Identity is known |
| 8 | Check RBAC at resource group scope | Admin workstation | `az role assignment list --assignee <principal-id-or-upn> --scope /subscriptions/<subscription-id>/resourceGroups/<resource-group> --all --output table` | `Get-AzRoleAssignment -ObjectId <principal-object-id> -Scope /subscriptions/<subscription-id>/resourceGroups/<resource-group>` | Principal has required role or does not |
| 9 | Check inherited RBAC | Admin workstation | `az role assignment list --assignee <principal-id-or-upn> --all --output table` | `Get-AzRoleAssignment -ObjectId <principal-object-id>` | Parent-scope role assignments are visible |
| 10 | Check role definition actions | Admin workstation | `az role definition list --name "<role-name>" --output json` | `Get-AzRoleDefinition -Name "<role-name>"` | Role includes required action |
| 11 | Check deny assignments | Admin workstation | `az rest --method get --url "https://management.azure.com/subscriptions/<subscription-id>/resourceGroups/<resource-group>/providers/Microsoft.Authorization/denyAssignments?api-version=2022-04-01"` | `Get-AzDenyAssignment -Scope /subscriptions/<subscription-id>/resourceGroups/<resource-group>` | No deny assignment blocks operation or blocker is identified |
| 12 | Check resource locks | Admin workstation | `az lock list --resource-group <resource-group> --output table` | `Get-AzResourceLock -ResourceGroupName <resource-group>` | No ReadOnly or CanNotDelete lock blocks operation |
| 13 | Check policy assignments at resource group | Admin workstation | `az policy assignment list --scope /subscriptions/<subscription-id>/resourceGroups/<resource-group> --output table` | `Get-AzPolicyAssignment -Scope /subscriptions/<subscription-id>/resourceGroups/<resource-group>` | Relevant policies are identified |
| 14 | Check policy assignments at subscription | Admin workstation | `az policy assignment list --scope /subscriptions/<subscription-id> --output table` | `Get-AzPolicyAssignment -Scope /subscriptions/<subscription-id>` | Subscription policies are identified |
| 15 | Check policy state for target resource | Admin workstation | `az policy state list --resource <resource-id> --output table` | `Get-AzPolicyState -ResourceId <resource-id>` | Policy compliance or denial context is visible |
| 16 | Check provider registration | Admin workstation | `az provider show --namespace Microsoft.<ProviderName> --query registrationState --output table` | `Get-AzResourceProvider -ProviderNamespace Microsoft.<ProviderName>` | Provider is Registered or NotRegistered |
| 17 | Register provider if approved | Admin workstation | `az provider register --namespace Microsoft.<ProviderName>` | `Register-AzResourceProvider -ProviderNamespace Microsoft.<ProviderName>` | Provider begins registering |
| 18 | Confirm provider registration complete | Admin workstation | `az provider show --namespace Microsoft.<ProviderName> --query registrationState --output table` | `Get-AzResourceProvider -ProviderNamespace Microsoft.<ProviderName> | Select-Object ProviderNamespace,RegistrationState` | Provider state is Registered |
| 19 | Check resource type locations | Admin workstation | `az provider show --namespace Microsoft.<ProviderName> --query "resourceTypes[?resourceType=='<type>'].locations"` | `(Get-AzResourceProvider -ProviderNamespace Microsoft.<ProviderName>).ResourceTypes | Where-Object {$_.ResourceTypeName -eq "<type>"}` | Target region supports resource type |
| 20 | Check VM SKU availability if compute related | Admin workstation | `az vm list-skus --location <region> --size <sku> --all --output table` | `Get-AzComputeResourceSku | Where-Object {$_.Name -eq "<sku>" -and $_.Locations -contains "<region>"}` | SKU is available or restricted |
| 21 | Check quota usage | Admin workstation | Azure portal > Subscription > Usage + quotas | `Get-AzVMUsage -Location <region>` | Quota headroom is confirmed |
| 22 | Validate ARM/Bicep before deploy | Admin workstation | `az deployment group validate -g <resource-group> -f <template-file> -p <parameters-file>` | `Test-AzResourceGroupDeployment -ResourceGroupName <resource-group> -TemplateFile <template-file> -TemplateParameterFile <parameters-file>` | Template validation passes or shows template errors |
| 23 | Run what-if before redeploy | Admin workstation | `az deployment group what-if -g <resource-group> -f <template-file> -p <parameters-file>` | `Get-AzResourceGroupDeploymentWhatIfResult -ResourceGroupName <resource-group> -TemplateFile <template-file> -TemplateParameterFile <parameters-file>` | Planned changes are understood |
| 24 | Fix confirmed root cause only | Admin workstation | Apply RBAC, policy, provider, quota, SKU, or template correction | Depends on fix | Minimal safe change is applied |
| 25 | Redeploy | Admin workstation | `az deployment group create -g <resource-group> -f <template-file> -p <parameters-file>` | `New-AzResourceGroupDeployment -ResourceGroupName <resource-group> -TemplateFile <template-file> -TemplateParameterFile <parameters-file>` | Deployment succeeds |
| 26 | Verify resource provisioning state | Admin workstation | `az resource show --ids <resource-id> --query properties.provisioningState` | `Get-AzResource -ResourceId <resource-id>` | Resource is Succeeded or expected state |
| 27 | Review Activity Log after remediation | Admin workstation | Azure portal > Monitor > Activity log | `Get-AzActivityLog -StartTime (Get-Date).AddHours(-2) -ResourceGroupName <resource-group>` | No remaining authorization, policy, or deployment errors |
| 28 | Record final state and rollback | Admin workstation | Update incident/change record | Not applicable | Evidence, fix, verification, and rollback are documented |

---

## 03_Troubleshoot_Azure_Deployment_RBAC_Policy_Quota_And_Provider_Registration_Issues_Detailed_Workflow

### Phase 1: Confirm Context And Scope

| Step | Task | Device | Command | PowerShell | Expected Result |
|---|---|---|---|---|---|
| 1 | Confirm current Azure account | Admin workstation | `az account show --output json` | `Get-AzContext` | Identity, tenant, and subscription are visible |
| 2 | Set target subscription | Admin workstation | `az account set --subscription <subscription-id>` | `Set-AzContext -Subscription <subscription-id>` | All commands target correct subscription |
| 3 | Confirm target resource group exists | Admin workstation | `az group show --name <resource-group> --output table` | `Get-AzResourceGroup -Name <resource-group>` | Resource group exists in expected location |
| 4 | Confirm deployment scope | Admin workstation | Identify whether deployment is tenant, management group, subscription, or resource group scoped | Not applicable | Correct deployment command family is used |
| 5 | Confirm deploying identity in pipeline | Pipeline runner | Add `az account show` step or inspect pipeline login output | Not applicable | Pipeline identity matches expected service principal or federated identity |
| 6 | Confirm object ID of deploying principal | Admin workstation | `az ad signed-in-user show --query id` or `az ad sp show --id <app-id> --query id` | Not applicable | Object ID is known for RBAC checks |

### Phase 2: Read Failed Deployment Operations

| Step | Task | Device | Command | PowerShell | Expected Result |
|---|---|---|---|---|---|
| 1 | List RG deployments | Admin workstation | `az deployment group list -g <resource-group> --output table` | `Get-AzResourceGroupDeployment -ResourceGroupName <resource-group>` | Failed deployment is located |
| 2 | Show deployment summary | Admin workstation | `az deployment group show -g <resource-group> -n <deployment-name> --output json` | `Get-AzResourceGroupDeployment -ResourceGroupName <resource-group> -Name <deployment-name>` | Provisioning state and top-level error are visible |
| 3 | List deployment operations | Admin workstation | `az deployment operation group list -g <resource-group> -n <deployment-name> --output json` | `Get-AzResourceGroupDeploymentOperation -ResourceGroupName <resource-group> -DeploymentName <deployment-name>` | Child operations are visible |
| 4 | Filter failed operations | Admin workstation | `az deployment operation group list -g <resource-group> -n <deployment-name> --query "[?properties.provisioningState=='Failed']"` | `Get-AzResourceGroupDeploymentOperation -ResourceGroupName <resource-group> -DeploymentName <deployment-name> | Where-Object {$_.ProvisioningState -eq "Failed"}` | Actual failed resource operation is isolated |
| 5 | Capture correlation ID | Admin workstation | Read deployment error details | Review deployment operation properties | Correlation ID is captured for escalation |
| 6 | Record exact error code | Admin workstation | Copy error code from operation | Copy error code from PowerShell object | Error class is known |
| 7 | Identify target resource ID | Admin workstation | Copy targetResource from failed operation | Copy TargetResource from failed operation | Exact failing resource is known |

### Phase 3: RBAC Troubleshooting

| Step | Task | Device | Command | PowerShell | Expected Result |
|---|---|---|---|---|---|
| 1 | Identify required Azure action | Admin workstation | Read error message for action like `Microsoft.X/write` | Not applicable | Missing action is known |
| 2 | Check role assignments at resource group scope | Admin workstation | `az role assignment list --assignee <principal-id-or-upn> --scope /subscriptions/<subscription-id>/resourceGroups/<resource-group> --all --output table` | `Get-AzRoleAssignment -ObjectId <principal-object-id> -Scope /subscriptions/<subscription-id>/resourceGroups/<resource-group>` | Required role is present or missing |
| 3 | Check role assignments at subscription scope | Admin workstation | `az role assignment list --assignee <principal-id-or-upn> --scope /subscriptions/<subscription-id> --all --output table` | `Get-AzRoleAssignment -ObjectId <principal-object-id> -Scope /subscriptions/<subscription-id>` | Inherited role is present or missing |
| 4 | Check role assignment through groups | Admin workstation | `az role assignment list --scope <scope> --all --output table` | `Get-AzRoleAssignment -Scope <scope>` | Group-based role path is identified |
| 5 | Confirm principal group membership | Admin workstation | `az ad user get-member-groups --id <user-upn>` or check Entra group membership | `Get-MgUserMemberOf -UserId <user-upn>` | Principal is in expected group |
| 6 | Check custom role definition | Admin workstation | `az role definition list --name "<custom-role-name>" --output json` | `Get-AzRoleDefinition -Name "<custom-role-name>"` | Custom role includes required actions |
| 7 | Check if deployment creates role assignments | Admin workstation | Search template for `Microsoft.Authorization/roleAssignments` | Not applicable | Deployer needs role assignment write permissions |
| 8 | Fix RBAC with least privilege | Admin workstation | `az role assignment create --assignee <principal-id> --role "<role>" --scope <scope>` | `New-AzRoleAssignment -ObjectId <principal-object-id> -RoleDefinitionName "<role>" -Scope <scope>` | Principal has required role |
| 9 | Re-authenticate if role was just added | Admin workstation | `az logout; az login --tenant <tenant-id>` | `Disconnect-AzAccount; Connect-AzAccount -Tenant <tenant-id>` | New token contains updated claims |
| 10 | Retry validation or deployment | Admin workstation | `az deployment group validate ...` then deploy | `Test-AzResourceGroupDeployment ...` then deploy | RBAC error is cleared |

### Phase 4: Deny Assignments And Locks

| Step | Task | Device | Command | PowerShell | Expected Result |
|---|---|---|---|---|---|
| 1 | Check resource group locks | Admin workstation | `az lock list --resource-group <resource-group> --output table` | `Get-AzResourceLock -ResourceGroupName <resource-group>` | Locks are listed |
| 2 | Check subscription locks | Admin workstation | `az lock list --scope /subscriptions/<subscription-id> --output table` | `Get-AzResourceLock -Scope /subscriptions/<subscription-id>` | Subscription locks are listed |
| 3 | Identify lock impact | Admin workstation | Review lock level: ReadOnly or CanNotDelete | Review lock object | Lock explains failed write/delete if present |
| 4 | Remove lock only with approval | Admin workstation | `az lock delete --ids <lock-id>` | `Remove-AzResourceLock -LockId <lock-id> -Force` | Lock no longer blocks deployment |
| 5 | Check deny assignments | Admin workstation | `az rest --method get --url "https://management.azure.com/<scope>/providers/Microsoft.Authorization/denyAssignments?api-version=2022-04-01"` | `Get-AzDenyAssignment -Scope <scope>` | Deny assignments are visible |
| 6 | Determine deny source | Admin workstation | Review deny assignment name and scope | Review deny assignment properties | Managed app, policy, or landing zone source is known |
| 7 | Escalate deny assignment if not removable | Admin workstation | Contact landing zone/subscription owner | Not applicable | Correct owner reviews deny source |
| 8 | Retry only after lock/deny path is resolved | Admin workstation | Redeploy | Redeploy | Lock or deny error is cleared |

### Phase 5: Azure Policy Troubleshooting

| Step | Task | Device | Command | PowerShell | Expected Result |
|---|---|---|---|---|---|
| 1 | Confirm policy denial in error | Admin workstation | Read error for `RequestDisallowedByPolicy` | Review deployment operation message | Policy denial is confirmed |
| 2 | Capture policy assignment name | Admin workstation | Copy assignment name from error details | Copy from operation status message | Assignment is known |
| 3 | List policy assignments at RG | Admin workstation | `az policy assignment list --scope /subscriptions/<subscription-id>/resourceGroups/<resource-group> --output table` | `Get-AzPolicyAssignment -Scope /subscriptions/<subscription-id>/resourceGroups/<resource-group>` | RG policy assignments are visible |
| 4 | List policy assignments at subscription | Admin workstation | `az policy assignment list --scope /subscriptions/<subscription-id> --output table` | `Get-AzPolicyAssignment -Scope /subscriptions/<subscription-id>` | Subscription policy assignments are visible |
| 5 | Check management group policy inheritance | Admin workstation | Azure portal > Management groups > Policy | `Get-AzPolicyAssignment -Scope /providers/Microsoft.Management/managementGroups/<mg-name>` | Inherited policy source is known |
| 6 | Review policy definition | Admin workstation | Azure Policy > Definitions > open policy | `Get-AzPolicyDefinition -Id <policy-definition-id>` | Deny condition is understood |
| 7 | Review initiative if applicable | Admin workstation | Azure Policy > Initiatives | `Get-AzPolicySetDefinition -Id <initiative-id>` | Initiative member policy is identified |
| 8 | Check policy state | Admin workstation | `az policy state list --resource <resource-id> --output table` | `Get-AzPolicyState -ResourceId <resource-id>` | Compliance state is visible |
| 9 | Compare template to policy rule | Admin workstation | Review denied field aliases such as location, SKU, tags, publicNetworkAccess | Not applicable | Required template change is known |
| 10 | Fix template to comply with policy | Admin workstation | Edit parameters or resource properties | Not applicable | Template no longer violates policy |
| 11 | Use exemption only when approved | Admin workstation | `az policy exemption create ...` | `New-AzPolicyExemption ...` | Exemption is scoped, temporary, and documented |
| 12 | Retest deployment | Admin workstation | `az deployment group what-if ...` and deploy | `Get-AzResourceGroupDeploymentWhatIfResult ...` and deploy | Policy denial is cleared |

### Phase 6: Resource Provider Registration Troubleshooting

| Step | Task | Device | Command | PowerShell | Expected Result |
|---|---|---|---|---|---|
| 1 | Identify provider namespace | Admin workstation | Extract from failed type: `Microsoft.Provider/resourceType` | Not applicable | Provider namespace is known |
| 2 | Check provider state | Admin workstation | `az provider show --namespace Microsoft.<ProviderName> --query registrationState --output table` | `Get-AzResourceProvider -ProviderNamespace Microsoft.<ProviderName>` | Registration state is visible |
| 3 | Register provider | Admin workstation | `az provider register --namespace Microsoft.<ProviderName>` | `Register-AzResourceProvider -ProviderNamespace Microsoft.<ProviderName>` | Provider starts registering |
| 4 | Wait for registration | Admin workstation | `az provider show --namespace Microsoft.<ProviderName> --query registrationState --output table` | `Get-AzResourceProvider -ProviderNamespace Microsoft.<ProviderName> | Select-Object RegistrationState` | Registration becomes Registered |
| 5 | Check resource type availability | Admin workstation | `az provider show --namespace Microsoft.<ProviderName> --query "resourceTypes[].resourceType"` | `(Get-AzResourceProvider -ProviderNamespace Microsoft.<ProviderName>).ResourceTypes.ResourceTypeName` | Resource type exists |
| 6 | Check supported locations | Admin workstation | `az provider show --namespace Microsoft.<ProviderName> --query "resourceTypes[?resourceType=='<type>'].locations"` | `(Get-AzResourceProvider -ProviderNamespace Microsoft.<ProviderName>).ResourceTypes | Where-Object {$_.ResourceTypeName -eq "<type>"}` | Target location is supported |
| 7 | Check API versions | Admin workstation | `az provider show --namespace Microsoft.<ProviderName> --query "resourceTypes[?resourceType=='<type>'].apiVersions"` | Review provider resource type API versions | Template API version is supported |
| 8 | Retry deployment | Admin workstation | Deploy again after provider is registered | Deploy again after provider is registered | Provider registration error is cleared |

### Phase 7: Quota, SKU, And Region Troubleshooting

| Step | Task | Device | Command | PowerShell | Expected Result |
|---|---|---|---|---|---|
| 1 | Identify quota from error | Admin workstation | Read error for quota name, family, or limit | Review failed operation | Quota category is known |
| 2 | Check regional VM quota | Admin workstation | Azure portal > Subscription > Usage + quotas | `Get-AzVMUsage -Location <region>` | Current usage and limit are visible |
| 3 | Check SKU availability | Admin workstation | `az vm list-skus --location <region> --size <sku> --all --output table` | `Get-AzComputeResourceSku | Where-Object {$_.Name -eq "<sku>" -and $_.Locations -contains "<region>"}` | SKU is available or restricted |
| 4 | Check zone support | Admin workstation | `az vm list-skus --location <region> --size <sku> --all --query "[].locationInfo"` | Inspect `LocationInfo` from compute SKU output | Zone support matches template |
| 5 | Check storage/service-specific quota | Admin workstation | Azure portal > Usage + quotas or service quota blade | Service-specific cmdlets vary | Limit is confirmed |
| 6 | Reduce requested capacity if possible | Admin workstation | Modify count, SKU, size, replicas, or region | Edit parameters | Deployment fits current quota |
| 7 | Change region if approved | Admin workstation | Update location parameter | Edit parameters | Region supports SKU and has quota |
| 8 | Request quota increase | Admin workstation | Azure portal > Usage + quotas > Request increase | Not applicable | Quota request submitted |
| 9 | Retry deployment after quota approval | Admin workstation | Redeploy | Redeploy | Quota error is cleared |

### Phase 8: Template, Parameters, And Dependency Troubleshooting

| Step | Task | Device | Command | PowerShell | Expected Result |
|---|---|---|---|---|---|
| 1 | Validate deployment | Admin workstation | `az deployment group validate -g <resource-group> -f <template-file> -p <parameters-file>` | `Test-AzResourceGroupDeployment -ResourceGroupName <resource-group> -TemplateFile <template-file> -TemplateParameterFile <parameters-file>` | Syntax and schema errors are shown |
| 2 | Run what-if | Admin workstation | `az deployment group what-if -g <resource-group> -f <template-file> -p <parameters-file>` | `Get-AzResourceGroupDeploymentWhatIfResult -ResourceGroupName <resource-group> -TemplateFile <template-file> -TemplateParameterFile <parameters-file>` | Planned changes are visible |
| 3 | Check parameter values | Admin workstation | Review parameter file and pipeline variables | Not applicable | Names, location, SKU, count, and IDs are correct |
| 4 | Check dependency order | Admin workstation | Review `dependsOn` or Bicep symbolic references | Not applicable | Parent resources deploy before children |
| 5 | Check existing resource conflicts | Admin workstation | `az resource list -g <resource-group> --output table` | `Get-AzResource -ResourceGroupName <resource-group>` | Existing conflicting resources are identified |
| 6 | Check global name availability | Admin workstation | Use service-specific name check command if available | Use service-specific cmdlet if available | Name is available |
| 7 | Check API version | Admin workstation | Compare template API version to provider API versions | Not applicable | API version is supported |
| 8 | Check secure parameter injection | Pipeline runner | Review secret/variable names in pipeline | Not applicable | Required secret values are present |
| 9 | Check Key Vault references | Admin workstation | Validate Key Vault name, secret URI, and deployer access | `Get-AzKeyVaultSecret -VaultName <kv-name> -Name <secret-name>` | Secret can be read if deployment requires it |
| 10 | Fix template or parameters | Admin workstation | Edit IaC source | Not applicable | Validation passes |
| 11 | Redeploy with same deployment name if safe | Admin workstation | `az deployment group create -g <resource-group> -n <deployment-name> -f <template-file> -p <parameters-file>` | `New-AzResourceGroupDeployment -Name <deployment-name> -ResourceGroupName <resource-group> -TemplateFile <template-file> -TemplateParameterFile <parameters-file>` | Deployment completes |
| 12 | Use a new deployment name if old operation state causes confusion | Admin workstation | `az deployment group create -g <resource-group> -n <new-deployment-name> -f <template-file> -p <parameters-file>` | `New-AzResourceGroupDeployment -Name <new-deployment-name> -ResourceGroupName <resource-group> -TemplateFile <template-file> -TemplateParameterFile <parameters-file>` | Fresh deployment history is created |

### Phase 9: Terraform And Pipeline-Specific Checks

| Step | Task | Device | Command | PowerShell | Expected Result |
|---|---|---|---|---|---|
| 1 | Confirm Terraform identity | Pipeline runner | `az account show --output json` before `terraform apply` | Not applicable | Apply identity is known |
| 2 | Confirm Terraform state backend access | Pipeline runner | `terraform init` | Not applicable | Backend initializes successfully |
| 3 | Validate Terraform config | Pipeline runner | `terraform validate` | Not applicable | Configuration is valid |
| 4 | Review plan output | Pipeline runner | `terraform plan -out=tfplan` | Not applicable | Planned operations are known |
| 5 | Apply with detailed logs if needed | Pipeline runner | `TF_LOG=INFO terraform apply tfplan` | Not applicable | Error is visible |
| 6 | Check Azure provider registration setting | Pipeline runner | Review provider block for registration behavior | Not applicable | Provider registration behavior is understood |
| 7 | Confirm service principal RBAC | Admin workstation | `az role assignment list --assignee <app-id-or-object-id> --all --output table` | `Get-AzRoleAssignment -ObjectId <principal-object-id>` | Pipeline principal has required access |
| 8 | Confirm GitHub OIDC federated credential | Admin workstation | Entra app registration > Federated credentials | Not applicable | Subject, issuer, and audience match workflow |
| 9 | Confirm pipeline subscription context | Pipeline runner | `az account show --query "{tenant:tenantId,subscription:id,user:user}"` | Not applicable | Pipeline targets expected subscription |
| 10 | Retry after Azure-side fix | Pipeline runner | Rerun failed job | Not applicable | Pipeline succeeds or new failure is isolated |

---

## 03_Troubleshoot_Azure_Deployment_RBAC_Policy_Quota_And_Provider_Registration_Issues_Verification_Commands

### Context Commands

| Purpose | Command |
|---|---|
| Show Azure CLI context | `az account show --output table` |
| List subscriptions | `az account list --output table` |
| Set subscription | `az account set --subscription <subscription-id>` |
| Show signed-in user | `az ad signed-in-user show --query "{displayName:displayName,userPrincipalName:userPrincipalName,id:id}"` |
| Show service principal | `az ad sp show --id <app-id>` |

### Deployment Commands

| Purpose | Command |
|---|---|
| List resource group deployments | `az deployment group list -g <resource-group> --output table` |
| Show deployment | `az deployment group show -g <resource-group> -n <deployment-name>` |
| List deployment operations | `az deployment operation group list -g <resource-group> -n <deployment-name> --output table` |
| Show failed deployment operations | `az deployment operation group list -g <resource-group> -n <deployment-name> --query "[?properties.provisioningState=='Failed']"` |
| Validate ARM/Bicep deployment | `az deployment group validate -g <resource-group> -f <template-file> -p <parameters-file>` |
| Run what-if | `az deployment group what-if -g <resource-group> -f <template-file> -p <parameters-file>` |
| Deploy ARM/Bicep | `az deployment group create -g <resource-group> -f <template-file> -p <parameters-file>` |

### RBAC Commands

| Purpose | Command |
|---|---|
| List role assignments for principal | `az role assignment list --assignee <principal-id-or-upn> --all --output table` |
| List role assignments at scope | `az role assignment list --assignee <principal-id-or-upn> --scope <scope> --all --output table` |
| List all role assignments at scope | `az role assignment list --scope <scope> --all --output table` |
| Show role definition | `az role definition list --name "<role-name>" --output json` |
| Create role assignment | `az role assignment create --assignee <principal-id> --role "<role-name>" --scope <scope>` |

### Policy Commands

| Purpose | Command |
|---|---|
| List policy assignments at subscription | `az policy assignment list --scope /subscriptions/<subscription-id> --output table` |
| List policy assignments at resource group | `az policy assignment list --scope /subscriptions/<subscription-id>/resourceGroups/<resource-group> --output table` |
| Show policy assignment | `az policy assignment show --name <assignment-name> --scope <scope>` |
| Show policy definition | `az policy definition show --name <definition-name>` |
| List policy state for resource | `az policy state list --resource <resource-id> --output table` |
| Trigger policy compliance scan | `az policy state trigger-scan --resource-group <resource-group>` |

### Provider Commands

| Purpose | Command |
|---|---|
| Show provider registration | `az provider show --namespace Microsoft.<ProviderName> --query registrationState --output table` |
| Register provider | `az provider register --namespace Microsoft.<ProviderName>` |
| List provider resource types | `az provider show --namespace Microsoft.<ProviderName> --query "resourceTypes[].resourceType" --output table` |
| Show provider locations for type | `az provider show --namespace Microsoft.<ProviderName> --query "resourceTypes[?resourceType=='<type>'].locations"` |
| Show API versions for type | `az provider show --namespace Microsoft.<ProviderName> --query "resourceTypes[?resourceType=='<type>'].apiVersions"` |

### Quota And SKU Commands

| Purpose | Command |
|---|---|
| List VM SKUs in region | `az vm list-skus --location <region> --output table` |
| Check specific VM SKU | `az vm list-skus --location <region> --size <sku> --all --output table` |
| Show regional usage list | `az vm list-usage --location <region> --output table` |
| Check storage account SKU support | `az storage account list-skus --output table` |
| Check App Service locations | `az appservice list-locations --sku <sku> --output table` |

### Azure PowerShell Verification

| Purpose | PowerShell |
|---|---|
| Show context | `Get-AzContext` |
| Set subscription | `Set-AzContext -Subscription <subscription-id>` |
| List RG deployments | `Get-AzResourceGroupDeployment -ResourceGroupName <resource-group>` |
| Show deployment | `Get-AzResourceGroupDeployment -ResourceGroupName <resource-group> -Name <deployment-name>` |
| List deployment operations | `Get-AzResourceGroupDeploymentOperation -ResourceGroupName <resource-group> -DeploymentName <deployment-name>` |
| Validate deployment | `Test-AzResourceGroupDeployment -ResourceGroupName <resource-group> -TemplateFile <template-file> -TemplateParameterFile <parameters-file>` |
| Run what-if | `Get-AzResourceGroupDeploymentWhatIfResult -ResourceGroupName <resource-group> -TemplateFile <template-file> -TemplateParameterFile <parameters-file>` |
| Deploy template | `New-AzResourceGroupDeployment -ResourceGroupName <resource-group> -TemplateFile <template-file> -TemplateParameterFile <parameters-file>` |
| List role assignments | `Get-AzRoleAssignment -ObjectId <principal-object-id>` |
| List role assignments at scope | `Get-AzRoleAssignment -ObjectId <principal-object-id> -Scope <scope>` |
| Show role definition | `Get-AzRoleDefinition -Name "<role-name>"` |
| Show locks | `Get-AzResourceLock -ResourceGroupName <resource-group>` |
| Show deny assignments | `Get-AzDenyAssignment -Scope <scope>` |
| Show provider state | `Get-AzResourceProvider -ProviderNamespace Microsoft.<ProviderName>` |
| Register provider | `Register-AzResourceProvider -ProviderNamespace Microsoft.<ProviderName>` |
| Show VM quota usage | `Get-AzVMUsage -Location <region>` |
| Show compute SKUs | `Get-AzComputeResourceSku | Where-Object {$_.Locations -contains "<region>"}` |

---

## 03_Troubleshoot_Azure_Deployment_RBAC_Policy_Quota_And_Provider_Registration_Issues_Known_Good_Baselines

### Common Deployment Scopes

| Scope | Azure CLI Deployment Command | PowerShell Deployment Command |
|---|---|---|
| Resource group | `az deployment group create -g <resource-group> -f <template-file> -p <parameters-file>` | `New-AzResourceGroupDeployment -ResourceGroupName <resource-group> -TemplateFile <template-file> -TemplateParameterFile <parameters-file>` |
| Subscription | `az deployment sub create -l <region> -f <template-file> -p <parameters-file>` | `New-AzSubscriptionDeployment -Location <region> -TemplateFile <template-file> -TemplateParameterFile <parameters-file>` |
| Management group | `az deployment mg create -m <management-group-id> -l <region> -f <template-file> -p <parameters-file>` | `New-AzManagementGroupDeployment -ManagementGroupId <management-group-id> -Location <region> -TemplateFile <template-file> -TemplateParameterFile <parameters-file>` |
| Tenant | `az deployment tenant create -l <region> -f <template-file> -p <parameters-file>` | `New-AzTenantDeployment -Location <region> -TemplateFile <template-file> -TemplateParameterFile <parameters-file>` |

### Common Error Codes And Meaning

| Error Code | Meaning | First Fix Path |
|---|---|---|
| AuthorizationFailed | Deployer lacks required action at scope | Check RBAC and role definition |
| RequestDisallowedByPolicy | Azure Policy denied deployment | Review failed policy assignment and template property |
| MissingSubscriptionRegistration | Resource provider not registered | Register provider namespace |
| ResourceProviderNotRegistered | Resource provider not registered | Register provider namespace |
| QuotaExceeded | Subscription/regional quota exceeded | Reduce request or request quota increase |
| SkuNotAvailable | SKU unavailable in region/subscription | Change SKU or region |
| InvalidTemplate | Template syntax, expression, schema, or dependency issue | Validate template and fix IaC |
| InvalidResourceName | Name violates resource rules | Fix name parameter |
| DeploymentFailed | Wrapper error; child operation contains real error | Expand deployment operations |
| Conflict | Existing resource, state conflict, or duplicate setting | Check existing resources and idempotency |
| ScopeLocked | Lock blocks operation | Review lock and remove with approval |
| LinkedAuthorizationFailed | Deployer can write target resource but lacks permission on linked resource | Assign access to linked scope |
| LocationNotAvailableForResourceType | Resource type not supported in region | Change region or type |
| NoRegisteredProviderFound | API version/resource type/location combination invalid | Check provider API versions and supported locations |
| PrincipalNotFound | Role assignment principal ID invalid or not replicated | Confirm object ID and wait for replication |
| RoleAssignmentExists | Duplicate deterministic role assignment name | Use stable GUID or skip duplicate creation |

### Common Least-Privilege Role Fixes

| Deployment Need | Typical Role |
|---|---|
| Deploy most resources in RG | Contributor at resource group |
| Assign RBAC during deployment | User Access Administrator or Owner at target scope |
| Read deployment operations | Reader |
| Manage policy assignments | Resource Policy Contributor |
| Register providers | Contributor or Owner at subscription |
| Create resource groups | Contributor at subscription |
| Manage network resources | Network Contributor |
| Manage VMs but not network | Virtual Machine Contributor plus needed network access |
| Manage Key Vault control plane | Key Vault Contributor |
| Read Key Vault secrets in RBAC mode | Key Vault Secrets User |
| Manage managed identities | Managed Identity Contributor |
| Assign managed identity to resource | Managed Identity Operator plus resource write role |

---

## 03_Troubleshoot_Azure_Deployment_RBAC_Policy_Quota_And_Provider_Registration_Issues_Rollback

### Rollback Principles

- Preserve failed deployment evidence before deleting anything.
- Do not remove policy assignments just to force a deployment through.
- Prefer template compliance over policy exemption.
- Prefer scoped temporary RBAC over subscription-wide Owner.
- Remove temporary RBAC after deployment if it is not part of the permanent design.
- Do not delete partially created resources until dependencies and state are reviewed.
- For Terraform, never manually delete state-managed resources without reconciling state.
- For ARM/Bicep, rerun idempotent deployment when safe instead of hand-editing resources.

### Rollback Table

| Change Made | Rollback Action | Verification |
|---|---|---|
| Added temporary Contributor | Remove role assignment after deployment | `az role assignment list --assignee <principal-id> --all` |
| Added temporary Owner/User Access Administrator | Remove role assignment immediately after role assignment task | IAM no longer shows temporary elevation |
| Removed resource lock | Recreate lock after deployment | `az lock list --resource-group <resource-group>` |
| Created policy exemption | Expire or delete exemption after approved window | Policy compliance returns to expected state |
| Registered resource provider | Usually no rollback required | Provider remains Registered |
| Changed region/SKU | Revert parameter to approved design if temporary | What-if shows intended design |
| Reduced replica/count for quota | Restore count after quota increase | Resource count matches design |
| Manually created missing dependency | Bring resource into IaC or remove after redeploy | Deployment is idempotent |
| Deleted failed partial resource | Redeploy from template | Resource state is Succeeded |
| Changed Terraform state | Back up and document state action; restore state backup if needed | `terraform plan` shows expected delta |
| Disabled policy assignment | Re-enable immediately after emergency deployment | Assignment state is active |

### Pre-Change Capture Commands

| Purpose | Command |
|---|---|
| Export failed deployment | `az deployment group show -g <resource-group> -n <deployment-name> > deployment_<deployment-name>.json` |
| Export failed operations | `az deployment operation group list -g <resource-group> -n <deployment-name> > deployment_operations_<deployment-name>.json` |
| Export role assignments | `az role assignment list --scope <scope> --all > role_assignments_before.json` |
| Export policy assignments | `az policy assignment list --scope <scope> > policy_assignments_before.json` |
| Export locks | `az lock list --scope <scope> > locks_before.json` |
| Export provider state | `az provider show --namespace Microsoft.<ProviderName> > provider_before.json` |
| Export resource list | `az resource list -g <resource-group> > resources_before.json` |

---

## 03_Troubleshoot_Azure_Deployment_RBAC_Policy_Quota_And_Provider_Registration_Issues_Failure_Checks

| Failure | Check | Fix |
|---|---|---|
| DeploymentFailed hides real issue | Expand deployment operations | Troubleshoot failed child operation |
| AuthorizationFailed despite Contributor | Deployment creates role assignments or uses linked scope | Add User Access Administrator or required role at linked scope |
| User has Owner but pipeline fails | Pipeline uses service principal or federated identity, not user | Check pipeline identity and assign RBAC to it |
| Role assignment just added but still denied | Token stale or propagation delay | Re-authenticate and wait briefly |
| RequestDisallowedByPolicy persists | Policy inherited from management group | Check parent scopes |
| Policy details unclear | Error message truncated | Use policy state and deployment operation JSON |
| Exemption does not work | Wrong scope, wrong policy assignment, expired exemption | Confirm exemption scope and assignment ID |
| Provider registered but error remains | Resource type location or API version unsupported | Check provider resource type locations and API versions |
| Quota looks available but deployment fails | Wrong region, wrong quota family, zone capacity issue | Check exact region/SKU/family |
| SKU available in list but cannot deploy | Subscription offer restriction or capacity restriction | Try alternate SKU/region or request support |
| VM quota exceeded | vCPU family quota, not total cores | Check family-specific quota |
| Role assignment principal not found | Object ID wrong or new service principal replication delay | Use object ID and wait for replication |
| Name already taken | Global service namespace conflict | Choose unique name |
| InvalidTemplate only in pipeline | Pipeline variables differ from local parameters | Print resolved parameters safely |
| Terraform plan succeeds but apply fails | Policy/RBAC checked only at apply or identity differs | Check apply identity and Azure deployment operations |
| Deployment stuck | Long-running resource or platform issue | Check Activity Log and resource provisioning state |
| Delete failed due to lock | RG/resource lock | Remove lock with approval |
| Delete failed due to dependencies | Child resources or references exist | Delete dependent resources in correct order |
| Key Vault reference fails | Secret missing or deployer lacks access | Check Key Vault access model and secret URI |
| Managed identity assignment fails | Missing Managed Identity Operator on identity | Assign required role on user-assigned identity |
| Network deployment fails | Missing Network Contributor or policy blocks public IP/NSG | Check RBAC and policy |

---

## 03_Troubleshoot_Azure_Deployment_RBAC_Policy_Quota_And_Provider_Registration_Issues_Escalation_Handoff

### Escalate To Azure Platform / Landing Zone Team When

- Policy is inherited from management group
- Deny assignments block deployment
- Subscription vending or landing zone automation owns the control
- Management group RBAC is involved
- Required region/SKU is blocked by enterprise policy
- Locks are intentionally managed by platform team
- Provider registration is restricted by subscription governance

### Escalate To Security / Governance Team When

- Policy blocks public IPs, public network access, insecure TLS, missing tags, or disallowed regions
- Deployment needs policy exemption
- Deployment tries to create privileged role assignments
- Deployment modifies Key Vault, managed identity, firewall, or diagnostic settings
- Emergency bypass is requested

### Escalate To Application / IaC Team When

- Template parameter values are wrong
- Dependency graph is invalid
- Resource names are not deterministic
- Terraform state does not match Azure
- Pipeline variables differ from local deployment
- Module version or provider version changed

### Escalate To Microsoft Support When

- SKU/capacity issue persists across supported regions
- Quota increase is blocked or urgent
- Provider registration fails repeatedly
- Deployment operation errors are inconsistent with visible state
- Resource is stuck in provisioning/deleting for extended period
- Correlation ID indicates platform-side failure

### Handoff Data To Include

| Field | Value |
|---|---|
| Incident ID | `<ticket-id>` |
| Tenant ID | `<tenant-id>` |
| Subscription ID | `<subscription-id>` |
| Resource group | `<resource-group>` |
| Deployment name | `<deployment-name>` |
| Deployment scope | `<scope>` |
| Region | `<region>` |
| Deploying identity | `<identity>` |
| Principal object ID | `<object-id>` |
| Error code | `<error-code>` |
| Error message | `<error-message>` |
| Correlation ID | `<correlation-id>` |
| Failed resource ID | `<resource-id>` |
| Provider namespace | `<provider>` |
| Policy assignment | `<policy-assignment>` |
| Quota/SKU | `<quota-or-sku>` |
| Activity log event | `<event-id>` |
| Template file | `<file-path>` |
| Parameter file | `<file-path>` |
| Remediation attempted | `<actions>` |
| Current blocker | `<blocker>` |
| Business impact | `<impact>` |

---

## 03_Troubleshoot_Azure_Deployment_RBAC_Policy_Quota_And_Provider_Registration_Issues_Incident_Record_Template

| Field | Entry |
|---|---|
| Date opened | `<yyyy-mm-dd>` |
| Opened by | `<name>` |
| Deployment owner | `<team>` |
| Subscription | `<subscription-name>` |
| Resource group | `<resource-group>` |
| Deployment name | `<deployment-name>` |
| Tool used | `<portal / CLI / PowerShell / Bicep / ARM / Terraform / pipeline>` |
| Symptom | `<symptom>` |
| Root cause category | `<RBAC / Policy / Quota / SKU / Provider / Lock / Template / Pipeline / Platform>` |
| Root cause | `<confirmed-root-cause>` |
| Evidence | `<deployment operation, activity log, policy state, command output>` |
| Fix applied | `<fix>` |
| Security impact | `<impact>` |
| Governance impact | `<impact>` |
| Rollback action | `<rollback>` |
| Verification performed | `<verification>` |
| Preventive action | `<preventive-action>` |
| Closed by | `<name>` |
| Date closed | `<yyyy-mm-dd>` |

---

## 03_Troubleshoot_Azure_Deployment_RBAC_Policy_Quota_And_Provider_Registration_Issues_Preventive_Controls

| Control | Target State |
|---|---|
| Deployment identity inventory | Every pipeline/service principal/managed identity is documented |
| Least privilege deployment roles | Deployment identities have scoped roles, not broad standing Owner |
| Provider registration baseline | Required provider namespaces are registered during subscription preparation |
| Policy-aware templates | Templates comply with landing zone policies by default |
| Preflight validation | Validate and what-if are run before production deployment |
| Quota precheck | Regional quotas checked before VM, AKS, App Service, or scale deployments |
| SKU precheck | SKU and zone support checked before deployment |
| Naming standards | Resource names validated before deployment |
| Parameter review | Environment-specific parameters are reviewed before deployment |
| Deployment evidence retention | Deployment operations and pipeline logs are retained |
| Temporary elevation cleanup | Temporary RBAC assignments are removed after deployment |
| Policy exemption governance | Exemptions are scoped, justified, approved, and time-bound |
| Terraform state protection | State is remote, locked, versioned, and backed up |
| Change log | Deployment changes record root cause, fix, and rollback |

---

## 03_Troubleshoot_Azure_Deployment_RBAC_Policy_Quota_And_Provider_Registration_Issues_Related_Labs

| Lab                                          | Purpose                                                |
| -------------------------------------------- | ------------------------------------------------------ |
| Deploy ARM template to resource group        | Practice deployment operations and failure review      |
| Deploy Bicep with parameters                 | Validate template and parameter troubleshooting        |
| Use Azure deployment what-if                 | Preview changes before deployment                      |
| Troubleshoot AuthorizationFailed             | Practice RBAC scope and role definition checks         |
| Configure Azure RBAC for deployment identity | Build least privilege deployment access                |
| Troubleshoot RequestDisallowedByPolicy       | Identify policy assignment and denied field            |
| Create Azure Policy exemption                | Practice approved temporary governance bypass          |
| Register Azure resource providers            | Resolve provider registration failures                 |
| Check Azure VM quota and SKU availability    | Prevent quota and capacity deployment failures         |
| Troubleshoot Terraform apply failure         | Compare state, plan, identity, and Azure operations    |
| Troubleshoot GitHub Actions Azure deployment | Validate OIDC identity, subscription context, and RBAC |
| Create deployment incident review record     | Document root cause, evidence, fix, and rollback       |