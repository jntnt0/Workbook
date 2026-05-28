QoS_WRED_Congestion_Avoidance.md

QoS_WRED_Congestion_Avoidance

# Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| `_KB_INDEX.md` | IOS XE QoS `(MQC/CBWFQ/LLQ/NBAR)` mapped to `iosxe_combined_pdfs_.md` lines `29,556-33,280` | Identifies IOS XE QoS Configuration Guide as the primary source for MQC queue-management syntax |
| `iosxe_combined_pdfs_.md` | QoS Configuration Guide, MQC policy-map class actions, `random-detect`, queue management, `show policy-map interface` | Provides IOS XE syntax for enabling WRED under MQC classes and verifying runtime counters |
| `CCNP and CCIE Enterprise Core ENCOR 350-401 Official Cert.md` | Congestion-Avoidance Tools, CBWFQ queue management commands, WRED, `random-detect dscp-based`, `queue-limit` | Supports the operational model for WRED as early drop congestion avoidance, not priority queueing |
| Related lab | `qos-wred-final` | Lab focused on Weighted Random Early Detection congestion avoidance |
# QoS_WRED_Congestion_Avoidance_Mental_Model
| Concept | Operational Meaning |
|---|---|
| WRED | Weighted Random Early Detection. Cisco congestion-avoidance mechanism that drops packets before a queue is completely full |
| Congestion avoidance | WRED tries to prevent full queue exhaustion by dropping selected packets early |
| Not congestion management | WRED does not schedule traffic like CBWFQ or LLQ. It manages queue depth by dropping packets |
| Not priority | WRED does not give strict priority service. Voice-style priority belongs in LLQ |
| Tail drop | Default behavior where packets are dropped only after the queue fills |
| Global TCP synchronization | Tail drop can cause many TCP sessions to back off at the same time, underutilize the link, then surge again |
| RED | Random Early Detection. Drops random packets before full queue exhaustion |
| WRED weighting | WRED can drop lower-priority markings more aggressively than higher-priority markings |
| DSCP-based WRED | Uses DSCP values to influence drop behavior |
| IP Precedence-based WRED | Uses IP precedence values to influence drop behavior |
| Minimum threshold | Queue depth where WRED begins probabilistic drops |
| Maximum threshold | Queue depth where WRED reaches strongest drop probability before full tail drop behavior |
| Mark probability denominator | Controls how aggressive the random drop probability is |
| TCP-friendly | WRED is most useful for TCP because dropped packets cause TCP senders to reduce transmission rate |
| Poor fit for voice/RTP | UDP real-time traffic usually does not back off from WRED drops, so WRED should not be used as the main protection mechanism for voice |
| MQC dependency | WRED is configured under a policy-map class and normally paired with `bandwidth` or `shape` in user-defined classes |
# QoS_WRED_Congestion_Avoidance_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm target interfaces are operational before QoS changes | Router/Switch | `show ip interface brief` | Interfaces needed for the lab are up/up |
| 2 | Identify the egress interface where congestion avoidance is needed | Router/Switch | `show cdp neighbors` / `show lldp neighbors` / `show interfaces status` | Correct outbound interface is known |
| 3 | Confirm interface bandwidth, rate, and existing drops | Router/Switch | `show interfaces <interface-id>` | Interface speed, output rate, queue drops, and line protocol are visible |
| 4 | Check for existing QoS policy attachment | Router/Switch | `show policy-map interface <interface-id>` | Existing output service-policy state is known |
| 5 | Decide which class should use WRED | Router/Switch | Design decision | WRED is assigned to TCP-heavy data classes or `class-default`, not voice priority traffic |
| 6 | Create ACL only if traffic classification requires source, destination, protocol, or port matching | Router/Switch | `ip access-list extended <ACL_NAME>` | Named ACL exists for classification |
| 7 | Add ACL permit statements for the WRED-managed traffic | Router/Switch | `permit <protocol> <source> <wildcard> <destination> <wildcard> <optional-port>` | Target traffic is identified for class-map matching |
| 8 | Create a class-map for the WRED-managed class | Router/Switch | `class-map match-any <CLASS_NAME>` | Device enters class-map configuration mode |
| 9 | Match traffic by ACL if classification is ACL-based | Router/Switch | `match access-group name <ACL_NAME>` | Class-map matches ACL-permitted traffic |
| 10 | Match traffic by DSCP if classification uses markings | Router/Switch | `match dscp <dscp-value>` | Class-map matches selected DSCP-marked traffic |
| 11 | Exit class-map mode | Router/Switch | `exit` | Device returns to global configuration mode |
| 12 | Create the WRED policy-map | Router/Switch | `policy-map <POLICY_NAME>` | Device enters policy-map configuration mode |
| 13 | Enter the class that will use WRED | Router/Switch | `class <CLASS_NAME>` | Device enters policy-map class mode |
| 14 | Add a queuing action required for user-defined WRED classes | Router/Switch | `bandwidth percent <percent>` / `bandwidth remaining percent <percent>` / `shape average <bps>` | Class has a supported queuing or shaping action before WRED |
| 15 | Enable basic WRED in the class | Router/Switch | `random-detect` | WRED is enabled with default behavior |
| 16 | Enable DSCP-based WRED when DSCP markings should influence drop behavior | Router/Switch | `random-detect dscp-based` | WRED uses DSCP values for weighted drop decisions |
| 17 | Tune DSCP threshold behavior if the lab requires explicit thresholds | Router/Switch | `random-detect dscp <dscp-value> <min-threshold> <max-threshold> <mark-prob-denominator>` | Selected DSCP has custom WRED threshold and drop probability behavior |
| 18 | Configure queue limit if the lab requires an explicit queue depth | Router/Switch | `queue-limit <value>` | Class queue limit is explicitly set |
| 19 | Exit the WRED class | Router/Switch | `exit` | Device returns to policy-map mode |
| 20 | Configure WRED under `class-default` if default TCP traffic needs congestion avoidance | Router/Switch | `class class-default` then `random-detect dscp-based` | Unmatched traffic receives DSCP-based WRED behavior |
| 21 | Avoid configuring WRED inside a strict-priority class | Router/Switch | `show running-config policy-map` | No `random-detect` command appears under a class using `priority` |
| 22 | Exit policy-map mode | Router/Switch | `exit` | Device returns to global configuration mode |
| 23 | Enter the target egress interface | Router/Switch | `interface <interface-id>` | Device enters interface configuration mode |
| 24 | Attach the WRED policy outbound | Router/Switch | `service-policy output <POLICY_NAME>` | WRED policy is active on outbound traffic |
| 25 | Return to privileged EXEC mode | Router/Switch | `end` | Device exits configuration mode |
| 26 | Verify policy structure | Router/Switch | `show policy-map <POLICY_NAME>` | Policy shows expected class, queueing action, and `random-detect` |
| 27 | Verify service-policy attachment | Router/Switch | `show running-config interface <interface-id>` | Interface shows `service-policy output <POLICY_NAME>` |
| 28 | Verify runtime WRED counters | Router/Switch | `show policy-map interface <interface-id>` | Matching class counters increment and WRED/drop statistics appear under load |
| 29 | Generate TCP-heavy traffic above the link or shaped rate | Hosts/Routers | `iperf`, `curl`, file transfer, or lab traffic generator | Queue depth increases and WRED has a reason to act |
| 30 | Confirm early drops occur before full queue exhaustion | Router/Switch | `show policy-map interface <interface-id>` | WRED random-drop counters increase before only tail-drop behavior dominates |
| 31 | Confirm protected traffic is not placed in the wrong class | Router/Switch | `show policy-map interface <interface-id>` | Voice or strict-priority traffic is not counted in the WRED class unless intentionally designed |
| 32 | Save the configuration | Router/Switch | `copy running-config startup-config` | WRED configuration survives reload |
# QoS_WRED_Congestion_Avoidance_Skeleton
configure terminal
ip access-list extended <ACL_NAME>
 permit <protocol> <source> <wildcard> <destination> <wildcard> <optional-port>
 exit
class-map match-any <CLASS_NAME>
 match access-group name <ACL_NAME>
 exit
policy-map <POLICY_NAME>
 class <CLASS_NAME>
  bandwidth remaining percent <PERCENT>
  random-detect dscp-based
 exit
 class class-default
  bandwidth remaining percent <PERCENT>
  fair-queue
  random-detect dscp-based
 exit
 exit
interface <EGRESS_INTERFACE>
 service-policy output <POLICY_NAME>
 end
show policy-map <POLICY_NAME>
show running-config interface <EGRESS_INTERFACE>
show policy-map interface <EGRESS_INTERFACE>
copy running-config startup-config
# QoS_WRED_Congestion_Avoidance_Example_Skeleton
configure terminal
ip access-list extended BULK_TCP
 permit tcp any any
 exit
class-map match-any BULK_TCP
 match access-group name BULK_TCP
 exit
class-map match-any BUSINESS_CRITICAL
 match dscp af31 af32 af33
 exit
policy-map WAN_WRED
 class BUSINESS_CRITICAL
  bandwidth remaining percent 40
 exit
 class BULK_TCP
  bandwidth remaining percent 30
  random-detect dscp-based
 exit
 class class-default
  bandwidth remaining percent 30
  fair-queue
  random-detect dscp-based
  queue-limit 64
 exit
 exit
interface GigabitEthernet1
 service-policy output WAN_WRED
 end
show policy-map WAN_WRED
show running-config interface GigabitEthernet1
show policy-map interface GigabitEthernet1
copy running-config startup-config
# QoS_WRED_Congestion_Avoidance_Threshold_Tuning_Example
configure terminal
class-map match-any AF_DATA
 match dscp af11 af12 af13 af21 af22 af23
 exit
policy-map WAN_WRED_THRESHOLDS
 class AF_DATA
  bandwidth remaining percent 50
  random-detect dscp-based
  random-detect dscp af13 20 40 10
  random-detect dscp af12 30 50 10
  random-detect dscp af11 40 60 10
 exit
 class class-default
  bandwidth remaining percent 50
  fair-queue
  random-detect dscp-based
 exit
 exit
interface GigabitEthernet1
 service-policy output WAN_WRED_THRESHOLDS
 end
show policy-map WAN_WRED_THRESHOLDS
show policy-map interface GigabitEthernet1
# QoS_WRED_Congestion_Avoidance_Verification_Commands
| Command | Purpose | Good Output |
|---|---|---|
| `show running-config class-map` | Confirms WRED-managed traffic classification | Class-map contains expected ACL, DSCP, or protocol match |
| `show running-config policy-map` | Confirms WRED policy actions | Policy class shows `bandwidth`, `bandwidth remaining`, or `shape`, plus `random-detect` |
| `show running-config interface <interface-id>` | Confirms service-policy attachment | Interface shows `service-policy output <POLICY_NAME>` |
| `show policy-map <POLICY_NAME>` | Verifies configured policy structure | WRED class appears with `random-detect` or `random-detect dscp-based` |
| `show policy-map interface <interface-id>` | Verifies runtime WRED behavior | Class counters increment and random-drop or WRED statistics appear under congestion |
| `show access-lists <ACL_NAME>` | Verifies ACL-based classification | ACL permit counters increment during test traffic |
| `show interfaces <interface-id>` | Checks output drops, queue behavior, and offered rate | Interface counters help confirm congestion exists |
| `clear counters <interface-id>` | Clears interface counters before validation | Interface counters reset for clean testing |
| `clear access-list counters <ACL_NAME>` | Clears ACL counters before validation | ACL counters reset for clean classification testing |
# QoS_WRED_Congestion_Avoidance_Rollback
configure terminal
interface <EGRESS_INTERFACE>
 no service-policy output <POLICY_NAME>
 exit
no policy-map <POLICY_NAME>
no class-map <CLASS_NAME>
no class-map AF_DATA
no class-map BULK_TCP
no class-map BUSINESS_CRITICAL
no ip access-list extended <ACL_NAME>
no ip access-list extended BULK_TCP
end
show policy-map interface <EGRESS_INTERFACE>
show running-config | section policy-map
show running-config | section class-map
show running-config interface <EGRESS_INTERFACE>
# QoS_WRED_Congestion_Avoidance_Failure_Checks
| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| WRED policy exists but does nothing | Policy-map is not attached outbound | `show running-config interface <interface-id>` | Add `service-policy output <POLICY_NAME>` |
| WRED counters stay at zero | No traffic matches the WRED class | `show policy-map interface <interface-id>` and `show access-lists <ACL_NAME>` | Fix ACL, DSCP match, class-map logic, or traffic generator |
| Traffic goes to `class-default` | Named class does not match actual packet headers | `show policy-map interface <interface-id>` | Match the real DSCP, protocol, source, destination, or port |
| WRED behavior is not visible | Interface or shaped class is not congested | `show interfaces <interface-id>` and `show policy-map interface <interface-id>` | Generate enough traffic to build the queue |
| Drops occur only after queue fills | WRED not enabled or thresholds are too high | `show running-config policy-map` | Add `random-detect dscp-based` or lower WRED thresholds |
| Too many packets are dropped early | WRED thresholds are too low or drop probability is too aggressive | `show policy-map interface <interface-id>` | Raise minimum and maximum thresholds or adjust mark probability denominator |
| CLI rejects `random-detect` in a user-defined class | Required queuing or shaping action is missing | `show running-config policy-map` | Add `bandwidth`, `bandwidth remaining`, or `shape` before WRED in that class |
| CLI rejects WRED under priority class | `random-detect` cannot coexist with strict priority in the same class | `show running-config policy-map` | Remove WRED from LLQ class and apply it to data or default classes |
| Voice quality degrades | Voice or RTP traffic was placed into WRED class | `show policy-map interface <interface-id>` | Move voice traffic to LLQ and remove it from WRED classification |
| UDP traffic does not slow down after drops | WRED relies mainly on TCP backoff behavior | Traffic type and class counters | Use policing, shaping, or application-specific controls for UDP-heavy classes |
| Lower-priority DSCP traffic is not dropped more aggressively | WRED is not DSCP-based or DSCP markings are missing | `show running-config policy-map` and `show policy-map interface <interface-id>` | Use `random-detect dscp-based` and ensure traffic is marked correctly |
| No difference between AF drop precedences | Custom thresholds were not configured or DSCP markings are not present | `show running-config policy-map` | Add DSCP-based thresholds or fix upstream marking |
##### Source_Basis
# QoS_WRED_Congestion_Avoidance_Mental_Model
# QoS_WRED_Congestion_Avoidance_Configuration_Checklist
# QoS_WRED_Congestion_Avoidance_Skeleton
# QoS_WRED_Congestion_Avoidance_Example_Skeleton
# QoS_WRED_Congestion_Avoidance_Threshold_Tuning_Example
# QoS_WRED_Congestion_Avoidance_Verification_Commands
# QoS_WRED_Congestion_Avoidance_Rollback
# QoS_WRED_Congestion_Avoidance_Failure_Checks
