EIGRP_Filtering_With_Route_Maps.md

EIGRP_Filtering_With_Route_Maps

# Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| `_KB_INDEX.md` | Book 3: EIGRP for IP, Chapter 7: Route Filters | Points EIGRP route filtering to `All_combined_part3.md` |
| `All_combined_part3.md` | Book 3, Chapter 7: Route Filters | Supports inbound and outbound EIGRP route filtering behavior |
| `All_combined_part3.md` | EIGRP Route Filters | Supports global and per-interface route filters, route-filter query-boundary behavior, and adjacency reset caveats |
| `All_combined_part3.md` | Prefix Lists, Improved Route Filters | Supports using prefix-lists as cleaner route-matching tools for route filters |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | EIGRP Filtering | Supports `distribute-list route-map <ROUTE_MAP_NAME> in/out <interface>` syntax |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | Route Maps and Conditional Forwarding | Supports route-map sequence logic, match behavior, permit/deny behavior, and first-match processing |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | Troubleshooting Route Filtering | Supports verification with `show ip prefix-list`, `show route-map`, `show ip protocols`, and `show ip route` |
# EIGRP_Filtering_With_Route_Maps_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Route-map filtering | Uses route-map permit/deny sequences to decide which EIGRP routes pass or get blocked |
| Distribute-list attachment | The route-map does nothing until referenced by `distribute-list route-map` under EIGRP |
| Inbound route-map filter | Controls which EIGRP routes are accepted from neighbors into the local routing process |
| Outbound route-map filter | Controls which EIGRP routes are advertised to neighbors |
| Global filter | Applies to all EIGRP neighbors in the process |
| Per-interface filter | Applies only to routes entering or leaving a specific interface |
| Route-map sequence order | Processed from lowest sequence number to highest |
| First-match behavior | Once a route matches a route-map sequence, processing stops |
| Route-map permit sequence | Allows matching routes through the distribute-list |
| Route-map deny sequence | Blocks matching routes from passing the distribute-list |
| Prefix-list match object | Usually the cleanest way to identify exact route prefixes and prefix lengths |
| ACL match object | Works, but is weaker for prefix-length matching |
| Match object deny lines | A prefix-list or ACL deny does not equal a route-map deny; it usually means the route does not match that route-map sequence |
| Implicit deny | A route-map has an implicit deny at the end unless an explicit final permit sequence is configured |
| Blocking one prefix safely | Use a route-map deny sequence matching the blocked prefix, then a route-map permit sequence for everything else |
| Allowing only selected prefixes | Use route-map permit sequences for allowed prefixes and let the implicit deny block everything else |
| Query boundary effect | EIGRP route filters can create query boundaries because filtered prefixes may not exist in the topology table |
| Operational warning | Changing distribute-lists or referenced match objects can reset affected EIGRP adjacencies |
# EIGRP_Filtering_With_Route_Maps_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm EIGRP-facing interfaces are operational | All routers | `show ip interface brief` | EIGRP-facing interfaces are `up/up` with correct IPv4 addresses |
| 2 | Confirm EIGRP neighbors are stable before applying route policy | All routers | `show ip eigrp neighbors` | Expected neighbors appear with stable uptime and `Q Cnt` of `0` |
| 3 | Confirm baseline EIGRP process state | All routers | `show ip protocols` | Correct AS, router ID, network statements, passive interfaces, K-values, and AD values are shown |
| 4 | Record baseline EIGRP routes before filtering | Filter router and affected neighbors | `show ip route eigrp` | Target prefixes are visible before the route-map filter is applied |
| 5 | Record baseline EIGRP topology before filtering | Filter router and affected neighbors | `show ip eigrp topology` | Target prefixes appear in the topology table before filtering |
| 6 | Identify the exact route to block or permit | Filter router | `show ip route <PREFIX>` | Prefix, prefix length, next hop, metric, and source protocol are confirmed |
| 7 | Decide whether the route-map should block selected routes or allow only selected routes | Filter router | Planning step | Filter intent is clear before writing the route-map |
| 8 | Decide whether filtering should be inbound or outbound | Filter router | Planning step | Inbound affects local route acceptance; outbound affects neighbor advertisement |
| 9 | Decide whether filtering should be global or per-interface | Filter router | Planning step | Global applies to all EIGRP neighbors; per-interface applies only to one link |
| 10 | Create a prefix-list that matches the target prefix | Filter router | `ip prefix-list <MATCH_LIST> seq 5 permit <PREFIX>/<LENGTH>` | Prefix-list matches the route that will be processed by the route-map |
| 11 | Verify the prefix-list match object | Filter router | `show ip prefix-list <MATCH_LIST>` | Prefix-list contains the expected prefix and prefix length |
| 12 | Create a route-map deny sequence to block the matched route | Filter router | `route-map <ROUTE_MAP_NAME> deny 10` | Route-map sequence is created with deny action |
| 13 | Attach the prefix-list to the deny sequence | Filter router | `match ip address prefix-list <MATCH_LIST>` | Routes matching the prefix-list are blocked by the route-map deny sequence |
| 14 | Add a final permit sequence so all other routes are allowed | Filter router | `route-map <ROUTE_MAP_NAME> permit 100` | Non-matching routes are permitted instead of being caught by implicit deny |
| 15 | Verify route-map logic before applying it to EIGRP | Filter router | `show route-map <ROUTE_MAP_NAME>` | Deny sequence references the prefix-list and final permit sequence exists |
| 16 | Enter classic EIGRP router configuration mode if using classic mode | Filter router | `router eigrp <AS_NUMBER>` | Router enters EIGRP router configuration mode |
| 17 | Apply the route-map as a global inbound filter if blocking received routes from all neighbors | Filter router | `distribute-list route-map <ROUTE_MAP_NAME> in` | Matching received EIGRP routes are blocked from all neighbors |
| 18 | Apply the route-map as a per-interface inbound filter if blocking received routes on one link | Filter router | `distribute-list route-map <ROUTE_MAP_NAME> in <INTERFACE>` | Matching received EIGRP routes are blocked only on that interface |
| 19 | Apply the route-map as a global outbound filter if blocking advertisements to all neighbors | Filter router | `distribute-list route-map <ROUTE_MAP_NAME> out` | Matching EIGRP routes are not advertised to any neighbor |
| 20 | Apply the route-map as a per-interface outbound filter if blocking advertisements on one link | Filter router | `distribute-list route-map <ROUTE_MAP_NAME> out <INTERFACE>` | Matching EIGRP routes are not advertised out that interface |
| 21 | Apply the route-map in named mode if the router is using named EIGRP | Filter router | `topology base` then `distribute-list route-map <ROUTE_MAP_NAME> in <INTERFACE>` | Named-mode filter is applied under address-family topology base |
| 22 | Exit configuration mode | Filter router | `end` | Router returns to privileged EXEC mode |
| 23 | Verify the route-map is attached to EIGRP | Filter router | `show running-config | section router eigrp` | Correct `distribute-list route-map <ROUTE_MAP_NAME> in/out` line appears |
| 24 | Verify route-map sequence counters | Filter router | `show route-map <ROUTE_MAP_NAME>` | Match counters increment after EIGRP processes the affected route |
| 25 | Verify prefix-list counters if supported | Filter router | `show ip prefix-list <MATCH_LIST>` | Prefix-list match counters increment for the target prefix |
| 26 | Verify inbound filtering locally | Filter router | `show ip route <PREFIX>` | Target prefix is absent locally if inbound filtering is working and no alternate route exists |
| 27 | Verify outbound filtering from the neighbor side | Affected neighbor | `show ip route <PREFIX>` | Target prefix is absent from the neighbor that should no longer receive it |
| 28 | Verify permitted EIGRP routes remain | Filter router and affected neighbors | `show ip route eigrp` | Non-filtered EIGRP routes remain installed |
| 29 | Verify topology table impact | Filter router and affected neighbors | `show ip eigrp topology <PREFIX>/<LENGTH>` | Filtered prefix is absent or treated as unreachable depending on filter direction |
| 30 | Verify EIGRP neighbor recovery after policy change | All affected routers | `show ip eigrp neighbors` | Neighbors are present with `Q Cnt` of `0` |
| 31 | Test reachability to a permitted route | Test router or host | `ping <PERMITTED_REMOTE_IP>` | Ping succeeds |
| 32 | Test reachability to the denied route from the filtered side | Test router or host | `ping <DENIED_REMOTE_IP>` | Ping fails if no alternate route exists |
| 33 | Trace the forwarding path after filtering | Test router or host | `traceroute <REMOTE_IP>` | Path follows expected post-filter behavior |
| 34 | Save the configuration if the route-map filter is correct | Changed routers | `write memory` | Configuration is saved |
# EIGRP_Filtering_With_Route_Maps_Skeleton
! Baseline verification
show ip interface brief
show ip eigrp neighbors
show ip protocols
show ip route eigrp
show ip eigrp topology
show ip route <PREFIX>
! Match the route that should be blocked
configure terminal
 ip prefix-list <MATCH_LIST> seq 5 permit <PREFIX>/<LENGTH>
end
! Verify the match object
show ip prefix-list <MATCH_LIST>
! Route-map model: block the matched prefix, permit everything else
configure terminal
 route-map <ROUTE_MAP_NAME> deny 10
  match ip address prefix-list <MATCH_LIST>
 route-map <ROUTE_MAP_NAME> permit 100
end
! Verify route-map logic before applying it
show route-map <ROUTE_MAP_NAME>
! Classic EIGRP, global inbound filter
configure terminal
 router eigrp <AS_NUMBER>
  distribute-list route-map <ROUTE_MAP_NAME> in
 end
! Classic EIGRP, per-interface inbound filter
configure terminal
 router eigrp <AS_NUMBER>
  distribute-list route-map <ROUTE_MAP_NAME> in <INTERFACE>
 end
! Classic EIGRP, global outbound filter
configure terminal
 router eigrp <AS_NUMBER>
  distribute-list route-map <ROUTE_MAP_NAME> out
 end
! Classic EIGRP, per-interface outbound filter
configure terminal
 router eigrp <AS_NUMBER>
  distribute-list route-map <ROUTE_MAP_NAME> out <INTERFACE>
 end
! Named-mode EIGRP variant
configure terminal
 router eigrp <PROCESS_NAME>
  address-family ipv4 unicast autonomous-system <AS_NUMBER>
   topology base
    distribute-list route-map <ROUTE_MAP_NAME> in <INTERFACE>
    ! or
    distribute-list route-map <ROUTE_MAP_NAME> out <INTERFACE>
   exit-af-topology
  exit-address-family
 end
! Verification after applying filter
show running-config | section router eigrp
show ip prefix-list <MATCH_LIST>
show route-map <ROUTE_MAP_NAME>
show ip eigrp neighbors
show ip route <PREFIX>
show ip route eigrp
show ip eigrp topology <PREFIX>/<LENGTH>
ping <PERMITTED_REMOTE_IP>
ping <DENIED_REMOTE_IP>
traceroute <REMOTE_IP>
! Save
write memory
# EIGRP_Filtering_With_Route_Maps_Verification_Commands
| Command | Purpose | Healthy Output |
|---|---|---|
| `show ip interface brief` | Confirms interface state before applying route policy | EIGRP-facing interfaces are `up/up` |
| `show ip eigrp neighbors` | Confirms neighbor health before and after filtering | Expected neighbors appear with stable uptime and `Q Cnt` of `0` |
| `show ip protocols` | Confirms EIGRP process state and filtering references | Shows correct AS, router ID, network statements, passive interfaces, and route filtering references |
| `show running-config | section router eigrp` | Confirms filter placement | Shows `distribute-list route-map <ROUTE_MAP_NAME> in/out` under the intended EIGRP hierarchy |
| `show ip prefix-list <MATCH_LIST>` | Verifies the prefix-list match object | Prefix-list contains the exact prefix and prefix length being matched |
| `show route-map <ROUTE_MAP_NAME>` | Verifies route-map sequence logic and counters | Deny sequence matches the intended prefix-list and final permit sequence exists |
| `show ip route <PREFIX>` | Confirms exact route result after filtering | Filtered prefix is absent where intended or still present where allowed |
| `show ip route eigrp` | Confirms installed EIGRP routes after filtering | Denied route is absent from affected routers; permitted EIGRP routes remain |
| `show ip eigrp topology <PREFIX>/<LENGTH>` | Checks EIGRP topology table effect | Prefix is absent, unreachable, or still present according to filter direction and scope |
| `show ip eigrp topology all-links` | Displays all known EIGRP paths | Confirms whether alternate EIGRP knowledge still exists |
| `show logging | include EIGRP|DUAL` | Checks for neighbor resets or convergence problems after route-map changes | No recurring instability after the policy settles |
| `ping <PERMITTED_REMOTE_IP>` | Confirms permitted routes still forward | Ping succeeds |
| `ping <DENIED_REMOTE_IP>` | Confirms denied route behavior | Ping fails from the filtered perspective if no alternate route exists |
| `traceroute <REMOTE_IP>` | Confirms forwarding path after filtering | Path follows expected post-filter routing behavior |
# EIGRP_Filtering_With_Route_Maps_Rollback
! Remove classic-mode global inbound route-map filter
configure terminal
 router eigrp <AS_NUMBER>
  no distribute-list route-map <ROUTE_MAP_NAME> in
 end
! Remove classic-mode per-interface inbound route-map filter
configure terminal
 router eigrp <AS_NUMBER>
  no distribute-list route-map <ROUTE_MAP_NAME> in <INTERFACE>
 end
! Remove classic-mode global outbound route-map filter
configure terminal
 router eigrp <AS_NUMBER>
  no distribute-list route-map <ROUTE_MAP_NAME> out
 end
! Remove classic-mode per-interface outbound route-map filter
configure terminal
 router eigrp <AS_NUMBER>
  no distribute-list route-map <ROUTE_MAP_NAME> out <INTERFACE>
 end
! Remove named-mode route-map filter if configured
configure terminal
 router eigrp <PROCESS_NAME>
  address-family ipv4 unicast autonomous-system <AS_NUMBER>
   topology base
    no distribute-list route-map <ROUTE_MAP_NAME> in <INTERFACE>
    no distribute-list route-map <ROUTE_MAP_NAME> out <INTERFACE>
   exit-af-topology
  exit-address-family
 end
! Remove route-map if it was created only for this filter
configure terminal
 no route-map <ROUTE_MAP_NAME>
end
! Remove prefix-list if it was created only for this filter
configure terminal
 no ip prefix-list <MATCH_LIST>
end
! Verify rollback
show running-config | section router eigrp
show ip prefix-list <MATCH_LIST>
show route-map <ROUTE_MAP_NAME>
show ip eigrp neighbors
show ip route <PREFIX>
show ip route eigrp
show ip eigrp topology <PREFIX>/<LENGTH>
ping <REMOTE_IP>
traceroute <REMOTE_IP>
write memory
# EIGRP_Filtering_With_Route_Maps_Failure_Checks
| Symptom | Likely Cause | Check Command | Fix |
|---|---|---|---|
| All EIGRP routes disappear | Route-map has no final permit sequence | `show route-map <ROUTE_MAP_NAME>` | Add `route-map <ROUTE_MAP_NAME> permit 100` |
| Target prefix is still present locally | Filter applied outbound when inbound was needed | `show running-config | section router eigrp` | Apply `distribute-list route-map <ROUTE_MAP_NAME> in` on the router that should reject the route |
| Neighbor still receives target prefix | Filter applied inbound when outbound was needed | `show running-config | section router eigrp` | Apply `distribute-list route-map <ROUTE_MAP_NAME> out <INTERFACE>` toward the neighbor |
| Prefix-list matches nothing | Wrong prefix or prefix length | `show ip route <PREFIX>` and `show ip prefix-list <MATCH_LIST>` | Correct the prefix-list to match the exact route |
| Route-map sequence counter never increments | Match object does not match route or route-map is not applied | `show route-map <ROUTE_MAP_NAME>` and `show running-config | section router eigrp` | Fix the match condition or attach the route-map to EIGRP |
| Route-map exists but has no effect | Route-map was configured but not referenced by distribute-list | `show running-config | section router eigrp` | Add `distribute-list route-map <ROUTE_MAP_NAME> in/out` |
| Route is accidentally permitted | Blocking prefix-list used `deny` instead of `permit` inside the match object | `show ip prefix-list <MATCH_LIST>` and `show route-map <ROUTE_MAP_NAME>` | Use prefix-list `permit` to match the route, then route-map `deny` to block it |
| Route is accidentally blocked | Implicit route-map deny caught unmatched routes | `show route-map <ROUTE_MAP_NAME>` | Add an explicit final permit route-map sequence |
| Filter affects too many neighbors | Global distribute-list used accidentally | `show running-config | section router eigrp` | Replace global filter with per-interface filter |
| Filter affects too few neighbors | Per-interface filter used when global policy was intended | `show running-config | section router eigrp` | Apply the route-map globally under EIGRP |
| EIGRP adjacency resets after route-map or match-object edit | Expected behavior from route-filter change | `show ip eigrp neighbors` and `show logging | include EIGRP` | Wait for recovery and verify `Q Cnt` returns to `0` |
| Prefix remains reachable after being filtered | Alternate route exists through another protocol or default route | `show ip route <PREFIX>` | Confirm actual RIB winner and remove unintended alternate path if needed |
| Named-mode filter does not apply | Filter placed outside `topology base` | `show running-config | section router eigrp` | Put `distribute-list route-map` under address-family `topology base` |
| Classic-mode filter does not apply | Filter placed under wrong router process or wrong AS | `show running-config | section router eigrp` | Place the filter under the active `router eigrp <AS_NUMBER>` process |
| Route-map set command does not change route attributes | EIGRP distribute-list route-map is being used for filtering, not general attribute manipulation | `show route-map <ROUTE_MAP_NAME>` and `show ip route <PREFIX>` | Use route-map permit/deny for filtering; use redistribution policy when attribute setting is required |
| Query behavior changes after filtering | Route-map filter created a query boundary | `show ip eigrp topology active` | Confirm this is intended or redesign filtering/summarization boundary |
| Ping to permitted prefix fails | Issue is not the route-map filter | `show ip route <PERMITTED_PREFIX>` and remote `show ip route <SOURCE_PREFIX>` | Check reverse route, ACL, NAT, interface state, or next-hop reachability |
# Index
# Source_Basis
# EIGRP_Filtering_With_Route_Maps_Mental_Model
# EIGRP_Filtering_With_Route_Maps_Configuration_Checklist
# EIGRP_Filtering_With_Route_Maps_Skeleton
# EIGRP_Filtering_With_Route_Maps_Verification_Commands
# EIGRP_Filtering_With_Route_Maps_Rollback
# EIGRP_Filtering_With_Route_Maps_Failure_Checks