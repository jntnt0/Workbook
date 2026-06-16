# Troubleshoot_Azure_Governance_RBAC_Policy_Locks_And_Cost_Issues

# Troubleshoot_Azure_Governance_RBAC_Policy_Locks_And_Cost_Issues_Index
06_Troubleshoot_Azure_Governance_RBAC_Policy_Locks_And_Cost_Issues.md
Troubleshoot_Azure_Governance_RBAC_Policy_Locks_And_Cost_Issues
Troubleshoot_Azure_Governance_RBAC_Policy_Locks_And_Cost_Issues_Source_Basis
Troubleshoot_Azure_Governance_RBAC_Policy_Locks_And_Cost_Issues_Mental_Model
Troubleshoot_Azure_Governance_RBAC_Policy_Locks_And_Cost_Issues_Planning_Table
Troubleshoot_Azure_Governance_RBAC_Policy_Locks_And_Cost_Issues_Configuration_Checklist
Troubleshoot_Azure_Governance_RBAC_Policy_Locks_And_Cost_Issues_Portal_Skeleton
Troubleshoot_Azure_Governance_RBAC_Policy_Locks_And_Cost_Issues_Azure_CLI_Precheck_Skeleton
Troubleshoot_Azure_Governance_RBAC_Policy_Locks_And_Cost_Issues_PowerShell_Precheck_Skeleton
Troubleshoot_Azure_Governance_RBAC_Policy_Locks_And_Cost_Issues_Scope_And_Context_Skeleton
Troubleshoot_Azure_Governance_RBAC_Policy_Locks_And_Cost_Issues_RBAC_Troubleshooting_Skeleton
Troubleshoot_Azure_Governance_RBAC_Policy_Locks_And_Cost_Issues_Policy_Troubleshooting_Skeleton
Troubleshoot_Azure_Governance_RBAC_Policy_Locks_And_Cost_Issues_Lock_Troubleshooting_Skeleton
Troubleshoot_Azure_Governance_RBAC_Policy_Locks_And_Cost_Issues_Move_Operation_Troubleshooting_Skeleton
Troubleshoot_Azure_Governance_RBAC_Policy_Locks_And_Cost_Issues_Cost_Management_Troubleshooting_Skeleton
Troubleshoot_Azure_Governance_RBAC_Policy_Locks_And_Cost_Issues_Activity_Log_Correlation_Skeleton
Troubleshoot_Azure_Governance_RBAC_Policy_Locks_And_Cost_Issues_Evidence_Export_Skeleton
Troubleshoot_Azure_Governance_RBAC_Policy_Locks_And_Cost_Issues_Verification_Commands
Troubleshoot_Azure_Governance_RBAC_Policy_Locks_And_Cost_Issues_Rollback
Troubleshoot_Azure_Governance_RBAC_Policy_Locks_And_Cost_Issues_Failure_Checks
Troubleshoot_Azure_Governance_RBAC_Policy_Locks_And_Cost_Issues_Related_Labs

# Troubleshoot_Azure_Governance_RBAC_Policy_Locks_And_Cost_Issues_Source_Basis

| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Azure management groups overview | Troubleshooting inherited governance scope and subscription placement |
| Microsoft Learn | Azure Resource Manager resource groups | Resource group scope, resource IDs, tags, locks, and resource movement |
| Microsoft Learn | Move Azure resources to a new resource group or subscription | Move prerequisites, same-tenant requirement, lock behavior, provider registration, and changed resource IDs |
| Microsoft Learn | Azure RBAC overview | Role assignments, scope inheritance, security principals, and access failures |
| Microsoft Learn | Troubleshoot Azure RBAC | Authorization errors, propagation delay, missing permission, and assignment issues |
| Microsoft Learn | Best practices for Azure RBAC | Least privilege, privileged role review, direct user assignment cleanup, and group-based access |
| Microsoft Learn | Azure Policy overview | Policy definitions, initiatives, effects, compliance, remediation, and RBAC distinction |
| Microsoft Learn | Azure Policy remediation | Managed identity, remediation task, and deployIfNotExists or modify failures |
| Microsoft Learn | Azure Resource Manager locks | Delete locks, read-only locks, lock inheritance, and blocked operations |
| Microsoft Learn | Microsoft Cost Management budgets | Budget scope, actual alerts, forecasted alerts, alert recipients, and evaluation delay |
| Microsoft Learn | Cost alerts | Active and dismissed alerts, budget alert behavior, and supported alert types |
| Microsoft Learn | Azure Advisor cost recommendations | Advisor cost recommendations, underutilized resources, impact, and savings review |
| Azure CLI | `az account`, `az role assignment`, `az policy`, `az lock`, `az resource`, `az advisor`, `az monitor` | Troubleshooting inventory and evidence capture |
| Azure PowerShell | `Az.Accounts`, `Az.Resources`, `Az.PolicyInsights`, `Az.Advisor`, `Az.Monitor` | Troubleshooting, validation, and evidence export |

# Troubleshoot_Azure_Governance_RBAC_Policy_Locks_And_Cost_Issues_Mental_Model

| Concept | Operational Meaning |
|---|---|
| Scope first | Most governance failures are scope problems before they are tool problems |
| Tenant context | The signed-in account must be in the correct Microsoft Entra tenant |
| Subscription context | CLI, PowerShell, and portal must point to the expected subscription |
| Management group inheritance | RBAC and Policy at management group scope can affect every child subscription |
| RBAC failure | User or identity lacks the action permission needed at the effective scope |
| Policy failure | Resource state violates a policy rule or is blocked by a deny effect |
| Lock failure | Resource, resource group, subscription, or parent scope has a delete or read-only lock |
| Cost visibility failure | User lacks Cost Management access or is viewing the wrong billing or Azure RBAC scope |
| Budget alert failure | Budget threshold, scope, filter, recipient, or evaluation timing is wrong |
| Advisor gap | Advisor needs enough data, access, eligible resources, and supported recommendation rules |
| Move failure | Resource cannot move because of unsupported type, provider registration, tenant mismatch, lock, dependency, or changed ID impact |
| Activity log | First source for control plane errors, denial events, move attempts, policy events, and lock conflicts |
| Effective access | Final access after direct assignments, group assignments, and inherited assignments are considered |
| First rule | Prove tenant, subscription, scope, identity, and operation before changing anything |
| Blunt rule | RBAC answers who can do it; Policy answers whether the resource state is allowed; locks answer whether the operation is blocked anyway; Cost Management answers who can see or manage spend |

# Troubleshoot_Azure_Governance_RBAC_Policy_Locks_And_Cost_Issues_Planning_Table

| Item | Example | Decision |
|---|---|---|
| Tenant ID | `aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee` | `<tenant-id>` |
| Subscription ID | `11111111-2222-3333-4444-555555555555` | `<subscription-id>` |
| Management group ID | `corp-platform` | `<management-group-id>` |
| Resource group | `rg-platform-001` | `<resource-group-name>` |
| Resource ID | `/subscriptions/<id>/resourceGroups/<rg>/providers/<provider>/<type>/<name>` | `<resource-id>` |
| User UPN | `user@contoso.com` | `<user-upn>` |
| User object ID | `bbbbbbbb-cccc-dddd-eeee-ffffffffffff` | `<user-object-id>` |
| Group name | `AZ-RG-Platform-Contributors` | `<group-display-name>` |
| Group object ID | `cccccccc-dddd-eeee-ffff-111111111111` | `<group-object-id>` |
| Managed identity name | `mi-platform-001` | `<managed-identity-name>` |
| Managed identity object ID | `dddddddd-eeee-ffff-1111-222222222222` | `<managed-identity-object-id>` |
| Target scope | `/subscriptions/<subscription-id>/resourceGroups/<resource-group-name>` | `<target-scope>` |
| Failed operation | `Microsoft.Compute/virtualMachines/write` | `<failed-operation>` |
| Correlation ID | `xxxxxxxx-yyyy-zzzz-aaaa-bbbbbbbbbbbb` | `<correlation-id>` |
| Policy assignment name | `assign-baseline-governance` | `<policy-assignment-name>` |
| Policy definition ID | `/providers/Microsoft.Authorization/policyDefinitions/<guid>` | `<policy-definition-id>` |
| Lock name | `lock-rg-delete-protection` | `<lock-name>` |
| Budget name | `budget-platform-monthly-001` | `<budget-name>` |
| Budget scope | `/subscriptions/<subscription-id>` | `<budget-scope>` |
| Advisor category | `Cost` | `<advisor-category>` |
| Evidence path | `C:\Azure-Governance\06-Troubleshoot-Governance` | `<evidence-path>` |
| Incident number | `INC0000001` | `<incident-number>` |
| Rollback target | Last known good assignment, policy, lock, budget, or resource state | `<rollback-target>` |

# Troubleshoot_Azure_Governance_RBAC_Policy_Locks_And_Cost_Issues_Configuration_Checklist

| Step | Task | Device | Portal / Command | PowerShell | Expected Result |
|---:|---|---|---|---|---|
| 1 | Confirm user-reported symptom | Operator | Record exact error, portal blade, command, timestamp, and correlation ID | Same | Symptom is documented |
| 2 | Confirm tenant context | Cloud Shell | `az account tenant list -o table` | `Get-AzTenant` | Correct tenant is visible |
| 3 | Confirm subscription context | Cloud Shell | `az account show -o table` | `Get-AzContext` | Correct subscription is active |
| 4 | Confirm target resource exists | Cloud Shell | `az resource show --ids "<resource-id>"` | `Get-AzResource -ResourceId "<resource-id>"` | Target resource is valid |
| 5 | Confirm resource group exists | Cloud Shell | `az group show --name "<resource-group-name>"` | `Get-AzResourceGroup -Name "<resource-group-name>"` | Resource group is valid |
| 6 | Confirm management group placement | Cloud Shell | `az account management-group show --name "<management-group-id>" --expand --recurse` | `Get-AzManagementGroup -GroupName "<management-group-id>" -Expand -Recurse` | Subscription hierarchy is known |
| 7 | Review Activity Log | Portal / Cloud Shell | Activity log or `az monitor activity-log list` | `Get-AzActivityLog` | Failure event and correlation ID are found |
| 8 | Check exact RBAC at target scope | Cloud Shell | `az role assignment list --scope "<target-scope>" -o table` | `Get-AzRoleAssignment -Scope "<target-scope>"` | Direct assignments are visible |
| 9 | Check inherited RBAC | Cloud Shell | `az role assignment list --scope "<target-scope>" --include-inherited -o table` | Parent scope queries | Inherited access is visible |
| 10 | Check group membership | Cloud Shell | `az ad group member list --group "<group-object-id>"` | `Get-AzADGroupMember` | User membership is verified |
| 11 | Check privileged assignments | Cloud Shell | Query Owner, Contributor, User Access Administrator, RBAC Administrator | Filter `Get-AzRoleAssignment` | High-risk assignments are identified |
| 12 | Check policy assignments | Cloud Shell | `az policy assignment list --scope "<target-scope>" -o table` | `Get-AzPolicyAssignment -Scope "<target-scope>"` | Policy assignments are visible |
| 13 | Check inherited policy | Cloud Shell | Query management group, subscription, and RG policy assignments | Parent scope queries | Inherited policy impact is known |
| 14 | Check policy compliance | Cloud Shell | `az policy state list --resource "<resource-id>" -o table` | `Get-AzPolicyState -ResourceId "<resource-id>"` | Compliance state is known |
| 15 | Check remediation identity | Cloud Shell | `az policy assignment show --name "<policy-assignment-name>" --scope "<target-scope>" --query identity` | `Get-AzPolicyAssignment` | Assignment identity is known |
| 16 | Check remediation identity RBAC | Cloud Shell | `az role assignment list --assignee "<managed-identity-object-id>" -o table` | `Get-AzRoleAssignment -ObjectId "<managed-identity-object-id>"` | Remediation permissions are known |
| 17 | Check locks | Cloud Shell | `az lock list --resource-group "<resource-group-name>" -o table` | `Get-AzResourceLock -ResourceGroupName "<resource-group-name>"` | Lock blockers are visible |
| 18 | Check parent-scope locks | Cloud Shell | `az lock list --scope "<target-scope>"`; query subscription and RG | `Get-AzResourceLock -Scope "<scope>"` | Inherited lock blockers are visible |
| 19 | Check resource move prerequisites | Cloud Shell | Provider, tenant, move support, source and destination locks | PowerShell equivalent | Move blocker is identified |
| 20 | Check Cost Management access | Portal | Cost Management + Billing > Access control or scope access | `Get-AzRoleAssignment -Scope "<budget-scope>"` | User has correct cost visibility |
| 21 | Check budget scope and filters | Portal | Cost Management > Budgets | Manual review | Budget targets the expected costs |
| 22 | Check cost alerts | Portal | Cost Management > Cost alerts | Manual review | Active or dismissed alerts are visible |
| 23 | Check Advisor cost recommendations | Cloud Shell | `az advisor recommendation list --category Cost` | `Get-AzAdvisorRecommendation -Category Cost` | Recommendations are visible |
| 24 | Export troubleshooting evidence | Admin Workstation | Run evidence export skeleton | Run evidence export skeleton | Evidence package is complete |
| 25 | Apply narrow fix | Operator | RBAC, policy exclusion, lock removal, budget correction, or Advisor action | Same | Only the root cause is changed |
| 26 | Validate after fix | Cloud Shell | Repeat failing operation or verification command | Repeat PowerShell validation | Issue is resolved or next cause is isolated |

# Troubleshoot_Azure_Governance_RBAC_Policy_Locks_And_Cost_Issues_Portal_Skeleton

| Step | Portal Area | Action | Expected Result |
|---:|---|---|---|
| 1 | Azure Portal | Sign in as troubleshooting admin | Correct tenant session opens |
| 2 | Directories + subscriptions | Confirm selected directory | Portal is scoped to correct tenant |
| 3 | Subscriptions | Confirm target subscription | Subscription context is correct |
| 4 | Management groups | Confirm subscription placement | Parent governance path is known |
| 5 | Target resource | Open resource overview | Resource exists and current state is visible |
| 6 | Activity log | Filter by failed timestamp or correlation ID | Failed operation is found |
| 7 | Activity log | Review operation name, caller, status, substatus, and JSON details | Error category is known |
| 8 | Access control (IAM) | Check access for user, group, or managed identity | Effective access is reviewed |
| 9 | Access control (IAM) | Review role assignments and inherited assignments | Direct and parent scope access is known |
| 10 | Policy | Open Policy > Compliance | Compliance state is visible |
| 11 | Policy | Open Policy > Assignments | Active policy assignments are visible |
| 12 | Policy assignment | Review scope, exclusions, parameters, enforcement mode, and identity | Policy configuration is understood |
| 13 | Locks | Open locks at resource, resource group, and subscription | Blocking lock is found or ruled out |
| 14 | Resource group | Review move history and current resources | Move impact is known |
| 15 | Cost Management + Billing | Select target scope | Cost view is scoped correctly |
| 16 | Cost Management > Budgets | Review budget configuration | Thresholds, recipients, filters, and period are verified |
| 17 | Cost Management > Cost alerts | Review active and dismissed alerts | Alert state is known |
| 18 | Cost Management > Cost analysis | Group by service, resource group, resource, and tag | Cost driver is isolated |
| 19 | Advisor recommendations | Filter category Cost | Cost optimization items are visible |
| 20 | Documentation | Record root cause, evidence, fix, and validation result | Troubleshooting record is complete |

# Troubleshoot_Azure_Governance_RBAC_Policy_Locks_And_Cost_Issues_Azure_CLI_Precheck_Skeleton

```bash
# Run in Azure Cloud Shell or an admin workstation with Azure CLI.
# Purpose: capture the baseline state before troubleshooting governance issues.

TENANT_ID="<tenant-id>"
SUBSCRIPTION_ID="<subscription-id>"
MANAGEMENT_GROUP_ID="<management-group-id>"
RESOURCE_GROUP="<resource-group-name>"
RESOURCE_ID="<resource-id>"
TARGET_SCOPE="<target-scope>"
USER_UPN="<user-upn>"
GROUP_OBJECT_ID="<group-object-id>"
POLICY_ASSIGNMENT_NAME="<policy-assignment-name>"
BUDGET_SCOPE="/subscriptions/$SUBSCRIPTION_ID"
CORRELATION_ID="<correlation-id>"
EVIDENCE_PATH="$HOME/azure-governance/06-troubleshoot-governance"

mkdir -p "$EVIDENCE_PATH"

az login --tenant "$TENANT_ID"
az account set --subscription "$SUBSCRIPTION_ID"

# Confirm active context.
az account show \
  --query "{Name:name, SubscriptionId:id, TenantId:tenantId, State:state}" \
  -o json | tee "$EVIDENCE_PATH/active-context.json"

# Confirm visible subscriptions.
az account list --all \
  --query "[].{Name:name, SubscriptionId:id, TenantId:tenantId, State:state}" \
  -o json | tee "$EVIDENCE_PATH/subscription-list.json"

# Confirm management group hierarchy.
az account management-group show \
  --name "$MANAGEMENT_GROUP_ID" \
  --expand \
  --recurse \
  -o json | tee "$EVIDENCE_PATH/management-group-tree.json"

# Confirm resource group.
az group show \
  --name "$RESOURCE_GROUP" \
  -o json | tee "$EVIDENCE_PATH/resource-group.json"

# Confirm resource.
az resource show \
  --ids "$RESOURCE_ID" \
  -o json | tee "$EVIDENCE_PATH/resource.json"

# Capture Activity Log events around the target resource group.
az monitor activity-log list \
  --resource-group "$RESOURCE_GROUP" \
  --max-events 100 \
  -o json | tee "$EVIDENCE_PATH/activity-log-resource-group.json"

# Capture Activity Log by correlation ID if available.
az monitor activity-log list \
  --correlation-id "$CORRELATION_ID" \
  -o json | tee "$EVIDENCE_PATH/activity-log-correlation-id.json"

# Capture RBAC at exact and inherited scope.
az role assignment list \
  --scope "$TARGET_SCOPE" \
  -o json | tee "$EVIDENCE_PATH/rbac-exact-scope.json"

az role assignment list \
  --scope "$TARGET_SCOPE" \
  --include-inherited \
  -o json | tee "$EVIDENCE_PATH/rbac-include-inherited.json"

# Capture user and group state.
az ad user show \
  --id "$USER_UPN" \
  -o json | tee "$EVIDENCE_PATH/user-object.json"

az ad group member list \
  --group "$GROUP_OBJECT_ID" \
  -o json | tee "$EVIDENCE_PATH/group-members.json"

# Capture policy assignments and policy state.
az policy assignment list \
  --scope "$TARGET_SCOPE" \
  -o json | tee "$EVIDENCE_PATH/policy-assignments-target-scope.json"

az policy state list \
  --resource "$RESOURCE_ID" \
  -o json | tee "$EVIDENCE_PATH/policy-state-resource.json"

# Capture locks.
az lock list \
  --resource-group "$RESOURCE_GROUP" \
  -o json | tee "$EVIDENCE_PATH/resource-group-locks.json"

az lock list \
  --scope "$TARGET_SCOPE" \
  -o json | tee "$EVIDENCE_PATH/target-scope-locks.json"

# Capture Advisor cost recommendations.
az advisor recommendation list \
  --category Cost \
  -o json | tee "$EVIDENCE_PATH/advisor-cost-recommendations.json"

# Capture resource tags for cost and policy troubleshooting.
az resource list \
  --query "[].{Name:name, Type:type, ResourceGroup:resourceGroup, Tags:tags, Id:id}" \
  -o json | tee "$EVIDENCE_PATH/resource-tags.json"
```

# Troubleshoot_Azure_Governance_RBAC_Policy_Locks_And_Cost_Issues_PowerShell_Precheck_Skeleton

```powershell
# Run in elevated PowerShell or Azure Cloud Shell.
# Purpose: capture the baseline state before troubleshooting governance issues.

$TenantId = "<tenant-id>"
$SubscriptionId = "<subscription-id>"
$ManagementGroupId = "<management-group-id>"
$ResourceGroupName = "<resource-group-name>"
$ResourceId = "<resource-id>"
$TargetScope = "<target-scope>"
$UserPrincipalName = "<user-upn>"
$GroupObjectId = "<group-object-id>"
$PolicyAssignmentName = "<policy-assignment-name>"
$CorrelationId = "<correlation-id>"
$EvidencePath = "C:\Azure-Governance\06-Troubleshoot-Governance"

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Import-Module Az.Accounts
Import-Module Az.Resources

Connect-AzAccount -Tenant $TenantId
Set-AzContext -SubscriptionId $SubscriptionId

# Confirm context.
Get-AzContext |
  Select-Object Name,Subscription,Tenant,Account |
  Tee-Object "$EvidencePath\active-context.txt"

# Confirm tenant and subscriptions.
Get-AzTenant |
  Select-Object Name,Id |
  Export-Csv "$EvidencePath\tenant-list.csv" -NoTypeInformation

Get-AzSubscription |
  Select-Object Name,Id,TenantId,State |
  Export-Csv "$EvidencePath\subscription-list.csv" -NoTypeInformation

# Confirm management group hierarchy.
Get-AzManagementGroup -GroupName $ManagementGroupId -Expand -Recurse |
  ConvertTo-Json -Depth 30 |
  Out-File "$EvidencePath\management-group-tree.json"

# Confirm resource group and resource.
Get-AzResourceGroup -Name $ResourceGroupName |
  ConvertTo-Json -Depth 10 |
  Out-File "$EvidencePath\resource-group.json"

Get-AzResource -ResourceId $ResourceId |
  Select-Object Name,ResourceType,Location,ResourceGroupName,ResourceId,Tags |
  ConvertTo-Json -Depth 10 |
  Out-File "$EvidencePath\resource.json"

# Capture Activity Log.
Get-AzActivityLog -ResourceGroupName $ResourceGroupName -MaxRecord 100 |
  Select-Object EventTimestamp,OperationName,ResourceGroupName,ResourceProviderName,Status,SubStatus,Caller,CorrelationId |
  Export-Csv "$EvidencePath\activity-log-resource-group.csv" -NoTypeInformation

Get-AzActivityLog -CorrelationId $CorrelationId -ErrorAction SilentlyContinue |
  Select-Object EventTimestamp,OperationName,ResourceGroupName,ResourceProviderName,Status,SubStatus,Caller,CorrelationId |
  Export-Csv "$EvidencePath\activity-log-correlation-id.csv" -NoTypeInformation

# Capture RBAC.
Get-AzRoleAssignment -Scope $TargetScope |
  Select-Object DisplayName,SignInName,ObjectType,RoleDefinitionName,Scope,ObjectId |
  Export-Csv "$EvidencePath\rbac-exact-scope.csv" -NoTypeInformation

Get-AzRoleAssignment -ObjectId (Get-AzADUser -UserPrincipalName $UserPrincipalName).Id -ErrorAction SilentlyContinue |
  Select-Object DisplayName,SignInName,ObjectType,RoleDefinitionName,Scope,ObjectId |
  Export-Csv "$EvidencePath\rbac-user-assignments.csv" -NoTypeInformation

# Capture group membership.
Get-AzADGroupMember -GroupObjectId $GroupObjectId -ErrorAction SilentlyContinue |
  Select-Object DisplayName,UserPrincipalName,Id,ObjectType |
  Export-Csv "$EvidencePath\group-members.csv" -NoTypeInformation

# Capture policy.
Get-AzPolicyAssignment -Scope $TargetScope -ErrorAction SilentlyContinue |
  ConvertTo-Json -Depth 30 |
  Out-File "$EvidencePath\policy-assignments-target-scope.json"

if (Get-Module -ListAvailable -Name Az.PolicyInsights) {
  Import-Module Az.PolicyInsights

  Get-AzPolicyState -ResourceId $ResourceId -ErrorAction SilentlyContinue |
    Select-Object Timestamp,ResourceId,PolicyAssignmentName,PolicyDefinitionName,ComplianceState |
    Export-Csv "$EvidencePath\policy-state-resource.csv" -NoTypeInformation
}

# Capture locks.
Get-AzResourceLock -ResourceGroupName $ResourceGroupName -ErrorAction SilentlyContinue |
  Export-Csv "$EvidencePath\resource-group-locks.csv" -NoTypeInformation

Get-AzResourceLock -Scope $TargetScope -ErrorAction SilentlyContinue |
  Export-Csv "$EvidencePath\target-scope-locks.csv" -NoTypeInformation

# Capture resources and tags.
Get-AzResource |
  Select-Object Name,ResourceType,ResourceGroupName,Location,ResourceId,Tags |
  Export-Csv "$EvidencePath\resource-tags.csv" -NoTypeInformation
```

# Troubleshoot_Azure_Governance_RBAC_Policy_Locks_And_Cost_Issues_Scope_And_Context_Skeleton

```powershell
# Purpose: prove tenant, subscription, management group, resource group, and resource scope before troubleshooting.

$TenantId = "<tenant-id>"
$SubscriptionId = "<subscription-id>"
$ManagementGroupId = "<management-group-id>"
$ResourceGroupName = "<resource-group-name>"
$ResourceId = "<resource-id>"
$EvidencePath = "C:\Azure-Governance\06-Troubleshoot-Governance"

Connect-AzAccount -Tenant $TenantId
Set-AzContext -SubscriptionId $SubscriptionId

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\scope-context-troubleshooting-transcript.txt"

# Confirm current context.
Get-AzContext |
  Format-List |
  Out-File "$EvidencePath\scope-context-current.txt"

# Confirm subscription tenant association.
(Get-AzSubscription -SubscriptionId $SubscriptionId).TenantId |
  Tee-Object "$EvidencePath\subscription-tenant-id.txt"

# Confirm management group path.
Get-AzManagementGroup -GroupName $ManagementGroupId -Expand -Recurse |
  ConvertTo-Json -Depth 30 |
  Out-File "$EvidencePath\scope-context-management-group-tree.json"

# Confirm resource group metadata.
Get-AzResourceGroup -Name $ResourceGroupName |
  Select-Object ResourceGroupName,Location,ProvisioningState,ResourceId,Tags |
  Tee-Object "$EvidencePath\scope-context-resource-group.txt"

# Confirm resource metadata.
Get-AzResource -ResourceId $ResourceId |
  Select-Object Name,ResourceType,Location,ResourceGroupName,ResourceId,Tags |
  Tee-Object "$EvidencePath\scope-context-resource.txt"

# Derive common scopes.
[PSCustomObject]@{
  TenantId          = $TenantId
  SubscriptionScope = "/subscriptions/$SubscriptionId"
  ResourceGroupScope = "/subscriptions/$SubscriptionId/resourceGroups/$ResourceGroupName"
  ResourceScope    = $ResourceId
  ManagementGroupScope = "/providers/Microsoft.Management/managementGroups/$ManagementGroupId"
} | Export-Csv "$EvidencePath\derived-scopes.csv" -NoTypeInformation

Stop-Transcript
```

```bash
# Azure CLI equivalent.

TENANT_ID="<tenant-id>"
SUBSCRIPTION_ID="<subscription-id>"
MANAGEMENT_GROUP_ID="<management-group-id>"
RESOURCE_GROUP="<resource-group-name>"
RESOURCE_ID="<resource-id>"
EVIDENCE_PATH="$HOME/azure-governance/06-troubleshoot-governance"

mkdir -p "$EVIDENCE_PATH"

az login --tenant "$TENANT_ID"
az account set --subscription "$SUBSCRIPTION_ID"

az account show \
  -o json | tee "$EVIDENCE_PATH/scope-context-current.json"

az account show \
  --subscription "$SUBSCRIPTION_ID" \
  --query tenantId \
  -o tsv | tee "$EVIDENCE_PATH/subscription-tenant-id.txt"

az account management-group show \
  --name "$MANAGEMENT_GROUP_ID" \
  --expand \
  --recurse \
  -o json | tee "$EVIDENCE_PATH/scope-context-management-group-tree.json"

az group show \
  --name "$RESOURCE_GROUP" \
  -o json | tee "$EVIDENCE_PATH/scope-context-resource-group.json"

az resource show \
  --ids "$RESOURCE_ID" \
  -o json | tee "$EVIDENCE_PATH/scope-context-resource.json"

cat > "$EVIDENCE_PATH/derived-scopes.txt" <<EOF
TenantId=$TENANT_ID
SubscriptionScope=/subscriptions/$SUBSCRIPTION_ID
ResourceGroupScope=/subscriptions/$SUBSCRIPTION_ID/resourceGroups/$RESOURCE_GROUP
ResourceScope=$RESOURCE_ID
ManagementGroupScope=/providers/Microsoft.Management/managementGroups/$MANAGEMENT_GROUP_ID
EOF
```

# Troubleshoot_Azure_Governance_RBAC_Policy_Locks_And_Cost_Issues_RBAC_Troubleshooting_Skeleton

```powershell
# Purpose: troubleshoot Azure RBAC authorization failures.

$TenantId = "<tenant-id>"
$SubscriptionId = "<subscription-id>"
$TargetScope = "<target-scope>"
$UserPrincipalName = "<user-upn>"
$GroupObjectId = "<group-object-id>"
$ManagedIdentityObjectId = "<managed-identity-object-id>"
$FailedOperation = "<failed-operation>"
$EvidencePath = "C:\Azure-Governance\06-Troubleshoot-Governance"

Connect-AzAccount -Tenant $TenantId
Set-AzContext -SubscriptionId $SubscriptionId

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\rbac-troubleshooting-transcript.txt"

# Resolve user.
$User = Get-AzADUser -UserPrincipalName $UserPrincipalName

$User |
  Select-Object DisplayName,UserPrincipalName,Id,AccountEnabled |
  Tee-Object "$EvidencePath\rbac-user-object.txt"

# Check direct assignments for user.
Get-AzRoleAssignment -ObjectId $User.Id -ErrorAction SilentlyContinue |
  Select-Object DisplayName,SignInName,ObjectType,RoleDefinitionName,Scope,ObjectId |
  Export-Csv "$EvidencePath\rbac-user-direct-assignments.csv" -NoTypeInformation

# Check group membership.
Get-AzADGroupMember -GroupObjectId $GroupObjectId -ErrorAction SilentlyContinue |
  Select-Object DisplayName,UserPrincipalName,Id,ObjectType |
  Export-Csv "$EvidencePath\rbac-group-members.csv" -NoTypeInformation

# Check group role assignments.
Get-AzRoleAssignment -ObjectId $GroupObjectId -ErrorAction SilentlyContinue |
  Select-Object DisplayName,ObjectType,RoleDefinitionName,Scope,ObjectId |
  Export-Csv "$EvidencePath\rbac-group-role-assignments.csv" -NoTypeInformation

# Check exact scope assignments.
Get-AzRoleAssignment -Scope $TargetScope |
  Select-Object DisplayName,SignInName,ObjectType,RoleDefinitionName,Scope,ObjectId |
  Export-Csv "$EvidencePath\rbac-target-scope-assignments.csv" -NoTypeInformation

# Check privileged assignments at subscription scope.
Get-AzRoleAssignment -Scope "/subscriptions/$SubscriptionId" |
  Where-Object {
    $_.RoleDefinitionName -in @(
      "Owner",
      "Contributor",
      "User Access Administrator",
      "Role Based Access Control Administrator"
    )
  } |
  Select-Object DisplayName,SignInName,ObjectType,RoleDefinitionName,Scope,ObjectId |
  Export-Csv "$EvidencePath\rbac-privileged-subscription-assignments.csv" -NoTypeInformation

# Check managed identity assignments if workload failure.
Get-AzRoleAssignment -ObjectId $ManagedIdentityObjectId -ErrorAction SilentlyContinue |
  Select-Object DisplayName,ObjectType,RoleDefinitionName,Scope,ObjectId |
  Export-Csv "$EvidencePath\rbac-managed-identity-assignments.csv" -NoTypeInformation

# Record failed operation.
[PSCustomObject]@{
  UserPrincipalName = $UserPrincipalName
  UserObjectId      = $User.Id
  TargetScope       = $TargetScope
  FailedOperation   = $FailedOperation
  TroubleshootingQuestion = "Does any direct, group, or inherited role assignment grant this operation at this scope?"
} | Export-Csv "$EvidencePath\rbac-failed-operation-record.csv" -NoTypeInformation

Stop-Transcript
```

```bash
# Azure CLI equivalent.

TENANT_ID="<tenant-id>"
SUBSCRIPTION_ID="<subscription-id>"
TARGET_SCOPE="<target-scope>"
USER_UPN="<user-upn>"
GROUP_OBJECT_ID="<group-object-id>"
MANAGED_IDENTITY_OBJECT_ID="<managed-identity-object-id>"
FAILED_OPERATION="<failed-operation>"
EVIDENCE_PATH="$HOME/azure-governance/06-troubleshoot-governance"

mkdir -p "$EVIDENCE_PATH"

az login --tenant "$TENANT_ID"
az account set --subscription "$SUBSCRIPTION_ID"

USER_OBJECT_ID=$(az ad user show --id "$USER_UPN" --query id -o tsv)

echo "$USER_OBJECT_ID" | tee "$EVIDENCE_PATH/rbac-user-object-id.txt"

az ad user show \
  --id "$USER_UPN" \
  -o json | tee "$EVIDENCE_PATH/rbac-user-object.json"

az role assignment list \
  --assignee "$USER_OBJECT_ID" \
  -o json | tee "$EVIDENCE_PATH/rbac-user-direct-assignments.json"

az ad group member list \
  --group "$GROUP_OBJECT_ID" \
  -o json | tee "$EVIDENCE_PATH/rbac-group-members.json"

az role assignment list \
  --assignee "$GROUP_OBJECT_ID" \
  -o json | tee "$EVIDENCE_PATH/rbac-group-role-assignments.json"

az role assignment list \
  --scope "$TARGET_SCOPE" \
  -o json | tee "$EVIDENCE_PATH/rbac-target-scope-assignments.json"

az role assignment list \
  --scope "$TARGET_SCOPE" \
  --include-inherited \
  -o json | tee "$EVIDENCE_PATH/rbac-target-scope-include-inherited.json"

az role assignment list \
  --assignee "$MANAGED_IDENTITY_OBJECT_ID" \
  -o json | tee "$EVIDENCE_PATH/rbac-managed-identity-assignments.json"

cat > "$EVIDENCE_PATH/rbac-failed-operation-record.txt" <<EOF
UserPrincipalName=$USER_UPN
UserObjectId=$USER_OBJECT_ID
TargetScope=$TARGET_SCOPE
FailedOperation=$FAILED_OPERATION
Question=Does direct, group, or inherited RBAC grant this operation at this scope?
EOF
```

# Troubleshoot_Azure_Governance_RBAC_Policy_Locks_And_Cost_Issues_Policy_Troubleshooting_Skeleton

```powershell
# Purpose: troubleshoot Azure Policy deny, audit, modify, deployIfNotExists, and remediation issues.

$TenantId = "<tenant-id>"
$SubscriptionId = "<subscription-id>"
$ManagementGroupId = "<management-group-id>"
$ResourceGroupName = "<resource-group-name>"
$ResourceId = "<resource-id>"
$TargetScope = "<target-scope>"
$PolicyAssignmentName = "<policy-assignment-name>"
$EvidencePath = "C:\Azure-Governance\06-Troubleshoot-Governance"

Connect-AzAccount -Tenant $TenantId
Set-AzContext -SubscriptionId $SubscriptionId

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\policy-troubleshooting-transcript.txt"

# Capture assignments at common scopes.
Get-AzPolicyAssignment -Scope "/providers/Microsoft.Management/managementGroups/$ManagementGroupId" -ErrorAction SilentlyContinue |
  ConvertTo-Json -Depth 30 |
  Out-File "$EvidencePath\policy-assignments-management-group.json"

Get-AzPolicyAssignment -Scope "/subscriptions/$SubscriptionId" -ErrorAction SilentlyContinue |
  ConvertTo-Json -Depth 30 |
  Out-File "$EvidencePath\policy-assignments-subscription.json"

Get-AzPolicyAssignment -Scope "/subscriptions/$SubscriptionId/resourceGroups/$ResourceGroupName" -ErrorAction SilentlyContinue |
  ConvertTo-Json -Depth 30 |
  Out-File "$EvidencePath\policy-assignments-resource-group.json"

Get-AzPolicyAssignment -Scope $TargetScope -ErrorAction SilentlyContinue |
  ConvertTo-Json -Depth 30 |
  Out-File "$EvidencePath\policy-assignments-target-scope.json"

# Capture selected assignment.
$Assignment = Get-AzPolicyAssignment -Name $PolicyAssignmentName -Scope $TargetScope -ErrorAction SilentlyContinue

if ($Assignment) {
  $Assignment |
    ConvertTo-Json -Depth 30 |
    Out-File "$EvidencePath\policy-selected-assignment.json"

  # Capture managed identity on assignment.
  $Assignment.Identity |
    ConvertTo-Json -Depth 10 |
    Out-File "$EvidencePath\policy-selected-assignment-identity.json"

  # Capture identity RBAC if identity exists.
  if ($Assignment.Identity.PrincipalId) {
    Get-AzRoleAssignment -ObjectId $Assignment.Identity.PrincipalId -ErrorAction SilentlyContinue |
      Select-Object DisplayName,ObjectType,RoleDefinitionName,Scope,ObjectId |
      Export-Csv "$EvidencePath\policy-assignment-identity-rbac.csv" -NoTypeInformation
  }
}

# Capture resource policy state.
if (Get-Module -ListAvailable -Name Az.PolicyInsights) {
  Import-Module Az.PolicyInsights

  Get-AzPolicyState -ResourceId $ResourceId -ErrorAction SilentlyContinue |
    Select-Object Timestamp,ResourceId,PolicyAssignmentName,PolicyDefinitionName,ComplianceState,PolicyDefinitionAction |
    Export-Csv "$EvidencePath\policy-state-resource.csv" -NoTypeInformation

  Get-AzPolicyState -SubscriptionId $SubscriptionId |
    Where-Object { $_.ComplianceState -eq "NonCompliant" } |
    Select-Object Timestamp,ResourceId,PolicyAssignmentName,PolicyDefinitionName,ComplianceState,PolicyDefinitionAction |
    Export-Csv "$EvidencePath\policy-state-noncompliant-subscription.csv" -NoTypeInformation

  Get-AzPolicyEvent -SubscriptionId $SubscriptionId -ErrorAction SilentlyContinue |
    Select-Object Timestamp,ResourceId,PolicyAssignmentName,PolicyDefinitionName,ComplianceState,PolicyDefinitionAction |
    Export-Csv "$EvidencePath\policy-events.csv" -NoTypeInformation
}

Stop-Transcript
```

```bash
# Azure CLI equivalent.

TENANT_ID="<tenant-id>"
SUBSCRIPTION_ID="<subscription-id>"
MANAGEMENT_GROUP_ID="<management-group-id>"
RESOURCE_GROUP="<resource-group-name>"
RESOURCE_ID="<resource-id>"
TARGET_SCOPE="<target-scope>"
POLICY_ASSIGNMENT_NAME="<policy-assignment-name>"
EVIDENCE_PATH="$HOME/azure-governance/06-troubleshoot-governance"

mkdir -p "$EVIDENCE_PATH"

az login --tenant "$TENANT_ID"
az account set --subscription "$SUBSCRIPTION_ID"

az policy assignment list \
  --scope "/providers/Microsoft.Management/managementGroups/$MANAGEMENT_GROUP_ID" \
  -o json | tee "$EVIDENCE_PATH/policy-assignments-management-group.json"

az policy assignment list \
  --scope "/subscriptions/$SUBSCRIPTION_ID" \
  -o json | tee "$EVIDENCE_PATH/policy-assignments-subscription.json"

az policy assignment list \
  --scope "/subscriptions/$SUBSCRIPTION_ID/resourceGroups/$RESOURCE_GROUP" \
  -o json | tee "$EVIDENCE_PATH/policy-assignments-resource-group.json"

az policy assignment list \
  --scope "$TARGET_SCOPE" \
  -o json | tee "$EVIDENCE_PATH/policy-assignments-target-scope.json"

az policy assignment show \
  --name "$POLICY_ASSIGNMENT_NAME" \
  --scope "$TARGET_SCOPE" \
  -o json | tee "$EVIDENCE_PATH/policy-selected-assignment.json"

az policy assignment show \
  --name "$POLICY_ASSIGNMENT_NAME" \
  --scope "$TARGET_SCOPE" \
  --query identity \
  -o json | tee "$EVIDENCE_PATH/policy-selected-assignment-identity.json"

az policy state list \
  --resource "$RESOURCE_ID" \
  -o json | tee "$EVIDENCE_PATH/policy-state-resource.json"

az policy state list \
  --subscription "$SUBSCRIPTION_ID" \
  --filter "complianceState eq 'NonCompliant'" \
  -o json | tee "$EVIDENCE_PATH/policy-state-noncompliant-subscription.json"

az policy state summarize \
  --subscription "$SUBSCRIPTION_ID" \
  -o json | tee "$EVIDENCE_PATH/policy-state-summary.json"
```

# Troubleshoot_Azure_Governance_RBAC_Policy_Locks_And_Cost_Issues_Lock_Troubleshooting_Skeleton

```powershell
# Purpose: troubleshoot delete, update, move, and deployment failures caused by management locks.

$TenantId = "<tenant-id>"
$SubscriptionId = "<subscription-id>"
$ResourceGroupName = "<resource-group-name>"
$ResourceId = "<resource-id>"
$TargetScope = "<target-scope>"
$EvidencePath = "C:\Azure-Governance\06-Troubleshoot-Governance"

Connect-AzAccount -Tenant $TenantId
Set-AzContext -SubscriptionId $SubscriptionId

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\lock-troubleshooting-transcript.txt"

# Subscription-level locks.
Get-AzResourceLock -Scope "/subscriptions/$SubscriptionId" -ErrorAction SilentlyContinue |
  Select-Object Name,LockLevel,Scope,Notes,ResourceId |
  Export-Csv "$EvidencePath\locks-subscription-scope.csv" -NoTypeInformation

# Resource group locks.
Get-AzResourceLock -ResourceGroupName $ResourceGroupName -ErrorAction SilentlyContinue |
  Select-Object Name,LockLevel,Scope,Notes,ResourceId |
  Export-Csv "$EvidencePath\locks-resource-group-scope.csv" -NoTypeInformation

# Resource-level locks.
Get-AzResourceLock -Scope $ResourceId -ErrorAction SilentlyContinue |
  Select-Object Name,LockLevel,Scope,Notes,ResourceId |
  Export-Csv "$EvidencePath\locks-resource-scope.csv" -NoTypeInformation

# Target scope locks.
Get-AzResourceLock -Scope $TargetScope -ErrorAction SilentlyContinue |
  Select-Object Name,LockLevel,Scope,Notes,ResourceId |
  Export-Csv "$EvidencePath\locks-target-scope.csv" -NoTypeInformation

# Activity log events that commonly show lock impact.
Get-AzActivityLog -ResourceGroupName $ResourceGroupName -StartTime (Get-Date).AddDays(-7) |
  Where-Object {
    $_.OperationName.Value -like "*locks*" -or
    $_.SubStatus.Value -like "*Locked*" -or
    $_.Status.Value -eq "Failed"
  } |
  Select-Object EventTimestamp,OperationName,ResourceGroupName,Status,SubStatus,Caller,CorrelationId |
  Export-Csv "$EvidencePath\lock-related-activity-log.csv" -NoTypeInformation

Stop-Transcript
```

```bash
# Azure CLI equivalent.

TENANT_ID="<tenant-id>"
SUBSCRIPTION_ID="<subscription-id>"
RESOURCE_GROUP="<resource-group-name>"
RESOURCE_ID="<resource-id>"
TARGET_SCOPE="<target-scope>"
EVIDENCE_PATH="$HOME/azure-governance/06-troubleshoot-governance"

mkdir -p "$EVIDENCE_PATH"

az login --tenant "$TENANT_ID"
az account set --subscription "$SUBSCRIPTION_ID"

az lock list \
  --scope "/subscriptions/$SUBSCRIPTION_ID" \
  -o json | tee "$EVIDENCE_PATH/locks-subscription-scope.json"

az lock list \
  --resource-group "$RESOURCE_GROUP" \
  -o json | tee "$EVIDENCE_PATH/locks-resource-group-scope.json"

az lock list \
  --scope "$RESOURCE_ID" \
  -o json | tee "$EVIDENCE_PATH/locks-resource-scope.json"

az lock list \
  --scope "$TARGET_SCOPE" \
  -o json | tee "$EVIDENCE_PATH/locks-target-scope.json"

az monitor activity-log list \
  --resource-group "$RESOURCE_GROUP" \
  --max-events 100 \
  -o json | tee "$EVIDENCE_PATH/lock-related-activity-log.json"
```

# Troubleshoot_Azure_Governance_RBAC_Policy_Locks_And_Cost_Issues_Move_Operation_Troubleshooting_Skeleton

```powershell
# Purpose: troubleshoot failed resource moves between resource groups or subscriptions.

$TenantId = "<tenant-id>"
$SourceSubscriptionId = "<source-subscription-id>"
$DestinationSubscriptionId = "<destination-subscription-id>"
$SourceResourceGroup = "<source-resource-group>"
$DestinationResourceGroup = "<destination-resource-group>"
$ResourceId = "<resource-id>"
$ProviderNamespace = "<provider-namespace>"
$EvidencePath = "C:\Azure-Governance\06-Troubleshoot-Governance"

Connect-AzAccount -Tenant $TenantId

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\move-operation-troubleshooting-transcript.txt"

# Confirm tenant match.
$SourceTenantId = (Get-AzSubscription -SubscriptionId $SourceSubscriptionId).TenantId
$DestinationTenantId = (Get-AzSubscription -SubscriptionId $DestinationSubscriptionId).TenantId

[PSCustomObject]@{
  SourceSubscriptionId      = $SourceSubscriptionId
  SourceTenantId            = $SourceTenantId
  DestinationSubscriptionId = $DestinationSubscriptionId
  DestinationTenantId       = $DestinationTenantId
  SameTenant                = ($SourceTenantId -eq $DestinationTenantId)
} | Export-Csv "$EvidencePath\move-tenant-match.csv" -NoTypeInformation

# Source checks.
Set-AzContext -SubscriptionId $SourceSubscriptionId

Get-AzResourceGroup -Name $SourceResourceGroup |
  ConvertTo-Json -Depth 10 |
  Out-File "$EvidencePath\move-source-resource-group.json"

Get-AzResource -ResourceId $ResourceId |
  Select-Object Name,ResourceType,Location,ResourceGroupName,ResourceId,Tags |
  ConvertTo-Json -Depth 10 |
  Out-File "$EvidencePath\move-source-resource.json"

Get-AzResourceLock -ResourceGroupName $SourceResourceGroup -ErrorAction SilentlyContinue |
  Export-Csv "$EvidencePath\move-source-locks.csv" -NoTypeInformation

# Destination checks.
Set-AzContext -SubscriptionId $DestinationSubscriptionId

Get-AzResourceGroup -Name $DestinationResourceGroup |
  ConvertTo-Json -Depth 10 |
  Out-File "$EvidencePath\move-destination-resource-group.json"

Get-AzResourceProvider -ProviderNamespace $ProviderNamespace |
  Select-Object ProviderNamespace,RegistrationState |
  Export-Csv "$EvidencePath\move-destination-provider-registration.csv" -NoTypeInformation

Get-AzResourceLock -ResourceGroupName $DestinationResourceGroup -ErrorAction SilentlyContinue |
  Export-Csv "$EvidencePath\move-destination-locks.csv" -NoTypeInformation

# Activity log checks.
Set-AzContext -SubscriptionId $SourceSubscriptionId

Get-AzActivityLog -ResourceGroupName $SourceResourceGroup -StartTime (Get-Date).AddDays(-7) |
  Where-Object {
    $_.OperationName.Value -like "*move*" -or
    $_.Status.Value -eq "Failed"
  } |
  Select-Object EventTimestamp,OperationName,ResourceGroupName,Status,SubStatus,Caller,CorrelationId |
  Export-Csv "$EvidencePath\move-source-activity-log.csv" -NoTypeInformation

Stop-Transcript
```

```bash
# Azure CLI equivalent.

TENANT_ID="<tenant-id>"
SOURCE_SUBSCRIPTION_ID="<source-subscription-id>"
DESTINATION_SUBSCRIPTION_ID="<destination-subscription-id>"
SOURCE_RG="<source-resource-group>"
DESTINATION_RG="<destination-resource-group>"
RESOURCE_ID="<resource-id>"
PROVIDER_NAMESPACE="<provider-namespace>"
EVIDENCE_PATH="$HOME/azure-governance/06-troubleshoot-governance"

mkdir -p "$EVIDENCE_PATH"

az login --tenant "$TENANT_ID"

SOURCE_TENANT_ID=$(az account show --subscription "$SOURCE_SUBSCRIPTION_ID" --query tenantId -o tsv)
DESTINATION_TENANT_ID=$(az account show --subscription "$DESTINATION_SUBSCRIPTION_ID" --query tenantId -o tsv)

cat > "$EVIDENCE_PATH/move-tenant-match.txt" <<EOF
SourceSubscriptionId=$SOURCE_SUBSCRIPTION_ID
SourceTenantId=$SOURCE_TENANT_ID
DestinationSubscriptionId=$DESTINATION_SUBSCRIPTION_ID
DestinationTenantId=$DESTINATION_TENANT_ID
SameTenant=$([ "$SOURCE_TENANT_ID" = "$DESTINATION_TENANT_ID" ] && echo true || echo false)
EOF

# Source checks.
az account set --subscription "$SOURCE_SUBSCRIPTION_ID"

az group show \
  --name "$SOURCE_RG" \
  -o json | tee "$EVIDENCE_PATH/move-source-resource-group.json"

az resource show \
  --ids "$RESOURCE_ID" \
  -o json | tee "$EVIDENCE_PATH/move-source-resource.json"

az lock list \
  --resource-group "$SOURCE_RG" \
  -o json | tee "$EVIDENCE_PATH/move-source-locks.json"

az monitor activity-log list \
  --resource-group "$SOURCE_RG" \
  --max-events 100 \
  -o json | tee "$EVIDENCE_PATH/move-source-activity-log.json"

# Destination checks.
az account set --subscription "$DESTINATION_SUBSCRIPTION_ID"

az group show \
  --name "$DESTINATION_RG" \
  -o json | tee "$EVIDENCE_PATH/move-destination-resource-group.json"

az provider show \
  --namespace "$PROVIDER_NAMESPACE" \
  --query "{Provider:namespace,State:registrationState}" \
  -o json | tee "$EVIDENCE_PATH/move-destination-provider-registration.json"

az lock list \
  --resource-group "$DESTINATION_RG" \
  -o json | tee "$EVIDENCE_PATH/move-destination-locks.json"
```

# Troubleshoot_Azure_Governance_RBAC_Policy_Locks_And_Cost_Issues_Cost_Management_Troubleshooting_Skeleton

```powershell
# Purpose: troubleshoot cost visibility, budget alert, tag, and Advisor recommendation issues.

$TenantId = "<tenant-id>"
$SubscriptionId = "<subscription-id>"
$BudgetScope = "/subscriptions/$SubscriptionId"
$UserPrincipalName = "<user-upn>"
$ResourceGroupName = "<resource-group-name>"
$EvidencePath = "C:\Azure-Governance\06-Troubleshoot-Governance"

Connect-AzAccount -Tenant $TenantId
Set-AzContext -SubscriptionId $SubscriptionId

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\cost-management-troubleshooting-transcript.txt"

# Resolve user.
$User = Get-AzADUser -UserPrincipalName $UserPrincipalName

$User |
  Select-Object DisplayName,UserPrincipalName,Id,AccountEnabled |
  Tee-Object "$EvidencePath\cost-user-object.txt"

# Check Cost Management relevant RBAC.
Get-AzRoleAssignment -ObjectId $User.Id -ErrorAction SilentlyContinue |
  Where-Object {
    $_.RoleDefinitionName -in @(
      "Owner",
      "Contributor",
      "Reader",
      "Cost Management Reader",
      "Cost Management Contributor"
    )
  } |
  Select-Object DisplayName,SignInName,ObjectType,RoleDefinitionName,Scope,ObjectId |
  Export-Csv "$EvidencePath\cost-user-relevant-rbac.csv" -NoTypeInformation

Get-AzRoleAssignment -Scope $BudgetScope -ErrorAction SilentlyContinue |
  Select-Object DisplayName,SignInName,ObjectType,RoleDefinitionName,Scope,ObjectId |
  Export-Csv "$EvidencePath\cost-budget-scope-rbac.csv" -NoTypeInformation

# Export resource groups and tags for chargeback review.
Get-AzResourceGroup |
  Select-Object ResourceGroupName,Location,Tags |
  Export-Csv "$EvidencePath\cost-resource-groups-tags.csv" -NoTypeInformation

Get-AzResource |
  Select-Object Name,ResourceType,ResourceGroupName,Location,ResourceId,Tags |
  Export-Csv "$EvidencePath\cost-resources-tags.csv" -NoTypeInformation

Get-AzResource |
  Where-Object { -not $_.Tags -or -not $_.Tags.ContainsKey("CostCenter") } |
  Select-Object Name,ResourceType,ResourceGroupName,ResourceId,Tags |
  Export-Csv "$EvidencePath\cost-resources-missing-costcenter.csv" -NoTypeInformation

# Export Advisor recommendations if available.
if (Get-Module -ListAvailable -Name Az.Advisor) {
  Import-Module Az.Advisor

  Get-AzAdvisorRecommendation -Category Cost |
    Select-Object Name,Category,Impact,ImpactedField,ImpactedValue,ShortDescription,ResourceId |
    Export-Csv "$EvidencePath\cost-advisor-recommendations.csv" -NoTypeInformation
}
else {
  "Az.Advisor module not installed. Use Azure CLI or portal for Advisor cost recommendations." |
    Out-File "$EvidencePath\cost-advisor-module-status.txt"
}

# Manual review record for portal-only budget checks.
[PSCustomObject]@{
  BudgetScope = $BudgetScope
  BudgetPortalPath = "Azure Portal > Cost Management + Billing > Cost Management > Budgets"
  CostAlertsPortalPath = "Azure Portal > Cost Management + Billing > Cost Management > Cost alerts"
  CostAnalysisPortalPath = "Azure Portal > Cost Management + Billing > Cost Management > Cost analysis"
  TroubleshootingQuestions = "Is scope correct? Are filters correct? Has cost data arrived? Did threshold evaluate? Are recipients correct? Is alert active or dismissed?"
} | Export-Csv "$EvidencePath\cost-management-portal-review-record.csv" -NoTypeInformation

Stop-Transcript
```

```bash
# Azure CLI equivalent.

TENANT_ID="<tenant-id>"
SUBSCRIPTION_ID="<subscription-id>"
BUDGET_SCOPE="/subscriptions/$SUBSCRIPTION_ID"
USER_UPN="<user-upn>"
EVIDENCE_PATH="$HOME/azure-governance/06-troubleshoot-governance"

mkdir -p "$EVIDENCE_PATH"

az login --tenant "$TENANT_ID"
az account set --subscription "$SUBSCRIPTION_ID"

USER_OBJECT_ID=$(az ad user show --id "$USER_UPN" --query id -o tsv)

az ad user show \
  --id "$USER_UPN" \
  -o json | tee "$EVIDENCE_PATH/cost-user-object.json"

az role assignment list \
  --assignee "$USER_OBJECT_ID" \
  -o json | tee "$EVIDENCE_PATH/cost-user-rbac.json"

az role assignment list \
  --scope "$BUDGET_SCOPE" \
  -o json | tee "$EVIDENCE_PATH/cost-budget-scope-rbac.json"

az group list \
  --query "[].{Name:name, Location:location, Tags:tags}" \
  -o json | tee "$EVIDENCE_PATH/cost-resource-groups-tags.json"

az resource list \
  --query "[].{Name:name, Type:type, ResourceGroup:resourceGroup, Location:location, Tags:tags, Id:id}" \
  -o json | tee "$EVIDENCE_PATH/cost-resources-tags.json"

az resource list \
  --query "[?tags.CostCenter == null].{Name:name, Type:type, ResourceGroup:resourceGroup, Tags:tags, Id:id}" \
  -o json | tee "$EVIDENCE_PATH/cost-resources-missing-costcenter.json"

az advisor recommendation list \
  --category Cost \
  -o json | tee "$EVIDENCE_PATH/cost-advisor-recommendations.json"

cat > "$EVIDENCE_PATH/cost-management-portal-review-record.txt" <<EOF
BudgetScope=$BUDGET_SCOPE
BudgetPortalPath=Azure Portal > Cost Management + Billing > Cost Management > Budgets
CostAlertsPortalPath=Azure Portal > Cost Management + Billing > Cost Management > Cost alerts
CostAnalysisPortalPath=Azure Portal > Cost Management + Billing > Cost Management > Cost analysis
Questions=Scope, filters, cost data delay, threshold evaluation, recipients, active versus dismissed alert, Advisor eligibility
EOF
```

# Troubleshoot_Azure_Governance_RBAC_Policy_Locks_And_Cost_Issues_Activity_Log_Correlation_Skeleton

```powershell
# Purpose: correlate governance failure evidence through Activity Log.

$TenantId = "<tenant-id>"
$SubscriptionId = "<subscription-id>"
$ResourceGroupName = "<resource-group-name>"
$ResourceId = "<resource-id>"
$CorrelationId = "<correlation-id>"
$EvidencePath = "C:\Azure-Governance\06-Troubleshoot-Governance"

Connect-AzAccount -Tenant $TenantId
Set-AzContext -SubscriptionId $SubscriptionId

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\activity-log-correlation-transcript.txt"

# Correlation ID search.
Get-AzActivityLog -CorrelationId $CorrelationId -ErrorAction SilentlyContinue |
  Select-Object EventTimestamp,OperationName,ResourceGroupName,ResourceProviderName,ResourceId,Status,SubStatus,Caller,CorrelationId,Properties |
  Export-Csv "$EvidencePath\activity-log-correlation-id.csv" -NoTypeInformation

# Failed operations in target resource group.
Get-AzActivityLog -ResourceGroupName $ResourceGroupName -StartTime (Get-Date).AddDays(-7) |
  Where-Object { $_.Status.Value -eq "Failed" } |
  Select-Object EventTimestamp,OperationName,ResourceGroupName,ResourceProviderName,ResourceId,Status,SubStatus,Caller,CorrelationId,Properties |
  Export-Csv "$EvidencePath\activity-log-failed-resource-group.csv" -NoTypeInformation

# Authorization-related events.
Get-AzActivityLog -StartTime (Get-Date).AddDays(-7) |
  Where-Object {
    $_.OperationName.Value -like "*authorization*" -or
    $_.OperationName.Value -like "*roleAssignments*" -or
    $_.ResourceProviderName.Value -eq "Microsoft.Authorization"
  } |
  Select-Object EventTimestamp,OperationName,ResourceGroupName,ResourceProviderName,ResourceId,Status,SubStatus,Caller,CorrelationId |
  Export-Csv "$EvidencePath\activity-log-authorization-events.csv" -NoTypeInformation

# Policy-related events.
Get-AzActivityLog -StartTime (Get-Date).AddDays(-7) |
  Where-Object {
    $_.OperationName.Value -like "*policy*" -or
    $_.ResourceProviderName.Value -eq "Microsoft.PolicyInsights"
  } |
  Select-Object EventTimestamp,OperationName,ResourceGroupName,ResourceProviderName,ResourceId,Status,SubStatus,Caller,CorrelationId |
  Export-Csv "$EvidencePath\activity-log-policy-events.csv" -NoTypeInformation

# Lock-related events.
Get-AzActivityLog -StartTime (Get-Date).AddDays(-7) |
  Where-Object {
    $_.OperationName.Value -like "*locks*" -or
    $_.SubStatus.Value -like "*Locked*"
  } |
  Select-Object EventTimestamp,OperationName,ResourceGroupName,ResourceProviderName,ResourceId,Status,SubStatus,Caller,CorrelationId |
  Export-Csv "$EvidencePath\activity-log-lock-events.csv" -NoTypeInformation

Stop-Transcript
```

```bash
# Azure CLI equivalent.

TENANT_ID="<tenant-id>"
SUBSCRIPTION_ID="<subscription-id>"
RESOURCE_GROUP="<resource-group-name>"
CORRELATION_ID="<correlation-id>"
EVIDENCE_PATH="$HOME/azure-governance/06-troubleshoot-governance"

mkdir -p "$EVIDENCE_PATH"

az login --tenant "$TENANT_ID"
az account set --subscription "$SUBSCRIPTION_ID"

az monitor activity-log list \
  --correlation-id "$CORRELATION_ID" \
  -o json | tee "$EVIDENCE_PATH/activity-log-correlation-id.json"

az monitor activity-log list \
  --resource-group "$RESOURCE_GROUP" \
  --status Failed \
  --max-events 100 \
  -o json | tee "$EVIDENCE_PATH/activity-log-failed-resource-group.json"

az monitor activity-log list \
  --resource-provider "Microsoft.Authorization" \
  --max-events 100 \
  -o json | tee "$EVIDENCE_PATH/activity-log-authorization-events.json"

az monitor activity-log list \
  --resource-provider "Microsoft.PolicyInsights" \
  --max-events 100 \
  -o json | tee "$EVIDENCE_PATH/activity-log-policy-events.json"

az monitor activity-log list \
  --max-events 100 \
  -o json | tee "$EVIDENCE_PATH/activity-log-recent.json"
```

# Troubleshoot_Azure_Governance_RBAC_Policy_Locks_And_Cost_Issues_Evidence_Export_Skeleton

```powershell
# Purpose: export a full governance troubleshooting evidence package.

$TenantId = "<tenant-id>"
$SubscriptionId = "<subscription-id>"
$ManagementGroupId = "<management-group-id>"
$ResourceGroupName = "<resource-group-name>"
$ResourceId = "<resource-id>"
$TargetScope = "<target-scope>"
$UserPrincipalName = "<user-upn>"
$GroupObjectId = "<group-object-id>"
$PolicyAssignmentName = "<policy-assignment-name>"
$ManagedIdentityObjectId = "<managed-identity-object-id>"
$CorrelationId = "<correlation-id>"
$EvidencePath = "C:\Azure-Governance\06-Troubleshoot-Governance"

Connect-AzAccount -Tenant $TenantId
Set-AzContext -SubscriptionId $SubscriptionId

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\06-governance-troubleshooting-final-evidence-transcript.txt"

# Context.
Get-AzTenant |
  Select-Object Name,Id |
  Export-Csv "$EvidencePath\tenants.csv" -NoTypeInformation

Get-AzSubscription |
  Select-Object Name,Id,TenantId,State |
  Export-Csv "$EvidencePath\subscriptions.csv" -NoTypeInformation

Get-AzContext |
  Format-List |
  Out-File "$EvidencePath\active-context.txt"

# Scope.
Get-AzManagementGroup -GroupName $ManagementGroupId -Expand -Recurse |
  ConvertTo-Json -Depth 30 |
  Out-File "$EvidencePath\management-group-tree.json"

Get-AzResourceGroup -Name $ResourceGroupName |
  ConvertTo-Json -Depth 10 |
  Out-File "$EvidencePath\resource-group.json"

Get-AzResource -ResourceId $ResourceId |
  Select-Object Name,ResourceType,Location,ResourceGroupName,ResourceId,Tags |
  ConvertTo-Json -Depth 10 |
  Out-File "$EvidencePath\resource.json"

# RBAC.
$User = Get-AzADUser -UserPrincipalName $UserPrincipalName -ErrorAction SilentlyContinue

if ($User) {
  $User |
    Select-Object DisplayName,UserPrincipalName,Id,AccountEnabled |
    Export-Csv "$EvidencePath\user-object.csv" -NoTypeInformation

  Get-AzRoleAssignment -ObjectId $User.Id -ErrorAction SilentlyContinue |
    Select-Object DisplayName,SignInName,ObjectType,RoleDefinitionName,Scope,ObjectId |
    Export-Csv "$EvidencePath\user-role-assignments.csv" -NoTypeInformation
}

Get-AzADGroupMember -GroupObjectId $GroupObjectId -ErrorAction SilentlyContinue |
  Select-Object DisplayName,UserPrincipalName,Id,ObjectType |
  Export-Csv "$EvidencePath\group-members.csv" -NoTypeInformation

Get-AzRoleAssignment -Scope $TargetScope -ErrorAction SilentlyContinue |
  Select-Object DisplayName,SignInName,ObjectType,RoleDefinitionName,Scope,ObjectId |
  Export-Csv "$EvidencePath\target-scope-role-assignments.csv" -NoTypeInformation

Get-AzRoleAssignment -ObjectId $ManagedIdentityObjectId -ErrorAction SilentlyContinue |
  Select-Object DisplayName,ObjectType,RoleDefinitionName,Scope,ObjectId |
  Export-Csv "$EvidencePath\managed-identity-role-assignments.csv" -NoTypeInformation

# Policy.
Get-AzPolicyAssignment -Scope $TargetScope -ErrorAction SilentlyContinue |
  ConvertTo-Json -Depth 30 |
  Out-File "$EvidencePath\target-scope-policy-assignments.json"

Get-AzPolicyAssignment -Name $PolicyAssignmentName -Scope $TargetScope -ErrorAction SilentlyContinue |
  ConvertTo-Json -Depth 30 |
  Out-File "$EvidencePath\selected-policy-assignment.json"

if (Get-Module -ListAvailable -Name Az.PolicyInsights) {
  Import-Module Az.PolicyInsights

  Get-AzPolicyState -ResourceId $ResourceId -ErrorAction SilentlyContinue |
    Select-Object Timestamp,ResourceId,PolicyAssignmentName,PolicyDefinitionName,ComplianceState,PolicyDefinitionAction |
    Export-Csv "$EvidencePath\resource-policy-state.csv" -NoTypeInformation

  Get-AzPolicyState -SubscriptionId $SubscriptionId |
    Where-Object { $_.ComplianceState -eq "NonCompliant" } |
    Select-Object Timestamp,ResourceId,PolicyAssignmentName,PolicyDefinitionName,ComplianceState,PolicyDefinitionAction |
    Export-Csv "$EvidencePath\noncompliant-policy-state.csv" -NoTypeInformation
}

# Locks.
Get-AzResourceLock -Scope "/subscriptions/$SubscriptionId" -ErrorAction SilentlyContinue |
  Export-Csv "$EvidencePath\subscription-locks.csv" -NoTypeInformation

Get-AzResourceLock -ResourceGroupName $ResourceGroupName -ErrorAction SilentlyContinue |
  Export-Csv "$EvidencePath\resource-group-locks.csv" -NoTypeInformation

Get-AzResourceLock -Scope $ResourceId -ErrorAction SilentlyContinue |
  Export-Csv "$EvidencePath\resource-locks.csv" -NoTypeInformation

# Cost and tags.
Get-AzResourceGroup |
  Select-Object ResourceGroupName,Location,Tags |
  Export-Csv "$EvidencePath\cost-resource-group-tags.csv" -NoTypeInformation

Get-AzResource |
  Select-Object Name,ResourceType,ResourceGroupName,Location,ResourceId,Tags |
  Export-Csv "$EvidencePath\cost-resource-tags.csv" -NoTypeInformation

Get-AzResource |
  Where-Object { -not $_.Tags -or -not $_.Tags.ContainsKey("CostCenter") } |
  Select-Object Name,ResourceType,ResourceGroupName,ResourceId,Tags |
  Export-Csv "$EvidencePath\cost-resources-missing-costcenter.csv" -NoTypeInformation

if (Get-Module -ListAvailable -Name Az.Advisor) {
  Import-Module Az.Advisor

  Get-AzAdvisorRecommendation -Category Cost |
    Select-Object Name,Category,Impact,ImpactedField,ImpactedValue,ShortDescription,ResourceId |
    Export-Csv "$EvidencePath\advisor-cost-recommendations.csv" -NoTypeInformation
}

# Activity logs.
Get-AzActivityLog -CorrelationId $CorrelationId -ErrorAction SilentlyContinue |
  Select-Object EventTimestamp,OperationName,ResourceGroupName,ResourceProviderName,ResourceId,Status,SubStatus,Caller,CorrelationId,Properties |
  Export-Csv "$EvidencePath\activity-log-correlation-id.csv" -NoTypeInformation

Get-AzActivityLog -ResourceGroupName $ResourceGroupName -StartTime (Get-Date).AddDays(-7) |
  Select-Object EventTimestamp,OperationName,ResourceGroupName,ResourceProviderName,ResourceId,Status,SubStatus,Caller,CorrelationId |
  Export-Csv "$EvidencePath\activity-log-resource-group-last-7-days.csv" -NoTypeInformation

# Summary.
[PSCustomObject]@{
  IncidentNumber = "<incident-number>"
  TenantId       = $TenantId
  SubscriptionId = $SubscriptionId
  ManagementGroupId = $ManagementGroupId
  ResourceGroupName = $ResourceGroupName
  ResourceId = $ResourceId
  TargetScope = $TargetScope
  UserPrincipalName = $UserPrincipalName
  PolicyAssignmentName = $PolicyAssignmentName
  CorrelationId = $CorrelationId
  EvidencePath = $EvidencePath
  ExportDate = (Get-Date)
} | Export-Csv "$EvidencePath\troubleshooting-summary.csv" -NoTypeInformation

Stop-Transcript
```

# Troubleshoot_Azure_Governance_RBAC_Policy_Locks_And_Cost_Issues_Verification_Commands

```powershell
# Context.
Get-AzContext
Get-AzTenant
Get-AzSubscription | Select-Object Name,Id,TenantId,State

# Management group hierarchy.
Get-AzManagementGroup -GroupName "<management-group-id>" -Expand -Recurse

# Resource scope.
Get-AzResourceGroup -Name "<resource-group-name>"
Get-AzResource -ResourceId "<resource-id>"

# Activity Log.
Get-AzActivityLog -CorrelationId "<correlation-id>"
Get-AzActivityLog -ResourceGroupName "<resource-group-name>" -MaxRecord 100

# RBAC.
Get-AzRoleAssignment -Scope "<target-scope>"
Get-AzRoleAssignment -ObjectId "<user-object-id>"
Get-AzRoleAssignment -ObjectId "<group-object-id>"
Get-AzRoleAssignment -ObjectId "<managed-identity-object-id>"

# Group membership.
Get-AzADGroupMember -GroupObjectId "<group-object-id>"

# Privileged assignments.
Get-AzRoleAssignment -Scope "/subscriptions/<subscription-id>" |
  Where-Object {
    $_.RoleDefinitionName -in @(
      "Owner",
      "Contributor",
      "User Access Administrator",
      "Role Based Access Control Administrator"
    )
  }

# Policy.
Get-AzPolicyAssignment -Scope "<target-scope>"
Get-AzPolicyAssignment -Name "<policy-assignment-name>" -Scope "<target-scope>"

# Policy state.
Import-Module Az.PolicyInsights
Get-AzPolicyState -ResourceId "<resource-id>"
Get-AzPolicyState -SubscriptionId "<subscription-id>" |
  Where-Object { $_.ComplianceState -eq "NonCompliant" }

# Locks.
Get-AzResourceLock -Scope "/subscriptions/<subscription-id>"
Get-AzResourceLock -ResourceGroupName "<resource-group-name>"
Get-AzResourceLock -Scope "<resource-id>"

# Cost tags.
Get-AzResource |
  Where-Object { -not $_.Tags -or -not $_.Tags.ContainsKey("CostCenter") }

# Advisor.
Import-Module Az.Advisor
Get-AzAdvisorRecommendation -Category Cost
```

```bash
# Azure CLI equivalents.

az account show -o table

az account list --all \
  --query "[].{Name:name, SubscriptionId:id, TenantId:tenantId, State:state}" \
  -o table

az account management-group show \
  --name "<management-group-id>" \
  --expand \
  --recurse \
  -o json

az group show --name "<resource-group-name>" -o json

az resource show --ids "<resource-id>" -o json

az monitor activity-log list \
  --correlation-id "<correlation-id>" \
  -o json

az monitor activity-log list \
  --resource-group "<resource-group-name>" \
  --status Failed \
  --max-events 100 \
  -o table

az role assignment list \
  --scope "<target-scope>" \
  -o table

az role assignment list \
  --scope "<target-scope>" \
  --include-inherited \
  -o table

az role assignment list \
  --assignee "<user-object-id>" \
  -o table

az role assignment list \
  --assignee "<group-object-id>" \
  -o table

az role assignment list \
  --assignee "<managed-identity-object-id>" \
  -o table

az ad group member list \
  --group "<group-object-id>" \
  -o table

az policy assignment list \
  --scope "<target-scope>" \
  -o table

az policy assignment show \
  --name "<policy-assignment-name>" \
  --scope "<target-scope>" \
  -o json

az policy state list \
  --resource "<resource-id>" \
  -o table

az policy state list \
  --subscription "<subscription-id>" \
  --filter "complianceState eq 'NonCompliant'" \
  -o table

az lock list \
  --scope "/subscriptions/<subscription-id>" \
  -o table

az lock list \
  --resource-group "<resource-group-name>" \
  -o table

az lock list \
  --scope "<resource-id>" \
  -o table

az resource list \
  --query "[?tags.CostCenter == null].{Name:name, Type:type, ResourceGroup:resourceGroup, Id:id}" \
  -o table

az advisor recommendation list \
  --category Cost \
  -o table
```

# Troubleshoot_Azure_Governance_RBAC_Policy_Locks_And_Cost_Issues_Rollback

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Capture current broken state | Admin Workstation | Run evidence export skeleton | Evidence is preserved before changes |
| 2 | Confirm root cause category | Operator | RBAC, Policy, Lock, Move, Cost, or Advisor | Fix path is selected |
| 3 | Roll back incorrect RBAC assignment | Cloud Shell | `az role assignment delete --assignee "<principal-id>" --role "<role>" --scope "<scope>"` | Incorrect access is removed |
| 4 | Restore required RBAC assignment | Cloud Shell | `az role assignment create --assignee-object-id "<principal-id>" --role "<role>" --scope "<scope>"` | Required access is restored |
| 5 | Disable or delete incorrect policy assignment | Cloud Shell | `az policy assignment delete --name "<assignment-name>" --scope "<scope>"` | Blocking policy assignment is removed |
| 6 | Add temporary policy exclusion if approved | Portal / Cloud Shell | Update assignment `notScopes` | Critical scope is excluded intentionally |
| 7 | Remove blocking lock if approved | Cloud Shell | `az lock delete --name "<lock-name>" --resource-group "<resource-group-name>"` | Operation blocker is removed |
| 8 | Restore lock after operation | Cloud Shell | `az lock create --name "<lock-name>" --lock-type CanNotDelete --resource-group "<resource-group-name>"` | Protection is restored |
| 9 | Move resource back if move caused breakage | Cloud Shell | `az resource move --destination-group "<source-resource-group>" --ids "<new-resource-id>"` | Resource returns to previous RG |
| 10 | Correct budget or alert config | Portal | Cost Management > Budgets | Threshold, recipient, filter, or scope is fixed |
| 11 | Revert unsafe Advisor action | Operator | Use resource-specific rollback plan | Workload returns to accepted state |
| 12 | Validate final state | Cloud Shell / Portal | Repeat failing operation and verification commands | Issue is resolved |
| 13 | Export final evidence | Admin Workstation | Run evidence export skeleton again | Before and after evidence exists |
| 14 | Document incident | Operator | Record symptom, root cause, fix, validation, and prevention | Troubleshooting record is complete |

# Troubleshoot_Azure_Governance_RBAC_Policy_Locks_And_Cost_Issues_Failure_Checks

| Symptom | Likely Cause | Device | Command | Fix |
|---|---|---|---|---|
| User cannot open subscription | Wrong tenant or no subscription role | Portal / Cloud Shell | `az account list --all -o table` | Switch directory or grant Reader at subscription scope |
| User can see subscription but not resource group | Missing RBAC at RG scope or inherited deny-like control from policy | Cloud Shell | `az role assignment list --assignee "<user-object-id>" --include-inherited -o table` | Assign proper role at RG or parent scope |
| Add role assignment is disabled | User lacks role assignment write permission | Portal / Cloud Shell | Check IAM at target scope | Use Owner, User Access Administrator, or RBAC Administrator |
| User has role but access still fails | Token propagation delay or stale session | User Workstation | Sign out, sign in, clear CLI account | Refresh token and wait briefly |
| User has too much access | Inherited role assignment from management group or subscription | Cloud Shell | `az role assignment list --scope "<target-scope>" --include-inherited -o table` | Remove or narrow parent assignment |
| Managed identity cannot access resource | Role assigned to wrong object ID or wrong scope | Cloud Shell | `az role assignment list --assignee "<principal-id>" -o table` | Assign role to principal ID at correct scope |
| Direct user assignment found | Access was assigned directly instead of through group | Cloud Shell | `az role assignment list --scope "<scope>" --query "[?principalType=='User']"` | Replace with group-based access |
| Deployment denied by policy | Deny policy assignment applies to scope | Portal / Cloud Shell | `az policy state list --resource "<resource-id>" -o table` | Update resource, change policy, add approved exclusion, or stage audit first |
| Resource marked non-compliant | Policy rule evaluates resource as invalid | Cloud Shell | `az policy state list --resource "<resource-id>"` | Remediate resource or adjust assignment parameters |
| Remediation task fails | Policy assignment identity lacks required role | Cloud Shell | `az role assignment list --assignee "<principal-id>" -o table` | Grant minimum required role at remediation scope |
| Policy assignment identity missing | Assignment was created without managed identity | Cloud Shell | `az policy assignment show --name "<assignment-name>" --scope "<scope>" --query identity` | Add identity or recreate assignment |
| Policy exclusion not working | Wrong excluded scope path | Cloud Shell | `az policy assignment show --name "<assignment-name>" --scope "<scope>" --query notScopes` | Correct exclusion to exact child scope |
| Policy state is stale | Compliance scan has not updated | Cloud Shell | `az policy state trigger-scan --subscription "<subscription-id>"` | Trigger scan and wait for evaluation |
| Delete operation blocked | CanNotDelete lock exists | Cloud Shell | `az lock list --resource-group "<resource-group-name>" -o table` | Remove lock through approved change, then restore |
| Update operation blocked | ReadOnly lock exists | Cloud Shell | `az lock list --scope "<scope>" -o table` | Remove or narrow read-only lock |
| Move operation fails | Resource type unsupported for move | Portal / Cloud Shell | Check provider move support and Activity Log | Rebuild or redeploy instead of moving |
| Cross-subscription move fails | Source and destination subscriptions are in different tenants | Cloud Shell | `az account show --subscription "<id>" --query tenantId -o tsv` | Use subscriptions in same tenant |
| Move fails because provider is not registered | Destination subscription lacks provider registration | Cloud Shell | `az provider show --namespace "<provider-namespace>"` | Register provider in destination subscription |
| Move fails because source or destination is locked | Resource group or resource has lock | Cloud Shell | `az lock list --resource-group "<rg>" -o table` | Remove lock temporarily and restore after move |
| Resource works before move but not after | Resource ID changed and dependency references old ID | Cloud Shell | Compare old and new resource IDs | Update scripts, alerts, RBAC, diagnostics, and dependencies |
| Cost Management blade shows no data | Wrong scope, no permission, or new subscription delay | Portal | Cost Management + Billing > Scope | Use correct scope, grant Cost Management Reader, or wait for data availability |
| Budget cannot be created | Missing Cost Management Contributor or unsupported scope | Portal | Check IAM at budget scope | Grant correct role or choose supported scope |
| Budget alert did not fire | Threshold not reached, wrong filter, wrong recipient, delayed evaluation, or alert dismissed | Portal | Cost Management > Budgets and Cost alerts | Correct threshold, filters, recipient list, or alert state |
| Budget alert expected to stop resources | Budgets only notify and do not stop consumption | Operator | Review budget behavior | Use policy, automation, or manual process for enforcement |
| Forecast alert missing | Forecasted cost did not cross configured threshold | Portal | Budget alert configuration | Adjust forecast threshold or review trend |
| Cost alert email missing | Mail filtering or recipient issue | Portal | Budget recipient list | Verify recipient and allow budget notification sender |
| Advisor cost recommendations empty | Not enough data, no eligible resources, or no access | Cloud Shell | `az advisor recommendation list --category Cost` | Wait for usage history, grant access, or validate eligible resources |
| Advisor recommendation unsafe to apply | Recommendation does not know business dependency | Portal | Advisor recommendation details | Validate owner, change window, metrics, and rollback before action |
| Resources missing cost ownership | Tags are missing or inconsistent | Cloud Shell | `az resource list --query "[?tags.CostCenter == null]"` | Apply tags or enforce with policy remediation |
| Activity Log does not show expected event | Wrong subscription, time window, or resource group filter | Cloud Shell | Remove filters and search broader time range | Re-run query with correct scope and timestamp |
| CLI results differ from portal | CLI context points to wrong subscription or tenant | Cloud Shell | `az account show -o table` | Set correct subscription and tenant |
| PowerShell results differ from portal | PowerShell context points to wrong subscription or stale session | PowerShell | `Get-AzContext` | Run `Set-AzContext -SubscriptionId "<subscription-id>"` |

# Troubleshoot_Azure_Governance_RBAC_Policy_Locks_And_Cost_Issues_Related_Labs

| Lab                                                                        | Relationship                                                                      |
| -------------------------------------------------------------------------- | --------------------------------------------------------------------------------- |
| 01_Manage_Subscriptions_Management_Groups_And_Tenant_Association.md        | Provides management group, subscription, tenant, and inherited governance context |
| 02_Manage_Resource_Groups_Tags_Locks_And_Resource_Move_Operations.md       | Provides resource group, tag, lock, and move operation baseline                   |
| 03_Configure_Azure_RBAC_Role_Assignments_And_Access_Reviews.md             | Provides RBAC assignment and access review baseline                               |
| 04_Configure_Azure_Policy_Initiatives_Assignments_And_Remediation.md       | Provides policy assignment, compliance, identity, and remediation baseline        |
| 05_Configure_Cost_Management_Budgets_Alerts_And_Advisor_Recommendations.md | Provides budget, alert, Advisor, and cost ownership baseline                      |