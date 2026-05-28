
QoS_MQC_Class_Map_Policy_Map_Service_Policy.md

QoS_MQC_Class_Map_Policy_Map_Service_Policy


# QoS_MQC_Class_Map_Policy_Map_Service_Policy_Mental_Model
| Concept | Operational Meaning |
|---|---|
| MQC | Modular QoS CLI. The reusable Cisco QoS framework built from class-maps, policy-maps, and service-policy attachment |
| Class-map | Defines what traffic is being matched. This is the classification stage |
| `match-any` | Logical OR. Traffic only needs to match one statement in the class-map |
| `match-all` | Logical AND. Traffic must match all statements in the class-map |
| ACL-based classification | ACL `permit` lines identify traffic for QoS classification. ACL `deny` lines should not be treated as QoS classification matches |
| Policy-map | Defines what happens to matched traffic. This is where marking, policing, shaping, queuing, WRED, or priority behavior is configured |
| Class inside policy-map | Binds a class-map to a QoS action |
| `class-default` | Catches traffic that does not match user-defined classes |
| Service-policy | Applies the policy-map to an interface in the input or output direction |
| Direction matters | Ingress policies classify and mark traffic before forwarding. Egress policies commonly queue, shape, police, or drop traffic leaving an interface |
| MQC has no effect until attached | A class-map and policy-map can exist in the running config but do nothing until `service-policy input` or `service-policy output` is applied |
# QoS_MQC_Class_Map_Policy_Map_Service_Policy_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm baseline forwarding before applying QoS | Router/Switch | `show ip interface brief` | Interfaces needed for the lab are up/up |
| 2 | Identify the interface where QoS must be enforced | Router/Switch | `show cdp neighbors` / `show lldp neighbors` / `show interfaces status` | Correct ingress or egress interface is known |
| 3 | Confirm whether policy should be ingress or egress | Router/Switch | `show interfaces <interface>` | You know whether traffic must be treated entering or leaving the device |
| 4 | Create any ACLs needed for classification | Router/Switch | `ip access-list extended <ACL_NAME>` | Named ACL exists for matching specific traffic |
| 5 | Add permit statements for traffic to classify | Router/Switch | `permit <protocol> <source> <destination> <port-condition>` | Target traffic is permitted by the ACL for QoS classification |
| 6 | Enter global configuration mode | Router/Switch | `configure terminal` | Device enters global config mode |
| 7 | Create a class-map for the first traffic class | Router/Switch | `class-map match-any <CLASS_NAME>` | Device enters class-map config mode |
| 8 | Match traffic by DSCP if already marked | Router/Switch | `match dscp <dscp-value>` | Class-map matches packets carrying the selected DSCP value |
| 9 | Match traffic by ACL if classification depends on source, destination, protocol, or port | Router/Switch | `match access-group name <ACL_NAME>` | Class-map uses the named ACL as a match condition |
| 10 | Match traffic by protocol if NBAR classification is required | Router/Switch | `match protocol <protocol-name>` | Class-map matches application/protocol traffic supported by NBAR |
| 11 | Exit class-map configuration mode | Router/Switch | `exit` | Device returns to global config mode |
| 12 | Repeat class-map creation for each traffic class | Router/Switch | `class-map match-any <NEXT_CLASS_NAME>` | Each traffic category has a separate class-map |
| 13 | Create the policy-map | Router/Switch | `policy-map <POLICY_NAME>` | Device enters policy-map config mode |
| 14 | Enter the first policy class | Router/Switch | `class <CLASS_NAME>` | Device enters policy-map class config mode |
| 15 | Apply the QoS action for that class | Router/Switch | `set dscp <value>` / `bandwidth <value>` / `priority <value>` / `police <rate>` / `shape average <rate>` / `random-detect` | Matched traffic receives the intended QoS treatment |
| 16 | Return to policy-map mode | Router/Switch | `exit` | Device returns to policy-map config mode |
| 17 | Configure additional classes in the policy-map | Router/Switch | `class <NEXT_CLASS_NAME>` | Each class receives the required QoS action |
| 18 | Configure default-class behavior if needed | Router/Switch | `class class-default` | Unmatched traffic has explicit default treatment |
| 19 | Exit policy-map configuration mode | Router/Switch | `exit` | Device returns to global config mode |
| 20 | Enter the target interface | Router/Switch | `interface <interface-id>` | Device enters interface config mode |
| 21 | Attach the policy inbound if traffic must be treated on ingress | Router/Switch | `service-policy input <POLICY_NAME>` | Policy-map is active on inbound traffic |
| 22 | Attach the policy outbound if traffic must be treated on egress | Router/Switch | `service-policy output <POLICY_NAME>` | Policy-map is active on outbound traffic |
| 23 | Exit interface configuration mode | Router/Switch | `end` | Device returns to privileged EXEC mode |
| 24 | Verify policy definition | Router/Switch | `show policy-map <POLICY_NAME>` | Policy-map shows expected classes and QoS actions |
| 25 | Verify service-policy attachment | Router/Switch | `show policy-map interface <interface-id>` | Interface shows the applied policy and class counters |
| 26 | Generate test traffic | Hosts/Routers | `ping`, `iperf`, `telnet`, `curl`, RTP/SIP test flow, or lab-specific traffic generator | Traffic crosses the policy interface |
| 27 | Recheck policy counters | Router/Switch | `show policy-map interface <interface-id>` | Matching class counters increment |
| 28 | Save the configuration | Router/Switch | `copy running-config startup-config` | MQC configuration survives reload |
# QoS_MQC_Class_Map_Policy_Map_Service_Policy_Skeleton
configure terminal
ip access-list extended <ACL_NAME>
 permit <protocol> <source> <wildcard> <destination> <wildcard> <optional-port-match>
 exit
class-map match-any <CLASS_NAME_1>
 match dscp <DSCP_VALUE>
 match access-group name <ACL_NAME>
 exit
class-map match-any <CLASS_NAME_2>
 match protocol <PROTOCOL_NAME>
 exit
policy-map <POLICY_NAME>
 class <CLASS_NAME_1>
  <QOS_ACTION>
 exit
 class <CLASS_NAME_2>
  <QOS_ACTION>
 exit
 class class-default
  <DEFAULT_QOS_ACTION>
 exit
 exit
interface <INTERFACE_ID>
 service-policy <input|output> <POLICY_NAME>
 end
show policy-map <POLICY_NAME>
show policy-map interface <INTERFACE_ID>
copy running-config startup-config
# QoS_MQC_Class_Map_Policy_Map_Service_Policy_Example_Skeleton
configure terminal
ip access-list extended RTP_AUDIO
 permit udp any any range 16384 32767
 exit
class-map match-any VOICE_RTP
 match dscp ef
 match access-group name RTP_AUDIO
 exit
class-map match-any CALL_CONTROL
 match dscp cs3 af31 af32 af33
 exit
policy-map EDGE_QOS_POLICY
 class VOICE_RTP
  set dscp ef
 exit
 class CALL_CONTROL
  set dscp cs3
 exit
 class class-default
  set dscp default
 exit
 exit
interface GigabitEthernet1
 service-policy output EDGE_QOS_POLICY
 end
show policy-map EDGE_QOS_POLICY
show policy-map interface GigabitEthernet1
copy running-config startup-config
# QoS_MQC_Class_Map_Policy_Map_Service_Policy_Verification_Commands
| Command | Purpose | Good Output |
|---|---|---|
| `show running-config class-map` | Confirms class-map definitions | Class-maps exist with expected match statements |
| `show running-config policy-map` | Confirms policy-map actions | Policy-map contains expected classes and actions |
| `show running-config interface <interface-id>` | Confirms service-policy attachment | Interface shows `service-policy input <POLICY_NAME>` or `service-policy output <POLICY_NAME>` |
| `show policy-map <POLICY_NAME>` | Verifies policy-map structure | Classes and QoS actions appear correctly |
| `show policy-map interface <interface-id>` | Verifies runtime policy behavior | Class counters increment when matching traffic crosses interface |
| `show access-lists <ACL_NAME>` | Verifies ACL match counters | ACL permit counters increment for matched traffic |
| `show ip nbar protocol-discovery interface <interface-id>` | Verifies protocol visibility when NBAR is used | Expected application/protocol appears in NBAR output |
| `clear counters <interface-id>` | Clears interface counters before testing | Counters reset for clean validation |
| `clear access-list counters <ACL_NAME>` | Clears ACL counters before testing | ACL counters reset for clean classification testing |
# QoS_MQC_Class_Map_Policy_Map_Service_Policy_Rollback
configure terminal
interface <INTERFACE_ID>
 no service-policy input <POLICY_NAME>
 no service-policy output <POLICY_NAME>
 exit
no policy-map <POLICY_NAME>
no class-map <CLASS_NAME_1>
no class-map <CLASS_NAME_2>
no ip access-list extended <ACL_NAME>
end
show policy-map interface <INTERFACE_ID>
show running-config | section class-map
show running-config | section policy-map
show running-config interface <INTERFACE_ID>
# QoS_MQC_Class_Map_Policy_Map_Service_Policy_Failure_Checks
| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| Policy exists but counters stay at zero | Policy-map is not attached to an interface | `show running-config interface <interface-id>` | Add `service-policy input <POLICY_NAME>` or `service-policy output <POLICY_NAME>` |
| Policy is attached but wrong traffic is counted | Class-map match condition is too broad | `show policy-map interface <interface-id>` and `show access-lists` | Tighten ACL, DSCP, protocol, or class-map logic |
| Expected class never matches | `match-all` requires every condition to be true | `show running-config class-map` | Use `match-any` when OR behavior is intended |
| ACL exists but class does not match traffic | ACL lacks the correct `permit` entry | `show access-lists <ACL_NAME>` | Add correct permit statement for the traffic being classified |
| Traffic falls into `class-default` | User-defined class-map does not match | `show policy-map interface <interface-id>` | Fix class-map match statement or traffic marking |
| QoS policy has no forwarding effect | QoS action is only marking, not queuing, shaping, policing, or dropping | `show running-config policy-map` | Add the correct action for the lab objective |
| Egress queueing behavior does not appear | Policy attached in the wrong direction | `show running-config interface <interface-id>` | Move policy to `service-policy output` if queueing or shaping is required |
| Ingress classification does not affect downstream behavior | Traffic is not marked before leaving the device | `show policy-map interface <interface-id>` | Apply ingress marking policy before egress treatment policy |
| Command rejected on interface | Platform or interface type does not support that QoS action or direction | CLI error and `show running-config interface <interface-id>` | Move policy to supported interface/direction or simplify action |
| Old counters confuse validation | Counters were not cleared before testing | `show policy-map interface <interface-id>` | Clear counters or record baseline before generating test traffic |
##### Source_Basis
# QoS_MQC_Class_Map_Policy_Map_Service_Policy_Mental_Model
# QoS_MQC_Class_Map_Policy_Map_Service_Policy_Configuration_Checklist
# QoS_MQC_Class_Map_Policy_Map_Service_Policy_Skeleton
# QoS_MQC_Class_Map_Policy_Map_Service_Policy_Example_Skeleton
# QoS_MQC_Class_Map_Policy_Map_Service_Policy_Verification_Commands
# QoS_MQC_Class_Map_Policy_Map_Service_Policy_Rollback
# QoS_MQC_Class_Map_Policy_Map_Service_Policy_Failure_Checks