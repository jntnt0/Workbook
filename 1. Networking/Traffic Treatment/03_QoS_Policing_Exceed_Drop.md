
QoS_Policing_Exceed_Drop.md

QoS_Policing_Exceed_Drop

# Source_Basis
# QoS_Policing_Exceed_Drop_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Policing | Enforces a traffic rate using a token bucket |
| Exceed drop | Traffic above the configured policing rate is dropped immediately |
| Not shaping | Policing does not buffer excess traffic and send it later |
| Abrupt enforcement | Exceeded packets are discarded instead of delayed |
| CIR | Committed Information Rate. The allowed traffic rate |
| BC | Committed Burst. Amount of burst traffic allowed before packets are considered exceeded |
| Conform action | Action applied to traffic within the configured rate |
| Exceed action | Action applied to traffic above the configured rate |
| Single-rate two-color policer | Traffic is treated as conforming or exceeding |
| `conform-action transmit` | Send traffic that is within the rate |
| `exceed-action drop` | Drop traffic that exceeds the rate |
| MQC dependency | Policing still depends on class-map classification, policy-map action, and service-policy attachment |
| Direction matters | Policing can be applied where rate enforcement is needed, commonly ingress at an edge or outbound toward a constrained link depending on platform and design |
# QoS_Policing_Exceed_Drop_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm target interfaces are operational | Router/Switch | `show ip interface brief` | Interfaces needed for the lab are up/up |
| 2 | Identify where rate enforcement must happen | Router/Switch | `show cdp neighbors` / `show lldp neighbors` / `show interfaces status` | Correct policy attachment interface is known |
| 3 | Confirm interface bandwidth and traffic direction | Router/Switch | `show interfaces <interface-id>` | Interface bandwidth, line protocol, and counters are visible |
| 4 | Check for existing QoS policy attachment | Router/Switch | `show policy-map interface <interface-id>` | Existing input or output service-policy state is known |
| 5 | Create an ACL if traffic must be matched by protocol, port, source, or destination | Router/Switch | `ip access-list extended <ACL_NAME>` | Named ACL exists |
| 6 | Add permit entries for traffic to police | Router/Switch | `permit <protocol> <source> <wildcard> <destination> <wildcard> <optional-port>` | Target traffic is identified for QoS classification |
| 7 | Create a class-map for the policed traffic | Router/Switch | `class-map match-any <CLASS_NAME>` | Device enters class-map configuration mode |
| 8 | Match traffic using the ACL | Router/Switch | `match access-group name <ACL_NAME>` | Class-map matches ACL-permitted traffic |
| 9 | Match traffic using DSCP if traffic is already marked | Router/Switch | `match dscp <dscp-value>` | Class-map matches packets with the selected DSCP value |
| 10 | Exit class-map mode | Router/Switch | `exit` | Device returns to global configuration mode |
| 11 | Create the policing policy-map | Router/Switch | `policy-map <POLICY_NAME>` | Device enters policy-map configuration mode |
| 12 | Enter the class that will be policed | Router/Switch | `class <CLASS_NAME>` | Device enters policy-map class mode |
| 13 | Configure single-rate two-color policing | Router/Switch | `police cir <bps> bc <bytes>` | Traffic is measured against the committed rate and burst size |
| 14 | Allow conforming traffic through | Router/Switch | `conform-action transmit` | Traffic within the configured rate is forwarded |
| 15 | Drop exceeding traffic | Router/Switch | `exceed-action drop` | Traffic above the configured rate is discarded |
| 16 | Exit the policy class | Router/Switch | `exit` | Device returns to policy-map mode |
| 17 | Configure default-class behavior if needed | Router/Switch | `class class-default` | Unmatched traffic can be explicitly treated |
| 18 | Leave default traffic unpoliced or apply separate treatment | Router/Switch | `exit` | Default traffic behavior is intentionally defined |
| 19 | Exit policy-map mode | Router/Switch | `exit` | Device returns to global configuration mode |
| 20 | Enter the target interface | Router/Switch | `interface <interface-id>` | Device enters interface configuration mode |
| 21 | Attach the policing policy inbound if enforcing traffic entering the device | Router/Switch | `service-policy input <POLICY_NAME>` | Policer is active on ingress traffic |
| 22 | Attach the policing policy outbound if enforcing traffic leaving the device | Router/Switch | `service-policy output <POLICY_NAME>` | Policer is active on egress traffic |
| 23 | Return to privileged EXEC mode | Router/Switch | `end` | Device exits configuration mode |
| 24 | Verify policy definition | Router/Switch | `show policy-map <POLICY_NAME>` | Policy shows the class and police action |
| 25 | Verify interface attachment and policer counters | Router/Switch | `show policy-map interface <interface-id>` | Policy appears under the interface with conformed and exceeded counters |
| 26 | Generate traffic below the policing rate | Hosts/Routers | `ping`, `iperf`, `curl`, `telnet`, or lab traffic generator` | Traffic forwards and conform counters increase |
| 27 | Generate traffic above the policing rate | Hosts/Routers | `iperf` or lab traffic generator above `<bps>` | Exceed counters increase and traffic loss appears |
| 28 | Confirm exceeded traffic is dropped | Router/Switch | `show policy-map interface <interface-id>` | Output shows exceeded bytes or packets with action `drop` |
| 29 | Confirm intended traffic is not landing in `class-default` | Router/Switch | `show policy-map interface <interface-id>` | Target traffic increments the configured policing class |
| 30 | Save the configuration | Router/Switch | `copy running-config startup-config` | Policing configuration survives reload |
# QoS_Policing_Exceed_Drop_Skeleton
configure terminal
ip access-list extended <ACL_NAME>
 permit <protocol> <source> <wildcard> <destination> <wildcard> <optional-port>
 exit
class-map match-any <CLASS_NAME>
 match access-group name <ACL_NAME>
 exit
policy-map <POLICY_NAME>
 class <CLASS_NAME>
  police cir <CIR_BPS> bc <BURST_BYTES>
   conform-action transmit
   exceed-action drop
 exit
 class class-default
 exit
 exit
interface <INTERFACE_ID>
 service-policy <input|output> <POLICY_NAME>
 end
show policy-map <POLICY_NAME>
show policy-map interface <INTERFACE_ID>
copy running-config startup-config
# QoS_Policing_Exceed_Drop_Example_Skeleton
configure terminal
ip access-list extended POLICED_WEB
 permit tcp any any eq 80
 permit tcp any any eq 443
 exit
class-map match-any POLICED_WEB
 match access-group name POLICED_WEB
 exit
policy-map WEB_POLICE_DROP
 class POLICED_WEB
  police cir 1000000 bc 31250
   conform-action transmit
   exceed-action drop
 exit
 class class-default
 exit
 exit
interface GigabitEthernet1
 service-policy input WEB_POLICE_DROP
 end
show policy-map WEB_POLICE_DROP
show policy-map interface GigabitEthernet1
copy running-config startup-config
# QoS_Policing_Exceed_Drop_DSCP_Example
configure terminal
class-map match-any BULK_DATA
 match dscp af11 af12 af13
 exit
policy-map BULK_DATA_POLICE_DROP
 class BULK_DATA
  police cir 2000000 bc 62500
   conform-action transmit
   exceed-action drop
 exit
 class class-default
 exit
 exit
interface GigabitEthernet1
 service-policy input BULK_DATA_POLICE_DROP
 end
show policy-map BULK_DATA_POLICE_DROP
show policy-map interface GigabitEthernet1
# QoS_Policing_Exceed_Drop_Verification_Commands
| Command | Purpose | Good Output |
|---|---|---|
| `show running-config class-map` | Confirms traffic classification | Class-map contains expected ACL, DSCP, or protocol match |
| `show running-config policy-map` | Confirms policing action | Policy class shows `police cir`, `conform-action transmit`, and `exceed-action drop` |
| `show running-config interface <interface-id>` | Confirms service-policy attachment | Interface shows `service-policy input <POLICY_NAME>` or `service-policy output <POLICY_NAME>` |
| `show policy-map <POLICY_NAME>` | Verifies policy structure | Policy-map shows expected class and policer configuration |
| `show policy-map interface <interface-id>` | Verifies runtime policing behavior | Conformed counters increase for allowed traffic and exceeded counters increase for dropped traffic |
| `show access-lists <ACL_NAME>` | Verifies ACL-based classification | ACL permit counters increment when test traffic is generated |
| `show interfaces <interface-id>` | Checks interface drops and traffic rate | Interface counters can be compared against policy-map counters |
| `clear counters <interface-id>` | Clears interface counters before testing | Interface counters reset for cleaner validation |
| `clear access-list counters <ACL_NAME>` | Clears ACL counters before testing | ACL counters reset for cleaner classification validation |
# QoS_Policing_Exceed_Drop_Rollback
configure terminal
interface <INTERFACE_ID>
 no service-policy input <POLICY_NAME>
 no service-policy output <POLICY_NAME>
 exit
no policy-map <POLICY_NAME>
no class-map <CLASS_NAME>
no ip access-list extended <ACL_NAME>
end
show policy-map interface <INTERFACE_ID>
show running-config | section policy-map
show running-config | section class-map
show running-config interface <INTERFACE_ID>
# QoS_Policing_Exceed_Drop_Failure_Checks
| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| Policy exists but does nothing | Policy-map is not attached to the interface | `show running-config interface <interface-id>` | Add `service-policy input <POLICY_NAME>` or `service-policy output <POLICY_NAME>` |
| No traffic matches the policing class | Class-map match condition is wrong | `show policy-map interface <interface-id>` | Fix ACL, DSCP match, protocol match, or traffic generator |
| ACL counters increment but policy class counters do not | ACL is not referenced by the class-map or wrong service-policy direction is used | `show running-config class-map` and `show running-config interface <interface-id>` | Correct `match access-group name <ACL_NAME>` or move service-policy to the correct direction |
| Traffic falls into `class-default` | User-defined class does not match packet headers | `show policy-map interface <interface-id>` | Match the actual packet DSCP, port, source, destination, or protocol |
| Traffic is not dropped above the rate | Offered traffic is below CIR or burst is too large for the test window | `show policy-map interface <interface-id>` | Increase test traffic, lower CIR, or tune BC |
| Too much traffic is dropped | CIR is too low or BC is too small | `show policy-map interface <interface-id>` | Increase CIR or BC to match the intended service rate |
| Application performs poorly after policing | Policing is dropping TCP or application traffic aggressively | `show policy-map interface <interface-id>` | Use shaping instead if buffering is required, or raise the policing rate |
| CLI rejects the police command | Syntax or platform support mismatch | CLI error and `show running-config policy-map` | Use supported syntax such as `police cir <bps> bc <bytes>` |
| Counters show exceeded traffic but user expected buffering | Policing was used instead of shaping | `show running-config policy-map` | Use `shape average <bps>` when delayed forwarding is required |
| Policy attached outbound but expected ingress enforcement | Wrong direction chosen for the lab goal | `show running-config interface <interface-id>` | Move policy to `service-policy input` for ingress enforcement |
| Policy attached inbound but expected egress rate limit | Wrong direction chosen for outbound link control | `show running-config interface <interface-id>` | Move policy to `service-policy output` if platform supports it and the design requires outbound policing |
##### Source_Basis
# QoS_Policing_Exceed_Drop_Mental_Model
# QoS_Policing_Exceed_Drop_Configuration_Checklist
# QoS_Policing_Exceed_Drop_Skeleton
# QoS_Policing_Exceed_Drop_Example_Skeleton
# QoS_Policing_Exceed_Drop_DSCP_Example
# QoS_Policing_Exceed_Drop_Verification_Commands
# QoS_Policing_Exceed_Drop_Rollback
# QoS_Policing_Exceed_Drop_Failure_Checks
