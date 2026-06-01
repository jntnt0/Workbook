Curated from current Microsoft Learn ARM, Azure CLI, RBAC, Policy, tag, lock, deployment, what-if, naming, limit, and move-resource documentation.  

ARM_Resource_Model.md
# ARM_Resource_Model_Configuration_Checklist
| Step | Phase | Task | Scope | Azure CLI / Az PowerShell Command | Expected Result |
|---:|---|---|---|---|---|
| 1 | Context | Confirm signed-in Azure identity | Tenant | `az account show --query "{user:user.name, tenant:tenantId, subscription:name}" --output table` | Correct tenant, user, and default subscription are shown |
| 2 | Context | Select the target subscription | Subscription | `az account set --subscription "<subscription-id-or-name>"` | CLI context points to the intended subscription |
| 3 | Context | Capture reusable variables | Local shell | `SUB_ID=$(az account show --query id -o tsv); LOCATION="<azure-region>"; RG="<resource-group-name>"; ENV="<dev-test-prod>"; APP="<app-name>"` | Variables are set for repeatable commands |
| 4 | Hierarchy | Identify management-group placement when applicable | Management group | `az account management-group list --query "[].{name:name, displayName:displayName}" -o table` | Target management group is known before policy/RBAC assignment |
| 5 | Naming | Validate naming pattern before creation | Resource / RG | `echo "rg-${APP}-${ENV}-${LOCATION}"` | Name follows organization convention and Azure naming constraints |
| 6 | Provider | Check required resource-provider registration state | Subscription | `az provider list --query "[?contains(namespace,'Microsoft.')].{Provider:namespace, State:registrationState}" -o table` | Required providers are visible and registration state is known |
| 7 | Provider | Register only required providers | Subscription | `az provider register --namespace "<Microsoft.ProviderNamespace>"` | Provider enters `Registering` or `Registered` state |
| 8 | Provider | Confirm provider locations and API versions | Subscription | `az provider show --namespace "<Microsoft.ProviderNamespace>" --query "resourceTypes[].{type:resourceType, locations:locations, apiVersions:apiVersions[0]}" -o table` | Resource type, supported regions, and API version are confirmed |
| 9 | Resource Group | Create resource group as lifecycle boundary | Resource group | `az group create --name "$RG" --location "$LOCATION" --tags Environment="$ENV" Application="$APP" ManagedBy="ARM"` | Resource group exists in target region with baseline tags |
| 10 | Resource Group | Verify resource group metadata location and tags | Resource group | `az group show --name "$RG" --query "{name:name, location:location, tags:tags}" -o table` | Resource group name, location, and tags match design |
| 11 | Tagging | Apply or merge standard tags | RG / resource / subscription | `RG_ID=$(az group show -n "$RG" --query id -o tsv); az tag update --resource-id "$RG_ID" --operation Merge --tags CostCenter="<cost-center>" Owner="<owner>" DataClass="<classification>"` | Existing tags are preserved and required tags are added |
| 12 | RBAC | Resolve assignee object ID | Scope target | `az ad user show --id "<user-upn>" --query id -o tsv` | User, group, service principal, or managed identity object ID is known |
| 13 | RBAC | Assign least-privilege role at the narrowest valid scope | Resource group | `az role assignment create --assignee "<principal-id-or-upn>" --role "<role-name-or-role-id>" --scope "$RG_ID"` | Principal receives required role only at intended scope |
| 14 | RBAC | Verify effective role assignment | Resource group | `az role assignment list --scope "$RG_ID" --assignee "<principal-id-or-upn>" --query "[].{role:roleDefinitionName, scope:scope}" -o table` | Expected role assignment appears at correct scope |
| 15 | Policy | Locate required built-in or custom policy definition | Subscription / MG | `az policy definition list --query "[?contains(displayName,'<keyword>')].{name:name, displayName:displayName, id:id}" -o table` | Correct policy definition ID is identified |
| 16 | Policy | Assign governance policy to intended scope | RG / subscription / MG | `az policy assignment create --name "<assignment-name>" --display-name "<display-name>" --scope "$RG_ID" --policy "<policy-definition-id>"` | Policy assignment exists and applies to child resources |
| 17 | Policy | Verify policy assignment | RG / subscription / MG | `az policy assignment list --scope "$RG_ID" --query "[].{name:name, displayName:displayName, enforcementMode:enforcementMode}" -o table` | Assignment appears with expected name and enforcement mode |
| 18 | IaC | Validate ARM/Bicep target scope before deployment | RG / subscription / MG / tenant | `az deployment group validate --resource-group "$RG" --template-file "<main.bicep-or-template.json>" --parameters "<params-file-or-inline-params>"` | Template validates without schema or parameter errors |
| 19 | IaC | Preview changes before deployment | Resource group | `az deployment group what-if --resource-group "$RG" --template-file "<main.bicep-or-template.json>" --parameters "<params-file-or-inline-params>"` | What-if output shows expected Create/Modify/Delete actions |
| 20 | IaC | Deploy resources to resource-group scope | Resource group | `az deployment group create --name "dep-${APP}-${ENV}-$(date +%Y%m%d%H%M%S)" --resource-group "$RG" --template-file "<main.bicep-or-template.json>" --parameters "<params-file-or-inline-params>"` | Deployment provisioning state is `Succeeded` |
| 21 | IaC | Deploy resources to subscription scope when creating RGs or subscription-level objects | Subscription | `az deployment sub create --name "dep-sub-${APP}-${ENV}" --location "$LOCATION" --template-file "<main.bicep-or-template.json>" --parameters "<params-file-or-inline-params>"` | Subscription-scope deployment succeeds |
| 22 | Inventory | List deployed resources and IDs | Resource group | `az resource list --resource-group "$RG" --query "[].{name:name, type:type, location:location, id:id}" -o table` | All expected resources exist and resource IDs are captured |
| 23 | Inventory | Confirm tags propagated where expected | Resource group | `az resource list --resource-group "$RG" --query "[].{name:name, type:type, tags:tags}" -o table` | Required tags are present on tag-supported resources |
| 24 | Locking | Apply delete protection after validation | Resource group / resource | `az lock create --name "lock-cannotdelete-${RG}" --lock-type CanNotDelete --resource-group "$RG"` | Resource group and child resources are protected from deletion |
| 25 | Locking | Verify inherited management locks | Resource group | `az lock list --resource-group "$RG" -o table` | Expected `CanNotDelete` or `ReadOnly` lock is visible |
| 26 | Operations | Review deployment history | Resource group | `az deployment group list --resource-group "$RG" --query "[].{name:name, state:properties.provisioningState, timestamp:properties.timestamp}" -o table` | Recent deployments and states are visible |
| 27 | Operations | Inspect failed deployment details when needed | Resource group | `az deployment operation group list --resource-group "$RG" --name "<deployment-name>" --query "[].{state:properties.provisioningState, status:properties.statusMessage}" -o table` | Failed operation and error reason are identifiable |
| 28 | Move Planning | Validate dependencies before moving resources | Resource group / subscription | `az resource list --resource-group "$RG" --query "[].{name:name, type:type, id:id}" -o table` | Parent resources and dependent resources are identified before move |
| 29 | Move Planning | Remove blocking locks before approved move operations | Resource group / resource | `az lock list --resource-group "$RG" -o table` | Locks are known before attempting resource moves |
| 30 | Move Execution | Move resources only after dependency and support checks | RG / subscription | `az resource move --destination-group "<target-rg>" --ids "<resource-id-1>" "<resource-id-2>"` | Supported resources move to destination scope; resource IDs change |
| 31 | Cost / Governance | Validate tag-based cost organization | Subscription / RG | `az resource list --tag CostCenter="<cost-center>" --query "[].{name:name, group:resourceGroup, type:type}" -o table` | Resources are discoverable by governance and cost tags |
| 32 | Limits | Check design against ARM limits before scale-out | Subscription / RG | `az group list --query "length([])" -o tsv` | Resource-group count and deployment scale are reviewed before expansion |
| 33 | Verification | Confirm no unexpected unmanaged resources exist | Resource group | `az resource list --resource-group "$RG" --query "[?tags.ManagedBy!='ARM'].{name:name,type:type,tags:tags}" -o table` | Exceptions are intentional or remediated |
| 34 | Rollback Prep | Export current resource-group template snapshot | Resource group | `az group export --name "$RG" > "${RG}-export.json"` | Current declarative snapshot is available for reference |
| 35 | Rollback | Remove governance lock before approved destructive rollback | Resource group | `LOCK_ID=$(az lock list --resource-group "$RG" --query "[?name=='lock-cannotdelete-${RG}'].id | [0]" -o tsv); az lock delete --ids "$LOCK_ID"` | Lock is removed only under approved rollback/change process |
| 36 | Rollback | Delete failed test resource group when lifecycle permits | Resource group | `az group delete --name "$RG" --yes --no-wait` | Resource group deletion is submitted asynchronously |
| 37 | Final Validation | Confirm final ARM resource-model state | Subscription / RG | `az group show --name "$RG" --query "{name:name, location:location, tags:tags, provisioningState:properties.provisioningState}" -o json` | Resource group is present, tagged, governed, and operational |
# ARM_Resource_Model_Verification_Commands
| Check | Command | Expected Result |
|---|---|---|
| Current subscription | `az account show --query "{name:name,id:id,tenant:tenantId}" -o table` | Correct subscription and tenant |
| Resource group exists | `az group show -n "$RG" -o table` | Resource group returned |
| Providers registered | `az provider list --query "[?registrationState!='Registered'].{Provider:namespace, State:registrationState}" -o table` | Required providers are registered |
| RBAC assignments | `az role assignment list --scope "$RG_ID" -o table` | Expected principals and roles |
| Policy assignments | `az policy assignment list --scope "$RG_ID" -o table` | Expected governance assignments |
| Locks | `az lock list --resource-group "$RG" -o table` | Expected lock state |
| Resources | `az resource list -g "$RG" -o table` | Expected deployed resources |
| Deployment history | `az deployment group list -g "$RG" -o table` | Recent deployment succeeded |
# ARM_Resource_Model_Failure_Checks
| Symptom | Likely Cause | Check | Corrective Action |
|---|---|---|---|
| Deployment fails with provider error | Resource provider not registered or still registering | `az provider show --namespace "<Microsoft.ProviderNamespace>" --query registrationState -o tsv` | Register provider and retry after regional registration is available |
| Deployment denied | Azure Policy deny effect | `az policy assignment list --scope "$RG_ID" -o table` | Update template to comply or request policy exemption |
| Authorization failure | Missing RBAC permission at target scope | `az role assignment list --assignee "<principal>" --all -o table` | Assign least-privilege role at correct scope |
| Resource name unavailable | Global or scoped name collision | `az resource list --name "<name>" -o table` | Choose compliant unique name |
| Tag update replaced existing tags | Used replace/create instead of merge | `az tag list --resource-id "$RG_ID"` | Reapply tags with `az tag update --operation Merge` |
| Delete blocked | Management lock inherited from parent scope | `az lock list --resource-group "$RG" -o table` | Remove lock only through approved change process |
| Move operation blocked | Unsupported resource type, missing dependency, or lock | `az resource list -g "$RG" -o table` and `az lock list -g "$RG" -o table` | Include dependencies, remove locks, or redesign move |
| What-if shows unexpected deletes | Complete mode or missing resources in template | `az deployment group what-if -g "$RG" --template-file "<file>"` | Use incremental mode or add intended resources to template |
