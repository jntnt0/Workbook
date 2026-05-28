OSPFv2_Baseline_Adjacency_And_LSDB.md

OSPFv2_Baseline_Adjacency_And_LSDB

# Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| `_KB_INDEX.md` | `All_combined_part3.md` Book 6, Chapter 8 OSPF | Points OSPF area, LSA, adjacency, authentication, and OSPFv3 material to the main project source |
| `All_combined_part3.md` | Chapter 8, Lab 8-1: Advertising Networks | Provides baseline OSPFv2 syntax: `router ospf`, `router-id`, host-specific `network` statements, loopback advertisement, adjacency confirmation, and OSPF route verification |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | Chapter 6, OSPF Configuration | Confirms that `network` statements select OSPF-enabled interfaces, process IDs are locally significant, interface OSPF enablement is valid, and passive interfaces advertise without forming adjacencies |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | Chapter 6, Confirmation of Interfaces / Neighbor Adjacencies / Installed Routes | Provides verification commands: `show ip ospf interface`, `show ip ospf interface brief`, `show ip ospf neighbor`, and `show ip route ospf` |
| Lab inventory | `ospf-two-routers-single-link-final`, `ospf-two-routers-final`, `ospf-four-routers-final`, `ospf-single-area-final` | Related baseline labs for OSPFv2 adjacency formation, LSDB synchronization, and intra-area route installation |
# OSPFv2_Baseline_Adjacency_And_LSDB_Mental_Model
| Concept | Operational Meaning |
|---|---|
| OSPF process | `router ospf <process-id>` starts the local OSPF process. The process ID is locally significant and does not need to match neighbors |
| Router ID | The router ID uniquely identifies the router in the OSPF domain. Set it manually to avoid unexpected election from loopbacks or physical interfaces |
| Network statement | The OSPF `network` statement does not directly “advertise a network.” It selects interfaces whose primary IPv4 address matches the wildcard mask, then places those interfaces into an OSPF area |
| Interface-based enablement | `ip ospf <process-id> area <area-id>` enables OSPF directly under an interface and gives explicit control when avoiding broad network statements |
| Area membership | OSPF area membership is interface-based. Two neighbors must agree on area ID before they can form adjacency |
| Hello process | OSPF neighbors discover each other with hello packets. Timers, area, subnet, authentication, MTU, and network type must be compatible |
| Adjacency | A working OSPF adjacency reaches `FULL` state. On broadcast segments the state also shows DR/BDR/DROTHER role; on point-to-point links it shows `FULL/-` |
| LSDB | The LSDB is the topology database for the area. Routers in the same area should have a synchronized view of the area topology |
| SPF | Each router runs SPF from its own perspective using the LSDB, then installs best routes into the routing table |
| Intra-area route | Routes learned inside the same area appear as `O` routes in the RIB |
| Loopback advertisement | Loopbacks may appear as /32 host routes unless the interface is changed with `ip ospf network point-to-point` when the real mask must be advertised |
| Passive interface | A passive interface is advertised into OSPF but does not send or process OSPF hellos, so it cannot form neighbors |
# OSPFv2_Baseline_Adjacency_And_LSDB_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm routed interfaces exist and are up | Each OSPF router | `show ip interface brief` | Interfaces that will run OSPF are `up/up` and have the expected IPv4 addresses |
| 2 | Confirm basic directly connected reachability before enabling OSPF | Each OSPF router | `ping <directly-connected-neighbor-ip>` | Direct neighbor interface IP responds |
| 3 | Enter global configuration mode | Each OSPF router | `configure terminal` | Router enters configuration mode |
| 4 | Create or confirm loopback interface for stable router ID | Each OSPF router | `interface Loopback0` | Loopback interface context opens |
| 5 | Assign loopback IPv4 address | Each OSPF router | `ip address <router-id-like-ip> <mask>` | Loopback has a stable IPv4 address |
| 6 | Force loopback to advertise with its real mask when required | Each OSPF router | `ip ospf network point-to-point` | Loopback prefix is advertised with configured mask instead of default host-style behavior |
| 7 | Return to global configuration mode | Each OSPF router | `exit` | Router returns to global config mode |
| 8 | Start the OSPFv2 process | Each OSPF router | `router ospf <process-id>` | OSPF process is created locally |
| 9 | Set deterministic OSPF router ID | Each OSPF router | `router-id <router-id>` | Router has a unique OSPF router ID |
| 10 | Enable OSPF on the transit interface with a host-specific network statement | Each OSPF router | `network <interface-ip> 0.0.0.0 area <area-id>` | OSPF is enabled only on the intended transit interface |
| 11 | Enable OSPF on the loopback with a host-specific network statement | Each OSPF router | `network <loopback-ip> 0.0.0.0 area <area-id>` | Loopback is injected into the OSPF area |
| 12 | Make non-neighbor-facing interfaces passive if they only need advertisement | Each OSPF router | `passive-interface <interface-id>` | Interface prefix remains in OSPF, but no neighbor forms on that interface |
| 13 | Optionally use passive-by-default for cleaner control | Each OSPF router | `passive-interface default` then `no passive-interface <neighbor-facing-interface>` | Only approved neighbor-facing interfaces send and receive OSPF hellos |
| 14 | Save the configuration | Each OSPF router | `end` then `write memory` | Running configuration is saved |
| 15 | Confirm OSPF is enabled on the intended interfaces | Each OSPF router | `show ip ospf interface brief` | Correct interfaces appear with expected process ID, area, cost, state, and neighbor count |
| 16 | Confirm detailed interface parameters | Each OSPF router | `show ip ospf interface <interface-id>` | Area, network type, timers, cost, and neighbor count match expectations |
| 17 | Confirm neighbor adjacency | Each OSPF router | `show ip ospf neighbor` | Neighbor state reaches `FULL` |
| 18 | Confirm LSDB exists for the area | Each OSPF router | `show ip ospf database` | Router, network, summary, or external LSA sections appear as expected for the topology |
| 19 | Confirm OSPF routes installed in RIB | Each OSPF router | `show ip route ospf` | Intra-area OSPF routes appear as `O` with AD 110 and expected metric |
| 20 | Confirm end-to-end reachability across learned routes | Each OSPF router | `ping <remote-loopback-or-remote-prefix>` | ICMP succeeds across the OSPF-learned path |
# OSPFv2_Baseline_Adjacency_And_LSDB_Skeleton
! =========================================================
! OSPFv2 baseline using network statements
! =========================================================
configure terminal
interface Loopback0
 ip address <LOOPBACK_IP> <LOOPBACK_MASK>
 ip ospf network point-to-point
exit
interface <TRANSIT_INTERFACE>
 ip address <TRANSIT_IP> <TRANSIT_MASK>
 no shutdown
exit
router ospf <PROCESS_ID>
 router-id <ROUTER_ID>
 network <TRANSIT_INTERFACE_IP> 0.0.0.0 area <AREA_ID>
 network <LOOPBACK_IP> 0.0.0.0 area <AREA_ID>
!
! Optional safer interface policy:
! passive-interface default
! no passive-interface <TRANSIT_INTERFACE>
end
write memory
! =========================================================
! OSPFv2 baseline using interface-specific enablement
! =========================================================
configure terminal
interface Loopback0
 ip address <LOOPBACK_IP> <LOOPBACK_MASK>
 ip ospf network point-to-point
 ip ospf <PROCESS_ID> area <AREA_ID>
exit
interface <TRANSIT_INTERFACE>
 ip address <TRANSIT_IP> <TRANSIT_MASK>
 ip ospf <PROCESS_ID> area <AREA_ID>
 no shutdown
exit
router ospf <PROCESS_ID>
 router-id <ROUTER_ID>
!
! Optional:
! passive-interface Loopback0
end
write memory
# OSPFv2_Baseline_Adjacency_And_LSDB_Verification_Commands
show ip interface brief
show running-config | section router ospf
show running-config interface <interface-id>
show ip protocols
show ip ospf
show ip ospf interface brief
show ip ospf interface <interface-id>
show ip ospf neighbor
show ip ospf neighbor detail
show ip ospf database
show ip ospf database router
show ip route ospf
show ip route <remote-prefix>
ping <neighbor-interface-ip>
ping <remote-loopback-ip>
traceroute <remote-loopback-ip>
# OSPFv2_Baseline_Adjacency_And_LSDB_Rollback
! Remove OSPF completely from the router
configure terminal
no router ospf <PROCESS_ID>
end
write memory
! Remove interface-specific OSPF only
configure terminal
interface <INTERFACE_ID>
 no ip ospf <PROCESS_ID> area <AREA_ID>
exit
end
write memory
! Remove one network statement only
configure terminal
router ospf <PROCESS_ID>
 no network <INTERFACE_IP_OR_LOOPBACK_IP> <WILDCARD_MASK> area <AREA_ID>
end
write memory
! Remove passive-interface setting
configure terminal
router ospf <PROCESS_ID>
 no passive-interface <INTERFACE_ID>
end
write memory
! Remove passive-by-default model
configure terminal
router ospf <PROCESS_ID>
 no passive-interface default
end
write memory
# OSPFv2_Baseline_Adjacency_And_LSDB_Failure_Checks
| Symptom | Likely Cause | Check | Corrective Action |
|---|---|---|---|
| No OSPF neighbor appears | OSPF not enabled on the intended interface | `show ip ospf interface brief` | Fix `network` wildcard or apply `ip ospf <process-id> area <area-id>` under the interface |
| Neighbor stuck in INIT or no return hello | One-way hello issue, ACL, multicast problem, or wrong interface | `show ip ospf neighbor`, `show ip interface <interface-id>` | Confirm both sides send/receive OSPF protocol 89 and multicast 224.0.0.5 |
| Neighbor does not form | Area mismatch | `show ip ospf interface <interface-id>` | Put both ends of the link in the same area |
| Neighbor does not form | Hello/dead timer mismatch | `show ip ospf interface <interface-id> | include Timer` | Match hello/dead timers or remove custom timer settings |
| Neighbor stuck in EXSTART/EXCHANGE | MTU mismatch | `show interface <interface-id>`, `show ip ospf neighbor detail` | Fix MTU mismatch or use `ip ospf mtu-ignore` only when appropriate |
| Interface advertised but no neighbor forms | Interface is passive | `show ip protocols` | Remove passive state with `no passive-interface <interface-id>` |
| Unexpected OSPF router ID | Router ID was auto-selected | `show ip ospf | include ID` | Configure `router-id <router-id>`, then restart OSPF process if required |
| OSPF enabled on too many interfaces | Broad wildcard statement matched unintended interfaces | `show ip ospf interface brief` | Replace broad `network` statement with host-specific statements or interface-specific OSPF |
| Loopback appears as /32 unexpectedly | Loopback default OSPF behavior | `show ip route ospf`, `show ip ospf database router` | Configure `ip ospf network point-to-point` under the loopback if the real mask must be advertised |
| Routes missing from RIB | LSDB has route but better route exists, route type issue, or no SPF result | `show ip ospf database`, `show ip route <prefix>` | Compare LSDB, OSPF route, connected/static routes, and administrative distance |
| LSDB not synchronized | Adjacency not FULL or LSA exchange failing | `show ip ospf neighbor`, `show ip ospf database` | Fix adjacency first, then recheck database and routes |
| Ping fails despite OSPF routes | Return path missing or wrong next-hop | `show ip route <destination>`, `traceroute <destination>` | Verify both forward and reverse OSPF routes |
##### Source_Basis
# OSPFv2_Baseline_Adjacency_And_LSDB_Mental_Model
# OSPFv2_Baseline_Adjacency_And_LSDB_Configuration_Checklist
# OSPFv2_Baseline_Adjacency_And_LSDB_Skeleton
# OSPFv2_Baseline_Adjacency_And_LSDB_Verification_Commands
# OSPFv2_Baseline_Adjacency_And_LSDB_Rollback
# OSPFv2_Baseline_Adjacency_And_LSDB_Failure_Checks
# index of each title throughout note, not in table format

