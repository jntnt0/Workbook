RIP_Three_Router_Distance_Vector_Propagation.md
# RIP_Three_Router_Distance_Vector_Propagation

# RIP_Three_Router_Distance_Vector_Propagation_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Three-router propagation | R1 does not need to be directly connected to R3 to learn R3 routes; R2 advertises what it knows to both sides |
| Distance vector | RIP advertises destination networks with a distance value, where distance is hop count |
| Vector | The next-hop router is the direction traffic follows toward the destination |
| Hop-count increment | Each router that receives a RIP route increments the metric by 1 before considering it for installation or re-advertisement |
| Metric limit | Hop count 15 is reachable; hop count 16 is unreachable |
| RIPv2 update behavior | RIPv2 sends routing updates to multicast address `224.0.0.9` using UDP port `520` |
| No topology map | RIP does not build a full link-state database; it trusts neighbor advertisements |
| Middle-router role | R2 becomes the propagation point between R1 and R3 |
| Expected R1 view | R1 should learn R3 LAN routes through R2 as RIP routes |
| Expected R3 view | R3 should learn R1 LAN routes through R2 as RIP routes |
| Expected R2 view | R2 should learn R1 LAN and R3 LAN routes with lower hop count because it is directly between them |
| Verification target | A working three-router RIP domain has RIPv2 enabled on all routed links, no auto-summary, correct network activation, and remote `R` routes installed across the full chain |

# RIP_Three_Router_Distance_Vector_Propagation_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm R1 routed interfaces are up | R1 | `show ip interface brief` | R1 LAN and R1-to-R2 transit interfaces are up/up |
| 2 | Confirm R2 routed interfaces are up | R2 | `show ip interface brief` | R2-to-R1 and R2-to-R3 transit interfaces are up/up |
| 3 | Confirm R3 routed interfaces are up | R3 | `show ip interface brief` | R3 LAN and R3-to-R2 transit interfaces are up/up |
| 4 | Confirm R1 connected routes before RIP | R1 | `show ip route connected` | R1 LAN and R1-to-R2 transit networks appear as connected |
| 5 | Confirm R2 connected routes before RIP | R2 | `show ip route connected` | R2-to-R1 and R2-to-R3 transit networks appear as connected |
| 6 | Confirm R3 connected routes before RIP | R3 | `show ip route connected` | R3 LAN and R3-to-R2 transit networks appear as connected |
| 7 | Enter global configuration mode | R1 | `configure terminal` | R1 enters global configuration mode |
| 8 | Start RIP on R1 | R1 | `router rip` | R1 enters RIP router configuration mode |
| 9 | Force RIPv2 on R1 | R1 | `version 2` | R1 uses RIPv2 updates |
| 10 | Disable auto-summary on R1 | R1 | `no auto-summary` | R1 does not perform classful summarization |
| 11 | Activate RIP on the R1-to-R2 transit major network | R1 | `network <R1_R2_TRANSIT_MAJOR_NETWORK>` | RIP runs on the R1 transit interface |
| 12 | Activate RIP on the R1 LAN major network | R1 | `network <R1_LAN_MAJOR_NETWORK>` | R1 advertises its LAN into RIP |
| 13 | Exit and save R1 configuration | R1 | `end` then `write memory` | R1 RIP configuration is saved |
| 14 | Enter global configuration mode | R2 | `configure terminal` | R2 enters global configuration mode |
| 15 | Start RIP on R2 | R2 | `router rip` | R2 enters RIP router configuration mode |
| 16 | Force RIPv2 on R2 | R2 | `version 2` | R2 uses RIPv2 updates |
| 17 | Disable auto-summary on R2 | R2 | `no auto-summary` | R2 does not perform classful summarization |
| 18 | Activate RIP on the R1-to-R2 transit major network | R2 | `network <R1_R2_TRANSIT_MAJOR_NETWORK>` | RIP runs on R2 interface toward R1 |
| 19 | Activate RIP on the R2-to-R3 transit major network | R2 | `network <R2_R3_TRANSIT_MAJOR_NETWORK>` | RIP runs on R2 interface toward R3 |
| 20 | Exit and save R2 configuration | R2 | `end` then `write memory` | R2 RIP configuration is saved |
| 21 | Enter global configuration mode | R3 | `configure terminal` | R3 enters global configuration mode |
| 22 | Start RIP on R3 | R3 | `router rip` | R3 enters RIP router configuration mode |
| 23 | Force RIPv2 on R3 | R3 | `version 2` | R3 uses RIPv2 updates |
| 24 | Disable auto-summary on R3 | R3 | `no auto-summary` | R3 does not perform classful summarization |
| 25 | Activate RIP on the R2-to-R3 transit major network | R3 | `network <R2_R3_TRANSIT_MAJOR_NETWORK>` | RIP runs on R3 interface toward R2 |
| 26 | Activate RIP on the R3 LAN major network | R3 | `network <R3_LAN_MAJOR_NETWORK>` | R3 advertises its LAN into RIP |
| 27 | Exit and save R3 configuration | R3 | `end` then `write memory` | R3 RIP configuration is saved |
| 28 | Verify RIP process behavior on R1 | R1 | `show ip protocols` | Output shows RIP, version 2, no auto-summary, and expected routing-for-networks entries |
| 29 | Verify RIP process behavior on R2 | R2 | `show ip protocols` | Output shows RIP active on both transit networks |
| 30 | Verify RIP process behavior on R3 | R3 | `show ip protocols` | Output shows RIP, version 2, no auto-summary, and expected routing-for-networks entries |
| 31 | Verify R1 learned R3 LAN through R2 | R1 | `show ip route <R3_LAN_PREFIX>` | Route appears as `R` with next hop toward R2 |
| 32 | Verify R3 learned R1 LAN through R2 | R3 | `show ip route <R1_LAN_PREFIX>` | Route appears as `R` with next hop toward R2 |
| 33 | Verify R2 learned both edge LANs | R2 | `show ip route rip` | R2 has RIP routes for R1 LAN and R3 LAN |
| 34 | Confirm end-to-end reachability from R1 to R3 LAN | R1 | `ping <R3_LAN_IP>` | Ping succeeds |
| 35 | Confirm end-to-end reachability from R3 to R1 LAN | R3 | `ping <R1_LAN_IP>` | Ping succeeds |
| 36 | Confirm forwarding path from R1 to R3 LAN | R1 | `traceroute <R3_LAN_IP>` | Path goes R1 to R2 to R3 |
| 37 | Confirm forwarding path from R3 to R1 LAN | R3 | `traceroute <R1_LAN_IP>` | Path goes R3 to R2 to R1 |

# RIP_Three_Router_Distance_Vector_Propagation_Skeleton
| Device | Configuration |
|---|---|
| R1 | `configure terminal` |
| R1 | `router rip` |
| R1 | `version 2` |
| R1 | `no auto-summary` |
| R1 | `network <R1_R2_TRANSIT_MAJOR_NETWORK>` |
| R1 | `network <R1_LAN_MAJOR_NETWORK>` |
| R1 | `end` |
| R1 | `write memory` |
| R2 | `configure terminal` |
| R2 | `router rip` |
| R2 | `version 2` |
| R2 | `no auto-summary` |
| R2 | `network <R1_R2_TRANSIT_MAJOR_NETWORK>` |
| R2 | `network <R2_R3_TRANSIT_MAJOR_NETWORK>` |
| R2 | `end` |
| R2 | `write memory` |
| R3 | `configure terminal` |
| R3 | `router rip` |
| R3 | `version 2` |
| R3 | `no auto-summary` |
| R3 | `network <R2_R3_TRANSIT_MAJOR_NETWORK>` |
| R3 | `network <R3_LAN_MAJOR_NETWORK>` |
| R3 | `end` |
| R3 | `write memory` |

# RIP_Three_Router_Distance_Vector_Propagation_Verification_Commands
| Command | Purpose | Good Output |
|---|---|---|
| `show ip interface brief` | Confirms physical and logical interface state | Required routed interfaces are up/up |
| `show ip route connected` | Confirms locally connected routes before RIP advertisement | LAN and transit prefixes appear as connected |
| `show running-config | section router rip` | Confirms RIP configuration | Shows `router rip`, `version 2`, `no auto-summary`, and correct `network` statements |
| `show ip protocols` | Confirms RIP process settings and activated networks | Shows RIP running for expected networks |
| `show ip route rip` | Displays all RIP-learned routes | Remote routes appear with code `R` |
| `show ip route <R1_LAN_PREFIX>` | Confirms exact route to R1 LAN | R2 or R3 has route pointing toward R1 side |
| `show ip route <R3_LAN_PREFIX>` | Confirms exact route to R3 LAN | R1 or R2 has route pointing toward R3 side |
| `show ip route <REMOTE_PREFIX>` | Confirms next hop and metric for a specific route | Route appears as `[120/<hop_count>] via <next_hop>` |
| `ping <REMOTE_LAN_IP>` | Tests end-to-end data-plane reachability | ICMP succeeds |
| `traceroute <REMOTE_LAN_IP>` | Confirms the path through the middle router | Path crosses R2 between R1 and R3 |
| `debug ip rip` | Shows live RIP updates being sent and received | Updates show advertised prefixes and metrics |
| `undebug all` | Stops debugging | Debugging is disabled |

# RIP_Three_Router_Distance_Vector_Propagation_Rollback
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Enter global configuration mode on R1 | R1 | `configure terminal` | R1 enters global configuration mode |
| 2 | Remove RIP from R1 | R1 | `no router rip` | RIP process is removed from R1 |
| 3 | Save R1 rollback | R1 | `end` then `write memory` | R1 rollback is saved |
| 4 | Enter global configuration mode on R2 | R2 | `configure terminal` | R2 enters global configuration mode |
| 5 | Remove RIP from R2 | R2 | `no router rip` | RIP process is removed from R2 |
| 6 | Save R2 rollback | R2 | `end` then `write memory` | R2 rollback is saved |
| 7 | Enter global configuration mode on R3 | R3 | `configure terminal` | R3 enters global configuration mode |
| 8 | Remove RIP from R3 | R3 | `no router rip` | RIP process is removed from R3 |
| 9 | Save R3 rollback | R3 | `end` then `write memory` | R3 rollback is saved |
| 10 | Confirm RIP is removed | R1/R2/R3 | `show running-config | section router rip` | No RIP process appears |
| 11 | Confirm RIP routes are gone | R1/R2/R3 | `show ip route rip` | No RIP-learned routes appear |

# RIP_Three_Router_Distance_Vector_Propagation_Failure_Checks
| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| R1 does not learn R3 LAN | R2 or R3 is not advertising the R3 LAN | `show ip protocols` on R2 and R3 | Add the missing `network <R3_LAN_MAJOR_NETWORK>` or transit network statement |
| R3 does not learn R1 LAN | R1 or R2 is not advertising the R1 LAN | `show ip protocols` on R1 and R2 | Add the missing `network <R1_LAN_MAJOR_NETWORK>` or transit network statement |
| R2 learns no routes from R1 | RIP not enabled on the R1-to-R2 transit link | `show running-config | section router rip` | Add `network <R1_R2_TRANSIT_MAJOR_NETWORK>` on both R1 and R2 |
| R2 learns no routes from R3 | RIP not enabled on the R2-to-R3 transit link | `show running-config | section router rip` | Add `network <R2_R3_TRANSIT_MAJOR_NETWORK>` on both R2 and R3 |
| Remote route appears summarized incorrectly | Auto-summary is enabled | `show ip protocols` | Configure `no auto-summary` under `router rip` |
| Route appears but ping fails | Return path is missing | `show ip route <SOURCE_LAN_PREFIX>` on the far router | Make sure both edge LANs are advertised into RIP |
| Route appears with unexpected next hop | RIP selected a different equal or lower hop-count path | `show ip route <REMOTE_PREFIX>` | Verify topology, metrics, and any route manipulation |
| Route is missing but connected links are up | Wrong classful `network` statement used | `show ip protocols` | Correct the `network <MAJOR_NETWORK>` statement |
| RIP updates are not seen during debug | RIP is not active on the interface or packets are blocked | `debug ip rip` and `show ip protocols` | Enable RIP on the correct interface network and verify interface state |
| RIP route is not installed | Better administrative distance route exists from another source | `show ip route <PREFIX>` | Remove or adjust competing static, OSPF, EIGRP, or BGP route |
| Route becomes unreachable | Hop count reached 16 | `show ip route rip` and `debug ip rip` | Fix loop, bad metric manipulation, or excessive path length |
| Traceroute stops at R2 | R2 lacks route toward destination or return path is broken | `show ip route <REMOTE_PREFIX>` on R2 | Fix missing RIP advertisement on the far-side network |

# Index
RIP_Three_Router_Distance_Vector_Propagation.md
RIP_Three_Router_Distance_Vector_Propagation
RIP_Three_Router_Distance_Vector_Propagation_Mental_Model
RIP_Three_Router_Distance_Vector_Propagation_Configuration_Checklist
RIP_Three_Router_Distance_Vector_Propagation_Skeleton
RIP_Three_Router_Distance_Vector_Propagation_Verification_Commands
RIP_Three_Router_Distance_Vector_Propagation_Rollback
RIP_Three_Router_Distance_Vector_Propagation_Failure_Checks