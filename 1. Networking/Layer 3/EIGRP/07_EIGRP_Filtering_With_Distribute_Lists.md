EIGRP_Filtering_With_Distribute_Lists.md

EIGRP_Filtering_With_Distribute_Lists

# Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| `_KB_INDEX.md` | Book 3: EIGRP for IP, Chapter 7: Route Filters | Points EIGRP route filtering to `All_combined_part3.md` |
| `All_combined_part3.md` | Book 3, Chapter 7: Route Filters | Supports inbound and outbound EIGRP distribute-list filtering |
| `All_combined_part3.md` | Table 7-6: Configuring EIGRP Route Filters | Supports `distribute-list <ACL> in`, `distribute-list <ACL> out`, and per-interface variants |
| `All_combined_part3.md` | EIGRP Route Filter Notes | Supports the rule that global and per-interface distribute-lists are combined, not overridden |
| `All_combined_part3.md` | EIGRP Route Filter Behavior | Supports update, reply, and query behavior when routes are filtered |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | Prefix Lists and EIGRP Filtering | Supports `ip prefix-list` plus `distribute-list prefix <PREFIX_LIST> in/out <interface>` syntax |
# EIGRP_Filtering_With_Distribute_Lists_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Distribute-list | A route filter applied to EIGRP updates |
| Inbound distribute-list | Controls which received EIGRP routes are accepted into the local EIGRP decision process |
| Outbound distribute-list | Controls which local EIGRP routes are advertised to neighbors |
| Global distribute-list | Applies to all EIGRP neighbors in the process |
| Per-interface distribute-list | Applies only to EIGRP updates entering or leaving a specific interface |
| ACL-based filtering | Older method that matches network numbers with wildcard masks |
| Prefix-list-based filtering | Cleaner modern method that matches prefix and prefix length |
| Permit statement | Allows the route through the filter |
| Deny statement | Blocks the route from passing the filter |
| Implicit deny | Anything not explicitly permitted is blocked |
| Inbound route removal | Filtered route should disappear from local `show ip route eigrp` |
| Outbound route suppression | Filtered route should disappear from the neighbor’s `show ip route eigrp` |
| Query boundary effect | Filtering can limit which routers know about a prefix and can affect query/reply behavior |
| Dangerous shortcut | Never apply a filter without a final permit statement unless the goal is to block everything |
| Operational warning | Changing distribute-lists or referenced ACLs/prefix-lists can reset EIGRP adjacencies |
# EIGRP_Filtering_With_Distribute_Lists_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm EIGRP interfaces are operational | All routers | `show ip interface brief` | EIGRP-facing interfaces are `up/up` with correct IPv4 addresses |
| 2 | Confirm EIGRP neighbors are stable before filtering | All routers | `show ip eigrp neighbors` | Expected neighbors appear with stable uptime and `Q Cnt` of `0` |
| 3 | Confirm baseline EIGRP process state | All routers | `show ip protocols` | Correct AS, router ID, network statements, passive interfaces, K-values, and AD values are shown |
| 4 | Record baseline EIGRP routes before filtering | Filter router and neighbor routers | `show ip route eigrp` | Target prefixes are visible before the filter is applied |
| 5 | Record baseline topology entries before filtering | Filter router | `show ip eigrp topology` | Target prefixes appear in the EIGRP topology table |
| 6 | Identify the exact prefix to filter | Filter router | `show ip route <PREFIX>` | Prefix, prefix length, next hop, and source protocol are confirmed |
| 7 | Choose inbound or outbound filtering | Filter router | Planning step | Inbound blocks received routes locally; outbound blocks advertised routes toward neighbors |
| 8 | Choose global or per-interface scope | Filter router | Planning step | Global applies process-wide; per-interface applies only on the selected interface |
| 9 | Create a prefix-list denying the target route | Filter router | `ip prefix-list <LIST_NAME> seq 5 deny <PREFIX>/<LENGTH>` | Target prefix is explicitly denied |
| 10 | Add a final permit to avoid blocking all other routes | Filter router | `ip prefix-list <LIST_NAME> seq 100 permit 0.0.0.0/0 le 32` | All other IPv4 prefixes are permitted |
| 11 | Verify the prefix-list before applying it | Filter router | `show ip prefix-list <LIST_NAME>` | Deny and permit lines appear in correct sequence order |
| 12 | Enter classic EIGRP router configuration mode | Filter router | `router eigrp <AS_NUMBER>` | Router enters EIGRP router configuration mode |
| 13 | Apply a global inbound prefix-list filter if filtering received routes from all neighbors | Filter router | `distribute-list prefix <LIST_NAME> in` | Matching received EIGRP routes are blocked locally from all neighbors |
| 14 | Apply a per-interface inbound prefix-list filter if filtering received routes from one interface | Filter router | `distribute-list prefix <LIST_NAME> in <INTERFACE>` | Matching received EIGRP routes are blocked only on that interface |
| 15 | Apply a global outbound prefix-list filter if suppressing advertisements to all neighbors | Filter router | `distribute-list prefix <LIST_NAME> out` | Matching routes are no longer advertised to any EIGRP neighbor |
| 16 | Apply a per-interface outbound prefix-list filter if suppressing advertisements toward one link | Filter router | `distribute-list prefix <LIST_NAME> out <INTERFACE>` | Matching routes are no longer advertised only out that interface |
| 17 | Exit configuration mode | Filter router | `end` | Router returns to privileged EXEC mode |
| 18 | Verify the applied distribute-list | Filter router | `show running-config | section router eigrp` | Correct `distribute-list prefix <LIST_NAME> in/out` line appears |
| 19 | Verify inbound filtering locally | Filter router | `show ip route <PREFIX>` | Filtered prefix is missing from local routing table if inbound filtering is working |
| 20 | Verify local EIGRP route table after inbound filtering | Filter router | `show ip route eigrp` | Denied prefix is absent; permitted EIGRP routes remain |
| 21 | Verify outbound filtering from the neighbor side | Neighbor router | `show ip route <PREFIX>` | Filtered prefix is missing from the neighbor that should no longer receive it |
| 22 | Verify neighbor still receives permitted routes | Neighbor router | `show ip route eigrp` | Non-filtered EIGRP routes remain installed |
| 23 | Verify EIGRP topology behavior after filtering | Filter router and neighbor router | `show ip eigrp topology <PREFIX>/<LENGTH>` | Prefix is absent or treated with infinite metric depending on filter direction and packet type |
| 24 | Verify EIGRP adjacencies recovered after applying the filter | All affected routers | `show ip eigrp neighbors` | Neighbors are present with `Q Cnt` of `0` |
| 25 | Verify route filter hit counters if supported | Filter router | `show ip prefix-list <LIST_NAME>` | Match counters increment for filtered or permitted prefixes |
| 26 | Test reachability to a permitted prefix | Filter router or host | `ping <PERMITTED_REMOTE_IP>` | Ping succeeds |
| 27 | Test reachability to the denied prefix from the filtered perspective | Filter router or affected neighbor | `ping <DENIED_REMOTE_IP>` | Ping fails if the filter intentionally removed the route |
| 28 | Trace the path to confirm forwarding behavior | Filter router or affected neighbor | `traceroute <REMOTE_IP>` | Path follows expected filtered or permitted routing behavior |
| 29 | Save the configuration if the filter is correct | Changed routers | `write memory` | Configuration is saved |
# EIGRP_Filtering_With_Distribute_Lists_Skeleton
! Baseline verification
show ip interface brief
show ip eigrp neighbors
show ip protocols
show ip route eigrp
show ip eigrp topology
show ip route <PREFIX>
! Create prefix-list filter
configure terminal
 ip prefix-list <LIST_NAME> seq 5 deny <PREFIX>/<LENGTH>
 ip prefix-list <LIST_NAME> seq 100 permit 0.0.0.0/0 le 32
end
! Verify prefix-list
show ip prefix-list <LIST_NAME>
! Classic EIGRP inbound filter, global
configure terminal
 router eigrp <AS_NUMBER>
  distribute-list prefix <LIST_NAME> in
 end
! Classic EIGRP inbound filter, per-interface
configure terminal
 router eigrp <AS_NUMBER>
  distribute-list prefix <LIST_NAME> in <INTERFACE>
 end
! Classic EIGRP outbound filter, global
configure terminal
 router eigrp <AS_NUMBER>
  distribute-list prefix <LIST_NAME> out
 end
! Classic EIGRP outbound filter, per-interface
configure terminal
 router eigrp <AS_NUMBER>
  distribute-list prefix <LIST_NAME> out <INTERFACE>
 end
! Named-mode variant if platform supports distribute-list under topology base
configure terminal
 router eigrp <PROCESS_NAME>
  address-family ipv4 unicast autonomous-system <AS_NUMBER>
   topology base
    distribute-list prefix <LIST_NAME> in <INTERFACE>
    ! or
    distribute-list prefix <LIST_NAME> out <INTERFACE>
   exit-af-topology
  exit-address-family
 end
! Verification after filtering
show running-config | section router eigrp
show ip prefix-list <LIST_NAME>
show ip eigrp neighbors
show ip route <PREFIX>
show ip route eigrp
show ip eigrp topology <PREFIX>/<LENGTH>
ping <PERMITTED_REMOTE_IP>
ping <DENIED_REMOTE_IP>
traceroute <REMOTE_IP>
! Save
write memory
# EIGRP_Filtering_With_Distribute_Lists_Verification_Commands
| Command | Purpose | Healthy Output |
|---|---|---|
| `show ip interface brief` | Confirms interface state before filtering | EIGRP-facing interfaces are `up/up` |
| `show ip eigrp neighbors` | Confirms adjacency health before and after applying the filter | Expected neighbors appear with stable uptime and `Q Cnt` of `0` |
| `show ip protocols` | Confirms EIGRP process state and route filter references | Correct AS, network statements, passive interfaces, and distribute-list references |
| `show running-config | section router eigrp` | Confirms filter placement | Shows `distribute-list prefix <LIST_NAME> in/out` under the intended EIGRP process |
| `show running-config | include prefix-list` | Confirms prefix-list lines | Deny and permit statements appear with correct sequence numbers |
| `show ip prefix-list <LIST_NAME>` | Verifies prefix-list logic and counters | Denied prefix and final permit statement are present |
| `show ip route eigrp` | Confirms EIGRP route installation after filtering | Denied routes are absent from affected routers; permitted routes remain |
| `show ip route <PREFIX>` | Confirms exact route result | Filtered prefix is missing where intended or still present where allowed |
| `show ip eigrp topology <PREFIX>/<LENGTH>` | Confirms EIGRP topology behavior | Prefix appears or disappears according to filter direction and scope |
| `show ip eigrp topology all-links` | Checks all known EIGRP paths | Confirms whether alternate EIGRP knowledge exists |
| `show logging | include EIGRP|DUAL` | Checks for neighbor reset or convergence events | No repeated instability after applying the filter |
| `ping <PERMITTED_REMOTE_IP>` | Confirms permitted routes still forward | Ping succeeds |
| `ping <DENIED_REMOTE_IP>` | Confirms denied route behavior | Ping fails from the filtered perspective if no alternate route exists |
| `traceroute <REMOTE_IP>` | Confirms forwarding path after filtering | Path follows expected post-filter routing behavior |
# EIGRP_Filtering_With_Distribute_Lists_Rollback
! Remove classic-mode global inbound filter
configure terminal
 router eigrp <AS_NUMBER>
  no distribute-list prefix <LIST_NAME> in
 end
! Remove classic-mode per-interface inbound filter
configure terminal
 router eigrp <AS_NUMBER>
  no distribute-list prefix <LIST_NAME> in <INTERFACE>
 end
! Remove classic-mode global outbound filter
configure terminal
 router eigrp <AS_NUMBER>
  no distribute-list prefix <LIST_NAME> out
 end
! Remove classic-mode per-interface outbound filter
configure terminal
 router eigrp <AS_NUMBER>
  no distribute-list prefix <LIST_NAME> out <INTERFACE>
 end
! Remove named-mode filter if configured
configure terminal
 router eigrp <PROCESS_NAME>
  address-family ipv4 unicast autonomous-system <AS_NUMBER>
   topology base
    no distribute-list prefix <LIST_NAME> in <INTERFACE>
    no distribute-list prefix <LIST_NAME> out <INTERFACE>
   exit-af-topology
  exit-address-family
 end
! Remove prefix-list if it was created only for this filter
configure terminal
 no ip prefix-list <LIST_NAME>
end
! Verify rollback
show running-config | section router eigrp
show ip prefix-list <LIST_NAME>
show ip eigrp neighbors
show ip route <PREFIX>
show ip route eigrp
show ip eigrp topology <PREFIX>/<LENGTH>
ping <REMOTE_IP>
traceroute <REMOTE_IP>
write memory
# EIGRP_Filtering_With_Distribute_Lists_Failure_Checks
| Symptom | Likely Cause | Check Command | Fix |
|---|---|---|---|
| All EIGRP routes disappear | Missing final permit statement | `show ip prefix-list <LIST_NAME>` | Add `ip prefix-list <LIST_NAME> seq 100 permit 0.0.0.0/0 le 32` |
| Target prefix is still present locally | Filter applied outbound instead of inbound | `show running-config | section router eigrp` | Apply `distribute-list prefix <LIST_NAME> in` on the router where the route should be blocked |
| Neighbor still receives target prefix | Filter applied inbound instead of outbound | `show running-config | section router eigrp` | Apply `distribute-list prefix <LIST_NAME> out <INTERFACE>` toward the neighbor |
| Prefix-list does not match the route | Wrong prefix or prefix length | `show ip route <PREFIX>` and `show ip prefix-list <LIST_NAME>` | Match the exact prefix and mask length |
| Filter affects too many neighbors | Global distribute-list used accidentally | `show running-config | section router eigrp` | Replace global filter with per-interface distribute-list |
| Filter affects too few neighbors | Per-interface filter used when global filter was intended | `show running-config | section router eigrp` | Apply the filter globally if the policy should affect all neighbors |
| EIGRP adjacency resets after filter edit | Expected behavior when distribute-list or referenced list changes | `show ip eigrp neighbors` and `show logging | include EIGRP` | Wait for adjacency recovery and verify `Q Cnt` returns to `0` |
| Prefix disappears from one router but remains elsewhere | Inbound filtering only affects local route acceptance | `show ip route <PREFIX>` on multiple routers | Apply outbound filtering closer to the source if propagation must stop |
| Prefix is blocked from routing table but still appears in some topology behavior | EIGRP packet behavior depends on filter direction and packet type | `show ip eigrp topology <PREFIX>/<LENGTH>` | Verify whether the filter is inbound, outbound, global, or per-interface |
| Query behavior changes unexpectedly | Filter created a query boundary | `show ip eigrp topology active` | Confirm this is intended or redesign filtering/summarization boundaries |
| Named-mode filter does not apply | Command placed in wrong hierarchy | `show running-config | section router eigrp` | Place the filter under the correct address-family topology context for the platform |
| ACL-based filter gives strange results | Standard ACL wildcard does not precisely match prefix length | `show access-lists` and `show ip route` | Prefer prefix-lists for route filtering |
| Route still works after denied prefix is removed | Alternate route exists through another protocol or default route | `show ip route <DENIED_PREFIX>` | Confirm actual RIB winner and remove unintended alternate path if necessary |
| Ping fails to permitted prefix | Return route or unrelated forwarding issue | `show ip route <SOURCE_PREFIX>` on remote router | Fix reverse path, ACL, NAT, or interface issue |
# Index
# Source_Basis
# EIGRP_Filtering_With_Distribute_Lists_Mental_Model
# EIGRP_Filtering_With_Distribute_Lists_Configuration_Checklist
# EIGRP_Filtering_With_Distribute_Lists_Skeleton
# EIGRP_Filtering_With_Distribute_Lists_Verification_Commands
# EIGRP_Filtering_With_Distribute_Lists_Rollback
# EIGRP_Filtering_With_Distribute_Lists_Failure_Checks