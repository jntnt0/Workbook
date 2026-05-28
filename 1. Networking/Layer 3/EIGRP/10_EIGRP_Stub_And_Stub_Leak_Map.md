EIGRP_Stub_And_Stub_Leak_Map.md

EIGRP_Stub_And_Stub_Leak_Map

# Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| `_KB_INDEX.md` | Book 3: EIGRP for IP | Points EIGRP stub routing, DUAL, convergence, and query control to `All_combined_part3.md` |
| `All_combined_part3.md` | Lab 7-7: EIGRP Stub | Supports `eigrp stub connected`, `summary`, `static`, `redistributed`, `receive-only`, and default `eigrp stub` behavior |
| `All_combined_part3.md` | EIGRP named-mode stub command help | Supports named-mode `eigrp stub` options including `connected`, `summary`, `static`, `redistributed`, `receive-only`, and `leak-map` |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | EIGRP Stub Router | Supports the mental model that a stub router does not advertise EIGRP-learned routes and is not queried when routes go active |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | Example 3-11: Configuring EIGRP Stub Routers | Supports classic and named-mode stub configuration syntax |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | Troubleshooting EIGRP Stub Routing | Supports verification with `show ip protocols`, `show ip eigrp neighbors detail`, and route-table comparison |
# EIGRP_Stub_And_Stub_Leak_Map_Mental_Model
| Concept | Operational Meaning |
|---|---|
| EIGRP stub router | A router that tells neighbors it should not be used as a transit router |
| Query boundary | Upstream routers do not send EIGRP queries to a stub router when routes go active |
| Default stub behavior | `eigrp stub` advertises connected and summary routes by default |
| Connected option | Allows connected routes matched by EIGRP network statements to be advertised |
| Summary option | Allows summary routes to be advertised |
| Static option | Allows static routes to be advertised if they are included in EIGRP policy |
| Redistributed option | Allows redistributed routes to be advertised from the stub |
| Receive-only option | Stub receives routes but advertises no routes; it cannot be combined with other stub options |
| Transit prevention | A stub router does not advertise routes learned from another EIGRP neighbor |
| Stub leak-map | Allows selected dynamic prefixes to be advertised from a stub even though normal stub behavior would suppress them |
| Stub leak-map match object | Usually a prefix-list matched by a route-map |
| Stub leak-map danger | A bad route-map can leak too many routes or no routes at all |
| Neighbor awareness | Stub status is advertised in EIGRP hellos and visible from the neighbor side |
| Adjacency reset | Changing stub options can reset EIGRP adjacency because peer capabilities changed |
| Correct placement | Classic mode places `eigrp stub` under `router eigrp <AS>`; named mode places it under the address family |
| Design rule | Stub belongs on edge or branch routers, not on core transit routers |
| Key warning | If a router has downstream EIGRP routers behind it, plain stub can break reachability unless leak-map, stub-site, or a different design is used |
# EIGRP_Stub_And_Stub_Leak_Map_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm EIGRP-facing interfaces are operational | All routers | `show ip interface brief` | EIGRP-facing interfaces are `up/up` with correct IPv4 addresses |
| 2 | Confirm EIGRP neighbors are stable before stub changes | All routers | `show ip eigrp neighbors` | Expected neighbors appear with stable uptime and `Q Cnt` of `0` |
| 3 | Confirm baseline EIGRP process state | All routers | `show ip protocols` | Correct AS, router ID, network statements, passive interfaces, K-values, and AD values are shown |
| 4 | Record baseline routes before enabling stub | Stub router and upstream router | `show ip route eigrp` | Current EIGRP-learned and advertised routes are documented |
| 5 | Confirm whether the candidate stub router is truly an edge router | Candidate stub router | `show ip eigrp neighbors` | Stub router should not be required as a transit path for other EIGRP routers |
| 6 | Confirm whether downstream EIGRP routers exist behind the candidate stub router | Candidate stub router | `show ip eigrp neighbors` | If downstream routers exist, plain stub may suppress their routes |
| 7 | Enter global configuration mode | Stub router | `configure terminal` | Router enters global configuration mode |
| 8 | Enter classic EIGRP process if using classic mode | Stub router | `router eigrp <AS_NUMBER>` | Router enters EIGRP router configuration mode |
| 9 | Configure normal stub behavior in classic mode | Stub router | `eigrp stub connected summary` | Stub advertises connected and summary routes and blocks transit behavior |
| 10 | Optionally advertise static routes from the stub | Stub router | `eigrp stub connected summary static` | Stub can advertise connected, summary, and static routes |
| 11 | Optionally advertise redistributed routes from the stub | Stub router | `eigrp stub connected summary redistributed` | Stub can advertise connected, summary, and redistributed routes |
| 12 | Configure receive-only stub only when the router should advertise nothing | Stub router | `eigrp stub receive-only` | Stub receives routes but advertises no routes |
| 13 | Avoid combining receive-only with other stub options | Stub router | Planning step | `receive-only` is used by itself |
| 14 | Exit configuration mode | Stub router | `end` | Router returns to privileged EXEC mode |
| 15 | Verify local stub configuration | Stub router | `show ip protocols` | Output identifies the router as an EIGRP stub and lists advertised stub route types |
| 16 | Verify upstream router recognizes the neighbor as a stub | Upstream router | `show ip eigrp neighbors detail` | Neighbor detail shows the peer is a stub |
| 17 | Verify upstream route table after stub configuration | Upstream router | `show ip route eigrp` | Upstream router receives only the permitted stub route types |
| 18 | Verify suppressed transit routes are no longer advertised | Upstream router | `show ip route <SUPPRESSED_PREFIX>` | EIGRP-learned downstream routes behind the stub are absent unless leaked |
| 19 | Create a prefix-list for the dynamic route that should be leaked from the stub | Stub router | `ip prefix-list <LEAK_PREFIX_LIST> seq 5 permit <LEAKED_PREFIX>/<LENGTH>` | Prefix-list matches only the route that should be leaked |
| 20 | Verify the leak prefix-list | Stub router | `show ip prefix-list <LEAK_PREFIX_LIST>` | Prefix-list contains the exact leaked prefix and prefix length |
| 21 | Create the leak route-map | Stub router | `route-map <LEAK_ROUTE_MAP> permit 10` | Route-map sequence is created |
| 22 | Match the leak prefix-list in the route-map | Stub router | `match ip address prefix-list <LEAK_PREFIX_LIST>` | Route-map permits only selected leaked prefixes |
| 23 | Verify the route-map before applying it | Stub router | `show route-map <LEAK_ROUTE_MAP>` | Route-map references the intended prefix-list |
| 24 | Enter classic EIGRP process for stub leak-map if supported by the platform | Stub router | `router eigrp <AS_NUMBER>` | Router enters EIGRP router configuration mode |
| 25 | Verify leak-map is available before applying it | Stub router | `eigrp stub ?` | CLI shows `leak-map` as an available option |
| 26 | Apply classic stub leak-map if supported | Stub router | `eigrp stub connected summary leak-map <LEAK_ROUTE_MAP>` | Stub advertises normal stub routes plus selected leaked dynamic prefixes |
| 27 | Apply named-mode stub leak-map if using named EIGRP | Stub router | `router eigrp <PROCESS_NAME>` then `address-family ipv4 unicast autonomous-system <AS_NUMBER>` then `eigrp stub connected summary leak-map <LEAK_ROUTE_MAP>` | Named-mode stub advertises normal stub routes plus selected leaked dynamic prefixes |
| 28 | Exit configuration mode | Stub router | `end` | Router returns to privileged EXEC mode |
| 29 | Verify local stub leak-map configuration | Stub router | `show running-config | section router eigrp` | `eigrp stub connected summary leak-map <LEAK_ROUTE_MAP>` appears |
| 30 | Verify the leaked route exists on the stub router | Stub router | `show ip route <LEAKED_PREFIX>` | Stub router has the leaked dynamic prefix in its routing table |
| 31 | Verify upstream router receives the leaked route | Upstream router | `show ip route <LEAKED_PREFIX>` | Leaked prefix appears as an EIGRP route upstream |
| 32 | Verify non-leaked dynamic routes remain suppressed | Upstream router | `show ip route <NON_LEAKED_DYNAMIC_PREFIX>` | Non-leaked dynamic route is absent upstream |
| 33 | Verify upstream router still recognizes stub status | Upstream router | `show ip eigrp neighbors detail` | Neighbor still shows stub status |
| 34 | Verify query reduction behavior during a controlled lab failure | Upstream router | `show ip eigrp topology active` | Upstream router should not query the stub for unrelated active routes |
| 35 | Verify EIGRP neighbor recovery after stub changes | All affected routers | `show ip eigrp neighbors` | Neighbors are up with `Q Cnt` of `0` |
| 36 | Test reachability to connected or summary routes advertised by the stub | Upstream router or host | `ping <STUB_CONNECTED_OR_SUMMARY_IP>` | Ping succeeds if return routing exists |
| 37 | Test reachability to the leaked dynamic route | Upstream router or host | `ping <LEAKED_PREFIX_IP>` | Ping succeeds if the route is properly leaked and return path exists |
| 38 | Confirm non-leaked dynamic route behavior | Upstream router or host | `traceroute <NON_LEAKED_DYNAMIC_IP>` | Traffic should fail or use another valid route if one exists |
| 39 | Save the configuration if stub and leak behavior is correct | Changed routers | `write memory` | Configuration is saved |
# EIGRP_Stub_And_Stub_Leak_Map_Skeleton
! Baseline verification
show ip interface brief
show ip eigrp neighbors
show ip eigrp neighbors detail
show ip protocols
show ip route eigrp
show ip eigrp topology
! Classic EIGRP stub
configure terminal
 router eigrp <AS_NUMBER>
  eigrp stub connected summary
 end
! Classic EIGRP stub with static route advertisement
configure terminal
 router eigrp <AS_NUMBER>
  eigrp stub connected summary static
 end
! Classic EIGRP receive-only stub
configure terminal
 router eigrp <AS_NUMBER>
  eigrp stub receive-only
 end
! Named-mode EIGRP stub
configure terminal
 router eigrp <PROCESS_NAME>
  address-family ipv4 unicast autonomous-system <AS_NUMBER>
   eigrp stub connected summary
  exit-address-family
 end
! Verify stub locally and from neighbor side
show ip protocols
show ip eigrp neighbors detail
show ip route eigrp
! Build leak match object
configure terminal
 ip prefix-list <LEAK_PREFIX_LIST> seq 5 permit <LEAKED_PREFIX>/<LENGTH>
 route-map <LEAK_ROUTE_MAP> permit 10
  match ip address prefix-list <LEAK_PREFIX_LIST>
end
! Verify leak objects
show ip prefix-list <LEAK_PREFIX_LIST>
show route-map <LEAK_ROUTE_MAP>
! Confirm leak-map option exists
configure terminal
 router eigrp <AS_NUMBER>
  eigrp stub ?
 end
! Classic EIGRP stub leak-map, if supported
configure terminal
 router eigrp <AS_NUMBER>
  eigrp stub connected summary leak-map <LEAK_ROUTE_MAP>
 end
! Named-mode EIGRP stub leak-map
configure terminal
 router eigrp <PROCESS_NAME>
  address-family ipv4 unicast autonomous-system <AS_NUMBER>
   eigrp stub connected summary leak-map <LEAK_ROUTE_MAP>
  exit-address-family
 end
! Final verification
show running-config | section router eigrp
show ip protocols
show ip eigrp neighbors detail
show ip route <LEAKED_PREFIX>
show ip route <NON_LEAKED_DYNAMIC_PREFIX>
show ip route eigrp
show ip eigrp topology active
ping <LEAKED_PREFIX_IP>
traceroute <LEAKED_PREFIX_IP>
! Save
write memory
# EIGRP_Stub_And_Stub_Leak_Map_Verification_Commands
| Command | Purpose | Healthy Output |
|---|---|---|
| `show ip interface brief` | Confirms interface state before stub changes | EIGRP-facing interfaces are `up/up` |
| `show ip eigrp neighbors` | Confirms adjacency health | Expected neighbors appear with stable uptime and `Q Cnt` of `0` |
| `show ip eigrp neighbors detail` | Confirms neighbor-side stub recognition | Upstream router identifies the peer as a stub |
| `show ip protocols` | Confirms local stub behavior | Output shows EIGRP stub status and advertised route types |
| `show running-config | section router eigrp` | Confirms stub and leak-map placement | Shows `eigrp stub` and optional `leak-map` under the correct EIGRP hierarchy |
| `show ip prefix-list <LEAK_PREFIX_LIST>` | Confirms leak match object | Prefix-list matches only the dynamic route intended for leaking |
| `show route-map <LEAK_ROUTE_MAP>` | Confirms leak route-map logic | Route-map references the intended prefix-list |
| `show ip route eigrp` | Verifies route advertisement results | Upstream router receives only allowed stub routes and leaked routes |
| `show ip route <LEAKED_PREFIX>` | Confirms leaked route availability | Leaked route appears upstream as an EIGRP route |
| `show ip route <NON_LEAKED_DYNAMIC_PREFIX>` | Confirms unwanted dynamic routes remain suppressed | Non-leaked route is absent upstream unless another path exists |
| `show ip eigrp topology active` | Checks query behavior | Stub should not be queried for unrelated active routes |
| `show logging | include EIGRP|DUAL|SIA|stub` | Checks convergence and adjacency events | No repeated neighbor resets or SIA events after convergence |
| `ping <LEAKED_PREFIX_IP>` | Confirms data-plane reachability to leaked route | Ping succeeds if forward and return paths exist |
| `traceroute <LEAKED_PREFIX_IP>` | Confirms forwarding path to leaked route | Path follows expected EIGRP next hops |
# EIGRP_Stub_And_Stub_Leak_Map_Rollback
! Remove classic EIGRP stub behavior
configure terminal
 router eigrp <AS_NUMBER>
  no eigrp stub
 end
! Remove named-mode EIGRP stub behavior
configure terminal
 router eigrp <PROCESS_NAME>
  address-family ipv4 unicast autonomous-system <AS_NUMBER>
   no eigrp stub
  exit-address-family
 end
! Remove leak route-map if created only for this mechanism
configure terminal
 no route-map <LEAK_ROUTE_MAP>
end
! Remove leak prefix-list if created only for this mechanism
configure terminal
 no ip prefix-list <LEAK_PREFIX_LIST>
end
! Verify rollback
show running-config | section router eigrp
show ip prefix-list <LEAK_PREFIX_LIST>
show route-map <LEAK_ROUTE_MAP>
show ip protocols
show ip eigrp neighbors
show ip eigrp neighbors detail
show ip route <LEAKED_PREFIX>
show ip route eigrp
show ip eigrp topology active
ping <REMOTE_IP>
traceroute <REMOTE_IP>
! Expected result:
! Stub status is removed.
! Upstream routers no longer identify the neighbor as a stub.
! Normal EIGRP advertisement behavior returns, unless another filter, summary, or policy blocks routes.
write memory
# EIGRP_Stub_And_Stub_Leak_Map_Failure_Checks
| Symptom | Likely Cause | Check Command | Fix |
|---|---|---|---|
| Upstream router is missing routes behind the stub | Stub router is suppressing EIGRP-learned routes | `show ip protocols` and upstream `show ip route eigrp` | Remove stub, use leak-map, or redesign so the stub is not required as transit |
| Stub router advertises nothing | `eigrp stub receive-only` was configured | `show ip protocols` | Remove receive-only or replace with `eigrp stub connected summary` |
| Receive-only command fails with other options | `receive-only` cannot be combined with other stub options | `show running-config | section router eigrp` | Use `eigrp stub receive-only` by itself |
| Upstream router does not recognize stub status | Stub not configured under the active EIGRP process or address family | `show ip eigrp neighbors detail` and `show running-config | section router eigrp` | Configure `eigrp stub` under the correct EIGRP hierarchy |
| Stub route types are not what expected | Wrong stub options selected | `show ip protocols` | Use `connected`, `summary`, `static`, or `redistributed` as required |
| Connected routes are not advertised from stub | Connected interface is not included in EIGRP | `show ip protocols` and `show ip route connected` | Add the correct `network` statement or named-mode network configuration |
| Static routes are not advertised from stub | Static option alone does not inject static routes without proper EIGRP policy | `show ip route static` and `show ip protocols` | Ensure static route exists and EIGRP is configured to advertise it as intended |
| Redistributed routes are not advertised from stub | `redistributed` option missing or redistribution missing | `show ip protocols` | Add `eigrp stub redistributed` and correct redistribution policy |
| Leaked route does not appear upstream | Prefix-list does not match the leaked route | `show ip prefix-list <LEAK_PREFIX_LIST>` and `show ip route <LEAKED_PREFIX>` | Correct prefix and prefix length |
| Leaked route still does not appear upstream | Route-map is not referenced by stub leak-map | `show running-config | section router eigrp` | Add `leak-map <LEAK_ROUTE_MAP>` to the `eigrp stub` command |
| Too many routes leak upstream | Route-map has no match condition or match object is too broad | `show route-map <LEAK_ROUTE_MAP>` | Add a precise prefix-list match |
| No routes leak even though leak-map exists | Route-map name is wrong or unsupported on platform | `show route-map <LEAK_ROUTE_MAP>` and `eigrp stub ?` | Correct the route-map name or confirm platform support |
| Stub config causes neighbor reset | Normal peer capability change | `show logging | include EIGRP|DUAL` | Wait for reconvergence and confirm neighbor returns with `Q Cnt` of `0` |
| Query reduction is not visible | Testing route has feasible successor or no active event occurred | `show ip eigrp topology active` | Test with a controlled route failure in a lab |
| Core loses reachability after stub is enabled | Stub was configured on the wrong router | `show ip eigrp neighbors detail` and topology review | Remove stub from transit/core routers and place it only on edge/branch routers |
| Ping to leaked prefix fails even though route exists | Return route missing or filtering exists elsewhere | Remote `show ip route <SOURCE_PREFIX>` | Fix reverse path, distribute-list, route-map, summary, ACL, or NAT issue |
| Non-leaked route is still reachable | Alternate route, summary route, or default route exists | `show ip route <NON_LEAKED_DYNAMIC_PREFIX>` | Confirm actual RIB winner before blaming stub behavior |
| Stub leak-map confused with summary leak-map | Wrong mechanism used | `show running-config interface <INTERFACE>` and `show running-config | section router eigrp` | Use `eigrp stub ... leak-map` for stub leaks; use `summary-address ... leak-map` for summary component leaks |
# Index
# Source_Basis
# EIGRP_Stub_And_Stub_Leak_Map_Mental_Model
# EIGRP_Stub_And_Stub_Leak_Map_Configuration_Checklist
# EIGRP_Stub_And_Stub_Leak_Map_Skeleton
# EIGRP_Stub_And_Stub_Leak_Map_Verification_Commands
# EIGRP_Stub_And_Stub_Leak_Map_Rollback
# EIGRP_Stub_And_Stub_Leak_Map_Failure_Checks
