
OSPF_Troubleshooting_Workflow.md

OSPF_Troubleshooting_Workflow

# Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| `_KB_INDEX.md` | OSPF topic map | Points enterprise OSPF to `All_combined_part3.md` Book 6 and ENARSI OSPF material as primary project sources |
| `All_combined_part3.md` | Book 6, Chapter 8 OSPF | Supports OSPF adjacency, LSDB, route-type, area, LSA, virtual-link, filtering, and verification workflow |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | Chapter 8, Troubleshooting OSPFv2 Neighbor Adjacencies | Supports neighbor failure checks: interface down, OSPF not enabled, timer mismatch, area mismatch, area type mismatch, subnet mismatch, passive interface, authentication, ACLs, MTU, duplicate router ID, and network type mismatch |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | Chapter 8, Troubleshooting OSPFv2 Routes | Supports route failure checks: missing LSDB entry, better route source, route filtering, stub area behavior, shut interfaces, wrong DR, duplicate router IDs, summarization, and discontiguous areas |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | Chapter 7, Advanced OSPF | Supports LSA type interpretation, path selection, stub/NSSA behavior, summarization, external routes, and virtual links |
| Related lab | `ospf-troubleshooting-final` | Main lab for end-to-end OSPF troubleshooting across adjacency, LSDB, RIB, and forwarding failures |
# OSPF_Troubleshooting_Workflow_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Troubleshooting order | Do not start with random config edits. Work in layers: underlay, interface, OSPF enablement, neighbor state, LSDB, RIB, forwarding |
| OSPF adjacency | Neighbor relationship must form before LSAs can be exchanged |
| FULL state | `FULL` means the routers successfully exchanged link-state information |
| 2-WAY state | Normal between DROTHER routers on broadcast segments, but suspicious when full adjacency is expected |
| INIT state | Local router received a hello, but the neighbor did not list the local router ID in its hello |
| EXSTART or EXCHANGE | Usually points to MTU mismatch, duplicate router ID, or database description negotiation failure |
| LOADING | Router is requesting missing LSAs. If stuck, suspect LSA exchange, corruption, resource issue, or MTU problem |
| Interface participation | OSPF must be enabled on the interface by `network` statement or `ip ospf <process-id> area <area-id>` |
| Interface command precedence | Interface-level `ip ospf <process-id> area <area-id>` takes precedence over matching `network` statements |
| Area membership | OSPF area is interface-based. Both ends of a link must be in the same area |
| Timers | OSPF hello and dead timers must match between neighbors |
| Area type | Stub, totally stub, NSSA, and totally NSSA flags must match inside the area |
| Subnet match | Neighboring interfaces must share the same subnet unless a specific network type design says otherwise |
| Passive interface | Passive suppresses OSPF hellos while still advertising the interface prefix |
| Authentication | Both sides must match authentication type, key ID, key string, and algorithm |
| ACL impact | Inbound ACLs can block OSPF protocol 89 or multicast 224.0.0.5 and 224.0.0.6 |
| MTU | MTU mismatch often causes EXSTART or EXCHANGE problems |
| Router ID | Router IDs must be unique. Duplicate router IDs break adjacencies and LSDB correctness |
| Network type | Broadcast, point-to-point, NBMA, and point-to-multipoint behave differently for timers, DR/BDR, and neighbor discovery |
| LSDB before RIB | If the prefix is missing from the LSDB, fix origination or flooding. If it is in the LSDB but not the RIB, troubleshoot route selection or filtering |
| Type 1 LSA | Router LSA. Local to area. Shows router links, stub networks, and topology information |
| Type 2 LSA | Network LSA. Created by DR on broadcast or NBMA multiaccess segments |
| Type 3 LSA | Summary LSA. Created by ABR to advertise interarea prefixes |
| Type 4 LSA | ASBR summary LSA. Created by ABR to tell other areas how to reach an ASBR |
| Type 5 LSA | External LSA. Created by ASBR for redistributed external routes |
| Type 7 LSA | NSSA external LSA. Created inside NSSA and translated to Type 5 by the NSSA ABR |
| RIB install | OSPF route must win route class, metric, administrative distance, and local filtering before it installs |
| Forwarding check | A route in the RIB is not enough. Verify CEF, next hop, return path, ACLs, and traceroute |
# OSPF_Troubleshooting_Workflow_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Define the failure symptom | Operator | `Symptom: neighbor down / route missing / bad path / traffic failure` | Troubleshooting has a clear target |
| 2 | Identify the affected source and destination | Operator | `<source-router> to <destination-prefix>` | Exact test path is known |
| 3 | Confirm physical or virtual interface state | Affected routers | `show ip interface brief` | Required interfaces are `up/up` |
| 4 | Confirm interface IP addressing | Affected routers | `show running-config interface <interface-id>` | IP address and mask match the design |
| 5 | Confirm direct connected reachability | Adjacent routers | `ping <direct-neighbor-ip>` | Direct neighbor responds |
| 6 | Confirm Layer 2 or tunnel neighbor expectation | Adjacent routers | `show cdp neighbors` or `show lldp neighbors` | Expected physical neighbor is visible where applicable |
| 7 | Confirm the active OSPF process | Affected routers | `show ip protocols` | Correct OSPF process ID and router ID are visible |
| 8 | Confirm OSPF is enabled on the intended interface | Affected routers | `show ip ospf interface brief` | Interface appears under the correct OSPF process and area |
| 9 | Check both OSPF enablement methods | Affected routers | `show running-config | section router ospf` and `show running-config interface <interface-id>` | OSPF is enabled by either `network` statement or interface-level `ip ospf` |
| 10 | Confirm the interface is not accidentally passive | Affected routers | `show ip protocols` | Neighbor-facing interface is not listed as passive |
| 11 | Confirm neighbor table state | Affected routers | `show ip ospf neighbor` | Expected neighbors are present and ideally `FULL` |
| 12 | Check detailed neighbor information | Affected routers | `show ip ospf neighbor detail` | Neighbor state, dead timer, interface, and adjacency details are visible |
| 13 | Interpret DOWN or missing neighbor | Affected routers | `show ip ospf interface <interface-id>` | Interface should show OSPF active with hello traffic expected |
| 14 | Interpret INIT state | Affected routers | `show ip ospf neighbor` | INIT means one-way hello visibility or missing self RID in neighbor hello |
| 15 | Interpret 2-WAY state | Affected routers | `show ip ospf neighbor` | 2-WAY is normal between DROTHERs, abnormal if full adjacency is expected |
| 16 | Interpret EXSTART or EXCHANGE state | Affected routers | `show ip ospf neighbor` | Suspect MTU mismatch or duplicate router ID |
| 17 | Interpret LOADING state | Affected routers | `show ip ospf neighbor detail` | Suspect LSA exchange, missing LSAs, corruption, resource issue, or MTU |
| 18 | Verify area ID on both sides | Adjacent routers | `show ip ospf interface <interface-id>` | Both sides use the same area ID |
| 19 | Verify hello and dead timers | Adjacent routers | `show ip ospf interface <interface-id> | include Timer` | Hello and dead timers match |
| 20 | Verify OSPF network type | Adjacent routers | `show ip ospf interface <interface-id> | include Network Type` | Network type is compatible on both sides |
| 21 | Verify subnet match | Adjacent routers | `show running-config interface <interface-id>` | Neighbor-facing IP addresses are in the same subnet |
| 22 | Verify area type | Affected routers and ABRs | `show ip ospf` | Area type matches across routers in the area |
| 23 | Verify authentication mode | Adjacent routers | `show ip ospf interface <interface-id>` | Authentication type matches on both sides |
| 24 | Verify authentication key configuration | Adjacent routers | `show running-config interface <interface-id>` | Key ID, key string, and authentication method match |
| 25 | Verify ACLs are not blocking OSPF | Affected routers | `show ip interface <interface-id>` and `show access-lists` | Inbound ACL permits OSPF protocol 89 and required multicast |
| 26 | Verify MTU consistency | Adjacent routers | `show interface <interface-id>` and `show running-config interface <interface-id>` | MTU values match or OSPF MTU ignore is intentionally used |
| 27 | Verify unique router IDs | All affected routers | `show ip protocols` and `show ip ospf | include ID` | No duplicate OSPF router IDs exist |
| 28 | Check logs for explicit OSPF errors | Affected routers | `show logging | include OSPF|DUP|AUTH|mismatch|ADJCHG` | Logs identify area, authentication, RID, or adjacency issues |
| 29 | Use debug only after show commands narrow the fault | Lab router | `debug ip ospf adj` or `debug ip ospf hello` | Debug confirms timer, area, authentication, or hello mismatch |
| 30 | Stop debug after test | Lab router | `undebug all` | Debugging is stopped |
| 31 | Confirm LSDB presence for the affected area | Affected routers | `show ip ospf database` | Expected LSA sections appear |
| 32 | Check Type 1 router LSAs for intra-area prefixes | Affected routers | `show ip ospf database router` | Source router LSA exists in the local area |
| 33 | Check Type 2 network LSAs for broadcast segments | Affected routers | `show ip ospf database network` | DR-originated network LSA exists where expected |
| 34 | Check Type 3 LSAs for interarea routes | Affected routers | `show ip ospf database summary <prefix>` | Summary LSA exists if route should be `O IA` |
| 35 | Check Type 4 LSAs for external routes from another area | Affected routers | `show ip ospf database asbr-summary` | ASBR reachability LSA exists when ASBR is remote |
| 36 | Check Type 5 LSAs for external routes | Affected routers | `show ip ospf database external <prefix>` | External LSA exists if route should be `O E1` or `O E2` |
| 37 | Check Type 7 LSAs for NSSA externals | NSSA routers | `show ip ospf database nssa-external <prefix>` | NSSA external LSA exists if route should be `O N1` or `O N2` |
| 38 | Confirm the route in the OSPF RIB view | Affected routers | `show ip route ospf` | Expected OSPF route is installed or confirmed missing |
| 39 | Check exact route source | Affected routers | `show ip route <prefix>` | Installed route source, AD, metric, next hop, and interface are visible |
| 40 | Determine if a better route source beat OSPF | Affected routers | `show ip route <prefix>` | Connected, static, EIGRP, BGP, or another source may beat OSPF |
| 41 | Check local OSPF route filtering | Affected routers | `show ip protocols` | Inbound distribute list is visible if applied |
| 42 | Verify prefix-list, ACL, or route-map filter | Affected routers | `show ip prefix-list`, `show access-lists`, `show route-map` | Filter logic is confirmed |
| 43 | Check ABR Type 3 filtering | ABRs | `show running-config | section router ospf` | `area <area-id> filter-list prefix <name> in/out` is identified if present |
| 44 | Check area summarization | ABRs | `show ip ospf` and `show ip route | include Null0` | Area range and Null0 summary route are confirmed |
| 45 | Check external summarization or suppression | ASBRs | `show running-config | section router ospf` | `summary-address` or `not-advertise` is identified |
| 46 | Check stub or NSSA suppression behavior | ABRs and area routers | `show ip ospf` | Area type explains missing Type 3, Type 5, or Type 7 LSAs |
| 47 | Check DR and BDR placement on multiaccess links | Affected routers | `show ip ospf interface <interface-id>` | Correct DR and BDR are elected for the topology |
| 48 | Check virtual links for discontiguous backbone issues | ABRs | `show ip ospf virtual-links` | Virtual link is present and `FULL` only if needed |
| 49 | Check default route behavior | Affected routers | `show ip route 0.0.0.0` | Default route exists or is missing as expected |
| 50 | Verify forwarding next hop | Affected routers | `show ip route <prefix>` | Next hop and outgoing interface are correct |
| 51 | Verify end-to-end reachability | Test router | `ping <destination-ip>` | Destination responds |
| 52 | Verify actual path | Test router | `traceroute <destination-ip>` | Path follows expected OSPF route |
| 53 | Verify reverse path | Destination-side routers | `show ip route <source-prefix>` | Return route exists and points correctly |
| 54 | Apply the smallest corrective change | Faulting router | `<specific fix command>` | Only the proven fault is changed |
| 55 | Reverify adjacency, LSDB, RIB, and forwarding | Affected routers | `show ip ospf neighbor`, `show ip ospf database`, `show ip route <prefix>`, `ping`, `traceroute` | OSPF control plane and data plane are both restored |
| 56 | Save only after successful validation | Changed routers | `write memory` | Known-good fix is saved |
# OSPF_Troubleshooting_Workflow_Skeleton
! =========================================================
! Phase 1: Underlay and interface baseline
! =========================================================
show ip interface brief
show running-config interface <INTERFACE_ID>
show interface <INTERFACE_ID>
ping <DIRECT_NEIGHBOR_IP>
show cdp neighbors
show lldp neighbors
! =========================================================
! Phase 2: OSPF process and interface participation
! =========================================================
show ip protocols
show running-config | section router ospf
show running-config interface <INTERFACE_ID>
show ip ospf
show ip ospf interface brief
show ip ospf interface <INTERFACE_ID>
! =========================================================
! Phase 3: Neighbor state triage
! =========================================================
show ip ospf neighbor
show ip ospf neighbor detail
show logging | include OSPF|DUP|AUTH|mismatch|ADJCHG
! =========================================================
! Phase 4: Neighbor parameter spot-check
! Compare both sides of the link.
! =========================================================
show ip ospf interface <INTERFACE_ID> | include Area|Network Type|Timer|Cost|Priority|authentication
show running-config interface <INTERFACE_ID>
show ip protocols
show ip interface <INTERFACE_ID>
show access-lists
! =========================================================
! Phase 5: LSDB investigation
! =========================================================
show ip ospf database
show ip ospf database router
show ip ospf database network
show ip ospf database summary
show ip ospf database summary <PREFIX>
show ip ospf database asbr-summary
show ip ospf database external
show ip ospf database external <PREFIX>
show ip ospf database nssa-external
show ip ospf database nssa-external <PREFIX>
! =========================================================
! Phase 6: RIB and route-selection investigation
! =========================================================
show ip route ospf
show ip route <PREFIX>
show ip route 0.0.0.0
show ip protocols
show ip prefix-list
show access-lists
show route-map
! =========================================================
! Phase 7: Forwarding validation
! =========================================================
ping <DESTINATION_IP>
traceroute <DESTINATION_IP>
show ip route <DESTINATION_PREFIX>
show ip cef <DESTINATION_IP> detail
! =========================================================
! Common fixes
! Apply only after the failing condition is proven.
! =========================================================
! Enable OSPF on an interface:
configure terminal
interface <INTERFACE_ID>
 ip ospf <PROCESS_ID> area <AREA_ID>
end
! Correct area:
configure terminal
interface <INTERFACE_ID>
 no ip ospf <PROCESS_ID> area <WRONG_AREA_ID>
 ip ospf <PROCESS_ID> area <CORRECT_AREA_ID>
end
! Remove accidental passive state:
configure terminal
router ospf <PROCESS_ID>
 no passive-interface <INTERFACE_ID>
end
! Match network type:
configure terminal
interface <INTERFACE_ID>
 ip ospf network <broadcast|point-to-point|point-to-multipoint|non-broadcast>
end
! Match timers:
configure terminal
interface <INTERFACE_ID>
 ip ospf hello-interval <SECONDS>
 ip ospf dead-interval <SECONDS>
end
! Match MTU:
configure terminal
interface <INTERFACE_ID>
 mtu <BYTES>
end
! OSPF MTU ignore for lab-specific exception:
configure terminal
interface <INTERFACE_ID>
 ip ospf mtu-ignore
end
! Set unique router ID:
configure terminal
router ospf <PROCESS_ID>
 router-id <UNIQUE_ROUTER_ID>
end
clear ip ospf process
! Correct DR priority:
configure terminal
interface <INTERFACE_ID>
 ip ospf priority <0-255>
end
! Stop debug:
undebug all
# OSPF_Troubleshooting_Workflow_Verification_Commands
show ip interface brief
show interface <interface-id>
show running-config interface <interface-id>
show running-config | section router ospf
show ip protocols
show ip ospf
show ip ospf interface brief
show ip ospf interface <interface-id>
show ip ospf neighbor
show ip ospf neighbor detail
show ip ospf database
show ip ospf database router
show ip ospf database network
show ip ospf database summary
show ip ospf database summary <prefix>
show ip ospf database asbr-summary
show ip ospf database external
show ip ospf database external <prefix>
show ip ospf database nssa-external
show ip ospf database nssa-external <prefix>
show ip route ospf
show ip route <prefix>
show ip route 0.0.0.0
show ip cef <destination-ip> detail
show ip interface <interface-id>
show access-lists
show ip prefix-list
show route-map
show cdp neighbors
show lldp neighbors
show ip ospf virtual-links
show logging | include OSPF|DUP|AUTH|mismatch|ADJCHG|SPF|LSA
debug ip ospf adj
debug ip ospf hello
debug ip ospf events
show debugging
undebug all
ping <neighbor-ip>
ping <destination-ip>
traceroute <destination-ip>
# OSPF_Troubleshooting_Workflow_Rollback
! =========================================================
! Remove interface-level OSPF
! =========================================================
configure terminal
interface <INTERFACE_ID>
 no ip ospf <PROCESS_ID> area <AREA_ID>
end
write memory
! =========================================================
! Remove a mistaken network statement
! =========================================================
configure terminal
router ospf <PROCESS_ID>
 no network <IP_ADDRESS> <WILDCARD_MASK> area <AREA_ID>
end
write memory
! =========================================================
! Restore passive state
! =========================================================
configure terminal
router ospf <PROCESS_ID>
 passive-interface <INTERFACE_ID>
end
write memory
! =========================================================
! Remove passive-by-default
! =========================================================
configure terminal
router ospf <PROCESS_ID>
 no passive-interface default
end
write memory
! =========================================================
! Remove manual OSPF network type
! =========================================================
configure terminal
interface <INTERFACE_ID>
 no ip ospf network
end
write memory
! =========================================================
! Remove manual OSPF timers
! =========================================================
configure terminal
interface <INTERFACE_ID>
 no ip ospf hello-interval
 no ip ospf dead-interval
end
write memory
! =========================================================
! Remove OSPF MTU ignore
! =========================================================
configure terminal
interface <INTERFACE_ID>
 no ip ospf mtu-ignore
end
write memory
! =========================================================
! Remove manual OSPF cost
! =========================================================
configure terminal
interface <INTERFACE_ID>
 no ip ospf cost
end
write memory
! =========================================================
! Remove manual OSPF priority
! =========================================================
configure terminal
interface <INTERFACE_ID>
 no ip ospf priority
end
write memory
! =========================================================
! Remove router ID
! Requires OSPF process clear to take effect.
! =========================================================
configure terminal
router ospf <PROCESS_ID>
 no router-id <ROUTER_ID>
end
clear ip ospf process
write memory
! =========================================================
! Remove inbound OSPF distribute list
! =========================================================
configure terminal
router ospf <PROCESS_ID>
 no distribute-list prefix <PREFIX_LIST_NAME> in
end
write memory
! =========================================================
! Remove Type 3 ABR filter
! =========================================================
configure terminal
router ospf <PROCESS_ID>
 no area <AREA_ID> filter-list prefix <PREFIX_LIST_NAME> in
 no area <AREA_ID> filter-list prefix <PREFIX_LIST_NAME> out
end
write memory
! =========================================================
! Remove area range summary
! =========================================================
configure terminal
router ospf <PROCESS_ID>
 no area <AREA_ID> range <SUMMARY_PREFIX> <SUMMARY_MASK>
end
write memory
! =========================================================
! Remove virtual link
! =========================================================
configure terminal
router ospf <PROCESS_ID>
 no area <TRANSIT_AREA_ID> virtual-link <REMOTE_ROUTER_ID>
end
write memory
! =========================================================
! Stop all debugs
! =========================================================
undebug all
! =========================================================
! Full OSPF process reset for lab only
! =========================================================
configure terminal
no router ospf <PROCESS_ID>
end
write memory
# OSPF_Troubleshooting_Workflow_Failure_Checks
| Symptom | Likely Cause | Check | Corrective Action |
|---|---|---|---|
| Interface is missing from OSPF | OSPF not enabled on the interface | `show ip ospf interface brief` | Add correct `network` statement or `ip ospf <process-id> area <area-id>` |
| Interface is in wrong area | Interface-level command overrides network statement | `show running-config interface <interface-id>` | Correct interface-level `ip ospf` area |
| Neighbor missing completely | Interface down or wrong IP/subnet | `show ip interface brief`, `show running-config interface <interface-id>` | Fix interface state, IP address, or mask |
| Neighbor missing completely | OSPF hello blocked by passive interface | `show ip protocols` | Configure `no passive-interface <interface-id>` |
| Neighbor missing completely | ACL blocks OSPF protocol 89 or multicast | `show ip interface <interface-id>`, `show access-lists` | Permit OSPF and 224.0.0.5/224.0.0.6 as required |
| Neighbor stuck in INIT | One-way hello reachability | `debug ip ospf hello`, `show ip ospf neighbor` | Fix ACL, multicast, L2 reachability, or return path |
| Neighbor stuck in 2-WAY | DROTHER-to-DROTHER state on broadcast network | `show ip ospf neighbor`, `show ip ospf interface <interface-id>` | No fix needed if DR/BDR adjacency exists |
| Neighbor stuck in 2-WAY when FULL expected | Wrong network type or DR/BDR behavior | `show ip ospf interface <interface-id>` | Correct OSPF network type or DR/BDR priority |
| Neighbor stuck in EXSTART | MTU mismatch or duplicate router ID | `show interface <interface-id>`, `show ip protocols` | Match MTU or configure unique router IDs |
| Neighbor stuck in EXCHANGE | MTU mismatch or DBD exchange problem | `show ip ospf neighbor detail`, `show interface <interface-id>` | Match MTU or use `ip ospf mtu-ignore` only when justified |
| Neighbor stuck in LOADING | Missing LSA exchange, corruption, or resource issue | `show ip ospf neighbor detail`, `show logging` | Check MTU, memory, LSA retransmissions, and neighbor stability |
| Neighbor fails after config change | Area mismatch | `show ip ospf interface <interface-id>` | Put both ends in the same area |
| Neighbor fails after config change | Hello/dead timer mismatch | `show ip ospf interface <interface-id> | include Timer` | Match timers on both ends |
| Neighbor fails after config change | Area type mismatch | `show ip ospf` | Match stub, NSSA, or normal area type |
| Neighbor fails after config change | Authentication mismatch | `show ip ospf interface <interface-id>`, `show running-config interface <interface-id>` | Match authentication type, key ID, and key string |
| Duplicate router ID log appears | Same OSPF RID used on multiple routers | `show logging | include DUP`, `show ip protocols` | Configure unique `router-id` and clear OSPF process |
| Routes missing but neighbor is FULL | Prefix interface is not OSPF-enabled | `show ip ospf interface brief`, `show ip protocols` | Enable OSPF on the source interface |
| Route missing from LSDB | Source prefix is not being originated | `show ip ospf database router <source-rid>` | Enable OSPF on the source interface or fix source route |
| Route present in LSDB but absent from RIB | Better route source wins | `show ip route <prefix>` | Check connected, static, EIGRP, BGP, or AD differences |
| Route present in LSDB but absent from RIB | OSPF distribute list blocks local install | `show ip protocols`, `show ip prefix-list` | Remove or correct inbound distribute list |
| Interarea route missing | Type 3 LSA not generated or blocked | `show ip ospf database summary <prefix>` | Check ABR area placement, Type 3 filtering, and summarization |
| Interarea routes missing broadly | Area 0 is disconnected | `show ip ospf`, `show ip ospf virtual-links` | Restore contiguous Area 0 or configure virtual link as temporary repair |
| External route missing | Redistribution not configured or source route missing | `show ip route <external-prefix>`, `show running-config | section router ospf` | Restore source route and redistribution policy |
| External route missing in stub area | Stub area blocks Type 5 LSAs | `show ip ospf`, `show ip ospf database external` | Use default route, NSSA, or normal area depending on design |
| NSSA external missing | Type 7 not originated or not translated | `show ip ospf database nssa-external`, `show ip ospf database external` | Check NSSA redistribution and ABR translation |
| Route appears as default only | Totally stub or totally NSSA suppresses details | `show ip ospf`, `show ip route 0.0.0.0` | Confirm area type is intentional |
| Specifics hidden by summary | ABR area range is active | `show ip ospf`, `show ip route | include Null0` | Verify summary range and component routes |
| Traffic to summarized unused subnet drops | Summary Null0 route is working | `show ip route <unused-prefix>` | Use narrower summary if blackhole is unacceptable |
| Wrong DR elected | Priority left to default or wrong router ID wins | `show ip ospf interface <interface-id>` | Set `ip ospf priority` and reset election in lab window |
| Routes missing on hub-and-spoke NBMA | Spoke became DR or DR unreachable | `show ip ospf interface <interface-id>` | Make hub DR and spokes priority 0 |
| Route path is unexpected | OSPF route class beats metric | `show ip route <prefix>` | Compare `O`, `O IA`, `O E1`, `O E2`, `O N1`, and `O N2` |
| Lower metric route loses | Route class or AD is more important | `show ip route <prefix>` | Fix route source or area design |
| Route exists but ping fails | Return path missing | `show ip route <source-prefix>` on return side | Restore reverse route |
| Route exists but ping fails | ACL, NAT, firewall, or data-plane issue | `show access-lists`, `traceroute <destination>` | Troubleshoot forwarding path after OSPF control plane is proven |
| Traceroute does not match expectation | Another router makes a different local decision | `show ip route <prefix>` on each hop | Verify RIB hop by hop |
| Debug output is noisy | Debug used too early | `show debugging` | Stop debug with `undebug all` and return to show-command workflow |
| Fix did not persist after reload | Configuration was not saved | `show startup-config | section router ospf` | Save with `write memory` after validation |
##### Source_Basis
# OSPF_Troubleshooting_Workflow_Mental_Model
# OSPF_Troubleshooting_Workflow_Configuration_Checklist
# OSPF_Troubleshooting_Workflow_Skeleton
# OSPF_Troubleshooting_Workflow_Verification_Commands
# OSPF_Troubleshooting_Workflow_Rollback
# OSPF_Troubleshooting_Workflow_Failure_Checks
# index of each title throughout note, not in table format

