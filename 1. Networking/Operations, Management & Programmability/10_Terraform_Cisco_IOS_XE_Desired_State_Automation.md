

Provider-specific syntax was checked against the CiscoDevNet IOS XE Terraform provider docs: the provider supports IOS XE through NETCONF by default, RESTCONF optionally, terraform init installs the provider, and provider fields include host, username, password, and protocol.  

Terraform_Cisco_IOS_XE_Desired_State_Automation.md

Terraform_Cisco_IOS_XE_Desired_State_Automation

# Terraform_Cisco_IOS_XE_Desired_State_Automation_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Terraform | Desired-state automation tool that compares HCL configuration, Terraform state, and real infrastructure |
| HCL | Human-readable Terraform configuration language stored in `.tf` files |
| Provider | Plugin that knows how to talk to a target platform, in this case IOS XE |
| IOS XE provider | Terraform provider used to manage IOS XE resources through NETCONF or RESTCONF |
| Resource | Terraform-managed object that should exist on the device, such as VLAN, ACL, routing, interface, or CLI-backed config |
| Data source | Read-only lookup used to pull existing information from the device |
| State file | Terraform record of objects it believes it manages |
| Plan | Dry-run comparison showing what Terraform will create, change, or destroy |
| Apply | Execution step that pushes the planned desired state to the device |
| Destroy | Removal workflow for Terraform-managed resources |
| Import | Brings existing device config into Terraform state so Terraform can manage it |
| Drift | Difference between Terraform state, HCL intent, and actual device config |
| Idempotence | Re-running apply should make no change if the device already matches the desired state |
| NETCONF default | IOS XE provider defaults to NETCONF, so the device needs `netconf-yang` unless RESTCONF is selected |
| RESTCONF option | RESTCONF requires HTTPS and `restconf`, and the provider must specify RESTCONF protocol |
| Commit behavior | With NETCONF candidate datastore workflows, commit behavior must be understood before using manual commit mode |
| Lab scope | This note covers Terraform workflow against IOS XE. It does not replace NETCONF, RESTCONF, or YANG notes |
| Related labs | `terraform-cisco-ios-xe-router-final` |
# Terraform_Cisco_IOS_XE_Desired_State_Automation_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm IOS XE management reachability | IOS XE router | `show ip interface brief` | Management interface has the expected IP and is up/up |
| 2 | Confirm IOS XE has a return route to the Terraform client | IOS XE router | `show ip route <CLIENT_IP>` | Route exists back toward the Terraform client |
| 3 | Confirm client can reach IOS XE | Terraform client | `ping <DEVICE_IP>` | Client can reach the IOS XE management IP |
| 4 | Confirm local automation user exists | IOS XE router | `show running-config | include ^username` | Expected local user exists |
| 5 | Enable NETCONF if using the default provider transport | IOS XE router | `configure terminal` then `netconf-yang` | NETCONF/YANG is enabled |
| 6 | Optionally enable candidate datastore for staged NETCONF commit workflows | IOS XE router | `netconf-yang feature candidate-datastore` | Candidate datastore is available if the lab requires it |
| 7 | Enable RESTCONF only if the Terraform provider will use RESTCONF | IOS XE router | `ip http authentication local` | HTTP authentication uses local credentials |
| 8 | Enable HTTPS only if using RESTCONF | IOS XE router | `ip http secure-server` | HTTPS server is enabled |
| 9 | Enable RESTCONF only if using RESTCONF | IOS XE router | `restconf` | RESTCONF service is enabled |
| 10 | Save IOS XE programmable access baseline | IOS XE router | `end` then `write memory` | NETCONF or RESTCONF baseline is saved |
| 11 | Verify NETCONF process if using NETCONF | IOS XE router | `show platform software yang-management process` | YANG management processes are running |
| 12 | Verify NETCONF TCP/830 if using NETCONF | Terraform client | `ssh -p 830 -s <USER>@<DEVICE_IP> netconf` | Device returns NETCONF hello and capabilities |
| 13 | Verify RESTCONF if using RESTCONF | Terraform client | `curl -k -u <USER>:<SECRET> -H "Accept: application/yang-data+json" https://<DEVICE_IP>/restconf/data/` | Device returns RESTCONF response |
| 14 | Confirm Terraform binary exists | Terraform client | `terraform version` | Terraform version is displayed |
| 15 | Create a clean lab working directory | Terraform client | `mkdir -p terraform-cisco-ios-xe-router && cd terraform-cisco-ios-xe-router` | Client is inside a dedicated Terraform project directory |
| 16 | Create `.gitignore` for local state and secrets | Terraform client | `cat > .gitignore <<'EOF'` | Ignore file creation begins |
| 17 | Add Terraform ignore entries | Terraform client | `.terraform/`, `*.tfstate`, `*.tfstate.backup`, `.terraform.lock.hcl`, `terraform.tfvars`, `*.tfvars` | Local provider cache, state, lock file, and secrets are not committed |
| 18 | Close `.gitignore` | Terraform client | `EOF` | `.gitignore` exists |
| 19 | Create `versions.tf` | Terraform client | `cat > versions.tf <<'EOF'` | Terraform version and provider constraint file begins |
| 20 | Define IOS XE provider requirement | Terraform client | `required_providers { iosxe = { source = "CiscoDevNet/iosxe" version = ">= 0.18.0" } }` | Terraform knows which provider to install |
| 21 | Close `versions.tf` | Terraform client | `EOF` | `versions.tf` exists |
| 22 | Create `variables.tf` | Terraform client | `cat > variables.tf <<'EOF'` | Variable definition file begins |
| 23 | Define device host variable | Terraform client | `variable "iosxe_host" { type = string }` | Device host is externalized |
| 24 | Define username variable | Terraform client | `variable "iosxe_username" { type = string }` | Username is externalized |
| 25 | Define password variable as sensitive | Terraform client | `variable "iosxe_password" { type = string sensitive = true }` | Password is treated as sensitive |
| 26 | Define protocol variable | Terraform client | `variable "iosxe_protocol" { type = string default = "netconf" }` | NETCONF is default unless overridden |
| 27 | Close `variables.tf` | Terraform client | `EOF` | `variables.tf` exists |
| 28 | Create `provider.tf` | Terraform client | `cat > provider.tf <<'EOF'` | Provider configuration file begins |
| 29 | Configure IOS XE provider | Terraform client | `provider "iosxe" { host = var.iosxe_host username = var.iosxe_username password = var.iosxe_password protocol = var.iosxe_protocol insecure = true }` | Provider has device connection settings |
| 30 | Close `provider.tf` | Terraform client | `EOF` | `provider.tf` exists |
| 31 | Create local lab variables file | Terraform client | `cat > terraform.tfvars <<'EOF'` | Local variable file begins |
| 32 | Set IOS XE host | Terraform client | `iosxe_host = "<DEVICE_IP>"` | Terraform points to the IOS XE router |
| 33 | Set IOS XE username | Terraform client | `iosxe_username = "<USER>"` | Terraform has username |
| 34 | Set IOS XE password | Terraform client | `iosxe_password = "<SECRET>"` | Terraform has password for lab use |
| 35 | Set provider protocol | Terraform client | `iosxe_protocol = "netconf"` | Terraform uses NETCONF unless changed to RESTCONF |
| 36 | Close `terraform.tfvars` | Terraform client | `EOF` | Local lab variables file exists |
| 37 | Create desired-state resource file | Terraform client | `cat > main.tf <<'EOF'` | Main resource file begins |
| 38 | Add IOS XE resource block for the lab objective | Terraform client | `resource "<IOSXE_RESOURCE_TYPE>" "<LOCAL_NAME>" { <ARGUMENTS> }` | Desired IOS XE object is declared |
| 39 | Add optional data source readback | Terraform client | `data "<IOSXE_DATA_SOURCE_TYPE>" "<LOCAL_NAME>" { <ARGUMENTS> }` | Terraform can read selected existing state |
| 40 | Close `main.tf` | Terraform client | `EOF` | Main resource file exists |
| 41 | Format Terraform files | Terraform client | `terraform fmt` | `.tf` files are normalized |
| 42 | Initialize Terraform project | Terraform client | `terraform init` | Provider is downloaded and project initializes |
| 43 | Validate Terraform syntax | Terraform client | `terraform validate` | Terraform configuration is syntactically valid |
| 44 | Build execution plan | Terraform client | `terraform plan` | Plan shows intended create, update, or destroy actions |
| 45 | Review plan before changing the router | Operator | `Read the plan output` | Planned changes match the intended lab objective |
| 46 | Apply desired state | Terraform client | `terraform apply` | Terraform prompts for approval and applies planned changes |
| 47 | Confirm apply | Terraform client | `yes` | Terraform completes successfully |
| 48 | Inspect Terraform state | Terraform client | `terraform state list` | Managed IOS XE resources appear in state |
| 49 | Show current Terraform state | Terraform client | `terraform show` | State details are displayed |
| 50 | Verify config on IOS XE | IOS XE router | `show running-config | section <FEATURE>` | IOS XE running config reflects Terraform intent |
| 51 | Verify operational effect on IOS XE | IOS XE router | `show <EQUIVALENT_OPERATIONAL_COMMAND>` | Device behavior matches desired state |
| 52 | Run second plan to prove idempotence | Terraform client | `terraform plan` | Plan shows no changes if device matches HCL and state |
| 53 | Save IOS XE config if provider action changed running config only | IOS XE router | `write memory` | Startup-config preserves lab changes |
# Terraform_Cisco_IOS_XE_Desired_State_Automation_Skeleton
# =========================
# IOS XE prerequisite check
# =========================
show ip interface brief
show ip route <CLIENT_IP>
show running-config | include ^username
# =========================
# IOS XE NETCONF default transport
# =========================
configure terminal
 netconf-yang
 ! Optional, only for candidate datastore workflows:
 ! netconf-yang feature candidate-datastore
end
write memory
# Verify from Terraform client:
ssh -p 830 -s <USER>@<DEVICE_IP> netconf
# =========================
# IOS XE RESTCONF optional transport
# Use only if Terraform provider protocol is restconf
# =========================
configure terminal
 ip http authentication local
 ip http secure-server
 restconf
end
write memory
# Verify from Terraform client:
curl -k -u <USER>:<SECRET> \
  -H "Accept: application/yang-data+json" \
  https://<DEVICE_IP>/restconf/data/
# =========================
# Terraform project directory
# =========================
mkdir -p terraform-cisco-ios-xe-router
cd terraform-cisco-ios-xe-router
cat > .gitignore <<'EOF'
.terraform/
*.tfstate
*.tfstate.backup
.terraform.lock.hcl
terraform.tfvars
*.tfvars
EOF
cat > versions.tf <<'EOF'
terraform {
  required_version = ">= 1.0"
  required_providers {
    iosxe = {
      source  = "CiscoDevNet/iosxe"
      version = ">= 0.18.0"
    }
  }
}
EOF
cat > variables.tf <<'EOF'
variable "iosxe_host" {
  type = string
}
variable "iosxe_username" {
  type = string
}
variable "iosxe_password" {
  type      = string
  sensitive = true
}
variable "iosxe_protocol" {
  type    = string
  default = "netconf"
}
EOF
cat > provider.tf <<'EOF'
provider "iosxe" {
  host     = var.iosxe_host
  username = var.iosxe_username
  password = var.iosxe_password
  protocol = var.iosxe_protocol
  insecure = true
}
EOF
cat > terraform.tfvars <<'EOF'
iosxe_host     = "<DEVICE_IP>"
iosxe_username = "<USER>"
iosxe_password = "<SECRET>"
iosxe_protocol = "netconf"
EOF
cat > main.tf <<'EOF'
# Replace this placeholder with the IOS XE provider resource used by the lab.
# Example shape:
#
# resource "<IOSXE_RESOURCE_TYPE>" "<LOCAL_NAME>" {
#   <ARGUMENT_1> = "<VALUE_1>"
#   <ARGUMENT_2> = "<VALUE_2>"
# }
#
# data "<IOSXE_DATA_SOURCE_TYPE>" "<LOCAL_NAME>" {
#   <ARGUMENT_1> = "<VALUE_1>"
# }
EOF
terraform fmt
terraform init
terraform validate
terraform plan
terraform apply
terraform state list
terraform show
terraform plan
# =========================
# IOS XE post-apply verification
# =========================
show running-config | section <FEATURE>
show <EQUIVALENT_OPERATIONAL_COMMAND>
write memory
# Terraform_Cisco_IOS_XE_Desired_State_Automation_Verification_Commands
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Verify IOS XE management IP | IOS XE router | `show ip interface brief` | Management interface is up/up with expected IP |
| 2 | Verify route back to client | IOS XE router | `show ip route <CLIENT_IP>` | Return path exists |
| 3 | Verify local automation user | IOS XE router | `show running-config | include ^username` | Expected user exists |
| 4 | Verify NETCONF if using default transport | IOS XE router | `show running-config | include ^netconf-yang` | `netconf-yang` is configured |
| 5 | Verify NETCONF process health | IOS XE router | `show platform software yang-management process` | YANG management processes are running |
| 6 | Verify NETCONF client access | Terraform client | `ssh -p 830 -s <USER>@<DEVICE_IP> netconf` | Device returns NETCONF hello and capabilities |
| 7 | Verify RESTCONF if selected | IOS XE router | `show running-config | include ^ip http secure-server|^restconf` | HTTPS and RESTCONF are configured |
| 8 | Verify RESTCONF client access | Terraform client | `curl -k -u <USER>:<SECRET> -H "Accept: application/yang-data+json" https://<DEVICE_IP>/restconf/data/` | Device returns RESTCONF response |
| 9 | Verify Terraform binary | Terraform client | `terraform version` | Terraform version is displayed |
| 10 | Verify Terraform files exist | Terraform client | `ls -la` | `versions.tf`, `provider.tf`, `variables.tf`, `main.tf`, and `terraform.tfvars` exist |
| 11 | Verify provider requirement | Terraform client | `cat versions.tf` | `CiscoDevNet/iosxe` provider source is present |
| 12 | Verify provider config | Terraform client | `cat provider.tf` | Provider references host, username, password, and protocol variables |
| 13 | Verify formatting | Terraform client | `terraform fmt -check` | Files are already formatted |
| 14 | Verify initialization | Terraform client | `terraform init` | Provider initializes successfully |
| 15 | Verify syntax and provider schema | Terraform client | `terraform validate` | Configuration is valid |
| 16 | Verify planned changes before apply | Terraform client | `terraform plan` | Output shows intended IOS XE changes only |
| 17 | Verify apply result | Terraform client | `terraform apply` | Apply completes without provider or device errors |
| 18 | Verify Terraform state list | Terraform client | `terraform state list` | Expected IOS XE resources appear |
| 19 | Verify Terraform state details | Terraform client | `terraform show` | State reflects expected resource attributes |
| 20 | Verify IOS XE running config | IOS XE router | `show running-config | section <FEATURE>` | Running config reflects Terraform intent |
| 21 | Verify IOS XE operational state | IOS XE router | `show <EQUIVALENT_OPERATIONAL_COMMAND>` | Operational state matches intended result |
| 22 | Verify idempotence | Terraform client | `terraform plan` | No changes are proposed after successful apply |
| 23 | Verify persistence after save | IOS XE router | `show startup-config | section <FEATURE>` | Startup-config contains intended config |
# Terraform_Cisco_IOS_XE_Desired_State_Automation_Rollback
# =========================
# Preferred rollback: restore desired state in HCL
# =========================
# Edit main.tf back to known-good desired state.
terraform fmt
terraform validate
terraform plan
terraform apply
# Verify on IOS XE:
show running-config | section <FEATURE>
show <EQUIVALENT_OPERATIONAL_COMMAND>
write memory
# =========================
# Remove Terraform-managed resources
# Use only when the lab objective is deletion
# =========================
terraform plan -destroy
terraform destroy
# Verify on IOS XE:
show running-config | section <FEATURE>
show <EQUIVALENT_OPERATIONAL_COMMAND>
# =========================
# Remove a single object from Terraform state only
# Does not remove config from IOS XE
# =========================
terraform state list
terraform state rm <RESOURCE_ADDRESS>
# =========================
# Re-import existing IOS XE config into Terraform state
# Requires resource block to already exist in HCL
# =========================
terraform import <RESOURCE_ADDRESS> <IMPORT_ID>
# =========================
# Remove local Terraform lab files
# This does not touch IOS XE
# =========================
rm -rf .terraform
rm -f .terraform.lock.hcl
rm -f terraform.tfstate
rm -f terraform.tfstate.backup
# =========================
# Disable IOS XE programmable services only if this lab enabled them
# Do not do this if NETCONF, RESTCONF, controllers, telemetry, or APIs still depend on them
# =========================
configure terminal
 no restconf
 no ip http secure-server
 no netconf-yang
end
write memory
# Terraform_Cisco_IOS_XE_Desired_State_Automation_Failure_Checks
| Symptom | Device | Command | Likely Cause | Corrective Action |
|---|---|---|---|---|
| Terraform client cannot reach router | Terraform client | `ping <DEVICE_IP>` | IP path, interface, VLAN, route, or gateway problem | Fix basic management reachability first |
| NETCONF test fails | Terraform client | `ssh -p 830 -s <USER>@<DEVICE_IP> netconf` | `netconf-yang` missing, TCP/830 blocked, auth issue, or YANG process issue | Enable `netconf-yang`, verify auth, and permit TCP/830 |
| RESTCONF test fails | Terraform client | `curl -k -u <USER>:<SECRET> https://<DEVICE_IP>/restconf/data/` | RESTCONF, HTTPS, or HTTP auth missing | Configure `ip http authentication local`, `ip http secure-server`, and `restconf` |
| `terraform init` fails | Terraform client | `terraform init` | Provider source/version issue, internet access issue, or bad required provider block | Fix `versions.tf`, connectivity, or provider version constraint |
| `terraform validate` fails | Terraform client | `terraform validate` | HCL syntax or provider schema error | Fix `.tf` syntax and resource arguments |
| `terraform plan` cannot authenticate | Terraform client | `terraform plan` | Wrong username, wrong password, wrong protocol, or wrong host | Correct `terraform.tfvars` or environment variables |
| `terraform plan` connects to wrong protocol | Terraform client | `cat terraform.tfvars` | Provider protocol mismatch | Use `iosxe_protocol = "netconf"` or `iosxe_protocol = "restconf"` intentionally |
| Plan shows unexpected destroy | Terraform client | `terraform plan` | HCL no longer declares an object currently in state | Stop and restore the missing resource block or review state before apply |
| Apply fails with provider timeout | Terraform client | `terraform apply` | Device slow, protocol blocked, or YANG process unhealthy | Verify NETCONF/RESTCONF reachability and process health |
| Apply succeeds but IOS XE config does not match | IOS XE router | `show running-config | section <FEATURE>` | Wrong resource, wrong provider target, or change was operational only | Recheck HCL resource, target device, and YANG mapping |
| Second plan still shows changes | Terraform client | `terraform plan` | Drift, provider normalization, unsupported argument, or manual device change | Compare HCL, Terraform state, and IOS XE running config |
| Manual CLI change gets reverted | IOS XE router | `show running-config | section <FEATURE>` | Terraform owns that object and restores declared state | Make change in HCL, not directly on CLI |
| Terraform state is missing object | Terraform client | `terraform state list` | Object was never applied, state was removed, or wrong workspace/directory | Use correct directory/workspace or import existing object |
| Existing IOS XE config conflicts with apply | Terraform client | `terraform apply` | Resource already exists on device but not in Terraform state | Import it or change HCL to use a non-conflicting object |
| Credentials are exposed in files | Terraform client | `cat terraform.tfvars` | Lab variables file stores password in plaintext | Keep `terraform.tfvars` out of Git and prefer environment variables for repeat use |
| `.tfstate` is committed accidentally | Terraform client | `git status` | `.gitignore` missing or incomplete | Add state files to `.gitignore` and remove them from Git tracking |
| RESTCONF certificate warning appears | Terraform client | Terraform provider or curl output | IOS XE uses self-signed certificate | Use `insecure = true` for lab testing, proper PKI for production |
| Provider lock/version causes inconsistent lab behavior | Terraform client | `cat .terraform.lock.hcl` | Provider version changed or lock file was removed | Pin provider version and re-run `terraform init` carefully |
| Destroy removes more than intended | Terraform client | `terraform plan -destroy` | Entire state is targeted for removal | Use resource-specific HCL rollback or targeted state handling instead of broad destroy |
##### Source_Basis
# Terraform_Cisco_IOS_XE_Desired_State_Automation_Mental_Model
# Terraform_Cisco_IOS_XE_Desired_State_Automation_Configuration_Checklist
# Terraform_Cisco_IOS_XE_Desired_State_Automation_Skeleton
# Terraform_Cisco_IOS_XE_Desired_State_Automation_Verification_Commands
# Terraform_Cisco_IOS_XE_Desired_State_Automation_Rollback
# Terraform_Cisco_IOS_XE_Desired_State_Automation_Failure_Checks