EIGRP_External_Route_Path_Selection.md

EIGRP_External_Route_Path_Selection

# Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| `_KB_INDEX.md` | Book 3: EIGRP for IP | Points EIGRP redistribution, external routes, metrics, DUAL, administrative distance, and troubleshooting to `All_combined_part3.md` |
| `All_combined_part3.md` | Book 3, Chapter 9: Integrating EIGRP with Other Enterprise Routing Protocols | Supports EIGRP external route behavior after redistribution |
| `All_combined_part3.md` | EIGRP redistribution and loop-prevention examples | Supports route tagging, seed metrics, and redistribution control |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | EIGRP Installed Routes | Confirms internal EIGRP routes use `D` and external EIGRP routes use `D EX` |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | EIGRP Administrative Distance | Confirms internal EIGRP AD is 90 and external EIGRP AD is 170 |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | Redistribution Troubleshooting | Supports the issue where OSPF AD 110 can beat EIGRP external AD 170 and cause redistribution feedback problems |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | Route Maps and Seed Metrics | Supports setting EIGRP external metrics with route-maps during redistribution |
# EIGRP_External_Route_Path_Selection_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Internal EIGRP route | Route originated inside the EIGRP AS; appears as `D`; default AD is 90 |
| External EIGRP route | Route redistributed into EIGRP from another protocol, static, connected, or another EIGRP AS; appears as `D EX`; default AD is 170 |
| RIB selection | The routing table chooses the best source by longest match first, then administrative distance, then metric |
| EIGRP topology table | EIGRP can still know a route even when that route is not installed in the routing table |
| Redistribution dependency | A route must usually be in the RIB before it can be redistributed into another protocol |
| External AD problem | OSPF AD 110 can beat EIGRP external AD 170 for the same prefix |
| Route feedback loop | A redistributed route can leave EIGRP, return through another protocol with better AD, replace the original, then withdraw and repeat |
| Seed metric | The metric assigned when a route is injected into EIGRP from another protocol |
| EIGRP external metric | Bandwidth, delay, reliability, load, and MTU supplied during redistribution |
| Route-map metric control | Lets you set different EIGRP seed metrics for different redistributed prefixes |
| Route tag | Marks redistributed routes so they can be filtered later and not redistributed back into their source protocol |
| External path tuning | Usually done with seed metrics, route tags, route filtering, and AD control |
| Dangerous shortcut | Do not fix external path selection by blindly changing AD everywhere; use targeted route-maps and prefix filters where possible |
| Clean design rule | Prefer one clear redistribution boundary per direction; if multiple boundaries exist, standardize tags and metrics |
# EIGRP_External_Route_Path_Selection_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm routed interfaces are operational | All routers | `show ip interface brief` | Routing interfaces are `up/up` with correct IPv4 addresses |
| 2 | Confirm EIGRP neighbor stability | EIGRP routers | `show ip eigrp neighbors` | Expected neighbors appear with stable uptime and `Q Cnt` of `0` |
| 3 | Confirm other protocol neighbors if redistribution is involved | Boundary routers | `show ip ospf neighbor` or `show ip protocols` | OSPF, RIP, or other routing protocols are stable before redistribution tuning |
| 4 | Confirm baseline EIGRP process state | EIGRP routers | `show ip protocols` | Correct AS, router ID, network statements, passive interfaces, K-values, and AD values are shown |
| 5 | Identify external EIGRP routes currently installed | EIGRP routers | `show ip route eigrp` | Redistributed routes appear as `D EX` |
| 6 | Inspect one target external prefix | Test router | `show ip route <PREFIX>` | Output shows `Known via "eigrp <AS>"`, distance `170`, metric, route type external, next hop, and tag if present |
| 7 | Check whether the same prefix exists in the EIGRP topology table | Test router | `show ip eigrp topology <PREFIX>/<LENGTH>` | Prefix appears in EIGRP topology even if another protocol owns the RIB |
| 8 | Check all EIGRP paths for the external prefix | Test router | `show ip eigrp topology all-links` | All known EIGRP candidate paths are visible |
| 9 | Confirm whether EIGRP external lost to another protocol | Test router | `show ip route <PREFIX>` | If OSPF, RIP, static, or another source owns the route, compare AD values |
| 10 | Identify the redistribution point creating the external route | Boundary router | `show ip protocols` | Output shows which protocol is redistributed into EIGRP |
| 11 | Confirm current redistribution syntax | Boundary router | `show running-config | section router eigrp` | Redistributed protocol, seed metric, and route-map references are visible |
| 12 | Confirm current route-map policy if one exists | Boundary router | `show route-map` | Existing match, set metric, and set tag logic is visible |
| 13 | Confirm prefix-list or ACL match objects if used | Boundary router | `show ip prefix-list` or `show access-lists` | Match objects correctly identify the target external prefixes |
| 14 | Create a prefix-list for the external prefix that needs controlled metric behavior | Boundary router | `ip prefix-list <PREFIX_LIST> seq 5 permit <PREFIX>/<LENGTH>` | Prefix-list matches the external prefix exactly |
| 15 | Create a route-map sequence for preferred external routes | Boundary router | `route-map <ROUTE_MAP> permit 10` | Route-map sequence is created |
| 16 | Match the target external prefix | Boundary router | `match ip address prefix-list <PREFIX_LIST>` | Route-map selects the intended prefix |
| 17 | Set the preferred EIGRP external seed metric | Boundary router | `set metric <BW> <DELAY> <RELIABILITY> <LOAD> <MTU>` | Redistributed route receives the intended EIGRP metric |
| 18 | Set an origin tag for loop prevention | Boundary router | `set tag <TAG_VALUE>` | Redistributed route carries the intended tag |
| 19 | Add a final permit route-map sequence for other redistributed routes | Boundary router | `route-map <ROUTE_MAP> permit 100` | Other routes are not accidentally blocked by implicit deny |
| 20 | Apply the route-map to OSPF into EIGRP redistribution | Boundary router | `router eigrp <AS_NUMBER>` then `redistribute ospf <OSPF_PROCESS_ID> route-map <ROUTE_MAP>` | OSPF routes enter EIGRP with controlled metric and tag |
| 21 | Apply the route-map to RIP into EIGRP redistribution | Boundary router | `router eigrp <AS_NUMBER>` then `redistribute rip route-map <ROUTE_MAP>` | RIP routes enter EIGRP with controlled metric and tag |
| 22 | Apply the route-map to static into EIGRP redistribution if needed | Boundary router | `router eigrp <AS_NUMBER>` then `redistribute static route-map <ROUTE_MAP>` | Static routes enter EIGRP with controlled metric and tag |
| 23 | Apply the route-map to connected into EIGRP redistribution if needed | Boundary router | `router eigrp <AS_NUMBER>` then `redistribute connected route-map <ROUTE_MAP>` | Connected routes enter EIGRP with controlled metric and tag |
| 24 | Use named-mode redistribution syntax if the router uses named EIGRP | Boundary router | `router eigrp <PROCESS_NAME>` then `address-family ipv4 unicast autonomous-system <AS_NUMBER>` then `topology base` then `redistribute <PROTOCOL> route-map <ROUTE_MAP>` | Redistribution is applied under named-mode topology base |
| 25 | Verify route-map counters after redistribution | Boundary router | `show route-map <ROUTE_MAP>` | Match counters increment for redistributed external routes |
| 26 | Verify external route metric in EIGRP topology | EIGRP routers | `show ip eigrp topology <PREFIX>/<LENGTH>` | External prefix shows expected metric and successor |
| 27 | Verify external route in the routing table | EIGRP routers | `show ip route <PREFIX>` | Route appears as `D EX` if EIGRP external wins the RIB |
| 28 | Verify route tag on the external route | EIGRP routers | `show ip route <PREFIX>` | Route shows expected tag if platform displays route tags |
| 29 | Compare multiple external paths if more than one boundary redistributes the same prefix | EIGRP routers | `show ip eigrp topology <PREFIX>/<LENGTH>` | Preferred path has the intended lowest EIGRP metric |
| 30 | Tune the alternate boundary route-map metric if the wrong external path wins | Alternate boundary router | `set metric <BW> <DELAY> <RELIABILITY> <LOAD> <MTU>` | Alternate external path becomes less or more preferred as intended |
| 31 | Verify the RIB winner after metric tuning | EIGRP routers | `show ip route <PREFIX>` | Installed next hop matches the intended boundary router |
| 32 | Check for external route competition from OSPF | Router seeing both routes | `show ip route <PREFIX>` | If OSPF owns the prefix, AD 110 is beating EIGRP external AD 170 |
| 33 | If needed, lower EIGRP external AD globally for a controlled lab | EIGRP router | `router eigrp <AS_NUMBER>` then `distance eigrp 90 <EXTERNAL_AD>` | External EIGRP AD changes from 170 to the configured value |
| 34 | If needed, change AD for routes learned from a specific neighbor | EIGRP router | `router eigrp <AS_NUMBER>` then `distance <AD> <NEIGHBOR_IP> 0.0.0.0` | Routes from that neighbor use the configured AD |
| 35 | If needed, use prefix-specific AD control with an ACL | EIGRP router | `access-list <ACL> permit <PREFIX> <WILDCARD>` then `router eigrp <AS_NUMBER>` then `distance <AD> <NEIGHBOR_IP> 0.0.0.0 <ACL>` | Only matched routes from the neighbor receive the changed AD |
| 36 | Prefer route tagging and filtering over broad AD changes in production-like designs | Boundary routers | Planning step | Loop prevention remains explicit and easier to audit |
| 37 | Verify no redistribution feedback loop exists | Multiple routers | `traceroute <DESTINATION_IP> numeric` | Trace reaches destination without repeating router sequence |
| 38 | Verify no route churn occurs | Boundary routers | `show logging | include EIGRP|DUAL|OSPF|RIP|ADJCHG` | No repeated install, withdraw, or adjacency instability appears |
| 39 | Verify the external route remains passive in EIGRP | EIGRP routers | `show ip eigrp topology active` | No active routes remain after convergence |
| 40 | Test reachability to the external destination | Test router or host | `ping <EXTERNAL_DESTINATION_IP>` | Ping succeeds |
| 41 | Verify forwarding path to the external destination | Test router or host | `traceroute <EXTERNAL_DESTINATION_IP> numeric` | Path follows the intended boundary router |
| 42 | Verify return routing from the external domain | External-side router | `show ip route <SOURCE_PREFIX>` | Return path exists toward the EIGRP domain |
| 43 | Save the configuration after path selection is stable | Changed routers | `write memory` | Configuration is saved |
# EIGRP_External_Route_Path_Selection_Skeleton
! Baseline verification
show ip interface brief
show ip protocols
show ip eigrp neighbors
show ip route eigrp
show ip route <PREFIX>
show ip eigrp topology <PREFIX>/<LENGTH>
show ip eigrp topology all-links
! Build match object for controlled external route selection
configure terminal
 ip prefix-list <PREFIX_LIST> seq 5 permit <PREFIX>/<LENGTH>
end
! Build route-map to set external EIGRP metric and tag
configure terminal
 route-map <ROUTE_MAP> permit 10
  match ip address prefix-list <PREFIX_LIST>
  set metric <BW> <DELAY> <RELIABILITY> <LOAD> <MTU>
  set tag <TAG_VALUE>
 route-map <ROUTE_MAP> permit 100
end
! Classic mode: redistribute OSPF into EIGRP with route-map
configure terminal
 router eigrp <AS_NUMBER>
  redistribute ospf <OSPF_PROCESS_ID> route-map <ROUTE_MAP>
 end
! Classic mode: redistribute RIP into EIGRP with route-map
configure terminal
 router eigrp <AS_NUMBER>
  redistribute rip route-map <ROUTE_MAP>
 end
! Classic mode: redistribute static into EIGRP with route-map
configure terminal
 router eigrp <AS_NUMBER>
  redistribute static route-map <ROUTE_MAP>
 end
! Named mode: redistribution under topology base
configure terminal
 router eigrp <PROCESS_NAME>
  address-family ipv4 unicast autonomous-system <AS_NUMBER>
   topology base
    redistribute <PROTOCOL> route-map <ROUTE_MAP>
   exit-af-topology
  exit-address-family
 end
! Optional global EIGRP AD adjustment
configure terminal
 router eigrp <AS_NUMBER>
  distance eigrp 90 <EXTERNAL_AD>
 end
! Optional neighbor-specific AD adjustment
configure terminal
 router eigrp <AS_NUMBER>
  distance <AD> <NEIGHBOR_IP> 0.0.0.0
 end
! Optional prefix-specific AD adjustment from a neighbor
configure terminal
 access-list <ACL_NUMBER> permit <PREFIX> <WILDCARD>
 router eigrp <AS_NUMBER>
  distance <AD> <NEIGHBOR_IP> 0.0.0.0 <ACL_NUMBER>
 end
! Verification
show running-config | section router eigrp
show ip prefix-list <PREFIX_LIST>
show route-map <ROUTE_MAP>
show ip protocols
show ip route <PREFIX>
show ip route eigrp
show ip eigrp topology <PREFIX>/<LENGTH>
show ip eigrp topology active
show logging | include EIGRP|DUAL|OSPF|RIP|ADJCHG
ping <EXTERNAL_DESTINATION_IP>
traceroute <EXTERNAL_DESTINATION_IP> numeric
! Save
write memory
# EIGRP_External_Route_Path_Selection_Verification_Commands
| Command | Purpose | Healthy Output |
|---|---|---|
| `show ip interface brief` | Confirms interface state before path-selection work | Routing interfaces are `up/up` |
| `show ip protocols` | Confirms EIGRP, redistribution, metrics, AD, and route-map references | Expected redistribution and distance settings are listed |
| `show ip eigrp neighbors` | Confirms EIGRP adjacency health | Expected neighbors appear with stable uptime and `Q Cnt` of `0` |
| `show running-config | section router eigrp` | Confirms redistribution, distance, and named-mode placement | Shows intended `redistribute`, `route-map`, and `distance` commands |
| `show ip prefix-list <PREFIX_LIST>` | Confirms prefix match object | Target external prefix is matched exactly |
| `show route-map <ROUTE_MAP>` | Confirms metric and tag policy | Route-map has correct match, set metric, set tag, and counters |
| `show ip route eigrp` | Lists installed EIGRP routes | External routes appear as `D EX` |
| `show ip route <PREFIX>` | Confirms exact RIB winner | Route source, AD, metric, tag, next hop, and outgoing interface match intent |
| `show ip eigrp topology <PREFIX>/<LENGTH>` | Confirms EIGRP topology state for the external route | Prefix is passive and has expected successor and metric |
| `show ip eigrp topology all-links` | Shows all EIGRP candidate paths | Multiple external paths can be compared |
| `show ip eigrp topology active` | Checks DUAL stability | No active routes remain after convergence |
| `show logging | include EIGRP|DUAL|OSPF|RIP|ADJCHG` | Checks redistribution churn and adjacency issues | No repeated route churn or neighbor resets |
| `traceroute <EXTERNAL_DESTINATION_IP> numeric` | Confirms forwarding path | Path exits through intended redistribution boundary |
| `ping <EXTERNAL_DESTINATION_IP>` | Confirms data-plane reachability | Ping succeeds |
# EIGRP_External_Route_Path_Selection_Rollback
! Remove route-map-based redistribution from classic EIGRP
configure terminal
 router eigrp <AS_NUMBER>
  no redistribute ospf <OSPF_PROCESS_ID>
  no redistribute rip
  no redistribute static
  no redistribute connected
 end
! Remove route-map-based redistribution from named EIGRP
configure terminal
 router eigrp <PROCESS_NAME>
  address-family ipv4 unicast autonomous-system <AS_NUMBER>
   topology base
    no redistribute <PROTOCOL>
   exit-af-topology
  exit-address-family
 end
! Restore default EIGRP administrative distances if changed globally
configure terminal
 router eigrp <AS_NUMBER>
  no distance eigrp
 end
! Remove neighbor-specific or prefix-specific AD change
configure terminal
 router eigrp <AS_NUMBER>
  no distance <AD> <NEIGHBOR_IP> 0.0.0.0
  no distance <AD> <NEIGHBOR_IP> 0.0.0.0 <ACL_NUMBER>
 end
! Remove route-map if created only for this mechanism
configure terminal
 no route-map <ROUTE_MAP>
end
! Remove prefix-list if created only for this mechanism
configure terminal
 no ip prefix-list <PREFIX_LIST>
end
! Remove ACL if created only for distance control
configure terminal
 no access-list <ACL_NUMBER>
end
! Lab-only refresh if stale routes remain
clear ip route *
clear ip eigrp neighbors
! Verify rollback
show running-config | section router eigrp
show ip prefix-list <PREFIX_LIST>
show route-map <ROUTE_MAP>
show ip protocols
show ip route <PREFIX>
show ip route eigrp
show ip eigrp topology <PREFIX>/<LENGTH>
show ip eigrp topology active
ping <EXTERNAL_DESTINATION_IP>
traceroute <EXTERNAL_DESTINATION_IP> numeric
! Expected result:
! Custom external metric policy is removed.
! Custom AD policy is removed.
! External EIGRP route behavior returns to default redistribution and AD behavior unless another policy remains.
write memory
# EIGRP_External_Route_Path_Selection_Failure_Checks
| Symptom | Likely Cause | Check Command | Fix |
|---|---|---|---|
| External route does not appear as `D EX` | Redistribution missing or route-map blocks it | `show ip protocols` and `show route-map <ROUTE_MAP>` | Add or correct redistribution and route-map permit logic |
| External route appears in topology but not RIB | Another protocol has better AD | `show ip route <PREFIX>` | Compare AD and decide whether EIGRP external should actually win |
| OSPF keeps beating EIGRP external | OSPF AD 110 is lower than EIGRP external AD 170 | `show ip route <PREFIX>` | Use targeted AD control, route filtering, or tag-based loop prevention |
| External route repeatedly appears and disappears | Redistribution feedback loop | `show logging | include EIGRP|OSPF|RIP|DUAL` | Use route tags and deny routes from returning to their source domain |
| Wrong external boundary is preferred | Seed metrics are equal or poorly chosen | `show ip eigrp topology <PREFIX>/<LENGTH>` | Set better EIGRP metrics with route-map `set metric` |
| Route-map has no effect | Route-map is not attached to the `redistribute` command | `show ip protocols` | Attach route-map to the correct redistribution statement |
| Route-map blocks all redistributed routes | Missing final permit sequence | `show route-map <ROUTE_MAP>` | Add `route-map <ROUTE_MAP> permit 100` |
| Target prefix does not match route-map | Prefix-list has wrong prefix or length | `show ip prefix-list <PREFIX_LIST>` and `show ip route <PREFIX>` | Correct prefix-list match |
| Route tag is missing | Route-map does not set tag or platform does not show it in brief output | `show route-map <ROUTE_MAP>` and `show ip route <PREFIX>` | Add `set tag <TAG_VALUE>` and inspect exact route output |
| AD change affects too many routes | Global `distance eigrp` was used too broadly | `show running-config | section router eigrp` | Use neighbor-specific or prefix-specific distance control instead |
| Prefix-specific AD change does not work | ACL does not match the route | `show access-lists <ACL_NUMBER>` and `show ip route <PREFIX>` | Correct ACL wildcard match |
| Named-mode redistribution does not apply | Command placed outside `topology base` | `show running-config | section router eigrp` | Place redistribution under address-family `topology base` |
| Classic redistribution does not apply | Command placed under wrong EIGRP AS | `show ip protocols` | Configure redistribution under the active `router eigrp <AS_NUMBER>` process |
| External route is learned but traffic fails | Return route missing | Remote `show ip route <SOURCE_PREFIX>` | Fix reverse path in the external domain |
| External route is installed but path is suboptimal | Seed metric lost real source-domain cost | `show ip eigrp topology <PREFIX>/<LENGTH>` | Use route-map metric tuning per boundary |
| Traceroute loops | Mutual redistribution without proper filtering | `traceroute <DESTINATION_IP> numeric` | Add route tags, deny return tags, or reduce redistribution points |
| Route remains active | DUAL query/reply issue after external route loss | `show ip eigrp topology active` | Troubleshoot missing replies, unstable neighbors, or query scope |
| Clearing routes disrupts lab | Clear command was used unnecessarily | `show logging` | Use clear commands only in lab or during maintenance |
# Index
# Source_Basis
# EIGRP_External_Route_Path_Selection_Mental_Model
# EIGRP_External_Route_Path_Selection_Configuration_Checklist
# EIGRP_External_Route_Path_Selection_Skeleton
# EIGRP_External_Route_Path_Selection_Verification_Commands
# EIGRP_External_Route_Path_Selection_Rollback
# EIGRP_External_Route_Path_Selection_Failure_Checks