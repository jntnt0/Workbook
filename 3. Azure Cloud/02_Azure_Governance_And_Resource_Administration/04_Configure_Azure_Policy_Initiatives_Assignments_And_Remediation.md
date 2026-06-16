# Configure_Azure_Policy_Initiatives_Assignments_And_Remediation

# Configure_Azure_Policy_Initiatives_Assignments_And_Remediation_Index
04_Configure_Azure_Policy_Initiatives_Assignments_And_Remediation.md
Configure_Azure_Policy_Initiatives_Assignments_And_Remediation
Configure_Azure_Policy_Initiatives_Assignments_And_Remediation_Source_Basis
Configure_Azure_Policy_Initiatives_Assignments_And_Remediation_Mental_Model
Configure_Azure_Policy_Initiatives_Assignments_And_Remediation_Planning_Table
Configure_Azure_Policy_Initiatives_Assignments_And_Remediation_Configuration_Checklist
Configure_Azure_Policy_Initiatives_Assignments_And_Remediation_Portal_Skeleton
Configure_Azure_Policy_Initiatives_Assignments_And_Remediation_Azure_CLI_Precheck_Skeleton
Configure_Azure_Policy_Initiatives_Assignments_And_Remediation_PowerShell_Precheck_Skeleton
Configure_Azure_Policy_Initiatives_Assignments_And_Remediation_Policy_Definition_Discovery_Skeleton
Configure_Azure_Policy_Initiatives_Assignments_And_Remediation_Initiative_Assignment_Skeleton
Configure_Azure_Policy_Initiatives_Assignments_And_Remediation_Policy_Assignment_Skeleton
Configure_Azure_Policy_Initiatives_Assignments_And_Remediation_Exclusion_And_Enforcement_Mode_Skeleton
Configure_Azure_Policy_Initiatives_Assignments_And_Remediation_Remediation_Identity_Skeleton
Configure_Azure_Policy_Initiatives_Assignments_And_Remediation_Remediation_Task_Skeleton
Configure_Azure_Policy_Initiatives_Assignments_And_Remediation_Compliance_Validation_Skeleton
Configure_Azure_Policy_Initiatives_Assignments_And_Remediation_Evidence_Export_Skeleton
Configure_Azure_Policy_Initiatives_Assignments_And_Remediation_Verification_Commands
Configure_Azure_Policy_Initiatives_Assignments_And_Remediation_Rollback
Configure_Azure_Policy_Initiatives_Assignments_And_Remediation_Failure_Checks
Configure_Azure_Policy_Initiatives_Assignments_And_Remediation_Related_Labs

# Configure_Azure_Policy_Initiatives_Assignments_And_Remediation_Source_Basis

| Source | Relevant Section | What It Supports |
|---|---|---|
| Microsoft Learn | Azure Policy overview | Policy definitions, initiatives, assignments, effects, compliance, and remediation |
| Microsoft Learn | Azure Policy assignment structure | Assignment scope, exclusions, parameters, enforcement mode, metadata, and non-compliance messages |
| Microsoft Learn | Azure Policy definition structure | Policy rule, parameters, aliases, effects, and role definition IDs |
| Microsoft Learn | Azure Policy initiative structure | Grouping related policy definitions into a policy set |
| Microsoft Learn | Azure Policy effects | Audit, deny, disabled, append, modify, deployIfNotExists, auditIfNotExists, and denyAction behavior |
| Microsoft Learn | Assign policy using Azure portal | Portal workflow for assigning policy definitions and initiatives |
| Microsoft Learn | Remediate non-compliant resources | Managed identity requirements, role assignments, and remediation task workflow |
| Microsoft Learn | Get compliance data | Compliance state, policy state queries, and scan behavior |
| Microsoft Learn | Azure RBAC and Azure Policy | Separation between access control and resource compliance control |
| Azure CLI | `az policy definition`, `az policy set-definition`, `az policy assignment`, `az policy state`, `az policy remediation` | Repeatable policy and remediation operations |
| Azure PowerShell | `Get-AzPolicyDefinition`, `New-AzPolicyAssignment`, `Get-AzPolicyState`, `Start-AzPolicyRemediation` | PowerShell policy assignment, compliance, and remediation evidence |

# Configure_Azure_Policy_Initiatives_Assignments_And_Remediation_Mental_Model

| Concept | Operational Meaning |
|---|---|
| Azure Policy | Governance engine that evaluates Azure resource state against business rules |
| Policy definition | JSON rule that checks a resource condition and applies an effect |
| Initiative | Policy set that groups multiple related policy definitions |
| Assignment | Application of a policy definition or initiative to a scope |
| Scope | Management group, subscription, resource group, or resource where policy applies |
| Exclusion | Child scope intentionally excluded from an assignment |
| Parameter | Reusable input value used by a policy or initiative assignment |
| Effect | Action Azure Policy takes when a resource matches the rule |
| Audit | Marks non-compliant resources without blocking deployment |
| Deny | Blocks non-compliant create or update requests |
| Modify | Alters resource properties during create or update when supported |
| DeployIfNotExists | Deploys related resources when the target resource lacks a required configuration |
| Remediation | Task that brings existing non-compliant resources into compliance for modify or deployIfNotExists policies |
| Managed identity | Identity attached to a policy assignment and used to remediate resources |
| Compliance state | Result of policy evaluation against resources |
| Non-compliance message | Human-readable explanation shown for denied or non-compliant resources |
| Enforcement mode | Whether policy effect is enforced or assignment is staged for observation |
| First rule | Start with audit before deny, modify, or deployIfNotExists in unfamiliar environments |
| Blunt rule | Policy controls resource state; RBAC controls who can perform actions |

# Configure_Azure_Policy_Initiatives_Assignments_And_Remediation_Planning_Table

| Item | Example | Decision |
|---|---|---|
| Tenant ID | `aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee` | `<tenant-id>` |
| Subscription ID | `11111111-2222-3333-4444-555555555555` | `<subscription-id>` |
| Management group ID | `corp-platform` | `<management-group-id>` |
| Resource group | `rg-policy-lab-001` | `<resource-group-name>` |
| Assignment scope type | `Subscription` | `<scope-type>` |
| Assignment scope | `/subscriptions/<subscription-id>` | `<assignment-scope>` |
| Excluded scope | `/subscriptions/<subscription-id>/resourceGroups/rg-excluded-001` | `<excluded-scope>` |
| Policy definition name | `Allowed locations` | `<policy-definition-name>` |
| Policy definition ID | `/providers/Microsoft.Authorization/policyDefinitions/<definition-guid>` | `<policy-definition-id>` |
| Initiative display name | `Baseline Governance Initiative` | `<initiative-display-name>` |
| Initiative name | `baseline-governance-initiative` | `<initiative-name>` |
| Initiative definition ID | `/subscriptions/<id>/providers/Microsoft.Authorization/policySetDefinitions/<name>` | `<initiative-definition-id>` |
| Assignment name | `assign-baseline-governance` | `<assignment-name>` |
| Assignment display name | `Baseline Governance Assignment` | `<assignment-display-name>` |
| Assignment location | `eastus` | `<assignment-location>` |
| Enforcement mode | `Default` or `DoNotEnforce` | `<enforcement-mode>` |
| Allowed locations | `eastus`, `eastus2` | `<allowed-locations>` |
| Required tag name | `CostCenter` | `<required-tag-name>` |
| Required tag value | `IT-1001` | `<required-tag-value>` |
| Managed identity type | `SystemAssigned` | `<managed-identity-type>` |
| Remediation role | `Contributor` or narrower built-in role | `<remediation-role-name>` |
| Remediation task name | `remediate-baseline-tags` | `<remediation-task-name>` |
| Evidence path | `C:\Azure-Governance\04-Policy-Initiatives-Assignments-Remediation` | `<evidence-path>` |
| Rollback action | Delete assignment and remediation task | `<rollback-action>` |

# Configure_Azure_Policy_Initiatives_Assignments_And_Remediation_Configuration_Checklist

| Step | Task | Device | Portal / Command | PowerShell | Expected Result |
|---:|---|---|---|---|---|
| 1 | Open Azure admin session | Admin Workstation / Cloud Shell | `https://portal.azure.com` | `Connect-AzAccount -Tenant "<tenant-id>"` | Admin is signed in to correct tenant |
| 2 | Confirm active tenant | Cloud Shell | `az account tenant list -o table` | `Get-AzTenant` | Tenant is confirmed |
| 3 | Confirm active subscription | Cloud Shell | `az account show -o table` | `Get-AzContext` | Correct subscription is selected |
| 4 | Confirm policy permissions | Cloud Shell | `az role assignment list --assignee "<admin-object-id>" --include-inherited -o table` | `Get-AzRoleAssignment -ObjectId "<admin-object-id>"` | Admin has policy assignment rights |
| 5 | Confirm assignment scope exists | Cloud Shell | `az group show --name "<resource-group-name>"` or management group/subscription query | `Get-AzResourceGroup` or `Get-AzManagementGroup` | Target scope exists |
| 6 | List built-in policy definitions | Cloud Shell | `az policy definition list --query "[].{Name:displayName,Id:id}" -o table` | `Get-AzPolicyDefinition -Builtin` | Built-in policy inventory is visible |
| 7 | Select policy definition | Cloud Shell | `az policy definition list --query "[?displayName=='<policy-definition-name>']"` | `Get-AzPolicyDefinition | Where-Object DisplayName -eq "<policy-definition-name>"` | Correct definition ID is known |
| 8 | Inspect policy parameters and effect | Cloud Shell | `az policy definition show --id "<policy-definition-id>" -o json` | `Get-AzPolicyDefinition -Id "<policy-definition-id>"` | Policy logic and parameters are understood |
| 9 | Create initiative when grouping policies | Cloud Shell | `az policy set-definition create ...` | `New-AzPolicySetDefinition ...` | Initiative exists |
| 10 | Assign audit policy first | Cloud Shell | `az policy assignment create --name "<assignment-name>" --scope "<assignment-scope>" --policy "<policy-definition-id>" --enforcement-mode DoNotEnforce` | `New-AzPolicyAssignment ... -EnforcementMode DoNotEnforce` | Policy is staged without enforcement |
| 11 | Assign initiative when ready | Cloud Shell | `az policy assignment create --name "<assignment-name>" --scope "<assignment-scope>" --policy-set-definition "<initiative-definition-id>"` | `New-AzPolicyAssignment -PolicySetDefinition ...` | Initiative is assigned to scope |
| 12 | Configure parameters | Cloud Shell | `--params policy-parameters.json` | `-PolicyParameterObject @{}` | Assignment uses intended values |
| 13 | Configure exclusions if needed | Portal / Cloud Shell | Assignment > Basics > Exclusions | Use `-NotScope` | Excluded child scopes are documented |
| 14 | Configure non-compliance message | Portal | Assignment > Non-compliance messages | N/A | Operators see clear compliance message |
| 15 | Use managed identity for modify or deployIfNotExists | Portal / Cloud Shell | Assignment > Remediation > Create managed identity | `New-AzPolicyAssignment -IdentityType SystemAssigned -Location "<location>"` | Assignment identity exists |
| 16 | Grant managed identity remediation rights if needed | Cloud Shell | `az role assignment create --assignee-object-id "<principal-id>" --role "<role>" --scope "<scope>"` | `New-AzRoleAssignment -ObjectId "<principal-id>" -RoleDefinitionName "<role>" -Scope "<scope>"` | Managed identity has minimum required role |
| 17 | Trigger compliance scan | Cloud Shell | `az policy state trigger-scan --subscription "<subscription-id>"` | `Start-AzPolicyComplianceScan -SubscriptionId "<subscription-id>"` | Policy evaluation is started |
| 18 | Review compliance state | Cloud Shell | `az policy state list --subscription "<subscription-id>" -o table` | `Get-AzPolicyState -SubscriptionId "<subscription-id>"` | Compliance state is visible |
| 19 | Create remediation task when supported | Cloud Shell | `az policy remediation create --name "<remediation-task-name>" --policy-assignment "<assignment-name>" --scope "<assignment-scope>"` | `Start-AzPolicyRemediation -Name "<remediation-task-name>" -PolicyAssignmentId "<assignment-id>"` | Existing non-compliant resources are remediated |
| 20 | Validate remediation status | Cloud Shell | `az policy remediation list --scope "<assignment-scope>" -o table` | `Get-AzPolicyRemediation` | Remediation progress is visible |
| 21 | Recheck compliance | Cloud Shell | `az policy state list --filter "complianceState eq 'NonCompliant'"` | `Get-AzPolicyState | Where-Object ComplianceState -eq "NonCompliant"` | Remaining non-compliance is identified |
| 22 | Move from audit to enforcement only after validation | Portal / Cloud Shell | Update assignment enforcement mode or effect | Update assignment | Policy enforcement is intentional |
| 23 | Export evidence | Admin Workstation | Run evidence export skeleton | Run evidence export skeleton | Evidence files are saved |
| 24 | Document final state | Operator | Record definitions, initiative, assignment, exclusions, identity, remediation, and compliance results | Same | Workbook record is complete |

# Configure_Azure_Policy_Initiatives_Assignments_And_Remediation_Portal_Skeleton

| Step | Portal Area | Action | Expected Result |
|---:|---|---|---|
| 1 | Azure Portal | Sign in as `<admin-upn>` | Correct tenant session opens |
| 2 | Directories + subscriptions | Confirm selected directory | Portal is scoped to correct tenant |
| 3 | Policy | Search for and open **Policy** | Azure Policy blade opens |
| 4 | Policy > Definitions | Search for target built-in definition | Policy definition is identified |
| 5 | Policy > Definitions | Review definition details, parameters, and effect | Operator understands rule behavior |
| 6 | Policy > Authoring > Definitions | Create custom definition only if needed | Custom policy exists if required |
| 7 | Policy > Authoring > Initiative definitions | Create initiative if grouping policies | Initiative exists |
| 8 | Policy > Assignments | Select **Assign policy** or **Assign initiative** | Assignment wizard opens |
| 9 | Basics tab | Select assignment scope | Correct management group, subscription, or resource group is selected |
| 10 | Basics tab | Add exclusions if needed | Child scopes are excluded intentionally |
| 11 | Basics tab | Select policy definition or initiative | Correct governance rule is selected |
| 12 | Basics tab | Set assignment name and description | Assignment is identifiable |
| 13 | Basics tab | Set enforcement mode | Policy is staged or enforced intentionally |
| 14 | Parameters tab | Configure required parameters | Assignment has expected values |
| 15 | Remediation tab | Create managed identity if modify or deployIfNotExists is used | Assignment identity is created |
| 16 | Remediation tab | Select system-assigned or user-assigned identity | Identity type is intentional |
| 17 | Non-compliance messages tab | Add message explaining failed rule | Operators can understand non-compliance |
| 18 | Review + create | Review assignment settings | Scope, exclusions, definition, parameters, and identity are correct |
| 19 | Policy > Compliance | Open assignment compliance | Compliance results are visible |
| 20 | Policy > Remediation | Create remediation task if supported | Remediation task starts |
| 21 | Policy > Compliance | Recheck compliance state | Remediation effect is visible |
| 22 | Activity log | Review assignment and remediation operations | Change evidence is available |

# Configure_Azure_Policy_Initiatives_Assignments_And_Remediation_Azure_CLI_Precheck_Skeleton

```bash
# Run in Azure Cloud Shell or an admin workstation with Azure CLI.
# Purpose: capture Azure Policy baseline before definition, initiative, assignment, or remediation changes.

TENANT_ID="<tenant-id>"
SUBSCRIPTION_ID="<subscription-id>"
MANAGEMENT_GROUP_ID="<management-group-id>"
RESOURCE_GROUP="<resource-group-name>"
ASSIGNMENT_SCOPE="/subscriptions/$SUBSCRIPTION_ID"
POLICY_DEFINITION_NAME="<policy-definition-name>"
EVIDENCE_PATH="$HOME/azure-governance/04-policy-initiatives-assignments-remediation"

mkdir -p "$EVIDENCE_PATH"

az login --tenant "$TENANT_ID"
az account set --subscription "$SUBSCRIPTION_ID"

# Confirm account context.
az account show \
  --query "{Name:name, SubscriptionId:id, TenantId:tenantId, State:state}" \
  -o json | tee "$EVIDENCE_PATH/active-context.json"

# Confirm resource group if resource group scope is used.
az group show \
  --name "$RESOURCE_GROUP" \
  -o json | tee "$EVIDENCE_PATH/resource-group-scope.json"

# Capture visible policy definitions.
az policy definition list \
  --query "[].{DisplayName:displayName, Name:name, Id:id, PolicyType:policyType, Mode:mode}" \
  -o json | tee "$EVIDENCE_PATH/policy-definitions-visible.json"

# Search for a policy definition by display name.
az policy definition list \
  --query "[?displayName=='$POLICY_DEFINITION_NAME'].{DisplayName:displayName, Name:name, Id:id, PolicyType:policyType, Mode:mode}" \
  -o json | tee "$EVIDENCE_PATH/policy-definition-selected.json"

# Capture visible initiatives.
az policy set-definition list \
  --query "[].{DisplayName:displayName, Name:name, Id:id, PolicyType:policyType}" \
  -o json | tee "$EVIDENCE_PATH/policy-set-definitions-visible.json"

# Capture current policy assignments at target scope.
az policy assignment list \
  --scope "$ASSIGNMENT_SCOPE" \
  --query "[].{DisplayName:displayName, Name:name, Id:id, Scope:scope, EnforcementMode:enforcementMode}" \
  -o json | tee "$EVIDENCE_PATH/policy-assignments-before.json"

# Capture current compliance state.
az policy state list \
  --subscription "$SUBSCRIPTION_ID" \
  --query "[].{ResourceId:resourceId, PolicyAssignmentName:policyAssignmentName, PolicyDefinitionName:policyDefinitionName, ComplianceState:complianceState}" \
  -o json | tee "$EVIDENCE_PATH/policy-state-before.json"
```

# Configure_Azure_Policy_Initiatives_Assignments_And_Remediation_PowerShell_Precheck_Skeleton

```powershell
# Run in elevated PowerShell or Azure Cloud Shell.
# Purpose: capture Azure Policy baseline before definition, initiative, assignment, or remediation changes.

$TenantId = "<tenant-id>"
$SubscriptionId = "<subscription-id>"
$ManagementGroupId = "<management-group-id>"
$ResourceGroupName = "<resource-group-name>"
$AssignmentScope = "/subscriptions/$SubscriptionId"
$PolicyDefinitionDisplayName = "<policy-definition-name>"
$EvidencePath = "C:\Azure-Governance\04-Policy-Initiatives-Assignments-Remediation"

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Import-Module Az.Accounts
Import-Module Az.Resources
Import-Module Az.PolicyInsights

Connect-AzAccount -Tenant $TenantId
Set-AzContext -SubscriptionId $SubscriptionId

# Confirm context.
Get-AzContext |
  Select-Object Name,Subscription,Tenant,Account |
  Tee-Object "$EvidencePath\active-context.txt"

# Confirm target resource group if used.
Get-AzResourceGroup -Name $ResourceGroupName -ErrorAction SilentlyContinue |
  Tee-Object "$EvidencePath\resource-group-scope.txt"

# Capture visible policy definitions.
Get-AzPolicyDefinition |
  Select-Object DisplayName,Name,PolicyType,ResourceId,Properties |
  Export-Csv "$EvidencePath\policy-definitions-visible.csv" -NoTypeInformation

# Search for selected definition.
Get-AzPolicyDefinition |
  Where-Object { $_.Properties.DisplayName -eq $PolicyDefinitionDisplayName } |
  ConvertTo-Json -Depth 20 |
  Out-File "$EvidencePath\policy-definition-selected.json"

# Capture visible initiatives.
Get-AzPolicySetDefinition |
  Select-Object DisplayName,Name,PolicyType,ResourceId |
  Export-Csv "$EvidencePath\policy-set-definitions-visible.csv" -NoTypeInformation

# Capture current assignments.
Get-AzPolicyAssignment -Scope $AssignmentScope |
  Select-Object Name,ResourceId,Properties |
  ConvertTo-Json -Depth 20 |
  Out-File "$EvidencePath\policy-assignments-before.json"

# Capture compliance state.
Get-AzPolicyState -SubscriptionId $SubscriptionId |
  Select-Object Timestamp,ResourceId,PolicyAssignmentName,PolicyDefinitionName,ComplianceState |
  Export-Csv "$EvidencePath\policy-state-before.csv" -NoTypeInformation
```

# Configure_Azure_Policy_Initiatives_Assignments_And_Remediation_Policy_Definition_Discovery_Skeleton

```powershell
# Purpose: discover built-in policies before assigning anything.
# Start with audit policies unless a production change plan explicitly approves enforcement.

$TenantId = "<tenant-id>"
$SubscriptionId = "<subscription-id>"
$SearchText = "Allowed locations"
$EvidencePath = "C:\Azure-Governance\04-Policy-Initiatives-Assignments-Remediation"

Connect-AzAccount -Tenant $TenantId
Set-AzContext -SubscriptionId $SubscriptionId

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

# Search policy definitions by display name.
$Definitions = Get-AzPolicyDefinition |
  Where-Object {
    $_.Properties.DisplayName -like "*$SearchText*"
  }

$Definitions |
  Select-Object Name,ResourceId,PolicyType,@{Name="DisplayName";Expression={$_.Properties.DisplayName}} |
  Tee-Object "$EvidencePath\policy-definition-search-results.txt"

# Export full selected definition for review.
$SelectedDefinition = $Definitions | Select-Object -First 1

$SelectedDefinition |
  ConvertTo-Json -Depth 30 |
  Out-File "$EvidencePath\selected-policy-definition-full.json"

# Inspect policy parameters and rule.
$SelectedDefinition.Properties.Parameters |
  ConvertTo-Json -Depth 20 |
  Out-File "$EvidencePath\selected-policy-definition-parameters.json"

$SelectedDefinition.Properties.PolicyRule |
  ConvertTo-Json -Depth 20 |
  Out-File "$EvidencePath\selected-policy-definition-rule.json"
```

```bash
# Azure CLI equivalent.

SUBSCRIPTION_ID="<subscription-id>"
SEARCH_TEXT="Allowed locations"
EVIDENCE_PATH="$HOME/azure-governance/04-policy-initiatives-assignments-remediation"

mkdir -p "$EVIDENCE_PATH"

az account set --subscription "$SUBSCRIPTION_ID"

# Search policy definitions by display name.
az policy definition list \
  --query "[?contains(displayName, '$SEARCH_TEXT')].{DisplayName:displayName, Name:name, Id:id, PolicyType:policyType, Mode:mode}" \
  -o json | tee "$EVIDENCE_PATH/policy-definition-search-results.json"

# Save a known policy definition.
az policy definition show \
  --id "<policy-definition-id>" \
  -o json | tee "$EVIDENCE_PATH/selected-policy-definition-full.json"

# Save only parameters.
az policy definition show \
  --id "<policy-definition-id>" \
  --query "parameters" \
  -o json | tee "$EVIDENCE_PATH/selected-policy-definition-parameters.json"

# Save only policy rule.
az policy definition show \
  --id "<policy-definition-id>" \
  --query "policyRule" \
  -o json | tee "$EVIDENCE_PATH/selected-policy-definition-rule.json"
```

# Configure_Azure_Policy_Initiatives_Assignments_And_Remediation_Initiative_Assignment_Skeleton

```powershell
# Purpose: create a basic custom initiative and assign it to a scope.
# Use this only after validating selected policy definitions and parameters.

$TenantId = "<tenant-id>"
$SubscriptionId = "<subscription-id>"
$AssignmentScope = "/subscriptions/$SubscriptionId"
$InitiativeName = "baseline-governance-initiative"
$InitiativeDisplayName = "Baseline Governance Initiative"
$InitiativeDescription = "Baseline governance controls for location, tags, and diagnostics."
$AssignmentName = "assign-baseline-governance"
$AssignmentDisplayName = "Baseline Governance Assignment"
$AssignmentLocation = "eastus"
$EvidencePath = "C:\Azure-Governance\04-Policy-Initiatives-Assignments-Remediation"

$PolicyDefinitionId1 = "<policy-definition-id-1>"
$PolicyDefinitionId2 = "<policy-definition-id-2>"

Connect-AzAccount -Tenant $TenantId
Set-AzContext -SubscriptionId $SubscriptionId

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\initiative-assignment-transcript.txt"

# Build initiative policy definition references.
$PolicyDefinitions = @(
  @{
    policyDefinitionId = $PolicyDefinitionId1
    policyDefinitionReferenceId = "policyRef-allowed-locations"
  },
  @{
    policyDefinitionId = $PolicyDefinitionId2
    policyDefinitionReferenceId = "policyRef-required-tag"
  }
)

$PolicyDefinitions |
  ConvertTo-Json -Depth 20 |
  Out-File "$EvidencePath\initiative-policy-definitions.json"

# Create initiative definition.
$Initiative = New-AzPolicySetDefinition `
  -Name $InitiativeName `
  -DisplayName $InitiativeDisplayName `
  -Description $InitiativeDescription `
  -PolicyDefinition $PolicyDefinitions `
  -Metadata '{"category":"Governance"}'

$Initiative |
  ConvertTo-Json -Depth 30 |
  Out-File "$EvidencePath\initiative-definition-created.json"

# Assign initiative.
$Assignment = New-AzPolicyAssignment `
  -Name $AssignmentName `
  -DisplayName $AssignmentDisplayName `
  -Scope $AssignmentScope `
  -PolicySetDefinition $Initiative `
  -EnforcementMode DoNotEnforce `
  -Location $AssignmentLocation

$Assignment |
  ConvertTo-Json -Depth 30 |
  Out-File "$EvidencePath\initiative-assignment-created.json"

Stop-Transcript
```

```bash
# Azure CLI equivalent.
# Note: initiative definitions are easier to maintain with JSON files.

SUBSCRIPTION_ID="<subscription-id>"
ASSIGNMENT_SCOPE="/subscriptions/$SUBSCRIPTION_ID"
INITIATIVE_NAME="baseline-governance-initiative"
INITIATIVE_DISPLAY_NAME="Baseline Governance Initiative"
ASSIGNMENT_NAME="assign-baseline-governance"
ASSIGNMENT_DISPLAY_NAME="Baseline Governance Assignment"
EVIDENCE_PATH="$HOME/azure-governance/04-policy-initiatives-assignments-remediation"

mkdir -p "$EVIDENCE_PATH"

az account set --subscription "$SUBSCRIPTION_ID"

cat > "$EVIDENCE_PATH/initiative-definitions.json" <<EOF
[
  {
    "policyDefinitionId": "<policy-definition-id-1>",
    "policyDefinitionReferenceId": "policyRef-allowed-locations"
  },
  {
    "policyDefinitionId": "<policy-definition-id-2>",
    "policyDefinitionReferenceId": "policyRef-required-tag"
  }
]
EOF

az policy set-definition create \
  --name "$INITIATIVE_NAME" \
  --display-name "$INITIATIVE_DISPLAY_NAME" \
  --description "Baseline governance controls for location, tags, and diagnostics." \
  --definitions "$EVIDENCE_PATH/initiative-definitions.json" \
  --metadata category=Governance \
  -o json | tee "$EVIDENCE_PATH/initiative-definition-created.json"

INITIATIVE_ID=$(az policy set-definition show \
  --name "$INITIATIVE_NAME" \
  --query id \
  -o tsv)

az policy assignment create \
  --name "$ASSIGNMENT_NAME" \
  --display-name "$ASSIGNMENT_DISPLAY_NAME" \
  --scope "$ASSIGNMENT_SCOPE" \
  --policy-set-definition "$INITIATIVE_ID" \
  --enforcement-mode DoNotEnforce \
  -o json | tee "$EVIDENCE_PATH/initiative-assignment-created.json"
```

# Configure_Azure_Policy_Initiatives_Assignments_And_Remediation_Policy_Assignment_Skeleton

```powershell
# Purpose: assign a single built-in policy definition with parameters.
# Example pattern: allowed locations or required tag.

$TenantId = "<tenant-id>"
$SubscriptionId = "<subscription-id>"
$AssignmentScope = "/subscriptions/$SubscriptionId"
$PolicyDefinitionId = "<policy-definition-id>"
$AssignmentName = "assign-allowed-locations-audit"
$AssignmentDisplayName = "Allowed Locations Audit"
$AssignmentDescription = "Audits resources that are deployed outside approved regions."
$AssignmentLocation = "eastus"
$EvidencePath = "C:\Azure-Governance\04-Policy-Initiatives-Assignments-Remediation"

$PolicyParameters = @{
  listOfAllowedLocations = @{
    value = @(
      "eastus",
      "eastus2"
    )
  }
}

Connect-AzAccount -Tenant $TenantId
Set-AzContext -SubscriptionId $SubscriptionId

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\single-policy-assignment-transcript.txt"

$PolicyDefinition = Get-AzPolicyDefinition -Id $PolicyDefinitionId

$PolicyDefinition |
  ConvertTo-Json -Depth 30 |
  Out-File "$EvidencePath\single-policy-definition-selected.json"

$Assignment = New-AzPolicyAssignment `
  -Name $AssignmentName `
  -DisplayName $AssignmentDisplayName `
  -Description $AssignmentDescription `
  -Scope $AssignmentScope `
  -PolicyDefinition $PolicyDefinition `
  -PolicyParameterObject $PolicyParameters `
  -EnforcementMode DoNotEnforce `
  -Location $AssignmentLocation

$Assignment |
  ConvertTo-Json -Depth 30 |
  Out-File "$EvidencePath\single-policy-assignment-created.json"

Stop-Transcript
```

```bash
# Azure CLI equivalent.

SUBSCRIPTION_ID="<subscription-id>"
ASSIGNMENT_SCOPE="/subscriptions/$SUBSCRIPTION_ID"
POLICY_DEFINITION_ID="<policy-definition-id>"
ASSIGNMENT_NAME="assign-allowed-locations-audit"
ASSIGNMENT_DISPLAY_NAME="Allowed Locations Audit"
EVIDENCE_PATH="$HOME/azure-governance/04-policy-initiatives-assignments-remediation"

mkdir -p "$EVIDENCE_PATH"

az account set --subscription "$SUBSCRIPTION_ID"

cat > "$EVIDENCE_PATH/policy-parameters.json" <<EOF
{
  "listOfAllowedLocations": {
    "value": [
      "eastus",
      "eastus2"
    ]
  }
}
EOF

az policy definition show \
  --id "$POLICY_DEFINITION_ID" \
  -o json | tee "$EVIDENCE_PATH/single-policy-definition-selected.json"

az policy assignment create \
  --name "$ASSIGNMENT_NAME" \
  --display-name "$ASSIGNMENT_DISPLAY_NAME" \
  --description "Audits resources that are deployed outside approved regions." \
  --scope "$ASSIGNMENT_SCOPE" \
  --policy "$POLICY_DEFINITION_ID" \
  --params "$EVIDENCE_PATH/policy-parameters.json" \
  --enforcement-mode DoNotEnforce \
  -o json | tee "$EVIDENCE_PATH/single-policy-assignment-created.json"
```

# Configure_Azure_Policy_Initiatives_Assignments_And_Remediation_Exclusion_And_Enforcement_Mode_Skeleton

```powershell
# Purpose: assign policy with a documented exclusion and staged enforcement.
# Use DoNotEnforce to test impact before enforcing deny, modify, or deployIfNotExists behavior.

$TenantId = "<tenant-id>"
$SubscriptionId = "<subscription-id>"
$AssignmentScope = "/subscriptions/$SubscriptionId"
$ExcludedScope = "/subscriptions/$SubscriptionId/resourceGroups/<excluded-resource-group>"
$PolicyDefinitionId = "<policy-definition-id>"
$AssignmentName = "assign-required-tag-stage"
$AssignmentDisplayName = "Required Tag Policy Staged"
$AssignmentLocation = "eastus"
$EvidencePath = "C:\Azure-Governance\04-Policy-Initiatives-Assignments-Remediation"

$PolicyParameters = @{
  tagName = @{
    value = "CostCenter"
  }
}

Connect-AzAccount -Tenant $TenantId
Set-AzContext -SubscriptionId $SubscriptionId

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\exclusion-enforcement-mode-transcript.txt"

$PolicyDefinition = Get-AzPolicyDefinition -Id $PolicyDefinitionId

$Assignment = New-AzPolicyAssignment `
  -Name $AssignmentName `
  -DisplayName $AssignmentDisplayName `
  -Scope $AssignmentScope `
  -NotScope $ExcludedScope `
  -PolicyDefinition $PolicyDefinition `
  -PolicyParameterObject $PolicyParameters `
  -EnforcementMode DoNotEnforce `
  -Location $AssignmentLocation

$Assignment |
  ConvertTo-Json -Depth 30 |
  Out-File "$EvidencePath\policy-assignment-with-exclusion-created.json"

# Document exclusion reason.
[PSCustomObject]@{
  AssignmentName  = $AssignmentName
  AssignmentScope = $AssignmentScope
  ExcludedScope   = $ExcludedScope
  EnforcementMode = "DoNotEnforce"
  Reason          = "Lab staging exclusion. Review before production use."
  ReviewRequired  = $true
} | Export-Csv "$EvidencePath\policy-exclusion-record.csv" -NoTypeInformation

Stop-Transcript
```

```bash
# Azure CLI equivalent.

SUBSCRIPTION_ID="<subscription-id>"
ASSIGNMENT_SCOPE="/subscriptions/$SUBSCRIPTION_ID"
EXCLUDED_SCOPE="/subscriptions/$SUBSCRIPTION_ID/resourceGroups/<excluded-resource-group>"
POLICY_DEFINITION_ID="<policy-definition-id>"
ASSIGNMENT_NAME="assign-required-tag-stage"
ASSIGNMENT_DISPLAY_NAME="Required Tag Policy Staged"
EVIDENCE_PATH="$HOME/azure-governance/04-policy-initiatives-assignments-remediation"

mkdir -p "$EVIDENCE_PATH"

az account set --subscription "$SUBSCRIPTION_ID"

cat > "$EVIDENCE_PATH/required-tag-parameters.json" <<EOF
{
  "tagName": {
    "value": "CostCenter"
  }
}
EOF

az policy assignment create \
  --name "$ASSIGNMENT_NAME" \
  --display-name "$ASSIGNMENT_DISPLAY_NAME" \
  --scope "$ASSIGNMENT_SCOPE" \
  --not-scopes "$EXCLUDED_SCOPE" \
  --policy "$POLICY_DEFINITION_ID" \
  --params "$EVIDENCE_PATH/required-tag-parameters.json" \
  --enforcement-mode DoNotEnforce \
  -o json | tee "$EVIDENCE_PATH/policy-assignment-with-exclusion-created.json"

cat > "$EVIDENCE_PATH/policy-exclusion-record.txt" <<EOF
AssignmentName=$ASSIGNMENT_NAME
AssignmentScope=$ASSIGNMENT_SCOPE
ExcludedScope=$EXCLUDED_SCOPE
EnforcementMode=DoNotEnforce
Reason=Lab staging exclusion. Review before production use.
ReviewRequired=true
EOF
```

# Configure_Azure_Policy_Initiatives_Assignments_And_Remediation_Remediation_Identity_Skeleton

```powershell
# Purpose: create a policy assignment with a managed identity for modify or deployIfNotExists remediation.
# The managed identity needs minimum required RBAC rights to update or deploy target resources.

$TenantId = "<tenant-id>"
$SubscriptionId = "<subscription-id>"
$AssignmentScope = "/subscriptions/$SubscriptionId/resourceGroups/<resource-group-name>"
$PolicyDefinitionId = "<modify-or-deployifnotexists-policy-definition-id>"
$AssignmentName = "assign-remediate-required-tag"
$AssignmentDisplayName = "Remediate Required Tag"
$AssignmentLocation = "eastus"
$RemediationRoleDefinitionName = "<remediation-role-name>"
$EvidencePath = "C:\Azure-Governance\04-Policy-Initiatives-Assignments-Remediation"

$PolicyParameters = @{
  tagName = @{
    value = "CostCenter"
  }
  tagValue = @{
    value = "IT-1001"
  }
}

Connect-AzAccount -Tenant $TenantId
Set-AzContext -SubscriptionId $SubscriptionId

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\remediation-identity-transcript.txt"

$PolicyDefinition = Get-AzPolicyDefinition -Id $PolicyDefinitionId

# Create assignment with system-assigned managed identity.
$Assignment = New-AzPolicyAssignment `
  -Name $AssignmentName `
  -DisplayName $AssignmentDisplayName `
  -Scope $AssignmentScope `
  -PolicyDefinition $PolicyDefinition `
  -PolicyParameterObject $PolicyParameters `
  -Location $AssignmentLocation `
  -IdentityType SystemAssigned

$Assignment |
  ConvertTo-Json -Depth 30 |
  Out-File "$EvidencePath\policy-assignment-managed-identity-created.json"

# Capture principal ID.
$AssignmentIdentityPrincipalId = $Assignment.Identity.PrincipalId

[PSCustomObject]@{
  AssignmentName = $AssignmentName
  AssignmentId   = $Assignment.ResourceId
  PrincipalId    = $AssignmentIdentityPrincipalId
  Scope          = $AssignmentScope
  RoleNeeded     = $RemediationRoleDefinitionName
} | Tee-Object "$EvidencePath\assignment-managed-identity-record.txt"

# Grant minimum remediation role manually if needed.
# Portal-created assignments may grant roles automatically when roleDefinitionIds are present.
New-AzRoleAssignment `
  -ObjectId $AssignmentIdentityPrincipalId `
  -RoleDefinitionName $RemediationRoleDefinitionName `
  -Scope $AssignmentScope

# Validate managed identity role assignment.
Get-AzRoleAssignment `
  -ObjectId $AssignmentIdentityPrincipalId `
  -Scope $AssignmentScope |
  Select-Object DisplayName,ObjectType,RoleDefinitionName,Scope,ObjectId |
  Tee-Object "$EvidencePath\assignment-managed-identity-rbac.txt"

Stop-Transcript
```

```bash
# Azure CLI equivalent.

SUBSCRIPTION_ID="<subscription-id>"
ASSIGNMENT_SCOPE="/subscriptions/$SUBSCRIPTION_ID/resourceGroups/<resource-group-name>"
POLICY_DEFINITION_ID="<modify-or-deployifnotexists-policy-definition-id>"
ASSIGNMENT_NAME="assign-remediate-required-tag"
ASSIGNMENT_DISPLAY_NAME="Remediate Required Tag"
ASSIGNMENT_LOCATION="eastus"
REMEDIATION_ROLE_NAME="<remediation-role-name>"
EVIDENCE_PATH="$HOME/azure-governance/04-policy-initiatives-assignments-remediation"

mkdir -p "$EVIDENCE_PATH"

az account set --subscription "$SUBSCRIPTION_ID"

cat > "$EVIDENCE_PATH/remediation-policy-parameters.json" <<EOF
{
  "tagName": {
    "value": "CostCenter"
  },
  "tagValue": {
    "value": "IT-1001"
  }
}
EOF

az policy assignment create \
  --name "$ASSIGNMENT_NAME" \
  --display-name "$ASSIGNMENT_DISPLAY_NAME" \
  --scope "$ASSIGNMENT_SCOPE" \
  --policy "$POLICY_DEFINITION_ID" \
  --params "$EVIDENCE_PATH/remediation-policy-parameters.json" \
  --mi-system-assigned \
  --location "$ASSIGNMENT_LOCATION" \
  -o json | tee "$EVIDENCE_PATH/policy-assignment-managed-identity-created.json"

ASSIGNMENT_ID=$(az policy assignment show \
  --name "$ASSIGNMENT_NAME" \
  --scope "$ASSIGNMENT_SCOPE" \
  --query id \
  -o tsv)

PRINCIPAL_ID=$(az policy assignment show \
  --name "$ASSIGNMENT_NAME" \
  --scope "$ASSIGNMENT_SCOPE" \
  --query identity.principalId \
  -o tsv)

echo "$ASSIGNMENT_ID" | tee "$EVIDENCE_PATH/assignment-id.txt"
echo "$PRINCIPAL_ID" | tee "$EVIDENCE_PATH/assignment-managed-identity-principal-id.txt"

# Grant remediation role manually when required.
az role assignment create \
  --assignee-object-id "$PRINCIPAL_ID" \
  --assignee-principal-type ServicePrincipal \
  --role "$REMEDIATION_ROLE_NAME" \
  --scope "$ASSIGNMENT_SCOPE" \
  -o json | tee "$EVIDENCE_PATH/assignment-managed-identity-rbac.json"
```

# Configure_Azure_Policy_Initiatives_Assignments_And_Remediation_Remediation_Task_Skeleton

```powershell
# Purpose: create a remediation task for an assignment that uses modify or deployIfNotExists.
# Remediation applies to existing non-compliant resources. It does not replace future policy evaluation.

$TenantId = "<tenant-id>"
$SubscriptionId = "<subscription-id>"
$AssignmentScope = "/subscriptions/$SubscriptionId/resourceGroups/<resource-group-name>"
$AssignmentName = "assign-remediate-required-tag"
$RemediationTaskName = "remediate-required-tag-existing-resources"
$EvidencePath = "C:\Azure-Governance\04-Policy-Initiatives-Assignments-Remediation"

Connect-AzAccount -Tenant $TenantId
Set-AzContext -SubscriptionId $SubscriptionId

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\remediation-task-transcript.txt"

# Get assignment.
$Assignment = Get-AzPolicyAssignment `
  -Name $AssignmentName `
  -Scope $AssignmentScope

$Assignment |
  ConvertTo-Json -Depth 30 |
  Out-File "$EvidencePath\assignment-before-remediation.json"

# Start remediation.
$Remediation = Start-AzPolicyRemediation `
  -Name $RemediationTaskName `
  -PolicyAssignmentId $Assignment.ResourceId `
  -ResourceDiscoveryMode ReEvaluateCompliance

$Remediation |
  ConvertTo-Json -Depth 30 |
  Out-File "$EvidencePath\remediation-task-created.json"

# Validate remediation task.
Get-AzPolicyRemediation `
  -Name $RemediationTaskName `
  -Scope $AssignmentScope |
  ConvertTo-Json -Depth 30 |
  Out-File "$EvidencePath\remediation-task-status.json"

Stop-Transcript
```

```bash
# Azure CLI equivalent.

SUBSCRIPTION_ID="<subscription-id>"
ASSIGNMENT_SCOPE="/subscriptions/$SUBSCRIPTION_ID/resourceGroups/<resource-group-name>"
ASSIGNMENT_NAME="assign-remediate-required-tag"
REMEDIATION_TASK_NAME="remediate-required-tag-existing-resources"
EVIDENCE_PATH="$HOME/azure-governance/04-policy-initiatives-assignments-remediation"

mkdir -p "$EVIDENCE_PATH"

az account set --subscription "$SUBSCRIPTION_ID"

ASSIGNMENT_ID=$(az policy assignment show \
  --name "$ASSIGNMENT_NAME" \
  --scope "$ASSIGNMENT_SCOPE" \
  --query id \
  -o tsv)

echo "$ASSIGNMENT_ID" | tee "$EVIDENCE_PATH/remediation-assignment-id.txt"

# Create remediation task.
az policy remediation create \
  --name "$REMEDIATION_TASK_NAME" \
  --policy-assignment "$ASSIGNMENT_ID" \
  --resource-discovery-mode ReEvaluateCompliance \
  --scope "$ASSIGNMENT_SCOPE" \
  -o json | tee "$EVIDENCE_PATH/remediation-task-created.json"

# Validate remediation task.
az policy remediation show \
  --name "$REMEDIATION_TASK_NAME" \
  --scope "$ASSIGNMENT_SCOPE" \
  -o json | tee "$EVIDENCE_PATH/remediation-task-status.json"

# List remediation tasks at scope.
az policy remediation list \
  --scope "$ASSIGNMENT_SCOPE" \
  -o json | tee "$EVIDENCE_PATH/remediation-tasks-visible.json"
```

# Configure_Azure_Policy_Initiatives_Assignments_And_Remediation_Compliance_Validation_Skeleton

```powershell
# Purpose: trigger scan and validate policy compliance after assignment or remediation.

$TenantId = "<tenant-id>"
$SubscriptionId = "<subscription-id>"
$ResourceGroupName = "<resource-group-name>"
$AssignmentName = "<assignment-name>"
$EvidencePath = "C:\Azure-Governance\04-Policy-Initiatives-Assignments-Remediation"

Connect-AzAccount -Tenant $TenantId
Set-AzContext -SubscriptionId $SubscriptionId

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\compliance-validation-transcript.txt"

# Trigger policy compliance scan.
Start-AzPolicyComplianceScan -SubscriptionId $SubscriptionId

# Capture all policy states.
Get-AzPolicyState -SubscriptionId $SubscriptionId |
  Select-Object Timestamp,ResourceId,PolicyAssignmentName,PolicyDefinitionName,ComplianceState |
  Export-Csv "$EvidencePath\policy-state-after-scan.csv" -NoTypeInformation

# Capture non-compliant resources.
Get-AzPolicyState -SubscriptionId $SubscriptionId |
  Where-Object { $_.ComplianceState -eq "NonCompliant" } |
  Select-Object Timestamp,ResourceId,PolicyAssignmentName,PolicyDefinitionName,ComplianceState |
  Export-Csv "$EvidencePath\non-compliant-resources-after-scan.csv" -NoTypeInformation

# Capture compliance for one assignment.
Get-AzPolicyState -SubscriptionId $SubscriptionId |
  Where-Object { $_.PolicyAssignmentName -eq $AssignmentName } |
  Select-Object Timestamp,ResourceId,PolicyAssignmentName,PolicyDefinitionName,ComplianceState |
  Export-Csv "$EvidencePath\assignment-compliance-after-scan.csv" -NoTypeInformation

# Capture policy events for troubleshooting.
Get-AzPolicyEvent -SubscriptionId $SubscriptionId |
  Select-Object Timestamp,ResourceId,PolicyAssignmentName,PolicyDefinitionName,ComplianceState,PolicyDefinitionAction |
  Export-Csv "$EvidencePath\policy-events.csv" -NoTypeInformation

Stop-Transcript
```

```bash
# Azure CLI equivalent.

SUBSCRIPTION_ID="<subscription-id>"
RESOURCE_GROUP="<resource-group-name>"
ASSIGNMENT_NAME="<assignment-name>"
EVIDENCE_PATH="$HOME/azure-governance/04-policy-initiatives-assignments-remediation"

mkdir -p "$EVIDENCE_PATH"

az account set --subscription "$SUBSCRIPTION_ID"

# Trigger policy compliance scan.
az policy state trigger-scan \
  --subscription "$SUBSCRIPTION_ID"

# Capture all policy states.
az policy state list \
  --subscription "$SUBSCRIPTION_ID" \
  --query "[].{ResourceId:resourceId, PolicyAssignmentName:policyAssignmentName, PolicyDefinitionName:policyDefinitionName, ComplianceState:complianceState}" \
  -o json | tee "$EVIDENCE_PATH/policy-state-after-scan.json"

# Capture non-compliant resources.
az policy state list \
  --subscription "$SUBSCRIPTION_ID" \
  --filter "complianceState eq 'NonCompliant'" \
  --query "[].{ResourceId:resourceId, PolicyAssignmentName:policyAssignmentName, PolicyDefinitionName:policyDefinitionName, ComplianceState:complianceState}" \
  -o json | tee "$EVIDENCE_PATH/non-compliant-resources-after-scan.json"

# Capture assignment-specific compliance.
az policy state list \
  --subscription "$SUBSCRIPTION_ID" \
  --filter "policyAssignmentName eq '$ASSIGNMENT_NAME'" \
  -o json | tee "$EVIDENCE_PATH/assignment-compliance-after-scan.json"

# Summarize compliance.
az policy state summarize \
  --subscription "$SUBSCRIPTION_ID" \
  -o json | tee "$EVIDENCE_PATH/policy-state-summary.json"
```

# Configure_Azure_Policy_Initiatives_Assignments_And_Remediation_Evidence_Export_Skeleton

```powershell
# Purpose: export final evidence for definitions, initiatives, assignments, compliance, remediation, identities, and activity logs.

$TenantId = "<tenant-id>"
$SubscriptionId = "<subscription-id>"
$ManagementGroupId = "<management-group-id>"
$ResourceGroupName = "<resource-group-name>"
$AssignmentScope = "/subscriptions/$SubscriptionId"
$AssignmentName = "<assignment-name>"
$InitiativeName = "<initiative-name>"
$RemediationTaskName = "<remediation-task-name>"
$EvidencePath = "C:\Azure-Governance\04-Policy-Initiatives-Assignments-Remediation"

Connect-AzAccount -Tenant $TenantId
Set-AzContext -SubscriptionId $SubscriptionId

New-Item -ItemType Directory -Force -Path $EvidencePath | Out-Null

Start-Transcript -Path "$EvidencePath\04-policy-final-evidence-transcript.txt"

# Tenant and subscription evidence.
Get-AzTenant |
  Select-Object Name,Id |
  Export-Csv "$EvidencePath\tenants.csv" -NoTypeInformation

Get-AzSubscription |
  Select-Object Name,Id,TenantId,State |
  Export-Csv "$EvidencePath\subscriptions.csv" -NoTypeInformation

Get-AzContext |
  Format-List |
  Out-File "$EvidencePath\active-context.txt"

# Resource group scope evidence.
Get-AzResourceGroup -Name $ResourceGroupName -ErrorAction SilentlyContinue |
  ConvertTo-Json -Depth 10 |
  Out-File "$EvidencePath\resource-group-scope-final.json"

# Policy definitions and initiatives.
Get-AzPolicyDefinition |
  Select-Object Name,ResourceId,PolicyType,@{Name="DisplayName";Expression={$_.Properties.DisplayName}} |
  Export-Csv "$EvidencePath\policy-definitions-visible-final.csv" -NoTypeInformation

Get-AzPolicySetDefinition |
  Select-Object Name,ResourceId,PolicyType,@{Name="DisplayName";Expression={$_.Properties.DisplayName}} |
  Export-Csv "$EvidencePath\policy-set-definitions-visible-final.csv" -NoTypeInformation

Get-AzPolicySetDefinition -Name $InitiativeName -ErrorAction SilentlyContinue |
  ConvertTo-Json -Depth 30 |
  Out-File "$EvidencePath\initiative-definition-final.json"

# Policy assignments.
Get-AzPolicyAssignment -Scope $AssignmentScope |
  ConvertTo-Json -Depth 30 |
  Out-File "$EvidencePath\policy-assignments-final.json"

Get-AzPolicyAssignment -Name $AssignmentName -Scope $AssignmentScope -ErrorAction SilentlyContinue |
  ConvertTo-Json -Depth 30 |
  Out-File "$EvidencePath\selected-policy-assignment-final.json"

# Compliance state.
Get-AzPolicyState -SubscriptionId $SubscriptionId |
  Select-Object Timestamp,ResourceId,PolicyAssignmentName,PolicyDefinitionName,ComplianceState |
  Export-Csv "$EvidencePath\policy-state-final.csv" -NoTypeInformation

Get-AzPolicyState -SubscriptionId $SubscriptionId |
  Where-Object { $_.ComplianceState -eq "NonCompliant" } |
  Select-Object Timestamp,ResourceId,PolicyAssignmentName,PolicyDefinitionName,ComplianceState |
  Export-Csv "$EvidencePath\non-compliant-resources-final.csv" -NoTypeInformation

# Remediation evidence.
Get-AzPolicyRemediation -Scope $AssignmentScope -ErrorAction SilentlyContinue |
  ConvertTo-Json -Depth 30 |
  Out-File "$EvidencePath\policy-remediations-final.json"

Get-AzPolicyRemediation -Name $RemediationTaskName -Scope $AssignmentScope -ErrorAction SilentlyContinue |
  ConvertTo-Json -Depth 30 |
  Out-File "$EvidencePath\selected-remediation-final.json"

# Policy-related activity log.
Get-AzActivityLog -StartTime (Get-Date).AddDays(-7) |
  Where-Object {
    $_.OperationName.Value -like "*policy*" -or
    $_.ResourceProviderName.Value -eq "Microsoft.Authorization" -or
    $_.ResourceProviderName.Value -eq "Microsoft.PolicyInsights"
  } |
  Select-Object EventTimestamp,OperationName,ResourceGroupName,ResourceProviderName,Status,Caller,CorrelationId |
  Export-Csv "$EvidencePath\policy-activity-log-last-7-days.csv" -NoTypeInformation

Stop-Transcript
```

# Configure_Azure_Policy_Initiatives_Assignments_And_Remediation_Verification_Commands

```powershell
# Confirm context.
Get-AzContext

# Confirm policy definitions.
Get-AzPolicyDefinition |
  Select-Object Name,ResourceId,@{Name="DisplayName";Expression={$_.Properties.DisplayName}}

# Confirm selected policy definition.
Get-AzPolicyDefinition -Id "<policy-definition-id>" |
  ConvertTo-Json -Depth 20

# Confirm initiatives.
Get-AzPolicySetDefinition |
  Select-Object Name,ResourceId,@{Name="DisplayName";Expression={$_.Properties.DisplayName}}

# Confirm selected initiative.
Get-AzPolicySetDefinition -Name "<initiative-name>" |
  ConvertTo-Json -Depth 20

# Confirm assignments at scope.
Get-AzPolicyAssignment -Scope "<assignment-scope>"

# Confirm selected assignment.
Get-AzPolicyAssignment -Name "<assignment-name>" -Scope "<assignment-scope>" |
  ConvertTo-Json -Depth 20

# Trigger compliance scan.
Start-AzPolicyComplianceScan -SubscriptionId "<subscription-id>"

# Confirm policy state.
Get-AzPolicyState -SubscriptionId "<subscription-id>"

# Confirm non-compliant resources.
Get-AzPolicyState -SubscriptionId "<subscription-id>" |
  Where-Object { $_.ComplianceState -eq "NonCompliant" }

# Confirm assignment-specific state.
Get-AzPolicyState -SubscriptionId "<subscription-id>" |
  Where-Object { $_.PolicyAssignmentName -eq "<assignment-name>" }

# Confirm remediation tasks.
Get-AzPolicyRemediation -Scope "<assignment-scope>"

# Confirm selected remediation task.
Get-AzPolicyRemediation -Name "<remediation-task-name>" -Scope "<assignment-scope>"

# Confirm managed identity RBAC for remediation.
Get-AzRoleAssignment -ObjectId "<assignment-managed-identity-principal-id>" -Scope "<assignment-scope>"
```

```bash
# Azure CLI equivalents.

az account show -o table

az policy definition list \
  --query "[].{DisplayName:displayName, Name:name, Id:id, PolicyType:policyType}" \
  -o table

az policy definition show \
  --id "<policy-definition-id>" \
  -o json

az policy set-definition list \
  --query "[].{DisplayName:displayName, Name:name, Id:id, PolicyType:policyType}" \
  -o table

az policy set-definition show \
  --name "<initiative-name>" \
  -o json

az policy assignment list \
  --scope "<assignment-scope>" \
  --query "[].{DisplayName:displayName, Name:name, Id:id, EnforcementMode:enforcementMode, Scope:scope}" \
  -o table

az policy assignment show \
  --name "<assignment-name>" \
  --scope "<assignment-scope>" \
  -o json

az policy state trigger-scan \
  --subscription "<subscription-id>"

az policy state list \
  --subscription "<subscription-id>" \
  -o table

az policy state list \
  --subscription "<subscription-id>" \
  --filter "complianceState eq 'NonCompliant'" \
  -o table

az policy state summarize \
  --subscription "<subscription-id>" \
  -o json

az policy remediation list \
  --scope "<assignment-scope>" \
  -o table

az policy remediation show \
  --name "<remediation-task-name>" \
  --scope "<assignment-scope>" \
  -o json

az role assignment list \
  --assignee "<assignment-managed-identity-principal-id>" \
  --scope "<assignment-scope>" \
  -o table
```

# Configure_Azure_Policy_Initiatives_Assignments_And_Remediation_Rollback

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm assignment before rollback | Cloud Shell | `az policy assignment show --name "<assignment-name>" --scope "<assignment-scope>"` | Assignment exists |
| 2 | Export assignment details | Cloud Shell | `az policy assignment show --name "<assignment-name>" --scope "<assignment-scope>" -o json` | Rollback evidence is saved |
| 3 | Stop or avoid new remediation | Cloud Shell | Do not create new remediation task | No new changes are pushed |
| 4 | Delete remediation task if lab-only | Cloud Shell | `az policy remediation delete --name "<remediation-task-name>" --scope "<assignment-scope>"` | Remediation task is removed |
| 5 | Delete policy assignment | Cloud Shell | `az policy assignment delete --name "<assignment-name>" --scope "<assignment-scope>"` | Assignment is removed |
| 6 | Remove managed identity RBAC if manually granted | Cloud Shell | `az role assignment delete --assignee "<principal-id>" --role "<role>" --scope "<scope>"` | Remediation identity no longer has access |
| 7 | Delete custom initiative if lab-only | Cloud Shell | `az policy set-definition delete --name "<initiative-name>"` | Custom initiative is removed |
| 8 | Delete custom policy definition if lab-only | Cloud Shell | `az policy definition delete --name "<policy-definition-name>"` | Custom definition is removed |
| 9 | Re-run compliance scan | Cloud Shell | `az policy state trigger-scan --subscription "<subscription-id>"` | Evaluation refresh starts |
| 10 | Validate assignment removal | Cloud Shell | `az policy assignment list --scope "<assignment-scope>" -o table` | Removed assignment is not listed |
| 11 | Validate no unintended RBAC remains | Cloud Shell | `az role assignment list --assignee "<principal-id>" -o table` | Temporary access is gone |
| 12 | Document rollback | Operator | Record removed assignment, remediation task, identity access, and validation result | Rollback record is complete |

```powershell
# PowerShell rollback examples.

$TenantId = "<tenant-id>"
$SubscriptionId = "<subscription-id>"
$AssignmentScope = "<assignment-scope>"
$AssignmentName = "<assignment-name>"
$RemediationTaskName = "<remediation-task-name>"
$ManagedIdentityPrincipalId = "<assignment-managed-identity-principal-id>"
$RemediationRoleDefinitionName = "<remediation-role-name>"
$InitiativeName = "<initiative-name>"
$CustomPolicyDefinitionName = "<custom-policy-definition-name>"

Connect-AzAccount -Tenant $TenantId
Set-AzContext -SubscriptionId $SubscriptionId

# Export assignment before removal.
Get-AzPolicyAssignment -Name $AssignmentName -Scope $AssignmentScope |
  ConvertTo-Json -Depth 30 |
  Out-File "C:\Azure-Governance\04-Policy-Initiatives-Assignments-Remediation\rollback-policy-assignment-before-delete.json"

# Remove remediation task if lab-only.
Remove-AzPolicyRemediation `
  -Name $RemediationTaskName `
  -Scope $AssignmentScope `
  -Force `
  -ErrorAction SilentlyContinue

# Remove policy assignment.
Remove-AzPolicyAssignment `
  -Name $AssignmentName `
  -Scope $AssignmentScope `
  -Force

# Remove managed identity role assignment if it was manually granted.
Remove-AzRoleAssignment `
  -ObjectId $ManagedIdentityPrincipalId `
  -RoleDefinitionName $RemediationRoleDefinitionName `
  -Scope $AssignmentScope `
  -ErrorAction SilentlyContinue

# Remove lab initiative if custom and no longer needed.
Remove-AzPolicySetDefinition `
  -Name $InitiativeName `
  -Force `
  -ErrorAction SilentlyContinue

# Remove lab custom definition if no longer needed.
Remove-AzPolicyDefinition `
  -Name $CustomPolicyDefinitionName `
  -Force `
  -ErrorAction SilentlyContinue

# Validate final state.
Get-AzPolicyAssignment -Scope $AssignmentScope
Get-AzPolicyRemediation -Scope $AssignmentScope -ErrorAction SilentlyContinue
```

# Configure_Azure_Policy_Initiatives_Assignments_And_Remediation_Failure_Checks

| Symptom | Likely Cause | Device | Command | Fix |
|---|---|---|---|---|
| Cannot create policy assignment | Missing policy assignment permissions | Cloud Shell | `az role assignment list --assignee "<admin-object-id>" --include-inherited -o table` | Grant Resource Policy Contributor, Owner, or a custom role with required policy operations |
| Assignment succeeds but no resources show compliance | Evaluation has not completed yet | Cloud Shell | `az policy state trigger-scan --subscription "<subscription-id>"` | Trigger scan and wait for policy state to update |
| Policy blocks deployment unexpectedly | Deny effect assigned too broadly | Portal / Cloud Shell | `az policy assignment list --scope "<scope>" --include-descendants -o table` | Disable assignment, add exclusion, or stage with audit first |
| Policy does not block deployment | Assignment enforcement mode is DoNotEnforce or effect is audit | Cloud Shell | `az policy assignment show --name "<assignment-name>" --scope "<scope>" --query enforcementMode` | Change enforcement mode only after validation |
| Resource is marked non-compliant even though it looks correct | Parameter value, alias, resource mode, or effect logic mismatch | Cloud Shell | `az policy definition show --id "<policy-definition-id>"` | Review policy rule, parameters, mode, and resource properties |
| Excluded resource still evaluated | Exclusion scope does not match child resource path | Cloud Shell | `az policy assignment show --name "<assignment-name>" --scope "<scope>" --query notScopes` | Correct the excluded scope resource ID |
| Remediation option is unavailable | Policy effect does not support remediation | Portal / Cloud Shell | Inspect policy effect | Use modify or deployIfNotExists policy for remediation |
| Remediation task fails | Assignment managed identity lacks required RBAC | Cloud Shell | `az role assignment list --assignee "<principal-id>" -o table` | Grant minimum required role at remediation scope |
| Managed identity principal ID is blank | Identity has not been created or replicated yet | Cloud Shell | `az policy assignment show --name "<assignment-name>" --scope "<scope>" --query identity` | Reopen assignment, enable identity, wait for replication |
| Portal grants identity roles but CLI assignment does not | Portal can auto-grant listed roles, CLI often requires manual role assignment | Cloud Shell | `az role assignment list --assignee "<principal-id>" -o table` | Manually assign required roles |
| Remediation partially succeeds | Some resources lack provider support, permissions, or valid state | Cloud Shell | `az policy remediation show --name "<remediation-task-name>" --scope "<scope>"` | Review failed deployments and resource-level errors |
| Initiative assignment has parameter errors | Initiative parameter mapping does not match child policy parameters | Cloud Shell | `az policy set-definition show --name "<initiative-name>"` | Correct initiative parameter mapping and recreate assignment |
| Custom definition cannot be deleted | Existing assignment still references it | Cloud Shell | `az policy assignment list --query "[?policyDefinitionId=='<definition-id>']"` | Delete assignments before deleting definition |
| Custom initiative cannot be deleted | Existing assignment still references it | Cloud Shell | `az policy assignment list --query "[?policyDefinitionId=='<initiative-id>']"` | Delete initiative assignment first |
| Compliance scan command fails | Az.PolicyInsights module missing or CLI extension issue | Admin Workstation | `Get-Module Az.PolicyInsights -ListAvailable`; `az version` | Use Azure Cloud Shell or install required module |
| Non-compliance message missing | Assignment did not define one | Portal | Assignment > Non-compliance messages | Add clear message explaining operator action |
| Policy assignment not visible to user | User lacks read access at assignment scope | Portal / Cloud Shell | `az role assignment list --assignee "<user-object-id>" --include-inherited` | Grant Reader or relevant role at scope |
| Remediation changes too much | Assignment scope too broad | Cloud Shell | `az policy assignment show --name "<assignment-name>" --scope "<scope>"` | Narrow scope or add exclusions |
| Diagnostics or tags do not remediate | Policy requires deployIfNotExists or modify and identity rights | Cloud Shell | Inspect policy effect and identity role assignments | Use correct built-in policy and assign managed identity permissions |
| Azure Policy confused with RBAC | Expecting Policy to grant access or RBAC to enforce resource state | Operator | Review RBAC assignments and Policy assignments separately | Use RBAC for user actions and Policy for resource compliance |

# Configure_Azure_Policy_Initiatives_Assignments_And_Remediation_Related_Labs

| Lab                                                                        | Relationship                                                                                                |
| -------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------- |
| 01_Manage_Subscriptions_Management_Groups_And_Tenant_Association.md        | Defines management group and subscription scopes where policy can be assigned                               |
| 02_Manage_Resource_Groups_Tags_Locks_And_Resource_Move_Operations.md       | Provides resource group, tag, lock, and move targets that policy governs                                    |
| 03_Configure_Azure_RBAC_Role_Assignments_And_Access_Reviews.md             | Provides permissions needed to assign policy and grant remediation identity rights                          |
| 05_Configure_Cost_Management_Budgets_Alerts_And_Advisor_Recommendations.md | Uses tags and policy compliance to improve cost accountability                                              |
| 06_Troubleshoot_Azure_Governance_RBAC_Policy_Locks_And_Cost_Issues.md      | Troubleshoots deny effects, remediation failures, inherited policy, locks, RBAC, and cost visibility issues |