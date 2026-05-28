

```
# Index
# Source_Basis
# ASA_Multicast_Mental_Model
# ASA_Multicast_Configuration_Checklist
# ASA_Multicast_PIM_SM_Basic_Skeleton
# ASA_Multicast_IGMP_Static_Group_Skeleton
# ASA_Multicast_IGMP_Protection_Skeleton
# ASA_Multicast_PIM_RP_Group_ACL_Skeleton
# ASA_Multicast_PIM_Bidir_RP_Skeleton
# ASA_Multicast_PIM_Neighbor_Filter_Skeleton
# ASA_Multicast_Static_Mroute_Skeleton
# ASA_Multicast_Old_IOS_Register_Compatibility_Skeleton
# ASA_Multicast_Verification_Commands
# ASA_Multicast_Rollback
# ASA_Multicast_Failure_Checks
```


# ASA_Multicast_Mental_Model
| Concept                  | Operational Meaning                                                                                                                                                                      |
| ------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ASA multicast routing    | Lets ASA participate in multicast forwarding instead of simply blocking or routing around multicast traffic                                                                              |
| Routed mode only         | ASA multicast routing is supported in routed mode, not transparent mode                                                                                                                  |
| Single context only      | ASA multicast routing is not supported in multiple-context mode                                                                                                                          |
| IGMP                     | Host-to-router membership protocol used by receivers to join multicast groups                                                                                                            |
| IGMP proxy/stub behavior | ASA can forward IGMP membership information from downstream hosts toward upstream multicast routers                                                                                      |
| Static IGMP group        | ASA can force an interface to join a multicast group even without dynamic receiver reports                                                                                               |
| PIM-SM                   | ASA-supported multicast routing protocol for sparse-mode multicast domains                                                                                                               |
| RP                       | Rendezvous Point used by PIM-SM for shared-tree multicast group discovery                                                                                                                |
| Shared tree              | Initial multicast forwarding tree rooted at the RP                                                                                                                                       |
| Shortest-path tree       | Source-specific forwarding tree used after multicast traffic is known                                                                                                                    |
| DR priority              | Determines which PIM router becomes Designated Router on a shared segment                                                                                                                |
| PIM neighbor filter      | Restricts which multicast routers can form PIM neighbor relationships or register sources                                                                                                |
| Static mroute            | Manually defines multicast RPF/forwarding behavior when normal unicast routing is not enough                                                                                             |
| RPF logic                | Multicast forwarding depends on the expected reverse path toward the source                                                                                                              |
| Multicast ACL filtering  | Standard ACLs can restrict RP mapping, PIM neighbor registration, or source behavior depending on feature                                                                                |
| Blunt rule               | ASA multicast is not “turn on one command and walk away.” You need routed mode, multicast routing, IGMP/PIM interface behavior, RP knowledge, RPF correctness, and explicit verification |
# ASA_Multicast_Configuration_Checklist
| Step | Task                                                                   | Device            | Command                                                                                                | Expected Result                                                                               |
| ---: | ---------------------------------------------------------------------- | ----------------- | ------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------- |
|    1 | Confirm ASA interface status                                           | ASA               | `show interface ip brief`                                                                              | Multicast ingress and egress interfaces are up/up                                             |
|    2 | Confirm logical interface names                                        | ASA               | `show nameif`                                                                                          | Interfaces are named correctly, such as `inside`, `outside`, `dmz`, or `multicast`            |
|    3 | Confirm ASA firewall mode                                              | ASA               | `show firewall`                                                                                        | ASA is in routed mode                                                                         |
|    4 | Confirm ASA is not using multiple context mode for this feature        | ASA               | `show mode`                                                                                            | ASA is not in multiple-context mode for multicast routing use                                 |
|    5 | Identify multicast source network                                      | ASA / Notes       | `<source-network> <source-mask>`                                                                       | Multicast source location is known                                                            |
|    6 | Identify receiver network                                              | ASA / Notes       | `<receiver-network> <receiver-mask>`                                                                   | Receiver location is known                                                                    |
|    7 | Identify multicast group                                               | ASA / Notes       | `<multicast-group>`                                                                                    | Group address is known, usually in administratively scoped range such as 239.0.0.0/8 for labs |
|    8 | Identify upstream interface toward source or RP                        | ASA / Notes       | `<upstream-interface>`                                                                                 | Interface toward source/RP is known                                                           |
|    9 | Identify downstream receiver interface                                 | ASA / Notes       | `<downstream-interface>`                                                                               | Interface toward receivers is known                                                           |
|   10 | Identify RP address for PIM-SM                                         | ASA / Notes       | `<rp-ip>`                                                                                              | RP address is known and reachable                                                             |
|   11 | Confirm unicast route to source                                        | ASA               | `show route <source-ip>`                                                                               | ASA has expected route toward multicast source                                                |
|   12 | Confirm unicast route to RP                                            | ASA               | `show route <rp-ip>`                                                                                   | ASA has expected route toward RP                                                              |
|   13 | Confirm unicast route to receivers if needed                           | ASA               | `show route <receiver-ip>`                                                                             | ASA knows receiver-side network path                                                          |
|   14 | Confirm upstream multicast router readiness                            | Upstream Router   | `show ip pim neighbor` or equivalent                                                                   | Upstream router is multicast-capable                                                          |
|   15 | Confirm downstream receiver behavior                                   | Receiver / Switch | IGMP join test or application start                                                                    | Receiver will send IGMP join/report for the group                                             |
|   16 | Review existing multicast configuration                                | ASA               | `show running-config multicast-routing`                                                                | Existing global multicast routing setting is visible                                          |
|   17 | Review existing IGMP configuration                                     | ASA               | `show running-config all igmp`                                                                         | Existing IGMP interface behavior is visible                                                   |
|   18 | Review existing PIM configuration                                      | ASA               | `show running-config all pim`                                                                          | Existing PIM interface and RP behavior is visible                                             |
|   19 | Review current multicast routes                                        | ASA               | `show mroute`                                                                                          | Current multicast routing table is visible                                                    |
|   20 | Review current PIM neighbors                                           | ASA               | `show pim neighbor`                                                                                    | Existing PIM neighbors are visible                                                            |
|   21 | Review current IGMP groups                                             | ASA               | `show igmp groups`                                                                                     | Existing group memberships are visible                                                        |
|   22 | Enter configuration mode                                               | ASA               | `configure terminal`                                                                                   | ASA enters global configuration mode                                                          |
|   23 | Enable multicast routing globally                                      | ASA               | `multicast-routing`                                                                                    | ASA enables multicast routing, IGMP, and PIM behavior                                         |
|   24 | Disable IGMP on interfaces that should not process receiver membership | ASA               | `interface <interface-name>` then `no igmp` or platform-supported `no igmp <option>`                   | IGMP is disabled only where multicast receiver membership is not required                     |
|   25 | Disable PIM on interfaces that should not form PIM adjacency           | ASA               | `interface <interface-name>` then `no pim interface`                                                   | PIM is disabled on non-participating interfaces                                               |
|   26 | Configure IGMP version if default IGMPv2 is not acceptable             | ASA               | `interface <downstream-interface>` then `igmp version <1-or-2>`                                        | Interface uses selected IGMP version                                                          |
|   27 | Configure IGMP state limit for protection                              | ASA               | `interface <downstream-interface>` then `igmp limit <limit>`                                           | Interface limits number of IGMP states                                                        |
|   28 | Configure IGMP query timeout if needed                                 | ASA               | `interface <downstream-interface>` then `igmp query-timeout <seconds>`                                 | ASA query-router takeover behavior is tuned                                                   |
|   29 | Configure static IGMP group if receiver join must be forced            | ASA               | `interface <downstream-interface>` then `igmp static-group <multicast-group>`                          | ASA statically joins the multicast group on that interface                                    |
|   30 | Mark IGMP forward interface if using IGMP forwarding/proxy behavior    | ASA               | `interface <downstream-interface>` then `igmp forward-interface`                                       | Interface is used for IGMP forwarding behavior                                                |
|   31 | Configure PIM DR priority if ASA should or should not win DR election  | ASA               | `interface <pim-interface>` then `pim dr-priority <value>`                                             | DR election priority is set                                                                   |
|   32 | Configure PIM hello interval if needed                                 | ASA               | `interface <pim-interface>` then `pim hello-interval <seconds>`                                        | PIM hello timer is set                                                                        |
|   33 | Configure PIM join/prune interval if needed                            | ASA               | `interface <pim-interface>` then `pim join-prune-interval <seconds>`                                   | PIM join/prune timer is set                                                                   |
|   34 | Configure RP for PIM-SM                                                | ASA               | `pim rp-address <rp-ip>`                                                                               | ASA knows the RP for sparse-mode multicast                                                    |
|   35 | Configure bidirectional RP only if design requires it                  | ASA               | `pim rp-address <rp-ip> bidir`                                                                         | RP is configured for bidirectional multicast behavior                                         |
|   36 | Create RP group ACL if RP should serve only selected groups            | ASA               | `access-list <rp-group-acl> standard permit <multicast-group> <wildcard-mask>`                         | ACL defines which multicast groups map to the RP                                              |
|   37 | Apply RP group ACL if required                                         | ASA               | `pim rp-address <rp-ip> <rp-group-acl>`                                                                | RP is limited to selected multicast groups                                                    |
|   38 | Create PIM neighbor/source filter ACL if needed                        | ASA               | `access-list <pim-filter-acl> standard permit host <trusted-pim-neighbor-ip>`                          | Only trusted PIM neighbor/source is permitted                                                 |
|   39 | Apply PIM neighbor filter                                              | ASA               | `interface <pim-interface>` then `pim neighbor-filter <pim-filter-acl>`                                | PIM neighbor/source registration is restricted                                                |
|   40 | Configure old register checksum only for older IOS interoperability    | ASA               | `pim old-register-checksum`                                                                            | ASA generates older-compatible PIM register checksum behavior                                 |
|   41 | Configure static multicast route only if RPF needs manual correction   | ASA               | `mroute <source-ip-or-network> <mask> <in-interface-name> <out-interface-name-or-next-hop> <distance>` | ASA has a static multicast route/RPF override                                                 |
|   42 | Confirm interface ACLs do not block required multicast control traffic | ASA               | `show access-list <interface-acl>`                                                                     | ACL permits required multicast/data/control path if ACLs are applied                          |
|   43 | Confirm NAT is not incorrectly applied to multicast traffic            | ASA               | `show nat detail`                                                                                      | NAT rules do not break multicast forwarding path                                              |
|   44 | Save configuration                                                     | ASA               | `write memory`                                                                                         | Multicast configuration is saved                                                              |
|   45 | Start receiver application or generate IGMP join                       | Receiver          | `<join multicast group>`                                                                               | Receiver sends IGMP membership report                                                         |
|   46 | Generate multicast source traffic                                      | Source            | `<send traffic to <multicast-group>>`                                                                  | Source sends multicast stream                                                                 |
|   47 | Verify IGMP group membership                                           | ASA               | `show igmp groups`                                                                                     | Receiver interface shows expected group membership                                            |
|   48 | Verify IGMP interface behavior                                         | ASA               | `show igmp interface`                                                                                  | IGMP version, query state, and interface behavior are visible                                 |
|   49 | Verify PIM interfaces                                                  | ASA               | `show pim interface`                                                                                   | Participating interfaces show PIM enabled and expected timers                                 |
|   50 | Verify PIM neighbors                                                   | ASA               | `show pim neighbor`                                                                                    | Expected PIM neighbors are present                                                            |
|   51 | Verify RP mapping                                                      | ASA               | `show pim group-map`                                                                                   | Multicast group maps to expected RP                                                           |
|   52 | Verify multicast route entry                                           | ASA               | `show mroute <multicast-group>`                                                                        | ASA shows expected `(*,G)` and/or `(S,G)` entries                                             |
|   53 | Verify multicast forwarding topology                                   | ASA               | `show pim topology`                                                                                    | Multicast tree information is visible                                                         |
|   54 | Verify multicast traffic counters                                      | ASA               | `show igmp traffic` and `show pim traffic`                                                             | IGMP/PIM counters increment                                                                   |
|   55 | Verify mroute summary                                                  | ASA               | `show mroute summary`                                                                                  | Multicast route summary shows expected state                                                  |
|   56 | Check drops if multicast fails                                         | ASA               | `show asp drop`                                                                                        | Drop reason points to ACL, RPF, PIM, IGMP, NAT, route, mode, or platform issue                |
|   57 | Debug IGMP only if needed                                              | ASA               | `debug igmp`                                                                                           | IGMP joins/leaves/queries are visible                                                         |
|   58 | Debug PIM only if needed                                               | ASA               | `debug pim`                                                                                            | PIM hello, join/prune, RP, and neighbor behavior is visible                                   |
|   59 | Stop debugs                                                            | ASA               | `undebug all`                                                                                          | Debug output stops                                                                            |

```
# ASA_Multicast_PIM_SM_Basic_Skeleton
configure terminal

multicast-routing

interface <upstream-interface>
 pim dr-priority <upstream-dr-priority>
 pim hello-interval <seconds>
 pim join-prune-interval <seconds>

interface <downstream-interface>
 igmp version 2
 igmp limit <limit>
 igmp query-timeout <seconds>
 pim dr-priority <downstream-dr-priority>

pim rp-address <rp-ip>

write memory
```

```
# ASA_Multicast_IGMP_Static_Group_Skeleton
configure terminal

multicast-routing

interface <receiver-interface>
 igmp static-group <multicast-group>
 igmp forward-interface

write memory
```

```
# ASA_Multicast_IGMP_Protection_Skeleton
configure terminal

interface <receiver-interface>
 igmp version 2
 igmp limit <limit>
 igmp query-timeout <seconds>

write memory
```

```
# ASA_Multicast_PIM_RP_Group_ACL_Skeleton
configure terminal

access-list <rp-group-acl> standard permit <multicast-group> <wildcard-mask>

pim rp-address <rp-ip> <rp-group-acl>

write memory
```

```
# ASA_Multicast_PIM_Bidir_RP_Skeleton
configure terminal

multicast-routing

pim rp-address <rp-ip> bidir

write memory
```

```
# ASA_Multicast_PIM_Neighbor_Filter_Skeleton
configure terminal

access-list <pim-filter-acl> standard permit host <trusted-pim-neighbor-ip>

interface <pim-interface>
 pim neighbor-filter <pim-filter-acl>

write memory
```

```
# ASA_Multicast_Static_Mroute_Skeleton
configure terminal

mroute <source-ip-or-network> <mask> <in-interface-name> <out-interface-name-or-next-hop> <distance>

write memory
```

```
# ASA_Multicast_Old_IOS_Register_Compatibility_Skeleton
configure terminal

pim old-register-checksum

write memory
```


# ASA_Multicast_Verification_Commands
| Task                              | Command                                 | Expected Result                                                       |                                             |
| --------------------------------- | --------------------------------------- | --------------------------------------------------------------------- | ------------------------------------------- |
| Verify routed mode                | `show firewall`                         | ASA is in routed mode                                                 |                                             |
| Verify single-context suitability | `show mode`                             | ASA is not using multiple-context mode for multicast routing          |                                             |
| Verify interface state            | `show interface ip brief`               | Multicast ingress and egress interfaces are up/up                     |                                             |
| Verify logical names              | `show nameif`                           | Interface names match multicast configuration                         |                                             |
| Verify unicast route to source    | `show route <source-ip>`                | ASA has expected RPF path toward source                               |                                             |
| Verify unicast route to RP        | `show route <rp-ip>`                    | ASA has expected route toward RP                                      |                                             |
| Verify multicast routing enabled  | `show running-config multicast-routing` | Output includes `multicast-routing`                                   |                                             |
| Verify IGMP interface state       | `show igmp interface`                   | IGMP is active on receiver-facing interface                           |                                             |
| Verify IGMP group membership      | `show igmp groups`                      | Expected multicast group appears on receiver-facing interface         |                                             |
| Verify IGMP traffic               | `show igmp traffic`                     | IGMP reports, queries, and leaves are visible through counters        |                                             |
| Verify PIM interface state        | `show pim interface`                    | PIM is enabled on expected interfaces                                 |                                             |
| Verify PIM neighbors              | `show pim neighbor`                     | Expected multicast router neighbors appear                            |                                             |
| Verify RP mapping                 | `show pim group-map`                    | Multicast group maps to expected RP                                   |                                             |
| Verify PIM range list             | `show pim range-list`                   | RP/group ranges appear if configured                                  |                                             |
| Verify PIM topology               | `show pim topology`                     | Shared tree and/or source tree information is visible                 |                                             |
| Verify join/prune behavior        | `show pim join-prune statistic`         | Join/prune counters increment when receivers join/leave               |                                             |
| Verify PIM traffic                | `show pim traffic`                      | PIM control-plane counters increment                                  |                                             |
| Verify PIM tunnel if used         | `show pim tunnel`                       | PIM tunnel state appears if relevant                                  |                                             |
| Verify multicast route            | `show mroute <multicast-group>`         | `(*,G)` and/or `(S,G)` entries appear                                 |                                             |
| Verify multicast route summary    | `show mroute summary`                   | Expected multicast state count is visible                             |                                             |
| Verify static mroute              | `show running-config                    | include ^mroute`                                                      | Static multicast route exists if configured |
| Verify PIM neighbor filter ACL    | `show access-list <pim-filter-acl>`     | PIM filter ACL exists and hit count behavior is visible               |                                             |
| Verify interface ACLs             | `show access-list <interface-acl>`      | ACLs do not block required multicast/control traffic                  |                                             |
| Verify ASP drops                  | `show asp drop`                         | No relevant multicast, ACL, RPF, route, or inspection drops increment |                                             |
| Debug IGMP cautiously             | `debug igmp`                            | IGMP joins, leaves, and queries are visible                           |                                             |
| Debug PIM cautiously              | `debug pim`                             | PIM neighbor, join/prune, register, and RP behavior is visible        |                                             |
| Debug PIM for one group           | `debug pim group <multicast-group>`     | Debug is limited to selected multicast group                          |                                             |
| Debug PIM on one interface        | `debug pim interface <interface-name>`  | Debug is limited to selected interface                                |                                             |
| Debug PIM neighbors               | `debug pim neighbor`                    | Neighbor adjacency behavior is visible                                |                                             |
| Stop debugs                       | `undebug all`                           | Debug output stops                                                    |                                             |

# ASA_Multicast_Rollback
| Step | Task                                                                  | Device | Command                                                                                                   | Expected Result                                                                 |                                        |
| ---: | --------------------------------------------------------------------- | ------ | --------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------- | -------------------------------------- |
|    1 | Identify multicast routing state                                      | ASA    | `show running-config multicast-routing`                                                                   | Global multicast setting is identified                                          |                                        |
|    2 | Identify IGMP interface configuration                                 | ASA    | `show running-config all igmp`                                                                            | IGMP static groups, limits, query timers, and versions are identified           |                                        |
|    3 | Identify PIM interface configuration                                  | ASA    | `show running-config all pim`                                                                             | PIM timers, DR priority, RP, filters, and compatibility commands are identified |                                        |
|    4 | Identify static mroutes                                               | ASA    | `show running-config                                                                                      | include ^mroute`                                                                | Static multicast routes are identified |
|    5 | Enter configuration mode                                              | ASA    | `configure terminal`                                                                                      | ASA enters global configuration mode                                            |                                        |
|    6 | Remove static IGMP group                                              | ASA    | `interface <receiver-interface>` then `no igmp static-group <multicast-group>`                            | Static group join is removed                                                    |                                        |
|    7 | Remove IGMP forward-interface behavior                                | ASA    | `interface <receiver-interface>` then `no igmp forward-interface`                                         | IGMP forward behavior is removed                                                |                                        |
|    8 | Restore IGMP version to default if changed                            | ASA    | `interface <receiver-interface>` then `no igmp version`                                                   | IGMP version returns to default behavior                                        |                                        |
|    9 | Remove IGMP state limit if lab-only                                   | ASA    | `interface <receiver-interface>` then `no igmp limit <limit>`                                             | IGMP limit override is removed                                                  |                                        |
|   10 | Remove IGMP query timeout if lab-only                                 | ASA    | `interface <receiver-interface>` then `no igmp query-timeout <seconds>`                                   | Query timeout override is removed                                               |                                        |
|   11 | Remove PIM interface DR priority if lab-only                          | ASA    | `interface <pim-interface>` then `no pim dr-priority <value>`                                             | DR priority override is removed                                                 |                                        |
|   12 | Remove PIM hello interval if lab-only                                 | ASA    | `interface <pim-interface>` then `no pim hello-interval <seconds>`                                        | PIM hello timer override is removed                                             |                                        |
|   13 | Remove PIM join/prune interval if lab-only                            | ASA    | `interface <pim-interface>` then `no pim join-prune-interval <seconds>`                                   | PIM join/prune timer override is removed                                        |                                        |
|   14 | Remove RP mapping                                                     | ASA    | `no pim rp-address <rp-ip>`                                                                               | RP mapping is removed                                                           |                                        |
|   15 | Remove bidirectional RP mapping                                       | ASA    | `no pim rp-address <rp-ip> bidir`                                                                         | Bidirectional RP mapping is removed                                             |                                        |
|   16 | Remove RP group ACL if unused                                         | ASA    | `clear configure access-list <rp-group-acl>`                                                              | RP group ACL is removed                                                         |                                        |
|   17 | Remove PIM neighbor filter                                            | ASA    | `interface <pim-interface>` then `no pim neighbor-filter <pim-filter-acl>`                                | PIM neighbor filter is removed from interface                                   |                                        |
|   18 | Remove PIM neighbor filter ACL if unused                              | ASA    | `clear configure access-list <pim-filter-acl>`                                                            | PIM filter ACL is removed                                                       |                                        |
|   19 | Remove old IOS register compatibility if lab-only                     | ASA    | `no pim old-register-checksum`                                                                            | Old register checksum behavior is disabled                                      |                                        |
|   20 | Remove static mroute                                                  | ASA    | `no mroute <source-ip-or-network> <mask> <in-interface-name> <out-interface-name-or-next-hop> <distance>` | Static multicast route is removed                                               |                                        |
|   21 | Disable PIM on selected interface if it should no longer participate  | ASA    | `interface <pim-interface>` then `no pim interface`                                                       | PIM is disabled on that interface                                               |                                        |
|   22 | Disable global multicast routing only if no multicast feature remains | ASA    | `no multicast-routing`                                                                                    | Global multicast routing is disabled                                            |                                        |
|   23 | Verify rollback                                                       | ASA    | `show mroute`                                                                                             | Removed multicast state ages out or disappears                                  |                                        |
|   24 | Verify PIM rollback                                                   | ASA    | `show pim neighbor`                                                                                       | Removed PIM neighbor relationships disappear                                    |                                        |
|   25 | Verify IGMP rollback                                                  | ASA    | `show igmp groups`                                                                                        | Removed static or dynamic groups disappear                                      |                                        |
|   26 | Stop any active debugs                                                | ASA    | `undebug all`                                                                                             | Debug output stops                                                              |                                        |
|   27 | Save rollback state                                                   | ASA    | `write memory`                                                                                            | Rollback is saved                                                               |                                        |

# ASA_Multicast_Failure_Checks
| Symptom                                        | Command                                                      | What Usually Broke                                                                                 |                                                                               |
| ---------------------------------------------- | ------------------------------------------------------------ | -------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------- |
| Multicast routing does not work at all         | `show firewall` and `show mode`                              | ASA is in transparent mode or multiple-context mode                                                |                                                                               |
| No multicast routes appear                     | `show running-config multicast-routing` and `show mroute`    | Global `multicast-routing` is missing                                                              |                                                                               |
| Receivers do not join group                    | `show igmp groups` and `show igmp interface`                 | Receiver not sending IGMP, IGMP disabled, wrong interface, wrong IGMP version, or switch issue     |                                                                               |
| Static group appears but traffic does not flow | `show mroute <multicast-group>`                              | Static IGMP group exists but RPF, PIM, RP, source, or ACL path is broken                           |                                                                               |
| IGMP state limit blocks joins                  | `show igmp groups` and `show igmp traffic`                   | `igmp limit` is too low                                                                            |                                                                               |
| ASA becomes unexpected IGMP querier            | `show igmp interface`                                        | Query timeout behavior or upstream querier behavior is wrong                                       |                                                                               |
| PIM neighbors do not form                      | `show pim neighbor`                                          | PIM disabled, interface down, ACL blocks 224.0.0.13, wrong segment, or neighbor filter blocks peer |                                                                               |
| PIM neighbor exists but no traffic flows       | `show mroute`, `show pim topology`                           | RP, RPF, mroute, source registration, or receiver join path is wrong                               |                                                                               |
| RP mapping is missing                          | `show pim group-map`                                         | `pim rp-address` missing or RP ACL does not include the group                                      |                                                                               |
| Group maps to wrong RP                         | `show pim group-map`                                         | RP group ACL or RP address is wrong                                                                |                                                                               |
| Source registers fail with older IOS router    | `debug pim`                                                  | Need `pim old-register-checksum` for old IOS interoperability                                      |                                                                               |
| DR election is wrong                           | `show pim interface`                                         | `pim dr-priority` is too high or too low on the wrong device                                       |                                                                               |
| Join/prune timing looks abnormal               | `show pim join-prune statistic`                              | PIM timer mismatch or overloaded multicast control plane                                           |                                                                               |
| RPF check fails                                | `show route <source-ip>` and `show mroute <multicast-group>` | Unicast route points to wrong interface or static `mroute` is needed                               |                                                                               |
| Static mroute does not work                    | `show running-config                                         | include ^mroute`                                                                                   | Source/mask/interface/next-hop syntax is wrong or route does not match source |
| Multicast traffic blocked by ACL               | `show access-list <interface-acl>`                           | Interface ACL blocks multicast group, source, PIM, or IGMP control traffic                         |                                                                               |
| Multicast traffic dropped by ASA               | `show asp drop`                                              | Drop reason points to ACL, RPF, multicast route, interface, or inspection issue                    |                                                                               |
| IGMP counters increment but PIM does not       | `show igmp traffic` and `show pim traffic`                   | Receiver joins ASA, but ASA is not building upstream PIM state                                     |                                                                               |
| PIM counters increment but IGMP does not       | `show pim traffic` and `show igmp traffic`                   | ASA has PIM control traffic but no receiver joins                                                  |                                                                               |
| `(*,G)` exists but no `(S,G)` appears          | `show mroute <multicast-group>`                              | Receiver joined group but no source traffic, RP/source registration issue, or RPF issue            |                                                                               |
| `(S,G)` exists but receiver gets nothing       | `show mroute`, `show conn`, `show asp drop`                  | OIL missing, receiver-side interface missing, ACL/drop, or downstream switch issue                 |                                                                               |
| Traffic floods where it should not             | `show mroute` and downstream switch checks                   | IGMP snooping/switch behavior, static group, or incorrect OIL behavior                             |                                                                               |
| Multicast over IPsec VPN fails                 | VPN and multicast design review                              | IPsec does not carry native multicast unless using an overlay or supported design workaround       |                                                                               |
| Debug output overwhelms ASA                    | `show debug`                                                 | Multicast debugs left enabled; run `undebug all`                                                   |                                                                               |