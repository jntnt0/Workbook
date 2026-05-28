CEF_ECMP_Per_Flow_Load_Sharing.md
# CEF_ECMP_Per_Flow_Load_Sharing

# CEF_ECMP_Per_Flow_Load_Sharing_Mental_Model

| Concept | Operational Meaning |
|---|---|
| ECMP | Equal-cost multipath means the routing table has multiple equal-best next hops for the same destination prefix |
| CEF load sharing | CEF chooses one forwarding adjacency from the equal-cost set instead of punting every packet to the CPU |
| Per-flow behavior | Traffic is normally load shared by hash, so a single flow usually stays on one path |
| Not round-robin | ECMP does not mean packet 1 uses path A, packet 2 uses path B, and packet 3 uses path C |
| Hash input | Source IP, destination IP, ports, or platform-specific fields can influence which path a flow uses |
| RIB requirement | ECMP must first exist in the routing table before CEF can load share across it |
| FIB programming | `show ip cef <destination>` proves whether CEF programmed multiple adjacencies for the prefix |
| Exact route lookup | `show ip cef exact-route <source> <destination>` shows which ECMP path CEF selects for a specific flow pair |
| Flow distribution | Multiple flows should distribute across paths; one flow may use only one path |
| Polarization risk | Repeated hashing across multiple routers can accidentally force many flows onto the same path |

# CEF_ECMP_Per_Flow_Load_Sharing_Configuration_Checklist

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm Layer 3 forwarding is enabled on multilayer switches | L3SW/RTR | `show running-config | include ^ip routing` | `ip routing` is present on multilayer switches; routers route by default |
| 2 | Confirm CEF is enabled | L3SW/RTR | `show ip cef` | CEF table is displayed |
| 3 | Enable CEF if required in the lab image | L3SW/RTR | `conf t`<br>`ip cef`<br>`end` | CEF is enabled globally |
| 4 | Confirm ECMP candidate interfaces are up | L3SW/RTR | `show ip interface brief` | All transit interfaces toward equal-cost next hops are `up/up` |
| 5 | Confirm each next hop is directly reachable or recursively reachable | L3SW/RTR | `ping <NEXT_HOP_1>`<br>`ping <NEXT_HOP_2>` | Both next hops respond or are reachable through valid recursion |
| 6 | Configure equal-cost static route path 1 | L3SW/RTR | `conf t`<br>`ip route <DESTINATION_PREFIX> <MASK> <NEXT_HOP_1>`<br>`end` | First static route is installed |
| 7 | Configure equal-cost static route path 2 | L3SW/RTR | `conf t`<br>`ip route <DESTINATION_PREFIX> <MASK> <NEXT_HOP_2>`<br>`end` | Second static route is installed with the same AD and metric |
| 8 | Optional: configure additional equal-cost static paths | L3SW/RTR | `conf t`<br>`ip route <DESTINATION_PREFIX> <MASK> <NEXT_HOP_3>`<br>`ip route <DESTINATION_PREFIX> <MASK> <NEXT_HOP_4>`<br>`end` | Additional equal-cost routes are installed if platform maximum-paths allows them |
| 9 | Verify the RIB has multiple equal-cost next hops | L3SW/RTR | `show ip route <DESTINATION_PREFIX>` | Route output shows multiple next hops for the same prefix |
| 10 | Verify CEF has multiple adjacencies for the destination | L3SW/RTR | `show ip cef <DESTINATION_IP>` | CEF output shows multiple next-hop adjacencies |
| 11 | Verify CEF exact-route decision for flow pair 1 | L3SW/RTR | `show ip cef exact-route <SOURCE_IP_1> <DESTINATION_IP>` | Output shows the selected egress interface and next hop for that flow |
| 12 | Verify CEF exact-route decision for flow pair 2 | L3SW/RTR | `show ip cef exact-route <SOURCE_IP_2> <DESTINATION_IP>` | A different source may hash to a different ECMP next hop |
| 13 | Generate traffic from multiple sources if available | Host/RTR | `ping <DESTINATION_IP> source <SOURCE_IP_OR_INTERFACE>` | Traffic succeeds and creates observable forwarding counters |
| 14 | Validate path selection with traceroute from source 1 | Host/RTR | `traceroute <DESTINATION_IP> source <SOURCE_IP_1>` | First routed hop matches the CEF exact-route result |
| 15 | Validate path selection with traceroute from source 2 | Host/RTR | `traceroute <DESTINATION_IP> source <SOURCE_IP_2>` | First routed hop may differ if hash selects another ECMP path |
| 16 | Check interface counters across ECMP links | L3SW/RTR | `show interfaces <ECMP_INTERFACE_1>`<br>`show interfaces <ECMP_INTERFACE_2>` | Counters increment across paths when multiple flows are present |
| 17 | Confirm no PBR overrides normal CEF ECMP behavior | L3SW/RTR | `show ip policy` | No unexpected route map is applied to the ingress interface |
| 18 | Confirm routing protocol maximum path setting if ECMP comes from dynamic routing | L3SW/RTR | `show ip protocols` | Protocol output shows allowed maximum paths for load sharing |
| 19 | Adjust maximum paths if the routing protocol is limiting ECMP | L3SW/RTR | `router <PROTOCOL>`<br>`maximum-paths <NUMBER>` | Routing process permits the required number of equal-cost routes |
| 20 | Save the working ECMP configuration | L3SW/RTR | `copy running-config startup-config` | ECMP configuration survives reload |

# CEF_ECMP_Per_Flow_Load_Sharing_Skeleton

conf t
!
! Required on multilayer switches.
ip routing
!
! CEF is normally enabled by default on modern Cisco IOS/IOS XE.
ip cef
!
interface GigabitEthernet0/0
 description LAN_OR_INGRESS_SIDE
 ip address 10.1.1.1 255.255.255.0
 no shutdown
!
interface GigabitEthernet0/1
 description ECMP_PATH_1_TO_NEXT_HOP_10.12.12.2
 ip address 10.12.12.1 255.255.255.0
 no shutdown
!
interface GigabitEthernet0/2
 description ECMP_PATH_2_TO_NEXT_HOP_10.13.13.3
 ip address 10.13.13.1 255.255.255.0
 no shutdown
!
! Static ECMP example.
ip route 192.168.100.0 255.255.255.0 10.12.12.2
ip route 192.168.100.0 255.255.255.0 10.13.13.3
!
end
write memory

# Optional_Dynamic_Routing_Maximum_Paths_Skeleton

conf t
!
router ospf 10
 maximum-paths 4
!
router eigrp 100
 maximum-paths 4
!
router bgp <ASN>
 maximum-paths 4
!
end
write memory

# CEF_ECMP_Per_Flow_Load_Sharing_Verification_Commands

show running-config | include ^ip routing
show running-config | include ^ip cef
show ip interface brief
show ip route <DESTINATION_PREFIX>
show ip route <DESTINATION_IP>
show ip protocols
show ip cef <DESTINATION_IP>
show ip cef <DESTINATION_PREFIX> <MASK>
show ip cef exact-route <SOURCE_IP_1> <DESTINATION_IP>
show ip cef exact-route <SOURCE_IP_2> <DESTINATION_IP>
show ip cef exact-route <SOURCE_IP_3> <DESTINATION_IP>
show adjacency
show adjacency detail
show ip arp
show ip arp <NEXT_HOP_1>
show ip arp <NEXT_HOP_2>
show ip policy
show interfaces <ECMP_INTERFACE_1>
show interfaces <ECMP_INTERFACE_2>
ping <NEXT_HOP_1>
ping <NEXT_HOP_2>
ping <DESTINATION_IP> source <SOURCE_IP_OR_INTERFACE>
traceroute <DESTINATION_IP> source <SOURCE_IP_1>
traceroute <DESTINATION_IP> source <SOURCE_IP_2>

# CEF_ECMP_Per_Flow_Load_Sharing_Rollback

conf t
no ip route <DESTINATION_PREFIX> <MASK> <NEXT_HOP_1>
no ip route <DESTINATION_PREFIX> <MASK> <NEXT_HOP_2>
no ip route <DESTINATION_PREFIX> <MASK> <NEXT_HOP_3>
no ip route <DESTINATION_PREFIX> <MASK> <NEXT_HOP_4>
end
write memory

! Optional dynamic routing rollback examples.

conf t
router ospf 10
 no maximum-paths 4
!
router eigrp 100
 no maximum-paths 4
!
router bgp <ASN>
 no maximum-paths 4
!
end
write memory

! Production note:
! Do not disable CEF as an ECMP rollback step.
! Remove or correct the ECMP route source instead.

# CEF_ECMP_Per_Flow_Load_Sharing_Failure_Checks

| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| Only one route appears in the routing table | Routes are not truly equal cost | `show ip route <DESTINATION_PREFIX>` | Make metric, AD, and route source equal where ECMP is intended |
| Multiple routes are learned but only one is installed | Maximum paths is too low or protocol best-path logic rejects alternates | `show ip protocols` | Configure `maximum-paths <NUMBER>` under the routing process |
| Static ECMP route does not install | Next hop is unreachable | `show ip route <NEXT_HOP_IP>` | Fix next-hop reachability or use a reachable next hop |
| CEF shows only one adjacency | RIB has only one installed path or adjacency is unresolved | `show ip cef <DESTINATION_IP>` and `show ip route <DESTINATION_IP>` | Fix RIB ECMP first, then fix ARP or adjacency state |
| CEF entry shows incomplete adjacency | Next-hop Layer 2 resolution failed | `show ip arp <NEXT_HOP_IP>` | Fix VLAN, interface, ARP, or next-hop addressing |
| One ping stream uses only one link | Normal per-flow hashing behavior | `show ip cef exact-route <SOURCE_IP> <DESTINATION_IP>` | Test with multiple source and destination pairs |
| Traffic distribution looks uneven | Too few flows or hash inputs produce uneven path selection | `show interfaces <ECMP_INTERFACE>` counters | Generate multiple flows or review hash behavior |
| Traceroute does not match expected ECMP path | The tested source/destination pair hashes to another path | `show ip cef exact-route <SOURCE_IP> <DESTINATION_IP>` | Use exact-route output as the forwarding source of truth |
| Path changes after source IP changes | Hash input changed | `show ip cef exact-route <NEW_SOURCE_IP> <DESTINATION_IP>` | Expected behavior; document which flows map to which paths |
| ECMP seems ignored | PBR overrides normal destination-based forwarding | `show ip policy` | Remove or correct ingress route map policy |
| Forwarding works one way but not return traffic | Return path is missing or asymmetric | Downstream `show ip route <SOURCE_IP>` | Add reverse routing or validate downstream ECMP |
| CPU spikes during testing | Packets are being punted due to unresolved adjacency or exception traffic | `show ip cef <DESTINATION_IP>` and `show adjacency detail` | Resolve adjacency and avoid exception traffic tests |
| All traffic uses the same upstream path across multiple routers | CEF polarization | Compare `show ip cef exact-route` across hops | Adjust topology, path count, or hash behavior in the polarization note |
| Dynamic protocol has unequal metrics | Protocol metric calculation differs per path | `show ip route <DESTINATION_PREFIX>` and protocol topology/database command | Tune cost, bandwidth, delay, or metric so intended paths are equal |
| Route exists but traffic fails after first hop | Downstream next hop lacks route or return path | `traceroute <DESTINATION_IP>` | Fix downstream route, ACL, firewall, or reverse path |

# Index

CEF_ECMP_Per_Flow_Load_Sharing.md
CEF_ECMP_Per_Flow_Load_Sharing
CEF_ECMP_Per_Flow_Load_Sharing_Mental_Model
CEF_ECMP_Per_Flow_Load_Sharing_Configuration_Checklist
CEF_ECMP_Per_Flow_Load_Sharing_Skeleton
Optional_Dynamic_Routing_Maximum_Paths_Skeleton
CEF_ECMP_Per_Flow_Load_Sharing_Verification_Commands
CEF_ECMP_Per_Flow_Load_Sharing_Rollback
CEF_ECMP_Per_Flow_Load_Sharing_Failure_Checks