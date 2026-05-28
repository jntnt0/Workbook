QoS_CBWFQ_Bandwidth_Guarantee.md

QoS_CBWFQ_Bandwidth_Guarantee

# QoS_CBWFQ_Bandwidth_Guarantee_Configuration_Checklist
| Step | Task                                                                                                | Device        | Command                                                                                | Expected Result                                                            |
| ---: | --------------------------------------------------------------------------------------------------- | ------------- | -------------------------------------------------------------------------------------- | -------------------------------------------------------------------------- |
|    1 | Confirm interface state before applying QoS                                                         | Router/Switch | `show ip interface brief`                                                              | Target interface is up/up                                                  |
|    2 | Identify the egress interface where congestion should be controlled                                 | Router/Switch | `show interfaces status` / `show cdp neighbors` / `show lldp neighbors`                | Correct outbound interface is known                                        |
|    3 | Confirm current interface bandwidth reference                                                       | Router/Switch | `show interfaces <interface-id>`                                                       | Interface bandwidth and line protocol are visible                          |
|    4 | Confirm baseline MQC configuration is absent or understood                                          | Router/Switch | `show policy-map interface <interface-id>`                                             | No unexpected existing output policy is attached                           |
|    5 | Create ACLs only if traffic classification requires source, destination, protocol, or port matching | Router/Switch | `ip access-list extended <ACL_NAME>`                                                   | Named ACL exists for class-map matching                                    |
|    6 | Add ACL permit entries for the traffic class                                                        | Router/Switch | `permit <protocol> <source> <wildcard> <destination> <wildcard> <optional-port>`       | Target traffic is identified by ACL permit statements                      |
|    7 | Create the first CBWFQ class-map                                                                    | Router/Switch | `class-map match-any <CLASS_NAME>`                                                     | Device enters class-map configuration mode                                 |
|    8 | Match traffic by ACL when needed                                                                    | Router/Switch | `match access-group name <ACL_NAME>`                                                   | Class-map matches ACL-permitted traffic                                    |
|    9 | Match traffic by DSCP when traffic is already marked                                                | Router/Switch | `match dscp <dscp-value>`                                                              | Class-map matches the selected DSCP value                                  |
|   10 | Exit class-map mode                                                                                 | Router/Switch | `exit`                                                                                 | Device returns to global configuration mode                                |
|   11 | Repeat class-map creation for each CBWFQ queue                                                      | Router/Switch | `class-map match-any <NEXT_CLASS_NAME>`                                                | Each traffic type has its own classification bucket                        |
|   12 | Create the CBWFQ policy-map                                                                         | Router/Switch | `policy-map <POLICY_NAME>`                                                             | Device enters policy-map configuration mode                                |
|   13 | Enter the first class inside the policy-map                                                         | Router/Switch | `class <CLASS_NAME>`                                                                   | Device enters policy-map class mode                                        |
|   14 | Assign fixed bandwidth if using kbps reservation                                                    | Router/Switch | `bandwidth <kbps>`                                                                     | Class receives a minimum kbps bandwidth guarantee                          |
|   15 | Assign percent bandwidth if using percentage reservation                                            | Router/Switch | `bandwidth percent <percent>`                                                          | Class receives a minimum percentage bandwidth guarantee                    |
|   16 | Assign remaining bandwidth if sharing leftover capacity                                             | Router/Switch | `bandwidth remaining percent <percent>`                                                | Class receives a percentage of remaining bandwidth                         |
|   17 | Assign ratio-based sharing if platform or design uses ratios                                        | Router/Switch | `bandwidth remaining ratio <ratio>`                                                    | Class receives bandwidth based on relative queue ratio                     |
|   18 | Exit the class and configure additional CBWFQ classes                                               | Router/Switch | `exit` then `class <NEXT_CLASS_NAME>`                                                  | Multiple traffic classes receive separate bandwidth treatment              |
|   19 | Configure `class-default` behavior                                                                  | Router/Switch | `class class-default`                                                                  | Unmatched traffic has an explicit queueing treatment                       |
|   20 | Assign bandwidth or fair queueing to default traffic if required                                    | Router/Switch | `bandwidth percent <percent>` / `bandwidth remaining percent <percent>` / `fair-queue` | Default traffic is not left unmanaged                                      |
|   21 | Verify the total bandwidth allocation does not exceed platform or policy limits                     | Router/Switch | `show running-config policy-map`                                                       | Bandwidth statements are consistent across classes                         |
|   22 | Enter the target outbound interface                                                                 | Router/Switch | `interface <interface-id>`                                                             | Device enters interface configuration mode                                 |
|   23 | Attach the CBWFQ policy outbound                                                                    | Router/Switch | `service-policy output <POLICY_NAME>`                                                  | CBWFQ policy is applied to egress traffic                                  |
|   24 | Exit configuration mode                                                                             | Router/Switch | `end`                                                                                  | Device returns to privileged EXEC mode                                     |
|   25 | Verify policy structure                                                                             | Router/Switch | `show policy-map <POLICY_NAME>`                                                        | Classes show expected bandwidth actions                                    |
|   26 | Verify policy attachment and runtime counters                                                       | Router/Switch | `show policy-map interface <interface-id>`                                             | Interface shows the output service-policy and per-class counters           |
|   27 | Generate test traffic for each class                                                                | Hosts/Routers | `ping`, `iperf`, `curl`, `telnet`, or lab-specific traffic generator                   | Traffic crosses the egress interface                                       |
|   28 | Confirm class counters increment                                                                    | Router/Switch | `show policy-map interface <interface-id>`                                             | Matching packet and byte counters increase under the expected class        |
|   29 | Confirm traffic is not falling into the wrong queue                                                 | Router/Switch | `show policy-map interface <interface-id>`                                             | Intended traffic increments the configured class, not only `class-default` |
|   30 | Save the configuration                                                                              | Router/Switch | `copy running-config startup-config`                                                   | CBWFQ policy survives reload                                               |
# QoS_CBWFQ_Bandwidth_Guarantee_Skeleton
configure terminal
ip access-list extended <ACL_NAME_1>
 permit <protocol> <source> <wildcard> <destination> <wildcard> <optional-port>
 exit
ip access-list extended <ACL_NAME_2>
 permit <protocol> <source> <wildcard> <destination> <wildcard> <optional-port>
 exit
class-map match-any <CLASS_NAME_1>
 match access-group name <ACL_NAME_1>
 exit
class-map match-any <CLASS_NAME_2>
 match access-group name <ACL_NAME_2>
 exit
policy-map <CBWFQ_POLICY_NAME>
 class <CLASS_NAME_1>
  bandwidth percent <PERCENT>
 exit
 class <CLASS_NAME_2>
  bandwidth percent <PERCENT>
 exit
 class class-default
  bandwidth percent <PERCENT>
  fair-queue
 exit
 exit
interface <EGRESS_INTERFACE>
 service-policy output <CBWFQ_POLICY_NAME>
 end
show policy-map <CBWFQ_POLICY_NAME>
show policy-map interface <EGRESS_INTERFACE>
copy running-config startup-config
# QoS_CBWFQ_Bandwidth_Guarantee_Example_Skeleton
configure terminal
ip access-list extended BUSINESS_CRITICAL
 permit tcp any any eq 443
 permit tcp any any eq 22
 exit
ip access-list extended BULK_DATA
 permit tcp any any eq ftp
 permit tcp any any eq ftp-data
 exit
class-map match-any BUSINESS_CRITICAL
 match access-group name BUSINESS_CRITICAL
 exit
class-map match-any BULK_DATA
 match access-group name BULK_DATA
 exit
policy-map WAN_CBWFQ
 class BUSINESS_CRITICAL
  bandwidth percent 30
 exit
 class BULK_DATA
  bandwidth percent 10
 exit
 class class-default
  bandwidth percent 25
  fair-queue
 exit
 exit
interface GigabitEthernet1
 service-policy output WAN_CBWFQ
 end
show policy-map WAN_CBWFQ
show policy-map interface GigabitEthernet1
copy running-config startup-config
# QoS_CBWFQ_Bandwidth_Guarantee_Remaining_Bandwidth_Example
configure terminal
class-map match-any CRITICAL_DATA
 match dscp af31 af32 af33
 exit
class-map match-any BULK_DATA
 match dscp af11 af12 af13
 exit
class-map match-any SCAVENGER
 match dscp cs1
 exit
policy-map WAN_CBWFQ_REMAINING
 class CRITICAL_DATA
  bandwidth remaining percent 40
 exit
 class BULK_DATA
  bandwidth remaining percent 30
 exit
 class SCAVENGER
  bandwidth remaining percent 10
 exit
 class class-default
  bandwidth remaining percent 20
 exit
 exit
interface GigabitEthernet1
 service-policy output WAN_CBWFQ_REMAINING
 end
show policy-map WAN_CBWFQ_REMAINING
show policy-map interface GigabitEthernet1
# QoS_CBWFQ_Bandwidth_Guarantee_Verification_Commands
| Command | Purpose | Good Output |
|---|---|---|
| `show running-config class-map` | Confirms traffic classification | Class-maps exist with expected match statements |
| `show running-config policy-map` | Confirms CBWFQ bandwidth actions | Policy-map classes show `bandwidth`, `bandwidth percent`, `bandwidth remaining percent`, or `bandwidth remaining ratio` |
| `show running-config interface <interface-id>` | Confirms policy attachment | Interface shows `service-policy output <POLICY_NAME>` |
| `show policy-map <POLICY_NAME>` | Verifies policy definition | Policy-map displays expected classes and bandwidth guarantees |
| `show policy-map interface <interface-id>` | Verifies runtime queue behavior | Output policy appears with packet and byte counters per class |
| `show access-lists <ACL_NAME>` | Verifies ACL-based classification | ACL permit counters increment for matched traffic |
| `show interfaces <interface-id>` | Checks interface rate, drops, and congestion clues | Interface is up/up and counters can be compared before and after testing |
| `clear counters <interface-id>` | Clears interface counters before testing | Interface counters reset |
| `clear access-list counters <ACL_NAME>` | Clears ACL counters before traffic generation | ACL counters reset for clean validation |
# QoS_CBWFQ_Bandwidth_Guarantee_Rollback
configure terminal
interface <EGRESS_INTERFACE>
 no service-policy output <CBWFQ_POLICY_NAME>
 exit
no policy-map <CBWFQ_POLICY_NAME>
no class-map <CLASS_NAME_1>
no class-map <CLASS_NAME_2>
no class-map <CLASS_NAME_3>
no ip access-list extended <ACL_NAME_1>
no ip access-list extended <ACL_NAME_2>
end
show policy-map interface <EGRESS_INTERFACE>
show running-config | section policy-map
show running-config | section class-map
show running-config interface <EGRESS_INTERFACE>
# QoS_CBWFQ_Bandwidth_Guarantee_Failure_Checks
| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| Policy-map exists but has no effect | Policy is not attached to the interface | `show running-config interface <interface-id>` | Add `service-policy output <POLICY_NAME>` |
| Counters stay at zero | Traffic does not match the class-map | `show policy-map interface <interface-id>` and `show access-lists <ACL_NAME>` | Fix ACL, DSCP match, protocol match, or traffic generator |
| Traffic goes to `class-default` | User-defined class-map does not match packets | `show policy-map interface <interface-id>` | Correct the class-map match statement |
| CBWFQ behavior is not visible | Interface is not congested | `show interfaces <interface-id>` and traffic generator output | Generate enough traffic to create egress congestion |
| Wrong class receives traffic | ACL or DSCP match is too broad | `show access-lists` and `show running-config class-map` | Tighten match criteria |
| CLI rejects bandwidth command | Mixed bandwidth command types or unsupported platform behavior | CLI error and `show running-config policy-map` | Use one bandwidth style consistently across classes |
| Bandwidth guarantees do not add up cleanly | Percent allocations exceed valid total | `show running-config policy-map` | Reduce class percentages so total allocation is valid |
| Expected low-latency voice behavior is missing | CBWFQ used instead of LLQ | `show running-config policy-map` | Use LLQ with `priority` for strict priority voice treatment |
| Queue drops still occur | CBWFQ guarantees bandwidth but does not eliminate congestion | `show policy-map interface <interface-id>` | Increase bandwidth, adjust class allocations, shape parent policy, or reduce offered load |
| Policy attached inbound | Queuing policy applied in wrong direction | `show running-config interface <interface-id>` | Move CBWFQ to `service-policy output` |
| Default traffic starves or behaves unpredictably | `class-default` has no explicit treatment | `show running-config policy-map` | Configure bandwidth or fair-queue under `class class-default` |
##### Source_Basis
# QoS_CBWFQ_Bandwidth_Guarantee_Mental_Model
# QoS_CBWFQ_Bandwidth_Guarantee_Configuration_Checklist
# QoS_CBWFQ_Bandwidth_Guarantee_Skeleton
# QoS_CBWFQ_Bandwidth_Guarantee_Example_Skeleton
# QoS_CBWFQ_Bandwidth_Guarantee_Remaining_Bandwidth_Example
# QoS_CBWFQ_Bandwidth_Guarantee_Verification_Commands
# QoS_CBWFQ_Bandwidth_Guarantee_Rollback
# QoS_CBWFQ_Bandwidth_Guarantee_Failure_Checks
