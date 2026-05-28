RIP_Maximum_Paths_ECMP.md
# RIP_Maximum_Paths_ECMP

# RIP_Maximum_Paths_ECMP_Mental_Model
| Concept | Operational Meaning |
|---|---|
| ECMP | Equal-cost multipath installs more than one route to the same prefix when the metric is equal |
| RIP metric | RIP only compares hop count, not bandwidth, delay, interface speed, or utilization |
| Equal RIP paths | Two paths with the same hop count can both be installed in the routing table |
| Maximum paths | `maximum-paths <NUMBER>` controls how many equal-cost RIP routes can be installed |
| No unequal-cost load balancing | RIP does not support EIGRP-style variance |
| Path eligibility | Routes must be for the same prefix, learned by RIP, have equal metric, and pass normal route selection |
| Forwarding behavior | The routing table may show multiple next hops, while CEF performs the actual load sharing |
| Verification target | The route table shows multiple RIP next hops for the same prefix and traffic can forward through either path |

# RIP_Maximum_Paths_ECMP_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm all routed links are up | R1/R2/R3/R4 | `show ip interface brief` | All ECMP transit and LAN interfaces are up/up |
| 2 | Confirm connected routes before enabling ECMP | R1/R2/R3/R4 | `show ip route connected` | Local LAN and transit networks appear as connected |
| 3 | Confirm baseline RIP configuration | R1/R2/R3/R4 | `show running-config | section router rip` | RIP is configured with `version 2`, `no auto-summary`, and expected `network` statements |
| 4 | Confirm the destination prefix can be reached through two equal-hop paths | R1 | `show ip route <DESTINATION_PREFIX>` | Route is learned through RIP or ready to be learned through RIP |
| 5 | Enter global configuration mode | R1 | `configure terminal` | R1 enters global configuration mode |
| 6 | Enter RIP router configuration mode | R1 | `router rip` | R1 enters RIP configuration mode |
| 7 | Set the number of allowed equal RIP paths | R1 | `maximum-paths <NUMBER>` | R1 can install up to the configured number of equal-cost RIP paths |
| 8 | Keep RIPv2 enabled | R1 | `version 2` | R1 uses RIPv2 updates |
| 9 | Keep auto-summary disabled | R1 | `no auto-summary` | R1 does not classfully summarize routes |
| 10 | Confirm RIP is enabled on the first equal-cost transit path | R1 | `network <PATH_1_MAJOR_NETWORK>` | RIP runs on the first transit interface |
| 11 | Confirm RIP is enabled on the second equal-cost transit path | R1 | `network <PATH_2_MAJOR_NETWORK>` | RIP runs on the second transit interface |
| 12 | Exit and save R1 configuration | R1 | `end` then `write memory` | R1 ECMP setting is saved |
| 13 | Enter global configuration mode on path routers | R2/R3 | `configure terminal` | Path routers enter global configuration mode |
| 14 | Enter RIP router configuration mode | R2/R3 | `router rip` | Path routers enter RIP configuration mode |
| 15 | Keep RIPv2 enabled on path routers | R2/R3 | `version 2` | Path routers use RIPv2 updates |
| 16 | Keep auto-summary disabled on path routers | R2/R3 | `no auto-summary` | Path routers advertise subnet routes correctly |
| 17 | Activate RIP on transit networks | R2/R3 | `network <TRANSIT_MAJOR_NETWORK>` | RIP runs on the expected transit interfaces |
| 18 | Exit and save path-router configuration | R2/R3 | `end` then `write memory` | Path-router RIP configuration is saved |
| 19 | Enter global configuration mode on destination router | R4 | `configure terminal` | Destination router enters global configuration mode |
| 20 | Enter RIP router configuration mode | R4 | `router rip` | R4 enters RIP configuration mode |
| 21 | Keep RIPv2 enabled on destination router | R4 | `version 2` | R4 uses RIPv2 updates |
| 22 | Keep auto-summary disabled on destination router | R4 | `no auto-summary` | R4 advertises subnet routes correctly |
| 23 | Advertise the destination LAN | R4 | `network <DESTINATION_LAN_MAJOR_NETWORK>` | Destination LAN is advertised into RIP |
| 24 | Advertise both upstream transit networks | R4 | `network <PATH_1_MAJOR_NETWORK>` and `network <PATH_2_MAJOR_NETWORK>` | R4 advertises reachability through both paths |
| 25 | Exit and save destination-router configuration | R4 | `end` then `write memory` | R4 RIP configuration is saved |
| 26 | Verify R1 RIP process ECMP setting | R1 | `show ip protocols` | Output shows RIP and the configured maximum path count |
| 27 | Verify multiple next hops for the destination prefix | R1 | `show ip route <DESTINATION_PREFIX>` | Same RIP prefix shows multiple equal-metric next hops |
| 28 | Verify all RIP-learned routes | R1 | `show ip route rip` | Destination prefix appears as a RIP route with equal-cost paths |
| 29 | Verify CEF forwarding entries | R1 | `show ip cef <DESTINATION_PREFIX>` | CEF shows forwarding through multiple next hops |
| 30 | Test reachability to destination LAN | R1 | `ping <DESTINATION_LAN_IP>` | Ping succeeds |
| 31 | Confirm traffic path selection | R1 | `traceroute <DESTINATION_LAN_IP>` | Traffic uses one of the valid equal-cost next hops |
| 32 | Confirm alternate path survives failure testing | R1 | `show ip route <DESTINATION_PREFIX>` after shutting one path | Route remains reachable through the remaining next hop |
| 33 | Restore failed test interface | R1/R2/R3/R4 | `no shutdown` | Interface returns up/up and ECMP can reconverge |

# RIP_Maximum_Paths_ECMP_Skeleton
| Device | Configuration |
|---|---|
| R1 | `configure terminal` |
| R1 | `router rip` |
| R1 | `version 2` |
| R1 | `no auto-summary` |
| R1 | `maximum-paths <NUMBER>` |
| R1 | `network <PATH_1_MAJOR_NETWORK>` |
| R1 | `network <PATH_2_MAJOR_NETWORK>` |
| R1 | `network <R1_LAN_MAJOR_NETWORK>` |
| R1 | `end` |
| R1 | `write memory` |
| R2 | `configure terminal` |
| R2 | `router rip` |
| R2 | `version 2` |
| R2 | `no auto-summary` |
| R2 | `network <R1_R2_MAJOR_NETWORK>` |
| R2 | `network <R2_R4_MAJOR_NETWORK>` |
| R2 | `end` |
| R2 | `write memory` |
| R3 | `configure terminal` |
| R3 | `router rip` |
| R3 | `version 2` |
| R3 | `no auto-summary` |
| R3 | `network <R1_R3_MAJOR_NETWORK>` |
| R3 | `network <R3_R4_MAJOR_NETWORK>` |
| R3 | `end` |
| R3 | `write memory` |
| R4 | `configure terminal` |
| R4 | `router rip` |
| R4 | `version 2` |
| R4 | `no auto-summary` |
| R4 | `network <R2_R4_MAJOR_NETWORK>` |
| R4 | `network <R3_R4_MAJOR_NETWORK>` |
| R4 | `network <DESTINATION_LAN_MAJOR_NETWORK>` |
| R4 | `end` |
| R4 | `write memory` |

# RIP_Maximum_Paths_ECMP_Verification_Commands
| Command | Purpose | Good Output |
|---|---|---|
| `show ip interface brief` | Confirms interface state | All ECMP transit interfaces are up/up |
| `show running-config | section router rip` | Confirms RIP and `maximum-paths` configuration | Shows `version 2`, `no auto-summary`, expected `network` statements, and `maximum-paths <NUMBER>` |
| `show ip protocols` | Confirms RIP process behavior | Shows RIP networks and maximum path behavior |
| `show ip route rip` | Displays RIP-learned routes | Destination route appears as an `R` route |
| `show ip route <DESTINATION_PREFIX>` | Confirms multiple equal-cost next hops | Same prefix shows multiple next hops with equal RIP metric |
| `show ip cef <DESTINATION_PREFIX>` | Confirms forwarding-plane entries | CEF shows multiple next-hop adjacencies |
| `show adjacency` | Confirms CEF adjacency state | Next-hop adjacencies are valid |
| `ping <DESTINATION_LAN_IP>` | Tests reachability | ICMP succeeds |
| `traceroute <DESTINATION_LAN_IP>` | Shows selected forwarding path | Path uses one of the valid ECMP next hops |
| `debug ip rip` | Confirms equal metric route advertisements | Updates show the same destination prefix learned with equal hop count |
| `undebug all` | Stops debugging | Debugging is disabled |

# RIP_Maximum_Paths_ECMP_Rollback
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Enter global configuration mode | R1 | `configure terminal` | R1 enters global configuration mode |
| 2 | Enter RIP router configuration mode | R1 | `router rip` | R1 enters RIP configuration mode |
| 3 | Reduce RIP to single-path routing if the lab requires ECMP removal | R1 | `maximum-paths 1` | R1 installs only one RIP path per prefix |
| 4 | Exit configuration mode | R1 | `end` | R1 returns to privileged EXEC mode |
| 5 | Verify ECMP is removed | R1 | `show ip route <DESTINATION_PREFIX>` | Only one RIP next hop is installed |
| 6 | Verify CEF has one forwarding path | R1 | `show ip cef <DESTINATION_PREFIX>` | CEF shows one active next hop |
| 7 | Save rollback | R1 | `write memory` | Rollback persists after reload |
| 8 | Optional, remove RIP entirely for full lab reset | R1/R2/R3/R4 | `configure terminal` then `no router rip` | RIP process is removed from all routers |
| 9 | Confirm full RIP removal if used | R1/R2/R3/R4 | `show ip route rip` | No RIP-learned routes remain |

# RIP_Maximum_Paths_ECMP_Failure_Checks
| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| Only one path is installed | `maximum-paths` is set to `1` or too low | `show running-config | section router rip` | Configure `maximum-paths <NUMBER>` greater than 1 |
| Only one path is installed | RIP metrics are not equal | `show ip route <DESTINATION_PREFIX>` | Fix topology or metric manipulation so both paths have equal hop count |
| Only one path is installed | One path is not learning the destination prefix | `show ip route rip` on path routers | Add missing RIP `network` statement or fix interface state |
| Both paths exist but one is not used | CEF adjacency issue | `show ip cef <DESTINATION_PREFIX>` and `show adjacency` | Fix next-hop reachability or interface adjacency problem |
| Route is missing completely | Destination LAN is not advertised into RIP | `show ip protocols` on destination router | Add `network <DESTINATION_LAN_MAJOR_NETWORK>` |
| Route is summarized unexpectedly | Auto-summary is enabled | `show ip protocols` | Configure `no auto-summary` |
| Route points through wrong router | RIP sees a different path with lower hop count | `show ip route <DESTINATION_PREFIX>` | Correct topology or metric manipulation |
| ECMP disappears after failure test | Failed path did not recover | `show ip interface brief` | Restore interface with `no shutdown` and verify cabling or CML link state |
| Ping works but traceroute shows one path only | Per-flow or per-destination CEF behavior selects one next hop for that flow | `show ip cef <DESTINATION_PREFIX>` | Verify CEF has multiple next hops instead of expecting every traceroute to alternate |
| Debug shows unequal advertised metrics | Offset-list or topology difference changed hop count | `debug ip rip` | Remove offset-list or correct the path design |
| RIP route is not installed | A route from another source wins administrative distance | `show ip route <DESTINATION_PREFIX>` | Remove or adjust competing static, OSPF, EIGRP, or BGP route |
| ECMP setting disappears after reload | Configuration was not saved | `show startup-config | section router rip` | Reconfigure and run `write memory` |

# Index
RIP_Maximum_Paths_ECMP.md
RIP_Maximum_Paths_ECMP
RIP_Maximum_Paths_ECMP_Mental_Model
RIP_Maximum_Paths_ECMP_Configuration_Checklist
RIP_Maximum_Paths_ECMP_Skeleton
RIP_Maximum_Paths_ECMP_Verification_Commands
RIP_Maximum_Paths_ECMP_Rollback
RIP_Maximum_Paths_ECMP_Failure_Checks
