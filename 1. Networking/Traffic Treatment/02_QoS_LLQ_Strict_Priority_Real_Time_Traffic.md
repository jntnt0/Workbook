
QoS_LLQ_Strict_Priority_Real_Time_Traffic.md

QoS_LLQ_Strict_Priority_Real_Time_Traffic



# QoS_LLQ_Strict_Priority_Real_Time_Traffic_Mental_Model
| Concept | Operational Meaning |
|---|---|
| LLQ | Low-Latency Queuing. LLQ is CBWFQ plus strict priority queueing |
| Strict priority queue | Queue serviced before normal CBWFQ queues during congestion |
| Real-time traffic | Voice, RTP, interactive video, or other delay-sensitive traffic that needs low delay and low jitter |
| `priority` | Enables strict priority treatment for the class |
| `priority percent <percent>` | Gives the priority class strict priority service with a percentage-based bandwidth/policer reference |
| `priority level 1` | Highest strict-priority level when multi-level priority is supported |
| `priority level 2` | Lower strict-priority level for a second real-time class, often video below voice |
| Congestion-aware policing | LLQ prevents priority traffic from starving non-priority queues during congestion |
| LLQ is not just more bandwidth | LLQ is about delay and jitter protection, not only bandwidth reservation |
| LLQ belongs outbound | Priority queueing is normally applied on egress because queues form as traffic leaves the interface |
| CBWFQ still matters | Non-priority classes should still receive `bandwidth`, `bandwidth percent`, or `bandwidth remaining percent` treatment |
| `class-default` | Catches unmatched traffic and should have explicit default behavior |
| Priority abuse risk | Too much traffic in LLQ can starve the rest of the policy or trigger drops under the priority policer |
# QoS_LLQ_Strict_Priority_Real_Time_Traffic_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm target interfaces are up before QoS changes | Router/Switch | `show ip interface brief` | Interfaces needed for the lab are up/up |
| 2 | Identify the egress interface where real-time traffic exits | Router/Switch | `show cdp neighbors` / `show lldp neighbors` / `show interfaces status` | Correct outbound interface is known |
| 3 | Confirm interface bandwidth reference | Router/Switch | `show interfaces <interface-id>` | Interface bandwidth and line protocol are visible |
| 4 | Check for existing QoS policies | Router/Switch | `show policy-map interface <interface-id>` | Existing service-policy state is known before changes |
| 5 | Define voice/RTP classification ACL if DSCP marking is not already reliable | Router/Switch | `ip access-list extended <VOICE_ACL>` | Named ACL exists for real-time traffic classification |
| 6 | Permit voice/RTP media traffic in the classification ACL | Router/Switch | `permit udp any any range <rtp-start> <rtp-end>` | RTP media traffic is identified for QoS classification |
| 7 | Define call-control classification ACL if needed | Router/Switch | `ip access-list extended <CALL_CONTROL_ACL>` | Named ACL exists for SIP, SCCP, or signaling classification |
| 8 | Permit call-control traffic in the ACL | Router/Switch | `permit tcp any any eq 5060` / `permit udp any any eq 5060` / `permit tcp any any eq 2000` | SIP or SCCP signaling traffic is identified |
| 9 | Create the voice class-map | Router/Switch | `class-map match-any <VOICE_CLASS>` | Device enters voice class-map mode |
| 10 | Match trusted voice DSCP marking | Router/Switch | `match dscp ef` | Voice class matches Expedited Forwarding traffic |
| 11 | Match RTP ACL if packet marking is not enough | Router/Switch | `match access-group name <VOICE_ACL>` | Voice class also matches RTP ACL traffic |
| 12 | Exit voice class-map mode | Router/Switch | `exit` | Device returns to global configuration mode |
| 13 | Create optional video or real-time class-map | Router/Switch | `class-map match-any <VIDEO_CLASS>` | Device enters second real-time class-map mode |
| 14 | Match video or real-time application marking | Router/Switch | `match dscp af41 af42 af43` | Video class matches expected DSCP values |
| 15 | Create call-control class-map if signaling gets separate treatment | Router/Switch | `class-map match-any <CALL_CONTROL_CLASS>` | Device enters call-control class-map mode |
| 16 | Match call-control DSCP or ACL | Router/Switch | `match dscp cs3` / `match access-group name <CALL_CONTROL_ACL>` | Signaling traffic is matched separately from RTP media |
| 17 | Create the LLQ policy-map | Router/Switch | `policy-map <LLQ_POLICY_NAME>` | Device enters policy-map configuration mode |
| 18 | Enter the voice class inside the policy-map | Router/Switch | `class <VOICE_CLASS>` | Device enters voice policy class mode |
| 19 | Configure single-level strict priority using fixed bandwidth | Router/Switch | `priority <kbps>` | Voice class receives strict priority treatment with a kbps reference |
| 20 | Configure single-level strict priority using percentage if preferred | Router/Switch | `priority percent <percent>` | Voice class receives strict priority treatment based on interface bandwidth |
| 21 | Configure highest multi-level priority if platform supports it | Router/Switch | `priority level 1 percent <percent>` | Voice class uses highest strict-priority level |
| 22 | Exit the voice policy class | Router/Switch | `exit` | Device returns to policy-map mode |
| 23 | Enter the video or second real-time class if used | Router/Switch | `class <VIDEO_CLASS>` | Device enters video policy class mode |
| 24 | Configure second-level strict priority if platform supports multi-level priority | Router/Switch | `priority level 2 percent <percent>` | Video or second real-time class receives lower strict-priority treatment than voice |
| 25 | Enter call-control class if signaling gets non-priority bandwidth treatment | Router/Switch | `class <CALL_CONTROL_CLASS>` | Device enters call-control policy class mode |
| 26 | Reserve bandwidth for call-control traffic | Router/Switch | `bandwidth percent <percent>` / `bandwidth remaining percent <percent>` | Signaling receives guaranteed non-priority bandwidth |
| 27 | Configure normal business-critical data classes if present | Router/Switch | `class <DATA_CLASS>` then `bandwidth remaining percent <percent>` | Non-real-time classes get bandwidth after priority traffic |
| 28 | Configure default class behavior | Router/Switch | `class class-default` | Unmatched traffic is handled explicitly |
| 29 | Give default traffic fair access to remaining bandwidth | Router/Switch | `bandwidth remaining percent <percent>` / `fair-queue` | Default traffic is not left completely unmanaged |
| 30 | Exit policy-map configuration mode | Router/Switch | `exit` | Device returns to global config mode |
| 31 | Enter the egress interface | Router/Switch | `interface <interface-id>` | Device enters interface configuration mode |
| 32 | Attach the LLQ policy outbound | Router/Switch | `service-policy output <LLQ_POLICY_NAME>` | LLQ policy is active on outbound traffic |
| 33 | Return to privileged EXEC mode | Router/Switch | `end` | Device exits configuration mode |
| 34 | Verify policy structure | Router/Switch | `show policy-map <LLQ_POLICY_NAME>` | Policy shows priority class and expected bandwidth actions |
| 35 | Verify interface attachment | Router/Switch | `show policy-map interface <interface-id>` | Interface shows output service-policy and per-class counters |
| 36 | Generate real-time test traffic | Hosts/Routers | RTP/SIP test call, UDP stream, `iperf -u`, or lab traffic generator | Real-time traffic crosses the egress interface |
| 37 | Confirm LLQ class counters increment | Router/Switch | `show policy-map interface <interface-id>` | Voice or real-time class packet counters increase |
| 38 | Confirm non-priority classes still receive service | Router/Switch | `show policy-map interface <interface-id>` | Data and `class-default` counters increment without excessive drops |
| 39 | Save the configuration | Router/Switch | `copy running-config startup-config` | LLQ configuration survives reload |
# QoS_LLQ_Strict_Priority_Real_Time_Traffic_Skeleton
configure terminal
ip access-list extended <VOICE_ACL>
 permit udp any any range <RTP_START_PORT> <RTP_END_PORT>
 exit
ip access-list extended <CALL_CONTROL_ACL>
 permit tcp any any eq 5060
 permit udp any any eq 5060
 exit
class-map match-any <VOICE_CLASS>
 match dscp ef
 match access-group name <VOICE_ACL>
 exit
class-map match-any <CALL_CONTROL_CLASS>
 match dscp cs3
 match access-group name <CALL_CONTROL_ACL>
 exit
class-map match-any <DATA_CLASS>
 match dscp af31 af32 af33
 exit
policy-map <LLQ_POLICY_NAME>
 class <VOICE_CLASS>
  priority percent <VOICE_PERCENT>
 exit
 class <CALL_CONTROL_CLASS>
  bandwidth remaining percent <CALL_CONTROL_PERCENT>
 exit
 class <DATA_CLASS>
  bandwidth remaining percent <DATA_PERCENT>
 exit
 class class-default
  bandwidth remaining percent <DEFAULT_PERCENT>
  fair-queue
 exit
 exit
interface <EGRESS_INTERFACE>
 service-policy output <LLQ_POLICY_NAME>
 end
show policy-map <LLQ_POLICY_NAME>
show policy-map interface <EGRESS_INTERFACE>
copy running-config startup-config
# QoS_LLQ_Strict_Priority_Real_Time_Traffic_Example_Skeleton
configure terminal
ip access-list extended VOICE_RTP
 permit udp any any range 16384 32767
 exit
ip access-list extended SIP_SIGNALING
 permit tcp any any eq 5060
 permit udp any any eq 5060
 exit
class-map match-any VOICE_RTP
 match dscp ef
 match access-group name VOICE_RTP
 exit
class-map match-any SIP_SIGNALING
 match dscp cs3
 match access-group name SIP_SIGNALING
 exit
class-map match-any BUSINESS_CRITICAL
 match dscp af31 af32 af33
 exit
policy-map WAN_LLQ
 class VOICE_RTP
  priority percent 10
 exit
 class SIP_SIGNALING
  bandwidth remaining percent 10
 exit
 class BUSINESS_CRITICAL
  bandwidth remaining percent 40
 exit
 class class-default
  bandwidth remaining percent 50
  fair-queue
 exit
 exit
interface GigabitEthernet1
 service-policy output WAN_LLQ
 end
show policy-map WAN_LLQ
show policy-map interface GigabitEthernet1
copy running-config startup-config
# QoS_LLQ_Strict_Priority_Real_Time_Traffic_Multi_Level_Example
configure terminal
class-map match-any VOICE_RTP
 match dscp ef
 exit
class-map match-any VIDEO_REALTIME
 match dscp af41 af42 af43
 exit
class-map match-any CRITICAL_DATA
 match dscp af31 af32 af33
 exit
policy-map WAN_MULTI_LEVEL_LLQ
 class VOICE_RTP
  priority level 1 percent 10
 exit
 class VIDEO_REALTIME
  priority level 2 percent 20
 exit
 class CRITICAL_DATA
  bandwidth percent 20
 exit
 class class-default
  bandwidth percent 25
  fair-queue
 exit
 exit
interface GigabitEthernet1
 service-policy output WAN_MULTI_LEVEL_LLQ
 end
show policy-map WAN_MULTI_LEVEL_LLQ
show policy-map interface GigabitEthernet1
# QoS_LLQ_Strict_Priority_Real_Time_Traffic_Verification_Commands
| Command | Purpose | Good Output |
|---|---|---|
| `show running-config class-map` | Confirms real-time traffic classification | Voice, video, or signaling classes contain expected DSCP, ACL, or protocol matches |
| `show running-config policy-map` | Confirms LLQ actions | Priority class shows `priority`, `priority percent`, or `priority level` |
| `show running-config interface <interface-id>` | Confirms policy attachment | Interface shows `service-policy output <LLQ_POLICY_NAME>` |
| `show policy-map <LLQ_POLICY_NAME>` | Verifies configured policy structure | Priority and bandwidth classes appear under the correct policy |
| `show policy-map interface <interface-id>` | Verifies runtime class counters and drops | Real-time class counters increment and drop counters are understood |
| `show access-lists <VOICE_ACL>` | Verifies ACL-based RTP classification | ACL permit counters increment when RTP traffic is generated |
| `show access-lists <CALL_CONTROL_ACL>` | Verifies signaling classification | SIP or SCCP ACL counters increment during call setup |
| `show interfaces <interface-id>` | Checks interface congestion, output drops, and rate | Interface is up/up and counters can be compared during testing |
| `clear counters <interface-id>` | Clears interface counters before validation | Interface counters reset for clean testing |
| `clear access-list counters <ACL_NAME>` | Clears ACL counters before validation | ACL counters reset for clean classification testing |
# QoS_LLQ_Strict_Priority_Real_Time_Traffic_Rollback
configure terminal
interface <EGRESS_INTERFACE>
 no service-policy output <LLQ_POLICY_NAME>
 exit
no policy-map <LLQ_POLICY_NAME>
no class-map <VOICE_CLASS>
no class-map <VIDEO_CLASS>
no class-map <CALL_CONTROL_CLASS>
no class-map <DATA_CLASS>
no ip access-list extended <VOICE_ACL>
no ip access-list extended <CALL_CONTROL_ACL>
end
show policy-map interface <EGRESS_INTERFACE>
show running-config | section policy-map
show running-config | section class-map
show running-config interface <EGRESS_INTERFACE>
# QoS_LLQ_Strict_Priority_Real_Time_Traffic_Failure_Checks
| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| LLQ policy exists but has no effect | Policy-map is not attached to the egress interface | `show running-config interface <interface-id>` | Add `service-policy output <LLQ_POLICY_NAME>` |
| Priority class counters stay at zero | Voice or RTP traffic does not match the class-map | `show policy-map interface <interface-id>` and `show access-lists <VOICE_ACL>` | Fix DSCP marking, ACL match, or traffic generator |
| Real-time traffic falls into `class-default` | Class-map does not match the actual packet headers | `show policy-map interface <interface-id>` | Match the correct DSCP, port range, protocol, or ACL |
| Voice quality still suffers | Interface is congested and priority bandwidth is too low | `show policy-map interface <interface-id>` | Increase LLQ priority allocation or reduce offered voice load |
| Data traffic starves | Priority class is too large or priority traffic is excessive | `show policy-map interface <interface-id>` | Lower priority allocation, police priority traffic, or move non-real-time traffic out of LLQ |
| CLI rejects `priority` with other queueing commands | `priority` is mixed incorrectly with `bandwidth`, `shape`, `fair-queue`, or `random-detect` in the same class | `show running-config policy-map` | Keep `priority` alone in the priority class except supported queue-limit behavior |
| CLI rejects mixed bandwidth statements | Policy mixes incompatible bandwidth styles | CLI error and `show running-config policy-map` | Use consistent bandwidth style across non-priority classes |
| `bandwidth percent` fails with unconstrained priority queue | Strict-priority queue is not constrained while fixed bandwidth guarantees are used elsewhere | CLI error and policy-map config | Use `priority percent` or use `bandwidth remaining percent` for non-priority classes |
| Video and voice are treated the same unexpectedly | Multiple real-time classes feed a single priority scheduler on platforms without effective multi-level behavior | `show policy-map <POLICY_NAME>` | Use `priority level 1` for voice and `priority level 2` for video if supported |
| Signaling is placed in strict priority queue | Call-control traffic was classified with RTP media | `show running-config class-map` | Keep signaling in a bandwidth class, not usually in LLQ strict priority |
| LLQ behavior is not visible in testing | No egress congestion exists | `show interfaces <interface-id>` and traffic generator results | Generate enough traffic to create congestion on the outbound interface |
| Policy attached inbound | Queueing policy is applied in the wrong direction | `show running-config interface <interface-id>` | Move LLQ policy to `service-policy output` |
##### Source_Basis
# QoS_LLQ_Strict_Priority_Real_Time_Traffic_Mental_Model
# QoS_LLQ_Strict_Priority_Real_Time_Traffic_Configuration_Checklist
# QoS_LLQ_Strict_Priority_Real_Time_Traffic_Skeleton
# QoS_LLQ_Strict_Priority_Real_Time_Traffic_Example_Skeleton
# QoS_LLQ_Strict_Priority_Real_Time_Traffic_Multi_Level_Example
# QoS_LLQ_Strict_Priority_Real_Time_Traffic_Verification_Commands
# QoS_LLQ_Strict_Priority_Real_Time_Traffic_Rollback
# QoS_LLQ_Strict_Priority_Real_Time_Traffic_Failure_Checks
