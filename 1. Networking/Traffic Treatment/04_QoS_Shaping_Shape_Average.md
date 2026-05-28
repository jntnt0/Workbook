

QoS_Shaping_Shape_Average.md

QoS_Shaping_Shape_Average


# QoS_Shaping_Shape_Average_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Shaping | Controls the outbound traffic rate by buffering excess packets and sending them later |
| `shape average` | Configures an average maximum transmit rate for a class |
| Not policing | Shaping delays excess traffic. Policing drops or remarks excess traffic |
| Egress behavior | Shaping is normally applied outbound because packets are queued as they leave an interface |
| Rate cap | Shaping restricts traffic to the configured rate even if the physical interface has more bandwidth available |
| Burst smoothing | Shaping absorbs short bursts so downstream devices or providers are less likely to drop packets |
| SP-facing use case | Commonly used when the local physical interface is faster than the provider handoff or contracted rate |
| Parent shaper | A parent policy shapes total outbound traffic to the WAN or provider rate |
| Child queuing policy | A child service-policy can run CBWFQ or LLQ inside the shaped rate |
| Shaped rate reference | Child queue bandwidth calculations use the shaped rate, not necessarily the physical interface speed |
| Class-based shaping | Shaping can be applied to `class class-default` for all traffic or to specific matched classes |
| Delay tradeoff | Shaping reduces drops but can increase delay when buffers fill |
# QoS_Shaping_Shape_Average_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm target interfaces are operational | Router/Switch | `show ip interface brief` | Interfaces needed for the lab are up/up |
| 2 | Identify the egress interface where shaping must occur | Router/Switch | `show cdp neighbors` / `show lldp neighbors` / `show interfaces status` | Correct outbound interface is known |
| 3 | Confirm physical interface speed and bandwidth reference | Router/Switch | `show interfaces <interface-id>` | Physical speed, bandwidth, line protocol, and counters are visible |
| 4 | Identify the intended shaped rate | Router/Switch | Lab design or provider handoff rate | Shaped rate is known before configuration |
| 5 | Check for existing QoS policy attachment | Router/Switch | `show policy-map interface <interface-id>` | Existing input or output service-policy state is known |
| 6 | Decide whether shaping applies to all traffic or a specific class | Router/Switch | Design decision | `class class-default` or a named class-map is selected |
| 7 | Create ACL only if shaping must target specific traffic | Router/Switch | `ip access-list extended <ACL_NAME>` | Named ACL exists for traffic classification |
| 8 | Add ACL permit statements for shaped traffic | Router/Switch | `permit <protocol> <source> <wildcard> <destination> <wildcard> <optional-port>` | Target traffic is identified for class-map matching |
| 9 | Create class-map for shaped traffic if not shaping all traffic | Router/Switch | `class-map match-any <CLASS_NAME>` | Device enters class-map configuration mode |
| 10 | Match traffic by ACL when classification is ACL-based | Router/Switch | `match access-group name <ACL_NAME>` | Class-map matches ACL-permitted traffic |
| 11 | Match traffic by DSCP if traffic is already marked | Router/Switch | `match dscp <dscp-value>` | Class-map matches packets with the selected DSCP value |
| 12 | Exit class-map mode | Router/Switch | `exit` | Device returns to global configuration mode |
| 13 | Create the shaping policy-map | Router/Switch | `policy-map <SHAPING_POLICY_NAME>` | Device enters policy-map configuration mode |
| 14 | Enter default class if shaping all outbound traffic | Router/Switch | `class class-default` | All unmatched traffic is selected for shaping |
| 15 | Enter named class if shaping only selected traffic | Router/Switch | `class <CLASS_NAME>` | Device enters the selected policy class |
| 16 | Configure average shaping using a fixed rate | Router/Switch | `shape average <bps>` | Class is shaped to the configured average rate |
| 17 | Configure average shaping using percentage if the design requires percentage-based shaping | Router/Switch | `shape average percent <percent>` | Class is shaped to the configured percentage rate |
| 18 | Add child policy under the parent shaper if nested queuing is needed | Router/Switch | `service-policy <CHILD_POLICY_NAME>` | CBWFQ or LLQ child policy runs inside the shaped rate |
| 19 | Exit policy class mode | Router/Switch | `exit` | Device returns to policy-map mode |
| 20 | Exit policy-map mode | Router/Switch | `exit` | Device returns to global configuration mode |
| 21 | Enter the egress interface | Router/Switch | `interface <interface-id>` | Device enters interface configuration mode |
| 22 | Attach the shaping policy outbound | Router/Switch | `service-policy output <SHAPING_POLICY_NAME>` | Shaping policy is active on outbound traffic |
| 23 | Return to privileged EXEC mode | Router/Switch | `end` | Device exits configuration mode |
| 24 | Verify policy structure | Router/Switch | `show policy-map <SHAPING_POLICY_NAME>` | Policy shows `shape average` under the correct class |
| 25 | Verify interface attachment | Router/Switch | `show running-config interface <interface-id>` | Interface shows `service-policy output <SHAPING_POLICY_NAME>` |
| 26 | Verify shaping counters and queue behavior | Router/Switch | `show policy-map interface <interface-id>` | Policy appears under the interface with shaped class counters |
| 27 | Generate traffic above the shaped rate | Hosts/Routers | `iperf`, `ping`, `curl`, or lab traffic generator | Offered traffic exceeds the shaped rate |
| 28 | Confirm shaping behavior under load | Router/Switch | `show policy-map interface <interface-id>` | Class counters, queue depth, and shaping statistics reflect outbound rate control |
| 29 | Confirm the downstream device is not seeing unmanaged bursts | Router/Switch | `show interfaces <interface-id>` | Output rate trends toward the shaped rate during sustained traffic |
| 30 | Save the configuration | Router/Switch | `copy running-config startup-config` | Shaping configuration survives reload |
# QoS_Shaping_Shape_Average_Skeleton
configure terminal
ip access-list extended <ACL_NAME>
 permit <protocol> <source> <wildcard> <destination> <wildcard> <optional-port>
 exit
class-map match-any <CLASS_NAME>
 match access-group name <ACL_NAME>
 exit
policy-map <SHAPING_POLICY_NAME>
 class <CLASS_NAME>
  shape average <BPS>
 exit
 class class-default
 exit
 exit
interface <EGRESS_INTERFACE>
 service-policy output <SHAPING_POLICY_NAME>
 end
show policy-map <SHAPING_POLICY_NAME>
show running-config interface <EGRESS_INTERFACE>
show policy-map interface <EGRESS_INTERFACE>
copy running-config startup-config
# QoS_Shaping_Shape_Average_Parent_Child_Skeleton
configure terminal
class-map match-any VOICE_RTP
 match dscp ef
 exit
class-map match-any BUSINESS_CRITICAL
 match dscp af31 af32 af33
 exit
policy-map CHILD_QUEUING
 class VOICE_RTP
  priority percent <VOICE_PERCENT>
 exit
 class BUSINESS_CRITICAL
  bandwidth remaining percent <DATA_PERCENT>
 exit
 class class-default
  bandwidth remaining percent <DEFAULT_PERCENT>
  fair-queue
 exit
 exit
policy-map PARENT_SHAPER
 class class-default
  shape average <WAN_RATE_BPS>
  service-policy CHILD_QUEUING
 exit
 exit
interface <EGRESS_INTERFACE>
 service-policy output PARENT_SHAPER
 end
show policy-map PARENT_SHAPER
show policy-map CHILD_QUEUING
show policy-map interface <EGRESS_INTERFACE>
copy running-config startup-config
# QoS_Shaping_Shape_Average_Verification_Commands
| Command | Purpose | Good Output |
|---|---|---|
| `show running-config class-map` | Confirms traffic classification | Class-map contains expected ACL, DSCP, or protocol match |
| `show running-config policy-map` | Confirms shaping action | Policy-map shows `shape average <bps>` or `shape average percent <percent>` |
| `show running-config interface <interface-id>` | Confirms service-policy attachment | Interface shows `service-policy output <SHAPING_POLICY_NAME>` |
| `show policy-map <SHAPING_POLICY_NAME>` | Verifies configured policy structure | Shaping class appears with the intended average rate |
| `show policy-map interface <interface-id>` | Verifies runtime shaping behavior | Class counters increment and shaping statistics appear under the outbound policy |
| `show policy-map type queueing interface <interface-id>` | Verifies queueing policy behavior when supported | Queueing and shaping statistics appear for the interface |
| `show access-lists <ACL_NAME>` | Verifies ACL-based classification | ACL permit counters increment during test traffic |
| `show interfaces <interface-id>` | Checks interface rate, output drops, and congestion clues | Output rate and counters can be compared against the shaped rate |
| `clear counters <interface-id>` | Clears interface counters before testing | Interface counters reset for clean validation |
| `clear access-list counters <ACL_NAME>` | Clears ACL counters before testing | ACL counters reset for clean classification testing |
# QoS_Shaping_Shape_Average_Rollback
configure terminal
interface <EGRESS_INTERFACE>
 no service-policy output <SHAPING_POLICY_NAME>
 exit
no policy-map <SHAPING_POLICY_NAME>
no policy-map <CHILD_POLICY_NAME>
no class-map <CLASS_NAME>
no class-map VOICE_RTP
no class-map BUSINESS_CRITICAL
no ip access-list extended <ACL_NAME>
end
show policy-map interface <EGRESS_INTERFACE>
show running-config | section policy-map
show running-config | section class-map
show running-config interface <EGRESS_INTERFACE>
# QoS_Shaping_Shape_Average_Failure_Checks
| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| Shaping policy exists but does nothing | Policy-map is not attached to the egress interface | `show running-config interface <interface-id>` | Add `service-policy output <SHAPING_POLICY_NAME>` |
| CLI accepts policy but counters stay at zero | Traffic does not match the shaping class | `show policy-map interface <interface-id>` and `show access-lists <ACL_NAME>` | Fix ACL, DSCP match, class-map logic, or traffic generator |
| Traffic falls into `class-default` | Named class does not match packet headers | `show policy-map interface <interface-id>` | Match the actual DSCP, protocol, source, destination, or port |
| Shaping behavior is not visible | Offered traffic is below the shaped rate | `show policy-map interface <interface-id>` and traffic generator rate | Generate traffic above the shaped rate |
| Output still bursts above expected rate | Policy applied to wrong interface or wrong direction | `show running-config interface <interface-id>` | Move policy to the correct egress interface with `service-policy output` |
| Traffic is dropped instead of delayed | Policing was configured instead of shaping | `show running-config policy-map` | Replace `police` action with `shape average` where buffering is required |
| Delay becomes excessive | Shaped rate is too low or queue is building under load | `show policy-map interface <interface-id>` | Raise shaped rate, tune child classes, or reduce offered load |
| Child CBWFQ or LLQ percentages behave unexpectedly | Child policy calculates against shaped rate, not physical interface rate | `show running-config policy-map` | Recalculate child policy bandwidth based on the parent shaped rate |
| CLI rejects nested service-policy | Platform or policy structure does not support the hierarchy used | CLI error and `show running-config policy-map` | Use supported parent-child policy format or simplify the policy |
| Downstream provider still drops traffic | Shaped rate is higher than actual provider policer or SLA rate | Provider rate, lab design, and `show policy-map interface` | Lower `shape average` below the provider enforcement rate |
| Policy attached inbound | Shaping applied in wrong direction | `show running-config interface <interface-id>` | Use `service-policy output <SHAPING_POLICY_NAME>` |
##### Source_Basis
# QoS_Shaping_Shape_Average_Mental_Model
# QoS_Shaping_Shape_Average_Configuration_Checklist
# QoS_Shaping_Shape_Average_Skeleton
# QoS_Shaping_Shape_Average_Parent_Child_Skeleton
# QoS_Shaping_Shape_Average_Verification_Commands
# QoS_Shaping_Shape_Average_Rollback
# QoS_Shaping_Shape_Average_Failure_Checks
