CEF_Polarization_Hash_Behavior.md
# CEF_Polarization_Hash_Behavior

# CEF_Polarization_Hash_Behavior_Mental_Model

| Concept | Operational Meaning |
|---|---|
| CEF polarization | Multiple routers repeatedly hash the same flows onto the same physical path, causing poor link utilization even though ECMP exists |
| Deterministic hash | CEF load sharing is predictable for a given flow because the same input fields usually produce the same path decision |
| Per-flow forwarding | A single flow normally stays on one path; ECMP load sharing becomes visible only when multiple flows exist |
| ECMP prerequisite | Polarization only matters where multiple equal-cost paths are installed and programmed into CEF |
| Not a routing-table failure | The routing table can look perfect while traffic still concentrates on one link because the problem is forwarding hash behavior |
| Hash input fields | Source IP, destination IP, Layer 4 ports, tunnel headers, or platform-specific fields may influence the selected ECMP path |
| Overlay risk | GRE, VXLAN, LISP, IPsec, or tunnel traffic can hide many inner flows behind the same outer source and destination, reducing hash diversity |
| Universal algorithm | The universal CEF algorithm is intended for most environments and helps reduce repeated hash alignment across routers |
| Tunnel algorithm | Tunnel-oriented hashing can help when many flows are encapsulated and normal hash inputs are not diverse enough |
| Validation method | Use `show ip cef exact-route` and interface counters across multiple source/destination pairs; do not judge ECMP using one ping flow |

# CEF_Polarization_Hash_Behavior_Configuration_Checklist

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm Layer 3 forwarding is enabled on multilayer switches | L3SW/RTR | `show running-config | include ^ip routing` | `ip routing` is present on multilayer switches |
| 2 | Confirm CEF is enabled | L3SW/RTR | `show ip cef` | CEF table is available |
| 3 | Confirm candidate ECMP interfaces are up | L3SW/RTR | `show ip interface brief` | All expected ECMP transit interfaces are `up/up` |
| 4 | Confirm the routing table has equal-cost paths | L3SW/RTR | `show ip route <DESTINATION_PREFIX>` | Multiple next hops appear for the same destination prefix |
| 5 | Confirm CEF programmed multiple adjacencies | L3SW/RTR | `show ip cef <DESTINATION_IP>` | Destination has multiple CEF next-hop adjacencies |
| 6 | Confirm next-hop Layer 2 resolution is complete | L3SW/RTR | `show ip arp <NEXT_HOP_1>`<br>`show ip arp <NEXT_HOP_2>` | Each next hop resolves to a MAC address on the expected interface |
| 7 | Capture the current CEF load-sharing configuration | L3SW/RTR | `show running-config all | include ip cef load-sharing algorithm` | Current CEF algorithm setting is documented before changes |
| 8 | Test exact-route hash result for flow pair 1 | L3SW/RTR | `show ip cef exact-route <SOURCE_IP_1> <DESTINATION_IP>` | Output shows selected egress interface and next hop |
| 9 | Test exact-route hash result for flow pair 2 | L3SW/RTR | `show ip cef exact-route <SOURCE_IP_2> <DESTINATION_IP>` | Output may select the same or different ECMP path |
| 10 | Test exact-route hash result for flow pair 3 | L3SW/RTR | `show ip cef exact-route <SOURCE_IP_3> <DESTINATION_IP>` | Multiple flows reveal whether hashing is spreading traffic |
| 11 | Generate multiple flows across the ECMP set | Host/RTR | `ping <DESTINATION_IP> source <SOURCE_IP_OR_INTERFACE>` | Traffic succeeds and creates interface counter movement |
| 12 | Check per-interface utilization across ECMP paths | L3SW/RTR | `show interfaces <ECMP_INTERFACE_1>`<br>`show interfaces <ECMP_INTERFACE_2>` | Counters show whether traffic is balanced or concentrated |
| 13 | Identify polarization pattern across multiple routers | L3SW/RTR | `show ip cef exact-route <SOURCE_IP> <DESTINATION_IP>` | Multiple routers selecting the same relative path indicates polarization risk |
| 14 | Configure the universal algorithm for normal routed ECMP environments | L3SW/RTR | `conf t`<br>`ip cef load-sharing algorithm universal`<br>`end` | CEF uses the universal load-sharing algorithm |
| 15 | Configure source-only hashing only when a lab or design explicitly requires it | L3SW/RTR | `conf t`<br>`ip cef load-sharing algorithm src-only`<br>`end` | CEF hashes using source address behavior where supported |
| 16 | Configure tunnel hashing when the design is tunnel-heavy and the platform supports it | L3SW/RTR | `conf t`<br>`ip cef load-sharing algorithm tunnel`<br>`end` | CEF uses tunnel-oriented hashing behavior |
| 17 | Configure include-ports hashing where supported and required for better flow diversity | L3SW/RTR | `conf t`<br>`ip cef load-sharing algorithm include-ports`<br>`end` | CEF includes Layer 4 port information where the platform supports this option |
| 18 | Recheck exact-route behavior after the hash change | L3SW/RTR | `show ip cef exact-route <SOURCE_IP_1> <DESTINATION_IP>`<br>`show ip cef exact-route <SOURCE_IP_2> <DESTINATION_IP>` | Flow-to-path mapping is re-evaluated after algorithm change |
| 19 | Recheck interface counters under multiple-flow traffic | L3SW/RTR | `show interfaces <ECMP_INTERFACE_1>`<br>`show interfaces <ECMP_INTERFACE_2>` | Utilization is less concentrated if hash diversity improved |
| 20 | Confirm no PBR is overriding the ECMP decision | L3SW/RTR | `show ip policy` | No unexpected route map overrides normal CEF forwarding |
| 21 | Confirm no single-flow test is being mistaken for polarization | L3SW/RTR | `show ip cef exact-route <SOURCE_IP> <DESTINATION_IP>` | One flow mapping to one link is normal per-flow behavior |
| 22 | Save the working configuration | L3SW/RTR | `copy running-config startup-config` | CEF load-sharing setting survives reload |

# CEF_Polarization_Hash_Behavior_Skeleton

conf t
!
! Required on multilayer switches.
ip routing
!
! CEF is normally enabled by default on modern Cisco IOS/IOS XE.
ip cef
!
interface GigabitEthernet0/0
 description INGRESS_SIDE
 ip address 10.1.1.1 255.255.255.0
 no shutdown
!
interface GigabitEthernet0/1
 description ECMP_PATH_1
 ip address 10.12.12.1 255.255.255.0
 no shutdown
!
interface GigabitEthernet0/2
 description ECMP_PATH_2
 ip address 10.13.13.1 255.255.255.0
 no shutdown
!
ip route 192.168.100.0 255.255.255.0 10.12.12.2
ip route 192.168.100.0 255.255.255.0 10.13.13.3
!
! General-purpose CEF load-sharing algorithm.
ip cef load-sharing algorithm universal
!
end
write memory

# Optional_CEF_Hash_Algorithm_Skeletons

conf t
!
! Lab or specific design use only.
ip cef load-sharing algorithm src-only
!
end

conf t
!
! Tunnel-heavy design use only, platform support required.
ip cef load-sharing algorithm tunnel
!
end

conf t
!
! Platform support required.
ip cef load-sharing algorithm include-ports
!
end

# CEF_Polarization_Hash_Behavior_Verification_Commands

show running-config | include ^ip routing
show running-config | include ^ip cef
show running-config all | include ip cef load-sharing algorithm
show ip interface brief
show ip route <DESTINATION_PREFIX>
show ip route <DESTINATION_IP>
show ip cef <DESTINATION_IP>
show ip cef <DESTINATION_PREFIX> <MASK>
show ip cef exact-route <SOURCE_IP_1> <DESTINATION_IP>
show ip cef exact-route <SOURCE_IP_2> <DESTINATION_IP>
show ip cef exact-route <SOURCE_IP_3> <DESTINATION_IP>
show ip cef exact-route <SOURCE_IP_4> <DESTINATION_IP>
show adjacency detail
show ip arp <NEXT_HOP_1>
show ip arp <NEXT_HOP_2>
show interfaces <ECMP_INTERFACE_1>
show interfaces <ECMP_INTERFACE_2>
show ip policy
ping <DESTINATION_IP> source <SOURCE_IP_OR_INTERFACE>
traceroute <DESTINATION_IP> source <SOURCE_IP_1>
traceroute <DESTINATION_IP> source <SOURCE_IP_2>

# CEF_Polarization_Hash_Behavior_Rollback

conf t
!
! Return to the general-purpose algorithm.
ip cef load-sharing algorithm universal
!
end
write memory

! If the lab used only static ECMP routes and the whole test must be removed:

conf t
no ip route <DESTINATION_PREFIX> <MASK> <NEXT_HOP_1>
no ip route <DESTINATION_PREFIX> <MASK> <NEXT_HOP_2>
no ip route <DESTINATION_PREFIX> <MASK> <NEXT_HOP_3>
no ip route <DESTINATION_PREFIX> <MASK> <NEXT_HOP_4>
end
write memory

! Production note:
! Do not disable CEF as a polarization fix.
! Correct the ECMP design, hash algorithm, tunnel design, or traffic distribution method instead.

# CEF_Polarization_Hash_Behavior_Failure_Checks

| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| All traffic appears to use one ECMP link | Single-flow test or poor hash diversity | `show ip cef exact-route <SOURCE_IP> <DESTINATION_IP>` | Test multiple source/destination pairs before changing configuration |
| Multiple flows still use one link | Hash polarization across equal-cost paths | `show ip cef exact-route <SOURCE_IP_1> <DESTINATION_IP>` through multiple source pairs | Change supported CEF load-sharing algorithm or redesign path diversity |
| ECMP exists in the RIB but not in CEF | CEF did not program multiple adjacencies | `show ip cef <DESTINATION_IP>` | Fix unresolved next hops, adjacency state, or route installation |
| One ECMP next hop is missing from the RIB | Routes are not equal cost | `show ip route <DESTINATION_PREFIX>` | Normalize metric, AD, or protocol path selection |
| Interface counters are uneven | Too few flows or repeated hash selection | `show interfaces <ECMP_INTERFACE>` | Generate more flows and validate with exact-route lookups |
| Hash algorithm command is rejected | Platform or image does not support that option | `ip cef load-sharing algorithm ?` | Use only the options supported on that platform |
| `include-ports` has no visible effect | Traffic lacks useful Layer 4 diversity or platform behavior differs | Compare exact-route outputs before and after change | Use tunnel hashing, universal hashing, or redesign flow diversity |
| Tunnel traffic concentrates on one path | Outer tunnel headers hide inner-flow diversity | `show ip cef exact-route <TUNNEL_SOURCE> <TUNNEL_DESTINATION>` | Use tunnel-aware hashing where supported or add tunnel/path diversity |
| Traceroute always shows the same path | Same source/destination flow hashes to the same path | `show ip cef exact-route <SOURCE_IP> <DESTINATION_IP>` | Change source/destination pair or generate multiple flows |
| PBR hides the CEF ECMP decision | Route map forces selected traffic to a next hop | `show ip policy` | Remove or correct PBR before testing ECMP hashing |
| Traffic fails after hash change | One ECMP path has broken downstream forwarding | `traceroute <DESTINATION_IP> source <SOURCE_IP>` | Fix downstream route, return route, ACL, or next-hop reachability |
| Link utilization worsens after changing algorithm | New hash setting is worse for this traffic mix | Compare interface counters before and after | Roll back to `ip cef load-sharing algorithm universal` |
| Recursive next-hop issue causes path loss | Next-hop route changed or disappeared | `show ip route <NEXT_HOP_IP>` | Fix recursion or use stable directly reachable next hops |
| ARP failure removes an ECMP path from forwarding | Next-hop MAC resolution failed | `show ip arp <NEXT_HOP_IP>` | Fix Layer 2 reachability, VLAN, trunking, or next-hop address |
| CEF exact-route differs from expected lab diagram | Diagram assumes round-robin load sharing | `show ip cef exact-route <SOURCE_IP> <DESTINATION_IP>` | Trust exact-route for the tested flow and document hash behavior |

# Index

CEF_Polarization_Hash_Behavior.md
CEF_Polarization_Hash_Behavior
CEF_Polarization_Hash_Behavior_Mental_Model
CEF_Polarization_Hash_Behavior_Configuration_Checklist
CEF_Polarization_Hash_Behavior_Skeleton
Optional_CEF_Hash_Algorithm_Skeletons
CEF_Polarization_Hash_Behavior_Verification_Commands
CEF_Polarization_Hash_Behavior_Rollback
CEF_Polarization_Hash_Behavior_Failure_Checks