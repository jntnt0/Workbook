OSPF_DR_BDR_Broadcast_Multiaccess.md

OSPF_DR_BDR_Broadcast_Multiaccess

# Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| `_KB_INDEX.md` | OSPF topic map | Points enterprise OSPF to `All_combined_part3.md` Book 6 as the main OSPF source |
| `All_combined_part3.md` | Book 6, Chapter 8 OSPF, Broadcast Networks | Supports OSPF broadcast network behavior, DR/BDR election, neighbor states, and network LSA verification |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | Chapter 6, The Designated Router and Backup Designated Router | Supports DR/BDR purpose, pseudonode behavior, election timing, priority, router ID tie-break, and non-preemption |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | Chapter 6, OSPF Network Types | Supports Ethernet default broadcast network type, `ip ospf network broadcast`, hello/dead timer defaults, and DR requirement |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | Chapter 8, Wrong Designated Router Elected | Supports failure checks for wrong DR placement, priority 0, network-type mismatch, and DR election troubleshooting |
| Related labs | `ospf-dr-bdr-final`, `ospf-dr-bdr-election-final` | Main labs for OSPF broadcast multiaccess adjacency, DR/BDR election, interface priority, and Type 2 network LSA behavior |
# OSPF_DR_BDR_Broadcast_Multiaccess_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Broadcast multiaccess segment | A shared Layer 2 segment where more than two OSPF routers may exist on the same subnet |
| Ethernet default | Ethernet interfaces default to OSPF broadcast network type |
| DR purpose | The designated router reduces full-mesh LSA flooding on a multiaccess segment |
| BDR purpose | The backup designated router maintains full adjacencies so it can take over quickly if the DR fails |
| Pseudonode | The DR represents the shared segment in the LSDB by originating a Type 2 network LSA |
| DROTHER | A router that is neither DR nor BDR on the segment |
| Neighbor state with DR/BDR | DROTHER routers form `FULL` adjacencies with the DR and BDR |
| Neighbor state between DROTHER routers | DROTHER-to-DROTHER neighbors normally remain in `2WAY`, not `FULL` |
| Election priority | Higher interface priority wins DR/BDR election |
| Priority 0 | `ip ospf priority 0` prevents that interface from becoming DR or BDR |
| Router ID tie-break | If priority ties, highest OSPF router ID wins |
| Non-preemption | A better priority or higher router ID does not replace the current DR automatically after election |
| Election reset | A DR/BDR role usually changes only after interface flap, OSPF process reset, or current DR/BDR failure |
| Broadcast timers | Broadcast OSPF defaults to 10 second hello and 40 second dead interval |
| Type 2 LSA | The DR originates the Type 2 network LSA for a multiaccess segment with at least two attached routers |
| Bad design pattern | Leaving DR election to chance can put a weak or edge router in the DR role |
# OSPF_DR_BDR_Broadcast_Multiaccess_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm the shared subnet interfaces are up | All routers on segment | `show ip interface brief` | All OSPF broadcast interfaces are `up/up` in the same IPv4 subnet |
| 2 | Confirm Layer 2 adjacency on the shared segment | All routers on segment | `show cdp neighbors` or `show lldp neighbors` | Expected routers are visible on the shared segment |
| 3 | Confirm direct IP reachability across the shared segment | All routers on segment | `ping <neighbor-interface-ip>` | Other routers on the shared subnet respond |
| 4 | Check current OSPF interface state before changes | All routers on segment | `show ip ospf interface <interface-id>` | Current network type, priority, DR, BDR, timers, and cost are visible |
| 5 | Check current OSPF neighbor state before changes | All routers on segment | `show ip ospf neighbor` | Existing DR, BDR, DROTHER, `FULL`, and `2WAY` states are known |
| 6 | Enter configuration mode | All routers on segment | `configure terminal` | Router enters global configuration mode |
| 7 | Configure stable router ID | All routers on segment | `router ospf <process-id>` then `router-id <router-id>` | Each router has a unique deterministic OSPF router ID |
| 8 | Exit to global configuration mode | All routers on segment | `exit` | Router returns to global configuration mode |
| 9 | Enter the shared broadcast interface | All routers on segment | `interface <broadcast-interface>` | Interface configuration mode opens |
| 10 | Enable OSPF on the broadcast interface | All routers on segment | `ip ospf <process-id> area <area-id>` | Interface participates in the intended OSPF area |
| 11 | Force broadcast network type if the lab requires explicit broadcast behavior | All routers on segment | `ip ospf network broadcast` | Interface uses OSPF broadcast behavior |
| 12 | Set the intended DR router with highest priority | Intended DR | `ip ospf priority <high-priority>` | Intended DR has the highest OSPF interface priority on the segment |
| 13 | Set the intended BDR router with second-highest priority | Intended BDR | `ip ospf priority <medium-priority>` | Intended BDR has the second-highest OSPF interface priority |
| 14 | Prevent non-DR routers from becoming DR/BDR if required | DROTHER routers | `ip ospf priority 0` | Router is ineligible for DR/BDR election |
| 15 | Confirm interface is not administratively down | All routers on segment | `no shutdown` | Interface remains active |
| 16 | Return to privileged EXEC mode | All routers on segment | `end` | Router exits configuration mode |
| 17 | Save the configuration | All routers on segment | `write memory` | Configuration is saved |
| 18 | Reset election only if changing DR/BDR placement is required | All routers on segment, during lab window | `clear ip ospf process` | OSPF adjacencies reset and DR/BDR election reruns |
| 19 | Verify network type is broadcast | All routers on segment | `show ip ospf interface <interface-id> | include Network Type` | Output shows `Network Type BROADCAST` |
| 20 | Verify interface priority | All routers on segment | `show ip ospf interface <interface-id> | include Priority` | Priority matches intended DR/BDR plan |
| 21 | Verify DR and BDR role placement | All routers on segment | `show ip ospf interface <interface-id> | include Designated|Backup` | Intended router is DR and intended backup is BDR |
| 22 | Verify neighbor state from DR | DR | `show ip ospf neighbor` | DR sees other routers as `FULL/BDR` or `FULL/DROTHER` |
| 23 | Verify neighbor state from BDR | BDR | `show ip ospf neighbor` | BDR sees DR and DROTHER routers as full adjacencies where expected |
| 24 | Verify neighbor state from DROTHER | DROTHER routers | `show ip ospf neighbor` | DROTHER shows DR and BDR as `FULL`; other DROTHER routers may show `2WAY` |
| 25 | Verify Type 2 network LSA exists | All routers in area | `show ip ospf database network` | Network LSA appears for the broadcast segment and is originated by the DR |
| 26 | Verify DR router LSA and network LSA relationship | All routers in area | `show ip ospf database router` and `show ip ospf database network` | LSDB represents the shared segment through the DR pseudonode |
| 27 | Verify OSPF route installation | All routers in area | `show ip route ospf` | Expected OSPF routes are installed |
| 28 | Verify end-to-end forwarding | All routers in area | `ping <remote-loopback-ip>` | Remote OSPF-learned destinations respond |
| 29 | Verify data path | All routers in area | `traceroute <remote-loopback-ip>` | Path follows expected OSPF next hop |
# OSPF_DR_BDR_Broadcast_Multiaccess_Skeleton
! =========================================================
! Intended DR router
! =========================================================
configure terminal
router ospf <PROCESS_ID>
 router-id <DR_ROUTER_ID>
exit
interface <BROADCAST_INTERFACE>
 description OSPF_BROADCAST_MULTIACCESS_SEGMENT
 ip address <DR_INTERFACE_IP> <MASK>
 ip ospf <PROCESS_ID> area <AREA_ID>
 ip ospf network broadcast
 ip ospf priority <DR_PRIORITY>
 no shutdown
exit
end
write memory
! =========================================================
! Intended BDR router
! =========================================================
configure terminal
router ospf <PROCESS_ID>
 router-id <BDR_ROUTER_ID>
exit
interface <BROADCAST_INTERFACE>
 description OSPF_BROADCAST_MULTIACCESS_SEGMENT
 ip address <BDR_INTERFACE_IP> <MASK>
 ip ospf <PROCESS_ID> area <AREA_ID>
 ip ospf network broadcast
 ip ospf priority <BDR_PRIORITY>
 no shutdown
exit
end
write memory
! =========================================================
! DROTHER router
! Use priority 0 when the router must never become DR/BDR.
! =========================================================
configure terminal
router ospf <PROCESS_ID>
 router-id <DROTHER_ROUTER_ID>
exit
interface <BROADCAST_INTERFACE>
 description OSPF_BROADCAST_MULTIACCESS_SEGMENT
 ip address <DROTHER_INTERFACE_IP> <MASK>
 ip ospf <PROCESS_ID> area <AREA_ID>
 ip ospf network broadcast
 ip ospf priority 0
 no shutdown
exit
end
write memory
! =========================================================
! Election reset for lab use only
! This disrupts OSPF adjacencies.
! =========================================================
clear ip ospf process
# OSPF_DR_BDR_Broadcast_Multiaccess_Verification_Commands
show ip interface brief
show running-config interface <broadcast-interface>
show running-config | section router ospf
show ip protocols
show ip ospf
show ip ospf interface brief
show ip ospf interface <broadcast-interface>
show ip ospf interface <broadcast-interface> | include Network Type|Priority|Designated|Backup|Timer
show ip ospf neighbor
show ip ospf neighbor detail
show ip ospf database
show ip ospf database router
show ip ospf database network
show ip route ospf
show ip route <remote-prefix>
ping <neighbor-interface-ip>
ping <remote-loopback-ip>
traceroute <remote-loopback-ip>
# OSPF_DR_BDR_Broadcast_Multiaccess_Rollback
! =========================================================
! Remove custom OSPF priority
! Returns interface priority to default behavior.
! =========================================================
configure terminal
interface <BROADCAST_INTERFACE>
 no ip ospf priority
end
write memory
! =========================================================
! Prevent a router from becoming DR/BDR
! =========================================================
configure terminal
interface <BROADCAST_INTERFACE>
 ip ospf priority 0
end
write memory
! =========================================================
! Restore explicit broadcast network type
! =========================================================
configure terminal
interface <BROADCAST_INTERFACE>
 ip ospf network broadcast
end
write memory
! =========================================================
! Remove explicit broadcast network type
! Interface returns to default OSPF type for its media.
! Ethernet normally defaults to broadcast.
! =========================================================
configure terminal
interface <BROADCAST_INTERFACE>
 no ip ospf network broadcast
end
write memory
! =========================================================
! Remove interface-level OSPF from the broadcast segment
! =========================================================
configure terminal
interface <BROADCAST_INTERFACE>
 no ip ospf <PROCESS_ID> area <AREA_ID>
end
write memory
! =========================================================
! Remove whole OSPF process for lab reset
! =========================================================
configure terminal
no router ospf <PROCESS_ID>
end
write memory
# OSPF_DR_BDR_Broadcast_Multiaccess_Failure_Checks
| Symptom | Likely Cause | Check | Corrective Action |
|---|---|---|---|
| Wrong router became DR | Priority values were not set before election | `show ip ospf interface <interface-id> | include Priority|Designated` | Set correct `ip ospf priority`, then reset OSPF process or flap interfaces during lab window |
| Higher-priority router did not take over as DR | OSPF DR election is non-preemptive | `show ip ospf interface <interface-id>` | Reset election only if role change is required |
| Router never becomes DR or BDR | Interface priority is 0 | `show ip ospf interface <interface-id> | include Priority` | Change priority to 1 to 255 if it should participate |
| No BDR appears | Only one eligible router exists on segment | `show ip ospf neighbor`, `show ip ospf interface <interface-id>` | Ensure at least two routers have priority above 0 |
| Neighbor stuck in 2WAY with DROTHER | Normal DROTHER-to-DROTHER behavior | `show ip ospf neighbor` | No fix needed if both routers are DROTHER and have full adjacency with DR/BDR |
| Neighbor does not reach FULL with DR or BDR | Area, timer, MTU, authentication, or network type mismatch | `show ip ospf neighbor detail`, `show ip ospf interface <interface-id>` | Match area, timers, MTU, authentication, and network type |
| Network type is not broadcast | Interface configured as point-to-point or another OSPF type | `show ip ospf interface <interface-id> | include Network Type` | Configure `ip ospf network broadcast` |
| DR differs across routers on same segment | Layer 2 segmentation, duplicate subnet issue, or mismatched OSPF view | `show ip ospf interface <interface-id>` on all routers | Verify VLAN, subnet, masks, L2 reachability, and OSPF area |
| Type 2 network LSA missing | No DR elected or only one attached router is active | `show ip ospf database network` | Verify broadcast network type, neighbor formation, and DR state |
| Excess Type 2 LSAs | Too many Ethernet links left as broadcast when they are really two-router transit links | `show ip ospf database network` | Convert dedicated two-router transit links to point-to-point in a separate mechanism note |
| OSPF route missing | LSDB or adjacency issue on the broadcast segment | `show ip ospf database`, `show ip route ospf` | Fix adjacency and LSDB before troubleshooting RIB installation |
| Routes exist but traffic fails | Return path, ACL, or wrong next hop issue | `show ip route <prefix>`, `traceroute <destination>` | Verify forward path, reverse path, and filtering |
##### Source_Basis
# OSPF_DR_BDR_Broadcast_Multiaccess_Mental_Model
# OSPF_DR_BDR_Broadcast_Multiaccess_Configuration_Checklist
# OSPF_DR_BDR_Broadcast_Multiaccess_Skeleton
# OSPF_DR_BDR_Broadcast_Multiaccess_Verification_Commands
# OSPF_DR_BDR_Broadcast_Multiaccess_Rollback
# OSPF_DR_BDR_Broadcast_Multiaccess_Failure_Checks
# index of each title throughout note, not in table format

