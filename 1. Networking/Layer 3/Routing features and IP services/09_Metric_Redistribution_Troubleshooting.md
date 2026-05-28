Metric_Redistribution_Troubleshooting.md
# Metric_Redistribution_Troubleshooting

# Metric_Redistribution_Troubleshooting_Mental_Model

| Concept | Operational Meaning |
|---|---|
| Redistribution | Taking routes from one routing source and injecting them into another routing process |
| Boundary router | The router performing redistribution between routing domains |
| Source protocol | The protocol or source where the route originally exists before redistribution |
| Destination protocol | The protocol receiving the redistributed route |
| Seed metric | The initial metric assigned to a route when it enters the destination protocol |
| Metric translation | Different protocols use different metric systems, so redistribution must assign a metric that makes sense to the destination protocol |
| EIGRP seed metric | EIGRP requires a usable seed metric; without one, redistributed routes can be treated as unreachable |
| OSPF external metric | OSPF redistributed routes become external routes, usually E2 by default unless configured as E1 |
| BGP redistribution metric | BGP can carry the IGP metric as MED when routes are redistributed into BGP |
| Administrative distance | AD decides which same-prefix route enters the RIB when multiple protocols offer the same route |
| Route tag | A non-forwarding attribute used to mark redistributed routes and prevent route feedback loops |
| Route feedback | A route redistributed from one domain into another gets redistributed back into the original domain |
| Suboptimal routing | Redistribution may make a route reachable through the wrong boundary router because of bad seed metrics or AD |
| Redistribution is not transitive | A route learned through redistribution is not automatically redistributed again unless explicitly configured |
| Route-map control | Route maps filter, tag, and set metrics during redistribution |
| Troubleshooting boundary | Always check the source RIB, redistribution syntax, route-map policy, destination protocol database, and downstream RIB separately |

# Metric_Redistribution_Troubleshooting_Configuration_Checklist

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm the route exists in the source routing table before redistribution | Boundary RTR | `show ip route <SOURCE_PREFIX>` | Source prefix exists in the RIB on the redistribution router |
| 2 | Confirm the source protocol owns or knows the route | Boundary RTR | `show ip route <SOURCE_PROTOCOL>` | Source protocol routes are visible in the routing table |
| 3 | Confirm the destination protocol is running | Boundary RTR | `show ip protocols` | Destination routing process is enabled and lists expected networks, neighbors, and redistribution settings |
| 4 | Confirm routing adjacencies are healthy before troubleshooting redistribution | Boundary RTR | `show ip eigrp neighbors`<br>`show ip ospf neighbor`<br>`show bgp ipv4 unicast summary` | Neighbor relationships are established |
| 5 | Confirm current redistribution configuration | Boundary RTR | `show running-config | section router` | `redistribute` commands are visible under the intended destination protocol |
| 6 | Confirm route maps referenced by redistribution exist | Boundary RTR | `show route-map` | Referenced route maps exist with correct permit, deny, match, and set logic |
| 7 | Confirm prefix lists or ACLs referenced by route maps exist | Boundary RTR | `show ip prefix-list`<br>`show access-lists` | Match objects exist and permit the intended prefixes |
| 8 | Verify redistribution into EIGRP includes a seed metric | Boundary RTR | `show running-config | section router eigrp` | Redistribute command includes `metric <bandwidth> <delay> <reliability> <load> <mtu>`, a route map with `set metric`, or `default-metric` |
| 9 | Configure EIGRP redistribution with metric directly on the command | Boundary RTR | `conf t`<br>`router eigrp <ASN>`<br>` redistribute <SOURCE_PROTOCOL> <SOURCE_PROCESS_OR_ASN> metric <BANDWIDTH> <DELAY> <RELIABILITY> <LOAD> <MTU>`<br>`end` | Routes from the source protocol are eligible for EIGRP redistribution with a usable metric |
| 10 | Configure EIGRP default metric if multiple redistribution statements need the same seed metric | Boundary RTR | `conf t`<br>`router eigrp <ASN>`<br>` default-metric <BANDWIDTH> <DELAY> <RELIABILITY> <LOAD> <MTU>`<br>`end` | EIGRP uses the configured default seed metric for redistribution statements without explicit metric values |
| 11 | Configure EIGRP redistribution using route-map metric control | Boundary RTR | `conf t`<br>`route-map RM-REDIST-INTO-EIGRP permit 10`<br>` match ip address prefix-list PL-REDIST-INTO-EIGRP`<br>` set metric <BANDWIDTH> <DELAY> <RELIABILITY> <LOAD> <MTU>`<br>` set tag <TAG>`<br>`router eigrp <ASN>`<br>` redistribute <SOURCE_PROTOCOL> <SOURCE_PROCESS_OR_ASN> route-map RM-REDIST-INTO-EIGRP`<br>`end` | Matching source routes are redistributed into EIGRP with controlled metric and tag |
| 12 | Verify redistributed routes enter the EIGRP topology table | Boundary RTR | `show ip eigrp topology` | Redistributed routes appear in EIGRP topology as external routes |
| 13 | Verify downstream routers receive EIGRP external routes | EIGRP RTR | `show ip route eigrp` | Redistributed routes appear as `D EX` routes where expected |
| 14 | Verify redistribution into OSPF includes `subnets` for classless prefixes | Boundary RTR | `show running-config | section router ospf` | OSPF redistribution command includes `subnets` when redistributing subnetted routes |
| 15 | Configure OSPF redistribution with explicit metric, metric type, tag, and route map | Boundary RTR | `conf t`<br>`router ospf <PROCESS_ID>`<br>` redistribute <SOURCE_PROTOCOL> <SOURCE_PROCESS_OR_ASN> subnets metric <METRIC> metric-type 1 tag <TAG> route-map RM-REDIST-INTO-OSPF`<br>`end` | Source routes are redistributed into OSPF as external routes with explicit metric behavior |
| 16 | Configure OSPF Type 2 external redistribution only when fixed external cost is intended | Boundary RTR | `conf t`<br>`router ospf <PROCESS_ID>`<br>` redistribute <SOURCE_PROTOCOL> <SOURCE_PROCESS_OR_ASN> subnets metric <METRIC> metric-type 2 tag <TAG>`<br>`end` | OSPF advertises external routes as E2 routes with fixed external metric |
| 17 | Verify OSPF external LSAs are created | Boundary RTR | `show ip ospf database external` | Type 5 external LSAs exist for redistributed prefixes |
| 18 | Verify OSPF downstream route type | OSPF RTR | `show ip route ospf` | Redistributed routes appear as `O E1`, `O E2`, `O N1`, or `O N2` depending on area and metric type |
| 19 | Configure BGP redistribution with route-map policy if needed | Boundary RTR | `conf t`<br>`router bgp <ASN>`<br>` address-family ipv4 unicast`<br>`  redistribute <SOURCE_PROTOCOL> <SOURCE_PROCESS_OR_ASN> route-map RM-REDIST-INTO-BGP`<br>`end` | Source routes are injected into BGP only if route-map policy permits them |
| 20 | Configure BGP default metric only if missing MED behavior must be normalized | Boundary RTR | `conf t`<br>`router bgp <ASN>`<br>` default-metric <METRIC>`<br>`end` | BGP assigns the configured metric when a redistributed route lacks a metric |
| 21 | Verify BGP redistributed route attributes | Boundary RTR | `show bgp ipv4 unicast <PREFIX>` | Redistributed route appears in BGP with expected origin, next hop, metric, and policy attributes |
| 22 | Create a route tag when redistributing from domain A into domain B | Boundary RTR-A | `conf t`<br>`route-map RM-A-INTO-B permit 10`<br>` match ip address prefix-list PL-A-ROUTES`<br>` set tag 10`<br>`end` | Routes injected from domain A are marked with tag `10` |
| 23 | Block tagged routes from being redistributed back into the original domain | Boundary RTR-B | `conf t`<br>`route-map RM-B-INTO-A deny 10`<br>` match tag 10`<br>`route-map RM-B-INTO-A permit 20`<br>`end` | Routes tagged from domain A are denied before feedback into domain A |
| 24 | Attach the tag-blocking route map to the reverse redistribution statement | Boundary RTR-B | `conf t`<br>`router <DESTINATION_PROTOCOL>`<br>` redistribute <SOURCE_PROTOCOL> <SOURCE_PROCESS_OR_ASN> route-map RM-B-INTO-A`<br>`end` | Reverse redistribution blocks feedback routes and permits allowed remaining routes |
| 25 | Verify route tags on redistributed routes | Boundary RTR | `show ip route <PREFIX>` | Route output shows the expected tag if the platform displays it in route detail |
| 26 | Verify tag-based route-map counters | Boundary RTR | `show route-map RM-B-INTO-A` | Deny sequence counter increments for tagged feedback routes |
| 27 | Compare RIB route selection against protocol databases | Boundary RTR | `show ip route <PREFIX>`<br>`show ip eigrp topology <PREFIX>/<LEN>`<br>`show ip ospf database external <PREFIX>`<br>`show bgp ipv4 unicast <PREFIX>` | Confirms whether route is missing from the protocol database or only losing RIB selection |
| 28 | Check administrative distance conflict for same-prefix routes | Boundary RTR | `show ip route <PREFIX>` | Installed route source has the lowest AD for the same prefix |
| 29 | Check OSPF E1 vs E2 path behavior if routing is suboptimal | OSPF RTR | `show ip route <PREFIX>`<br>`show ip ospf database external <PREFIX>` | E1 includes internal cost to ASBR; E2 preserves external metric by default |
| 30 | Check EIGRP external route selection and metric values | EIGRP RTR | `show ip eigrp topology <PREFIX>/<LEN>` | EIGRP external route has valid composite metric and expected successor |
| 31 | Check whether route filtering is blocking redistribution | Boundary RTR | `show route-map`<br>`show ip prefix-list`<br>`show access-lists` | Filter objects permit the expected source prefix |
| 32 | Check whether redistribution references the wrong process ID or ASN | Boundary RTR | `show running-config | section router` | Redistribute command references the correct source protocol process or ASN |
| 33 | Check whether connected interfaces are missing during redistribution | Boundary RTR | `show ip route connected` | Connected routes exist but may require `redistribute connected` or `include-connected` depending on protocol and context |
| 34 | Validate downstream reachability after redistribution | Downstream RTR | `ping <REMOTE_PREFIX_HOST> source <SOURCE_INTERFACE_OR_IP>` | Traffic reaches remote redistributed network |
| 35 | Trace path to detect suboptimal routing or loops | Downstream RTR | `traceroute <REMOTE_PREFIX_HOST> source <SOURCE_IP>` | Path follows intended boundary router and does not loop |
| 36 | Save the working configuration | Boundary RTR | `copy running-config startup-config` | Redistribution policy survives reload |

# EIGRP_Redistribution_Metric_Skeleton

conf t
!
ip prefix-list PL-REDIST-INTO-EIGRP seq 10 permit <PREFIX>/<LEN>
!
route-map RM-REDIST-INTO-EIGRP permit 10
 match ip address prefix-list PL-REDIST-INTO-EIGRP
 set metric <BANDWIDTH> <DELAY> <RELIABILITY> <LOAD> <MTU>
 set tag <TAG>
!
router eigrp <ASN>
 redistribute <SOURCE_PROTOCOL> <SOURCE_PROCESS_OR_ASN> route-map RM-REDIST-INTO-EIGRP
!
end
write memory

# EIGRP_Default_Metric_Skeleton

conf t
!
router eigrp <ASN>
 default-metric <BANDWIDTH> <DELAY> <RELIABILITY> <LOAD> <MTU>
 redistribute <SOURCE_PROTOCOL> <SOURCE_PROCESS_OR_ASN>
!
end
write memory

# OSPF_Redistribution_Metric_Skeleton

conf t
!
ip prefix-list PL-REDIST-INTO-OSPF seq 10 permit <PREFIX>/<LEN>
!
route-map RM-REDIST-INTO-OSPF permit 10
 match ip address prefix-list PL-REDIST-INTO-OSPF
 set tag <TAG>
!
router ospf <PROCESS_ID>
 redistribute <SOURCE_PROTOCOL> <SOURCE_PROCESS_OR_ASN> subnets metric <METRIC> metric-type 1 tag <TAG> route-map RM-REDIST-INTO-OSPF
!
end
write memory

# OSPF_E2_Redistribution_Skeleton

conf t
!
router ospf <PROCESS_ID>
 redistribute <SOURCE_PROTOCOL> <SOURCE_PROCESS_OR_ASN> subnets metric <METRIC> metric-type 2 tag <TAG>
!
end
write memory

# BGP_Redistribution_Metric_Skeleton

conf t
!
ip prefix-list PL-REDIST-INTO-BGP seq 10 permit <PREFIX>/<LEN>
!
route-map RM-REDIST-INTO-BGP permit 10
 match ip address prefix-list PL-REDIST-INTO-BGP
 set metric <MED>
!
router bgp <ASN>
 address-family ipv4 unicast
  redistribute <SOURCE_PROTOCOL> <SOURCE_PROCESS_OR_ASN> route-map RM-REDIST-INTO-BGP
!
end
write memory

# Route_Tag_Loop_Prevention_Skeleton

conf t
!
ip prefix-list PL-DOMAIN-A seq 10 permit <DOMAIN_A_PREFIX>/<LEN>
!
route-map RM-A-INTO-B permit 10
 match ip address prefix-list PL-DOMAIN-A
 set tag 10
!
route-map RM-B-INTO-A deny 10
 match tag 10
!
route-map RM-B-INTO-A permit 20
!
router <PROTOCOL_B>
 redistribute <PROTOCOL_A> <PROCESS_OR_ASN_A> route-map RM-A-INTO-B
!
router <PROTOCOL_A>
 redistribute <PROTOCOL_B> <PROCESS_OR_ASN_B> route-map RM-B-INTO-A
!
end
write memory

# Metric_Redistribution_Troubleshooting_Verification_Commands

show ip protocols
show running-config | section router
show running-config | include redistribute
show route-map
show route-map <ROUTE_MAP_NAME>
show ip prefix-list
show ip prefix-list <PREFIX_LIST_NAME>
show access-lists
show ip route
show ip route <PREFIX>
show ip route <SOURCE_PROTOCOL>
show ip route eigrp
show ip route ospf
show ip route bgp
show ip eigrp neighbors
show ip eigrp topology
show ip eigrp topology <PREFIX>/<LEN>
show ip ospf neighbor
show ip ospf database external
show ip ospf database external <PREFIX>
show ip ospf database nssa-external
show bgp ipv4 unicast summary
show bgp ipv4 unicast
show bgp ipv4 unicast <PREFIX>/<LEN>
show bgp ipv4 unicast neighbors <NEIGHBOR_IP> advertised-routes
show bgp ipv4 unicast neighbors <NEIGHBOR_IP> routes
ping <REMOTE_PREFIX_HOST> source <SOURCE_INTERFACE_OR_IP>
traceroute <REMOTE_PREFIX_HOST> source <SOURCE_IP>

# Metric_Redistribution_Troubleshooting_Rollback

conf t
!
router eigrp <ASN>
 no redistribute <SOURCE_PROTOCOL> <SOURCE_PROCESS_OR_ASN> route-map RM-REDIST-INTO-EIGRP
 no redistribute <SOURCE_PROTOCOL> <SOURCE_PROCESS_OR_ASN> metric <BANDWIDTH> <DELAY> <RELIABILITY> <LOAD> <MTU>
 no default-metric <BANDWIDTH> <DELAY> <RELIABILITY> <LOAD> <MTU>
!
router ospf <PROCESS_ID>
 no redistribute <SOURCE_PROTOCOL> <SOURCE_PROCESS_OR_ASN> subnets metric <METRIC> metric-type 1 tag <TAG> route-map RM-REDIST-INTO-OSPF
 no redistribute <SOURCE_PROTOCOL> <SOURCE_PROCESS_OR_ASN> subnets metric <METRIC> metric-type 2 tag <TAG>
!
router bgp <ASN>
 address-family ipv4 unicast
  no redistribute <SOURCE_PROTOCOL> <SOURCE_PROCESS_OR_ASN> route-map RM-REDIST-INTO-BGP
 exit-address-family
 no default-metric <METRIC>
!
no route-map RM-REDIST-INTO-EIGRP
no route-map RM-REDIST-INTO-OSPF
no route-map RM-REDIST-INTO-BGP
no route-map RM-A-INTO-B
no route-map RM-B-INTO-A
!
no ip prefix-list PL-REDIST-INTO-EIGRP
no ip prefix-list PL-REDIST-INTO-OSPF
no ip prefix-list PL-REDIST-INTO-BGP
no ip prefix-list PL-DOMAIN-A
!
end
write memory

# Metric_Redistribution_Troubleshooting_Failure_Checks

| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| Route is not redistributed at all | Source route is missing from the boundary router RIB | `show ip route <PREFIX>` | Fix the source protocol or source route first |
| Route exists in source protocol but not destination protocol | Missing or incorrect `redistribute` command | `show running-config | include redistribute` | Add redistribution under the correct destination routing process |
| Redistribution references wrong process | Wrong OSPF process ID, EIGRP ASN, or BGP ASN | `show running-config | section router` | Correct the source process or ASN in the `redistribute` command |
| EIGRP redistributed route is missing | Missing EIGRP seed metric | `show running-config | section router eigrp` | Add `metric`, `default-metric`, or route-map `set metric` |
| EIGRP external route has bad path selection | Poor EIGRP seed metric values | `show ip eigrp topology <PREFIX>/<LEN>` | Adjust bandwidth, delay, reliability, load, and MTU values |
| OSPF redistributed subnets are missing | `subnets` keyword omitted | `show running-config | section router ospf` | Add `subnets` to the OSPF redistribution command |
| OSPF path is suboptimal | E2 metric ignores internal cost to ASBR | `show ip route <PREFIX>` and `show ip ospf database external <PREFIX>` | Use `metric-type 1` or tune seed metrics on ASBRs |
| OSPF route appears as E2 when E1 was expected | Metric type not set | `show running-config | section router ospf` | Add `metric-type 1` to redistribution |
| OSPF NSSA route appears as N2 instead of N1 | NSSA external metric type defaulted or configured incorrectly | `show ip route ospf` | Set NSSA redistribution metric type as required |
| BGP route appears with unexpected MED | Redistributed IGP metric or route-map set metric differs from design | `show bgp ipv4 unicast <PREFIX>/<LEN>` | Set MED with route map or normalize with `default-metric` |
| Route is in protocol database but not RIB | Another protocol has better AD for the same prefix | `show ip route <PREFIX>` | Adjust AD carefully or fix the intended route source |
| More specific route wins over redistributed route | Longest prefix match overrides same-prefix AD comparison | `show ip route <DESTINATION_IP>` | Correct overlapping prefix advertisements |
| Redistributed routes loop back into original domain | No tag-based loop prevention | `show ip route <PREFIX>` and `show route-map` | Tag routes on redistribution and deny matching tags in reverse direction |
| Route-map deny blocks all redistributed routes | Missing final permit sequence | `show route-map <ROUTE_MAP_NAME>` | Add an explicit `route-map <NAME> permit <SEQ>` for allowed routes |
| Prefix list denies intended route | Prefix-list mask or `ge`/`le` logic is wrong | `show ip prefix-list <NAME>` | Correct prefix-list entry and sequence |
| ACL match object behaves unexpectedly | ACL wildcard is wrong or ACL is matching packets instead of prefixes in the wrong context | `show access-lists` | Use prefix lists for route prefixes where possible |
| Route-map counters do not increment | Route-map is not attached or route never reaches it | `show route-map <NAME>` and `show running-config | include route-map` | Attach route map to the redistribution command or fix source route |
| Route tag is missing | Tag not set during redistribution or feature does not preserve/display it as expected | `show ip route <PREFIX>` | Add `set tag` or `tag <VALUE>` under redistribution |
| Suboptimal boundary router is selected | Seed metrics differ incorrectly across redistribution points | Compare `show ip route <PREFIX>` on downstream routers | Normalize or intentionally tune metrics at each boundary router |
| Route feedback creates blackhole | A reintroduced route beats the original route | `traceroute <DESTINATION>` and `show ip route <PREFIX>` | Use tags, route filters, or AD changes to block feedback |
| Connected routes are not redistributed | Connected routes are not included in the redistribution source | `show ip route connected` | Add `redistribute connected` or protocol-specific include option where required |
| Static routes are not redistributed | Static route not in RIB or static redistribution missing | `show ip route static` | Ensure static route is installed and configure `redistribute static` |
| Default route is not redistributed | Protocol requires explicit default-origination behavior | `show ip route 0.0.0.0` | Use protocol-specific default-information or default route redistribution carefully |
| Redistribution works on boundary router but not downstream | Neighbor issue, filtering, summarization, or area restriction blocks propagation | Neighbor and database verification commands | Fix adjacency, filtering, area type, or summarization boundary |
| OSPF external LSAs absent in stub area | Stub area blocks Type 5 external LSAs | `show ip ospf` and `show ip ospf database external` | Use NSSA where redistribution is required inside the area |
| Metric change does not affect installed route | Another route source still wins by AD or longer prefix | `show ip route <PREFIX>` | Check AD and prefix length before tuning metrics |
| BGP redistribution pollutes BGP table | Route map is too broad or missing | `show bgp ipv4 unicast` | Apply a restrictive route map and prefix list |
| Rollback leaves redistribution active | Route-map removed but redistribution command remains | `show running-config | include redistribute` | Remove redistribution statements first, then remove policy objects |
| Lab works one way but ping fails | Return route missing after redistribution | Remote router `show ip route <SOURCE_PREFIX>` | Redistribute or advertise reverse path correctly |

# Index

Metric_Redistribution_Troubleshooting.md
Metric_Redistribution_Troubleshooting
Metric_Redistribution_Troubleshooting_Mental_Model
Metric_Redistribution_Troubleshooting_Configuration_Checklist
EIGRP_Redistribution_Metric_Skeleton
EIGRP_Default_Metric_Skeleton
OSPF_Redistribution_Metric_Skeleton
OSPF_E2_Redistribution_Skeleton
BGP_Redistribution_Metric_Skeleton
Route_Tag_Loop_Prevention_Skeleton
Metric_Redistribution_Troubleshooting_Verification_Commands
Metric_Redistribution_Troubleshooting_Rollback
Metric_Redistribution_Troubleshooting_Failure_Checks