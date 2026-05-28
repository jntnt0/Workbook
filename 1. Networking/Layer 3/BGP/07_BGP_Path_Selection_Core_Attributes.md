

```
# Index
# Source_Basis
# BGP_Path_Selection_Core_Attributes_Mental_Model
# BGP_Path_Selection_Core_Attributes_Configuration_Checklist
# BGP_Path_Selection_Core_Attributes_Weight_Inbound_Route_Map_Skeleton
# BGP_Path_Selection_Core_Attributes_Neighbor_Weight_Skeleton
# BGP_Path_Selection_Core_Attributes_Local_Preference_Skeleton
# BGP_Path_Selection_Core_Attributes_AS_Path_Prepending_Outbound_Skeleton
# BGP_Path_Selection_Core_Attributes_Origin_Modification_Skeleton
# BGP_Path_Selection_Core_Attributes_MED_Skeleton
# BGP_Path_Selection_Core_Attributes_MED_Behavior_Tuning_Skeleton
# BGP_Path_Selection_Core_Attributes_Best_Path_Verification_Skeleton
# BGP_Path_Selection_Core_Attributes_Verification_Commands
# BGP_Path_Selection_Core_Attributes_Rollback
# BGP_Path_Selection_Core_Attributes_Failure_Checks
# Attached_Labs
```

# BGP_Path_Selection_Core_Attributes_Mental_Model
| Concept                 | Operational Meaning                                                                                              |
| ----------------------- | ---------------------------------------------------------------------------------------------------------------- |
| Longest prefix match    | Route lookup chooses the most specific prefix before BGP attribute comparison matters                            |
| BGP Loc-RIB             | BGP keeps available paths and their attributes even when only one becomes best                                   |
| Best path               | The path BGP selects for a prefix before offering it to the routing table                                        |
| RIB install             | Best BGP path must still be accepted by the routing table                                                        |
| Next-hop validity       | BGP path must have a resolvable next hop before it is eligible as best                                           |
| Weight                  | Cisco local-only attribute; highest wins and affects only the local router                                       |
| Local preference        | AS-wide internal preference; highest wins and is propagated to iBGP peers                                        |
| Locally originated      | Locally originated or aggregated routes beat routes received from peers after weight and local preference tie    |
| AIGP                    | Optional metric-like attribute used in controlled domains; lower AIGP is preferred when present                  |
| AS path length          | Shorter AS path is preferred after earlier criteria tie                                                          |
| Origin code             | IGP origin is preferred over EGP, which is preferred over incomplete                                             |
| MED                     | Lower MED is preferred, normally when paths come from the same neighboring AS                                    |
| eBGP over iBGP          | External path is preferred over internal path after earlier criteria tie                                         |
| Closest IGP next hop    | Lower IGP metric to the BGP next hop wins after earlier criteria tie                                             |
| Oldest eBGP path        | Older stable eBGP path can win as a late tie-breaker                                                             |
| Router ID tie-breaker   | Lower advertising router ID wins late in the process                                                             |
| Cluster list length     | Shorter cluster list wins in route-reflector designs                                                             |
| Neighbor IP tie-breaker | Lowest neighbor IP wins as the final tie-breaker                                                                 |
| Route map policy        | Common method used to match prefixes and set BGP attributes                                                      |
| Prefix list             | Common method used to match exact prefixes before setting attributes                                             |
| Inbound policy          | Changes how this router treats routes received from a neighbor                                                   |
| Outbound policy         | Changes what this router sends to a neighbor                                                                     |
| Blunt rule              | Do not touch BGP attributes until neighbor state, next-hop reachability, and route visibility are already proven |


# BGP_Path_Selection_Core_Attributes_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm BGP neighbor state | Router | `show bgp ipv4 unicast summary` | Neighbor shows a number under `State/PfxRcd` |
| 2 | Confirm target prefix exists in BGP table | Router | `show bgp ipv4 unicast <prefix>` | Prefix appears with one or more paths |
| 3 | Confirm path is valid before policy work | Router | `show bgp ipv4 unicast <prefix>` | Candidate path has `*` or shows valid in detail output |
| 4 | Confirm current best path | Router | `show bgp ipv4 unicast <prefix>` | Current best path is marked with `>` |
| 5 | Confirm why best path won | Router | `show bgp ipv4 unicast <prefix> best-path-reason` | Router displays best-path evaluation reason if IOS XE supports it |
| 6 | Confirm next-hop reachability | Router | `show ip route <next-hop-ip>` | BGP next hop is resolvable |
| 7 | Confirm current route install | Router | `show ip route <prefix>` | BGP route is installed or RIB failure reason is investigated |
| 8 | Confirm forwarding entry | Router | `show ip cef <prefix>` | CEF points to expected next hop/interface |
| 9 | Identify desired traffic direction | Router / Notes | `outbound-from-local-AS` or `inbound-to-local-AS` | Attribute choice matches traffic engineering goal |
| 10 | Choose weight only for local-router outbound preference | Router / Notes | `weight` | Only this router should prefer the path |
| 11 | Choose local preference for AS-wide outbound preference | Router / Notes | `local-preference` | iBGP peers in the AS should prefer the path |
| 12 | Choose AS path prepend for influencing external inbound traffic | Router / Notes | `as-path prepend` | Remote AS should see one path as less preferred |
| 13 | Choose origin manipulation only for lab or narrow policy cases | Router / Notes | `set origin` | Origin is changed intentionally, not casually |
| 14 | Choose MED only for neighboring-AS entry-point preference | Router / Notes | `metric / MED` | Lower MED should influence the adjacent AS |
| 15 | Confirm target neighbor or peer group | Router / Notes | `<neighbor-ip>` or `<peer-group-name>` | Policy attachment point is known |
| 16 | Confirm policy direction | Router / Notes | `in` or `out` | Route map direction is chosen deliberately |
| 17 | Review existing policy | Router | `show running-config | include route-map|prefix-list|filter-list|distribute-list` | Existing policy is known before changes |
| 18 | Review existing BGP config | Router | `show running-config | section router bgp` | Neighbor and address-family config are visible |
| 19 | Create prefix list for the target prefix | Router | `ip prefix-list <pl-name> permit <prefix>/<length>` | Prefix list matches only the intended route |
| 20 | Create route map sequence | Router | `route-map <rm-name> permit 10` | Route-map sequence exists |
| 21 | Match the prefix list | Router | `match ip address prefix-list <pl-name>` | Route map matches target prefix |
| 22 | Set weight for local-only preference | Router | `set weight <weight-value>` | Matching route receives local Cisco weight |
| 23 | Set local preference for AS-wide preference | Router | `set local-preference <local-pref-value>` | Matching route receives higher or lower local preference |
| 24 | Set AS path prepend | Router | `set as-path prepend <asn> <asn> <asn>` | Matching route is advertised with longer AS path |
| 25 | Set origin attribute | Router | `set origin igp` or `set origin incomplete` | Matching route receives chosen origin code |
| 26 | Set MED | Router | `set metric <med-value>` | Matching route receives MED value |
| 27 | Add permissive route-map sequence if only selected routes should be modified | Router | `route-map <rm-name> permit 20` | Unmatched routes continue without being denied |
| 28 | Enter BGP process | Router | `router bgp <local-asn>` | Router enters BGP configuration mode |
| 29 | Enter IPv4 unicast address family | Router | `address-family ipv4 unicast` | Router enters IPv4 unicast AFI |
| 30 | Apply inbound route map | Router | `neighbor <neighbor-ip> route-map <rm-name> in` | Routes received from neighbor are modified or filtered by route map |
| 31 | Apply outbound route map | Router | `neighbor <neighbor-ip> route-map <rm-name> out` | Routes advertised to neighbor are modified or filtered by route map |
| 32 | Apply policy to peer group if used | Router | `neighbor <peer-group-name> route-map <rm-name> in` or `out` | Peer group members inherit policy |
| 33 | Configure neighbor-wide weight if all routes from peer should be preferred locally | Router | `neighbor <neighbor-ip> weight <weight-value>` | All routes from that neighbor receive the configured weight |
| 34 | Exit address-family mode | Router | `exit-address-family` | Router returns to BGP config mode |
| 35 | Soft clear inbound after inbound policy change | Router | `clear bgp ipv4 unicast <neighbor-ip> soft in` | Inbound route policy is re-evaluated without hard reset |
| 36 | Soft clear outbound after outbound policy change | Router | `clear bgp ipv4 unicast <neighbor-ip> soft out` | Outbound advertisements are refreshed without hard reset |
| 37 | Verify route-map hit count | Router | `show route-map <rm-name>` | Route-map sequence counters increment |
| 38 | Verify prefix-list hit count | Router | `show ip prefix-list <pl-name>` | Prefix-list match count increments if supported |
| 39 | Verify BGP table attributes | Router | `show bgp ipv4 unicast <prefix>` | Weight, local preference, MED, origin, and AS path reflect policy |
| 40 | Verify best path changed or stayed intentionally | Router | `show bgp ipv4 unicast <prefix> best-path-reason` | Best-path reason matches the intended attribute |
| 41 | Verify local route install | Router | `show ip route <prefix>` | Desired BGP path installs if selected and valid |
| 42 | Verify iBGP propagation for local preference | iBGP Peer | `show bgp ipv4 unicast <prefix>` | iBGP peers see the local preference value |
| 43 | Verify weight stays local | iBGP Peer | `show bgp ipv4 unicast <prefix>` | Other routers do not inherit the local router’s weight |
| 44 | Verify AS path prepend on external peer | External Peer | `show bgp ipv4 unicast <prefix>` | External peer sees longer AS path on prepended advertisement |
| 45 | Verify MED on receiving peer | Neighbor Router | `show bgp ipv4 unicast <prefix>` | Receiving router sees intended MED value |
| 46 | Verify traffic path | Router | `traceroute <destination-ip> source <source-ip>` | Traffic follows the intended selected BGP path |
| 47 | Save configuration | Router | `copy running-config startup-config` | Policy configuration is saved |
```
# BGP_Path_Selection_Core_Attributes_Weight_Inbound_Route_Map_Skeleton
configure terminal

ip prefix-list <pl-name> permit <prefix>/<length>

route-map <rm-name> permit 10
 match ip address prefix-list <pl-name>
 set weight <weight-value>

route-map <rm-name> permit 20

router bgp <local-asn>
 address-family ipv4 unicast
  neighbor <neighbor-ip> route-map <rm-name> in
 exit-address-family

end
clear bgp ipv4 unicast <neighbor-ip> soft in
copy running-config startup-config
```


```
# BGP_Path_Selection_Core_Attributes_Neighbor_Weight_Skeleton
configure terminal

router bgp <local-asn>
 address-family ipv4 unicast
  neighbor <neighbor-ip> weight <weight-value>
 exit-address-family

end
clear bgp ipv4 unicast <neighbor-ip> soft in
copy running-config startup-config
```


```
# BGP_Path_Selection_Core_Attributes_Local_Preference_Skeleton
configure terminal

ip prefix-list <pl-name> permit <prefix>/<length>

route-map <rm-name> permit 10
 match ip address prefix-list <pl-name>
 set local-preference <local-pref-value>

route-map <rm-name> permit 20

router bgp <local-asn>
 address-family ipv4 unicast
  neighbor <ebgp-neighbor-ip> route-map <rm-name> in
 exit-address-family

end
clear bgp ipv4 unicast <ebgp-neighbor-ip> soft in
copy running-config startup-config
```


```
# BGP_Path_Selection_Core_Attributes_AS_Path_Prepending_Outbound_Skeleton
configure terminal

ip prefix-list <pl-name> permit <prefix>/<length>

route-map <rm-name> permit 10
 match ip address prefix-list <pl-name>
 set as-path prepend <local-asn> <local-asn> <local-asn>

route-map <rm-name> permit 20

router bgp <local-asn>
 address-family ipv4 unicast
  neighbor <external-neighbor-ip> route-map <rm-name> out
 exit-address-family

end
clear bgp ipv4 unicast <external-neighbor-ip> soft out
copy running-config startup-config
```


```
# BGP_Path_Selection_Core_Attributes_Origin_Modification_Skeleton
configure terminal

ip prefix-list <pl-name> permit <prefix>/<length>

route-map <rm-name> permit 10
 match ip address prefix-list <pl-name>
 set origin incomplete

route-map <rm-name> permit 20

router bgp <local-asn>
 address-family ipv4 unicast
  neighbor <neighbor-ip> route-map <rm-name> in
 exit-address-family

end
clear bgp ipv4 unicast <neighbor-ip> soft in
copy running-config startup-config
```


```
# BGP_Path_Selection_Core_Attributes_MED_Skeleton
configure terminal

ip prefix-list <pl-name> permit <prefix>/<length>

route-map <rm-name> permit 10
 match ip address prefix-list <pl-name>
 set metric <med-value>

route-map <rm-name> permit 20

router bgp <local-asn>
 address-family ipv4 unicast
  neighbor <neighbor-ip> route-map <rm-name> out
 exit-address-family

end
clear bgp ipv4 unicast <neighbor-ip> soft out
copy running-config startup-config
```


```
# BGP_Path_Selection_Core_Attributes_MED_Behavior_Tuning_Skeleton
configure terminal

router bgp <local-asn>
 bgp bestpath med missing-as-worst
 bgp deterministic-med
 bgp always-compare-med

end
copy running-config startup-config
```


```
# BGP_Path_Selection_Core_Attributes_Best_Path_Verification_Skeleton
show bgp ipv4 unicast summary
show bgp ipv4 unicast <prefix>
show bgp ipv4 unicast <prefix> bestpath
show bgp ipv4 unicast <prefix> best-path-reason
show ip route <prefix>
show ip cef <prefix>
show route-map <rm-name>
show ip prefix-list <pl-name>
show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes
show running-config | section router bgp
show running-config | section route-map
```



# BGP_Path_Selection_Core_Attributes_Verification_Commands
| Task                           | Command                                                           | Expected Result                                                          |                                                                      |                                                      |
| ------------------------------ | ----------------------------------------------------------------- | ------------------------------------------------------------------------ | -------------------------------------------------------------------- | ---------------------------------------------------- |
| Verify BGP session state       | `show bgp ipv4 unicast summary`                                   | Neighbor is established                                                  |                                                                      |                                                      |
| Verify target prefix           | `show bgp ipv4 unicast <prefix>`                                  | Prefix has multiple candidate paths or expected path                     |                                                                      |                                                      |
| Verify next-hop validity       | `show ip route <next-hop-ip>`                                     | BGP next hop is reachable                                                |                                                                      |                                                      |
| Verify current best path       | `show bgp ipv4 unicast <prefix>`                                  | Best path is marked with `>`                                             |                                                                      |                                                      |
| Verify path attributes         | `show bgp ipv4 unicast <prefix>`                                  | Weight, local preference, MED, origin, AS path, and next hop are visible |                                                                      |                                                      |
| Verify best path only          | `show bgp ipv4 unicast <prefix> bestpath`                         | Only selected best path is displayed if supported                        |                                                                      |                                                      |
| Verify best path reason        | `show bgp ipv4 unicast <prefix> best-path-reason`                 | Output explains why one path won or lost if supported                    |                                                                      |                                                      |
| Verify route-map config        | `show running-config                                              | section route-map`                                                       | Match and set clauses are correct                                    |                                                      |
| Verify route-map hits          | `show route-map <rm-name>`                                        | Route-map sequence hit counters increment                                |                                                                      |                                                      |
| Verify prefix-list config      | `show ip prefix-list <pl-name>`                                   | Prefix list matches intended prefix only                                 |                                                                      |                                                      |
| Verify BGP neighbor policy     | `show running-config                                              | section router bgp`                                                      | Neighbor route-map or weight is applied in correct AFI and direction |                                                      |
| Verify local weight            | `show bgp ipv4 unicast <prefix>`                                  | Weight value appears only on local router                                |                                                                      |                                                      |
| Verify local preference        | `show bgp ipv4 unicast <prefix>`                                  | Local preference appears and is propagated inside the AS                 |                                                                      |                                                      |
| Verify AS path prepend         | `show bgp ipv4 unicast <prefix>` on external peer                 | Prepended AS path appears on received route                              |                                                                      |                                                      |
| Verify origin code             | `show bgp ipv4 unicast <prefix>`                                  | Origin code shows `i`, `e`, or `?` as intended                           |                                                                      |                                                      |
| Verify MED                     | `show bgp ipv4 unicast <prefix>`                                  | Metric/MED column shows expected value                                   |                                                                      |                                                      |
| Verify RIB install             | `show ip route <prefix>`                                          | Desired BGP path installs in routing table                               |                                                                      |                                                      |
| Verify forwarding path         | `show ip cef <prefix>`                                            | CEF points toward desired next hop/interface                             |                                                                      |                                                      |
| Verify advertised routes       | `show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes` | Outbound policy changes are visible to neighbor                          |                                                                      |                                                      |
| Verify received route behavior | `show bgp ipv4 unicast neighbors <neighbor-ip> routes`            | Inbound paths reflect expected attributes if command is supported        |                                                                      |                                                      |
| Verify traffic path            | `traceroute <destination-ip> source <source-ip>`                  | Traffic follows the selected path                                        |                                                                      |                                                      |
| Verify logs                    | `show logging                                                     | include BGP                                                              | ADJCHANGE`                                                           | No unexpected session resets or policy-related churn |


# BGP_Path_Selection_Core_Attributes_Rollback
| Step | Task                                         | Device | Command                                             | Expected Result                                                             |                                                             |
| ---: | -------------------------------------------- | ------ | --------------------------------------------------- | --------------------------------------------------------------------------- | ----------------------------------------------------------- |
|    1 | Identify current BGP policy                  | Router | `show running-config                                | section router bgp`                                                         | Neighbor route maps, weight, and AFI policy are visible     |
|    2 | Identify route maps                          | Router | `show running-config                                | section route-map`                                                          | Route-map names, match clauses, and set clauses are visible |
|    3 | Identify prefix lists                        | Router | `show ip prefix-list`                               | Prefix lists used by policy are visible                                     |                                                             |
|    4 | Enter configuration mode                     | Router | `configure terminal`                                | Router enters global configuration mode                                     |                                                             |
|    5 | Enter BGP process                            | Router | `router bgp <local-asn>`                            | Router enters BGP config mode                                               |                                                             |
|    6 | Enter IPv4 unicast AFI                       | Router | `address-family ipv4 unicast`                       | Router enters IPv4 unicast address-family                                   |                                                             |
|    7 | Remove inbound route map                     | Router | `no neighbor <neighbor-ip> route-map <rm-name> in`  | Inbound policy is detached                                                  |                                                             |
|    8 | Remove outbound route map                    | Router | `no neighbor <neighbor-ip> route-map <rm-name> out` | Outbound policy is detached                                                 |                                                             |
|    9 | Remove neighbor-wide weight                  | Router | `no neighbor <neighbor-ip> weight <weight-value>`   | Neighbor-wide local weight is removed                                       |                                                             |
|   10 | Exit AFI mode                                | Router | `exit-address-family`                               | Router returns to BGP config mode                                           |                                                             |
|   11 | Remove MED behavior tuning if lab-only       | Router | `no bgp bestpath med missing-as-worst`              | Missing MED behavior returns to default                                     |                                                             |
|   12 | Remove deterministic MED if lab-only         | Router | `no bgp deterministic-med`                          | Deterministic MED behavior is removed                                       |                                                             |
|   13 | Remove always-compare MED if lab-only        | Router | `no bgp always-compare-med`                         | MED comparison returns to default behavior                                  |                                                             |
|   14 | Remove route-map sequence                    | Router | `no route-map <rm-name> permit <seq>`               | Route-map sequence is removed                                               |                                                             |
|   15 | Remove route-map permit fallback if lab-only | Router | `no route-map <rm-name> permit 20`                  | Fallback sequence is removed                                                |                                                             |
|   16 | Remove prefix list                           | Router | `no ip prefix-list <pl-name>`                       | Prefix list is removed                                                      |                                                             |
|   17 | Soft clear inbound after inbound rollback    | Router | `clear bgp ipv4 unicast <neighbor-ip> soft in`      | Inbound routes are reprocessed                                              |                                                             |
|   18 | Soft clear outbound after outbound rollback  | Router | `clear bgp ipv4 unicast <neighbor-ip> soft out`     | Outbound advertisements are refreshed                                       |                                                             |
|   19 | Verify attribute rollback                    | Router | `show bgp ipv4 unicast <prefix>`                    | Weight, local preference, AS path, origin, or MED returns to expected value |                                                             |
|   20 | Verify selected path after rollback          | Router | `show bgp ipv4 unicast <prefix> best-path-reason`   | Best path reason reflects rollback                                          |                                                             |
|   21 | Save rollback                                | Router | `copy running-config startup-config`                | Rollback is saved                                                           |                                                             |


# BGP_Path_Selection_Core_Attributes_Failure_Checks
| Symptom                                         | Command                                                            | What Usually Broke                                                                                                        |                                                                               |                                                                              |
| ----------------------------------------------- | ------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------- | ---------------------------------------------------------------------------- |
| Attribute policy has no effect                  | `show running-config                                               | section router bgp`                                                                                                       | Route map applied to wrong neighbor, wrong address family, or wrong direction |                                                                              |
| Route-map hits stay zero                        | `show route-map <rm-name>`                                         | Prefix list does not match, traffic route is absent, or policy is attached wrong                                          |                                                                               |                                                                              |
| Prefix list hits stay zero                      | `show ip prefix-list <pl-name>`                                    | Prefix or prefix length is wrong                                                                                          |                                                                               |                                                                              |
| Route disappears after route-map applied        | `show route-map <rm-name>`                                         | Route map has no permissive fallback sequence, causing unmatched routes to be denied                                      |                                                                               |                                                                              |
| Weight works only on one router                 | `show bgp ipv4 unicast <prefix>`                                   | Expected behavior; weight is local-only                                                                                   |                                                                               |                                                                              |
| Local preference does not affect other routers  | `show bgp ipv4 unicast <prefix>` on iBGP peers                     | Local preference was not set inbound at the edge, iBGP did not propagate route, or route reflector/full mesh issue exists |                                                                               |                                                                              |
| Local preference is missing on eBGP peer        | External peer `show bgp ipv4 unicast <prefix>`                     | Expected behavior; local preference is not advertised outside the AS                                                      |                                                                               |                                                                              |
| AS path prepend does not change local best path | `show bgp ipv4 unicast <prefix>`                                   | Prepending was applied outbound to influence remote inbound traffic, not local selection                                  |                                                                               |                                                                              |
| AS path prepend not visible on peer             | Peer `show bgp ipv4 unicast <prefix>`                              | Route map applied inbound instead of outbound, wrong prefix matched, or peer received another route                       |                                                                               |                                                                              |
| Origin manipulation does not win                | `show bgp ipv4 unicast <prefix> best-path-reason`                  | Higher-priority attributes like weight, local preference, or AS path already decided the path                             |                                                                               |                                                                              |
| MED does not influence path                     | `show bgp ipv4 unicast <prefix>`                                   | Paths are from different neighboring ASNs or earlier attributes already decided                                           |                                                                               |                                                                              |
| Missing MED path wins unexpectedly              | `show bgp ipv4 unicast <prefix>`                                   | Missing MED is treated like 0 by default                                                                                  |                                                                               |                                                                              |
| MED comparison seems inconsistent               | `show running-config                                               | include bgp deterministic-med                                                                                             | always-compare-med`                                                           | Deterministic MED or always-compare MED behavior is not enabled consistently |
| Best path does not change after policy edit     | `clear bgp ipv4 unicast <neighbor-ip> soft in` or `soft out`       | Routes were not refreshed after policy change                                                                             |                                                                               |                                                                              |
| Prefix appears but is not valid                 | `show bgp ipv4 unicast <prefix>` and `show ip route <next-hop-ip>` | Next hop is unreachable                                                                                                   |                                                                               |                                                                              |
| Prefix is best in BGP but not in routing table  | `show bgp ipv4 unicast <prefix>` and `show ip route <prefix>`      | RIB failure, better administrative distance route, or install constraint                                                  |                                                                               |                                                                              |
| Traffic still follows old path                  | `show ip cef <prefix>`                                             | CEF still points to old path, route did not install, or recursive next-hop path differs                                   |                                                                               |                                                                              |
| iBGP peers disagree on best path                | `show bgp ipv4 unicast <prefix>` across routers                    | Attribute policy inconsistent, IGP metric to next hop differs, route reflection behavior, or local weight differs         |                                                                               |                                                                              |
| Policy change caused session reset              | `show logging                                                      | include BGP                                                                                                               | ADJCHANGE`                                                                    | Hard clear was used or neighbor-affecting config caused reset                |
| Troubleshooting starts at wrong place           | `show bgp ipv4 unicast summary` and `show ip route <next-hop-ip>`  | Neighbor or next-hop issue exists before attribute policy matters                                                         |                                                                               |                                                                              |

# Attached_Labs
| Lab Name                                | Attach Under This Note                  | Why                                                                                                      |
| --------------------------------------- | --------------------------------------- | -------------------------------------------------------------------------------------------------------- |
| `bgp-attribute-weight-final`            | `BGP_Path_Selection_Core_Attributes.md` | Primary lab for Cisco local-only weight behavior and first-step best-path influence                      |
| `bgp-local-preference-final`            | `BGP_Path_Selection_Core_Attributes.md` | Primary lab for AS-wide internal path preference through local preference                                |
| `bgp-attribute-locally-originate-final` | `BGP_Path_Selection_Core_Attributes.md` | Secondary cross-reference only; primary attachment remains under `BGP_Origination_And_Default_Routes.md` |