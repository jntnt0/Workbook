EIGRP_Summarization_And_Summary_Leak_Map.md

EIGRP_Summarization_And_Summary_Leak_Map

# Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| `_KB_INDEX.md` | Book 3: EIGRP for IP | Points EIGRP summarization, route filtering, metrics, convergence, and troubleshooting to `All_combined_part3.md` |
| `All_combined_part3.md` | Book 3, Chapter 7: Route Filters, Tasks 5-8 | Supports classic EIGRP interface summarization with `ip summary-address eigrp <AS> <network> <mask>` |
| `All_combined_part3.md` | EIGRP summary route verification | Supports the local Null0 discard route created when summarization is configured |
| `All_combined_part3.md` | EIGRP leak-map behavior | Supports `leak-map` behavior and route-map based leaking of selected component routes |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | EIGRP Summarization | Supports classic syntax `ip summary-address eigrp <as-number> <network> <subnet-mask> [leak-map <route-map-name>]` |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | Named Mode Summarization | Supports named-mode syntax under `af-interface <interface-id>` using `summary-address <network> <subnet-mask> [leak-map <route-map-name>]` |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | Summary route behavior | Confirms EIGRP summary routes are interface-specific and advertised only out the configured outgoing interface |
# EIGRP_Summarization_And_Summary_Leak_Map_Mental_Model
| Concept | Operational Meaning |
|---|---|
| EIGRP summarization | Reduces multiple component routes into one aggregate route |
| Component route | A more-specific route that falls inside the summary range |
| Summary route | The aggregate prefix advertised to a neighbor instead of every component route |
| Outgoing-interface behavior | EIGRP summaries are configured on the interface that advertises the summary outward |
| Per-neighbor control | Only neighbors reached through the summary-configured interface receive that summary |
| Component suppression | By default, component routes inside the summary are hidden from the receiving neighbor |
| Null0 discard route | Local summarizing router installs a discard route for the summary to prevent routing loops |
| Summary route AD | EIGRP local summary discard route commonly appears with AD 5 |
| Summary activation | Summary is advertised only when at least one matching component route exists |
| Leak-map | Allows selected component routes to be advertised along with the summary |
| Longest-match effect | Leaked specifics override the summary on receiving routers because longer prefixes win |
| Route-map role | The leak-map references a route-map that selects which component routes escape suppression |
| Prefix-list role | Cleanest way to match exact leaked component prefixes and prefix lengths |
| Wrong leak-map name | If the summary references a nonexistent route-map, only the summary is advertised |
| Empty route-map danger | A route-map permit sequence with no match can leak all component routes |
| Named-mode caveat | `summary-address` must be configured under a specific `af-interface`; not under `af-interface default` |
| Design use | Summarize to shrink routing tables and query scope, then leak only the specifics that need longest-match forwarding |
# EIGRP_Summarization_And_Summary_Leak_Map_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm EIGRP-facing interfaces are operational before summarizing | All routers | `show ip interface brief` | EIGRP-facing interfaces are `up/up` with correct IPv4 addresses |
| 2 | Confirm EIGRP neighbors are stable | All routers | `show ip eigrp neighbors` | Expected neighbors appear with stable uptime and `Q Cnt` of `0` |
| 3 | Confirm baseline EIGRP process state | All routers | `show ip protocols` | Correct AS, router ID, network statements, passive interfaces, K-values, and AD values are shown |
| 4 | Confirm the summarizing router has component routes | Summarizing router | `show ip route <COMPONENT_PREFIX>` | Component routes exist locally as connected, EIGRP, static, or redistributed routes |
| 5 | Confirm the component routes are currently advertised before summarization | Receiving router | `show ip route eigrp` | Receiving router sees the more-specific component routes before summary is applied |
| 6 | Identify the exact summary range | Summarizing router | Planning step | Summary network and mask cleanly cover intended component routes only |
| 7 | Identify the outgoing interface toward the receiving neighbor | Summarizing router | `show ip route <RECEIVER_NEXT_HOP>` | Outgoing interface toward the neighbor is confirmed |
| 8 | Enter global configuration mode | Summarizing router | `configure terminal` | Router enters global configuration mode |
| 9 | Enter the outgoing interface in classic mode | Summarizing router | `interface <OUTGOING_INTERFACE>` | Router enters interface configuration mode |
| 10 | Configure classic EIGRP summarization | Summarizing router | `ip summary-address eigrp <AS_NUMBER> <SUMMARY_NETWORK> <SUMMARY_MASK>` | Router advertises the summary out that interface and suppresses covered specifics |
| 11 | Exit configuration mode | Summarizing router | `end` | Router returns to privileged EXEC mode |
| 12 | Verify local Null0 summary route | Summarizing router | `show ip route <SUMMARY_NETWORK> <SUMMARY_MASK>` | Summary route appears locally via `Null0` |
| 13 | Verify the summary on the receiving router | Receiving router | `show ip route eigrp` | Receiving router sees the summary route instead of all component routes |
| 14 | Verify suppressed component routes are removed from the receiving router | Receiving router | `show ip route <COMPONENT_PREFIX>` | Covered component route is absent unless leaked or learned another way |
| 15 | Create a prefix-list for the component route that should be leaked | Summarizing router | `ip prefix-list <LEAK_PREFIX_LIST> seq 5 permit <LEAKED_PREFIX>/<LENGTH>` | Prefix-list matches the specific component route to leak |
| 16 | Add more leaked component routes if required | Summarizing router | `ip prefix-list <LEAK_PREFIX_LIST> seq 10 permit <LEAKED_PREFIX_2>/<LENGTH>` | Additional permitted component routes are selected for leaking |
| 17 | Verify the leak prefix-list | Summarizing router | `show ip prefix-list <LEAK_PREFIX_LIST>` | Prefix-list contains only the component routes that should leak |
| 18 | Create the leak route-map | Summarizing router | `route-map <LEAK_ROUTE_MAP> permit 10` | Route-map sequence is created |
| 19 | Match the leak prefix-list in the route-map | Summarizing router | `match ip address prefix-list <LEAK_PREFIX_LIST>` | Route-map permits only selected component routes for leak behavior |
| 20 | Verify the route-map before applying it | Summarizing router | `show route-map <LEAK_ROUTE_MAP>` | Route-map references the intended prefix-list |
| 21 | Re-enter the outgoing interface | Summarizing router | `interface <OUTGOING_INTERFACE>` | Router enters the interface where summary is advertised |
| 22 | Add the leak-map to the classic EIGRP summary | Summarizing router | `ip summary-address eigrp <AS_NUMBER> <SUMMARY_NETWORK> <SUMMARY_MASK> leak-map <LEAK_ROUTE_MAP>` | Summary remains advertised, and selected component routes are leaked |
| 23 | Exit configuration mode | Summarizing router | `end` | Router returns to privileged EXEC mode |
| 24 | Verify the summary remains installed locally | Summarizing router | `show ip route <SUMMARY_NETWORK> <SUMMARY_MASK>` | Local summary discard route remains via `Null0` |
| 25 | Verify the receiving router sees the summary route | Receiving router | `show ip route <SUMMARY_NETWORK>` | Summary route is learned through EIGRP |
| 26 | Verify the receiving router sees the leaked component route | Receiving router | `show ip route <LEAKED_PREFIX>` | Leaked component route is present along with the summary |
| 27 | Verify non-leaked component routes remain suppressed | Receiving router | `show ip route <NON_LEAKED_COMPONENT_PREFIX>` | Non-leaked component route is absent unless learned another way |
| 28 | Verify longest-match forwarding behavior | Receiving router | `show ip route <LEAKED_PREFIX>` | Leaked specific route is preferred over the broader summary |
| 29 | Test reachability to the leaked component route | Receiving router or host | `ping <LEAKED_COMPONENT_IP>` | Ping succeeds using the leaked route |
| 30 | Test reachability to a non-leaked component destination if applicable | Receiving router or host | `ping <NON_LEAKED_COMPONENT_IP>` | Traffic follows the summary route if valid, or fails if no valid destination path exists |
| 31 | Trace forwarding toward the leaked component route | Receiving router or host | `traceroute <LEAKED_COMPONENT_IP>` | Path follows the longest-match leaked specific route |
| 32 | Verify neighbor stability after summary change | All affected routers | `show ip eigrp neighbors` | Neighbors remain present with `Q Cnt` of `0` |
| 33 | Verify EIGRP topology after summarization and leak-map | Receiving router | `show ip eigrp topology` | Summary and leaked specifics appear as expected |
| 34 | Save the configuration | Changed routers | `write memory` | Configuration is saved |
# EIGRP_Summarization_And_Summary_Leak_Map_Skeleton
! Baseline verification
show ip interface brief
show ip eigrp neighbors
show ip protocols
show ip route eigrp
show ip eigrp topology
show ip route <COMPONENT_PREFIX>
! Classic EIGRP summary only
configure terminal
 interface <OUTGOING_INTERFACE>
  ip summary-address eigrp <AS_NUMBER> <SUMMARY_NETWORK> <SUMMARY_MASK>
 end
! Verify summary behavior
show ip route <SUMMARY_NETWORK> <SUMMARY_MASK>
show ip route eigrp
show ip eigrp topology
! Create leak prefix-list
configure terminal
 ip prefix-list <LEAK_PREFIX_LIST> seq 5 permit <LEAKED_PREFIX>/<LENGTH>
 ip prefix-list <LEAK_PREFIX_LIST> seq 10 permit <LEAKED_PREFIX_2>/<LENGTH>
end
! Create leak route-map
configure terminal
 route-map <LEAK_ROUTE_MAP> permit 10
  match ip address prefix-list <LEAK_PREFIX_LIST>
end
! Verify leak match logic
show ip prefix-list <LEAK_PREFIX_LIST>
show route-map <LEAK_ROUTE_MAP>
! Apply classic EIGRP summary with leak-map
configure terminal
 interface <OUTGOING_INTERFACE>
  ip summary-address eigrp <AS_NUMBER> <SUMMARY_NETWORK> <SUMMARY_MASK> leak-map <LEAK_ROUTE_MAP>
 end
! Named-mode variant
configure terminal
 router eigrp <PROCESS_NAME>
  address-family ipv4 unicast autonomous-system <AS_NUMBER>
   af-interface <OUTGOING_INTERFACE>
    summary-address <SUMMARY_NETWORK> <SUMMARY_MASK> leak-map <LEAK_ROUTE_MAP>
   exit-af-interface
  exit-address-family
 end
! Final verification
show running-config interface <OUTGOING_INTERFACE>
show running-config | section router eigrp
show ip route <SUMMARY_NETWORK> <SUMMARY_MASK>
show ip route <LEAKED_PREFIX>
show ip route <NON_LEAKED_COMPONENT_PREFIX>
show ip eigrp topology
show ip eigrp neighbors
ping <LEAKED_COMPONENT_IP>
traceroute <LEAKED_COMPONENT_IP>
! Save
write memory
# EIGRP_Summarization_And_Summary_Leak_Map_Verification_Commands
| Command | Purpose | Healthy Output |
|---|---|---|
| `show ip interface brief` | Confirms interface state before summarization | EIGRP-facing interfaces are `up/up` |
| `show ip eigrp neighbors` | Confirms adjacency health | Expected neighbors appear with stable uptime and `Q Cnt` of `0` |
| `show ip protocols` | Confirms EIGRP process state | Correct AS, network statements, passive interfaces, K-values, AD values, and filter/summarization references if shown |
| `show running-config interface <OUTGOING_INTERFACE>` | Confirms classic summary placement | Shows `ip summary-address eigrp <AS> <SUMMARY_NETWORK> <SUMMARY_MASK>` |
| `show running-config | section router eigrp` | Confirms named-mode summary placement if used | Shows `af-interface <interface>` and `summary-address <network> <mask>` |
| `show ip prefix-list <LEAK_PREFIX_LIST>` | Confirms leak match object | Prefix-list permits only the component routes that should leak |
| `show route-map <LEAK_ROUTE_MAP>` | Confirms leak-map route-map logic | Route-map references the intended prefix-list |
| `show ip route <SUMMARY_NETWORK> <SUMMARY_MASK>` | Confirms local summary discard route | Summarizing router shows summary route via `Null0` |
| `show ip route eigrp` | Confirms received summary and leaked specifics | Receiving router sees summary route and selected leaked component routes |
| `show ip route <LEAKED_PREFIX>` | Confirms leaked route is present | Leaked component route appears as a more-specific EIGRP route |
| `show ip route <NON_LEAKED_COMPONENT_PREFIX>` | Confirms suppression of non-leaked specifics | Non-leaked component route is absent unless learned another way |
| `show ip eigrp topology <SUMMARY_NETWORK>/<PREFIX_LENGTH>` | Confirms EIGRP topology entry for the summary | Summary appears in passive state |
| `show ip eigrp topology <LEAKED_PREFIX>/<PREFIX_LENGTH>` | Confirms EIGRP topology entry for the leaked prefix | Leaked prefix appears in passive state |
| `show ip eigrp topology all-links` | Displays all known EIGRP paths | Confirms alternate paths if they exist |
| `ping <LEAKED_COMPONENT_IP>` | Confirms reachability to leaked component route | Ping succeeds |
| `traceroute <LEAKED_COMPONENT_IP>` | Confirms forwarding path to leaked component route | Path follows the expected longest-match route |
# EIGRP_Summarization_And_Summary_Leak_Map_Rollback
! Remove classic summary with leak-map
configure terminal
 interface <OUTGOING_INTERFACE>
  no ip summary-address eigrp <AS_NUMBER> <SUMMARY_NETWORK> <SUMMARY_MASK>
 end
! Remove named-mode summary with leak-map
configure terminal
 router eigrp <PROCESS_NAME>
  address-family ipv4 unicast autonomous-system <AS_NUMBER>
   af-interface <OUTGOING_INTERFACE>
    no summary-address <SUMMARY_NETWORK> <SUMMARY_MASK>
   exit-af-interface
  exit-address-family
 end
! Remove leak route-map if it was created only for this summary
configure terminal
 no route-map <LEAK_ROUTE_MAP>
end
! Remove leak prefix-list if it was created only for this summary
configure terminal
 no ip prefix-list <LEAK_PREFIX_LIST>
end
! Verify rollback
show running-config interface <OUTGOING_INTERFACE>
show running-config | section router eigrp
show ip prefix-list <LEAK_PREFIX_LIST>
show route-map <LEAK_ROUTE_MAP>
show ip route <SUMMARY_NETWORK> <SUMMARY_MASK>
show ip route <LEAKED_PREFIX>
show ip route eigrp
show ip eigrp topology
show ip eigrp neighbors
ping <REMOTE_IP>
traceroute <REMOTE_IP>
! Expected result:
! Summary advertisement is removed.
! Local Null0 summary route is removed.
! Component routes advertise normally again if not blocked by another policy.
write memory
# EIGRP_Summarization_And_Summary_Leak_Map_Failure_Checks
| Symptom | Likely Cause | Check Command | Fix |
|---|---|---|---|
| Summary route is not advertised | No matching component route exists | `show ip route <COMPONENT_PREFIX>` | Ensure at least one component route inside the summary exists locally |
| Summary route appears locally but not on neighbor | Summary applied to wrong outgoing interface | `show running-config interface <OUTGOING_INTERFACE>` | Apply summary on the interface facing the neighbor that should receive it |
| Component routes still appear on receiving router | Summary not applied or leak-map leaks all specifics | `show running-config interface <OUTGOING_INTERFACE>` and `show route-map <LEAK_ROUTE_MAP>` | Correct summary placement and route-map match logic |
| All component routes are leaked | Route-map permit sequence has no match condition | `show route-map <LEAK_ROUTE_MAP>` | Add `match ip address prefix-list <LEAK_PREFIX_LIST>` to leak only selected routes |
| Only summary appears and no leaked specifics appear | Leak-map references a nonexistent route-map | `show running-config interface <OUTGOING_INTERFACE>` and `show route-map <LEAK_ROUTE_MAP>` | Create the referenced route-map or correct the route-map name |
| Summary plus all specifics appear unexpectedly | Route-map references no match condition or bad match object behavior | `show route-map <LEAK_ROUTE_MAP>` and `show ip prefix-list <LEAK_PREFIX_LIST>` | Use a valid prefix-list and match it in the route-map |
| Selected leaked prefix does not appear | Prefix-list does not match exact component prefix | `show ip prefix-list <LEAK_PREFIX_LIST>` and `show ip route <LEAKED_PREFIX>` | Correct prefix and prefix length in the prefix-list |
| Non-leaked component route still reachable | Summary route still covers the destination | `show ip route <NON_LEAKED_COMPONENT_PREFIX>` | This can be normal because the summary still forwards toward the summarizing router |
| Traffic to unused part of summary loops or blackholes | Summary range is too broad or upstream route points back | `show ip route <DESTINATION>` | Tighten the summary range or correct upstream/downstream route design |
| Local summarizing router drops traffic to missing component | Null0 summary discard route is working as designed | `show ip route <SUMMARY_NETWORK> <SUMMARY_MASK>` | Add the missing component route or use a more precise summary |
| Named-mode summary does not apply | Configured under `af-interface default` instead of specific interface | `show running-config | section router eigrp` | Configure `summary-address` under `af-interface <OUTGOING_INTERFACE>` |
| Classic summary does not apply | Configured under router mode instead of interface mode | `show running-config | section router eigrp` and `show running-config interface <OUTGOING_INTERFACE>` | Configure `ip summary-address eigrp` under the outgoing interface |
| Wrong neighbor receives the summary | Summary applied on the wrong interface | `show ip eigrp neighbors` and `show running-config interface <INTERFACE>` | Move summary to the intended neighbor-facing interface |
| EIGRP neighbor flaps after summary change | Expected convergence event or unstable link | `show ip eigrp neighbors` and `show logging | include EIGRP|DUAL` | Wait for convergence, then fix physical/logical instability if flaps continue |
| Query behavior changes after summarization | Summary created a query boundary | `show ip eigrp topology active` | Confirm this is intended; summarization normally reduces query scope |
| Ping to leaked prefix fails | Return route or data-plane issue, not necessarily leak-map | Remote `show ip route <SOURCE_PREFIX>` | Verify reverse path, ACLs, NAT, and interface state |
| Receiving router prefers leaked route over summary | Normal longest-match behavior | `show ip route <LEAKED_PREFIX>` | No fix needed unless leak was not intended |
# Index
# Source_Basis
# EIGRP_Summarization_And_Summary_Leak_Map_Mental_Model
# EIGRP_Summarization_And_Summary_Leak_Map_Configuration_Checklist
# EIGRP_Summarization_And_Summary_Leak_Map_Skeleton
# EIGRP_Summarization_And_Summary_Leak_Map_Verification_Commands
# EIGRP_Summarization_And_Summary_Leak_Map_Rollback
# EIGRP_Summarization_And_Summary_Leak_Map_Failure_Checks