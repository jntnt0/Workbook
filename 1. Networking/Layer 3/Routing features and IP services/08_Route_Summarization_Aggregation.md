Route_Summarization_Aggregation.md
# Route_Summarization_Aggregation

# Route_Summarization_Aggregation_Mental_Model

| Concept | Operational Meaning |
|---|---|
| Route summarization | Multiple specific prefixes are represented by one shorter aggregate prefix |
| Aggregation | Another name for summarization, commonly used in BGP contexts |
| Component routes | The more specific prefixes that fall inside the summary range |
| Summary route | The aggregate route advertised to neighbors or installed locally |
| Suppression | More specific component routes may be hidden from downstream routers when the summary is advertised |
| Stability benefit | Flapping component routes can be hidden behind a stable summary |
| Scale benefit | Fewer routes means smaller routing tables and less control-plane churn |
| Longest prefix match | If a leaked or more specific route exists, it still beats the summary route for matching traffic |
| Discard route | A Null0 route for the summary prevents loops when traffic matches the summary but no component route exists |
| Blackhole risk | A summary can attract traffic for addresses that are not actually reachable |
| EIGRP boundary | EIGRP summarization is applied outbound on a specific interface |
| OSPF ABR boundary | OSPF inter-area summarization is done on an ABR with `area range` |
| OSPF ASBR boundary | OSPF external summarization is done on an ASBR with `summary-address` |
| BGP boundary | BGP aggregation is done with `aggregate-address`, or by advertising a static Null0 summary with `network` |
| Verification rule | Always verify both the local discard behavior and the downstream routing table |

# Route_Summarization_Aggregation_Configuration_Checklist

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify the component prefixes to summarize | RTR/L3SW | `show ip route <COMPONENT_PREFIX>` | More specific routes exist and fall inside the intended summary range |
| 2 | Calculate the correct aggregate prefix and mask | RTR/L3SW | `! Example: 172.16.0.0/24 through 172.16.15.0/24 summarize to 172.16.0.0/20` | Summary covers only the intended component block |
| 3 | Confirm no unrelated networks fall inside the summary | RTR/L3SW | `show ip route <SUMMARY_PREFIX> <SUMMARY_MASK>` | No unintended more-specific routes are accidentally covered |
| 4 | Confirm where summarization should occur | RTR/L3SW | `show ip protocols` | Correct routing process and boundary router are identified |
| 5 | Confirm routing adjacencies are healthy before summarizing | RTR/L3SW | `show ip eigrp neighbors`<br>`show ip ospf neighbor`<br>`show bgp ipv4 unicast summary` | Routing peers are established before policy change |
| 6 | Confirm the downstream table before summarization | Downstream RTR | `show ip route <COMPONENT_PREFIX>` | Downstream router currently sees the specific routes |
| 7 | Optional: install a static discard route for protocol-neutral summarization | Summary RTR | `conf t`<br>`ip route <SUMMARY_PREFIX> <SUMMARY_MASK> Null0`<br>`end` | Local router discards traffic matching the summary when no better route exists |
| 8 | Optional: advertise a static summary with a routing process | Summary RTR | `router bgp <ASN>`<br>` network <SUMMARY_PREFIX> mask <SUMMARY_MASK>` | Summary can be advertised if the static Null0 route exists in the RIB |
| 9 | Configure EIGRP classic interface summarization | EIGRP Summary RTR | `conf t`<br>`interface <OUTBOUND_INTERFACE>`<br>` ip summary-address eigrp <ASN> <SUMMARY_PREFIX> <SUMMARY_MASK>`<br>`end` | EIGRP advertises the summary out the selected interface when at least one component route exists |
| 10 | Optional: configure EIGRP classic summary with leak map | EIGRP Summary RTR | `interface <OUTBOUND_INTERFACE>`<br>` ip summary-address eigrp <ASN> <SUMMARY_PREFIX> <SUMMARY_MASK> leak-map <ROUTE_MAP>` | Summary is advertised and selected leaked component routes are also advertised |
| 11 | Configure EIGRP named-mode interface summarization | EIGRP Summary RTR | `conf t`<br>`router eigrp <NAME>`<br>` address-family ipv4 unicast autonomous-system <ASN>`<br>`  af-interface <OUTBOUND_INTERFACE>`<br>`   summary-address <SUMMARY_PREFIX> <SUMMARY_MASK>`<br>`end` | Named-mode EIGRP advertises the summary out the selected interface |
| 12 | Configure OSPF inter-area summarization on an ABR | OSPF ABR | `conf t`<br>`router ospf <PROCESS_ID>`<br>` area <SOURCE_AREA_ID> range <SUMMARY_PREFIX> <SUMMARY_MASK>`<br>`end` | ABR advertises a Type 3 summary LSA into other areas |
| 13 | Optional: set OSPF inter-area summary metric | OSPF ABR | `router ospf <PROCESS_ID>`<br>` area <SOURCE_AREA_ID> range <SUMMARY_PREFIX> <SUMMARY_MASK> cost <METRIC>` | Summary LSA uses the configured metric |
| 14 | Optional: filter an OSPF range with summarization | OSPF ABR | `router ospf <PROCESS_ID>`<br>` area <SOURCE_AREA_ID> range <SUMMARY_PREFIX> <SUMMARY_MASK> not-advertise` | Matching range is suppressed from advertisement into other areas |
| 15 | Configure OSPF external summarization on an ASBR | OSPF ASBR | `conf t`<br>`router ospf <PROCESS_ID>`<br>` summary-address <SUMMARY_PREFIX> <SUMMARY_MASK>`<br>`end` | ASBR summarizes redistributed external prefixes |
| 16 | Configure BGP dynamic aggregation | BGP RTR | `conf t`<br>`router bgp <ASN>`<br>` address-family ipv4 unicast`<br>`  aggregate-address <SUMMARY_PREFIX> <SUMMARY_MASK>`<br>`end` | BGP creates an aggregate when a matching component exists in the BGP table |
| 17 | Configure BGP dynamic aggregation while suppressing specifics | BGP RTR | `router bgp <ASN>`<br>` address-family ipv4 unicast`<br>`  aggregate-address <SUMMARY_PREFIX> <SUMMARY_MASK> summary-only` | BGP advertises the aggregate and suppresses matching component prefixes |
| 18 | Configure BGP aggregate with AS path preservation where needed | BGP RTR | `router bgp <ASN>`<br>` address-family ipv4 unicast`<br>`  aggregate-address <SUMMARY_PREFIX> <SUMMARY_MASK> as-set` | Aggregate includes AS_SET information for component AS paths |
| 19 | Verify the summary exists locally | Summary RTR | `show ip route <SUMMARY_PREFIX> <SUMMARY_MASK>` | Summary route or discard route appears locally |
| 20 | Verify protocol-specific summary advertisement | Summary RTR | `show ip eigrp topology <SUMMARY_PREFIX>/<PREFIX_LENGTH>`<br>`show ip ospf database summary <SUMMARY_PREFIX>`<br>`show bgp ipv4 unicast <SUMMARY_PREFIX>/<PREFIX_LENGTH>` | Routing protocol shows the summary in the expected database or table |
| 21 | Verify downstream routers receive the summary | Downstream RTR | `show ip route <SUMMARY_PREFIX>` | Downstream router sees the aggregate route |
| 22 | Verify component route suppression if intended | Downstream RTR | `show ip route <COMPONENT_PREFIX>` | Component route is absent when summary-only behavior is intended |
| 23 | Verify leaked component routes if intended | Downstream RTR | `show ip route <LEAKED_COMPONENT_PREFIX>` | Leaked component route appears and wins by longest prefix match |
| 24 | Verify forwarding to reachable component destinations | Downstream RTR | `ping <REACHABLE_COMPONENT_HOST> source <SOURCE_IP_OR_INTERFACE>` | Traffic to real component networks succeeds |
| 25 | Verify unreachable addresses inside the summary do not loop | Summary RTR | `show ip route <UNUSED_IP_INSIDE_SUMMARY>` | Best match is the summary discard route or otherwise drops safely |
| 26 | Trace reachable traffic path | Downstream RTR | `traceroute <REACHABLE_COMPONENT_HOST> source <SOURCE_IP>` | Traffic follows the summarized path toward the summarizing router |
| 27 | Trace unused address inside the summary | Downstream RTR | `traceroute <UNUSED_IP_INSIDE_SUMMARY> source <SOURCE_IP>` | Traffic stops at or beyond the summarizing router instead of looping |
| 28 | Confirm CEF forwarding for the summary | Summary RTR | `show ip cef <SUMMARY_PREFIX>` | CEF points to expected next hop or Null0 discard |
| 29 | Confirm component route churn is hidden downstream | Downstream RTR | `show ip route <SUMMARY_PREFIX>` | Summary remains stable even if one component route changes |
| 30 | Save the working configuration | RTR/L3SW | `copy running-config startup-config` | Summary configuration survives reload |

# Route_Summarization_Aggregation_Skeleton

conf t
!
! Protocol-neutral discard route.
! Use this when advertising a summary with a network statement or when you need local loop prevention.
ip route <SUMMARY_PREFIX> <SUMMARY_MASK> Null0
!
end
write memory

# EIGRP_Classic_Interface_Summarization_Skeleton

conf t
!
interface <OUTBOUND_INTERFACE>
 ip summary-address eigrp <ASN> <SUMMARY_PREFIX> <SUMMARY_MASK>
!
end
write memory

# EIGRP_Named_Mode_Interface_Summarization_Skeleton

conf t
!
router eigrp <NAME>
 address-family ipv4 unicast autonomous-system <ASN>
  af-interface <OUTBOUND_INTERFACE>
   summary-address <SUMMARY_PREFIX> <SUMMARY_MASK>
!
end
write memory

# EIGRP_Summarization_With_Leak_Map_Skeleton

conf t
!
ip prefix-list PL-LEAK-COMPONENT seq 10 permit <LEAKED_COMPONENT_PREFIX>/<PREFIX_LENGTH>
!
route-map RM-LEAK-COMPONENT permit 10
 match ip address prefix-list PL-LEAK-COMPONENT
!
interface <OUTBOUND_INTERFACE>
 ip summary-address eigrp <ASN> <SUMMARY_PREFIX> <SUMMARY_MASK> leak-map RM-LEAK-COMPONENT
!
end
write memory

# OSPF_Inter_Area_ABR_Summarization_Skeleton

conf t
!
router ospf <PROCESS_ID>
 area <SOURCE_AREA_ID> range <SUMMARY_PREFIX> <SUMMARY_MASK>
!
end
write memory

# OSPF_Inter_Area_ABR_Summarization_With_Cost_Skeleton

conf t
!
router ospf <PROCESS_ID>
 area <SOURCE_AREA_ID> range <SUMMARY_PREFIX> <SUMMARY_MASK> cost <METRIC>
!
end
write memory

# OSPF_External_ASBR_Summarization_Skeleton

conf t
!
router ospf <PROCESS_ID>
 summary-address <SUMMARY_PREFIX> <SUMMARY_MASK>
!
end
write memory

# BGP_Dynamic_Aggregation_Skeleton

conf t
!
router bgp <ASN>
 address-family ipv4 unicast
  aggregate-address <SUMMARY_PREFIX> <SUMMARY_MASK>
!
end
write memory

# BGP_Dynamic_Aggregation_Summary_Only_Skeleton

conf t
!
router bgp <ASN>
 address-family ipv4 unicast
  aggregate-address <SUMMARY_PREFIX> <SUMMARY_MASK> summary-only
!
end
write memory

# BGP_Static_Null0_Summary_Network_Skeleton

conf t
!
ip route <SUMMARY_PREFIX> <SUMMARY_MASK> Null0
!
router bgp <ASN>
 address-family ipv4 unicast
  network <SUMMARY_PREFIX> mask <SUMMARY_MASK>
!
end
write memory

# Route_Summarization_Aggregation_Verification_Commands

show ip route
show ip route <SUMMARY_PREFIX> <SUMMARY_MASK>
show ip route <COMPONENT_PREFIX>
show ip route <UNUSED_IP_INSIDE_SUMMARY>
show ip cef <SUMMARY_PREFIX>
show ip cef <UNUSED_IP_INSIDE_SUMMARY>
show ip protocols
show running-config | include ^ip route
show running-config interface <OUTBOUND_INTERFACE>
show running-config | section router eigrp
show ip eigrp neighbors
show ip eigrp topology
show ip eigrp topology <SUMMARY_PREFIX>/<PREFIX_LENGTH>
show ip route eigrp
show running-config | section router ospf
show ip ospf neighbor
show ip ospf database summary
show ip ospf database external
show ip route ospf
show running-config | section router bgp
show bgp ipv4 unicast summary
show bgp ipv4 unicast <SUMMARY_PREFIX>/<PREFIX_LENGTH>
show bgp ipv4 unicast neighbors <NEIGHBOR_IP> advertised-routes
show bgp ipv4 unicast neighbors <NEIGHBOR_IP> routes
ping <REACHABLE_COMPONENT_HOST> source <SOURCE_IP_OR_INTERFACE>
ping <UNUSED_IP_INSIDE_SUMMARY> source <SOURCE_IP_OR_INTERFACE>
traceroute <REACHABLE_COMPONENT_HOST> source <SOURCE_IP>
traceroute <UNUSED_IP_INSIDE_SUMMARY> source <SOURCE_IP>

# Route_Summarization_Aggregation_Rollback

conf t
!
! Protocol-neutral static summary rollback.
no ip route <SUMMARY_PREFIX> <SUMMARY_MASK> Null0
!
end
write memory

# EIGRP_Classic_Summarization_Rollback

conf t
!
interface <OUTBOUND_INTERFACE>
 no ip summary-address eigrp <ASN> <SUMMARY_PREFIX> <SUMMARY_MASK>
!
end
write memory

# EIGRP_Named_Mode_Summarization_Rollback

conf t
!
router eigrp <NAME>
 address-family ipv4 unicast autonomous-system <ASN>
  af-interface <OUTBOUND_INTERFACE>
   no summary-address <SUMMARY_PREFIX> <SUMMARY_MASK>
!
end
write memory

# OSPF_Inter_Area_Summarization_Rollback

conf t
!
router ospf <PROCESS_ID>
 no area <SOURCE_AREA_ID> range <SUMMARY_PREFIX> <SUMMARY_MASK>
!
end
write memory

# OSPF_External_Summarization_Rollback

conf t
!
router ospf <PROCESS_ID>
 no summary-address <SUMMARY_PREFIX> <SUMMARY_MASK>
!
end
write memory

# BGP_Dynamic_Aggregation_Rollback

conf t
!
router bgp <ASN>
 address-family ipv4 unicast
  no aggregate-address <SUMMARY_PREFIX> <SUMMARY_MASK>
  no aggregate-address <SUMMARY_PREFIX> <SUMMARY_MASK> summary-only
  no aggregate-address <SUMMARY_PREFIX> <SUMMARY_MASK> as-set
!
end
write memory

# BGP_Static_Null0_Summary_Network_Rollback

conf t
!
router bgp <ASN>
 address-family ipv4 unicast
  no network <SUMMARY_PREFIX> mask <SUMMARY_MASK>
!
no ip route <SUMMARY_PREFIX> <SUMMARY_MASK> Null0
!
end
write memory

# Route_Summarization_Aggregation_Failure_Checks

| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| Summary is not advertised | No matching component route exists | `show ip route <COMPONENT_PREFIX>` | Restore at least one component route inside the summary range |
| Summary covers too much address space | Aggregate mask is too short | `show ip route <SUMMARY_PREFIX> <SUMMARY_MASK>` | Recalculate the summary and use a tighter mask |
| Traffic blackholes inside summary | Summary attracts traffic for unused space | `show ip route <UNUSED_IP_INSIDE_SUMMARY>` | Keep a Null0 discard route and avoid summarizing unrelated space |
| Traffic loops after summarization | No discard route exists on the summarizing router | `show ip route <SUMMARY_PREFIX> <SUMMARY_MASK>` | Add or restore the Null0 summary route |
| Downstream still sees all component routes | Summary does not suppress specifics in that protocol or option was not used | Downstream `show ip route <COMPONENT_PREFIX>` | Use EIGRP summary behavior, OSPF range behavior, or BGP `summary-only` where intended |
| Downstream lost a required specific route | Summary suppressed a route that needed longest-match treatment | Downstream `show ip route <COMPONENT_PREFIX>` | Use EIGRP leak map, BGP policy, or avoid suppressing that specific route |
| EIGRP summary is not sent | Summary configured on wrong outbound interface | `show running-config interface <OUTBOUND_INTERFACE>` | Move `ip summary-address eigrp` to the neighbor-facing outbound interface |
| EIGRP summary missing in local table | No component route matches the summary | `show ip eigrp topology` | Restore component route or correct summary range |
| EIGRP Null0 summary surprises forwarding | Built-in discard route is active | `show ip route <SUMMARY_PREFIX> <SUMMARY_MASK>` | This is normal loop-prevention behavior; verify more specific component routes exist |
| OSPF inter-area summary does not work | Router is not an ABR for the source area | `show ip ospf` and `show ip ospf neighbor` | Configure `area range` only on the ABR connected to the source area |
| OSPF summary has wrong metric | Default summary metric used lowest component metric | `show ip ospf database summary <SUMMARY_PREFIX>` | Configure `area <AREA> range <SUMMARY> <MASK> cost <METRIC>` |
| OSPF `not-advertise` hides expected routes | Range filter is suppressing the prefix | `show running-config | section router ospf` | Remove `not-advertise` or narrow the range |
| OSPF external summary does not appear | Summary configured with wrong command or not on ASBR | `show ip ospf database external` | Use `summary-address` on the ASBR that redistributes the externals |
| BGP aggregate does not appear | No component route exists in BGP table | `show bgp ipv4 unicast` | Ensure at least one matching component is in BGP |
| BGP static network summary does not advertise | Static Null0 route missing from RIB | `show ip route <SUMMARY_PREFIX> <SUMMARY_MASK>` | Add `ip route <SUMMARY_PREFIX> <SUMMARY_MASK> Null0` |
| BGP specifics still advertise | `summary-only` not configured | `show bgp ipv4 unicast neighbors <NEIGHBOR_IP> advertised-routes` | Add `aggregate-address <SUMMARY> <MASK> summary-only` if suppression is intended |
| BGP aggregate loses AS path detail | Aggregate created without `as-set` | `show bgp ipv4 unicast <SUMMARY_PREFIX>/<PREFIX_LENGTH>` | Use `as-set` only where preserving component AS information is required |
| More specific route still wins over summary | Longest prefix match is working normally | `show ip route <DESTINATION_IP>` | Accept this if intentional, or suppress the specific advertisement |
| Summary route remains during component failure | Static Null0 summary is always present | `show ip route <SUMMARY_PREFIX> <SUMMARY_MASK>` | Use dynamic protocol aggregation if summary should depend on components |
| Route churn still visible downstream | Components are still being advertised | Downstream `show ip route` and protocol database | Suppress specifics or summarize closer to the unstable edge |
| Summarization causes asymmetric routing | Only one side of the path was summarized or return path differs | Traceroute from both directions | Fix return routing or summarize consistently |
| Lab works for ping but fails for real hosts | Return routes missing from component networks | Downstream `show ip route <SOURCE_PREFIX>` | Add reverse routes or summarize both directions correctly |

# Index

Route_Summarization_Aggregation.md
Route_Summarization_Aggregation
Route_Summarization_Aggregation_Mental_Model
Route_Summarization_Aggregation_Configuration_Checklist
Route_Summarization_Aggregation_Skeleton
EIGRP_Classic_Interface_Summarization_Skeleton
EIGRP_Named_Mode_Interface_Summarization_Skeleton
EIGRP_Summarization_With_Leak_Map_Skeleton
OSPF_Inter_Area_ABR_Summarization_Skeleton
OSPF_Inter_Area_ABR_Summarization_With_Cost_Skeleton
OSPF_External_ASBR_Summarization_Skeleton
BGP_Dynamic_Aggregation_Skeleton
BGP_Dynamic_Aggregation_Summary_Only_Skeleton
BGP_Static_Null0_Summary_Network_Skeleton
Route_Summarization_Aggregation_Verification_Commands
Route_Summarization_Aggregation_Rollback
EIGRP_Classic_Summarization_Rollback
EIGRP_Named_Mode_Summarization_Rollback
OSPF_Inter_Area_Summarization_Rollback
OSPF_External_Summarization_Rollback
BGP_Dynamic_Aggregation_Rollback
BGP_Static_Null0_Summary_Network_Rollback
Route_Summarization_Aggregation_Failure_Checks