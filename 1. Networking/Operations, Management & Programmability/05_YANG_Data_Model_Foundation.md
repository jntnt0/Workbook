

YANG_Data_Model_Foundation.md

YANG_Data_Model_Foundation

# Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| `_KB_INDEX.md` | NETCONF / RESTCONF / YANG / telemetry / gRPC | Points IOS XE model-driven programmability topics to `iosxe_combined_pdfs_.md`, `Ios-xs_combined_epubs.md`, and `CiscoPress_combined_part2.md` |
| `CCNP and CCIE Enterprise Core ENCOR 350-401 Official Cert.md` | Ch28, Data Models and Supporting Protocols | Defines YANG as the model language used by NETCONF and RESTCONF |
| `CCNP and CCIE Enterprise Core ENCOR 350-401 Official Cert.md` | Ch28, YANG Data Models | Supports the mental model of YANG as a hierarchical tree made of modules, containers, lists, leaves, config data, and state data |
| `CCNP and CCIE Enterprise Core ENCOR 350-401 Official Cert.md` | Ch28, NETCONF and RESTCONF | Supports the relationship between YANG models, NETCONF XML RPCs, RESTCONF HTTP methods, and datastores |
| `CiscoPress_combined_part2.md` | Ch12, Network Programmability | Supports YANG model use with RESTCONF and IOS XE programmable data access |
| `CiscoPress_combined_part2.md` | Ch13, Model-Driven Telemetry | Supports operational YANG data, `config false`, telemetry subscriptions, NETCONF requirement, and `show platform software yang-management process` verification |
| `Ios-xs_combined_epubs.md` | Model-Driven Telemetry with YANG Suite | Supports YANG Suite use, device profiles, YANG repositories, module sets, XPath selection, and telemetry validation |
| `iosxe_combined_pdfs_.md` | IOS XE programmability overview | Supports IOS XE use of NETCONF, RESTCONF, IETF YANG Push, OpenConfig, IETF YANG, and Cisco native YANG models |
# YANG_Data_Model_Foundation_Mental_Model
| Concept | Operational Meaning |
|---|---|
| YANG | YANG is the schema language. It defines what data exists, what type it is, whether it can be changed, and where it lives in the model tree |
| YANG is not the transport | NETCONF, RESTCONF, gNMI, and telemetry consume YANG-modeled data. YANG itself is not a session protocol |
| Module | A YANG module is a named model file with a namespace and revision |
| Tree | YANG data is navigated as a hierarchy of containers, lists, keys, and leaves |
| Container | Logical grouping of related data |
| List | Repeating table-like structure, usually with a key such as interface name |
| Leaf | Individual value such as interface description, enabled state, MTU, or operational status |
| Key | The unique identifier for a list entry, such as `name` in an interface list |
| Namespace | The unique XML/RESTCONF identity for a module, preventing name collisions between models |
| Revision | The model version. IOS XE release and YANG model revision must be treated as linked |
| Config data | Read-write intended configuration, such as interface description or enabled state |
| State data | Read-only operational data, such as current interface status or counters |
| `config false` | Marks operational read-only data. You can read it, but you cannot configure it |
| Datastore | Logical storage target used by model-driven protocols, such as running configuration or operational state |
| XPath | Path expression used to target a specific YANG subtree or telemetry stream |
| Cisco native models | Cisco IOS XE specific models, commonly used when the CLI structure maps closely to native IOS XE features |
| IETF models | Standards-based models useful for cross-vendor features such as interfaces |
| OpenConfig models | Vendor-neutral operational/configuration models commonly used in automation and telemetry ecosystems |
| Encoding | NETCONF commonly uses XML. RESTCONF can use JSON or XML. gRPC/gNMI commonly uses compact binary encoding |
| Correct workflow | Pick the operational goal, identify the model, identify the path, confirm read-write vs read-only, then choose NETCONF, RESTCONF, or telemetry |
| Common mistake | Treating a RESTCONF URL, NETCONF RPC, or telemetry XPath as the model itself. Those are access methods into the model |
| Related labs | `netconf-final`, `ios-xe-netconf-restconf-final`, `ios-xe-rest-api-final`, `restconf-r1-bridge-final`, `grpc-final` |
# YANG_Data_Model_Foundation_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm management IP reachability before model-driven testing | IOS XE | `show ip interface brief` | Management interface or SVI has the expected IP and is up/up |
| 2 | Confirm the device has a route back to the automation client | IOS XE | `show ip route <CLIENT_IP>` | Route exists toward the client |
| 3 | Confirm the client can reach the IOS XE device | Client | `ping <DEVICE_IP>` | Ping succeeds |
| 4 | Confirm local credentials exist for model-driven access | IOS XE | `show running-config | include ^username` | Expected local user exists |
| 5 | Enter global configuration mode if a protocol consumer must be enabled | IOS XE | `configure terminal` | Device enters global config mode |
| 6 | Enable NETCONF only when the foundation lab uses NETCONF or YANG Suite over NETCONF | IOS XE | `netconf-yang` | NETCONF/YANG management process is enabled |
| 7 | Enable HTTP authentication for RESTCONF only when the lab uses RESTCONF | IOS XE | `ip http authentication local` | HTTP server uses the local username database |
| 8 | Enable HTTPS for RESTCONF only when the lab uses RESTCONF | IOS XE | `ip http secure-server` | HTTPS service is available for RESTCONF |
| 9 | Enable RESTCONF only when the lab uses RESTCONF | IOS XE | `restconf` | RESTCONF service is enabled |
| 10 | Exit configuration mode | IOS XE | `end` | Device returns to privileged EXEC mode |
| 11 | Save protocol foundation if any service was enabled | IOS XE | `write memory` | Startup-config keeps the model-driven access baseline |
| 12 | Verify YANG management processes | IOS XE | `show platform software yang-management process` | YANG management processes are present and running |
| 13 | Identify the operational goal before choosing a model | Operator | `Goal: <READ_STATE_OR_CHANGE_CONFIG>` | Goal is classified as read-only state, read-write config, action, or notification |
| 14 | Identify the candidate model family | Operator | `Model family: <Cisco native | IETF | OpenConfig>` | Correct model family is selected for the lab objective |
| 15 | Identify the module name | YANG tool or repository | `<MODULE_NAME>` | Example format resembles `ietf-interfaces` or `Cisco-IOS-XE-native` |
| 16 | Identify the module namespace | YANG tool or repository | `<MODULE_NAMESPACE>` | Namespace is recorded for NETCONF XML or RESTCONF path construction |
| 17 | Identify the module revision | YANG tool or repository | `<MODULE_REVISION>` | Revision matches or is compatible with the IOS XE release |
| 18 | Identify the target container, list, or leaf | YANG tool or repository | `<MODULE>:<CONTAINER>/<LIST>/<LEAF>` | Exact model path is known before the API call is built |
| 19 | Confirm whether the target path is config or state | YANG tool or repository | `Check access or config false` | Operator knows whether the path is read-write or read-only |
| 20 | Record the XPath for telemetry or subtree targeting | YANG tool or repository | `<XPATH_FILTER>` | XPath points to the intended YANG subtree |
| 21 | Choose the access protocol | Operator | `Protocol: <NETCONF | RESTCONF | gNMI/gRPC telemetry>` | Protocol matches the lab objective |
| 22 | Choose the encoding | Operator | `Encoding: <XML | JSON | KVGPB | GPB>` | Encoding matches protocol and collector/client requirements |
| 23 | Validate a read-only lookup before attempting config changes | Client | `GET or get operation against <YANG_PATH>` | Client receives structured data from the device |
| 24 | Compare returned model data to CLI truth | IOS XE | `show <EQUIVALENT_CLI_COMMAND>` | CLI and model output describe the same device state |
| 25 | Attempt write only against confirmed read-write config path | Client | `edit-config, PUT, PATCH, or POST against <CONFIG_PATH>` | Change is accepted only when the target path is configurable |
| 26 | Verify the config effect from CLI | IOS XE | `show running-config | section <FEATURE>` | Device config reflects the intended model-driven change |
| 27 | Verify the operational effect from CLI | IOS XE | `show <OPERATIONAL_COMMAND>` | Device state reflects the intended outcome |
| 28 | Document the model path used by the lab | Operator | `Model path: <MODULE>:<PATH>` | Future NETCONF, RESTCONF, and telemetry notes can reuse the same path |
# YANG_Data_Model_Foundation_Skeleton
! =========================
! IOS XE baseline checks
! =========================
show ip interface brief
show ip route <CLIENT_IP>
show running-config | include ^username
! =========================
! Optional NETCONF foundation
! Use only when the lab uses NETCONF or YANG Suite over NETCONF
! =========================
configure terminal
 netconf-yang
end
write memory
! =========================
! Optional RESTCONF foundation
! Use only when the lab uses RESTCONF
! =========================
configure terminal
 ip http authentication local
 ip http secure-server
 restconf
end
write memory
! =========================
! YANG process verification
! =========================
show platform software yang-management process
! =========================
! YANG discovery worksheet
! =========================
Operational goal: <READ_STATE | CHANGE_CONFIG | ACTION | NOTIFICATION>
Model family: <Cisco native | IETF | OpenConfig>
Module name: <MODULE_NAME>
Namespace: <MODULE_NAMESPACE>
Revision: <MODULE_REVISION>
Target path: <MODULE>:<CONTAINER>/<LIST>/<LEAF>
Access type: <read-only | read-write>
Protocol: <NETCONF | RESTCONF | gNMI/gRPC telemetry>
Encoding: <XML | JSON | KVGPB | GPB>
CLI comparison command: show <EQUIVALENT_CLI_COMMAND>
! =========================
! RESTCONF read test pattern
! =========================
curl -k -u <USER>:<SECRET> \
  -H "Accept: application/yang-data+json" \
  https://<DEVICE_IP>/restconf/data/<MODULE>:<PATH>
! =========================
! NETCONF raw capability test pattern
! =========================
ssh -p 830 -s <USER>@<DEVICE_IP> netconf
! =========================
! Model file tree view pattern, if local YANG files exist
! =========================
pyang -f tree <MODULE>.yang
# YANG_Data_Model_Foundation_Verification_Commands
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Verify management IP state | IOS XE | `show ip interface brief` | Management interface is up/up with expected IP |
| 2 | Verify route back to automation client | IOS XE | `show ip route <CLIENT_IP>` | Route exists toward the client |
| 3 | Verify local user exists | IOS XE | `show running-config | include ^username` | Expected automation user exists |
| 4 | Verify NETCONF config if used | IOS XE | `show running-config | include netconf-yang` | `netconf-yang` is present |
| 5 | Verify RESTCONF config if used | IOS XE | `show running-config | include restconf|ip http` | RESTCONF and required HTTP settings are present |
| 6 | Verify YANG management processes | IOS XE | `show platform software yang-management process` | Required YANG management processes are running |
| 7 | Verify RESTCONF HTTPS reachability if used | Client | `curl -k -u <USER>:<SECRET> https://<DEVICE_IP>/restconf/data/` | RESTCONF root responds instead of timing out |
| 8 | Verify RESTCONF JSON read against a chosen model path | Client | `curl -k -u <USER>:<SECRET> -H "Accept: application/yang-data+json" https://<DEVICE_IP>/restconf/data/<MODULE>:<PATH>` | Client receives JSON modeled data |
| 9 | Verify RESTCONF XML read if XML is being tested | Client | `curl -k -u <USER>:<SECRET> -H "Accept: application/yang-data+xml" https://<DEVICE_IP>/restconf/data/<MODULE>:<PATH>` | Client receives XML modeled data |
| 10 | Verify NETCONF TCP/830 reachability if used | Client | `ssh -p 830 -s <USER>@<DEVICE_IP> netconf` | NETCONF subsystem opens and capabilities are returned |
| 11 | Verify local YANG file can be parsed if model files exist | Client | `pyang -f tree <MODULE>.yang` | Model tree renders without parse errors |
| 12 | Verify target path is read-only or read-write before changing it | YANG tool or model file | `Check access, operations, or config false` | Operator knows whether the path supports edit operations |
| 13 | Verify read-only model output against CLI | IOS XE | `show <EQUIVALENT_CLI_COMMAND>` | CLI state matches the modeled output |
| 14 | Verify config model output against running config | IOS XE | `show running-config | section <FEATURE>` | Running config matches the modeled config value |
| 15 | Verify telemetry path if the model is used for streaming | IOS XE | `show telemetry ietf subscription all` | Subscription exists when telemetry is configured |
| 16 | Verify telemetry connection if gRPC is the consumer | IOS XE | `show telemetry internal connection` | Collector connection is present when telemetry is active |
# YANG_Data_Model_Foundation_Rollback
! =========================
! Remove RESTCONF foundation only if this note enabled it
! =========================
configure terminal
 no restconf
 no ip http secure-server
 no ip http authentication local
end
write memory
! =========================
! Remove NETCONF foundation only if this note enabled it
! =========================
configure terminal
 no netconf-yang
end
write memory
! =========================
! Do not delete local usernames here unless this lab created them
! Local-auth rollback belongs in the management-plane note
! =========================
! =========================
! Clear local worksheet values
! =========================
Operational goal: <cleared>
Model family: <cleared>
Module name: <cleared>
Namespace: <cleared>
Revision: <cleared>
Target path: <cleared>
Access type: <cleared>
Protocol: <cleared>
Encoding: <cleared>
CLI comparison command: <cleared>
# YANG_Data_Model_Foundation_Failure_Checks
| Symptom | Device | Command | Likely Cause | Corrective Action |
|---|---|---|---|---|
| Client cannot reach IOS XE device | Client | `ping <DEVICE_IP>` | IP reachability, interface, VLAN, gateway, or routing problem | Fix management reachability before troubleshooting YANG |
| NETCONF does not answer on TCP/830 | IOS XE | `show running-config | include netconf-yang` | NETCONF/YANG not enabled | Configure `netconf-yang` |
| RESTCONF URL times out | IOS XE | `show running-config | include restconf|ip http` | RESTCONF or HTTPS server is not enabled | Configure `ip http authentication local`, `ip http secure-server`, and `restconf` |
| Authentication fails | IOS XE | `show running-config | include ^username` | Missing local user, wrong password, or wrong HTTP/NETCONF auth method | Correct local user and authentication method |
| YANG process appears unhealthy | IOS XE | `show platform software yang-management process` | YANG management process issue | Restart or re-enable the affected model-driven service during lab maintenance window |
| RESTCONF path returns not found | Client | `curl -k -u <USER>:<SECRET> https://<DEVICE_IP>/restconf/data/<MODULE>:<PATH>` | Wrong module name, namespace, container, list key, or path syntax | Recheck the YANG tree and construct the path again |
| NETCONF RPC returns unknown element | Client | NETCONF RPC reply | Wrong namespace or unsupported module path | Confirm module namespace and IOS XE model support |
| Write operation fails | Client | RESTCONF or NETCONF error response | Target path is read-only or marked `config false` | Use a read-write config path, or treat the value as operational state only |
| Returned data does not match expected CLI state | IOS XE | `show <EQUIVALENT_CLI_COMMAND>` | Wrong model family or wrong subtree selected | Compare Cisco native, IETF, and OpenConfig paths and choose the correct one |
| Model parses locally but fails on device | Client | `pyang -f tree <MODULE>.yang` | Local model revision does not match IOS XE supported revision | Use the model revision matching the device software release |
| Telemetry subscription accepts XPath but sends no useful data | IOS XE | `show telemetry ietf subscription all` | XPath targets wrong subtree or no matching operational data exists | Rebuild XPath from the correct YANG model path |
| Telemetry collector receives no data | IOS XE | `show telemetry internal connection` | Collector IP, port, protocol, encoding, or reachability is wrong | Fix receiver settings and transport reachability |
| RESTCONF JSON/XML formatting fails | Client | RESTCONF response body | Wrong `Content-Type` or `Accept` header | Use `application/yang-data+json` or `application/yang-data+xml` as required |
| Operator confuses API path with model path | Operator | Model worksheet | RESTCONF URL was documented but module, namespace, and leaf path were not | Record module, namespace, revision, path, access type, and protocol separately |
##### Source_Basis
# YANG_Data_Model_Foundation_Mental_Model
# YANG_Data_Model_Foundation_Configuration_Checklist
# YANG_Data_Model_Foundation_Skeleton
# YANG_Data_Model_Foundation_Verification_Commands
# YANG_Data_Model_Foundation_Rollback
# YANG_Data_Model_Foundation_Failure_Checks